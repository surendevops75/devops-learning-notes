# Logging and Observability

## 1. Introduction

Production Python automation must be observable.

A script that only prints:

```text
Started
Failed
Done
```

is difficult to operate in real environments.

Production automation should make it possible to answer:

```text
What happened?
When did it happen?
Which environment?
Which AWS account?
Which EKS cluster?
Which namespace?
Which service?
Which Git commit?
Which operation?
How long did it take?
How many retries occurred?
Where did it fail?
What was the final state?
```

The production observability model is:

```text
Logs
  +
Metrics
  +
Events
  +
Artifacts
  +
Health/verification
  =
Operational visibility
```

For the user's DevOps stack, Python observability should integrate naturally with:

```text
AWS
EKS
Kubernetes
Jenkins
GitHub Actions
ArgoCD
Prometheus
Grafana
ELK
Git
Terraform
Helm
```

---

# 2. Logging vs Monitoring vs Observability

These terms are related but different.

### Logging

Records individual events:

```text
deployment started
deployment failed
retrying API call
```

### Monitoring

Tracks measurable signals:

```text
failure rate
duration
retry count
CPU
memory
```

### Observability

Helps determine internal system state from available outputs:

```text
logs
metrics
events
traces where appropriate
```

For DevOps automation:

```text
logs -> detailed evidence
metrics -> trends and alerts
events -> important state changes
```

---

# 3. Why `print()` Is Not Enough

Bad production pattern:

```python
print("Deployment started")
print("Deployment failed")
```

Problems:

```text
no severity
no timestamp control
no structured fields
harder filtering
harder aggregation
poor CI integration
poor ELK ingestion
no consistent format
```

Use Python's `logging` module.

---

# 4. Python Logging Module

Basic:

```python
import logging

logger = logging.getLogger(__name__)

logger.info("Deployment started")
logger.warning("Retrying API request")
logger.error("Deployment failed")
```

---

# 5. Logging Levels

Common levels:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

Typical DevOps usage:

```text
DEBUG    -> detailed troubleshooting
INFO     -> normal workflow
WARNING  -> recoverable/degraded condition
ERROR    -> operation failed
CRITICAL -> severe system-level failure
```

Do not use `ERROR` for every unusual event.

---

# 6. DEBUG

Use DEBUG for detailed diagnostics:

```python
logger.debug(
    "Resolved deployment image=%s",
    image
)
```

Examples:

```text
API request parameters excluding secrets
resource IDs
decision branches
retry calculations
polling state
```

DEBUG should normally be disabled in normal production output unless needed.

---

# 7. INFO

INFO should describe normal operational milestones:

```text
Starting deployment
Validated target
Connected to EKS
Updated manifest
ArgoCD sync requested
Deployment verified
```

---

# 8. WARNING

WARNING indicates something abnormal but not necessarily a failed operation:

```text
retrying after timeout
resource already exists
cleanup delayed
API rate limit encountered
optional dependency unavailable
```

---

# 9. ERROR

ERROR means an operation failed:

```text
Terraform execution failed
deployment verification failed
Git push failed
AWS API operation failed
```

---

# 10. CRITICAL

Use sparingly.

Examples:

```text
configuration corruption
unsafe production target detected
required control-plane dependency unavailable
automation cannot establish a safe execution context
```

A normal deployment failure does not automatically need CRITICAL.

---

# 11. Basic Production Logger

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format=(
        "%(asctime)s %(levelname)s "
        "%(name)s %(message)s"
    ),
)

logger = logging.getLogger("devops-automation")
```

Then:

```python
logger.info("Starting deployment")
```

---

# 12. Avoid Root Logger Abuse

Prefer:

```python
logger = logging.getLogger(__name__)
```

instead of configuring and using the root logger everywhere.

This allows modules to participate in one application-wide logging configuration.

---

# 13. Logger Hierarchy

Example:

```text
devops_automation
├── aws
├── kubernetes
├── git
├── terraform
├── argocd
└── deployment
```

Python loggers can inherit configuration from parent loggers.

---

# 14. Centralized Logging Configuration

Application code should normally do:

```python
logger = logging.getLogger(__name__)
```

The entry point should configure handlers and formatting.

This keeps business logic separate from logging setup.

---

# 15. Handler

A handler determines where logs go.

Common destinations:

```text
stdout
stderr
file
rotating file
external logging system
```

For containerized DevOps automation, stdout/stderr is often preferred.

---

# 16. Why Container Logs Should Go to stdout

In Kubernetes:

```text
Python application
      |
      v
stdout/stderr
      |
      v
container runtime
      |
      v
logging pipeline
      |
      v
ELK
```

This avoids managing application log files inside containers.

---

# 17. File Logging

For long-running non-containerized services, file logging can be appropriate.

Use rotation:

```python
from logging.handlers import RotatingFileHandler
```

Otherwise logs can consume the disk.

---

# 18. Rotating Logs

Example:

```python
handler = RotatingFileHandler(
    "automation.log",
    maxBytes=10 * 1024 * 1024,
    backupCount=5,
)
```

This limits individual file size.

---

# 19. Time-Based Rotation

Another option:

```python
from logging.handlers import TimedRotatingFileHandler

