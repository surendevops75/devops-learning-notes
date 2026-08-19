# Python API Automation for DevOps

## 1. Overview

The previous files established:

```text
01-HTTP-and-REST
02-Requests-Library
```

Now we move from understanding HTTP and Requests to building **real DevOps API automation**.

The goal is not simply:

```python
requests.get(url)
```

The goal is:

```text
Authenticate
    |
    v
Call API
    |
    v
Validate response
    |
    v
Handle errors
    |
    v
Poll asynchronous operation
    |
    v
Reconcile state
    |
    v
Take next action
    |
    v
Verify final state
    |
    v
Report result
```

This is how Python becomes a DevOps automation/orchestration tool.

---

# 2. What Is API Automation?

API automation means using software to interact with APIs without requiring manual human actions.

Example:

Manual:

```text
Open Jenkins
    |
Click Build
    |
Wait
    |
Open ArgoCD
    |
Check sync
    |
Open Kubernetes
    |
Check pods
```

Automated:

```text
Python
 |
 +-- Trigger Jenkins
 |
 +-- Check build
 |
 +-- Verify artifact
 |
 +-- Check ArgoCD
 |
 +-- Verify Kubernetes
 |
 +-- Verify application health
 |
 v
Release result
```

---

# 3. Why API Automation Matters in DevOps

Modern platforms expose APIs because automation needs programmatic control.

Examples:

```text
GitHub API
Jenkins API
GitLab API
AWS APIs
ECR APIs
Kubernetes API
ArgoCD API
SonarQube API
Prometheus API
Elasticsearch API
```

A DevOps engineer should be able to connect these systems.

---

# 4. Automation vs Orchestration

These terms are related but different.

### Automation

Automates one operation.

Example:

```text
Trigger Jenkins build
```

### Orchestration

Coordinates multiple operations.

Example:

```text
GitHub
  |
  v
Jenkins
  |
  v
Security scan
  |
  v
ECR
  |
  v
GitOps
  |
  v
ArgoCD
  |
  v
EKS
  |
  v
Health verification
```

Python is particularly useful for orchestration.

---

# 5. Basic API Automation Flow

A generic workflow:

```text
1. Load configuration
2. Load credentials
3. Create API session
4. Validate connectivity
5. Execute API operation
6. Validate response
7. Poll if asynchronous
8. Handle transient errors
9. Verify desired state
10. Emit logs/metrics
11. Return success/failure
```

---

# 6. Production Architecture

```text
                    CI/CD Pipeline
                          |
                          v
                 Python Automation
                          |
                +---------+---------+
                |                   |
                v                   v
          Configuration         Secret Store
                |                   |
                +---------+---------+
                          |
                          v
                    API Clients
                          |
        +-----------------+-----------------+
        |                 |                 |
        v                 v                 v
     GitHub            Jenkins           AWS
        |                 |                 |
        |                 |                ECR
        |                 |                 |
        +-----------------+-----------------+
                          |
                          v
                       ArgoCD
                          |
                          v
                         EKS
                          |
                +---------+---------+
                |                   |
                v                   v
           Prometheus              ELK
```

---

# 7. Automation Design Principles

Production API automation should be:

```text
Idempotent
Retryable where safe
Observable
Secure
Bounded
Testable
Maintainable
Environment-aware
```

---

# 8. Configuration Management

Do not hardcode:

```python
API_URL = "https://prod.example.com"
```

Instead:

```python
import os

api_url = os.environ["API_URL"]
```

For larger systems, use a configuration object.

```python
from dataclasses import dataclass


@dataclass
class Config:

    api_url: str
    timeout: tuple
    environment: str
```

---

# 9. Environment Separation

Typical:

```text
dev
staging
prod
```

Configuration:

```text
DEV    -> dev API
STAGING -> staging API
PROD   -> production API
```

Never allow a typo such as:

```text
prodution
```

to silently select production.

Validate environment values.

---

# 10. Configuration Validation

```python
ALLOWED_ENVIRONMENTS = {
    "dev",
    "staging",
    "prod"
}

if environment not in ALLOWED_ENVIRONMENTS:
    raise ValueError(
        f"Unsupported environment: {environment}"
    )
```

Fail early.

---

# 11. Secret Management

Credentials should come from:

```text
CI secret store
AWS Secrets Manager
HashiCorp Vault
Kubernetes Secret
OIDC
Environment injection
```

depending on architecture.

Do not store production secrets in:

```text
Git
Docker image
Source code
README
Logs
```

---

# 12. Least Privilege

An automation token should have only the permissions required.

Bad:

```text
Admin token for everything
```

Better:

```text
Read repository
Trigger workflow
Read deployment
```

This reduces blast radius.

---

# 13. API Client Layer

A reusable generic client:

```python
import requests


class APIClient:

    def __init__(
        self,
        base_url,
        token,
        timeout=(5, 20)
    ):
        self.base_url = base_url.rstrip("/")
        self.timeout = timeout

        self.session = requests.Session()

        self.session.headers.update({
            "Authorization": f"Bearer {token}",
            "Accept": "application/json",
            "User-Agent":
                "devops-automation/1.0"
        })

    def get(self, path, **kwargs):

        return self.session.get(
            self.base_url + path,
            timeout=self.timeout,
            **kwargs
        )

    def post(self, path, **kwargs):

        return self.session.post(
            self.base_url + path,
            timeout=self.timeout,
            **kwargs
        )
```

This centralizes transport behavior.

---

# 14. Service-Specific Clients

Build specialized clients above the generic client.

```text
APIClient
   |
   +-- GitHubClient
   +-- JenkinsClient
   +-- ArgoCDClient
   +-- SonarQubeClient
   +-- PrometheusClient
```

Each service client understands its own API.

---

# 15. Business Logic Layer

