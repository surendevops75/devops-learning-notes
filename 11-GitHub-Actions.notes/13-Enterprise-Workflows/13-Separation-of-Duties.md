# Separation of Duties

Separation of Duties (SoD) is a security and operational control where critical responsibilities are divided among different people, teams, or systems.

The main goal is to ensure that one person does not have complete control over a sensitive process.

Separation of Duties helps prevent:

    Unauthorized Changes
    +
    Privilege Abuse
    +
    Fraud
    +
    Security Risks
    +
    Human Errors
    +
    Lack of Accountability

A typical Separation of Duties flow is:

    Request
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    Execution
        |
        ↓
    Validation
        |
        ↓
    Audit

---

# Why Separation of Duties Is Important

Without Separation of Duties:

    Developer
        |
        ↓
    Creates Code
        |
        ↓
    Approves Code
        |
        ↓
    Deploys Production
        |
        ↓
    Audits Own Change

This creates a significant security and compliance risk.

With Separation of Duties:

    Developer
        |
        ↓
    Creates Code
        |
        ↓
    Reviewer
        |
        ↓
    Reviews Code
        |
        ↓
    Release Manager
        |
        ↓
    Approves Release
        |
        ↓
    CI/CD
        |
        ↓
    Deploys
        |
        ↓
    Operations
        |
        ↓
    Monitors
        |
        ↓
    Audit
        |
        ↓
    Verifies

---

# Core Principle

The core principle is:

    No Single Person
    Should Control
    Every Critical Step
    Of A High-Risk Process

Critical activities may include:

    Code Creation
    Code Review
    Security Validation
    Infrastructure Changes
    Production Approval
    Production Deployment
    Access Management
    Audit

---

# Separation of Duties in DevOps

DevOps environments can implement Separation of Duties across:

    Development
    +
    Code Review
    +
    QA
    +
    Security
    +
    Release Management
    +
    Production
    +
    Operations
    +
    Audit

Example:

    Developer
        |
        ↓
    Git
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
    Security Checks
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Monitoring

---

# Developer and Reviewer Separation

A developer should not always be the person approving their own change.

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
    Reviewer
        |
        ↓
    Approval
        |
        ↓
    Merge

This provides independent review.

---

# Pull Request Approval

Pull Requests can enforce:

    Required Review
    Required Approvals
    Required Status Checks
    Branch Protection
    Restricted Direct Push

Example:

    Feature Branch
        |
        ↓
    Pull Request
        |
        ↓
    CI Checks
        |
        ↓
    Reviewer
        |
        ↓
    Approval
        |
        ↓
    Main Branch

---

# Branch Protection

Protected branches help enforce Separation of Duties.

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
    Required Reviewer
        |
        ↓
    Required CI
        |
        ↓
    Main Branch

Direct pushes can be restricted.

---

# Developer and Production Separation

A developer should not necessarily have unrestricted production deployment access.

Preferred:

    Developer
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
    CI
        |
        ↓
    Approval
        |
        ↓
    Production
        |
        ↓
    Deployment

---

# Production Approval

Production deployments may require an independent approval.

Example:

    Developer
        |
        ↓
    Code
        |
        ↓
    CI
        |
        ↓
    Artifact
        |
        ↓
    Production Approval
        |
        ↓
    Deployment

The person creating the change should not automatically control the production approval.

---

# CI/CD Separation of Duties

A CI/CD pipeline can separate:

    Code Creation
        |
        ↓
    Code Review
        |
        ↓
    Build
        |
        ↓
    Testing
        |
        ↓
    Security
        |
        ↓
    Approval
        |
        ↓
    Deployment

This provides controlled software delivery.

---

# Build and Deployment Separation

Build and deployment can be separated.

Example:

    Developer
        |
        ↓
    Build
        |
        ↓
    Artifact
        |
        ↓
    Security Scan
        |
        ↓
    Approval
        |
        ↓
    Deployment

