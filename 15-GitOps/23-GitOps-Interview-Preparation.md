# GitOps with Argo CD — Interview Preparation

## 1. Purpose

This file is the final interview-preparation module for the GitOps with Argo CD section.

It is designed for DevOps / Cloud DevOps / DevSecOps interviews involving:

- GitOps
- Argo CD
- Kubernetes
- AWS EKS
- Helm
- Kustomize
- ApplicationSet
- multi-cluster management
- multi-account AWS
- CI/CD
- DevSecOps
- ECR
- ALB Ingress
- Terraform
- RBAC
- secrets
- observability
- production troubleshooting
- disaster recovery
- architecture design

The objective is not to memorize definitions.

The objective is to explain:

```text
what
why
how
architecture
trade-offs
failure scenarios
production decisions
```

---

# 2. Interview Answer Framework

For most GitOps questions use:

```text
1. Definition
2. Why it is needed
3. How it works
4. Production example
5. Security/reliability consideration
```

Example:

> GitOps is a deployment model where declarative desired state is stored in Git and a reconciliation controller such as Argo CD continuously compares that desired state with the Kubernetes cluster. It is useful because it provides auditability, drift detection, repeatability and separation between CI and production cluster access. In my EKS environment, CI builds and scans images, pushes them to ECR, updates the GitOps repository, and Argo CD reconciles the approved state into EKS.

---

# 3. What Is GitOps?

### Answer

GitOps is an operating model where the desired state of infrastructure or applications is declared in Git and a reconciliation system continuously works to make the runtime environment match that state.

Core ideas:

```text
Git = source of truth
desired state = declarative
reconciliation = continuous
changes = Git-based
audit = Git history
```

---

# 4. Why Do We Need GitOps?

Traditional deployment may look like:

```text
Jenkins
   |
   v
kubectl apply
   |
   v
Kubernetes
```

Problems can include:

```text
cluster credentials in CI
manual changes
configuration drift
weak auditability
different deployment methods
```

GitOps provides:

```text
declarative state
reviewable changes
continuous reconciliation
drift detection
repeatability
rollback through Git
```

---

# 5. What Is the Source of Truth?

### Answer

The GitOps repository containing the approved desired deployment state is the source of truth.

For example:

```yaml
image:
  repository: ECR/cart
  tag: "1.4.2"
```

This tells Argo CD which version should run.

The running cluster is the actual state.

---

# 6. Desired State vs Actual State

Desired:

```text
Git:
cart replicas = 3
image = 1.4.2
```

Actual:

```text
EKS:
cart replicas = 2
image = 1.4.1
```

Argo CD detects:

```text
difference
```

and reconciles toward:

```text
3 replicas
1.4.2
```

---

# 7. What Is Reconciliation?

Reconciliation means continuously comparing:

```text
desired state
```

with:

```text
live state
```

and taking action when they differ.

This is fundamental to both GitOps and Kubernetes controller architecture.

---

# 8. What Is Configuration Drift?

Configuration drift occurs when the actual environment differs from the declared desired configuration.

Example:

```bash
kubectl scale deployment cart --replicas=5
```

while Git says:

```yaml
replicas: 3
```

The cluster has drifted from Git.

---

# 9. How Does Argo CD Correct Drift?

If automated sync and self-healing are configured appropriately:

```text
Git = 3 replicas
EKS = 5 replicas
       |
       v
Argo CD detects difference
       |
       v
reconciliation
       |
       v
EKS = 3
```

However, intentional controller-managed fields should not be blindly overwritten.

---

# 10. GitOps Pull vs Push

Push model:

```text
CI
 |
 | credentials
 v
Kubernetes
```

Pull model:

```text
Git
 |
 v
Argo CD
 |
 v
Kubernetes
```

Argo CD pulls desired state and applies it from inside the cluster/control plane.

---

# 11. Why Is Pull-Based Deployment More Secure?

The CI system does not necessarily need direct production cluster credentials.

Instead:

```text
CI → ECR
CI → GitOps Git

Argo CD → EKS
```

This separates:

```text
artifact creation
```

from:

```text
cluster deployment
```

---

# 12. Is GitOps Only for Kubernetes?

No.

The principles can be used for:

```text
Kubernetes
infrastructure
cloud configuration
policy
applications
platform configuration
```

Argo CD is primarily Kubernetes-focused.

---

# 13. GitOps vs CI/CD

CI:

```text
build
test
scan
package
publish
```

GitOps CD:

```text
desired state
reconciliation
deployment
drift correction
```

---

# 14. Should CI Deploy Directly to Kubernetes?

In a mature GitOps model, normally no.

Preferred:

```text
CI
 |
 v
ECR
 |
 v
GitOps PR
 |
 v
Argo CD
 |
 v
EKS
```

This provides stronger separation of duties.

---

# 15. What Does Argo CD Do?

Argo CD:

```text
reads Git
renders desired manifests
compares desired/live state
reports status
syncs resources
detects drift
supports rollback/history
```

It is a Kubernetes continuous delivery/GitOps controller.

---

# 16. Argo CD Architecture

Main components:

```text
API Server
Application Controller
Repo Server
Redis
ApplicationSet Controller
```

plus integrations with:

```text
Git repositories
Kubernetes API servers
identity providers
notifications
```

---

# 17. What Does API Server Do?

The API Server provides the interface used by:

```text
CLI
UI
automation
external integrations
```

It handles operations such as:

```text
authentication
authorization
application operations
repository operations
cluster operations
```

---

# 18. What Does Application Controller Do?

The Application Controller is the reconciliation engine.

It:

```text
watches Applications
compares desired/live state
calculates sync status
tracks health
initiates synchronization
```

This is one of the most important Argo CD components.

---

# 19. What Does Repo Server Do?

Repo Server is responsible for repository-related manifest generation.

It can work with sources such as:

```text
plain YAML
Helm
Kustomize
```

It retrieves repository content and generates the Kubernetes manifests Argo CD needs.

---

# 20. What Is Redis Used For?

Redis is used by Argo CD for caching/internal state-related operations.

It should be treated as an internal Argo CD dependency, not as the application database.

Production availability requirements should be considered according to the Argo CD deployment mode.

---

# 21. What Does ApplicationSet Controller Do?

ApplicationSet Controller generates Argo CD Applications from templates and generators.

Useful for:

```text
multiple environments
multiple clusters
many services
fleet management
```

---

# 22. Application vs ApplicationSet

Application:

```text
one desired application deployment
```

ApplicationSet:

```text
generator that creates multiple Applications
```

Example:

```text
ApplicationSet
 |
 +--> cart-dev
 +--> cart-qa
 +--> cart-prod
```

---

# 23. What Is an Argo CD Project?

An AppProject provides a logical/security boundary.

It can restrict:

```text
source repositories
destination clusters
destination namespaces
resource types
```

It is an important enterprise security control.

---

# 24. Why Use AppProjects?

Without proper restrictions, an application could potentially be given access to more resources than necessary.

With a Project:

```text
RoboShop project
 |
 +--> approved Git repo
 +--> approved EKS cluster
 +--> approved namespace
 +--> approved resource types
```

---

# 25. What Is an Argo CD Application?

An Application describes:

```text
where desired configuration is stored
```

and:

```text
where it should be deployed
```

Core areas:

```text
project
source
destination
sync policy
```

---

# 26. Application Manifest

Typical:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cart
  namespace: argocd
spec:
  project: roboshop

  source:
    repoURL: https://github.com/example/gitops.git
    targetRevision: main
    path: helm/cart

  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

# 27. What Is repoURL?

`repoURL` identifies the source Git repository.

Example:

```yaml
repoURL: https://github.com/example/roboshop-gitops.git
```

For private repositories, Argo CD needs repository credentials.

---

# 28. What Is targetRevision?

It identifies the Git revision to use.

Examples:

```text
main
v1.2.0
commit SHA
tag
branch
```

Production strategy should favor controlled, auditable revisions.

---

# 29. What Is path?

It tells Argo CD where the manifests/source configuration lives.

Example:

```yaml
path: helm/cart
```

---

# 30. What Is destination?

It specifies the target Kubernetes cluster and namespace.

Example:

```yaml
destination:
  server: https://kubernetes.default.svc
  namespace: roboshop
```

For multi-cluster:

```yaml
destination:
  server: '{{server}}'
```

can be generated by an ApplicationSet.

---

# 31. What Is syncPolicy?

It controls synchronization behavior.

Example:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

---

# 32. Automated Sync

Automated sync means Argo CD can synchronize detected Git changes without a human clicking Sync.

Useful for:

```text
development
low-risk workloads
highly automated environments
```

Production may require additional controls.

---

# 33. Manual Sync

Manual sync requires an operator or automation to initiate synchronization.

Useful for:

```text
production approvals
maintenance windows
high-risk changes
regulated environments
```

---

# 34. What Is Prune?

Prune removes resources that are no longer part of the desired state.

Example:

Git previously contains:

```text
Deployment
Service
ConfigMap
```

Service is removed from Git.

With pruning enabled, Argo CD can remove the corresponding live resource.

---

# 35. Why Is Prune Dangerous?

A bad Git commit can remove resources.

Therefore production should use:

```text
protected branches
reviews
AppProjects
controlled permissions
careful sync settings
```

---

# 36. What Is Self-Heal?

Self-heal causes Argo CD to correct certain live-state modifications that differ from Git.

Example:

```text
Git = replicas 3
Live = replicas 5
```

Argo CD can restore:

```text
replicas 3
```

if that field is Git-owned and self-healing is enabled.

---

# 37. What Is Sync Status?

Common states:

```text
Synced
OutOfSync
Unknown
```

Synced means desired and live state are considered aligned.

OutOfSync means there is a difference.

Unknown indicates Argo CD cannot determine the state reliably.

---

# 38. What Is Health Status?

Typical health states include:

```text
Healthy
Progressing
Degraded
Suspended
Missing
Unknown
```

Health and sync are different concepts.

---

# 39. Synced but Degraded

