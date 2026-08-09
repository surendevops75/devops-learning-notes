# Deployment Window

A Deployment Window is a predefined and approved period during which a production deployment or infrastructure change is allowed to take place.

In an enterprise DevOps environment, deployment windows help teams coordinate:

    Production Deployment
        +
    Business Availability
        +
    Technical Support
        +
    Monitoring
        +
    Change Management
        +
    Incident Response

A typical enterprise flow is:

    Release Ready
        |
        ↓
    Change Request
        |
        ↓
    Approval
        |
        ↓
    Deployment Window
        |
        ↓
    Pre-Checks
        |
        ↓
    Production Deployment
        |
        ↓
    Validation
        |
        ↓
    Monitoring
        |
        ↓
    Change Closure

---

# Purpose of a Deployment Window

The purpose of a deployment window is to define when a production change can safely be implemented.

A deployment window helps ensure:

    Required Teams Are Available
        +
    Business Impact Is Controlled
        +
    Monitoring Is Available
        +
    Support Is Available
        +
    Rollback Can Be Performed
        +
    Change Is Authorized

The key principle is:

    Approved Change
        +
    Approved Time
        =
    Controlled Deployment

---

# Why Deployment Windows Matter

Production deployments can cause:

    Application Downtime
    Performance Degradation
    User Impact
    Database Changes
    Infrastructure Changes
    Integration Failures

A controlled deployment window allows the organization to prepare for these risks.

---

# Deployment Window Example

Example:

    Application:
    Payment Service

    Version:
    1.4.7

    Environment:
    Production

    Change:
    CHG-10245

    Deployment Window:
    Approved Maintenance Period

During this period:

    DevOps
    QA
    Application Team
    Business Team
    Support Team

may be available to support the release.

---

# Deployment Window Lifecycle

A typical lifecycle is:

    Change Created
        |
        ↓
    Risk Assessment
        |
        ↓
    Approval
        |
        ↓
    Window Scheduled
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
        |
        ↓
    Closure

---

# Deployment Window vs Deployment Time

Deployment Window:

    Allowed Period

Deployment Time:

    Actual Start / End Time

Example:

    Deployment Window:
    10:00 PM - 12:00 AM

Actual deployment:

    Start:
    10:15 PM

    End:
    10:42 PM

The deployment occurred inside the approved window.

---

# Deployment Window Components

A deployment window may define:

    Start Time
    End Time
    Time Zone
    Environment
    Application
    Change ID
    Deployment Owner
    Approvers
    Support Team
    Validation Plan
    Rollback Plan

---

# Time Zone

Enterprise teams may work across multiple time zones.

Example:

    India Team
    US Team
    Europe Team

The deployment window should clearly specify the time zone.

Example:

    Deployment Window:
    22:00 - 23:30 IST

Avoid ambiguous statements such as:

    "Deploy at 10 PM"

because different teams may interpret the time differently.

---

# Deployment Window and Change Request

The deployment window should be associated with the change request.

Example:

    JIRA Change
        |
        ↓
    CHG-10245
        |
        ↓
    Approved Window
        |
        ↓
    Production Deployment

The change request records the planned timing.

---

# Deployment Window and Approval

Typical flow:

    Change Request
        |
        ↓
    Risk Assessment
        |
        ↓
    Approval
        |
        ↓
    Deployment Window
        |
        ↓
    Production

Approval means:

    "The change is authorized."

Deployment window means:

    "The change is authorized during this period."

---

# Deployment Window and Risk

High-risk changes may require:

    Longer Window
    More Monitoring
    More Support
    More Approvers
    Stronger Rollback Planning

Example:

    Small Application Update
        |
        ↓
    Short Window

    Major Database Migration
        |
        ↓
    Larger Window

---

# Deployment Window Based on Business Traffic

Avoid high-impact changes during peak traffic when possible.

Example:

    Peak Traffic
        |
        ↓
    Avoid High-Risk Deployment

Instead:

    Low Traffic
        |
        ↓
    Maintenance Window
        |
        ↓
    Deployment

The exact timing depends on business requirements.

---

# Deployment Window and Business Hours

