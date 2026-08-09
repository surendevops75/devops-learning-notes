# Trivy with GitHub Actions

Trivy is an open-source security scanner commonly used in CI/CD pipelines to identify vulnerabilities and other security issues.

In GitHub Actions, Trivy can be used to scan:

- Docker Images
- Filesystems
- Git Repositories
- Dependencies
- Infrastructure as Code
- Kubernetes-related targets
- Secrets
- Licenses
- Configuration Misconfigurations

A typical GitHub Actions DevSecOps flow is:

Developer
    |
    ↓
GitHub Repository
    |
    ↓
GitHub Actions
    |
    +-- Checkout
    |
    +-- Build
    |
    +-- Test
    |
    +-- SonarQube
    |
    +-- Docker Build
    |
    +-- Trivy Scan
    |
    ↓
Security Gate
    |
    +-- Pass → Continue
    |
    +-- Fail → Stop

---

# What Is Trivy?

Trivy is a security scanner developed by Aqua Security.

It can detect known vulnerabilities and can also scan for other security-related findings depending on the target and scanners enabled.

Trivy's main scanner categories include:

- Vulnerability Scanner
- Misconfiguration Scanner
- Secret Scanner
- License Scanner

The exact scanners enabled depend on the Trivy command and configuration.

---

# Why Use Trivy with GitHub Actions?

Without automated vulnerability scanning:

Developer
    |
    ↓
Build
    |
    ↓
Docker Image
    |
    ↓
Registry
    |
    ↓
Deployment

A vulnerable dependency or image could move through the pipeline unnoticed.

With Trivy:

Developer
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
    +-- Pass → Registry
    |
    +-- Fail → Stop

This allows security checks to happen automatically during CI.

---

# Trivy in DevSecOps

Trivy can be one stage in a broader DevSecOps pipeline.

Example:

    Code
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
    Deployment

SonarQube and Trivy can complement each other.

---

# SonarQube vs Trivy

SonarQube primarily focuses on source-code quality and code analysis.

Trivy can scan dependencies, container images, filesystems, repositories, and configurations depending on the command and scanner configuration.

Conceptually:

    SonarQube
        |
        ↓
    Source Code Analysis

    Trivy
        |
        +-- Container
        +-- Dependencies
        +-- Filesystem
        +-- Repository
        +-- IaC
        +-- Secrets
        +-- Licenses

---

# GitHub Actions Trivy Integration

A commonly used GitHub Actions integration is:

    aquasecurity/trivy-action

Example:

    - name: Trivy Scan
      uses: aquasecurity/trivy-action@v0.36.0
      with:
        image-ref: myapp:${{ github.sha }}
        format: table
        exit-code: '1'
        ignore-unfixed: true
        vuln-type: 'os,library'
        severity: 'CRITICAL,HIGH'

The Trivy Action release version should be reviewed and controlled according to your organization's dependency-management policy.

---

# Current Trivy Action Version

The official Trivy Action repository currently lists:

    v0.36.0

as a release.

Always verify the action version before using it in a production workflow.

The Trivy Action project also documents release and security-related changes in its GitHub repository.

---

# GitHub Actions Workflow Location

GitHub Actions workflows are stored under:

    .github/workflows/

Example:

    .github/
    └── workflows/
        └── trivy.yml

---

# Basic Trivy Image Scan

Example:

    name: Trivy Image Scan

    on:
      push:
        branches:
          - main

    jobs:
      security:
        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Build Image
            run: |
              docker build -t myapp:${{ github.sha }} .

          - name: Trivy Scan
            uses: aquasecurity/trivy-action@v0.36.0
            with:
              image-ref: myapp:${{ github.sha }}
              format: table
              exit-code: '1'
              severity: 'CRITICAL,HIGH'

Flow:

Git Push
    |
    ↓
Checkout
    |
    ↓
Docker Build
    |
    ↓
Trivy
    |
    ↓
Pass / Fail

---

# Why Scan the Docker Image?

The Docker image contains:

- Base OS Packages
- Runtime
- Application Dependencies
- Application Components

A vulnerability may come from the base image rather than the application source.

Example:

    Application
        |
        ↓
    Node.js Runtime
        |
        ↓
    OS Packages
        |
        ↓
    Vulnerability

Trivy can identify applicable vulnerabilities in supported packages and components.

---

# Trivy Image Command

The equivalent CLI command is:

    trivy image myapp:v1.0.0

Example with severity:

    trivy image --severity HIGH,CRITICAL myapp:v1.0.0

Example with exit code:

    trivy image \
      --exit-code 1 \
      --severity HIGH,CRITICAL \
      myapp:v1.0.0

---

# exit-code

The `exit-code` option determines what happens when findings matching the configured scan criteria are found.

Example:

    exit-code: '1'

Conceptually:

    Trivy
      |
      ↓
    Findings
      |
      +-- No qualifying findings → Exit 0
      |
      +-- Qualifying findings → Exit 1

In CI, a non-zero exit code normally causes the step to fail.

---

# Why Use exit-code: 1?

Without an appropriate exit code:

    Vulnerability Found
          |
          ↓
       Report Only
          |
          ↓
       Pipeline Continues

With:

    exit-code: '1'

the pipeline can enforce:

    Vulnerability Found
          |
          ↓
       Step Fails
          |
          ↓
       Pipeline Stops

This creates a security gate.

---

# severity

Severity can be used to focus the scan on selected levels.

Example:

    severity: 'CRITICAL,HIGH'

Possible severity categories include:

- UNKNOWN
- LOW
- MEDIUM
- HIGH
- CRITICAL

The exact vulnerability severity comes from the vulnerability data and matching logic used by Trivy.

---

# Security Gate

