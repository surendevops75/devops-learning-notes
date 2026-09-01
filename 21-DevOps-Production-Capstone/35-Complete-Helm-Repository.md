# 35 — Complete Helm Repository

This chapter defines a production-oriented Helm repository for the RoboShop-style microservices platform running on AWS EKS.

The design is intentionally production-focused: reusable charts, environment overlays, immutable images, health probes, resources, HPA, PDB, security contexts, ALB Ingress, NetworkPolicy, Prometheus integration, GitOps with Argo CD, CI validation, rollback and troubleshooting.

Responsibility boundaries:

- Terraform: AWS and EKS infrastructure.
- Helm: Kubernetes application packaging and templating.
- GitOps repository: desired deployment configuration.
- Argo CD: reconciliation.
- EKS/Kubernetes: runtime orchestration.
- ECR: container images and OCI artifacts.
- Prometheus/Grafana: metrics and visualization.
- ELK: centralized logging.

---

# 1. Production Architecture

```text
Developer
   |
   v
Application Repository
   |
   v
CI Pipeline
   +--> Build/Test
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   +--> Docker build
   +--> ECR push
   |
   v
GitOps Repository
   |
   v
Argo CD
   |
   v
Helm
   |
   v
AWS EKS
   +--> Applications
   +--> Services
   +--> ALB Ingress
   +--> HPA/PDB
   +--> Prometheus
   +--> Grafana
   +--> ELK
```

For production, CI should normally update GitOps configuration rather than directly running `kubectl apply` against production.

---

# 2. Repository Structure

```text
helm-production/
├── README.md
├── charts/
│   └── roboshop/
│       ├── Chart.yaml
│       ├── values.yaml
│       ├── values.schema.json
│       ├── templates/
│       │   ├── _helpers.tpl
│       │   ├── deployment.yaml
│       │   ├── service.yaml
│       │   ├── serviceaccount.yaml
│       │   ├── configmap.yaml
│       │   ├── ingress.yaml
│       │   ├── hpa.yaml
│       │   ├── pdb.yaml
│       │   ├── networkpolicy.yaml
│       │   ├── servicemonitor.yaml
│       │   ├── prometheusrule.yaml
│       │   └── NOTES.txt
│       └── examples/
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
├── scripts/
│   ├── lint.sh
│   ├── render.sh
│   ├── validate.sh
│   ├── package.sh
│   └── diff.sh
└── .github/workflows/helm-validation.yaml
```

A reusable chart is preferred when services share the same deployment contract. A separate chart per service is also valid when services have materially different lifecycle requirements.

---

# 3. Chart.yaml

```yaml
apiVersion: v2
name: roboshop
description: Production Helm chart for RoboShop microservices
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - roboshop
  - microservices
  - kubernetes
  - eks
maintainers:
  - name: platform-team
kubeVersion: ">=1.29.0-0"
dependencies: []
```

`version` is the chart version. `appVersion` describes the application release. They are independent.

Example:

```text
chart: 1.4.2
application: 2.8.4
image: catalogue@sha256:<digest>
```

---

# 4. Production values.yaml

```yaml
global:
  environment: prod
  region: ap-south-1

nameOverride: ""
fullnameOverride: ""

replicaCount: 3

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop
  tag: "2026.08.31-8f31c1a"
  digest: ""
  pullPolicy: IfNotPresent

imagePullSecrets: []

serviceAccount:
  create: true
  automount: true
  annotations: {}
  name: ""

podAnnotations: {}
podLabels: {}

podSecurityContext:
  runAsNonRoot: true
  seccompProfile:
    type: RuntimeDefault

securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL

service:
  type: ClusterIP
  port: 8080
  targetPort: http

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 15
  targetCPUUtilizationPercentage: 65
  targetMemoryUtilizationPercentage: 75

podDisruptionBudget:
  enabled: true
  minAvailable: 2

networkPolicy:
  enabled: true

config:
  enabled: true
  data: {}

env: []
envFrom: []

livenessProbe:
  enabled: true
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 20
  periodSeconds: 10
  timeoutSeconds: 3
  failureThreshold: 3

readinessProbe:
  enabled: true
  httpGet:
    path: /health
    port: http
  initialDelaySeconds: 10
  periodSeconds: 5
  timeoutSeconds: 2
  failureThreshold: 3

startupProbe:
  enabled: true
  httpGet:
    path: /health
    port: http
  periodSeconds: 5
  failureThreshold: 30

ingress:
  enabled: false
  className: alb
  annotations: {}
  hosts: []
  tls: []

monitoring:
  serviceMonitor:
    enabled: false
    labels: {}
  prometheusRule:
    enabled: false

nodeSelector: {}
tolerations: []
affinity: {}
topologySpreadConstraints: []

volumes: []
volumeMounts: []

terminationGracePeriodSeconds: 30
```

---

# 5. Helpers

```gotemplate
{{- define "roboshop.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "roboshop.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name (include "roboshop.name" .) | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}

{{- define "roboshop.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "roboshop.selectorLabels" -}}
app.kubernetes.io/name: {{ include "roboshop.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{- define "roboshop.labels" -}}
helm.sh/chart: {{ include "roboshop.chart" . }}
{{ include "roboshop.selectorLabels" . }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
app.kubernetes.io/part-of: roboshop
{{- end }}

{{- define "roboshop.serviceAccountName" -}}
{{- if .Values.serviceAccount.create }}
{{- default (include "roboshop.fullname" .) .Values.serviceAccount.name }}
{{- else }}
{{- default "default" .Values.serviceAccount.name }}
{{- end }}
{{- end }}
```

---