Possible:

```text
Sync status = Synced
Health = Degraded
```

This means Argo CD applied the desired configuration, but the runtime resource is unhealthy.

Example:

```text
Deployment accepted
Pods failing readiness
```

---

# 40. OutOfSync but Healthy

Possible:

```text
Sync = OutOfSync
Health = Healthy
```

The workload may currently be functioning, but live configuration differs from Git.

---

# 41. What Does Progressing Mean?

A resource is in transition.

Example:

```text
new Deployment
   |
   v
new Pods starting
   |
   v
readiness pending
```

If Progressing remains indefinitely, investigate the workload.

---

# 42. What Does Degraded Mean?

The resource is not meeting expected health conditions.

Possible causes:

```text
CrashLoopBackOff
ImagePullBackOff
readiness failure
insufficient resources
failed dependency
```

---

# 43. What Does Unknown Mean?

Argo CD may not have enough information to determine health/status.

Investigate:

```text
cluster connectivity
controller errors
resource type
custom health logic
API access
```

---

# 44. How Does Argo CD Determine Health?

It evaluates resource status and health information.

For standard Kubernetes resources it uses resource-specific status.

For custom resources, health behavior may depend on the controller and Argo CD configuration/version.

---

# 45. What Is Refresh?

Refresh causes Argo CD to re-evaluate repository/live information.

There are different refresh behaviors and cache considerations.

Conceptually:

```text
Git changed
 |
 v
refresh
 |
 v
new desired state
```

---

# 46. What Is Reconciliation?

Reconciliation is the continuous process of converging live state toward desired state.

It is not the same as a one-time deployment.

---

# 47. What Is Drift Detection?

Drift detection identifies differences between:

```text
Git desired state
```

and:

```text
Kubernetes live state
```

---

# 48. What Are Ignore Differences?

Some fields can be intentionally ignored when they are modified by another controller.

Example:

```text
HPA changes replicas
```

If Argo CD is expected to tolerate that controller-owned field, configuration can account for the difference.

Do not use ignore rules to hide legitimate configuration drift.

---

# 49. What Are Sync Options?

Sync options modify synchronization behavior.

Common examples include:

```text
CreateNamespace=true
PruneLast=true
ServerSideApply=true
```

Use only options required by the workload.

---

# 50. What Are Sync Waves?

Sync waves allow resource ordering.

Example:

```text
wave -2:
Namespace

wave -1:
Config/secret dependencies

wave 0:
Application

wave 1:
Ingress
```

Use ordering when dependencies require it.

---

# 51. What Are Hooks?

Hooks are resources/actions associated with lifecycle events.

Common concepts include:

```text
PreSync
Sync
PostSync
SyncFail
```

Example:

```text
PreSync migration
       ↓
application deployment
       ↓
PostSync validation
```

Use hooks carefully because failure can block deployments.

---

# 52. What Is a Rollback?

Rollback returns the application to a previous known version.

Possible methods:

```text
Argo CD history/rollback
Git revert
```

For GitOps, a Git revert is often preferable as the durable source-of-truth change.

---

# 53. Why Is Git Revert Often Better?

Suppose production is:

```text
Git commit B
```

and B is broken.

Reverting to:

```text
commit A
```

makes Git explicitly represent the desired state again.

Then:

```text
Argo CD
   |
   v
A
```

The system remains auditable.

---

# 54. What Is Resource Tracking?

Argo CD needs to identify which resources belong to an Application.

Resource tracking prevents confusion when many Applications share a cluster.

---

# 55. What Is App of Apps?

App of Apps uses one parent Application to manage child Applications.

Example:

```text
platform-root
 |
 +--> ingress
 +--> monitoring
 +--> secrets
 +--> roboshop
```

---

# 56. What Is ApplicationSet?

ApplicationSet generates Applications dynamically.

Example:

```text
one template
+
cluster generator
=
many Applications
```

---

# 57. ApplicationSet Generators

Know these:

```text
List
Git
Directory
Cluster
Matrix
Merge
Pull Request
SCM-related generators
```

---

# 58. List Generator

Use when values are explicitly defined.

Example:

```text
dev
qa
prod
```

Advantages:

```text
simple
predictable
explicit
```

---

# 59. Git Generator

Uses Git repository content to generate Applications.

Useful when repository structure itself defines deployment units.

---

# 60. Directory Generator

Useful when directories represent applications.

Example:

```text
apps/
├── cart
├── catalogue
├── orders
```

Each directory can become an Application.

---

# 61. Cluster Generator

Uses registered Argo CD clusters.

Useful for:

```text
multi-cluster
environment labels
regional deployment
fleet management
```

---

# 62. Matrix Generator

Combines generators.

Conceptually:

```text
environment
   ×
cluster
```

could generate:

```text
dev-cluster-a
dev-cluster-b
prod-cluster-a
prod-cluster-b
```

when permitted by the design.

---

# 63. Merge Generator

Combines generator results according to matching keys.

Useful when multiple sources of metadata need to be combined.

---

# 64. Pull Request Generator

Useful for preview environments.

Concept:

```text
Pull Request
    |
    v
temporary Application
    |
    v
preview namespace
```

When the PR closes, the preview environment can be cleaned up according to the ApplicationSet lifecycle.

---

# 65. Multi-Cluster Argo CD

### Interview answer

Argo CD can manage multiple Kubernetes clusters from a central control plane. Target clusters are registered, and Applications specify the destination cluster. ApplicationSets can dynamically generate Applications using cluster labels.

---

# 66. Can One Argo CD Manage Multiple EKS Clusters?

Yes.

Architecture:

```text
                    Argo CD
                   /   |   \
                  v    v    v
                EKS1 EKS2 EKS3
```

The Argo CD control plane needs appropriate access to each target cluster.

---

# 67. How Would You Manage DEV/QA/PROD?

Use:

```text
separate EKS clusters
or
strong namespace isolation where appropriate
```

and:

```text
AppProjects
ApplicationSets
environment-specific values
protected Git changes
```

---

# 68. How Do You Select Production Clusters?

Use cluster metadata such as:

```text
environment=prod
```

and ApplicationSet Cluster Generator selectors.

---

# 69. How Do You Prevent Dev Applications From Deploying to Prod?

Use multiple layers:

```text
AppProject destination restrictions
RBAC
Git permissions
CODEOWNERS
cluster labels
repository separation where required
```

Never rely on one control alone for critical boundaries.

---

# 70. Multi-Account EKS Interview Answer

> I would normally isolate production into a separate AWS account. Argo CD can run in a controlled management environment and connect to registered EKS clusters. I would restrict cluster access using least privilege and use AppProjects to constrain which Applications can target production clusters and namespaces.

---

# 71. Central vs Per-Cluster Argo CD

Central:

```text
one control plane
many clusters
```

Advantages:

```text
central governance
single UI
less duplicated configuration
```

Risks:

```text
larger management-plane blast radius
```

Per cluster:

```text
strong isolation
independent failure domains
```

Tradeoff:

```text
more operational overhead
```

---

# 72. When Would You Use Multiple Argo CD Instances?

Consider:

```text
regulatory isolation
business-unit isolation
regional autonomy
high blast-radius concerns
separate security domains
```

---

# 73. How Does Argo CD Authenticate to EKS?

At a high level:

```text
Argo CD
 |
 v
Kubernetes API
 |
 v
authentication
 |
 v
authorization/RBAC
```

The exact EKS authentication mechanism depends on the EKS configuration and supported Argo CD setup.

Never use unnecessary cluster-admin permissions.

---

# 74. What Is RBAC?

RBAC controls:

```text
who
can perform
which action
on which resource
```

---

# 75. Argo CD RBAC vs Kubernetes RBAC

Argo CD RBAC controls access to Argo CD resources and operations.

Kubernetes RBAC controls access to Kubernetes API resources.

They are separate layers.

---

# 76. Why Do We Need Both?

Example:

```text
User
 |
 v
Argo CD RBAC
 |
 v
Application
 |
 v
Kubernetes API
 |
 v
Kubernetes RBAC
```

Defense in depth is important.

---

# 77. SSO Interview Answer

Use an identity provider with Argo CD SSO integration.

Concept:

```text
User
 |
 v
Identity Provider
 |
 v
Argo CD
 |
 v
RBAC
```

Map groups/claims to appropriate Argo CD roles.

---

# 78. Production Repository Security

Use:

```text
private repositories
branch protection
required reviewers
CODEOWNERS
secret scanning
least privilege
credential rotation
```

---

# 79. How Do You Handle Private Git Repositories?

Configure Argo CD repository credentials using approved authentication methods:

```text
SSH
HTTPS/token
```

Credentials should be stored securely and rotated.

---

# 80. SSH vs HTTPS for Git

Both can work.

SSH:

```text
deploy key/SSH credential
```

HTTPS:

```text
token/credential
```

Choose based on enterprise security standards and operational requirements.

---

# 81. Why Should Secrets Not Be Stored in Git?

Even private Git is not automatically a secure secret vault.

Problems:

```text
Git history persists
accidental exposure
broad repository access
credential reuse
```

Use a dedicated secret-management solution.

---

# 82. How Would You Manage AWS Secrets?

Typical architecture:

```text
AWS Secrets Manager
        |
        v
External Secrets Operator
        |
        v
Kubernetes Secret
        |
        v
Pod
```

Git contains the reference/configuration rather than the secret value.

---

# 83. GitOps Security Interview Question

### What if someone compromises the GitOps repository?

They could potentially change desired state.

Mitigations:

```text
MFA
protected branches
PR approvals
CODEOWNERS
signed commits where required
least-privilege Git access
monitoring
audit
policy checks
```

---

# 84. Image Security

Use:

```text
SonarQube
Trivy
Veracode
ECR
immutable tags
digests
```

Do not rely only on a successful image build.

---

# 85. Image Tag vs Digest

Tag:

```text
cart:1.2.3
```

Digest:

```text
cart@sha256:...
```

Digest is immutable for a specific image artifact.

