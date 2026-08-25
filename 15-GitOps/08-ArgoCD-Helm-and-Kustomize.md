# 08-ArgoCD-Helm-and-Kustomize

## 1. Purpose

This file explains how Argo CD works with the two most common Kubernetes configuration and packaging approaches:

```text
Helm
Kustomize
```

The focus is production GitOps rather than only basic syntax.

The goal is to understand:

```text
Git repository
      |
      v
Argo CD Application
      |
      +--> Helm
      |
      +--> Kustomize
      |
      +--> Plain YAML
      |
      v
Rendered Kubernetes manifests
      |
      v
Argo CD reconciliation
      |
      v
EKS
```

This file covers:

- Helm architecture
- Helm charts
- Chart.yaml
- values.yaml
- templates
- helpers
- dependencies
- environment values
- Argo CD Helm integration
- Helm parameters
- valueFiles
- valuesObject
- releaseName
- OCI charts
- private Helm repositories
- Kustomize bases
- overlays
- patches
- image transformations
- namespace handling
- generators
- components
- Helm vs Kustomize
- production repository structures
- RoboShop implementation
- production YAMLs
- troubleshooting
- security
- interview scenarios

---

# 2. Why Helm and Kustomize Matter in GitOps

A production Kubernetes platform may have:

```text
10s of Applications
100s of Kubernetes resources
Multiple environments
Multiple clusters
Different configuration per environment
```

Managing every YAML independently can become difficult.

Helm and Kustomize provide structured configuration mechanisms.

Argo CD consumes their output.

Important mental model:

```text
Helm/Kustomize
=
Manifest generation/configuration

Argo CD
=
Deployment + reconciliation
```

---

# 3. Argo CD Does Not Replace Helm

A common misconception is:

> Argo CD and Helm are competing deployment tools.

They are not necessarily competitors.

A common architecture is:

```text
Git
 |
 v
Helm Chart
 |
 v
Argo CD Repo Server
 |
 v
Rendered YAML
 |
 v
Application Controller
 |
 v
Kubernetes
```

Helm handles packaging/template rendering.

Argo CD handles GitOps reconciliation.

---

# 4. Helm Mental Model

A Helm chart is a package containing Kubernetes templates and configuration.

Typical structure:

```text
cart/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   └── _helpers.tpl
└── charts/
```

---

# 5. Chart.yaml

Example:

```yaml
apiVersion: v2
name: cart
description: RoboShop Cart service
type: application
version: 1.0.0
appVersion: "2026.08.19"
```

Important fields:

```text
apiVersion
name
description
type
version
appVersion
dependencies
```

---

# 6. Helm Chart Version vs Application Version

These are different.

```yaml
version: 1.0.0
appVersion: "2026.08.19"
```

`version` is the Helm chart/package version.

`appVersion` describes the application version.

Do not assume:

```text
chart version == image version
```

They can evolve independently.

---

# 7. Helm Chart Versioning

Example:

```text
Chart 1.0.0
Chart 1.0.1
Chart 1.1.0
```

Use semantic versioning where appropriate.

A chart change may be:

```text
Template change
Default value change
New resource
Dependency change
```

The application image can remain unchanged.

---

# 8. values.yaml

Example:

```yaml
replicaCount: 3

image:
  repository: nginx
  tag: "1.27"

service:
  type: ClusterIP
  port: 80

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

Values provide configurable inputs to templates.

---

# 9. Helm Templates

Example:

```yaml
spec:
  replicas: {{ .Values.replicaCount }}
```

Helm renders:

```yaml
spec:
  replicas: 3
```

Argo CD does not deploy the template directly.

It deploys the rendered Kubernetes manifest.

---

# 10. Helm Rendering Flow

```text
Chart
 |
 +--> Chart.yaml
 +--> values.yaml
 +--> templates
 |
 v
Helm rendering
 |
 v
Kubernetes YAML
 |
 v
Argo CD comparison
 |
 v
Kubernetes API
```

---

# 11. Helm Helpers

Example:

```yaml
{{- define "cart.fullname" -}}
{{ include "cart.name" . }}-{{ .Release.Name }}
{{- end }}
```

Helpers reduce duplication.

Typical helper file:

```text
templates/_helpers.tpl
```

Use helpers for:

```text
Names
Labels
Selectors
Annotations
Common metadata
```

---

# 12. Helm Label Strategy

Production labels should be consistent.

Example:

```yaml
app.kubernetes.io/name: cart
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
```

Consistent labels help:

```text
Monitoring
Troubleshooting
Cost allocation
Resource identification
```

---

# 13. Environment-Specific Values

Recommended structure:

```text
helm/cart/
├── Chart.yaml
├── values.yaml
├── templates/
└── values/
    ├── dev.yaml
    ├── qa.yaml
    └── prod.yaml
```

Example:

```yaml
# values/prod.yaml

replicaCount: 3

image:
  repository: <account>.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
  tag: "2026.08.19-abc123"
```

---

# 14. Base Values vs Environment Values

`values.yaml`:

```yaml
replicaCount: 2
```

Production:

```yaml
replicaCount: 5
```

Argo CD:

```yaml
helm:
  valueFiles:
    - values/prod.yaml
```

The environment-specific file overrides the defaults.

---

# 15. Why Not Duplicate Entire Charts?

Bad structure:

```text
cart-dev/
cart-qa/
cart-prod/
```

Each containing a complete chart.

This causes:

```text
Duplication
Configuration drift
Hard maintenance
Repeated fixes
```

Prefer:

```text
One reusable chart
+
Environment-specific values
```

when the environments share the same application structure.

---

# 16. Helm Values Precedence

When multiple configuration sources are used, Helm applies precedence rules.

Conceptually:

```text
Chart defaults
      |
      v
values.yaml
      |
      v
valueFiles
      |
      v
inline values / valuesObject
      |
      v
parameters
```

Exact precedence should be verified against the Argo CD/Helm version being used.

The practical rule is:

```text
More specific configuration overrides less specific configuration.
```

---

# 17. Argo CD Helm Configuration

Example:

```yaml
source:
  repoURL: https://github.com/example/roboshop-gitops.git
  targetRevision: main
  path: helm/cart

  helm:
    releaseName: cart
    valueFiles:
      - values/prod.yaml
```

Argo CD passes the source configuration to its manifest-generation process.

---

# 18. releaseName

Example:

```yaml
helm:
  releaseName: cart-prod
```

This controls the Helm release name used for rendering.

Choose names consistently.

For example:

```text
cart-dev
cart-qa
cart-prod
```

or:

```text
cart
```

when the namespace already provides strong environment isolation.

---

# 19. Why Release Naming Matters

Release names can affect:

```text
Generated resource names
Labels
Selectors
Helm template behavior
Operational identification
```

Changing a release name may cause resources to appear as different objects.

Treat release-name changes carefully.

---

# 20. valueFiles

Example:

```yaml
helm:
  valueFiles:
    - values/common.yaml
    - values/prod.yaml
```

This is a clean way to represent:

```text
Common configuration
+
Environment-specific configuration
```

---

# 21. valuesObject

Argo CD can provide values as structured data.

Conceptual:

```yaml
helm:
  valuesObject:
    replicaCount: 3
    image:
      tag: "2026.08.19"