# 6. Deployment Template

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "roboshop.fullname" . }}
  labels:
    {{- include "roboshop.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  revisionHistoryLimit: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  selector:
    matchLabels:
      {{- include "roboshop.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "roboshop.selectorLabels" . | nindent 8 }}
        {{- with .Values.podLabels }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
        {{- with .Values.podAnnotations }}
        {{- toYaml . | nindent 8 }}
        {{- end }}
    spec:
      terminationGracePeriodSeconds: {{ .Values.terminationGracePeriodSeconds }}
      serviceAccountName: {{ include "roboshop.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}{{ if .Values.image.digest }}@{{ .Values.image.digest }}{{ else }}:{{ .Values.image.tag }}{{ end }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: {{ .Values.service.targetPort }}
              protocol: TCP
          {{- if .Values.livenessProbe.enabled }}
          livenessProbe:
            {{- omit .Values.livenessProbe "enabled" | toYaml | nindent 12 }}
          {{- end }}
          {{- if .Values.readinessProbe.enabled }}
          readinessProbe:
            {{- omit .Values.readinessProbe "enabled" | toYaml | nindent 12 }}
          {{- end }}
          {{- if .Values.startupProbe.enabled }}
          startupProbe:
            {{- omit .Values.startupProbe "enabled" | toYaml | nindent 12 }}
          {{- end }}
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
          env:
            {{- toYaml .Values.env | nindent 12 }}
          envFrom:
            {{- toYaml .Values.envFrom | nindent 12 }}
          {{- with .Values.volumeMounts }}
          volumeMounts:
            {{- toYaml . | nindent 12 }}
          {{- end }}
      {{- with .Values.volumes }}
      volumes:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.topologySpreadConstraints }}
      topologySpreadConstraints:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

`maxUnavailable: 0` and `maxSurge: 1` favor availability during a normal rolling update. This still depends on sufficient cluster capacity.

---

# 7. Service Template

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "roboshop.fullname" . }}
  labels:
    {{- include "roboshop.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - name: http
      port: {{ .Values.service.port }}
      targetPort: {{ .Values.service.targetPort }}
      protocol: TCP
  selector:
    {{- include "roboshop.selectorLabels" . | nindent 4 }}
```

Most internal microservices should use `ClusterIP`. Public exposure is handled by AWS ALB Ingress rather than creating a public load balancer for every service.

---

# 8. ServiceAccount and EKS Identity

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ include "roboshop.serviceAccountName" . }}
  labels:
    {{- include "roboshop.labels" . | nindent 4 }}
  {{- with .Values.serviceAccount.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
automountServiceAccountToken: {{ .Values.serviceAccount.automount }}
```

Applications that need AWS permissions should use an approved EKS workload identity mechanism instead of static AWS access keys.

Example reference:

```yaml
serviceAccount:
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/roboshop-payment
```

Terraform should create and control the IAM role. Helm references it.

---

# 9. ConfigMaps and Secrets

Non-sensitive configuration belongs in ConfigMaps:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "roboshop.fullname" . }}-config
data:
  LOG_LEVEL: {{ .Values.config.data.LOG_LEVEL | quote }}
  APP_ENV: {{ .Values.global.environment | quote }}
```

Secrets must not be hard-coded in Git.

Reference a Kubernetes Secret:

```yaml
envFrom:
  - secretRef:
      name: payment-secrets
```

Or:

```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: payment-db
        key: password
```

Preferred enterprise flow:

```text
AWS Secrets Manager
        |
        v
External Secrets / approved integration
        |
        v
Kubernetes Secret
        |
        v
Pod
```

Git contains references, not secret values.

---

# 10. Health Probes

Use the probes for different purposes:

```text
startupProbe  -> allows slow applications to initialize
readinessProbe -> controls traffic eligibility
livenessProbe -> detects an unhealthy process
```

A readiness failure should normally remove a pod from Service endpoints without restarting the container.

Do not make liveness depend on an external database unless there is a strong reason. Otherwise a database outage can cause every application pod to restart repeatedly.

Example:

```yaml
startupProbe:
  httpGet:
    path: /health
    port: http
  periodSeconds: 5
  failureThreshold: 30
```

This gives approximately 150 seconds of startup probing before failure.

---

# 11. Resources and Scheduling

Production containers should define requests and limits:

```yaml
resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi
```

Requests influence scheduling and capacity planning.

Limits constrain resource consumption.

Common CPU units:

```text
1000m = 1 CPU
500m  = 0.5 CPU
250m  = 0.25 CPU
```

Memory should be selected from load testing and observed production behavior rather than copied blindly.

Without requests, HPA behavior and cluster capacity planning become less predictable.

---

# 12. HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "roboshop.fullname" . }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "roboshop.fullname" . }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 60
        - type: Pods
          value: 4
          periodSeconds: 60
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 25
          periodSeconds: 60
      selectPolicy: Min
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: {{ .Values.autoscaling.targetMemoryUtilizationPercentage }}
```

A slower scale-down helps avoid oscillation during fluctuating traffic. HPA is not a substitute for node-level capacity management.

---

# 13. PodDisruptionBudget

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: {{ include "roboshop.fullname" . }}
spec:
  minAvailable: {{ .Values.podDisruptionBudget.minAvailable }}
  selector:
    matchLabels:
      {{- include "roboshop.selectorLabels" . | nindent 6 }}
```

For three replicas:

```yaml
minAvailable: 2
```

PDB protects against certain voluntary disruptions. It does not protect against application crashes, node failure, AZ outage, or a bad deployment.

---

# 14. Topology and High Availability

Preferred production placement:

```yaml
affinity:
  podAntiAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          topologyKey: kubernetes.io/hostname
          labelSelector:
            matchLabels:
              app.kubernetes.io/name: roboshop
```

For stronger AZ distribution:

```yaml
topologySpreadConstraints:
  - maxSkew: 1
    topologyKey: topology.kubernetes.io/zone
    whenUnsatisfiable: DoNotSchedule
    labelSelector:
      matchLabels:
        app.kubernetes.io/name: roboshop
```

Strict constraints require sufficient nodes across AZs. Otherwise pods can remain Pending.

---

# 15. ALB Ingress

This capstone uses AWS ALB Ingress, not API Gateway.

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "roboshop.fullname" . }}
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: "443"
    alb.ingress.kubernetes.io/certificate-arn: "arn:aws:acm:REGION:ACCOUNT:certificate/REDACTED"
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
                name: {{ include "roboshop.fullname" . }}
                port:
                  number: {{ .Values.service.port }}
```

The AWS Load Balancer Controller translates the Ingress resource into ALB configuration. IAM, subnet discovery, controller health and security groups must be validated separately.

---

# 16. ALB TLS

Production should terminate HTTPS at the ALB.

Typical annotations:

```yaml
alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
alb.ingress.kubernetes.io/ssl-redirect: "443"
alb.ingress.kubernetes.io/certificate-arn: "arn:aws:acm:REGION:ACCOUNT:certificate/REDACTED"
```

Do not hard-code real production certificate identifiers in a reusable base chart. Put environment-specific values in GitOps configuration.

---

# 17. NetworkPolicy

A production application may need:

```text
DNS
other microservices
Redis
RabbitMQ
databases
AWS endpoints
external APIs
```

Therefore policies must be designed from actual dependency graphs.

Example baseline:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: {{ include "roboshop.fullname" . }}
spec:
  podSelector:
    matchLabels:
      {{- include "roboshop.selectorLabels" . | nindent 6 }}
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
          port: {{ .Values.service.targetPort }}
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: roboshop
    - to:
        - namespaceSelector: {}
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

Never introduce a deny-all policy into production without validating every dependency.

---

# 18. Prometheus ServiceMonitor

If Prometheus Operator is used:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: {{ include "roboshop.fullname" . }}
  labels:
    {{- include "roboshop.labels" . | nindent 4 }}
    {{- toYaml .Values.monitoring.serviceMonitor.labels | nindent 4 }}
spec:
  selector:
    matchLabels:
      {{- include "roboshop.selectorLabels" . | nindent 6 }}
  endpoints:
    - port: metrics
      path: /metrics
      interval: 30s
      scrapeTimeout: 10s
```

The Service must expose a `metrics` port and the application must actually provide Prometheus-compatible metrics.

---

# 19. PrometheusRule

Application-level rules can live with the application chart:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: {{ include "roboshop.fullname" . }}
  labels:
    {{- include "roboshop.labels" . | nindent 4 }}
spec:
  groups:
    - name: roboshop-{{ .Release.Name }}
      rules:
        - alert: RoboshopHighErrorRate
          expr: |
            (
              sum(rate(http_requests_total{
                namespace="{{ .Release.Namespace }}",
                status=~"5.."
              }[5m]))
              /
              sum(rate(http_requests_total{
                namespace="{{ .Release.Namespace }}"
              }[5m]))
            ) > 0.05
          for: 10m
          labels:
            severity: critical
            team: application
            environment: prod
          annotations:
            summary: "High HTTP error rate"
            description: "HTTP 5xx error rate exceeded 5% for 10 minutes."
            runbook_url: "https://runbooks.example.com/roboshop/high-error-rate"
```

Use application charts for application-owned alerts. Keep node, API-server and cluster-wide alerts in the platform monitoring layer to avoid duplicates.

---

# 20. Recording Rules

Repeated expensive PromQL calculations can be converted into recording rules.

Example:

```yaml
- record: roboshop:http_requests:rate5m
  expr: sum(rate(http_requests_total[5m]))
```

Then dashboards and alerts can query:

```promql
roboshop:http_requests:rate5m
```

Recording rules improve query reuse and can reduce repeated computation.

---

# 21. SLO-Based Alerting

Resource alerts are useful, but user-impact alerts are usually more important.

Examples:

```text
availability SLO
latency SLO
error-rate SLO
successful transaction rate
```

A service with CPU at 85% is not necessarily broken.

A service with 8% failed customer requests is immediately relevant.

For mature environments, use SLI/SLO and burn-rate alerting so pages are tied to actual reliability impact.

---

# 22. Environment Values

Dev:

```yaml
global:
  environment: dev
replicaCount: 1
autoscaling:
  enabled: false
podDisruptionBudget:
  enabled: false
```

QA:

```yaml
global:
  environment: qa
replicaCount: 2
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 5
```

Prod:

```yaml
global:
  environment: prod
replicaCount: 3
autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 20
podDisruptionBudget:
  enabled: true
  minAvailable: 2
```

Do not copy the complete base values file into every environment. Keep common defaults in `values.yaml` and override only what changes.

---

# 23. Immutable Images

Avoid:

```yaml
tag: latest
```

Prefer:

```yaml
tag: "2026.08.31-8f31c1a"
```

or an image digest:

```yaml
digest: "sha256:REDACTED"
```

An immutable reference makes the deployment reproducible and makes rollback deterministic.

A production release should be traceable to:

```text
Git commit
CI build
image digest
ECR repository
Helm chart version
GitOps revision
Argo CD application revision
```

---

# 24. Argo CD Integration

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-catalogue-prod
  namespace: argocd
spec:
  project: roboshop
  source:
    repoURL: https://git.example.com/platform/gitops.git
    targetRevision: main
    path: apps/catalogue
    helm:
      valueFiles:
        - values-prod.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

The exact path can differ. The key principle is:

```text
Git = desired state
Argo CD = reconciler
Helm = renderer
Kubernetes = runtime
```

---

# 25. GitOps Rollback

For an Argo CD-managed application:

```text
bad GitOps commit
      |
      v
identify last good revision
      |
      v
git revert
      |
      v
review + merge
      |
      v
Argo CD sync
      |
      v
previous known-good application
```

A manual `helm rollback` may temporarily change the cluster while Git still describes the bad state. Fix the source of truth for a durable rollback.

---

# 26. Helm Validation

Run:

```bash
helm lint charts/roboshop   -f environments/prod/roboshop-values.yaml
```

Render:

```bash
helm template roboshop-prod   charts/roboshop   -n roboshop   -f environments/prod/roboshop-values.yaml   > rendered.yaml
```

Validate:

```bash
kubectl apply   --dry-run=server   --validate=true   -f rendered.yaml
```

The sequence matters:

```text
template syntax
   |
   v
rendered manifest
   |
   v
Kubernetes schema/API
   |
   v
policy/security validation
   |
   v
runtime validation
```

---

# 27. values.schema.json

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
      "required": ["repository", "tag"],
      "properties": {
        "repository": {
          "type": "string",
          "minLength": 1
        },
        "tag": {
          "type": "string",
          "minLength": 1
        }
      }
    }
  },
  "required": ["image"]
}
```

Schema validation catches configuration mistakes before they become runtime failures.

---

# 28. Security Baseline

Recommended baseline where supported:

```yaml
podSecurityContext:
  runAsNonRoot: true
  seccompProfile:
    type: RuntimeDefault

securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL
```

Also consider:

```text
Pod Security Standards
NetworkPolicy
resource requests/limits
image scanning
immutable image references
admission policies
RBAC
workload identity
secret management
```

Do not apply a security setting blindly if the application requires a documented exception.

---

# 29. Read-Only Root Filesystem

Some applications need writable temporary directories.

Use an `emptyDir` for temporary state:

```yaml
volumeMounts:
  - name: tmp
    mountPath: /tmp

volumes:
  - name: tmp
    emptyDir: {}
```

Then the root filesystem can remain read-only:

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

This is safer than making the entire container filesystem writable.

---

# 30. Graceful Shutdown

Production applications should handle `SIGTERM`.

Recommended:

```yaml
terminationGracePeriodSeconds: 30
```

Operational sequence:

```text
termination requested
      |
      v
SIGTERM
      |
      v
stop accepting new work
      |
      v
finish in-flight requests
      |
      v
process exits
```

This reduces dropped traffic during rolling deployments.

---

# 31. Configuration Checksum

A checksum annotation causes a Deployment rollout when the ConfigMap content changes:

```yaml
annotations:
  checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
```

Without this, changing a ConfigMap does not automatically restart existing pods.

For applications that can dynamically reload configuration, a restart may not be required.

---

# 32. Application Metrics

Useful metrics include:

```text
http_requests_total
http_request_duration_seconds
application_errors_total
database_connection_pool
queue_depth
business_transaction_total
```

For RoboShop services, combine technical metrics with user-impact metrics.

Examples:

```text
catalogue 5xx rate
payment failure rate
cart request latency
shipping processing failures
frontend availability
```

---

# 33. Alert Ownership

Application chart should generally own:

```text
application error rate
application latency
application availability
business transaction failures
application saturation
```

Platform monitoring should own:

```text
nodes
EKS control plane
Kubernetes API
cluster capacity
ALB controller
node disk pressure
node memory pressure
cluster-wide failures
```

Clear ownership prevents duplicate pages and makes incident routing easier.

---

# 34. Helm Testing

A Helm test can perform a simple smoke check:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "roboshop.fullname" . }}-test"
  annotations:
    "helm.sh/hook": test
spec:
  restartPolicy: Never
  containers:
    - name: test
      image: curlimages/curl:8.10.1
      command:
        - curl
        - --fail
        - http://{{ include "roboshop.fullname" . }}:8080/health
```

