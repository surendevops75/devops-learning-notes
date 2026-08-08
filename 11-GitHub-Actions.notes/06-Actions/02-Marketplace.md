# GitHub Actions Marketplace

GitHub Actions Marketplace is a catalog where you can discover reusable Actions created by GitHub, organizations, and third-party developers.

Marketplace Actions can add functionality to workflows without requiring you to implement everything from scratch.

Common use cases:

```text
Code Checkout
Cloud Authentication
Build
Testing
Security Scanning
Docker
Kubernetes
Terraform
Notifications
Artifact Management
Deployment
```

---

# Why Use the Marketplace?

Without reusable Actions, you may need to implement common functionality yourself.

For example:

```text
Checkout repository
      |
      ↓
Write Git commands
      |
      ↓
Handle authentication
      |
      ↓
Handle credentials
      |
      ↓
Handle edge cases
```

Instead:

```yaml
- name: Checkout
  uses: actions/checkout@v4
```

The Action provides the reusable implementation.

---

# Marketplace Workflow

Conceptually:

```text
GitHub Actions Marketplace
          |
          ↓
Discover Action
          |
          ↓
Review Action
          |
          ↓
Select Version
          |
          ↓
Add to Workflow
          |
          ↓
Test
          |
          ↓
Production
```

Do not treat Marketplace discovery as automatic approval.

---

# Marketplace Action Example

A common Action:

```yaml
- name: Checkout Code
  uses: actions/checkout@v4
```

Another:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: '21'
```

Another:

```yaml
- name: Upload Artifact
  uses: actions/upload-artifact@v4
  with:
    name: application
    path: target/
```

---

# Marketplace Categories

Marketplace contains Actions for areas such as:

```text
CI/CD
AWS
Azure
Google Cloud
Docker
Kubernetes
Terraform
Security
Testing
Code Quality
Monitoring
Notifications
Artifacts
Deployment
```

---

# Finding an Action

When looking for an Action, define the requirement first.

Example:

```text
Requirement:
Authenticate GitHub Actions with AWS
```

Then search for:

```text
AWS authentication GitHub Actions
```

Do not immediately choose the first result.

---

# Evaluate the Action

Before using a Marketplace Action, review:

```text
Repository
Maintainer
Source Code
Version
Release History
Documentation
Issues
Security
Permissions
Dependencies
Usage
```

---

# Official vs Third-Party Actions

There are different sources of Actions.

### Official GitHub Actions

Examples:

```text
actions/checkout
actions/setup-java
actions/setup-node
actions/upload-artifact
```

These are commonly used GitHub-maintained Actions.

### Third-Party Actions

Examples include Actions maintained by:

```text
Cloud vendors
Security vendors
Open-source organizations
Individual developers
Companies
```

Review them carefully before production use.

---

# Marketplace Does Not Mean Trusted

Important principle:

```text
Marketplace
    ≠
Automatically Trusted
```

An Action is executable code.

Therefore:

```text
Marketplace Action
       |
       ↓
Workflow Runner
       |
       ↓
Permissions / Secrets / Network
```

A compromised Action can potentially affect the workflow environment according to the permissions available to it.

---

# Action Source Code

Prefer Actions whose source code is publicly available and reviewable.

Example:

```text
Marketplace
    |
    ↓
Action Repository
    |
    ├── action.yml
    ├── Source Code
    ├── Tests
    └── Documentation
```

Review the repository before adopting an unfamiliar Action.

---

# `action.yml`

An Action normally describes its metadata through an `action.yml` or `action.yaml` file.

Conceptually:

```yaml
name: My Action

description: Example custom action

inputs:
  environment:
    description: Target environment
    required: true

outputs:
  result:
    description: Action result
```

The metadata defines how the Action is consumed.

---

# Marketplace Action Documentation

Before using an Action, read:

```text
Usage
Inputs
Outputs
Examples
Required Permissions
Environment Variables
Secrets
Supported Runners
Version Information
```

Example:

```yaml
- name: Example Action
  uses: owner/action@v1
  with:
    environment: production
