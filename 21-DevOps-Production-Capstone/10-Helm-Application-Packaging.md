# 10 — Helm Application Packaging

## Production DevOps Capstone

> Purpose: Build a production-grade Helm packaging model for Kubernetes applications running on Amazon EKS and deployed through GitOps with Argo CD.

---

# 1. Chapter Objectives

By the end of this chapter you should be able to:

1. Explain why Helm is used in production Kubernetes platforms.
2. Design a maintainable Helm chart structure.
3. Build reusable Kubernetes templates.
4. Separate application defaults from environment-specific configuration.
5. Manage Deployments, Services, Ingress, HPA, PDB, ServiceAccount and ConfigMaps through Helm.
6. Use Helm values safely without creating configuration chaos.
7. Implement helper templates and naming conventions.
8. Manage Helm dependencies.
9. Validate, lint and test charts before deployment.
10. Package and version charts.
11. Store charts in OCI registries such as Amazon ECR.
12. Deploy Helm charts through Argo CD.
13. Understand Helm release state and rollback behavior.
14. Secure Helm charts and prevent secret leakage.
15. Design production-ready charts for multiple environments.
16. Troubleshoot Helm rendering and deployment failures.
17. Explain Helm architecture confidently in DevOps interviews.

---

# 2. Helm in the Production Platform

Helm is the package manager commonly used to package Kubernetes application resources.

Instead of maintaining large numbers of independent Kubernetes YAML files, an application can be represented as a versioned chart.

Typical flow:

Developer
    |
    v
Application source
    |
    v
CI pipeline
    |
    +--> build image
    +--> security scan
    +--> push image
    |
    v
Helm chart / GitOps configuration
    |
    v
Argo CD
    |
    v
EKS
    |
    +--> Deployment
    +--> Service
    +--> Ingress
    +--> HPA
    +--> PDB
    +--> ConfigMap
    +--> ServiceAccount
    +--> NetworkPolicy

Helm does not replace Kubernetes.

Helm generates and manages Kubernetes manifests.

---

# 3. Why Production Teams Use Helm

Without Helm, an application can require:

- deployment.yaml
- service.yaml
- ingress.yaml
- configmap.yaml
- serviceaccount.yaml
- hpa.yaml
- pdb.yaml
- networkpolicy.yaml
- secret configuration
- multiple environment copies

For dev, staging and production, this can quickly become duplicated YAML.

Helm introduces parameterization.

Example:

```yaml
replicaCount: 3

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
  tag: "1.4.2"

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

The same chart can be used with different values.

---

# 4. Helm Core Concepts

Important Helm concepts:

| Concept | Meaning |
|---|---|
| Chart | Kubernetes application package |
| Release | Installed instance of a chart |
| Repository | Location containing charts |
| Values | Configuration supplied to templates |
| Template | Dynamic Kubernetes manifest |
| Hook | Lifecycle-triggered resource/action |
| Dependency | Another chart required by the application |
| Chart version | Version of packaging/templates |
| App version | Version of the application |
| Helm CLI | Command-line client |

Example:

```text
Chart
  |
  +-- templates/
  +-- values.yaml
  +-- Chart.yaml
  +-- charts/
  +-- helpers
```

---

# 5. Helm Chart Anatomy

Recommended production structure:

```text
roboshop-cart/
├── Chart.yaml
├── values.yaml
├── values.schema.json
├── README.md
├── .helmignore
├── templates/
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── configmap.yaml
│   ├── serviceaccount.yaml
│   ├── hpa.yaml
│   ├── pdb.yaml
│   ├── networkpolicy.yaml
│   ├── tests/
│   │   └── test-connection.yaml
│   └── NOTES.txt
├── charts/
└── ci/
    ├── dev-values.yaml
    ├── staging-values.yaml
    └── production-values.yaml
```

The exact structure should reflect application complexity.

Do not create unnecessary templates simply to make the chart look sophisticated.

---

# 6. Chart.yaml

Example:

```yaml
apiVersion: v2

name: roboshop-cart

description: Production Helm chart for RoboShop cart service

type: application

version: 1.3.0

appVersion: "2.1.0"

home: https://example.internal

keywords:
  - roboshop
  - cart
  - kubernetes
  - eks

maintainers:
  - name: platform-team
    email: platform@example.internal
```

Important distinction:

`version` is the Helm chart version.

`appVersion` identifies the application version.

They are not interchangeable.

---

# 7. Chart Versioning

Suppose:

```text
Application version = 2.1.0
Chart version = 1.5.0
```

A chart template change can require a chart version bump even when the application image did not change.

Example:

```text
Chart 1.4.0
  image: cart:2.0.0

Chart 1.5.0
  image: cart:2.0.0
  + new PDB
  + securityContext
```

The application did not change.

The deployment packaging did.

Therefore chart version changes independently.

---

# 8. Semantic Versioning

Recommended chart versioning:

```text
MAJOR.MINOR.PATCH
```

Example:

```text
1.4.2
```

Typical interpretation:

- MAJOR: breaking chart interface/behavior
- MINOR: backward-compatible functionality
- PATCH: backward-compatible fixes

Changing a values key used by many environments can be a breaking chart change.

---

# 9. values.yaml

`values.yaml` contains default configuration.

Example:

```yaml
replicaCount: 2

image:
  repository: ""
  pullPolicy: IfNotPresent
  tag: ""

service:
  type: ClusterIP
  port: 8080
  targetPort: 8080

resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
```

Defaults should be safe.

Do not make production defaults intentionally weak.

---

# 10. Values Layering

Helm can combine values from multiple sources.

Conceptually:

```text
values.yaml
    +
environment values
    +
CLI --set values
    =
final values
```

Example:

```bash
helm template cart ./roboshop-cart \
  -f values.yaml \
  -f values-production.yaml \
  --set image.tag=2.1.0
```

In GitOps, avoid uncontrolled CLI overrides.

Prefer version-controlled configuration.

---

# 11. Environment Values

Example:

```text
helm/
└── roboshop-cart/
    ├── values.yaml
    ├── values-dev.yaml
    ├── values-staging.yaml
    └── values-production.yaml
```

Production:

```yaml
replicaCount: 5

resources:
  requests:
    cpu: 300m
    memory: 256Mi
  limits:
    cpu: 1000m
    memory: 1Gi

autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 20
```

Development:

```yaml
replicaCount: 1

autoscaling:
  enabled: false
```

---

# 12. GitOps Recommendation

In a GitOps architecture, distinguish:

```text
Application chart
```

from:

```text
Environment configuration
```

One possible architecture:

```text
app-chart-repository
    |
    +-- chart templates

gitops-repository
    |
    +-- dev
    +-- staging
    +-- production
```

The chart defines how the application is deployed.

GitOps configuration defines which version/configuration each environment consumes.

This reduces environment-specific template duplication.

---

# 13. Helm Template Engine

Templates use Go templating.

Example:

```yaml
metadata:
  name: {{ include "roboshop-cart.fullname" . }}
```

Values:

```yaml
replicaCount: 3
```

Template:

```yaml
spec:
  replicas: {{ .Values.replicaCount }}
```

Helm renders the template into normal Kubernetes YAML.

---

# 14. Template Context

`.` represents the current context.

Common objects:

```text
.Values
.Chart
.Release
.Capabilities
.Files
.Template
```

Examples:

```gotemplate
{{ .Values.image.repository }}

{{ .Chart.Name }}

{{ .Chart.Version }}

{{ .Release.Name }}

{{ .Release.Namespace }}

{{ .Capabilities.KubeVersion.Version }}
```

Understanding context is essential for debugging templates.

---

# 15. Helm Built-in Objects

Useful values:

```gotemplate
.Release.Name
.Release.Namespace
.Release.Service
.Chart.Name
.Chart.Version
.Chart.AppVersion
.Values
.Capabilities.APIVersions
.Capabilities.KubeVersion.Version
```

Example:

```yaml
metadata:
  labels:
    app.kubernetes.io/instance: {{ .Release.Name }}
```

---

# 16. Template Functions

Helm provides functions for:

- string manipulation
- formatting
- default values
- dictionaries
- lists
- encoding
- type conversion
- hashing
- date handling

Example:

```gotemplate
{{ .Values.image.tag | quote }}
```

Another:

```gotemplate
{{ default "latest" .Values.image.tag }}
```

Be careful with `latest` in production.

Immutable image tags or digests are preferred.

---

# 17. Pipelines

Helm uses pipelines.

Example:

```gotemplate
{{ .Values.image.tag | quote }}
```

Equivalent concept:

```text
take value
    |
    v
send through quote
```

Another example:

```gotemplate
{{ .Values.service.port | default 8080 }}
```

---

# 18. Conditionals

Example:

```gotemplate
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
...
{{- end }}
```

This allows optional resources.

But do not make every field optional.

Excessive conditionals make charts difficult to understand.

---

# 19. Production Conditional Design

Good:

```yaml
ingress:
  enabled: true
```

Template:

```gotemplate
{{- if .Values.ingress.enabled }}
...
{{- end }}
```

Bad design:

```yaml
create:
  ingress:
    resource:
      enabled:
        use:
          condition: true
