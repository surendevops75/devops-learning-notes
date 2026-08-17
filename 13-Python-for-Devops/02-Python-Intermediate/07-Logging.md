# Logging

## DevOps Focus

Logging is one of the most important parts of production automation.

A DevOps engineer uses logs to understand:

- why a deployment failed
- why a Kubernetes pod restarted
- why an API returned errors
- why a CI/CD job failed
- why a script stopped
- which resource was modified
- when an incident started
- which step took the most time
- which request caused a failure
- whether an automation action was successful

Python provides the built-in `logging` module for application and automation logs.

> Core principle: **logs should help an engineer answer what happened, when it happened, where it happened, and why it happened.**

---

## 1. Why Use Logging Instead of `print()`

Simple script:

```python
print("Deployment started")
```

This is useful for quick debugging, but production automation needs more.

Logging provides:

```text
log level
timestamp
logger name
structured context
exception information
handlers
formatters
file output
stream output
filtering
```

Basic example:

```python
import logging

logging.basicConfig(
    level=logging.INFO
)

logging.info(
    "Deployment started"
)
```

---

## 2. Import Logging

```python
import logging
```

Create a logger:

```python
logger = logging.getLogger(
    __name__
)
```

Then:

```python
logger.info(
    "Deployment started"
)
```

Using a module-level logger is preferred over repeatedly calling the root logger.

---

## 3. Root Logger vs Named Logger

Root:

```python
logging.info("message")
```

Named:

```python
logger = logging.getLogger(
    __name__
)
```

Named loggers are better for larger applications because logs can identify the module or component producing them.

Example:

```text
deployment
kubernetes
aws
database
security
```

---

## 4. Logging Levels

Python provides:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

Typical meaning:

```text
DEBUG    detailed troubleshooting
INFO     normal operational event
WARNING  unexpected but recoverable condition
ERROR    operation failed
CRITICAL severe failure
```

---

## 5. DEBUG

Use DEBUG for detailed information useful during troubleshooting.

```python
logger.debug(
    "Loaded configuration: %s",
    config,
)
```

Do not log secrets just because the level is DEBUG.

Production DEBUG logging can expose:

```text
tokens
passwords
headers
credentials
sensitive configuration
```

---

## 6. INFO

INFO describes normal operational events.

```python
logger.info(
    "Deployment started"
)

logger.info(
    "Deployment completed successfully"
)
```

Good INFO messages answer:

```text
what happened
which resource
which environment
```

---

## 7. WARNING

WARNING indicates something unexpected but not necessarily fatal.

```python
logger.warning(
    "Backup is older than expected: %s",
    age,
)
```

Do not use WARNING for every normal event.

Too many warnings create alert fatigue.

---

## 8. ERROR

ERROR means an operation failed or a significant problem occurred.

```python
logger.error(
    "Deployment failed"
)
```

Include enough context to investigate.

---

## 9. CRITICAL

CRITICAL indicates severe failure.

```python
logger.critical(
    "Unable to initialize production configuration"
)
```

Use this level sparingly.

If everything is CRITICAL, nothing is meaningfully critical.

---

## 10. Basic Configuration

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format=(
        "%(asctime)s "
        "%(levelname)s "
        "%(name)s "
        "%(message)s"
    ),
)

logger = logging.getLogger(
    __name__
)

logger.info(
    "Application started"
)
```

---

## 11. Log Format

Useful fields:

```text
timestamp
level
logger
message
```

Example:

```text
2026-08-16 10:30:00,123 INFO deployment Deployment started
```

For production systems, you may also want:

```text
process
thread
module
request ID
environment
service
```

---

## 12. `asctime`

```python
"%(asctime)s"
```

adds a formatted timestamp.

Example:

```text
2026-08-16 10:30:00,123
```

For centralized logging, prefer an explicit and consistently documented timestamp format, often UTC.

---

## 13. Log Level in Format

```python
"%(levelname)s"
```

Example:

```text
INFO
WARNING
ERROR
```

This allows downstream systems and engineers to quickly identify severity.

---

## 14. Logger Name

```python
"%(name)s"
```

If:

```python
logger = logging.getLogger(
    __name__
)
```

the name usually identifies the Python module.

This helps locate the source of an event.

---

## 15. Message Formatting

Prefer logging's parameterized style:

```python
logger.info(
    "Deployment %s started",
    deployment_name,
)
```

instead of:

```python
logger.info(
    f"Deployment {deployment_name} started"
)
```

Parameterized logging allows the logging framework to defer string formatting when the message is filtered out.

---

## 16. Multiple Parameters

```python
logger.info(
    "Deploying %s version %s to %s",
    service,
    version,
    environment,
)
```

This produces readable operational logs.

---

## 17. Logging Exceptions

Use:

```python
try:
    deploy()
except Exception:
    logger.exception(
        "Deployment failed"
    )
    raise
```

`logger.exception()` automatically includes traceback information when called inside an exception handler.

---

## 18. `logger.exception()` vs `logger.error()`

Inside an exception handler:

```python
logger.exception(
    "API request failed"
)
```

includes traceback.

This:

```python
logger.error(
    "API request failed"
)
```

does not automatically include the traceback.

Use `logger.exception()` when the traceback is useful.

---

## 19. `exc_info=True`

You can also do:

```python
logger.error(
    "Deployment failed",
    exc_info=True,
)
```

This includes exception information.

`logger.exception()` is the convenient pattern inside `except`.

---

## 20. Avoid Swallowing Exceptions

Bad:

```python
try:
    deploy()
except Exception:
    logger.exception(
        "Deployment failed"
    )
```

The script may continue and eventually exit successfully.

Better:

```python
try:
    deploy()
except Exception:
    logger.exception(
        "Deployment failed"
    )
    raise
```

Logging and error handling are separate responsibilities.

---

## 21. Log and Exit in CLI Scripts

```python
import sys

try:
    deploy()
except Exception:
    logger.exception(
        "Deployment failed"
    )
    sys.exit(1)
```

CI/CD systems can then correctly detect failure.

---

## 22. Logging to a File

```python
logging.basicConfig(
    filename="deployment.log",
    level=logging.INFO,
)
```

This writes logs to a file.

In containers, however, writing to local files may not be the preferred strategy.

---

## 23. Logging to Standard Output

For containers:

```python
import sys

logging.basicConfig(
    stream=sys.stdout,
    level=logging.INFO,
)
```

Container platforms can then collect stdout/stderr.

Typical architecture:

```text
Python
  |
  v
stdout/stderr
  |
  v
container runtime
  |
  v
log collector
  |
  v
ELK / logging platform
```

---

## 24. Why Containers Prefer stdout/stderr

Containers are often ephemeral.

If logs are stored only inside:

```text
/app/logs/application.log
```

they may disappear when the container is replaced.

A better architecture is:

```text
application
   |
   v
stdout/stderr
   |
   v
collector
   |
   v
