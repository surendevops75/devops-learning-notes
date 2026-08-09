# JIRA Change Request

A JIRA Change Request is a formal record used to plan, track, approve, implement, and document a change to an application, infrastructure, configuration, database, or production environment.

In enterprise DevOps environments, a change request provides:

    Change Visibility
        +
    Approval
        +
    Risk Assessment
        +
    Deployment Planning
        +
    Rollback Planning
        +
    Auditability

A typical production release flow is:

    Development
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
    Business Sign-Off
        |
        ↓
    JIRA Change Request
        |
        ↓
    Approval
        |
        ↓
    Production Deployment
        |
        ↓
    Validation
        |
        ↓
    Change Closure

---

# Purpose of a JIRA Change Request

The purpose of a change request is to ensure that production changes are controlled and traceable.

A change request answers:

    What is changing?

    Why is it changing?

    When will it change?

    Who will implement it?

    What is the risk?

    How will it be validated?

    What happens if it fails?

---

# Why Change Management Is Important

Production changes can affect:

    Application Availability
    Infrastructure
    Database
    Security
    Network
    Users
    Business Operations
    Data

Therefore:

    Production Change
        |
        ↓
    Assessment
        |
        ↓
    Approval
        |
        ↓
    Implementation
        |
        ↓
    Validation
        |
        ↓
    Closure

---

# JIRA Change Request Lifecycle

A typical lifecycle is:

    Create Change
        |
        ↓
    Review
        |
        ↓
    Risk Assessment
        |
        ↓
    Approval
        |
        ↓
    Scheduled
        |
        ↓
    Implementation
        |
        ↓
    Validation
        |
        ↓
    Successful
        |
        ↓
    Closure

If implementation fails:

    Implementation
        |
        ↓
    Failure
        |
        ↓
    Rollback
        |
        ↓
    Recovery
        |
        ↓
    Validation
        |
        ↓
    Change Review

---

# JIRA Change Request vs Development Ticket

A development ticket generally tracks:

    Feature
    Bug
    Task
    Story

A change request tracks:

    Controlled Environment Change
    Production Deployment
    Infrastructure Change
    Database Change
    Configuration Change

Example:

    JIRA Story
        |
        ↓
    Implement Payment Feature

Then:

    JIRA Change
        |
        ↓
    Deploy Payment Version 1.4.7 To Production

---

# JIRA Change Request Example

Example:

    Change ID:
    CHG-10245

    Application:
    Payment Service

    Version:
    1.4.7

    Environment:
    Production

    Change Type:
    Application Deployment

    Planned Date:
    Approved Deployment Window

    Risk:
    Medium

    Rollback:
    Kubernetes Rollback

---

# Change Request Information

A change request commonly contains:

    Change Title
    Change Description
    Business Reason
    Application
    Environment
    Version
    Change Type
    Implementation Plan
    Risk
    Impact
    Dependencies
    Validation Plan
    Rollback Plan
    Deployment Window
    Owner
    Approvers
    Related Tickets

---

# Change Title

The title should clearly explain the change.

Good:

    Deploy Payment Service v1.4.7 To Production

Better than:

    Payment Update

The title should be:

    Clear
    Specific
    Traceable

---

# Change Description

The description explains what will change.

Example:

    Deploy Payment Service version 1.4.7
    to the production EKS environment.

    The release contains:
        - Payment API improvements
        - Bug fixes
        - Configuration updates

The description should provide enough information for reviewers to understand the change.

---

# Business Reason

The business reason explains why the change is required.

Examples:

    New Business Feature
    Critical Bug Fix
    Security Patch
    Performance Improvement
    Infrastructure Upgrade
    Compliance Requirement

Example:

    Business Reason:

    Deploy the approved payment service release
    to provide the new payment validation capability
    and resolve the approved production defect.

---

# Change Type

Common change types include:

    Standard Change
    Normal Change
    Emergency Change

Organizations may define additional categories.

---

# Standard Change

A standard change is:

    Low Risk
    Repeatable
    Well Understood
    Pre-Approved

Example:

    Regularly Scheduled Application Deployment

A standard change follows a documented procedure.

---

# Normal Change

