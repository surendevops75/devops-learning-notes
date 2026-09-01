# 39 - Architecture Review

## Production DevOps / DevSecOps Capstone

---

# 1. Purpose of This Architecture Review

This chapter performs a senior-level review of the complete production DevOps/DevSecOps architecture developed throughout the capstone.

The review is not only a description of components. It evaluates:

- architecture decisions
- reliability
- availability
- scalability
- security
- observability
- deployment safety
- disaster recovery
- operational maturity
- cost
- maintainability
- failure domains
- bottlenecks
- trade-offs
- production risks
- improvement opportunities
- interview-level design reasoning

The target platform is a production-style RoboShop microservices environment running on AWS EKS.

The primary technology stack is:

- AWS
- VPC
- IAM
- Security Groups
- NACLs
- EKS
- EC2 worker nodes
- ECR
- ALB
- Kubernetes
- Helm
- Terraform
- Git
- GitHub/GitLab
- CI pipelines
- SonarQube
- Trivy
- Veracode
- GitOps
- Argo CD
- Prometheus
- Grafana
- ELK
- Linux
- Java
- Node.js
- Python
- Bash
- Maven

The architecture intentionally uses:

- AWS ALB Ingress for external HTTP/HTTPS traffic
- Prometheus for metrics and alert evaluation
- Grafana for visualization
- ELK for centralized logs
- Argo CD as the GitOps reconciliation engine
- Terraform for infrastructure provisioning
- Helm for Kubernetes application packaging

Jaeger and OpenTelemetry are intentionally outside this architecture.

---

# 2. Executive Architecture Summary

The platform follows a layered production architecture.

```text
                         INTERNET / USERS
                               |
                               v
                        Route 53 / DNS
                               |
                               v
                       AWS ALB / HTTPS
                               |
                               v
                     Kubernetes Ingress
                               |
              +----------------+----------------+
              |                                 |
              v                                 v
       RoboShop Services                  Other APIs
              |
              v
        EKS Kubernetes
              |
    +---------+----------+
    |         |          |
    v         v          v
  Pods     Services    Config
    |
    +-----------------------------+
    |                             |
    v                             v
Prometheus                       ELK
    |                             |
    v                             v
Alertmanager                 Elasticsearch
    |                             |
    v                             v
On-call / Teams                  Kibana
    |
    v
Operations

CI/CD:

Developer
   |
   v
Git
   |
   v
CI
   |
   +--> Build
   +--> Test
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   +--> Container Build
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

Infrastructure:

Terraform
   |
   +--> VPC
   +--> IAM
   +--> EKS
   +--> Node Groups
   +--> ECR
   +--> Supporting AWS resources
```

The design separates:

1. infrastructure provisioning
2. application build
3. artifact storage
4. deployment desired state
5. deployment reconciliation
6. observability
7. incident response

This separation reduces operational coupling and improves auditability.

---

# 3. Architecture Principles

The architecture is based on the following principles.

## 3.1 Infrastructure as Code

Infrastructure must be reproducible.

Terraform should define:

- VPC
- subnets
- routing
- EKS
- node groups
- IAM
- ECR
- supporting infrastructure

Manual production infrastructure changes should be minimized.

---

## 3.2 Immutable Artifacts

Production should deploy immutable container images.

Prefer:

```text
roboshop/frontend:git-<commit-sha>
```

or, even better, deploy using an immutable digest:

```text
image@sha256:<digest>
```

Avoid:

```text
latest
```

because `latest` does not identify an immutable release.

---

# 4. Separation of Responsibilities

A mature platform separates responsibilities.

| Layer | Responsibility |
|---|---|
| Developer | Application code |
| CI | Build, test, security validation |
| ECR | Artifact storage |
| GitOps repo | Deployment desired state |
| Argo CD | Kubernetes reconciliation |
| EKS | Runtime |
| Prometheus | Metrics |
| Grafana | Visualization |
| ELK | Logs |
| Alertmanager | Alert routing |
| Terraform | Infrastructure |
| Operations | Incident response |

This separation prevents one system from becoming responsible for the entire lifecycle.

---

# 5. AWS Architecture Review

## 5.1 VPC

The VPC should be designed across multiple Availability Zones.

Example:

```text
VPC
|
+-- AZ-a
|   +-- Public subnet
|   +-- Private application subnet
|   +-- Private data subnet
|
+-- AZ-b
|   +-- Public subnet
|   +-- Private application subnet
|   +-- Private data subnet
|
+-- AZ-c
    +-- Public subnet
    +-- Private application subnet
    +-- Private data subnet
```

The primary EKS worker nodes should run in private subnets.

The ALB can use public subnets when the application is internet-facing.

---

# 6. Multi-AZ Review

Multi-AZ is mandatory for production availability.

A single-AZ design creates a major failure domain.

Example failure:

```text
AZ-a outage
     |
     v
All nodes unavailable
     |
     v
Application outage
```

Multi-AZ:

```text
AZ-a          AZ-b          AZ-c
Nodes         Nodes         Nodes
Pods          Pods          Pods
  \             |             /
   +------------+------------+
                |
               ALB
```

If one AZ fails, Kubernetes can continue serving traffic from healthy AZs.

---

# 7. ALB Architecture Review

AWS ALB is appropriate for HTTP/HTTPS application traffic.

Flow:

```text
User
 |
 v
Route 53
 |
 v
ALB
 |
 v
Kubernetes Ingress
 |
 v
Service
 |
 v
Pod
```

Advantages:

- managed AWS service
- Layer 7 routing
- TLS termination
- host-based routing
- path-based routing
- health checks
- integration with EKS
- high availability