For high-assurance production systems, digest pinning can provide stronger reproducibility.

---

# 86. Should We Use Latest?

Avoid `latest` for production.

Why?

```text
mutable
hard to audit
hard to reproduce
rollback ambiguity
```

---

# 87. How Does CI Update GitOps?

A typical process:

```text
CI builds image
        |
        v
push ECR
        |
        v
update image tag/digest
        |
        v
Git commit/PR
        |
        v
review
        |
        v
Argo CD
```

---

# 88. Should CI Directly Push to Main?

Production organizations often prefer:

```text
Pull Request
+
automated checks
+
approval
+
merge
```

rather than unrestricted direct writes.

---

# 89. What Is the Best Branching Strategy?

There is no universal answer.

Common approach:

```text
main
```

with:

```text
PR
review
validation
merge
```

Environment branches can work but may introduce promotion and merge complexity.

---

# 90. How Do You Manage Environment Differences?

Use:

```text
Helm values
Kustomize overlays
Application parameters
ApplicationSet variables
```

Keep common configuration centralized and environment-specific differences explicit.

---

# 91. Helm vs Kustomize

Helm:

```text
templates
packaging
values
reusable charts
```

Kustomize:

```text
base
overlays
patches
native Kubernetes
```

Choose based on application/team needs.

---

# 92. Can Argo CD Use Helm?

Yes.

Argo CD can render Helm charts and then apply the resulting Kubernetes manifests.

Important:

> Argo CD uses Helm primarily as a manifest generation mechanism; Argo CD remains responsible for GitOps synchronization.

---

# 93. Can Argo CD Use Kustomize?

Yes.

Argo CD can render Kustomize configurations and reconcile the resulting manifests.

---

# 94. What Happens During an Argo CD Sync?

Conceptually:

```text
Git revision
    |
    v
Source retrieval
    |
    v
Manifest generation
    |
    v
Desired/live comparison
    |
    v
Sync planning
    |
    v
Kubernetes API
    |
    v
Resource reconciliation
    |
    v
Health evaluation
```

---

# 95. What Happens If Sync Fails?

Start with:

```bash
argocd app get <app>
argocd app diff <app>
```

Then:

```bash
kubectl describe application <app> -n argocd
kubectl get events -n <namespace>
```

Then inspect:

```text
resource errors
RBAC
manifest rendering
cluster connectivity
hooks
dependencies
```

---

# 96. Application Stuck OutOfSync

Troubleshooting:

```bash
argocd app get <app>
argocd app diff <app>
```

Determine:

```text
which resource differs?
which field differs?
is it intentional?
is another controller changing it?
```

Then fix ownership or desired configuration.

---

# 97. Application Stuck Progressing

Check:

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl get events -n <namespace>
```

Common causes:

```text
readiness failure
startup delay
image pull
resource shortage
dependency
configuration
```

---

# 98. Application Degraded

First identify the degraded resource:

```bash
argocd app get <app>
```

Then:

```bash
kubectl get <resource>
kubectl describe <resource>
kubectl get events
```

---

# 99. Missing Resource

If Argo says a resource is missing:

Check:

```text
Git path
manifest
namespace
Application source
sync
prune
resource permissions
```

---

# 100. Helm Rendering Failure

Run:

```bash
helm lint <chart>
helm template <release> <chart>
```

Check:

```text
values
template syntax
missing files
chart dependencies
API versions
```

---

# 101. Git Authentication Failure

Symptoms:

```text
repository unavailable
failed to fetch
authentication error
```

Check:

```bash
argocd repo list
```

Then verify:

```text
credential
SSH key/token
repository URL
network
certificate
permissions
```

---

# 102. Kubernetes API Connection Failure

Check:

```bash
argocd cluster list
```

Then:

```text
cluster endpoint
credentials
RBAC
network path
EKS status
authentication
```

---

# 103. Permission Denied

If sync fails due to RBAC:

Determine:

```text
Argo CD project permissions
cluster role
service account
destination
resource kind
namespace
```

Do not immediately grant cluster-admin.

---

# 104. ApplicationSet Not Generating

Check:

```bash
kubectl get applicationset -n argocd
kubectl describe applicationset <name> -n argocd
```

Then:

```text
generator
selector
cluster labels
Git path
template
controller logs
```

---

# 105. Multi-Cluster Deployment Failure

Check:

```bash
argocd cluster list
argocd app get <app>
```

Then:

```text
target server
cluster registration
credentials
RBAC
network
namespace
AppProject destination
```

---

# 106. Controller Troubleshooting

Check Argo CD Pods:

```bash
kubectl get pods -n argocd
```

Then logs for relevant components:

```bash
kubectl logs -n argocd <pod>
```

Important components:

```text
argocd-server
argocd-application-controller
argocd-repo-server
argocd-applicationset-controller
```

Names can vary slightly by deployment/version.

---

# 107. Repo Server Troubleshooting

Look for:

```text
Git errors
Helm errors
Kustomize errors
credential errors
resource limits
network errors
```

Commands:

```bash
kubectl logs -n argocd deployment/argocd-repo-server
kubectl describe pod -n argocd <repo-server-pod>
```

---

# 108. Application Controller Troubleshooting

Look for:

```text
reconciliation errors
API errors
resource tracking
sync failures
cluster connection
```

Commands:

```bash
kubectl logs -n argocd deployment/argocd-application-controller
```

Controller deployment details can vary with the installed Argo CD version.

---

# 109. Sync Wave Failure

If wave 0 depends on wave -1:

```text
wave -1 failed
```

then later resources may not progress as expected.

Check:

```text
hook
resource health
wave annotation
dependency
```

---

# 110. Hook Failure

If a PreSync hook fails:

```text
application sync may stop
```

Check:

```bash
kubectl get jobs -n <namespace>
kubectl describe job <job>
kubectl logs job/<job> -n <namespace>
```

---

# 111. Rollback Failure

Possible causes:

```text
image unavailable
database schema incompatible
resource missing
configuration dependency
old manifest invalid
```

Important:

> Application rollback does not automatically roll back external data changes.

---

# 112. Database Rollback Question

### Does Argo CD rollback the database?

No.

Argo CD can restore Kubernetes desired state.

Database schema/data recovery requires a separate migration and backup strategy.

---

# 113. Sync Succeeds but Pods Fail

This is a common interview scenario.

Answer:

> A successful Argo CD sync only proves that desired Kubernetes resources were applied successfully. It does not guarantee application runtime health. I would check Deployment status, Pod events, probes, logs, Services, EndpointSlices, Ingress/ALB and application dependencies.

---

# 114. ALB Troubleshooting

Commands:

```bash
kubectl get ingress -n roboshop
kubectl describe ingress roboshop -n roboshop
kubectl get svc -n roboshop
```

Check controller logs:

```bash
kubectl logs -n kube-system deployment/aws-load-balancer-controller
```

Investigate:

```text
IngressClass
subnets
IAM
security groups
target health
annotations
```

---

# 115. ECR Troubleshooting

If:

```text
ImagePullBackOff
```

check:

```bash
kubectl describe pod <pod> -n <namespace>
```

Investigate:

```text
repository
tag
digest
ECR authorization
network
node/pod AWS identity
```

---

# 116. HPA Troubleshooting

Check:

```bash
kubectl get hpa -n <namespace>
kubectl describe hpa <name> -n <namespace>
```

Possible:

```text
metrics unavailable
requests missing
CPU target mismatch
capacity issue
```

---

# 117. Argo CD vs Jenkins

### Why Argo CD?

Jenkins is excellent for:

```text
CI
automation
pipelines
builds
tests
```

Argo CD is specialized for:

```text
GitOps
continuous reconciliation
Kubernetes deployment
drift detection
```

They can complement each other.

---

# 118. Argo CD vs Flux

Both support Kubernetes GitOps.

Comparison:

| Area | Argo CD | Flux |
|---|---|---|
| GitOps | Yes | Yes |
| Kubernetes | Strong | Strong |
| UI | Rich built-in UI | More ecosystem/tooling dependent |
| ApplicationSet | Yes | Different model |
| Multi-cluster | Yes | Yes |
| Enterprise RBAC | Strong | Strong |
| CLI | Yes | Yes |

Choose based on platform requirements and organizational standards.

---

# 119. Argo CD vs Spinnaker

Argo CD focuses strongly on:

```text
declarative Kubernetes GitOps
```

Spinnaker has historically focused on broader deployment orchestration and multi-cloud delivery patterns.

Use the platform that matches the delivery architecture.

---

# 120. Argo CD vs kubectl

`kubectl` is an imperative/admin interface.

Argo CD is a declarative GitOps reconciliation platform.

You may still use:

```bash
kubectl
```

for:

```text
troubleshooting
inspection
emergency operations
```

but normal application changes should preferably go through Git.

---

# 121. Why Not Edit Production With kubectl?

Because:

```text
Git no longer represents reality
```

This creates drift and audit problems.

Use:

```text
Git change
+
review
+
Argo CD
```

for normal changes.

---

# 122. Emergency kubectl Change

If an emergency requires direct intervention:

```text
incident
  ↓
temporary change
  ↓
stabilize service
  ↓
document
  ↓
update Git
  ↓
reconcile
```

The Git state must eventually reflect the intended final state.

---

# 123. How Would You Explain Your RoboShop GitOps Project?

Use this structure:

```text
1. Application architecture
2. CI pipeline
3. security scanning
4. ECR
5. GitOps repository
6. Argo CD
7. EKS
8. ALB
9. observability
10. rollback
```

---

# 124. RoboShop Interview Answer

> I worked with a Kubernetes-based RoboShop microservices platform on AWS EKS. CI handled source validation, testing, code-quality checks, security scanning and Docker image creation. Images were pushed to ECR. Deployment configuration was maintained separately in a GitOps repository using Helm. Argo CD continuously reconciled that configuration into EKS. For production-style environments I used environment-specific configuration, ApplicationSets for scalable application generation, AppProjects for boundaries, AWS Load Balancer Controller for ALB ingress, and Prometheus, Grafana and ELK for observability.

---

# 125. RoboShop CI Flow

```text
Developer
   |
   v