A normal change generally requires:

    Review
    Risk Assessment
    Approval
    Scheduled Implementation

Example:

    Major Application Release
    Database Schema Change
    Infrastructure Modification

---

# Emergency Change

An emergency change is required to address a critical situation.

Examples:

    Critical Security Vulnerability
    Major Production Outage
    Severe Data Issue
    Critical Business Failure

Flow:

    Incident
        |
        ↓
    Emergency Change
        |
        ↓
    Approval
        |
        ↓
    Implementation
        |
        ↓
    Validation

Emergency changes should still be documented and audited.

---

# Change Risk

Risk assessment identifies the potential impact of the change.

Possible risk levels:

    Low
    Medium
    High
    Critical

The exact classification depends on organizational standards.

---

# Low-Risk Change

Example:

    Small Application Configuration Update

Potential impact:

    Limited

Rollback:

    Simple

---

# Medium-Risk Change

Example:

    Application Version Upgrade

Potential impact:

    Some Application Components

Rollback:

    Available

---

# High-Risk Change

Examples:

    Database Migration
    Major Infrastructure Change
    Kubernetes Upgrade
    Large Application Release

Potential impact:

    Multiple Services
    Multiple Users
    Business Operations

Requires stronger planning and validation.

---

# Critical Change

A critical change may affect:

    Entire Production Environment
    Core Business Operations
    Data
    Security
    Availability

Such changes require strong controls and detailed planning.

---

# Change Impact

Impact describes what could be affected.

Examples:

    Application
    Database
    Infrastructure
    Network
    Users
    External Systems
    Business Operations

Example:

    Impact:

    Payment Service users may experience
    brief service degradation during deployment.

---

# Change Dependencies

Identify dependencies before implementation.

Examples:

    Database Migration
    Kubernetes
    ECR Image
    External API
    Network
    IAM
    Configuration
    Other Microservices

Example:

    Payment Service
        |
        ↓
    Database
        |
        ↓
    External Payment Gateway

---

# Change Implementation Plan

The implementation plan describes exactly how the change will be executed.

Example:

    1. Verify production health.

    2. Confirm approved artifact.

    3. Verify production configuration.

    4. Verify database readiness.

    5. Deploy application.

    6. Monitor rollout.

    7. Run health checks.

    8. Run smoke tests.

    9. Monitor metrics.

    10. Validate business workflow.

    11. Record deployment result.

---

# Pre-Implementation Checklist

Before starting:

    Change Approved
        |
        ↓
    Artifact Verified
        |
        ↓
    Version Verified
        |
        ↓
    Configuration Verified
        |
        ↓
    Secrets Verified
        |
        ↓
    Database Ready
        |
        ↓
    Monitoring Ready
        |
        ↓
    Rollback Ready
        |
        ↓
    Deployment Window Open

---

# Production Health Pre-Check

Before deployment:

    kubectl get nodes

    kubectl get pods -A

Check:

    Node Health
    Pod Health
    Resource Availability
    Existing Failures
    Active Incidents

Do not start a major deployment while the environment is already unstable unless the change is specifically intended to mitigate the incident.

---

# Artifact Verification

Verify the exact artifact approved for production.

Example:

    Application:
    payment

    Version:
    1.4.7

    Image:
    payment:1.4.7

    Git Commit:
    abc123

The production artifact should match the artifact validated in previous environments.

---

# Build Once and Promote

Preferred flow:

    Build
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
    Production

Example:

    payment:1.4.7

The same artifact should be promoted rather than rebuilding independently for production.

---

# Change Validation Plan

The validation plan defines how success will be confirmed.

Example:

    Deployment Successful
        |
        ↓
    Pods Healthy
        |
        ↓
    Health Checks Pass
        |
        ↓
    Smoke Test Pass
        |
        ↓
    Error Rate Normal
        |
        ↓
    Latency Normal
        |
        ↓
    Critical Business Workflow Pass

---

# Rollback Plan

Every significant production change should have a rollback plan.

Example:

    New Version
        |
        ↓
    Validation
        |
        ↓
    Failure
        |
        ↓
    Rollback
        |
        ↓
    Previous Version
        |
        ↓
    Validation