---

# 8. Why ALB Instead of API Gateway

The architecture explicitly chooses ALB.

API Gateway is useful for:

- API management
- throttling
- serverless APIs
- API keys
- advanced API lifecycle controls

But this capstone's primary requirement is Kubernetes application ingress.

ALB provides a direct and operationally appropriate path:

```text
Internet
   |
  ALB
   |
Ingress
   |
Kubernetes Service
```

Introducing API Gateway without a requirement would add another infrastructure layer and operational surface.

---

# 9. EKS Architecture Review

EKS is appropriate because the platform contains:

- multiple microservices
- independent deployments
- autoscaling requirements
- containerized workloads
- service discovery
- rolling deployments
- GitOps requirements

EKS reduces the control-plane management burden compared with self-managed Kubernetes.

---

# 10. EKS Control Plane vs Data Plane

The architecture must distinguish:

```text
EKS Control Plane
        |
        v
API Server
Scheduler
Controllers
etcd-managed control plane
        |
        v
Worker Data Plane
        |
        +--> EC2 Nodes
        |
        +--> Pods
```

EKS manages the Kubernetes control plane.

The organization remains responsible for:

- workload configuration
- node capacity
- pod configuration
- security
- IAM integration
- networking
- observability
- application reliability

---

# 11. Worker Node Strategy

Production nodes should be distributed across multiple AZs.

Example:

```text
Node Group
|
+-- AZ-a
|   +-- node-a1
|   +-- node-a2
|
+-- AZ-b
|   +-- node-b1
|   +-- node-b2
|
+-- AZ-c
    +-- node-c1
    +-- node-c2
```

Use:

- multiple nodes
- autoscaling
- pod anti-affinity where appropriate
- topology spread constraints
- resource requests and limits

---

# 12. Pod Scheduling Review

A production application should not unintentionally place all replicas on one node.

Bad:

```text
Node-1
 |
 +-- frontend-1
 +-- frontend-2
 +-- frontend-3
```

If Node-1 fails, all replicas disappear.

Better:

```text
Node-1 -> frontend-1
Node-2 -> frontend-2
Node-3 -> frontend-3
```

Use:

- pod anti-affinity
- topology spread constraints
- node labels
- taints/tolerations where justified

---

# 13. Requests and Limits

Every production workload should define resource requests.

Example:

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "256Mi"
  limits:
    cpu: "1000m"
    memory: "512Mi"
```

Requests influence scheduling.

Limits constrain maximum resource consumption.

Incorrect limits can cause:

- OOMKilled
- CPU throttling
- poor scheduling
- wasted capacity

---

# 14. Horizontal Pod Autoscaling

HPA is useful for application-level scaling.

Example:

```text
Traffic increases
      |
      v
CPU / memory / custom metric increases
      |
      v
HPA evaluates target
      |
      v
Replica count increases
      |
      v
Kubernetes schedules additional pods
```

HPA does not create infrastructure nodes by itself.

If no node capacity exists:

```text
HPA
 |
 v
More pods requested
 |
 v
No node capacity
 |
 v
Pending pods
```

Cluster/node autoscaling must address infrastructure capacity.

---

# 15. Autoscaling Review

Production scaling has multiple layers.

```text
User traffic
    |
    v
ALB
    |
    v
HPA
    |
    v
Pod count
    |
    v
Cluster/node capacity
    |
    v
Node autoscaling
```

These layers must be configured together.

A common architecture mistake is configuring HPA without enough node capacity.

---

# 16. ECR Review

ECR is the correct artifact registry for AWS-hosted EKS.

Recommended lifecycle:

```text
CI
 |
 v
Build image
 |
 v
Scan image
 |
 v
Push to ECR
 |
 v
Immutable tag/digest
 |
 v
GitOps deployment
```

Enable:

- image scanning
- lifecycle policies
- encryption
- repository policies
- least-privilege IAM

---

# 17. Container Image Security

The image pipeline should include:

1. minimal base image
2. dependency update
3. vulnerability scanning
4. non-root execution where possible
5. no embedded credentials
6. immutable release identification

Example:

```dockerfile
USER 10001
```

rather than running unnecessarily as root.

---

# 18. Terraform Architecture Review

Terraform should be the source of truth for AWS infrastructure.

Example:

```text
terraform/
|
+-- modules/
|   +-- vpc/
|   +-- eks/
|   +-- iam/
|   +-- ecr/
|
+-- environments/
    +-- dev/
    +-- qa/
    +-- prod/
```

Use remote state with locking.

Production should not rely on a local laptop state file.

---

# 19. Terraform State Security

Terraform state may contain sensitive infrastructure information.

Protect it using:

- encrypted backend storage
- access controls
- state locking
- restricted IAM
- versioning
- audit logging

Never commit:

```text
terraform.tfstate
```

to source control.

---

# 20. Terraform Production Workflow

Recommended:

```text
Pull Request
    |
    v
terraform fmt
    |
    v
terraform validate
    |
    v
terraform plan
    |
    v
Review
    |
    v
Approval
    |
    v
terraform apply
```

Avoid uncontrolled:

```text
terraform apply -auto-approve
```

against production unless it is part of a controlled automation process.

---

# 21. GitOps Architecture Review

GitOps is one of the strongest architectural decisions in this capstone.

The desired state lives in Git.

Example:

```text
GitOps Repository
       |
       v
Deployment YAML / Helm values
       |
       v
Argo CD
       |
       v
EKS
```

Argo CD continuously compares:

```text
Git desired state
        vs
