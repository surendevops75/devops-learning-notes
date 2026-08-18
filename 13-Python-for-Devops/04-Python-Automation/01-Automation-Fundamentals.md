# 01-Automation-Fundamentals

## Python Automation for DevOps

Automation is one of the most important practical uses of Python in DevOps.

A DevOps engineer repeatedly performs tasks such as:

```text
Run commands
Create files
Modify configuration
Collect logs
Check services
Validate deployments
Back up data
Call APIs
Query cloud resources
Send notifications
Generate reports
```

Python allows these tasks to become:

```text
repeatable
consistent
auditable
testable
scalable
```

The goal of automation is not simply:

> "Replace a shell command with Python."

The goal is:

> **Turn a repeatable operational process into reliable, safe, observable software.**

---

# 1. What Is Automation?

Automation means using software to perform a task with minimal manual intervention.

Manual:

```text
SSH
 ↓
run command
 ↓
check output
 ↓
copy file
 ↓
restart service
 ↓
verify
```

Automated:

```text
Python
 ↓
validate
 ↓
execute
 ↓
verify
 ↓
report
```

---

# 2. Why Python for DevOps Automation?

Python is useful because it provides:

```text
standard library
rich ecosystem
API support
JSON/YAML handling
OS integration
subprocess execution
cloud SDKs
network libraries
testing frameworks
```

It also works well across Linux environments.

---

# 3. Python vs Bash

Bash is excellent for:

```text
short Linux commands
pipelines
simple glue
interactive administration
```

Python becomes more useful when you need:

```text
complex logic
structured data
error handling
API calls
configuration
testing
reusable functions
parallelism
JSON/YAML
larger automation
```

---

# 4. Example: Simple Bash Automation

```bash
#!/bin/bash

df -h /
systemctl is-active nginx
curl -f http://localhost/health
```

This is perfectly reasonable for a small task.

---

# 5. Python Equivalent

```python
import subprocess

subprocess.run(
    ["df", "-h", "/"],
    check=True,
)

subprocess.run(
    ["systemctl", "is-active", "nginx"],
    check=True,
)

subprocess.run(
    ["curl", "-f", "http://localhost/health"],
    check=True,
)
```

Python becomes more valuable when we need to interpret results and make decisions.

---

# 6. Automation Lifecycle

A production automation should usually follow:

```text
Understand
   ↓
Validate
   ↓
Plan
   ↓
Execute
   ↓
Verify
   ↓
Report
```

Do not start changing the system before validating assumptions.

---

# 7. The Five Questions

Before automating a task ask:

```text
1. What is the desired state?
2. What inputs are required?
3. What can go wrong?
4. How do I verify success?
5. What happens if it fails?
```

These questions prevent fragile scripts.

---

# 8. Desired State

Example:

```text
nginx should be installed
nginx should be enabled
nginx should be running
port 443 should listen
HTTPS health endpoint should return 200
```

Automation should move the system toward this state and verify it.

---

# 9. Imperative vs Desired State

Imperative:

```text
run:
dnf install nginx
systemctl start nginx
```

Desired state:

```text
nginx:
  installed: true
  enabled: true
  running: true
```

Desired-state thinking is important in DevOps.

---

# 10. Idempotency

An operation is idempotent when running it multiple times produces the same intended final state.

Example:

```text
create directory /opt/app
```

First run:

```text
created
```

Second run:

```text
already exists
```

Final state is the same.

---

# 11. Why Idempotency Matters

Without idempotency:

```text
run 1 → success
run 2 → duplicate
run 3 → corruption
```

With idempotency:

```text
run 1 → configured
run 2 → already configured
run 3 → already configured
```

Automation should generally be safe to re-run.

---

# 12. Python Idempotent Directory Creation

```python
from pathlib import Path

Path("/opt/myapp").mkdir(
    parents=True,
    exist_ok=True,
)
```

This is naturally idempotent.

---

# 13. Idempotent File Creation

```python
from pathlib import Path

path = Path(
    "/opt/myapp/config.txt"
)

if not path.exists():
    path.write_text(
        "enabled=true\n",
        encoding="utf-8",
    )
```

But be careful: existence alone does not mean correct content.

---

# 14. Desired Content

Better:

```python
desired = "enabled=true\n"

if path.read_text(
    encoding="utf-8"
) != desired:

    path.write_text(
        desired,
        encoding="utf-8",
    )
```

This checks desired state.

---

# 15. Automation Safety

Before making a change:

```text
validate input
validate target
validate permissions
validate prerequisites
```

Then:

```text
change
verify
```

---

# 16. Dry Run

A dry run tells the operator:

```text
what would change
```

without actually changing it.

Example:

```bash
python deploy.py --dry-run
```

Output:

```text
Would create /opt/app
Would update nginx.conf
Would restart nginx
```

---

# 17. Why Dry Run Matters

Useful for:

```text
production
change review
debugging
CI/CD
incident response
```

It reduces accidental changes.

---

# 18. Check Before Change

Bad:

```python
subprocess.run(
    ["systemctl", "restart", "nginx"]
)
```

Better:

```text
check configuration
 ↓
check service state
 ↓
change only if required
 ↓
verify
```

---

# 19. Preflight Checks

Typical preflight:

```text
OS
disk
memory
permissions
network
dependencies
configuration
version
```

---

# 20. Postflight Checks

After automation:

```text
file exists?
configuration valid?
service active?
port listening?
health endpoint working?
expected version deployed?
```

---

# 21. Automation Must Verify

Never assume:

```text
command returned
```

means:

```text
task succeeded
```

For example:

```text
systemctl restart nginx
```

may return successfully while the service later fails.

Always verify the desired state.

---

# 22. Verification

Example:

```python
import subprocess

subprocess.run(
    ["systemctl", "restart", "nginx"],
    check=True,
)

result = subprocess.run(
    ["systemctl", "is-active", "nginx"],
    capture_output=True,
    text=True,
    check=False,
)

if result.stdout.strip() != "active":
    raise RuntimeError(
        "nginx failed verification"
    )
```

---

# 23. Automation Failure Model

Possible failures:

```text
invalid input
missing dependency
permission denied
command not found
network timeout
authentication failure
partial execution
unexpected output
configuration error
resource exhaustion
```

Design for these explicitly.

---

# 24. Exception Handling

```python
try:
    run_task()
except Exception as exc:
    logger.exception(
        "Automation failed: %s",
        exc,
    )
    raise
```

Do not hide exceptions.

---

# 25. Catch Specific Exceptions

Prefer:

```python
try:
    path.read_text(
        encoding="utf-8"
    )
except FileNotFoundError:
    ...
except PermissionError:
    ...
```

rather than catching every error and pretending the task succeeded.

---

# 26. Logging

Automation should produce useful logs:

```text
what happened
when
where
which target
what changed
result
error
```

Example:

```text
INFO Updating nginx configuration
INFO Configuration validation passed
INFO Restarting nginx
INFO nginx active
```

---

# 27. Avoid Secret Logging

Never log:

```text
password
API token
private key
database credential
AWS secret
authorization header
```

Bad:

```python
logger.info(
    "Using token %s",
    token,
)
```

---

# 28. Structured Logging

Useful fields:

```text
timestamp
level
host
environment
task
action
status
duration
request_id
```

Structured logs make automation easier to troubleshoot.

---

# 29. Automation Configuration

Avoid hardcoding:

```python
HOST = "prod01"
PORT = 443
```

Prefer:

```text
CLI arguments
environment variables
YAML
JSON
configuration files
secret managers
```

---

# 30. Configuration Example

```yaml
application:
  name: myapp
  directory: /opt/myapp

service:
  name: myapp

health:
  url: http://127.0.0.1:8080/health
```

---

# 31. Configuration Separation

Keep:

```text
code
```

separate from:

```text
environment-specific configuration
```

Example:

```text
code
+
dev.yaml
+
staging.yaml
+
prod.yaml
```

---

# 32. Environment Variables

Useful for deployment-specific values:

```bash
export APP_ENV=production
```

Python:

```python
import os

environment = os.environ.get(
    "APP_ENV",
    "development",
)
```

---

# 33. Don't Store Secrets in Environment Variables by Default

Environment variables are convenient, but secrets can appear in:

```text
process inspection
debug output
CI logs
crash reports
```

For sensitive credentials prefer a proper secret-management solution.

---

# 34. Secret Management

Examples:

```text
AWS Secrets Manager
AWS Systems Manager Parameter Store
Kubernetes Secrets
Vault
CI/CD secret store
```

Python automation should retrieve only the minimum secret data required.

---

# 35. Input Validation

Never blindly trust:

```text
filename
hostname
command
URL
service name
environment
user input
```

Validate them before using them.

---

# 36. Command Injection

Dangerous:

```python
import os

os.system(
    f"systemctl restart {service}"
)
```

If `service` is user-controlled, command injection can occur.

---

# 37. Safer Subprocess

Prefer:

```python
subprocess.run(
    [
        "systemctl",
        "restart",
        service,
    ],
    check=True,
)
```

Then validate `service` against an allowlist when appropriate.

---

# 38. Shell=False

`subprocess.run()` normally uses:

```python
shell=False
```

This is safer because arguments are passed directly rather than interpreted by a shell.

---

# 39. When `shell=True` Is Dangerous

Avoid:

```python
subprocess.run(
    user_input,
    shell=True,
)
```

especially with external input.

It can execute unintended shell commands.

---

# 40. Absolute Command Paths

For security-sensitive automation, you may prefer:

```text
/usr/bin/systemctl
/usr/bin/python3
/usr/bin/curl
```

instead of relying on an uncontrolled PATH.

Whether this is necessary depends on the execution environment.

---

# 41. Environment Control

Automation can fail because:

```text
PATH differs
HOME differs
working directory differs
locale differs
permissions differ
```

CI/CD environments often behave differently from interactive shells.

---

# 42. Working Directory

Use:

```python
subprocess.run(
    command,
    cwd="/opt/myapp",
    check=True,
)
```

instead of assuming the current directory.

---

# 43. Environment Variables in Subprocess

```python
import os
import subprocess

env = os.environ.copy()

env["APP_ENV"] = "production"

subprocess.run(
    ["./deploy.sh"],
    env=env,
    check=True,
)
```

---

# 44. Capture Output

```python
result = subprocess.run(
    ["systemctl", "status", "nginx"],
    capture_output=True,
    text=True,
    check=False,
)

print(result.stdout)
print(result.stderr)
```

Use output for diagnostics.

---

# 45. Return Code

```python
result.returncode
```

Example:

```text
0 = success
non-zero = failure
```

Do not assume every command follows exactly the same semantic convention; check its documentation.

---

# 46. `check=True`

```python
subprocess.run(
    command,
    check=True,
)
```

Raises:

```text
subprocess.CalledProcessError
```

when the command exits non-zero.

---

# 47. `check=False`

Useful when failure is an expected branch:

```python
result = subprocess.run(
    command,
    check=False,
)
```

Then explicitly inspect the return code.

---

# 48. Command Timeout

```python
subprocess.run(
    command,
    timeout=30,
    check=True,
)
```

Never allow external commands to hang forever.

