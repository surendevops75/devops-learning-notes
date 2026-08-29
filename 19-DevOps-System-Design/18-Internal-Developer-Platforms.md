# 19-DevOps-System-Design
# 18-Internal-Developer-Platforms

## 1. Purpose

An Internal Developer Platform (IDP) is a productized collection of
self-service capabilities that allows engineering teams to create, deploy,
operate, secure and understand software without manually coordinating every
underlying infrastructure operation.

A useful abstraction is:

```text
Developer
   |
   v
Internal Developer Platform
   |
+--+-------------------------------+
|                                  |
Developer Portal              Platform APIs
|                                  |
Golden Paths                 Controllers / Automation
|                                  |
+----------------+-----------------+
                 |
        Infrastructure APIs
                 |
       AWS / Kubernetes / SaaS
```

The IDP should reduce cognitive load without hiding operational reality.

---

# PART I — IDP FOUNDATIONS

## 2. What an IDP Is

An IDP combines:

```text
self-service
templates
platform APIs
automation
guardrails
observability
documentation
ownership
```

It is not simply a dashboard.

---

## 3. What an IDP Is Not

An IDP is not necessarily:

```text
Kubernetes
Backstage
Terraform
Argo CD
a ticketing system
```

Those may be components.

The IDP is the complete developer-facing operating model.

---

## 4. Primary Goal

The primary goal is:

```text
make the correct production path easy
```

while maintaining:

```text
security
reliability
governance
developer autonomy
```

---

# PART II — DEVELOPER JOURNEY

## 5. End-to-End Journey

```text
Idea
 |
Repository
 |
Template
 |
Local Development
 |
CI
 |
Artifact
 |
Environment
 |
Deployment
 |
Observability
 |
Production
 |
Incident
```

The IDP should support the entire lifecycle.

---

## 6. Time to First Deployment

A useful IDP metric is:

```text
time from service creation
to first successful deployment
```

Reducing this without reducing safety is a major platform outcome.

---

# PART III — PERSONAS

## 7. Application Developer

Needs:

```text
service creation
CI
deployment
logs
metrics
documentation
```

---

## 8. Platform Engineer

Needs:

```text
templates
policies
controllers
platform health
adoption metrics
```

---

## 9. SRE

Needs:

```text
SLO
alerts
dependency graphs
incident context
rollback
```

---

## 10. Security Engineer

Needs:

```text
policy
audit
vulnerability status
identity
compliance
```

---

# PART IV — REFERENCE IDP ARCHITECTURE

## 11. Enterprise Reference

```text
                         Developers
                              |
                              v
                    +-------------------+
                    | Developer Portal  |
                    +-------------------+
                              |
                    +-------------------+
                    | Platform API      |
                    +-------------------+
                              |
       +----------------------+----------------------+
       |                      |                      |
 Service Catalog       Template Engine       Workflow Engine
       |                      |                      |
       +----------------------+----------------------+
                              |
                       Policy / Security
                              |
                    +---------+---------+
                    |                   |
              GitOps / CI         Infrastructure
                    |                   |
                  EKS                  AWS
                    |                   |
              Applications       VPC/IAM/RDS/S3
                    |
             Observability
                    |
       Metrics / Logs / Traces / SLO
```

---

# PART V — PORTAL

## 12. Developer Portal

A portal can expose:

```text
catalog
templates
documentation
deployments
SLOs
dependencies
cost
incidents
```

---

## 13. Portal Design Principle

The portal should answer:

```text
What do I own?
What is running?
Where is it running?
Is it healthy?
What depends on it?
How do I deploy?
How do I recover?
```

---

# PART VI — SOFTWARE CATALOG

## 14. Catalog

A catalog record may contain:

```yaml
name: checkout
owner: payments-team
lifecycle: production
repository: checkout
runtime: java
environment:
  - staging
  - production
```

---

## 15. Catalog Relationships

Represent:

```text
service -> owned by -> team
service -> depends on -> database
service -> calls -> payment-api
service -> deployed to -> cluster
service -> monitored by -> dashboard
```

---

# PART VII — OWNERSHIP

## 16. Ownership

Every production service should have:

```text
primary team
on-call
repository
runbook
dashboard
SLO
```

---

## 17. Ownership During Incident

The portal should make it possible to identify:

```text
service
 |
owner
 |
on-call
 |
runbook
```

quickly.

---

# PART VIII — TEMPLATES

## 18. Service Template

A template can generate:

```text
repository
README
Dockerfile
CI
Helm
Kubernetes
GitOps
monitoring
alerts
catalog metadata
```

---

## 19. Runtime Templates

Provide standard templates for:

```text
Java
Node.js
Python
Go
```

where appropriate.

---

# PART IX — TEMPLATE ENGINEERING

## 20. Template Versioning

Use explicit versions:

```text
service-template-v1
service-template-v2
```

---

## 21. Template Updates

A platform should provide:

```text
migration guidance
automated update
deprecation notices
```

rather than silently breaking consumers.

---

# PART X — GOLDEN PATHS

## 22. Golden Path

Example:

```text
Create Service
 |
Repository
 |
CI
 |
Security Scan
 |
Container
 |
Registry
 |
GitOps
 |
EKS
 |
Observability
```

---

## 23. Golden Path Requirements

A good path is:

```text
supported
documented
secure
observable
versioned
maintained
```

---

# PART XI — SELF-SERVICE

## 24. Self-Service Workflow

Developer:

```text
select template
 |
provide configuration
 |
submit
 |
validation
 |
automation
 |
resource ready
```