Kubernetes actual state
```

---

# 22. Why GitOps Is Valuable

GitOps provides:

- auditability
- reviewability
- version history
- repeatability
- drift detection
- controlled deployment
- rollback through Git
- separation between CI and CD

A production deployment can answer:

- Who changed it?
- What changed?
- When?
- Which commit caused it?
- Which image was deployed?
- What was the previous version?

---

# 23. CI vs CD Responsibility

CI should not directly own the production runtime.

Preferred:

```text
CI
 |
 +-- Build
 +-- Test
 +-- Scan
 +-- Push image
 |
 v
Update GitOps
 |
 v
Argo CD
 |
 v
EKS
```

This is cleaner than:

```text
CI
 |
 +-- docker build
 +-- kubectl apply
 +-- kubectl rollout
```

for a GitOps production model.

---

# 24. Argo CD Architecture Review

Argo CD acts as the deployment reconciliation engine.

```text
Git
 |
 v
Argo CD
 |
 +--> Cluster-1
 +--> Cluster-2
 +--> Cluster-3
```

This also supports centralized multi-cluster management.

---

# 25. Multi-Cluster Review

A centralized Argo CD control plane can manage:

```text
Argo CD
 |
 +-- Dev EKS
 |
 +-- QA EKS
 |
 +-- Production EKS
 |
 +-- DR EKS
```

Benefits:

- centralized visibility
- common GitOps model
- consistent deployment
- environment separation

Risk:

Argo CD itself becomes a management-plane dependency.

Therefore:

- secure Argo CD
- back up configuration
- restrict cluster credentials
- deploy HA components
- test recovery

---

# 26. Environment Separation

Recommended model:

```text
dev
 |
 +-- lower cost
 +-- faster changes

qa
 |
 +-- integration validation

prod
 |
 +-- controlled changes
 +-- stricter approvals
 +-- high availability
```

Do not copy production credentials into lower environments.

---

# 27. Secrets Architecture Review

Secrets must not be stored in plain Git.

Bad:

```yaml
password: MyProductionPassword123
```

Better approaches include:

- AWS Secrets Manager
- encrypted secret delivery
- External Secrets patterns
- tightly controlled Kubernetes Secrets

The important principle is:

```text
Git contains configuration references,
not production credentials.
```

---

# 28. IAM Review

IAM should follow least privilege.

Separate:

- CI permissions
- Terraform permissions
- Argo CD permissions
- EKS workload permissions
- operator permissions

Avoid:

```text
AdministratorAccess
```

for normal workload identities.

---

# 29. Kubernetes RBAC Review

Use namespace and role boundaries.

Example:

```text
platform-admin
application-team
read-only
monitoring
deployment-controller
```

Do not give every engineer:

```text
cluster-admin
```

---

# 30. Network Security Review

Use layered controls.

```text
Internet
   |
   v
ALB
   |
Security Groups
   |
   v
Ingress
   |
Network Policies
   |
Service
   |
Pod
```

AWS controls and Kubernetes controls complement each other.

---

# 31. Security Group Review

Security groups should permit only required traffic.

Example:

```text
Internet -> ALB : 443
ALB -> application nodes/pods : application ports
Nodes -> required AWS services : required egress
```

Avoid:

```text
0.0.0.0/0
```

on unnecessary ports.

---

# 32. NACL Review

NACLs operate at subnet level.

They are stateless.

Security groups are stateful.

Do not depend on NACLs as the only security mechanism.

A layered design is better.

---

# 33. Kubernetes NetworkPolicy

NetworkPolicy can restrict pod-to-pod communication.

Example principle:

```text
frontend -> catalog
frontend -> user
frontend -> cart

frontend -X-> unrelated database
```

This reduces lateral movement risk.

---

# 34. Observability Architecture Review

The architecture uses:

```text
Metrics -> Prometheus
Visualization -> Grafana
Logs -> ELK
Alert routing -> Alertmanager
```

This creates clear responsibilities.

---

# 35. Metrics Architecture

```text
Applications
    |
    v
Metrics endpoints
    |
    v
Prometheus
    |
    +--> Alert rules
    |
    +--> Recording rules
    |
    v
Grafana
```

Prometheus is responsible for metric collection and evaluation.

Grafana should not become the metric collection engine.

---

# 36. Logging Architecture

```text
Pods
 |
 v
Container logs
 |
 v
Log collection
 |
 v
Logstash / ingestion
 |
 v
Elasticsearch
 |
 v
Kibana
```

ELK provides centralized operational visibility.

---

# 37. Alerting Architecture

```text
Prometheus
    |
    v
Alert rules
    |
    v
Alertmanager
    |
    +--> Slack-style notification
    +--> Email
    +--> PagerDuty-style escalation
```

Alertmanager should handle:

- grouping
- routing
- silencing
- inhibition
- deduplication

---

# 38. Golden Signals Review

Production alerting should prioritize:

1. latency
2. traffic
3. errors
4. saturation

Example:

```text
Traffic increasing
+
Latency increasing
+
5xx increasing
=
Potential application incident
```

CPU alone is not a complete health indicator.

---

# 39. SLI/SLO Architecture Review

An SLI measures service behavior.

Examples:

```text
Availability SLI
Successful requests / total requests
```

```text
Latency SLI
Requests under threshold / total requests
```

An SLO defines the target.

Example:

```text
99.9% availability
```

Alerting should protect SLOs rather than simply alerting on every resource fluctuation.

---

# 40. Alert Quality

Bad alert:

```text
CPU > 70%
```

for every server.

Better:

```text
CPU > 85%
for 15 minutes
AND
workload is under sustained load
```

Best:

Use a user-impact or saturation-oriented signal where possible.

---

# 41. Production Alert Design

A good alert answers:

- What failed?
- How severe?
- Which environment?
- Which service?
- Which team owns it?
- What should the operator do?
- Where is the runbook?

Example labels:

```yaml
labels:
  severity: critical
  environment: prod
  team: platform
  service: checkout
