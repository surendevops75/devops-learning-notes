# 17-JFrog-Artifactory
# 08-Docker-Repositories

## 1. Purpose

This file covers Docker/OCI repositories in JFrog Artifactory from fundamentals through production DevOps usage.

It covers:

- Container image fundamentals
- Docker Registry architecture
- OCI concepts
- Artifactory Docker local, remote and virtual repositories
- Image names, tags and digests
- Docker push and pull
- Docker authentication
- Repository naming
- Immutable images
- Base image management
- CI/CD integration
- Jenkins
- GitHub Actions
- GitLab CI
- Kubernetes and EKS
- imagePullSecrets
- IAM and registry authentication concepts
- image scanning
- supply-chain security
- Docker content trust/provenance concepts
- image promotion
- retention
- cleanup
- caching
- high availability
- troubleshooting
- production architecture
- real-world scenarios
- interview preparation

---

# PART I — CONTAINER FUNDAMENTALS

## 2. What Is a Container Image?

A container image is a packaged filesystem and metadata used to create a container.

Conceptually:

```text
Application
+
Runtime
+
Libraries
+
Configuration defaults
+
Metadata
=
Container Image
```

---

## 3. Docker Image

A Docker image can contain:

```text
application code
runtime
system libraries
dependencies
entrypoint
environment metadata
filesystem layers
```

Example:

```text
payment-service:4.2.1
```

---

## 4. Container Image Layers

A Docker image is commonly composed of layers.

Conceptually:

```text
Application Layer
      ↓
Dependency Layer
      ↓
Runtime Layer
      ↓
Base OS Layer
```

Layers can be reused between images.

---

## 5. Why Layers Matter

Layers provide:

```text
storage efficiency
build caching
faster transfers
reusability
```

Poor Dockerfile design can create unnecessary layers and large images.

---

## 6. Container Registry

A container registry stores and distributes container images.

Examples include:

```text
Docker Hub
Amazon ECR
JFrog Artifactory
GitHub Container Registry
```

---

## 7. Artifactory as a Docker Registry

Artifactory can act as a private Docker/OCI registry.

Architecture:

```text
Docker Client
      |
      v
Artifactory
      |
      v
Docker Repository
```

It can provide:

```text
image storage
image distribution
access control
remote proxy/cache
security scanning integration
promotion
auditability
retention
```

---

# PART II — DOCKER REGISTRY ARCHITECTURE

## 8. Docker Local Repository

A Docker local repository stores organization-owned images.

Example:

```text
docker-local
```

Publishing:

```text
CI
 ↓
Docker Build
 ↓
docker push
 ↓
docker-local
```

---

## 9. Docker Remote Repository

A Docker remote repository proxies an external registry.

Example:

```text
docker-remote
```

Flow:

```text
Docker Client
 ↓
docker-virtual
 ↓
docker-remote
 ↓
Docker Hub / Approved Registry
```

---

## 10. Docker Virtual Repository

A virtual repository provides a unified image endpoint.

Example:

```text
docker-virtual
```

It can aggregate:

```text
docker-local
docker-remote
```

---

## 11. Recommended Architecture

```text
                         Developers / CI
                               |
                               v
                         docker-virtual
                           /          \
                          /            \
                         v              v
                  docker-local      docker-remote
                                         |
                                         v
                                  Approved Registry
```

---

## 12. Why Use Docker Virtual?

It provides:

```text
one endpoint
centralized access
controlled upstreams
simpler client configuration
```

It also allows repository topology to evolve without changing every consumer.

---

## 13. Internal Image Flow

```text
CI
 ↓
docker build
 ↓
docker-local
 ↓
payment-service:4.2.1
```

---

## 14. External Base Image Flow

Example:

```text
FROM node:22
```

Conceptually:

```text
Docker Build
 ↓
docker-virtual
 ↓
docker-remote
 ↓
Approved external registry
```