---

## 25. Self-Service Examples

```text
create service
create database
create queue
create bucket
create environment
create DNS
request certificate
enable feature
```

---

# PART XII — PLATFORM API

## 26. API Model

Example:

```text
POST /services
POST /environments
POST /databases
GET  /operations/{id}
```

The exact implementation is less important than a stable contract.

---

## 27. API Validation

Validate:

```text
identity
authorization
schema
quota
policy
ownership
```

before provisioning.

---

# PART XIII — ASYNCHRONOUS OPERATIONS

## 28. Why Async?

Infrastructure operations can take:

```text
seconds
minutes
```

Do not require a client to maintain a long synchronous request.

---

## 29. Operation Model

```text
request
 |
operation ID
 |
Pending
 |
Running
 |
Ready
```

or:

```text
Failed
```

---

# PART XIV — IDEMPOTENCY

## 30. Idempotent Requests

Repeated requests should not accidentally create:

```text
duplicate databases
duplicate DNS
duplicate queues
```

Use idempotency keys or deterministic resource identities where appropriate.

---

# PART XV — WORKFLOW ENGINE

## 31. Workflow

```text
request
 |
validate
 |
policy
 |
provision
 |
configure
 |
register
 |
observe
 |
complete
```

---

# PART XVI — CONTROLLERS

## 32. Reconciliation

```text
desired
 |
controller
 |
actual
 |
reconcile
```

Controllers should converge toward desired state.

---

# PART XVII — FAILURE HANDLING

## 33. Partial Provisioning

Example:

```text
network created
IAM created
database failed
```

The platform must know whether to:

```text
retry
rollback
pause
```

---

# PART XVIII — STATUS

## 34. Status Contract

Expose:

```text
phase
reason
message
last update
operation ID
```

---

# PART XIX — ERROR EXPERIENCE

## 35. Good Error

Bad:

```text
Provisioning failed.
```

Better:

```text
Database creation failed because the selected subnet group has no
available capacity. Choose an approved subnet group or retry after capacity
is restored.
```

---

# PART XX — GITOPS

## 36. GitOps Integration

A typical IDP workflow:

```text
Developer
 |
Portal
 |
Git repository
 |
Pull Request
 |
Argo CD
 |
EKS
```

---

## 37. Desired State

The platform should make desired state visible and reviewable.

---

# PART XXI — CI

## 38. Standard CI

```text
checkout
 |
compile
 |
test
 |
security
 |
build
 |
SBOM
 |
sign
 |
publish
```

---

# PART XXII — CD

## 39. Standard CD

```text
artifact
 |
GitOps
 |
deployment
 |
health
 |
progressive delivery
 |
promotion
```

---

# PART XXIII — ARTIFACTS

## 40. Immutable Artifacts

Prefer:

```text
image@sha256:digest
```

over:

```text
latest
```

---

# PART XXIV — ENVIRONMENT CREATION

## 41. Environment Self-Service

```text
request
 |
policy
 |
namespace/account
 |
configuration
 |
observability
 |
ready
```

---

# PART XXV — PREVIEW ENVIRONMENTS

## 42. Pull Request Environment

```text
PR
 |
environment
 |
tests
 |
review
 |
destroy
```

---

## 43. TTL

Every temporary environment should ideally have:

```text
owner
creation time
expiry
cleanup
```

---

# PART XXVI — KUBERNETES IDP

## 44. Kubernetes Capabilities

Provide standard:

```text
Deployment
Service
Ingress/Gateway
Config
Secret integration
HPA
PDB
ServiceMonitor
```

---

# PART XXVII — EKS

## 45. EKS Platform

Standardize:

```text
cluster
network
IAM
ingress
DNS
certificates
secrets
autoscaling
observability
GitOps
```

---

# PART XXVIII — NAMESPACE PROVISIONING

## 46. Namespace

Self-service can create:

```text
namespace
RBAC
quota
network policy
resource limits
observability
```

---

# PART XXIX — MULTI-TENANCY

## 47. Tenant Isolation

Use appropriate combinations of:

```text
namespace
RBAC
network policy
quotas
cluster
account
```

---

# PART XXX — ACCOUNT PROVISIONING

## 48. AWS Account

Enterprise IDP can integrate with account-vending workflows.

```text
request
 |
approval
 |
account
 |
baseline
 |
security
 |
logging
 |
network
 |
ready
```

---

# PART XXXI — NETWORKING

## 49. Network Self-Service

Expose high-level intent:

```text
private application
database access
internal DNS
public ingress
```

rather than low-level route-table operations.

---

# PART XXXII — IAM

## 50. Identity

Use:

```text
workload identity
IAM roles
RBAC
temporary access
```

---

# PART XXXIII — SECRETS

## 51. Secret Workflow

```text
request
 |
secret store
 |
identity
 |
application
```

Avoid storing plaintext secrets in:

```text
Git
container images
logs
```

---

# PART XXXIV — DATABASE PLATFORM

## 52. Database Self-Service

Developer may request:

```text
engine
size
backup
retention
availability
```

Platform enforces organizational policy.

---

# PART XXXV — DATABASE GUARDRAILS

## 53. Defaults

Examples:

```text
encryption
backup
private networking
monitoring
```

---

# PART XXXVI — QUEUE PLATFORM

## 54. Queue

Self-service can provide:

```text
queue
DLQ
encryption
retention
alarms
```

---

# PART XXXVII — STORAGE PLATFORM

## 55. Object Storage

Provide:

```text
bucket
encryption
versioning
retention
access policy
logging
```

