# GitLab Fundamentals

> Production-oriented GitLab notes for DevOps/DevSecOps engineers. This file establishes the foundation for GitLab repositories, Merge Requests, CI/CD, Runners, security, AWS/EKS, Terraform, ArgoCD/GitOps, troubleshooting, and production architecture.

---

## 1. What Is GitLab?

GitLab is a DevOps platform built around Git that provides:

- Git source-code management
- Repository hosting
- Branches and Merge Requests
- Code review
- CI/CD
- Runners
- Package and container registries
- Security scanning
- Release and environment management
- APIs and automation
- Project/group administration

For a DevOps engineer, GitLab can become the central software-delivery control point:

```text
Developer
   ↓
Git Repository
   ↓
Merge Request
   ↓
GitLab CI/CD
   ↓
Build → Test → Security → Package
   ↓
Container Registry / ECR
   ↓
Deployment / GitOps
   ↓
Kubernetes / EKS
```

---

## 2. Git vs GitLab

### Git

Git is a distributed version-control system providing:

- commits
- branches
- merges
- rebases
- tags
- history
- local/remote repositories

Git does not provide a complete CI/CD platform by itself.

### GitLab

GitLab adds:

- repository hosting
- Merge Requests
- CI/CD
- Runners
- permissions
- protected branches
- protected environments
- registries
- security features
- APIs
- project/group management

### Interview answer

> Git is the version-control tool, while GitLab is a DevOps platform that provides Git repository hosting plus collaboration, CI/CD, security, registry, deployment, and automation capabilities.

---

## 3. GitLab Project

A project is the main workspace for an application, infrastructure repository, or automation workload.

It can contain:

- Git repository
- branches
- commits
- Merge Requests
- CI/CD configuration
- pipelines
- variables
- artifacts
- environments
- deployments
- issues
- registries
- security results

Example application:

```text
roboshop-application/
├── .gitlab-ci.yml
├── Dockerfile
├── src/
├── tests/
└── helm/
```

Example infrastructure project:

```text
roboshop-infrastructure/
├── modules/
├── environments/
├── backend.tf
├── providers.tf
├── main.tf
├── variables.tf
└── .gitlab-ci.yml
```

---

## 4. GitLab Groups

Groups organize multiple projects.

```text
Company
├── Platform
│   ├── terraform-infrastructure
│   ├── eks-platform
│   └── networking
├── Applications
│   ├── user
│   ├── cart
│   └── orders
└── DevSecOps
    ├── security-pipeline
    └── compliance-automation
```

Groups are useful for:

- centralized permissions
- shared CI/CD variables
- shared runners
- common policies
- project organization

---

## 5. Group vs Project

| Feature | Group | Project |
|---|---|---|
| Purpose | Organizes projects | Represents a workload/repository |
| Repository | Not the primary unit | Yes |
| Variables | Can provide shared variables | Project-specific variables |
| Permissions | Group membership | Project membership |
| Runners | Can share runners | Can use runners |
| Typical use | Team/platform organization | Application/infrastructure |

---

## 6. GitLab Roles

Common GitLab roles include:

- Guest
- Reporter
- Developer
- Maintainer
- Owner

Use least privilege.

Production principle:

```text
Minimum required access
        ↓
Protected branches
        ↓
Protected environments
        ↓
Approval controls
        ↓
Restricted deployment credentials
```

Do not give every engineer Maintainer/Owner access.

---

## 7. Repository Basics

GitLab repositories use normal Git commands:

```bash
git clone <repository>
git status
git branch
git switch
git add .
git commit -m "Add deployment configuration"
git push
git pull
```

GitLab adds collaboration and automation around Git.

---

## 8. Repository Authentication

Common approaches include:

- SSH keys
- Personal Access Tokens
- Project Access Tokens
- Group Access Tokens
- CI Job Tokens
- Deploy Tokens
- OAuth/SSO mechanisms

For automation, prefer purpose-specific identities and short-lived credentials where supported.

Never hard-code credentials in repositories.

---

## 9. SSH Authentication

Generate an Ed25519 key:

```bash
ssh-keygen -t ed25519 -C "devops@example.com"
```

Test GitLab connectivity:

```bash
ssh -T git@gitlab.com
```

For self-managed GitLab:

```bash
ssh -T git@gitlab.example.com
```

Only the public key should be uploaded.

Never expose:

```text
~/.ssh/id_ed25519
```

---

## 10. Personal Access Tokens

Personal Access Tokens can authenticate supported GitLab API/Git operations.

Production controls:

- minimum scopes
- expiration
- secure storage
- rotation
- auditability
- no source-code storage

Avoid using one highly privileged personal token for every automation.

---