handler = TimedRotatingFileHandler(
    "automation.log",
    when="midnight",
    backupCount=7,
)
```

Useful for daily operational logs.

---

# 20. Production Recommendation

For:

```text
Docker
Kubernetes
EKS
Jenkins agents
GitHub Actions
```

prefer:

```text
stdout/stderr
```

and let the platform collect logs.

For:

```text
long-running VM services
```

file rotation may be appropriate when required.

---

# 21. Log Format

A useful text format:

```text
timestamp level logger message
```

Example:

```text
2026-08-18T10:15:22Z INFO deployment Deployment started
```

For centralized systems, structured JSON is often better.

---

# 22. Structured Logging

Instead of:

```text
Deployment orders failed
```

produce fields such as:

```json
{
  "level": "ERROR",
  "event": "deployment_failed",
  "service": "orders",
  "environment": "staging",
  "run_id": "7812"
}
```

Structured fields make ELK queries easier.

---

# 23. Why JSON Logs Help ELK

With structured logs:

```text
environment=production
service=orders
level=ERROR
```

Kibana can filter fields directly.

With unstructured messages:

```text
"orders deployment failed in production"
```

parsing becomes harder and less reliable.

---

# 24. Recommended Event Fields

Useful fields:

```text
timestamp
level
event
logger
run_id
environment
service
operation
status
duration_ms
attempt
error_type
error_code
```

Only include fields that are actually useful.

---

# 25. Environment Field

Always make environment explicit when possible:

```text
environment=dev
environment=staging
environment=production
```

This reduces ambiguity during incidents.

---

# 26. Run ID

Generate a unique identifier for each automation execution.

Example:

```python
import uuid

run_id = str(uuid.uuid4())
```

Use it throughout the workflow.

---

# 27. Why Run IDs Matter

Suppose Jenkins runs:

```text
100 deployments
```

at the same time.

A run ID lets you correlate:

```text
Python logs
AWS operations
Kubernetes events
ArgoCD
CI job
ELK
```

for one execution.

---

# 28. Git Commit SHA

Include:

```text
commit_sha
```

when the automation is deployment-related.

Then you can correlate:

```text
Git commit
 ↓
CI run
 ↓
Python automation
 ↓
ArgoCD
 ↓
Kubernetes
```

---

# 29. Build Number

For Jenkins:

```text
BUILD_NUMBER
BUILD_URL
JOB_NAME
```

For GitHub Actions:

```text
GITHUB_RUN_ID
GITHUB_SHA
GITHUB_WORKFLOW
```

Capture these safely when available.

---

# 30. Correlation Context

A deployment log could contain:

```text
run_id
commit_sha
environment
service
cluster
namespace
```

This creates a common correlation key across systems.

---

# 31. Logger Adapter

Python provides `LoggerAdapter` for contextual logging.

Example:

```python
adapter = logging.LoggerAdapter(
    logger,
    {
        "run_id": run_id,
        "environment": environment,
    },
)

adapter.info("Deployment started")
```

---

# 32. Contextual Logging

Instead of repeatedly doing:

```python
logger.info(
    "run_id=%s environment=%s deployment started",
    run_id,
    environment,
)
```

a context-aware logger can automatically add those fields.

---

# 33. Context Variables

For larger applications, `contextvars` can carry execution context.

Useful for:

```text
run ID
request ID
environment
```

especially when asynchronous code is involved.

---

# 34. Logging Context in Concurrency

When multiple tasks run concurrently:

```text
task A
task B
task C
```

logs must identify the task/resource.

At minimum include:

```text
run_id
resource
operation
```

Otherwise logs become difficult to correlate.

---

# 35. Event-Based Logging

Prefer meaningful events:

```text
deployment_started
deployment_verified
deployment_failed
retry_started
cleanup_started
cleanup_failed
```

rather than logging every line of code execution.

---

# 36. Event Naming

Use stable names:

```text
deployment_started
deployment_failed
deployment_completed
api_request_failed
retry_attempt
verification_failed
```

Avoid inconsistent names such as:

```text
deployStart
deploymentStart
startedDeployment
```

---

# 37. Logging State Transitions

Useful:

```text
state=STARTING
state=DEPLOYING
state=VERIFYING
state=SUCCESS
```

This can make workflows easier to understand.

---

# 38. Don't Log Everything

Excessive logging causes:

```text
noise
storage cost
slower searching
larger ELK index
```

Log operationally meaningful information.

---

# 39. Don't Log Too Little

If the only log is:

```text
failed
```

operators cannot diagnose the problem.

The right level is:

```text
enough context to reconstruct the operation
```

without dumping sensitive or irrelevant data.

---

# 40. Secrets in Logs

Never log:

```text
AWS secret access keys
passwords
tokens
private keys
Kubernetes Secret values
Authorization headers
database passwords
```

---

# 41. Secret Redaction

Bad:

```python
logger.debug("headers=%s", headers)
```

if headers contain:

```text
Authorization
```

Better:

```python
safe_headers = {
    "Content-Type": headers.get("Content-Type")
}
logger.debug("headers=%s", safe_headers)
```

Prefer excluding secrets entirely rather than relying only on redaction.

---

# 42. Kubernetes Secrets

Never do:

```bash
kubectl get secret my-secret -o yaml
```

and send the output directly to logs.

Secret manifests can contain encoded sensitive values.

---

# 43. AWS Credentials

Never log:

```python
session.get_credentials()
```

or credential objects.

Log identity metadata instead:

```text
AWS account
AWS region
role name
```

when safe.

---

# 44. Safe AWS Identity Logging

Useful:

```text
account_id
region
role_arn
```

Avoid:

```text
secret access key
session token
```

---

# 45. API Tokens

Never log:

```text
Authorization: Bearer <token>
```

Use:

```text
Authorization: [REDACTED]
```

or omit the header.

---

# 46. Exception Leakage

Third-party exceptions can contain request details.

Do not assume:

```python
logger.exception(...)
```

is automatically safe.

Review the exception content and logging destination.

---

# 47. Log Injection

External input can contain newline characters or misleading content.

For example:

```text
username = "admin\nERROR fake event"
```

Structured logging reduces this risk.

Validate or encode untrusted values before rendering them into plain-text logs.

---

# 48. Structured JSON Logging

A simple formatter can emit JSON.

Concept:

```python
import json
import logging


