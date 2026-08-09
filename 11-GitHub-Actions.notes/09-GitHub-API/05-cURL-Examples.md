# GitHub API cURL Examples

This file contains practical cURL examples for interacting with the GitHub REST API.

The examples progress from:

```text
Basic API Requests
        ↓
Authentication
        ↓
Repositories
        ↓
Branches
        ↓
Commits
        ↓
Pull Requests
        ↓
Dependabot
        ↓
Workflows
        ↓
Workflow Runs
        ↓
Workflow Dispatch
        ↓
Production Automation
```

These examples are intended for DevOps, DevSecOps, CI/CD, GitHub Actions, Jenkins, and production automation.

---

# 1. GitHub API Base URL

The GitHub REST API base URL is:

```text
https://api.github.com
```

Basic request:

```bash
curl https://api.github.com
```

For automation, prefer:

```bash
curl -sS https://api.github.com
```

---

# 2. Basic Repository Request

```bash
curl -sS \
  "https://api.github.com/repos/OWNER/REPOSITORY"
```

Example:

```bash
curl -sS \
  "https://api.github.com/repos/daws-86s/catalogue"
```

---

# 3. Pretty Print JSON

Use `jq`:

```bash
curl -sS \
  "https://api.github.com/repos/daws-86s/catalogue" \
  | jq
```

---

# 4. Get Repository Name

```bash
curl -sS \
  "https://api.github.com/repos/${OWNER}/${REPO}" \
  | jq -r '.name'
```

---

# 5. Get Default Branch

```bash
curl -sS \
  "https://api.github.com/repos/${OWNER}/${REPO}" \
  | jq -r '.default_branch'
```

---

# 6. Get Repository Visibility

```bash
curl -sS \
  "https://api.github.com/repos/${OWNER}/${REPO}" \
  | jq -r '.visibility'
```

Possible values include:

```text
public
private
internal
```

depending on repository configuration.

---

# 7. Get Multiple Repository Fields

```bash
curl -sS \
  "https://api.github.com/repos/${OWNER}/${REPO}" \
  | jq '{
      name: .name,
      default_branch: .default_branch,
      visibility: .visibility,
      private: .private
    }'
```

---

# 8. Authentication with GITHUB_TOKEN

Inside GitHub Actions:

```yaml
env:
  GH_TOKEN: ${{ github.token }}
```

Then:

```bash
curl -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}"
```

---

# 9. Production cURL Pattern

For CI/CD:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}"
```

Important options:

```text
--fail-with-body
-s
-S
```

Together:

```text
--fail-with-body -sS
```

provide useful error handling while keeping normal output clean.

---

# 10. Store API Response

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}" \
  -o repository.json
```

Read it:

```bash
jq '.' repository.json
```

---

# 11. Get HTTP Status Code

```bash
curl -sS \
  -o response.json \
  -w "%{http_code}" \
  "https://api.github.com/repos/${OWNER}/${REPO}"
```

Example:

```text
200
```

---

# 12. Get HTTP Status and Response

```bash
http_code="$(
  curl -sS \
    -o response.json \
    -w "%{http_code}" \
    "https://api.github.com/repos/${OWNER}/${REPO}"
)"

echo "HTTP status: ${http_code}"

jq '.' response.json
```

---

# 13. Fail on HTTP Errors

Recommended:

```bash
curl --fail-with-body -sS \
  "https://api.github.com/repos/${OWNER}/${REPO}"
```

This is better for CI/CD than silently accepting an HTTP error.

---

# 14. Define Repository Variables

Instead of repeating values:

```bash
OWNER="daws-86s"
REPO="catalogue"
```

Then:

```bash
curl -sS \
  "https://api.github.com/repos/${OWNER}/${REPO}"
```

---

# 15. Dynamic Repository in GitHub Actions

Use:

```bash
https://api.github.com/repos/${{ github.repository }}
```

Example:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${{ github.repository }}"
```

---

# 16. List Branches

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/branches"
```

---

# 17. Get Branch Names

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/branches" \
  | jq -r '.[].name'
```

---

# 18. Get a Specific Branch

```bash
BRANCH="main"

curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/branches/${BRANCH}"
```

---

# 19. Get Commits

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/commits"
```

---

# 20. Get Commit SHAs

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/commits" \
  | jq -r '.[].sha'
```

---

# 21. Get Latest Commit on Main

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/commits/main" \
  | jq -r '.sha'
```

---

# 22. Get Commit Information

```bash
COMMIT_SHA="8a92f31"

curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/commits/${COMMIT_SHA}"
```

---

# 23. Extract Commit Message

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/commits/${COMMIT_SHA}" \
  | jq -r '.commit.message'
```

---

# 24. Get Pull Requests

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/pulls"
```

---

# 25. Get Open Pull Requests

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/pulls?state=open"
```

---

# 26. Get Closed Pull Requests

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/pulls?state=closed"
```

---

# 27. Extract Pull Request Information

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/pulls?state=open" \
  | jq '.[] | {
      number,
      title,
      state,
      head: .head.ref,
      base: .base.ref
    }'
```

---

# 28. Get Issues

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/issues"
```

---

# 29. Get Open Issues

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/issues?state=open"
```

---

# 30. Get Tags

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/tags"
```

Extract:

```bash
... | jq -r '.[].name'
```

---

# 31. Get Releases

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/releases"
```

---

# 32. Get Latest Release

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/releases/latest"
```

---

# 33. Get Latest Release Version

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/releases/latest" \
  | jq -r '.tag_name'
```

---

# 34. List Workflows

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows"
```

---

# 35. Extract Workflow Information

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows" \
  | jq '.workflows[] | {
      id,
      name,
      path,
      state
    }'
```

---

# 36. Get Specific Workflow

```bash
WORKFLOW="ci.yml"

curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/${WORKFLOW}"
```

---

# 37. List Workflow Runs

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs"
```

---

# 38. List Runs for Specific Workflow

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/${WORKFLOW}/runs"
```

---

# 39. Get Latest Workflow Run

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/${WORKFLOW}/runs?per_page=1" \
  | jq '.workflow_runs[0] | {
      id,
      run_number,
      status,
      conclusion,
      head_branch,
      head_sha
    }'
```

---

# 40. Get Workflow Run

```bash
RUN_ID="123456789"

curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}"
```

---

# 41. Get Workflow Run Status

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}" \
  | jq '{
      status,
      conclusion
    }'
```

---

# 42. Get Run Commit SHA

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}" \
  | jq -r '.head_sha'
```

---

# 43. Get Workflow Jobs

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}/jobs"
```

---

# 44. Get Failed Jobs

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}/jobs" \
  | jq '.jobs[] | select(.conclusion == "failure") | {
      id,
      name,
      status,
      conclusion
    }'
```

---

# 45. Download Workflow Logs

```bash
curl -L \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}/logs" \
  -o workflow-logs.zip
```

---

# 46. Cancel Workflow Run

```bash
curl --fail-with-body -sS \
  -X POST \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}/cancel"
```

---

# 47. Re-run Workflow

```bash
curl --fail-with-body -sS \
  -X POST \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}/rerun"
```

Only re-run when appropriate.

Do not blindly re-run failed production deployments.

---

# 48. Trigger Workflow Dispatch

The target workflow must support:

```yaml
on:
  workflow_dispatch:
```

API:

```bash
curl --fail-with-body -sS \
  -X POST \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  -H "Content-Type: application/json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/deploy.yml/dispatches" \
  -d '{
    "ref": "main"
  }'
```

---

# 49. Trigger Workflow with Inputs

```bash
curl --fail-with-body -sS \
  -X POST \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  -H "Content-Type: application/json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/deploy.yml/dispatches" \
  -d '{
    "ref": "main",
    "inputs": {
      "environment": "qa",
      "service": "catalogue"
    }
  }'
```

---

# 50. Dependabot Alerts

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/dependabot/alerts"
```

---

# 51. Open Dependabot Alerts

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/dependabot/alerts?state=open"
```

---

# 52. Critical Dependabot Alerts

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/dependabot/alerts?severity=critical"
```

---

# 53. Open Critical Dependabot Alerts

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/dependabot/alerts?severity=critical&state=open"
```

