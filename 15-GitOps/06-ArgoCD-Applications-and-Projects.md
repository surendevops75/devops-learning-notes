# ArgoCD-Applications-and-Projects

## 1. Purpose

This file explains the two most important declarative objects used to organize Argo CD deployments:

```text
Application
AppProject
```

An Argo CD Application defines:

```text
What should be deployed?
From where?
To which cluster?
To which namespace?
How should it synchronize?
```

An AppProject defines the security and organizational boundary around Applications:

```text
Which Git repositories are allowed?
Which clusters are allowed?
Which namespaces are allowed?
Which Kubernetes resources are allowed?
```

For production GitOps, understanding Applications and Projects is essential because they connect:

```text
Git
 |
 v
Argo CD
 |
 +--> AppProject security boundary
 |
 +--> Application desired state
 |
 v
Kubernetes cluster
```

This file covers the Application and AppProject APIs deeply, including Helm, Kustomize, sync policies, lifecycle behavior, production patterns, RoboShop examples, troubleshooting, and interview scenarios.

---

# 2. Application Mental Model

An Argo CD Application is a declarative description of a deployment relationship.

Think:

```text
Application
    |
    +--> Source
    |      |
    |      +--> Git repository
    |      +--> revision
    |      +--> path
    |      +--> Helm/Kustomize/plain YAML
    |
    +--> Destination
    |      |
    |      +--> cluster
    |      +--> namespace
    |
    +--> Project
    |      |
    |      +--> security boundary
    |
    +--> Sync policy
           |
           +--> manual
           +--> automated
           +--> prune
           +--> selfHeal
```

---

# 3. Application CRD

The core resource is:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
```

This is a Kubernetes Custom Resource.

The object is normally stored in the Argo CD namespace:

```yaml
metadata:
  namespace: argocd
```

The workload itself does not have to run in the `argocd` namespace.

This distinction is critical.

---

# 4. Control Plane vs Workload Namespace

Example:

```text
argocd namespace
 |
 +--> Application object
 +--> AppProject
 +--> Argo CD components

roboshop namespace
 |
 +--> Deployment
 +--> Service
 +--> Ingress
 +--> HPA
```

The Application object describes the workload.

The workload resources are created in the destination namespace.

---

# 5. Complete Application Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application

metadata:
  name: roboshop-cart-prod
  namespace: argocd

spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/example/roboshop-gitops.git
    targetRevision: main
    path: helm/roboshop

    helm:
      releaseName: cart
      valueFiles:
        - values/prod.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true

    syncOptions:
      - CreateNamespace=true

    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

This example is intentionally production-oriented.

---

# 6. apiVersion

```yaml
apiVersion: argoproj.io/v1alpha1
```

This identifies the Argo CD API group and version.

It tells Kubernetes:

```text
This resource belongs to Argo CD.
```

The Application CRD must already exist.

Verify:

```bash
kubectl get crd applications.argoproj.io
```

---

# 7. kind

```yaml
kind: Application
```

This tells Kubernetes which Argo CD custom resource is being created.

Other important Argo CD resources include:

```text
Application
ApplicationSet
AppProject
```

---

# 8. metadata

Example:

```yaml
metadata:
  name: roboshop-cart-prod
  namespace: argocd
```

Important metadata fields include:

```text
name
namespace
labels
annotations
finalizers
```

---

# 9. Application Naming

Application names should clearly communicate ownership and environment.

Good:

```text
roboshop-cart-dev
roboshop-cart-qa
roboshop-cart-prod
```

For multi-cluster environments:

```text
roboshop-cart-prod-ap-south-1
```

Avoid ambiguous names such as:

```text
app1
test
demo
```

Production names should be searchable and understandable during incidents.

---

# 10. Application Labels

Labels can support:

```text
Environment
Team
Business unit
Application
Cluster
Cost center
```

Example:

```yaml
metadata:
  labels:
    app.kubernetes.io/part-of: roboshop
    environment: prod
    team: platform
```

Labels are useful for operational filtering and automation.

---

# 11. Application Annotations

Annotations can provide metadata used by Argo CD or surrounding tooling.

Example:

```yaml
metadata:
  annotations:
    notifications.argoproj.io/subscribe.on-sync-failed.slack: platform-alerts
```

The exact notification configuration depends on the installed notification system.

Do not add annotations without understanding their controller behavior.

---

# 12. Finalizers

Applications may use an Argo CD resources finalizer.

Example:

```yaml
metadata:
  finalizers:
    - resources-finalizer.argocd.argoproj.io
```

The finalizer affects Application deletion behavior.

---

# 13. Why Finalizers Matter

Without understanding finalizers, deleting an Application can be surprising.

With a resources finalizer, Argo CD can clean up resources owned by the Application according to its deletion behavior.

Conceptually:

```text
Delete Application
       |
       v
Finalizer executes
       |
       v
Application-owned resources cleaned up
       |
       v
Application removed
```

This is why Application deletion must be treated as a potentially destructive operation.

---

# 14. Application Project

Example:

```yaml
spec:
  project: roboshop-prod
```

The Project determines what this Application is permitted to do.

Think:

```text
Application
      |
      v
AppProject
      |
      +--> Allowed repositories
      +--> Allowed destinations
      +--> Resource permissions
```

---

# 15. Why Projects Matter

Without proper Project boundaries, a deployment definition can potentially have broader reach than intended.

For example:

```text
Application
 |
 +--> arbitrary Git repository
 |
 +--> production cluster
 |
 +--> arbitrary namespace
 |
 +--> cluster-scoped resources
```

Projects reduce this blast radius.

---

# 16. Project as a Security Boundary

Example:

```text
roboshop-prod Project
 |
 +--> GitOps repository
 |
 +--> prod EKS cluster
 |
 +--> roboshop namespace
 |
 +--> approved resource kinds
```

This creates a controlled deployment boundary.

---

# 17. Source

The Application source tells Argo CD where desired state comes from.

Example:

```yaml
source:
  repoURL: https://github.com/example/roboshop-gitops.git
  targetRevision: main
  path: environments/prod/cart
```

Important fields include:

```text
repoURL
targetRevision
path
Helm
Kustomize
directory
plugin
```

---

# 18. repoURL

Example:

```yaml
repoURL: https://github.com/example/roboshop-gitops.git
```

This identifies the repository.

It can point to:

```text
GitHub
GitLab
Bitbucket
Enterprise Git
Other supported Git repositories
```

The repository must be configured and accessible by Argo CD.

---

# 19. Private Repository

For a private repository:

```text
Application
    |
    v
Repo Server
    |
    v
Repository credentials
    |
    v
