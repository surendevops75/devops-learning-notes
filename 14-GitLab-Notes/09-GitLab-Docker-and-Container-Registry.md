# GitLab Docker and Container Registry

> Production-oriented guide to Docker builds in GitLab CI/CD, GitLab Container Registry, AWS ECR integration, image tagging, immutable digests, authentication, multi-stage builds, caching, security scanning, image promotion, Kubernetes/EKS deployment, GitOps, troubleshooting, and senior DevOps interview scenarios.

---

## 1. Why Containers in GitLab CI/CD?

Containers provide consistent application packaging:

```text
Source
 ↓
Build
 ↓
Docker Image
 ↓
Registry
 ↓
Kubernetes/EKS
```

GitLab CI can automate the complete image lifecycle.

---

## 2. Container Pipeline

Typical production flow:

```text
Git Push
 ↓
GitLab Pipeline
 ↓
Test
 ↓
Build Docker Image
 ↓
Security Scan
 ↓
Push Registry
 ↓
Record Digest
 ↓
GitOps Update
 ↓
ArgoCD
 ↓
EKS
```

---

## 3. Container Registry

A container registry stores Docker/OCI images.

Examples:

- GitLab Container Registry
- Amazon ECR
- JFrog Artifactory
- other OCI-compatible registries

Your AWS/EKS architecture can use ECR as the production image registry.

---

## 4. GitLab Container Registry

GitLab can provide a container registry associated with projects.

Concept:

```text
GitLab Project
     │
     ├── Source
     ├── CI/CD
     └── Container Registry
```

This is convenient when keeping source and image lifecycle together.

---

## 5. Amazon ECR

Amazon Elastic Container Registry stores container images in AWS.

Architecture:

```text
GitLab CI
   ↓
AWS identity
   ↓
ECR
   ↓
EKS
```

ECR is a natural fit for AWS/EKS deployments.

---

## 6. GitLab Registry vs ECR

### GitLab Registry

Useful for:

- GitLab-centric workflows
- development images
- internal CI artifacts
- simple project integration

### ECR

Useful for:

- EKS workloads
- AWS IAM integration
- AWS-native deployments
- production image storage

Choose based on architecture rather than assuming one registry is always better.

---

## 7. Docker Image

An image contains:

```text
Application
+
Runtime
+
Dependencies
+
Filesystem
+
Metadata
```

A container is a running instance of the image.

---

## 8. Image vs Container

```text
Docker Image
     ↓
Immutable package
     ↓
Container
     ↓
Running process
```

The image is stored in a registry.

The container runs the image.

---

## 9. Dockerfile

A Dockerfile defines how an image is built.

Example:

```dockerfile
FROM eclipse-temurin:17-jre
COPY target/app.jar /app/app.jar
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

Use approved and controlled base images.

---

## 10. Docker Build Context

Example:

```bash
docker build -t app:build .
```

The final `.` means:

```text
Current directory = build context
```

Large build contexts increase build time and can accidentally expose files.

---

## 11. `.dockerignore`

Use `.dockerignore` to exclude:

```text
.git
target
node_modules
.env
terraform state
logs
temporary files
```

Example:

```text
.git
.env
*.log
terraform.tfstate
```

---

## 12. Why `.dockerignore` Matters

It reduces:

```text
Build context
Upload time
Image risk
Accidental secret inclusion
```

Never rely only on `.dockerignore` for secret protection.

---

## 13. Multi-Stage Docker Builds

Example:

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
COPY src ./src
RUN mvn package

FROM eclipse-temurin:17-jre
COPY --from=build /app/target/app.jar /app/app.jar
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

The build tools do not need to remain in the final runtime image.

---

## 14. Benefits of Multi-Stage Builds

Benefits:

- smaller images
- lower attack surface
- fewer runtime dependencies
- cleaner production image

This is a standard production pattern.

---

## 15. Image Size

Large images cause:

```text
Slow build
Slow push
Slow pull
Slow deployment
More storage
Larger attack surface
```

Optimize images where practical.

---

## 16. Minimal Runtime Images

Prefer runtime-oriented images where compatible.

Example:

```text
Build image
     ↓
Large tooling
     ↓
Final runtime image
     ↓