```

Avoid deeply nested meaningless configuration.

Values should model operational decisions.

---

# 20. Loops

Example:

```gotemplate
env:
{{- range .Values.env }}
  - name: {{ .name }}
    value: {{ .value | quote }}
{{- end }}
```

Values:

```yaml
env:
  - name: LOG_LEVEL
    value: INFO
  - name: REGION
    value: ap-south-1
```

This renders multiple environment variables.

---

# 21. `with`

Example:

```gotemplate
{{- with .Values.podSecurityContext }}
securityContext:
{{ toYaml . | nindent 8 }}
{{- end }}
```

`with` changes the current context.

This can improve readability for nested configuration.

---

# 22. `toYaml`

Very common production function:

```gotemplate
resources:
{{- toYaml .Values.resources | nindent 12 }}
```

Values:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

This avoids manually templating every resource field.

---

# 23. `nindent`

YAML indentation matters.

Example:

```gotemplate
{{- toYaml .Values.resources | nindent 12 }}
```

The output is inserted with 12 spaces.

Many Helm rendering failures are indentation errors.

---

# 24. Helper Templates

Helpers are stored in:

```text
templates/_helpers.tpl
```

Example:

```gotemplate
{{/*
Create chart name.
*/}}
{{- define "roboshop-cart.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}
```

Helpers prevent duplication.

---

# 25. Fullname Helper

Example:

```gotemplate
{{/*
Create a default fully qualified app name.
*/}}
{{- define "roboshop-cart.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
```

This standardizes resource names.

---

# 26. Standard Kubernetes Labels

Use Kubernetes recommended labels.

Example:

```yaml
labels:
  app.kubernetes.io/name: {{ include "roboshop-cart.name" . }}
  app.kubernetes.io/instance: {{ .Release.Name }}
  app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
  app.kubernetes.io/managed-by: {{ .Release.Service }}
  helm.sh/chart: {{ include "roboshop-cart.chart" . }}
```

Consistent labels improve:

- observability
- troubleshooting
- policy selection
- ownership
- dashboards

---

# 27. Deployment Template

Production baseline:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "roboshop-cart.fullname" . }}
  labels:
    {{- include "roboshop-cart.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  revisionHistoryLimit: {{ .Values.revisionHistoryLimit }}
  selector:
    matchLabels:
      {{- include "roboshop-cart.selectorLabels" . | nindent 6 }}
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: {{ .Values.strategy.maxUnavailable }}
      maxSurge: {{ .Values.strategy.maxSurge }}
```

---

# 28. Pod Template

Example:

```yaml
template:
  metadata:
    labels:
      {{- include "roboshop-cart.selectorLabels" . | nindent 6 }}
  spec:
    serviceAccountName: {{ include "roboshop-cart.serviceAccountName" . }}
    securityContext:
      {{- toYaml .Values.podSecurityContext | nindent 6 }}
```

Continue with:

```yaml
containers:
  - name: cart
    image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

---

# 29. Immutable Images

Prefer:

```yaml
image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/cart
  tag: "2.1.0"
```

Even stronger:

```yaml
image:
  repository: ...
  digest: "sha256:..."
```

Production deployment should avoid:

```yaml
tag: latest
```

Why?

Because `latest` can point to different images over time.

That damages:

- reproducibility
- rollback confidence
- incident investigation

---

# 30. Image Pull Policy

Typical configuration:

```yaml
image:
  pullPolicy: IfNotPresent
```

For immutable production tags:

```text
IfNotPresent
```

is generally reasonable.

For mutable tags, Kubernetes pull behavior becomes less predictable operationally.

Best practice:

```text
immutable image tag
+
image digest
+
controlled registry
```

---

# 31. Container Ports

Values:

```yaml
service:
  port: 8080
  targetPort: http
```

Container:

```yaml
ports:
  - name: http
    containerPort: 8080
    protocol: TCP
```

Named ports improve readability.

---

# 32. Environment Variables

Values:

```yaml
env:
  LOG_LEVEL: INFO
  REGION: ap-south-1
```

Template:

```gotemplate
env:
{{- range $key, $value := .Values.env }}
  - name: {{ $key }}
    value: {{ $value | quote }}
{{- end }}
```

Do not place sensitive credentials here.

---

# 33. ConfigMap Integration

Values:

```yaml
config:
  enabled: true
  data:
    LOG_LEVEL: INFO
    FEATURE_FLAG_X: "true"
```

Template:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "roboshop-cart.fullname" . }}
data:
{{- toYaml .Values.config.data | nindent 2 }}
```

Application:

```yaml
envFrom:
  - configMapRef:
      name: {{ include "roboshop-cart.fullname" . }}
```

---

# 34. Secrets

Do not commit real production secrets into:

```text
values.yaml
values-production.yaml
```

Bad:

```yaml
databasePassword: "SuperSecret123"
```

Better architecture:

```text
Helm
  |
  v
External Secrets / Secrets Manager
  |
  v
Kubernetes Secret
  |
  v
Pod
```

On AWS, secrets can be stored in:

- AWS Secrets Manager
- AWS Systems Manager Parameter Store

and synchronized through an appropriate secrets operator/controller.

---

# 35. Secret References

Application:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: cart-db
        key: password
```

The chart can template the reference without knowing the actual secret.

This is safer than embedding credentials.

---

# 36. Service Template

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "roboshop-cart.fullname" . }}
spec:
  type: {{ .Values.service.type }}
  selector:
    {{- include "roboshop-cart.selectorLabels" . | nindent 4 }}
  ports:
    - name: http
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
      protocol: TCP
```

Default:

```text
ClusterIP
```

is appropriate for most internal services.

---

# 37. Ingress Template

Example:

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "roboshop-cart.fullname" . }}
  annotations:
    {{- toYaml .Values.ingress.annotations | nindent 4 }}
spec:
  ingressClassName: {{ .Values.ingress.className }}
  rules:
    {{- toYaml .Values.ingress.rules | nindent 4 }}
{{- end }}
```

For EKS, this may integrate with AWS Load Balancer Controller.

---

# 38. HPA Template

Example:

```yaml
{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "roboshop-cart.fullname" . }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "roboshop-cart.fullname" . }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  metrics:
    {{- toYaml .Values.autoscaling.metrics | nindent 4 }}
{{- end }}
```

Avoid hardcoding only CPU if production scaling requires memory or custom metrics.

---

# 39. PDB Template

Example:

```yaml
{{- if .Values.pdb.enabled }}
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: {{ include "roboshop-cart.fullname" . }}
spec:
  minAvailable: {{ .Values.pdb.minAvailable }}
  selector:
    matchLabels:
      {{- include "roboshop-cart.selectorLabels" . | nindent 6 }}
{{- end }}
```

PDB protects availability during voluntary disruptions.

---

# 40. Probes

Values:

```yaml
probes:
  readiness:
    enabled: true
    httpGet:
      path: /health
      port: http
    initialDelaySeconds: 5
    periodSeconds: 10

  liveness:
    enabled: true
    httpGet:
      path: /health
      port: http
    periodSeconds: 20
```

Templates should avoid blindly enabling probes if the application does not support them.

---

# 41. Startup Probe

For slow-starting applications:

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: http
  failureThreshold: 30
  periodSeconds: 10
```

Startup probes prevent liveness checks from killing an application before initialization completes.

---

# 42. Security Context

Production example:

```yaml
podSecurityContext:
  runAsNonRoot: true
  seccompProfile:
    type: RuntimeDefault
```

Container:

```yaml
securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL
```

Not every application can use a read-only root filesystem without adjustments.

Test first.

---

# 43. ServiceAccount

Values:

```yaml
serviceAccount:
  create: true
  automount: false
  annotations: {}
```

For AWS workload identity, annotations may be configured as required by the chosen identity mechanism.

The chart should support configuration without hardcoding environment-specific AWS role identifiers.

---

# 44. Resource Requests and Limits

Production values should define requests.

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

Requests affect scheduling.

Limits constrain resource consumption.

Incorrect values can cause:

- throttling
- OOMKilled
- poor bin packing
- unnecessary cost

---

# 45. Scheduling Configuration

Values:

```yaml
affinity: {}

tolerations: []

topologySpreadConstraints: []
```

Template:

```gotemplate
{{- with .Values.affinity }}
affinity:
{{ toYaml . | nindent 8 }}
{{- end }}
```

This keeps scheduling policy configurable.

---

# 46. Node Affinity

Example:

```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
        - matchExpressions:
            - key: workload
              operator: In
              values:
                - application
```

Use this when workload placement is intentional.

Avoid unnecessary hard constraints that make scheduling fragile.

---

# 47. Pod Anti-Affinity

For highly available services:

```yaml
podAntiAffinity:
  preferredDuringSchedulingIgnoredDuringExecution:
    - weight: 100
      podAffinityTerm:
        topologyKey: kubernetes.io/hostname
        labelSelector:
          matchLabels:
            app.kubernetes.io/name: cart
```

This encourages replicas onto different nodes.

---

# 48. Topology Spread

Example:

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app.kubernetes.io/name: cart
```

This can improve multi-AZ resilience.

---

# 49. NetworkPolicy

Optional chart feature:

```yaml
networkPolicy:
  enabled: true
```

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: {{ include "roboshop-cart.fullname" . }}
spec:
  podSelector:
    matchLabels:
      {{- include "roboshop-cart.selectorLabels" . | nindent 6 }}
  policyTypes:
    - Ingress
    - Egress
```

NetworkPolicy behavior depends on the cluster networking implementation.

---

# 50. Avoiding Dangerous Defaults

Never make this default:

```yaml
hostNetwork: true
```

unless required.

Avoid:

```yaml
privileged: true
```

Avoid:

```yaml
runAsUser: 0
```

Avoid unrestricted:

```yaml
hostPath:
  /etc
```

A production chart should start from least privilege.

---

# 51. Values Schema

`values.schema.json` validates input values.

Example:

```json
{
  "$schema": "http://json-schema.org/schema#",
  "type": "object",
  "properties": {
    "replicaCount": {
      "type": "integer",
      "minimum": 1
    },
    "image": {
      "type": "object",
      "required": ["repository", "tag"]
    }
  }
}
```

This prevents invalid configuration from reaching the cluster.

---

# 52. Why Schema Validation Matters

Without schema:

```yaml
replicaCount: "five"
```

could produce invalid Kubernetes configuration.

With schema:

```text
Expected integer
Received string
```

The error occurs before deployment.

Shift validation left.

---

# 53. Chart Dependencies

Applications sometimes require dependent charts.

Example:

```yaml
dependencies:
  - name: redis
    version: 20.0.0
    repository: oci://registry.example.com/charts
```

Use dependencies carefully.

For platform-provided infrastructure, it may be better to manage shared services separately.

Do not install an independent Redis instance per microservice unless the architecture intentionally requires it.

---

# 54. Dependency Management

Commands:

```bash
helm dependency update ./cart
```

```bash
helm dependency build ./cart
```

Inspect:

```bash
ls charts/
```

Commit strategy depends on organizational policy.

Many teams pin dependency versions and maintain lock files.

---

# 55. Chart.lock

Dependency resolution can create:

```text
Chart.lock
```

This helps reproduce dependency versions.

Production principle:

```text
pin versions
+
review upgrades
+
test dependencies
```

Avoid uncontrolled dependency drift.

---

# 56. Helm Lint

Basic validation:

```bash
helm lint ./roboshop-cart
```

This catches many chart-level issues.

However:

```text
helm lint
```

does not guarantee Kubernetes deployment success.

You still need rendering and cluster validation.

---

# 57. Helm Template

Render locally:

```bash
helm template cart ./roboshop-cart
```

Save:

```bash
helm template cart ./roboshop-cart > rendered.yaml
```

Then inspect:

```bash
less rendered.yaml
```

This is one of the most useful Helm debugging techniques.

---

# 58. Helm Template with Environment

```bash
helm template cart ./roboshop-cart \
  -f values-production.yaml
```

Inspect only relevant output:

```bash
helm template cart ./roboshop-cart \
  -f values-production.yaml \
  --show-only templates/deployment.yaml
```

This isolates template problems.

---

# 59. Kubernetes Dry Run

After rendering:

```bash
kubectl apply --dry-run=server -f rendered.yaml
```

This validates against the Kubernetes API server.

This is stronger than purely local YAML validation.

---

# 60. Helm Install

Example:

```bash
helm install cart ./roboshop-cart \
  --namespace roboshop \
  --create-namespace
```

Helm creates a release.

Check:

```bash
helm list -n roboshop
```

---

# 61. Helm Upgrade

Example:

```bash
helm upgrade cart ./roboshop-cart \
  -n roboshop \
  -f values-production.yaml
```

Production upgrades should be controlled and auditable.

In GitOps environments, Argo CD normally performs synchronization rather than engineers running manual upgrades.

---

# 62. Helm Upgrade with Install

For automation:

```bash
helm upgrade --install cart ./roboshop-cart \
  -n roboshop \
  -f values-production.yaml
```

This is useful in CI for environments where Helm is intentionally the deployment mechanism.

Do not mix manual Helm ownership with Argo CD ownership for the same resources.

---

# 63. Helm Release History

Check:

```bash
helm history cart -n roboshop
```

Example:

```text
REVISION  STATUS      CHART
1         superseded  cart-1.0.0
2         superseded  cart-1.1.0
3         deployed    cart-1.2.0
```

This helps identify deployment history.

---

# 64. Helm Rollback

Example:

```bash
helm rollback cart 2 -n roboshop
```

Rollback should be understood as restoring a previous release configuration.

It is not always a complete application rollback.

External state such as:

- databases
- queues
- migrations
- external APIs

may not automatically roll back.

---

# 65. Database Migration Warning

Suppose release 10 changes:

```text
database schema:
v1 -> v2
```

Then rollback application code to release 9.

If release 9 cannot work with schema v2, rollback fails operationally.

Therefore use backward-compatible migration patterns:

```text
expand
  ->
migrate
  ->
deploy
  ->
contract
```

Helm cannot solve application/data compatibility by itself.

---

# 66. Helm Hooks

Hooks can run at lifecycle events.

Examples:

```text
pre-install
post-install
pre-upgrade
post-upgrade
pre-delete
```

Example annotation:

```yaml
metadata:
  annotations:
    "helm.sh/hook": pre-upgrade
```

Hooks are powerful but can introduce operational complexity.

Use them only when the lifecycle action genuinely belongs to the release.

---

# 67. Helm Hook Risks

Problems can include:

- hook jobs not completing
- failed upgrades
- orphaned resources
- ordering confusion
- non-idempotent operations

For database migrations, carefully evaluate whether a Helm hook is the correct mechanism.

In many production platforms, migrations are handled by a dedicated deployment workflow.

---

# 68. Helm Tests

Helm supports test resources.

Example:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "roboshop-cart.fullname" . }}-test"
  annotations:
    "helm.sh/hook": test
