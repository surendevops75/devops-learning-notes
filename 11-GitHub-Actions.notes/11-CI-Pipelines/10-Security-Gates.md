# Security Gates in GitHub Actions

A security gate is a controlled checkpoint in a CI/CD pipeline that determines whether a build, artifact, or deployment is allowed to continue based on defined security requirements.

The basic principle is:

    Code
      |
      ↓
    Build
      |
      ↓
    Security Checks
      |
      ↓
    Security Gate
      |
      +-- PASS → Continue
      |
      +-- FAIL → Stop
      |
      ↓
    Publish / Deploy

A security gate prevents known security issues from automatically progressing to later environments.

---

# Why Security Gates Are Important

Without security gates:

    Developer
        |
        ↓
    Code
        |
        ↓
    Build
        |
        ↓
    Docker Image
        |
        ↓
    Deploy
        |
        ↓
    Vulnerable Application

With security gates:

    Developer
        |
        ↓
    Code
        |
        ↓
    Build
        |
        ↓
    Security Scan
        |
        ↓
    Security Gate
        |
        +-- FAIL → Stop
        |
        +-- PASS
              |
              ↓
          Publish
              |
              ↓
           Deploy

---

# What Is a Security Gate?

A security gate is a condition that must be satisfied before the pipeline continues.

Example:

    Trivy Scan
        |
        ↓
    Critical Vulnerabilities?
        |
        +-- YES → FAIL
        |
        +-- NO → PASS
                     |
                     ↓
                  Publish

The gate converts security findings into a pipeline decision.

---

# Security Gate vs Security Scan

They are related but different.

Security Scan:

    Finds security issues.

Security Gate:

    Decides whether those issues should block the pipeline.

Example:

    Trivy
      |
      ↓
    Finds:
      2 HIGH
      1 CRITICAL
      |
      ↓
    Security Gate
      |
      ↓
    CRITICAL found
      |
      ↓
    Pipeline FAILED

---

# Common DevSecOps Security Checks

A GitHub Actions security pipeline may include:

- SAST
- SCA
- Secret Scanning
- Container Scanning
- IaC Scanning
- DAST
- Dependency Scanning
- License Checks

Typical flow:

    Source Code
        |
        +-- SAST
        |
        +-- SCA
        |
        +-- Secret Scan
        |
        ↓
    Build
        |
        ↓
    Docker Image
        |
        ↓
    Container Scan
        |
        ↓
    Security Gate
        |
        ↓
    Publish

---

# SAST

SAST means:

    Static Application Security Testing

It analyzes source code without running the application.

Example:

    Source Code
        |
        ↓
    SAST
        |
        ↓
    Security Findings

SAST can identify patterns such as:

- Injection risks
- Insecure coding patterns
- Hardcoded credentials
- Dangerous APIs
- Security weaknesses

---

# SonarQube in Security Gates

SonarQube can analyze:

- Code Quality
- Bugs
- Vulnerabilities
- Code Smells
- Security Hotspots
- Test Coverage

Example:

    Source Code
        |
        ↓
    SonarQube
        |
        ↓
    Quality Gate
        |
        +-- PASS
        |
        +-- FAIL

SonarQube can therefore participate in a CI quality/security gate.

---

# SCA

SCA means:

    Software Composition Analysis

SCA analyzes third-party dependencies.

Example:

    Application
        |
        +-- Spring
        +-- Jackson
        +-- npm package
        +-- Python package
        |
        ↓
    Dependency Scan
        |
        ↓
    Vulnerability Database
        |
        ↓
    Findings

---

# Why SCA Is Important

Modern applications depend heavily on open-source libraries.

Example:

    Application
        |
        ↓
    100 Dependencies
        |
        ↓
    One Dependency
        |
        ↓
    Known CVE
        |
        ↓
    Potential Risk

SCA helps identify vulnerable dependencies before release.

---

# Container Security Scanning

After building a Docker image:

    Dockerfile
        |
        ↓
    Docker Build
        |
        ↓
    Image
        |
        ↓
    Trivy
        |
        ↓
    Vulnerability Findings
        |
        ↓
    Security Gate

This is especially important before pushing images to ECR or another container registry.

---

# Trivy Security Gate

Example:

    - name: Trivy Scan
      uses: aquasecurity/trivy-action@v0.36.0
      with:
        image-ref: myapp:${{ github.sha }}
        format: table
        exit-code: '1'
        severity: 'CRITICAL,HIGH'
        vuln-type: 'os,library'