```

Annotations:

```yaml
annotations:
  summary: Checkout error rate is high
  description: HTTP 5xx rate exceeded the production threshold.
  runbook_url: https://runbooks.example.internal/checkout/errors
```

---

# 42. PrometheusRule Review

Production rules should have:

- meaningful names
- stable labels
- environment
- service
- team
- severity
- actionable annotations
- sensible `for` duration

Avoid alerts that trigger immediately for transient noise unless the failure is truly critical.

---

# 43. Alertmanager Review

Alertmanager should implement:

```text
critical
   |
   +--> on-call escalation

warning
   |
   +--> team notification

info
   |
   +--> dashboard / lower-priority channel
```

Grouping prevents 500 identical alerts from becoming 500 notifications.

---

# 44. Alert Storm Protection

A node failure can cause:

```text
NodeDown
PodDown
DeploymentUnavailable
ServiceUnavailable
ALBTargetUnhealthy
ApplicationErrorRateHigh
```

Without grouping/inhibition, operators receive a flood.

Alertmanager should suppress secondary symptoms when a root cause is known.

---

# 45. Monitoring Trade-off

More alerts do not mean better monitoring.

The objective is:

```text
High signal
Low noise
Fast action
```

An alert that nobody responds to is operationally useless.

---

# 46. High Availability Review

Critical components should avoid single points of failure.

Review:

| Component | HA consideration |
|---|---|
| ALB | AWS managed HA |
| EKS control plane | AWS managed HA |
| Worker nodes | Multi-AZ |
| Pods | Multiple replicas |
| Prometheus | HA/retention strategy |
| Alertmanager | HA |
| Grafana | Multiple replicas if required |
| ELK | Multi-node production design |
| Argo CD | HA for critical environments |
| Git | Remote durable platform |
| Terraform state | Durable remote backend |

---

# 47. Application Availability

Every production deployment should consider:

- replicas
- readiness probe
- liveness probe
- startup probe when required
- PDB
- topology spread
- resource requests
- autoscaling

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  periodSeconds: 10
```

Readiness controls whether traffic should reach the pod.

---

# 48. Liveness vs Readiness

Readiness:

```text
Can this pod receive traffic?
```

Liveness:

```text
Is this process unhealthy enough to restart?
```

A common production mistake is using aggressive liveness checks that restart healthy but slow-starting applications.

---

# 49. Deployment Strategy Review

Rolling updates are the default safe mechanism.

```text
Version 1
replicas = 6

        |
        v

Version 1 = 5
Version 2 = 1

        |
        v

Version 1 = 3
Version 2 = 3

        |
        v

Version 2 = 6
```

Use:

- maxUnavailable
- maxSurge
- readiness checks
- health checks

---

# 50. Rollback Architecture

Rollback should exist at multiple layers.

```text
Application
 |
 +--> Git revert
 |
 +--> GitOps version rollback
 |
 +--> Argo CD sync
 |
 +--> Helm rollback where applicable
 |
 +--> Kubernetes rollout undo
 |
 +--> Image digest rollback
```

Database rollback requires special consideration because schema changes may not be backward compatible.

---

# 51. Database Deployment Review

The application deployment process should not assume database rollback is equivalent to application rollback.

Example:

```text
App v2
requires DB schema v2
```

If the application is rolled back to v1:

```text
DB schema v2
App v1
```

may be incompatible.

Use backward-compatible migration strategies where possible.

---

# 52. Disaster Recovery Review

HA and DR are different.

HA:

```text
Keep service running during expected failures.
```

DR:

```text
Recover service after a major disaster.
```

Examples:

```text
AZ failure -> HA
Region failure -> DR
Accidental deletion -> Backup/Restore
```

---

# 53. RPO and RTO

RPO:

```text
Maximum acceptable data loss.
```

RTO:

```text
Maximum acceptable recovery time.
```

Example:

```text
RPO = 15 minutes
RTO = 60 minutes
```

Architecture decisions should be derived from these requirements.

---

# 54. DR Architecture

A possible production DR model:

```text
Primary Region
   |
   +--> EKS
   +--> ECR replication
   +--> GitOps
   +--> Data services
   |
   v
Replication / Backup
   |
   v
DR Region
   |
   +--> EKS recovery environment
   +--> Required infrastructure
   +--> Data restore
```

Not every workload requires an always-running duplicate region.

---

# 55. Backup Review

Back up:

- databases
- Kubernetes critical configuration where required
- Terraform state
- GitOps repository
- application configuration
- encryption keys where appropriate
- ELK data according to retention requirements

Most importantly:

```text
Backup is not proven until restore is tested.
```

---

# 56. Production Troubleshooting Architecture

Troubleshooting should follow dependency order.

Example:

```text
User
 |
 v
DNS
 |
 v
ALB
 |
 v
Ingress
 |
 v
Service
 |
 v
Pod
 |
 v
Application
 |
 v
Database / dependency
```

Do not immediately restart pods.

First establish the failing layer.

---

# 57. Troubleshooting Method

Use:

```text
Symptom
  |
  v
Scope
  |
  v
Timeline
  |
  v
Metrics
  |
  v
Logs
  |
  v
Events
  |
  v
Configuration
  |
  v
Dependency
  |
  v
Root cause
  |
  v
Fix
  |
  v
Prevention
```