Kubernetes example:

    kubectl rollout undo deployment/payment -n production

---

# Rollback Criteria

Define when rollback should happen.

Examples:

    Critical API Failure
    HTTP 5xx Increase
    Severe Latency
    Application Crash
    Data Integrity Risk
    Business Workflow Failure
    Security Issue

Example:

    If HTTP 5xx increases significantly
    and the issue cannot be mitigated quickly,
    stop the rollout and initiate rollback.

---

# Rollback Validation

After rollback:

    kubectl rollout status deployment/payment -n production

Then validate:

    Pods
    Health Checks
    Error Rate
    Latency
    Logs
    Critical Business Workflow

Rollback is not complete until the recovered version is validated.

---

# Database Rollback

Database rollback requires special care.

Application rollback:

    New Application
        |
        ↓
    Previous Application

Database rollback:

    Database
        |
        ↓
    Potential Data Changes

Therefore:

    Database Rollback
        =
    More Complex

Use backward-compatible migration strategies whenever possible.

---

# Implementation Window

A change request should define when implementation will occur.

Example:

    Start:
    Approved Window

    End:
    Approved Window

The exact time depends on:

    Business Traffic
    Maintenance Policy
    Risk
    Dependencies
    Team Availability

---

# Why Deployment Windows Matter

A controlled deployment window provides:

    Required Team Availability
    Business Awareness
    Monitoring Coverage
    Support Availability
    Reduced Business Impact

---

# Change Owner

The change owner is responsible for coordinating the change.

Responsibilities may include:

    Preparing Change
    Coordinating Approvals
    Coordinating Deployment
    Monitoring Progress
    Communicating Status
    Recording Results
    Closing Change

---

# Approvers

Approvers depend on organizational policy.

Possible approvers:

    DevOps Lead
    QA Lead
    Product Owner
    Release Manager
    Change Manager
    Business Owner

---

# Separation of Duties

Enterprise environments may separate responsibilities.

Example:

    Developer
        |
        ↓
    Creates Code

    QA
        |
        ↓
    Validates Release

    DevOps
        |
        ↓
    Deploys

    Change Manager
        |
        ↓
    Approves Change

    Business Owner
        |
        ↓
    Accepts Business Impact

This reduces operational risk.

---

# Related JIRA Tickets

A change request can reference related tickets.

Examples:

    Story
    Bug
    Task
    Incident
    Problem
    Release

Example:

    Story:
    PROJ-245

    Bug:
    PROJ-251

    Change:
    CHG-10245

    Incident:
    INC-7781

This provides traceability.

---

# Change Traceability

A strong enterprise process provides:

    Requirement
        |
        ↓
    Development Ticket
        |
        ↓
    Code Commit
        |
        ↓
    CI Build
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
    Change Request
        |
        ↓
    Production
        |
        ↓
    Validation

---

# JIRA Change and Git

The change request should reference:

    Git Commit
    Pull Request
    Release
    Tag
    Artifact

Example:

    JIRA Change
        |
        ↓
    Release 1.4.7
        |
        ↓
    Git Tag v1.4.7
        |
        ↓
    Image payment:1.4.7

---

# JIRA Change and CI/CD

A CI/CD pipeline can reference the change request.

Example:

    JIRA Change
        |
        ↓
    Approved
        |
        ↓
    GitHub Actions
        |
        ↓
    Production Deployment

The exact integration depends on the organization's tooling.

---

# JIRA Change and GitHub Actions

Conceptual flow:

    JIRA
      |
      ↓
    Approved Change
      |
      ↓
    GitHub Actions
      |
      ↓
    Production Deployment
      |
      ↓
    Validation
      |
      ↓
    JIRA Updated

The pipeline should use appropriate authentication and authorization.

---

# JIRA Change and ArgoCD

GitOps flow:

    JIRA Change
        |
        ↓
    Approval
        |
        ↓
    Production Git Change
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
    Production

JIRA provides change governance while Git remains the source of truth for deployment configuration.

---

# Production Change Using ArgoCD

