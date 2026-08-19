# Production Scripting

## 1. Introduction

A Python script used in a production DevOps environment is very different from a small local utility.

A local script may be:

```text
read input
 ↓
do work
 ↓
print result
```

A production automation script must additionally handle:

```text
validation
authentication
configuration
timeouts
retries
logging
security
idempotency
cleanup
signals
exit codes
observability
concurrency
failure recovery
```

A useful production model is:

```text
INPUT
  ↓
VALIDATE
  ↓
PRE-FLIGHT
  ↓
PLAN
  ↓
APPROVE
  ↓
EXECUTE
  ↓
VERIFY
  ↓
CLEANUP
  ↓
REPORT
```

The goal is not simply:

```text
"Make the script work."
```

The goal is:

```text
"Make the automation safe, repeatable, observable, testable,
recoverable, and suitable for production."
```

---

# 2. Production Script vs Basic Script

## Basic script

```python
print("Deploying...")
deploy()
print("Done")
```

Problems:

```text
no validation
no timeout
no error classification
no structured logging
no cleanup
no exit code policy
no environment protection
```

---

## Production script

```text
parse arguments
 ↓
load configuration
 ↓
validate configuration
 ↓
validate environment
 ↓
validate credentials
 ↓
run pre-flight checks
 ↓
execute operation
 ↓
verify result
 ↓
cleanup
 ↓
return correct exit code
```

---

# 3. Production Scripting Principles

A production DevOps script should be:

```text
1. Explicit
2. Idempotent
3. Observable
4. Secure
5. Testable
6. Configurable
7. Recoverable
8. Bounded
9. Environment-aware
10. Easy to troubleshoot
```

---

# 4. Production Automation Architecture

A useful architecture is:

```text
                CLI
                 |
                 v
          Configuration
                 |
                 v
            Validation
                 |
                 v
          Pre-flight Checks
                 |
                 v
             Workflow
          /      |      \
         /       |       \
       AWS      K8s     Git/CI
         \       |       /
          \      |      /
             Verification
                 |
                 v
              Cleanup
                 |
                 v
             Exit Code
```

---

# 5. Recommended Python Repository Structure

A production automation repository can look like:

```text
devops-automation/
├── src/
│   └── automation/
│       ├── __init__.py
│       ├── cli.py
│       ├── config.py
│       ├── logging_config.py
│       ├── validators.py
│       ├── exceptions.py
│       ├── workflow.py
│       ├── aws/
│       │   ├── __init__.py
│       │   └── client.py
│       ├── kubernetes/
│       │   ├── __init__.py
│       │   └── client.py
│       ├── git/
│       │   ├── __init__.py
│       │   └── client.py
│       └── utils/
│           ├── __init__.py
│           └── subprocess.py
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
├── scripts/
├── configs/
├── pyproject.toml
├── README.md
├── Dockerfile
└── .gitignore
```

---

# 6. Why Structure Matters

A single:

```text
deploy.py
```

containing:

```text
AWS
Kubernetes
Git
Docker
Terraform
logging
configuration
```

quickly becomes difficult to maintain.

Instead separate:

```text
CLI
configuration
validation
clients
workflow
policies
```

This makes testing and troubleshooting easier.

---

# 7. Entry Point

A production application should have a clear entry point.

Example:

```python
def main():
    config = load_config()
    validate_config(config)
    return run(config)


if __name__ == "__main__":
    raise SystemExit(main())
```

The important pattern is:

```text
main()
 ↓
return exit status
 ↓
SystemExit
```

---

# 8. Why `SystemExit` Matters

CI/CD systems rely on exit codes.

Typically:

```text
0 -> success
non-zero -> failure
```

A script that prints:

```text
Deployment failed
```

but exits with:

```text
0
```

can cause CI to incorrectly mark the build successful.

---

# 9. Exit Codes

A useful policy:

```text
0  SUCCESS
1  GENERAL_FAILURE
2  INVALID_INPUT
3  AUTHENTICATION_FAILURE
4  ENVIRONMENT_FAILURE
5  DEPENDENCY_FAILURE
6  TIMEOUT
7  SECURITY_FAILURE
8  VERIFICATION_FAILURE
```

The exact values are project-specific.

The important point is consistency.

---

# 10. Exit Code Example

```python
SUCCESS = 0
GENERAL_FAILURE = 1
INVALID_INPUT = 2
TIMEOUT = 6


def main():
    try:
        run()
        return SUCCESS
    except TimeoutError:
        return TIMEOUT
    except ValueError:
        return INVALID_INPUT
    except Exception:
        return GENERAL_FAILURE


if __name__ == "__main__":
    raise SystemExit(main())
```

---

# 11. Do Not Swallow Exceptions

Bad:

```python
try:
    deploy()
except Exception:
    print("Something went wrong")
```

This hides the root cause.

Better:

```python
try:
    deploy()
except Exception:
    logger.exception("Deployment failed")
    raise
```

At the application boundary, convert the exception into the appropriate exit code.

---

# 12. CLI Design

DevOps scripts frequently run from:

```text
developer terminal
Jenkins
GitHub Actions
cron
Kubernetes Job
AWS automation
```

A CLI should clearly expose:

```text
command
arguments
options
help
defaults
```

---

# 13. `argparse`

Python's standard library provides:

```python
import argparse
```

Example:

```python
parser = argparse.ArgumentParser(
    description="Deploy application"
)

parser.add_argument(
    "--environment",
    required=True
)

args = parser.parse_args()
```

Run:

```bash
python deploy.py --environment staging
```

---

# 14. CLI Help

A production CLI should provide useful help:

```bash
python deploy.py --help
```

It should explain:

```text
purpose
required options
optional options
examples
```

---

# 15. Subcommands

For larger automation tools:

```text
devops-tool
├── deploy
├── validate
├── rollback
├── diagnose
├── cleanup
└── status
```

Example:

```bash
python -m automation deploy --environment staging
```

---

# 16. CLI Design Principle

Prefer:

```bash
deploy --environment staging
```

over:

```bash
deploy staging true false abc
```

Named options are more readable and less error-prone.

---

# 17. Required CLI Arguments

Example:

```python
parser.add_argument(
    "--environment",
    choices=["dev", "staging", "production"],
    required=True
)
```

This prevents unsupported values.

---

# 18. CLI Defaults

Use safe defaults.

For example:

```text
dry-run -> true
```

may be appropriate for dangerous operations.

Do not default destructive commands to production.

---

# 19. Dry-Run Mode

A production automation tool should often support:

```bash
--dry-run
```

Flow:

```text
validate
 ↓
show planned changes
 ↓
do not mutate
```

---

# 20. Dry-Run Example

```python
def deploy(config, dry_run=False):
    if dry_run:
        logger.info("DRY RUN: deployment skipped")
        return

    perform_deployment(config)
```

---

# 21. Dry-Run Is Not a Substitute for Validation

Even in dry-run mode, perform:

```text
configuration validation
environment validation
authentication checks where safe
resource checks
```

The purpose is to show whether execution would be valid.

---

# 22. Plan Before Apply

Production automation should often follow:

```text
PLAN
 ↓
REVIEW
 ↓
APPROVE
 ↓
APPLY
```

This is the same principle used by:

```text
Terraform plan/apply
```

and can be applied to Python automation.

---

# 23. Environment Selection

Never rely on accidental environment selection.

Bad:

```python
environment = os.getenv("ENV")
```

with no validation.

Better:

```python
environment = os.getenv("ENV")

if environment not in {
    "dev",
    "staging",
    "production"
}:
    raise ValueError("Invalid environment")
```

---

# 24. Production Environment Guard

For destructive automation:

```python
if environment == "production":
    require_approval()
```

The exact approval mechanism depends on the CI/CD system.

---

# 25. AWS Account Guard

Before changing AWS resources:

```text
expected account
actual account
```

must match.

Concept:

```python
if actual_account != expected_account:
    raise RuntimeError(
        "AWS account mismatch"
    )
```

---

# 26. AWS Region Guard

Validate:

```text
expected region
actual region
```

Example:

```python
if region != "ap-south-1":
    raise RuntimeError("Unexpected region")
```

Do not let an incorrect profile silently operate in another region.

---

# 27. EKS Cluster Guard

Before running Kubernetes mutations:

```text
expected cluster
actual cluster
```

must match.

Example:

```text
expected: dev-eks
actual: production-eks
```

Result:

```text
BLOCK
```

---

# 28. Kubernetes Namespace Guard

Validate:

```text
namespace
environment
cluster
```

before:

```text
apply
delete
scale
restart
```

---

# 29. Git Branch Guard

Before modifying GitOps configuration:

```text
repository
branch
path
```

must be validated.

Example:

```text
expected branch: deployment-config
actual branch: main
```

The automation should stop if the operation is restricted to a specific branch.

---

# 30. Configuration Sources

Production scripts commonly receive configuration from:

```text
CLI arguments
environment variables
configuration files
secret managers
CI/CD variables
```

A common precedence model is:

```text
CLI
 ↓
environment
 ↓
config file
 ↓
defaults
```

Document the precedence.

---

# 31. Configuration Validation

Validate:

```text
required fields
type
format
range
allowed values
environment
```

Example:

```python
if replicas < 1:
    raise ValueError(
        "replicas must be >= 1"
    )
```

---

# 32. Never Trust Configuration

Even if configuration comes from:

```text
CI
Terraform
Git
```

validate it at the execution boundary.

Configuration mistakes can become production incidents.

---

# 33. Environment Variables

Example:

```python
import os

region = os.environ["AWS_REGION"]
```

For optional values:

```python
environment = os.getenv(
    "ENVIRONMENT",
    "dev"
)
```

But be careful with dangerous defaults.

---

# 34. Dangerous Defaults

Bad:

```python
environment = os.getenv(
    "ENVIRONMENT",
    "production"
)
```

If the variable is missing, the script can accidentally operate on production.

Prefer:

```python
environment = os.getenv("ENVIRONMENT")

if not environment:
    raise ValueError(
        "ENVIRONMENT is required"
    )
```

---

# 35. Secret Management

Never hard-code:

```text
AWS keys
passwords
tokens
Kubeconfig credentials
Git tokens
```

Use:

```text
IAM roles
OIDC
secret managers
CI/CD secret stores
short-lived credentials
```

where supported.

---

# 36. AWS Authentication

Prefer:

```text
IAM role
```

over:

```text
long-lived access key
```

for workloads running in AWS.

---

# 37. CI Authentication

For GitHub Actions and other CI systems, prefer:

```text
OIDC
+
short-lived cloud credentials
```

when supported by the environment.

---

# 38. Kubernetes Authentication

Prefer controlled identity mechanisms such as:

```text
cloud IAM integration
service accounts
RBAC
short-lived credentials
```

instead of sharing a long-lived admin kubeconfig.

---

# 39. Credential Validation

Before performing work, verify that credentials are usable.

For AWS, a safe identity call can confirm:

```text
account
principal
```

before mutation.

---

# 40. Credential Identity Logging

Log:

```text
account
role/principal
region
environment
```

but never:

```text
secret
token
private key
password
```

---

# 41. Pre-Flight Checks

Pre-flight checks happen before mutation.

Typical checks:

```text
configuration
credentials
AWS account
AWS region
EKS cluster
namespace
Git branch
required tools
network
dependencies
```

---

# 42. Pre-Flight Architecture

```text
CLI
 ↓
Config
 ↓
AWS identity
 ↓
Cluster identity
 ↓
Git state
 ↓
Dependency checks
 ↓
Ready to execute
```

If any critical check fails:

```text
STOP
```

---

# 43. Required CLI Tool Validation

If the script invokes:

```text
kubectl
helm
terraform
docker
aws
git
```

verify the required binary exists before starting.

---

# 44. `shutil.which`

Python provides:

```python
from shutil import which

if which("kubectl") is None:
    raise RuntimeError(
        "kubectl is required"
    )
```

---

# 45. Tool Version Validation

For production automation, version compatibility matters.

Check:

```bash
kubectl version
helm version
terraform version
aws --version
```

where appropriate.

---

# 46. Version Policy

Avoid silently supporting unknown versions.

For critical automation:

```text
supported range
```

should be documented and tested.

---

# 47. Subprocess Execution

Python often integrates with DevOps CLIs using:

```python
subprocess.run()
```

Example:

```python
import subprocess

result = subprocess.run(
    ["kubectl", "get", "pods"],
    check=True,
    capture_output=True,
    text=True,
)
```

---

# 48. Prefer Argument Lists

Prefer:

```python
subprocess.run(
    ["kubectl", "get", "pods"]
)
```

over:

```python
subprocess.run(
    "kubectl get pods",
    shell=True
)
```

The list form avoids many shell parsing and injection problems.

---

# 49. `shell=True`

Use:

```python
shell=True
```

only when there is a strong reason and inputs are tightly controlled.

For automation involving external input, avoid it.

---

# 50. Command Injection

Bad:

```python
command = f"kubectl get {resource}"
subprocess.run(
    command,
    shell=True
)
```

An attacker-controlled `resource` could alter the command.

Better:

```python
subprocess.run(
    ["kubectl", "get", resource],
    check=True
)
```

and validate `resource`.

---

# 51. `check=True`

Use:

```python
check=True
```

when a non-zero exit code should fail the operation.

Otherwise a failed command can be ignored accidentally.

---

# 52. Capturing Output

Useful:

```python
capture_output=True
text=True
```

Then:

```python
result.stdout
result.stderr
```

can be processed.

Be careful not to log secrets from command output.

---

# 53. Subprocess Timeout

Always consider a bounded timeout:

```python
subprocess.run(
    ["kubectl", "get", "pods"],
    check=True,
    timeout=30
)
```

A hung command should not hang a CI job forever.

---

# 54. Subprocess Error Handling

```python
try:
    subprocess.run(
        command,
        check=True,
        timeout=60
    )
except subprocess.TimeoutExpired:
    logger.error("Command timed out")
    raise
except subprocess.CalledProcessError as exc:
    logger.error(
        "Command failed with %s",
        exc.returncode
    )
    raise
```

---

# 55. Logging

Production scripts need logs that answer:

```text
What happened?
When?
Where?
Which environment?
Which resource?
What failed?
What was the result?
```

---

# 56. `print()` vs Logging

For simple scripts:

```python
print("Deploying")
```

may be enough.

For production automation:

```python
import logging

logger = logging.getLogger(__name__)

logger.info("Deployment started")
```

Use the logging framework.

---

# 57. Log Levels

Typical levels:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

Use:

```text
DEBUG -> detailed troubleshooting
INFO -> normal workflow
WARNING -> unusual but recoverable
ERROR -> operation failed
CRITICAL -> severe failure
```

---

# 58. Logging Example

```python
logger.info(
    "Deployment started environment=%s service=%s",
    environment,
    service,
)
```

Structured fields are useful for CI and log aggregation.

---

# 59. Avoid Secret Logging

Never:

```python
logger.info("token=%s", token)
```

Instead:

```python
logger.info("Authentication token loaded")
```

---

# 60. Structured Logging

A useful production event contains:

```text
timestamp
level
event
environment
resource
run_id
result
```

Example conceptual output:

```text
2026-08-18T10:20:31Z
INFO
event=deployment_started
environment=staging
service=orders
run_id=7812
```

---

# 61. Run ID

Every automation run should ideally have a unique identifier:

```text
run_id
```

It can come from:

```text
CI build ID
GitHub run ID
UUID
```

Use it for correlation.

---

# 62. Correlation

Pass the same run ID into:

```text
logs
AWS tags
Kubernetes labels
Git commits where appropriate
test reports
```

This makes troubleshooting much easier.

---

# 63. Error Messages

Bad:

```text
Deployment failed.
```

Better:

```text
Deployment failed:
service=orders
environment=staging
reason=rollout_timeout
```

Never expose secrets.

---

# 64. Exception Hierarchy

Define application-specific exceptions:

```python
class AutomationError(Exception):
    pass


class ConfigurationError(AutomationError):
    pass


class AuthenticationError(AutomationError):
    pass


class VerificationError(AutomationError):
    pass


class TimeoutError(AutomationError):
    pass
```

This makes classification easier.

---

# 65. Error Classification

Typical categories:

```text
configuration
authentication
authorization
dependency
timeout
validation
verification
unexpected
```

Do not treat all failures as identical.

---

# 66. Retryable vs Non-Retryable

Retryable examples:

```text
timeout
HTTP 503
HTTP 429
temporary connection failure
```

Usually non-retryable:

```text
invalid configuration
HTTP 400
HTTP 401
HTTP 403
wrong cluster
wrong AWS account
```

The exact policy depends on the operation.

---

# 67. Retry Architecture

```text
operation
 ↓
failure
 ↓
classify
 ↓
retryable?
 ├── no -> fail
 └── yes
       ↓
     backoff
       ↓
      retry
```

---

# 68. Maximum Retries

Never retry forever.

Use:

```text
max_attempts
```

Example:

```python
MAX_ATTEMPTS = 3
```

---

# 69. Exponential Backoff

Concept:

```text
attempt 1 -> 1s
attempt 2 -> 2s
attempt 3 -> 4s
attempt 4 -> 8s
```

Add jitter when many workers could retry simultaneously.

---

# 70. Jitter

Without jitter:

```text
many workers
 ↓
same retry time
 ↓
thundering herd
```

Jitter spreads retries.

---

# 71. Retry Safety

Before retrying a mutation, ask:

```text
Is this operation idempotent?
```

A retry can otherwise create duplicate resources or repeated side effects.

---

# 72. Idempotent Automation

Prefer:

```text
ensure resource exists
```

rather than:

```text
create resource every time
```

---

# 73. Idempotency Example

```python
def ensure_namespace(client, name):
    if client.exists(name):
        return "already_exists"

    client.create(name)
    return "created"
```

Running twice should produce the same final state.

---

# 74. Idempotency in AWS

Examples:

```text
ensure S3 bucket configuration
ensure IAM policy
ensure tags
ensure security group rule
```

Use current-state checks and stable identifiers.

---

# 75. Idempotency in Kubernetes

Use desired-state APIs or declarative resources where possible.

Example:

```text
apply deployment
```

instead of repeatedly trying to create duplicate resources.

---

# 76. Idempotency in GitOps

When updating a GitOps manifest:

```text
if desired tag already exists:
    no change
```

Avoid creating unnecessary commits.

---

# 77. Dry Run + Idempotency

These complement each other:

```text
dry-run -> preview
idempotency -> safe repeat
```

---

# 78. Verification

