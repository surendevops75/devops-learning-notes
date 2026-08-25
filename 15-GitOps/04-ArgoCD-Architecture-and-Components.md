# 04-ArgoCD-Architecture-and-Components

## 1. Purpose

This file provides a deep production-oriented explanation of Argo CD architecture and its internal components.

The previous files established:

```text
Git = Source of Truth
Argo CD = GitOps Reconciliation Platform
Kubernetes = Desired-State Runtime Platform
EKS = Managed Kubernetes Platform
```

This file focuses specifically on how Argo CD is built and how its components work together.

The goal is to understand Argo CD not merely as:

```bash
argocd app sync
```

but as a production GitOps control plane capable of managing:

- Kubernetes applications
- Multiple namespaces
- Multiple environments
- Multiple EKS clusters
- Multiple AWS accounts
- Helm applications
- Kustomize applications
- Raw manifests
- ApplicationSets
- Platform components
- Production workloads

---

# 2. What Is Argo CD?

Argo CD is a declarative GitOps continuous delivery platform for Kubernetes.

Its primary responsibility is to continuously compare:

```text
Desired State
     |
     | Git
     v
Argo CD
     |
     | Kubernetes API
     v
Live State
```

When the desired state and live state differ, Argo CD identifies the difference.

Depending on configuration, it can:

```text
Detect drift
    |
    +--> Report OutOfSync
    |
    +--> Automatically synchronize
    |
    +--> Self-heal
```

Argo CD is therefore a reconciliation-oriented deployment platform rather than a traditional push-based CI server.

---

# 3. Argo CD in the GitOps Model

A production GitOps flow is:

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
Amazon ECR
    |
    v
GitOps Repository
    |
    v
Argo CD
    |
    v
Kubernetes API
    |
    v
EKS
```

Argo CD owns the deployment/reconciliation part.

CI should normally focus on:

```text
Build
Test
Security
Package
Publish
```

Argo CD focuses on:

```text
Desired state
Deployment
Reconciliation
Drift detection
Health
Synchronization
```

---

# 4. Why Argo CD Has Multiple Components

Argo CD performs several different jobs.

It needs to:

- Receive API/UI requests
- Authenticate users
- Read Git repositories
- Render Helm/Kustomize/manifests
- Compare desired and live state
- Synchronize resources
- Monitor Kubernetes resources
- Generate Applications
- Store/cache information
- Send notifications

Separating these responsibilities into components improves:

- Scalability
- Reliability
- Maintainability
- Security boundaries
- Independent troubleshooting

---

# 5. High-Level Argo CD Architecture

A simplified architecture is:

```text
                         Users
                           |
                    +------+------+
                    |             |
                  Web UI        CLI/API
                    |             |
                    +------+------+
                           |
                           v
                     Argo CD API
                        Server
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
     Repository        Application     Authentication
       Access          Operations          / RBAC
          |
          v
      Repo Server
          |
          v
     Git / Helm / Kustomize


                Application Controller
                         |
                         v
                 Kubernetes API
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
           EKS-DEV     EKS-QA     EKS-PROD
```

Additional components such as Redis, ApplicationSet Controller, Notifications, and Dex/SSO can participate depending on the installation and configuration.

---

# 6. Core Argo CD Components

The major components to understand are:

```text
argocd-server
argocd-repo-server
argocd-application-controller
argocd-redis
argocd-applicationset-controller
```

Additional components/features may include:

```text
Dex
Notifications Controller
Argo CD CLI
Argo CD UI
```

The exact deployment can vary by Argo CD version and installation method.

---

# 7. Component Responsibility Overview

| Component | Primary Responsibility |
|---|---|
| API Server | UI, CLI, API, authentication, authorization |
| Repo Server | Fetch and render repository content |
| Application Controller | Reconcile Applications with Kubernetes |
| Redis | Cache and internal acceleration |
| ApplicationSet Controller | Generate Applications |
| Dex / SSO integration | Authentication federation |
| Notifications | Application/event notifications |
| CLI | Operational interface |
| UI | Visual operational interface |

The most important distinction is:

```text
Repo Server
    =
Source retrieval + manifest generation

Application Controller
    =
Application reconciliation
```

---

# 8. Argo CD API Server

The API Server is the main interface between users/tools and Argo CD.

It handles requests from:

```text
Web UI
CLI
Automation
API clients
```

Conceptually:

```text
User
 |
 v
argocd-server
 |
 +--> Authentication
 +--> Authorization
 +--> Application operations
 +--> Repository operations
 +--> Cluster operations
 +--> Project operations
```

---

# 9. API Server Responsibilities

The API Server provides functionality for:

- Applications
- Projects
- Repositories
- Clusters
- Accounts
- RBAC
- Application operations
- Sync operations
- Health information
- Diff information
- History
- Logs/streaming operations depending on configuration

It acts as the front door to the Argo CD control plane.

---

# 10. UI Request Flow

When a user opens an Application in the Argo CD UI:

```text
Browser
   |
   v
Argo CD API Server
   |
   +--> Application information
   +--> Repository information
   +--> Controller state
   +--> Kubernetes state
   |
   v
UI
```

The UI is not itself the reconciliation engine.

The controller performs reconciliation.

---

# 11. CLI Request Flow

For:

```bash
argocd app get roboshop-cart
```

the CLI communicates with the Argo CD API Server.

Conceptually:

```text
argocd CLI
    |
    v
argocd-server
    |
    v
Application information
```

For:

```bash
argocd app sync roboshop-cart
```

the CLI sends a sync request through the API Server.

The controller then performs the actual reconciliation work.

---

# 12. API Server Does Not Equal Controller

This distinction is important.

The API Server:

```text
Receives commands
Provides APIs
Handles authentication/authorization
```

The Application Controller:

```text
Performs reconciliation
Compares desired/live state
Applies changes
Monitors health
```

Therefore:

```text
argocd-server down
```

and:

```text
argocd-application-controller down
```

are different failure scenarios.

---

# 13. Application Controller

The Application Controller is the heart of Argo CD's reconciliation system.

Its primary job is to watch Argo CD Applications and ensure the live Kubernetes state matches the desired state.

Conceptually:

```text
Git
 |
 v
Desired State
 |
 v
Application Controller
 |
 +--> Compare
 +--> Reconcile
 +--> Sync
 +--> Health
 |
 v
Kubernetes
```

---

# 14. Application Controller Control Loop

A simplified control loop is:

```text
Observe
   |
   v
Compare
   |
   v
Determine difference
   |
   v
Evaluate sync policy
   |
   +--> Manual sync required
   |
   +--> Automated sync
   |
   v
Apply desired resources
   |
   v
Observe new state
   |
   v
Evaluate health
   |
   v
Repeat
```

This is the central GitOps reconciliation mechanism.

---

# 15. Controller Watch Model

The Application Controller needs information about:

```text
Applications
Repositories
Kubernetes resources
Application health
Desired revisions
```

It maintains internal state/cache and continuously processes changes.

This enables Argo CD to detect state changes without requiring a user to manually run a deployment command every time.

---

# 16. Desired State Calculation

Before Argo CD can compare state, it needs to know:

```text
What should exist?
```

The desired state comes from:

```text
Application source
```

For example:

```yaml
source:
  repoURL: https://github.com/example/gitops.git
  targetRevision: main
  path: applications/cart
```

Argo CD uses the source configuration to obtain and render manifests.

---

# 17. Repo Server

The Repo Server is responsible for repository-related processing.

Conceptually:

```text
Application Controller
        |
        v
    Repo Server
        |
        +--> Git
        +--> Helm
        +--> Kustomize
        +--> Plugins
        |
        v
Rendered manifests
```

This separation prevents the Application Controller from having to perform all repository rendering itself.

---

# 18. Repo Server Responsibilities

The Repo Server can:

- Connect to Git repositories
- Fetch revisions
- Read repository files
- Render Helm charts
- Render Kustomize applications
- Process directory manifests
- Run supported config-management tooling
- Return generated manifests

The exact capabilities depend on Argo CD version and configuration.

---

# 19. Why Manifest Rendering Is Important

Argo CD does not always deploy files exactly as they appear in Git.

For Helm:

```text
Chart + Values
      |
      v