Some organizations prefer production deployments:

    Outside Business Hours

because fewer users may be affected.

Example:

    Business Hours
        |
        ↓
    Normal Operations

    After Hours
        |
        ↓
    Production Deployment

This is not always required. Many modern organizations deploy continuously during business hours using safe deployment strategies.

---

# Deployment Window and Zero Downtime

A deployment window does not automatically mean downtime.

With:

    Rolling Deployment
    Blue-Green Deployment
    Canary Deployment
    Proper Health Checks

teams can deploy with little or no user-visible downtime.

---

# Deployment Window and Rolling Deployment

Example:

    Deployment Window
        |
        ↓
    Rolling Update
        |
        ↓
    New Pod
        |
        ↓
    Health Check
        |
        ↓
    Replace Old Pod
        |
        ↓
    Repeat
        |
        ↓
    Complete

---

# Deployment Window and Blue-Green

Example:

    Deployment Window
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
        |
        ↓
    Complete

If the new environment fails:

    Switch Traffic
        |
        ↓
    Blue

---

# Deployment Window and Canary

Example:

    Deployment Window
        |
        ↓
    Deploy Canary
        |
        ↓
    Small Traffic
        |
        ↓
    Monitor
        |
        ↓
    Increase Traffic
        |
        ↓
    Full Deployment

---

# Deployment Window and Maintenance Window

A maintenance window is a broader period during which maintenance activities are allowed.

A deployment window can be part of a maintenance window.

Example:

    Maintenance Window
        |
        +-- Database Maintenance
        |
        +-- Infrastructure Change
        |
        +-- Application Deployment
        |
        +-- Security Update

---

# Deployment Window and Change Freeze

A change freeze is a period when normal production changes are restricted.

Example:

    Change Freeze
        |
        ↓
    No Normal Deployments

After freeze:

    Approved Deployment Window
        |
        ↓
    Production

---

# Change Freeze Example

During a major business event:

    Change Freeze
        |
        ↓
    Production Changes Restricted

After the event:

    Normal Change Process
        |
        ↓
    Deployment Window

---

# Emergency Deployment During Freeze

If a critical issue occurs:

    Production Incident
        |
        ↓
    Emergency Change
        |
        ↓
    Emergency Approval
        |
        ↓
    Emergency Deployment
        |
        ↓
    Validation

Emergency deployments should still be documented and controlled.

---

# Deployment Window and Support Teams

Before deployment, ensure required teams are available.

Possible teams:

    DevOps
    Application Team
    QA
    Database Team
    Network Team
    Security Team
    Business Team
    Support Team

---

# Deployment Window and On-Call

For production deployments, the responsible engineers may be on-call during the window.

Example:

    Deployment
        |
        ↓
    Monitoring
        |
        ↓
    Alert
        |
        ↓
    On-Call Engineer
        |
        ↓
    Investigation

---

# Deployment Window and Monitoring

Monitoring should be active during the deployment.

Monitor:

    CPU
    Memory
    Request Rate
    Error Rate
    Latency
    Pod Restarts
    Application Health
    Database Health

Tools:

    Prometheus
    Grafana
    ELK

---

# Deployment Window Pre-Checks

Before the window starts:

    Change Approved
        |
        ↓
    Artifact Verified
        |
        ↓
    Environment Healthy
        |
        ↓
    Monitoring Ready
        |
        ↓
    Rollback Ready
        |
        ↓
    Team Available

---

# Production Health Check

Before deployment:

    kubectl get nodes

    kubectl get pods -A

Check:

    Nodes
    Pods
    Services
    Existing Alerts
    Resource Availability

If the environment is already unhealthy:

    Stop
        |
        ↓
    Investigate
        |
        ↓
    Reschedule If Required

---

# Deployment Window Start

When the approved window opens:

    Confirm Time
        |
        ↓
    Confirm Change
        |
        ↓
    Confirm Approvals
        |
        ↓
    Confirm Environment
        |
        ↓
    Start Deployment

Do not start simply because the team is ready.

The change must also be within the approved window and satisfy organizational controls.

---

# Deployment Window Close

