# GitHub REST API

The GitHub REST API allows us to interact with GitHub programmatically.

Instead of manually using the GitHub UI, we can use API requests to:

```text
Read Repository Information
Read Commits
Read Pull Requests
Read Issues
Read Dependabot Alerts
Trigger Workflows
Check Workflow Runs
Manage Releases
Manage Branches
Manage Repository Data
```

For DevOps and CI/CD, the GitHub API is useful when we need to automate GitHub operations from:

```text
GitHub Actions
Jenkins
Shell Scripts
Python Scripts
Terraform
Deployment Automation
Monitoring
Security Automation
```

---

# REST API Concept

REST API follows the basic model:

```text
Client
  |
  | HTTP Request
  ↓
GitHub API
  |
  | HTTP Response
  ↓
Client
```

Example:

```text
curl
  |
  ↓
api.github.com
  |
  ↓
JSON Response
```

---

# GitHub API Base URL

The GitHub REST API uses:

```text
https://api.github.com
```

Example:

```bash
curl -s \
  https://api.github.com/repos/OWNER/REPOSITORY
```

---

# Repository API

Example:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue"
```

This returns repository information in JSON format.

Typical information includes:

```text
Repository Name
Description
Owner
Default Branch
Visibility
Clone URLs
Created Date
Updated Date
```

---

# JSON Response

GitHub API responses are generally JSON.

Example:

```json
{
  "name": "catalogue",
  "full_name": "daws-86s/catalogue",
  "private": false,
  "default_branch": "main"
}
```

We can process JSON using:

```bash
jq
```

---

# Using jq

Example:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue" \
  | jq '.name'
```

Output:

```text
"catalogue"
```

---

# Extract Multiple Fields

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue" \
  | jq '{
      name: .name,
      branch: .default_branch,
      private: .private
    }'
```

---

# HTTP Methods

REST APIs commonly use:

```text
GET
POST
PUT
PATCH
DELETE
```

### GET

Read information.

```text
GET /repos/OWNER/REPO
```

### POST

Create something or trigger an operation.

```text
POST /repos/OWNER/REPO/dispatches
```

### PUT

Replace or create a resource depending on the endpoint.

### PATCH

Partially update a resource.

### DELETE

Delete a resource.

---

# GET Example

Get repository information:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue"
```

---

# GET Commits

Example:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/commits"
```

Extract commit SHAs:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/commits" \
  | jq '.[].sha'
```

---

# Get Latest Commit

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/commits/main" \
  | jq '.sha'
```

Output:

```text
"8a92f31..."
```

The actual SHA will depend on the repository state.

---

# Why Commit SHA Matters

A commit SHA provides an immutable identifier for a specific Git commit.

Example:

```text
main
 ↓
8a92f31
```

For production deployment:

```text
Git SHA
 ↓
Build
 ↓
Image
 ↓
Deployment
```

This provides traceability.

---

# Pull Requests

Get pull requests:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/pulls"
```

Only open PRs:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/pulls?state=open"
```

---

# Pull Request States

Common states:

```text
open
closed
```

Example:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/pulls?state=closed"
```

---

# Issues

Get repository issues:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/issues"
```

Open issues:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/issues?state=open"
```

---

# Branches

Get branches:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/branches"
```

Extract branch names:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/branches" \
  | jq '.[].name'
```

---

# Tags

Get repository tags:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/tags"
```

Extract tag names:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/tags" \
  | jq '.[].name'
```

---

# Releases

Get releases:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/releases"
```

Get latest release:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/releases/latest"
```

Extract version:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/releases/latest" \
  | jq '.tag_name'
```

---

# API URL Using Variables

Instead of hardcoding repository information:

```bash
OWNER="daws-86s"
REPO="catalogue"

curl -s \
  "https://api.github.com/repos/${OWNER}/${REPO}"