Helm Rendering
      |
      v
Kubernetes YAML
```

For Kustomize:

```text
Base + Overlay
      |
      v
Kustomize Rendering
      |
      v
Kubernetes YAML
```

Argo CD compares the rendered desired resources with live resources.

---

# 20. Git Repository Flow

A simplified flow is:

```text
Application
    |
    v
Repository URL
    |
    v
Revision
    |
    v
Path
    |
    v
Repo Server
    |
    v
Fetch source
    |
    v
Render manifests
    |
    v
Desired resources
```

The controller then uses the resulting manifests.

---

# 21. Redis

Argo CD uses Redis for caching and internal performance-related data.

The exact data and behavior depend on Argo CD version and architecture.

Conceptually:

```text
API Server
     |
     v
Redis

Controller
     |
     v
Redis

Repo Server
     |
     v
Redis
```

Redis should be viewed as an internal acceleration/cache component rather than the authoritative source of desired application configuration.

---

# 22. Git Is Still the Source of Truth

Even when Redis is used:

```text
Git
```

remains the desired-state source.

Redis is not:

```text
Primary GitOps source
```

It is an internal component used to improve performance and reduce repeated work.

---

# 23. Redis Failure

If Redis becomes unavailable, impact depends on the Argo CD version, architecture, and operation being performed.

The important operational principle is:

```text
Redis failure != Git repository loss
```

Git remains the durable desired-state source.

However, Redis availability should still be considered when designing highly available production Argo CD installations.

---

# 24. ApplicationSet Controller

The ApplicationSet Controller generates Argo CD Application resources from templates and generators.

Without ApplicationSet:

```text
Application 1
Application 2
Application 3
...
```

must be created separately.

With ApplicationSet:

```text
Generator
   |
   v
Application Template
   |
   +--> Application 1
   +--> Application 2
   +--> Application 3
```

This is extremely useful for:

- Multiple environments
- Multiple clusters
- Multiple applications
- Fleet management

---

# 25. Application vs ApplicationSet

An `Application` describes one Argo CD managed application.

An `ApplicationSet` describes a mechanism for generating multiple Applications.

Conceptually:

```text
Application:
    "Deploy cart to EKS-DEV"

ApplicationSet:
    "Generate cart Applications for DEV, QA, PROD"
```

---

# 26. ApplicationSet Architecture

```text
Git / Cluster / List / Other Generator
                  |
                  v
          ApplicationSet Controller
                  |
                  v
             Template
                  |
        +---------+---------+
        |         |         |
        v         v         v
      App-DEV   App-QA   App-PROD
```

The generated Applications are then handled by the normal Application Controller.

---

# 27. ApplicationSet Controller vs Application Controller

These components have different responsibilities.

```text
ApplicationSet Controller
        |
        | Generates
        v
Application objects
        |
        v
Application Controller
        |
        | Reconciles
        v
Kubernetes resources
```

This distinction is essential for troubleshooting ApplicationSet deployments.

---

# 28. Dex and SSO

Argo CD can integrate with external identity providers.

Common enterprise concepts include:

```text
OIDC
SAML through an identity platform
LDAP integrations through supported identity architecture
Enterprise SSO
```

Dex can act as an identity broker in supported configurations.

The architecture can be:

```text
User
 |
 v
Argo CD
 |
 v
Identity Provider
 |
 v
Authentication
 |
 v
Argo CD RBAC
```

The exact implementation depends on the chosen identity provider and Argo CD configuration.

---

# 29. Authentication vs Authorization

These are separate.

Authentication:

> Who is the user?

Authorization:

> What can the user do?

Example:

```text
Developer
   |
   v
SSO
   |
   v
Authenticated
   |
   v
Argo CD RBAC
   |
   v
Can sync DEV
Cannot sync PROD
```

Enterprise GitOps requires both.

---

# 30. Argo CD RBAC

Argo CD RBAC can restrict actions against:

- Applications
- Projects
- Repositories
- Clusters
- Other Argo CD resources

A production model might be:

```text
Developers
    |
    +--> View PROD
    +--> Sync DEV
    +--> Sync QA

Release Managers
    |
    +--> Sync PROD

Platform Team
    |
    +--> Manage Projects
    +--> Manage Clusters
```

The exact roles should reflect organizational responsibilities.

---

# 31. AppProject

Argo CD Projects provide boundaries around applications.

An AppProject can constrain:

- Allowed repositories
- Allowed destination clusters
- Allowed namespaces
- Resource types
- Application permissions

Conceptually:

```text
AppProject
    |
    +--> Allowed Git repositories
    |
    +--> Allowed clusters
    |
    +--> Allowed namespaces
    |
    +--> Allowed resource kinds
```

This is a major security control.

---

# 32. Why Projects Matter

Without project boundaries, an Application could potentially be configured too broadly.

For example, an application might be allowed to target:

```text
Any cluster
Any namespace
Any repository
```

A production AppProject should instead restrict the blast radius.

---

# 33. Production Project Boundary

Example concept:

```text
roboshop-prod project
 |
 +--> repo: company/gitops
 |
 +--> cluster: EKS-PROD
 |
 +--> namespaces: roboshop
 |
 +--> approved resource kinds
```

Then a typo in an Application destination is less likely to result in deployment to an unintended environment.

---

# 34. Application

An Argo CD Application is a Kubernetes custom resource representing a deployed application.

Basic structure:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-cart
  namespace: argocd
spec:
  project: roboshop-prod
  source:
    repoURL: https://github.com/example/roboshop-gitops.git
    targetRevision: main
    path: environments/prod/cart
  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop
```

Later files will deeply cover every field.

---

# 35. Application Resource Flow

```text
Application CR
      |
      v
Application Controller
      |
      +--> Read source
      |
      +--> Render manifests
      |
      +--> Compare
      |
      +--> Sync
      |
      v
Kubernetes resources
```

The Application is the central declarative object representing the relationship between source and destination.

---

# 36. Application Source

The source identifies:

```text
Where desired state comes from
```

It can refer to:

- Git repository
- Path
- Revision
- Helm configuration
- Kustomize configuration
- Directory configuration
- Plugin configuration

Example:

```yaml
source:
  repoURL: https://github.com/example/gitops.git
  targetRevision: main
  path: environments/prod/cart
```

---

# 37. Application Destination

The destination identifies:

```text
Where the desired state should be deployed
```

Example:

```yaml
destination:
  server: https://kubernetes.default.svc
  namespace: roboshop
```

For a registered remote cluster, the server can point to the remote Kubernetes API endpoint.

---

# 38. Local Cluster vs Remote Cluster

Argo CD can manage its own cluster.

Example:

```text
Argo CD
 |
 v
Same EKS cluster
```

It can also manage remote clusters:

```text
Argo CD in EKS-Management
      |
      +--> EKS-DEV
      +--> EKS-QA
      +--> EKS-PROD
```

This is the foundation of centralized multi-cluster GitOps.

---

# 39. Centralized Argo CD Architecture

A production model:

```text
                       Git
                        |
                        v
                 Central Argo CD
                        |
             +----------+----------+
             |          |          |
             v          v          v
          EKS-DEV    EKS-QA     EKS-PROD
```

The Argo CD management cluster acts as the GitOps control plane.

Target clusters remain separate Kubernetes runtime environments.

---

# 40. Trust Boundaries

In centralized GitOps, there are multiple trust boundaries:

```text
Git
 |
 | Repository credentials
 v
Argo CD
 |
 | Cluster credentials
 v
Target EKS
 |
 v
Kubernetes resources
```

Security must protect:

1. Git repository access
2. Argo CD control plane
3. Cluster credentials
4. Kubernetes RBAC
5. Production Applications

---

# 41. Cluster Credentials

For remote clusters, Argo CD needs a mechanism to authenticate to the Kubernetes API.

Conceptually:

```text
Argo CD
   |
   | Cluster credential
   v
EKS API
```

These credentials must be protected.

Do not treat cluster credentials as ordinary application configuration.

---

# 42. Application Controller and Kubernetes API