At the end of the window:

    Deployment Complete
        |
        ↓
    Validation Complete
        |
        ↓
    Monitoring Stable
        |
        ↓
    Deployment Closed

If the deployment is still running near the end:

    Assess Status
        |
        ↓
    Continue / Stop / Rollback
        |
        ↓
    Follow Change Procedure

---

# Deployment Exceeds Window

Suppose:

    Window:
    10:00 PM - 11:00 PM

Deployment:

    Started:
    10:30 PM

At:

    11:00 PM

Deployment is still running.

Do not automatically continue without considering:

    Change Policy
    Current Deployment State
    Business Impact
    Rollback Risk
    Support Availability

The correct action depends on organizational procedure.

---

# Deployment Window Extension

Some organizations allow a window to be extended.

Example:

    Original:
    10:00 PM - 11:00 PM

    Deployment:
    Still Running

Possible process:

    Assess
        |
        ↓
    Request Extension
        |
        ↓
    Approval
        |
        ↓
    Continue

Do not assume an extension is automatically authorized.

---

# Deployment Window Cancellation

A deployment may be cancelled when:

    Critical Incident Exists
    Environment Is Unstable
    Dependency Is Unavailable
    Required Approval Missing
    Business Priority Changes
    Security Issue Appears

Flow:

    Scheduled
        |
        ↓
    Problem Identified
        |
        ↓
    Cancel
        |
        ↓
    Reschedule

---

# Deployment Window Rescheduling

Example:

    Approved Deployment
        |
        ↓
    Dependency Failure
        |
        ↓
    Reschedule
        |
        ↓
    New Window
        |
        ↓
    Revalidation

Depending on policy, a new window may require updated approval.

---

# Deployment Window and Dependency

Example:

    Payment Deployment
        |
        ↓
    Database Migration

If database migration is not ready:

    Payment Deployment
        |
        X
    Blocked

Deployment should not proceed simply because the window is open.

---

# Deployment Window and External Dependency

Example:

    Application
        |
        ↓
    External Payment Gateway

Before deployment:

    Verify External Dependency

If unavailable:

    Delay Deployment

---

# Deployment Window and Database

Database changes can require additional planning.

Example:

    Database Migration
        |
        ↓
    Application Deployment
        |
        ↓
    Validation

Ensure:

    Backup
    Migration Plan
    Compatibility
    Recovery Strategy

are ready before the window.

---

# Deployment Window and Infrastructure

Infrastructure changes may include:

    Terraform
    EKS
    VPC
    IAM
    ALB
    RDS
    Security Groups

Example:

    Terraform Plan
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    Deployment Window
        |
        ↓
    terraform apply
        |
        ↓
    Validation

---

# Deployment Window and Terraform

Before production Terraform changes:

    terraform fmt
        |
        ↓
    terraform validate
        |
        ↓
    terraform plan
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    Deployment Window
        |
        ↓
    terraform apply

---

# Deployment Window and Kubernetes

Example:

    Approved Window
        |
        ↓
    Kubernetes Deployment
        |
        ↓
    Rollout
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Monitoring

---

# Deployment Window and Helm

Example:

    Approved Window
        |
        ↓
    helm upgrade
        |
        ↓
    Rollout
        |
        ↓
    Validation

Example command:

    helm upgrade --install payment ./payment-chart \
      --namespace production \
      --set image.tag=1.4.7

---

# Deployment Window and ArgoCD

GitOps flow:

    Change Approved
        |
        ↓
    Deployment Window
        |
        ↓
    Production Manifest
        |
        ↓
    Git
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Validation

---

# Deployment Window and GitHub Actions

Conceptual flow:

    GitHub
        |
        ↓
    CI
        |
        ↓
    QA
        |
        ↓
    UAT
        |
        ↓
    Approval
        |
        ↓
    Production Environment
        |
        ↓
    Deployment Window
        |
        ↓
    Deployment
        |
        ↓
    Validation

---

# Deployment Window and Scheduled Workflows

A CI/CD system can trigger workflows at predefined times.

Conceptual example:

    Scheduled Time
        |
        ↓
    Pipeline
        |
        ↓
    Pre-Checks
        |
        ↓
    Deployment
        |
        ↓
    Validation

