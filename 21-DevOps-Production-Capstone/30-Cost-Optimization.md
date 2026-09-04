# Cost Optimization

## 1 Purpose

Cost optimization in a production DevOps environment is not simply about making AWS bills smaller.

The objective is:

```text
Deliver required reliability, performance, security and availability
at the lowest sustainable total cost.
```

For this capstone, the production platform includes:

```text
AWS
 ├── VPC
 ├── EKS
 ├── EC2 / managed node groups
 ├── ECR
 ├── ALB
 ├── NAT Gateway
 ├── CloudWatch where required
 └── supporting AWS services

Kubernetes
 ├── Applications
 ├── Services
 ├── Ingress
 ├── HPA
 ├── PDB
 └── observability

Observability
 ├── Prometheus
 ├── Grafana
 └── ELK
```

Each layer can create unnecessary cost if it is not designed carefully.

---

 # 2 Cost Optimization Principles

Production cost optimization should follow these principles:

1. Measure before optimizing.
2. Optimize waste before reducing capacity.
3. Protect production reliability.
4. Protect security.
5. Use workload-driven sizing.
6. Automate repetitive optimization.
7. Review costs continuously.
8. Tag resources consistently.
9. Allocate costs to teams and environments.
10. Optimize total cost of ownership, not only individual resources.

A cheap architecture that causes frequent outages is not cost optimized.

---

 # 3 Cost vs Reliability

A common mistake is:

```text
Reduce replicas
Reduce nodes
Delete monitoring
Remove backups
Disable redundancy
```

This may reduce the bill temporarily but increase:

```text
outage probability
MTTR
security risk
data-loss risk
engineering effort
```

The correct question is:

```text
What is the minimum cost that still satisfies production requirements?
```

---

 # 4 Cost Optimization Lifecycle

```text
Measure
   |
   v
Identify waste
   |
   v
Prioritize
   |
   v
Model change
   |
   v
Test
   |
   v
Implement
   |
   v
Monitor
   |
   v
Review
```

Cost optimization is a continuous operational process.

---

 # 5 AWS Cost Categories

For the capstone, major cost areas can include:

```text
EKS
EC2
EBS
NAT Gateway
ALB
ECR
CloudWatch
data transfer
observability infrastructure
database infrastructure
backup storage
DNS
```

The exact bill depends on workload, region, traffic, retention and architecture.

---

 # 6 EKS Cost Model

EKS cost has multiple components.

Conceptually:

```text
EKS control plane
+
worker compute
+
storage
+
networking
+
load balancing
+
observability
+
data transfer
```

Do not evaluate EKS cost only by looking at the EKS cluster charge.

The surrounding infrastructure often represents the larger portion.

---

 # 7 EKS Cluster Cost Optimization

Potential optimization areas:

- right-size node groups
- use multiple instance types
- use autoscaling
- remove unused nodes
- separate workload classes
- use Spot where appropriate
- schedule non-production workloads
- optimize pod requests
- optimize observability workloads
- review storage
- review networking

---

 # 8 Node Right-Sizing

Suppose a node has:

```text
8 vCPU
32 GiB RAM
```

but workloads consistently use:

```text
2 vCPU
8 GiB
```

The cluster may be over-provisioned.

Collect actual utilization before changing node size.

Useful commands:

```bash
kubectl top nodes
kubectl top pods -A
```

Metrics from Prometheus provide longer-term trends.

---

 # 9 Kubernetes Requests and Cost

Requests influence scheduling and therefore cluster capacity.

Example:

```yaml
resources:
  requests:
    cpu: "2"
    memory: "4Gi"
```

If the application actually requires:

```text
200m CPU
512Mi memory
```

the request is excessively high.

Consequences:

```text
pods consume scheduling capacity
more nodes are required
AWS compute cost increases
```

---

 # 10 Do Not Under-Request

Over-requesting wastes money.

Under-requesting creates different problems:

```text
CPU throttling
memory pressure
OOMKilled
evictions
unstable applications
```

The objective is:

```text
accurate requests
+
appropriate limits
```

based on observed production behavior.

---

 # 11 Resource Profiling

Use:

```bash
kubectl top pods -A
kubectl top nodes
```

Prometheus can provide historical analysis.

Example:

