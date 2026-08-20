# GitLab DevOps Projects

> Production-oriented hands-on project guide for building DevOps and DevSecOps projects with GitLab. Projects progress from foundational CI/CD automation to AWS, Terraform, Docker, EKS, ArgoCD, security, observability, release automation and a complete production-grade microservices platform.

---

## 1. Why Build GitLab DevOps Projects?

Projects convert GitLab knowledge into operational skill.

A strong project should demonstrate:

```text
source control
CI/CD
security
infrastructure
containers
Kubernetes
GitOps
observability
troubleshooting
```

---

## 2. Project Progression

```text
Project 1 → Basic CI/CD
Project 2 → Docker CI
Project 3 → AWS Deployment
Project 4 → Terraform
Project 5 → EKS
Project 6 → DevSecOps
Project 7 → GitOps
Project 8 → Monitoring
Project 9 → Release Automation
Project 10 → Production Platform
```

---

## 3. Project 1 — GitLab CI/CD Fundamentals

Goal:

```text
Git push
 ↓
GitLab Pipeline
 ↓
Build
 ↓
Test
```

---

## 4. Project 1 Requirements

Create:

```text
application
.gitlab-ci.yml
README
```

---

## 5. Project 1 Pipeline

```yaml
stages:
  - test
  - build

test:
  stage: test
  script:
    - echo "Running tests"

build:
  stage: build
  script:
    - echo "Building application"
```

---

## 6. Project 1 Learning Outcomes

Learn:

```text
stages
jobs
scripts
pipeline
artifacts
```

---

## 7. Project 1 Improvement

Add:

```text
variables
rules
cache
artifacts
```

---

## 8. Project 1 Interview Value

Be able to explain:

```text
how pipeline starts
how jobs are scheduled
how runners execute jobs
```

---

## 9. Project 2 — Docker Build Pipeline

Goal:

```text
Git push
 ↓
test
 ↓
Docker build
 ↓
image
```

---

## 10. Project 2 Dockerfile

Use a small production-oriented base image where compatible.

---

## 11. Project 2 Pipeline

```text
test
 ↓
build image
 ↓
scan image
```

---

## 12. Project 2 Registry

Push to:

```text
GitLab Container Registry
```

or an approved external registry.

---

## 13. Project 2 Image Tagging

Use:

```text
commit SHA
```

rather than relying only on:

```text
latest
```

---

## 14. Project 2 Security

Add:

```text
Trivy image scan
```

---

## 15. Project 2 Artifact Traceability

Record:

```text
commit
image tag
image digest
pipeline
```

---

## 16. Project 3 — AWS EC2 Deployment

Goal:

```text
GitLab
 ↓
CI
 ↓
AWS EC2
 ↓
Application
```

---

## 17. Project 3 AWS Architecture

```text
GitLab
  │
  ▼
Runner
  │
  ▼
AWS
  │
  ▼
EC2
  │
  ▼
Application
```

---

## 18. Project 3 AWS Authentication

Avoid hardcoded long-lived AWS access keys where OIDC is supported.

---

## 19. Project 3 Deployment

Pipeline:

```text
Build
 ↓
Test
 ↓
Package
 ↓
Deploy
 ↓
Smoke Test
```

---

## 20. Project 3 Rollback

Keep a known-good artifact and deployment mechanism.

---

## 21. Project 4 — Terraform Infrastructure

Goal:

```text
GitLab
 ↓
Terraform
 ↓
AWS Infrastructure
```

---

## 22. Project 4 Resources

Provision:

```text
VPC
Subnets
Security Groups
IAM
EC2
S3
```

as appropriate.

---

## 23. Project 4 Terraform Structure

```text
terraform/
├── providers.tf
├── variables.tf
├── outputs.tf
├── main.tf
├── backend.tf
└── modules/
```

---

## 24. Project 4 Pipeline

```text
fmt
 ↓
validate
 ↓
security scan
 ↓
plan
 ↓
approval
 ↓
apply
```

---

## 25. Project 4 Remote State

Use an S3 backend with appropriate access control and state locking behavior supported by the chosen Terraform version/backend configuration.

---

## 26. Project 4 State Security

Protect state because it may contain sensitive infrastructure information.

---

## 27. Project 5 — EKS Infrastructure

Goal:

```text
GitLab
 ↓
Terraform
 ↓
EKS
```

---

## 28. Project 5 EKS Components