The artifact is created first and deployed only after required controls pass.

---

# Security and Development Separation

Security validation should not depend only on the developer.

Example:

    Developer
        |
        ↓
    Code
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
    Security Gate
        |
        ↓
    Deployment

Automated security controls provide independent validation.

---

# QA and Development Separation

QA provides independent validation of application functionality.

Example:

    Developer
        |
        ↓
    Build
        |
        ↓
    QA
        |
        ↓
    Testing
        |
        ↓
    Validation
        |
        ↓
    Release

---

# Security Team and Development

Security teams may review high-risk changes.

Example:

    Developer
        |
        ↓
    Change
        |
        ↓
    Security Scan
        |
        ↓
    Security Review
        |
        ↓
    Release Approval
        |
        ↓
    Production

The exact approval process depends on organizational policy.

---

# Operations and Development Separation

Development and production operations may have separate responsibilities.

Example:

    Development
        |
        ↓
    Application Code

    Operations
        |
        ↓
    Production Platform

This reduces the risk of one person controlling both application changes and production infrastructure.

---

# Infrastructure Separation

Infrastructure changes should follow controlled review and approval.

Example:

    Engineer
        |
        ↓
    Terraform Change
        |
        ↓
    Pull Request
        |
        ↓
    Infrastructure Reviewer
        |
        ↓
    Approval
        |
        ↓
    Terraform Apply

---

# Terraform Separation of Duties

A production Terraform workflow can be:

    Engineer
        |
        ↓
    Terraform Code
        |
        ↓
    Pull Request
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
    terraform apply

This provides:

    Change Control
    +
    Review
    +
    Approval
    +
    Traceability

---

# Terraform Plan Review

Terraform plan should be reviewed before applying significant production changes.

Example:

    Terraform Code
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
    terraform apply

Review should consider:

    Resources Added
    Resources Changed
    Resources Destroyed
    Security Impact
    Network Impact
    Cost Impact
    Availability Impact

---

# IAM Separation of Duties

IAM administration is highly privileged.

Avoid:

    Developer
        |
        ↓
    IAM Administration
        |
        ↓
    Production Access
        |
        ↓
    Application Changes
        |
        ↓
    Audit

Preferred:

    Developer
        |
        ↓
    Application Access

    IAM Administrator
        |
        ↓
    Identity Management

    Audit
        |
        ↓
    Access Review

---

# Least Privilege

Separation of Duties works together with least privilege.

Example:

    Developer
        |
        ↓
    Required Permissions

    CI/CD
        |
        ↓
    Deployment Permissions

    Security Team
        |
        ↓
    Security Permissions

    Administrator
        |
        ↓
    Administrative Permissions

Avoid:

    Everyone
        |
        ↓
    Administrator Access

---

# Production Access

Production access should be restricted.

Example:

    Engineer
        |
        ↓
    Access Request
        |
        ↓
    Approval
        |
        ↓
    Required Access
        |
        ↓
    Activity
        |
        ↓
    Audit Log

---

# Temporary Production Access

Temporary access can reduce unnecessary permanent privileges.

Example:

    Access Request
        |
        ↓
    Approval
        |
        ↓
    Temporary Access
        |
        ↓
    Production Activity
        |
        ↓
    Access Removed

---

# Privileged Access

Privileged access may include:

    IAM Administration
    Production Access
    Kubernetes Administration
    Database Administration
    Network Administration
    Security Administration

Such access should be:

    Restricted
    Approved
    Monitored
    Audited
    Periodically Reviewed

---

# Break-Glass Access

Break-glass access is emergency privileged access.

Example:

    Critical Incident
        |
        ↓
    Emergency Access
        |
        ↓
    Production Action
        |
        ↓
    Recovery
        |
        ↓
    Review
        |
        ↓
    Audit

Break-glass access should not become permanent unrestricted access.

---

# Emergency Change Separation

Emergency changes still require accountability.