```promql
avg_over_time(
  container_cpu_usage_seconds_total[7d]
)
```

In practice, CPU usage should be evaluated using appropriate rate calculations and aggregation rather than raw cumulative counters.

For example:

```promql
sum by (pod) (
  rate(container_cpu_usage_seconds_total{
    container!="",
    image!=""
  }[5m])
)
```

---

 # 12 HPA and Cost

Horizontal Pod Autoscaling can reduce idle capacity.

Example:

```text
Normal:
3 replicas

Peak:
10 replicas
```

Instead of permanently running:

```text
10 replicas
```

HPA scales according to workload.

Example:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: payment
  namespace: roboshop
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: payment
  minReplicas: 3
  maxReplicas: 12
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 65
```

---

 # 13 HPA Trade-Off

More aggressive scale-down:

```text
lower cost
```

but may cause:

```text
cold starts
latency spikes
insufficient capacity
```

More replicas:

```text
higher cost
+
more resilience
```

The correct values should come from workload behavior.

---

 # 14 Cluster Autoscaling

If pods require more capacity:

```text
Pending pods
   |
   v
node autoscaling
   |
   v
new node
   |
   v
pods scheduled
```

When demand falls:

```text
unused node
   |
   v
scale down
```

This reduces idle compute cost.

---

 # 15 Autoscaling Failure and Cost

Bad autoscaling configuration can increase cost.

Example:

```text
HPA min = 10
Cluster minimum = 10 nodes
Actual workload = 2 pods
```

The platform remains expensive.

Another example:

```text
HPA max = 100
```

with an application bug causing endless requests.

This can create a scaling storm.

Cost controls must be combined with reliability and alerting.

---

 # 16 Spot Instances

AWS Spot capacity can provide substantial compute savings for workloads that tolerate interruption.

Good candidates:

- batch jobs
- CI runners
- stateless workers
- development environments
- fault-tolerant workloads

Risk:

```text
Spot capacity can be interrupted.
```

Do not place critical stateful production workloads on Spot without a validated architecture.

---

 # 17 Mixed Node Groups

A production EKS cluster can use:

```text
On-Demand nodes
+
Spot nodes
```

Example:

```text
system-critical
    → On-Demand

stateless scalable workers
    → Spot
```

Use taints/tolerations and node affinity to control placement.

---

 # 18 Spot Capacity Diversification

Do not depend on one Spot instance type.

Prefer multiple compatible types:

```text
m6i.large
m6a.large
m5.large
m5a.large
```

subject to workload compatibility and current regional availability.

This increases the chance of obtaining capacity.

---

 # 19 Node Consolidation

If many nodes have low utilization:

```text
node1 → 15%
node2 → 20%
node3 → 18%
node4 → 17%
```

consider consolidation.

But evaluate:

- PDBs
- topology constraints
- daemonsets
- local storage
- availability zones
- disruption budgets

Cost optimization must not create an availability incident.

---

 # 20 Environment-Based Optimization

Production:

```text
high availability
24x7
```

QA:

```text
lower capacity
scheduled shutdown
```

Development:

```text
minimal capacity
scheduled shutdown
```

A common strategy:

```text
PROD → always running
QA → business hours
DEV → on demand
```

---

 # 21 Non-Production Scheduling

Non-production resources can often be stopped outside working hours.

Example concept:

```text
Monday-Friday:
08:00 → start

20:00 → stop

Weekend:
stop
```

This can dramatically reduce unnecessary compute consumption.

Production resources should not be handled by the same simplistic schedule.

---

 # 22 Namespace Cost Allocation

Use labels:

```yaml
metadata:
  labels:
    environment: prod
    team: payments
    cost-center: engineering
    application: payment