Git
   |
   v
Jenkins/GitHub Actions
   |
   +--> Maven/npm/Python tests
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
Docker
   |
   v
ECR
```

---

# 126. RoboShop CD Flow

```text
ECR
 |
 v
GitOps repository
 |
 v
Argo CD
 |
 v
Helm
 |
 v
Kubernetes API
 |
 v
EKS
 |
 v
ALB
```

---

# 127. Why ECR?

ECR provides a private AWS container registry.

Benefits:

```text
AWS integration
IAM
private images
lifecycle policies
regional repositories
```

---

# 128. Why Helm?

Helm helps package reusable Kubernetes application configuration.

For RoboShop:

```text
one chart pattern
+
environment values
```

can reduce duplication.

---

# 129. Why ApplicationSet for RoboShop?

If there are:

```text
many services
many environments
many clusters
```

manually creating every Application becomes difficult.

ApplicationSet can generate Applications from templates.

---

# 130. Why AppProjects?

They provide:

```text
source restrictions
destination restrictions
resource restrictions
```

and help implement least privilege.

---

# 131. How Would You Deploy 20 Services?

Do not manually maintain 20 independent pipelines that directly run `kubectl`.

Prefer:

```text
CI per service
      |
      v
ECR
      |
      v
GitOps
      |
      v
ApplicationSet/App of Apps
      |
      v
Argo CD
```

The exact Application structure depends on ownership and lifecycle.

---

# 132. How Would You Deploy to 10 Clusters?

Use:

```text
central Argo CD
+
registered clusters
+
cluster labels
+
ApplicationSet Cluster Generator
```

Then restrict access with:

```text
AppProjects
RBAC
```

---

# 133. How Would You Deploy Only to Production?

Use:

```text
cluster label:
environment=prod
```

and an ApplicationSet selector.

Also enforce production access through AppProjects and Git approval controls.

---

# 134. How Would You Prevent Wrong Cluster Deployment?

Multiple checks:

```text
Application destination
AppProject destinations
cluster labels
environment values
PR review
automated validation
```

---

# 135. How Would You Handle Production Approval?

Example:

```text
CI
 |
 v
ECR
 |
 v
GitOps PR
 |
 v
automated checks
 |
 v
CODEOWNERS approval
 |
 v
merge
 |
 v
Argo CD
```

Alternatively production sync itself can be controlled if required.

---

# 136. How Would You Handle Secrets?

Answer:

> I would avoid plaintext secrets in Git. I would use AWS Secrets Manager or another approved secret store and synchronize only the required secret references into Kubernetes through an operator such as External Secrets Operator.

---

# 137. How Would You Monitor Argo CD?

Monitor:

```text
Applications
sync failures
health failures
reconciliation
controller resources
repo server
API server
ApplicationSet controller
```

Use Prometheus/Grafana according to the deployment's supported metrics.

---

# 138. How Would You Alert on Deployment Failure?

Examples:

```text
sync failure
Application Degraded
Application OutOfSync beyond threshold
rollout failure
Pod CrashLoopBackOff
ALB unhealthy
```

---

# 139. How Would You Design Argo CD HA?

Use the supported HA architecture for the chosen Argo CD version.

Consider:

```text
multiple server replicas
controller scaling
repo-server scaling
Redis availability
cluster resources
network
monitoring
recovery
```

Do not invent a custom HA topology without checking the deployed version's documentation.

---

# 140. What Is the Difference Between Availability and Reconciliation?

Application availability:

```text
users can access application
```

Argo CD availability:

```text
desired state can be reconciled
```

They are related but not identical.

---

# 141. If Argo CD Is Down, Is the Application Down?

Usually no.

Argo CD is not normally in:

```text
ALB → Service → Pod
```

traffic path.

Existing Kubernetes workloads can continue serving traffic.

---

# 142. If EKS API Is Down, Is the Application Down?

Not necessarily immediately.

Existing workloads may continue running, but:

```text
deployments
scheduling
reconciliation
administrative operations
```

can be affected.

---

# 143. If One Worker Node Dies?

With multiple replicas and sufficient capacity:

```text
Pods can be rescheduled
```

depending on scheduling and storage constraints.

---

# 144. If One AZ Dies?

A multi-AZ architecture can continue if:

```text
replicas
nodes
capacity
load balancing
```

are appropriately distributed.

---

# 145. If Git Is Down?

Existing workloads generally continue.

New desired-state changes cannot be retrieved until Git is available.

---

# 146. If ECR Is Down?

Existing Pods may continue if images are already present.

New Pods requiring unavailable images can fail to start.

---

# 147. What Is GitOps Disaster Recovery?

GitOps DR means the environment can be rebuilt from declarative configuration plus required infrastructure, artifact and data recovery mechanisms.

It is not simply:

```text
Git repository backup
```

---

# 148. What Must Be Recovered?

```text
AWS infrastructure
EKS
Argo CD
Git repositories
secrets
ECR artifacts
databases
DNS
observability
```

---

# 149. How Would You Bootstrap Argo CD?

Typical:

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
root Application
   |
   v
platform
   |
   v
applications
```

---

# 150. Why Can Terraform Install Argo CD?

Terraform can manage:

```text
Helm release
Kubernetes resources
cluster infrastructure
```

But after bootstrap, Argo CD can manage many Kubernetes resources.

Avoid ownership conflicts.

---

# 151. Terraform and Argo CD Ownership Interview Answer

> I define a clear ownership boundary. Terraform owns AWS infrastructure such as VPC, EKS, IAM, ECR and RDS. Argo CD owns application Kubernetes resources. If Terraform bootstraps Argo CD itself, I avoid having both Terraform and Argo CD continuously manage the same application resources.

---

# 152. Production Architecture Question

### Design GitOps for 3 environments and 5 clusters.

Answer:

```text
Central Argo CD
       |
       +--> DEV clusters
       +--> QA clusters
       +--> PROD clusters
```

Use:

```text
ApplicationSet
Cluster Generator
environment labels
AppProjects
RBAC
```

---

# 153. Architecture Question

### How would you isolate production?

Use:

```text
separate AWS account
separate EKS cluster
AppProject
RBAC
protected Git
production CODEOWNERS
separate credentials
```

---

# 154. Architecture Question

### How would you support regional production?

```text
Central/Regional Argo CD
        |
        +--> EKS AP
        +--> EKS US
```

Use:

```text
cluster labels
ApplicationSet
regional values
artifact replication/availability
data strategy
```

---

# 155. Architecture Question

### How would you design ALB?

```text
Route 53
   |
   v
ALB
   |
   v
Ingress
   |
   v
Service
   |
   v
Pod
```

AWS Load Balancer Controller reconciles Kubernetes Ingress into AWS ALB resources.

---

# 156. Architecture Question

### Why not expose every Service using LoadBalancer?

Because:

```text
cost
attack surface
unnecessary public endpoints
management overhead
```

Prefer:

```text
ClusterIP
+
Ingress/ALB
```

for externally exposed HTTP applications where appropriate.

---

# 157. Architecture Question

### How would you secure ALB?

Consider:

```text
HTTPS
ACM certificate
security groups
WAF if required
private/public scheme
network design
health checks
least privilege
```

---

# 158. Architecture Question

### How would you secure Pods?

Use:

```text
non-root
allowPrivilegeEscalation=false
seccomp
read-only filesystem where possible
minimal capabilities
resource limits
NetworkPolicy
workload identity
```

---

# 159. Architecture Question

### How would you prevent noisy neighbors?

Use:

```text
requests
limits
ResourceQuota
LimitRange
HPA
namespace boundaries
cluster capacity
```

---

# 160. Architecture Question

### How would you make a service highly available?

Use:

```text
multiple replicas
multi-AZ scheduling
readiness probes
PDB
HPA where appropriate
resource requests
rolling updates
observability
```

---

# 161. Architecture Question

### How would you handle a breaking deployment?

```text
detect
 |
 v
stop rollout
 |
 v
rollback
 |
 v
validate
 |
 v
root cause
 |
 v
fix Git
```

---

# 162. Architecture Question

### How would you handle a database migration?

Use backward-compatible migration patterns where possible.

Avoid assuming:

```text
application rollback
=
database rollback
```

Use:

```text
expand
migrate
contract
```

patterns where appropriate.

---

# 163. Senior Scenario: OutOfSync

Question:

> Production is Healthy but OutOfSync. What do you do?

Answer:

```text
1. argocd app get
2. argocd app diff
3. identify resource/field
4. determine whether drift is intentional
5. identify controller ownership
6. fix Git or ignore only legitimate controller-managed differences
7. sync if required
```

---

# 164. Senior Scenario: Self-Heal Causes Loop

Possible cause:

```text
Argo CD changes field
controller changes field
Argo CD changes field again
```

This is an ownership conflict.

Fix:

```text
define ownership
configure ignore differences when appropriate
change chart/controller behavior
```

Do not blindly disable self-heal globally.

---

# 165. Senior Scenario: ApplicationSet Explosion

Suppose a selector suddenly matches 100 clusters.

Result:

```text
100 Applications
```

Possible operational impact.

Prevent with:

```text
careful labels
selectors
AppProjects
review
ApplicationSet testing
```

---

# 166. Senior Scenario: Production Prune

A bad commit removes an Application resource.

With prune:

```text
resource may be deleted
```

Controls:

```text
PR review
protected branch
approval
sync policies
resource exclusions where appropriate
```

---

# 167. Senior Scenario: GitOps Repo Compromised

Response:

```text
freeze deployments
revoke/rotate Git credentials
review commits
review Argo audit
identify malicious resources
restore trusted revision
validate clusters
rotate affected credentials
```

Then:

```text
root cause
security hardening
```

---

# 168. Senior Scenario: Argo CD Management Cluster Lost

Recovery:

```text
rebuild management infrastructure
install Argo CD
restore/bootstrap configuration
register target clusters
restore repository access
restore Projects
restore Applications/ApplicationSets
validate reconciliation
```

---

# 169. Senior Scenario: Target Cluster Lost

Recovery:

```text
rebuild EKS
restore infrastructure
restore IAM
register cluster
Argo CD syncs applications
restore external dependencies
validate traffic
```

---

# 170. Senior Scenario: Region Lost

Requires:

```text
DR cluster
artifact availability
data replication
secret recovery
DNS/failover
GitOps
observability
capacity
```

GitOps handles desired application state, not the complete regional DR problem.

---

# 171. Production Interview Question

### What is the difference between Argo CD and Kubernetes controllers?

Kubernetes controllers reconcile Kubernetes resources.

Argo CD reconciles Git desired state against Kubernetes state.

They form nested control loops:

```text
Git
 |
 v
Argo CD
 |
 v
Kubernetes desired objects
 |
 v
Kubernetes controllers
 |
 v
runtime
```

---

# 172. Production Interview Question

### Can Argo CD manage CRDs?

Yes, subject to permissions and resource handling.

However, CRD lifecycle must be carefully planned because:

```text
CRD
+
CR
```

may have ordering/dependency requirements.

---

# 173. Production Interview Question

### Why use sync waves for CRDs?

Because custom resources may depend on their CRDs existing first.

Concept:

```text
CRD
 ↓
Controller
 ↓
Custom Resource
```

Use supported ordering mechanisms deliberately.

---

# 174. Production Interview Question

### How would you deploy platform components before applications?

Use:

```text
App of Apps
sync waves
dependencies
ApplicationSets
```

Example:

```text
Ingress controller
 ↓
External Secrets
 ↓
monitoring
 ↓
applications
```

The exact order depends on dependencies.

---

# 175. Production Interview Question

### How do you manage namespaces?

Possible options:

```text
Application creates namespace
Terraform creates namespace
platform GitOps creates namespace
```

Choose one owner.

Do not have multiple systems fight over namespace configuration.

---

# 176. Production Interview Question

### How do you prevent namespace deletion?

Use:

```text
AppProject restrictions
RBAC
Git review
careful prune behavior
```

Critical namespaces should have stronger operational controls.

---

# 177. Production Interview Question

### What is a GitOps anti-pattern?

Examples:

```text
kubectl edit production
latest image
raw secrets in Git
cluster-admin everywhere
CI direct production deployment
two tools owning one resource
unreviewed production sync
no observability
no DR
```

---

# 178. Production Interview Question

### Is GitOps fully immutable?

Git commits are immutable historical records in the sense that history provides traceability, but branches and repositories can be changed through authorized Git operations.

Therefore:

```text
GitOps security
=
Git integrity
+
access control
+
review
+
audit
```

---

# 179. Production Interview Question

### Does GitOps eliminate configuration drift?

It can continuously detect and correct drift, but it does not prevent every possible external change.

Drift management requires:

```text
permissions
self-heal
ownership
policy
monitoring
```

---

# 180. Production Interview Question

### What is declarative deployment?

You describe:

```text
what should exist
```

rather than:

```text
step-by-step commands to create it
```

Example:

```yaml
replicas: 3
```

The controllers determine how to reach that state.

---

# 181. Production Interview Question

### Why is declarative state useful?

Because it enables:

```text
repeatability
review
automation
recovery
audit
reconciliation
```

---

# 182. Production Interview Question

### What is idempotency?

An operation is idempotent when applying the desired state repeatedly produces the same intended result.

GitOps reconciliation depends heavily on this concept.

---

# 183. Production Interview Question

### Why is Kubernetes naturally suited to GitOps?

Kubernetes is declarative and controller-driven.

You specify:

```text
desired resources
```

and Kubernetes controllers continuously work toward that state.

Argo CD extends this model by making Git the external desired-state source.

---

# 184. Production Interview Question

### What does Argo CD actually send to Kubernetes?

It sends the rendered desired Kubernetes resources through the Kubernetes API according to the sync process and configured options.

For Helm:

```text
Helm chart
 ↓
rendered manifests
 ↓
Argo CD
 ↓
Kubernetes API
```

---

# 185. Production Interview Question

### Does Argo CD run Helm install?

Conceptually, Argo CD uses Helm to render manifests rather than relying on Helm's normal release lifecycle as the primary deployment authority.

Argo CD owns synchronization.

---

# 186. Production Interview Question

### What happens if Helm release metadata is missing?

Argo CD can still manage the rendered Kubernetes resources because its model is based on desired manifests and resource tracking rather than treating Helm as the sole deployment controller.

---

# 187. Production Interview Question

### Why separate GitOps repository?

Benefits:

```text
deployment ownership
audit
security
environment control
separation of duties
```

---

# 188. Production Interview Question

### One repository or multiple?

One:

```text
centralized governance
```

Multiple:

```text
team autonomy
access boundaries
```

Neither is universally correct.

---

# 189. Production Interview Question

### How do you avoid massive GitOps repositories?

Use:

```text
clear ownership
directory conventions
ApplicationSets
multiple repositories when justified
platform/application separation
```

---

# 190. Production Interview Question

### How do you prevent configuration duplication?

Use:

```text
Helm base values
environment overrides
Kustomize bases/overlays
ApplicationSet templates
```

Avoid copying entire manifests unnecessarily.

---

# 191. Production Interview Question

### How do you handle environment-specific image tags?

Examples:

```text
dev → 1.5.0
qa → 1.5.0
prod → 1.5.0
```

Prefer promoting the same immutable artifact rather than rebuilding.

---

# 192. Production Interview Question

### How do you handle hotfixes?

Possible flow:

```text
hotfix code
 |
 v
CI
 |
 v
security checks
 |
 v
ECR
 |
 v
GitOps hotfix PR
 |
 v
approval
 |
 v
Argo CD
```

Use an emergency process only when necessary.

---

# 193. Production Interview Question

### How do you audit who deployed production?

Combine:

```text
Git history
PR approvals
Argo CD history
identity provider logs
Kubernetes audit logs where available
CI records
```

---

# 194. Production Interview Question

### How do you prove what version is running?

Check:

```text
Git commit
Application desired state
Deployment image
Pod imageID/digest
ECR artifact
```

A strong pipeline links these identifiers.

---

# 195. Production Interview Question

### How do you detect unauthorized production changes?

Use:

```text
Argo CD drift detection
self-heal
Kubernetes audit
RBAC
alerts
```

---

# 196. Production Interview Question

### What if a controller intentionally modifies a resource?

Identify:

```text
controller ownership
```

Then avoid false-positive drift by using appropriate configuration.

Never hide broad differences without understanding them.

---

# 197. Production Interview Question

### How do you monitor deployment frequency?

Track:

```text
deployment count
deployment success rate
deployment lead time
rollback rate
change failure rate
```

These align with DevOps delivery metrics.

---

# 198. Production Interview Question

### How do you measure GitOps health?

Track:

```text
sync success
sync duration
OutOfSync duration
Degraded applications
controller errors
repo errors
cluster connectivity
```

---

# 199. Production Interview Question

### What are good GitOps SLOs?

Examples:

```text
Argo CD availability
sync latency
deployment success rate
drift detection latency
```

Actual targets must be defined by business/platform requirements.

---

# 200. Production Interview Question

### How do you scale Argo CD?

Consider:

```text
number of Applications
number of resources
number of clusters
reconciliation frequency
repo size
manifest complexity
controller replicas
repo-server capacity
API-server load
```

Do not scale based only on Pod count.

---

# 201. Production Interview Question

### What causes Argo CD performance problems?

Possible:

```text
large repositories
many Applications
many clusters
large manifests
frequent changes
API server throttling
resource limits
repo-server bottlenecks
controller workload
```

---

# 202. Production Interview Question

### How do you troubleshoot Argo CD performance?

Check:

```text
Argo CD component CPU/memory
controller logs
repo-server logs
API errors
reconciliation latency
Application count
cluster count
Git repository size
```

---

# 203. Production Interview Question

### How do you design a large-scale GitOps platform?

Use:

```text
clear repository structure
AppProjects
ApplicationSets
cluster labels
multiple Argo CD instances when justified
HA
observability
RBAC
policy
DR
```

---

# 204. Production Interview Question

### How do you onboard a new cluster?

Typical:

```text
1. provision EKS
2. configure IAM
3. install required platform controllers
4. register cluster with Argo CD
5. apply labels
6. validate Project permissions
7. ApplicationSet detects cluster
8. platform Applications deploy
9. business applications deploy
10. validate health
```

---

# 205. Production Interview Question

### How do you remove a cluster safely?

```text
stop new deployments
drain/migrate workloads
remove ApplicationSet targeting
validate Applications
remove cluster registration
revoke credentials
decommission infrastructure
```

Be careful with pruning/deletion semantics.

---

# 206. Production Interview Question

### How do you onboard a new service?

```text
application repo
 ↓
CI
 ↓
ECR
 ↓
Helm chart/config
 ↓
GitOps Application
 ↓
Argo CD
 ↓
namespace
 ↓
Deployment
 ↓
Service
 ↓
Ingress if required
```

---

# 207. Production Interview Question

### How do you onboard a new environment?

Define:

```text
cluster
namespace
values
AppProject
Application/ApplicationSet
secrets
observability
networking
```

Then validate the environment-specific policy.

---

# 208. Production Interview Question

### How do you onboard a new region?

Consider:

```text
EKS
VPC
IAM
ECR/artifact availability
secrets
data
DNS
ALB
observability
ApplicationSet targeting
DR/failover
```

---

# 209. Production Interview Question

### What is the biggest advantage of GitOps?

A strong answer:

> The biggest advantage is that deployment state becomes declarative, version-controlled and continuously reconciled. This improves auditability, repeatability and drift management while allowing CI and production cluster access to be separated.

---

# 210. Production Interview Question

### What is the biggest disadvantage?

Possible:

```text
Git becomes critical governance infrastructure
initial complexity
large-scale Application management
controller troubleshooting
learning curve
```

GitOps improves control but adds operational architecture.

---

# 211. Production Interview Question

### When would you not use GitOps?

Possible cases:

```text
legacy systems without declarative interfaces
extremely dynamic runtime changes
systems where Git cannot safely represent desired state
```

Even then, parts of the system may still benefit from GitOps.

---

# 212. Production Interview Question

### Is GitOps a tool?

No.

GitOps is an operating model.

Tools include:

```text
Argo CD
Flux
Git
Helm
Kustomize
policy engines
secret operators
```

---

# 213. Production Interview Question

### What is the role of Git in GitOps?

Git provides:

```text
desired state
version history
review
approval
audit
rollback
```

---

# 214. Production Interview Question

### What is the role of Argo CD?

Argo CD provides:

```text
reconciliation
sync
drift detection
health status
deployment history
multi-cluster management
```

---

# 215. Production Interview Question

### What is the role of Kubernetes?

Kubernetes provides:

```text
runtime orchestration
scheduling
service discovery
resource management
controller loops
```

---

# 216. Production Interview Question

### What is the role of CI?

CI provides:

```text
build
test
security validation
artifact creation
artifact publishing
```

---

# 217. Production Interview Question

### What is the role of ECR?

ECR stores:

```text
container artifacts
```

that EKS Pods consume.

---

# 218. Production Interview Question

### What is the role of AWS Load Balancer Controller?

It observes Kubernetes resources such as Ingress and reconciles them into AWS load-balancing resources such as ALB.

---

# 219. Production Interview Question

### What is the role of Terraform?

In this architecture Terraform provisions:

```text
VPC
EKS
IAM
ECR
RDS
security/network prerequisites
```

while Argo CD handles application deployment.

---

# 220. Production Interview Question

### What is the role of Prometheus/Grafana/ELK?

```text
Prometheus → metrics
Grafana → visualization
ELK → logs
```

This provides operational visibility into the GitOps-managed platform.

---

# 221. Rapid-Fire Questions

## Q: GitOps source of truth?

Git.

## Q: Pull or push?

Argo CD uses a pull/reconciliation model.

## Q: Argo CD controller?

Application Controller.

## Q: Manifest generation?

Repo Server.

## Q: UI/API?

API Server.

## Q: Fleet generation?

ApplicationSet Controller.

## Q: Multiple clusters?

Yes.

## Q: EKS?

Yes.

## Q: Helm?

Yes.

## Q: Kustomize?

Yes.

## Q: Drift?

Detected and optionally corrected.

## Q: AppProject?

Security/configuration boundary.

## Q: ApplicationSet?

Generates Applications.

## Q: App of Apps?

Parent Application manages child Applications.

## Q: Production secrets in Git?

Avoid.

## Q: Latest tag?

Avoid.

## Q: CI direct kubectl?

Prefer not in a mature GitOps architecture.

---

# 222. Rapid-Fire: Sync

### What is Sync?

Apply desired Git state to the cluster.

### What is OutOfSync?

Desired and live state differ.

### What is Synced?

Desired/live state align.

### What is prune?

Delete resources removed from desired state.

### What is selfHeal?

Correct eligible live-state drift automatically.

### What is refresh?

Re-evaluate source/live information.

### What is reconciliation?

Continuous convergence toward desired state.

---

# 223. Rapid-Fire: Health

### Healthy?

Resource is operating as expected.

### Progressing?

Resource is changing toward desired healthy state.

### Degraded?

Resource is unhealthy.

### Missing?

Expected resource is absent.

### Unknown?

Health cannot be determined.

---

# 224. Rapid-Fire: Kubernetes

### ClusterIP?

Internal service.

### ALB?

External HTTP/HTTPS load balancing.

### Deployment?

Manages application Pods/ReplicaSets.

### HPA?

Scales replicas based on metrics.

### PDB?

Controls voluntary disruption availability.

### Readiness?

Determines whether Pod should receive traffic.

### Liveness?

Determines whether container needs restarting.

---

# 225. Rapid-Fire: AWS

### ECR?

Container registry.

### EKS?

Managed Kubernetes.

### ALB?

Layer 7 load balancer.

### Route 53?

DNS.

### ACM?

TLS certificates.

### Secrets Manager?

Managed secret storage.

### IAM?

AWS authorization/identity.

---

# 226. Scenario: Pod OOMKilled

Answer:

```text
kubectl describe pod
kubectl logs --previous
kubectl get pod
```

Check:

```text
memory requests
memory limits
application behavior
traffic
leak
HPA
```

Then fix:

```text
application
resources
scaling
```

as appropriate.

---

# 227. Scenario: CrashLoopBackOff

Answer:

```text
kubectl describe pod
kubectl logs
kubectl logs --previous
```

Check:

```text
startup error
configuration
secret
dependency
probe
permissions
```

---

# 228. Scenario: ImagePullBackOff

Check:

```text
repository
tag
ECR permissions
network
registry
image architecture
```

---

# 229. Scenario: Readiness Probe Failure

Check:

```text
endpoint
port
startup time
dependency
application logs
```

Do not simply remove the readiness probe.

---

# 230. Scenario: Liveness Probe Failure

Check:

```text
application health
probe timeout
initial delay
resource starvation
deadlock
```

A badly designed liveness probe can create restart loops.

---

# 231. Scenario: ALB Returns 503

Check:

```text
ALB target health
Service endpoints
Pod readiness
Ingress
target type
security groups
```

---

# 232. Scenario: Service Has No Endpoints

Check:

```bash
kubectl get endpointslice -n <namespace>
kubectl get pods --show-labels -n <namespace>
```

Likely issue:

```text
Service selector != Pod labels
```

---

# 233. Scenario: HPA Does Not Scale

Check:

```bash
kubectl describe hpa <name>
```

Possible:

```text
metrics unavailable
resource requests missing
wrong target
metrics-server issue
```

---

# 234. Scenario: Application Is Healthy but Traffic Fails

Check:

```text
ALB
Ingress
Service
EndpointSlice
Pod readiness
security groups
DNS
TLS
```

Argo CD can be completely healthy while network traffic is broken outside the Application resource health.

---

# 235. Scenario: Git Commit Is Correct but Argo Shows Old State

Investigate:

```text
targetRevision
repository
path
refresh/cache
commit
credentials
webhook/refresh mechanism
```

Do not assume Git is the problem.

---

# 236. Scenario: Webhook Not Triggering

Argo CD can also discover changes through its normal reconciliation/refresh mechanisms.

For webhook problems check:

```text
Git provider configuration
webhook endpoint
network
authentication
Argo CD server
repository
```

---

# 237. Scenario: Production Sync Is Blocked

Check:

```text
AppProject
RBAC
sync window
resource restrictions
destination
repository
approval process
```

---

# 238. Scenario: Namespace Does Not Exist

Possible solution:

```yaml
syncOptions:
  - CreateNamespace=true
```

or have a dedicated platform process own namespace creation.

Do not create two competing owners.

---

# 239. Scenario: Resource Is Forbidden

Check:

```text
AppProject
Kubernetes RBAC
resource whitelist
cluster whitelist
namespace
```

---

# 240. Scenario: CRD Not Found

Likely:

```text
CRD not installed
wrong version
sync ordering
controller missing
```

Deploy:

```text
CRD
controller
custom resource
```

in the correct dependency order.

---

# 241. Scenario: Controller Is Fighting Argo CD

Identify:

```text
which controller changes the field?
```

Then choose:

```text
single owner
ignore legitimate difference
change configuration
```

---

# 242. Production GitOps Design Exercise

### Requirement

A company has:

```text
3 environments
5 EKS clusters
20 services
2 AWS accounts
```

Design:

```text
CI
GitOps
Argo CD
ApplicationSets
security
observability
```

---

# 243. Design Answer

Use:

```text
Application repositories
+
GitOps repository
+
CI
+
ECR
+
central Argo CD where appropriate
+
ApplicationSets
+
AppProjects
+
environment isolation
+
separate production account
```

---

# 244. Design Diagram

```text
                 Application Git
                        |
                        v
                       CI
                        |
                        v
                       ECR
                        |
                        v
                  GitOps Repository
                        |
                        v
                    Argo CD
             /           |           \
            v            v            v
         DEV EKS       QA EKS       PROD EKS
          / \            |            / \
        C1  C2          C3          C4  C5
```

---

# 245. Design Exercise: Production Security

Requirements:

```text
developers cannot deploy directly
production requires approval
secrets cannot be in Git
multiple teams share platform
```

Answer:

```text
SSO
Argo RBAC
AppProjects
CODEOWNERS
protected Git
secret manager
least privilege
```

---

# 246. Design Exercise: DR

Requirement:

```text
management cluster can be lost
```

Answer:

```text
Terraform rebuild
Argo CD bootstrap
Git source
secret recovery
target cluster registration
ApplicationSet
validation
```

---

# 247. Design Exercise: Regional DR

Requirement:

```text
AP-South-1 primary
US-East-1 DR
```

Need:

```text
EKS
artifact availability
GitOps
secrets
database replication
DNS
observability
failover
```

---

# 248. Design Exercise: Zero-Downtime Deployment

Use:

```text
multiple replicas
readiness
rolling strategy
PDB
HPA
multi-AZ
ALB
```

For high-risk releases:

```text
canary/blue-green
```

---

# 249. Design Exercise: Secure Supply Chain

Use:

```text
Git
 |
 v
CI
 |
 +--> SonarQube
 +--> Trivy
 +--> Veracode
 |
 v
ECR
 |
 v
GitOps
 |
 v
Argo CD
 |
 v
EKS
```

Add:

```text
immutable artifacts
protected branches
policy
least privilege
```

---

# 250. Senior-Level Interview Questions

## 1. How would you design Argo CD for 100+ clusters?

Discuss:

```text
control-plane topology
ApplicationSet
cluster labels
AppProjects
RBAC
HA
scaling
repo organization
failure domains
multiple instances
regional distribution
```

