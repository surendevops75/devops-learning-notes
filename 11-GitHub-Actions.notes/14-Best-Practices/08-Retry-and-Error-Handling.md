# Retry and Error Handling

Retry and error handling are essential parts of reliable DevOps automation.

Production systems can fail because of:

    Network Problems
    +
    Temporary Service Failures
    +
    Resource Constraints
    +
    Configuration Errors
    +
    Authentication Failures
    +
    Application Failures
    +
    Infrastructure Failures

A reliable system should distinguish between:

    Temporary Failure
        +
    Permanent Failure

and respond appropriately.

The goal is:

    Detect Failure
        |
        ↓
    Understand Failure
        |
        ↓
    Retry If Safe
        |
        ↓
    Recover
        |
        ↓
    Validate
        |
        ↓
    Continue

If recovery is not possible:

    Fail Clearly
        +
    Preserve Evidence
        +
    Notify
        +
    Roll Back / Recover

---

# 1. What Is Error Handling?

Error handling is the process of detecting, managing, and responding to failures.

Example:

    Application
        |
        ↓
    Database Request
        |
        X
    Connection Failed
        |
        ↓
    Error Handling
        |
        +------ Retry
        |
        +------ Fail
        |
        +------ Fallback

---

# 2. What Is Retry?

Retry means attempting an operation again after a failure.

Example:

    Request
        |
        X
    Temporary Failure
        |
        ↓
    Retry
        |
        ↓
    Success

Retries are useful when the failure is temporary.

---

# 3. Why Retries Matter

Distributed systems frequently experience temporary failures.

Examples:

    Network Timeout
    +
    Temporary DNS Failure
    +
    API Rate Limit
    +
    Service Unavailable
    +
    Connection Reset
    +
    Temporary Cloud API Failure

A retry can allow the operation to succeed without manual intervention.

---

# 4. Retry Is Not Always the Answer

Do not retry every error.

Example:

    Invalid Password
        |
        ↓
    Retry
        |
        ↓
    Invalid Password
        |
        ↓
    Retry Again
        |
        ↓
    Same Failure

This creates unnecessary load.

---

# 5. Transient vs Permanent Errors

## Transient Error

Temporary problem.

Examples:

    Timeout
    +
    Connection Reset
    +
    HTTP 503
    +
    Temporary Network Failure

Potential response:

    Retry

## Permanent Error

Retry will not normally fix the underlying problem.

Examples:

    Invalid Configuration
    +
    Invalid Credentials
    +
    Missing Required File
    +
    Invalid Dockerfile
    +
    Invalid Terraform Configuration

Potential response:

    Fail
        +
    Investigate

---

# 6. Error Classification

A useful model:

    Error
        |
        ↓
    Classify
        |
        +------ Transient → Retry
        |
        +------ Permanent → Fail
        |
        +------ Unknown → Investigate Carefully

---

# 7. Retry Decision

Before retrying ask:

    Is The Error Temporary?

    Is The Operation Safe To Retry?

    Could Retry Create A Duplicate?

    How Many Times Should We Retry?

    How Long Should We Wait?

    Will Retry Increase System Load?

    What Happens If All Retries Fail?

---

# 8. Retry and Idempotency

Retries and idempotency are closely related.

Example:

    Operation
        |
        X
    Network Failure
        |
        ↓
    Retry

If operation is idempotent:

    Safer Retry

If operation is non-idempotent:

    Possible Duplicate Side Effect

Therefore:

    Retry
        +
    Idempotency
        |
        ↓
    Safer Automation

---

# 9. Simple Retry

Basic pattern:

    Attempt 1
        |
        X
    Attempt 2
        |
        X
    Attempt 3
        |
        ↓
    Success

But immediate retries are not always ideal.

---

# 10. Immediate Retry Problem

Example:

    Service Overloaded
        |
        ↓
    Request Fails
        |
        ↓
    Immediate Retry
        |
        ↓
    Service Still Overloaded
        |
        ↓
    More Requests

This can make the problem worse.

---

# 11. Fixed Delay Retry

Wait the same amount of time between attempts.

Example:

    Attempt 1
        |
        X
      Wait 5s
        |
        ↓
    Attempt 2
        |
        X
      Wait 5s
        |
        ↓
    Attempt 3

Simple but can cause synchronized retry traffic.

---

# 12. Exponential Backoff

Increase the delay after every failure.

Example:

    Attempt 1
        |
        X
      Wait 1s
        |
        ↓
    Attempt 2
        |
        X
      Wait 2s
        |
        ↓
    Attempt 3
        |
        X
      Wait 4s
        |
        ↓
    Attempt 4

Typical pattern:

    1
    2
    4
    8
    16

seconds.

---

# 13. Why Exponential Backoff Helps

Instead of:

    Retry
    Retry
    Retry
    Retry

use:

    Retry
        |
        ↓
    Wait
        |
        ↓
    Retry
        |
        ↓
    Longer Wait
        |
        ↓
    Retry

This gives the dependent service time to recover.

---

# 14. Maximum Retry Delay

Do not allow exponential backoff to grow forever.

Example:

    1s
    2s
    4s
    8s
    16s
    30s
    30s

Maximum delay:

    30 seconds

---

# 15. Maximum Retry Count

Retries should have a limit.

Example:

    Maximum Attempts = 3

Flow:

    Attempt 1
        |
        X
    Attempt 2
        |
        X
    Attempt 3
        |
        X
    Final Failure

Then:

    Stop Retrying
        +
    Report Error

---

# 16. Retry Budget

A retry budget limits how much retry traffic a system can generate.

Without a limit:

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
    Retry Storm