```

This allows cost attribution.

Recommended dimensions:

```text
environment
team
application
business-unit
cost-center
owner
```

---

 # 23 AWS Tagging Strategy

Example:

```text
Environment = prod
Application = roboshop
Team = platform
Owner = devops
CostCenter = engineering
ManagedBy = terraform
Criticality = high
```

Tags support:

- cost reporting
- ownership
- automation
- inventory
- governance

---

 # 24 Terraform Tagging

Use common tags:

```hcl
locals {
  common_tags = {
    Environment = var.environment
    Application = "roboshop"
    ManagedBy   = "terraform"
    Team        = "platform"
    CostCenter  = "engineering"
  }
}
```

Apply consistently to supported resources.

---

 # 25 Cost Allocation

A production organization should be able to answer:

```text
How much does production cost?
How much does QA cost?
Which team costs the most?
Which service consumes the most?
How much is observability?
How much is networking?
```

Without attribution, optimization becomes guesswork.

---

 # 26 NAT Gateway Cost

NAT Gateway can become a significant cost component.

Architecture:

```text
Private subnet
    |
    v
NAT Gateway
    |
    v
Internet
```

Costs can come from:

- hourly NAT Gateway charge
- processed data
- cross-AZ traffic

High-volume workloads can make this expensive.

---

 # 27 NAT Gateway Optimization

Potential strategies:

- keep traffic in the same AZ where appropriate
- use VPC endpoints for AWS services
- avoid unnecessary internet paths
- reduce cross-AZ traffic
- inspect high-volume destinations

Do not remove NAT simply to save money if private workload connectivity requires it.

---

 # 28 VPC Endpoints

AWS services such as ECR and S3 can use VPC endpoints where supported.

Concept:

```text
EKS private subnet
      |
      v
VPC endpoint
      |
      v
AWS service
```

This can reduce:

```text
NAT dependency
data processing
network path complexity
```

depending on architecture and endpoint costs.

---

 # 29 ECR Cost Optimization

ECR costs can grow because of unused images.

Example:

```text
payment:1
payment:2
...
payment:5000
```

Use lifecycle policies to remove images that are no longer needed.

But retain:

```text
current production
previous production
approved rollback versions
```

---

 # 30 ECR Lifecycle Policy Example

Conceptual example:

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Retain recent production images",
      "selection": {
        "tagStatus": "tagged",
        "tagPrefixList": ["prod-"],
        "countType": "imageCountMoreThan",
        "countNumber": 20
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

Validate lifecycle policies carefully before applying them.

---

 # 31 ECR Retention Trade-Off

Too much retention:

```text
higher storage cost
```

Too little retention:

```text
rollback risk
```

Production retention should be based on:

```text
release frequency
rollback window
compliance
incident history
```

---

 # 32 ALB Cost

Application Load Balancers generate cost based on:

- load balancer usage
- capacity units
- traffic-related usage

Avoid unnecessary duplicate load balancers where architecture permits.

However, consolidating ALBs must not create:

- security boundary problems
- routing complexity
- noisy neighbors
- failure-domain problems

---

 # 33 ALB Consolidation

Instead of:

```text
ALB-1 → service A
ALB-2 → service B
ALB-3 → service C
```

an architecture may use:

```text
ALB
 |
 +--> /service-a
 +--> /service-b
 +--> /service-c
```

with appropriate Ingress rules.

But separate ALBs may still be justified for:

- isolation
- different security policies
- different certificates
- different traffic patterns
- separate ownership

---

 # 34 Data Transfer Cost

AWS data transfer can become expensive.

Common causes:

```text
cross-AZ traffic
cross-region traffic
internet egress
large log transfers
large image transfers
```

Review traffic paths before changing architecture.

---

 # 35 Cross-AZ Traffic

Example:

```text
Pod in AZ-A
   |
   v
Database in AZ-B
```

Repeated high-volume traffic may generate additional network cost and latency.

High availability still requires multi-AZ architecture.

The objective is not:

```text
eliminate all cross-AZ traffic
```

but:

```text
avoid unnecessary cross-AZ traffic while preserving HA
```

---

 # 36 Kubernetes Service Types and Cost

LoadBalancer services can create cloud load balancers.

For internal communication:

```text
ClusterIP
```

is often sufficient.

Do not expose every service as:

```yaml
type: LoadBalancer
```

For this architecture:

```text
Internet
   |
   v
ALB Ingress
   |
   v