spec:
  restartPolicy: Never
  containers:
    - name: curl
      image: curlimages/curl
      command:
        - curl
        - --fail
        - http://{{ include "roboshop-cart.fullname" . }}:8080/health
```

Run:

```bash
helm test cart -n roboshop
```

---

# 69. Test Strategy

Recommended layers:

```text
helm lint
    |
    v
helm template
    |
    v
schema validation
    |
    v
kubeconform / kubeval-style validation
    |
    v
policy checks
    |
    v
cluster dry-run
    |
    v
integration tests
    |
    v
deployment
```

No single test catches everything.

---

# 70. Static YAML Validation

Rendered manifests can be validated with tools such as:

```text
kubeconform
```

Conceptually:

```bash
helm template cart ./roboshop-cart |
  kubeconform -strict
```

This catches schema/API structure problems.

---

# 71. Security Scanning

Helm-rendered manifests should be checked for security issues.

Potential tools/categories:

```text
Trivy
Checkov-style policy tools
Kyverno
OPA/Gatekeeper
Conftest
Kube-score-style analyzers
```

Examples of checks:

- privileged containers
- host networking
- missing resource requests
- missing probes
- root user
- dangerous capabilities
- insecure images

---

# 72. CI Pipeline for Helm

Production example:

```text
Pull Request
    |
    +--> helm lint
    |
    +--> schema validation
    |
    +--> helm template
    |
    +--> Kubernetes schema validation
    |
    +--> security scan
    |
    +--> policy checks
    |
    +--> unit/template tests
    |
    v
Merge
    |
    v
Package chart
    |
    v
Sign / provenance
    |
    v
Publish OCI artifact
```

---

# 73. Chart Packaging

Package:

```bash
helm package ./roboshop-cart
```

Output:

```text
roboshop-cart-1.3.0.tgz
```

Inspect:

```bash
tar -tzf roboshop-cart-1.3.0.tgz
```

The package should be treated as a versioned artifact.

---

# 74. OCI Registries

Modern Helm supports OCI registries.

Example:

```bash
helm registry login <registry>
```

Push:

```bash
helm push roboshop-cart-1.3.0.tgz \
  oci://<registry>/helm
```

Pull:

```bash
helm pull \
  oci://<registry>/helm/roboshop-cart \
  --version 1.3.0
```

AWS environments can use Amazon ECR for OCI artifacts.

---

# 75. Helm + Amazon ECR

A production architecture can store:

```text
Docker images
      |
      v
ECR repositories

Helm charts
      |
      v
ECR OCI repositories
```

This provides a centralized AWS-native artifact strategy.

Access should be controlled using IAM.

---

# 76. Artifact Promotion

Do not rebuild the chart for each environment unnecessarily.

Prefer:

```text
Build chart 1.3.0
       |
       v
Test
       |
       v
Promote same artifact
       |
       +--> dev
       +--> staging
       +--> production
```

The configuration can change while the immutable artifact remains the same.

---

# 77. Chart Signing

For high-assurance environments, chart integrity can be strengthened with signing and provenance mechanisms supported by the Helm ecosystem.

The objective is:

```text
artifact
   |
   +--> integrity
   +--> authenticity
   +--> traceability
```

Combine this with registry access controls and CI identity.

---

# 78. Argo CD and Helm

Argo CD can deploy Helm charts.

Example conceptual Application:

```yaml
spec:
  source:
    repoURL: https://git.example.com/platform/gitops.git
    path: applications/cart
    helm:
      valueFiles:
        - values-production.yaml
```

Argo CD renders Helm and applies the resulting Kubernetes resources.

---

# 79. Helm Is Not the GitOps Controller

Important interview point:

```text
Helm
=
packaging/template/release tooling

