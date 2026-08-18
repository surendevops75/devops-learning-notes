# Command-Automation

## Python Command Automation for DevOps

Command automation is one of the most common uses of Python in Linux and DevOps.

A DevOps engineer regularly needs to automate commands such as:

```text
systemctl
docker
kubectl
helm
terraform
ansible-playbook
git
curl
grep
find
df
du
ss
ip
journalctl
```

The important skill is not simply knowing how to execute a command from Python.

The production skill is knowing how to:

```text
construct
validate
execute
capture
interpret
retry
timeout
verify
report
```

commands safely.

---

# 1. What Is Command Automation?

Command automation means using Python to execute and control operating-system commands.

Basic flow:

```text
Python
  ↓
Build command
  ↓
Validate inputs
  ↓
Execute
  ↓
Capture result
  ↓
Interpret exit code
  ↓
Verify desired state
  ↓
Report
```

---

# 2. Why Automate Commands?

Manual:

```text
SSH
 ↓
type command
 ↓
read output
 ↓
repeat
```

Python:

```text
input
 ↓
automation
 ↓
repeatable execution
 ↓
structured result
```

Benefits:

```text
consistency
repeatability
speed
logging
validation
error handling
CI/CD integration
```

---

# 3. Python Command Execution Options

Common approaches:

```text
subprocess.run()
subprocess.Popen()
subprocess.check_output()
subprocess.check_call()
os.system()
os.popen()
```

For modern DevOps automation, prefer:

```text
subprocess
```

---

# 4. `subprocess.run()`

The most common interface:

```python
import subprocess

subprocess.run(
    ["df", "-h", "/"],
    check=True,
)
```

---

# 5. Why Use a List of Arguments?

Prefer:

```python
[
    "systemctl",
    "restart",
    "nginx",
]
```

over:

```python
"systemctl restart nginx"
```

The list makes argument boundaries explicit and avoids unnecessary shell interpretation.

---

# 6. Basic Command

```python
import subprocess

result = subprocess.run(
    ["hostname"],
    capture_output=True,
    text=True,
)

print(result.stdout)
```

---

# 7. Return Code

```python
print(result.returncode)
```

Typically:

```text
0 = success
non-zero = failure
```

The exact meaning of non-zero codes depends on the command.

---

# 8. `check=True`

```python
subprocess.run(
    ["systemctl", "restart", "nginx"],
    check=True,
)
```

If the command exits non-zero:

```text
subprocess.CalledProcessError
```

is raised.

---

# 9. `check=False`

Useful when the return code is part of normal decision-making:

```python
result = subprocess.run(
    ["systemctl", "is-active", "nginx"],
    capture_output=True,
    text=True,
    check=False,
)

if result.returncode == 0:
    print("Service active")
```

---

# 10. Capture stdout

```python
result = subprocess.run(
    ["hostname"],
    capture_output=True,
    text=True,
)

hostname = result.stdout.strip()
```

---

# 11. Capture stderr

```python
result = subprocess.run(
    ["systemctl", "status", "nginx"],
    capture_output=True,
    text=True,
)

print(result.stderr)
```

Some commands write useful diagnostics to stderr even when the operation itself behaves as expected.

---

# 12. `capture_output=True`

Equivalent conceptually to:

```python
stdout=subprocess.PIPE
stderr=subprocess.PIPE
```

It is convenient when you need both streams.

---

# 13. `text=True`

Without `text=True`:

```python
result.stdout
```

may be bytes.

With:

```python
text=True
```

Python decodes command output into strings using the configured/default text encoding behavior.

---

# 14. Explicit Encoding

When the environment requires predictable encoding:

```python
result = subprocess.run(
    ["hostname"],
    capture_output=True,
    text=True,
    encoding="utf-8",
)
```

---

# 15. Command Timeout

Never let an external command wait forever:

```python
subprocess.run(
    ["curl", "https://example.com"],
    timeout=10,
    check=True,
)
```

---

# 16. Timeout Handling

```python
try:
    subprocess.run(
        ["curl", "https://example.com"],
        timeout=10,
        check=True,
    )
except subprocess.TimeoutExpired:
    print("Command timed out")
```

In production, log the command context safely and decide whether retrying is appropriate.

---

# 17. `CalledProcessError`

```python
try:
    subprocess.run(
        ["systemctl", "restart", "myapp"],
        check=True,
    )
except subprocess.CalledProcessError as exc:
    print(
        f"Command failed: {exc.returncode}"
    )
```

---

# 18. Command Wrapper

A reusable wrapper reduces duplicate code:

```python
import subprocess

def run_command(command):
    return subprocess.run(
        command,
        capture_output=True,
        text=True,
        check=False,
    )
```

---

# 19. Production Command Wrapper

A better wrapper can handle:

```text
timeout
logging
return code
stdout
stderr
duration
exception classification
```

---

# 20. Example Wrapper

```python
import logging
import subprocess
import time

logger = logging.getLogger(__name__)

def run_command(
    command,
    timeout=30,
):
    start = time.monotonic()

    try:
        result = subprocess.run(
            command,
            capture_output=True,
            text=True,
            timeout=timeout,
            check=False,
        )
    except subprocess.TimeoutExpired:
        logger.error(
            "Command timed out: %s",
            command[0],
        )
        raise

    duration = (
        time.monotonic() - start
    )

    logger.info(
        "Command completed rc=%s duration=%.2fs",
        result.returncode,
        duration,
    )

    return result
```

---

# 21. Never Log Secrets

Avoid:

```python
logger.info(
    "Running %s",
    command,
)
```

if the command may contain:

```text
password
token
private key
secret
authorization header
```

Use redaction.

---

# 22. Command Injection

Dangerous:

```python
import subprocess

service = input(
    "Service: "
)

subprocess.run(
    f"systemctl restart {service}",
    shell=True,
)
```

The input may be interpreted as shell syntax.

---

# 23. Safer Execution

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

Then validate the service name.

---

# 24. Allowlist Services

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

# 25. `shell=False`

This is the default for `subprocess` and is generally safer.

```python
subprocess.run(
    ["systemctl", "restart", "nginx"],
    shell=False,
)
```

Do not use `shell=True` unless shell interpretation is actually required and the input is fully controlled.

---

# 26. When Is `shell=True` Appropriate?

There are legitimate cases involving shell features such as:

```text
pipes
shell expansion
redirection
compound shell syntax
```

But prefer composing commands directly in Python where possible.

---

# 27. Safer Pipeline

Instead of:

```bash
ps -ef | grep nginx
```

use Python APIs where practical, or execute pipeline stages separately.

Example with `psutil`:

```python
import psutil

for process in psutil.process_iter(
    ["pid", "name"]
):
    if process.info["name"] == "nginx":
        print(process.info)
```

---

# 28. If a Pipeline Is Required

You can connect processes with `Popen`:

```python
import subprocess

ps = subprocess.Popen(
    ["ps", "-ef"],
    stdout=subprocess.PIPE,
    text=True,
)

grep = subprocess.Popen(
    ["grep", "nginx"],
    stdin=ps.stdout,
    stdout=subprocess.PIPE,
    text=True,
)

ps.stdout.close()

output, _ = grep.communicate()

print(output)
```

Use this only when there is a good reason; Python APIs may be clearer.

---

# 29. Why Avoid Shell Pipelines?

Direct Python logic can provide:

```text
better error handling
structured data
portability
testability
```

---

# 30. Command Existence

Before running a command:

```python
import shutil

if shutil.which("kubectl") is None:
    raise RuntimeError(
        "kubectl is not installed"
    )
```

---

# 31. Validate Dependencies

Automation may require:

```text
python
kubectl
helm
aws
terraform
git
curl
```

Check dependencies before execution.

---

# 32. Preflight Dependency Check

```python
REQUIRED = [
    "kubectl",
    "helm",
]

for command in REQUIRED:
    if shutil.which(command) is None:
        raise RuntimeError(
            f"Missing dependency: {command}"
        )
```

---

# 33. Version Validation

Check:

```bash
kubectl version --client
```

from Python:

```python
result = subprocess.run(
    ["kubectl", "version", "--client"],
    capture_output=True,
    text=True,
    check=False,
)
```

---

# 34. Why Version Checks Matter

Automation can fail because:

```text
CLI version changed
flag removed
output format changed
API compatibility changed
```

---

# 35. Prefer Structured CLI Output

If supported:

```bash
kubectl get pods -o json
```

Then parse JSON instead of human-formatted output.

---

# 36. Kubernetes JSON Example

```python
import json
import subprocess

result = subprocess.run(
    [
        "kubectl",
        "get",
        "pods",
        "-o",
        "json",
    ],
    capture_output=True,
    text=True,
    check=True,
)

data = json.loads(
    result.stdout
)
```

---

# 37. AWS CLI JSON

Example:

```python
result = subprocess.run(
    [
        "aws",
        "ec2",
        "describe-instances",
        "--output",
        "json",
    ],
    capture_output=True,
    text=True,
    check=True,
)

data = json.loads(
    result.stdout
)
```

For substantial AWS automation, prefer Boto3 rather than wrapping the AWS CLI unnecessarily.

---

# 38. Terraform Automation

Python can execute:

```text
terraform fmt
terraform validate
terraform plan
terraform apply
```

but Terraform should remain the infrastructure source of truth.

---

# 39. Terraform Validation

```python
subprocess.run(
    ["terraform", "validate"],
    check=True,
)
```

---

# 40. Terraform Plan

```python
result = subprocess.run(
    [
        "terraform",
        "plan",
        "-out=tfplan",
    ],
    capture_output=True,
    text=True,
    check=False,
)
```

Do not automatically apply based only on textual output.

---

# 41. Terraform Apply

For automation:

```text
plan
 ↓
approval
 ↓
apply
 ↓
verify
```

Use the organization's approved workflow.

---

# 42. Ansible Automation

Python can call:

```python
subprocess.run(
    [
        "ansible-playbook",
        "-i",
        "inventory",
        "site.yml",
    ],
    check=True,
)
```

Python can then perform post-deployment verification.

---

# 43. Git Automation

Common commands:

```text
git clone
git fetch
git checkout
git diff
git status
git rev-parse
```

---

# 44. Git Version

```python
result = subprocess.run(
    ["git", "--version"],
    capture_output=True,
    text=True,
    check=True,
)

print(result.stdout.strip())
```

---

# 45. Git Commit SHA

```python
result = subprocess.run(
    [
        "git",
        "rev-parse",
        "HEAD",
    ],
    capture_output=True,
    text=True,
    check=True,
)

commit = result.stdout.strip()
```

Useful for deployment metadata.

---

# 46. Git Status

```python
result = subprocess.run(
    [
        "git",
        "status",
        "--porcelain",
    ],
    capture_output=True,
    text=True,
    check=True,
)

if result.stdout.strip():
    print("Working tree has changes")
```

---

# 47. Git Diff Automation

Use:

```bash
git diff --exit-code
```

and inspect the return code.

Do not parse human output if an exit status provides the required signal.

---

# 48. Docker Automation

Python can invoke:

```text
docker build
docker tag
docker push
docker inspect
docker ps
```

For richer automation, use the Docker SDK or platform APIs where appropriate.

---

# 49. Docker Image Check

```python
result = subprocess.run(
    [
        "docker",
        "image",
        "inspect",
        "myapp:latest",
    ],
    capture_output=True,
    text=True,
    check=False,
)

if result.returncode == 0:
    print("Image exists")
```

---

# 50. Docker Build

```python
subprocess.run(
    [
        "docker",
        "build",
        "-t",
        "myapp:1.0",
        ".",
    ],
    check=True,
)
```

---

# 51. Container Security

Before pushing:

```text
build
 ↓
scan
 ↓
policy
 ↓
push
```

For example:

```text
Trivy
```

can produce machine-readable results that Python can evaluate.

---

# 52. Helm Automation

Commands:

```text
helm lint
helm template
helm upgrade
helm status
```

Use Python to orchestrate only when the surrounding workflow benefits from it.

---

# 53. Helm Validation

```python
subprocess.run(
    [
        "helm",
        "lint",
        "./chart",
    ],
    check=True,
)
```

---

# 54. Helm Template

```python
result = subprocess.run(
    [
        "helm",
        "template",
        "myapp",
        "./chart",
    ],
    capture_output=True,
    text=True,
    check=True,
)

manifest = result.stdout
```

---

# 55. Kubernetes Deployment Check

```python
subprocess.run(
    [
        "kubectl",
        "rollout",
        "status",
        "deployment/myapp",
        "--timeout=120s",
    ],
    check=True,
)
```

The command timeout is still worth controlling at the Python process level when appropriate.

---

# 56. Kubernetes Pod Diagnostics

Python can automate:

```text
kubectl get pods
kubectl describe pod
kubectl logs
kubectl get events
```

and produce a structured incident report.

---

# 57. Kubernetes Health Script

Example flow:

```text
check nodes
 ↓
check deployment
 ↓
check pods
 ↓
check services
 ↓
check ingress
 ↓
report
```

---

# 58. `kubectl` Context Safety

Before modifying Kubernetes:

```python
result = subprocess.run(
    [
        "kubectl",
        "config",
        "current-context",
    ],
    capture_output=True,
    text=True,
    check=True,
)

context = result.stdout.strip()
```

Validate that the context is expected.

---

# 59. Kubernetes Namespace Safety

Explicitly specify:

```text
-n production
```

rather than relying on the current namespace.

---

# 60. AWS Account Safety

Before AWS CLI changes, verify the caller identity:

```python
subprocess.run(
    [
        "aws",
        "sts",
        "get-caller-identity",
    ],
    check=True,
)
```

For serious automation, parse and validate the returned account/ARN.

---

# 61. AWS Region Safety

Explicitly configure:

```text
AWS_REGION
```

or pass the expected region rather than relying on an accidental default.

---

# 62. Systemd Automation

Common commands:

```text
systemctl start
systemctl stop
systemctl restart
systemctl reload
systemctl enable
systemctl is-active
systemctl is-enabled
```

---

# 63. Start Service

```python
subprocess.run(
    [
        "systemctl",
        "start",
        "nginx",
    ],
    check=True,
)
```

---

# 64. Restart vs Reload

Use:

```text
reload
```

when the service supports configuration reload without disrupting connections.

Use:

```text
restart
```

when a full process restart is required.

---

# 65. Service Verification

```python
result = subprocess.run(
    [
        "systemctl",
        "is-active",
        "nginx",
    ],
    capture_output=True,
    text=True,
    check=False,
)

if result.stdout.strip() != "active":
    raise RuntimeError(
        "nginx is not active"
    )
```

---

# 66. Enable at Boot

```python
subprocess.run(
    [
        "systemctl",
        "enable",
        "nginx",
    ],
    check=True,
)
```

---

# 67. Service Configuration Validation

A good automation sequence:

```text
render
 ↓
validate
 ↓
reload
 ↓
verify
```

Do not restart first and validate afterward.

---

# 68. Nginx Validation

```python
subprocess.run(
    ["nginx", "-t"],
    check=True,
)
```

Then:

```python
subprocess.run(
    [
        "systemctl",
        "reload",
        "nginx",
    ],
    check=True,
)
```

---

# 69. Linux Disk Commands

Useful commands:

```text
df
du
ls
find
stat
mount
```

Python can execute them, but `shutil`, `pathlib`, and `os` often provide better structured interfaces.

---

# 70. Disk Usage with Python

```python
import shutil

usage = shutil.disk_usage("/")

used_percent = (
    usage.used / usage.total
) * 100

print(
    f"{used_percent:.1f}%"
)
```

---

# 71. Memory

Instead of parsing:

```bash
free -m
```

consider:

```text
psutil
```

when an external dependency is acceptable.

---

# 72. Process Commands

Useful:

```text
ps
top
pgrep
pkill
kill
```

Use Python APIs when practical.

---

# 73. Process Termination

Do not blindly:

```bash
kill -9
```

Start with graceful termination where possible.

```text
SIGTERM
 ↓
grace period
 ↓
SIGKILL if necessary
```

---

# 74. Python Process Termination

With `psutil`:

```python
process.terminate()
```

then wait and escalate only if necessary.

---

# 75. Network Commands

Useful:

```text
ip
ss
ping
curl
dig
nslookup
traceroute
```

Python libraries may provide better control for network operations.

---

# 76. Port Check

Simple TCP connectivity:

```python
import socket

with socket.create_connection(
    ("127.0.0.1", 8080),
    timeout=3,
):
    print("Port reachable")
```

---

# 77. HTTP Check

```python
import urllib.request

with urllib.request.urlopen(
    "http://127.0.0.1:8080/health",
    timeout=5,
) as response:

    if response.status != 200:
        raise RuntimeError(
            "Health check failed"
        )
```

---

# 78. DNS Check

Python can use:

```python
import socket

ip = socket.gethostbyname(
    "example.com"
)

print(ip)
```

For advanced DNS automation, use an appropriate DNS library.

---

# 79. Curl Automation

If curl is standardized in the environment:

```python
subprocess.run(
    [
        "curl",
        "-f",
        "--max-time",
        "5",
        "https://example.com/health",
    ],
    check=True,
)
```

---

# 80. Why `curl -f`?

It causes curl to treat HTTP 4xx/5xx responses as failures, which can make CI checks clearer.

---

# 81. SSH Command Automation

For a few controlled tasks:

```text
ssh
```

can be wrapped with subprocess.

For richer multi-host automation:

```text
Paramiko
AsyncSSH
Ansible
SSM
```

may be more appropriate.

---

# 82. SSH CLI