However, scheduling alone does not replace:

    Approval
    Change Management
    Risk Assessment

---

# Deployment Window and Automation

Automation can handle:

    Pre-Checks
    Deployment
    Health Checks
    Smoke Tests
    Rollback
    Notifications
    Evidence Collection

Human decisions can remain for:

    Risk
    Approval
    Business Impact
    Emergency Decisions

---

# Automated Deployment Window Checks

Before deployment:

    Is Change Approved?
        |
        ↓
    Is Current Time Valid?
        |
        ↓
    Is Environment Healthy?
        |
        ↓
    Is Artifact Correct?
        |
        ↓
    Is Rollback Ready?
        |
        ↓
    Deploy

---

# Deployment Window Gate

Conceptual pipeline:

    Build
        |
        ↓
    Test
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
    Deployment Window Gate
        |
        +------ Not Allowed → Stop
        |
        +------ Allowed
                 |
                 ↓
             Production

---

# Deployment Window Validation

Before starting:

    Change ID Correct
    Version Correct
    Environment Correct
    Window Correct
    Approvals Complete
    Dependencies Ready

---

# Deployment Window Checklist

## Before Window

    Change Approved
    Release Approved
    Artifact Verified
    Version Verified
    Configuration Verified
    Secrets Verified
    Database Ready
    Dependencies Ready
    Monitoring Ready
    Rollback Ready
    Support Teams Available

## During Window

    Pre-Checks
    Deployment
    Rollout Monitoring
    Health Checks
    Smoke Tests
    Metrics
    Logs
    Alerts

## After Window

    Application Validation
    Business Validation
    Monitoring
    Evidence
    Communication
    Change Update
    Closure

---

# Deployment Window Communication

Before deployment:

    Deployment Planned

During deployment:

    Deployment Started

After deployment:

    Deployment Successful

If failure occurs:

    Deployment Failed
        |
        ↓
    Rollback / Recovery

Communication should include:

    Change ID
    Application
    Version
    Environment
    Status
    Impact

---

# Deployment Start Notification

Example:

    Production deployment for Payment Service
    version 1.4.7 has started under approved
    change CHG-10245.

---

# Deployment Success Notification

Example:

    Production deployment for Payment Service
    version 1.4.7 completed successfully.

    Health checks passed.
    Smoke tests passed.
    No critical alerts detected.

---

# Deployment Failure Notification

Example:

    Production deployment for Payment Service
    version 1.4.7 encountered an issue.

    Rollback has been initiated.
    Validation is in progress.

---

# Deployment Window and Business Communication

Business stakeholders may need to know:

    Deployment Start
    Expected Impact
    Deployment Result
    Service Recovery

Example:

    Payment Service deployment completed
    successfully with no user-visible downtime.

---

# Deployment Window and User Impact

Before deployment, estimate:

    Expected Downtime
    Potential Degradation
    Affected Users
    Affected Services

Example:

    Expected Impact:
    No downtime

or:

    Expected Impact:
    Brief degradation during maintenance

---

# Deployment Window and Maintenance Notification

If users may be affected:

    Maintenance Notification
        |
        ↓
    Deployment
        |
        ↓
    Validation
        |
        ↓
    Service Available

The exact communication process depends on business policy.

---

# Deployment Window and Zero Downtime

For zero-downtime deployments:

    Multiple Replicas
        |
        ↓
    Readiness Probes
        |
        ↓
    Rolling Update
        |
        ↓
    Load Balancer
        |
        ↓
    Healthy Traffic

---

# Deployment Window and Readiness

A Pod should receive production traffic only after readiness succeeds.

Flow:

    New Pod
        |
        ↓
    Startup
        |
        ↓
    Readiness Check
        |
        +------ Fail → No Traffic
        |
        +------ Pass
                 |
                 ↓
             Traffic

---

# Deployment Window and Liveness

Liveness probes help identify containers that are no longer functioning correctly.

Flow:

    Container
        |
        ↓
    Liveness Check
        |
        +------ Healthy
        |
        +------ Failed
                 |
                 ↓
              Restart

Readiness and liveness solve different problems.

---

