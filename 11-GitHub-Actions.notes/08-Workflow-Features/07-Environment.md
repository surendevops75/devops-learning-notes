# GitHub Actions Environments

GitHub Actions Environments provide a way to define deployment targets such as:

```text
QA
SIT
UAT
Production
```

Environments can be used to control:

```text
Deployment Protection
Environment Secrets
Environment Variables
Approvals
Deployment Rules
Production Access
```

For production CI/CD, environments are especially important because they provide a controlled boundary between:

```text
CI
 ↓
Testing
 ↓
UAT
 ↓
Production
```

---

# Why Environments Matter

Without environment controls:

```text
Developer
   |
   ↓
GitHub Actions
   |
   ↓
Production
```

This creates a large risk.

With environments:

```text
GitHub Actions
      |
      ↓
Validation
      |
      ↓
Production Environment
      |
      ↓
Approval / Protection
      |
      ↓
Deployment
```

---

# Common Environments

A typical application may have:

```text
QA
SIT
UAT
Production
```

Example:

```text
Developer
   ↓
Feature Branch
   ↓
Pull Request
   ↓
CI
   ↓
QA
   ↓
SIT
   ↓
UAT
   ↓
Production
```

The exact promotion flow depends on the organization's release process.

---

# Environment Definition

GitHub environments are configured at the repository level.

Example names:

```text
qa
sit
uat
production
```

Use clear and consistent naming.

---

# Environment in a Job

Example:

```yaml
jobs:

  deploy:

    runs-on: ubuntu-latest

    environment:
      name: production

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

This associates the job with:

```text
production
```

---

# Dynamic Environment

An environment can be selected using an input.

Example:

```yaml
on:
  workflow_dispatch:

    inputs:

      environment:
        required: true
        type: choice
        options:
          - qa
          - uat
          - production

jobs:

  deploy:

    runs-on: ubuntu-latest

    environment:
      name: ${{ inputs.environment }}

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

This allows one workflow to target different environments.

---

# Be Careful with Dynamic Production Environments

A dynamic environment is powerful:

```text
Input
  ↓
Environment
  ↓
Secrets
  ↓
Deployment
```

Therefore, do not allow arbitrary user-controlled environment names to select privileged environments without appropriate validation and protection.

Prefer:

```yaml
type: choice
```

with an explicit allowlist.

---

# Environment Variables

Environments can have environment-specific variables.

Example:

```text
QA
 └── API_URL=https://qa.example.com

UAT
 └── API_URL=https://uat.example.com

Production
 └── API_URL=https://api.example.com
```

This avoids hardcoding environment-specific values in workflow files.

---

# Using Environment Variables

Example:

```yaml
jobs:

  deploy:

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Show API URL
        run: |
          echo "Deploying to $API_URL"
        env:
          API_URL: ${{ vars.API_URL }}
```

---

# Environment Secrets

Environment secrets allow sensitive values to be associated with a specific environment.

Example:

```text
QA
 └── QA_DEPLOY_TOKEN

UAT
 └── UAT_DEPLOY_TOKEN

Production
 └── PROD_DEPLOY_TOKEN
```

A production job can access the secrets associated with the production environment, subject to GitHub's environment and secret rules.

---

# Why Environment Secrets Matter

Without environment separation:

```text
One Production Secret
   |
   ↓
Many Workflows
```

This can make access control harder.

With environments:

```text
QA Workflow
   ↓
QA Secrets

UAT Workflow
   ↓
UAT Secrets

Production Workflow
   ↓
Production Secrets
```

---

# Environment vs Repository Secret

### Repository Secret

Available at repository scope according to GitHub's secret access rules.

### Environment Secret

Associated with a specific environment.

Example:

```text
Repository
 ├── General Secret
 │
 ├── QA Environment
 │    └── QA Secret
 │
 ├── UAT Environment
 │    └── UAT Secret
 │
 └── Production Environment
      └── Production Secret
```

Use environment-specific secrets when access should be tied to deployment targets.

---

