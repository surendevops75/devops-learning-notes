# 13-GitLab — 24 GitLab Interview Preparation

> Final interview-preparation guide for GitLab and GitLab-based DevOps/DevSecOps roles. Covers fundamentals, CI/CD, runners, security, AWS, Terraform, Docker, EKS, Helm, ArgoCD/GitOps, monitoring, troubleshooting, production architecture, scenario-based questions, project discussions, coding/automation questions and senior-level interview answers.

---

## 1. Interview Preparation Strategy

For GitLab interviews, prepare at four levels:

```text
Concept
 ↓
Implementation
 ↓
Troubleshooting
 ↓
Architecture
```

---

## 2. Answer Structure

Use:

```text
What
Why
How
Production example
Failure handling
```

---

## 3. Avoid Memorized Answers

Interviewers usually probe:

```text
why?
what if it fails?
how did you implement it?
how did you troubleshoot it?
```

---

## 4. Your GitLab Positioning

Present GitLab as part of your broader DevOps platform:

```text
GitLab
+
AWS
+
Terraform
+
Docker
+
EKS
+
ArgoCD
+
DevSecOps
+
Observability
```

---

## 5. GitLab Fundamentals

GitLab provides capabilities for:

```text
source control
CI/CD
security
packages
container registry
release management
automation
```

---

## 6. What Is GitLab?

GitLab is a DevSecOps platform that integrates source-code management with software delivery and security workflows.

---

## 7. GitLab vs Git

Git is:

```text
distributed version control
```

GitLab is a platform built around Git repositories and software delivery workflows.

---

## 8. GitLab Project

A project contains resources such as:

```text
repository
issues
merge requests
CI/CD
variables
packages
registry
```

---

## 9. GitLab Group

Groups organize multiple projects and support centralized governance.

---

## 10. Namespace

A namespace identifies the location of a project under a user or group hierarchy.

---

## 11. Repository

The repository stores:

```text
source code
branches
commits
tags
history
```

---

## 12. Branch

A branch provides an independent line of development.

---

## 13. Protected Branch

Protected branches restrict potentially dangerous operations such as direct pushes or unauthorized merges.

---

## 14. Merge Request

An MR provides:

```text
code review
discussion
pipeline validation
approval
merge
```

---

## 15. CODEOWNERS

CODEOWNERS can define reviewers or owners for specific paths.

---

## 16. GitLab CI/CD

GitLab CI/CD automates:

```text
build
test
scan
package
deploy
```

---

## 17. `.gitlab-ci.yml`

This file defines the CI/CD configuration for a project.

---

## 18. Pipeline

A pipeline is a collection of jobs executed according to configured stages, rules and dependencies.

---

## 19. Job

A job defines work executed by a runner.

---

## 20. Stage

A stage groups jobs into a logical execution phase.

---

## 21. Pipeline Example

```yaml
stages:
  - test
  - build
  - deploy

test:
  stage: test
  script:
    - ./run-tests.sh

build:
  stage: build
  script:
    - ./build.sh

deploy:
  stage: deploy
  script:
    - ./deploy.sh
```

---

## 22. Parallel Jobs

Jobs in the same stage can run in parallel when dependencies do not force serialization.

---

## 23. `needs`

`needs` creates a dependency relationship and can enable DAG-style execution.

---

## 24. DAG Pipelines

A DAG allows independent jobs to proceed without waiting for unrelated stage completion.

---

## 25. `rules`

`rules` control whether and how jobs are included.

---

## 26. `workflow: rules`

Workflow rules can control whether an entire pipeline is created.

---

## 27. `only/except`

Older configurations may use `only/except`. Prefer current `rules`-based design for new pipelines where appropriate.

---

## 28. Manual Job

A manual job requires an explicit action before execution.

---

## 29. Scheduled Pipeline

A scheduled pipeline runs according to a configured schedule.

---

## 30. Triggered Pipeline

A pipeline can be triggered by:

```text
API
pipeline trigger
another pipeline
webhook
```

depending on configuration.

---

## 31. Parent-Child Pipeline

Parent-child pipelines divide complex CI/CD workflows into smaller pipeline configurations.

---

## 32. Multi-Project Pipeline

A pipeline can trigger work in another project.

---

## 33. Pipeline Optimization

Improve pipeline performance with:

```text
parallelization
needs
cache
artifacts
smaller images
dependency optimization
```

---

## 34. Pipeline Failure

Start with:

```text
job
runner
logs
exit code
dependency
```

---

## 35. Runner

A GitLab Runner executes CI/CD jobs.

---

## 36. Runner Registration

A runner must be associated with GitLab using supported authentication/configuration mechanisms.

---

## 37. Runner Types

Conceptually:

```text
instance/shared
group
project
```

depending on GitLab configuration.

---

## 38. Runner Executors

Common executors:

```text
Docker
Kubernetes
Shell
```

---

## 39. Docker Executor

Jobs execute using Docker containers.

---

## 40. Kubernetes Executor

Jobs execute as Kubernetes Pods.

---

## 41. Shell Executor

Jobs execute directly on the runner host.

---

## 42. Runner Tags

Tags allow jobs to select appropriate runners.

---

## 43. Protected Runner

Protected runners can be restricted to protected refs according to configuration.

---

## 44. Runner Security

Secure:

```text
runner host
credentials
network
executor
job isolation
```

---

## 45. Privileged Runner

Privileged execution increases risk and should be enabled only when required.

---

## 46. Runner Pending Job

Check:

```text
runner online
tags
protected status
capacity
concurrency
```

---

## 47. Runner Offline

Check:

```text
runner process
host
network
authentication
```

---

## 48. Kubernetes Runner

Architecture:

```text
GitLab
 ↓
Runner
 ↓
Kubernetes API
 ↓
Job Pod
```

---

## 49. Runner Autoscaling

Scale runners based on:

```text
queue
concurrency
job duration
resource availability
```

---

## 50. CI Variables

Variables provide configurable values to jobs.

---

## 51. Protected Variables

Protected variables can be restricted to protected branches/tags.

---

## 52. Masked Variables

Mask secrets where supported so they are not exposed in job logs.

---

## 53. Secret Management

Prefer:

```text
short-lived credentials
OIDC
secret manager
protected variables
```

over hardcoded secrets.

---

## 54. AWS OIDC

Preferred architecture:

```text
GitLab Job
 ↓
OIDC token
 ↓
AWS STS
 ↓
IAM Role
 ↓
AWS API
```

---

## 55. Why OIDC?

Benefits:

```text
short-lived credentials
no long-lived access keys
auditable identity
least privilege
```

---

## 56. AWS AccessDenied

Troubleshooting:

```text
aws sts get-caller-identity
 ↓
identify action
 ↓
identify resource
 ↓
inspect IAM
```

---

## 57. Docker in GitLab

A pipeline can:

```text
build
scan
tag
push
```

container images.

---

## 58. Image Tagging

Prefer:

```text
commit SHA
version
digest
```

over mutable `latest` for production traceability.

---

## 59. ECR Integration

```text
GitLab CI
 ↓
AWS OIDC
 ↓
ECR authentication
 ↓
docker push
```

---

## 60. ECR Push Failure

Check:

```text
AWS identity
IAM permissions
region
repository
Docker authentication
network
```

---

## 61. Container Scan

Trivy can scan container images for vulnerabilities.

---

## 62. SonarQube

SonarQube can provide code quality and static analysis.

---

## 63. Veracode

Veracode can be integrated into application security workflows where required.

---

## 64. DevSecOps Pipeline

```text
Source
 ↓
Test
 ↓
SAST
 ↓
SCA
 ↓
Secret Detection
 ↓
Build
 ↓
Container Scan
 ↓
Publish
 ↓
Deploy
```

---

## 65. Security Gate

A security gate can prevent promotion when policy-defined findings exceed acceptable thresholds.

---

## 66. False Positive

Do not simply suppress findings.

Use:

```text
validation
documentation
approval
time-bound exception
```

---

## 67. Terraform with GitLab

Terraform pipeline:

```text
fmt
 ↓
validate
 ↓
security
 ↓
plan
 ↓
approval
 ↓
apply
```

---

## 68. Terraform State

Use remote state with appropriate access control and supported locking.

---

## 69. Terraform Drift

Run:

```text
terraform plan
```

and determine whether external changes should be reconciled into code or reverted.

---

## 70. Terraform Partial Failure

After failure:

```text
inspect state
inspect actual resources
run plan
```

before retrying.

---

## 71. Terraform Production Safety

Do not automatically apply production changes without appropriate review and approval.

---

## 72. EKS with GitLab

A common architecture:

```text
GitLab
 ↓
Terraform
 ↓
AWS VPC
 ↓
EKS
 ↓
Kubernetes
```

---

## 73. EKS Deployment Pipeline

```text
Build
 ↓
Scan
 ↓
ECR
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
```

---

## 74. Kubernetes Deployment

Use:

```yaml
Deployment
Service
ConfigMap
Secret
Ingress
HPA
```

as appropriate.

---

## 75. Kubernetes Rollout

Check:

```bash
kubectl rollout status deployment/<name> -n <namespace>
```

---

## 76. CrashLoopBackOff

Investigate:

```text
events
logs
previous logs
exit code
resources
environment
probes
```

---

## 77. OOMKilled

Check:

```text
memory request
memory limit
usage
application behavior
```

---

## 78. ImagePullBackOff

Check:

```text
image
tag
registry
credentials
IAM
network
```

---

## 79. Pending Pod

Check:

```text
resources
taints
tolerations
affinity
quota
PVC
```

---

## 80. Readiness Failure

A readiness failure normally removes the Pod from service traffic.

---

## 81. Liveness Failure

A liveness failure can cause container restarts.

---

## 82. Startup Probe

Use startup probes for applications that require significant initialization time.

---

## 83. Service No Endpoints

Check:

```text
selector
Pod labels
Pod readiness
namespace
```

---

## 84. ALB Ingress

Architecture:

```text
Route53
 ↓
ALB
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

---

## 85. ALB 503

Check:

```text
target health
Service
endpoints
Pod readiness
```

---

## 86. ALB 502

Investigate:

```text
target connection
application port
health
```

---

## 87. ALB 504

Investigate:

```text
backend latency
timeout
network
```

---

## 88. ArgoCD

ArgoCD implements GitOps reconciliation between desired and live Kubernetes state.

---

## 89. GitOps Flow

```text
GitLab CI
 ↓
Build
 ↓
ECR
 ↓
GitOps repository
 ↓
ArgoCD
 ↓
EKS
```

---

## 90. CI vs ArgoCD

CI:

```text
build
test
scan
publish
```

ArgoCD:

```text
deploy
reconcile
detect drift
```

---

## 91. ArgoCD OutOfSync

Check:

```text
Git revision
diff
live state
manual changes
```

---

## 92. ArgoCD Sync Failure

Check:

```text
manifest
RBAC
Kubernetes API
resource validation
```

---

## 93. ArgoCD Degraded

Check:

```text
Pod
Deployment
Service
Ingress
application health
```

---

## 94. GitOps Rollback

Revert the desired-state change to a known-good image digest/revision and allow ArgoCD to reconcile.

---

## 95. Helm

Helm packages Kubernetes manifests into reusable charts.

---

## 96. Helm Values

Environment-specific configuration can be supplied through values.

---

## 97. Helm Interview Question

Why Helm?

> It reduces duplicated Kubernetes manifests and provides reusable templates with configurable values.

---

## 98. Multi-Environment Deployment

Typical flow:

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

## 99. Environment Protection

Production should have stricter:

```text
permissions
approval
credentials
```

---

## 100. Build Once, Promote Many

Build the artifact once and promote the same immutable artifact through environments.

---

## 101. Why Immutable Artifacts?

They improve:

```text
traceability
rollback
reproducibility
security
```

---

## 102. Deployment Traceability

Trace:

```text
MR
 ↓
