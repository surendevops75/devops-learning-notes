# Complete Production YAMLs

## 1. Purpose

This chapter is the production YAML reference for the RoboShop DevOps/DevSecOps capstone.

It consolidates the Kubernetes, Helm, Argo CD, monitoring, security, networking, autoscaling, and GitOps manifests used throughout the architecture.

The examples are intentionally production-oriented rather than toy manifests.

The examples use placeholders for:

- AWS account IDs
- ECR repositories
- domains
- certificates
- secrets
- cluster identifiers
- notification credentials

Never commit real credentials into Git.

---

# 2. Production YAML Philosophy

Production YAML should be:

- declarative
- version controlled
- reviewable
- environment-aware
- immutable where possible
- security-conscious
- observable
- resource-constrained
- rollback-friendly

A production manifest should answer:

```text
What runs?
Where does it run?
How much resource does it need?
How is it exposed?
How is it secured?
How is it monitored?
How does Kubernetes know it is healthy?
How does it scale?
How is it recovered?
Who owns it?
```

---

# 3. Recommended Repository Structure

```text
gitops/
|
+-- clusters/
|   +-- prod/
|   +-- qa/
|   +-- dev/
|
+-- infrastructure/
|   +-- namespaces/
|   +-- network-policies/
|   +-- monitoring/
|   +-- ingress/
|
+-- applications/
|   +-- catalogue/
|   +-- user/
|   +-- cart/
|   +-- payment/
|   +-- shipping/
|   +-- frontend/
|
+-- argocd/
|   +-- projects/
|   +-- applications/
|   +-- applicationsets/
|
+-- monitoring/
|   +-- prometheusrules/
|   +-- alertmanager/
|
+-- security/
    +-- serviceaccounts/
    +-- roles/
    +-- rolebindings/
```

---

# 4. Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
  labels:
    environment: prod
    platform: roboshop
    security.example.com/enforce: restricted
```

## Important fields

`apiVersion` identifies the Kubernetes API.

`kind` identifies the resource type.

`metadata.name` identifies the namespace.

Labels allow policy engines, monitoring systems and automation to select resources.

---

# 5. Resource Quota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: roboshop-quota
  namespace: roboshop
spec:
  hard:
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
    pods: "100"
    services: "50"
    persistentvolumeclaims: "30"
```

This prevents one namespace from consuming unlimited cluster resources.

---

# 6. LimitRange

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: roboshop-default-limits
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
      max:
        cpu: "4"
        memory: 8Gi
      min:
        cpu: 10m
        memory: 32Mi
```

LimitRange provides namespace-level defaults and boundaries.

---

# 7. ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: catalogue
  namespace: roboshop
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<AWS_ACCOUNT_ID>:role/roboshop-prod-catalogue
automountServiceAccountToken: true
```

The IAM role should provide only the AWS permissions actually required by the service.

If a service does not need Kubernetes or AWS API credentials, prefer:

```yaml
automountServiceAccountToken: false
```

---

# 8. Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: catalogue
  namespace: roboshop
  labels:
    app.kubernetes.io/name: catalogue
    app.kubernetes.io/part-of: roboshop
    app.kubernetes.io/component: backend
    environment: prod
spec:
  replicas: 3

  revisionHistoryLimit: 10

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
        environment: prod
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/path: /metrics
        prometheus.io/port: "8080"

    spec:
      serviceAccountName: catalogue

      terminationGracePeriodSeconds: 60

      securityContext:
        seccompProfile:
          type: RuntimeDefault

      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                topologyKey: topology.kubernetes.io/zone
                labelSelector:
                  matchLabels:
                    app.kubernetes.io/name: catalogue

      containers:
        - name: catalogue
          image: <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/catalogue@sha256:<IMAGE_DIGEST>

          imagePullPolicy: IfNotPresent

          ports:
            - name: http
              containerPort: 8080
              protocol: TCP

          env:
            - name: ENVIRONMENT
              value: production

            - name: LOG_LEVEL
              value: INFO

          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: "1"
              memory: 512Mi

          securityContext:
            runAsNonRoot: true
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL

          readinessProbe:
            httpGet:
              path: /health/ready
              port: http
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 3
            failureThreshold: 3
            successThreshold: 1

          livenessProbe:
            httpGet:
              path: /health/live
              port: http
            initialDelaySeconds: 30
            periodSeconds: 20
            timeoutSeconds: 3
            failureThreshold: 3

          startupProbe:
            httpGet:
              path: /health/startup
              port: http
            periodSeconds: 5
            timeoutSeconds: 3
            failureThreshold: 30

          lifecycle:
            preStop:
              exec:
                command:
                  - /bin/sh
                  - -c
                  - sleep 10
```

---

# 9. Deployment Design Explanation

## `replicas`

Three replicas provide basic redundancy.

Production replica count should be based on:

- traffic
- availability requirements
- HPA behavior
- pod startup time
- node capacity
- failure-domain requirements

## `maxUnavailable: 0`

The deployment attempts to avoid reducing available replicas during normal rolling updates.

## `maxSurge: 1`

One additional pod may be created during rollout.

## Probes

Startup, readiness and liveness have different responsibilities.

```text
Startup
   |
   v
Can application start?

Readiness
   |
   v
Can application receive traffic?

Liveness
   |
   v
Is application still functioning?
```

---

# 10. Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue
  namespace: roboshop
  labels:
    app.kubernetes.io/name: catalogue
    environment: prod
spec:
  type: ClusterIP

  selector:
    app.kubernetes.io/name: catalogue

  ports:
    - name: http
      port: 8080
      targetPort: http
      protocol: TCP

  sessionAffinity: None
```

The Service provides stable service discovery even when pods are replaced.

---

# 11. Headless Service

Use a headless service only when the application needs direct endpoint discovery.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: catalogue-headless
  namespace: roboshop
spec:
  clusterIP: None
  selector:
    app.kubernetes.io/name: catalogue
  ports:
    - name: http
      port: 8080
      targetPort: http
```

Do not use headless Services unnecessarily.

---

# 12. PodDisruptionBudget

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

This protects availability during voluntary disruptions such as:

- node drain
- cluster maintenance
- upgrades

A PDB does not protect against every failure.

---

# 13. HorizontalPodAutoscaler

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
  maxReplicas: 15

  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
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
      selectPolicy: Max

  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 65

    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 75
```

The exact thresholds should come from performance testing.

---

# 14. NetworkPolicy — Default Deny

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: roboshop-default-deny
  namespace: roboshop
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

This establishes a deny-by-default baseline.

---

# 15. NetworkPolicy — Catalogue Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue-ingress
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: catalogue

  policyTypes:
    - Ingress

  ingress:
    - from:
        - podSelector:
            matchLabels:
              app.kubernetes.io/name: frontend

        - podSelector:
            matchLabels:
              app.kubernetes.io/name: cart

      ports:
        - protocol: TCP
          port: 8080
```

Only approved workloads should reach the service.

---

# 16. NetworkPolicy — DNS Egress

If egress is restricted, DNS must remain reachable.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: catalogue-dns
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: catalogue

  policyTypes:
    - Egress

  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

---

# 17. ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: catalogue-config
  namespace: roboshop
data:
  LOG_LEVEL: INFO
  HTTP_PORT: "8080"
  REQUEST_TIMEOUT_MS: "3000"
```

Do not store credentials here.

---

# 18. Secret

Example structure only:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: catalogue-secret
  namespace: roboshop
type: Opaque
stringData:
  DATABASE_USERNAME: "<managed-secret-reference>"
  DATABASE_PASSWORD: "<managed-secret-reference>"
```

For production, use the approved secrets-management integration rather than committing actual secret values to Git.

---

# 19. Secret Consumption

```yaml
env:
  - name: DATABASE_USERNAME
    valueFrom:
      secretKeyRef:
        name: catalogue-secret
        key: DATABASE_USERNAME

  - name: DATABASE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: catalogue-secret
        key: DATABASE_PASSWORD