class JsonFormatter(logging.Formatter):
    def format(self, record):
        data = {
            "timestamp": self.formatTime(record),
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
        }
        return json.dumps(data)
```

Production implementations should handle:

```text
exception information
structured fields
timestamps
serialization failures
```

---

# 49. Structured Fields

Use:

```text
event
run_id
environment
service
operation
status
```

instead of encoding everything inside the message string.

---

# 50. Logging Event Example

```json
{
  "timestamp": "2026-08-18T10:15:22Z",
  "level": "INFO",
  "event": "deployment_started",
  "run_id": "7812",
  "environment": "staging",
  "service": "orders",
  "commit_sha": "a1b2c3d"
}
```

---

# 51. Error Event Example

```json
{
  "timestamp": "2026-08-18T10:16:14Z",
  "level": "ERROR",
  "event": "deployment_failed",
  "run_id": "7812",
  "environment": "staging",
  "service": "orders",
  "error_type": "TimeoutError",
  "error_code": "TIMEOUT_001",
  "attempt": 3
}
```

---

# 52. Duration Logging

Measure important operations:

```python
import time

start = time.monotonic()

deploy()

duration = time.monotonic() - start

logger.info(
    "Deployment completed duration_seconds=%.2f",
    duration,
)
```

---

# 53. Why Duration Matters

Duration helps detect:

```text
slow AWS APIs
slow Terraform
slow Helm deployments
slow ArgoCD sync
slow Kubernetes rollout
```

and trends over time.

---

# 54. Use Monotonic Clock for Duration

Correct:

```python
start = time.monotonic()
```

not:

```python
start = time.time()
```

for elapsed duration.

Wall-clock time can change.

---

# 55. Timing Context Manager

Reusable pattern:

```python
from contextlib import contextmanager
import time


@contextmanager
def measure(operation):
    start = time.monotonic()

    try:
        yield
    finally:
        duration = time.monotonic() - start
        logger.info(
            "%s duration_seconds=%.2f",
            operation,
            duration,
        )
```

---

# 56. Logging Start and End

For important operations:

```text
deployment_started
deployment_completed
```

If failure occurs:

```text
deployment_failed
```

Include duration when possible.

---

# 57. Metrics

Logs answer:

```text
What happened in this run?
```

Metrics answer:

```text
How often is this happening?
```

Examples:

```text
deployment_success_total
deployment_failure_total
deployment_duration_seconds
retry_total
```

---

# 58. Counters

Counters represent cumulative events:

```text
automation_runs_total
automation_failures_total
retry_total
```

Conceptually:

```python
counter.inc()
```

---

# 59. Gauges

Gauges represent current values:

```text
active_jobs
queue_depth
running_deployments
```

A gauge can increase and decrease.

---

# 60. Histograms

Histograms are useful for distributions:

```text
deployment duration
API latency
Terraform execution time
```

They allow:

```text
P50
P95
P99
```

analysis depending on the metrics system.

---

# 61. Prometheus Integration

For long-running Python automation services, the Prometheus Python client can expose metrics.

Typical architecture:

```text
Python service
      |
      v
/metrics
      |
      v
Prometheus
      |
      v
Grafana
```

For short-lived batch jobs, pushing metrics requires additional architecture and should be designed carefully.

---

# 62. Prometheus Counter Example

Concept:

```python
from prometheus_client import Counter

runs = Counter(
    "automation_runs_total",
    "Total automation runs",
)

runs.inc()
```

---

# 63. Prometheus Histogram Example

```python
from prometheus_client import Histogram

duration = Histogram(
    "deployment_duration_seconds",
    "Deployment duration",
)

with duration.time():
    deploy()
```

---

# 64. Labels

Useful labels:

```text
environment
service
operation
status
```

But use labels carefully.

---

# 65. High Cardinality

Avoid labels containing:

```text
run_id
request_id
user ID
full URL
commit SHA
pod name
timestamp
```

if those values create huge numbers of unique time series.

High cardinality can cause Prometheus performance and storage problems.

---

# 66. Good Prometheus Labels

Reasonable:

```text
environment=production
service=orders
operation=deployment
```

Potentially dangerous:

```text
run_id=7812
```

because every run creates a new label value.

Put unique IDs in logs, not metrics.

---

# 67. Logs + Metrics

Use both:

```text
Prometheus:
"deployment failure rate increased"