commit
 ↓
pipeline
 ↓
artifact
 ↓
image digest
 ↓
GitOps commit
 ↓
ArgoCD
 ↓
Pod
```

---

## 103. Monitoring GitLab

Monitor:

```text
pipeline success
pipeline duration
runner queue
API latency
errors
```

---

## 104. Prometheus

Prometheus collects time-series metrics.

---

## 105. Grafana

Grafana visualizes metrics and supports dashboards and alerting workflows.

---

## 106. ELK

```text
Elasticsearch
Logstash
Kibana
```

supports centralized log analysis.

---

## 107. GitLab Observability

Monitor:

```text
GitLab
runners
Kubernetes
applications
AWS
```

---

## 108. Golden Signals

Use:

```text
latency
traffic
errors
saturation
```

for service health.

---

## 109. Pipeline SLI

Possible metrics:

```text
success rate
queue time
duration
failure rate
```

---

## 110. Alert Design

Good alerts are:

```text
actionable
specific
owned
```

---

## 111. Production Troubleshooting Method

Use:

```text
impact
timeline
recent changes
metrics
logs
events
dependencies
mitigation
verification
RCA
```

---

## 112. GitLab Outage Scenario

Answer:

> I first confirm whether the outage is global or limited to a project, user, region or function. I check load balancer health, GitLab application health, database/Redis dependencies, resource saturation and recent changes. I establish impact and mitigate before performing deeper RCA.

---

## 113. Pipeline Pending Scenario

Answer:

> I check whether matching runners are online, whether tags match, whether the job requires a protected runner, and whether runner concurrency or capacity is exhausted.

---

## 114. Pipeline Failed Scenario

Answer:

> I inspect the exact failed job, logs and exit code, then determine whether the failure is application code, dependency installation, runner environment, security tooling, registry access or deployment.

---

## 115. Deployment Failed Scenario

Answer:

> I identify whether failure occurred in CI, GitOps or Kubernetes. I verify the commit, image digest and ArgoCD revision, then inspect Kubernetes events, Pod logs, readiness and dependency health.

---

## 116. Production Rollback Scenario

Answer:

> If the previous artifact is known-good and rollback is safe, I revert the deployment desired state to that immutable artifact, allow ArgoCD to reconcile, and verify application health.

---

## 117. Runner Security Scenario

Answer:

> I isolate production workloads, use protected runners, restrict network access, avoid unnecessary privileged execution, use least-privilege credentials and rebuild compromised runners rather than trusting them.

---

## 118. AWS Security Scenario

Answer:

> I use GitLab OIDC with AWS STS, restrict IAM trust to intended project/ref claims and grant only required AWS permissions.

---

## 119. Secret Leak Scenario

Answer:

```text
revoke
 ↓
rotate
 ↓
audit
 ↓
remove exposure
 ↓
verify
```

---

## 120. GitLab API

GitLab provides APIs for automation.

---

## 121. API Automation

Use APIs to automate:

```text
projects
MRs
pipelines
variables
members
deployments
```

according to permissions.

---

## 122. API Pagination

Always account for paginated responses.

---

## 123. API Rate Limits

Use:

```text
backoff
jitter
bounded retries
```

---

## 124. API 401

Likely:

```text
invalid/expired credential
```

---

## 125. API 403

Likely:

```text
permission problem
```

---

## 126. API 404

Check:

```text
URL
project ID
namespace
visibility
```

---

## 127. API 429

Implement controlled retry with backoff.

---

## 128. Webhooks

Webhooks allow GitLab events to trigger external automation.

---

## 129. Webhook Security

Validate:

```text
secret
event
payload
source
```

---

## 130. Webhook Idempotency

Do not process the same event multiple times accidentally.

---

## 131. Python GitLab Automation

Useful automation:

```text
project inventory
pipeline reporting
MR automation
cleanup
compliance
```

---

## 132. Python Error Handling

Use:

```python
try:
    result = operation()
except Exception as exc:
    logger.exception("Operation failed: %s", exc)
    raise
