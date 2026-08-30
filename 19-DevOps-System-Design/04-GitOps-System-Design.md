# GitOps-System-Design

## 1. Purpose

This is a production-oriented, senior-level guide to designing GitOps
platforms for enterprise DevOps and Kubernetes environments.

The objective is to understand GitOps as an operating model and system
architecture, not simply as "Argo CD watches Git."

A mature GitOps architecture provides:

```text
declarative desired state
version-controlled changes
automated reconciliation
auditability
drift detection
repeatable deployments
multi-environment promotion
multi-cluster management
controlled access
safe rollback
disaster recovery
```

Reference:

```text
                    Developers
                         |
                         v
                  Application Repo
                         |
                         v
                    CI Pipeline
                         |
              +----------+----------+
              |                     |
         Build/Test/Security    Artifact
              |                     |
              +----------+----------+
                         |
                         v
                 GitOps Repository
                         |
                 Pull Request / Review
                         |
                         v
                    Argo CD
                         |
             +-----------+-----------+
             |           |           |
          Cluster A   Cluster B   Cluster C
             |           |           |
             +-----------+-----------+
                         |
                    Applications
                         |
                         v
                  Observability
```

---

# PART I — GITOPS FUNDAMENTALS

## 2. What Is GitOps?

GitOps uses Git as the authoritative source for declarative desired
state, combined with an automated reconciliation process that brings
the runtime environment toward that state.

Conceptually:

```text
Desired State in Git
        |
        v
Reconciler
        |
        v
Actual Runtime State
```

If actual state differs from desired state:

```text
Actual != Desired
       |
       v
Reconcile
       |
       v
Actual -> Desired
```

---

## 3. Declarative vs Imperative

Imperative:

```text
kubectl scale deployment payment --replicas=5
```

The operator directly tells the system what action to perform.

Declarative:

```yaml
spec:
  replicas: 5
```

The desired state is stored and a controller reconciles toward it.

GitOps emphasizes:

```text
declare desired state
+
automatically reconcile
```

---

## 4. GitOps Principles

A practical GitOps platform should provide:

```text
1. Desired state is declarative.
2. Desired state is version controlled.
3. Changes are reviewed.
4. Agents/controllers reconcile state.
5. Drift is observable.
6. Access is controlled.
7. Deployments are auditable.
```

---

## 5. GitOps Is More Than Git

A Git repository alone is not GitOps.

This:

```text
Git repository
+
manual kubectl apply
```

is version-controlled Kubernetes administration, but not a complete
GitOps operating model.

A stronger model is:

```text
Git
 |
desired state
 |
controller
 |
continuous reconciliation
 |
cluster
```

---

# PART II — ARCHITECTURE

## 6. GitOps Control Plane

A GitOps control plane commonly includes:

```text
Git provider
 |
GitOps repository
 |
Argo CD
 |
Application definitions
 |
Cluster credentials / cluster registration
 |
Kubernetes API
```

---

## 7. GitOps Data Flow

```text
Developer
 |
PR
 |
GitOps change
 |
Review
 |
Merge
 |
Argo CD detects revision
 |
Compare desired/actual
 |
Sync
 |
Kubernetes API
 |
Resources
```

---

## 8. Application Repository vs GitOps Repository

A common enterprise separation:

```text
Application Repo
 |
+-- source
+-- tests
+-- Dockerfile
+-- CI
```

GitOps Repo:

```text
GitOps Repo
 |
+-- environments
+-- clusters
+-- applications
+-- Helm values
+-- Kustomize overlays
+-- Argo CD definitions
```

This creates a useful boundary:

```text
CI owns artifact creation.
GitOps owns deployment intent.
```

---

## 9. Why Separate Repositories?

Benefits:

```text
different access control
different change lifecycle
clear deployment audit
independent release permissions
reduced coupling
```

But it also adds:

```text
repository coordination
promotion automation
more repositories
```

The trade-off should be deliberate.

---

# PART III — ARGO CD

## 10. Argo CD Role

Argo CD is a declarative GitOps continuous delivery system for Kubernetes.

Conceptually:

```text
Git
 |
Argo CD
 |
Kubernetes
```

It continuously evaluates desired state and actual cluster state.

---

## 11. Argo CD Architecture

Conceptual components:

```text
                 Argo CD API/UI
                       |
                 Repo Server
                       |
              Application Controller
                       |
                  Redis/cache
                       |
                 Kubernetes APIs
```

The exact deployment topology depends on version and configuration.

---

## 12. Application Controller

The application controller is responsible for reconciliation.

Conceptually:

```text
Git desired state
       |
       v
Application Controller
       |
       +--> compare
       |
       +--> sync
       |
       v
Kubernetes
```

---

## 13. Repo Server

The repository server handles retrieval and rendering of application
manifests using supported sources such as:

```text
Helm
Kustomize
directory manifests
plugins where configured
```

---

## 14. API Server

The API layer supports:

```text
UI
CLI
automation
authentication
application management
```

---

## 15. Redis / Cache

Argo CD deployments can use caching components to improve performance.

The exact HA and cache design should follow the supported Argo CD
architecture for the deployed version.

Do not design around undocumented internal behavior.

---

# PART IV — ARGO CD CORE OBJECTS

## 16. Application

An Argo CD Application represents:

```text
source
+
destination
+
desired deployment configuration
```

Conceptually:

```yaml
spec:
  source:
    repoURL: ...
    path: ...
  destination:
    server: ...
    namespace: ...
```

---

## 17. ApplicationSet

ApplicationSet is important for fleets.

Instead of manually defining:

```text
app-a-cluster-1
app-a-cluster-2
app-a-cluster-3
...
```

a generator can create Applications systematically.

---

## 18. Project

Argo CD Projects provide logical boundaries for:

```text
allowed repositories
allowed clusters
allowed namespaces
resource restrictions
```

They are an important multi-team governance mechanism.

---

# PART V — SOURCE TYPES

## 19. Plain YAML

Simple structure:

```text
app/
 |
deployment.yaml
service.yaml
configmap.yaml
```

Advantages:

```text
simple
explicit
easy to understand
```

Disadvantages:

```text
duplication
environment management difficulty
```

---

## 20. Kustomize

Example:

```text
base/
 |
deployment.yaml
service.yaml

overlays/
 |
+-- dev
+-- stage
+-- prod
```

