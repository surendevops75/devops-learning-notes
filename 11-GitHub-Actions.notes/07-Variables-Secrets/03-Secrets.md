# GitHub Actions Secrets

GitHub Actions Secrets are encrypted values used to store sensitive information required by workflows.

Typical examples:

```text
API Tokens
Passwords
Private Keys
Cloud Credentials
JIRA Tokens
Docker Credentials
Database Credentials
Certificates
Webhook Secrets
```

The fundamental rule is:

```text
Configuration → Variables
Sensitive Data → Secrets
Cloud Authentication → OIDC where supported
```

---

# Why Use Secrets?

Never hardcode sensitive values in workflow files.

Bad:

```yaml
env:
  DB_PASSWORD: MyPassword123
```

Bad:

```yaml
run: |
  curl -H "Authorization: Bearer abc123..."
```

Better:

```yaml
env:
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

Then:

```yaml
run: |
  ./deploy.sh
```

---

# Secret Flow

```text
GitHub Secret
      |
      ↓
Workflow
      |
      ↓
Job
      |
      ↓
Step
      |
      ↓
Process
```

Example:

```text
JIRA_API_TOKEN
      |
      ↓
JIRA Validation Action
      |
      ↓
JIRA API
```

---

# Types of Secrets

Secrets can be configured at different scopes.

Conceptually:

```text
Organization Secrets
        |
        ↓
Repository Secrets
        |
        ↓
Environment Secrets
```

Use the narrowest practical scope.

---

# Repository Secrets

Repository secrets are available to workflows in a specific repository, subject to the repository's settings and workflow context.

Example:

```text
Repository
   |
   └── Secrets
        ├── JIRA_API_TOKEN
        └── API_TOKEN
```

Workflow:

```yaml
env:
  JIRA_API_TOKEN: ${{ secrets.JIRA_API_TOKEN }}
```

---

# Organization Secrets

Organization-level secrets can be shared with selected repositories according to organization settings.

Conceptually:

```text
Organization
      |
      ├── Repository A
      ├── Repository B
      ├── Repository C
      └── Repository D
```

Useful for organization-wide integrations when the same sensitive value genuinely needs to be shared.

---

# Environment Secrets

GitHub Environments can have their own secrets.

Example:

```text
qa
 └── secrets

uat
 └── secrets

production
 └── secrets
```

This is useful when credentials or tokens differ between environments.

---

# Environment Secret Example

```yaml
jobs:

  deploy:

    environment:
      name: production

    runs-on: ubuntu-latest

    steps:

      - name: Deploy
        env:
          API_TOKEN: ${{ secrets.API_TOKEN }}
        run: |
          ./deploy.sh
```

The job is associated with the `production` environment.

---

# Environment Protection

GitHub Environments can be used for production deployment controls.

Conceptually:

```text
Production Job
      |
      ↓
Production Environment
      |
      ├── Environment Secrets
      ├── Protection Rules
      └── Required Approvals
      |
      ↓
Deployment
```

This is stronger than simply setting:

```text
ENVIRONMENT=production
```

---

# Secrets vs Variables

### Variable

```text
AWS_REGION=ap-south-1
```

Non-sensitive.

### Secret

```text
JIRA_API_TOKEN=******
```

Sensitive.

---

# Example

```yaml
env:
  AWS_REGION: ${{ vars.AWS_REGION }}
  JIRA_API_TOKEN: ${{ secrets.JIRA_API_TOKEN }}
```

The first is configuration.

The second is sensitive authentication data.

---

# Never Hardcode Secrets

Bad:

```yaml
env:
  PASSWORD: "SuperSecretPassword"
```

Bad:

```bash
curl -u user:password https://example.com
```

Bad:

```javascript
const token = "ghp_xxxxxxxxx";
```

Good:

```yaml
env:
  PASSWORD: ${{ secrets.PASSWORD }}
```

---

# Passing a Secret Through `env`

Example:

```yaml
steps:

  - name: Deploy
    env:
      API_TOKEN: ${{ secrets.API_TOKEN }}
    run: |
      ./deploy.sh
```

The command can access:

```text
$API_TOKEN
```

without putting the secret directly in the command.

---

# Passing a Secret Through `with`

Some Actions accept secret values as inputs.

Example:

```yaml
- name: Login
  uses: example/login-action@v1
  with:
    token: ${{ secrets.API_TOKEN }}
