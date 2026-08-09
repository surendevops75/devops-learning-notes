# GitHub API Authentication

GitHub API authentication determines how an automation system proves its identity and what permissions it receives.

Authentication is required when an API operation needs access beyond what is available anonymously.

Common authentication approaches include:

```text
GITHUB_TOKEN
Personal Access Token (PAT)
GitHub App
GitHub App Installation Token
```

For DevOps automation, the authentication method should be selected based on:

```text
Security
Permissions
Repository Scope
Organization Scope
Automation Lifetime
Ownership
```

---

# Authentication Flow

Basic model:

```text
Client
  |
  ↓
Authentication Credential
  |
  ↓
GitHub REST API
  |
  ↓
Authorization Check
  |
  ↓
Response
```

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
What are you allowed to do?
```

---

# Authentication vs Authorization

These concepts are different.

### Authentication

Verifies identity.

```text
Token / App
   ↓
Who is making the request?
```

### Authorization

Determines permissions.

```text
Authenticated Client
       ↓
What can it access?
```

Example:

```text
Token
 ↓
Authenticated
 ↓
contents:read
 ↓
Can read repository
```

---

# Anonymous API Requests

Some public GitHub API information can be accessed without authentication.

Example:

```bash
curl -sS \
  "https://api.github.com/repos/daws-86s/catalogue"
```

However, anonymous requests have more limited access and lower rate limits than authenticated requests.

For CI/CD automation, authenticated requests are generally preferred when the workflow needs reliable access or private repository data.

---

# GITHUB_TOKEN

GitHub Actions automatically provides a repository-scoped token:

```text
GITHUB_TOKEN
```

It is designed for workflows running inside GitHub Actions.

Example:

```yaml
- name: GitHub API Request
  env:
    GH_TOKEN: ${{ github.token }}
  run: |
    curl --fail-with-body -sS \
      -H "Authorization: Bearer $GH_TOKEN" \
      -H "Accept: application/vnd.github+json" \
      "https://api.github.com/repos/${{ github.repository }}"
```

---

# Why GITHUB_TOKEN Is Useful

It avoids manually creating a long-lived token for many workflow operations.

Example:

```text
GitHub Actions
      |
      ↓
GITHUB_TOKEN
      |
      ↓
GitHub API
```

It is especially useful for operations involving the repository where the workflow is running, subject to the token's permissions.

---

# GITHUB_TOKEN Permissions

The token's capabilities are controlled through workflow permissions.

Example:

```yaml
permissions:
  contents: read
```

This follows the least-privilege principle.

If the workflow needs to write repository contents:

```yaml
permissions:
  contents: write
```

Only grant the permissions actually required.

---

# Default Permission Principle

Do not assume:

```text
GITHUB_TOKEN
=
Full Repository Access
```

Its available permissions depend on the repository/organization configuration and the workflow's permissions.

Explicitly define the permissions needed by the workflow.

---

# Least Privilege

Example:

```yaml
permissions:
  contents: read
```

is preferable to unnecessarily granting broad access.

Think:

```text
Required Permission
       ↓
Grant Permission
       ↓
Nothing More
```

---

# Example: Read Repository

```yaml
permissions:
  contents: read

jobs:

  check:

    runs-on: ubuntu-latest

    steps:

      - name: Get Repository
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          curl --fail-with-body -sS \
            -H "Authorization: Bearer $GH_TOKEN" \
            -H "Accept: application/vnd.github+json" \
            "https://api.github.com/repos/${{ github.repository }}"
```

---

# Example: Write Repository Contents

If a workflow needs to push changes:

```yaml
permissions:
  contents: write
```

Example architecture:

```text
GitHub Actions
      |
      ↓
GITHUB_TOKEN
      |
      ↓
contents: write
      |
      ↓
Repository
```

Only use write access when required.

---

# Example: Pull Request Permissions

If a workflow needs to interact with pull requests, configure the appropriate pull-request permission.

Conceptually:

```yaml
permissions:
  contents: read
  pull-requests: write
```

The exact permission required depends on the API operation.

---

# Example: Issues

For issue-related operations:

```yaml
permissions:
  contents: read
  issues: write
