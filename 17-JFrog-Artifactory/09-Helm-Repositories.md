# 17-JFrog-Artifactory
# 09-Helm-Repositories

## 1. Purpose

This file covers Helm repositories in JFrog Artifactory from fundamentals through production DevOps and Kubernetes usage.

It covers:

- Helm fundamentals
- Helm charts
- Chart structure
- Chart.yaml
- values.yaml
- templates
- dependencies
- Helm packaging
- Helm repositories
- Artifactory Helm local, remote and virtual repositories
- chart versioning
- application versioning
- chart publishing
- chart installation
- chart authentication
- CI/CD integration
- Jenkins
- GitHub Actions
- GitLab CI
- Kubernetes and EKS
- OCI-based Helm charts
- chart security
- provenance and signing concepts
- chart promotion
- dependency management
- chart retention
- production architecture
- troubleshooting
- real-world scenarios
- interview preparation

---

# PART I — HELM FUNDAMENTALS

## 2. What Is Helm?

Helm is a package manager for Kubernetes.

It packages Kubernetes application resources into reusable units called:

```text
Helm Charts
```

A chart can contain:

```text
Deployments
Services
ConfigMaps
Secrets references
Ingress
ServiceAccounts
RBAC
Jobs
CRDs
other Kubernetes resources
```

---

## 3. Why Helm Is Used

Without Helm:

```text
Many Kubernetes YAML files
        ↓
manual parameter changes
        ↓
manual deployment
```

With Helm:

```text
Helm Chart
    +
Values
    ↓
Rendered Kubernetes Manifests
    ↓
Deployment
```

Benefits:

```text
templating
reusability
versioning
release management
configuration
dependency management
rollback
```

---

## 4. Helm Architecture

Conceptually:

```text
Developer / CI
      |
      v
   Helm CLI
      |
      +------> Chart Repository
      |
      v
 Kubernetes API Server
```

Helm itself renders charts and communicates with Kubernetes through the Kubernetes API.

---

## 5. Helm Chart

A Helm chart is a directory/package containing Kubernetes deployment templates and metadata.

Typical structure:

```text
payment-service/
├── Chart.yaml
├── values.yaml
├── charts/
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   └── _helpers.tpl
└── .helmignore
```

---

# PART II — CHART STRUCTURE

## 6. Chart.yaml

`Chart.yaml` contains chart metadata.

Example:

```yaml
apiVersion: v2
name: payment-service
description: Payment service Helm chart
type: application
version: 1.4.0
appVersion: "4.2.1"
```

---

## 7. Chart Version

Example:

```yaml
version: 1.4.0
```

This is the Helm chart version.

It should be incremented when the chart itself changes.

---

## 8. App Version

Example:

```yaml
appVersion: "4.2.1"
```

This describes the application version represented by the chart.

Important:

```text
chart version
≠
application version
```

---

## 9. Why Separate Chart and App Versions?

Example:

```text
Chart:
1.4.0

Application:
4.2.1
```

The application can remain unchanged while the chart changes because of:

```text
resource changes
security settings
probes
affinity
HPA
Ingress
configuration
```

---

## 10. values.yaml

`values.yaml` contains default configurable values.

Example:

```yaml
replicaCount: 3

image:
  repository: artifactory.company.com/docker-local/payment-service
  tag: "4.2.1"

service:
  port: 8080
```

---

## 11. Values Override

Values can be overridden during deployment.

Example:

```bash
helm upgrade --install payment \
  ./payment-service \
  -f values-prod.yaml
```

Or:

```bash
helm upgrade --install payment \
  ./payment-service \
  --set image.tag=4.2.2
```

For production, prefer controlled values files or GitOps configuration rather than excessive command-line overrides.

---

## 12. templates Directory

Templates contain Kubernetes resource definitions.

Example:

```text
templates/
├── deployment.yaml
├── service.yaml
├── ingress.yaml
├── serviceaccount.yaml
└── _helpers.tpl
```

