# Production Architecture

## 1. Document Purpose

This document converts the requirements from `01-Capstone-Requirements.md` into a concrete production reference architecture.

This architecture becomes the baseline for the remaining capstone implementation documents.

The goal is to design a platform that is:

```text
Highly Available
Secure
Scalable
Observable
Recoverable
Automated
Cost-aware
Operable
```

The architecture is intentionally designed from a production engineer's perspective rather than as a simple tutorial deployment.

---

# 2. Architecture Goals

The platform must support the complete application lifecycle:

```text
Developer
   |
   v
Source Control
   |
   v
CI / DevSecOps
   |
   v
Container Image
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
   |
   v
Application
   |
   +--> Database
   |
   +--> Cache
   |
   +--> Messaging
   |
   +--> External AWS Services
   |
   v
Observability
   |
   +--> Metrics
   +--> Logs
   +--> Traces
   +--> Alerts
```

The architecture must also support:

```text
Failure
   |
Detection
   |
Diagnosis
   |
Recovery
   |
Validation
   |
Post-incident improvement
```

---

# 3. High-Level Architecture

Reference architecture:

```text
                              INTERNET
                                  |
                                  v
                         +----------------+
                         |    Route 53    |
                         +----------------+
                                  |
                                  v
                         +----------------+
                         | AWS WAF        |
                         | Optional       |
                         +----------------+
                                  |
                                  v
                         +----------------+
                         | HTTPS / ALB    |
                         +----------------+
                                  |
                                  v
                    +--------------------------+
                    | EKS Ingress              |
                    +--------------------------+
                                  |
                                  v
                    +--------------------------+
                    | Kubernetes Services      |
                    +--------------------------+
                                  |
                 +----------------+----------------+
                 |                |                |
                 v                v                v
             Frontend        API Services      Workers
                 |                |                |
                 +----------------+----------------+
                                  |
               +------------------+------------------+
               |                  |                  |
               v                  v                  v
            Database            Cache            Messaging
               |                  |                  |
               +------------------+------------------+
                                  |
                                  v
                           External Services
```

---

# 4. AWS Landing Architecture

At organizational level:

```text
AWS Organization
|
+-- Management Account
|
+-- Security Account
|
+-- Log Archive Account
|
+-- Shared Services Account
|
+-- Development Account
|
+-- Staging Account
|
+-- Production Account
|
+-- DR Account / Region
```

A smaller implementation can consolidate accounts, but the logical boundaries should remain documented.

---

# 5. Production Account

The production account should contain only production workloads and their required infrastructure.

Example:

```text
Production Account
|
+-- VPC
|
+-- EKS
|
+-- ECR / replicated artifacts
|
+-- KMS
|
+-- S3
|
+-- IAM
|
+-- ALB
|
+-- Route 53 integrations
|
+-- CloudWatch / AWS audit integration
|
+-- Supporting managed services
```

Do not mix development resources into the production environment unnecessarily.

---

# 6. Region and AZ Architecture

Example:

```text
Primary Region: ap-south-1

                 AWS Region
                     |
       +-------------+-------------+
       |             |             |
      AZ-A          AZ-B          AZ-C
       |             |             |
       v             v             v
    Subnets       Subnets       Subnets
       |             |             |
       +-------------+-------------+
                     |
                    EKS
```

A production cluster should distribute critical capacity across multiple AZs.

---

# 7. VPC Architecture

Reference:

```text
                         VPC 10.0.0.0/16
                               |
       +-----------------------+-----------------------+
       |                       |                       |
       v                       v                       v
   Public AZ-A             Public AZ-B             Public AZ-C
       |                       |                       |
      ALB                     ALB                     ALB
       |                       |                       |
       +-----------------------+-----------------------+
                               |
                    Private Application Subnets
                               |
              +----------------+----------------+
              |                |                |
             AZ-A             AZ-B             AZ-C
              |                |                |
            EKS Nodes        EKS Nodes        EKS Nodes
              |
              v
                    Private Data Subnets
```

---

# 8. Public Subnets

Public subnets contain resources that require public-facing network attachment.

Primary example:

```text
Application Load Balancer
```

Public subnets should not be treated as general-purpose application compute locations.

Avoid putting EKS worker nodes directly in public subnets unless the design has a specific reason.

---

# 9. Private Application Subnets

Private application subnets contain:

```text
EKS worker nodes
internal application resources
supporting platform components
```