Argo CD
=
GitOps reconciliation/controller
```

With Argo CD:

```text
Git
 |
 v
desired state
 |
 v
Argo CD
 |
 v
Helm rendering
 |
 v
Kubernetes API
```

Argo CD continuously compares desired and live state.

---

# 80. GitOps Repository Pattern

Example:

```text
gitops/
├── environments/
│   ├── dev/
│   │   └── cart/
│   │       └── values.yaml
│   ├── staging/
│   │   └── cart/
│   │       └── values.yaml
│   └── production/
│       └── cart/
│           └── values.yaml
└── applications/
    └── cart.yaml
```

Alternative patterns are valid.

Choose one and standardize it.

---

# 81. ApplicationSet Pattern

For many microservices/environments:

```text
ApplicationSet
      |
      +--> cart-dev
      +--> cart-staging
      +--> cart-prod
      |
      +--> catalogue-dev
      +--> catalogue-staging
      +--> catalogue-prod
```

This reduces repetitive Argo CD Application definitions.

---

# 82. Helm Values in Argo CD

Keep environment configuration version controlled.

Example:

```yaml
helm:
  valuesObject:
    replicaCount: 5
    image:
      tag: "2.1.0"
```

Or use values files:

```yaml
helm:
  valueFiles:
    - values-production.yaml
```

Values files are often easier to review when configuration becomes large.

---

# 83. Image Tag Updates

A common GitOps model:

```text
CI
 |
 +--> build image
 |
 +--> push image
 |
 +--> update GitOps image tag
 |
 v
Git commit
 |
 v
Argo CD
 |
 v
EKS
```

This provides an audit trail.

Example commit:

```text
chore(cart): promote image to 2.1.0
```

---

# 84. Image Automation

Tools can automate image updates, but governance matters.

Production should require:

- approved image source
- vulnerability threshold
- immutable tags
- environment promotion
- Git audit trail
- rollback mechanism

Do not allow an automated image updater to blindly deploy every newly published image to production.

---

# 85. Helm Diff

A diff capability is useful before changes.

Conceptually:

```text
current state
      |
      v
new rendered state
      |
      v
diff
```

Review:

- replicas
- images
- ports
- probes
- security contexts
- RBAC
- ingress
- network policy

In GitOps, the Argo CD diff view serves a similar purpose.

---

# 86. Production Values Example

```yaml
replicaCount: 4

revisionHistoryLimit: 10

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
  tag: "2.1.0"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 8080
  targetPort: http

resources:
  requests:
    cpu: 300m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 1Gi

autoscaling:
  enabled: true
  minReplicas: 4
  maxReplicas: 20

pdb:
  enabled: true
  minAvailable: 3
```

---

# 87. Production Deployment Strategy

Example:

```yaml
strategy:
  maxUnavailable: 0
  maxSurge: 1
```

This can preserve capacity during rolling deployments.

But it increases temporary resource consumption.

The correct strategy depends on:

- application startup time
- cluster capacity
- replica count
- traffic
- cost constraints

---

# 88. Graceful Shutdown

Values:

```yaml
terminationGracePeriodSeconds: 60
```

Container:

```yaml
lifecycle:
  preStop:
    exec:
      command:
        - /bin/sh
        - -c
        - sleep 10
```

This is only useful if the application and traffic flow actually benefit from it.

Prefer application-level graceful shutdown handling.

---

# 89. Helm Naming Constraints

Kubernetes names have length restrictions.

Use:

```gotemplate
| trunc 63
| trimSuffix "-"
```

Avoid generating names from arbitrary long user input.

A good helper centralizes naming behavior.

---

# 90. Avoid Name Collisions

Use release-aware names:

```text
{{ .Release.Name }}-{{ .Chart.Name }}
```

But avoid unnecessarily long names.

For shared namespaces:

```text
cart-prod
cart-staging
cart-dev
```

should be distinguishable.

---

# 91. Namespace Strategy

Usually:

```text
roboshop-dev
roboshop-staging
roboshop-prod
```

or service/team-oriented namespaces depending on organizational policy.

Do not allow every chart to silently create arbitrary namespaces.

Namespace ownership should normally be platform-controlled.

---

# 92. RBAC in Helm

A chart may optionally create:

- ServiceAccount
- Role
- RoleBinding

Example:

```yaml
rbac:
  create: true
```

Use least privilege.

Do not create:

```yaml
ClusterRole:
  resources:
    - "*"
  verbs:
    - "*"
```

for a normal application.

---

# 93. ClusterRole Risk

A Helm chart that creates cluster-wide privileges can create a major security risk.

Review:

```yaml
apiGroups:
resources:
verbs:
```

Question:

> Does this workload actually need cluster-wide permissions?

If not, use namespace-scoped Role/RoleBinding.

---

# 94. CRDs

Some charts install CRDs.

Examples include operators and platform controllers.

Be careful with:

```text
CRD ownership
upgrade behavior
deletion behavior
version compatibility
```

Application charts should not casually manage platform CRDs unless the ownership model is explicit.

---

# 95. Helm and CRD Upgrades

CRDs can have lifecycle semantics different from ordinary resources.

Before upgrading a chart that includes CRDs:

```text
review CRD version
review controller compatibility
review conversion strategy
review backup/recovery
```

Treat CRD upgrades as platform changes.

---

# 96. Optional Resources

A chart may expose:

```yaml
serviceMonitor:
  enabled: false
```

If enabled:

```text
ServiceMonitor
      |
      v
Prometheus Operator
      |
      v
metrics
```

Do not assume every Kubernetes cluster has the required CRD.

Conditional resources must match cluster capabilities.

---

# 97. Capability Checks

Helm can check API capabilities.

Example:

```gotemplate
{{- if .Capabilities.APIVersions.Has "monitoring.coreos.com/v1" }}
...
{{- end }}
```

This can protect against rendering resources unsupported by the target cluster.

However, explicit version pinning and environment validation are often clearer.

---

# 98. Kubernetes API Versions

Avoid deprecated APIs.

Example:

```yaml
apiVersion: apps/v1
```

For Ingress:

```yaml
apiVersion: networking.k8s.io/v1
```

Production chart maintenance includes tracking Kubernetes API deprecations.

---

# 99. Helm Upgrade Compatibility

When Kubernetes is upgraded:

```text
Kubernetes upgrade
       |
       v
render existing charts
       |
       v
check deprecated APIs
       |
       v
upgrade charts if necessary
```

A chart that worked on an old cluster may fail after API removal.

---

# 100. Multi-Cluster Helm

Example:

```text
GitOps
 |
 +--> EKS production cluster
 |
 +--> EKS DR cluster
 |
 +--> EKS staging cluster
```

The same chart can be reused.

Cluster-specific values may include:

```yaml
region:
clusterName:
ingress:
storageClass:
```

Do not hardcode one cluster's infrastructure assumptions into the chart.

---

# 101. Regional Configuration

Example:

```yaml
aws:
  region: ap-south-1
```

For another environment:

```yaml
aws:
  region: ap-southeast-1
```

But ask whether the application really needs the region as an environment variable.

Avoid exposing infrastructure configuration to applications unnecessarily.

---

# 102. Storage Configuration

If a chart supports persistence:

```yaml
persistence:
  enabled: true
  storageClass: gp3
  size: 20Gi
  accessModes:
    - ReadWriteOnce
```

For stateful workloads, carefully evaluate:

- backup
- restore
- AZ behavior
- volume lifecycle
- data durability

Do not treat a PVC as a complete backup solution.

---

# 103. StatefulSet Packaging

For stateful workloads, a chart may template:

```yaml
kind: StatefulSet
```

with:

```yaml
volumeClaimTemplates:
```

Stateful applications require additional operational design.

For many RoboShop stateless microservices, Deployment is generally more appropriate.

---

# 104. DaemonSet Packaging

Platform components may use:

```yaml
kind: DaemonSet
```

Examples:

- log agents
- node monitoring agents
- security agents

Application Helm charts generally should not deploy platform-wide DaemonSets unless that ownership is intentional.

---

# 105. Jobs and CronJobs

Helm can package:

```yaml
kind: Job
```

or:

```yaml
kind: CronJob
```

Example:

```yaml
schedule: "*/15 * * * *"
```

Validate:

- concurrencyPolicy
- successfulJobsHistoryLimit
- failedJobsHistoryLimit
- resource limits
- idempotency

---

# 106. Production CronJob

Example:

```yaml
concurrencyPolicy: Forbid

successfulJobsHistoryLimit: 3
failedJobsHistoryLimit: 5

startingDeadlineSeconds: 300
```

A scheduled job should not accidentally overlap with itself when execution takes longer than expected.

---

# 107. Configurable Pod Labels

Values:

```yaml
podLabels:
  team: shopping
  component: cart
```

Template:

```gotemplate
{{- with .Values.podLabels }}
{{ toYaml . | nindent 8 }}
{{- end }}
```

Labels should have clear ownership semantics.

---

# 108. Pod Annotations

Values:

```yaml
podAnnotations: {}
```

Use cases include:

- observability
- reload mechanisms
- platform integration

Avoid putting arbitrary operational behavior into annotations without documentation.

---

# 109. Environment Variables from Secret and ConfigMap

Production pattern:

```yaml
envFrom:
  - configMapRef:
      name: cart-config
  - secretRef:
      name: cart-secret