```

This is useful for generated Application definitions.

However, excessive inline configuration can make Applications difficult to review.

Prefer Git-managed values files for substantial configuration.

---

# 22. values vs valuesObject

`values` is text:

```yaml
helm:
  values: |
    replicaCount: 3
```

`valuesObject` is structured YAML data:

```yaml
helm:
  valuesObject:
    replicaCount: 3
```

The exact supported fields depend on the Argo CD version.

Use the representation that keeps the Application maintainable.

---

# 23. Helm Parameters

Example:

```yaml
helm:
  parameters:
    - name: image.tag
      value: "2026.08.19-abc123"
```

This is useful when an external automation system needs to override a small number of values.

But avoid hiding the deployment version outside Git.

---

# 24. GitOps Image Update Pattern

Preferred:

```text
CI
 |
 v
Build image
 |
 v
Push ECR
 |
 v
Update GitOps values
 |
 v
Git commit
 |
 v
Argo CD
```

This keeps:

```text
Image version
+
Deployment state
```

auditable in Git.

---

# 25. Anti-Pattern: Direct Helm CLI Deployment

Avoid making the normal production path:

```bash
helm upgrade --install ...
```

from CI while Argo CD also manages the same resources.

This creates competing deployment mechanisms.

Prefer:

```text
CI -> GitOps repo -> Argo CD
```

---

# 26. Helm Dependencies

A chart can declare dependencies.

Example:

```yaml
dependencies:
  - name: redis
    version: 20.0.0
    repository: https://charts.example.com
```

Dependencies should be version-pinned.

Avoid uncontrolled dependency ranges for production.

---

# 27. Dependency Risk

A chart dependency can introduce:

```text
New Kubernetes resources
New permissions
New images
New configuration
New vulnerabilities
```

Therefore dependencies should be reviewed and scanned.

---

# 28. Private Helm Repositories

Production environments may use:

```text
JFrog Artifactory
AWS ECR OCI
Git-based charts
Private chart repository
```

Argo CD needs credentials where required.

Do not store:

```text
Username
Password
Token
```

inside Application manifests.

Use Argo CD repository credentials/secret management.

---

# 29. OCI Helm Charts

Modern Helm supports OCI registries.

Example conceptual source:

```text
oci://registry.example.com/helm/cart
```

OCI can integrate with artifact registries such as:

```text
Amazon ECR
JFrog Artifactory
```

Exact Argo CD configuration depends on the Argo CD version and repository integration.

---

# 30. ECR and Helm

The user's environment already uses:

```text
AWS ECR
```

for container images.

A production platform may also use ECR for OCI Helm artifacts.

Architecture:

```text
Helm chart
 |
 v
OCI registry / ECR
 |
 v
Argo CD
 |
 v
EKS
```

Separate repository permissions should be used for charts and images.

---

# 31. Helm Secrets

Do not put:

```yaml
password: production-password
```

into:

```text
values/prod.yaml
```

in a normal Git repository.

Instead use:

```text
External Secrets
AWS Secrets Manager
Secrets Store CSI Driver
SOPS
Sealed Secrets
```

depending on platform standards.

---

# 32. Helm and External Secrets

Example:

```text
Git
 |
 v
ExternalSecret manifest
 |
 v
Argo CD
 |
 v
External Secrets Controller
 |
 v
AWS Secrets Manager
 |
 v
Kubernetes Secret
```

Argo CD manages the declaration.

The external system supplies secret data.

---

# 33. Helm and Kubernetes Secret Template

A Helm chart may template a Secret:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cart
type: Opaque
stringData:
  DATABASE_URL: {{ .Values.database.url | quote }}
```

This is technically valid but can be unsafe if the value comes from Git.

The safer design is to reference an externally managed Secret.

---

# 34. Helm Security

Review:

```text
Template functions
Dependencies
Image references
RBAC resources
ServiceAccounts
SecurityContexts
Secrets
NetworkPolicies
```

A Helm chart can generate privileged Kubernetes resources.

Never assume:

```text
Helm chart = safe
```

---

# 35. Helm Chart Testing

Before Git merge:

```bash
helm lint ./helm/cart
helm template cart ./helm/cart \
  -f ./helm/cart/values/prod.yaml
```

Then validate rendered manifests.

CI should catch:

```text
Syntax errors
Missing values
Invalid templates
Unexpected resources
```

---

# 36. Helm Diff in CI

A diff-oriented workflow can compare:

```text
Current desired state
vs
new rendered state
```

Use tooling appropriate to your CI environment.

The goal is to review meaningful Kubernetes changes before production.

---

# 37. Kustomize Mental Model

Kustomize works differently from Helm.

Instead of:

```text
Template + values
```

it generally uses:

```text
Base resources
+
Overlays
+
Patches
```

Conceptually:

```text
Base
 |
 +--> Deployment
 +--> Service
 |
 v
Overlay
 |
 +--> dev changes
 |
 v
Rendered YAML
```

---

# 38. Kustomize Repository

Example:

```text
kustomize/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
│
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patch.yaml
    │
    ├── qa/
    │   ├── kustomization.yaml
    │   └── patch.yaml
    │
    └── prod/
        ├── kustomization.yaml
        └── patch.yaml
```

---

# 39. Base

Example:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
```

The base defines common configuration.

---

# 40. Development Overlay

Example:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

namespace: roboshop-dev

images:
  - name: cart
    newName: <account>.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
    newTag: "dev-123"
```

---

# 41. Production Overlay

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

namespace: roboshop

images:
  - name: cart
    newName: <account>.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
    newTag: "2026.08.19-abc123"
```

---

# 42. Kustomize Patches

Patches allow environment-specific changes.

Example:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
spec:
  replicas: 5
```

The overlay applies this patch to the base Deployment.

---

# 43. Strategic Merge vs JSON Patch

Kustomize supports different patch mechanisms.

Conceptually:

```text
Strategic merge
JSON patch
```

Choose the simplest mechanism that expresses the intended change.

Avoid giant patches that duplicate the entire resource.

---

# 44. Kustomize Images

Example:

```yaml
images:
  - name: cart
    newName: <account>.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
    newTag: "1.4.7"
```

This provides a convenient environment-specific image transformation.

---

# 45. Kustomize Name Prefix

Example:

```yaml
namePrefix: prod-
```

This can produce:

```text
prod-cart
```

Use naming transformations carefully when Services, selectors, or external integrations depend on stable names.

---

# 46. Kustomize Common Labels

Example:

```yaml
commonLabels:
  environment: prod
  team: roboshop
```

Labels can improve:

```text
Filtering
Monitoring
Ownership
Cost reporting
```

Be careful not to alter selector labels in a way that breaks workload relationships.

---

# 47. Kustomize Namespace

Example:

```yaml
namespace: roboshop
```

This can apply namespace configuration to namespaced resources.

Still ensure the Application destination and Kustomize configuration are aligned.

---

# 48. Kustomize Generators

Kustomize supports generators such as:

```text
ConfigMapGenerator
SecretGenerator
```

These can create generated resources.

Be careful with SecretGenerator in GitOps repositories because generated input data can still contain secrets.

---