Run:

```bash
helm test roboshop -n roboshop
```

Use this alongside, not instead of, real smoke tests and SLI monitoring.

---

# 35. CI Quality Gates

Recommended pipeline:

```text
PR
 |
 +--> helm lint
 |
 +--> values schema
 |
 +--> helm template
 |
 +--> Kubernetes validation
 |
 +--> Trivy config scan
 |
 +--> kube-linter / policy checks
 |
 +--> review
 |
 v
merge
 |
 v
chart package
 |
 v
OCI/ECR
 |
 v
GitOps update
 |
 v
Argo CD
```

The chart should fail fast when required values or security requirements are missing.

---

# 36. GitHub Actions Example

```yaml
name: Helm Validation

on:
  pull_request:
    paths:
      - "charts/**"
      - "environments/**"

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Helm
        uses: azure/setup-helm@v4

      - name: Lint
        run: |
          helm lint charts/roboshop             -f environments/prod/roboshop-values.yaml

      - name: Render
        run: |
          helm template roboshop-prod             charts/roboshop             -n roboshop             -f environments/prod/roboshop-values.yaml             > rendered.yaml

      - name: Validate
        run: |
          test -s rendered.yaml
```

Add organization-approved Kubernetes, security and policy validation tools as required.

---

# 37. Jenkins Example

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Helm Lint') {
            steps {
                sh '''
                    helm lint charts/roboshop                       -f environments/prod/roboshop-values.yaml
                '''
            }
        }

        stage('Render') {
            steps {
                sh '''
                    helm template roboshop-prod                       charts/roboshop                       -n roboshop                       -f environments/prod/roboshop-values.yaml                       > rendered.yaml
                '''
            }
        }

        stage('Validation') {
            steps {
                sh 'test -s rendered.yaml'
            }
        }
    }
}
```

---

# 38. Helm Packaging and OCI

Package:

```bash
helm package charts/roboshop
```

Push to an OCI registry:

```bash
helm push roboshop-1.0.0.tgz   oci://123456789012.dkr.ecr.ap-south-1.amazonaws.com/helm
```

Pull:

```bash
helm pull   oci://123456789012.dkr.ecr.ap-south-1.amazonaws.com/helm/roboshop   --version 1.0.0
```

The exact ECR authentication procedure should follow the organization's approved AWS credential and registry-access model.

---

# 39. Troubleshooting — ImagePullBackOff

Symptom:

```text
ImagePullBackOff
```

Investigation:

```bash
kubectl describe pod <pod> -n roboshop

kubectl get deployment catalogue   -n roboshop   -o jsonpath='{.spec.template.spec.containers[0].image}'

aws ecr describe-images   --repository-name roboshop/catalogue   --region ap-south-1
```

Possible root causes:

```text
wrong repository
wrong tag
image absent
ECR authorization
network failure
wrong CPU architecture
```

Prevention:

```text
immutable image references
CI image-existence check
ECR lifecycle policy
workload/node registry access
```

---

# 40. Troubleshooting — CrashLoopBackOff

Investigation:

```bash
kubectl logs <pod> -n roboshop --previous
kubectl describe pod <pod> -n roboshop
```

Check:

```text
application exception
missing environment variable
missing Secret
incorrect command
permission error
OOMKilled
probe failure
dependency failure
```

Do not restart blindly. Capture logs and events first.

---

# 41. Troubleshooting — Pending Pod

Commands:

```bash
kubectl describe pod <pod> -n roboshop
kubectl get nodes
kubectl describe nodes
```

Common causes:

```text
insufficient CPU
insufficient memory
node taint
nodeSelector
affinity
topology spread constraint
quota
storage
```

If the scheduler reports insufficient capacity, check whether node autoscaling can add capacity and whether AWS subnet/instance capacity is available.

---

# 42. Troubleshooting — Probe Failure

Test from inside the container:

```bash
kubectl exec -it <pod> -n roboshop --   curl -v http://127.0.0.1:8080/health
```

Investigate:

```text
wrong port
wrong path
application startup time
dependency behavior
CPU starvation
memory pressure
TLS mismatch
```

Correct the probe to match actual application behavior rather than simply increasing thresholds indefinitely.

---

# 43. Troubleshooting — Service Not Routing

Check:

```bash
kubectl get svc -n roboshop
kubectl get endpointslices -n roboshop
kubectl get pods -n roboshop --show-labels
```

The Service selector must match pod labels.

If endpoints are empty:

```text
selector mismatch
pods not Ready
wrong namespace
wrong port
```

A `Running` pod that is not `Ready` may not appear as a usable endpoint.

---

# 44. Troubleshooting — ALB

Commands:

```bash
kubectl describe ingress catalogue -n roboshop

kubectl logs   -n kube-system   deployment/aws-load-balancer-controller
```

Investigate:

```text
IAM permissions
subnet discovery
security groups
IngressClass
annotations
certificate ARN
target health
health-check path
Service/port
```

If the ALB exists but targets are unhealthy, follow the path:

```text
ALB
 |
 v
target group
 |
 v
pod IP
 |
 v
Service/application port
 |
 v