Minimal dependencies
```

Do not remove required libraries just to minimize size.

---

## 17. Base Image Selection

Consider:

```text
Security
Support lifecycle
Compatibility
Size
Patch frequency
Architecture
Vendor trust
```

Do not choose a base image solely because it is small.

---

## 18. Pin Base Images

Avoid relying only on:

```dockerfile
FROM ubuntu:latest
```

Prefer controlled versions.

For stronger reproducibility, pin image digests when your operational process supports digest management.

---

## 19. Base Image Supply Chain

Your image depends on:

```text
Base image
 ↓
OS packages
 ↓
Application dependencies
 ↓
Application
```

A vulnerable base image can make an otherwise secure application image vulnerable.

---

## 20. Docker Image Tags

Common tags:

```text
latest
dev
staging
1.4.2
commit-sha
```

Tags are mutable references.

Do not treat a tag as equivalent to immutable content.

---

## 21. Commit SHA Tagging

Recommended:

```text
app:$CI_COMMIT_SHORT_SHA
```

Example:

```text
app:a81c92f
```

This provides source traceability.

---

## 22. Version Tagging

For releases:

```text
app:1.4.2
```

Version tags are useful for human readability.

Use immutable release processes so a version tag is not unexpectedly overwritten.

---

## 23. Image Digest

An image digest identifies exact image content:

```text
sha256:...
```

Production deployment should record the digest.

---

## 24. Tag vs Digest

Tag:

```text
app:1.4.2
```

means:

```text
Reference
```

Digest:

```text
app@sha256:...
```

means:

```text
Exact content
```

Use digests for strong deployment traceability.

---

## 25. Build Once, Promote

Recommended:

```text
Build
 ↓
Scan
 ↓
Push
 ↓
Promote same image
```

Avoid:

```text
Build for Dev
 ↓
Rebuild for Staging
 ↓
Rebuild for Production
```

The second model can produce different artifacts.

---

## 26. Docker Build in GitLab CI

Conceptual:

```yaml
build_image:
  script:
    - docker build -t "$IMAGE_NAME:$CI_COMMIT_SHA" .
```

The exact build architecture depends on the Runner executor and security model.

---

## 27. Docker-in-Docker

DinD can provide a Docker daemon for CI builds.

Concept:

```text
GitLab Job
   ↓
Docker client
   ↓
Docker daemon
   ↓
Image
```

Evaluate security and performance before using it.

---

## 28. Docker Socket Pattern

Another approach mounts:

```text
/var/run/docker.sock
```

into the CI container.

This gives access to the host Docker daemon.

Treat this as a privileged security boundary.

---

## 29. DinD vs Socket

### DinD

```text
Job
 ↓
Dedicated Docker daemon
```

### Socket

```text
Job
 ↓
Host Docker daemon
```

Neither should be selected casually.

---

## 30. Rootless Build Strategy

Rootless builders can reduce host privilege requirements.

Possible technologies include:

```text
BuildKit
Kaniko
other approved OCI builders
```

Select based on compatibility, caching, and security requirements.

---

## 31. Docker BuildKit

BuildKit can improve:

- build performance
- caching
- parallelization
- build features

It is commonly used in modern Docker environments.

---

## 32. Docker Build Cache

Build caching can reuse unchanged layers.

Example:

```text
Dependency install
 ↓
Application source
```

If only source changes, dependency layers may be reused.

---

## 33. Docker Layer Ordering

Bad:

```dockerfile
COPY . .
RUN install dependencies
```

Every source change may invalidate dependency installation.

Better:

```dockerfile
COPY dependency-files .
RUN install dependencies
COPY source .
```

This improves cache reuse.

---

## 34. Cache and Security

Do not cache:

```text
secrets
private credentials
sensitive runtime data
```

Docker build cache should not become a credential storage mechanism.

---

## 35. Build Arguments

Example:

```dockerfile
ARG APP_VERSION
```

Build arguments are useful for non-secret metadata.

Do not pass secrets as ordinary `ARG` values because they may appear in build metadata/history depending on the build process.

---

## 36. Environment Variables in Dockerfile

Example:

```dockerfile
ENV APP_ENV=production
```

Use `ENV` for runtime configuration that is not secret.

Do not bake production passwords into the image.

---

## 37. Secret-Aware Docker Builds

For build-time credentials, use build mechanisms designed for secrets where supported.

Goal:

```text
Credential available during build
 ↓