```

The Action receives the token through its defined input.

---

# Secret as Command Argument

This can be risky:

```yaml
run: |
  ./deploy.sh --token "${{ secrets.API_TOKEN }}"
```

Prefer passing secrets through environment variables or supported Action inputs when possible.

The exact security implications depend on the command and process behavior.

---

# Secrets in Shell

Example:

```yaml
steps:

  - name: Deploy
    env:
      API_TOKEN: ${{ secrets.API_TOKEN }}
    shell: bash
    run: |
      ./deploy.sh
```

The application can read:

```text
$API_TOKEN
```

---

# Secrets in PowerShell

Example:

```yaml
steps:

  - name: Deploy
    env:
      API_TOKEN: ${{ secrets.API_TOKEN }}
    shell: pwsh
    run: |
      ./deploy.ps1
```

PowerShell accesses it through:

```text
$env:API_TOKEN
```

---

# Secrets in Python

Example:

```yaml
env:
  API_TOKEN: ${{ secrets.API_TOKEN }}

steps:

  - name: Script
    run: |
      python deploy.py
```

Python:

```python
import os

token = os.environ["API_TOKEN"]
```

---

# Secrets in Node.js

Example:

```yaml
env:
  API_TOKEN: ${{ secrets.API_TOKEN }}
```

Node.js:

```javascript
const token = process.env.API_TOKEN;
```

---

# Secrets in Java

Java applications can access environment variables through:

```java
System.getenv("API_TOKEN");
```

The workflow can provide:

```yaml
env:
  API_TOKEN: ${{ secrets.API_TOKEN }}
```

---

# Secret Masking

GitHub Actions attempts to mask registered secrets when they appear in logs.

However:

```text
Do not rely on masking as a substitute for secure handling.
```

Never intentionally print secrets.

Bad:

```yaml
run: |
  echo "$API_TOKEN"
```

---

# Secret Masking Example

If a workflow needs to handle a sensitive value that is not already stored as a GitHub secret, GitHub Actions provides a masking mechanism through workflow commands.

Conceptually:

```text
Sensitive Value
      |
      ↓
Add Mask
      |
      ↓
Logs
      |
      ↓
Masked Value
```

Use this carefully and avoid generating secrets unnecessarily inside workflows.

---

# Do Not Transform Secrets Carelessly

Masking can be complicated when a secret is transformed.

For example:

```text
Original Secret
      ↓
Base64
      ↓
Encoded Value
```

The transformed value may not be automatically protected in the same way.

Avoid putting transformed secret values into logs.

---

# Secret in Debug Logs

Never do:

```yaml
run: |
  echo "TOKEN=$API_TOKEN"
```

Even when debugging.

Instead:

```yaml
run: |
  echo "Token is configured"
```

Do not reveal the actual value.

---

# Secret Names

Use descriptive names:

```text
JIRA_API_TOKEN
DOCKERHUB_TOKEN
DATABASE_PASSWORD
SLACK_WEBHOOK
API_TOKEN
```

Avoid:

```text
SECRET1
KEY
TOKEN
PASS
```

Clear names improve maintainability.

---

# Secret Naming Convention

A common convention:

```text
UPPERCASE_WITH_UNDERSCORES
```

Examples:

```text
AWS_ROLE_ARN
JIRA_API_TOKEN
DATABASE_PASSWORD
DOCKER_USERNAME
DOCKER_TOKEN
```

---

# Secret Rotation

Secrets should not be permanent.

Example:

```text
Secret Created
      |
      ↓
Used
      |
      ↓
Rotate
      |
      ↓
Update
      |
      ↓
Old Secret Revoked
```

---

# Why Rotate Secrets?

Rotation reduces the impact of:

```text
Accidental Exposure
Credential Theft
Long-Lived Credentials
Employee Changes
Third-Party Incidents
Security Events
```

---

# Secret Rotation Strategy

Example:

```text
Current Secret
      |
      ↓
Generate New Secret
      |
      ↓
Update GitHub Secret
      |
      ↓
Test
      |
      ↓