Never assume an operation succeeded because the API returned without an exception.

Verify the resulting state.

Example:

```text
apply deployment
 ↓
wait rollout
 ↓
verify ready replicas
```

---

# 79. Verify Desired vs Actual

Production automation should compare:

```text
desired state
```

with:

```text
actual state
```

---

# 80. AWS Verification

After creating/updating a resource:

```text
query resource
 ↓
verify configuration
```

---

# 81. Kubernetes Verification

After deployment:

```text
deployment exists
pods ready
service endpoints
ingress
application health
```

---

# 82. Terraform Verification

After apply:

```text
outputs
resource state
connectivity
security properties
```

should be verified where appropriate.

---

# 83. Git Verification

After automation modifies Git:

```text
branch
commit
changed files
remote
```

should be checked.

---

# 84. GitOps Verification

After changing Git:

```text
Git commit
 ↓
ArgoCD sync
 ↓
health
 ↓
Kubernetes rollout
```

Do not stop at:

```text
git push succeeded
```

---

# 85. Cleanup

Production automation must clean up:

```text
temporary files
temporary AWS resources
temporary Kubernetes namespaces
test artifacts
temporary credentials
```

---

# 86. `try/finally`

Use:

```python
resource = create_resource()

try:
    run_test(resource)
finally:
    delete_resource(resource)
```

The cleanup runs even when the test fails.

---

# 87. Cleanup Failure

Cleanup itself can fail.

Do not hide it.

Log:

```text
resource
error
run ID
```

and send the event to the appropriate cleanup mechanism.

---

# 88. Cleanup Safety

Cleanup automation must have its own guards.

Bad:

```python
delete_everything()
```

Better:

```text
delete only resources with:
Environment=Test
RunID=<current>
ManagedBy=Automation
```

---

# 89. Resource Tags

For AWS resources use tags such as:

```text
Environment=Test
ManagedBy=DevOpsAutomation
RunID=7812
Owner=Platform
```

This makes cleanup and cost tracking easier.

---

# 90. Kubernetes Labels

Use:

```yaml
metadata:
  labels:
    managed-by: devops-automation
    run-id: "7812"
```

Then cleanup can target the correct resources.

---

# 91. Temporary Directories

Use Python's:

```python
from tempfile import TemporaryDirectory
```

Example:

```python
with TemporaryDirectory() as path:
    run_workflow(path)
```

The temporary directory is cleaned automatically.

---

# 92. File Permissions

Sensitive temporary files should have restrictive permissions.

Never assume the default filesystem permissions are appropriate for secrets.

Prefer secret manager integration where possible.

---

# 93. Signal Handling

CI jobs and containers can receive:

```text
SIGTERM
SIGINT
```

A production process should shut down cleanly.

---

# 94. SIGTERM

Kubernetes commonly sends:

```text
SIGTERM
```

before termination.

Python can register a handler:

```python
import signal

def handle_sigterm(signum, frame):
    logger.info("Shutdown requested")
    raise SystemExit(0)

signal.signal(
    signal.SIGTERM,
    handle_sigterm
)
```

---

# 95. Graceful Shutdown

During shutdown:

```text
stop accepting new work
 ↓
finish safe operations
 ↓
cleanup
 ↓
exit
```

Do not start new destructive work after termination is requested.

---

# 96. Signal + Cleanup

Combine:

```text
signal handling
+
finally cleanup
```

so temporary resources are not abandoned unnecessarily.

---

# 97. Locking

Some automation must not run concurrently.

Examples:

```text
production migration
Terraform state operation
shared deployment
certificate renewal
```

Use a lock mechanism appropriate to the environment.

---

# 98. Local File Lock

For single-host automation, a file lock may prevent two local processes from running simultaneously.

But local locks do not protect against multiple hosts.

---

# 99. Distributed Lock

For distributed CI/CD workflows, use a shared mechanism such as:

```text
CI concurrency control
DynamoDB/other coordination service
database lock
distributed workflow lock
```

according to architecture.

---

# 100. CI Concurrency

For GitHub Actions or Jenkins, configure concurrency where supported.

Example policy:

```text
only one production deployment per service
```

This prevents overlapping deployments.

---

# 101. Race Conditions

Example:

```text
Job A reads image tag
Job B updates image tag
Job A overwrites B's change
```

Prevent using:

```text
locking
optimistic concurrency
branch protection
commit checks
```

---

# 102. Optimistic Concurrency

Before updating a resource:

```text
read version
 ↓
modify
 ↓
update only if version unchanged
```

This protects against stale updates.

---

# 103. Git Optimistic Concurrency

Before pushing a generated change:

```text
fetch
 ↓
verify base
 ↓
modify
 ↓
commit
 ↓
push
```

If remote changed unexpectedly:

```text
stop/rebase according to policy
```

rather than blindly overwriting.

---

# 104. Production Script Performance

Performance matters when scripts process:

```text
thousands of AWS resources
thousands of Kubernetes objects
large log files
many API calls
```

---

# 105. Avoid Unnecessary API Calls

Bad:

```text
call API for each item
```

when the API supports:

```text
pagination
batch
filtering
```

Use server-side filtering where possible.

---

# 106. Pagination

Production APIs commonly paginate.

Always test:

```text
one page
multiple pages
empty response
last page
```

---

# 107. API Rate Limits

Cloud APIs can throttle.

Respect:

```text
429
Throttling
Retry-After
service-specific retry policies
```

---

# 108. Parallelism

Parallel operations can improve speed but increase:

```text
API pressure
resource contention
failure complexity
```

Use bounded concurrency.

---

# 109. Bounded Concurrency

Avoid:

```python
for resource in resources:
    deploy(resource)
```

when thousands of independent resources exist.

But also avoid:

```text
10000 concurrent requests
```

Use a controlled worker count.

---

# 110. Production CLI Output

A CLI should provide concise status:

```text
[INFO] Validating environment
[INFO] AWS account verified
[INFO] EKS cluster verified
[INFO] Deployment started
[INFO] Rollout completed
[INFO] Smoke test passed
[SUCCESS] Deployment completed
```

Detailed diagnostics can go to logs/artifacts.

---

# 111. Human vs Machine Output

CI systems often need:

```text
machine-readable output
```

Humans need:

```text
readable summary
```

Consider supporting:

```bash
--output text
--output json
```

---

# 112. JSON Output

Example:

```json
{
  "status": "success",
  "environment": "staging",
  "service": "orders",
  "run_id": "7812"
}
```

This can be consumed by another automation system.

---

# 113. Exit Code + JSON

Use both:

```text
exit code -> machine success/failure
JSON -> detailed structured result
```

This is useful for CI/CD.

---

# 114. Idempotent CLI

A good CLI should define behavior for:

```text
resource already exists
resource already desired
deployment already current
Git change already applied
```

---

# 115. State Awareness

Before mutation:

```text
read current state
```

Then decide:

```text
no change
update
create
delete
```

---

# 116. Reconciliation Pattern

A useful model:

```text
desired state
      |
      v
current state
      |
      v
calculate difference
      |
      v
apply minimum required change
      |
      v
verify
```

---

# 117. Minimum Change Principle

Avoid unnecessary changes.

If:

```text
replicas already 3
```

do not repeatedly update replicas to 3.

This reduces:

```text
API calls
rollouts
risk
noise
```

---

# 118. Safe Delete

Deletion is the highest-risk action.

Before delete:

```text
identify resource
validate environment
validate ownership
validate dependencies
confirm policy
```

---

# 119. Deletion Preview

A production deletion tool should support:

```bash
cleanup --dry-run
```

Output:

```text
Would delete:
- namespace/test-run-7812
- ECR tag/test-7812
```

---

# 120. Deletion Ownership

Only delete resources created by the automation.

Use:

```text
owner tag
run ID
environment
```

to prove ownership.

---

# 121. Production Delete Guard

For production:

```text
delete -> blocked by default
```

unless the operation is explicitly supported and approved.

---

# 122. Database Safety

Never make a general-purpose cleanup script delete:

```text
production database
```

based only on:

```text
name contains "test"
```

Use explicit ownership metadata and environment boundaries.

---

# 123. Network Safety

A script changing:

```text
security groups
route tables
NACLs
load balancers
```

can cause large blast radius.

Use:

```text
plan
validation
least privilege
approval
verification
```

---

# 124. Blast Radius

Before production automation, ask:

```text
What is the maximum number of resources this command can affect?
```

Prefer:

```text
one service
one namespace
one environment
```

over:

```text
all clusters
all accounts
```

unless intentionally designed.

---

# 125. Scope Restriction

CLI:

```bash
deploy --service orders --environment staging
```

is safer than:

```bash
deploy --all
```

especially for operational tooling.

---

# 126. Explicit `--all`

If an `--all` operation exists:

```text
require stronger validation
```

and possibly:

```text
approval
```

---

# 127. Production Scripting with AWS

Typical workflow:

```text
CLI
 ↓
load config
 ↓
AWS identity
 ↓
account guard
 ↓
region guard
 ↓
resource discovery
 ↓
plan
 ↓
approval
 ↓
mutation
 ↓
verification
```

---

# 128. Production Scripting with EKS

Workflow:

```text
AWS identity
 ↓
EKS cluster identity
 ↓
Kubernetes authentication
 ↓
namespace validation
 ↓
resource validation
 ↓
apply/operation
 ↓
rollout
 ↓
health
 ↓
cleanup
```