---

# 58. Example: HTTP 502

Possible layers:

```text
User
 |
 v
ALB
 |
 v
Target
 |
 v
Ingress
 |
 v
Service
 |
 v
Pod
```

Investigation:

```bash
kubectl get ingress -A
kubectl describe ingress <name> -n <namespace>
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>
kubectl get pods -n <namespace>
```

Then inspect:

```bash
kubectl logs <pod> -n <namespace>
```

---

# 59. Example: Pod OOMKilled

Check:

```bash
kubectl describe pod <pod> -n <namespace>
kubectl get pod <pod> -n <namespace> -o yaml
```

Look for:

```text
Reason: OOMKilled
Exit Code: 137
```

Then correlate with:

- memory requests
- memory limits
- application heap
- traffic
- historical memory usage
- recent release

---

# 60. Example: Pending Pod

Check:

```bash
kubectl get pods -A
kubectl describe pod <pod> -n <namespace>
```

Common causes:

- insufficient CPU
- insufficient memory
- taints
- node selectors
- affinity constraints
- PVC constraints
- topology constraints

Do not simply increase replica count without checking capacity.

---

# 61. Example: Argo CD OutOfSync

Investigate:

```text
Git desired state
       |
       v
Argo CD comparison
       |
       v
Kubernetes live state
```

Possible causes:

- manual change
- generated field difference
- wrong values
- wrong cluster
- wrong namespace
- failed sync
- ignored fields configuration

The desired state should normally be corrected in Git rather than manually patched.

---

# 62. Security Architecture Review

Security must exist at every layer.

```text
Developer
   |
Git security
   |
CI security
   |
Container security
   |
AWS IAM
   |
Network security
   |
Kubernetes RBAC
   |
Pod security
   |
Runtime monitoring
```

This is defense in depth.

---

# 63. CI Security Review

CI should include:

```text
SAST
Dependency scanning
Container scanning
Secret detection
Quality gates
Artifact validation
```

The capstone uses:

- SonarQube
- Trivy
- Veracode

Each tool should have a defined purpose.

---

# 64. SonarQube Role

SonarQube focuses primarily on code quality and static analysis.

Examples:

- bugs
- code smells
- maintainability
- vulnerabilities
- coverage-related quality gates

It should not be treated as the only security control.

---

# 65. Trivy Role

Trivy can scan:

- container images
- filesystem dependencies
- configuration
- infrastructure code

Use severity thresholds appropriate for the environment.

Do not automatically block every theoretical vulnerability without risk evaluation.

---

# 66. Veracode Role

Veracode provides additional application security analysis.

The architecture should avoid blindly duplicating tools.

The objective is layered coverage rather than tool count.

---

# 67. Cost Architecture Review

Production reliability has a cost.

Major cost categories:

- EKS worker nodes
- ALB
- NAT Gateway
- ECR storage
- Elasticsearch
- log ingestion
- data transfer
- Prometheus storage
- Grafana infrastructure
- CI runners
- backup storage

---

# 68. NAT Gateway Cost

NAT Gateways can become expensive at scale.

Review:

- number of NAT gateways
- cross-AZ traffic
- private subnet egress
- VPC endpoints
- data transfer

For high-volume AWS service access, VPC endpoints may reduce NAT dependency.

---

# 69. Logging Cost

ELK can become one of the largest observability costs.

Reasons:

```text
High log volume
+
Long retention
+
Verbose application logs
=
High storage and ingestion cost
```

Use:

- retention policies
- index lifecycle management
- useful log levels
- structured logging
- sampling where appropriate

---

# 70. Prometheus Cost

Metrics cardinality must be controlled.

Bad:

```text
user_id
request_id
transaction_id
```

as unbounded labels.

This can create enormous time-series counts.

Prefer stable dimensions:

```text
service
method
status
environment
```

---

# 71. Scalability Review

Scalability has multiple dimensions.

### Horizontal

More replicas.

### Vertical

More CPU/memory.

### Infrastructure

More nodes.

### Data

Database scaling.

### Observability

More monitoring capacity.

A scalable application can still fail if its database or logging platform cannot scale.

---

# 72. Bottleneck Analysis

Potential bottlenecks:

```text
ALB
 |
EKS
 |
Service
 |
Pod
 |
Database
 |
ELK
```

The architecture should identify the actual bottleneck rather than scaling every layer.

---

# 73. Dependency Failure Review

Microservices create dependency chains.

Example:

```text
frontend
   |
   v
catalog
   |
   v
database
```

If the database fails:

```text
database failure
     |
     v
catalog errors
     |
     v
frontend errors
```

Observability must help identify the original failure.

---

# 74. Failure Domain Review

Review failures by domain:

```text
Pod
Node
AZ
Region
AWS service
Cluster
Git
CI
Argo CD
Registry
Database
Observability
Identity
Network
```

A mature architecture does not assume all failures are application failures.

---

# 75. Deployment Failure Domain

Deployment systems themselves can fail.

Example:

```text
CI succeeds
ECR succeeds
GitOps update succeeds
Argo CD fails
```

The application may remain on the old version.

This is preferable to uncontrolled partial deployment.

---

# 76. Git Failure Domain

If GitHub/GitLab is unavailable:

Existing workloads should continue running.

Kubernetes does not require Git for already-applied runtime state.

Argo CD cannot reconcile new changes while Git is unavailable, but the current application should continue.

This is an important resilience characteristic.

---

# 77. ECR Failure Domain

If ECR becomes temporarily unavailable:

- running containers continue running
- new pod creation may fail if the required image cannot be pulled
- deployment operations may fail

Mitigations include:

- node image caching
- controlled deployment timing
- registry availability
- image immutability
- regional replication for advanced DR

---

# 78. Observability Failure Domain

If Grafana is down:

Prometheus data may still exist.

If Prometheus is down:

Applications may still run.

If ELK is down:

Application runtime may continue, but troubleshooting becomes harder.

Therefore observability is critical but should not unnecessarily become a runtime dependency.

---

# 79. Production Architecture Trade-offs

Every design has trade-offs.

## EKS

Pros:

- managed control plane
- AWS integration
- Kubernetes ecosystem

Cons:

- operational complexity
- networking complexity
- Kubernetes expertise required

---

# 80. ALB

Pros:

- managed
- AWS-native
- HTTP routing
- TLS integration

Cons:

- AWS-specific
- cost
- controller configuration complexity

---

# 81. Argo CD

Pros:

- GitOps
- drift detection
- auditability
- multi-cluster management

Cons:

- additional platform component
- operational responsibility
- Git dependency for changes

---

# 82. Terraform

Pros:

- reproducibility
- infrastructure version control
- reviewable changes

Cons:

- state management
- provider dependencies
- state drift
- learning curve

---

# 83. Prometheus

Pros:

- Kubernetes-native ecosystem
- PromQL
- flexible alerting
- strong community

Cons:

- cardinality management
- storage considerations
- HA architecture complexity

---

# 84. ELK

Pros:

- powerful log search
- centralized logs
- flexible analysis

Cons:

- operational complexity
- storage cost
- scaling Elasticsearch
- index lifecycle management

---

# 85. Architecture Risks

The senior review identifies these major risks:

1. excessive observability cardinality
2. insufficient pod distribution
3. weak resource requests/limits
4. overly broad IAM
5. secrets in Git
6. single-region dependency
7. untested backups
8. untested DR
9. alert noise
10. insufficient log retention strategy
11. uncontrolled manual changes
12. database migration incompatibility
13. Argo CD management-plane dependency
14. Terraform state exposure
15. uncontrolled production access

---

# 86. Risk Matrix

| Risk | Impact | Likelihood | Priority |
|---|---|---|---|
| Single AZ workload | High | Medium | High |
| Bad resource limits | High | High | High |
| Alert noise | Medium | High | High |
| Unprotected secrets | Critical | Medium | Critical |
| Untested restore | Critical | Medium | Critical |
| Broad IAM | Critical | Medium | Critical |
| High metric cardinality | High | Medium | High |
| ELK cost explosion | High | Medium | High |
| Manual production changes | High | High | High |
| Database rollback incompatibility | Critical | Medium | Critical |

---

# 87. Recommended Improvements

The architecture should continuously improve through:

- stronger SLOs
- better alert design
- automated DR testing
- image digest pinning
- stricter IAM
- Kubernetes network policies
- policy-as-code
- automated backup verification
- capacity planning
- metric cardinality controls
- log retention controls
- production change governance

---

# 88. Production Readiness Checklist

## Infrastructure

- [ ] Multi-AZ VPC
- [ ] Private worker subnets
- [ ] Correct route tables
- [ ] NAT strategy
- [ ] Security groups
- [ ] NACL review
- [ ] Terraform remote state
- [ ] State locking
- [ ] Infrastructure tagging

## EKS

- [ ] Multi-AZ nodes
- [ ] Node autoscaling
- [ ] Pod autoscaling
- [ ] Resource requests
- [ ] Resource limits
- [ ] Probes
- [ ] PDBs
- [ ] topology spread
- [ ] RBAC
- [ ] network policies

---

# 89. Production Readiness - CI/CD

- [ ] Unit tests
- [ ] Integration tests
- [ ] SonarQube
- [ ] Trivy
- [ ] Veracode
- [ ] Immutable images
- [ ] ECR scanning
- [ ] GitOps update
- [ ] Production approval
- [ ] Rollback strategy

---

# 90. Production Readiness - GitOps

- [ ] Git repository protected
- [ ] Pull request review
- [ ] Argo CD authentication secured
- [ ] Cluster credentials protected
- [ ] Drift detection
- [ ] Sync policies reviewed
- [ ] Rollback procedure tested
- [ ] Argo CD backup/recovery plan

---

# 91. Production Readiness - Observability

- [ ] Prometheus
- [ ] Grafana
- [ ] ELK
- [ ] Alertmanager
- [ ] Critical alerts
- [ ] Warning alerts
- [ ] SLO alerts
- [ ] Runbook links
- [ ] On-call ownership
- [ ] Alert escalation
- [ ] Log retention
- [ ] Metric retention

---

# 92. Production Readiness - Security

- [ ] Least privilege IAM
- [ ] RBAC
- [ ] Secrets management
- [ ] TLS
- [ ] Image scanning
- [ ] SAST
- [ ] dependency scanning
- [ ] network policy
- [ ] audit logging
- [ ] production access control

---

# 93. Production Readiness - DR

- [ ] RPO defined
- [ ] RTO defined
- [ ] backups
- [ ] restore tests
- [ ] DR documentation
- [ ] DR drill
- [ ] failover process
- [ ] failback process
- [ ] data validation

---

# 94. Senior-Level Architecture Decision

The architecture is production-capable when the following principles are enforced:

```text
Infrastructure -> Terraform
Application packaging -> Helm
Application delivery -> GitOps
Reconciliation -> Argo CD
Runtime -> EKS
Ingress -> ALB
Registry -> ECR
Metrics -> Prometheus
Dashboards -> Grafana
Logs -> ELK
Alert routing -> Alertmanager
Security -> layered DevSecOps
Recovery -> Backup + DR
```

