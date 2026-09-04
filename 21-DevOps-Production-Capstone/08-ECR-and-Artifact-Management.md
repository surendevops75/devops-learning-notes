# ECR and Artifact Management

## 1. Purpose

This document defines the production artifact-management architecture for the DevOps Production Capstone.

The objective is to build a secure and reliable software supply chain:

```text
Developer
   |
GitLab
   |
CI Pipeline
   |
Build
   |
Test
   |
Security Scan
   |
SBOM
   |
Image Sign
   |
Amazon ECR
   |
GitOps
   |
ArgoCD
   |
EKS
```

The artifact platform must support:

```text
Security
Traceability
Immutability
Reproducibility
Availability
Rollback
Retention
Multi-environment promotion
Multi-region recovery
Cost control
Compliance
```

---

# 2. What Is an Artifact?

An artifact is a versioned output produced by a software delivery process.

Examples:

```text
Container image
Helm chart
Binary
Package
SBOM
Signature
Configuration bundle
```

For this capstone, the primary artifact is the container image.

Example:

```text
roboshop/catalogue
roboshop/user
roboshop/cart
roboshop/payment
```

---

# 3. Why Artifact Management Matters

A production deployment should not build code again during deployment.

Preferred model:

```text
Build once
   |
Scan once
   |
Store once
   |
Promote same artifact
```

Avoid:

```text
Build Dev image
Build Staging image
Build Production image
```

because the artifacts may differ.

---

# 4. Build Once, Promote Many

Recommended:

```text
Git Commit
    |
CI Build
    |
Image SHA
    |
ECR
    |
Dev
    |
Staging
    |
Production
```

The exact same image digest should be promoted.

---

# 5. Amazon ECR

Amazon Elastic Container Registry provides managed container image storage integrated with AWS.

Conceptually:

```text
GitLab CI
   |
Docker/BuildKit
   |
ECR Repository
   |
EKS
```

ECR integrates with:

```text
IAM
EKS
AWS Organizations
CloudTrail
EventBridge
KMS
```

where applicable.

---

# 6. ECR Repository Strategy

Possible approaches:

```text
One repository per application
```

Example:

```text
catalogue
user
cart
payment
shipping
```

This is generally clearer than placing unrelated application images into one large repository.

---

# 7. Environment Separation

Avoid unnecessary duplication such as:

```text
catalogue-dev
catalogue-stage
catalogue-prod
```

when the same artifact can be promoted.

Prefer:

```text
catalogue
```

with environment deployment controlled by GitOps.

---

# 8. Multi-Account Strategy

For stronger isolation:

```text
Dev Account
   |
Dev ECR

Staging Account
   |
Staging ECR

Production Account
   |
Production ECR
```

Another model is:

```text
Central Artifact Account
   |
ECR
   |
Cross-account pull
```

The correct choice depends on security, organizational boundaries, and recovery requirements.

---

# 9. Production Recommendation

For a large organization, consider:

```text
Central artifact strategy
+
production account isolation
+
controlled cross-account access
```

Production workloads should not automatically have write permissions to their own image repositories.

---

# 10. ECR Naming

Use predictable repository names.

Example:

```text
roboshop/catalogue
roboshop/user
roboshop/cart
roboshop/payment
```

Or:

```text
platform/catalogue
platform/user
```

Choose one convention and enforce it.

---

# 11. Repository Ownership

Define:

```text
Application Team
    |
Owns application repository

Platform Team
    |
Owns ECR platform policies

Security Team
    |
Defines scanning/signing policy
```

Avoid unclear ownership.

---

# 12. ECR Permissions

CI requires permissions such as:

```text
GetAuthorizationToken
BatchCheckLayerAvailability
InitiateLayerUpload
UploadLayerPart
CompleteLayerUpload
PutImage
```

The exact policy should be restricted to required repositories.

---

# 13. Pull Permissions

EKS nodes or workload identities need appropriate ECR read permissions.

Typical actions include:

```text
ecr:GetAuthorizationToken
ecr:BatchCheckLayerAvailability
ecr:GetDownloadUrlForLayer
ecr:BatchGetImage
```

Use least privilege.

---

# 14. Push vs Pull Roles

Separate:

```text
CI Push Role
```

from:

```text
EKS Pull Role
```

CI:

```text
Write
```

EKS:

```text
Read
```

This reduces blast radius.

---

# 15. GitLab OIDC to ECR

Recommended CI authentication:

```text
GitLab
   |
OIDC token
   |
AWS STS
   |
AssumeRoleWithWebIdentity
   |
IAM Role
   |
ECR
```

Avoid storing:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

as long-lived CI variables.

---

# 16. ECR Login

A CI job can authenticate to ECR using AWS-supported authentication.

Conceptually:

```bash
aws ecr get-login-password \
  --region "$AWS_REGION" \
  | docker login \
      --username AWS \
      --password-stdin \
      "$ECR_REGISTRY"
```

The credentials should come from short-lived AWS identity.

---

# 17. CI Build Flow

Production pipeline:

```text
Checkout
   |
Unit Tests
   |
Build
   |
Container Scan
   |
SBOM
   |
Sign
   |
Push ECR
   |
Record Digest
   |
Update GitOps
```

Do not deploy an image before required security gates pass.

---

# 18. Image Tags

Example:

```text
catalogue:1.8.4
```

Tags are human-friendly references.

But production deployments should ultimately resolve to an immutable digest.

---

# 19. Image Digest

Example:

```text
catalogue@sha256:abcdef...
```

A digest identifies the exact image content.

This is stronger than:

```text
catalogue:latest
```

---

# 20. Why Avoid Latest?

`latest` can change.

Today:

```text
latest -> image A
```

Tomorrow:

```text
latest -> image B
```

A deployment referencing `latest` is not reproducible.

Production should avoid mutable tags as the deployment identity.

---

# 21. Immutable Tags

Use version tags:

```text
1.8.4
```

or:

```text
release-2026.08.29
```

Ideally configure repository policies to prevent tag mutation where supported by the chosen ECR configuration.

---

# 22. Git SHA Tags

Useful pattern:

```text
catalogue:git-8f72ab1
```

Advantages:

```text
Traceability
Easy correlation with Git
```

