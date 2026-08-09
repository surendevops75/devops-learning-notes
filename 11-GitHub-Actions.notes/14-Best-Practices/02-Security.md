# Security

Security is a core part of DevOps and should be integrated throughout the software delivery lifecycle.

Modern DevOps security follows:

    Plan
        |
        ↓
    Code
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
    Secure
        |
        ↓
    Deploy
        |
        ↓
    Monitor
        |
        ↓
    Improve

The goal is:

    Secure Code
        +
    Secure Infrastructure
        +
    Secure CI/CD
        +
    Secure Cloud
        +
    Secure Kubernetes
        +
    Secure Secrets
        +
    Continuous Monitoring

---

# 1. DevSecOps

DevSecOps integrates security into DevOps instead of treating security as a final production-stage activity.

Traditional approach:

    Development
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
    Security Review

DevSecOps approach:

    Development
        |
        ↓
    Code Security
        |
        ↓
    Build
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
    DAST
        |
        ↓
    Deploy
        |
        ↓
    Monitor

---

# 2. Shift Left Security

Shift Left means identifying security issues as early as possible.

Example:

    Developer
        |
        ↓
    Code
        |
        ↓
    Security Checks
        |
        ↓
    Build
        |
        ↓
    Deployment

Instead of:

    Developer
        |
        ↓
    Code
        |
        ↓
    Build
        |
        ↓
    Deployment
        |
        ↓
    Security Issue

Benefits:

    Earlier Detection
    +
    Lower Remediation Cost
    +
    Faster Feedback
    +
    Reduced Production Risk

---

# 3. Security Throughout SDLC

Security should exist across:

    Planning
    Coding
    Building
    Testing
    Deployment
    Operations

Example:

    Plan
        |
        ↓
    Threat Modeling
        |
        ↓
    Code
        |
        ↓
    SAST
        |
        ↓
    Build
        |
        ↓
    Dependency Scan
        |
        ↓
    Container Scan
        |
        ↓
    Deploy
        |
        ↓
    Runtime Monitoring

---

# 4. Secure Coding

Developers should follow secure coding practices.

Common areas:

    Input Validation
    Authentication
    Authorization
    Error Handling
    Secure APIs
    Dependency Management
    Secret Handling
    Logging

---

# 5. Input Validation

Never blindly trust user input.

Example:

    User Input
        |
        ↓
    Validation
        |
        ↓
    Sanitization
        |
        ↓
    Application

Validation should consider:

    Type
    Length
    Format
    Range
    Allowed Values

---

# 6. Prevent Injection Attacks

Applications should protect against injection attacks.

Examples:

    SQL Injection
    Command Injection
    LDAP Injection
    Template Injection

Preferred:

    User Input
        |
        ↓
    Validation
        |
        ↓
    Parameterized Queries
        |
        ↓
    Application

---

# 7. Secure Authentication

Authentication verifies:

    Who Is The User?

Common controls:

    Strong Password Policies
    MFA
    Token-Based Authentication
    Session Management
    Account Lockout
    Secure Credential Storage

---

# 8. Secure Authorization

Authorization verifies:

    What Is The User Allowed To Do?

Example:

    User
        |
        ↓
    Authentication
        |
        ↓
    Authorization
        |
        ↓
    Resource

Avoid assuming that authentication automatically means authorization.

---

# 9. Least Privilege

Give users and services only the permissions they need.

Example:

    Application
        |
        ↓
    IAM Role
        |
        ↓
    Required Permissions

Avoid:

    Application
        |
        ↓
    Administrator Access

---

# 10. Identity and Access Management

IAM controls:

    Users
    Roles
    Permissions
    Policies
    Service Identities

Example:

    User
        |
        ↓
    Identity
        |
        ↓
    Role
        |
        ↓
    Permissions
        |
        ↓
    Resource

---

# 11. Avoid Long-Lived Credentials

Long-lived credentials increase security risk.

Avoid:

    Access Key
        |
        ↓
    Stored Permanently
        |
        ↓
    Application

Prefer:

    Workload
        |
        ↓
    IAM Role
        |
        ↓
    Temporary Credentials
        |
        ↓
    AWS Resource

---

# 12. Protect Access Keys

If access keys must exist:

    Store Securely
    +
    Restrict Permissions
    +
    Rotate
    +
    Monitor
    +
    Remove When Unused

Never commit credentials to Git.

---

# 13. Never Store Secrets in Git

Never intentionally store:

    Passwords
    API Keys
    Access Keys
    Private Keys
    Database Credentials
    Tokens
    Certificates

inside source code repositories.

Bad:

    application.properties

    DB_PASSWORD=password123

Better:

    Secure Secret Management
        |
        ↓
    Application
        |
        ↓
    Runtime Secret

---

# 14. Environment Variables

Environment variables can be used for runtime configuration.

Example:

    Application
        |
        ↓
    Environment Variable
        |
        ↓
    Configuration

However, environment variables should not automatically be considered secure secret storage.

Use appropriate secret-management mechanisms for sensitive values.

---

# 15. Secret Management

A secure secret flow is:

    Secret
        |
        ↓
    Secret Store
        |
        ↓
    Controlled Access
        |
        ↓
    Application
        |
        ↓
    Runtime

Secrets should not be exposed unnecessarily in:

    Source Code
    Logs
    Pipeline Output
    Docker Images
    Public Configuration

---

# 16. Secret Rotation

Secrets should be rotated according to organizational security requirements.

Example:

    Existing Secret
        |
        ↓
    Rotation
        |
        ↓
    New Secret
        |
        ↓
    Application
        |
        ↓
    Validation

---

# 17. Secret Exposure in Logs