health endpoint
```

---

# 45. Troubleshooting — HPA

Commands:

```bash
kubectl get hpa -n roboshop
kubectl describe hpa catalogue -n roboshop
kubectl top pods -n roboshop
```

Check:

```text
metrics availability
resource requests
target utilization
min/max replicas
actual workload
```

Do not assume HPA is broken simply because it has not scaled. If utilization is below the configured target, no scale-up is expected.

---

# 46. Troubleshooting — PDB

If node drain is blocked:

```bash
kubectl get pdb -n roboshop
```

Example:

```text
replicas = 3
minAvailable = 3
```

can prevent voluntary eviction of any pod.

A practical production relationship is:

```text
replicas > minAvailable
```

for workloads that must tolerate routine node maintenance.

---

# 47. Troubleshooting — Argo CD Drift

Check:

```bash
argocd app get roboshop-catalogue-prod
argocd app diff roboshop-catalogue-prod
```

Possible causes:

```text
manual kubectl change
Helm values change
generated field differences
resource ownership conflict
CRD/controller mutation
```

If the change is intentional, update Git. If it is not intentional, allow reconciliation to restore desired state.

---

# 48. Rollout Failure

Commands:

```bash
kubectl rollout status deployment/catalogue -n roboshop
kubectl get pods -n roboshop -w
kubectl describe deployment catalogue -n roboshop
```

If new pods never become Ready:

```text
keep healthy old pods
investigate events/logs
fix Git/values/image/application
allow Argo CD to reconcile
```

Do not force-delete healthy old pods during an unresolved production rollout.

---

# 49. Database Migration Strategy

Rolling deployments can temporarily run old and new application versions simultaneously.

Therefore database migrations should prefer an expand-and-contract pattern:

```text
1. Add compatible schema
2. Deploy code supporting old + new schema
3. Backfill
4. Switch traffic/behavior
5. Remove obsolete schema later
```

Avoid deploying code that requires a schema change that has not yet reached every old pod.

---

# 50. Do Not Bundle Everything

Avoid making the application chart own:

```text
VPC
EKS
IAM
KMS
ECR
NAT
Prometheus platform
ELK platform
all databases
all queues
all AWS infrastructure
```

That creates excessive coupling.

The application chart should package application runtime resources. Platform services should have their own lifecycle and ownership.

---

# 51. Chart Dependencies

Dependencies can be declared:

```yaml
dependencies:
  - name: redis
    version: 20.0.0
    repository: oci://registry.example.com/charts
```

Then:

```bash
helm dependency update charts/roboshop
```

For production, prefer independently managed managed-services/platform components when their lifecycle differs significantly from the application.

---

# 52. Semantic Versioning

Use:

```text
MAJOR.MINOR.PATCH
```

Examples:

```text
1.0.0 -> 1.0.1  bug fix
1.0.1 -> 1.1.0  backward-compatible feature
1.1.0 -> 2.0.0  breaking chart change
```

Changing values structure or template behavior in a way that breaks consumers should be treated as a compatibility change.

---

# 53. Production Labels

Recommended Kubernetes labels:

```text
app.kubernetes.io/name
app.kubernetes.io/instance
app.kubernetes.io/version
app.kubernetes.io/component
app.kubernetes.io/part-of
app.kubernetes.io/managed-by
helm.sh/chart
```

Operational labels can include:

```text
environment
team
service
owner
cost-center
```

Do not add high-cardinality identifiers to Prometheus labels without understanding the storage cost.

---

# 54. Production Release Metadata

A release should be traceable:

```text
Application: catalogue
Environment: prod
Chart: roboshop-1.4.2
Image: catalogue@sha256:...
Git commit: 8f31c1a
Argo CD revision: 8f31c1a
```

This supports incident response, compliance and rollback.

---

# 55. Helm vs Kustomize

Helm is primarily template/package oriented.

Kustomize is primarily overlay/patch oriented.

Both are valid.

For this capstone:

```text
Helm -> reusable application packaging
GitOps -> desired state
Argo CD -> reconciliation
```

The important point is consistent ownership rather than using every available tool.

---

# 56. Helm vs Raw YAML

Raw YAML is direct Kubernetes configuration.

Helm adds:

```text
templating
values
packaging
versioning
dependencies
release operations
```

Helm is particularly useful when many services share common deployment patterns but have different environment values.

---

# 57. Helm vs Terraform

Terraform should manage infrastructure:

```text
VPC
EKS
IAM
KMS
ECR
AWS networking
```

Helm should manage Kubernetes application resources:

```text
Deployment
Service
ConfigMap
HPA
PDB
Ingress
NetworkPolicy
application monitoring
```

Terraform can install platform components in some architectures, but ownership boundaries must remain explicit.

---

# 58. Production Run Commands

```bash
helm version
helm list -A
helm lint charts/roboshop
helm template roboshop-prod charts/roboshop -n roboshop -f environments/prod/roboshop-values.yaml
helm package charts/roboshop
helm history roboshop-catalogue -n roboshop
helm status roboshop-catalogue -n roboshop
helm get values roboshop-catalogue -n roboshop
helm get manifest roboshop-catalogue -n roboshop

kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
kubectl logs <pod> -n roboshop --previous
kubectl get svc -n roboshop
kubectl get endpointslices -n roboshop
kubectl get ingress -n roboshop
kubectl get hpa -n roboshop
kubectl get pdb -n roboshop
kubectl rollout status deployment/catalogue -n roboshop

argocd app get roboshop-catalogue-prod
argocd app diff roboshop-catalogue-prod
argocd app history roboshop-catalogue-prod
```

---

# 59. Production Pre-Merge Checklist

```text
[ ] chart version considered
[ ] values schema passes
[ ] helm lint passes
[ ] helm template passes
[ ] rendered manifests reviewed
[ ] server-side validation passes
[ ] security scan passes
[ ] policy checks pass
[ ] no secrets committed
[ ] immutable image reference
[ ] resources defined
[ ] readiness probe reviewed
[ ] liveness probe reviewed
[ ] startup probe reviewed
[ ] HPA reviewed
[ ] PDB reviewed
[ ] NetworkPolicy reviewed
[ ] ALB changes reviewed
[ ] monitoring changes reviewed
[ ] rollback path understood
```

---

# 60. Production Deployment Checklist

```text
[ ] image exists in ECR
[ ] image digest/tag is correct
[ ] GitOps PR approved
[ ] correct cluster selected
[ ] correct namespace selected
[ ] capacity available
[ ] Argo CD project permits deployment
[ ] dashboards available
[ ] alert routing tested
[ ] rollback revision identified
[ ] migration compatibility checked
[ ] ALB target health expected
```

---

# 61. Post-Deployment Checklist

```text
[ ] Argo CD Synced
[ ] Argo CD Healthy
[ ] Deployment available
[ ] all expected pods Ready
[ ] no CrashLoopBackOff
[ ] no ImagePullBackOff
[ ] no unexpected OOMKilled
[ ] ALB targets healthy
[ ] smoke test successful
[ ] error rate normal
[ ] latency normal
[ ] logs normal
[ ] no new critical alerts
```

---

# 62. Failure Scenario — Bad Values

Bad:

```yaml
replicaCount: "three"
```

With schema validation this is rejected before production.

The prevention pattern is:

```text
values.schema.json
   |
   v
