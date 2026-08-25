# GitOps-Progressive-Delivery.md

# GitOps Progressive Delivery

## 1. Purpose

Progressive delivery extends GitOps from:

```text
Deploy version
```

to:

```text
Deploy safely
Observe
Validate
Increase exposure gradually
Stop automatically when unhealthy
Rollback when required
```

This file covers production-oriented progressive delivery using Argo CD and Argo Rollouts, with AWS/EKS, Kubernetes, ALB Ingress, Prometheus, Grafana and the RoboShop microservices platform.

Core strategies covered:

- Rolling updates
- Blue/Green
- Canary
- Argo Rollouts
- Stable and canary services
- Traffic management
- AWS ALB integration concepts
- AnalysisTemplates
- Prometheus analysis
- Automated promotion
- Automated rollback
- Abort
- Experiments
- Pause conditions
- GitOps promotion
- Image promotion
- Environment promotion
- Production YAMLs
- Troubleshooting
- Interview preparation

---

# 2. Why Progressive Delivery Is Needed

Traditional deployment:

```text
Version 1
   |
   v
Version 2
   |
   v
100% traffic
```

If Version 2 has a production bug:

```text
100% users affected
```

Progressive delivery changes this:

```text
Version 1
   |
   +------------------+
                      |
                      v
                  Version 2
                  5% traffic
                      |
                  observe
                      |
                +-----+-----+
                |           |
              healthy     bad
                |           |
                v           v
             25%          abort
                |
               50%
                |
               100%
```

The key idea is controlled exposure.

---

# 3. Progressive Delivery vs Continuous Delivery

Continuous delivery answers:

```text
Can we release software frequently?
```

Progressive delivery answers:

```text
How do we expose a release safely?
```

A mature platform uses both.

---

# 4. Traditional Kubernetes Deployment

A Kubernetes Deployment commonly performs a rolling update:

```text
old Pods
  |
  +--> terminate/create
  |
  v
new Pods
```

This is useful, but it does not inherently provide:

```text
5% traffic
10% traffic
metric analysis
automated promotion
automated abort
```

---

# 5. Rolling Update

Example:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxUnavailable: 0
    maxSurge: 1
```

This controls Pod replacement.

It does not necessarily control traffic percentage precisely.

---

# 6. Rolling Update vs Canary

Rolling update:

```text
replace Pods gradually
```

Canary:

```text
send controlled traffic to new version
```

They are not identical.

---

# 7. Blue/Green Deployment

Blue/Green maintains two environments:

```text
Blue  = current version
Green = new version
```

Example:

```text
                Load Balancer
                     |
             +-------+-------+
             |               |
             v               v
          Blue v1         Green v2
```

Only one normally receives production traffic.

---

# 8. Blue/Green Traffic Switch

Before release:

```text
100% -> Blue
```

After validation:

```text
100% -> Green
```

The switch can be fast.

---

# 9. Blue/Green Advantages

Advantages:

- Simple mental model
- Fast cutover
- Easy validation
- Easy traffic rollback
- Can keep previous version available temporarily

---

# 10. Blue/Green Disadvantages

Costs can be higher because:

```text
old version
+
new version
```

may run simultaneously.

For large workloads this can significantly increase:

```text
CPU
memory
nodes
database connections
```

---

# 11. Canary Deployment

Canary exposes a small percentage of users to the new version.

Example:

```text
Stable v1 -> 95%
Canary v2 -> 5%
```

Then:

```text
5%
 |
 v
10%
 |
 v
25%
 |
 v
50%
 |
 v
100%
```

---

# 12. Canary Advantages

Canary reduces blast radius.

If Version 2 fails:

```text
5% affected
```

instead of:

```text
100% affected
```

This is particularly valuable for:

```text
payments
orders
authentication
critical APIs
```

---

# 13. Canary Disadvantages

Canary requires stronger observability.

You need to know:

```text
Is canary healthy?
```

That requires metrics such as:

```text
error rate
latency
HTTP 5xx
request rate
business KPIs
```

---

# 14. Progressive Delivery Architecture

```text
                         Git
                          |
                          v
                       Argo CD
                          |
                          v
                    Argo Rollouts
                          |
                +---------+---------+
                |                   |
                v                   v
             Stable              Canary
              v1                   v2
                \                   /
                 \                 /
                  +------Traffic---+
                          |
                          v
                       Users
                          |
                          v
                     Prometheus
                          |
                          v
                   AnalysisTemplate
                          |
                    +-----+-----+
                    |           |
                  healthy      bad
                    |           |
                    v           v
                promote       abort
```

---

# 15. Argo CD and Argo Rollouts

Argo CD:

```text
GitOps synchronization
```

Argo Rollouts:

```text
progressive workload delivery
```

They complement each other.

```text
Git
 |
 v
Argo CD
 |
 v
Rollout resource
 |
 v
Argo Rollouts controller
 |
 v
Canary / Blue-Green
```

---

# 16. Why Argo Rollouts Instead of Deployment

A standard Deployment supports:

```text
RollingUpdate
Recreate
```

Argo Rollouts adds advanced strategies such as:

```text
Canary
BlueGreen
Analysis
Pause
Promotion
Abort
Traffic management
Experiments
```

---

# 17. Argo Rollouts Custom Resource

Instead of:

```yaml
kind: Deployment
```

you can use:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
```

The Rollouts controller manages progressive behavior.

---

# 18. Basic Rollout

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: cart
  namespace: roboshop
spec:
  replicas: 6

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
          image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart@sha256:REPLACE
          ports:
            - containerPort: 8080

  strategy:
    canary:
      steps:
        - setWeight: 10
        - pause:
            duration: 5m
        - setWeight: 25
        - pause:
            duration: 10m
        - setWeight: 50
        - pause:
            duration: 10m
```

---

# 19. Rollout Selector

The selector:

```yaml
selector:
  matchLabels:
    app: cart
```

must match the Pod template labels.

Incorrect selectors can prevent the Rollout from managing Pods correctly.

---

# 20. Canary Steps

Example:

```yaml
steps:
  - setWeight: 10
  - pause:
      duration: 5m
  - setWeight: 25
  - pause:
      duration: 10m
```

The Rollout gradually increases exposure.

---

# 21. Pause

A pause allows observation before proceeding.

Example:

```yaml
- pause:
    duration: 10m
```

This gives monitoring systems time to detect problems.

---

# 22. Indefinite Pause

A pause can also be used for manual approval.

Conceptually:

```yaml
- pause: {}
```

The rollout remains paused until explicitly promoted or otherwise advanced.

---

# 23. Manual Promotion

During a manual gate:

```bash
kubectl argo rollouts promote cart -n roboshop
```

The exact command depends on the installed Rollouts CLI/plugin version.

---

# 24. Abort

If the canary is unhealthy:

```bash
kubectl argo rollouts abort cart -n roboshop
```

This tells the Rollouts controller to stop the rollout.

---

# 25. Rollback

A progressive delivery rollback returns traffic/workload behavior toward the stable revision.

A production rollback must also consider:

```text
database changes
API compatibility
configuration changes
external dependencies
```

---

# 26. Stable and Canary Replica Sets

During canary:

```text
Stable ReplicaSet
     |
     v
v1 Pods

Canary ReplicaSet
     |
     v
v2 Pods
```

The Rollouts controller coordinates these versions.

---

# 27. Stable Service

A stable Service identifies the stable workload.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cart-stable
  namespace: roboshop
spec:
  selector:
    app: cart
  ports:
    - port: 80
      targetPort: 8080
```

In production, selector details must align with the Rollouts traffic-management mechanism.

---

# 28. Canary Service

A canary Service can target the canary ReplicaSet.

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cart-canary
  namespace: roboshop
spec:
  selector:
    app: cart
  ports:
    - port: 80
      targetPort: 8080
```

The exact selector behavior is managed by the Rollouts controller when configured for traffic routing.

---

# 29. Why Services Matter

Services provide stable networking while the underlying Pods change.

Progressive delivery uses services to distinguish:

```text
stable traffic
```

from:

```text
canary traffic
```

when traffic-management integration requires it.

---

# 30. Traffic Management

Canary can be implemented through:

```text
replica weighting
Ingress
service mesh
load balancer
gateway
```

The method determines how accurately traffic percentages are controlled.

---

# 31. Replica-Based Canary

A basic canary can change the number of canary Pods.

Example:

```text
10 total Pods