Example:

    Critical Incident
        |
        ↓
    Emergency Approval
        |
        ↓
    Engineer
        |
        ↓
    Production Change
        |
        ↓
    Validation
        |
        ↓
    Post-Incident Review

---

# GitOps and Separation of Duties

GitOps provides a strong Separation of Duties model.

Example:

    Developer
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
    Merge
        |
        ↓
    ArgoCD
        |
        ↓
    Kubernetes

The developer does not need direct production cluster access for normal deployments.

---

# ArgoCD and Separation of Duties

ArgoCD can act as the deployment mechanism.

Flow:

    Developer
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
    Merge
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

Benefits:

    Reduced Direct Production Access
    +
    Git-Based Traceability
    +
    Controlled Deployment
    +
    Consistent State

---

# Kubernetes RBAC

Kubernetes RBAC can separate responsibilities.

Example:

    Developer
        |
        ↓
    Limited Namespace Permissions

    Deployment System
        |
        ↓
    Deployment Permissions

    Cluster Administrator
        |
        ↓
    Administrative Permissions

Avoid giving every user:

    cluster-admin

---

# Developer Kubernetes Access

A developer may need permissions such as:

    Get Pods
    List Pods
    View Logs
    Describe Resources

They may not need:

    Cluster Administration
    RBAC Administration
    Node Administration

Permissions should match responsibilities.

---

# CI/CD Service Account

CI/CD should use a dedicated identity.

Example:

    GitHub Actions
        |
        ↓
    Deployment Identity
        |
        ↓
    Required Permissions
        |
        ↓
    AWS / Kubernetes

Avoid:

    GitHub Actions
        |
        ↓
    Full Administrator

---

# Service Account Separation

Different workloads should use appropriate identities.

Example:

    CI Service
        |
        ↓
    CI Role

    Deployment Service
        |
        ↓
    Deployment Role

    Application
        |
        ↓
    Application Role

This improves access isolation.

---

# Production Database Access

Developers should not automatically have unrestricted production database access.

Preferred:

    Application
        |
        ↓
    Database

    DBA / Authorized Operator
        |
        ↓
    Controlled Administrative Access

---

# Database Change Separation

Production database changes should follow a controlled process.

Example:

    Developer
        |
        ↓
    Migration Code
        |
        ↓
    Code Review
        |
        ↓
    QA Testing
        |
        ↓
    DBA / Authorized Review
        |
        ↓
    Approval
        |
        ↓
    Production

---

# Release Manager

A Release Manager may be responsible for:

    Release Coordination
    Change Validation
    Production Approval
    Deployment Window
    Rollback Coordination

This responsibility can be separated from development.

---

# Production Deployment Roles

A typical enterprise model:

    Developer
        |
        ↓
    Creates Code

    Reviewer
        |
        ↓
    Reviews Code

    QA
        |
        ↓
    Tests Application

    Security
        |
        ↓
    Validates Security

    Release Manager
        |
        ↓
    Approves Release

    CI/CD
        |
        ↓
    Deploys

    Operations
        |
        ↓
    Monitors

    Audit
        |
        ↓
    Verifies

---

# Responsibility Matrix

Example:

    Activity                  Owner

    Code Development          Developer

    Code Review               Reviewer

    Testing                   QA

    Security Validation      Security

    Production Approval      Release Manager

    Deployment                CI/CD

    Monitoring               Operations

    Audit                     Compliance / Audit

The exact responsibilities depend on the organization.

---

# RACI and Separation of Duties

RACI can help define responsibilities.

    R = Responsible
    A = Accountable
    C = Consulted
    I = Informed

Example:

    Code Development
        |
        ↓
    Developer = R

    Code Review
        |
        ↓
    Reviewer = R

    Security Validation
        |
        ↓
    Security = R

    Production Approval
        |
        ↓
    Release Manager = A

    Deployment
        |
        ↓
    CI/CD = R

---

# Production Deployment RACI

