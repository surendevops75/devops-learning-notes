# GitOps Production Architecture

## 1. Purpose

This file defines production-grade GitOps architectures using Argo CD, Kubernetes, AWS EKS, ECR, Helm, ALB Ingress, CI/CD, Terraform, and enterprise security controls.

The goal is to understand not only how Argo CD deploys an application, but how a complete organization can design:

- Git repositories
- CI pipelines
- artifact promotion
- Argo CD control planes
- single-cluster environments
- multi-cluster environments
- multi-account AWS environments
- multi-region environments
- platform and application separation
- environment isolation
- security boundaries
- observability
- disaster recovery
- operational ownership

The architecture is based on a production-oriented RoboShop microservices platform.

---

# 2. Production GitOps Mental Model

GitOps introduces a control loop:

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
   +--> SAST
   +--> SCA
   +--> Container Scan
   +--> Build
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
Kubernetes API
   |
   v
EKS
   |
   v
Application
```

Argo CD continuously compares:

```text
Desired State
     |
     v
Git
     |
     v
Actual State
     |
     v
Kubernetes
```

and reconciles the difference.

---

# 3. Core Architectural Principle

The most important separation is:

```text
CI builds artifacts.
GitOps controls desired deployment state.
Argo CD reconciles desired state.
Kubernetes runs the application.
```

CI should not normally require direct production cluster credentials.

---

# 4. CI vs CD Responsibilities

## CI

Typical responsibilities:

```text
source checkout
unit tests
lint
SAST
SCA
build
Docker image
Trivy scan
Veracode
push image to ECR
publish artifact metadata
```

## GitOps CD

Typical responsibilities:

```text
update desired image reference
review/approve Git change
Argo CD detects Git change
render manifests
sync to EKS
reconcile
health validation
drift correction
```

---

# 5. Production RoboShop Flow

```text
Developer
   |
   v
Application Git
   |
   v
Jenkins / GitHub Actions
   |
   +--> Maven/npm/Python tests
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
GitOps Repository
   |
   v
Pull Request
   |
   v
Approval
   |
   v
Argo CD
   |
   v
EKS
   |
   v
ALB
   |
   v
Users
```

---

# 6. Why Separate Application and GitOps Repositories?

A common production model is:

```text
application-repo
gitops-repo
```

Application repository:

```text
source code
Dockerfile
tests
CI
```

GitOps repository:

```text
Helm
Kustomize
environment configuration
Argo Applications
ApplicationSets
Projects
```

This provides a clear deployment boundary.

---

# 7. Application Repository

Example:

```text
roboshop-cart/
├── src/
├── tests/
├── Dockerfile
├── pom.xml
└── .github/
    └── workflows/
```

The application repository owns:

```text
application code
```

not production cluster state.

---

# 8. GitOps Repository

Example:

```text
roboshop-gitops/
├── argocd/
├── applications/
├── applicationsets/
├── projects/
├── environments/
├── helm/
├── platform/
└── clusters/
```

The GitOps repository owns:

```text
desired deployment state
```

---

# 9. Production Repository Boundary

A strong architecture separates:

```text
Application Code
        |
        v
CI Pipeline
        |
        v
Artifact Registry
        |
        v
GitOps Repository
        |
        v
Argo CD
```

This creates an auditable chain.

---

# 10. Single EKS Cluster Architecture

Basic production architecture:

```text
                 Git Repository
                       |
                       v
                  Argo CD
                       |
                       v
                Kubernetes API
                       |
                       v
                     EKS
              +--------+--------+
              |        |        |
             Pod      Pod      Pod
              |
              v
           Service
              |
              v
             ALB
              |
              v
            Users
```

---

# 11. Single Cluster Components

Typical components:

```text
EKS Control Plane
Managed Node Groups
Argo CD
AWS Load Balancer Controller
CoreDNS
Metrics Server
Prometheus
Grafana
ELK integration
Application workloads
```

---

# 12. Namespace Architecture

Example:

```text
EKS
├── argocd
├── roboshop
├── monitoring
├── ingress-system
├── external-secrets
└── platform-system
```

Namespaces provide logical boundaries.

They are not a complete security boundary by themselves.

---

# 13. Application Namespace

Example:

```text
roboshop
```

contains:

```text
cart
catalogue
user
orders
payment
shipping
notification
frontend
```

depending on the organization's service ownership model.

---

# 14. Platform Namespace

Platform components can be separated:

```text
argocd
monitoring
ingress-system
external-secrets
```

This makes ownership clearer.

---

# 15. Production Cluster Layers

Think in layers:

```text
Layer 1:
AWS networking

Layer 2:
EKS control plane

Layer 3:
Node compute

Layer 4:
Platform controllers

Layer 5:
GitOps control plane

Layer 6:
Applications

Layer 7:
Observability
```

---

# 16. Multiple EKS Cluster Architecture

A common enterprise model:

```text
                    Git
                     |
                     v
                Central Argo CD
                  /    |    \
                 /     |     \
                v      v      v
            EKS-DEV EKS-QA EKS-PROD
```

One Argo CD instance can manage multiple registered Kubernetes clusters.

---

# 17. Central Argo CD Control Plane

Example:

```text
Management EKS
      |
      +--> Argo CD
      |
      +--> Monitoring
      |
      +--> Platform tooling
```

Target clusters:

```text
Dev EKS
QA EKS
Prod EKS
```

The management cluster hosts the GitOps control plane.

---

# 18. Centralized Multi-Cluster Model

```text
                         Git
                          |
                          v
                    Central Argo CD
                    /      |       \
                   /       |        \
                  v        v         v
               DEV EKS   QA EKS   PROD EKS
```

Advantages:

```text
centralized governance
single GitOps interface
consistent deployment model
centralized audit
ApplicationSet automation
```

Tradeoff:

```text
management-plane dependency
```

---

# 19. Failure Scenario: Central Argo CD Down

If the Argo CD management cluster becomes unavailable:

```text
Existing workloads
continue running
```

because Kubernetes already has their desired resources.

However:

```text
new syncs
drift correction
Git reconciliation
```

may stop until Argo CD is restored.

This is a critical distinction.

---

# 20. Failure Scenario: Target EKS Down

If PROD EKS is unavailable:

```text
Argo CD
   |
   X
PROD EKS
```

Argo CD cannot reconcile that cluster.

Other clusters can continue operating.

---

# 21. Failure Scenario: Git Provider Down

Existing applications continue running.

However:

```text
new Git changes
```

cannot normally be detected or retrieved.

This illustrates an important GitOps property:

> Git is the source of desired state, but a temporary Git outage does not automatically stop already-running Kubernetes workloads.

---

# 22. Failure Scenario: ECR Down

Existing Pods with already-pulled images can continue running.

New Pods may fail if the required image cannot be pulled.

Therefore production architecture should protect:

```text
artifact retention
image availability
regional strategy
```

---

# 23. Multi-Account AWS Architecture

Example:

```text
AWS Organization
|
+-- Shared / Management Account
|      |
|      +-- Argo CD
|
+-- Dev Account
|      |
|      +-- EKS DEV
|
+-- QA Account
|      |
|      +-- EKS QA
|
+-- Prod Account
       |
       +-- EKS PROD
```

---

# 24. Why Separate AWS Accounts?

Benefits:

```text
blast-radius reduction
billing separation
IAM boundaries
production isolation
security governance
compliance
```

Production should not rely on the same AWS account boundary as development when strong isolation is required.

---

# 25. Multi-Account Argo CD Trust Model

```text
Argo CD
   |
   +--> cluster credentials
   |
   +--> EKS DEV
   |
   +--> EKS QA
   |
   +--> EKS PROD
```

The management plane must have only the permissions required for its target clusters.

---

# 26. Cluster Credentials

Argo CD requires a mechanism to authenticate to target clusters.

The exact method depends on:

```text
Argo CD version
EKS authentication model
organization security model
```

Credentials should be:

```text
least privilege
rotatable
audited
protected
```

---

# 27. Cluster Registration Concept

Conceptually:

```text
argocd cluster add <context>
```

registers a target cluster.

The registration establishes:

```text
cluster endpoint
authentication
authorization
cluster metadata
```

In enterprise environments, registration should be automated and governed.

---

# 28. Production Cluster Labels

Use labels such as:

```text
environment=dev
environment=qa
environment=prod

region=ap-south-1
account=prod
team=platform
```

ApplicationSets can use these labels to select clusters.

---

# 29. Multi-Cluster Selection

Example concept:

```text
ApplicationSet
      |
      v
Cluster Generator
      |
      v
environment=prod
      |
      +--> EKS-PROD-AP
      +--> EKS-PROD-US
```

This makes deployment targeting dynamic.

---

# 30. Production Architecture: Multiple Production Clusters

```text
                         Git
                          |
                          v
                    Central Argo CD
                     /          \
                    /            \
                   v              v
             PROD-AP-SOUTH   PROD-US-EAST
                   |              |
                  EKS            EKS
                   |              |
                  ALB            ALB
                   |              |
                Users          Users