---

# 54. Count Critical Dependabot Alerts

```bash
critical_alerts="$(
  curl --fail-with-body -sS \
    -H "Authorization: Bearer ${GH_TOKEN}" \
    -H "Accept: application/vnd.github+json" \
    "https://api.github.com/repos/${OWNER}/${REPO}/dependabot/alerts?severity=critical&state=open" \
    | jq '. | length'
)"

echo "Critical alerts: ${critical_alerts}"
```

---

# 55. Dependabot Security Gate

```bash
set -euo pipefail

critical_alerts="$(
  curl --fail-with-body -sS \
    -H "Authorization: Bearer ${GH_TOKEN}" \
    -H "Accept: application/vnd.github+json" \
    "https://api.github.com/repos/${OWNER}/${REPO}/dependabot/alerts?severity=critical&state=open" \
    | jq '. | length'
)"

if [ "$critical_alerts" -gt 0 ]; then
  echo "Critical Dependabot alerts found."
  exit 1
fi

echo "Dependabot security gate passed."
```

---

# 56. Save Dependabot Report

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/dependabot/alerts?state=open" \
  -o dependabot-alerts.json
```

---

# 57. Extract Dependabot Package Names

```bash
jq -r \
  '.[].dependency.package.name' \
  dependabot-alerts.json
```

---

# 58. Extract Dependabot Severities

```bash
jq -r \
  '.[].security_advisory.severity' \
  dependabot-alerts.json
```

---

# 59. Generate Dependabot Summary

```bash
jq -r '
  .[] |
  [
    .number,
    .dependency.package.name,
    .security_advisory.severity,
    .state
  ] |
  @tsv
' dependabot-alerts.json
```

---

# 60. API with Variables

Reusable script:

```bash
#!/usr/bin/env bash

set -euo pipefail

OWNER="${OWNER:?OWNER is required}"
REPO="${REPO:?REPO is required}"
GH_TOKEN="${GH_TOKEN:?GH_TOKEN is required}"

API_URL="https://api.github.com/repos/${OWNER}/${REPO}"

curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "$API_URL"
```

---

# 61. API Helper Function

A reusable Bash function:

```bash
github_api() {
  local url="$1"

  curl --fail-with-body -sS \
    -H "Authorization: Bearer ${GH_TOKEN}" \
    -H "Accept: application/vnd.github+json" \
    "$url"
}
```

Use:

```bash
github_api \
  "https://api.github.com/repos/${OWNER}/${REPO}"
```

---

# 62. POST Helper Function

```bash
github_api_post() {
  local url="$1"
  local payload="$2"

  curl --fail-with-body -sS \
    -X POST \
    -H "Authorization: Bearer ${GH_TOKEN}" \
    -H "Accept: application/vnd.github+json" \
    -H "Content-Type: application/json" \
    "$url" \
    -d "$payload"
}
```

---

# 63. Trigger Workflow Using Function

```bash
payload='{
  "ref": "main"
}'

github_api_post \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/deploy.yml/dispatches" \
  "$payload"
```

---

# 64. POST JSON with jq

Avoid manually constructing complex JSON when values are dynamic.

Example:

```bash
payload="$(
  jq -n \
    --arg ref "main" \
    --arg environment "qa" \
    --arg service "catalogue" \
    '{
      ref: $ref,
      inputs: {
        environment: $environment,
        service: $service
      }
    }'
)"
```

Then:

```bash
github_api_post \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/deploy.yml/dispatches" \
  "$payload"
```

---

# 65. API Pagination

Simple page request:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/commits?per_page=100&page=1"
```

Next page:

```bash
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/commits?per_page=100&page=2"
```

---

# 66. Pagination Loop

Example:

```bash
page=1

while true; do

  response="$(
    curl --fail-with-body -sS \
      -H "Authorization: Bearer ${GH_TOKEN}" \
      -H "Accept: application/vnd.github+json" \
      "https://api.github.com/repos/${OWNER}/${REPO}/commits?per_page=100&page=${page}"
  )"

  count="$(echo "$response" | jq 'length')"

  echo "Page ${page}: ${count} records"

  if [ "$count" -eq 0 ]; then
    break
  fi

  echo "$response" | jq -r '.[].sha'

  page=$((page + 1))
done
```