Revoke Old Secret
```

Do not revoke the old credential before confirming the new credential works when the underlying system requires overlap.

---

# Long-Lived Cloud Credentials

Avoid storing:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

as long-lived GitHub secrets when OIDC can be used.

Instead:

```text
GitHub Actions
      |
      ↓
OIDC
      |
      ↓
AWS IAM Role
      |
      ↓
Temporary Credentials
```

---

# OIDC

OIDC allows GitHub Actions to obtain short-lived cloud credentials without storing long-lived cloud credentials in GitHub.

Conceptually:

```text
Workflow
   |
   ↓
OIDC Token
   |
   ↓
Cloud Identity Provider
   |
   ↓
IAM Role
   |
   ↓
Temporary Credentials
```

---

# AWS OIDC Permissions

A workflow commonly needs:

```yaml
permissions:
  id-token: write
  contents: read
```

The exact permissions should match the workflow's needs.

---

# AWS OIDC Example

Conceptually:

```yaml
permissions:
  contents: read
  id-token: write

steps:

  - name: Configure AWS Credentials
    uses: aws-actions/configure-aws-credentials@v4
    with:
      role-to-assume: ${{ vars.AWS_ROLE_ARN }}
      aws-region: ${{ vars.AWS_REGION }}
```

The role ARN can be non-secret configuration.

The credentials are obtained through OIDC.

---

# OIDC Trust Policy

The AWS IAM trust relationship should restrict which GitHub repositories, branches, tags, or environments can assume the role.

Conceptually:

```text
GitHub Repository
      |
      ↓
OIDC
      |
      ↓
Trust Policy
      |
      ↓
IAM Role
```

Do not create an overly broad trust relationship.

---

# Environment-Based OIDC

For stronger production isolation:

```text
QA Workflow
   |
   ↓
QA Environment
   |
   ↓
QA IAM Role

PROD Workflow
   |
   ↓
Production Environment
   |
   ↓
Production IAM Role
```

This helps separate cloud permissions.

---

# JIRA Credentials

Example:

```yaml
env:
  JIRA_BASE_URL: ${{ vars.JIRA_BASE_URL }}
  JIRA_API_TOKEN: ${{ secrets.JIRA_API_TOKEN }}
```

Here:

```text
JIRA_BASE_URL
    ↓
Configuration

JIRA_API_TOKEN
    ↓
Sensitive Credential
```

---

# Production JIRA Validation

```text
Production Workflow
       |
       ↓
JIRA Validation Action
       |
       ├── JIRA_BASE_URL
       ├── JIRA_API_TOKEN
       └── JIRA Ticket
       |
       ↓
JIRA API
       |
       ↓
Approved?
```

The token should never appear in logs.

---

# Docker Registry Credentials

If a registry requires authentication:

```yaml
env:
  REGISTRY_USERNAME: ${{ secrets.REGISTRY_USERNAME }}
  REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
```

Then:

```bash
docker login \
  -u "$REGISTRY_USERNAME" \
  --password-stdin
```

Provide the password through standard input rather than putting it directly into the command line.

---

# GitHub Container Registry

For GitHub Container Registry workflows, use the appropriate token and permissions rather than manually creating long-lived credentials when the built-in GitHub token is sufficient.

Conceptually:

```text
GitHub Workflow
      |
      ↓
GITHUB_TOKEN
      |
      ↓
GHCR
```

---

# ECR Authentication

For AWS ECR, prefer OIDC-based AWS authentication where possible.

Flow:

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
ECR
```

This avoids storing long-lived AWS access keys.

---

# Database Credentials

Example:

```yaml
env:
  DB_HOST: ${{ vars.DB_HOST }}
  DB_USERNAME: ${{ secrets.DB_USERNAME }}
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

Separate:

```text
Host → Configuration
Username → Sensitive
Password → Sensitive
```

depending on your organization's security requirements.

---

# Private SSH Keys

Private keys should be treated as secrets.

Example:

```text
SSH_PRIVATE_KEY
```

Do not commit:

```text
id_rsa
id_ed25519
```

to the repository.

---

# SSH Key Handling

If a workflow genuinely requires SSH:

```yaml
env:
  SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

Then securely configure the SSH client.

Never:

```bash
echo "$SSH_PRIVATE_KEY"
```