```python
subprocess.run(
    [
        "ssh",
        "-o",
        "BatchMode=yes",
        "app01",
        "hostname",
    ],
    check=True,
)
```

Use known hosts and approved credentials.

---

# 83. SSH Timeout

```python
subprocess.run(
    [
        "ssh",
        "-o",
        "ConnectTimeout=5",
        "app01",
        "hostname",
    ],
    timeout=15,
    check=True,
)
```

---

# 84. SSH Security

Avoid:

```text
StrictHostKeyChecking=no
```

as a blanket production solution.

Host-key verification protects against man-in-the-middle attacks.

---

# 85. SSH Keys

Prefer:

```text
SSH key
```

over:

```text
password embedded in script
```

Use agent/secret-management approaches according to your environment.

---

# 86. AWS SSM

For AWS-managed instances, SSM can avoid direct SSH exposure.

Python can invoke approved SSM APIs/commands using Boto3.

---

# 87. Command Automation with SSM

Concept:

```text
Python
 ↓
AWS SSM API
 ↓
EC2 instance
 ↓
command
 ↓
result
```

This is often preferable to opening SSH access broadly.

---

# 88. Remote Command Result

Remote automation should capture:

```text
status
stdout
stderr
execution ID
target
```

---

# 89. Remote Command Timeout

Remote execution can involve multiple timeouts:

```text
connection timeout
command timeout
polling timeout
overall workflow timeout
```

Define them separately.

---

# 90. Polling

For asynchronous remote commands:

```text
submit
 ↓
get execution ID
 ↓
poll status
 ↓
complete/fail/timeout
```

Avoid infinite polling.

---

# 91. Polling Backoff

Instead of:

```text
poll every 1ms
```

use:

```text
1s
2s
4s
8s
```

up to a maximum interval.

---

# 92. Command Output Size

Commands can produce huge output.

Avoid keeping unlimited stdout in memory.

Use:

```text
streaming
file redirection
bounded output
```

where appropriate.

---

# 93. `Popen`

Use `subprocess.Popen()` when you need:

```text
streaming output
interactive process control
long-running processes
multiple connected processes
```

---

# 94. Streaming Output

```python
import subprocess

process = subprocess.Popen(
    ["journalctl", "-u", "nginx", "-n", "50"],
    stdout=subprocess.PIPE,
    stderr=subprocess.STDOUT,
    text=True,
)

for line in process.stdout:
    print(line.rstrip())

return_code = process.wait()
```

---

# 95. Process Lifecycle

With `Popen`:

```text
start
 ↓
communicate/read
 ↓
wait
 ↓
inspect return code
```

Always ensure child processes are cleaned up.

---

# 96. `communicate()`

Use:

```python
stdout, stderr = process.communicate(
    timeout=30
)
```

This helps avoid common pipe-buffer deadlocks when both streams are involved.

---

# 97. Handling Popen Timeout

```python
try:
    stdout, stderr = process.communicate(
        timeout=30
    )
except subprocess.TimeoutExpired:
    process.kill()
    stdout, stderr = process.communicate()
    raise
```

---

# 98. Environment Variables

Pass controlled environment:

```python
import os
import subprocess

env = os.environ.copy()
env["APP_ENV"] = "staging"

subprocess.run(
    ["./deploy.sh"],
    env=env,
    check=True,
)
```

---

# 99. Environment Sanitization

Do not blindly pass every environment variable to untrusted subprocesses.

Sensitive variables may include:

```text
tokens
credentials
proxy credentials
cloud credentials
```

---

# 100. Working Directory

```python
subprocess.run(
    ["git", "status"],
    cwd="/opt/repository",
    check=True,
)
```

This is safer than assuming the current working directory.

---

# 101. File Descriptor Handling

Long-running automation should avoid leaking:

```text
stdin
stdout
stderr
sockets
files
```

Use context managers and proper process cleanup.

---

# 102. Command Output Parsing

Avoid:

```python
output.split()[3]
```

when a structured format exists.

Prefer:

```text
JSON
CSV
API
exit code
```

---

# 103. JSON CLI Output

Example:

```python
result = subprocess.run(
    [
        "kubectl",
        "get",
        "nodes",
        "-o",
        "json",
    ],
    capture_output=True,
    text=True,
    check=True,
)
```

Then:

```python
import json

data = json.loads(
    result.stdout
)
```

---

# 104. CSV Output

Some tools support:

```text
--csv
```

Parse using Python's `csv` module rather than manual string splitting.

---

# 105. Exit Codes as API

Treat CLI exit codes as part of the command contract.

Example:

```text
0 = healthy
1 = unhealthy
2 = usage/configuration
```

Always consult the command's documentation.

---

# 106. Signals

Linux processes receive signals such as:

```text
SIGTERM
SIGINT
SIGKILL
SIGHUP
```

Automation should understand whether the target process handles graceful shutdown.

---

# 107. Graceful Shutdown

For a long-running subprocess:

```text
SIGTERM
 ↓
wait
 ↓
SIGKILL if necessary
```

This gives the process an opportunity to clean up.

---

# 108. Signal Handling in Python

```python
import signal

def handle_signal(
    signum,
    frame,
):
    print(
        f"Received signal {signum}"
    )

signal.signal(
    signal.SIGTERM,
    handle_signal,
)
```

Use appropriate cleanup logic for services/workers.

---

# 109. Parent/Child Processes

If Python launches children, understand:

```text
who owns the process
who terminates it
what happens on parent failure
```

---

# 110. Process Groups

For complex subprocess trees, process groups can help terminate the entire child process tree safely.

This requires platform-aware handling.

---

# 111. Command Logging

Log:

```text
command category
target
start
duration
result
```

Avoid raw command strings when they may contain secrets.

---

# 112. Sensitive Argument Redaction

Example:

```python
def redact_args(
    args,
):
    sensitive = {
        "--password",
        "--token",
    }

    result = []
    hide_next = False

    for arg in args:
        if hide_next:
            result.append("[REDACTED]")
            hide_next = False
        elif arg in sensitive:
            result.append(arg)
            hide_next = True
        else:
            result.append(arg)

    return result
```

For production, use a robust structured redaction strategy.

---

# 113. Command Duration

Measure:

```python
start = time.monotonic()

result = subprocess.run(...)

duration = (
    time.monotonic() - start
)
```

Long durations can identify infrastructure bottlenecks.

---

# 114. Command Metrics

Track:

```text
command_success_total
command_failure_total
command_timeout_total
command_duration_seconds
```

Group by safe command category rather than exposing secret-bearing arguments.

---

# 115. Retry Wrapper

Concept:

```python
def run_with_retry(
    command,
    attempts=3,
):
    ...
```

Classify failures before retrying.

---

# 116. Retryable Return Codes

Some tools define transient codes.

Example:

```text
network unavailable
temporary lock
throttling
```

Do not assume every non-zero return code is retryable.

---

# 117. Backoff Example

```python
import random
import time

delay = 1

for attempt in range(3):
    try:
        run_task()
        break
    except TemporaryError:
        time.sleep(
            delay + random.random()
        )
        delay *= 2
else:
    raise RuntimeError(
        "All attempts failed"
    )
```

Use a maximum delay in real implementations.

---

# 118. Retry and Idempotency

Before retrying:

```text
Did the command execute?
Did it partially succeed?
Can executing again duplicate a side effect?
```

This is essential for production automation.

---

# 119. Command Atomicity

Commands may not be atomic.

Example:

```text
copy file
 ↓
partial transfer
 ↓
network failure
```

Use temporary files and verification for important transfers.

---

# 120. Temporary Command Output

For large output:

```python
from tempfile import NamedTemporaryFile

with NamedTemporaryFile(
    mode="w+",
    encoding="utf-8",
) as output:
    ...
```

Protect sensitive output appropriately.

---

# 121. Command Redirection

Instead of:

```text
shell=True
```

use Python file handles:

```python
with open(
    "output.txt",
    "w",
    encoding="utf-8",
) as file:

    subprocess.run(
        ["some-command"],
        stdout=file,
        check=True,
    )
```

---

# 122. stderr Redirection

```python
with open(
    "error.log",
    "w",
    encoding="utf-8",
) as file:

    subprocess.run(
        command,
        stderr=file,
        check=True,
    )
```

---

# 123. Combine stderr

```python
subprocess.run(
    command,
    stdout=subprocess.PIPE,
    stderr=subprocess.STDOUT,
    text=True,
)
```

Useful when a unified command log is desired.

---

# 124. stdin

Provide controlled input:

```python
subprocess.run(
    ["command"],
    input="approved\n",
    text=True,
    check=True,
)
```

Avoid using this as a way to inject passwords into commands when secure non-interactive authentication is available.

---

# 125. Interactive Commands

Avoid interactive commands in CI/CD.

Prefer:

```text
non-interactive flags
configuration
environment
API
```

---

# 126. `--yes` Flags

Some package managers support:

```text
-y
--yes
```

Use only when the operation is already approved and the target scope is correct.

---

# 127. Package Management

Python can automate:

```text
dnf
apt
yum
```

but configuration-management tools may be more appropriate for fleet-wide package state.

---

# 128. Package Installation

Example:

```python
subprocess.run(
    [
        "dnf",
        "install",
        "-y",
        "nginx",
    ],
    check=True,
)
```

Only do this when package installation is an approved part of the automation.

---

# 129. Package Verification

After installation:

```text
package installed
 ↓
version expected
 ↓
service configured
 ↓
service active
```

---

# 130. Service Dependency

A package may install successfully while the application still fails because of:

```text
configuration
permissions
ports
dependencies
```

Verify the service, not just the package.

---

# 131. Command Preconditions

Before:

```text
systemctl restart nginx
```

check:

```text
nginx installed
configuration valid
required files exist
```

---

# 132. Postconditions

After:

```text
restart
```

check:

```text
active
port listening
health endpoint
```

---

# 133. Preconditions and Postconditions

This pattern:

```text
PRE
 ↓
ACTION
 ↓
POST
```

makes command automation more reliable.

---

# 134. Command Contract

For every important command define:

```text
Input
Expected output
Expected return code
Timeout
Retry policy
Side effects
Verification
```

---

# 135. Example Contract

```text
Command:
systemctl reload nginx

Pre:
nginx -t passes

Expected:
return code 0

Post:
service active
HTTPS health passes

Timeout:
30s

Retry:
not automatic
```

---

# 136. Dry Run for Commands

Example:

```python
def execute(
    command,
    dry_run=False,
):
    if dry_run:
        print(
            "WOULD RUN:",
            command[0],
        )
        return

    subprocess.run(
        command,
        check=True,
    )
```

Production implementations should produce safe, useful plan output.

---

# 137. Command Categories

Categorize commands:

```text
READ
CHANGE
DESTRUCTIVE
```

Example:

```text
df -h = READ
systemctl reload = CHANGE
rm = DESTRUCTIVE
```

---

# 138. Require Stronger Guardrails

Destructive commands should require:

```text
explicit scope
approval
dry run
backup
confirmation
```

as appropriate.

---

# 139. Command Policy

A policy can define:

```text
allowed command
allowed arguments
allowed environment
required approval
```

---

# 140. Example Command Allowlist

```python
ALLOWED = {
    "systemctl": {
        "start",
        "stop",
        "restart",
        "reload",
    },
}
```

For complex authorization, use a dedicated policy mechanism rather than ad-hoc checks.

---

# 141. Command Runner Class

A reusable design:

```python
class CommandRunner:

    def run(
        self,
        command,
        timeout=30,
    ):
        ...
```

This centralizes execution behavior.

---

# 142. Command Result Class

Return:

```text
command
returncode
stdout
stderr
duration
timed_out
```

A dataclass is useful:

```python
from dataclasses import dataclass

@dataclass
class CommandResult:
    returncode: int
    stdout: str
    stderr: str
    duration: float
```

---

# 143. Why Structured Results?

Instead of returning only:

```text
stdout
```

the caller gets enough information to decide:

```text
success
failure
retry
report
```

---

# 144. Command Runner Example

```python
from dataclasses import dataclass
import subprocess
import time

@dataclass
class CommandResult:
    returncode: int
    stdout: str
    stderr: str
    duration: float

def run_command(
    command,
    timeout=30,
):
    start = time.monotonic()

    result = subprocess.run(
        command,
        capture_output=True,
        text=True,
        timeout=timeout,
        check=False,
    )

    return CommandResult(
        returncode=result.returncode,
        stdout=result.stdout,
        stderr=result.stderr,
        duration=(
            time.monotonic() - start
        ),
    )
```

---

# 145. Result Evaluation

```python
result = run_command(
    ["systemctl", "is-active", "nginx"]
)

if result.returncode == 0:
    print("PASS")
else:
    print(
        result.stderr
        or result.stdout
    )
```

---

# 146. Exception Boundary

A CLI application should have one clear top-level exception boundary:

```python
def main():
    ...

if __name__ == "__main__":
    try:
        main()
    except Exception:
        logger.exception(
            "Automation failed"
        )
        raise SystemExit(1)
```

---

# 147. Avoid Nested Broad Exceptions

Do not write:

```python
try:
    everything()
except Exception:
    pass
```

This destroys diagnostic information.

---

# 148. Command Errors vs Application Errors

Separate:

```text
command failed
```

from:

```text
Python validation failed
```

This helps incident diagnosis.

---

# 149. Example Error Classification

```python
class CommandExecutionError(
    RuntimeError
):
    pass

class CommandTimeoutError(
    RuntimeError
):
    pass
```

---

# 150. Command Execution Error

```python
if result.returncode != 0:
    raise CommandExecutionError(
        "Command failed"
    )
```

Include safe context.

---

# 151. Standard Output vs Standard Error

Do not assume:

```text
stdout = success
stderr = failure
```

Many commands use stderr for:

```text
warnings
progress
diagnostics
```

Use exit status plus documented behavior.

---

# 152. Command Output Encoding

If a command emits non-UTF-8 output, choose an appropriate encoding or error strategy rather than crashing unnecessarily.

---

# 153. `errors="replace"`

For diagnostic logs:

```python
subprocess.run(
    command,
    capture_output=True,
    text=True,
    errors="replace",
)
```

This can preserve troubleshooting output when invalid byte sequences occur.

---

# 154. Large stdout

Avoid:

```python
capture_output=True
```

for commands that can produce unlimited output.

Stream or redirect to a bounded destination.

---

# 155. Logging Command Output

Do not blindly log:

```python
logger.info(result.stdout)
```

Output may contain:

```text
credentials
tokens
customer data
```

Redact or limit output.

---

# 156. Output Limits

A production runner can implement:

```text
max stdout bytes
max stderr bytes
```

and mark output as truncated.

---

# 157. Command Audit

Audit record:

```json
{
  "action": "restart-service",
  "target": "app01",
  "result": "success",
  "duration_seconds": 2.4
}
```

Do not store sensitive arguments.

---

# 158. Correlation ID

Pass:

```text
run_id
```

through:

```text
logs
results
notifications
CI artifacts
```

---

# 159. CI Exit Status

If the Python script detects failure:

```python
raise SystemExit(1)
```

or allow a top-level exception to produce a non-zero process exit.

---

# 160. CI Success

Return:

```text
0
```

only when the required workflow actually succeeded.

---

# 161. CI Partial Failure

Decide whether:

```text
one host failure
```

means:

```text
entire job failed
```

based on the automation contract.

---

# 162. Jenkins Integration

Example:

```groovy
stage('Validation') {
    steps {
        sh 'python3 validate.py'
    }
}
```

Python's exit code controls the Jenkins step result.

---

# 163. GitHub Actions Integration

```yaml
- name: Validate deployment
  run: |
    python3 validate.py \
      --environment staging
```

---

# 164. GitLab CI Integration

```yaml
validate:
  script:
    - python3 validate.py --environment staging
```

---

# 165. Command Automation in Pipelines

Typical:

```text
checkout
 ↓
validate
 ↓
build
 ↓
scan
 ↓
deploy
 ↓
command-based verification
```

---

# 166. Pipeline Safety

Never allow:

```text
arbitrary PR input
```

to become:

```text
shell command
```

without validation.

---

# 167. Pull Request Automation

Safe examples:

```text
run tests
validate Terraform
lint Helm
generate report
```

Avoid granting untrusted PR code production credentials.

---

# 168. CI Runner Permissions

Use:

```text
minimal cloud role
minimal repository permissions
isolated runner
temporary credentials
```

---

# 169. Command Environment

Automation may behave differently between:

```text
local shell
Jenkins
GitHub runner
GitLab runner
Docker
Kubernetes
```

Print safe environment metadata when debugging.

---

# 170. Debug Mode

Provide:

```bash
--debug
```

to increase diagnostic logging.

Do not make debug mode print secrets.

---

# 171. Verbose Mode

```bash
--verbose
```

can expose:

```text
command timings
targets
decision points
```

without exposing sensitive arguments.

---

# 172. Quiet Mode

```bash
--quiet
```

is useful for automation where only errors should appear.

---

# 173. CLI Design

A production command tool should support:

```text
--help
--version
--config
--environment
--dry-run
--verbose
--timeout
```

as appropriate.

---

# 174. Argparse

Example:

```python
import argparse

parser = argparse.ArgumentParser()

parser.add_argument(
    "--environment",
    required=True,
)

parser.add_argument(
    "--dry-run",
    action="store_true",
)

args = parser.parse_args()
```

---

# 175. Command Automation Architecture

Recommended:

```text
src/
├── cli.py
├── config.py
├── runner.py
├── validators.py
├── commands.py
├── reports.py
└── main.py
```

---

# 176. `commands.py`