ClusterIP services
```

is generally more appropriate.

---

 # 37 ELK Cost

ELK can become one of the largest observability costs.

Storage increases with:

```text
log volume × retention period
```

If applications generate:

```text
100 GB/day
```

and retain:

```text
30 days
```

raw storage can quickly become several terabytes before replication, indexing and overhead.

---

 # 38 Log Volume Reduction

Do not simply collect everything forever.

Reduce unnecessary logs:

- DEBUG in production
- repeated health checks
- duplicate messages
- verbose framework logs
- sensitive data
- redundant application events

Use appropriate production log levels:

```text
INFO
WARN
ERROR
```

with controlled debug logging during incidents.

---

 # 39 ELK Retention

Example policy:

```text
Hot:
7 days

Warm:
23 days

Cold/archive:
longer retention where required
```

Exact values depend on:

- compliance
- troubleshooting requirements
- business needs
- storage architecture

---

 # 40 Log Sampling

For extremely high-volume workloads, sampling may reduce cost.

However:

```text
sampling too aggressively
=
missing important evidence
```

Never sample away:

- security events
- critical errors
- audit events
- transaction failures

---

 # 41 Prometheus Cost

Prometheus cost depends on:

```text
number of series
scrape interval
retention
storage
replication
query workload
```

High-cardinality labels are a major risk.

Bad label:

```text
user_id
```

on a high-volume metric.

Potential result:

```text
millions of time series
```

---

 # 42 Prometheus Cardinality

Example:

```prometheus
http_requests_total{
  service="payment",
  endpoint="/payment",
  user_id="123456"
}
```

If millions of users exist:

```text
millions of series
```

Better:

```text
service
method
route
status_class
```

Use bounded dimensions.

---

 # 43 Scrape Interval

If a metric does not require one-second resolution:

```text
scrape every 30s
```

may be sufficient.

Reducing unnecessary scrape frequency can reduce:

```text
samples
storage
CPU
network traffic
```

Critical infrastructure may require shorter intervals.

---

 # 44 Recording Rules

Recording rules can reduce expensive repeated PromQL calculations.

Example:

```yaml
groups:
  - name: roboshop-recording
    interval: 30s
    rules:
      - record: roboshop:payment_http_requests:rate5m
        expr: |
          sum by (status) (
            rate(http_requests_total{
              service="payment"
            }[5m])
          )
```

Grafana can query the precomputed series.

---

 # 45 Grafana Cost

Grafana itself may have low compute requirements compared with data storage.

The larger issue is:

```text
Grafana queries → Prometheus
```

Poor dashboards can generate expensive queries.

Avoid:

```text
huge time ranges
high-cardinality groupings
hundreds of panels
```

on every refresh.

---

 # 46 Dashboard Optimization

Good dashboard:

```text
small number of meaningful panels
```

Examples:

```text
availability
error rate
latency
traffic
CPU
memory
capacity
deployment health
```

Use variables carefully.

---

 # 47 Observability Architecture Cost

For the capstone:

```text
                    Applications
                         |
            +------------+------------+
            |                         |
          Metrics                    Logs
            |                         |
            v                         v
       Prometheus                    ELK
            |                         |
            v                         v
         Grafana                  Dashboards
            |
            v
       Alertmanager
```

Cost optimization should preserve this observability chain.

Do not remove monitoring blindly to save money.

---

 # 48 Prometheus Retention

Long retention increases storage requirements.

Use a retention period based on:

```text
incident investigation
capacity planning
SLO analysis
business requirements
```

For longer historical data, consider an appropriate long-term metrics architecture if required.

---

 # 49 Kubernetes EmptyDir and Storage

Temporary data should not automatically use expensive persistent storage.

For ephemeral workloads:

```yaml
volumes:
  - name: tmp
    emptyDir: {}
```

For durable data:

```text
persistent volume
```

Use the correct storage class based on requirements.

---

 # 50 EBS Cost

Review:

- unused volumes
- oversized volumes
- unattached volumes
- snapshots
- provisioned performance
- retention

Find unattached resources through AWS inventory and automation.

---

 # 51 Snapshot Cost

Snapshots are useful for recovery but can accumulate.

Define:

```text
daily
weekly
monthly
```

retention based on business requirements.

Do not delete backups blindly for cost reduction.

---

 # 52 Backup vs Cost

A backup is valuable only if:

```text
it can be restored
```

Therefore cost optimization must include:

```text
backup storage
+
restore testing
+
operational recovery
```

A cheap backup strategy that cannot restore is false economy.

---

 # 53 Database Cost Optimization

Database optimization may include:

- right-sizing
- storage optimization
- read replicas only where justified
- reserved capacity where appropriate
- lifecycle management
- connection pooling
- query optimization
- caching

Do not reduce database capacity below safe production requirements.

---

 # 54 Connection Pooling

Without pooling:

```text
application
  |
  +--> new DB connection
  +--> new DB connection
  +--> new DB connection
