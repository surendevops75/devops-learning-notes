# GitOps-Multi-Environment-Strategy

## 1. Purpose

A production GitOps platform must manage environments without creating configuration chaos.

The goal is not simply:

```text
dev/
qa/
prod/
```

The goal is to create a controlled promotion system where:

```text
Same application artifact
        |
        v
DEV
        |
        v
QA
        |
        v
PROD
```

while allowing legitimate environment differences such as:

```text
replicas
resources
URLs
autoscaling
feature flags
external endpoints
secrets references
AWS region
cluster destination
```

This file covers:

- Dev/QA/Prod strategy
- Environment isolation
- Repository models
- Branching strategies
- Directory strategies
- Helm values
- Kustomize overlays
- ApplicationSets
- Multi-cluster environments
- Image promotion
- Immutable tags and digests
- CI + GitOps promotion
- Production approvals
- Configuration management
- Secrets
- Environment parity
- Drift
- Rollbacks
- Disaster recovery
- RoboShop promotion
- Production YAMLs
- Security
- Troubleshooting
- Interview preparation

---

# 2. What Is a GitOps Environment Strategy?

An environment strategy defines:

```text
Where application configuration lives
How environments differ
How changes are promoted
Who approves production
Which cluster receives each environment
How secrets are handled
How rollback works
```

A good strategy makes the path predictable:

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
DEV
   |
   v
QA
   |
   v
PROD
```

---

# 3. Why Environment Strategy Matters

Without a clear strategy, teams often create:

```text
dev.yaml
dev-final.yaml
dev-final-v2.yaml
prod-new.yaml
prod-new-fixed.yaml
prod-final.yaml
```

This causes:

```text
Configuration drift
Unclear ownership
Unsafe promotion
Difficult rollback
Manual changes
Production surprises
```

GitOps should reduce this complexity.

---

# 4. Desired Properties

A production environment strategy should provide:

```text
Reproducibility
Auditability
Isolation
Promotion control
Rollback capability
Environment parity
Security
Clear ownership
Low duplication
```

---

# 5. Environment Model

A common model:

```text
DEV
 |
 v
QA
 |
 v
PROD
```

Each environment has:

```text
Git configuration
Kubernetes destination
Secrets
Resource sizing
Observability
Approval policy
```

---

# 6. Environment vs Cluster

An environment is a logical concept.

A cluster is infrastructure.

One environment can contain:

```text
one cluster
```

or:

```text
multiple clusters
```

For example:

```text
PROD
 |
 +--> EKS-PROD-PRIMARY
 |
 +--> EKS-PROD-DR
```

---

# 7. Environment vs AWS Account

Similarly:

```text
Environment
```

does not have to equal:

```text
AWS account
```

But production organizations often isolate:

```text
DEV -> DEV account
QA  -> QA account
PROD -> PROD account
```

for stronger security boundaries.

---

# 8. Recommended RoboShop Model

For the user's platform:

```text
DEV
  |
  +--> EKS-DEV
  +--> AWS DEV Account

QA
  |
  +--> EKS-QA
  +--> AWS QA Account

PROD
  |
  +--> EKS-PROD
  +--> AWS PROD Account
```

Optional:

```text
PROD-DR
  |
  +--> EKS-DR
  +--> DR AWS Account/Region
```

---

# 9. Environment Isolation

Isolation should exist at multiple levels:

```text
Git
AWS account
EKS cluster
Namespace
IAM
Secrets
Argo CD Project
```

For example:

```text
DEV developer
```

should not automatically be able to:

```text
sync PROD
```

---

# 10. Argo CD Project Boundary

A strong model is:

```text
roboshop-dev
roboshop-qa
roboshop-prod
```

Each Project can restrict:

```text
repositories
clusters
namespaces
resource types
```

---

# 11. Environment Naming

Use consistent names.

Example:

```text
roboshop-dev
roboshop-qa
roboshop-prod
```

Clusters:

```text
eks-roboshop-dev
eks-roboshop-qa
eks-roboshop-prod
```

Namespaces:

```text
roboshop-dev
roboshop-qa
roboshop
```

Production may use a clean namespace such as:

```text
roboshop
```

inside a dedicated production cluster.

---

# 12. Repository Strategy Overview

Common GitOps repository models include:

```text
1. Single repository
2. Environment directories
3. Branch per environment
4. Repository per environment
5. Application repo + separate GitOps repo
6. Platform repo + application GitOps repo
```

There is no single universal answer.

---

# 13. Application Repository vs GitOps Repository

Recommended separation:

```text
Application Repository
        |
        +--> source code
        +--> tests
        +--> Dockerfile
        +--> application build

GitOps Repository
        |
        +--> Helm
        +--> manifests
        +--> environment values
        +--> Argo CD
```

This is especially useful for the user's Jenkins/GitHub Actions + Argo CD architecture.

---

# 14. Application Repository

Example:

```text
roboshop-cart/
├── src/
├── tests/
├── Dockerfile
├── pom.xml
└── README.md
```

CI operates here.

---

# 15. GitOps Repository

Example:

```text
roboshop-gitops/
├── applications/
├── applicationsets/
├── projects/
├── charts/
├── environments/
└── platform/
```

Argo CD operates from here.

---

# 16. Why Separate Repositories?

Benefits:

```text
Clear ownership
Separate permissions
Independent release lifecycle
Production GitOps controls
Reduced CI cluster access
Cleaner audit trail
```

---

# 17. Single GitOps Repository

Example:

```text
roboshop-gitops/
├── dev/
├── qa/
└── prod/
```

Advantages:

```text
Easy global visibility
Simple onboarding
Centralized governance
```

Risks:

```text
Large repository
Broad access
More coordination
```

---

# 18. Multiple GitOps Repositories

Example:

```text
roboshop-dev-gitops
roboshop-qa-gitops
roboshop-prod-gitops
```

Benefits:

```text
Strong environment isolation
Independent access
Production repository protection
```

Costs:

```text
More repositories
More maintenance
Potential duplication
```

---

# 19. Recommended Enterprise Pattern

A common compromise:

```text
One GitOps repository
+
strong environment directories
+
CODEOWNERS
+
branch protection
+
Argo CD Projects
```

This gives central visibility without necessarily duplicating everything.

---

# 20. Directory-Based Environment Strategy

Example:

```text
environments/
├── dev/
├── qa/
└── prod/
```

This is easy to understand.

---

# 21. Environment Directory Structure

```text
environments/
├── dev/
│   ├── values.yaml
│   └── applications.yaml
│
├── qa/
│   ├── values.yaml
│   └── applications.yaml
│
└── prod/
    ├── values.yaml
    └── applications.yaml