---

## 13. Helm Template Syntax

Example:

```yaml
image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

Helm substitutes values during rendering.

---

## 14. Template Rendering

Flow:

```text
Chart
 +
values.yaml
 +
override values
      |
      v
Helm Template Engine
      |
      v
Kubernetes YAML
```

---

## 15. helm template

Use:

```bash
helm template payment ./payment-service
```

This renders Kubernetes manifests without installing them.

Useful for:

```text
debugging
CI validation
review
policy scanning
```

---

## 16. Helm Lint

Use:

```bash
helm lint ./payment-service
```

It validates chart structure and detects common issues.

---

## 17. Helm Package

Package a chart:

```bash
helm package ./payment-service
```

Output may look like:

```text
payment-service-1.4.0.tgz
```

---

# PART III — ARTIFACTORY HELM REPOSITORIES

## 18. Helm Local Repository

A Helm local repository stores organization-owned charts.

Example:

```text
helm-local
```

Publishing flow:

```text
CI
 ↓
helm package
 ↓
helm-local
```

---

## 19. Helm Remote Repository

A Helm remote repository proxies an external Helm chart source.

Example:

```text
helm-remote
```

Conceptually:

```text
Helm Client
 ↓
helm-virtual
 ↓
helm-remote
 ↓
Approved External Chart Repository
```

---

## 20. Helm Virtual Repository

A virtual repository aggregates:

```text
helm-local
helm-remote
```

Example:

```text
helm-virtual
```

Consumers use the virtual endpoint.

---

## 21. Recommended Architecture

```text
                     Developers / CI
                            |
                            v
                       helm-virtual
                        /         \
                       /           \
                      v             v
                 helm-local     helm-remote
                                    |
                                    v
                            Approved Chart Source
```

---

## 22. Why Use a Virtual Repository?

Benefits:

```text
one endpoint
centralized governance
private chart access
approved external charts
simpler client configuration
```

---

## 23. Internal Chart Flow

```text
Developer / CI
      |
      v
helm-virtual
      |
      v
helm-local
      |
      v
payment-service-1.4.0.tgz
```

---

## 24. External Chart Flow

```text
Helm Client
    |
    v
helm-virtual
    |
    v
helm-remote
    |
    v
External Chart Repository
```

---

# PART IV — HELM CHART VERSIONING

## 25. Chart Versioning

Helm charts commonly use semantic versioning.

Example:

```text
1.4.0
```

---

## 26. Chart Version Changes

Increment chart version when changing:

```text
templates
values
dependencies
deployment behavior
security configuration
```

---

## 27. Application Version

Example:

```text
appVersion: "4.2.1"
```

The application version can change independently from chart behavior.

---

## 28. Example

```text
Chart:
payment-service 1.4.0

Application:
payment-service 4.2.1
```

A chart-only change:

```text
1.4.0 → 1.4.1
```

could still deploy:

```text
application 4.2.1
```

---

## 29. Immutable Chart Releases

Once:

```text
payment-service-1.4.0.tgz
```

is approved, avoid replacing it with different content.

Benefits:

```text
reproducibility
rollback
auditability
```

---

# PART V — HELM DEPENDENCIES

## 30. Chart Dependencies

A chart can depend on other charts.

Example:

```text
payment-service
    |
    +---- redis
    |
    +---- postgres
```

The exact production architecture may instead use managed services or separately managed workloads.

---

## 31. dependencies in Chart.yaml

Conceptually:

```yaml
dependencies:
  - name: redis
    version: 20.x.x
    repository: https://example.com/charts
