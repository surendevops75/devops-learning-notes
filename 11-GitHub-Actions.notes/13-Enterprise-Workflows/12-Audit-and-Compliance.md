# Audit and Compliance

Audit and Compliance is the process of ensuring that systems, infrastructure, deployments, security controls, and operational activities follow defined organizational, regulatory, and security requirements.

In an enterprise DevOps environment, audit and compliance focuses on:

    Traceability
    +
    Access Control
    +
    Change Management
    +
    Security
    +
    Data Protection
    +
    Evidence
    +
    Accountability
    +
    Continuous Monitoring

A typical compliance flow is:

    Requirement
        |
        ↓
    Policy
        |
        ↓
    Control
        |
        ↓
    Implementation
        |
        ↓
    Monitoring
        |
        ↓
    Evidence
        |
        ↓
    Audit
        |
        ↓
    Findings
        |
        ↓
    Remediation
        |
        ↓
    Validation
        |
        ↓
    Continuous Improvement

---

# What Is Audit?

An audit is a systematic review of systems, processes, controls, and evidence to determine whether requirements are being followed.

An audit may verify:

    Who Made A Change?
    What Was Changed?
    When Was It Changed?
    Who Approved It?
    Was Testing Completed?
    Was Security Validation Completed?
    Was The Change Successful?
    Was The Activity Authorized?

---

# What Is Compliance?

Compliance means following applicable:

    Laws
    Regulations
    Standards
    Policies
    Contracts
    Organizational Controls

Examples may include:

    Internal Security Policies
    Data Protection Requirements
    Industry Standards
    Customer Requirements
    Regulatory Requirements

The exact requirements depend on the organization and industry.

---

# Audit vs Compliance

Audit:

    Checks Whether Requirements
    Are Being Followed

Compliance:

    Ensures Requirements
    Are Being Met

Relationship:

    Requirement
        |
        ↓
    Compliance
        |
        ↓
    Evidence
        |
        ↓
    Audit

---

# Why Audit and Compliance Matter

Enterprise organizations need to demonstrate:

    Controlled Access
    Controlled Changes
    Secure Deployments
    Data Protection
    Operational Accountability
    Security Monitoring
    Incident Management
    Recovery Capability

Compliance is not only about documentation.

It also requires effective technical controls.

---

# DevOps and Compliance

DevOps teams participate in compliance through:

    Git
    CI/CD
    Infrastructure as Code
    Kubernetes
    Cloud IAM
    Security Scanning
    Logging
    Monitoring
    Change Management
    Deployment Approvals
    Backup
    Disaster Recovery

---

# DevSecOps and Compliance

DevSecOps integrates security and compliance into the delivery process.

Flow:

    Developer
        |
        ↓
    Git
        |
        ↓
    CI
        |
        +-- Build
        +-- Test
        +-- SonarQube
        +-- Trivy
        +-- Veracode
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Monitoring
        |
        ↓
    Audit Evidence

---

# Auditability

Auditability means that an activity can be traced back through reliable records.

Example:

    Production Deployment
        |
        ↓
    Git Commit
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    CI Pipeline
        |
        ↓
    Security Checks
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Result

This creates traceability.

---

# Traceability

Traceability answers:

    What?
    Who?
    When?
    Why?
    Where?
    Approval?
    Result?

Example:

    Change:
    CHG-10525

    Application:
    Payment

    Version:
    1.4.8

    Git SHA:
    abc123

    Approved By:
    Release Manager

    Deployment:
    Production

    Result:
    Successful

---

# Change Traceability

A production change should ideally be traceable to:

    Requirement
        |
        ↓
    Ticket
        |
        ↓
    Git Commit
        |
        ↓
    Pull Request
        |
        ↓
    Pipeline
        |
        ↓
    Artifact
        |
        ↓
    Approval
        |
        ↓
    Deployment

---

# Git as an Audit Source

Git provides records such as:

    Commit
    Author
    Timestamp
    Branch
    Pull Request
    Review
    Merge
    Commit SHA

Example:

    Developer
        |
        ↓
    Commit
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    Merge

---

# Pull Requests and Compliance

Pull Requests can provide:

    Code Review
    Change Discussion
    Approval
    Traceability
    Review History

Example:

    Feature
        |
        ↓
    Pull Request
        |
        ↓
    Reviewer
        |
        ↓
    Approval
        |
        ↓
    Merge

---

# Branch Protection

Branch protection can enforce controls such as:

    Required Pull Request
    Required Review
    Required Status Checks
    Restricted Direct Push
    Required Approvals

Example:

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
    CI Checks
        |
        ↓
    Main Branch

---

# Separation of Duties

Separation of Duties means critical activities should not always be controlled by a single person.

Example:

    Developer
        |
        ↓
    Creates Change

    Reviewer
        |
        ↓
    Reviews Change

    Release Approver
        |
        ↓
    Approves Production

This reduces the risk of unauthorized changes.

---

# Separation of Duties in CI/CD

Example:

    Developer
        |
        ↓
    Code
        |
        ↓
    Pull Request
        |
        ↓
    Reviewer
        |
        ↓
    CI
        |
        ↓
    Release Approval
        |
        ↓
    Production

---

# Least Privilege

Least privilege means users and systems receive only the permissions required to perform their responsibilities.

Example:

    Developer
        |
        ↓
    Development Access

    CI/CD
        |
        ↓
    Deployment Permissions

    Production Operator
        |
        ↓
    Required Production Access

Avoid:

    Everyone
        |
        ↓
    Administrator Access

---

# IAM and Compliance

IAM controls:

    Who Can Access?
    What Can They Access?
    What Can They Do?

Example:

    User
        |
        ↓
    IAM Role
        |
        ↓
    Permissions
        |
        ↓
    AWS Resource

---

# IAM Audit

Review:

    Users
    Roles
    Policies
    Permissions
    Access Keys
    Service Accounts
    Trust Relationships

Look for:

    Excessive Permissions
    Unused Access
    Long-Lived Credentials
    Unauthorized Access

---

# Service Accounts

Applications may use service identities.

