# 19-GitOps-Production-Kubernetes-YAMLs.md

# GitOps Production Kubernetes YAMLs

## 1. Purpose

This file is the practical YAML implementation section for the GitOps with Argo CD notes.

The objective is not to provide toy manifests.

The examples are designed around a production-style AWS EKS environment using:

```text
AWS
EKS
ECR
Kubernetes
Helm
AWS Load Balancer Controller
ALB Ingress
Argo CD
Argo Rollouts where required
Prometheus
Grafana
ELK
Jenkins / GitHub Actions
Terraform
```

The examples are intentionally adaptable.

Before production use, validate:

```text
Kubernetes version
Argo CD version
AWS Load Balancer Controller version
Helm chart version
EKS networking
IAM
security policies
organizational naming
```

---

# 2. Production GitOps Repository

A practical repository can be organized as:

```text
gitops-repo/
├── README.md
│
├── argocd/
│   ├── projects/
│   ├── applications/
│   ├── applicationsets/
│   ├── repositories/
│   ├── clusters/
│   └── rbac/
│
├── environments/
│   ├── dev/
│   │   ├── values/
│   │   └── patches/
│   ├── qa/
│   │   ├── values/
│   │   └── patches/
│   └── prod/
│       ├── values/
│       └── patches/
│
├── applications/
│   ├── roboshop/
│   │   ├── cart/
│   │   ├── catalog/
│   │   ├── inventory/
│   │   ├── notification/
│   │   ├── orders/
│   │   ├── payment/
│   │   └── user/
│   │
│   └── platform/
│       ├── ingress/
│       ├── monitoring/
│       └── autoscaling/
│
├── helm/
│   └── roboshop/
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── values-dev.yaml
│       ├── values-qa.yaml
│       ├── values-prod.yaml
│       └── templates/
│
├── manifests/
│   ├── namespaces/
│   ├── network-policies/
│   └── policies/
│
└── platform/
    ├── aws-load-balancer-controller/
    ├── prometheus/
    └── grafana/
```

---

# 3. Repository Design Principles

A production repository should provide:

```text
clear ownership
environment separation
reviewability
rollback capability
least privilege
consistent naming
repeatable deployment
```

Avoid:

```text
random YAML files
manual production edits
environment values mixed together
secrets committed in plaintext
```

---

# 4. Source of Truth

The desired application state should be represented by Git.

Example:

```yaml
replicas: 6
image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop-cart
  tag: "2.4.1"
```

Git stores the desired configuration.

Argo CD reconciles it into Kubernetes.

---

# 5. Environment Separation

A common model:

```text
dev
qa
prod
```

with progressively stricter controls.

Example:

```text
dev:
automatic sync

qa:
automatic sync + tests

prod:
controlled promotion + approval
```

The exact policy should follow organizational requirements.

---

# 6. Namespace Manifest

A production namespace can include labels used by:

```text
monitoring
security
network policies
cost allocation
```

Example:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
  labels:
    app.kubernetes.io/part-of: roboshop
    environment: prod
    team: platform
    kubernetes.io/metadata.name: roboshop
```

---

# 7. Namespace Best Practices

Use:

```text
one clear ownership model
labels
resource quotas where appropriate
network policies
RBAC
```

Avoid putting unrelated production applications into one uncontrolled namespace.

---

# 8. ResourceQuota

A production namespace can use a quota.

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: roboshop-compute
  namespace: roboshop
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    pods: "100"
```

Tune values from actual capacity planning.

---

# 9. LimitRange

A LimitRange can establish defaults.

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
        memory: "1Gi"
      defaultRequest:
        cpu: "100m"
        memory: "128Mi"
      max:
        cpu: "4"
        memory: "8Gi"
```

Do not use namespace defaults as a replacement for service-specific resource engineering.

---

# 10. Deployment

A production Deployment should normally define:

```text
replicas
selector
Pod template
image
resources
probes
security context
rolling update strategy
```

---

# 11. Production Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
  namespace: roboshop
  labels:
    app.kubernetes.io/name: cart
    app.kubernetes.io/part-of: roboshop
    app.kubernetes.io/component: backend
spec:
  replicas: 3

  revisionHistoryLimit: 5

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

  selector:
    matchLabels:
      app.kubernetes.io/name: cart

  template:
    metadata:
      labels:
        app.kubernetes.io/name: cart
        app.kubernetes.io/part-of: roboshop
        app.kubernetes.io/component: backend
    spec:
      serviceAccountName: cart

      terminationGracePeriodSeconds: 30

      securityContext:
        seccompProfile:
          type: RuntimeDefault

      containers:
        - name: cart
          image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop-cart@sha256:REPLACE_WITH_DIGEST

          imagePullPolicy: IfNotPresent

          ports:
            - name: http
              containerPort: 8080
              protocol: TCP

          resources:
            requests:
              cpu: 200m
              memory: 256Mi
            limits:
              cpu: "1"
              memory: 1Gi

          envFrom:
            - configMapRef:
                name: cart-config
            - secretRef:
                name: cart-secret

          readinessProbe:
            httpGet:
              path: /health/ready
              port: http
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 3
            successThreshold: 1

          livenessProbe:
            httpGet:
              path: /health/live
              port: http
            initialDelaySeconds: 30
            periodSeconds: 20
            timeoutSeconds: 2
            failureThreshold: 3

          startupProbe:
            httpGet:
              path: /health/startup
              port: http
            periodSeconds: 5
            timeoutSeconds: 2
            failureThreshold: 30

          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
```

---

# 12. Why Use Image Digests

Prefer:

```text
image@sha256:...
```

over mutable tags alone.

Digest-based deployment gives stronger immutability.

Example:

```text
cart:2.4.1
```

can theoretically point to a different image later.

A digest identifies a specific image artifact.

---

# 13. Image Promotion

A production pipeline can use:

```text
build
 |
 v
scan
 |
 v
ECR
 |
 v
dev
 |
 v
qa
 |
 v
prod
```

Promotion should preferably reference the same immutable image digest.

---

# 14. ServiceAccount

Production applications should use dedicated ServiceAccounts when they interact with Kubernetes or AWS.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cart
  namespace: roboshop
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/roboshop-cart
```

The ARN is an example and must be replaced.

---

# 15. IRSA / EKS Pod Identity

The exact AWS workload identity mechanism depends on the EKS architecture.

Common approaches include:

```text
EKS Pod Identity
IRSA
```

Use the organization's approved method.

Do not attach broad AWS permissions to every worker node.

---

# 16. Service

A production Service should expose only what is required.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cart
  namespace: roboshop
  labels:
    app.kubernetes.io/name: cart
    app.kubernetes.io/part-of: roboshop
spec:
  type: ClusterIP
  selector:
    app.kubernetes.io/name: cart
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP
```

---

# 17. Why ClusterIP

Internal microservices should generally use:

```text
ClusterIP
```

rather than:

```text
NodePort
```

or individual:

```text
LoadBalancer
```

for every service.

This keeps external exposure centralized.

---

# 18. ALB Ingress

The user's architecture uses AWS ALB rather than API Gateway.

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
    alb.ingress.kubernetes.io/healthcheck-path: /health/ready
    alb.ingress.kubernetes.io/success-codes: "200-399"
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
                name: cart
                port:
                  name: http
```

TLS certificate annotations and WAF configuration should be added according to the organization's AWS design.

---

# 19. ALB Ingress Notes

Important concepts:

```text
ingressClassName: alb
target-type: ip
```

With IP target mode:

```text
ALB -> Pod IP
```

This is common for EKS workloads using the AWS Load Balancer Controller.

---

# 20. TLS

Production ALB ingress should normally use HTTPS.

A common annotation is:

```yaml
alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:ap-south-1:123456789012:certificate/REPLACE
```

Never commit an incorrect or fake certificate ARN into production.

---

# 21. WAF

If required:

```yaml
alb.ingress.kubernetes.io/wafv2-acl-arn: arn:aws:wafv2:...
```

Use WAF according to security architecture.

---

# 22. Security Groups

ALB and EKS security groups should be designed so that:

```text
Internet
  |
  v
ALB
  |
  v
approved application path
```

is allowed while unnecessary direct Pod access is restricted.

---

# 23. ConfigMap

Non-sensitive configuration belongs in ConfigMaps.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cart-config
  namespace: roboshop
data:
  APP_ENV: "prod"
  LOG_LEVEL: "INFO"
  CART_PORT: "8080"
```