9 stable
1 canary
```

This approximates:

```text
90/10
```

but does not guarantee exact traffic percentages.

---

# 32. Traffic-Based Canary

Traffic routing can explicitly control:

```text
5%
10%
25%
50%
```

independently of replica count.

This is more precise.

---

# 33. Why Traffic Routing Is Better for Critical Services

Suppose:

```text
stable = 3 Pods
canary = 1 Pod
```

Traffic is not necessarily exactly:

```text
75% / 25%
```

because load balancing and connection behavior affect distribution.

Explicit traffic routing provides stronger control.

---

# 34. AWS ALB

The user's production architecture uses:

```text
AWS ALB
+
Kubernetes Ingress
```

rather than API Gateway.

Progressive delivery therefore needs to fit the ALB-based traffic architecture.

---

# 35. ALB Progressive Delivery Concept

Conceptually:

```text
                    AWS ALB
                       |
              +--------+--------+
              |                 |
              v                 v
        Stable Target      Canary Target
             v1                v2
              \                 /
               \               /
                +-------------+
```

Traffic weighting depends on the supported Kubernetes/ALB integration and Rollouts configuration.

---

# 36. ALB Target Groups

AWS ALB commonly routes traffic to target groups.

Conceptually:

```text
ALB Listener
    |
    +--> Stable Target Group
    |
    +--> Canary Target Group
```

The Rollouts integration can coordinate routing when supported by the configured traffic-routing mechanism.

---

# 37. ALB Weighting

A weighted routing model can look like:

```text
Stable target group = 95
Canary target group = 5
```

Then:

```text
90 / 10
75 / 25
50 / 50
0 / 100
```

The exact implementation must be validated against the installed AWS Load Balancer Controller and Argo Rollouts versions.

---

# 38. Production ALB Architecture

```text
                    Internet
                       |
                       v
                  AWS ALB
                       |
               Listener :443
                       |
            +----------+----------+
            |                     |
            v                     v
       Stable TG              Canary TG
          v1                      v2
            \                     /
             \                   /
              +------EKS--------+
```

---

# 39. TLS

Use HTTPS:

```text
Client
  |
  | HTTPS
  v
AWS ALB
  |
  v
Kubernetes Service
```

TLS termination can be performed at ALB using AWS ACM certificates.

---

# 40. ALB Health Checks

Canary routing depends on healthy targets.

Health checks should use a real endpoint such as:

```text
/health
```

rather than simply checking that a TCP connection exists.

---

# 41. Kubernetes Probes

Application Pods should also have:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 10
  periodSeconds: 10

livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 20
```

Tune these for the actual application.

---

# 42. Readiness and Progressive Delivery

Readiness determines whether a Pod should receive Service traffic.

If canary Pods are not ready:

```text
canary traffic should not be sent
```

This is the first safety layer.

---

# 43. Readiness Is Not Enough

A Pod can be:

```text
Ready
```

but still have:

```text
high latency
5xx errors
incorrect business behavior
```

Therefore progressive delivery needs metric analysis.

---

# 44. AnalysisTemplate

Argo Rollouts can use AnalysisTemplates.

Example concept:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: cart-success-rate
  namespace: roboshop
spec:
  metrics:
    - name: success-rate
      interval: 1m
      successCondition: result[0] >= 0.99
      failureLimit: 2
      provider:
        prometheus:
          address: http://prometheus.monitoring.svc.cluster.local:9090
          query: |
            sum(rate(http_requests_total{app="cart",status=~"2.."}[5m]))
            /
            sum(rate(http_requests_total{app="cart"}[5m]))
```

The metric names must match the application's actual Prometheus instrumentation.

---

# 45. AnalysisTemplate Purpose

It defines:

```text
what to measure
how often
what success means
what failure means
```

---

# 46. Prometheus Analysis

Prometheus can provide:

```text
error rate
request rate
latency
CPU
memory
custom business metrics
```

For progressive delivery, focus on metrics that represent user impact.

---

# 47. Error Rate

Example concept:

```text
5xx / total requests
```

If error rate exceeds a threshold:

```text
abort canary
```

---

# 48. Latency

Canary may have:

```text
higher p95
higher p99
```

even if average latency appears normal.

For user-facing APIs, tail latency is often more useful.

---

# 49. Business Metrics

Technical health is not enough.

Examples:

```text
payment success rate
order completion rate
cart checkout success
authentication success
```

These can be powerful canary signals.

---

# 50. AnalysisRun

When a Rollout uses an AnalysisTemplate, an AnalysisRun evaluates the metrics.

Conceptually:

```text
Rollout
   |
   v
AnalysisRun
   |
   v
Prometheus
   |
   v
success / failure
```

---

# 51. Analysis Result

Possible outcome:

```text
Successful
```

or:

```text
Failed
```

or:

```text
Inconclusive
```

The Rollout reacts according to the configured strategy.

---

# 52. Inline Analysis

An analysis can be configured directly within Rollout steps.

Concept:

```yaml
steps:
  - setWeight: 10
  - pause:
      duration: 5m
  - analysis:
      templates:
        - templateName: cart-success-rate
```

---

# 53. Background Analysis

Some analysis runs can execute in the background while the rollout progresses.

This is useful for longer-running validation.

The exact strategy should be selected according to the risk of the release.

---

# 54. Failure Threshold

Example:

```yaml
failureLimit: 2
```

This means the analysis can tolerate a limited number of failed measurements before being considered failed.

Do not choose thresholds arbitrarily.

---

# 55. Measurement Interval

Example:

```yaml
interval: 1m
```

Short intervals detect problems quickly but can be noisy.

Long intervals reduce noise but increase detection delay.

---

# 56. Canary Analysis Design

A good canary analysis should consider:

```text
baseline
sample size
measurement interval
error budget
traffic volume
warm-up period
```

---

# 57. Low-Traffic Problem

If a service receives:

```text
2 requests/minute
```

then a five-minute error-rate calculation may be statistically weak.

Do not automatically abort based on one failed request.

---

# 58. Warm-Up Period

New versions may need time for:

```text
JIT compilation
cache warming
connection pools
application initialization
```

Allow a reasonable warm-up before evaluating strict metrics.

---

# 59. Baseline Comparison

Instead of asking:

```text
Is canary error rate < 1%?
```

you can ask:

```text
Is canary materially worse than stable?
```

This is more sophisticated and can reduce false positives.

---

# 60. Canary Metric Examples

```text
5xx rate < 1%
p95 latency < 300ms
CPU < 80%
payment success > 99%
```

Thresholds must reflect the actual service SLOs.

---

# 61. Production Analysis Strategy

Example:

```text
10%
 |
 v
warm-up 5 min
 |
 v
error-rate analysis
 |
 v
latency analysis
 |
 v
25%
 |
 v
analysis
 |
 v
50%
 |
 v
analysis
 |
 v
100%
```

---

# 62. Automated Promotion

If analysis succeeds:

```text
10%
 |
 v
25%
 |
 v
50%
 |
 v
100%
```

No manual intervention is required if policy allows automated progression.

---

# 63. Automated Abort

If analysis fails:

```text
canary
  |
  v
analysis failure
  |
  v
abort
  |
  v
stable version
```

This reduces mean time to recovery.

---

# 64. Abort vs Rollback

Abort:

```text
stop the current progressive operation
```

Rollback:

```text
return toward a previous known-good version
```

The exact resulting workload state depends on the Rollout strategy and configuration.

---

# 65. Blue/Green with Argo Rollouts

Example:

```yaml
strategy:
  blueGreen:
    activeService: cart-active
    previewService: cart-preview
    autoPromotionEnabled: false
```

This creates:

```text
Active = production
Preview = new version
```

---

# 66. Blue/Green Validation

Flow:

```text
deploy Green
   |
   v
preview service
   |
   v
tests
   |
   v
manual/automatic promotion
   |
   v