# Environment Protection Rules

Production environments can be protected with rules such as:

```text
Required Reviewers
Deployment Branch Restrictions
Other Protection Rules
```

These controls can prevent a deployment from proceeding automatically.

---

# Required Reviewers

Example flow:

```text
Production Deployment
       |
       ↓
Approval Required
       |
       ↓
Reviewer
       |
   ┌───┴───┐
   ↓       ↓
Approve   Reject
   |       |
   ↓       ↓
Deploy    Stop
```

This is useful for controlled production releases.

---

# Production Approval

Example:

```text
Build
  ↓
Test
  ↓
Security
  ↓
UAT
  ↓
JIRA / CR Validation
  ↓
Production Environment
  ↓
Required Reviewer
  ↓
Production Deployment
```

This provides a manual authorization point.

---

# Environment Protection vs Input

Do not confuse:

```text
Workflow Input
```

with:

```text
Environment Protection
```

An input says:

```text
Where should this workflow attempt to deploy?
```

Environment protection says:

```text
Is this deployment allowed to proceed?
```

---

# Environment Branch Restrictions

Production environments can restrict which branches or tags can deploy.

Conceptually:

```text
Production
   |
   └── Allowed:
       main
       release/*
       approved tags
```

This helps prevent deployments from arbitrary development branches.

---

# Why Branch Restrictions Matter

Without restrictions:

```text
feature/test
   ↓
Production
```

could become possible if the workflow is poorly designed.

With restrictions:

```text
feature/test
   ↓
Blocked
```

while:

```text
main
   ↓
Production
```

is permitted according to the environment policy.

---

# Environment Protection Architecture

```text
                 Production
                     |
        ┌────────────┼────────────┐
        ↓            ↓            ↓
    Reviewers      Branches     Rules
        |
        ↓
   Authorization
        |
        ↓
    Deployment
```

---

# Environment and Secrets Flow

```text
Workflow
   |
   ↓
Select Environment
   |
   ↓
Environment Protection
   |
   ↓
Approval
   |
   ↓
Environment Secrets
   |
   ↓
Deployment
```

The workflow should not assume production secrets are available before the environment protection requirements are satisfied.

---

# Environment and Deployment

Example:

```yaml
jobs:

  deploy:

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
        run: |
          ./deploy.sh
```

---

# Environment URL

You can associate a deployment with a URL.

Example:

```yaml
environment:
  name: production
  url: https://example.com
```

This helps users identify the deployed environment.

---

# Dynamic Environment URL

Example:

```yaml
environment:
  name: ${{ inputs.environment }}
  url: ${{ steps.deploy.outputs.url }}
```

The exact implementation depends on how your deployment exposes the URL.

---

# Deployment Visibility

GitHub can associate workflow deployments with environments.

This provides visibility into:

```text
Which environment
Which workflow
Which deployment
Which commit
```

was involved.

---

# Environment and Commit SHA

A strong deployment trace is:

```text
Commit SHA
    ↓
Workflow Run
    ↓
Environment
    ↓
Deployment
```

Example:

```text
SHA:
8a92f31

Environment:
production
```

This helps troubleshooting and auditing.

---

# Environment and GitHub Actions

A production workflow may look like:

```yaml
name: Production Deployment

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:

  deploy:

    runs-on: ubuntu-latest

    environment:
      name: production

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Deploy
        run: |
          ./deploy.sh
```

The environment configuration is maintained separately in GitHub.

---

# Environment and Concurrency

Production deployments should usually avoid overlapping deployments.

Example:

```yaml
jobs:

  deploy:

    concurrency:
      group: production-deployment
      cancel-in-progress: false

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

This helps prevent:

```text
Deployment A
Deployment B
```

from modifying production simultaneously.

---

# Per-Service Concurrency

For microservices:

```yaml
concurrency:
  group: deploy-${{ inputs.service }}-${{ inputs.environment }}
  cancel-in-progress: false