```

---

# 22. Application-Centric Structure

Another model:

```text
applications/
├── cart/
│   ├── base/
│   └── environments/
│       ├── dev/
│       ├── qa/
│       └── prod/
│
├── user/
│   ├── base/
│   └── environments/
│       ├── dev/
│       ├── qa/
│       └── prod/
```

This makes application ownership obvious.

---

# 23. Platform-Centric Structure

Another model:

```text
platform/
├── ingress/
├── monitoring/
├── secrets/
└── security/
```

Business applications remain separate.

---

# 24. Recommended Combined Structure

For RoboShop:

```text
roboshop-gitops/
│
├── applicationsets/
│
├── projects/
│
├── platform/
│
├── charts/
│
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
└── clusters/
    ├── dev/
    ├── qa/
    └── prod/
```

---

# 25. Environment Configuration Principle

Use:

```text
Common configuration
+
small environment-specific overrides
```

Avoid:

```text
Three completely independent copies
```

because they eventually diverge.

---

# 26. Configuration Duplication Problem

Bad:

```text
dev deployment.yaml
qa deployment.yaml
prod deployment.yaml
```

with 200 identical lines in each.

A bug fixed in one file may remain in another.

---

# 27. Better Approach

Use a common base:

```text
base/
```

and environment-specific overlays:

```text
overlays/
├── dev/
├── qa/
└── prod/
```

Kustomize is well suited to this model.

---

# 28. Helm Environment Strategy

Helm can use:

```text
values.yaml
values-dev.yaml
values-qa.yaml
values-prod.yaml
```

Example:

```text
charts/roboshop/
├── Chart.yaml
├── values.yaml
└── templates/

values/
├── dev.yaml
├── qa.yaml
└── prod.yaml
```

---

# 29. Helm Base Values

Example:

```yaml
replicaCount: 2

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart

service:
  port: 80
  targetPort: 8080
```

---

# 30. DEV Values

```yaml
replicaCount: 1

image:
  tag: git-abc123

resources:
  requests:
    cpu: 50m
    memory: 64Mi
  limits:
    cpu: 200m
    memory: 256Mi
```

---

# 31. QA Values

```yaml
replicaCount: 2

image:
  tag: git-abc123

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 400m
    memory: 512Mi
```

---

# 32. PROD Values

```yaml
replicaCount: 3

image:
  tag: git-abc123

resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 1000m
    memory: 1Gi
```

The exact values should be based on production load testing.

---

# 33. Environment Differences

Legitimate differences include:

```text
replicas
CPU/memory
HPA limits
URLs
feature flags
AWS region
external service endpoints
database endpoints
logging level
availability requirements
```

Do not make arbitrary differences.

---

# 34. Environment Parity

The closer environments are structurally, the lower the risk of:

```text
Works in DEV
Fails in PROD
```

Aim for:

```text
Same chart
Same container
Same deployment pattern
Same probes
Same security model
```

with controlled differences.

---

# 35. Artifact Parity

The most important promotion principle:

```text
Build once
Promote the same artifact
```

Do not rebuild:

```text
DEV image
QA image
PROD image
```

from the same source and assume they are identical.

---

# 36. Why Rebuilding Is Dangerous

Even with the same source:

```text
Dependency changes
Base image changes
Build environment changes
Timestamp differences
```

can produce different artifacts.

Instead:

```text
Build -> scan -> publish -> promote
```

the same immutable image.

---

# 37. Immutable Tags

Good:

```text
cart:git-a1b2c3
```

Better:

```text
cart@sha256:<digest>
```

Avoid production:

```text
cart:latest
```

---

# 38. Git Commit and Image Tag

A useful mapping:

```text
Git commit:
a1b2c3d

Image:
cart:git-a1b2c3d
```

This makes the deployment traceable.

---

# 39. GitOps Traceability

You should be able to answer:

```text
Which source commit produced this image?
Which image is running?
Which GitOps commit deployed it?
Who approved production?
Which Argo CD Application deployed it?
Which cluster is running it?
```

This is a major GitOps advantage.

---

# 40. Image Digest Traceability

For strongest traceability:

```text
Source commit
     |
     v
Image digest
     |
     v
GitOps commit
     |
     v
Argo CD revision
     |
     v
Kubernetes workload
```

---

# 41. Promotion Models

Common models:

```text
1. Automatic promotion
2. PR-based promotion
3. Manual approval
4. Git tag-based promotion
5. Release branch
6. Image updater automation
```

Production often uses stronger approval controls than DEV.

---

# 42. Automatic DEV Promotion

Example:

```text
Developer merge
   |
   v
CI
   |
   v
Build/scan
   |
   v
ECR
   |
   v
Update DEV GitOps
   |
   v
Argo CD
   |
   v
EKS DEV
```

---

# 43. QA Promotion

After DEV validation:

```text
DEV success
   |
   v
Promotion PR
   |
   v
QA GitOps
   |
   v
Argo CD
   |
   v
EKS QA
```

---

# 44. Production Promotion

A controlled production flow:

```text
QA validation
   |
   v
Production promotion PR
   |
   v
Review
   |
   v
Approval
   |
   v
Merge
   |
   v
Argo CD
   |
   v
