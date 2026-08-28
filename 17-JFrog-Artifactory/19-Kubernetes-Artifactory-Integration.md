# Kubernetes-Artifactory-Integration

## 1. Purpose

This file provides a production-oriented guide to integrating Kubernetes with JFrog Artifactory.

The focus is the complete container supply chain:

```text
Developer
   |
   v
Git
   |
   v
CI/CD
   |
   +--> Build
   +--> Test
   +--> Scan
   |
   v
Artifactory
   |
   +--> Docker/OCI Images
   +--> Helm Charts
   +--> Build Info
   |
   v
Kubernetes
   |
   +--> Pull Image
   +--> Start Pod
   +--> Run Application
   |
   v
Production
```

This file covers:

- Kubernetes and Artifactory architecture
- Container image repositories
- imagePullSecrets
- private registries
- ServiceAccounts
- namespaces
- RBAC
- imagePullSecrets security
- Docker/OCI image references
- image tags and digests
- Kubernetes Deployments
- StatefulSets
- DaemonSets
- Jobs/CronJobs
- Helm integration
- OCI Helm concepts
- GitOps
- Argo CD
- EKS
- private networking
- authentication
- secret rotation
- admission controls
- image signing concepts
- SBOM concepts
- security scanning
- CI/CD separation
- runtime identity
- troubleshooting
- production scenarios
- disaster recovery
- interview preparation

---

# PART I — FUNDAMENTALS

## 2. Why Kubernetes Needs Artifactory

Kubernetes needs container images to create application containers.

A typical flow:

```text
Kubernetes Node
     |
     v
Container Runtime
     |
     v
Artifactory
     |
     v
Container Image
```

---

## 3. Artifactory as Container Registry

Artifactory can provide:

```text
Docker registry
OCI registry
image storage
remote image caching
virtual repositories
RBAC
audit
retention
Build Info
security controls
```

---

## 4. Kubernetes Responsibilities

Kubernetes handles:

```text
scheduling
deployment
scaling
service discovery
health checks
rolling updates
rollback
```

Artifactory handles:

```text
image/package storage
distribution
versioning
artifact metadata
repository access
```

---

# PART II — PRODUCTION ARCHITECTURE

## 5. Basic Architecture

```text
                 CI/CD
                   |
                   v
             Artifactory
             /          \
      Docker/OCI       Helm
          |               |
          +-------+-------+
                  |
                  v
              Kubernetes
                  |
          +-------+-------+
          |               |
        Pod A           Pod B
          |               |
          +-------+-------+
                  |
              Application
```

---

## 6. Image Pull Flow

When a Pod starts:

```text
Kubernetes Scheduler
       |
       v
Node selected
       |
       v
Kubelet
       |
       v
Container Runtime
       |
       v
Artifactory Registry
       |
       v
Image
       |
       v
Container
```

---

## 7. Pull Happens on the Node

The Kubernetes API server does not download the container image.

The node's container runtime performs the pull.

Typical runtime:

```text
containerd
```

---

## 8. Why This Matters

If a Pod reports:

```text
ImagePullBackOff
```

investigate the node-to-registry path.

Do not only investigate:

```text
Kubernetes API server
```

---

# PART III — REPOSITORY DESIGN

## 9. Docker Local

Stores internal images:

```text
docker-local
```

Example:

```text
payment-service:4.2.1
```

---

## 10. Docker Remote

Caches approved external images:

```text
docker-remote
```

Example:

```text
public base images
```

---

## 11. Docker Virtual

Provides a common endpoint:

```text
docker-virtual
 |
 +--> docker-local
 |
 +--> docker-remote
```

---

## 12. Kubernetes Consumption

A common architecture:

```text
Kubernetes
    |
    v
docker-virtual
```

This allows a controlled registry endpoint.

---

# PART IV — IMAGE REFERENCES

## 13. Image Tag

Example:

```yaml
image:
  repository: artifactory.example.com/docker-local/payment-service
  tag: "4.2.1"
```

---

## 14. Image Digest

Example:

```text
artifactory.example.com/docker-local/payment-service@sha256:...
```

---

## 15. Tag vs Digest

Tag:

```text
4.2.1
```

is readable.

Digest:

```text
sha256:...
```

identifies exact content.

---

## 16. Production Recommendation

Use:

```text
immutable version
+
digest verification
```

Avoid:

```text
latest
```

for production deployments.

---

# PART V — AUTHENTICATION

## 17. Private Registry

If Artifactory requires authentication:

```text
Kubernetes
    |
    v
Registry Credentials
    |
    v
Artifactory
```

---

## 18. imagePullSecrets

Kubernetes can use an image pull secret.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: artifactory-registry
type: kubernetes.io/dockerconfigjson
```

The secret data should be generated securely rather than committed with real credentials.

---

## 19. Creating a Registry Secret

Concept:

```bash
kubectl create secret docker-registry artifactory-registry \
  --docker-server=artifactory.example.com \
  --docker-username="$ARTIFACTORY_USER" \
  --docker-password="$ARTIFACTORY_TOKEN" \
  -n payment
```

Do not expose the command containing the actual secret in shell history or CI logs.

---

## 20. Using imagePullSecrets

```yaml
spec:
  imagePullSecrets:
    - name: artifactory-registry
```

---

## 21. Pod-Level Usage

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: payment
spec:
  imagePullSecrets:
    - name: artifactory-registry
  containers:
    - name: payment
      image: artifactory.example.com/docker-local/payment-service:4.2.1
```

---

# PART VI — SERVICEACCOUNT

## 22. ServiceAccount

A ServiceAccount can provide default image pull secrets for Pods using that ServiceAccount.

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: payment-sa
imagePullSecrets:
  - name: artifactory-registry
```

---

## 23. Pod Using ServiceAccount

```yaml
spec:
  serviceAccountName: payment-sa
```

---

## 24. Why Use a ServiceAccount?

It can standardize:

```text
image pull configuration
workload identity
RBAC
application identity
```

---

# PART VII — NAMESPACE DESIGN

## 25. Namespace

Example:

```text
payment
orders
platform
monitoring
```

---

## 26. Secret Isolation

A Secret is namespace-scoped.

Therefore:

```text
payment namespace
    |
    +--> artifactory-registry
```

does not automatically make the secret available to:

```text
orders namespace
```

---

## 27. Enterprise Pattern

Use namespace boundaries:

```text
payment
  |
  +--> payment ServiceAccount
  +--> payment registry secret

orders
  |
  +--> orders ServiceAccount
  +--> orders registry secret
```

---

# PART VIII — DEPLOYMENT

## 28. Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payment
  template:
    metadata:
      labels:
        app: payment
    spec:
      serviceAccountName: payment-sa
      containers:
        - name: payment
          image: artifactory.example.com/docker-local/payment-service:4.2.1
          ports:
            - containerPort: 8080
```

---

## 29. Digest Deployment

For stronger content pinning:

```yaml
containers:
  - name: payment
    image: artifactory.example.com/docker-local/payment-service@sha256:...
```

---

# PART IX — ROLLING UPDATES

## 30. Kubernetes Rolling Update

```text
Old Pods
  |
  +--> Pod 1
  +--> Pod 2
  +--> Pod 3

New Pods
  |
  +--> Pod 4
  +--> Pod 5
  +--> Pod 6
```

The exact rollout behavior depends on Deployment strategy and update settings.

---

## 31. Image Update

```text
4.2.1
 ↓
4.2.2
```

Kubernetes pulls the new image when new Pods are created.

---

## 32. imagePullPolicy

Common behavior:

```yaml
imagePullPolicy: IfNotPresent
```

or:

```yaml
imagePullPolicy: Always
```

Choose based on immutable tagging and operational policy.

---

# PART X — IMAGEPULLPOLICY

## 33. IfNotPresent

The runtime uses a locally cached image if available.

Useful with:

```text
immutable version tags
```

---

## 34. Always

The runtime checks the registry for the referenced image.

Even with `Always`, an immutable digest provides stronger content identity.