helm lint/template
   |
   v
CI failure
```

Fail early instead of discovering invalid configuration during deployment.

---

# 63. Failure Scenario — Wrong Image

Bad:

```yaml
image:
  tag: "1.9.99"
```

If the tag does not exist:

```text
Pod
 |
 v
ImagePullBackOff
```

Prevention:

```text
CI verifies image
immutable release identifier
ECR promotion process
GitOps review
```

---

# 64. Failure Scenario — Missing Secret

Symptom:

```text
CreateContainerConfigError
```

Investigate:

```bash
kubectl describe pod <pod> -n roboshop
kubectl get secret -n roboshop
```

If the secret is missing, repair the secret-management path. Do not put the password directly into the chart as an emergency workaround.

---

# 65. Failure Scenario — ConfigMap Changed

If Git changes a ConfigMap but pods continue using old configuration, use a checksum annotation:

```yaml
checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
```

The changed pod template triggers a new ReplicaSet.

---

# 66. Failure Scenario — HPA Does Not Scale

Check:

```bash
kubectl describe hpa catalogue -n roboshop
kubectl top pods -n roboshop
```

Investigate:

```text
metrics unavailable
resource requests missing
wrong target
traffic not high enough
maxReplicas reached
```

The correct response depends on the evidence, not simply increasing `maxReplicas`.

---

# 67. Failure Scenario — ALB Unhealthy

Trace:

```text
ALB
 |
 v
Target group
 |
 v
Pod IP
 |
 v
Service/application
 |
 v
health endpoint
```

Check the Ingress, AWS Load Balancer Controller events/logs, target health, security groups, Service ports and application readiness.

---

# 68. Failure Scenario — Argo CD OutOfSync

An OutOfSync state can mean:

```text
Git changed
manual cluster change
controller mutation
Helm rendering difference
resource ownership conflict
```

Use:

```bash
argocd app diff roboshop-catalogue-prod
```

Then decide whether Git or the live cluster is wrong. The durable desired state should remain in Git.

---

# 69. Senior Interview — Why Helm?

Strong answer:

Helm provides reusable Kubernetes packaging and templating. In our EKS platform I use it to standardize Deployments, Services, probes, resources, HPA, PDB, ServiceAccounts, ALB Ingress, NetworkPolicy and application monitoring across microservices. Environment-specific configuration is supplied through values. CI validates the chart and Argo CD uses the GitOps state to reconcile Helm-rendered resources into EKS.

---

# 70. Senior Interview — Helm vs YAML?

Kubernetes YAML directly describes resources. Helm adds reusable templates, values, packaging, versioning and dependency management. I use Helm when multiple environments or services share a deployment pattern and need controlled variation.

---

# 71. Senior Interview — Secrets?

I never store production secret values in Helm charts or Git. I use an approved secret manager such as AWS Secrets Manager and expose secrets through a controlled Kubernetes integration. Helm contains references and non-sensitive configuration only.

---

# 72. Senior Interview — Immutable Images?

Mutable tags such as `latest` make deployments non-deterministic. I prefer immutable tags or image digests so a deployment always refers to a known artifact and rollback is reproducible.

---

# 73. Senior Interview — Helm and Argo CD?

Argo CD can use a Helm chart as its source. Helm renders the Kubernetes manifests and Argo CD compares the desired result with live state and reconciles differences. In production I keep Git as the source of truth rather than mixing direct Helm upgrades with Argo CD.

---

# 74. Senior Interview — Rollback?

For a GitOps deployment I identify the last known-good Git revision, revert the bad GitOps change, validate it, merge it through the normal process and allow Argo CD to reconcile. A manual Helm rollback can be useful for diagnosis but should not leave Git and the cluster permanently inconsistent.

---

# 75. Senior Interview — Probes?

Readiness determines whether a pod receives traffic. Liveness determines whether a container should be restarted. Startup gives slow applications time to initialize. I design them from actual application behavior and avoid using liveness as a dependency-health check.

---

# 76. Senior Interview — PDB?

A PodDisruptionBudget protects availability during voluntary disruptions such as node maintenance. It does not protect against node crashes or bad releases, so it must be combined with replicas, multi-AZ scheduling, probes and capacity planning.

---

# 77. Senior Interview — Production Troubleshooting?

I first classify the problem: GitOps, scheduling, container, application, Service, ingress, AWS or observability. Then I use evidence such as Argo CD diff, Kubernetes events, pod logs, endpoint slices, ALB target health and Prometheus metrics. I avoid changing multiple things before identifying the failure mode.

---

# 78. Senior Interview — Database Migration?

Because rolling deployments can run old and new versions simultaneously, I prefer expand-and-contract migrations. First introduce backward-compatible schema, then deploy code that can work with both versions, migrate data, switch behavior and remove obsolete schema later.

---

# 79. Senior Interview — Security?

My baseline includes non-root execution, restricted capabilities, no privilege escalation, read-only filesystems where supported, resource limits, NetworkPolicy, immutable images, secret references, workload identity, chart scanning and policy-as-code. Exceptions are documented and reviewed.

---

# 80. Final Production Mental Model

```text
Terraform
  -> builds AWS/EKS platform

Helm
  -> packages application Kubernetes resources

GitOps
  -> records desired application state

Argo CD
  -> reconciles desired state

EKS
  -> runs workloads

ALB
  -> external HTTP/HTTPS entry point

Prometheus
  -> metrics and alert evaluation

Grafana
  -> visualization

ELK
  -> centralized logging