```

But explicit environment references are sometimes preferable because they make dependencies and names visible.

Choose based on maintainability and security.

---

# 110. Resource Defaults

A chart should define sane defaults.

Example:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
```

But platform policy can enforce minimum requests using admission policies.

Defense in depth:

```text
Helm defaults
+
CI policy
+
Admission policy
```

---

# 111. Pod Security Admission

Helm should produce manifests compatible with the cluster's pod security policy.

Production expectation:

```text
non-root
no privilege escalation
drop capabilities
seccomp RuntimeDefault
```

Where possible, namespace-level Pod Security Admission should enforce baseline/restricted standards.

---

# 112. Policy as Code

A Helm chart can be checked by policy before deployment.

Example policy goals:

```text
deny privileged containers
deny hostPath
require resources
require non-root
require image registry allowlist
require probes
```

This means chart authors cannot accidentally bypass platform security standards.

---

# 113. Helm Secrets Strategy

Never do:

```yaml
password: "prod-password"
```

in Git.

Even private Git repositories should not be treated as a secret store.

Use:

```text
AWS Secrets Manager
        |
        v
External Secrets
        |
        v
Kubernetes Secret
        |
        v
Application
```

Git stores references, not plaintext secrets.

---

# 114. Secret Rotation

Secret rotation should not require chart source changes.

Example:

```text
AWS Secrets Manager
        |
      rotate
        |
        v
External Secrets controller
        |
        v
Kubernetes Secret
        |
        v
Pod restart/reload mechanism
```

The exact reload mechanism depends on application behavior.

---

# 115. Helm Rollout on Config Change

A common issue:

ConfigMap changes but Pods do not restart.

A checksum annotation can solve this.

Example concept:

```gotemplate
checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
```

When ConfigMap content changes, the Pod template changes, triggering a rollout.

---

# 116. Why Checksums Matter

Without checksum:

```text
Git change
  |
  v
ConfigMap updated
  |
  v
existing Pod continues
  |
  v
application still uses old configuration
```

With checksum:

```text
ConfigMap change
  |
  v
checksum changes
  |
  v
Deployment PodTemplate changes
  |
  v
rolling update
```

This is a useful production Helm pattern.

---

# 117. Secret Checksum

The same pattern can be used when appropriate for secret references.

However, avoid rendering secret contents into Helm output simply to calculate a checksum.

Prefer hashing a safe reference/configuration representation or use a dedicated secret reload mechanism.

---

# 118. Helm and External Secrets

Example:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: cart
spec:
  refreshInterval: 1h
```

The chart should only include this if the platform has the corresponding CRD/controller.

Do not bundle platform controllers inside every application chart.

---

# 119. Chart Responsibility Boundary

Good application chart responsibility:

```text
Deployment
Service
Ingress
HPA
PDB
ConfigMap
ServiceAccount
application NetworkPolicy
```

Platform responsibility:

```text
EKS
AWS Load Balancer Controller
External Secrets controller
Prometheus operator
Argo CD
Karpenter
Fluent Bit
cert-manager
```

This separation is critical in large environments.

---

# 120. Platform vs Application Helm Charts

Platform charts:

```text
argocd
prometheus
grafana
external-secrets
aws-load-balancer-controller
```

Application charts:

```text
cart
catalogue
user
payment
shipping
```

Different ownership and lifecycle.

---

# 121. Library Charts

Helm supports reusable library charts.

Use them for:

```text
common labels
security defaults
common deployment patterns
standard annotations
```

This can reduce duplication across dozens of microservice charts.

But avoid building an overly abstract internal framework that makes simple applications difficult to understand.

---

# 122. Standardization Strategy

A large organization may provide:

```text
company-library-chart
```

with reusable helpers.

Application chart:

```text
cart
```

consumes common templates/helpers.

Benefits:

- consistent labels
- security defaults
- probes
- resource conventions
- observability
- naming

Governance is essential.

---

# 123. Helm Anti-Pattern: Giant Generic Chart

Bad:

```text
one chart supports every application type
```

with hundreds of switches:

```yaml
enableDeployment:
enableStatefulSet:
enableDaemonSet:
enableJob:
enableCronJob:
enableRedis:
enableKafka:
...
```

This becomes difficult to test and maintain.

Prefer focused charts or well-designed reusable components.

---

# 124. Helm Anti-Pattern: Copy-Paste Environment Charts

Bad:

```text
cart-dev/
cart-staging/
cart-prod/
```

with duplicated templates.

Better:

```text
one chart
+
environment values
```

unless environments genuinely have different deployment models.

---

# 125. Helm Anti-Pattern: Hardcoded Environment Values

Bad:

```gotemplate
replicas: 10
```

when replica count is expected to vary.

Better:

```gotemplate
replicas: {{ .Values.replicaCount }}
```

Hardcoded values should be intentional platform constants, not accidental environment configuration.

---

# 126. Helm Anti-Pattern: Excessive `--set`

Avoid production commands such as:

```bash
helm upgrade ... \
  --set a=x \
  --set b=y \
  --set c=z \
  --set d=q \
  --set e=p
```

This creates poor auditability.

Prefer version-controlled values.

---

# 127. Helm Anti-Pattern: Secrets in CLI

Never:

```bash
helm upgrade \
  --set password='secret'
```

CLI arguments can appear in:

- shell history
- process inspection
- CI logs

Use a proper secret management mechanism.

---

# 128. Helm Anti-Pattern: Mutable Image Tags

Avoid:

```yaml
tag: latest
```

Better:

```yaml
tag: "2.1.0"
```

Best for high-assurance deployments:

```yaml
digest: sha256:...
```

---

# 129. Helm Anti-Pattern: No Resource Requests

Without requests:

```text
scheduler has weak information
+
autoscaling becomes unreliable
+
capacity planning becomes difficult
```

Every production workload should have measured resource requests.

---

# 130. Helm Anti-Pattern: No Probes

Without readiness:

```text
Service
  |
  v
Pod
  |
  v
application not ready
```

Traffic may reach an application before initialization completes.

Production applications should have meaningful readiness checks.

---

# 131. Helm Anti-Pattern: Liveness Equals Readiness

Do not blindly use:

```text
same endpoint
same thresholds
```

for both.

Readiness answers:

> Can this pod receive traffic?

Liveness answers:

> Is this process unhealthy enough that restarting it is appropriate?

These are different questions.

---

# 132. Helm Chart README

Every production chart should document:

```text
purpose
installation
required values
optional values
dependencies
configuration
security considerations
upgrade process
rollback
testing
troubleshooting
ownership
```

Example:

```bash
helm lint .
helm template .
```

Documentation reduces operational dependency on chart authors.

---

# 133. Values Documentation

Document important values:

```yaml
replicaCount: 3

# Minimum number of application replicas.
# Production recommendation: >= 3 for HA.
```

For large charts, generate values documentation automatically where practical.

---

# 134. Required Values

Schema can enforce:

```json
"required": [
  "image"
]
```

Avoid forcing operators to specify every value.

Defaults should cover safe common behavior.

---

# 135. Validation in Templates

Functions such as `required` can enforce critical configuration.

Example:

```gotemplate
{{ required "image.repository is required" .Values.image.repository }}
```

This produces an immediate Helm error.

Use this for truly required configuration.

---

# 136. Fail Fast

Good Helm design:

```text
missing production-critical value
        |
        v
Helm rendering fails
        |
        v
deployment does not happen
```

Bad:

```text
missing value
        |
        v
empty resource generated
        |
        v
cluster deployment fails later
```

Fail early whenever possible.

---

# 137. Template Quoting

Be careful with YAML types.

Example:

```yaml
port: {{ .Values.service.port }}
```

Numeric value should remain numeric.

Environment variables should generally be strings:

```gotemplate
value: {{ .Values.env.LOG_LEVEL | quote }}
```

Incorrect quoting can change Kubernetes API types.

---

# 138. Boolean Values

Values:

```yaml
enabled: true
```

Template:

```gotemplate
{{- if .Values.enabled }}
```

Do not use string booleans unnecessarily:

```yaml
enabled: "true"
```

Schema validation can enforce expected types.

---

# 139. Numeric Values

Example:

```yaml
replicaCount: 3
```

Schema:

```json
"type": "integer"
```

This prevents:

```yaml
replicaCount: "three"
```

from reaching deployment.

---

# 140. Lists and Dictionaries

Helm values commonly contain:

```yaml
tolerations:
  - key: workload
    operator: Equal
    value: application
    effect: NoSchedule
```

and:

```yaml
nodeSelector:
  workload: application
```

Use `toYaml` to preserve structure.

---

# 141. Deep Merge Awareness

Helm values merging can surprise operators when nested structures are overridden.

For example, replacing a nested object may remove defaults rather than performing the merge an operator expected.

Therefore:

```text
test final rendered output
```

rather than assuming values behave like a generic deep-merge configuration system.

---

# 142. Final Values Debugging

Useful:

```bash
helm template cart ./chart \
  -f values.yaml \
  -f values-production.yaml \
  --debug