# 49. Kustomize ConfigMap Generator

Example:

```yaml
configMapGenerator:
  - name: cart-config
    literals:
      - LOG_LEVEL=INFO
```

Kustomize may generate a name with a content hash.

This helps trigger rollouts when configuration changes.

---

# 50. Kustomize Secret Generator

Technically:

```yaml
secretGenerator:
  - name: cart-secret
    literals:
      - PASSWORD=example
```

Do not use plaintext production secrets in Git.

Use external secret management instead.

---

# 51. Kustomize Components

Components can provide reusable optional configuration.

Conceptually:

```text
Base
 |
 +--> Component A
 |
 +--> Component B
 |
 v
Overlay
```

Examples:

```text
Monitoring
NetworkPolicy
PodSecurity
```

Use components when they improve reuse without making the configuration difficult to understand.

---

# 52. Helm vs Kustomize

| Feature | Helm | Kustomize |
|---|---|---|
| Packaging | Strong | Minimal |
| Templating | Strong | Limited |
| Values | Strong | Overlay based |
| Reuse | Charts | Bases/components |
| Complexity | Can become high | Usually YAML-centric |
| Dependencies | Native | Resource composition |
| Ecosystem | Very large | Kubernetes-native |
| Argo CD integration | Strong | Strong |

---

# 53. When to Use Helm

Use Helm when:

```text
Many configurable parameters
Reusable application package
Chart distribution needed
Dependency management
Versioned packaging
```

For a reusable RoboShop service template, Helm is a strong choice.

---

# 54. When to Use Kustomize

Use Kustomize when:

```text
You already have Kubernetes YAML
Environment differences are patch-oriented
You want minimal templating
You prefer native Kubernetes configuration
```

---

# 55. When Not to Use Helm

Avoid Helm when the chart becomes:

```text
Hundreds of nested if statements
Difficult template logic
Hard-to-debug value combinations
```

If the chart is effectively a programming language, simplify it.

---

# 56. When Not to Use Kustomize

Avoid Kustomize overlays that become:

```text
Base
 |
 +--> patch
 +--> patch
 +--> patch
 +--> patch
 +--> patch
```

with many hidden interactions.

At that point, a structured packaging approach may be easier.

---

# 57. Hybrid Strategy

A production platform can use both.

Example:

```text
Third-party platform components
 |
 +--> Helm

Internal applications
 |
 +--> Helm

Small platform customization
 |
 +--> Kustomize
```

Argo CD supports both at the Application level.

---

# 58. Helm + Kustomize Together

A workflow can render Helm and then apply Kustomize-style processing in supported configurations.

Use this only when there is a real requirement.

Avoid unnecessary rendering layers:

```text
Helm
+
Kustomize
+
Plugin
+
Generator
```

can become difficult to troubleshoot.

---

# 59. Argo CD Source Detection

Argo CD determines how a source should be rendered based on the configured source and repository content/configuration.

Common source types:

```text
Directory/YAML
Helm
Kustomize
Plugin
```

Be explicit when production behavior needs to be obvious.

---

# 60. Explicit Helm Source

Example:

```yaml
source:
  repoURL: https://github.com/example/gitops.git
  targetRevision: main
  path: helm/cart
  helm:
    valueFiles:
      - values/prod.yaml
```

This makes the Application intent clear.

---

# 61. Explicit Kustomize Source

Example:

```yaml
source:
  repoURL: https://github.com/example/gitops.git
  targetRevision: main
  path: kustomize/overlays/prod
  kustomize:
    commonLabels:
      managed-by: argocd
```

Use only fields supported by the installed Argo CD version.

---

# 62. Production Git Repository Pattern: Helm

```text
gitops-repo/
├── charts/
│   ├── cart/
│   │   ├── Chart.yaml
│   │   ├── values.yaml
│   │   ├── values/
│   │   │   ├── dev.yaml
│   │   │   ├── qa.yaml
│   │   │   └── prod.yaml
│   │   └── templates/
│   ├── user/
│   └── payment/
│
├── applications/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
└── projects/
```

---

# 63. Production Git Repository Pattern: Kustomize

```text
gitops-repo/
├── kustomize/
│   ├── cart/
│   │   ├── base/
│   │   └── overlays/
│   │       ├── dev/
│   │       ├── qa/
│   │       └── prod/
│   ├── user/
│   └── payment/
│
├── applications/
└── projects/
```

---

# 64. Recommended RoboShop Helm Structure

```text
roboshop-gitops/
│
├── applications/
│   ├── dev/
│   │   ├── cart.yaml
│   │   ├── user.yaml
│   │   └── payment.yaml
│   │
│   ├── qa/
│   └── prod/
│
├── projects/
│   ├── dev.yaml
│   ├── qa.yaml
│   └── prod.yaml
│
└── helm/
    ├── cart/
    │   ├── Chart.yaml
    │   ├── values.yaml
    │   ├── values/
    │   │   ├── dev.yaml
    │   │   ├── qa.yaml
    │   │   └── prod.yaml
    │   └── templates/
    │
    ├── user/
    ├── payment/
    ├── catalogue/
    └── frontend/
```

---

# 65. Production Chart.yaml

```yaml
apiVersion: v2
name: cart
description: RoboShop cart microservice
type: application
version: 1.2.0
appVersion: "2026.08.19"
```

Use meaningful versioning.

---

# 66. Production values.yaml

```yaml
replicaCount: 2

image:
  repository: nginx
  tag: "stable"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 8080

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi

probes:
  readiness:
    path: /health
    port: 8080
  liveness:
    path: /health
    port: 8080
```

The default should be safe and predictable.

---

# 67. Production values/prod.yaml

```yaml
replicaCount: 3

image:
  repository: <account>.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
  tag: "2026.08.19-abc123"
  pullPolicy: IfNotPresent

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: 1000m
    memory: 1Gi
```

Use real account/registry values only in the actual repository.

---

# 68. Production Deployment Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "cart.fullname" . }}
  labels:
    app.kubernetes.io/name: {{ include "cart.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}

spec:
  replicas: {{ .Values.replicaCount }}

  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "cart.name" . }}
      app.kubernetes.io/instance: {{ .Release.Name }}

  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "cart.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}

    spec:
      containers:
        - name: cart
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}

          ports:
            - containerPort: 8080

          resources:
            {{- toYaml .Values.resources | nindent 12 }}

          readinessProbe:
            httpGet:
              path: {{ .Values.probes.readiness.path }}
              port: {{ .Values.probes.readiness.port }}
            initialDelaySeconds: 10
            periodSeconds: 10

          livenessProbe:
            httpGet:
              path: {{ .Values.probes.liveness.path }}
              port: {{ .Values.probes.liveness.port }}
            initialDelaySeconds: 30
            periodSeconds: 20
```

---

# 69. Production Service Template

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "cart.fullname" . }}

spec:
  type: {{ .Values.service.type }}

  selector:
    app.kubernetes.io/name: {{ include "cart.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}

  ports:
    - port: {{ .Values.service.port }}
      targetPort: 8080
      protocol: TCP
```

---

# 70. Production HPA Template

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "cart.fullname" . }}

spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "cart.fullname" . }}

  minReplicas: 3
  maxReplicas: 10

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