Example:

```text
APIClient
     |
     v
ArgoCDClient
     |
     v
DeploymentOrchestrator
```

The orchestrator should not need to know:

```text
requests.Session
headers
URL construction
HTTP adapters
```

It should think in business operations:

```text
sync application
wait for healthy
verify revision
```

---

# 16. First Real Automation — GitHub Repository Check

Goal:

```text
Check whether repository exists.
```

Flow:

```text
Python
 |
 v
GitHub API
 |
 v
Repository
 |
 +-- 200 -> exists
 +-- 404 -> not found
 +-- 401 -> authentication issue
 +-- 403 -> permission/rate limit
```

---

# 17. GitHub Repository Check

Conceptual:

```python
def repository_exists(
    github,
    owner,
    repo
):
    response = github.get(
        f"/repos/{owner}/{repo}"
    )

    if response.status_code == 200:
        return True

    if response.status_code == 404:
        return False

    response.raise_for_status()
```

---

# 18. Why Not Treat Every 404 as Failure?

A 404 can be an expected business result.

For:

```text
repository_exists()
```

404 means:

```text
Repository does not exist
```

For:

```text
get_production_deployment()
```

404 might mean:

```text
Critical state missing
```

The API client should expose the information; business logic decides what it means.

---

# 19. GitHub Workflow Automation

Example flow:

```text
Python
 |
 | GET repository
 v
Repository exists
 |
 | POST workflow dispatch
 v
Workflow started
 |
 | GET workflow status
 v
Completed
 |
 +-- success
 +-- failure
```

---

# 20. Triggering a CI Workflow

Conceptual:

```python
payload = {
    "ref": "main",
    "inputs": {
        "environment": "staging"
    }
}

response = github.post(
    f"/repos/{owner}/{repo}/actions/workflows/{workflow}/dispatches",
    json=payload
)

if response.status_code not in {200, 201, 202, 204}:
    response.raise_for_status()
```

The exact status and endpoint behavior depends on the API.

---

# 21. Trigger Does Not Mean Success

A workflow trigger may return:

```text
Accepted
```

The pipeline still needs to:

```text
Find workflow run
Poll status
Check conclusion
```

---

# 22. Workflow Polling

Conceptual:

```python
while not deadline_expired():

    run = github.get(
        workflow_status_path
    )

    status = run["status"]
    conclusion = run.get("conclusion")

    if status == "completed":
        if conclusion == "success":
            return True

        raise RuntimeError(
            f"Workflow failed: {conclusion}"
        )

    sleep_with_backoff()
```

---

# 23. Jenkins Automation

Typical flow:

```text
Python
 |
 | POST
 v
Jenkins
 |
 v
Build queued
 |
 | GET queue
 v
Build number
 |
 | GET build
 v
Running
 |
 v
Success/Failure
```

This is a classic asynchronous API pattern.

---

# 24. Jenkins Queue

When a build is triggered, Jenkins may first create a queue item.

Therefore:

```text
POST build
   |
   v
queue item
   |
   v
build number
```

The automation must understand this lifecycle.

---

# 25. Jenkins Build Polling

```text
QUEUED
  |
  v
STARTED
  |
  v
RUNNING
  |
  +----> SUCCESS
  |
  +----> FAILURE
  |
  +----> ABORTED
```

Do not treat the trigger response as final build status.

---

# 26. Artifact Verification

After CI completes:

```text
Build
 |
 v
Artifact
 |
 v
Registry
```

Python can verify:

```text
Image exists
Tag exists
Digest matches
Build metadata exists
```

---

# 27. Tag vs Digest

Tag:

```text
payment:1.5.0
```

can move.

Digest:

```text
sha256:abcd...
```

identifies immutable image content.

Production deployments should prefer immutable references where supported.

---

# 28. ECR Verification

Automation can verify:

```text
Repository
Image tag
Image digest
Push timestamp
```

Conceptually:

```python
image = ecr.describe_image(
    repository="payment",
    tag="release-123"
)
```

For AWS-native operations, prefer the AWS SDK where appropriate rather than manually calling raw HTTP endpoints.

---

# 29. Security Gate Automation

CI pipeline:

```text
Build
 |
 v
Unit Tests
 |
 v
SonarQube
 |
 v
Trivy
 |
 v
Veracode
 |
 v
Approval
 |
 v
Artifact
```

Python can query APIs to verify gate state.

---

# 30. SonarQube Quality Gate

Conceptual:

```text
Python
 |
 v
SonarQube API
 |
 v
Quality Gate
 |
 +-- OK
 |
 +-- ERROR
```

Do not deploy if a mandatory quality gate fails.

---

# 31. Trivy Integration

Trivy commonly runs as a CLI.

Python can:

```text
Run Trivy
Parse output
Evaluate severity
Fail pipeline
```

Or consume results produced by the CI platform.

API automation and CLI automation can coexist.

---

# 32. Veracode Integration

Python can retrieve:

```text
Scan status
Policy status
Findings
```

through the provider's supported APIs.

The automation should enforce the organization's security policy rather than hardcoding assumptions.

---

# 33. GitOps Automation

Typical architecture:

```text
Build
 |
 v
ECR
 |
 v
Update Git manifest
 |
 v
GitHub
 |
 v
ArgoCD
 |
 v
EKS
```

Python can coordinate verification around this workflow.

---

# 34. GitOps Principle

Python should not become the source of truth for Kubernetes state.

Instead:

```text
Git
=
desired state
```

Python should update or coordinate the GitOps process according to the approved workflow.

---

# 35. Example GitOps Flow

```text
1. Build image
2. Push image to ECR
3. Get image digest
4. Update deployment manifest
5. Commit to Git
6. ArgoCD detects change
7. ArgoCD syncs
8. Kubernetes rolls out
9. Python verifies health
```