---

# PART XXXVIII — DNS PLATFORM

## 56. DNS

Provide controlled:

```text
internal hostname
public hostname
certificate
routing
```

---

# PART XXXIX — CERTIFICATE PLATFORM

## 57. TLS

Automate:

```text
request
validation
association
renewal
```

---

# PART XL — OBSERVABILITY

## 58. Default Observability

Every standard service should receive:

```text
metrics
logs
traces
dashboard
alerts
```

as appropriate.

---

# PART XLI — SERVICE METRICS

## 59. Golden Signals

Use:

```text
traffic
errors
latency
saturation
```

---

# PART XLII — LOGGING

## 60. Structured Logs

Standard fields:

```text
timestamp
service
version
environment
trace_id
request_id
```

Avoid sensitive data.

---

# PART XLIII — TRACING

## 61. Distributed Tracing

```text
frontend
 |
API
 |
service
 |
database
```

Trace propagation should be standardized.

---

# PART XLIV — SLO

## 62. Service SLO

IDP can provide:

```text
availability
latency
error budget
```

templates.

---

# PART XLV — ALERTS

## 63. Standard Alerts

Examples:

```text
high error rate
high latency
resource saturation
pod crash
queue backlog
```

---

# PART XLVI — DASHBOARDS

## 64. Standard Dashboard

Show:

```text
traffic
errors
latency
resources
dependencies
deployment
```

---

# PART XLVII — SECURITY

## 65. Security-by-Default

Standard services should receive:

```text
image scanning
secret scanning
dependency scanning
IAM controls
network controls
```

---

# PART XLVIII — POLICY AS CODE

## 66. Policy

```text
resource
 |
policy
 |
allow / deny
```

---

## 67. Policy Examples

```text
deny public database
deny privileged pod
require encryption
require approved registry
require owner
```

---

# PART XLIX — ADMISSION

## 68. Kubernetes Admission

Validate:

```text
security context
resource requests
image registry
labels
```

---

# PART L — IDP SECURITY

## 69. Portal Security

Portal requires:

```text
authentication
authorization
audit
rate limiting
```

---

# PART LI — ROLE MODEL

## 70. Roles

Possible:

```text
developer
service owner
platform engineer
security
administrator
auditor
```

---

# PART LII — LEAST PRIVILEGE

## 71. Principle

The developer should not receive broad:

```text
AWS AdministratorAccess
```

just to create an application.

The platform performs privileged operations through controlled identities.

---

# PART LIII — DELEGATED ACCESS

## 72. Platform Identity

```text
Developer
 |
limited request
 |
Platform
 |
scoped privileged role
 |
AWS
```

---

# PART LIV — AUDIT

## 73. Audit Trail

Record:

```text
requester
resource
action
policy result
approver
automation
result
```

---

# PART LV — APPROVALS

## 74. When Approval Is Appropriate

Use approvals for high-risk operations such as:

```text
production account
high privilege
destructive database action
security exception
```

Do not require approval for every routine operation.

---

# PART LVI — POLICY EXCEPTIONS

## 75. Exception Workflow

```text
request
 |
reason
 |
risk
 |
approval
 |
expiry
 |
audit
```

---

# PART LVII — COST

## 76. Cost Visibility

Show:

```text
service
team
environment
resource
```

cost where possible.

---

# PART LVIII — COST GUARDRAILS

## 77. Example

Platform may prevent or flag:

```text
oversized database
unbounded node group
long-lived preview environment
```

---

# PART LIX — QUOTAS

## 78. Team Quotas

Use:

```text
CPU
memory
storage
accounts
databases
CI minutes
```

where appropriate.

---

# PART LX — NOISY NEIGHBOR

## 79. Isolation

One team should not consume all shared:

```text
CI
cluster
observability
```

capacity.

---

# PART LXI — PLATFORM SLO

## 80. IDP SLO

Define reliability for:

```text
portal
API
provisioning
GitOps
templates
```

---

# PART LXII — IDP ERROR BUDGET

## 81. Error Budget

If provisioning frequently fails:

```text
stop adding features
 |
improve reliability
```

---

# PART LXIII — PLATFORM OBSERVABILITY

## 82. Platform Metrics

Track:

```text
API latency
API errors
workflow duration
provisioning success
queue depth
controller errors
```

---

# PART LXIV — DEVELOPER EXPERIENCE METRICS

## 83. Metrics

Track:

```text
time to first deployment
time to production
self-service success
platform adoption
support tickets
developer satisfaction
```

---

# PART LXV — ADOPTION

## 84. Adoption

Measure:

```text
services using golden path
services using standard templates
services with observability
services with SLOs
```

---

# PART LXVI — PLATFORM PRODUCT MANAGEMENT

## 85. Roadmap

Prioritize:

```text
developer pain
business impact
risk reduction
reliability
```

---

# PART LXVII — FEEDBACK

## 86. Feedback Loops

Provide:

```text
surveys
support
office hours
issue tracking
usage analytics
```

---

# PART LXVIII — PLATFORM DESIGN

## 87. Product Thinking

Ask:

```text
Who uses this?
What problem does it solve?
How often?
What is the success metric?
What is the failure mode?
```

---

# PART LXIX — BUILD VS BUY

## 88. Decision

Consider:

```text
build
buy
open source
managed service
```

Evaluate:

```text
cost
security
maintenance
integration
skills
```

---

# PART LXX — BACKSTAGE-STYLE PORTAL

## 89. Portal Architecture

Possible components:

```text
catalog
templates
docs
plugins
authentication
permissions
```