Example:

    Application
        |
        ↓
    Service Account
        |
        ↓
    IAM Role
        |
        ↓
    AWS Resource

Service identities should follow least privilege.

---

# Credentials and Compliance

Avoid storing credentials in:

    Git
    Docker Images
    Plain Text Files
    CI Logs
    Kubernetes Manifests

Use secure secret-management mechanisms.

---

# Secrets Management

Secrets may include:

    Passwords
    API Tokens
    Database Credentials
    Certificates
    Private Keys

Protect them through:

    Access Control
    Encryption
    Rotation
    Audit Logging

---

# Secret Rotation

Example:

    Secret
        |
        ↓
    Rotation
        |
        ↓
    New Credential
        |
        ↓
    Application
        |
        ↓
    Validation

Rotation reduces long-term credential exposure.

---

# Secret Exposure

If a secret is exposed:

    Detect
        |
        ↓
    Revoke / Rotate
        |
        ↓
    Investigate
        |
        ↓
    Validate
        |
        ↓
    Prevent Recurrence

Do not simply delete the exposed file and assume the secret is safe.

---

# CI/CD Audit

A CI/CD audit may verify:

    Who Triggered Pipeline?
    Which Commit Was Built?
    Which Tests Ran?
    Which Security Checks Ran?
    Which Artifact Was Created?
    Who Approved Deployment?
    Where Was It Deployed?
    What Was The Result?

---

# Pipeline Traceability

Example:

    Git Commit
        |
        ↓
    GitHub Actions
        |
        ↓
    Build
        |
        ↓
    Test
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Veracode
        |
        ↓
    Artifact
        |
        ↓
    Approval
        |
        ↓
    Production

---

# Artifact Traceability

Every production artifact should ideally be traceable to its source.

Example:

    Git SHA
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
    Deployment

Example:

    payment:1.4.8

should map to:

    Git SHA:
    abc123

---

# Immutable Artifacts

An immutable artifact should not be modified after it is published.

Example:

    Build
        |
        ↓
    payment:1.4.8
        |
        ↓
    ECR
        |
        ↓
    Deploy Same Artifact

This improves:

    Reproducibility
    Traceability
    Auditability

---

# Docker Image Traceability

Example:

    Developer
        |
        ↓
    Git Commit
        |
        ↓
    CI
        |
        ↓
    Docker Build
        |
        ↓
    Image
        |
        ↓
    ECR
        |
        ↓
    EKS

The deployed image should be identifiable.

---

# Container Security Compliance

Container images can be scanned using:

    Trivy

Typical flow:

    Dockerfile
        |
        ↓
    Build Image
        |
        ↓
    Trivy
        |
        ↓
    Vulnerability Results
        |
        ↓
    Security Gate
        |
        ↓
    Artifact

---

# SAST and Compliance

Static Application Security Testing can identify security issues in source code.

Example:

    Source Code
        |
        ↓
    SonarQube / SAST
        |
        ↓
    Findings
        |
        ↓
    Quality / Security Gate

---

# SCA and Compliance

Software Composition Analysis checks dependencies.

It can identify:

    Vulnerable Libraries
    Outdated Dependencies
    License Risks

The exact tool and policy depend on the organization.

---

# DAST and Compliance

Dynamic Application Security Testing evaluates a running application.

Example:

    Application
        |
        ↓
    DAST
        |
        ↓
    Security Findings
        |
        ↓
    Remediation

---

# Veracode and Compliance

Veracode can be used as part of an organization's application security process.

Example:

    Build
        |
        ↓
    Application Security Scan
        |
        ↓
    Findings
        |
        ↓
    Security Gate
        |
        ↓
    Approval

The exact policies and scan types depend on organizational configuration.

---

# Security Gates

A security gate prevents an artifact from progressing when required security criteria are not met.

Example:

    Build
        |
        ↓
    Security Scan
        |
        ↓
    Critical Vulnerability
        |
        X
    Deployment Blocked

---

# Quality Gates

Quality gates can enforce:

    Code Quality
    Test Coverage
    Security Conditions
    Vulnerability Thresholds

Example:

    SonarQube
        |
        ↓
    Quality Gate
        |
        +------ Pass → Continue
        |
        +------ Fail → Stop

---

# Compliance Gates

A compliance gate may verify:

    Required Approval
    Security Scan
    Change Ticket
    Testing
    Artifact Traceability

Example:

    Deployment Request
        |
        ↓
    Change Ticket
        |
        ↓
    Security
        |
        ↓
    Approval
        |
        ↓
    Production

---

# Production Approval

Production deployments may require approval.

Example:

    CI
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
    Production Approval
        |
        ↓
    Production

---

# Change Management

Compliance often requires controlled changes.

A change may include:

    Change ID
    Description
    Reason
    Risk
    Impact
    Testing
    Implementation Plan
    Rollback Plan
    Approval
    Deployment Window
    Result

---

# Change Ticket

Example:

    CHG-10525

    Application:
    Payment

    Version:
    1.4.8

    Environment:
    Production

    Risk:
    Medium

    Rollback:
    Previous Version

    Approval:
    Required

---

# Standard Change

A standard change is a predefined, low-risk, repeatable change that follows an approved procedure.

Examples may include:

    Routine Maintenance
    Approved Configuration Update
    Repeated Operational Procedure

The exact classification depends on organizational policy.

---

# Normal Change

A normal change generally requires assessment, planning, testing, and approval according to organizational procedures.

Example:

    Application Release
        |
        ↓
    Change Request
        |
        ↓
    Risk Assessment
        |
        ↓
    Testing
        |
        ↓
    Approval
        |
        ↓
    Deployment

---

# Emergency Change

An emergency change is performed to address urgent issues such as:

    Major Outage
    Critical Security Vulnerability
    Severe Production Failure

Even emergency changes should maintain appropriate evidence and retrospective review.

---

# Emergency Change Flow

    Critical Issue
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
        |
        ↓
    Documentation
        |
        ↓
    Review

---

# Audit Evidence

Evidence demonstrates that a control was performed.

Examples:

    Git Logs
    Pull Requests
    Pipeline Logs
    Security Scan Results
    Approval Records
    Change Tickets
    Deployment Records
    Access Reviews
    Backup Reports
    DR Test Results