The important point is not the number of tools.

The important point is that each tool has a clear responsibility.

---

# 95. What Should Not Be Added Without a Requirement

Avoid architecture inflation.

Do not automatically add:

- API Gateway
- service mesh
- Kafka
- OpenTelemetry
- Jaeger
- additional ingress controllers
- unnecessary databases
- multiple monitoring systems
- multiple CI systems

Every component creates:

- cost
- operational overhead
- security surface
- upgrade requirements
- failure modes

Use the simplest architecture that satisfies production requirements.

---

# 96. Architecture Evolution

A production platform should evolve.

Phase 1:

```text
EKS + Terraform + Helm + Argo CD
```

Phase 2:

```text
+ Prometheus
+ Grafana
+ ELK
+ Alertmanager
```

Phase 3:

```text
+ DR
+ advanced security
+ policy enforcement
+ multi-cluster
```

Phase 4:

```text
+ advanced capacity planning
+ automated resilience testing
+ advanced SLO management
```

Do not implement Phase 4 complexity before Phase 1 and Phase 2 are stable.

---

# 97. Operational Maturity Model

## Level 1 - Manual

```text
kubectl apply
manual deployments
manual monitoring
```

## Level 2 - Automated

```text
CI/CD
Terraform
monitoring
```

## Level 3 - GitOps

```text
Git desired state
Argo CD reconciliation
```

## Level 4 - Production mature

```text
SLOs
automated alerting
DR
security automation
capacity planning
```

## Level 5 - Highly mature

```text
automated resilience testing
predictive capacity
advanced policy enforcement
continuous reliability engineering
```

---

# 98. Senior Interview Explanation

A strong interview explanation would be:

> I designed the platform as a layered AWS EKS production architecture. Terraform provisions the AWS foundation and EKS infrastructure. Applications are packaged with Helm and built through CI, where tests, SonarQube, Trivy and Veracode checks are performed before the container image is pushed to ECR. CI then updates the GitOps repository with the immutable application version. Argo CD continuously reconciles the Git desired state with EKS. AWS ALB provides external HTTP/HTTPS ingress. Prometheus collects metrics and evaluates alerts, Grafana provides dashboards, ELK centralizes logs, and Alertmanager routes actionable alerts to the responsible teams. The platform is distributed across multiple AZs, uses Kubernetes autoscaling and resource controls, follows least-privilege security, and includes backup, rollback and disaster recovery procedures.

This explanation demonstrates architecture understanding rather than tool memorization.

---

# 99. Senior Interview Follow-up: Why GitOps?

Answer:

> GitOps gives us a version-controlled desired state. Instead of CI directly changing production, CI produces a validated artifact and updates the deployment configuration. Argo CD then reconciles that desired state with the Kubernetes cluster. This gives us auditability, drift detection, controlled deployment and Git-based rollback.

---

# 100. Senior Interview Follow-up: Why Argo CD?

Answer:

> Argo CD is responsible for continuous reconciliation. It compares the Git desired state with the Kubernetes live state and applies the required changes. It also gives us deployment visibility, drift detection and a centralized control model for multiple EKS clusters.

---

# 101. Senior Interview Follow-up: Why ALB?

Answer:

> The workload is primarily HTTP/HTTPS microservices running on EKS, so AWS ALB provides managed Layer 7 ingress, TLS termination, health checks and Kubernetes integration. API Gateway would add capabilities that are not required for the primary Kubernetes ingress use case.

---

# 102. Senior Interview Follow-up: Why Prometheus?

Answer:

> Prometheus is well suited to Kubernetes because it provides metric collection, PromQL and native alert evaluation. We use Grafana for visualization and Alertmanager for routing, grouping, inhibition and notification management.

---

# 103. Senior Interview Follow-up: Why ELK?

Answer:

> Prometheus handles metrics, but it is not a centralized log search platform. ELK provides centralized application and infrastructure logs, allowing operators to search and correlate events across services and Kubernetes workloads.

---

# 104. Senior Interview Follow-up: What Is the Biggest Risk?

A strong answer:

> The biggest risk is not normally one specific component. It is an operational failure such as an untested recovery procedure, excessive alert noise, weak IAM, poor capacity configuration, or uncontrolled production changes. Production reliability depends on the complete system and operating process.

---

# 105. Senior Interview Follow-up: What Happens If an AZ Fails?

Answer:

> Worker nodes and application replicas are distributed across multiple AZs. The ALB continues routing to healthy targets, Kubernetes schedules workloads on remaining capacity, and autoscaling can replace lost capacity if configured. I would immediately validate node health, pod distribution, ALB targets, capacity and application error rates.

---

# 106. Senior Interview Follow-up: What Happens If Argo CD Fails?

Answer:

> Existing workloads continue running because Kubernetes does not require Argo CD to keep already-applied workloads running. However, new Git changes will not be reconciled while Argo CD is unavailable. For production, I would run Argo CD in HA configuration and maintain a recovery procedure.

---

# 107. Senior Interview Follow-up: What Happens If Git Is Down?

Answer:

> Existing workloads continue running with their current Kubernetes state. New changes cannot be reconciled until Git becomes available. This demonstrates why GitOps is a control-plane dependency rather than a direct runtime dependency.

---

# 108. Senior Interview Follow-up: What Happens If Prometheus Is Down?

Answer:

> Application traffic may continue, but metric visibility and alert evaluation are affected. For critical production environments, Prometheus availability and alerting continuity should be designed appropriately. The incident should be treated as an observability degradation rather than automatically as an application outage.

