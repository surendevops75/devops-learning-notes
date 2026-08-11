# Prometheus Configuration

Prometheus configuration defines:

```
What to scrape
How often to scrape
How targets are discovered
Which labels are applied
Which metrics are filtered
Which rules are evaluated
How alerts are sent
How Prometheus behaves in the environment
```

The main configuration file is:

```
prometheus.yml
```

A simplified architecture is:

```
prometheus.yml
      ↓
Prometheus Server
      ↓
Service Discovery
      ↓
Scrape Configuration
      ↓
Targets
      ↓
Metrics
      ↓
TSDB
```

For a Kubernetes production environment:

```
Kubernetes
    ↓
Service Discovery
    ↓
ServiceMonitor / PodMonitor
    ↓
Prometheus Operator
    ↓
Prometheus
    ↓
TSDB
```

---

# 1. Prometheus Configuration File

The main configuration file is normally:

```text
/etc/prometheus/prometheus.yml
```

In Kubernetes, the configuration is commonly generated and managed by the Prometheus Operator rather than manually editing a traditional `prometheus.yml`.

---

# 2. Basic Configuration

A simple configuration:

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets:
          - "localhost:9090"
```

This configuration contains:

```
Global settings
Scrape configuration
Target configuration
```

---

# 3. Configuration Structure

A Prometheus configuration can contain major sections such as:

```yaml
global:

rule_files:

scrape_configs:

alerting:

remote_write:

remote_read:
```

The exact supported options depend on the Prometheus version.

---

# 4. Global Configuration

The `global` section defines defaults.

Example:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
```

The two important concepts are:

```
scrape_interval

evaluation_interval
```

---

# 5. Scrape Interval

Example:

```yaml
global:
  scrape_interval: 15s
```

This means Prometheus attempts to scrape targets every 15 seconds unless a job overrides the interval.

Architecture:

```
Target
   ↑
   |
Every 15 seconds
   |
Prometheus
```

---

# 6. Evaluation Interval

Example:

```yaml
global:
  evaluation_interval: 15s
```

This controls how frequently rule groups are evaluated.

Rules include:

```
Recording Rules

Alerting Rules
```

Example:

```
Metrics
   ↓
PromQL Rule
   ↓
Every 15s
   ↓
Evaluation
```

---

# 7. Scrape Interval vs Evaluation Interval

Scrape interval:

```
How often metrics are collected.
```

Evaluation interval:

```
How often rules are evaluated.
```

Example:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 30s
```

This means:

```
Scraping → every 15 seconds

Rules → every 30 seconds
```

They solve different problems.

---

# 8. Scrape Timeout

Example:

```yaml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
```

The timeout should not exceed the scrape interval.

Conceptually:

```
Scrape
   ↓
Wait for response
   ↓
Timeout
   ↓
Scrape Failure
```

---

# 9. Job Configuration

A scrape job defines how a group of targets is monitored.

Example:

```yaml
scrape_configs:
  - job_name: "node-exporter"
    static_configs:
      - targets:
          - "node1:9100"
          - "node2:9100"
```

The job name becomes an important label:

```text
job="node-exporter"
```

---

# 10. Static Configuration

Static configuration explicitly defines targets.

Example:

```yaml
scrape_configs:
  - job_name: "linux-servers"
    static_configs:
      - targets:
          - "10.0.1.10:9100"
          - "10.0.1.11:9100"
          - "10.0.1.12:9100"
```

Architecture:

```
prometheus.yml
      ↓
Static Targets
      ↓
Prometheus
      ↓
Scrape
```

---

# 11. Static Configuration Use Cases

Static targets are useful for:

```
Small Environments

Lab Environments

Stable Servers

Testing
```

They become difficult to maintain when infrastructure changes frequently.

---

# 12. Dynamic Configuration

Production cloud environments usually benefit from dynamic discovery.

Architecture:

```
Cloud / Kubernetes
      ↓
Service Discovery
      ↓
Prometheus
      ↓
Dynamic Targets
```

This avoids manually updating IP addresses.

---

# 13. Kubernetes Configuration

In Kubernetes, monitoring commonly uses:

```
ServiceMonitor

PodMonitor

Prometheus Operator
```

Instead of manually listing every pod.

Architecture:

```
Kubernetes API
     ↓
Prometheus Operator
     ↓
ServiceMonitor / PodMonitor
     ↓
Prometheus
     ↓
Application
```

---

# 14. Job Labels

Suppose:

```yaml
job_name: "node-exporter"
```

Prometheus adds:

```text
job="node-exporter"
```

A target may therefore produce:

```text
up{
  job="node-exporter",
  instance="10.0.1.10:9100"
}
```

Labels are fundamental to Prometheus querying.

---

# 15. Instance Label

Prometheus normally identifies the target using an `instance` label.

Example:

```text
instance="10.0.1.10:9100"
```

This allows engineers to distinguish multiple targets within the same job.

---

# 16. Static Target Example

```yaml
scrape_configs:
  - job_name: "linux"
    static_configs:
      - targets:
          - "10.0.1.10:9100"
          - "10.0.1.11:9100"
```

Prometheus creates separate time series for each instance.

---

# 17. Multiple Jobs

A real environment can have multiple jobs:

```yaml
scrape_configs:

  - job_name: "prometheus"
    static_configs:
      - targets:
          - "localhost:9090"

  - job_name: "node-exporter"
    static_configs:
      - targets:
          - "10.0.1.10:9100"

  - job_name: "database"
    static_configs:
      - targets:
          - "10.0.1.20:9187"
```

This provides logical separation.

---

# 18. Configuration Reload

When configuration changes, Prometheus needs to reload the configuration.

A common approach is:

```bash
sudo systemctl restart prometheus
```

For production, a configuration reload can be preferable to a full process restart when supported by the deployment.

---

# 19. Reload Configuration Using HTTP

Prometheus can support configuration reload through its lifecycle endpoint when lifecycle management is enabled.

A common pattern is:

```bash
curl -X POST http://localhost:9090/-/reload
```

The exact behavior depends on how Prometheus was started.

---

# 20. Reload Using SIGHUP

Prometheus can also reload configuration when it receives the appropriate signal.

For example:

```bash
kill -HUP <prometheus-pid>
```

Use care when doing this in production.

---

# 21. Validate Configuration

Always validate configuration before applying it.

Use:

```bash
promtool check config /etc/prometheus/prometheus.yml
```

For a Kubernetes environment, validate the relevant YAML and monitoring resources through the Kubernetes/Helm workflow as well.

---

# 22. Why Configuration Validation Matters

Without validation:

```
Edit Configuration
      ↓