---

# 49. Timeout Handling

```python
try:
    subprocess.run(
        command,
        timeout=30,
        check=True,
    )
except subprocess.TimeoutExpired:
    logger.error(
        "Command timed out"
    )
    raise
```

---

# 50. Retry Strategy

Retry only operations that are safe to retry.

Good candidates:

```text
temporary network failure
API throttling
transient service dependency
```

Bad candidates:

```text
non-idempotent database write
duplicate payment
destructive command
```

---

# 51. Exponential Backoff

Example:

```python
import time

for attempt in range(3):
    try:
        run_task()
        break
    except TemporaryError:
        time.sleep(
            2 ** attempt
        )
```

Production systems should also consider jitter and maximum delay.

---

# 52. Retry With Jitter

Concept:

```text
base delay
+
random small delay
```

This prevents many workers from retrying simultaneously.

---

# 53. Maximum Retries

Always define:

```text
max attempts
max delay
overall timeout
```

Avoid infinite retry loops.

---

# 54. Partial Failure

Suppose automation processes:

```text
server1
server2
server3
server4
```

and:

```text
server1 PASS
server2 PASS
server3 FAIL
server4 NOT RUN
```

The tool must clearly report the partial state.

---

# 55. Continue-on-Error

Some tasks should continue:

```text
backup 100 servers
```

if one server fails.

Others should stop immediately:

```text
database migration
```

when an earlier step fails.

The policy depends on the operation.

---

# 56. Fail-Fast vs Continue

Fail-fast:

```text
critical prerequisite failed
→ stop
```

Continue:

```text
independent host failed
→ record failure
→ process remaining hosts
```

---

# 57. Transaction Thinking

Python automation often cannot provide a true transaction.

Instead use:

```text
precheck
change
verify
rollback if possible
```

---

# 58. Backup Before Change

For risky configuration:

```text
read current
 ↓
backup
 ↓
write new
 ↓
validate
 ↓
reload
 ↓
verify
```

This pattern is fundamental in production automation.

---

# 59. Atomic File Updates

Avoid:

```python
open(
    "/etc/myapp.conf",
    "w",
)
```

and writing incomplete content directly to a critical file.

A safer approach is:

```text
write temporary file
 ↓
flush
 ↓
validate
 ↓
atomic replace
```

---

# 60. Atomic Replace

Python:

```python
from pathlib import Path

tmp = Path(
    "/etc/myapp.conf.tmp"
)

tmp.write_text(
    content,
    encoding="utf-8",
)

tmp.replace(
    "/etc/myapp.conf"
)
```

For critical production files, also consider permissions, ownership, filesystem behavior, and durability requirements.

---

# 61. File Permissions

Automation must preserve intended:

```text
owner
group
mode
```

Example:

```python
path.chmod(0o640)
```

Be careful with sensitive configuration.

---

# 62. Ownership

Linux automation may need:

```bash
chown app:app /opt/app/config
```

Python can use:

```python
import os

os.chown(
    path,
    uid,
    gid,
)
```

This may require elevated privileges.

---

# 63. Never Use 777 by Default

Avoid:

```bash
chmod 777
```

as a generic fix.

Use least privilege:

```text
644
640
600
755
750
```

according to the requirement.

---

# 64. Automation Permissions

A script should run with only the permissions it needs.

For example:

```text
read logs
write /opt/app
restart one service
```

does not necessarily require:

```text
full root access
```

---

# 65. Sudo

If elevated access is required:

```text
sudo
```

should be restricted to approved commands.

Avoid giving an automation user:

```text
ALL=(ALL) NOPASSWD: ALL
```

unless there is an exceptional, reviewed reason.

---

# 66. Automation User

A dedicated account can provide:

```text
controlled permissions
auditing
separation
credential rotation
```

Example:

```text
devops-automation
```

with narrowly defined privileges.

---

# 67. Automation and SSH

Remote automation commonly follows:

```text
Python
 ↓
SSH
 ↓
remote command
 ↓
verify
```

Use SSH keys or an approved centralized access mechanism.

Avoid embedding passwords in Python source.

---

# 68. Automation and AWS

Python can automate AWS using:

```python
import boto3
```

Examples:

```text
EC2
S3
EKS
IAM
RDS
Route53
ELB
SSM
```

Use IAM roles where possible.

---

# 69. AWS Automation Principle

Prefer:

```text
IAM role
```

over:

```text
hardcoded AWS access key
```

on AWS compute.

---

# 70. Least-Privilege IAM

An automation role should have only required permissions.

For example, a backup script may need:

```text
s3:PutObject
```

but not:

```text
iam:*
```

---

# 71. Automation and Kubernetes

Python can automate:

```text
Kubernetes API
kubectl commands
manifest validation
deployment verification
pod diagnostics
```

For application deployment, GitOps/IaC should remain the source of truth where that is the architecture.

---

# 72. Automation and Terraform

Terraform should generally manage:

```text
infrastructure desired state
```

Python can handle:

```text
preflight
validation
reporting
orchestration around Terraform
```

Avoid replacing Terraform resource management with ad-hoc Python calls when Terraform is the intended source of truth.

---

# 73. Automation and Ansible

Ansible is strong for:

```text
configuration management
multi-host orchestration
idempotent changes
```

Python is useful for:

```text
custom logic
API integrations
data processing
pre/post validation
tools around Ansible
```

---

# 74. Choosing the Right Tool

Use:

```text
Bash
→ simple local shell task

Python
→ logic/API/data/complex automation

Ansible
→ configuration across hosts

Terraform
→ infrastructure provisioning

Kubernetes
→ container orchestration

Argo CD
→ GitOps deployment
```

A strong DevOps engineer chooses the simplest correct tool.

---

# 75. Automation Wrapper

Python can orchestrate existing tools:

```text
Terraform
 ↓
Ansible
 ↓
kubectl
 ↓
health checks
```

But it should not hide important failures.

---

# 76. Example Orchestrator

```python
import subprocess

def run(command):
    subprocess.run(
        command,
        check=True,
    )

run(
    ["terraform", "plan"]
)

run(
    ["ansible-playbook", "site.yml"]
)

run(
    ["kubectl", "rollout", "status",
     "deployment/myapp"]
)
```

Add validation, logging, timeout handling, and reporting for production use.

---

# 77. Automation Pipelines

A common pattern:

```text
Input
 ↓
Validate
 ↓
Plan
 ↓
Approve
 ↓
Execute
 ↓
Verify
 ↓
Report
```

This is safer than:

```text
Input
 ↓
Execute
```

---

# 78. Human Approval

For high-risk production operations:

```text
automation
 ↓
plan
 ↓
approval
 ↓
execute
```

Python can generate the plan/report but should not necessarily bypass organizational approval.

---

# 79. Audit Trail

Record:

```text
who
what
when
where
why
result
```

Example:

```text
user=jenkins
action=restart
service=nginx
host=prod-web-01
result=success
```

Do not record secrets.

---

# 80. Change ID

For production automation, include:

```text
ticket
change ID
deployment ID
commit SHA
```

This helps correlate automation with approved changes.

---

# 81. Automation Naming

Good:

```text
backup_database
rotate_logs
validate_deployment
check_service
collect_diagnostics
```

Avoid:

```text
run_stuff
do_it
fix_server
```

Names should describe intent.

---

# 82. Functions

Break automation into small functions:

```python
def validate():
    ...

def backup():
    ...

def apply():
    ...

def verify():
    ...

def rollback():
    ...
```

This improves testing and readability.

---

# 83. `main()`

A clean automation entry point:

```python
def main():
    validate()
    backup()
    apply()
    verify()

if __name__ == "__main__":
    main()
```

---

# 84. Return Values

Functions should return useful data:

```python
return {
    "status": "PASS",
    "changed": True,
}
```

This is easier to aggregate than printing everything.

---

# 85. Print vs Logging

Use:

```text
print
```

for intended CLI output.

Use:

```text
logging
```

for operational diagnostics.

Don't mix them without a clear reason.

---

# 86. Logging Configuration

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format=(
        "%(asctime)s "
        "%(levelname)s "
        "%(message)s"
    ),
)

logger = logging.getLogger(
    __name__
)
```

---

# 87. Log Rotation

Long-running automation should avoid filling the disk with logs.

Use:

```text
systemd journal
logrotate
RotatingFileHandler
centralized logging
```

depending on the deployment model.

---

# 88. Temporary Files

Use:

```python
from tempfile import (
    NamedTemporaryFile,
)

with NamedTemporaryFile(
    mode="w",
    delete=True,
) as file:
    file.write("data")
```

Temporary files should not contain secrets unless required and properly protected.

---

# 89. Cleanup

Use:

```python
try:
    do_work()
finally:
    cleanup()
```

This ensures cleanup even when the operation fails.

---

# 90. Context Managers

Prefer context managers for resources:

```python
with open(
    "file.txt",
    encoding="utf-8",
) as file:
    data = file.read()
```

Resources are closed automatically.

---

# 91. Resource Management

Automation can leak:

```text
file descriptors
processes
temporary files
network connections
threads
```

Always define ownership and cleanup.

---

# 92. File Locks

For scheduled automation, use a lock if concurrent execution would be unsafe.

Example:

```text
cron starts run 1
 ↓
run 1 still running
 ↓
cron starts run 2
 ↓
lock prevents duplicate execution
```

---

# 93. Automation Concurrency

For multiple independent tasks:

```python
from concurrent.futures import (
    ThreadPoolExecutor,
)
```

can provide bounded parallelism.

---

# 94. ThreadPoolExecutor

Example:

```python
def process(host):
    return check_host(host)

with ThreadPoolExecutor(
    max_workers=10
) as pool:

    results = list(
        pool.map(
            process,
            hosts,
        )
    )
```

Do not choose an unlimited worker count.

---

# 95. CPU-Bound Automation

Threads may not provide the expected speedup for CPU-heavy Python work because of the GIL in standard CPython.

For CPU-heavy processing consider:

```text
multiprocessing
ProcessPoolExecutor
native tools
distributed processing
```

---

# 96. I/O-Bound Automation

Threads are often useful for:

```text
SSH
HTTP
file I/O
API calls
```

because the workload spends time waiting.

---

# 97. AsyncIO

For very high-volume network automation:

```text
asyncio
```

can reduce thread overhead.

But complexity should be justified by scale.

---

# 98. Rate Limiting

When calling APIs:

```text
AWS
GitHub
Kubernetes
Slack
PagerDuty
```

respect rate limits.

Use:

```text
bounded concurrency
backoff
retry-after
batching
caching
```

---

# 99. API Automation

Example:

```python
import urllib.request

request = urllib.request.Request(
    "https://api.example.com/status"
)

with urllib.request.urlopen(
    request,
    timeout=5,
) as response:

    data = response.read()