Useful for environment overlays.

---

## 21. Helm

Structure:

```text
chart/
 |
Chart.yaml
values.yaml
templates/
```

Environment values can be separated:

```text
values-dev.yaml
values-stage.yaml
values-prod.yaml
```

---

## 22. Helm vs Kustomize

Helm:

```text
templating
packaging
parameterization
```

Kustomize:

```text
overlay
patching
native Kubernetes YAML transformation
```

Choose based on platform conventions and workload complexity.

---

# PART VI — REPOSITORY DESIGN

## 23. Monorepo GitOps

```text
gitops/
 |
+-- apps/
+-- clusters/
+-- environments/
```

Advantages:

```text
central visibility
simpler global changes
```

Risks:

```text
large repository
ownership complexity
```

---

## 24. Multi-Repo GitOps

```text
platform-repo
application-repo
environment-repo
cluster-repo
```

Advantages:

```text
stronger ownership boundaries
independent lifecycle
```

Risks:

```text
coordination
more automation
```

---

## 25. Recommended Enterprise Separation

One practical model:

```text
Application repositories
        |
        v
Artifact repositories
        |
        v
Environment/GitOps repository
        |
        v
Argo CD
```

Platform configuration can be separated from application deployment
configuration when organizational scale justifies it.

---

# PART VII — ENVIRONMENT DESIGN

## 26. Environment Hierarchy

```text
dev
 |
test
 |
stage
 |
prod
```

GitOps represents the desired state of each environment.

---

## 27. Environment Promotion

```text
Artifact 1.8.0
 |
DEV
 |
tests
 |
STAGE
 |
validation
 |
PROD
```

Promotion should modify deployment references while preserving the
artifact identity.

---

## 28. Configuration Separation

Same image:

```text
payment-api@sha256:ABC...
```

Different environment values:

```text
DEV
DATABASE_HOST=dev-db

STAGE
DATABASE_HOST=stage-db

PROD
DATABASE_HOST=prod-db
```

Never rebuild the image solely to change environment configuration.

---

# PART VIII — MULTI-CLUSTER

## 29. Cluster Fleet

Example:

```text
Argo CD
 |
+-- dev-cluster
+-- stage-cluster
+-- prod-cluster-a
+-- prod-cluster-b
+-- dr-cluster
```

---

## 30. Cluster Registration

Argo CD needs controlled access to target clusters.

The credential design must follow least privilege.

Avoid:

```text
one unrestricted credential for every cluster
```

---

## 31. Cluster Labels

Cluster metadata can support selection:

```text
environment=prod
region=ap-south-1
tier=critical
```

ApplicationSet generators can use cluster metadata to create targeted
applications.

---

# PART IX — APPLICATIONSET

## 32. Why ApplicationSet?

At fleet scale:

```text
100 clusters
+
50 applications
```

manual Application objects become difficult to manage.

ApplicationSet can generate Applications from structured sources.

---

## 33. Cluster Generator

Conceptually:

```text
clusters matching:
environment=prod
```

Then generate:

```text
application -> each matching cluster
```

---

## 34. Git Generator

A Git directory structure can drive application generation.

Example:

```text
apps/
 |
service-a
service-b
service-c
```

ApplicationSet can generate corresponding Applications.

---

## 35. Matrix Generation

A matrix can combine dimensions:

```text
applications
      x
clusters
```

Result:

```text
service-a -> cluster-1
service-a -> cluster-2
service-b -> cluster-1
service-b -> cluster-2
```

This is powerful but must be governed carefully to avoid accidental
fleet-wide changes.

---

# PART X — PROMOTION MODELS

## 36. Image Tag Promotion

Example:

```text
dev -> image 2.4.0
stage -> image 2.4.0
prod -> image 2.4.0
```

Automation updates the GitOps reference.

---

## 37. Image Digest Promotion

More immutable:

```text
service@sha256:abc123
```

The digest identifies exact image content.

---

## 38. Git Promotion

```text
dev branch/config
 |
promotion PR
 |
stage
 |
promotion PR
 |
prod
```

Benefits:

```text
audit
review
traceability
```

---

## 39. Automated Promotion

```text
DEV healthy
 |
automation
 |
STAGE
 |
health
 |
PROD
```

Use automated promotion only when gates and risk requirements support it.

---

# PART XI — DRIFT

## 40. What Is Drift?

Drift exists when:

```text
Git desired state != cluster actual state
```

Example:

```text
Git replicas = 3
Cluster replicas = 5
```

---

## 41. Sources of Drift

```text
manual kubectl change
operator
another controller
emergency modification
configuration error
failed deployment
```

---

## 42. Drift Detection

Argo CD can report differences between desired and live state.

Operators should investigate why drift happened.

---

## 43. Self-Healing

If configured:

```text
manual change
 |
Argo detects drift
 |
reconciliation
 |
desired state restored
```

This is powerful but dangerous if operators expect emergency manual
changes to persist.

---

# PART XII — MANUAL EMERGENCY CHANGES

## 44. Break-Glass

A controlled emergency process:

```text
Incident
 |
temporary manual change
 |
document
 |
stabilize
 |
commit intended final state to Git
 |
reconcile
```

Never allow permanent undocumented production drift.

---

# PART XIII — SYNC

## 45. Manual Sync

```text
Git changed
 |
Argo detects OutOfSync
 |
operator reviews
 |
sync
```

Useful for controlled environments.

---

## 46. Automated Sync

```text
Git changed
 |
Argo detects revision
 |
automatic sync
```

Good for mature low-risk deployment paths.

---

## 47. Prune

Pruning removes resources that are no longer declared.

Powerful:

```text
Git resource removed
 |
sync
 |
resource deleted
```

Therefore pruning should be enabled only with careful repository governance.

---

## 48. Self-Heal

Self-heal restores desired state after live-state changes.

Use it where automatic correction is safe.

---

# PART XIV — SYNC WAVES

## 49. Ordering

Some resources must exist before others.

Conceptual:

```text
wave 0
CRDs
 |
wave 1
operators
 |
wave 2
infrastructure
 |
wave 3
applications
```

Use explicit ordering only where dependency ordering is actually needed.

---

# PART XV — HEALTH

## 50. Health Assessment

GitOps should distinguish:

```text
Synced
Healthy
```

A resource can be:

```text
Synced but unhealthy
```