```

Use approved repository endpoints in enterprise environments.

---

## 32. helm dependency update

Common command:

```bash
helm dependency update
```

It resolves chart dependencies according to the chart configuration.

---

## 33. helm dependency build

Common command:

```bash
helm dependency build
```

It rebuilds dependencies from the lock/configuration state where applicable.

---

## 34. Chart.lock

Helm can generate:

```text
Chart.lock
```

to capture dependency resolution information.

This improves repeatability for chart dependency management.

---

## 35. Dependency Repository Governance

Avoid allowing every developer or CI job to pull charts from arbitrary internet repositories.

Prefer:

```text
Approved chart source
 ↓
Artifactory remote
 ↓
helm-virtual
 ↓
CI
```

---

# PART VI — HELM OCI

## 36. Helm and OCI

Modern Helm versions support storing charts in OCI-compliant registries.

Conceptually:

```text
Helm Chart
 ↓
OCI Registry
 ↓
Artifactory
```

This differs from the traditional HTTP Helm repository model.

---

## 37. OCI Chart Reference

Conceptually:

```text
oci://artifactory.company.com/helm-local
```

The exact URL structure depends on the Artifactory configuration.

---

## 38. OCI Login

Helm can authenticate to OCI registries using supported registry authentication mechanisms.

Conceptually:

```bash
helm registry login artifactory.company.com
```

Credentials must be supplied securely.

---

## 39. OCI Push

Conceptually:

```bash
helm push payment-service-1.4.0.tgz \
  oci://artifactory.company.com/helm-local
```

Validate exact command syntax against the Helm and Artifactory versions in use.

---

## 40. OCI Pull

Conceptually:

```bash
helm pull \
  oci://artifactory.company.com/helm-local/payment-service \
  --version 1.4.0
```

---

## 41. Traditional vs OCI Helm Repositories

Traditional:

```text
Helm Repository
 ↓
index.yaml
 ↓
Chart Packages
```

OCI:

```text
OCI Registry
 ↓
OCI Artifacts
 ↓
Helm Charts
```

Organizations should standardize one or both models deliberately.

---

# PART VII — HELM INDEX AND REPOSITORY METADATA

## 42. Traditional Helm Repository

A traditional Helm repository commonly exposes:

```text
index.yaml
```

This metadata allows Helm clients to discover charts and versions.

---

## 43. Chart Discovery

Conceptually:

```text
helm repo add
       ↓
Repository URL
       ↓
index.yaml
       ↓
Available Charts
```

---

## 44. helm repo add

Example:

```bash
helm repo add company \
  https://artifactory.company.com/artifactory/helm-virtual/
```

The exact URL depends on the deployment.

---

## 45. helm repo update

Use:

```bash
helm repo update
```

to refresh local repository metadata.

---

## 46. helm search repo

Example:

```bash
helm search repo company/payment-service
```

This can help discover available chart versions.

---

# PART VIII — HELM INSTALLATION AND RELEASES

## 47. Helm Install

Example:

```bash
helm install payment \
  company/payment-service \
  --version 1.4.0
```

Flow:

```text
Helm Client
 ↓
Chart Repository
 ↓
Chart
 ↓
Render
 ↓
Kubernetes API
 ↓
Resources
```

---

## 48. Helm Upgrade

Example:

```bash
helm upgrade payment \
  company/payment-service \
  --version 1.4.1
```

---

## 49. helm upgrade --install

Common deployment pattern:

```bash
helm upgrade --install payment \
  company/payment-service \
  --version 1.4.0
```

This is useful for CI/CD and automation.

---

## 50. Helm Release

A Helm release represents a deployed instance of a chart in a Kubernetes environment.

Example:

```text
Release:
payment-prod
```

Chart:

```text
payment-service-1.4.0
```

---

## 51. Helm Release History

Use:

```bash
helm history payment-prod
```

This helps investigate:

```text
previous versions
upgrade history
rollback targets
deployment status
```

---

## 52. Helm Rollback

Example:

```bash
helm rollback payment-prod 5
```

Rollback should be tested and included in the production deployment strategy.

---

## 53. Rollback Principle

Prefer:

```text
Known-good chart
+
Known-good image
+
Known-good configuration
```

rather than rebuilding during an incident.

---

# PART IX — HELM + DOCKER ARTIFACTORY

## 54. Helm Chart + Docker Image

A Helm chart often references a Docker image:

```yaml
image:
  repository: artifactory.company.com/docker-local/payment-service
  tag: "4.2.1"