```

For complex APIs, an approved HTTP client library may be more convenient.

---

# 100. HTTP Status Handling

Check:

```text
2xx success
4xx client/request problem
5xx server problem
```

and handle redirects/timeouts according to the API contract.

---

# 101. API Authentication

Common mechanisms:

```text
Bearer token
API key
OAuth
AWS SigV4
mTLS
```

Credentials should come from a secure source.

---

# 102. API Retry

Retry:

```text
429
temporary 5xx
network timeout
```

when the operation is safe to retry.

Do not blindly retry:

```text
400
401
403
```

unless the API contract indicates a transient cause.

---

# 103. Pagination

Cloud and SaaS APIs often paginate results.

Automation must not assume:

```text
one API response = all resources
```

Implement pagination correctly.

---

# 104. Idempotent API Operations

Safer automation:

```text
PUT desired resource
```

may be naturally idempotent depending on API semantics.

Riskier:

```text
POST create
```

may create duplicates if retried.

Use idempotency keys when supported.

---

# 105. Automation and JSON

Python makes JSON handling simple:

```python
import json

data = json.loads(
    response_text
)
```

Validate the expected fields before using them.

---

# 106. Automation and YAML

YAML is convenient for:

```text
configuration
inventory
automation rules
environment settings
```

Use a safe loader:

```python
import yaml

with open(
    "config.yaml",
    encoding="utf-8",
) as file:

    config = yaml.safe_load(file)
```

Never use unsafe YAML loading on untrusted content.

---

# 107. Configuration Schema

For larger automation, validate:

```text
required fields
types
ranges
allowed values
```

before execution.

---

# 108. Environment Validation

Example:

```python
allowed = {
    "dev",
    "staging",
    "production",
}

if environment not in allowed:
    raise ValueError(
        "Invalid environment"
    )
```

---

# 109. Production Guardrails

For destructive commands:

```python
if environment == "production":
    require_confirmation()
```

But interactive confirmation is not enough for unattended CI/CD.

Use:

```text
explicit approval
environment allowlist
change ticket
protected pipeline
```

---

# 110. Production Confirmation

Avoid:

```text
Are you sure? y/n
```

in automation that runs unattended.

Instead:

```bash
--environment production \
--approved-change CHG-1234
```

and validate the approval through the surrounding workflow.

---

# 111. Safe Defaults

Good defaults:

```text
read-only
dry-run
limited scope
short timeout
no destructive action
```

Bad defaults:

```text
production
force
delete
restart-all
```

---

# 112. Scope Limiting

A dangerous script:

```bash
cleanup --all
```

Better:

```bash
cleanup \
    --host app01 \
    --path /var/tmp \
    --older-than 7d
```

Automation should make scope explicit.

---

# 113. Allowlisting

Example:

```python
ALLOWED_SERVICES = {
    "nginx",
    "myapp",
}

if service not in ALLOWED_SERVICES:
    raise ValueError(
        "Service not allowed"
    )
```

---

# 114. Denylisting Is Not Enough

Avoid relying only on:

```text
if "rm" not in command
```

Attackers can use alternate syntax.

Prefer structured arguments and allowlists.

---

# 115. Path Traversal

Dangerous:

```python
base / user_input
```

if the input can contain:

```text
../../etc/passwd
```

Validate and resolve paths.

---

# 116. Safe Path Validation

Concept:

```python
base = Path(
    "/opt/backups"
).resolve()

target = (
    base / filename
).resolve()

if base not in target.parents:
    raise ValueError(
        "Path escapes backup directory"
    )
```

---

# 117. Backup Automation

Automation often handles:

```text
database dumps
configuration backups
application artifacts
logs
```

Always define:

```text
source
destination
retention
encryption
verification
restore procedure
```

---

# 118. Backup Does Not Mean Restore

A backup is only useful if it can be restored.

Automate:

```text
backup
 ↓
verify
 ↓
periodic restore test
```

---

# 119. Backup Verification

Check:

```text
file exists
size reasonable
checksum
archive integrity
object storage upload
```

For databases, verify that the backup can actually be restored.

---

# 120. Automation and Compression

Python can use:

```text
gzip
zipfile
tarfile
```

But for large production data, native database or system tools may be more appropriate.

---

# 121. Encryption

Sensitive backups should use approved encryption:

```text
KMS
GPG
encrypted storage
database-native encryption
```

Never invent your own encryption scheme.

---

# 122. Backup Retention

Example:

```text
daily = 7
weekly = 4
monthly = 12
```

The exact retention must follow organizational policy.

---

# 123. Log Automation

Python can automate:

```text
collection
rotation coordination
compression
archiving
error extraction
report generation
```

Avoid modifying production logs without understanding the logging system.

---

# 124. Log Parsing

Example:

```python
from pathlib import Path

for line in Path(
    "/var/log/app.log"
).read_text(
    encoding="utf-8",
    errors="replace",
).splitlines():

    if "ERROR" in line:
        print(line)
```

For large logs, stream line-by-line rather than loading the entire file into memory.

---

# 125. Streaming Large Logs

```python
with open(
    "/var/log/app.log",
    encoding="utf-8",
    errors="replace",
) as file:

    for line in file:
        if "ERROR" in line:
            print(line.rstrip())
```

---

# 126. Log Rotation

Do not write custom log rotation unless required.

Linux commonly provides:

```text
logrotate
journald
```

Application frameworks may provide their own rotation.

Python can orchestrate or validate the configuration.

---

# 127. Notification Automation

Automation often sends:

```text
email
Slack
Teams
PagerDuty
SNS
webhook
```

A notification should contain:

```text
what failed
where
when
severity
run ID
next action
```

---

# 128. Avoid Notification Spam

Use:

```text
deduplication
cooldowns
severity
state changes
```

rather than notifying on every loop iteration.

---

# 129. Example Notification Payload

```json
{
  "environment": "production",
  "host": "app01",
  "severity": "CRITICAL",
  "message": "Root filesystem at 94%",
  "run_id": "health-123"
}
```

---

# 130. Automation Metrics

Track:

```text
execution count
success count
failure count
duration
retry count
```

This allows you to monitor the automation itself.

---

# 131. Automation Reliability

Measure:

```text
success rate
mean duration
failure rate
timeout rate
```

An automation system should have operational visibility.

---

# 132. Automation Testing

At minimum test:

```text
happy path
invalid input
permission failure
command failure
timeout
partial failure
rollback
```

---

# 133. Unit Tests

Example:

```python
def test_is_production():
    assert (
        is_production("production")
        is True
    )
```

Keep business logic separate from side effects.

---

# 134. Mocking Commands

Instead of executing:

```text
systemctl
```

during every unit test, mock the executor.

This makes tests:

```text
fast
repeatable
safe
```

---

# 135. Integration Tests

Use a controlled environment to test:

```text
real files
real services
real network
real APIs
```

where necessary.

---

# 136. Test Containers

Containers can provide isolated environments for:

```text
Linux commands
HTTP services
databases
configuration tests
```

depending on the project.

---

# 137. CI Testing

Example:

```text
Git push
 ↓
lint
 ↓
unit tests
 ↓
integration tests
 ↓
security scan
 ↓
package
```

Do not deploy untested automation into production.

---

# 138. Security Scanning

Useful Python security tooling may include:

```text
Bandit
pip-audit
dependency scanners
SAST tools
```

Use the tools approved by your organization.

---

# 139. Dependency Vulnerabilities

A small automation script can become a production security risk if a dependency contains a known vulnerability.

Keep dependencies:

```text
minimal
pinned/controlled
updated
scanned
```

---

# 140. Version Control

All meaningful automation should be stored in Git.

Track:

```text
code
configuration
tests
documentation
runbooks
```

---

# 141. Code Review

Review automation for:

```text
security
idempotency
failure handling
scope
permissions
rollback
logging
```

A script that can modify production deserves the same engineering discipline as application code.

---

# 142. Documentation

Document:

```text
purpose
usage
inputs
outputs
permissions
examples
failure behavior
rollback
```

---

# 143. README

Example:

```text
# Deployment Validator

Checks:
- service state
- port
- HTTP health
- version

Usage:
python validate.py --config prod.yaml
```

---

# 144. Runbook

Example:

```text
If validation fails:

1. Check JSON report.
2. Identify failed check.
3. Open linked runbook.
4. Check recent deployment.
5. Inspect logs/metrics.
6. Roll back if approved.
```

---

# 145. Automation Ownership

Define:

```text
owner
maintainer
on-call team
repository
runbook
```

Unowned automation becomes dangerous technical debt.

---

# 146. Automation Lifecycle

```text
build
 ↓
test
 ↓
review
 ↓
deploy
 ↓
monitor
 ↓
maintain
 ↓
retire
```

Remove automation that no longer serves a purpose.

---

# 147. Common Mistake — Hardcoded Credentials

Bad:

```python
PASSWORD = "secret123"
```

Never commit credentials.

---

# 148. Common Mistake — Hardcoded Production Host

Bad:

```python
HOST = "prod-server-01"
```

Prefer:

```text
inventory
configuration
environment
service discovery
```

---

# 149. Common Mistake — No Timeout

Bad:

```python
requests.get(url)
```

with no timeout.

A network operation can hang indefinitely.

---

# 150. Common Mistake — Infinite Retry

Bad:

```python
while True:
    try:
        run()
        break
    except Exception:
        continue
```

This can create an endless loop.

---

# 151. Common Mistake — Catch Everything

Bad:

```python
try:
    run()
except Exception:
    pass
```

This hides failures.

---

# 152. Common Mistake — No Verification

Bad:

```text
restart service
→ print "success"
```

Better:

```text
restart
→ check service
→ check port
→ check health
→ report
```

---

# 153. Common Mistake — Shell Injection

Bad:

```python
os.system(
    f"rm -rf {path}"
)
```

Use safe APIs and validated paths.

---

# 154. Common Mistake — `shell=True`

Avoid it when structured subprocess arguments can be used.

---

# 155. Common Mistake — Parsing Human Output

Bad:

```python
output.split()[7]
```

Prefer:

```text
Python API
JSON
machine-readable output
```

---

# 156. Common Mistake — No Dry Run

For configuration-changing automation, dry run is extremely useful.

---

# 157. Common Mistake — No Backup

Before changing critical configuration:

```text
backup
```

when a rollback copy is appropriate.

---

# 158. Common Mistake — No Rollback

For high-risk automation, define:

```text
how to restore previous state
```

before executing.

---

# 159. Common Mistake — No Audit Trail

You should be able to answer:

```text
who changed what?
when?
on which host?
why?
what was the result?
```

---

# 160. Common Mistake — Running as Root

Do not default to root simply because it makes the script easier.

Use least privilege.

---

# 161. Common Mistake — One Giant Script

A 3,000-line script with:

```text
AWS
SSH
backup
deployment
monitoring
notifications
```

is difficult to test.

Split responsibilities into modules.

---

# 162. Separation of Concerns

Example:

```text
config.py
executor.py
checks.py
backup.py
notify.py
report.py
cli.py
```

---

# 163. Common Mistake — Mixing Logic and Side Effects

Hard to test:

```python
def deploy():
    run_command()
    print()
    send_email()
    write_file()
