# 33-Complete-GitOps-Repository

## 33.1 Purpose

This chapter defines a realistic production GitOps repository for the RoboShop platform running on AWS EKS.

The repository is responsible for Kubernetes desired state.

The architecture is:

```text
Developer
   |
   v
Application Source Repository
   |
   v
CI Pipeline
   |
   +--> Build
   +--> Unit Tests
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   +--> Container Image
   |
   v
ECR
   |
   v
GitOps Repository
   |
   v
Argo CD
   |
   v
EKS
```

The critical principle is:

```text
Git = desired state
Kubernetes = actual state
Argo CD = reconciliation engine
```

---

# 33.2 Why GitOps

Traditional deployment:

```text
CI
 |
 +--> kubectl apply
 |
 v
Kubernetes
```

GitOps:

```text
CI
 |
 +--> build image
 |
 +--> update GitOps
 |
 v
Git
 |
 v
Argo CD
 |
 v
Kubernetes
```

GitOps provides:

- auditability
- version history
- peer review
- reproducibility
- drift detection
- rollback through Git
- centralized deployment control
- separation between build and deployment

---

# 33.3 Repository Responsibilities

The GitOps repository should contain:

```text
Kubernetes manifests
Helm values
Argo CD Applications
Argo CD Projects
environment configuration
application versions
monitoring configuration
policy configuration
```

It should not normally contain:

```text
application source code
Dockerfiles
Terraform infrastructure
CI credentials
plaintext production secrets
```

---

# 33.4 Recommended Repository

```text
roboshop-gitops/
├── README.md
├── .gitignore
├── CODEOWNERS
├── docs/
├── argocd/
├── environments/
├── applications/
├── platform/
├── monitoring/
├── policies/
└── scripts/
```

---

# 33.5 Complete Structure

```text
roboshop-gitops/
│
├── README.md
├── CODEOWNERS
├── .gitignore
│
├── argocd/
│   ├── project-roboshop.yaml
│   ├── applicationset.yaml
│   └── applications/
│       ├── roboshop-dev.yaml
│       ├── roboshop-qa.yaml
│       └── roboshop-prod.yaml
│
├── applications/
│   ├── cart/
│   │   ├── base/
│   │   └── overlays/
│   ├── catalogue/
│   │   ├── base/
│   │   └── overlays/
│   ├── user/
│   ├── payment/
│   ├── shipping/
│   ├── frontend/
│   └── checkout/
│
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
├── platform/
│   ├── namespaces/
│   ├── ingress/
│   ├── rbac/
│   ├── network-policies/
│   └── resource-policies/
│
├── monitoring/
│   ├── prometheus-rules/
│   ├── alertmanager/
│   └── dashboards/
│
├── policies/
│   ├── pod-security/
│   └── admission/
│
└── scripts/
    ├── validate.sh
    └── diff.sh
```

---

# 33.6 Environment Strategy

Use:

```text
dev
qa
prod
```

Production should have stricter controls.

Example:

```text
dev
  replicas: 1
  autoscaling: limited

qa
  replicas: 2

prod
  replicas: 3+
  PDB enabled
  HPA enabled
  stricter resources
  production alerts
```

---

# 33.7 Branching Strategy

A practical approach:

```text
main
 |
 +--> production desired state
```

Development can use:

```text
feature branches
pull requests
```

Avoid long-lived environment branches when possible.

A single main branch with environment directories provides a clear desired-state model.

---

# 33.8 CODEOWNERS

Example:

```text
* @platform-team

/argocd/ @platform-team
/platform/ @platform-team
/monitoring/ @observability-team
/environments/prod/ @platform-team @sre-team
```

Production changes should require appropriate reviewers.

---

# 33.9 GitOps Security

Protect:

```text
main branch
production directories
Argo CD credentials
repository deploy keys
automation tokens
```

Use:

```text
branch protection
required reviews
status checks
signed commits where required
least-privilege access
```

---

# 33.10 Application Base

Example:

```text
applications/catalogue/base/
├── deployment.yaml
├── service.yaml
├── serviceaccount.yaml
├── configmap.yaml
├── hpa.yaml
├── pdb.yaml
├── networkpolicy.yaml
└── kustomization.yaml
```

The base defines common application behavior.

---

# 33.11 Kustomization Base

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
  - serviceaccount.yaml
  - configmap.yaml
  - hpa.yaml
  - pdb.yaml
  - networkpolicy.yaml
```

---

# 33.12 Production Overlay

```text
applications/catalogue/overlays/prod/
├── kustomization.yaml
├── replica-patch.yaml
├── resources-patch.yaml
└── config-patch.yaml
```

---

# 33.13 Production Kustomization

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: roboshop

resources:
  - ../../base

patches:
  - path: replica-patch.yaml
  - path: resources-patch.yaml
```