---

## 35. Production Guidance

Prefer:

```text
immutable version/digest
```

rather than relying on:

```text
mutable tags
+
Always
```

as the primary correctness mechanism.

---

# PART XI — STATEFULSETS

## 36. StatefulSet

For stateful workloads:

```text
database
message broker
stateful service
```

Images can still come from Artifactory.

---

## 37. Example Flow

```text
StatefulSet
 ↓
Pod
 ↓
containerd
 ↓
Artifactory
 ↓
Image
```

---

# PART XII — DAEMONSETS

## 38. DaemonSet

DaemonSets run Pods across nodes.

Examples:

```text
logging agents
monitoring agents
security agents
network components
```

---

## 39. Registry Impact

A DaemonSet rollout can cause many nodes to pull an image.

Therefore consider:

```text
registry capacity
network bandwidth
image size
cache
rollout strategy
```

---

# PART XIII — JOBS AND CRONJOBS

## 40. Jobs

Jobs can use Artifactory images:

```yaml
containers:
  - name: migration
    image: artifactory.example.com/docker-local/db-migration:4.2.1
```

---

## 41. CronJobs

Recurring jobs can also pull from Artifactory.

Monitor:

```text
registry availability
credentials
image retention
```

---

# PART XIV — HELM

## 42. Helm + Artifactory

Helm charts can be stored in Artifactory.

Architecture:

```text
CI
 ↓
Artifactory Helm/OCI
 ↓
Helm
 ↓
Kubernetes
```

---

## 43. Helm Values

Example:

```yaml
image:
  repository: artifactory.example.com/docker-local/payment-service
  tag: "4.2.1"
```

---

## 44. Digest in Helm

Example:

```yaml
image:
  repository: artifactory.example.com/docker-local/payment-service
  digest: "sha256:..."
```

The chart must be designed to render the resulting image reference correctly.

---

# PART XV — GITOPS

## 45. GitOps Architecture

```text
Developer
   |
   v
Git
   |
   v
CI
   |
   v
Artifactory
   |
   v
GitOps Repository
   |
   v
Argo CD
   |
   v
Kubernetes
```

---

## 46. CI Responsibility

CI:

```text
build
test
scan
publish
```

---

## 47. GitOps Responsibility

GitOps:

```text
desired state
version update
deployment synchronization
drift detection
rollback
```

---

## 48. Artifactory Responsibility

Artifactory:

```text
immutable artifact
image distribution
artifact metadata
```

---

# PART XVI — ARGO CD

## 49. Argo CD Pull Model

Argo CD runs inside/around the Kubernetes environment and reconciles desired state from Git.

Concept:

```text
Git
 ↓
Argo CD
 ↓
Kubernetes
```

---

## 50. Image Source

The Kubernetes manifest points to:

```text
Artifactory image
```

Example:

```text
artifactory.example.com/docker-local/payment-service:4.2.1
```

---

## 51. Release Flow

```text
Developer
 ↓
Git
 ↓
CI
 ↓
Artifactory
 ↓
Update Git desired state
 ↓
Argo CD
 ↓
Kubernetes
```

---

# PART XVII — EKS

## 52. EKS Architecture

```text
AWS
 |
 +--> VPC
      |
      +--> EKS
            |
            +--> Nodes
                  |
                  +--> Pods
                         |
                         v
                    Artifactory
```

---

## 53. Private EKS

Production EKS clusters may use:

```text
private subnets
private nodes
security groups
VPC routing
private DNS
```

---

## 54. Artifactory Network Path

Example:

```text
EKS Node
   |
   v
VPC Route
   |
   v
Private Connectivity
   |
   v
Artifactory
```

The exact connectivity depends on where Artifactory is hosted.

---

# PART XVIII — EKS IMAGE PULL

## 55. Pull Process

```text
Pod scheduled
 ↓
Kubelet
 ↓
containerd
 ↓
DNS
 ↓
network
 ↓
TLS
 ↓
authentication
 ↓
Artifactory
 ↓
image layers
```