```

This makes scripts reusable.

---

# GitHub Actions Repository Context

Inside GitHub Actions:

```text
github.repository
```

returns:

```text
OWNER/REPOSITORY
```

Example:

```text
daws-86s/catalogue
```

Therefore:

```bash
curl -s \
  "https://api.github.com/repos/${{ github.repository }}"
```

can dynamically target the current repository.

---

# GitHub API in GitHub Actions

Example:

```yaml
- name: Get Repository Information
  run: |
    curl -s \
      -H "Accept: application/vnd.github+json" \
      "https://api.github.com/repos/${{ github.repository }}"
```

---

# Accept Header

GitHub examples commonly use:

```bash
-H "Accept: application/vnd.github+json"
```

Example:

```bash
curl -s \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${{ github.repository }}"
```

This explicitly requests GitHub's JSON representation.

---

# Authorization Header

Authenticated API requests commonly use:

```bash
-H "Authorization: Bearer TOKEN"
```

Example:

```bash
curl -s \
  -H "Authorization: Bearer ${GITHUB_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${{ github.repository }}"
```

Never print the token.

---

# GitHub Actions Token

GitHub Actions provides a token through:

```text
GITHUB_TOKEN
```

A workflow can use:

```yaml
${{ secrets.GITHUB_TOKEN }}
```

or the recommended workflow token context:

```yaml
${{ github.token }}
```

Example:

```yaml
- name: GitHub API Request
  env:
    GH_TOKEN: ${{ github.token }}
  run: |
    curl -s \
      -H "Authorization: Bearer $GH_TOKEN" \
      -H "Accept: application/vnd.github+json" \
      "https://api.github.com/repos/${{ github.repository }}"
```

---

# Permissions

GitHub Actions permissions should follow least privilege.

Example:

```yaml
permissions:
  contents: read
```

If the workflow needs to perform an operation requiring more access, grant only the required permission.

Example:

```yaml
permissions:
  contents: write
```

Do not automatically use broad permissions.

---

# API Request Structure

A typical request:

```bash
curl \
  -X GET \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}"
```

Structure:

```text
curl
 |
 ├── HTTP Method
 ├── Headers
 └── URL
```

---

# Query Parameters

Query parameters are appended using:

```text
?
```

Multiple parameters use:

```text
&
```

Example:

```text
?state=open&per_page=100
```

---

# Pagination

Many GitHub API endpoints return results in pages.

Example:

```text
?page=1
?page=2
?page=3
```

You can also control page size:

```text
?per_page=100
```

Example:

```bash
curl -s \
  "https://api.github.com/repos/daws-86s/catalogue/commits?per_page=100"
```

---

# Why Pagination Matters

Suppose a repository has:

```text
500 commits
```

An API request may not return all 500 in one response.

Your automation must account for pagination when processing large result sets.

---

# API Rate Limits

GitHub APIs have rate limits.

Repeated requests:

```text
Request
Request
Request
...
```

can eventually reach the allowed limit.

Production scripts should:

```text
Avoid unnecessary calls
Handle errors
Respect rate limits
Use pagination carefully
Cache data where appropriate
```

---

# HTTP Status Codes

Common API responses include:

```text
200 → Success
201 → Created
204 → Success with no response body
400 → Bad Request
401 → Unauthorized
403 → Forbidden / rate limit or permission issue
404 → Not Found
409 → Conflict
422 → Validation Error
```

Always check the response status in production automation.

---

# Checking HTTP Status

Example:

```bash
curl -sS \
  -o response.json \
  -w "%{http_code}" \
  "https://api.github.com/repos/${OWNER}/${REPO}"
```

This returns the HTTP status code.

---

# Fail on HTTP Errors

Use:

```bash
curl --fail-with-body
```

Example:

```bash
curl --fail-with-body -sS \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}"
```

This is useful in CI/CD because the command can fail when the HTTP request returns an error response.

---

# Production API Pattern

Recommended structure:

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

Avoid exposing secrets in logs.

---

# API in CI/CD

Example:

```text
GitHub Actions
      |
      ↓