```

This supports regional deployment patterns.

---

# 31. Multi-Region Architecture

Possible reasons:

```text
disaster recovery
latency
regional availability
data residency
business continuity
```

Do not introduce multi-region complexity unless the business requires it.

---

# 32. Active-Active Architecture

```text
Users
  |
  +----> Region A EKS
  |
  +----> Region B EKS
```

Both regions run production workloads.

GitOps manages both.

Requirements include:

```text
data strategy
DNS/load balancing
application consistency
secrets
observability
failover
```

---

# 33. Active-Passive Architecture

```text
Primary:
Region A

DR:
Region B
```

GitOps can keep DR configuration synchronized while capacity or application activation is controlled separately.

---

# 34. GitOps and Terraform Boundary

A clean architecture:

```text
Terraform
   |
   +--> VPC
   +--> EKS
   +--> IAM
   +--> RDS
   +--> S3
   +--> ECR
   +--> ALB prerequisites
   |
   v
Infrastructure ready
   |
   v
Argo CD
   |
   +--> Kubernetes workloads
```

---

# 35. Terraform Should Not Fight Argo CD

Avoid:

```text
Terraform manages Deployment
Argo CD manages same Deployment
```

Two systems owning the same resource create drift and ownership conflicts.

---

# 36. Infrastructure Ownership

Terraform:

```text
AWS infrastructure
cluster infrastructure
IAM
networking
databases
```

Argo CD:

```text
Kubernetes application resources
platform resources intentionally assigned to GitOps
```

There can be exceptions, but ownership must be explicit.

---

# 37. ECR Ownership

Terraform can create:

```text
ECR repositories
lifecycle policy
encryption
repository policy
```

CI pushes:

```text
images
```

GitOps references:

```text
image tag/digest
```

---

# 38. ALB Ownership

Terraform can create:

```text
VPC
subnets
security groups
```

AWS Load Balancer Controller can create:

```text
ALB
listeners
target groups
```

from Kubernetes Ingress.

This separates infrastructure prerequisites from dynamic application routing.

---

# 39. Production Ingress Architecture

```text
Internet
   |
   v
Route 53
   |
   v
ALB
   |
   v
AWS Load Balancer Controller
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

No API Gateway is required in this architecture.

---

# 40. ALB Controller Role

The AWS Load Balancer Controller watches Kubernetes resources and provisions AWS load balancing resources.

GitOps manages:

```text
Ingress desired state
```

Controller manages:

```text
AWS implementation
```

---

# 41. Platform vs Application GitOps

A large organization can separate:

```text
platform-gitops
application-gitops
```

Platform:

```text
Argo CD
Ingress controller
monitoring
external secrets
policies
```

Application:

```text
RoboShop
frontend
backend
microservices
```

---

# 42. Platform Team Responsibilities

Typical:

```text
cluster
network integration
GitOps control plane
security guardrails
observability
ingress
secret integration
```

---

# 43. Application Team Responsibilities

Typical:

```text
application code
Docker image
Helm values
deployment configuration
health probes
resources
service configuration
```

---

# 44. Enterprise Ownership Model

```text
Platform Team
      |
      +--> clusters
      +--> Argo CD
      +--> guardrails
      |
      v
Application Teams
      |
      +--> application Git
      +--> GitOps application config
```

The exact boundary depends on organizational maturity.

---

# 45. App of Apps Architecture

Example:

```text
platform-root
       |
       +--> cart
       +--> catalog
       +--> user
       +--> orders
       +--> payment
       +--> shipping
       +--> notification
```

The parent Application manages child Applications.

---

# 46. App of Apps Repository

```text
gitops/
├── applications/
│   ├── cart.yaml
│   ├── catalog.yaml
│   ├── user.yaml
│   └── orders.yaml
└── root/
    └── platform-root.yaml
```

---

# 47. ApplicationSet vs App of Apps

ApplicationSet:

```text
generates Applications dynamically
```

App of Apps:

```text
parent Application manages child Applications
```

They solve different problems and can be used together.

---

# 48. Production Git Repository Strategy

One possible structure:

```text
gitops/
├── argocd/
│   ├── projects/
│   ├── applications/
│   └── applicationsets/
│
├── platform/
│   ├── ingress/
│   ├── monitoring/
│   └── external-secrets/
│
├── applications/
│   └── roboshop/
│
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
└── clusters/
    ├── dev/
    ├── qa/
    └── prod/
```

---

# 49. Environment Isolation

Production should not accidentally consume:

```text
dev values
QA secrets
test images
```

Use explicit:

```text
values-dev.yaml
values-qa.yaml
values-prod.yaml
```

and controlled Application/ApplicationSet definitions.

---

# 50. Branching Strategy

A practical strategy is:

```text
main
```

as the production-quality GitOps source, with changes introduced through pull requests.

Alternative:

```text
main
release/*
environment branches
```

can work, but adds complexity.

The key is not the branch name.

The key is:

```text
clear promotion rules
review
auditability
```

---

# 51. Image Promotion Architecture

Recommended flow:

```text
Build image
    |
    v
Scan
    |
    v
ECR
    |
    v
Promote immutable artifact
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

The same immutable artifact should ideally be promoted rather than rebuilt for each environment.

---

# 52. Tagging Strategy

Avoid relying only on:

```text
latest
```

Prefer:

```text
cart:1.7.3
```

or immutable digest:

```text
cart@sha256:...
```

---

# 53. Why Immutable Images Matter

If:

```text
cart:1.7.3
```

can be overwritten, then the same Git state can produce different runtime code.

Immutable artifacts provide stronger reproducibility.

---

# 54. GitOps Deployment Record

A production Git commit should allow you to answer:

```text
What version?
Which image?
Which configuration?
Which environment?
Who approved?
When deployed?
```

---

# 55. Production Approval Flow

```text
Developer
   |
   v
CI
   |
   v
Image
   |
   v
GitOps PR
   |
   v
Automated validation
   |
   v
Reviewer
   |
   v
Merge
   |
   v
Argo CD
```

For production, additional approval gates may be required.

---

# 56. Pull-Based Security Architecture

Traditional push CD:

```text
CI
 |
 +--> production credentials
 |
 v
Kubernetes
```

GitOps:

```text
CI
 |
 v
Git

Argo CD
 |
 v
Kubernetes
```

CI does not need direct production cluster write access.

---

# 57. Blast Radius Reduction

If CI is compromised:

```text
push-based:
attacker may access production cluster
```

GitOps:

```text
attacker may modify Git
```

Then protected branch/PR controls can prevent direct production deployment.

GitOps does not eliminate risk, but can significantly improve separation of duties.

---

# 58. Git Repository Security

Use:

```text
branch protection
required reviews
signed commits where required
secret scanning
CODEOWNERS
least privilege
short-lived credentials where possible
```

---

# 59. CODEOWNERS

Production paths can require platform approval.

Example:

```text
/environments/prod/ @platform-team
/argocd/            @platform-team
```

Application teams can own:

```text
/applications/roboshop/
```

according to the organization.

---

# 60. Argo CD RBAC Architecture

```text
User
 |
 v
SSO
 |
 v
Argo CD RBAC
 |
 +--> platform-admin
 +--> developer
 +--> readonly
 +--> application-team
```

Permissions should be scoped.

---

# 61. AppProject Security Boundary

Projects can restrict:

```text
repositories
clusters
namespaces
resource kinds
```

Example concept:

```text
roboshop-prod project
        |
        +--> GitOps repository
        +--> PROD cluster
        +--> roboshop namespace
```

---

# 62. Secret Architecture

Never store raw production secrets in ordinary Git.

Possible pattern:

```text
Git
 |
 v
ExternalSecret
 |
 v
Secrets Manager
 |
 v
External Secrets Operator
 |
 v
Kubernetes Secret
 |
 v
Pod
```

---

# 63. AWS Secrets Manager Integration

```text
AWS Secrets Manager
        |
        v
External Secrets Operator
        |
        v
Kubernetes Secret
        |
        v
Application
```

Authentication should use an appropriate AWS workload identity mechanism.

---

# 64. Secret Failure Scenario

If secret retrieval fails:

```text
ExternalSecret
   |
   X
AWS Secrets Manager
```

Application may fail to start.

Troubleshoot:

```text
IAM
secret name
region
KMS
operator
network
```

---

# 65. Production Network Architecture

```text
                    Internet
                       |
                       v
                  Route 53
                       |
                       v
                      ALB
                       |
                 Public Subnets
                       |
                       v
                Private EKS Nodes
                       |
            +----------+----------+
            |          |          |
           Pod        Pod        Pod
            |
            v
       Private services
```

Applications should normally run on private worker networking where appropriate.

---

# 66. EKS VPC Architecture

Typical:

```text
VPC
├── Public Subnets
│   └── ALB
│
└── Private Subnets
    ├── EKS nodes
    ├── Pods
    └── internal services