---

# Evidence Collection

Example:

    Control
        |
        ↓
    Activity
        |
        ↓
    Evidence
        |
        ↓
    Audit

Example:

    Production Approval Required

Evidence:

    Approval Record

---

# Evidence Quality

Good evidence should be:

    Relevant
    Accurate
    Complete
    Traceable
    Time-Stamped
    Protected

---

# Audit Evidence Example

Requirement:

    Production Changes Must Be Approved

Evidence:

    Change Ticket
        +
    Pull Request Approval
        +
    Pipeline Approval
        +
    Deployment Record

---

# Logs and Audit

Logs can provide:

    Event
    Timestamp
    Identity
    Action
    Result

Example:

    User:
    deployment-service

    Action:
    Deployment

    Resource:
    payment

    Time:
    22:15

    Result:
    Successful

---

# Log Integrity

Audit logs should be protected from unauthorized modification.

Consider:

    Access Control
    Centralized Logging
    Retention
    Encryption
    Monitoring

---

# ELK and Audit

ELK can centralize application and operational logs.

Flow:

    Applications
        |
        ↓
    Logs
        |
        ↓
    ELK
        |
        ↓
    Search / Analysis
        |
        ↓
    Investigation / Evidence

---

# Log Retention

Define:

    What Logs?
    How Long?
    Who Can Access?
    How Are They Protected?
    When Can They Be Deleted?

Retention requirements vary by:

    Organization
    Industry
    Contract
    Regulation

---

# Monitoring and Compliance

Monitoring can provide evidence for:

    Availability
    Security
    Performance
    Operational Health

Example:

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

# Compliance Monitoring

Continuous monitoring can detect:

    Unauthorized Changes
    Security Findings
    Configuration Drift
    Failed Deployments
    Access Anomalies
    Policy Violations

---

# Configuration Compliance

Infrastructure should follow approved configuration.

Example:

    Approved Configuration
        |
        ↓
    Actual Infrastructure
        |
        ↓
    Compare
        |
        ↓
    Drift Detected
        |
        ↓
    Investigate

---

# Infrastructure as Code and Compliance

Terraform provides version-controlled infrastructure definitions.

Example:

    Git
        |
        ↓
    Terraform
        |
        ↓
    Infrastructure

Benefits:

    Version Control
    Review
    Traceability
    Repeatability
    Standardization

---

# Terraform Pull Request

Example:

    Terraform Change
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    terraform plan
        |
        ↓
    Approval
        |
        ↓
    Apply

This creates an auditable infrastructure change process.

---

# Terraform Plan as Evidence

Terraform plan can show:

    Resources To Add
    Resources To Change
    Resources To Destroy

Example:

    Plan
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    Apply

The plan output can be retained according to organizational procedures.

---

# Terraform Compliance

Terraform can be integrated with policy checks.

Example:

    Terraform
        |
        ↓
    Plan
        |
        ↓
    Policy Validation
        |
        +------ Pass
        |
        +------ Fail

Examples of policies:

    No Public Storage
    Required Tags
    Approved Regions
    Encryption Required

The exact policy tooling depends on the organization.

---

# Kubernetes Compliance

Kubernetes compliance may include:

    RBAC
    Network Policies
    Pod Security
    Secrets
    Image Security
    Resource Controls
    Audit Logging

---

# Kubernetes RBAC

RBAC controls:

    Who
        |
        ↓
    Can Perform
        |
        ↓
    Which Action
        |
        ↓
    On Which Resource

Example:

    Developer
        |
        ↓
    Namespace
        |
        ↓
    Read Pods

---

# Namespace Isolation

Example:

    Production
        |
        ↓
    Restricted Access

    Development
        |
        ↓
    Developer Access

Separate namespaces can support organizational isolation, but namespace separation alone is not a complete security boundary.

---

# Kubernetes Secrets

Kubernetes Secrets should be handled securely.

Consider:

    Access Control
    Encryption
    Rotation
    Avoiding Secret Exposure
    Auditability

Do not expose secrets through:

    Logs
    Git
    Container Images

---

# Container Image Compliance

Verify:

    Approved Registry
    Image Provenance
    Vulnerability Scan
    Image Tag
    Digest
    Security Policy

Example:

    ECR
        |
        ↓
    Image
        |
        ↓
    Trivy
        |
        ↓
    Security Gate
        |
        ↓
    EKS

---

# Image Digest

Tags can change depending on registry practices.

A digest identifies a specific image content.

Example:

    image:tag
        |
        ↓
    Image Digest
        |
        ↓
    Exact Artifact

Using immutable identifiers can improve traceability.

---

# Production Deployment Evidence

A production deployment record may include:

    Application
    Version
    Image
    Git SHA
    Change ID
    Approver
    Deployment Time
    Environment
    Result

---

# Deployment Record

Example:

    Application:
    payment

    Version:
    1.4.8

    Image:
    payment:1.4.8

    Git SHA:
    abc123

    Environment:
    Production

    Change:
    CHG-10525

    Result:
    Successful

---

# Audit Trail

A complete audit trail can look like:

    Requirement
        |
        ↓
    Ticket
        |
        ↓
    Code
        |
        ↓
    Pull Request
        |
        ↓
    Review
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
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Monitoring
        |
        ↓
    Result

---

# Access Review

Access should be periodically reviewed.

Check:

    Who Has Access?
    Why Do They Have Access?
    Is Access Still Required?
    Are Permissions Excessive?
    Are Former Users Removed?

---

# Joiner-Mover-Leaver Process

A common access lifecycle:

    Joiner
        |
        ↓
    Access Granted

    Mover
        |
        ↓
    Access Updated

    Leaver
        |
        ↓
    Access Removed

---

# Access Review Evidence

Evidence may include:

    User List
    Role Assignments
    Approval
    Review Date
    Reviewer
    Remediation

---

# Privileged Access

Privileged access includes permissions such as:

    Administrator
    Root
    Production Access
    IAM Administration

Protect with:

    Strong Authentication
    Least Privilege
    Approval
    Monitoring
    Periodic Review