Private Git repository
```

The Application itself should not contain:

```text
Git password
PAT
SSH private key
```

Those credentials belong in Argo CD's repository configuration.

---

# 20. targetRevision

Example:

```yaml
targetRevision: main
```

This tells Argo CD which Git revision to track.

Possible values can include:

```text
branch
tag
commit SHA
```

For production, the choice should be deliberate.

---

# 21. Tracking a Branch

Example:

```yaml
targetRevision: main
```

Advantages:

```text
Simple
Continuous updates
Easy automation
```

Risk:

```text
Any approved change to main can affect the Application.
```

This is acceptable when branch protection and release controls are strong.

---

# 22. Tracking a Tag

Example:

```yaml
targetRevision: v1.4.0
```

This can provide explicit release boundaries.

Conceptually:

```text
Git commit
   |
   v
Release tag
   |
   v
Argo CD
```

This can be useful for controlled promotion.

---

# 23. Tracking a Commit SHA

Example:

```yaml
targetRevision: 3f4e7a9...
```

This provides a highly specific immutable revision.

It can be useful for:

```text
Reproducibility
Auditing
Emergency pinning
Release verification
```

But operational workflows must still make future updates manageable.

---

# 24. Production Recommendation

There is no single universal answer.

For example:

```text
Development -> branch
QA          -> controlled branch/tag
Production  -> controlled release/tag or protected branch
```

The important requirement is:

```text
Controlled promotion
+
Traceability
+
Review
```

---

# 25. path

Example:

```yaml
path: environments/prod/cart
```

The path tells Argo CD where manifests or the Helm/Kustomize configuration are located.

Example repository:

```text
gitops-repo/
└── environments/
    └── prod/
        └── cart/
            ├── deployment.yaml
            ├── service.yaml
            └── ingress.yaml
```

---

# 26. Directory Source

If the path contains ordinary Kubernetes YAML:

```text
deployment.yaml
service.yaml
configmap.yaml
```

Argo CD can render the directory as Kubernetes manifests.

Example:

```yaml
source:
  repoURL: https://github.com/example/gitops.git
  targetRevision: main
  path: manifests/cart
```

---

# 27. Plain YAML Workflow

```text
Git
 |
 v
manifests/cart/
 |
 +--> deployment.yaml
 +--> service.yaml
 +--> configmap.yaml
 |
 v
Repo Server
 |
 v
Rendered manifests
 |
 v
Application Controller
 |
 v
Kubernetes
```

---

# 28. Helm Source

Example:

```yaml
source:
  repoURL: https://github.com/example/gitops.git
  targetRevision: main
  path: helm/roboshop

  helm:
    releaseName: cart
    valueFiles:
      - values/prod.yaml
```

Argo CD renders the Helm chart and then reconciles the resulting Kubernetes manifests.

---

# 29. Helm Is Not the Deployment Controller

A common interview misconception:

> Does Argo CD run `helm install` like a normal Helm deployment?

The important mental model is:

```text
Helm
  |
  v
Render templates
  |
  v
Kubernetes manifests
  |
  v
Argo CD reconciliation
```

Argo CD uses Helm as a manifest-generation mechanism for Helm sources.

---

# 30. releaseName

Example:

```yaml
helm:
  releaseName: cart
```

This controls the Helm release name used during rendering.

Use meaningful names:

```text
cart
user
payment
inventory
```

Be consistent across environments.

---

# 31. valueFiles

Example:

```yaml
helm:
  valueFiles:
    - values/prod.yaml
```

Repository:

```text
helm/roboshop/
├── values.yaml
└── values/
    ├── dev.yaml
    ├── qa.yaml
    └── prod.yaml
```

This supports environment-specific configuration.

---

# 32. Multiple Value Files

Example:

```yaml
helm:
  valueFiles:
    - values/common.yaml
    - values/prod.yaml
```

The final values are composed according to Helm's value precedence rules.

Be careful with overlapping keys.

Make the hierarchy understandable.

---

# 33. Helm Parameters

Example:

```yaml
helm:
  parameters:
    - name: image.tag
      value: "2026.08.19-abc123"
```

This can be useful for CI-driven image updates.

However, avoid creating a deployment process where important configuration is hidden in CLI-generated Application mutations.

Git should remain the authoritative record.

---

# 34. Helm Values Inline

Argo CD can also support values embedded in the Application definition.

Conceptually:

```yaml
helm:
  values: |
    replicaCount: 3
    image:
      tag: "1.4.7"
```

This is convenient but can make the Application object very large.

For maintainability, repository-managed values files are often preferable.

---

# 35. Kustomize Source

Kustomize can be selected by directory structure and configuration.

Example:

```text
kustomize/
├── base/
└── overlays/
    ├── dev/
    ├── qa/
    └── prod/
```

Application:

```yaml
source:
  repoURL: https://github.com/example/gitops.git
  targetRevision: main
  path: kustomize/overlays/prod
```

---

# 36. Helm vs Kustomize

### Helm

Good for:

```text
Reusable application packages
Parameterized deployments
Versioned charts
Many services
```

### Kustomize

Good for:

```text
Overlay-based configuration
Native Kubernetes YAML
Environment patches
Minimal templating
```

Choose based on team standards and workload complexity.

---

# 37. Multiple Sources

Modern Argo CD versions support multiple sources for an Application.

This can be useful when desired state is composed from more than one repository/source.

Example conceptual architecture:

```text
Application
 |
 +--> Application manifests
 |
 +--> Values repository
 |
 v
Rendered desired state
```

Use multiple sources carefully.

Overusing them can make dependency relationships harder to understand.

---

# 38. Destination

Example:

```yaml
destination:
  server: https://kubernetes.default.svc
  namespace: roboshop
```

Destination answers:

```text
Which cluster?
Which namespace?
```

---

# 39. Destination Server

For the same cluster hosting Argo CD:

```yaml
server: https://kubernetes.default.svc
```

For another registered cluster, the destination references that cluster's server/registered identity.

Example conceptual value:

```text
https://ABC123.eks.amazonaws.com
```

The actual registered cluster information should be obtained from Argo CD.

---

# 40. Destination Name

Argo CD can also target a registered cluster using a cluster name in supported configurations.

Conceptually:

```yaml
destination:
  name: eks-prod
  namespace: roboshop
```

This is especially useful for multi-cluster ApplicationSets.

---

# 41. Destination Namespace

Example:

```yaml
namespace: roboshop
```

This is where namespaced resources are deployed.

Example:

```text
Deployment/cart
Service/cart
ConfigMap/cart
```

will normally live under:

```text
roboshop
```

---

# 42. Namespace Ownership

Decide who owns the namespace.

Option A:

```text
Platform GitOps
 |
 v