Example:

    UAT Approved
        |
        ↓
    JIRA Change Created
        |
        ↓
    Change Approved
        |
        ↓
    Production Manifest Updated
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
    ArgoCD Sync
        |
        ↓
    Production
        |
        ↓
    Validation

---

# Change Status

A change may move through statuses such as:

    Draft
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    Scheduled
        |
        ↓
    In Progress
        |
        ↓
    Validation
        |
        ↓
    Completed
        |
        ↓
    Closed

Organizations may use different workflow names.

---

# Draft

During Draft:

    Change Details
    Risk
    Impact
    Plan
    Rollback
    Validation

are prepared.

---

# Review

During Review:

    Technical Team
        |
        ↓
    Review Change
        |
        ↓
    Validate Risk
        |
        ↓
    Validate Implementation
        |
        ↓
    Validate Rollback

---

# Approval

After review:

    Change
        |
        ↓
    Approver
        |
        +------ Reject ------→ Update
        |
        +------ Approve
                |
                ↓
              Schedule

---

# Scheduled

The change is ready for the approved implementation window.

Example:

    Approved
        |
        ↓
    Scheduled
        |
        ↓
    Deployment Window

---

# In Progress

When implementation begins:

    Status = In Progress

Track:

    Deployment
    Progress
    Issues
    Validation

---

# Validation

After deployment:

    Deployment
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Monitoring
        |
        ↓
    Business Validation

---

# Completed

If validation succeeds:

    Deployment
        |
        ↓
    Validation
        |
        ↓
    Success
        |
        ↓
    Completed

---

# Closed

The change can be closed after:

    Validation Complete
    Evidence Recorded
    Approvals Recorded
    Issues Resolved
    Deployment Result Documented

---

# Change Rejection

A change may be rejected when:

    Risk Is Too High
    Plan Is Incomplete
    Rollback Is Missing
    Testing Is Incomplete
    Approval Is Missing
    Deployment Window Is Not Available

Flow:

    Review
        |
        ↓
    Rejected
        |
        ↓
    Update
        |
        ↓
    Review Again

---

# Change Cancellation

A scheduled change may be cancelled because of:

    Business Issue
    Active Incident
    Dependency Failure
    Missing Approval
    Environment Instability
    Security Concern

Example:

    Scheduled Change
        |
        ↓
    Production Incident
        |
        ↓
    Cancel
        |
        ↓
    Reschedule

---

# Change Failure

If implementation fails:

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
    Validation

Then document:

    What Failed
    Why It Failed
    Impact
    Rollback
    Recovery
    Next Action

---

# Failed Change Example

Change:

    Deploy Payment 1.4.7

Deployment:

    Started

Result:

    HTTP 5xx Increased

Action:

    Stop Rollout
        |
        ↓
    Rollback
        |
        ↓
    Payment 1.4.6
        |
        ↓
    Validate
        |
        ↓
    Service Recovered

JIRA should record the outcome.

---

# Change Failure Analysis

After a failed change, investigate:

    Technical Cause
    Process Cause
    Testing Gap
    Configuration Issue
    Dependency Issue
    Infrastructure Issue

This helps improve future deployments.

---

# Change Failure vs Incident

Change failure:

    Deployment did not complete successfully

Incident:

    Production service was negatively affected

They may be related.

Example:

    Change
        |
        ↓
    Deployment Failure
        |
        ↓
    Production Outage
        |
        ↓
    Incident

Both records may need to be linked.

---

# Change and Incident Relationship

Example:

    Change:
    CHG-10245

    Incident:
    INC-7781

Relationship:

    CHG-10245
        |
        ↓
    Deployment Failure
        |
        ↓
    INC-7781
        |
        ↓
    Service Recovery

---

# Change and Problem Management

If a change repeatedly causes issues:

    Change Failures
        |
        ↓
    Investigation
        |
        ↓
    Problem Record
        |
        ↓
    Root Cause
        |
        ↓
    Corrective Action

---

# Change Closure

Before closing:

    Deployment Result Recorded
        |
        ↓
    Validation Completed
        |
        ↓
    Monitoring Stable
        |
        ↓
    Business Validation
        |
        ↓
    Evidence Attached
        |
        ↓
    Change Closed

---

# Change Evidence