Not persisted in final image
```

---

## 38. Image Labels

Useful metadata:

```text
org.opencontainers.image.revision
org.opencontainers.image.version
org.opencontainers.image.source
```

Labels can improve traceability.

---

## 39. OCI Metadata

OCI image metadata can connect:

```text
Image
 ↓
Repository
 ↓
Commit
 ↓
Version
```

This helps during production investigations.

---

## 40. Container Registry Authentication

Registry operations generally require:

```text
Authentication
+
Authorization
```

Examples:

```text
GitLab → GitLab Registry credentials
GitLab → AWS OIDC → ECR
```

Use short-lived/scoped credentials where possible.

---

## 41. GitLab Registry Authentication

A CI job may authenticate to the GitLab Container Registry using supported CI credentials.

Avoid storing personal Docker credentials unnecessarily.

---

## 42. ECR Authentication

Typical flow:

```text
AWS identity
 ↓
ECR authentication
 ↓
Docker login
 ↓
docker push
```

A common validation command is:

```bash
aws sts get-caller-identity
```

before performing sensitive AWS operations.

---

## 43. ECR Login

Conceptually:

```bash
aws ecr get-login-password --region "$AWS_REGION" |
docker login --username AWS --password-stdin "$ECR_REGISTRY"
```

Use an IAM role with only the required ECR permissions.

---

## 44. ECR Push Flow

```text
Docker build
 ↓
Docker tag
 ↓
ECR login
 ↓
Docker push
 ↓
Digest
```

Capture the final image identity for deployment.

---

## 45. ECR Repository

Example:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/user-service
```

Keep repository naming consistent.

---

## 46. ECR Repository Strategy

Possible structures:

```text
roboshop/user
roboshop/cart
roboshop/order
```

or:

```text
user-service
cart-service
order-service
```

Choose a naming model that works across teams and environments.

---

## 47. One Registry Per Environment?

Options include:

```text
Same registry
 + separate repositories/tags

or

Separate AWS accounts/registries
```

Separate AWS accounts provide stronger isolation.

Use architecture appropriate to organizational security requirements.

---

## 48. ECR Image Scanning

ECR supports image vulnerability scanning capabilities.

Combine registry scanning with CI scanning when policy requires defense in depth.

---

## 49. Trivy Before Push

A common CI flow:

```text
Build image
 ↓
Trivy scan
 ↓
Policy decision
 ↓
Push ECR
```

This prevents known-bad images from reaching the production registry when the policy is configured as a blocking gate.

---

## 50. Trivy After Push

Another option:

```text
Build
 ↓
Push
 ↓
Registry scan
 ↓
Promotion decision
```

This can provide registry-side visibility.

Use both approaches if required by your security model.

---

## 51. Vulnerability Severity

Common categories:

```text
LOW
MEDIUM
HIGH
CRITICAL
```

Define organizational thresholds.

Do not automatically fail every image because it contains a low-severity vulnerability if the policy allows exceptions.

---

## 52. Vulnerability Exceptions

Sometimes a vulnerability cannot immediately be fixed.

Use a controlled exception:

```text
CVE
 ↓
Risk assessment
 ↓
Business justification
 ↓
Expiration date
 ↓
Approval
```

Avoid permanent blanket exclusions.

---

## 53. Image Scanning Pipeline

Example:

```text
Build
 ↓
Trivy
 ↓
SonarQube
 ↓
Veracode
 ↓
Policy
 ↓
Push
```

Exact ordering depends on tool capabilities.

---

## 54. Container Image Signing

A stronger supply-chain model:

```text
Build
 ↓
Scan
 ↓
Sign
 ↓
Registry
 ↓
Verify
 ↓
Deploy
```

Signing proves authorization/integrity when verification is enforced.

---

## 55. Digest Verification

Deployment should verify the exact digest:

```text
GitOps
 ↓
Image digest
 ↓
Registry
 ↓
Exact image
```

This prevents accidental tag movement from changing the deployed content.

---

## 56. Kubernetes Pulling from ECR

EKS workloads need permission to pull images from ECR.

AWS-managed node/workload identity mechanisms should be configured appropriately.

Do not embed ECR passwords in Kubernetes manifests.

---

## 57. EKS Image Pull Architecture

```text
EKS Pod
 ↓
AWS identity
 ↓
ECR authorization
 ↓
Image pull
```

The exact identity path depends on the EKS configuration.