Still record the digest as the authoritative artifact identity.

---

# 23. Semantic Version Tags

Example:

```text
catalogue:2.4.1
```

Useful for:

```text
Human readability
Release management
Promotion
```

But never assume a version tag alone guarantees immutability unless repository policy enforces it.

---

# 24. Multiple Tags

One image can have:

```text
catalogue:2.4.1
catalogue:git-8f72ab1
```

Both may resolve to the same digest.

GitOps should preferably deploy:

```text
digest
```

or an immutable tag.

---

# 25. Digest Recording

CI should record:

```text
Repository
Tag
Digest
Git commit
Pipeline ID
Build time
SBOM reference
Signature
```

Example:

```text
catalogue
version=2.4.1
commit=8f72ab1
digest=sha256:...
pipeline=19382
```

---

# 26. Traceability

Production should answer:

```text
Which Git commit produced this image?
Which pipeline built it?
Which dependencies were included?
Which scanner evaluated it?
Which deployment used it?
```

This is supply-chain traceability.

---

# 27. OCI Image

Container images follow OCI-compatible standards.

A registry stores:

```text
Manifest
Layers
Configuration
Metadata
```

---

# 28. Image Layers

Example:

```text
Base OS layer
     +
Runtime layer
     +
Dependencies
     +
Application
```

Docker/BuildKit can reuse unchanged layers.

---

# 29. Layer Optimization

Bad:

```text
COPY entire source
RUN install everything
```

Better:

```text
Copy dependency definition
Install dependencies
Copy application source
```

This improves build caching.

---

# 30. Multi-Stage Docker Builds

Example:

```dockerfile
FROM node:22 AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

FROM node:22-alpine

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules

CMD ["node", "dist/server.js"]
```

The production image should contain only required runtime content.

---

# 31. Minimal Base Images

Possible choices:

```text
Alpine
Distroless
Minimal Debian/Ubuntu
Official language runtime images
```

Evaluate:

```text
Compatibility
Security
Debuggability
Support lifecycle
```

Do not choose a tiny image if it breaks runtime requirements.

---

# 32. Base Image Pinning

Avoid uncontrolled:

```dockerfile
FROM ubuntu:latest
```

Prefer a specific version or digest strategy.

Example:

```dockerfile
FROM ubuntu:24.04
```

For stronger reproducibility, pin the base image digest.

---

# 33. Base Image Lifecycle

Track:

```text
OS support
Runtime support
CVE status
End-of-life date
```

A secure application can become vulnerable because its base image is abandoned.

---

# 34. Image Scanning

Scan before production.

Flow:

```text
Build
 |
Scan
 |
Policy
 |
Push
```

or:

```text
Build
 |
Push
 |
ECR scanning
 |
Policy decision
 |
Promote
```

The pipeline design should ensure that a failed security gate cannot reach production.

---

# 35. Vulnerability Severity

Common severity levels:

```text
CRITICAL
HIGH
MEDIUM
LOW
```

Organizations should define explicit gates.

Example:

```text
Block:
Critical

Review:
High

Track:
Medium/Low
```

Do not blindly use one policy for every application.

---

# 36. Vulnerability Exceptions

Sometimes a vulnerability cannot immediately be fixed.

Use a controlled exception process:

```text
CVE
 |
Risk assessment
 |
Compensating controls
 |
Owner
 |
Expiration date
 |
Approval
```

Never create permanent undocumented exceptions.

---

# 37. False Positives

Scanners can produce false positives.

Process:

```text
Identify
 |
Validate
 |
Document
 |
Exception if justified
 |
Track expiration
```

Security scanning is not equivalent to security engineering.

---

# 38. ECR Enhanced Scanning

ECR can integrate with AWS vulnerability scanning capabilities.

The exact scanning features and supported configuration should be reviewed against the current AWS service capabilities and account setup.

---

# 39. SBOM

Software Bill of Materials identifies software components inside an artifact.

Example:

```text
Application
 |
Dependencies
 |
Open-source libraries
 |
System packages
```

SBOM formats commonly include:

```text
CycloneDX
SPDX
```

---

# 40. Why SBOM?

SBOM helps answer:

```text
Is package X inside our production image?
Which services contain vulnerable library Y?
Which applications use a vulnerable version?
```

This accelerates incident response.

---

# 41. SBOM Generation

Pipeline:

```text
Build image
   |
SBOM generator
   |
SBOM artifact
   |
Store
```

The SBOM should be associated with the exact image digest.

---

# 42. SBOM Example Metadata

```text
image:
catalogue@sha256:...

format:
SPDX

commit:
8f72ab1

build:
19382
```

---

# 43. Image Signing

Signing provides artifact authenticity.

Concept:

```text
Build
 |
Sign image
 |
Registry
 |
Admission verification
 |
EKS
```

Tools may include:

```text
Cosign
Sigstore
AWS-native integrations
```

Select tooling based on enterprise standards.

---

# 44. Why Sign Images?

Without signing:

```text
Repository access
   |
Push malicious image
   |
Cluster pulls it
```

With signature verification:

```text
Image
 |
Signature verification
 |
Trusted?
 |
Deploy
```

---

# 45. Signing Identity

The signing identity should be tied to:

```text
CI workload
Repository
Organization
Environment
```

Avoid using a developer's personal signing key for production artifacts.

---

# 46. Keyless Signing

Modern signing approaches can use short-lived identities and transparency mechanisms rather than long-lived private keys.

This reduces key-management risk.

---

# 47. Admission Verification

For stronger supply-chain security:

```text
Developer
 |
CI
 |
Build
 |
Scan
 |
Sign
 |
ECR
 |
Kubernetes admission
 |
Verify signature
 |
Allow
```

An untrusted image should be rejected.

---

# 48. Policy as Code

Example policy:

```text
Image must:
- come from approved ECR
- use immutable digest
- have valid signature
- pass vulnerability policy
- have required metadata
```

Policy engines may include:

```text
Kyverno
OPA Gatekeeper
```

depending on platform standards.

---

# 49. Image Provenance

Provenance describes:

```text
Where was artifact built?
From which source?
Using which pipeline?
Using which builder?
```

This is increasingly important for software supply-chain security.

---

# 50. SLSA Concepts