# Deployment Window and Health Checks

After deployment:

    Deployment
        |
        ↓
    Pod Health
        |
        ↓
    Service Health
        |
        ↓
    ALB Target Health
        |
        ↓
    Application Health
        |
        ↓
    Business Workflow

---

# Deployment Window and Smoke Testing

Smoke tests should verify critical functionality.

Example:

    HTTPS
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
    Business Workflow

---

# Deployment Window and Monitoring

Monitor during:

    Pre-Deployment
    Deployment
    Post-Deployment

Compare:

    Before
        |
        ↓
    During
        |
        ↓
    After

Metrics:

    Error Rate
    Latency
    Traffic
    CPU
    Memory
    Pod Restarts

---

# Production Baseline

Before deployment, capture the normal state.

Example:

    Error Rate:
    Normal

    Latency:
    Normal

    CPU:
    Normal

After deployment:

    Error Rate:
    Increased

    Latency:
    Increased

This may indicate a deployment problem.

---

# Deployment Window and Grafana

Grafana dashboards can be used to monitor:

    Request Rate
    Error Rate
    Latency
    CPU
    Memory
    Pod Health

Example:

    Deployment
        |
        ↓
    Grafana
        |
        ↓
    Compare Metrics
        |
        ↓
    Healthy / Degraded

---

# Deployment Window and ELK

ELK can be used to inspect:

    Application Errors
    Exceptions
    HTTP Errors
    Database Errors
    Integration Failures

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
    Investigation

---

# Deployment Window and Rollback

If a serious issue appears:

    Deployment
        |
        ↓
    Failure
        |
        ↓
    Stop Rollout
        |
        ↓
    Rollback
        |
        ↓
    Validation
        |
        ↓
    Recovery

---

# Kubernetes Rollback

Example:

    kubectl rollout undo deployment/payment -n production

Then:

    kubectl rollout status deployment/payment -n production

Validate:

    kubectl get pods -n production

---

# Helm Rollback

Check history:

    helm history payment -n production

Rollback:

    helm rollback payment <revision> -n production

Then validate:

    kubectl get pods -n production

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
    Validation

---

# Deployment Window and Incident

If a deployment creates an incident:

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
    Recovery
        |
        ↓
    Validation

The deployment change should be linked to the incident where appropriate.

---

# Deployment Window and Change Failure

If deployment fails:

    Change
        |
        ↓
    Deployment
        |
        X
    Failure
        |
        ↓
    Rollback
        |
        ↓
    Recovery
        |
        ↓
    Change Updated

Document:

    Failure
    Cause
    Impact
    Rollback
    Recovery

---

# Deployment Window and Change Closure

A change can be closed after:

    Deployment Complete
        |
        ↓
    Validation Complete
        |
        ↓
    Monitoring Stable
        |
        ↓
    Business Validation
        |
        ↓
    Evidence Recorded
        |
        ↓
    Change Closed

---

# Deployment Window Evidence

Record:

    Change ID
    Start Time
    End Time
    Application
    Version
    Git Commit
    Artifact
    Deployment Result
    Health Check
    Smoke Test
    Monitoring Result
    Rollback Result
    Approvals

---

# Deployment Window Auditability

A complete record should answer:

    Who Deployed?

    What Was Deployed?

    When Was It Deployed?

    Why Was It Deployed?

    Who Approved It?

    Was It Successful?

    Was Rollback Required?

---

# Deployment Window and DORA Metrics

Deployment windows should not become a reason to slow delivery unnecessarily.

Track:

    Deployment Frequency
    Lead Time
    Change Failure Rate
    Time To Restore

The goal is:

    Fast
        +
    Reliable
        +
    Controlled

---

# Deployment Window Optimization

Improve deployment windows by:

    Automating Pre-Checks
    Automating Deployment
    Automating Health Checks
    Automating Smoke Tests
    Automating Rollback
    Improving Monitoring
    Reducing Deployment Duration
    Using Canary
    Using Rolling Deployment
    Using Blue-Green

---

# Shorter Deployment Windows

A mature pipeline can reduce the amount of time required.