active service -> Green
```

---

# 67. Blue/Green Rollback

Before switching:

```text
Blue = active
Green = preview
```

After successful switch:

```text
Green = active
Blue = previous
```

If failure occurs:

```text
keep Blue active
```

or switch back according to the configured strategy.

---

# 68. Blue/Green Cost

Running both versions can require:

```text
2x compute
```

during deployment.

For a large service:

```text
100 Pods
```

temporary doubling can be expensive.

---

# 69. Canary vs Blue/Green

| Feature | Canary | Blue/Green |
|---|---|---|
| Gradual traffic | Excellent | Usually no |
| Fast switch | Moderate | Excellent |
| Resource cost | Lower | Higher |
| Metric-based progression | Excellent | Excellent |
| Blast radius | Low | Depends on cutover |
| Rollback | Fast | Very fast |
| Complexity | Higher | Moderate |

---

# 70. When to Use Canary

Use canary for:

```text
high traffic
high risk changes
critical APIs
frequent releases
strong observability
```

---

# 71. When to Use Blue/Green

Use blue/green for:

```text
fast cutover
easy rollback
short validation windows
applications that can afford duplicate capacity
```

---

# 72. Rolling vs Canary vs Blue/Green

```text
Rolling:
replace gradually

Canary:
expose gradually

Blue/Green:
switch environments
```

These are different dimensions of deployment behavior.

---

# 73. GitOps Progressive Delivery Flow

```text
Developer
   |
   v
Application Git
   |
   v
CI
   |
   +--> tests
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
ECR
   |
   v
GitOps PR
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
Argo Rollouts
   |
   v
Canary
   |
   v
Prometheus Analysis
   |
   +--> success --> promote
   |
   +--> failure --> abort
```

---

# 74. Git as the Promotion Record

The GitOps repository should record:

```text
which image
which environment
which rollout configuration
```

A promotion can therefore be represented as a Git commit.

---

# 75. Image Promotion

Example:

DEV:

```yaml
image:
  digest: sha256:AAA
```

QA:

```yaml
image:
  digest: sha256:AAA
```

PROD:

```yaml
image:
  digest: sha256:AAA
```

The same immutable artifact is promoted.

---

# 76. Why Rebuilding for Production Is Bad

If CI rebuilds the same source separately:

```text
DEV image != PROD image
```

even if the source commit appears identical.

Better:

```text
build once
scan
sign
promote same artifact
```

---

# 77. Production Git Repository

```text
gitops-repo/
├── applications/
│   └── roboshop/
│       ├── cart.yaml
│       └── payment.yaml
├── environments/
│   ├── dev/
│   │   └── roboshop/
│   ├── qa/
│   │   └── roboshop/
│   └── prod/
│       └── roboshop/
├── rollouts/
│   ├── cart/
│   └── payment/
├── analysis/
│   ├── error-rate.yaml
│   └── latency.yaml
└── platform/
    └── ingress/
```

---

# 78. Helm and Progressive Delivery

Helm can template:

```text
Rollout
Service
Ingress
AnalysisTemplate
ConfigMap
HPA
```

Example values:

```yaml
rollout:
  strategy: canary

canary:
  steps:
    - 10
    - 25
    - 50
```

---

# 79. Helm Values by Environment

DEV:

```yaml
canary:
  steps:
    - 50
    - 100
```

PROD:

```yaml
canary:
  steps:
    - 5
    - 10
    - 25
    - 50
    - 100
```

Production gets smaller increments.

---

# 80. Production Canary Strategy

A useful pattern:

```text
5%
 |
5 minute observation
 |
10%
 |
10 minute observation
 |
25%
 |
15 minute observation
 |
50%
 |
30 minute observation
 |
100%
```

Actual values should come from service risk and traffic volume.

---

# 81. Critical Payment Service

For payment:

```text
1%
 |
10m
 |
5%
 |
15m
 |
10%
 |
30m
 |
25%
 |
30m
 |
50%
 |
validation
 |
100%
```

This is intentionally slower.

---

# 82. Low-Risk Internal Service

For an internal service:

```text
25%
 |
5m
 |
50%
 |
5m
 |
100%
```

Progressive delivery should be risk-based.

---

# 83. Analysis Metrics for Payment

Possible metrics:

```text
payment success rate
payment failure rate
5xx rate
p95 latency
timeout rate
```

Business metrics are especially valuable.

---

# 84. Analysis Metrics for Cart

Possible metrics:

```text
add-to-cart success
cart API latency
5xx
dependency failures
```

---

# 85. Analysis Metrics for Orders

Possible metrics:

```text
order creation success
order processing latency
queue failure
database errors
```

---

# 86. Dependency Risk

A canary can appear unhealthy because a dependency is failing.

For example:

```text
cart v2
   |
   v
inventory
   |
   v
inventory outage
```

The rollout analysis must distinguish:

```text
application regression
```

from:

```text
dependency outage
```

---

# 87. Analysis Windows

Do not analyze an extremely short window if the service has sparse traffic.

Prefer enough samples to make the result meaningful.

---

# 88. Error Budget

If the service SLO is:

```text
99.9% availability
```

the progressive delivery threshold should align with the organization's error budget.

---

# 89. Observability Stack

The user's stack includes:

```text
Prometheus
Grafana
ELK
```

For progressive delivery:

```text
Prometheus
```

is particularly important for automated metric analysis.

Grafana is useful for:

```text
human visualization
```

ELK is useful for:

```text
log investigation
```

---

# 90. Progressive Delivery Observability

```text
Argo Rollouts
      |
      v
AnalysisRun
      |
      v
Prometheus
      |
      +--> error rate
      +--> latency
      +--> request rate
      +--> business metric
```

For investigation:

```text
Grafana
ELK
```

---

# 91. Metrics and Logs Together

Metric:

```text
5xx increased
```

Then:

```text
Grafana -> identify timing
ELK -> inspect application errors
```

This provides faster root-cause analysis.

---

# 92. Production Alerting

Alert on:

```text
Rollout aborted
Analysis failed
Canary error rate high
Canary latency high
Rollout stuck
```

Avoid alerting on every normal pause.

---

# 93. Rollout Status

Useful command:

```bash
kubectl argo rollouts get rollout cart -n roboshop
```

This provides rollout status and progression.

---

# 94. Rollout History

Useful:

```bash
kubectl argo rollouts history rollout cart -n roboshop
```

Use it to understand revision changes.

---

# 95. Rollout Status

Another useful command:

```bash
kubectl argo rollouts status rollout cart -n roboshop
```

This can wait for rollout completion.

---

# 96. Rollout Dashboard

The Rollouts dashboard can provide visual insight into:

```text
stable revision
canary revision
traffic
steps
analysis
```

Use only approved authenticated access in production.

---

# 97. Kubernetes Inspection

Standard commands remain essential:

```bash
kubectl get rollout -n roboshop
kubectl describe rollout cart -n roboshop
kubectl get rs -n roboshop
kubectl get pods -n roboshop
kubectl get events -n roboshop --sort-by=.lastTimestamp
```

---

# 98. AnalysisRun Troubleshooting

```bash
kubectl get analysisrun -n roboshop
kubectl describe analysisrun <name> -n roboshop
```

Check:

```text
metric result
provider
query
failure count
measurement window
```

---

# 99. Prometheus Query Troubleshooting

Before trusting an AnalysisTemplate, run the query directly in Prometheus.

Validate:

```text
returns data
labels are correct
time range is correct
result is numeric
```

---

# 100. Empty Metric Result

An empty result can happen because:

```text
no traffic
wrong metric name
wrong labels
wrong namespace
wrong service
wrong Prometheus endpoint
```

Do not automatically treat empty data as healthy.

---

# 101. Prometheus Endpoint

Inside the cluster:

```text
http://prometheus.monitoring.svc.cluster.local:9090
```

is only an example.

Use the actual Service name and namespace.

---

# 102. AnalysisTemplate Production Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: roboshop-cart-canary
  namespace: roboshop
spec:
  metrics:
    - name: success-rate
      interval: 1m
      count: 5
      successCondition: result[0] >= 0.99
      failureLimit: 2
      provider:
        prometheus:
          address: http://prometheus.monitoring.svc.cluster.local:9090
          query: |
            (
              sum(rate(http_requests_total{
                app="cart",
                status=~"2.."
              }[5m]))
            )
            /
            (
              sum(rate(http_requests_total{
                app="cart"
              }[5m]))
            )
```

The query must be adapted to the application's actual metrics.

---

# 103. Canary Rollout Production Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: cart
  namespace: roboshop