to logs.

---

# Certificates

Certificates and private keys may require secret handling.

For example:

```text
TLS_PRIVATE_KEY
TLS_CERTIFICATE
```

Store sensitive material using an appropriate secret mechanism.

For larger or complex certificate bundles, an external secret manager may be more suitable.

---

# Webhook Secrets

Webhook signing secrets should be stored as secrets.

Example:

```text
SLACK_WEBHOOK_SECRET
```

Do not put webhook URLs or signing credentials directly in workflow source when they are sensitive.

---

# Third-Party API Tokens

Examples:

```text
JIRA_API_TOKEN
VERACODE_API_ID
VERACODE_API_KEY
SLACK_WEBHOOK
SONAR_TOKEN
```

Store sensitive credentials as secrets.

---

# Security Tool Credentials

For a DevSecOps pipeline:

```text
SonarQube
Trivy
Veracode
```

Some tools may require credentials or tokens.

Example:

```yaml
env:
  SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
  VERACODE_API_KEY: ${{ secrets.VERACODE_API_KEY }}
```

Keep sensitive credentials outside workflow source.

---

# Secret Scope

Use the smallest practical scope.

Example:

```text
Organization Secret
        ↓
Many Repositories

Repository Secret
        ↓
One Repository

Environment Secret
        ↓
Specific Deployment Environment
```

If only production requires a credential:

```text
Production Environment Secret
```

is preferable to a broadly shared organization secret.

---

# Principle of Least Privilege

Apply least privilege to both:

```text
Secrets
Permissions
Cloud Roles
Repositories
Environments
```

Example:

```text
QA Deployment
   ↓
QA Credentials

PROD Deployment
   ↓
PROD Credentials
```

Do not share production credentials with QA unnecessarily.

---

# Production Secret Isolation

Recommended:

```text
QA
 ├── QA secrets
 └── QA permissions

UAT
 ├── UAT secrets
 └── UAT permissions

PROD
 ├── PROD secrets
 └── PROD permissions
```

---

# Secrets and Pull Requests

Be careful when workflows run for:

```text
pull_request
```

especially when code comes from forks.

Do not assume untrusted pull-request code should receive sensitive credentials.

---

# Fork Security

Conceptually:

```text
External Contributor
       |
       ↓
Fork
       |
       ↓
Pull Request
       |
       ↓
Workflow
```

The workflow should not expose sensitive credentials to untrusted code.

---

# `pull_request_target`

`pull_request_target` has access to the base repository context and can have access to secrets, depending on the workflow.

It must be used carefully.

Never blindly execute untrusted fork code with privileged credentials.

Bad security pattern:

```text
Untrusted PR Code
       |
       ↓
Privileged Token
       |
       ↓
Production API
```

---

# Secret Access in Forks

For security reasons, workflows triggered from forks have restrictions around secrets.

Do not design CI pipelines assuming that repository secrets are freely available to forked pull requests.

---

# Dependabot

Automated dependency update workflows also require careful secret handling.

Do not give unnecessary production credentials to dependency-update workflows.

Use minimal permissions.

---

# Secrets and Self-Hosted Runners

Self-hosted runners require additional care.

```text
Secret
  |
  ↓
Self-Hosted Runner
  |
  ↓
Job
```

If the runner environment is compromised, secrets available to the job may be at risk.

---

# Self-Hosted Runner Security

Use:

```text
Ephemeral Runners
Runner Groups
Labels
Network Isolation
Minimal Permissions
Environment Separation
```

Avoid using a permanently shared runner for untrusted workloads.

---

# Production Runner Isolation

Example:

```text
CI Runner
  |
  └── Build / Test

Security Runner
  |
  └── Security Scanning

Production Runner
  |
  └── Production Deployment
```

Keep privileged deployment workloads isolated.

---

# ARC and Secrets

With Actions Runner Controller:

```text
GitHub
   |
   ↓
ARC
   |
   ↓
Ephemeral Runner
   |
   ↓
Workflow
   |
   ↓
Secret
```

Ephemeral runners can reduce persistence of sensitive workload data after a job finishes.

---

# Secrets and Logs

Potential secret leakage locations:

```text
Workflow Logs
Error Messages
Debug Output
Command Arguments
Generated Files
Artifacts
Docker Build Logs
Application Logs
```