because desired configuration was applied but the workload failed.

---

## 51. Deployment Health

Useful checks:

```text
available replicas
progressing state
pod readiness
service health
custom application health
```

---

# PART XVI — SYNC WINDOWS

## 52. Deployment Windows

Organizations may restrict production changes:

```text
business hours
maintenance window
freeze period
```

Exceptions can exist for:

```text
security emergency
critical incident
```

Rules should be explicit and auditable.

---

# PART XVII — ROLLBACK

## 53. Git Revert

If:

```text
commit A = good
commit B = bad
```

rollback can be:

```text
revert B
 |
Git desired state returns toward A
 |
Argo reconciles
```

---

## 54. Rollback to Previous Artifact

Example:

```text
prod
 |
v2.0.0 bad
 |
Git change
 |
v1.9.0
 |
Argo sync
```

---

## 55. Rollback Limitation

Git rollback does not automatically reverse:

```text
database migrations
external side effects
data mutations
irreversible API operations
```

GitOps must be combined with application-safe rollback design.

---

# PART XVIII — PROGRESSIVE DELIVERY

## 56. GitOps + Canary

Concept:

```text
Git desired state
 |
Canary controller
 |
5%
 |
20%
 |
50%
 |
100%
```

Health analysis determines promotion.

---

## 57. Blue-Green

```text
Blue = current
Green = new
```

Git can declare the desired active version while traffic management
controls which environment receives requests.

---

# PART XIX — MULTI-ACCOUNT

## 58. AWS Account Boundaries

```text
Argo CD / Management
 |
controlled role
 |
Prod Account
 |
EKS
```

Avoid broad credentials spanning every AWS account.

---

## 59. IAM Roles

Possible:

```text
Argo deployment role
 |
specific cluster resources
```

Runtime identity remains separate:

```text
Application Pod
 |
Workload Identity
 |
AWS role
```

---

# PART XX — SECURITY

## 60. GitOps Threat Model

Threats:

```text
malicious Git change
compromised developer account
compromised CI
stolen Argo credential
malicious manifest
container compromise
repository compromise
```

---

## 61. Repository Security

Protect:

```text
main branch
production directory
deployment manifests
Argo configuration
cluster definitions
```

Use:

```text
PR reviews
CODEOWNERS
branch protection
required checks
```

---

## 62. Argo RBAC

Define:

```text
platform-admin
platform-operator
team-a
team-b
readonly
```

Limit:

```text
applications
projects
clusters
namespaces
actions
```

---

## 63. AppProject Security

Projects can constrain:

```text
source repositories
destination clusters
destination namespaces
resource kinds
```

This is an important multi-tenant control.

---

## 64. Namespace Isolation

Combine:

```text
Argo Project
+
Kubernetes namespace
+
RBAC
+
NetworkPolicy
+
resource quotas
```

No single mechanism should be expected to solve every isolation need.

---

# PART XXI — SECRETS

## 65. Secrets in Git

Plaintext production secrets should not be stored directly in Git.

Alternatives include:

```text
external secret systems
sealed/encrypted secret workflows
cloud secret managers
secret operators
```

The chosen method must support:

```text
rotation
audit
least privilege
recovery
```

---

## 66. Secret Flow

Example:

```text
Git
 |
secret reference
 |
Kubernetes secret controller
 |
Secret Manager
 |
Kubernetes Secret
 |
Pod
```

The exact implementation depends on the selected secret-management
solution.

---

# PART XXII — CERTIFICATES

## 67. Certificate Automation

GitOps can manage:

```text
certificate resources
issuer configuration
ingress configuration
```

A certificate controller can handle issuance and renewal.

Monitor renewal failures because expired certificates can cause outages.

---

# PART XXIII — POLICY

## 68. Policy as Code

GitOps can integrate policy controls:

```text
No privileged containers
No public LoadBalancer where prohibited
Approved registries only
Resource requests required
Required labels
```

Policy can run:

```text
PR
 |
CI
 |
Admission
 |
runtime
```

Defense in depth is stronger than relying on one policy layer.

---

# PART XXIV — ADMISSION CONTROL

## 69. Admission

Kubernetes admission controls can validate or mutate requests before
objects are persisted.

Examples:

```text
image policy
security policy
required labels
resource requirements
```

GitOps should work with admission policy, not attempt to replace it.

---

# PART XXV — SUPPLY CHAIN

## 70. Artifact Chain

```text
Source
 |
CI
 |
Build
 |
Scan
 |
SBOM
 |
Sign
 |
Registry
 |
GitOps
 |
Argo
 |
Cluster
```

GitOps controls deployment intent; it does not automatically prove the
artifact is safe.

---

## 71. Image Verification

Production deployment can require:

```text
approved registry
digest
signature
provenance
security policy
```

---

# PART XXVI — OBSERVABILITY

## 72. GitOps Observability

Monitor:

```text
sync status
health status
sync failures
reconciliation errors
Git fetch errors
API errors
queue/backlog where applicable
```

---

## 73. Deployment Correlation

Record:

```text
commit
 |
application version
 |
deployment timestamp
 |
cluster
 |
namespace
 |
health
```

Then correlate deployment events with application telemetry.

---

# PART XXVII — ALERTING

## 74. Important Alerts

```text
application OutOfSync unexpectedly
application unhealthy
sync repeatedly failing
repository unavailable
cluster unreachable
certificate renewal failure
high deployment failure rate
```

Avoid paging for every harmless temporary synchronization state.

---

# PART XXVIII — ARGO CD HIGH AVAILABILITY

## 75. HA Principles

Consider:

```text
API availability
controller availability
repo server availability
cache
database/state dependencies
network
Git provider
target Kubernetes API
```

The exact HA topology should match the supported Argo CD deployment
model and organizational scale.

---

## 76. Controller Failure

If one controller instance fails in an HA deployment:

```text
remaining controller capacity
 |
continues reconciliation
```

Test the actual configuration rather than assuming HA works.

---

# PART XXIX — GIT PROVIDER FAILURE

## 77. Git Failure

If Git becomes unavailable:

```text
existing workloads
 |
continue running
```

But:

```text
new GitOps changes
 |
may not be detected
```

This separation is important.

---

# PART XXX — ARGO FAILURE

## 78. Argo Controller Failure

Existing applications can continue running in Kubernetes.

However:

```text
new desired state
 |
not reconciled
```