---

# PART LXXI — PORTAL PLUGINS

## 90. Plugins

Useful integrations:

```text
Git
CI
Kubernetes
cloud
incident management
observability
cost
```

---

# PART LXXII — PORTAL FAILURE

## 91. Resilience

Portal outage should not normally stop:

```text
running applications
```

---

# PART LXXIII — API FAILURE

## 92. API

Existing workloads should not become unavailable because the provisioning API
is down.

New operations may be delayed.

---

# PART LXXIV — GIT FAILURE

## 93. Git

Running applications should continue even if Git is temporarily unavailable.

---

# PART LXXV — GITOPS FAILURE

## 94. GitOps

Running workloads should generally continue when the GitOps control plane is
temporarily unavailable.

---

# PART LXXVI — CI FAILURE

## 95. CI

CI outage should affect delivery, not necessarily runtime.

---

# PART LXXVII — REGISTRY FAILURE

## 96. Registry

Existing running containers should continue operating.

New deployments may be affected.

---

# PART LXXVIII — CONTROL PLANE

## 97. Management vs Runtime

Separate:

```text
management plane
```

from:

```text
application runtime
```

where practical.

---

# PART LXXIX — PLATFORM BLAST RADIUS

## 98. Shared Platform

A platform failure can affect hundreds of teams.

Use:

```text
rate limits
quotas
isolation
HA
```

---

# PART LXXX — MULTI-REGION IDP

## 99. Regional Architecture

For critical platforms:

```text
Region A
 |
platform
 |
Region B
```

with appropriate state replication.

---

# PART LXXXI — MULTI-CLUSTER

## 100. Fleet

```text
platform
 |
cluster A
cluster B
cluster C
```

---

# PART LXXXII — FLEET MANAGEMENT

## 101. Standardization

Manage:

```text
Kubernetes version
addons
policies
observability
security
```

across the fleet.

---

# PART LXXXIII — PLATFORM UPGRADES

## 102. Upgrade Waves

```text
lab
 |
staging
 |
pilot
 |
5%
 |
25%
 |
100%
```

---

# PART LXXXIV — CLUSTER UPGRADE

## 103. EKS

Validate:

```text
API compatibility
addons
CNI
storage
ingress
workloads
```

---

# PART LXXXV — TEMPLATE UPGRADES

## 104. Service Template

Provide:

```text
new version
migration
automated PR
validation
```

---

# PART LXXXVI — PLATFORM DEPRECATION

## 105. Deprecation

Use:

```text
announce
 |
migrate
 |
deadline
 |
remove
```

---

# PART LXXXVII — ESCAPE HATCHES

## 106. Advanced Workloads

Allow controlled customization for:

```text
special networking
custom scheduling
special compliance
unusual workloads
```

---

# PART LXXXVIII — CUSTOMIZATION

## 107. Safe Extension

Provide:

```text
approved extension points
```

instead of allowing arbitrary modifications everywhere.

---

# PART LXXXIX — PLATFORM CONTRACTS

## 108. Contract

Document:

```text
inputs
outputs
SLO
security
support
limitations
```

---

# PART XC — API VERSIONING

## 109. Versioned API

Avoid breaking every consumer.

Use:

```text
v1
v2
```

with migration paths.

---

# PART XCI — TEMPLATE CONTRACTS

## 110. Contract Testing

Verify templates produce valid:

```text
CI
Kubernetes
GitOps
observability
```

configuration.

---

# PART XCII — PLATFORM TESTING

## 111. Test Layers

```text
unit
 |
integration
 |
contract
 |
security
 |
end-to-end
 |
upgrade
 |
failure
```

---

# PART XCIII — EPHEMERAL PLATFORM TESTING

## 112. Test Account

```text
create
 |
test
 |
validate
 |
destroy
```

---

# PART XCIV — CHAOS

## 113. Failure Tests

Test:

```text
API unavailable
Git unavailable
registry unavailable
controller failure
AWS API throttling
```

---

# PART XCV — RATE LIMITING

## 114. Cloud APIs

Platform automation should handle AWS API throttling with:

```text
backoff
jitter
bounded retry
```

---

# PART XCVI — AWS QUOTAS

## 115. Quotas

Provisioning should understand AWS limits such as:

```text
API limits
resource limits
IP capacity
service quotas
```

---

# PART XCVII — KUBERNETES QUOTAS

## 116. ResourceQuota

Prevent one namespace from consuming unlimited resources.

---

# PART XCVIII — PLATFORM QUEUES

## 117. Async Work

Queue expensive operations:

```text
account creation
environment creation
cluster creation
```

---

# PART XCIX — WORKER SCALING

## 118. Platform Workers

Scale workers based on:

```text
queue depth
processing latency
```

---

# PART C — DEAD LETTER

## 119. Failed Operations

Persistent failures can enter:

```text
dead-letter queue
```

for investigation.

---

# PART CI — RETRY POLICY

## 120. Safe Retry

Retry only operations where repeating the action is safe or idempotent.

---

# PART CII — STATE MANAGEMENT

## 121. Platform State

Protect:

```text
catalog
workflow state
audit
configuration
```

---

# PART CIII — DATABASE

## 122. IDP Database

If the platform has persistent metadata, define:

```text
backup
HA
RPO
RTO
migration
```

---

# PART CIV — DATABASE MIGRATION

## 123. Safe Migration

Use:

```text
expand
migrate
validate
contract
```

---

# PART CV — BACKUP

## 124. Backup

Back up unique platform state.