SLSA provides a framework for improving software supply-chain integrity.

Concepts include:

```text
Build provenance
Source integrity
Builder trust
Artifact integrity
```

Adopt the level appropriate for organizational maturity.

---

# 51. Trusted Builder

Production builds should run on controlled CI infrastructure.

Avoid:

```text
Developer laptop
   |
Build
   |
Production image
```

Prefer:

```text
Git
 |
Controlled CI runner
 |
Reproducible build
```

---

# 52. GitLab Runner Security

CI runners should be treated as privileged infrastructure.

Protect against:

```text
Secret theft
Docker socket abuse
Cross-job contamination
Untrusted code execution
Supply-chain attacks
```

Use isolated runners for sensitive production pipelines where appropriate.

---

# 53. Docker-in-Docker

Docker-in-Docker can work but introduces complexity.

Evaluate:

```text
BuildKit
Kaniko
Rootless builders
Dedicated build runners
```

Choose based on security and operational requirements.

---

# 54. BuildKit

BuildKit provides advanced image building and caching capabilities.

Benefits include:

```text
Parallel builds
Efficient cache
Multi-platform builds
Secret handling
Modern build features
```

---

# 55. Build Secrets

Never put secrets into:

```dockerfile
ARG SECRET=...
```

or:

```dockerfile
ENV AWS_SECRET=...
```

because they can leak into image history or layers.

Use secure build secret mechanisms.

---

# 56. Runtime Secrets

Runtime secrets belong outside the image.

Preferred:

```text
AWS Secrets Manager
       |
External secret integration
       |
Kubernetes
       |
Pod
```

Never bake production passwords into container images.

---

# 57. Artifact Promotion

Promotion should move the same immutable artifact:

```text
ECR digest
 |
Dev
 |
Validation
 |
Staging
 |
Approval
 |
Production
```

Not:

```text
Rebuild for production
```

---

# 58. GitOps Image Update

Example:

```yaml
image:
  repository: 123456789.dkr.ecr.ap-south-1.amazonaws.com/catalogue
  digest: sha256:abcdef...
```

ArgoCD reconciles the desired state.

---

# 59. Image Updater

Automation can update GitOps manifests after successful CI.

Concept:

```text
CI
 |
Push image
 |
Get digest
 |
Update GitOps repo
 |
Pull request / commit
 |
ArgoCD
```

The exact implementation should preserve review and security controls.

---

# 60. Environment Promotion

Example:

```text
Dev:
digest A

Staging:
digest A

Production:
digest A
```

If testing fails:

```text
Production
still remains on previous digest
```

---

# 61. Rollback

Rollback means restoring a known-good artifact.

Example:

```text
Current:
sha256:BBB

Previous:
sha256:AAA
```

GitOps changes:

```text
BBB -> AAA
```

ArgoCD reconciles the rollback.

---

# 62. Artifact Retention

Retention must balance:

```text
Rollback capability
Storage cost
Compliance
Security
```

Do not delete all old images immediately.

---

# 63. Retention Strategy

Example:

```text
Production releases:
Keep 30 versions

Non-production:
Keep 10 versions

Unreferenced old artifacts:
Lifecycle cleanup
```

Actual values should follow organizational requirements.

---

# 64. ECR Lifecycle Policies

Lifecycle rules can expire images based on:

```text
Age
Count
Tags
Untagged status
```

Test policies before production rollout.

---

# 65. Untagged Images

Failed builds or tag changes can leave untagged images.

These can consume storage.

Use lifecycle rules to clean up unreferenced untagged images.

---

# 66. Cache Strategy

Container builds repeatedly download:

```text
Base images
Dependencies
Build layers
```

Use appropriate build caching.

Possible locations:

```text
CI cache
ECR
OCI cache
BuildKit cache
```

---

# 67. ECR Pull Through Cache

ECR supports pull-through cache capabilities for supported upstream registries.

Concept:

```text
EKS / CI
   |
ECR
   |
Upstream registry
```

This can reduce external registry dependencies and centralize image consumption.

Validate current AWS-supported upstreams and configuration.

---

# 68. Approved Base Images

Enterprise strategy:

```text
Public base image
      |
Security scan
      |
Approved internal image
      |
Application build
```

This creates a trusted base-image layer.

---

# 69. Golden Images

Examples:

```text
company/node-runtime
company/python-runtime
company/java-runtime
```

Application teams build on approved foundations.

Platform/security teams maintain them.

---

# 70. Golden Image Pipeline

```text
Upstream image
 |
Security scan
 |
Hardening
 |
SBOM
 |
Sign
 |
ECR
 |
Application teams
```

---

# 71. Base Image Updates

When a base image changes:

```text
New base
 |
Rebuild dependent images
 |
Scan
 |
Test
 |
Promote
```

Do not wait for a production incident to discover stale dependencies.

---

# 72. Dependency Drift

Example:

```text
Application image:
6 months old

Base OS:
security fixes missing
```

Implement automated rebuilds or scheduled dependency updates where appropriate.

---

# 73. Multi-Architecture Images

Example:

```text
catalogue:2.4.1
      |
      +-- amd64
      |
      +-- arm64
```

This is represented through a multi-platform manifest.

---

# 74. Multi-Architecture Build

Concept:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --push \
  -t "$IMAGE" .
```

Validate architecture compatibility before production use.

---

# 75. Architecture Testing

Do not assume:

```text
Build succeeds
```

means:

```text
Application works on arm64
```

Test:

```text
Startup
Dependencies
Native libraries
Performance
```

---

# 76. ECR Cross-Region Replication

For DR:

```text
Primary ECR
    |
Replication
    |
DR ECR
```

This reduces recovery dependency on the primary region.

---

# 77. Cross-Account Replication

Possible architecture:

```text
Build Account
    |
ECR
    |
Replication / controlled access
    |
Production Account
```

Production should only consume approved artifacts.

---

# 78. Replication Validation

Do not assume replication works.

Test:

```text
Image
 |
Push primary
 |
Replication
 |
Verify digest in DR
```

The digest must match.

---

# 79. Artifact Availability

For production:

```text
Artifact
 |
Multiple availability mechanisms
```

Consider:

```text
Regional ECR
Cross-region replication
Backup strategy
```

---

# 80. DR Artifact Flow

If primary region fails:

```text
DR EKS
   |