until controller functionality is restored.

---

# PART XXXI — KUBERNETES API FAILURE

## 79. Target Cluster API Failure

Argo may report:

```text
cluster unavailable
```

Applications already running may continue serving traffic depending on
the failure scope.

Deployment operations and reconciliation are affected.

---

# PART XXXII — NETWORK FAILURE

## 80. Argo to Cluster Network Failure

```text
Argo
 |
X
 |
Kubernetes API
```

Impact:

```text
no reconciliation
no deployment
health visibility degraded
```

Runtime may continue independently.

---

# PART XXXIII — MULTI-CLUSTER FAILURE CONTAINMENT

## 81. Fleet-Wide Risk

A bad Git change can affect:

```text
100 clusters
```

if a global generator immediately propagates it.

Controls:

```text
deployment waves
cluster labels
progressive rollout
environment separation
PR approval
health gates
```

---

# PART XXXIV — FLEET MANAGEMENT

## 82. Cluster Lifecycle

GitOps can manage:

```text
cluster addons
namespaces
RBAC
policies
applications
observability
```

Cluster infrastructure itself may remain managed by Terraform/IaC.

A useful separation is:

```text
Terraform
 |
AWS infrastructure / EKS

GitOps
 |
Kubernetes desired state
```

---

# PART XXXV — BOOTSTRAPPING

## 83. App-of-Apps

A bootstrap Application can create other Applications.

Concept:

```text
Root Application
 |
+--> platform
+--> observability
+--> security
+--> applications
```

This can simplify bootstrap but creates a critical root dependency.

Protect the root configuration strongly.

---

## 84. Bootstrap Order

Example:

```text
Cluster
 |
Argo CD
 |
CRDs
 |
Platform operators
 |
Secrets integration
 |
Ingress
 |
Observability
 |
Applications
```

---

# PART XXXVI — DR

## 85. GitOps DR Advantage

If desired state is stored in Git:

```text
new cluster
 |
install Argo
 |
connect Git
 |
apply bootstrap
 |
reconcile applications
```

This can make reconstruction much easier.

But Git is not sufficient if:

```text
secrets unavailable
images unavailable
data unavailable
DNS unavailable
```

---

# PART XXXVII — BACKUP

## 86. What to Back Up

Even with GitOps:

```text
Git repositories
Argo configuration
secret-management configuration
cluster-specific state
persistent data
external system state
```

Recreatable manifests do not replace database backups.

---

# PART XXXVIII — RESTORE TEST

## 87. GitOps Restore

```text
Provision cluster
 |
Install Argo
 |
Restore access
 |
Connect Git
 |
Bootstrap
 |
Sync platform
 |
Sync applications
 |
Restore data
 |
Validate
```

Measure:

```text
time to cluster
time to Argo
time to first app
time to business availability
```

---

# PART XXXIX — DR REGION

## 88. Multi-Region GitOps

Possible:

```text
Central Git
 |
Argo CD
 |
+--> Region A
+--> Region B
```

or separate Argo installations per region.

Choose based on:

```text
failure isolation
network independence
operational complexity
RTO
```

---

# PART XL — DISASTER SCENARIO

## 89. Entire Argo Region Lost

Recovery:

```text
Provision replacement control plane
 |
Restore Argo configuration
 |
Connect Git
 |
Register surviving clusters
 |
Reconcile
```

The applications may remain healthy while the delivery control plane is
being recovered.

---

# PART XLI — PRODUCTION CHANGE FLOW

## 90. Normal Release

```text
Developer
 |
Application PR
 |
CI
 |
Artifact
 |
GitOps automation creates promotion PR
 |
Review
 |
Merge
 |
Argo detects
 |
Sync
 |
Health
 |
Complete
```

---

# PART XLII — AUTOMATED IMAGE UPDATE

## 91. Image Automation

Possible flow:

```text
New image
 |
scan
 |
policy
 |
update GitOps reference
 |
PR
 |
review
 |
merge
 |
Argo
```

Avoid blindly deploying every newly published image.

---

# PART XLIII — GITOPS AND CI BOUNDARY

## 92. Responsibility Split

CI:

```text
compile
test
scan
build
package
publish
```

GitOps:

```text
desired deployment state
environment promotion
deployment configuration
```

Argo:

```text
reconciliation
sync
health
drift
```

Kubernetes:

```text
runtime orchestration
```

---

# PART XLIV — GITOPS ANTI-PATTERNS

## 93. Anti-Patterns

```text
manual kubectl changes as normal workflow
plaintext secrets
one admin credential for every cluster
unprotected GitOps repository
latest image tag
global auto-sync with no blast-radius controls
no drift monitoring
no rollback plan
no health checks
no repository backup
no restore testing
mixing infrastructure and application ownership without boundaries
```

---

# PART XLV — PRODUCTION SECURITY MODEL

## 94. Defense in Depth

```text
Developer Identity
       |
Git Controls
       |
CI Security
       |
Artifact Security
       |
GitOps Review
       |
Argo RBAC
       |
Kubernetes Admission
       |
Runtime RBAC
       |
NetworkPolicy
       |
Cloud IAM
```

Compromise of one layer should not automatically provide unrestricted
control of every layer.

---

# PART XLVI — MULTI-TENANCY

## 95. Team Isolation

Example:

```text
Project: payments
 |
allowed repo
 |
prod cluster
 |
payments namespace
```

Another:

```text
Project: orders
 |
orders repo
 |
prod cluster
 |
orders namespace
```

Use project restrictions to prevent cross-team access.

---

# PART XLVII — PLATFORM TEAM

## 96. Platform Responsibilities

Platform team owns:

```text
Argo CD
cluster registration
projects
shared templates
policy
observability
upgrade strategy
security baseline
```

Application teams own:

```text
application code
service configuration
service-specific manifests
application health
runbooks
```

Ownership must be explicitly documented.

---

# PART XLVIII — DEVELOPER EXPERIENCE

## 97. Golden Path

Developer experience:

```text
Create service
 |
repository template
 |
CI automatically configured
 |
artifact published
 |
GitOps deployment generated
 |
environment available
 |
dashboard available
```

GitOps should reduce operational friction rather than add manual steps.

---

# PART XLIX — TEMPLATE DESIGN

## 98. Application Template

Example:

```text
service-template/
 |
deployment
service
hpa
pdb
networkpolicy
serviceaccount
observability
```