```

Better:

```text
business logic
+
execution layer
+
notification layer
```

---

# 164. Common Mistake — Ignoring Partial Failure

Multi-host automation must report:

```text
success
failure
skipped
unreachable
```

for every target.

---

# 165. Common Mistake — Unbounded Parallelism

Don't create:

```python
ThreadPoolExecutor(
    max_workers=1000
)
```

without understanding the target systems and API limits.

---

# 166. Common Mistake — No Rate Limiting

Cloud APIs and SaaS platforms enforce limits.

Respect:

```text
429
Retry-After
service quotas
```

---

# 167. Common Mistake — Blind Retry

Retries can duplicate side effects.

Understand whether an operation is idempotent before retrying it.

---

# 168. Common Mistake — No Locking

Scheduled jobs can overlap and corrupt:

```text
backup
state
temporary files
deployment
```

Use locking when needed.

---

# 169. Common Mistake — Unsafe Cleanup

Never automate deletion without:

```text
scope
retention policy
dry run
logging
verification
```

---

# 170. Common Mistake — Trusting User Input

Validate:

```text
path
host
service
URL
environment
resource ID
```

---

# 171. Common Mistake — Secrets in Logs

Use redaction.

For example:

```text
Authorization: [REDACTED]
```

---

# 172. Common Mistake — Secrets in Command Line

Command-line arguments can sometimes be visible to other processes.

Prefer secure credential injection mechanisms where possible.

---

# 173. Common Mistake — Assuming Local Behavior Equals CI

CI may have:

```text
different PATH
different user
different filesystem
different permissions
different network
```

Test in an environment close to production.

---

# 174. Common Mistake — Ignoring OS Differences

Commands can differ across:

```text
Amazon Linux
RHEL
Ubuntu
Debian
Alpine
```

Use Python APIs for portable functionality where possible.

---

# 175. Common Mistake — Assuming systemd Everywhere

Containers and minimal distributions may not run systemd.

Detect the environment before relying on it.

---

# 176. Common Mistake — Ignoring Containers

Inside a container:

```text
PID 1
filesystem
network
service manager
```

may behave differently from a full VM.

---

# 177. Container Automation

Python can automate:

```text
docker CLI
Docker API
Kubernetes API
```

but should follow the platform's source of truth.

---

# 178. Container Image Automation

Common workflow:

```text
build
 ↓
scan
 ↓
tag
 ↓
push
 ↓
deploy
 ↓
verify
```

Python can orchestrate steps, but dedicated CI/CD tools often provide better native capabilities.

---

# 179. DevSecOps Automation

Python automation can integrate:

```text
SonarQube
Trivy
Veracode
dependency scanning
secret scanning
```

as pipeline steps.

Security checks should fail or warn according to defined policy.

---

# 180. Automation Gates

Example:

```text
Build
 ↓
SAST
 ↓
SCA
 ↓
Container scan
 ↓
Approval
 ↓
Deploy
 ↓
Health
```

Python can generate reports and enforce custom policy.

---

# 181. Policy Automation

Example:

```text
if critical_vulnerabilities > 0:
    fail
```

Better when the policy is:

```text
version controlled
reviewed
audited
```

---

# 182. Automation Reports

Generate:

```text
JSON
CSV
Markdown
HTML
```

depending on the consumer.

---

# 183. Markdown Report

Example:

```text
# Deployment Report

Environment: production
Version: 2.4.1

## Checks

- Disk: PASS
- Service: PASS
- HTTP: PASS
- Version: PASS

Overall: PASS
```

---

# 184. Automation Artifacts

CI can retain:

```text
logs
JSON reports
screenshots
configuration snapshots
backup metadata
```

Avoid storing secrets in artifacts.

---

# 185. Automation and GitHub Actions

Typical workflow:

```yaml
- name: Run validation
  run: |
    python automation.py \
      --environment staging
```

The Python process controls success/failure through exit status.

---

# 186. Automation and Jenkins

Typical:

```text
Checkout
 ↓
Install dependencies
 ↓
Run Python automation
 ↓
Publish report
 ↓
Gate deployment
```

---

# 187. Automation and GitLab CI

Same model:

```text
job
 ↓
python script
 ↓
exit code
 ↓
artifact/report
```

---

# 188. Automation and Argo CD

Python can perform:

```text
post-sync validation
external smoke testing
release verification
```

while Argo CD remains responsible for GitOps reconciliation.

---

# 189. Automation and Terraform

Example:

```text
terraform plan
 ↓
Python policy validation
 ↓
approval
 ↓
terraform apply
 ↓
Python post-check
```

This can be useful when custom validation is needed.

---

# 190. Automation and Ansible

Example:

```text
Python preflight
 ↓
Ansible configuration
 ↓
Python validation
```

Ansible remains the configuration engine.

---

# 191. Automation and Monitoring

Example:

```text
Python detects custom condition
 ↓
structured event
 ↓
Prometheus/ELK/notification system
```

Do not create duplicate alerting logic unnecessarily.

---

# 192. Event-Driven Automation

Modern systems can trigger automation from:

```text
webhook
message queue
CloudWatch/EventBridge
GitHub event
Jenkins event
Kubernetes event
```

The handler should:

```text
validate event
process safely
be idempotent
log result
```

---

# 193. Webhook Security

Validate:

```text
signature
source
timestamp
payload schema
event type
```

Do not trust incoming webhook payloads blindly.

---

# 194. Idempotent Webhook Processing

If the same event arrives twice:

```text
event_id = 123
```

the automation should avoid performing the same destructive action twice.

Use:

```text
event IDs
deduplication storage
idempotency keys
```

---

# 195. Queue-Based Automation

Example:

```text
Event
 ↓
Queue
 ↓
Python worker
 ↓
Validate
 ↓
Execute
 ↓
Report
```

Queues can provide:

```text
buffering
retry
scaling
decoupling
```

---

# 196. Dead-Letter Queue

Failed automation messages may be moved to:

```text
DLQ
```

for investigation instead of retrying forever.

---

# 197. Automation State

Some workflows need state:

```text
started
in progress
completed
failed
rolled back
```

Persist state when the workflow can outlive a single process.

---

# 198. State Storage

Possible:

```text
DynamoDB
S3
database
Redis
Git
CI artifact
```

Choose based on durability and concurrency requirements.

---

# 199. Stateless Automation

Prefer stateless scripts when possible.

Example:

```text
input
 ↓
execute
 ↓
output
```

Stateless automation is easier to retry and scale.

---

# 200. Statefulness

Use persistent state only when needed:

```text
long-running workflow
deduplication
approval
checkpoint
rollback
```

---

# 201. Checkpointing

For large automation:

```text
1000 hosts
```

record:

```text
completed hosts
failed hosts
pending hosts
```

so a retry can resume safely.

---

# 202. Batch Processing

Instead of:

```text
1000 hosts at once
```

use:

```text
batch 1 = 20
batch 2 = 20
...
```

This limits blast radius.

---

# 203. Rolling Automation

```text
Batch 1
 ↓
validate
 ↓
Batch 2
 ↓
validate
 ↓
Batch 3
```

Stop when failure exceeds the defined threshold.

---

# 204. Blast Radius

Good automation controls:

```text
how many hosts can change at once
```

This is critical in production.

---

# 205. Canary Automation

Start with:

```text
1 host
```

Then:

```text
5%
25%
50%
100%
```

if the deployment process supports it.

---

# 206. Rollback Strategy

Rollback may be:

```text
restore config
restore artifact
deploy previous version
revert Git commit
restore backup
```

Choose the rollback mechanism before deployment.

---

# 207. Rollback Verification

After rollback:

```text
service
health
version
dependencies
```

must be verified.

---

# 208. Automation Observability

Every significant automation should expose:

```text
start time
end time
duration
target
action
result
failure reason
```

---

# 209. Correlation ID

Example:

```text
automation_run=run-20260817-001
```

Include it in logs and reports.

---

# 210. Host Identity

Use stable identifiers:

```text
instance ID
hostname
Kubernetes node name
account/region
```

rather than relying only on IP addresses.

---

# 211. Cloud Region

AWS automation should record:

```text
account
region
resource ID
environment
```

This prevents ambiguity.

---

# 212. Multi-Account Automation

For AWS organizations:

```text
management
dev
staging
production
```

automation should explicitly select the intended account.

Never infer production solely from a resource name.

---

# 213. Multi-Region Automation

Control:

```text
region
account
resource scope
concurrency
```

when operating across regions.

---

# 214. Production Safety Check

Before destructive AWS automation:

```text
Who am I?
Which account?
Which region?
Which resources?
What is the change?
```

Print or log the scope safely before execution.

---

# 215. AWS Account Guardrail

Concept:

```python
expected_account = "123456789012"

actual_account = get_current_account()

if actual_account != expected_account:
    raise RuntimeError(
        "Unexpected AWS account"
    )
```

Do not hardcode sensitive identifiers in public repositories; use approved configuration.

---

# 216. Environment Guardrail

Example:

```python
if environment == "production":
    require_approved_change()
```

The approval mechanism should be external and auditable.

---

# 217. Automation and IAM

Use:

```text
role assumption
temporary credentials
least privilege
session tags
```

where supported.

Avoid long-lived keys.

---

# 218. Automation and S3

Python can automate:

```text
upload
download
backup
inventory
metadata
retention
```

Use:

```python
boto3.client("s3")
```

and handle:

```text
timeouts
permissions
retries
checksums
```

---

# 219. Automation and ECR

Python can validate:

```text
repository exists
image tag exists
image digest
scan state
```

before deployment.

---

# 220. Automation and EKS

Python can validate:

```text
cluster reachable
nodes ready
deployment rollout
pods ready
services available
ingress endpoint
```

after infrastructure/application changes.

---

# 221. Automation and Route53

Automation can validate:

```text
record exists
expected target
TTL
routing policy
```

before or after changes.

Be cautious with automated DNS modifications because blast radius can be large.

---

# 222. Automation and ALB

Validate:

```text
listener
target groups
target health
HTTP response
```

rather than only checking that the load balancer exists.

---

# 223. Automation and RDS

Validate:

```text
instance status
endpoint DNS
port
connection
storage/metrics
```

according to the application requirement.

---

# 224. Automation and Secrets

Automation can retrieve secrets but should avoid:

```text
printing secret
writing secret to temporary files
embedding secret in command
```

unless explicitly required and protected.

---

# 225. Secret Redaction

Implement:

```python
def redact(value):
    return "[REDACTED]"
```

for sensitive fields.

For real systems, use structured logging filters and secret-aware tooling.

---

# 226. Automation Output

Separate:

```text
human output
machine output
```

Example:

```bash
--json
```

for machine consumption.

---

# 227. Machine-Friendly Exit Codes

CI systems should not parse:

```text
"SUCCESS"
```

from stdout if an exit code can communicate success/failure reliably.

---

# 228. Automation Contract

Define:

```text
Inputs
Outputs
Exit codes
Side effects
Permissions
Timeouts
Retries
```

This makes automation predictable.

---

# 229. Semantic Versioning

If the automation is a reusable tool:

```text
MAJOR
MINOR
PATCH
```

can communicate compatibility changes.

---

# 230. Backward Compatibility

Avoid changing:

```text
CLI flags
JSON schema
exit code meaning
configuration format
```

without considering downstream consumers.

---

# 231. CLI Help

Always provide:

```bash
python tool.py --help
```

with:

```text
description
arguments
examples
```

---

# 232. Example CLI

```bash
python deploy.py \
    --environment staging \
    --version 2.4.1 \
    --dry-run