If HPA owns replicas, avoid conflicting Git-managed replica behavior.

---

# 71. Production ALB Ingress Template

The user's architecture uses AWS ALB Ingress rather than API Gateway.

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80}]'

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

The AWS Load Balancer Controller then manages the corresponding ALB.

---

# 72. ALB Ownership Model

```text
Git
 |
 v
Ingress spec
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

Argo CD does not directly create the ALB API resource.

The Kubernetes controller does.

---

# 73. SecurityContext

Production Helm templates should expose security settings.

Example:

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```

Not every application can use all settings immediately.

Test compatibility before enforcing them.

---

# 74. Pod Security

Use:

```text
runAsNonRoot
drop capabilities
readOnly filesystem where possible
seccomp
non-privileged containers
```

These controls should be part of production chart standards.

---

# 75. Resource Requests and Limits

Example:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: 1000m
    memory: 1Gi
```

Requests affect:

```text
Scheduling
Capacity planning
HPA behavior
```

Limits can protect nodes but should be set based on workload characteristics.

---

# 76. Helm Values Anti-Pattern

Avoid:

```yaml
everything:
  deeply:
    nested:
      with:
        dozens:
          of:
            switches:
```

If operators need to understand the chart by reading 500 lines of values, simplify the interface.

---

# 77. Kustomize Production Example

Base:

```text
kustomize/cart/base/
├── deployment.yaml
├── service.yaml
└── kustomization.yaml
```

Overlay:

```text
kustomize/cart/overlays/prod/
├── kustomization.yaml
└── patch.yaml
```

---

# 78. Base Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart

spec:
  replicas: 2

  selector:
    matchLabels:
      app: cart

  template:
    metadata:
      labels:
        app: cart

    spec:
      containers:
        - name: cart
          image: cart:base
          ports:
            - containerPort: 8080
```

---

# 79. Base Kustomization

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
```

---

# 80. Production Overlay

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

namespace: roboshop

images:
  - name: cart
    newName: <account>.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
    newTag: "2026.08.19-abc123"

patches:
  - path: patch.yaml
```

---

# 81. Production Patch

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart

spec:
  replicas: 3

  template:
    spec:
      containers:
        - name: cart
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: 1
              memory: 1Gi
```

---

# 82. Argo CD Kustomize Application

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
    path: kustomize/cart/overlays/prod

  destination:
    name: eks-prod
    namespace: roboshop

  syncPolicy:
    automated:
      selfHeal: true
      prune: true

    syncOptions:
      - CreateNamespace=true
```

---

# 83. Argo CD Helm Application

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
      releaseName: cart-prod
      valueFiles:
        - values/prod.yaml

  destination:
    name: eks-prod
    namespace: roboshop

  syncPolicy:
    automated:
      selfHeal: true
      prune: true

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

# 84. Helm Application Review

Check:

```text
repoURL
targetRevision
path
releaseName
valueFiles
destination
project
syncPolicy
```

These fields determine the deployment behavior.

---

# 85. Kustomize Application Review

Check:

```text
repoURL
targetRevision
path
overlay
destination
project
syncPolicy
```

The overlay determines environment-specific configuration.

---

# 86. Helm Environment Strategy

A simple model:

```text
Chart
 |
 +--> values.yaml
 |
 +--> dev.yaml
 +--> qa.yaml
 +--> prod.yaml
```

Application:

```text
DEV -> dev.yaml
QA  -> qa.yaml
PROD -> prod.yaml
```

---

# 87. Kustomize Environment Strategy

```text
base
 |
 +--> dev overlay
 +--> qa overlay
 +--> prod overlay
```

This makes the environment difference explicit in the directory structure.

---

# 88. Helm Multi-Cluster Strategy

One chart can deploy to multiple clusters:

```text
Helm chart
 |
 +--> EKS DEV
 +--> EKS QA
 +--> EKS PROD
```

Different Applications/ApplicationSets select:

```text
Cluster
Values
Namespace
```

---

# 89. Kustomize Multi-Cluster Strategy

Use overlays or generated Applications:

```text
base
 |
 +--> prod-ap-south-1
 +--> prod-ap-southeast-1
```

However, avoid creating an overlay for every tiny difference.

Use ApplicationSet and cluster labels when the deployment model scales.

---

# 90. Environment-Specific Image Strategy

Bad:

```text
dev -> cart:latest
qa  -> cart:latest
prod -> cart:latest
```

Better:

```text
dev -> immutable version
qa  -> same immutable version
prod -> approved same immutable version
```

Example:

```text
cart@sha256:abc...
```

---

# 91. Promotion Model

```text
Build once
 |
 v
ECR image
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

The GitOps repository records the promotion.

This reduces the chance of:

```text
"QA tested a different image than production."
```

---

# 92. Helm Rollback

Helm has its own release concepts, but in Argo CD GitOps the preferred rollback mechanism is usually:

```text
Git revert
```

rather than independently running:

```bash
helm rollback
```

against a cluster managed by Argo CD.

Otherwise Git and live state can diverge.

---

# 93. Kustomize Rollback

Kustomize has no separate release history equivalent to Helm.

Rollback is naturally:

```text
Git revert
 |
 v
Previous rendered state
 |
 v
Argo CD
```

This is one reason Git history is central to Kustomize-based GitOps.

---

# 94. Helm History vs Argo CD History

Helm may maintain release metadata.

Argo CD maintains Application synchronization history.

When Argo CD is the deployment controller:

```text
Argo CD Application history
+
Git history
```

should be the primary operational audit trail.

---

# 95. Production Chart Repository Ownership

A mature organization may separate:

```text
Application source repository
```

from:

```text
GitOps configuration repository
```

Example:

```text
roboshop-cart-source
        |
        v
CI
        |
        v
ECR

roboshop-gitops
        |
        v
Argo CD
```

This cleanly separates:

```text
Application code
Deployment configuration
```

---

# 96. GitOps Repository vs Chart Repository

They can be:

```text
Same repository
```

or:

```text
Separate repositories
```

Both are valid.

Choose based on:

```text
Ownership
Release model
Security
Team boundaries
Repository scale
```

---

# 97. Separate Helm Chart Repository

Example:

```text
helm-charts/
└── cart/
```

GitOps:

```text
gitops/
└── applications/
```

Argo CD can reference the chart source according to the supported repository configuration.

---

# 98. Advantages of Separate Chart Repository

```text
Reusable charts
Versioned packages
Central standards
Multiple consuming teams
```

But it introduces:

```text
Dependency management
Chart release process
Version coordination
```

---

# 99. Application Source and Values Separation

A sophisticated model:

```text
Application chart
       |
       v
Chart repository

Environment values
       |
       v
GitOps repository
```

Argo CD can combine sources in supported configurations.

This can work well for platform-scale environments but increases complexity.

---

# 100. Avoid Overengineering

Start with:

```text
GitOps repo
 |
 +--> Applications
 +--> Helm charts
 +--> Values
