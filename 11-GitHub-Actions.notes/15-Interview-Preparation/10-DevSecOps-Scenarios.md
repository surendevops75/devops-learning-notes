# GitHub Actions - DevSecOps Scenarios

DevSecOps interview questions test whether you can integrate security
into the complete software delivery lifecycle instead of treating
security as a separate activity after deployment.

A production DevSecOps pipeline should follow:

    Developer
        |
        ↓
    Pull Request
        |
        ↓
    Source Validation
        |
        ↓
    Build
        |
        +--- SAST
        +--- SCA
        +--- Secret Detection
        |
        ↓
    Tests
        |
        ↓
    Docker Build
        |
        +--- Container Scan
        |
        ↓
    Artifact Registry
        |
        ↓
    Deployment
        |
        +--- Kubernetes Security
        +--- Runtime Validation
        |
        ↓
    Monitoring
        |
        ↓
    Continuous Security

The main principles are:

    Shift Left
        +
    Defense in Depth
        +
    Least Privilege
        +
    Secure Defaults
        +
    Continuous Scanning
        +
    Immutable Artifacts
        +
    Supply Chain Security
        +
    Continuous Monitoring

---

# 1. Design a Complete DevSecOps Pipeline

Question:

    Design a DevSecOps pipeline using GitHub Actions, SonarQube,
    Trivy, Veracode, Docker, ECR, ArgoCD, and EKS.

Answer:

I would integrate security into multiple stages.

    Developer
        |
        ↓
    Pull Request
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
    Dependency Security
        |
        ↓
    Veracode
        |
        ↓
    Docker Build
        |
        ↓
    Trivy Image Scan
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
        |
        ↓
    Runtime Validation

The goal is to identify vulnerabilities as early as possible.

---

# 2. What Is DevSecOps?

Question:

    What is DevSecOps?

Answer:

DevSecOps integrates security throughout the software development
and delivery lifecycle.

Traditional approach:

    Development
        |
        ↓
    Operations
        |
        ↓
    Security Later

DevSecOps:

    Development
        |
        +--- Security
        |
        ↓
    Build
        |
        +--- Security
        |
        ↓
    Deployment
        |
        +--- Security
        |
        ↓
    Runtime
        |
        +--- Security

Security becomes a shared responsibility.

---

# 3. Shift-Left Security

Question:

    What does shift-left security mean?

Answer:

Shift-left means identifying security issues earlier in the
development lifecycle.

    Developer
        |
        ↓
    Code
        |
        ↓
    Security Check
        |
        ↓
    Build
        |
        ↓
    Deployment

Instead of discovering a vulnerability in production, identify it
during development or CI.

Benefits:

    Lower Remediation Cost
        +
    Faster Feedback
        +
    Reduced Production Risk

---

# 4. Security Gates

Question:

    What is a security gate in CI/CD?

Answer:

A security gate is a condition that must pass before the pipeline
can continue.

Example:

    Code
        |
        ↓
    SonarQube
        |
       / \
    Pass  Fail
     |      |
     ↓      X
 Continue  Stop

The same model can be applied to:

    Trivy
        +
    Veracode
        +
    Dependency Checks

---

# 5. Design SAST Integration

Question:

    Where would you integrate SAST into GitHub Actions?

Answer:

    Checkout
        |
        ↓
    Build
        |
        ↓
    SAST
        |
        ↓
    Quality Gate
        |
        ↓
    Continue

For the user's stack, SonarQube and Veracode can provide source-code
security and quality checks.

---

# 6. SonarQube Quality Gate Failure

Question:

    SonarQube quality gate fails during CI. What should happen?

Answer:

If the organization's policy requires the gate to block:

    SonarQube
        |
        ↓
    Quality Gate Failed
        |
        X
    Pipeline Stops

The developer should fix the issue and submit another change.

A security or quality gate should not be bypassed casually.

---

# 7. SonarQube Finds a Security Issue

Question:

    SonarQube identifies a security vulnerability in newly changed
    code. What would you do?

Answer:

I would:

    Review Finding
        |
        ↓
    Determine Severity
        |
        ↓
    Identify Root Cause
        |
        ↓
    Fix Code
        |
        ↓
    Run Tests
        |
        ↓
    Rescan

If the issue is false positive, it should go through the approved
exception process rather than simply being ignored.

---

# 8. SAST vs SCA

Question:

    What is the difference between SAST and SCA?

Answer:

### SAST

Analyzes application source code for security problems.

    Source Code
        |
        ↓
    SAST
        |
        ↓
    Security Findings

### SCA

Analyzes third-party dependencies and identifies known
vulnerabilities.

    Dependencies
        |
        ↓
    SCA
        |
        ↓
    Known Vulnerabilities

Both are important because application code and dependencies create
different security risks.

---

# 9. Dependency Vulnerability

Question:

    A dependency contains a critical vulnerability. What would you
    do?

Answer:

    Dependency
        |
        ↓
    Vulnerability
        |
        ↓
    Identify Version
        |
        ↓
    Find Patched Version
        |
        ↓
    Update
        |
        ↓
    Test
        |
        ↓
    Rescan

If no patched version exists, assess the risk and follow the
approved exception process.

---

# 10. Critical Dependency Vulnerability

Question:

    A critical vulnerability is discovered in a production
    dependency. What would you do?

Answer:

First determine:

    Is It Exploitable?
        +
    Is It Used?
        +
    Is Production Exposed?
        +
    Is A Patch Available?

Then:

    Patch
        |
        ↓
    Test
        |
        ↓
    Security Scan
        |
        ↓
    Controlled Deployment

If actively exploited, the response should be accelerated.

---

# 11. Dependency Update Breaks Application

Question:

    You update a vulnerable dependency, but the new version breaks
    the application. What would you do?

Answer:

I would determine whether:

    Another Patched Version Exists
        +
    A Compatible Upgrade Exists
        +
    Temporary Mitigation Is Possible

I would not knowingly deploy an insecure version without risk
assessment and appropriate approval.

---

# 12. Secret Detection

Question:

    How would you prevent secrets from being committed to Git?

Answer:

Use multiple layers:

    Developer Awareness
        +
    Secret Scanning
        +
    Pre-Commit Checks
        +
    CI Validation
        +
    Protected Secrets

If a secret is detected:

    Stop Pipeline
        |
        ↓
    Revoke / Rotate
        |
        ↓
    Investigate
        |
        ↓
    Fix Exposure

---

# 13. Secret Found in GitHub Repository

Question:

    An AWS access key is accidentally committed to GitHub.
    What is your immediate response?

Answer:

Treat the credential as compromised.

    Detect
        |
        ↓
    Revoke
        |
        ↓
    Rotate
        |
        ↓
    Review Usage
        |
        ↓
    Remove Exposure
        |
        ↓
    Move to OIDC

Removing the key from the latest commit does not make the leaked
credential safe.

---

# 14. Secret Appears in GitHub Actions Logs

Question:

    A production secret appears in GitHub Actions logs.
    What would you do?

Answer:

Immediate response:

    Identify Secret
        |
        ↓
    Revoke / Rotate
        |
        ↓
    Identify Why It Was Printed
        |
        ↓
    Remove Unsafe Logging
        |
        ↓
    Review Logs / Access
        |
        ↓
    Validate

Secrets should never be printed intentionally.

---