```

NAT or VPC endpoints can provide controlled outbound access depending on design.

---

# 67. AWS Dependency Architecture

RoboShop may depend on:

```text
ECR
RDS
ElastiCache/Redis
S3
Secrets Manager
Cloud networking
```

GitOps manages Kubernetes configuration for these integrations, while Terraform or AWS-native infrastructure tooling manages the underlying AWS resources where appropriate.

---

# 68. Database Ownership

Do not normally let an application Helm chart create production databases without a deliberate architecture.

Prefer:

```text
Terraform / platform provisioning
       |
       v
RDS
       |
       v
Application
```

Argo CD deploys application configuration referencing the database.

---

# 69. Database Migration Architecture

Migration can be:

```text
CI migration
```

or:

```text
Argo CD hook/job
```

or:

```text
dedicated migration platform
```

Choose based on:

```text
idempotency
rollback
locking
data safety
availability
```

---

# 70. Production Database Migration Risk

Application rollback does not necessarily roll back database schema.

Therefore:

```text
application rollback
!=
database rollback
```

Use backward-compatible migrations where possible.

---

# 71. Progressive Delivery Architecture

Basic:

```text
Argo CD
 |
 v
100% new version
```

Progressive:

```text
Argo CD
 |
 v
10% new
90% old
 |
 v
validation
 |
 v
50%
 |
 v
100%
```

This may use additional progressive delivery tooling.

---

# 72. Blue-Green

```text
ALB
 |
 +--> Blue
 |
 +--> Green
```

Deploy Green:

```text
validate
switch traffic
```

Rollback:

```text
switch back
```

---

# 73. Canary

```text
Users
 |
 +--> 90% old
 |
 +--> 10% new
```

Gradually increase new version traffic.

---

# 74. Progressive Delivery Decision

Use progressive delivery for:

```text
high-risk changes
critical services
large user bases
```

Simple rolling updates may be enough for:

```text
low-risk services
internal workloads
```

---

# 75. Observability Architecture

```text
Applications
   |
   +--> Metrics --> Prometheus --> Grafana
   |
   +--> Logs ----> ELK
   |
   +--> Kubernetes events
   |
   +--> ALB metrics/logs where integrated
```

Argo CD should also be monitored.

---

# 76. GitOps Observability

Monitor:

```text
Application health
sync failures
OutOfSync duration
reconciliation
controller errors
repo-server errors
ApplicationSet errors
```

---

# 77. Alerting Architecture

```text
Prometheus
   |
   v
Alerting
   |
   v
On-call
```

Examples:

```text
Prod Application Degraded
Prod Application OutOfSync > threshold
Sync failures
ALB unhealthy
Pod restart spike
HPA max replicas
Node NotReady
```

---

# 78. Logging Architecture

```text
Pods
 |
 v
Container logs
 |
 v
Log collector
 |
 v
Elasticsearch
 |
 v
Kibana
```

Use correlation fields where possible:

```text
service
version
request ID
environment
cluster
namespace
```

---

# 79. GitOps Audit Architecture

```text
Git history
      |
      +--> who changed
      +--> what changed
      +--> review
      |
      v
Argo CD history
      |
      +--> deployed revision
      |
      v
Kubernetes
      |
      +--> resource state
```

This creates a strong audit trail.

---

# 80. Disaster Recovery Architecture

DR should restore:

```text
GitOps repository
Argo CD
cluster
secrets integration
infrastructure
applications
```

Git alone is not the complete DR solution.

---

# 81. GitOps DR Advantage

Because Kubernetes desired state is stored declaratively:

```text
new EKS cluster
       |
       v
Argo CD bootstrap
       |
       v
Git
       |
       v
Applications restored
```

This can dramatically reduce manual reconstruction.

---

# 82. What Must Be Backed Up?

Depending on architecture:

```text
Git repositories
Argo CD configuration
AppProjects
Applications/ApplicationSets
repository credentials
cluster registration configuration
secret-management configuration
AWS infrastructure state
external secret references
critical Kubernetes state not reconstructible from Git
```

---

# 83. What Should Not Be Treated as Backup?

Do not assume:

```text
running Pods
```

are backup.

Pods are ephemeral.

Desired configuration and persistent data require separate recovery strategies.

---

# 84. Persistent Data DR

Applications may use:

```text
RDS
Redis
S3
EBS
```

Their DR must be designed independently.

GitOps can restore application configuration, but cannot magically restore lost database data.

---

# 85. Argo CD Management Cluster DR

If Argo CD runs on a dedicated management EKS cluster:

```text
backup/restore
or
rebuild cluster + bootstrap Argo CD
```

should be tested.

---

# 86. Bootstrap Architecture

A common pattern:

```text
Terraform
   |
   v
EKS
   |
   v
Argo CD installation
   |
   v
Root Application
   |
   v
Platform Applications
   |
   v
ApplicationSets
   |
   v
Business Applications
```

This creates a repeatable bootstrap path.

---

# 87. Bootstrap Repository

Example:

```text
bootstrap/
├── argocd/
├── projects/
├── root/
└── platform/
```

The first Argo CD Application can bootstrap the rest.

---

# 88. Bootstrap Dependency Problem

You cannot use Argo CD to install Argo CD before Argo CD exists.

Therefore:

```text
Terraform/Helm/bootstrap script
        |
        v
Argo CD
        |
        v
GitOps everything else
```

This boundary must be explicit.

---

# 89. Enterprise Bootstrap

```text
AWS Organizations
        |
        v
Terraform
        |
        v
VPC + EKS + IAM
        |
        v
Argo CD
        |
        v
AppProjects
        |
        v
Applications/ApplicationSets
        |
        v
Platform
        |
        v
Applications
```

---

# 90. High Availability Architecture

Argo CD should be deployed according to its supported HA architecture for production environments.

Key concerns:

```text
server availability
repo-server availability
controller availability
Redis availability
network availability
storage/configuration recovery
```

---

# 91. EKS Node High Availability

Use multiple Availability Zones.

```text
EKS
|
+-- AZ-A
|    +-- Nodes
|
+-- AZ-B
|    +-- Nodes
|
+-- AZ-C
     +-- Nodes
```

Application replicas should be distributed where appropriate.

---

# 92. Pod Distribution

Use:

```text
topologySpreadConstraints
podAntiAffinity
```

to reduce correlated failures.

---

# 93. Production Replica Strategy

For critical services:

```yaml
replicas: 3
```

may be a starting point, but the correct number depends on:

```text
traffic
SLO
capacity
failure model
cost
```

Do not use 3 as a universal rule.

---

# 94. Resource Management

Every production workload should define appropriate:

```text
requests
limits
```

Requests influence scheduling.

Limits constrain resource consumption.

---

# 95. Resource Architecture

```text
Application
 |
 +--> requests
 |
 +--> limits
 |
 +--> HPA
 |
 +--> PDB
 |
 +--> node capacity
```

These settings must be considered together.

---

# 96. SecurityContext Architecture

Production workloads should generally use hardened settings where compatible:

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  seccompProfile:
    type: RuntimeDefault
```

Container capabilities should be minimized.

---

# 97. Network Security

Layered security:

```text
AWS Security Groups
       |
       v
Kubernetes NetworkPolicy
       |
       v
Application authentication
       |
       v
TLS
```

No single layer should be treated as the only security boundary.

---

# 98. Production TLS

Typical:

```text
User
 |
HTTPS
 |
ALB
 |
HTTPS or HTTP
 |
Service
 |
Pod
```

Whether TLS terminates at the ALB or continues internally depends on security requirements.

---

# 99. Internal Service Communication

For sensitive workloads:

```text
Service
 |
 mTLS/TLS where required
 |
 Dependency
```

The architecture should reflect compliance and threat model.

---

# 100. Multi-Tenant Architecture

If multiple teams share EKS:

```text
Team A namespace
Team B namespace
Team C namespace
```

Add:

```text
ResourceQuota
LimitRange
NetworkPolicy
RBAC
AppProject
```

Namespace separation alone is insufficient.

---

# 101. Argo CD Project Per Team

Possible model:

```text
project-team-a
project-team-b
project-team-c
```

Each project restricts:

```text
Git repositories
clusters
namespaces
resource types
```

---

# 102. Production Environment Promotion

A mature promotion flow:

```text
DEV
 |
 v
automated tests
 |
 v
QA
 |
 v
validation
 |
 v
PROD approval
 |
 v
PROD
```

The exact gates depend on risk and organization.

---

# 103. Promotion Should Move Artifacts, Not Rebuild Them

Bad:

```text
Build for DEV
Build again for QA
Build again for PROD
```

Preferred:

```text
Build once
 |
 v
Scan
 |
 v
Promote same immutable artifact
```

This improves reproducibility.

---

# 104. GitOps Environment Configuration

Example:

```text
environments/
├── dev/
│   └── values.yaml
├── qa/
│   └── values.yaml
└── prod/
    └── values.yaml
```