```

Do not silently swallow production failures.

---

## 133. Python Retry

Retry only transient failures.

Avoid retrying:

```text
authentication failure
invalid configuration
```

indefinitely.

---

## 134. Bash Automation

Use strict handling where appropriate:

```bash
set -euo pipefail
```

---

## 135. Shell Script Safety

Validate variables before using them in destructive commands.

---

## 136. Idempotent Automation

A script should safely produce the intended state when run multiple times.

---

## 137. GitLab Security Interview

Question:

> How do you protect CI/CD secrets?

Answer:

> I use protected and masked variables where appropriate, prefer short-lived OIDC credentials, restrict production environments and avoid printing secrets in logs.

---

## 138. GitLab CI Security Interview

Question:

> How do you prevent untrusted code from accessing production credentials?

Answer:

> I separate protected environments and runners, restrict variables to protected refs, enforce branch/MR controls and avoid giving production credentials to untrusted jobs.

---

## 139. GitLab Runner Interview

Question:

> Why use Kubernetes runners?

Answer:

> Kubernetes runners provide isolated, ephemeral job Pods and can scale with workload demand, making them useful for dynamic CI workloads.

---

## 140. Runner Executor Comparison

```text
Shell       → simple, low isolation
Docker      → container isolation
Kubernetes  → ephemeral/scalable Pods
```

---

## 141. GitLab CI Optimization Interview

Answer:

> I use DAG dependencies, parallel jobs, appropriate caching, reusable templates, smaller images and dependency optimization while keeping the pipeline secure and deterministic.

---

## 142. Cache vs Artifact

Cache:

```text
speed/reusable dependencies
```

Artifact:

```text
job output passed or retained
```

---

## 143. Artifact vs Image

Artifact:

```text
pipeline output
```

Image:

```text
deployable container artifact
```

---

## 144. Cache Key

Use cache keys that correctly represent dependency changes.

---

## 145. Artifact Retention

Keep artifacts only as long as required.

---

## 146. Pipeline Failure Rate

Track:

```text
failed pipelines / total pipelines
```

and investigate trends rather than isolated failures.

---

## 147. Pipeline Duration

Break duration into:

```text
queue
checkout
dependencies
test
build
scan
publish
deploy
```

---

## 148. Queue Time

High queue time usually indicates:

```text
runner shortage
tag mismatch
capacity pressure
```

---

## 149. Deployment Frequency

Deployment frequency is one useful delivery-performance metric.

---

## 150. Change Failure Rate

Track how frequently deployments result in:

```text
rollback
incident
hotfix
```

---

## 151. Mean Time to Restore

Measure how quickly service returns to acceptable operation after failure.

---

## 152. DORA Metrics

Common metrics include:

```text
deployment frequency
lead time for changes
change failure rate
time to restore
```

Use them as engineering signals rather than vanity metrics.

---

## 153. Production Architecture Interview

Question:

> Design GitLab for a production environment.

Answer:

> I would separate the application layer from stateful dependencies, provide appropriate HA for GitLab components, use durable object storage, redundant runners, protected environments, centralized observability, secure identity, backups and tested disaster recovery.

---

## 154. Production AWS Architecture

```text
GitLab
 ↓
OIDC
 ↓
AWS
 ↓
ECR
 ↓
EKS
 ↓
ALB
 ↓
Applications
```

---

## 155. Production GitOps Architecture

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

## 156. Production Observability

```text
Prometheus → Grafana
ELK        → Kibana
```

---

## 157. Disaster Recovery Interview

Question:

> How do you define DR?

Answer:

> I start with business RPO and RTO, identify critical state, design backups/replication, automate recovery where possible and regularly test restoration.

---

## 158. RPO

```text
maximum acceptable data loss
```

---

## 159. RTO

```text
maximum acceptable recovery time
```

---

## 160. Backup Testing

A backup is trustworthy only after successful restore testing.

---

## 161. High Availability Interview

Question:

> How do you remove single points of failure?

Answer:

> I map dependencies and failure domains, then add redundancy for components whose failure violates the required availability target. I validate the design through failure testing.

---

## 162. Scalability Interview

Question:

> How do you scale GitLab CI?

Answer:

> I measure peak concurrent jobs and queue time, then scale runner capacity horizontally, use autoscaling where appropriate and separate workload classes when isolation is required.

---

## 163. Security Architecture Interview

Question:

> How do you reduce CI/CD blast radius?

Answer:

> I use least privilege, environment isolation, protected branches, protected variables, dedicated runners for sensitive workloads, short-lived credentials and network restrictions.

---

## 164. GitOps Architecture Interview

Question:

> Why keep deployment state in Git?

Answer:

> Git provides reviewable, auditable and versioned desired state. ArgoCD can continuously reconcile that state with Kubernetes.

---

## 165. GitOps Drift Interview

Question:

> What happens when someone manually changes production?

Answer:

> The live cluster can drift from Git. ArgoCD detects the difference, and if automated self-healing is enabled it can restore the desired state. The permanent change should be represented in Git.

---

## 166. Terraform vs ArgoCD

Terraform:

```text
infrastructure
```

ArgoCD:

```text
Kubernetes application desired state
```

Avoid overlapping ownership.

---

## 167. GitLab vs Jenkins

A strong answer:

> Both can implement CI/CD. GitLab provides source control, merge requests, CI/CD, security and repository workflows in one platform, while Jenkins is primarily an automation server with a large plugin ecosystem.

---

## 168. GitLab vs GitHub Actions

A strong answer:

> Both provide repository-integrated CI/CD. The choice depends on organizational ecosystem, governance, integrations, runner model, security requirements and existing platform standards.

---

## 169. GitLab Runner vs Jenkins Agent

Both execute automation workloads, but their management model and ecosystem differ.

---

## 170. GitLab CI Interview Scenario

Question:

> A pipeline is green but deployment is unhealthy. What do you do?

Answer:

> I treat CI success and runtime health as separate signals. I verify the image digest, GitOps revision, ArgoCD state, Kubernetes rollout, Pod health, ingress and application metrics/logs.

---

## 171. Security Scan Passes but Production Is Vulnerable

Possible reasons:

```text
scanner coverage gap
new runtime configuration
dependency introduced elsewhere
false negative
```

Investigate the entire supply chain.

---

## 172. Pipeline Green but Image Wrong

Verify:

```text
commit
Docker build context
tag
digest
registry
GitOps manifest
```

---

## 173. ArgoCD Healthy but Application Broken

ArgoCD health does not prove business correctness.

Check:

```text
application metrics
logs
dependencies
smoke tests
```

---

## 174. Monitoring Green but Users Report Failure

Investigate:

```text
synthetic tests
business metrics
DNS
client path
regional issue
```

---

## 175. Alert Fatigue

Reduce:

```text
noise
duplicate alerts
non-actionable alerts
```

---

## 176. SLO-Based Alerting

Prefer alerts that indicate meaningful reliability degradation.

---

## 177. Incident Communication Interview

Answer:

> I communicate impact, current mitigation, known facts and next update time. I avoid speculation and keep technical and stakeholder communication appropriate to the audience.

---

## 178. RCA Interview

Answer:

> I document the timeline, technical root cause, contributing factors, detection gap, mitigation and corrective/preventive actions.

---

## 179. Blameless RCA

Focus on:

```text
system
process
controls
```

rather than individual blame.

---

## 180. Senior Scenario — Pipeline Suddenly Doubles in Duration

Approach:

```text
compare baseline
 ↓