The nodes should not have public IP addresses.

Outbound access can use:

```text
NAT Gateway
```

or private AWS service endpoints where possible.

---

# 10. Private Data Subnets

Data services should be isolated from public ingress.

Examples:

```text
RDS
ElastiCache
database nodes
other private data services
```

The application tier should communicate with the data tier through explicitly controlled security rules.

---

# 11. Route Flow

External traffic:

```text
Internet
   |
Route 53
   |
ALB
   |
Target / Ingress
   |
Kubernetes Service
   |
Pod
```

Outbound application traffic:

```text
Pod
 |
Node
 |
Private Route Table
 |
NAT Gateway / VPC Endpoint
 |
AWS Service / Internet
```

Database traffic:

```text
Pod
 |
Service / private endpoint
 |
Database
```

---

# 12. Security Group Architecture

Think in layers:

```text
Internet
   |
ALB Security Group
   |
EKS / Node / Pod Security Controls
   |
Application
   |
Database Security Group
```

Example logical rules:

```text
Internet -> ALB : 443
ALB -> Application : application port
Application -> Database : database port
Application -> Redis : Redis port
Application -> Broker : broker port
```

Do not use:

```text
0.0.0.0/0
```

for internal database access unless there is an exceptional, documented requirement.

---

# 13. EKS Architecture

Reference:

```text
                  EKS Cluster
                      |
       +--------------+--------------+
       |              |              |
     AZ-A           AZ-B           AZ-C
       |              |              |
  Node Group A    Node Group B    Node Group C
       |              |              |
     Pods           Pods           Pods
```

The Kubernetes control plane is AWS-managed.

Worker capacity runs in the VPC.

---

# 14. EKS Node Architecture

Possible design:

```text
EKS
|
+-- System Node Group
|     |
|     +-- CoreDNS
|     +-- CNI components
|     +-- Core platform workloads
|
+-- Application Capacity
|     |
|     +-- Microservices
|
+-- Observability Capacity
      |
      +-- Monitoring / logging components
```

The final separation should be based on resource requirements and operational needs.

---

# 15. On-Demand and Spot Strategy

Example:

```text
Critical:
On-Demand

Fault-tolerant:
Spot
```

Suitable Spot workloads may include:

```text
stateless workers
batch processing
non-critical asynchronous workloads
```

Do not automatically move everything to Spot just to reduce cost.

---

# 16. Kubernetes Logical Architecture

```text
EKS Cluster
|
+-- Platform Namespace
|
+-- Ingress Namespace
|
+-- Argo CD Namespace
|
+-- Monitoring Namespace
|
+-- Logging Namespace
|
+-- Messaging Namespace
|
+-- Development Namespace
|
+-- Staging Namespace
|
+-- Production Namespace
```

The exact namespace strategy can be refined in the Kubernetes platform document.

---

# 17. Application Architecture

Example:

```text
                         ALB
                          |
                       frontend
                          |
              +-----------+-----------+
              |           |           |
          catalogue      cart        user
              |           |           |
              |          Redis        DB
              |
             DB

       checkout/order processing
                 |
        +--------+--------+
        |        |        |
      payment shipping  dispatch
        |        |        |
        +--------+--------+
                 |
              Messaging
```

This represents logical relationships, not necessarily exact application dependencies.

---

# 18. Stateless vs Stateful Workloads

Stateless:

```text
frontend
API services
workers where state is externalized
```

Stateful:

```text
databases
message brokers
persistent storage
```

Prefer managed AWS services for stateful infrastructure when that improves reliability and operational simplicity.

---

# 19. Database Strategy

A production architecture should avoid placing critical databases inside ordinary application pods unless there is a strong reason.

Preferred model:

```text
EKS
 |
Private Network
 |
Managed Database
```

Examples:

```text
RDS
Aurora
ElastiCache
DocumentDB
DynamoDB
```

Selection depends on application requirements.

---

# 20. Cache Architecture

Example:

```text
Application
    |
    v
Redis / ElastiCache
    |
    v
Database
```

Cache should not become the only source of truth unless explicitly designed as such.

Failure behavior must be defined:

```text
Cache unavailable
      |
Application fallback
      |
Database
```

---

# 21. Messaging Architecture

Asynchronous processing:

```text
Application
   |
   v
RabbitMQ / Kafka
   |
   +--> Consumer A
   |
   +--> Consumer B
   |
   +--> Consumer C
```