---

# 129. Production Scripting with Terraform

Python may orchestrate:

```text
terraform fmt
terraform validate
terraform plan
policy checks
approval
terraform apply
verification
```

Python should not reinvent Terraform state management.

---

# 130. Production Scripting with Helm

Workflow:

```text
validate values
 ↓
helm lint
 ↓
helm template
 ↓
policy
 ↓
helm upgrade
 ↓
rollout
 ↓
smoke
```

---

# 131. Production Scripting with GitOps

Workflow:

```text
validate repo
 ↓
validate branch
 ↓
read current manifest
 ↓
calculate desired change
 ↓
update
 ↓
validate YAML
 ↓
commit
 ↓
push
 ↓
ArgoCD sync
 ↓
verify health
```

---

# 132. Production Scripting with Jenkins

Python can:

```text
trigger job
poll build
retrieve result
download artifacts
```

Use:

```text
timeouts
poll intervals
authentication
```

---

# 133. Production Scripting with GitHub Actions

Python can interact with:

```text
workflow dispatch
run status
artifacts
checks
```

Use short-lived credentials where possible.

---

# 134. Production Scripting with ArgoCD

Python can:

```text
query app
trigger sync where appropriate
wait for health
collect operation details
```

Avoid bypassing GitOps policy accidentally.

---

# 135. Production Scripting with Prometheus

Python can query:

```text
error rate
latency
availability
CPU
memory
```

Then apply release policy.

---

# 136. Production Scripting with ELK

Python can query or integrate with logging infrastructure to retrieve:

```text
deployment errors
application failures
correlation IDs
```

Never dump large log volumes into CI output.

---

# 137. Deployment Verification Script

Conceptual workflow:

```python
def verify_deployment(
    expected_version,
    namespace,
    service
):
    verify_namespace(namespace)
    verify_rollout(service)
    verify_version(expected_version)
    verify_health(service)
    verify_metrics(service)
```

Each function should be independently testable.

---

# 138. Pre-Flight Script

Example conceptual checks:

```python
def preflight(config):
    validate_config(config)
    validate_tools()
    validate_aws_identity(config)
    validate_cluster(config)
    validate_namespace(config)
    validate_git_state(config)
```

If one fails:

```text
do not mutate
```

---

# 139. Deployment Workflow

```python
def deploy(config):
    preflight(config)

    plan = build_plan(config)

    if config.dry_run:
        show_plan(plan)
        return

    execute(plan)

    verify(config)
```

This cleanly separates:

```text
plan
execute
verify
```

---

# 140. Verification Failure

If:

```text
execute -> success
verify -> failure
```

the final result should be:

```text
FAIL
```

The operation succeeded technically but the desired outcome was not achieved.

---

# 141. Transaction Thinking

Infrastructure changes are often not fully transactional.

Example:

```text
update service
 ↓
update ingress
 ↓
update DNS
```

If step 3 fails, steps 1 and 2 may already be applied.

Design:

```text
recovery
rollback
reconciliation
```

instead of assuming atomicity.

---

# 142. Partial Failure

Always ask:

```text
What if step 1 succeeds and step 2 fails?
```

Production automation should have an explicit answer.

---

# 143. Saga-Like Workflow

For multi-step operations:

```text
Step A
 ↓
Step B
 ↓
Step C
```

define compensating actions where possible:

```text
C fails
 ↓
rollback B
 ↓
rollback A
```

Not every infrastructure action is safely reversible, so recovery must be designed carefully.

---

# 144. Checkpointing

For long workflows, record progress:

```text
validated
planned
artifact created
Git updated
deployment started
deployment verified
```

This can help safely resume or diagnose interrupted runs.

---

# 145. Resume vs Restart

A production automation tool should define:

```text
resume behavior
```

rather than blindly restarting every side effect.

---

# 146. Run State

Possible state:

```text
RUNNING
VALIDATED
EXECUTING
VERIFYING
SUCCESS
FAILED
ROLLING_BACK
ROLLED_BACK
```

---

# 147. State Machine

```text
START
  |
  v
VALIDATE
  |
  v
PLAN
  |
  +----> BLOCKED
  |
  v
EXECUTE
  |
  +----> FAILED
  |
  v
VERIFY
  |
  +----> ROLLBACK
  |          |
  |          v
  |       RECOVERY
  |
  v
SUCCESS
```

---

# 148. State Transitions

Do not allow invalid transitions.

Example:

```text
SUCCESS -> EXECUTE
```

should not happen accidentally.

---

# 149. Production Script Testing

Every production script should have tests for:

```text
success
invalid input
wrong environment
authentication failure
authorization failure
timeout
dependency failure
partial failure
verification failure
cleanup
```

---

# 150. Unit Testing Production Scripts

Focus on:

```text
validation
policy
decision logic
error classification
command construction
result parsing
```

---

# 151. Integration Testing Production Scripts

Focus on:

```text
AWS
Kubernetes
Git
ArgoCD
CI/CD
```

real boundaries.

---

# 152. E2E Testing Production Scripts

Focus on:

```text
complete workflow
```

in:

```text
isolated test environment
```

---

# 153. Testing Destructive Logic

Use:

```text
mock clients
test accounts
test namespaces
dry-run
```

Never test destructive logic against production merely to prove it works.

---

# 154. Dependency Injection

Instead of:

```python
def deploy():
    client = KubernetesClient()
```

prefer:

```python
def deploy(client):
    ...
```

This makes unit tests easy.

---

# 155. Client Abstraction

Example:

```python
class KubernetesClient:
    def rollout_status(self, name, namespace):
        ...
```

Workflow code can then depend on the interface rather than low-level API details.

---

# 156. Configuration Object

Instead of passing many arguments:

```python
def deploy(
    environment,
    region,
    cluster,
    namespace,
    service,
    image
):
    ...
```

use a configuration object:

```python
@dataclass
class DeploymentConfig:
    environment: str
    region: str
    cluster: str
    namespace: str
    service: str
    image: str
```

---

# 157. Dataclasses

Python's:

```python
from dataclasses import dataclass
```

can provide structured configuration.

Example:

```python
@dataclass(frozen=True)
class Config:
    environment: str
    region: str
    namespace: str
```

`frozen=True` can prevent accidental mutation.

---

# 158. Configuration Validation Layer

Use:

```text
load
 ↓
parse
 ↓
validate
 ↓
freeze/use
```

Do not repeatedly read environment variables throughout the application.

---

# 159. Centralized Configuration

Bad:

```python
os.getenv("ENV")
```

in 20 files.

Better:

```text
config.py
 ↓
validated Config
 ↓
dependency injection
```

---

# 160. Constants

Use constants for stable policy:

```python
DEFAULT_TIMEOUT = 30
MAX_RETRIES = 3
```

Do not scatter magic numbers throughout the code.

---

# 161. Timeouts as Configuration

Where operationally appropriate:

```text
API timeout
rollout timeout
poll interval
retry count
```

should be configurable but bounded by safe defaults.

---

# 162. Do Not Make Everything Configurable

Too many knobs create:

```text
configuration complexity
```

Expose only settings that operators genuinely need.

---

# 163. Configuration Validation

For example:

```python
if timeout <= 0:
    raise ValueError("timeout must be positive")

if max_retries < 0:
    raise ValueError("max_retries cannot be negative")
```

---

# 164. Production Defaults

Defaults should be:

```text
safe
conservative
documented
```

For destructive operations:

```text
fail closed
```

is generally preferable.

---

# 165. Fail Closed

Example:

```text
cannot identify AWS account
```

Result:

```text
BLOCK
```

Do not assume:

```text
probably staging
```

---

# 166. Fail Open

Fail-open behavior may be appropriate only for certain non-critical operational checks.

For production mutation guards:

```text
fail closed
```

is generally safer.

---

# 167. Health Checks

Before execution:

```text
dependency health
```

may be checked.

After execution:

```text
result health
```

must be checked.

---

# 168. Dependency Health

Examples:

```text
AWS API reachable
Kubernetes API reachable
Git remote reachable
ArgoCD reachable
```

Use short pre-flight checks where appropriate.

---

# 169. Do Not Over-Check

Too many pre-flight API calls can:

```text
slow automation
increase API pressure
create more failure points
```

Only perform checks that reduce meaningful risk.

---

# 170. Network Resilience

External APIs can fail due to:

```text
DNS
TCP
TLS
proxy
routing
service outage
```

Production scripts should distinguish:

```text
connection failure
HTTP failure
authentication failure
```

---

# 171. Proxy Configuration

CI environments may use:

```text
HTTP_PROXY
HTTPS_PROXY
NO_PROXY
```

Validate network assumptions when needed.

For Kubernetes, ensure internal cluster endpoints are handled correctly by proxy settings.

---

# 172. DNS Testing

If an automation depends on a hostname:

```text
resolve
connect
authenticate
```

Do not assume DNS success from a cached local environment.

---

# 173. TLS Verification

Never disable certificate verification globally just to make automation work.

Bad:

```text
verify=False
```

without a justified controlled exception.

---

# 174. Time Synchronization

Authentication and certificates can fail if system time is incorrect.

Production automation should rely on properly synchronized hosts/runners.

---

# 175. Timezone

Prefer:

```text
UTC
```

for automation logs and timestamps unless there is a specific operational requirement.

---

# 176. Datetime

