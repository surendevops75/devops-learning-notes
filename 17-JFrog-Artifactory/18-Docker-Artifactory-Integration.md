# 17-JFrog-Artifactory
# 18-Docker-Artifactory-Integration

## 1. Purpose

This file provides a production-oriented guide to integrating Docker/OCI workloads with JFrog Artifactory.

The focus is the complete container artifact lifecycle:

```text
Source
 ↓
Dockerfile
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Tag
 ↓
Push
 ↓
Build Info
 ↓
Promote
 ↓
Deploy
 ↓
Run
 ↓
Monitor
 ↓
Retain / Retire
```

The guide covers:

- Docker and OCI fundamentals
- Artifactory Docker repositories
- Local, remote and virtual Docker repositories
- Docker Registry API concepts
- Authentication
- Access tokens
- Docker login
- Image naming
- Tags and digests
- Immutable releases
- Docker builds
- Multi-stage builds
- BuildKit concepts
- Docker Buildx
- Multi-architecture images
- Docker manifest/index concepts
- Base images
- Remote caching
- Dependency control
- Image scanning
- Build Info
- JFrog CLI
- Jenkins
- GitHub Actions
- GitLab CI
- Kubernetes
- EKS
- Helm
- GitOps
- Image promotion
- Image retention
- Garbage collection concepts
- Production security
- Private networking
- Troubleshooting
- Real production scenarios
- Interview preparation

---

# PART I — DOCKER + ARTIFACTORY FUNDAMENTALS

## 2. What Is Docker?

Docker packages an application and its runtime dependencies into a container image.

Conceptually:

```text
Application
+
Dependencies
+
Runtime
+
Configuration
=
Container Image
```

---

## 3. What Is a Container Image?

A container image is an immutable collection of filesystem layers plus metadata used to create a container.

Example:

```text
payment-service:4.2.1
```

---

## 4. What Is a Container Registry?

A container registry stores and distributes container images.

Artifactory can act as an enterprise Docker/OCI registry.

Architecture:

```text
Developer / CI
      |
      v
Artifactory Docker Repository
      |
      v
Kubernetes
```

---

## 5. Why Use Artifactory for Docker?

Artifactory can provide:

```text
centralized image storage
RBAC
remote repositories
virtual repositories
artifact metadata
Build Info
security controls
retention
auditability
enterprise availability
```

---

# PART II — DOCKER REPOSITORY TYPES

## 6. Docker Local Repository

Used to store organization-owned images.

Example:

```text
docker-local
```

Flow:

```text
CI
 ↓
docker-local
```

---

## 7. Docker Remote Repository

Used to proxy/cache images from external registries.

Example:

```text
docker-remote
```

Concept:

```text
CI
 ↓
docker-virtual
 ↓
docker-remote
 ↓
External Registry
```

---

## 8. Docker Virtual Repository

Provides a single endpoint over multiple repositories.

Example:

```text
docker-virtual
 |
 +--> docker-local
 |
 +--> docker-remote
```

This is useful for dependency/base-image resolution.

---

## 9. Production Repository Model

A common enterprise pattern:

```text
docker-virtual
     |
     +--> internal images
     |
     +--> approved external images
```

Publication:

```text
CI
 ↓
docker-local
```

Consumption:

```text
Kubernetes
 ↓
docker-virtual
```

The exact design should follow organizational governance.

---

# PART III — DOCKER REGISTRY NAMING

## 10. Image Reference

General form:

```text
REGISTRY/REPOSITORY/IMAGE:TAG
```

Example:

```text
artifactory.example.com/docker-local/payment-service:4.2.1
```

---

## 11. Registry Host

```text
artifactory.example.com
```

---

## 12. Repository

```text
docker-local
```

---

## 13. Image Name

```text
payment-service
```

---

## 14. Tag

```text
4.2.1
```

---

## 15. Digest

Example:

```text
sha256:abcdef...
```

A digest identifies content.

---

# PART IV — TAG VS DIGEST

## 16. Docker Tag

Example:

```text
payment-service:4.2.1
```

A tag is a human-friendly reference.

---

## 17. Docker Digest

Example:

```text
payment-service@sha256:abcdef...
```

A digest identifies the exact image content.

---

## 18. Production Recommendation

Prefer:

```text
immutable version
+
digest verification
```

Avoid using:

```text
latest
```

as a production release identity.

---

## 19. Why latest Is Dangerous

If:

```text
latest → image A
```

today and:

```text
latest → image B
```