The platform should provide secure defaults.

---

# PART L — POLICY GOVERNANCE

## 99. Required Metadata

Standard labels may include:

```text
app
team
environment
owner
cost-center
version
```

This improves:

```text
observability
cost allocation
ownership
operations
```

---

# PART LI — RESOURCE GOVERNANCE

## 100. Production Defaults

Require where appropriate:

```text
CPU request
memory request
limits
probes
replica strategy
security context
```

GitOps can enforce standards through templates and policy.

---

# PART LII — AUTOSCALING

## 101. HPA

Git-managed HPA:

```yaml
minReplicas: 3
maxReplicas: 20
```

Do not choose arbitrary values.

Base them on:

```text
load
latency
CPU
memory
business metrics
downstream capacity
```

---

# PART LIII — CLUSTER AUTOSCALING

## 102. Workload to Node Scaling

```text
Traffic
 |
HPA
 |
more Pods
 |
insufficient nodes
 |
cluster autoscaler / node provisioning
 |
more capacity
```

The chain must be tested under real workloads.

---

# PART LIV — SYNC AND AUTOSCALING

## 103. Avoid Fighting Controllers

Multiple controllers should not continuously overwrite the same field.

Example problem:

```text
Git says replicas=3
HPA wants replicas=8
```

Design ownership correctly so GitOps does not fight Kubernetes autoscaling.

---

# PART LV — STATE OWNERSHIP

## 104. Field Ownership

Define:

```text
GitOps owns desired application configuration
HPA owns dynamic replica scaling
controller owns status fields
```

Avoid ambiguous ownership.

---

# PART LVI — TROUBLESHOOTING

## 105. Application OutOfSync

Check:

```text
Git revision
rendered manifest
live resource
diff
ignored fields
mutating controllers
manual changes
```

---

## 106. Sync Failed

Check:

```text
Argo logs
Kubernetes API
RBAC
resource validity
webhooks
CRDs
dependencies
```

---

## 107. Sync Succeeds but App Unhealthy

Then:

```text
desired state applied
runtime failed
```

Investigate:

```text
pods
events
logs
probes
config
secrets
network
dependencies
```

Do not keep resyncing blindly.

---

# PART LVII — TROUBLESHOOTING DRIFT

## 108. Persistent Drift

If drift immediately returns:

```text
Git
 |
Argo
 |
resource
 |
another controller changes field
 |
Argo sees drift
```

Identify the competing controller.

---

# PART LVIII — TROUBLESHOOTING ACCESS

## 109. Cluster Permission Failure

Check:

```text
Argo cluster credentials
IAM
Kubernetes RBAC
service account
network connectivity
API endpoint
```

Separate:

```text
AWS authentication
from
Kubernetes authorization
```

---

# PART LIX — TROUBLESHOOTING HELM

## 110. Helm Rendering Failure

Check:

```text
values
chart version
template syntax
required values
API versions
```

Render locally or in CI before merging where practical.

---

# PART LX — TROUBLESHOOTING KUSTOMIZE

## 111. Overlay Failure

Check:

```text
base path
patch target
resource names
namespace
generated resources
```

---

# PART LXI — PRODUCTION INCIDENT

## 112. Bad GitOps Commit

```text
Detect
 |
stop propagation
 |
identify commit
 |
rollback/revert
 |
sync known-good state
 |
validate
 |
investigate
```

If already propagated to many clusters, use fleet-wide containment.

---

# PART LXII — FLEET-WIDE BAD RELEASE

## 113. Containment

```text
Bad revision
 |
pause promotion
 |
stop further waves
 |
rollback affected clusters
 |
validate
```

A single bad commit should not automatically become a global outage.

---

# PART LXIII — BREAK-GLASS

## 114. Emergency Production Access

During severe incident:

```text
Incident authorization
 |
temporary access
 |
manual mitigation
 |
record exact change
 |
restore Git desired state
 |
commit final configuration
```

Emergency access must remain auditable.

---

# PART LXIV — ARCHITECTURE TRADE-OFFS

## 115. GitOps Benefits

```text
auditability
repeatability
drift correction
declarative state
review
rollback
multi-cluster automation
```

Costs:

```text
learning curve
controller complexity
repository management
debugging reconciliation
```

---

# PART LXV — GITOPS VS TRADITIONAL CD

## 116. Comparison

```text
Traditional:
CI -> deploy API -> cluster

GitOps:
CI -> Git -> Argo -> cluster
```

GitOps adds a reconciliation layer and stronger declarative state
management.

---

# PART LXVI — GITOPS VS TERRAFORM

## 117. Different Responsibilities

Terraform commonly manages:

```text
AWS infrastructure
VPC
EKS
IAM
RDS
```

GitOps commonly manages:

```text
Kubernetes applications
Kubernetes configuration
cluster addons
```

There can be overlap, so ownership must be explicit.

---

# PART LXVII — TERRAFORM + GITOPS

## 118. Layered Model

```text
Terraform
 |
AWS infrastructure
 |
EKS
 |
Argo CD
 |
Kubernetes resources
 |
Applications
```

This is a common enterprise separation.

---

# PART LXVIII — OBSERVABILITY ARCHITECTURE

## 119. GitOps Metrics

Track:

```text
sync duration
sync failures
application health
reconciliation errors
cluster reachability
deployment frequency
rollback rate
```

---

# PART LXIX — SLO

## 120. GitOps Platform SLO

Examples:

```text
99.9% availability for Argo API
99.9% successful reconciliation availability
defined maximum reconciliation latency
```

Exact objectives should reflect business impact.

---

# PART LXX — COST

## 121. GitOps Cost Drivers

```text
Argo compute
repository storage
CI image automation
observability
multi-cluster control planes
```

At large scale:

```text
100 clusters
+
thousands of applications
```

requires careful controller sizing and reconciliation design.

---

# PART LXXI — SCALE

## 122. Application Scale

Monitor:

```text
number of Applications
number of resources
number of clusters
Git repository size
manifest rendering time
reconciliation load
API server load
```

---

# PART LXXII — REPOSITORY SCALE

## 123. Large GitOps Repository

Problems:

```text
large clone
slow rendering
complex ownership
large PRs
```

Solutions may include:

```text
repository partitioning
directory ownership
application boundaries
shallow/optimized workflows where supported
```