```

Avoid putting secrets into command-line arguments because they may become visible in process metadata.

---

# 20. ALB Ingress

The capstone uses AWS ALB Ingress.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
    alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:<AWS_REGION>:<AWS_ACCOUNT_ID>:certificate/<CERTIFICATE_ID>
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/success-codes: "200-399"
    alb.ingress.kubernetes.io/load-balancer-attributes: idle_timeout.timeout_seconds=60
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

---

# 21. ALB Security Considerations

The ALB should:

- terminate TLS
- use ACM certificates
- restrict security-group access
- redirect HTTP to HTTPS
- expose only required listeners
- use appropriate health checks
- send traffic to healthy targets

Do not expose internal services directly through public ALBs unless required.

---

# 22. TLS Certificate Placeholder

Never commit a real certificate ARN blindly across environments.

Use environment-specific values:

```yaml
alb:
  certificateArn: arn:aws:acm:<AWS_REGION>:<AWS_ACCOUNT_ID>:certificate/<CERTIFICATE_ID>
```

Helm values should provide environment-specific configuration.

---

# 23. ServiceMonitor

When Prometheus Operator is used:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: catalogue
  namespace: monitoring
  labels:
    release: prometheus
spec:
  namespaceSelector:
    matchNames:
      - roboshop

  selector:
    matchLabels:
      app.kubernetes.io/name: catalogue

  endpoints:
    - port: http
      path: /metrics
      interval: 30s
      scrapeTimeout: 10s
```

The exact label required by the Prometheus installation depends on the Prometheus Operator configuration.

---

# 24. PodMonitor Alternative

If the application exposes metrics directly from pods:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: catalogue
  namespace: monitoring
  labels:
    release: prometheus
spec:
  namespaceSelector:
    matchNames:
      - roboshop

  selector:
    matchLabels:
      app.kubernetes.io/name: catalogue

  podMetricsEndpoints:
    - port: http
      path: /metrics
      interval: 30s
```

Use either ServiceMonitor or PodMonitor according to the application architecture.

---

# 25. PrometheusRule — Recording Rules

Recording rules precompute frequently used PromQL expressions.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-recording-rules
  namespace: monitoring
  labels:
    release: prometheus
spec:
  groups:
    - name: roboshop.recording
      interval: 30s
      rules:

        - record: roboshop:catalogue_request_rate:5m
          expr: |
            sum(rate(http_requests_total{
              namespace="roboshop",
              service="catalogue"
            }[5m]))

        - record: roboshop:catalogue_5xx_ratio:5m
          expr: |
            sum(rate(http_requests_total{
              namespace="roboshop",
              service="catalogue",
              status=~"5.."
            }[5m]))
            /
            clamp_min(
              sum(rate(http_requests_total{
                namespace="roboshop",
                service="catalogue"
              }[5m])),
              0.001
            )
```

Recording rules improve dashboard and alert query efficiency.

---

# 26. PrometheusRule — Node Alerts

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-node-alerts
  namespace: monitoring
  labels:
    release: prometheus
    team: platform
    environment: prod
spec:
  groups:
    - name: node.rules
      rules:

        - alert: NodeNotReady
          expr: |
            kube_node_status_condition{
              condition="Ready",
              status="true"
            } == 0
          for: 10m
          labels:
            severity: critical
            team: platform
            environment: prod
          annotations:
            summary: "Kubernetes node is not ready"
            description: "Node {{ $labels.node }} has been NotReady for more than 10 minutes."
            runbook_url: "https://runbooks.example.com/kubernetes/node-not-ready"

        - alert: NodeCPUHigh
          expr: |
            100 * (
              1 -
              avg by (instance) (
                rate(node_cpu_seconds_total{
                  mode="idle"
                }[5m])
              )
            ) > 85
          for: 15m
          labels:
            severity: warning
            team: platform
            environment: prod
          annotations:
            summary: "Node CPU utilization is high"
            description: "Node {{ $labels.instance }} has CPU utilization above 85%."
            runbook_url: "https://runbooks.example.com/kubernetes/node-cpu"

        - alert: NodeMemoryPressure
          expr: |
            kube_node_status_condition{
              condition="MemoryPressure",
              status="true"
            } == 1
          for: 10m
          labels:
            severity: critical
            team: platform
            environment: prod
          annotations:
            summary: "Node memory pressure detected"
            description: "Node {{ $labels.node }} is reporting MemoryPressure."
            runbook_url: "https://runbooks.example.com/kubernetes/node-memory"
```

---

# 27. PrometheusRule — Pod Restart Alerts

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-pod-alerts
  namespace: monitoring
  labels:
    release: prometheus
    team: platform
    environment: prod
spec:
  groups:
    - name: pod.rules
      rules:

        - alert: ContainerRestartingFrequently
          expr: |
            increase(
              kube_pod_container_status_restarts_total{
                namespace="roboshop"
              }[15m]
            ) > 5
          for: 10m
          labels:
            severity: warning
            team: platform
            environment: prod
          annotations:
            summary: "Container restarting frequently"
            description: "Pod {{ $labels.namespace }}/{{ $labels.pod }} container {{ $labels.container }} restarted more than 5 times in 15 minutes."
            runbook_url: "https://runbooks.example.com/kubernetes/container-restarts"

        - alert: PodNotReady
          expr: |
            kube_pod_status_ready{
              namespace="roboshop",
              condition="true"
            } == 0
          for: 15m
          labels:
            severity: warning
            team: platform
            environment: prod
          annotations:
            summary: "Pod is not ready"
            description: "Pod {{ $labels.namespace }}/{{ $labels.pod }} has not been Ready for 15 minutes."
            runbook_url: "https://runbooks.example.com/kubernetes/pod-not-ready"
```

---

# 28. OOMKilled Alert

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-oom-alerts
  namespace: monitoring
  labels:
    release: prometheus
    team: platform
    environment: prod
spec:
  groups:
    - name: container.oom
      rules:

        - alert: ContainerOOMKilled
          expr: |
            kube_pod_container_status_last_terminated_reason{
              namespace="roboshop",
              reason="OOMKilled"
            } == 1
          for: 5m
          labels:
            severity: critical
            team: platform
            environment: prod
          annotations:
            summary: "Container was OOMKilled"
            description: "Container {{ $labels.container }} in pod {{ $labels.pod }} was terminated because of an out-of-memory condition."
            runbook_url: "https://runbooks.example.com/kubernetes/oomkilled"
```

---

# 29. Deployment Availability Alert

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-deployment-alerts
  namespace: monitoring
  labels:
    release: prometheus
    team: platform
    environment: prod
spec:
  groups:
    - name: deployment.rules
      rules:

        - alert: DeploymentReplicasUnavailable
          expr: |
            kube_deployment_status_replicas_available{
              namespace="roboshop"
            }
            <
            kube_deployment_spec_replicas{
              namespace="roboshop"
            }
          for: 10m
          labels:
            severity: critical
            team: platform
            environment: prod
          annotations:
            summary: "Deployment has unavailable replicas"
            description: "Deployment {{ $labels.namespace }}/{{ $labels.deployment }} has fewer available replicas than desired."
            runbook_url: "https://runbooks.example.com/kubernetes/deployment-unavailable"
```

---

# 30. Application Error-Rate Alert

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-application-alerts
  namespace: monitoring
  labels:
    release: prometheus
    team: application
    environment: prod
spec:
  groups:
    - name: application.rules
      rules:

        - alert: CatalogueHighErrorRate
          expr: |
            (
              sum(rate(http_requests_total{
                namespace="roboshop",
                service="catalogue",
                status=~"5.."
              }[5m]))
              /
              clamp_min(
                sum(rate(http_requests_total{
                  namespace="roboshop",
                  service="catalogue"
                }[5m])),
                0.001
              )
            ) > 0.05
          for: 10m
          labels:
            severity: critical
            team: application
            service: catalogue
            environment: prod
          annotations:
            summary: "Catalogue error rate is high"
            description: "Catalogue HTTP 5xx error rate has exceeded 5% for 10 minutes."
            runbook_url: "https://runbooks.example.com/catalogue/high-error-rate"
```

---

# 31. Application Latency Alert

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-latency-alerts
  namespace: monitoring
  labels:
    release: prometheus
    team: application
    environment: prod
spec:
  groups:
    - name: application.latency
      rules:

        - alert: CatalogueHighP95Latency
          expr: |
            histogram_quantile(
              0.95,
              sum by (
                le
              ) (
                rate(http_request_duration_seconds_bucket{
                  namespace="roboshop",
                  service="catalogue"
                }[5m])
              )
            ) > 0.5
          for: 10m
          labels:
            severity: warning
            team: application
            service: catalogue
            environment: prod
          annotations:
            summary: "Catalogue p95 latency is high"
            description: "Catalogue p95 request latency has exceeded 500ms for 10 minutes."
            runbook_url: "https://runbooks.example.com/catalogue/high-latency"
```