```text
VPC
EKS
Node Groups
IAM
Security Groups
ALB
ECR
```

---

## 29. Project 5 EKS Architecture

```text
AWS VPC
 │
 ├── Public Subnets
 │      └── ALB
 │
 └── Private Subnets
        └── EKS Nodes
              └── Pods
```

---

## 30. Project 5 Terraform Modules

Use modules for:

```text
VPC
security
ECR
EKS
ALB
IAM
```

---

## 31. Project 5 Kubernetes Validation

Validate:

```bash
kubectl get nodes
kubectl get pods -A
```

---

## 32. Project 5 Learning Outcomes

Demonstrate:

```text
IaC
AWS
EKS
Kubernetes
CI/CD
```

---

## 33. Project 6 — Microservices CI/CD

Goal:

```text
multiple services
 ↓
independent builds
 ↓
container images
```

---

## 34. Project 6 Example Services

```text
user
catalog
cart
orders
payment
inventory
notification
```

---

## 35. Project 6 Repository Strategy

Possible approaches:

```text
one repository
multiple repositories
monorepo
polyrepo
```

Choose based on project goals.

---

## 36. Project 6 Pipeline Strategy

Each service should have:

```text
test
build
scan
publish
```

---

## 37. Project 6 Image Strategy

Use:

```text
service:commit-sha
```

and record the digest.

---

## 38. Project 6 Parallelization

Build independent services in parallel.

---

## 39. Project 6 Failure Isolation

A failed service pipeline should not unnecessarily rebuild unrelated services.

---

## 40. Project 7 — DevSecOps Pipeline

Goal:

```text
Source
 ↓
Build
 ↓
SAST
 ↓
SCA
 ↓
Secret Scan
 ↓
Container Scan
 ↓
Deploy
```

---

## 41. Project 7 SonarQube

Integrate code quality and static analysis.

---

## 42. Project 7 Dependency Scanning

Identify vulnerable dependencies using the organization's selected scanner.

---

## 43. Project 7 Trivy

Scan:

```text
container image
filesystem
IaC
```

where appropriate.

---

## 44. Project 7 Veracode

Integrate Veracode where required for application security validation.

---

## 45. Project 7 Security Gates

Define policy:

```text
critical vulnerability
→ block promotion
```

Tune severity and exceptions according to actual policy.

---

## 46. Project 7 Exception Process

Security exceptions should be:

```text
documented
approved
time-bound
reviewed
```

---

## 47. Project 8 — GitLab + ArgoCD GitOps

Goal:

```text
GitLab CI
 ↓
Image
 ↓
GitOps repository
 ↓
ArgoCD
 ↓
EKS
```

---

## 48. Project 8 CI Responsibility

CI performs:

```text
build
test
scan
publish
```

---

## 49. Project 8 GitOps Responsibility

GitOps manages:

```text
desired deployment state
```

---

## 50. Project 8 Image Update

Automation updates:

```text
image tag
```

or preferably an immutable:

```text
image digest
```

---

## 51. Project 8 Merge Request

Create:

```text
GitOps branch
 ↓
MR
 ↓
validation
 ↓
approval
 ↓
merge
```

---

## 52. Project 8 ArgoCD

ArgoCD detects the Git change and reconciles the cluster.

---

## 53. Project 8 Drift

Manually changing production Kubernetes resources can create drift.

---

## 54. Project 8 Self-Heal

If enabled, ArgoCD can reconcile live state back toward Git desired state.

---

## 55. Project 9 — Multi-Environment GitOps

Environments:

```text
dev
test
staging
production
```

---

## 56. Project 9 Promotion

```text
dev
 ↓
test
 ↓
staging
 ↓
production
```

---

## 57. Project 9 Environment Isolation

Separate:

```text
credentials
namespaces
clusters/accounts
```

according to risk.

---

## 58. Project 9 Production Approval

Require explicit approval before production promotion.

---

## 59. Project 9 Rollback

Rollback the GitOps revision or revert the desired-state change.

---

## 60. Project 10 — Prometheus/Grafana Monitoring

Goal:

```text
EKS
 ↓
Prometheus
 ↓
Grafana
```

---

## 61. Project 10 Metrics

Monitor:

```text
CPU
memory
Pod restarts
request rate
error rate
latency
```

---

## 62. Project 10 Dashboards

Create dashboards for:

```text
cluster
nodes
applications
deployments
```

---

## 63. Project 10 Alerts

Examples:

```text
PodCrashLooping
HighCPU
HighMemory
High5xx
```

Tune thresholds to the workload.

---

## 64. Project 11 — ELK Logging

Goal:

```text
Applications
 ↓
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

---

## 65. Project 11 Structured Logging

Use fields:

```text
timestamp
level
service
environment
message
```

---

## 66. Project 11 Log Correlation

Add:

```text
request ID
version
deployment
```

where appropriate.

---

## 67. Project 11 Log Retention

Configure retention based on:

```text
troubleshooting
security
cost
compliance
```

---

## 68. Project 12 — GitLab API Automation

Goal:

```text
Python
 ↓
GitLab API
 ↓
Automation
```

---

## 69. Project 12 Inventory Script

Generate:

```text
project
namespace
visibility
default branch
last activity
```

---

## 70. Project 12 Pipeline Report

Generate:

```text
pipeline ID
status
duration
ref
```

---

## 71. Project 12 MR Automation

Automate:

```text
create branch
update file
create MR
add labels
```

---

## 72. Project 12 Authentication

Use a narrowly scoped identity and store credentials securely.

---

## 73. Project 12 Pagination

Implement API pagination correctly.

---

## 74. Project 12 Rate Limiting

Implement:

```text
429 handling
backoff
jitter
```

---

## 75. Project 13 — GitLab Webhook Automation

Architecture:

```text
GitLab
 ↓
Webhook
 ↓
Python Service
 ↓
Queue
 ↓
Worker
```

---

## 76. Project 13 Events

Process:

```text
push
merge request
pipeline
deployment
release
```

---

## 77. Project 13 Idempotency

Prevent duplicate event processing.

---

## 78. Project 13 Security

Validate webhook authentication and payload.

---

## 79. Project 13 Failure Handling

Use:

```text
retry
dead-letter queue
alert
```

---

## 80. Project 14 — Automated Release Pipeline

Goal:

```text
Merge
 ↓
Build
 ↓
Scan
 ↓
Publish
 ↓
Tag
 ↓
Release
```

---

## 81. Project 14 Versioning

Use a consistent versioning strategy.

---

## 82. Project 14 Release Notes

Generate notes from:

```text
merged MRs
commits
labels
```

---

## 83. Project 14 Artifact Traceability

Record:

```text
release
commit
pipeline
image digest
deployment
```

---

## 84. Project 15 — Canary Deployment

Architecture:

```text
Traffic
 ├── stable
 └── canary
```

---

## 85. Project 15 Canary Flow

```text
deploy canary
 ↓
smoke test
 ↓
monitor
 ↓
increase traffic
 ↓
full rollout
```

---

## 86. Project 15 Rollback

Rollback if:

```text
error rate rises
latency rises
health fails
business check fails
```

---

## 87. Project 16 — Blue-Green Deployment

Architecture:

```text
Blue  → current
Green → new
```

---

## 88. Project 16 Switch

After validation:

```text
traffic
 ↓
Green
```

---

## 89. Project 16 Rollback

Switch traffic back to Blue if the new environment fails.

---

## 90. Project 17 — Infrastructure Health Checker

Build a Python tool that checks:

```text
EC2
EKS
ECR
S3
RDS
```

---

## 91. Project 17 Output

Example:

```text
EC2     OK
EKS     OK
ECR     OK
S3      OK
RDS     WARNING
```

---

## 92. Project 17 Exit Codes

Return non-zero when critical checks fail.

---

## 93. Project 17 CI Integration

Run the checker:

```text
after infrastructure deployment
```

---

## 94. Project 18 — EKS Pod Health Monitor

Python automation checks:

```text
Pod status
restarts
readiness
OOMKilled
```

---

## 95. Project 18 Alert Conditions

Alert on:

```text
CrashLoopBackOff
Pending
high restarts
OOMKilled
```

---

## 96. Project 18 Kubernetes API

Use a Kubernetes client or controlled `kubectl` integration.

---

## 97. Project 18 Security

Use least-privilege Kubernetes RBAC.

---

## 98. Project 19 — Kubernetes Cleanup Automation

Identify:

```text
completed Jobs
stale resources
old ReplicaSets
```

according to safe cleanup policies.

---

## 99. Project 19 Dry Run

Support:

```bash
python cleanup.py --dry-run
```

---

## 100. Project 19 Deletion Safety

Never delete resources solely because they are old.

Check ownership and workload state.

---

## 101. Project 20 — GitLab Pipeline Health Dashboard

Collect:

```text
pipeline success
failure
duration
queue time
```

---

## 102. Project 20 Dashboard

Use Grafana for visualization when metrics are available through your monitoring architecture.

---

## 103. Project 20 Alerting

Alert on:

```text
failure spikes
queue growth
runner shortage
```

---

## 104. Project 21 — Runner Capacity Monitor

Track:

```text
active jobs
queued jobs
runner count
CPU
memory
```

---

## 105. Project 21 Capacity Alert

Alert when:

```text
queue > threshold
```

for a sustained period.

---

## 106. Project 21 Autoscaling

Scale runners according to demand.

---

## 107. Project 22 — GitLab Security Compliance Checker

Check repositories for:

```text
protected branches
security jobs
CODEOWNERS
secret exposure controls
```

---

## 108. Project 22 Compliance Report

Output:

```text
project
control
status
finding
recommendation
```

---

## 109. Project 22 Remediation

Prefer:

```text
create MR
```

over direct destructive changes.

---

## 110. Project 23 — Terraform Drift Detector

Architecture:

```text
Schedule
 ↓