```

This creates a separate concurrency group for each service/environment combination.

Example:

```text
catalogue-production
orders-production
payment-production
```

---

# Environment + Concurrency

These solve different problems.

### Environment

Controls:

```text
Authorization
Secrets
Deployment Target
Protection
```

### Concurrency

Controls:

```text
Overlapping Workflow Runs
```

Together:

```text
Environment
   +
Concurrency
   =
Safer Deployment
```

---

# Environment + Timeout

Add a job timeout:

```yaml
jobs:

  deploy:

    timeout-minutes: 30

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

Environment:

```text
Controls authorization
```

Timeout:

```text
Controls execution duration
```

---

# Environment + Artifacts

Production deployment can consume a validated artifact.

Example:

```text
Build
  ↓
Artifact
  ↓
Security
  ↓
UAT
  ↓
Production Environment
  ↓
Approval
  ↓
Deploy
```

This supports a Build Once, Promote Many model.

---

# Environment + ECR

For your containerized applications:

```text
Build
  ↓
Docker Image
  ↓
ECR
  ↓
Security
  ↓
UAT
  ↓
Production Environment
  ↓
Approval
  ↓
EKS
```

Use immutable image identifiers for production promotion where possible.

---

# Environment + Image Digest

Example:

```text
catalogue
   |
   ↓
ECR
   |
   ↓
sha256:abc123...
   |
   ↓
Production Environment
   |
   ↓
EKS
```

Using a digest makes the deployed image unambiguous.

---

# Environment + Helm

Example:

```yaml
jobs:

  deploy:

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Helm Deploy
        run: |
          helm upgrade --install catalogue ./helm/catalogue \
            --namespace production \
            --wait \
            --timeout 15m \
            --atomic
```

Environment protection happens at the GitHub deployment level.

Helm controls the Kubernetes deployment operation.

---

# Environment + Kubernetes

Flow:

```text
GitHub Environment
       |
       ↓
Authorization
       |
       ↓
GitHub Actions
       |
       ↓
AWS / EKS Authentication
       |
       ↓
Helm
       |
       ↓
Kubernetes
```

---

# Environment + AWS OIDC

For AWS, prefer short-lived OIDC credentials.

Conceptual flow:

```text
GitHub Actions
      |
      ↓
GitHub OIDC
      |
      ↓
AWS STS
      |
      ↓
IAM Role
      |
      ↓
EKS / ECR / AWS
```

The IAM trust policy should restrict which repositories/environments can assume the role.

---

# Production OIDC Model

```text
Production Environment
       |
       ↓
Approved Deployment
       |
       ↓
GitHub OIDC
       |
       ↓
AWS IAM Role
       |
       ↓
EKS
```

This is stronger than storing long-lived AWS access keys.

---

# Environment + JIRA Change Request

For your production process:

```text
Workflow
   |
   ↓
JIRA Ticket
   |
   ↓
Change Request
   |
   ↓
Approved?
   |
   ↓
Deployment Window?
   |
   ↓
Commit SHA Valid?
   |
   ↓
Tests Passed?
   |
   ↓
Security Passed?
   |
   ↓
Production Environment
   |
   ↓
Reviewer Approval
   |
   ↓
Deploy
```

The JIRA validation can be implemented in a reusable workflow or dedicated validation job.

---

# Production Environment as a Gate

Think of:

```text
production
```

as a controlled gate.

```text
              Production
                   |
          ┌────────┴────────┐
          ↓                 ↓
       Allowed             Blocked
          |
          ↓
       Approval
          |
          ↓
       Deploy
```

---

# Environment and Change Management

A production release should have:

```text
JIRA Ticket
Change Request
Approvals
Deployment Window
Commit SHA
Test Results
Security Results
Rollback Plan
```

The GitHub Environment provides one part of this control model.

---

# Environment and UAT

UAT can also be protected.

Example:

```text
UAT
 |
 ├── Required Reviewer
 └── UAT Secret
```

This can ensure that UAT deployment is controlled before production promotion.

---

# Environment Promotion

Example:

```text
QA
 ↓
SIT
 ↓
UAT
 ↓
Production
```

Each environment can have different:

```text
Secrets
Variables
Approvals
Deployment Rules
URLs
```

---

# Environment-Specific Configuration

Example:

```text
QA:
API_URL=https://qa-api.example.com

SIT:
API_URL=https://sit-api.example.com

UAT:
API_URL=https://uat-api.example.com

Production:
API_URL=https://api.example.com
```

The application deployment logic can remain the same while the environment configuration changes.

---

# Environment Variables vs Repository Variables

### Repository Variables

General repository-level configuration.

### Environment Variables

Configuration associated with a specific environment.

Example:

```text
Repository:
APP_NAME

Production:
API_URL
REGION
CLUSTER_NAME
```

Use environment scope when the value differs by deployment target.

---

# Environment Secrets vs Variables

### Variable

Generally non-sensitive configuration:

```text
REGION
CLUSTER_NAME
API_URL
```

### Secret

Sensitive value:

```text
TOKEN
PASSWORD
PRIVATE_KEY
```

Never use a normal variable for credentials.

---

# Environment Secrets and OIDC

With AWS OIDC, you may not need AWS access keys stored as secrets.

Instead:

```text
GitHub OIDC
   ↓
IAM Role
```

This reduces long-lived credential exposure.

---

# Environment and Least Privilege

Each environment should have only the permissions and secrets required for that environment.

Example:

```text
QA
 ↓
QA AWS Role

UAT
 ↓
UAT AWS Role

Production
 ↓
Production AWS Role
```

Do not automatically give QA workflows production access.

---

# Environment Isolation

Strong model:

```text
QA Workflow
   ↓
QA Role
   ↓
QA Resources

UAT Workflow
   ↓
UAT Role
   ↓
UAT Resources

Production Workflow
   ↓
Production Role
   ↓
Production Resources
```

---

# Environment and AWS IAM

Use separate roles where appropriate:

```text
github-actions-qa
github-actions-uat
github-actions-production
```

Each role should have permissions appropriate to its environment.

---

# Environment and EKS

Example:

```text
QA
 └── EKS QA Cluster / Namespace

UAT
 └── EKS UAT Cluster / Namespace

Production
 └── EKS Production Cluster / Namespace
```

The exact architecture depends on whether environments use separate clusters or namespaces.

---

# Environment + Namespace

A workflow can deploy to:

```text
namespace: qa
namespace: uat
namespace: production
```

But namespace separation alone is not always sufficient security isolation.

Use appropriate:

```text
AWS IAM
Kubernetes RBAC
Network Policies
Resource Policies
Environment Controls
```

---

# Environment + Kubernetes RBAC

Production deployment identity should have only the Kubernetes permissions required.

Conceptually:

```text
GitHub OIDC
    ↓
AWS IAM
    ↓
EKS Authentication
    ↓
Kubernetes RBAC
    ↓
Deployment Permissions
```

---

# Environment and Secrets Management

For larger production environments, consider whether secrets should remain in GitHub Environment Secrets or be managed by a dedicated secrets platform.

Examples of dedicated secret systems include:

```text
AWS Secrets Manager
HashiCorp Vault
External Secrets
```

The correct choice depends on architecture and organizational requirements.

---

# Environment + Secrets Manager

Example architecture:

```text
GitHub Actions
      |
      ↓
OIDC
      |
      ↓
AWS IAM
      |
      ↓
AWS Secrets Manager
      |
      ↓
Application / Deployment
```

This avoids placing long-lived application secrets directly into workflow files.

---

# Environment and GitOps

With GitOps:

```text
GitHub Environment
      |
      ↓
Production Authorization
      |
      ↓
GitOps Repository
      |
      ↓
ArgoCD
      |
      ↓
EKS
```

The GitHub Environment controls the workflow's production promotion.

ArgoCD remains responsible for reconciliation.

---

# Environment + ArgoCD

Example:

```text
GitHub Actions
   |
   ↓
Production Environment
   |
   ↓
Approval
   |
   ↓
Update GitOps Repository
   |
   ↓
ArgoCD
   |
   ↓
EKS
```

This separates:

```text
Release authorization
```

from:

```text
Cluster reconciliation
```

---

# Environment and Drift

ArgoCD can detect:

```text
Git Desired State
        vs
Cluster State
```

GitHub Environment does not replace ArgoCD drift detection.

They solve different problems.

---

# Environment and Rollback

A production environment does not automatically define your rollback strategy.

Rollback may use:

```text
Helm rollback
GitOps revert
Previous image digest
Previous release
```

Environment:

```text
Authorization
```

Rollback mechanism:

```text
Recovery
```

---

# Environment + Helm Atomic

Example:

```bash
helm upgrade --install catalogue ./helm/catalogue \
  --wait \
  --timeout 15m \
  --atomic
```

This gives Helm rollback behavior for failures during the Helm operation.

The GitHub Environment still controls whether the deployment is authorized to start.

---

# Environment + Diagnostics

If deployment fails:

```text
Production Environment
       |
       ↓
Deployment
       |
       ↓
Failure
       |
       ↓
Collect Diagnostics
       |
       ↓
Upload Artifact
```

Useful diagnostics:

```text
Pod Status
Pod Logs
Events
Helm Status
Helm History
Deployment Status
```

---

# Environment + Failure Handling

Example:

```yaml
- name: Collect Diagnostics
  if: ${{ always() }}
  run: |
    kubectl get pods -n production
    kubectl get events -n production \
      --sort-by=.lastTimestamp
```

Be careful not to expose sensitive information in logs or uploaded artifacts.

---

# Environment + Timeout

Example:

```yaml
jobs:

  deploy:

    environment:
      name: production

    timeout-minutes: 30

    steps:

      - name: Deploy
        run: |
          helm upgrade --install \
            catalogue ./helm/catalogue \
            --wait \
            --timeout 15m \
            --atomic
```

Hierarchy:

```text
GitHub Job
  ↓
30 minutes

Helm
  ↓
15 minutes
```

The outer timeout should allow enough time for the internal operation plus required verification.

---

# Environment + Concurrency + Timeout

Production-grade combination:

```yaml
jobs:

  deploy:

    concurrency:
      group: production-${{ inputs.service }}
      cancel-in-progress: false

    timeout-minutes: 30

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        run: |
          ./deploy.sh
```

This provides:

```text
Environment
→ Authorization

Concurrency
→ Prevent overlapping deployments

Timeout
→ Prevent indefinite execution
```

---

# Production Workflow

```yaml
name: Production Deployment

on:
  workflow_dispatch:

    inputs:

      service:
        required: true
        type: choice
        options:
          - user
          - catalogue
          - cart
          - orders
          - payment
          - inventory
          - notification

      image-digest:
        required: true
        type: string

      jira-ticket:
        required: true
        type: string

permissions:
  contents: read
  id-token: write

jobs:

  deploy:

    runs-on: ubuntu-latest

    timeout-minutes: 30

    concurrency:
      group: production-${{ inputs.service }}
      cancel-in-progress: false

    environment:
      name: production

    steps:

      - name: Checkout
        uses: actions/checkout@v4

      - name: Validate Inputs
        env:
          SERVICE: ${{ inputs.service }}
          IMAGE_DIGEST: ${{ inputs.image-digest }}
          JIRA_TICKET: ${{ inputs.jira-ticket }}
        run: |
          test -n "$SERVICE"
          test -n "$IMAGE_DIGEST"
          test -n "$JIRA_TICKET"

      - name: Validate Change Request
        env:
          JIRA_TICKET: ${{ inputs.jira-ticket }}
        run: |
          ./scripts/validate-change-request.sh "$JIRA_TICKET"

      - name: Configure AWS
        run: |
          ./scripts/configure-aws.sh

      - name: Deploy
        env:
          SERVICE: ${{ inputs.service }}
          IMAGE_DIGEST: ${{ inputs.image-digest }}
        run: |
          ./scripts/deploy.sh "$SERVICE" "$IMAGE_DIGEST"

      - name: Verify Deployment
        env:
          SERVICE: ${{ inputs.service }}
        run: |
          kubectl rollout status \
            deployment/"$SERVICE" \
            -n production \
            --timeout=10m

      - name: Collect Diagnostics
        if: ${{ always() }}
        run: |
          mkdir -p diagnostics

          kubectl get pods \
            -n production \
            > diagnostics/pods.txt

          kubectl get events \
            -n production \
            --sort-by=.lastTimestamp \
            > diagnostics/events.txt

      - name: Upload Diagnostics
        if: ${{ always() }}
        uses: actions/upload-artifact@v4
        with:
          name: production-diagnostics
          path: diagnostics/
```