## 11. Project Access Tokens

A Project Access Token is associated with a project.

Useful for:

```text
Project
   ↓
Automation identity
   ↓
GitLab API / repository / registry
```

Benefits include:

- project-scoped ownership
- reduced dependency on one employee
- controlled permissions

Still apply expiration, minimum scope, and secure storage.

---

## 12. Group Access Tokens

Group Access Tokens can support automation spanning multiple projects.

Use them only when the broader scope is actually required.

If automation operates on one project, prefer a project-scoped identity.

---

## 13. CI Job Token

GitLab CI jobs can receive a job token for supported GitLab operations.

```text
Pipeline
   ↓
Job
   ↓
CI Job Token
   ↓
Authorized GitLab resources
```

This can avoid storing a long-lived personal token in CI.

Cross-project access must be explicitly controlled.

---

## 14. Deploy Tokens

Deploy Tokens can support specific repository/package/registry access patterns.

Production requirements:

- limited scope
- expiration
- rotation
- secure storage
- no logging of credentials

---

## 15. GitLab CI/CD

GitLab CI/CD automates software delivery.

Basic flow:

```text
Commit
  ↓
Pipeline
  ↓
Build
  ↓
Test
  ↓
Security
  ↓
Package
  ↓
Deploy
  ↓
Verify
```

The pipeline is normally defined in:

```text
.gitlab-ci.yml
```

---

## 16. Basic `.gitlab-ci.yml`

```yaml
stages:
  - build
  - test

build:
  stage: build
  script:
    - echo "Building application"

test:
  stage: test
  script:
    - echo "Running tests"
```

The YAML describes the CI workflow.

---

## 17. Pipeline Stages

A production-oriented pipeline may use:

```yaml
stages:
  - validate
  - build
  - test
  - security
  - package
  - deploy
  - verify
```

Conceptually:

```text
validate
   ↓
build
   ↓
test
   ↓
security
   ↓
package
   ↓
deploy
   ↓
verify
```

Jobs in the same stage can run in parallel when their dependencies allow.

---

## 18. Job

A job is one unit of pipeline work.

```yaml
unit_test:
  stage: test
  script:
    - pytest
```

A job can define:

- stage
- script
- image
- variables
- artifacts
- cache
- rules
- dependencies/needs
- environment

---

## 19. GitLab Runner

GitLab Runner executes CI/CD jobs.

```text
GitLab
   │
   │ assigns job
   ▼
GitLab Runner
   │
   ▼
Execute job
   │
   ▼
Return logs/status
```

GitLab coordinates the pipeline; the Runner performs the actual job commands.

---

## 20. Runner Executors

Common executor types include:

- Shell
- Docker
- Kubernetes

### Shell

```text
GitLab
 ↓
Shell Runner
 ↓
Linux host
 ↓
Commands
```

Simple, but host sharing and privilege exposure require strong controls.

### Docker

```text
GitLab
 ↓
Runner
 ↓
Docker container
 ↓
Job
```

Provides a more reproducible job environment.

### Kubernetes

```text
GitLab
 ↓
Runner
 ↓
Kubernetes API
 ↓
Temporary Job Pod
```

Useful when the organization already operates Kubernetes/EKS.

---

## 21. Runner Security

A runner is sensitive because CI code executes on it.

A compromised job could potentially access:

- source code
- CI variables
- cloud credentials
- registries
- Kubernetes credentials
- deployment credentials

Use:

```text
Isolation
+
Least privilege
+
Trusted images
+
Protected jobs
+
Short-lived credentials
```

Avoid exposing production credentials to arbitrary pipelines.

---

## 22. Shared vs Dedicated Runners

### Shared Runner

Can serve multiple projects according to configuration.

### Dedicated/Specific Runner

Restricted to selected projects/workloads.

For production deployments:

```text
Protected Environment
        ↓
Trusted Pipeline
        ↓
Restricted Runner
        ↓
Production
```

---

## 23. CI/CD Variables

Variables provide configuration to jobs.

```yaml
variables:
  APP_ENV: "dev"
```

Access:

```bash
echo "$APP_ENV"
```

Do not put secrets directly into `.gitlab-ci.yml`.

Bad:

```yaml
variables:
  AWS_SECRET_ACCESS_KEY: "actual-secret"
```

Use protected/masked variables or workload identity.

---

## 24. Protected Variables

Production secrets should be restricted by:

- protected branch
- protected environment
- deployment job
- scope
- least privilege

Never print secrets:

```bash
echo "$SECRET"
```

Even masked variables can become exposed through unsafe commands or generated files, so applications should avoid logging secrets entirely.

---

## 25. Protected Branches

Important branches should be protected:

```text
main
production
release/*
```

Typical workflow:

```text
feature
   ↓
Merge Request
   ↓
Review
   ↓
CI
   ↓
main
```

Production branches should not allow arbitrary direct pushes.

---

## 26. Merge Requests

Merge Requests provide controlled integration.

```text
Feature Branch
      ↓
Merge Request
      ↓
Automated Tests
      ↓
Security Checks
      ↓
Code Review
      ↓
Approval
      ↓
Merge
```

MRs are an important DevSecOps control point.

---

## 27. Merge Request Pipeline

A strong MR pipeline can include:

```text
Lint
 ↓
Unit Tests
 ↓
Build
 ↓
SonarQube
 ↓
Trivy
 ↓
SCA
 ↓
Policy Gate
```

Untrusted MR code should not automatically receive production credentials.

---

## 28. GitLab Environments

Typical environments:

```text
development
staging
production
```

Pipeline:

```text
Build
 ↓
Development
 ↓
Staging
 ↓
Approval
 ↓
Production
```

Environments provide deployment visibility and control.

---

## 29. Protected Production Environment

Production should have stronger controls:

```text
production
   │
   ├── Protected
   ├── Approved deployers
   ├── Restricted jobs
   └── Production credentials
```

This prevents arbitrary CI jobs from deploying to production.

---

## 30. Artifacts

Artifacts are files produced by jobs.

```yaml
build:
  script:
    - mvn package
  artifacts:
    paths:
      - target/app.jar
```

Artifacts can contain:

- binaries
- test reports
- security reports
- Terraform plans
- generated files

---

## 31. Artifact vs Cache

### Artifact

Transfers or preserves job output:

```text
Build
 ↓
Artifact
 ↓
Deploy
```

### Cache

Speeds repeated work:

```text
Job
 ↓
Cache dependencies
 ↓
Later job
 ↓
Reuse cache
```

Cache should not be treated as the authoritative release artifact.

---

## 32. Production Pipeline for Your Stack

A realistic pipeline:

```text
GitLab
   ↓
Validate
   ↓
Maven / npm Build
   ↓
Unit Tests
   ↓
SonarQube
   ↓
Trivy
   ↓
Veracode
   ↓
Docker Build
   ↓
ECR
   ↓
GitOps Repository
   ↓
ArgoCD
   ↓
EKS
   ↓
ALB Ingress
   ↓
Smoke Test
```

This architecture will be expanded in later GitLab files.

---

## 33. GitLab + ECR

Typical flow:

```text
GitLab CI
    ↓
Docker Build
    ↓
Security Scan
    ↓
AWS Authentication
    ↓
ECR Login
    ↓
Push Image
    ↓
Image Digest
```

Prefer short-lived AWS federation/OIDC where supported rather than long-lived access keys.

---

## 34. GitLab + Terraform

Typical workflow:

```text
Merge Request
    ↓
terraform fmt
    ↓
terraform validate
    ↓
terraform plan
    ↓
Review
    ↓
Approval
    ↓
terraform apply
```

Production should not allow arbitrary branches to run unrestricted `terraform apply`.

---

## 35. GitLab + EKS

A practical flow:

```text
GitLab
   ↓
Build
   ↓
Security
   ↓
ECR
   ↓
GitOps
   ↓
ArgoCD
   ↓
EKS
```

GitLab can orchestrate the delivery workflow while ArgoCD remains the Kubernetes reconciliation layer.

---

## 36. GitLab + ArgoCD

Recommended boundary:

```text
GitLab CI
    ↓
Build + Test + Scan
    ↓
Publish Image
    ↓
Update GitOps Repository
    ↓
ArgoCD
    ↓
EKS
```

If ArgoCD owns production Kubernetes state, avoid using ad-hoc:

```bash
kubectl apply
```

as the normal deployment mechanism.

---

## 37. GitLab + DevSecOps

A production security pipeline can be:

```text
Source
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
Veracode / DAST where required
  ↓
Policy Gate
  ↓
Artifact
  ↓
Deployment
```

Security should be part of the delivery workflow.

---

## 38. Production Deployment Verification

Never treat:

```text
CI job succeeded
```

as:

```text
Application is healthy
```

Verify:

```text
CI Success
    ↓
Artifact Available
    ↓
GitOps Revision Updated
    ↓
ArgoCD Synced
    ↓
Kubernetes Rollout Healthy
    ↓
Pods Ready
    ↓
Service Endpoints Ready
    ↓
ALB Healthy
    ↓
Smoke Test Passed
```

---

## 39. GitLab API

GitLab APIs can automate:

- projects
- branches
- Merge Requests
- pipelines
- jobs
- variables
- environments
- deployments
- groups
- users where authorized