```

A strong production engineer understands where each responsibility starts and ends.

---

# 81. Complete Definition of Done

```text
[ ] reusable chart
[ ] Chart.yaml
[ ] values.yaml
[ ] values.schema.json
[ ] helper templates
[ ] Deployment
[ ] Service
[ ] ServiceAccount
[ ] ConfigMap
[ ] Ingress
[ ] HPA
[ ] PDB
[ ] NetworkPolicy
[ ] monitoring integration
[ ] PrometheusRule
[ ] environment values
[ ] immutable images
[ ] no credentials in Git
[ ] CI validation
[ ] security scanning
[ ] GitOps integration
[ ] rollback documentation
[ ] troubleshooting documentation
[ ] production checklists
```

---

# 82. Final Summary

Helm is the application packaging and templating layer in this production DevOps platform.

The complete model is:

```text
Terraform -> infrastructure
Helm -> Kubernetes application packaging
GitOps -> desired deployment state
Argo CD -> reconciliation
EKS -> runtime
ALB -> ingress
Prometheus -> metrics/alerts
Grafana -> visualization
ELK -> logs
```

The objective is not merely to reduce YAML. The objective is to make application deployment repeatable, reviewable, secure, observable, environment-aware and rollback-friendly.

For the RoboShop production capstone, every service should follow a consistent operational contract while allowing service-specific resources, configuration, dependencies and scaling behavior.

---

# Appendix A — Production Values Matrix

| Setting | Dev | QA | Prod |
|---|---|---|---|
| Replicas | 1 | 2 | 3+ |
| HPA | Optional | Yes | Yes |
| PDB | Optional | Yes | Yes |
| Resource requests | Yes | Yes | Yes |
| Resource limits | Yes | Yes | Yes |
| ALB | Usually no | Optional | Yes |
| TLS | Optional | Yes | Yes |
| NetworkPolicy | Test | Yes | Yes |
| Prometheus | Yes | Yes | Yes |
| Immutable image | Yes | Yes | Yes |
| External secrets | Optional | Yes | Yes |

---

# Appendix B — Utility Scripts

`lint.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail
CHART_DIR="${1:-charts/roboshop}"
VALUES_FILE="${2:-environments/prod/roboshop-values.yaml}"
helm lint "$CHART_DIR" --values "$VALUES_FILE"
```

`render.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail
CHART_DIR="${1:-charts/roboshop}"
VALUES_FILE="${2:-environments/prod/roboshop-values.yaml}"
OUTPUT="${3:-rendered.yaml}"

helm template roboshop-prod "$CHART_DIR"   --namespace roboshop   --values "$VALUES_FILE" > "$OUTPUT"

test -s "$OUTPUT"
```

`validate.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail
kubectl apply --dry-run=server --validate=true -f "${1:-rendered.yaml}"
```

`package.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail
helm dependency update charts/roboshop
helm lint charts/roboshop
mkdir -p dist
helm package charts/roboshop --destination dist/
```

`diff.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail
helm diff upgrade roboshop-prod charts/roboshop   --namespace roboshop   --values environments/prod/roboshop-values.yaml
```

---

# Appendix C — Production Ownership Matrix

| Area | Primary Owner |
|---|---|
| AWS VPC | Platform/Terraform |
| EKS | Platform/Terraform |
| IAM | Platform/Security |
| ECR | Platform |
| Helm chart | Application/Platform |
| Environment values | GitOps/Application |
| Secrets | Security/Platform |
| Argo CD | Platform |
| Deployment desired state | GitOps |
| Kubernetes runtime | Platform |
| ALB controller | Platform |
| Application metrics | Application |
| Cluster metrics | Platform |
| Application logs | Application/Platform |
| ELK platform | Platform |
| Incident response | On-call team |

---

# Appendix D — Production Decision Tree

```text
Deployment changed
      |
      v
Argo CD Synced?
      |
   +--No--> inspect Git/Argo diff
   |
  Yes
   |
   v
Pods Ready?
   |
 +--No--> describe pod
 |          +--> image problem
 |          +--> crash
 |          +--> pending
 |          +--> probe
 |          +--> config/secret
 |
 Yes
 |
 v
Service endpoints present?
 |
 +--No--> selector/readiness/service
 |
 Yes
 |
 v
ALB targets healthy?
 |
 +--No--> ingress/controller/AWS
 |
 Yes
 |
 v
Application SLIs healthy?
 |
 +--No--> application/dependency/performance
 |
 Yes
 |
 v
Healthy
```

---

# Appendix E — Final Senior-Level Answer

I use Helm as the standardized application packaging layer on EKS. The chart contains reusable production patterns for Deployments, Services, health checks, resource governance, autoscaling, disruption budgets, security contexts, ingress, network controls and application monitoring.

Environment-specific configuration is kept in version-controlled values, while production secrets are referenced from external secret management. Images are immutable and stored in ECR.

CI validates and scans the chart and artifact. The GitOps repository records the desired deployment state, and Argo CD continuously reconciles that state into EKS.

For operations, I validate rendered manifests, monitor rollout health through Kubernetes and Argo CD, use Prometheus/Grafana and ELK for observability, and roll back through Git history so the cluster and desired state remain consistent.

The important part is understanding where Helm fits into the complete production delivery architecture rather than treating Helm as only a YAML templating tool.

---

# Service Example 1 — Catalogue

A production values file for `catalogue` can follow this pattern:

```yaml
global:
  environment: prod

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/catalogue
  tag: "RELEASE"

replicaCount: 3

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 15
  targetCPUUtilizationPercentage: 65

podDisruptionBudget:
  enabled: true
  minAvailable: 2

service:
  type: ClusterIP
  port: 8080
  targetPort: http

env:
  - name: APP_ENV
    value: production
```

Operational review for `catalogue`:

```text
[ ] image exists in ECR
[ ] image is immutable
[ ] readiness endpoint works
[ ] liveness endpoint is safe
[ ] resource requests reflect measurements
[ ] HPA target is realistic
[ ] PDB permits maintenance
[ ] pod placement is HA-aware
[ ] dependencies are covered by NetworkPolicy
[ ] application metrics are exposed where supported
[ ] alerts have an owner
[ ] rollback image is known
```

---

# Service Example 2 — User

A production values file for `user` can follow this pattern:

```yaml
global:
  environment: prod

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/user
  tag: "RELEASE"

replicaCount: 3

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 15
  targetCPUUtilizationPercentage: 65

podDisruptionBudget:
  enabled: true
  minAvailable: 2

service:
  type: ClusterIP
  port: 8080
  targetPort: http

env:
  - name: APP_ENV
    value: production
```

Operational review for `user`:

```text
[ ] image exists in ECR
[ ] image is immutable
[ ] readiness endpoint works
[ ] liveness endpoint is safe
[ ] resource requests reflect measurements
[ ] HPA target is realistic
[ ] PDB permits maintenance
[ ] pod placement is HA-aware
[ ] dependencies are covered by NetworkPolicy
[ ] application metrics are exposed where supported
[ ] alerts have an owner
[ ] rollback image is known
```

---

# Service Example 3 — Cart

A production values file for `cart` can follow this pattern:

```yaml
global:
  environment: prod

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart
  tag: "RELEASE"

replicaCount: 3

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 15
  targetCPUUtilizationPercentage: 65

podDisruptionBudget:
  enabled: true
  minAvailable: 2