Example:

    Activity                  Developer   QA   Security   Release

    Code                      R           I      C         I

    Code Review               R           I      C         I

    Testing                   C           R      I         I

    Security Scan             I           C      R         I

    Production Approval       I           C      C         A

    Deployment                I           I      I         A

The exact RACI should be defined by the organization.

---

# Separation of Duties in CI

Example:

    Developer
        |
        ↓
    Commit
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

Automated checks provide independent validation.

---

# Separation of Duties in CD

Example:

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
        |
        ↓
    Validation

---

# Environment Separation

Different environments can have different access controls.

Example:

    Development
        |
        ↓
    Developer Access

    QA
        |
        ↓
    QA Access

    SIT
        |
        ↓
    Controlled Access

    UAT
        |
        ↓
    Business Validation

    Production
        |
        ↓
    Restricted Access

---

# Build Once Deploy Many

The same artifact can be promoted across environments.

Example:

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

This improves:

    Traceability
    +
    Consistency
    +
    Separation
    +
    Auditability

---

# Immutable Artifact and Separation of Duties

Example:

    Developer
        |
        ↓
    Build
        |
        ↓
    Artifact
        |
        ↓
    Security Scan
        |
        ↓
    Approval
        |
        ↓
    Production

The approved artifact should not be modified before deployment.

---

# Security Gates

Security gates provide automated control.

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
    Veracode
        |
        ↓
    Security Gate
        |
        +------ Pass
        |
        +------ Fail

If the security gate fails:

    Deployment Blocked

---

# Quality Gates

Example:

    Code
        |
        ↓
    SonarQube
        |
        ↓
    Quality Gate
        |
        +------ Pass
        |
        +------ Fail

A quality gate provides an automated validation point before deployment.

---

# Change Management and Separation of Duties

A production change may involve:

    Requester
        |
        ↓
    Implementer
        |
        ↓
    Reviewer
        |
        ↓
    Approver
        |
        ↓
    Executor
        |
        ↓
    Validator

A single person should not necessarily perform every role for high-risk changes.

---

# Production Change Example

    Requester
        |
        ↓
    Change Request
        |
        ↓
    Implementer
        |
        ↓
    Developer
        |
        ↓
    Reviewer
        |
        ↓
    Release Approver
        |
        ↓
    CI/CD
        |
        ↓
    Production
        |
        ↓
    Validator

---

# Four-Eyes Principle

The four-eyes principle means that a sensitive action requires review or approval by another authorized person.

Example:

    High-Risk Change
        |
        ↓
    Engineer
        |
        ↓
    Reviewer
        |
        ↓
    Approval
        |
        ↓
    Execution

This is useful for:

    Production Changes
    IAM Changes
    Database Changes
    Security Changes
    Infrastructure Changes

---

# Two-Person Approval

High-risk changes may require multiple approvals.

Example:

    Change
        |
        ↓
    Reviewer 1
        |
        ↓
    Reviewer 2
        |
        ↓
    Release Approval
        |
        ↓
    Production

The required number of approvals depends on organizational policy.

---

# Risk-Based Separation of Duties

Not every change needs the same level of control.

Example:

    Low Risk
        |
        ↓
    Standard Review

    Medium Risk
        |
        ↓
    Additional Review

    High Risk
        |
        ↓
    Multiple Approvals

Controls should be proportional to risk.

---

# High-Risk Changes

Examples:

    IAM Changes
    Network Changes
    Production Database Changes
    Authentication Changes
    Security Configuration
    Production Infrastructure
    Cluster Configuration

These may require stronger approval controls.

---

# Low-Risk Changes

Examples:

    Documentation
    Non-Production Configuration
    Minor Application Changes

These may require fewer controls depending on organizational policy.

---

# Separation of Duties and Audit

Audit should independently verify that controls are working.

Example:

    Production Change
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Evidence
        |
        ↓
    Auditor
        |
        ↓
    Control Validation