This depends on the configured Docker registry and client behavior.

---

# PART III — IMAGE REFERENCES

## 15. Docker Image Name

A container image reference can include:

```text
registry
repository
image
tag
```

Example:

```text
artifactory.company.com/docker-local/payment-service:4.2.1
```

---

## 16. Registry Host

Example:

```text
artifactory.company.com
```

This identifies the registry endpoint.

---

## 17. Repository

Example:

```text
docker-local
```

This identifies the Artifactory repository.

---

## 18. Image Name

Example:

```text
payment-service
```

The image name identifies the application or component.

---

## 19. Tag

Example:

```text
4.2.1
```

Tags are human-friendly references.

---

## 20. Tag Is Not an Immutable Identity

A tag can potentially be moved or overwritten depending on repository policy.

Therefore:

```text
payment-service:4.2.1
```

does not provide the same immutability guarantee as a digest unless tag immutability is enforced.

---

## 21. Image Digest

An image digest identifies content cryptographically.

Example concept:

```text
sha256:<digest>
```

A digest can provide content-addressable identity.

---

## 22. Tag vs Digest

```text
Tag:
payment-service:4.2.1

Digest:
payment-service@sha256:abcdef...
```

For high-assurance production deployment, digest-based pinning can provide stronger immutability.

---

## 23. Latest Tag

Avoid using:

```text
latest
```

as the production deployment identity.

Why?

```text
latest can change
rollback becomes ambiguous
audit becomes harder
reproducibility suffers
```

Prefer:

```text
4.2.1
```

and/or a verified digest.

---

# PART IV — DOCKER PUSH AND PULL

## 24. Docker Login

Conceptually:

```bash
docker login artifactory.company.com
```

Credentials should be securely provided.

---

## 25. Docker Build

Example:

```bash
docker build -t payment-service:4.2.1 .
```

---

## 26. Tag for Artifactory

Example:

```bash
docker tag payment-service:4.2.1 \
  artifactory.company.com/docker-local/payment-service:4.2.1
```

---

## 27. Push

Example:

```bash
docker push \
  artifactory.company.com/docker-local/payment-service:4.2.1
```

Flow:

```text
Docker Client
 ↓
Authentication
 ↓
Artifactory
 ↓
docker-local
 ↓
Image Layers
```

---

## 28. Pull

Example:

```bash
docker pull \
  artifactory.company.com/docker-virtual/payment-service:4.2.1
```

Flow:

```text
Docker Client
 ↓
Artifactory
 ↓
Repository
 ↓
Image Manifest + Layers
 ↓
Local Docker Cache
```

---

## 29. Image Manifest

A container registry manages image manifests describing image content and platform information.

A manifest can reference:

```text
configuration
layers
architecture
OS
digest
```

---

## 30. Multi-Architecture Images

An image can support multiple platforms:

```text
linux/amd64
linux/arm64
```

A multi-platform image can use an image index/manifest list to reference platform-specific images.

---

# PART V — ARTIFACTORY DOCKER REPOSITORIES

## 31. Docker Local Repository

Use for:

```text
application images
internal base images
release images
platform images
```

---

## 32. Docker Remote Repository

Use for:

```text
approved external images
base images
vendor images
public registry content
```

---

## 33. Docker Virtual Repository

Use for:

```text
developer pulls
CI base-image pulls
approved image consumption
```

---

## 34. Base Image Architecture

Example:

```text
CI
 ↓
docker-virtual
 ↓
docker-remote
 ↓
Approved Base Image
 ↓
Docker Build
 ↓
Application Image
 ↓
docker-local
```

---

## 35. Why Proxy Base Images?

Instead of:

```text
Every CI job
 ↓
Docker Hub
```

use:

```text
Every CI job
 ↓
Artifactory
 ↓
Cached Base Image
```

Benefits:

```text
faster builds
reduced public registry traffic
centralized governance
better availability
```

---

## 36. Base Image Governance