Example:

    Manual Process

    Pre-Checks
        |
        ↓
    Manual Deployment
        |
        ↓
    Manual Validation

versus:

    Automated Pre-Checks
        |
        ↓
    Automated Deployment
        |
        ↓
    Automated Health Checks
        |
        ↓
    Automated Smoke Tests

Automation reduces deployment duration and human error.

---

# Deployment Window Anti-Pattern

## Deploying Outside Approved Window

Bad:

    Approved:
    10 PM - 11 PM

Actual:

    9 PM

This is an unauthorized timing change unless explicitly approved.

---

# Deployment Window Anti-Pattern

## Starting Without Approval

Bad:

    Deployment Window Open
        |
        ↓
    No Approval
        |
        ↓
    Deploy

Better:

    Approval
        |
        ↓
    Window
        |
        ↓
    Deploy

---

# Deployment Window Anti-Pattern

## No Monitoring

Bad:

    Deploy
        |
        ↓
    Leave

Better:

    Deploy
        |
        ↓
    Monitor
        |
        ↓
    Validate

---

# Deployment Window Anti-Pattern

## Ignoring Existing Incidents

Bad:

    Existing Production Incident
        |
        ↓
    Deploy Unrelated Change

Better:

    Existing Incident
        |
        ↓
    Assess
        |
        ↓
    Delay / Reschedule If Appropriate

---

# Deployment Window Anti-Pattern

## Deploying When Dependencies Are Unavailable

Bad:

    External Dependency Down
        |
        ↓
    Deploy

Better:

    Dependency Check
        |
        ↓
    Unavailable
        |
        ↓
    Delay

---

# Deployment Window Anti-Pattern

## No Rollback

Bad:

    Deployment
        |
        ↓
    Failure
        |
        ↓
    Unknown

Better:

    Deployment
        |
        ↓
    Failure
        |
        ↓
    Predefined Rollback
        |
        ↓
    Recovery

---

# Deployment Window Anti-Pattern

## Extending Window Without Approval

Bad:

    Window Ends
        |
        ↓
    Continue Automatically

Better:

    Window Ends
        |
        ↓
    Assess
        |
        ↓
    Follow Extension Procedure
        |
        ↓
    Continue / Stop

---

# Deployment Window Best Practices

- Define clear start and end times
- Specify the time zone
- Link the window to the change request
- Obtain required approvals
- Verify the environment before deployment
- Verify the exact artifact
- Confirm support team availability
- Confirm monitoring availability
- Confirm rollback readiness
- Validate dependencies
- Use safe deployment strategies
- Monitor throughout the window
- Perform smoke tests
- Validate critical business workflows
- Record deployment evidence
- Communicate deployment status
- Follow extension procedures
- Follow emergency procedures
- Close the change after validation

---

# Deployment Window Checklist

## Before Window

    Change Approved
    Release Approved
    Version Verified
    Artifact Verified
    Configuration Verified
    Database Ready
    Dependencies Ready
    Monitoring Ready
    Rollback Ready
    Support Available
    Deployment Window Confirmed

## During Window

    Pre-Checks
    Deployment
    Rollout Monitoring
    Health Checks
    Smoke Tests
    Metrics
    Logs
    Alerts

## After Window

    Business Validation
    Monitoring Stable
    Evidence Recorded
    Communication Sent
    JIRA Updated
    Change Closed

---

# Real-World Example

Application:

    Payment Service

Version:

    1.4.7

Change:

    CHG-10245

Deployment Window:

    Approved Production Maintenance Period

Flow:

    Change Approved
        |
        ↓
    Window Opens
        |
        ↓
    Cluster Health Check
        |
        ↓
    Artifact Verification
        |
        ↓
    Production Deployment
        |
        ↓
    Rollout Validation
        |
        ↓
    Smoke Test
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
    Deployment Complete

---

# Real-World Example: Deployment Delayed

Situation:

    Deployment Window:
    Approved

    Environment:
    Production

But:

    Database Dependency
    Not Ready

Action:

    Stop
        |
        ↓
    Notify Stakeholders
        |
        ↓
    Do Not Deploy
        |
        ↓
    Reschedule
        |
        ↓
    New Window
        |
        ↓
    Revalidation