```

Architecture:

```text
Helm Chart
    |
    v
Docker Image Reference
    |
    v
Artifactory Docker Repository
```

---

## 55. Two Artifact Types

Artifactory may manage:

```text
Helm Chart
+
Docker Image
```

Example:

```text
helm-local/payment-service:1.4.0

docker-local/payment-service:4.2.1
```

These versions serve different purposes.

---

## 56. Chart Version vs Image Version

Example:

```text
Chart:
1.4.0

Image:
4.2.1
```

The chart controls Kubernetes deployment configuration.

The image contains the application runtime.

---

## 57. Production Traceability

A deployment should allow you to identify:

```text
Helm chart version
Docker image tag
Docker image digest
Git commit
CI build
deployment revision
```

---

# PART X — HELM AUTHENTICATION

## 58. Developer Authentication

Developers need permission to:

```text
pull charts
```

They generally do not need:

```text
publish
delete
admin
```

---

## 59. CI Authentication

CI may need:

```text
READ helm-virtual
DEPLOY helm-local
```

depending on the pipeline.

---

## 60. Runtime Authentication

Kubernetes does not normally need Helm repository access during ordinary Pod startup.

Important distinction:

```text
CI / GitOps controller
     ↓
Helm Chart Repository

Kubernetes
     ↓
Docker Registry
```

The deployed Pods pull container images, not Helm charts.

---

## 61. Secret Handling

Never commit:

```text
Artifactory password
access token
registry credential
```

Use:

```text
CI secret store
Kubernetes Secret where appropriate
workload identity where supported
```

---

# PART XI — HELM + JENKINS

## 62. Jenkins Architecture

```text
Git
 ↓
Jenkins
 ↓
Helm
 ↓
Artifactory Helm Repository
 ↓
Kubernetes
```

---

## 63. Jenkins Chart Dependency

CI may:

```bash
helm dependency update
```

using the approved Artifactory endpoint.

---

## 64. Jenkins Chart Validation

Recommended:

```bash
helm lint ./chart
helm template ./chart
```

Then run:

```text
security scanning
policy checks
```

---

## 65. Jenkins Package

Example:

```bash
helm package ./payment-service
```

Output:

```text
payment-service-1.4.0.tgz
```

---

## 66. Jenkins Publication

The chart can then be published to the configured Artifactory Helm repository.

For OCI:

```bash
helm push ...
```

For traditional repository workflows, use the supported Artifactory publishing mechanism for the deployed version.

---

## 67. Jenkins Deployment

Example:

```bash
helm upgrade --install payment-prod \
  company/payment-service \
  --version 1.4.0 \
  -f values-prod.yaml
```

Production deployments should use approvals/GitOps controls where required.

---

# PART XII — HELM + GITHUB ACTIONS

## 68. GitHub Actions Flow

```text
GitHub
 ↓
Actions
 ↓
Helm Lint
 ↓
Helm Template
 ↓
Security Scan
 ↓
Package
 ↓
Publish
 ↓
Deploy
```

---

## 69. GitHub Actions Concept

```yaml
- name: Helm Lint
  run: helm lint ./chart

- name: Render
  run: helm template payment ./chart

- name: Package
  run: helm package ./chart
```

---

## 70. OCI Publication

Conceptually:

```yaml
- run: helm registry login artifactory.company.com
- run: helm push payment-service-1.4.0.tgz oci://artifactory.company.com/helm-local
```

Authentication must use GitHub's secure credential mechanism.

---

# PART XIII — HELM + GITLAB

## 71. GitLab CI Flow

```text
GitLab
 ↓