GitHub REST API
      |
 ┌────┼───────────────┐
 ↓    ↓               ↓
PRs  Dependabot    Workflows
```

This allows workflow automation to make decisions based on GitHub data.

---

# Dependabot Use Case

A production pipeline may check:

```text
Are there critical open Dependabot alerts?
```

If yes:

```text
Stop deployment
```

If no:

```text
Continue
```

Example:

```bash
curl -s \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${{ github.repository }}/dependabot/alerts?severity=critical&state=open"
```

Then:

```bash
jq '. | length'
```

---

# Count Critical Dependabot Alerts

Example:

```bash
critical_alerts="$(
  curl -s \
    -H "Authorization: Bearer ${GH_TOKEN}" \
    -H "Accept: application/vnd.github+json" \
    "https://api.github.com/repos/${{ github.repository }}/dependabot/alerts?severity=critical&state=open" \
    | jq '. | length'
)"

echo "Critical alerts: $critical_alerts"
```

Then:

```bash
if [ "$critical_alerts" -gt 0 ]; then
  echo "Critical Dependabot alerts found"
  exit 1
fi
```

---

# Security Gate

Flow:

```text
Build
  ↓
Dependabot API
  ↓
Critical Alerts?
  |
 ┌┴──────────┐
 ↓           ↓
YES          NO
 ↓           ↓
FAIL       Continue
```

This can become part of a DevSecOps gate.

---

# Hardcoded Repository

Example:

```bash
curl -s \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/daws-86s/catalogue/dependabot/alerts?severity=critical&state=open"
```

This works for that specific repository.

---

# Dynamic Repository

Better for reusable workflows:

```bash
curl -s \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${{ github.repository }}/dependabot/alerts?severity=critical&state=open"
```

Now the workflow can operate against the repository where it runs, subject to token permissions.

---

# API and Workflow Automation

The REST API can be used to:

```text
Inspect Workflow Runs
Trigger Workflows
Check Workflow Status
Read Commit Information
Read Pull Requests
Read Issues
Read Security Alerts
```

---

# Workflow Runs

A workflow run endpoint can be used to inspect workflow execution.

Conceptually:

```text
Repository
   |
   ↓
Actions
   |
   ↓
Workflow Runs
```

Example endpoint pattern:

```text
/repos/OWNER/REPO/actions/runs
```

---

# Get Workflow Runs

Example:

```bash
curl -s \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs"
```

Extract run IDs:

```bash
... | jq '.workflow_runs[].id'
```

---

# Workflow Status

A workflow run contains information such as:

```text
Status
Conclusion
Branch
Commit SHA
Workflow Name
Run Number
Created Time
Updated Time
```

Useful for external automation.

---

# Workflow Status Example

You can inspect:

```bash
curl -s \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs" \
  | jq '.workflow_runs[] | {
      id,
      name,
      status,
      conclusion,
      head_sha
    }'
```

---

# Workflow Triggering

GitHub REST API can be used to trigger workflows through supported workflow dispatch mechanisms.

Conceptually:

```text
External System
      |
      ↓
GitHub REST API
      |
      ↓
workflow_dispatch
      |
      ↓
GitHub Actions
```

This is useful for:

```text
Deployment Automation
Release Automation
Cross-Repository Pipelines
Scheduled External Systems
```

---

# Production Example

```text
Jenkins
  |
  ↓
GitHub REST API
  |
  ↓
Trigger GitHub Actions
  |
  ↓
Production Workflow
```

Or:

```text
GitHub Actions
  |
  ↓
REST API
  |
  ↓
Another Repository Workflow
```

---

# Cross-Repository Automation

Example:

```text
Application Repository
       |
       ↓
Build Image
       |
       ↓
ECR
       |
       ↓
GitHub API
       |
       ↓
GitOps Repository
       |
       ↓