Keep command construction separate:

```python
def restart_service(
    service,
):
    return [
        "systemctl",
        "restart",
        service,
    ]
```

---

# 177. Validation Layer

```python
def validate_service(
    service,
):
    if service not in ALLOWED:
        raise ValueError(
            "Invalid service"
        )
```

---

# 178. Execution Layer

```python
def execute(command):
    return subprocess.run(
        command,
        check=False,
        ...
    )
```

---

# 179. Reporting Layer

```python
def report(result):
    return {
        "status": ...,
        "duration": ...,
    }
```

---

# 180. Why Separate Layers?

It makes the code:

```text
testable
reusable
maintainable
auditable
```

---

# 181. Unit Test Command Construction

```python
def test_restart_command():
    assert restart_service(
        "nginx"
    ) == [
        "systemctl",
        "restart",
        "nginx",
    ]
```

No actual restart occurs.

---

# 182. Unit Test Validation

```python
def test_invalid_service():
    try:
        validate_service(
            "unknown"
        )
    except ValueError:
        assert True
```

Use pytest for cleaner assertions.

---

# 183. Mock Command Execution

Mock:

```python
subprocess.run
```

so tests don't modify the real host.

---

# 184. Integration Test

A controlled integration test may execute:

```text
real command
```

inside:

```text
test VM
container
sandbox
```

---

# 185. Command Automation Test Matrix

Test:

```text
success
non-zero
timeout
missing binary
permission denied
invalid input
unexpected output
large output
```

---

# 186. Missing Binary

Example:

```python
try:
    subprocess.run(
        ["does-not-exist"],
        check=True,
    )
except FileNotFoundError:
    print(
        "Required command not installed"
    )
```

---

# 187. Permission Denied

```text
PermissionError
```

may occur when Python cannot execute or access the target.

Handle it distinctly from:

```text
command returned non-zero
```

---

# 188. Permission Failure Example

```python
try:
    result = subprocess.run(
        ["/root/protected-script"],
        check=True,
    )
except PermissionError:
    raise RuntimeError(
        "Execution permission denied"
    )
```

---

# 189. Command Not Found vs Non-Zero

Important distinction:

```text
FileNotFoundError
```

means Python could not start the executable.

```text
CalledProcessError
```

means the executable started and returned a non-zero status.

---

# 190. Timeout vs Failure

These are different:

```text
timeout
```

means execution exceeded the configured time.

```text
non-zero
```

means the process completed but reported failure.

---

# 191. Retry Classification Table

| Condition | Usually Retry? |
|---|---|
| Command not found | No |
| Invalid argument | No |
| Permission denied | No |
| Configuration syntax error | No |
| Temporary network failure | Maybe |
| API throttling | Yes, with backoff |
| Service startup timeout | Maybe |
| Unknown failure | Investigate |

---

# 192. Command Safety Table

| Operation | Risk |
|---|---|
| `df -h` | Read |
| `systemctl status` | Read |
| `systemctl reload` | Change |
| `systemctl restart` | Change |
| `rm` | Destructive |
| `terraform apply` | High impact |
| `kubectl delete` | Destructive |
| `aws terminate-instances` | Highly destructive |

Use stronger controls as impact increases.

---

# 193. Destructive Command Guard

```python
if action == "delete":
    if not approved:
        raise RuntimeError(
            "Deletion requires approval"
        )
```

---

# 194. `rm` Automation

Avoid constructing:

```python
["rm", "-rf", user_path]
```

when Python filesystem APIs can provide safer controlled deletion.

---

# 195. Python File Deletion

For a known file:

```python
from pathlib import Path

Path(
    "/opt/app/tmp.txt"
).unlink(
    missing_ok=True
)
```

Validate the path first.

---

# 196. Directory Deletion

```python
import shutil
from pathlib import Path

path = Path(
    "/opt/app/cache"
)

if path.is_dir():
    shutil.rmtree(path)
```

Only do this after strict scope validation.

---

# 197. Why Prefer Python APIs?

They provide:

```text
less shell interpretation
structured errors
cross-platform behavior
clearer intent
```

---

# 198. Shell Commands Still Matter

Some tools are best accessed through their CLI:

```text
kubectl
helm
terraform
git
systemctl
journalctl
```

The goal is not to eliminate commands; it is to execute them safely.

---

# 199. Command Wrapper for DevOps Tools

```python
TOOLS = {
    "kubectl": shutil.which("kubectl"),
    "helm": shutil.which("helm"),
    "terraform": shutil.which("terraform"),
}
```

Validate required tools before running.

---

# 200. Tool Version Report

At startup:

```text
Python 3.x
kubectl x.y
helm x.y
terraform x.y
```

This is useful for troubleshooting.

---

# 201. Version Compatibility

A script should fail early when an unsupported tool version is detected.

Example:

```text
required kubectl >= 1.30
```

---

# 202. Semantic Version Parsing

Avoid comparing versions as strings:

```python
"1.10" < "1.9"
```

can produce incorrect lexical comparisons.

Use a proper version parser when needed.

---

# 203. Command Availability Report

```text
kubectl: PASS
helm: PASS
terraform: PASS
aws: FAIL
```

Fail before making partial changes if a required dependency is missing.

---

# 204. Preflight Function

```python
def preflight():
    check_python()
    check_commands()
    check_permissions()
    check_environment()
```

---

# 205. Permission Preflight

Verify:

```text
read config
write target
execute command
access network
```

before starting a multi-step workflow.

---

# 206. Network Preflight

Check required endpoints:

```text
DNS
TCP
HTTP
AWS API
Git
container registry
```

Do not use a single ping as proof that an application dependency is healthy.

---

# 207. Dependency Preflight

Example:

```text
Git repository reachable
 ↓
ECR reachable
 ↓
Kubernetes API reachable
```

Only then begin deployment.

---

# 208. Command Ordering

Bad:

```text
restart service
download config
validate
```

Better:

```text
download
validate
backup
apply
restart/reload
verify
```

---

# 209. Fail Before Change

Whenever possible, validate all known preconditions before changing production state.

This reduces partial execution.

---

# 210. Command Chaining

Avoid giant shell strings such as:

```text
cmd1 && cmd2 && cmd3
```

unless shell semantics are intentionally required.

Python can make each stage explicit.

---

# 211. Explicit Steps

```python
run(["step1"])
run(["step2"])
run(["step3"])
```

Now you can identify exactly which step failed.

---

# 212. Step Results

```text
step1 PASS
step2 PASS
step3 FAIL
```

This is much better than:

```text
script failed
```

---

# 213. Workflow Step Object

```python
steps = [
    ("validate", validate),
    ("backup", backup),
    ("apply", apply),
    ("verify", verify),
]
```

Then execute and report each step.

---

# 214. Step Timing

Record:

```text
validate = 0.4s
backup = 3.2s
apply = 1.7s
verify = 2.1s
```

This helps optimize slow workflows.

---

# 215. Step Dependencies

Some steps depend on previous success:

```text
backup
 ↓
apply
```

Others can run independently:

```text
CPU check
memory check
disk check
```

---

# 216. Parallel Checks

Independent read-only checks can run concurrently.

Example:

```python
from concurrent.futures import (
    ThreadPoolExecutor,
)

with ThreadPoolExecutor(
    max_workers=3
) as executor:

    results = list(
        executor.map(
            check,
            checks,
        )
    )
```

---

# 217. Do Not Parallelize Blindly

Avoid concurrent execution when:

```text
operations modify same resource
API has strict rate limit
ordering matters
database lock exists
```

---

# 218. Bounded Concurrency

Use:

```text
max_workers
```

appropriate for:

```text
CPU
network
API quota
target capacity
```

---

# 219. Command Queue

For large automation:

```text
commands
 ↓
queue
 ↓
workers
 ↓
results
```

This allows controlled execution.

---

# 220. Worker Failure

A worker should report:

```text
target
command category
result
error
duration
```

rather than silently disappearing.

---

# 221. Fleet Report

Example:

```text
Total: 100
PASS: 94
FAIL: 4
TIMEOUT: 2
```

---

# 222. Failure Report

```text
app07: timeout
app32: permission denied
app54: service returned 1
app81: dependency unavailable
```

This enables focused troubleshooting.

---

# 223. Retry Failed Targets

If safe:

```text
first pass
 ↓
collect failures
 ↓
retry transient failures
 ↓
final report
```

Do not retry permanent failures.

---

# 224. Retry Budget

Define:

```text
max attempts
max total time
```

for the entire operation.

---

# 225. Overall Workflow Timeout

Even if every individual command has a timeout:

```text
30s each × 100 hosts
```

the workflow could run too long.

Define an overall operational deadline where appropriate.

---

# 226. Deadline Thinking

```text
workflow deadline
 ↓
remaining time
 ↓
per-command timeout
```

This prevents late-stage work from running indefinitely.

---

# 227. Command Cancellation