Runner
 ↓
helm lint
 ↓
helm template
 ↓
Scan
 ↓
Package
 ↓
Publish
 ↓
Deploy/GitOps
```

---

## 72. GitLab Variables

Use:

```text
masked variables
protected variables
environment approvals
```

for sensitive credentials.

---

# PART XIV — HELM + GITOPS

## 73. Helm in GitOps

A common production pattern is:

```text
Git
 ↓
Helm Chart / Values
 ↓
GitOps Controller
 ↓
Kubernetes
```

The GitOps controller can retrieve charts from:

```text
Artifactory
```

and deploy them to Kubernetes.

---

## 74. Argo CD + Artifactory

Conceptually:

```text
Git
 ↓
Application configuration
 ↓
Argo CD
 ↓
Helm Chart
 ↓
Artifactory
 ↓
Kubernetes
```

Authentication is configured according to the GitOps controller and Artifactory security model.

---

## 75. Multi-Cluster GitOps

For multiple clusters:

```text
Git
 ↓
Argo CD
 ↓
Cluster A
Cluster B
Cluster C
```

Charts can be centrally stored in:

```text
Artifactory
```

while environment-specific values remain controlled through GitOps configuration.

---

## 76. Why Artifactory + GitOps?

Separation of responsibilities:

```text
Git
→ desired configuration

Artifactory
→ packaged artifacts/charts/images

GitOps Controller
→ synchronization

Kubernetes
→ runtime
```

---

# PART XV — HELM SECURITY

## 77. Chart Security

Scan Helm charts for:

```text
misconfigured RBAC
privileged containers
hostPath
hostNetwork
unsafe capabilities
missing securityContext
secrets
public services
weak resource policies
```

---

## 78. Render Before Scanning

Do not only scan the raw templates.

Render:

```bash
helm template
```

Then inspect the resulting Kubernetes manifests.

---

## 79. Why Rendered Manifests Matter

Templates can hide final values.

Example:

```text
values
 +
templates
 =
actual Kubernetes YAML
```

Security scanners should evaluate effective manifests where possible.

---

## 80. Kubernetes Security in Helm

A production chart should consider:

```text
runAsNonRoot
readOnlyRootFilesystem
allowPrivilegeEscalation
capabilities
seccomp
resource requests/limits
NetworkPolicy
RBAC
Pod Security
```

---

## 81. Secrets

Avoid placing real production secrets directly in:

```text
values.yaml
```

Prefer:

```text
External Secrets
Secrets Manager
Vault
Kubernetes Secret
```

according to the environment.

---

## 82. Chart Signing and Provenance

Helm supports chart provenance/signing mechanisms.

Conceptually:

```text
Chart
 ↓
Sign
 ↓
Publish
 ↓
Verify
```

Use signing where organizational supply-chain requirements demand it.

---

## 83. OCI Artifact Security

When Helm charts are stored as OCI artifacts:

```text
Chart
 ↓
OCI Registry
 ↓
Artifactory
```

the organization can apply registry-level security and access controls.

---

# PART XVI — HELM TROUBLESHOOTING

## 84. Troubleshooting Layers

Use:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP/OCI Registry
 ↓
Authentication
 ↓
Authorization
 ↓
Repository
 ↓
Chart
 ↓
Kubernetes Rendering
 ↓
Kubernetes API
```

---

## 85. Helm 401

Check:

```text
repository authentication
token
credentials
registry login
CI secrets
```

---

## 86. Helm 403

Check:

```text
repository permission
project access
publish permission
token scope
```

---

## 87. Helm 404

Check:

```text
chart name
chart version
repository URL
virtual repository
OCI path
```

---

## 88. Chart Not Found

Check:

```bash
helm repo update
helm search repo <name>
```

For OCI, verify the registry path and version.

---

## 89. helm dependency update Failure