Reconstruct reproducible infrastructure from:

```text
Git
IaC
templates
```

where possible.

---

# PART CVI — DISASTER RECOVERY

## 125. DR

A complete DR design includes:

```text
code
IaC
configuration
state
artifacts
secrets
runbooks
```

---

# PART CVII — CLEAN ROOM

## 126. Reconstruction

Test whether the platform can be rebuilt in a clean environment.

---

# PART CVIII — PLATFORM RTO

## 127. RTO

Define:

```text
time to restore self-service
```

separately from application runtime recovery.

---

# PART CIX — PLATFORM RPO

## 128. RPO

Identify platform state that cannot be reconstructed.

---

# PART CX — INCIDENT MANAGEMENT

## 129. IDP Incident

Example:

```text
All deployments fail
```

Response:

```text
declare incident
 |
identify common dependency
 |
protect running services
 |
restore platform
 |
validate
 |
resume
```

---

# PART CXI — INCIDENT CONTEXT

## 130. Portal

Portal should surface:

```text
deployment
 |
commit
 |
version
 |
owner
 |
incident
```

---

# PART CXII — SERVICE SCORECARD

## 131. Scorecard

A scorecard can show whether a service meets platform standards:

```text
[✓] owner
[✓] CI
[✓] security scan
[✓] observability
[✓] SLO
[✓] backup
[ ] dependency policy
```

---

# PART CXIII — SCORECARD GOVERNANCE

## 132. Avoid Vanity Metrics

A scorecard should identify meaningful risk, not become a compliance checklist
with no operational value.

---

# PART CXIV — PLATFORM MATURITY

## 133. Maturity Levels

```text
Level 0 -> manual
Level 1 -> automated
Level 2 -> self-service
Level 3 -> standardized
Level 4 -> policy-driven
Level 5 -> optimized
```

---

# PART CXV — LEVEL 0

## 134. Manual

```text
tickets
manual AWS
manual deployment
```

---

# PART CXVI — LEVEL 1

## 135. Automated

Scripts automate repeated operations.

---

# PART CXVII — LEVEL 2

## 136. Self-Service

Developers initiate operations themselves.

---

# PART CXVIII — LEVEL 3

## 137. Standardized

Golden paths become organization-wide standards.

---

# PART CXIX — LEVEL 4

## 138. Policy-Driven

Security and governance become automated guardrails.

---

# PART CXX — LEVEL 5

## 139. Optimized

Platform continuously improves through:

```text
metrics
feedback
automation
```

---

# PART CXXI — IDP COST

## 140. Cost Model

Costs include:

```text
engineering
infrastructure
SaaS
observability
support
```

---

# PART CXXII — PLATFORM ROI

## 141. Value

Measure:

```text
hours saved
faster delivery
fewer incidents
reduced security risk
```

---

# PART CXXIII — DEVELOPER SATISFACTION

## 142. Feedback

Measure:

```text
ease of use
reliability
documentation
speed
```

---

# PART CXXIV — PLATFORM SUPPORT

## 143. Support

Provide:

```text
docs
office hours
chat
runbooks
escalation
```

---

# PART CXXV — DOCUMENTATION

## 144. Docs

Every golden path should explain:

```text
purpose
steps
architecture
troubleshooting
limitations
```

---

# PART CXXVI — TROUBLESHOOTING

## 145. Self-Service Diagnostics

Provide useful information:

```text
deployment status
pod status
recent events
logs
metrics
```

---

# PART CXXVII — DIAGNOSTIC SECURITY

## 146. Sensitive Data

Do not expose:

```text
secrets
tokens
private credentials
sensitive payloads
```

through portal diagnostics.

---

# PART CXXVIII — PLATFORM API OBSERVABILITY

## 147. Correlation ID

Every workflow should ideally have:

```text
request ID
operation ID
trace ID
```

---

# PART CXXIX — DISTRIBUTED WORKFLOW

## 148. Trace

```text
portal
 |
API
 |
workflow
 |
Terraform
 |
AWS
```

Correlate events across components.

---

# PART CXXX — PLATFORM EVENTING

## 149. Events

Examples:

```text
ServiceCreated
EnvironmentReady
DeploymentStarted
DeploymentCompleted
```

---

# PART CXXXI — EVENT CONSUMERS

## 150. Consumers

Events can update:

```text
catalog
notifications
audit
metrics
```

---

# PART CXXXII — EVENT SCHEMA

## 151. Version

Version event schemas to avoid breaking consumers.

---

# PART CXXXIII — NOTIFICATIONS

## 152. Developer Notifications

Notify for:

```text
deployment complete
provisioning failure
security issue
SLO breach
```

---

# PART CXXXIV — CHATOPS

## 153. Controlled Operations

Expose carefully authorized actions:

```text
status
pause
rollback
```

---

# PART CXXXV — PLATFORM SECURITY OPERATIONS

## 154. Security Events

Detect:

```text
privilege escalation
policy bypass
unusual provisioning
```

---

# PART CXXXVI — SUPPLY CHAIN

## 155. Protect Templates

Treat:

```text
templates
pipeline definitions
Helm charts
Terraform modules
```

as production code.

---

# PART CXXXVII — TEMPLATE REVIEW

## 156. Security Review

A malicious template can affect hundreds of services.

Review shared templates carefully.

---

# PART CXXXVIII — PLATFORM CREDENTIALS

## 157. Credential Scope

Platform automation should use narrowly scoped identities.

---

# PART CXXXIX — ROTATION

## 158. Credentials

Automate:

```text
rotation
revocation
replacement
```