This is an architectural example. The actual AWS authentication, JIRA validation, deployment script, and security controls should be implemented according to the organization's approved design.

---

# Production Environment Flow

```text
                    Production Request
                           |
                           ↓
                     JIRA / CR Check
                           |
                           ↓
                    Commit Validation
                           |
                           ↓
                    Security Results
                           |
                           ↓
                     Test Results
                           |
                           ↓
                GitHub Production Environment
                           |
                           ↓
                    Required Approval
                           |
                           ↓
                       Concurrency
                           |
                           ↓
                       Deployment
                           |
                           ↓
                    Kubernetes Health
                           |
                 ┌─────────┴─────────┐
                 ↓                   ↓
               PASS                FAIL
                 |                   |
                 ↓                   ↓
              Success             Diagnose
                                     |
                                     ↓
                                  Rollback
```

---

# Production Environment Checklist

```text
☐ QA environment configured
☐ SIT environment configured
☐ UAT environment configured
☐ Production environment configured
☐ Environment-specific variables configured
☐ Environment-specific secrets configured
☐ Least-privilege permissions configured
☐ Production required reviewers configured
☐ Production branch restrictions configured
☐ OIDC configured for AWS
☐ Separate AWS roles considered
☐ Production concurrency configured
☐ Deployment timeout configured
☐ JIRA/CR validation configured
☐ Security gates configured
☐ Test gates configured
☐ Immutable image reference used
☐ Rollback strategy documented
☐ Kubernetes health verification configured
☐ Failure diagnostics configured
☐ Sensitive data protected
```

---

# Common Mistakes

### 1. Using one environment for everything

```text
QA
UAT
Production
```

should not necessarily share the same credentials and controls.

### 2. Hardcoding environment configuration

Avoid:

```text
production URLs
credentials
cluster names
```

directly throughout workflow logic.

### 3. Using repository secrets for every environment

Prefer environment-scoped secrets when appropriate.

### 4. Giving production credentials to QA

This violates environment isolation.

### 5. Using long-lived AWS keys

Prefer OIDC.

### 6. No production approval

A critical production deployment should have appropriate authorization controls.

### 7. No concurrency

Two deployments may overlap.

### 8. No timeout

Deployment may run indefinitely.

### 9. Assuming environment means rollback

It does not.

### 10. Using dynamic environment names without validation

This can create unexpected access behavior.

---

# Best Practices

- Create separate environments for meaningful deployment stages.
- Use environment-specific secrets and variables.
- Protect production with appropriate reviewers.
- Restrict production deployment branches/tags.
- Use least-privilege permissions.
- Prefer OIDC for AWS authentication.
- Use separate cloud roles for different environments where appropriate.
- Use immutable image digests for production.
- Combine environments with concurrency.
- Combine environments with realistic timeouts.
- Validate JIRA/CR before production deployment where required.
- Keep security and test gates before production.
- Collect diagnostics after deployment failures.
- Keep rollback strategy separate from authorization controls.
- Use GitOps and ArgoCD according to the desired-state architecture.
- Never expose secrets in logs or artifacts.
- Treat production environment configuration as security-sensitive.

---

# Key Takeaways

GitHub Actions Environments provide a controlled deployment boundary.