Do not put passwords or tokens here.

---

# 24. Secret

A Kubernetes Secret is not automatically a secure external secret-management solution.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cart-secret
  namespace: roboshop
type: Opaque
stringData:
  REDIS_HOST: "redis.roboshop.svc.cluster.local"
  REDIS_PASSWORD: "REPLACE_AT_DEPLOYMENT"
```

Do not commit real production credentials.

---

# 25. Production Secret Strategy

Prefer:

```text
AWS Secrets Manager
+
External Secrets Operator
```

or another approved secret-management system.

Git should contain:

```text
reference
configuration
SecretStore
ExternalSecret
```

rather than plaintext credentials.

---

# 26. ExternalSecret Example

If External Secrets Operator is used:

```yaml
apiVersion: external-secrets.io/v1
kind: ExternalSecret
metadata:
  name: cart-secret
  namespace: roboshop
spec:
  refreshInterval: 1h

  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore

  target:
    name: cart-secret
    creationPolicy: Owner

  data:
    - secretKey: REDIS_PASSWORD
      remoteRef:
        key: /roboshop/prod/cart
        property: redisPassword
```

Verify the CRD API version against the installed External Secrets Operator release.

---

# 27. Secret Flow

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

Argo CD manages the desired ExternalSecret configuration.

The secret value does not need to live in Git.

---

# 28. PodDisruptionBudget

A production service with multiple replicas can use a PDB.

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

This helps protect availability during voluntary disruptions.

---

# 29. HPA

Example:

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
  maxReplicas: 10

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 60
      selectPolicy: Max

    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 25
          periodSeconds: 60
      selectPolicy: Max

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70

    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 75
```

Tune from workload behavior.

---

# 30. HPA and GitOps

Git should define:

```text
minReplicas
maxReplicas
target
behavior
```

Kubernetes changes:

```text
current replicas
```

Argo CD must be configured thoughtfully so HPA-driven replica changes do not create noisy drift.

---

# 31. Ignore Differences for HPA

In some designs, Argo CD can ignore a Deployment replica field that HPA controls.

Example:

```yaml
spec:
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
```

Use this only when appropriate.

Do not hide meaningful drift accidentally.

---

# 32. NetworkPolicy

A production application can restrict traffic.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cart
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: cart

  policyTypes:
    - Ingress
    - Egress

  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: roboshop
      ports:
        - protocol: TCP
          port: 8080

  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: roboshop
      ports:
        - protocol: TCP
          port: 6379

    - to:
        - namespaceSelector: {}
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

This is an example and must be adapted to actual dependencies.

---

# 33. NetworkPolicy Caution

A restrictive policy can break:

```text
DNS
metrics scraping
service-to-service traffic
external APIs
```

Test in non-production first.

---

# 34. SecurityContext

Recommended baseline:

```yaml
securityContext:
  runAsNonRoot: true
  seccompProfile:
    type: RuntimeDefault
```

Container-level:

```yaml
securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
```

Use `readOnlyRootFilesystem: true` when the application supports it.

---

# 35. Pod-Level SecurityContext

Example:

```yaml
spec:
  securityContext:
    runAsNonRoot: true
    seccompProfile:
      type: RuntimeDefault
```

Do not set arbitrary UID values unless the image supports that UID.

---

# 36. Pod Anti-Affinity

For high availability:

```yaml
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          topologyKey: kubernetes.io/hostname
          labelSelector:
            matchLabels:
              app.kubernetes.io/name: cart
```

This prefers distributing replicas across nodes.

---

# 37. Topology Spread

Another approach:

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: ScheduleAnyway
    labelSelector:
      matchLabels:
        app.kubernetes.io/name: cart
```

Use zone-aware distribution where the cluster has multiple AZs.

---

# 38. Production Deployment Strategy

For a normal service:

```text
RollingUpdate
```

can be appropriate.

For advanced release control:

```text
Argo Rollouts
```

can provide:

```text
Canary
Blue/Green
analysis
promotion
abort
```

---

# 39. Argo CD Application

Production-style Application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-cart-prod
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
  labels:
    environment: prod
    app.kubernetes.io/part-of: roboshop
spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/example-org/gitops-repo.git
    targetRevision: main
    path: helm/roboshop/cart

    helm:
      releaseName: cart

      valueFiles:
        - values.yaml
        - values-prod.yaml

      values: |
        replicaCount: 3

  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true

    syncOptions:
      - CreateNamespace=false
      - PrunePropagationPolicy=foreground
      - PruneLast=true

    retry:
      limit: 5
      backoff:
        duration: 10s
        factor: 2
        maxDuration: 5m
```

---

# 40. Application Field Explanation

```text
apiVersion
```

Identifies the Argo CD API.

```text
kind: Application
```

Creates an Argo CD Application resource.

```text
metadata
```

Provides identity and lifecycle behavior.

```text
spec.project
```

Defines Argo CD governance.

---

# 41. Source

```yaml
source:
  repoURL:
  targetRevision:
  path:
```

means:

```text
Repository
   +
Revision
   +
Path
```

from which Argo CD obtains desired manifests.

---

# 42. Helm Source

```yaml
helm:
  releaseName: cart
  valueFiles:
    - values.yaml
    - values-prod.yaml
```

Argo CD renders Helm templates.

Argo CD does not require Helm to maintain a live Helm release in the traditional Helm-controller sense.

It renders the manifests and applies desired resources through Kubernetes/Argo CD reconciliation.

---

# 43. Destination

```yaml
destination:
  server: https://kubernetes.default.svc
  namespace: roboshop
```

means:

```text
target Kubernetes API
+
target namespace
```

For a remote cluster, the destination server is the registered cluster API endpoint.

---

# 44. Automated Sync

```yaml
automated:
  prune: true
  selfHeal: true
```

means Argo CD can automatically:

```text
apply desired changes
remove resources no longer desired
correct drift
```

Production teams should decide carefully whether automatic pruning is appropriate for every resource.

---

# 45. Prune

If a resource is removed from Git:

```text
Git no longer declares resource
```

with pruning enabled:

```text
Argo CD can delete the resource
```

This is powerful and dangerous.

---

# 46. Self-Heal

If someone changes:

```text
replicas
image
environment
```

manually in the cluster:

```text
Argo CD detects drift
```

with self-heal enabled:

```text
Argo CD restores desired state
```

---

# 47. Sync Options

Useful options can include:

```text
PruneLast=true
CreateNamespace=true
ApplyOutOfSyncOnly=true
ServerSideApply=true
```

Use only what the application needs.

---

# 48. Retry

Retry policies help recover from transient errors.

Example:

```yaml
retry:
  limit: 5
  backoff:
    duration: 10s
    factor: 2
    maxDuration: 5m
```

Retries should not mask permanent configuration failures.

---

# 49. Argo CD Project

Projects enforce governance.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: roboshop-prod
  namespace: argocd
spec:
  description: RoboShop production applications

  sourceRepos:
    - https://github.com/example-org/gitops-repo.git

  destinations:
    - namespace: roboshop
      server: https://kubernetes.default.svc

  clusterResourceWhitelist:
    - group: ""
      kind: Namespace

  namespaceResourceWhitelist:
    - group: apps
      kind: Deployment
    - group: ""
      kind: Service
    - group: ""
      kind: ConfigMap
    - group: ""
      kind: Secret
    - group: networking.k8s.io
      kind: Ingress
    - group: autoscaling
      kind: HorizontalPodAutoscaler
```

Restrict resources further based on actual requirements.

---

# 50. Why Projects Matter

Projects provide boundaries for:

```text
repositories
clusters
namespaces
resource kinds
RBAC
```

They are especially important in multi-team environments.

---

# 51. AppProject Security Model

Example:

```text
team A
  |
  +--> repo A
  +--> namespace A
  +--> cluster A

team B
  |
  +--> repo B
  +--> namespace B
  +--> cluster B