Namespace
```

Option B:

```text
Application
 |
 +--> CreateNamespace=true
```

Avoid having both independently manage the same Namespace object.

---

# 43. syncPolicy

Example:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

This controls synchronization behavior.

Important areas include:

```text
automated
prune
selfHeal
syncOptions
retry
```

---

# 44. Manual Sync

If automated synchronization is absent:

```yaml
syncPolicy:
```

or no automated section is configured.

Then an operator can use:

```bash
argocd app sync roboshop-cart-prod
```

Manual sync is often used where explicit production approval is required.

---

# 45. Automated Sync

Example:

```yaml
syncPolicy:
  automated: {}
```

Argo CD can automatically synchronize changes.

This is useful for:

```text
Development
Low-risk environments
Continuous deployment
```

Production automation should be governed by release controls.

---

# 46. Automated Sync with Prune

```yaml
automated:
  prune: true
```

Prune means Argo CD can remove resources that are no longer part of the desired state.

Example:

```text
Git
 |
 | Deployment exists
 v
Kubernetes

Git commit removes Deployment
 |
 v
Argo CD
 |
 v
Prune
 |
 v
Deployment removed
```

Prune is powerful and potentially destructive.

---

# 47. Self-Heal

Example:

```yaml
automated:
  selfHeal: true
```

If someone changes a managed resource directly:

```bash
kubectl edit deployment cart -n roboshop
```

Argo CD can detect the drift and restore the Git-defined desired state.

Conceptually:

```text
Git desired state
       |
       v
Argo CD
       |
       v
Kubernetes actual state

Manual kubectl change
       |
       v
Drift
       |
       v
Self-heal
       |
       v
Desired state restored
```

---

# 48. Prune vs Self-Heal

They solve different problems.

### Prune

```text
Resource exists in cluster
but no longer exists in desired state.
```

### Self-heal

```text
Resource exists in both
but live configuration differs.
```

Remember:

```text
Prune = lifecycle cleanup
Self-heal = drift correction
```

---

# 49. CreateNamespace

Example:

```yaml
syncOptions:
  - CreateNamespace=true
```

This tells Argo CD to create the destination namespace if it does not exist.

Use it carefully.

---

# 50. Server-Side Apply

Modern Kubernetes environments may benefit from server-side apply in appropriate scenarios.

Argo CD provides sync options for different Kubernetes apply behaviors.

Before enabling them broadly:

```text
Understand field ownership
Understand managedFields
Test CRDs
Test operators
```

Do not enable every sync option simply because it exists.

---

# 51. Replace

Argo CD provides a replace-style sync option.

This can have stronger behavior than normal apply and can be destructive for certain resources.

Production rule:

```text
Do not enable Replace globally without testing.
```

Use it only when the workload specifically requires it.

---

# 52. Force

Force-style synchronization can delete/recreate resources.

This is particularly risky.

Use only when:

```text
The resource explicitly requires it
The impact is understood
The runbook documents it
```

---

# 53. Respect Ignore Differences

Argo CD can be configured to ignore specific fields when another controller is expected to mutate them.

Example conceptual case:

```text
Argo CD desired state
        |
        v
Kubernetes operator
        |
        v
Controlled field mutation
```

Without appropriate configuration:

```text
Operator changes field
       |
       v
Argo CD sees drift
       |
       v
Repeated correction
```

Ignoring differences should be narrowly scoped.

---

# 54. Retry

Example:

```yaml
retry:
  limit: 5
  backoff:
    duration: 5s
    factor: 2
    maxDuration: 3m
```

Retry is useful for transient failures such as:

```text
Temporary API issue
Transient dependency
Repository availability
Short-lived admission/controller condition
```

Retry does not fix a permanent configuration error.

---

# 55. Retry Backoff

Example:

```text
Attempt 1 -> 5s
Attempt 2 -> 10s
Attempt 3 -> 20s
Attempt 4 -> 40s
...
```

The maximum duration limits the backoff.

This reduces aggressive retry loops.

---

# 56. Sync Options

Common options may address:

```text
Namespace creation
Apply strategy
Pruning behavior
Validation
Selective synchronization
```

Only enable options that solve a known requirement.

Production configuration should be intentionally minimal.

---

# 57. Sync Windows

Argo CD can support sync windows to restrict when Applications can synchronize.

Example concept:

```text
Production
 |
 +--> No sync during release freeze
 |
 +--> Allowed during release window
```

This is useful for:

```text
Change freezes
Business-critical periods
Maintenance windows
Controlled release processes
```

---

# 58. Sync Waves

Resources can be ordered using sync waves.

Concept:

```text
Wave -1
  |
  v
Namespace / prerequisites

Wave 0
  |
  v
Config / platform resources

Wave 1
  |
  v
Application Deployment

Wave 2
  |
  v
Dependent resources
```

Detailed ordering is covered in Advanced Argo CD.

---

# 59. Hooks

Argo CD supports lifecycle hooks.

Conceptually:

```text
PreSync
   |
   v
Sync
   |
   v
PostSync
```

Hooks can be used for controlled lifecycle operations.

Do not use hooks as a substitute for a clean Kubernetes architecture.

---

# 60. Application Status

An Application has two major operational dimensions:

```text
Sync status
Health status
```

They answer different questions.

---

# 61. Sync Status

Typical states:

```text
Synced
OutOfSync
Unknown
```

### Synced

Live state matches desired state.

### OutOfSync

Live state differs from desired state.

### Unknown

Argo CD cannot determine the state reliably.

---

# 62. Health Status

Typical health states include:

```text
Healthy
Progressing
Degraded
Suspended
Missing
Unknown
```

Health describes workload runtime condition.

---

# 63. Synced Does Not Mean Healthy

This is a very important interview point.

Example:

```text
Git desired state
       |
       v
Argo CD applies it
       |
       v
Sync = Synced

But:
Deployment has crash-looping Pods

Health = Degraded
```

Therefore:

```text
Synced != Healthy
```

---

# 64. Healthy Does Not Mean Synced

A resource could happen to be running correctly even though the desired configuration differs.

Therefore evaluate both:

```text
Sync
+
Health
```

---

# 65. Application Tree

The Argo CD UI can show the resource hierarchy.

Conceptually:

```text
Application
 |
 +--> Deployment
 |      |
 |      +--> ReplicaSet
 |              |
 |              +--> Pod
 |
 +--> Service
 |
 +--> ConfigMap
 |
 +--> Ingress