Messaging is useful for:

```text
decoupling
asynchronous processing
event propagation
load leveling
retry
integration
```

---

# 22. RabbitMQ Placement

For RabbitMQ:

```text
Producers
   |
Exchange
   |
Queue
   |
Consumers
```

Critical messaging workloads require:

```text
durability
HA
persistent queues where required
consumer acknowledgements
retry
DLQ
monitoring
```

---

# 23. Kafka Placement

For Kafka:

```text
Producer
   |
Topic
   |
Partitions
   |
Consumer Group
   |
Consumers
```

Operational requirements include:

```text
partition planning
replication
retention
consumer lag monitoring
broker health
capacity planning
```

---

# 24. Synchronous Communication

Use synchronous communication where the caller requires an immediate response.

Example:

```text
Frontend
   |
HTTP
   |
API
   |
Database
```

Failure should include:

```text
timeout
bounded retry where safe
circuit breaking where appropriate
clear error response
```

---

# 25. Asynchronous Communication

Use asynchronous communication where immediate completion is unnecessary.

Example:

```text
Order Service
      |
      v
Message Broker
      |
      v
Shipping Worker
```

This reduces direct coupling.

---

# 26. CI Architecture

Reference:

```text
Developer
   |
   v
GitLab
   |
   v
GitLab CI
   |
   +--> Lint
   +--> Test
   +--> SAST
   +--> Dependency Scan
   +--> Secret Scan
   +--> IaC Scan
   +--> Docker Build
   +--> Image Scan
   |
   v
ECR
```

The CI pipeline produces the artifact.

It should not become an unmanaged production deployment script.

---

# 27. GitOps Architecture

Deployment responsibility:

```text
CI
 |
Build image
 |
Push ECR
 |
Update GitOps desired state
 |
GitOps Repository
 |
Argo CD
 |
EKS
```

This creates a clean separation:

```text
CI = Build / Validate
CD = Reconcile Desired State
```

---

# 28. Argo CD Architecture

```text
GitOps Repository
       |
       v
    Argo CD
       |
 +-----+-----+-----+
 |           |     |
Dev        Staging Prod
EKS         EKS    EKS
```

Argo CD continuously compares desired state with cluster state.

---

# 29. Production Promotion

Recommended:

```text
Developer
   |
Pull Request
   |
CI
   |
Build Artifact
   |
Dev
   |
Automated Validation
   |
Staging
   |
Approval / Policy
   |
Production
```

Promotion should reference an immutable artifact.

---

# 30. Multi-Cluster Architecture

Reference:

```text
                     GitOps
                       |
                     Argo CD
                       |
       +---------------+---------------+
       |               |               |
       v               v               v
   Dev Cluster     Staging Cluster   Prod Cluster
       |               |               |
      EKS             EKS             EKS
```

A DR cluster may be added:

```text
                     Argo CD
                       |
             +---------+---------+
             |                   |
          Primary              DR
           EKS                  EKS
```

---

# 31. Secrets Architecture

Preferred:

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

Access path:

```text
Pod
 |
ServiceAccount
 |
IAM role
 |
AWS Secrets Manager
```

No static secret should be embedded in:

```text
Dockerfile
Git
Helm values
CI variables without controls
source code
```

---

# 32. Encryption Architecture

Encryption should exist at multiple layers.

```text
Data at Rest
   |
KMS / service encryption

Data in Transit
   |
TLS

Secrets
   |
Secrets Manager / KMS

Terraform State
   |
Encrypted remote backend
```

Encryption does not replace access control.

---

# 33. ALB Architecture

External traffic:

```text
Client
 |
HTTPS
 |
Route 53
 |
ALB
 |
Ingress Rules
 |
Service
 |
Pod
```

The ALB provides:

```text
TLS termination
routing
health checks
availability across AZs
integration with Kubernetes
```

---

# 34. Ingress Routing

Example:

```text
shop.example.com
        |
       ALB
        |
   +----+----+
   |         |
   v         v
 /api      /
   |         |
 API       Frontend
```

Path or host-based routing can be used.

---

# 35. Autoscaling Architecture

```text
Application Load
       |
       v
Metrics
       |
       v
HPA
       |
       v
Pod Count
       |
       v
Node Resource Demand
       |
       v
Karpenter / Node Autoscaling
```

Scaling must be based on measured workload behavior.

---

# 36. Observability Architecture

Three primary signals:

```text
Metrics
Logs
Traces
```

Architecture:

```text
Applications
   |
   +----> Prometheus
   |
   +----> Log Collector -> Elasticsearch -> Kibana
   |
   +----> OpenTelemetry -> Jaeger
```

---

# 37. Metrics

Metrics answer:

```text
Is the service healthy?
How much traffic exists?
How many errors occur?
How much capacity is consumed?
```

Examples:

```text
CPU
Memory
Request rate
Error rate
Latency
Queue depth
Consumer lag
```

---

# 38. Logs

Logs answer:

```text
What happened?
Which service?
Which pod?
Which request?
What error?
```

Use structured logging where possible.

Correlation fields:

```text
request_id
trace_id
service
environment
```

---

# 39. Traces

Tracing answers:

```text
Where did latency occur?
Which downstream service failed?
How did one request traverse the system?
```

Example:

```text
Frontend
   |
Catalogue
   |
Database
```

A trace can reveal which component contributed most of the latency.

---

# 40. Alerting Architecture

```text
Prometheus
    |
Alert Rules
    |
Alertmanager
    |
+---+---+---+
|   |   |   |
Email Slack Pager
```

Only actionable alerts should page engineers.

---

# 41. SLO Architecture

Example:

```text
Request
  |
Success?
  |
Latency?
  |
Availability SLI
  |
SLO
  |
Error Budget
```

Use SLOs to drive operational decisions.

---

# 42. Security Architecture

Security layers:

```text
Internet
   |
WAF / ALB
   |
Network
   |
EKS
   |
RBAC
   |
Pod Security
   |
NetworkPolicy
   |
Application
   |
Data
```

Additional controls:

```text
IAM
KMS
Secrets Manager
image scanning
dependency scanning
audit logging
```

---

# 43. Kubernetes Network Security

Logical model:

```text
Frontend
   |
   +--> Catalogue
   +--> Cart

Cart
   |
   +--> Redis

Payment
   |
   +--> Database
```

NetworkPolicy should block unnecessary lateral communication.

---

# 44. Supply Chain Security

Artifact path:

```text
Source
 |
CI
 |
Dependency Validation
 |
SAST
 |
Secret Scan
 |
Container Build
 |
Container Scan
 |
ECR
 |
Deployment
```

The image digest should be traceable back to source revision.

---

# 45. Image Provenance

Recommended metadata:

```text
Git commit SHA
Build ID
Application version
Build timestamp
Base image information
```

Example:

```text
frontend:v1.4.2
```

with a corresponding commit SHA.

---

# 46. Backup Architecture

```text
Production Data
      |
      +----> Backup
      |
      +----> Snapshot
      |
      +----> Replication where required
```

Backups should have:

```text
retention
encryption
access control
restore procedure
restore testing
```

---

# 47. Disaster Recovery Architecture

Reference:

```text
              Primary Region
                   |
                 EKS
                   |
                Traffic
                   |
             DNS / Route 53
                   |
                   X
              Primary Failure
                   |
                   v
                DR Region
                   |
                 EKS
                   |
              Applications
```

Data replication/restore strategy must be defined per service.

---

# 48. RPO and RTO

Example:

```text
RPO = 15 minutes
RTO = 60 minutes
```

These are illustrative.

Architecture decisions must be driven by actual business requirements.

---

# 49. DR Dependency Chain

DR is not only EKS.

Required recovery dependencies include:

```text
Networking
IAM
KMS
Secrets
Container images
GitOps
DNS
Certificates
Databases
Messaging
Application configuration
Observability
```

A DR plan that restores only Kubernetes is incomplete.

---

# 50. Failure Domain Strategy

Design for:

```text
Pod failure
Node failure
AZ failure
Service failure
Dependency failure
Region failure
```

Recovery becomes progressively more complex:

```text
Pod
 ↓
Node
 ↓
AZ
 ↓
Region
```

---

# 51. Deployment Failure Strategy

If a deployment fails:

```text
Deployment
    |
Health failure
    |
Alert
    |
Investigate
    |
Rollback / Fix Forward
    |
Validation
```

The rollback decision depends on:

```text
severity
blast radius
data compatibility
migration state
availability
```

---

# 52. Database Failure Strategy

If the database becomes unavailable:

```text
Application
    |
Timeout
    |
Retry only if safe
    |
Controlled failure
    |
Alert
    |
Database recovery
    |
Connection recovery
```