```

Also inspect:

```bash
helm get values cart -n roboshop
```

and:

```bash
helm get manifest cart -n roboshop
```

These help determine what Helm actually used/rendered.

---

# 143. Helm Release Inspection

Commands:

```bash
helm status cart -n roboshop
```

```bash
helm get values cart -n roboshop
```

```bash
helm get manifest cart -n roboshop
```

```bash
helm history cart -n roboshop
```

During incidents, these are valuable.

---

# 144. Helm vs Kubectl

`kubectl` manages Kubernetes resources directly.

Helm manages:

```text
versioned application packaging
+
templating
+
release metadata
```

Example:

```bash
kubectl apply -f deployment.yaml
```

versus:

```bash
helm upgrade --install cart ./chart
```

In GitOps:

```text
Argo CD
+
Helm
+
Kubernetes API
```

---

# 145. Helm vs Kustomize

Helm:

```text
templates
+
values
+
package management
```

Kustomize:

```text
base manifests
+
overlays
+
patches
```

Both can be used in production.

The key is consistency.

Do not create unnecessary tool complexity for one application.

---

# 146. Helm vs Raw YAML

Raw YAML is perfectly valid for simple resources.

Helm becomes valuable when you need:

- reuse
- packaging
- versioning
- configuration
- dependencies
- environment variation

Do not use Helm solely because it is fashionable.

---

# 147. Helm vs Terraform

Terraform:

```text
infrastructure
```

Helm:

```text
Kubernetes application packaging
```

In this capstone:

```text
Terraform
   |
   v
AWS + EKS infrastructure

Helm
   |
   v
application manifests

Argo CD
   |
   v
GitOps reconciliation
```

Clear ownership prevents resource conflicts.

---

# 148. Terraform and Helm Ownership

Avoid:

```text
Terraform creates Deployment
+
Argo CD manages same Deployment
```

or:

```text
Terraform installs chart
+
Argo CD manages same release
```

unless there is an explicitly designed ownership model.

One resource should have one authoritative owner.

---

# 149. CI and Helm Ownership

Recommended:

```text
CI
 |
 +--> build
 +--> test
 +--> scan
 +--> publish image
 +--> update GitOps
 |
 v
Git
 |
 v
Argo CD
 |
 v
Helm
 |
 v
EKS
```

CI should normally not bypass GitOps by directly deploying production.

---

# 150. Promotion Model

Example:

```text
commit
 |
 v
CI
 |
 v
image 2.1.0
 |
 v
dev
 |
 +--> automated tests
 |
 v
staging
 |
 +--> integration tests
 |
 v
production approval
 |
 v
production
```

The Helm chart can remain the same.

Environment configuration references the promoted image.

---

# 151. Rollback Model

Application rollback:

```text
Git
 |
 v
previous image tag
 |
 v
Argo CD
 |
 v
Helm render
 |
 v
Deployment rollout
```

If urgent:

```text
revert Git commit
```

This is often preferable in GitOps because the source of truth remains consistent.

---

# 152. Emergency Rollback

If production is failing:

1. Confirm incident.
2. Stop further rollout.
3. Identify known-good image/chart version.
4. Revert GitOps change.
5. Let Argo CD reconcile.
6. Monitor rollout.
7. Validate application.
8. Record incident details.

Avoid undocumented manual changes.

---

# 153. Helm Troubleshooting Decision Tree

```text
Deployment failed
      |
      v
Does helm template fail?
      |
   +--Yes--> fix template/values
   |
   No
   |
   v
Does Kubernetes reject manifest?
   |
 +--Yes--> inspect API/schema
 |
 No
 |
 v
Pods unhealthy?
 |
 +--> describe pod
 +--> logs
 +--> events
 +--> probes
 +--> resources
```

---

# 154. Template Failure

Error example:

```text
nil pointer evaluating interface
```

Check:

```text
missing values
incorrect path
wrong context
with/range context
```

Use:

```bash
helm template --debug
```

---

# 155. YAML Parse Error

Example:

```text
YAML parse error
```

Typical causes:

- indentation
- missing colon
- incorrect quoting
- malformed conditional
- list indentation

Render the chart:

```bash
helm template .
```

and inspect the generated YAML.

---

# 156. Kubernetes Validation Error

Example:

```text
unknown field
```

Possible causes:

```text
wrong apiVersion
deprecated API
incorrect field name
wrong value type
```

Use:

```bash
kubectl explain deployment.spec
```

and Kubernetes API documentation/version-specific validation.

---

# 157. Pod CrashLoopBackOff

Helm itself may be fine.

Investigate:

```bash
kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop --previous
```

Possible causes:

- bad environment variable
- missing secret
- application startup failure
- database unavailable
- incorrect command
- incorrect image

---

# 158. ImagePullBackOff

Check:

```bash
kubectl describe pod <pod> -n roboshop
```

Look for:

```text
repository
tag
digest
registry access
IAM
imagePullSecrets
network
```

In EKS, verify node/pod access to ECR as applicable.

---

# 159. Readiness Failure

Symptoms:

```text
Pod Running
but not Ready
```

Check:

```bash
kubectl describe pod
```

Possible causes:

- wrong path
- wrong port
- dependency unavailable
- startup too slow
- probe timeout too low

Do not simply disable the readiness probe to hide the problem.

---

# 160. HPA Not Scaling

Check:

```bash
kubectl get hpa -n roboshop
kubectl describe hpa <name> -n roboshop
```

Then verify:

```text
resource requests
metrics-server / metrics source
CPU/memory metrics
target values
maxReplicas
```

HPA depends on meaningful resource requests for resource-based metrics.

---

# 161. PDB Blocking Drain

If node drain hangs:

```text
PDB may be protecting too many pods
```

Example:

```text
replicas = 3
minAvailable = 3
```

No pod can voluntarily be disrupted.

A PDB must reflect availability requirements without making maintenance impossible.

---

# 162. Ingress Failure

Check:

```bash
kubectl get ingress -n roboshop
kubectl describe ingress <name> -n roboshop
```

Then inspect:

```text
AWS Load Balancer Controller
target groups
health checks
security groups
subnets
service ports
```

The Helm template may be correct even when AWS infrastructure configuration is wrong.

---

# 163. Service Has No Endpoints

Check:

```bash
kubectl get endpointslice -n roboshop
```

Typical cause:

```text
Service selector
      !=
Pod labels
```

This is why standardized helper labels are important.

---

# 164. Helm Debugging Example

Command:

```bash
helm template cart ./chart \
  -f values-production.yaml \
  --debug
```

Then:

```bash
kubectl apply --dry-run=server -f rendered.yaml
```

Then:

```bash
kubectl diff -f rendered.yaml
```

This separates:

```text
template problem
```

from:

```text
cluster/API problem
```

---

# 165. Production Change Review

Before merging a chart change ask:

### Application
- Is the image version correct?
- Is startup behavior unchanged?

### Kubernetes
- Are requests/limits correct?
- Are probes correct?
- Are selectors unchanged?
- Are ports correct?

### Security
- Any new permissions?
- Any privileged behavior?
- Any secrets exposed?

### Availability
- Does rollout preserve capacity?
- Is PDB compatible?
- Is topology spread still valid?

### Operations
- Is rollback possible?
- Is documentation updated?

---

# 166. Helm Selector Safety

Do not casually change:

```yaml
spec:
  selector:
```

Deployment selectors are effectively immutable.

A chart change that changes selector labels can cause deployment failures or resource replacement issues.

Treat selector labels as stable API.

---

# 167. Label Contract

Use stable:

```text
app.kubernetes.io/name
app.kubernetes.io/instance
```

Do not include frequently changing values in selectors.

Example:

Bad selector:

```text
version: 2.1.0
```

Better:

```text
app.kubernetes.io/name: cart
```

Version belongs in metadata, not stable identity selectors.

---

# 168. Canary Deployment Consideration

Standard Helm Deployment provides rolling updates.

Advanced delivery may use:

```text
Argo Rollouts
```

for:

- canary
- blue/green
- weighted traffic
- automated analysis

Do not overload Helm templates with sophisticated rollout logic if a dedicated controller is more appropriate.

---

# 169. Helm with Argo Rollouts

Application chart may conditionally create:

```yaml
kind: Rollout
```

instead of:

```yaml
kind: Deployment
```

This requires the Argo Rollouts CRD/controller.

Keep that dependency explicit.

---

# 170. Production Observability

A chart should make observability easy.

Examples:

```yaml
podAnnotations:
  prometheus.io/scrape: "true"
```

However, if Prometheus Operator is used, a `ServiceMonitor` may be more appropriate.

Modern production design should standardize the observability integration rather than rely on inconsistent annotations.

---

# 171. Logging

The chart should configure application output expectations.

Prefer:

```text
stdout
stderr
```

rather than application-specific log files inside containers.

Then:

```text
Fluent Bit / log agent
      |
      v
Elasticsearch/OpenSearch
      |
      v
Kibana
```

This integrates with the capstone logging architecture.

---

# 172. Tracing

Applications may expose OpenTelemetry configuration:

```yaml
env:
  OTEL_SERVICE_NAME: cart
  OTEL_EXPORTER_OTLP_ENDPOINT: ...