EKS PROD
```

---

# 45. Separation of Duties

Production should ideally require:

```text
Developer creates change
+
Reviewer approves
+
Argo CD deploys
```

rather than:

```text
Developer directly changes cluster
```

---

# 46. Branching Strategy

Possible strategies:

```text
main
develop
release/*
```

But GitOps does not require a complex Git branching model.

A simpler strategy can be safer.

---

# 47. Main-Only GitOps

Example:

```text
main
 |
 +--> dev/
 +--> qa/
 +--> prod/
```

Promotion happens through PRs modifying the appropriate environment configuration.

Advantages:

```text
Single source
Simple history
Easy audit
Less merge complexity
```

---

# 48. Branch-per-Environment

Example:

```text
dev
qa
prod
```

Promotion:

```text
merge dev -> qa
merge qa -> prod
```

Potential problem:

```text
Branches can drift.
```

This can make it harder to know the exact desired state.

---

# 49. Directory-per-Environment

Example:

```text
main
 |
 +--> environments/dev
 +--> environments/qa
 +--> environments/prod
```

Promotion is:

```text
change prod reference
```

instead of:

```text
merge long-lived branches
```

This is often easier to reason about.

---

# 50. Repository-per-Environment

Example:

```text
roboshop-dev-gitops
roboshop-qa-gitops
roboshop-prod-gitops
```

Strong isolation, but more operational overhead.

---

# 51. Recommended Strategy for RoboShop

A practical model:

```text
Application repositories
        |
        v
One central GitOps repository
        |
        +--> dev
        +--> qa
        +--> prod
        |
        v
Argo CD Projects
        |
        v
ApplicationSets
```

Production directories are protected with:

```text
CODEOWNERS
branch protection
required reviews
```

---

# 52. Production Git Repository

```text
roboshop-gitops/
│
├── applicationsets/
│   ├── dev.yaml
│   ├── qa.yaml
│   └── prod.yaml
│
├── projects/
│   ├── dev.yaml
│   ├── qa.yaml
│   └── prod.yaml
│
├── charts/
│   └── roboshop/
│
├── environments/
│   ├── dev/
│   │   └── values/
│   ├── qa/
│   │   └── values/
│   └── prod/
│       └── values/
│
└── platform/
```

---

# 53. Helm-Based ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-environments
  namespace: argocd

spec:
  generators:
    - list:
        elements:
          - environment: dev
            cluster: eks-dev
            namespace: roboshop-dev
            valuesFile: values/dev.yaml

          - environment: qa
            cluster: eks-qa
            namespace: roboshop-qa
            valuesFile: values/qa.yaml

          - environment: prod
            cluster: eks-prod
            namespace: roboshop
            valuesFile: values/prod.yaml

  template:
    metadata:
      name: 'roboshop-{{environment}}'

    spec:
      project: 'roboshop-{{environment}}'

      source:
        repoURL: https://github.com/example/roboshop-gitops.git
        targetRevision: main
        path: charts/roboshop

        helm:
          valueFiles:
            - '{{valuesFile}}'

      destination:
        name: '{{cluster}}'
        namespace: '{{namespace}}'

      syncPolicy:
        automated:
          prune: true
          selfHeal: true

        syncOptions:
          - CreateNamespace=true
```

The exact template interpolation behavior should be validated against the ApplicationSet/Argo CD version in use.

---

# 54. Why This ApplicationSet Is Useful

One template defines:

```text
Application structure
Helm chart
Sync behavior
Project
```

The list generator supplies:

```text
environment
cluster
namespace
values
```

This reduces duplicate Application manifests.

---

# 55. Production Caution

Do not assume:

```text
dev
qa
prod
```

should all have:

```text
automated prune=true
selfHeal=true
```

Production sync policy should reflect change-management requirements.

---

# 56. Manual Production Sync

A production Application can intentionally omit automated sync:

```yaml
syncPolicy: {}
```

or use a controlled sync process.

The exact configuration should follow the organization's approval model.

---

# 57. Automatic DEV, Controlled PROD

A common model:

```text
DEV
  automated sync
  self-heal
  prune

QA
  automated or controlled

PROD
  manual/approved sync
```

This creates increasing change control as risk increases.

---

# 58. Environment Promotion PR

Example:

```text
DEV currently:
cart: git-a1b2c3

QA currently:
cart: git-998877
```

After testing:

```text
QA:
cart: git-a1b2c3
```

The promotion PR changes only the desired artifact reference.

---

# 59. Production Promotion PR

Example:

```text
PROD:
cart: git-998877
```

Change:

```text
cart: git-a1b2c3
```

The PR should show:

```text
old image
new image
commit
release notes
security results
test results
```

---

# 60. Promotion Automation

CI can automatically create:

```text
Promotion PR
```

instead of directly editing production.

Example:

```text
CI
 |
 v
Image published
 |
 v
DEV GitOps update
 |
 v
DEV validation
 |
 v
Promotion PR
 |
 v
QA
 |
 v
Promotion PR
 |
 v
PROD
```

---

# 61. Image Updater Pattern

Argo CD Image Updater or another approved automation can update image references.

However, automatic image promotion should be governed carefully.

For production:

```text
Automatic image discovery
```

does not necessarily mean:

```text
Automatic production deployment
```

---

# 62. Security Scanning Before Promotion

For the user's DevSecOps flow:

```text
Build
 |
 +--> SonarQube
 +--> Trivy
 +--> Veracode
 |
 v
ECR
 |
 v
Promotion
```

Only images that satisfy policy should advance.

---

# 63. Security Gate

Example:

```text
Critical vulnerability
      |
      v
Promotion blocked
```

This is better than:

```text
Deploy first
scan later
```

for release-critical policies.

---

# 64. GitOps and SAST

SonarQube can inspect:

```text
Source code quality
Security issues
Code smells
```

before image promotion.

---

# 65. GitOps and Container Scanning

Trivy can scan:

```text
Container image
Filesystem
Configuration
```

according to the organization's CI policy.

---

# 66. GitOps and DAST

Veracode or other approved security tooling can provide relevant application security validation.

The exact DAST/SAST/SCA classification should match the actual scanner configuration.

---

# 67. Environment Configuration vs Secret Configuration

Normal configuration:

```text
LOG_LEVEL=INFO
ENVIRONMENT=prod
```

can be Git-managed.

Secrets:

```text
DB_PASSWORD
API_TOKEN
AWS_SECRET
```

should use a secure secret-management system.

---

# 68. Environment-Specific Secrets

Example:

```text
AWS Secrets Manager

dev/roboshop/cart
qa/roboshop/cart
prod/roboshop/cart
```

Each environment has independent secret values.

---

# 69. Environment-Specific AWS IAM

A strong model:

```text
DEV ServiceAccount -> DEV IAM role
QA ServiceAccount  -> QA IAM role
PROD ServiceAccount -> PROD IAM role
```

Avoid:

```text
same IAM role
```

for every environment.

---

# 70. Environment-Specific Resource Sizing

DEV:

```text
1 replica
low CPU
low memory
```

PROD:

```text
3+ replicas
higher resources
HPA
PDB
```

This is a legitimate environment difference.

---

# 71. Pod Disruption Budget

Production may use:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: cart
  namespace: roboshop
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: cart
```

This protects availability during voluntary disruptions.

---

# 72. Production HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: cart
  namespace: roboshop
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: cart

  minReplicas: 3
  maxReplicas: 15

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

Production values should come from load testing and SLO requirements.

---

# 73. Production Resources

Every production workload should normally define:

```text
CPU requests
Memory requests
CPU limits where appropriate
Memory limits
```

This improves:

```text
Scheduling
Capacity planning
HPA behavior
Reliability
```

---

# 74. Production Probes

At minimum evaluate:

```text
Readiness
Liveness
Startup
```

Readiness answers:

```text
Can this pod receive traffic?
```

Liveness answers:

```text
Should Kubernetes restart it?
```

Startup protects slow-starting applications.

---

# 75. Environment Parity for Probes

Do not use:

```text
different health endpoints
```

without a strong reason.

Prefer:

```text
same application behavior
```

with environment-specific external dependencies where needed.

---

# 76. Configuration Drift Between Environments

Drift can happen when:

```text
DEV chart updated
QA not updated
PROD not updated
```

This is not necessarily Kubernetes drift.

It is:

```text
environment configuration divergence
```

Track it through Git.

---

# 77. Environment Drift Detection

A useful review asks:

```text
What differs between DEV and QA?
What differs between QA and PROD?
Are all differences intentional?
```

Avoid unexplained divergence.

---

# 78. Golden Configuration

Define a common baseline:

```text
Base chart
Base securityContext
Base probes
Base service
Base deployment
```

Then environments override only:

```text
replicas
resources
URLs
feature flags
```

---

# 79. Helm Values Inheritance

Conceptually:

```text
base values
     |
     +--> dev overrides
     |
     +--> qa overrides
     |
     +--> prod overrides
```

Keep the base reusable.

---

# 80. Kustomize Overlay Model

Example:

```text
base/
├── deployment.yaml
├── service.yaml
└── kustomization.yaml

overlays/
├── dev/
│   └── kustomization.yaml
├── qa/
│   └── kustomization.yaml
└── prod/
    └── kustomization.yaml
```

---

# 81. Kustomize Advantages

Useful when you want:

```text
Declarative patches
Base/overlay structure
Environment-specific changes
No template language
```

---

# 82. Helm vs Kustomize

Helm is strong for:

```text
Reusable application packages
Parameterization
Complex templates
Chart distribution
```

Kustomize is strong for:

```text
Overlay-based configuration
Patch-oriented changes
Native Kubernetes tooling
```

Both can be managed by Argo CD.

---

# 83. Which Should RoboShop Use?

If the platform already uses:

```text
Helm
```

a Helm-centric GitOps model is practical.

Example:

```text
One chart
+
environment-specific values
+
ApplicationSet
```

Kustomize can be introduced where overlay behavior is more appropriate.

---

# 84. Environment-Specific Helm Values

Example:

```text
charts/roboshop/
values/
├── dev.yaml
├── qa.yaml
└── prod.yaml
```

Avoid putting everything into one giant values file.

---

# 85. Per-Service Values

For a microservices platform:

```text
values/
├── dev/
│   ├── cart.yaml
│   ├── user.yaml
│   ├── payment.yaml
│   └── frontend.yaml
│
├── qa/
└── prod/
```

This can make ownership clearer.

---

# 86. Environment ApplicationSet Matrix

For multiple services and environments:

```text
services
   x
environments
```

can be represented conceptually using a Matrix generator.

Example:

```text
cart x dev
cart x qa
cart x prod

user x dev
user x qa
user x prod
```

This can generate many Applications automatically.

---

# 87. Matrix Explosion Risk

If:

```text
20 services
x
10 clusters
x
3 environments
```

you can generate:

```text
600 Applications
```

This may be valid, but operational complexity increases.

Use ApplicationSets deliberately.

---

# 88. Application Count Planning

Before creating generators, estimate:

```text
services
clusters
environments
platform applications
```

Then calculate approximate Application count.

This helps size:

```text
Argo CD controllers
Repo Server
Redis
```

---

# 89. Production Environment Naming in ApplicationSet

Example:

```text
roboshop-cart-dev
roboshop-cart-qa
roboshop-cart-prod
```

Avoid ambiguous names.

---

# 90. Multi-Cluster Environment Mapping

Example:

```text
environment=prod
region=ap-south-1
role=primary
```

ApplicationSet can select:

```text
PROD primary
```

while another selects:

```text
PROD DR
```

---

# 91. Primary and DR Configuration

Primary:

```yaml
replicas: 6
```

DR:

```yaml
replicas: 3
```

The application artifact remains the same.

Only operational configuration differs.

---

# 92. Production Approval Strategy

Possible controls:

```text
Git branch protection
CODEOWNERS
Required PR review
Security checks
Change ticket
Manual Argo CD sync
```

Use the controls appropriate to your organization.

---

# 93. Who Should Approve Production?

A common model:

```text
Developer
   |
   v
Service owner review
   |
   v
Platform/release approval
   |
   v
Production merge
```

The exact chain depends on team structure.

---

# 94. GitOps Audit Trail

A production deployment can be reconstructed from:

```text
Git commit
PR
Reviewer
Image tag/digest
Argo CD revision
Application history
Kubernetes resource
```

This is much easier than reconstructing a sequence of manual:

```text
kubectl apply
```

commands.

---

# 95. Rollback Strategy

Rollback can happen at multiple levels:

```text
Git rollback
Image rollback
Argo CD application rollback
Kubernetes rollout undo
```

GitOps should prefer restoring desired state through Git.

---

# 96. Git Revert

Suppose:

```text
prod image = git-new
```

and it causes failure.

Revert the GitOps commit:

```text
prod image = git-old
```

Argo CD detects the new desired state and reconciles.

---

# 97. Argo CD Rollback

Argo CD also has application history and rollback mechanisms.

Example:

```bash
argocd app history roboshop-prod
```

Then:

```bash
argocd app rollback roboshop-prod <id>
```

Exact CLI behavior depends on Argo CD version.

For durable GitOps auditability, reconcile the final desired state back into Git.

---

# 98. Kubernetes Rollout Rollback

Kubernetes can also rollback:

```bash
kubectl rollout history deployment/cart -n roboshop
kubectl rollout undo deployment/cart -n roboshop
```

But a manual rollback may create Git drift.

Therefore:

```text
Emergency rollback
+
Git correction
```

should be the operational model.

---

# 99. GitOps Rollback Principle

Preferred:

```text
Git
 |
 v
Argo CD
 |
 v
Kubernetes
```

Emergency:

```text
Kubernetes
 |
 v
Immediate rollback
 |
 v
Git correction
```

Never leave the emergency state unexplained.

---

# 100. Progressive Promotion

Promotion can be:

```text
DEV
 |
 v
QA
 |
 v
PROD CANARY
 |
 v
PROD FULL
```

This prepares for the later Progressive Delivery section.

---

# 101. Production Canary Cluster

If multiple production clusters exist:

```text
PROD-CANARY
PROD-PRIMARY
PROD-SECONDARY
```

Deploy to canary first.

Validate:

```text
error rate
latency
resource usage
business metrics
```

Then promote.

---

# 102. Multi-Cluster Production Rollout

```text
EKS-PROD-01
    |
    v
Validate
    |
    v
EKS-PROD-02
    |
    v
Validate
    |
    v
EKS-PROD-03
```

This reduces fleet-wide blast radius.

---

# 103. ApplicationSet for Cluster Waves

ApplicationSet can select groups of clusters using labels.

Example:

```text
rollout=wave1
rollout=wave2
rollout=wave3
```

Then create separate ApplicationSets or controlled generator logic.

---

# 104. Why Not Deploy Everywhere Immediately?

Suppose 20 production clusters receive:

```text
bad image
```

simultaneously.

Impact:

```text
20 clusters
```

A staged rollout can limit impact to:

```text
1 cluster
```

first.

---

# 105. Environment Freeze

During major incidents or release windows:

```text
Production promotion may be frozen.
```

GitOps makes this operationally visible.

Possible controls:

```text
Pause promotion automation
Require manual approval
Restrict merge permissions
```

---

# 106. Release Windows

Some organizations define:

```text
Normal release window
Change freeze
Emergency release
```

GitOps should integrate with these processes.

---

# 107. Git Tags for Releases

A production image may map to:

```text
v2.4.1
```

while the container uses:

```text
git-a1b2c3d
```

Keep both release semantics and immutable artifact identity where useful.

---

# 108. Semantic Versioning

Application releases may use:

```text
v2.4.1
```

but container tags should still be immutable.

If tags can be overwritten, the same tag may point to different content.

Prefer:

```text
version + immutable digest
```

where practical.

---

# 109. Release Metadata

A GitOps promotion PR can include:

```text
Application: cart
Version: v2.4.1
Image: git-a1b2c3d
Digest: sha256:...
Source commit: a1b2c3d
Security status: passed
QA status: passed
```

This makes releases auditable.

---

# 110. Production YAML: Helm Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cart-prod
  namespace: argocd

spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/example/roboshop-gitops.git
    targetRevision: main
    path: charts/roboshop

    helm:
      releaseName: cart

      valueFiles:
        - values/prod/cart.yaml

  destination:
    name: eks-prod
    namespace: roboshop

  syncPolicy:
    automated:
      prune: false
      selfHeal: true

    syncOptions:
      - CreateNamespace=true
```

For production, `prune` should be an intentional policy decision.

---

# 111. Production YAML: Kustomize Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cart-prod
  namespace: argocd

spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/example/roboshop-gitops.git
    targetRevision: main
    path: applications/cart/overlays/prod

  destination:
    name: eks-prod
    namespace: roboshop

  syncPolicy:
    automated:
      selfHeal: true
```

---

# 112. Production YAML: Environment Project

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: roboshop-prod
  namespace: argocd

spec:
  description: RoboShop production applications

  sourceRepos:
    - https://github.com/example/roboshop-gitops.git

  destinations:
    - name: eks-prod
      namespace: roboshop
      server: https://kubernetes.default.svc

  namespaceResourceWhitelist:
    - group: apps
      kind: Deployment
    - group: ""
      kind: Service
    - group: ""
      kind: ConfigMap
    - group: ""
      kind: Secret
    - group: autoscaling
      kind: HorizontalPodAutoscaler
    - group: networking.k8s.io
      kind: Ingress
```

Destination configuration must match the actual Argo CD cluster registration.

---

# 113. Production YAML: Environment ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-environments
  namespace: argocd

spec:
  generators:
    - list:
        elements:
          - environment: dev
            cluster: eks-dev
            namespace: roboshop-dev

          - environment: qa
            cluster: eks-qa
            namespace: roboshop-qa

          - environment: prod
            cluster: eks-prod
            namespace: roboshop

  template:
    metadata:
      name: 'roboshop-{{environment}}'

    spec:
      project: 'roboshop-{{environment}}'

      source:
        repoURL: https://github.com/example/roboshop-gitops.git
        targetRevision: main
        path: 'environments/{{environment}}'

      destination:
        name: '{{cluster}}'
        namespace: '{{namespace}}'
```

---

# 114. Production YAML: Cluster-Label Strategy

Cluster labels should be assigned through the approved cluster registration process.

Example target metadata:

```text
environment=prod
application=roboshop
region=ap-south-1
role=primary
```

ApplicationSet:

```yaml
generators:
  - clusters:
      selector:
        matchLabels:
          environment: prod
          application: roboshop
          role: primary
```

---

# 115. Production YAML: Environment Kustomize Base

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
  - configmap.yaml
```

---

# 116. Production YAML: Dev Overlay

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

nameSuffix: -dev

namespace: roboshop-dev

patches:
  - path: deployment-patch.yaml
```

---

# 117. Production YAML: Prod Overlay

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

namespace: roboshop

patches:
  - path: deployment-patch.yaml
```

---

# 118. Environment Repository Example

```text
roboshop-gitops/
│
├── applications/
│   ├── cart/
│   │   ├── base/
│   │   └── overlays/
│   │       ├── dev/
│   │       ├── qa/
│   │       └── prod/
│   │
│   ├── user/
│   └── payment/
│
├── applicationsets/
│
├── projects/
│
├── platform/
│
└── clusters/
```

---

# 119. Service Ownership

For a microservices platform:

```text
cart team
user team
payment team
catalogue team
frontend team
```

can own:

```text
application chart
service configuration
service release
```

Platform team owns:

```text
Argo CD
EKS
Ingress
observability
security
shared platform
```

---

# 120. Environment Ownership

A possible model:

```text
Developers
   |
   +--> DEV

QA/Release
   |
   +--> QA

Platform/Release
   |
   +--> PROD
```

Exact ownership depends on the organization.

---

# 121. CODEOWNERS

Example:

```text
/applications/cart/       @cart-team
/applications/user/       @user-team
/environments/prod/       @platform-team @release-team
/applicationsets/         @platform-team
/projects/                @platform-team
/clusters/prod/           @platform-team
```

This prevents accidental production configuration changes.

---

# 122. Git Branch Protection

Protect:

```text
main
```

with:

```text
Required pull request
Required reviewers
Status checks
No force push
Restricted branch deletion
```

---

# 123. Production Change Validation

Before merge:

```text
helm lint
helm template
kustomize build
YAML validation
policy checks
security checks
```

Then:

```text
Argo CD diff
```

provides another validation layer.

---

# 124. Argo CD Diff in Promotion

Example:

```bash
argocd app diff roboshop-prod
```

Review:

```text
Deployment image
replicas
resources
Ingress
ConfigMap
HPA
```

before production synchronization.

---

# 125. Production Sync

Manual:

```bash
argocd app sync roboshop-prod
```

Then:

```bash
argocd app get roboshop-prod
```

Verify:

```text
Sync status
Health
Revision
Resources
```

---

# 126. Production History

Use:

```bash
argocd app history roboshop-prod
```

This helps answer:

```text
What revision was deployed?
When?
Which previous revisions existed?
```

---

# 127. Environment Comparison

A useful investigation:

```bash
argocd app diff roboshop-dev
argocd app diff roboshop-qa
argocd app diff roboshop-prod
```

The exact diff output depends on the application source and current live state.

---

# 128. Configuration Drift vs Environment Difference

Important distinction:

### Intended difference

```text
PROD replicas=5
DEV replicas=1
```

This is not drift.

### Unintended difference

```text
PROD Git says image=A
PROD cluster has image=B
```

This is drift.

---

# 129. Drift Detection

Argo CD compares:

```text
Desired state
```

with:

```text
Live state
```

If they differ:

```text
OutOfSync
```

may be reported.

---

# 130. Self-Healing

With:

```yaml
automated:
  selfHeal: true
```

Argo CD can restore the Git-defined state after unauthorized/manual changes, subject to Argo CD behavior and resource-specific exceptions.

---

# 131. Production Self-Heal

Self-heal is powerful but should be evaluated carefully.

If an operator intentionally makes an emergency change:

```text
kubectl patch
```

Argo CD may revert it.

Therefore emergency procedures must include:

```text
Pause/reconcile strategy
+
Git correction
```

---

# 132. Prune

Prune means:

```text
Remove resources that are no longer declared.
```

Example:

Git removes:

```text
ConfigMap old-config
```

If prune is enabled:

```text
Argo CD can delete old-config.
```

---

# 133. Production Prune Risk

A mistaken Git deletion can cause:

```text
Production resource deletion.
```

Therefore:

```text
review
policy
ownership
sync options
```

must be considered before enabling automatic prune.

---

# 134. Environment-Specific Prune

A reasonable model can be:

```text
DEV: automatic prune
QA: automatic or controlled
PROD: controlled depending on policy
```

There is no universal answer.

---

# 135. Sync Windows

Argo CD can be configured with sync windows to control when automated synchronization is allowed.

Conceptually:

```text
Production
   |
   +--> sync allowed during release window
   |
   +--> sync blocked during freeze
```

This is useful for:

```text
change freezes
maintenance windows
business-critical periods
```

---

# 136. Environment Freeze

During freeze:

```text
Git changes may still be reviewed
```

but:

```text
production synchronization
```

can be restricted according to policy.

This separates:

```text
change preparation
```

from:

```text
change execution
```

---

# 137. Environment Rollback

Suppose production release:

```text
v2.4.1
```

causes errors.

GitOps rollback:

```text
prod desired state
v2.4.1
     |
     v
revert
     |
     v
v2.4.0
     |
     v
Argo CD
     |
     v
EKS
```

---

# 138. Rollback Speed

Rollback is fast when:

```text
previous image exists
previous Git commit exists
database compatibility is maintained
```

Therefore retain:

```text
image versions
Git history
release metadata
```

---

# 139. Database Rollback Warning

Application rollback does not automatically mean:

```text
database rollback
```

A release may include:

```text
schema migration
```

that cannot safely be reversed.

Use backward-compatible database migration patterns.

---

# 140. Expand-and-Contract Migration

A safer database migration pattern:

```text
Expand
   |
   v
Deploy compatible app
   |
   v
Migrate data
   |
   v
Switch behavior
   |
   v
Contract
```

This reduces rollback risk.

---

# 141. Environment Data Isolation

Never point:

```text
DEV application
```

at:

```text
PROD database
```

unless explicitly required and secured.

Environment-specific endpoints should be controlled through configuration.

---

# 142. Environment Network Isolation

A strong model:

```text
DEV VPC
QA VPC
PROD VPC
```

with controlled cross-account connectivity.

Do not allow broad:

```text
DEV -> PROD
```

network access.

---

# 143. Production Namespace Isolation

Within an EKS cluster:

```text
roboshop
monitoring
platform
```

can be separate namespaces.

Namespace isolation is useful but is not equivalent to an AWS account boundary.

---

# 144. ResourceQuota

Production namespace can use:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: roboshop
  namespace: roboshop

spec:
  requests.cpu: "20"
  requests.memory: 40Gi
  limits.cpu: "40"
  limits.memory: 80Gi
```

Values should be based on cluster capacity and workload requirements.

---

# 145. LimitRange

A LimitRange can provide defaults or constraints for resources.

This can prevent workloads from being deployed without resource specifications.

---

# 146. Environment Governance

Each environment should have:

```text
Allowed repositories
Allowed clusters
Allowed namespaces
Allowed resource types
Allowed teams
Approval policy
Sync policy
```

Argo CD Projects help enforce many of these boundaries.

---

# 147. Environment Security Matrix

Example:

| Environment | Sync | Approval | Access |
|---|---|---|---|
| DEV | Automated | Minimal | Developers |
| QA | Automated/controlled | QA/Release | QA + Dev |
| PROD | Controlled | Required | Platform/Release |

This is a policy example, not a universal rule.

---

# 148. Production Access Matrix

```text
Developer
   |
   +--> View PROD
   |
   +--> No direct sync

Release Engineer
   |
   +--> Approve PROD

Platform Team
   |
   +--> Manage Argo CD
   +--> Manage clusters
```

Least privilege should be applied.

---

# 149. Environment Promotion Audit

Record:

```text
Source commit
Image digest
GitOps commit
PR number
Reviewer
Argo CD revision
Cluster
Timestamp
```

This supports incident response and compliance.

---

# 150. Promotion Automation Design

A CI system can:

```text
1. Build image.
2. Scan image.
3. Push image to ECR.
4. Update DEV GitOps.
5. Open QA promotion PR after DEV validation.
6. Open PROD promotion PR after QA validation.
```

Argo CD handles deployment.

---

# 151. CI Should Not Edit Production Directly

Avoid:

```bash
kubectl apply -f prod/
```

from CI.

Prefer:

```text
CI -> GitOps PR
```

and:

```text
Argo CD -> Kubernetes
```

---

# 152. Production Promotion Example

```text
CI Build #842
      |
      v
Image:
cart:git-a1b2c3d
      |
      v
ECR
      |
      v
DEV GitOps
      |
      v
Argo CD DEV
      |
      v
Tests
      |
      v
QA promotion PR
      |
      v
Argo CD QA
      |
      v
Validation
      |
      v
PROD promotion PR
      |
      v
Approval
      |
      v
Argo CD PROD
```

---

# 153. GitOps Release Artifact

A release manifest can contain:

```yaml
release:
  version: v2.4.1
  commit: a1b2c3d
  image:
    repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
    tag: git-a1b2c3d
    digest: sha256:...
```

This can improve traceability.

---

# 154. Environment Promotion Using Digests

Production can use:

```yaml
image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
  digest: sha256:...
```

The exact Helm chart structure determines how digest references are represented.

---

# 155. Tag vs Digest

Tag:

```text
human-friendly
```

Digest:

```text
content-immutable
```

A mature system may track both.

---

# 156. Environment Configuration Validation

Before promotion:

```text
Does the chart render?
Does the namespace exist?
Does the service exist?
Does the image exist?
Are secrets references valid?
Are resource values valid?
Is the Ingress valid?
```

Automate as much validation as possible.

---

# 157. Helm Validation

Example:

```bash
helm lint ./charts/roboshop
```

Render:

```bash
helm template roboshop ./charts/roboshop \
  -f values/prod.yaml
```

Then inspect:

```text
image
replicas
resources
Ingress
HPA
```

---

# 158. Kustomize Validation

Example:

```bash
kustomize build applications/cart/overlays/prod
```

Then:

```bash
kubectl apply --dry-run=server -f -
```

where access to the intended target cluster is available and appropriate.

---

# 159. Argo CD Validation

Before production sync:

```bash
argocd app get roboshop-prod
argocd app diff roboshop-prod
```

Review differences.

---

# 160. Environment Failure: DEV Works, QA Fails

Check:

```text
QA values
QA image
QA secrets
QA IAM
QA cluster
QA namespace
QA ingress
```

Do not assume the application code is broken.

---

# 161. Environment Failure: QA Works, PROD Fails

Common differences:

```text
IAM
secrets
network
database
resource sizing
ALB
security policy
image permissions
```

This is why environment parity matters.

---

# 162. Environment Failure: Same Image, Different Behavior

If:

```text
same image digest
```

works in QA but fails in PROD, investigate environment configuration:

```text
ConfigMap
Secret
IAM
DNS
network
database
resource limits
external dependencies
```

The artifact is not necessarily the problem.

---

# 163. Environment Failure: Production Drift

Check:

```bash
argocd app get roboshop-prod
argocd app diff roboshop-prod
```

Then inspect:

```bash
kubectl get deployment cart -n roboshop -o yaml
```

Compare desired and actual state.

---

# 164. Environment Failure: Wrong Image

Check:

```bash
kubectl get deployment cart -n roboshop \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
```

Then compare against GitOps.

---

# 165. Environment Failure: Wrong Cluster

Check:

```bash
argocd app get roboshop-prod
```

Inspect:

```text
destination
server
namespace
```

Also inspect the ApplicationSet generator.

A selector mistake can deploy to the wrong cluster.

---

# 166. Environment Failure: Wrong Namespace

Check:

```bash
kubectl get applications -n argocd
```

and:

```bash
argocd app get roboshop-prod
```

Verify:

```text
destination.namespace
```

---

# 167. Environment Failure: ApplicationSet Generated Unexpected PROD App

Investigate:

```text
cluster labels
selector
generator
template
Project
```

Example accidental label:

```text
environment=prod
```

on a DEV cluster can cause a production ApplicationSet to select it.

---

# 168. Cluster Label Governance

Treat cluster labels as production configuration.

Only approved automation/platform teams should modify:

```text
environment
role
application
```

labels used for deployment selection.

---

# 169. ApplicationSet Selector Safety

Avoid overly broad selectors:

```yaml
matchLabels:
  managed=true
```

if every cluster has:

```text
managed=true
```

Prefer:

```yaml
matchLabels:
  environment: prod
  application: roboshop
```

---

# 170. Environment Generator Safety

List generators are explicit:

```text
dev -> EKS DEV
qa -> EKS QA
prod -> EKS PROD
```

This can be safer for small, tightly controlled environments.

Cluster generators scale better when clusters are dynamically onboarded.

---

# 171. List vs Cluster Generator

List generator:

```text
Explicit mapping
```

Cluster generator:

```text
Dynamic discovery
```

Use list when:

```text
Small environment fleet
Strict explicit mapping
```

Use cluster generator when:

```text
Many clusters
Dynamic onboarding
Label-driven targeting
```

---

# 172. Multi-Cluster Production Mapping

Explicit:

```yaml
- environment: prod
  cluster: eks-prod-01
  role: primary

- environment: prod
  cluster: eks-prod-02
  role: secondary
```

This gives clear visibility.

---

# 173. Cluster Generator Production Mapping

Labels:

```text
environment=prod
application=roboshop
role=primary
```

ApplicationSet selects:

```yaml
matchLabels:
  environment: prod
  application: roboshop
  role: primary
```

This is scalable.

---

# 174. Environment Promotion and Clusters

Promotion should not depend on:

```text
which cluster happens to be online
```

Instead:

```text
Environment desired state
        |
        v
Approved cluster selection
```

---

# 175. Disaster Recovery and GitOps

If PROD cluster is lost:

```text
Provision replacement EKS
        |
        v
Register cluster
        |
        v
Apply environment labels
        |
        v
ApplicationSet generates Apps
        |
        v
Argo CD reconciles
```

This is a major GitOps advantage.

---

# 176. DR Dependency List

Before recovery, verify:

```text
ECR images
Secrets
Database
DNS
ACM
ALB prerequisites
IAM
Network
Argo CD connectivity
```

GitOps alone is not enough.

---

# 177. DR Git Repository

Git should remain available outside the failed application cluster.

Use:

```text
Remote Git provider
Backups
Protected repository
Multiple maintainers
```

Do not store the only copy of GitOps configuration inside the cluster.

---

# 178. Management Cluster DR

For centralized Argo CD:

```text
Management EKS
```

is itself a critical platform dependency.

Recovery plan:

```text
Recreate management EKS
Install Argo CD
Restore configuration
Restore credentials
Register target clusters
Reconcile Applications
```

---

# 179. Argo CD Backup

Back up or preserve:

```text
Applications
ApplicationSets
Projects
RBAC
repository credentials
cluster credentials
Argo CD configuration
```

Sensitive values must be stored securely.

---

# 180. Git as Configuration Backup

Even if the Argo CD management cluster is destroyed:

```text
Git
```

should contain enough information to reconstruct:

```text
Projects
Applications
ApplicationSets
Charts
Environment configuration
```

Credentials may need separate secure recovery.

---

# 181. Environment Disaster Recovery Test

A good test:

```text
1. Create temporary EKS.
2. Register it.
3. Apply prod-like labels.
4. Generate Applications.
5. Validate workloads.
6. Validate secrets.
7. Validate ALB.
8. Validate monitoring.
9. Remove test cluster.
```

This proves the GitOps design works.

---

# 182. Production Readiness Checklist

```text
[ ] Application repo separated from GitOps repo
[ ] Environment model documented
[ ] Cluster mapping documented
[ ] Production Project configured
[ ] CODEOWNERS configured
[ ] Branch protection enabled
[ ] Immutable images used
[ ] Promotion process documented
[ ] Secrets externalized
[ ] CI security gates configured
[ ] Production approval configured
[ ] Rollback tested
[ ] DR tested
[ ] Monitoring configured
[ ] Alerts configured
```

---

# 183. RoboShop Environment Architecture

```text
                   Application Git
                         |
                         v
                  CI / DevSecOps
                         |
              +----------+----------+
              |                     |
              v                     v
             ECR               GitOps Repo
                                    |
                                    v
                              Central Argo CD
                                    |
                 +------------------+------------------+
                 |                  |                  |
                 v                  v                  v
              DEV EKS            QA EKS             PROD EKS
                 |                  |                  |
             RoboShop            RoboShop            RoboShop
                 |                  |                  |
              ALB DEV             ALB QA             ALB PROD
```

---

# 184. RoboShop Promotion Example

Initial:

```text
DEV  -> cart:git-a1b2c3
QA   -> cart:git-998877
PROD -> cart:git-887766
```

After successful DEV testing:

```text
QA -> cart:git-a1b2c3
```

After QA validation:

```text
PROD -> cart:git-a1b2c3
```

The same artifact moves through environments.

---

# 185. RoboShop Production Release

```text
1. Developer commits code.
2. CI runs tests.
3. SonarQube validation.
4. Trivy scan.
5. Veracode validation.
6. Build image.
7. Push immutable image to ECR.
8. Update DEV GitOps.
9. Argo CD deploys DEV.
10. Validate DEV.
11. Promote exact image to QA.
12. Validate QA.
13. Approve PROD promotion.
14. Update PROD GitOps.
15. Argo CD syncs PROD.
16. Validate application health.
17. Monitor Prometheus/Grafana/ELK.
```

---

# 186. Production Deployment Evidence

For every RoboShop release, retain:

```text
Git source commit
CI build number
Security results
ECR image tag
ECR digest
GitOps commit
PR approvals
Argo CD revision
Target cluster
Deployment timestamp
```

---

# 187. Interview Question: How Do You Manage Dev, QA and Prod in GitOps?

### Answer

> I keep application source and GitOps configuration separate. The GitOps repository contains common application definitions with controlled environment-specific values. CI builds and scans one immutable image, and the same artifact is promoted from DEV to QA to PROD through Git changes and approvals. Argo CD reconciles each environment to its EKS cluster.

---

# 188. Interview Question: Branch Per Environment or Directory Per Environment?

### Answer

> I generally prefer a simple main-branch model with environment directories because long-lived environment branches can drift. For example, `environments/dev`, `environments/qa` and `environments/prod` can represent desired state while CODEOWNERS and branch protection control production changes.

---

# 189. Interview Question: How Do You Prevent DEV and PROD From Diverging?

### Answer

> I use a common Helm chart or Kustomize base and keep only intentional environment-specific overrides. I also promote the same immutable image through environments and regularly review configuration differences.

---

# 190. Interview Question: Should Each Environment Use a Different Docker Image?

### Answer

> No. I prefer build-once and promote-the-same-artifact. DEV, QA and PROD should normally use the same image digest. Environment-specific behavior belongs in configuration, not separate rebuilt artifacts.

---

# 191. Interview Question: How Do You Handle Production Approval?

### Answer

> I use a protected GitOps production path with PR review, required status checks and CODEOWNERS. Depending on the organization's change process, production Argo CD synchronization can also be manual or restricted to approved release windows.

---

# 192. Interview Question: Why Use Immutable Image Tags?

### Answer

> Immutable tags or digests make deployments reproducible and auditable. If a tag is overwritten, the same Git commit could deploy different image content later. A digest guarantees the exact image content.

---

# 193. Interview Question: How Do You Roll Back?

### Answer

> My preferred rollback is to revert the GitOps change to the previously known-good image or configuration and let Argo CD reconcile it. In an emergency I may use Kubernetes or Argo CD rollback mechanisms for immediate recovery, but I then reconcile the final state back into Git.

---

# 194. Interview Question: What Is Environment Drift?

### Answer

> Environment drift can mean two things. Kubernetes drift occurs when live state differs from the desired Git state. Environment configuration divergence occurs when DEV, QA and PROD differ beyond the intentionally defined overrides. I address the first with Argo CD reconciliation and the second with common bases and controlled environment overlays.

---

# 195. Interview Question: How Would You Promote RoboShop to Production?

### Answer

> CI builds and scans the image, pushes it to ECR and updates the DEV GitOps configuration. After DEV validation, the same immutable image digest is promoted to QA. After QA validation and required approvals, a production GitOps PR updates the production image reference. Argo CD then synchronizes the approved state to EKS PROD.

---

# 196. Interview Scenario: DEV Works, PROD Fails With Same Image

### Answer

I would compare:

```text
Secrets
ConfigMaps
IAM
network
DNS
database
resource requests/limits
HPA
ALB
external dependencies
```

Because if the image digest is identical, the environment itself becomes the primary suspect.

---

# 197. Interview Scenario: Production Is Automatically Syncing Unexpected Changes

Check:

```text
syncPolicy
automated
selfHeal
prune
sync windows
ApplicationSet
Git commits
```

Then determine whether:

```text
Git changed
```

or:

```text
live state changed
```

---

# 198. Interview Scenario: PROD Cluster Received DEV Configuration

Investigate:

```text
ApplicationSet generator
cluster labels
destination
environment values
Project
```

A wrong cluster label or template parameter can cause incorrect placement.

---

# 199. Interview Scenario: Promotion PR Contains Huge Diff

Likely causes:

```text
Environment duplication
Chart version mismatch
Kustomize base divergence
Unrelated configuration changes
Manual cluster modifications
```

A promotion PR should ideally contain a focused release change.

---

# 200. Interview Scenario: Rollback Is Not Working

Check:

```text
previous image still exists
database compatibility
GitOps revision
Application history
Deployment history
image pull
health probes
```

A rollback may fail because the previous image was deleted or because the database schema is incompatible.

---

# 201. Interview Scenario: Production Image Was Deleted From ECR

Recovery options depend on:

```text
ECR replication
image retention
backup/registry strategy
source rebuild
```

This is why production registries need lifecycle policies that retain rollback-capable artifacts.

---

# 202. Interview Scenario: GitOps Repository Is Unavailable

Existing workloads generally continue running.

However:

```text
new changes
reconciliation after desired-state changes
```

are affected.

When Git returns:

```text
Argo CD refreshes
and reconciles
```

according to its configured behavior.

---

# 203. Interview Scenario: GitOps Repository Was Compromised

Immediate priorities:

```text
Restrict repository access
Stop dangerous sync if required
Review malicious commits
Rotate credentials
Restore trusted revision
Review Argo CD audit data
Assess cluster blast radius
```

Because Git is the desired-state source, repository security is critical.

---

# 204. Production Principles

1. Separate application source from deployment configuration.
2. Build artifacts once.
3. Promote immutable artifacts.
4. Keep common configuration reusable.
5. Keep environment differences explicit.
6. Protect production GitOps paths.
7. Use Projects for environment boundaries.
8. Use ApplicationSets for scale.
9. Avoid long-lived environment branch drift where possible.
10. Keep secrets outside normal Git configuration.
11. Use Git as the durable desired-state source.
12. Use Argo CD as the reconciliation engine.
13. Test rollback.
14. Test disaster recovery.
15. Monitor every environment independently.
16. Treat cluster labels as deployment controls.
17. Do not let CI require broad production cluster access.
18. Keep Terraform and Argo CD ownership separate.
19. Use progressive promotion for high-risk releases.
20. Preserve release evidence.

---

# 205. Final Mental Model

The complete environment strategy is:

```text
                 Application Source
                         |
                         v
                  CI / DevSecOps
                         |
                         v
                    Immutable ECR
                         |
                         v
                    GitOps Repo
                         |
             +-----------+-----------+
             |           |           |
            DEV         QA          PROD
             |           |           |
          Git state    Git state   Git state
             |           |           |
             v           v           v
          Argo CD     Argo CD     Argo CD
             |           |           |
             v           v           v
          EKS DEV     EKS QA     EKS PROD
```

The artifact remains the same.

The desired configuration changes intentionally.

The environment controls:

```text
where
how
when
and by whom
```

the application is deployed.

---