DR ECR
   |
Same image digest
   |
ArgoCD
```

The DR cluster should not depend on pulling an image only available in the failed region.

---

# 81. ECR Encryption

ECR supports encryption at rest.

Organizations may use:

```text
AWS-managed encryption
```

or appropriate:

```text
Customer managed KMS key
```

depending on compliance and key-management requirements.

---

# 82. KMS Considerations

If using a customer-managed key:

```text
ECR
 |
KMS
```

Review:

```text
Key policy
IAM
Rotation
Deletion protection
Cross-account access
DR implications
```

---

# 83. CloudTrail

Track important ECR API operations where CloudTrail coverage applies.

Useful for:

```text
Who pushed?
Who deleted?
Who changed repository configuration?
```

---

# 84. ECR Repository Policy

Repository policies can control cross-account access.

Example conceptual policy:

```text
Production ECR
 |
Allow specific CI role
Allow specific EKS pull identity
Deny unauthorized principals
```

Always validate both identity-based and resource-based policy behavior.

---

# 85. Public ECR vs Private ECR

For production application images:

```text
Private ECR
```

is normally appropriate.

Public ECR is intended for publicly distributable artifacts.

---

# 86. ECR Access from Private Subnets

Private EKS nodes may access ECR using:

```text
VPC endpoints
```

for required ECR APIs and supporting services, or:

```text
NAT
```

depending on architecture.

---

# 87. ECR Networking

Common components:

```text
ECR API endpoint
ECR DKR endpoint
S3
STS where required
DNS
Route tables
Security controls
```

Validate private-cluster connectivity carefully.

---

# 88. ECR Pull Failure

Symptoms:

```text
ImagePullBackOff
ErrImagePull
```

Check:

```text
Repository exists
Image exists
Tag/digest correct
IAM permissions
Network
DNS
ECR endpoints
S3 access path
```

---

# 89. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What can you do?
```

A valid AWS identity can still receive:

```text
AccessDenied
```

if permissions are insufficient.

---

# 90. ECR AccessDenied

Troubleshooting:

```text
1. Identify caller identity.
2. Check IAM role.
3. Check repository policy.
4. Check SCP.
5. Check permission boundary.
6. Check region.
7. Check ECR repository.
```

---

# 91. AWS Organizations SCP

An ECR permission may appear correct but be blocked by:

```text
Service Control Policy
```

Production troubleshooting should consider organization-level controls.

---

# 92. Permission Boundaries

An IAM role may have:

```text
Allow ECR Push
```

but its permission boundary may prevent the action.

Always inspect effective permissions.

---

# 93. ECR Repository Not Found

Possible causes:

```text
Wrong AWS account
Wrong region
Wrong repository name
Repository not created
Incorrect CI variable
```

Verify the active AWS identity before debugging Kubernetes.

---

# 94. Wrong Region

Example:

```text
CI pushes:
ap-south-1

EKS pulls:
us-east-1
```

The image reference must point to the intended registry endpoint.

---

# 95. Digest Mismatch

If the expected digest differs:

```text
Stop promotion.
```

Investigate:

```text
Tag mutation
Replication
Wrong repository
Wrong build
```

Never ignore an unexpected digest.

---

# 96. Supply-Chain Threat Model

Threats include:

```text
Compromised dependency
Compromised base image
Malicious developer commit
Compromised CI runner
Registry compromise
Credential theft
Image tag mutation
Artifact substitution
```

Controls must cover the entire chain.

---

# 97. Supply-Chain Defense

```text
Source control
   |
Branch protection
   |
CI isolation
   |
Dependency scanning
   |
Image scanning
   |
SBOM
   |
Signing
   |
Provenance
   |
Trusted registry
   |
Admission policy
   |
Runtime monitoring
```

---

# 98. Dependency Scanning

Before image build, scan application dependencies.

Examples:

```text
npm audit
pip-audit
Maven dependency scanning
OS package scanning
```

Tool selection depends on language and organization.

---

# 99. Secret Scanning

CI should detect secrets in:

```text
Source
Dockerfile
Build context
Configuration
Git history where possible
```

Do not allow accidental credentials into images.

---

# 100. Image History

Inspect:

```bash
docker history <image>
```

Look for:

```text
Secrets
Tokens
Internal URLs
Unnecessary files
Build credentials
```

---

# 101. Runtime Image Hardening

Production image should ideally:

```text
Run as non-root
Contain minimal packages
Use read-only filesystem where possible
Drop capabilities
Avoid shells when practical
Avoid package managers in runtime image
```

---

# 102. Dockerfile Security

Avoid:

```dockerfile
USER root
```

as the permanent runtime identity when unnecessary.

Prefer:

```dockerfile
USER 10001
```

or an appropriate non-root user.

---

# 103. Package Manager Cleanup

If runtime does not need package management:

```text
Do not ship apt
Do not ship npm CLI
Do not ship build toolchain
```

Multi-stage builds help remove them.

---

# 104. Debugging Minimal Images

Minimal images can be harder to debug.

Production strategy:

```text
Minimal runtime image
+
Ephemeral debug tooling
```

rather than permanently shipping large debugging toolsets.

---

# 105. Artifact Metadata

Recommended labels:

```text
org.opencontainers.image.source
org.opencontainers.image.revision
org.opencontainers.image.version
org.opencontainers.image.created
```

These improve traceability.

---

# 106. Example Docker Labels

Conceptually:

```dockerfile
LABEL org.opencontainers.image.source="https://git.example/repo"
LABEL org.opencontainers.image.revision="$GIT_SHA"
LABEL org.opencontainers.image.version="$VERSION"
```

Use the appropriate CI/build mechanism for injecting values.

---

# 107. Reproducible Builds

Goal:

```text
Same source
+
Same build inputs
=
Predictable artifact
```

Control:

```text
Dependencies
Base image
Build tools
Environment
Build process
```

---

# 108. Dependency Lock Files

Examples:

```text
package-lock.json
poetry.lock
requirements lock
go.sum
pom dependency management
```

Use lock mechanisms appropriate to the ecosystem.

---

# 109. Artifact Promotion Policy

Example:

```text
Development:
Automatic

Staging:
Automatic after tests

Production:
Approval required
```

This provides a balance between delivery speed and production control.

---

# 110. Production Approval

Approval should happen before:

```text
Production GitOps change
```

not by rebuilding the image.

The approved digest is promoted.

---

# 111. Four-Eyes Principle

For high-risk production releases:

```text
Author
+
Independent reviewer
```

can be required.

This reduces single-person deployment risk.

---

# 112. Emergency Releases

Emergency process should still preserve:

```text
Traceability
Approval
Audit
Artifact identity
Rollback capability
```

Do not bypass every control during an incident.

---

# 113. Artifact Rollback Strategy

Maintain:

```text
Current digest
Previous known-good digest
Release history
```

GitOps repository provides the desired-state history.

ECR provides artifact availability.

---

# 114. Deleted Artifact Risk

If an old production image is deleted:

```text
Rollback may fail
```

Therefore retention policy must account for:

```text
Rollback window
DR requirements
Compliance
```

---

# 115. Artifact Garbage Collection

Clean only artifacts that are no longer needed.

Consider references from:

```text
Production
Staging
DR
Release records
Rollback windows
```

---

# 116. ECR Lifecycle Policy Design

A mature policy may distinguish:

```text
Tagged release images
Untagged images
Development images
Feature branch images
```

Example:

```text
Delete untagged after 7 days
Keep last 30 release images
Keep production release history longer
```

Exact retention must be approved.

---

# 117. Feature Branch Images

Feature branches may produce:

```text
catalogue:feature-123-8f72ab1
```

These should have shorter retention.

Otherwise registry storage can grow rapidly.

---

# 118. CI Failure Images

Failed pipelines may produce images that should never be promoted.

Mark or retain them only as required for debugging.

Do not allow failed build artifacts to become production candidates.

---

# 119. Artifact Registry Governance

Define:

```text
Naming
Ownership
Retention
Scanning
Signing
Access
Replication
Encryption
Audit
```

as organizational standards.

---

# 120. ECR Repository Creation

Terraform should create repositories rather than engineers creating them manually.

Example:

```hcl
resource "aws_ecr_repository" "catalogue" {
  name                 = "roboshop/catalogue"
  image_tag_mutability = "IMMUTABLE"
}
```

Validate current provider/resource behavior before applying.

---

# 121. ECR Terraform Variables

Possible variables:

```text
repository_name
tag_mutability
scan_configuration
encryption_type
kms_key
lifecycle_rules
tags
```

---

# 122. ECR Lifecycle Terraform

Manage lifecycle policy through infrastructure as code.

This provides:

```text
Version control
Review
Repeatability
Auditability
```

---

# 123. ECR Repository Policy Terraform

Cross-account permissions should also be managed through Terraform.

Avoid manually editing production repository policies.

---

# 124. ECR Replication Terraform

Replication configuration should be declarative.

Example concept:

```text
Source registry
 |
Replication rule
 |
Destination region/account
```

---

# 125. ECR Terraform Guardrails

Production module should reject:

```text
Unapproved repository naming
Public repository
Mutable production tags
Missing encryption
Missing lifecycle policy
```

where such controls are required.

---

# 126. CI Variables

Keep non-secret configuration such as:

```text
AWS_REGION
ECR_REGISTRY
ECR_REPOSITORY
```

separate from sensitive credentials.

With OIDC, long-lived AWS credentials should not be necessary.

---

# 127. ECR CI Example

Concept:

```yaml
build:
  script:
    - docker build -t "$ECR_IMAGE:$CI_COMMIT_SHA" .
    - docker push "$ECR_IMAGE:$CI_COMMIT_SHA"
    - docker inspect --format='{{index .RepoDigests 0}}' \
        "$ECR_IMAGE:$CI_COMMIT_SHA"
```

Production implementation should use secure build and authentication practices.

---

# 128. Digest-Based GitOps Update

CI can extract:

```text
sha256:...
```

and update:

```yaml
image:
  repository: ...
  digest: sha256:...
```

This makes deployment deterministic.

---

# 129. GitOps Commit

Example flow:

```text
Application CI
 |
Build image
 |
Push ECR
 |
Get digest
 |
Create GitOps PR
 |
Review
 |
Merge
 |
ArgoCD sync
```

This provides an auditable promotion path.

---

# 130. ArgoCD Verification

ArgoCD should show:

```text
Application
Image
Revision
Health
Sync status
```

This allows operators to correlate:

```text
Git
ECR
Kubernetes
```

---

# 131. Production Release Record

For every production release record:

```text
Application
Version
Image digest
Git commit
GitOps commit
CI pipeline
Approver
Deployment timestamp
Rollback version
```

---

# 132. Artifact Incident Investigation

If a malicious image is suspected:

```text
1. Identify digest.
2. Identify affected environments.
3. Identify CI pipeline.
4. Identify source commit.
5. Inspect SBOM.
6. Check signature/provenance.
7. Quarantine artifact.
8. Stop promotion.
9. Roll back workloads.
10. Rotate affected credentials.
11. Investigate CI/registry access.
```

---

# 133. Compromised CI Runner

If runner compromise is suspected:

```text
Stop runner
 |
Revoke temporary access
 |
Review recent builds
 |
Identify affected artifacts
 |
Verify signatures/provenance
 |
Rebuild from trusted infrastructure
 |
Rotate secrets if necessary
```

---

# 134. Malicious Base Image

If a base image is compromised:

```text
Identify affected versions
 |
Find dependent images through SBOM
 |
Rebuild
 |
Scan
 |
Sign
 |
Promote
```

SBOM greatly accelerates this process.

---

# 135. Vulnerable Library Incident

Example:

```text
CVE-XXXX
```

Process:

```text
Search SBOM inventory
 |
Identify affected digests
 |
Rebuild with fixed dependency
 |
Scan
 |
Promote
 |
Deploy
```

---

# 136. Registry Outage

If ECR is temporarily unavailable:

```text
Existing pods
```

may continue running if their images are already present on nodes.

New pods requiring unavailable image pulls may fail.

This demonstrates why:

```text
Capacity headroom
+
regional strategy
+
artifact replication
```

matter.

---

# 137. Node Replacement During Registry Outage