With a retry budget:

    Failure
        |
        ↓
    Limited Retries
        |
        ↓
    Stop
        |
        ↓
    Recover / Fail

---

# 17. Jitter

Jitter adds randomness to retry delays.

Without jitter:

    Service Failure
        |
        +--- Client A → Retry at 5s
        +--- Client B → Retry at 5s
        +--- Client C → Retry at 5s
        +--- Client D → Retry at 5s

All clients retry together.

---

# 18. Retry Storm

A retry storm occurs when many clients repeatedly retry a failing service.

Example:

    Service Failure
        |
        ↓
    1,000 Clients
        |
        ↓
    1,000 Retries
        |
        ↓
    Service Overloaded
        |
        ↓
    More Failures
        |
        ↓
    More Retries

This creates a feedback loop.

---

# 19. Jitter Prevents Synchronization

With jitter:

    Client A → Retry 4.2s
    Client B → Retry 5.8s
    Client C → Retry 3.9s
    Client D → Retry 6.1s

Retries become distributed over time.

---

# 20. Exponential Backoff + Jitter

A common production pattern is:

    Failure
        |
        ↓
    Exponential Backoff
        +
    Jitter
        |
        ↓
    Retry

This reduces synchronized retry traffic.

---

# 21. Timeout

A timeout defines how long an operation can wait.

Example:

    API Request
        |
        ↓
    Wait
        |
        ↓
    10 Seconds
        |
        X
    Timeout

Without timeouts:

    Request
        |
        ↓
    Waiting
        |
        ↓
    Waiting
        |
        ↓
    Waiting Forever

---

# 22. Why Timeouts Matter

Timeouts prevent resources from being held indefinitely.

They protect:

    Threads
    +
    Connections
    +
    Workers
    +
    Pipeline Jobs
    +
    Application Resources

---

# 23. Connection Timeout

Time allowed to establish a connection.

Example:

    Client
        |
        ↓
    Connect
        |
        X
    Timeout

---

# 24. Read Timeout

Time allowed to receive a response after connection.

Example:

    Connection Established
        |
        ↓
    Waiting For Response
        |
        X
    Read Timeout

---

# 25. Pipeline Timeout

CI/CD jobs should have appropriate timeouts.

Example:

    Build
        |
        ↓
    Test
        |
        ↓
    Deployment
        |
        X
    Unexpected Hang

Timeout:

    Stop Job
        +
    Report Failure

---

# 26. Retry vs Timeout

Timeout:

    How Long Should I Wait?

Retry:

    Should I Try Again?

They solve different problems.

Example:

    API Request
        |
        ↓
    Wait 10s
        |
        X
    Timeout
        |
        ↓
    Retry

---

# 27. Retry + Timeout

Use both carefully.

Example:

    Request
        |
        ↓
    Timeout = 10s
        |
        X
    Retry
        |
        ↓
    Timeout = 10s
        |
        X
    Retry
        |
        ↓
    Final Failure

Also consider the total operation deadline.

---

# 28. Total Retry Deadline

Suppose:

    Total Allowed Time = 30s

Do not allow:

    Attempt 1 = 10s
    Attempt 2 = 10s
    Attempt 3 = 10s
    Backoff = additional time

to exceed the overall deadline.

---

# 29. Circuit Breaker

A circuit breaker prevents repeated calls to a failing dependency.

States:

    CLOSED
        |
        ↓
    OPEN
        |
        ↓
    HALF-OPEN

---

# 30. Closed State

Normal operation:

    Application
        |
        ↓
    Dependency
        |
        ↓
    Success

Requests flow normally.

---

# 31. Open State

After repeated failures:

    Application
        |
        X
    Dependency Failing
        |
        ↓
    Circuit Opens
        |
        ↓
    Stop Calling Dependency

This prevents additional load.

---

# 32. Half-Open State

After waiting:

    Circuit Open
        |
        ↓
    Wait
        |
        ↓
    Half-Open
        |
        ↓
    Test Request

If successful:

    CLOSED

If failure:

    OPEN

---

# 33. Circuit Breaker Flow

    CLOSED
      |
      | Failures exceed threshold
      ↓
    OPEN
      |
      | Wait
      ↓
    HALF-OPEN
      |
      +------ Success → CLOSED
      |
      +------ Failure → OPEN

---

# 34. Retry vs Circuit Breaker

Retry:

    Try Again

Circuit Breaker:

    Stop Trying For Now

Use them together carefully.

Example:

    Temporary Failure
        |
        ↓
    Limited Retry

If failures continue:

    Circuit Opens

---

# 35. Bulkhead Pattern

Bulkhead isolation prevents one failure from consuming all resources.

Example:

    Service A
        |
        ↓
    Connection Pool A

    Service B
        |
        ↓
    Connection Pool B

If Service A fails:

    Service B
        |
        ↓
    Still Has Resources

---

# 36. Error Propagation

A failure in one service can affect another.

Example:

    API
        |
        ↓
    Payment Service
        |
        ↓
    Database
        X
    Failure

Without proper handling:

    Database Failure
        |
        ↓
    Payment Failure
        |
        ↓
    API Failure
        |
        ↓
    User Impact

---

# 37. Graceful Failure

Instead of allowing uncontrolled failure:

    Dependency Failure
        |
        ↓
    Detect
        |
        ↓
    Handle
        |
        +------ Retry
        |
        +------ Fallback
        |
        +------ Return Controlled Error

---

# 38. Fallback

A fallback provides an alternative response when a dependency fails.

Example:

    Product Recommendation Service
        |
        X
    Failure
        |
        ↓
    Fallback
        |
        ↓
    Default Recommendations

Use fallbacks only when they preserve acceptable behavior.

---

# 39. Fail Fast

Fail fast when continuing cannot succeed.