```

This creates overhead.

With pooling:

```text
application
     |
     v
connection pool
     |
     v
database
```

Proper pool sizing can reduce:

```text
connection overhead
database resource consumption
latency
```

---

 # 55 Application Cost Optimization

Optimize:

```text
CPU
memory
network
storage
database calls
external API calls
logging
```

Efficient code can reduce infrastructure requirements.

Examples:

```text
cache repeated reads
batch operations
reduce unnecessary polling
compress appropriate payloads
```

---

 # 56 CI/CD Cost Optimization

CI can consume significant compute.

Optimize:

- cache dependencies
- avoid duplicate builds
- parallelize where useful
- clean unused runners
- use ephemeral runners
- use appropriate runner sizes
- avoid unnecessary pipeline triggers

---

 # 57 Maven Cache

For Java builds, dependency downloads can be expensive.

Use caching for:

```text
~/.m2/repository
```

where the CI platform safely supports it.

This can reduce:

```text
build time
network traffic
runner consumption
```

---

 # 58 Docker Build Optimization

Use:

```dockerfile
COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src
RUN mvn package
```

This improves layer reuse when dependencies have not changed.

Multi-stage builds can reduce final image size.

---

 # 59 Container Image Size

Large image:

```text
1.5 GB
```

Smaller image:

```text
200 MB
```

Benefits can include:

- faster pulls
- lower registry storage
- lower network usage
- faster deployment

Do not optimize image size at the expense of security or operational clarity.

---

 # 60 Multi-Stage Docker Builds

Example:

```dockerfile
FROM maven:3.9-eclipse-temurin-21 AS build

WORKDIR /app
COPY pom.xml .
COPY src ./src

RUN mvn clean package -DskipTests

FROM eclipse-temurin:21-jre

WORKDIR /app
COPY --from=build /app/target/app.jar app.jar

USER 10001

ENTRYPOINT ["java", "-jar", "app.jar"]
```

The runtime image does not need the Maven build environment.

---

 # 61 Terraform Cost Governance

Terraform can enforce standards.

Examples:

```text
mandatory tags
approved instance types
approved regions
encryption
private networking
```

Policy-as-code can prevent expensive or unsafe resources from being created.

---

 # 62 Terraform Plan Cost Review

Before applying:

```bash
terraform plan
```

Review:

```text
new resources
resource replacements
instance types
storage
load balancers
NAT gateways
```

For large organizations, integrate a cost-estimation tool into CI where appropriate.

---

 # 63 Preventing Accidental Expensive Resources

Guard against:

```text
large EC2 instances
large EBS volumes
many NAT gateways
unexpected load balancers
high-end database instances
unbounded autoscaling
```

Use:

- Terraform modules
- policy checks
- CI validation
- approvals
- budgets
- alerts

---

 # 64 AWS Budgets

Create budgets by:

```text
account
environment
service
team
cost center
```

Alerts can warn when actual or forecast spend crosses thresholds.

Example:

```text
80% → warning
90% → urgent
100% → critical
```

Thresholds should be aligned with business expectations.

---

 # 65 Cost Anomaly Detection

Unexpected cost growth may indicate:

```text
resource leak
autoscaling bug
data-transfer spike
logging storm
crypto-mining/security compromise
misconfiguration
```

Cost monitoring can therefore become a security and reliability signal.

---

 # 66 Production Cost Alerting

Useful alerts:

```text
Unexpected AWS spend
NAT data processing spike
EKS node count anomaly
EBS growth anomaly
ELK ingestion spike
Prometheus cardinality spike
```

These can complement standard production alerts.

---

 # 67 Log Storm Cost Incident

Scenario:

```text
Application enters exception loop.
```

It generates:

```text
50 GB/hour
```

Impact:

```text
ELK storage ↑
network traffic ↑
CPU ↑
query performance ↓
AWS cost ↑
```

Response:

```text
fix application
reduce log verbosity
protect logging pipeline
preserve critical evidence
```

Cost optimization and incident response intersect here.

---

 # 68 Kubernetes Cost Incident

Scenario:

```text
HPA maxReplicas = 200
```

A traffic bug causes scaling:

```text
5 → 20 → 80 → 200
```

Cluster autoscaling adds nodes.

Cost explodes.

Response:

```text
1. Detect scaling anomaly.
2. Confirm legitimate traffic.
3. Check application behavior.
4. Cap unsafe scaling if required.
5. Restore stable workload.
6. Fix root cause.
7. Review autoscaling limits.
```

Never blindly reduce capacity if legitimate customer traffic is causing the increase.

---

 # 69 Cost Optimization and SLOs

Cost must be evaluated against SLOs.

Example:

```text
SLO:
99.9% availability
```

If reducing replicas causes:

```text
99.9% → 99.5%
```

the cost saving may not be acceptable.

Use:

```text
cost
+
reliability
+
performance
+
security
```

as the optimization objective.

---

 # 70 Unit Economics

Useful production metrics include:

```text
cost per request
cost per transaction
cost per customer
cost per deployment
cost per GB log
cost per million API calls
```

For RoboShop:

```text
AWS platform cost
------------------
successful checkout transactions
```

This can be more useful than total monthly spend alone.

---

 # 71 Cost per Environment

Track:

```text
PROD
QA
DEV
```

Example:

```text
PROD: 70%
QA:   20%
DEV:  10%
```

If QA consumes 50% of cost, investigate.

---

 # 72 Cost per Team

Using tags:

```text
team=payments
team=platform
team=frontend
```

can support chargeback or showback.

The goal is not to punish teams.

The goal is:

```text
visibility
accountability
better engineering decisions
```

---

 # 73 Cost Review Cadence

A practical model:

```text
Daily:
critical anomalies