Example:

    Trivy
      |
      ↓
    CRITICAL / HIGH
      |
      ↓
    Security Gate
      |
      +-- Findings → Fail
      |
      +-- No Findings → Continue

The exact policy should be defined by the organization.

---

# ignore-unfixed

Example:

    ignore-unfixed: true

This can exclude vulnerabilities for which no fix is currently available.

Example:

    Vulnerability
        |
        +-- Fixed Version Available
        |
        +-- No Fixed Version

With:

    ignore-unfixed: true

the scan can focus on vulnerabilities that have a known fix.

Whether this is appropriate depends on the organization's security policy.

---

# Important Warning About ignore-unfixed

Using:

    ignore-unfixed: true

does not mean the vulnerability is harmless.

It only changes what is considered for the scan result.

An organization may instead want visibility into all known vulnerabilities, including those without fixes.

---

# vuln-type

Example:

    vuln-type: 'os,library'

This focuses vulnerability scanning on:

    OS
      +
    Library

This is particularly useful for container images.

---

# Container Image Scan Flow

Dockerfile
    |
    ↓
Docker Build
    |
    ↓
Docker Image
    |
    ↓
Trivy
    |
    +-- OS Packages
    |
    +-- Application Libraries
    |
    ↓
Security Result

---

# Trivy Scan Before Push

Recommended flow:

    Source Code
        |
        ↓
    Docker Build
        |
        ↓
    Trivy Scan
        |
        +-- Fail → Do Not Push
        |
        +-- Pass
              |
              ↓
            ECR

This prevents an image that fails the defined security policy from being pushed.

---

# Trivy Scan After Push

Another architecture is:

    Docker Build
        |
        ↓
    Push ECR
        |
        ↓
    Trivy
        |
        ↓
    Security Result

This can still provide security visibility, but scanning before promotion can prevent non-compliant images from entering the registry.

---

# Recommended CI Approach

For many pipelines:

    Build
      |
      ↓
    Scan
      |
      ↓
    Push

instead of:

    Build
      |
      ↓
    Push
      |
      ↓
    Scan

This allows the pipeline to enforce a security gate before publishing the image.

---

# Trivy and ECR

Typical AWS flow:

    GitHub Actions
         |
         ↓
    Docker Build
         |
         ↓
    Trivy Scan
         |
         ↓
    Security Gate
         |
         ↓
    Amazon ECR
         |
         ↓
    EKS

The image should be pushed only after the required CI security checks pass.

---

# Trivy with Docker Build

Example:

    name: Docker Security

    on:
      push:
        branches:
          - main

    jobs:

      security:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Build Docker Image
            run: |
              docker build \
                -t myapp:${{ github.sha }} \
                .

          - name: Trivy Scan
            uses: aquasecurity/trivy-action@v0.36.0
            with:
              image-ref: myapp:${{ github.sha }}
              format: table
              exit-code: '1'
              severity: 'CRITICAL,HIGH'
              vuln-type: 'os,library'

---

# Trivy Scan and Git Commit

A useful tagging strategy is:

    myapp:${{ github.sha }}

Example:

    Git Commit
        |
        ↓
    8f3a91d
        |
        ↓
    Docker Image
        |
        ↓
    myapp:8f3a91d

Then:

    Trivy
        |
        ↓
    myapp:8f3a91d

This provides traceability from the image back to the Git commit.

---

# Trivy and Image Tags

Possible tags:

    myapp:v1.0.0

    myapp:${{ github.sha }}

    myapp:${{ github.run_number }}

Using a Git commit SHA is useful for traceability.

Using release versions is useful for human-readable release management.

---

# Immutable Image Strategy

A strong pipeline can use:

    Build
      |
      ↓
    Tag
      |
      ↓
    Scan
      |
      ↓
    Push
      |
      ↓
    Deploy

The same validated image should then be promoted through environments.

---

# Build Once, Deploy Many

Example:

    GitHub Actions
          |
          ↓
    Docker Build
          |
          ↓
    Trivy
          |
          ↓
    ECR
          |
          +----> Development
          |
          +----> Staging
          |
          +----> Production

The image should not be rebuilt differently for each environment when the goal is artifact immutability.

---

# Trivy Filesystem Scan

Trivy can scan a local filesystem.

Command:

    trivy fs .

This can identify applicable:

- Vulnerabilities
- Misconfigurations
- Secrets
- Licenses

The exact scanners enabled depend on the command and options.

---

# GitHub Actions Filesystem Scan

Example:

    - name: Trivy Filesystem Scan
      uses: aquasecurity/trivy-action@v0.36.0
      with:
        scan-type: fs
        scan-ref: .
        severity: 'CRITICAL,HIGH'
        exit-code: '1'

Flow:

    GitHub Repository
          |
          ↓
    Checkout
          |
          ↓
    Trivy FS Scan
          |
          ↓
    Security Result

---

# Why Scan the Filesystem?

Filesystem scanning can identify vulnerabilities in dependency manifests and lock files.

Examples:

    package-lock.json
    package.json
    requirements.txt
    Gemfile.lock

It can also be used for supported configuration and secret scanning depending on the enabled scanners.

---

# Trivy Repository Scan

Trivy also provides repository scanning.

CLI:

    trivy repo .

Repository scanning is designed for code repositories and can inspect supported dependency files and other supported targets.

For remote repositories:

    trivy repo https://github.com/example/example

Private repository authentication requires appropriate configuration.

---

# Repository Scan vs Filesystem Scan

Filesystem:

    trivy fs .

Repository:

    trivy repo .

Conceptually:

    fs
      |
      ↓
    Local Filesystem

    repo
      |
      ↓
    Repository-oriented scanning

The appropriate command depends on what you want to scan.

---

# Trivy Misconfiguration Scanning

Trivy can detect configuration misconfigurations.