ELK:
"which deployment failed and why?"
```

This is a strong DevOps observability pattern.

---

# 68. Grafana

Grafana can combine:

```text
Prometheus metrics
ELK logs
```

into dashboards.

Example dashboard:

```text
Automation Success Rate
Retry Rate
P95 Deployment Duration
Top Failure Categories
Recent Errors
```

---

# 69. ELK Integration

Typical flow:

```text
Python
  |
  v
stdout / JSON logs
  |
  v
Log collection
  |
  v
Logstash
  |
  v
Elasticsearch
  |
  v
Kibana
```

---

# 70. JSON + ELK

Structured JSON makes fields such as:

```text
environment
service
operation
error_code
run_id
```

easy to index and search.

---

# 71. Kibana Search

Useful operational query concepts:

```text
environment:production
AND level:ERROR
```

or:

```text
service:orders
AND event:deployment_failed
```

The exact query syntax depends on the Kibana/Elasticsearch version and configured fields.

---

# 72. ELK Index Strategy

Avoid creating an index per:

```text
run
service instance
pod
```

without a retention strategy.

A common pattern is time-based indexing/data streams with lifecycle management.

---

# 73. Log Retention

Define:

```text
retention period
storage tier
deletion policy
```

based on:

```text
incident requirements
compliance
cost
operational value
```

---

# 74. Log Rotation

If writing files:

```text
rotate
compress
retain
delete
```

Do not allow automation logs to fill the disk.

---

# 75. Disk-Full Failure

A common production failure:

```text
logs grow
 ↓
disk fills
 ↓
application cannot write
 ↓
automation fails
```

Use:

```text
rotation
retention
centralized collection
disk monitoring
```

---

# 76. Logging Performance

Logging itself has cost.

Avoid expensive operations when DEBUG is disabled.

Instead of:

```python
logger.debug(f"Large object: {expensive_function()}")
```

use lazy logging where possible:

```python
logger.debug(
    "Object: %s",
    object_value,
)
```

For truly expensive calculations, guard them explicitly.

---

# 77. Logging Large Payloads

Do not log:

```text
entire Kubernetes manifests
entire Terraform state
large API responses
large application logs
```

unless specifically needed for diagnostics.

Prefer:

```text
resource name
status
version
hash
summary
```

---

# 78. Logging API Requests

Useful:

```text
method
host/service
endpoint category
status code
duration
attempt
```

Avoid:

```text
authorization
cookies
passwords
secret query parameters
```

---

# 79. Correlation ID

A correlation ID ties multiple events to one workflow.

Example:

```text
run_id=7812
```

appears in:

```text
Python
Jenkins
ArgoCD
Kubernetes diagnostics
```

where supported.

---

# 80. Trace ID

If distributed tracing is used, a trace ID can connect operations across services.

For the user's current observability stack, the primary focus remains:

```text
Prometheus
Grafana
ELK
```

Tracing is optional and should not be introduced unnecessarily into a simple automation script.

---

# 81. Health Checks

Long-running Python services should expose health endpoints where appropriate:

```text
/health
/ready
```

Example:

```text
/health -> process is alive
/ready  -> dependencies are ready
```

---

# 82. Liveness vs Readiness

Liveness:

```text
Should the process be restarted?
```

Readiness:

```text
Should this process receive work?
```

Do not make liveness depend on every external dependency unless restart behavior is genuinely desired.

---

# 83. Kubernetes Health Architecture

```text
Kubernetes
    |
    +--> liveness probe
    |
    +--> readiness probe
    |
    v
Python automation service
    |
    +--> dependency checks
```

---

# 84. Health Endpoint Example

For a service framework, conceptually:

```python
@app.get("/health")
def health():
    return {"status": "ok"}
```

Readiness can check only critical dependencies:

```python
@app.get("/ready")
def ready():
    if not dependency_ready():
        raise HTTPException(status_code=503)
    return {"status": "ready"}
```

Use the framework appropriate to the service architecture.

---

# 85. Do Not Overload Health Checks

Bad readiness endpoint:

```text
check every AWS resource
check every Kubernetes resource
check every Git repository
check ELK
check Prometheus
```

A health check should be:

```text
fast
predictable
low cost
focused
```

---

# 86. Error Metrics

A production automation service should expose:

```text
automation_runs_total
automation_failures_total
automation_retries_total
automation_timeouts_total
```

Potentially:

```text
automation_duration_seconds
```

as a histogram.

---

# 87. Business/Operational Metrics

Depending on the automation:

```text
deployments_total
successful_deployments_total
rollbacks_total
terraform_applies_total
terraform_failures_total
argocd_sync_failures_total
```

Keep metric naming consistent.

---

# 88. Metric Naming

Prefer clear names:

```text
automation_runs_total
```

rather than:

```text
runs
```

Prometheus conventions generally use descriptive names and `_total` for counters.

---

# 89. Metric Labels

Good:

```text
environment
service
operation
status
```

Bad high-cardinality labels:

```text
run_id
commit_sha
request_id
pod_uid
```

Use logs for those identifiers.

---

# 90. Alerting

Good alerts identify meaningful conditions:

```text
automation failure rate > threshold
deployment verification failures increasing
P95 deployment duration exceeds threshold
retry rate abnormally high
cleanup failures increasing
```

Avoid alerting on every retry.

---

# 91. Alert Severity

Example:

```text
WARNING:
retry rate elevated