Workflow
       |
       ↓
ArgoCD
```

The API can act as the integration layer between GitHub repositories.

---

# API and GitOps

A workflow may:

```text
Build image
      ↓
Get image digest
      ↓
Update GitOps repository
      ↓
Commit changes
      ↓
Push
      ↓
ArgoCD detects change
      ↓
EKS
```

The exact API operations depend on the GitOps implementation.

---

# API and Pull Request Automation

Possible automation:

```text
Build
 ↓
Test
 ↓
Security
 ↓
GitHub API
 ↓
Check PR
 ↓
Update / Validate
```

For example, automation can inspect:

```text
PR State
PR Labels
Review Status
Commit SHA
Checks
```

---

# API and Release Automation

Example:

```text
Build
 ↓
Security
 ↓
Tests
 ↓
GitHub API
 ↓
Create / Inspect Release
 ↓
Deploy
```

---

# API and Repository Metadata

Automation can retrieve:

```text
Repository Name
Default Branch
Visibility
Topics
Branch Information
Tags
Releases
```

This can help scripts dynamically discover repository configuration.

---

# API and Branch Protection

The API can be used to inspect or manage supported repository branch rules/settings, subject to permissions.

This can support governance such as:

```text
Required Reviews
Status Checks
Protected Branches
```

Use API automation carefully because changes can affect developer workflows.

---

# API and Governance

A platform team could periodically inspect repositories:

```text
Repository A
Repository B
Repository C
Repository D
```

and verify:

```text
Required workflows
Security controls
Branch configuration
Actions usage
```

This can become a governance automation system.

---

# API and DevSecOps Governance

Example:

```text
Repository Inventory
       |
       ↓
GitHub API
       |
 ┌─────┼─────────────┐
 ↓     ↓             ↓
Actions Security   Branches
 ↓       ↓            ↓
Workflow Dependabot  Rules
```

Then generate:

```text
Compliance Report
```

---

# API and Monitoring

External monitoring can periodically query workflow runs:

```text
GitHub API
   |
   ↓
Workflow Runs
   |
   ↓
Status
   |
   ↓
Monitoring System
```

Possible checks:

```text
Failed deployments
Failed builds
Long-running workflows
Repeated failures
```

---

# API and Incident Response

During an incident, automation can retrieve:

```text
Recent commits
Recent deployments
Workflow runs
Pull requests
Release information
```

This can help answer:

```text
What changed?
When did it change?
Which workflow deployed it?
Which commit was deployed?
```

---

# API and Commit Traceability

Production:

```text
Commit SHA
   ↓
Workflow Run
   ↓
Image
   ↓
Deployment
```

GitHub API can help retrieve the GitHub-side information needed to establish this chain.

---

# API and Production Validation

Before deployment:

```text
GitHub API
   |
   ├── Commit
   ├── PR
   ├── Checks
   ├── Workflow
   └── Release
          |
          ↓
       Validation
          |
          ↓
       Production
```

---

# API Authentication Overview

Common authentication mechanisms include:

```text
GitHub Actions GITHUB_TOKEN
Personal Access Token
GitHub App Authentication
```

For automation, choose the mechanism based on:

```text
Scope
Security
Ownership
Repository Access
Organization Requirements
```

Authentication will be covered in detail in:

```text
02-Authentication.md
```

---

# API Security Principles

Never:

```text
Hardcode tokens
Print tokens
Commit tokens
Store tokens in source code
Use excessive permissions
```

Prefer:

```text
Secrets
OIDC where applicable
GitHub Apps
Fine-grained access
Least privilege
```

---

# API and Environment Secrets

Example:

```yaml
jobs:

  check:

    environment:
      name: production

    steps:

      - name: API Request
        env:
          GH_TOKEN: ${{ secrets.GH_TOKEN }}
        run: |
          curl \
            -H "Authorization: Bearer $GH_TOKEN" \
            ...