Define approved base images such as:

```text
company/node-base
company/python-base
company/java-base
company/nginx-base
```

or approved external images exposed through a controlled repository.

---

## 37. Golden Base Images

A company can maintain hardened base images:

```text
Ubuntu hardened
Node hardened
Python hardened
Java hardened
```

Pipeline:

```text
Official Base
 ↓
Security Hardening
 ↓
Scan
 ↓
Publish
 ↓
docker-local
 ↓
Application Teams
```

---

# PART VI — IMAGE TAGGING STRATEGY

## 38. Bad Tagging

Avoid:

```text
latest
dev
test
prod
```

as the only image identity.

These can be ambiguous.

---

## 39. Better Version Tags

Examples:

```text
4.2.1
4.2.1-build.721
2026.08.26-721
```

Use an organization-standard strategy.

---

## 40. Git Commit Tags

A useful pattern:

```text
payment-service:4.2.1-abc1234
```

This can connect the image to source control.

---

## 41. Build Number Tags

Example:

```text
payment-service:4.2.1-build-721
```

This helps identify the CI execution.

---

## 42. Version + Commit

A strong traceability pattern:

```text
payment-service:4.2.1-abc1234
```

with metadata mapping:

```text
version
commit
pipeline
build
artifact digest
```

---

## 43. Production Deployment Identity

Prefer:

```text
image:
  repository: payment-service
  digest: sha256:...
```

when the deployment platform and release process support digest pinning.

---

# PART VII — IMAGE IMMUTABILITY

## 44. What Is Image Immutability?

An immutable release image means the content associated with the approved production identity does not unexpectedly change.

---

## 45. Why Immutability Matters

It supports:

```text
reproducibility
rollback
audit
incident investigation
compliance
```

---

## 46. Mutable Tag Risk

Suppose:

```text
payment-service:4.2.1
```

initially points to:

```text
digest A
```

and later points to:

```text
digest B
```

A deployment referencing the tag may receive different content.

---

## 47. Immutable Release Policy

Recommended:

```text
Release tag created
 ↓
Artifact approved
 ↓
Tag cannot be overwritten
 ↓
Digest retained
```

---

# PART VIII — DOCKER AUTHENTICATION

## 48. CI Authentication

CI needs credentials to:

```text
pull base images
push application images
```

---

## 49. Pull-Only Identity

Example:

```text
ci-build-read
```

Permissions:

```text
READ docker-virtual
```

---

## 50. Push Identity

Example:

```text
ci-payment-publisher
```

Permissions:

```text
READ docker-virtual
DEPLOY docker-local
```

No admin permission.

---

## 51. Kubernetes Pull Identity

Runtime workloads generally need:

```text
READ
```

not:

```text
DEPLOY
DELETE
ADMIN
```

---

## 52. Token Rotation

Pattern:

```text
Create new credential
 ↓
Update CI/Kubernetes
 ↓
Test
 ↓
Revoke old credential
```

---

## 53. Never Store Docker Credentials in Git

Do not commit:

```text
docker login password
Artifactory token
registry secret
```

Use:

```text
CI secret store
Kubernetes Secret
cloud workload identity
```

where supported.

---

# PART IX — DOCKER + JENKINS

## 54. Jenkins Pipeline

Typical:

```text
Checkout
 ↓
Test
 ↓
Build
 ↓
Scan
 ↓
Docker Login
 ↓
Docker Push
```

---

## 55. Jenkins Build Example

Conceptually:

```groovy
stage('Build Image') {
    steps {
        sh 'docker build -t payment-service:${BUILD_NUMBER} .'
    }
}
```

---

## 56. Jenkins Push

Conceptually:

```groovy
stage('Push Image') {
    steps {
        sh '''
        docker tag payment-service:${BUILD_NUMBER} \
          artifactory.company.com/docker-local/payment-service:${BUILD_NUMBER}

        docker push \
          artifactory.company.com/docker-local/payment-service:${BUILD_NUMBER}
        '''
    }
}
```