The important setting is:

    exit-code: '1'

This causes the step to return a failure status when matching vulnerabilities are found.

---

# How Trivy Blocks the Pipeline

Flow:

    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    HIGH / CRITICAL
        |
        ↓
    exit-code: 1
        |
        ↓
    Step Failed
        |
        ↓
    Job Failed
        |
        ↓
    Publish Blocked

---

# Trivy Severity Levels

Trivy can report vulnerability severities such as:

    UNKNOWN
    LOW
    MEDIUM
    HIGH
    CRITICAL

A security policy may decide which severities block the pipeline.

Example:

    LOW       → Informational
    MEDIUM    → Review
    HIGH      → Block
    CRITICAL  → Block

The exact policy depends on organizational risk requirements.

---

# Security Gate Policy

A security gate should define:

    What?
      |
      ↓
    Which vulnerabilities?

    Severity?
      |
      ↓
    Which severity blocks?

    Scope?
      |
      ↓
    Which application / image?

    Exception?
      |
      ↓
    Is there an approved exception?

---

# Security Gate Example

Policy:

    CRITICAL → Block
    HIGH     → Block
    MEDIUM   → Review
    LOW      → Informational

Pipeline:

    Scan
      |
      ↓
    Findings
      |
      +-- CRITICAL → FAIL
      +-- HIGH     → FAIL
      +-- MEDIUM   → Continue / Review
      +-- LOW      → Continue

---

# Security Gate in CI

Typical pipeline:

    Checkout
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
    Dependency Scan
       |
       ↓
    Docker Build
       |
       ↓
    Trivy
       |
       ↓
    Security Gate
       |
       +-- FAIL → Stop
       |
       +-- PASS
              |
              ↓
         Publish Artifact

---

# Security Gate Before Publishing

This is an important principle.

Bad:

    Build
      |
      ↓
    Publish Image
      |
      ↓
    Scan Image

The vulnerable image has already been published.

Better:

    Build
      |
      ↓
    Scan
      |
      ↓
    Security Gate
      |
      ↓
    Push Image

This reduces the chance of publishing known vulnerable artifacts.

---

# Security Gate Before Deployment

Another model:

    Artifact
       |
       ↓
    Security Scan
       |
       ↓
    Gate
       |
       +-- PASS → Deploy
       |
       +-- FAIL → Stop

This provides an additional protection layer.

---

# Multiple Security Gates

An enterprise pipeline may have several gates.

    Source Code
        |
        ↓
    SAST
        |
        ↓
    Gate 1
        |
        ↓
    Dependency Scan
        |
        ↓
    Gate 2
        |
        ↓
    Docker Build
        |
        ↓
    Container Scan
        |
        ↓
    Gate 3
        |
        ↓
    Publish
        |
        ↓
    Deploy

---

# Gate 1: Source Security

Example checks:

- SAST
- Secret Scanning
- Security Hotspots

Flow:

    Source
      |
      ↓
    SAST
      |
      ↓
    Gate

---

# Gate 2: Dependency Security

Example:

    Dependencies
        |
        ↓
    SCA
        |
        ↓
    CVE Check
        |
        ↓
    Gate

---

# Gate 3: Container Security

Example:

    Docker Image
        |
        ↓
    Trivy
        |
        ↓
    CVE Check
        |
        ↓
    Gate

---

# Gate 4: Deployment Security

Before production:

    Artifact
        |
        ↓
    Security Validation
        |
        ↓
    Approval
        |
        ↓
    Production

This may include:

- Vulnerability status
- Image provenance
- Deployment policy
- Required approvals
- Environment protection

---

# GitHub Actions Job-Level Security Gate

Example:

    jobs:

      security:
        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Security Scan
            run: ./security-scan.sh

      publish:
        needs: security
        runs-on: ubuntu-latest

        steps:

          - name: Publish
            run: ./publish.sh

Flow:

    Security
       |
       +-- PASS → Publish
       |
       +-- FAIL → Publish blocked

---

# needs as a Security Dependency

The `needs` keyword creates a dependency between jobs.

Example:

    publish:
      needs: security

Meaning:

    Security Job
        |
        ↓
    Publish Job

The publish job waits for the security job.

If the security job fails, the publish job does not normally run.

---

# Multiple Security Dependencies

Example:

    publish:
      needs:
        - test
        - sonarqube
        - security

Flow:

    Unit Tests
         \
    SonarQube -----> Publish
         /
    Trivy

The publish job requires all listed dependencies to succeed.

---