Terraform Plan
 ↓
Detect Drift
 ↓
Report
```

---

## 111. Project 23 Drift Handling

Do not automatically apply changes unless the workflow explicitly requires it.

---

## 112. Project 24 — GitOps Drift Detector

Check:

```text
ArgoCD
applications
sync status
health
```

---

## 113. Project 24 Alert

Alert when production applications remain OutOfSync beyond an approved threshold.

---

## 114. Project 25 — End-to-End Microservices Platform

Final portfolio project:

```text
GitLab
Terraform
AWS
ECR
EKS
Helm
ArgoCD
Prometheus
Grafana
ELK
SonarQube
Trivy
Veracode
```

---

## 115. Project 25 Architecture

```text
Developer
    │
    ▼
 GitLab
    │
    ▼
 CI/CD
 ┌──┼───────────────┐
 ▼  ▼               ▼
Test Build       Security
       │
       ▼
      ECR
       │
       ▼
 GitOps Repository
       │
       ▼
     ArgoCD
       │
       ▼
      EKS
       │
       ▼
      ALB
       │
       ▼
 Microservices
       │
 ┌─────┴─────┐
 ▼           ▼
Metrics     Logs
 ▼           ▼
Prometheus   ELK
 ▼           ▼
Grafana     Kibana
```

---

## 116. Project 25 Infrastructure Layer

Terraform provisions:

```text
VPC
subnets
security groups
ECR
EKS
IAM
ALB-related resources
RDS/S3
```

as required.

---

## 117. Project 25 Application Layer

Deploy:

```text
user
catalog
cart
orders
payment
inventory
notification
```

---

## 118. Project 25 Container Layer

Each service gets:

```text
Dockerfile
image
scan
digest
```

---

## 119. Project 25 CI Layer

Pipeline:

```text
checkout
 ↓
test
 ↓
SonarQube
 ↓
dependency/security scan
 ↓
Docker build
 ↓
Trivy
 ↓
ECR push
```

---

## 120. Project 25 GitOps Layer

```text
CI
 ↓
update image digest
 ↓
GitOps MR
 ↓
approval
 ↓
merge
 ↓