Production implementations should use secure credentials and controlled versioning.

---

## 57. Jenkins Credentials

Use:

```text
Jenkins Credentials
```

for registry authentication.

Do not expose credentials in:

```text
console logs
Jenkinsfile
environment dumps
```

---

# PART X — DOCKER + GITHUB ACTIONS

## 58. GitHub Actions Flow

```text
GitHub
 ↓
Actions Runner
 ↓
Docker Build
 ↓
Security Scan
 ↓
Artifactory Login
 ↓
Docker Push
```

---

## 59. GitHub Secrets

Store registry credentials in:

```text
GitHub Secrets
```

or approved identity federation mechanisms.

---

## 60. GitHub Actions Concept

```yaml
- uses: actions/checkout@v4

- name: Build
  run: docker build -t payment-service:${{ github.sha }} .

- name: Push
  run: docker push artifactory.company.com/docker-local/payment-service:${{ github.sha }}
```

Authentication must be configured securely before push.

---

# PART XI — DOCKER + GITLAB

## 61. GitLab CI Flow

```text
GitLab
 ↓
Runner
 ↓
Docker Build
 ↓
Scan
 ↓
Registry Login
 ↓
Docker Push
```

---

## 62. GitLab Variables

Use:

```text
masked variables
protected variables
```

or approved identity mechanisms.

---

## 63. GitLab Docker Pipeline

Conceptually:

```yaml
build:
  script:
    - docker build -t payment-service:$CI_COMMIT_SHA .

publish:
  script:
    - docker push artifactory.company.com/docker-local/payment-service:$CI_COMMIT_SHA
```

---

# PART XII — DOCKER + KUBERNETES

## 64. Kubernetes Image Pull

Architecture:

```text
Kubernetes Node
      |
      v
Container Runtime
      |
      v
Artifactory Docker Registry
      |
      v
Image
```

---

## 65. Kubernetes Does Not Build Images

A production Kubernetes cluster normally consumes already-built images.

Preferred:

```text
CI
 ↓
Build
 ↓
Scan
 ↓
Push
 ↓
Kubernetes
```

---

## 66. imagePullSecrets

Kubernetes can use registry credentials through:

```text
imagePullSecrets
```

Example concept:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: artifactory-registry
type: kubernetes.io/dockerconfigjson
```

The exact secret content must be managed securely.

---

## 67. Pod Using imagePullSecrets

Conceptually:

```yaml
spec:
  imagePullSecrets:
    - name: artifactory-registry
```

---

## 68. Service Account Association

Instead of repeating secrets in every Pod, organizations can associate registry pull credentials with a ServiceAccount where appropriate.

Conceptually:

```text
ServiceAccount
 ↓
imagePullSecrets
 ↓
Pods
```

---

## 69. EKS Registry Authentication

For EKS, authentication can be designed using:

```text
Kubernetes image pull secrets
```

or platform/cloud-specific identity mechanisms where supported by the registry and architecture.

Do not assume AWS IAM alone automatically authenticates to every Artifactory deployment.

---

## 70. Kubernetes Pull Failure

If:

```text
ErrImagePull
ImagePullBackOff
```

check:

```text
DNS
network
registry endpoint
authentication
authorization
image name
tag/digest
TLS
```

---

# PART XIII — DOCKER SECURITY

## 71. Image Security

Scan for:

```text
OS vulnerabilities
application vulnerabilities
malware
secrets
misconfigurations
license issues
```

---

## 72. Base Image Security

A vulnerable base image can affect every downstream image.

Architecture:

```text
Vulnerable Base
      ↓
100 Application Images
      ↓
100 Potentially Vulnerable Images
```

Centralized base-image governance is therefore valuable.

---

## 73. Image Scanning Pipeline

Recommended:

```text
Docker Build
 ↓
Image Scan
 ↓