# Security Gate Workflow

Example:

    jobs:

      test:
        ...

      sonarqube:
        needs: test
        ...

      trivy:
        needs: test
        ...

      publish:
        needs:
          - sonarqube
          - trivy
        ...

This creates a security-aware publishing flow.

---

# Parallel Security Checks

Security checks do not always need to run sequentially.

Example:

    Build
      |
      +-------- SonarQube
      |
      +-------- Dependency Scan
      |
      +-------- Secret Scan
      |
      +-------- IaC Scan
      |
      ↓
    Security Gate
      |
      ↓
    Publish

Parallel checks can reduce pipeline execution time.

---

# Central Security Gate

Instead of publishing directly after scans:

    SonarQube
       |
    Trivy
       |
    SCA
       |
    Secret Scan
       |
       ↓
    Security Gate
       |
       ↓
    Publish

The gate job can aggregate results.

---

# Gate Aggregation

Conceptually:

    SAST
      |
      +-- PASS
      |
    SCA
      |
      +-- PASS
      |
    Trivy
      |
      +-- PASS
      |
    Secret Scan
      |
      +-- PASS
      |
      ↓
    Security Gate
      |
      ↓
    Publish

If one required check fails:

    Security Gate
        |
        ↓
      FAIL
        |
        X
    Publish

---

# Security Gate With Outputs

A security job can produce an output.

Conceptually:

    Security Scan
        |
        ↓
    Decision
        |
        ↓
    security_passed=true

Another job can use that output.

Example concept:

    security:
      outputs:
        passed: ${{ steps.check.outputs.passed }}

Then:

    publish:
      needs: security

This can support explicit policy decisions.

---

# Security Gate Using Conditions

Example:

    publish:
      needs: security
      if: needs.security.result == 'success'

This ensures the publish job runs only after the security job succeeds.

---

# Security Gate and Pull Requests

For Pull Requests:

    Pull Request
        |
        ↓
    CI
        |
        +-- Build
        +-- Unit Test
        +-- SonarQube
        +-- Dependency Scan
        +-- Secret Scan
        |
        ↓
    Required Checks
        |
        +-- PASS → Merge
        |
        +-- FAIL → Block Merge

---

# Branch Protection

GitHub branch protection can require specific status checks before merging.

Example:

    Pull Request
        |
        ↓
    Required Checks
        |
        +-- Build
        +-- Test
        +-- Security
        |
        ↓
    Branch Protection
        |
        +-- Pass → Merge
        |
        +-- Fail → Block

This creates a repository-level security gate.

---

# Security Gate and Main Branch

A typical main branch flow:

    Pull Request
       |
       ↓
    Security Checks
       |
       ↓
    Merge
       |
       ↓
    Main
       |
       ↓
    Build
       |
       ↓
    Container Scan
       |
       ↓
    Publish
       |
       ↓
    Deploy

---

# Security Gate and Production

Production should have stronger controls than Pull Request validation.

Example:

    Pull Request
       |
       +-- SAST
       +-- SCA
       +-- Secret Scan
       |
       ↓
    Merge
       |
       ↓
    Main
       |
       +-- Container Scan
       +-- Artifact Validation
       |
       ↓
    QA
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

# Environment Protection

GitHub Actions environments can be used to add deployment controls.

Example:

    Production Environment
        |
        +-- Required Approval
        +-- Environment Secrets
        +-- Deployment Controls
        |
        ↓
    Production Deployment

Security gates and environment protection can work together.

---

# Security Gate vs Approval

Security Gate:

    Automated Decision

Approval:

    Human Decision

Example:

    Trivy
      |
      ↓
    Security Gate
      |
      ↓
    PASS
      |
      ↓
    Production Approval
      |
      ↓
    Deploy

A human approval should not replace automated security checks.

---

# Automated Security + Human Approval

A mature pipeline can use:

    Automated Checks
          |
          ↓
    Security Gate
          |
          ↓
    Quality Gate
          |
          ↓
    Human Approval
          |
          ↓
    Production

Each layer addresses a different risk.

---

# Secret Scanning

Secret scanning detects credentials accidentally committed to source code.

Examples:

    AWS Access Key
    API Token
    Private Key
    Password
    Service Credential

Flow:

    Git Commit
        |
        ↓
    Secret Scan
        |
        ↓
    Secret Found?
        |
        +-- YES → Block
        |
        +-- NO → Continue

---

# Why Secret Scanning Is Critical