Do not split repositories purely for fashion.

---

# PART LXXIII — CONTROLLER SCALE

## 124. Scaling Principle

Controller sizing depends on:

```text
applications
resources
reconciliation frequency
clusters
manifest complexity
API latency
```

Benchmark the real workload.

---

# PART LXXIV — API SERVER PRESSURE

## 125. Reconciliation Load

Too many controllers or overly aggressive reconciliation can create:

```text
Kubernetes API load
Git traffic
CPU consumption
```

Tune architecture based on measured behavior.

---

# PART LXXV — NETWORK DESIGN

## 126. Argo Connectivity

```text
Argo
 |
network
 |
Git
```

and:

```text
Argo
 |
network
 |
Kubernetes API
```

Need:

```text
DNS
TLS
firewall
security groups
routing
proxy if applicable
```

---

# PART LXXVI — PRIVATE CLUSTERS

## 127. Private EKS API

If target cluster API is private:

```text
Argo
 |
private network path
 |
EKS API
```

Possible architecture:

```text
central Argo VPC
 |
Transit Gateway / private connectivity
 |
prod VPC
 |
private EKS API
```

The exact network design depends on account and topology.

---

# PART LXXVII — CENTRAL VS IN-CLUSTER ARGO

## 128. Central Argo

```text
Central Argo
 |
+--> Cluster A
+--> Cluster B
+--> Cluster C
```

Benefits:

```text
central management
central visibility
```

Risk:

```text
central control-plane blast radius
```

---

## 129. Regional Argo

```text
Region A Argo -> Region A clusters
Region B Argo -> Region B clusters
```

Benefits:

```text
failure isolation
regional autonomy
```

Costs:

```text
more operations
more controllers
```

---

# PART LXXVIII — ACTIVE-ACTIVE CONTROL PLANE

## 130. Considerations

Do not simply deploy two Argo installations against the same resources
without understanding ownership and conflict.

Define:

```text
which controller owns which Application
```

Avoid dual writers.

---

# PART LXXIX — DRY RUN / VALIDATION

## 131. Pre-Merge Validation

CI can perform:

```text
YAML validation
Helm template
Kustomize build
policy checks
schema validation
security scans
```

Then:

```text
Git merge
 |
Argo
```

This reduces runtime synchronization failures.

---

# PART LXXX — PR PREVIEW

## 132. Diff Review

A production GitOps PR should show:

```text
old image -> new image
old replicas -> new replicas
configuration changes
RBAC changes
network changes
```

Reviewers should understand the impact before merge.

---

# PART LXXXI — CHANGE RISK

## 133. Risk Classification

Low:

```text
image patch
```

Higher:

```text
RBAC
NetworkPolicy
Ingress
CRD
database configuration
```

Highest:

```text
cluster-wide controllers
security policy
networking
storage
```

Use stronger review for higher-risk changes.

---

# PART LXXXII — CLUSTER BOOTSTRAP REPOSITORY

## 134. Structure

```text
clusters/
 |
prod-a/
 |   bootstrap.yaml
 |   platform/
 |   applications/
 |
prod-b/
 |   bootstrap.yaml
 |   platform/
 |   applications/
```

This makes cluster recovery more systematic.

---

# PART LXXXIII — PLATFORM ADDONS

## 135. Addon Order

Example:

```text
CRDs
 |
controllers
 |
configuration
 |
applications
```

Use sync ordering or separate bootstrap layers where necessary.

---

# PART LXXXIV — CERTIFICATE AND DNS

## 136. Dependencies

Example:

```text
Ingress
 |
Certificate
 |
DNS
 |
Load Balancer
```

Ensure controllers have the required permissions and ordering.

---

# PART LXXXV — STORAGE

## 137. Stateful Applications

GitOps can manage:

```text
StatefulSet
PVC
StorageClass references
```

But persistent data lifecycle is not equivalent to YAML lifecycle.

Be careful with:

```text
deletion
recreation
restore
```

---

# PART LXXXVI — DATABASES

## 138. Database Ownership

A database may be managed by:

```text
Terraform
DB migration tooling
application deployment
```

Do not let Argo and Terraform fight over the same resource.

---

# PART LXXXVII — QUEUES

## 139. Queue Configuration

GitOps can manage Kubernetes workloads that consume queues, while
cloud queue infrastructure may be managed by IaC.

Define ownership clearly.

---

# PART LXXXVIII — SECURITY INCIDENT

## 140. Malicious Git Commit

```text
Detect
 |
freeze promotion
 |
revoke compromised identity
 |
identify affected clusters
 |
restore trusted revision
 |
rotate secrets if needed
 |
audit
```

---

# PART LXXXIX — COMPROMISED ARGO

## 141. Argo Credential Compromise

Actions:

```text
contain
 |
revoke
 |
rotate
 |
restrict cluster access
 |
audit actions
 |
restore trusted configuration
```

Because Argo can control workloads, its credentials are high-value.

---

# PART XC — COMPROMISED CLUSTER

## 142. Cluster Incident

GitOps alone does not clean a compromised cluster.

Possible process:

```text
isolate cluster
 |
preserve evidence
 |
rebuild trusted cluster
 |
bootstrap Argo
 |
reconcile trusted state
 |
restore data
```

---

# PART XCI — PRODUCTION READINESS

## 143. Git

```text
[ ] branch protection
[ ] PR reviews
[ ] ownership
[ ] audit
[ ] backup
```

## 144. Argo

```text
[ ] HA where required
[ ] RBAC
[ ] Projects
[ ] repository access
[ ] cluster access
[ ] monitoring
```

## 145. Kubernetes

```text
[ ] RBAC
[ ] NetworkPolicy
[ ] admission policy
[ ] resources
[ ] probes
[ ] observability
```

---

# PART XCII — PRODUCTION RELEASE CHECKLIST

## 146. Before Merge

```text
[ ] artifact exists
[ ] artifact scanned
[ ] digest verified
[ ] manifests rendered
[ ] policies pass
[ ] dependencies reviewed
[ ] migration reviewed
[ ] rollback considered
```

## 147. After Merge

```text
[ ] Argo detects change
[ ] sync succeeds
[ ] health is healthy
[ ] metrics normal
[ ] logs normal
[ ] business metrics normal
```

---

# PART XCIII — GOLDEN PATH

## 148. Enterprise Golden Path