---

## 56. Failure Domains

A pull can fail at:

```text
DNS
network
TLS
authentication
authorization
repository
image
node storage
```

---

# PART XIX — EKS SECURITY GROUPS

## 57. Security Groups

Ensure the node or workload network path can reach the registry endpoint.

Typical secure path:

```text
EKS node
 ↓
TCP 443
 ↓
Artifactory
```

Do not open unnecessary ports.

---

# PART XX — PRIVATE DNS

## 58. DNS

If Artifactory uses a private hostname:

```text
artifactory.company.internal
```

ensure EKS nodes can resolve it.

---

## 59. Troubleshooting DNS

From a diagnostic Pod:

```bash
nslookup artifactory.company.internal
```

or:

```bash
dig artifactory.company.internal
```

---

# PART XXI — TLS

## 60. TLS

Kubernetes nodes must trust the Artifactory certificate chain.

Check:

```text
hostname
certificate
chain
expiration
trust store
```

---

## 61. Do Not Disable Verification

Avoid permanent fixes such as:

```text
insecure registry
TLS verification disabled
```

unless a narrowly controlled architecture explicitly requires it and security has approved it.

---

# PART XXII — REGISTRY SECRET MANAGEMENT

## 62. Secret Lifecycle

```text
Create credential
 ↓
Create Kubernetes Secret
 ↓
Deploy
 ↓
Pull images
 ↓
Rotate credential
 ↓
Update Secret
 ↓
Validate
 ↓
Revoke old credential
```

---

## 63. Rotation Challenge

Kubernetes Pods may continue running after a registry credential changes.

The new credential matters when a node needs to pull an image.

Test:

```text
new Pod
```

after rotation.

---

# PART XXIII — EXTERNAL SECRET MANAGERS

## 64. Why External Secret Management?

Kubernetes Secrets are sensitive objects and should be protected appropriately.

Enterprise environments may use:

```text
AWS Secrets Manager
HashiCorp Vault
External Secrets Operator
other approved secret platforms
```

---

## 65. Generic Architecture

```text
Secret Manager
      |
      v
Kubernetes Secret
      |
      v
ServiceAccount / Pod
      |
      v
Artifactory
```

Use the organization's approved implementation.

---

# PART XXIV — WORKLOAD IDENTITY

## 66. CI vs Runtime Identity

Do not confuse:

```text
CI identity
```

with:

```text
application runtime identity
```

CI may:

```text
push image
```

Runtime may:

```text
pull image
```

---

## 67. Least Privilege

Example:

```text
CI:
DEPLOY docker-local

Runtime:
READ docker-virtual
```

---

# PART XXV — RBAC

## 68. Kubernetes RBAC

Kubernetes RBAC controls access to Kubernetes API resources.

It does not directly grant Artifactory registry permissions.

---

## 69. Separate Authorization Layers

```text
Kubernetes RBAC
       +
Registry Authorization
       +
Network Authorization
```

All three may matter.

---

# PART XXVI — NETWORKPOLICY

## 70. NetworkPolicy

A Kubernetes NetworkPolicy may restrict Pod egress.

If application Pods or nodes need registry connectivity, ensure the architecture permits it.

---

## 71. Diagnostic Example

If:

```text
DNS works
TLS works
credentials work
```

but image pull still fails, investigate:

```text
node/network path
NetworkPolicy
security groups
firewall
proxy
```

---

# PART XXVII — ADMISSION CONTROL

## 72. Why Admission Control?

Production clusters can enforce policies such as:

```text
only approved registries
no latest
must use signed images
must use non-root
```

---

## 73. Approved Registry Policy

Example rule:

```text
images must originate from:
artifactory.example.com/*
```

This prevents workloads from bypassing the enterprise registry.

---

# PART XXVIII — IMAGE SIGNING

## 74. Image Signing

Image signing provides a mechanism to verify:

```text
who published the image
whether content was modified
```

---

## 75. Deployment Verification

A production policy may require:

```text
trusted signature
+
approved registry
+
approved image
```