Bad:

    echo "DB_PASSWORD=$DB_PASSWORD"

This may expose credentials in CI/CD logs.

Better:

    Avoid Printing Secrets

Sensitive values should be masked where supported.

---

# 18. Secret Exposure in Docker Images

Bad:

    Dockerfile
        |
        ↓
    COPY secrets.txt
        |
        ↓
    Docker Image

The secret may remain inside image layers.

Better:

    Secret Store
        |
        ↓
    Runtime Injection
        |
        ↓
    Application

---

# 19. Secure GitHub Actions

GitHub Actions workflows should protect:

    Secrets
    Permissions
    Tokens
    Actions
    Runners
    Artifacts
    Environments

Use:

    Minimal Permissions
    +
    Protected Environments
    +
    Secret Management
    +
    Review Controls

---

# 20. GitHub Actions GITHUB_TOKEN Permissions

Avoid granting unnecessary permissions.

Example:

    permissions:
      contents: read

Only request additional permissions when required.

---

# 21. Secure GitHub Actions Secrets

Use repository, organization, or environment secrets where appropriate.

Example:

    GitHub Actions
        |
        ↓
    Secret
        |
        ↓
    Workflow
        |
        ↓
    Deployment

Avoid hardcoding credentials in YAML.

---

# 22. Protect Production Environments

Production environments can require:

    Approval
    Restricted Access
    Deployment Protection
    Environment Secrets

Example:

    GitHub Actions
        |
        ↓
    Production Environment
        |
        ↓
    Approval
        |
        ↓
    Deployment

---

# 23. Secure Third-Party Actions

Do not blindly trust every third-party action.

Review:

    Source Repository
    Maintainer
    Version
    Permissions
    Security History
    Required Credentials

---

# 24. Pin GitHub Actions Versions

Avoid uncontrolled action changes.

Example:

    uses: actions/checkout@v4

For higher supply-chain control, organizations may pin actions to immutable references such as commit SHAs.

---

# 25. Avoid Excessive Workflow Permissions

Bad:

    permissions:
      write-all

Better:

    Grant Only Required Permissions

Example:

    permissions:
      contents: read

---

# 26. Protect Self-Hosted Runners

Self-hosted runners require strong security controls.

Risks include:

    Malicious Workflows
    Credential Theft
    Network Access
    Persistent Files
    Privilege Escalation

---

# 27. Isolate Self-Hosted Runners

Use appropriate:

    Network Segmentation
    Runner Groups
    Access Controls
    Ephemeral Runners
    Monitoring

Avoid exposing sensitive production networks unnecessarily.

---

# 28. Clean Runner Workspaces

Sensitive files may remain on persistent runners.

Example:

    Job
        |
        ↓
    Credentials
        |
        ↓
    Temporary Files
        |
        ↓
    Job Complete

Clean sensitive data after execution.

Ephemeral runners can reduce persistence risk.

---

# 29. Protect CI/CD Credentials

CI/CD systems can access:

    Cloud
    Kubernetes
    Container Registry
    Artifact Repository
    Git Repositories

Treat CI/CD credentials as highly sensitive.

---

# 30. Container Security

Container security should cover:

    Base Images
    Dependencies
    Dockerfile
    Image Configuration
    Registry
    Runtime

Flow:

    Source
        |
        ↓
    Docker Build
        |
        ↓
    Image Scan
        |
        ↓
    Registry
        |
        ↓
    Deployment
        |
        ↓
    Runtime

---

# 31. Scan Container Images

Use tools such as:

    Trivy

Example:

    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Vulnerability Scan
        |
        ↓
    Security Gate
        |
        ↓
    Registry

---

# 32. Container Vulnerability Severity

Vulnerabilities can be categorized by severity.

Common levels:

    Critical
    High
    Medium
    Low

Organizations should define policies for which severities block deployments.

---

# 33. Fail the Pipeline on Critical Vulnerabilities

Example:

    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Critical Vulnerability
        |
        X
    Pipeline Failed

This prevents known high-risk images from progressing.

---

# 34. Scan Base Images

Example:

    Base Image
        |
        ↓
    Vulnerability Scan
        |
        ↓
    Approved
        |
        ↓
    Application Image

Do not assume an official image is automatically vulnerability-free.

---

# 35. Use Minimal Base Images

Smaller images can reduce:

    Packages
    Attack Surface
    Image Size
    Vulnerability Count

Example:

    Large Base Image
        |
        ↓
    Many Packages
        |
        ↓
    Larger Attack Surface

Better:

    Appropriate Minimal Base Image
        |
        ↓
    Required Runtime
        |
        ↓
    Smaller Attack Surface

---

# 36. Multi-Stage Docker Builds

Example:

    Build Stage
        |
        ↓
    Compile
        |
        ↓
    Runtime Stage
        |
        ↓
    Application

Benefits:

    Smaller Image
    +
    Fewer Build Tools
    +
    Reduced Attack Surface

---

# 37. Do Not Run Containers as Root

Bad:

    Container
        |
        ↓
    root

Better:

    Container
        |
        ↓
    Non-Root User

This reduces the impact of container compromise.

---

# 38. Read-Only Filesystems

Where appropriate:

    Container
        |
        ↓
    Read-Only Filesystem

This can reduce the ability of an attacker to modify files.

---

# 39. Drop Unnecessary Linux Capabilities

Containers should receive only required capabilities.

Avoid unnecessary privileged capabilities.

Example:

    Container
        |
        ↓
    Required Capabilities Only

---

# 40. Avoid Privileged Containers

Avoid:

    --privileged

unless there is a justified requirement.

Privileged containers can significantly increase host-level risk.

---