---

## 58. ECR Cross-Account Pull

If EKS and ECR are in different AWS accounts:

```text
EKS Account
     ↓
AWS IAM
     ↓
ECR Account
     ↓
Repository policy
```

Configure both identity and resource-side permissions appropriately.

---

## 59. ECR Lifecycle Policies

Images accumulate:

```text
commit-1
commit-2
commit-3
...
```

Use lifecycle policies to remove obsolete images while retaining required releases.

---

## 60. Image Retention

Keep enough images for:

```text
Rollback
Incident investigation
Release history
Compliance
```

Delete images according to policy.

---

## 61. Never Delete Active Production Image

Before cleanup, determine:

```text
Currently deployed digest
Protected releases
Rollback versions
```

Lifecycle policies should be tested.

---

## 62. Image Promotion

A strong model:

```text
Build
 ↓
Scan
 ↓
Push
 ↓
Promote digest
```

Promotion should not rebuild the image.

---

## 63. Registry Promotion Strategies

Options:

```text
Same repository + immutable digest
```

or:

```text
Separate repositories
```

or:

```text
Separate AWS accounts
```

Choose based on trust boundaries.

---

## 64. Tag Promotion

Example:

```text
app:build-abc123
```

then:

```text
app:staging
```

then:

```text
app:production
```

If tags are mutable, deployment should still record the immutable digest.

---

## 65. Digest Promotion

Better:

```text
digest sha256:ABC
 ↓
staging
 ↓
production
```

The exact image content remains unchanged.

---

## 66. GitOps Image Update

GitLab CI can update:

```yaml
image:
  repository: 123.dkr.ecr.ap-south-1.amazonaws.com/user-service
  digest: sha256:...
```

Then:

```text
Git commit
 ↓
ArgoCD
 ↓
EKS
```

---

## 67. CI Should Not Need Cluster Admin

For GitOps:

```text
CI
 ↓
Git repository
```

rather than:

```text
CI
 ↓
kubectl cluster-admin
```

This is a major security boundary.

---

## 68. Container Image and ArgoCD

Flow:

```text
GitLab CI
 ↓
Build image
 ↓
Scan
 ↓
Push ECR
 ↓
Update GitOps
 ↓
ArgoCD detects change
 ↓
EKS
```

This fits a production GitOps architecture.

---

## 69. ImagePullBackOff

If EKS cannot pull the image:

```text
kubectl describe pod <pod>
```

Check:

```text
image name
registry
tag/digest
IAM
network
architecture
```

---

## 70. ECR Image Not Found

Potential causes:

- wrong repository
- wrong AWS region
- wrong account
- incorrect tag
- digest mismatch
- image never pushed

Validate the exact image reference.

---

## 71. ECR AccessDenied

Check:

```text
AWS identity
 ↓
IAM permissions
 ↓
ECR repository policy
 ↓
Account
 ↓
Region
```

First identify the actual identity:

```bash
aws sts get-caller-identity
```

---

## 72. Docker Build Failure

Common causes:

```text
Dockerfile syntax
Base image unavailable
Network
Dependency failure
Build context
Permissions
Disk
CPU/memory
```

Read the first meaningful error rather than only the final line.

---

## 73. Docker Push Failure

Check:

```text
Registry login
 ↓
Repository
 ↓
Permissions
 ↓
Network
 ↓
Image tag
```

A successful local build does not prove push authorization.

---

## 74. Docker Login Failure

Possible causes:

- expired credentials
- wrong registry
- wrong AWS region
- OIDC failure
- network/proxy
- incorrect command

Do not store passwords in shell history.

---

## 75. Registry Network Failure

Check:

```text
DNS
TLS
Proxy
Firewall
NAT
VPC endpoints where applicable
```

For private EKS/AWS architectures, validate the required AWS service network path.

---

## 76. ECR in Private Subnets

If Runner jobs run in private subnets:

```text
Private Runner
 ↓
NAT Gateway / required VPC endpoints
 ↓
AWS APIs/ECR
```

Ensure required connectivity exists.

---

## 77. ECR Cost Considerations

Costs can come from:

```text
Image storage
Data transfer
NAT Gateway
CI compute
Repeated pulls
```

Optimize image size and network architecture.

---

## 78. Image Pull Optimization

Large images slow:

```text
Pod startup
Deployment
Autoscaling
Rollback
```