---

# 32. Availability Alert

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-availability-alerts
  namespace: monitoring
  labels:
    release: prometheus
    team: sre
    environment: prod
spec:
  groups:
    - name: availability
      rules:

        - alert: ServiceAvailabilityLow
          expr: |
            (
              sum(rate(http_requests_total{
                namespace="roboshop",
                service="catalogue",
                status!~"5.."
              }[5m]))
              /
              clamp_min(
                sum(rate(http_requests_total{
                  namespace="roboshop",
                  service="catalogue"
                }[5m])),
                0.001
              )
            ) < 0.995
          for: 10m
          labels:
            severity: critical
            team: sre
            service: catalogue
            environment: prod
          annotations:
            summary: "Catalogue availability is below target"
            description: "Catalogue availability has fallen below 99.5%."
            runbook_url: "https://runbooks.example.com/slo/catalogue-availability"
```

---

# 33. SLO Fast-Burn Alert

A production SLO system should distinguish rapid incidents from slow budget consumption.

Example conceptual fast-burn alert:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-slo-alerts
  namespace: monitoring
  labels:
    release: prometheus
    team: sre
    environment: prod
spec:
  groups:
    - name: slo.catalogue
      rules:

        - alert: CatalogueSLOFastBurn
          expr: |
            (
              1 -
              (
                sum(rate(http_requests_total{
                  namespace="roboshop",
                  service="catalogue",
                  status!~"5.."
                }[5m]))
                /
                clamp_min(
                  sum(rate(http_requests_total{
                    namespace="roboshop",
                    service="catalogue"
                  }[5m])),
                  0.001
                )
              )
            ) > 0.05
          for: 5m
          labels:
            severity: critical
            team: sre
            service: catalogue
            environment: prod
            slo: availability
          annotations:
            summary: "Catalogue SLO is burning error budget rapidly"
            description: "Catalogue availability is consuming error budget at a critical rate."
            runbook_url: "https://runbooks.example.com/slo/catalogue-fast-burn"
```

This expression is intentionally shown as a practical starting point. Mature SLO implementations generally use multi-window, multi-burn-rate alerting.

---

# 34. SLO Multi-Window Pattern

A stronger implementation uses multiple windows.

Conceptually:

```text
Short window
+
Long window
+
Burn-rate threshold
```

Example structure:

```yaml
- alert: CatalogueSLOMultiWindowBurn
  expr: |
    (
      burn_rate_short > 14.4
      and
      burn_rate_long > 14.4
    )
    or
    (
      burn_rate_short > 6
      and
      burn_rate_long > 6
    )
  for: 5m
```

The actual burn-rate recording rules should be calculated from the organization's SLO and error-budget policy.

---

# 35. PrometheusRule — Disk Space

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-disk-alerts
  namespace: monitoring
  labels:
    release: prometheus
    team: platform
    environment: prod
spec:
  groups:
    - name: disk.rules
      rules:

        - alert: NodeFilesystemAlmostFull
          expr: |
            (
              node_filesystem_avail_bytes{
                fstype!~"tmpfs|overlay",
                mountpoint="/"
              }
              /
              node_filesystem_size_bytes{
                fstype!~"tmpfs|overlay",
                mountpoint="/"
              }
            ) < 0.15
          for: 15m
          labels:
            severity: warning
            team: platform
            environment: prod
          annotations:
            summary: "Node filesystem has less than 15% free space"
            description: "Filesystem {{ $labels.mountpoint }} on {{ $labels.instance }} is almost full."
            runbook_url: "https://runbooks.example.com/kubernetes/disk-full"
```

---

# 36. PrometheusRule — Network

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: roboshop-network-alerts
  namespace: monitoring
  labels:
    release: prometheus
    team: platform
    environment: prod
spec:
  groups:
    - name: network.rules
      rules:

        - alert: NodeNetworkReceiveErrors
          expr: |
            increase(
              node_network_receive_errs_total{
                device!~"lo|veth.*"
              }[10m]
            ) > 100
          for: 10m
          labels:
            severity: warning
            team: platform
            environment: prod
          annotations:
            summary: "Node network receive errors detected"
            description: "Network receive errors are increasing on {{ $labels.instance }}."
            runbook_url: "https://runbooks.example.com/kubernetes/network-errors"
```

Thresholds should be tuned to the actual environment.

---

# 37. EKS Control Plane Considerations

EKS control-plane metrics are not equivalent to ordinary node metrics.

Useful operational signals can include:

- API server availability
- API request latency
- API request errors
- authentication failures
- admission failures
- controller health
- scheduler behavior
- cluster events

AWS-side monitoring and Kubernetes metrics should be correlated.

---

# 38. ALB Alert Example

ALB metrics can be monitored from AWS/CloudWatch and integrated into the organization's alerting pipeline.

Important metrics include:

```text
HTTPCode_ELB_5XX_Count
HTTPCode_Target_5XX_Count
TargetResponseTime
HealthyHostCount
UnHealthyHostCount
RejectedConnectionCount
ActiveConnectionCount
```

A production design should alert on sustained abnormal behavior rather than isolated single-minute spikes.

---

# 39. Alertmanager Configuration

Example production-style configuration:

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
      group_by:
        - alertname
        - cluster
        - environment
        - team
        - service

      group_wait: 30s
      group_interval: 5m
      repeat_interval: 4h

      receiver: default

      routes:

        - matchers:
            - severity="critical"
            - environment="prod"
          receiver: prod-critical
          continue: false

        - matchers:
            - severity="warning"
            - environment="prod"
          receiver: prod-warning
          continue: false

        - matchers:
            - environment=~"dev|qa"
          receiver: nonprod
          continue: false

    inhibit_rules:

      - source_matchers:
          - severity="critical"
        target_matchers:
          - severity="warning"
        equal:
          - alertname
          - cluster
          - service

    receivers:

      - name: default
        email_configs:
          - to: platform@example.com
            from: alertmanager@example.com
            smarthost: smtp.example.com:587
            auth_username: alertmanager@example.com
            auth_password: "<SMTP_PASSWORD_PLACEHOLDER>"
            require_tls: true

      - name: prod-critical
        email_configs:
          - to: oncall@example.com
            from: alertmanager@example.com
            smarthost: smtp.example.com:587
            auth_username: alertmanager@example.com
            auth_password: "<SMTP_PASSWORD_PLACEHOLDER>"
            require_tls: true

        webhook_configs:
          - url: "<PAGERDUTY_OR_INCIDENT_WEBHOOK>"
            send_resolved: true

      - name: prod-warning
        email_configs:
          - to: platform@example.com
            from: alertmanager@example.com
            smarthost: smtp.example.com:587
            auth_username: alertmanager@example.com
            auth_password: "<SMTP_PASSWORD_PLACEHOLDER>"
            require_tls: true

      - name: nonprod
        email_configs:
          - to: devops@example.com
            from: alertmanager@example.com
            smarthost: smtp.example.com:587
            auth_username: alertmanager@example.com
            auth_password: "<SMTP_PASSWORD_PLACEHOLDER>"
            require_tls: true
```

---

# 40. Alertmanager Routing

Routing hierarchy:

```text
Alert
 |
 v
Global Route
 |
 +--> Production Critical
 |       |
 |       +--> On-call
 |       +--> Incident Management
 |
 +--> Production Warning
 |       |
 |       +--> Team
 |
 +--> Dev/QA
         |
         +--> Engineering
```

This prevents every alert from reaching every person.

---

# 41. Alertmanager Grouping

Example:

```yaml
group_by:
  - alertname
  - cluster
  - environment
  - team
  - service
```

If ten pods in the same service fail simultaneously, grouping prevents ten separate notifications.

Instead:

```text
CatalogueHighErrorRate
cluster=prod
environment=prod
team=application
service=catalogue
```

can produce one meaningful incident notification.

---

# 42. Alertmanager Inhibition

Inhibition suppresses secondary alerts when a more fundamental failure is already known.

Example:

```yaml
inhibit_rules:
  - source_matchers:
      - alertname="ClusterDown"
      - severity="critical"

    target_matchers:
      - severity="warning"

    equal:
      - cluster