```

Use the narrowest scope needed.

---

# API Headers

Common headers:

```bash
-H "Accept: application/vnd.github+json"
```

Authentication:

```bash
-H "Authorization: Bearer ${GH_TOKEN}"
```

API version headers may also be used according to GitHub's current API documentation.

---

# API URL Construction

Avoid unsafe string construction.

Good:

```bash
OWNER="daws-86s"
REPO="catalogue"

API_URL="https://api.github.com/repos/${OWNER}/${REPO}"
```

Then:

```bash
curl -sS "$API_URL"
```

Validate inputs when they originate outside trusted configuration.

---

# cURL Options

Common options:

```text
-s
-S
-sS
-H
-X
-d
-o
-w
```

### `-s`

Silent mode.

### `-S`

Show errors even with silent mode.

### `-sS`

Silent except errors.

### `-H`

Add HTTP header.

### `-X`

Specify HTTP method.

### `-d`

Send request data.

### `-o`

Write response to a file.

### `-w`

Write response metadata.

---

# Recommended cURL Pattern

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "$API_URL"
```

This is a good foundation for CI/CD scripts.

---

# Store Response

Example:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "$API_URL" \
  -o response.json
```

Then:

```bash
jq '.' response.json
```

---

# Parse Response Safely

Example:

```bash
repo_name="$(jq -r '.name' response.json)"
```

Then:

```bash
echo "Repository: $repo_name"
```

Use `-r` when you want raw string output instead of JSON-quoted strings.

---

# API Error Handling

Bad:

```bash
curl -s "$API_URL"
```

and assume success.

Better:

```bash
curl --fail-with-body -sS "$API_URL"
```

Then handle failure explicitly.

---

# Production Error Flow

```text
API Request
    |
    ↓
HTTP Response
    |
 ┌──┴────────────┐
 ↓               ↓
2xx             Error
 ↓               ↓
Continue      Stop / Retry
```

Some errors may be transient and suitable for retry; others, such as authentication or validation errors, should not simply be retried.

---

# Retry Strategy

For transient failures:

```text
Request
 ↓
Failure
 ↓
Wait
 ↓
Retry
 ↓
Success / Final Failure
```

Use bounded retries and backoff.

Do not create infinite API retry loops.

---

# API and Rate Limit Handling

A production script should consider:

```text
Rate Limits
429 Responses
403 Responses
Retry-After
Request Volume
Pagination
```

Do not continuously hammer the API.

---

# API and Pagination Strategy

Example:

```text
Page 1
 ↓
Page 2
 ↓
Page 3
 ↓
No More Pages
```

Your script should stop when all results are processed.

---

# API and jq

`jq` is especially useful in DevOps automation.

Example:

```bash
curl -sS "$API_URL" | jq '.workflow_runs[]'
```

Filter:

```bash
curl -sS "$API_URL" \
  | jq '.workflow_runs[] | select(.conclusion == "failure")'
```

---

# Count Results

Example:

```bash
curl -sS "$API_URL" \
  | jq '.workflow_runs | length'
```

Same pattern as:

```bash
jq '. | length'
```

for an array response.

---

# Dependabot Example from Your Notes

```bash
curl -s \
  -H "Authorization: Bearer " \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${{ github.repository }}/dependabot/alerts?severity=critical&state=open"
```

Then:

```bash
jq '. | length'
```

Production version:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${{ github.repository }}/dependabot/alerts?severity=critical&state=open" \
  | jq '. | length'
```

---

# Production DevSecOps Gate

```bash
critical_alerts="$(
  curl --fail-with-body -sS \
    -H "Authorization: Bearer ${GH_TOKEN}" \
    -H "Accept: application/vnd.github+json" \
    "https://api.github.com/repos/${{ github.repository }}/dependabot/alerts?severity=critical&state=open" \
    | jq '. | length'
)"

if [ "$critical_alerts" -gt 0 ]; then
  echo "Critical Dependabot alerts found"
  exit 1
fi
```