Weekly:
resource waste

Monthly:
service/team cost review

Quarterly:
architecture optimization
```

Large environments may use automated dashboards continuously.

---

 # 74 Production Cost Dashboard

A useful dashboard includes:

```text
Total spend
Spend by account
Spend by environment
Spend by service
EKS compute
NAT
ALB
EBS
ECR
Observability
Data transfer
Cost anomalies
Forecast
```

---

 # 75 Cost Optimization Workflow for RoboShop

```text
RoboShop
   |
   +--> EKS compute
   |      |
   |      +--> right-size pods
   |      +--> HPA
   |      +--> node autoscaling
   |
   +--> Networking
   |      |
   |      +--> NAT review
   |      +--> VPC endpoints
   |      +--> cross-AZ traffic
   |
   +--> ECR
   |      |
   |      +--> image retention
   |
   +--> Observability
   |      |
   |      +--> log retention
   |      +--> metric cardinality
   |      +--> scrape optimization
   |
   +--> CI/CD
   |      |
   |      +--> caching
   |      +--> ephemeral runners
   |
   +--> Environments
          |
          +--> schedule DEV/QA
```

---

 # 76 Production Cost Optimization Checklist

## Compute

- [ ] Node utilization reviewed
- [ ] Pod requests reviewed
- [ ] HPA configured correctly
- [ ] Cluster autoscaling configured
- [ ] Spot evaluated
- [ ] Idle nodes investigated

## Storage

- [ ] Unused EBS volumes removed
- [ ] Snapshot retention reviewed
- [ ] ECR lifecycle configured
- [ ] ELK retention reviewed
- [ ] Prometheus retention reviewed

## Networking

- [ ] NAT traffic reviewed
- [ ] VPC endpoints evaluated
- [ ] Cross-AZ traffic reviewed
- [ ] Data transfer reviewed

## Observability

- [ ] Prometheus cardinality reviewed
- [ ] Scrape intervals reviewed
- [ ] Log levels reviewed
- [ ] Log retention reviewed
- [ ] Dashboard queries optimized

## CI/CD

- [ ] Build caching enabled
- [ ] Runner sizing reviewed
- [ ] Unused runners removed
- [ ] Duplicate builds eliminated

## Governance

- [ ] Tags applied
- [ ] Budgets configured
- [ ] Cost anomaly detection enabled
- [ ] Ownership defined
- [ ] Monthly review established

---

 # 77 Cost Optimization Anti-Patterns

## Anti-Pattern 1: Delete Monitoring

Bad:

```text
Monitoring costs money
→ delete monitoring
```

Result:

```text
blind production
```

Better:

```text
optimize retention/cardinality/storage
```

---

## Anti-Pattern 2: Reduce Replicas Blindly

Bad:

```text
6 replicas → 1
```

Result:

```text
single point of failure
```

---

## Anti-Pattern 3: Use Cheapest Instance Everywhere

Different workloads have different requirements.

Optimize:

```text
price/performance
```

not:

```text
lowest hourly price
```

---

## Anti-Pattern 4: Remove NAT Without Architecture Review

This can break:

```text
private workloads
package downloads
external APIs
AWS service access
```

---

## Anti-Pattern 5: Delete Backups

Backup storage costs money.

But deleting required backups increases:

```text
RPO risk
data-loss risk
business risk
```

---

 # 78 Production Cost Optimization Architecture

```text
                  AWS
                   |
        +----------+----------+
        |          |          |
       EKS        ECR        ALB
        |                     |
        v                     v
    Kubernetes             Traffic
        |
   +----+----+
   |         |