```

---

# 233. Configuration Precedence

Define a predictable order:

```text
defaults
 ↓
config file
 ↓
environment variables
 ↓
CLI arguments
```

Document it.

---

# 234. Configuration Validation Before Execution

Load and validate all configuration before changing anything.

This prevents:

```text
half the workflow executed
then configuration error
```

---

# 235. Plan Phase

For complex automation:

```text
parse
 ↓
validate
 ↓
calculate changes
 ↓
display plan
 ↓
execute
```

This is similar to infrastructure-as-code workflows.

---

# 236. Plan Output

Example:

```text
PLAN

CREATE /opt/app
UPDATE /etc/myapp.conf
RESTART myapp
VERIFY HTTPS

No changes have been made.
```

---

# 237. Execute Phase

Only after validation:

```text
apply planned changes
```

Keep plan and execution logic separate where practical.

---

# 238. Verify Phase

Verification should validate:

```text
desired state
```

not simply:

```text
command completed
```

---

# 239. Report Phase

Final report:

```text
CHANGED: 3
UNCHANGED: 5
FAILED: 1
SKIPPED: 2
```

This is especially useful for fleet automation.

---

# 240. Change Tracking

Track:

```text
changed
unchanged
failed
skipped
```

rather than only:

```text
success/failure
```

---

# 241. Idempotency Report

Example:

```text
app01: CHANGED
app02: UNCHANGED
app03: CHANGED
```

This shows what the automation actually modified.

---

# 242. Automation Metrics

Example:

```text
automation_runs_total
automation_failures_total
automation_duration_seconds
automation_changes_total
```

These can be sent to a monitoring system if appropriate.

---

# 243. Automation SLO

For a production automation service:

```text
99.9% successful executions
```

may be a meaningful target.

For a one-off script, formal SLOs may be unnecessary.

---

# 244. Dependency Health

Automation itself depends on:

```text
network
AWS APIs
SSH
DNS
Git
Kubernetes API
package repositories
```

Failures in these dependencies should be classified correctly.

---

# 245. Dependency Failure Example

If:

```text
AWS API timeout
```

the automation should report:

```text
AWS dependency unavailable
```

rather than:

```text
resource does not exist
```

---

# 246. Distinguish Not Found from Unreachable

Important:

```text
404 / ResourceNotFound
```

is different from:

```text
timeout
```

Do not convert both to the same message.

---

# 247. Retry Classification

Classify errors:

```text
retryable
non-retryable
unknown
```

Then apply retries only where appropriate.

---

# 248. Error Taxonomy

Example:

```python
class AutomationError(Exception):
    pass

class ValidationError(
    AutomationError
):
    pass

class DependencyError(
    AutomationError
):
    pass

class ExecutionError(
    AutomationError
):
    pass
```

This can make failure handling clearer.

---

# 249. Exit Code Taxonomy

Example:

```text
0 = success
1 = execution failure
2 = validation failure
3 = dependency unavailable
4 = usage/configuration error
```

Document the contract.

---

# 250. Error Messages

Bad:

```text
failed
```

Better:

```text
Failed to restart nginx on app01:
service exited with code 1
```

Best:

```text
Failed to restart nginx on app01:
configuration validation returned exit code 1.
See /var/log/nginx/error.log.
```

Avoid leaking sensitive information.

---

# 251. Automation Recovery

When possible:

```text
detect failure
 ↓
collect diagnostics
 ↓
rollback
 ↓
verify rollback
 ↓
report
```

---

# 252. Diagnostic Collection

On failure collect relevant:

```text
service status
recent logs
configuration checksum
resource metrics
deployment version
```

Do not collect everything indiscriminately.

---

# 253. Failure Bundle

Example:

```text
incident/
├── health.json
├── service-status.txt
├── recent-errors.txt
├── version.txt
└── metadata.json
```

Store according to retention and security policy.

---

# 254. Avoid Sensitive Diagnostics

A diagnostic bundle can accidentally contain:

```text
environment variables
tokens
credentials
private keys
```

Sanitize before uploading or sharing.

---

# 255. Automation and Incident Response

Python is useful for:

```text
collecting diagnostics
checking fleet state
comparing versions
finding failed services
creating reports
```

Be cautious with automatic remediation during incidents.

---

# 256. Incident Automation

Good first response:

```text
detect
collect
summarize
notify
```

Then:

```text
remediate
```

only when the procedure is well understood.

---

# 257. Runbook Automation

A runbook step:

```text
Check nginx status
```

can become:

```text
Python automation
```

that gathers:

```text
status
recent logs
port
configuration validation
```

This reduces manual investigation time.

---

# 258. ChatOps

Automation can integrate with:

```text
Slack
Teams
```

for commands such as:

```text
/health app01
```

But command authorization and audit logging are critical.

---

# 259. ChatOps Security

Never allow:

```text
/slack execute arbitrary shell
```

Use:

```text
allowlisted actions
RBAC
approval
audit
```

---

# 260. Event Automation

Example:

```text
Cloud event
 ↓
Lambda/Python
 ↓
validate
 ↓
action
 ↓
report
```

Python is often used in AWS Lambda for lightweight automation.

---

# 261. Lambda Automation

Good use cases:

```text
resource tagging
event processing
reporting
scheduled checks
cleanup with guardrails
```

Be mindful of:

```text
timeout
cold start
permissions
idempotency
```

---

# 262. Lambda Timeout

Always design within the function timeout.

For long-running jobs:

```text
queue
Step Functions
ECS
EC2
worker
```

may be more appropriate.

---

# 263. Scheduled AWS Automation

Possible triggers:

```text
EventBridge
cron-like schedule
```

Then:

```text
Lambda
 ↓
Python
 ↓
AWS API
```

---

# 264. Automation and Step Functions

For multi-step workflows:

```text
validate
 ↓
backup
 ↓
change
 ↓
verify
 ↓
notify
```

Step Functions can provide durable workflow orchestration.

Python can implement individual steps.

---

# 265. When Not to Use Python

Do not automatically choose Python for:

```text
simple one-line command
standard Terraform resource
standard Ansible configuration
simple CI command
```

Use the simplest appropriate tool.

---

# 266. Automation Decision Matrix

```text
Task
 |
 +-- shell only? → Bash
 |
 +-- infrastructure? → Terraform
 |
 +-- host configuration? → Ansible
 |
 +-- Kubernetes deployment? → GitOps/Kubernetes
 |
 +-- complex logic/API/data? → Python
 |
 +-- continuous metrics? → Prometheus
```

Tools complement each other.

---

# 267. Automation Maturity

Level 1:

```text
manual commands
```

Level 2:

```text
shell scripts
```

Level 3:

```text
Python automation
```

Level 4:

```text
idempotent, tested automation
```

Level 5:

```text
event-driven, observable, policy-controlled automation
```

---

# 268. Level 1 — Manual

Example:

```text
SSH
run commands
copy output
```

High human effort.

---

# 269. Level 2 — Script

```text
script.sh
```

Repeatable but may have limited error handling.

---

# 270. Level 3 — Python

```text
configuration
logging
exceptions
APIs
structured output
```

More maintainable.

---

# 271. Level 4 — Production Automation

Add:

```text
tests
idempotency
dry run
rollback
observability
security
CI/CD
```

---

# 272. Level 5 — Platform Automation

Add:

```text
event-driven
policy
approval
fleet scaling
workflow state
centralized reporting
```

---

# 273. Daily DevOps Automation Examples

Python scripts commonly automate:

```text
check disk
check services
restart approved service
backup configs
parse logs
validate deployment
compare versions
check AWS resources
query Kubernetes
send alerts
generate reports
```

---

# 274. Daily Script — Disk Report

```python
import shutil

usage = shutil.disk_usage("/")

percent = (
    usage.used / usage.total
) * 100

print(
    f"Root disk: {percent:.1f}%"
)
```

---

# 275. Daily Script — Service Check

```python
import subprocess

result = subprocess.run(
    ["systemctl", "is-active", "nginx"],
    capture_output=True,
    text=True,
    check=False,
)

if result.stdout.strip() == "active":
    print("nginx: PASS")
else:
    print("nginx: FAIL")
```

---

# 276. Daily Script — Health Endpoint

```python
import urllib.request

try:
    with urllib.request.urlopen(
        "http://localhost/health",
        timeout=5,
    ) as response:

        print(
            "HTTP:",
            response.status,
        )

except Exception as exc:
    print(
        "Health check failed:",
        exc,
    )
```

Use specific exceptions in production.

---

# 277. Daily Script — Process Check

```python
import psutil

target = "nginx"

running = any(
    process.info["name"] == target
    for process in psutil.process_iter(
        ["name"]
    )
)

print(
    f"{target}: "
    f"{'RUNNING' if running else 'NOT RUNNING'}"
)
```

For systemd services, service-state checks are usually stronger.

---

# 278. Daily Script — File Age

```python
from pathlib import Path
import time

path = Path(
    "/opt/app/backup.tar.gz"
)

age = (
    time.time()
    - path.stat().st_mtime
)

print(
    f"Age: {age / 3600:.1f} hours"
)
```

Useful for backup freshness checks.

---

# 279. Daily Script — Configuration Checksum

```python
import hashlib
from pathlib import Path

data = Path(
    "/etc/myapp.conf"
).read_bytes()

checksum = hashlib.sha256(
    data
).hexdigest()

print(checksum)
```

Useful for configuration drift detection.

---

# 280. Daily Script — Directory Size

For a simple check:

```python
from pathlib import Path

total = sum(
    item.stat().st_size
    for item in Path(
        "/var/log/myapp"
    ).rglob("*")
    if item.is_file()
)

print(
    total / 1024**3,
    "GB",
)
```

For very large filesystems, native tools or filesystem metrics may be more efficient.

---

# 281. Daily Script — Recent Errors

```python
from pathlib import Path

log = Path(
    "/var/log/myapp/app.log"
)

with log.open(
    encoding="utf-8",
    errors="replace",
) as file:

    for line in file:
        if "ERROR" in line:
            print(line.rstrip())
```

---

# 282. Daily Script — Version Check

```python
import subprocess

result = subprocess.run(
    ["myapp", "--version"],
    capture_output=True,
    text=True,
    check=False,
)

print(
    result.stdout.strip()
)
```

---

# 283. Daily Script — Command Wrapper

```python
import subprocess

def run(command):
    result = subprocess.run(
        command,
        capture_output=True,
        text=True,
        check=False,
    )

    if result.returncode != 0:
        raise RuntimeError(
            result.stderr.strip()
        )

    return result.stdout
```

Centralizing execution makes logging and error handling easier.

---

# 284. Better Command Wrapper

A production wrapper can include:

```text
timeout
logging
duration
return code
stdout/stderr
redaction
```

---

# 285. Automation Utility Module

Example:

```text
automation/
├── cli.py
├── config.py
├── executor.py
├── logging.py
├── retry.py
├── validators.py
└── reporters.py
```

Reusable utilities reduce duplication.

---

# 286. Configuration Loader

```python
from pathlib import Path
import yaml