```

---

# Inputs

Marketplace Actions often expose inputs.

Example:

```yaml
- name: Setup Java
  uses: actions/setup-java@v4
  with:
    distribution: temurin
    java-version: '21'
```

Inputs:

```text
distribution
java-version
```

Always check the Action documentation for valid input names and values.

---

# Required Inputs

Some Actions require inputs.

Example:

```yaml
with:
  token: ${{ secrets.TOKEN }}
```

If the required input is missing, the Action may fail.

Always verify:

```text
Required
Optional
Default
Allowed values
```

---

# Outputs

Some Marketplace Actions expose outputs.

Example concept:

```yaml
- name: Generate Version
  id: version
  uses: owner/version-action@v1
```

Later:

```yaml
run: echo "${{ steps.version.outputs.version }}"
```

Check the Action documentation for the exact output names.

---

# Action Version

Marketplace Actions are referenced using:

```yaml
uses: owner/repository@ref
```

Examples:

```yaml
uses: actions/checkout@v4
```

or:

```yaml
uses: owner/action@v2
```

or a specific commit:

```yaml
uses: owner/action@<commit-sha>
```

---

# Versioning Strategies

Common strategies:

```text
Major Version
@v4

Specific Release
@v4.x.x

Commit SHA
@<sha>
```

Each has different maintenance and security characteristics.

---

# Why Avoid `@main`?

Example:

```yaml
uses: owner/action@main
```

The branch can change over time.

Therefore:

```text
Today
  ↓
Code A

Later
  ↓
Code B
```

The same workflow reference can execute different code.

This can affect:

```text
Reproducibility
Stability
Security
Change Management
```

---

# Production Version Strategy

For production workflows:

```text
Do not blindly use moving branches.
```

Use an organization-approved strategy such as:

```text
Reviewed release tag
```

or:

```text
Reviewed commit SHA
```

For higher-assurance environments, SHA pinning provides stronger immutability.

---

# SHA Pinning

Example:

```yaml
uses: owner/action@<commit-sha>
```

The reference points to a specific commit.

Benefits:

```text
Predictability
Immutability
Supply-chain protection
Reproducibility
```

Trade-off:

```text
Updates must be intentionally reviewed and changed.
```

---

# Action Pinning Policy

An organization may define:

```text
Official Actions
    |
    └── Approved major version

Third-Party Actions
    |
    └── Security review + SHA pinning

Internal Actions
    |
    └── Approved release process
```

The exact policy depends on organizational requirements.

---

# Marketplace Action Security Review

Before approving an Action:

```text
1. Identify owner
2. Review source repository
3. Check maintenance activity
4. Review releases
5. Review dependencies
6. Check permissions
7. Check secrets
8. Check network behavior
9. Check security history
10. Test in non-production
```

---

# Maintainer Reputation

Review:

```text
Who maintains it?
Is the repository active?
Are issues addressed?
Are releases maintained?
Is there a documented security process?
```

Avoid choosing an Action purely because:

```text
It has many stars.
```

Popularity is useful context, but not a security guarantee.

---

# Release History

Review release history.

Example:

```text
v1.0
v1.1
v1.2
v2.0
```

Check:

```text
Breaking changes
Security fixes
Deprecations
Runner compatibility
Input changes
```

---

# Issues and Pull Requests

Reviewing repository issues can reveal:

```text
Known bugs
Security concerns
Compatibility problems
Unmaintained functionality
Breaking changes
```

This is especially useful for third-party Actions.

---

# Dependencies

An Action can depend on other software.

Conceptually:

```text
Marketplace Action
      |
      ├── Dependency A
      ├── Dependency B
      └── Dependency C
```

The dependency chain becomes part of the supply chain.

---

# Supply Chain

A production workflow can look like:

```text
Source Code
    |
    ↓
Workflow
    |
    ↓
Marketplace Action
    |
    ↓
Action Dependencies
    |
    ↓
Build
    |
    ↓