If the overall workflow expires:

```text
stop scheduling new work
 ↓
cancel/wind down running work
 ↓
collect results
 ↓
report timeout
```

---

# 228. Safe Cancellation

Do not abruptly terminate critical operations without understanding their consequences.

---

# 229. Long-Running Commands

Examples:

```text
terraform apply
docker build
database backup
kubectl rollout
```

Use:

```text
longer explicit timeout
streaming logs
progress
```

rather than unlimited execution.

---

# 230. Progress Reporting

For long tasks:

```text
10%
25%
50%
75%
100%
```

or step-level progress.

Avoid producing excessive CI log noise.

---

# 231. Command Output Streaming

Useful for:

```text
terraform plan
docker build
kubectl logs
```

so operators can see progress.

---

# 232. Streaming and Secret Risk

Live output can contain:

```text
credentials
tokens
URLs with secrets
```

Use redaction before displaying output.

---

# 233. Command Environment Security

Avoid:

```python
env = os.environ.copy()
```

when passing environment to untrusted subprocesses if it exposes credentials unnecessarily.

Create a minimal environment where appropriate.

---

# 234. Minimal Environment

```python
env = {
    "PATH": "/usr/bin:/bin",
    "LANG": "C",
}
```

Then add only required variables.

---

# 235. PATH Security

A malicious executable earlier in PATH can be invoked unexpectedly.

For privileged automation, use:

```text
controlled PATH
absolute executable paths
```

when appropriate.

---

# 236. Privileged Automation

If Python runs as root:

```text
every subprocess is powerful
```

Therefore:

```text
validate
allowlist
absolute paths
minimal environment
minimal code
```

become even more important.

---

# 237. Root Automation Rule

Never allow:

```text
untrusted input
+
root shell
```

---

# 238. Sudo Command Allowlist

If using sudo:

```text
automation user
 ↓
specific sudo rule
 ↓
specific command
```

not:

```text
automation user
 ↓
ALL commands
```

---

# 239. Command Injection Through Environment

Even without shell input, environment variables can affect commands:

```text
PATH
LD_PRELOAD
config variables
```

Privileged processes should sanitize their environment.

---

# 240. Command Injection Through Working Directory

Some tools discover:

```text
configuration
plugins
executables
```

relative to the working directory.

Use a trusted `cwd`.

---

# 241. Command Injection Through Config

A command may read configuration from:

```text
~/.config
/etc/tool
environment
```

Use controlled configuration for automation.

---

# 242. `HOME`

Automation under CI may have:

```text
different HOME
```

which changes tool behavior.

Set it deliberately when necessary.

---

# 243. Locale

Output parsing can break because of locale.

For machine-oriented commands, use a stable locale when appropriate:

```text
LC_ALL=C
```

But prefer structured output over locale-dependent text parsing.

---

# 244. Time Zone

Automation should use:

```text
UTC
```

for logs and machine timestamps where practical.

---

# 245. Current Directory

Never assume:

```text
pwd
```

is the repository or application directory.

Set `cwd` explicitly.

---

# 246. File System Race Conditions

This is unsafe:

```python
if not path.exists():
    path.write_text(...)
```

because another process can change the path between the check and write.

Use atomic operations or exclusive creation where required.

---

# 247. Exclusive File Creation

For files that must not already exist:

```python
path.open(
    "x",
    encoding="utf-8",
)
```

This fails if the file already exists.

---

# 248. Command Race Conditions

Checking:

```text
service inactive
```

and then:

```text
restart service
```

may be affected by another process changing state.

For critical workflows, minimize time between decision and action and use the authoritative control mechanism.

---

# 249. TOCTOU

TOCTOU means:

```text
Time Of Check
vs
Time Of Use
```

The state may change between validation and execution.

This is important in security-sensitive automation.

---

# 250. Avoid Unsafe Temporary Names

Do not create predictable temporary filenames for sensitive operations.

Use:

```python
tempfile
```

instead.

---

# 251. Command Automation and Containers

Inside containers:

```text
systemctl
```

may not exist.

Use the container platform's process model instead.

---

# 252. Detect Container

Environment detection can help:

```text
/.dockerenv
/proc/1
```

but avoid fragile assumptions. Prefer explicit configuration.

---

# 253. Container PID 1

PID 1 has special signal and child-reaping behavior.

Automation inside containers should understand the container's process model.

---

# 254. Kubernetes Exec

Commands can be run inside pods with:

```text
kubectl exec
```

but this should be used for controlled diagnostics rather than becoming the primary application management mechanism.

---

# 255. Kubernetes Exec Safety

Specify:

```text
context
namespace
pod
container
```

and use read-only diagnostics whenever possible.

---

# 256. Kubernetes Rollout

Better than repeatedly polling arbitrary process state:

```bash
kubectl rollout status
```

because Kubernetes understands deployment rollout state.

---

# 257. Kubernetes Wait

Use:

```text
kubectl wait
```

when it matches the condition you need.

---

# 258. Helm Status

After Helm deployment:

```python
subprocess.run(
    [
        "helm",
        "status",
        "myapp",
        "-n",
        "production",
    ],
    check=True,
)
```

Then perform application-level health checks.

---

# 259. Terraform Output

Use:

```text
terraform output -json
```

rather than parsing normal human output.

---

# 260. Terraform State Safety

Do not manipulate:

```text
terraform.tfstate
```

directly with Python.

Use Terraform commands and APIs/workflows.

---

# 261. GitOps Safety

Do not use Python to make direct production Kubernetes changes if Git is the declared source of truth.

Instead:

```text
Python updates approved Git configuration
 ↓
PR/review
 ↓
Argo CD reconciles
```

when that matches the architecture.

---

# 262. Command Automation and GitOps

Python can automate:

```text
generate manifest
validate manifest
update image tag
create PR
```

while Git remains the source of truth.

---

# 263. Image Tag Automation

Example:

```text
build image
 ↓
scan
 ↓
digest
 ↓
update Git manifest
 ↓
Argo CD sync
 ↓
verify rollout
```

---

# 264. Git Command Safety

Never execute:

```text
git reset --hard
git clean -fd
```

automatically on production repositories without explicit scope and approval.

These can destroy local changes.

---

# 265. `git diff --exit-code`

Useful for determining whether a generated configuration changed:

```python
result = subprocess.run(
    [
        "git",
        "diff",
        "--exit-code",
    ],
    check=False,
)

if result.returncode == 0:
    print("No changes")
```

---

# 266. Generated Configuration

Python can generate:

```text
YAML
JSON
Helm values
Terraform variables
environment files
```

Then validate before committing/deploying.

---

# 267. Template Rendering

Keep:

```text
template
+
validated data
=
generated configuration
```

Avoid concatenating untrusted shell fragments into configuration.

---

# 268. Command Automation and Configuration

Do not use:

```text
sed -i random replacement
```

for complex configuration unless the exact change is controlled.

Prefer parsing or generating structured configuration where possible.

---

# 269. Shell Editing vs Python

For simple controlled replacement:

```text
sed
```

may be fine.

For structured JSON/YAML:

```text
Python parser
```

is usually safer.

---

# 270. YAML Automation

```python
import yaml

with open(
    "values.yaml",
    encoding="utf-8",
) as file:

    data = yaml.safe_load(file)

data["image"]["tag"] = "2.4.1"
```

Then write the validated result.

---

# 271. JSON Automation

```python
import json

with open(
    "config.json",
    encoding="utf-8",
) as file:

    data = json.load(file)

data["version"] = "2.4.1"
```

---

# 272. Command Validation

Before deployment:

```text
generated config
 ↓
JSON/YAML parse
 ↓
schema validation
 ↓
tool-specific validation
```

---

# 273. Kubernetes Manifest Validation

```text
YAML parse
 ↓
kubectl apply --dry-run=client
 ↓
policy checks
 ↓
deployment
```

Use server-side validation where appropriate to catch cluster/API-specific issues.

---

# 274. Helm Validation

```text
helm lint
 ↓
helm template
 ↓
policy/security validation
```

---

# 275. Terraform Validation

```text
terraform fmt
 ↓
terraform validate
 ↓
terraform plan
 ↓
policy
 ↓
approval
```

---

# 276. Ansible Validation

```text
ansible-lint
 ↓
syntax check
 ↓
check/dry run
 ↓
apply
```

---

# 277. Command Automation and DevSecOps

Python can enforce:

```text
validation
security gates
policy
approval
deployment verification
```

without replacing dedicated security tools.

---

# 278. Automation Audit Log

A good record:

```text
run_id
actor
environment
target
action
start
end
result
change_id
```

---

# 279. Actor Identity

Identify whether execution came from:

```text
developer
Jenkins
GitHub Actions
GitLab
scheduled job
AWS Lambda
Kubernetes CronJob
```

---

# 280. Automation Ownership

Every production command automation should have:

```text
repository
owner
documentation
support contact
```

---