A leaked credential can allow:

    Source Code
        |
        ↓
    Leaked Credential
        |
        ↓
    Unauthorized Access
        |
        ↓
    Cloud / Database / API

Security gates can help stop the credential from progressing.

---

# Hardcoded Secrets

Bad:

    AWS_ACCESS_KEY=xxxxx
    AWS_SECRET_KEY=xxxxx

Better:

    GitHub Secrets
        +
    OIDC / Short-Lived Credentials
        +
    External Secret Management

Never commit production credentials to source code.

---

# Dependency Security Gate

Example:

    package-lock.json
        |
        ↓
    Dependency Scanner
        |
        ↓
    CVE
        |
        ↓
    Severity
        |
        ↓
    Policy
        |
        +-- Block
        |
        +-- Allow

The policy should be documented.

---

# Vulnerability Exception

Sometimes a vulnerability cannot immediately be fixed.

A mature process may allow a documented exception.

Example:

    Vulnerability
        |
        ↓
    Risk Assessment
        |
        ↓
    Exception Request
        |
        ↓
    Security Approval
        |
        ↓
    Temporary Exception

The exception should have:

- Reason
- Risk
- Owner
- Expiration
- Compensating Control

---

# Do Not Simply Ignore Vulnerabilities

Bad approach:

    HIGH Vulnerability
        |
        ↓
    continue-on-error: true
        |
        ↓
    Publish

This hides the security problem.

Better:

    Vulnerability
        |
        ↓
    Assess
        |
        +-- Block
        |
        +-- Approved Exception
        |
        +-- Fix

---

# continue-on-error and Security

Be careful with:

    continue-on-error: true

If used on a security scan, the pipeline may continue even when the scan reports a failure.

Use it only when the scan is intentionally informational and is not supposed to be a blocking gate.

---

# Security Gate Policy as Code

Security rules can be automated.

Example concept:

    if critical_count > 0:
        fail

    if high_count > threshold:
        fail

    otherwise:
        pass

This creates consistent security decisions.

---

# Threshold-Based Security Gate

Example policy:

    Critical:
      Allowed = 0

    High:
      Allowed = 0

    Medium:
      Allowed = 10

    Low:
      Informational

Pipeline:

    Scan Results
        |
        ↓
    Policy Evaluation
        |
        ↓
    Gate Decision

---

# Security Thresholds

Do not choose thresholds blindly.

Consider:

- Application Risk
- Environment
- Internet Exposure
- Vulnerability Exploitability
- Business Impact
- Availability of Fix
- Compensating Controls

Production internet-facing applications may require stricter policies.

---

# False Positives

Security scanners may occasionally report false positives.

Example:

    Scanner
       |
       ↓
    Finding
       |
       ↓
    Manual Review
       |
       ↓
    False Positive

Do not simply disable the scanner.

Instead:

    Validate Finding
        |
        ↓
    Document Decision
        |
        ↓
    Apply Approved Exception
        |
        ↓
    Track Until Resolved

---

# Security Gate and Risk Management

Not every vulnerability has the same risk.

Example:

    Vulnerability A
      |
      +-- Critical
      +-- Internet Exposed
      +-- Exploitable
      |
      ↓
    High Priority

    Vulnerability B
      |
      +-- Medium
      +-- Internal
      +-- Difficult to Exploit
      |
      ↓
    Lower Priority

Security gates should be aligned with organizational risk policy.

---

# IaC Security Gate

Infrastructure code should also be scanned.

Example:

    Terraform
       |
       ↓
    IaC Scanner
       |
       ↓
    Security Findings
       |
       ↓
    Gate

Possible checks:

- Public S3 Bucket
- Open Security Group
- Weak IAM Policy
- Public Database
- Encryption Disabled

---

# Terraform Security Pipeline

    Terraform Code
        |
        ↓
    terraform fmt
        |
        ↓
    terraform validate
        |
        ↓
    IaC Security Scan
        |
        ↓
    terraform plan
        |
        ↓
    Security Gate
        |
        ↓
    Approval
        |
        ↓
    Apply

---

# Kubernetes Security Gate

Kubernetes manifests can also be scanned.

Example:

    Kubernetes YAML
        |
        ↓
    Security Scan
        |
        +-- Privileged Container
        +-- Host Network
        +-- Missing Limits
        +-- Excessive Permissions
        |
        ↓
    Security Gate

---

# Container Security Policy

A policy may require:

    No Critical CVEs
    No High CVEs
    Non-Root Container
    Approved Base Image
    No Secrets in Image