centralized logging
```

---

## 25. Kubernetes Logging

A Python container can write logs to stdout:

```python
logger.info(
    "Payment service started"
)
```

Then:

```bash
kubectl logs <pod>
```

can retrieve the container logs.

For multiple replicas, centralized collection is required for durable operational analysis.

---

## 26. Kubernetes Multi-Container Pods

If a pod has multiple containers:

```bash
kubectl logs <pod> -c <container>
```

Python logs should identify the service/container context where appropriate.

---

## 27. Logging Architecture in EKS

Typical architecture:

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
log collector
        |
        v
Elasticsearch
        |
        v
Kibana
```

The exact collector can vary.

The key principle is centralized collection.

---

## 28. ELK Integration

Python does not need to know Elasticsearch internals just to produce useful logs.

Application:

```text
structured log
     |
     v
stdout
     |
     v
collector
     |
     v
Logstash / pipeline
     |
     v
Elasticsearch
     |
     v
Kibana
```

Keep application logging independent from the final storage platform.

---

## 29. Structured Logging

Plain:

```text
Deployment failed for payment
```

Structured:

```json
{
  "level": "ERROR",
  "service": "payment",
  "environment": "production",
  "deployment_id": "deploy-123",
  "message": "Deployment failed"
}
```

Structured logs are easier to search and aggregate.

---

## 30. JSON Logging

A JSON log record can contain:

```text
timestamp
level
service
environment
event
request_id
message
```

Example:

```json
{
  "level": "INFO",
  "service": "payment",
  "environment": "production",
  "event": "deployment_started"
}
```

Use a consistent schema across services.

---

## 31. Standard Library vs JSON Logging Library

Python's built-in logging supports formatters and handlers.

For production JSON logs, teams often use a structured logging package or a custom JSON formatter.

The choice depends on:

```text
platform
team standards
logging backend
performance
schema requirements
```

Do not add a dependency when the built-in formatter is sufficient.

---

## 32. Structured Event Names

Prefer:

```text
deployment_started
deployment_completed
deployment_failed
rollback_started
rollback_completed
```

over inconsistent free-text descriptions.

Event names make dashboards and searches easier.

---

## 33. Context Fields

Useful fields:

```text
service
environment
region
cluster
namespace
pod
version
image
request_id
deployment_id
```

Avoid adding sensitive fields.

---

## 34. Correlation ID

A correlation ID connects related operations.

Example:

```text
request_id=abc123
```

A request may generate logs in:

```text
API
payment
inventory
notification
```

The same ID allows engineers to search across the workflow.

---

## 35. Request ID

Example:

```python
logger.info(
    "Processing request",
    extra={
        "request_id": request_id
    },
)
```

The formatter must be designed to render custom fields correctly.

For large systems, structured logging libraries can make this easier.

---

## 36. Deployment ID

Use a unique deployment identifier:

```text
deployment_id=deploy-20260816-001
```

Then logs can be filtered:

```text
deployment_id = deploy-20260816-001
```

This is extremely useful during production rollouts.

---

## 37. Environment Context

Include:

```text
dev
staging
production
```

when it is not already supplied by the logging platform.

Example:

```text
environment=production
```

This helps prevent investigating the wrong environment.

---

## 38. Region Context

For AWS workloads:

```text
region=ap-south-1
```

can be useful.

Also consider:

```text
cluster
account
namespace
service
```

Do not log account credentials.

---

## 39. Kubernetes Context

Useful fields:

```text
cluster
namespace
pod
container
node
deployment
```

Some of these may already be added by the logging collector.

Avoid duplicating huge amounts of metadata unnecessarily.

---

## 40. Log Schema

Define a common schema:

```text
timestamp
level
service
environment
event
message
request_id
```

Then optional fields:

```text
error
deployment_id
version
region
```

Consistency matters more than having hundreds of fields.

---

## 41. Avoid High-Cardinality Fields

Some fields create huge cardinality:

```text
full request body
unique stack trace text
random IDs
large payloads
```

Logs can contain identifiers, but do not blindly turn every unique value into an indexed field in the backend.

---

## 42. Sensitive Data

Never log:

```text
password
API key
AWS secret access key
private key
session token
database password
authorization header
credit card data
```

Even DEBUG logs must follow security rules.

---

## 43. Secret Redaction

If an API response may contain secrets:

```python
safe_response = {
    "status": response["status"],
    "request_id": response["request_id"],
}

logger.debug(
    "API response: %s",
    safe_response,
)
```

Prefer allowlisting fields instead of attempting to redact every possible secret after logging.

---

## 44. Never Log Full Environment Variables

Bad:

```python
logger.debug(
    "Environment: %s",
    os.environ,
)
```

Environment variables may contain credentials.

Instead:

```python
logger.debug(
    "Required environment variables loaded"
)
```

or log only safe field names.

---

## 45. Never Log Full HTTP Headers

Bad:

```python
logger.debug(
    "Headers: %s",
    headers,
)
```

Headers may contain:

```text
Authorization
Cookie
API tokens
session IDs
```

Log safe metadata only.

---

## 46. Password Redaction

If a configuration object must be logged, create a sanitized copy:

```python
safe = dict(config)

safe["password"] = "***"

logger.debug(
    "Configuration: %s",
    safe,
)
```

For complex nested structures, use a reliable sanitization utility.

---

## 47. Logging Exceptions Safely

Tracebacks can sometimes contain sensitive values.

Before enabling verbose exception logging in production, understand whether exception messages may include:

```text
request data
credentials
tokens
file contents
```

Log useful diagnostic context without exposing secrets.

---

## 48. Log Injection

Never blindly log untrusted user input if the logging backend interprets control characters.

Example:

```python
logger.info(
    "User input: %s",
    user_input,
)
```

Sanitize or encode dangerous control characters when required by your logging environment.

---

## 49. Log Forging

An attacker may try to inject:

```text
\n
ERROR fake event
```

into a log message.

Structured logging and safe serialization reduce this risk.

Do not construct raw log lines by string concatenation from untrusted data.

---

## 50. Logging Levels in Production

Typical default:

```text
INFO
```

Temporary troubleshooting:

```text
DEBUG
```

Errors:

```text
ERROR
```

Severe failures:

```text
CRITICAL
```

The exact policy should be defined by the service.

---

## 51. Dynamic Log Levels

A service may support changing log level without rebuilding the application.

Example concept:

```text
INFO
  |
  v
incident
  |
  v
DEBUG temporarily
  |
  v
back to INFO
```

Do this carefully because DEBUG can dramatically increase volume.

---

## 52. Log Volume

Too many logs cause:

```text
storage cost
network traffic
Elasticsearch pressure
slow queries
alert noise
```

Log meaningful events, not every internal variable.

---

## 53. Logging Every Loop Iteration

Bad:

```python
while True:
    logger.info(
        "Checking status"
    )
```

This can create enormous log volume.

Better:

```python
logger.debug(
    "Polling deployment status"
)
```

or log only meaningful state transitions.

---

## 54. State-Change Logging

Instead of:

```text
INFO checking status
INFO checking status
INFO checking status
```

log:

```text
INFO deployment entered Progressing
INFO deployment reached Available
```

This produces higher-value logs.

---

## 55. Log Sampling

High-volume services may sample repetitive logs.

Example concept:

```text
10,000 identical requests
        |
        v
sample / aggregate
        |
        v
representative logs
```