check queue
 ↓
runner capacity
 ↓
dependency downloads
 ↓
cache
 ↓
build/test
 ↓
recent changes
```

---

## 181. Senior Scenario — All Runners Are Busy

Approach:

```text
confirm demand
 ↓
check job types
 ↓
check stuck jobs
 ↓
scale capacity
 ↓
check autoscaling
```

---

## 182. Senior Scenario — Production Runner Compromised

Approach:

```text
isolate
 ↓
revoke credentials
 ↓
preserve evidence
 ↓
rebuild runner
 ↓
audit
 ↓
verify
```

---

## 183. Senior Scenario — AWS OIDC Suddenly Fails

Check:

```text
OIDC token
trust policy
audience
subject claims
role
GitLab configuration
```

---

## 184. Senior Scenario — ECR Push Suddenly Fails

Check:

```text
caller identity
IAM
repository
region
authentication
network
```

---

## 185. Senior Scenario — ArgoCD Stops Syncing

Check:

```text
ArgoCD health
Git repository access
revision
application
RBAC
Kubernetes API
```

---

## 186. Senior Scenario — EKS Pods Cannot Pull Image

Check:

```text
image
registry
IAM
credentials
network
DNS
```

---

## 187. Senior Scenario — Production Pods Restart After Deployment

Check:

```text
Pod events
previous logs
resource usage
probes
configuration
dependency
```

---

## 188. Senior Scenario — High 5xx After Release

Correlate:

```text
release time
error rate
image digest
logs
dependency failures
```

Then rollback or fix forward based on evidence and safety.

---

## 189. Senior Scenario — Terraform Apply Failed Halfway

Do not blindly rerun.

Check:

```text
Terraform state
cloud resources
plan
dependency graph
```

---

## 190. Senior Scenario — State Lock Exists

Determine whether another Terraform operation is actually active before considering lock recovery.

---

## 191. Senior Scenario — GitOps and Terraform Both Manage Resource

Establish one authoritative owner and remove overlapping management.

---

## 192. Senior Scenario — Secret Appears in Job Log

Immediately:

```text
revoke
rotate
investigate
```

Then determine how the secret reached the log.

---

## 193. Senior Scenario — Pipeline Triggered Twice

Check:

```text
workflow rules
push
MR
webhook
schedule
trigger
```

---

## 194. Senior Scenario — Deployment Job Missing

Check:

```text
rules
workflow
environment protection
branch
pipeline source
```

---

## 195. Senior Scenario — Production Deployment Requires Approval

Explain:

> Production approval is a governance control. I verify the change, pipeline results, security status and deployment plan before approving rather than bypassing the control.

---

## 196. Senior Scenario — Developer Wants Direct Production kubectl Access

Explain:

> I would prefer GitOps and controlled operational access. If emergency access is required, it should be temporary, least-privileged and audited.

---

## 197. Senior Scenario — Need Emergency Production Fix

Approach:

```text
assess impact
 ↓
mitigate
 ↓
use approved break-glass process
 ↓
document
 ↓
reconcile Git
 ↓