```text
Developer
 |
Application Repo
 |
PR
 |
CI
 |
Secure Artifact
 |
Promotion PR
 |
GitOps Repo
 |
Argo CD
 |
EKS
 |
Observability
```

Every step is:

```text
auditable
repeatable
automatable
secure
```

---

# PART XCIV — SENIOR INTERVIEW FRAMEWORK

## 149. How to Answer "Design GitOps"

Use:

```text
1. Clarify scale.
2. Clarify environments.
3. Clarify cluster count.
4. Clarify security.
5. Define repository model.
6. Separate CI and CD.
7. Design Argo control plane.
8. Design cluster registration.
9. Design ApplicationSets.
10. Design promotion.
11. Design drift handling.
12. Design security.
13. Design secrets.
14. Design observability.
15. Design HA.
16. Design DR.
17. Design blast-radius controls.
18. Explain trade-offs.
```

---

# PART XCV — SENIOR SCENARIOS

## 150. Scenario: 100 Clusters

Answer:

```text
I would not manually create 100 independent Application definitions.
I would use ApplicationSet or another fleet-management mechanism,
cluster metadata, environment boundaries and progressive deployment
waves. Cluster access would use dedicated least-privilege identities.
```

---

## 151. Scenario: One Bad Commit

```text
Stop propagation
 |
identify revision
 |
revert
 |
sync known-good state
 |
validate
 |
continue through controlled waves
```

---

## 152. Scenario: Manual kubectl Change

```text
Detect drift
 |
determine emergency vs unauthorized change
 |
if valid emergency, document it
 |
commit final desired state
 |
reconcile
```

---

## 153. Scenario: Argo Down

```text
Existing workloads continue based on Kubernetes state.
New desired state is not reconciled until Argo is restored.
The platform has an HA/recovery design and the Git repositories remain
the authoritative desired state.
```

---

## 154. Scenario: Git Down

```text
Existing applications continue.
New desired-state revisions cannot be retrieved.
Restore Git or use the organization's documented continuity procedure.
```

---

## 155. Scenario: Cluster Down

```text
Argo reports the cluster unavailable.
If workloads are expected to fail over, traffic management and the
application DR architecture shift service to the recovery cluster.
Argo then reconciles desired state there.
```

---

## 156. Scenario: Secret Rotation Breaks Application

```text
Check secret-controller status.
Check new secret version.
Check application compatibility.
Restore known-good credential if safe.
Fix rotation process.
Validate before re-enabling automatic progression.
```

---

## 157. Scenario: Deployment Succeeds but Pods Crash

```text
Argo sync success means Kubernetes accepted desired state.
It does not mean the application is healthy.

Investigate:
pods
events
logs
probes
config
secrets
dependencies
resource limits
```

---

# PART XCVI — REAL-WORLD PAYMENT PLATFORM

## 158. Architecture

```text
GitHub/GitLab
 |
CI
 |
Container
 |
Artifact Registry
 |
GitOps PR
 |
Argo
 |
Production EKS
 |
+--> Payment API
+--> Order API
+--> Worker
 |
+--> Redis
+--> Queue
+--> Database
 |
Observability
```

Deployment:

```text
5% canary
 |
health
 |
20%
 |
health
 |
50%
 |
health
 |
100%
```

---

# PART XCVII — MULTI-REGION PAYMENT PLATFORM

## 159. Architecture

```text
Global Traffic
 |
+-------------------+
|                   |
Region A            Region B
|                   |
Argo A              Argo B
|                   |
EKS A               EKS B
|                   |
Apps                Apps
```

Data replication and consistency must be designed separately.

---

# PART XCVIII — PLATFORM ENGINEERING

## 160. GitOps as Platform Capability

Platform team provides:

```text
repository template
deployment template
security policy
observability
Argo Project
cluster registration
```

Developers provide:

```text
application
configuration
service ownership
```

---

# PART XCIX — ARCHITECTURE TRADE-OFFS

## 161. Central Argo vs Distributed Argo

Central:

```text
+ simpler fleet view
+ centralized operations
- larger control-plane blast radius
```

Distributed:

```text
+ failure isolation
+ regional autonomy
- more operational overhead
```

---

## 162. Monorepo vs Multi-Repo

Monorepo:

```text
+ visibility
+ global changes
- large PRs
- ownership complexity
```

Multi-repo:

```text
+ isolation
+ team ownership
- coordination
```

---

# PART C — FINAL PRODUCTION MODEL

## 163. End-to-End Reference

```text
                    DEVELOPER
                        |
                        v
                 APPLICATION GIT
                        |
                        v
                       PR
                        |
                        v
                       CI
             +----------+----------+
             |          |          |
           Tests     Security    Build
             |          |          |
             +----------+----------+
                        |
                        v
                IMMUTABLE ARTIFACT
                        |
                        v
                 ARTIFACT REGISTRY
                        |
                        v
              GITOPS PROMOTION PR
                        |
                        v
                  GITOPS REPO
                        |
                        v
                     ARGO CD
                        |
        +---------------+---------------+
        |               |               |
     Cluster A       Cluster B       Cluster C
        |               |               |
      Canary          Canary          Canary
        |               |               |
      Health          Health          Health
        |               |               |
        +---------------+---------------+
                        |
                        v
                  PRODUCTION
                        |
                        v
              METRICS / LOGS / TRACES
                        |
                        v
                  ALERTING / SRE
```

---

# PART CI — 200 PRODUCTION GOLDEN RULES

## 164. Rules 1–25

```text
1. GitOps is a reconciliation model, not just Git storage.
2. Keep desired state declarative.
3. Version desired state.
4. Review production changes.
5. Protect GitOps repositories.
6. Separate application source from deployment intent where useful.
7. CI builds artifacts.
8. GitOps manages deployment intent.
9. Argo reconciles desired state.
10. Kubernetes runs workloads.
11. Build once.
12. Promote the same artifact.
13. Prefer immutable artifact identities.
14. Prefer image digests for production.
15. Do not use latest as a production release strategy.
16. Treat GitOps repositories as production code.
17. Protect production directories.
18. Use CODEOWNERS where appropriate.
19. Use branch protection.
20. Require meaningful checks.
21. Define repository ownership.
22. Define environment ownership.
23. Define cluster ownership.
24. Define application ownership.
25. Document architecture decisions.
```

## 165. Rules 26–50