```

Avoid giving every Application access to every cluster.

---

# 52. ApplicationSet

A production ApplicationSet can generate Applications.

Example list generator:

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
            namespace: roboshop-dev
            revision: develop
          - environment: qa
            namespace: roboshop-qa
            revision: release
          - environment: prod
            namespace: roboshop-prod
            revision: main

  template:
    metadata:
      name: 'roboshop-{{environment}}'
      labels:
        environment: '{{environment}}'
    spec:
      project: roboshop-prod

      source:
        repoURL: https://github.com/example-org/gitops-repo.git
        targetRevision: '{{revision}}'
        path: helm/roboshop

        helm:
          valueFiles:
            - values.yaml
            - 'values-{{environment}}.yaml'

      destination:
        server: https://kubernetes.default.svc
        namespace: '{{namespace}}'

      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

In a real design, use separate AppProjects for dev/qa/prod when stronger isolation is required.

---

# 53. ApplicationSet Production Consideration

A single template can generate:

```text
dev
qa
prod
```

but governance should still enforce:

```text
different destinations
different permissions
different repositories/paths if required
different sync policies
```

---

# 54. Multi-Cluster ApplicationSet

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-multi-cluster
  namespace: argocd
spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            application: roboshop
            environment: prod

  template:
    metadata:
      name: 'roboshop-{{name}}'
      labels:
        environment: prod
        cluster: '{{name}}'

    spec:
      project: roboshop-prod

      source:
        repoURL: https://github.com/example-org/gitops-repo.git
        targetRevision: main
        path: helm/roboshop

        helm:
          valueFiles:
            - values.yaml
            - values-prod.yaml

      destination:
        server: '{{server}}'
        namespace: roboshop

      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

---

# 55. Cluster Generator

The cluster generator uses registered Argo CD clusters.

Cluster metadata can contain labels such as:

```text
environment=prod
region=ap-south-1
account=production
application=roboshop
```

The ApplicationSet selects matching clusters.

---

# 56. Central Argo CD Architecture

```text
                 Git
                  |
                  v
             Central Argo CD
              /     |      \
             /      |       \
            v       v        v
         EKS-DEV  EKS-QA  EKS-PROD
```

This allows centralized GitOps control.

---

# 57. App of Apps

Root Application:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: platform-root
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default

  source:
    repoURL: https://github.com/example-org/gitops-repo.git
    targetRevision: main
    path: argocd/applications

  destination:
    server: https://kubernetes.default.svc
    namespace: argocd

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

# 58. App of Apps Repository

```text
argocd/
└── applications/
    ├── cart.yaml
    ├── catalog.yaml
    ├── inventory.yaml
    ├── orders.yaml
    └── payment.yaml
```

The root Application manages the child Application resources.

---

# 59. App of Apps Workflow

```text
platform-root
      |
      v
argocd/applications/
      |
      +--> cart Application
      +--> catalog Application
      +--> inventory Application
      +--> orders Application
      +--> payment Application
```

---

# 60. App of Apps Use Cases

Useful for:

```text
platform bootstrap
application fleet management
environment bootstrap
cluster bootstrap
```

Be cautious with excessive nesting.

---

# 61. Finalizer

Example:

```yaml
finalizers:
  - resources-finalizer.argocd.argoproj.io
```

The finalizer can ensure Argo CD handles managed-resource cleanup when an Application is deleted.

Understand cleanup implications before enabling it broadly.

---

# 62. Helm Chart Structure

```text
cart/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-qa.yaml
├── values-prod.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── serviceaccount.yaml
    ├── ingress.yaml
    ├── hpa.yaml
    ├── pdb.yaml
    └── networkpolicy.yaml
```

---

# 63. Chart.yaml

```yaml
apiVersion: v2
name: cart
description: RoboShop cart service
type: application
version: 1.0.0
appVersion: "2.4.1"
```

Chart version and application version serve different purposes.

---

# 64. values.yaml

```yaml
replicaCount: 3

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop-cart
  digest: ""

service:
  port: 80
  targetPort: http

resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 1Gi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  cpuUtilization: 70
```

---

# 65. values-prod.yaml

```yaml
replicaCount: 5

image:
  digest: "sha256:REPLACE_WITH_APPROVED_DIGEST"

resources:
  requests:
    cpu: 300m
    memory: 512Mi
  limits:
    cpu: "2"
    memory: 2Gi

autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 20
  cpuUtilization: 65
```

---

# 66. Deployment Helm Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "cart.fullname" . }}
  labels:
    {{- include "cart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}

  selector:
    matchLabels:
      {{- include "cart.selectorLabels" . | nindent 6 }}

  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1

  template:
    metadata:
      labels:
        {{- include "cart.selectorLabels" . | nindent 8 }}

    spec:
      serviceAccountName: {{ include "cart.serviceAccountName" . }}

      securityContext:
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault

      containers:
        - name: cart
          image: "{{ .Values.image.repository }}@{{ .Values.image.digest }}"

          ports:
            - name: http
              containerPort: 8080

          resources:
            {{- toYaml .Values.resources | nindent 12 }}

          readinessProbe:
            httpGet:
              path: /health/ready
              port: http
            periodSeconds: 10
            timeoutSeconds: 2

          livenessProbe:
            httpGet:
              path: /health/live
              port: http
            periodSeconds: 20
            timeoutSeconds: 2
```

---

# 67. Helm Values Precedence

When multiple values sources are used, understand their precedence.

Typical concept:

```text
base values
+
environment values
+
explicit values/parameters
```

Do not assume precedence without checking the Argo CD/Helm behavior for the exact configuration.

---

# 68. Kustomize Alternative

Argo CD can render Kustomize.

Example:

```text
base/
├── deployment.yaml
├── service.yaml
└── kustomization.yaml

overlays/
├── dev/
├── qa/
└── prod/
```

---

# 69. Kustomization Base

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
```

---

# 70. Kustomization Production Overlay

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: roboshop-prod

resources:
  - ../../base

patches:
  - path: deployment-patch.yaml
```

---

# 71. Environment Patch

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
spec:
  replicas: 5
```

Kustomize applies the patch to the base resource.

---

# 72. Helm vs Kustomize

Helm is strong for:

```text
reusable charts
parameterized configuration
packaged applications
```

Kustomize is strong for:

```text
base + overlays
patch-based customization
native Kubernetes YAML
```

Choose based on team architecture.

---

# 73. Production Argo CD Helm Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cart-prod
  namespace: argocd
spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/example-org/gitops-repo.git
    targetRevision: main
    path: helm/roboshop/cart

    helm:
      releaseName: cart
      valueFiles:
        - values.yaml
        - values-prod.yaml

  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true

    syncOptions:
      - PruneLast=true

    retry:
      limit: 5
      backoff:
        duration: 15s
        factor: 2
        maxDuration: 5m
```

---

# 74. Production Argo CD Kustomize Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: cart-prod
  namespace: argocd
spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/example-org/gitops-repo.git
    targetRevision: main
    path: applications/roboshop/cart/overlays/prod

  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

# 75. Sync Windows

Production teams may restrict deployments during protected windows.

Conceptually:

```text
allow deployment:
08:00-18:00

deny:
outside change window
```

Exact configuration should be implemented through the Argo CD Project sync-window mechanism and organizational change policy.

---

# 76. Manual Production Approval

A common production approach:

```text
CI
 |
 v
Git PR
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
manual production sync
```

This provides stronger change control.

---

# 77. Automated Production Sync

Another model:

```text
Git merge
 |
 v
Argo CD
 |
 v
automated sync
```

This is appropriate only when:

```text
tests
security gates
observability
rollback
ownership
```

are mature.

---

# 78. CI + GitOps

The CI pipeline should build and validate.

Example:

```text
checkout
 |
test
 |
SonarQube
 |
Trivy
 |
Veracode
 |
build image
 |
push ECR
 |
update GitOps image digest
```

Argo CD then deploys.

---

# 79. Jenkins Example Flow

Conceptual:

```text
Jenkins
 |
 +--> Maven/npm tests
 +--> SonarQube
 +--> Trivy
 +--> Veracode
 |
 +--> Docker build
 |
 +--> ECR push
 |
 +--> GitOps PR
```

Do not let Jenkins directly run:

```bash
kubectl apply
```

in a pull-based GitOps architecture unless there is a deliberate exception.

---

# 80. GitOps Image Update

The CI system updates:

```yaml
image:
  digest: sha256:...
```

in the GitOps repository.

Then:

```text
Git
 |
 v
Argo CD
 |
 v
EKS
```

---

# 81. Image Updater