spec:
  replicas: 6

  revisionHistoryLimit: 5

  selector:
    matchLabels:
      app: cart

  template:
    metadata:
      labels:
        app: cart
    spec:
      serviceAccountName: cart
      securityContext:
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault

      containers:
        - name: cart
          image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart@sha256:REPLACE

          ports:
            - name: http
              containerPort: 8080

          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi

          readinessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 10
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 3

          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 30
            periodSeconds: 20
            timeoutSeconds: 2
            failureThreshold: 3

          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL

  strategy:
    canary:
      steps:
        - setWeight: 5
        - pause:
            duration: 5m

        - analysis:
            templates:
              - templateName: roboshop-cart-canary

        - setWeight: 10
        - pause:
            duration: 10m

        - analysis:
            templates:
              - templateName: roboshop-cart-canary

        - setWeight: 25
        - pause:
            duration: 15m

        - analysis:
            templates:
              - templateName: roboshop-cart-canary

        - setWeight: 50
        - pause:
            duration: 20m

        - analysis:
            templates:
              - templateName: roboshop-cart-canary
```

---

# 104. Why This Rollout Is Production-Oriented

It includes:

```text
immutable image
resource requests
resource limits
readiness
liveness
non-root
seccomp
capability drop
revision history
small initial canary
metric analysis
multiple validation points
```

---

# 105. Service Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cart
  namespace: roboshop
spec:
  type: ClusterIP
  selector:
    app: cart
  ports:
    - name: http
      port: 80
      targetPort: http
```

The service provides stable internal access to the Rollout-managed Pods.

---

# 106. ALB Ingress Concept

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
spec:
  ingressClassName: alb
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /cart
            pathType: Prefix
            backend:
              service:
                name: cart
                port:
                  number: 80
```

This is a basic ALB ingress example; progressive traffic routing requires the appropriate Rollouts traffic-routing integration and annotations/configuration supported by the installed versions.

---

# 107. Production ALB Requirements

Before implementing ALB progressive delivery verify:

```text
AWS Load Balancer Controller version
Argo Rollouts version
supported traffic routing integration
Ingress model
target type
health checks
security groups
TLS
```

Version compatibility is critical.

---

# 108. AWS Load Balancer Controller

The controller translates Kubernetes Ingress resources into AWS ALB configuration.

Conceptually:

```text
Ingress
   |
   v
AWS Load Balancer Controller
   |
   v
AWS ALB
```

Progressive delivery must coordinate with this controller.

---

# 109. ALB Failure Scenario

If the ALB controller is unhealthy:

```text
Argo Rollouts may update Kubernetes desired state
```

but:

```text
AWS traffic configuration may not change correctly
```

Check:

```bash
kubectl get pods -n kube-system
kubectl logs -n kube-system deploy/aws-load-balancer-controller
```

Deployment names can vary.

---

# 110. Canary Failure Scenario

Suppose:

```text
Canary = 5%
```

Prometheus reports:

```text
5xx = 8%
```

Expected workflow:

```text
AnalysisRun
   |
   v
failure
   |
   v
Rollout abort
   |
   v
stable remains primary
```

Then investigate:

```text
ELK
Grafana
Pod logs
events
dependencies
```

---

# 111. Canary Timeout

A rollout can become stuck if:

```text
pause never ends
analysis never completes
traffic routing fails
new Pods never become ready
```

Troubleshoot all layers.

---

# 112. Canary Pods Not Ready

Check:

```bash
kubectl get pods -n roboshop -l app=cart
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
```

Focus on:

```text
image
config
secret
probe
resource limit
dependency
```

---

# 113. Canary Pods CrashLoopBackOff

Use:

```bash
kubectl logs <pod> --previous -n roboshop
kubectl describe pod <pod> -n roboshop
```

Do not blame Argo Rollouts immediately.

The application may simply be broken.

---

# 114. Canary Has High Latency

Investigate:

```text
CPU throttling
memory pressure
database latency
network
downstream services
application code
```

Prometheus identifies the symptom; ELK can help identify the cause.

---

# 115. Canary Has Higher Error Rate

Compare:

```text
stable
```

against:

```text
canary
```

Check:

```text
same request path
same traffic profile
same dependencies
same configuration
```

---

# 116. Analysis False Positive

Possible causes:

```text
insufficient traffic
bad PromQL
short window
warm-up period
dependency incident
metric cardinality issue
```

Tune the analysis instead of simply raising the failure threshold.

---

# 117. Analysis False Negative

A canary can pass while users still experience problems.

Causes:

```text
wrong metric
insufficient sample
metric aggregation hides failures
business metric not included
```

This is why metric selection matters more than simply having an AnalysisTemplate.

---

# 118. Progressive Delivery and SLOs

An SLO might be:

```text
99.9% successful requests
p95 latency < 300ms
```

Progressive delivery should evaluate release behavior against these objectives.

---

# 119. Progressive Delivery and Error Budget

If a service is already consuming its error budget:

```text
do not aggressively increase release risk
```

A mature platform can integrate release decisions with reliability policy.

---

# 120. Release Risk Classification

Example:

```text
Low:
documentation/config change

Medium:
internal service

High:
public API

Critical:
payment/auth/order processing
```

Higher risk should mean:

```text
smaller canary
longer analysis
stronger approval
more rollback preparation
```

---

# 121. Manual Approval Strategy

Production can use:

```text
5%
 |
analysis
 |
manual approval
 |
25%
 |
analysis
 |
manual approval
 |
100%
```

This is useful when:

```text
business validation
customer support validation
release manager approval
```

is required.

---

# 122. Automated Strategy

For mature teams:

```text
5%
 |
automated analysis
 |
25%
 |
automated analysis
 |
50%
 |
automated analysis
 |
100%
```

This reduces human deployment overhead.

---

# 123. Hybrid Strategy

A practical production pattern:

```text
5%
 |
automated analysis
 |
25%
 |
manual approval
 |
50%
 |
automated analysis
 |
100%
```

This combines automation with business control.

---

# 124. Progressive Delivery for DEV

DEV can use:

```text
fast rollout
large steps
minimal analysis
```

because the blast radius is low.

---

# 125. Progressive Delivery for QA

QA can use:

```text
moderate canary
automated tests
integration tests
```

---

# 126. Progressive Delivery for PROD

PROD should use:

```text
small canary
strong health checks
metric analysis
approval policy where needed
automated abort
immutable images
audit
```

---

# 127. Multi-Cluster Progressive Delivery

For multiple EKS clusters:

```text
Git
 |
 v
Central Argo CD
 |
 +--> DEV Rollout
 |
 +--> QA Rollout
 |
 +--> PROD Rollout
```

Production rollout can be further staged:

```text
prod-cluster-1
      |
      v
validate
      |
      v
prod-cluster-2
      |
      v
validate
      |
      v
remaining clusters
```

---

# 128. Multi-Region Progressive Delivery

Example:

```text
ap-south-1
   |
   v
canary

validate

ap-southeast-1
   |
   v
canary