Then:

    Docker Image
        |
        ↓
    Security Checks
        |
        ↓
    Gate

---

# Base Image Security

Example:

    FROM ubuntu:old-version

Potential issue:

    Old Base Image
        |
        ↓
    Known CVEs
        |
        ↓
    Trivy
        |
        ↓
    Gate Failure

Use supported and maintained base images.

---

# Dependency Security and Base Images

Container security includes:

    Application Dependencies
          +
    OS Packages
          +
    Base Image
          |
          ↓
    Container Security

A clean application dependency list does not guarantee a secure image.

---

# Security Gate Before ECR

For an AWS EKS pipeline:

    GitHub Actions
        |
        ↓
    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Security Gate
        |
        +-- FAIL → No Push
        |
        +-- PASS
              |
              ↓
             ECR
              |
              ↓
             EKS

This is a strong DevSecOps pattern.

---

# Security Gate Before ArgoCD

For a GitOps workflow:

    GitHub Actions
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
    Security Gate
        |
        ↓
    Image / Artifact
        |
        ↓
    GitOps Repository
        |
        ↓
    ArgoCD
        |
        ↓
    EKS

The security gate happens before promoting the deployment configuration.

---

# Complete DevSecOps Security Flow

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    GitHub Actions
        |
        +-- Unit Tests
        |
        +-- SonarQube
        |
        +-- Dependency Scan
        |
        +-- Secret Scan
        |
        +-- IaC Scan
        |
        ↓
    Build
        |
        ↓
    Docker Image
        |
        ↓
    Trivy
        |
        ↓
    Security Gate
        |
        +-- FAIL → Stop
        |
        +-- PASS
              |
              ↓
          ECR / Artifact Repository
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
        Post-Deployment Validation

---

# Security Gate for Your DevOps Stack

A practical pipeline using your tools:

    GitHub
       |
       ↓
    GitHub Actions
       |
       +-- Build
       |
       +-- Unit Tests
       |
       +-- SonarQube
       |
       +-- Docker Build
       |
       +-- Trivy
       |
       +-- Veracode
       |
       ↓
    Security / Quality Gate
       |
       +-- FAIL → Stop
       |
       +-- PASS
              |
              ↓
          ECR / Artifact Repository
              |
              ↓
          ArgoCD
              |
              ↓
             EKS

---

# Security Gate and Veracode

Veracode can be integrated into the application security pipeline.

Conceptually:

    Application
        |
        ↓
    Build
        |
        ↓
    Veracode
        |
        ↓
    Security Analysis
        |
        ↓
    Policy / Gate
        |
        ↓
    Continue or Stop

The exact integration depends on the Veracode scanning product and organization policy.

---

# Security Gate and SonarQube

Typical flow:

    Source
       |
       ↓
    Build / Test
       |
       ↓
    SonarQube
       |
       ↓
    Quality Gate
       |
       +-- FAIL → Stop
       |
       +-- PASS → Continue

The SonarQube quality gate can be treated as a required CI condition.

---

# Security Gate and Trivy

Typical flow:

    Docker Build
       |
       ↓
    Trivy
       |
       ↓
    Vulnerability Findings
       |
       ↓
    Severity Policy
       |
       +-- Fail
       |
       +-- Pass
              |
              ↓
            Push

---

# Quality Gate vs Security Gate

Quality Gate:

    Code Quality
       |
       +-- Bugs
       +-- Coverage
       +-- Maintainability
       |
       ↓
    Decision

Security Gate:

    Security
       |
       +-- Vulnerabilities
       +-- Secrets
       +-- Dependencies
       +-- Container Risks
       |
       ↓
    Decision

They may be combined into one overall pipeline gate.

---

# Combined Quality and Security Gate

    Unit Tests
         |
    SonarQube
         |
    Trivy
         |
    SCA
         |
         ↓
    Quality + Security Gate
         |
         +-- FAIL → Stop
         |
         +-- PASS → Publish

---

# Security Gate Failure Handling

When a gate fails:

    Security Finding
        |
        ↓
    Pipeline Failed
        |
        ↓
    Developer Notified
        |
        ↓
    Investigate
        |
        ↓
    Fix
        |
        ↓
    New Commit
        |
        ↓
    Pipeline Re-runs

This creates a feedback loop.

---

# Security Gate Feedback

A useful failure message should explain:

    What failed?
    |
    ↓
    Which vulnerability?
    |
    ↓
    Which severity?
    |
    ↓
    Which component?
    |
    ↓
    What should be fixed?