# 41. Kubernetes Security

Kubernetes security should cover:

    RBAC
    Network Policies
    Secrets
    Pod Security
    Image Security
    Service Accounts
    Resource Controls
    Admission Controls

---

# 42. Kubernetes RBAC

RBAC controls who can perform actions.

Example:

    User
        |
        ↓
    Role
        |
        ↓
    Permissions
        |
        ↓
    Kubernetes Resource

---

# 43. Avoid cluster-admin

Do not give developers or workloads unnecessary:

    cluster-admin

permissions.

Use:

    Namespace
        |
        ↓
    Role
        |
        ↓
    Required Permissions

---

# 44. Use Service Accounts

Applications should use appropriate service accounts.

Example:

    Pod
        |
        ↓
    Service Account
        |
        ↓
    Required Permissions

Avoid sharing highly privileged service accounts.

---

# 45. Disable Unnecessary Service Account Token Mounting

If an application does not need Kubernetes API access, avoid exposing unnecessary service account credentials.

Concept:

    Application
        |
        X
    Kubernetes API Credentials

Only provide access when required.

---

# 46. Kubernetes Secrets

Kubernetes Secrets should be handled carefully.

Remember:

    Secret Object
        |
        ↓
    Access Control
        +
    Encryption
        +
    Secure Delivery

Do not assume that simply creating a Kubernetes Secret makes sensitive information fully secure.

---

# 47. Encrypt Sensitive Data

Sensitive data should be protected:

    At Rest
        +
    In Transit

Examples:

    Database Encryption
    Storage Encryption
    TLS
    Encrypted Volumes

---

# 48. TLS

Use TLS for sensitive communication.

Example:

    Client
        |
        ↓
    HTTPS / TLS
        |
        ↓
    Application

TLS protects data in transit.

---

# 49. Private Networking

Keep internal services private when public access is unnecessary.

Example:

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

# 50. Security Groups

Use restrictive security groups.

Example:

    ALB
        |
        ↓
    Application
        |
        ↓
    Database

Only required traffic should be allowed.

---

# 51. Network Segmentation

Separate:

    Public
    +
    Application
    +
    Database

Example:

    Public Subnet
        |
        ↓
    Application Subnet
        |
        ↓
    Database Subnet

---

# 52. Kubernetes Network Policies

Network Policies can restrict pod-to-pod communication.

Example:

    Frontend
        |
        ↓
    Backend
        |
        ↓
    Database

Instead of:

    Every Pod
        |
        ↓
    Every Pod

Allow only required communication.

---

# 53. API Security

APIs should implement:

    Authentication
    Authorization
    Input Validation
    Rate Limiting
    TLS
    Secure Error Handling

---

# 54. Rate Limiting

Rate limiting can reduce:

    Abuse
    Brute Force
    Resource Exhaustion

Example:

    Client
        |
        ↓
    Rate Limiter
        |
        ↓
    API

---

# 55. Secure Error Messages

Avoid exposing sensitive internal information.

Bad:

    Database password:
    xyz123

Better:

    Internal Server Error

Detailed information should remain in controlled logs.

---

# 56. Dependency Security

Applications depend on:

    Libraries
    Packages
    Frameworks
    Modules

These dependencies can contain vulnerabilities.

---

# 57. Software Composition Analysis

SCA identifies vulnerable dependencies.

Example:

    Application
        |
        ↓
    Dependencies
        |
        ↓
    SCA
        |
        ↓
    Vulnerability
        |
        ↓
    Remediation

---

# 58. Keep Dependencies Updated

Security updates should be evaluated regularly.

Flow:

    Dependency
        |
        ↓
    New Version
        |
        ↓
    Test
        |
        ↓
    Security Scan
        |
        ↓
    Deploy

---

# 59. Remove Unused Dependencies

Unused dependencies increase:

    Attack Surface
    Maintenance
    Vulnerability Exposure

Remove dependencies that are no longer required.

---

# 60. Software Supply Chain Security

Modern applications depend on:

    Source Code
    +
    Dependencies
    +
    Build Tools
    +
    Base Images
    +
    CI/CD
    +
    Artifacts

Each component can introduce risk.

---

# 61. Supply Chain Security Flow

    Developer
        |
        ↓
    Source Code
        |
        ↓
    Dependencies
        |
        ↓
    Build
        |
        ↓
    Security Scan
        |
        ↓
    Artifact
        |
        ↓
    Registry
        |
        ↓
    Deployment

Each stage should have appropriate controls.

---

# 62. Artifact Integrity

Artifacts should be traceable.

Example:

    Git Commit
        |
        ↓
    Build
        |
        ↓
    Artifact
        |
        ↓
    SHA / Digest
        |
        ↓
    Registry
        |
        ↓
    Deployment

---

# 63. Use Image Digests

Instead of relying only on mutable tags:

    app:latest

use immutable image references where appropriate:

    app@sha256:<digest>

This improves artifact integrity and traceability.

---

# 64. Artifact Repository Security

Protect:

    ECR
    Artifactory
    Other Registries

Controls may include:

    Authentication
    Authorization
    Image Scanning
    Retention
    Encryption
    Access Logging

---

# 65. Protect ECR

For AWS ECR:

    Developer
        |
        ↓
    CI/CD
        |
        ↓
    ECR
        |
        ↓
    EKS

Grant only required push and pull permissions.

---

# 66. Artifact Promotion

Preferred:

    Build
        |
        ↓
    Scan
        |
        ↓
    QA
        |
        ↓
    Approval
        |
        ↓
    Production

Use the same approved artifact.

---

# 67. Terraform Security

Terraform manages sensitive infrastructure.