---

## 2. How would you avoid a central Argo CD becoming a single point of failure?

Answer:

> Argo CD should be deployed using an appropriate HA architecture, monitored and recoverable. Existing applications are not normally dependent on Argo CD for request serving, but reconciliation and deployment control are affected if it is unavailable. For very large or highly isolated environments, multiple Argo CD control planes can reduce blast radius.

---

## 3. How would you manage 1,000 Applications?

Discuss:

```text
ApplicationSet
repository organization
controller capacity
repo-server scaling
Application grouping
team ownership
AppProjects
observability
```

---

## 4. How do you prevent ApplicationSet from creating unauthorized applications?

Use:

```text
AppProject
RBAC
repository restrictions
cluster restrictions
generator selectors
Git review
```

---

## 5. How do you manage platform dependencies?

Use:

```text
App of Apps
sync waves
hooks
ApplicationSets
```

with explicit ownership.

---

# 251. Staff-Level Architecture Question

### Design GitOps for a global organization.

Consider:

```text
global Git governance
regional Argo CD
central standards
regional clusters
separate AWS accounts
environment isolation
identity
secrets
DR
observability
policy
```

There is no single correct architecture.

The answer should explain trade-offs.

---

# 252. Staff-Level Question

### Should every team share one Argo CD?

Not necessarily.

Use one when:

```text
central governance
manageable scale
shared trust
```

Use multiple when:

```text
strong isolation
regional autonomy
business-unit separation
compliance
blast-radius requirements
```

---

# 253. Staff-Level Question

### What should be centralized?

Potentially:

```text
GitOps standards
templates
security policy
platform conventions
observability
```

What should be decentralized:

```text
application ownership
service configuration
application lifecycle
```

The exact model depends on the organization.

---

# 254. Staff-Level Question

### What is the biggest GitOps scaling challenge?

Not necessarily the number of YAML files.

The harder challenges are:

```text
ownership
repository design
controller load
cluster count
permissions
secrets
promotion
observability
failure domains
```

---

# 255. Staff-Level Question

### How do you prevent GitOps becoming "Git as a dumping ground"?

Use:

```text
repository conventions
schemas
templates
validation
ownership
CODEOWNERS
documentation
policy
```

---

# 256. Staff-Level Question

### How do you balance developer autonomy and production security?

Provide:

```text
golden paths
self-service templates
ApplicationSets
standard Helm charts
```

while enforcing:

```text
AppProjects
RBAC
policy
production approvals
```

---

# 257. Staff-Level Question

### What would you automate first?

Prioritize:

```text
application templates
CI
security checks
image promotion
GitOps PR
ApplicationSet
observability
DR bootstrap
```

---

# 258. Staff-Level Question

### How do you measure whether GitOps improved the organization?

Use:

```text
deployment frequency
lead time
change failure rate
rollback time
drift incidents
manual production changes
audit effort
recovery time
```

---

# 259. Interview Mistakes to Avoid

Do not say:

```text
"Argo CD replaces Jenkins."
```

Better:

```text
"Argo CD complements CI by providing GitOps CD and reconciliation."
```

---

# 260. Interview Mistake

Do not say:

```text
"Argo CD is a Kubernetes scheduler."
```

It is not.

Kubernetes schedules workloads.

Argo CD manages desired state synchronization.

---

# 261. Interview Mistake

Do not say:

```text
"GitOps means everything must be in one Git repository."
```

GitOps does not require one repository.

---

# 262. Interview Mistake

Do not say:

```text
"Argo CD guarantees zero downtime."
```

It does not.

Zero downtime depends on:

```text
application design
replicas
probes
rollout strategy
capacity
load balancing
dependencies
```

---

# 263. Interview Mistake

Do not say:

```text
"Self-heal fixes every Kubernetes issue."
```

Self-heal corrects desired-state drift.

It does not fix:

```text
application bugs
bad images
database failures
capacity issues
external dependency failures
```

---

# 264. Interview Mistake

Do not say:

```text
"Rollback always fixes the incident."
```

Rollback can fail because of:

```text
database changes
missing images
dependency incompatibility
configuration
```

---

# 265. Interview Mistake

Do not say:

```text
"Git is the backup for everything."
```

Git does not back up:

```text
database data
runtime state
external AWS data
```

---

# 266. Interview Mistake

Do not say:

```text
"Terraform and Argo CD can both manage the same Kubernetes resources."
```

They can technically interact, but shared ownership often creates conflicts and should be avoided.

---

# 267. Interview Mistake

Do not claim tools you did not use.

For your practical stack, keep the interview story aligned with:

```text
AWS
EKS
ECR
Kubernetes
Helm
ALB
Jenkins/GitHub Actions
Argo CD
Terraform
SonarQube
Trivy
Veracode
Prometheus
Grafana
ELK
```

---

# 268. Practical Interview Story

A concise answer:

> My GitOps architecture separates CI from CD. Jenkins or GitHub Actions builds and tests the application, runs SonarQube, Trivy and Veracode checks, builds the Docker image and publishes it to ECR. The deployment configuration is maintained in a separate GitOps repository using Helm. Argo CD watches that repository and reconciles the desired state into AWS EKS. For multiple environments or clusters, I use ApplicationSets and AppProjects. External traffic is handled through Kubernetes Ingress and AWS ALB, while Prometheus, Grafana and ELK provide observability.

---

# 269. Practical Interview Story — Drift

> If someone manually changes a Git-managed Kubernetes resource, Argo CD detects the difference between the live state and Git. With self-healing enabled, Argo CD can restore the desired state. If the field is intentionally controlled by another controller, I first determine ownership and configure the GitOps model accordingly rather than creating a reconciliation loop.

---

# 270. Practical Interview Story — Multi-Cluster

> I can run Argo CD as a centralized GitOps control plane and register multiple EKS clusters. I label clusters with metadata such as environment, region and account. ApplicationSet Cluster Generators select the appropriate clusters and generate Applications dynamically. AppProjects and RBAC prevent teams from deploying to unauthorized clusters or namespaces.

---

# 271. Practical Interview Story — Security

> I keep production cluster credentials out of CI where possible. CI publishes immutable artifacts and changes the GitOps desired state. Production changes go through protected Git workflows and approvals. Argo CD has restricted RBAC and AppProjects. Secrets are retrieved from a dedicated secret store instead of storing plaintext values in Git.

---

# 272. Practical Interview Story — Failure

> If Argo CD fails, existing workloads normally continue serving because Argo CD is not in the application traffic path. New changes and drift reconciliation are affected. I would restore the Argo CD control plane and verify Applications, repositories, cluster connections and ApplicationSets. This is why I design Argo CD for appropriate HA and have a tested bootstrap/DR process.

---

# 273. Practical Interview Story — Troubleshooting

> I start with the Argo CD Application to determine sync and health state, then use `argocd app diff` to identify configuration differences. If the problem is runtime-related, I move into Kubernetes with `kubectl describe`, events and Pod logs. For deployment-specific failures I check Helm rendering, repository access, cluster permissions and controller logs. For external traffic I continue through Service, EndpointSlice, Ingress and ALB health.

---

# 274. Production Troubleshooting Command Sheet

```bash
# Applications
argocd app list
argocd app get <app>
argocd app diff <app>
argocd app sync <app>
argocd app history <app>
argocd app rollback <app> <id>

# Clusters
argocd cluster list

# Repositories
argocd repo list

# Kubernetes Applications
kubectl get applications -n argocd
kubectl describe application <app> -n argocd

# ApplicationSets
kubectl get applicationsets -n argocd
kubectl describe applicationset <name> -n argocd

# Workloads
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> --previous -n <namespace>

# Deployment
kubectl rollout status deployment/<name> -n <namespace>

# Events
kubectl get events -n <namespace> --sort-by=.lastTimestamp

# Services
kubectl get svc -n <namespace>
kubectl get endpointslice -n <namespace>

# Ingress
kubectl get ingress -n <namespace>
kubectl describe ingress <name> -n <namespace>

# HPA
kubectl get hpa -n <namespace>
kubectl describe hpa <name> -n <namespace>

# Argo CD components
kubectl get pods -n argocd
kubectl logs -n argocd deployment/argocd-server
kubectl logs -n argocd deployment/argocd-repo-server
kubectl logs -n argocd deployment/argocd-application-controller
kubectl logs -n argocd deployment/argocd-applicationset-controller
```

---

# 275. Troubleshooting Decision Tree

```text
Application failed
      |
      v
Argo CD status?
      |
 +----+----+
 |         |
 v         v
OutOfSync  Synced
 |         |
 v         v
Diff?    Health?
           |
       +---+---+
       |       |
       v       v
    Healthy  Degraded
               |
               v
        Kubernetes runtime
               |
        +------+------+
        |             |
        v             v
      Pods          Network
        |             |
        v             v
      Logs          Service
                      |
                      v
                    ALB
```

---

# 276. Interview Whiteboard Architecture

Draw:

```text
             Developer
                 |
                 v
           Application Git
                 |
                 v
                CI
                 |
       +---------+---------+
       |         |         |
    SonarQube  Trivy   Veracode
       |         |         |
       +---------+---------+
                 |
                 v
                ECR
                 |
                 v
            GitOps Git
                 |
                 v
              Argo CD
            /    |    \
           v     v     v
        DEV EKS QA EKS PROD EKS
           |     |     |
          ALB   ALB   ALB
```

Then explain:

```text
CI = artifact
GitOps = desired state
Argo CD = reconciliation
EKS = runtime
ALB = traffic
observability = feedback
```

---

# 277. Interview Whiteboard — Multi-Cluster

Draw:

```text
                    Git
                     |
                     v
                 Argo CD
             /      |      \
            /       |       \
           v        v        v
        EKS-DEV  EKS-QA   EKS-PROD
           |        |         |
        Namespace Namespace Namespace
```

Add:

```text
ApplicationSet
AppProject
RBAC
```

---

# 278. Interview Whiteboard — DR