where appropriate.

---

# PART CXL — PLATFORM AVAILABILITY

## 159. HA

Critical IDP components may require:

```text
multiple replicas
multiple AZs
managed databases
```

depending on their SLO.

---

# PART CXLI — PLATFORM DEPENDENCY

## 160. Dependency Mapping

Map:

```text
portal
 |
API
 |
workflow
 |
Git
 |
AWS
```

and identify failure domains.

---

# PART CXLII — SHARED DEPENDENCY

## 161. Common Failure

A single:

```text
identity provider
```

failure could block many operations.

Design appropriate fallback and recovery.

---

# PART CXLIII — RATE LIMITING

## 162. Fairness

Rate-limit self-service operations to prevent one automation loop from
overloading the platform.

---

# PART CXLIV — PLATFORM QUEUE FAIRNESS

## 163. Priority

Use priority carefully for:

```text
critical production
standard
development
```

without starving lower-priority work.

---

# PART CXLV — SECURITY AND PROD

## 164. Production Actions

Production destructive operations should have stronger controls than normal
development operations.

---

# PART CXLVI — ENVIRONMENT POLICIES

## 165. Environment

```text
dev -> flexible
staging -> controlled
prod -> strict
```

---

# PART CXLVII — PLATFORM TENANCY

## 166. Tenant Boundary

Define:

```text
identity
resource
network
data
cost
```

boundaries.

---

# PART CXLVIII — CROSS-TEAM DEPENDENCIES

## 167. Service Graph

Expose dependencies so teams can understand change impact.

---

# PART CXLIX — CHANGE IMPACT

## 168. Before Deployment

Identify:

```text
downstream services
database
queues
external APIs
```

---

# PART CL — PROGRESSIVE DELIVERY

## 169. IDP Integration

A standard application template can include:

```text
Argo Rollout
analysis
metrics
rollback
```

---

# PART CLI — FEATURE FLAGS

## 170. Integration

The platform can provide standard feature-flag integration while leaving
business decisions to service teams.

---

# PART CLII — RELEASE SAFETY

## 171. Standard Controls

Every production deployment can require:

```text
health checks
observability
rollback
ownership
```

---

# PART CLIII — PLATFORM POLICY

## 172. Policy Example

```text
production service
 |
must have owner
must have image scan
must have logs
must have metrics
must have rollback
```

---

# PART CLIV — EXCEPTION

## 173. Exception

If a service cannot meet a standard:

```text
document
 |
approve
 |
expire
```

---

# PART CLV — PLATFORM ROADMAP

## 174. Prioritization

Do not build a portal feature simply because another organization has it.

Start from:

```text
developer problem
```

---

# PART CLVI — PRODUCT DISCOVERY

## 175. Discovery

Interview developers about:

```text
pain
waiting
manual steps
failure
confusion
```

---

# PART CLVII — PLATFORM PILOT

## 176. Pilot

Start with:

```text
1–3 teams
```

validate:

```text
workflow
reliability
adoption
```

then expand.

---

# PART CLVIII — MIGRATION

## 177. Existing Services

Migration should be incremental:

```text
inventory
 |
classify
 |
prioritize
 |
migrate
 |
measure
```

---

# PART CLIX — LEGACY

## 178. Legacy Support

Do not force every legacy service into the newest path immediately.

Provide:

```text
migration path
```

---

# PART CLX — PLATFORM ADOPTION STRATEGY

## 179. Adoption

Use:

```text
better experience
```

rather than only:

```text
mandate
```

---

# PART CLXI — PLATFORM PRODUCT-MARKET FIT

## 180. Signal

Strong signals include:

```text
developers voluntarily adopt
teams request features
support burden decreases
```

---

# PART CLXII — ANTI-PATTERNS

## 181. Portal-Only Platform

A beautiful portal over manual operations is not a real IDP.

---

## 182. Ticket-Based Self-Service

```text
click button
 |
ticket
 |
human
```

is not true self-service.

---

## 183. Overly Broad Abstraction

Do not hide all infrastructure details.

---

## 184. No Ownership

A platform without clear ownership becomes orphaned infrastructure.

---

## 185. No SLO

A platform without reliability objectives cannot prioritize reliability.

---

## 186. No Feedback

Without developer feedback, the platform becomes technology-driven.

---

# PART CLXIII — SENIOR SYSTEM DESIGN

## 187. Design IDP for 500 Teams

Reference:

```text
Developer Portal
 |
Platform API
 |
Workflow Engine
 |
Policy
 |
+------------------------+
|                        |
AWS                  Kubernetes
|                        |
Accounts               EKS
VPC                    GitOps
IAM                    Observability
RDS                    Security
S3
```

Need:

```text
multi-tenancy
quotas
self-service
audit
HA
DR
```

---

## 188. Design IDP for 1000 Services

Use:

```text
templates
catalog
GitOps
automated onboarding
standard observability
```

Avoid manually registering each service.

---

## 189. Design Multi-Account IDP

```text
Organization
 |
Account Factory
 |
Baseline
 |
Security
 |
Network
 |
Logging
 |
Application Account
```

---

## 190. Design Multi-Cluster IDP

```text
Portal
 |
platform API
 |
fleet
 |
cluster A
cluster B
cluster C
```

---

## 191. Design Regulated IDP

Add:

```text
policy
approval
audit
evidence
segregation
```

---

## 192. Design High-Availability IDP

Separate:

```text
portal
API
workflow
database
queue
```

and provide appropriate redundancy.

---

## 193. Design IDP During AWS Outage