Security considerations:

    State
    +
    IAM
    +
    Secrets
    +
    Providers
    +
    Backend
    +
    Permissions

---

# 68. Protect Terraform State

Terraform state may contain sensitive infrastructure information.

Use:

    Secure Backend
    +
    Access Control
    +
    Encryption
    +
    Versioning
    +
    Locking

Do not expose state publicly.

---

# 69. Terraform IAM

Terraform execution should use an appropriate role.

Example:

    GitHub Actions
        |
        ↓
    Terraform Role
        |
        ↓
    Required AWS Permissions

Avoid unrestricted administrator permissions unless explicitly justified.

---

# 70. Terraform Secret Handling

Avoid hardcoding:

    Passwords
    API Keys
    Access Keys

inside Terraform files.

Use appropriate secret-management mechanisms.

---

# 71. Security Scanning in CI

A DevSecOps pipeline may contain:

    Checkout
        |
        ↓
    Build
        |
        ↓
    Unit Test
        |
        ↓
    SonarQube
        |
        ↓
    Dependency Scan
        |
        ↓
    Trivy
        |
        ↓
    Veracode
        |
        ↓
    Quality / Security Gates
        |
        ↓
    Artifact

---

# 72. SAST

Static Application Security Testing analyzes source code.

Example:

    Source Code
        |
        ↓
    SAST
        |
        ↓
    Security Findings
        |
        ↓
    Fix
        |
        ↓
    Rescan

---

# 73. DAST

Dynamic Application Security Testing evaluates a running application.

Example:

    Application
        |
        ↓
    Running Environment
        |
        ↓
    DAST
        |
        ↓
    Security Findings

---

# 74. SCA

Software Composition Analysis identifies vulnerable dependencies.

Example:

    Application
        |
        ↓
    Dependencies
        |
        ↓
    SCA
        |
        ↓
    Vulnerability Report

---

# 75. Container Scanning

Example:

    Docker Image
        |
        ↓
    Trivy
        |
        ↓
    Vulnerability Report
        |
        ↓
    Security Gate

---

# 76. Secret Scanning

Secret scanning identifies accidentally committed credentials.

Example:

    Git Commit
        |
        ↓
    Secret Scanner
        |
        +------ Secret Found
        |          |
        |          ↓
        |      Block Commit
        |
        +------ No Secret
                   |
                   ↓
                Continue

---

# 77. Git History Scanning

Deleting a secret from the latest commit does not necessarily remove it from Git history.

Example:

    Secret
        |
        ↓
    Commit
        |
        ↓
    Secret Removed
        |
        ↓
    Old Commit Still Contains Secret

If a credential is exposed:

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

---

# 78. Rotate Compromised Credentials

If a credential is exposed:

    Detection
        |
        ↓
    Revoke Credential
        |
        ↓
    Generate New Credential
        |
        ↓
    Update Application
        |
        ↓
    Validate
        |
        ↓
    Investigate

Do not simply delete the visible line and assume the problem is solved.

---

# 79. Security Gates

Security gates prevent unsafe changes from progressing.

Example:

    Build
        |
        ↓
    Security Scan
        |
        ↓
    Gate
        |
        +------ Pass
        |         |
        |         ↓
        |      Deploy
        |
        +------ Fail
                  |
                  ↓
              Stop Pipeline

---

# 80. Quality Gates

Quality gates can enforce:

    Code Quality
    Security Issues
    Coverage
    Maintainability

Example:

    SonarQube
        |
        ↓
    Quality Gate
        |
        +------ Pass
        |
        +------ Fail

---

# 81. Security Policy as Code

Security controls can be automated.

Example:

    Infrastructure
        |
        ↓
    Policy Check
        |
        ↓
    Pass / Fail
        |
        ↓
    Deployment

This reduces manual security review for repeatable controls.

---

# 82. Policy Enforcement

Policies can validate:

    Public Resources
    Encryption
    IAM
    Network Rules
    Container Configuration
    Kubernetes Configuration

---

# 83. Prevent Public Storage

Storage containing sensitive data should not be unintentionally public.

Example:

    Storage
        |
        ↓
    Access Policy
        |
        ↓
    Public?
        |
        +------ Yes → Block
        |
        +------ No → Continue

---

# 84. Encryption at Rest

Examples:

    EBS
    RDS
    S3
    ECR
    Databases
    Backups

Encryption should be enabled according to organizational requirements.

---

# 85. Encryption in Transit

Examples:

    HTTPS
    TLS
    Secure Database Connections
    Secure API Communication

Flow:

    Client
        |
        ↓
    TLS
        |
        ↓
    Service

---

# 86. Logging and Security

Security events should be observable.

Examples:

    Authentication Failure
    Authorization Failure
    Privilege Changes
    IAM Changes
    Production Access
    Deployment Events

---

# 87. Centralized Security Logs

Example:

    Application
        |
        ↓
    Logs
        |
        ↓
    Centralized Logging
        |
        ↓
    ELK
        |
        ↓
    Security Investigation

---

# 88. Avoid Logging Sensitive Information

Do not log:

    Passwords
    Tokens
    API Keys
    Private Keys
    Sensitive Personal Data

Example:

    Bad:
    Authorization: Bearer eyJ...

Better:

    Authorization: [REDACTED]

---

# 89. Security Monitoring

Monitor:

    Authentication
    Authorization
    Privilege Changes
    Network Activity
    Application Errors
    Vulnerabilities
    Deployment Activity

---

# 90. Vulnerability Management

A vulnerability management lifecycle is:

    Discover
        |
        ↓
    Assess
        |
        ↓
    Prioritize
        |
        ↓
    Remediate
        |
        ↓
    Validate
        |
        ↓
    Monitor