Do not sample security-critical or incident-critical events without understanding the consequences.

---

## 56. Error Aggregation

Instead of logging the same error thousands of times:

```text
connection failed
connection failed
connection failed
...
```

the platform can aggregate:

```text
error type
service
time window
count
```

This makes incident analysis easier.

---

## 57. Exception Classification

Bad:

```python
except Exception:
    logger.error("failed")
```

Better:

```python
except TimeoutError:
    logger.warning(
        "Dependency timed out"
    )
```

Then handle unexpected errors separately.

---

## 58. Error Context

Good:

```python
logger.error(
    "Deployment failed for service=%s environment=%s",
    service,
    environment,
)
```

Better if structured fields are supported:

```text
event=deployment_failed
service=payment
environment=production
```

---

## 59. Logging External Commands

A DevOps script may execute:

```python
subprocess.run(...)
```

Log:

```text
command purpose
target environment
result
duration
```

Be careful with commands containing secrets.

---

## 60. Never Log Secret-Bearing Commands

Bad:

```text
aws ... --password secret123
```

If a command contains credentials, sanitize it before logging.

Better:

```text
Running database migration
```

rather than the full secret-bearing command.

---

## 61. Subprocess Logging

```python
import subprocess

logger.info(
    "Running Terraform plan"
)

result = subprocess.run(
    [
        "terraform",
        "plan",
    ],
    text=True,
    capture_output=True,
)

if result.returncode != 0:
    logger.error(
        "Terraform plan failed"
    )
```

For large output, avoid storing huge stdout/stderr strings unnecessarily.

---

## 62. Capture vs Stream

```python
capture_output=True
```

stores output in memory.

For large commands, streaming output may be better.

The choice depends on whether you need:

```text
post-processing
real-time CI logs
full output retention
```

---

## 63. CI/CD Logging

A good CI/CD log should show:

```text
stage started
stage completed
duration
result
important artifact/version
failure reason
```

Example:

```text
INFO stage=build event=started
INFO stage=build event=completed duration=82.4
```

---

## 64. CI Exit Codes

Logging a failure does not automatically fail CI.

Use:

```python
raise
```

or:

```python
sys.exit(1)
```

when appropriate.

A production automation should make its success/failure state unambiguous.

---

## 65. Logging Terraform Automation

Useful:

```text
workspace
environment
module
plan started
plan completed
apply started
apply completed
resource count
```

Avoid logging:

```text
terraform state contents
credentials
sensitive variable values
```

---

## 66. Logging Kubernetes Automation

Useful:

```text
cluster
namespace
deployment
image
desired replicas
rollout status
duration
```

Example:

```python
logger.info(
    "Starting rollout service=%s "
    "namespace=%s",
    service,
    namespace,
)
```

---

## 67. Logging AWS Automation

Useful:

```text
service
region
resource ID
action
result
duration
```

Do not log:

```text
secret access keys
session tokens
private credentials
```

Use IAM roles and SDK credential mechanisms.

---

## 68. AWS SDK Exceptions

With `boto3`:

```python
try:
    client.describe_instances()
except Exception:
    logger.exception(
        "Failed to query EC2"
    )
    raise
```

For production, catch relevant AWS exceptions where you need specific retry or recovery behavior.

---

## 69. Kubernetes Client Exceptions

Example:

```python
try:
    api.read_namespaced_deployment(
        name,
        namespace,
    )
except Exception:
    logger.exception(
        "Failed to read deployment"
    )
    raise
```

Again, classify retryable versus permanent errors.

---

## 70. Logging Deployment Rollbacks

Log explicit events:

```text
deployment_started
deployment_failed
rollback_started
rollback_completed
```

Example:

```python
logger.warning(
    "Rollback started service=%s",
    service,
)
```

Then:

```python
logger.info(
    "Rollback completed service=%s",
    service,
)
```

---

## 71. Logging Security Scans

For DevSecOps:

```text
scan_started
scan_completed
vulnerabilities_found
policy_failed
```

Do not log the entire security report if it contains secrets or excessive data.

Prefer:

```text
critical=0
high=2
medium=8
policy=FAILED
```

and store the detailed report securely.

---

## 72. Logging SonarQube/Trivy/Veracode Results

A CI script can log:

```text
tool
version
scan target
duration
finding summary
policy result
```

Example:

```text
INFO tool=trivy target=image
INFO critical=0 high=1 medium=4
ERROR security_policy=FAILED
```

---

## 73. Logging Docker Image Builds

Useful:

```text
image
tag
commit SHA
build duration
result
```

Example:

```text
INFO image=payment tag=abc123 build_started
```

Avoid relying on mutable tags alone when production identity requires immutability.

---

## 74. Image Digest Logging

For deployment traceability:

```text
image repository
tag
digest
```

A digest is more reliable as an immutable image identity than a mutable tag.

Log the digest when it is operationally useful.

---

## 75. Git Commit Logging

CI/CD logs should capture:

```text
repository
branch
commit SHA
deployment environment
```

Example:

```text
INFO commit=7f3a91d environment=production
```

Avoid logging secrets embedded accidentally in commit messages.

---

## 76. GitOps Logging

A GitOps deployment can correlate:

```text
Git commit
   |
   v
image digest
   |
   v
ArgoCD sync
   |
   v
Kubernetes rollout
```

This creates an auditable deployment chain.

---

## 77. Request Correlation in Microservices

Example:

```text
request_id=abc123

API
 |
 +--> user service
 |
 +--> product service
 |
 +--> payment service
 |
 +--> notification service
```

Every service logs the same correlation ID.

This is extremely useful during incidents.

---

## 78. Logging Async Work

For asynchronous systems:

```text
producer
   |
   v
RabbitMQ
   |
   v
consumer
```

Log:

```text
message ID
correlation ID
queue
consumer
processing start
processing completion
failure
```

Do not log the entire message if it contains sensitive data.

---

## 79. Logging Retries

Useful:

```python
logger.warning(
    "Retrying operation attempt=%d "
    "delay=%.2fs",
    attempt,
    delay,
)
```

This helps explain why an operation took longer.

---

## 80. Logging Backoff

```text
attempt=1 delay=1.2
attempt=2 delay=2.8
attempt=3 delay=4.1
```

Avoid logging every internal calculation if the retry library already exposes metrics.

---

## 81. Logging Timeouts

Example:

```python
logger.error(
    "Dependency timeout "
    "service=%s timeout=%ss",
    service,
    timeout,
)
```

Include the dependency and timeout policy.

---

## 82. Logging Health Checks

Do not log successful health checks every second.

Prefer:

```text
health state changed
```

Example:

```text
INFO health=healthy
WARNING health=degraded
ERROR health=unhealthy
```

---

## 83. Logging State Transitions

High-value events:

```text
STARTING -> READY
READY -> DEGRADED
DEGRADED -> FAILED
FAILED -> RECOVERED
```

State transitions are usually more useful than repetitive status polling logs.

---

## 84. Logging Startup

Useful startup information:

```text
service
version
environment
configuration source
port
build/commit
```

Do not log:

```text
passwords
tokens
full environment
```

Example:

```python
logger.info(
    "Service started version=%s "
    "environment=%s",
    version,
    environment,
)
```