Avoid simply reporting:

    SECURITY FAILED

Provide actionable information.

---

# Security Gate and Pull Request Comments

Security tools may provide reports or annotations.

Example:

    Pull Request
        |
        ↓
    Security Scan
        |
        ↓
    Finding
        |
        ↓
    Developer Feedback

This allows developers to fix vulnerabilities before merge.

---

# Security Gate and Branch Protection

Recommended:

    Pull Request
        |
        ↓
    Security Workflow
        |
        ↓
    Required Status Check
        |
        ↓
    Branch Protection
        |
        +-- Pass → Merge
        |
        +-- Fail → Block

This makes the security control enforceable.

---

# Security Gate Bypass

Sometimes an emergency may require a bypass.

A mature process should define:

    Who can approve?
    |
    ↓
    Why?
    |
    ↓
    How long?
    |
    ↓
    What compensating control?
    |
    ↓
    When will it be fixed?

Bypasses should be rare, documented, and auditable.

---

# Security Gate Auditability

A production pipeline should allow you to answer:

    Which code was scanned?
        |
        ↓
    Which scanner?
        |
        ↓
    Which scanner version?
        |
        ↓
    Which findings?
        |
        ↓
    Which policy?
        |
        ↓
    Why did the gate pass?
        |
        ↓
    Which artifact was published?

This improves compliance and incident investigation.

---

# Security Gate and Artifact Traceability

Strong flow:

    Git Commit
        |
        ↓
    CI Run
        |
        ↓
    Security Scan
        |
        ↓
    Gate Result
        |
        ↓
    Artifact
        |
        ↓
    Registry
        |
        ↓
    Deployment

The artifact should be traceable back to the code and security checks that produced it.

---

# Security Gate and Immutable Artifacts

After the gate passes:

    Build
      |
      ↓
    Scan
      |
      ↓
    Gate
      |
      ↓
    Artifact
      |
      ↓
    Immutable
      |
      ↓
    Promote

Do not modify the artifact after it has passed security validation.

---

# Security Gate Best Practices

- Scan before publishing
- Scan before production deployment
- Use defined severity thresholds
- Fail the pipeline for policy violations
- Do not ignore critical findings
- Avoid `continue-on-error` for mandatory security checks
- Scan dependencies
- Scan container images
- Scan IaC
- Scan for secrets
- Use least-privilege permissions
- Protect production environments
- Use branch protection
- Document exceptions
- Set expiration dates for exceptions
- Keep scanners updated
- Keep security policies version-controlled
- Preserve scan results
- Maintain artifact traceability
- Use immutable artifacts
- Audit security decisions

---

# Security Gate Anti-Patterns

## Anti-Pattern 1: Scan After Publishing

Bad:

    Build
      |
      ↓
    Push Image
      |
      ↓
    Trivy

Better:

    Build
      |
      ↓
    Trivy
      |
      ↓
    Gate
      |
      ↓
    Push

---

# Anti-Pattern 2: Ignore All HIGH Findings

Bad:

    HIGH Vulnerability
        |
        ↓
    Ignore

Better:

    HIGH Vulnerability
        |
        ↓
    Risk Assessment
        |
        +-- Fix
        |
        +-- Approved Exception

---

# Anti-Pattern 3: continue-on-error for Mandatory Scan

Bad:

    Trivy
      |
      ↓
    continue-on-error: true
      |
      ↓
    Publish

This can allow known vulnerabilities to continue.

---

# Anti-Pattern 4: Hardcoded Security Credentials

Bad:

    username: admin
    password: secret

Better:

    GitHub Secrets
        +
    OIDC
        +
    Short-Lived Credentials

---

# Anti-Pattern 5: No Branch Protection

If security checks exist but developers can merge without passing them:

    Security Scan
        |
        ↓
      FAIL
        |
        ↓
    Merge Allowed

The security control is not enforceable.

Use required status checks.

---

# Anti-Pattern 6: Permanent Exceptions

Bad:

    Vulnerability
        |
        ↓
    Exception
        |
        ↓
    Never Review Again

Better:

    Exception
        |
        ↓
    Owner
        |
        ↓
    Expiration Date
        |
        ↓
    Reassess
        |
        ↓
    Fix / Renew With Approval

---

# Anti-Pattern 7: Scanning Only Source Code

Source code scanning alone is insufficient.

Also consider:

    Dependencies
    Containers
    IaC
    Secrets
    Deployment Configuration

Security is a lifecycle concern.

---

# Interview Questions