Examples can include:

- Dockerfile
- Kubernetes manifests
- Terraform
- Cloud configuration
- Other supported IaC formats

Misconfiguration scanning must be enabled where it is not part of the default scanner set for the selected command.

---

# Trivy IaC Scan

Example:

    trivy config .

This can scan supported Infrastructure as Code configuration.

Example project:

    infrastructure/
    ├── main.tf
    ├── variables.tf
    ├── outputs.tf
    └── modules/

Command:

    trivy config infrastructure/

---

# Trivy and Terraform

Example:

    - name: Trivy Terraform Scan
      uses: aquasecurity/trivy-action@v0.36.0
      with:
        scan-type: config
        scan-ref: infrastructure/
        severity: 'HIGH,CRITICAL'
        exit-code: '1'

Flow:

    Terraform
        |
        ↓
    Trivy Config Scan
        |
        ↓
    Misconfiguration
        |
        ↓
    Security Gate

---

# Trivy and Kubernetes

Trivy can also be used with Kubernetes-related security scanning.

Possible targets include:

- Kubernetes configuration
- Kubernetes components
- Kubernetes cluster-related assets

For CI, Kubernetes manifests can be scanned before deployment.

Example:

    trivy config k8s/

---

# Kubernetes Security Pipeline

    Kubernetes Manifests
          |
          ↓
        Trivy
          |
          ↓
    Misconfiguration Scan
          |
          ↓
    Security Gate
          |
          +-- Pass → Deploy
          |
          +-- Fail → Stop

---

# Trivy Secret Scanning

Trivy can detect secrets in supported scan targets.

Examples:

- API Keys
- Credentials
- Tokens
- Private Keys

Example:

    trivy fs --scanners secret .

Never treat a secret finding as a normal warning.

If a real secret is discovered:

    1. Remove the secret
    2. Revoke / rotate it
    3. Investigate exposure
    4. Update the application
    5. Prevent recurrence

---

# Trivy License Scanning

Trivy can also scan supported license information.

Conceptually:

    Dependencies
        |
        ↓
    License Detection
        |
        ↓
    Policy Review

This can help organizations identify dependencies with licenses that conflict with internal policies.

---

# Trivy Scanners

Trivy has four main scanner categories:

    Vulnerability
    Misconfiguration
    Secret
    License

Conceptually:

    Trivy
      |
      +-- Vulnerability
      |
      +-- Misconfiguration
      |
      +-- Secret
      |
      +-- License

The available scanner combinations depend on the target and command.

---

# Scanner Selection

Example:

    --scanners vuln,misconfig,secret

This can enable multiple scanner types where supported.

Example:

    trivy fs \
      --scanners vuln,misconfig,secret \
      .

This allows one scan to cover several security categories.

---

# Trivy Image Scan with Multiple Scanners

Example:

    trivy image \
      --scanners vuln,misconfig,secret \
      myapp:v1.0.0

The appropriate scanner combination depends on the image content and the security objectives.

---

# Vulnerability Database

Trivy uses vulnerability databases to identify known vulnerabilities.

Conceptually:

    Trivy
       |
       ↓
    Vulnerability Database
       |
       ↓
    Package Detection
       |
       ↓
    Vulnerability Matching

Trivy automatically retrieves and maintains the relevant databases as required by its scanning process.

---

# Why Database Updates Matter

Security information changes continuously.

Example:

    Day 1
       |
       ↓
    No Known Vulnerability

    Day 10
       |
       ↓
    New CVE Published

A scanner with updated vulnerability data can identify the new issue.

Therefore, CI environments should have an appropriate database update and caching strategy.

---

# Trivy Cache

Trivy uses caching to reduce repeated downloads and improve scanning efficiency.

In CI:

    GitHub Runner
         |
         ↓
    Trivy
         |
         ↓
    Database
         |
         ↓
    Cache

Caching can reduce repeated database download overhead.

---

# Trivy Cache in GitHub Actions

GitHub Actions runners can be ephemeral.

Therefore:

    Runner 1
       |
       ↓
    Trivy DB
       |
       ↓
    Runner Removed
       |
       ↓
    Cache Lost

A persistent GitHub Actions cache can be used where appropriate.

---

# Why Cache Trivy?

Benefits:

- Faster Scans
- Less Network Usage
- Reduced Database Download Time
- More Efficient CI

However, the cache must be managed so that security data does not become unnecessarily stale.

---

# Trivy Cache Security

Do not sacrifice security for cache speed.

The pipeline should balance:

    Fresh Security Database
           +
    Fast CI

Use an appropriate cache strategy and update policy.

---

# Trivy Scan Output

Default output can be displayed as a table.

Example:

    format: table

Other output formats may include:

- table
- json
- sarif
- template

The selected format depends on how results will be consumed.

---

# JSON Output

Example:

    trivy image \
      --format json \
      myapp:v1.0.0

JSON is useful when another system needs to process the scan results.

---

# SARIF Output

SARIF can be used to integrate security findings with compatible code-scanning systems.

Example:

    trivy image \
      --format sarif \
      --output trivy-results.sarif \
      myapp:v1.0.0

Then the SARIF file can be uploaded to the appropriate GitHub security interface using the relevant GitHub Actions integration.

---

# GitHub Security Integration

Conceptually:

    Trivy
      |
      ↓
    SARIF
      |
      ↓
    GitHub Code Scanning
      |
      ↓
    Security Results

The exact permissions and upload action configuration depend on the repository security setup.

---

# Trivy Result Artifact

The scan result can also be saved as a workflow artifact.

Example:

    trivy image \
      --format json \
      --output trivy-results.json \
      myapp:${{ github.sha }}

Then:

    Upload Artifact
          |
          ↓
    trivy-results.json

This is useful for auditing and troubleshooting.