```

Do not hardcode environment-specific collector addresses.

Use values or platform injection.

---

# 173. Standard Application Interface

A platform Helm chart can standardize:

```text
HTTP port
health endpoint
metrics endpoint
resource requests
graceful shutdown
security context
service account
labels
annotations
```

This makes applications easier to operate consistently.

---

# 174. Production Readiness Checklist — Chart

Before production:

```text
[ ] Chart.yaml valid
[ ] chart version correct
[ ] app version correct
[ ] values schema present where useful
[ ] defaults reviewed
[ ] image immutable
[ ] resource requests defined
[ ] limits reviewed
[ ] readiness probe
[ ] liveness probe
[ ] startup probe if needed
[ ] securityContext
[ ] non-root
[ ] capabilities dropped
[ ] ServiceAccount reviewed
[ ] RBAC least privilege
[ ] PDB configured
[ ] HPA configured where needed
[ ] topology policy reviewed
[ ] network policy reviewed
[ ] secrets externalized
```

---

# 175. Production Readiness Checklist — CI

```text
[ ] helm lint
[ ] helm template
[ ] values schema validation
[ ] Kubernetes schema validation
[ ] security scanning
[ ] policy checks
[ ] rendered manifest review
[ ] chart package created
[ ] artifact version immutable
[ ] image digest verified
[ ] dependency versions pinned
[ ] automated tests passed
```

---

# 176. Production Readiness Checklist — GitOps

```text
[ ] Git is source of truth
[ ] environment configuration version controlled
[ ] chart version pinned
[ ] image version pinned
[ ] promotion workflow defined
[ ] Argo CD owns resources
[ ] no manual drift
[ ] rollback procedure documented
[ ] production approval defined
[ ] audit trail available
```

---

# 177. Production Readiness Checklist — Security

```text
[ ] no plaintext secrets
[ ] registry allowlist
[ ] image vulnerability policy
[ ] non-root
[ ] no privileged containers
[ ] no unnecessary host access
[ ] least-privilege RBAC
[ ] NetworkPolicy
[ ] Pod Security Admission
[ ] admission policy
[ ] chart dependency review
[ ] artifact integrity
```

---

# 178. Production Readiness Checklist — Operations

```text
[ ] rollout strategy reviewed
[ ] PDB tested
[ ] drain behavior tested
[ ] HPA tested
[ ] failure scenarios tested
[ ] rollback tested
[ ] logs available
[ ] metrics available
[ ] alerts available
[ ] dashboards available
[ ] runbook available
```

---

# 179. Complete Helm Deployment Flow

```text
Developer
    |
    v
Git commit
    |
    v
CI
    |
    +--> unit tests
    +--> helm lint
    +--> schema validation
    +--> helm template
    +--> security scan
    +--> policy checks
    |
    v
Build application image
    |
    v
Push image to ECR
    |
    v
Package Helm chart
    |
    v
Publish chart OCI artifact
    |
    v
Update GitOps repository
    |
    v
Argo CD detects change
    |
    v
Helm rendering
    |
    v
Kubernetes API
    |
    v
EKS
    |
    v
Deployment rollout
    |
    v
Health checks
    |
    v
Prometheus/Grafana/Logs
```

---

# 180. RoboShop Helm Architecture

Example:

```text
RoboShop GitOps
       |
       +----------------+
       |                |
       v                v
cart chart         catalogue chart
       |                |
       v                v
Deployment          Deployment
Service             Service
HPA                 HPA
PDB                 PDB
ConfigMap           ConfigMap
       |                |
       +-------+--------+
               |
               v
              EKS
```

Each microservice can have its own release/configuration while sharing platform standards.

---

# 181. Example RoboShop Cart Values

```yaml
replicaCount: 3

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
  tag: "1.0.0"

service:
  port: 8080
  targetPort: http

env:
  LOG_LEVEL: INFO

resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10

pdb:
  enabled: true
  minAvailable: 2
```

---

# 182. Example Production Override

```yaml
replicaCount: 5

image:
  tag: "1.2.3"

resources:
  requests:
    cpu: 300m
    memory: 512Mi
  limits:
    cpu: "1"
    memory: 1Gi

autoscaling:
  enabled: true
  minReplicas: 5
  maxReplicas: 20

pdb:
  enabled: true
  minAvailable: 4
```

The production override changes capacity and image version without changing chart templates.

---

# 183. Chart Repository Model

Possible repository layout:

```text
helm-charts/
├── charts/
│   ├── cart/
│   ├── catalogue/
│   ├── payment/
│   └── shipping/
├── library/
│   └── platform-common/
└── docs/
```

Alternative:

```text
one repository per application
```

Both can work.

Choose based on team ownership and release independence.

---

# 184. Monorepo vs Multi-Repo

Monorepo:

```text
helm/
  cart/
  catalogue/
  payment/
```

Advantages:

- centralized standards
- simple discovery
- shared tooling

Disadvantages:

- larger change surface
- permissions can be harder
- unrelated releases share repository workflows

Multi-repo:

```text
cart-chart
catalogue-chart
payment-chart
```

Advantages:

- independent ownership
- independent release lifecycle

Disadvantages:

- standardization requires more automation

---

# 185. Chart Release Strategy

Recommended:

```text
PR
 |
 v
validation
 |
 v
merge
 |
 v
chart package
 |
 v
versioned artifact
 |
 v
registry
```

Do not overwrite an existing production chart version.

Immutable artifacts make rollback safer.

---

# 186. Chart Compatibility Matrix

Maintain awareness of:

```text
Chart version
Application version
Kubernetes version
EKS version
CRD/controller versions
```

Example:

| Chart | App | Kubernetes | Status |
|---|---|---|---|
| 1.3.x | 2.1.x | 1.31 | supported |
| 1.2.x | 2.0.x | 1.30 | supported |
| 1.0.x | 1.x | 1.28 | legacy |

This is especially important for long-lived enterprise platforms.

---

# 187. Upgrade Testing

Before production chart upgrade:

```text
render
  |
  v
static validation
  |
  v
security checks
  |
  v
deploy test environment
  |
  v
integration tests
  |
  v
upgrade staging
  |
  v
rollback test
  |
  v
production
```

Test rollback before you need it.

---

# 188. Helm Disaster Recovery

Helm is not a backup system.

Back up:

- Git repositories
- chart artifacts
- GitOps state
- critical configuration
- external secret definitions
- Kubernetes data separately

For disaster recovery:

```text
Git
+
artifact registry
+
infrastructure code
+
secrets source
+
persistent data backups
```

must all be recoverable.

---

# 189. Rebuilding a Cluster

A good GitOps model should allow:

```text
New EKS cluster
       |
       v
Terraform
       |
       v
platform controllers
       |
       v
Argo CD
       |
       v
GitOps repository
       |
       v
Helm charts
       |
       v
applications
```

This is one of the strongest reasons to keep configuration declarative.

---

# 190. Helm and Disaster Recovery Principle

Do not depend on:

```text
manual helm history
```

as your only recovery source.

The desired configuration should exist in:

```text
Git
```

and artifacts should exist in:

```text
registry
```

Then the platform can be reconstructed.

---

# 191. Production Incident Example

Scenario:

```text
Cart version 2.4.0 deployed
```

Symptoms:

```text
5xx increased
latency increased
pods restarting
```

Response:

```text
1. Confirm deployment change.
2. Check Argo CD sync history.
3. Check Helm rendered configuration.
4. Check pod logs.
5. Check probes.
6. Compare image version.
7. Revert GitOps commit.
8. Confirm Argo CD sync.
9. Monitor recovery.
10. Preserve evidence.
```

---

# 192. Incident Example — Bad Values

Production values accidentally contain:

```yaml
service:
  targetPort: 9090
```

Application listens on:

```text
8080
```

Result:

```text
Service
  |
  v
targetPort 9090
  |
  X
Pod listens 8080
```

Detection:

```bash
kubectl get svc
kubectl describe svc
kubectl get endpointslice
```

Fix:

```text
correct Git value
+
validate
+
deploy
```

---

# 193. Incident Example — Wrong Selector

Service:

```yaml
selector:
  app: cart
```

Pods:

```yaml
labels:
  app: shopping-cart
```

Result:

```text
Service
  |
  X
No endpoints
```

This is why labels must be standardized through helpers.

---

# 194. Incident Example — Bad Resource Limit

Values:

```yaml
limits:
  memory: 128Mi