An organization may use Argo CD Image Updater or another controlled automation.

If used:

```text
ECR
 |
 v
image automation
 |
 v
Git
 |
 v
Argo CD
```

Security and write permissions must be tightly controlled.

---

# 82. Immutable Promotion

Recommended flow:

```text
Build image
   |
   v
digest D1
   |
   +--> dev
   +--> QA
   +--> prod
```

Do not rebuild the same source independently for each environment if the goal is immutable promotion.

---

# 83. Production Git PR

A production image update should be reviewable.

Example diff:

```diff
- digest: sha256:OLD
+ digest: sha256:NEW
```

This gives clear auditability.

---

# 84. Argo CD Notifications

Notifications can inform:

```text
sync succeeded
sync failed
health degraded
```

Use notification channels approved by the organization.

---

# 85. Production Notification Flow

```text
Argo CD
 |
 v
Notification Controller
 |
 +--> Slack
 +--> Email
 +--> Incident platform
```

---

# 86. Application Finalizer

A production Application often uses:

```yaml
finalizers:
  - resources-finalizer.argocd.argoproj.io
```

This makes deletion behavior explicit.

---

# 87. Resource Tracking

Argo CD must know which resources belong to an Application.

Avoid manual manipulation of ownership metadata.

If multiple systems manage the same resource, ownership conflicts can occur.

---

# 88. One Resource, One Owner

A strong production principle:

```text
one declarative owner
```

For example:

```text
Argo CD -> Deployment
HPA -> replica count
AWS controller -> ALB infrastructure
```

Do not let two systems continuously overwrite the same field.

---

# 89. Controller Conflict Example

Bad:

```text
Argo CD sets replicas=5
HPA sets replicas=8
```

This can create continuous reconciliation noise.

Design field ownership deliberately.

---

# 90. Server-Side Apply

For some resource-management patterns:

```yaml
syncOptions:
  - ServerSideApply=true
```

can be useful.

Use it when field ownership and large resources justify it.

---

# 91. CreateNamespace

If desired:

```yaml
syncOptions:
  - CreateNamespace=true
```

Argo CD can create the destination namespace.

In strongly governed environments, namespace creation may instead be managed by a platform bootstrap Application.

---

# 92. PruneLast

Example:

```yaml
syncOptions:
  - PruneLast=true
```

This can help apply desired resources before pruning obsolete ones.

This is useful when deletion ordering matters.

---

# 93. Replace

Some advanced sync options can replace resources rather than patching.

This can be disruptive.

Do not use destructive synchronization options without understanding:

```text
resource downtime
immutable fields
dependencies
```

---

# 94. Force

Forceful replacement can recreate resources.

Treat it as an exception, not a default production setting.

---

# 95. Sync Waves

Example annotations:

```yaml
metadata:
  annotations:
    argocd.argoproj.io/sync-wave: "-1"
```

Then:

```text
wave -1 -> configuration
wave 0  -> application
wave 1  -> post-deployment
```

---

# 96. Hook

Example PreSync Job:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: cart-precheck
  namespace: roboshop
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: precheck
          image: public.ecr.aws/docker/library/busybox:1.36
          command:
            - sh
            - -c
            - |
              echo "Running pre-deployment checks"
              exit 0
```

Use trusted image sources and pinned versions in production.

---

# 97. PostSync Hook

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: cart-postsync
  namespace: roboshop
  annotations:
    argocd.argoproj.io/hook: PostSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: validation
          image: public.ecr.aws/docker/library/curl:8.10.1
          command:
            - sh
            - -c
            - |
              curl --fail http://cart.roboshop.svc.cluster.local/health/ready
```

Pin images to approved registries and versions in production.

---

# 98. Hook Security

Hooks can execute code.

Therefore:

```text
review them
restrict permissions
pin images
avoid arbitrary curl-to-shell
```

---

# 99. Migration Hook

Database migrations may use:

```text
PreSync
```

but migrations must be designed for:

```text
backward compatibility
retry
idempotency
rollback
```

---

# 100. Production YAML Validation

Before merging:

```bash
kubectl apply --dry-run=server -f manifests/
```

where appropriate.

For Helm:

```bash
helm lint ./helm/roboshop/cart
helm template cart ./helm/roboshop/cart -f values.yaml -f values-prod.yaml
```

---

# 101. Kubeconform / Schema Validation

A production CI pipeline can use:

```text
kubeconform
kubeval where still appropriate
Helm lint
```

to catch malformed manifests before Argo CD sees them.

---

# 102. Argo CD Diff in CI

For environments where it is operationally appropriate:

```bash
argocd app diff cart-prod
```

can be part of deployment validation.

Do not expose Argo CD credentials unnecessarily in CI.

---

# 103. YAML Formatting

Use:

```text
yamllint
```

to catch:

```text
indentation
syntax
formatting
```

---

# 104. Policy Validation

Production pipelines can enforce:

```text
no privileged containers
resources required
runAsNonRoot
approved registries
no latest tags
```

using policy tools such as:

```text
Kyverno
OPA Gatekeeper
Conftest
```

according to organizational standards.

---

# 105. Example Policy Requirement

Every production Deployment should have:

```text
resources.requests
resources.limits
readinessProbe
livenessProbe
securityContext
```

This can be automatically checked.

---

# 106. Admission Control

Even if GitOps contains an unsafe manifest:

```text
Kubernetes admission policy
```

can reject it.

This creates defense in depth:

```text
Git review
+
CI policy
+
Argo CD
+
Kubernetes admission
```

---

# 107. Production YAML Security Checklist

```text
[ ] No plaintext production secrets
[ ] No latest image
[ ] Immutable image digest
[ ] Resource requests
[ ] Resource limits
[ ] Probes
[ ] Non-root
[ ] No privilege escalation
[ ] Capabilities dropped
[ ] Appropriate ServiceAccount
[ ] Network policy
[ ] PDB where required
[ ] HPA where required
[ ] TLS
[ ] Restricted ALB
```

---

# 108. Production Application Complete Example

The following resources form a realistic application set:

```text
Namespace
ServiceAccount
ConfigMap
ExternalSecret
Deployment
Service
Ingress
HPA
PDB
NetworkPolicy
```

They should generally be generated by Helm or Kustomize rather than manually duplicated for every service.

---

# 109. Complete Resource Relationship

```text
Application
    |
    +--> Namespace
    |
    +--> ServiceAccount
    |
    +--> ConfigMap
    |
    +--> ExternalSecret
    |
    +--> Deployment
    |       |
    |       +--> Pods
    |
    +--> Service
    |
    +--> Ingress
    |
    +--> HPA
    |
    +--> PDB
    |
    +--> NetworkPolicy
```

---

# 110. Production Application Lifecycle

```text
Git PR
 |
 v
Validation
 |
 v
Security scan
 |
 v
Merge
 |
 v
Argo CD detects revision
 |
 v
Manifest generation
 |
 v
Diff
 |
 v
Sync
 |
 v
Kubernetes
 |
 v
Pods
 |
 v
Health checks
 |
 v
ALB
 |
 v
Users
```

---

# 111. Multi-Environment Values

Example:

```text
values.yaml
values-dev.yaml
values-qa.yaml
values-prod.yaml
```

The base should contain:

```text
safe defaults
common configuration
```

Environment files should contain:

```text
environment-specific values
```

---

# 112. Avoid Environment Duplication

Do not copy entire Deployment YAML three times.

Prefer:

```text
base
+
environment overrides
```

This reduces configuration drift.

---

# 113. Production Environment Difference

Dev:

```yaml
replicaCount: 1
```

QA:

```yaml
replicaCount: 2
```

Prod:

```yaml
replicaCount: 5
```

Production can additionally use:

```text
PDB
HPA
topology spreading
stricter policies
```

---

# 114. Multi-Cluster Values

Cluster-specific values may include:

```text
region
domain
ALB certificate
resource sizing
external dependencies
```

Do not put sensitive values directly into Git.

---

# 115. Multi-Account Architecture

Example:

```text
AWS Organization
 |
 +--> Dev Account
 |      |
 |      +--> EKS Dev
 |
 +--> QA Account
 |      |
 |      +--> EKS QA
 |
 +--> Production Account
        |
        +--> EKS Prod
```

Central Argo CD may manage all clusters if organizational security allows.

---