tomorrow,

the deployment reference has changed without changing the visible tag.

---

# PART V — AUTHENTICATION

## 20. Docker Login

A client authenticates to the registry before push/pull.

Example:

```bash
docker login artifactory.example.com
```

Use secure credentials.

---

## 21. Service Identity

CI should use:

```text
svc-ci-payment
```

rather than a human account.

---

## 22. Least Privilege

Example:

```text
READ:
docker-virtual

DEPLOY:
payment-docker-local

DELETE:
NO

ADMIN:
NO
```

---

## 23. Runtime Identity

Kubernetes runtime access should use a separate identity from CI whenever practical.

```text
GitHub Actions
    |
    +--> CI credential

Kubernetes
    |
    +--> Runtime image-pull credential
```

---

# PART VI — DOCKER CREDENTIALS

## 24. Credential Storage

Never store:

```text
docker login password
```

in Git.

Use:

```text
CI secret store
Jenkins Credentials
GitHub Secrets
GitLab protected variables
Kubernetes secret
external secret manager
```

as appropriate.

---

## 25. Credential Rotation

Example:

```text
Create new credential
 ↓
Update CI/runtime
 ↓
Test
 ↓
Revoke old credential
```

---

## 26. Credential Exposure

If a token appears in logs:

```text
Assume compromised
 ↓
Rotate/revoke
 ↓
Audit
 ↓
Fix logging
```

---

# PART VII — DOCKERFILE

## 27. Basic Dockerfile

```dockerfile
FROM eclipse-temurin:21-jre

WORKDIR /app

COPY target/payment-service.jar app.jar

USER 10001

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Production improvements should include trusted base images, minimal packages, non-root execution and reproducible dependency handling.

---

## 28. Base Image

The base image is part of the software supply chain.

Example:

```text
eclipse-temurin:21-jre
```

Do not blindly use arbitrary public images.

---

## 29. Base Image Governance

Use:

```text
approved images
approved registries
version policy
vulnerability scanning
update process
```

---

# PART VIII — MULTI-STAGE BUILDS

## 30. Why Multi-Stage Builds?

Separate build dependencies from runtime dependencies.

Example:

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build

WORKDIR /src

COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

FROM eclipse-temurin:21-jre

WORKDIR /app

COPY --from=build /src/target/payment-service.jar app.jar

USER 10001

ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 31. Benefits

```text
smaller runtime image
fewer packages
smaller attack surface
faster deployment
less unnecessary tooling
```

---

# PART IX — BUILD CONTEXT

## 32. Docker Build Context

The build context contains files available to the Docker build.

Avoid copying:

```text
.git
credentials
local configuration
private keys
node_modules
large temporary files
```

---

## 33. .dockerignore

Example:

```text
.git
.gitignore
node_modules
target
*.log
.env
```

---

# PART X — BUILD PROCESS

## 34. Build

Example:

```bash
docker build \
  -t artifactory.example.com/docker-local/payment-service:4.2.1 \
  .
```

---

## 35. Build Arguments

Example:

```bash
docker build \
  --build-arg APP_VERSION=4.2.1 \
  -t payment-service:4.2.1 .
```

Do not pass secrets as ordinary build arguments because they can become exposed in build metadata/history depending on usage.

---

## 36. BuildKit Secrets

For sensitive build-time data, use BuildKit-supported secret mechanisms rather than embedding secrets in Dockerfile layers.

Concept:

```text
CI secret
 ↓
BuildKit secret mount
 ↓
Build step
```

---

# PART XI — DOCKER BUILD CACHE

## 37. Why Cache?

Caching can improve:

```text
build speed
CI efficiency
developer productivity
```

---

## 38. Cache Risk

A stale or incorrectly configured cache can cause unexpected build behavior.

Use reproducible build practices.

---

## 39. Artifactory as Dependency Cache

Artifactory remote repositories can cache approved external images/packages according to configuration.

Concept:

```text
Docker Build
 ↓
Artifactory Remote
 ↓
External Registry
```

After caching:

```text
Docker Build
 ↓
Artifactory Cache
```

---

# PART XII — PUSH

## 40. Docker Push

```bash
docker push \
  artifactory.example.com/docker-local/payment-service:4.2.1
```

---

## 41. Push Permissions

The identity needs:

```text
DEPLOY
```

permission for the target repository/path.

---

## 42. Push Failure

If authentication succeeds but push fails:

```text
401
→ authentication