# 15. GitHub Secrets vs AWS Secrets

Question:

    When would you use GitHub Secrets and when would you use AWS
    secret management?

Answer:

The choice depends on the architecture.

GitHub Secrets can be useful for:

    CI/CD-Specific Secrets

AWS secret management can be useful for:

    Runtime Application Secrets
        +
    Centralized Secret Lifecycle
        +
    AWS-Based Applications

For AWS authentication from GitHub Actions, OIDC is preferred over
long-lived AWS access keys.

---

# 16. Replace AWS Access Keys With OIDC

Question:

    Your GitHub Actions workflow stores AWS access keys as secrets.
    How would you improve it?

Answer:

Replace long-lived credentials with OIDC.

    GitHub Actions
        |
        ↓
    OIDC Token
        |
        ↓
    AWS IAM Trust
        |
        ↓
    Temporary Credentials
        |
        ↓
    AWS

Benefits:

    No Long-Lived Keys
        +
    Short-Lived Credentials
        +
    Better Trust Controls
        +
    Better Auditability

---

# 17. OIDC Trust Policy Too Broad

Question:

    Your GitHub OIDC role trusts all repositories in the
    organization. What is the problem?

Answer:

The trust boundary is too broad.

I would restrict the trust relationship using appropriate conditions
such as:

    Organization
        +
    Repository
        +
    Branch
        +
    Environment

The production role should only be assumable by the intended
production workflow.

---

# 18. Least Privilege in GitHub Actions

Question:

    How would you implement least privilege in GitHub Actions?

Answer:

At the GitHub level:

    Minimal GITHUB_TOKEN Permissions

At AWS:

    Minimal IAM Permissions

At Kubernetes:

    Minimal RBAC Permissions

The workflow should receive only what it needs.

---

# 19. GITHUB_TOKEN Permissions

Question:

    Why should you explicitly configure GITHUB_TOKEN permissions?

Answer:

Because excessive permissions increase blast radius.

Instead of:

    Broad Permissions

prefer:

    Required Permission Only

For example:

    contents: read

when the workflow only needs repository checkout.

---

# 20. Production Workflow Permissions

Question:

    A production deployment workflow has write access to every
    repository. Is that a concern?

Answer:

Yes.

The workflow should have only the permissions required for its
specific responsibilities.

Broad permissions increase the impact of:

    Compromised Workflow
        +
    Malicious Dependency
        +
    Malicious Pull Request

---

# 21. Third-Party GitHub Action Security

Question:

    What security risks exist when using third-party GitHub Actions?

Answer:

A third-party action executes code inside the workflow environment.

Risks include:

    Malicious Update
        +
    Compromised Repository
        +
    Excessive Permissions
        +
    Secret Exposure
        +
    Supply Chain Attack

---

# 22. Pin GitHub Actions

Question:

    Why should production workflows pin third-party actions?

Answer:

A mutable action reference can change unexpectedly.

Using a trusted immutable reference reduces the risk of an
unexpected action update.

Conceptually:

    Workflow
        |
        ↓
    Trusted Action Version
        |
        ↓
    Controlled Execution

For high-security environments, pinning to a specific commit SHA
provides stronger immutability.

---

# 23. Compromised Third-Party Action

Question:

    A third-party GitHub Action used by your organization is
    compromised. What would you do?

Answer:

    Identify Affected Repositories
        |
        ↓
    Stop Using Action
        |
        ↓
    Replace With Trusted Version / Action
        |
        ↓
    Review Workflow Activity
        |
        ↓
    Rotate Credentials If Required
        |
        ↓
    Audit

---

# 24. Software Supply Chain Security

Question:

    How would you secure the software supply chain?

Answer:

    Source Control
        |
        ↓
    Dependency Validation
        |
        ↓
    SAST
        |
        ↓
    SCA
        |
        ↓
    Build
        |
        ↓
    Container Scan
        |
        ↓
    Artifact Registry
        |
        ↓
    Artifact Verification
        |
        ↓
    Deployment
        |
        ↓
    Runtime Monitoring

Security should exist across the complete chain.

---

# 25. Container Security

Question:

    How would you secure Docker images in CI/CD?

Answer:

I would use:

    Trusted Base Image
        +
    Minimal Packages
        +
    Non-Root User
        +
    Vulnerability Scanning
        +
    Secret Detection
        +
    Immutable Tagging
        +
    Registry Controls

Trivy can be used for container vulnerability scanning.

---

# 26. Trivy Finds Critical Vulnerability

Question:

    Trivy finds a critical vulnerability in a Docker image.
    What should the pipeline do?

Answer:

If the security policy blocks critical vulnerabilities:

    Docker Build
        |
        ↓
    Trivy
        |
        ↓
    Critical Finding
        |
        X
    Pipeline Fails

Then:

    Update Base Image / Package
        |
        ↓
    Rebuild
        |
        ↓
    Rescan

---

# 27. Trivy Finds Vulnerability in Base Image

Question:

    Your application code is secure, but the Docker base image
    contains critical vulnerabilities. What would you do?

Answer:

I would:

    Identify Vulnerable Package
        |
        ↓
    Check Updated Base Image
        |
        ↓
    Update Dockerfile
        |
        ↓
    Rebuild
        |
        ↓
    Rescan

The application is only as secure as the complete container image.

---

# 28. Vulnerability Has No Fix

Question:

    Trivy reports a vulnerability but no fixed version exists.
    What would you do?

Answer:

I would assess:

    Severity
        +
    Exploitability
        +
    Exposure
        +
    Actual Usage

Then:

    Mitigate
        OR
    Replace Component
        OR
    Apply Approved Exception

The exception should be documented, approved, and reviewed.

---

# 29. False Positive Security Finding

Question:

    A security scanner reports a vulnerability that is a false
    positive. What would you do?

Answer:

I would:

    Validate Finding
        |
        ↓
    Confirm False Positive
        |
        ↓
    Document Evidence
        |
        ↓
    Use Approved Suppression / Exception
        |
        ↓
    Continue

I would not simply disable the scanner.

---

# 30. Security Exception

Question:

    When should a security exception be allowed?

Answer:

Only when:

    Risk Is Understood
        +
    Business Need Exists
        +
    No Immediate Fix Is Available
        +
    Compensating Controls Exist
        +
    Authorized Team Approves

Exceptions should be:

    Documented
        +
    Time-Bounded
        +
    Reviewed

---

# 31. Dockerfile Security

Question:

    What Dockerfile practices would you follow?

Answer:

    Use Trusted Base Image
        +
    Keep Image Small
        +
    Remove Unnecessary Packages
        +
    Avoid Secrets
        +
    Use Non-Root User
        +
    Pin Important Dependencies
        +
    Scan Image

---

# 32. Secrets in Dockerfile

Question:

    A developer uses an API key in a Dockerfile during build.
    Why is this dangerous?

Answer:

Secrets can become exposed through:

    Image Layers
        +
    Build History
        +
    Registry
        +
    Logs

Secrets should not be baked into container images.

---

# 33. Secrets in Environment Variables

Question:

    Is passing secrets through environment variables always safe?

Answer:

Not automatically.

Environment variables can potentially appear through:

    Logs
        +
    Debugging
        +
    Process Inspection
        +
    Application Errors

Secret handling should be designed according to the application's
runtime requirements.

---

# 34. Runtime Secret Management

