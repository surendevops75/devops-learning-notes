# 01 — DevOps Production Capstone Requirements

## 1. Purpose

This is the foundation document for the final DevOps Production Capstone.

The capstone simulates the responsibilities of a DevOps, Platform, or SRE engineer operating a production application platform on AWS and Kubernetes.

The objective is not simply to deploy an application. The objective is to demonstrate the complete production lifecycle:

```text
Plan → Design → Provision → Build → Secure → Test
→ Deploy → Observe → Operate → Troubleshoot → Recover → Optimize
```

Every subsequent capstone document builds on these requirements.

---

# 2. Capstone Philosophy

A production platform must satisfy multiple dimensions simultaneously:

```text
                 Production Platform
                        |
      +-----------------+-----------------+
      |                 |                 |
   Reliability        Security        Scalability
      |                 |                 |
      +-----------------+-----------------+
                        |
                 Observability
                        |
                  Operability
                        |
                Recoverability
                        |
                     Cost
```

A system that is fast but insecure is not production-ready.

A system that works but cannot recover from failure is not production-ready.

A system that is secure but impossible to operate is not production-ready.

The capstone therefore evaluates the complete system.

---

# 3. Business Scenario

Build a production-grade microservices platform representing an e-commerce application.

A RoboShop-style reference application can contain:

```text
frontend
catalogue
cart
user
shipping
payment
dispatch
web
```

Supporting infrastructure may include:

```text
MongoDB
Redis
MySQL
RabbitMQ
Kafka
```

The exact dependency of every service must be documented rather than assumed.

---

# 4. Business Requirements

The platform must support:

```text
User access
Product browsing
Cart operations
Order processing
Payment processing
Shipping
Notification/event processing
```

The system should remain available during normal infrastructure failures.

---

# 5. Production Baseline

Training baseline:

```text
Cloud:
AWS

Container platform:
Amazon EKS

Registry:
Amazon ECR

Infrastructure as Code:
Terraform

Packaging:
Helm

CI:
GitLab CI

CD:
Argo CD / GitOps

Ingress:
AWS Load Balancer Controller / ALB

Monitoring:
Prometheus

Dashboards:
Grafana

Logging:
ELK

Tracing:
OpenTelemetry + Jaeger

Messaging:
RabbitMQ and/or Kafka

Secrets:
AWS Secrets Manager / External Secrets

DNS:
Route 53

TLS:
AWS Certificate Manager

Object Storage:
S3
```

These are capstone choices, not universal requirements. Each should be justified against the workload.

---

# 6. Environment Strategy

Support at least:

```text
Development
Staging
Production
```

Reference flow:

```text
                 Git
                  |
             CI Pipeline
                  |
        +---------+---------+
        |         |         |
       Dev     Staging   Production
        |         |         |
      EKS       EKS       EKS
```

Environment-specific configuration must remain controlled and auditable.

---

# 7. Production Environment

Production should represent a high-availability environment.

Required characteristics:

```text
Multi-AZ
Private worker nodes
Controlled public ingress
Encrypted storage
Secure IAM
Centralized secrets
Monitoring
Logging
Alerting
Backup
DR
```

---

# 8. AWS Account Strategy

Recommended organizational model:

```text
AWS Organization
|
+-- Management
+-- Security
+-- Log Archive
+-- Shared Services
+-- Development
+-- Staging
+-- Production
```

If multiple AWS accounts are impractical for a learning environment, separate environments may be simulated inside one account. Document this as an explicit compromise.

---

# 9. Region Strategy

Define:

```text
Primary Region
Secondary / DR Region
```

Example:

```text
Primary: ap-south-1
DR:      ap-southeast-1
```

These are examples. Final selection must consider latency, service availability, compliance, business requirements and cost.

---

# 10. Availability Zones

Production EKS must span multiple Availability Zones.

```text
                 VPC
                  |
       +----------+----------+
       |          |          |
      AZ-A       AZ-B       AZ-C
       |          |          |
    Subnet     Subnet     Subnet
       |          |          |
    Nodes       Nodes      Nodes
```