The exact implementation depends on the organization's signing and admission stack.

---

# PART XXIX — SBOM

## 76. SBOM

An SBOM describes components inside an image.

Example:

```text
payment-service
 ├── Java
 ├── Spring
 ├── Jackson
 └── Linux packages
```

---

## 77. Incident Response

If a vulnerability affects:

```text
library X
```

SBOM data helps identify:

```text
which images contain X
```

---

# PART XXX — IMAGE SCANNING

## 78. Scan Before Promotion

Preferred:

```text
Build
 ↓
Scan
 ↓
Publish
 ↓
Promote
```

or a controlled candidate repository:

```text
Build
 ↓
Candidate
 ↓
Scan
 ↓
Promote
```

---

## 79. Scan Types

Possible checks:

```text
CVE
OS packages
application dependencies
secrets
licenses
configuration
malware
```

---

# PART XXXI — DEPLOYMENT STRATEGY

## 80. Rolling Update

```text
Old:
3 Pods

New:
0
```

Then gradually:

```text
Old:
2
New:
1

Old:
1
New:
2

Old:
0
New:
3
```

---

## 81. Blue-Green

```text
Blue → current
Green → new
```

Switch traffic after validation.

---

## 82. Canary

```text
1% traffic
 ↓
5%
 ↓
25%
 ↓
50%
 ↓
100%
```

Exact traffic management depends on the platform.

---

# PART XXXII — ROLLBACK

## 83. Kubernetes Rollback

A Deployment can roll back to a previous revision.

Example:

```bash
kubectl rollout history deployment/payment -n payment
kubectl rollout undo deployment/payment -n payment
```

---

## 84. Artifact Rollback

The previous image must still exist:

```text
4.2.1
```

If it has been deleted, Kubernetes rollback may not be able to recreate the Pods.

---

## 85. Best Practice

Retention must support:

```text
current
+
known-good rollback versions
```

---

# PART XXXIII — DISASTER RECOVERY

## 86. Artifactory Failure

If Artifactory becomes unavailable:

```text
new Pods may fail to pull images
```

depending on node cache and workload state.

---

## 87. Existing Pods

Already-running containers may continue running.

But:

```text
node replacement
Pod rescheduling
rolling deployment
scaling
```

may require image access.

---

## 88. DR Requirements

Plan for:

```text
Artifactory backup
replication
HA
registry availability
DNS failover where applicable
credential recovery
```

---

# PART XXXIV — TROUBLESHOOTING

## 89. ImagePullBackOff

First:

```bash
kubectl describe pod <pod> -n <namespace>
```

Look at Events.

---

## 90. ErrImagePull

Common causes:

```text
wrong image
wrong tag
registry unreachable
authentication
authorization
TLS
```

---

## 91. Unauthorized

Example:

```text
pull access denied
401
```

Check:

```text
imagePullSecret
token
username
registry hostname
```

---

## 92. Forbidden

Example:

```text
403
```

Check:

```text
Artifactory permission target
repository
service identity
READ permission
```

---

## 93. Secret Not Found

Check:

```bash
kubectl get secret artifactory-registry -n payment
```

---

## 94. ServiceAccount Not Using Secret

Check:

```bash
kubectl get serviceaccount payment-sa -n payment -o yaml
```

Verify:

```yaml
imagePullSecrets:
  - name: artifactory-registry
```

---

## 95. Wrong Namespace

A registry secret in:

```text
default
```

cannot automatically satisfy a Pod in:

```text
payment
```

---

## 96. DNS Troubleshooting

From a diagnostic Pod:

```bash
nslookup artifactory.example.com
curl -vk https://artifactory.example.com/
```

---

## 97. TLS Troubleshooting

Check:

```bash
openssl s_client \
  -connect artifactory.example.com:443 \
  -servername artifactory.example.com
```

Use this only as a diagnostic command; certificate verification should remain enabled in production.

---

## 98. Network Troubleshooting

Check:

```text
security groups
route tables
firewall
proxy
NetworkPolicy
NAT
private endpoints
```

---