---

# 33.14 Production Replica Patch

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalogue
spec:
  replicas: 3
```

---

# 33.15 Production Resources

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalogue
spec:
  template:
    spec:
      containers:
        - name: catalogue
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: "1"
              memory: 512Mi
```

---

# 33.16 Production Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalogue
  labels:
    app.kubernetes.io/name: catalogue
    app.kubernetes.io/part-of: roboshop
spec:
  replicas: 3

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

  selector:
    matchLabels:
      app.kubernetes.io/name: catalogue

  template:
    metadata:
      labels:
        app.kubernetes.io/name: catalogue
        app.kubernetes.io/part-of: roboshop

    spec:
      serviceAccountName: catalogue

      securityContext:
        seccompProfile:
          type: RuntimeDefault

      containers:
        - name: catalogue
          image: <account>.dkr.ecr.<region>.amazonaws.com/roboshop/catalogue@sha256:<digest>

          ports:
            - name: http
              containerPort: 8080

          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: "1"
              memory: 512Mi

          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
            readOnlyRootFilesystem: true

          readinessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 3
            failureThreshold: 3

          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 20
            timeoutSeconds: 3
            failureThreshold: 3
```

---

# 33.17 Why Image Digest

Production should prefer:

```text
image@sha256:<digest>
```

over:

```text
image:latest
```

Digest pinning gives:

- immutable reference
- reproducibility
- easier incident investigation
- reduced tag mutation risk

---

# 33.18 Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue
  labels:
    app.kubernetes.io/name: catalogue
spec:
  type: ClusterIP

  selector:
    app.kubernetes.io/name: catalogue

  ports:
    - name: http
      port: 8080
      targetPort: http
      protocol: TCP
```

---

# 33.19 ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: catalogue
  namespace: roboshop
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<account>:role/roboshop-prod-catalogue
```

The actual ARN should be generated and managed through the infrastructure/security process.

Do not place AWS access keys in Kubernetes manifests.

---

# 33.20 ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: catalogue-config
  namespace: roboshop
data:
  LOG_LEVEL: "INFO"
  PORT: "8080"
  ENVIRONMENT: "production"
```

Non-sensitive configuration belongs here.

---

# 33.21 Secrets

Do not commit:

```yaml
stringData:
  DB_PASSWORD: real-password
```

Instead integrate with:

```text
AWS Secrets Manager
External Secrets mechanism
or another approved secret-management system
```

The GitOps repository should contain references/configuration, not secret values.

---

# 33.22 PodDisruptionBudget

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: catalogue
  namespace: roboshop
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app.kubernetes.io/name: catalogue
```

This protects availability during voluntary disruptions.

---

# 33.23 HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: catalogue
  namespace: roboshop
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: catalogue

  minReplicas: 3
  maxReplicas: 10

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
    scaleDown:
      stabilizationWindowSeconds: 300

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

---

# 33.24 NetworkPolicy

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: catalogue

  policyTypes:
    - Ingress
    - Egress

  ingress:
    - from:
        - podSelector:
            matchLabels:
              app.kubernetes.io/name: frontend
      ports:
        - protocol: TCP
          port: 8080

  egress:
    - to:
        - podSelector:
            matchLabels:
              app.kubernetes.io/name: mongodb
      ports:
        - protocol: TCP
          port: 27017
```

In production, DNS egress and required dependencies must also be allowed explicitly when default-deny policies are used.

---

# 33.25 Default Deny

A namespace-level default-deny policy can be used:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

Additional allow policies must then be carefully defined.

---

# 33.26 Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
  labels:
    environment: prod
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

---

# 33.27 Argo CD Project

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: roboshop
  namespace: argocd