Use:

- smaller runtime images
- local node caching where appropriate
- efficient layers
- reasonable image retention

---

## 79. Image Layer Strategy

Put stable layers first:

```text
Base
 ↓
OS/runtime dependencies
 ↓
Application dependencies
 ↓
Source
```

This improves cache reuse.

---

## 80. Dependency Installation Layer

For Maven:

```dockerfile
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package
```

The exact optimization depends on project structure.

---

## 81. Node.js Image Build

Example pattern:

```dockerfile
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build
```

This allows dependency installation to be cached when only application source changes.

---

## 82. Python Image Build

Example:

```dockerfile
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
```

Pin dependencies appropriately.

---

## 83. Non-Root Container

Prefer running application processes as non-root where practical.

Example concept:

```dockerfile
USER 10001
```

Validate filesystem permissions before deployment.

---

## 84. Root Container Risk

If an application is compromised while running as root:

```text
Application
 ↓
Root inside container
 ↓
Potentially larger impact
```

Non-root reduces privilege.

It does not eliminate container escape risk.

---

## 85. Read-Only Filesystem

For suitable applications:

```text
readOnlyRootFilesystem
```

can reduce write capability.

Applications requiring temporary files should use controlled writable volumes.

---

## 86. Linux Capabilities

Containers may receive Linux capabilities.

Remove unnecessary capabilities where supported.

Least privilege should apply at the container level too.

---

## 87. Health Checks

Docker can define a health check, but Kubernetes readiness/liveness probes are usually the primary production mechanism for EKS workloads.

Do not assume a Docker health check replaces Kubernetes probes.

---

## 88. Environment Configuration

Do not bake environment-specific values into images.

Better:

```text
Same image
 ↓
Dev configuration
```

and:

```text
Same image
 ↓
Production configuration
```

This supports build-once/promote.

---

## 89. Twelve-Factor Configuration

Application configuration should be externalized.

Example:

```text
IMAGE
+
ENVIRONMENT CONFIG
+
SECRETS
```

rather than:

```text
different image per environment
```

---

## 90. Image Reproducibility

Control:

```text
Base image
Dependency versions
Build tool
Source commit
Build arguments
```

A reproducible image should be traceable to known inputs.

---

## 91. Dockerfile Security

Review:

```text
Base image
RUN commands
ADD/COPY
USER
ENV
ARG
Secrets
Network downloads
```

Avoid arbitrary scripts downloaded from the internet during builds.

---

## 92. Curl Pipe Shell Risk

Avoid patterns such as:

```bash
curl https://example/script.sh | sh
```

unless the source, integrity, and organizational policy explicitly justify it.

Prefer:

```text
Verified package
Pinned artifact
Checksum/signature
```

---

## 93. Package Manager Security

Use trusted repositories.

For Linux packages:

```text
Official repositories
Approved internal mirror
Pinned versions where appropriate
```

For application packages:

```text
Approved registry
Lock file
Dependency scanning
```

---

## 94. Image SBOM

Generate an SBOM for release images where required.

Flow:

```text
Image
 ↓
SBOM
 ↓
Vulnerability analysis
 ↓
Release evidence
```

---

## 95. Container Metadata

Useful metadata:

```text
version
commit
source repository
build timestamp
vendor
license
```

Use OCI-compatible labels where supported.

---

## 96. Container Registry Permissions

Separate:

```text
push
pull
delete
admin
```

permissions.

A CI build role may need:

```text
push
```

but should not automatically have:

```text
delete all repositories
```

---

## 97. ECR IAM Least Privilege

For push jobs, restrict permissions to required ECR APIs and repository resources.

Avoid:

```text
Resource: *
```

when resource-level restriction is supported and practical.

---

## 98. ECR Repository Policies

Cross-account access may require repository policies.

Review both:

```text
IAM identity policy
+
ECR repository policy
```

Access can fail if either side is incorrect.

---

## 99. Registry Credential Rotation

For static registry credentials:

```text
Create replacement
 ↓
Update consumers
 ↓
Verify
 ↓
Revoke old
```

Prefer workload identity where supported.

---

## 100. Container Registry Audit

Monitor:

```text
Image push
Image delete
Repository changes
Authentication
Unexpected tags
Unexpected users/roles
```

This helps detect supply-chain incidents.

---

## 101. Image Immutability