Question:

    How should Kubernetes applications retrieve production secrets?

Answer:

Use an appropriate secure secret-management architecture.

The key principles are:

    No Hardcoded Secrets
        +
    Encryption
        +
    Access Control
        +
    Rotation
        +
    Auditability

---

# 35. Kubernetes RBAC Security

Question:

    How would you secure Kubernetes permissions?

Answer:

Use least privilege.

    User / Service Account
        |
        ↓
    Role
        |
        ↓
    Required Resources
        +
    Required Verbs

Avoid giving application service accounts unnecessary cluster-wide
permissions.

---

# 36. GitHub Actions Kubernetes Permissions

Question:

    Your GitHub Actions workflow has cluster-admin access.
    What would you do?

Answer:

Reduce privileges.

Instead of:

    cluster-admin

use the minimum permissions required for:

    Deployment
        +
    Service
        +
    ConfigMap
        +
    Other Required Resources

The exact permissions should match the deployment model.

---

# 37. Service Account Security

Question:

    How would you secure Kubernetes service accounts?

Answer:

    Minimal Permissions
        +
    Namespace Scope
        +
    Avoid Unnecessary Token Access
        +
    Separate Service Accounts
        +
    Audit Usage

---

# 38. Kubernetes Network Security

Question:

    How would you limit communication between microservices?

Answer:

Use network policies.

    Service A
        |
        X
    Unauthorized Service

    Service A
        |
        ↓
    Authorized Service

The goal is to reduce unnecessary east-west communication.

---

# 39. EKS Security Design

Question:

    What security controls would you consider for an EKS-based
    production platform?

Answer:

    IAM
        +
    OIDC
        +
    Kubernetes RBAC
        +
    Network Policies
        +
    Security Groups
        +
    Private Networking Where Appropriate
        +
    Image Scanning
        +
    Secrets Management
        +
    Logging
        +
    Monitoring

---

# 40. Terraform Security

Question:

    How would you secure Terraform in CI/CD?

Answer:

Use:

    Pull Request Review
        +
    terraform validate
        +
    Security Scanning
        +
    terraform plan
        +
    Approval
        +
    Least Privilege AWS Role
        +
    Protected State

---

# 41. Terraform Plan Security Review

Question:

    Terraform plan shows a security group opening port 0.0.0.0/0.
    What would you do?

Answer:

I would stop the deployment.

    Terraform Plan
        |
        ↓
    Dangerous Rule
        |
        X
    Production Blocked

Then determine:

    Intended?
        +
    Required?
        +
    Secure Alternative?

---

# 42. Infrastructure Security Drift

Question:

    Someone manually changes a production security group.
    How would you detect and handle it?

Answer:

    Actual AWS State
        |
        ↓
    Terraform
        |
        ↓
    Detect Drift
        |
        ↓
    Investigate
        |
        ↓
    Reconcile

---

# 43. Branch Protection as Security Control

Question:

    Why is branch protection part of DevSecOps?

Answer:

Because CI/CD configuration itself is security-sensitive.

Branch protection can require:

    Pull Request
        +
    Review
        +
    Status Checks

This reduces unauthorized workflow and application changes.

---

# 44. CODEOWNERS for Security

Question:

    Why would you use CODEOWNERS for GitHub Actions workflows?

Answer:

Workflow files can control:

    Secrets
        +
    AWS Access
        +
    Production Deployment

Therefore sensitive workflow changes should require review from
appropriate owners.

---

# 45. Pull Request Security

Question:

    What security concerns exist with pull requests?

Answer:

A PR may contain:

    Malicious Code
        +
    Malicious Workflow Changes
        +
    Dependency Changes
        +
    Secret Exposure
        +
    Unsafe Scripts

Therefore untrusted PR workflows should not automatically receive
production credentials.

---

# 46. Pull Request and Production Secrets

Question:

    Should a pull request from an untrusted fork receive production
    secrets?

Answer:

No.

Production credentials should not be exposed to untrusted code.

A safer architecture is:

    Untrusted PR
        |
        ↓
    Limited CI
        |
        ↓
    No Production Credentials

Trusted code should pass required reviews before production access.

---

# 47. Workflow Injection

Question:

    What is command or workflow injection in GitHub Actions?

Answer:

It occurs when untrusted user-controlled input is inserted into shell
commands or workflow logic without proper handling.

Risk:

    Pull Request Input
        |
        ↓
    Workflow
        |
        ↓
    Shell
        |
        ↓
    Unexpected Command

Inputs should be treated as untrusted.

---

# 48. Secure Shell Usage

Question:

    How would you reduce shell injection risks in GitHub Actions?

Answer:

I would:

    Avoid Direct Interpolation of Untrusted Input
        +
    Validate Inputs
        +
    Use Environment Variables Carefully
        +
    Quote Values Appropriately
        +
    Limit Permissions

---

# 49. Malicious Commit Message

Question:

    A workflow uses the commit message inside a shell command.
    Why could this be dangerous?

Answer:

Commit messages are user-controlled input.

If directly interpolated:

    Commit Message
        |
        ↓
    Shell Command
        |
        ↓
    Injection Risk

Inputs should be handled safely.

---

# 50. Pull Request Title Injection

Question:

    A workflow uses the PR title in a shell command.
    What should you consider?

Answer:

Treat the PR title as untrusted input.

Safer approach:

    PR Title
        |
        ↓
    Environment Variable
        |
        ↓
    Controlled Processing

Do not construct shell commands by blindly concatenating user input.

---

# 51. Untrusted Artifact

Question:

    Why is downloading an arbitrary artifact during a privileged
    production job risky?

Answer:

The artifact could contain malicious code.

A privileged workflow should:

    Verify Artifact
        +
    Control Source
        +
    Validate Integrity
        +
    Limit Permissions

---

# 52. Artifact Integrity

Question:

    How would you verify artifact integrity?

Answer:

Use:

    Immutable Identifier
        +
    Digest
        +
    Trusted Registry
        +
    Signature Where Applicable

Conceptually:

    Artifact
        |
        ↓
    Verify
        |
        ↓
    Trusted?
       / \
     Yes  No
      |    |
      ↓    X
   Deploy Stop

---

# 53. Container Image Signing

Question:

    Why would you sign container images?

Answer:

Image signing can provide:

    Authenticity
        +
    Integrity
        +
    Provenance

Architecture:

    Build
        |
        ↓
    Image
        |
        ↓
    Sign
        |
        ↓
    Registry
        |
        ↓
    Verify
        |
        ↓
    Deploy

---

# 54. Image Digest vs Image Tag

Question:

    Why is an image digest stronger than a mutable tag?

Answer:

A tag can point to different images over time.

A digest identifies a specific image content.

    Tag
        |
        ↓
    May Change

    Digest
        |
        ↓
    Specific Content

Production deployments should prefer immutable identifiers.

---

# 55. Artifact Provenance

Question:

    What is artifact provenance?

Answer:

It provides information about how an artifact was produced.

For example:

    Source Commit
        |
        ↓
    Workflow
        |
        ↓
    Build
        |
        ↓
    Image
        |
        ↓
    Registry
        |
        ↓
    Deployment

This helps answer:

    "Where did this production artifact come from?"

---

# 56. Software Bill of Materials

Question:

    What is an SBOM?

Answer:

An SBOM describes the components and dependencies contained in
software.