```

If the entire cluster is unavailable, dozens of downstream warnings may not be actionable.

---

# 43. Silences

A silence temporarily suppresses matching alerts.

Use cases:

- planned maintenance
- node replacement
- controlled migration
- approved deployment window

Silences must have:

- owner
- reason
- start time
- expiry time

Avoid permanent silences.

---

# 44. Severity Model

A practical model:

```text
critical
    |
    +--> immediate response
    +--> production user impact
    +--> page on-call

warning
    |
    +--> investigate soon
    +--> no immediate page

info
    |
    +--> informational
    +--> normally no page
```

Not every alert should wake someone at 3 AM.

---

# 45. Alert Labels

Recommended labels:

```yaml
labels:
  severity: critical
  team: application
  service: catalogue
  environment: prod
  cluster: roboshop-prod
  region: ap-south-1
  category: availability
```

Labels are machine-readable routing metadata.

---

# 46. Alert Annotations

Annotations are human-readable context.

```yaml
annotations:
  summary: "Catalogue availability degraded"
  description: "Catalogue availability has remained below 99.5% for 10 minutes."
  runbook_url: "https://runbooks.example.com/catalogue/availability"
```

A good alert should answer:

```text
What happened?
Where?
How severe?
What should I do?
```

---

# 47. Runbook URL Design

Every actionable production alert should have a runbook.

Example:

```text
https://runbooks.example.com/catalogue/high-error-rate
```

The runbook should contain:

- symptoms
- immediate checks
- commands
- dashboards
- likely causes
- mitigation
- rollback instructions
- escalation path

---

# 48. Dead-Man's-Switch Concept

A dead-man's-switch alert verifies that the monitoring pipeline itself is alive.

Concept:

```text
Prometheus
   |
   v
Watchdog / AlwaysFiring Alert
   |
   v
Alertmanager
   |
   v
Monitoring notification channel
```

If the expected watchdog signal disappears, monitoring may itself be broken.

This protects against silent monitoring failure.

---

# 49. Watchdog Alert

Example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: monitoring-watchdog
  namespace: monitoring
  labels:
    release: prometheus
spec:
  groups:
    - name: watchdog
      rules:
        - alert: Watchdog
          expr: vector(1)
          labels:
            severity: none
            team: platform
          annotations:
            summary: "Monitoring watchdog"
            description: "This alert should continuously reach the monitoring validation path."
```

The external monitoring system should verify receipt according to its own dead-man's-switch design.

---

# 50. Flapping Alerts

A flapping alert repeatedly transitions:

```text
FIRING
  |
RESOLVED
  |
FIRING
  |
RESOLVED
```

Common causes:

- threshold too close to normal value
- insufficient `for` duration
- noisy metric
- unstable application
- poor query design

Mitigation:

- use `for`
- use smoothing
- use appropriate thresholds
- use multi-window logic
- investigate the underlying behavior

---

# 51. Alert Storm Prevention

An outage can generate thousands of alerts.

Controls include:

```text
Grouping
Inhibition
Routing
Deduplication
Threshold tuning
Dependency-aware alerting
Silencing during approved maintenance
```

The objective is:

> One incident should produce actionable context, not notification overload.

---

# 52. Alert Testing

Validate Prometheus configuration:

```bash
promtool check rules alerts.yaml
```

Validate Alertmanager configuration:

```bash
amtool check-config alertmanager.yaml
```

Depending on the deployed versions, use the matching supported validation commands.

---

# 53. Query Testing

Test PromQL directly.

Example:

```promql
sum(rate(http_requests_total{
  namespace="roboshop",
  service="catalogue",
  status=~"5.."
}[5m]))
```

Then test the complete ratio.

Never deploy an alert rule without confirming that the metric actually exists.

---

# 54. Alert Rule Test Strategy

```text
Write PromQL
   |
   v
Test query
   |
   v
Validate labels
   |
   v
Test threshold
   |
   v
Test firing
   |
   v
Test notification
   |
   v
Test resolution
```

A notification that never fires is not a tested alert.

---

# 55. Alertmanager HA

For production, Alertmanager can run multiple replicas.

Example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Alertmanager
metadata:
  name: main
  namespace: monitoring
spec:
  replicas: 3

  retention: 120h

  resources:
    requests:
      cpu: 100m
      memory: 256Mi
    limits:
      cpu: 500m
      memory: 1Gi
```

The exact configuration depends on the Prometheus Operator deployment.

Alertmanager clustering allows instances to coordinate alert state and reduce duplicate notifications.

---

# 56. Prometheus HA

A production monitoring platform can run multiple Prometheus replicas.

```text
Targets
   |
   +--> Prometheus A
   |
   +--> Prometheus B
```

High availability does not automatically mean unlimited historical storage.

Long-term metrics storage and retention should be designed separately.

---

# 57. Prometheus Retention

Example concept:

```yaml
spec:
  retention: 15d
  resources:
    requests:
      cpu: 1
      memory: 4Gi
```

Retention should be based on:

- storage capacity
- query patterns
- operational needs
- compliance
- cost

---

# 58. Grafana Datasource

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-datasource
  namespace: monitoring
data:
  datasource.yaml: |
    apiVersion: 1

    datasources:
      - name: Prometheus
        type: prometheus
        access: proxy
        url: http://prometheus-operated.monitoring.svc:9090
        isDefault: true
        editable: false
```

The actual Service name depends on the Prometheus deployment.

---

# 59. Grafana Dashboard Provisioning

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: grafana-dashboard-provider
  namespace: monitoring
data:
  provider.yaml: |
    apiVersion: 1

    providers:
      - name: roboshop
        orgId: 1
        folder: RoboShop
        type: file
        disableDeletion: true
        editable: false
        options:
          path: /var/lib/grafana/dashboards/roboshop
```

---

# 60. Application Dashboard Panels

Recommended dashboard panels:

```text
Request Rate
Error Rate
p50 Latency
p95 Latency
p99 Latency
CPU
Memory
Pod Count
Restart Count
HPA Replicas
Network
Dependency Errors
```

A deployment dashboard should also show:

```text
Current version
Previous version
Deployment time
Argo CD sync state
```

---

# 61. ELK Log Shipping Concept

```text
Pod stdout/stderr
       |
       v
Node log files
       |
       v
Log collector
       |
       v
Logstash / pipeline
       |
       v
Elasticsearch
       |
       v
Kibana
```

Prometheus is not a replacement for centralized logs.

---

# 62. Structured Logging

Application logs should preferably be structured.

Example:

```json
{
  "timestamp": "2026-08-31T10:15:30Z",
  "level": "ERROR",
  "service": "catalogue",
  "environment": "prod",
  "trace_id": "REDACTED",
  "request_id": "REQ-12345",
  "message": "Database connection failed"
}
```

Do not log:

- passwords
- access tokens
- session secrets
- private keys
- unnecessary personal data

---

# 63. Argo CD Project

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: production
  namespace: argocd
spec:
  description: RoboShop production applications

  sourceRepos:
    - https://git.example.com/platform/gitops.git

  destinations:
    - namespace: roboshop
      server: https://kubernetes.default.svc

  clusterResourceWhitelist:
    - group: ""
      kind: Namespace

  namespaceResourceWhitelist:
    - group: "*"
      kind: "*"
```

Production projects should use the narrowest possible permissions.

---

# 64. Argo CD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: catalogue-prod
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: production

  source:
    repoURL: https://git.example.com/platform/gitops.git
    targetRevision: main
    path: applications/catalogue/overlays/prod

  destination:
    server: https://kubernetes.default.svc
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true

    syncOptions:
      - CreateNamespace=true
      - PrunePropagationPolicy=foreground