ArgoCD
```

---

## 121. Project 25 Kubernetes Layer

Use:

```text
Deployments
Services
ConfigMaps
Secrets
Ingress
HPA
PDB
```

where appropriate.

---

## 122. Project 25 Helm

Use Helm for reusable Kubernetes deployment templates.

---

## 123. Project 25 Values

Separate environment values:

```text
dev
test
staging
production
```

---

## 124. Project 25 Security

Implement:

```text
RBAC
NetworkPolicies
workload identity
image scanning
secret controls
protected environments
```

---

## 125. Project 25 Observability

Metrics:

```text
Prometheus
Grafana
```

Logs:

```text
ELK
```

---

## 126. Project 25 Production Alerts

Examples:

```text
Pod crash
OOMKilled
high error rate
high latency
deployment failure
ArgoCD degraded
```

---

## 127. Project 25 Deployment Verification

After deployment:

```text
rollout status
Pod readiness
service health
ALB health
application smoke tests
metrics
logs
```

---

## 128. Project 25 Rollback

Use:

```text
GitOps revert
```

or another controlled rollback mechanism.

---

## 129. Project 25 Disaster Recovery

Document:

```text
Terraform recreation
ECR images
Git repositories
GitOps state
database backup
secrets recovery
```

---

## 130. Project 25 Interview Story

Explain:

```text
problem
architecture
implementation
security
deployment
monitoring
failure
recovery
```

---

## 131. Project Portfolio Structure

Recommended repository structure:

```text
project/
├── README.md
├── .gitlab-ci.yml
├── app/
├── docker/
├── terraform/
├── helm/
├── gitops/
├── scripts/
├── tests/
└── docs/
```

---

## 132. README Architecture

Every project README should include:

```text
Overview
Architecture
Technologies
Prerequisites
Installation
Configuration
Deployment
Testing
Security
Monitoring
Troubleshooting
Cleanup
```

---

## 133. Architecture Diagram

Include a clear diagram showing:

```text
GitLab
CI
AWS
ECR
EKS
ArgoCD
Monitoring
```

---

## 134. Project Prerequisites

Document:

```text
AWS account
GitLab
Docker
Terraform
kubectl
Helm
AWS CLI
ArgoCD
```

as required.

---

## 135. Project Variables

Document variables without exposing secrets.

---

## 136. Project Secrets

Never commit:

```text
AWS keys
GitLab tokens
passwords
private keys
```

---

## 137. `.gitignore`

Include:

```text
.terraform/
*.tfstate
*.tfstate.*
.env
credentials
```

and other project-specific secrets.

---

## 138. CI Variables

Use protected/masked variables where appropriate.

---

## 139. OIDC Project

Demonstrate:

```text
GitLab
 ↓
OIDC
 ↓
AWS STS
 ↓
IAM role
```

---

## 140. OIDC Trust Policy

Restrict trust to the intended GitLab project/ref/environment claims.

---

## 141. Project Security Scan

Scan:

```text
source
dependencies
IaC
containers
```

---

## 142. IaC Security

Use appropriate tools to detect:

```text
public resources
weak IAM
open security groups
```

---

## 143. Container Security

Check:

```text
vulnerabilities
secrets
root user
unnecessary packages
```

---

## 144. Kubernetes Security

Check:

```text
RBAC
privileged containers
host mounts
network policies
```

---

## 145. Project Testing

Include:

```text
unit
integration
smoke
deployment
```

tests where appropriate.

---

## 146. Pipeline Test Strategy

```text
fast tests
 ↓
security
 ↓
build
 ↓
deployment
 ↓
smoke
```

---

## 147. Failure Testing

Intentionally test:

```text
bad image
bad configuration
missing secret
insufficient IAM
failed Pod
```

in a safe environment.

---

## 148. Project Troubleshooting

Document common failures:

```text
pipeline pending
image pull
AWS permission
Pod crash
ArgoCD sync
ALB health
```

---

## 149. Project Runbook

Create:

```text
docs/runbook.md
```

for operational procedures.

---

## 150. Project Cleanup

Provide safe commands to remove:

```text
EKS
EC2
ECR
S3
RDS
ALB
```

where the project creates them.

---

## 151. Cost Control

For learning projects:

```text
destroy unused infrastructure
use smaller resources
avoid unnecessary multi-AZ expensive services
```

while preserving the architecture lesson.

---

## 152. Project Demo

A strong demo shows:

```text
commit
 ↓
pipeline
 ↓
security
 ↓
image
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
 ↓
application
 ↓