```

Again, grant only what the workflow actually needs.

---

# Example: OIDC

For cloud authentication such as AWS OIDC:

```yaml
permissions:
  contents: read
  id-token: write
```

This is different from GitHub API authentication.

```text
GITHUB_TOKEN
→ GitHub API

OIDC
→ Cloud identity federation
```

---

# GITHUB_TOKEN vs OIDC

### GITHUB_TOKEN

Used primarily for:

```text
GitHub API
Repository Operations
Actions Operations
GitHub Resources
```

### OIDC

Used for:

```text
AWS
Azure
GCP
Other Trusted Identity Providers
```

Example:

```text
GitHub Actions
   |
   ├── GITHUB_TOKEN → GitHub
   |
   └── OIDC → AWS
```

---

# Token in Environment Variable

Recommended pattern:

```yaml
env:
  GH_TOKEN: ${{ github.token }}
```

Then:

```bash
curl \
  -H "Authorization: Bearer $GH_TOKEN" \
  ...
```

Avoid putting tokens directly into command strings when unnecessary.

---

# Never Print Tokens

Never do:

```bash
echo "$GH_TOKEN"
```

Avoid:

```bash
set -x
```

in scripts that could expose credentials.

Secrets and tokens should never appear in logs.

---

# Personal Access Token

A Personal Access Token (PAT) can authenticate API requests.

Conceptually:

```text
User
 ↓
PAT
 ↓
GitHub API
```

PATs can be useful for certain automation scenarios, but they create a credential tied to a user identity.

---

# PAT Security Risk

If:

```text
User leaves organization
```

or:

```text
PAT expires
```

automation depending on that token can stop working.

Therefore, PATs should not automatically be the first choice for long-lived organization automation.

---

# Fine-Grained PAT

GitHub provides fine-grained personal access tokens that can be restricted by:

```text
Repository
Permissions
Expiration
```

This is generally preferable to broad, long-lived access when a PAT is actually required.

---

# PAT Storage

Never put a PAT directly into:

```text
Workflow YAML
Shell Script
Git Repository
Dockerfile
README
Terraform Code
```

Store it in an approved secret-management mechanism.

For GitHub Actions:

```text
Repository Secret
Environment Secret
Organization Secret
```

according to the required scope.

---

# PAT Example

Suppose a secret is:

```text
GH_PAT
```

Workflow:

```yaml
- name: API Request
  env:
    GH_TOKEN: ${{ secrets.GH_PAT }}
  run: |
    curl --fail-with-body -sS \
      -H "Authorization: Bearer $GH_TOKEN" \
      -H "Accept: application/vnd.github+json" \
      "https://api.github.com/repos/OWNER/REPO"
```

---

# PAT vs GITHUB_TOKEN

### GITHUB_TOKEN

```text
Automatically provided
Workflow-scoped
Repository-focused
Short-lived
Good for GitHub Actions
```

### PAT

```text
Created by user
Can be used outside Actions
User-associated
Requires lifecycle management
```

Prefer `GITHUB_TOKEN` when it provides the required permissions.

---

# GitHub App

GitHub Apps provide another authentication model.

Conceptually:

```text
GitHub App
     |
     ↓
Installation
     |
     ↓
Installation Token
     |
     ↓
GitHub API
```

GitHub Apps are useful for organization-level integrations and automation that should not depend on an individual user's credentials.

---

# Why GitHub Apps?

GitHub Apps can provide:

```text
Fine-Grained Permissions
Repository Selection
Independent Identity
Installation-Based Access
Centralized Ownership
```

This can be more appropriate for enterprise integrations.

---

# GitHub App Architecture

```text
Organization
     |
     ↓
GitHub App
     |
     ↓
Selected Repositories
     |
     ↓
Installation
     |
     ↓
Installation Token
     |
     ↓
GitHub API
```

---

# GitHub App vs PAT

### PAT

```text
User
 ↓
Token
 ↓
API
```

### GitHub App

```text
Application
 ↓
Installation
 ↓
Token
 ↓