Use timezone-aware timestamps.

Avoid mixing:

```text
naive datetime
```

with:

```text
timezone-aware datetime
```

---

# 177. Audit Timestamps

Record:

```text
started_at
completed_at
```

in a consistent timezone.

---

# 178. Run Duration

Measure:

```text
duration = completed_at - started_at
```

Track it over time.

A suddenly slower script may indicate:

```text
API throttling
network issue
resource growth
regression
```

---

# 179. Performance Metrics

Production automation can expose:

```text
execution duration
API calls
retry count
resources processed
success count
failure count
```

---

# 180. Observability for Automation

The automation itself should be observable.

Monitor:

```text
success rate
failure rate
duration
timeouts
retry count
resource count
```

---

# 181. Automation Metrics

Example:

```text
devops_automation_runs_total
devops_automation_failures_total
devops_automation_duration_seconds
devops_automation_retries_total
```

The exact metric naming depends on the monitoring standard.

---

# 182. Automation Health

A healthy automation system should show:

```text
stable execution time
low unexpected failure rate
low retry rate
no resource leaks
```

---

# 183. Alerts

Alert on meaningful conditions:

```text
automation failure rate high
production deployment verification failing
cleanup failures increasing
automation duration abnormal
```

Avoid alerting on every normal retry.

---

# 184. Automation Logs in ELK

If logs are shipped to ELK, include fields:

```text
service
environment
run_id
event
status
duration
```

This makes queries easier.

---

# 185. Automation Metrics in Prometheus

If the environment supports metrics:

```text
run status
duration
failure reason
```

can be exposed or pushed through an appropriate architecture.

Avoid creating high-cardinality labels such as:

```text
unique request ID
```

for every metric label.

---

# 186. High Cardinality

Do not use unbounded values as Prometheus labels:

```text
commit SHA
request ID
UUID
full URL
```

These can create huge time-series counts.

Put them in logs instead.

---

# 187. Logs vs Metrics

Use:

```text
metrics -> trends/counts
logs -> detailed events
```

Example:

```text
metric:
deployment failures = 7

log:
service=orders
run_id=7812
reason=rollout_timeout
```

---

# 188. Traceability

For a deployment automation run:

```text
Git SHA
 ↓
CI run ID
 ↓
image digest
 ↓
GitOps commit
 ↓
ArgoCD operation
 ↓
Kubernetes rollout
```

should be traceable.

---

# 189. Artifact Evidence

Store:

```text
test report
security report
deployment report
verification report
```

as CI artifacts where appropriate.

---

# 190. Production Script Packaging

For reusable tools, prefer packaging the project rather than copying a `.py` file manually between servers.

Use:

```text
pyproject.toml
```

and a package layout.

---

# 191. `pyproject.toml`

It can define:

```text
project metadata
dependencies
entry points
tool configuration
```

---

# 192. CLI Entry Point

A package can expose:

```bash
devops-tool deploy
```

instead of:

```bash
python scripts/deploy.py
```

This is cleaner for reusable automation.

---

# 193. Dependency Pinning

Use a controlled dependency strategy.

For production automation, unexpected dependency upgrades can change behavior.

Use:

```text
lock files
constraints
pinned versions
```

as appropriate.

---

# 194. Virtual Environment

Local development:

```bash
python -m venv .venv
```

then install project dependencies.

Do not rely on random global Python packages.

---

# 195. Production Python Environment

Possible deployment models:

```text
Docker image
virtual environment
managed runtime
Kubernetes Job
CI runner
```

For DevOps automation, containers often provide reproducibility.

---

# 196. Dockerizing Python Automation

Example:

```text
Python source
 ↓
requirements/lock
 ↓
Docker build
 ↓
test
 ↓
scan
 ↓
registry
 ↓
CI/Kubernetes
```

---

# 197. Production Docker Image

Use:

```text
minimal base
non-root user
pinned dependencies
health/command behavior
no secrets
```

---

# 198. Container Entry Point

Example concept:

```dockerfile
ENTRYPOINT ["devops-tool"]
```

Then:

```bash
docker run image deploy --environment staging
```

---

# 199. Environment Variables in Containers

Pass:

```text
configuration
```

through environment variables or mounted configuration.

Do not bake secrets into the image.

---

# 200. Image Immutability

Build:

```text
one immutable image
```

and promote it through:

```text
test
staging
production
```

rather than rebuilding different images for each environment.

---

# 201. Container User

Avoid running production automation as:

```text
root
```

unless explicitly required.

Use a dedicated non-root user.

---

# 202. File System Permissions

A non-root container may need write access to:

```text
temporary directory
workspace
```

Give only the permissions required.

---

# 203. Read-Only Filesystem

For some automation containers, a read-only root filesystem can improve security.

Provide writable temporary storage only where needed.

---

# 204. Container Signal Handling

Use an exec-form entrypoint so signals reach Python correctly:

```text
ENTRYPOINT ["python", "-m", "automation"]
```

rather than unnecessarily wrapping Python inside a shell.

---

# 205. Kubernetes Job for Automation

Production automation can run as:

```text
Kubernetes Job
```

Flow:

```text
CI
 ↓
create Job
 ↓
automation runs
 ↓
Job completes
 ↓
CI reads result
```

---

# 206. Kubernetes Job Safety

Configure:

```text
restart policy
backoff limit
active deadline
service account
resource limits
```

according to workload requirements.

---

# 207. Automation Service Account

Use a dedicated:

```text
ServiceAccount
```

with minimal RBAC.

Do not use:

```text
cluster-admin
```

for convenience.

---

# 208. AWS Identity for Kubernetes Jobs

If the automation needs AWS access, use the cluster's supported workload identity mechanism and least privilege.

Avoid static AWS credentials in Secrets whenever possible.

---

# 209. Jenkins Agent

Python automation may run inside:

```text
Jenkins agent container
```

This provides:

```text
known Python version
known tools
reproducible environment
```

---

# 210. GitHub Actions Runner

A CI job can run:

```text
checkout
 ↓
setup Python
 ↓
install dependencies
 ↓
test
 ↓
run automation
```

Use:

```text
OIDC
```

for cloud identity when supported.

---

# 211. CI Pipeline for Production Script

Example:

```text
Checkout
   |
   v
Install
   |
   v
Lint
   |
   v
Unit Tests
   |
   v
Security Scan
   |
   v
Build
   |
   v
Integration Tests
   |
   v
Package
   |
   v
Publish
```

---

# 212. Deployment of Automation Tool

```text
Git
 ↓
CI
 ↓
test
 ↓
security
 ↓
build image
 ↓
push ECR
 ↓
deploy automation
```

The automation tool itself should follow the same DevSecOps principles as application code.

---

# 213. Versioning

Use explicit versions:

```text
1.0.0
1.1.0
1.1.1
```

or the organization's standard.

Record the automation version in logs.

---

# 214. Automation Version in Logs

Example:

```text
tool_version=1.4.2
run_id=7812
```

This is valuable when an old version caused an incident.

---

# 215. Backward Compatibility

If operators depend on the CLI:

```text
devops-tool deploy --environment staging
```

avoid breaking arguments without a migration plan.

---

# 216. Deprecation

When removing an option:

```text
warn
 ↓
document
 ↓
deprecate
 ↓
remove
```

Do not silently reinterpret old arguments.

---

# 217. Documentation

A production automation repository should document:

```text
installation
authentication
configuration
usage
permissions
examples
failure behavior
rollback
troubleshooting
```

---

# 218. README Example

Minimum sections:

```text
# Tool Name

## Purpose
## Architecture
## Installation
## Authentication
## Configuration
## Usage
## Dry Run
## Permissions
## CI/CD
## Troubleshooting
## Security
## Development
## Testing
```

---

# 219. Operational Runbook

For production automation, create a runbook covering:

```text
normal execution
common failures
rollback
manual recovery
cleanup
escalation
```

---

# 220. Runbook Example

```text
Deployment timeout
 ↓
check CI logs
 ↓
check ArgoCD
 ↓
check deployment
 ↓
check pods/events
 ↓
check ALB
 ↓
check application logs
 ↓
rollback if required
```

---

# 221. Troubleshooting — Script Hangs

Check:

```text
network call
subprocess
poll loop
lock
deadlock
API rate limit
```

Ensure every external wait has a timeout.

---

# 222. Troubleshooting — Script Hangs on `kubectl`

Check:

```text
Kubernetes authentication
API endpoint
network
proxy
context
```

and use:

```text
subprocess timeout
```

---

# 223. Troubleshooting — Script Hangs on AWS API

Check:

```text
network
credential provider chain
proxy
API endpoint
retry configuration
```

---

# 224. Troubleshooting — Infinite Poll

Bad:

```python
while not ready:
    check()
```

Better:

```python
deadline = time.monotonic() + timeout

while time.monotonic() < deadline:
    if ready():
        return
    time.sleep(interval)

raise TimeoutError("Timed out")
```

---

# 225. Why `time.monotonic()`

For measuring durations, prefer:

```python
time.monotonic()
```

because wall-clock changes should not make elapsed-time calculations incorrect.

---

# 226. Troubleshooting — Wrong Environment

Check:

```text
CLI
environment variable
AWS profile
AWS account
region
Kube context
namespace
Git branch
```

The script should print enough safe identity information to diagnose the mismatch.

---

# 227. Troubleshooting — Permission Denied