For production systems, consider using GitHub CLI or a dedicated API client when pagination, rate-limit handling, and retries become complex.

---

# 67. API Error Handling

```bash
if ! response="$(
  curl --fail-with-body -sS \
    -H "Authorization: Bearer ${GH_TOKEN}" \
    -H "Accept: application/vnd.github+json" \
    "$API_URL"
)"; then

  echo "GitHub API request failed."
  exit 1
fi

echo "$response"
```

---

# 68. Retry Concept

A production API client should use bounded retries.

Conceptually:

```text
Attempt 1
   ↓
Failure
   ↓
Wait
   ↓
Attempt 2
   ↓
Failure
   ↓
Wait Longer
   ↓
Attempt 3
   ↓
Success / Final Failure
```

Do not retry indefinitely.

---

# 69. API and Rate Limits

For large automation:

```text
Avoid unnecessary requests
Use pagination
Use appropriate filtering
Cache data when appropriate
Respect rate limits
Use bounded retries
```

---

# 70. Production Security Rules

Never:

```bash
echo "$GH_TOKEN"
```

Never:

```bash
TOKEN="ghp_..."
```

Never:

```text
Commit tokens
Put tokens in URLs
Print Authorization headers
```

Prefer:

```bash
-H "Authorization: Bearer ${GH_TOKEN}"
```

---

# 71. GitHub Actions API Workflow

Complete example:

```yaml
name: GitHub API Validation

on:
  workflow_dispatch:

permissions:
  actions: read
  contents: read

jobs:

  api-check:

    runs-on: ubuntu-latest

    steps:

      - name: Get Latest Workflow Run
        env:
          GH_TOKEN: ${{ github.token }}
        run: |

          set -euo pipefail

          curl --fail-with-body -sS \
            -H "Authorization: Bearer ${GH_TOKEN}" \
            -H "Accept: application/vnd.github+json" \
            "https://api.github.com/repos/${{ github.repository }}/actions/runs?per_page=1" \
            | jq '.workflow_runs[0] | {
                id,
                run_number,
                status,
                conclusion,
                head_sha
              }'
```

---

# 72. Production Dependabot Gate

```yaml
name: Dependency Security Gate

on:
  pull_request:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:

  security:

    runs-on: ubuntu-latest

    steps:

      - name: Check Dependabot
        env:
          GH_TOKEN: ${{ github.token }}
        run: |

          set -euo pipefail

          curl --fail-with-body -sS \
            -H "Authorization: Bearer ${GH_TOKEN}" \
            -H "Accept: application/vnd.github+json" \
            "https://api.github.com/repos/${{ github.repository }}/dependabot/alerts?severity=critical&state=open" \
            -o dependabot-alerts.json

          critical_alerts="$(
            jq '. | length' dependabot-alerts.json
          )"

          echo "Critical alerts: ${critical_alerts}"

          if [ "$critical_alerts" -gt 0 ]; then
            echo "Security gate failed."
            exit 1
          fi
```

---

# 73. Production Workflow Trigger

External automation:

```bash
set -euo pipefail

OWNER="my-org"
REPO="deployment"
WORKFLOW="deploy.yml"
REF="main"

curl --fail-with-body -sS \
  -X POST \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  -H "Content-Type: application/json" \
  "https://api.github.com/repos/${OWNER}/${REPO}/actions/workflows/${WORKFLOW}/dispatches" \
  -d "$(jq -n \
    --arg ref "$REF" \
    --arg environment "production" \
    '{
      ref: $ref,
      inputs: {
        environment: $environment
      }
    }'
  )"
```

This should only be used with appropriate production authorization and workflow protections.

---

# 74. Poll Workflow Status

After triggering a workflow, an external system may need to wait for the resulting run.

Conceptually:

```text
Trigger
  ↓
Wait
  ↓
Check Run
  ↓
Completed?
 ┌─┴─┐
No  Yes
 |    |
Wait Result
```

Do not poll indefinitely.

---