Running workloads should remain operational when possible.

Self-service operations can:

```text
queue
retry
pause
```

until dependency recovery.

---

## 194. Design IDP During Git Outage

Existing applications continue running.

New GitOps changes may wait.

---

## 195. Design IDP During Argo CD Outage

Existing workloads remain running.

Deployment operations may be paused.

---

# PART CLXIV — INTERVIEW FRAMEWORK

## 196. Senior Answer

When asked:

```text
How would you design an Internal Developer Platform?
```

Answer:

```text
1. Identify developer journeys.
2. Identify platform customers.
3. Define golden paths.
4. Design developer portal.
5. Design service catalog.
6. Design self-service APIs.
7. Design asynchronous workflows.
8. Make operations idempotent.
9. Add policy and security.
10. Integrate CI/CD.
11. Integrate GitOps.
12. Integrate Kubernetes/EKS.
13. Integrate AWS infrastructure.
14. Provide secrets.
15. Provide observability.
16. Provide SLOs.
17. Provide cost visibility.
18. Provide ownership.
19. Design multi-tenancy.
20. Design failure isolation.
21. Design platform SLOs.
22. Design DR.
23. Define escape hatches.
24. Define adoption metrics.
25. Treat the IDP as a product.
```

---

# PART CLXV — PRODUCTION RUNBOOK

## 197. New Service

```text
1. Developer authenticates.
2. Selects approved template.
3. Provides service metadata.
4. Platform validates ownership.
5. Security policies run.
6. Repository is generated.
7. CI is configured.
8. Registry is configured.
9. GitOps configuration is generated.
10. Catalog entry is created.
11. Observability is enabled.
12. Deployment is performed.
13. Health is verified.
14. Developer receives status.
```

---

## 198. New Database

```text
1. Developer requests database.
2. Platform validates team quota.
3. Environment policy is evaluated.
4. Approved module is selected.
5. Encryption is enabled.
6. Private networking is configured.
7. Backup is configured.
8. Monitoring is enabled.
9. Credentials are integrated with secret management.
10. Resource is registered.
11. Connection information is exposed securely.
```

---

## 199. Production Deployment

```text
1. Verify artifact.
2. Verify security.
3. Verify owner.
4. Verify observability.
5. Verify SLO.
6. Verify rollback.
7. Deploy candidate.
8. Validate readiness.
9. Start progressive delivery.
10. Analyze.
11. Promote or rollback.
12. Record evidence.
```

---

## 200. Platform Incident

```text
1. Identify affected platform capability.
2. Determine runtime impact.
3. Stop risky changes.
4. Identify dependency failure.
5. Restore control plane.
6. Validate self-service.
7. Validate GitOps.
8. Validate new deployments.
9. Resume gradually.
10. Document incident.
```

---

# PART CLXVI — 250 PRODUCTION GOLDEN RULES

## 201. Rules 1–50

```text
1. Treat the IDP as a product.
2. Treat developers as customers.
3. Start with developer journeys.
4. Reduce cognitive load.
5. Build golden paths.
6. Make secure paths easy.
7. Provide self-service.
8. Avoid ticket-driven automation.
9. Define platform boundaries.
10. Define ownership.
11. Provide service catalog.
12. Provide templates.
13. Version templates.
14. Provide migration paths.
15. Provide documentation.
16. Provide support.
17. Provide runbooks.
18. Provide escape hatches.
19. Avoid over-abstraction.
20. Standardize common workflows.
21. Automate repetitive tasks.
22. Prefer declarative desired state.
23. Use reconciliation.
24. Make workflows idempotent.
25. Handle partial failures.
26. Handle retries safely.
27. Use asynchronous operations.
28. Provide operation status.
29. Provide actionable errors.
30. Provide correlation IDs.
31. Provide audit trails.
32. Track service owners.
33. Track dependencies.
34. Track SLOs.
35. Track runbooks.
36. Track production environments.
37. Measure adoption.
38. Measure developer experience.
39. Measure time to first deployment.
40. Measure time to production.
41. Measure self-service success.
42. Measure platform reliability.
43. Measure platform availability.
44. Measure API latency.
45. Measure provisioning success.
46. Define platform SLOs.
47. Define platform error budgets.
48. Monitor platform dependencies.
49. Test degraded modes.
50. Treat the platform as production software.
```

## 202. Rules 51–100

```text
51. Standardize CI.
52. Standardize CD.
53. Standardize artifact handling.
54. Standardize GitOps.
55. Standardize observability.
56. Provide progressive delivery.
57. Provide rollback.
58. Use immutable artifacts.
59. Prefer image digests.
60. Protect registries.
61. Protect CI runners.
62. Protect GitOps credentials.
63. Protect platform APIs.
64. Use least privilege.
65. Use workload identity.
66. Avoid broad administrator access.
67. Audit privileged operations.
68. Protect secrets.
69. Encrypt sensitive data.
70. Rotate credentials.
71. Enforce secure defaults.
72. Use policy as code.
73. Use admission controls where appropriate.
74. Prevent public resources by default.
75. Prevent privileged workloads by default.
76. Require resource requests.
77. Provide namespace isolation.
78. Provide RBAC.
79. Provide quotas.
80. Provide network policies.
81. Provide Pod security controls.
82. Provide backup standards.
83. Provide DR standards.
84. Test restores.
85. Separate production and nonproduction.
86. Separate critical failure domains.
87. Use multi-account architecture where appropriate.
88. Avoid centralized blast-radius mistakes.
89. Protect break-glass access.
90. Test emergency access.
91. Audit emergency access.
92. Keep recovery procedures documented.
93. Protect platform state.
94. Protect catalog data.
95. Protect audit records.
96. Protect workflow state.
97. Test clean reconstruction.
98. Test platform recovery.
99. Test shared-service failure.
100. Maintain a platform lifecycle.
```