If the registry supports tag immutability, consider enabling it for release repositories.

Goal:

```text
1.4.2
 ↓
cannot silently become another image
```

This protects release traceability.

---

## 102. Mutable Development Tags

Development may use:

```text
dev
latest
```

but production should rely on immutable identities.

Separate development convenience from production controls.

---

## 103. Registry Namespace Strategy

Example:

```text
company/
 ├── dev/
 ├── staging/
 └── production/
```

or separate accounts.

The naming structure should make trust boundaries obvious.

---

## 104. Image Promotion Between Registries

For stronger isolation:

```text
Dev ECR
 ↓
Approved image
 ↓
Prod ECR
 ↓
Production EKS
```

Promotion copies the same content rather than rebuilding.

---

## 105. Cross-Account ECR Promotion

Example:

```text
CI Account
   ↓
Build
   ↓
Security
   ↓
Prod Account ECR
   ↓
EKS
```

Use narrowly scoped cross-account permissions.

---

## 106. Image Replication

Registry replication can support:

```text
Region A
 ↓
Region B
```

Useful for:

- disaster recovery
- multi-region deployments
- reduced pull latency

Validate consistency and security policies.

---

## 107. Multi-Architecture Images

A service may need:

```text
linux/amd64
linux/arm64
```

Build architecture:

```text
docker buildx build
```

where supported.

---

## 108. Multi-Arch Manifest

A multi-platform image can point to architecture-specific manifests:

```text
app:1.0
 ├── amd64
 └── arm64
```

The runtime pulls the correct architecture.

---

## 109. Architecture Troubleshooting

Error examples:

```text
exec format error
```

Possible cause:

```text
ARM image
 ↓
AMD64 node
```

Check:

```bash
uname -m
```

on the relevant environment and inspect image platforms.

---

## 110. Buildx

Buildx supports advanced Docker builds including multi-platform builds.

Example concept:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  ...
```

Use only when the Runner/build environment supports it.

---

## 111. Registry Cache for BuildKit

BuildKit can use registry-backed cache.

Concept:

```text
Runner
 ↓
BuildKit
 ↓
Registry cache
```

This can be useful with ephemeral Runners.

Secure cache repositories separately from release repositories.

---

## 112. Image Build Performance

Optimize:

```text
Dockerfile layers
Build context
Dependency caching
Base image size
Runner CPU
Network
Registry location
```

Measure before and after changes.

---

## 113. Docker Build Time Troubleshooting

Break down:

```text
Context upload
Base pull
Dependency download
Compilation
Docker layers
Push
```

This identifies the actual bottleneck.

---

## 114. Slow ECR Push

Possible causes:

```text
Large image
Slow network
NAT bottleneck
Many layers
Runner location
Registry region
```

Reduce image size and validate network architecture.

---

## 115. Slow ECR Pull

Possible causes:

```text
Large image
Cold node
Network path
Cross-region pull
Registry latency
```

Use appropriate image size and deployment architecture.

---

## 116. ECR Image Lifecycle Incident

If old images disappear unexpectedly:

```text
Check lifecycle policy
 ↓
Check image tags
 ↓
Check release retention
 ↓
Check deployed digest
```

Ensure lifecycle rules exclude protected releases where required.

---

## 117. Wrong Docker Tag

Example:

```text
IMAGE_TAG=$CI_COMMIT_SHORT_SHA
```

but GitOps expects:

```text
$CI_COMMIT_SHA
```

The deployment may reference a nonexistent image.

Standardize image identity across pipeline stages.

---

## 118. Image Tag Collision

If multiple pipelines use:

```text
latest
```

they can overwrite each other's deployment target.

Use commit or release identifiers.

---

## 119. Container Registry and GitLab Variables

Typical non-secret variables:

```text
AWS_REGION
ECR_REGISTRY
ECR_REPOSITORY
IMAGE_TAG
IMAGE_DIGEST
```

Sensitive credentials should use protected identity mechanisms.

---

## 120. Image Build Job Example

Conceptual:

```yaml
build_image:
  stage: build
  script:
    - docker build -t "$ECR_REGISTRY/$ECR_REPOSITORY:$CI_COMMIT_SHA" .