Flow:

```text
Workflow
   ↓
Dependabot API
   ↓
Critical Alerts
   ↓
Count
   ↓
0?
 ┌─┴─┐
 ↓   ↓
YES NO
 ↓   ↓
Pass Fail
```

---

# Important Security Improvement

Never use:

```bash
-H "Authorization: Bearer "
```

with an empty or hardcoded token in a real workflow.

Use:

```bash
env:
  GH_TOKEN: ${{ github.token }}
```

and:

```bash
-H "Authorization: Bearer ${GH_TOKEN}"
```

---

# GitHub API in Jenkins

The same REST API can be called from Jenkins.

Example:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GITHUB_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}"
```

Jenkins should store the token securely in its credentials system.

---

# GitHub API in GitHub Actions vs Jenkins

### GitHub Actions

Can use:

```text
github.token
```

### Jenkins

Usually requires:

```text
Jenkins Credential
```

Both can communicate with:

```text
GitHub REST API
```

---

# REST API and GitHub Actions Comparison

```text
GitHub Actions
     |
     ↓
Native GitHub integration
     |
     ↓
GITHUB_TOKEN
```

Jenkins:

```text
Jenkins
   |
   ↓
Credential
   |
   ↓
GitHub API
```

---

# Production Use Cases

GitHub REST API can support:

```text
1. Dependabot Security Gates
2. Workflow Status Checks
3. Cross-Repository Workflow Triggers
4. Release Automation
5. PR Automation
6. Repository Governance
7. Deployment Validation
8. Commit Verification
9. Incident Investigation
10. Platform Compliance
```

---

# API Automation Architecture

```text
                 GitHub REST API
                        |
       ┌────────────────┼────────────────┐
       ↓                ↓                ↓
   Repository       Security         Actions
       |                |                |
       ↓                ↓                ↓
    Commits         Dependabot       Workflows
    Branches        Alerts           Runs
    PRs
       |
       └───────────────┬────────────────┘
                       ↓
                 DevOps Automation
                       |
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       Jenkins      GitHub       Scripts
                     Actions
```

---

# Production Workflow Example

```text
GitHub Actions
      |
      ↓
Get Commit SHA
      |
      ↓
Check Dependabot Critical Alerts
      |
      ↓
Check Required Workflow Status
      |
      ↓
Validate PR / Branch
      |
      ↓
Build
      |
      ↓
Security
      |
      ↓
ECR
      |
      ↓
Production Environment
      |
      ↓
Deploy
```

---

# GitHub API + Change Request

A production workflow can combine GitHub API with JIRA:

```text
JIRA
  ↓
Change Request
  ↓
GitHub API
  ├── Commit
  ├── PR
  ├── Checks
  └── Workflow
  ↓
Validation
  ↓
Production
```

This gives a stronger change-control process.

---

# GitHub API + Commit Validation

Example concept:

```text
Input SHA
   |
   ↓
GitHub API
   |
   ↓
Commit Exists?
   |
   ↓
Expected Repository?
   |
   ↓
Expected Branch / Release?
   |
   ↓
Continue
```

Do not trust a user-provided version string without validation.

---

# API + Release Version

Example:

```text
Release:
v1.5.2

Commit:
8a92f31

Image:
catalogue:8a92f31

Digest:
sha256:...
```

GitHub API can provide GitHub-side release and commit information.

---

# API + Production Traceability

```text
JIRA Ticket
     |
     ↓
Git Commit
     |
     ↓
GitHub Workflow
     |
     ↓
Artifact
     |
     ↓
ECR Image
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

This is the type of traceability expected in mature DevOps environments.

---

# Best Practices