---

# 36. Why Digest Verification Matters

Suppose:

```text
payment:latest
```

was pushed.

Another build can move the tag.

Instead:

```text
payment@sha256:...
```

identifies exact image content.

The automation should verify that the deployed digest matches the intended artifact.

---

# 37. ArgoCD Automation

Conceptual:

```text
Python
 |
 v
ArgoCD API
 |
 +-- Get application
 +-- Trigger sync if policy allows
 +-- Get operation
 +-- Poll
 +-- Verify health
```

---

# 38. ArgoCD Desired vs Live State

GitOps:

```text
Git
 |
 | desired state
 v
ArgoCD
 |
 | reconciliation
 v
Kubernetes
 |
 | live state
```

Python should verify the final live state.

---

# 39. ArgoCD Verification

Check:

```text
Sync status
Health status
Operation phase
Revision
Resources
```

Example desired result:

```text
Sync = Synced
Health = Healthy
Operation = Succeeded
Revision = expected
```

---

# 40. Kubernetes Verification

After ArgoCD:

```text
Python
 |
 v
Kubernetes API
 |
 +-- Deployment
 +-- ReplicaSet
 +-- Pods
 +-- Services
```

Verify:

```text
Available replicas
Ready replicas
Pod states
Container readiness
```

---

# 41. Kubernetes API vs kubectl

Both can interact with Kubernetes.

### kubectl

```text
CLI
```

### Kubernetes Python client

```text
Python API client
```

API automation is useful when the workflow itself is written in Python.

---

# 42. Deployment Verification

Example conceptual:

```python
deployment = apps_api.read_namespaced_deployment(
    name="payment",
    namespace="prod"
)

desired = deployment.spec.replicas
ready = deployment.status.ready_replicas or 0

if ready < desired:
    raise RuntimeError(
        "Deployment not ready"
    )
```

---

# 43. Health Endpoint Verification

After Kubernetes rollout:

```text
GET /health
```

Then:

```text
HTTP 200
+
healthy response
```

should be validated.

Example:

```python
response = session.get(
    health_url,
    timeout=10
)

response.raise_for_status()

data = response.json()

if data.get("status") != "healthy":
    raise RuntimeError(
        "Application health check failed"
    )
```

---

# 44. Multi-Level Verification

Do not verify only one layer.

Use:

```text
Layer 1:
API accepted request

Layer 2:
CI build succeeded

Layer 3:
Artifact exists

Layer 4:
GitOps revision updated

Layer 5:
ArgoCD synced

Layer 6:
Kubernetes rollout succeeded

Layer 7:
Application health endpoint succeeds

Layer 8:
Metrics/logs show healthy behavior
```

This is production-grade verification.

---

# 45. Deployment State Machine

Model deployment as:

```text
INIT
 |
 v
VALIDATED
 |
 v
BUILDING
 |
 v
SCANNING
 |
 v
PUBLISHED
 |
 v
GITOPS_UPDATED
 |
 v
SYNCING
 |
 v
ROLLED_OUT
 |
 v
HEALTHY
 |
 v
SUCCESS
```

Failure from any stage should be explicit.

---

# 46. Failure States

```text
BUILD_FAILED
SCAN_FAILED
PUBLISH_FAILED
GITOPS_FAILED
SYNC_FAILED
ROLLOUT_FAILED
HEALTH_CHECK_FAILED
TIMEOUT
```

Do not return a generic:

```text
Automation failed
```

when a specific failure can be identified.

---

# 47. State Machine Benefits

It helps with:

```text
Retries
Recovery
Observability
Audit
Resume after crash
Interview explanations
```

---

# 48. Resume After Failure

Suppose Python crashes after:

```text
ECR push
```

but before:

```text
Git update
```

On restart:

```text
Check artifact
 |
 v
Already exists
 |
 v
Skip build/push
 |
 v
Continue GitOps
```

This is much better than starting from zero.

---

# 49. Idempotent Automation

An operation is idempotent when running it multiple times produces the same intended state.

Example:

```text
Ensure repository exists
Ensure branch exists
Ensure manifest has image digest
Ensure deployment has replicas=3
```

These are generally easier to make idempotent than:

```text
Trigger deployment
Create release
Send notification
```

---

# 50. Ensure vs Execute

Good automation often uses:

```text
ensure_state()
```

rather than only:

```text
execute_action()
```

Example:

```python
ensure_image_exists()
ensure_manifest_updated()
ensure_application_synced()
ensure_application_healthy()
```

This makes recovery easier.

---

# 51. Reconciliation

Reconciliation means:

```text
Desired state
     |
     v
Current state
     |
     v
Difference
     |
     v
Action
     |
     v
Verify
```

This is the same fundamental idea behind GitOps controllers.

---

# 52. API Automation as Reconciliation

Example:

```text
Desired:
image digest = sha256:abc

Current:
image digest = sha256:def

Difference:
image mismatch

Action:
update Git manifest

Verify:
ArgoCD + Kubernetes
```

---

# 53. Dry Run

Production automation should sometimes support:

```bash
python release.py \
  --environment prod \
  --dry-run
```

Dry run should show:

```text
What would change
Which APIs would be called
Which resources would be affected
```

without making destructive changes.

---

# 54. Approval Gates

Production deployments may require:

```text
Human approval
```

Python should not bypass organizational controls.

Architecture:

```text
Automation
 |
 v
Validation
 |
 v
Approval
 |
 v
Production change
```

---

# 55. Audit Trail

Record:

```text
Who triggered
What release
What artifact
What digest
What environment
What commit
What deployment
What result
When
Request IDs
```

This is important for incident investigation and compliance.

---

# 56. Release Metadata

Example:

```json
{
  "release_id": "rel-2026-001",
  "commit": "abc123",
  "image_digest": "sha256:...",
  "environment": "prod",
  "requested_by": "ci",
  "status": "success"
}
```

Use immutable release identifiers.

---

# 57. Correlation ID

Use one release ID across systems:

```text
GitHub
   |
Jenkins
   |
Python
   |
ArgoCD
   |
Kubernetes
   |
Logs
```

Example:

```text
release_id=rel-2026-001
```

This makes cross-system troubleshooting much easier.

---

# 58. API Automation Logging

Log:

```text
release_id
environment
service
operation
endpoint/path
status
duration
attempt
request_id
```

Do not log:

```text
token
password
private key
secret payload
```

---

# 59. Structured Logging

Prefer:

```json
{
  "event": "deployment_check",
  "release_id": "rel-123",
  "service": "payment",
  "status": "healthy",
  "duration_ms": 420
}
```

over unstructured messages when logs will be queried by machines.

---

# 60. Metrics

Expose metrics such as:

```text
automation_runs_total
automation_failures_total
automation_duration_seconds
api_requests_total
api_retries_total
deployment_success_total
deployment_failure_total
```

This allows Prometheus/Grafana monitoring.

---

# 61. Automation SLO

For example:

```text
99% of staging deployments complete within 15 minutes
```

Track:

```text
Success rate
Latency
Timeouts
Dependency failures
```

This turns automation into an observable production service.

---

# 62. Concurrency

Suppose Python must deploy:

```text
20 services
```

Options:

```text
Sequential
Parallel
Bounded parallel
```

Do not immediately use unlimited parallelism.

---

# 63. Bounded Concurrency

Concept:

```text
20 services
 |
 v
Worker pool = 4
 |
 +-- service 1
 +-- service 2
 +-- service 3
 +-- service 4
 |
 v
Next batch
```

This protects:

```text
API
CI runner
Cluster
Network
```

---

# 64. Dependency-Aware Concurrency

Some services may depend on others.

Example:

```text
database
   |
   v
backend
   |
   v
frontend
```

Do not deploy them blindly in parallel if the architecture requires ordering.

---

# 65. Parallel API Calls

For independent read-only operations:

```text
Get GitHub status
Get ECR status
Get ArgoCD status
Get Prometheus status
```

bounded concurrency can reduce overall latency.

But always respect:

```text
Rate limits
Connection pools
API capacity
```

---

# 66. Timeouts at Multiple Levels

Use:

```text
HTTP timeout
API operation deadline
Workflow timeout
Pipeline timeout
```

Example:

```text
HTTP request = 10 sec
Deployment polling = 10 min
Pipeline stage = 15 min
Pipeline = 30 min
```

Each layer protects the one above it.

---

# 67. Cancellation

If a pipeline is cancelled:

```text
Python should stop polling
```

and where supported:

```text
cancel external operation
```

Do not leave orphaned operations unnecessarily.

---

# 68. Cleanup

After failure:

```text
Temporary files
Temporary branches
Temporary resources
Locks
```

may need cleanup.

Use cleanup logic carefully so that a cleanup failure does not hide the original failure.

---

# 69. Locking

Prevent two automation runs from modifying the same environment simultaneously.

Example:

```text
Production deployment lock
```

Architecture:

```text
Run A -> lock acquired
Run B -> waits/rejected
Run A -> release
Run B -> proceeds
```

The lock mechanism should be external/persistent when multiple runners are involved.

---

# 70. Race Conditions

Example:

```text
Run A reads version 1
Run B reads version 1
Run A updates version 2
Run B updates version 3
```

The second operation may overwrite the first.

Use:

```text
ETags
Locks
Optimistic concurrency
Git conflict handling
Server-side compare-and-swap
```

where supported.

---

# 71. API Automation and GitOps

Avoid:

```text
Python directly modifies production Kubernetes objects
```

if GitOps is the organization's source of truth.

Prefer:

```text
Python
 |
 v
Git
 |
 v
ArgoCD
 |
 v
Kubernetes
```

Python verifies the resulting state.

---

# 72. When Direct Kubernetes API Is Appropriate

Direct API automation can be appropriate for:

```text
Read-only verification
Diagnostics
Temporary operational tooling
Approved control-plane operations
```

Permanent desired state should follow the organization's GitOps model.

---

# 73. API Automation with AWS

For AWS:

```text
Python
 |
 v
boto3
 |
 v
AWS APIs
```

AWS-native SDKs are generally preferable to manually constructing AWS REST requests.

Use:

```text
Requests
```

for HTTP APIs that do not have a suitable SDK/client.

---

# 74. API Automation with Kubernetes

For Kubernetes:

```text
Python
 |
 v
kubernetes Python client
 |
 v
Kubernetes API
```

Use the official client for Kubernetes operations rather than manually constructing raw requests when practical.

---

# 75. API Automation with GitHub

Use:

```text
Requests
```

or an approved GitHub client library.

For straightforward API calls, Requests can be enough.

---

# 76. API Automation with Jenkins

Requests is useful because Jenkins exposes HTTP endpoints.

Typical operations:

```text
Trigger
Queue
Build status
Console output
Artifacts
```

---

# 77. API Automation with ArgoCD

Use an approved ArgoCD API/client mechanism.

Automation may:

```text
Read application
Check sync
Check health
Monitor operation
```

Be careful with version-specific endpoints and authentication.

---

# 78. API Automation with Prometheus

Use Prometheus's HTTP API for:

```text
Deployment verification
Health metrics
SLO checks
Capacity checks
```

Example:

```text
Query:
up{job="payment"}
```

---

# 79. API Automation with Elasticsearch

Use Elasticsearch APIs for:

```text
Log verification
Error searches
Deployment correlation
Incident analysis
```

Example workflow:

```text
Deploy
 |
 v
Search logs for startup errors
 |
 v
No critical errors
 |
 v
Continue
```

---

# 80. Post-Deployment Verification

A robust deployment does:

```text
Deployment
 |
 v
Kubernetes rollout
 |
 v
Pod readiness
 |
 v
Health endpoint
 |
 v
Prometheus metrics
 |
 v
ELK logs
 |
 v
Success
```

This is much stronger than checking only:

```text
kubectl rollout status
```

---

# 81. Verification Window

Do not declare success immediately after rollout.

Example:

```text
Rollout complete
 |
 v
Observe 2 minutes
 |
 v
Check:
error rate
latency
restarts
logs
 |
 v
Declare success
```

The exact window depends on the application.

---

# 82. Canary Verification

For progressive delivery:

```text
Deploy 10%
 |
 v
Check metrics
 |
 +-- bad -> rollback
 |
 +-- good
 |
 v
Deploy 50%
 |
 v
Check
 |
 v
Deploy 100%
```

Python can orchestrate verification if the organization's deployment platform exposes suitable APIs.

---

# 83. Rollback Automation

Rollback should be based on:

```text
Known good revision
```

not:

```text
latest
```

Example:

```text
Current:
sha256:new

Previous known good:
sha256:old
```

Rollback:

```text
Git desired state -> old digest
 |
 v
ArgoCD
 |
 v
EKS
```

---

# 84. Rollback Verification

After rollback:

```text
ArgoCD synced
+
Kubernetes healthy
+
Application healthy
+
Error rate recovered
```

Then report:

```text
Rollback successful
```

---

# 85. Failure Notification

A useful failure notification contains:

```text
Service
Environment
Release ID
Commit
Artifact digest
Failed stage
HTTP status
Request ID
Error summary
Next action
```

Avoid dumping:

```text
Full token-bearing request
```

into Slack/email.

---

# 86. Slack/Notification Integration

Architecture:

```text
Python
 |
 v
Notification API
 |
 v
Slack / Teams / Email
```

The notification should summarize the failure rather than replace logs.

---

# 87. API Automation and CI/CD

Example Jenkins pipeline:

```text
Checkout
 |
 v
Build
 |
 v
Test
 |
 v
Security Scan
 |
 v
Python Release Automation
 |
 +-- ECR
 +-- GitOps
 +-- ArgoCD
 +-- Kubernetes
 +-- Verification
 |
 v
Notify
```

---

# 88. Python CLI

A good automation tool can expose:

```bash
python release.py \
  --service payment \
  --environment staging \
  --version 1.5.0
```

Possible commands:

```bash
python release.py validate
python release.py deploy
python release.py status
python release.py rollback
```

---

# 89. Argument Validation

Use:

```python
argparse
```

or a CLI framework.

Validate:

```text
service
environment
version
release ID
```

before calling external APIs.

---

# 90. Example CLI Design

```text
release.py
 |
 +-- validate
 +-- build
 +-- publish
 +-- deploy
 +-- verify
 +-- rollback
```

Each command can use the same API clients.

---

# 91. Exit Codes

CI/CD systems depend on exit codes.

Typical:

```text
0 = success
non-zero = failure
```

Example:

```python
raise SystemExit(1)
```

for failure.

Use meaningful exit behavior.

---

# 92. Automation Return Contract

A mature tool should return:

```text
Success
Failure
Timeout
Skipped
Cancelled
```

and map those to appropriate pipeline behavior.

---

# 93. API Automation Package Structure

Recommended:

```text
devops_automation/
├── cli.py
├── config.py
├── logging_config.py
├── api/
│   ├── client.py
│   ├── errors.py
│   └── retry.py
├── clients/
│   ├── github.py
│   ├── jenkins.py
│   ├── argocd.py
│   ├── sonarqube.py
│   └── prometheus.py
├── workflows/
│   ├── build.py
│   ├── deploy.py
│   ├── verify.py
│   └── rollback.py
└── tests/
```

---

# 94. Separation of Responsibilities

```text
client.py
=
HTTP transport

github.py
=
GitHub API semantics

deploy.py
=
deployment workflow

verify.py
=
verification logic

cli.py
=
user interface
```

This makes testing easier.

---

# 95. Unit Test Architecture

```text
Workflow
 |
 v
Mock GitHubClient
Mock ArgoCDClient
Mock KubernetesClient
 |
 v
Assert state transitions
```

You do not need a live cluster for every unit test.

---

# 96. Integration Test Architecture

```text
Test Python
 |
 v
Staging APIs
 |
 +-- GitHub test repo
 +-- Jenkins test job
 +-- ArgoCD test app
 +-- EKS staging
```

Use isolated test environments.

---

# 97. Failure Injection

Production-grade automation should be tested against:

```text
Timeout
429
500
503
Network failure
Malformed JSON
Expired token
403
Deployment failure
Pod failure
ArgoCD degraded state
```

This validates recovery logic.

---

# 98. Chaos Testing Example

Simulate:

```text
ArgoCD unavailable
```

Expected behavior:

```text
Retry boundedly
 |
 v
Timeout
 |
 v
Clear failure
 |
 v
No duplicate deployment
```

---

# 99. API Contract Changes

An API provider changes:

```json
{
  "status": "healthy"
}
```

to:

```json
{
  "state": "healthy"
}
```

Without validation, automation may silently break.

Use:

```text
Schema validation
Contract tests
Versioned APIs
```

---

# 100. Defensive Parsing

Avoid blindly:

```python
status = data["status"]
```

unless the schema is guaranteed.

For expected optional fields:

```python
status = data.get("status")
```

Then validate explicitly.

---

# 101. Do Not Hide API Errors

Bad:

```python
except Exception:
    return False
```

This loses:

```text
Root cause
Status
Request ID
```

Better:

```text
Raise structured error
```

and let the workflow decide.

---

# 102. Structured API Error

Example:

```python
class APIError(Exception):

    def __init__(
        self,
        message,
        status_code=None,
        request_id=None,
        response_body=None
    ):
        super().__init__(message)

        self.status_code = status_code
        self.request_id = request_id
        self.response_body = response_body
```

Sanitize `response_body` before logging.

---

# 103. Error Classification

Example:

```text
AUTHENTICATION_ERROR
AUTHORIZATION_ERROR
VALIDATION_ERROR
NOT_FOUND
CONFLICT
RATE_LIMIT
TRANSIENT_SERVER_ERROR
TIMEOUT
NETWORK_ERROR
UNKNOWN_API_ERROR
```

This allows consistent pipeline behavior.

---

# 104. Retry Matrix

| Error | Retry? | Action |
|---|---:|---|
| 400 | No | Fix input |
| 401 | No | Refresh/fix auth |
| 403 | No | Fix permission |
| 404 | Usually no | Validate resource |
| 409 | Reconcile | Check state |
| 422 | No | Fix validation |
| 429 | Yes | Backoff |
| 500 | Sometimes | Bounded retry |
| 502 | Often | Backoff |
| 503 | Often | Backoff |
| 504 | Often | Reconcile if state-changing |
| Timeout | Depends | Reconcile if state-changing |
| ConnectionError | Often | Bounded retry |

---

# 105. The Critical Rule for Timeouts

A timeout does not always mean:

```text
Operation failed
```

It means:

```text
Client did not receive a response within the configured time.
```

The server may have:

```text
received
processed
completed
```

the request.

Always consider this for state-changing operations.

---

# 106. API Automation and Exactly-Once

Exactly-once execution is difficult across distributed systems.

Instead, design for:

```text
At-least-once delivery
+
Idempotent operations
+
State reconciliation
```

This is a major distributed-systems principle.

---

# 107. At-Least-Once Example

```text
POST deployment
 |
 v
Server succeeds
 |
 X
Response lost
 |
 v
Client retries
```

Now two requests may exist.

Use:

```text
Idempotency key
```

or:

```text
Check current state
```

to make the workflow safe.

---

# 108. Distributed Transaction Problem

A release may span:

```text
GitHub
Jenkins
ECR
Git
ArgoCD
Kubernetes
```

There is no simple single transaction covering all systems.

Therefore:

```text
State machine
+
Compensation
+
Reconciliation
```

are more realistic than trying to create one giant transaction.

---

# 109. Compensation

Example:

```text
Deployment started
 |
 v
Verification failed
```

Compensation may be:

```text
Rollback Git revision
```

rather than:

```text
Undo every API call
```

The correct compensation depends on the system.

---

# 110. Release Workflow Example

```text
START
 |
 v
Validate input
 |
 v
Check dependencies
 |
 v
Build
 |
 v
Test
 |
 v
Security gate
 |
 v
Publish immutable artifact
 |
 v
Update GitOps
 |
 v
Wait for ArgoCD
 |
 v
Verify Kubernetes
 |
 v
Verify application
 |
 v
Observe metrics/logs
 |
 v
SUCCESS
```

---

# 111. Production Failure Example

Suppose:

```text
ECR push = success
Git commit = success
ArgoCD sync = success
Kubernetes rollout = success
Health endpoint = 200
Prometheus error rate = high
```

Should automation report success?

No.

The deployment is technically rolled out, but application verification failed.

The release should be:

```text
FAILED
```

or:

```text
DEGRADED
```

according to the organization's release policy.

---

# 112. Multi-Signal Verification

Strong verification combines:

```text
Control-plane state
+
Data-plane state
+
Application state
+
Observability state
```

Example:

```text
ArgoCD = Synced
Kubernetes = Ready
HTTP = Healthy
Prometheus = Normal
ELK = No critical errors
```

This is much stronger than a single signal.

---

# 113. API Automation Troubleshooting

When the automation fails:

```text
1. Identify failed stage
2. Identify API
3. Capture status code
4. Capture request ID
5. Inspect sanitized response
6. Check network/TLS
7. Check authentication
8. Check authorization
9. Check dependency state
10. Check whether operation may have succeeded
11. Reconcile
12. Retry/rollback if appropriate
```

---

# 114. Troubleshooting by Layer

```text
Python
 |
 +-- Configuration
 |
 +-- Authentication
 |
 +-- HTTP
 |
 +-- Network
 |
 +-- API Gateway/ALB
 |
 +-- Service
 |
 +-- Dependency
 |
 +-- Kubernetes
 |
 +-- Application
```

Start at the layer where the failure is observed.

---

# 115. Production Example — API 503

```text
Python
 |
 v
ArgoCD API
 |
 503
```

Investigate:

```text
ArgoCD server
Ingress
Load balancer
Redis
Kubernetes
Network
```

Do not automatically blame Python.

---

# 116. Production Example — API 403

```text
Python
 |
 v
Kubernetes API
 |
 403
```

Investigate:

```text
Service account
IAM/RBAC
Role
RoleBinding
Namespace
API permissions
```

This is usually an authorization problem.

---

# 117. Production Example — DNS Failure

```text
requests.get(
    "https://argocd.example.com"
)
```

fails before HTTP.

Investigate:

```text
DNS resolution
Resolver
Route
Network
```

Useful commands:

```bash
nslookup argocd.example.com
dig argocd.example.com
```

where available.

---

# 118. Production Example — TLS Failure

Investigate:

```text
Certificate expiration
Hostname
CA
Proxy
Trust store
```

Do not immediately disable verification.

---

# 119. Production Example — Slow API

Measure:

```text
DNS
Connect
TLS
Server processing
Read
```

Then compare:

```text
p50
p95
p99
```