Useful evidence includes:

    Deployment Logs
    CI Build
    Git Commit
    Image Tag
    Test Results
    Smoke Test
    Health Checks
    Monitoring Screenshots
    Approval
    Rollback Result
    Validation Result

---

# Change Audit Trail

A good JIRA change record can show:

    Who Created
    Who Reviewed
    Who Approved
    Who Implemented
    What Changed
    When Changed
    What Version
    Which Environment
    Result

This supports auditability.

---

# Change Risk Assessment

Before approval, consider:

    Change Scope
    Number Of Services
    Database Changes
    User Impact
    Downtime
    Dependencies
    Rollback Complexity
    Security Impact
    Business Impact

---

# Risk Matrix

A simple conceptual model:

    Low Probability + Low Impact
        |
        ↓
    Low Risk

    High Probability + Low Impact
        |
        ↓
    Medium Risk

    Low Probability + High Impact
        |
        ↓
    High Risk

    High Probability + High Impact
        |
        ↓
    Critical Risk

Actual organizational risk models may differ.

---

# Risk Mitigation

If risk is high:

    Reduce Scope
        |
        ↓
    Use Canary
        |
        ↓
    Add Monitoring
        |
        ↓
    Improve Rollback
        |
        ↓
    Schedule Carefully
        |
        ↓
    Add Approval

---

# Change Impact Assessment

Assess impact on:

    Users
    Application
    Database
    Infrastructure
    Network
    Security
    External Systems
    Business Operations

Example:

    Payment Deployment
        |
        ↓
    Payment Service
        |
        +-- Order
        +-- Inventory
        +-- Notification
        +-- Database
        +-- External Gateway

---

# Dependency Assessment

Before production deployment:

    Payment
        |
        ↓
    Dependencies

Check:

    Order Service
    Database
    RabbitMQ
    External Payment Gateway
    Authentication

Ensure required dependencies are available.

---

# Production Change Communication

Communicate:

    Change Start
    Deployment Progress
    Deployment Success
    Deployment Failure
    Rollback
    Recovery
    Change Completion

Stakeholders may include:

    DevOps
    QA
    Developers
    Business
    Support
    Release Management

---

# Change Freeze

Organizations may restrict changes during:

    Peak Business Periods
    Major Events
    Financial Close
    Holidays
    Critical Operations

If a change is not critical:

    Change Request
        |
        ↓
    Freeze
        |
        ↓
    Reschedule

---

# Emergency Change During Freeze

If a critical incident occurs:

    Incident
        |
        ↓
    Emergency Change
        |
        ↓
    Approval
        |
        ↓
    Implementation
        |
        ↓
    Validation

Emergency changes should still be documented.

---

# Production Change Checklist

## Before Change

    JIRA Change Created
    Description Complete
    Business Reason Complete
    Risk Assessed
    Impact Assessed
    Dependencies Identified
    Artifact Verified
    Implementation Plan Ready
    Validation Plan Ready
    Rollback Plan Ready
    Approvals Complete
    Deployment Window Confirmed

## During Change

    Change Started
    Deployment Executed
    Rollout Monitored
    Logs Reviewed
    Health Checks Executed
    Metrics Monitored

## After Change

    Smoke Tests Passed
    Business Workflow Validated
    Monitoring Stable
    No Critical Alerts
    Evidence Recorded
    Result Documented
    Change Closed

---

# Production Change Example

Example:

    Change ID:
    CHG-10245

    Application:
    Payment

    Version:
    1.4.7

    Environment:
    Production

    Risk:
    Medium

    Implementation:

    1. Verify cluster health.
    2. Verify image.
    3. Deploy version 1.4.7.
    4. Monitor rollout.
    5. Run health checks.
    6. Run smoke tests.
    7. Monitor metrics.
    8. Validate payment workflow.

    Rollback:

    kubectl rollout undo deployment/payment -n production

---

# JIRA Change and Production Deployment Example

Complete flow:

    UAT
      |
      ↓
    Business Sign-Off
      |
      ↓
    JIRA Change
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
    GitHub Actions / ArgoCD
      |
      ↓
    Production
      |
      ↓
    Health Checks
      |
      ↓
    Smoke Tests
      |
      ↓
    Monitoring
      |
      ↓
    Validation
      |
      ↓
    JIRA Update
      |
      ↓
    Change Closure