Review all of these.

---

# Secret in Artifact

Bad pattern:

```text
Generate config
   |
   ↓
config.env
   |
   ↓
Upload Artifact
```

If the file contains secrets, it can become an accidental credential distribution mechanism.

Avoid uploading secrets as artifacts.

---

# Secret in Docker Image

Never bake secrets into Docker images.

Bad:

```dockerfile
ENV API_TOKEN=secret
```

Bad:

```dockerfile
COPY .env /app/.env
```

Secrets embedded in images can remain accessible through image layers.

---

# Runtime Secret Injection

Prefer:

```text
Secret Store
     |
     ↓
Runtime
     |
     ↓
Application
```

rather than:

```text
Secret
  |
  ↓
Docker Image
```

---

# Kubernetes Secrets

For Kubernetes workloads:

```text
GitHub Actions
      |
      ↓
Secret Management
      |
      ↓
Kubernetes
      |
      ↓
Pod
```

Use an appropriate Kubernetes/external secret management architecture.

Do not casually commit raw production credentials into Kubernetes YAML.

---

# GitOps and Secrets

A GitOps repository should not contain plaintext production secrets.

Instead use mechanisms such as:

```text
External Secrets
Sealed Secrets
Cloud Secret Manager
Vault
Other approved secret-management systems
```

according to organizational architecture.

---

# GitOps Security Model

```text
Git Repository
      |
      ├── Application Configuration
      └── Secret References
                 |
                 ↓
          Secret Manager
                 |
                 ↓
               EKS
```

This separates:

```text
Configuration
```

from:

```text
Secret Material
```

---

# Secret Scanning

Use secret scanning to detect accidental credential commits.

Possible controls:

```text
GitHub Secret Scanning
Push Protection
TruffleHog
Gitleaks
Other approved scanners
```

---

# DevSecOps Secret Flow

A mature pipeline:

```text
Developer
    |
    ↓
Git Commit
    |
    ↓
Secret Scan
    |
    ↓
Build
    |
    ↓
SAST
    |
    ↓
SCA
    |
    ↓
Container Scan
    |
    ↓
Deploy
```

Secret detection should happen as early as practical.

---

# Secret Scanning Failure

If a secret is detected:

```text
Commit
  |
  ↓
Secret Scan
  |
  ↓
Detected
  |
  ↓
Block Pipeline
```

If the secret was already exposed:

```text
Detect
  |
  ↓
Revoke
  |
  ↓
Rotate
  |
  ↓
Investigate
  |
  ↓
Remove From History if Appropriate
```

Removing a secret from Git history does not replace revocation.

---

# Secret Exposure Incident

If a production credential is exposed:

```text
1. Revoke credential
2. Rotate credential
3. Identify affected systems
4. Review access logs
5. Investigate exposure
6. Remove accidental exposure
7. Update workflows
8. Add preventive controls
```

Do not assume deleting the Git commit is enough.

---

# Secret Rotation Automation

Conceptually:

```text
Secret Manager
      |
      ↓
Generate New Secret
      |
      ↓
Update Consumer
      |
      ↓
Validate
      |
      ↓
Revoke Old Secret
```

For critical production systems, design rotation with an overlap period where supported.

---

# Secret Manager Integration

Instead of storing every secret directly in GitHub:

```text
GitHub Actions
      |
      ↓
OIDC
      |
      ↓
AWS Secrets Manager / Vault / Approved Secret Store
      |
      ↓
Secret
```

This can centralize:

```text
Rotation
Audit
Access Control
Secret Lifecycle
```

---

# External Secret Manager

Useful for:

```text
Database Passwords
API Credentials
Certificates
Application Secrets
Cloud Credentials
Third-Party Tokens
```

GitHub Actions can authenticate to the secret manager using a short-lived identity mechanism where supported.

---

# Production Secret Architecture

```text
                  GitHub Actions
                        |
             ┌──────────┴──────────┐
             ↓                     ↓
        GitHub Secrets          OIDC
             |                     |
             ↓                     ↓
       Small GitHub-scoped     Cloud IAM
       sensitive values            |
                                   ↓
                            Secret Manager
                                   |
                                   ↓
                              Application
```