Averages can hide tail latency.

---

# 120. API Automation Performance

Optimize in this order:

```text
1. Correctness
2. Reliability
3. Security
4. Observability
5. Performance
```

Do not sacrifice reliability just to save a few seconds.

---

# 121. API Call Reduction

Reduce calls using:

```text
Filtering
Pagination
Caching
Batch APIs
Bulk endpoints
Webhooks
State tracking
```

---

# 122. Batch APIs

If the provider supports:

```text
POST /bulk
```

it may be better than:

```text
POST /item/1
POST /item/2
POST /item/3
...
```

But batch operations may have different failure semantics.

Understand partial failure behavior.

---

# 123. Partial Failure

Suppose:

```text
10 resources
```

are updated in one batch.

Result:

```text
8 success
2 failure
```

The automation must understand:

```text
Which succeeded?
Which failed?
Can only failures be retried?
```

---

# 124. API Automation and Eventual Consistency

Distributed systems may not immediately reflect changes.

Example:

```text
POST resource
 |
 v
201 Created
 |
 v
GET immediately
 |
 v
404
```

The backend may be eventually consistent.

Use bounded polling when the API contract requires it.

---

# 125. Do Not Sleep Blindly

Bad:

```python
time.sleep(60)
```

then assume success.

Better:

```text
Poll
 |
 +-- ready -> continue
 |
 +-- not ready -> wait
 |
 +-- failed -> stop
 |
 +-- deadline -> timeout
```

---

# 126. Polling Interval

Choose based on:

```text
Operation duration
API rate limit
Required responsiveness
System load
```

For deployment verification:

```text
5–15 seconds
```

may be reasonable, but use the actual platform characteristics.

---

# 127. Polling Jitter

For many concurrent deployments, add small randomization:

```text
Service A -> 8.2s
Service B -> 10.1s
Service C -> 11.4s
```

This prevents synchronized polling spikes.

---

# 128. Automation Deadlines

Every long-running workflow should have a deadline:

```python
deadline = (
    time.monotonic()
    + timeout_seconds
)
```

Pass remaining time to operations where practical.

---

# 129. Deadline Propagation

Example:

```text
Pipeline deadline
      |
      v
Release deadline
      |
      v
Deployment deadline
      |
      v
HTTP timeout
```

Each layer should respect the remaining budget.

---

# 130. Production API Automation Principles

```text
Use APIs instead of screen automation
Use official SDKs when appropriate
Use Requests for generic HTTP APIs
Keep credentials external
Use least privilege
Use timeouts
Retry only safe transient failures
Use backoff and jitter
Respect rate limits
Reconcile after uncertain writes
Verify final state
Log safely
Measure everything
```

---

# 131. Interview Questions

## Q1. What is API automation?

Using code to interact with APIs to perform, verify, and orchestrate operations without manual intervention.

---

## Q2. Automation vs orchestration?

Automation handles individual tasks.

Orchestration coordinates multiple dependent tasks into a workflow.

---

## Q3. How would you automate a deployment using APIs?

I would:

```text
Validate inputs
Authenticate
Trigger build
Monitor build
Verify artifact
Update GitOps
Monitor ArgoCD
Verify Kubernetes
Check application health
Check metrics/logs
Report result
```

---

## Q4. Why is state reconciliation important?

Because network failures can make the client uncertain about whether a state-changing operation succeeded.

Reconciliation determines actual external state before retrying.

---

## Q5. How do you design idempotent automation?

Prefer:

```text
Ensure desired state
Check before create
Use idempotency keys
Use immutable identifiers
Reconcile after failures
```

---

## Q6. How would you handle an asynchronous API?

```text
Submit
 |
 v
Receive operation ID
 |
 v
Poll
 |
 v
Success/Failure
```

Use:

```text
Backoff
Deadline
State validation
```

---

## Q7. How do you automate a Jenkins build?

```text
POST trigger
 |
 v
Queue ID
 |
 v
Build number
 |
 v
Poll build
 |
 v
Check result
```

---

## Q8. How do you verify a Kubernetes deployment?

Check multiple layers:

```text
Deployment replicas
Pod readiness
Rollout
ArgoCD state
Application health
Metrics/logs
```

---

## Q9. Why should Python not directly become the Kubernetes source of truth in GitOps?

Because Git is the declarative source of truth.

Python should coordinate and verify the GitOps workflow unless direct cluster changes are explicitly required.

---

## Q10. How do you handle a timeout after POST?

I do not assume failure.

I reconcile the external state using:

```text
Operation ID
Release ID
Resource lookup
Logs
```

Then determine whether retry is safe.

---

# 132. Scenario Questions

## Scenario 1

A deployment script reports failure because the POST request timed out. Later you discover the deployment succeeded.

### What went wrong?

The automation treated:

```text
No response
```

as:

```text
Operation failed
```

Correct design:

```text
Timeout
 |
 v
Unknown state
 |
 v
Reconcile
 |
 +-- success
 +-- failed
 +-- still running
```

---

## Scenario 2

A script triggers 100 Jenkins jobs simultaneously and Jenkins starts returning 429.

### Solution

Use:

```text
Bounded concurrency
Rate limiting
Backoff
Retry-After
Queue awareness
```

---

## Scenario 3

ArgoCD reports Synced but users receive 500 errors.

### Solution

Synced means:

```text
Desired state matches Git
```

It does not guarantee application correctness.

Check:

```text
Kubernetes
Pods
Health endpoint
Prometheus
ELK
```

---

## Scenario 4

ECR contains the expected image tag but the deployed application runs a different image.

### Investigation

Check:

```text
Image tag mutability
Image digest
Git manifest
ArgoCD revision
Deployment image
Running container image
```

Prefer digest-based verification.

---

## Scenario 5

Two deployment pipelines update the same GitOps manifest.