The Application Controller needs access to target clusters.

For each Application:

```text
Application
   |
   v
Destination
   |
   v
Cluster
   |
   v
Kubernetes API
```

The controller reads and applies resources according to its permissions.

---

# 43. Repository Credentials

The Repo Server needs access to private Git repositories.

Conceptually:

```text
Repo Server
    |
    | SSH / HTTPS credential
    v
Private Git repository
```

Credentials can include:

- SSH private keys
- HTTPS tokens
- Other supported authentication mechanisms

They should be stored securely and rotated.

---

# 44. Private Repository Flow

```text
Application
   |
   v
Repo Server
   |
   +--> Repository credentials
   |
   v
Private Git
   |
   v
Manifest
```

If credentials are invalid:

```text
Manifest generation/fetch
        |
        X
Authentication failure
```

This can result in Application errors or stale desired state.

---

# 45. SSH Repository Authentication

A private repository can be accessed using SSH.

Conceptually:

```text
ssh://git@example.com/company/gitops.git
```

The Repo Server uses the configured SSH credential.

Production considerations:

- Protect private keys
- Verify host keys
- Rotate credentials
- Use least privilege
- Monitor authentication failures

---

# 46. HTTPS Repository Authentication

A repository may also be accessed over HTTPS using a supported token mechanism.

Conceptually:

```text
https://git.example.com/company/gitops.git
```

The credential should not be embedded directly in the repository URL.

Store it through Argo CD's repository credential configuration.

---

# 47. Repository Access Failure

Typical symptoms:

```text
Application
   |
   v
Manifest generation error
```

Potential causes:

- Wrong URL
- Invalid token
- Expired token
- Invalid SSH key
- Host-key mismatch
- Network connectivity
- Repository renamed
- Repository deleted
- Branch/tag does not exist

Start investigation with:

```bash
argocd repo list
argocd app get <application>
```

---

# 48. Cluster Access Failure

A cluster access failure can occur when:

- Cluster endpoint is unreachable
- Credentials are invalid
- Authorization changed
- EKS access configuration changed
- Network routing fails
- Certificate validation fails
- Cluster is unavailable

Useful command:

```bash
argocd cluster list
```

Then inspect the affected Application.

---

# 49. Argo CD Reconciliation States

Argo CD commonly exposes two important dimensions:

```text
Sync Status
Health Status
```

They answer different questions.

Sync:

> Does live configuration match desired configuration?

Health:

> Is the application/resource functioning as expected?

---

# 50. Sync Status

Common concepts include:

```text
Synced
OutOfSync
Unknown
```

Synced means:

```text
Desired and live state are considered matching
```

OutOfSync means:

```text
A difference exists
```

Unknown means:

```text
Argo CD cannot reliably determine state
```

---

# 51. Health Status

Health may include states such as:

```text
Healthy
Progressing
Degraded
Suspended
Missing
Unknown
```

The exact resource-specific behavior depends on the resource type and Argo CD health assessment.

---

# 52. Synced + Healthy

Ideal state:

```text
Sync = Synced
Health = Healthy
```

Meaning:

```text
Desired state matches
and
Resources are healthy
```

This is the normal production target.

---

# 53. Synced + Degraded

Example:

```text
Sync = Synced
Health = Degraded
```

This means:

```text
Git configuration matches
but
application runtime is unhealthy
```

Examples:

- Pods crash
- Deployment unavailable
- Ingress unhealthy
- Custom resource reports failure

This distinction is critical during interviews.

---

# 54. OutOfSync + Healthy

Possible state:

```text
Sync = OutOfSync
Health = Healthy
```

For example, an engineer manually changes a non-critical field and the workload continues serving traffic.

The application may be healthy now but is not aligned with Git.

That is configuration drift.

---

# 55. OutOfSync + Degraded

This is more serious:

```text
Desired != Live
and
Application unhealthy
```

Investigate both:

```text
Configuration difference
+
Runtime failure
```

---

# 56. Application Controller and Health

The Application Controller evaluates resource health.

For a Deployment it may consider:

```text
Desired replicas
Available replicas
Progress
Conditions
```

For custom resources, health may depend on resource-specific health logic.

Therefore health is not simply:

```text
Pod exists
```

It is a higher-level assessment.

---

# 57. Reconciliation Trigger

Argo CD reconciliation can be influenced by:

- Repository changes
- Application changes
- Kubernetes resource changes
- Refresh requests
- Webhooks
- Scheduled/periodic reconciliation
- Manual operations

This creates continuous synchronization behavior.

---

# 58. Repository Webhook

A Git provider webhook can notify Argo CD that repository content changed.

Conceptually:

```text
Git push
   |
   v
Webhook
   |
   v
Argo CD
   |
   v
Refresh/reconciliation
```

Without relying only on periodic polling, webhooks can make GitOps updates more responsive.

---

# 59. Why Webhooks Are Useful

Suppose:

```text
Developer merges PR at 10:00:00
```

A webhook can immediately notify Argo CD.

Then:

```text
Git
 |
 v
Webhook
 |
 v
Argo CD refresh
 |
 v
Sync
```

This can reduce time between Git merge and deployment.

---

# 60. Webhook Security

Production webhooks should consider:

- Authentication/signature validation
- HTTPS
- Network restrictions
- Provider-specific webhook security
- Replay protection where applicable
- Minimal exposure

Do not expose an unauthenticated administrative endpoint unnecessarily.

---

# 61. Argo CD Refresh

Refresh means Argo CD reevaluates application state/source information.

Conceptually:

```text
Git/source
    |
    v
Refresh
    |
    v
Desired state recalculated
    |
    v
Compare with live state
```

Refresh and sync are different.

---

# 62. Refresh vs Sync

Refresh:

```text
Recalculate/observe state
```

Sync:

```text
Apply desired resources
```

Example:

```text
Git changed
 |
 v
Refresh
 |
 v
OutOfSync
 |
 v
Sync
 |
 v
Live state updated
```

An automated sync policy can perform the second step automatically.

---

# 63. Sync Operation

A sync generally means:

```text
Take desired state
and
apply it to target cluster
```

Depending on sync options, it can involve:

- Create
- Update
- Delete
- Replace
- Apply
- Prune
- Ordering

Detailed sync behavior will be covered in the next files.

---

# 64. Automated Sync

An Application can specify:

```yaml
syncPolicy:
  automated: {}
```

This means Argo CD can automatically synchronize when configured conditions indicate that synchronization is required.

Production environments should use appropriate safeguards rather than assuming automatic sync is always correct.

---

# 65. Prune

Pruning means deleting resources that are no longer present in desired state when pruning is enabled.

Example:

Git removes:

```text
old-service.yaml
```

Desired state no longer contains the Service.

With pruning enabled:

```text
Argo CD
   |
   v
Delete old Service
```

Without pruning:

```text
Old Service may remain
```

Pruning is powerful and should be controlled carefully.

---

# 66. Self-Heal

Self-healing means Argo CD can restore drifted resources toward the desired state.

Example:

Git:

```text
replicas = 3
```

Engineer manually changes:

```bash
kubectl scale deployment cart --replicas=1
```

With self-healing:

```text
Argo CD detects drift
        |
        v
replicas restored to 3
```

This is one of the major GitOps benefits.

---

# 67. Sync Options

Argo CD provides synchronization options that influence how resources are applied.

Examples include concepts such as:

```text
CreateNamespace
Prune
Replace
ServerSideApply
Validate
PruneLast
```

The exact behavior and availability depend on the Argo CD version.

Production teams should enable only the options needed for their workloads.

---

# 68. Sync Waves

Sync waves allow resources to be ordered.

Example:

```text
Wave -2
Namespace

Wave -1
CRD

Wave 0
Application resources

Wave 1
Dependent resources
```

This is useful when resources have dependencies.

---

# 69. Hooks

Argo CD hooks can execute resources at lifecycle stages.

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

Hooks can be used for controlled deployment workflows.

They should be used carefully because they introduce operational complexity.

---

# 70. Resource Tracking

Argo CD needs to determine:

```text
Which Kubernetes resources belong to this Application?
```