CRITICAL:
production deployment automation failing consistently
```

Severity should reflect operational impact.

---

# 92. Alert Context

An alert should provide:

```text
environment
service
operation
time window
failure rate
dashboard link
run/log correlation
```

Do not make operators search blindly.

---

# 93. Observability Architecture

```text
                   Python Automation
                         |
          +--------------+--------------+
          |              |              |
          v              v              v
        Logs           Metrics        Events
          |              |              |
          v              v              v
         ELK         Prometheus       CI/Systems
          |              |
          v              v
       Kibana          Grafana
          \              /
           \            /
            +----------+
                 |
                 v
             Operator
```

---

# 94. DevOps Deployment Architecture

```text
Git
 |
 v
Jenkins / GitHub Actions
 |
 v
Python Automation
 |
 +--> AWS
 |
 +--> EKS
 |
 +--> Terraform
 |
 +--> Helm
 |
 +--> ArgoCD
 |
 +--> Git
 |
 +--> Prometheus metrics
 |
 +--> ELK logs
 |
 v
Grafana / Kibana
```

---

# 95. Production Logging Configuration

A production configuration should centralize:

```text
level
handlers
format
JSON/text formatter
stdout/stderr destination
file rotation where needed
```

Do not configure logging separately in every module.

---

# 96. `dictConfig`

Python supports `logging.config.dictConfig`.

Concept:

```python
from logging.config import dictConfig

dictConfig({
    "version": 1,
    "disable_existing_loggers": False,
    "formatters": {
        "standard": {
            "format": (
                "%(asctime)s %(levelname)s "
                "%(name)s %(message)s"
            )
        }
    },
    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "formatter": "standard",
        }
    },
    "root": {
        "level": "INFO",
        "handlers": ["console"],
    },
})
```

For containerized environments, console output is usually preferable.

---

# 97. Environment-Based Log Level

Example:

```text
DEV        -> DEBUG
STAGING    -> INFO
PRODUCTION -> INFO
```

Do not enable verbose DEBUG globally in production unless necessary.

---

# 98. Dynamic Log Level

A long-running service may support changing log levels without redeploying.

If implemented, protect the control mechanism and audit changes.

For simple automation scripts, environment-based configuration is often enough.

---

# 99. Logging in CI/CD

CI logs should make major stages obvious:

```text
[VALIDATE]
[BUILD]
[SECURITY]
[DEPLOY]
[VERIFY]
[CLEANUP]
```

Python can emit equivalent events.

---

# 100. Jenkins Example

Useful context:

```text
job_name
build_number
build_url
git_commit
environment
```

Do not log credentials injected into the job.

---

# 101. GitHub Actions Example

Useful context:

```text
workflow
run_id
sha
repository
environment
```

GitHub-provided environment variables should be handled as metadata, not blindly dumped.

---

# 102. ArgoCD Correlation

When deploying through GitOps, correlate:

```text
commit SHA
ArgoCD application
Python run ID
Kubernetes namespace
deployment version
```

This dramatically improves incident investigation.

---

# 103. Kubernetes Events

When a deployment fails, collect safe diagnostic evidence:

```bash
kubectl describe deployment <name>
kubectl get pods
kubectl describe pod <pod>
kubectl get events
```

Python automation can invoke or use the Kubernetes client to gather summaries.

Do not automatically dump Kubernetes Secrets.

---

# 104. ELK Troubleshooting Flow

Example:

```text
Deployment failed
      |
      v
Grafana -> failure spike
      |
      v
Kibana -> search run_id
      |
      v
Find deployment_failed event
      |
      v
Find retry/timeout events
      |
      v
Correlate Kubernetes events
      |
      v
Root cause
```

---

# 105. Prometheus Troubleshooting Flow

Example:

```text
Deployment duration increased
      |
      v
Prometheus
      |
      v
P95 latency
      |
      v
Grafana
      |
      v
Identify service/environment
      |
      v
Kibana
      |
      v
Inspect corresponding logs
```

---

# 106. Metrics Should Not Replace Logs

Metric:

```text
deployment_failure_total = 17
```

does not tell you:

```text
which run
which pod
which error
which commit
```

Use logs for details.

---

# 107. Logs Should Not Replace Metrics

If you only have logs, calculating:

```text
failure rate
P95 duration
retry rate
```

becomes more expensive and less reliable.

Use metrics for aggregation.

---

# 108. Event Logs vs Audit Logs

Operational event:

```text
deployment_started
```

Audit event:

```text
who changed production
what changed
when
from where
```

For production automation, audit requirements should be considered separately from application logs.

---

# 109. Auditability

For deployment automation, retain:

```text
actor/automation identity
Git SHA
environment
target
change
timestamp
result
approval/reference when required
```

Never record credentials as audit data.

---

# 110. Log Sampling

High-volume services may use sampling for repetitive DEBUG events.

For deployment automation, keep important state transitions and failures unsampled.

---

# 111. Sensitive Data Classification

Before logging a field ask:

```text
Is it needed for diagnosis?
Is it sensitive?
Is it high-cardinality?
Could it expose infrastructure details?
```

If the answer is unnecessary:

```text
do not log it
```

---

# 112. Performance of JSON Logging

Structured logging adds serialization overhead.

For normal DevOps automation volumes, this is usually acceptable.

For high-throughput services:

```text
benchmark
buffer where appropriate
avoid giant payloads
```

---

# 113. Log Backpressure

A logging destination can become slow.

A long-running service should consider:

```text
queue-based logging
bounded buffers
drop policy for low-priority logs
```

depending on scale.

Never allow DEBUG logging to bring down the application.

---

# 114. Async Logging

For high-throughput applications, `QueueHandler` and `QueueListener` can move formatting/I/O away from the main execution path.

Concept:

```text
application
   |