## 203. Rules 101–150

```text
101. Standardize EKS lifecycle.
102. Test Kubernetes upgrades.
103. Test addon compatibility.
104. Version addons.
105. Automate cluster creation.
106. Automate cluster bootstrap.
107. Automate cluster configuration.
108. Automate cluster observability.
109. Automate cluster security.
110. Automate cluster upgrades.
111. Use GitOps for desired state.
112. Keep Git recoverable.
113. Keep IaC recoverable.
114. Keep artifacts recoverable.
115. Keep runbooks accessible.
116. Separate management from runtime where practical.
117. Do not make CI a runtime dependency.
118. Do not make Git a runtime dependency.
119. Do not make portal availability a runtime dependency.
120. Do not make registry availability a runtime dependency for running
     workloads.
121. Design degraded modes.
122. Protect shared services.
123. Use rate limits.
124. Use quotas.
125. Prevent noisy neighbors.
126. Protect CI capacity.
127. Protect observability capacity.
128. Protect API capacity.
129. Protect workflow queues.
130. Use exponential backoff.
131. Use jitter.
132. Bound retries.
133. Use dead-letter handling.
134. Detect stalled operations.
135. Detect drift.
136. Reconcile safely.
137. Avoid blindly overwriting manual resources.
138. Document exceptions.
139. Give exceptions an expiry.
140. Review exceptions.
141. Remove obsolete exceptions.
142. Retire obsolete templates.
143. Retire obsolete modules.
144. Retire unsupported versions.
145. Maintain compatibility.
146. Announce breaking changes.
147. Provide migration paths.
148. Test platform changes.
149. Use pilot environments.
150. Use rollout waves.
```

## 204. Rules 151–200

```text
151. Provide service catalog.
152. Provide ownership visibility.
153. Provide dependency visibility.
154. Provide deployment visibility.
155. Provide SLO visibility.
156. Provide incident context.
157. Provide cost visibility.
158. Provide documentation links.
159. Provide runbook links.
160. Provide dashboards.
161. Provide alerts.
162. Provide standard metrics.
163. Provide standard logging.
164. Provide tracing support.
165. Provide business metric support.
166. Provide feature-flag integration where useful.
167. Provide preview environments where useful.
168. Give preview environments TTLs.
169. Clean up ephemeral resources.
170. Prevent cost leakage.
171. Allocate cost to teams.
172. Use tags and labels.
173. Provide FinOps visibility.
174. Monitor platform cost.
175. Review expensive defaults.
176. Balance standardization with flexibility.
177. Support advanced workloads through controlled extension points.
178. Keep APIs simple.
179. Keep common operations simple.
180. Provide meaningful errors.
181. Provide operation status.
182. Provide correlation IDs.
183. Provide audit records.
184. Provide change history.
185. Provide event schemas.
186. Version event schemas.
187. Provide notifications.
188. Integrate incident context.
189. Provide scorecards.
190. Make scorecards meaningful.
191. Avoid vanity metrics.
192. Define platform maturity.
193. Define platform roadmap.
194. Gather developer feedback.
195. Run platform pilots.
196. Measure adoption.
197. Migrate incrementally.
198. Support legacy migration.
199. Deprecate safely.
200. Keep platform ownership explicit.
```

## 205. Rules 201–250

```text
201. Run platform game days.
202. Test portal failure.
203. Test API failure.
204. Test Git failure.
205. Test CI failure.
206. Test registry failure.
207. Test GitOps failure.
208. Test AWS API throttling.
209. Test partial provisioning.
210. Test controller failure.
211. Test workflow retries.
212. Test reconciliation.
213. Test rollback.
214. Test disaster recovery.
215. Test clean-room reconstruction.
216. Measure platform RTO.
217. Measure platform RPO where applicable.
218. Preserve recovery evidence.
219. Treat templates as production code.
220. Treat Terraform modules as production code.
221. Treat pipeline templates as production code.
222. Review shared automation for security.
223. Scope platform credentials.
224. Rotate platform credentials.
225. Monitor privileged actions.
226. Protect platform databases.
227. Back up unique platform state.
228. Reconstruct reproducible infrastructure from Git/IaC where possible.
229. Protect developer data.
230. Protect audit data.
231. Avoid exposing sensitive diagnostics.
232. Protect portal authentication.
233. Protect portal authorization.
234. Rate-limit portal APIs.
235. Protect workflow queues.
236. Provide fair scheduling.
237. Protect critical production operations.
238. Use stronger controls for destructive operations.
239. Avoid unnecessary approvals.
240. Automate routine safe operations.
241. Keep exception workflows explicit.
242. Make the golden path the easiest path.
243. Optimize for outcomes, not tool count.
244. Prefer simplicity over fashionable architecture.
245. Prefer reusable capabilities.
246. Prefer measurable developer value.
247. Prefer safe automation over manual repetition.
248. Continuously remove platform friction.
249. A successful IDP makes engineering teams faster without making production
     less safe.
250. The ultimate goal is a reliable, secure, self-service internal product
     that reduces cognitive load, standardizes proven engineering practices,
     preserves autonomy, and enables teams to ship and operate software
     confidently at organizational scale.
```

# END OF 18-Internal-Developer-Platforms.md