Grafana
```

---

## 153. Portfolio Evidence

Capture:

```text
architecture diagram
pipeline
security scan
ArgoCD
Kubernetes
Grafana
Kibana
```

screenshots where appropriate.

---

## 154. GitLab Repository README

State:

```text
what problem it solves
what you built
why technologies were selected
```

---

## 155. Production Claims

Do not claim:

```text
production-grade
high availability
zero downtime
```

unless the implementation actually demonstrates the required controls.

---

## 156. Resume Mapping

Map project work to:

```text
GitLab CI/CD
AWS
Terraform
Docker
EKS
ArgoCD
DevSecOps
Prometheus
Grafana
ELK
```

---

## 157. LinkedIn Project Post

Explain:

```text
problem
architecture
automation
security
result
```

without posting credentials or sensitive infrastructure details.

---

## 158. Project 25 Resume Bullet

Example:

> Designed and automated a GitLab-based DevSecOps delivery platform using Terraform, AWS ECR/EKS, Docker, Helm, ArgoCD, SonarQube, Trivy, Prometheus, Grafana and ELK for secure build, deployment, GitOps reconciliation and production observability.

---

## 159. Project 25 Interview Explanation

Start with:

```text
architecture
```

then explain:

```text
CI/CD
security
infrastructure
deployment
observability
troubleshooting
```

---

## 160. Scenario — Pipeline Fails

Explain:

```text
check stage
check logs
check runner
check dependency
```

---

## 161. Scenario — Image Scan Fails

Explain:

```text
identify vulnerability
check severity
validate exploitability
fix dependency/base image
rescan
```

---

## 162. Scenario — ECR Push Fails

Explain:

```text
STS identity
IAM
repository
region
authentication
network
```

---

## 163. Scenario — ArgoCD OutOfSync

Explain:

```text
Git desired state
live state
diff
manual change
sync
```

---

## 164. Scenario — Pod CrashLoopBackOff

Explain:

```text
events
logs
previous logs
exit code
resources
configuration
probes
```

---

## 165. Scenario — ALB Returns 503

Explain:

```text
target health
Service endpoints
Pod readiness
ports
security groups
```

---

## 166. Scenario — Deployment Increased 5xx

Explain:

```text
deployment timeline
image digest
Pod logs
metrics
rollback
```

---

## 167. Scenario — Runner Queue Grows

Explain:

```text
runner availability
tags
concurrency
resource capacity
autoscaling
```

---

## 168. Scenario — Terraform Drift

Explain:

```text
plan
identify external change
decide ownership
update code or reconcile
```

---

## 169. Scenario — Secret Leaked

Explain:

```text
revoke
rotate
audit
remove exposure
verify
```

---

## 170. Project Failure Injection

Use controlled failure experiments to demonstrate troubleshooting skill.

---

## 171. Failure Injection Example

Break:

```text
image tag
```

and diagnose:

```text
ImagePullBackOff
```

---

## 172. Failure Injection Example

Remove a required environment variable and diagnose:

```text
application startup failure
```

---

## 173. Failure Injection Example

Reduce memory limit and observe:

```text
OOMKilled
```

---

## 174. Failure Injection Example

Remove a runner tag match and diagnose:

```text
pending job
```

---

## 175. Failure Injection Example

Introduce an invalid GitOps manifest and diagnose:

```text
ArgoCD sync failure
```

---

## 176. Failure Injection Example

Introduce a security group rule problem and diagnose:

```text
application connectivity failure
```

---

## 177. Project Observability

Every production-oriented project should answer:

```text
Is it healthy?
Why is it unhealthy?
What changed?
How do I recover?
```

---

## 178. Project Metrics

Collect:

```text
request rate
error rate
latency
resource usage
deployment frequency
pipeline duration
```

---

## 179. Project Logs

Centralize:

```text
application
Kubernetes
CI/CD
```

logs where appropriate.

---

## 180. Project Alerts

Create alerts for:

```text
availability
errors
latency
resource saturation
deployment failure
```

---

## 181. Project SLO

Define at least one measurable reliability target for the application.

---

## 182. Project DR

Document:

```text
what is backed up
where
how to restore
who performs recovery
```

---

## 183. Project Security Review

Review:

```text
IAM
secrets
network
containers
Kubernetes
GitLab
```

before presenting it as production-oriented.

---

## 184. Project Architecture Review

Ask:

```text
Where is the single point of failure?
What is the blast radius?
How does rollback work?
How is it monitored?
How is it recovered?
```

---

## 185. Project Cost Review

Identify expensive components:

```text
EKS
NAT
RDS
load balancers
storage
observability
```

---

## 186. Project Scalability Review

Explain how to scale:

```text
CI runners
EKS nodes
Pods
database
storage
```

---

## 187. Project Security Review

Explain:

```text
who can deploy
who can approve
who can access AWS
who can access production
```

---

## 188. Project GitOps Review

Explain:

```text
source repository
GitOps repository
CI boundary
ArgoCD boundary
production access
```

---

## 189. Project Production Readiness

A project is stronger when it includes:

```text
automation
security
observability
recovery
documentation
```

---

## 190. Project Interview Checklist

Be prepared to explain:

```text
architecture
CI/CD
AWS
Terraform
Docker
EKS
Helm
ArgoCD
security
monitoring
troubleshooting
rollback
DR
```

---

## 191. Senior Interview — Which Project Would You Present First?

> I would present the end-to-end microservices platform because it demonstrates the complete DevOps lifecycle: GitLab CI/CD, security scanning, Docker, ECR, Terraform, EKS, Helm, ArgoCD, Prometheus, Grafana and ELK. I would explain the architecture first and then walk through a production deployment and failure scenario.

---

## 192. Senior Interview — How Did You Design Your CI/CD Pipeline?

> I separated validation, security, build, artifact publication and deployment responsibilities. Independent jobs run in parallel where possible, immutable images are published, and production deployment is controlled through GitOps and approval boundaries.

---

## 193. Senior Interview — Why Use Terraform?

> Terraform provides repeatable, reviewable infrastructure provisioning. I keep infrastructure configuration in Git, use modules for reusable components, store state remotely and run validation, security checks and plans through GitLab CI.

---

## 194. Senior Interview — Why EKS?

> EKS provides managed Kubernetes control-plane operations while allowing the platform team to manage workloads, networking, IAM integration and deployment strategy. It fits well with the Kubernetes and GitOps architecture.

---

## 195. Senior Interview — Why ArgoCD?

> ArgoCD provides continuous reconciliation between Git desired state and Kubernetes live state. It also gives visibility into synchronization, health and drift.

---

## 196. Senior Interview — How Do You Secure the Pipeline?

> I use protected branches and environments, least-privilege identities, secret management, SAST/SCA, container scanning, IaC scanning where applicable, approval gates and immutable artifacts.

---

## 197. Senior Interview — How Do You Deploy Without Rebuilding?

> I build and scan the artifact once, publish it with an immutable identifier, and promote that exact artifact across environments. This avoids environment-specific rebuilds that can produce different binaries.

---

## 198. Senior Interview — How Do You Roll Back?

> With GitOps I revert the deployment desired state to a known-good image digest or revision, allow ArgoCD to reconcile it, and verify application health through Kubernetes, metrics and logs.

---

## 199. Senior Interview — How Do You Troubleshoot the Complete Platform?

> I isolate the failing layer: GitLab, runner, build, security, registry, GitOps, Kubernetes, ingress or application. I use pipeline logs, Prometheus metrics, ELK logs, Kubernetes events and deployment history to build an evidence-based diagnosis.

---

## 200. Senior Interview — What Makes Your Project Production-Oriented?

> It is not just a successful deployment. It includes security controls, immutable artifacts, environment isolation, GitOps, monitoring, alerts, rollback, failure testing, backup/recovery planning, documentation and operational troubleshooting.

---

## 201. Final Project Selection

For a strong DevOps portfolio, prioritize:

```text
1. End-to-End Microservices Platform
2. GitLab + Terraform + EKS
3. GitLab + ArgoCD GitOps
4. DevSecOps Pipeline
5. Monitoring and Logging
6. API/Automation
```

---

## 202. Recommended Capstone

Build one integrated platform rather than many disconnected demos.

```text
GitLab
 +