spec:
  description: RoboShop production GitOps project

  sourceRepos:
    - https://github.com/<org>/roboshop-gitops.git

  destinations:
    - namespace: roboshop
      server: https://kubernetes.default.svc

  clusterResourceWhitelist:
    - group: ""
      kind: Namespace

  namespaceResourceWhitelist:
    - group: "*"
      kind: "*"

  roles:
    - name: readonly
      description: Read-only access
      policies:
        - p, proj:roboshop:readonly, applications, get, roboshop/*, allow
```

Production projects should be more restrictive than broad examples.

---

# 33.28 Argo CD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: catalogue-prod
  namespace: argocd

  annotations:
    notifications.argoproj.io/subscribe.on-sync-failed.slack: platform-alerts

spec:
  project: roboshop

  source:
    repoURL: https://github.com/<org>/roboshop-gitops.git
    targetRevision: main
    path: applications/catalogue/overlays/prod

  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop

  syncPolicy:
    automated:
      prune: false
      selfHeal: true

    syncOptions:
      - CreateNamespace=false

  revisionHistoryLimit: 10
```

---

# 33.29 Why prune false

Production pruning can be dangerous.

With:

```yaml
prune: false
```

Argo CD does not automatically delete resources removed from Git.

A controlled organization may enable pruning after mature governance and testing.

---

# 33.30 Why Self-Heal

```yaml
selfHeal: true
```

means Argo CD can correct drift.

Example:

```text
Git:
replicas=3

Live:
replicas=5
```

Argo CD detects the difference and reconciles toward Git.

---

# 33.31 ApplicationSet

For multiple environments/clusters, ApplicationSet can generate Applications.

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop
  namespace: argocd

spec:
  generators:
    - list:
        elements:
          - name: dev
            cluster: https://dev.example
            path: environments/dev

          - name: qa
            cluster: https://qa.example
            path: environments/qa

          - name: prod
            cluster: https://prod.example
            path: environments/prod

  template:
    metadata:
      name: 'roboshop-{{name}}'

    spec:
      project: roboshop

      source:
        repoURL: https://github.com/<org>/roboshop-gitops.git
        targetRevision: main
        path: '{{path}}'

      destination:
        server: '{{cluster}}'
        namespace: roboshop
```

---

# 33.32 Multi-Cluster GitOps

Production architecture:

```text
                 Git
                  |
                  v
              Argo CD
              Control Plane
              /    |    \
             /     |     \
          Dev      QA     Prod
          EKS      EKS     EKS
```

Argo CD centrally reconciles multiple clusters.

---

# 33.33 Cluster Separation

A production design can use:

```text
dev EKS
qa EKS
prod EKS
```

or separate production clusters by region/business boundary.

The GitOps repository remains the source of desired state.

---

# 33.34 Environment Directory

Example:

```text
environments/
├── dev/
│   ├── kustomization.yaml
│   └── apps.yaml
├── qa/
│   ├── kustomization.yaml
│   └── apps.yaml
└── prod/
    ├── kustomization.yaml
    └── apps.yaml
```

---

# 33.35 Production Environment Kustomization

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../applications/frontend/overlays/prod
  - ../../applications/catalogue/overlays/prod
  - ../../applications/cart/overlays/prod
  - ../../applications/user/overlays/prod
  - ../../applications/payment/overlays/prod
  - ../../applications/shipping/overlays/prod
  - ../../applications/checkout/overlays/prod
```

---

# 33.36 Complete RoboShop Services

A representative platform can include:

```text
frontend
catalogue
cart
user
payment
shipping
checkout
```

Additional infrastructure dependencies may include:

```text
MongoDB
Redis
RabbitMQ
```

Database ownership and deployment strategy should be determined separately.

---

# 33.37 Application Ownership

Example:

```text
frontend  → web team
catalogue → commerce team
cart      → commerce team
user      → identity team
payment   → payments team
shipping  → fulfillment team
checkout  → commerce team
```

Labels should make ownership visible.

---

# 33.38 Standard Labels

Use:

```yaml
labels:
  app.kubernetes.io/name: catalogue
  app.kubernetes.io/part-of: roboshop
  app.kubernetes.io/component: backend
  app.kubernetes.io/managed-by: argocd
  app.kubernetes.io/version: "1.12.4"
  environment: prod
  team: commerce
```

These labels improve:

- querying
- dashboards
- ownership
- troubleshooting
- cost analysis

---

# 33.39 Image Update Flow

Suppose CI builds:

```text
catalogue:1.12.4
```

CI:

```text
build
→ test
→ scan
→ push ECR
→ obtain digest
→ update GitOps
```

Example:

```text
catalogue@sha256:abc123...
```

---

# 33.40 GitOps Commit

Example:

```text
chore(catalogue): deploy 1.12.4 to production
```

PR includes:

```text
old digest
new digest
CI run
security scan
release notes
rollback version
```

---

# 33.41 Pull Request Flow

```text
CI
 |
 v
Create GitOps PR
 |
 v
Validation
 |
 +--> YAML lint
 +--> Kustomize build
 +--> policy checks
 +--> security checks
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

---

# 33.42 GitOps Validation

Useful local commands:

```bash
kubectl kustomize applications/catalogue/overlays/prod
```

Validate rendered output:

```bash
kubectl apply \
  --dry-run=server \
  -f <rendered-manifest.yaml>
```

Where server-side validation is appropriate and access is available.

---

# 33.43 Kustomize Build

```bash
kustomize build \
  applications/catalogue/overlays/prod
```

Save output:

```bash
kustomize build \
  applications/catalogue/overlays/prod \
  > /tmp/catalogue-prod.yaml
```

Review before deployment.

---

# 33.44 Diff

```bash
kubectl diff \
  -f /tmp/catalogue-prod.yaml
```

This helps identify changes before applying.

In GitOps, Argo CD should ultimately perform reconciliation.

---

# 33.45 Repository Validation Script

Example:

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Validating repository..."

find . \
  -name 'kustomization.yaml' \
  -print0 |
while IFS= read -r -d '' file; do
  dir="$(dirname "$file")"
  echo "Rendering $dir"
  kustomize build "$dir" >/dev/null
done

echo "Validation completed."
```

---

# 33.46 GitOps Policy

Every production deployment should answer:

```text
Who requested it?
What changed?
Which image?
Which digest?
Which commit?
Which CI run?
Who approved it?
When was it deployed?
What is rollback version?
```

Git provides much of this audit trail.

---

# 33.47 Rollback Through Git

Preferred process:

```text
bad commit
   |
   v
git revert
   |
   v
PR
   |
   v
approval
   |
   v
merge
   |
   v
Argo CD
   |
   v
previous desired state
```

This is safer than manually changing production and forgetting the desired state.

---

# 33.48 Emergency Rollback

If customer impact is severe:

```bash
kubectl rollout undo deployment/catalogue -n roboshop
```

Then immediately reconcile the GitOps repository.

The emergency state should not remain unmanaged.

---

# 33.49 Argo CD Drift

Example:

```text
Git replicas: 3
Live replicas: 5
```

Argo CD:

```text
OutOfSync
```

Possible causes:

```text
manual kubectl edit
HPA
operator
controller-generated state
```

Not every live-field difference should be interpreted as unauthorized drift.

---

# 33.50 Ignore Differences Carefully

Argo CD can ignore selected fields.

Do not broadly ignore entire resources.

Ignoring too much can hide real drift.

---

# 33.51 Monitoring in GitOps

Monitoring configuration can live in:

```text
monitoring/
├── prometheus-rules/
├── alertmanager/
└── dashboards/
```

Example:

```text
Git
 |
 v
Argo CD
 |
 +--> application
 +--> PrometheusRule
 +--> Alertmanager config
```

---

# 33.52 PrometheusRule

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: catalogue-alerts
  namespace: monitoring
  labels:
    release: kube-prometheus-stack
spec:
  groups:
    - name: catalogue.rules

      rules:
        - alert: CatalogueHighErrorRate
          expr: |
            (
              sum(rate(http_requests_total{
                namespace="roboshop",
                app="catalogue",
                status=~"5.."
              }[5m]))
              /
              sum(rate(http_requests_total{
                namespace="roboshop",
                app="catalogue"
              }[5m]))
            ) > 0.05

          for: 10m

          labels:
            severity: critical
            team: commerce
            environment: prod

          annotations:
            summary: Catalogue error rate is high
            description: Catalogue 5xx rate has exceeded 5% for 10 minutes.
            runbook_url: https://runbooks.example.com/roboshop/catalogue/high-error-rate
```

Metric names must match the application's actual instrumentation.

---

# 33.53 Alert Ownership

Every actionable alert should contain:

```text
severity
team
environment
service
runbook
```

Example:

```yaml
labels:
  severity: critical
  team: commerce
  environment: prod
  service: catalogue
```

---

# 33.54 Alertmanager GitOps

Example configuration:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: alertmanager-main
  namespace: monitoring
type: Opaque
stringData:
  alertmanager.yaml: |
    global:
      resolve_timeout: 5m

    route:
      receiver: default
      group_by:
        - alertname
        - cluster
        - namespace
        - service

      routes:
        - matchers:
            - severity="critical"
            - environment="prod"
          receiver: production-critical

        - matchers:
            - severity="warning"
            - environment="prod"
          receiver: production-warning

    receivers:
      - name: default

      - name: production-critical

      - name: production-warning
```

Actual webhook URLs and credentials must be supplied securely rather than committed as plaintext.

---

# 33.55 Secret Management

The above example illustrates structure only.

A production implementation should use a secret-management mechanism such as:

```text
AWS Secrets Manager
External Secrets Operator
sealed/encrypted secret workflow
```

depending on organizational standards.

---

# 33.56 Argo CD Notifications

Argo CD can notify on:

```text
sync succeeded
sync failed
health degraded
application out of sync
```

Example annotation:

```yaml
metadata:
  annotations:
    notifications.argoproj.io/subscribe.on-sync-failed.slack: platform-alerts
```

---

# 33.57 Production Notifications

Use notifications for:

```text
production sync failure
deployment failure
health degradation
```

Avoid sending every normal reconciliation event to paging systems.

---

# 33.58 Repository Security Scanning

Recommended checks:

```text
YAML validation
Kubernetes schema validation
secret scanning
policy validation
container image policy
RBAC review
NetworkPolicy validation
```

Tools can include:

```text
Trivy
Conftest / OPA
Kyverno
kubeconform
gitleaks
```

The exact tooling should match the approved platform.

---

# 33.59 Secret Scanning

Run secret scanning before merge.

Examples of things to detect:

```text
AWS access keys
private keys
passwords
tokens
webhooks
database credentials
```

Never assume `.gitignore` is enough after a secret has already been committed.

---

# 33.60 Secret Rotation

If a secret was committed:

```text
1. Assume compromised.
2. Revoke.
3. Rotate.
4. Remove from Git history if required.
5. Check downstream exposure.
6. Review access logs.
```

Git deletion alone does not revoke a credential.

---

# 33.61 Production Branch Protection

Require:

```text
pull request
at least one or more reviewers
successful validation
security checks
no direct push
```

For highly critical repositories, require separate production approval.

---

# 33.62 Separation of Duties

A mature model:

```text
Developer
  |
  v
CI
  |
  v
GitOps PR
  |
  v
Reviewer
  |
  v
Merge
  |
  v
Argo CD
```

Developers do not need direct production write access.

---

# 33.63 Argo CD RBAC

Separate:

```text
read-only
developer
operator
administrator
```

Production access should be least privilege.

---

# 33.64 Application Ownership Metadata

Example:

```yaml
metadata:
  labels:
    team: payments
    cost-center: engineering
    environment: prod
```

This supports operations and chargeback/showback.

---

# 33.65 Complete Repository Example

```text
roboshop-gitops/
│
├── README.md
├── CODEOWNERS
├── .gitignore
│
├── argocd/
│   ├── project-roboshop.yaml
│   ├── applicationset.yaml
│   └── applications/
│       ├── roboshop-dev.yaml
│       ├── roboshop-qa.yaml
│       └── roboshop-prod.yaml
│
├── applications/
│   ├── frontend/
│   │   ├── base/
│   │   │   ├── deployment.yaml
│   │   │   ├── service.yaml
│   │   │   ├── serviceaccount.yaml
│   │   │   ├── hpa.yaml
│   │   │   ├── pdb.yaml
│   │   │   ├── networkpolicy.yaml
│   │   │   └── kustomization.yaml
│   │   └── overlays/
│   │       ├── dev/
│   │       ├── qa/
│   │       └── prod/
│   │
│   ├── catalogue/
│   ├── cart/
│   ├── user/
│   ├── payment/
│   ├── shipping/
│   └── checkout/
│
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
├── platform/
│   ├── namespaces/
│   ├── ingress/
│   ├── rbac/
│   ├── network-policies/
│   └── resource-policies/
│
├── monitoring/
│   ├── prometheus-rules/
│   ├── alertmanager/
│   └── dashboards/
│
├── policies/
│   ├── pod-security/
│   └── admission/
│
└── scripts/
    ├── validate.sh
    └── diff.sh
```

---

# 33.66 README

The repository README should document:

```text
purpose
architecture
repository structure
deployment flow
environment strategy
Argo CD access
application ownership
promotion process
rollback
emergency procedure
security
validation
```

---

# 33.67 Promotion Strategy

A controlled promotion:

```text
Dev
 |
 v
QA
 |
 v
Production
```

CI can update dev first.

After validation:

```text
QA promotion
```

After approval:

```text
Production promotion
```

---

# 33.68 Immutable Artifact Promotion

Prefer:

```text
same image digest
```

across environments.

Example:

```text
dev  → sha256:abc
qa   → sha256:abc
prod → sha256:abc
```

rather than rebuilding different binaries for every environment.

---

# 33.69 Why This Matters

If the image is rebuilt between environments:

```text
dev binary != qa binary
qa binary  != prod binary
```

This weakens confidence.

Promotion should move the tested artifact.

---

# 33.70 Environment Configuration

Environment-specific values may differ:

```text
replicas
resources
URLs
feature flags
autoscaling
logging level
external endpoints
```

But application code should remain identical.

---

# 33.71 Configuration Anti-Pattern

Avoid:

```yaml
image: catalogue:latest
```

and:

```yaml
env:
  DB_PASSWORD: "production-password"
```

and:

```yaml
command: ["curl", "...", "|", "sh"]
```

inside production manifests.

---

# 33.72 Production Manifest Review

Review:

```text
image
digest
replicas
resources
probes
securityContext
serviceAccount
network policy
PDB
HPA
labels
namespace
```

---

# 33.73 Probes

A production application should distinguish:

```text
startup
readiness
liveness
```

Readiness answers:

```text
Can this pod receive traffic?
```

Liveness answers:

```text
Should this container be restarted?
```

Startup protects slow-starting applications.

---

# 33.74 Deployment Strategy

Default:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

This supports controlled rollout.

Exact values depend on application startup time and capacity.

---

# 33.75 Topology Spread

Production workloads should consider spreading replicas across nodes/AZs.

Example:

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: ScheduleAnyway
    labelSelector:
      matchLabels:
        app.kubernetes.io/name: catalogue
```

For critical services, stronger scheduling constraints may be appropriate.

---

# 33.76 Anti-Affinity

Another option:

```yaml
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          topologyKey: kubernetes.io/hostname
          labelSelector:
            matchLabels:
              app.kubernetes.io/name: catalogue
```

This reduces concentration on one node.

---

# 33.77 Resource Quotas

Namespace-level resource controls can prevent runaway consumption.

Example:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: roboshop-quota
  namespace: roboshop
spec:
  requests.cpu: "20"
  requests.memory: 40Gi
  limits.cpu: "40"
  limits.memory: 80Gi
  pods: "200"
```

Tune values from actual capacity planning.

---

# 33.78 LimitRange

Example:

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: roboshop-defaults
  namespace: roboshop
spec:
  limits:
    - type: Container
      default:
        cpu: "1"
        memory: 512Mi
      defaultRequest:
        cpu: 100m
        memory: 128Mi
```

Defaults should be deliberate rather than arbitrary.

---

# 33.79 Production Ingress

The GitOps repository can manage ALB Ingress.

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: "443"
spec:
  ingressClassName: alb

  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

Certificate and WAF configuration should be managed according to the approved AWS design.

---

# 33.80 GitOps and ALB

Flow:

```text
Git
 |
 v
Ingress manifest
 |
 v
Argo CD
 |
 v
Kubernetes
 |
 v
AWS Load Balancer Controller
 |
 v
AWS ALB
```

Argo CD does not directly create the ALB.

The Kubernetes controller reconciles the Ingress into AWS resources.

---

# 33.81 Monitoring Ownership

Monitoring configuration should have ownership.

Example:

```yaml
labels:
  team: sre
  environment: prod
```

Alert ownership should be explicit.

---

# 33.82 Alert Runbook URL

Example:

```yaml
annotations:
  runbook_url: https://runbooks.example.com/catalogue/high-error-rate
```

The runbook should tell responders:

```text
symptom
commands
metrics
likely causes
safe remediation
rollback
escalation
```

---

# 33.83 GitOps Incident Flow

```text
Alert
 |
 v
Engineer
 |
 v
Grafana / Prometheus
 |
 v
Argo CD
 |
 v
Recent commit
 |
 v
Git diff
 |
 v
Identify change
 |
 v
Rollback/revert
 |
 v
Argo CD sync
 |
 v
Validation
```

This makes Git history an important incident-management tool.

---

# 33.84 Deployment Audit

For a deployment, record:

```text
Git commit
image digest
Argo CD revision
deployment timestamp
operator/automation
environment
application version
```

---

# 33.85 Rollback Audit

A rollback should produce:

```text
rollback commit
reason
incident ID
previous digest
new digest
approval
validation result
```

---

# 33.86 GitOps Drift Detection

Drift can be caused by:

```text
manual kubectl
controller
operator
HPA
admission mutation
```

Use Argo CD diff and Kubernetes ownership metadata to determine whether the difference is expected.

---

# 33.87 Manual kubectl Policy

Normal production changes:

```text
Git → Argo CD
```

Emergency:

```text
kubectl → temporary mitigation
```

After emergency:

```text
temporary change → Git
```

This preserves the desired-state model.

---

# 33.88 Repository Backup

Git itself should be protected through:

```text
remote redundancy
repository backups
organization-level recovery
branch protection
access recovery
```

The GitOps repository is a critical recovery asset.

---

# 33.89 Disaster Recovery Dependency

If the production EKS cluster is lost:

```text
Terraform
   |
   v
new EKS
   |
   v
Argo CD
   |
   v
GitOps repository
   |
   v
applications restored
```

This is why infrastructure and desired state must be version-controlled independently.

---

# 33.90 Bootstrap Sequence

A new production cluster can be bootstrapped:

```text
1. Create AWS infrastructure
2. Create EKS
3. Configure IAM
4. Install required controllers
5. Install Argo CD
6. Configure repository access
7. Create AppProject
8. Create Applications
9. Sync platform
10. Sync applications
11. Validate
```

---

# 33.91 Argo CD Bootstrap

Example:

```bash
kubectl create namespace argocd

kubectl apply \
  -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

In production, pin and approve a specific Argo CD release rather than consuming an uncontrolled moving target.

---

# 33.92 Repository Registration

Argo CD requires access to the Git repository.

Use:

```text
SSH deploy key
or
managed repository credential
```

Use least privilege.

---

# 33.93 No Long-Lived Developer Credentials

Do not embed:

```text
GitHub personal access token
AWS access key
database password
Slack webhook
```

in manifests.

Use secret management.

---

# 33.94 GitOps Repository Review Checklist

```text
[ ] No plaintext secrets
[ ] Images pinned
[ ] Production reviewers configured
[ ] CODEOWNERS configured
[ ] Kustomize builds
[ ] Policies pass
[ ] Network policies reviewed
[ ] Resources configured
[ ] Probes configured
[ ] PDB configured
[ ] HPA configured
[ ] Argo CD project restricted
[ ] Rollback path documented
[ ] Monitoring configured
```

---

# 33.95 Production Promotion Checklist

```text
[ ] CI passed
[ ] tests passed
[ ] SonarQube passed
[ ] Trivy passed
[ ] Veracode passed
[ ] image pushed to ECR
[ ] image digest recorded
[ ] GitOps PR opened
[ ] manifest validation passed
[ ] approval obtained
[ ] merge completed
[ ] Argo CD sync successful
[ ] rollout successful
[ ] health validated
[ ] business transaction validated
```

---

# 33.96 Production Rollback Checklist

```text
[ ] confirm impact
[ ] identify bad commit
[ ] identify last known-good digest
[ ] assess rollback safety
[ ] revert GitOps change
[ ] merge/approve
[ ] Argo CD sync
[ ] monitor rollout
[ ] validate ALB
[ ] validate application
[ ] validate metrics
[ ] communicate recovery
```

---

# 33.97 Failure Scenario — Bad Manifest

Symptom:

```text
Argo CD sync failed
```

Investigation:

```bash
kubectl get applications -n argocd
kubectl describe application catalogue-prod -n argocd
```

Root cause:

```text
invalid field in manifest
```

Fix:

```text
revert/fix Git commit
```

Prevention:

```text
schema validation
Kustomize build
CI validation
```

---

# 33.98 Failure Scenario — Wrong Image

Symptom:

```text
ImagePullBackOff
```

Root cause:

```text
incorrect ECR repository/tag/digest
```

Fix:

```text
correct GitOps image reference
```

Prevention:

```text
automated image verification
digest pinning
```

---

# 33.99 Failure Scenario — Manual Drift

Symptom:

```text
Argo CD OutOfSync
```

Root cause:

```text
manual kubectl scale
```

Fix:

```text
restore Git state
```

Prevention:

```text
RBAC
GitOps discipline
drift alerts
```

---

# 33.100 Failure Scenario — Argo CD Cannot Sync

Possible causes:

```text
repository unavailable
credentials invalid
Kubernetes RBAC
missing CRD
invalid manifest
API server unavailable
```

Investigation should proceed from:

```text
Git
→ Argo CD
→ Kubernetes API
→ resource
```

---

# 33.101 GitOps Repository Health

Monitor:

```text
Argo CD sync failures
OutOfSync applications
degraded applications
repository access
manifest generation failures
```

---

# 33.102 Production Repository Metrics

Useful metrics include:

```text
deployment frequency
deployment lead time
sync failures
rollback frequency
change failure rate
mean time to recovery
```

GitOps enables reliable deployment measurement.

---

# 33.103 GitOps and DORA

Git history plus CI/CD data can support:

```text
deployment frequency
lead time for changes
change failure rate
MTTR
```

These should be measured from actual system events rather than guessed.

---

# 33.104 Complete Deployment Lifecycle

```text
Developer
   |
   v
Application Git
   |
   v
CI
   |
   +--> Maven/npm/pip
   +--> tests
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
Docker image
   |
   v
ECR
   |
   v
GitOps PR
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
   |
   v
Prometheus/Grafana/ELK
```

---

# 33.105 Senior DevOps Explanation

If asked:

> Why use a separate GitOps repository?

Answer:

```text
I separate application source from deployment desired state.

The application repository contains source code and build configuration.
The GitOps repository contains environment-specific Kubernetes desired state.

CI builds, tests, scans and publishes an immutable image to ECR.
It then updates the GitOps repository with the approved image digest.
Argo CD watches that repository and reconciles the desired state into EKS.

This gives us auditability, review, reproducibility, controlled promotion and a clear rollback mechanism.
```

---

# 33.106 Senior Question — Why Not kubectl from CI?

Strong answer:

```text
Direct kubectl deployment from CI tightly couples the build system to cluster write access.

With GitOps, CI only needs to publish the artifact and update desired state.
Argo CD owns the deployment operation.

This reduces direct production credentials in CI, improves auditability and gives us a clear desired-state model.
```

---

# 33.107 Senior Question — How Do You Roll Back?

Answer:

```text
The preferred rollback is a Git revert to the previous known-good desired state.

Argo CD reconciles that state into Kubernetes.

For severe incidents, I can use a temporary kubectl rollback to reduce customer impact, but I immediately reconcile Git so the emergency state does not become unmanaged drift.
```

---

# 33.108 Senior Question — What If Someone Changes Production Manually?

Answer:

```text
Argo CD detects the difference between Git and the live cluster.

If self-healing is enabled and the difference is not an expected controller-managed field, Argo CD can restore the Git state.

I also investigate why the manual change was possible and tighten RBAC if necessary.
```

---

# 33.109 Senior Question — How Do You Protect Production?

Answer:

```text
I protect the GitOps main branch with reviews and status checks, restrict Argo CD project destinations and repositories, use least-privilege RBAC, avoid plaintext secrets, pin production images by digest, validate manifests in CI, and separate emergency access from normal deployment access.
```

---

# 33.110 Senior Question — How Do You Promote the Same Artifact?

Answer:

```text
I promote the same immutable ECR image digest from dev to QA to production.

That means the artifact tested in lower environments is the artifact deployed to production.

Environment differences are represented by configuration, not by rebuilding the application.
```

---

# 33.111 Senior Question — How Does GitOps Help DR?

Answer:

```text
The GitOps repository contains the desired Kubernetes state independently of the cluster.

If an EKS cluster is lost, Terraform can recreate infrastructure, Argo CD can be installed and pointed to the repository, and the workloads can be reconciled into the new cluster.

This reduces dependence on manual reconstruction.
```

---

# 33.112 Production GitOps Anti-Patterns

Avoid:

```text
latest tags
plaintext secrets
direct production kubectl
one giant manifest
no reviewers
unrestricted Argo CD project
cluster-admin everywhere
manual image changes
long-lived tokens
environment drift
unvalidated manifests
```

---

# 33.113 GitOps Best Practices

Use:

```text
immutable images
Git review
CODEOWNERS
least privilege
automated validation
environment overlays
clear ownership
PDB
HPA
probes
NetworkPolicy
resource requests
monitoring
runbooks
rollback procedures
```

---

# 33.114 Final Repository Architecture

```text
                    ┌─────────────────────┐
                    │ Application Git     │
                    └──────────┬──────────┘
                               │
                               v
                    ┌─────────────────────┐
                    │ CI / Security       │
                    │ Build/Test/Scan     │
                    └──────────┬──────────┘
                               │
                               v
                    ┌─────────────────────┐
                    │ AWS ECR             │
                    │ Immutable Image      │
                    └──────────┬──────────┘
                               │
                               v
                    ┌─────────────────────┐
                    │ GitOps Repository   │
                    │ Desired State       │
                    └──────────┬──────────┘
                               │
                               v
                    ┌─────────────────────┐
                    │ Argo CD             │
                    │ Reconciliation      │
                    └──────┬──────┬───────┘
                           │      │
                    ┌──────┘      └──────┐
                    v                     v
               Dev EKS                 Prod EKS
                    │                     │
                    v                     v
              RoboShop Apps          RoboShop Apps
                    │                     │
                    └──────────┬──────────┘
                               v
                    Prometheus/Grafana/ELK
```

---

# 33.115 Final Production Checklist

```text
Repository
[ ] structure documented
[ ] CODEOWNERS
[ ] protected main
[ ] no secrets

Applications
[ ] immutable images
[ ] probes
[ ] resources
[ ] security context
[ ] ServiceAccount
[ ] PDB
[ ] HPA
[ ] NetworkPolicy

Argo CD
[ ] AppProject
[ ] restricted destinations
[ ] repository allow-list
[ ] environment applications
[ ] sync policy
[ ] notifications

Security
[ ] secret scanning
[ ] RBAC
[ ] least privilege
[ ] image scanning
[ ] policy validation

Operations
[ ] alerts
[ ] dashboards
[ ] runbooks
[ ] rollback
[ ] DR bootstrap

Delivery
[ ] CI
[ ] artifact promotion
[ ] GitOps PR
[ ] approval
[ ] Argo CD reconciliation
```

---

# 33.116 Final Takeaway

The production GitOps repository is not simply a directory containing YAML files.

It is the **declarative control plane for application delivery**.

The complete model is:

```text
Code
  ↓
CI
  ↓
Security
  ↓
Immutable Artifact
  ↓
GitOps Desired State
  ↓
Review
  ↓
Argo CD
  ↓
EKS
  ↓
Observability
  ↓
Operations
```

The most important production principle is:

```text
Build once.
Scan once.
Promote the same immutable artifact.
Declare desired state in Git.
Let Argo CD reconcile it.
Observe the result.
Rollback through version-controlled desired state.
```

This creates a deployment system that is auditable, reproducible, secure, operationally recoverable and suitable for a production EKS environment.