---

# Production Access

Production access should be controlled.

Example:

    Request
        |
        ↓
    Approval
        |
        ↓
    Temporary / Required Access
        |
        ↓
    Activity
        |
        ↓
    Audit Log

---

# Direct Production Changes

Risky:

    Engineer
        |
        ↓
    Direct Server Change
        |
        ↓
    No Ticket
        |
        ↓
    No Review

Preferred:

    Git / Change Process
        |
        ↓
    Review
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

# Configuration Change Audit

Example:

    Configuration
        |
        ↓
    Git
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    Pipeline
        |
        ↓
    Deployment

This is more traceable than undocumented manual changes.

---

# Compliance and CI/CD Approval

Example:

    Build
        |
        ↓
    Tests
        |
        ↓
    Security
        |
        ↓
    Artifact
        |
        ↓
    Approval
        |
        ↓
    Production

---

# Environment Controls

Different environments may have different controls.

Example:

    Development
        |
        ↓
    Developer Access

    QA
        |
        ↓
    Testing

    SIT
        |
        ↓
    Integration

    UAT
        |
        ↓
    Business Validation

    Production
        |
        ↓
    Restricted Access + Approval

---

# Environment Promotion

Example:

    Build Once
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
    Production

Promoting the same artifact improves traceability.

---

# Build Once, Deploy Many

Preferred pattern:

    Source
        |
        ↓
    Build
        |
        ↓
    Artifact
        |
        +------ QA
        |
        +------ SIT
        |
        +------ UAT
        |
        +------ Production

Avoid rebuilding different artifacts for each environment when the goal is to promote the same tested artifact.

---

# Compliance and Artifact Promotion

Example:

    payment:1.4.8

    QA
        ✓

    SIT
        ✓

    UAT
        ✓

    Production
        ✓

The same artifact can be traced across environments.

---

# Audit and Release Management

Release management should maintain:

    Release Version
    Change ID
    Testing
    Approval
    Deployment
    Rollback
    Result

---

# Release Evidence

Example:

    Release:
    1.4.8

    Change:
    CHG-10525

    Tests:
    Passed

    Security:
    Passed

    Approval:
    Completed

    Production:
    Successful

---

# Incident and Audit

When an incident occurs, retain:

    Timeline
    Alerts
    Logs
    Changes
    Actions
    Communications
    Root Cause
    Corrective Actions

---

# Incident Traceability

Example:

    Incident
        |
        ↓
    Deployment
        |
        ↓
    Git Commit
        |
        ↓
    Change Ticket
        |
        ↓
    Root Cause
        |
        ↓
    Corrective Action

---

# Change Failure and Audit

When a deployment fails:

    Change
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
    RCA
        |
        ↓
    Evidence

---

# Disaster Recovery and Audit

DR testing should produce evidence such as:

    Test Date
    Scope
    RTO
    RPO
    Recovery Steps
    Results
    Failures
    Corrective Actions

---

# DR Audit Evidence

Example:

    DR Exercise
        |
        ↓
    Primary Failure Simulated
        |
        ↓
    DR Activated
        |
        ↓
    Application Recovered
        |
        ↓
    RTO Measured
        |
        ↓
    RPO Measured
        |
        ↓
    Business Validation
        |
        ↓
    Report

---

# Backup Audit

Verify:

    Backup Frequency
    Backup Success
    Retention
    Encryption
    Access
    Restore Testing

---

# Backup Failure

If backups fail:

    Detect
        |
        ↓
    Alert
        |
        ↓
    Investigate
        |
        ↓
    Fix
        |
        ↓
    Re-run Backup
        |
        ↓
    Validate

A failed backup should not remain unnoticed.

---

# Compliance Monitoring

Continuous monitoring can detect:

    Failed Backups
    Vulnerabilities
    Unauthorized Access
    Configuration Drift
    Failed Deployments
    Policy Violations

---

# Continuous Compliance

Traditional model:

    Audit
        |
        ↓
    Once A Year

Continuous model:

    Policy
        |
        ↓
    Automated Control
        |
        ↓
    Continuous Monitoring
        |
        ↓
    Evidence
        |
        ↓
    Remediation

---

# Policy as Code

Policy as Code expresses compliance rules in machine-readable form.

Example:

    Terraform
        |
        ↓
    Policy
        |
        ↓
    Validation
        |
        +------ Compliant
        |
        +------ Non-Compliant

Possible policies:

    Encryption Required
    Approved Region
    Required Tags
    No Public Resources

---

# Policy Enforcement

Example:

    Infrastructure Change
        |
        ↓
    Terraform Plan
        |
        ↓
    Policy Check
        |
        X
    Violation
        |
        ↓
    Deployment Blocked

---

# Compliance Automation

Automate:

    Security Scans
    Policy Checks
    Access Reviews
    Configuration Checks
    Backup Validation
    Deployment Evidence
    Change Validation

Automation reduces manual effort and improves consistency.

---

# Compliance Dashboard

A dashboard can show:

    Open Findings
    Security Issues
    Failed Controls
    Failed Backups
    Unauthorized Changes
    Deployment Failures
    Access Review Status

---

# Compliance Findings

An audit finding identifies a gap between:

    Requirement
        |
        ↓
    Expected Control

and:

    Actual State

Example:

    Requirement:
    Production changes require approval.

    Actual:
    Direct production changes were found.

Finding:

    Control Gap

---

# Finding Severity

Organizations may classify findings as:

    Critical
    High
    Medium
    Low

The exact classification depends on organizational policy.

---

# Remediation

After finding:

    Finding
        |
        ↓
    Root Cause
        |
        ↓
    Remediation
        |
        ↓
    Validation
        |
        ↓
    Closure

---

# Remediation Example

Finding:

    Developers Can Directly Modify Production

Remediation:

    Restrict Production Access
        |
        ↓
    Enforce CI/CD
        |
        ↓
    Require Approval
        |
        ↓
    Monitor Access
        |
        ↓
    Validate

---

# Compliance Exceptions

Sometimes a requirement cannot immediately be met.

An exception should be:

    Documented
    Approved
    Time-Bounded
    Risk-Assessed
    Reviewed