```

---

# 65. Argo CD Multi-Cluster Application

Conceptually:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: catalogue-prod-ap-south-1
  namespace: argocd
spec:
  project: production

  source:
    repoURL: https://git.example.com/platform/gitops.git
    targetRevision: main
    path: applications/catalogue/overlays/prod

  destination:
    name: roboshop-prod-ap-south-1
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

A centralized Argo CD control plane can manage multiple registered EKS clusters.

---

# 66. ApplicationSet Pattern

For many clusters:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: roboshop-prod
  namespace: argocd
spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            environment: prod
            platform: eks

  template:
    metadata:
      name: '{{name}}-catalogue'

    spec:
      project: production

      source:
        repoURL: https://git.example.com/platform/gitops.git
        targetRevision: main
        path: applications/catalogue/overlays/prod

      destination:
        server: '{{server}}'
        namespace: roboshop
```

This pattern reduces duplicated Application manifests.

---

# 67. Argo CD Notifications

A conceptual notification subscription:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
  namespace: argocd
data:
  subscriptions: |
    - recipients:
        - slack:platform-deployments
      triggers:
        - on-sync-failed
        - on-health-degraded
```

The exact notification configuration depends on the installed Argo CD Notifications setup.

---

# 68. Argo CD Deployment Failure Alert

A Prometheus rule can alert from Argo CD metrics if those metrics are enabled and correctly labeled.

Conceptual:

```yaml
- alert: ArgoCDApplicationDegraded
  expr: |
    argocd_app_info{
      dest_namespace="roboshop"
    }
    and
    argocd_app_health_status{
      health_status="Degraded"
    } == 1
  for: 10m
  labels:
    severity: critical
    team: platform
    environment: prod
  annotations:
    summary: "Argo CD application is degraded"
    description: "Argo CD application {{ $labels.name }} is degraded."
    runbook_url: "https://runbooks.example.com/argocd/application-degraded"
```

Metric names and labels should be verified against the deployed Argo CD version.

---

# 69. GitOps Drift

With Argo CD self-heal enabled:

```text
Git
 |
 v
Desired state
 |
 v
Argo CD
 |
 v
EKS

Manual kubectl change
 |
 v
Actual state differs
 |
 v
Argo CD detects drift
 |
 v
Reconcile
```

Manual production changes should still be avoided.

---

# 70. CI Deployment Status ConfigMap

Deployment metadata can be represented through labels/annotations.

Example:

```yaml
metadata:
  annotations:
    deployment.example.com/git-sha: "8f3c1a2"
    deployment.example.com/build-id: "jenkins-1842"
    deployment.example.com/release: "catalogue-1.8.3"
    deployment.example.com/owner: "catalogue-team"
```

This improves incident investigation.

---

# 71. Pod Topology Spread

For stronger failure-domain distribution:

```yaml
spec:
  topologySpreadConstraints:
    - maxSkew: 1
      topologyKey: topology.kubernetes.io/zone
      whenUnsatisfiable: DoNotSchedule
      labelSelector:
        matchLabels:
          app.kubernetes.io/name: catalogue
```

This helps prevent all replicas from being scheduled into one Availability Zone.

---

# 72. Node Selector

Example:

```yaml
nodeSelector:
  workload: application
```

Use only when dedicated node placement is actually required.

---

# 73. Tolerations

Example:

```yaml
tolerations:
  - key: workload
    operator: Equal
    value: application
    effect: NoSchedule
```

Tolerations allow pods to run on appropriately tainted nodes.

A toleration does not itself force scheduling onto those nodes; combine with affinity/node selection where required.

---

# 74. Pod Security Context

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
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL
```

Applications that require writable directories should mount specific writable `emptyDir` volumes rather than making the entire root filesystem writable.

---

# 75. Writable Temporary Directory

```yaml
volumeMounts:
  - name: tmp
    mountPath: /tmp

volumes:
  - name: tmp
    emptyDir: {}
```

Then:

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

This gives applications controlled temporary storage.

---

# 76. PriorityClass

For critical platform components:

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: roboshop-critical
value: 100000
globalDefault: false
description: "Priority for critical production workloads"
```

Do not assign very high priority to ordinary application pods.

---

# 77. CronJob

Example backup or maintenance job:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: roboshop-maintenance
  namespace: roboshop
spec:
  schedule: "15 2 * * *"

  concurrencyPolicy: Forbid

  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 5

  jobTemplate:
    spec:
      backoffLimit: 3
      template:
        spec:
          restartPolicy: Never

          containers:
            - name: maintenance
              image: <ECR_REPOSITORY>@sha256:<IMAGE_DIGEST>

              command:
                - /app/maintenance.sh

              resources:
                requests:
                  cpu: 100m
                  memory: 128Mi
                limits:
                  cpu: 500m
                  memory: 512Mi
```

---

# 78. Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: catalogue-migration-001
  namespace: roboshop
spec:
  backoffLimit: 2

  template:
    spec:
      restartPolicy: Never

      containers:
        - name: migration
          image: <ECR_REPOSITORY>@sha256:<IMAGE_DIGEST>
          command:
            - /app/migrate
```

Database migrations should be designed carefully for rollback compatibility.

---

# 79. Horizontal Pod Autoscaler Dependency

HPA depends on metrics.

For CPU utilization:

```text
Pod resource requests
        |
        v
Metrics Server
        |
        v
HPA
        |
        v
Replica count
```

If requests are missing or inaccurate, CPU-based HPA decisions may be misleading.

---

# 80. PodDisruptionBudget and Autoscaling

PDB, HPA and cluster autoscaling interact.

Example:

```text
HPA wants more pods
        |
        v
Cluster lacks capacity
        |
        v
Cluster Autoscaler adds node
        |
        v
Pods schedule
```

During node maintenance:

```text
PDB
 |
 v
Protect minimum availability
```

Do not create a PDB so strict that cluster upgrades become impossible.

---

# 81. ServiceAccount RBAC Role

Example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: catalogue-read-config
  namespace: roboshop
rules:
  - apiGroups: [""]
    resources:
      - configmaps
    resourceNames:
      - catalogue-config
    verbs:
      - get
```

Grant only the exact required permissions.

---

# 82. RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: catalogue-read-config
  namespace: roboshop
subjects:
  - kind: ServiceAccount
    name: catalogue
    namespace: roboshop
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: catalogue-read-config
```

Avoid broad `cluster-admin` permissions for applications.

---

# 83. Ingress Internal ALB

For an internal service:

```yaml
metadata:
  annotations:
    alb.ingress.kubernetes.io/scheme: internal
```

Internal ALBs are appropriate for private application traffic.

---

# 84. AWS ALB Target Type

The capstone uses:

```yaml
alb.ingress.kubernetes.io/target-type: ip
```

This allows ALB targets to be pod IPs when supported by the AWS Load Balancer Controller architecture.

This can simplify traffic flow:

```text
ALB
 |
 v
Pod IP
```

---

# 85. AWS Load Balancer Controller ServiceAccount

A production installation normally uses an IAM role associated with the controller service account.

Conceptual:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: aws-load-balancer-controller
  namespace: kube-system
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<AWS_ACCOUNT_ID>:role/AmazonEKSLoadBalancerControllerRole
```

The actual IAM policy must be maintained according to the supported AWS Load Balancer Controller release.

---

# 86. NetworkPolicy for Frontend

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-policy
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app.kubernetes.io/name: frontend

  policyTypes:
    - Ingress
    - Egress

  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: roboshop

  egress:
    - to:
        - podSelector:
            matchLabels:
              app.kubernetes.io/name: catalogue
      ports:
        - protocol: TCP
          port: 8080
```

In a real environment, refine this according to the complete application dependency graph.

---

# 87. NetworkPolicy Design Rule

Do not create random policies service-by-service.

First build a dependency matrix:

```text
frontend -> catalogue
frontend -> user
frontend -> cart

cart -> redis
cart -> catalogue

payment -> rabbitmq
shipping -> rabbitmq
```

Then translate the dependency matrix into NetworkPolicies.

---

# 88. Pod Anti-Affinity

A simple preferred policy:

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

This tries to distribute replicas across nodes.

---

# 89. Environment Labels

Every production workload should carry environment metadata.

Example:

```yaml
labels:
  environment: prod
  team: catalogue
  cost-center: platform
  application: catalogue
```

Standardize labels across the organization.

---

# 90. Common Required Labels

Recommended Kubernetes application labels:

```yaml
app.kubernetes.io/name: catalogue
app.kubernetes.io/instance: catalogue-prod
app.kubernetes.io/version: "1.8.3"
app.kubernetes.io/component: backend
app.kubernetes.io/part-of: roboshop
app.kubernetes.io/managed-by: Helm
```

Do not claim a version label is authoritative if the actual image digest is the release source of truth.

---

# 91. Helm Values — Production

```yaml
replicaCount: 3