Critical workloads must not depend on one AZ.

---

# 11. VPC Requirements

Create a dedicated VPC with logical separation:

```text
VPC
|
+-- Public Subnets
|    |
|    +-- ALB
|
+-- Private Application Subnets
|    |
|    +-- EKS Nodes
|
+-- Private Data Subnets
     |
     +-- Data services where appropriate
```

Use appropriate:

```text
route tables
security groups
network ACLs
NAT strategy
VPC endpoints
```

---

# 12. CIDR Planning

Plan CIDRs before Terraform implementation.

Example:

```text
VPC:
10.0.0.0/16

Public:
10.0.1.0/24
10.0.2.0/24
10.0.3.0/24

Private application:
10.0.11.0/24
10.0.12.0/24
10.0.13.0/24

Private data:
10.0.21.0/24
10.0.22.0/24
10.0.23.0/24
```

Final design must account for:

```text
pod networking
service networking
future growth
VPC connectivity
secondary CIDRs
```

Avoid overlapping ranges between connected networks.

---

# 13. Internet Access

Public exposure should be minimized.

Expected path:

```text
Internet
   |
Route 53
   |
ALB
   |
Ingress
   |
Kubernetes Service
   |
Pod
```

Do not expose worker nodes directly to the Internet.

---

# 14. NAT Strategy

Private workloads may require outbound Internet access.

```text
Private subnet
     |
Route Table
     |
NAT Gateway
     |
Internet Gateway
     |
Internet
```

Evaluate:

```text
availability
NAT cost
cross-AZ traffic
VPC endpoints
failure behavior
```

Use private AWS service endpoints where appropriate.

---

# 15. IAM Requirements

Use least privilege for:

```text
Human access
CI
Terraform
EKS workloads
Argo CD
AWS Load Balancer Controller
External Secrets
Monitoring
Logging
```

Avoid broad administrator permissions for routine workloads.

---

# 16. Workload Identity

Application pods should receive only required AWS permissions.

Concept:

```text
Pod
 |
ServiceAccount
 |
IAM Role
 |
AWS API
```

Do not distribute long-lived AWS access keys inside containers.

Use supported EKS/AWS workload identity mechanisms.

---

# 17. Infrastructure as Code

Terraform should manage infrastructure such as:

```text
VPC
Subnets
Route tables
NAT
Security groups
IAM
EKS
Node groups
ECR
S3
KMS
Required AWS integrations
```

Avoid unmanaged production changes.

---

# 18. Terraform State

Use protected remote state.

Concept:

```text
Developer / CI
      |
Terraform
      |
Remote State
      |
S3
```

Use state locking/concurrency protection supported by the selected Terraform/OpenTofu workflow.

Protect state because it may contain sensitive infrastructure information.

---

# 19. EKS Requirements

Production EKS should provide:

```text
Multi-AZ worker capacity
Private networking
Managed node groups and/or Karpenter where justified
Workload identity
Cluster logging where required
Control-plane audit visibility
Cluster security
Upgrade strategy
```

Install only components that have a clear operational purpose.

---

# 20. Node Strategy

Separate node groups where workload isolation justifies it:

```text
System nodes
Application nodes
Observability nodes
```

Potential capacity types:

```text
On-Demand
Spot
```

Use Spot for suitable workloads only.

Do not put critical stateful workloads on interruptible capacity without a deliberate resilience strategy.

---

# 21. Kubernetes Namespace Strategy

Example:

```text
argocd
monitoring
logging
ingress
messaging
dev
staging
production
```

Namespace design should support:

```text
RBAC
quotas
NetworkPolicy
resource management
observability
```

---

# 22. Application Requirements

Production services should define appropriate:

```text
Deployment
Service
ConfigMap
Secret references
ServiceAccount
HPA
PDB
NetworkPolicy
resource requests
resource limits
readiness probe
liveness probe
startup probe when needed
```

Not every workload requires every object.

---

# 23. Container Requirements

Images should:

```text
Use an appropriate minimal base
Run as non-root where possible
Contain no secrets
Control dependency versions
Handle SIGTERM
Expose health behavior
Write logs to stdout/stderr
```