QueueHandler
   |
queue
   |
QueueListener
   |
stdout/file
```

This is usually unnecessary for small one-shot DevOps scripts.

---

# 115. Exception Tracebacks

Use:

```python
logger.exception("Deployment failed")
```

inside an exception handler.

This records the traceback.

For expected operational errors, a concise structured error may be preferable when the traceback adds no value.

---

# 116. Error Message Design

Bad:

```text
Something went wrong.
```

Good:

```text
event=deployment_failed
environment=staging
service=orders
category=TIMEOUT
duration_seconds=600
```

The message and structured fields should complement each other.

---

# 117. Log Correlation Across Tools

Strong production correlation:

```text
Git SHA
   |
   v
Jenkins Build / GitHub Run
   |
   v
Python run_id
   |
   v
ArgoCD Application
   |
   v
Kubernetes Deployment
   |
   v
ELK logs
```

This is more useful than isolated tool-specific logs.

---

# 118. Production Observability Example

Suppose deployment duration increases from:

```text
P95 = 180s
```

to:

```text
P95 = 600s
```

Prometheus/Grafana identifies the trend.

Then:

```text
Kibana
```

can find:

```text
deployment_timeout
pod_pending
image_pull
API timeout
```

The systems complement each other.

---

# 119. Production Failure Example

```text
Python deployment
       |
       v
HTTP timeout
       |
       +--> log retry
       |
       +--> retry metric
       |
       v
second attempt succeeds
       |
       v
verification succeeds
```

The final run succeeds, but:

```text
retry_total
```

still records degradation.

---

# 120. Production Failure Example — Repeated Failure

```text
Python deployment
       |
       v
timeout
       |
       v
retry
       |
       v
timeout
       |
       v
retry
       |
       v