Artifact
```

Every layer should be considered during security reviews.

---

# Marketplace Action with Secrets

Example:

```yaml
- name: Deploy
  uses: owner/deploy-action@v1
  with:
    token: ${{ secrets.DEPLOY_TOKEN }}
```

Before doing this, verify:

```text
Why is the token required?
Where is it used?
Can it be replaced with OIDC?
Does the Action log the token?
Does the Action send data externally?
```

---

# Prefer Short-Lived Credentials

For cloud deployments, prefer identity federation such as GitHub OIDC when supported.

Conceptually:

```text
GitHub Workflow
      |
      ↓
OIDC
      |
      ↓
Cloud IAM
      |
      ↓
Temporary Credentials
```

This is generally preferable to storing long-lived cloud access keys.

---

# Marketplace Action + AWS

Example concept:

```text
GitHub Actions
      |
      ↓
AWS Authentication Action
      |
      ↓
OIDC
      |
      ↓
AWS IAM Role
      |
      ↓
AWS Resources
```

Review the Action and IAM trust policy together.

---

# Marketplace Action + ECR

Example workflow concept:

```text
Checkout
   |
   ↓
Build Docker Image
   |
   ↓
AWS Authentication
   |
   ↓
ECR Login
   |
   ↓
Push Image
```

Marketplace Actions may be used for authentication or registry operations.

---

# Marketplace Action + EKS

Example:

```text
Build
  |
  ↓
Security Scan
  |
  ↓
Push Image to ECR
  |
  ↓
Authenticate
  |
  ↓
Helm / kubectl
  |
  ↓
EKS
```

For a GitOps architecture:

```text
Build
  |
  ↓
ECR
  |
  ↓
Git Manifest
  |
  ↓
ArgoCD
  |
  ↓
EKS
```

This can reduce direct cluster privileges on the GitHub runner.

---

# Marketplace Action + Terraform

A Terraform workflow may use Actions for:

```text
Terraform setup
AWS authentication
Caching
Security scanning
Plan reporting
```

Example:

```yaml
- name: Setup Terraform
  uses: hashicorp/setup-terraform@v3
```

Then:

```yaml
- name: Terraform Init
  run: terraform init

- name: Terraform Plan
  run: terraform plan
```

---

# Marketplace Action + Security

Security-related Actions can support:

```text
SAST
SCA
Container Scanning
Secret Scanning
IaC Scanning
Dependency Scanning
DAST
```

However, a security Action should not automatically become a release gate.

Define:

```text
Severity thresholds
Failure conditions
Approval process
Exception process
Reporting
```

---

# DevSecOps Example

```text
Checkout
    |
    ↓
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
Docker Build
    |
    ↓
Container Scan
    |
    ↓
Push ECR
```

Marketplace Actions may support some of these integrations.

---

# Security Gate

Example:

```text
Security Scan
      |
      ↓
Critical Vulnerability?
      |
   ┌──┴──┐
  YES    NO
   |      |
   ↓      ↓
 FAIL    Continue
```

Do not hide critical security failures with:

```yaml
continue-on-error: true
```

unless the exception is intentional and governed.

---

# Action Permissions

Workflows should explicitly define permissions where practical.

Example:

```yaml
permissions:
  contents: read
```

For OIDC:

```yaml
permissions:
  contents: read
  id-token: write
```

Do not grant unnecessary write permissions.

---

# `GITHUB_TOKEN`

Marketplace Actions may interact with GitHub using:

```text
GITHUB_TOKEN
```

Its permissions are controlled by the workflow/job configuration.

Example:

```yaml
permissions:
  contents: read
```

If an Action needs to create a release or modify repository contents, it may require additional permissions.

Only grant what is necessary.

---

# Dangerous Configuration

Avoid:

```yaml
permissions: write-all
```

unless there is a strong and documented reason.

A compromised Action with broad permissions can have a larger impact.

---

# Action Allowlisting

Enterprise organizations may maintain an approved Action list.

Example:

```text
Approved:

actions/*
hashicorp/*
company/*
approved-security/*
```

Unapproved:

```text
Other third-party sources
```

may require security review.

---

# Internal Marketplace Governance

A platform team can define:

```text
Approved Actions
       |
       ├── CI
       ├── Security
       ├── Cloud
       ├── Docker
       └── Deployment
```

Application teams consume approved building blocks.

---

# Marketplace and Platform Engineering

A platform engineering team can standardize:

```text
Java Build Action
Docker Build Action
Terraform Action
Security Scan Action
ECR Push Action
Helm Deployment Action
```

Then application teams use:

```yaml
uses: company/platform-actions/docker-build@v1
```

This creates a standardized CI/CD platform.

---

# Marketplace Action Governance

Maintain an inventory:

| Action | Purpose | Owner | Version | Security Status |
|---|---|---|---|---|
| checkout | Repository checkout | GitHub | Approved | Approved |
| setup-java | Java setup | GitHub | Approved | Approved |
| terraform | Terraform setup | HashiCorp | Approved | Reviewed |
| security scan | Security | Vendor | Reviewed | Approved |

---

# Action Lifecycle

Treat Marketplace Actions like dependencies.

```text
Discover
   |
   ↓
Evaluate
   |
   ↓
Approve
   |
   ↓
Use
   |
   ↓
Monitor
   |
   ↓
Update
   |
   ↓
Retire
```

---

# Updating an Action

Suppose:

```text
Current:
action@v1

New:
action@v2
```

Process:

```text
Read release notes
      |
      ↓
Review breaking changes
      |
      ↓
Update branch
      |
      ↓
Run CI
      |
      ↓
Security test
      |
      ↓
Code review
      |
      ↓
Production
```

---

# Emergency Security Update

If a vulnerability is discovered:

```text
Action Vulnerability
        |
        ↓
Identify affected repositories
        |
        ↓
Find approved version
        |
        ↓
Test
        |
        ↓
Update
        |
        ↓
Deploy
```

Organizations should automate dependency discovery where practical.

---

# Dependabot and Actions

GitHub dependency management can help identify updates to workflow dependencies and Actions.

A production organization can use automated update workflows with review.

Conceptually:

```text
Action Version
      |
      ↓
New Release
      |
      ↓
Dependency Update
      |
      ↓
Pull Request
      |
      ↓
Tests
      |
      ↓
Review
      |
      ↓
Merge
```

---

# Action Security Alert Example

A production pipeline uses:

```yaml
uses: vendor/security-action@v1
```

A security issue is discovered.

Response:

```text
1. Identify affected version
2. Check advisory
3. Find fixed version
4. Test
5. Update all repositories
6. Review permissions
7. Rotate credentials if exposure occurred
8. Monitor deployments
```

---

# Action Failure Troubleshooting

When an Action fails:

```text
Workflow
   |
   ↓
Action
   |
   ↓
Failure
```

Check:

```text
Action version
Inputs
Outputs
Permissions
Secrets
Runner OS
Dependencies
Network
Documentation
Release notes
```

---

# Example: Authentication Failure

Suppose:

```text
AWS authentication failed
```

Check:

```text
OIDC permission
IAM trust policy
Repository / environment conditions
Role ARN
Action version
AWS account
Runner connectivity
```

Example:

```yaml
permissions:
  contents: read
  id-token: write
```

Without the required OIDC permission, the authentication flow may fail.

---

# Example: Permission Failure

Suppose an Action tries to create a release:

```text
Resource not accessible
```

Check:

```yaml
permissions:
  contents: write
```

But do not blindly add write permission.

First verify:

```text
Does the Action really need it?
Can the operation be redesigned?
Can permissions be scoped at job level?
```

---

# Job-Level Permissions

Instead of broad workflow permissions:

```yaml
permissions:
  contents: write
```

consider limiting permissions to the specific job when appropriate.

Example:

```yaml
jobs:

  release:

    permissions:
      contents: write
```

Other jobs can remain read-only.

---

# Marketplace Action in a Job

```yaml
jobs:

  scan:

    permissions:
      contents: read

    runs-on: ubuntu-latest

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Scan
        uses: vendor/security-action@<reviewed-version>
```

This follows the principle:

```text
Minimum permissions
for
Minimum scope
```

---

# Marketplace Action with Production Environment

```yaml
jobs:

  deploy:

    environment:
      name: production

    permissions:
      contents: read
      id-token: write

    runs-on:
      - self-hosted
      - linux
      - production

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Authenticate
        uses: approved/cloud-auth-action@<reviewed-version>

      - name: Deploy
        run: ./deploy.sh
```

This combines:

```text
Environment protection
+
Runner controls
+
OIDC
+
Marketplace/internal Actions
```

---

# Third-Party Action Review Example

Suppose you find:

```yaml
uses: vendor/deploy-action@v1
```

Before production:

```text
Repository
   ↓
Source code
   ↓
Maintainer
   ↓
Release history
   ↓
Dependencies
   ↓
Permissions
   ↓
Secrets
   ↓
Network
   ↓
Security history
   ↓
Test
```

Then approve or reject.

---

# Questions to Ask Before Approval

```text
Who owns the Action?

Is the source public?

Is it actively maintained?

What permissions does it require?

Does it access secrets?

Does it send data externally?

What dependencies does it use?

Can we pin the Action?

Does it support our runner?

Does it support our operating system?

What happens if the Action is compromised?
```

---

# Action Supply Chain Attack

Conceptually:

```text
Third-Party Action
       |
       ↓
Compromised Dependency
       |
       ↓
Malicious Code
       |
       ↓
Runner
       |
       ↓
Secrets / Token / Cloud
```

This is why Action selection is a security decision.

---

# Minimize Blast Radius

If an Action is compromised:

```text
Limited Permissions
       +
Limited Secrets
       +
Limited Runner Access
       +
Limited Network Access
```

can reduce the impact.

This is the principle of defense in depth.

---

# Untrusted Pull Request

Do not expose sensitive production Actions/secrets to arbitrary PR code.

Better architecture:

```text
External PR
     |
     ↓
Isolated CI
     |
     ↓
Tests
     |
     ↓
Review
     |
     ↓
Merge
     |
     ↓
Protected Production Workflow
     |
     ↓
Production Actions
```

---

# Marketplace + Self-Hosted Runners

If an Action executes on a self-hosted runner:

```text
Marketplace Action
       |
       ↓
Self-Hosted Runner
       |
       ↓
Private Infrastructure
```

The risk can be greater than on a disposable isolated runner because the runner may have access to internal resources.

Use strong runner isolation and permissions.

---

# Marketplace + ARC

With ARC:

```text
Marketplace Action
       |
       ↓
ARC Runner Pod
       |
       ↓
Kubernetes
```

Security controls include:

```text
Runner Groups
Labels
Ephemeral lifecycle
Kubernetes RBAC
Network Policies
Pod Security
IAM
```

---

# Marketplace + GitOps

Recommended architecture:

```text
Marketplace / Internal Actions
           |
           ↓
ARC Runner
           |
      ┌────┴────┐
      ↓         ↓
    Build     Security
      |         |
      └────┬────┘
           ↓
          ECR
           |
           ↓
       Git Manifest
           |
           ↓
         ArgoCD
           |
           ↓
          EKS
```

This keeps deployment responsibilities separated.

---

# Marketplace Action Checklist

```text
Discovery
   ↓
Source Review
   ↓
Maintainer Review
   ↓
Version Review
   ↓
Permission Review
   ↓
Secret Review
   ↓
Dependency Review
   ↓
Security Testing
   ↓
Approval
   ↓
Production
```

---

# Best Practices

- Prefer official or internally approved Actions where appropriate.
- Review third-party Actions before use.
- Never assume Marketplace means trusted.
- Avoid moving branch references such as `@main` for production.
- Use a deliberate versioning strategy.
- Consider SHA pinning for higher assurance.
- Use least-privilege permissions.
- Avoid unnecessary secrets.
- Prefer OIDC for supported cloud authentication.
- Review Action dependencies.
- Monitor security advisories.
- Keep Actions updated.
- Test upgrades before production.
- Maintain an approved Action inventory.
- Separate production workflows from untrusted PR workflows.
- Use job-level permissions where appropriate.
- Treat Actions as software supply-chain dependencies.

---

# Common Mistakes

### 1. Selecting an Action only because it is popular

```text
Many stars ≠ guaranteed security
```

### 2. Using `@main`

```yaml
uses: vendor/action@main
```

### 3. Giving broad permissions

```yaml
permissions: write-all
```

### 4. Passing secrets unnecessarily

### 5. Using unreviewed third-party Actions in production

### 6. Ignoring dependencies

### 7. Never updating Actions

### 8. Automatically trusting Marketplace Actions

### 9. Exposing privileged Actions to untrusted PRs

### 10. Allowing production Actions to run with excessive runner privileges

---

# Production Marketplace Governance

A mature organization can maintain:

```text
Approved Action Registry
        |
        ├── Official Actions
        ├── Internal Actions
        ├── Security Actions
        ├── Cloud Actions
        └── Deployment Actions
```

Each entry can contain:

```text
Name
Repository
Owner
Version
SHA
Purpose
Permissions
Security Review
Approval Date
Update Owner
```

---

# Enterprise Example

```text
Platform Team
      |
      ↓
Approved Actions
      |
 ┌────┼────────┐
 ↓    ↓        ↓
CI   Security  Cloud
 |      |        |
 ↓      ↓        ↓
Build  Scan     AWS
 |
 ↓
Application Teams
```

This allows platform teams to provide standardized building blocks while maintaining governance.

---

# Interview Questions

## Basic

1. What is GitHub Actions Marketplace?
2. Why would you use Marketplace Actions?
3. What is the difference between an official Action and a third-party Action?
4. How do you reference a Marketplace Action?
5. What are Action inputs?
6. What are Action outputs?
7. What should you check before using a Marketplace Action?

## Intermediate

8. Why should you not blindly trust Marketplace Actions?
9. Why is `@main` not ideal for production workflows?
10. What is SHA pinning?
11. How do you evaluate a third-party Action?
12. How do Action permissions affect security?
13. How can Marketplace Actions access `GITHUB_TOKEN`?
14. How do you protect secrets when using Marketplace Actions?
15. How would you manage Action versions across an organization?
16. How would you troubleshoot a failing Marketplace Action?
17. What is an Action supply-chain risk?

## Advanced / Production

18. Design an enterprise governance model for Marketplace Actions.
19. How would you create an approved Action allowlist?
20. How would you securely use third-party Actions in production?
21. A third-party Action used by 100 repositories is compromised. Walk through your response.
22. How would you detect which repositories use a vulnerable Action version?
23. How would you roll out a security update to an Action across hundreds of repositories?
24. How would you combine Marketplace Actions with GitHub OIDC and AWS IAM?
25. How would you secure Marketplace Actions running on self-hosted runners?
26. How would you secure Marketplace Actions running on ARC-managed ephemeral runners?
27. How would you prevent an untrusted pull request from using a privileged Marketplace Action?
28. How would you design an internal Action platform for standardized Docker, Terraform, security, and deployment workflows?
29. Explain how Marketplace Actions fit into a DevSecOps pipeline using SonarQube, Trivy, and Veracode.
30. Design a production GitOps pipeline using Marketplace Actions, ECR, ArgoCD, and EKS.
31. Why is SHA pinning stronger than a version tag?
32. What are the operational trade-offs of SHA pinning?
33. How would you review the source code and dependencies of a third-party Action?
34. How would you minimize the blast radius if a Marketplace Action becomes compromised?
35. Why should Action permissions be scoped at the job level when possible?
36. How would you build a secure Marketplace Action approval and lifecycle process?