---

# Real-World Example: Deployment Failure

Situation:

    Deployment Started
        |
        ↓
    New Pods Deployed
        |
        ↓
    Health Check Failed

Action:

    Stop Rollout
        |
        ↓
    Investigate
        |
        ↓
    Rollback
        |
        ↓
    Validate
        |
        ↓
    Notify
        |
        ↓
    Update Change

---

# Real-World Example: Deployment Exceeds Window

Situation:

    Window:
    10:00 PM - 11:00 PM

    Deployment:
    Still Running At 11:00 PM

Action:

    Assess Deployment State
        |
        ↓
    Assess User Impact
        |
        ↓
    Assess Rollback Risk
        |
        ↓
    Follow Change Extension Procedure
        |
        ↓
    Continue / Stop / Rollback

Do not blindly continue or terminate the deployment.

---

# Real-World Example: Emergency Deployment

Situation:

    Critical Production Vulnerability

Normal:

    Change Freeze

Emergency:

    Critical Issue
        |
        ↓
    Emergency Change
        |
        ↓
    Emergency Approval
        |
        ↓
    Emergency Deployment Window
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

# Deployment Window Interview Questions

## Basic

1. What is a deployment window?

2. Why are deployment windows used?

3. What is the difference between deployment time and deployment window?

4. What is a maintenance window?

5. What is a change freeze?

6. Why should a deployment window specify a time zone?

7. What is a deployment window extension?

8. What is a deployment window cancellation?

9. What is a deployment window reschedule?

10. Why is monitoring required during deployment?

---

# Deployment Window Interview Questions

## Intermediate

11. How do you plan a production deployment window?

12. What do you verify before starting a deployment?

13. What happens if a deployment exceeds the approved window?

14. How do you handle a dependency that is unavailable during the window?

15. How do you handle an unstable production environment before deployment?

16. How do deployment windows relate to change requests?

17. How do deployment windows relate to approvals?

18. How do you perform a zero-downtime deployment during a maintenance window?

19. How do you monitor a deployment during the window?

20. How do you handle a deployment failure?

---

# Deployment Window Interview Questions

## Advanced

21. How would you design an enterprise deployment-window process?

22. How would you automate deployment-window validation?

23. How would you integrate deployment windows with GitHub Actions?

24. How would you integrate deployment windows with JIRA?

25. How would you implement deployment-window controls for ArgoCD?

26. How would you handle deployments across multiple time zones?

27. How would you handle a deployment that exceeds the approved window?

28. How would you handle an emergency deployment during a change freeze?

29. How would you reduce deployment-window duration?

30. How would you design a zero-downtime production deployment process?

31. How would you prevent unauthorized deployments outside approved windows?

32. How would you design deployment windows for high-risk database changes?

---

# Scenario-Based Interview Question

## Deployment Window Is Open but Environment Is Unhealthy

Situation:

    Window = Open

But:

    Existing Production Incident
        |
        ↓
    Pods Unhealthy
        |
        ↓
    High Error Rate

Action:

    Do Not Automatically Deploy

Instead:

    Assess Incident
        |
        ↓
    Stabilize Environment
        |
        ↓
    Reassess Change
        |
        ↓
    Deploy / Reschedule

---

# Scenario-Based Interview Question

## Deployment Is Still Running When Window Ends

Action:

    Check Deployment State
        |
        ↓
    Check User Impact
        |
        ↓
    Check Rollback Risk
        |
        ↓
    Follow Extension Procedure
        |
        ↓
    Continue / Stop / Rollback

The correct decision depends on organizational policy and the state of the deployment.

---

# Scenario-Based Interview Question

## Dependency Becomes Unavailable

Example:

    Payment Deployment
        |
        ↓
    External Payment Gateway
        |
        X
    Unavailable

Action:

    Stop
        |
        ↓
    Assess
        |
        ↓
    Delay / Reschedule
        |
        ↓
    Notify Stakeholders

---

# Scenario-Based Interview Question

## Critical Security Issue During Change Freeze

Situation:

    Change Freeze
        |
        ↓
    Critical Vulnerability