```

Move to multi-repository/multi-source architecture only when the organization needs it.

---

# 101. Argo CD Manifest Generation Troubleshooting

When a Helm Application fails before synchronization:

```bash
argocd app get <app>
```

Look for:

```text
Manifest generation error
```

Then check:

```text
Chart
Values
Dependencies
Repository
Helm version compatibility
Plugins
```

---

# 102. Local Helm Reproduction

Clone the repository:

```bash
git clone <gitops-repository>
cd <gitops-repository>
```

Render:

```bash
helm template cart ./helm/cart \
  -f ./helm/cart/values/prod.yaml
```

If this fails locally:

```text
Fix Helm first.
```

---

# 103. Helm Lint

Run:

```bash
helm lint ./helm/cart
```

This catches common chart issues.

CI should run linting before merge.

---

# 104. Helm Template Validation

Render:

```bash
helm template cart ./helm/cart \
  -f ./helm/cart/values/prod.yaml \
  > /tmp/cart-rendered.yaml
```

Then inspect:

```bash
kubectl apply --dry-run=server \
  -f /tmp/cart-rendered.yaml
```

Use appropriate Kubernetes credentials/context for validation.

---

# 105. Kustomize Build

Run:

```bash
kustomize build ./kustomize/cart/overlays/prod
```

or:

```bash
kubectl kustomize ./kustomize/cart/overlays/prod
```

Then validate the generated manifests.

---

# 106. Kustomize Troubleshooting

If an Application fails:

```text
Check overlay path
Check base path
Check patch target
Check resource names
Check image transformation
Check namespace
```

Then reproduce locally.

---

# 107. Helm Value File Missing

Error pattern:

```text
values/prod.yaml not found
```

Check:

```bash
find helm/cart -maxdepth 3 -type f
```

Verify:

```yaml
valueFiles:
  - values/prod.yaml
```

matches the repository path.

---

# 108. Helm Template Nil Value

Example:

```text
can't evaluate field X in type interface {}
```

Usually means:

```text
Expected value missing
```

Check:

```text
values.yaml
prod.yaml
template
```

Use defaults where appropriate:

```gotemplate
{{ .Values.someValue | default "value" }}
```

Do not hide critical configuration errors with excessive defaults.

---

# 109. Helm Conditional Resource

Example:

```gotemplate
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
...
{{- end }}
```

This lets environments selectively enable resources.

Use this carefully.

Too many conditionals can make the chart difficult to reason about.

---

# 110. Helm Environment Feature Flags

Example:

```yaml
ingress:
  enabled: true
```

Dev:

```yaml
ingress:
  enabled: false
```

Prod:

```yaml
ingress:
  enabled: true
```

Document feature flags clearly.

---

# 111. Kustomize Feature Differences

Kustomize can express environment differences with:

```text
Resources
Patches
Components
Images
Namespace
Labels
Annotations
```

This can be easier to understand than many Helm conditionals.

---

# 112. Helm and CRDs

Some Helm charts include CRDs.

CRD lifecycle requires special attention.

Questions:

```text
Who owns the CRD?
When is it installed?
How is it upgraded?
Can it be safely removed?
```

Do not treat CRDs like ordinary Deployments.

---

# 113. Kustomize and CRDs

Kustomize can also deploy CRDs.

Again:

```text
CRD ownership
+
Ordering
+
Upgrade compatibility
```

must be planned.

---

# 114. Production Platform Separation

A useful structure:

```text
platform/
 |
 +--> CRDs
 +--> Controllers
 +--> ALB Controller
 +--> External Secrets
 +--> Monitoring

applications/
 |
 +--> RoboShop
 +--> Other business workloads
```

Argo CD Projects can enforce this separation.

---

# 115. Helm Third-Party Controllers

For example:

```text
AWS Load Balancer Controller
```

may be installed using Helm.

Argo CD can manage that Helm chart.

Architecture:

```text
Git
 |
 v
Argo CD
 |
 v
Helm chart
 |
 v
AWS Load Balancer Controller
 |
 v
AWS APIs
```

The controller then manages ALBs.

---

# 116. Third-Party Chart Pinning

Do not blindly track:

```text
latest chart
```

Prefer:

```text
Pinned chart version
+
Reviewed values
```

This prevents unexpected upstream changes.

---

# 117. Helm Dependency Updates

When a dependency changes:

```text
Review chart
Review rendered resources
Review permissions
Review images
Run security scans
Test in dev
Promote
```

Do not update production dependencies casually.

---

# 118. Argo CD and Helm Plugins

Some organizations use Helm plugins for specialized rendering.

This introduces additional security and operational complexity.

Before using plugins:

```text
Verify source
Verify version
Verify reproducibility
Verify permissions
Verify build environment
```

Plugins should not become hidden deployment logic.

---

# 119. Reproducibility

A production GitOps render should be reproducible.

Given:

```text
Git revision
Chart version
Values
Dependencies
Tool versions
```

you should be able to produce the same intended manifests.

This is essential for:

```text
Auditing
Incident response
Rollback
Disaster recovery
```

---

# 120. Pin Dependencies

For reproducibility:

```text
Chart version
Image version/digest
Dependency version
Git revision
```

should be controlled.

Avoid:

```text
latest
unbounded ranges
floating dependencies
```

for critical production artifacts.

---

# 121. Helm Chart Security Scanning

CI should scan:

```text
Rendered manifests
Chart dependencies
Container images
Configuration
```

The user's DevSecOps stack includes:

```text
SonarQube
Trivy
Veracode
```

Use the appropriate tool at the appropriate stage.

---

# 122. Trivy in Helm Workflow

Example:

```text
Helm chart
 |
 v
Render manifests
 |
 v
Trivy configuration scan
 |
 v
Build/publish image
 |
 v
Trivy image scan
```

This helps catch:

```text
Misconfiguration
Vulnerabilities
Exposed secrets
```

depending on the configured scanners.

---

# 123. SonarQube Scope

SonarQube is primarily useful for:

```text
Application source code quality/security
```

It should not be treated as the only Kubernetes configuration security tool.

---

# 124. Veracode Scope

Veracode can provide application security testing depending on the organization's integration and licensed capabilities.

The GitOps workflow should keep:

```text
Application security
+
Infrastructure/configuration security
```

as complementary controls.

---

# 125. GitOps CI Integration

A practical pipeline:

```text
Developer
 |
 v
Source Git
 |
 v
Jenkins/GitHub Actions
 |
 +--> Build
 +--> Unit Test
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
Update Helm values
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
```

---

# 126. Image Tag Update

Example:

```yaml
image:
  repository: <account>.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
  tag: "2026.08.19-abc123"
```

CI updates only the intended field.

Do not rewrite the whole environment configuration unnecessarily.

---

# 127. GitOps Commit Message

Good:

```text
promote cart to 2026.08.19-abc123 in prod
```

This makes deployment history searchable.

---

# 128. GitOps Pull Request

PR should show:

```diff
- tag: "2026.08.18-old"
+ tag: "2026.08.19-abc123"
```

Reviewers immediately see the deployment change.

This is far safer than:

```text
kubectl set image
```

with no Git record.

---

# 129. Helm Values and Secrets

Good:

```yaml
database:
  host: db.internal
```

Bad:

```yaml
database:
  password: "ProductionPassword123"
```

Instead:

```yaml
database:
  secretName: cart-db