Example:

    Invalid Configuration
        |
        ↓
    Detect
        |
        ↓
    Fail Immediately

Do not waste time retrying a permanent error.

---

# 40. Fail Safe

Fail safe means failing in a way that minimizes harm.

Examples:

    Reject Unsafe Operation
    +
    Preserve Data
    +
    Protect Security
    +
    Maintain System Integrity

---

# 41. Error Messages

Good error messages should explain:

    What Failed
        +
    Where It Failed
        +
    Why It Failed
        +
    What Can Be Done

Bad:

    Error occurred.

Better:

    Terraform failed to create the EKS node group because the
    specified IAM role could not be assumed.

---

# 42. Preserve Root Cause

Do not hide the original error.

Example:

    Deployment Failed
        |
        ↓
    Kubernetes Error
        |
        ↓
    ImagePullBackOff
        |
        ↓
    Registry Authentication Failed

The final message should preserve useful context.

---

# 43. Error Logging

Record:

    Timestamp
    +
    Service
    +
    Error Type
    +
    Error Message
    +
    Request ID
    +
    Environment
    +
    Version

Avoid logging:

    Passwords
    +
    Tokens
    +
    Private Keys
    +
    Sensitive Information

---

# 44. Structured Error Logging

Example:

    {
      "level": "ERROR",
      "service": "payment-service",
      "error": "database_timeout",
      "request_id": "abc123",
      "environment": "production"
    }

Structured logs make searching easier.

---

# 45. Error Categories

Useful categories:

    Validation Error
    +
    Authentication Error
    +
    Authorization Error
    +
    Network Error
    +
    Timeout
    +
    Dependency Error
    +
    Resource Error
    +
    Configuration Error
    +
    Application Error

---

# 46. Validation Error

Example:

    Invalid Input
        |
        ↓
    Validation Error

Usually:

    Do Not Retry

Fix the input instead.

---

# 47. Authentication Error

Example:

    Invalid Credentials

Usually:

    Do Not Retry Repeatedly

Instead:

    Validate Credentials
        +
    Check Secret
        +
    Check Token
        +
    Check Permissions

---

# 48. Authorization Error

Example:

    Access Denied

Repeated retries usually will not fix missing permissions.

Investigate:

    IAM
    +
    RBAC
    +
    Policies
    +
    Roles

---

# 49. Network Error

Examples:

    Connection Reset
    +
    Connection Refused
    +
    DNS Failure
    +
    Network Timeout

Some network errors may be transient.

Use:

    Timeout
        +
    Limited Retry
        +
    Backoff

---

# 50. HTTP 429

HTTP 429 means:

    Too Many Requests

The client may have exceeded a rate limit.

Approach:

    Respect Retry-After When Provided
        +
    Backoff
        +
    Jitter
        +
    Retry Limit

---

# 51. HTTP 500

HTTP 500 indicates an internal server error.

It may be transient or persistent.

Approach:

    Limited Retry
        +
    Backoff
        +
    Monitoring
        +
    Root Cause Investigation

---

# 52. HTTP 502

HTTP 502 indicates a bad gateway response.

Potential causes:

    Upstream Failure
    +
    Load Balancer Problem
    +
    Proxy Problem
    +
    Network Issue

Check:

    Load Balancer
    +
    Upstream
    +
    Application
    +
    Logs

---

# 53. HTTP 503

HTTP 503 means service unavailable.

Possible causes:

    No Healthy Pods
    +
    Overload
    +
    Deployment
    +
    Dependency Failure

Limited retries may be appropriate depending on the request and system behavior.

---

# 54. HTTP 504

HTTP 504 indicates a gateway timeout.

Possible causes:

    Slow Application
    +
    Slow Database
    +
    Network Delay
    +
    Upstream Timeout

Investigate the full request path.

---

# 55. HTTP 400

HTTP 400 usually indicates a client request problem.

Generally:

    Do Not Retry Automatically

Fix the request.

---

# 56. HTTP 401

HTTP 401 indicates authentication is required or invalid.

Usually:

    Validate Credentials
        +
    Refresh Token If Appropriate
        +
    Retry Only If Safe

Do not endlessly retry invalid credentials.

---

# 57. HTTP 403

HTTP 403 indicates forbidden access.

Usually:

    Do Not Blindly Retry

Check:

    IAM
    +
    RBAC
    +
    Authorization Policy

---

# 58. HTTP 404

HTTP 404 indicates the requested resource was not found.

Usually:

    Do Not Retry Repeatedly

unless the resource may become available as part of a known asynchronous workflow.

---

# 59. HTTP Error Handling Model

    HTTP Response
        |
        ↓
    Classify
        |
        +------ 2xx → Success
        |
        +------ 4xx → Usually Client / Permission Problem
        |
        +------ 5xx → Potential Server / Dependency Problem
        |
        +------ 429 → Rate Limited
        |
        +------ Timeout → Potentially Transient

---

# 60. Kubernetes Error Handling

Kubernetes workloads can fail because of:

    ImagePullBackOff
    +
    CrashLoopBackOff
    +
    OOMKilled
    +
    FailedScheduling
    +
    Probe Failures
    +
    FailedMount

Do not simply restart pods repeatedly.

Investigate the underlying cause.

---

# 61. CrashLoopBackOff

Flow:

    Pod Starts
        |
        X
    Application Crashes
        |
        ↓
    Kubernetes Restarts
        |
        X
    Application Crashes Again
        |
        ↓
    Backoff

Investigate:

    kubectl logs <pod> --previous

and:

    kubectl describe pod <pod>

---

# 62. ImagePullBackOff

Possible causes:

    Wrong Image Name
    +
    Wrong Tag
    +
    Registry Authentication
    +
    Network
    +
    Image Does Not Exist