---

# 91. Prioritize Vulnerabilities

Not every vulnerability has the same risk.

Consider:

    Severity
    +
    Exploitability
    +
    Exposure
    +
    Asset Criticality
    +
    Business Impact

---

# 92. Critical Vulnerability Handling

Example:

    Critical Vulnerability
        |
        ↓
    Assess
        |
        ↓
    Immediate Remediation
        |
        ↓
    Rescan
        |
        ↓
    Validate
        |
        ↓
    Release

---

# 93. Security Incident Response

A security incident flow:

    Detection
        |
        ↓
    Investigation
        |
        ↓
    Containment
        |
        ↓
    Eradication
        |
        ↓
    Recovery
        |
        ↓
    Lessons Learned

---

# 94. Compromised Credential Response

Example:

    Credential Compromised
        |
        ↓
    Revoke
        |
        ↓
    Rotate
        |
        ↓
    Investigate Usage
        |
        ↓
    Check Logs
        |
        ↓
    Validate
        |
        ↓
    Prevent Recurrence

---

# 95. Compromised Container Response

Example:

    Suspicious Container
        |
        ↓
    Isolate
        |
        ↓
    Stop / Replace
        |
        ↓
    Investigate
        |
        ↓
    Scan Image
        |
        ↓
    Rebuild From Trusted Source
        |
        ↓
    Redeploy

---

# 96. Compromised Kubernetes Pod

Example:

    Suspicious Pod
        |
        ↓
    Isolate
        |
        ↓
    Investigate
        |
        ↓
    Check Logs
        |
        ↓
    Check Service Account
        |
        ↓
    Check Network Activity
        |
        ↓
    Replace Pod
        |
        ↓
    Remediate Root Cause

---

# 97. Security and Monitoring

Security should be continuously monitored.

Example:

    Application
        |
        ↓
    Prometheus
        |
        ↓
    Metrics

    Application
        |
        ↓
    ELK
        |
        ↓
    Logs

    Dashboards
        |
        ↓
    Grafana

---

# 98. Security Alerts

Examples:

    High Error Rate
    Unauthorized Access
    Authentication Failures
    Unexpected Privilege Changes
    Critical Vulnerabilities
    Suspicious Network Activity

Alerts should be:

    Actionable
    Meaningful
    Prioritized

---

# 99. Security Runbooks

A security alert should have an appropriate response procedure.

Example:

    Alert
        |
        ↓
    Runbook
        |
        ↓
    Investigation
        |
        ↓
    Containment
        |
        ↓
    Recovery

---

# 100. Security and Disaster Recovery

Security incidents may require recovery.

Example:

    Security Incident
        |
        ↓
    Containment
        |
        ↓
    Clean Environment
        |
        ↓
    Restore
        |
        ↓
    Validate
        |
        ↓
    Monitor

---

# 101. Backup Security

Backups should be protected against:

    Unauthorized Access
    Accidental Deletion
    Data Corruption
    Ransomware
    Credential Compromise

---

# 102. Test Backup Restoration

Example:

    Backup
        |
        ↓
    Restore
        |
        ↓
    Validate
        |
        ↓
    Security Check

A backup that has never been restored should not automatically be considered reliable.

---

# 103. Security and Disaster Recovery

DR should consider:

    Identity
    Secrets
    Encryption Keys
    Network
    Infrastructure
    Applications
    Databases
    Monitoring

---

# 104. Secure Infrastructure as Code

IaC should be scanned before deployment.

Example:

    Terraform
        |
        ↓
    Security Policy Scan
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    Apply

---

# 105. Secure Terraform Configuration

Review for:

    Public Resources
    Open Security Groups
    Excessive IAM
    Missing Encryption
    Public Databases
    Unrestricted Network Access

---

# 106. Avoid Open Security Groups

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

# 107. Restrict Database Access

Avoid:

    Internet
        |
        ↓
    Database

Better:

    Application
        |
        ↓
    Database

Only required application or administrative paths should have access.

---

# 108. Secure ALB Architecture

Example:

    Internet
        |
        ↓
    ALB
        |
        ↓
    Private Application
        |
        ↓
    Private Database

Use TLS and appropriate security controls.

---

# 109. Secure EKS Architecture

Example:

    Internet
        |
        ↓
    ALB
        |
        ↓
    EKS
        |
        ↓
    Private Services
        |
        ↓
    RDS

Security controls:

    IAM
    RBAC
    Network Policies
    Security Groups
    Private Networking
    Image Scanning
    Secrets Management

---

# 110. Secure CI/CD Architecture

    Developer
        |
        ↓
    Git
        |
        ↓
    Pull Request
        |
        ↓
    GitHub Actions
        |
        +-- SAST
        +-- SCA
        +-- Trivy
        +-- Veracode
        |
        ↓
    Security Gate
        |
        ↓
    Artifact
        |
        ↓
    Approval
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        ↓
    Monitoring

---

# 111. Security and Separation of Duties

Security responsibilities should be appropriately separated.

Example:

    Developer
        |
        ↓
    Creates Code

    Reviewer
        |
        ↓
    Reviews Code

    Security
        |
        ↓
    Validates Security

    Release Manager
        |
        ↓
    Approves Production

    CI/CD
        |
        ↓
    Deploys

    Audit
        |
        ↓
    Verifies

---

# 112. Security and Change Management

Security-sensitive changes should have:

    Request
    +
    Risk Assessment
    +
    Review
    +
    Approval
    +
    Implementation
    +
    Validation
    +
    Audit

---

# 113. Security Documentation

Document:

    Security Architecture
    Access Model
    Secret Management
    Incident Response
    Vulnerability Management
    Backup
    Disaster Recovery
    Security Controls