verify
```

---

## 198. Project Discussion Framework

When asked:

> Tell me about your project.

Answer:

```text
1. Business/technical problem
2. Architecture
3. CI/CD
4. Infrastructure
5. Security
6. Deployment
7. Monitoring
8. Failure handled
9. Result
```

---

## 199. Project Architecture Answer

Example:

> I worked on a microservices platform deployed on AWS EKS. Terraform provisioned the infrastructure, GitLab handled CI/CD and security checks, ECR stored images, Helm packaged Kubernetes workloads, ArgoCD handled GitOps deployment, and Prometheus/Grafana plus ELK provided observability.

---

## 200. Project CI/CD Answer

Example:

> The pipeline validated code, ran tests and security checks, built Docker images, scanned them with Trivy, pushed immutable images to ECR and updated the GitOps deployment state.

---

## 201. Project Security Answer

Example:

> We integrated SonarQube, Trivy and Veracode into the DevSecOps workflow and protected production credentials and deployment paths using least privilege and protected environments.

---

## 202. Project GitOps Answer

Example:

> CI did not directly manage long-term Kubernetes desired state. After publishing the image, the deployment repository was updated and ArgoCD reconciled the desired state into EKS.

---

## 203. Project Monitoring Answer

Example:

> Prometheus collected metrics, Grafana provided dashboards and alerting, and ELK centralized application and infrastructure logs for troubleshooting.

---

## 204. Project Troubleshooting Answer

Example:

> When a deployment failed, I checked the pipeline logs first, then verified the image and GitOps revision, inspected ArgoCD status, Kubernetes events and Pod logs, and correlated the issue with application metrics.

---

## 205. Project Rollback Answer

Example:

> We rolled back through the GitOps desired state to a known-good immutable image/revision and verified recovery using Kubernetes health, application metrics and logs.

---

## 206. Project Challenge Answer

Use:

```text
Challenge
Action
Result
Learning
```

---

## 207. Project Failure Answer

Never say:

> Everything worked perfectly.

A stronger answer explains a real or controlled failure and how it was diagnosed.

---

## 208. Project Improvement Answer

Explain:

```text
what was weak
what you changed
why
result
```

---

## 209. Interview Coding — Parse Pipeline Status

Be able to write Python that:

```text
reads API response
filters failed pipelines
prints project/status
```

---

## 210. Interview Coding — Retry API Request

Implement:

```text
bounded retries
exponential backoff
429/5xx handling
```

---

## 211. Interview Coding — Parse Logs

Write a script to:

```text
read log
count errors
group by service
```

---

## 212. Interview Coding — Check Kubernetes Pods

Automate:

```text
list Pods
identify unhealthy Pods
report restart count
```

---

## 213. Interview Coding — AWS Health Check

Use AWS APIs to report:

```text
resource state
```

without hardcoding credentials.

---

## 214. Interview Coding — YAML Parsing

Know how to read and modify YAML safely with a parser rather than fragile string replacement.

---

## 215. Interview Coding — JSON Parsing

Know:

```python
import json
```

and safely access nested API responses.

---

## 216. Interview Coding — Shell

Know common patterns:

```bash
grep
awk
sed
cut
sort
uniq
xargs
jq
```

---

## 217. Interview Coding — Exit Codes

Understand:

```text
0 → success
non-zero → failure
```

and how CI uses exit codes.

---

## 218. Interview Coding — Idempotency

Explain why repeated execution should not create unintended duplicate changes.

---

## 219. Interview Coding — Logging

Use structured logging with:

```text
timestamp
level
context
error
```

---

## 220. Interview Coding — Error Handling

Distinguish:

```text
transient
permanent
configuration
authentication
```

errors.

---

## 221. GitLab Advanced Question — Include

Reusable pipeline configuration can be composed using GitLab's supported include mechanisms.

---

## 222. GitLab Advanced Question — Components

GitLab CI/CD components can help standardize reusable pipeline logic.

---

## 223. GitLab Advanced Question — Child Pipeline

Use child pipelines to split complex workflows.

---

## 224. GitLab Advanced Question — Parent Pipeline

A parent pipeline can orchestrate child workflows.

---

## 225. GitLab Advanced Question — Multi-Project Pipeline

Useful when separate repositories must coordinate delivery.

---

## 226. GitLab Advanced Question — Rules

Use rules to prevent unnecessary pipelines and control deployment behavior.

---

## 227. GitLab Advanced Question — `needs`

Use `needs` to reduce unnecessary stage waiting and build dependency-aware pipelines.

---

## 228. GitLab Advanced Question — Environments

Environments represent deployment targets such as:

```text
dev
staging
production
```

---

## 229. GitLab Advanced Question — Protected Environments

Use protection to control who can deploy to sensitive environments.

---

## 230. GitLab Advanced Question — Deployment Freeze

A freeze can prevent deployments during defined periods when configured.

---

## 231. GitLab Advanced Question — Artifacts

Artifacts preserve job outputs for later jobs or retention.

---

## 232. GitLab Advanced Question — Cache

Cache speeds repeated jobs by reusing suitable dependency data.

---

## 233. GitLab Advanced Question — Dependency Proxy

A dependency proxy can reduce repeated upstream image pulls and improve reliability where supported.

---

## 234. GitLab Advanced Question — Package Registry

GitLab can host supported package types in its package registry.

---

## 235. GitLab Advanced Question — Container Registry

GitLab can store OCI/container images.

---

## 236. GitLab Advanced Question — Release

A GitLab release associates a version/tag with release information and artifacts.

---

## 237. GitLab Advanced Question — Deployment

A deployment represents delivery of an application version to an environment.

---

## 238. GitLab Advanced Question — Environment

An environment represents the target where an application version runs.

---

## 239. GitLab Advanced Question — Variables Scope

Variables can be controlled by environment and protection settings where supported.

---

## 240. GitLab Advanced Question — Secret Exposure

Never:

```text
echo secret
commit secret
store secret in image
```

---

## 241. GitLab Advanced Question — Runner Isolation

Separate trust levels when workloads have different security requirements.

---

## 242. GitLab Advanced Question — Shared Runner Risk

A shared runner can increase cross-project risk if isolation is inadequate.

---

## 243. GitLab Advanced Question — Shell Runner Risk

Shell executor jobs run directly on the host, increasing blast radius.

---

## 244. GitLab Advanced Question — Kubernetes Runner Benefit

Kubernetes runners can provide ephemeral job environments and dynamic scaling.

---

## 245. GitLab Advanced Question — DinD

Docker-in-Docker can be used for container builds but introduces security and operational considerations.

---

## 246. GitLab Advanced Question — BuildKit

Modern container builds can use BuildKit-compatible approaches for performance and caching.

---

## 247. GitLab Advanced Question — Image Provenance

Track:

```text
source commit
builder
pipeline
image digest
```

for supply-chain traceability.

---

## 248. GitLab Advanced Question — SBOM

Generate and retain software bill of materials where required by the security program.

---

## 249. GitLab Advanced Question — Vulnerability Management

Prioritize findings based on:

```text
severity
exploitability
exposure
asset criticality
```

---

## 250. GitLab Advanced Question — Security Pipeline Failure

Determine whether failure is:

```text
real vulnerability
scanner problem
policy problem
network problem
```

---

## 251. GitLab Advanced Question — Quality Gate

A quality gate can prevent progression when defined quality criteria fail.

---

## 252. GitLab Advanced Question — MR Approval

Use approvals to enforce review requirements for sensitive changes.

---

## 253. GitLab Advanced Question — CODEOWNERS + Protected Branch

Together they can strengthen review and ownership controls.

---

## 254. GitLab Advanced Question — Auditability

Trace:

```text
user
commit
MR
pipeline
deployment
environment
```

---

## 255. GitLab Advanced Question — API Token Security

Use the smallest scope and shortest practical lifetime.

---

## 256. GitLab Advanced Question — Webhook Security

Validate webhook secrets and reject unexpected or malformed requests.

---

## 257. GitLab Advanced Question — Webhook Reliability

Use:

```text
retry
idempotency
queue
dead-letter handling
```

where appropriate.

---

## 258. GitLab Advanced Question — Rate Limit

Respect API limits and implement backoff.

---

## 259. GitLab Advanced Question — Pipeline Fan-Out

Use parallel jobs when independent services can build concurrently.

---

## 260. GitLab Advanced Question — Pipeline Fan-In

Use dependency jobs to collect outputs after parallel processing.

---

## 261. GitLab Advanced Question — Monorepo

Use path-based rules and dependency-aware pipelines to avoid unnecessary builds.

---

## 262. GitLab Advanced Question — Polyrepo

Centralize reusable CI templates while preserving service ownership.

---

## 263. GitLab Advanced Question — Shared CI Template Risk

A bad template change can break many projects.

Use:

```text
versioning
testing
canary rollout
```

---

## 264. GitLab Advanced Question — Production Template Change

Roll out to a small set of projects first.

---

## 265. GitLab Advanced Question — Pipeline DR

Keep pipeline configuration in Git and maintain external dependencies required to rebuild the delivery process.

---

## 266. GitLab Advanced Question — Runner DR

Maintain redundant runner capacity and documented recovery procedures.

---

## 267. GitLab Advanced Question — Artifact DR

Define which artifacts are reconstructable and which require durable retention.

---

## 268. GitLab Advanced Question — Registry DR

Understand image retention, replication and recovery requirements.

---

## 269. GitLab Advanced Question — GitLab Backup

Follow the supported backup and restore mechanisms for the selected GitLab deployment model.

---

## 270. Senior Architecture Question — Design a Complete DevOps Platform

Answer:

```text
GitLab
 ↓