### Risk

Race condition.

### Solutions

```text
Concurrency lock
Optimistic concurrency
Git conflict handling
Serialized production releases
```

---

# 133. Senior-Level Interview Question

## Design a production Python release orchestrator.

### Answer Structure

I would use:

```text
CLI
 |
 v
Configuration
 |
 v
Workflow Engine
 |
 +-- GitHub Client
 +-- Jenkins Client
 +-- ECR/AWS Client
 +-- Git Client
 +-- ArgoCD Client
 +-- Kubernetes Client
 +-- Prometheus Client
 +-- Elasticsearch Client
```

The workflow would use a state machine:

```text
VALIDATE
BUILD
SCAN
PUBLISH
UPDATE_GITOPS
SYNC
ROLLOUT
VERIFY
OBSERVE
SUCCESS
```

Each state would have:

```text
Timeout
Retry policy
Error classification
Logging
Metrics
Idempotency strategy
```

State would be persisted or recoverable using a release ID.

---

# 134. Senior-Level Design Principle

The most important production concept is:

> **Never design automation around the assumption that the network is reliable.**

Instead assume:

```text
Requests can timeout
Responses can be lost
APIs can rate-limit
Servers can restart
Tokens can expire
Dependencies can fail
Processes can crash
```

Then design:

```text
Retries
Reconciliation
Idempotency
Deadlines
Observability
```

---

# 135. Production Checklist

```text
Architecture
[ ] Service-specific clients
[ ] Generic API transport
[ ] State machine
[ ] Persistent/recoverable release ID

Security
[ ] Least privilege
[ ] Secrets externalized
[ ] TLS verified
[ ] Input validated
[ ] Production approval

Reliability
[ ] Timeouts
[ ] Retry matrix
[ ] Backoff
[ ] Jitter
[ ] Rate-limit handling
[ ] Idempotency
[ ] Reconciliation
[ ] Bounded polling

Observability
[ ] Structured logs
[ ] Request IDs
[ ] Metrics
[ ] Release IDs
[ ] Failure classification

Deployment
[ ] Artifact digest verified
[ ] GitOps revision verified
[ ] ArgoCD sync verified
[ ] Kubernetes rollout verified
[ ] Application health verified
[ ] Prometheus verified
[ ] Logs verified

Testing
[ ] Unit tests
[ ] Integration tests
[ ] Failure injection
[ ] API contract tests
```

---

# 136. Final Production Architecture

```text
                           Developer
                               |
                               v
                           GitHub
                               |
                               v
                     Jenkins / GitHub Actions
                               |
                               v
                      Python Orchestrator
                               |
        +----------------------+----------------------+
        |                      |                      |
        v                      v                      v
   Configuration          Secret Provider        Release ID
        |                      |                      |
        +----------------------+----------------------+
                               |
                               v
                        Workflow State
                               |
              +----------------+----------------+
              |                |                |
              v                v                v
        GitHub Client     Jenkins Client     AWS/ECR
              |                |                |
              +----------------+----------------+
                               |
                               v
                           GitOps
                               |
                               v
                             ArgoCD
                               |
                               v
                              EKS
                               |
                  +------------+------------+
                  |                         |
                  v                         v
             Kubernetes API            Application
                  |                         |
                  +------------+------------+
                               |
                    +----------+----------+
                    |                     |
                    v                     v
                Prometheus               ELK
                    |                     |
                    +----------+----------+
                               |
                               v
                       Release Decision
                               |
                 +-------------+-------------+
                 |                           |
                 v                           v
              SUCCESS                     ROLLBACK
```

---

# 137. Final Mental Model

Think about API automation as:

```text
CALL
 |
 v
VALIDATE
 |
 v
WAIT
 |
 v
RECONCILE
 |
 v
VERIFY
 |
 v
OBSERVE
 |
 v
DECIDE
```

Not:

```text
CALL API
```

A production DevOps automation tool must understand the **state of the external system**, not just whether an HTTP request returned a response.

---

# 138. What You Should Know Before Moving On

You should now be able to explain:

```text
What is API automation?
Automation vs orchestration
Generic API client
Service-specific clients
Configuration
Secret management
Least privilege
GitHub automation
Jenkins automation
ECR verification
GitOps automation
ArgoCD automation
Kubernetes verification
Prometheus verification
ELK verification
State machines
Idempotency
Reconciliation
Polling
Backoff
Rate limits
Bounded concurrency
Race conditions
Locks
Dry runs
Approvals
Audit trails
Release IDs
Correlation IDs
Rollback
Post-deployment verification
Failure classification
Testing
Production architecture
```

---

# 139. Connection to the Next Topic

Current progress:

```text
08-Python-APIs/
├── 01-HTTP-and-REST.md       ✓
├── 02-Requests-Library.md    ✓
├── 03-API-Automation.md      ✓
├── 04-Authentication.md
├── 05-API-Error-Handling.md
└── 06-DevOps-API-Projects.md
```

The next topic is:

# `04-Authentication.md`

That file will go deeper into:

```text
API keys
Basic authentication
Bearer tokens
JWT
OAuth 2.0
OIDC
Access vs refresh tokens
Token expiration
Token refresh
AWS authentication concepts
IAM and SigV4
Kubernetes authentication
Service accounts
ArgoCD authentication
GitHub tokens
Jenkins authentication
Secret management
Credential rotation
Least privilege
Production authentication architecture
Authentication troubleshooting
Security interview questions
Scenario-based interview questions
```

The progression is:

```text
HTTP/REST
    ↓
Requests
    ↓
API Automation
    ↓
Authentication
    ↓
Error Handling
    ↓
Real DevOps API Projects
```

> **API automation becomes production-grade when it can safely call, wait, reconcile, verify, and recover across multiple systems.**