Conceptually:

    Application
        |
        +--- Library A
        +--- Library B
        +--- Package C
        +--- Base Image
        |
        ↓
       SBOM

It helps with:

    Vulnerability Management
        +
    Supply Chain Visibility
        +
    Compliance

---

# 57. SBOM in CI/CD

Question:

    Where would you generate an SBOM?

Answer:

A typical location is after building the artifact.

    Build
        |
        ↓
    Image
        |
        ↓
    SBOM
        |
        ↓
    Store / Track
        |
        ↓
    Deploy

The exact implementation depends on the organization's tooling.

---

# 58. Base Image Management

Question:

    How would you manage Docker base images securely?

Answer:

Use:

    Trusted Sources
        +
    Approved Versions
        +
    Regular Updates
        +
    Vulnerability Scanning
        +
    Rebuild Automation

---

# 59. Outdated Base Image

Question:

    Your production image uses a base image that is six months old.
    What would you do?

Answer:

I would check:

    Security Advisories
        +
    Vulnerabilities
        +
    Available Updates
        +
    Compatibility

Then:

    Update
        |
        ↓
    Test
        |
        ↓
    Scan
        |
        ↓
    Release

---

# 60. Automated Base Image Updates

Question:

    How would you keep container base images updated?

Answer:

Possible flow:

    New Base Image
        |
        ↓
    Automated Update PR
        |
        ↓
    CI
        |
        +--- Tests
        +--- Security
        |
        ↓
    Review
        |
        ↓
    Merge
        |
        ↓
    Deployment

---

# 61. Dependency Pinning

Question:

    Why should dependencies be pinned?

Answer:

Uncontrolled dependency versions can introduce unexpected changes.

Pinning improves:

    Reproducibility
        +
    Predictability
        +
    Supply Chain Control

---

# 62. Dependency Lock Files

Question:

    Why are lock files important?

Answer:

They capture resolved dependency versions.

Without a lock file:

    Build
        |
        ↓
    Dependency Resolution
        |
        ↓
    Potentially Different Versions

With a lock file:

    Build
        |
        ↓
    Locked Dependencies
        |
        ↓
    Reproducible Build

---

# 63. Malicious Dependency

Question:

    A dependency used by your application is compromised upstream.
    How would you respond?

Answer:

    Identify Affected Version
        |
        ↓
    Determine Exposure
        |
        ↓
    Stop New Builds
        |
        ↓
    Replace / Upgrade Dependency
        |
        ↓
    Rebuild
        |
        ↓
    Rescan
        |
        ↓
    Redeploy

If production was affected, investigate for compromise.

---

# 64. Dependency Confusion

Question:

    What is dependency confusion?

Answer:

It occurs when a package manager is tricked into downloading a
malicious package instead of the intended internal package.

Controls include:

    Trusted Registries
        +
    Package Namespace Controls
        +
    Dependency Pinning
        +
    Internal Repository Configuration
        +
    Dependency Verification

---

# 65. JFrog Artifactory in DevSecOps

Question:

    How could JFrog Artifactory fit into a DevSecOps pipeline?

Answer:

    Build
        |
        ↓
    Artifact
        |
        ↓
    Artifactory
        |
        ↓
    Validation
        |
        ↓
    Deployment

It can provide centralized artifact management depending on the
organization's architecture.

---

# 66. ECR Security

Question:

    How would you secure Docker images in Amazon ECR?

Answer:

Use:

    Private Repositories
        +
    IAM Access Control
        +
    Image Scanning
        +
    Lifecycle Policies
        +
    Immutable Tags Where Appropriate
        +
    Controlled Access

---

# 67. ECR Access Control

Question:

    Developers should pull images but should not delete production
    images. How would you implement this?

Answer:

Use separate IAM permissions.

    Developer
        |
        +--- Pull
        |
        X--- Delete

    Release Role
        |
        +--- Push
        +
        +--- Required Management

Least privilege should be applied to registry access.

---

# 68. Production Artifact Deletion

Question:

    Someone deletes the production Docker image from ECR.
    What would you do?

Answer:

First determine whether the running workload is affected.

Then:

    Restore Artifact If Possible
        +
    Preserve Known-Good Versions
        +
    Review IAM Access
        +
    Improve Lifecycle Policy

Production rollback depends on artifact availability.

---

# 69. Artifact Retention

Question:

    How long should production artifacts be retained?

Answer:

Retention should support:

    Rollback
        +
    Incident Investigation
        +
    Compliance
        +
    Recovery

The exact retention period depends on organizational requirements.

---

# 70. Security Scanning vs Deployment Speed

Question:

    Your security scans add 15 minutes to CI. Developers complain.
    What would you do?

Answer:

I would optimize without removing important controls.

Options:

    Parallel Scans
        +
    Dependency Caching
        +
    Incremental Scanning
        +
    Scan Optimization
        +
    Appropriate Execution Frequency

Security should remain effective while unnecessary pipeline delay
is reduced.

---

# 71. Security Scan in Parallel

Question:

    Which security checks can run in parallel?

Answer:

Independent checks can often run concurrently.

    Build
        |
        +--- SonarQube
        +--- Dependency Scan
        +--- Secret Scan

Then:

    Security Results
        |
        ↓
    Gate

This reduces total pipeline duration.

---

# 72. Security Gate vs Warning

Question:

    Should every security finding fail the pipeline?

Answer:

Not necessarily.

The policy should consider:

    Severity
        +
    Exploitability
        +
    Environment
        +
    Business Risk
        +
    Exposure

For example:

    Critical
        |
        ↓
    Block

    Low
        |
        ↓
    Track / Remediate

The exact policy should be defined by the organization.

---

# 73. Security Severity Policy

Question:

    How would you design vulnerability thresholds?

Answer:

Example:

    Critical → Block
    High     → Block or Review
    Medium   → Track / Review
    Low      → Track

The exact thresholds should be based on organizational risk policy.

---

# 74. DevSecOps in Pull Requests

Question:

    What security checks should happen before merging code?

Answer:

    SAST
        +
    Dependency Security
        +
    Secret Detection
        +
    Unit Tests
        +
    Code Quality

Then:

    Required Checks
        |
        ↓
    Review
        |
        ↓
    Merge

---

# 75. DevSecOps in Build Stage

Question:

    What security checks belong around the build stage?

Answer:

    Dependency Scan
        +
    SAST
        +
    Secret Detection
        +
    Secure Build
        +
    Dockerfile Validation
        +
    Container Scan

---

# 76. DevSecOps in Deployment Stage

Question:

    What security controls should exist before production deployment?

Answer:

    Approved Artifact
        +
    Immutable Digest
        +
    Environment Protection
        +
    Approval
        +
    Least Privilege
        +
    Deployment Validation

---

# 77. DevSecOps at Runtime

Question:

    Does DevSecOps end after deployment?

Answer:

No.

Runtime security includes:

    Monitoring
        +
    Logs
        +
    Access Control
        +
    Vulnerability Management
        +
    Incident Response
        +
    Continuous Improvement

---

# 78. Security Monitoring

Question:

    What should you monitor from a DevSecOps perspective?

Answer:

    Authentication
        +
    Authorization
        +
    Deployment Activity
        +
    Infrastructure Changes
        +
    Application Errors
        +
    Security Findings

---

# 79. Production Security Incident

Question:

    A production application shows suspicious activity after a
    deployment. What would you do?