Terraform
 +
AWS
 +
Docker
 +
ECR
 +
EKS
 +
Helm
 +
ArgoCD
 +
SonarQube
 +
Trivy
 +
Veracode
 +
Prometheus
 +
Grafana
 +
ELK
```

---

## 203. Capstone Repository Layout

```text
devops-platform/
├── README.md
├── application/
├── terraform/
│   ├── modules/
│   └── environments/
├── docker/
├── helm/
├── gitops/
├── scripts/
├── tests/
├── docs/
└── .gitlab-ci.yml
```

---

## 204. Capstone CI/CD

```text
Commit
 ↓
Validate
 ↓
Test
 ↓
SonarQube
 ↓
Security/SCA
 ↓
Docker Build
 ↓
Trivy
 ↓
ECR
 ↓
GitOps Update
 ↓
MR
 ↓
Approval
 ↓
ArgoCD
```

---

## 205. Capstone Production Flow

```text
Developer
 ↓
GitLab
 ↓
CI
 ↓
Immutable Image
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
 ↓
ALB
 ↓
Microservices
 ↓
Prometheus/Grafana
 ↓
ELK
```

---

## 206. Capstone Failure Flow

```text
Deployment
 ↓
Pod Failure
 ↓
Prometheus Alert
 ↓
ELK Logs
 ↓
Incident Diagnosis
 ↓
GitOps Rollback
 ↓
ArgoCD
 ↓
Healthy Version
```

---

## 207. Capstone Security Flow

```text
Developer
 ↓
Protected Git
 ↓