```

Application normally requires:

```text
400Mi
```

Result:

```text
OOMKilled
```

Troubleshooting:

```bash
kubectl describe pod
kubectl get pod -o json
```

Correct solution:

```text
measure actual usage
+
set appropriate request/limit
```

not simply:

```text
remove limits
```

---

# 195. Incident Example — Probe Too Aggressive

Application startup:

```text
90 seconds
```

Probe:

```text
initialDelaySeconds: 5
failureThreshold: 3
periodSeconds: 10
```

Potential result:

```text
liveness kills application
before startup completes
```

Solution:

```text
startupProbe
```

or correctly tuned startup/readiness behavior.

---

# 196. Incident Example — PDB Too Strict

```text
replicas = 3
minAvailable = 3
```

Node maintenance requires one pod eviction.

Eviction is blocked.

Correct design might be:

```text
minAvailable = 2
```

depending on actual availability requirements.

---

# 197. Senior Interview Question 1

### Why use Helm if Kubernetes already uses YAML?

Answer:

> Kubernetes consumes manifests but does not provide a native application packaging and templating model equivalent to Helm. Helm lets us package related Kubernetes resources, version them, parameterize environment-specific configuration, manage dependencies, and maintain repeatable releases. In our production architecture, Helm handles application packaging while Argo CD handles GitOps reconciliation.

---

# 198. Senior Interview Question 2

### What is the difference between chart version and app version?

Answer:

> Chart version identifies the packaging and template version. App version identifies the application version represented by the chart. A chart can change because of a Kubernetes template, security, or configuration change without changing the application image version.

---

# 199. Senior Interview Question 3

### How do you manage dev, staging and production?

Answer:

> We keep a reusable application chart and separate environment-specific values in the GitOps repository. The same versioned chart is promoted across environments, while each environment controls configuration such as replica counts, resources, autoscaling and image version. Argo CD reconciles each environment from Git.

---

# 200. Senior Interview Question 4

### Do you store secrets in Helm values?

Answer:

> No. We avoid plaintext production secrets in Helm values or Git. In AWS, secrets can be stored in Secrets Manager or Parameter Store and synchronized through an approved external-secrets mechanism. Helm templates the reference to the secret rather than embedding the credential.

---

# 201. Senior Interview Question 5

### How do you validate Helm charts?

Answer:

> We validate at multiple layers: Helm linting, values schema validation, template rendering, Kubernetes API/schema validation, security scanning, policy checks and integration testing. For production, I also review the rendered diff before allowing the GitOps promotion.

---

# 202. Senior Interview Question 6

### How do you rollback a Helm deployment?

Answer:

> If Helm is the direct deployment mechanism, I can use Helm history and rollback to a previous revision. In our GitOps architecture, I prefer reverting the GitOps change to the known-good chart/image version and allowing Argo CD to reconcile. That keeps Git consistent with the actual cluster state.

---

# 203. Senior Interview Question 7

### What happens if the application database schema changed?

Answer:

> A Helm rollback does not automatically roll back database state. I use backward-compatible migration patterns, typically expand-and-contract migrations, so the old and new application versions can coexist during deployment and rollback. Database rollback is handled as a separate controlled data operation.

---

# 204. Senior Interview Question 8

### Why not use one giant Helm chart for all microservices?

Answer:

> A giant chart creates excessive conditional logic and couples independent services. I prefer focused application charts with standardized helpers or a library chart for common patterns. Platform components are managed separately from application charts.

---

# 205. Senior Interview Question 9

### How do you prevent bad Helm values from reaching production?

Answer:

> We use schema validation, CI rendering, Kubernetes schema validation, security and policy checks, and environment-specific review. Admission policies provide a final cluster-side guardrail. Production deployment happens through GitOps, so changes are auditable.

---

# 206. Senior Interview Question 10

### What is the difference between Helm and Argo CD?

Answer:

> Helm is primarily a Kubernetes packaging and templating/release tool. Argo CD is a GitOps controller that continuously reconciles the desired state from Git with the Kubernetes cluster. Argo CD can use Helm to render applications, but Helm itself is not a GitOps reconciliation controller.

---

# 207. Senior Interview Question 11

### What causes Helm YAML parse errors?

Answer:

> Common causes are incorrect indentation, malformed Go-template expressions, incorrect quoting, invalid list structure and conditional blocks producing malformed YAML. I first run `helm template --debug` and inspect the rendered manifest before investigating the Kubernetes cluster.

---

# 208. Senior Interview Question 12

### How do you handle chart dependencies?

Answer:

> I pin dependency versions, update them deliberately, review their security and compatibility, and test the resulting chart. Shared platform components are generally managed separately rather than bundling infrastructure dependencies into every application chart.

---

# 209. Senior Interview Question 13

### How do you secure Helm charts?

Answer:

> I avoid plaintext secrets, enforce non-root and restricted security contexts, use least-privilege RBAC, scan rendered manifests, validate policies in CI, use trusted image registries, pin versions, review dependencies and protect the chart/artifact supply chain.

---

# 210. Senior Interview Question 14

### How do you handle chart upgrades during Kubernetes upgrades?

Answer:

> I maintain a compatibility matrix, scan for deprecated APIs, render charts against the target Kubernetes version, test them in a non-production cluster, and upgrade platform CRDs/controllers where required. I do not assume a chart that worked on an older EKS version will automatically work after an API removal.

---

# 211. Senior Interview Question 15

### What is a production-ready Helm chart?

Answer:

> A production-ready chart has predictable naming, stable selectors, validated values, immutable images, resource requests, appropriate probes, secure security contexts, least-privilege identity, availability controls such as PDB, optional autoscaling, observability integration, externalized secrets, CI validation, documented upgrades and a tested rollback strategy.

---

# 212. Production Helm Standards

For this capstone, the recommended standard is:

```text
Helm
 |
 +--> reusable application packaging
 |
 +--> immutable versions
 |
 +--> schema validation
 |
 +--> secure defaults
 |
 +--> environment values
 |
 +--> CI validation
 |
 +--> OCI artifact registry
 |
 +--> GitOps integration
 |
 +--> Argo CD reconciliation
```

---

# 213. Final Architecture

```text
                    ┌──────────────────────┐
                    │      Developer       │
                    └──────────┬───────────┘
                               |
                               v
                    ┌──────────────────────┐
                    │       Git Source     │
                    └──────────┬───────────┘
                               |
                               v
                    ┌──────────────────────┐
                    │         CI           │
                    │ lint/schema/security │
                    └──────────┬───────────┘
                               |
                 +-------------+-------------+
                 |                           |
                 v                           v
        ┌────────────────┐        ┌────────────────┐
        │ Docker Image   │        │  Helm Package   │
        │      ECR       │        │    OCI/ECR      │
        └────────────────┘        └───────┬────────┘
                                          |
                                          v
                                 ┌────────────────┐
                                 │ GitOps Repo    │
                                 │ env values     │
                                 └───────┬────────┘
                                         |
                                         v
                                  ┌──────────────┐
                                  │   Argo CD    │
                                  └──────┬───────┘
                                         |
                                         v
                                  ┌──────────────┐
                                  │ Helm Render   │
                                  └──────┬───────┘
                                         |
                                         v
                                  ┌──────────────┐
                                  │ Kubernetes    │
                                  │ EKS           │
                                  └──────┬───────┘
                                         |
                       +-----------------+----------------+
                       |                 |                |
                       v                 v                v
                  Deployment          Service          Ingress
                       |                 |                |
                       v                 v                v
                     Pods             DNS/traffic      ALB/WAF
                       |
          +------------+-------------+
          |            |             |
          v            v             v
      Prometheus     Logs         Traces
          |            |             |
          +------------+-------------+
                       |
                       v
                 Operations Team
```

---

# 214. Key Takeaways

Remember these production principles:

1. Helm packages Kubernetes applications.
2. Chart version and application version are different.
3. Keep templates reusable and values explicit.
4. Use schema validation.
5. Never store production plaintext secrets in Git.
6. Use immutable image versions.
7. Define resource requests.
8. Use meaningful readiness, liveness and startup probes.
9. Protect availability with appropriate PDBs.
10. Use least-privilege RBAC.
11. Keep platform and application ownership separate.
12. Validate rendered manifests in CI.
13. Pin dependencies.
14. Store immutable chart artifacts.
15. Use OCI registries such as ECR where appropriate.
16. Let Git remain the source of truth in GitOps.
17. Let Argo CD perform reconciliation.
18. Prefer Git-based rollback in a GitOps workflow.
19. Test upgrades and rollback before production.
20. Design charts for recovery, not only initial deployment.

---

# 215. Capstone Implementation Target

The final DevOps capstone should implement:

```text
01. Application source
02. Docker image
03. ECR image repository
04. Helm chart
05. Helm validation
06. OCI chart artifact
07. GitOps values
08. Argo CD Application
09. EKS deployment
10. Service
11. ALB Ingress
12. HPA
13. PDB
14. ServiceAccount
15. External secret reference
16. NetworkPolicy
17. Prometheus integration
18. Grafana visibility
19. ELK logging
20. Alerting
21. Rollback
22. Disaster recovery
23. Production runbook
```

This Helm chapter therefore acts as the bridge between the Kubernetes platform chapter and the CI/CD + GitOps chapters that follow.

---

# 216. Next Capstone Integration

The next stages will build on this chapter:

```text
09 Kubernetes Platform
          |
          v
10 Helm Application Packaging
          |
          v
11 CI Pipeline
          |
          v
12 DevSecOps Pipeline
          |
          v
13 GitOps Repository
          |
          v
14 ArgoCD Deployment
          |
          v
15 Multi-Environment Deployment
```

The key architectural chain is:

```text
Code
  |
  v
CI
  |
  v
Image
  |
  v
ECR
  |
  v
Helm
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

This is the production deployment model used throughout the remaining capstone.

---

# 217. Final Senior-Level Mental Model

When explaining Helm in an interview, think in five layers:

```text
1. Packaging
   Helm Chart

2. Configuration
   values.yaml + environment values

3. Validation
   lint + schema + rendering + security/policy

4. Distribution
   OCI registry / ECR

5. Deployment
   Argo CD + Kubernetes
```

Then connect it to operational concerns:

```text
Security
Availability
Observability
Scalability
Rollback
Disaster Recovery
Auditability
```

That is the difference between knowing Helm commands and understanding Helm as part of a production DevOps platform.