Action:

    Emergency Change
        |
        ↓
    Risk Assessment
        |
        ↓
    Emergency Approval
        |
        ↓
    Controlled Deployment
        |
        ↓
    Validation
        |
        ↓
    Monitoring

---

# Scenario-Based Interview Question

## Deployment Starts Outside Approved Window

Situation:

    Approved Window:
    10 PM

    Actual Start:
    9 PM

Treat this as a control violation.

Action:

    Stop If Safe
        |
        ↓
    Assess Impact
        |
        ↓
    Notify Stakeholders
        |
        ↓
    Investigate
        |
        ↓
    Document
        |
        ↓
    Improve Controls

---

# Scenario-Based Interview Question

## Deployment Window Is Too Short

Example:

    Planned Window:
    30 Minutes

Historical deployment:

    Average:
    45 Minutes

Action:

    Analyze Deployment Duration
        |
        ↓
    Automate Steps
        |
        ↓
    Reduce Manual Work
        |
        ↓
    Improve Rollout
        |
        ↓
    Increase Window If Required

Do not knowingly schedule an unsafe window.

---

# Scenario-Based Interview Question

## Multiple Applications Need Deployment

Example:

    Payment
    Order
    Inventory

Determine:

    Dependencies
    Deployment Order
    Risk
    Validation

Possible flow:

    Payment
        |
        ↓
    Validate
        |
        ↓
    Order
        |
        ↓
    Validate
        |
        ↓
    Inventory
        |
        ↓
    Validate

Or use independent parallel deployments where dependencies allow it.

---

# Scenario-Based Interview Question

## Deployment Requires Database Migration

Plan:

    Pre-Checks
        |
        ↓
    Database Readiness
        |
        ↓
    Migration
        |
        ↓
    Application Deployment
        |
        ↓
    Validation
        |
        ↓
    Monitoring

Use backward-compatible migrations whenever possible.

---

# Scenario-Based Interview Question

## Deployment Causes 503 During Window

Check:

    ALB
    Ingress
    Service
    Endpoints
    Pods
    Readiness
    Application Logs

Then:

    Mitigate
        |
        ↓
    Rollback If Required
        |
        ↓
    Validate
        |
        ↓
    Communicate

---

# Scenario-Based Interview Question

## Deployment Is Successful but Business Workflow Fails

Example:

    Pods Healthy
        |
        ↓
    Health Check Pass
        |
        ↓
    Smoke Test Pass
        |
        ↓
    Payment Workflow Fails

Action:

    Treat As Production Failure
        |
        ↓
    Assess Impact
        |
        ↓
    Rollback / Fix
        |
        ↓
    Validate Business Workflow

Infrastructure health alone is not sufficient.

---

# Final Deployment Window Mental Model

Remember:

    CHANGE
      |
      ↓
    APPROVAL
      |
      ↓
    WINDOW
      |
      ↓
    PRE-CHECK
      |
      ↓
    DEPLOY
      |
      ↓
    VALIDATE
      |
      ↓
    MONITOR
      |
      ↓
    CLOSE

If something goes wrong:

    DEPLOY
      |
      ↓
    FAILURE
      |
      ↓
    STOP
      |
      ↓
    ROLLBACK
      |
      ↓
    RECOVER
      |
      ↓
    VALIDATE
      |
      ↓
    DOCUMENT

---

# Final Concept

A deployment window is not simply a time on the calendar.

It is a controlled period that combines:

    Authorization
        +
    Timing
        +
    Team Availability
        +
    Monitoring
        +
    Deployment Planning
        +
    Rollback Readiness
        +
    Business Coordination

The ideal enterprise process is:

    Tested Release
        |
        ↓
    Approved Change
        |
        ↓
    Approved Deployment Window
        |
        ↓
    Automated Pre-Checks
        |
        ↓
    Safe Deployment
        |
        ↓
    Automated Validation
        |
        ↓
    Monitoring
        |
        ↓
    Business Validation
        |
        ↓
    Change Closure

The key principle is:

    Deploy At The Right Time
        +
    With The Right Approval
        +
    With The Right People
        +
    With The Right Controls
        +
    With A Safe Recovery Plan