image:
  repository: <AWS_ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/catalogue
  digest: "sha256:<IMAGE_DIGEST>"
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 8080

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

securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
```

---

# 92. Helm Production Environment Separation

Example:

```text
values.yaml
values-dev.yaml
values-qa.yaml
values-prod.yaml
```

Do not duplicate entire manifests for every environment.

Keep common templates centralized and override environment-specific configuration.

---

# 93. Argo CD Production Sync Strategy

A production application may use:

```yaml
syncPolicy:
  automated:
    prune: true
    selfHeal: true
```

But high-risk changes may use manual approval.

The correct policy depends on:

- service criticality
- change risk
- compliance
- release maturity
- rollback confidence

---

# 94. Production YAML Validation Pipeline

Before merge:

```bash
helm lint charts/catalogue

helm template catalogue \
  charts/catalogue \
  -f values/prod.yaml > rendered.yaml

kubectl apply \
  --dry-run=server \
  -f rendered.yaml

promtool check rules monitoring/rules.yaml

amtool check-config monitoring/alertmanager.yaml
```

Add organization-approved policy validation.

---

# 95. Kubernetes Diff

Before production deployment:

```bash
kubectl diff -f rendered.yaml
```

For GitOps, the preferred comparison should be the Git desired state versus cluster state through Argo CD.

---

# 96. YAML Security Checklist

```text
[ ] No plaintext credentials
[ ] No private keys
[ ] No tokens
[ ] No latest image
[ ] Immutable image reference
[ ] Resource requests
[ ] Resource limits
[ ] Readiness probe
[ ] Liveness probe
[ ] Startup probe where needed
[ ] Non-root
[ ] No privilege escalation
[ ] Capabilities dropped
[ ] Appropriate NetworkPolicy
[ ] Correct ServiceAccount
[ ] Least-privilege RBAC
[ ] PDB where appropriate
[ ] HPA where appropriate
[ ] Standard labels
```

---

# 97. Production Alert Design Checklist

```text
[ ] Clear alert name
[ ] Correct PromQL
[ ] Appropriate threshold
[ ] Appropriate for duration
[ ] Severity
[ ] Team
[ ] Service
[ ] Environment
[ ] Summary
[ ] Description
[ ] Runbook URL
[ ] Routing rule
[ ] Deduplication/grouping
[ ] Inhibition if appropriate
[ ] Tested firing
[ ] Tested resolution
```

---

# 98. Alert Ownership

Every production alert should have a clear owner.

Example:

```yaml
labels:
  team: payments
  service: payment
```

If an alert fires at 2 AM, the responder should not have to determine ownership manually.

---

# 99. Team-Based Alert Routing

Example:

```yaml
routes:

  - matchers:
      - team="platform"
      - environment="prod"
    receiver: platform-oncall

  - matchers:
      - team="application"
      - environment="prod"
    receiver: application-oncall

  - matchers:
      - team="security"
      - environment="prod"
    receiver: security-oncall
```

Routing should be deterministic.

---

# 100. Environment-Based Routing

```yaml
routes:

  - matchers:
      - environment="prod"
      - severity="critical"
    receiver: prod-critical

  - matchers:
      - environment=~"dev|qa"
    receiver: engineering-nonprod
```

Production critical alerts should have the strongest escalation.

---

# 101. Critical vs Warning

Critical example:

```yaml
labels:
  severity: critical
```

Use when:

- users are significantly impacted
- availability is below an emergency threshold
- production capacity is failing
- cluster health is compromised

Warning example:

```yaml
labels:
  severity: warning
```

Use when:

- degradation is developing
- capacity needs attention
- immediate user impact is not established

---

# 102. Alert Routing with Continue

Example:

```yaml
- matchers:
    - severity="critical"
    - environment="prod"
  receiver: prod-critical
  continue: true

- matchers:
    - team="platform"
  receiver: platform-oncall
```

`continue: true` allows the alert to continue evaluating subsequent sibling routes.

Use carefully to avoid duplicate notifications.

---

# 103. Alert Deduplication

Alertmanager deduplicates alerts based on alert identity and routing/grouping behavior.

Do not include highly dynamic values unnecessarily in labels.

Bad:

```yaml
labels:
  pod_uid: "<unique-pod-id>"
```

This can create separate alerts for every replacement pod.

Prefer stable dimensions:

```yaml
labels:
  service: catalogue
  environment: prod
  team: application
```

---

# 104. Dynamic Values Belong in Annotations

Prefer:

```yaml
annotations:
  description: "Pod {{ $labels.pod }} is restarting."
```

rather than adding highly dynamic pod identity into alert grouping labels unless it is operationally required.

---

# 105. Kubernetes Events

Useful commands:

```bash
kubectl get events \
  -n roboshop \
  --sort-by=.lastTimestamp
```

Filter:

```bash
kubectl get events \
  -n roboshop \
  --field-selector type=Warning
```

Events are useful for diagnosis but are not a replacement for durable centralized logging.

---

# 106. Production Debugging YAML Resources

Check deployment:

```bash
kubectl get deployment catalogue -n roboshop -o yaml
```

Check service:

```bash
kubectl get service catalogue -n roboshop -o yaml
```

Check ingress:

```bash
kubectl get ingress roboshop -n roboshop -o yaml
```

Check HPA:

```bash
kubectl get hpa catalogue -n roboshop -o yaml
```

Check PDB:

```bash
kubectl get pdb catalogue -n roboshop -o yaml
```

---

# 107. YAML Rollout Verification

```bash
kubectl rollout status \
  deployment/catalogue \
  -n roboshop \
  --timeout=5m
```

Then:

```bash
kubectl get rs \
  -n roboshop \
  -l app.kubernetes.io/name=catalogue
```

Confirm old ReplicaSets are scaled appropriately.

---

# 108. Production Manifest Anti-Patterns

Avoid:

```yaml
image: catalogue:latest
```

Avoid:

```yaml
privileged: true
```

unless explicitly required.

Avoid:

```yaml
runAsUser: 0
```

for ordinary workloads.

Avoid:

```yaml
resources: {}
```

in production.

Avoid:

```yaml
hostNetwork: true
```

unless required.

Avoid broad:

```yaml
verbs: ["*"]
resources: ["*"]
```

RBAC permissions.

Avoid plaintext credentials.

---

# 109. Example Production Application Bundle

A typical application bundle contains:

```text
catalogue/
|
+-- deployment.yaml
+-- service.yaml
+-- serviceaccount.yaml
+-- configmap.yaml
+-- networkpolicy.yaml
+-- hpa.yaml
+-- pdb.yaml
+-- servicemonitor.yaml
```

Ingress may be centralized or application-specific depending on the routing architecture.

---

# 110. Complete Application Dependency Example

```text
ALB
 |
 v
frontend
 |
 +--> catalogue
 +--> user
 +--> cart
        |
        +--> redis
 |
 +--> payment
        |
        +--> rabbitmq
 |
 +--> shipping
        |
        +--> rabbitmq
```

Every edge should be explicitly considered for:

- DNS
- network policy
- ports
- timeouts
- retries
- metrics
- logging
- alerts

---

# 111. Production YAML Lifecycle

```text
Developer
   |
   v
Edit Helm/YAML
   |
   v
Lint
   |
   v
Template
   |
   v
Schema validation
   |
   v
Security policy
   |
   v
Pull Request
   |
   v
Review
   |
   v
Git merge
   |
   v
Argo CD
   |
   v
EKS
   |
   v
Monitoring
```

---

# 112. Production YAML Ownership

Recommended ownership:

```text
Terraform
   |
   +--> AWS infrastructure

Helm
   |
   +--> application packaging

GitOps
   |
   +--> environment desired state

Argo CD
   |
   +--> reconciliation

PrometheusRule
   |
   +--> alert definitions

Alertmanager
   |
   +--> notification routing
```

Do not mix infrastructure responsibilities unnecessarily.

---

# 113. Complete Production YAML Validation Checklist

```text
Kubernetes:
[ ] API versions valid
[ ] Namespaces valid
[ ] Selectors match labels
[ ] Services target correct ports
[ ] Probes valid
[ ] Resource values valid
[ ] SecurityContext valid