Use the appropriate storage mechanism based on the credential's lifecycle and access requirements.

---

# Secrets and Permissions

Secrets and permissions are separate controls.

Example:

```yaml
permissions:
  contents: read
```

does not mean:

```text
all secrets are inaccessible
```

Secret availability and token permissions must be designed independently.

---

# Secrets and `GITHUB_TOKEN`

GitHub automatically provides a `GITHUB_TOKEN` for workflows.

Do not create a personal access token when the built-in token is sufficient.

Use minimal permissions.

Example:

```yaml
permissions:
  contents: read
```

For a job that needs to write:

```yaml
permissions:
  contents: write
```

only where required.

---

# Personal Access Tokens

If a PAT is genuinely required:

```text
Store it as a secret
Use minimum required permissions
Prefer fine-grained tokens where appropriate
Rotate regularly
Avoid long-lived credentials where possible
```

But first determine whether:

```text
GITHUB_TOKEN
```

or:

```text
GitHub App
```

can satisfy the requirement.

---

# GitHub App Authentication

For organization-level automation, a GitHub App can be preferable to a long-lived personal access token.

Conceptually:

```text
Workflow
   |
   ↓
GitHub App
   |
   ↓
Installation Token
   |
   ↓
GitHub API
```

This can provide more controlled permissions and lifecycle management.

---

# Secrets in Reusable Workflows

Reusable workflows can receive secrets explicitly.

Conceptually:

```text
Caller Workflow
       |
       ↓
Reusable Workflow
       |
       ↓
Secret
```

Only pass secrets that the reusable workflow actually needs.

---

# Secrets and Custom Actions

A custom Action should clearly document:

```text
Required Secrets
Purpose
Permissions
API Access
```

Example:

```text
JIRA_API_TOKEN
Purpose: JIRA API authentication
Required by: jira-validation Action
```

Do not make consumers provide unnecessary credentials.

---

# Custom Action Security

A custom Action should:

```text
Validate Inputs
Avoid Logging Secrets
Use Minimal Permissions
Handle API Errors
Protect Dependencies
Document Secrets
Use Secure Authentication
```

---

# Production JIRA Action

Example architecture:

```text
Production Workflow
        |
        ↓
JIRA Validation Action
        |
        ├── JIRA_BASE_URL
        ├── JIRA_API_TOKEN
        └── JIRA_TICKET
        |
        ↓
JIRA API
        |
        ↓
Validation
        |
        ↓
PASS / FAIL
```

Only:

```text
JIRA_API_TOKEN
```

is sensitive.

---

# Production CR Validation

A custom Action can validate:

```text
JIRA Ticket
Project
Component
Version / SHA
Approvals
Testing Results
Security Results
Deployment Window
Rollback Plan
```

Credentials remain protected.

---

# Secret + Production Environment

Strong pattern:

```text
Production Job
      |
      ↓
environment: production
      |
      ├── Production Secrets
      ├── Production Variables
      └── Approval Rules
      |
      ↓
Deployment
```

This helps keep production configuration isolated.

---

# Secret + EKS

For GitOps:

```text
GitHub Actions
      |
      ↓
Update Git Manifest
      |
      ↓
ArgoCD
      |
      ↓
EKS
      |
      ↓
External Secret
      |
      ↓
Secret Manager
```

The CI workflow does not need to hold every application secret.

---

# Secret + ECR

For ECR:

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
ECR
```

Prefer this over:

```text
AWS Access Key Secret
AWS Secret Key Secret
```

when OIDC is available and suitable.

---

# Secret + Terraform

Terraform may require cloud authentication.

Prefer:

```text
GitHub OIDC
     |
     ↓
AWS IAM
```

rather than storing long-lived AWS keys.

Be careful with Terraform state because state can contain sensitive values.

---

# Terraform State

Important:

```text
Terraform State
      |
      ↓
May contain sensitive data
```

Protect the state backend using:

```text
Encryption
Access Control
Restricted IAM
Remote Backend
State Locking
Audit
```

Do not treat GitHub secrets as a solution for securing Terraform state.

---

# Secrets in Terraform Variables

Example:

```text
TF_VAR_database_password
```

can pass sensitive data to Terraform.

But remember:

```text
Sensitive Terraform Variable
       |
       ↓