# 75. Basic Polling Example

```bash
for attempt in {1..30}; do

  status="$(
    curl --fail-with-body -sS \
      -H "Authorization: Bearer ${GH_TOKEN}" \
      -H "Accept: application/vnd.github+json" \
      "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}" \
      | jq -r '.status'
  )"

  echo "Workflow status: ${status}"

  if [ "$status" = "completed" ]; then
    break
  fi

  sleep 10

done
```

Use a bounded timeout.

---

# 76. Poll Conclusion

```bash
conclusion="$(
  curl --fail-with-body -sS \
    -H "Authorization: Bearer ${GH_TOKEN}" \
    -H "Accept: application/vnd.github+json" \
    "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}" \
    | jq -r '.conclusion'
)"

echo "Conclusion: ${conclusion}"
```

Then:

```bash
if [ "$conclusion" != "success" ]; then
  exit 1
fi
```

---

# 77. Production Polling Pattern

```bash
set -euo pipefail

MAX_ATTEMPTS=30
SLEEP_SECONDS=10

for ((attempt=1; attempt<=MAX_ATTEMPTS; attempt++)); do

  response="$(
    curl --fail-with-body -sS \
      -H "Authorization: Bearer ${GH_TOKEN}" \
      -H "Accept: application/vnd.github+json" \
      "https://api.github.com/repos/${OWNER}/${REPO}/actions/runs/${RUN_ID}"
  )"

  status="$(echo "$response" | jq -r '.status')"
  conclusion="$(echo "$response" | jq -r '.conclusion // empty')"

  echo "Attempt: ${attempt}"
  echo "Status: ${status}"
  echo "Conclusion: ${conclusion}"

  if [ "$status" = "completed" ]; then

    if [ "$conclusion" = "success" ]; then
      echo "Workflow succeeded."
      exit 0
    fi

    echo "Workflow completed unsuccessfully."
    exit 1
  fi

  sleep "$SLEEP_SECONDS"

done

echo "Workflow did not complete within the allowed time."
exit 1
```

---

# 78. Production Cross-Repository Flow

```text
Application Repository
        |
        ↓
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
GitHub Workflow API
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

---

# 79. Production API Gate

Before triggering deployment:

```text
Commit
 ↓
PR
 ↓
Tests
 ↓
Dependabot
 ↓
Trivy
 ↓
SonarQube
 ↓
Veracode
 ↓
Artifact
 ↓
Change Request
 ↓
GitHub API
 ↓
Production Workflow
```

---

# 80. Production Deployment Validation

After deployment:

```text
Workflow API
      |
      ↓
Workflow Run
      |
      ↓
Jobs
      |
      ↓
Deployment Job
      |
      ↓
Status
      |
      ↓
Success?
   ┌──┴──┐
  YES    NO
   |      |
   ↓      ↓
Continue Incident
         / Rollback
```

---

# 81. Workflow API + ECR

Example architecture:

```text
GitHub Actions
      |
      ↓
Build Docker Image
      |
      ↓
Security Scan
      |
      ↓
ECR
      |
      ↓
GitHub Workflow API
      |
      ↓
Deployment Workflow
```

Use immutable image references where possible.

Example:

```text
sha256:...
```

rather than relying only on mutable tags.

---

# 82. Workflow API + ArgoCD + EKS

```text
GitHub
  |
  ↓
Actions
  |
  ↓
Build
  |
  ↓
ECR
  |
  ↓
GitOps Update
  |
  ↓
ArgoCD
  |
  ↓
EKS
```

Workflow API can coordinate the GitHub-side workflow transitions.

---

# 83. Full DevSecOps Production Workflow

```text
Developer
   |
   ↓
Pull Request
   |
   ↓
GitHub Actions
   |
   ├── Unit Tests
   ├── SonarQube
   ├── Dependabot
   ├── Trivy
   └── Veracode
   |
   ↓
Security Gate
   |
   ↓
Docker Build
   |
   ↓
ECR
   |
   ↓
GitOps Update
   |
   ↓
ArgoCD
   |
   ↓
EKS
   |
   ↓
Smoke Tests
   |
   ↓