Check:

```text
dependency repository
chart name
version constraint
network
authentication
repository availability
```

---

## 90. helm upgrade Failure

Start with:

```bash
helm status <release>
helm history <release>
```

Then inspect Kubernetes:

```bash
kubectl get events
kubectl describe pod
```

---

## 91. Helm Template Failure

Use:

```bash
helm lint ./chart
helm template ./chart
```

Inspect:

```text
values
templates
indentation
conditionals
required values
```

---

## 92. Helm Values Not Applied

Check precedence among:

```text
chart defaults
values files
multiple -f files
--set
```

Avoid excessive overrides that make production behavior difficult to reason about.

---

## 93. Chart Published but Cannot Install

Check:

```text
chart repository
virtual membership
chart version
consumer permission
chart metadata
```

---

## 94. Helm Repository Works Locally but CI Fails

Possible:

```text
developer credentials
local repository cache
different Helm version
different repository URL
missing CI secret
```

Standardize CI.

---

# PART XVII — PRODUCTION ARCHITECTURE

## 95. Standard Helm Architecture

```text
                    Developers / CI
                           |
                           v
                      helm-virtual
                       /        \
                      /          \
                     v            v
                helm-local     helm-remote
                                  |
                                  v
                           Approved Sources
```

---

## 96. Production CI Flow

```text
Git
 ↓
Helm Lint
 ↓
Helm Template
 ↓
Security Scan
 ↓
Package
 ↓
Publish
 ↓
Artifactory
 ↓
Promotion
 ↓
GitOps / Deployment
 ↓
Kubernetes
```

---

## 97. Helm + Docker + Kubernetes

```text
                  Artifactory
                 /           \
                /             \
        Helm Repository     Docker Repository
             |                    |
             v                    v
         Helm Chart           Image
             \                    /
              \                  /
               v                v
                    Kubernetes
```

---

## 98. Helm + Argo CD

```text
                     Git
                      |
                      v
                    Argo CD
                   /      \
                  /        \
                 v          v
             Helm Chart    Values
                  |
                  v
             Artifactory
                  |
                  v
              Kubernetes
```

---

## 99. Multi-Environment

Example:

```text
values-dev.yaml
values-stage.yaml
values-prod.yaml
```

Chart:

```text
same chart
```

Application:

```text
same image digest
```

Configuration:

```text
environment-specific
```

---

## 100. Multi-Cluster

```text
             GitOps
                |
                v
              Argo CD
        /         |         \
       v          v          v
   EKS-Dev    EKS-Stage   EKS-Prod
       \          |          /
        \         |         /
         \        |        /
             Artifactory
```

The exact topology can vary.

---

# PART XVIII — HELM RETENTION

## 101. Why Retain Charts?

For:

```text
rollback
audit
reproducibility
incident response
release history
```

---

## 102. Development Chart Cleanup

Development charts can often have shorter retention.

Example:

```text
feature builds
temporary charts
```

can be cleaned according to policy.

---

## 103. Production Chart Retention

Keep charts required for:

```text
supported releases
rollback
compliance
incident investigation
```

---

## 104. Never Delete Blindly

Before cleanup:

```text
Check active releases
Check rollback targets
Check GitOps references
Check retention policy
```

---

# PART XIX — HIGH AVAILABILITY AND DR

## 105. Helm Repository Availability

CI/GitOps depends on chart availability.

Design for:

```text
Artifactory HA
network resilience
storage resilience
backup
DR
```

as appropriate to the environment.

---

## 106. Helm Chart Backup

Back up:

```text
chart artifacts
repository configuration
permissions
required metadata
```

according to the supported Artifactory architecture.

---

## 107. Restore Testing

A backup is not enough.

Test:

```text
restore
chart retrieval
authentication
CI publication
GitOps synchronization
```

---

# PART XX — REAL-WORLD SCENARIOS