Keep differences intentional.

---

# 105. Avoid Configuration Duplication

If 95% of values are identical:

```text
base
+
environment overlay
```

is often better than three complete copies.

Helm and Kustomize can implement this.

---

# 106. Environment-Specific Differences

Examples:

```text
replicas
resources
hostnames
external dependencies
feature flags
autoscaling
PDB
ingress
```

Secrets should come from a secret-management system rather than plain values files.

---

# 107. Production GitOps Directory Architecture

```text
gitops/
|
+-- argocd/
|   +-- projects/
|   +-- applications/
|   +-- applicationsets/
|
+-- helm/
|   +-- roboshop/
|
+-- environments/
|   +-- dev/
|   +-- qa/
|   +-- prod/
|
+-- platform/
|   +-- ingress/
|   +-- monitoring/
|   +-- secrets/
|
+-- clusters/
    +-- dev/
    +-- qa/
    +-- prod/
```

---

# 108. Application-Level Repository Architecture

```text
helm/
└── roboshop/
    ├── cart/
    ├── catalog/
    ├── user/
    ├── orders/
    ├── payment/
    ├── shipping/
    └── notification/
```

---

# 109. Production Application Structure

Example:

```text
cart/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-qa.yaml
├── values-prod.yaml
└── templates/
    ├── deployment.yaml
    ├── service.yaml
    ├── ingress.yaml
    ├── hpa.yaml
    ├── pdb.yaml
    └── serviceaccount.yaml
```

---

# 110. AppProject Architecture

A production project should express:

```text
who can deploy
where they can deploy
what they can deploy
from which repositories
```

This is an authorization boundary.

---

# 111. Application Architecture

Each Application defines:

```text
source
destination
project
sync behavior
```

For example:

```text
cart-prod
 |
 +--> Git repo
 +--> Helm path
 +--> prod revision
 +--> PROD EKS
 +--> roboshop namespace
```

---

# 112. ApplicationSet Architecture

ApplicationSet provides fleet automation.

Example:

```text
one template
   |
   +--> dev
   +--> qa
   +--> prod
```

or:

```text
one template
   |
   +--> EKS-DEV
   +--> EKS-QA
   +--> EKS-PROD
```

---

# 113. Multi-Cluster ApplicationSet

```text
Cluster Generator
       |
       v
environment=prod
       |
       +--> cluster-prod-ap
       +--> cluster-prod-us
       +--> cluster-prod-eu
```

This avoids manually maintaining many Applications.

---

# 114. Cluster Generator Metadata

A cluster Secret can expose labels/metadata used for selection.

Examples:

```text
environment
region
account
team
tier
```

ApplicationSet uses these attributes to build dynamic Applications.

---

# 115. Dynamic Destination

Conceptually:

```yaml
destination:
  server: '{{server}}'
```

where:

```text
{{server}}
```

comes from generator data.

This enables centralized multi-cluster deployment.

---

# 116. Multi-Cluster Security

Do not give every Application access to every cluster.

Use:

```text
AppProjects
cluster restrictions
namespace restrictions
RBAC
Git repository restrictions
```

---

# 117. Production Control Plane Trust Boundary

```text
                    TRUST BOUNDARY
                         |
                         v
                 +---------------+
                 |   Argo CD     |
                 +---------------+
                    |    |    |
                    v    v    v
                   DEV  QA   PROD
```

Production credentials must be more tightly protected.

---

# 118. Production Credential Isolation

Possible model:

```text
Argo CD
 |
 +--> Dev credentials
 +--> QA credentials
 +--> Prod credentials
```

Production access should be restricted to the Argo CD control plane and authorized operators.

---

# 119. Management Plane vs Workload Plane

Management plane:

```text
Argo CD
monitoring
platform controllers
```

Workload plane:

```text
RoboShop
```

This separation improves operational clarity.

---

# 120. Dedicated Management Cluster

For larger organizations:

```text
Management EKS
 |
 +--> Argo CD
 +--> monitoring
 +--> platform services

Workload EKS
 |
 +--> applications
```

This reduces competition between management and workload resources.

---

# 121. Tradeoff of Dedicated Management Cluster

Advantages:

```text
stronger isolation
independent scaling
better blast-radius control
```

Disadvantages:

```text
additional cluster
additional cost
additional operations
```

---

# 122. Small Organization Architecture

For a smaller platform:

```text
One EKS
 |
 +--> Argo CD
 +--> Applications
```

This is simpler.

---

# 123. Enterprise Organization Architecture

For larger platforms:

```text
Management EKS
       |
       v
Central Argo CD
       |
 +-----+-----+-----+
 |     |     |     |
 v     v     v     v
DEV   QA   PROD  DR
EKS   EKS   EKS   EKS
```

---

# 124. When to Use Central Argo CD

Centralized management is attractive when:

```text
many clusters
consistent governance
central platform team
standardized GitOps
```

are important.

---

# 125. When Separate Argo CD Instances Make Sense

Separate instances may be preferred for:

```text
strong isolation
regulatory boundaries
independent business units
regional autonomy
extreme blast-radius requirements
```

---

# 126. Centralized vs Distributed

| Model | Advantage | Tradeoff |
|---|---|---|
| Central Argo CD | centralized governance | management-plane dependency |
| Per-cluster Argo CD | isolation | more operational overhead |
| Per-region Argo CD | regional autonomy | more complexity |
| Hybrid | flexible | governance complexity |

---

# 127. Hybrid Architecture

Example:

```text
Global Argo CD
 |
 +--> Dev/QA clusters

Regional Argo CD
 |
 +--> Production clusters
```

This can reduce production blast radius while preserving centralized governance for lower environments.

---

# 128. Production Change Ownership

Define:

```text
Application team
Platform team
Security team
SRE/Operations
```

for different resource categories.

Without ownership, GitOps repositories can become chaotic.

---

# 129. CODEOWNERS Architecture

Example:

```text
/argocd/              platform
/platform/            platform
/environments/prod/   platform + application
/helm/roboshop/       application team
```

---

# 130. Production PR Validation

Before merge:

```text
YAML validation
Helm lint
Helm template
Kubernetes schema validation
policy validation
security scanning
```

Possible tooling depends on the organization.

---

# 131. GitOps Policy as Code

Policy can prevent:

```text
privileged containers
hostNetwork
latest tags
missing resources
missing probes
public LoadBalancers
```

before production deployment.

---

# 132. Security Pipeline

```text
Pull Request
 |
 +--> YAML validation
 +--> Helm lint
 +--> policy checks
 +--> secret scan
 |
 v
Approval
 |
 v
Merge
 |
 v
Argo CD
```

---

# 133. Production Drift Strategy

Recommended:

```text
selfHeal enabled
```

where appropriate.

But some resources may intentionally be controlled by:

```text
HPA
operators
cloud controllers
```

Therefore ownership must be explicit.

---

# 134. Drift and External Controllers

Examples:

```text
HPA -> replicas
ALB Controller -> AWS resources
External Secrets -> Secret data
cert-manager -> Certificates
operators -> custom resources
```

Argo CD should not fight legitimate controllers.

---

# 135. Resource Ownership Matrix

Example:

| Resource | Owner |
|---|---|
| VPC | Terraform |
| EKS | Terraform |
| ECR repository | Terraform |
| Deployment | Argo CD |
| Service | Argo CD |
| Ingress | Argo CD |
| ALB | AWS Load Balancer Controller |
| Replica count | HPA |
| Secret data | External Secrets |
| Application code | Git application repo |

---

# 136. Production Sync Strategy

For DEV:

```text
automated sync
prune
selfHeal
```

may be appropriate.

For PROD:

```text
automated or controlled sync
approval
maintenance windows
```

depending on risk.

---

# 137. Sync Windows

Organizations may restrict production synchronization during:

```text
business hours
freeze periods
critical events
```

unless an emergency process overrides the restriction.

---

# 138. Production Rollout Strategy

Typical:

```text
RollingUpdate
```

with:

```text
readiness
startup
liveness
PDB
topology
resources
```

For high-risk changes:

```text
canary
blue-green
```

may be better.

---

# 139. Production Rollout Architecture

```text
Git
 |
 v
Argo CD
 |
 v
Deployment
 |
 +--> old ReplicaSet
 |
 +--> new ReplicaSet
 |
 v
Readiness
 |
 v
Service
 |
 v
ALB
```

---

# 140. Failure During Rolling Update

If new Pods fail readiness:

```text
old Pods may remain serving
```

depending on rollout configuration.

This is why readiness is a critical safety mechanism.

---

# 141. Deployment Availability

The production design should prevent:

```text
all replicas disappearing
```

during normal rollout.

Use:

```text
replicas
maxUnavailable
maxSurge
PDB
```

together.

---

# 142. Production Namespace Policy

Example namespace:

```text
roboshop-prod
```

can have:

```text
ResourceQuota
LimitRange
NetworkPolicy
RBAC
Pod security controls
```