---

## 85. Logging Shutdown

Useful:

```python
logger.info(
    "Service shutdown started"
)
```

and:

```python
logger.info(
    "Service shutdown completed"
)
```

This helps distinguish graceful termination from crashes.

---

## 86. SIGTERM and Kubernetes

Kubernetes may send:

```text
SIGTERM
```

before terminating a container.

A service can log:

```text
shutdown initiated
```

and complete graceful cleanup.

Do not rely on logs alone; configure appropriate:

```text
terminationGracePeriodSeconds
```

and application shutdown behavior.

---

## 87. Log Rotation

For file-based logs, rotation prevents unlimited growth.

Python provides:

```python
from logging.handlers import (
    RotatingFileHandler,
)
```

Example:

```python
handler = RotatingFileHandler(
    "app.log",
    maxBytes=10_000_000,
    backupCount=5,
)
```

---

## 88. Time-Based Rotation

Python provides:

```python
from logging.handlers import (
    TimedRotatingFileHandler,
)
```

Example:

```python
handler = TimedRotatingFileHandler(
    "app.log",
    when="midnight",
    backupCount=7,
)
```

Use this when logs need time-based rotation.

---

## 89. Container Logging vs File Rotation

In Kubernetes, prefer:

```text
stdout/stderr
```

and let the platform/logging pipeline handle retention and rotation where possible.

File rotation inside the application may be appropriate for non-containerized systems.

---

## 90. Handler

A handler determines:

```text
where logs go
```

Examples:

```text
console
file
rotating file
syslog
custom destination
```

---

## 91. Formatter

A formatter determines:

```text
how a log record looks
```

Example:

```python
formatter = logging.Formatter(
    "%(asctime)s "
    "%(levelname)s "
    "%(name)s "
    "%(message)s"
)
```

---

## 92. Handler + Formatter

```python
handler = logging.StreamHandler()

formatter = logging.Formatter(
    "%(asctime)s "
    "%(levelname)s "
    "%(message)s"
)

handler.setFormatter(
    formatter
)

logger.addHandler(handler)
```

This is the basic logging architecture:

```text
logger
  |
  v
handler
  |
  v
formatter
  |
  v
destination
```

---

## 93. Logger Level vs Handler Level

A logger can have:

```text
level = INFO
```

while a handler can have:

```text
level = ERROR
```

This allows different routing policies.

Understand both levels when debugging why a log message does not appear.

---

## 94. Propagation

Named loggers can propagate records to parent loggers.

This can accidentally create duplicate output if handlers are attached at multiple levels.

If you configure handlers manually, understand:

```python
logger.propagate
```

before changing it.

---

## 95. Duplicate Logs

Common cause:

```text
handler added repeatedly
```

or:

```text
child logger
+
parent logger
+
propagation
```

Result:

```text
same message printed twice
```

Centralize logging configuration and avoid adding handlers every time a function runs.

---

## 96. `basicConfig()` Pitfall

`basicConfig()` only configures logging if handlers have not already been configured.

In larger applications, use explicit logging configuration rather than assuming `basicConfig()` will always change existing configuration.

---

## 97. Logging Configuration

For larger services:

```text
application
    |
    v
logging configuration
    |
    +--> console handler
    +--> file handler
    +--> JSON formatter
```

Keep configuration separate from business logic.

---

## 98. `dictConfig`

Python supports:

```python
from logging.config import dictConfig
```

This allows logging configuration to be defined as structured configuration.

It is useful for larger applications with multiple handlers and formatters.

---

## 99. Environment-Based Logging

Example:

```text
development -> DEBUG
staging     -> INFO
production  -> INFO
```

Production should not automatically run at DEBUG because of:

```text
volume
cost
performance
security
```

---

## 100. Testing Logging

Test:

```text
important event logged
exception contains traceback
secret is not logged
correct level used
correct structured fields exist
```

Python testing frameworks can capture log records for assertions.

---

## 101. `caplog` in Pytest

Example:

```python
def test_deployment_logs(
    caplog,
):
    with caplog.at_level(
        logging.INFO
    ):
        deploy()

    assert (
        "Deployment started"
        in caplog.text
    )
```

This verifies operational logging behavior.

---

## 102. Logging as an Operational Contract

Logs are not just developer messages.

They can be consumed by:

```text
ELK
alerts
dashboards
incident tools
security systems
CI/CD
operations teams
```

Changing log structure can affect downstream consumers.

Treat important structured logs as an interface.

---

## 103. Log Retention

Retention depends on:

```text
security policy
compliance
incident investigation
storage cost
business requirements
```

Examples:

```text
7 days
30 days
90 days
1 year
```

Do not hard-code retention without understanding organizational requirements.

---

## 104. Log Storage Cost

Centralized logging can become expensive because of:

```text
high volume
long retention
large documents
high-cardinality fields
debug logs
duplicate logs
```

Optimize:

```text
log levels
sampling
retention
field selection
compression
```

---

## 105. Log Index Strategy

For Elasticsearch-style systems, plan:

```text
index naming
retention
sharding
mapping
field types
```

Python applications should produce predictable structured data so the ingestion pipeline can map it consistently.

---

## 106. Log Timestamp Mapping

Ensure the centralized logging pipeline recognizes the timestamp field correctly.

If the application emits:

```json
{
  "timestamp": "2026-08-16T10:30:00Z"
}
```

the collector should map it to the appropriate time field.

Otherwise dashboards may show incorrect event times.

---

## 107. Log Parsing

If the application emits plain text:

```text
INFO deployment started service=payment
```

the collector may need parsing.

If it emits structured JSON:

```json
{
  "level": "INFO",
  "event": "deployment_started",
  "service": "payment"
}
```

parsing is much more reliable.

---

## 108. Avoid Regex When Structured Logs Exist

Bad:

```text
regex against a JSON string
```

Better:

```python
json.loads()
```

for JSON data.

Regex is useful for genuinely unstructured logs.

---

## 109. JSON Logging Example

A simple custom formatter:

```python
import json
import logging

class JsonFormatter(
    logging.Formatter
):
    def format(self, record):
        data = {
            "level": record.levelname,
            "logger": record.name,
            "message": record.getMessage(),
        }

        return json.dumps(data)
```

Production implementations should also handle timestamps, exceptions, safe context, and serialization edge cases.

---

## 110. Structured JSON With Timestamp

Conceptually:

```python
data = {
    "timestamp": datetime.now(
        timezone.utc
    ).isoformat(),
    "level": record.levelname,
    "logger": record.name,
    "message": record.getMessage(),
}
```

Use the log record's timestamp when designing a complete formatter so the event time is consistent with the logging framework.

---

## 111. Exception in JSON Logs

A structured error record may contain:

```json
{
  "level": "ERROR",
  "event": "deployment_failed",
  "exception_type": "TimeoutError",
  "message": "Deployment timed out"
}
```

The traceback can be stored in a dedicated field if the backend and security policy support it.

---

## 112. Logging Event Duration

```python
start = time.monotonic()

operation()

duration = (
    time.monotonic()
    - start
)

logger.info(
    "Operation completed "
    "duration=%.3fs",
    duration,
)
```