```

This is extremely useful during troubleshooting.

---

# 66. Application Conditions

Application conditions provide additional information about failures.

Examples may include:

```text
ComparisonError
SyncError
InvalidSpec
SharedResourceWarning
```

Use:

```bash
argocd app get <app>
```

to inspect current status.

---

# 67. Application Events

Kubernetes events can help:

```bash
kubectl get events -n argocd
```

For workload namespace:

```bash
kubectl get events -n roboshop
```

Events often reveal:

```text
Scheduling
Image pull
Admission
RBAC
Probe failures
Mount failures
```

---

# 68. AppProject Deep Dive

An AppProject is:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
```

It groups Applications and defines allowed deployment boundaries.

---

# 69. Project Source Repositories

Example:

```yaml
sourceRepos:
  - https://github.com/example/roboshop-gitops.git
```

This prevents an Application in the Project from sourcing from unauthorized repositories.

---

# 70. Wildcard Repository

A wildcard can allow broad sources:

```yaml
sourceRepos:
  - '*'
```

This is dangerous for production.

Prefer explicit repository URLs whenever possible.

---

# 71. Project Destinations

Example:

```yaml
destinations:
  - server: https://kubernetes.default.svc
    namespace: roboshop
```

This restricts where Applications in the Project can deploy.

---

# 72. Destination Wildcards

Avoid broad patterns such as:

```yaml
destinations:
  - server: '*'
    namespace: '*'
```

unless there is a very deliberate platform-level reason.

This can effectively remove important isolation.

---

# 73. Cluster Resource Whitelist

Example:

```yaml
clusterResourceWhitelist:
  - group: ""
    kind: Namespace
```

This controls which cluster-scoped resource kinds an Application can manage.

---

# 74. Namespace Resource Whitelist

Example:

```yaml
namespaceResourceWhitelist:
  - group: apps
    kind: Deployment
  - group: ""
    kind: Service
```

This allows specific namespaced resource kinds.

---

# 75. Resource Blacklists

A Project can also explicitly deny certain resource types.

This is useful where:

```text
Most resources are allowed
but specific dangerous kinds are denied.
```

A least-privilege design should still prefer precise permissions.

---

# 76. Cluster-Scoped Resource Risk

Cluster-scoped resources can affect the entire cluster.

Examples:

```text
ClusterRole
ClusterRoleBinding
CRD
Namespace
```

Grant these permissions only to trusted platform Applications.

---

# 77. Application vs Platform Projects

A mature environment may use:

```text
platform-project
 |
 +--> ingress controller
 +--> monitoring
 +--> namespaces
 +--> policies

application-project
 |
 +--> RoboShop
 +--> business services
```

This separates:

```text
Platform ownership
Application ownership
```

---

# 78. Production Project Structure

Example:

```text
projects/
├── platform.yaml
├── roboshop-dev.yaml
├── roboshop-qa.yaml
└── roboshop-prod.yaml
```

This makes environment boundaries explicit.

---

# 79. Production AppProject

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: roboshop-prod
  namespace: argocd

spec:
  description: RoboShop production deployment boundary

  sourceRepos:
    - https://github.com/example/roboshop-gitops.git

  destinations:
    - server: https://prod-cluster.example.internal
      namespace: roboshop

  clusterResourceWhitelist: []

  namespaceResourceWhitelist:
    - group: apps
      kind: Deployment
    - group: ""
      kind: Service
    - group: ""
      kind: ConfigMap
    - group: autoscaling
      kind: HorizontalPodAutoscaler
    - group: networking.k8s.io
      kind: Ingress
```

This intentionally does not allow arbitrary cluster-scoped resources.

---

# 80. Why Separate Prod Project?

A production Project can provide:

```text
Different repository policy
Different cluster
Different namespace
Different resource permissions
Different approval model
Different RBAC
```

This creates a strong environment boundary.

---

# 81. Development Project

Development may allow broader experimentation:

```text
roboshop-dev
 |
 +--> dev cluster
 +--> dev namespace
 +--> development repository/branch
```

Production should not inherit these permissions automatically.

---

# 82. QA Project

QA may represent:

```text
Controlled integration testing
Release candidate validation
Automated regression
```

It can use:

```text
QA cluster
QA namespace
Approved Git revision
```

---

# 83. Production Project

Production should be the most restricted:

```text
Protected Git source
Protected cluster
Protected namespace
Limited resource types
Limited operators
Restricted sync rights
```

---

# 84. RBAC Relationship

The access model becomes:

```text
User
 |
 v
SSO Group
 |
 v
Argo CD RBAC
 |
 v
AppProject
 |
 v
Application
 |
 v
Kubernetes
```

Every layer should enforce least privilege.

---

# 85. Developer Role Example

A developer may need:

```text
View dev Applications
View health
View logs through approved tooling
Sync dev
```

They may not need:

```text
Manage production Project
Register clusters
Manage repositories
Modify RBAC
```

---

# 86. Release Manager Role

A release manager may have:

```text
View all applications
Sync approved QA/prod Applications
Inspect history
Trigger rollback
```

But should not necessarily be allowed to:

```text
Modify cluster credentials
Change Projects
Change global RBAC
```

---

# 87. Platform Administrator

A platform administrator may manage:

```text
Projects
Repositories
Clusters
RBAC
Argo CD configuration
Upgrades
```

This is a higher-risk role.

---

# 88. Application Ownership

Use Git ownership and CODEOWNERS to control who can modify:

```text
prod/
applications/
projects/
```

Example conceptual ownership:

```text
/platform/    -> Platform team
/environments/prod/ -> Application owners + platform approval
```

This complements Argo CD RBAC.

---

# 89. Production Approval Flow

A strong workflow can be:

```text
Developer
   |
   v
Pull Request
   |
   v
CI
   |
   +--> Unit tests
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
Review
   |
   v
Merge
   |
   v
GitOps production revision
   |
   v
Argo CD
```

Argo CD should not become a bypass around Git review controls.

---

# 90. RoboShop Application Model

RoboShop services may include:

```text
frontend
catalogue
cart
user
shipping
payment
dispatch
notification
```

The exact services depend on the application version.

Each service can be represented by an Application:

```text
roboshop-frontend
roboshop-catalogue
roboshop-cart
roboshop-user
roboshop-payment
```

---

# 91. RoboShop Helm Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-cart-prod
  namespace: argocd

spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/example/roboshop-gitops.git
    targetRevision: main
    path: helm/cart

    helm:
      releaseName: cart
      valueFiles:
        - values/prod.yaml

  destination:
    name: eks-prod
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true

    syncOptions:
      - CreateNamespace=true

    retry:
      limit: 5
      backoff:
        duration: 10s
        factor: 2
        maxDuration: 5m
```

---

# 92. RoboShop GitOps Repository