Answer:

First:

    Assess Impact
        |
        ↓
    Contain
        |
        ↓
    Preserve Evidence
        |
        ↓
    Revoke Compromised Credentials
        |
        ↓
    Investigate
        |
        ↓
    Recover
        |
        ↓
    Prevent Recurrence

---

# 80. Compromised GitHub Runner

Question:

    A self-hosted runner used for production deployments may be
    compromised. What would you do?

Answer:

    Isolate Runner
        |
        ↓
    Revoke / Rotate Credentials
        |
        ↓
    Review Workflows
        |
        ↓
    Investigate
        |
        ↓
    Destroy Runner
        |
        ↓
    Provision Clean Runner
        |
        ↓
    Validate

---

# 81. Persistent Self-Hosted Runner Risk

Question:

    Why are persistent self-hosted runners risky?

Answer:

A previous job may leave:

    Files
        +
    Credentials
        +
    Processes
        +
    Modified Environment
        +
    Cached Data

A later job may inherit the compromised state.

Ephemeral runners reduce this risk.

---

# 82. Secure Self-Hosted Runner Architecture

Question:

    Design a secure self-hosted runner architecture.

Answer:

    GitHub
        |
        ↓
    Runner Pool
        |
        +--- Ephemeral Runner
        +--- Isolated Network
        +--- Minimal IAM
        |
        ↓
    Job
        |
        ↓
    Destroy Runner

Avoid unnecessary access from the runner to production systems.

---

# 83. Production Runner Network Access

Question:

    Should a CI runner have unrestricted access to the production
    network?

Answer:

No.

Network access should be limited to what the deployment actually
requires.

    Runner
        |
        ↓
    Required Endpoint
        |
        X
    Unnecessary Network

---

# 84. DevSecOps and Infrastructure as Code

Question:

    Why is Infrastructure as Code important for DevSecOps?

Answer:

Infrastructure can be:

    Version Controlled
        +
    Reviewed
        +
    Tested
        +
    Scanned
        +
    Audited
        +
    Reproduced

Terraform enables security checks before infrastructure reaches
production.

---

# 85. Terraform Security Scanning

Question:

    What security issues can be identified before Terraform apply?

Answer:

Potential issues include:

    Public Resources
        +
    Open Security Groups
        +
    Excessive IAM Permissions
        +
    Unencrypted Resources
        +
    Insecure Configuration

The exact checks depend on the scanning tools and policies used.

---

# 86. IAM Policy Security

Question:

    A Terraform change adds AdministratorAccess to an application
    role. What would you do?

Answer:

Stop the change.

    Terraform Plan
        |
        ↓
    Excessive Permission
        |
        X
    Deployment Blocked

Then apply least privilege.

---

# 87. Kubernetes Security Scanning

Question:

    What Kubernetes manifest security checks would you perform?

Answer:

Check for:

    Privileged Containers
        +
    Root User
        +
    Excessive Capabilities
        +
    Missing Resource Limits
        +
    Host Network
        +
    Host Path
        +
    Excessive RBAC

---

# 88. Container Running as Root

Question:

    A production container runs as root. What would you do?

Answer:

If the application does not require root:

    Create Non-Root User
        |
        ↓
    Update Dockerfile
        |
        ↓
    Test
        |
        ↓
    Scan
        |
        ↓
    Deploy

Running as non-root reduces the impact of container compromise.

---

# 89. Privileged Container

Question:

    A Kubernetes deployment uses privileged containers.
    What would you investigate?

Answer:

Determine:

    Why Is Privileged Mode Required?
        +
    Can It Be Removed?
        +
    Is There A Safer Alternative?

If unnecessary:

    Remove Privilege
        |
        ↓
    Validate

---

# 90. Kubernetes Secrets in Git

Question:

    A developer commits a Kubernetes Secret manifest containing
    plaintext credentials. What would you do?

Answer:

Treat the credential as exposed.

    Remove Exposure
        |
        ↓
    Rotate Secret
        |
        ↓
    Move To Secure Secret Management
        |
        ↓
    Audit Repository
        |
        ↓
    Prevent Recurrence

---

# 91. Base64 Is Not Encryption

Question:

    A developer says Kubernetes Secrets are safe because the value
    is base64 encoded. How would you respond?

Answer:

Base64 is encoding, not encryption.

    Secret
        |
        ↓
    Base64
        |
        ↓
    Encoded Value

Anyone with access to the encoded value can decode it.

Secret storage and access controls are still required.

---

# 92. Production Secret Rotation

Question:

    How would you design secret rotation without causing downtime?

Answer:

A safe rotation pattern can be:

    New Secret
        |
        ↓
    Application Supports New Credential
        |
        ↓
    Rotate
        |
        ↓
    Validate
        |
        ↓
    Remove Old Credential

Applications should support overlapping credential validity where
appropriate.

---

# 93. Security and Zero Downtime

Question:

    How would you deploy a security patch without downtime?

Answer:

Use:

    Immutable Artifact
        +
    Rolling / Canary Deployment
        +
    Health Checks
        +
    Multiple Replicas
        +
    Controlled Promotion

---

# 94. Emergency Security Patch

Question:

    A critical vulnerability is actively exploited.
    How would you accelerate deployment?

Answer:

Use an emergency release path:

    Patch
        |
        ↓
    Automated Tests
        |
        ↓
    Security Validation
        |
        ↓
    Authorized Approval
        |
        ↓
    Production
        |
        ↓
    Monitoring

Emergency does not mean bypassing all security controls.

---

# 95. Security Incident and Rollback

Question:

    A new release introduces a security vulnerability.
    Would you rollback?

Answer:

It depends.

If rollback removes the vulnerability and is safe:

    Rollback

If rollback returns to an older version with the same or another
security issue:

    Fix Forward

The decision must consider:

    Security Risk
        +
    Availability
        +
    Data Compatibility
        +
    Recovery Time

---

# 96. Security vs Availability

Question:

    How do you balance security and availability during an incident?

Answer:

Assess both risks.

    Security Risk
        +
    Availability Risk
        |
        ↓
    Risk-Based Decision

The immediate objective is to:

    Contain Threat
        +
    Protect Users
        +
    Restore Service
        +
    Prevent Further Damage

---

# 97. DevSecOps and GitOps

Question:

    How does GitOps improve security?

Answer:

GitOps provides:

    Version Control
        +
    Review
        +
    Audit Trail
        +
    Desired State
        +
    Drift Detection
        +
    Controlled Changes

Production changes become more traceable.

---

# 98. GitOps Drift as Security Issue

Question:

    Someone manually modifies a production Kubernetes deployment.
    Why can this be a security issue?

Answer:

Manual changes can bypass:

    Review
        +
    Security Checks
        +
    Approval
        +
    Audit

GitOps reconciliation can help restore the approved desired state.

---

# 99. Unauthorized GitOps Change

Question:

    An unauthorized production manifest is merged into GitOps.
    What would you do?

Answer:

    Identify Change
        |
        ↓
    Revert
        |
        ↓
    Validate
        |
        ↓
    ArgoCD Reconcile
        |
        ↓
    Audit
        |
        ↓
    Strengthen Review Controls

---

# 100. Secure GitOps Repository

Question:

    How would you secure a GitOps repository?

Answer:

Use:

    Branch Protection
        +
    Pull Request Review
        +
    CODEOWNERS
        +
    Environment Separation
        +
    Secret Avoidance
        +
    Auditability
        +
    Limited Access

---

# 101. DevSecOps and Separation of Duties

Question:

    How would you implement separation of duties in DevSecOps?

Answer:

    Developer
        |
        ↓
    Code
        |
        ↓
    Reviewer
        |
        ↓
    CI
        |
        ↓
    Security Gates
        |
        ↓
    Release Approver
        |
        ↓
    Production

This prevents one person from controlling the entire release path.

---

# 102. Security Approval

Question:

    Should security approve every deployment?

Answer:

Not necessarily.

Security approval can be required for:

    High-Risk Changes
        +
    Security Exceptions
        +
    Sensitive Infrastructure
        +
    Critical Security Fixes

Routine low-risk deployments can remain automated under established
policies.

---

# 103. Security Policy as Code

Question:

    What is policy as code?

Answer:

Security and compliance rules are represented in machine-readable
policies and evaluated automatically.

    Infrastructure
        |
        ↓
    Policy
        |
       / \
    Pass  Fail
     |      |
     ↓      X
 Continue  Stop

---

# 104. Prevent Public S3 Bucket

Question:

    How could DevSecOps prevent a Terraform change from creating a
    public S3 bucket?

Answer:

    Pull Request
        |
        ↓
    Terraform Plan
        |
        ↓
    Security Policy
        |
        ↓
    Public Bucket
        |
        X
    Pipeline Fails

This prevents insecure infrastructure from reaching production.

---

# 105. Prevent Public Security Group

Question:

    How could you prevent a security group from exposing SSH to
    the internet?

Answer:

A policy check can detect:

    Port 22
        +
    0.0.0.0/0

and block the Terraform change.

---

# 106. DevSecOps Metrics

Question:

    What DevSecOps metrics would you track?

Answer:

Useful metrics include:

    Vulnerabilities Found
        +
    Vulnerabilities Fixed
        +
    Mean Time To Remediate
        +
    Security Gate Failure Rate
        +
    Dependency Risk
        +
    Container Vulnerability Count
        +
    Secret Detection Events
        +
    Security Exceptions

---

# 107. Mean Time to Remediate

Question:

    What is MTTR in a DevSecOps context?

Answer:

Mean Time to Remediate measures how long it takes to address a
security issue.

    Vulnerability Detected
        |
        ↓
    Fix
        |
        ↓
    Validation
        |
        ↓
    Deployment

Lower remediation time generally reduces exposure.

---

# 108. Security Debt

Question:

    What is security debt?

Answer:

Security debt represents unresolved security weaknesses that
accumulate over time.

Examples:

    Outdated Dependencies
        +
    Vulnerable Images
        +
    Excessive Permissions
        +
    Unresolved Findings
        +
    Manual Security Exceptions

---

# 109. Security Debt Management

Question:

    How would you reduce security debt?

Answer:

    Identify
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

Prioritize based on:

    Severity
        +
    Exploitability
        +
    Exposure
        +
    Business Impact

---

# 110. DevSecOps and Technical Debt

Question:

    Why should security technical debt be treated like engineering
    technical debt?

Answer:

Because deferred security problems increase future:

    Risk
        +
    Cost
        +
    Complexity
        +
    Incident Probability

Security should be part of normal engineering work.

---

# 111. Security Scan False Negative

Question:

    Your security scanner reports no issues, but a vulnerability
    is later discovered in production. What would you do?

Answer:

Investigate:

    Scanner Coverage
        +
    Configuration
        +
    Detection Capability
        +
    Dependency Version
        +
    Runtime Exposure

Then improve:

    Detection
        +
    Testing
        +
    Monitoring

A scanner passing does not mean the system is guaranteed secure.

---

# 112. Security Scanner Failure

Question:

    Trivy itself fails during CI. Should the pipeline continue?

Answer:

It depends on the security policy.

For a mandatory security gate:

    Scanner Failure
        |
        X
    Pipeline Stops

This prevents the pipeline from treating "scan unavailable" as
"secure."

---

# 113. Fail Open vs Fail Closed

Question:

    What does fail-open vs fail-closed mean in a security pipeline?

Answer:

### Fail Open

Security control fails but deployment continues.

    Security Check Failed
        |
        ↓
    Continue

### Fail Closed

Security control fails and deployment stops.

    Security Check Failed
        |
        X
    Deployment Blocked

For critical production security controls, fail-closed behavior is
often preferred when practical.

---

# 114. Security Gate Availability

Question:

    What happens if your security scanning service is unavailable
    during a production release?

Answer:

The organization should have a defined policy.

Possible approach:

    Security Service Down
        |
        ↓
    Production Deployment Blocked
        |
        OR
        |
    Approved Emergency Exception

The decision should not be improvised during every incident.

---

# 115. Security Exception During Outage

Question:

    Production is down and the security scanner is unavailable.
    The team wants to bypass it. What would you do?

Answer:

Use the organization's emergency process.

    Incident
        |
        ↓
    Risk Assessment
        |
        ↓
    Authorized Decision
        |
        ↓
    Emergency Deployment
        |
        ↓
    Post-Incident Security Scan

The bypass should be documented and reviewed afterward.

---

# 116. DevSecOps Pipeline Performance

Question:

    How would you make a DevSecOps pipeline faster without reducing
    security?

Answer:

    Parallel Security Checks
        +
    Dependency Cache
        +
    Incremental Scanning
        +
    Reusable Workflows
        +
    Efficient Docker Builds
        +
    Appropriate Scan Frequency

---

# 117. Security Checks in Parallel

Question:

    How would you parallelize security checks?

Answer:

    Build
        |
        +--- SonarQube
        +--- Dependency Scan
        +--- Secret Scan
        |
        ↓
    Security Gate
        |
        ↓
    Continue

Independent checks should not unnecessarily wait for each other.

---

# 118. DevSecOps and CI/CD Cost

Question:

    Security scans increase GitHub Actions cost. How would you
    optimize?

Answer:

    Cache Dependencies
        +
    Parallelize Jobs
        +
    Scan Only Changed Components Where Appropriate
        +
    Reuse Results Where Safe
        +
    Optimize Runner Usage

Cost optimization should not weaken critical security controls.

---

# 119. DevSecOps and Developer Experience

Question:

    Developers complain that security checks slow them down.
    How would you improve the experience?

Answer:

Security should provide fast and actionable feedback.

Improve:

    Early Detection
        +
    Clear Findings
        +
    Developer-Friendly Reports
        +
    Fast CI
        +
    Automated Fix Suggestions Where Appropriate
        +
    Clear Policies

---

# 120. Security Findings With No Context

Question:

    Developers receive hundreds of scanner findings and do not know
    what to fix first. What would you do?

Answer:

Prioritize findings using:

    Severity
        +
    Exploitability
        +
    Exposure
        +
    Business Impact

Then provide:

    Clear Ownership
        +
    Remediation Guidance
        +
    Deadlines

---

# 121. Security Ownership

Question:

    Who is responsible for fixing security vulnerabilities?

Answer:

DevSecOps follows shared responsibility.

    Developers
        +
    DevOps
        +
    Security
        +
    Platform
        +
    Operations

The exact ownership depends on the issue.

---

# 122. Security Vulnerability Ownership