Do not treat exceptions as permanent bypasses.

---

# Risk Acceptance

If a control cannot currently be implemented:

    Identify Risk
        |
        ↓
    Assess Risk
        |
        ↓
    Define Mitigation
        |
        ↓
    Obtain Approval
        |
        ↓
    Set Expiration
        |
        ↓
    Review

---

# Audit Finding Closure

A finding is not necessarily closed because:

    "We Fixed It"

Evidence should demonstrate:

    Remediation Completed
        |
        ↓
    Control Tested
        |
        ↓
    Evidence Collected
        |
        ↓
    Validation
        |
        ↓
    Closure

---

# Compliance Metrics

Useful metrics can include:

    Open Findings
    Closed Findings
    Overdue Findings
    Vulnerabilities
    Failed Controls
    Unauthorized Changes
    Access Review Completion
    Backup Success Rate
    DR Test Success
    Change Failure Rate

---

# Security Metrics

Examples:

    Critical Vulnerabilities
    High Vulnerabilities
    Mean Time To Remediate
    Secret Exposure Events
    Failed Security Gates
    Unauthorized Access Attempts

---

# Change Metrics

Examples:

    Deployment Count
    Change Failure Rate
    Rollback Rate
    Emergency Changes
    Unauthorized Changes
    Failed Changes

---

# Audit Metrics

Examples:

    Open Findings
    Finding Closure Time
    Repeat Findings
    Evidence Completion
    Control Failure Rate

---

# Repeat Findings

A repeat finding indicates that a previous problem was not adequately addressed.

Example:

    Audit Finding
        |
        ↓
    Remediation
        |
        ↓
    Next Audit
        |
        ↓
    Same Finding

This indicates a systemic improvement problem.

---

# Compliance Culture

A strong compliance culture means:

    Security Is Everyone's Responsibility
    Changes Are Traceable
    Access Is Controlled
    Evidence Is Maintained
    Issues Are Remediated
    Exceptions Are Managed

Compliance should be integrated into normal engineering processes.

---

# Compliance by Design

Instead of:

    Build
        |
        ↓
    Deploy
        |
        ↓
    Audit Later

Prefer:

    Design
        |
        ↓
    Security
        |
        ↓
    Compliance
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
    Monitor

---

# Secure SDLC and Compliance

A secure SDLC can include:

    Requirements
        |
        ↓
    Design
        |
        ↓
    Development
        |
        ↓
    Code Review
        |
        ↓
    Testing
        |
        ↓
    Security Testing
        |
        ↓
    Deployment
        |
        ↓
    Monitoring
        |
        ↓
    Maintenance

---

# Compliance in Development

Developers can contribute by:

    Secure Coding
    Code Review
    Dependency Management
    Secret Protection
    Testing
    Documentation

---

# Compliance in CI

CI can enforce:

    Build
    Tests
    Quality
    Security
    Dependency Checks
    Policy Checks

Example:

    Commit
        |
        ↓
    Build
        |
        ↓
    Tests
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Veracode
        |
        ↓
    Gate

---

# Compliance in CD

CD can enforce:

    Approved Artifact
    Environment Controls
    Change Approval
    Deployment Restrictions
    Health Checks
    Validation

---

# Compliance in Kubernetes

Controls may include:

    RBAC
    Network Policies
    Image Security
    Namespace Controls
    Resource Policies
    Secret Controls
    Audit Logging

---

# Compliance in AWS

Controls may include:

    IAM
    Encryption
    S3 Controls
    Network Security
    Logging
    Monitoring
    Backup
    Resource Policies

---

# Compliance and Networking

Review:

    VPC
    Subnets
    Security Groups
    Network ACLs
    Load Balancers
    Internet Exposure

Questions:

    Is This Resource Public?

    Is Access Required?

    Are Security Rules Too Broad?

---

# Public Resource Risk

Example:

    Internet
        |
        ↓
    Public Resource
        |
        ↓
    Sensitive Data

This may create significant security risk.

Preferred:

    Internet
        |
        ↓
    ALB
        |
        ↓
    Application
        |
        ↓
    Private Database

The exact architecture depends on requirements.

---

# Encryption Compliance

Protect data:

    At Rest
        +
    In Transit

Examples:

    TLS
    Encrypted Storage
    Encrypted Database
    Encrypted Backups

---

# Data Protection

Identify:

    What Data?
    Where Stored?
    Who Can Access?
    How Protected?
    How Long Retained?
    How Deleted?

---

# Data Classification

Organizations may classify data such as:

    Public
    Internal
    Confidential
    Restricted

The exact categories depend on organizational policy.

---

# Data Retention

Retention defines:

    How Long Data Is Kept

Consider:

    Business Requirements
    Security
    Legal Requirements
    Regulatory Requirements
    Storage Cost

---

# Data Deletion

Secure deletion may be required when data reaches the end of its retention period.

Consider:

    Primary Data
    Backups
    Replicas
    Logs
    Temporary Copies

---

# Compliance and Logging

Logs should support:

    Security Investigation
    Operational Troubleshooting
    Audit
    Incident Response

But logs should not expose:

    Passwords
    Tokens
    Secrets
    Sensitive Data

---

# Audit Logging

Audit logs should capture appropriate events such as:

    Authentication
    Authorization
    Configuration Changes
    Administrative Actions
    Deployment Events

---

# Audit Log Protection

Protect logs through:

    Access Control
    Retention
    Encryption
    Centralization
    Monitoring

---

# Compliance Incident Response

When a compliance-related incident occurs:

    Detect
        |
        ↓
    Assess
        |
        ↓
    Contain
        |
        ↓
    Investigate
        |
        ↓
    Remediate
        |
        ↓
    Document
        |
        ↓
    Report If Required

Reporting requirements depend on the organization and applicable regulations.

---

# Audit Preparation

Before an audit:

    Review Controls
        |
        ↓
    Collect Evidence
        |
        ↓
    Identify Gaps
        |
        ↓
    Remediate
        |
        ↓
    Validate
        |
        ↓
    Prepare Evidence Package

---

# Audit Evidence Package