Classify:

```text
authentication
authorization
resource policy
network
```

A 403 is usually not solved by blindly retrying.

---

# 228. Troubleshooting — API Throttling

Look for:

```text
429
ThrottlingException
Retry-After
```

Then:

```text
backoff
jitter
reduce concurrency
```

---

# 229. Troubleshooting — Resource Not Found

Determine whether:

```text
resource genuinely doesn't exist
```

or:

```text
eventual consistency
wrong account
wrong region
wrong cluster
wrong namespace
```

---

# 230. Troubleshooting — Deployment Succeeds but Verification Fails

Check:

```text
image
version
readiness
service
ingress
application logs
Prometheus
```

The deployment command may have succeeded while the application failed.

---

# 231. Troubleshooting — Cleanup Did Not Run

Check:

```text
finally
CI post stage
Kubernetes Job lifecycle
process termination
```

Then add a regression test.

---

# 232. Troubleshooting — Cleanup Deletes Wrong Resources

Check:

```text
resource selectors
tags
labels
run ID
environment
account
region
```

This is a high-priority safety issue.

---

# 233. Troubleshooting — CI Says Success but Deployment Failed

Check:

```text
exit code
exception handling
subprocess check=True
pipeline stage result
verification result
```

The automation must return non-zero on failure.

---

# 234. Troubleshooting — Logs Missing

Check:

```text
logging configuration
log level
stdout/stderr
CI capture
container logging
```

---

# 235. Troubleshooting — Secrets in Logs

Check:

```text
command output
exception text
environment dumps
debug logs
CI artifacts
```

Never use:

```python
logger.debug(vars(config))
```

if configuration contains secrets.

---

# 236. Troubleshooting — Dependency Version Conflict

Check:

```text
Python version
lock file
installed package versions
CI image
local environment
```

Reproduce inside the same container/runtime as CI.

---

# 237. Troubleshooting — Works Locally, Fails in Jenkins

Compare:

```text
Python
PATH
AWS identity
Kube context
network
proxy
filesystem
permissions
environment variables
```

---

# 238. Troubleshooting — Works in Jenkins, Fails in GitHub Actions

Compare:

```text
runner image
tool versions
OIDC/IAM
secrets
network
checkout depth
environment variables
```

---

# 239. Troubleshooting — Git Push Fails

Check:

```text
branch protection
authentication
remote
concurrent changes
permissions
```

Do not force-push production configuration blindly.

---

# 240. Troubleshooting — ArgoCD Does Not Sync

Check:

```text
repository
revision
path
application
sync policy
manifest validity
ArgoCD health
```

---

# 241. Troubleshooting — ArgoCD Synced but Application Unhealthy

Check:

```text
pods
events
readiness
ConfigMap
Secret
service
ingress
application logs
```

---

# 242. Troubleshooting — EKS API Unreachable

Check:

```text
AWS identity
cluster endpoint
network path
security controls
authentication
Kubeconfig/context
```

---

# 243. Troubleshooting — Wrong Kube Context

A production safety check should display:

```text
cluster
namespace
user/service account
```

before mutation.

---

# 244. Troubleshooting — Slow Script

Measure:

```text
API call duration
number of calls
retries
resource count
subprocess duration
```

Then optimize the real bottleneck.

---

# 245. Troubleshooting — API Call Explosion

If processing:

```text
10,000 resources
```

and making:

```text
10,000+ API calls
```

consider:

```text
batching
pagination
server-side filtering
caching
bounded parallelism
```

---

# 246. Troubleshooting — High Memory Usage

Check:

```text
loading entire API response
large log files
unbounded lists
large JSON
```

Use:

```text
streaming
pagination
bounded collections
```

where appropriate.

---

# 247. Troubleshooting — Duplicate Resources

Check:

```text
idempotency
resource lookup
stable identifiers
concurrent runs
retry logic
```

---

# 248. Troubleshooting — Duplicate Git Commits

Check:

```text
current desired value
Git diff
concurrency
retry behavior
```

Do not commit if there is no meaningful change.

---

# 249. Troubleshooting — Race Condition

Check:

```text
concurrent jobs
shared resource
timing
locking
optimistic concurrency
```

---

# 250. Troubleshooting — Rollback Failed

Check:

```text
previous artifact
previous manifest
cluster health
dependency compatibility
database changes
```

Rollback is not always possible if data/schema changes are irreversible.

---

# 251. Database Migration Safety

For deployments involving database schema changes:

```text
expand
 ↓
deploy compatible code
 ↓
migrate
 ↓
contract later
```

can be safer than:

```text
destructive schema change
 ↓
new application
```

The exact strategy depends on the database and application.

---

# 252. Production Script Security Model

```text
Identity
 ↓
Authentication
 ↓
Authorization
 ↓
Validation
 ↓
Least privilege
 ↓
Execution
 ↓
Audit
```

---

# 253. Least Privilege

Give the script only:

```text
required permissions
```

Example:

```text
deployment script
```

should not automatically have:

```text
administrator permissions
```

---

# 254. IAM Scope

Prefer:

```text
specific resource
specific action
specific environment
```

over:

```text
*
```

where practical.

---

# 255. Kubernetes RBAC Scope

Prefer:

```text
namespace Role
```

when cluster-wide access is unnecessary.

---

# 256. Secret Exposure Through Environment

Remember:

```text
environment variables
```

can sometimes appear in:

```text
process inspection
debug output
CI diagnostics
```

Use secret stores and platform controls appropriate to the environment.

---

# 257. Input Validation

Treat:

```text
CLI
environment
API
Git
file
```

inputs as untrusted until validated.

---

# 258. Path Traversal

If a script accepts:

```text
file path
```

do not blindly concatenate it into sensitive directories.

Validate allowed paths.

---

# 259. URL Validation

If automation calls a user-supplied URL:

```text
validate scheme
host
allowed destinations
```

This can prevent SSRF-style problems in automation tools.

---

# 260. Unsafe Deserialization

Avoid unsafe deserialization of untrusted data.

Prefer safe parsers and explicit schemas.

---

# 261. YAML Safety

Use safe YAML loading for untrusted YAML.

Avoid unsafe object construction.

---

# 262. Dependency Supply Chain

Production scripts themselves are software.

Protect:

```text
dependencies
build process
container
registry
release
```

---

# 263. Dependency Scanning

Use:

```text
SCA
```

to identify known vulnerable Python packages.

---

# 264. Secret Scanning

Run secret scanning against:

```text
source
Git history where required
configuration
test fixtures
```

---

# 265. Static Analysis

Use tools such as:

```text
Ruff
Bandit
SonarQube
```

according to project standards.

---

# 266. Code Formatting

Use a consistent formatter.

Examples:

```text
Ruff formatter
Black
```

Choose one project standard.

---

# 267. Linting

Lint for:

```text
unused imports
unsafe patterns
complexity
style
potential bugs
```

---

# 268. Type Checking

For larger production tools, consider:

```text
mypy
pyright
```

to catch incorrect interfaces before runtime.

---

# 269. Type Hints

Example:

```python
def validate_environment(
    expected: str,
    actual: str
) -> None:
    if expected != actual:
        raise RuntimeError(
            "Environment mismatch"
        )
```

Type hints improve:

```text
readability
IDE support
static analysis
```

---

# 270. Production Code Quality

Aim for:

```text
small functions
clear names
explicit dependencies
limited side effects
testable policies
```

---

# 271. Function Size

Avoid huge functions such as:

```text
deploy_everything()
```

with hundreds of lines.

Break into:

```text
validate
plan
execute
verify
cleanup
```

---

# 272. Single Responsibility

Each component should have a focused purpose.

Example:

```text
ConfigLoader
EnvironmentValidator
AWSClient
KubernetesClient
DeploymentVerifier
```

---

# 273. Dependency Inversion

Business logic should not depend directly on:

```text
subprocess
boto3
Kubernetes API
```

where abstraction provides meaningful testing value.

---

# 274. Adapter Layer

Example:

```text
DeploymentWorkflow
      |
      v
KubernetesAdapter
      |
      v
Kubernetes API
```

The workflow can be tested using a fake adapter.

---

# 275. Production Testing Strategy

```text
Pre-commit
 ↓
Lint
 ↓
Unit
 ↓
Security
 ↓
Integration
 ↓
Build
 ↓
E2E
 ↓
Release
```

---

# 276. Fast Feedback

Put fast tests first:

```text
lint
unit
basic security
```

Slow tests later:

```text
integration
E2E
```

---

# 277. Test Selection

Not every code change requires every expensive test.

Use:

```text
test markers
path-based selection
dependency-aware CI
```

where reliable.

---

# 278. Production Change Test

For a change to:

```text
AWS client
```

run:

```text
unit AWS tests
integration AWS tests
```

For a change to:

```text
CLI parser
```

run:

```text
CLI tests
unit tests
```

---

# 279. Regression Testing

Every production incident should ideally produce:

```text
regression test
```

Example:

```text
Incident:
wrong cluster selected

Regression:
wrong cluster -> BLOCK
```

---

# 280. Operational Metrics

Track:

```text
deployment success rate
automation success rate
mean duration
failure rate
rollback rate
cleanup failures
```

---

# 281. MTTR

Automation can help reduce:

```text
Mean Time To Recovery
```

by:

```text
automatic diagnostics
health checks
rollback
repeatable recovery
```