Retrying without fixing the cause will not solve the problem.

---

# 63. OOMKilled

Possible causes:

    Memory Limit Too Low
    +
    Memory Leak
    +
    Traffic Increase
    +
    Unexpected Workload

Approach:

    Check Metrics
        |
        ↓
    Check Logs
        |
        ↓
    Check Memory Limits
        |
        ↓
    Identify Root Cause

---

# 64. Probe Failure

Possible causes:

    Wrong Path
    +
    Wrong Port
    +
    Slow Startup
    +
    Application Failure

Do not simply increase probe thresholds without understanding the issue.

---

# 65. Terraform Error Handling

Terraform can fail because of:

    Invalid Configuration
    +
    Provider Error
    +
    Permission Error
    +
    API Error
    +
    Resource Conflict
    +
    State Problem
    +
    Dependency Problem

---

# 66. Terraform Retry

Some provider/API operations may be retried internally or can be safely retried by rerunning Terraform after a transient failure.

But first:

    Inspect Error
        |
        ↓
    Check State
        |
        ↓
    Run terraform plan
        |
        ↓
    Understand Difference
        |
        ↓
    Apply Again If Safe

---

# 67. Terraform Partial Failure

Example:

    VPC
        |
        ↓
    Subnets
        |
        ↓
    Security Groups
        X
    Failure

Do not automatically destroy everything.

Instead:

    Inspect State
        |
        ↓
    Identify Created Resources
        |
        ↓
    Fix Root Cause
        |
        ↓
    terraform plan
        |
        ↓
    Apply Again

---

# 68. Terraform Lock Errors

If Terraform state is locked:

    Do Not Immediately Force Unlock

First determine:

    Is Another Run Active?

    Did A Previous Run Crash?

    Is The Lock Stale?

Only remove a stale lock after verifying it is safe to do so.

---

# 69. Ansible Error Handling

Ansible supports failure handling through:

    failed_when
    +
    changed_when
    +
    ignore_errors
    +
    block
    +
    rescue
    +
    always

Use these carefully.

---

# 70. Ansible `failed_when`

Use when the default success/failure interpretation does not match your requirement.

Conceptually:

    Task Result
        |
        ↓
    Evaluate Condition
        |
        ↓
    Failure?

---

# 71. Ansible `changed_when`

Controls whether a task should be reported as changed.

Useful when a command executes successfully but should not always be considered a state change.

---

# 72. Ansible `block`

A block groups tasks.

Example:

    block:
        Task A
        Task B
        Task C

This can be combined with:

    rescue
        +
    always

for structured error handling.

---

# 73. Ansible Rescue

Conceptually:

    Main Task
        |
        X
    Failure
        |
        ↓
    Rescue
        |
        ↓
    Recovery Action

---

# 74. Ansible Always

`always` tasks run regardless of whether the block succeeded or failed.

Useful for:

    Cleanup
    +
    Logging
    +
    Validation

---

# 75. Jenkins Error Handling

Jenkins pipelines can use structured stages and post actions.

Conceptually:

    Build
        |
        ↓
    Test
        |
        X
    Failure
        |
        ↓
    Post Actions
        |
        +------ Notify
        +------ Collect Logs
        +------ Cleanup

---

# 76. Jenkins Retry

A Jenkins stage may retry transient operations.

Example:

    Deploy
        |
        X
    Temporary Failure
        |
        ↓
    Retry
        |
        ↓
    Success

Do not blindly retry permanent failures.

---

# 77. GitHub Actions Error Handling

GitHub Actions workflows can control behavior using:

    Conditions
    +
    Timeouts
    +
    Job Dependencies
    +
    Failure Handling
    +
    Status Checks

---

# 78. GitHub Actions Step Failure

Typical pipeline:

    Checkout
        |
        ↓
    Build
        |
        ↓
    Test
        |
        X
    Failure
        |
        ↓
    Stop Pipeline

This prevents deployment of failed artifacts.

---

# 79. Continue-on-Error

Some workflows may intentionally allow a step to fail without failing the entire job.

Use this carefully.

Example:

    Optional Security Report
        |
        X
    Report Generation
        |
        ↓
    Continue

Do not use it to hide critical failures.

---

# 80. Conditions After Failure

A cleanup or diagnostic step may need to run even if an earlier step failed.

Conceptually:

    Main Step
        |
        X
    Failure
        |
        ↓
    Cleanup / Diagnostics

This preserves useful information.

---

# 81. Pipeline Failure Handling

A production pipeline should have:

    Failure Detection
        +
    Logs
        +
    Diagnostics
        +
    Cleanup
        +
    Notification
        +
    Controlled Recovery

---

# 82. CI Failure

Example:

    Build
        X
    Compilation Error

Correct response:

    Stop
        +
    Show Error
        +
    Notify Developer

Do not retry compilation indefinitely.

---

# 83. Test Failure

Example:

    Unit Tests
        X
    Failed

Response:

    Stop Deployment
        +
    Report Failed Tests
        +
    Investigate

Retries may be appropriate only when tests are known to have legitimate transient behavior.

---

# 84. Flaky Tests

A flaky test sometimes passes and sometimes fails without a relevant code change.

Problem:

    Test
        |
        X
    Failure
        |
        ↓
    Retry
        |
        ↓
    Pass

Repeated retries can hide a real quality problem.

Track flaky tests separately.

---

# 85. Security Scan Failure

Example:

    Trivy
        |
        X
    Critical Vulnerability

Do not automatically retry.

Instead:

    Investigate Vulnerability
        +
    Update Dependency / Image
        +
    Rebuild
        +
    Rescan

---