This is resource tracking.

It allows Argo CD to understand:

```text
Application A
   |
   +--> Deployment
   +--> Service
   +--> ConfigMap
```

and distinguish them from:

```text
Application B
```

Resource tracking becomes particularly important in shared namespaces and multi-application environments.

---

# 71. Why Resource Tracking Matters

Without reliable tracking, Argo CD could have difficulty determining:

- Ownership
- Drift
- Pruning
- Application resource trees

Production repositories should avoid overlapping ownership.

For example:

```text
App A -> Deployment/cart
App B -> Deployment/cart
```

is an unsafe design.

One resource should have one clear desired-state owner.

---

# 72. Finalizers

Argo CD Applications can use Kubernetes finalizers to control cleanup behavior.

A common concept is:

```text
Application deleted
       |
       v
Finalizer
       |
       v
Managed resources cleaned up
```

This is powerful.

Removing an Application can potentially remove the resources it owns, depending on configuration and finalizers.

Production teams must understand this before deleting Applications.

---

# 73. Application Deletion Risk

Consider:

```text
platform-root
 |
 +--> prod-cart
 +--> prod-orders
 +--> prod-payment
```

If deletion cascades are enabled:

```text
Delete parent
   |
   v
Child Applications
   |
   v
Managed resources
```

This can create a large blast radius.

Always inspect:

```bash
kubectl get application <name> -n argocd -o yaml
```

before destructive production operations.

---

# 74. High Availability Architecture

For production Argo CD, consider redundancy for critical components.

Conceptually:

```text
                  Load Balancer
                       |
                +------+------+
                |             |
             API Server    API Server
                |             |
                +------+------+
                       |
                  Controllers
                       |
                 Target EKS
```

The exact number of replicas and HA topology should be based on:

- Application count
- Cluster count
- Repository size
- Sync frequency
- Availability requirements

---

# 75. API Server Scaling

If many users and automation clients access Argo CD:

```text
Users
 |
 v
Load Balancer
 |
 +--> API Server
 +--> API Server
```

Horizontal scaling can improve availability and throughput.

API Server scaling alone does not solve controller bottlenecks.

---

# 76. Application Controller Scaling

The Application Controller is often one of the most important components for large installations.

A large environment might manage:

```text
Hundreds or thousands of Applications
```

and potentially:

```text
Many Kubernetes clusters
```

Controller capacity must be planned based on actual workload.

Argo CD provides scaling mechanisms, including sharding approaches in supported versions/configurations.

---

# 77. Repo Server Scaling

Repo Server workload increases with:

- Number of repositories
- Manifest generation frequency
- Helm rendering
- Kustomize rendering
- Repository size
- Refresh activity

A large GitOps platform may need multiple Repo Server replicas.

Caching can also reduce repeated work.

---

# 78. Redis High Availability

Redis availability matters for production installations.

The exact HA implementation depends on deployment mode and supported Argo CD version.

The key principle is:

```text
Do not treat internal cache infrastructure as an afterthought.
```

Evaluate:

- Availability
- Persistence requirements
- Recovery
- Monitoring
- Resource limits

---

# 79. Multi-Cluster Architecture

A centralized Argo CD can manage multiple EKS clusters:

```text
                         Git
                          |
                          v
                   Central Argo CD
                    /      |      \
                   /       |       \
                  v        v        v
              EKS-DEV   EKS-QA   EKS-PROD
```

Each cluster has:

```text
Kubernetes API
Cluster credentials
RBAC boundary
Workloads
```

Argo CD acts as the central control plane.

---

# 80. Multi-Account AWS Architecture

A mature organization may separate:

```text
AWS Account DEV
AWS Account QA
AWS Account PROD
```

Example:

```text
Central GitOps Control Plane
             |
      +------+------+
      |      |      |
      v      v      v
    DEV     QA     PROD
   Account Account Account
      |      |      |
     EKS    EKS    EKS
```

This improves isolation and reduces production blast radius.

---

# 81. Cross-Account Security

Central Argo CD managing cross-account EKS environments introduces important security considerations.

The architecture must define:

```text
Who can access each cluster?
Which identity does Argo CD use?
Which Kubernetes permissions exist?
Which AWS account boundaries apply?
```

Do not give the central control plane unrestricted access simply because it is convenient.

---

# 82. Blast Radius

Suppose one Argo CD instance manages:

```text
DEV
QA
PROD
```

A compromised or misconfigured Argo CD installation can potentially affect all three.

Therefore:

```text
Centralized GitOps
```

provides operational efficiency but creates a larger control-plane blast radius.

Organizations may mitigate this with:

- Strong RBAC
- AppProjects
- Separate Argo CD instances
- Separate production control planes
- AWS account isolation
- Network isolation
- Protected repositories

---

# 83. When to Use Separate Argo CD Instances

Separate Argo CD instances may be appropriate when:

- Production requires stronger isolation
- Regulatory boundaries exist
- Business units need independent control
- Cluster fleets are very large
- Failure domains must be isolated
- Teams need independent upgrade schedules

There is no universal rule that every organization must use one Argo CD instance.

---

# 84. Centralized vs Distributed Argo CD

### Centralized

```text
One Argo CD
 |
 +--> DEV
 +--> QA
 +--> PROD
```

Advantages:

- Centralized management
- Consistent policies
- Easier fleet visibility
- Less duplicated control-plane infrastructure

Risks:

- Larger blast radius
- Central control-plane dependency
- More complex permissions

### Distributed

```text
Argo CD-DEV --> DEV
Argo CD-QA  --> QA
Argo CD-PROD --> PROD
```

Advantages:

- Stronger isolation
- Smaller blast radius
- Independent upgrades

Risks:

- More operational overhead
- Multiple control planes
- More management effort

---

# 85. Argo CD and EKS

For the user's environment:

```text
AWS
 |
 +--> VPC
 |
 +--> EKS
      |
      +--> Argo CD
      |
      +--> RoboShop
      |
      +--> Prometheus
      +--> Grafana
      +--> ELK
```

Argo CD can be installed into an EKS cluster and used as the GitOps control plane.

For centralized multi-cluster management, it can also be hosted on a dedicated management EKS cluster.

---

# 86. Argo CD Installation Model

Argo CD is normally installed into a Kubernetes namespace, commonly:

```text
argocd
```

A conceptual installation:

```bash
kubectl create namespace argocd
```

Then install Argo CD using an approved installation method.

The exact installation commands should be verified against the Argo CD version being deployed.

Detailed installation and configuration are covered in:

```text
05-ArgoCD-Installation-and-Configuration.md
```

---

# 87. Production Installation Principle

Do not treat:

```text
kubectl apply -f installation.yaml
```

as the entire production design.

Production planning should include:

- Namespace
- Resource sizing
- High availability
- Ingress
- TLS
- Authentication
- RBAC
- Repository credentials
- Cluster credentials
- Network controls
- Monitoring
- Backup/recovery
- Upgrade process

---

# 88. Argo CD Namespace

The control plane is commonly isolated in:

```text
argocd
```

Example:

```bash
kubectl get pods -n argocd
```

Expected components may include:

```text
argocd-server
argocd-repo-server
argocd-application-controller
argocd-redis
argocd-applicationset-controller
```

Additional components depend on enabled features.

---

# 89. Checking Argo CD Components

Useful command:

```bash
kubectl get pods -n argocd
```

Then:

```bash
kubectl get deployments -n argocd
kubectl get statefulsets -n argocd
kubectl get services -n argocd
```

For detailed diagnosis:

```bash
kubectl describe pod <pod> -n argocd
```

---

# 90. API Server Troubleshooting

If the UI or CLI cannot connect:

```bash
kubectl get pods -n argocd
kubectl get svc -n argocd
kubectl logs deployment/argocd-server -n argocd
```

Investigate:

- Pod status
- Service
- Ingress
- TLS
- Network
- Authentication
- Load balancer
- DNS

---

# 91. Repo Server Troubleshooting

If applications show manifest generation errors:

```bash
kubectl get pods -n argocd
kubectl logs deployment/argocd-repo-server -n argocd
```

Check:

- Git authentication
- Repository URL
- Revision
- Helm errors
- Kustomize errors
- Plugin failures
- Repository connectivity
- Resource limits

---

# 92. Application Controller Troubleshooting

If applications do not synchronize or update correctly:

```bash
kubectl logs statefulset/argocd-application-controller -n argocd
```

The exact workload type can vary by version/configuration, so verify:

```bash
kubectl get pods -n argocd
```

Look for:

- Reconciliation errors
- Kubernetes API failures
- Permission errors
- Resource comparison errors
- Cluster connectivity
- Application processing problems

---

# 93. ApplicationSet Controller Troubleshooting

If Applications are not being generated:

```bash
kubectl logs deployment/argocd-applicationset-controller -n argocd
```

Then inspect:

```bash
kubectl get applicationset -A
kubectl describe applicationset <name> -n argocd
kubectl get applications -A
```

Investigate:

- Generator configuration
- Git paths
- Cluster labels
- Template errors
- RBAC
- Repository access

---

# 94. Redis Troubleshooting

Check:

```bash
kubectl get pods -n argocd
```

Then:

```bash
kubectl logs <redis-pod> -n argocd
```

Investigate:

- Pod status
- Memory
- Connectivity
- Restarts
- Service discovery
- Resource pressure

Redis is an internal dependency, so monitor it as part of the Argo CD platform.

---

# 95. Component Dependency Model

A simplified dependency model:

```text
User
 |
 v
API Server
 |
 +----------------+
 |                |
 v                v
Redis          Application data
                  |
                  v
            Application Controller
                  |
          +-------+-------+
          |               |
          v               v
      Repo Server      Kubernetes API
          |
          v
         Git
```

ApplicationSet Controller adds:

```text
ApplicationSet
     |
     v
Applications
     |
     v
Application Controller
```

---

# 96. Argo CD and Git

Git is outside the Argo CD cluster.

This is an important architectural boundary:

```text
Git Provider
     |
     | HTTPS/SSH
     v
Repo Server
```

The Git repository remains the durable source of desired configuration.

Argo CD should be designed so loss of a Pod or cache does not destroy desired state.

---

# 97. Argo CD and Kubernetes

The Kubernetes API is another external boundary when managing remote clusters.

```text
Argo CD
 |
 | HTTPS/API
 v
Target Kubernetes API
```

For centralized multi-cluster deployments:

```text
             Argo CD
            /   |   \
           /    |    \
          v     v     v
       EKS-DEV EKS-QA EKS-PROD
```

---

# 98. Argo CD as a Kubernetes-Native System

Argo CD itself is largely deployed as Kubernetes resources.

This means:

```text
Kubernetes
   |
   v
Argo CD
   |
   v
Kubernetes
```

Argo CD can manage Kubernetes Applications, including potentially platform resources.

This recursive-looking architecture is one reason bootstrapping and ownership must be carefully designed.

---

# 99. Bootstrapping Argo CD

A common bootstrap pattern is:

```text
Terraform
 |
 v
EKS
 |
 v
Argo CD installation
 |
 v
Root Application
 |
 v
Platform Applications
 |
 v
Application Applications
```

After initial bootstrap, Argo CD can manage much of the remaining Kubernetes configuration.

---

# 100. Bootstrap Problem

There is a fundamental question:

> Who installs Argo CD before Argo CD can manage itself?

Common answers:

```text
Terraform
Helm
Infrastructure pipeline
Manual bootstrap
Cluster provisioning automation
```

After bootstrap:

```text
Argo CD manages Argo CD configuration
```

if the organization chooses self-management.

---

# 101. Self-Managed Argo CD

Argo CD can potentially manage its own configuration.

Conceptually:

```text
Bootstrap
   |
   v
Argo CD
   |
   v
Git
   |
   v
Argo CD configuration
```

But this creates a dependency:

```text
Argo CD controls itself
```

Therefore recovery procedures must exist for cases where Argo CD becomes unhealthy.

---

# 102. Production Bootstrap Recommendation

Keep the bootstrap path documented and reproducible.

For example:

```text
Terraform
   |
   +--> EKS
   |
   +--> IAM
   |
   v
Argo CD installation
   |
   v
Bootstrap Application
   |
   v
Platform
   |
   v
Applications
```

Do not rely on an undocumented manual setup that only one engineer knows.

---

# 103. Argo CD Security Boundaries

Major security boundaries include:

```text
1. Git credentials
2. Argo CD users
3. Argo CD RBAC
4. AppProjects
5. Repository access
6. Cluster credentials
7. Kubernetes RBAC
8. AWS IAM
9. Network access
10. Production Git branches
```

A secure GitOps design addresses all ten.

---

# 104. Least Privilege

Avoid:

```text
Argo CD = unrestricted administrator everywhere
```

unless there is a documented reason.

Prefer:

```text
Project restrictions
+
Cluster restrictions
+
Namespace restrictions
+
Resource restrictions
+
Kubernetes RBAC
+
AWS isolation
```

The exact permissions should be tested against the application's actual needs.

---

# 105. Repository Security

Production Git repositories should have:

- Protected branches
- Pull requests
- Required reviews
- CODEOWNERS where appropriate
- Secret scanning
- Audit logs
- Restricted write permissions
- CI validation
- Signed commits/attestations where organizationally required

GitOps turns repository access into deployment access.

Therefore:

> Write access to a production GitOps repository is highly privileged.

---

# 106. Production Branch Protection

A common model:

```text
feature/*
   |
   v
Pull Request
   |
   +--> Validation
   +--> Security
   +--> Review
   |
   v
main
   |
   v
Production GitOps
```

Do not allow arbitrary direct changes to the production branch unless there is a controlled emergency procedure.

---

# 107. Argo CD and Auditability

GitOps provides a useful audit chain:

```text
Developer
   |
   v
Git commit
   |
   v
Pull Request
   |
   v
Approval
   |
   v
Merge
   |
   v
Argo CD sync
   |
   v
Kubernetes
```

This is much easier to audit than undocumented manual changes.

---

# 108. Argo CD and Production Access

A mature organization may minimize direct production Kubernetes access.

Instead of:

```text
Engineer
 |
 v
kubectl apply
 |
 v
Production
```

the preferred normal path is:

```text
Engineer
 |
 v
Git Pull Request
 |
 v
Review
 |
 v
Argo CD
 |
 v
Production
```

Emergency access may still exist but should be controlled and audited.

---

# 109. Why Direct kubectl Is Dangerous

Direct commands such as:

```bash
kubectl edit deployment cart -n roboshop
```

can create drift.

The correct production workflow is usually:

```text
Change Git
   |
   v
Review
   |
   v
Merge
   |
   v
Argo CD
```

Emergency changes should be followed by reconciliation of Git so the desired state remains accurate.

---

# 110. Argo CD and Incident Response

During an incident, the team should determine:

```text
Was the failure caused by:
- Git change?
- Argo CD?
- Kubernetes?
- Application?
- AWS?
```

Do not automatically blame Argo CD simply because it is the deployment mechanism.

The architecture is layered.

---

# 111. Layered Troubleshooting Model

Use:

```text
Layer 1: Git
   |
Layer 2: Argo CD source/rendering
   |
Layer 3: Argo CD reconciliation
   |
Layer 4: Kubernetes API
   |
Layer 5: Kubernetes controllers
   |
Layer 6: Pod/runtime
   |
Layer 7: AWS integration
   |
Layer 8: Application dependencies
```

This is one of the most important operational mental models.

---

# 112. Scenario: Application Stuck OutOfSync

Investigate:

```bash
argocd app get <app>
argocd app diff <app>
```

Determine:

```text
What resource?
What field?
Who changed it?
Is it controller-managed?
Is it intentional?
```

Then decide whether to:

```text
Fix Git
or
Configure expected differences
or
Correct another controller
```

Do not immediately enable ignore-differences without understanding the root cause.

---

# 113. Scenario: Application Stuck Progressing

Check:

```bash
kubectl get deployment -n <namespace>
kubectl get pods -n <namespace>
kubectl describe deployment <name> -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl get events -n <namespace>
```