A practical structure:

```text
roboshop-gitops/
│
├── applications/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
├── projects/
│   ├── dev.yaml
│   ├── qa.yaml
│   └── prod.yaml
│
├── helm/
│   ├── cart/
│   ├── catalogue/
│   ├── frontend/
│   ├── payment/
│   └── user/
│
└── platform/
    ├── namespaces/
    ├── ingress/
    └── policies/
```

---

# 93. One Application per Microservice

Advantages:

```text
Independent sync
Independent health
Independent rollback
Clear ownership
Clear history
```

Example:

```text
RoboShop
 |
 +--> cart Application
 +--> user Application
 +--> payment Application
 +--> catalogue Application
```

This is often useful for microservices.

---

# 94. One Application for Entire Platform

Another approach:

```text
roboshop Application
 |
 +--> all services
```

Advantages:

```text
Simple bootstrap
Single status
Simple initial setup
```

Disadvantages:

```text
Large blast radius
Less independent lifecycle
Harder ownership
```

Choose based on platform scale.

---

# 95. Application Granularity

Do not blindly create:

```text
1 Application = 1 YAML file
```

Instead decide based on:

```text
Deployment lifecycle
Ownership
Dependencies
Release frequency
Security boundary
Rollback needs
```

---

# 96. Shared Resources

Be careful if two Applications manage the same resource.

Example:

```text
Application A -> ConfigMap/shared
Application B -> ConfigMap/shared
```

This can create ownership conflicts.

Prefer one clear owner.

---

# 97. Shared Resource Warning

Argo CD can identify situations where multiple Applications attempt to manage the same resource.

Use:

```bash
argocd app get <application>
```

and inspect warnings/conditions.

Fix ownership rather than repeatedly forcing synchronization.

---

# 98. Application Deletion

Deleting an Application is not equivalent to merely hiding it.

Depending on finalizers and deletion settings:

```text
Application deleted
       |
       v
Managed resources may be deleted
```

Always inspect finalizers and resource ownership before production deletion.

---

# 99. Safe Application Deletion Procedure

Before deleting:

```bash
argocd app get <app>
```

Check:

```text
Resources
Project
Destination
Finalizers
Sync policy
```

Then determine whether resources should remain.

For production, use a documented deletion procedure.

---

# 100. Application Disable vs Delete

If the objective is:

```text
Stop automated deployment
```

you may not need to delete the Application.

Instead adjust:

```text
Automated sync
Sync windows
Git revision
```

Deletion is a much stronger operation.

---

# 101. Application History

Use:

```bash
argocd app history roboshop-cart-prod
```

This helps answer:

```text
Which revision was deployed?
When?
What source?
```

During an incident:

```text
Current revision
     |
     v
Previous known-good revision
```

can be identified.

---

# 102. Application Diff

Always use:

```bash
argocd app diff roboshop-cart-prod
```

before significant manual synchronization.

It helps identify:

```text
Image changes
Replica changes
Environment changes
Ingress changes
RBAC changes
Resource deletion
```

---

# 103. Application Sync

Manual:

```bash
argocd app sync roboshop-cart-prod
```

Then:

```bash
argocd app get roboshop-cart-prod
```

Check:

```text
Sync status
Health
Conditions
Resource tree
```

---

# 104. Selective Sync

For some operational cases, Argo CD supports syncing selected resources.

Use carefully.

Selective deployment can be useful for troubleshooting but may bypass dependency assumptions.

Production runbooks should specify when it is allowed.

---

# 105. Sync Failure Analysis

When sync fails:

```text
Application
   |
   v
argocd app get
   |
   +--> Sync operation
   +--> Conditions
   +--> Resource
   |
   v
kubectl describe
   |
   v
Events
```

Do not retry blindly.

First determine whether the error is:

```text
Configuration
Authentication
Authorization
Dependency
Admission
Runtime
```

---

# 106. InvalidSpecError

Common causes:

```text
Wrong repository URL
Wrong path
Invalid Helm values
Invalid destination
Invalid Project
Unsupported source configuration
```

Start with:

```bash
argocd app get <app>
```

and inspect conditions.

---

# 107. ComparisonError

Comparison errors often indicate Argo CD cannot correctly calculate desired state or compare it with live state.

Investigate:

```text
Repo Server
Kubernetes API
CRDs
Manifest generation
Resource schema
Permissions
```

---

# 108. SyncError

A SyncError means Argo CD attempted an operation and it failed.

Possible causes:

```text
RBAC
Admission webhook
Invalid manifest
Resource conflict
API timeout
Immutable field
Missing dependency
```

Check the affected resource first.

---

# 109. Health Degraded

If:

```text
Sync = Synced
Health = Degraded
```

then Git configuration may be correct while the workload is unhealthy.

Check:

```bash
kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
kubectl get events -n roboshop
```

---

# 110. Health Progressing

`Progressing` can be normal during deployment.

Investigate if it remains indefinitely.

Possible causes:

```text
Pods not becoming Ready
Deployment rollout stuck
Image pull issue
Probe failure
Scheduling failure
Dependency unavailable
```

---

# 111. Resource Missing

If Argo CD reports a resource as missing:

```text
Desired state says resource exists
Live cluster says resource does not exist
```

Possible causes:

```text
Sync not executed
Resource was manually deleted
Admission/controller deleted it
Wrong namespace
Wrong cluster
```

---

# 112. Resource Orphaned

An orphaned resource exists in the cluster but is not associated with the expected Application.

Investigate:

```text
Ownership labels
Application tracking
Duplicate Applications
Manual resource creation
```

---

# 113. Tracking Method

Argo CD needs to determine which resources belong to which Applications.

Resource tracking can involve labels/annotations depending on configuration.

The key principle:

```text
Application
      |
      v
Tracking metadata
      |
      v
Managed Kubernetes resources
```

This becomes important with shared resources and generated resources.

---

# 114. Ignore Differences

Suppose an operator modifies:

```yaml
spec.replicas
```

or another controlled field.

If Argo CD constantly reports drift because another controller owns that field, carefully configure ignore differences.

Do not ignore entire resources.

Prefer:

```text
Specific resource
+
Specific field
+
Known controller
```

---

# 115. Project Role

AppProjects can also define project-level roles in supported configurations.

Conceptually:

```text
Project
 |
 +--> role:developer
 +--> role:release-manager
```

This allows scoped access to Applications inside the Project.

---

# 116. Project Role Token

Project roles can support automation identities.

Example use:

```text
External release automation
        |
        v
Argo CD API
        |
        v
Specific Project
```

The token should have the minimum required permissions.

---

# 117. Automation Identity

Do not use:

```text
admin token
```