Avoid infinite retries.

---

# 53. Messaging Failure Strategy

If consumers fail:

```text
Producer
   |
Broker
   |
Queue / Topic
   |
Consumer failure
   |
Messages accumulate
   |
Alert
   |
Consumer recovery
   |
Backlog processing
```

Monitor backlog and recovery rate.

---

# 54. Observability During Incidents

Incident workflow:

```text
Alert
 |
Grafana
 |
Metrics
 |
Logs
 |
Trace
 |
Kubernetes state
 |
AWS state
 |
Root cause
```

Do not troubleshoot using logs alone.

---

# 55. Production Troubleshooting Layers

Use a structured order:

```text
1. User impact
2. Application
3. Kubernetes
4. Networking
5. AWS infrastructure
6. Dependencies
7. Data
8. Recent changes
```

Example:

```text
HTTP 503
 |
ALB healthy?
 |
Ingress correct?
 |
Service endpoints?
 |
Pods Ready?
 |
Application listening?
 |
Dependency healthy?
```

---

# 56. Recent Change Correlation

When an incident begins:

```text
Current failure
      |
Recent deployment?
      |
Infrastructure change?
      |
Configuration change?
      |
Secret rotation?
      |
Dependency change?
```

Change correlation is often faster than random troubleshooting.

---

# 57. Cost Architecture

Major cost areas:

```text
EKS
EC2
NAT Gateway
ALB
Data transfer
Storage
Databases
Observability
Logging
Messaging
```

Optimization should be continuous.

---

# 58. Cost vs Reliability

Do not blindly minimize cost.

Example:

```text
One NAT Gateway
      |
Lower cost
      |
Potential AZ dependency
```

versus:

```text
NAT per AZ
      |
Higher cost
      |
Better isolation
```

The correct choice depends on business requirements.

---

# 59. Production Access Architecture

Human access:

```text
Engineer
   |
SSO / Identity Provider
   |
AWS IAM
   |
Role
   |
AWS / EKS
```

Avoid permanent shared credentials.

---

# 60. Audit Architecture

Track:

```text
AWS API activity
Kubernetes API activity
Git changes
CI pipeline activity
Argo CD changes
Production deployments
Security events
```

A production system should be able to answer:

> Who changed what, when, and why?

---

# 61. End-to-End Request Flow

Example:

```text
1. User opens shop.example.com
2. Route 53 resolves DNS
3. Client establishes HTTPS connection
4. ALB receives request
5. ALB routes to ingress target
6. Kubernetes Service selects pod
7. Frontend calls API
8. API accesses required data/cache
9. Asynchronous work enters broker
10. Worker consumes event
11. Metrics are generated
12. Logs are collected
13. Trace is propagated
14. Alerts evaluate system health
```

---

# 62. End-to-End Deployment Flow

```text
Developer
   |
Git commit
   |
Merge Request
   |
CI
   |
Tests
   |
Security Scans
   |
Docker Build
   |
Image Scan
   |
ECR
   |
GitOps Repository Update
   |
Argo CD
   |
EKS
   |
Deployment
   |
Health Checks
   |
Metrics / Logs / Traces
```

---

# 63. Infrastructure Change Flow

```text
Terraform Code
   |
Pull Request
   |
terraform fmt
   |
terraform validate
   |
terraform plan
   |
Security / Policy Checks
   |
Review
   |
Apply
   |
AWS
   |
Validate
```

Production applies should be controlled and auditable.

---

# 64. GitOps Change Flow

```text
Application Image
      |
      v
ECR
      |
      v
GitOps values/manifests
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
```

---

# 65. Platform Separation

Separate responsibilities:

```text
Terraform
   |
Cloud Infrastructure

Helm
   |
Application Packaging

GitLab CI
   |
Build / Test / Security

GitOps
   |
Desired Kubernetes State

Argo CD
   |
Reconciliation

Prometheus/Grafana
   |
Metrics / Dashboards

ELK
   |
Logs
```

This separation improves maintainability.

---

# 66. Production Blast Radius

Design changes so failures affect the smallest possible scope.

Examples:

```text
Namespace isolation
Environment isolation
Account isolation
AZ distribution
Canary deployment
Progressive rollout
Least privilege
NetworkPolicy
```

---

# 67. Reliability Principles

Use:

```text
Redundancy
Health checks
Timeouts
Retries
Idempotency
Backpressure
Circuit breaking where appropriate
Graceful shutdown
Load balancing
Autoscaling
Backup
DR
```