validate
```

This limits regional blast radius.

---

# 129. Multi-Account Strategy

Example:

```text
AWS Dev Account
AWS QA Account
AWS Prod Account
```

Central GitOps can coordinate deployment while maintaining:

```text
least-privilege cluster access
account isolation
environment-specific IAM
```

---

# 130. Progressive Delivery and Secrets

Canary should use the same approved secret source unless the environment requires different credentials.

Do not create ad-hoc production secrets for canary.

Use:

```text
AWS Secrets Manager
+
External Secrets Operator
```

where appropriate.

---

# 131. Progressive Delivery and ConfigMap

Configuration changes can be as risky as code changes.

A canary can validate:

```text
application image
+
configuration
```

together.

---

# 132. Database Compatibility

The biggest progressive delivery risk is often database schema incompatibility.

Before canary:

```text
verify backward compatibility
```

For example:

```text
v1 -> schema N
v2 -> schema N+1
```

must support coexistence if v1 and v2 run simultaneously.

---

# 133. Queue Compatibility

For asynchronous RoboShop components:

```text
RabbitMQ
```

message schemas should be backward compatible during rollout.

Avoid changing a message contract in a way that breaks old consumers.

---

# 134. Event Compatibility

During canary:

```text
old producer
new consumer
```

or:

```text
new producer
old consumer
```

may coexist.

Use versioned or backward-compatible contracts.

---

# 135. Progressive Delivery for Workers

Workers do not always receive HTTP traffic.

Canary can instead be based on:

```text
consumer group
queue partition
worker replica percentage
processing error rate
message latency
```

Traffic management differs from HTTP services.

---

# 136. Worker Analysis Metrics

Examples:

```text
messages processed
processing failures
processing latency
queue depth
dead-letter rate
```

---

# 137. Progressive Delivery for Frontend

For frontend services, canary can evaluate:

```text
HTTP 5xx
latency
asset loading
browser errors
business conversion
```

depending on observability instrumentation.

---

# 138. Production Rollout Checklist

```text
[ ] immutable image
[ ] approved Git revision
[ ] security scans passed
[ ] readiness probe
[ ] liveness probe
[ ] resource requests/limits
[ ] stable service
[ ] canary strategy
[ ] traffic routing
[ ] analysis template
[ ] Prometheus query tested
[ ] failure threshold
[ ] rollback plan
[ ] database compatibility
[ ] dependency compatibility
[ ] alerting
[ ] audit
```

---

# 139. Troubleshooting: Rollout Stuck

Run:

```bash
kubectl argo rollouts get rollout cart -n roboshop
```

Then:

```bash
kubectl describe rollout cart -n roboshop
```

Check:

```text
current step
pause
analysis
ReplicaSets
Pod readiness
traffic routing
```

---

# 140. Troubleshooting: Rollout Not Progressing

Possible causes:

```text
manual pause
failed analysis
unhealthy Pods
traffic routing error
insufficient replicas
controller issue
```

---

# 141. Troubleshooting: AnalysisRun Failed

```bash
kubectl get analysisrun -n roboshop
kubectl describe analysisrun <name> -n roboshop
```

Then execute the PromQL directly.

---

# 142. Troubleshooting: Analysis Has No Data

Check:

```text
Prometheus endpoint
metric name
labels
namespace
time range
traffic volume
```

---

# 143. Troubleshooting: Traffic Does Not Shift

Check:

```text
Rollout strategy
stable/canary services
Ingress
AWS Load Balancer Controller
ALB listener
target groups
health checks
traffic-routing configuration
```

---

# 144. Troubleshooting: ALB Targets Unhealthy

Check:

```bash
kubectl get ingress -n roboshop
kubectl describe ingress roboshop -n roboshop
kubectl get endpoints -n roboshop
```

Then inspect:

```text
Pod readiness
security groups
target port
health check path
ALB target registration
```

---

# 145. Troubleshooting: Canary Receives No Traffic

Possible causes:

```text
canary weight = 0
canary service wrong
traffic routing not configured
target group unhealthy
Ingress mismatch
```

---

# 146. Troubleshooting: Stable Receives Canary Traffic

Investigate:

```text
Service selectors
Rollout labels
Ingress routing
target groups
controller state
```

Never assume traffic percentages based only on Pod count.

---

# 147. Troubleshooting: Rollout Controller

Check:

```bash
kubectl get pods -n argo-rollouts
kubectl logs -n argo-rollouts <rollouts-controller-pod>
```

The namespace may differ in your installation.

---

# 148. Troubleshooting: Argo CD Sees Rollout OutOfSync

Check:

```bash
argocd app get roboshop-prod
argocd app diff roboshop-prod
```

Determine whether:

```text
Git changed
controller mutated status
ignored fields
generated fields
```

are responsible.

---

# 149. Troubleshooting: Rollout Aborted Automatically

Check:

```text
AnalysisRun
Prometheus
application logs
Grafana
ELK
dependency health
```

The correct response is investigation, not simply increasing thresholds.

---

# 150. Troubleshooting: Rollback Does Not Restore Service

Check:

```text
database schema
ConfigMap
Secret
image
Ingress
external dependency
persistent data
```

Rollback of application Pods cannot reverse irreversible data migrations.

---

# 151. Troubleshooting: Canary Works but Production Fails

Possible causes:

```text
traffic distribution
scale
load profile
large-user behavior
cache
database contention
dependency rate limits
```

A 5% canary may not reproduce production-scale conditions.

---

# 152. Production Validation Layers

Use multiple layers:

```text
Layer 1: Pod readiness
Layer 2: Kubernetes health
Layer 3: ALB health
Layer 4: application metrics
Layer 5: logs
Layer 6: business metrics
```

---

# 153. Defense in Depth

Progressive delivery should not depend on a single signal.

Example:

```text
readiness
+
5xx
+
latency
+
business success
```

gives stronger confidence.

---

# 154. Production YAML Organization

```text
rollouts/
├── cart-rollout.yaml
├── cart-service.yaml
├── cart-ingress.yaml
└── cart-analysis.yaml
```

This makes the deployment lifecycle easier to review.

---

# 155. Helm Organization

```text
helm/
└── roboshop/
    ├── Chart.yaml
    ├── values.yaml
    ├── values-dev.yaml
    ├── values-qa.yaml
    ├── values-prod.yaml
    └── templates/
        ├── rollout.yaml
        ├── service.yaml
        ├── ingress.yaml
        └── analysis-template.yaml
```

---

# 156. Production GitOps Review

Before merging a rollout change:

```text
What changed?
Which service?
Which image?
What is the canary percentage?
What are the success metrics?
What aborts the rollout?
What is the rollback plan?
Are database changes compatible?
```

---

# 157. Rollout Change Example

PR:

```text
cart v2.4.1
```

contains:

```text
image digest
canary steps
analysis threshold
```

Reviewers can evaluate release risk before deployment.

---

# 158. CI Responsibilities

CI should:

```text
build
test
scan
package
publish
```

CI should not normally directly run:

```bash
kubectl apply
```

against production in a pull-based GitOps model.

---

# 159. GitOps CD Responsibilities

Argo CD should:

```text
observe Git
render desired state
synchronize
track health
reconcile drift
```

Argo Rollouts should:

```text
control progressive workload delivery
```

---

# 160. Complete CI + GitOps + Progressive Delivery

```text
             Application Git
                    |
                    v
            Jenkins / GitHub Actions
                    |
       +------------+------------+
       |            |            |
      Test       SonarQube      Trivy
       |                         |
       +------------+------------+
                    |
                 Veracode
                    |
                    v
                 Docker
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
              Argo Rollouts
                    |
             +------+------+
             |             |
             v             v
          Stable        Canary
             |             |
             +------ALB----+
                    |
                    v
                  Users
                    |
                    v
               Prometheus
                    |
                    v
              AnalysisRun
                    |
            +-------+-------+
            |               |
          PASS            FAIL
            |               |
            v               v
         promote          abort
```

---

# 161. Production Architecture: Single EKS

```text
                        Git
                         |
                         v
                     Argo CD
                         |
                         v
                  Argo Rollouts
                         |
                +--------+--------+
                |                 |
                v                 v
             Stable            Canary
                |                 |
                +-------+---------+
                        |
                        v
                    AWS ALB
                        |
                        v
                      Users
                        |
                        v
                   Prometheus
```

---

# 162. Production Architecture: Multiple EKS

```text
                         Git
                          |
                          v
                    Central Argo CD
                          |
             +------------+------------+
             |            |            |
             v            v            v
          EKS-DEV      EKS-QA       EKS-PROD
                                      |
                                      v
                                Argo Rollouts
                                      |
                                  +---+---+
                                  |       |
                                  v       v
                               Stable   Canary
                                  |
                                  v
                                ALB
```

---

# 163. Progressive Delivery Across Clusters

Do not automatically roll out to every production cluster simultaneously.

A safer model:

```text
Cluster A
   |
   v
canary
   |
   v
validate
   |
   v
Cluster B
   |
   v
canary
   |
   v
validate
   |
   v
remaining clusters
```

This reduces regional or cluster-wide blast radius.

---

# 164. Progressive Delivery and ApplicationSet

ApplicationSet can generate Rollout-based Applications for:

```text
dev
qa
prod
```

and multiple clusters.

Example conceptual matrix:

```text
Services x Clusters
```

The generated Application deploys the appropriate Rollout configuration.

---

# 165. Progressive Delivery and App of Apps

App of Apps can bootstrap:

```text
Argo Rollouts controller
Prometheus
ALB controller
service Applications
```

Then service Applications manage individual Rollouts.

---

# 166. Controller Dependency

Argo CD can synchronize the Rollout resource only if:

```text
Argo Rollouts CRD
+
Argo Rollouts controller
```

already exist.

Use platform bootstrap ordering.

---

# 167. CRD Ordering

```text
Wave -2
Argo Rollouts CRDs

Wave -1
Argo Rollouts controller

Wave 0
Service Rollouts
```

This is an example pattern, not a universal requirement.

---

# 168. Progressive Delivery Security

The Rollouts controller can change:

```text
Pods
ReplicaSets
Services
traffic routing
```

Therefore it needs appropriate RBAC.

Do not grant it unrestricted cluster-admin access unless absolutely required and justified.

---

# 169. Analysis Security

Prometheus access should be:

```text
network restricted
authenticated where required
read-only
```

Analysis should not need write access to Prometheus.

---

# 170. Production RBAC Boundary

```text
Argo CD
   |
   v