service:
  type: ClusterIP
  port: 8080
  targetPort: http

env:
  - name: APP_ENV
    value: production
```

Operational review for `cart`:

```text
[ ] image exists in ECR
[ ] image is immutable
[ ] readiness endpoint works
[ ] liveness endpoint is safe
[ ] resource requests reflect measurements
[ ] HPA target is realistic
[ ] PDB permits maintenance
[ ] pod placement is HA-aware
[ ] dependencies are covered by NetworkPolicy
[ ] application metrics are exposed where supported
[ ] alerts have an owner
[ ] rollback image is known
```

---

# Service Example 4 — Shipping

A production values file for `shipping` can follow this pattern:

```yaml
global:
  environment: prod

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/shipping
  tag: "RELEASE"

replicaCount: 3

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 15
  targetCPUUtilizationPercentage: 65

podDisruptionBudget:
  enabled: true
  minAvailable: 2

service:
  type: ClusterIP
  port: 8080
  targetPort: http

env:
  - name: APP_ENV
    value: production
```

Operational review for `shipping`:

```text
[ ] image exists in ECR
[ ] image is immutable
[ ] readiness endpoint works
[ ] liveness endpoint is safe
[ ] resource requests reflect measurements
[ ] HPA target is realistic
[ ] PDB permits maintenance
[ ] pod placement is HA-aware
[ ] dependencies are covered by NetworkPolicy
[ ] application metrics are exposed where supported
[ ] alerts have an owner
[ ] rollback image is known
```

---

# Service Example 5 — Payment

A production values file for `payment` can follow this pattern:

```yaml
global:
  environment: prod

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/payment
  tag: "RELEASE"

replicaCount: 3

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 15
  targetCPUUtilizationPercentage: 65

podDisruptionBudget:
  enabled: true
  minAvailable: 2

service:
  type: ClusterIP
  port: 8080
  targetPort: http

env:
  - name: APP_ENV
    value: production
```

Operational review for `payment`:

```text
[ ] image exists in ECR
[ ] image is immutable
[ ] readiness endpoint works
[ ] liveness endpoint is safe
[ ] resource requests reflect measurements
[ ] HPA target is realistic
[ ] PDB permits maintenance
[ ] pod placement is HA-aware
[ ] dependencies are covered by NetworkPolicy
[ ] application metrics are exposed where supported
[ ] alerts have an owner
[ ] rollback image is known
```

---

# Service Example 6 — Frontend

A production values file for `frontend` can follow this pattern:

```yaml
global:
  environment: prod

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/frontend
  tag: "RELEASE"

replicaCount: 3

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 15
  targetCPUUtilizationPercentage: 65

podDisruptionBudget:
  enabled: true
  minAvailable: 2

service:
  type: ClusterIP
  port: 8080
  targetPort: http

env:
  - name: APP_ENV
    value: production
```

Operational review for `frontend`:

```text
[ ] image exists in ECR
[ ] image is immutable
[ ] readiness endpoint works
[ ] liveness endpoint is safe
[ ] resource requests reflect measurements
[ ] HPA target is realistic
[ ] PDB permits maintenance
[ ] pod placement is HA-aware
[ ] dependencies are covered by NetworkPolicy
[ ] application metrics are exposed where supported
[ ] alerts have an owner
[ ] rollback image is known
```

---

# Service Example 7 — Dispatch

A production values file for `dispatch` can follow this pattern:

```yaml
global:
  environment: prod

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/dispatch
  tag: "RELEASE"

replicaCount: 3

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 15
  targetCPUUtilizationPercentage: 65

podDisruptionBudget:
  enabled: true
  minAvailable: 2

service:
  type: ClusterIP
  port: 8080
  targetPort: http

env:
  - name: APP_ENV
    value: production
```

Operational review for `dispatch`:

```text
[ ] image exists in ECR
[ ] image is immutable
[ ] readiness endpoint works
[ ] liveness endpoint is safe
[ ] resource requests reflect measurements
[ ] HPA target is realistic
[ ] PDB permits maintenance
[ ] pod placement is HA-aware
[ ] dependencies are covered by NetworkPolicy
[ ] application metrics are exposed where supported
[ ] alerts have an owner
[ ] rollback image is known
```

---

# Service Example 8 — Ratings

A production values file for `ratings` can follow this pattern:

```yaml
global:
  environment: prod

image:
  repository: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/ratings
  tag: "RELEASE"

replicaCount: 3

resources:
  requests:
    cpu: 250m
    memory: 256Mi
  limits:
    cpu: "1"
    memory: 512Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 15
  targetCPUUtilizationPercentage: 65

podDisruptionBudget:
  enabled: true
  minAvailable: 2

service:
  type: ClusterIP
  port: 8080
  targetPort: http

env:
  - name: APP_ENV
    value: production
```

Operational review for `ratings`:

```text
[ ] image exists in ECR
[ ] image is immutable
[ ] readiness endpoint works
[ ] liveness endpoint is safe
[ ] resource requests reflect measurements
[ ] HPA target is realistic
[ ] PDB permits maintenance
[ ] pod placement is HA-aware
[ ] dependencies are covered by NetworkPolicy
[ ] application metrics are exposed where supported
[ ] alerts have an owner
[ ] rollback image is known
```

---

# Appendix F — Expanded Production Review

## Deployment Review

```text
Deployment strategy reviewed
maxUnavailable reviewed
maxSurge reviewed
revisionHistoryLimit reviewed
terminationGracePeriod reviewed
probe behavior reviewed
security context reviewed
resources reviewed
pod placement reviewed
```

## Service Review

```text
ClusterIP used for internal service
selectors match Deployment labels
ports are named consistently
targetPort matches container port
```

## Ingress Review

```text
IngressClass is correct
ALB annotations are supported by deployed controller
certificate is correct for environment
health-check path is valid
target-type is intentional
security groups are correct
```

## Autoscaling Review

```text
requests exist
metrics server is healthy
minimum capacity is sufficient
maximum capacity is safe
scale-down behavior is stable
```

## Security Review

```text
no plaintext secrets
no privileged container
no hostNetwork unless approved
no hostPID unless approved
no unnecessary capabilities
non-root execution
workload identity
network policy
image scanning
```

## GitOps Review

```text
Git is source of truth
chart version is pinned
image is pinned
environment is explicit
Argo CD destination is correct
manual production mutation is avoided
```

## Observability Review

```text
metrics endpoint exists
ServiceMonitor selector is correct
PrometheusRule labels are correct
runbook URL exists
team owner exists
critical alerts are actionable
logs go to stdout/stderr
```