for every pipeline.

Prefer:

```text
project-scoped automation identity
```

where the workflow requires Argo CD API interaction.

Even better, avoid direct API-triggered deployment when Git changes can naturally trigger reconciliation.

---

# 118. CI Should Prefer Git Changes

Instead of:

```text
CI
 |
 +--> argocd app sync
```

prefer:

```text
CI
 |
 v
GitOps commit
 |
 v
Argo CD detects revision
 |
 v
Sync
```

This preserves the GitOps audit trail.

---

# 119. When Direct Sync Can Be Useful

Direct sync may still be useful for:

```text
Emergency operations
Controlled release orchestration
Testing
Manual production runbooks
```

But it should not become the normal deployment path if Git is intended to remain authoritative.

---

# 120. ApplicationSet Preview

ApplicationSet is covered deeply later, but understand the relationship:

```text
ApplicationSet
      |
      v
Generates Applications
      |
      +--> dev
      +--> qa
      +--> prod
```

Therefore:

```text
ApplicationSet = Application factory
Application = individual deployment
```

---

# 121. App of Apps Preview

App of Apps uses an Application to create/manage other Applications.

Conceptually:

```text
platform-root
      |
      +--> cart Application
      +--> user Application
      +--> payment Application
      +--> frontend Application
```

This pattern is covered later in detail.

---

# 122. Production Pattern: Project + ApplicationSet

A strong structure is:

```text
Project
 |
 v
Security boundary

ApplicationSet
 |
 v
Generate Applications

Applications
 |
 v
Deploy workloads
```

This separates:

```text
Security
Generation
Deployment
```

---

# 123. Production Pattern: Environment Isolation

```text
roboshop-dev Project
       |
       v
dev cluster
       |
       v
dev namespace

roboshop-qa Project
       |
       v
qa cluster
       |
       v
qa namespace

roboshop-prod Project
       |
       v
prod cluster
       |
       v
prod namespace
```

This is easier to reason about than one unrestricted Project.

---

# 124. Production Pattern: Cluster Isolation

For highly sensitive production environments:

```text
Argo CD Management
 |
 +--> Dev
 |
 +--> QA
 |
 +--> Prod
```

But Project and RBAC restrictions must still prevent:

```text
Dev operator
   |
   X
Prod deployment
```

---

# 125. Production Pattern: Separate Argo CD

Some organizations choose:

```text
Argo CD Dev
Argo CD NonProd
Argo CD Prod
```

Reasons:

```text
Strong isolation
Reduced blast radius
Different security boundary
Independent upgrades
Regulatory requirements
```

Centralized multi-cluster Argo CD is not always the best answer.

The correct architecture depends on organizational risk.

---

# 126. Application Security Checklist

For each Application:

```text
[ ] Correct Project
[ ] Approved repoURL
[ ] Approved targetRevision
[ ] Correct path
[ ] Correct destination cluster
[ ] Correct namespace
[ ] Appropriate sync policy
[ ] Prune intentionally enabled/disabled
[ ] Self-heal intentionally enabled/disabled
[ ] Retry intentionally configured
[ ] No plaintext secrets
[ ] No unauthorized cluster resources
```

---

# 127. Production Application YAML Review

Before merge, reviewers should inspect:

```yaml
project:
source:
  repoURL:
  targetRevision:
  path:
destination:
  server/name:
  namespace:
syncPolicy:
```

These fields determine deployment reach and behavior.

---

# 128. Production Project YAML Review

Review:

```yaml
sourceRepos:
destinations:
clusterResourceWhitelist:
namespaceResourceWhitelist:
roles:
```

Questions:

```text
Can this Project deploy to prod?
Can it deploy outside its namespace?
Can it create cluster-scoped resources?
Can it access an unauthorized repository?
```

---

# 129. Application Troubleshooting Commands

```bash
argocd app get <app>
argocd app diff <app>
argocd app history <app>
argocd app sync <app>
argocd app list
```

Kubernetes:

```bash
kubectl get application <app> -n argocd -o yaml
kubectl describe application <app> -n argocd
kubectl get events -n argocd
```

Workload:

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
```

---

# 130. Troubleshooting: OutOfSync

Step 1:

```bash
argocd app get <app>
```

Step 2:

```bash
argocd app diff <app>
```

Step 3:

Identify:

```text
Which resource?
Which field?
Desired value?
Live value?
```

Step 4:

Determine source:

```text
Git
kubectl
Operator
Webhook
Controller
```

Step 5:

Correct the authoritative source.

---

# 131. Troubleshooting: Sync Permission Denied

Possible causes:

```text
AppProject restrictions
Kubernetes RBAC
Cluster registration permissions
Admission policy
```

Check:

```bash
argocd app get <app>
kubectl describe application <app> -n argocd
```

Then inspect the target resource and cluster permissions.

---

# 132. Troubleshooting: Destination Not Permitted

If an Application cannot target a cluster/namespace:

```text
Application destination
        |
        v
AppProject destinations
```

Compare the two.

The Application must be allowed by its Project.

---

# 133. Troubleshooting: Repository Not Permitted

If the Project rejects a repository:

```text
Application source.repoURL
        |
        v
AppProject sourceRepos
```

The repository must match the Project's allowed sources.

---

# 134. Troubleshooting: Cluster-Scoped Resource Rejected

If an Application tries to create:

```text
ClusterRole
CRD
Namespace
```

and the Project does not permit it:

```text
Sync fails
```

Do not simply broaden the Project.

Ask:

```text
Should this Application own this resource?
```

Often the correct answer is to move the resource to a platform Application.

---

# 135. Troubleshooting: Helm Values Not Applied

Check:

```text
path
valueFiles
file path
Helm parameters
targetRevision
```

Then inspect Repo Server logs.

A common mistake is specifying:

```yaml
valueFiles:
  - values/prod.yaml
```

while the actual file is:

```text
environments/prod/values.yaml
```

Paths are relative to the chart/source context.

---

# 136. Troubleshooting: Wrong Cluster

If the Application deployed to an unexpected cluster:

Immediately inspect:

```bash
argocd app get <app>
```

Verify:

```text
Destination server/name
Project
ApplicationSet template
```

For production, cluster selection must be explicit and reviewed.

---

# 137. Troubleshooting: Wrong Namespace

Check:

```yaml
destination:
  namespace:
```

Then inspect:

```bash
kubectl get all -A | grep <app>
```

Possible cause:

```text
Helm namespace configuration
Application destination
Namespace override
```

The source manifests and destination settings must be consistent.

---

# 138. Troubleshooting: Application Deleted

First determine:

```text
Who deleted it?
Git?
ApplicationSet?
Operator?
Human?
Bootstrap Application?
```

If the Application was generated by ApplicationSet:

```text
ApplicationSet is the parent source.
```

Fix the generator/template rather than manually recreating the child.

---

# 139. Troubleshooting: Project Deleted

If an Application's Project disappears, Application behavior can be affected.

Project resources are foundational security objects.

Protect them with:

```text
Git review
RBAC
Dedicated platform ownership
Backup
Monitoring
```

---

# 140. Production GitOps Flow

For RoboShop:

```text
Developer
    |
    v
Application Git
    |
    v
CI
 |
 +--> Build
 +--> Test
 +--> SonarQube
 +--> Trivy
 +--> Veracode
 |
 v
Docker Image
 |
 v
ECR
 |
 v
GitOps repo update
 |
 v
Application revision
 |
 v
Argo CD Application
 |
 v
AppProject validation
 |
 v
Repo Server
 |
 v
Manifest rendering
 |
 v
Application Controller
 |
 v
EKS
 |
 v
Kubernetes
```

---

# 141. GitOps Audit Trail

A production change should be traceable through:

```text
Developer
   |
   v
Git commit
   |
   v
PR
   |
   v
CI checks
   |
   v
Merge
   |
   v
Argo CD revision
   |
   v
Kubernetes state
```

This is one of the strongest benefits of GitOps.

---

# 142. Git Revert as Rollback

Example:

```text
Current:
image.tag = 1.5.0

Known good:
image.tag = 1.4.9
```

Rollback:

```text
Git revert
 |
 v
image.tag = 1.4.9
 |
 v
Argo CD reconciliation
 |
 v
Kubernetes
```

This creates a durable audit trail.

---

# 143. Emergency Production Rollback

If an emergency requires direct Argo CD rollback:

```bash
argocd app history <app>
argocd app rollback <app> <history-id>
```

Afterward:

```text
Determine corresponding Git revision
       |
       v
Restore Git consistency
```

Never leave production permanently diverged from Git.

---

# 144. Application Health and Kubernetes Probes

Argo CD can report health based on resource state.

For Deployments:

```text
Desired replicas
Available replicas
Rollout condition
```

For Pods:

```text
Ready
Running
CrashLoopBackOff
```

Application-level health can be affected by these Kubernetes conditions.

---

# 145. Health vs Monitoring

Argo CD health is not a replacement for:

```text
Prometheus
Grafana
ELK
```

Argo CD answers:

```text
Is the declared resource reconciled and considered healthy?
```

Observability answers:

```text
Is the application actually performing well?
```

Use both.

---

# 146. Application Notifications

Production Applications can integrate notifications for:

```text
Sync failed
Sync succeeded
Health degraded
Application created
Application deleted
```

Notifications should route to appropriate channels:

```text
Slack
Email
Incident management
```

Avoid alert fatigue.

---

# 147. Production Alert Examples

Useful alerts:

```text
Application OutOfSync for too long
Application Degraded
Sync failure
Repository failure
Cluster unavailable
Application Controller unhealthy
Repo Server unhealthy
```

Alert severity should reflect business impact.

---

# 148. Application Metrics

Argo CD can expose metrics for monitoring.

A Prometheus-based stack can monitor:

```text
Application status
Sync operations
Controller performance
Repo Server activity
API Server behavior
```

The exact metric names should be checked against the installed Argo CD version.

---

# 149. Application Controller Capacity

The Application Controller is critical.

If it becomes overloaded:

```text
Reconciliation slows
Applications remain stale
Sync operations delay
```

Capacity planning should consider:

```text
Number of Applications
Number of managed resources
Reconciliation frequency
Cluster count
API latency
Repository complexity
```

---

# 150. Repo Server Capacity

Repo Server handles manifest generation.

Heavy workloads include:

```text
Large Helm charts
Many Applications
Large Git repositories
Kustomize processing
Plugins
Frequent refreshes
```

Monitor and scale based on real usage.

---

# 151. Project Design Principle

A good Project should answer:

> What is this team/application allowed to deploy, where, and from which source?

If you cannot answer that clearly, the Project is probably too broad.

---

# 152. Application Design Principle

A good Application should answer:

> What exact desired state does this Application own?

Avoid Applications that silently own unrelated resources.

---

# 153. Ownership Principle

Every production resource should ideally have one clear owner:

```text
AWS infrastructure -> Terraform
Platform Kubernetes -> Platform GitOps
Application Kubernetes -> Application GitOps
Runtime mutation -> Explicit operator/controller
```

Document exceptions.

---

# 154. Common Beginner Mistakes

### Mistake 1

```yaml
sourceRepos:
  - '*'
```

### Mistake 2

```yaml
destinations:
  - server: '*'
    namespace: '*'