API
```

A GitHub App is generally better suited to an integration that needs its own identity and controlled repository access.

---

# GitHub App Installation

An App can be installed on:

```text
One Repository
Multiple Repositories
Organization
```

depending on the installation configuration and permissions.

---

# GitHub App Permissions

An App can request specific permissions such as:

```text
Repository Contents
Pull Requests
Issues
Actions
Metadata
```

Only request permissions needed by the integration.

---

# GitHub App Authentication Flow

High-level:

```text
GitHub App
    |
    ↓
App Credentials
    |
    ↓
Generate App JWT
    |
    ↓
GitHub
    |
    ↓
Installation ID
    |
    ↓
Installation Access Token
    |
    ↓
REST API
```

The JWT and installation token have different purposes.

---

# App JWT

A GitHub App uses a signed JSON Web Token (JWT) to authenticate as the App when requesting an installation access token.

Conceptually:

```text
App Private Key
      |
      ↓
Generate JWT
      |
      ↓
GitHub
      |
      ↓
Installation Token
```

The private key must be protected securely.

---

# Installation Token

The installation access token is used to perform API operations as the installed GitHub App.

Flow:

```text
App
 ↓
JWT
 ↓
Installation
 ↓
Installation Token
 ↓
REST API
```

---

# Do Not Store App Private Keys in Repository

Never commit:

```text
GitHub App Private Key
```

to source control.

Use:

```text
Secret Manager
GitHub Secret
Vault
Cloud Secret Manager
```

according to your security architecture.

---

# Authentication Decision

Use:

```text
GITHUB_TOKEN
```

when:

```text
Workflow needs GitHub access
```

Use:

```text
PAT
```

when:

```text
A user-associated token is genuinely required
```

Use:

```text
GitHub App
```

when:

```text
Long-lived organization integration
Independent application identity
Fine-grained repository access
```

---

# Authentication Decision Tree

```text
Need GitHub API Access?
        |
        ↓
Running inside GitHub Actions?
        |
       YES
        |
        ↓
Can GITHUB_TOKEN provide required permissions?
        |
    ┌───┴───┐
    ↓       ↓
   YES      NO
    |        |
    ↓        ↓
GITHUB_TOKEN  Consider
             GitHub App / PAT
```

For organization-wide automation, evaluate GitHub Apps before using a user-owned PAT.

---

# GitHub API Authentication Headers

Authenticated request:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}"
```

---

# API Version Header

GitHub may document an API version header for a specific API version.

Example:

```bash
-H "X-GitHub-Api-Version: YYYY-MM-DD"
```

Use the currently supported version specified by your organization's GitHub API integration documentation.

---

# Production cURL Pattern

```bash
set -euo pipefail

response="$(
  curl --fail-with-body -sS \
    -H "Authorization: Bearer ${GH_TOKEN}" \
    -H "Accept: application/vnd.github+json" \
    "https://api.github.com/repos/${OWNER}/${REPO}"
)"

echo "$response" | jq '.name'
```

---

# Authentication in GitHub Actions

Example:

```yaml
name: GitHub API Check

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:

  check:

    runs-on: ubuntu-latest

    steps:

      - name: Query GitHub API
        env:
          GH_TOKEN: ${{ github.token }}
        run: |
          curl --fail-with-body -sS \
            -H "Authorization: Bearer $GH_TOKEN" \
            -H "Accept: application/vnd.github+json" \
            "https://api.github.com/repos/${{ github.repository }}"
```

---

# Authentication in Jenkins

Jenkins should store credentials in its credential management system.

Conceptually:

```text
Jenkins Credentials
        |
        ↓
Pipeline
        |
        ↓
GH_TOKEN
        |
        ↓
GitHub API
```

Do not store:

```text
PAT
App Private Key
```

inside the Jenkinsfile.

---

# Jenkins Example

Conceptual:

```groovy
withCredentials([
  string(
    credentialsId: 'github-token',
    variable: 'GH_TOKEN'
  )
]) {
    sh '''
      curl --fail-with-body -sS \
        -H "Authorization: Bearer ${GH_TOKEN}" \
        -H "Accept: application/vnd.github+json" \
        "https://api.github.com/repos/OWNER/REPO"
    '''
}
```

The exact Jenkins credential type should match the credential being stored.

---

# Authentication and Dependabot

Your Dependabot gate can use:

```text
GITHUB_TOKEN
```

when the workflow has the required access to the Dependabot endpoint.

Example:

```yaml
permissions:
  contents: read
  security-events: read
```

Then:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${{ github.repository }}/dependabot/alerts?severity=critical&state=open"
```

The exact permissions required should be verified against the endpoint and repository configuration.

---

# Dependabot Security Gate

```text
GitHub Actions
      |
      ↓
GITHUB_TOKEN
      |
      ↓
Dependabot API
      |
      ↓
Critical Alerts
      |
 ┌────┴────┐
 ↓         ↓
YES        NO
 ↓         ↓
FAIL      Continue
```

---

# Authentication and Workflow API

For workflow-related API operations:

```text
GitHub Actions
      |
      ↓
Token
      |
      ↓
Actions API
```

The token must have the required Actions/workflow permissions for the specific operation.

---

# Authentication and Workflow Dispatch

Conceptually:

```text
System A
   |
   ↓
GitHub API
   |
   ↓
Workflow Dispatch
   |
   ↓
System B Workflow
```

The authentication credential must have permission to trigger the target workflow.

---

# Cross-Repository Authentication

Example:

```text
Repository A
     |
     ↓
Authentication
     |
     ↓
GitHub API
     |
     ↓
Repository B
```

A repository's `GITHUB_TOKEN` does not automatically mean unrestricted access to every repository in an organization.

For cross-repository automation, carefully design:

```text
Repository Access
Permissions
Token Type
Trust Relationship
```

---

# GitHub App for Cross-Repository Automation

A GitHub App can be installed only on the repositories it needs.

Example:

```text
GitHub App
   |
   ├── Repository A
   ├── Repository B
   └── Repository C
```

Then:

```text
Installation Token
       ↓
GitHub API
```

This can provide a clean organization-level integration model.

---

# Security Principle: Avoid Broad PATs

Avoid a design like:

```text
Developer PAT
     |
     ↓
All Repositories
     |
     ↓
Production Automation
```

A better design may be:

```text
GitHub App
     |
     ↓
Required Repositories
     |
     ↓
Required Permissions
```

---

# Token Rotation

Credentials have lifecycles.

For PATs:

```text
Create
 ↓
Use
 ↓
Rotate
 ↓
Expire / Revoke
```

For GitHub Apps:

```text
Private Key
 ↓
Rotate
 ↓
Update Secret
```

For short-lived installation tokens:

```text
Generate
 ↓
Use
 ↓
Expire
```

Short-lived credentials reduce long-term exposure.

---

# Credential Rotation Strategy

Production automation should define:

```text
Who owns the credential?
When does it expire?
How is it rotated?
How is failure detected?
How is the old credential revoked?
```

Do not wait until production deployment fails because a credential silently expired.

---

# Secret Exposure

Potential exposure points:

```text
Workflow Logs
Shell Debugging
Error Messages
Artifacts
Environment Dumps
Pull Requests
Source Code
```

Protect all of them.

---

# Avoid Environment Dumps

Avoid commands such as:

```bash
env
```

when sensitive credentials are present.

Similarly:

```bash
set -x
```

can expose command arguments and should be avoided around secrets.

---

# Masking

GitHub Actions can mask registered secrets in logs, but masking should not be treated as permission to print secrets.

Correct approach:

```text
Do not print secrets
```

rather than:

```text
Print and depend on masking
```

---

# Token in URL

Avoid putting credentials in URLs.

Bad:

```text
https://TOKEN@api.github.com/...
```

Use headers:

```bash
-H "Authorization: Bearer ${GH_TOKEN}"
```

---

# Token in Command Line

Passing a token through an environment variable is generally preferable to hardcoding it into scripts.

Example:

```yaml
env:
  GH_TOKEN: ${{ github.token }}
```

Then:

```bash
curl -H "Authorization: Bearer ${GH_TOKEN}" ...
```

---

# Authentication and Pull Requests

Be careful with secrets when workflows run from pull requests.

Especially:

```text
Untrusted Fork PR
      |
      ↓
Workflow
      |
      ↓
Secrets
```

Do not expose privileged secrets to untrusted code.

---

# Fork PR Security

Potentially dangerous:

```text
External Contributor
       |
       ↓