def load_config(path):
    with Path(path).open(
        encoding="utf-8"
    ) as file:

        return yaml.safe_load(
            file
        )
```

Add schema validation for production use.

---

# 287. Environment Validator

```python
def validate_environment(
    value
):
    allowed = {
        "dev",
        "staging",
        "production",
    }

    if value not in allowed:
        raise ValueError(
            f"Unsupported environment: {value}"
        )
```

---

# 288. Path Validator

```python
from pathlib import Path

def validate_path(
    base,
    target,
):
    base = Path(base).resolve()
    target = Path(target).resolve()

    if base not in target.parents:
        raise ValueError(
            "Target outside allowed directory"
        )
```

---

# 289. Retry Decorator Concept

Reusable retry logic can be implemented as:

```python
def retry(
    attempts=3,
    delay=1,
):
    ...
```

But keep the behavior explicit and avoid retrying unsafe operations.

---

# 290. Backoff Helper

```python
def backoff(
    attempt,
    base=1,
):
    return base * (
        2 ** attempt
    )
```

Add jitter and a maximum delay in production.

---

# 291. Timing Helper

```python
import time

start = time.monotonic()

run_task()

duration = (
    time.monotonic()
    - start
)

print(
    f"{duration:.2f}s"
)
```

Use `monotonic()` for elapsed durations.

---

# 292. Why `monotonic()`?

Wall-clock time can change because of:

```text
NTP
manual clock changes
timezone adjustments
```

`time.monotonic()` is appropriate for measuring elapsed time.

---

# 293. Automation Timestamp

For event timestamps:

```python
from datetime import datetime, timezone

timestamp = datetime.now(
    timezone.utc
)
```

Use timezone-aware datetimes.

---

# 294. Automation State File

For small local tools:

```text
/var/lib/myautomation/state.json
```

can store:

```text
last run
last success
last version
```

Protect the file appropriately.

---

# 295. State Corruption

Write state safely:

```text
temporary file
 ↓
flush
 ↓
atomic replace
```

so a process crash does not leave invalid JSON.

---

# 296. Lock File

For local automation:

```text
/var/run/myautomation.lock
```

Use a proper locking mechanism rather than simply checking whether a file exists.

---

# 297. Process Locking

A file existence check has a race:

```text
process A checks → absent
process B checks → absent
A creates
B creates
```

Use OS-level file locking where required.

---

# 298. Scheduling

Automation can run via:

```text
cron
systemd timer
Jenkins
GitHub Actions
GitLab CI
EventBridge
Kubernetes CronJob
```

Choose based on the environment.

---

# 299. Kubernetes CronJob

Python automation can run as:

```text
Kubernetes CronJob
```

Useful for:

```text
periodic reports
cleanup
validation
batch tasks
```

Add:

```text
concurrencyPolicy
resource limits
timeouts
retry policy
```

as appropriate.

---

# 300. CronJob Safety

Avoid overlapping jobs:

```yaml
concurrencyPolicy: Forbid
```

when overlap would be unsafe.

---

# 301. Resource Limits

Automation running in Kubernetes should define:

```text
requests
limits
```

so the automation itself does not become a resource problem.

---

# 302. Job Retry

Kubernetes Jobs can retry failed Pods.

Make sure the Python operation is idempotent before enabling aggressive retries.

---

# 303. Automation Containers

Package Python automation in a container:

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY src/ .

CMD ["python", "main.py"]
```

Pin approved base images and dependencies according to organizational policy.

---

# 304. Container Security

Use:

```text
non-root user
minimal base image
read-only filesystem where possible
no unnecessary capabilities
dependency scanning
```

---

# 305. Automation Image

Example:

```text
Python automation
 ↓
Docker image
 ↓
ECR
 ↓
EKS CronJob
```

This is a practical cloud-native pattern.

---

# 306. Automation CI/CD

```text
Git
 ↓
CI
 ↓
Lint
 ↓
Tests
 ↓
Security scan
 ↓
Build image
 ↓
Push ECR
 ↓
Deploy
 ↓
Verify
```

---

# 307. Automation Repository

Recommended:

```text
python-automation/
├── src/
├── tests/
├── config/
├── Dockerfile
├── requirements.txt
├── README.md
└── .gitignore
```

---

# 308. `.gitignore`

Never commit:

```text
.env
credentials
private keys
local state
logs
venv
```

---

# 309. Automation Secrets in Git

If a secret is accidentally committed:

```text
remove from repository
rotate credential
check history
audit usage
```

Deleting the latest file does not necessarily remove it from Git history.

---

# 310. Secret Scanning

Use:

```text
GitHub secret scanning
TruffleHog
Gitleaks
organization-approved scanners
```

to detect accidental secrets.

---

# 311. Automation Code Review Checklist

Review:

```text
[ ] Input validation
[ ] No secrets
[ ] Least privilege
[ ] Idempotency
[ ] Timeout
[ ] Retry policy
[ ] Logging
[ ] Verification
[ ] Rollback
[ ] Tests
[ ] Dry run
[ ] Scope control
```

---

# 312. Production Automation Checklist

Before release:

```text
[ ] Code reviewed
[ ] Tests pass
[ ] Security scan passes
[ ] Dependencies controlled
[ ] Configuration validated
[ ] Permissions approved
[ ] Dry run tested
[ ] Failure path tested
[ ] Rollback tested
[ ] Monitoring configured
[ ] Runbook written
```

---

# 313. Automation Design Example

Requirement:

```text
Restart myapp only if it is unhealthy.
```

Bad:

```python
systemctl restart myapp
```

Better:

```text
check service
 ↓
check HTTP
 ↓
if unhealthy:
    capture diagnostics
    restart once
    verify
    notify if still unhealthy
```

---

# 314. Implementation Flow

```python
health = check_health()

if health.ok:
    return

collect_diagnostics()

restart_service()

if not check_health().ok:
    notify_failure()
    raise SystemExit(1)
```

This is much safer than unconditional restart.

---

# 315. Automation Decision Tree

```text
Is system healthy?
       |
      YES
       |
      STOP

       NO
       |
Collect evidence
       |
Is automatic remediation approved?
       |
   +---+---+
   |       |
  NO      YES
   |       |
Alert    Remediate
           |
         Verify
           |
      +----+----+
      |         |
    PASS       FAIL
      |         |
    Done       Alert
```

---

# 316. Automation Runbook Integration

Every remediation should reference:

```text
runbook URL/document
```

The operator should understand:

```text
why
what
risk
rollback
```

---

# 317. Blast Radius Control

For fleet changes:

```text
1 host
 ↓
verify
 ↓
5 hosts
 ↓
verify
 ↓
25%
 ↓
verify
 ↓
100%
```

This is safer than changing everything at once.

---

# 318. Production Change Example

Suppose you need to update:

```text
nginx configuration
```

Safe flow:

```text
fetch config
 ↓
backup
 ↓
render desired config
 ↓
validate nginx -t
 ↓
atomic replace
 ↓
reload nginx
 ↓
verify service
 ↓
verify HTTP
```

---

# 319. Why Validate Before Reload?

If configuration is invalid:

```text
nginx -t FAIL
```

you can stop before affecting the running service.

---

# 320. Rollback Example

```text
new config
 ↓
validation PASS
 ↓
reload
 ↓
health FAIL
 ↓
restore backup
 ↓
reload
 ↓
health PASS
```

This is a classic production automation pattern.

---

# 321. Configuration Management Principle

Automation should make:

```text
desired state
```

explicit.

Avoid scripts that simply execute:

```text
random sequence of commands
```

without understanding state.

---

# 322. Automation State Example

```python
if service_is_active():
    print("UNCHANGED")
else:
    start_service()
    print("CHANGED")
```

---

# 323. Changed vs Unchanged

This distinction is important for:

```text
audit
performance
reporting
idempotency
```

---

# 324. Check Mode

A reusable automation tool can support:

```bash
--check
```

to inspect state without changes.

Equivalent conceptually to:

```text
dry run
```

---

# 325. Force Mode

If a force option exists:

```bash
--force
```

it should be explicit and carefully restricted.

Never make force the default.

---

# 326. Confirmation Token

For production operations:

```bash
--confirm-production
```

can act as an additional guardrail, but should not replace proper authorization.

---

# 327. Two-Person Approval

High-risk operations may require:

```text
engineer A proposes
engineer B approves
automation executes
```

Python can enforce the workflow if integrated with an approval system.

---

# 328. Automation and Compliance

Automation should support:

```text
audit logs
change records
access control
approval
retention
```

where required.

---

# 329. Compliance Evidence

Generate:

```text
who ran
what changed
when
target
result
```

as machine-readable evidence.

---

# 330. Automation and Disaster Recovery

Automate:

```text
backup validation
restore testing
failover checks
DNS validation
service health
```

but test recovery procedures regularly.

---

# 331. DR Automation

Example:

```text
primary unhealthy
 ↓
validate secondary
 ↓
verify data freshness
 ↓
approval
 ↓
switch traffic
 ↓
verify
```

Do not automatically fail over without understanding data consistency and business requirements.

---

# 332. Backup Freshness

A health check can validate:

```text
last backup age
```

Example:

```text
last backup = 3 hours ago
RPO = 1 hour
FAIL
```

---

# 333. RPO/RTO

Automation should understand:

```text
RPO
=
how much data loss is acceptable

RTO
=
how quickly service must recover
```

These influence backup and recovery automation.

---

# 334. Automation and Database Backup

Typical:

```text
backup
 ↓
compress/encrypt
 ↓
upload
 ↓
verify
 ↓
record metadata
```

A production design must also consider restore testing.

---

# 335. Automation and Logs

Typical:

```text
detect log condition
 ↓
collect
 ↓
compress/archive
 ↓
upload
 ↓
verify
```

Do not delete the only copy before successful archival.

---

# 336. Automation and Notifications

Notification should happen after:

```text
result known
```

not simply:

```text
command started
```

---

# 337. Notification Severity

Example:

```text
INFO
WARN
CRITICAL
RECOVERY
```

Only route CRITICAL to paging if the condition warrants immediate human action.

---

# 338. Notification Content

Include:

```text
service
environment
host
error
timestamp
run ID
impact
next action
```

Avoid enormous stack traces in chat messages.

---

# 339. Notification Deduplication

Use an alert key:

```text
production:app01:nginx-down
```

so repeated failures can be grouped.

---

# 340. Notification Recovery

When the condition clears:

```text
RECOVERY: nginx is healthy
```

This closes the operational loop.

---

# 341. Automation Performance

Measure:

```text
CPU
memory
network
execution time
API calls
```

A script should not consume more resources than the task justifies.

---

# 342. Large File Processing

Use streaming:

```python
for line in file:
    ...
```

instead of:

```python
data = file.read()
```

for very large files.

---

# 343. Large API Results

Use pagination:

```text
page 1
page 2
page 3
```

instead of assuming everything fits into memory.

---

# 344. Large Host Lists

Process in batches:

```text
batch 1
batch 2
...
```

rather than loading and executing everything simultaneously.

---

# 345. Memory-Safe Automation