## Basic

1. What is a security gate?
2. What is the difference between a security scan and a security gate?
3. What is DevSecOps?
4. What is SAST?
5. What is SCA?
6. What is DAST?
7. What is container scanning?
8. What is secret scanning?
9. What is an IaC security scan?
10. Why should security scans run in CI?

---

# Intermediate Interview Questions

11. How do you implement a security gate in GitHub Actions?

12. How does Trivy block a GitHub Actions pipeline?

13. What does `exit-code: '1'` do in a Trivy scan?

14. How do you block publishing when vulnerabilities are found?

15. How do you use `needs` for security jobs?

16. How do you make security checks required before merging?

17. How do you configure severity thresholds?

18. How do you handle false positives?

19. How do you handle vulnerability exceptions?

20. How do you protect production deployments?

21. How do you scan Docker images?

22. How do you scan Terraform code?

23. How do you scan Kubernetes manifests?

24. How do you prevent secrets from being committed?

25. How do you combine SonarQube and Trivy in CI?

---

# Advanced Interview Questions

26. Design an enterprise DevSecOps security gate using GitHub Actions.

27. How would you design multiple security gates across the SDLC?

28. How would you prevent vulnerable Docker images from reaching ECR?

29. How would you integrate GitHub Actions with SonarQube, Trivy, and Veracode?

30. How would you implement a centralized security policy?

31. How would you handle false positives without weakening security?

32. How would you design a vulnerability exception process?

33. How would you make security checks mandatory for production?

34. How would you design security gates for microservices?

35. How would you secure GitHub Actions itself?

36. How would you protect against dependency and cache-related supply-chain attacks?

37. How would you implement artifact traceability?

38. How would you design security gates for a GitOps pipeline?

39. How would you design security controls for Terraform and Kubernetes?

40. How would you balance developer velocity with strict security gates?

---

# Scenario Question

## Trivy finds one CRITICAL vulnerability in your Docker image. What would you do?

I would not publish the image automatically.

Flow:

    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    CRITICAL
        |
        ↓
    Security Gate
        |
        ↓
      FAIL
        |
        X
      ECR

Then I would:

    1. Identify the vulnerable package
    2. Determine whether the vulnerability is exploitable
    3. Check for an updated package or base image
    4. Rebuild
    5. Rescan
    6. Publish only after the security policy passes

If an exception is genuinely required, it should go through the organization's documented security approval process.

---

# Scenario Question

## Your security scan fails, but the developer says it is a false positive. What do you do?

I would:

    Finding
      |
      ↓
    Validate
      |
      ↓
    Security Review
      |
      +-- Real → Fix
      |
      +-- False Positive
             |
             ↓
        Document
             |
             ↓
        Approved Exception
             |
             ↓
        Track

I would not simply disable the scanner or ignore the finding globally.

---

# Scenario Question

## Developers complain that security gates slow down development. What would you do?

I would optimize the pipeline without removing important security controls.

Possible improvements:

    Parallel Scans
        +
    Dependency Caching
        +
    Faster Runners
        +
    Incremental Scanning
        +
    Test Optimization
        +
    Better Policy

For example:

    Build
      |
      +-- SonarQube
      +-- SCA
      +-- Secret Scan
      |
      ↓
    Docker Build
      |
      ↓
    Trivy
      |
      ↓
    Gate

The goal is:

    Fast Security
        +
    Strong Security

---

# Scenario Question

## How would you prevent a vulnerable image from being pushed to ECR?

I would make Trivy a required dependency of the push job.

Example:

    trivy:
      ...

    push:
      needs: trivy

If Trivy returns a failure:

    Trivy
      |
      ↓
    FAIL
      |
      X
    Push

Only a passing scan allows the image to reach ECR.

---

# Scenario Question

## How would you implement security gates for Terraform?

I would use:

    Terraform Code
        |
        ↓
    terraform fmt
        |
        ↓
    terraform validate
        |
        ↓
    IaC Security Scan
        |
        ↓
    terraform plan
        |
        ↓
    Security Gate
        |
        ↓
    Approval
        |
        ↓
    Apply

This catches infrastructure security issues before changes are applied.

---

# Scenario Question

## How would you secure a GitOps deployment?

I would use:

    Developer
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
    Security Gate
       |
       ↓
    Artifact
       |
       ↓
    GitOps Repository
       |
       ↓
    ArgoCD
       |
       ↓
    EKS

The security gate happens before promoting the application to the deployment stage.

---

# Scenario Question