If new nodes need to pull images during an ECR outage:

```text
Pod scheduling
 |
Image pull
 |
Failure
```

This can amplify an incident.

DR architecture should account for registry dependencies.

---

# 138. Artifact Caching

Node-local images provide temporary caching.

But do not treat node caches as durable artifact storage.

A terminated node loses that local cache.

---

# 139. ECR Availability Strategy

Production strategy:

```text
Primary ECR
+
Cross-region replication
+
DR ECR
+
Known-good release inventory
```

---

# 140. Regional Failover

During region failure:

```text
DNS / traffic failover
 |
DR ALB
 |
DR EKS
 |
DR ECR
 |
Same image digest
```

All dependencies must also be available in DR.

---

# 141. Artifact Compliance

Compliance may require:

```text
Retention
Encryption
Audit
SBOM
Vulnerability scanning
Provenance
Approval
```

Map each requirement to an automated control.

---

# 142. Artifact Access Reviews

Regularly review:

```text
Who can push?
Who can pull?
Who can delete?
Who can change repository policy?
Who can administer replication?
```

Remove unnecessary access.

---

# 143. Delete Permissions

Production CI usually should not have unrestricted:

```text
ecr:BatchDeleteImage
```

Deletion should be controlled.

Lifecycle policies are safer for routine cleanup.

---

# 144. Break-Glass Access

Emergency registry administration may require a dedicated role.

Example:

```text
break-glass-ecr-admin
```

Protect it with:

```text
MFA
Approval
Logging
Short duration
```

---

# 145. ECR and Kubernetes Separation

ECR stores artifacts.

Kubernetes stores desired deployment state.

ArgoCD manages reconciliation.

Do not mix responsibilities.

```text
ECR:
Artifact

Git:
Desired state

ArgoCD:
Reconciliation

EKS:
Runtime
```

---

# 146. ECR and Helm

Helm values may specify:

```yaml
image:
  repository: ...
  digest: ...
```

The chart should not hardcode:

```text
latest
```

---

# 147. Helm Values by Environment

Example:

```text
values-dev.yaml
values-staging.yaml
values-production.yaml
```

The image identity can be promoted without rebuilding the artifact.

---

# 148. Production Helm Example

```yaml
image:
  repository: 123456789.dkr.ecr.ap-south-1.amazonaws.com/roboshop/catalogue
  digest: sha256:abcdef123456
```

Use digest-aware chart patterns consistently.

---

# 149. ImagePullPolicy

For immutable digests:

```text
Image identity cannot silently change.
```

The exact `imagePullPolicy` should still be selected according to Kubernetes behavior and deployment requirements.

---

# 150. Admission Policy

Example conceptual rule:

```text
IF image registry != approved ECR
THEN reject

IF image uses mutable tag
THEN reject

IF signature missing
THEN reject

IF required metadata missing
THEN reject
```

---

# 151. Policy Enforcement Stages

Best:

```text
CI
 |
Policy
 |
Registry
 |
Admission
 |
Runtime
```

Defense in depth means a missed CI control can still be caught later.

---

# 152. ECR Repository Policy and Admission

These solve different problems.

ECR policy:

```text
Who can access registry?
```

Admission:

```text
Which image may run?
```

Both are useful.

---

# 153. Artifact Integrity

Integrity means:

```text
The image deployed
=
The image that was built
```

Digest and signature verification provide strong controls.

---

# 154. Artifact Authenticity

Authenticity asks:

```text
Was this image produced by a trusted source?
```

Signing and provenance address this.

---

# 155. Artifact Confidentiality

Container images can contain proprietary code.

Use:

```text
Private ECR
IAM
Encryption
Network controls
```

Do not publish private application images publicly.

---

# 156. Artifact Availability

Availability means:

```text
Authorized workloads can obtain required artifacts when needed.
```

Consider:

```text
ECR availability
Network path
IAM
Replication
DR
```

---

# 157. Artifact Lifecycle

Complete lifecycle:

```text
Source
 |
Build
 |
Test
 |
Scan
 |
SBOM
 |
Sign
 |
Publish
 |
Promote
 |
Deploy
 |
Observe
 |
Retain
 |
Expire
```

---

# 158. Artifact State Model

Possible states:

```text
BUILT
SCANNED
APPROVED
SIGNED
PUBLISHED
PROMOTED
DEPLOYED
RETIRED
REVOKED
```

This makes release governance explicit.

---

# 159. Revoked Artifact

If an artifact is compromised:

```text
Mark revoked
 |
Stop promotion
 |
Identify deployments
 |
Rollback
 |
Delete/quarantine if appropriate
```

Do not rely solely on deletion because running pods may already have the image locally.

---

# 160. Running Compromised Image

If a compromised image is running:

```text
Deployment
 |
Scale down / rollback
 |
Verify replacement
 |
Investigate
```

Also determine whether:

```text
Secrets
AWS credentials
Service accounts
```

could have been accessed.

---

# 161. EKS Image Pull Identity

Understand which identity performs the pull.

Depending on architecture:

```text
Node IAM
```

may provide ECR pull access.

The exact EKS/runtime model should be validated.

---

# 162. Node Role Permissions

Do not grant:

```text
ECR full admin
```

to worker nodes.

They generally require pull-related permissions only.

---

# 163. Cross-Account ECR Pull

Architecture:

```text
Production EKS
 |
Node/workload identity
 |
Production account IAM
 |
ECR repository policy
 |
Central artifact account
```

Both sides must allow the access.

---

# 164. Cross-Account Troubleshooting

Check:

```text
Caller role
Repository policy
Identity policy
SCP
Region
ECR endpoint
```

---

# 165. ECR Repository Replication and Security

Replication should not accidentally replicate:

```text
Development junk
Unapproved images
Sensitive test artifacts
```

Use selective rules where supported.

---

# 166. Artifact Inventory

Maintain an inventory containing:

```text
Application
Digest
Version
Environment
Release date
SBOM
Scan status
Signature
Owner
```

This becomes valuable during security incidents.

---

# 167. Continuous Vulnerability Monitoring

A vulnerability can be discovered after deployment.

Therefore:

```text
Scan at build time
+
Monitor deployed artifacts
+
Re-scan when new CVEs appear
```

Build-time scanning alone is insufficient.