May contain:

    Policies
    Procedures
    Access Reviews
    Change Records
    Deployment Records
    Security Scan Results
    Backup Reports
    DR Test Reports
    Incident Records
    Monitoring Evidence

---

# Evidence Ownership

Each control should have an owner.

Example:

    Control:
    Production Approval

    Owner:
    Release Management

    Evidence:
    Deployment Approval

    Review:
    Periodic

Clear ownership prevents missing evidence.

---

# Control Mapping

Example:

    Requirement
        |
        ↓
    Control
        |
        ↓
    Implementation
        |
        ↓
    Evidence

Example:

    Requirement:
    Production changes must be authorized.

    Control:
    Production approval.

    Implementation:
    CI/CD environment approval.

    Evidence:
    Approval record.

---

# Control Testing

Control testing determines whether the control actually works.

Example:

    Control:
    Production Requires Approval

Test:

    Attempt Deployment Without Approval

Expected:

    Deployment Blocked

Result:

    Control Effective

---

# Preventive vs Detective Controls

Preventive control:

    Prevents Problem

Example:

    Production Approval Gate

Detective control:

    Detects Problem

Example:

    Unauthorized Deployment Alert

---

# Corrective Control

Corrective control helps recover after a problem.

Example:

    Unauthorized Change
        |
        ↓
    Detect
        |
        ↓
    Rollback
        |
        ↓
    Investigate

---

# Three Control Types

    Preventive
        |
        ↓
    Stop Problem

    Detective
        |
        ↓
    Find Problem

    Corrective
        |
        ↓
    Recover Problem

A mature system uses all three.

---

# Compliance Control Example

Requirement:

    Production changes must be reviewed.

Preventive:

    Branch Protection

Detective:

    Audit Log Review

Corrective:

    Revert Unauthorized Change

---

# Compliance and Automation

Example:

    Terraform Pull Request
        |
        ↓
    Security Policy
        |
        ↓
    Compliance Check
        |
        +------ Fail
        |
        ↓
    Block Merge

This shifts compliance left.

---

# Compliance and GitOps

GitOps can improve traceability.

Example:

    Git
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes
        |
        ↓
    Audit Trail

---

# GitOps Drift Detection

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
        |
        ↓
    Investigate
        |
        ↓
    Reconcile / Remediate

---

# Unauthorized Kubernetes Change

Example:

    Engineer
        |
        ↓
    kubectl
        |
        ↓
    Production
        |
        ↓
    Configuration Changed

Compliance response:

    Detect
        |
        ↓
    Identify User
        |
        ↓
    Identify Change
        |
        ↓
    Assess
        |
        ↓
    Reconcile
        |
        ↓
    Review

---

# Audit and ArgoCD

ArgoCD can provide useful deployment visibility such as:

    Application
    Sync Status
    Revision
    Health
    Deployment State

This supports operational traceability.

---

# Audit and Terraform

Terraform provides:

    Configuration History
    Plan
    Apply
    Resource Changes

Git provides:

    Who Changed Code
    Review
    Commit History

Together:

    Git
        +
    Terraform
        =
    Infrastructure Traceability

---

# Audit and GitHub Actions

GitHub Actions can provide:

    Workflow Runs
    Commit
    Branch
    Job Results
    Logs
    Artifacts

This can support CI/CD audit evidence.

---

# Audit and Security Tools

Security tools can provide evidence from:

    SonarQube
    Trivy
    Veracode

Example:

    Commit
        |
        ↓
    Security Scan
        |
        ↓
    Results
        |
        ↓
    Gate
        |
        ↓
    Artifact

---

# Compliance Failure Handling

If a control fails:

    Detect
        |
        ↓
    Assess
        |
        ↓
    Contain
        |
        ↓
    Remediate
        |
        ↓
    Validate
        |
        ↓
    Document

---

# Example: Failed Security Gate

    Build
        |
        ↓
    Trivy
        |
        X
    Critical Vulnerability
        |
        ↓
    Deployment Blocked
        |
        ↓
    Developer Fix
        |
        ↓
    Rebuild
        |
        ↓
    Rescan
        |
        ✓
    Continue

---

# Example: Missing Production Approval

    Deployment Request
        |
        X
    Approval Missing
        |
        ↓
    Deployment Blocked

Action:

    Obtain Approval
        |
        ↓
    Revalidate
        |
        ↓
    Deploy

---

# Example: Unauthorized Production Change

    Production Change
        |
        ↓
    Audit Detection
        |
        ↓
    Identify User
        |
        ↓
    Identify Change
        |
        ↓
    Assess Impact
        |
        ↓
    Restore Approved State
        |
        ↓
    Investigate
        |
        ↓
    Prevent Recurrence

---

# Example: Terraform Public Resource

    Terraform Plan
        |
        ↓
    Resource Public
        |
        X
    Policy Violation
        |
        ↓
    Pipeline Blocked
        |
        ↓
    Developer Fix
        |
        ↓
    Plan
        |
        ✓
    Approved

---

# Example: Vulnerable Container

    Docker Build
        |
        ↓
    Trivy
        |
        X
    Critical CVE
        |
        ↓
    Security Gate
        |
        X
    Deployment Blocked
        |
        ↓
    Upgrade Dependency
        |
        ↓
    Rebuild
        |
        ↓
    Scan
        |
        ✓
    Deploy

---

# Example: Failed Access Review

    Access Review
        |
        ↓
    Unused Production Access
        |
        ↓
    Remove Access
        |
        ↓
    Validate
        |
        ↓
    Evidence

---

# Compliance Best Practices

- Use least privilege
- Enforce separation of duties
- Protect production access
- Require code review
- Protect main branches
- Maintain change records
- Use immutable artifacts
- Track Git SHA
- Track deployment versions
- Use security gates
- Scan dependencies
- Scan container images
- Protect secrets
- Encrypt sensitive data
- Centralize logs
- Protect audit logs
- Maintain backups
- Test disaster recovery
- Maintain evidence
- Automate compliance checks
- Monitor continuously
- Review access regularly
- Remediate findings
- Track exceptions
- Keep documentation current

---