Rollout resource
   |
   v
Argo Rollouts controller
   |
   v
Kubernetes resources
```

Users should not need direct write access to all those resources if GitOps is the control path.

---

# 171. Break-Glass Access

Emergency access should be:

```text
time-limited
audited
least privilege
documented
```

If someone uses:

```bash
kubectl argo rollouts abort
```

during an incident, record why and reconcile the final state through Git.

---

# 172. Progressive Delivery and Disaster Recovery

Back up:

```text
Git repository
Argo CD configuration
AppProjects
ApplicationSets
Applications
Rollout manifests
AnalysisTemplates
Helm values
```

Do not rely on Kubernetes runtime state as the only record.

---

# 173. Rebuild Principle

If the Argo CD management cluster is lost:

```text
recreate cluster
install Argo CD
restore GitOps bootstrap
```

Git should allow reconstruction of desired state.

---

# 174. Rollout Disaster Recovery

If a target EKS cluster is recreated:

```text
Argo CD
   |
   v
register cluster
   |
   v
Application
   |
   v
Rollout
   |
   v
new Pods
```

The application can be rebuilt from Git.

---

# 175. Progressive Delivery During Incident

If production is unstable:

```text
stop promotion
 |
 v
abort canary
 |
 v
keep stable version
 |
 v
investigate
```

Do not continue a rollout simply because the deployment pipeline expects it to finish.

---

# 176. Change Freeze

A platform may temporarily freeze progressive delivery when:

```text
major incident
dependency outage
database incident
capacity issue
```

The goal is to avoid increasing change during instability.

---

# 177. Progressive Delivery Metrics Dashboard

Grafana should ideally show:

```text
stable request rate
canary request rate
stable 5xx
canary 5xx
stable latency
canary latency
CPU
memory
restart count
business success rate
```

---

# 178. Canary Dashboard Layout

```text
                 Cart Canary

Traffic
Stable: 95%
Canary: 5%

Error Rate
Stable: 0.2%
Canary: 0.3%

P95 Latency
Stable: 180ms
Canary: 190ms

Status:
HEALTHY
```

This makes human approval faster.

---

# 179. Log Correlation

Logs should include:

```text
service
version
environment
request ID
trace/correlation ID
```

Even without distributed tracing, correlation IDs improve troubleshooting.

---

# 180. Version Labels

Add labels such as:

```yaml
metadata:
  labels:
    app: cart
    version: v2.4.1
    environment: prod
```

This makes Prometheus and logs easier to analyze.

---

# 181. Prometheus Labels

Metrics should make stable/canary comparison possible.

Concept:

```text
app="cart"
version="v2.4.1"
environment="prod"
```

Avoid unbounded high-cardinality labels.

---

# 182. Cardinality Risk

Do not use:

```text
user_id
request_id
session_id
```

as uncontrolled Prometheus labels.

This can cause high memory usage and poor query performance.

---

# 183. Progressive Delivery and HPA

HPA and Rollouts can interact.

Consider:

```text
canary replicas
HPA replicas
traffic weight
```

Avoid having controllers compete over replicas.

Test the interaction before production.

---

# 184. Progressive Delivery and PDB

A PodDisruptionBudget can protect availability during changes.

Example:

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
      app: cart
```

Ensure the policy is compatible with rollout capacity.

---

# 185. Progressive Delivery and Resource Limits

Canary capacity should account for:

```text
stable workload
+
canary workload
```

During rollout, resource usage increases.

Ensure the EKS node group can handle the temporary load.

---

# 186. Cluster Capacity

Before a canary:

```text
check node capacity
check autoscaling
check Pod requests
```

Otherwise the canary may fail simply because there is no capacity.

---

# 187. EKS Node Autoscaling

A canary may create extra Pods.

The cluster autoscaling mechanism should have enough time to react.

If scale-up takes:

```text
5 minutes
```

do not use a:

```text
30-second
```

canary pause expecting stable capacity.

---

# 188. Progressive Delivery Timing

Timing should consider:

```text
Pod startup
node scale-up
ALB registration
application warm-up
metric collection
```

The fastest possible rollout is not always the safest rollout.

---

# 189. Production Canary Timing

Example:

```text
Deployment:
2m startup

ALB:
1m target registration

Warm-up:
5m

Analysis:
5m

Total minimum meaningful observation:
~13m
```

This is why arbitrary 30-second pauses can be misleading.

---

# 190. Canary Sample Size

Suppose:

```text
5% traffic = 10 requests
```

A single error creates:

```text
10% error rate
```

That may not be statistically meaningful.

Wait for enough traffic.

---

# 191. Progressive Delivery and SRE

Progressive delivery reduces:

```text
blast radius
```

and can reduce:

```text
MTTR
```

when automated abort and rollback work correctly.

---

# 192. Production Failure Example

Release:

```text
cart v2.5.0
```

Canary:

```text
5%
```

Metrics:

```text
5xx stable = 0.2%
5xx canary = 8%
```

Analysis fails.

Action:

```text
abort canary
investigate
```

Likely result:

```text
stable remains serving users
```

---

# 193. Production Success Example

Release:

```text
payment v3.1.2
```

Steps:

```text
5%
10%
25%
50%
100%
```

At every stage:

```text
5xx < 0.5%
p95 < 300ms
payment success > 99.5%
```

Result:

```text
automatic promotion
```

---

# 194. Production Database Failure Example

Release includes:

```text
schema migration
```

Migration removes a column still used by v1.

Canary starts:

```text
v1 stable
v2 canary
```

Both access the database.

v1 fails.

Lesson:

```text
schema must support both versions during coexistence
```

---

# 195. Production Dependency Failure Example

Canary error rate rises because:

```text
inventory service
```

is down.

Do not immediately conclude:

```text
cart v2 is broken
```

Compare:

```text
stable cart
canary cart
inventory
```

---

# 196. Production ALB Failure Example

Canary Pods are healthy but ALB marks targets unhealthy.

Check:

```text
health check path
target port
security group
readiness
target type
```

The application may be healthy inside Kubernetes while unreachable through ALB.

---

# 197. Production Rollout Review

Before enabling automated promotion:

```text
PromQL tested
threshold validated
traffic verified
rollback tested
database compatibility verified
capacity verified
alerts configured
```

---

# 198. Rollback Drill

Do not wait for a real incident.

Test:

```text
deploy
canary
fail analysis
abort
restore stable
```

This validates:

```text
controller
traffic
metrics
rollback
alerts
runbooks
```

---

# 199. Game Day

A production platform can periodically simulate:

```text
high 5xx
latency spike
Pod failure
ALB target failure
Prometheus outage
Argo Rollouts controller outage
```

Then verify progressive delivery behavior.

---

# 200. Prometheus Failure

If Prometheus becomes unavailable:

```text
analysis cannot reliably evaluate
```

Do not treat missing monitoring data as:

```text
healthy
```

The rollout should use safe failure semantics appropriate to the release.

---

# 201. Monitoring Failure as a Deployment Risk

Progressive delivery depends on observability.

Therefore:

```text
No reliable metrics
      |
      v
No reliable automated promotion
```

A safe platform should fail closed for critical releases when analysis data is unavailable.

---

# 202. Analysis Provider Failure

Possible:

```text
Prometheus timeout
DNS failure
authentication failure
query error
```

Investigate the provider before changing rollout thresholds.

---

# 203. Production Analysis Checklist

```text
[ ] query returns data
[ ] query has correct labels
[ ] enough traffic
[ ] warm-up considered
[ ] threshold aligned to SLO
[ ] failureLimit justified
[ ] interval justified
[ ] timeout configured
[ ] missing data behavior understood
[ ] alerting configured
```

---

# 204. Advanced Canary Strategy

A mature strategy may use:

```text
Step 1:
5% traffic

Step 2:
Analysis

Step 3:
10%

Step 4:
Analysis

Step 5:
25%

Step 6:
Business validation

Step 7:
50%

Step 8:
Analysis

Step 9:
100%
```

---

# 205. Progressive Delivery with Approval

```text
Git PR
 |
 v
CI
 |
 v
Approval
 |
 v
Argo CD
 |
 v
5% canary
 |
 v
automated analysis
 |
 v
manual production approval
 |
 v
100%
```