Production Validation
```

---

# 84. Troubleshooting API Authentication

If you receive:

```text
401 Unauthorized
```

check:

```text
Token
Token format
Credential validity
Authentication header
```

Example:

```bash
-H "Authorization: Bearer ${GH_TOKEN}"
```

---

# 85. Troubleshooting 403

A 403 may indicate:

```text
Insufficient Permissions
Access Restrictions
Rate Limit
Organization Policy
```

Check:

```text
Token Permissions
Workflow permissions
Repository access
Organization policy
API rate limit
```

---

# 86. Troubleshooting 404

Check:

```text
Owner
Repository
Workflow
Workflow ID
Run ID
Repository visibility
Token access
```

A resource may also appear unavailable when the authenticated identity cannot access it.

---

# 87. Troubleshooting 422

A 422 commonly indicates a validation problem.

For workflow dispatch, check:

```text
Workflow exists
workflow_dispatch is configured
ref exists
Input names are correct
Input values are valid
```

---

# 88. API Debugging

Do not expose credentials.

Use:

```bash
curl -v
```

carefully because verbose output can expose sensitive request information.

Prefer controlled debugging:

```bash
curl --fail-with-body -sS \
  -o response.json \
  -w "%{http_code}\n" \
  ...
```

Then inspect:

```bash
jq '.' response.json
```

---

# 89. Common cURL Mistakes

### Missing quotes

Bad:

```bash
curl https://api.github.com/repos/$OWNER/$REPO
```

Better:

```bash
curl "https://api.github.com/repos/${OWNER}/${REPO}"
```

### Missing headers

For authenticated API calls:

```bash
-H "Authorization: Bearer ${GH_TOKEN}"
```

### Wrong HTTP method

Dispatch requires:

```bash
-X POST
```

not:

```bash
-X GET
```

### Invalid JSON

Validate dynamic payloads with:

```bash
jq
```

---

# 90. Common Security Mistakes

Never:

```text
Hardcode PAT
Hardcode GitHub App private key
Print token
Put token in URL
Commit secrets
Use broad permissions
Expose production credentials to fork PRs
```

Prefer:

```text
GITHUB_TOKEN
GitHub App
Least Privilege
Secrets
OIDC
Environment Protection
```

---

# 91. cURL Cheat Sheet

```bash
# GET
curl -sS "$URL"

# Authenticated GET
curl -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  "$URL"

# POST
curl -sS \
  -X POST \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Content-Type: application/json" \
  "$URL" \
  -d "$PAYLOAD"

# Save response
curl -sS "$URL" -o response.json

# HTTP status
curl -sS -o response.json -w "%{http_code}" "$URL"

# Production GET
curl --fail-with-body -sS \
  -H "Authorization: Bearer ${GH_TOKEN}" \
  -H "Accept: application/vnd.github+json" \
  "$URL"
```

---

# 92. jq Cheat Sheet

```bash
# Pretty print
jq '.'

# String
jq -r '.name'

# Array length
jq 'length'

# Array elements
jq '.[]'

# Filter
jq '.[] | select(.state == "open")'

# Multiple fields
jq '{
  name: .name,
  state: .state
}'
```

---

# 93. Production cURL Checklist

```text
☐ Use HTTPS
☐ Authenticate securely
☐ Use least privilege
☐ Use --fail-with-body
☐ Use -sS
☐ Use Accept header
☐ Check HTTP status
☐ Validate JSON
☐ Handle pagination
☐ Handle rate limits
☐ Use bounded retries
☐ Never log tokens
☐ Validate inputs
☐ Use immutable references
☐ Preserve commit SHA
```

---

# 94. Interview Scenarios

## Scenario 1

A production workflow is triggered through the GitHub API but returns 422.

Check:

```text
workflow_dispatch
workflow file
workflow name
ref
inputs
input names
```

---

## Scenario 2

The API returns 403.

Check:

```text
Token permissions
Repository access
Actions permissions
Organization policy
Rate limit
```

---

## Scenario 3

Dependabot API returns an error.

Do not:

```text
Assume zero vulnerabilities
```

Instead:

```text
Retry if transient
Investigate
Fail security gate if required
```

---

## Scenario 4

A workflow fails after deployment begins.

Do not immediately:

```text
Re-run
```

First determine:

```text
What changed?
Which job failed?
What was deployed?
Was the deployment partially completed?
Is rollback required?
```

---

## Scenario 5

Jenkins needs to trigger a GitHub deployment workflow.

Architecture:

```text
Jenkins
   |
   ↓