```text
26. Use least-privilege Argo access.
27. Separate team permissions.
28. Use AppProjects for logical boundaries.
29. Restrict repositories.
30. Restrict destination clusters.
31. Restrict namespaces.
32. Restrict resource kinds where appropriate.
33. Protect Argo administration.
34. Protect cluster credentials.
35. Protect cloud credentials.
36. Prefer short-lived credentials.
37. Never commit plaintext production secrets.
38. Integrate approved secret management.
39. Rotate credentials.
40. Audit privileged actions.
41. Treat CI as trusted infrastructure.
42. Treat PR code as potentially untrusted.
43. Do not expose production credentials to PR jobs.
44. Protect signing keys.
45. Validate artifact provenance.
46. Scan dependencies.
47. Scan containers.
48. Generate SBOM where required.
49. Enforce approved registries.
50. Use defense in depth.
```

## 166. Rules 51–75

```text
51. Detect drift.
52. Investigate persistent drift.
53. Do not normalize manual production changes.
54. Use break-glass procedures.
55. Commit valid emergency changes back to Git.
56. Use automated sync where risk allows.
57. Use manual sync where stronger control is required.
58. Understand prune before enabling it.
59. Understand self-heal before enabling it.
60. Avoid competing controllers.
61. Define field ownership.
62. Do not let GitOps fight HPA.
63. Do not let Terraform fight Argo.
64. Do not let two Argo installations own the same application.
65. Define sync dependencies.
66. Use sync waves only where necessary.
67. Validate manifests in CI.
68. Render Helm in CI.
69. Build Kustomize overlays in CI.
70. Validate policies before merge.
71. Review diffs.
72. Review database changes.
73. Review RBAC changes.
74. Review network changes.
75. Review cluster-wide resources.
```

## 167. Rules 76–100

```text
76. Use ApplicationSets for appropriate fleets.
77. Label clusters consistently.
78. Avoid uncontrolled fleet-wide propagation.
79. Use deployment waves.
80. Use canaries for risky releases.
81. Monitor canaries.
82. Stop unhealthy progression.
83. Preserve previous artifacts.
84. Test rollback.
85. Remember Git rollback does not reverse data changes.
86. Design database migrations for compatibility.
87. Separate runtime availability from delivery availability.
88. Existing workloads should survive GitOps outages where possible.
89. Existing workloads should survive CI outages where possible.
90. Existing workloads should survive registry outages where possible.
91. Design Argo HA where required.
92. Design Git availability.
93. Design cluster connectivity.
94. Design API access.
95. Monitor reconciliation.
96. Monitor sync failures.
97. Monitor application health.
98. Monitor cluster reachability.
99. Maintain operational runbooks.
100. Test failure scenarios.
```

## 168. Rules 101–125

```text
101. Back up Git.
102. Protect Git history.
103. Back up non-recreatable state.
104. Back up persistent data separately.
105. Test cluster rebuild.
106. Test Argo rebuild.
107. Test GitOps restore.
108. Measure recovery time.
109. Define RTO.
110. Define RPO.
111. Test regional recovery when required.
112. Keep artifacts available during DR.
113. Keep secrets available during DR.
114. Keep DNS available during DR.
115. Keep observability available during DR.
116. Do not assume Git alone provides DR.
117. Do not assume Kubernetes manifests contain data.
118. Do not assume a second region is automatically DR-ready.
119. Document recovery dependencies.
120. Run game days.
121. Record recovery gaps.
122. Fix recurring failure patterns.
123. Keep bootstrap definitions current.
124. Automate cluster registration where appropriate.
125. Standardize cluster configuration.
```

## 169. Rules 126–150

```text
126. Separate infrastructure and application ownership.
127. Use Terraform/IaC for infrastructure where appropriate.
128. Use GitOps for Kubernetes desired state where appropriate.
129. Avoid resource ownership overlap.
130. Provide golden paths.
131. Provide secure defaults.
132. Provide self-service.
133. Reduce developer cognitive load.
134. Standardize labels.
135. Standardize health checks.
136. Standardize resource requests.
137. Standardize security context.
138. Standardize observability.
139. Standardize deployment strategies.
140. Version platform templates.
141. Do not break every application with one template change.
142. Test platform upgrades.
143. Canary platform changes.
144. Monitor controller capacity.
145. Monitor repository rendering.
146. Monitor Kubernetes API load.
147. Monitor Git traffic.
148. Monitor Argo resource usage.
149. Capacity-plan large fleets.
150. Benchmark before scaling architecture.
```

## 170. Rules 151–175

```text
151. Make releases traceable.
152. Link commit to artifact.
153. Link artifact to GitOps revision.
154. Link GitOps revision to cluster.
155. Link deployment to telemetry.
156. Link incident to deployment.
157. Preserve audit evidence.
158. Protect audit data.
159. Use SLOs for critical platform services.
160. Define actionable alerts.
161. Avoid alert fatigue.
162. Maintain runbooks.
163. Define on-call ownership.
164. Practice incident response.
165. Automate safe recovery.
166. Stop bad propagation quickly.
167. Reduce blast radius.
168. Prefer progressive rollout.
169. Separate critical and non-critical applications.
170. Use criticality to drive deployment policy.
171. Protect platform root applications.
172. Protect cluster bootstrap.
173. Protect Argo administration.
174. Test emergency access.
175. Rotate break-glass credentials.
```

## 171. Rules 176–200

```text
176. Use explicit environment boundaries.
177. Use explicit cluster boundaries.
178. Use explicit account boundaries.
179. Use explicit regional boundaries.
180. Use network segmentation.
181. Use Kubernetes RBAC.
182. Use NetworkPolicy.
183. Use admission policy.
184. Use workload identity.
185. Restrict egress where appropriate.
186. Protect ingress.
187. Encrypt traffic.
188. Encrypt sensitive data.
189. Monitor certificate renewal.
190. Monitor secret rotation.
191. Keep deployment systems out of the application request path.
192. Make desired state recoverable.
193. Make deployments repeatable.
194. Make rollback understandable.
195. Make failures observable.
196. Make recovery measurable.
197. Make ownership explicit.
198. Test assumptions.
199. Prefer simple designs when requirements permit.
200. The goal of GitOps is not merely automated deployment; it is a
     controlled, auditable, recoverable and continuously reconciled
     production operating model.
```
---