This is useful for performance troubleshooting.

---

## 113. Context Manager for Timing

A reusable helper:

```python
from contextlib import contextmanager
import time

@contextmanager
def log_duration(
    logger,
    operation,
):
    start = time.monotonic()

    try:
        yield
    finally:
        duration = (
            time.monotonic()
            - start
        )

        logger.info(
            "%s duration=%.3fs",
            operation,
            duration,
        )
```

Use:

```python
with log_duration(
    logger,
    "terraform_plan",
):
    run_plan()
```

---

## 114. Logging a Deployment Step

```python
logger.info(
    "stage_started stage=%s",
    stage,
)

try:
    run_stage(stage)

except Exception:
    logger.exception(
        "stage_failed stage=%s",
        stage,
    )
    raise

logger.info(
    "stage_completed stage=%s",
    stage,
)
```

This creates a clear operational trail.

---

## 115. Logging Configuration Validation

```python
logger.info(
    "Configuration validation started"
)

validate(config)

logger.info(
    "Configuration validation completed"
)
```

On failure:

```python
logger.exception(
    "Configuration validation failed"
)
```

Do not log the full configuration if it may contain secrets.

---

## 116. Logging Terraform Plan

```python
logger.info(
    "Terraform plan started"
)

result = run_terraform_plan()

if result.returncode != 0:
    logger.error(
        "Terraform plan failed"
    )
    raise RuntimeError(
        "Terraform plan failed"
    )

logger.info(
    "Terraform plan completed"
)
```

---

## 117. Logging Helm Deployment

```python
logger.info(
    "Helm deployment started "
    "release=%s namespace=%s",
    release,
    namespace,
)
```

Then:

```python
logger.info(
    "Helm deployment completed "
    "release=%s",
    release,
)
```

Keep detailed Helm output available in CI logs without duplicating everything into application logs.

---

## 118. Logging ArgoCD Synchronization

A deployment automation may log:

```text
git_revision
application
sync_started
sync_completed
health_status
```

Example:

```text
event=argocd_sync_completed
application=payment
revision=7f3a91d
health=Healthy
```

This creates deployment traceability.

---

## 119. Logging Kubernetes Failures

Useful event:

```text
event=rollout_failed
service=payment
namespace=production
reason=ProgressDeadlineExceeded
```

Then collect detailed Kubernetes diagnostics separately:

```bash
kubectl describe
kubectl logs
kubectl get events
```

Do not dump the entire cluster state into every application log.

---

## 120. Logging CrashLoopBackOff Investigation

A Python troubleshooting tool can log:

```text
pod
namespace
restart_count
last_state
reason
exit_code
```

Example:

```python
logger.warning(
    "Pod restart detected "
    "pod=%s namespace=%s "
    "restarts=%d",
    pod,
    namespace,
    restart_count,
)
```

---

## 121. Logging OOMKilled

```text
event=container_terminated
reason=OOMKilled
pod=payment-abc
container=payment
```

This is far more useful than:

```text
something went wrong
```

---

## 122. Logging ImagePullBackOff

Useful fields:

```text
pod
namespace
image
reason
registry
```

Do not log registry credentials.

---

## 123. Logging Node Pressure

Useful:

```text
node
condition
resource
observed value
threshold
```

Example:

```text
event=node_pressure
node=worker-01
condition=DiskPressure
```

---

## 124. Logging Security Events

Security-relevant events may include:

```text
authentication failure
authorization failure
secret access failure
policy violation
image vulnerability
unexpected configuration change
```

Do not include the actual secret or credential.

---

## 125. Audit vs Application Logs

Application logs explain:

```text
what application did
```

Audit logs explain:

```text
who performed an action
what action
which resource
when
result
```

Do not assume application logs replace proper audit logging.

---

## 126. Logging IAM Failures

A useful operational log:

```text
event=aws_api_failed
service=ec2
operation=DescribeInstances
region=ap-south-1
error=AccessDenied
```

Avoid logging credentials or full authorization headers.

---

## 127. Logging Database Failures

Useful:

```text
database
operation
duration
error type
retry attempt
```

Avoid logging:

```text
password
full SQL containing sensitive values
customer records
```

Use parameterized queries and safe diagnostics.

---

## 128. Logging Network Failures

Useful:

```text
host
port
dependency
timeout
retry attempt
```

Example:

```text
event=dependency_timeout
dependency=inventory
timeout=5
attempt=2
```

---

## 129. Logging DNS Failures

Useful:

```text
hostname
resolver error
attempt
duration
environment
```

Avoid dumping entire DNS responses if not necessary.

---

## 130. Logging HTTP Failures

Useful:

```text
method
safe URL/path
status code
duration
request ID
retry attempt
```

Avoid:

```text
Authorization header
cookies
full request body
sensitive query parameters
```

---

## 131. Logging HTTP Status Codes

Typical interpretation:

```text
2xx -> success
3xx -> redirect
4xx -> client/request issue
5xx -> server/dependency issue
```

But do not automatically classify every 4xx or 5xx as retryable.

---

## 132. Logging Rate Limits

```text
event=rate_limited
dependency=api
status=429
retry_after=10
```

This helps explain latency and retry behavior.

---

## 133. Logging Circuit Breaker State

Useful:

```text
closed
open
half_open
```

Log transitions:

```text
event=circuit_opened
dependency=payment
```

Avoid logging every request while the circuit remains open.

---

## 134. Logging Health State

Use state changes:

```text
healthy -> degraded
degraded -> unhealthy
unhealthy -> healthy
```

This avoids noisy repeated health logs.

---

## 135. Logging Feature Flags

If deployment behavior depends on a feature flag, logging the safe flag state can help explain behavior.

Do not log sensitive flag values if they contain secrets or internal data.

---

## 136. Logging Configuration Version

Useful:

```text
config_version
git_commit
deployment_version
```

This helps answer:

```text
Which configuration was active when the incident happened?
```

---

## 137. Logging Image Digest

For Kubernetes:

```text
image=repo/payment
digest=sha256:...
```

This is useful when a mutable tag points to different image builds over time.

---

## 138. Logging Node/Pod Identity

When troubleshooting Kubernetes:

```text
cluster
namespace
pod
container
node
```

can make a log much easier to correlate with infrastructure state.

---

## 139. Avoid Logging Huge Objects

Bad:

```python
logger.debug(
    "Full Kubernetes object: %s",
    huge_object,
)
```

This can produce:

```text
huge logs
high cost
sensitive data
slow pipelines
```

Log selected fields.

---

## 140. Allowlist Logging

Prefer:

```python
safe = {
    "name": obj["name"],
    "namespace": obj["namespace"],
    "status": obj["status"],
}
```

Then:

```python
logger.info(
    "Resource state=%s",
    safe,
)
```

This is safer than trying to remove secrets from a complete object.

---

## 141. Log Sampling and Debugging

During an incident, increasing logging can help.

But use:

```text
temporary
targeted
scoped
```

debugging.

After the incident, return to the normal level.

---

## 142. Logging Performance

Logging can consume:

```text
CPU
memory
disk
network
```

Avoid expensive work just to create a DEBUG message.