# 86. SonarQube Failure

Example:

    Quality Gate
        |
        X
    Failed

Retrying the same analysis will normally produce the same result.

Correct response:

    Fix Code
        |
        ↓
    Rebuild
        |
        ↓
    Rescan

---

# 87. Veracode Failure

If a security policy blocks the build:

    Policy Failure
        |
        ↓
    Investigate
        |
        ↓
    Fix Vulnerability
        |
        ↓
    Rescan

Do not use retries to bypass security gates.

---

# 88. Trivy Failure

Example:

    Critical CVE
        |
        ↓
    Build Blocked

Correct response:

    Identify Vulnerable Package
        |
        ↓
    Update
        |
        ↓
    Rebuild Image
        |
        ↓
    Scan Again

---

# 89. Artifact Publishing Failure

Example:

    Build
        |
        ↓
    Artifact
        |
        X
    Registry Timeout

Potential approach:

    Retry With Backoff

because registry/network failure may be transient.

---

# 90. Docker Push Failure

Potential transient errors:

    Network Timeout
    +
    Registry Unavailable
    +
    Temporary Connection Failure

Possible response:

    Limited Retry
        +
    Backoff
        +
    Verify Final Push

---

# 91. Docker Build Failure

Example:

    Docker Build
        X
    Invalid Dockerfile

Retrying will not solve the problem.

Correct response:

    Fix Dockerfile
        |
        ↓
    Build Again

---

# 92. ECR Push Failure

Possible causes:

    Authentication
    +
    Network
    +
    Registry
    +
    Image Tag

Classify first.

Do not automatically retry authentication/configuration errors indefinitely.

---

# 93. Kubernetes Deployment Failure

Example:

    Deployment
        |
        X
    Pods Not Ready

Investigate:

    kubectl get pods
    +
    kubectl describe pod
    +
    kubectl logs

Then determine:

    Retry
        +
    Rollback
        +
    Fix Configuration

---

# 94. ArgoCD Sync Failure

Potential causes:

    Invalid Manifest
    +
    Missing Resource
    +
    Permission
    +
    Health Failure
    +
    Cluster Problem

Retry only when the failure is transient.

---

# 95. AWS API Failure

Cloud APIs can return temporary errors.

Possible approach:

    API Error
        |
        ↓
    Classify
        |
        +------ Transient → Backoff + Retry
        |
        +------ Permanent → Fix Configuration
        |
        +------ Permission → Fix IAM

---

# 96. Rate Limiting

Cloud APIs and external services may limit request rates.

Response:

    Detect Rate Limit
        |
        ↓
    Respect Retry-After
        |
        ↓
    Backoff
        |
        ↓
    Jitter
        |
        ↓
    Retry

---

# 97. DNS Failure

DNS failures can be temporary or configuration-related.

Investigate:

    DNS Resolution
        +
    Record
        +
    Resolver
        +
    Network
        +
    TTL

Do not assume every DNS error should simply be retried.

---

# 98. Dependency Failure

Example:

    Service A
        |
        ↓
    Service B
        X

Service A should:

    Timeout
        +
    Limited Retry
        +
    Circuit Breaker
        +
    Controlled Error

where appropriate.

---

# 99. Retry Amplification

Suppose:

    Service A
        |
        ↓
    Service B
        |
        ↓
    Service C

If every layer retries heavily:

    A → B retries
    B → C retries

One user request can generate many downstream requests.

This is retry amplification.

---

# 100. Avoid Retry Amplification

Use:

    Limited Retries
        +
    Clear Ownership
        +
    Timeouts
        +
    Backoff
        +
    Circuit Breakers

Do not configure aggressive retries independently at every layer without considering the whole request path.

---

# 101. Retry Placement

Choose where retry belongs.

Example:

    Client
        |
        ↓
    Service
        |
        ↓
    Database

If both client and service retry aggressively:

    Duplicate Retry Layers

Prefer a deliberate retry strategy.

---

# 102. Retry Storm Prevention

Use:

    Exponential Backoff
    +
    Jitter
    +
    Maximum Attempts
    +
    Maximum Delay
    +
    Circuit Breaker
    +
    Rate Limiting

---

# 103. Error Budget and Retries

Retries consume resources.

Too many retries can:

    Increase Traffic
    +
    Increase Latency
    +
    Increase CPU
    +
    Increase Database Load

Therefore retries should be treated as a reliability mechanism with a cost.

---

# 104. Logging Retry Attempts

Record:

    Attempt Number
    +
    Maximum Attempts
    +
    Error
    +
    Delay
    +
    Request ID

Example:

    Retry attempt 2/3
    reason=connection_timeout
    delay=4s

---

# 105. Metrics for Retries

Useful metrics:

    retry_count
    +
    retry_success_count
    +
    retry_failure_count
    +
    timeout_count
    +
    circuit_open_count

These help determine whether retries are helping or hiding problems.

---

# 106. Retry Success Rate

Example:

    1,000 Retry Attempts
        |
        ↓
    800 Successful
        |
        ↓
    Retry Success Rate = 80%

A very low retry success rate may indicate retries are not useful.

---

# 107. Error Rate vs Retry Rate

Example:

    Error Rate
        ↑
    Retry Rate
        ↑
    Latency
        ↑

This may indicate the system is under stress.

---

# 108. Observability During Failures

Use:

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

Use them together to determine:

    What Failed?
        +
    How Often?
        +
    Where?
        +
    When?
        +
    Why?

---

# 109. Retry Alert

Example:

    retry_count
        |
        ↓
    Suddenly Increasing
        |
        ↓
    Alert
        |
        ↓
    Investigate Dependency

High retry volume can be an early warning signal.

---

# 110. Error Handling During Deployment