---

# 206. Progressive Delivery Without Manual Approval

```text
Git merge
 |
 v
Argo CD
 |
 v
5%
 |
 v
analysis
 |
 v
25%
 |
 v
analysis
 |
 v
50%
 |
 v
analysis
 |
 v
100%
```

This is ideal for mature, well-instrumented services.

---

# 207. GitOps Governance

Progressive delivery should be represented in Git:

```text
rollout strategy
traffic steps
analysis
thresholds
```

Therefore a change to:

```text
5% -> 25%
```

should itself be reviewable.

---

# 208. Progressive Delivery and Git History

A Git commit can show:

```text
image changed
```

Another can show:

```text
canary changed
```

Another:

```text
analysis threshold changed
```

This creates a strong audit trail.

---

# 209. Anti-Pattern: kubectl Set Image

Avoid making production image changes directly:

```bash
kubectl set image ...
```

in a GitOps-controlled workload.

It creates:

```text
cluster state != Git state
```

---

# 210. Anti-Pattern: latest Tag

Avoid:

```yaml
image: cart:latest
```

because:

```text
same Git commit
+
different image contents
```

can become possible.

Use immutable artifacts.

---

# 211. Anti-Pattern: No Readiness Probe

Without readiness:

```text
new Pod may receive traffic too early
```

This is dangerous during progressive delivery.

---

# 212. Anti-Pattern: No Analysis

Canary without analysis:

```text
gradual deployment
```

but no automated quality gate.

It is safer than all-at-once deployment but weaker than metric-driven progressive delivery.

---

# 213. Anti-Pattern: Broad Ignore Differences

This hides drift.

Use narrow ownership-based rules.

---

# 214. Anti-Pattern: Arbitrary Metrics

Do not use:

```text
CPU alone
```

as the primary release decision for a user-facing API.

Prefer:

```text
error rate
latency
business success
```

with infrastructure metrics as supporting evidence.

---

# 215. Anti-Pattern: Tiny Sample

Do not promote after:

```text
3 requests
```

because the metric is statistically weak.

---

# 216. Anti-Pattern: Instant Rollout

If:

```text
5%
10%
25%
50%
100%
```

happens in:

```text
30 seconds
```

there may be insufficient time for:

```text
traffic
metrics
warm-up
```

---

# 217. Anti-Pattern: Ignoring Database

Application rollback does not necessarily rollback database schema.

Always design schema migrations for version coexistence.

---

# 218. Anti-Pattern: Testing Only Pod Health

Pod health can be:

```text
Healthy
```

while user experience is:

```text
Broken
```

Use application and business metrics.

---

# 219. Interview: What Is Progressive Delivery?

### Answer

> Progressive delivery is a release strategy where a new version is exposed gradually, observed using automated or manual validation, and promoted or rolled back based on defined health criteria. It reduces blast radius compared with an all-at-once deployment.

---

# 220. Interview: Canary vs Blue/Green?

### Answer

> Canary gradually exposes traffic to the new version, such as 5%, 10%, 25% and so on. Blue/Green maintains two environments and switches traffic from the current version to the new version. Canary provides gradual risk reduction, while blue/green provides fast cutover and rollback.

---

# 221. Interview: What Does Argo CD Do in Progressive Delivery?

### Answer

> Argo CD synchronizes the Rollout resource and its configuration from Git. Argo Rollouts then manages the progressive deployment lifecycle, including canary steps, analysis, pauses and promotion.

---

# 222. Interview: Why Use Argo Rollouts?

### Answer

> Kubernetes Deployments provide standard rolling updates, while Argo Rollouts adds advanced progressive strategies such as canary, blue/green, analysis, automated promotion and abort behavior.

---

# 223. Interview: What Is an AnalysisTemplate?

### Answer

> An AnalysisTemplate defines metrics and success/failure conditions used by Argo Rollouts to evaluate whether a release is healthy enough to continue.

---

# 224. Interview: Why Use Prometheus?

### Answer

> Prometheus provides measurable signals such as error rate, latency, request rate and business metrics. Argo Rollouts can query Prometheus during a rollout and use the results as automated release gates.

---

# 225. Interview: What Happens If Analysis Fails?

### Answer

> Depending on the rollout strategy and configuration, the rollout can be aborted or prevented from progressing. The stable version can continue serving traffic while the failed canary is investigated.

---

# 226. Interview: What Is a Canary Step?

### Answer

> A canary step changes the exposure of the new version or pauses progression. For example, a rollout can set weight to 10%, wait, run analysis, then increase to 25%.

---

# 227. Interview: Why Is Readiness Important?

### Answer

> Readiness prevents traffic from being sent to Pods that are not ready to serve requests. It is a foundational safety mechanism for progressive delivery.

---

# 228. Interview: Why Is Readiness Not Enough?

### Answer

> A Pod can be technically ready while the application has high latency, elevated 5xx errors or incorrect business behavior. Progressive delivery therefore needs application and business-level metrics.

---

# 229. Interview: How Do You Implement Canary on AWS EKS?

### Answer

> I run Argo Rollouts on EKS, use Kubernetes Services and the AWS Load Balancer Controller for ALB-based traffic routing where supported, and use Prometheus for analysis. Argo CD manages the Rollout configuration from Git. I verify compatibility among Argo Rollouts, the AWS Load Balancer Controller and the Kubernetes/Ingress configuration before production use.

---

# 230. Interview: Why Not Just Use ALB Weights Manually?

### Answer

> Manual ALB changes bypass GitOps and create operational drift. With a supported progressive delivery integration, traffic changes can be coordinated with the Rollout lifecycle and represented as deployment behavior managed by the platform.

---

# 231. Interview: How Do You Roll Back a Canary?

### Answer

> First stop or abort the rollout, then verify that stable traffic is serving correctly. If a version must be removed permanently, update Git to the known-good image revision and let Argo CD reconcile the desired state. I also verify database and configuration compatibility.

---

# 232. Interview: How Do You Handle Database Migrations During Canary?

### Answer

> I use backward-compatible expand-and-contract migrations. Both old and new application versions must be able to operate against the schema while they coexist. I avoid irreversible destructive schema changes before the old version is fully retired.

---

# 233. Interview: What If Prometheus Is Down?

### Answer

> Automated analysis cannot be trusted without metrics. For critical production releases, I prefer safe behavior that prevents blind promotion. I then restore monitoring or use an approved manual release process.

---

# 234. Interview: What Is the Difference Between Abort and Rollback?

### Answer

> Abort stops the current progressive operation. Rollback is the broader recovery action that returns the application toward a previous known-good version. The exact behavior depends on the rollout strategy, but both should be followed by reconciliation of Git to the intended final state.

---

# 235. Interview: How Do You Prevent a Canary From Consuming Too Many Resources?

### Answer

> I account for stable and canary capacity, set resource requests and limits, verify EKS node capacity and autoscaling behavior, and avoid aggressive canary steps that exceed cluster capacity.

---

# 236. Interview: How Do You Monitor Canary vs Stable?

### Answer

> I use version-aware Prometheus metrics to compare request rate, 5xx rate, latency and relevant business metrics between stable and canary. Grafana provides visualization and ELK supports log-level investigation.

---

# 237. Interview: How Would You Canary Payment?

### Answer

> I would start with a very small percentage, use longer observation windows, validate payment success rate, 5xx, timeout and latency metrics, and use automated abort for severe regressions. I would also verify database and payment-provider compatibility before promotion.

---

# 238. Interview: How Would You Canary a Low-Traffic Service?

### Answer

> I would avoid relying on a tiny sample. I might use a longer observation window, synthetic traffic, business validation or other statistically meaningful signals before promoting.

---

# 239. Interview: What Is Traffic-Based Canary?

### Answer

> Traffic-based canary explicitly controls how much user traffic reaches the new version, independent of simply changing replica counts. It provides more precise exposure control when a supported traffic-routing mechanism is available.

---

# 240. Interview: Replica-Based vs Traffic-Based Canary?

### Answer

> Replica-based canary approximates traffic through the number of Pods, while traffic-based canary explicitly controls routing. Traffic-based routing is generally more precise for critical services.

---

# 241. Interview: What Is Blue/Green Preview?

### Answer

> In a blue/green rollout, the new version can run behind a preview service while the current version remains active. Tests and validation can occur before switching production traffic to the preview version.

---

# 242. Interview: Why Is Blue/Green More Expensive?

### Answer