Metrics     Logs
   |         |
   v         v
Prometheus   ELK
   |
   v
Grafana

Cost visibility:
        |
        v
AWS billing / budgets
        |
        +--> Account
        +--> Environment
        +--> Team
        +--> Service
        +--> Cost center
```

---

 # 79 Cost Optimization Decision Example

Suppose monthly spend rises 30%.

Investigation:

```text
EKS compute       +10%
NAT               +5%
ELK                +12%
ECR                +1%
ALB                +2%
```

Total increase:

```text
30%
```

Root cause analysis:

```text
ELK log volume increased 3x.
```

Why?

```text
Application logging level changed from INFO to DEBUG.
```

Fix:

```text
restore production log level
```

Result:

```text
ELK ingestion falls
storage growth falls
query performance improves
```

This is better than cutting EKS replicas.

---

 # 80 Senior-Level Cost Optimization Approach

A senior DevOps engineer should say:

```text
First I establish cost visibility and ownership.

Then I identify the largest cost drivers.

I distinguish required production capacity from waste.

I use utilization metrics to right-size workloads.

For EKS, I evaluate pod requests, HPA, node autoscaling,
instance types, Spot opportunities and non-production schedules.

For networking, I investigate NAT, cross-AZ traffic and data transfer.

For observability, I review log volume, retention, metric cardinality,
scrape frequency and storage.

For CI/CD, I optimize runners, caching and duplicate work.

Every optimization is validated against availability, performance,
security and SLO requirements.
```

---

 # 81 Interview Questions

## Q1. How do you optimize EKS cost?

I start with utilization and workload requests. I right-size pods and nodes, use HPA and node autoscaling, evaluate Spot for fault-tolerant workloads, schedule non-production environments, and review networking and observability costs. I avoid reducing production capacity blindly because availability requirements come first.

## Q2. What is the biggest Kubernetes cost mistake?

Over-provisioning requests and nodes without measuring actual usage is a common mistake. Another is running non-production infrastructure 24x7 without need.

## Q3. How do Kubernetes requests affect cost?

Requests determine scheduling requirements. Excessive requests can force additional nodes even when actual CPU and memory usage is low, increasing infrastructure cost.

## Q4. How can Prometheus become expensive?

High cardinality, very short scrape intervals, excessive retention, replication and expensive queries can increase compute and storage requirements.

## Q5. How do you control ELK cost?

I control log volume, avoid unnecessary DEBUG logging, remove duplicate events, use appropriate retention tiers, review index design and monitor ingestion growth.

## Q6. How do you optimize NAT Gateway cost?

I inspect NAT traffic and data processing, evaluate VPC endpoints for applicable AWS services, reduce unnecessary cross-AZ traffic and review whether workloads are using the correct network path.

## Q7. Would you use Spot in production?

Yes, for workloads that tolerate interruption and have sufficient redundancy. I would not blindly move critical stateful workloads to Spot.

## Q8. How do you optimize ECR?

Use immutable image tags, lifecycle policies and controlled retention. I retain enough versions for rollback and incident recovery.

## Q9. How do you optimize CI/CD cost?

Use dependency caching, Docker layer caching, appropriate runner sizes, ephemeral runners where suitable, avoid duplicate pipelines and clean unused runners.

## Q10. How do you balance cost and availability?

I treat SLOs and reliability requirements as constraints. I optimize waste and inefficiency first rather than removing required redundancy.

## Q11. What is unit cost?

Unit cost expresses infrastructure cost relative to business output, such as cost per transaction or cost per million requests. It is often more actionable than total spend.

## Q12. How can cost monitoring help security?

Unexpected cost spikes can indicate compromised resources, cryptocurrency mining, traffic abuse or runaway workloads. Therefore cost anomalies can become an additional security signal.

---

 # 82 Practical Production Cost Commands

AWS inventory:

```bash
aws resourcegroupstaggingapi get-resources
```

EKS:

```bash
aws eks list-clusters
aws eks describe-cluster --name <cluster-name>
```

Nodes:

```bash
kubectl top nodes
```

Pods:

```bash
kubectl top pods -A
```

Deployments:

```bash
kubectl get deployments -A
```

HPA:

```bash
kubectl get hpa -A
```

PVCs:

```bash
kubectl get pvc -A
```

ECR images:

```bash
aws ecr describe-images \
  --repository-name <repository>