Parameterized logging helps:

```python
logger.debug(
    "Resource=%s",
    resource,
)
```

---

## 143. Expensive Debug Data

Bad:

```python
logger.debug(
    "Huge object=%s",
    expensive_function(),
)
```

The function may execute even when DEBUG is disabled.

Better:

```python
if logger.isEnabledFor(
    logging.DEBUG
):
    logger.debug(
        "Huge object=%s",
        expensive_function(),
    )
```

Use this only when the diagnostic computation itself is expensive.

---

## 144. Logging and Memory

Do not accumulate logs in a Python list:

```python
logs.append(...)
```

for an unbounded long-running process.

Emit records through the logging system.

---

## 145. Logging and Disk

If file logging is required:

```text
rotation
retention
permissions
filesystem capacity
```

must be considered.

A full filesystem can cause the application and logging system to fail together.

---

## 146. Disk Full Scenario

If:

```text
/var/log
```

reaches:

```text
100%
```

logging can fail and other application writes can fail.

Production systems should have:

```text
log rotation
retention
disk monitoring
centralized collection
```

---

## 147. Logging Backpressure

If the logging destination is slow:

```text
application
   |
   v
logging
   |
   v
slow destination
```

the application can experience latency depending on the logging architecture.

High-volume systems may use asynchronous or buffered logging.

---

## 148. Async Logging

Concept:

```text
application
    |
    v
queue
    |
    v
logging worker
    |
    v
destination
```

This can reduce application-thread blocking.

But queue sizing, shutdown behavior, and dropped-log policy must be understood.

---

## 149. Logging Failure Policy

Ask:

```text
Should application fail if logging backend is unavailable?
```

Usually business-critical application work should not fail merely because a non-critical log destination is unavailable.

But security/audit systems may have stricter requirements.

The policy must be explicit.

---

## 150. Log Integrity

For security-sensitive audit logs, consider:

```text
access control
immutability
retention
centralization
tamper resistance
```

Application logs and security audit logs may have different requirements.

---

## 151. Permissions for Log Files

If using local files:

```text
least privilege
correct owner
correct group
restricted permissions
```

Avoid world-readable files containing operational or sensitive data.

---

## 152. Log Rotation With System Tools

On Linux, applications may work with:

```text
logrotate
```

rather than implementing all rotation logic themselves.

Understand who owns:

```text
rotation
compression
retention
permissions
```

---

## 153. Container Log Rotation

Container runtimes and orchestration platforms can rotate logs.

Do not assume:

```text
stdout = unlimited storage
```

Configure appropriate platform-level retention.

---

## 154. ELK Troubleshooting Using Logs

Typical flow:

```text
Python application
   |
   v
stdout
   |
   v
collector
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

If logs are missing, troubleshoot each layer.

---

## 155. Missing Logs — Layer 1

Check application:

```text
Did the code reach the log statement?
Is logger configured?
Is level high enough?
```

Example:

```python
logger.info(
    "Reached deployment step"
)
```

---

## 156. Missing Logs — Layer 2

Check container:

```bash
kubectl logs <pod>
```

If nothing appears:

```text
stdout/stderr?
container running?
wrong container?
wrong namespace?
crashed before startup?
```

---

## 157. Missing Logs — Layer 3

Check collector:

```text
collector running?
permissions?
input path?
stdout collection?
network connectivity?
```

---

## 158. Missing Logs — Layer 4

Check Logstash/pipeline:

```text
input
filter
output
parsing
mapping
errors
```

---

## 159. Missing Logs — Layer 5

Check Elasticsearch/Kibana:

```text
index exists?
timestamp mapping?
retention?
query time range?
index pattern/data view?
```

Do not immediately modify Python code when the application logs already exist in stdout.

---

## 160. Log Parsing Failure

If JSON logs arrive but fields are missing:

```text
raw message
   |
   v
JSON parsing
   |
   +--> failure
```

Check:

```text
valid JSON
escaping
newline behavior
schema
collector configuration
```

---

## 161. Timestamp Parsing Failure

Symptoms:

```text
logs appear at wrong time
logs missing from dashboard
events appear in future
```

Check:

```text
timestamp field
timezone
format
collector parsing
Elasticsearch mapping
```

---

## 162. Duplicate Logs in ELK

Possible causes:

```text
application emits twice
collector reads twice
multiple handlers
logger propagation
pipeline duplicates
```

Find the first layer where duplication occurs.

---

## 163. Log Volume Spike

Troubleshoot:

```text
log level changed?
new loop logging?
retry storm?
traffic spike?
application error loop?
collector duplication?
```

Then reduce unnecessary volume without hiding important errors.

---

## 164. Error Log Storm

Example:

```text
dependency unavailable
   |
   v
every request fails
   |
   v
every request logs ERROR
   |
   v
millions of logs
```

Use:

```text
aggregation
rate limiting
state-change logging
metrics
```

while retaining enough diagnostic information.

---

## 165. Logs vs Metrics

Logs answer:

```text
What happened?
```

Metrics answer:

```text
How much/how often?
```

Example:

```text
log -> payment timeout request=abc
metric -> payment_timeout_total = 1042
```

Use both.

---

## 166. Logs vs Traces

Logs provide event details.

Traces provide request flow across services.

A strong observability architecture correlates:

```text
metrics
logs
traces
```

using common identifiers where possible.

---

## 167. Prometheus + Logs

Example:

```text
metric:
http_requests_total

log:
request failed status=500
```

Metrics identify the problem at scale; logs provide detailed context.

---

## 168. Grafana + Logs

Grafana can display logs alongside metrics depending on the configured logging backend.

A useful incident workflow is:

```text
metric spike
   |
   v
dashboard
   |
   v
logs
   |
   v
specific error
```

---

## 169. Logging and Alerts

Do not alert on every ERROR log.

Better:

```text
error rate
error count threshold
specific security event
critical state change
```

Logs can support alert investigation while metrics often provide more stable alert signals.

---

## 170. Log-Based Alert

If a security event must trigger immediately:

```text
event=authentication_failure
```

the centralized logging platform may generate an alert.

Use carefully to prevent alert floods.

---

## 171. Production Logging Checklist

```text
1. Use logging, not random print statements.
2. Use named loggers.
3. Choose meaningful log levels.
4. Prefer structured logs.
5. Include useful context.
6. Use UTC timestamps.
7. Correlate requests/deployments.
8. Never log secrets.
9. Sanitize sensitive data.
10. Avoid huge objects.
11. Avoid noisy loops.
12. Log state changes.
13. Use exception tracebacks for failures.
14. Preserve CI exit codes.
15. Prefer stdout/stderr in containers.
16. Centralize production logs.
17. Configure retention.
18. Monitor log volume.
19. Use rotation for local file logs.
20. Test logging behavior.
21. Keep schema consistent.
22. Avoid duplicate handlers.
23. Understand propagation.
24. Use monotonic time for durations.
25. Add deployment/version context.
26. Separate audit logs from normal application logs.
27. Protect log storage.
28. Avoid high-cardinality fields where unnecessary.
29. Validate timestamp parsing.
30. Troubleshoot logging layer by layer.
```

---

## 172. Practical Script — Basic Production Logger

```python
import logging
import sys