# 116. Cluster Registration

Conceptually:

```text
Central Argo CD
      |
      +--> EKS Dev
      +--> EKS QA
      +--> EKS Prod
```

Cluster credentials must be stored and protected by Argo CD.

Use the Argo CD CLI or supported configuration mechanisms.

---

# 117. Cluster Labels

Production labels:

```text
environment=prod
region=ap-south-1
account=prod
application=roboshop
```

These enable ApplicationSet targeting.

---

# 118. Cluster Generator Targeting

```yaml
generators:
  - clusters:
      selector:
        matchLabels:
          environment: prod
          application: roboshop
```

This avoids hard-coding every target cluster.

---

# 119. Multi-Cluster Security

Central Argo CD requires access to target clusters.

Therefore secure:

```text
Argo CD -> target EKS
```

with:

```text
least privilege
network controls
RBAC
credential protection
audit
```

---

# 120. Cluster RBAC

Avoid giving Argo CD:

```text
cluster-admin
```

unless genuinely required.

Prefer permissions based on:

```text
project
namespace
resource
operation
```

---

# 121. Production Argo CD RBAC

Example concept:

```csv
p, role:roboshop-prod, applications, get, roboshop-prod/*, allow
p, role:roboshop-prod, applications, sync, roboshop-prod/*, allow
p, role:roboshop-prod, applications, update, roboshop-prod/*, allow
g, platform-team, role:roboshop-prod
```

The exact Argo CD RBAC configuration belongs in `argocd-rbac-cm`.

---