Images must be scanned before production use.

---

# 24. Container Registry

Use ECR.

Example:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/frontend
```

Prefer immutable versioning:

```text
v1.2.3
git-sha
```

Do not rely only on:

```text
latest
```

---

# 25. Image Lifecycle

Define:

```text
Tagging
Retention
Cleanup
Immutability
Vulnerability scanning
Replication where required
```

Avoid unlimited image accumulation.

---

# 26. CI Requirements

Pipeline stages:

```text
checkout
   |
lint
   |
unit tests
   |
integration tests
   |
dependency checks
   |
SAST
   |
secret scanning
   |
container build
   |
container scan
   |
image push
```

Only validated artifacts should progress toward deployment.

---

# 27. DevSecOps Requirements

Integrate security into CI:

```text
SAST
Dependency scanning
Secret detection
Container scanning
IaC scanning
License checks where required
```

Define policies for critical findings rather than merely generating reports.

---

# 28. Artifact Promotion

Preferred lifecycle:

```text
Build once
   |
Immutable artifact
   |
Dev
   |
Staging
   |
Production
```

Do not rebuild an application separately for each environment unless there is a documented reason.

---

# 29. GitOps Requirements

Separate application source from deployment configuration where appropriate.

Application repository:

```text
source
Dockerfile
tests
CI
```

GitOps repository:

```text
environments
Helm values
manifests
Argo CD applications
```

---

# 30. Argo CD Requirements

Argo CD should provide:

```text
Continuous reconciliation
Deployment visibility
Drift detection
Application health
Rollback capability
Environment management
```

Production access must be secured.

---

# 31. Environment Promotion

Recommended:

```text
Developer
   |
CI
   |
Dev
   |
Validation
   |
Staging
   |
Approval / Policy
   |
Production
```

Promotion criteria:

```text
Tests passed
Security checks passed
Artifact available
Deployment healthy
Smoke tests passed
Approval where required
```

---

# 32. Multi-Cluster Requirements

Demonstrate management of:

```text
Dev cluster
Staging cluster
Production cluster
DR cluster
```

Concept:

```text
              Argo CD
          /      |             Dev    Staging   Prod
       EKS      EKS      EKS
```

Document the recovery strategy for the GitOps control plane itself.

---

# 33. Secrets Requirements

Never store production secrets directly in Git.

Preferred:

```text
AWS Secrets Manager
       |
External Secrets
       |
Kubernetes Secret
       |
Application
```

Use KMS-backed protection and least-privilege access where appropriate.

---

# 34. Secret Rotation

Design for:

```text
Create
 |
Deploy
 |
Rotate
 |
Reload / Restart
 |
Validate
```

Document whether applications can dynamically reload secrets.

---

# 35. Ingress Requirements

Use AWS ALB for external HTTP/HTTPS traffic.

```text
Internet
 |
Route 53
 |
ACM Certificate
 |
ALB
 |
Kubernetes Ingress
 |
Service
 |
Pod
```

Configure where appropriate:

```text
HTTPS
HTTP redirect
Health checks
Security groups
Access logging
WAF
```

---

# 36. DNS Requirements

Use Route 53.

Example:

```text
shop.example.com
```

should resolve to the production ALB.

Separate environments through appropriate DNS names or zones.

---

# 37. TLS Requirements

External path:

```text
Client
 |
HTTPS
 |
ALB
 |
Service
```

Use ACM-managed certificates where appropriate.

Evaluate internal encryption according to security requirements.

---

# 38. Autoscaling Requirements

Demonstrate multiple layers:

```text
Pod Autoscaling
       |
Node Autoscaling
       |
AWS Capacity
```

Application scaling may use:

```text
HPA
KEDA
```

when appropriate.

Node scaling may use:

```text
Managed Node Groups
Karpenter
```

based on the chosen architecture.

---

# 39. Scaling Signals

Possible signals:

```text
CPU
Memory
Request rate
Latency
Queue depth
Kafka consumer lag
Custom business metrics
```

Scaling must account for downstream capacity.

---

# 40. Resource Requirements

Every production workload should define requests.

Example:

```yaml
resources:
  requests:
    cpu: "100m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "512Mi"