---

# 168. CVE Response Automation

Example:

```text
New critical CVE
 |
Find affected image digests
 |
Find environments
 |
Create remediation ticket
 |
Rebuild
 |
Scan
 |
Promote
```

Automation reduces mean time to remediation.

---

# 169. Artifact Metrics

Track:

```text
Images built/day
Scan failure rate
Critical vulnerabilities
Mean remediation time
Promotion duration
Registry storage
Pull failures
Image startup time
```

---

# 170. ECR Cost Optimization

Control:

```text
Old images
Feature branch images
Untagged images
Large base images
Duplicate layers
```

Use lifecycle policies and image optimization.

---

# 171. Image Size

Smaller images generally improve:

```text
Pull time
Startup time
Storage cost
Network transfer
```

But optimize without sacrificing maintainability and security.

---

# 172. Image Pull Performance

For large images:

```text
Cold node
 |
Pull image
 |
Extract layers
 |
Start container
```

Large images increase deployment latency.

Optimize:

```text
Layer ordering
Base image
Dependencies
Unnecessary files
Compression
```

---

# 173. Lazy Image Loading

Advanced image-loading techniques may reduce startup transfer requirements.

Evaluate them only when the runtime/platform supports them and the operational complexity is justified.

---

# 174. ECR Monitoring

Monitor:

```text
Repository size
Image count
Pull activity
Push failures
Replication failures
Security findings
```

---

# 175. CloudWatch/EventBridge

AWS events can be integrated with operational workflows where supported.

Potential use cases:

```text
ECR image events
Replication events
Security events
Automation triggers
```

---

# 176. Production Artifact Dashboard

Grafana or another reporting system can display:

```text
Current production digest
Current version
Last deployment
Scan status
Critical CVEs
Image age
```

---

# 177. Artifact Audit Question

A strong production system should answer:

```text
What exactly is running in production?
```

Answer:

```text
Application
+
Image digest
+
Git commit
+
GitOps commit
+
CI pipeline
```

---

# 178. ECR Architecture

```text
                     GitLab
                        |
                  OIDC Authentication
                        |
                    AWS STS
                        |
                  CI IAM Role
                        |
                     Build
                        |
                Scan / SBOM / Sign
                        |
                        v
                +---------------+
                |     ECR       |
                |               |
                | catalogue     |
                | user          |
                | cart          |
                | payment       |
                +-------+-------+
                        |
             +----------+----------+
             |                     |
          Primary                DR
          Region               Region
             |                     |
           ECR                   ECR
             |                     |
           EKS                   EKS
             |                     |
          ArgoCD                ArgoCD
```

---

# 179. Complete Supply Chain

```text
Developer
   |
GitLab
   |
Protected Branch
   |
CI
   |
Dependency Scan
   |
BuildKit
   |
Container Scan
   |
SBOM
   |
Signature
   |
Provenance
   |
ECR
   |
GitOps PR
   |
Review
   |
ArgoCD
   |
EKS Admission
   |
Running Pod
```

---

# 180. Production ECR Checklist

```text
[ ] Private repositories
[ ] Naming convention
[ ] Terraform-managed repositories
[ ] Immutable tags where required
[ ] Lifecycle policies
[ ] Encryption
[ ] Least-privilege push role
[ ] Least-privilege pull role
[ ] OIDC CI authentication
[ ] Vulnerability scanning
[ ] SBOM
[ ] Image signing
[ ] Provenance
[ ] Admission policy
[ ] Cross-region replication
[ ] Cross-account controls
[ ] Audit logging
[ ] Storage monitoring
[ ] Rollback retention
```

---

# 181. CI/CD Artifact Checklist

```text
[ ] Build once
[ ] Unit tests
[ ] Dependency scan
[ ] Build image
[ ] Image scan
[ ] Generate SBOM
[ ] Sign image
[ ] Push ECR
[ ] Capture digest
[ ] Update GitOps
[ ] Review
[ ] Deploy
[ ] Verify
```

---

# 182. Security Checklist

```text
[ ] No long-lived AWS CI credentials
[ ] No secrets in Dockerfile
[ ] No secrets in image
[ ] Non-root runtime
[ ] Minimal image
[ ] Approved base images
[ ] Vulnerability policy
[ ] Exception process
[ ] Image signing
[ ] Admission verification
[ ] Registry least privilege
```

---

# 183. Disaster Recovery Checklist

```text
[ ] ECR replication
[ ] DR repository access
[ ] Image digest verified
[ ] DR EKS can pull
[ ] GitOps points to valid image
[ ] Secrets available
[ ] Dependencies available
[ ] DNS failover tested
[ ] Rollback tested
```

---

# 184. Interview — Explain Your ECR Architecture

Strong answer:

```text
I use private ECR repositories per application and separate push and
pull permissions. GitLab authenticates to AWS through OIDC and STS
rather than long-lived access keys. CI builds the image once, scans it,
generates an SBOM, signs it, pushes it to ECR, and records the immutable
digest. GitOps promotes that same digest across environments. For
production resilience I also consider cross-region replication and
controlled cross-account access.
```

---

# 185. Interview — Why Digest Instead of Tag?

```text
A tag can be mutable, while a digest identifies the exact image content.
For production I want deterministic deployment, so the GitOps state
should reference an immutable artifact identity such as an image digest.
```

---

# 186. Interview — Why Build Once?

```text
If I rebuild for each environment, the resulting artifacts can differ
even when the source commit is the same. Building once and promoting
the same digest gives us stronger reproducibility and makes rollback
more reliable.
```

---

# 187. Interview — How Do You Secure ECR?

```text
I use private repositories, least-privilege IAM, short-lived CI
credentials through OIDC, encryption, lifecycle policies, vulnerability
scanning, SBOMs, image signing, audit logging, and controlled
cross-account access. At the Kubernetes layer I can additionally
enforce trusted registries and signature policies through admission
controls.
```

---

# 188. Interview — What Happens If ECR Pull Fails?

```text
I first determine whether it is an image, authentication, authorization,
network, or registry issue. I check the image digest or tag, repository
and region, node IAM role, repository policy, SCPs, DNS, ECR endpoints,
and the pod events. I avoid treating ImagePullBackOff as an application
failure until the registry path is validated.
```