Policy Evaluation
 ↓
Pass
 ↓
Publish
```

or an equivalent controlled workflow.

---

## 74. Fail the Pipeline

For critical vulnerabilities:

```text
Scan
 ↓
Policy
 ↓
FAIL
```

Do not automatically publish an image that violates mandatory security policy.

---

## 75. Vulnerability Exceptions

If an exception is required:

```text
document
owner
reason
risk
expiry
approval
compensating control
```

Do not create permanent silent exceptions.

---

## 76. Secrets in Images

Never bake:

```text
AWS keys
database passwords
API tokens
private keys
```

into container images.

Use runtime secret management.

---

## 77. Dockerfile Security

Avoid unnecessary packages.

Prefer:

```text
minimal base
non-root user
read-only filesystem where possible
least privileges
```

---

## 78. Non-Root Container

Example concept:

```dockerfile
USER 10001
```

The actual UID and user strategy should follow the organization's standards.

---

## 79. Dockerfile Supply Chain

Pin or control:

```text
base image
package versions
build tools
external downloads
```

Do not blindly execute arbitrary remote scripts during builds.

---

# PART XIV — IMAGE PROMOTION

## 80. Build Once, Promote Many

Preferred:

```text
Build
 ↓
Scan
 ↓
Publish
 ↓
Promote
 ↓
Stage
 ↓
Production
```

Avoid:

```text
Build Dev
Build Stage
Build Prod
```

because the outputs may differ.

---

## 81. Image Promotion

Conceptually:

```text
docker-dev-local
       ↓
validated
       ↓
docker-release-local
       ↓
production
```

The exact repository/promotion model depends on organizational design.

---

## 82. Why Promote Instead of Rebuild?

Because the exact same image:

```text
digest
```

is tested and deployed.

This improves:

```text
traceability
reproducibility
rollback
confidence
```

---

## 83. Image Digest During Promotion

Example:

```text
payment-service
sha256:abc123...
```

The digest remains the content identity while environment labels/tags may change according to promotion policy.

---

# PART XV — REMOTE IMAGE CACHING

## 84. Docker Remote Cache

First pull:

```text
CI
 ↓
Artifactory
 ↓
Remote
 ↓
External Registry
 ↓
Cache
```

Later:

```text
CI
 ↓
Artifactory Cache
```

---

## 85. Cache Benefits

```text
faster builds
less internet traffic
reduced upstream rate pressure
better resilience
centralized access
```

---

## 86. Cache Is Not Full Backup

A remote cache should not automatically be treated as:

```text
complete registry backup
```

Backups must follow the supported Artifactory architecture.

---

## 87. External Registry Outage

If the required base image is cached:

```text
Docker Build
 ↓
Artifactory Cache
```

may continue.

If uncached:

```text
Docker Build
 ↓
Artifactory
 ↓
External Registry unavailable
```

may fail.

---

# PART XVI — IMAGE RETENTION

## 88. Why Retention Matters

Docker images can consume large storage.

Growth can come from:

```text
every build
every commit
every tag
old environments
unused images
remote cache
```

---

## 89. Retention Strategy

Different policies may apply to:

```text
development images
feature images
release images
production images
remote cache
```

---

## 90. Development Cleanup

Example policy:

```text
remove old unreferenced development images
```

after an approved retention period.

---

## 91. Production Retention

Keep enough versions for:

```text
rollback
incident investigation
compliance
release support
```

---

## 92. Never Delete Blindly

Before cleanup:

```text
Check active deployments
Check rollback versions
Check dependencies
Check promotion status
Check retention policy
```

---

# PART XVII — DOCKER TROUBLESHOOTING

## 93. Troubleshooting Model

Use:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP Registry API
 ↓
Authentication
 ↓
Authorization
 ↓
Repository
 ↓
Image
 ↓
Manifest
 ↓
Layer
```

---

## 94. Docker 401

Usually:

```text
authentication failure
```

Check:

```text
docker login
token
credential
registry hostname
```

---

## 95. Docker 403

Usually:

```text
authenticated
but not authorized
```

Check:

```text
repository permissions
deploy/pull permissions
project access
token scope
```

---

## 96. Docker 404

Check:

```text
registry
repository
image
tag
digest
```

---

## 97. Docker Push Failure

Check:

```text
login
repository
deploy permission
image tag
storage
network
TLS
```

---

## 98. Docker Pull Failure

Check:

```text
DNS
network
authentication
authorization
image existence
tag/digest
TLS
```

---

## 99. Kubernetes ImagePullBackOff

Check:

```text
kubectl describe pod
```

Look for:

```text
ErrImagePull
authentication failure
not found
TLS failure
network timeout
```

Then verify:

```text
registry
image
secret
service account
node network
```

---

## 100. Docker Manifest Unknown

Usually indicates:

```text
requested tag/digest does not exist
```

Verify:

```text
image
tag
repository
architecture
```

---

## 101. Architecture Mismatch

Example:

```text
Image built for:
linux/amd64

Node:
linux/arm64
```

The workload may fail to start.

Use appropriate multi-platform images.

---

## 102. Slow Image Pull

Investigate:

```text
image size
layer count
network
Artifactory latency
node bandwidth
cache hit rate
storage
```

---

## 103. Large Image

Use:

```text
multi-stage builds
smaller base images
dependency cleanup
package cleanup
```

---

# PART XVIII — PRODUCTION ARCHITECTURE

## 104. Standard Architecture

```text
                      Developers / CI
                             |
                             v
                       docker-virtual
                        /           \
                       /             \
                      v               v
               docker-local      docker-remote
                                      |
                                      v
                              Approved Registry
```

---

## 105. Production CI/CD

```text
Git
 ↓
CI
 ↓
Unit Tests
 ↓
Docker Build
 ↓
Image Scan
 ↓
Policy Gate
 ↓
Push
 ↓
docker-local
 ↓
Promotion
 ↓
Kubernetes
```

---

## 106. Production Kubernetes

```text
                  GitOps / Deployment
                           |
                           v
                      Kubernetes
                           |
                           v
                    Container Runtime
                           |
                           v
                   Artifactory Registry
                           |
                           v
                     Image by Digest
```

---

## 107. Enterprise Architecture

```text
                    External Registries
                           |
                           v
                    docker-remote
                           |
                           v
                    Security / Policy
                           |
                           v
                    docker-virtual
                       /        \
                      /          \
                  CI / Dev      Runtime
                      |
                      v
                 docker-local
                      |
                      v
                Release Images
```

---

## 108. Multi-Team Strategy

Possible organizational model:

```text
docker-local
 ├── payments/
 ├── orders/
 ├── platform/
 └── shared/
```

The exact structure should be based on ownership and access boundaries.

---

## 109. Repository Naming

Good:

```text
docker-local
docker-remote
docker-virtual
```

or standardized organizational variants.

Avoid:

```text
repo1
new-docker
testfinal
```

---

# PART XIX — HIGH AVAILABILITY AND RELIABILITY

## 110. Registry Availability

Production Artifactory should be designed for:

```text
node failure
storage failure
network failure
upstream failure
load spikes
```

according to the organization's edition and architecture.

---

## 111. Load Balancer

A typical enterprise architecture may use:

```text
Docker Client
     |
     v
Load Balancer
     |
 +---+---+
 |       |
Node A  Node B
 |       |
 +---+---+
     |
 Artifactory
```

---

## 112. Storage

Container repositories can grow rapidly.

Plan:

```text
capacity
IOPS
throughput
backup
replication
```

---

## 113. Registry Outage Strategy

For critical workloads:

```text
High availability
+
cached images
+
rollback images
+
monitoring
+
tested DR
```

---

# PART XX — REAL-WORLD SCENARIOS