```

and let an external secret mechanism provide the data.

---

# 130. Kustomize and Secrets

Similarly:

```text
secretGenerator
```

does not automatically make secret values safe.

If source input contains:

```text
plaintext secret
```

the Git repository still contains sensitive data.

Use an appropriate secret-management architecture.

---

# 131. Production Secret Architecture

For AWS:

```text
AWS Secrets Manager
        |
        v
External Secrets Controller
        |
        v
Kubernetes Secret
        |
        v
RoboShop Pod
```

Git contains:

```text
ExternalSecret definition
```

not the secret value.

---

# 132. Helm Chart Security Defaults

A production chart should expose:

```yaml
securityContext:
  runAsNonRoot: true

containerSecurityContext:
  allowPrivilegeEscalation: false
```

But templates should match actual application needs.

Do not enable settings blindly if the application requires privileged behavior.

---

# 133. NetworkPolicy

A production chart can include:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
```

Use Helm values to enable it where appropriate.

For example:

```yaml
networkPolicy:
  enabled: true
```

---

# 134. PodDisruptionBudget

Production applications may need:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
```

This protects availability during:

```text
Node maintenance
Cluster upgrades
Voluntary disruptions
```

Do not use PDBs with impossible availability requirements.

---

# 135. Topology Spread

Production workloads can use:

```yaml
topologySpreadConstraints:
```

to distribute Pods.

This can improve:

```text
Availability
Failure-domain resilience
```

Use constraints compatible with actual EKS node topology.

---

# 136. Affinity and Anti-Affinity

Helm values can expose:

```yaml
affinity:
```

or:

```yaml
podAntiAffinity:
```

This should be designed carefully because strict scheduling rules can cause:

```text
Pending Pods
```

---

# 137. Production Chart Validation

Before merge:

```text
helm lint
helm template
kubectl server-side dry-run
security scan
policy validation
```

Then:

```text
Deploy DEV
Run tests
Promote QA
Promote PROD
```

---

# 138. Policy Validation

A production platform can validate rendered manifests for:

```text
Non-root containers
Required resource requests
Approved registries
No privileged containers
Required labels
Approved namespaces
```

Use policy tooling appropriate to the platform.

---

# 139. Helm and Kustomize in ApplicationSet

ApplicationSet can generate Applications that point to:

```text
helm/<service>
```

or:

```text
kustomize/overlays/<environment>
```

This enables:

```text
One generator
+
Many Applications
```

ApplicationSets are covered in depth in the next major file.

---

# 140. Multi-Environment Helm Example

ApplicationSet concept:

```text
dev -> values/dev.yaml
qa  -> values/qa.yaml
prod -> values/prod.yaml
```

Generated Applications:

```text
cart-dev
cart-qa
cart-prod
```

---

# 141. Multi-Cluster Helm Example

```text
Cluster label:
environment=prod

ApplicationSet
 |
 v
Select clusters
 |
 v
Application
 |
 v
Helm chart
 |
 v
EKS
```

The chart remains reusable.

---

# 142. Kustomize Multi-Cluster Example

```text
base
 |
 +--> overlays/prod
```

ApplicationSet can generate:

```text
prod-cluster-a
prod-cluster-b
```

using the same overlay when the clusters share the same desired configuration.

---

# 143. Cluster-Specific Differences

If clusters differ in:

```text
Region
ALB hostname
Storage class
Node architecture
External endpoints
```

you may need:

```text
cluster-specific values
```

or:

```text
cluster-specific overlays
```

Do not force identical configuration where the infrastructure genuinely differs.

---

# 144. Architecture: Helm-Based RoboShop

```text
                    GitOps Repository
                           |
             +-------------+-------------+
             |                           |
        Application                  Helm Chart
             |                           |
             +-------------+-------------+
                           |
                           v
                      Argo CD
                           |
                           v
                   Helm Rendering
                           |
                           v
                Kubernetes Manifests
                           |
                           v
                         EKS
                           |
             +-------------+-------------+
             |             |             |
          Deployment     Service       Ingress
                                         |
                                         v
                                       ALB
```

---

# 145. Architecture: Kustomize-Based RoboShop

```text
                 GitOps Repository
                        |
                        v
                     Base YAML
                        |
                        v
                     Overlay
                   /    |    \
                 dev    qa    prod
                        |
                        v
                      Argo CD
                        |
                        v
                    EKS
```

---

# 146. Helm vs Kustomize Decision for RoboShop

For the user's RoboShop microservices:

```text
Helm
```

is a strong choice when each microservice follows a common deployment pattern.

For example:

```text
cart
user
payment
catalogue
frontend
```

can reuse:

```text
Deployment
Service
HPA
Probes
Resources
SecurityContext
```

with different values.

---

# 147. Generic Helm Chart for Microservices

A standardized internal chart could expose:

```yaml
image:
replicaCount:
service:
ingress:
resources:
probes:
env:
config:
autoscaling:
securityContext:
podSecurityContext:
nodeSelector:
tolerations:
affinity:
```

This allows teams to deploy consistently.

---

# 148. Generic Chart Risk

A generic chart can become too complex.

Avoid:

```text
One chart containing every possible Kubernetes feature
```

Instead:

```text
Common defaults
+
Optional features
+
Clear documentation
```

---

# 149. Platform Chart vs Application Chart

Platform chart:

```text
ALB Controller
External Secrets
Monitoring
```

Application chart:

```text
RoboShop cart
RoboShop user
RoboShop payment
```

Keep responsibilities separate.

---

# 150. Production Upgrade Strategy

For Helm charts:

```text
Change chart
 |
 v
Render
 |
 v
Test DEV
 |
 v
QA
 |
 v
Production
```

For Kustomize:

```text
Change base/overlay
 |
 v
Render
 |
 v
Test
 |
 v
Promotion
```

Git history should record each promotion.

---

# 151. Chart Upgrade Rollback

If a chart update breaks production:

```text
Git revert
 |
 v
Previous chart/version/config
 |
 v
Argo CD
 |
 v
Kubernetes
```

This is generally safer than mixing:

```text
helm rollback
+
Argo CD
```

without reconciling Git.

---

# 152. Kustomize Rollback

If an overlay change breaks production:

```text
Git revert
 |
 v
Argo CD
 |
 v
Previous manifests
```

The rollback is represented in Git.

---

# 153. GitOps Principle: Configuration Is Code

Helm:

```text
Chart + values
```

Kustomize:

```text
Base + overlays
```

Both should be treated like code:

```text
Review
Test
Version
Scan
Audit
```

---

# 154. Pull Request Validation

For Helm:

```bash
helm lint
helm template
```

For Kustomize:

```bash
kustomize build
```

For Kubernetes:

```bash
kubectl apply --dry-run=server
```

Then policy/security validation.

---

# 155. Manifest Size

Very large rendered manifests can increase:

```text
Repo Server CPU
Comparison cost
Application Controller load
Review complexity
```

Keep Applications reasonably scoped.

---

# 156. Large Helm Charts

If one chart deploys:

```text
50 microservices
```

ask whether:

```text
Independent Applications
```

would provide better:

```text
Ownership
Rollbacks
Scaling
Troubleshooting
```

---

# 157. One Chart Per Microservice

Often appropriate for microservices:

```text
charts/
├── cart
├── user
├── payment
└── catalogue
```

This provides independent lifecycle management.

---

# 158. Umbrella Chart

An umbrella chart can combine multiple subcharts.

Example:

```text
roboshop/
 |
 +--> cart
 +--> user
 +--> payment
 +--> catalogue