> Both versions may need to run simultaneously, so compute, memory and other resources can temporarily increase significantly.

---

# 243. Interview: What Metrics Would You Use?

### Answer

> I prioritize user-impact metrics: HTTP 5xx/error rate, p95/p99 latency, request success rate and service-specific business KPIs. CPU and memory are supporting signals rather than the only release gates.

---

# 244. Interview: What Is a Good Canary Percentage?

### Answer

> There is no universal percentage. It depends on traffic volume, service criticality, observability, capacity and SLOs. A critical payment service might start at 1–5%, while a low-risk internal service could start higher.

---

# 245. Interview: What Makes a Good Analysis Query?

### Answer

> It should measure a meaningful user-impact signal, use correct labels, have enough traffic and a suitable time window, return predictable results, and align with the service's SLO or error budget.

---

# 246. Interview: What If Canary Is Healthy but Stable Is Also Failing?

### Answer

> I would not promote blindly. I would determine whether there is a shared dependency or platform incident. Progressive delivery should distinguish application regression from environmental failure.

---

# 247. Interview: Explain RoboShop Progressive Delivery

### Answer

> For a RoboShop service such as Cart, CI builds and scans the image and publishes an immutable artifact to ECR. A GitOps PR updates the approved image digest and Rollout configuration. Argo CD synchronizes the Rollout. Argo Rollouts sends a small canary percentage through the approved EKS/ALB traffic path. Prometheus evaluates error rate, latency and service-specific metrics. Successful analysis increases traffic; failed analysis aborts the rollout. Grafana and ELK support investigation.

---

# 248. Interview: Why Keep CI and CD Separate?

### Answer

> CI should build, test, scan and publish artifacts. GitOps CD should synchronize the approved desired state. This separation improves auditability, reduces direct production credentials in CI and makes Git the deployment source of truth.

---

# 249. Interview: Why Immutable Images?

### Answer

> An immutable image digest guarantees that the artifact being promoted is the same artifact that was built and scanned. Mutable tags such as latest can point to different content over time and weaken deployment reproducibility.

---

# 250. Interview: What Is Progressive Delivery's Main Benefit?

### Answer

> Reduced blast radius. Instead of exposing every user to a potentially defective release immediately, we expose a controlled subset, measure it, and stop or promote based on evidence.

---

# 251. Senior-Level Scenario: Canary Has 5xx Spike

### Situation

```text
Stable 5xx = 0.2%
Canary 5xx = 6%
```

### Response

```text
1. Stop promotion.
2. Confirm AnalysisRun result.
3. Inspect canary logs.
4. Compare request patterns.
5. Check dependencies.
6. Inspect Grafana.
7. Inspect ELK.
8. Abort if application regression is confirmed.
9. Verify stable traffic.
10. Fix Git.
```

---

# 252. Senior-Level Scenario: Canary Latency Increases

Check:

```text
CPU throttling
memory
GC/JIT
database
network
dependency latency
ALB behavior
```

Do not immediately roll back without identifying whether the increase is caused by the canary.

---

# 253. Senior-Level Scenario: Canary Pods Healthy, ALB Unhealthy

Likely layers:

```text
Pod
 |
 v
Service
 |
 v
Target registration
 |
 v
ALB health check
```

Inspect each layer independently.

---

# 254. Senior-Level Scenario: Analysis Keeps Failing

Check:

```text
PromQL
metric labels
traffic volume
time window
Prometheus connectivity
threshold
dependency health
```

---

# 255. Senior-Level Scenario: Rollout Stuck at 25%

Possible:

```text
pause
analysis
manual approval
unhealthy Pods
traffic routing
controller
```

Use:

```bash
kubectl argo rollouts get rollout cart -n roboshop
```

and inspect events.

---

# 256. Senior-Level Scenario: Production Rollout Must Stop Immediately

Use the approved emergency procedure:

```bash
kubectl argo rollouts abort cart -n roboshop
```

Then verify:

```text
stable traffic
ALB
Pods
metrics
```

Finally reconcile Git.

---

# 257. Senior-Level Scenario: Database Migration Failed

Do not automatically retry indefinitely.

Check:

```text
migration Job
database connectivity
schema state
partial migration
locks
permissions
```

Then determine whether the migration is safely retryable.

---

# 258. Senior-Level Scenario: Argo CD Is Healthy but Rollout Is Not

This is possible because:

```text
Argo CD
```

can successfully synchronize:

```text
Rollout desired state
```

while:

```text
Argo Rollouts controller
```

fails to execute the progressive strategy.

Troubleshoot the Rollouts controller separately.

---

# 259. Senior-Level Scenario: Prometheus Is Healthy but Query Is Wrong

Argo CD and Rollouts may appear healthy.

The failure is:

```text
observability configuration
```

Test PromQL independently before trusting automated deployment gates.

---

# 260. Production Runbook: Canary Failure

```text
1. Confirm canary failure.
2. Stop promotion.
3. Check AnalysisRun.
4. Check Prometheus query.
5. Compare stable/canary metrics.
6. Inspect canary Pods.
7. Inspect ALB/traffic.
8. Inspect ELK.
9. Check dependencies.
10. Abort if regression confirmed.
11. Verify stable.
12. Fix Git.
13. Record incident.
```

---

# 261. Production Runbook: Successful Canary

```text
1. Confirm image digest.
2. Confirm Git revision.
3. Confirm Pods ready.
4. Confirm ALB healthy.
5. Run analysis.
6. Promote next weight.
7. Repeat analysis.
8. Reach 100%.
9. Confirm old revision cleanup.
10. Verify Grafana.
11. Record deployment.
```

---

# 262. Production Runbook: Blue/Green

```text
1. Deploy preview.
2. Verify readiness.
3. Verify preview service.
4. Run smoke tests.
5. Run analysis.
6. Approve promotion.
7. Switch active traffic.
8. Verify production.
9. Retain previous version according to policy.
10. Clean up when safe.
```

---

# 263. Progressive Delivery Maturity Levels

### Level 1

```text
Rolling updates
```

### Level 2

```text
Canary percentages
```

### Level 3

```text
Metric-based analysis
```

### Level 4

```text
Automated promotion/abort
```

### Level 5

```text
Business metrics
+
multi-cluster progressive rollout
+
policy automation
```

---

# 264. Recommended RoboShop Maturity

For the user's RoboShop environment:

```text
Jenkins/GitHub Actions
        |
        v
security gates
        |
        v
ECR immutable image
        |
        v
GitOps PR
        |
        v
Argo CD
        |
        v
Argo Rollouts
        |
        v
ALB canary
        |
        v
Prometheus analysis
        |
        v
Grafana + ELK
```

This is a strong production-oriented architecture.

---

# 265. Final Progressive Delivery Checklist

```text
[ ] Understand rolling updates
[ ] Understand blue/green
[ ] Understand canary
[ ] Understand Argo Rollouts
[ ] Understand Rollout CRD
[ ] Understand stable/canary
[ ] Understand traffic routing
[ ] Understand AWS ALB integration
[ ] Understand AnalysisTemplate
[ ] Understand AnalysisRun
[ ] Understand Prometheus analysis
[ ] Understand automated promotion
[ ] Understand automated abort
[ ] Understand pauses
[ ] Understand experiments
[ ] Understand database compatibility
[ ] Understand HPA interaction
[ ] Understand PDB interaction
[ ] Understand cluster capacity
[ ] Understand multi-cluster rollout
[ ] Understand GitOps promotion
[ ] Understand immutable images
[ ] Understand SLO-based analysis
[ ] Understand production rollback
[ ] Understand troubleshooting
```

---

# 266. Final Mental Model

Remember:

```text
Argo CD
=
desired-state synchronization

Argo Rollouts
=
progressive workload delivery

Prometheus
=
automated release evidence

Grafana
=
visual analysis

ELK
=
log investigation

AWS ALB
=
external traffic entry/routing

ECR
=
immutable image storage

Git
=
deployment source of truth
```

Complete production flow:

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
GitOps Repository
   |
   v
Argo CD
   |
   v
Argo Rollouts
   |
   v
5% Canary
   |
   v
Prometheus
   |
   +-------- PASS --------> 25% -> 50% -> 100%
   |
   +-------- FAIL --------> Abort -> Stable
```

The key production principle is:

> **Do not ask only whether the new version deployed successfully. Ask whether the new version is behaving better than the old version under real traffic.**

That is the central idea behind progressive delivery.