# 281. Automation Retirement

Retire scripts when:

```text
tool replaced
system decommissioned
workflow obsolete
security risk
duplicate capability
```

Delete unused production automation rather than leaving dangerous stale scripts.

---

# 282. Command Automation Checklist

Before execution:

```text
[ ] Target correct?
[ ] Environment correct?
[ ] Account correct?
[ ] Permissions correct?
[ ] Dependency installed?
[ ] Input validated?
[ ] Command safe?
[ ] Timeout set?
[ ] Retry policy defined?
```

---

# 283. After Execution

```text
[ ] Exit code checked
[ ] Output inspected
[ ] Desired state verified
[ ] Logs recorded
[ ] Changes reported
[ ] Failure handled
```

---

# 284. Production Command Checklist

```text
Validate
 ↓
Backup if needed
 ↓
Dry run
 ↓
Approval
 ↓
Execute
 ↓
Verify
 ↓
Report
```

---

# 285. Real-World Example — Restart Nginx Safely

```python
import subprocess

def restart_nginx():
    subprocess.run(
        ["nginx", "-t"],
        check=True,
    )

    subprocess.run(
        [
            "systemctl",
            "reload",
            "nginx",
        ],
        check=True,
    )

    result = subprocess.run(
        [
            "systemctl",
            "is-active",
            "nginx",
        ],
        capture_output=True,
        text=True,
        check=False,
    )

    if result.stdout.strip() != "active":
        raise RuntimeError(
            "nginx verification failed"
        )
```

---

# 286. Real-World Example — Kubernetes Rollout

```python
import subprocess

def verify_rollout():
    subprocess.run(
        [
            "kubectl",
            "rollout",
            "status",
            "deployment/myapp",
            "-n",
            "production",
            "--timeout=120s",
        ],
        check=True,
    )
```

Then add:

```text
HTTP health
metrics
logs
business validation
```

as required.

---

# 287. Real-World Example — Terraform Validation

```python
import subprocess

def validate_terraform():
    subprocess.run(
        [
            "terraform",
            "fmt",
            "-check",
        ],
        check=True,
    )

    subprocess.run(
        [
            "terraform",
            "validate",
        ],
        check=True,
    )
```

---

# 288. Real-World Example — Docker Image Validation

```python
import subprocess

def inspect_image(image):
    return subprocess.run(
        [
            "docker",
            "image",
            "inspect",
            image,
        ],
        capture_output=True,
        text=True,
        check=False,
    )
```

---

# 289. Real-World Example — Git Commit Metadata

```python
import subprocess

def git_sha():
    result = subprocess.run(
        [
            "git",
            "rev-parse",
            "HEAD",
        ],
        capture_output=True,
        text=True,
        check=True,
    )

    return result.stdout.strip()
```

---

# 290. Real-World Example — Disk Gate

```python
import shutil

def check_disk(
    path="/",
    threshold=85,
):
    usage = shutil.disk_usage(path)

    percent = (
        usage.used
        / usage.total
        * 100
    )

    if percent >= threshold:
        raise RuntimeError(
            f"Disk usage is {percent:.1f}%"
        )
```

---

# 291. Real-World Example — HTTP Gate

```python
import urllib.request

def check_http(url):
    with urllib.request.urlopen(
        url,
        timeout=5,
    ) as response:

        if response.status >= 400:
            raise RuntimeError(
                f"HTTP failure: "
                f"{response.status}"
            )
```

---

# 292. Real-World Example — Command Health

```python
def command_ok(command):
    result = subprocess.run(
        command,
        stdout=subprocess.DEVNULL,
        stderr=subprocess.DEVNULL,
        check=False,
    )

    return result.returncode == 0
```

Useful for simple checks, but keep diagnostics available when failures matter.

---

# 293. Real-World Example — Multiple Checks

```python
checks = {
    "disk": lambda:
        check_disk("/"),
    "nginx": lambda:
        command_ok(
            [
                "systemctl",
                "is-active",
                "nginx",
            ]
        ),
}
```

---

# 294. Check Report

```python
results = {}

for name, check in checks.items():
    try:
        value = check()
        results[name] = (
            "PASS"
            if value is not False
            else "FAIL"
        )
    except Exception:
        results[name] = "FAIL"
```

For production, capture structured error details instead of only `"FAIL"`.

---

# 295. Automation Report Example

```text
SYSTEM HEALTH

Disk       PASS
Nginx      PASS
Port 443   PASS
HTTP       PASS

Overall    PASS
```

---

# 296. Machine Report

```json
{
  "overall": "PASS",
  "checks": {
    "disk": "PASS",
    "nginx": "PASS",
    "port_443": "PASS",
    "http": "PASS"
  }
}
```

---

# 297. Human + Machine Output

A strong tool can provide:

```text
human-readable CLI
+
JSON output
```

This makes it useful for both engineers and CI systems.

---

# 298. JSON CLI Flag

```bash
python health.py --json
```

Output:

```json
{
  "status": "PASS"
}
```

---

# 299. Exit Code + JSON

CI should consume:

```text
exit code
```

while dashboards and other automation can consume:

```text
JSON
```

---

# 300. Command Automation Interview Questions

## Q1. What is the preferred Python module for executing shell commands?

**Answer:**

> `subprocess` is the preferred modern standard-library module because it provides control over arguments, stdout, stderr, return codes, environment, working directory, timeouts, and process lifecycle.

---

## Q2. What is the difference between `subprocess.run()` and `Popen()`?

**Answer:**

> `subprocess.run()` is a convenient high-level interface for running a command and waiting for it to finish. `Popen()` provides lower-level process control and is useful for streaming output, long-running processes, pipelines, and more advanced process management.

---

## Q3. Why is `shell=True` risky?

**Answer:**

> It passes the command through a shell, so untrusted input can become shell syntax and potentially lead to command injection. I prefer structured argument lists and `shell=False` unless shell behavior is explicitly required.

---

## Q4. How do you handle command timeouts?

**Answer:**

> I set an explicit timeout on the subprocess, catch `TimeoutExpired`, collect safe diagnostics, clean up the process if necessary, and classify whether the operation should be retried.

---

## Q5. How do you parse command output reliably?

**Answer:**

> I prefer structured output such as JSON or CSV, official APIs, or exit codes. I avoid parsing human-readable output with fragile string positions when a machine-readable interface is available.

---

## Q6. What is the difference between stdout and stderr?

**Answer:**

> They are separate output streams. stdout normally contains primary output while stderr is conventionally used for diagnostics, but commands can use them differently. I use the documented behavior and the process exit code rather than assuming stderr always means failure.

---

## Q7. How do you run a command with environment variables?

**Answer:**

> I create a controlled environment dictionary and pass it using the `env` parameter. I avoid unnecessarily passing secrets or the entire parent environment to privileged or untrusted processes.

---

## Q8. How do you execute a command in a specific directory?

**Answer:**

> I use the `cwd` parameter of `subprocess.run()` or `Popen()` rather than relying on the process's current working directory.

---

## Q9. How do you execute commands on remote Linux servers?

**Answer:**

> For simple controlled operations I can use SSH through subprocess. For richer automation I prefer tools such as Ansible, Paramiko/AsyncSSH, or AWS Systems Manager depending on the environment and security model.

---

## Q10. How do you safely automate Kubernetes commands?

**Answer:**

> I explicitly specify context and namespace, validate the target, prefer structured JSON output, use bounded timeouts, and verify the desired state after changes. In a GitOps environment I avoid bypassing the Git source of truth with direct production mutations.

---

## Q11. How would you automate Terraform from Python?

**Answer:**

> I would use Python for preflight validation, orchestration, policy checks, reporting, and post-apply verification while keeping Terraform as the infrastructure source of truth. I would not manipulate Terraform state directly.

---

## Q12. How do you prevent a Python command automation script from executing on the wrong AWS account?

**Answer:**

> I validate the caller identity before any mutation, compare the account to an approved target, explicitly control the region, use least-privilege IAM roles, and require production approval where appropriate.

---

## Q13. How do you automate commands across hundreds of servers?

**Answer:**

> I use inventory, bounded concurrency, batches, timeouts, retry classification, per-host results, checkpointing, and progressive rollout. I start with a canary or small batch to reduce blast radius.

---

## Q14. What happens if a command is not installed?

**Answer:**

> Python may raise `FileNotFoundError` when the executable cannot be started. I detect required dependencies during preflight and fail before making partial production changes.

---

## Q15. What is the difference between `FileNotFoundError` and `CalledProcessError`?

**Answer:**

> `FileNotFoundError` means Python could not start the executable. `CalledProcessError` is raised by `check=True` when the executable started but returned a non-zero exit status.

---

## Q16. How do you prevent infinite retries?

**Answer:**

> I define a maximum attempt count, maximum delay, and preferably an overall workflow deadline. I retry only classified transient failures and use exponential backoff with jitter when appropriate.