Potential causes:

- Pods not scheduling
- Readiness failure
- Image pull
- CrashLoopBackOff
- Insufficient resources
- Dependency failure

---

# 114. Scenario: Application Degraded

Start:

```bash
argocd app get <app>
```

Identify unhealthy resources.

Then:

```bash
kubectl get <resource>
kubectl describe <resource>
kubectl get events
```

If Pods are involved:

```bash
kubectl logs
```

If ALB is involved:

```text
Ingress
Controller
AWS ALB
Target health
```

---

# 115. Scenario: Argo CD API Server Down

Symptoms:

```text
UI unavailable
CLI unable to connect
```

Check:

```bash
kubectl get pods -n argocd
kubectl get svc -n argocd
kubectl logs deployment/argocd-server -n argocd
```

If HA replicas exist, verify:

```text
Pod availability
Service endpoints
Load balancer
Ingress
TLS
```

---

# 116. Scenario: Repo Server Down

Symptoms may include:

```text
Manifest generation failures
Application refresh failures
```

Check:

```bash
kubectl get pods -n argocd
kubectl logs deployment/argocd-repo-server -n argocd
```

Verify:

```text
Git connectivity
Credentials
Resource limits
Rendering tools
Repository size
```

---

# 117. Scenario: Application Controller Down

Symptoms may include:

```text
Applications stop reconciling
Syncs do not progress
State becomes stale
```

Check:

```bash
kubectl get pods -n argocd
```

and controller logs.

Because the controller is the reconciliation engine, this is a high-priority production failure.

---

# 118. Scenario: ApplicationSet Controller Down

Symptoms:

```text
Existing Applications may remain
New generated Applications are not created/updated
```

Check:

```bash
kubectl get applicationsets -A
kubectl get applications -A
kubectl logs deployment/argocd-applicationset-controller -n argocd
```

Distinguish:

```text
ApplicationSet generation
```

from:

```text
Application reconciliation
```

---

# 119. Scenario: Git Repository Unavailable

If Git becomes unavailable:

```text
Argo CD may not obtain new desired revisions
```

Existing workloads can continue running in Kubernetes.

This illustrates an important GitOps property:

```text
Git unavailable
!=
Immediate application outage
```

But new deployments and reconciliation against changed desired state can be affected.

---

# 120. Scenario: EKS API Unavailable

If the target EKS API is unavailable:

```text
Argo CD cannot reconcile live state
```

Existing Pods may continue running.

When the API becomes available again, Argo CD can resume reconciliation, subject to the actual state and configuration.

---

# 121. Failure Domain Thinking

Production Argo CD design should consider:

```text
Git provider failure
Argo CD failure
Redis failure
Repo Server failure
Controller failure
EKS API failure
AWS failure
Application failure
```

Each has different symptoms and recovery procedures.

---

# 122. Argo CD Monitoring

Monitor at least:

```text
API Server
Repo Server
Application Controller
ApplicationSet Controller
Redis
Applications
Repositories
Clusters
Sync failures
Health degradation
```

Monitoring should answer:

```text
Is GitOps operational?
Are applications synchronized?
Are applications healthy?
Are clusters reachable?
Are repositories reachable?
```

---

# 123. Argo CD Metrics

Argo CD exposes operational metrics that can be integrated into a monitoring stack.

For the user's environment:

```text
Argo CD
   |
   v
Prometheus
   |
   v
Grafana
```

Useful dashboards can show:

- Application sync status
- Health status
- Reconciliation behavior
- Controller performance
- Repository operations
- API server behavior

The exact metric names vary by Argo CD version.

---

# 124. Argo CD Logs

Logs are essential for root-cause analysis.

Example:

```bash
kubectl logs deployment/argocd-server -n argocd
kubectl logs deployment/argocd-repo-server -n argocd
```

For controller workloads, first identify the actual Kubernetes workload:

```bash
kubectl get pods -n argocd
```

Then inspect logs for the relevant Pod.

---

# 125. ELK Integration

The user's observability stack includes ELK.

A production architecture can centralize Argo CD logs:

```text
Argo CD Pods
    |
    v
Log Collection
    |
    v
Elasticsearch
    |
    v
Kibana
```

This makes it easier to correlate:

```text
Git deployment
+
Argo CD sync
+
Kubernetes events
+
Application logs
```

---

# 126. Prometheus + Grafana + Argo CD

A useful observability model:

```text
Argo CD
 |
 +--> Metrics --> Prometheus --> Grafana
 |
 +--> Logs ----> ELK
 |
 v
EKS
 |
 +--> Application metrics
 +--> Kubernetes metrics
```

This allows platform teams to monitor both:

```text
GitOps health
```

and:

```text
Application health
```

---

# 127. Argo CD Notifications

Argo CD can be integrated with notification mechanisms.

Typical events include:

```text
Sync succeeded
Sync failed
Application degraded
Application healthy
```

Potential destinations depend on configured integrations.

The operational goal is:

```text
Important GitOps events
        |
        v
Team notification
```

Avoid alerting on every insignificant event.

---

# 128. Production Alert Examples

Good alerts include:

```text
Production Application Degraded
Production Sync Failed
Target Cluster Unreachable
Repository Authentication Failed
Application OutOfSync for excessive duration
Argo CD Controller unavailable
Repo Server unavailable
```

The exact thresholds should reflect business impact.

---

# 129. Argo CD Upgrades

Argo CD should be upgraded deliberately.

Production process:

```text
Read release notes
 |
 v
Review breaking changes
 |
 v
Test in DEV
 |
 v
Test in QA
 |
 v
Backup/recovery verification
 |
 v
Production upgrade
 |
 v
Validate Applications
```

Do not upgrade the production GitOps control plane casually.

---

# 130. Upgrade Dependencies

When upgrading Argo CD, evaluate:

```text
Kubernetes version
EKS version
CRDs
Helm
Kustomize
Plugins
SSO
Ingress
Monitoring
ApplicationSet
Notifications
```

Also verify compatibility with existing Applications.

---

# 131. Argo CD Resource Sizing

Resource requests and limits should be configured based on workload.

Consider:

```text
Number of Applications
Number of clusters
Repository size
Manifest generation frequency
Sync frequency
Concurrent operations
HA requirements
```

Do not copy arbitrary production resource values from an unrelated environment.

---

# 132. Argo CD Capacity Planning

Suppose an organization has:

```text
20 applications
```

and another has:

```text
2,000 applications
```

They should not necessarily use the same Argo CD sizing.

Large installations should measure:

```text
Controller CPU
Controller memory
Repo Server CPU
Repo Server memory
API Server traffic
Redis behavior
Reconciliation latency
```

Then scale accordingly.

---

# 133. Argo CD Security Hardening

A production hardening checklist:

```text
TLS
SSO
RBAC
Least privilege
Protected repositories
Secure repository credentials
Restricted cluster credentials
Network policies
Restricted ingress
Audit logging
Secret encryption/management
Pod security
Image security
Regular upgrades
```

---

# 134. Network Architecture

A production EKS GitOps platform can look like:

```text
                   Internet / Corporate Network
                              |
                              v
                        ALB / Internal LB
                              |
                              v
                         Argo CD API
                              |
               +--------------+--------------+
               |                             |
               v                             v
          Repo Server                  Application Controller
               |                             |
               v                             v
              Git                        EKS API
                                             |
                                  +----------+----------+
                                  |          |          |
                                  v          v          v
                               DEV EKS     QA EKS     PROD EKS
```

The actual network topology depends on whether clusters are public/private and how connectivity is established.

---

# 135. Private EKS Control Plane

If EKS APIs are private:

```text
Argo CD
   |
   | Private network
   v
EKS private endpoint
```

This can reduce public exposure.

However, Argo CD must have network connectivity through the appropriate VPC/network architecture.

---

# 136. Private Git Connectivity

If Git is private:

```text
Repo Server
   |
   | Private connectivity
   v
Internal Git service
```

or access may occur through controlled outbound connectivity to a hosted Git provider.

Network design must support:

```text
Repo Server -> Git
```

without exposing unnecessary internal services.

---