```

EBS volumes:

```bash
aws ec2 describe-volumes \
  --filters Name=status,Values=available
```

These commands are starting points. Production cost reporting should generally use AWS billing, inventory and monitoring systems rather than manual command execution alone.

---

 # 83 Cost Optimization Incident Runbook

### Symptom

```text
AWS bill increased unexpectedly.
```

### Step 1

Identify:

```text
account
region
service
environment
```

### Step 2

Compare:

```text
current period
previous period
```

### Step 3

Identify top drivers:

```text
EKS
EC2
NAT
ELB
storage
observability
data transfer
```

### Step 4

Investigate workload changes:

```text
deployments
scaling
traffic
logging
infrastructure changes
```

### Step 5

Contain runaway behavior.

### Step 6

Implement safe optimization.

### Step 7

Monitor recovery.

### Step 8

Add prevention:

```text
budget
alert
policy
automation
```

---

 # 84 Cost Optimization and GitOps

Configuration affecting cost should be managed through Git where practical.

Examples:

```yaml
replicas: 3

resources:
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  minReplicas: 3
  maxReplicas: 12
```

Changes are:

```text
Git
 ↓
PR
 ↓
review
 ↓
Argo CD
 ↓
EKS
```

This creates an audit trail for capacity changes.

---

 # 85 Cost Optimization and Terraform

Infrastructure cost changes should similarly be controlled through Terraform.

Example:

```hcl
node_group = {
  min_size     = 3
  desired_size = 3
  max_size     = 8
}
```

A proposed change:

```text
max_size = 8 → 20
```

should be reviewed for:

```text
capacity
cost
scaling behavior
business demand
```

---

 # 86 Cost Optimization Governance

A mature organization defines:

```text
who owns costs
who approves infrastructure
who reviews anomalies
who owns budgets
who can create expensive resources
```

Production DevOps should treat cost as an engineering metric.

---

 # 87 Final Cost Optimization Model

```text
                 COST VISIBILITY
                       |
                       v
                 MEASURE USAGE
                       |
                       v
                 FIND WASTE
                       |
          +------------+------------+
          |                         |
       Compute                   Storage
          |                         |
       EKS/EC2                 EBS/ECR/ELK
          |
          +------------+
          |
       Networking
          |
      NAT / AZ / Egress
          |
          v
     OBSERVABILITY
          |
 Prometheus / ELK
          |
          v
       CI/CD
          |
          v
      GOVERNANCE
          |
          v
     OPTIMIZATION
          |
          v
   VALIDATE SLO / SECURITY
          |
          v
       CONTINUOUS REVIEW
```

The production cost mindset is:

```text
Do not ask:
"How can I make AWS cheaper?"

Ask:
"Where are we paying for capacity, traffic, storage,
or operations that we do not need, while still meeting
our reliability, security and performance objectives?"
```

That is the difference between simple cost cutting and production-grade cloud cost optimization.