---

## Q17. How do you handle huge command output?

**Answer:**

> I avoid unbounded `capture_output=True` for commands that may produce very large output. I stream output or redirect it to a controlled file and enforce output limits where necessary.

---

## Q18. How do you safely restart a service?

**Answer:**

> I validate the service, validate configuration if applicable, perform the approved action, verify the service state, verify the listening port or health endpoint, and report the result. For production systems I also define rollback or recovery behavior.

---

## Q19. How do you automate a Linux command without using a shell?

**Answer:**

```python
subprocess.run(
    ["systemctl", "is-active", "nginx"],
    check=False,
)
```

> The arguments are passed directly to the executable.

---

## Q20. How do you automate a command pipeline?

**Answer:**

> I first ask whether Python or a structured API can replace the pipeline. If a pipeline is necessary, I can use `Popen()` with connected stdin/stdout streams. I avoid passing untrusted strings to a shell.

---

# 301. Production Scenario — Wrong Kubernetes Context

**Question:**

Your Python script is supposed to restart a staging deployment but accidentally uses the production context. How do you prevent this?

**Answer:**

> I would not rely on the current kubectl context. The automation should explicitly select or validate the expected context, explicitly specify the namespace, and compare the resolved cluster identity with the expected environment before allowing mutations. Production should have additional approval and access controls.

---

# 302. Production Scenario — Terraform Command Hangs

**Answer:**

> I would inspect the running process and Terraform logs, determine whether it is waiting on a provider, lock, network call, or interactive input, and enforce an appropriate workflow timeout. I would not blindly kill a potentially active infrastructure operation without understanding its state. I would also ensure CI uses non-interactive execution and proper state locking.

---

# 303. Production Scenario — SSH Automation Fails Randomly

**Answer:**

> I would distinguish DNS, TCP connection, SSH authentication, host-key verification, command execution, and remote application failures. I would use explicit connection and command timeouts, controlled retries for transient network failures, and structured per-host results.

---

# 304. Production Scenario — Command Works Manually but Fails in Jenkins

**Answer:**

> I would compare the execution environments: user, PATH, HOME, working directory, credentials, environment variables, installed tool versions, network access, and permissions. CI should use explicit paths/configuration rather than assumptions from an interactive shell.

---

# 305. Production Scenario — `kubectl` Works on Laptop but Not CI

**Answer:**

> I would check the CI service account or cloud identity, kubeconfig/context, cluster endpoint access, network restrictions, RBAC, namespace, and kubectl version. I would avoid copying a developer's personal kubeconfig into CI and instead use the approved workload identity.

---

# 306. Production Scenario — `systemctl restart` Returns Success but Service Is Down

**Answer:**

> I would treat the command exit code as only one signal. I would check `systemctl is-active`, recent journal logs, port listeners, application health, and dependencies. The automation should not report success until the desired application state is verified.

---

# 307. Production Scenario — Automation Runs Twice

**Answer:**

> I would determine why concurrency occurred, add scheduler or application-level locking, make the operation idempotent, and ensure overlapping executions cannot cause duplicate side effects.

---

# 308. Production Scenario — API Throttling

**Answer:**

> I would identify the rate limit, reduce concurrency, respect `Retry-After` if supplied, use exponential backoff with jitter, and avoid retrying non-retryable failures.

---

# 309. Production Scenario — Deployment Command Partially Completed

**Answer:**

> I would preserve the step and target state, stop further changes if the failure is critical, collect diagnostics, determine whether the completed operation is safe to repeat, and either resume or roll back according to the deployment strategy.

---

# 310. Production Scenario — Need to Run a Dangerous Command

**Question:**

The team asks for automated `kubectl delete` in production.

**Answer:**

> I would first determine whether deletion is actually required and whether a safer Kubernetes-native operation exists. If deletion is approved, I would constrain the namespace/resource scope, validate the cluster identity, use least-privilege RBAC, provide dry-run where supported, require approval, and log the operation. I would never accept arbitrary resource names from untrusted input.

---

# 311. Production Scenario — Command Contains a Password

**Answer:**

> I would redesign the authentication flow so the password is not passed through the command line. I would use a secure credential mechanism, environment/secret store only when appropriate, or a native authentication method. I would also ensure the value is excluded from logs and artifacts.

---

# 312. Production Scenario — Need Command Output for Troubleshooting

**Answer:**

> I would capture stdout and stderr, but sanitize and limit them. I would preserve enough information for diagnosis without exposing credentials, customer data, or sensitive configuration.

---

# 313. Production Scenario — 10,000 Commands

**Answer:**

> I would not create 10,000 unrestricted subprocesses. I would classify the work, batch it, use bounded concurrency, respect target/API capacity, implement timeouts and retries, checkpoint progress, and produce aggregate and per-target results.

---

# 314. Production Scenario — Need to Run Shell Syntax

**Question:**

A script requires shell expansion and redirection. What do you do?

**Answer:**

> First I would determine whether Python APIs can replace the shell behavior. If shell execution is genuinely required, I would isolate the shell command, ensure all inputs are trusted or safely validated, avoid interpolating untrusted data, use a controlled environment, set a timeout, and log the operation safely.

---

# 315. Production Scenario — Security Review

**Question:**

What would a security reviewer look for in command automation?

**Answer:**

```text
command injection
secret handling
least privilege
path traversal
unsafe shell execution
environment manipulation
dependency vulnerabilities
logging of sensitive data
production guardrails
authorization
audit trail
```

---

# 316. End-to-End Command Automation Example

A production deployment validation tool could follow:

```text
CLI input
   ↓
Validate environment
   ↓
Validate dependencies
   ↓
Validate AWS account
   ↓
Validate Kubernetes context
   ↓
Validate namespace
   ↓
Run prechecks
   ↓
Execute approved command
   ↓
Capture result
   ↓
Verify desired state
   ↓
Generate JSON report
   ↓
Publish CI artifact
   ↓
Notify result
```

---

# 317. Example Project Structure

```text
command-automation/
├── src/
│   ├── cli.py
│   ├── runner.py
│   ├── validators.py
│   ├── commands.py
│   ├── kubernetes.py
│   ├── aws.py
│   ├── terraform.py
│   └── reports.py
├── tests/
│   ├── test_runner.py
│   ├── test_validators.py
│   └── test_commands.py
├── config/
│   └── environments.yaml
├── requirements.txt
├── README.md
└── Dockerfile
```

---

# 318. Example Production Flow

```text
Developer
   ↓
Git
   ↓
Jenkins/GitHub Actions/GitLab
   ↓
Python command automation
   ↓
Preflight
   ↓
Terraform / Ansible / Helm / kubectl
   ↓
Verification
   ↓
Prometheus/Grafana/ELK
   ↓
Notification
```

Python acts as the automation/orchestration layer rather than replacing every DevOps tool.

---

# 319. Key Commands a DevOps Engineer Should Be Comfortable Automating

```text
Linux:
systemctl
journalctl
df
du
find
ps
ss
ip
curl

Git:
git status
git diff
git rev-parse
git fetch

Docker:
docker build
docker inspect
docker ps

Kubernetes:
kubectl get
kubectl describe
kubectl logs
kubectl rollout
kubectl wait

Helm:
helm lint
helm template
helm status

Terraform:
terraform fmt
terraform validate
terraform plan

Ansible:
ansible-playbook
ansible-lint

AWS:
aws sts
aws ec2
aws s3
aws eks
aws ecr
aws ssm
```

---

# 320. Final Principles

Remember:

```text
1. Prefer subprocess over os.system.
2. Prefer argument lists over shell strings.
3. Avoid shell=True unless necessary.
4. Validate every external input.
5. Use allowlists for sensitive actions.
6. Set timeouts.
7. Classify retryable failures.
8. Never blindly retry side effects.
9. Prefer structured output.
10. Check exit codes.
11. Verify desired state.
12. Control working directory.
13. Control environment variables.
14. Protect secrets.
15. Limit output size.
16. Use bounded concurrency.
17. Control blast radius.
18. Use dry runs where appropriate.
19. Keep production approval outside the script when possible.
20. Log safely.
21. Return meaningful exit codes.
22. Test commands without executing production changes.
23. Keep Terraform/Ansible/GitOps as the source of truth where applicable.
24. Prefer native Python APIs when they provide safer structured access.
25. Treat automation as production software.
```

---

# 321. Final Interview Summary

When explaining command automation in a DevOps interview, focus on this:

> **I use Python's `subprocess` module for controlled command execution when a CLI is the appropriate interface. I avoid unsafe shell interpolation, validate inputs, use explicit timeouts, capture and interpret return codes, handle transient failures with bounded retries, and verify the resulting desired state. For Kubernetes, AWS, Terraform, and Ansible, I use Python as an orchestration or validation layer while keeping the platform-specific tool as the source of truth.**

That demonstrates not only Python knowledge, but **production DevOps judgment**.