# Audit and Compliance Anti-Patterns

## No Traceability

Bad:

    Production
        |
        ↓
    Unknown Change

Better:

    Change
        |
        ↓
    Ticket
        |
        ↓
    Git
        |
        ↓
    Pipeline
        |
        ↓
    Deployment

---

# Anti-Pattern: Shared Accounts

Bad:

    Engineer A
        |
        ↓
    Shared Admin Account

    Engineer B
        |
        ↓
    Same Account

Problem:

    Difficult Attribution

Better:

    Individual Identity
        |
        ↓
    Role
        |
        ↓
    Required Permissions

---

# Anti-Pattern: Direct Production Access

Bad:

    Developer
        |
        ↓
    Production Server
        |
        ↓
    Manual Change

Better:

    Git
        |
        ↓
    Review
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

# Anti-Pattern: Secrets in Git

Bad:

    Git
        |
        ↓
    Password
        |
        ↓
    Repository

Better:

    Secure Secret Store
        |
        ↓
    Application

---

# Anti-Pattern: No Security Gates

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
    Security
        |
        ↓
    Approval
        |
        ↓
    Deploy

---

# Anti-Pattern: Manual Evidence Collection

Bad:

    Audit
        |
        ↓
    Search For Evidence
        |
        ↓
    Missing Records

Better:

    Process
        |
        ↓
    Automatic Evidence
        |
        ↓
    Central Repository

---

# Anti-Pattern: Compliance Only Before Audit

Bad:

    Audit Coming
        |
        ↓
    Start Collecting Evidence

Better:

    Continuous Compliance
        |
        ↓
    Continuous Evidence

---

# Anti-Pattern: Ignoring Findings

Bad:

    Finding
        |
        ↓
    Ignore
        |
        ↓
    Repeat Finding

Better:

    Finding
        |
        ↓
    Remediation
        |
        ↓
    Validation
        |
        ↓
    Closure

---

# Anti-Pattern: Permanent Exceptions

Bad:

    Exception
        |
        ↓
    Never Reviewed

Better:

    Exception
        |
        ↓
    Risk Assessment
        |
        ↓
    Approval
        |
        ↓
    Expiration
        |
        ↓
    Review

---

# Audit and Compliance Checklist

## Access

    IAM Reviewed
    Least Privilege
    Production Access Restricted
    Privileged Access Controlled
    Access Reviews Completed
    Former Users Removed

## Code

    Pull Requests Required
    Code Review Required
    Branch Protection Enabled
    Commit Traceability Available

## CI/CD

    Pipeline Logs Available
    Security Scans Available
    Quality Gates Available
    Approval Gates Configured
    Artifact Traceability Available

## Infrastructure

    Terraform Version Controlled
    Terraform State Protected
    Infrastructure Changes Reviewed
    Policy Checks Available
    Configuration Drift Monitored

## Kubernetes

    RBAC Configured
    Secrets Protected
    Images Scanned
    Production Access Restricted
    Configuration Changes Traceable

## Security

    SonarQube
    Trivy
    Veracode
    Vulnerability Remediation
    Secret Protection
    Encryption

## Operations

    Logs Available
    Monitoring Available
    Incident Records Available
    Change Records Available
    Rollback Process Available

## Recovery

    Backups Configured
    Restore Tested
    DR Tested
    RTO Defined
    RPO Defined

## Audit

    Evidence Available
    Findings Tracked
    Remediation Tracked
    Exceptions Documented
    Control Owners Defined

---

# Audit Interview Questions

## Basic

1. What is an audit?

2. What is compliance?

3. What is the difference between audit and compliance?

4. What is auditability?

5. What is traceability?

6. What is separation of duties?

7. What is least privilege?

8. What is audit evidence?

9. Why is change management important?

10. Why is logging important for compliance?

---

# Audit Interview Questions

## Intermediate

11. How do you make a CI/CD pipeline auditable?

12. How do you track production changes?

13. How does Git help with compliance?

14. How does Terraform help with auditability?

15. How do you secure production access?

16. How do you manage secrets?

17. How do you implement security gates?

18. How do you maintain deployment traceability?

19. How do you collect audit evidence?

20. How do you handle an audit finding?

---

# Audit Interview Questions

## Advanced

21. How would you design an enterprise DevSecOps compliance pipeline?

22. How would you implement separation of duties in CI/CD?

23. How would you prevent unauthorized production changes?

24. How would you implement continuous compliance?

25. How would you design auditability for Terraform?

26. How would you design compliance for EKS?

27. How would you implement policy as code?

28. How would you handle a compliance exception?

29. How would you design evidence collection?

30. How would you reduce repeat audit findings?

31. How would you integrate security and compliance into GitOps?

32. How would you prove that a production deployment was authorized?

---

# Scenario-Based Interview Question

## Prove Who Deployed To Production

Response:

    Deployment
        |
        ↓
    Pipeline
        |
        ↓
    Approver
        |
        ↓
    Git Commit
        |
        ↓
    Artifact
        |
        ↓
    Deployment Record

Use:

    User Identity
    Commit SHA
    Image Version
    Change ID
    Approval
    Timestamp

---

# Scenario-Based Interview Question

## Developer Modified Production Directly

Response:

    Detect
        |
        ↓
    Identify User
        |
        ↓
    Identify Change
        |
        ↓
    Assess Impact
        |
        ↓
    Restore Approved State
        |
        ↓
    Investigate
        |
        ↓
    Restrict Access
        |
        ↓
    Add Preventive Control

---

# Scenario-Based Interview Question

## Critical Vulnerability Found During CI

Response:

    Build
        |
        ↓
    Trivy
        |
        X
    Critical Vulnerability
        |
        ↓
    Security Gate
        |
        X
    Deployment Blocked
        |
        ↓
    Remediation
        |
        ↓
    Rescan
        |
        ✓
    Continue

---

# Scenario-Based Interview Question

## Audit Finds No Production Approval

Response:

    Finding
        |
        ↓
    Investigate
        |
        ↓
    Identify Root Cause
        |
        ↓
    Implement Approval Gate
        |
        ↓
    Test Control
        |
        ↓
    Collect Evidence
        |
        ↓
    Submit Remediation