CI
 ├── SAST
 ├── SCA
 ├── Secret Scan
 ├── IaC Scan
 └── Container Scan
 ↓
Approval
 ↓
Production
```

---

## 208. Capstone AWS Identity Flow

```text
GitLab Job
 ↓
OIDC
 ↓
AWS STS
 ↓
IAM Role
 ├── ECR
 ├── Terraform
 └── Required AWS APIs
```

---

## 209. Capstone Kubernetes Security Flow

```text
GitOps
 ↓
ArgoCD
 ↓
RBAC
 ↓
Namespace
 ↓
ServiceAccount
 ↓
Workload Identity
 ↓
Application
```

---

## 210. Capstone Observability Flow

```text
Applications
 ├── Metrics → Prometheus → Grafana
 └── Logs   → ELK
```

---

## 211. Capstone DR Flow

```text
Failure
 ↓
Detect
 ↓
Assess
 ↓
Restore infrastructure
 ↓
Restore required state
 ↓
Reconcile GitOps
 ↓
Verify
```

---

## 212. Capstone Final Checklist

```text
[ ] GitLab repository
[ ] GitLab CI/CD
[ ] Docker
[ ] ECR
[ ] Terraform
[ ] AWS VPC
[ ] EKS
[ ] Helm
[ ] ArgoCD
[ ] GitOps
[ ] SonarQube
[ ] Trivy
[ ] Veracode
[ ] Prometheus
[ ] Grafana
[ ] ELK
[ ] Security
[ ] Secrets
[ ] IAM/OIDC
[ ] Rollback
[ ] DR
[ ] Troubleshooting
[ ] README
[ ] Architecture diagram
```

---

## 213. Final Mental Model

```text
                       GITLAB DEVOPS PROJECTS

                              Git
                               │
                               ▼
                            GitLab
                               │
                               ▼
                         CI/CD Pipeline
                               │
          ┌────────────────────┼────────────────────┐
          ▼                    ▼                    ▼
        Test                 Security              Build
                               │                    │
                               └─────────┬──────────┘
                                         ▼
                                        ECR
                                         │
                                         ▼
                                   GitOps Repository
                                         │
                                         ▼
                                       ArgoCD
                                         │
                                         ▼
                                        EKS
                                         │
                         ┌───────────────┼───────────────┐
                         ▼               ▼               ▼
                        ALB         Microservices      HPA
                                         │
                         ┌───────────────┴───────────────┐
                         ▼                               ▼
                    Prometheus                          ELK
                         │                               │
                       Grafana                         Kibana
                         │                               │
                         └───────────────┬───────────────┘
                                         ▼
                                  Incident Response
                                         │
                                         ▼
                                  Rollback / Recovery
```

> **Core principle:** A strong GitLab DevOps project is an end-to-end engineering system, not just a `.gitlab-ci.yml` file. The best portfolio project demonstrates how source code becomes a tested and secured immutable artifact, how infrastructure is provisioned with Terraform, how Kubernetes is deployed through GitOps, how production is observed with Prometheus/Grafana and ELK, and how failures are detected, diagnosed and recovered safely.

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md
├── 02-GitLab-Repository-and-Git-Workflow.md
├── 03-GitLab-Branches-and-Merge-Requests.md
├── 04-GitLab-CI-CD-Fundamentals.md
├── 05-GitLab-CI-CD-Configuration.md
├── 06-GitLab-Runners.md
├── 07-GitLab-Variables-Secrets-and-Environments.md
├── 08-GitLab-Artifacts-Caching-and-Dependencies.md
├── 09-GitLab-Docker-and-Container-Registry.md
├── 10-GitLab-AWS-Integration.md
├── 11-GitLab-Kubernetes-and-EKS.md
├── 12-GitLab-Terraform-and-IaC.md
├── 13-GitLab-ArgoCD-and-GitOps.md
├── 14-GitLab-DevSecOps.md
├── 15-GitLab-Security.md
├── 16-GitLab-Advanced-CI-CD.md
├── 17-GitLab-Multi-Environment-Deployments.md
├── 18-GitLab-Advanced-Pipelines.md
├── 19-GitLab-API-and-Automation.md
├── 20-GitLab-Monitoring-and-Observability.md
├── 21-GitLab-Production-Troubleshooting.md
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md ✓
└── 24-GitLab-Interview-Preparation.md
```

**Next: `24-GitLab-Interview-Preparation.md` — final file in the planned GitLab section.**