Draw:

```text
Terraform
   |
   v
EKS
   |
   v
Argo CD
   |
   v
Root Application
   |
   v
ApplicationSets
   |
   v
Applications
```

Explain:

```text
Git = desired state
Terraform = infrastructure
Secrets = external
Data = independent backup/replication
```

---

# 279. Interview Whiteboard — Drift

Draw:

```text
          Git
           |
      desired = 3
           |
           v
        Argo CD
           |
           v
        EKS
     actual = 5
           |
           v
        diff
           |
           v
      self-heal
           |
           v
        actual = 3
```

Then mention:

```text
controller-owned fields require special handling
```

---

# 280. Interview Whiteboard — CI vs CD

Draw two planes:

```text
CI PLANE
Git → CI → Scan → ECR

CD PLANE
GitOps → Argo CD → EKS
```

This is one of the strongest diagrams to use in interviews.

---

# 281. Interview Whiteboard — Terraform Boundary

```text
Terraform
   |
   +--> VPC
   +--> EKS
   +--> IAM
   +--> ECR
   +--> RDS
   |
   v
Infrastructure

Argo CD
   |
   +--> Deployment
   +--> Service
   +--> Ingress
   +--> HPA
   +--> PDB
   |
   v
Applications
```

---

# 282. Interview Question: Why not Terraform for application deployment?

Terraform can manage Kubernetes resources, but in this architecture Argo CD provides specialized GitOps reconciliation for application delivery.

Using both is possible, but ownership must be clearly separated.

---

# 283. Interview Question: Why not use only Helm?

Helm packages and renders Kubernetes configuration.

Argo CD provides:

```text
continuous reconciliation
GitOps
drift detection
Application health
multi-cluster management
```

They complement each other.

---

# 284. Interview Question: Why not use only Git?

Git stores desired state.

It does not itself continuously reconcile that state into Kubernetes.

Argo CD provides the control loop.

---

# 285. Interview Question: Why not use only Kubernetes?

Kubernetes reconciles runtime resources.

GitOps adds:

```text
version-controlled desired state
review
audit
promotion
```

outside the cluster.

---

# 286. Interview Question: What makes GitOps different from simply storing YAML in Git?

The reconciliation loop.

```text
YAML in Git
+
controller continuously applying/reconciling it
=
GitOps
```

---

# 287. Interview Question: What is continuous reconciliation?

It means the system repeatedly evaluates whether:

```text
live state == desired state
```

and acts when they differ.

---

# 288. Interview Question: What is declarative infrastructure?

You specify:

```text
desired result
```

instead of:

```text
manual sequence of operations
```

---

# 289. Interview Question: What is imperative deployment?

Example:

```bash
kubectl scale deployment cart --replicas=5
kubectl set image deployment/cart cart=cart:2.0
```

It tells the cluster how to change immediately.

GitOps generally prefers declarative state.

---

# 290. Interview Question: Can GitOps still use imperative commands?

Yes.

Operators may use:

```bash
kubectl
argocd
```

for troubleshooting and controlled emergency actions.

But normal desired-state changes should be represented in Git.

---

# 291. Interview Question: What is drift remediation?

Detect:

```text
difference
```

then restore:

```text
desired state
```

---

# 292. Interview Question: What is an immutable deployment?

A deployment where the artifact reference cannot silently change.

Example:

```text
image digest
```

is stronger than a mutable tag.

---

# 293. Interview Question: Why promote the same image?

It ensures:

```text
tested artifact
=
promoted artifact
```

rather than rebuilding potentially different binaries for each environment.

---

# 294. Interview Question: What is a golden path?

A standardized, supported way for developers to build and deploy services.

Example:

```text
service template
+
CI template
+
Helm
+
Argo CD
+
observability
```

---

# 295. Interview Question: What is platform engineering's relationship to GitOps?

Platform engineering can use GitOps as the delivery mechanism for standardized infrastructure and application capabilities.

---

# 296. Interview Question: How do you reduce developer complexity?

Provide:

```text
templates
automation
self-service
standard charts
ApplicationSets
prebuilt CI
```

while hiding unnecessary infrastructure complexity.

---

# 297. Interview Question: What should an SRE/DevOps engineer own?

Depending on the organization:

```text
platform
GitOps
cluster reliability
observability
security integration
deployment standards
incident response
DR
```

Application teams own application behavior.

---

# 298. Interview Question: How do you handle ownership conflicts?

Ask:

```text
Who owns the resource?
Who owns the field?
Who owns the lifecycle?
```

Then establish one authoritative owner.

---

# 299. Interview Question: What is the most important GitOps design principle?

A strong answer:

> Every important resource should have a clear source of truth and clear ownership. Git should represent the desired state, Argo CD should reconcile it, and other controllers should own only the fields/resources they are designed to manage.

---

# 300. Final Interview Preparation Checklist

## GitOps

```text
[ ] definition
[ ] principles
[ ] declarative model
[ ] desired vs actual
[ ] pull model
[ ] reconciliation
[ ] drift
[ ] audit
[ ] rollback
```

## Argo CD

```text
[ ] architecture
[ ] components
[ ] Application
[ ] AppProject
[ ] ApplicationSet
[ ] sync
[ ] refresh
[ ] health
[ ] drift
[ ] hooks
[ ] waves
[ ] rollback
[ ] RBAC
[ ] repositories
```

## Kubernetes

```text
[ ] Deployment
[ ] Service
[ ] Ingress
[ ] probes
[ ] resources
[ ] HPA
[ ] PDB
[ ] RBAC
[ ] NetworkPolicy
```

## AWS/EKS

```text
[ ] EKS
[ ] ECR
[ ] IAM
[ ] VPC
[ ] ALB
[ ] Route 53
[ ] ACM
[ ] Secrets Manager
```

## Production

```text
[ ] multi-environment
[ ] multi-cluster
[ ] multi-account
[ ] security
[ ] observability
[ ] HA
[ ] DR
[ ] rollback
[ ] troubleshooting
```

## RoboShop

```text
[ ] CI
[ ] SonarQube
[ ] Trivy
[ ] Veracode
[ ] ECR
[ ] Helm
[ ] Argo CD
[ ] EKS
[ ] ALB
[ ] Prometheus
[ ] Grafana
[ ] ELK
```

---

# 301. Final 30-Second Answer

If the interviewer says:

> Explain your GitOps implementation.

Answer:

> I use Git as the source of truth for Kubernetes desired state. Jenkins or GitHub Actions handles CI, including testing, SonarQube, Trivy, Veracode, Docker image creation and publishing to ECR. The deployment configuration is maintained separately in a GitOps repository using Helm. Argo CD continuously reconciles that desired state into AWS EKS and detects configuration drift. For multiple environments and clusters I use ApplicationSets, while AppProjects and RBAC provide deployment boundaries. External traffic is exposed through AWS ALB using the Load Balancer Controller, and Prometheus, Grafana and ELK provide observability. Production changes are protected through Git review, least privilege, immutable artifacts, controlled promotion and rollback procedures.

---

# 302. Final 2-Minute Architecture Answer

> The architecture has two major planes. The CI plane starts from application Git. Jenkins or GitHub Actions checks out the code, runs tests, performs SonarQube analysis and security checks such as Trivy and Veracode, builds the Docker image and publishes an immutable artifact to ECR.
>
> The CD plane is GitOps. CI updates the desired image reference in a separate GitOps repository through a controlled Git change. Argo CD watches the repository, renders Helm or Kubernetes configuration, compares the desired state with the live EKS state and continuously reconciles differences.
>
> For multiple environments and clusters, I use ApplicationSets. A list generator can manage explicit DEV, QA and PROD environments, while a cluster generator can select registered EKS clusters using labels such as environment, region and account. AppProjects restrict which repositories, clusters, namespaces and resource types an application can use.
>
> In AWS, the application runs on EKS and external HTTP/HTTPS traffic enters through Route 53 and an AWS Application Load Balancer managed by the AWS Load Balancer Controller. Terraform owns infrastructure such as VPC, EKS, IAM, ECR and databases, while Argo CD owns application Kubernetes resources.
>
> Secrets are kept outside normal Git configuration using a secret-management system such as AWS Secrets Manager and an appropriate Kubernetes integration. Prometheus and Grafana provide metrics and dashboards, while ELK provides centralized logs.
>
> For production, I also consider multi-AZ deployment, HPA, PDB, probes, resource requests and limits, RBAC, NetworkPolicy, immutable images, protected Git branches, rollback, auditability, HA and disaster recovery. The key principle is clear ownership: CI builds artifacts, Git stores desired deployment state, Argo CD reconciles it, Kubernetes runs the workloads, and AWS controllers manage AWS-integrated resources.

---

# 303. Final Senior-Level Mental Model

Remember:

```text
GitOps
    =
Declarative desired state
+
Git
+
Reconciliation
+
Automation
```

Argo CD:

```text
Git
 ↓
Desired State
 ↓
Compare
 ↓
Sync
 ↓
Kubernetes
 ↓
Health
 ↓
Drift Detection
 ↓
Reconcile
```

Production:

```text
Developer
 ↓
Git
 ↓
CI
 ↓
Security
 ↓
ECR
 ↓
GitOps
 ↓
Argo CD
 ↓
EKS
 ↓
ALB
 ↓
Users

Monitoring:
Prometheus
Grafana
ELK

Infrastructure:
Terraform

Secrets:
AWS Secrets Manager
```

---

# 304. Final Interview Principle

Do not answer GitOps questions as isolated tool definitions.

Connect the systems:

```text
Git
   ↓
CI
   ↓
Security
   ↓
ECR
   ↓
GitOps repository
   ↓
Argo CD
   ↓
Kubernetes
   ↓
AWS
   ↓
Application
   ↓
Observability
   ↓
Feedback
```

When you can explain this complete lifecycle, including:

```text
security
failure
rollback
drift
multi-cluster
DR
ownership
```

you are demonstrating production-level GitOps understanding rather than only memorized Argo CD commands.