---

# 282. Automation Reliability

A production automation platform should measure:

```text
successful runs
failed runs
retry rate
timeout rate
manual intervention rate
```

---

# 283. Manual Intervention

A high manual-intervention rate may indicate:

```text
poor validation
weak automation
unstable infrastructure
unclear failure handling
```

---

# 284. Production Readiness Checklist

Before releasing a DevOps Python tool:

```text
[ ] CLI documented
[ ] Configuration validated
[ ] Secrets externalized
[ ] IAM/RBAC least privilege
[ ] AWS account guard
[ ] region guard
[ ] cluster guard
[ ] namespace guard
[ ] branch guard
[ ] dry-run
[ ] idempotency
[ ] timeout
[ ] retry policy
[ ] logging
[ ] metrics
[ ] exit codes
[ ] cleanup
[ ] signal handling
[ ] unit tests
[ ] integration tests
[ ] security scans
[ ] container scan
[ ] rollback/recovery
[ ] runbook
```

---

# 285. Production Deployment Checklist

```text
[ ] Correct version
[ ] Correct environment
[ ] Correct AWS account
[ ] Correct region
[ ] Correct EKS cluster
[ ] Correct namespace
[ ] Correct Git branch
[ ] Correct image
[ ] Security passed
[ ] Approval present
[ ] Dry-run/plan reviewed
[ ] Deployment started
[ ] Rollout healthy
[ ] Smoke test passed
[ ] Metrics healthy
[ ] Logs healthy
[ ] Result recorded
```

---

# 286. Production Cleanup Checklist

```text
[ ] Temporary files removed
[ ] Temporary namespaces removed
[ ] Test resources removed
[ ] Temporary credentials cleaned
[ ] CI workspace cleaned
[ ] Cleanup failures recorded
```

---

# 287. Security Checklist

```text
[ ] No hard-coded secrets
[ ] No secret logging
[ ] Safe subprocess usage
[ ] Input validation
[ ] TLS verification
[ ] Least privilege
[ ] Secret scanning
[ ] Dependency scanning
[ ] Static analysis
[ ] Container scanning
[ ] Non-root container
```

---

# 288. Observability Checklist

```text
[ ] Structured logs
[ ] Run ID
[ ] Environment
[ ] Version
[ ] Duration
[ ] Failure reason
[ ] Metrics
[ ] CI artifacts
[ ] Audit trail
```

---

# 289. Reliability Checklist

```text
[ ] Timeout
[ ] Retry classification
[ ] Backoff
[ ] Jitter
[ ] Idempotency
[ ] Cleanup
[ ] Signal handling
[ ] Lock/concurrency policy
[ ] Verification
[ ] Recovery
```

---

# 290. Real-World Architecture — DevOps Automation Platform

Consider a production environment using:

```text
AWS
EKS
Terraform
Docker
Helm
Jenkins/GitHub Actions
ArgoCD
Prometheus
Grafana
ELK
SonarQube
Trivy
Veracode
```

Python acts as an automation and verification layer.

Architecture:

```text
                         Git
                          |
                          v
                Jenkins / GitHub Actions
                          |
              +-----------+-----------+
              |           |           |
              v           v           v
           Pytest     SonarQube     Trivy
              |                       |
              +-----------+-----------+
                          |
                          v
                    Python Tool
                          |
          +---------------+---------------+
          |               |               |
          v               v               v
         AWS         Kubernetes         Git
          |               |               |
          v               v               v
        EKS              Helm          GitOps
          |                               |
          +---------------+---------------+
                          |
                          v
                        ArgoCD
                          |
                          v
                         EKS
                          |
              +-----------+-----------+
              |                       |
              v                       v
          Prometheus                  ELK
              |                       |
              +-----------+-----------+
                          |
                          v
                    Verification
                          |
                    +-----+-----+
                    |           |
                   PASS        FAIL
                    |           |
                    v           v
                 Release     Rollback/
                             Diagnose
```

---

# 291. Python's Role

Python should act as:

```text
orchestrator
validator
policy engine
API client
diagnostic tool
verification layer
```

It should not unnecessarily replace:

```text
Terraform
Kubernetes
ArgoCD
CI/CD controllers
```

---

# 292. Terraform + Python

Terraform owns:

```text
infrastructure provisioning
```

Python can:

```text
validate
trigger
inspect
verify
report
```

---

# 293. Kubernetes + Python

Kubernetes owns:

```text
desired-state orchestration
```

Python can:

```text
inspect
diagnose
test
verify
```

---

# 294. ArgoCD + Python

ArgoCD owns:

```text
GitOps reconciliation
```

Python can:

```text
query
wait
verify
diagnose
gate
```

---

# 295. CI/CD + Python

CI owns:

```text
workflow orchestration
```

Python can provide:

```text
custom automation logic
```

---

# 296. Security Tools + Python

Security tools own:

```text
scanning
```

Python can:

```text
parse results
apply policy
produce release decision
```

---

# 297. Observability + Python

Prometheus/ELK own:

```text
telemetry storage and querying
```

Python can:

```text
query signals
correlate deployment
produce verification result
```

---

# 298. Release Gate Architecture

```text
Tests
  |
  v
Security
  |
  v
Artifact
  |
  v
ArgoCD
  |
  v
Kubernetes
  |
  v
Smoke
  |
  v
Prometheus
  |
  v
Release Decision
```

---

# 299. Release Decision

Example:

```python
def release_allowed(
    tests,
    security,
    deployment,
    smoke,
    metrics,
):
    return all([
        tests,
        security,
        deployment,
        smoke,
        metrics,
    ])
```

This is simple, deterministic, and easy to test.

---

# 300. Production Release Output

Example:

```text
==================================================
Production Release Verification
==================================================

Environment : production
Service     : orders
Version     : 2026.08.18-42
Run ID      : 7812

Tests       : PASS
Security    : PASS
Artifact    : VERIFIED
ArgoCD      : HEALTHY
EKS         : READY
Smoke       : PASS
Metrics     : HEALTHY

==================================================
RESULT      : GO
==================================================
```

---

# 301. Failure Output

```text
==================================================
Production Release Verification
==================================================

Environment : production
Service     : orders
Run ID      : 7812

Tests       : PASS
Security    : PASS
Artifact    : VERIFIED
ArgoCD      : HEALTHY
EKS         : READY
Smoke       : FAIL
Metrics     : UNKNOWN

Reason      : critical API returned HTTP 503

==================================================
RESULT      : BLOCK
==================================================
```

---

# 302. Production Automation Golden Rules

```text
1. Validate before mutate.
2. Identify the environment explicitly.
3. Fail closed on safety checks.
4. Use least privilege.
5. Never hard-code secrets.
6. Use timeouts.
7. Retry only transient failures.
8. Make mutations idempotent.
9. Verify actual state.
10. Clean up temporary resources.
11. Return correct exit codes.
12. Log enough to troubleshoot.
13. Never log secrets.
14. Protect destructive operations.
15. Keep blast radius small.
16. Separate plan from execution.
17. Test failure paths.
18. Test rollback/recovery.
19. Track every production run.
20. Convert incidents into regression tests.
```

---

# 303. Senior Interview — What Makes a Python Script Production-Ready?

A strong answer:

```text
I treat a production script as an operational application rather
than a one-off utility.

I add structured configuration, input validation, environment
guards, least-privilege authentication, timeouts, classified
retries, idempotency, logging, cleanup, correct exit codes and
post-operation verification.

For destructive workflows I also use dry-run/plan behavior,
approval controls and explicit AWS account, region and Kubernetes
cluster validation.
```

---

# 304. Senior Interview — How Do You Make a Script Safe?

Answer:

```text
First I validate all inputs and the target environment.

For AWS automation I verify account and region.

For Kubernetes I verify cluster and namespace.

For GitOps I verify repository and branch.

I use least privilege, dry-run where possible, idempotent
operations and stronger approval requirements for destructive
production changes.
```

---

# 305. Senior Interview — How Do You Handle Failures?

Answer:

```text
I classify failures into configuration, authentication,
authorization, timeout, dependency and verification failures.

Transient failures can be retried with bounded exponential
backoff and jitter.

Permanent failures fail immediately.

After execution I verify the resulting state and collect
diagnostics if verification fails.
```

---

# 306. Senior Interview — Why Is Idempotency Important?

Answer:

```text
CI/CD jobs and automation can be retried or accidentally executed
more than once.

If the operation is idempotent, repeated execution converges to
the same desired state instead of creating duplicate resources
or repeated side effects.
```

---

# 307. Senior Interview — How Do You Prevent Wrong Cluster Deployment?

Answer:

```text
Before any mutation I retrieve the actual cluster identity and
compare it with the expected cluster from validated configuration.

If they do not match, the script fails closed.

I also validate the namespace and workload context.
```

---

# 308. Senior Interview — How Do You Handle Secrets?

Answer:

```text
I never hard-code secrets or commit them to Git.

I prefer IAM roles, OIDC and managed secret stores depending on
the platform.

I also ensure credentials are not included in logs, exceptions,
debug output or CI artifacts.
```

---

# 309. Senior Interview — How Do You Handle Subprocesses?

Answer:

```text
I prefer argument lists over shell command strings, use check=True
when failures must stop execution, capture output carefully, and
always use bounded timeouts for commands that can hang.

I also validate inputs to prevent command injection.
```

---