Pull Request
       |
       ↓
Privileged Workflow
       |
       ↓
Production Secret
```

A secure workflow must separate:

```text
Untrusted CI
```

from:

```text
Privileged Deployment
```

---

# Production Authentication Boundary

Recommended conceptual architecture:

```text
Pull Request
    |
    ↓
Unprivileged CI
    |
    ├── Tests
    ├── SAST
    └── Container Scan
```

Then:

```text
Trusted Branch
    |
    ↓
Protected Environment
    |
    ↓
OIDC / Production Credentials
    |
    ↓
Production
```

---

# Authentication + GitHub Environment

For production:

```text
GitHub Environment
       |
       ↓
Required Approval
       |
       ↓
Production Workflow
       |
       ↓
Authentication
       |
       ↓
Production API
```

This provides an authorization boundary before privileged operations.

---

# Authentication + AWS OIDC

Important distinction:

```text
GitHub API Authentication
       ↓
GITHUB_TOKEN / PAT / GitHub App

AWS Authentication
       ↓
GitHub OIDC
```

Do not use a GitHub PAT as a replacement for AWS identity federation.

---

# Authentication + ECR/EKS

Production flow:

```text
GitHub Actions
       |
       ↓
GitHub OIDC
       |
       ↓
AWS IAM Role
       |
       ├── ECR
       └── EKS
```

GitHub REST API operations remain separately authenticated through GitHub's authentication mechanisms.

---

# Authentication + ArgoCD

If GitHub Actions updates a GitOps repository:

```text
GitHub Actions
      |
      ↓
GitHub Authentication
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

If ArgoCD itself accesses GitHub, ArgoCD has its own Git repository authentication configuration.

Do not confuse these identities.

---

# Multiple Identities

A production pipeline may involve several identities:

```text
GitHub Actions
    |
    ├── GITHUB_TOKEN
    |       ↓
    |    GitHub API
    |
    └── OIDC
            ↓
         AWS IAM
            ↓
        ECR / EKS
```

ArgoCD:

```text
ArgoCD Identity
      ↓
GitOps Repository
      ↓
EKS
```

Each identity should have only the permissions required for its role.

---

# Authentication Audit

Periodically review:

```text
PATs
GitHub Apps
Repository Secrets
Environment Secrets
Organization Secrets
GITHUB_TOKEN Permissions
OIDC Trust Policies
AWS IAM Roles
```

Look for:

```text
Unused Credentials
Excessive Permissions
Expired Tokens
Unexpected Repository Access
Old Service Accounts
```

---

# Authentication Governance

Enterprise governance should define:

```text
Approved Authentication Methods
Token Lifetime
Secret Storage
Rotation
Permissions
Repository Scope
Production Access
Audit Requirements
```

---

# Recommended Authentication Hierarchy

For GitHub Actions:

```text
1. GITHUB_TOKEN
       ↓
   If sufficient

2. GitHub App
       ↓
   For broader/centralized integration

3. PAT
       ↓
   Only when justified
```

For cloud:

```text
OIDC
   ↓
Short-Lived Cloud Credentials
```

Avoid long-lived static credentials whenever possible.

---

# Authentication Checklist

```text
☐ Identify required API operation
☐ Determine required permission
☐ Prefer GITHUB_TOKEN in GitHub Actions
☐ Use least privilege
☐ Consider GitHub App for centralized integration
☐ Use PAT only when justified
☐ Store credentials securely
☐ Never hardcode credentials
☐ Never print credentials
☐ Avoid credentials in URLs
☐ Protect fork PR workflows
☐ Use environment protection for production
☐ Use OIDC for cloud authentication
☐ Rotate long-lived credentials
☐ Review token scope
☐ Monitor authentication failures
```

---

# Production Authentication Architecture

```text
                    GitHub
                      |
          ┌───────────┼────────────┐
          ↓           ↓            ↓
     GITHUB_TOKEN    App          PAT
          |           |            |
          └───────────┼────────────┘
                      ↓
                 GitHub API
                      |
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
    Repository     Actions      Security
        |
        ↓
 GitHub Actions
        |
        ↓
   GitHub Environment
        |
        ↓
     Approval
        |
        ↓
      OIDC
        |
        ↓
    AWS IAM Role
        |
     ┌──┴──┐
     ↓     ↓
    ECR    EKS
```