The person executing the change should not be the only person validating compliance.

---

# Audit Evidence

Evidence may include:

    Pull Request
    Reviewer
    Approval
    Commit SHA
    Pipeline Run
    Security Scan
    Artifact
    Change Ticket
    Deployment Record
    Timestamp

---

# Complete Audit Trail

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
    Reviewer
        |
        ↓
    CI
        |
        ↓
    Security
        |
        ↓
    Approver
        |
        ↓
    Deployment
        |
        ↓
    Audit Log

---

# Traceability

Every critical production action should answer:

    Who?

    What?

    When?

    Why?

    Who Reviewed?

    Who Approved?

    What Was Deployed?

    What Was The Result?

---

# Production Deployment Traceability

Example:

    Developer
        |
        ↓
    Git Commit
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
    Security Checks
        |
        ↓
    Artifact
        |
        ↓
    Production Approval
        |
        ↓
    Deployment
        |
        ↓
    Monitoring
        |
        ↓
    Audit

---

# Separation of Duties and GitOps

GitOps can remove direct deployment responsibility from developers.

Example:

    Developer
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
    Merge
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

This creates:

    Code Separation
    +
    Review Separation
    +
    Deployment Separation

---

# Separation of Duties and Terraform

Terraform can provide controlled infrastructure changes.

Example:

    Engineer
        |
        ↓
    Terraform Code
        |
        ↓
    Pull Request
        |
        ↓
    terraform plan
        |
        ↓
    Infrastructure Review
        |
        ↓
    Approval
        |
        ↓
    terraform apply
        |
        ↓
    AWS

---

# Separation of Duties and Kubernetes

Kubernetes RBAC can separate:

    Developer
        |
        ↓
    Application Permissions

    Deployment System
        |
        ↓
    Deployment Permissions

    Cluster Administrator
        |
        ↓
    Cluster Permissions

This prevents unnecessary administrative access.

---

# Separation of Duties and AWS

AWS IAM can separate:

    Developer Role
        |
        ↓
    Development Permissions

    Deployment Role
        |
        ↓
    Deployment Permissions

    Security Role
        |
        ↓
    Security Permissions

    Administrator Role
        |
        ↓
    Administrative Permissions

---

# Separation of Duties and Secrets

Secrets should not be controlled by every participant in the delivery process.

Example:

    Developer
        |
        ↓
    Application Code
        |
        X
    Secret

    Secure Secret Store
        |
        ↓
    Controlled Access
        |
        ↓
    Application

---

# Separation of Duties and Production Database

Example:

    Developer
        |
        ↓
    Migration Code

    Reviewer
        |
        ↓
    Review

    DBA
        |
        ↓
    Production Approval

    CI/CD
        |
        ↓
    Controlled Execution

This reduces the risk of unauthorized database changes.

---

# Scenario: Developer Has Production Access

Situation:

    Developer
        |
        ↓
    Production Access

Response:

    Review Why Access Is Required
        |
        ↓
    Identify Required Permissions
        |
        ↓
    Remove Unnecessary Permissions
        |
        ↓
    Implement CI/CD / GitOps
        |
        ↓
    Monitor Remaining Access
        |
        ↓
    Review Periodically

---

# Scenario: Developer Approves Own Pull Request

Problem:

    Developer
        |
        ↓
    Creates Pull Request
        |
        ↓
    Same Developer
        |
        ↓
    Approves Pull Request

Solution:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    Independent Reviewer
        |
        ↓
    Approval
        |
        ↓
    Merge

---

# Scenario: Developer Deploys Directly to Production

Problem:

    Developer
        |
        ↓
    kubectl
        |
        ↓
    Production

Risks:

    No Review
    No Approval
    Reduced Traceability
    Configuration Drift

Better:

    Developer
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
    CI/CD / ArgoCD
        |
        ↓
    Production

---

# Scenario: Terraform Production Change

Problem:

    Engineer
        |
        ↓
    terraform apply
        |
        ↓
    Production