Restart Prometheus
      ↓
Invalid YAML
      ↓
Prometheus Fails
      ↓
Monitoring Outage
```

With validation:

```
Edit Configuration
      ↓
promtool check config
      ↓
Valid
      ↓
Reload
      ↓
Verify
```

---

# 23. Configuration Management

Production Prometheus configuration should be:

```
Version Controlled

Peer Reviewed

Tested

Audited

Reproducible
```

For example:

```text
monitoring/
├── prometheus/
│   ├── prometheus.yml
│   ├── rules/
│   └── alerts/
└── README.md
```

---

# 24. Git-Based Configuration

A production workflow:

```
Engineer
   ↓
Edit Configuration
   ↓
Git Commit
   ↓
Pull Request
   ↓
Review
   ↓
Validation
   ↓
Deploy
   ↓
Prometheus
```

This is safer than manually changing production servers.

---

# 25. Global Labels

Prometheus can apply external labels.

Example:

```yaml
global:
  external_labels:
    cluster: "production"
    environment: "prod"
```

This can help identify the source of metrics in centralized architectures.

---

# 26. External Labels

Suppose two Prometheus servers exist:

```text
Prometheus A
cluster="prod"

Prometheus B
cluster="staging"
```

Then a centralized system can distinguish the metrics.

External labels are especially useful for:

```
Multi-Cluster Monitoring

HA Architectures

Remote Storage

Federation
```

---

# 27. Scrape Configuration

The `scrape_configs` section controls metric collection.

Example:

```yaml
scrape_configs:
  - job_name: "application"
    scrape_interval: 15s
    scrape_timeout: 10s

    static_configs:
      - targets:
          - "application:8080"
```

---

# 28. Per-Job Scrape Interval

A job can override the global interval.

Example:

```yaml
global:
  scrape_interval: 30s

scrape_configs:
  - job_name: "critical-application"
    scrape_interval: 10s
    static_configs:
      - targets:
          - "application:8080"
```

Here:

```
Default = 30s

Critical application = 10s
```

---

# 29. Choosing Scrape Intervals

Do not automatically use the shortest possible interval.

Consider:

```
Metric Importance

Application Behavior

Alerting Requirements

Network Usage

Prometheus CPU

Storage Growth
```

For many applications:

```
15s
```

is a reasonable starting point.

But production values should be workload-driven.

---

# 30. Static Config with Labels

Example:

```yaml
scrape_configs:
  - job_name: "linux"
    static_configs:
      - targets:
          - "10.0.1.10:9100"
        labels:
          environment: "production"
          team: "platform"
```

This adds labels to the discovered target.

---

# 31. Target Labels

Example target:

```text
10.0.1.10:9100
```

Configured labels:

```text
environment="production"
team="platform"
```

A resulting metric can contain:

```text
job="linux"
instance="10.0.1.10:9100"
environment="production"
team="platform"
```

---

# 32. Why Labels Matter

Labels allow queries such as:

```promql
up{environment="production"}
```

or:

```promql
up{team="platform"}
```

They provide dimensions for filtering and aggregation.

---

# 33. Label Cardinality

Labels must be carefully designed.

Good:

```text
environment
region
service
method
status
```

Potentially dangerous:

```text
user_id
request_id
session_id
transaction_id
```

Unbounded labels can create huge numbers of time series.

---

# 34. Relabeling

Relabeling modifies target metadata before scraping.

Architecture:

```
Service Discovery
      ↓
Discovered Labels
      ↓
Relabeling
      ↓
Final Target
      ↓
Scrape
```

Relabeling is heavily used in Kubernetes environments.

---

# 35. Basic Relabeling

Example:

```yaml
relabel_configs:
  - source_labels: [__meta_example_label]
    target_label: environment
```

This copies one discovered label into another label.

The actual source labels depend on the service discovery mechanism.

---

# 36. Keep Targets

Relabeling can be used to keep only selected targets.

Conceptual example:

```yaml
relabel_configs:
  - source_labels: [__meta_example_environment]
    regex: production
    action: keep
```

Only matching targets remain.

---

# 37. Drop Targets

Example:

```yaml
relabel_configs:
  - source_labels: [__meta_example_environment]
    regex: development
    action: drop
```

Development targets are excluded.

This can reduce unnecessary scraping.

---

# 38. Replace Labels

Example:

```yaml
relabel_configs:
  - source_labels: [__meta_example_region]
    target_label: region
    action: replace
```

This allows discovered metadata to become normal Prometheus labels.

---

# 39. Target Relabeling vs Metric Relabeling

Target relabeling:

```
Discovery
   ↓
Target Relabeling
   ↓
Scrape
```

Metric relabeling:

```
Scrape
   ↓
Metric Relabeling
   ↓
TSDB
```

This distinction is extremely important.

---

# 40. Metric Relabeling

Example:

```yaml
metric_relabel_configs:
  - source_labels: [__name__]
    regex: "unwanted_metric"
    action: drop
```

This removes matching scraped metrics before storage.

---

# 41. Why Metric Relabeling Is Useful

It can help:

```
Remove Unnecessary Metrics

Reduce Storage

Reduce Cardinality

Control Data Volume
```

However, do not use metric relabeling as a substitute for fixing poor instrumentation.

---

# 42. Metric Filtering Example

Suppose an exporter exposes:

```text
metric_a
metric_b
metric_c
metric_d
metric_e
```

You only need:

```text
metric_a
metric_b
metric_c
```

Metric relabeling can drop the unnecessary metrics.

---

# 43. Scrape Parameters

A job can specify:

```yaml
scrape_configs:
  - job_name: "application"

    scrape_interval: 15s

    scrape_timeout: 10s

    metrics_path: /metrics

    scheme: http

    static_configs:
      - targets:
          - "application:8080"
```

Important settings include:

```
scrape_interval

scrape_timeout

metrics_path

scheme
```

---

# 44. Metrics Path

Default:

```text
/metrics
```

If the application exposes:

```text
/actuator/prometheus
```

configure:

```yaml
metrics_path: /actuator/prometheus
```

---

# 45. HTTP Scheme

Default scraping commonly uses:

```yaml
scheme: http
```

For TLS:

```yaml
scheme: https
```

The target must actually support the selected protocol.

---

# 46. TLS Configuration

For HTTPS endpoints, Prometheus can use TLS configuration.

Conceptual example:

```yaml
tls_config:
  ca_file: /etc/prometheus/certs/ca.crt
  cert_file: /etc/prometheus/certs/client.crt
  key_file: /etc/prometheus/certs/client.key