failure
```

Output:

```text
ERROR deployment_failed
category=TIMEOUT
attempts=3
```

Prometheus increments failure metrics.

ELK retains the detailed logs.

---

# 121. Production Failure Example — Wrong Cluster

```text
expected_cluster=staging-eks
actual_cluster=prod-eks
```

Correct result:

```text
CRITICAL/ERROR
deployment blocked
```

No retries.

The safety guard itself should produce a high-value event.

---

# 122. Production Failure Example — Secret Exposure Risk

If an API call fails:

```text
HTTP 401
```

log:

```text
authentication_failed
```

not:

```text
token=<secret>
```

---

# 123. Production Observability Checklist

```text
[ ] Python logging module
[ ] Appropriate levels
[ ] Central configuration
[ ] stdout/stderr for containers
[ ] log rotation where files are used
[ ] structured JSON where appropriate
[ ] run ID
[ ] environment
[ ] service
[ ] operation
[ ] Git SHA
[ ] CI build/run ID
[ ] duration
[ ] attempt
[ ] error category
[ ] secret-safe logging
[ ] Prometheus metrics
[ ] Grafana dashboards
[ ] ELK integration
[ ] log retention
[ ] alerting
[ ] correlation across systems
```

---

# 124. Prometheus Checklist

```text
[ ] Counters for runs/failures/retries
[ ] Histograms for duration
[ ] Low-cardinality labels
[ ] No run IDs as labels
[ ] Alert thresholds
[ ] Grafana dashboards
[ ] Metric naming conventions
[ ] Retention appropriate for workload
```

---

# 125. ELK Checklist

```text
[ ] JSON logs
[ ] Stable event names
[ ] Consistent fields
[ ] Run ID
[ ] Environment
[ ] Service
[ ] Error code
[ ] Timestamp
[ ] Secret filtering
[ ] Index/data-stream strategy
[ ] Retention
[ ] Kibana dashboards
```

---

# 126. Kubernetes Checklist

```text
[ ] stdout/stderr logging
[ ] structured application logs
[ ] liveness probe where required
[ ] readiness probe where required
[ ] resource limits
[ ] log volume consideration
[ ] ELK collection
[ ] Kubernetes events
[ ] safe diagnostic collection
```

---

# 127. CI/CD Checklist

```text
[ ] stage names
[ ] run/build ID
[ ] Git SHA
[ ] environment
[ ] exit code
[ ] failure event
[ ] artifacts
[ ] retry metrics
[ ] deployment duration
[ ] links/correlation identifiers
```

---

# 128. Logging Anti-Patterns

Avoid:

```text
print() everywhere
logging secrets
logging huge API responses
logging full Kubernetes Secrets
logging every loop iteration
no timestamps
inconsistent event names
high-cardinality metrics
DEBUG permanently enabled
logs without environment
logs without operation context
```

---

# 129. Observability Anti-Patterns

Avoid:

```text
metrics with unbounded labels
alerts for every warning
no retention policy
no dashboard
logs without correlation IDs
only logs, no metrics
only metrics, no diagnostic logs
health checks that perform expensive workflows
```

---

# 130. Senior Interview — Why Use Logging Instead of print()?

Strong answer:

> I use Python's logging framework because it provides levels, handlers, formatting, centralized configuration and integration with log collection systems. In containerized environments I can write structured logs to stdout and let the platform collect them into ELK. `print()` does not provide the same operational controls.

---

# 131. Senior Interview — What Should You Log?

Strong answer:

> I log meaningful workflow events such as start, completion, retries, failures and verification results. I include safe context like environment, service, operation, run ID, commit SHA and duration. I avoid secrets, large payloads and unnecessary high-volume DEBUG output.

---

# 132. Senior Interview — Why Structured Logging?

Strong answer:

> Structured logging represents important information as fields instead of embedding everything inside a message. This makes ELK/Kibana filtering and aggregation much easier. It also makes logs more consistent across automation components.

---

# 133. Senior Interview — How Do You Correlate a Deployment Across Tools?

Strong answer:

> I use a run or correlation ID and combine it with the Git SHA, CI build/run ID, environment, service, ArgoCD application and Kubernetes target. The ID is placed in Python logs and operational events, while high-cardinality identifiers remain in logs rather than Prometheus labels.

---

# 134. Senior Interview — What Metrics Would You Expose?

Strong answer:

> For automation I would expose total runs, failures, retries and timeouts as counters, and operation duration as a histogram. I would use low-cardinality labels such as environment, service and operation. I would not use run IDs or commit SHAs as Prometheus labels.

---

# 135. Senior Interview — Why Not Put Run ID in Prometheus Labels?

Strong answer:

> Run IDs are high-cardinality and continuously create new time series. That can increase memory and storage usage significantly. Run IDs belong in logs and events, while metrics should use stable low-cardinality dimensions.

---

# 136. Senior Interview — How Do Logs and Metrics Work Together?

Strong answer:

> Metrics tell me that something is wrong and show the trend, such as an increase in deployment failures or P95 duration. Logs provide the detailed evidence for a specific run, such as the exception, Kubernetes resource, commit SHA and retry history. I use Grafana for metrics and Kibana/ELK for detailed log investigation.

---

# 137. Senior Interview — How Would You Monitor Python Automation in Kubernetes?

Strong answer:

> I would write structured logs to stdout/stderr, collect them through the cluster logging pipeline into ELK, expose Prometheus metrics for a long-running service, create Grafana dashboards and alerts, and configure liveness/readiness probes when the application is a service. For batch jobs I would also retain CI/Kubernetes job results and logs.

---

# 138. Senior Interview — How Do You Handle Secrets in Logs?

Strong answer:

> I prevent secrets from entering log statements rather than depending only on redaction. I never log passwords, tokens, private keys, Kubernetes Secret values or authorization headers. For AWS I log safe identity metadata such as account and region where appropriate.

---

# 139. Senior Interview — What Is the Difference Between Logging and Monitoring?

Strong answer:

> Logging records detailed individual events. Monitoring uses metrics and thresholds to identify changes or failures over time. Observability combines these signals to help us understand system behavior and diagnose problems.

---

# 140. Senior Interview — What Is a Good Production Log?

Strong answer:

> A good production log is structured, searchable and actionable. It identifies the event, environment, operation, resource and correlation/run ID, includes the relevant status or error category, and avoids sensitive information. It should help an operator understand what happened without reading source code.

---

# 141. Senior Interview — Why Use stdout in Kubernetes?

Strong answer:

> Containers are normally treated as ephemeral workloads, so application-managed log files are inconvenient. Writing to stdout/stderr lets the container runtime and cluster logging pipeline collect the logs. Those logs can then be shipped to ELK without requiring the application to manage local log files.

---

# 142. Senior Interview — How Do You Prevent Log Files Filling a VM?

Strong answer:

> If file logging is required, I use rotation based on size or time, retention limits, compression where appropriate and disk monitoring. For containerized workloads I prefer stdout/stderr with centralized collection, which avoids managing application log files inside containers.

---

# 143. Senior Interview — How Would You Debug a Failed Deployment?

Strong answer:

```text
1. Check Grafana for failure/duration trends.
2. Find the CI run/build.
3. Search ELK using run ID, service and commit SHA.
4. Inspect Python deployment events.
5. Check retry/timeout history.
6. Inspect Kubernetes deployment/pods/events.
7. Check ArgoCD sync and health if GitOps is involved.
8. Correlate the failure with the Git SHA.
9. Identify root cause.
10. Record the final result and recovery.
```

---

# 144. Senior Interview — What Is High Cardinality?

Strong answer:

> High cardinality means a metric label has many unique values. Using run IDs, request IDs, pod UIDs or commit SHAs as Prometheus labels can create huge numbers of time series. I keep those identifiers in logs and use stable labels such as environment, service and operation for metrics.

---

# 145. Senior Interview — What Should a Deployment Dashboard Show?

Strong answer:

```text
deployment success rate
deployment failure rate
P95/P99 duration
retry rate
timeout rate
rollback count
failure categories
environment
service
```

Then I use ELK/Kibana to investigate individual failures.

---

# 146. Senior Interview — Why Measure Duration?

Strong answer:

> Duration reveals degradation even before failures occur. A deployment can still be successful while moving from 2 minutes to 10 minutes. Histograms let us monitor P50/P95/P99 behavior and alert on meaningful latency changes.

---

# 147. Senior Interview — What Should a Health Endpoint Do?

Strong answer:

> A liveness endpoint should indicate whether the process is alive. A readiness endpoint should indicate whether the service is ready to receive work. Both should be fast and predictable. I avoid making liveness depend on every external dependency because that can cause unnecessary restart loops.

---

# 148. Senior Interview — Logs vs Metrics

Strong answer:

```text
Metrics:
"What is happening at scale?"