```

Values are examples only. Determine production values from measurements and load testing.

---

# 41. Health Checks

Use appropriate:

```text
startupProbe
readinessProbe
livenessProbe
```

Do not configure liveness checks so aggressively that temporary dependency problems create restart storms.

---

# 42. Pod Disruption Budget

Critical services should define disruption expectations.

Example:

```text
replicas = 3
minimum available = 2
```

Actual values depend on workload and availability requirements.

---

# 43. Pod Distribution

Avoid concentrating replicas on one node or AZ.

Use where appropriate:

```text
topologySpreadConstraints
podAntiAffinity
node affinity
```

---

# 44. Kubernetes Security

Implement:

```text
RBAC
NetworkPolicy
Pod Security controls
non-root containers
read-only filesystem where practical
seccomp
capability dropping
resource quotas
LimitRanges
service-account isolation
```

Grant only required permissions.

---

# 45. NetworkPolicy

Use restrictive communication where practical.

Example:

```text
frontend -> catalogue
frontend -> cart
cart -> redis
checkout -> payment
payment -> database
```

Block unnecessary lateral movement.

---

# 46. Monitoring Requirements

Prometheus should collect appropriate:

```text
Node metrics
Pod metrics
Kubernetes metrics
Application metrics
Ingress metrics
Messaging metrics
```

Monitor:

```text
Availability
Latency
Traffic
Errors
Saturation
```

---

# 47. Grafana Requirements

Create dashboards for:

```text
Cluster
Nodes
Namespaces
Applications
Ingress
Messaging
Databases where metrics exist
CI/CD
Business health
```

Dashboards should answer operational questions, not merely display every metric.

---

# 48. Logging Requirements

Centralize logs:

```text
Pods
 |
Log Collector
 |
Elasticsearch
 |
Kibana
```

Useful fields:

```text
timestamp
service
environment
pod
namespace
level
message
request ID
trace ID
business ID where appropriate
```

Never log:

```text
passwords
tokens
private keys
unnecessary sensitive customer information
```

---

# 49. Alerting Requirements

Alerts must be actionable.

Examples:

```text
High error rate
High latency
CrashLoopBackOff
Node unavailable
Disk pressure
High consumer lag
Queue growth
DLQ growth
Certificate expiry
Deployment failure
API availability failure
```

Each critical alert should have:

```text
Owner
Severity
Runbook
Notification path
Expected response
```

---

# 50. SLO Requirements

Define service objectives.

Training example:

```text
Availability: 99.9%
p95 latency: < 500 ms
Error rate: < 1%
```

Real targets must come from business requirements.

---

# 51. SLI Requirements

Measure the indicators behind SLOs:

```text
Successful request ratio
Request latency
Queue processing latency
Deployment success rate
```

---

# 52. Error Budget

Conceptually:

```text
100% - SLO = Error Budget
```

Use it to balance reliability and delivery velocity.

---

# 53. Backup Requirements

Back up critical:

```text
Databases
Required object storage
Terraform state
Critical configuration
Required Kubernetes resources
Messaging data according to business requirements
```

Replication is not a substitute for backup.

---

# 54. Restore Requirements

A backup is not trustworthy until restoration is tested.

```text
Backup
 |
Restore
 |
Validate
 |
Reconnect
 |
Verify integrity
```

Record:

```text
Restore duration
Data recovered
Data missing
Errors
```

---

# 55. Disaster Recovery

Define:

```text
RPO
RTO
DR region
Backup strategy
Replication strategy
DNS failover
Secrets recovery
Infrastructure recreation
Application deployment
Data validation
```

---

# 56. DR Exercise

Perform:

```text
Simulate primary failure
 |
Activate DR
 |
Restore infrastructure
 |
Restore data
 |
Deploy applications
 |
Validate dependencies
 |
Switch traffic
 |
Smoke test
 |