```

Advantages:

```text
Single deployment
Central versioning
```

Disadvantages:

```text
Large blast radius
Coupled releases
Complex upgrades
```

For independently released microservices, separate Applications may be better.

---

# 159. Argo CD Application per Helm Release

Conceptually:

```text
cart Application
    |
    v
cart Helm chart

user Application
    |
    v
user Helm chart
```

This is easy to operate.

---

# 160. ApplicationSet for Repeated Helm Applications

At scale:

```text
ApplicationSet
 |
 +--> cart
 +--> user
 +--> payment
 +--> catalogue
```

The generator can produce Applications using a common template.

This reduces repetitive Application YAML.

---

# 161. Production Troubleshooting Matrix

| Symptom | First Check | Likely Area |
|---|---|---|
| Manifest generation failed | `argocd app get` | Helm/Kustomize |
| OutOfSync | `argocd app diff` | Drift |
| Sync failed | Resource error | Kubernetes/RBAC |
| Synced but Degraded | Pods/events | Runtime |
| Helm value ignored | Rendered output | Values precedence |
| Kustomize patch failed | `kustomize build` | Overlay |
| Wrong image | Rendered manifest | Values/images |
| Wrong namespace | Destination/overlay | Namespace config |
| HPA conflict | Diff | Ownership |
| Old resource remains | Prune | Lifecycle |
| Resource unexpectedly deleted | Git history | Prune/ownership |

---

# 162. Troubleshooting: Helm Image Not Updated

Check:

```bash
argocd app diff <app>
```

If image did not change:

```text
Check valueFiles
Check image.tag
Check Helm parameter
Check generated values
```

Then render locally:

```bash
helm template ...
```

---

# 163. Troubleshooting: Kustomize Image Not Updated

Check:

```yaml
images:
  - name: cart
    newName: ...
    newTag: ...
```

The `name` must match the image reference used in the base in a way Kustomize can transform.

Then:

```bash
kustomize build overlays/prod
```

---

# 164. Troubleshooting: Helm Dependency Missing

Possible causes:

```text
Dependency not downloaded
Repository unavailable
Version unavailable
Chart.lock mismatch
```

Verify dependency configuration and repository access.

---

# 165. Troubleshooting: Kustomize Base Not Found

Example:

```text
../../base
```

may be wrong.

From the overlay directory:

```bash
pwd
ls -la
ls ../../base
```

Reproduce with:

```bash
kustomize build .
```

---

# 166. Troubleshooting: Values File Not Committed

A frequent GitOps problem:

```text
Application references:
values/prod.yaml

Git branch:
file absent
```

Result:

```text
Manifest generation failure
```

Always verify repository content at the target revision.

---

# 167. Troubleshooting: Wrong Git Revision

The Application may track:

```yaml
targetRevision: main
```

while the desired values are on:

```text
release/prod
```

Check:

```bash
argocd app get <app>
```

Then inspect the configured revision.

---

# 168. Troubleshooting: Chart Defaults Override Environment

If production unexpectedly receives:

```text
replicas=2
```

instead of:

```text
replicas=5
```

inspect:

```text
valueFiles
valuesObject
parameters
chart defaults
```

Then render the chart locally.

---

# 169. Troubleshooting: Secret Template Error

If Helm rendering fails because a secret value is absent:

```text
Do not solve by committing the production secret.
```

Instead verify:

```text
External secret architecture
Secret reference
Controller
Namespace
RBAC
```

---

# 170. Troubleshooting: Rendered YAML Valid but Sync Fails

Then the issue is likely after rendering:

```text
Kubernetes API
RBAC
Admission
Resource conflict
Dependency
```

Move troubleshooting from:

```text
Helm/Kustomize
```

to:

```text
Kubernetes
```

---

# 171. Production Validation Pipeline

A strong pipeline:

```text
Git PR
 |
 +--> YAML validation
 |
 +--> Helm lint
 |
 +--> Helm template
 |
 +--> Kustomize build
 |
 +--> Kubernetes schema validation
 |
 +--> Security scan
 |
 +--> Policy checks
 |
 v
Review
 |
 v
Merge
```

---

# 172. Production Promotion Pipeline

```text
Source Git
 |
 v
CI
 |
 v
ECR
 |
 v
GitOps DEV
 |
 v
Argo CD DEV
 |
 v
Tests
 |
 v
GitOps QA
 |
 v
Argo CD QA
 |
 v
Approval
 |
 v
GitOps PROD
 |
 v
Argo CD PROD
```

---

# 173. No Direct Production kubectl

For normal deployment changes:

```text
Do not:
kubectl edit
kubectl set image
kubectl scale
```

Instead:

```text
Change Git
```

Emergency procedures are exceptions and must be documented.

---

# 174. Helm and Terraform Boundary

Terraform can provision:

```text
EKS
VPC
IAM
ALB infrastructure where appropriate
ECR
RDS
S3
```

Argo CD/Helm can manage:

```text
Kubernetes Deployments
Services
Ingress
HPA
ConfigMaps
Application-level resources
```

Avoid managing the same object through both.

---

# 175. Production Architecture With Terraform

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
AWS infrastructure

GitOps
 |
 +--> Helm/Kustomize
 |
 v
Argo CD
 |
 v
EKS workloads
```

This gives a clear infrastructure/application boundary.

---

# 176. Argo CD + Helm + ECR

```text
Developer
 |
 v
Git
 |
 v
Jenkins/GitHub Actions
 |
 +--> Build
 +--> Test
 +--> Security
 |
 v
Docker image
 |
 v
ECR
 |
 v
Update Helm values
 |
 v
GitOps repo
 |
 v
Argo CD
 |
 v
EKS
```

---

# 177. Production Security Checklist

### Helm

```text
[ ] Dependencies pinned
[ ] Chart reviewed
[ ] Templates scanned
[ ] No secrets in values
[ ] Images pinned
[ ] SecurityContext configured
[ ] Resource limits reviewed
```

### Kustomize

```text
[ ] Bases reviewed
[ ] Overlays reviewed
[ ] Patches minimized
[ ] No secrets in Git
[ ] Image references controlled
```

### Argo CD

```text
[ ] Project restricted
[ ] Repository restricted
[ ] Destination restricted
[ ] RBAC configured
[ ] Production sync controlled
```

---

# 178. Interview Questions

## Q1. How does Argo CD use Helm?

### Answer

> Argo CD uses Helm primarily for manifest generation. It takes the chart, selected values and other source configuration, renders the chart into Kubernetes manifests, compares those manifests with live cluster state, and then reconciles the resulting resources. Argo CD remains the GitOps reconciliation controller.

---

## Q2. Does Argo CD replace Helm?

### Answer

> No. Helm and Argo CD solve different problems. Helm provides packaging and templating, while Argo CD provides GitOps synchronization and reconciliation. They are commonly used together.

---

## Q3. What is the difference between Helm and Kustomize?

### Answer