They can provide:

```text
Environment Variables
Environment Secrets
Required Reviewers
Deployment Restrictions
Deployment Visibility
```

Typical environments:

```text
QA
SIT
UAT
Production
```

The core workflow syntax is:

```yaml
environment:
  name: production
```

For production, combine:

```text
Environment
+
Approval
+
Branch Restrictions
+
OIDC
+
Concurrency
+
Timeout
+
Security Gates
+
Test Gates
+
Rollback
```

For your DevOps architecture:

```text
GitHub Actions
      |
      ↓
Build
      |
      ↓
SonarQube
      |
      ↓
Trivy
      |
      ↓
Veracode
      |
      ↓
ECR
      |
      ↓
QA
      |
      ↓
SIT
      |
      ↓
UAT
      |
      ↓
JIRA / CR
      |
      ↓
Production Environment
      |
      ↓
Approval
      |
      ↓
GitOps
      |
      ↓
ArgoCD
      |
      ↓
EKS
```

Remember:

```text
Environment
→ Controls deployment target and protection

Secret
→ Protects sensitive data

Concurrency
→ Prevents overlapping deployments

Timeout
→ Limits execution duration

Approval
→ Provides authorization

Helm / GitOps
→ Performs deployment

Rollback
→ Recovers from failed deployment
```

The environment is one layer of a production deployment control system, not the entire deployment strategy.

---

# Interview Questions

## Basic

1. What is a GitHub Actions Environment?
2. Why are environments used?
3. What is an environment secret?
4. What is an environment variable?
5. How do you associate a job with an environment?
6. What is the difference between repository secrets and environment secrets?
7. What are required reviewers?
8. Why should production deployments be protected?
9. What is the purpose of deployment branch restrictions?
10. What is the difference between an environment and a workflow input?

## Intermediate

11. How would you configure QA, SIT, UAT, and Production environments?
12. How would you use environment-specific variables?
13. How would you use environment-specific secrets?
14. How would you configure a dynamic environment?
15. Why should dynamic environment names be validated?
16. How would you combine environments with concurrency?
17. How would you combine environments with timeout?
18. How would you use environments with Helm?
19. How would you use environments with EKS?
20. How would you use environments with ECR?
21. How would you use environments with GitOps and ArgoCD?
22. How would you use environment protection with JIRA change requests?
23. How would you configure AWS OIDC for production?
24. Why should QA not have production AWS credentials?
25. How would you implement production approval?

## Advanced / Production

26. Design a QA → SIT → UAT → Production promotion strategy.
27. How would you design environment isolation for an EKS platform?
28. How would you separate AWS IAM roles between QA, UAT, and Production?
29. How would you use GitHub Environments with AWS OIDC?
30. How would you protect a production deployment from untrusted branches?
31. How would you combine Environment, Concurrency, and Timeout?
32. How would you combine GitHub Environment approval with JIRA CR approval?
33. How would you validate a JIRA ticket before allowing production deployment?
34. How would you ensure only an approved image digest reaches production?
35. How would you design a production deployment using Helm `--atomic`?
36. How would you design production deployment through GitOps and ArgoCD?
37. How would you handle a production deployment that times out after partial rollout?
38. How would you collect Kubernetes diagnostics after a failed production deployment?
39. How would you separate deployment authorization from rollback logic?
40. How would you prevent environment secrets from being exposed to pull-request workflows?
41. How would you design environment-specific Kubernetes namespaces and AWS permissions?
42. How would you design production access using OIDC, IAM, EKS authentication, and Kubernetes RBAC?
43. How would you protect self-hosted runners used for production environments?
44. How would you design a production environment for multiple microservices with independent deployment concurrency?
45. Design an enterprise-grade GitHub Actions environment architecture covering QA, SIT, UAT, Production, environment secrets, variables, approvals, branch restrictions, JIRA/CR validation, OIDC, IAM, ECR, immutable image digests, Helm, ArgoCD, EKS, concurrency, timeouts, diagnostics, and rollback.