---

# GitHub Actions Artifact Flow

    Trivy
      |
      ↓
    JSON / SARIF
      |
      ↓
    Upload Artifact
      |
      ↓
    GitHub Actions Run

Artifacts can preserve scan output after the runner is destroyed.

---

# Trivy and Pull Requests

Trivy can run on Pull Requests.

Example:

    on:
      pull_request:

Flow:

    Developer
       |
       ↓
    Pull Request
       |
       ↓
    GitHub Actions
       |
       ↓
    Trivy
       |
       ↓
    Security Result

This gives developers security feedback before merging.

---

# Pull Request Security Workflow

Example:

    name: Security Scan

    on:
      pull_request:
      push:
        branches:
          - main

    jobs:

      trivy:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Build Image
            run: |
              docker build \
                -t myapp:${{ github.sha }} \
                .

          - name: Trivy Scan
            uses: aquasecurity/trivy-action@v0.36.0
            with:
              image-ref: myapp:${{ github.sha }}
              format: table
              exit-code: '1'
              severity: 'CRITICAL,HIGH'

---

# Trivy and Branch Protection

A Trivy workflow can be included among required checks.

Conceptually:

    Pull Request
         |
         ↓
    Required Checks
         |
         +-- Build
         +-- Test
         +-- SonarQube
         +-- Trivy
         |
         ↓
    All Passed?
         |
         +-- Yes → Merge
         |
         +-- No → Block

This makes security scanning part of the merge policy.

---

# Trivy and GitHub Actions Secrets

Trivy image scanning normally does not require a secret when scanning a locally built public-accessible image.

However, secrets may be required when accessing:

- Private container registries
- Private repositories
- Cloud services

Never hardcode registry credentials.

---

# Trivy Private ECR Image

If scanning an image already stored in private ECR:

    GitHub Actions
        |
        ↓
    AWS Authentication
        |
        ↓
    ECR Login
        |
        ↓
    Trivy
        |
        ↓
    Private Image

For CI simplicity, scanning the locally built image before pushing can avoid requiring registry access for the scan itself.

---

# Trivy with AWS ECR

Typical flow:

    Checkout
       |
       ↓
    Build
       |
       ↓
    Trivy
       |
       ↓
    AWS Authentication
       |
       ↓
    ECR Login
       |
       ↓
    Push Image

This separates vulnerability scanning from registry publishing.

---

# GitHub OIDC and ECR

A GitHub Actions workflow can use OpenID Connect to authenticate to AWS without storing long-lived AWS access keys in GitHub secrets.

Conceptually:

    GitHub Actions
          |
          ↓
        OIDC
          |
          ↓
        AWS IAM
          |
          ↓
        ECR

Then:

    Docker Push
          |
          ↓
        ECR

This is a broader GitHub Actions security topic and is covered in more detail in the authentication sections of your notes.

---

# Trivy and Docker Build Pipeline

A practical pipeline:

    Git Push
       |
       ↓
    Checkout
       |
       ↓
    Application Build
       |
       ↓
    Unit Tests
       |
       ↓
    SonarQube
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
       +-- Fail → Stop
       |
       +-- Pass
             |
             ↓
            ECR

---

# Trivy and ArgoCD

ArgoCD should not be responsible for building the container image.

Typical separation:

    GitHub Actions
         |
         +-- Build
         +-- Test
         +-- SonarQube
         +-- Docker Build
         +-- Trivy
         +-- Push ECR
         |
         ↓
    GitOps Repository
         |
         ↓
    ArgoCD
         |
         ↓
    EKS

CI creates and validates the artifact.

ArgoCD handles GitOps-based deployment.

---

# Trivy and Kubernetes Deployment

Recommended flow:

    Application Code
         |
         ↓
    GitHub Actions
         |
         ↓
    Docker Image
         |
         ↓
    Trivy
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

The same scanned image can then be deployed.

---

# Trivy and Terraform

Terraform configuration can be scanned before infrastructure deployment.

Example:

    Terraform
       |
       ↓
    Trivy Config
       |
       ↓
    Misconfiguration
       |
       ↓
    Security Gate
       |
       ↓
    Terraform Plan / Apply

This provides IaC security checks before infrastructure changes are applied.

---

# Trivy and Dockerfile

Trivy can detect applicable configuration and security issues in Docker-related files when the relevant scanners are enabled.

Example:

    Dockerfile
        |
        ↓
    Trivy
        |
        ↓
    Misconfiguration Result

This can complement the actual image vulnerability scan.

---

# Image Scan vs Dockerfile Scan

Dockerfile scan:

    Configuration
        |
        ↓
    Misconfiguration

Image scan:

    Built Image
        |
        ↓
    OS / Library Vulnerabilities

Both can provide useful security coverage.

---

# Trivy Security Gate Design

A mature security gate can evaluate:

    Vulnerability
        |
        ↓
    Severity
        |
        ↓
    Fixed / Unfixed
        |
        ↓
    Organizational Policy
        |
        ↓
    Pass / Fail

Example policy:

    CRITICAL → Fail
    HIGH     → Fail
    MEDIUM   → Review
    LOW      → Informational

The actual policy should be defined by the organization.

---

# Do Not Blindly Fail on Every Vulnerability

A pipeline should have an intentional policy.

Example:

    Every Vulnerability
          |
          ↓
        Fail

may create excessive noise.

Instead, organizations may define:

    CRITICAL
       ↓
    Immediate Failure

    HIGH
       ↓
    Failure or Review

    MEDIUM
       ↓
    Review

    LOW
       ↓
    Informational

The correct threshold depends on risk tolerance.

---

# Vulnerability Remediation