---

# 143. Production ResourceQuota

Purpose:

```text
prevent one team from consuming entire cluster capacity
```

Control:

```text
CPU
memory
Pods
Services
```

according to requirements.

---

# 144. Production NetworkPolicy Model

Example:

```text
frontend
   |
   v
cart
   |
   v
redis
```

Only required flows should be allowed.

---

# 145. Production Service Account

Use dedicated ServiceAccounts.

Avoid:

```text
default
```

for sensitive workloads.

---

# 146. AWS Workload Identity

Applications needing AWS APIs should use an appropriate workload identity mechanism rather than storing:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

inside Pods.

---

# 147. Production EKS Security Model

```text
IAM
 |
 v
EKS authentication
 |
 v
Kubernetes RBAC
 |
 v
ServiceAccount
 |
 v
Application
```

The exact EKS authentication mechanism depends on the EKS configuration/version.

---

# 148. Production Observability Ownership

Platform monitors:

```text
cluster
Argo CD
controllers
nodes
```

Application teams monitor:

```text
application SLI/SLO
business errors
dependencies
```

Both layers should be connected.

---

# 149. SLI/SLO Architecture

For a critical API:

```text
Availability
Latency
Error rate
```

Example:

```text
99.9% availability
p95 latency < target
5xx < target
```

Actual targets must be business-defined.

---

# 150. Deployment Safety Metrics

Before and after deployment compare:

```text
error rate
latency
request rate
CPU
memory
restart count
ALB target health
```

---

# 151. GitOps Deployment Dashboard

A useful dashboard includes:

```text
Applications Synced
Applications OutOfSync
Applications Degraded
Recent sync failures
Recent deployments
Cluster health
Argo CD controller health
```

---

# 152. Production Architecture for RoboShop

```text
                         Developers
                              |
                              v
                     Application Git
                              |
                              v
                  Jenkins / GitHub Actions
                              |
          +-------------------+-------------------+
          |                   |                   |
       SonarQube            Trivy             Veracode
          |                   |                   |
          +-------------------+-------------------+
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
                +-------------+-------------+
                |             |             |
                v             v             v
             DEV EKS       QA EKS       PROD EKS
                |             |             |
               ALB           ALB           ALB
                |             |             |
              Users         Users         Users
```

---

# 153. RoboShop GitOps Service Model

Example services:

```text
frontend
user
catalogue
cart
shipping
payment
orders
notification
```

Each can be represented as an independent Application or grouped according to ownership and lifecycle.

---

# 154. RoboShop Deployment Flow

```text
Developer changes cart
        |
        v
Application Git
        |
        v
CI
        |
        v
Tests + Security
        |
        v
ECR image
        |
        v
GitOps PR
        |
        v
Review
        |
        v
Argo CD
        |
        v
EKS
        |
        v
Cart Pods
```

---

# 155. RoboShop Production Environment

```text
PROD EKS
|
+-- argocd
|
+-- roboshop-prod
|    +-- frontend
|    +-- cart
|    +-- catalogue
|    +-- user
|    +-- orders
|    +-- payment
|    +-- shipping
|    +-- notification
|
+-- monitoring
+-- ingress-system
+-- external-secrets
```

---

# 156. Production Traffic Flow

```text
Client
 |
 v
Route 53
 |
 v
AWS ALB
 |
 v
Ingress
 |
 v
frontend Service
 |
 v
frontend Pod
 |
 v
backend Services
 |
 +--> cart
 +--> user
 +--> catalogue
 +--> orders
```

---

# 157. Production Dependency Flow

```text
cart
 |
 +--> Redis

orders
 |
 +--> database

payment
 |
 +--> payment provider

notification
 |
 +--> messaging/email provider
```

The actual dependency map should reflect the application implementation.

---

# 158. GitOps Ownership in RoboShop

```text
Terraform:
AWS/EKS/network/IAM

CI:
build/test/scan/image

GitOps:
Helm/Kubernetes desired state

Argo CD:
reconciliation

EKS:
runtime

AWS Load Balancer Controller:
ALB

Prometheus/Grafana:
metrics

ELK:
logs
```

---

# 159. Production GitOps Repository Example

```text
roboshop-gitops/
├── argocd/
│   ├── projects/
│   │   └── roboshop-prod.yaml
│   ├── applications/
│   │   └── roboshop-root.yaml
│   └── applicationsets/
│       └── roboshop-clusters.yaml
│
├── helm/
│   └── roboshop/
│       ├── cart/
│       ├── catalogue/
│       ├── user/
│       └── orders/
│
├── environments/
│   ├── dev/
│   ├── qa/
│   └── prod/
│
├── platform/
│   ├── ingress/
│   ├── monitoring/
│   └── external-secrets/
│
└── clusters/
    ├── dev/
    ├── qa/
    └── prod/
```

---

# 160. GitOps Root Application

```text
root Application
       |
       +--> platform Applications
       |
       +--> RoboShop Applications
       |
       +--> environment ApplicationSets
```

This provides bootstrap orchestration.

---

# 161. Production ApplicationSet Model

```text
ApplicationSet
 |
 +--> cart-dev
 +--> cart-qa
 +--> cart-prod
```

or:

```text
ApplicationSet
 |
 +--> roboshop-dev-cluster
 +--> roboshop-qa-cluster
 +--> roboshop-prod-cluster
```

---

# 162. Centralized Multi-Cluster Model

```text
                     GitOps Repository
                            |
                            v
                       Argo CD
                            |
          +-----------------+-----------------+
          |                 |                 |
          v                 v                 v
       EKS DEV           EKS QA           EKS PROD
          |                 |                 |
      Namespace          Namespace         Namespace
          |                 |                 |
      RoboShop           RoboShop          RoboShop
```

---

# 163. Production Blast Radius

Consider:

```text
Application
Cluster
AWS Account
Region
Argo CD instance
Git repository
```

A failure should not unnecessarily affect every environment.

---

# 164. Blast Radius Controls

Use:

```text
separate AWS accounts
separate clusters
AppProjects
RBAC
protected branches
environment isolation
separate credentials
```

---

# 165. Production Git Repository Failure

If Git becomes unavailable:

```text
existing workloads continue
```

but:

```text
new changes wait
```

Operational teams should understand this before treating Git as a runtime dependency.

---

# 166. Argo CD Failure

If Argo CD becomes unavailable:

```text
existing Kubernetes workloads continue
```

but:

```text
reconciliation stops
```

This means HA is important for deployment control, not because Argo CD is directly serving application traffic.

---

# 167. Kubernetes Control Plane Failure

If EKS API becomes unavailable:

```text
new changes cannot be applied
```

Existing Pods may continue running depending on the failure.

The AWS/EKS control-plane availability model should be part of the platform design.

---

# 168. Node Failure

If one worker node fails:

```text
Pods may be recreated on other nodes
```

provided:

```text
capacity exists
scheduling constraints allow placement
replicas exist
```

---

# 169. AZ Failure

A resilient application should have:

```text
multiple replicas
multi-AZ nodes
topology-aware scheduling
```

for workloads requiring high availability.

---

# 170. Region Failure

Multi-region requires:

```text
another EKS cluster
artifact availability
GitOps availability
DNS/failover
data replication
secret strategy
observability
```

GitOps is one part of the DR architecture, not the entire solution.

---

# 171. Production DR Sequence

```text
Region failure
      |
      v
Activate DR region
      |
      v
Infrastructure available
      |
      v
Argo CD available
      |
      v
Sync applications
      |
      v
Secrets available
      |
      v
Data available
      |
      v
Traffic switched
```

---

# 172. RTO and RPO

RTO:

```text
how quickly service must be restored
```

RPO:

```text
how much data loss is acceptable
```

GitOps improves configuration recovery, but database RPO requires database-specific architecture.

---

# 173. DR Testing

Do not claim DR readiness without testing:

```text
restore
bootstrap
sync
traffic
secrets
data
rollback
```

A quarterly or organization-defined DR exercise can validate the design.

---

# 174. Production Upgrade Strategy

Upgrade:

```text
Argo CD
EKS
Kubernetes
AWS Load Balancer Controller
Helm charts
external operators
```

through controlled processes.

---

# 175. Argo CD Upgrade

Before upgrade:

```text
review compatibility
backup/recovery
read release notes
test in non-production
validate CRDs
```

Then:

```text
upgrade
verify components
verify Applications
verify ApplicationSets
```

---

# 176. EKS Upgrade

Typical sequence:

```text
test
upgrade control plane
upgrade add-ons
upgrade nodes
validate workloads
```

Exact order and tooling depend on EKS version and AWS recommendations.

---

# 177. Kubernetes Version Compatibility

Before upgrading:

```text
Argo CD
Helm
controllers
CRDs
admission policies
applications
```

must be checked for compatibility.

---

# 178. GitOps Repository Upgrade Strategy

Do not combine unrelated changes:

```text
Kubernetes upgrade
+
application release
+
network policy change
```

in one production change unless necessary.

Smaller changes are easier to troubleshoot.

---

# 179. Production Change Window

For risky platform changes:

```text
change request
approval
maintenance window
backup
rollback plan
execution
validation
```

---

# 180. Production Architecture Review Checklist

Before approving an architecture:

```text
[ ] Git source of truth
[ ] CI/CD separation
[ ] immutable artifacts
[ ] Argo CD HA
[ ] AppProjects
[ ] RBAC
[ ] secret management
[ ] cluster isolation
[ ] multi-AZ
[ ] resource requests
[ ] probes
[ ] HPA/PDB
[ ] NetworkPolicy
[ ] ALB
[ ] observability
[ ] audit
[ ] DR
[ ] rollback
[ ] upgrade strategy
```

---

# 181. Architecture Anti-Patterns

Avoid:

```text
CI directly deploying to prod
shared cluster-admin token
raw secrets in Git
latest image tags
Terraform and Argo owning same resource
one giant Application for everything
manual production edits
single-AZ workload
no readiness probe
no resource requests
no rollback plan
no Git protection
```

---

# 182. Anti-Pattern: Direct kubectl From Jenkins

Traditional:

```text
Jenkins
 |
 +--> kubectl apply
 |
 v
PROD
```

This creates:

```text
CI -> production credentials
```

and weakens GitOps separation.

---

# 183. Better Pattern

```text
Jenkins
 |
 v
ECR
 |
 v
GitOps PR
 |
 v
Argo CD
 |
 v
PROD
```

The cluster credentials stay with the GitOps control plane.

---

# 184. Anti-Pattern: Mutable Production Tags

Bad:

```text
image: cart:latest
```

because the same Git commit may point to different code later.

Prefer:

```text
image: cart:1.7.3
```

or digest pinning.

---

# 185. Anti-Pattern: Shared Production Secrets

Avoid:

```text
one credential
shared by every service
```

Prefer:

```text
service-specific identity
service-specific secret
least privilege
```

---

# 186. Anti-Pattern: One Cluster for Everything

Development and production in one cluster can create:

```text
blast radius
resource contention
security risk
operational coupling
```

Separate clusters/accounts may be more appropriate for production isolation.

---

# 187. Architecture Decision: Namespace vs Cluster

Use namespaces when:

```text
teams share trust boundary
```

Use separate clusters when stronger isolation is required.

Use separate AWS accounts when stronger cloud-level isolation is required.

---

# 188. Architecture Decision: Central vs Distributed Argo CD

Centralized:

```text
simpler governance
```

Distributed:

```text
stronger isolation
```

Hybrid:

```text
balance
```

The decision should be driven by:

```text
scale
security
compliance
team structure
failure domains
```

---

# 189. Architecture Decision: Helm vs Kustomize

Helm is strong for:

```text
reusable application packaging
templating
versioned charts
```

Kustomize is strong for:

```text
native Kubernetes overlays
environment patches
```

Organizations may use either or both.

---

# 190. Architecture Decision: App of Apps vs ApplicationSet

Use App of Apps when:

```text
hierarchical bootstrapping
explicit child Applications
platform composition
```

Use ApplicationSet when:

```text
dynamic generation
many environments
many clusters
fleet management
```

They can coexist.

---

# 191. Architecture Decision: One Repo vs Multiple Repos

One GitOps repository:

```text
central governance
simple discovery
```

Multiple repositories:

```text
team autonomy
access boundaries
independent lifecycle
```

Choose based on scale and ownership.

---

# 192. Architecture Decision: One Argo CD vs Multiple

One:

```text
central governance
```

Multiple:

```text
isolation
regional autonomy
```

Do not create multiple instances solely because it "sounds enterprise."

---

# 193. Architecture Decision: Automated Production Sync

Automated sync provides:

```text
fast delivery
continuous reconciliation
```

Manual/approval-controlled sync provides:

```text
stronger change control
```

The correct strategy depends on:

```text
risk
compliance
service criticality
deployment maturity
```

---

# 194. Enterprise Reference Architecture

```text
                         USERS
                           |
                           v
                       Route 53
                           |
                           v
                    AWS Load Balancer
                           |
                           v
                    EKS Workload
                           |
                     +-----+-----+
                     |           |
                     v           v
                  Service      Pods
                                  |
                       +----------+----------+
                       |                     |
                       v                     v
                    Database              Redis
```

Deployment plane:

```text
Developer
   |
   v
Application Git
   |
   v
CI
   |
   v
ECR
   |
   v
GitOps Git
   |
   v
Argo CD
   |
   v
EKS
```

---

# 195. Enterprise Multi-Cluster Reference

```text
                         GitOps Git
                             |
                             v
                       Central Argo CD
                             |
          +------------------+------------------+
          |                  |                  |
          v                  v                  v
       DEV EKS             QA EKS            PROD EKS
          |                  |                  |
        ALB                ALB                ALB
          |                  |                  |
      Services           Services           Services
          |                  |                  |
        Pods               Pods               Pods
```

---

# 196. Enterprise Multi-Account Reference

```text
                 AWS Organization
                        |
        +---------------+---------------+
        |               |               |
        v               v               v
   Management         NonProd          Prod
     Account           Account         Account
        |               |               |
    Argo CD          Dev/QA EKS       Prod EKS
        |               |               |
        +---------------+---------------+
                        |
                    GitOps Git
```

The exact account model depends on the organization's AWS landing zone.

---

# 197. Enterprise Multi-Region Reference

```text
                     Global DNS
                         |
              +----------+----------+
              |                     |
              v                     v
         Region A                Region B
          EKS                     EKS
           |                       |
          ALB                     ALB
           |                       |
       Applications           Applications
```

GitOps keeps desired configuration consistent.

Data replication and traffic failover are separate concerns.

---

# 198. Production Architecture: Full RoboShop

```text
                              Users
                                |
                                v
                             Route53
                                |
                                v
                          AWS ALB Ingress
                                |
                                v
                         Frontend Service
                                |
                                v
                           Frontend Pod
                                |
        +-----------------------+-----------------------+
        |           |            |           |          |
        v           v            v           v          v
      User      Catalogue      Cart        Orders    Payment
        |           |            |           |          |
        |           |            v           |          |
        |           |          Redis         |          |
        |           |                        |          |
        +-----------+------------------------+----------+
                                |
                         Notification/Shipping
```

Deployment architecture:

```text
Application Git
      |
      v
Jenkins/GitHub Actions
      |
      +--> SonarQube
      +--> Trivy
      +--> Veracode
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
   EKS PROD
```

---

# 199. Full Platform Architecture

```text
AWS
|
+-- VPC
|   +-- Public Subnets
|   |    +-- ALB
|   |
|   +-- Private Subnets
|        +-- EKS Nodes
|
+-- EKS
|   |
|   +-- Argo CD
|   +-- AWS LB Controller
|   +-- External Secrets
|   +-- Monitoring
|   +-- RoboShop
|
+-- ECR
+-- RDS
+-- Redis
+-- Secrets Manager
+-- Route53
+-- ACM
```

---

# 200. Full GitOps Control Flow

```text
1. Developer changes code
2. Push to application Git
3. CI starts
4. Tests execute
5. SonarQube analysis
6. Trivy scans image
7. Veracode security checks
8. Image pushed to ECR
9. GitOps image reference updated
10. Pull Request created
11. Review and approval
12. Git merge
13. Argo CD refresh
14. Desired manifests rendered
15. Diff calculated
16. Sync starts
17. Kubernetes API receives changes
18. Controllers create/update resources
19. Pods start
20. Readiness passes
21. Service receives endpoints
22. ALB target becomes healthy
23. Users receive traffic
24. Prometheus/Grafana/ELK observe workload
25. Argo CD continuously reconciles
```

---

# 201. Production Control Loops

There are several independent control loops:

```text
GitOps:
Git → Argo CD → Kubernetes

Kubernetes:
Desired resources → Controllers → Actual resources

HPA:
Metrics → HPA → Replica count

ALB Controller:
Ingress → AWS resources

External Secrets:
Secret store → Kubernetes Secret

Monitoring:
Workload → Metrics/Logs → Alerting
```

The architecture must ensure these controllers do not conflict.

---

# 202. Controller Ownership Is Critical

For every field ask:

```text
Who owns this field?
```

Examples:

```text
Deployment replicas -> HPA
Ingress AWS resources -> ALB Controller
Secret data -> External Secrets
Application manifests -> Argo CD
Infrastructure -> Terraform
```

GitOps maturity is partly about defining ownership correctly.

---

# 203. Desired State Hierarchy

A useful mental model:

```text
Business intent
      |
      v
Git configuration
      |
      v
Argo desired state
      |
      v
Kubernetes API objects
      |
      v
Controller-created runtime state
```

Not every runtime field should be manually controlled.

---

# 204. Production Architecture Principle: Minimize Shared State

Reduce:

```text
shared credentials
shared namespaces
shared cluster-admin
shared repositories
shared mutable artifacts
```

Increase:

```text
clear ownership
least privilege
immutable artifacts
isolated environments
```

---

# 205. Production Architecture Principle: Automate Recovery

Good GitOps systems can rebuild:

```text
namespace
Deployment
Service
Ingress
HPA
PDB
configuration
platform resources
```

from Git.

---

# 206. Production Architecture Principle: Make Recovery Observable

After recovery verify:

```text
Argo CD
Kubernetes
Pods
Services
ALB
Metrics
Logs
business functionality
```

---

# 207. Production Architecture Principle: Separate Control Plane and Data Plane

Control plane:

```text
Git
CI
Argo CD
Terraform
```

Data plane:

```text
EKS workloads
ALB
databases
```

A control-plane failure should not unnecessarily stop running business traffic.

---

# 208. Production Architecture Principle: Git Is Not a Runtime Dependency

The application should not call Git to serve requests.

Git provides:

```text
desired configuration
```

Argo CD provides:

```text
reconciliation
```

Kubernetes provides:

```text
runtime
```

---

# 209. Production Architecture Principle: Argo CD Is Not the Application Runtime

Argo CD does not serve:

```text
HTTP requests
```

It manages desired state.

Application traffic remains:

```text
ALB → Service → Pod
```

---

# 210. Production Architecture Principle: Kubernetes Is the Runtime

Kubernetes handles:

```text
scheduling
restarts
service discovery
rolling updates
resource management
```

Argo CD handles:

```text
GitOps synchronization
```

---

# 211. Production Architecture Principle: AWS Controllers Bridge Planes

AWS Load Balancer Controller bridges:

```text
Kubernetes Ingress
       |
       v
AWS ALB
```

External Secrets bridges:

```text
AWS Secrets Manager
       |
       v
Kubernetes Secret
```

These integrations must be treated as separate reconciliation systems.

---

# 212. Production Failure Domains

Document failure domains:

```text
Git provider
CI platform
ECR
Argo CD
management cluster
target EKS
AZ
region
AWS service
database
application
```

For each:

```text
impact
detection
mitigation
recovery
```

---

# 213. Failure Domain Matrix

| Failure | Existing Apps | New Deployment | Recovery |
|---|---|---|---|
| Git unavailable | Usually continue | Blocked | Git restored |
| Argo CD unavailable | Continue | Reconciliation blocked | Argo restored |
| ECR unavailable | Existing pulled images may continue | New image pulls affected | ECR restored |
| Target EKS API unavailable | Runtime may continue | Deployment blocked | API restored |
| One node fails | Replicas can recover | Depends on capacity | Node recovery/reschedule |
| One AZ fails | Multi-AZ app can survive | Depends on capacity | AZ recovery |
| Region fails | Region unavailable | Failover needed | DR region |

---

# 214. Production DR Architecture Decision

Ask:

```text
What must survive?
```

Possible answers:

```text
application traffic
configuration
artifacts
secrets
database
DNS
monitoring
GitOps control plane
```

Each needs a specific recovery design.

---

# 215. Production Capacity Planning

Capacity must cover:

```text
normal workload
rolling deployment
node failure
AZ failure
autoscaling
system Pods
Argo CD
monitoring
```

If a cluster runs at 95% capacity, a rolling deployment can fail even when the application itself is healthy.

---

# 216. Headroom

Maintain enough capacity for:

```text
new replicas
rescheduling
system components
unexpected traffic
```

Exact headroom should be based on workload and failure requirements.

---

# 217. Production Cost Architecture

Cost areas:

```text
EKS
EC2
ALB
NAT Gateway
ECR
RDS
Redis
observability
Argo CD management cluster
cross-region traffic
```

GitOps does not automatically reduce infrastructure cost.

It improves:

```text
repeatability
governance
deployment consistency
```

---

# 218. Cost Optimization Without Weakening Reliability

Possible:

```text
right-size requests
use autoscaling
optimize log retention
ECR lifecycle policies
appropriate node types
avoid unnecessary clusters
```

Do not remove redundancy from production purely for cost.

---

# 219. Architecture Review: Security

Ask:

```text
Who can modify Git?
Who can approve PROD?
Who can access Argo CD?
Who can register clusters?
Who can deploy to PROD?
Who can access secrets?
Who can assume AWS roles?
```

Every answer should be explicit.

---

# 220. Architecture Review: Reliability

Ask:

```text
What if Argo CD fails?
What if Git fails?
What if ECR fails?
What if one node fails?
What if one AZ fails?
What if the cluster fails?
What if the region fails?
```

---

# 221. Architecture Review: Operations

Ask:

```text
How is Argo CD upgraded?
How are credentials rotated?
How are rollbacks performed?
How are incidents detected?
How is drift handled?
How is DR tested?
```

---

# 222. Architecture Review: Developer Experience

Ask:

```text
How does a developer deploy?
How do they promote an image?
How do they see sync status?
How do they rollback?
How do they troubleshoot?
```

A good GitOps platform should reduce operational friction.

---

# 223. Developer Experience Flow

```text
Developer
 |
 +--> push code
 |
 +--> CI
 |
 +--> image
 |
 +--> GitOps PR
 |
 +--> review
 |
 +--> merge
 |
 +--> Argo CD
 |
 +--> deployment
```

The developer should not need production `kubectl` access for normal deployments.

---

# 224. Platform Self-Service

A mature platform can provide:

```text
service template
Helm chart
ApplicationSet
namespace
RBAC
observability
CI template
GitOps template
```

Teams consume standardized building blocks.

---

# 225. Golden Path

Example:

```text
New microservice
      |
      v
Repository template
      |
      +--> CI
      +--> Dockerfile
      +--> security
      |
      v
GitOps template
      |
      +--> Helm
      +--> Application
      +--> monitoring
      |
      v
EKS
```

This is a platform-engineering extension of GitOps.

---

# 226. RoboShop Golden Path

For a new service:

```text
Create repo
 |
 v
Dockerfile
 |
 v
Jenkins/GitHub Actions
 |
 v
ECR
 |
 v
Helm chart
 |
 v
GitOps Application
 |
 v
Argo CD
 |
 v
EKS
```

---

# 227. Production GitOps Maturity Levels

## Level 1

```text
manual kubectl
```

## Level 2

```text
Git manifests
```

## Level 3

```text
Argo CD automated sync
```

## Level 4

```text
ApplicationSets
multi-cluster
security
observability
```

## Level 5

```text
platform engineering
progressive delivery
policy as code
automated promotion
DR
```

---

# 228. Senior Engineer Perspective

A senior engineer should think beyond:

```text
How do I deploy this?
```

and ask:

```text
Who owns it?
What happens if it fails?
How do I rollback?
How is it secured?
How is it observed?
How does it scale?
How does it recover?
How do I prove what changed?
```

---

# 229. Production Architecture Checklist

## Git

```text
[ ] protected branches
[ ] reviews
[ ] CODEOWNERS
[ ] audit
[ ] secret scanning
```

## CI

```text
[ ] tests
[ ] SonarQube
[ ] Trivy
[ ] Veracode
[ ] immutable ECR image
```

## Argo CD

```text
[ ] HA
[ ] RBAC
[ ] AppProjects
[ ] secure repositories
[ ] monitoring
[ ] backup/recovery
```

## EKS

```text
[ ] multi-AZ
[ ] capacity
[ ] resource policies
[ ] NetworkPolicy
[ ] Pod security
[ ] workload identity
```

## AWS

```text
[ ] IAM
[ ] VPC
[ ] private nodes
[ ] ALB
[ ] ACM
[ ] Route 53
[ ] ECR
[ ] Secrets Manager
```

---

# 230. Production Architecture Interview Questions

## Q1. Design GitOps for multiple EKS clusters.

### Answer

I would use a centralized Argo CD control plane for appropriate environments, register target EKS clusters with least-privilege access, use AppProjects for security boundaries and ApplicationSets with cluster generators for dynamic multi-cluster deployment. For stronger isolation or regulatory requirements, I would use separate Argo CD instances.

---

## Q2. How would you separate CI and CD?

### Answer

CI builds, tests, scans and publishes immutable images to ECR. CD is represented by a GitOps repository containing the desired image reference and Kubernetes configuration. Argo CD reads that repository and reconciles it into EKS.

---

## Q3. How do you prevent Jenkins from having production cluster credentials?

### Answer

Jenkins should update the GitOps repository rather than directly calling `kubectl` against production. Argo CD, running inside the management/control plane, owns the production cluster credentials.

---

## Q4. How do you design multi-account EKS GitOps?

### Answer

I would typically use separate AWS accounts for environments, a controlled Argo CD management plane, registered target clusters, least-privilege cluster access, AppProjects, protected Git repositories and explicit cluster labels for ApplicationSet targeting.

---

## Q5. What happens if Argo CD goes down?

### Answer