Deployment flow:

    Deploy
        |
        ↓
    Health Check
        |
        X
    Failure
        |
        ↓
    Diagnose
        |
        +------ Transient → Retry
        |
        +------ Persistent → Rollback

---

# 111. Deployment Retry

Retry may be appropriate for:

    Temporary API Timeout
    +
    Registry Timeout
    +
    Temporary Network Failure

Retry is usually not the right answer for:

    Invalid Manifest
    +
    Wrong Image Tag
    +
    Missing IAM Permission
    +
    Invalid Configuration

---

# 112. Deployment Rollback

If the new version is unhealthy:

    New Version
        |
        X
    Health Failure
        |
        ↓
    Rollback
        |
        ↓
    Previous Version
        |
        ↓
    Validate

---

# 113. Rollback vs Retry

Retry:

    Same Operation
        |
        ↓
    Temporary Failure

Rollback:

    New Version Is Bad
        |
        ↓
    Return To Known Good Version

Do not confuse them.

---

# 114. Health Check Failure

Example:

    Deployment
        |
        ↓
    Readiness Probe
        X
    Failure

Possible causes:

    Application Not Ready
    +
    Wrong Configuration
    +
    Dependency Failure
    +
    Wrong Port

Investigate before deciding whether to retry or rollback.

---

# 115. Post-Deployment Validation

After recovery:

    Health
        +
    Error Rate
        +
    Latency
        +
    Logs
        +
    Application Functionality

Validate before declaring success.

---

# 116. Cleanup After Failure

Failure handling may require:

    Temporary Files
    +
    Temporary Containers
    +
    Temporary Resources
    +
    Locks
    +
    Workspace Cleanup

Cleanup should not destroy evidence required for debugging.

---

# 117. Preserve Evidence

Before cleanup where necessary:

    Collect Logs
        +
    Capture Error
        +
    Record Version
        +
    Record Environment
        +
    Capture Deployment Details

Then:

    Cleanup

---

# 118. Notification

Important failures should notify the appropriate team.

Possible information:

    Pipeline
    +
    Environment
    +
    Version
    +
    Stage
    +
    Error
    +
    Link To Logs
    +
    Recommended Action

---

# 119. Good Failure Notification

Example:

    Production Deployment Failed

    Version: 1.5.0
    Stage: Kubernetes Deployment
    Reason: Pods Not Ready
    Environment: Production

    Action:
    Deployment stopped and previous version remains active.

This is much better than:

    Deployment failed.

---

# 120. Runbooks

A runbook defines what to do after an alert.

Example:

    Alert:
    High Error Rate

    Step 1:
    Open Grafana

    Step 2:
    Identify Service

    Step 3:
    Check Recent Deployment

    Step 4:
    Search ELK

    Step 5:
    Check Dependencies

    Step 6:
    Rollback If Required

---

# 121. Error Handling Runbook

Generic structure:

    Detect
        |
        ↓
    Classify
        |
        ↓
    Check Impact
        |
        ↓
    Retry If Safe
        |
        ↓
    Recover
        |
        ↓
    Validate
        |
        ↓
    Document

---

# 122. Dead Letter Queue

For asynchronous processing, permanently failed messages can be moved to a dead letter queue.

Flow:

    Message
        |
        ↓
    Consumer
        |
        X
    Processing Failure
        |
        ↓
    Retry
        |
        X
    Maximum Attempts Reached
        |
        ↓
    Dead Letter Queue

---

# 123. Why Dead Letter Queues Matter

They prevent one permanently invalid message from blocking processing indefinitely.

They allow:

    Isolation
        +
    Investigation
        +
    Manual Recovery
        +
    Reprocessing

---

# 124. Poison Message

A poison message repeatedly fails processing.

Example:

    Message
        |
        ↓
    Consumer
        X
    Invalid Data
        |
        ↓
    Retry
        |
        X
    Same Error

Without limits:

    Infinite Retry

Better:

    Limited Attempts
        |
        ↓
    Dead Letter Queue

---

# 125. Retry Queue

A retry mechanism can temporarily hold failed messages.

Example:

    Main Queue
        |
        ↓
    Consumer
        X
    Temporary Failure
        |
        ↓
    Retry Queue
        |
        ↓
    Delayed Retry
        |
        ↓
    Consumer

---

# 126. Error Handling for RabbitMQ

Potential failures:

    Connection Failure
    +
    Consumer Failure
    +
    Message Processing Failure
    +
    Queue Failure

Use:

    Connection Recovery
        +
    Retry
        +
    Acknowledgement Strategy
        +
    Dead Lettering
        +
    Monitoring

---

# 127. Acknowledgement

For message processing:

    Receive Message
        |
        ↓
    Process
        |
        +------ Success → Acknowledge
        |
        +------ Failure → Retry / Reject / Dead Letter

The exact behavior depends on the messaging design.

---

# 128. Duplicate Messages

Message systems may deliver messages more than once.

Therefore consumers should often be designed to handle duplicate processing safely.

Use:

    Idempotency Key
        +
    Unique Constraint
        +
    Processed Message ID

where appropriate.

---

# 129. Exactly Once vs At Least Once

Distributed systems often use:

    At-Least-Once Delivery

Meaning:

    Message May Be Delivered More Than Once

Consumers should therefore consider idempotent processing.

---

# 130. At-Most-Once

At-most-once processing means a message may be lost but is not intentionally retried indefinitely.

Trade-off:

    Less Duplicate Processing
        +
    Higher Risk Of Lost Work

---

# 131. At-Least-Once

At-least-once processing prioritizes delivery.

Trade-off:

    Lower Risk Of Lost Messages
        +
    Possible Duplicate Processing