When Trivy finds a vulnerability:

    Vulnerability
         |
         ↓
    Identify Package
         |
         ↓
    Identify Fixed Version
         |
         ↓
    Update Dependency
         |
         ↓
    Rebuild Image
         |
         ↓
    Rescan
         |
         ↓
    Pass

---

# Base Image Vulnerability

Example:

    FROM node:20-alpine

Trivy:

    Node Image
       |
       ↓
    OS Package
       |
       ↓
    CVE

Possible remediation:

    1. Update Base Image
    2. Rebuild
    3. Rescan

Do not immediately modify application code if the vulnerability comes from the base image.

---

# Application Dependency Vulnerability

Example:

    package-lock.json
          |
          ↓
    Vulnerable npm Package
          |
          ↓
    Trivy
          |
          ↓
    CVE

Remediation:

    Update Dependency
          |
          ↓
    npm install / npm ci
          |
          ↓
    Test
          |
          ↓
    Docker Build
          |
          ↓
    Trivy

---

# Vulnerability Has No Fix

If no fixed version exists:

    Vulnerability
        |
        ↓
    No Fix Available
        |
        ↓
    Risk Assessment
        |
        +-- Accept Temporarily
        +-- Mitigate
        +-- Replace Dependency
        +-- Monitor
        +-- Exception Process

Do not simply hide the vulnerability without documenting the reason.

---

# Trivy Ignore File

Trivy supports ignore configuration for findings that an organization has intentionally decided to suppress.

A common file is:

    .trivyignore

Example concept:

    CVE-YYYY-NNNN

The use of ignore rules should be controlled.

Avoid broad or permanent suppression without justification.

---

# Trivy Ignore Best Practices

If a finding is ignored:

- Document Why
- Document Owner
- Document Expiration
- Review Periodically
- Avoid Wildcard Suppression
- Prefer Specific Findings
- Reassess When a Fix Becomes Available

Security exceptions should not become permanent blind spots.

---

# Trivy and SBOM

Trivy can also work with Software Bill of Materials workflows.

An SBOM describes software components contained in an artifact.

Conceptually:

    Docker Image
        |
        ↓
    SBOM
        |
        +-- Packages
        +-- Versions
        +-- Components
        |
        ↓
    Security Analysis

SBOM generation and vulnerability scanning are related but are not the same thing.

---

# SBOM Generation

Trivy can generate SBOM-related output formats.

Example concept:

    trivy image \
      --format cyclonedx \
      --output sbom.json \
      myapp:v1.0.0

The exact output format should be selected based on the organization's SBOM requirements.

---

# Trivy and Supply Chain Security

Trivy can contribute to software supply-chain security by scanning:

- Dependencies
- Container Images
- Configuration
- Secrets
- Licenses

Typical flow:

    Source
       |
       ↓
    Dependency
       |
       ↓
    Build
       |
       ↓
    Image
       |
       ↓
    Trivy
       |
       ↓
    Security Gate

---

# Trivy in a Microservices Pipeline

Suppose the application has:

    user
    catalog
    cart
    order
    payment
    inventory

Each service can produce its own image.

Example:

    user:8f3a91d
    catalog:8f3a91d
    cart:8f3a91d
    order:8f3a91d
    payment:8f3a91d

Each image can be scanned independently.

---

# Microservices Matrix Scan

GitHub Actions matrix:

    strategy:
      matrix:
        service:
          - user
          - catalog
          - cart
          - order
          - payment

Conceptually:

    Matrix
      |
      +-- user
      +-- catalog
      +-- cart
      +-- order
      +-- payment

Each job can build and scan the corresponding service image.

---

# Matrix Build and Scan

Example:

    jobs:

      build-and-scan:

        strategy:
          matrix:
            service:
              - user
              - catalog
              - cart

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6

          - name: Build Image
            run: |
              docker build \
                -t ${{ matrix.service }}:${{ github.sha }} \
                ${{ matrix.service }}

          - name: Trivy Scan
            uses: aquasecurity/trivy-action@v0.36.0
            with:
              image-ref: ${{ matrix.service }}:${{ github.sha }}
              format: table
              exit-code: '1'
              severity: 'CRITICAL,HIGH'

---

# Trivy and Reusable Workflows

Trivy can be included in a reusable workflow.

Conceptually:

    Application Repository
          |
          ↓
    Reusable CI Workflow
          |
          +-- Build
          +-- Test
          +-- Trivy
          |
          ↓
    Result

This allows multiple repositories to follow a standardized security process.

---

# Reusable Security Workflow

Example concept:

    jobs:

      security:
        uses: organization/security-workflows/.github/workflows/trivy.yml@main

The reusable workflow can standardize:

- Trivy Version
- Severity Threshold
- Scan Type
- Exit Code
- Reporting
- Security Policy

Organizations should control changes to the reusable workflow.

---

# Trivy Workflow Permissions

A workflow should use least-privilege permissions.

Example:

    permissions:
      contents: read

If SARIF upload or another GitHub integration requires additional permissions, grant only the permissions needed for that feature.

---

# Trivy Action Pinning

Example:

    uses: aquasecurity/trivy-action@v0.36.0

Organizations with stricter supply-chain requirements may pin actions to immutable commit SHAs.

The important principle is:

    Controlled Action Version
           |
           ↓
    Predictable CI Behavior

---

# Why Avoid Uncontrolled Action References?

Using an uncontrolled reference can introduce unexpected changes into the pipeline.

Prefer:

    @v0.36.0

or an approved commit SHA.

The organization's security policy should define the exact standard.

---

# Trivy Action Inputs

Common inputs include:

    scan-type

    scan-ref

    image-ref

    format

    exit-code

    severity

    ignore-unfixed

    vuln-type

Other inputs may be available depending on the Trivy Action version.

Always check the action version's documentation before using a new input.

---

# scan-type