---

# JIRA Change and GitHub Actions Example

Conceptual flow:

    JIRA
      |
      ↓
    Approved Change
      |
      ↓
    Production Workflow
      |
      ↓
    GitHub Actions
      |
      +-- Verify Artifact
      +-- Deploy
      +-- Health Check
      +-- Smoke Test
      |
      ↓
    Production
      |
      ↓
    Result
      |
      ↓
    JIRA Updated

---

# JIRA Change and ArgoCD Example

GitOps flow:

    JIRA Change
        |
        ↓
    Approved
        |
        ↓
    Production Manifest
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
    Production
        |
        ↓
    Validation
        |
        ↓
    JIRA Updated

---

# JIRA Change Interview Questions

## Basic

1. What is a JIRA Change Request?

2. Why do we create change requests?

3. What information is included in a change request?

4. What is change management?

5. What is a deployment window?

6. What is a rollback plan?

7. What is change approval?

8. What is a change owner?

9. What is a change closure?

10. What is an audit trail?

---

# JIRA Change Interview Questions

## Intermediate

11. How do you create a production change request?

12. What details do you include in a change request?

13. How do you assess change risk?

14. How do you define change impact?

15. How do you create an implementation plan?

16. How do you create a rollback plan?

17. How do you validate a completed change?

18. What happens when a change fails?

19. How do you link a change to a production deployment?

20. How do you link a change to an incident?

---

# JIRA Change Interview Questions

## Advanced

21. How would you design an enterprise change management process?

22. How would you integrate JIRA with CI/CD?

23. How would you integrate JIRA with GitHub Actions?

24. How would you integrate JIRA with ArgoCD?

25. How would you implement production approval gates?

26. How would you maintain traceability from requirement to production?

27. How would you handle an emergency production change?

28. How would you handle a failed production change?

29. How would you reduce change failure rate?

30. How would you design a rollback strategy for a high-risk change?

31. How would you handle database changes in a production change request?

32. How would you implement separation of duties?

---

# Scenario-Based Interview Question

## Production Change Was Approved but Deployment Fails

Process:

    Deployment Failure
        |
        ↓
    Stop Rollout
        |
        ↓
    Investigate
        |
        ↓
    Rollback If Required
        |
        ↓
    Validate
        |
        ↓
    Update JIRA
        |
        ↓
    Record Failure

Do not mark the change successful if the deployment failed.

---

# Scenario-Based Interview Question

## Change Succeeds but Users Experience 503

Check:

    ALB
    Ingress
    Service
    Endpoints
    Pods
    Readiness
    Application Logs

Flow:

    User
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
    Pod

If the impact is severe:

    Mitigate
        |
        ↓
    Rollback
        |
        ↓
    Validate

---

# Scenario-Based Interview Question

## Emergency Security Patch

Situation:

    Critical Vulnerability
        |
        ↓
    Production At Risk

Process:

    Identify Vulnerability
        |
        ↓
    Emergency Change
        |
        ↓
    Risk Assessment
        |
        ↓
    Emergency Approval
        |
        ↓
    Deployment
        |
        ↓
    Security Validation
        |
        ↓
    Monitoring
        |
        ↓
    Change Closure

---

# Scenario-Based Interview Question

## Change Is High Risk

Reduce risk by:

    Smaller Scope
        |
        ↓
    Canary
        |
        ↓
    Strong Monitoring
        |
        ↓
    Tested Rollback
        |
        ↓
    Controlled Window
        |
        ↓
    Additional Approval

---

# Scenario-Based Interview Question

## Change Request Missing Rollback Plan

Do not approve until the rollback strategy is defined.

Expected:

    Change
        |
        ↓
    Rollback Plan
        |
        ↓
    Validation Plan
        |
        ↓
    Approval

---

# Scenario-Based Interview Question

## Production Change Is Required During Change Freeze

First determine:

    Is It Critical?

If not:

    Reschedule

If critical:

    Emergency Change
        |
        ↓
    Emergency Approval
        |
        ↓
    Controlled Deployment
        |
        ↓
    Validation