logging.basicConfig(
    level=logging.INFO,
    stream=sys.stdout,
    format=(
        "%(asctime)s "
        "%(levelname)s "
        "%(name)s "
        "%(message)s"
    ),
)

logger = logging.getLogger(
    __name__
)

logger.info(
    "Service started"
)
```

---

## 173. Practical Script — Exception Logging

```python
try:
    deploy()
except Exception:
    logger.exception(
        "Deployment failed"
    )
    raise
```

---

## 174. Practical Script — CI Failure

```python
import sys

try:
    validate()
except Exception:
    logger.exception(
        "Validation failed"
    )
    sys.exit(1)
```

---

## 175. Practical Script — Timed Operation Logging

```python
import time

start = time.monotonic()

try:
    deploy()

except Exception:
    logger.exception(
        "Deployment failed"
    )
    raise

finally:
    duration = (
        time.monotonic()
        - start
    )

    logger.info(
        "Deployment duration=%.2fs",
        duration,
    )
```

---

## 176. Practical Script — Retry Logging

```python
import logging
import random
import time

logger = logging.getLogger(
    __name__
)

for attempt in range(1, 4):
    try:
        call_api()
        break

    except TemporaryError:
        if attempt == 3:
            logger.exception(
                "Operation failed after "
                "%d attempts",
                attempt,
            )
            raise

        delay = min(
            2 ** attempt,
            30,
        )

        delay += random.uniform(
            0,
            1,
        )

        logger.warning(
            "Retrying attempt=%d "
            "delay=%.2fs",
            attempt,
            delay,
        )

        time.sleep(delay)
```

---

## 177. Practical Script — Safe Context Logging

```python
safe_context = {
    "service": service,
    "environment": environment,
    "version": version,
}

logger.info(
    "Deployment started context=%s",
    safe_context,
)
```

Only include approved non-sensitive fields.

---

## 178. Practical Script — Kubernetes Rollout Logging

```python
logger.info(
    "Rollout started "
    "service=%s namespace=%s",
    service,
    namespace,
)

try:
    wait_for_rollout()

except TimeoutError:
    logger.error(
        "Rollout timed out "
        "service=%s namespace=%s",
        service,
        namespace,
    )
    raise

logger.info(
    "Rollout completed "
    "service=%s namespace=%s",
    service,
    namespace,
)
```

---

## 179. Practical Script — AWS Automation Logging

```python
logger.info(
    "Querying EC2 "
    "region=%s",
    region,
)

try:
    response = ec2.describe_instances()

except Exception:
    logger.exception(
        "EC2 query failed "
        "region=%s",
        region,
    )
    raise

logger.info(
    "EC2 query completed"
)
```

---

## 180. Practical Script — Security Scan Logging

```python
logger.info(
    "Security scan started "
    "image=%s",
    image,
)

result = run_scan()

logger.info(
    "Security scan completed "
    "critical=%d high=%d",
    result.critical,
    result.high,
)

if result.policy_failed:
    logger.error(
        "Security policy failed"
    )
    raise SystemExit(1)
```

---

## 181. Practical Script — Deployment Event Logging

```python
logger.info(
    "event=deployment_started "
    "service=%s version=%s",
    service,
    version,
)

try:
    deploy()

except Exception:
    logger.exception(
        "event=deployment_failed "
        "service=%s version=%s",
        service,
        version,
    )
    raise

logger.info(
    "event=deployment_completed "
    "service=%s version=%s",
    service,
    version,
)
```

---

## 182. Interview — Why Logging Instead of Print?

### Answer

> `print()` is fine for quick local debugging, but production automation needs log levels, timestamps, handlers, structured context, exception tracebacks, and centralized collection. I use Python's logging framework for operational scripts and services.

---

## 183. Interview — What Are Python Logging Levels?

### Answer

> Python provides DEBUG, INFO, WARNING, ERROR, and CRITICAL. I normally use INFO for operational events, DEBUG for detailed troubleshooting, WARNING for unexpected recoverable conditions, ERROR for failed operations, and CRITICAL for severe failures.

---

## 184. Interview — How Do You Log Exceptions?

### Answer

> Inside an exception handler I use `logger.exception()` because it records the message and traceback. I then either re-raise the exception or return an appropriate failure code so CI/CD does not incorrectly report success.

---

## 185. Interview — Why Use stdout in Containers?

### Answer

> Containers are ephemeral, so local log files can disappear when a container is replaced. I prefer writing application logs to stdout/stderr and letting Kubernetes/container infrastructure and the centralized logging pipeline collect and retain them.

---

## 186. Interview — What Should Never Be Logged?

### Answer

> Passwords, API keys, AWS secret credentials, tokens, private keys, authorization headers, cookies, and sensitive customer data. I use allowlisted safe fields and redaction where necessary.

---

## 187. Interview — What Is Structured Logging?

### Answer

> Structured logging emits machine-readable fields such as timestamp, level, service, environment, event, request ID, and deployment ID. It makes centralized search, dashboards, alerting, and incident investigation much easier than parsing inconsistent free-text messages.

---

## 188. Interview — How Do You Avoid Log Noise?

### Answer

> I choose appropriate levels, avoid logging every polling iteration, log meaningful state transitions, aggregate repetitive failures where possible, and control DEBUG logging in production. Metrics are often better than logs for high-frequency counts.

---

## 189. Interview — How Do You Correlate Logs Across Microservices?

### Answer

> I propagate a correlation or request ID across service boundaries and include it in structured logs. During an incident I can search for that identifier across API, application, and infrastructure logs.

---

## 190. Interview — How Do You Troubleshoot Missing Kubernetes Logs?

### Answer

> First I verify that the application reached the log statement and that its logger level permits the message. Then I check `kubectl logs`, the correct namespace/container, stdout/stderr behavior, the collector, the Logstash pipeline, Elasticsearch indexing, and finally Kibana's time range and query.

---

## 191. Interview — How Do You Handle Log Rotation?

### Answer

> For file-based applications I can use Python's rotating handlers or OS-level logrotate. In Kubernetes I generally prefer stdout/stderr and let the container/logging platform manage rotation and retention.

---

## 192. Interview — Why Is Logging a Security Concern?

### Answer

> Logs are often centralized and retained for a long time, so accidentally logging credentials can create a major security exposure. I treat logs as sensitive operational data, use least privilege, avoid secrets, sanitize inputs, and protect the logging backend.

---

## 193. Scenario — Production Logs Suddenly Disappear

Investigate in this order:

```text
application
   |
   v
Python logger
   |
   v
stdout/stderr
   |
   v
container
   |
   v
collector
   |
   v
Logstash/pipeline
   |
   v
Elasticsearch
   |
   v