## 108. Scenario — Helm Chart Cannot Be Downloaded

Check:

```text
DNS
TLS
repository URL
authentication
authorization
chart
version
virtual repository
```

---

## 109. Scenario — GitOps Controller Cannot Pull Chart

Check:

```text
Artifactory URL
repository type
controller credentials
repository permission
chart version
network
TLS
```

---

## 110. Scenario — Helm Upgrade Breaks Production

Response:

```text
Check release status
 ↓
Check Helm history
 ↓
Identify last known-good revision
 ↓
Check Kubernetes events
 ↓
Rollback if appropriate
 ↓
Investigate root cause
```

---

## 111. Scenario — Chart Changed but Application Image Did Not

Possible:

```text
chart-only release
```

For example:

```text
Chart:
1.4.0 → 1.4.1

Image:
4.2.1
```

This is valid when only Kubernetes deployment configuration changes.

---

## 112. Scenario — Image Changed but Chart Did Not

A pipeline may update:

```text
image.tag
```

or digest through controlled values.

Example:

```text
Chart:
1.4.0

Image:
4.2.1 → 4.2.2
```

---

## 113. Scenario — External Chart Repository Down

If chart is already available through Artifactory cache:

```text
may continue
```

If unavailable and uncached:

```text
dependency may fail
```

For critical charts, consider internally governed copies or approved dependencies.

---

## 114. Scenario — Security Issue in Helm Chart

Flow:

```text
Identify vulnerable chart
 ↓
Identify deployed releases
 ↓
Render manifests
 ↓
Assess actual impact
 ↓
Patch chart
 ↓
Scan
 ↓
Test
 ↓
Publish
 ↓
Promote
 ↓
Deploy
```

---

## 115. Scenario — Helm Repository Storage Full

Response:

```text
Identify growth
 ↓
Review old development charts
 ↓
Review retention
 ↓
Clean approved content
 ↓
Expand capacity
 ↓
Monitor forecasting
```

---

# PART XXI — INTERVIEW PREPARATION

## 116. What Is a Helm Repository in Artifactory?

Answer:

```text
It is an Artifactory repository used to store, proxy or aggregate
Helm charts. Local repositories store internal charts, remote
repositories provide controlled access to external chart sources and
virtual repositories provide a unified endpoint.
```

---

## 117. What Is the Difference Between Chart Version and App Version?

Answer:

```text
The chart version identifies the Helm package itself, while
appVersion describes the application version represented by the
chart. They can change independently.
```

---

## 118. Why Use Helm Virtual Repository?

Answer:

```text
It provides one stable endpoint for consumers while Artifactory
combines internal charts with approved external chart sources.
```

---

## 119. What Is the Difference Between Helm and Docker Repositories?

Answer:

```text
A Helm repository stores Kubernetes application packages, while a
Docker/OCI repository stores container images. A Helm chart often
references an image stored in the Docker repository.
```

---

## 120. How Do You Secure Helm Charts?

Answer:

```text
I control repository access, use least privilege, scan rendered
manifests, avoid hardcoded secrets, use approved dependencies,
control chart provenance/signing where required and promote immutable
chart releases.
```

---

## 121. How Do You Troubleshoot Helm 401?

Answer:

```text
I check repository authentication, credentials, token validity,
registry login and CI secret injection.
```

---

## 122. How Do You Troubleshoot Helm 403?

Answer:

```text
I verify that authentication succeeded and then inspect repository
permissions, project access, token scope and publication rights.
```

---

## 123. How Do You Troubleshoot a Chart Not Found?

Answer:

```text
I verify the repository URL, chart name, version and virtual
repository membership. For traditional repositories I also refresh
the repository metadata. For OCI charts I verify the OCI registry
path and version.
```

---

## 124. How Do You Use Helm with GitOps?

Answer:

```text
Git stores desired configuration, Artifactory stores packaged charts
and images, and the GitOps controller such as Argo CD synchronizes
the desired state to Kubernetes. The running Pods pull container
images from the approved registry.
```

---

## 125. Does Kubernetes Need Helm Repository Access at Runtime?

Answer:

```text
Normally the deployment controller needs chart access, but the
application Pods do not need Helm repository access. Pods normally
pull their container images from the Docker/OCI registry.
```

---

## 126. How Do You Promote Helm Releases?

Answer:

```text
I validate and publish the chart once, then promote the same immutable
chart through environments while using environment-specific
configuration. I avoid rebuilding the chart for each environment.
```

---

## 127. How Do You Handle a Helm Production Failure?

Answer:

```text
I inspect helm status and history, Kubernetes events and workload
health, identify the last known-good release and rollback when
appropriate. Then I investigate the root cause and prevent recurrence.
```

---

## 128. How Would You Design Helm Repositories for Large Organizations?

Answer:

```text
I standardize local, remote and virtual repositories, define chart
ownership and naming, use RBAC, control external dependencies, enforce
versioning and immutability, scan rendered manifests, implement
retention and provide HA, backup and DR.
```

---

# PART XXII — PRODUCTION CHECKLIST

## 129. Repository

```text
[ ] helm-local
[ ] helm-remote
[ ] helm-virtual
[ ] naming standard
[ ] ownership
[ ] approved external sources
[ ] access controls
```

---

## 130. Charts

```text
[ ] Chart.yaml
[ ] chart version
[ ] appVersion
[ ] values.yaml
[ ] templates
[ ] dependencies
[ ] Chart.lock where applicable
[ ] immutable release policy
```

---

## 131. Security

```text
[ ] repository authentication
[ ] least privilege
[ ] chart scanning
[ ] rendered manifest scanning
[ ] dependency scanning
[ ] secret protection
[ ] provenance/signing where required
```

---

## 132. CI/CD

```text
[ ] helm lint
[ ] helm template
[ ] security scan
[ ] package
[ ] publish
[ ] provenance
[ ] promotion
[ ] rollback
```

---

## 133. GitOps

```text
[ ] Git source
[ ] Artifactory chart source
[ ] Argo CD/controller authentication
[ ] environment values
[ ] multi-cluster strategy
[ ] sync monitoring
```

---

## 134. Operations

```text
[ ] monitoring
[ ] audit
[ ] storage
[ ] retention
[ ] backup
[ ] restore testing
[ ] HA
[ ] DR
```

---

# PART XXIII — GOLDEN RULES

## 135. Rules

```text
1. Use Artifactory as the controlled Helm chart boundary.

2. Use helm-local for organization-owned charts.

3. Use helm-remote for approved external chart sources.

4. Use helm-virtual for consumer access where appropriate.

5. Keep chart versions immutable after release.

6. Do not confuse chart version with application version.

7. Keep Docker image identity separately traceable.

8. Prefer immutable image digests for high-assurance production
   deployments.

9. Do not store production secrets in values.yaml.

10. Render Helm charts before security and policy evaluation.

11. Use helm lint and helm template in CI.

12. Control chart dependencies through approved repositories.

13. Use Chart.lock where appropriate for reproducibility.

14. Build once and promote the same chart.

15. Keep rollback-capable chart versions.

16. Use least-privilege Artifactory credentials.

17. Do not give Kubernetes application Pods Helm repository access
    unless there is a specific architectural requirement.

18. Use GitOps for controlled multi-environment and multi-cluster
    deployment where appropriate.

19. Monitor chart repository health and storage.

20. Do not treat remote cache as backup.

21. Test restore and disaster recovery.

22. Treat repository and chart changes as production changes.

23. Document chart ownership and lifecycle.

24. Control external chart sources.

25. Validate exact Helm, OCI and Artifactory commands and repository
    URLs against the deployed versions before production rollout.
```

---

# END OF 09-Helm-Repositories.md