403
→ authorization
```

Investigate accordingly.

---

# PART XIII — IMAGE SCANNING

## 43. Why Scan Images?

Images can contain:

```text
OS vulnerabilities
application vulnerabilities
malicious packages
secrets
unsafe configuration
```

---

## 44. Scan Pipeline

Recommended:

```text
Build
 ↓
Scan
 ↓
Quality Gate
 ↓
Push/Promote
```

or:

```text
Build
 ↓
Push to controlled quarantine/candidate location
 ↓
Scan
 ↓
Promote
```

depending on the enterprise workflow.

---

## 45. Critical Vulnerability

Example:

```text
Critical CVE
 ↓
Block production promotion
```

Actual blocking criteria should follow security policy.

---

# PART XIV — IMAGE CONTENT

## 46. Image Layers

Conceptually:

```text
Layer 1
Layer 2
Layer 3
Layer 4
```

Layers can be shared between images.

---

## 47. Why Layers Matter

They affect:

```text
storage
download time
build cache
security scanning
```

---

## 48. Minimize Layers Carefully

Use sensible Dockerfile ordering.

Example:

```dockerfile
COPY dependency-files .
RUN install-dependencies

COPY application .
RUN build
```

This can improve cache reuse when dependency files change less frequently.

---

# PART XV — MULTI-ARCH IMAGES

## 49. Multi-Architecture

A release may support:

```text
linux/amd64
linux/arm64
```

---

## 50. Buildx

Example:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t artifactory.example.com/docker-local/payment-service:4.2.1 \
  --push .
```

The registry stores the multi-platform manifest/index and referenced platform images.

---

## 51. Why Multi-Arch Matters

Different environments may use:

```text
x86_64
ARM64
```

Example:

```text
AWS Graviton
```

---

## 52. Production Verification

Verify:

```text
supported platforms
digest
manifest/index
base image support
runtime compatibility
```

---

# PART XVI — IMAGE PROMOTION

## 53. Candidate Image

```text
payment-service:4.2.1
```

After validation:

```text
Promote
```

---

## 54. Promotion Principle

Do not rebuild.

```text
Build
 ↓
Scan
 ↓
Publish
 ↓
Promote
```

---

## 55. Promotion Integrity

Preserve:

```text
same digest
same content
same release identity
```

---

# PART XVII — BUILD INFO

## 56. Build Info

Build Info connects:

```text
Git commit
CI run
Docker build
dependencies
image
```

---

## 57. Example

```text
Git:
abc1234

Pipeline:
721

Image:
payment-service:4.2.1

Digest:
sha256:...
```

---

## 58. Incident Traceability

During an incident:

```text
Production image
 ↓
Digest
 ↓
Artifactory artifact
 ↓
Build Info
 ↓
CI build
 ↓
Git commit
```

---

# PART XVIII — JFROG CLI

## 59. CLI

JFrog CLI can automate Docker/Artifactory operations.

Conceptually:

```text
CI
 ↓
JFrog CLI
 ↓
Artifactory
```

---

## 60. Upload/Publish

Use the CLI where it provides useful build-info and repository automation.

Avoid exposing tokens in shell history or logs.

---

# PART XIX — JENKINS INTEGRATION

## 61. Jenkins Pipeline

```text
Checkout
 ↓
Build
 ↓
Test
 ↓
Scan
 ↓
Docker Push
 ↓
Build Info
 ↓
Promotion
```

---

## 62. Jenkins Credential

Store the Artifactory credential in:

```text
Jenkins Credentials Store
```

---

## 63. Jenkins Example

```groovy
stage('Docker Build') {
    steps {
        sh '''
          docker build \
            -t artifactory.example.com/docker-local/payment-service:${VERSION} .
        '''
    }
}

stage('Docker Push') {
    steps {
        sh '''
          docker push \
            artifactory.example.com/docker-local/payment-service:${VERSION}
        '''
    }
}
```

The real pipeline should authenticate securely before push.

---

# PART XX — GITHUB ACTIONS

## 64. GitHub Actions Flow

```text
Checkout
 ↓
Build
 ↓
Scan
 ↓
Login
 ↓
Push
 ↓
Build Info
```

---

## 65. Security

Production publishing should normally be restricted to:

```text
protected branches
release tags
protected environments
```

Do not expose production registry credentials to untrusted PRs.

---

# PART XXI — GITLAB CI

## 66. GitLab Flow

```text
Build
 ↓
Scan
 ↓
Login
 ↓
Push
 ↓
Build Info
 ↓
Promotion
```

---