Prefer:

```text
generators
streaming
pagination
batching
```

for large workloads.

---

# 346. Generator Example

```python
def read_hosts(path):
    with open(
        path,
        encoding="utf-8",
    ) as file:

        for line in file:
            host = line.strip()

            if host:
                yield host
```

---

# 347. Batch Processing

```python
def batches(items, size):
    for index in range(
        0,
        len(items),
        size,
    ):
        yield items[
            index:index + size
        ]
```

---

# 348. Automation Performance Bottleneck

Before optimizing, measure.

Potential bottlenecks:

```text
SSH connection setup
API latency
disk I/O
serialization
sequential execution
```

---

# 349. Connection Reuse

For multiple API calls, reuse clients/sessions when supported.

This can reduce:

```text
TCP/TLS setup
latency
resource usage
```

---

# 350. SSH Connection Reuse

For many remote commands on the same host, use a persistent connection where the SSH library supports it rather than reconnecting for every command.

---

# 351. Automation Reliability

Reliable automation has:

```text
bounded execution
clear state
safe retries
verification
observability
```

---

# 352. Automation Determinism

Given:

```text
same input
same environment
```

automation should generally produce predictable behavior.

Avoid:

```text
hidden random state
uncontrolled external dependencies
```

unless required.

---

# 353. Randomness

If randomness is used for:

```text
jitter
IDs
temporary names
```

make its purpose explicit.

Do not use weak randomness for security-sensitive tokens.

---

# 354. Secure Randomness

For security tokens use:

```python
import secrets
```

rather than:

```python
import random
```

---

# 355. Automation and Cryptographic Hashes

Use:

```text
SHA-256
```

for integrity checks where appropriate.

Do not use MD5 as a security mechanism.

---

# 356. Checksum Validation

After transferring a file:

```text
source checksum
=
destination checksum
```

This confirms content integrity.

---

# 357. File Transfer Automation

Typical:

```text
upload
 ↓
checksum
 ↓
permissions
 ↓
ownership
 ↓
validation
```

---

# 358. Configuration Deployment

Never assume copying a file means configuration is valid.

Use:

```text
copy
 ↓
validate
 ↓
activate/reload
 ↓
verify
```

---

# 359. Application Deployment

Typical:

```text
artifact
 ↓
verify
 ↓
backup current
 ↓
deploy
 ↓
restart/reload
 ↓
health
 ↓
rollback if required
```

---

# 360. Release Metadata

Record:

```text
artifact
version
digest
commit
build ID
deployment ID
```

This makes deployments traceable.

---

# 361. Container Digest

Prefer immutable identifiers such as:

```text
image digest
```

when the deployment system supports them.

Tags can move.

---

# 362. Automation and Artifact Integrity

Before deployment:

```text
expected digest
=
actual digest
```

This reduces accidental or unauthorized artifact changes.

---

# 363. Automation and Security Scanning

Before deployment:

```text
image exists
scan complete
policy passes
digest verified
```

Then deploy.

---

# 364. Automation and Trivy

Python can consume Trivy output in JSON:

```bash
trivy image \
    --format json \
    myimage:tag
```

Then enforce policy based on:

```text
severity
fix availability
approved exceptions
```

---

# 365. Automation and SonarQube

Python can query an API to retrieve:

```text
quality gate
coverage
issues
```

but CI integration should remain the primary enforcement point when available.

---

# 366. Automation and Veracode

Python can integrate with approved APIs to retrieve:

```text
scan status
policy status
findings
```

and produce deployment gates.

---

# 367. Automation Policy Engine

A reusable policy function:

```python
def allowed_to_deploy(
    findings
):
    return not any(
        item["severity"]
        == "CRITICAL"
        for item in findings
    )
```

Real policy should support exceptions and organizational rules.

---

# 368. Policy as Code

Store rules in:

```text
Git
```

and review them like application code.

---

# 369. Automation Documentation as Code

Keep:

```text
README
runbooks
config schema
examples
```

in the same repository when practical.

---

# 370. Automation Repository Branching

Use your organization's Git workflow:

```text
feature
 ↓
PR
 ↓
review
 ↓
main
 ↓
release
```

Avoid manually editing production scripts on servers.

---

# 371. Immutable Automation

Prefer:

```text
build artifact
 ↓
deploy artifact
```

rather than:

```text
edit script directly on server
```

This improves reproducibility.

---

# 372. Automation Packaging

Possible package:

```text
system-health-1.2.0-py3-none-any.whl
```

or:

```text
automation-image:1.2.0
```

---

# 373. Versioned Releases

Tag:

```bash
git tag v1.2.0
```

Then deploy that exact version.

---

# 374. Rollback Automation Version

If:

```text
automation v1.3.0
```

is faulty:

```text
rollback to v1.2.0
```

This is another reason not to edit scripts directly on servers.

---

# 375. Automation Compatibility

Test across supported:

```text
Python versions
Linux distributions
AWS environments
Kubernetes versions
```

only as needed by your support matrix.

---

# 376. Python Version

For production:

```text
pin supported Python version
```

and test dependencies against it.

Do not assume the latest interpreter is available everywhere.

---

# 377. Virtual Environment

For host-based automation:

```bash
python3 -m venv /opt/automation/.venv
```

This avoids dependency conflicts.

---

# 378. System Package vs Pip

Decide whether dependencies are managed through:

```text
OS packages
pip/venv
container image
internal package repository
```

and use one consistent strategy.

---

# 379. Offline Environments

Some production networks have limited internet access.

Prepare:

```text
internal package repository
wheelhouse
container registry
```

for dependencies.

---

# 380. Automation Bootstrap

A bootstrap process can:

```text
install Python
create venv
install dependencies
install tool
configure service
run self-test
```

Keep bootstrap scripts small and auditable.

---

# 381. Bootstrap Idempotency

Running bootstrap twice should not:

```text
duplicate configuration
break dependencies
reset production state
```

---

# 382. Automation Upgrade

Upgrade flow:

```text
validate new version
 ↓
deploy
 ↓
self-test
 ↓
health check
 ↓
keep or rollback
```

---

# 383. Self-Test

A tool can verify:

```text
Python version
dependencies
config
permissions
network access
```

before starting the real workflow.

---

# 384. Health Endpoint for Automation Service

If automation runs as a service:

```text
/health
/ready
/metrics
```

can expose its own operational state.

---

# 385. Automation Worker Health

Track:

```text
queue depth
jobs running
job failures
job duration
worker heartbeat
```

---

# 386. Queue Backlog

If:

```text
queue depth increasing
```

but:

```text
workers healthy
```

you may need:

```text
more workers
faster processing
dependency investigation
```

---

# 387. Automation Scaling

Scale based on:

```text
queue depth
job latency
worker utilization
API limits
```

not simply CPU.

---

# 388. Backpressure

Automation should slow down when downstream systems are overloaded.

Examples:

```text
API throttling
database overload
SSH saturation
```

Use:

```text
rate limits
bounded concurrency
queues
```

---

# 389. Circuit Breaker

For repeatedly failing dependencies:

```text
closed
 ↓ failures
open
 ↓ cooldown
half-open
 ↓ success
closed
```

This prevents hammering a broken dependency.

---

# 390. Circuit Breaker Use

Useful for:

```text
external API
database
notification service
cloud service
```

when automation can otherwise generate repeated load.

---

# 391. Automation Dependency Timeout

Example:

```text
Slack timeout
```

should not necessarily cause:

```text
backup failure
```

unless notification is a critical requirement.

Separate critical and non-critical dependencies.

---

# 392. Notification Failure

If:

```text
automation succeeds
notification fails
```

report both:

```text
TASK = SUCCESS
NOTIFICATION = FAILED
```

Do not incorrectly report the core task as failed.

---

# 393. Multi-Stage Result

Example:

```json
{
  "task": "backup",
  "status": "PASS",
  "notification": "FAIL"
}
```

This is more accurate.

---

# 394. Criticality Model

Define dependencies as:

```text
critical
optional
```

For example:

```text
database = critical
Slack = optional
```

---

# 395. Automation Workflow

```text
Start
 ↓
Validate
 ↓
Precheck
 ↓
Backup
 ↓
Change
 ↓
Verify
 ↓
Report
 ↓
Notify
```

---

# 396. Failure Workflow

```text
Failure
 ↓
Classify
 ↓
Collect diagnostics
 ↓
Rollback if approved
 ↓
Verify
 ↓
Report
 ↓
Notify
```

---

# 397. Automation State Diagram

```text
PENDING
   |
   v
VALIDATING
   |
   v
READY
   |
   v
RUNNING
   |
 +--+--+
 |     |
PASS  FAIL
 |     |
 v     v
DONE  RECOVERY
        |
      +--+--+
      |     |
    PASS   FAIL
      |     |
     DONE  FAILED
```

---

# 398. Final Automation Principles

Remember:

```text
1. Automate repeatable work.
2. Define desired state.
3. Validate before changing.
4. Make operations idempotent.
5. Use least privilege.
6. Never hardcode secrets.
7. Use timeouts.
8. Retry only safe operations.
9. Limit blast radius.
10. Verify every important change.
11. Keep rollback possible.
12. Log useful information.
13. Never log secrets.
14. Test failure paths.
15. Version automation.
16. Monitor the automation itself.
17. Use the right tool for the job.
18. Prefer simple automation over clever automation.
```

---

# 399. Interview Questions — Automation Fundamentals

## Q1. Why do you use Python for DevOps automation?

**Answer:**

> I use Python when automation requires more than simple shell execution, especially for API integrations, structured data processing, complex logic, validation, error handling, reporting, and reusable tooling. I still use Bash for small Linux tasks and use Terraform, Ansible, Kubernetes, and GitOps where those tools are the appropriate source of truth.

---

## Q2. What is idempotency?

**Answer:**

> Idempotency means that running the same automation multiple times results in the same desired final state without causing unintended duplicate changes. For example, creating a directory with `exist_ok=True` is idempotent because the second execution doesn't create another directory or fail simply because it already exists.

---

## Q3. How do you make a Python script production-ready?

**Answer:**

> I add configuration management, input validation, structured logging, exception handling, timeouts, safe retries, idempotency, dry-run support where appropriate, verification, rollback or recovery, tests, security controls, clear exit codes, documentation, and CI/CD validation.

---

## Q4. How do you execute Linux commands from Python?

**Answer:**

> I normally use `subprocess.run()` with a list of arguments and `shell=False`. I capture output when needed, inspect the return code, set a timeout, and handle expected exceptions.

---

## Q5. Why avoid `os.system()`?

**Answer:**

> `subprocess` provides better control over arguments, return codes, stdout, stderr, timeouts, and exceptions. It also makes it easier to avoid shell interpretation and command-injection risks.

---

## Q6. What is command injection?

**Answer:**

> Command injection happens when untrusted input is interpreted as part of a shell command and causes unintended commands to execute. I avoid constructing shell commands from raw user input, use structured subprocess arguments, and validate inputs against allowlists when necessary.

---

## Q7. When would you use Bash instead of Python?

**Answer:**