> Helm is a package manager and templating system using charts and values. Kustomize is Kubernetes-native configuration composition using bases, overlays and patches. Helm is often stronger for reusable application packages, while Kustomize is often simpler when starting from native YAML and applying environment-specific changes.

---

## Q4. Why use valueFiles?

### Answer

> Value files allow the same Helm chart to be reused across environments while keeping environment-specific configuration separate. For example, dev, QA and production can use the same cart chart with different values files.

---

## Q5. Why should production images be immutable?

### Answer

> Mutable tags such as `latest` can point to different images over time. Immutable tags or digests make deployments reproducible, auditable and safer to roll back.

---

## Q6. What happens if Helm rendering fails?

### Answer

> Argo CD cannot generate the desired Kubernetes manifests, so synchronization cannot proceed normally. I would inspect the Application error, verify the chart and values, and reproduce the rendering with `helm template` locally or in CI.

---

## Q7. How do you troubleshoot a Kustomize failure?

### Answer

> I reproduce the overlay locally using `kustomize build` or `kubectl kustomize`, then inspect the base path, patches, resource names, image transformations and namespace configuration. Once the manifests render correctly, I investigate Kubernetes-side errors if synchronization still fails.

---

## Q8. Can Helm and Kustomize be used together?

### Answer

> Yes, Argo CD supports different manifest-generation approaches and supported combinations can be used when there is a real requirement. However, I avoid unnecessary rendering layers because they increase complexity and troubleshooting difficulty.

---

## Q9. Where should secrets be stored?

### Answer

> I would not store plaintext production secrets in Helm values or Kustomize files. In AWS/EKS I would typically use AWS Secrets Manager with an external secret mechanism, so Git stores the declaration/reference while the secret value remains outside Git.

---

## Q10. What is the best rollback approach in a GitOps environment?

### Answer

> Normally I revert the Git change so the rollback itself becomes the new desired state. This preserves auditability and keeps Git and Kubernetes consistent. Direct Helm rollback or Argo CD rollback can be useful in emergencies, but the final desired state should be reconciled back into Git.

---

# 179. Scenario Interview: Helm Chart Works Locally but Argo CD Fails

Answer:

> I would compare the exact repository revision, Helm version, values files, dependencies and rendering environment. I would inspect the Repo Server error and reproduce the chart using the same chart path and values. If local and Argo environments differ, I would eliminate the environmental difference rather than changing the Application blindly.

---

# 180. Scenario Interview: Production Has Wrong Image

Response:

```text
1. argocd app diff
2. Inspect rendered manifest
3. Check targetRevision
4. Check values/prod.yaml
5. Check image parameters
6. Check Git commit
7. Check ECR image existence
8. Reconcile
```

Do not directly change the Deployment unless responding to an emergency.

---

# 181. Scenario Interview: HPA and Argo CD Keep Fighting

Answer:

> I would identify ownership of the replica field. HPA should own runtime replica scaling while Git owns the Deployment specification. I would remove conflicting fixed replica configuration where appropriate or configure a narrowly scoped ignore difference for the HPA-controlled field.

---

# 182. Scenario Interview: Why Not Put Every Environment in One values.yaml?

Answer:

> It becomes harder to review environment-specific changes and increases the risk of accidentally deploying production configuration to another environment. Separate values files make environment differences explicit and easier to control through Git review.

---

# 183. Scenario Interview: Why Not Maintain Separate Helm Charts for Dev and Prod?

Answer:

> If the application architecture is the same, separate charts duplicate templates and create maintenance drift. I prefer one reusable chart with environment-specific values. Separate charts are justified when the environments genuinely have different application architecture rather than just different configuration.

---

# 184. Scenario Interview: Helm Chart Upgrade Broke Production

Response:

```text
1. Stop further promotion.
2. Inspect Git diff.
3. Inspect rendered manifests.
4. Compare chart versions.
5. Check Argo CD Application history.
6. Revert Git to known-good chart/config.
7. Verify health.
8. Analyze root cause.
9. Add regression test.
```

---

# 185. Scenario Interview: Kustomize Patch Changed Wrong Resource

Possible cause:

```text
Patch target too broad
```

Fix:

```text
Use explicit group
Use explicit version
Use explicit kind
Use explicit name
```

Example:

```yaml
target:
  group: apps
  version: v1
  kind: Deployment
  name: cart
```

Specific targeting reduces accidental modifications.

---

# 186. Scenario Interview: Why Use Kustomize Over Helm?

Answer:

> If I already have clean Kubernetes manifests and environment differences are mostly patches or overlays, Kustomize can be simpler because it avoids introducing a templating language. I would use Helm when reusable packaging, values-driven configuration and chart distribution provide more value.

---

# 187. Scenario Interview: Why Use Helm Over Kustomize?

Answer:

> For many reusable microservices with common deployment patterns, Helm provides a strong packaging and templating model. I can create a standard chart with configurable image, resources, probes, service, ingress and autoscaling settings and reuse it across RoboShop services and environments.

---

# 188. Final Mental Model

Remember:

```text
HELM
=
Chart + Values + Templates
        |
        v
Rendered Kubernetes YAML

KUSTOMIZE
=
Base + Overlay + Patch
        |
        v
Rendered Kubernetes YAML

ARGO CD
=
Git + Rendered Desired State
        |
        v
Compare + Sync + Reconcile
```

---

# 189. Key Takeaways

1. Helm and Kustomize are manifest/configuration mechanisms.
2. Argo CD is the GitOps reconciliation controller.
3. Helm packages and templates Kubernetes resources.
4. Kustomize composes native Kubernetes YAML.
5. One reusable Helm chart can serve multiple environments.
6. Kustomize bases and overlays can express environment differences.
7. Production image references should be immutable.
8. Git should remain the deployment source of truth.
9. Avoid running Helm CLI deployments against resources managed by Argo CD.
10. Keep secrets outside normal Git values files.
11. Pin chart and dependency versions.
12. Render and validate manifests before merge.
13. Use CI for Helm linting and Kustomize builds.
14. Use Argo CD for reconciliation.
15. HPA/controller-owned fields require explicit ownership design.
16. CRDs require special lifecycle planning.
17. Third-party Helm charts must be reviewed and pinned.
18. Terraform and Argo CD should have separate ownership boundaries.
19. Helm is often a strong fit for reusable RoboShop microservice charts.
20. Kustomize is often a strong fit for native YAML overlays.
21. The simplest configuration model that meets the requirement is usually the best production model.

---

# 190. Next File

```text
09-ArgoCD-ApplicationSets.md
```

The next file will deeply cover ApplicationSets:

- Why ApplicationSet exists
- Application vs ApplicationSet
- ApplicationSet controller
- List generator
- Git generator
- Directory generator
- Cluster generator
- Matrix generator
- Merge generator
- Pull Request generator
- SCM generators
- Templates
- Dynamic application names
- Dynamic namespaces
- Dynamic clusters
- Dynamic Helm values
- Dev/QA/Prod list-generator example
- Multi-cluster EKS deployment
- Cluster labels
- Centralized Argo CD
- ApplicationSet security
- Production repository structures
- Production YAMLs
- RoboShop ApplicationSets
- Troubleshooting generated Applications
- Interview questions