# 137. Argo CD Ingress

The Argo CD UI/API can be exposed through an ingress/load balancer.

For AWS:

```text
Client
 |
 v
AWS ALB
 |
 v
Argo CD Ingress
 |
 v
argocd-server
```

TLS should be enabled for production access.

---

# 138. Authentication Flow

A typical enterprise flow:

```text
User
 |
 v
ALB / Ingress
 |
 v
Argo CD API Server
 |
 v
SSO / Identity Provider
 |
 v
Identity
 |
 v
Argo CD RBAC
 |
 v
Allowed action
```

The exact flow depends on the chosen identity provider.

---

# 139. Production Argo CD Architecture Summary

```text
                           Git
                            |
                            v
                     +-------------+
                     | Repo Server |
                     +-------------+
                            |
                     Render manifests
                            |
                            v
+---------+          +----------------------+
|  User   | --------> |    API Server       |
+---------+          +----------------------+
                            |
                         RBAC/SSO
                            |
                            v
                  +----------------------+
                  | Application Controller|
                  +----------------------+
                     |       |       |
                     v       v       v
                  DEV EKS  QA EKS  PROD EKS

ApplicationSet Controller
          |
          v
   Generates Applications

Redis
          |
          v
 Internal cache/coordination support
```

---

# 140. Complete Argo CD Mental Model

Think of Argo CD as five major responsibilities:

```text
1. SOURCE
   Git repositories

2. RENDER
   Helm / Kustomize / manifests

3. COMPARE
   Desired vs live

4. RECONCILE
   Apply desired state

5. OBSERVE
   Health / sync / events
```

Components support these responsibilities.

---

# 141. Component-to-Responsibility Mapping

```text
API Server
    |
    +--> Users / CLI / API / RBAC

Repo Server
    |
    +--> Git / Rendering

Application Controller
    |
    +--> Compare / Sync / Health

ApplicationSet Controller
    |
    +--> Application generation

Redis
    |
    +--> Cache/internal acceleration

SSO/Dex
    |
    +--> Identity integration

Notifications
    |
    +--> Event delivery
```

---

# 142. Interview: What Is the Most Important Argo CD Component?

### Answer

> The Application Controller is the core reconciliation component because it continuously evaluates Argo CD Applications, compares desired and live state, performs synchronization when appropriate, and evaluates resource health. However, Argo CD is a multi-component system, so the Repo Server, API Server, Redis, ApplicationSet Controller, and authentication/notification components have distinct responsibilities.

---

# 143. Interview: What Does the Repo Server Do?

### Answer

> The Repo Server handles repository access and manifest generation. It retrieves the configured Git revision and renders the application source using mechanisms such as Helm, Kustomize, or directory-based manifests. The resulting Kubernetes manifests are then used by Argo CD for comparison and synchronization.

---

# 144. Interview: What Does the Application Controller Do?

### Answer

> The Application Controller is the reconciliation engine. It watches Applications, determines desired state, compares it with live Kubernetes resources, performs synchronization according to policy, and evaluates resource health. It is the component most directly responsible for continuous GitOps reconciliation.

---

# 145. Interview: What Is the Difference Between API Server and Application Controller?

### Answer

> The API Server is the interface used by the UI, CLI, and API clients. It handles requests, authentication, and authorization. The Application Controller is the reconciliation engine that actually processes Applications and manages their desired state against Kubernetes.

---

# 146. Interview: Why Does Argo CD Use Redis?

### Answer

> Redis is used for internal caching and performance-related purposes. It is not the source of truth for desired application configuration. Git remains the durable desired-state source. Redis availability should still be considered when designing a production Argo CD installation.

---

# 147. Interview: What Is ApplicationSet?

### Answer

> ApplicationSet is a mechanism for generating and managing multiple Argo CD Application resources from generators and templates. It is particularly useful for multi-environment and multi-cluster deployments. For example, a list or cluster generator can create DEV, QA, and PROD Applications from one ApplicationSet.

---

# 148. Interview: Application vs ApplicationSet

### Answer

> An Application represents one deployment relationship between a source and a destination. An ApplicationSet generates multiple Applications using generators and a template. The ApplicationSet Controller creates the Applications, and the Application Controller then reconciles those Applications with Kubernetes.

---

# 149. Interview: How Does Argo CD Manage Multiple EKS Clusters?

### Answer

> A centralized Argo CD instance can register multiple target EKS clusters. Applications specify their destination cluster, and the Application Controller communicates with each cluster's Kubernetes API using the configured cluster credentials and permissions. ApplicationSets can generate Applications for multiple clusters, often using cluster labels to select environments such as DEV, QA, and PROD.

---

# 150. Interview: Why Use AppProjects?

### Answer

> AppProjects provide security and organizational boundaries for Applications. They can restrict which Git repositories an Application may use, which clusters it may target, which namespaces it may deploy into, and which resource types it may create. This reduces accidental or malicious blast radius.

---

# 151. Interview: What Is the Difference Between Sync and Refresh?

### Answer

> Refresh causes Argo CD to reevaluate source and live state. Sync is the operation that applies the desired state to the target cluster. A typical flow is Git change, refresh, OutOfSync detection, and then synchronization. Automated sync can perform the synchronization without manual intervention.

---

# 152. Interview: What Does OutOfSync Mean?

### Answer

> OutOfSync means Argo CD has identified a difference between desired state and live state. It does not necessarily mean the application is broken. The application could be healthy but drifted from Git. I would use `argocd app diff` to determine the exact difference before deciding whether to synchronize or correct the source.

---

# 153. Interview: Synced vs Healthy

### Answer

> Synced refers to configuration alignment between desired and live state. Healthy refers to runtime/resource health. An application can be Synced but Degraded, meaning the desired configuration was applied but the application is unhealthy. It can also be OutOfSync but Healthy when a manual change has created drift without immediately breaking the workload.

---

# 154. Interview: What Happens If the Git Repository Is Down?

### Answer

> Argo CD may be unable to retrieve new desired revisions or refresh applications based on new Git content. Existing Kubernetes workloads do not necessarily stop immediately because they are already running in the cluster. However, new deployments and desired-state reconciliation can be affected until repository access is restored.

---

# 155. Interview: What Happens If the Kubernetes API Is Down?

### Answer

> Argo CD cannot reliably read or modify the target cluster while the Kubernetes API is unavailable. Existing workloads may continue running, but reconciliation, health observation, and synchronization are affected. Once API connectivity returns, Argo CD can resume reconciliation.

---

# 156. Interview: What Is the Difference Between Centralized and Distributed Argo CD?

### Answer

> Centralized Argo CD uses one control plane to manage multiple clusters, providing centralized visibility and governance but creating a larger control-plane blast radius. Distributed Argo CD uses separate control planes for different environments or business boundaries, improving isolation at the cost of more operational overhead.

---

# 157. Interview Scenario: Application Is Not Updating After Git Commit

### Answer

I would investigate in this order:

```text
1. Verify Git commit and branch
2. Verify Application targetRevision
3. Check Argo CD refresh
4. Check Application status
5. Check repository connectivity
6. Run argocd app get
7. Run argocd app diff
8. Inspect Repo Server logs
9. Inspect Application Controller logs
10. Check sync policy
```

Useful commands:

```bash
argocd app get <app>
argocd app diff <app>
argocd repo list
kubectl logs deployment/argocd-repo-server -n argocd
```

---

# 158. Interview Scenario: All Applications Suddenly Stop Syncing

I would first suspect a shared control-plane dependency.

Check:

```text
Application Controller
Repo Server
Kubernetes API connectivity
Redis
Network
Git repository
```

Commands:

```bash
kubectl get pods -n argocd
kubectl get applications -A
```

Then inspect:

```bash
kubectl logs <application-controller-pod> -n argocd
kubectl logs deployment/argocd-repo-server -n argocd
```

If all Applications fail simultaneously, a common infrastructure dependency is more likely than independent application failures.

---

# 159. Interview Scenario: Only One Application Fails

If only one Application fails, investigate application-specific configuration first:

```text
Repository path
Revision
Helm values
Kustomize overlay
Destination
Project
Resource
RBAC
Application dependencies
```