Networking:
[ ] Ingress class correct
[ ] ALB annotations valid
[ ] TLS configured
[ ] NetworkPolicies tested
[ ] DNS egress allowed

Observability:
[ ] ServiceMonitor labels correct
[ ] Metrics endpoint exists
[ ] PrometheusRules load
[ ] PromQL tested
[ ] Alertmanager config valid
[ ] Routes tested

GitOps:
[ ] Argo Application valid
[ ] Project permissions correct
[ ] Repository allowed
[ ] Destination allowed
[ ] Sync policy intentional

Security:
[ ] No secrets committed
[ ] Non-root
[ ] No privilege escalation
[ ] RBAC least privilege
[ ] Image immutable
```

---

# 114. Production Incident YAML Example

During a release issue, the desired state might be reverted from:

```yaml
image:
  digest: "sha256:BAD_RELEASE"
```

to:

```yaml
image:
  digest: "sha256:KNOWN_GOOD_RELEASE"
```

Then:

```text
Git commit
   |
   v
Argo CD detects change
   |
   v
Sync
   |
   v
Kubernetes rollout
   |
   v
Health validation
```

The Git history remains the audit record.

---

# 115. Production Alert Example — Full Object

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: catalogue-production-alerts
  namespace: monitoring
  labels:
    release: prometheus
    team: application
    environment: prod
spec:
  groups:
    - name: catalogue.production
      interval: 30s

      rules:

        - alert: CatalogueUnavailable
          expr: |
            sum(
              kube_deployment_status_replicas_available{
                namespace="roboshop",
                deployment="catalogue"
              }
            ) < 1
          for: 5m

          labels:
            severity: critical
            team: application
            service: catalogue
            environment: prod
            category: availability

          annotations:
            summary: "Catalogue has no available replicas"
            description: "The catalogue deployment has had zero available replicas for more than five minutes."
            runbook_url: "https://runbooks.example.com/catalogue/unavailable"

        - alert: CatalogueHighErrorRate
          expr: |
            (
              sum(rate(http_requests_total{
                namespace="roboshop",
                service="catalogue",
                status=~"5.."
              }[5m]))
              /
              clamp_min(
                sum(rate(http_requests_total{
                  namespace="roboshop",
                  service="catalogue"
                }[5m])),
                0.001
              )
            ) > 0.05
          for: 10m

          labels:
            severity: critical
            team: application
            service: catalogue
            environment: prod
            category: reliability

          annotations:
            summary: "Catalogue error rate above 5%"
            description: "Catalogue is returning HTTP 5xx responses above the configured threshold."
            runbook_url: "https://runbooks.example.com/catalogue/high-error-rate"
```

---

# 116. How PrometheusRule Works

```text
Prometheus
   |
   +--> evaluates expression
   |
   +--> condition false
   |       |
   |       +--> no alert
   |
   +--> condition true
           |
           v
        `for` period
           |
           v
        FIRING
           |
           v
      Alertmanager
```

The `for` duration prevents short-lived spikes from immediately paging.

---

# 117. How Alertmanager Works

```text
Prometheus
    |
    v
Alert
    |
    v
Alertmanager
    |
    +--> Group
    |
    +--> Deduplicate
    |
    +--> Inhibit
    |
    +--> Route
    |
    +--> Receiver
    |
    v
Notification
```

Alertmanager does not replace Prometheus rule evaluation.

---

# 118. What Grafana Does

Grafana primarily provides:

- dashboards
- visualization
- exploration
- correlation

Prometheus provides:

- metrics collection
- query evaluation
- alert rule evaluation

Alertmanager provides:

- routing
- grouping
- inhibition
- notification handling

ELK provides:

- centralized log storage
- log search
- log analysis
- visualization through Kibana

---

# 119. Production Monitoring Separation

```text
Metrics
   |
   v
Prometheus
   |
   +--> Alert rules
   |
   v
Alertmanager

Metrics
   |
   v
Grafana
```

and:

```text
Logs
 |
 v
ELK
 |
 v
Kibana
```

This separation makes each observability component easier to reason about.

---

# 120. Senior Interview Questions

## Q1. Why do you use PrometheusRule instead of writing alerts in Grafana?

PrometheusRule keeps alert definitions with the Prometheus ecosystem and allows them to be version controlled and deployed declaratively through Kubernetes/GitOps. Grafana can still provide dashboards and may support alerting, but in this architecture Prometheus handles metric alert evaluation and Alertmanager handles routing.

## Q2. What is the purpose of Alertmanager?

Alertmanager receives alerts from Prometheus and handles grouping, deduplication, routing, inhibition, silencing and notification delivery.

## Q3. What is the difference between labels and annotations?

Labels are machine-readable dimensions used for grouping, routing and querying. Annotations provide human-readable context such as descriptions and runbook URLs.

## Q4. Why use `for: 10m`?

It prevents transient metric spikes from immediately becoming incidents. The alert must remain active for the specified duration before firing.

## Q5. What is alert inhibition?

It suppresses secondary alerts when a higher-level root-cause alert is already firing.

## Q6. Why are runbook URLs important?

They reduce mean time to recovery by giving responders an immediate investigation and mitigation path.

## Q7. Why shouldn't every warning page the on-call engineer?

Excessive paging creates alert fatigue. Critical alerts should represent actionable conditions requiring immediate response.

## Q8. What is an alert storm?

An alert storm occurs when one failure generates a large number of alerts. Grouping, inhibition, routing and good alert design reduce this noise.

## Q9. How would you alert on Kubernetes OOMKilled containers?

I would use kube-state-metrics metrics such as the last termination reason and alert when the `OOMKilled` condition is present, then correlate it with memory usage, resource limits, restart counts and application behavior.

## Q10. How do you test an alert?

I validate the PromQL, confirm the required metrics and labels exist, test the rule with `promtool`, deliberately reproduce or simulate the condition, verify Prometheus shows it as firing, verify Alertmanager routing and confirm the notification and resolution paths.

## Q11. How do you handle multi-cluster alerting?

I include cluster and environment labels and route alerts based on those dimensions. Prometheus instances can monitor individual clusters while a centralized operational process receives alerts from multiple environments.

## Q12. How do you avoid alert duplication?

Use stable labels, Alertmanager grouping and deduplication, and avoid putting unnecessary dynamic identifiers into labels.

---

# 121. Production Alerting Troubleshooting

## Alert never fires

Check:

```bash
kubectl get prometheusrule -A
```

Then inspect Prometheus.

Check:

```promql
up
```

Verify the source metric:

```promql
http_requests_total
```

Then execute the exact alert expression.

Common causes:

- metric missing
- wrong label
- wrong namespace
- incorrect service label
- query returns empty vector
- threshold never reached
- PrometheusRule not selected

---

# 122. Alert Fires but No Notification

Check:

```text
Prometheus
   |
   v
Alertmanager
```

Verify the alert is present in Alertmanager.

Then inspect:

- route matchers
- receiver
- inhibition
- silence
- notification integration
- credentials
- webhook connectivity

---

# 123. Too Many Notifications

Check:

```yaml
group_by:
group_wait:
group_interval:
repeat_interval:
```

Then check:

- duplicate alert rules
- dynamic labels
- missing inhibition
- overly broad routes
- thresholds
- flapping conditions

---

# 124. Alert Fires Repeatedly

Investigate whether:

```text
FIRING -> RESOLVED -> FIRING
```

is caused by the metric crossing the threshold.

Use:

- `for`
- smoothing
- better thresholds
- recording rules
- multi-window logic

---

# 125. PrometheusRule Not Loaded

Check:

```bash
kubectl get prometheusrule \
  -n monitoring
```

Then:

```bash
kubectl describe prometheusrule \
  catalogue-production-alerts \
  -n monitoring
```

Check Prometheus Operator selectors.

A common production issue is that the rule exists in Kubernetes but is not selected by Prometheus because required labels do not match.

---

# 126. Alertmanager Configuration Not Loaded

Validate:

```bash
amtool check-config alertmanager.yaml
```

Then inspect the Alertmanager pod:

```bash
kubectl logs \
  -n monitoring \
  <alertmanager-pod>
```