Every mechanism should have a purpose.

Retries without backoff can amplify outages.

---

# 68. Graceful Shutdown

Application:

```text
SIGTERM
 |
Stop accepting new work
 |
Finish in-flight requests
 |
Close connections
 |
Exit
```

Kubernetes:

```text
Pod termination
 |
preStop if needed
 |
terminationGracePeriod
 |
process shutdown
```

Applications must handle termination correctly.

---

# 69. Capacity Planning

Plan:

```text
Current traffic
Peak traffic
Growth rate
CPU
Memory
Network
Storage
Database capacity
Message throughput
```

Example:

```text
Current:
100 req/s

Peak:
500 req/s

Expected growth:
2x
```

Capacity planning should include headroom.

---

# 70. Load Testing

Before production readiness:

```text
Baseline test
 |
Load test
 |
Stress test
 |
Spike test
 |
Soak test
 |
Analyze
```

Measure:

```text
p50
p95
p99
error rate
CPU
memory
database latency
queue depth
```

---

# 71. Performance Bottleneck Analysis

Typical path:

```text
High latency
 |
Check ALB
 |
Check ingress
 |
Check pod
 |
Check downstream API
 |
Check cache
 |
Check database
 |
Check network
```

Do not scale blindly.

First identify the bottleneck.

---

# 72. Production Readiness Review

Before release:

```text
Architecture approved
Infrastructure tested
Security reviewed
CI passing
Images scanned
GitOps tested
Rollback tested
Monitoring ready
Alerts tested
Backup verified
Restore tested
DR documented
Runbooks available
Capacity validated
Cost reviewed
```

---

# 73. Operational Runbook Model

Every critical component should have:

```text
Purpose
Dependencies
Normal state
Key metrics
Common failures
Diagnostic commands
Recovery steps
Rollback steps
Escalation
```

---

# 74. Incident Response Model

```text
Detect
 |
Triage
 |
Assign
 |
Contain
 |
Diagnose
 |
Recover
 |
Validate
 |
Communicate
 |
Postmortem
 |
Prevent recurrence
```

---

# 75. Production Change Risk Model

Low-risk:

```text
Documentation
Non-production configuration
```

Medium-risk:

```text
Application deployment
Scaling change
```

High-risk:

```text
Database migration
Networking
IAM
Security groups
EKS upgrade
Terraform destructive change
```

High-risk changes require additional validation.

---

# 76. Architecture Decision Records

Important architectural decisions should be documented.

Example:

```text
ADR-001:
Why EKS?

ADR-002:
Why GitOps?

ADR-003:
Why ALB?

ADR-004:
Why managed database?

ADR-005:
Why multi-AZ?

ADR-006:
Why selected messaging system?
```

Record:

```text
Context
Decision
Alternatives
Trade-offs
Consequences
```

---

# 77. Security Boundaries

Primary boundaries:

```text
Internet
   |
Public AWS
   |
Private AWS
   |
EKS
   |
Namespace
   |
Pod
   |
Application
   |
Data
```

Every boundary should have explicit controls.

---

# 78. Trust Model

Do not assume:

```text
Inside VPC = trusted
Inside Kubernetes = trusted
Inside namespace = trusted
```

Instead:

```text
Authenticate
Authorize
Encrypt
Restrict
Audit
```

---

# 79. Production Data Protection

Protect against:

```text
Accidental deletion
Unauthorized access
Corruption
Ransomware-like operational events
Application bugs
Bad migrations
Region failure
```

Use:

```text
Backups
Versioning where applicable
Encryption
Access controls
Restore testing
```

---

# 80. Reference Production Architecture

Final logical architecture:

```text
                           USERS
                             |
                             v
                         Route 53
                             |
                             v
                       WAF / ALB
                             |
                             v
                      EKS Ingress
                             |
              +--------------+--------------+
              |              |              |
              v              v              v
          Frontend        APIs           Workers
              |              |              |
              |       +------+-------+      |
              |       |      |       |      |
              |       v      v       v      |
              |     DB     Redis   Services  |
              |                       |       |
              |                       v       |
              |                    Broker <---+
              |                       |
              |                +------+------+
              |                |             |
              |             Consumer       Consumer
              |
              +-------------------------------+
                              |
                     Observability
                 +------------+------------+
                 |            |            |
              Metrics       Logs        Traces
                 |            |            |
            Prometheus       ELK        Jaeger
                 |
             Grafana
                 |
             Alerting
```