Measure RTO/RPO
```

---

# 57. Failure Testing

Intentionally test:

```text
Pod failure
Node failure
AZ workload disruption
Application crash
Bad deployment
Database unavailable
Message broker unavailable
High traffic
Disk pressure
Secret rotation
Certificate issue
Network restriction
```

---

# 58. Rollback Requirements

Application rollback:

```text
Bad version
   |
Detect
   |
Rollback
   |
Health check
   |
Validate
```

Infrastructure rollback requires additional caution because destructive changes may not be safely reversible.

---

# 59. Database Migration Requirements

Treat migrations as production changes.

Consider:

```text
Backward compatibility
Migration ordering
Rollback
Long-running migrations
Locks
Data volume
Application compatibility
```

Prefer expand-and-contract strategies for high-risk schema changes.

---

# 60. Messaging Requirements

Messaging must support:

```text
Durability
Retry
DLQ
Idempotency
Observability
Security
Scaling
Failure recovery
```

Use RabbitMQ/Kafka based on workload requirements.

---

# 61. Application Configuration

Separate:

```text
Code
Configuration
Secrets
```

Environment-specific configuration should not require rebuilding application images.

---

# 62. Cost Requirements

Track:

```text
EKS
EC2
NAT Gateway
ALB
ECR
S3
CloudWatch
OpenSearch/Elasticsearch
Data transfer
Storage
Databases
Messaging
```

Optimize without violating:

```text
Availability
Security
Performance
RTO/RPO
```

---

# 63. Cost Optimization Experiments

Evaluate:

```text
Spot capacity
Right-sizing
NAT reduction using VPC endpoints
S3 lifecycle policies
ECR lifecycle policies
Log retention
Autoscaling
Reserved capacity where appropriate
```

Never optimize only for cost while ignoring operational risk.

---

# 64. Security Hardening

Final platform should address:

```text
AWS IAM
KMS
Security groups
NetworkPolicy
TLS
Secrets
Container security
Kubernetes RBAC
Audit logging
Image scanning
Dependency scanning
IaC scanning
```

---

# 65. Compliance Mindset

Document:

```text
Who can access production
What changes are allowed
Where secrets are stored
How changes are audited
How logs are retained
How backups are protected
How incidents are handled
```

---

# 66. Change Management

Production changes should follow:

```text
Plan
 |
Review
 |
Test
 |
Deploy
 |
Observe
 |
Validate
 |
Document
```

For high-risk changes consider:

```text
Canary
Blue/Green
Progressive delivery
Manual approval
```

---

# 67. Production Access

Human access should be:

```text
Authenticated
Authorized
Auditable
Least privilege
Time-bounded where practical
```

Never share:

```text
Root credentials
Administrator credentials
Database passwords
Cluster-admin credentials
```

---

# 68. CI/CD Security

CI should avoid permanent broad AWS credentials.

Use short-lived or federated identity mechanisms where supported.

Concept:

```text
GitLab
 |
OIDC / Trusted Identity
 |
AWS Role
 |