## What if a critical vulnerability has no available fix?

I would:

    Critical Vulnerability
        |
        ↓
    Risk Assessment
        |
        +-- Is it exploitable?
        +-- Is the component exposed?
        +-- Is there a workaround?
        +-- Is there a patched alternative?
        |
        ↓
    Security Decision

If the organization permits an exception:

    Exception
       |
       +-- Owner
       +-- Business Justification
       +-- Compensating Control
       +-- Expiration
       |
       ↓
    Approved Temporarily

The vulnerability should remain tracked until it can be remediated.

---

# Scenario Question

## How would you explain a security gate to an interviewer?

A strong answer:

"A security gate is an automated checkpoint in the CI/CD pipeline that evaluates security results against an organization's defined policy. For example, after building a Docker image, I can run Trivy and configure the pipeline to fail when HIGH or CRITICAL vulnerabilities are detected. The publishing job depends on the security job using `needs`, so a failed security check prevents the image from being pushed to ECR. Similar gates can be applied to SonarQube, dependency scanning, secret scanning, and IaC security checks."

---

# Example Security Pipeline

    name: Secure CI

    on:
      pull_request:
      push:
        branches:
          - main

    permissions:
      contents: read

    jobs:

      test:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Run Tests
            run: |
              echo "Run application tests here"

      security:

        needs: test
        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Run Security Scan
            run: |
              ./security-scan.sh

      publish:

        needs: security
        runs-on: ubuntu-latest

        steps:

          - name: Publish Artifact
            run: |
              echo "Publishing validated artifact"

---

# Example Container Security Pipeline

    name: Container Security

    on:
      push:
        branches:
          - main

    permissions:
      contents: read

    jobs:

      build:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Build Image
            run: |
              docker build \
                -t myapp:${{ github.sha }} \
                .

      security:

        needs: build
        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Trivy Scan
            uses: aquasecurity/trivy-action@v0.36.0
            with:
              image-ref: myapp:${{ github.sha }}
              format: table
              exit-code: '1'
              severity: 'CRITICAL,HIGH'

      publish:

        needs: security
        runs-on: ubuntu-latest

        steps:

          - name: Publish Image
            run: |
              echo "Push image to ECR here"

---

# Important Security Gate Concepts

Remember:

    Scan
      |
      ↓
    Policy
      |
      ↓
    Gate
      |
      ↓
    Decision

Security scan:

    Finds Problems

Security policy:

    Defines Acceptable Risk

Security gate:

    Enforces the Policy

---

# Security Gate Mental Model

Think of a security gate as a door.

    Application
        |
        ↓
    Security Scanner
        |
        ↓
    Security Gate
        |
        +----------------+
        |                |
      PASS              FAIL
        |                |
        ↓                ↓
     Continue           Stop
        |                |
        ↓                ↓
     Publish          Fix Issue
        |                |
        ↓                ↓
     Deploy          Scan Again

---

# Final DevSecOps Security Flow

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
        +-- Unit Tests
        |
        +-- SonarQube
        |
        +-- Dependency Scan
        |
        +-- Secret Scan
        |
        +-- IaC Scan
        |
        ↓
    Build
        |
        ↓
    Docker Image
        |
        ↓
    Trivy
        |
        ↓
    Security Gate
        |
        +-- FAIL → Fix / Exception
        |
        +-- PASS
              |
              ↓
          ECR / Artifact Repository
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
              ↓
       Post-Deployment Checks

---

# Final Mental Model

The most important concepts are:

    Security Scan
        ↓
    Finds vulnerabilities

    Security Policy
        ↓
    Defines what is acceptable

    Security Gate
        ↓
    Enforces the policy

    Branch Protection
        ↓
    Prevents insecure code from merging

    Artifact Gate
        ↓
    Prevents vulnerable artifacts from being published

    Deployment Gate
        ↓
    Prevents unsafe deployments

The core DevSecOps principle is:

    Find Security Issues Early
              |
              ↓
    Automatically Enforce Policy
              |
              ↓
    Block Unsafe Changes
              |
              ↓
    Allow Only Validated Artifacts
              |
              ↓
    Deploy Secure Software

A strong GitHub Actions security gate does not simply run a scanner. It connects security scanning to an enforceable decision. SonarQube, Trivy, dependency scanning, secret scanning, IaC scanning, and tools such as Veracode can identify risks, while GitHub Actions job dependencies, conditions, branch protection, environment controls, and defined security policies determine whether the software is allowed to progress through the CI/CD pipeline.