This contrasts with a fleet-wide failure where shared Argo CD components are more suspicious.

---

# 160. Production Runbook: Application Sync Failure

```text
Step 1
argocd app get <app>

Step 2
argocd app diff <app>

Step 3
Identify failing resource

Step 4
kubectl describe <resource>

Step 5
kubectl get events

Step 6
Inspect controller/repo logs

Step 7
Correct Git or platform issue

Step 8
Sync after validation

Step 9
Verify health

Step 10
Confirm application traffic
```

Never skip the root-cause analysis just because a manual sync is available.

---

# 161. Production Runbook: Argo CD Control Plane Failure

```text
1. Check all Argo CD Pods
2. Identify failed component
3. Check recent restarts
4. Inspect logs
5. Check resource pressure
6. Check Redis
7. Check Git connectivity
8. Check Kubernetes API connectivity
9. Check networking
10. Restore failed component
11. Validate Applications
12. Review alerts/incidents
```

---

# 162. Production Runbook: Cluster Unreachable

```bash
argocd cluster list
```

Then determine:

```text
Which cluster?
When did it fail?
Is EKS healthy?
Is API endpoint reachable?
Are credentials valid?
Did RBAC change?
Did network paths change?
```

Do not modify Applications until the cluster access problem is understood.

---

# 163. Production Runbook: Repository Unreachable

```bash
argocd repo list
```

Check:

```text
URL
Credential
Authentication
DNS
Network
Certificate
Repository existence
Revision
```

Then inspect Repo Server logs.

---

# 164. Production Runbook: Controller Resource Pressure

If Application Controller restarts:

```bash
kubectl get pods -n argocd
kubectl describe pod <controller-pod> -n argocd
```

Look for:

```text
OOMKilled
CPU throttling
Node pressure
Probe failure
Scheduling failure
```

Then evaluate:

```text
Application count
Cluster count
Reconciliation load
Resource sizing
```

---

# 165. Argo CD Architecture — Practical Mental Model

Remember:

```text
API Server
=
"Talk to Argo CD"

Repo Server
=
"Get and render Git"

Application Controller
=
"Reconcile Applications"

ApplicationSet Controller
=
"Create Applications"

Redis
=
"Cache/internal support"

Kubernetes API
=
"Apply/read runtime resources"

Git
=
"Desired-state source"
```

This model is sufficient to orient yourself during most production incidents.

---

# 166. RoboShop Argo CD Architecture

For the user's RoboShop platform:

```text
                         GitHub
                           |
                           v
                   GitOps Repository
                           |
                           v
                      Repo Server
                           |
                           v
                   Render Helm Charts
                           |
                           v
                  Application Controller
                           |
                           v
                         EKS
                           |
       +-------------------+-------------------+
       |                   |                   |
       v                   v                   v
   Deployment           Service             Ingress
       |                                       |
       v                                       v
     Pods                          AWS Load Balancer Controller
                                               |
                                               v
                                              ALB
```

CI remains separate:

```text
Jenkins / GitHub Actions
        |
        v
ECR
        |
        v
GitOps repository update
        |
        v
Argo CD
```

---

# 167. RoboShop Multi-Cluster Extension

A larger architecture:

```text
                           GitOps Repo
                                |
                                v
                         Central Argo CD
                                |
                +---------------+---------------+
                |               |               |
                v               v               v
             EKS-DEV         EKS-QA          EKS-PROD
                |               |               |
             RoboShop        RoboShop        RoboShop
                |               |               |
               ALB             ALB             ALB
```

ApplicationSets can later automate this fleet.

---

# 168. Production Ownership Model

A recommended ownership split for the RoboShop environment is:

```text
Terraform
 |
 +--> VPC
 +--> EKS
 +--> IAM
 +--> AWS infrastructure

Argo CD
 |
 +--> Kubernetes applications
 +--> Helm values
 +--> Ingress
 +--> HPA
 +--> Policies

AWS Controllers
 |
 +--> ALB resources

Kubernetes Controllers
 |
 +--> ReplicaSets
 +--> Pods
 +--> HPA runtime scaling
```

This avoids overlapping ownership.

---

# 169. Common Architecture Mistakes

### Mistake 1

Using Argo CD to manage Terraform infrastructure directly as if it were Kubernetes.

Better:

```text
Terraform -> AWS/EKS infrastructure
Argo CD   -> Kubernetes workloads
```

unless a deliberate Terraform-controller architecture is used.

### Mistake 2

Allowing every Application to target every cluster.

Better:

```text
AppProject restrictions
```

### Mistake 3

Giving every developer production sync rights.

Better:

```text
RBAC + approvals
```

### Mistake 4

Storing production passwords directly in Git.

Better:

```text
External secret-management architecture
```

### Mistake 5

Assuming Synced means Healthy.

Better:

```text
Check both sync and health.
```

---

# 170. Advanced Architecture Principle: Separate Control Plane and Workload Plane

For centralized management:

```text
Management Plane
    |
    +--> Argo CD
    +--> GitOps
    +--> Monitoring

Workload Plane
    |
    +--> EKS-DEV
    +--> EKS-QA
    +--> EKS-PROD
```

This makes architectural boundaries easier to reason about.

---

# 171. Advanced Architecture Principle: GitOps Is Pull-Oriented

CI does not need direct production Kubernetes credentials for normal deployment.

Instead:

```text
CI
 |
 v
ECR + Git
```

and:

```text
Argo CD
 |
 v
EKS
```

This reduces the number of systems that require production cluster write access.

---

# 172. Security Benefit

Traditional pipeline:

```text
CI Server
 |
 | Production cluster credentials
 v
EKS
```

GitOps:

```text
CI
 |
 +--> ECR
 +--> Git

Argo CD
 |
 +--> EKS
```

This separates:

```text
Build trust
```

from:

```text
Deployment trust
```

which can reduce blast radius.

---

# 173. Advanced Architecture Principle: Git Commit as Deployment Intent

A GitOps deployment can be represented as:

```text
Commit abc123
```

meaning:

```text
Deploy this desired state.
```

The commit becomes a useful audit reference.

Example:

```text
Production incident
      |
      v
Argo CD revision
      |
      v
Git commit
      |
      v
Pull Request
      |
      v
Author / reviewer
```

---

# 174. Production Change Management

A production change should ideally follow:

```text
Change request
     |
     v
Developer PR
     |
     v
CI validation
     |
     v
Security checks
     |
     v
Review
     |
     v
Merge
     |
     v
Argo CD
     |
     v
EKS
```

This creates controlled, traceable delivery.

---

# 175. Summary

Argo CD is not just a command-line deployment tool.

It is a Kubernetes-native GitOps control plane composed of multiple cooperating components.

The most important architecture is:

```text
Git
 |
 v
Repo Server
 |
 v
Rendered Desired State
 |
 v
Application Controller
 |
 v
Kubernetes API
 |
 v
EKS
```

The supporting control plane includes:

```text
API Server
ApplicationSet Controller
Redis
SSO/Dex where configured
Notifications where configured
```

The most important operational distinction is:

```text
API Server
    =
Interface

Repo Server
    =
Source + Rendering

Application Controller
    =
Reconciliation

ApplicationSet Controller
    =
Application Generation
```

For production AWS/EKS environments, Argo CD can operate as a centralized GitOps control plane managing multiple clusters and accounts, provided cluster access, RBAC, repository permissions, project boundaries, and failure domains are carefully designed.

---

# 176. Next File

The next file is:

```text
05-ArgoCD-Installation-and-Configuration.md
```

It will move from architecture into hands-on implementation:

- EKS prerequisites
- Argo CD installation
- Namespace
- Installation methods
- Helm installation
- Manifest installation
- Production configuration
- Server exposure
- ALB Ingress
- TLS
- CLI setup
- Initial admin access
- Repository configuration
- Private Git repositories
- Cluster registration
- EKS authentication
- RBAC
- Projects
- Production hardening
- Resource sizing
- HA configuration
- Verification commands
- Upgrade considerations
- Installation troubleshooting
- Production installation runbook
- Interview questions