# 310. Senior Interview — How Do You Make Python Automation Observable?

Answer:

```text
I use structured logging with environment, resource, run ID,
version and event fields.

For larger automation platforms I expose execution metrics such
as success rate, duration, retries and failures.

Detailed diagnostics go to logs or CI artifacts while metrics
provide operational trends.
```

---

# 311. Senior Interview — How Do You Handle Cleanup?

Answer:

```text
I put cleanup in finally blocks or CI post actions so it runs
even after failures.

Temporary resources are tagged or labeled with ownership and
run IDs.

Cleanup itself is guarded so that it cannot accidentally delete
resources outside the test scope.
```

---

# 312. Senior Interview — How Do You Prevent Infinite Loops?

Answer:

```text
Every polling operation has a deadline or timeout.

I use time.monotonic for elapsed time and bounded polling with
an appropriate interval.

If the condition is not met before the deadline, the script
raises a clear timeout failure and collects diagnostics.
```

---

# 313. Senior Interview — What Is Fail-Closed Behavior?

Answer:

```text
If the automation cannot establish that the target environment
is safe, it stops rather than guessing.

For example, if AWS account identity or EKS cluster identity
cannot be verified, a production mutation is blocked.
```

---

# 314. Senior Interview — How Do You Test Production Scripts?

Answer:

```text
I use unit tests for validation and policy logic, integration
tests for real AWS/Kubernetes/Git/API boundaries, and E2E tests
for critical workflows.

I also test negative paths such as wrong environment, timeout,
permission failure, rollout failure, cleanup failure and
verification failure.
```

---

# 315. Senior Interview — How Do You Deploy Python Automation?

Answer:

```text
For reusable production automation I package it as a Python
application and often build an immutable container image.

The image is tested and security-scanned before being published.

It can then run through Jenkins, GitHub Actions, Kubernetes Jobs
or another controlled execution environment.
```

---

# 316. Senior Interview — Why Use Python Instead of Bash?

Answer:

```text
Bash is excellent for simple command orchestration.

For larger workflows Python provides stronger structure,
exception handling, API clients, testing, type hints, data
processing and reusable modules.

I use Bash for simple shell operations and Python when the
automation becomes application-like.
```

---

# 317. Senior Interview — Python vs Terraform

Answer:

```text
Terraform should own declarative infrastructure provisioning.

Python should handle orchestration, validation, API integration,
diagnostics and verification.

I avoid writing Python that duplicates Terraform's state management.
```

---

# 318. Senior Interview — Python vs ArgoCD

Answer:

```text
ArgoCD should own GitOps reconciliation.

Python can query ArgoCD, validate application state, wait for
sync and health, and use those signals in release verification.

I avoid bypassing GitOps by directly mutating production resources
when GitOps is the established deployment mechanism.
```

---

# 319. Senior Interview — How Do You Handle Partial Failure?

Answer:

```text
I design the workflow around explicit states and checkpoints.

After each major mutation I verify the result.

If a later step fails, I either execute a tested compensating
action or move the workflow into a controlled recovery state.

I never assume multi-step infrastructure changes are atomic.
```

---

# 320. Senior Interview — How Do You Reduce Blast Radius?

Answer:

```text
I scope operations to a specific environment, account, cluster,
namespace and service.

Destructive operations use explicit selectors and ownership
metadata.

For high-risk operations I use plan/dry-run and approval gates.
```

---

# 321. Senior Interview — What Should a Production Script Log?

Answer:

```text
I log the operation, environment, resource, run ID, version,
status, duration and failure reason.

I do not log credentials, tokens, passwords or secret payloads.
```

---

# 322. Senior Interview — How Do You Diagnose a Failed Deployment?

Answer:

```text
I first identify whether the failure is CI, artifact, GitOps,
Kubernetes, networking, application or observability related.

Then I inspect ArgoCD status, deployment rollout, pod events,
logs, service/ingress state and Prometheus signals.

The automation should collect these diagnostics automatically
when possible.
```

---

# 323. Senior Interview — How Do You Handle Retries?

Answer:

```text
I retry only known transient conditions.

Timeouts, 429 and selected 5xx responses may be retryable.

Authentication errors, invalid configuration and permission
errors normally should fail immediately.

Retries are bounded and use exponential backoff with jitter.
```

---

# 324. Senior Interview — How Do You Make a Deployment Tool Auditable?

Answer:

```text
I record the operator or service identity, environment,
commit SHA, tool version, image digest, run ID, action,
result and timestamps.

The audit record must not contain secrets.
```

---

# 325. Senior Interview — How Do You Handle Concurrent Deployments?

Answer:

```text
I define concurrency policy explicitly.

For environments where overlapping deployments are unsafe,
I use CI concurrency controls or distributed locking.

I also use optimistic concurrency for Git changes where needed.
```

---

# 326. Senior Interview — How Do You Optimize a Slow Automation Script?

Answer:

```text
I measure first.

I inspect API call counts, retry rates, subprocess duration,
resource counts and memory usage.

Then I use server-side filtering, pagination, batching,
bounded concurrency or caching where appropriate.

I avoid optimizing based on assumptions.
```

---

# 327. Senior Interview — How Do You Handle Production Rollback?

Answer:

```text
Rollback is treated as a tested workflow, not a theoretical command.

I verify the previous artifact or manifest, trigger rollback
under defined conditions, then verify rollout, application health
and metrics.

I also test cases where rollback itself fails.
```

---

# 328. Senior Interview — What Is the Most Important Production Principle?

A strong answer:

```text
Fail safely.

If the automation cannot prove that the target environment,
identity, configuration and desired state are correct, it should
stop rather than guessing.

The automation must protect production even when dependencies,
credentials or inputs behave unexpectedly.
```

---

# 329. Production Script Review Checklist

Before approving code:

```text
Architecture
[ ] Responsibilities separated
[ ] Side effects isolated
[ ] Dependencies injectable

Safety
[ ] Environment guard
[ ] Account guard
[ ] Cluster guard
[ ] Namespace guard
[ ] Branch guard
[ ] Blast radius limited

Reliability
[ ] Timeouts
[ ] Retry classification
[ ] Backoff
[ ] Idempotency
[ ] Cleanup
[ ] Recovery

Security
[ ] No hard-coded secrets
[ ] Least privilege
[ ] Safe subprocess
[ ] Input validation
[ ] TLS verification
[ ] Dependency scanning

Observability
[ ] Structured logs
[ ] Run ID
[ ] Metrics
[ ] Exit codes
[ ] Audit information

Testing
[ ] Unit
[ ] Integration
[ ] Negative tests
[ ] Failure tests
[ ] Regression tests
```

---

# 330. Final Production Scripting Model

Remember this architecture:

```text
                 PRODUCTION SCRIPT
                        |
       +----------------+----------------+
       |                |                |
       v                v                v
   CONFIG           IDENTITY          SAFETY
       |                |                |
       +----------------+----------------+
                        |
                        v
                    PREFLIGHT
                        |
                        v
                      PLAN
                        |
                   +----+----+
                   |         |
                DRY-RUN    APPROVAL
                   |         |
                   +----+----+
                        |
                        v
                    EXECUTE
                        |
              +---------+---------+
              |                   |
              v                   v
           RETRY              FAILURE
              |                   |
              v                   v
          VERIFY              RECOVERY
              |
              v
           CLEANUP
              |
              v
            REPORT
              |
              v
          EXIT CODE
```

---

# 331. Complete Production Script Lifecycle

```text
1. Parse CLI
2. Load configuration
3. Validate configuration
4. Validate credentials
5. Validate target environment
6. Validate dependencies
7. Run pre-flight checks
8. Build execution plan
9. Support dry-run
10. Obtain approval if required
11. Execute bounded operations
12. Retry transient failures
13. Verify actual state
14. Collect diagnostics
15. Cleanup temporary resources
16. Record audit information
17. Produce machine-readable result
18. Return correct exit code
```

---

# 332. Final Takeaway

Production Python scripting is not primarily about writing more code.

It is about controlling risk around automation.

The production mindset is:

```text
INPUT
 ↓
VALIDATE
 ↓
IDENTIFY TARGET
 ↓
PLAN
 ↓
EXECUTE SAFELY
 ↓
VERIFY
 ↓
RECOVER IF REQUIRED
 ↓
CLEANUP
 ↓
REPORT
```

The most important properties are:

```text
SAFE
REPEATABLE
OBSERVABLE
IDEMPOTENT
SECURE
TESTABLE
RECOVERABLE
```

For a DevOps Engineer, Python becomes especially powerful when it is used as the glue between:

```text
AWS
Kubernetes
Terraform
Docker
Git
Jenkins
GitHub Actions
ArgoCD
Prometheus
ELK
DevSecOps tooling
```

while allowing each specialized platform to remain responsible for its own domain.

The objective is not to create a giant Python script that controls everything.

The objective is to build **small, testable, safe automation components that orchestrate production systems without increasing operational risk.**

---

# 333. Section Progress

```text
10-Python-Production/
│
├── 01-Production-Scripting.md        ✓
├── 02-Error-Handling-and-Retry.md
├── 03-Logging-and-Observability.md
├── 04-Security.md
├── 05-Performance.md
├── 06-Concurrency.md
├── 07-Configuration-Management.md
└── 08-Production-Best-Practices.md
```

## `01-Production-Scripting.md` completed.