## 99. Node-Level Troubleshooting

Check runtime logs where appropriate:

```bash
journalctl -u containerd
```

Exact runtime/service name depends on the Kubernetes distribution.

---

# PART XXXV — REAL-WORLD SCENARIOS

## 100. Scenario — Pods Cannot Pull After New Node Launch

Existing nodes work.

New nodes fail.

Likely areas:

```text
new node IAM/network
DNS
security group
route
image pull secret
runtime configuration
```

---

## 101. Scenario — Existing Pods Work but Scaling Fails

Likely:

```text
new image pull required
```

Investigate:

```text
registry
credentials
network
image retention
```

---

## 102. Scenario — Only One Namespace Fails

Check:

```text
namespace Secret
ServiceAccount
RBAC
NetworkPolicy
```

---

## 103. Scenario — Image Exists in Artifactory but Pull Fails

Check exact:

```text
registry
repository
image
tag
digest
```

Then:

```text
READ permission
credentials
TLS
network
```

---

## 104. Scenario — Production Pulled Wrong Image

Investigate:

```text
mutable tag
imagePullPolicy
registry behavior
deployment manifest
digest
```

Fix:

```text
immutable version
+
digest
```

---

## 105. Scenario — Artifactory Token Leaked

Response:

```text
revoke/rotate
 ↓
audit Artifactory
 ↓
identify image access
 ↓
review deployments
 ↓
replace credentials
```

---

## 106. Scenario — Vulnerable Image Already Running

Response:

```text
identify affected digest
 ↓
identify workloads
 ↓
build fixed image
 ↓
scan
 ↓
publish
 ↓
deploy
 ↓
verify
```

---

# PART XXXVI — PRODUCTION ARCHITECTURE

## 107. Enterprise Model

```text
                    Git
                     |
                     v
                    CI
                     |
          +----------+----------+
          |                     |
        Build                  Scan
          |                     |
          +----------+----------+
                     |
                     v
                Artifactory
              /             \
          Docker            Helm
             |                |
             +-------+--------+
                     |
                     v
                 GitOps Repo
                     |
                     v
                  Argo CD
                     |
                     v
                  EKS
                     |
              +------+------+
              |             |
           Nodes          Pods
              |             |
              +------+------+
                     |
                     v
                Artifactory
```

---

# PART XXXVII — PRODUCTION SECURITY

## 108. Registry Access

```text
[ ] TLS
[ ] least privilege
[ ] service identities
[ ] token rotation
[ ] audit
```

---

## 109. Kubernetes

```text
[ ] namespace isolation
[ ] ServiceAccounts
[ ] RBAC
[ ] NetworkPolicies
[ ] protected Secrets
[ ] admission controls
```

---

## 110. Images

```text
[ ] approved base images
[ ] vulnerability scan
[ ] SBOM
[ ] signing where required
[ ] immutable tags
[ ] digest verification
```

---

# PART XXXVIII — INTERVIEW PREPARATION

## 111. How Does Kubernetes Pull an Image from Artifactory?

Answer:

```text
When a Pod is scheduled, the node's kubelet and container runtime
resolve the image reference and contact the Artifactory registry. If
the registry is private, Kubernetes provides credentials through an
imagePullSecret or the configured workload/registry authentication
mechanism. The runtime downloads the image layers and starts the
container.
```

---

## 112. What Is imagePullSecret?

Answer:

```text
It is a Kubernetes Secret containing registry authentication
information that allows the container runtime to authenticate to a
private registry such as Artifactory.
```

---

## 113. Why Is the Secret Namespace-Specific?

Answer:

```text
Kubernetes Secrets are namespace-scoped. A Secret created in one
namespace is not automatically available to Pods in another namespace.
```

---

## 114. How Do You Troubleshoot ImagePullBackOff?

Answer:

```text
I first inspect Pod events with kubectl describe pod. Then I verify
the image reference, registry DNS, network connectivity, TLS,
imagePullSecret, Artifactory READ permission and image existence. I
also inspect the node/container runtime when required.
```

---