---

# Scenario-Based Interview Question

## Deployment Succeeds but Change Validation Fails

Example:

    Deployment = Success

    Smoke Test = Failed

Action:

    Change = Not Successful

Then:

    Investigate
        |
        ↓
    Rollback / Fix
        |
        ↓
    Validate
        |
        ↓
    Update JIRA

---

# Change Management Best Practices

- Create a change request for controlled production changes
- Clearly describe the change
- Document the business reason
- Identify affected systems
- Assess risk
- Assess impact
- Identify dependencies
- Define implementation steps
- Define validation steps
- Define rollback steps
- Obtain required approvals
- Use approved deployment windows
- Promote tested artifacts
- Maintain Git traceability
- Monitor production carefully
- Record deployment evidence
- Link related tickets
- Document failures
- Close changes only after validation
- Maintain auditability

---

# Change Management Anti-Patterns

## Deploying Without a Change

Bad:

    Developer
        |
        ↓
    Production

Better:

    UAT
      |
      ↓
    Change Request
      |
      ↓
    Approval
      |
      ↓
    Production

---

# Change Management Anti-Pattern

## Incomplete Change Description

Bad:

    "Deploy update."

Better:

    "Deploy Payment Service v1.4.7
    to Production EKS to release the
    approved payment validation feature
    and associated defect fixes."

---

# Change Management Anti-Pattern

## No Risk Assessment

Every significant change should consider:

    Impact
    Dependencies
    Failure Possibility
    Rollback Complexity

---

# Change Management Anti-Pattern

## No Validation Plan

Bad:

    Deploy
      |
      ↓
    Done

Better:

    Deploy
      |
      ↓
    Health Check
      |
      ↓
    Smoke Test
      |
      ↓
    Monitoring
      |
      ↓
    Business Validation

---

# Change Management Anti-Pattern

## No Rollback Plan

Bad:

    Failure
      |
      ↓
    "What do we do now?"

Better:

    Failure
      |
      ↓
    Predefined Rollback
      |
      ↓
    Recovery
      |
      ↓
    Validation

---

# Change Management Anti-Pattern

## Closing Change Without Validation

Do not close a change immediately after deployment.

Correct:

    Deployment
        |
        ↓
    Validation
        |
        ↓
    Monitoring
        |
        ↓
    Success
        |
        ↓
    Closure

---

# Complete Enterprise Change Flow

    Requirement
        |
        ↓
    Development
        |
        ↓
    Code Review
        |
        ↓
    CI
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
    Business Sign-Off
        |
        ↓
    JIRA Change
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
        |
        ↓
    Health Check
        |
        ↓
    Smoke Test
        |
        ↓
    Monitoring
        |
        ↓
    Business Validation
        |
        ↓
    Change Result
        |
        ↓
    Closure

---

# Final Mental Model

Remember:

    JIRA Change Request
        =
    Controlled Production Change

The change process is:

    PLAN
      |
      ↓
    ASSESS
      |
      ↓
    APPROVE
      |
      ↓
    SCHEDULE
      |
      ↓
    IMPLEMENT
      |
      ↓
    VALIDATE
      |
      ↓
    CLOSE

If the change fails:

    IMPLEMENT
        |
        ↓
    FAILURE
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

A JIRA Change Request is not just a ticket for tracking deployment.

It provides:

    Governance
        +
    Risk Management
        +
    Approval
        +
    Deployment Planning
        +
    Rollback Planning
        +
    Traceability
        +
    Auditability

The complete enterprise relationship is:

    JIRA Change
        |
        ↓
    Approved Release
        |
        ↓
    CI/CD
        |
        ↓
    Production
        |
        ↓
    Monitoring
        |
        ↓
    Validation
        |
        ↓
    JIRA Update
        |
        ↓
    Change Closure

The key principle is:

    No Uncontrolled Production Change

Every important production change should be:

    Planned
        +
    Assessed
        +
    Approved
        +
    Implemented
        +
    Validated
        +
    Documented

The next enterprise topic is:

    13-Enterprise-Workflows
        |
        ↓
    06-Approval-Process