## 67. Protected Variables

Store credentials as:

```text
protected
masked
```

and scope them appropriately.

---

# PART XXII — KUBERNETES CONSUMPTION

## 68. Kubernetes Pull

```text
Kubernetes
    |
    v
Artifactory Docker Repository
    |
    v
Image
```

---

## 69. imagePullSecrets

If authentication requires a Kubernetes secret:

```yaml
imagePullSecrets:
  - name: artifactory-registry
```

---

## 70. Deployment

Prefer an immutable version:

```yaml
containers:
  - name: payment
    image: artifactory.example.com/docker-local/payment-service:4.2.1
```

For stronger pinning:

```yaml
containers:
  - name: payment
    image: artifactory.example.com/docker-local/payment-service@sha256:...
```

---

# PART XXIII — EKS

## 71. EKS Architecture

```text
GitHub/GitLab/Jenkins
        |
        v
Artifactory
        |
        v
EKS
```

---

## 72. EKS Node Access

The EKS nodes or workloads need an appropriate mechanism to authenticate to the private registry.

Do not assume the CI credential is automatically available to the cluster.

---

## 73. Private EKS

A production environment may use:

```text
private subnets
security groups
VPC routing
private DNS
NAT/proxy where required
```

---

# PART XXIV — HELM + DOCKER

## 74. Helm Relationship

Example:

```text
Helm Chart:
1.4.0

Application:
4.2.1

Image:
4.2.1
Digest:
sha256:...
```

---

## 75. Values

Example:

```yaml
image:
  repository: artifactory.example.com/docker-local/payment-service
  tag: "4.2.1"
```

For digest-based deployment:

```yaml
image:
  repository: artifactory.example.com/docker-local/payment-service
  digest: "sha256:..."
```

The exact chart implementation should determine how tag and digest are represented.

---

# PART XXV — GITOPS

## 76. GitOps Architecture

```text
Git
 ↓
Argo CD
 ↓
Helm
 ↓
Kubernetes
 ↓
Artifactory
```

---

## 77. GitOps Release

CI:

```text
Build
 ↓
Scan
 ↓
Publish
```

GitOps:

```text
Update desired image version/digest
 ↓
Argo CD
 ↓
Deploy
```

---

## 78. Separation of Responsibilities

CI should build artifacts.

GitOps should manage desired deployment state.

Artifactory should provide the immutable artifact.

---

# PART XXVI — RETENTION

## 79. Why Retain Images?

For:

```text
rollback
audit
incident response
support
reproducibility
```

---

## 80. Development Images

Examples:

```text
feature builds
commit builds
nightly images
```

can usually have shorter retention according to policy.

---

## 81. Production Images

Retain:

```text
current release
rollback versions
supported releases
```

according to business and operational requirements.

---

## 82. Cleanup

Before deleting:

```text
Check age
 ↓
Check deployment
 ↓
Check rollback
 ↓
Check support status
 ↓
Check retention policy
 ↓
Delete if eligible
```

---

# PART XXVII — IMAGE SECURITY

## 83. Run as Non-Root

Example:

```dockerfile
USER 10001
```

---

## 84. Minimal Runtime Image

Prefer:

```text
minimal base
```

when compatible.

---

## 85. No Secrets in Image

Never bake:

```text
AWS credentials
database passwords
API tokens
private keys
```

into the image.

---

## 86. Runtime Secrets

Inject through:

```text
Kubernetes Secrets
external secret manager
workload identity
```

according to the platform design.

---

# PART XXVIII — IMAGE SIGNING AND TRUST

## 87. Why Sign Images?

Image signing can provide:

```text
publisher identity
integrity verification
supply-chain trust
```

---

## 88. Verification

A production deployment policy can require:

```text
trusted signature
trusted source
trusted registry
```

The exact implementation depends on the organization's chosen signing technology and admission controls.

---

# PART XXIX — SBOM

## 89. SBOM

Software Bill of Materials identifies components inside the image.

Example:

```text
payment-service
 ├── OpenJDK
 ├── Spring
 ├── Jackson
 └── OS packages
```

---

## 90. Why SBOM Matters

Useful for:

```text
vulnerability response
compliance
dependency visibility
software supply-chain governance
```

---

# PART XXX — PRODUCTION TROUBLESHOOTING

## 91. Docker Login Fails

Check:

```text
registry hostname
DNS
TLS
credential
token
```

---

## 92. 401 Unauthorized

Likely:

```text
authentication problem
```

Check:

```text
token
identity
expiration
login command
```

---

## 93. 403 Forbidden

Likely:

```text
authorization problem
```

Check:

```text
permission target
repository
DEPLOY/READ
project
```

---

## 94. Manifest Unknown

Check:

```text
image name
tag
repository
publication completed
architecture
```

---

## 95. Image Push Is Slow

Investigate:

```text
image size
layer count
network bandwidth
runner location
registry location
compression
```

---

## 96. Image Pull Is Slow

Check:

```text
image size
node network
Artifactory location
caching
layer reuse
```

---

## 97. Kubernetes ImagePullBackOff

Check:

```bash
kubectl describe pod <pod>
```

Then inspect:

```text
image name
tag/digest
imagePullSecrets
DNS
network
registry authorization
Artifactory availability
```

---

## 98. Image Exists but Kubernetes Cannot Pull

Possible causes:

```text
wrong repository
wrong image path
wrong credentials
network restriction
TLS trust
registry endpoint
```

---

## 99. Digest Mismatch

Investigate:

```text
mutable tag
wrong repository
wrong image
multi-arch manifest
deployment reference
```

Use the exact digest from the approved release.

---

# PART XXXI — REAL-WORLD SCENARIOS

## 100. Scenario — Production Uses latest

Problem:

```text
deployment:
latest
```

Fix:

```text
immutable release tag
+
digest tracking
```

---

## 101. Scenario — Developer Can Delete Production Images

Problem:

```text
DELETE
```

permission too broad.

Fix:

```text
remove DELETE
 ↓
restrict repository
 ↓
protect release workflow
```

---

## 102. Scenario — Vulnerable Base Image

Example:

```text
base image
 ↓
critical CVE
```

Response:

```text
identify affected images
 ↓
update base image
 ↓
rebuild
 ↓
scan
 ↓
publish new version
 ↓
promote
 ↓
redeploy
```

---

## 103. Scenario — Image Was Deleted Before Rollback

Response:

```text
identify missing image
 ↓
check retention
 ↓
restore if backup/replication supports it
 ↓
fix retention
 ↓
protect rollback versions
```

---

## 104. Scenario — Artifactory Is Down During Deployment

The deployment may fail to pull new images.

Design:

```text
high availability
caching
capacity planning
backup/DR
```

Kubernetes already-running containers may continue running, but new image pulls and replacement pods can fail depending on availability and image cache state.

---

## 105. Scenario — Runner Compromised

Response:

```text
stop runner
 ↓
revoke registry credentials
 ↓
review Artifactory audit
 ↓
identify pushed images
 ↓
scan/inspect suspicious images
 ↓
review deployments
 ↓
rotate credentials
```

---

## 106. Scenario — Same Tag Points to Different Content

Problem:

```text
4.2.1
```

was overwritten.

Fix:

```text
enforce immutability
 ↓
reject overwrite
 ↓
create corrected release
```

---

# PART XXXII — ENTERPRISE ARCHITECTURE

## 107. Full Production Architecture

```text
                         Git
                          |
             +------------+------------+
             |                         |
         Jenkins/GitHub            GitLab
             |                         |
             +------------+------------+
                          |
                          v
                    CI Build Runner
                          |
                 +--------+--------+
                 |        |        |
               Build     Test     Scan
                 |        |        |
                 +--------+--------+
                          |
                          v
                     Artifactory
                    /     |      \
                   /      |       \
             Docker     Helm     Generic
                |
                v
              EKS
                |
                v
           Kubernetes
                |
                v
            Monitoring
```

---

## 108. Repository Trust Model

```text
External Registry
       |
       v
Artifactory Remote
       |
       v
Virtual Repository
       |
       v
CI
```

For internal release:

```text
CI
 ↓
Docker Local
 ↓
Scan/Approve
 ↓
Promote
 ↓
Production
```

---

# PART XXXIII — INTERVIEW PREPARATION

## 109. How Do You Integrate Docker with Artifactory?

Answer:

```text
I configure Artifactory as the enterprise Docker registry, use
virtual repositories for controlled dependency/base-image access and
local repositories for internal images. CI authenticates using a
least-privilege service identity, builds and scans the image, pushes an
immutable release, records Build Info and promotes the same image
through environments.
```

---

## 110. Tag vs Digest?

Answer:

```text
A tag is a human-readable reference that can potentially change,
while a digest identifies the exact image content. For production
traceability I prefer immutable release versions and capture or pin
the digest.
```

---

## 111. Why Avoid latest?