Existing workloads normally continue running because Argo CD is not the application runtime. Reconciliation and new deployment operations are affected until Argo CD is restored.

---

## Q6. What happens if Git goes down?

### Answer

Existing workloads generally continue running. New desired-state changes cannot be retrieved, and deployment reconciliation waits for Git availability.

---

## Q7. How would you design DR for GitOps?

### Answer

I would make infrastructure reproducible with Terraform, keep Git repositories protected and recoverable, make Argo CD bootstrapable, externalize secrets, maintain artifact retention and design independent DR for persistent data. Then I would test the complete restoration path.

---

## Q8. How do Terraform and Argo CD coexist?

### Answer

Terraform manages cloud infrastructure and cluster-level infrastructure. Argo CD manages Kubernetes application desired state. I avoid having both tools own the same Kubernetes resource.

---

## Q9. How do you manage production secrets?

### Answer

I would use a secret manager such as AWS Secrets Manager with an integration such as External Secrets Operator. Git contains references/configuration, not raw secret values.

---

## Q10. How would you secure production Argo CD?

### Answer

Use SSO, least-privilege RBAC, AppProjects, protected repositories, restricted cluster access, TLS, credential rotation, audit logging, network controls and separate production boundaries where necessary.

---

## Q11. How do you manage dozens of EKS clusters?

### Answer

I would use ApplicationSets with cluster generators and labels such as environment, account, region and team. This allows one template to generate Applications dynamically while AppProjects enforce security boundaries.

---

## Q12. When would you choose separate Argo CD instances?

### Answer

When isolation, compliance, regional autonomy or blast-radius requirements justify the additional operational cost.

---

## Q13. Why should image promotion use the same artifact?

### Answer

Rebuilding for every environment means the artifact can differ between environments. Promoting an immutable image provides stronger reproducibility and auditability.

---

## Q14. How do you prevent configuration drift?

### Answer

Git is the desired state, Argo CD continuously compares desired and live state, and automated sync/self-heal can correct unauthorized changes where appropriate. For fields intentionally owned by another controller, I configure explicit ownership handling.

---

## Q15. How do you design ALB integration with GitOps?

### Answer

Git contains the Kubernetes Ingress configuration. Argo CD applies it to EKS. The AWS Load Balancer Controller watches the Ingress and reconciles the corresponding AWS ALB resources.

---

# 231. Senior Scenario: Central Argo CD + Three EKS Clusters

Requirement:

```text
DEV
QA
PROD
```

Architecture:

```text
                     Git
                      |
                      v
                Central Argo CD
                 /     |      \
                v      v       v
              DEV     QA      PROD
              EKS     EKS     EKS
```

Implementation:

```text
ApplicationSet
+
Cluster Generator
+
environment labels
+
AppProject
```

Security:

```text
least privilege
production restrictions
SSO
RBAC
```

---

# 232. Senior Scenario: Two Production Regions

Requirement:

```text
PROD AP-SOUTH-1
PROD US-EAST-1
```

Use:

```text
ApplicationSet
cluster labels:
environment=prod
region=...
```

The same desired application definition can be rendered for both clusters while allowing regional differences.

---

# 233. Senior Scenario: DR Cluster

Primary:

```text
prod-ap
```

DR:

```text
prod-dr
```

GitOps keeps configuration ready.

Activation requires:

```text
infrastructure
secrets
data
DNS
capacity
application sync
```

---

# 234. Senior Scenario: Argo CD Management Cluster Failure

Recovery:

```text
Terraform
 |
 v
new management EKS
 |
 v
install Argo CD
 |
 v
bootstrap root Application
 |
 v
register target clusters
 |
 v
ApplicationSets
 |
 v
reconciliation
```

This is why bootstrap configuration must be version-controlled and tested.

---

# 235. Senior Scenario: Production Cluster Compromise

A serious security incident may require:

```text
isolate cluster
rotate credentials
review Git
review Argo audit
review AWS IAM
rebuild cluster
bootstrap from trusted Git
```

GitOps makes rebuild easier, but does not automatically make a compromised Git repository trustworthy.

---

# 236. Git Trust Model

GitOps security depends on:

```text
Git integrity
+
Argo CD integrity
+
cluster integrity
```

If an attacker controls the GitOps repository, they can potentially control desired state.

Therefore protect:

```text
repository
branches
reviews
credentials
```

---

# 237. Supply Chain Architecture

```text
Source
 |
 v
CI
 |
 +--> SAST
 +--> SCA
 +--> Secret Scan
 |
 v
Container
 |
 +--> Trivy
 |
 v
ECR
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

Each stage provides a security boundary.

---

# 238. Supply Chain Integrity

For stronger assurance:

```text
immutable artifact
image digest
signed artifact where required
protected Git
review
policy
```

The exact implementation can use organizationally approved tooling.

---

# 239. Production Policy Architecture

Policies can enforce:

```text
no privileged containers
no latest
resources required
probes required
approved registries only
approved namespaces
approved ingress exposure
```

Policy can be applied before runtime.

---

# 240. Policy Failure Handling

If a deployment is rejected:

```text
Argo CD sync
      |
      v
Admission
      |
      X
Policy
```

Do not bypass policy immediately.

Determine:

```text
is the policy correct?
is the workload compliant?
is an exception justified?
```

---

# 241. Production Exception Process

Exceptions should be:

```text
documented
time-bound
approved
audited
```

Avoid permanent policy bypasses.

---

# 242. Architecture Documentation

Every production service should document:

```text
repository
Application
Project
cluster
namespace
image
dependencies
secrets
ALB
SLO
rollback
owner
```

---

# 243. Production Service Catalog

A platform can maintain:

```text
service
team
repository
GitOps path
cluster
namespace
dashboard
logs
on-call
```

This dramatically improves incident response.

---

# 244. Operational Readiness Review

Before production:

```text
[ ] health probes
[ ] resources
[ ] HPA
[ ] PDB
[ ] securityContext
[ ] NetworkPolicy
[ ] secret integration
[ ] logging
[ ] metrics
[ ] alerts
[ ] rollback
[ ] DR
```

---

# 245. Deployment Readiness Review

Before merge:

```text
[ ] image exists
[ ] image immutable
[ ] Helm renders
[ ] policies pass
[ ] values correct
[ ] target cluster correct
[ ] target namespace correct
[ ] approvals complete
```

---

# 246. Post-Deployment Validation

After Argo sync:

```bash
argocd app get <app>
kubectl rollout status deployment/<name> -n <namespace>
kubectl get pods -n <namespace>
kubectl get endpointslice -n <namespace>
```

Then verify:

```text
ALB
metrics
logs
business endpoint
```

---

# 247. Continuous Reconciliation

The production system should converge:

```text
Git desired state
       |
       v
Argo CD
       |
       v
Kubernetes
       |
       v
actual state
       |
       +---- difference ----+
                            |
                            v
                         reconcile
```

This is the core operational loop.

---

# 248. Production Architecture Summary

A mature GitOps architecture has:

```text
protected Git
      +
secure CI
      +
immutable ECR artifacts
      +
Argo CD
      +
AppProjects/RBAC
      +
EKS
      +
AWS controllers
      +
observability
      +
DR
```

---

# 249. Final Reference Architecture

```text
                         DEVELOPER
                             |
                             v
                      APPLICATION GIT
                             |
                             v
                 JENKINS / GITHUB ACTIONS
                             |
              +--------------+--------------+
              |              |              |
           SonarQube       Trivy        Veracode
              |              |              |
              +--------------+--------------+
                             |
                             v
                            ECR
                             |
                             v
                       GITOPS GIT
                             |
                         Protected PR
                             |
                             v
                          ARGO CD
                             |
             +---------------+---------------+
             |               |               |
             v               v               v
          DEV EKS          QA EKS          PROD EKS
             |               |               |
            ALB             ALB             ALB
             |               |               |
        Applications    Applications    Applications
             |               |               |
        Prometheus      Prometheus      Prometheus
        Grafana         Grafana         Grafana
        ELK             ELK             ELK
```

---

# 250. Final Production Mental Model

Remember these ownership boundaries:

```text
Terraform
    ↓
Cloud infrastructure

CI
    ↓
Build + Test + Security + Artifact

GitOps Repository
    ↓
Desired application state

Argo CD
    ↓
Reconciliation

Kubernetes
    ↓
Runtime orchestration

AWS Controllers
    ↓
AWS-integrated resources

Prometheus/Grafana/ELK
    ↓
Observability
```

The complete production architecture is therefore:

```text
Developer
   ↓
Git
   ↓
CI
   ↓
Security
   ↓
ECR
   ↓
GitOps
   ↓
Argo CD
   ↓
EKS
   ↓
ALB
   ↓
Users

                 ↑
                 |
          Reconciliation

                 ↑
                 |
        Prometheus / Grafana / ELK
```

This architecture provides the foundation for the next GitOps topics: project implementation, production operations, and the complete RoboShop GitOps project.