Examples:

    scan-type: image

    scan-type: fs

    scan-type: config

The exact supported values depend on the action version.

---

# scan-ref

For filesystem scanning:

    scan-ref: .

Example:

    - name: Trivy Filesystem Scan
      uses: aquasecurity/trivy-action@v0.36.0
      with:
        scan-type: fs
        scan-ref: .
        exit-code: '1'
        severity: 'CRITICAL,HIGH'

---

# image-ref

For Docker image scanning:

    image-ref: myapp:${{ github.sha }}

Example:

    - name: Trivy Image Scan
      uses: aquasecurity/trivy-action@v0.36.0
      with:
        image-ref: myapp:${{ github.sha }}
        exit-code: '1'
        severity: 'CRITICAL,HIGH'

---

# Trivy Configuration Example

A project may maintain Trivy configuration in the repository.

Possible files include:

    trivy.yaml

    .trivyignore

The exact configuration options should match the installed Trivy version.

---

# Trivy Configuration Management

Configuration should be:

- Version Controlled
- Reviewed
- Documented
- Tested
- Updated

Avoid changing security thresholds directly in an emergency without documenting the reason.

---

# Trivy CI Best Practices

- Scan before image publishing
- Use versioned images
- Scan OS packages
- Scan application libraries
- Scan IaC where appropriate
- Scan repositories where appropriate
- Use severity thresholds
- Use exit codes as security gates
- Keep vulnerability databases current
- Cache carefully
- Store scan reports when useful
- Review ignored vulnerabilities
- Do not hardcode credentials
- Pin action versions according to policy
- Use least-privilege GitHub permissions
- Rebuild after dependency updates

---

# Trivy Pipeline Best Practice

Recommended:

    Checkout
       |
       ↓
    Build Application
       |
       ↓
    Test
       |
       ↓
    SonarQube
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

# Common Trivy Problems

Common issues include:

- Trivy Database Download Failure
- Network Connectivity Problem
- Docker Image Not Found
- Incorrect Image Tag
- Docker Daemon Access Problem
- Private Registry Authentication Failure
- Excessive Vulnerability Findings
- False Positives / Matching Issues
- Old Trivy Version
- Stale Cache
- Incorrect Scan Type
- Incorrect Severity Configuration
- Incorrect Exit Code
- SARIF Upload Failure

---

# Troubleshooting: Image Not Found

Error concept:

    image not found

Check:

    docker images

Then:

    docker image inspect myapp:${{ github.sha }}

Verify that the image was built with the same tag used by Trivy.

---

# Troubleshooting: Wrong Image Tag

Build:

    docker build -t myapp:${{ github.sha }} .

Scan:

    image-ref: myapp:${{ github.sha }}

Both must match.

Bad:

    Build → myapp:latest
    Scan  → myapp:${{ github.sha }}

Good:

    Build → myapp:${{ github.sha }}
    Scan  → myapp:${{ github.sha }}

---

# Troubleshooting: Trivy Database Failure

Possible causes:

- Network Failure
- Registry Access
- Proxy
- DNS
- Temporary Service Problem
- Cache Problem

Check the Trivy logs and runner network connectivity.

---

# Troubleshooting: Docker Socket

If Trivy is scanning a local Docker image, the scanner needs access to the container image source.

When using Trivy as a container, access to the Docker engine may require mounting the Docker socket.

When using the GitHub Action on a GitHub-hosted runner, the action handles the supported setup according to its implementation.

---

# Troubleshooting: Private Registry

If scanning a private registry image:

    Private Registry
          |
          ↓
    Authentication
          |
          ↓
    Trivy
          |
          ↓
    Scan

Verify:

- Registry Login
- Credentials
- Permissions
- Image Name
- Network Access

---

# Troubleshooting: Too Many Vulnerabilities

If thousands of vulnerabilities appear:

Check:

    1. Base Image
    2. OS Version
    3. Dependency Versions
    4. Scanner Version
    5. Severity Threshold
    6. Whether Unfixed Vulnerabilities Are Included

Then determine which findings are actually actionable.

Do not blindly suppress the entire scan.

---

# Troubleshooting: False Positive

If a finding appears incorrect:

    1. Verify Package
    2. Verify Installed Version
    3. Verify Vulnerability
    4. Check Trivy Matching Information
    5. Check Vendor Advisory
    6. Determine Actual Exposure
    7. Apply an approved exception only if justified

---

# Troubleshooting: Scan Takes Too Long

Possible causes:

- Large Image
- Large Dependency Tree
- Database Download
- No Cache
- Large Filesystem
- Multiple Scanner Types
- Slow Network

Possible improvements:

- Cache appropriately
- Reduce Image Size
- Reduce Build Context
- Avoid Unnecessary Files
- Use Appropriate Scan Target
- Avoid Repeated Database Downloads

---

# Trivy and Large Docker Images

A large image can contain:

- More OS Packages
- More Libraries
- More Files
- More Potential Vulnerabilities

Therefore:

    Smaller Runtime Image
           |
           ↓
    Smaller Attack Surface

Use multi-stage builds where appropriate.

---

# Trivy and Multi-Stage Docker Builds

Example:

    Build Stage
        |
        +-- Compiler
        +-- Build Tools
        +-- Dev Dependencies
        |
        ↓
    Runtime Stage
        |
        +-- Runtime
        +-- Application
        |
        ↓
    Final Image
        |
        ↓
    Trivy

The final runtime image should be scanned because it is what will actually be deployed.

---

# Scan the Final Image

Do not rely only on scanning the build environment.

Preferred:

    Build
      |
      ↓
    Final Runtime Image
      |
      ↓
    Trivy
      |
      ↓
    ECR

The final image is the artifact that will be deployed.

---

# Trivy with JFrog Artifactory