Question:

    A vulnerable dependency belongs to one application team.
    Who should fix it?

Answer:

The application team generally owns the application dependency,
while the security/platform teams can provide:

    Detection
        +
    Guidance
        +
    Policy
        +
    Platform Support

---

# 123. DevSecOps Culture

Question:

    What is the cultural difference between DevOps and DevSecOps?

Answer:

DevOps emphasizes:

    Collaboration
        +
    Automation
        +
    Fast Delivery
        +
    Shared Responsibility

DevSecOps extends that model by making security a continuous shared
responsibility.

---

# 124. Security as a Shared Responsibility

Question:

    Why should developers care about security?

Answer:

Because security issues often originate in:

    Application Code
        +
    Dependencies
        +
    Configuration
        +
    Infrastructure
        +
    Deployment

Security cannot be solved only by a separate security team.

---

# 125. DevSecOps Training

Question:

    How would you improve developer security awareness?

Answer:

Use:

    Secure Coding Guidelines
        +
    CI Feedback
        +
    Security Training
        +
    Real Examples
        +
    Automated Checks
        +
    Documentation

The goal is to make secure development part of normal engineering.

---

# 126. DevSecOps Incident Response

Question:

    A vulnerability is discovered in a production application.
    Walk through your response.

Answer:

    Detect
        |
        ↓
    Assess
        |
        ↓
    Contain
        |
        ↓
    Patch / Mitigate
        |
        ↓
    Test
        |
        ↓
    Deploy
        |
        ↓
    Validate
        |
        ↓
    Monitor
        |
        ↓
    Root Cause
        |
        ↓
    Prevent Recurrence

---

# 127. Production Vulnerability With No Exploit

Question:

    A critical vulnerability exists in production but there is
    currently no known exploit. Would you immediately shut down
    the application?

Answer:

Not automatically.

I would assess:

    Exposure
        +
    Exploitability
        +
    Business Impact
        +
    Available Mitigation
        +
    Patch Availability

Then select the safest response.

---

# 128. Actively Exploited Vulnerability

Question:

    A vulnerability is actively being exploited against systems
    similar to yours. What changes?

Answer:

The risk becomes much higher.

I would prioritize:

    Immediate Mitigation
        +
    Patch
        +
    Controlled Deployment
        +
    Monitoring
        +
    Credential Review

---

# 129. DevSecOps and Production Rollback

Question:

    A security scan discovers that the newly deployed version has
    a critical vulnerability. What should happen?

Answer:

If the current version is safer:

    Rollback
        |
        ↓
    Validate
        |
        ↓
    Fix
        |
        ↓
    Rescan
        |
        ↓
    Redeploy

If rollback introduces greater risk:

    Fix Forward

---

# 130. Security Regression

Question:

    A vulnerability was fixed, but a later release reintroduces it.
    How would you prevent this?

Answer:

Add automated regression coverage.

    Security Fix
        |
        ↓
    Test
        |
        ↓
    Security Regression Test
        |
        ↓
    CI Gate

Security fixes should become part of permanent validation.

---

# 131. DevSecOps and Regression Testing

Question:

    Why should security tests be part of regression testing?

Answer:

Because future changes can accidentally reintroduce:

    Vulnerability
        +
    Insecure Configuration
        +
    Broken Authorization
        +
    Secret Exposure

---

# 132. Authorization Testing

Question:

    How would you validate authorization security in CI/CD?

Answer:

Test that:

    Authorized User
        |
        ↓
    Allowed Action

while:

    Unauthorized User
        |
        ↓
    Blocked Action

Security testing should validate access control behavior.

---

# 133. Authentication Testing

Question:

    What authentication checks could be included in CI/CD?

Answer:

Validate:

    Valid Credentials
        +
    Invalid Credentials
        +
    Expired Credentials
        +
    Missing Credentials
        +
    Token Validation

---

# 134. API Security Testing

Question:

    How would you include API security testing?

Answer:

    Build
        |
        ↓
    Deploy Test Environment
        |
        ↓
    API Security Tests
        |
        ↓
    Results
        |
        ↓
    Deployment Decision

The exact tools depend on the organization's testing stack.

---

# 135. DAST vs SAST

Question:

    What is the difference between SAST and DAST?

Answer:

### SAST

Tests source code without executing the application.

    Source
        |
        ↓
    SAST

### DAST

Tests the running application.

    Running Application
        |
        ↓
    DAST

They identify different classes of security issues.

---

# 136. Where Does DAST Fit?

Question:

    Where would DAST fit in CI/CD?

Answer:

Usually after deploying to a suitable test environment.

    Build
        |
        ↓
    Deploy Test Environment
        |
        ↓
    DAST
        |
        ↓
    Security Gate
        |
        ↓
    Continue

---

# 137. DevSecOps With Test Environment

Question:

    Why is a test environment useful for security testing?

Answer:

It allows:

    Dynamic Testing
        +
    Integration Testing
        +
    API Testing
        +
    DAST

without directly attacking production.

---

# 138. Production Security Validation

Question:

    Should security testing happen in production?

Answer:

Some validation can happen in production, but destructive or
aggressive security testing should generally occur in controlled
environments.

Production can use:

    Configuration Validation
        +
    Runtime Monitoring
        +
    Access Verification
        +
    Safe Smoke Tests

---

# 139. DevSecOps and Compliance

Question:

    How does DevSecOps support compliance?

Answer:

Automation can provide evidence of:

    Code Review
        +
    Security Scans
        +
    Approvals
        +
    Artifact Traceability
        +
    Deployment History
        +
    Access Control

---

# 140. Audit Trail

Question:

    What should a DevSecOps audit trail contain?

Answer:

    Commit
        +
    Pull Request
        +
    Reviewer
        +
    Security Results
        +
    Artifact
        +
    Deployment
        +
    Approver
        +
    Environment
        +
    Identity

---

# 141. Security and Change Management

Question:

    How does DevSecOps integrate with change management?

Answer:

    Code Change
        |
        ↓
    Automated Validation
        |
        ↓
    Security
        |
        ↓
    Review
        |
        ↓
    Approval
        |
        ↓
    Deployment
        |
        ↓
    Validation

Automation provides evidence for the change.

---

# 142. DevSecOps and Separation of Duties

Question:

    Why is separation of duties important in DevSecOps?

Answer:

It reduces the risk of one compromised account or individual
controlling the entire release path.

For example:

    Developer
        |
        ↓
    Code

    Reviewer
        |
        ↓
    Approval

    CI
        |
        ↓
    Build

    Release Approver
        |
        ↓
    Production

---

# 143. Production Deployment Approval

Question:

    How would you ensure production deployment requires an
    authorized person?

Answer:

Use:

    Protected Environment
        +
    Required Reviewers
        +
    Branch Protection
        +
    Restricted Production Role

---

# 144. DevSecOps and Protected Environments

Question:

    Why are GitHub protected environments useful?

Answer:

They can provide:

    Deployment Approval
        +
    Environment-Specific Secrets
        +
    Environment Protection

This creates stronger controls around production.

---

# 145. Environment-Specific Secrets

Question:

    Why should DEV and PROD secrets be separated?

Answer:

Because compromise of DEV should not automatically provide access
to production.

Architecture:

    DEV Workflow
        |
        ↓
    DEV Credentials

    PROD Workflow
        |
        ↓
    PROD Credentials

---