```text
☐ Use authentication appropriately
☐ Apply least-privilege permissions
☐ Never hardcode tokens
☐ Never print tokens
☐ Use GITHUB_TOKEN where appropriate
☐ Prefer GitHub Apps for suitable long-lived integrations
☐ Use OIDC for cloud authentication where applicable
☐ Use --fail-with-body -sS with cURL
☐ Validate HTTP responses
☐ Handle pagination
☐ Handle rate limits
☐ Use bounded retries
☐ Parse JSON with jq
☐ Validate API inputs
☐ Avoid unnecessary API calls
☐ Use dynamic repository context where appropriate
☐ Preserve commit SHA traceability
☐ Protect production API operations
```

---

# Common Mistakes

### 1. Hardcoding tokens

```bash
TOKEN="ghp_xxxxx"
```

Never do this.

### 2. Printing tokens

Avoid:

```bash
echo "$GH_TOKEN"
```

### 3. Ignoring HTTP status

A JSON response does not automatically mean the request succeeded.

### 4. Ignoring pagination

Large repositories may return multiple pages.

### 5. Ignoring rate limits

Repeated API requests can fail.

### 6. Using excessive permissions

Use least privilege.

### 7. Hardcoding repository names

Prefer:

```text
github.repository
```

when appropriate.

### 8. No input validation

Do not trust arbitrary repository, branch, environment, or ticket input.

### 9. Infinite retries

Use bounded retries and backoff.

### 10. Using API calls unnecessarily

Reduce API traffic and improve pipeline performance.

---

# Interview Questions

## Basic

1. What is the GitHub REST API?
2. What is the GitHub API base URL?
3. What is REST?
4. What are GET, POST, PUT, PATCH, and DELETE?
5. How do you get repository information using cURL?
6. How do you get branches using the GitHub API?
7. How do you get pull requests?
8. How do you get commits?
9. What is JSON?
10. Why is `jq` useful with GitHub API?

## Intermediate

11. How do you authenticate to the GitHub API?
12. What is `GITHUB_TOKEN`?
13. How do you use `github.repository` in an API URL?
14. How do you check HTTP status codes?
15. What is API pagination?
16. Why do GitHub APIs have rate limits?
17. How do you handle API errors?
18. How do you implement bounded retries?
19. How do you retrieve workflow runs?
20. How can you trigger a workflow through the GitHub API?
21. How would you check open Dependabot alerts?
22. How would you count critical Dependabot alerts using `jq`?
23. How would you use the GitHub API from Jenkins?
24. How would you use the GitHub API from GitHub Actions?
25. How would you dynamically construct an API URL for the current repository?

## Advanced / Production

26. Design a production GitHub Actions security gate using the Dependabot API.
27. How would you prevent deployment when critical Dependabot alerts exist?
28. How would you validate a production commit using the GitHub API?
29. How would you verify that the commit being deployed belongs to the expected branch or release?
30. How would you trigger a workflow in another repository?
31. How would you monitor workflow failures using the GitHub API?
32. How would you design a repository governance automation system using the GitHub API?
33. How would you handle API pagination in a production script?
34. How would you handle GitHub API rate limits?
35. How would you securely store and use GitHub API credentials in Jenkins?
36. How would you securely use the GitHub API from GitHub Actions?
37. How would you integrate GitHub API validation with JIRA change requests?
38. How would you build a production deployment gate using commit SHA, PR status, security results, and workflow status?
39. How would you use the GitHub API to connect application repositories with a GitOps repository?
40. How would you design GitHub API automation for multiple microservices?
41. How would you prevent an API-based deployment trigger from being abused?
42. How would you implement least-privilege authentication for GitHub API automation?
43. How would you troubleshoot a GitHub API request returning 401, 403, 404, or 422?
44. How would you design retry and error handling for GitHub API calls in CI/CD?
45. Design an enterprise-grade GitHub REST API automation architecture covering GitHub Actions, Jenkins, authentication, Dependabot, workflow triggering, commit validation, JIRA/CR checks, rate limits, pagination, security gates, ECR, GitOps, ArgoCD, and EKS.