```

Only configure the options required by the target's security model.

---

# 47. Authentication

Some metrics endpoints require authentication.

Prometheus supports authentication mechanisms such as:

```
Basic Authentication

Bearer Token

TLS Client Authentication
```

The credentials should be stored securely.

Do not commit plaintext production secrets to Git.

---

# 48. Basic Authentication

Conceptual configuration:

```yaml
basic_auth:
  username: prometheus
  password_file: /etc/prometheus/secrets/password
```

Using a file rather than embedding credentials directly is generally preferable.

---

# 49. Bearer Token

Prometheus can use a bearer token for authenticated endpoints.

Conceptual example:

```yaml
bearer_token_file: /etc/prometheus/secrets/token
```

The token file should have restricted permissions.

---

# 50. Kubernetes Secrets

In Kubernetes, sensitive values should generally be managed using:

```
Kubernetes Secrets

External Secret Systems

Secret Management Platforms
```

rather than putting credentials directly into monitoring configuration stored in Git.

---

# 51. Kubernetes Service Discovery

Prometheus supports Kubernetes service discovery.

A conceptual configuration can use:

```yaml
scrape_configs:
  - job_name: "kubernetes-pods"
    kubernetes_sd_configs:
      - role: pod
```

The discovered targets then need appropriate relabeling.

---

# 52. Kubernetes Pod Discovery

With:

```yaml
kubernetes_sd_configs:
  - role: pod
```

Prometheus can discover pods through the Kubernetes API.

Architecture:

```
Kubernetes API
     ↓
Pod Discovery
     ↓
Relabeling
     ↓
Prometheus
     ↓
Scrape
```

---

# 53. Kubernetes Service Discovery Roles

Depending on the use case, Prometheus can discover Kubernetes resources using roles such as:

```
pod

service

endpoints / endpoint slices

node

ingress

and other supported resource roles
```

The exact configuration should match the Prometheus version and Kubernetes environment.

---

# 54. Kubernetes Relabeling

Kubernetes discovery exposes metadata labels such as:

```text
__meta_kubernetes_namespace
__meta_kubernetes_pod_name
__meta_kubernetes_pod_label_*
```

These can be used with relabeling.

Example concept:

```yaml
relabel_configs:
  - source_labels: [__meta_kubernetes_namespace]
    target_label: namespace
```

---

# 55. Kubernetes Namespace Filtering

Suppose only:

```text
production
```

should be monitored.

A relabeling strategy can keep only targets belonging to the desired namespace.

Conceptually:

```text
production
    ↓
KEEP

development
    ↓
DROP
```

In real production environments, namespace selection is often managed through Prometheus Operator selectors.

---

# 56. Kubernetes Pod Annotation Discovery

Older Kubernetes monitoring designs sometimes use pod annotations such as:

```text
prometheus.io/scrape: "true"
prometheus.io/path: "/metrics"
prometheus.io/port: "8080"
```

This can work with manually configured Kubernetes discovery and relabeling.

However, in modern Kubernetes environments using Prometheus Operator, `ServiceMonitor` and `PodMonitor` are often preferred.

---

# 57. ServiceMonitor Configuration

A ServiceMonitor example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: order-service
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: order-service

  endpoints:
    - port: metrics
      path: /metrics
      interval: 15s
```

---

# 58. ServiceMonitor Architecture

```
Kubernetes Service
      ↓
ServiceMonitor
      ↓
Prometheus Operator
      ↓
Prometheus Configuration
      ↓
Scrape Target
```

This means engineers do not have to manually edit the generated Prometheus configuration for every application.

---

# 59. PodMonitor Configuration

Example:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: PodMonitor
metadata:
  name: order-service
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: order-service

  podMetricsEndpoints:
    - port: metrics
      path: /metrics
      interval: 15s
```

---

# 60. Prometheus Selector Configuration

The Prometheus Operator must be configured to discover the intended ServiceMonitors and PodMonitors.

Conceptually:

```text
Prometheus
    ↓
ServiceMonitor Selector
    ↓
Matching ServiceMonitors
    ↓
Target Services
```

If the selector is wrong, Prometheus may not discover the ServiceMonitor.

---

# 61. Common ServiceMonitor Problem

Application exists:

```text
order-service
```

ServiceMonitor exists:

```text
order-service-monitor
```

But Prometheus does not scrape it.

Possible causes:

```
ServiceMonitor selector mismatch

Prometheus selector mismatch

Wrong namespace

Wrong port name

Wrong metrics path

Application not exposing metrics
```

---

# 62. Debugging ServiceMonitor

Check:

```bash
kubectl get servicemonitor -A
```

Inspect:

```bash
kubectl describe servicemonitor <name> -n <namespace>
```

Check the Service:

```bash
kubectl get svc -n <namespace>
```

Check labels:

```bash
kubectl get svc <service-name> -n <namespace> --show-labels
```

---

# 63. Check Application Endpoint

Test:

```bash
curl http://<service>:<port>/metrics
```

If the endpoint fails:

```
Fix Application
```

If endpoint works:

```
Check Service

Check ServiceMonitor