# 146. Production Credential Blast Radius

Question:

    How do you reduce the blast radius of compromised CI
    credentials?

Answer:

Use:

    OIDC
        +
    Short-Lived Credentials
        +
    Least Privilege
        +
    Environment Separation
        +
    Restricted Trust
        +
    Audit

---

# 147. Compromised Developer Account

Question:

    A developer account is compromised. How would your pipeline
    architecture limit production impact?

Answer:

A secure design should require:

    Protected Branch
        +
    Review
        +
    Security Checks
        +
    Protected Environment
        +
    Production Approval
        +
    Restricted IAM

One compromised account should not automatically control production.

---

# 148. Compromised CI Workflow

Question:

    A malicious workflow modification is merged. How can your
    architecture limit damage?

Answer:

Use:

    CODEOWNERS
        +
    Required Reviews
        +
    Restricted Token Permissions
        +
    OIDC Trust Conditions
        +
    Protected Environments
        +
    Least Privilege

---

# 149. DevSecOps Supply Chain Attack

Question:

    A compromised dependency attempts to steal CI credentials.
    How would you reduce the impact?

Answer:

Use:

    Least Privilege
        +
    OIDC
        +
    Restricted Network
        +
    Minimal Secrets
        +
    Dependency Pinning
        +
    Trusted Registries
        +
    Ephemeral Runners

---

# 150. Complete DevSecOps Architecture Scenario

Question:

    Design a production-grade DevSecOps pipeline for a
    microservices application using:

    - GitHub Actions
    - Jenkins where required
    - Docker
    - ECR
    - Terraform
    - Helm
    - ArgoCD
    - EKS
    - SonarQube
    - Trivy
    - Veracode
    - Prometheus
    - Grafana
    - ELK

    Requirements:

    - Secure CI/CD
    - No long-lived AWS keys
    - Production approval
    - Vulnerability scanning
    - Immutable artifacts
    - GitOps
    - High availability
    - Rollback
    - Auditability

Answer:

## Complete Architecture

    Developer
        |
        ↓
    GitHub
        |
        ↓
    Pull Request
        |
        ↓
    Branch Protection
        |
        ↓
    GitHub Actions
        |
        +--- Unit Tests
        |
        +--- SonarQube
        |
        +--- Dependency Security
        |
        +--- Veracode
        |
        +--- Secret Detection
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
    Immutable Image
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
    Production Validation

---

## AWS Authentication

Use:

    GitHub Actions
        |
        ↓
    OIDC
        |
        ↓
    IAM Role
        |
        ↓
    Temporary Credentials

No long-lived AWS access keys should be required for normal CI/CD.

---

## IAM Design

Separate:

    DEV Role
        +
    QA Role
        +
    PROD Role

Production should have the strongest restrictions.

---

## Source Security

Use:

    Pull Request
        +
    Required Review
        +
    Branch Protection
        +
    CODEOWNERS

---

## CI Security

Use:

    SonarQube
        +
    Dependency Scanning
        +
    Veracode
        +
    Secret Detection

---

## Container Security

Use:

    Trusted Base Image
        +
    Non-Root User
        +
    Trivy
        +
    Immutable Image
        +
    ECR

---

## Infrastructure Security

Terraform should be:

    Version Controlled
        +
    Reviewed
        +
    Validated
        +
    Security Scanned
        +
    Planned
        +
    Approved
        +
    Applied

---

## Kubernetes Security

Use:

    RBAC
        +
    Network Policies
        +
    Resource Limits
        +
    Non-Root Containers
        +
    Secure Service Accounts
        +
    Image Validation

---

## GitOps Security

    CI
        |
        ↓
    Approved Image
        |
        ↓
    GitOps Repository
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
    EKS

Git remains the desired-state source of truth.

---

## Production Security

Production should require:

    Protected Environment
        +
    Authorized Approval
        +
    Restricted IAM
        +
    Immutable Artifact
        +
    Deployment Validation

---

## Deployment Security

Use:

    Rolling
        OR
    Canary
        OR
    Blue-Green

depending on application risk.

---

## Runtime Security

Monitor:

    Application Logs
        +
    Metrics
        +
    Deployment Changes
        +
    Authentication
        +
    Authorization
        +
    Security Events

---

## Rollback

    Deployment
        |
        ↓
    Health Validation
        |
       / \
   Healthy Unhealthy
      |       |
      ↓       ↓
 Continue  Rollback
              |
              ↓
       Known-Good Artifact
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

# 151. DevSecOps Interview Answer Framework

For almost any DevSecOps scenario, answer in this order:

    1. Identify the security risk
            |
            ↓
    2. Assess severity and exposure
            |
            ↓
    3. Contain the risk
            |
            ↓
    4. Fix the root cause
            |
            ↓
    5. Validate the fix
            |
            ↓
    6. Deploy safely
            |
            ↓
    7. Monitor
            |
            ↓
    8. Document
            |
            ↓
    9. Prevent recurrence

A strong interview response sounds like:

    "First I would assess the severity and exposure of the issue.
     Then I would contain the risk and prevent further promotion if
     necessary. I would identify the root cause, apply the appropriate
     fix, rerun security and functional validation, and deploy through
     the controlled CI/CD process. After deployment I would monitor
     the application and security signals, document the incident, and
     add preventive controls so the same issue is detected earlier in
     the future."

---

# 152. DevSecOps Golden Rules

## Rule 1

    Security should start before deployment.

---

## Rule 2

    Never treat a security scan failure as a successful scan.

---

## Rule 3

    Never commit production credentials to Git.

---

## Rule 4

    Treat exposed credentials as compromised.

---

## Rule 5

    Prefer short-lived credentials over long-lived keys.

---

## Rule 6

    Use least privilege everywhere.

---

## Rule 7

    Do not give untrusted code production credentials.

---

## Rule 8

    Protect CI/CD workflow files.

---

## Rule 9

    Pin trusted third-party actions.

---

## Rule 10

    Scan dependencies and container images continuously.

---

## Rule 11

    Use immutable artifacts for production.

---

## Rule 12

    Separate DEV, QA, and PROD permissions.

---

## Rule 13

    Keep GitOps repositories protected.

---

## Rule 14

    Test rollback and recovery.

---

## Rule 15

    Security exceptions must be documented and approved.

---

# 153. Final DevSecOps Mindset

DevSecOps is not:

    Development
        |
        ↓
    Operations
        |
        ↓
    Security Team At The End

It is:

    Development
        |
        +--- Security
        |
        ↓
    Build
        |
        +--- Security
        |
        ↓
    Artifact
        |
        +--- Security
        |
        ↓
    Deployment
        |
        +--- Security
        |
        ↓
    Runtime
        |
        +--- Security
        |
        ↓
    Continuous Improvement

A mature DevSecOps engineer thinks about:

    Identity
        +
    Access
        +
    Source Code
        +
    Dependencies
        +
    Containers
        +
    Infrastructure
        +
    Kubernetes
        +
    CI/CD
        +
    Artifacts
        +
    Secrets
        +
    Runtime
        +
    Monitoring
        +
    Incident Response

The final objective is:

    Secure By Design
        |
        ↓
    Secure By Default
        |
        ↓
    Continuously Validated
        |
        ↓
    Safely Deployed
        |
        ↓
    Continuously Monitored
        |
        ↓
    Quickly Recoverable