CI/CD
 ↓
DevSecOps
 ↓
ECR
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
 ↓
ALB
 ↓
Applications
 ↓
Prometheus/Grafana + ELK
```

Explain ownership and failure boundaries at each layer.

---

## 271. Senior Architecture Question — Why Separate CI and CD?

Answer:

> Separating build/security from deployment reconciliation reduces coupling and allows GitOps to maintain an auditable desired state.

---

## 272. Senior Architecture Question — How Do You Reduce Blast Radius?

Answer:

```text
least privilege
environment isolation
protected runners
progressive deployment
immutable artifacts
network segmentation
```

---

## 273. Senior Architecture Question — How Do You Scale the Platform?

Answer:

```text
measure
 ↓
identify bottleneck
 ↓
scale appropriate layer
 ↓
validate
```

Do not scale every component blindly.

---

## 274. Senior Architecture Question — How Do You Design for Failure?

Answer:

```text
redundancy
timeouts
retries
backoff
health checks
autoscaling
rollback
backup
DR
```

---

## 275. Senior Architecture Question — How Do You Design Secure CI/CD?

Answer:

```text
identity
least privilege
secrets
protected branches
protected environments
security scanning
immutable artifacts
audit
```

---

## 276. Senior Architecture Question — How Do You Prevent Deployment Drift?

Answer:

> Store desired state in Git and continuously reconcile it through ArgoCD, while avoiding overlapping ownership between Terraform and GitOps.

---

## 277. Senior Architecture Question — How Do You Handle Emergency Changes?

Answer:

> Use the approved break-glass process, make the smallest safe change, document it, and immediately reconcile the permanent desired state back into Git.

---

## 278. Senior Architecture Question — What Makes a Pipeline Production Grade?

Answer:

```text
deterministic
secure
observable
reproducible
fast enough
recoverable
auditable
```

---

## 279. Senior Architecture Question — What Makes a DevOps Project Strong?

Answer:

> It demonstrates not only deployment, but security, infrastructure automation, observability, failure handling, rollback, recovery and clear operational ownership.

---

## 280. Interview Whiteboard — Complete Platform

Draw:

```text
                    Developer
                       │
                       ▼
                     GitLab
                       │
                 ┌─────┴─────┐
                 ▼           ▼
                MR           CI
                             │
                    ┌────────┼─────────┐
                    ▼        ▼         ▼
                  Test    Security    Build
                                      │
                                      ▼
                                     ECR
                                      │
                                      ▼
                                  GitOps Repo
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
                           ┌──────────┴──────────┐
                           ▼                     ▼
                       Prometheus               ELK
                           │                     │
                         Grafana               Kibana
```

---

## 281. Whiteboard — CI Boundary

Draw:

```text
Developer
 ↓
GitLab
 ↓
CI
 ├── Test
 ├── Security
 ├── Build
 └── Publish
```

Then explain why deployment state is separated.

---

## 282. Whiteboard — GitOps Boundary

```text
Image Registry
 ↓
GitOps Repository
 ↓
ArgoCD
 ↓
Kubernetes
```

---

## 283. Whiteboard — AWS Identity

```text
GitLab
 ↓
OIDC
 ↓
STS
 ↓
IAM Role
 ↓
AWS
```

---

## 284. Whiteboard — Observability

```text
Application
 ├── Metrics → Prometheus → Grafana
 └── Logs → ELK
```

---

## 285. Whiteboard — Failure Flow

```text
Alert
 ↓
Impact
 ↓
Metrics
 ↓
Logs
 ↓
Layer isolation
 ↓
Mitigation
 ↓
Verification
 ↓