Therefore:

    Idempotent Consumer

is important.

---

# 132. Error Handling in Microservices

Use:

    Timeout
        +
    Retry
        +
    Backoff
        +
    Circuit Breaker
        +
    Bulkhead
        +
    Fallback
        +
    Observability

Not every service needs every mechanism.

Use based on failure characteristics.

---

# 133. Microservices Failure Example

    Order Service
        |
        ↓
    Payment Service
        |
        ↓
    Payment Database
        X
    Timeout

Possible flow:

    Timeout
        |
        ↓
    Limited Retry
        |
        ↓
    Still Failing
        |
        ↓
    Circuit Breaker
        |
        ↓
    Controlled Error
        |
        ↓
    Alert

---

# 134. Avoid Cascading Failure

A single dependency failure should not bring down the entire platform.

Use:

    Timeouts
    +
    Circuit Breakers
    +
    Bulkheads
    +
    Rate Limits
    +
    Graceful Degradation

---

# 135. Cascading Failure

Example:

    Database Failure
        |
        ↓
    Service A Threads Block
        |
        ↓
    Service A Becomes Slow
        |
        ↓
    Service B Waits
        |
        ↓
    Service B Becomes Slow
        |
        ↓
    API Becomes Slow
        |
        ↓
    Entire Platform Impacted

---

# 136. Preventing Cascading Failure

Use:

    Short Appropriate Timeouts
        +
    Limited Retries
        +
    Circuit Breakers
        +
    Resource Isolation
        +
    Backpressure

---

# 137. Backpressure

Backpressure means controlling incoming work when downstream capacity is limited.

Example:

    Incoming Traffic
        |
        ↓
    Queue
        |
        ↓
    Consumer Capacity
        |
        X
    Too Much Work
        |
        ↓
    Slow / Reject / Queue

This prevents uncontrolled overload.

---

# 138. Graceful Degradation

When a non-critical dependency fails:

    Critical Function
        |
        ↓
    Continue

    Optional Function
        |
        X
    Temporarily Disabled

Example:

    Payment
        |
        ↓
    Must Work

    Recommendations
        |
        X
    Can Be Disabled

---

# 139. Error Handling Security

Never expose internal details unnecessarily.

Bad response:

    Database password:
    xyz123

Better:

    Internal Server Error

while detailed information is stored securely in internal logs.

---

# 140. Error Handling and Secrets

Never include secrets in:

    Error Messages
    +
    Pipeline Logs
    +
    Application Logs
    +
    Notifications

Use:

    Secret Stores
    +
    Masking
    +
    Access Controls

---

# 141. Error Handling and Security Gates

Security failures should generally fail clearly.

Example:

    Trivy
        X
    Critical Vulnerability

Pipeline:

    Stop
        |
        ↓
    Report
        |
        ↓
    Fix
        |
        ↓
    Rescan

Do not hide security failures with retry logic.

---

# 142. Error Handling and Quality Gates

Example:

    SonarQube
        |
        X
    Quality Gate Failed

Response:

    Stop
        |
        ↓
    Fix Code
        |
        ↓
    Rebuild
        |
        ↓
    Analyze Again

---

# 143. Error Handling and Approval Gates

Example:

    Production Deployment
        |
        ↓
    Approval
        |
        X
    Rejected

This is not a transient technical failure.

Do not automatically retry approval indefinitely.

---

# 144. Error Handling and Change Management

Example:

    Production Change
        |
        X
    Change Request Missing

Correct response:

    Stop Deployment
        +
    Report Missing Change
        +
    Complete Required Process

Do not bypass governance through retries.

---

# 145. Error Handling and Separation of Duties

If approval is required:

    Developer
        |
        ↓
    Deployment Request
        |
        ↓
    Authorized Approver
        |
        ↓
    Deployment

A failed approval should not be bypassed through automation.

---

# 146. Error Handling and Disaster Recovery

Recovery process:

    Incident
        |
        ↓
    Detect
        |
        ↓
    Assess
        |
        ↓
    Recover
        |
        ↓
    Validate
        |
        ↓
    Monitor
        |
        ↓
    RCA

---

# 147. Recovery Validation

Never assume recovery succeeded.

Validate:

    Application Health
        +
    Error Rate
        +
    Latency
        +
    Resource Health
        +
    Business Functionality

---

# 148. Root Cause Analysis

After recovery:

    What Failed?
        +
    Why Did It Fail?
        +
    Why Was It Not Detected Earlier?
        +
    Why Did Retry Not Help?
        +
    What Prevents Recurrence?

---

# 149. Five Whys

Example:

    Why did deployment fail?
        ↓
    Pods could not start.

    Why?
        ↓
    Image could not be pulled.

    Why?
        ↓
    Wrong image tag.

    Why?
        ↓
    Pipeline generated incorrect tag.

    Why?
        ↓
    Tag generation logic was not validated.

Root cause:

    Missing validation.

---

# 150. Error Handling Improvement Loop

    Failure
        |
        ↓
    Detection
        |
        ↓
    Recovery
        |
        ↓
    RCA
        |
        ↓
    Improvement
        |
        ↓
    Automation
        |
        ↓
    Test
        |
        ↓
    Prevent Recurrence

---

# 151. Retry Testing

Test:

    Success On First Attempt
        +
    Failure Then Success
        +
    Failure All Attempts
        +
    Timeout
        +
    Rate Limit
        +
    Network Failure
        +
    Permanent Error

---

# 152. Chaos Testing

Controlled failures can validate recovery mechanisms.

Examples:

    Kill Pod
    +
    Block Dependency
    +
    Introduce Latency
    +
    Restart Service

Then verify:

    Retry
        +
    Recovery
        +
    Alerting
        +
    Observability