## 114. Scenario — CI Cannot Push

Flow:

```text
CI
 ↓
Artifactory
 ↓
docker-local
```

Check:

```text
DNS
TLS
login
permissions
repository
storage
```

---

## 115. Scenario — CI Cannot Pull Base Image

Flow:

```text
Docker Build
 ↓
docker-virtual
 ↓
docker-remote
```

Check:

```text
remote repository
upstream
cache
authentication
network
```

---

## 116. Scenario — Kubernetes Cannot Pull Image

Check:

```text
registry endpoint
DNS
network
imagePullSecret
service account
image
tag/digest
permissions
TLS
```

---

## 117. Scenario — Image Tag Changed

If:

```text
payment-service:4.2.1
```

now points to a different digest:

```text
Investigate tag mutation
Check repository policy
Check deployment history
Check audit logs
Restore immutability
```

---

## 118. Scenario — Vulnerability in Base Image

Flow:

```text
Base Image
 ↓
Vulnerability discovered
 ↓
Identify dependent images
 ↓
Rebuild
 ↓
Scan
 ↓
Publish
 ↓
Redeploy
```

---

## 119. Scenario — Critical Vulnerability in Production Image

Response:

```text
Identify deployed digest
 ↓
Assess vulnerability
 ↓
Build patched image
 ↓
Scan
 ↓
Test
 ↓
Promote
 ↓
Deploy
 ↓
Verify
```

---

## 120. Scenario — Artifactory Storage Full

Response:

```text
Protect service
 ↓
Identify growth source
 ↓
Review development image retention
 ↓
Review remote cache
 ↓
Clean approved content
 ↓
Expand capacity
 ↓
Improve forecasting
```

---

## 121. Scenario — External Registry Outage

If image cached:

```text
Pull may continue
```

If not cached:

```text
Pull/build may fail
```

Mitigation:

```text
approved remote
cache critical images
maintain internal golden images
```

---

# PART XXI — INTERVIEW PREPARATION

## 122. What Is a Docker Repository in Artifactory?

Answer:

```text
It is an Artifactory repository used to store, proxy or aggregate
Docker/OCI images. Local repositories store internal images, remote
repositories proxy external registries and virtual repositories
provide a unified consumer endpoint.
```

---

## 123. What Is the Difference Between Local, Remote and Virtual Docker Repositories?

Answer:

```text
Local stores organization-owned images. Remote represents external
registries and can cache images. Virtual aggregates local and remote
repositories behind a single endpoint for consumers.
```

---

## 124. Why Use Artifactory Instead of Docker Hub Directly?

Answer:

```text
Artifactory gives the organization centralized control over image
storage, access, approved upstreams, caching, security, auditing,
retention and promotion.
```

---

## 125. Tag vs Digest?

Answer:

```text
A tag is a human-readable mutable reference unless immutability is
enforced. A digest identifies image content. For high-assurance
production deployment I prefer a verified immutable release and can
pin the deployment to its digest.
```

---

## 126. Why Avoid latest in Production?

Answer:

```text
latest does not provide a stable release identity. It can point to
different image content over time, making rollback and reproducibility
harder.
```

---

## 127. How Do You Secure Docker CI?

Answer:

```text
I use dedicated registry identities, secret-managed credentials,
least-privilege permissions, image scanning, immutable versioning,
controlled base images and audit logging.
```

---

## 128. How Do You Secure Kubernetes Image Pulls?

Answer:

```text
I provide read-only registry access through the approved Kubernetes
authentication mechanism, use imagePullSecrets or supported workload
identity patterns, pin production images appropriately and verify
DNS, network, TLS and repository permissions.
```

---

## 129. How Do You Troubleshoot ImagePullBackOff?

Answer:

```text
I start with kubectl describe pod to identify the registry error.
Then I check the image reference, DNS, network connectivity,
authentication, imagePullSecret, ServiceAccount, repository
permissions, TLS and image existence.
```