A private registry such as JFrog Artifactory can store container images.

Flow:

    GitHub Actions
         |
         ↓
    Docker Build
         |
         ↓
    Trivy
         |
         ↓
    Artifactory
         |
         ↓
    Deployment

Authentication should be handled securely.

---

# Trivy with GitHub Container Registry

Another option is GitHub Container Registry.

Flow:

    GitHub Actions
         |
         ↓
    Docker Build
         |
         ↓
    Trivy
         |
         ↓
    GHCR
         |
         ↓
    Deployment

Registry credentials should be handled using GitHub's supported authentication mechanisms.

---

# Trivy in a Java Pipeline

Example:

    Checkout
       |
       ↓
    Maven Build
       |
       ↓
    Unit Test
       |
       ↓
    SonarQube
       |
       ↓
    Docker Build
       |
       ↓
    Trivy
       |
       ↓
    ECR

---

# Trivy in a Node.js Pipeline

Example:

    Checkout
       |
       ↓
    npm ci
       |
       ↓
    npm test
       |
       ↓
    SonarQube
       |
       ↓
    Docker Build
       |
       ↓
    Trivy
       |
       ↓
    ECR

---

# Trivy in a Python Pipeline

Example:

    Checkout
       |
       ↓
    pip install
       |
       ↓
    pytest
       |
       ↓
    SonarQube
       |
       ↓
    Docker Build
       |
       ↓
    Trivy
       |
       ↓
    ECR

---

# Complete GitHub Actions DevSecOps Workflow

Example:

    name: DevSecOps CI

    on:
      push:
        branches:
          - main

      pull_request:

    permissions:
      contents: read

    jobs:

      build-test-scan:

        runs-on: ubuntu-latest

        steps:

          - name: Checkout
            uses: actions/checkout@v6
            with:
              fetch-depth: 0

          - name: Setup Node.js
            uses: actions/setup-node@v4
            with:
              node-version: '20'
              cache: npm

          - name: Install Dependencies
            run: npm ci

          - name: Run Tests
            run: npm test -- --coverage

          - name: SonarQube
            uses: SonarSource/sonarqube-scan-action@v8.2.1
            env:
              SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}

          - name: Docker Build
            run: |
              docker build \
                -t myapp:${{ github.sha }} \
                .

          - name: Trivy Scan
            uses: aquasecurity/trivy-action@v0.36.0
            with:
              image-ref: myapp:${{ github.sha }}
              format: table
              exit-code: '1'
              severity: 'CRITICAL,HIGH'
              vuln-type: 'os,library'

---

# Complete CI Security Flow

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    GitHub Actions
        |
        +-- Checkout
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
        ↓
    Security / Quality Gates
        |
        +-- Fail → Stop
        |
        +-- Pass
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

# Scenario Interview Question

## A Docker image contains CRITICAL vulnerabilities. What would you do?

I would:

    1. Identify the vulnerable package
    2. Determine whether it comes from the base image or application dependency
    3. Check whether a fixed version exists
    4. Update the base image or dependency
    5. Rebuild the image
    6. Run tests
    7. Run Trivy again
    8. Verify the security gate
    9. Push only the validated image

---

# Scenario Interview Question

## Trivy works locally but fails in GitHub Actions. How would you troubleshoot it?

I would compare:

- Trivy Version
- Docker Version
- Image Tag
- Runner Environment
- Network
- Database Access
- Docker Daemon
- Registry Access
- Action Version
- Scan Configuration

First I would inspect the GitHub Actions logs and verify that the image exists in the runner.

---

# Scenario Interview Question

## Trivy reports hundreds of vulnerabilities. What would you do?

I would not immediately disable the scan.

I would:

    1. Identify the source of vulnerabilities
    2. Separate OS and library findings
    3. Review severity
    4. Check whether fixes exist
    5. Update the base image
    6. Update dependencies
    7. Remove unnecessary packages
    8. Rebuild
    9. Rescan

Then I would apply the organization's security policy.

---

# Scenario Interview Question

## How would you prevent vulnerable images from reaching ECR?

I would use:

    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Security Gate
        |
        +-- Fail → Stop
        |
        +-- Pass
              |
              ↓
             ECR

The Docker push step should depend on the successful scan.

---

# Scenario Interview Question

## How would you integrate Trivy with a microservices CI pipeline?

I would use either separate jobs or a matrix strategy.

Example:

    Services
       |
       +-- User
       +-- Catalog
       +-- Cart
       +-- Order
       +-- Payment
       |
       ↓
    Build Each Image
       |
       ↓
    Trivy Scan
       |
       ↓
    Security Gate
       |
       ↓
    Push Valid Images

---

# Scenario Interview Question

## How would you scan Terraform before applying it?

I would add a Trivy configuration scan before Terraform apply.

Flow:

    Terraform
        |
        ↓
    Trivy Config Scan
        |
        ↓
    Security Gate
        |
        +-- Fail → Stop
        |
        +-- Pass
              |
              ↓
         Terraform Plan
              |
              ↓
         Terraform Apply

---

# Scenario Interview Question

## How would you scan Kubernetes manifests before deployment?

I would scan the manifests before ArgoCD or another deployment mechanism applies them.

Flow:

    Kubernetes YAML
          |
          ↓
        Trivy
          |
          ↓
    Misconfiguration
          |
          ↓
    Security Gate
          |
          +-- Pass → GitOps
          |
          +-- Fail → Stop

---

# Scenario Interview Question

## Why should Trivy run before Docker Push?

Because the security gate can prevent a non-compliant artifact from entering the registry.

Preferred:

    Build
      |
      ↓
    Scan
      |
      ↓
    Push

Instead of:

    Build
      |
      ↓
    Push
      |
      ↓
    Scan

---