Temporary Credentials
```

---

# 69. Repository Requirements

Final capstone should produce:

```text
Application repository
Terraform repository
GitOps repository
Helm repository/chart structure
CI/CD configuration
Documentation
Runbooks
Architecture diagrams
```

---

# 70. Documentation Requirements

Document:

```text
Architecture
Deployment
Configuration
Security
Monitoring
Troubleshooting
Backup
Restore
DR
Rollback
Incident response
Cost
```

---

# 71. Architecture Diagram Requirements

Create diagrams for:

```text
AWS architecture
VPC
EKS
Application flow
CI/CD
GitOps
Observability
Logging
Security
DR
```

Each diagram should communicate one primary concept clearly.

---

# 72. Naming Standards

Use consistent naming.

Example:

```text
company-environment-region-component
```

Examples:

```text
roboshop-prod-eks
roboshop-prod-ecr
roboshop-prod-vpc
```

---

# 73. Tagging Requirements

AWS resources should use consistent tags.

Example:

```text
Environment = production
Project = roboshop
ManagedBy = terraform
Owner = platform-team
CostCenter = engineering
```

Tagging supports:

```text
Cost allocation
Ownership
Automation
Inventory
```

---

# 74. Production Readiness Gate

Before declaring production readiness:

```text
[ ] Infrastructure reproducible
[ ] Terraform state protected
[ ] Multi-AZ architecture
[ ] Private worker nodes
[ ] Secure ingress
[ ] TLS
[ ] IAM least privilege
[ ] Secrets protected
[ ] Containers hardened
[ ] Kubernetes RBAC
[ ] NetworkPolicy
[ ] Resource requests
[ ] Health probes
[ ] Autoscaling
[ ] CI
[ ] Security scanning
[ ] GitOps
[ ] Monitoring
[ ] Logging
[ ] Alerting
[ ] Backup
[ ] Restore tested
[ ] DR documented
[ ] DR tested
[ ] Rollback tested
[ ] Failure testing completed
[ ] Runbooks written
[ ] Cost reviewed
```

---

# 75. Definition of Done

The engineer must be able to demonstrate:

## Infrastructure

```text
AWS
Terraform
VPC
EKS
ECR
IAM
KMS
```

## Platform

```text
Kubernetes
Helm
Ingress
Autoscaling
RBAC
NetworkPolicy
```

## Delivery

```text
Git
GitLab CI
DevSecOps
ECR
Argo CD
GitOps
```

## Operations

```text
Prometheus
Grafana
ELK
Alerting
Tracing
Runbooks
```

## Reliability

```text
HA
Retry
DLQ
Idempotency
Backup
Restore
DR
Rollback
```

## Security

```text
IAM
TLS
Secrets
Container security
Kubernetes security
Network controls
```

---

# 76. Required Demonstrations

## Demo 1 — Normal Deployment

```text
Git push
  ↓
CI
  ↓
Security checks
  ↓
Image
  ↓
ECR
  ↓
GitOps update
  ↓
Argo CD
  ↓
EKS
```

Show:

```text
Deployment
Health
Metrics
Logs
```

---

## Demo 2 — Bad Deployment

Deploy an intentionally faulty version.

Demonstrate:

```text
Deployment
|
Failure
|
Alert
|
Investigation
|
Rollback
|
Recovery
```

---

## Demo 3 — Pod Failure

Delete an application pod.

Show:

```text
Pod disappears
|
Kubernetes recreates it
|
Traffic continues
|
Metrics recover
```

---

## Demo 4 — Node Failure

Terminate a worker node in a controlled environment.

Show:

```text
Node failure
|
Pod rescheduling
|
Service recovery
```

---

## Demo 5 — Traffic Spike

Generate controlled load.

Show:

```text
Traffic increases
|
HPA reacts
|
Pods scale
|
Node capacity scales if required
|
Latency remains controlled
```

---

## Demo 6 — Dependency Failure

Isolate a dependency.

Show:

```text
Dependency failure
|
Timeout / retry
|
Alert
|
Controlled degradation
|
Recovery
```

---

## Demo 7 — Messaging Failure

Disrupt a consumer or broker.

Show:

```text
Messages accumulate
|
Recovery
|
Processing resumes
|
Lag decreases
```

---

## Demo 8 — DR

Execute the documented DR procedure.

Show:

```text
Primary failure
|
DR activation
|
Restore
|
Traffic switch
|
Validation
```

Record:

```text
RTO
RPO
```

---

# 77. Interview Acceptance Criteria

Be able to explain:

```text
Why EKS?
Why Terraform?
Why Helm?
Why Argo CD?
Why GitOps?
Why ALB?
Why private subnets?
Why multiple AZs?
Why HPA?
Why Prometheus?
Why Grafana?
Why ELK?
Why Secrets Manager?
Why NetworkPolicy?
Why workload identity?
Why Kafka/RabbitMQ?
Why outbox/idempotency?
Why DR?
Why backups?
Why immutable images?
Why build once and promote?
```

Do not memorize technology names. Explain the trade-offs.

---

# 78. Senior Engineer Expectation

A senior DevOps answer should include:

```text
Trade-offs
Failure modes
Security
Cost
Operability
Scaling
Observability
Recovery
```

Example:

> Why use NAT Gateway?

Weak answer:

> To provide Internet access to private subnets.

Strong answer:

```text
Why outbound access is needed
+
Availability model
+
NAT cost
+
Cross-AZ traffic
+
VPC endpoints
+
Egress requirements
+
Failure behavior
```

---

# 79. Final Capstone Success Model

```text
                 CAPSTONE
                    |
      +-------------+-------------+
      |             |             |
 Infrastructure   Delivery     Security
      |             |             |
    AWS/IaC       CI/CD/GitOps   IAM/TLS
      |             |             |
      +-------------+-------------+
                    |
               Kubernetes
                    |
      +-------------+-------------+
      |             |             |
 Reliability   Observability   Operations
      |             |             |
 HA/DR/Rollback  Metrics/Logs   Runbooks
      |             |             |
      +-------------+-------------+
                    |
                 Business
                    |
              Reliable Service