Check Prometheus selectors
```

---

# 64. Rule Files

Prometheus can load recording and alerting rules from files.

Example:

```yaml
rule_files:
  - /etc/prometheus/rules/*.yml
```

Architecture:

```
Rule Files
    ↓
Prometheus
    ↓
Rule Evaluation
```

---

# 65. Recording Rule Example

```yaml
groups:
  - name: application-recording-rules
    rules:
      - record: service:http_requests_rate5m
        expr: |
          sum by (service) (
            rate(http_requests_total[5m])
          )
```

This creates a new time series:

```text
service:http_requests_rate5m
```

---

# 66. Alerting Rule Example

```yaml
groups:
  - name: application-alerts
    rules:
      - alert: HighErrorRate
        expr: |
          rate(http_requests_total{status=~"5.."}[5m]) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High HTTP error rate"
```

---

# 67. `for` Duration

The:

```yaml
for: 5m
```

means the alert condition should remain active for the specified duration before the alert becomes firing.

This helps prevent alerts caused by brief spikes.

Architecture:

```
Condition True
     ↓
   1 min
     ↓
   3 min
     ↓
   5 min
     ↓
Alert Firing
```

---

# 68. Alert Labels

Example:

```yaml
labels:
  severity: critical
  team: platform
```

Labels can be used by Alertmanager for routing.

For example:

```text
severity=critical
```

can be routed to the on-call channel.

---

# 69. Alert Annotations

Annotations provide human-readable information.

Example:

```yaml
annotations:
  summary: "High HTTP error rate"
  description: "The service has a high 5xx error rate."
```

Annotations are useful for:

```
Engineers

Incident Response

Notification Messages
```

---

# 70. Alertmanager Configuration

Prometheus can be configured to send alerts to Alertmanager.

Conceptual configuration:

```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "alertmanager:9093"
```

In Kubernetes Operator deployments, Alertmanager integration is commonly managed declaratively.

---

# 71. Alert Flow

```text
Prometheus
    ↓
Alert Rule
    ↓
Alert Firing
    ↓
Alertmanager
    ↓
Grouping
    ↓
Routing
    ↓
Notification
```

---

# 72. Alertmanager Receivers

Alertmanager can route notifications to supported receivers such as:

```
Email

Slack-compatible integrations

PagerDuty

Webhooks
```

and others.

The exact integration depends on the environment and Alertmanager configuration.

---

# 73. Alertmanager Routing

Example concept:

```text
severity=critical
        ↓
    On-Call Team

severity=warning
        ↓
    Team Channel
```

Labels become important for routing.

---

# 74. Remote Write Configuration

Prometheus can send metrics to a remote system.

Conceptually:

```yaml
remote_write:
  - url: "https://remote-metrics.example/api/v1/write"
```

Production implementations normally require:

```
Authentication

TLS

Queue Configuration

Capacity Planning

Failure Handling
```

---

# 75. Remote Write Architecture

```text
Targets
   ↓
Prometheus
   ↓
Local TSDB
   │
   └────→ Remote Write
              ↓
       Long-Term Storage
```

This is useful for:

```
Long-Term Retention

Multi-Cluster Monitoring

Centralized Metrics
```

---

# 76. Remote Write Security

Do not expose a remote-write endpoint without appropriate security.

Consider:

```
TLS

Authentication

Network Restrictions

Credential Management

Rate / Capacity Controls
```

---

# 77. Remote Read

Prometheus can also be configured to read from compatible remote systems.

Conceptually:

```yaml
remote_read:
  - url: "https://remote-metrics.example/api/v1/read"
```

Remote read is architecture-dependent and should be evaluated carefully before use.

---

# 78. External Labels with Remote Storage

Example:

```yaml
global:
  external_labels:
    cluster: "production"
    region: "ap-south-1"
```

This helps identify where metrics originated.

---

# 79. Production AWS Example

Suppose there are:

```
EKS Production
EKS Staging
```

Each has Prometheus.

Production:

```text
cluster="prod"
```

Staging:

```text
cluster="staging"
```

Centralized storage can distinguish the two environments using those labels.

---

# 80. Configuration and Secrets

Never put secrets directly into a public Git repository.

Bad:

```yaml
basic_auth:
  username: admin
  password: MyProductionPassword
```

Better:

```text
Kubernetes Secret
        ↓
Prometheus
        ↓
Authenticated Endpoint
```

---

# 81. Configuration File Permissions

For Linux:

```bash
sudo chown prometheus:prometheus /etc/prometheus/prometheus.yml
```

Restrict access where appropriate.

Check:

```bash
ls -l /etc/prometheus/prometheus.yml
```

---

# 82. Prometheus Configuration in Kubernetes

With Prometheus Operator, configuration is generally generated from Kubernetes resources.

The workflow becomes:

```
ServiceMonitor
   +
PodMonitor
   +
PrometheusRule
   +
Prometheus CR
   ↓
Prometheus Operator
   ↓
Generated Prometheus Configuration
   ↓
Prometheus
```

This is preferable to manually editing generated configuration in an Operator-managed deployment.

---

# 83. Helm Values Management

For kube-prometheus-stack, maintain production configuration in a values file.

Example:

```text
monitoring/
├── helm/
│   ├── values-prod.yaml
│   ├── values-stage.yaml
│   └── values-dev.yaml
├── servicemonitors/
├── prometheusrules/
└── README.md
```

---

# 84. Multi-Environment Configuration

A real DevOps environment may use:

```text
Development
    ↓
values-dev.yaml

Staging
    ↓
values-stage.yaml

Production
    ↓
values-prod.yaml
```

This avoids using exactly the same configuration everywhere.

---

# 85. Production Values

Example:

```yaml
prometheus:
  prometheusSpec:
    retention: 15d

    resources:
      requests:
        cpu: 500m
        memory: 2Gi

      limits:
        cpu: "2"
        memory: 4Gi
```

These values are examples.

Production sizing must be based on:

```
Active Series

Ingestion Rate

Query Load

Retention
```

---

# 86. Helm Upgrade Workflow

A safe workflow:

```
Modify values
     ↓
Git Commit
     ↓
Pull Request
     ↓
Review
     ↓
helm template / validation
     ↓
Staging
     ↓
Production
     ↓
Verify
```

---

# 87. Helm Template

Before applying a Helm change, render the manifests:

```bash
helm template monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f values-prod.yaml
```

This helps identify unexpected generated resources.

---

# 88. Helm Diff

If the Helm Diff plugin is available:

```bash
helm diff upgrade monitoring prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f values-prod.yaml
```

This allows engineers to review changes before upgrading.

---

# 89. Configuration Deployment Through ArgoCD

For GitOps:

```text
Git
 ↓
Helm Chart + Values
 ↓
ArgoCD
 ↓
EKS
 ↓
Prometheus
```

The desired monitoring configuration lives in Git.

---

# 90. Configuration Drift

Without GitOps:

```text
Git Configuration
       ↓
Production
       ↓
Manual Change
       ↓
Drift
```

With GitOps:

```text
Git
 ↓
Desired State
 ↓
ArgoCD
 ↓
Production
```

ArgoCD can detect and reconcile drift.

---

# 91. Configuration Validation in CI/CD

A production pipeline can perform:

```text
Git Commit
   ↓
YAML Validation
   ↓
Helm Lint
   ↓
Helm Template
   ↓
Policy Checks
   ↓
Review
   ↓
Deploy
```

This reduces configuration-related production failures.

---

# 92. Helm Lint

Run:

```bash
helm lint prometheus-community/kube-prometheus-stack \
  -f values-prod.yaml
```

Depending on your workflow and chart version, validate the chart and values appropriately.

---

# 93. Prometheus Configuration Testing

Use:

```bash
promtool check config prometheus.yml
```

For rules:

```bash
promtool check rules rules/*.yml
```

This should be part of CI/CD where applicable.

---

# 94. Unit Testing Rules

Prometheus provides tooling for rule testing.

A test file can define:

```
Input Series

Evaluation Time

Expression

Expected Result
```

This is useful for production alert rules.

---

# 95. Rule Testing Workflow

```text
Alert Rule
   ↓
promtool Test
   ↓
Expected Result
   ↓
Pass
   ↓
Deploy
```

This helps avoid broken alerts.

---

# 96. Alert Rule Testing

Before production:

```text
Condition
   ↓
Expected Alert
   ↓
Evaluate
   ↓
Alert Firing
   ↓
Alertmanager
   ↓
Notification
```

Testing should cover the complete path.

---

# 97. Configuration Reload Safety

Never assume a configuration change is safe because YAML syntax is valid.

A configuration can be syntactically valid but operationally incorrect.

Examples:

```
Wrong Target

Wrong Port

Wrong Selector

Wrong Relabeling

Wrong Alert Expression
```

Therefore:

```
Syntax Validation
    +
Functional Validation
    +
Monitoring Verification
```

---

# 98. Common Configuration Mistake: Wrong Port

Example:

```yaml
targets:
  - "application:8080"
```

But the metrics endpoint is actually:

```text
application:9090
```

Prometheus will fail to scrape.

Check:

```bash
curl http://application:9090/metrics
```

---

# 99. Common Configuration Mistake: Wrong Path

Prometheus:

```yaml
metrics_path: /metrics
```

Application:

```text
/actuator/prometheus
```

Result:

```
404
```

Fix the metrics path.

---

# 100. Common Configuration Mistake: Wrong Label Selector

Service:

```yaml
labels:
  app: order-service
```

ServiceMonitor:

```yaml
selector:
  matchLabels:
    app: payment-service
```

Result:

```
No Matching Service
```

The selector must match the intended Service.

---

# 101. Common Configuration Mistake: Wrong Namespace

Service:

```text
namespace: production
```

ServiceMonitor:

```text
namespace: monitoring
```

Depending on the Prometheus Operator configuration, the ServiceMonitor may not discover the intended Service.

Always understand:

```
Service Namespace

ServiceMonitor Namespace

Prometheus Namespace

Selector Configuration
```

---

# 102. Common Configuration Mistake: High Cardinality

Bad metric:

```text
http_requests_total{
  user_id="12345"
}
```

Every user can create a new series.

Better:

```text
http_requests_total{
  service="order",
  method="GET",
  status="200"
}
```

Keep Prometheus labels bounded.

---

# 103. Common Configuration Mistake: Very Short Scrape Interval

Example:

```yaml
scrape_interval: 1s
```

This can create unnecessary:

```
CPU Usage

Network Traffic

Storage Growth
```

Unless there is a clear requirement, use a reasonable interval.

---

# 104. Common Configuration Mistake: Excessive Retention

Example:

```text
Prometheus
   ↓
Huge Local Disk
   ↓
Unlimited Retention
```

This is not an appropriate long-term storage strategy for many large environments.

Evaluate:

```
Remote Storage

Object Storage

Long-Term Metrics Platforms
```

---

# 105. Common Configuration Mistake: Public Access

Bad:

```text
Internet
   ↓
Prometheus:9090
```

Better:

```text
Engineer
   ↓
VPN / Secure Access
   ↓
Internal Monitoring
```

---

# 106. Common Configuration Mistake: Plaintext Secrets

Never commit:

```yaml
password: production-password
```

to Git.

Use:

```
Kubernetes Secrets

Secret Management Systems

External Secret Operators

Secure CI/CD Secret Stores
```

---

# 107. Configuration Debugging Workflow

When a target is missing:

```text
1. Is Application Running?
          ↓
2. Is /metrics Working?
          ↓
3. Is Service Correct?
          ↓
4. Is ServiceMonitor Correct?
          ↓
5. Does Prometheus Select ServiceMonitor?
          ↓
6. Is Target Visible?
          ↓
7. Is Target UP?
          ↓
8. Is Metric Present?
```

This prevents random troubleshooting.

---

# 108. Configuration Debugging Commands

Check Prometheus:

```bash
kubectl get pods -n monitoring
```

Check Services:

```bash
kubectl get svc -A
```

Check ServiceMonitors:

```bash
kubectl get servicemonitor -A
```

Check PodMonitors:

```bash
kubectl get podmonitor -A
```

Check rules:

```bash
kubectl get prometheusrule -A
```

---

# 109. Check Prometheus Targets

Inside Prometheus:

```text
Status
   ↓
Targets
```

Look for:

```
Endpoint

State

Labels

Last Scrape

Error
```

This is one of the first places to check when scraping fails.

---

# 110. Check Prometheus Configuration

Prometheus provides its active configuration through its UI/API.

This can help verify whether the expected scrape configuration is actually loaded.

This is especially important when using:

```
Prometheus Operator

Helm

GitOps
```

because the generated configuration may differ from what you expect.

---

# 111. Configuration Architecture in Operator-Based Kubernetes

The architecture is:

```text
Git
 ↓
Helm Values
 ↓
Prometheus CR
 ↓
ServiceMonitor
 ↓
PodMonitor
 ↓
PrometheusRule
 ↓
Prometheus Operator
 ↓
Generated Configuration
 ↓
Prometheus
```

This is the preferred mental model for Operator-managed Prometheus.

---

# 112. PrometheusRule Architecture

```text
PrometheusRule
      ↓
Prometheus Operator
      ↓
Prometheus
      ↓
Rule Evaluation
      ↓
Alert
      ↓
Alertmanager
```

Recording rules follow:

```text
PrometheusRule
      ↓
Recording Rule
      ↓
Precomputed Metric
      ↓
PromQL / Grafana
```

---

# 113. Real-World Java Configuration

Suppose a Java microservice exposes:

```text
/actuator/prometheus
```

Kubernetes Service:

```yaml
ports:
  - name: metrics
    port: 9090
```

ServiceMonitor:

```yaml
endpoints:
  - port: metrics
    path: /actuator/prometheus
    interval: 15s
```

Architecture:

```text
Java
 ↓
Micrometer
 ↓
/actuator/prometheus
 ↓
Service
 ↓
ServiceMonitor
 ↓
Prometheus
```

---

# 114. Real-World Node.js Configuration

Suppose:

```text
/metrics
```

is exposed on:

```text
9090
```

Service:

```yaml
ports:
  - name: metrics
    port: 9090
```

ServiceMonitor:

```yaml
endpoints:
  - port: metrics
    path: /metrics
```

---

# 115. Real-World Python Configuration

Suppose:

```text
/metrics
```

is exposed on:

```text
8000
```

Service:

```yaml
ports:
  - name: metrics
    port: 8000
```

ServiceMonitor:

```yaml
endpoints:
  - port: metrics
    path: /metrics
```

---

# 116. Multi-Service Configuration

For a microservices platform:

```text
user-service
product-service
cart-service
order-service
payment-service
inventory-service
```

Each service can expose:

```text
/metrics
```

The monitoring architecture becomes:

```text
Microservices
      ↓
Services
      ↓
ServiceMonitors
      ↓
Prometheus
      ↓
TSDB
      ↓
Grafana
```

---

# 117. Environment Labels

A production platform should clearly distinguish environments.

Example:

```text
environment="production"
```

or:

```text
environment="staging"
```

This allows queries such as:

```promql
sum by (environment) (
  rate(http_requests_total[5m])
)
```

---

# 118. Region Labels

In AWS:

```text
region="ap-south-1"
```

can help distinguish metrics from different regions.

Example:

```promql
sum by (region) (
  rate(http_requests_total[5m])
)
```

---

# 119. Cluster Labels

For multiple EKS clusters:

```text
cluster="prod-eks"
```

Example:

```promql
sum by (cluster) (
  rate(http_requests_total[5m])
)
```

This is useful for centralized dashboards.

---

# 120. Service Labels

For microservices:

```text
service="order"
```

Example:

```promql
sum by (service) (
  rate(http_requests_total[5m])
)
```

This allows service-level dashboards.

---

# 121. Label Strategy

A good label model might include:

```text
environment
region
cluster
namespace
service
instance
method
status
```

Avoid adding identifiers that can create unbounded combinations.

---

# 122. Configuration Best Practice: Consistent Labels

Use consistent labels across environments.

For example:

```text
environment
cluster
namespace
service
```

This makes Grafana dashboards reusable.

Instead of creating completely different queries for:

```text
prod_service
production_service
service_name
app_name
```

standardize the label model.

---

# 123. Configuration Best Practice: Separate Environments

Use:

```text
values-dev.yaml
values-stage.yaml
values-prod.yaml
```

rather than editing production configuration manually.

---

# 124. Configuration Best Practice: GitOps

Use:

```text
Git
 ↓
Pull Request
 ↓
Review
 ↓
CI Validation
 ↓
ArgoCD
 ↓
EKS
```

This provides an auditable configuration lifecycle.

---

# 125. Configuration Best Practice: Test Rules

Before production:

```bash
promtool check rules rules/*.yml
```

Where appropriate, also use rule unit tests.

---

# 126. Configuration Best Practice: Monitor Configuration Changes

Every production configuration change should have:

```
Owner

Review

Change Record

Rollback Plan

Validation
```

This is especially important for alerting configuration.

---

# 127. Configuration Best Practice: Avoid Manual Production Editing

Avoid:

```text
SSH
 ↓
vim prometheus.yml
 ↓
Restart
```

for normal production operations.

Prefer:

```text
Git
 ↓
Review
 ↓
CI
 ↓
Deployment
```

---

# 128. Configuration Best Practice: Keep Configuration Modular

Separate:

```
Scrape Configuration

Recording Rules

Alerting Rules

ServiceMonitors

PodMonitors

Dashboards

Helm Values
```

This makes the monitoring platform easier to maintain.

---

# 129. Production Repository Example

A real repository can look like:

```text
monitoring/
├── prometheus/
│   ├── helm/
│   │   ├── values-dev.yaml
│   │   ├── values-stage.yaml
│   │   └── values-prod.yaml
│   │
│   ├── servicemonitors/
│   │   ├── order-service.yaml
│   │   ├── payment-service.yaml
│   │   └── inventory-service.yaml
│   │
│   ├── podmonitors/
│   │
│   ├── rules/
│   │   ├── infrastructure.yaml
│   │   ├── kubernetes.yaml
│   │   └── applications.yaml
│   │
│   └── README.md
```

---

# 130. Configuration Deployment Architecture

```text
Developer
   ↓
Git
   ↓
Pull Request
   ↓
CI
   ├── YAML Validation
   ├── Helm Lint
   ├── Helm Template
   └── Prometheus Rule Tests
   ↓
Approval
   ↓
ArgoCD
   ↓
EKS
   ↓
Prometheus Operator
   ↓
Prometheus
```

---

# 131. Production Configuration Example

Conceptually:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

  external_labels:
    environment: production
    cluster: prod-eks

rule_files:
  - /etc/prometheus/rules/*.yml

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets:
          - localhost:9090

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093
```

This is a conceptual example.

Kubernetes Operator deployments normally generate the final configuration from Kubernetes resources.

---

# 132. Production Configuration Review

Before deploying:

```text
Scrape Interval
       ↓
Timeout
       ↓
Targets
       ↓
Selectors
       ↓
Relabeling
       ↓
Rules
       ↓
Alerts
       ↓
Storage
       ↓
Security
```

Review each area.

---

# 133. Configuration and Observability

Prometheus configuration should support the broader observability platform.

Metrics:

```text
Prometheus
```

Logs:

```text
ELK
```

Traces:

```text
OpenTelemetry
   ↓
Jaeger
```

Visualization:

```text
Grafana
```

---

# 134. Configuration and Microservices

A production microservices platform should standardize:

```
Metrics Path

Metrics Port

Metric Names

Label Names

Service Names

Environment Labels
```

This makes monitoring configuration predictable.

---

# 135. Configuration and DevSecOps

Monitoring configuration should also pass security checks.

CI can inspect:

```
Plaintext Secrets

Unsafe Configuration

Public Endpoints

Excessive Permissions

Invalid Kubernetes Resources
```

The monitoring stack should follow the same DevSecOps principles as application infrastructure.

---

# 136. Configuration and Terraform

Infrastructure such as:

```
EKS

IAM

Security Groups

Storage
```

can be provisioned using Terraform.

Then:

```text
Terraform
   ↓
EKS Infrastructure
   ↓
ArgoCD
   ↓
Monitoring Stack
```

This separates infrastructure provisioning from Kubernetes application configuration.

---

# 137. Configuration and ArgoCD

ArgoCD manages the desired Kubernetes state.

Example:

```text
Git
 ├── Helm Values
 ├── ServiceMonitors
 ├── PodMonitors
 └── PrometheusRules
       ↓
     ArgoCD
       ↓
      EKS
```

This is a strong production deployment model.

---

# 138. Troubleshooting Scenario: Prometheus Configuration Reload Failed

Symptoms:

```text
Prometheus still using old configuration
```

Steps:

```
1. Validate configuration.

2. Check Prometheus logs.

3. Verify reload request.

4. Check active configuration.

5. Verify targets.
```

Example:

```bash
promtool check config /etc/prometheus/prometheus.yml
```

Then inspect logs:

```bash
journalctl -u prometheus -n 100
```

---

# 139. Troubleshooting Scenario: New Target Not Appearing

Check:

```text
1. Target exists
2. Discovery sees target
3. Relabeling keeps target
4. Correct port
5. Correct path
6. Network connectivity
7. Prometheus configuration
```

In Kubernetes also check:

```text
Service
ServiceMonitor
Prometheus selector
Namespace
Labels
```

---

# 140. Troubleshooting Scenario: Target Appears but Is DOWN

Check:

```text
Target Address
       ↓
Port
       ↓
Metrics Path
       ↓
Network
       ↓
TLS
       ↓
Authentication
       ↓
Application
```

Test manually:

```bash
curl http://<target>:<port>/metrics
```

---

# 141. Troubleshooting Scenario: Metric Missing

If target is UP but a metric is missing:

```
Check /metrics output

Check metric name

Check metric relabeling

Check labels

Check PromQL

Check whether the application actually exports the metric
```

Do not assume a target being UP means every expected metric exists.

---

# 142. Troubleshooting Scenario: Too Many Metrics

Symptoms:

```
High Memory

High Disk Growth

High Query Latency
```

Investigate:

```text
Active Series
       ↓
Metric Cardinality
       ↓
Scrape Volume
       ↓
Unnecessary Metrics
```

Then:

```
Remove unnecessary metrics

Fix labels

Adjust scraping

Review retention
```

---

# 143. Troubleshooting Scenario: Prometheus Configuration Is Correct but ServiceMonitor Not Working

Check:

```bash
kubectl get servicemonitor -A
```

Then:

```bash
kubectl describe servicemonitor <name> -n <namespace>
```

Check:

```
Selector

Port

Path

Namespace

Prometheus Selector
```

Then verify the Service itself.

---

# 144. Troubleshooting Scenario: Alert Rule Not Loading

Check:

```bash
promtool check rules rules/*.yml
```

Then inspect Prometheus rule status.

Verify:

```
YAML

Expression

Rule Group

PrometheusRule Selector

Namespace

Rule Evaluation
```

---

# 145. Troubleshooting Scenario: Alert Rule Loads but Does Not Fire

Check:

```
Metric Exists

Query Returns Data

Condition Is True

`for` Duration

Evaluation Interval

Labels
```

Example:

```promql
rate(http_requests_total{status=~"5.."}[5m])
```

Run the expression manually in Prometheus.

---

# 146. Troubleshooting Scenario: Alert Fires but Notification Missing

Follow:

```text
Prometheus
   ↓
Alert Rule
   ↓
Alert Firing
   ↓
Alertmanager
   ↓
Route
   ↓
Receiver
   ↓
Notification
```

Check every layer.

---

# 147. Configuration Security Checklist

```
[ ] No plaintext passwords

[ ] No tokens in Git

[ ] TLS where required

[ ] Restricted Prometheus access

[ ] Restricted Grafana access

[ ] RBAC reviewed

[ ] NetworkPolicies reviewed

[ ] Service accounts reviewed

[ ] Secrets managed securely
```

---

# 148. Configuration Reliability Checklist

```
[ ] Configuration version controlled

[ ] CI validation

[ ] promtool validation

[ ] Rule testing

[ ] Helm lint

[ ] Helm template review

[ ] Staging validation

[ ] Production approval

[ ] Rollback plan
```

---

# 149. Configuration Performance Checklist

```
[ ] Reasonable scrape intervals

[ ] Controlled cardinality

[ ] Unnecessary metrics removed

[ ] Query performance monitored

[ ] Recording rules used where appropriate

[ ] Retention reviewed

[ ] Remote storage evaluated where needed
```

---

# 150. Interview Question: Explain Prometheus Configuration.

### Answer

Prometheus configuration is primarily defined through `prometheus.yml`.

The major sections include:

```
global
scrape_configs
rule_files
alerting
remote_write
remote_read
```

The `global` section defines defaults such as:

```
scrape_interval

evaluation_interval
```

Individual scrape jobs define how targets are discovered and scraped.

In Kubernetes environments using Prometheus Operator, configuration is commonly managed declaratively through:

```
Prometheus

ServiceMonitor

PodMonitor

PrometheusRule
```

The Operator then generates the effective Prometheus configuration.

---

# 151. Interview Question: What Is the Difference Between Scrape Interval and Evaluation Interval?

### Answer

Scrape interval defines how often Prometheus collects metrics.

For example:

```yaml
scrape_interval: 15s
```

Evaluation interval defines how frequently recording and alerting rules are evaluated.

For example:

```yaml
evaluation_interval: 30s
```

So:

```
Scrape = Collect Metrics

Evaluation = Evaluate Rules
```

---

# 152. Interview Question: What Is Relabeling?

### Answer

Relabeling allows Prometheus to modify or filter target metadata discovered from service discovery.

For example, I can:

```
Keep Targets

Drop Targets

Rename Labels

Add Labels

Transform Metadata
```

It is especially important in Kubernetes environments.

---

# 153. Interview Question: Target Relabeling vs Metric Relabeling?

### Answer

Target relabeling happens before scraping.

```text
Discovery
   ↓
Target Relabeling
   ↓
Scrape
```

Metric relabeling happens after the target has been scraped.

```text
Scrape
   ↓
Metric Relabeling
   ↓
Storage
```

Target relabeling is mainly used for target selection and metadata.

Metric relabeling is mainly used for filtering or transforming scraped metrics.

---

# 154. Interview Question: How Would You Configure Prometheus for Kubernetes?

### Answer

For Kubernetes, I would generally use Prometheus Operator with kube-prometheus-stack.

I would configure:

```
Prometheus

ServiceMonitors

PodMonitors

PrometheusRules

Persistent Storage

Resource Requests/Limits

Retention

Alertmanager
```

Then I would manage the configuration through Helm values and GitOps.

---

# 155. Interview Question: How Do You Configure a Microservice for Prometheus Monitoring?

### Answer

First, the application exposes a Prometheus-compatible metrics endpoint.

For example:

```text
/metrics
```

Then I configure a Kubernetes Service with a named metrics port.

Then I create a ServiceMonitor or PodMonitor.

The flow is:

```text
Application
   ↓
/metrics
   ↓
Service
   ↓
ServiceMonitor
   ↓
Prometheus
```

Finally, I verify the target is UP and query the metrics in Prometheus.

---

# 156. Interview Question: Why Should You Not Manually Edit Prometheus Configuration in an Operator Deployment?

### Answer

In a Prometheus Operator deployment, the effective Prometheus configuration is generated from Kubernetes resources.

If I manually edit generated configuration, the Operator can overwrite my changes.

Therefore I would manage:

```
Prometheus CR

ServiceMonitor

PodMonitor

PrometheusRule

Helm Values
```

through the declarative Kubernetes/GitOps workflow.

---

# 157. Interview Question: How Would You Secure Prometheus Configuration?

### Answer

I would:

```
Avoid plaintext secrets

Use Kubernetes Secrets or an external secret manager

Restrict network access

Use TLS where required

Review RBAC

Restrict service account permissions

Keep configuration in a private Git repository

Run configuration validation in CI
```

---

# 158. Interview Question: How Would You Manage Prometheus Configuration Across Dev, Stage, and Production?

### Answer

I would maintain environment-specific configuration.

For example:

```text
values-dev.yaml
values-stage.yaml
values-prod.yaml
```

Common configuration would be reused, while environment-specific values would define:

```
Retention

Resources

Storage

External Labels

Alerting

Remote Storage
```

I would deploy through CI/CD or GitOps rather than manually editing production.

---

# 159. Interview Question: How Would You Troubleshoot a ServiceMonitor That Does Not Appear in Prometheus?

### Answer

I would check:

```
1. ServiceMonitor exists.

2. Prometheus selects that ServiceMonitor.

3. Namespace selection is correct.

4. Service selector matches.

5. Service exists.

6. Metrics port name matches.

7. Metrics path is correct.

8. Application exposes `/metrics`.

9. Prometheus Operator logs.

10. Prometheus target status.
```

This gives me a systematic path from Kubernetes resource discovery to actual scraping.

---

# 160. Interview Question: How Would You Prevent Prometheus Configuration From Breaking Production?

### Answer

I would implement:

```text
Git
 ↓
Pull Request
 ↓
YAML Validation
 ↓
promtool Validation
 ↓
Rule Tests
 ↓
Helm Lint
 ↓
Helm Template
 ↓
Staging
 ↓
Approval
 ↓
Production
```

I would also maintain:

```
Version Control

Change Review

Rollback Plan

Monitoring
```

This prevents configuration mistakes from directly reaching production.

---

# 161. Real-World Configuration Architecture

A production EKS environment can use:

```text
                    Git Repository
                         │
              ┌──────────┴──────────┐
              │                     │
         Helm Values          Monitoring CRs
              │                     │
              ├──────────┬──────────┤
              │          │          │
        ServiceMonitor PodMonitor PrometheusRule
              │          │          │
              └──────────┼──────────┘
                         ↓
                       ArgoCD
                         ↓
                        EKS
                         ↓
                Prometheus Operator
                         ↓
                  Prometheus
                    │      │
                    │      └────→ Alertmanager
                    │
                    └────→ Grafana
```

---

# 162. Complete Configuration Mental Model

Remember Prometheus configuration as:

```text
GLOBAL
  ↓
Defaults

SCRAPE_CONFIGS
  ↓
What to collect

SERVICE_DISCOVERY
  ↓
Where targets are

RELABELING
  ↓
Which targets and labels

RULE_FILES
  ↓
What to calculate / alert

ALERTING
  ↓
Where alerts go

REMOTE_WRITE
  ↓
Where metrics are copied

EXTERNAL_LABELS
  ↓
Where metrics came from
```

In Kubernetes:

```text
Helm
 ↓
Prometheus Operator
 ↓
Prometheus CR
 ├── ServiceMonitor
 ├── PodMonitor
 └── PrometheusRule
 ↓
Generated Configuration
 ↓
Prometheus
```

---

# 163. Final Production Configuration Checklist

## Global

```
[ ] scrape_interval

[ ] scrape_timeout

[ ] evaluation_interval

[ ] external_labels
```

---

## Scraping

```
[ ] Jobs

[ ] Targets

[ ] Service Discovery

[ ] Metrics Path

[ ] Scheme

[ ] TLS

[ ] Authentication
```

---

## Relabeling

```
[ ] Target relabeling

[ ] Metric relabeling

[ ] Keep/drop rules

[ ] Cardinality control
```

---

## Kubernetes

```
[ ] Prometheus CR

[ ] ServiceMonitor

[ ] PodMonitor

[ ] PrometheusRule

[ ] Namespace selectors

[ ] Label selectors
```

---

## Rules

```
[ ] Recording Rules

[ ] Alerting Rules

[ ] Rule Testing

[ ] Evaluation Interval
```

---

## Alerting

```
[ ] Alertmanager

[ ] Routing

[ ] Grouping

[ ] Receivers

[ ] Notification Testing
```

---

## Storage

```
[ ] Retention

[ ] Persistent Storage

[ ] Remote Write if required

[ ] External Labels
```

---

## Security

```
[ ] Secrets protected

[ ] TLS where required

[ ] Network access restricted

[ ] RBAC reviewed

[ ] No public Prometheus exposure
```

---

## DevOps

```
[ ] Git

[ ] Pull Request

[ ] CI Validation

[ ] Helm Lint

[ ] promtool

[ ] Rule Tests

[ ] Staging

[ ] GitOps

[ ] Rollback
```

The core principle is:

```text
DO NOT JUST CONFIGURE PROMETHEUS.

DESIGN THE CONFIGURATION SO THAT IT IS:

    DISCOVERABLE
    SECURE
    TESTABLE
    VERSION CONTROLLED
    REPRODUCIBLE
    SCALABLE
    OBSERVABLE
    RECOVERABLE
```

A production Prometheus configuration should therefore follow:

```text
Git
 ↓
Validate
 ↓
Review
 ↓
Deploy
 ↓
Verify
 ↓
Monitor
 ↓
Improve
```