---

# 81. Recovery Architecture

```text
                 Failure
                    |
          +---------+---------+
          |         |         |
        Pod       Node       AZ
          |         |         |
     Kubernetes  Reschedule  Multi-AZ
          |         |         |
          +---------+---------+
                    |
                Recovery
                    |
              Service Health
```

Regional failure:

```text
Primary Region
      |
      X
      |
DNS / Traffic Management
      |
      v
DR Region
      |
Infrastructure
      |
Data
      |
Applications
      |
Validation
```

---

# 82. Architecture Principles

The capstone follows these principles:

### Principle 1 — Immutable artifacts

Build once and promote.

### Principle 2 — Infrastructure as code

Production infrastructure must be reproducible.

### Principle 3 — Git as source of truth

Desired deployment state belongs in GitOps.

### Principle 4 — Least privilege

Every identity receives only required permissions.

### Principle 5 — Defense in depth

No single security control is assumed sufficient.

### Principle 6 — Observable by default

Every critical service must expose operational signals.

### Principle 7 — Failure is expected

Design for failure rather than assuming perfect infrastructure.

### Principle 8 — Automate repeatable operations

Manual repetitive work should become automation.

### Principle 9 — Recoverability matters

Backups and DR must be tested, not merely documented.

### Principle 10 — Measure before optimizing

Scaling and cost decisions should be evidence-based.

---

# 83. Architecture-to-Document Mapping

This architecture is implemented progressively:

```text
02 Production Architecture
        |
03 Architecture Diagram
        |
04 AWS Account Strategy
        |
05 AWS VPC
        |
06 Terraform
        |
07 EKS
        |
08 ECR
        |
09 Kubernetes
        |
10 Helm
        |
11 CI
        |
12 DevSecOps
        |
13 GitOps
        |
14 Argo CD
        |
15 Multi-Environment
        |
16 Multi-Cluster
        |
17 Secrets
        |
18 ALB
        |
19 Autoscaling
        |
20 Kubernetes Security
        |
21 Monitoring
        |
22 Grafana
        |
23 ELK
        |
24 Alerting
        |
25 DR
        |
26 Backup
        |
27 Troubleshooting
        |
28 Incident Response
        |
29 Rollback
        |
30 Cost
        |
31 Hardening
        |
32 Runbook
        |
33-37 Complete Repositories
        |
38 Failure Scenarios
        |
39 Architecture Review
        |
40 Interview
        |
41 Mock Interview
```

---

# 84. Architecture Review Questions

Before moving forward, the engineer should be able to answer:

```text
1. Where does public traffic enter?
2. Why are EKS nodes private?
3. How are AZ failures handled?
4. How does an application reach AWS services?
5. How does a pod receive AWS permissions?
6. Where are secrets stored?
7. How is an image promoted?
8. Who performs deployment?
9. How does Argo CD detect drift?
10. How are application replicas distributed?
11. How does HPA work?
12. How does node scaling work?
13. How are logs collected?
14. How are metrics collected?
15. How are traces correlated?
16. What happens when a pod crashes?
17. What happens when a node disappears?
18. What happens when an AZ becomes unavailable?
19. How do you rollback?
20. How do you recover data?
21. How do you execute DR?
22. What is your RTO?
23. What is your RPO?
24. What is the largest cost component?
25. How do you prove the system is production-ready?
```

---

# 85. Final Architectural Standard

The final platform should not be judged by how many technologies it contains.

It should be judged by whether it can reliably answer:

```text
How do we deploy?
How do we secure?
How do we scale?
How do we observe?
How do we troubleshoot?
How do we rollback?
How do we recover?
How do we survive failure?
How do we control cost?
How do we prove reliability?
```

If the architecture can answer those questions clearly, the platform is moving toward production readiness.

---

# 86. Next Document

The next file is:

```text
03-Architecture-Diagram.md
```

It will turn this logical architecture into detailed visual architecture views, including:

```text
AWS account architecture
VPC architecture
Subnet architecture
EKS architecture
Application architecture
CI/CD architecture
GitOps architecture
Observability architecture
Security architecture
Messaging architecture
DR architecture
End-to-end traffic flow
End-to-end deployment flow
Failure/recovery flow
```

Those diagrams will become the visual reference for the implementation documents that follow.

---