```

Use a secure executor/build architecture.

---

## 121. ECR Push Job Example

Conceptual:

```yaml
push_image:
  stage: publish
  script:
    - aws sts get-caller-identity
    - aws ecr get-login-password --region "$AWS_REGION" |
      docker login --username AWS --password-stdin "$ECR_REGISTRY"
    - docker push "$IMAGE"
```

Production permissions should be minimal.

---

## 122. Image Scan Job

Conceptual:

```yaml
scan_image:
  stage: security
  script:
    - trivy image "$IMAGE"
```

Configure the actual vulnerability threshold according to policy.

---

## 123. GitLab CI Container Flow

```text
build
 ↓
scan
 ↓
push
 ↓
digest
 ↓
gitops-update
```

Keep deployment independent from rebuilding.

---

## 124. Image Digest Extraction

After push, obtain the exact digest from the registry or image metadata.

Store it as controlled pipeline output.

Do not guess the digest from the tag.

---

## 125. Digest as GitLab Artifact

A small metadata file can store:

```text
IMAGE_DIGEST=sha256:...
```

Downstream jobs can consume this exact identity.

This is safer than recomputing or retagging.

---

## 126. Artifact → Registry → GitOps

Strong flow:

```text
Build
 ↓
Image
 ↓
Scan
 ↓
Push ECR
 ↓
Digest artifact
 ↓
GitOps update
 ↓
ArgoCD
```

This connects CI evidence with production deployment.

---

## 127. GitOps Update Safety

Before updating GitOps:

```text
Verify repository
Verify branch
Verify service
Verify image digest
Verify environment
```

A wrong variable can deploy the correct image to the wrong environment.

---

## 128. Production Deployment Validation

After ArgoCD sync:

```text
kubectl get pods
kubectl describe pod
kubectl get deployment
```

Check:

```text
image digest
readiness
replicas
events
```

---

## 129. Rollback

Rollback can be:

```text
GitOps commit rollback
```

or:

```text
known-good image digest
```

Prefer deterministic immutable references.

---

## 130. Container Security Checklist

```text
[ ] Trusted base image
[ ] Pinned versions
[ ] Multi-stage build
[ ] Small runtime image
[ ] Non-root user
[ ] No secrets in image
[ ] `.dockerignore`
[ ] Dependency lock files
[ ] Vulnerability scanning
[ ] SBOM where required
[ ] Image signing where required
[ ] Immutable production identity
[ ] Registry access control
[ ] Lifecycle policy
[ ] Audit logging
[ ] ECR least-privilege IAM
```

---

## 131. Production Architecture

```text
Developer
    │
    ▼
 GitLab
    │
    ▼
 CI Runner
    │
    ├── Test
    ├── Build
    └── Security
          │
          ▼
       Docker Image
          │
          ▼
         ECR
          │
          ▼
      Image Digest
          │
          ▼
     GitOps Repository
          │
          ▼
        ArgoCD
          │
          ▼
         EKS
          │
          ▼
     Kubernetes Pods