---

# 114. Security Runbooks

Useful security runbooks include:

    Credential Compromise
    Container Vulnerability
    Unauthorized Access
    Production Breach
    Critical Vulnerability
    Secret Exposure
    Suspicious Pod
    IAM Misconfiguration

---

# 115. Security Testing

Security testing can include:

    SAST
    SCA
    DAST
    Container Scanning
    Secret Scanning
    IaC Scanning
    Penetration Testing
    Configuration Review

---

# 116. Security Testing Pipeline

    Source
        |
        ↓
    SAST
        |
        ↓
    Dependencies
        |
        ↓
    SCA
        |
        ↓
    Docker Image
        |
        ↓
    Trivy
        |
        ↓
    Running Application
        |
        ↓
    DAST
        |
        ↓
    Security Gate

---

# 117. Security Gates in CI/CD

Example:

    Build
        |
        ↓
    SonarQube
        |
        ↓
    SCA
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
    Artifact
        |
        ↓
    Deployment

---

# 118. Do Not Ignore Security Findings

Bad:

    Security Scan
        |
        ↓
    Finding
        |
        ↓
    Ignore
        |
        ↓
    Production

Better:

    Finding
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
    Approve

---

# 119. Exception Management

Sometimes a vulnerability cannot immediately be fixed.

Example:

    Vulnerability
        |
        ↓
    Risk Assessment
        |
        ↓
    Exception
        |
        ↓
    Compensating Control
        |
        ↓
    Expiration Date
        |
        ↓
    Remediation

Exceptions should be documented and approved.

---

# 120. Security Metrics

Useful security metrics include:

    Critical Vulnerabilities
    High Vulnerabilities
    Mean Time To Remediate
    Failed Security Gates
    Secret Exposure Events
    Security Incidents
    Patch Compliance
    Privileged Access Reviews

---

# 121. Mean Time To Remediate

MTTR for security vulnerabilities measures how quickly vulnerabilities are resolved.

Example:

    Vulnerability Detected
        |
        ↓
    Remediation
        |
        ↓
    Validation

Goal:

    Reduce Remediation Time

---

# 122. Security Awareness

Developers and operations teams should understand:

    Secure Coding
    Secret Handling
    IAM
    Phishing
    Dependency Risks
    Container Security
    Cloud Security
    Incident Response

---

# 123. Security Review

Security reviews should consider:

    Application
    Infrastructure
    Network
    IAM
    CI/CD
    Containers
    Kubernetes
    Data
    Monitoring

---

# 124. Secure-by-Default

Systems should start with secure defaults.

Examples:

    Private Resources
    Encryption Enabled
    Least Privilege
    Authentication Required
    Secure Protocols
    Restricted Network Access

---

# 125. Defense in Depth

Do not rely on one security control.

Example:

    Authentication
        +
    Authorization
        +
    Network Security
        +
    Encryption
        +
    Image Scanning
        +
    Runtime Monitoring
        +
    Incident Response

If one control fails, additional controls remain.

---

# 126. Zero Trust Principles

Zero Trust generally emphasizes:

    Verify
        +
    Least Privilege
        +
    Continuous Validation
        +
    Assume Breach

Do not automatically trust a user or service simply because it is inside a network.

---

# 127. Secure Service-to-Service Communication

Example:

    Service A
        |
        ↓
    Authentication
        |
        ↓
    Authorization
        |
        ↓
    Service B

Use appropriate authentication and encrypted communication.

---

# 128. Secure Kubernetes Service Communication

Example:

    Frontend
        |
        ↓
    Backend Service
        |
        ↓
    Database

Restrict unnecessary communication using:

    Network Policies
    Authentication
    Authorization

---

# 129. Security and Observability

Security events should be observable.

Example:

    Security Event
        |
        ↓
    Logs
        |
        ↓
    ELK
        |
        ↓
    Investigation

Application metrics can also help detect abnormal behavior.

---

# 130. Security and Prometheus

Prometheus can help monitor operational indicators related to security.

Examples:

    Authentication Failure Rate
    Error Rate
    Request Rate
    Resource Usage

Prometheus is primarily a metrics system and should be combined with appropriate security logging and detection mechanisms.

---

# 131. Security and Grafana

Grafana can provide dashboards for:

    Application Health
    Infrastructure Health
    Security-Related Metrics
    Vulnerability Trends

---

# 132. Security and ELK

ELK can help centralize:

    Application Logs
    Access Logs
    Authentication Logs
    Security Events
    Infrastructure Logs

Example:

    Applications
        |
        ↓
    Logs
        |
        ↓
    ELK
        |
        ↓
    Search
        |
        ↓
    Investigation

---

# 133. Security Incident Flow

Complete flow:

    Detection
        |
        ↓
    Alert
        |
        ↓
    Investigation
        |
        ↓
    Containment
        |
        ↓
    Eradication
        |
        ↓
    Recovery
        |
        ↓
    Validation
        |
        ↓
    Post-Incident Review

---

# 134. Security Incident Example

Situation:

    Compromised Application Credential

Flow:

    Detection
        |
        ↓
    Credential Revoked
        |
        ↓
    New Credential Generated
        |
        ↓
    Application Updated
        |
        ↓
    Logs Investigated
        |
        ↓
    Unauthorized Activity Checked
        |
        ↓
    Monitoring Increased
        |
        ↓
    Root Cause Addressed

---

# 135. Security Best Practices