---

# Key Takeaways

Authentication is about securely establishing identity for API operations.

Main mechanisms:

```text
GITHUB_TOKEN
Personal Access Token
GitHub App
```

For GitHub Actions:

```text
GITHUB_TOKEN
```

should generally be the first option when it provides the required access.

For centralized organization integrations:

```text
GitHub App
```

can provide an independent identity and controlled repository access.

For cloud authentication:

```text
OIDC
```

should generally be preferred over long-lived cloud credentials.

The most important security principles are:

```text
Least Privilege
Short-Lived Credentials
Secure Storage
No Credential Logging
Repository/Environment Isolation
Controlled Production Access
Regular Rotation
```

For your DevSecOps architecture:

```text
GitHub Actions
     |
     ├── GITHUB_TOKEN
     │      ↓
     │   GitHub API
     │
     └── OIDC
            ↓
         AWS IAM
            ↓
       ECR / EKS
            |
            ↓
          ArgoCD
            |
            ↓
           EKS
```

Keep GitHub authentication and cloud authentication as separate security boundaries.

---

# Interview Questions

## Basic

1. What is GitHub API authentication?
2. What is the difference between authentication and authorization?
3. What is `GITHUB_TOKEN`?
4. How do you use `GITHUB_TOKEN` with cURL?
5. What is a Personal Access Token?
6. What is a fine-grained PAT?
7. What is a GitHub App?
8. What is an installation token?
9. What is least privilege?
10. Why should tokens never be hardcoded?

## Intermediate

11. What is the difference between GITHUB_TOKEN and PAT?
12. When would you use a GitHub App instead of a PAT?
13. How do you configure GITHUB_TOKEN permissions?
14. What is `permissions: contents: read`?
15. When would you use `contents: write`?
16. How do you authenticate to GitHub API from Jenkins?
17. How do you securely store a PAT in GitHub Actions?
18. How do you securely store a GitHub App private key?
19. What is the difference between GITHUB_TOKEN and OIDC?
20. Why is OIDC preferred for AWS authentication?
21. How do you authenticate a cross-repository GitHub API operation?
22. How would you secure Dependabot API access?
23. How do you prevent tokens from appearing in logs?
24. Why should credentials not be placed in URLs?
25. How would you rotate long-lived credentials?

## Advanced / Production

26. Design a secure authentication architecture for GitHub Actions and AWS.
27. How would you design GitHub App authentication for 100+ repositories?
28. How would you determine whether GITHUB_TOKEN has enough permissions for an API operation?
29. How would you securely trigger a workflow in another repository?
30. How would you secure cross-repository automation?
31. How would you protect GitHub API credentials from fork-based pull requests?
32. How would you separate untrusted CI from privileged production deployment?
33. How would you combine GitHub Environments with API authentication?
34. How would you combine GITHUB_TOKEN, OIDC, AWS IAM, ECR, EKS, and ArgoCD?
35. How would you design separate authentication boundaries for GitHub API, AWS, and Kubernetes?
36. How would you audit GitHub PATs and GitHub Apps across an organization?
37. How would you handle an expired PAT during a production deployment?
38. How would you troubleshoot a GitHub API 401 or 403 response?
39. How would you design least-privilege permissions for Dependabot and Actions APIs?
40. How would you prevent a compromised GitHub workflow from abusing a production GitHub App?
41. How would you design GitHub App permissions for repository contents, pull requests, and Actions?
42. How would you implement credential rotation without causing deployment downtime?
43. How would you protect GitHub App private keys in an enterprise environment?
44. How would you design authentication for Jenkins → GitHub API → GitHub Actions?
45. Design an enterprise-grade GitHub authentication architecture covering GITHUB_TOKEN, GitHub Apps, fine-grained PATs, GitHub Environments, fork PR security, least privilege, token rotation, Dependabot, workflow APIs, AWS OIDC, IAM, ECR, EKS, GitOps, ArgoCD, and production deployment security.