Better:

    Engineer
        |
        ↓
    Terraform Change
        |
        ↓
    Pull Request
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
    terraform apply

---

# Scenario: CI/CD Has Administrator Access

Problem:

    CI/CD
        |
        ↓
    Administrator
        |
        ↓
    All AWS Resources

Risk:

    Excessive Privileges
    +
    Large Blast Radius

Better:

    CI/CD
        |
        ↓
    Deployment Role
        |
        ↓
    Required Permissions

---

# Scenario: Emergency Production Fix

Situation:

    Critical Production Incident

Flow:

    Incident
        |
        ↓
    Emergency Change
        |
        ↓
    Authorized Approval
        |
        ↓
    Engineer
        |
        ↓
    Production Fix
        |
        ↓
    Validation
        |
        ↓
    Post-Incident Review
        |
        ↓
    Audit Evidence

---

# Scenario: Critical Security Vulnerability

Flow:

    Security Scan
        |
        ↓
    Critical Vulnerability
        |
        ↓
    Deployment Blocked
        |
        ↓
    Developer
        |
        ↓
    Remediation
        |
        ↓
    Rebuild
        |
        ↓
    Security Rescan
        |
        ↓
    Approval
        |
        ↓
    Deployment

---

# Scenario: Production IAM Change

Flow:

    IAM Change
        |
        ↓
    Pull Request
        |
        ↓
    IAM / Security Review
        |
        ↓
    Approval
        |
        ↓
    Apply
        |
        ↓
    Audit

---

# Scenario: Unauthorized Production Change

Flow:

    Unauthorized Change
        |
        ↓
    Detection
        |
        ↓
    Identify User
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
    Prevent Recurrence

---

# Separation of Duties Best Practices

- Separate code creation and approval
- Require independent code reviews
- Protect main branches
- Restrict direct production access
- Use least privilege
- Separate development and production permissions
- Use CI/CD for controlled deployments
- Use GitOps for Kubernetes deployments
- Require production approval where appropriate
- Review Terraform changes
- Use Kubernetes RBAC
- Use AWS IAM roles
- Restrict privileged access
- Use temporary access where appropriate
- Protect service accounts
- Use security gates
- Maintain deployment traceability
- Use immutable artifacts
- Maintain audit logs
- Review privileged access regularly
- Document emergency changes
- Use risk-based approval
- Periodically test controls

---

# Separation of Duties Anti-Patterns

## Single Person Controls Everything

Bad:

    Developer
        |
        ↓
    Code
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    Production
        |
        ↓
    Audit

Better:

    Developer
        |
        ↓
    Code

    Reviewer
        |
        ↓
    Review

    Approver
        |
        ↓
    Approval

    CI/CD
        |
        ↓
    Deployment

    Auditor
        |
        ↓
    Validation

---

# Anti-Pattern: Shared Administrator Account

Bad:

    Multiple Engineers
        |
        ↓
    Shared Administrator Account

Problems:

    Poor Attribution
    Excessive Access
    Difficult Auditing

Better:

    Individual Identity
        |
        ↓
    Role
        |
        ↓
    Required Permissions

---

# Anti-Pattern: Everyone Has Production Access

Bad:

    Developer A
    Developer B
    Developer C
    Developer D
        |
        ↓
    Production Administrator

Better:

    Developer
        |
        ↓
    Limited Access

    Authorized Operator
        |
        ↓
    Production Access

---

# Anti-Pattern: CI/CD Has Full Administrator Access

Bad:

    CI/CD
        |
        ↓
    Full AWS Administrator

Better:

    CI/CD
        |
        ↓
    Deployment Role
        |
        ↓
    Required Permissions Only

---

# Anti-Pattern: Self-Approval

Bad:

    Developer
        |
        ↓
    Change
        |
        ↓
    Same Developer
        |
        ↓
    Approval