- Use least privilege
- Protect secrets
- Never commit credentials
- Rotate compromised credentials
- Use MFA where appropriate
- Restrict production access
- Protect CI/CD systems
- Minimize GitHub Actions permissions
- Review third-party actions
- Scan dependencies
- Scan container images
- Use SonarQube
- Use Trivy
- Use Veracode
- Use security gates
- Protect Terraform state
- Secure Kubernetes RBAC
- Avoid cluster-admin
- Use network policies
- Encrypt sensitive data
- Use TLS
- Restrict public resources
- Monitor security events
- Centralize logs
- Maintain incident response procedures
- Test backups
- Review privileged access
- Maintain audit trails
- Patch vulnerabilities
- Use defense in depth
- Practice continuous security improvement

---

# Security Anti-Patterns

## Secrets in Git

Bad:

    Git Repository
        |
        ↓
    Password
        |
        ↓
    Production

Better:

    Secret Store
        |
        ↓
    Controlled Access
        |
        ↓
    Application

---

# Anti-Pattern: Administrator Everywhere

Bad:

    Developer
    CI/CD
    Application
    Tester
        |
        ↓
    Administrator

Better:

    Developer
        |
        ↓
    Developer Role

    CI/CD
        |
        ↓
    Deployment Role

    Application
        |
        ↓
    Application Role

---

# Anti-Pattern: Public Database

Bad:

    Internet
        |
        ↓
    Database

Better:

    Application
        |
        ↓
    Private Database

---

# Anti-Pattern: Privileged Container

Bad:

    Application
        |
        ↓
    Privileged Container
        |
        ↓
    Host

Better:

    Application
        |
        ↓
    Non-Root Container
        |
        ↓
    Restricted Permissions

---

# Anti-Pattern: Unscanned Images

Bad:

    Docker Build
        |
        ↓
    ECR
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
    ECR
        |
        ↓
    Production

---

# Anti-Pattern: No Secret Rotation

Bad:

    Secret
        |
        ↓
    Never Changed
        |
        ↓
    Long-Term Exposure

Better:

    Secret
        |
        ↓
    Rotation
        |
        ↓
    Validation

---

# Anti-Pattern: Direct Production Access

Bad:

    Developer
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

# Anti-Pattern: Ignore Critical Vulnerabilities

Bad:

    Critical Vulnerability
        |
        ↓
    Ignore
        |
        ↓
    Production

Better:

    Critical Vulnerability
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

# Anti-Pattern: Excessive Network Access

Bad:

    Every Service
        |
        ↓
    Every Service

Better:

    Service A
        |
        ↓
    Required Service B

    Service B
        |
        ↓
    Required Database

---

# Anti-Pattern: No Audit Trail

Bad:

    Production Change
        |
        ↓
    No Record

Better:

    Production Change
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

# Security Checklist

## Application

    Secure Coding
    Input Validation
    Authentication
    Authorization
    Secure Error Handling
    Dependency Management

## Source Control

    Branch Protection
    Pull Requests
    Secret Scanning
    Access Control
    Audit Trail

## CI/CD

    Least Privilege
    Secret Protection
    SAST
    SCA
    Container Scanning
    Security Gates
    Protected Environments

## Containers

    Trusted Base Images
    Minimal Images
    Non-Root User
    Image Scanning
    No Privileged Containers
    No Secrets in Images

## Kubernetes

    RBAC
    Service Accounts
    Network Policies
    Restricted Permissions
    Secure Secrets
    Pod Security
    Image Security

## AWS

    IAM
    Least Privilege
    Private Networking
    Security Groups
    Encryption
    Secure ECR
    Secure S3
    Secure RDS

## Infrastructure

    Terraform Security
    State Protection
    IAM Review
    Network Review
    Encryption
    Security Scanning

## Monitoring

    Prometheus
    Grafana
    ELK
    Security Logs
    Alerts
    Runbooks

## Incident Response

    Detection
    Investigation
    Containment
    Eradication
    Recovery
    Post-Incident Review

---

# DevSecOps Security Architecture

    Developer
        |
        ↓
    Git
        |
        ↓
    Pull Request
        |
        ↓
    GitHub Actions
        |
        +-- SAST
        +-- SCA
        +-- Secret Scan
        +-- SonarQube
        +-- Trivy
        +-- Veracode
        |
        ↓
    Security Gate
        |
        ↓
    Docker Image
        |
        ↓
    ECR
        |
        ↓
    Approval
        |
        ↓
    ArgoCD
        |
        ↓
    EKS
        |
        +-- RBAC
        +-- Network Policies
        +-- Service Accounts
        +-- Pod Security
        |
        ↓
    ALB
        |
        ↓
    Application
        |
        ↓
    RDS
        |
        ↓
    Monitoring
        |
        +-- Prometheus
        +-- Grafana
        +-- ELK

---

# Complete Security Lifecycle

    PLAN
        |
        ↓
    Threat Model
        |
        ↓
    CODE
        |
        ↓
    Secure Coding
        |
        ↓
    BUILD
        |
        ↓
    SAST / SCA
        |
        ↓
    CONTAINER
        |
        ↓
    Trivy
        |
        ↓
    SECURITY GATE
        |
        ↓
    ARTIFACT
        |
        ↓
    APPROVAL
        |
        ↓
    DEPLOY
        |
        ↓
    EKS
        |
        ↓
    MONITOR
        |
        ↓
    DETECT
        |
        ↓
    RESPOND
        |
        ↓
    IMPROVE

---

# Security Interview Questions

## Basic

1. What is DevSecOps?

2. What is Shift Left Security?

3. What is least privilege?

4. What is authentication?

5. What is authorization?

6. Why should secrets not be stored in Git?

7. What is SAST?

8. What is DAST?

9. What is SCA?

10. What is container scanning?

11. What is RBAC?

12. What is IAM?