# 122. RBAC ConfigMap Example

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.csv: |
    p, role:prod-readonly, applications, get, roboshop-prod/*, allow
    p, role:prod-deployer, applications, get, roboshop-prod/*, allow
    p, role:prod-deployer, applications, sync, roboshop-prod/*, allow
    g, platform-readonly, role:prod-readonly
    g, platform-deployers, role:prod-deployer

  policy.default: role:readonly
```

Use SSO group names that actually exist in your identity provider.

---

# 123. RBAC Principle

Separate:

```text
view
sync
admin
```

permissions.

Not every developer needs production sync permission.

---

# 124. Repository Credentials

Do not store private repository passwords in Git.

Configure credentials through:

```text
Argo CD repository secrets
```

or approved secret-management integration.

---

# 125. Repository Credential Secret Concept

A repository credential Secret is typically labeled for Argo CD repository integration.

Example structure varies by Argo CD version/configuration.

Conceptually:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: repo-gitops
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
type: Opaque
stringData:
  type: git
  url: https://github.com/example-org/private-gitops.git
  username: git-user
  password: REPLACE_WITH_TOKEN
```

Do not commit the real credential.

---

# 126. SSH Repository

For SSH repositories, use an approved private key credential and known-host configuration.

Never disable host verification simply to make a connection work.

---

# 127. Repository Authentication Troubleshooting

```bash
argocd repo list
```

Then inspect:

```text
repository URL
connection state
credential
known hosts
network
```

---

# 128. Production Repository Access

Prefer:

```text
read-only Git credential
```

for Argo CD where possible.

CI needs write access to GitOps repository if it updates image references, but that should be a separate identity.

---

# 129. Separation of Duties

Example:

```text
CI:
build + scan + update GitOps PR

Argo CD:
read Git + deploy to Kubernetes

Developer:
PR/change

Production approver:
approve promotion
```

This reduces direct production mutation.

---

# 130. AppProject Destination Restrictions

A production project should restrict destinations.

Example:

```yaml
destinations:
  - namespace: roboshop
    server: https://prod-cluster.example
```

Do not allow:

```text
*
```

unless there is a deliberate reason.

---

# 131. Source Repository Restrictions

Use:

```yaml
sourceRepos:
  - https://github.com/example-org/gitops-repo.git
```

rather than unrestricted repositories.

---

# 132. Resource Whitelisting

Restrict cluster-scoped resources.

Example:

```yaml
clusterResourceWhitelist:
  - group: ""
    kind: Namespace
```

Only allow additional cluster resources when required.

---

# 133. ApplicationSet Governance

ApplicationSets are powerful because one object can create many Applications.

Therefore protect:

```text
ApplicationSet repositories
generator configuration
template
cluster selectors
```

A malicious change could target many clusters.

---

# 134. Production YAML Review

For every PR ask:

```text
Which cluster?
Which namespace?
Which resources?
Which image?
Which permissions?
Which secrets?
Which traffic path?
Which rollback?
```

---

# 135. YAML Review Checklist

```text
[ ] Correct API version
[ ] Correct namespace
[ ] Correct cluster
[ ] Correct image digest
[ ] Correct resource sizing
[ ] Probes valid
[ ] Service selector correct
[ ] ALB rules correct
[ ] HPA target correct
[ ] PDB compatible
[ ] NetworkPolicy compatible
[ ] No plaintext secrets
[ ] RBAC least privilege
```

---

# 136. Common YAML Mistake: Selector Mismatch

Deployment:

```yaml
selector:
  matchLabels:
    app: cart
```

Pod:

```yaml
labels:
  app: shopping-cart
```

This breaks the Deployment selector relationship.

Always ensure selectors and labels match.

---

# 137. Common YAML Mistake: Wrong Target Port

Service:

```yaml
targetPort: 8080
```

Pod:

```yaml
containerPort: 3000
```

This can result in traffic failures.

Use named ports when appropriate.

---

# 138. Common YAML Mistake: Missing Requests

Without requests:

```text
HPA behavior can be affected
scheduling is less predictable
capacity planning is harder
```

Define realistic requests.

---

# 139. Common YAML Mistake: Excessive Limits

Example:

```yaml
limits:
  cpu: 16
```

for a service that normally needs:

```text
500m
```

can distort capacity planning.

Tune limits from workload behavior.

---

# 140. Common YAML Mistake: Bad Probe

If readiness is too strict:

```text
Pods never receive traffic
```

If liveness is too aggressive:

```text
healthy but slow Pods restart
```

Tune probes based on application startup and failure behavior.

---

# 141. Common YAML Mistake: Plaintext Secret

Never commit:

```yaml
password: MyProductionPassword
```

Use:

```text
External Secrets
sealed secrets
SOPS
or approved secret system
```

---

# 142. Common YAML Mistake: latest Tag

Avoid:

```yaml
image: cart:latest
```

Use:

```text
version
```

or preferably:

```text
immutable digest
```

---

# 143. Common YAML Mistake: Broad ServiceAccount

Avoid unnecessary:

```text
cluster-admin
```

for application workloads.

Use workload identity and namespace-scoped permissions.

---

# 144. Common YAML Mistake: Public Service

Do not expose internal services as:

```text
LoadBalancer
```

unless intentionally required.

Prefer:

```text
ClusterIP
```

and a controlled ingress layer.

---

# 145. Common YAML Mistake: Unbounded Prune

Prune is powerful.

Before enabling:

```yaml
prune: true
```

ensure:

```text
resource ownership
repository structure
deletion policies
```

are understood.

---

# 146. Common YAML Mistake: Environment Mix-Up

A prod Application accidentally points to:

```text
values-qa.yaml
```

This is why:

```text
labels
paths
reviews
ApplicationSet templates
```

must be carefully designed.

---

# 147. Production Application Naming

Use names such as:

```text
cart-dev
cart-qa
cart-prod
```

or:

```text
roboshop-cart-prod
```

Names should clearly identify:

```text
application
environment
```

---

# 148. Production Labels

Recommended:

```yaml
labels:
  app.kubernetes.io/name: cart
  app.kubernetes.io/part-of: roboshop
  app.kubernetes.io/component: backend
  app.kubernetes.io/managed-by: argocd
  environment: prod
  team: roboshop
```

Do not rely on labels alone for security boundaries.

---

# 149. Annotations

Use annotations for:

```text
ALB configuration
Argo CD hooks
sync waves
external tooling
```

Avoid excessive annotations that obscure ownership.

---

# 150. Kubernetes Secret Encryption

EKS/Kubernetes secret storage should use the platform's approved encryption-at-rest design.

GitOps does not replace Kubernetes encryption.

---

# 151. AWS KMS

AWS KMS can be used as part of the encryption architecture for EKS and AWS secret services.

The exact implementation depends on the EKS version and cluster design.

---

# 152. External Secrets Security

External Secrets should have:

```text
least-privilege IAM
restricted secret paths
namespace boundaries
audit
```

Example concept:

```text
cart ServiceAccount
  |
  v
IAM role
  |
  v
only /roboshop/prod/cart/*
```

---

# 153. Production Helm Values and Secrets

Good:

```yaml
redis:
  host: redis.roboshop.svc.cluster.local
```

Secret value:

```text
AWS Secrets Manager
```

Bad:

```yaml
redisPassword: production-secret
```

---

# 154. Production YAML Deployment Flow

```text
values-prod.yaml
      |
      v
Helm rendering
      |
      v
Argo CD diff
      |
      v
Sync
      |
      v
Deployment
      |
      v
Pods
```

---

# 155. Manifest Rendering Troubleshooting

If Argo CD says:

```text
manifest generation error
```

check:

```text
Helm syntax
values files
chart dependencies
Kustomize overlays
plugins
```

Run locally:

```bash
helm lint
helm template
```

---

# 156. Helm Dependency

If a chart uses dependencies:

```text
Chart.yaml
Chart.lock
charts/
```

ensure CI/Argo CD can resolve them.

Prefer deterministic dependencies.

---

# 157. Chart Lock

Commit:

```text
Chart.lock
```

when using Helm dependencies where appropriate.

This helps reproducibility.

---

# 158. OCI Helm Charts

Argo CD can work with OCI-based Helm artifacts depending on version/configuration.

If using OCI:

```text
registry authentication
artifact immutability
versioning
```

must be controlled.

---

# 159. Production YAML Testing

Use multiple validation layers:

```text
YAML lint
Helm lint
Helm template
Kubernetes schema
policy
security scan
Argo CD diff
```

---

# 160. GitOps PR Pipeline

```text
Pull Request
 |
 +--> yamllint
 |
 +--> helm lint
 |
 +--> helm template
 |
 +--> kubeconform
 |
 +--> policy
 |
 +--> security
 |
 v
Approval
 |
 v
Merge
```

---

# 161. Production Promotion

```text
Build
 |
 v
ECR
 |
 v
Dev
 |
 v
QA
 |
 +--> tests
 +--> security
 |
 v
Prod PR
 |
 v
Approval
 |
 v
Argo CD
```

---

# 162. Rollback

GitOps rollback should normally mean:

```text
revert Git change
```

rather than:

```text
manual kubectl rollback
```

This restores the source of truth.

---

# 163. Emergency Rollback

If production is actively failing:

```text
1. restore service quickly according to incident policy
2. revert desired state in Git
3. allow Argo CD to reconcile
4. verify health
5. document emergency action
```

---

# 164. Deployment History

Argo CD Application history helps identify:

```text
revision
deployment event
source
```

Git history provides the stronger long-term audit trail.

---

# 165. Disaster Recovery

A GitOps repository is a major recovery asset.

If a cluster is rebuilt:

```text
Terraform
 |
 v
EKS
 |
 v
Argo CD bootstrap
 |
 v
GitOps repository
 |
 v
Applications
```

---

# 166. DR Bootstrap

Example:

```text
1. Create AWS infrastructure.
2. Create EKS.
3. Install Argo CD.
4. Configure repository access.
5. Configure target cluster.
6. Apply root Application.
7. ApplicationSets generate Applications.
8. Applications reconcile.
9. Validate workloads.
```

---

# 167. Root Application as Bootstrap

The root Application can point to:

```text
argocd/applications/
```

or:

```text
argocd/bootstrap/
```

This allows Git to reconstruct the application platform.

---

# 168. DR Dependencies

Remember:

```text
Git repository
container registry
secret store
DNS
ACM
AWS IAM
EKS
Argo CD
monitoring
logging
```

GitOps alone does not recreate external dependencies automatically unless those dependencies are also codified.

---

# 169. Terraform + GitOps Boundary

Use Terraform for infrastructure such as:

```text
VPC
EKS
IAM
ECR
RDS
S3
ALB-related infrastructure where appropriate
```

Use Argo CD for Kubernetes application state such as:

```text
Deployment
Service
Ingress
HPA
ConfigMap
NetworkPolicy
```

---

# 170. Avoid Terraform vs Argo CD Ownership Conflict

Bad:

```text
Terraform manages Deployment replicas
Argo CD manages Deployment replicas
```

Choose one owner.

---

# 171. RoboShop Ownership Model

Example:

```text
Terraform
  |
  +--> VPC
  +--> EKS
  +--> IAM
  +--> ECR
  +--> RDS
  +--> S3

Argo CD
  |
  +--> Namespaces
  +--> Deployments
  +--> Services
  +--> ALB Ingress
  +--> HPA
  +--> PDB
  +--> NetworkPolicy
```

---

# 172. Jenkins Ownership

Jenkins should not own runtime Kubernetes resources in a pure GitOps model.

It should own:

```text
build
test
scan
artifact
GitOps update
```

---

# 173. GitHub Actions Alternative

Same principle:

```text
GitHub Actions
 |
 +--> build
 +--> test
 +--> scan
 +--> ECR
 +--> GitOps PR
```

Then:

```text
Argo CD -> EKS
```

---

# 174. DevSecOps Integration

Production GitOps should include:

```text
SAST
SCA
image scanning
DAST where applicable
IaC scanning
policy validation
```

The user's stack includes:

```text
SonarQube
Trivy
Veracode
```

---

# 175. Security Gate Example

```text
Developer
 |
 v
CI
 |
 +--> SonarQube
 +--> Trivy
 +--> Veracode
 |
 +--> build image
 |
 +--> ECR
 |
 v
GitOps PR
```

Only approved artifacts should reach production.

---

# 176. Production YAML Security Gate

CI can reject:

```text
privileged: true
runAsRoot
latest tag
missing resources
plaintext Secret
unapproved registry
```

before merge.

---

# 177. Argo CD Project for RoboShop

A stronger project might include:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: roboshop-prod
  namespace: argocd
spec:
  description: Production RoboShop workloads

  sourceRepos:
    - https://github.com/example-org/gitops-repo.git

  destinations:
    - server: https://prod-eks.example
      namespace: roboshop

  namespaceResourceWhitelist:
    - group: apps
      kind: Deployment
    - group: ""
      kind: Service
    - group: ""
      kind: ConfigMap
    - group: ""
      kind: Secret
    - group: networking.k8s.io
      kind: Ingress
    - group: autoscaling
      kind: HorizontalPodAutoscaler
    - group: policy
      kind: PodDisruptionBudget
    - group: networking.k8s.io
      kind: NetworkPolicy
```

---

# 178. Production Application Example with ALB

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-prod
  namespace: argocd
spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/example-org/gitops-repo.git
    targetRevision: main
    path: helm/roboshop

    helm:
      valueFiles:
        - values.yaml
        - values-prod.yaml

  destination:
    server: https://prod-eks.example
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true

    syncOptions:
      - PruneLast=true
```

---

# 179. Production Multi-Cluster ApplicationSet

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-prod-fleet
  namespace: argocd
spec:
  goTemplate: true

  generators:
    - clusters:
        selector:
          matchLabels:
            application: roboshop
            environment: prod

  template:
    metadata:
      name: 'roboshop-{{.name}}'

    spec:
      project: roboshop-prod

      source:
        repoURL: https://github.com/example-org/gitops-repo.git
        targetRevision: main
        path: helm/roboshop

        helm:
          valueFiles:
            - values.yaml
            - values-prod.yaml

      destination:
        server: '{{.server}}'
        namespace: roboshop

      syncPolicy:
        automated:
          prune: true
          selfHeal: true
```

`goTemplate` syntax and generator fields must match the installed ApplicationSet controller/version.

---

# 180. ApplicationSet Matrix Example

A matrix generator can combine dimensions such as:

```text
environment
+
cluster
```

Conceptually:

```text
dev + cluster-a
dev + cluster-b
qa  + cluster-a
qa  + cluster-b
prod + cluster-c
```

This is useful for fleet management but must be bounded to prevent accidental application explosion.

---

# 181. Git Directory Generator

Repository:

```text
applications/
├── cart/
├── catalog/
├── orders/
└── payment/
```

ApplicationSet can discover directories and create Applications dynamically.

---

# 182. Pull Request Generator

For preview environments:

```text
Pull Request
 |
 v
ApplicationSet
 |
 v
temporary namespace
 |
 v
preview Application
```

This is useful for ephemeral testing.

---

# 183. Preview Environment Example

```text
PR #152
 |
 v
preview-pr-152
 |
 v
EKS namespace
 |
 v
ALB/path
```

Cleanup should be automatic when the PR closes.

---

# 184. Production Caution for PR Environments

PR environments can create:

```text
ALBs
Pods
EBS
DNS
cost
```

Use:

```text
quotas
TTL
cleanup
resource limits
```

---

# 185. Production YAML Governance

Store:

```text
Application
ApplicationSet
AppProject
Helm
Kustomize
RBAC
monitoring
policies
```

in reviewed Git paths.

---

# 186. Code Owners

Use Git repository ownership controls where available:

```text
platform team
security team
application team
```

Production configuration should require appropriate reviewers.

---

# 187. Branch Protection

Production GitOps repository should use:

```text
protected main
PR reviews
status checks
no direct pushes
```

---

# 188. Signed Commits

Organizations may use:

```text
GPG
SSH signing
Sigstore
```

depending on policy.

This strengthens Git provenance.

---

# 189. Artifact Provenance

A production image should be traceable to:

```text
source commit
build
security scans
ECR digest
GitOps revision
deployment
```

---

# 190. Complete Provenance Chain

```text
Source commit
     |
     v
CI build
     |
     v
Image digest
     |
     v
ECR
     |
     v
GitOps commit
     |
     v
Argo CD revision
     |
     v
EKS workload
```

This is a powerful interview and production concept.

---

# 191. Production Verification

After sync:

```bash
argocd app get roboshop-prod
```

Then:

```bash
kubectl get deploy -n roboshop
kubectl get pods -n roboshop
kubectl get svc -n roboshop
kubectl get ingress -n roboshop
kubectl get hpa -n roboshop
```

---

# 192. Application Diff

Before a production sync:

```bash
argocd app diff roboshop-prod
```

Look for:

```text
unexpected namespace
unexpected image
unexpected replicas
unexpected resources
```

---

# 193. Sync

```bash
argocd app sync roboshop-prod
```

Then verify:

```bash
argocd app get roboshop-prod
```

---

# 194. History

```bash
argocd app history roboshop-prod
```

Use this to understand prior revisions.

---

# 195. Rollback

```bash
argocd app rollback roboshop-prod <ID>
```

Treat rollback as an operational action and restore Git afterward so desired state remains correct.

---

# 196. Cluster List

```bash
argocd cluster list
```

Useful for centralized Argo CD.

---

# 197. Repository List

```bash
argocd repo list
```

Useful when Application rendering fails because Git access is broken.

---

# 198. Kubernetes Application CRD

Argo CD Applications are Kubernetes custom resources.

```bash
kubectl get applications -n argocd
```

This is useful when the Argo CD CLI is unavailable.

---

# 199. Describe Application

```bash
kubectl describe application roboshop-prod -n argocd
```

Inspect:

```text
conditions
operation state
sync status
health
resources
```

---

# 200. Production Verification Commands

```bash
kubectl get pods -n roboshop -o wide
kubectl get events -n roboshop --sort-by=.lastTimestamp
kubectl get endpointslice -n roboshop
kubectl get ingress -n roboshop
kubectl get hpa -n roboshop
kubectl get pdb -n roboshop
```

---

# 201. Complete Production Checklist

```text
Git
[ ] protected branch
[ ] PR reviews
[ ] code ownership
[ ] immutable artifacts

CI
[ ] tests
[ ] SonarQube
[ ] Trivy
[ ] Veracode
[ ] image pushed to ECR

Argo CD
[ ] Project
[ ] Application
[ ] repository access
[ ] cluster access
[ ] RBAC
[ ] sync policy

Kubernetes
[ ] Namespace
[ ] Deployment
[ ] Service
[ ] ConfigMap
[ ] ExternalSecret
[ ] Ingress
[ ] HPA
[ ] PDB
[ ] NetworkPolicy

Security
[ ] non-root
[ ] no privilege escalation
[ ] capabilities dropped
[ ] least privilege
[ ] secrets externalized

Operations
[ ] Prometheus
[ ] Grafana
[ ] ELK
[ ] alerts
[ ] runbooks
```

---

# 202. Production Architecture Summary

```text
Developer
   |
   v
Git
   |
   v
CI
 |
 +--> Test
 +--> SonarQube
 +--> Trivy
 +--> Veracode
 |
 v
Docker
 |
 v
ECR
 |
 v
GitOps PR
 |
 v
GitOps Repository
 |
 v
Argo CD
 |
 v
AppProject
 |
 v
Application / ApplicationSet
 |
 v
Helm / Kustomize
 |
 v
EKS
 |
 +--> Deployment
 +--> Service
 +--> HPA
 +--> PDB
 +--> NetworkPolicy
 +--> ALB
 |
 +--> Prometheus
 |      |
 |      v
 |    Grafana
 |
 +--> ELK
```

---

# 203. Production Principles

Remember:

```text
Git owns desired state.
Argo CD owns reconciliation.
Terraform owns infrastructure.
CI owns build/test/security/artifacts.
ECR owns container artifacts.
Kubernetes owns runtime execution.
HPA owns runtime replica scaling.
ALB Controller owns AWS load-balancer resources.
Prometheus owns metrics collection.
Grafana owns visualization.
ELK owns log search.
```

The exact ownership boundaries can vary, but overlapping controllers must be avoided.

---

# 204. Interview Questions

## Q1. Why use Kubernetes YAML with GitOps?

### Answer

Because declarative YAML describes the desired state. Git provides versioning, review, history and rollback, while Argo CD continuously reconciles that desired state with Kubernetes.

---

## Q2. Why should you not use `kubectl apply` from Jenkins in a pure GitOps architecture?

### Answer

It bypasses the GitOps control plane and makes the cluster state dependent on CI execution. A pull-based design keeps Git as the source of truth and allows Argo CD to reconcile the cluster.

---

## Q3. Why use image digests?

### Answer

Digests provide immutable artifact identity. They prevent a mutable tag from silently pointing to another image.

---

## Q4. What is the purpose of an AppProject?

### Answer

It provides governance boundaries around repositories, destinations, resource types and access permissions.

---

## Q5. Why use ApplicationSet?

### Answer

ApplicationSet automates generation of multiple Applications from reusable templates and generators. It is especially useful for multi-environment and multi-cluster deployments.

---

## Q6. How can one Argo CD manage multiple EKS clusters?

### Answer

Each target EKS cluster is registered with Argo CD. Applications specify the destination cluster, and ApplicationSets can dynamically select clusters using labels.

---

## Q7. How do you protect production secrets?

### Answer

I do not commit plaintext credentials to Git. I use a secret-management solution such as AWS Secrets Manager with External Secrets Operator or another approved mechanism.

---

## Q8. How do Terraform and Argo CD coexist?

### Answer

Terraform manages infrastructure such as VPC, EKS, IAM and ECR, while Argo CD manages Kubernetes application resources. The same resource should not have two competing declarative owners.

---

## Q9. Why is `selfHeal` useful?

### Answer

It allows Argo CD to correct unintended runtime drift back to the Git-defined desired state.

---

## Q10. Why is `prune` dangerous?

### Answer

If a resource is removed from Git and pruning is enabled, Argo CD may delete it. Incorrect repository changes can therefore cause destructive production actions.

---

## Q11. What is a PDB?

### Answer

A PodDisruptionBudget limits voluntary disruption so that a minimum amount of application capacity remains available during operations such as node maintenance.

---

## Q12. Why do you need both readiness and liveness probes?

### Answer

Readiness determines whether the Pod should receive traffic. Liveness determines whether the container should be restarted. They solve different problems.

---

## Q13. Why use HPA with GitOps?

### Answer

GitOps defines the HPA policy and bounds, while Kubernetes HPA dynamically adjusts replicas based on runtime metrics.

---

## Q14. What happens if HPA changes replicas while Argo CD manages the Deployment?

### Answer

The two controllers can conflict if Argo CD continuously enforces the replica field. The design can use appropriate ignore-differences configuration so HPA owns runtime replica count.

---

## Q15. Why use ALB Ingress?

### Answer

It provides a centralized AWS load-balancing layer for external HTTP/HTTPS traffic while internal microservices remain ClusterIP services.

---

## Q16. Why use `target-type: ip`?

### Answer

The AWS Load Balancer Controller can route ALB traffic directly to Pod IPs, which is a common EKS architecture.

---

## Q17. Why use NetworkPolicy?

### Answer

NetworkPolicy limits which Pods can communicate, reducing the blast radius of compromised workloads and enforcing application communication boundaries.

---

## Q18. What is the difference between ConfigMap and Secret?

### Answer

ConfigMap stores non-sensitive configuration. Secret is intended for sensitive values, although Secret values still require appropriate encryption and access controls. For production, external secret management is preferable.

---

## Q19. Why use Helm with Argo CD?

### Answer

Helm provides reusable, parameterized application templates. Argo CD uses Helm to render desired Kubernetes manifests and then reconciles them into the cluster.

---

## Q20. Helm or Kustomize?

### Answer

Helm is strong for reusable packaged applications and parameterized configuration. Kustomize is strong for base-plus-overlay customization of Kubernetes YAML. The choice depends on application and organizational needs.

---

## Q21. What is an ApplicationSet cluster generator?

### Answer

It discovers registered Argo CD clusters and generates Applications for clusters matching a selector, such as `environment=prod`.

---

## Q22. Why use cluster labels?

### Answer

Labels allow declarative targeting. Instead of hard-coding every cluster, ApplicationSet can select clusters based on environment, region, account or application labels.

---

## Q23. How do you rollback a GitOps deployment?

### Answer

Prefer reverting the Git change to restore the desired state. During an emergency, Argo CD rollback can be used according to the operational procedure, followed by reconciling Git so the source of truth is correct.

---

## Q24. Why use sync waves?

### Answer

They control deployment ordering when dependencies require one resource group to exist before another.

---

## Q25. What are hooks?

### Answer

Hooks are specially annotated resources that run at lifecycle points such as PreSync, Sync, PostSync or SyncFail.

---

## Q26. Why are hooks dangerous?

### Answer

Hooks can execute workloads with permissions in the cluster. Poorly controlled hooks can become a security and reliability risk.

---

## Q27. What is App of Apps?

### Answer

A root Argo CD Application manages child Application resources. It is commonly used to bootstrap and organize multiple applications.

---

## Q28. What should be stored in Git?

### Answer

Desired configuration, manifests, Helm charts, Application definitions, Projects, ApplicationSets and non-secret configuration. Sensitive secret values should use an approved external secret mechanism.

---

## Q29. What should not be stored in Git?

### Answer

Plaintext production credentials, private keys and other sensitive secrets.

---

## Q30. What is the production image flow?

### Answer

Developer code is tested and scanned in CI, the image is built and pushed to ECR, the immutable image digest is promoted through GitOps configuration, and Argo CD deploys the desired state into EKS.

---

# 205. Senior Interview Scenarios

## Scenario 1: Production deployment deleted a resource

Investigate:

```text
Git diff
Application diff
prune
resource ownership
AppProject
```

Determine whether the deletion was:

```text
expected
or
configuration error
```

---

## Scenario 2: HPA and Argo CD fight over replicas

Symptoms:

```text
replicas constantly changing
OutOfSync repeatedly
```

Solution:

```text
define ownership
configure ignoreDifferences if appropriate
```

---

## Scenario 3: ALB returns 503 after GitOps sync

Check:

```text
Ingress
Service
EndpointSlice
Pod readiness
target health
security groups
```

---

## Scenario 4: Application is Synced but not serving traffic

Check:

```text
Pod health
Service selector
Service endpoints
Ingress
ALB
readiness
```

---

## Scenario 5: ApplicationSet deployed to wrong cluster

Check:

```text
cluster labels
generator selector
template destination
Project destination restrictions
Git revision
```

---

## Scenario 6: Production secret changed unexpectedly

Investigate:

```text
ExternalSecret
AWS Secrets Manager
IAM
Kubernetes Secret
Git history
audit logs
```

---

## Scenario 7: Argo CD continuously shows OutOfSync

Check:

```text
diff
controller-managed fields
HPA
mutating webhooks
ignoreDifferences
```

---

## Scenario 8: Deployment is stuck Progressing

Check:

```text
Pods
readiness
events
resource capacity
image pull
probe
HPA
PDB
```

---

## Scenario 9: Rollback succeeded temporarily but Argo CD changed it back

Likely cause:

```text
Git still declares the newer revision
```

Fix:

```text
revert Git desired state
```

---

## Scenario 10: Cluster is rebuilt after disaster

Use:

```text
Terraform
+
Argo CD bootstrap
+
GitOps repository
+
external secret system
```

then validate:

```text
Applications
ALB
metrics
logs
DNS
```

---

# 206. Final Production YAML Checklist

Before production:

```text
Application
[ ] project restricted
[ ] repo restricted
[ ] destination restricted
[ ] sync policy intentional
[ ] prune understood
[ ] selfHeal intentional
[ ] retry configured
[ ] finalizer understood

ApplicationSet
[ ] generator bounded
[ ] cluster labels correct
[ ] environment labels correct
[ ] template reviewed
[ ] destination restricted

Kubernetes
[ ] namespace
[ ] Deployment
[ ] Service
[ ] probes
[ ] resources
[ ] securityContext
[ ] HPA
[ ] PDB
[ ] NetworkPolicy
[ ] ServiceAccount

AWS
[ ] ECR
[ ] IAM
[ ] ALB
[ ] ACM
[ ] security groups
[ ] DNS

Secrets
[ ] no plaintext credentials
[ ] external secret store
[ ] least privilege

Operations
[ ] Prometheus
[ ] Grafana
[ ] ELK
[ ] alerts
[ ] runbooks
[ ] rollback
```

---

# 207. Final Production Pattern

The recommended RoboShop GitOps pattern is:

```text
Developer
     |
     v
Application Git
     |
     v
Jenkins / GitHub Actions
     |
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
GitOps Repository
     |
     +--> Helm values
     +--> Application
     +--> ApplicationSet
     +--> Project
     |
     v
Argo CD
     |
     v
EKS
     |
     +--> Namespace
     +--> Deployment / Rollout
     +--> Service
     +--> HPA
     +--> PDB
     +--> NetworkPolicy
     +--> ALB Ingress
     |
     +--> Prometheus
     |       |
     |       v
     |     Grafana
     |
     +--> ELK
```

---

# 208. Key Production Lessons

1. **Git should remain the source of truth.**
2. **Use immutable image digests for production.**
3. **Keep CI responsible for build, test, security and artifact publication.**
4. **Keep Argo CD responsible for pull-based deployment and reconciliation.**
5. **Use AppProjects to enforce boundaries.**
6. **Use ApplicationSets for repeatable multi-environment and multi-cluster deployment.**
7. **Do not store plaintext secrets in Git.**
8. **Avoid Terraform and Argo CD managing the same resource fields.**
9. **Use resource requests, limits and probes.**
10. **Use HPA/PDB/topology controls for production availability.**
11. **Use AWS ALB Ingress for the external traffic path in the RoboShop architecture.**
12. **Use Prometheus/Grafana/ELK for operational visibility.**
13. **Use policy and security validation before merge.**
14. **Treat pruning and hooks as powerful production controls.**
15. **Make rollback restore Git desired state.**

---

# 209. Final Mental Model

The complete production GitOps chain is:

```text
CODE
 |
 v
CI
 |
 +--> TEST
 +--> SECURITY
 +--> BUILD
 |
 v
ECR
 |
 v
GITOPS
 |
 v
ARGO CD
 |
 +--> PROJECT
 +--> APPLICATION
 +--> APPLICATIONSET
 |
 v
HELM / KUSTOMIZE
 |
 v
EKS
 |
 +--> DEPLOYMENT
 +--> SERVICE
 +--> ALB
 +--> HPA
 +--> PDB
 +--> NETWORK POLICY
 |
 v
OBSERVABILITY
 |
 +--> PROMETHEUS
 +--> GRAFANA
 +--> ELK
 |
 v
OPERATIONS
 |
 +--> ALERT
 +--> TROUBLESHOOT
 +--> ROLLBACK
 +--> RECOVER
```

This is the production-oriented YAML foundation for the GitOps and Argo CD section.