Python can be used as the automation layer.

Detailed API automation is covered later in:

```text
19-GitLab-API-and-Automation.md
```

---

## 40. GitLab Webhooks

Webhooks can notify external systems about:

- push
- Merge Request
- pipeline
- deployment
- tag

Security controls:

- HTTPS
- signature/token validation
- event filtering
- replay protection
- idempotency

---

## 41. Production Observability

Track:

- pipeline duration
- job duration
- failure rate
- retry count
- deployment frequency
- deployment success rate
- security gate failures
- runner queue time
- runner capacity

Integration concept:

```text
GitLab CI Logs
     ↓
ELK

CI/CD Metrics
     ↓
Prometheus
     ↓
Grafana
```

---

## 42. Production Security Boundaries

A useful model is:

```text
Developer Code
      ↓
Untrusted/Controlled CI
      ↓
Security Gates
      ↓
Trusted Immutable Artifact
      ↓
Protected Deployment
      ↓
Production
```

Production credentials should not automatically be available to every pipeline.

---

## 43. Common GitLab Mistakes

### Secrets in `.gitlab-ci.yml`

Never hard-code credentials.

### Production credentials in every job

Restrict them to protected deployment jobs.

### Unprotected production branch

Use branch protection and review.

### Mutable image tags

Prefer digest-based identity.

### Unlimited retries

Use retry budgets and deadlines.

### Privileged shared runner

Use isolation and restricted runners.

### Direct Kubernetes mutation

Respect GitOps ownership when ArgoCD is authoritative.

### Terraform apply without review

Use plan/approval controls.

### Security bypass

Do not disable security gates simply to make a pipeline pass.

---

## 44. Interview Questions

### Q1. What is GitLab?

> GitLab is a DevOps platform built around Git that provides source-code management, Merge Requests, CI/CD, Runners, security, registries, environments, APIs, and deployment capabilities.

### Q2. Git vs GitLab?

> Git is the version-control system. GitLab provides Git repository hosting plus collaboration, CI/CD, security, registry, deployment, and automation features.

### Q3. What is GitLab Runner?

> GitLab Runner is the execution agent that runs CI/CD jobs assigned by GitLab.

### Q4. What is `.gitlab-ci.yml`?

> It defines the GitLab pipeline including stages, jobs, scripts, variables, artifacts, rules, environments, and dependencies.

### Q5. How would you integrate GitLab with EKS?

> GitLab CI builds and scans the application, publishes an immutable image to ECR, updates the GitOps desired state, ArgoCD reconciles it into EKS, and the pipeline verifies rollout and application health.

### Q6. Why use ArgoCD if GitLab has CI/CD?

> GitLab can handle build, test, scan, and artifact delivery, while ArgoCD continuously reconciles Kubernetes desired state from Git. This creates a clear GitOps deployment ownership boundary.

### Q7. How do you protect production?

> Protected branches, protected environments, approvals, least-privilege credentials, trusted runners, immutable artifacts, security gates, and post-deployment verification.

### Q8. Why should production credentials not be available to every job?

> Because CI jobs execute code. If an untrusted job is compromised, broad credentials could provide a path to production.

---

## 45. Production Checklist

```text
[ ] Protected branches
[ ] Protected environments
[ ] Least-privilege access
[ ] No hard-coded secrets
[ ] Masked/protected variables where appropriate
[ ] Short-lived cloud credentials
[ ] Trusted runners
[ ] Runner isolation
[ ] Pipeline timeouts
[ ] Retry limits
[ ] Security gates
[ ] Immutable artifacts
[ ] Source SHA tracking
[ ] Image digest tracking
[ ] Terraform plan/approval
[ ] GitOps ownership defined
[ ] ArgoCD integration
[ ] Deployment verification
[ ] Smoke tests
[ ] Pipeline observability
[ ] Audit evidence
[ ] Rollback strategy
[ ] Documentation
```

---

## 46. Key Takeaway

A DevOps engineer should understand GitLab as more than Git hosting.

The complete delivery chain is:

```text
GitLab
  ↓
Git
  ↓
Merge Requests
  ↓
CI/CD
  ↓
Runners
  ↓
Security
  ↓
Artifacts
  ↓
ECR
  ↓
Terraform / GitOps
  ↓
ArgoCD
  ↓
EKS
  ↓
Verification
  ↓
Observability
```

> **GitLab provides the controlled software-delivery workflow where source, security evidence, artifacts, approvals, deployments, and verification can be connected and audited.**

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md                 ✓
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
└── 24-GitLab-Interview-Preparation.md
```

**Next: `02-GitLab-Repository-and-Git-Workflow.md`**