Secure Credential / GitHub App
   |
   ↓
GitHub REST API
   |
   ↓
workflow_dispatch
   |
   ↓
GitHub Actions
   |
   ↓
Production Environment
```

---

# 95. Production Interview Scenario

### Problem

You have:

```text
20 microservices
GitHub Actions
ECR
ArgoCD
EKS
Jenkins
JIRA
SonarQube
Trivy
Veracode
Dependabot
```

You need to trigger production deployments automatically through the GitHub API.

### Design

```text
Developer
   |
   ↓
PR
   |
   ↓
GitHub Actions
   |
   ├── Tests
   ├── SonarQube
   ├── Dependabot
   ├── Trivy
   └── Veracode
   |
   ↓
Security Gate
   |
   ↓
Docker Build
   |
   ↓
ECR
   |
   ↓
JIRA Change Validation
   |
   ↓
GitHub Workflow API
   |
   ↓
Production Workflow
   |
   ↓
GitHub Environment
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

---

# 96. Production Requirements

The automation should implement:

```text
Authentication
Authorization
Least Privilege
Input Validation
Security Gates
Change Management
Immutable Artifacts
Workflow Concurrency
Timeouts
API Error Handling
Rate-Limit Handling
Retries
Auditability
Deployment Verification
Rollback Strategy
```

---

# 97. Important Production Principle

The GitHub API should be treated as an automation interface, not as a replacement for deployment safety controls.

Do not build:

```text
API
 ↓
Production
```

Build:

```text
API
 ↓
Validation
 ↓
Security
 ↓
Authorization
 ↓
Environment Protection
 ↓
Deployment
 ↓
Verification
```

---

# 98. Final Architecture

```text
                         GitHub REST API
                               |
        ┌──────────────────────┼──────────────────────┐
        ↓                      ↓                      ↓
  Repository API         Dependabot API        Workflow API
        |                      |                      |
        ↓                      ↓                      ↓
 Commits / PRs             Security             Actions
 Branches / Tags            Alerts              Runs
 Releases                                         Jobs
                                                   |
                                                   ↓
                                            GitHub Actions
                                                   |
                                  ┌────────────────┼────────────────┐
                                  ↓                ↓                ↓
                               Build           Security          Deploy
                                  |                |                |
                                  ↓                ↓                ↓
                                 ECR          DevSecOps       GitHub Env
                                                   |                |
                                                   ↓                ↓
                                           SonarQube/Trivy       Approval
                                           Veracode/Dependabot      |
                                                                    ↓
                                                                  GitOps
                                                                    |
                                                                    ↓
                                                                  ArgoCD
                                                                    |
                                                                    ↓
                                                                   EKS
```

---

# Key Takeaways

GitHub API automation allows DevOps teams to programmatically interact with GitHub.

Core operations:

```text
GET Repository
GET Branches
GET Commits
GET Pull Requests
GET Dependabot Alerts
GET Workflows
GET Workflow Runs
GET Jobs
GET Logs
POST Workflow Dispatch
POST Re-run
POST Cancel
```

For production automation:

```text
Use authentication
Use least privilege
Validate inputs
Check HTTP responses
Handle pagination
Handle rate limits
Use bounded retries
Protect production environments
Preserve commit SHA
Use immutable artifacts
Maintain auditability
```

A strong production workflow is:

```text
Code
 ↓
PR
 ↓
CI
 ↓
Security
 ↓
Build
 ↓
ECR
 ↓
Change Approval
 ↓
GitHub Workflow API
 ↓
Production Workflow
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
 ↓
Verification
```

The GitHub REST API becomes especially powerful when combined with GitHub Actions, Jenkins, Dependabot, SonarQube, Trivy, Veracode, ECR, GitOps, ArgoCD, and EKS.