Terraform State
```

The value may still appear in state.

Secure the backend appropriately.

---

# Secrets and Docker Build

Avoid:

```bash
docker build --build-arg API_TOKEN="$API_TOKEN" .
```

for persistent credentials unless you fully understand the build mechanism and secret handling.

Prefer Docker BuildKit secret mechanisms when build-time secrets are genuinely required.

Conceptually:

```text
Secret
  |
  ↓
BuildKit Secret
  |
  ↓
Build
  |
  ↓
Secret not persisted in final image
```

---

# Secret Lifecycle

```text
Create
  |
  ↓
Store
  |
  ↓
Access
  |
  ↓
Use
  |
  ↓
Rotate
  |
  ↓
Revoke
  |
  ↓
Delete
```

Every production secret should have a lifecycle.

---

# Secret Ownership

Document:

```text
Secret
Owner
Purpose
Consumers
Rotation Frequency
Emergency Contact
```

Example:

| Secret | Purpose | Owner | Scope |
|---|---|---|---|
| JIRA_API_TOKEN | JIRA API | Release Team | Production |
| VERACODE_API_KEY | Security Scan | DevSecOps | CI |
| DB_PASSWORD | Database | Platform | Production |

---

# Secret Inventory

Maintain an inventory for important secrets:

```text
Name
Purpose
Owner
Scope
Rotation
Last Rotated
Consumers
```

This helps identify:

```text
Unused Secrets
Expired Secrets
Overly Broad Secrets
Unknown Owners
```

---

# Secret Governance

Enterprise governance can define:

```text
Naming
Ownership
Rotation
Scope
Access
Approval
Audit
Incident Response
```

---

# Production Secret Checklist

```text
☐ Secret is not hardcoded
☐ Correct scope selected
☐ Production secret isolated
☐ Least privilege
☐ No secret logging
☐ Rotation process defined
☐ Owner assigned
☐ Secret scanning enabled
☐ Runner security reviewed
☐ External secret manager evaluated
☐ OIDC used where appropriate
☐ Incident response defined
```

---

# Common Mistakes

### 1. Hardcoding credentials

Never commit secrets.

### 2. Printing secrets

Never intentionally print them.

### 3. Using production secrets in CI unnecessarily

Keep privileged credentials isolated.

### 4. Using long-lived cloud credentials

Prefer OIDC where supported.

### 5. Passing secrets to untrusted PR code

Do not expose credentials to untrusted workflows.

### 6. Baking secrets into Docker images

Never embed production credentials in images.

### 7. Uploading secret-containing files as artifacts

Review artifact contents.

### 8. Sharing organization secrets too broadly

Use narrow scope.

### 9. No rotation process

Every important secret needs a lifecycle.

### 10. Assuming masking solves everything

Prevent exposure rather than relying on log masking.

---

# Best Practices

- Never hardcode secrets.
- Use GitHub Secrets for appropriate sensitive values.
- Use environment secrets for environment-specific credentials.
- Use organization secrets only when broad sharing is actually required.
- Prefer OIDC for cloud authentication where supported.
- Use least-privilege permissions.
- Never print secrets.
- Avoid passing secrets to untrusted code.
- Do not bake secrets into Docker images.
- Do not commit plaintext Kubernetes secrets.
- Protect Terraform state.
- Use secret scanning.
- Rotate credentials.
- Revoke exposed credentials immediately.
- Consider an external secret manager for critical production secrets.
- Document secret ownership.
- Keep production credentials isolated.
- Review self-hosted runner security.
- Use ephemeral runners where appropriate.
- Use GitHub Environments for production protection.

---

# Production-Level Secret Architecture

```text
                         GitHub Actions
                               |
              ┌────────────────┼────────────────┐
              ↓                ↓                ↓
       GitHub Secrets       GitHub Vars        OIDC
              |                |                |
              ↓                ↓                ↓
       Sensitive Values   Configuration      Cloud IAM
              |                                  |
              |                                  ↓
              |                           Secret Manager
              |                                  |
              └────────────────┬─────────────────┘
                               ↓
                         Deployment Job
                               |
                               ↓
                         Production
                               |
                               ↓
                              EKS