```

---

# 80. Final Objective

At the end of this capstone, the engineer should be able to say:

> I can take a production application from source control to a secure AWS Kubernetes platform using infrastructure as code, CI/CD, DevSecOps and GitOps. I can design multi-AZ networking, EKS, IAM, ingress, autoscaling and secrets management. I can implement monitoring, logging, alerting and tracing. I can troubleshoot application, Kubernetes, networking and infrastructure failures. I can perform controlled rollback, backup restoration and disaster recovery. I can measure reliability using SLOs and improve the platform based on performance, security and cost data.

That is the standard this capstone is designed to demonstrate.

---

# 81. Dependency Map

The remaining documents build on this requirements document:

```text
01 Requirements
      |
02 Production Architecture
      |
03 Architecture Diagram
      |
04 AWS Account Strategy
      |
05 AWS VPC Architecture
      |
06 Terraform Infrastructure
      |
07 EKS Cluster Architecture
      |
08 ECR / Artifact Management
      |
09 Kubernetes Platform
      |
10 Helm
      |
11 CI
      |
12 DevSecOps
      |
13 GitOps Repository
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
26 Backup / Restore
      |
27 Troubleshooting
      |
28 Incident Response
      |
29 Rollback / Recovery
      |
30 Cost Optimization
      |
31 Security Hardening
      |
32 Production Runbook
      |
33 Complete GitOps
      |
34 Complete Terraform
      |
35 Complete Helm
      |
36 Complete CI/CD
      |
37 Complete Production YAMLs
      |
38 Failure Scenarios
      |
39 Architecture Review
      |
40 Capstone Interview
      |
41 Final DevOps Mock Interview
```

---

# 82. Non-Negotiable Principle

Do not optimize the capstone for:

```text
Number of YAML files
Number of tools
Number of commands
Number of technologies
```

Optimize it for:

```text
Correct architecture
Repeatability
Security
Reliability
Observability
Operability
Recovery
Production reasoning
```

The final project should resemble something a real DevOps/Platform team could operate, not merely a collection of tutorial commands.

---

# 83. Final Capstone Deliverable

The completed project should integrate:

```text
AWS
+
Terraform
+
EKS
+
Docker
+
ECR
+
Kubernetes
+
Helm
+
GitLab CI
+
DevSecOps
+
GitOps
+
Argo CD
+
ALB
+
Autoscaling
+
Secrets
+
Security
+
Prometheus
+
Grafana
+
ELK
+
Alerting
+
Messaging
+
Backup
+
DR
+
Troubleshooting
+
Incident Response
+
Rollback
+
Cost Optimization
+
Production Runbooks
+
Interview Preparation
```

This is not one deployment.

It is a complete production operating model.

---

# 84. Starting Point for Document 02

`02-Production-Architecture.md` must transform these requirements into a concrete end-to-end reference architecture.

It must answer:

```text
What components exist?
Where do they run?
How do they communicate?
How is traffic routed?
Where is data stored?
How is the platform secured?
How does deployment happen?
How is it monitored?
How does it recover?
```

Every later capstone document should remain consistent with that reference architecture.