---

# 153. Error Budget for Automation

Track:

    Pipeline Failures
    +
    Deployment Failures
    +
    Recovery Time
    +
    Retry Rate

The goal is not zero failures.

The goal is:

    Controlled Failure
        +
    Fast Detection
        +
    Fast Recovery
        +
    Continuous Improvement

---

# 154. Common Retry Mistakes

## Mistake 1

Retrying every error.

## Mistake 2

No timeout.

## Mistake 3

No maximum retry count.

## Mistake 4

No backoff.

## Mistake 5

No jitter.

## Mistake 6

Retrying non-idempotent operations.

## Mistake 7

Retrying at every service layer.

## Mistake 8

Ignoring rate limits.

## Mistake 9

Hiding errors with `continue-on-error`.

## Mistake 10

No observability around retries.

---

# 155. Common Error Handling Mistakes

## Mistake 1

Generic error messages.

## Mistake 2

Swallowing exceptions.

## Mistake 3

Logging secrets.

## Mistake 4

Retrying permanent failures.

## Mistake 5

No cleanup.

## Mistake 6

No rollback strategy.

## Mistake 7

No validation after recovery.

## Mistake 8

No alerting.

## Mistake 9

No runbook.

## Mistake 10

No RCA.

---

# 156. Production Retry Checklist

Before enabling retries:

    Is Failure Transient?

    Is Operation Idempotent?

    What Is Maximum Attempts?

    What Is Maximum Delay?

    Is Backoff Used?

    Is Jitter Used?

    Is There A Timeout?

    What Happens After Final Failure?

    Could Retry Overload Dependency?

    Is Retry Observable?

---

# 157. Production Error Handling Checklist

    Clear Error Classification
        +
    Appropriate Timeouts
        +
    Limited Retries
        +
    Exponential Backoff
        +
    Jitter
        +
    Circuit Breaking Where Appropriate
        +
    Structured Logging
        +
    Metrics
        +
    Alerts
        +
    Runbooks
        +
    Rollback
        +
    Validation
        +
    RCA

---

# 158. CI/CD Error Handling Checklist

## Build

    Compilation Failure
    +
    Dependency Failure
    +
    Timeout

## Test

    Failed Tests
    +
    Flaky Tests
    +
    Test Environment Failure

## Security

    SonarQube
    +
    Trivy
    +
    Veracode

## Artifact

    Registry Failure
    +
    Authentication
    +
    Network Timeout

## Deployment

    Kubernetes Failure
    +
    Terraform Failure
    +
    ArgoCD Failure
    +
    Health Check Failure

## Recovery

    Retry
    +
    Rollback
    +
    Notification
    +
    Validation

---

# 159. Production Incident Flow

    ALERT
      |
      ↓
    ASSESS
      |
      ↓
    CLASSIFY ERROR
      |
      +----------------+
      |                |
   TRANSIENT        PERMANENT
      |                |
      ↓                ↓
   RETRY             FIX
      |
      ↓
   SUCCESS?
      |
   +--+--+
   |     |
  YES    NO
   |     |
   ↓     ↓
VALIDATE  RECOVER / ROLLBACK
          |
          ↓
        VALIDATE
          |
          ↓
          RCA

---

# 160. Final Retry and Error Handling Model

    FAILURE
       |
       ↓
    DETECT
       |
       ↓
    CLASSIFY
       |
       +----------------------+
       |                      |
   TRANSIENT              PERMANENT
       |                      |
       ↓                      ↓
   TIMEOUT                 FAIL FAST
       |
       ↓
   BACKOFF
       |
       ↓
    JITTER
       |
       ↓
    RETRY
       |
       ↓
   SUCCESS?
       |
   +---+---+
   |       |
  YES      NO
   |       |
   ↓       ↓
VALIDATE  RETRY LIMIT
           |
           ↓
       CIRCUIT / FAIL
           |
           ↓
        RECOVER
           |
           ↓
        ALERT
           |
           ↓
          RCA

---

# 161. Final DevOps Reliability Pattern

A reliable DevOps platform should combine:

    Idempotency
        +
    Timeouts
        +
    Retries
        +
    Exponential Backoff
        +
    Jitter
        +
    Circuit Breakers
        +
    Bulkheads
        +
    Rate Limiting
        +
    Observability
        +
    Rollback
        +
    Validation

The objective is not:

    Never Fail

The objective is:

    Fail Predictably
        +
    Detect Quickly
        +
    Recover Safely
        +
    Avoid Cascading Failure
        +
    Learn From Failure

---

# 162. Final Concept

Retry and error handling should follow this principle:

    DO NOT
    Retry Everything

Instead:

    Detect
        |
        ↓
    Classify
        |
        ↓
    Is It Temporary?
        |
        +------ No → Fail Clearly
        |
        +------ Yes
                  |
                  ↓
             Is It Safe?
                  |
             +----+----+
             |         |
            No        Yes
             |         |
             ↓         ↓
          Handle    Timeout
          Carefully    |
                      ↓
                   Backoff
                      |
                      ↓
                    Jitter
                      |
                      ↓
                    Retry
                      |
                      ↓
                  Validate
                      |
                 +----+----+
                 |         |
               Success   Failure
                 |         |
                 ↓         ↓
                Done    Retry Limit
                           |
                           ↓
                        Recover
                           |
                           ↓
                         Alert
                           |
                           ↓
                          RCA

The strongest DevOps automation is not automation that assumes
everything will succeed.

It is automation that knows how to:

    Detect Failure
        +
    Classify Failure
        +
    Retry Safely
        +
    Recover From Failure
        +
    Prevent Cascading Failure
        +
    Validate Recovery
        +
    Learn From Incidents