```

---

## 132. Senior Interview — Why Use ECR?

> ECR integrates naturally with AWS and EKS, supports IAM-based access, image scanning capabilities, lifecycle management, and AWS-native registry operations. For an AWS production platform, it is a strong choice for storing deployable container images.

---

## 133. Senior Interview — Tag vs Digest

> A tag is a mutable reference while a digest identifies exact image content. For production I prefer immutable digest-based deployment and retain tags for human-friendly versioning and traceability.

---

## 134. Senior Interview — How Do You Secure Docker Builds?

> I use trusted and controlled base images, multi-stage builds, non-root containers, dependency locking, secret-aware build mechanisms, vulnerability scanning, SBOM/signing where required, restricted Runner privileges, and controlled registry access.

---

## 135. Senior Interview — Why Not Mount Docker Socket?

> Mounting the Docker socket can give the job extensive control over the host Docker daemon and therefore increases the security blast radius. I use a safer build architecture when possible and grant only the capabilities required.

---

## 136. Senior Interview — How Do You Build Images in EKS?

> I would choose a Kubernetes-compatible build strategy such as BuildKit, Kaniko, or another approved builder depending on security and performance requirements. I avoid giving arbitrary CI jobs unnecessary host-level Docker privileges.

---

## 137. Senior Interview — How Do You Handle ECR Credentials?

> I prefer short-lived AWS credentials obtained through GitLab OIDC and an IAM role. The role is scoped to the required ECR repositories and operations.

---

## 138. Senior Interview — How Do You Prevent Production From Using a Different Image?

> I build once, scan once, push the image, capture the immutable digest, and update GitOps with that digest. Staging and production therefore reference the same image content.

---

## 139. Senior Interview — How Do You Troubleshoot ImagePullBackOff?

> I inspect the Pod events and verify the exact image reference, registry, tag/digest, AWS identity, ECR permissions, network connectivity, and node architecture.

---

## 140. Senior Interview — How Do You Reduce Image Size?

> I use multi-stage builds, runtime-specific base images, `.dockerignore`, efficient Dockerfile layer ordering, and removal of unnecessary build dependencies from the final image.

---

## 141. Senior Interview — How Do You Handle Vulnerabilities That Cannot Be Fixed Immediately?

> I assess exploitability and business risk, document an approved temporary exception with an owner and expiration, apply compensating controls, and ensure it does not become a permanent blanket exception.

---

## 142. Senior Interview — How Do You Handle Image Rollback?

> I deploy a known-good immutable image digest or revert the GitOps revision. I avoid rebuilding an old source commit during an incident because the resulting artifact may differ.

---

## 143. Senior Interview — How Do You Handle Multi-Architecture Images?

> I build separate architecture variants using a multi-platform build system and publish a multi-architecture manifest. I verify that the target EKS nodes support the requested architecture.

---

## 144. Senior Interview — What Is Your Production Container Flow?

> GitLab CI runs tests and security checks, builds the container image, scans it with tools such as Trivy, pushes the approved image to ECR, records the immutable digest, updates the GitOps repository, and ArgoCD reconciles the desired state into EKS.

---

## 145. Final Production Checklist

```text
[ ] Dockerfile reviewed
[ ] Multi-stage build
[ ] Small runtime image
[ ] Non-root user
[ ] No secrets baked into image
[ ] `.dockerignore`
[ ] Dependencies pinned
[ ] Base image controlled
[ ] Image scanned
[ ] SBOM generated where required
[ ] Image signed where required
[ ] ECR authentication secured
[ ] IAM least privilege
[ ] Repository policy reviewed
[ ] Lifecycle policy configured
[ ] Release tags controlled
[ ] Digest captured
[ ] GitOps uses immutable identity
[ ] EKS pull access validated
[ ] Rollback image retained
[ ] Registry audit enabled
[ ] CI build architecture secured
```

---

## 146. Final Mental Model

```text
                  GitLab
                     │
                     ▼
                CI Pipeline
                     │
             ┌───────┴────────┐
             ▼                ▼
           Build            Security
             │                │
             └───────┬────────┘
                     ▼
                Docker Image
                     │
                     ▼
                    ECR
                     │
              Immutable Digest
                     │
                     ▼
               GitOps Repo
                     │
                     ▼
                   ArgoCD
                     │
                     ▼
                    EKS
                     │
                     ▼
                 Containers
```

> **The container registry is the durable source for deployable images. GitLab CI should build and validate the image once, ECR should store the immutable artifact, GitOps should reference the exact digest, and ArgoCD should reconcile that desired state into EKS.**

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md
├── 02-GitLab-Repository-and-Git-Workflow.md
├── 03-GitLab-Branches-and-Merge-Requests.md
├── 04-GitLab-CI-CD-Fundamentals.md
├── 05-GitLab-CI-CD-Configuration.md
├── 06-GitLab-Runners.md
├── 07-GitLab-Variables-Secrets-and-Environments.md
├── 08-GitLab-Artifacts-Caching-and-Dependencies.md
├── 09-GitLab-Docker-and-Container-Registry.md ✓
├── 10-GitLab-AWS-Integration.md
├── 11-GitLab-Kubernetes-and-EKS.md
├── 12-GitLab-Terraform-and-IaC.md
├── 13-GitLab-ArgoCD-and-GitOps.md
├── 14-GitLab-DevSecOps.md
├── 15-GitLab-Security.md
├── 16-GitLab-Advanced-CI-CD.md
├── 17-GitLab-Multi-Environment-Deployments.md
├── 18-GitLab-Advanced-Pipelines.md
├── 19-GitLab-API-and-Automation.md
├── 20-GitLab-Monitoring-and-Observability.md
├── 21-GitLab-Production-Troubleshooting.md
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md
```

**Next: `10-GitLab-AWS-Integration.md`**