---

# 189. Interview — What Is an SBOM?

```text
An SBOM is an inventory of software components contained in an artifact.
It lets us quickly identify which production images contain a vulnerable
library or package when a new CVE is announced.
```

---

# 190. Interview — Why Sign Images?

```text
Scanning tells me about known vulnerabilities, but signing addresses
artifact authenticity. I can verify that an image was produced by a
trusted CI identity and reject images that do not meet the organization's
trust policy.
```

---

# 191. Interview — What If a Base Image Has a Critical CVE?

```text
I identify all dependent production artifacts, ideally through SBOM
inventory, rebuild them using a patched base image, rescan and sign
them, and then promote the new digest. I do not simply change a tag and
assume running workloads have been remediated.
```

---

# 192. Interview — How Do You Roll Back?

```text
I revert the GitOps deployment to the previous known-good image digest.
ArgoCD reconciles the cluster to that state. Because the previous
artifact is immutable and retained according to the rollback policy,
the rollback does not require rebuilding anything.
```

---

# 193. Interview — How Do You Handle ECR in DR?

```text
I replicate approved artifacts to the DR region so the DR EKS cluster
does not depend on the primary region's registry. During a DR test I
verify that the exact production image digest exists in the DR registry
and that the DR cluster can pull and run it.
```

---

# 194. Interview — Why Not Give EKS Push Access?

```text
The cluster only needs to consume approved artifacts. Giving runtime
identities image-push permissions increases the blast radius if a
workload is compromised. I separate CI publishing permissions from
runtime pull permissions.
```

---

# 195. Interview — What Is Supply-Chain Security?

```text
It is protecting the complete path from source code to running software.
I address source protection, dependency security, CI runner security,
artifact scanning, SBOMs, signing, provenance, registry access, admission
controls, and runtime monitoring rather than treating container scanning
as the entire supply-chain solution.
```

---

# 196. Interview — What If an Image Is Compromised?

```text
I identify the digest and all environments using it, stop further
promotion, quarantine or revoke the artifact, roll back workloads to a
known-good digest, investigate the source and CI pipeline, rotate
credentials if necessary, and rebuild from a trusted source. I also
verify whether the compromised workload accessed AWS or Kubernetes
resources.
```

---

# 197. Interview — How Do You Prevent `latest` Problems?

```text
I don't use latest as the production deployment identity. I use immutable
versioning and preferably image digests. Repository tag immutability and
admission policy can provide additional protection against accidental
tag mutation.
```

---

# 198. Interview — How Do You Optimize Image Size?

```text
I use multi-stage builds, minimal supported runtime images, dependency
cleanup, correct Dockerfile layer ordering, and exclusion of development
tools. I balance image size against security, compatibility, and
debuggability rather than choosing the smallest possible image blindly.
```

---

# 199. Interview — How Does ECR Fit Into GitOps?

```text
ECR stores the immutable artifact, while Git stores the desired
deployment state. CI publishes the image and updates the GitOps
repository with its digest. ArgoCD detects that desired-state change
and reconciles it into EKS.
```

---

# 200. Senior Production Architecture Answer

```text
Our artifact platform follows a build-once-promote-many model.
Application code is built in GitLab CI using short-lived AWS
authentication through OIDC. The pipeline performs tests, dependency
and container scanning, SBOM generation, signing, and provenance
generation before publishing the immutable image to private ECR.

Each application has a controlled repository. CI has push permissions,
while EKS has pull-only permissions. Production deployments reference
an immutable image digest rather than a mutable tag. The digest is
promoted through GitOps, so Dev, Staging, and Production run the exact
same artifact.

For resilience, approved production artifacts can be replicated to a
secondary region and, where required, across accounts. Terraform
manages repository configuration, lifecycle policies, encryption,
replication, and access controls. ArgoCD manages Kubernetes deployment
state. Admission policies can verify the image registry, digest,
signature, and other supply-chain requirements before allowing a
workload to run.
```

---

# 201. Final Artifact Architecture

```text
                    SOURCE
                      |
                   GitLab
                      |
                 Protected Code
                      |
                      v
                     CI
                      |
        +-------------+-------------+
        |             |             |
      Test        Dependency      Secret
                   Scan           Scan
        |             |             |
        +-------------+-------------+
                      |
                    Build
                      |
                   BuildKit
                      |
          +-----------+-----------+
          |                       |
       Image                    SBOM
          |                       |
          +-----------+-----------+
                      |
                 Vulnerability
                    Scan
                      |
                    Sign
                      |
                 Provenance
                      |
                      v
                    ECR
                      |
              Immutable Digest
                      |
                      v
                 GitOps PR
                      |
                   Review
                      |
                    Merge
                      |
                    ArgoCD
                      |
                    EKS
                      |
               Admission Policy
                      |
                 Running Pod
```

---

# 202. Final Production Principles

```text
1. Build once.
2. Promote the same artifact.
3. Use immutable image identity.
4. Prefer digest-based production deployment.
5. Separate push and pull permissions.
6. Use short-lived CI authentication.
7. Generate and retain SBOMs.
8. Scan continuously, not only at build time.
9. Sign production artifacts.
10. Verify trust at deployment time.
11. Maintain artifact traceability.
12. Retain enough history for rollback.
13. Replicate critical artifacts for DR.
14. Manage ECR configuration with Terraform.
15. Manage application promotion with GitOps.
16. Treat CI runners as production security boundaries.
17. Keep secrets out of images.
18. Use approved base images.
19. Automate vulnerability remediation.
20. Test the entire artifact recovery path.
```

---

# 203. Definition of Done

The ECR/artifact platform is production-ready when:

```text
A developer commits code
        |
CI securely authenticates
        |
Application is tested
        |
Image is built reproducibly
        |
Image is scanned
        |
SBOM is generated
        |
Image is signed
        |
Image is pushed to ECR
        |
Digest is captured
        |
GitOps is updated
        |
Review occurs
        |
ArgoCD deploys the exact digest
        |
EKS admission verifies policy
        |
Application runs
        |
Metrics/logs/traces confirm health
        |
Previous digest remains available
        |
DR can retrieve the same digest
```

That is the production artifact-management model this capstone should demonstrate.

---