13. What is encryption at rest?

14. What is encryption in transit?

15. What is network segmentation?

---

# Intermediate

16. How would you secure a CI/CD pipeline?

17. How would you secure GitHub Actions?

18. How would you protect GitHub Actions secrets?

19. How would you prevent developers from accessing production directly?

20. How would you secure Docker images?

21. How would you secure Kubernetes?

22. How would you implement least privilege in AWS?

23. How would you secure Terraform?

24. How would you secure Terraform state?

25. How would you handle a leaked AWS access key?

26. How would you handle a secret committed to Git?

27. How would you implement container security?

28. How would you implement security gates?

29. How would you secure EKS?

30. How would you restrict Kubernetes permissions?

---

# Advanced

31. How would you design an enterprise DevSecOps pipeline?

32. How would you implement defense in depth?

33. How would you design a secure GitHub Actions architecture?

34. How would you secure self-hosted GitHub Actions runners?

35. How would you implement supply-chain security?

36. How would you secure container images from source to production?

37. How would you implement security for a multi-account AWS environment?

38. How would you design secure EKS networking?

39. How would you implement zero-trust principles?

40. How would you design incident response for compromised credentials?

41. How would you handle a critical vulnerability discovered in production?

42. How would you secure a Terraform-based infrastructure pipeline?

43. How would you implement security controls without slowing CI/CD significantly?

44. How would you design a secure production deployment process?

45. How would you prove security compliance during an audit?

---

# Interview Scenario

## Secret Accidentally Committed to Git

Answer:

    First, I would treat the secret as compromised.

    I would immediately revoke or rotate the credential.

    Then I would investigate whether the credential was used
    and review relevant logs.

    I would remove the secret from the repository and history
    where appropriate, but I would not consider history cleanup
    a replacement for credential rotation.

    Finally, I would identify why the secret was committed and
    implement secret scanning or additional controls to prevent
    recurrence.

Flow:

    Secret Detected
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
    Remove Exposure
        |
        ↓
    Add Preventive Control

---

# Interview Scenario

## Critical Vulnerability in Docker Image

Answer:

    I would first identify the affected image and determine
    the severity and exposure.

    If organizational policy requires blocking critical
    vulnerabilities, I would stop the deployment.

    Then I would update the affected dependency or base image,
    rebuild the image, run Trivy again, and validate the result.

Flow:

    Vulnerability
        |
        ↓
    Assess
        |
        ↓
    Block
        |
        ↓
    Remediate
        |
        ↓
    Rebuild
        |
        ↓
    Trivy
        |
        ↓
    Validate
        |
        ↓
    Deploy

---

# Interview Scenario

## Developer Has cluster-admin

Answer:

    I would first identify why cluster-admin is required.

    I would replace the permission with a namespace-scoped role
    containing only the required permissions where possible.

    I would review existing access, remove unnecessary
    privileges, and monitor privileged operations.

Flow:

    cluster-admin
        |
        ↓
    Access Review
        |
        ↓
    Least Privilege
        |
        ↓
    Role / RoleBinding
        |
        ↓
    Required Permissions

---

# Interview Scenario

## CI/CD Has Full AWS Administrator Access

Answer:

    I would identify all AWS operations performed by the pipeline.

    Then I would create a dedicated IAM role containing only
    the permissions required for those operations.

    I would test the pipeline with the restricted role and
    monitor its activity.

Flow:

    CI/CD
        |
        ↓
    Required Actions
        |
        ↓
    IAM Policy
        |
        ↓
    Deployment Role
        |
        ↓
    AWS Resources

---

# Interview Scenario

## Production Database Is Publicly Accessible

Answer:

    I would first determine whether public access is actually
    required.

    If it is not required, I would move the database to private
    networking and restrict access through appropriate security
    controls.

    I would also review credentials, encryption, logs, and
    existing network rules.

Flow:

    Internet
        |
        X
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

# Interview Scenario

## Suspicious Kubernetes Pod

Answer:

    I would isolate the affected workload where appropriate,
    investigate logs and activity, review the service account
    permissions, inspect network activity, and determine the
    root cause.

    After containment, I would replace the compromised workload
    from a trusted image and remediate the underlying issue.

Flow:

    Suspicious Pod
        |
        ↓
    Isolate
        |
        ↓
    Investigate
        |
        ↓
    Check RBAC
        |
        ↓
    Check Network
        |
        ↓
    Replace
        |
        ↓
    Remediate
        |
        ↓
    Monitor

---

# Final Security Mental Model

Remember:

    IDENTIFY
        |
        ↓
    PROTECT
        |
        ↓
    DETECT
        |
        ↓
    RESPOND
        |
        ↓
    RECOVER
        |
        ↓
    IMPROVE

Security is not a single pipeline stage.

It is a continuous process across:

    Code
        +
    CI/CD
        +
    Infrastructure
        +
    Cloud
        +
    Containers
        +
    Kubernetes
        +
    Identity
        +
    Secrets
        +
    Monitoring
        +
    Incident Response

---

# Final Concept

DevSecOps security means:

    Security Is Built In
        +
    Not Added At The End.

The goal is to create a delivery platform where:

    Code Is Secure
        +
    Dependencies Are Scanned
        +
    Containers Are Scanned
        +
    Secrets Are Protected
        +
    Infrastructure Is Secured
        +
    Access Is Restricted
        +
    Deployments Are Controlled
        +
    Production Is Monitored
        +
    Incidents Are Recoverable

The ultimate goal is:

    Secure Delivery
        +
    Fast Feedback
        +
    Least Privilege
        +
    Reduced Attack Surface
        +
    Continuous Detection
        +
    Reliable Recovery