RCA
```

---

## 286. Final Rapid-Fire Questions

Be ready to answer:

```text
What is GitLab?
What is a runner?
What is a pipeline?
What is a stage?
What is a job?
What is needs?
What are rules?
What is a protected branch?
What is an environment?
What is an artifact?
What is cache?
What is Docker executor?
What is Kubernetes executor?
What is OIDC?
What is ECR?
What is EKS?
What is Helm?
What is ArgoCD?
What is GitOps?
What is Terraform state?
What is SAST?
What is SCA?
What is Trivy?
What is Prometheus?
What is Grafana?
What is ELK?
```

---

## 287. Final Troubleshooting Rapid-Fire

Know how to troubleshoot:

```text
pipeline not created
pipeline pending
runner offline
job failure
cache failure
artifact failure
Docker build failure
ECR push failure
AWS AccessDenied
Terraform failure
Pod Pending
CrashLoopBackOff
OOMKilled
ImagePullBackOff
ALB 503
ALB 502
ArgoCD OutOfSync
ArgoCD SyncFailed
high 5xx
high latency
```

---

## 288. Final Scenario Rapid-Fire

Practice:

```text
production deployment failed
runner capacity exhausted
AWS OIDC broken
secret leaked
wrong image deployed
GitOps drift detected
Terraform drift detected
EKS node failed
database unavailable
registry unavailable
monitoring unavailable
```

---

## 289. Final Project Rapid-Fire

Be able to explain:

```text
architecture
why GitLab
why Terraform
why EKS
why ArgoCD
why Helm
why ECR
why OIDC
why SonarQube
why Trivy
why Prometheus
why Grafana
why ELK
```

---

## 290. Final Behavioral Technical Answer

When asked:

> What was the most difficult production issue you handled?

Use:

```text
Situation
Impact
Investigation
Root cause
Action
Recovery
Prevention
```

Never exaggerate responsibility or invent incidents.

---

## 291. Final Senior Answer Style

Prefer:

> “I would first…”

> “I would verify…”

> “The reason is…”

> “If that fails, I would…”

> “Once recovered, I would…”

This demonstrates engineering reasoning rather than memorization.

---

## 292. Final Interview Checklist

```text
[ ] Git fundamentals
[ ] GitLab projects/groups
[ ] Branches/MRs
[ ] CI/CD
[ ] YAML
[ ] rules
[ ] needs
[ ] artifacts
[ ] cache
[ ] runners
[ ] runner security
[ ] variables
[ ] secrets
[ ] Docker
[ ] ECR
[ ] OIDC
[ ] AWS
[ ] Terraform
[ ] EKS
[ ] Kubernetes
[ ] Helm
[ ] ArgoCD
[ ] GitOps
[ ] SonarQube
[ ] Trivy
[ ] Veracode
[ ] Prometheus
[ ] Grafana
[ ] ELK
[ ] API automation
[ ] Python
[ ] troubleshooting
[ ] rollback
[ ] DR
[ ] production architecture
[ ] project explanation
[ ] scenario questions
```

---

## 293. Final Interview Mental Model

```text
                    CODE
                      │
                      ▼
                   GITLAB
                      │
                      ▼
                    CI/CD
                      │
          ┌───────────┼───────────┐
          ▼           ▼           ▼
         TEST      SECURITY      BUILD
                                  │
                                  ▼
                                 ECR
                                  │
                                  ▼
                              GITOPS
                                  │
                                  ▼
                                ARGOCD
                                  │
                                  ▼
                                 EKS
                                  │
                                  ▼
                              APPLICATION
                                  │
                     ┌────────────┴────────────┐
                     ▼                         ▼
                  METRICS                    LOGS
                     ▼                         ▼
                PROMETHEUS                    ELK
                     ▼                         ▼
                  GRAFANA                   KIBANA
                     │                         │
                     └────────────┬────────────┘
                                  ▼
                           TROUBLESHOOTING
                                  │
                                  ▼
                            RECOVERY / RCA
```

---

## 294. Final Answer to “Tell Me About Your DevOps Experience”

A strong structure:

> I have worked across the DevOps lifecycle, including Linux, AWS, Docker, Kubernetes/EKS, Terraform, GitLab/Jenkins CI/CD, GitOps with ArgoCD, and DevSecOps tools such as SonarQube, Trivy and Veracode. I focus on automating infrastructure and delivery, deploying containerized microservices, implementing security checks and using Prometheus, Grafana and ELK for observability.

---

## 295. Final Answer to “What Is Your Strongest Area?”

A strong answer:

> My strongest area is production-oriented DevOps automation: connecting infrastructure as code, CI/CD, containerization, Kubernetes, GitOps and security into a reliable delivery workflow.

---

## 296. Final Answer to “What Do You Do When You Don't Know the Answer?”

A strong answer:

> I first clarify the failure or requirement, identify what I know, gather evidence through logs, metrics and documentation, test the safest hypothesis and then implement or escalate the solution. I avoid guessing in production.

---

## 297. Final Answer to “How Do You Approach Production?”

A strong answer:

> I prioritize reliability, security and observability. Any production change should be controlled, traceable, reversible where practical and monitored after deployment.

---

## 298. Final Interview Preparation Plan

Before interviews:

```text
Review concepts
 ↓
Practice commands
 ↓
Practice scenarios
 ↓
Practice project explanation
 ↓
Practice architecture diagrams
 ↓
Practice troubleshooting aloud
 ↓
Practice concise answers
```

---

## 299. Final GitLab Section Completion

The planned GitLab section is now complete:

```text
01 Fundamentals
02 Repository & Git Workflow
03 Branches & Merge Requests
04 CI/CD Fundamentals
05 CI/CD Configuration
06 Runners
07 Variables, Secrets & Environments
08 Artifacts, Cache & Dependencies
09 Docker & Container Registry
10 AWS Integration
11 Kubernetes & EKS
12 Terraform & IaC
13 ArgoCD & GitOps
14 DevSecOps
15 Security
16 Advanced CI/CD
17 Multi-Environment Deployments
18 Advanced Pipelines
19 API & Automation
20 Monitoring & Observability
21 Production Troubleshooting
22 Production Architecture
23 DevOps Projects
24 Interview Preparation
```

---

## 300. Final GitLab Interview Principle

> **Do not prepare GitLab as an isolated tool. Prepare it as a production DevOps platform.**

Your interview story should connect:

```text
Git
 ↓
GitLab
 ↓
CI/CD
 ↓
Security
 ↓
Docker
 ↓
ECR
 ↓
Terraform
 ↓
AWS
 ↓
EKS
 ↓
Helm
 ↓
ArgoCD
 ↓
GitOps
 ↓
Prometheus/Grafana
 ↓
ELK
 ↓
Troubleshooting
 ↓
Recovery
```

That complete chain demonstrates the ability to design, automate, secure, deploy, observe and troubleshoot a modern DevOps platform.

---

## Section Complete

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
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md ✓
```

**GitLab section complete.**