Answer:

```text
latest is mutable. The same deployment manifest can resolve to
different image content over time, which makes reproducibility and
rollback difficult.
```

---

## 112. How Do You Secure Docker Registry Access?

Answer:

```text
I use dedicated service identities, least-privilege repository
permissions, secure CI secrets, TLS, credential rotation and
separation between CI publishing credentials and Kubernetes runtime
pull credentials.
```

---

## 113. How Do You Handle a Vulnerable Image?

Answer:

```text
I identify the affected image and digest, determine its source build,
update the vulnerable dependency or base image, rebuild, scan the new
image, publish an immutable replacement, promote it and redeploy
affected workloads.
```

---

## 114. How Do You Troubleshoot ImagePullBackOff?

Answer:

```text
I inspect the pod events and verify image name, repository, tag or
digest, imagePullSecrets, DNS, network connectivity, TLS and
Artifactory authorization. I separate registry authentication
problems from Kubernetes scheduling or application problems.
```

---

## 115. How Do You Implement Build Once, Promote Many for Docker?

Answer:

```text
I build the image once, scan it, push it to Artifactory, capture its
digest and Build Info and promote that exact image through staging and
production without rebuilding.
```

---

## 116. How Do You Handle Multi-Architecture Images?

Answer:

```text
I build and publish the required platforms using Buildx or an
equivalent OCI build process, verify the manifest/index and platform
images, scan them and record the release identity and digests.
```

---

## 117. Why Use Artifactory Remote Docker Repositories?

Answer:

```text
They can cache approved external images and provide a controlled
enterprise access path, reducing direct dependency on public
registries and improving build consistency and availability.
```

---

# PART XXXIV — PRODUCTION CHECKLIST

## 118. Registry

```text
[ ] local repository
[ ] remote repository
[ ] virtual repository
[ ] naming convention
[ ] repository permissions
```

---

## 119. Security

```text
[ ] least privilege
[ ] service identity
[ ] TLS
[ ] token rotation
[ ] image scanning
[ ] secret scanning
[ ] signing where required
[ ] SBOM where required
```

---

## 120. Build

```text
[ ] reproducible Dockerfile
[ ] trusted base image
[ ] multi-stage build
[ ] .dockerignore
[ ] no secrets in image
[ ] non-root runtime
```

---

## 121. Release

```text
[ ] immutable version
[ ] digest
[ ] Build Info
[ ] promotion
[ ] approval
[ ] rollback target
```

---

## 122. Kubernetes

```text
[ ] imagePullSecrets if required
[ ] digest tracking
[ ] registry connectivity
[ ] runtime identity
[ ] rollback
```

---

## 123. Operations

```text
[ ] retention
[ ] cleanup
[ ] audit
[ ] backup/DR
[ ] monitoring
[ ] incident response
```

---

# PART XXXV — GOLDEN RULES

## 124. Rules

```text
1. Treat container images as production software artifacts.

2. Use Artifactory as the controlled enterprise registry.

3. Separate Docker local, remote and virtual repository purposes.

4. Use virtual repositories for controlled dependency/base-image access.

5. Use dedicated service identities for CI.

6. Never use Artifactory administrator credentials for normal image
   publishing.

7. Restrict READ, DEPLOY and DELETE permissions independently.

8. Never store registry credentials in Git.

9. Never bake secrets into container images.

10. Use trusted and maintained base images.

11. Prefer multi-stage builds for smaller runtime images.

12. Run production containers as non-root where practical.

13. Scan images before trusted production promotion.

14. Use SBOM and signing where organizational security requirements
    call for them.

15. Do not use latest as a production release identity.

16. Use immutable version tags and track image digests.

17. Build once and promote the same image.

18. Do not rebuild the image separately for each environment.

19. Keep CI publishing credentials separate from Kubernetes runtime
    credentials.

20. Protect production rollback images with appropriate retention.

21. Use private connectivity for private Artifactory deployments.

22. Do not disable TLS verification as a permanent workaround.

23. Use remote repositories to control external image dependencies.

24. Audit image publication, promotion and deletion.

25. Treat runner compromise as a supply-chain incident.

26. Test image rollback in a real environment.

27. Verify the exact digest running in production.

28. Keep Docker, OCI, Helm and Kubernetes release relationships
    traceable.

29. Validate exact Artifactory, Docker/OCI, JFrog CLI and CI integration
    behavior against the deployed versions before production rollout.
```

---

# END OF 18-Docker-Artifactory-Integration.md