```

### Mistake 3

Granting:

```text
ClusterRole
ClusterRoleBinding
CRD
```

to normal application Projects.

### Mistake 4

Using:

```text
latest
```

for production images.

### Mistake 5

Putting secrets in Git.

### Mistake 6

Deleting Applications without checking finalizers.

### Mistake 7

Using direct `kubectl` changes as normal operations.

### Mistake 8

Allowing CI to hold unrestricted cluster-admin credentials.

---

# 155. Production Best Practices

1. Use explicit Projects.
2. Restrict source repositories.
3. Restrict destination clusters.
4. Restrict namespaces.
5. Limit cluster-scoped resources.
6. Use protected Git branches.
7. Use immutable image tags/digests.
8. Keep secrets outside plain Git.
9. Use SSO and group-based RBAC.
10. Keep production sync controlled.
11. Enable self-heal where appropriate.
12. Enable prune only after lifecycle behavior is understood.
13. Use ApplicationSets for scale.
14. Avoid shared resource ownership.
15. Monitor Argo CD.
16. Back up control-plane configuration.
17. Test disaster recovery.
18. Document rollback.
19. Review Projects regularly.
20. Treat cluster credentials as production secrets.

---

# 156. Interview Questions

## Q1. What is an Argo CD Application?

### Answer

> An Argo CD Application is a Kubernetes custom resource that defines the desired deployment relationship between a source repository and a destination Kubernetes cluster/namespace. It specifies the Git revision and path, the destination, the Project security boundary, and synchronization behavior.

---

## Q2. What is an AppProject?

### Answer

> An AppProject is an Argo CD security and organizational boundary. It restricts which Git repositories Applications may use, which clusters and namespaces they may deploy to, and which Kubernetes resource types they may manage. It can also define scoped roles.

---

## Q3. What is the difference between an Application and an ApplicationSet?

### Answer

> An Application represents one deployment definition. An ApplicationSet generates and manages multiple Applications from templates and generators. For example, an ApplicationSet can generate the same RoboShop Application for dev, QA, and production clusters.

---

## Q4. What is the difference between Synced and Healthy?

### Answer

> Synced means the live configuration matches the desired state from Git. Healthy means the resource/application is considered operational according to its health assessment. An application can be Synced but Degraded if the deployed Pods are failing.

---

## Q5. What is prune?

### Answer

> Prune allows Argo CD to remove resources that are no longer present in the desired state. It is useful for lifecycle cleanup but can be destructive, so it should be enabled deliberately and tested before production use.

---

## Q6. What is selfHeal?

### Answer

> Self-healing allows Argo CD to automatically correct drift caused by changes made directly to live Kubernetes resources. If Git says a Deployment should have three replicas and someone manually changes it to one, Argo CD can restore the desired state when self-heal is enabled.

---

## Q7. Why use AppProjects?

### Answer

> Projects reduce deployment blast radius. They allow me to restrict repositories, destination clusters, namespaces, and resource types. For example, a RoboShop production Project can allow only the RoboShop Git repository, production EKS cluster, roboshop namespace, and approved namespaced resource types.

---

## Q8. Why should cluster-scoped resources be restricted?

### Answer

> Cluster-scoped resources such as ClusterRoles, CRDs, and Namespaces can affect workloads across the entire cluster. Granting them to ordinary application Projects increases the blast radius. I would normally put platform-owned cluster-scoped resources under a separate platform Project.

---

## Q9. How would you design production Applications for RoboShop?

### Answer

> I would generally separate Applications according to service ownership and lifecycle, such as cart, user, payment, catalogue and frontend. They would use a restricted production AppProject, a protected GitOps repository, immutable image versions, Helm values for production, controlled synchronization, and explicit cluster/namespace destinations.

---

## Q10. Why shouldn't Terraform and Argo CD manage the same Deployment?

### Answer

> That creates competing sources of truth. Terraform may continuously try to enforce one configuration while Argo CD tries to enforce another. I prefer Terraform for AWS infrastructure and Argo CD for Kubernetes application resources, with clearly documented exceptions.

---

# 157. Scenario Interview: Developer Changed Production with kubectl

Question:

> A developer manually changes the production Deployment replicas using kubectl. What happens?

Answer:

```text
Git desired state
       |
       v
Argo CD
       |
       X
Live state changed
       |
       v
OutOfSync
       |
       v
Self-heal
       |
       v
Git state restored
```

If self-heal is disabled:

```text
OutOfSync
```

remains until a synchronization occurs.

---

# 158. Scenario Interview: Production Is Synced but Degraded

Answer:

> I would treat this as a runtime health problem rather than an immediate Git configuration problem. I would inspect the Application resource tree, then check Pods, Deployment rollout status, events, probes, image pulls, configuration, dependencies and application logs. The key distinction is that synchronization confirms desired-state convergence, not business-level health.

---

# 159. Scenario Interview: Application Can Deploy Anywhere

Question:

> A production Application can select any cluster and namespace. Is that acceptable?

Answer:

> No. I would investigate the AppProject destination configuration. Production Projects should normally restrict destinations to the intended production cluster and namespace. Broad wildcard destinations increase the blast radius and can allow accidental or unauthorized deployments.

---

# 160. Scenario Interview: Application Needs a CRD

Question:

> A business Application needs to create a CRD. Would you simply add CRD permission to its Project?

Answer:

> Not automatically. I would first determine ownership. CRDs are cluster-scoped platform resources and usually belong to a platform/operator layer. If the application truly owns the CRD, I would explicitly document and approve that design. Otherwise I would deploy the CRD through a dedicated platform Application.

---

# 161. Scenario Interview: Git Revert vs Argo Rollback

Question:

> Which rollback method do you prefer?

Answer:

> For normal GitOps operations, I prefer reverting the Git change because Git remains the source of truth and the rollback is fully auditable. Argo CD rollback can be useful during an emergency, but afterward I would reconcile Git with the recovered production state.

---

# 162. Scenario Interview: Wrong Application Namespace

Question:

> An Application is healthy but the workload appears in the wrong namespace. What do you inspect?

Answer:

```text
Application destination.namespace
Helm release/value configuration
Kustomize namespace configuration
Generated manifests
Actual Kubernetes namespace
```

Then:

```bash
argocd app get <app>
argocd app diff <app>
```

---

# 163. Scenario Interview: GitOps Repository Is Compromised

The repository is a high-impact trust boundary.

Potential impact:

```text
Attacker modifies manifests
       |
       v
Argo CD sees desired state
       |
       v
Production changes
```

Controls should include:

```text
Branch protection
MFA
Least privilege
PR reviews
CI security checks
Commit/repository auditing
Secret scanning
Protected production paths
```

GitOps does not eliminate supply-chain risk.

---

# 164. Scenario Interview: CI Credential Compromise

If CI can write to the GitOps repository:

```text
CI credential
     |
     v
GitOps repository
     |
     v
Argo CD
     |
     v
Production
```

Therefore CI credentials must be treated as production-sensitive credentials.

Use:

```text
Least privilege
Short-lived credentials where possible
Protected branches
Approval controls
Audit
Rotation
```

---

# 165. Final Mental Model

Remember:

```text
AppProject
    |
    | security boundary
    v
Application
    |
    | source
    v
Git
    |
    | desired manifests
    v
Repo Server
    |
    | rendered manifests
    v
Application Controller
    |
    | reconcile
    v
Kubernetes
```

And:

```text
Application
=
WHAT + WHERE + HOW
```

while:

```text
AppProject
=
WHO + FROM WHERE + WHERE ALLOWED + WHAT ALLOWED
```

---

# 166. Key Takeaways

1. `Application` is the core deployment declaration.
2. `AppProject` is the security and organizational boundary.
3. `source` defines desired state location.
4. `targetRevision` defines the Git revision.
5. `path` identifies the source configuration.
6. Helm and Kustomize are manifest-generation mechanisms.
7. `destination` defines the target cluster and namespace.
8. `syncPolicy` controls reconciliation behavior.
9. `prune` removes resources no longer declared.
10. `selfHeal` corrects live drift.
11. `Synced` and `Healthy` are different states.
12. Finalizers affect Application deletion.
13. Projects should restrict repositories and destinations.
14. Cluster-scoped resources require stronger controls.
15. One resource should have one clear owner.
16. Git should remain the authoritative deployment record.
17. Production rollback should normally be represented in Git.
18. ApplicationSets generate Applications at scale.
19. App of Apps uses Applications to manage Applications.
20. Strong GitOps design minimizes deployment blast radius.

---