Better:

    Developer
        |
        ↓
    Change
        |
        ↓
    Independent Reviewer
        |
        ↓
    Approval

---

# Anti-Pattern: Manual Production Changes

Bad:

    SSH
        |
        ↓
    Production
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

# Anti-Pattern: Permanent Break-Glass Access

Bad:

    Emergency Access
        |
        ↓
    Permanent Access

Better:

    Emergency Access
        |
        ↓
    Temporary
        |
        ↓
    Logged
        |
        ↓
    Reviewed
        |
        ↓
    Removed

---

# Anti-Pattern: No Audit Trail

Bad:

    Change
        |
        ↓
    Production
        |
        ↓
    No Record

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
    Approval
        |
        ↓
    Pipeline
        |
        ↓
    Deployment Record

---

# Separation of Duties Checklist

## Development

    Developer Cannot Bypass Required Review
    Pull Requests Used
    Code Review Required
    Main Branch Protected

## CI/CD

    Pipeline Automated
    Security Gates Configured
    Quality Gates Configured
    Production Approval Configured
    Deployment Logs Available

## Production

    Direct Access Restricted
    Least Privilege Applied
    Privileged Access Controlled
    Emergency Access Audited

## Infrastructure

    Terraform Changes Reviewed
    Production Apply Controlled
    IAM Changes Reviewed
    Network Changes Reviewed

## Kubernetes

    RBAC Configured
    Developer Permissions Restricted
    Deployment Identity Controlled
    Cluster Administrator Access Restricted

## Security

    SonarQube
    Trivy
    Veracode
    Security Review
    Vulnerability Gates

## Audit

    Change Records
    Approval Records
    Deployment Records
    Access Logs
    Evidence Retained

---

# Separation of Duties Interview Questions

## Basic

1. What is Separation of Duties?

2. Why is Separation of Duties important?

3. What is the four-eyes principle?

4. What is least privilege?

5. Why should developers not always have production access?

6. Why is independent code review important?

7. What is self-approval?

8. What is privileged access?

9. What is break-glass access?

10. How does Separation of Duties improve security?

---

# Intermediate

11. How would you implement Separation of Duties in CI/CD?

12. How would you prevent developers from deploying directly to production?

13. How would you implement production approval?

14. How does branch protection help with Separation of Duties?

15. How would you implement Separation of Duties for Terraform?

16. How would you implement Separation of Duties in Kubernetes?

17. How would you separate developer and production access?

18. How would you control IAM administration?

19. How would you implement Separation of Duties for database changes?

20. How would you audit production changes?

---

# Advanced

21. How would you design enterprise-wide Separation of Duties?

22. How would you implement risk-based approval?

23. How would you prevent self-approval?

24. How would you design a production deployment workflow that satisfies Separation of Duties?

25. How would you implement Separation of Duties using GitOps?

26. How would you prevent CI/CD from having excessive permissions?

27. How would you design temporary production access?

28. How would you handle emergency changes while maintaining Separation of Duties?

29. How would you prove Separation of Duties compliance during an audit?

30. How would you handle a situation where the same engineer is both developer and production operator?

---

# Interview Scenario

## Developer Has Direct Production Access

Answer:

    First, I would identify why direct access is required.

    Then I would review the existing permissions and determine
    whether the access follows least privilege.

    I would move routine deployments toward CI/CD or GitOps,
    restrict unnecessary production access, and implement
    approval controls for high-risk operations.

    I would also ensure that production activities are logged
    and periodically reviewed.

Final model:

    Developer
        |
        ↓
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

# Interview Scenario

## Developer Approves Own Pull Request

Answer:

    I would enforce branch protection and require approval
    from another authorized reviewer.

    I would configure repository controls to prevent
    self-approval where organizational policy requires
    independent review.

Flow:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    Independent Reviewer
        |
        ↓
    Approval
        |
        ↓
    Merge

---

# Interview Scenario

## Production Terraform Change