> For a short, local Linux task with simple command composition, Bash is often simpler. If the task grows to require complex logic, APIs, structured data, testing, reusable components, or significant error handling, I prefer Python.

---

## Q8. How do you handle command failures?

**Answer:**

> I inspect the return code or use `check=True` when a non-zero exit should raise an exception. I log the relevant context, classify the failure, perform cleanup or rollback when appropriate, and return a non-zero exit status to the caller.

---

## Q9. Why are timeouts important?

**Answer:**

> External commands and network operations can hang indefinitely. A timeout creates a bounded execution window and prevents one dependency from blocking the entire automation workflow.

---

## Q10. How do you design retries?

**Answer:**

> I retry only transient and safe operations, use bounded attempts, exponential backoff and jitter where appropriate, and distinguish retryable from non-retryable failures. I avoid retrying non-idempotent operations blindly.

---

## Q11. What is a dry run?

**Answer:**

> A dry run calculates and reports what the automation would change without actually applying the changes. It is useful for production review, testing, and reducing accidental changes.

---

## Q12. How do you verify automation success?

**Answer:**

> I validate the resulting desired state rather than trusting command completion. For example, after updating a service configuration, I validate the configuration, reload the service, check the service state, verify the listening port, and test the application health endpoint.

---

## Q13. How do you handle partial failures?

**Answer:**

> For independent targets I continue processing while recording each result. For critical sequential workflows I fail fast. The final report distinguishes changed, unchanged, failed, skipped, and unreachable targets.

---

## Q14. How do you secure Python automation?

**Answer:**

> I use least privilege, avoid hardcoded credentials, validate input, avoid unsafe shell execution, use secure secret storage, protect logs and artifacts, control dependencies, limit network access, and audit production changes.

---

## Q15. How do you handle secrets?

**Answer:**

> I retrieve them from approved secret-management systems or CI/CD secret stores and only expose them to the process when required. I never commit them to Git or print them in logs.

---

## Q16. What is the difference between Terraform, Ansible, and Python?

**Answer:**

> Terraform is primarily for infrastructure provisioning and desired infrastructure state. Ansible is strong for host configuration and multi-host orchestration. Python is a general-purpose automation language that is especially useful for custom logic, API integration, validation, reporting, and orchestration around those tools.

---

## Q17. How would you automate a production configuration change?

**Answer:**

> I would validate the target and configuration, generate a plan, back up the current configuration, validate the new configuration, apply it atomically, reload or restart only when required, verify service and application health, and retain a rollback path.

---

## Q18. How do you prevent automation from changing the wrong environment?

**Answer:**

> I explicitly identify the environment and account, validate the target against an allowlist or approved inventory, use separate credentials/roles, require appropriate production approval, and log the target context before executing the change.

---

## Q19. How would you automate across 1,000 servers?

**Answer:**

> I would use inventory, bounded concurrency, batching, timeouts, retry policies, idempotent operations, checkpointing, structured results, and progressive rollout. I would start with a canary or small batch and expand only after validation.

---

## Q20. How do you prevent a script from running twice at the same time?

**Answer:**

> I use an appropriate locking mechanism or scheduler-level concurrency control. A simple existence check on a lock file is not sufficient because it has a race condition.

---

# 400. Interview Scenario — Production Restart Automation

**Question:**

A production application occasionally becomes unhealthy. Would you write a Python script that restarts it automatically?

**Strong answer:**

> I would not immediately implement unconditional restarts. First I would define the health signal and collect diagnostics. If automatic remediation is approved, I would make it bounded: detect the unhealthy state, collect evidence, restart at most once or according to a controlled policy, verify the application, and alert if it remains unhealthy. I would also prevent restart loops and keep the remediation auditable.

---

# 401. Interview Scenario — Deployment Script Failed Halfway

**Question:**

Your Python deployment script updated three servers and failed on the fourth. What do you do?

**Answer:**

> I would preserve the per-host state and identify exactly what changed. I would not blindly rerun the entire script. I would determine whether the operation is idempotent, retry the failed host if safe, and use rollback for already changed hosts if the deployment policy requires consistency. For larger fleets I would use batches and checkpoints.

---

# 402. Interview Scenario — Python Script Hangs

**Question:**

A Python automation script hangs during production deployment.

**Answer:**

> I would identify which operation is blocking, inspect the logs and process state, and terminate the run safely if necessary. Then I would add explicit timeouts to subprocess and network operations. If the operation is retryable, I would use bounded retries rather than allowing indefinite blocking.

---

# 403. Interview Scenario — AWS Script Deleted the Wrong Resources

**Question:**

How would you prevent this?

**Answer:**

> I would add account and region validation, explicit resource selection, dry-run/plan mode, allowlists, least-privilege IAM, protected production credentials, approval gates, and audit logging. For high-risk changes I would start with a canary or very small scope.

---

# 404. Interview Scenario — Automation Keeps Retrying

**Question:**

What could be wrong?

**Answer:**

> The script may be retrying non-transient errors, have no maximum retry count, lack backoff, or not distinguish between validation failures and dependency failures. I would classify errors and make the retry policy explicit.

---

# 405. Interview Scenario — Service Restart Succeeded but Application Is Still Down

**Answer:**

> The service state is only one layer of validation. I would check the process, port, application health endpoint, logs, dependencies, and external traffic path. I would not consider the deployment successful until the required application-level health checks pass.

---

# 406. Interview Scenario — Disk Cleanup Automation

**Question:**

A production server is at 95% disk usage. Would you automatically delete old files?

**Answer:**

> Only if an approved retention and cleanup policy already exists. I would first identify the filesystem, inode usage, largest consumers, logs, container storage, and deleted-open files. Any cleanup should be scoped, logged, reversible where possible, and verified afterward.

---

# 407. Interview Scenario — Python vs Ansible

**Question:**

You need to install and configure nginx on 100 servers. What would you use?

**Answer:**

> I would normally use Ansible because this is a classic multi-host configuration-management task. Python could be used for custom validation, inventory processing, reporting, or orchestration around Ansible, but I would not replace a mature configuration-management tool with custom Python unnecessarily.

---

# 408. Interview Scenario — Python vs Terraform

**Question:**

You need to create 20 AWS VPC resources.

**Answer:**

> I would use Terraform because infrastructure provisioning and desired-state management are its strengths. Python could support preflight validation or custom reporting, but I would keep Terraform as the infrastructure source of truth.

---

# 409. Interview Scenario — Python vs Bash

**Question:**

You need to check whether nginx is active and return an exit code.

**Answer:**

> A small Bash command may be the simplest solution. I would use Python if this check is becoming part of a larger automation framework that needs configuration, structured output, API integration, testing, or multiple related checks.

---

# 410. Interview Scenario — Automation Security

**Question:**

A developer asks you to create a Python script that accepts a service name and runs `systemctl restart <service>`. What do you consider?

**Answer:**

> I would validate the service name against an allowlist and pass it as a structured subprocess argument. I would avoid `shell=True`, require appropriate permissions, log the action without secrets, and define which services the automation is actually allowed to restart.

---

# 411. Interview Scenario — API Rate Limit

**Question:**

Your Python script receives HTTP 429 from AWS or another API.

**Answer:**

> I would respect the service's retry guidance, use exponential backoff with jitter, limit concurrency, and avoid hammering the API. I would also distinguish throttling from authentication or validation failures.

---

# 412. Interview Scenario — Backup Automation

**Question:**

How would you design a production backup script?

**Answer:**

> I would define the backup source, destination, retention, encryption, naming, integrity verification, monitoring, and restore procedure. After creating the backup I would verify it exists and is valid. Periodically I would perform a restore test because a backup that has never been restored is not proven recoverable.

---

# 413. Interview Scenario — Log Automation

**Question:**

A 20 GB application log needs error extraction.

**Answer:**

> I would stream the file line by line rather than loading it into memory. For repeated production processing I would also consider centralized logging such as ELK rather than repeatedly scanning large files locally.

---

# 414. Interview Scenario — Production Configuration

**Question:**

How would you safely update nginx configuration using Python?

**Answer:**

> I would render the desired configuration, preserve a backup, write the new file safely, run `nginx -t`, and only then reload nginx. After reload I would verify the service and application endpoint. If health checks fail, I would restore the known-good configuration and verify recovery.

---

# 415. Interview Scenario — Multi-Host Deployment

**Question:**

How do you avoid taking down all servers?

**Answer:**

> I would use a rolling or canary strategy. Start with one host, validate it, then process small batches while monitoring health. I would stop the rollout if the failure rate exceeds the predefined threshold.

---

# 416. Interview Scenario — Automation Failure During Production Incident

**Question:**

Should automation automatically remediate every incident?

**Answer:**

> No. Automation should be proportional to the confidence and risk. Detection and diagnostic collection are generally safer to automate. Remediation should be automated only when the failure mode is well understood, the action is safe and idempotent, the blast radius is controlled, and rollback is defined.

---

# 417. Interview Scenario — How Would You Explain Python Automation in Your DevOps Project?

**Answer:**

> I used Python mainly for operational automation around Linux and cloud environments. I created scripts for system health checks, command execution, configuration validation, log processing, deployment verification, and API-based checks. I designed them with logging, exception handling, timeouts, structured output, and safe execution so they could integrate with CI/CD and production operations.

---

# 418. Interview Scenario — Your Automation Runs Successfully but Changes Nothing

**Answer:**

> That can be a valid idempotent outcome. I would report `UNCHANGED` separately from `CHANGED`. The important question is whether the desired state was already satisfied and whether verification passed.

---

# 419. Interview Scenario — How Do You Measure Automation Quality?

**Answer:**

> I look at successful execution rate, failure rate, execution duration, retry rate, false positives, rollback frequency, security findings, and operational impact. For production automation I also monitor whether the automation itself is healthy and observable.

---

# 420. Interview Scenario — What Makes DevOps Automation Production-Grade?

**Answer:**

> Production-grade automation is predictable, idempotent, secure, observable, tested, bounded by timeouts and scope, safe to retry, capable of verifying results, and designed with recovery or rollback. It is also version-controlled and integrated into an auditable delivery process.

---

# 421. Final Summary

Python automation is not primarily about executing more commands.

It is about building a reliable operational workflow:

```text
                    AUTOMATION
                         |
                 +-------+-------+
                 |               |
             Desired State    Inputs
                 |               |
                 +-------+-------+
                         |
                     Validate
                         |
                       Plan
                         |
                     Execute
                         |
                     Verify
                         |
                +--------+--------+
                |                 |
             SUCCESS            FAILURE
                |                 |
              Report        Diagnose/Rollback
                |                 |
                +--------+--------+
                         |
                      Observe
```

The DevOps mindset is:

> **Automate repeatable work, but engineer the automation as carefully as production software.**

That means:

```text
Idempotency
+
Security
+
Validation
+
Timeouts
+
Retries
+
Observability
+
Verification
+
Rollback
+
Testing
+
Controlled blast radius
```

These principles will be reused throughout the remaining Python automation files:

```text
02-Command-Automation.md
03-File-and-Directory-Automation.md
04-Configuration-Automation.md
05-Backup-Automation.md
06-Log-Automation.md
07-Notification-Automation.md
```