## 115. How Do You Secure Artifactory Access from EKS?

Answer:

```text
I use private network connectivity where possible, TLS, least-
privilege registry identities, protected registry credentials,
namespace isolation and separate CI and runtime identities. I also
consider image signing and admission policies for production.
```

---

## 116. Why Separate CI and Runtime Credentials?

Answer:

```text
CI needs to publish artifacts, while Kubernetes workloads generally
only need to pull them. Giving runtime workloads CI-level DEPLOY
permissions unnecessarily increases the blast radius of a compromise.
```

---

## 117. Why Use Image Digests?

Answer:

```text
A digest identifies exact image content. A tag can be mutable, while a
digest provides stronger reproducibility and release traceability.
```

---

## 118. What Happens if Artifactory Goes Down?

Answer:

```text
Existing containers may continue running if their image is already
present on the node, but new Pods, scaling, replacements and
deployments may fail when the runtime needs to pull an image. This is
why registry availability, caching, HA and DR are important.
```

---

## 119. How Do You Implement GitOps with Artifactory?

Answer:

```text
CI builds, scans and publishes an immutable image to Artifactory.
Then the desired image version or digest is updated in the GitOps
repository. Argo CD reconciles that desired state into Kubernetes.
```

---

# PART XXXIX — PRODUCTION CHECKLIST

## 120. Artifactory

```text
[ ] Docker/OCI local
[ ] Docker/OCI remote
[ ] Docker/OCI virtual
[ ] RBAC
[ ] audit
[ ] retention
[ ] HA/DR
```

---

## 121. Kubernetes

```text
[ ] namespaces
[ ] ServiceAccounts
[ ] imagePullSecrets
[ ] RBAC
[ ] NetworkPolicies
[ ] admission policies
```

---

## 122. Network

```text
[ ] DNS
[ ] TLS
[ ] routing
[ ] security groups
[ ] firewall
[ ] proxy
[ ] private connectivity
```

---

## 123. Image Security

```text
[ ] approved base images
[ ] scanning
[ ] SBOM
[ ] signing
[ ] immutable versions
[ ] digest
```

---

## 124. Operations

```text
[ ] rollback
[ ] retention
[ ] backup
[ ] DR
[ ] monitoring
[ ] incident response
```

---

# PART XL — GOLDEN RULES

## 125. Rules

```text
1. Kubernetes consumes images; Artifactory manages them.

2. The Kubernetes node/container runtime performs the image pull.

3. Use dedicated Artifactory identities.

4. Separate CI DEPLOY permissions from Kubernetes READ permissions.

5. Never put registry credentials directly in Deployment YAML.

6. Protect registry Secrets.

7. Remember that Secrets are namespace-scoped.

8. Use ServiceAccounts to standardize workload identity and pull
   configuration where appropriate.

9. Use private network connectivity for private Artifactory where
   practical.

10. Use TLS for registry communication.

11. Do not permanently disable TLS verification.

12. Prefer immutable release tags and digest pinning.

13. Avoid latest in production.

14. Scan images before production promotion.

15. Use SBOM and signing where required.

16. Enforce approved registries with admission policy where appropriate.

17. Keep CI and runtime credentials separate.

18. Design retention to preserve rollback versions.

19. Test new-node image pulls, not only existing Pods.

20. Treat ImagePullBackOff as a multi-layer troubleshooting problem:
    image, registry, credentials, DNS, TLS and network.

21. Use Artifactory remote repositories to control external images.

22. Use virtual repositories to provide controlled registry endpoints.

23. Use GitOps to separate artifact creation from deployment state.

24. Record image digests and Build Info for traceability.

25. Correlate production Pods to image digest, Artifactory artifact,
    CI build and Git commit.

26. Plan for Artifactory outage because scaling and Pod replacement
    can require image pulls.

27. Test rollback with retained artifacts.

28. Treat leaked registry credentials and compromised runners as
    supply-chain security incidents.

29. Validate exact Kubernetes, containerd, Artifactory and registry
    integration behavior against the deployed versions before
    production rollout.
```

---