Kibana
```

Do not change every layer at once.

---

## 194. Scenario — Logs Are Duplicated

Check:

```text
multiple handlers
logger propagation
application started logging configuration twice
collector duplication
pipeline duplication
```

Fix the first layer producing duplicates.

---

## 195. Scenario — Elasticsearch Costs Suddenly Increase

Investigate:

```text
log volume
DEBUG enabled
retry storm
duplicate collection
large payloads
high-cardinality fields
retention
index growth
```

Reduce unnecessary logging without losing critical diagnostic information.

---

## 196. Scenario — Application Logs Contain Credentials

Immediate response:

```text
stop further exposure
remove/disable unsafe logging
rotate exposed credentials
investigate log retention/copies
restrict access
add regression tests
```

Do not assume deleting the latest log file removes the exposure from centralized systems or backups.

---

## 197. Scenario — Kibana Shows Wrong Log Times

Check:

```text
application timezone
timestamp format
UTC conversion
collector parser
Elasticsearch mapping
Kibana time field
dashboard timezone
```

Normalize machine timestamps to UTC.

---

## 198. Scenario — One Service Produces Millions of Logs

Check:

```text
DEBUG level
tight polling loop
retry loop
error storm
traffic increase
duplicate handlers
collector duplication
```

Then:

```text
reduce noisy logs
aggregate repetitive events
use metrics
retain useful error context
```

---

## 199. Scenario — CI Job Says Success Despite Error Logs

Important principle:

> **A log message does not determine process success.**

Check:

```python
raise
```

or:

```python
sys.exit(1)
```

and verify the shell/CI pipeline handles the exit code correctly.

---

## 200. Scenario — Rollout Logs Say "Success" but Deployment Is Broken

The logging script may be reporting:

```text
command executed successfully
```

instead of:

```text
deployment actually became healthy
```

Fix by checking actual Kubernetes state:

```text
available replicas
ready replicas
conditions
rollout status
pod health
```

Then log the real outcome.

---

## 201. Scenario — Logs Are Too Large

Check:

```text
full API responses
full Kubernetes objects
large stack traces
request/response bodies
debug payloads
```

Log selected fields and store detailed artifacts separately when needed.

---

## 202. Scenario — Log Collector Cannot Parse JSON

Check:

```text
valid JSON
one event per line
escaping
newline characters
timestamp field
schema
collector parser
```

Test with a representative log sample before changing production configuration.

---

## 203. Scenario — Log Search Is Slow

Possible causes:

```text
huge index
poor field mappings
high cardinality
long time range
too many shards
large documents
```

Improve:

```text
index strategy
retention
field selection
query filters
time ranges
```

---

## 204. Scenario — Python Script Logs Nothing

Check:

```python
logger = logging.getLogger(__name__)
```

Then:

```text
logger level
handler
handler level
configuration initialization
propagation
```

Remember that:

```python
logger.debug(...)
```

will not appear when the effective level is INFO.

---

## 205. Production Architecture — Python Logging

```text
                Python Service
                     |
              named logger
                     |
              logging records
                     |
          +----------+----------+
          |                     |
       stdout                 file*
          |                     |
          v                     v
     collector              rotation
          |                     |
          +----------+----------+
                     |
                     v
             centralized logs
                     |
                     v
               Elasticsearch
                     |
                     v
                  Kibana

* mainly for non-containerized workloads
```

---

## 206. Production Architecture — DevOps Automation

```text
Git / CI
   |
   v
Python automation
   |
   +--> Terraform
   +--> AWS
   +--> Kubernetes
   +--> Helm
   +--> Security tools
   |
   v
structured logs
   |
   v
CI stdout / centralized logging
   |
   v
incident investigation
```

---

## 207. Production Architecture — Observability

```text
                  Application
                      |
        +-------------+-------------+
        |             |             |
      Logs         Metrics        Traces
        |             |             |
        v             v             v
       ELK        Prometheus      tracing
        |             |             |
        +-------------+-------------+
                      |
                      v
                   Grafana
                      |
                      v
                 Investigation
```

Logs provide detailed event context while metrics provide aggregate system behavior.

---

## 208. Production Architecture — DevSecOps Pipeline Logs

```text
Git
 |
 v
CI pipeline
 |
 +--> Build
 |      |
 |      v
 |    logs
 |
 +--> SonarQube
 |      |
 |      v
 |    findings
 |
 +--> Trivy
 |      |
 |      v
 |    vulnerability summary
 |
 +--> Veracode
 |      |
 |      v
 |    policy result
 |
 +--> Docker
 |
 +--> ECR
 |
 +--> ArgoCD
 |
 +--> EKS
```

Each stage should produce enough information to diagnose failure without exposing secrets.

---

## 209. Production Logging Standards

A mature Python DevOps project should define:

```text
log levels
timestamp format
timezone
event naming
required context
secret handling
retention
rotation
structured schema
correlation ID
error policy
```

This prevents every script from inventing a different logging format.

---

## 210. Recommended Log Event Schema

A practical baseline:

```json
{
  "timestamp": "2026-08-16T10:30:00Z",
  "level": "INFO",
  "service": "deployment-controller",
  "environment": "production",
  "event": "deployment_completed",
  "message": "Deployment completed",
  "version": "v42",
  "deployment_id": "deploy-123"
}
```

Add fields only when they provide operational value.

---

## 211. Logging Anti-Patterns

Avoid:

```text
1. print() everywhere
2. logging secrets
3. logging full environment variables
4. logging full HTTP headers
5. logging full Kubernetes objects
6. logging every polling iteration
7. DEBUG permanently enabled
8. swallowing exceptions
9. duplicate handlers
10. inconsistent timestamp formats
11. local-only logs in ephemeral containers
12. unlimited log files
13. giant unstructured messages
14. ambiguous event names
15. treating logs as the only observability signal
```

---

## 212. Python Logging Quick Reference

```python
import logging

logger = logging.getLogger(__name__)

logger.debug("Detailed diagnostic")
logger.info("Normal operation")
logger.warning("Unexpected condition")
logger.error("Operation failed")
logger.critical("Severe failure")
```

Exception:

```python
try:
    operation()
except Exception:
    logger.exception(
        "Operation failed"
    )
    raise
```

---

## 213. Production Logging Quick Reference

```text
Application
   |
   v
named logger
   |
   v
structured record
   |
   v
stdout/stderr
   |
   v
collector
   |
   v
ELK
   |
   v
search / dashboard / investigation
```

---

## 214. Final Mental Model

```text
                  LOGGING
                     |
       +-------------+-------------+
       |             |             |
     LEVEL         CONTEXT      TIMESTAMP
       |             |             |
   INFO/ERROR     service/id       UTC
       |             |             |
       +-------------+-------------+
                     |
                     v
               STRUCTURED LOG
                     |
                     v
                stdout/stderr
                     |
                     v
                 collector
                     |
                     v
                    ELK
                     |
                     v
              troubleshooting
```

Remember:

```text
Logs should be:
consistent
searchable
actionable
secure
timestamped
correlatable
```

The goal is not to produce more logs.

The goal is to produce **better operational evidence**.

---

## 215. Next File

```text
08-Argparse.md
```

The next topic will cover Python CLI automation for DevOps:

```text
command-line arguments
argparse
positional arguments
optional arguments
flags
defaults
choices
types
required arguments
subcommands
help
validation
exit codes
CLI design
Terraform/Kubernetes-style commands
production scripts
CI/CD integration
real DevOps automation
interview questions
scenario-based questions
```