---

# Scenario-Based Interview Question

## Terraform Creates Public Infrastructure

Response:

    Terraform Plan
        |
        ↓
    Policy Check
        |
        X
    Public Resource
        |
        ↓
    Pipeline Blocked
        |
        ↓
    Fix Configuration
        |
        ↓
    Re-run Policy Check
        |
        ✓
    Approval
        |
        ↓
    Apply

---

# Scenario-Based Interview Question

## Secret Appears in CI Logs

Response:

    Secret Exposure
        |
        ↓
    Stop Exposure
        |
        ↓
    Rotate Secret
        |
        ↓
    Review Logs
        |
        ↓
    Assess Impact
        |
        ↓
    Update Pipeline
        |
        ↓
    Validate
        |
        ↓
    Document Incident

---

# Scenario-Based Interview Question

## Audit Requests Deployment Evidence

Provide:

    Change ID
    Git Commit
    Pull Request
    Reviewer
    CI Results
    Security Results
    Artifact
    Approval
    Deployment Time
    Deployment Result

Complete trace:

    Requirement
        |
        ↓
    Change
        |
        ↓
    Code
        |
        ↓
    Review
        |
        ↓
    Security
        |
        ↓
    Artifact
        |
        ↓
    Approval
        |
        ↓
    Production

---

# Scenario-Based Interview Question

## Auditor Finds Repeated Security Issues

Response:

    Repeat Finding
        |
        ↓
    Root Cause Analysis
        |
        ↓
    Identify Systemic Gap
        |
        ↓
    Automate Control
        |
        ↓
    Add Security Gate
        |
        ↓
    Test
        |
        ↓
    Monitor
        |
        ↓
    Validate

---

# Scenario-Based Interview Question

## Production Access Is Too Broad

Response:

    Access Review
        |
        ↓
    Identify Excess Permissions
        |
        ↓
    Risk Assessment
        |
        ↓
    Reduce Permissions
        |
        ↓
    Validate Application
        |
        ↓
    Document
        |
        ↓
    Periodic Review

---

# Complete Enterprise Audit Flow

    Business Requirement
        |
        ↓
    Policy
        |
        ↓
    Control
        |
        ↓
    Implementation
        |
        ↓
    Git
        |
        ↓
    CI
        |
        +-- Build
        +-- Test
        +-- SonarQube
        +-- Trivy
        +-- Veracode
        |
        ↓
    Artifact
        |
        ↓
    Approval
        |
        ↓
    CD
        |
        ↓
    Production
        |
        ↓
    Monitoring
        |
        ↓
    Logs
        |
        ↓
    Evidence
        |
        ↓
    Audit
        |
        ↓
    Findings
        |
        ↓
    Remediation
        |
        ↓
    Validation
        |
        ↓
    Continuous Compliance

---

# Real-World DevSecOps Compliance Flow

    Developer
        |
        ↓
    GitHub
        |
        ↓
    Pull Request
        |
        ↓
    Code Review
        |
        ↓
    GitHub Actions
        |
        +-- Build
        +-- Unit Test
        +-- SonarQube
        +-- Trivy
        +-- Veracode
        |
        ↓
    Artifact
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    EKS
        |
        ↓
    ArgoCD
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
    Audit Evidence

---

# Real-World Infrastructure Compliance Flow

    Terraform Code
        |
        ↓
    Git
        |
        ↓
    Pull Request
        |
        ↓
    Review
        |
        ↓
    terraform plan
        |
        ↓
    Policy Check
        |
        ↓
    Approval
        |
        ↓
    terraform apply
        |
        ↓
    AWS
        |
        ↓
    Monitoring
        |
        ↓
    Audit Evidence

---

# Real-World Production Release Audit

    Requirement
        |
        ↓
    JIRA Change Request
        |
        ↓
    Git Commit
        |
        ↓
    Pull Request
        |
        ↓
    Code Review
        |
        ↓
    CI
        |
        ↓
    SonarQube
        |
        ↓
    Trivy
        |
        ↓
    Veracode
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
    Health Checks
        |
        ↓
    Monitoring
        |
        ↓
    Deployment Record
        |
        ↓
    Audit Evidence

---

# Final Audit and Compliance Mental Model

Remember:

    IDENTIFY
        |
        ↓
    CONTROL
        |
        ↓
    IMPLEMENT
        |
        ↓
    MONITOR
        |
        ↓
    EVIDENCE
        |
        ↓
    AUDIT
        |
        ↓
    REMEDIATE
        |
        ↓
    VALIDATE
        |
        ↓
    IMPROVE

The key DevOps compliance relationship is:

    Git
        +
    CI/CD
        +
    Security
        +
    IAM
        +
    Infrastructure as Code
        +
    Kubernetes Controls
        +
    Monitoring
        +
    Audit Evidence
        =
    Traceable And Controlled Delivery

---

# Final Concept

Audit and Compliance is not:

    "Prepare Documents When Auditors Arrive"

It is:

    "Build Systems And Processes
    That Continuously Demonstrate
    Security, Control, And Accountability."

A mature enterprise DevOps organization provides:

    Controlled Access
        +
    Reviewed Changes
        +
    Secure CI/CD
        +
    Immutable Artifacts
        +
    Infrastructure as Code
        +
    Security Gates
        +
    Protected Secrets
        +
    Centralized Logs
        +
    Monitoring
        +
    Backup
        +
    Disaster Recovery
        +
    Continuous Evidence

The most important principle is:

    If A Critical Action Happens,
    You Should Be Able To Explain:

    Who
    |
    ↓
    What
    |
    ↓
    When
    |
    ↓
    Why
    |
    ↓
    Approval
    |
    ↓
    Result

The complete enterprise compliance flow is:

    Requirement
        |
        ↓
    Policy
        |
        ↓
    Control
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
    Security
        |
        ↓
    Artifact
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Monitoring
        |
        ↓
    Evidence
        |
        ↓
    Audit
        |
        ↓
    Finding
        |
        ↓
    Remediation
        |
        ↓
    Validation
        |
        ↓
    Continuous Improvement