Logs:
"What happened in this specific operation?"

Example:

Prometheus:
deployment_failure_rate increased

ELK:
run_id=7812
service=orders
error=ImagePullBackOff
commit_sha=a1b2c3d
```

---

# 149. Production Observability Architecture

```text
                         Git
                          |
                          v
                  Jenkins / GitHub Actions
                          |
                          v
                   Python Automation
                          |
       +------------------+------------------+
       |                  |                  |
       v                  v                  v
      AWS                EKS             GitOps/ArgoCD
       |                  |                  |
       +------------------+------------------+
                          |
                 +--------+--------+
                 |                 |
                 v                 v
              Logs             Metrics
                 |                 |
                 v                 v
                ELK           Prometheus
                 |                 |
                 v                 v
              Kibana            Grafana
                 \                 /
                  \               /
                   +-------------+
                         |
                         v
                      Operator
```

---

# 150. Production Logging Flow

```text
Python
  |
  | structured JSON
  v
stdout
  |
  v
Kubernetes/container runtime
  |
  v
log collector
  |
  v
Logstash
  |
  v
Elasticsearch
  |
  v
Kibana
```

For VM-based services:

```text
Python
  |
  v
rotating log file / stdout
  |
  v
collector
  |
  v
ELK
```

---

# 151. Production Metrics Flow

```text
Python service
     |
     v
/metrics
     |
     v
Prometheus
     |
     v
Grafana
     |
     v
alerts + dashboards
```

For short-lived batch automation, use a collection design appropriate to the job lifecycle instead of assuming a normal scrape endpoint will remain available after the job exits.

---

# 152. Complete DevOps Observability Workflow

```text
Developer
   |
   v
Git commit
   |
   v
CI pipeline
   |
   v
Python automation
   |
   +--> logs --> ELK --> Kibana
   |
   +--> metrics --> Prometheus --> Grafana
   |
   +--> AWS
   |
   +--> EKS
   |
   +--> Terraform
   |
   +--> Helm
   |
   +--> ArgoCD
   |
   v
Verification
   |
   +--> success
   |
   +--> failure
          |
          v
       diagnostics
          |
          v
     logs + metrics
          |
          v
       incident
```

---

# 153. Production Golden Rules

```text
1. Use logging instead of print for production automation.
2. Use appropriate log levels.
3. Centralize logging configuration.
4. Prefer stdout/stderr in containers.
5. Use structured logs for centralized systems.
6. Generate a run/correlation ID.
7. Include safe environment and operation context.
8. Correlate with CI build/run and Git SHA.
9. Never log secrets.
10. Avoid huge payloads.
11. Measure important operation duration.
12. Use Prometheus for aggregate metrics.
13. Use Grafana for dashboards and alerts.
14. Use ELK for detailed searchable logs.
15. Keep Prometheus labels low-cardinality.
16. Put unique IDs in logs, not metric labels.
17. Configure log retention.
18. Rotate VM log files.
19. Keep health checks fast and focused.
20. Make failures observable and actionable.
```

---

# 154. Production Review Questions

Before approving Python automation, ask:

```text
Can I identify one execution with a run ID?

Can I identify the environment?

Can I identify the target service/resource?

Can I correlate the Git SHA?

Can I correlate the CI build/run?

Can I find the failure in ELK?

Can I measure success/failure rate?

Can I measure duration?

Can I measure retry rate?

Are metric labels low-cardinality?

Can an operator diagnose the failure without reading source code?

Could any log expose credentials?

Can logs fill the disk?

Are logs retained appropriately?

Can Grafana show the operational trend?

Can Kibana find the detailed failure?
```

---

# 155. Final Takeaway

Production observability is not:

```text
print("started")
print("failed")
```

It is:

```text
                 AUTOMATION
                     |
        +------------+------------+
        |            |            |
        v            v            v
      LOGS         METRICS       EVENTS
        |            |            |
        v            v            v
       ELK       Prometheus      CI/Systems
        |            |
        v            v
     Kibana        Grafana
        \            /
         \          /
          v        v
        Operator
            |
            v
        Diagnosis
```

The DevOps mindset is:

```text
Logs tell the story.
Metrics show the trend.
Events show important state changes.
Correlation IDs connect the systems.
Dashboards expose patterns.
Alerts bring attention to real problems.
```

A production Python automation system should make failures **searchable, measurable, correlated, and actionable**.

---

# 156. Section Progress

```text
10-Python-Production/
│
├── 01-Production-Scripting.md        ✓
├── 02-Error-Handling-and-Retry.md    ✓
├── 03-Logging-and-Observability.md   ✓
├── 04-Security.md
├── 05-Performance.md
├── 06-Concurrency.md
├── 07-Configuration-Management.md
└── 08-Production-Best-Practices.md
```

Next:

```text
04-Security.md
```