```

---

# Production CI/CD Example

```text
Developer
    |
    ↓
GitHub
    |
    ↓
CI
 ├── Build
 ├── SonarQube
 ├── Trivy
 └── Veracode
    |
    ↓
ECR
    |
    ↓
UAT
    |
    ↓
E2E Tests
    |
    ↓
Production Gate
 ├── JIRA
 ├── CR
 ├── Approval
 ├── SHA Validation
 └── Deployment Window
    |
    ↓
Production
    |
    ↓
ArgoCD
    |
    ↓
EKS
```

Secrets should only be exposed to the specific stages that genuinely require them.

---

# Key Takeaways

```text
Secrets
=
Sensitive values required by workflows.
```

Use:

```text
Variables
    ↓
Non-sensitive configuration

Secrets
    ↓
Sensitive credentials

OIDC
    ↓
Short-lived cloud identity

External Secret Manager
    ↓
Centralized production secret lifecycle
```

Remember:

```text
Never hardcode
Never log
Never bake into images
Never expose to untrusted code
Never give broader access than required
```

The most important production principle:

```text
A secret should be accessible only to the job,
environment, identity, and process that actually needs it.
```

---

# Interview Questions

## Basic

1. What are GitHub Actions Secrets?
2. Why should we use secrets instead of variables for credentials?
3. What is the difference between repository secrets and environment secrets?
4. What are organization secrets?
5. How do you access a secret in a workflow?
6. How do you pass a secret to a shell command?
7. How do you pass a secret to an Action?
8. How do you prevent secrets from appearing in logs?
9. What is secret masking?
10. Why should you never hardcode credentials in workflow files?

## Intermediate

11. What is the difference between secrets, variables, inputs, and outputs?
12. How do environment-specific secrets work?
13. How would you manage separate QA, UAT, and production credentials?
14. How would you rotate GitHub secrets?
15. What happens if a secret is accidentally committed to Git?
16. How would you respond to an exposed production credential?
17. How would you securely use JIRA credentials in GitHub Actions?
18. How would you securely authenticate with ECR?
19. How would you securely provide Docker registry credentials?
20. How do secrets interact with self-hosted runners?
21. Why should secrets not be baked into Docker images?
22. How can secret scanning help?
23. What is the difference between `GITHUB_TOKEN` and a personal access token?
24. When would you use a GitHub App instead of a PAT?
25. Why should secrets be scoped as narrowly as possible?

## Advanced / Production

26. Design a secure secrets architecture for DEV, QA, SIT, UAT, and PROD.
27. How would you replace long-lived AWS credentials with GitHub OIDC?
28. How would you restrict an AWS IAM role to a specific GitHub repository and production environment?
29. How would you securely integrate GitHub Actions with AWS Secrets Manager?
30. How would you protect secrets when using self-hosted runners?
31. How would you secure secrets on ARC-based ephemeral runners?
32. How would you prevent secrets from being exposed to fork-based pull requests?
33. Explain the security risks of `pull_request_target`.
34. How would you design a production JIRA change-request validation Action?
35. How would you protect JIRA API credentials?
36. How would you integrate GitHub Secrets with a DevSecOps pipeline containing SonarQube, Trivy, and Veracode?
37. How would you prevent secrets from entering Docker image layers?
38. How would you securely manage secrets in a GitOps workflow with ArgoCD and EKS?
39. How would you manage Kubernetes secrets without committing plaintext secrets to Git?
40. How would you secure Terraform state when it contains sensitive information?
41. How would you design secret rotation with zero downtime?
42. How would you respond if a GitHub secret were accidentally printed in workflow logs?
43. How would you respond if an organization-wide secret were compromised?
44. How would you audit which repositories have access to a shared secret?
45. How would you design a production secret inventory and ownership model?
46. How would you prevent unnecessary jobs from receiving production secrets?
47. How would you combine GitHub Environments, environment secrets, OIDC, and least-privilege permissions?
48. How would you design a secure production deployment workflow where JIRA approval, SHA validation, ECR, ArgoCD, and EKS are involved?
49. What is the difference between storing a secret in GitHub Secrets and storing it in an external secret manager?
50. Design an enterprise-grade GitHub Actions secret-management strategy covering storage, access, rotation, auditing, incident response, and retirement.