Look for:

- YAML parsing errors
- invalid matchers
- receiver configuration errors
- TLS failures
- authentication failures

---

# 127. ALB YAML Troubleshooting

Check:

```bash
kubectl describe ingress \
  roboshop \
  -n roboshop
```

Then inspect AWS Load Balancer Controller logs.

Possible causes:

- invalid annotation
- IAM permission issue
- certificate issue
- subnet discovery failure
- security group issue
- target health failure
- service port mismatch

---

# 128. Service YAML Troubleshooting

Check:

```bash
kubectl get service catalogue -n roboshop -o yaml
```

Then:

```bash
kubectl get endpointslice \
  -n roboshop \
  -l kubernetes.io/service-name=catalogue
```

If no endpoints exist:

```text
Service selector
        |
        X
Pod labels
```

do not match.

---

# 129. Deployment YAML Troubleshooting

Check:

```bash
kubectl describe deployment \
  catalogue \
  -n roboshop
```

Check:

```bash
kubectl get rs -n roboshop
kubectl get pods -n roboshop
```

Common causes:

- image pull failure
- insufficient resources
- failed probes
- scheduling failure
- invalid environment configuration
- admission policy rejection

---

# 130. HPA YAML Troubleshooting

Check:

```bash
kubectl get hpa catalogue -n roboshop
kubectl describe hpa catalogue -n roboshop
```

If metrics are unavailable:

```text
HPA
 |
 X
Metrics API
```

Check:

```bash
kubectl top pods -n roboshop
```

If `kubectl top` fails, investigate Metrics Server or the metrics pipeline.

---

# 131. NetworkPolicy YAML Troubleshooting

If application communication fails:

```bash
kubectl exec -n roboshop <pod> -- \
  getent hosts catalogue
```

Then test connectivity:

```bash
kubectl exec -n roboshop <pod> -- \
  curl -v http://catalogue:8080/health
```

Check policies:

```bash
kubectl get networkpolicy -n roboshop
```

Remember that NetworkPolicy behavior depends on the cluster's network plugin implementation.

---

# 132. Security YAML Review

For every workload ask:

```text
Does it need AWS access?
Does it need Kubernetes API access?
Does it need root?
Does it need write access to filesystem?
Does it need privileged capabilities?
Does it need host networking?
Does it need public exposure?
Does it need egress to the internet?
```

The correct production answer should be the minimum required access.

---

# 133. Production YAML Change Process

```text
Change requested
      |
      v
Developer updates Git
      |
      v
Lint / validation
      |
      v
Security policy
      |
      v
Pull Request
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
Monitoring
```

No manual production YAML edits should be necessary during normal operation.

---

# 134. Emergency Change

If emergency access is required:

```text
Incident
 |
 v
Emergency mitigation
 |
 v
Production change
 |
 v
Service restored
 |
 v
Document exact change
 |
 v
Commit equivalent GitOps change
 |
 v
Post-incident review
```

This prevents permanent configuration drift.

---

# 135. Complete YAML Architecture

```text
                       GitOps Repository
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
          Helm Values    Argo CD Apps     Monitoring
             |                |                |
             v                v                v
       Kubernetes      Application Sync   PrometheusRule
             |                                |
     +-------+-------+                        v
     |       |       |                   Prometheus
     v       v       v                        |
 Deployment Service Ingress                   v
     |       |       |                  Alertmanager
     |       |       |                        |
     |       |       +--> ALB                 v
     |       |                           On-call
     |       |
     |       +--> Service Discovery
     |
     +--> HPA
     +--> PDB
     +--> NetworkPolicy
     +--> ServiceAccount
     +--> RBAC
     +--> ConfigMap
     +--> Secrets
```

---

# 136. Final Production YAML Principles

1. Keep all desired state in Git.
2. Use Helm for reusable application packaging.
3. Use GitOps for environment state.
4. Use Argo CD for reconciliation.
5. Use immutable images.
6. Never commit real credentials.
7. Use least-privilege RBAC.
8. Run containers as non-root.
9. Drop unnecessary Linux capabilities.
10. Use readiness, liveness and startup probes appropriately.
11. Define CPU and memory requests.
12. Define CPU and memory limits where appropriate.
13. Use PDBs for critical replicated services.
14. Use HPA based on measured workload behavior.
15. Use NetworkPolicies intentionally.
16. Use ALB Ingress for the capstone's external HTTP entry point.
17. Use PrometheusRule for declarative metric alerts.
18. Use Alertmanager for routing and notification management.
19. Give every actionable alert an owner and runbook.
20. Test YAML before merge.
21. Test alert firing and resolution.
22. Monitor deployment health after every release.
23. Keep production manifests auditable.
24. Avoid manual changes.
25. Reconcile emergency changes back into Git.

---

# 137. Final Senior-Level Understanding

A senior DevOps engineer should not memorize YAML syntax alone.

The important skill is understanding the relationship:

```text
Git
 |
 v
Helm / Kubernetes YAML
 |
 v
Argo CD
 |
 v
EKS
 |
 +--> Workloads
 +--> Services
 +--> ALB
 +--> HPA
 +--> PDB
 +--> NetworkPolicy
 |
 v
Prometheus
 |
 +--> Recording Rules
 +--> Alert Rules
 |
 v
Alertmanager
 |
 +--> Grouping
 +--> Deduplication
 +--> Inhibition
 +--> Routing
 |
 v
On-call
```

When production fails, the engineer must understand the entire chain rather than looking at one YAML file in isolation.

---

# 138. Interview Summary

A strong production answer is:

> We manage Kubernetes desired state through GitOps. Helm provides reusable application templates, while environment-specific values define production configuration. Argo CD continuously reconciles the Git state into EKS. Applications use immutable ECR image references, resource requests and limits, health probes, PDBs, HPA, security contexts and NetworkPolicies. AWS ALB Ingress provides external traffic entry. Prometheus collects metrics and evaluates PrometheusRules, while Alertmanager handles grouping, routing, inhibition and notification. Grafana provides visualization and ELK provides centralized logging. All YAML changes are validated, reviewed, version controlled and traceable, and emergency changes are reconciled back into Git.

---

# 139. Completion Checklist

```text
Kubernetes YAML:
[✓] Namespace
[✓] ResourceQuota
[✓] LimitRange
[✓] Deployment
[✓] Service
[✓] ConfigMap
[✓] Secret pattern
[✓] ServiceAccount
[✓] RBAC
[✓] HPA
[✓] PDB
[✓] NetworkPolicy
[✓] Ingress
[✓] CronJob
[✓] Job
[✓] Scheduling controls

Monitoring YAML:
[✓] ServiceMonitor
[✓] PodMonitor
[✓] Recording rules
[✓] Node alerts
[✓] Pod alerts
[✓] Deployment alerts
[✓] OOM alerts
[✓] Disk alerts
[✓] Network alerts
[✓] Application alerts
[✓] Availability alerts
[✓] SLO alert pattern
[✓] Watchdog

Alertmanager:
[✓] Global configuration
[✓] Routes
[✓] Receivers
[✓] Grouping
[✓] Inhibition
[✓] Severity routing
[✓] Environment routing
[✓] Team routing
[✓] HA example

Argo CD:
[✓] AppProject
[✓] Application
[✓] Multi-cluster Application
[✓] ApplicationSet
[✓] Notification pattern
[✓] Drift/reconciliation

Production:
[✓] Validation
[✓] Security
[✓] Troubleshooting
[✓] Rollback model
[✓] Incident model
[✓] Interview preparation
```

---

# 140. Final Takeaway

Production YAML is not simply configuration.

It is executable infrastructure and operational policy.

The strongest production implementation connects:

```text
Application
      |
      v
Container
      |
      v
Kubernetes
      |
      +--> Security
      +--> Networking
      +--> Scaling
      +--> Availability
      +--> Observability
      |
      v
GitOps
      |
      v
Argo CD
      |
      v
Production
```

And the production feedback loop is:

```text
Deploy
  |
  v
Observe
  |
  v
Alert
  |
  v
Investigate
  |
  v
Mitigate
  |
  v
Recover
  |
  v
Improve
  |
  v
Update Git
```

That lifecycle is the foundation of a reliable DevOps/DevSecOps production platform.