---

## 130. How Do You Handle Base Image Vulnerabilities?

Answer:

```text
I identify affected application images, update the base image,
rebuild, scan, test and promote the patched image. I then redeploy
affected workloads and track the remediation.
```

---

## 131. Why Use Remote Docker Repositories?

Answer:

```text
They provide controlled access and caching for external images. This
reduces direct registry dependency, improves build performance and
allows the organization to govern external image sources.
```

---

## 132. Is a Docker Remote Cache a Backup?

Answer:

```text
No. A remote cache is a dependency-access mechanism, not a substitute
for a supported backup and disaster-recovery strategy.
```

---

## 133. How Do You Design Docker Repositories for Large Organizations?

Answer:

```text
I standardize local, remote and virtual repositories, define image
ownership, use least-privilege RBAC, centralize approved external
registries, enforce immutable releases, implement scanning and
retention and capacity-plan for CI and Kubernetes image-pull bursts.
```

---

## 134. How Do You Promote Images?

Answer:

```text
I build the image once, scan and validate it, publish the immutable
artifact and promote the same image/digest through environments
rather than rebuilding it for each environment.
```

---

# PART XXII — PRODUCTION CHECKLIST

## 135. Repository

```text
[ ] docker-local
[ ] docker-remote
[ ] docker-virtual
[ ] naming standard
[ ] ownership
[ ] approved upstreams
[ ] repository permissions
```

---

## 136. Images

```text
[ ] versioned tags
[ ] immutable release policy
[ ] digest tracking
[ ] no production latest
[ ] base image governance
[ ] multi-architecture strategy
```

---

## 137. Security

```text
[ ] image scanning
[ ] dependency scanning
[ ] secret scanning
[ ] least privilege
[ ] secure registry credentials
[ ] audit
[ ] vulnerability exceptions documented
```

---

## 138. CI/CD

```text
[ ] build
[ ] test
[ ] scan
[ ] push
[ ] provenance
[ ] promotion
[ ] rollback
```

---

## 139. Kubernetes

```text
[ ] registry connectivity
[ ] authentication
[ ] imagePullSecret/identity
[ ] ServiceAccount strategy
[ ] digest/version strategy
[ ] pull monitoring
```

---

## 140. Operations

```text
[ ] storage monitoring
[ ] cache monitoring
[ ] request monitoring
[ ] error monitoring
[ ] upstream monitoring
[ ] retention
[ ] backup
[ ] DR
[ ] restore testing
```

---

# PART XXIII — GOLDEN RULES

## 141. Rules

```text
1. Use Artifactory as the controlled Docker/OCI registry boundary.

2. Use docker-local for organization-owned images.

3. Use docker-remote for approved external image sources.

4. Use docker-virtual for consumer access where appropriate.

5. Treat image tags as references, not inherently immutable identities.

6. Prefer immutable release tags and verified digests.

7. Avoid latest as a production deployment identity.

8. Use dedicated CI registry identities.

9. Give runtime workloads read-only access.

10. Never commit registry credentials.

11. Scan images before production promotion.

12. Govern base images centrally.

13. Do not bake secrets into images.

14. Build once and promote the same image.

15. Keep rollback-capable image versions.

16. Do not treat remote cache as backup.

17. Monitor storage and cache growth.

18. Capacity-plan for CI build bursts and Kubernetes pull bursts.

19. Investigate ImagePullBackOff from the actual Kubernetes event/error.

20. Check DNS, network, TLS, authentication and authorization
    separately during registry troubleshooting.

21. Maintain approved upstream registries.

22. Document repository ownership and retention.

23. Test backup and disaster recovery.

24. Treat registry configuration changes as production changes.

25. Validate exact Docker/OCI behavior, Artifactory URLs and
    authentication mechanisms against the deployed versions before
    production rollout.
```

---

# END OF 08-Docker-Repositories.md