# Scenario Interview Question

## How would you handle an unfixed vulnerability?

I would:

    1. Confirm the vulnerability
    2. Check whether a fix exists
    3. Assess actual exposure
    4. Determine available mitigations
    5. Document the risk
    6. Follow the organization's exception process
    7. Continue monitoring for a fix

I would not permanently ignore it without justification.

---

# Scenario Interview Question

## What is the difference between Trivy image and filesystem scanning?

Image scanning:

    trivy image myapp:v1.0.0

Focuses on the built container image and its detected components.

Filesystem scanning:

    trivy fs .

Focuses on files in a local project and can detect supported vulnerabilities, secrets, misconfigurations, and licenses.

---

# Scenario Interview Question

## What is the difference between vulnerability scanning and misconfiguration scanning?

Vulnerability scanning:

    Known Security Vulnerability
          |
          ↓
    Package / Component
          |
          ↓
    CVE / Finding

Misconfiguration scanning:

    Configuration
          |
          ↓
    Insecure Setting
          |
          ↓
    Misconfiguration Finding

Both can be useful in DevSecOps.

---

# Scenario Interview Question

## Why is the final Docker image the most important image to scan?

Because the final image is the artifact that will be deployed.

Flow:

    Build Stage
       |
       ↓
    Final Runtime Image
       |
       ↓
    Trivy
       |
       ↓
    ECR
       |
       ↓
    Production

Scanning only intermediate build layers does not replace scanning the actual deployment artifact.

---

# Scenario Interview Question

## How would you optimize Trivy in GitHub Actions?

I would consider:

- Database Caching
- Appropriate Scan Target
- Smaller Images
- Avoiding Unnecessary Files
- Controlled Scanner Selection
- Appropriate Severity
- Reusing Build Artifacts
- Avoiding Duplicate Scans
- Keeping Trivy Updated

The goal is:

    Fast CI
       +
    Current Security Data
       +
    Useful Findings

---

# Scenario Interview Question

## How would you design a production DevSecOps pipeline using SonarQube and Trivy?

I would use:

    Developer
       |
       ↓
    Pull Request
       |
       ↓
    GitHub Actions
       |
       +-- Checkout
       +-- Build
       +-- Test
       +-- SonarQube
       +-- Quality Gate
       +-- Docker Build
       +-- Trivy
       +-- Security Gate
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

SonarQube handles source-code quality/security analysis.

Trivy handles container/dependency/configuration security scanning according to the selected scan targets.

---

# Best Practices Checklist

- Scan images before publishing
- Use versioned image tags
- Prefer immutable image references
- Scan the final runtime image
- Scan OS packages
- Scan application libraries
- Scan IaC where appropriate
- Scan repositories where appropriate
- Use severity thresholds
- Use exit-code to enforce policy
- Keep Trivy databases current
- Use caching carefully
- Store reports when required
- Use .trivyignore carefully
- Review ignored findings
- Do not hardcode credentials
- Use GitHub Secrets for sensitive values
- Use least-privilege permissions
- Control Trivy Action versions
- Keep Trivy updated
- Rebuild images after dependency updates
- Do not push images that fail mandatory security gates
- Use the same validated image for deployment

---

# Important Commands

Basic image scan:

    trivy image myapp:v1.0.0

Severity:

    trivy image \
      --severity HIGH,CRITICAL \
      myapp:v1.0.0

Exit code:

    trivy image \
      --exit-code 1 \
      --severity HIGH,CRITICAL \
      myapp:v1.0.0

Filesystem:

    trivy fs .

Repository:

    trivy repo .

Configuration:

    trivy config .

Secret scan:

    trivy fs --scanners secret .

Multiple scanners:

    trivy fs \
      --scanners vuln,misconfig,secret \
      .

SARIF:

    trivy image \
      --format sarif \
      --output trivy-results.sarif \
      myapp:v1.0.0

JSON:

    trivy image \
      --format json \
      --output trivy-results.json \
      myapp:v1.0.0

---

# Important GitHub Actions Syntax

Action:

    uses: aquasecurity/trivy-action@v0.36.0

Image:

    image-ref: myapp:${{ github.sha }}

Filesystem:

    scan-type: fs

Filesystem path:

    scan-ref: .

Output:

    format: table

Pipeline failure:

    exit-code: '1'

Severity:

    severity: 'CRITICAL,HIGH'

Vulnerability types:

    vuln-type: 'os,library'

Ignore unfixed:

    ignore-unfixed: true

---

# Important Trivy Concepts

- Trivy
- Vulnerability Scanning
- Container Image Scanning
- Filesystem Scanning
- Repository Scanning
- Configuration Scanning
- Secret Scanning
- License Scanning
- Misconfiguration
- CVE
- Severity
- CRITICAL
- HIGH
- MEDIUM
- LOW
- Exit Code
- Security Gate
- Trivy Database
- Trivy Cache
- .trivyignore
- SBOM
- SARIF
- Docker
- ECR
- GitHub Actions
- GitHub Secrets
- GitHub Security
- Terraform
- Kubernetes
- SonarQube
- DevSecOps
- ArgoCD
- EKS

---

# Quick Revision

Remember the GitHub Actions Trivy flow:

    Git Push
       |
       ↓
    GitHub Actions
       |
       ↓
    Checkout
       |
       ↓
    Build Application
       |
       ↓
    Test
       |
       ↓
    SonarQube
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
       +-- Fail → Stop
       |
       +-- Pass
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

Core idea:

Trivy adds automated security scanning to GitHub Actions. In a DevSecOps pipeline, it can scan the final Docker image, dependencies, repositories, filesystems, infrastructure configuration, secrets, and other supported targets. The pipeline can use severity thresholds and exit codes to turn security findings into an automated gate before an image is published or deployed.