Answer:

    I would keep Terraform code in Git and require a Pull Request.

    The CI pipeline would run terraform plan and relevant
    validation checks.

    An authorized reviewer would review the plan.

    After approval, the controlled deployment process would
    execute terraform apply.

Flow:

    Terraform Code
        |
        ↓
    Pull Request
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
    terraform apply

---

# Interview Scenario

## Emergency Production Change

Answer:

    I would use the organization's emergency change process.

    The change should have an authorized emergency approval,
    the action should be logged, and the result should be
    validated.

    After recovery, I would perform retrospective review and
    document the change and evidence.

Flow:

    Incident
        |
        ↓
    Emergency Approval
        |
        ↓
    Change
        |
        ↓
    Validation
        |
        ↓
    Post-Incident Review
        |
        ↓
    Audit Evidence

---

# Interview Scenario

## CI/CD Requires Administrator Permissions

Answer:

    I would identify the actual permissions required by the
    deployment process and replace broad administrator access
    with a dedicated least-privilege deployment role.

    I would review and test the permissions, monitor the role,
    and periodically review whether the permissions are still
    required.

Flow:

    CI/CD
        |
        ↓
    Deployment Role
        |
        ↓
    Required Permissions
        |
        ↓
    AWS Resources

---

# Interview Scenario

## Auditor Asks How Production Deployment Is Controlled

Answer:

    I would demonstrate the complete traceability chain:

    Developer
        |
        ↓
    Git Commit
        |
        ↓
    Pull Request
        |
        ↓
    Independent Review
        |
        ↓
    CI
        |
        ↓
    Security Checks
        |
        ↓
    Artifact
        |
        ↓
    Production Approval
        |
        ↓
    CI/CD or ArgoCD
        |
        ↓
    Production
        |
        ↓
    Deployment Record

This demonstrates:

    Who
    +
    What
    +
    Review
    +
    Approval
    +
    Artifact
    +
    Deployment
    +
    Result

---

# Complete Enterprise Separation of Duties Flow

    Developer
        |
        ↓
    Feature Branch
        |
        ↓
    Pull Request
        |
        ↓
    Code Reviewer
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
    QA
        |
        ↓
    SIT
        |
        ↓
    UAT
        |
        ↓
    Release Manager
        |
        ↓
    Production Approval
        |
        ↓
    CI/CD / ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Health Checks
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
    Audit

---

# Complete Role Separation Model

    Developer
        |
        ↓
    Creates Change

    Reviewer
        |
        ↓
    Reviews Change

    QA
        |
        ↓
    Validates Functionality

    Security
        |
        ↓
    Validates Security

    Release Manager
        |
        ↓
    Approves Release

    CI/CD
        |
        ↓
    Executes Deployment

    Operations
        |
        ↓
    Monitors System

    Audit
        |
        ↓
    Verifies Controls

---

# Final Separation of Duties Mental Model

Remember:

    CREATE
        |
        ↓
    REVIEW
        |
        ↓
    VALIDATE
        |
        ↓
    APPROVE
        |
        ↓
    EXECUTE
        |
        ↓
    MONITOR
        |
        ↓
    AUDIT

The goal is not to create unnecessary bureaucracy.

The goal is to ensure that high-risk actions have:

    Independent Review
        +
    Controlled Approval
        +
    Limited Privileges
        +
    Automated Enforcement
        +
    Traceability
        +
    Accountability

---

# Final Concept

Separation of Duties in DevOps means:

    The Developer
    Does Not Automatically
    Control Everything.

Instead:

    Developer
        |
        ↓
    Creates Change

    Reviewer
        |
        ↓
    Reviews Change

    Security
        |
        ↓
    Validates Security

    Release Manager
        |
        ↓
    Approves Release

    CI/CD
        |
        ↓
    Deploys

    Operations
        |
        ↓
    Monitors

    Audit
        |
        ↓
    Verifies

This provides a controlled, secure, and auditable delivery process.