---

# 109. Senior Interview Follow-up: How Do You Prevent Alert Fatigue?

Answer:

> I classify alerts by severity, add meaningful `for` durations, use SLO-oriented alerts, group related alerts, use inhibition for secondary symptoms, define ownership and runbooks, and continuously review alert frequency. The goal is actionable alerts rather than maximum alert count.

---

# 110. Senior Interview Follow-up: How Do You Review Production Architecture?

Use this sequence:

```text
Business requirements
        |
        v
Availability
        |
        v
Scalability
        |
        v
Security
        |
        v
Observability
        |
        v
Deployment
        |
        v
Failure recovery
        |
        v
Cost
        |
        v
Operational maturity
```

This is a useful senior-level architecture review framework.

---

# 111. Final Architecture Assessment

The capstone architecture is strong because it separates:

```text
Provisioning
     |
Terraform

Packaging
     |
Helm

Build & Security
     |
CI

Artifact
     |
ECR

Desired State
     |
GitOps

Reconciliation
     |
Argo CD

Runtime
     |
EKS

Ingress
     |
ALB

Metrics
     |
Prometheus

Visualization
     |
Grafana

Logs
     |
ELK

Alert Routing
     |
Alertmanager
```

This is a coherent production platform rather than a collection of unrelated tools.

---

# 112. Final Senior-Level Review

The architecture is production-ready only when implementation is accompanied by operational discipline.

Technology alone does not create production reliability.

The real production model is:

```text
Good Architecture
       +
Automation
       +
Security
       +
Observability
       +
Testing
       +
Incident Response
       +
Backup
       +
DR
       +
Operational Discipline
       =
Production Reliability
```

The most important senior DevOps mindset is:

```text
Do not ask only:

"Can I deploy it?"

Also ask:

"Can I observe it?"
"Can I secure it?"
"Can I scale it?"
"Can I troubleshoot it?"
"Can I roll it back?"
"Can I recover it?"
"Can I explain what happened?"
"Can I operate it at 3 AM?"
```

That is the difference between a deployment-focused DevOps engineer and a production-focused DevOps engineer.

---

# 113. Architecture Review Sign-Off Checklist

Before considering the capstone complete:

## Infrastructure

- [ ] AWS architecture reviewed
- [ ] VPC reviewed
- [ ] Multi-AZ design reviewed
- [ ] IAM reviewed
- [ ] Terraform reviewed
- [ ] EKS reviewed
- [ ] ECR reviewed

## Kubernetes

- [ ] Workload design reviewed
- [ ] Resources reviewed
- [ ] Probes reviewed
- [ ] HPA reviewed
- [ ] PDB reviewed
- [ ] scheduling reviewed
- [ ] security reviewed

## Delivery

- [ ] CI reviewed
- [ ] security gates reviewed
- [ ] image immutability reviewed
- [ ] GitOps reviewed
- [ ] Argo CD reviewed
- [ ] rollback reviewed

## Observability

- [ ] Prometheus reviewed
- [ ] Grafana reviewed
- [ ] ELK reviewed
- [ ] Alertmanager reviewed
- [ ] alert ownership reviewed
- [ ] SLO alerting reviewed

## Reliability

- [ ] HA reviewed
- [ ] failure domains reviewed
- [ ] backups reviewed
- [ ] DR reviewed
- [ ] RPO reviewed
- [ ] RTO reviewed
- [ ] restore tests reviewed

## Operations

- [ ] troubleshooting reviewed
- [ ] incident response reviewed
- [ ] runbooks reviewed
- [ ] escalation reviewed
- [ ] production access reviewed

## Cost

- [ ] EKS cost reviewed
- [ ] NAT cost reviewed
- [ ] ALB cost reviewed
- [ ] observability cost reviewed
- [ ] storage cost reviewed
- [ ] data transfer reviewed

---

# 114. Key Takeaways

1. Production architecture is a system of responsibilities, not a list of tools.
2. EKS provides the Kubernetes runtime, not the complete platform.
3. Terraform should manage AWS infrastructure.
4. Helm should package Kubernetes applications.
5. CI should build, test and secure artifacts.
6. ECR should store immutable images.
7. GitOps should represent deployment desired state.
8. Argo CD should reconcile desired state with Kubernetes.
9. ALB should provide external Kubernetes HTTP/HTTPS ingress.
10. Prometheus should provide metrics and alert evaluation.
11. Grafana should provide visualization.
12. ELK should provide centralized logging.
13. Alertmanager should provide alert routing and noise control.
14. Multi-AZ improves availability.
15. Autoscaling must consider both pods and nodes.
16. IAM and RBAC must follow least privilege.
17. Secrets should never be committed as plaintext.
18. Backups are useless until restores are tested.
19. HA is not the same as DR.
20. Rollback is not automatically safe for database changes.
21. Alert quality is more important than alert quantity.
22. Observability systems must themselves be operated reliably.
23. Cost optimization should not blindly sacrifice reliability.
24. Every production change needs an audit trail.
25. A senior DevOps engineer thinks about failure before failure happens.

---

# 115. End of Architecture Review

The next chapter continues the capstone with:

```text
40-Capstone-Interview-Questions.md
```

That chapter will convert the complete architecture into senior-level interview questions and production-style answers covering AWS, Terraform, EKS, Kubernetes, Helm, CI/CD, DevSecOps, GitOps, Argo CD, ALB, Prometheus, Grafana, ELK, security, reliability, troubleshooting, DR, rollback, cost and architecture decisions.
