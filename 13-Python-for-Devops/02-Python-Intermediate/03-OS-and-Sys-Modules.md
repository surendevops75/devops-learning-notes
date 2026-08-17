# 03-OS-and-Sys-Modules

## 1. Overview

The `os` and `sys` modules are among the most important Python standard-library modules for DevOps automation.

They allow Python scripts to interact with the operating system and the Python runtime.

Typical DevOps use cases include:

- reading environment variables
- checking files and directories
- discovering the current user
- inspecting processes
- checking system information
- executing operating-system commands
- reading command-line arguments
- controlling exit codes
- detecting the Python runtime
- interacting with Linux environments
- building CI/CD helper scripts
- automating operational checks

A common architecture is:

```text
Python script
     |
     +--> os
     |     |
     |     +--> filesystem
     |     +--> environment
     |     +--> process/runtime
     |
     +--> sys
           |
           +--> arguments
           +--> exit codes
           +--> interpreter information
```

For DevOps, Python does not replace Linux knowledge.

Instead:

```text
Linux knowledge
       +
Python automation
       =
reliable DevOps tooling
```

---

# 2. Importing `os`

```python
import os
```

The `os` module provides interfaces to operating-system functionality.

Examples:

```python
os.getcwd()
os.listdir()
os.getenv()
os.environ
os.path
os.stat()
```

---

# 3. Importing `sys`

```python
import sys
```

The `sys` module provides access to Python runtime information and controls.

Common examples:

```python
sys.argv
sys.exit()
sys.version
sys.executable
sys.path
sys.platform
```

---

# 4. `os.getcwd()`

Get the current working directory:

```python
import os

print(os.getcwd())
```

Example:

```text
/home/devops/project
```

This is important in CI/CD because the current working directory may not be what you expect.

---

# 5. Current Working Directory vs Script Directory

These are not necessarily the same.

Suppose:

```text
/opt/automation/
├── scripts/
│   └── check.py
└── config/
    └── app.yaml
```

If the script is executed from:

```bash
cd /opt
python /opt/automation/scripts/check.py
```

then:

```python
os.getcwd()
```

returns:

```text
/opt
```

not:

```text
/opt/automation/scripts
```

This distinction causes many production automation bugs.

---

# 6. Script Location

Using `pathlib`:

```python
from pathlib import Path

script_dir = Path(__file__).resolve().parent

print(script_dir)
```

This identifies the directory containing the script.

For reusable automation, explicitly decide whether paths should be based on:

```text
current working directory
```

or:

```text
project/script directory
```

---

# 7. Change Working Directory

Python can change the process working directory:

```python
import os

os.chdir("/opt/app")

print(os.getcwd())
```

Use this carefully.

Changing the working directory globally can make later operations harder to reason about.

Prefer passing explicit paths when possible.

---

# 8. List Directory Contents

```python
import os

items = os.listdir("/var/log")

for item in items:
    print(item)
```

This returns names, not full paths.

For modern Python code, `pathlib.Path.iterdir()` is often clearer.

---

# 9. List With `pathlib`

```python
from pathlib import Path

for item in Path("/var/log").iterdir():
    print(item)
```

This returns `Path` objects.

---

# 10. Check Whether a Path Exists

```python
import os

if os.path.exists("/etc/hosts"):
    print("Exists")
```

Or:

```python
from pathlib import Path

if Path("/etc/hosts").exists():
    print("Exists")
```

---

# 11. File vs Directory

Using `os.path`:

```python
if os.path.isfile("/etc/hosts"):
    print("File")

if os.path.isdir("/etc"):
    print("Directory")
```

---

# 12. Build Paths With `os.path.join`

```python
import os

path = os.path.join(
    "/opt/app",
    "logs",
    "application.log",
)

print(path)
```

For modern code, prefer:

```python
from pathlib import Path

path = Path("/opt/app") / "logs" / "application.log"
```

---

# 13. Absolute Path

```python
import os

path = os.path.abspath("config.yaml")

print(path)
```

This converts a relative path into an absolute path.

---

# 14. Normalize a Path

```python
import os

path = os.path.normpath(
    "/opt/app/../app/config.yaml"
)

print(path)
```

This normalizes path syntax.

Do not confuse normalization with security validation.

For security-sensitive paths, use `resolve()` and validate the resulting path against an approved base.

---

# 15. File Size

```python
import os

size = os.path.getsize("app.log")

print(size)
```

Returns bytes.

---

# 16. File Metadata

```python
import os

info = os.stat("app.log")

print(info.st_size)
print(info.st_mtime)
print(info.st_mode)
```

Metadata can be used for:

```text
size checks
age checks
permissions
timestamps
```

---

# 17. Environment Variables

Environment variables are extremely important in DevOps.

Examples:

```text
AWS_REGION
APP_ENV
DATABASE_HOST
PORT
CI
BUILD_NUMBER
KUBECONFIG
```

Read one:

```python
import os

environment = os.getenv("APP_ENV")

print(environment)
```

---

# 18. `os.getenv()` With Default

```python
import os

environment = os.getenv(
    "APP_ENV",
    "development",
)

print(environment)
```

If `APP_ENV` is missing:

```text
development
```

is returned.

---

# 19. Required Environment Variable

For required configuration:

```python
import os

token = os.getenv("API_TOKEN")

if not token:
    raise RuntimeError(
        "API_TOKEN is required"
    )
```

This is safer than silently continuing with missing credentials.

---

# 20. `os.environ`

You can access environment variables through:

```python
import os

print(os.environ["APP_ENV"])
```

If missing, this raises:

```text
KeyError
```

Use this when the variable is mandatory and failure is appropriate.

---

# 21. `getenv()` vs `environ[]`

```python
os.getenv("NAME")
```

returns:

```text
None
```

if missing.

```python
os.environ["NAME"]
```

raises:

```text
KeyError
```

Choose based on whether missing configuration is expected or an error.

---

# 22. Set an Environment Variable

Inside the current process:

```python
import os

os.environ["APP_ENV"] = "production"
```

This affects the current Python process and child processes started afterward.

It does not permanently modify the user's shell environment.

---

# 23. Environment Variables Are Process State

If:

```python
os.environ["ENV"] = "production"
```

then:

```text
Python process
      |
      +--> child subprocesses can inherit it
```

But another already-running process is not automatically changed.

---

# 24. Environment Variables in CI/CD

A pipeline might provide:

```text
AWS_REGION
ENVIRONMENT
IMAGE_TAG
BUILD_NUMBER
```

Python reads them:

```python
import os

image_tag = os.getenv("IMAGE_TAG")

if not image_tag:
    raise RuntimeError(
        "IMAGE_TAG is missing"
    )
```

This is much better than hard-coding pipeline-specific values.

---

# 25. Never Hard-Code Secrets

Bad:

```python
PASSWORD = "SuperSecret123"
```

Better:

```python
password = os.getenv("DB_PASSWORD")
```

Even better for production:

```text
approved secret manager
      |
      v
runtime credential
      |
      v
Python process
```

Environment variables are not a complete secret-management strategy by themselves.

---

# 26. AWS Credentials

Do not write:

```python
AWS_ACCESS_KEY = "..."
AWS_SECRET_KEY = "..."
```

Prefer AWS's standard credential mechanisms such as:

```text
IAM roles
workload identity
AWS credential chain
CI/CD identity mechanisms
```

For EC2/EKS workloads, use the platform's identity mechanism rather than embedding long-lived credentials.

---

# 27. EKS and Environment Variables

A Kubernetes workload may receive:

```text
APP_ENV
LOG_LEVEL
AWS_REGION
SERVICE_NAME
```

Python:

```python
import os

service = os.getenv(
    "SERVICE_NAME",
    "unknown",
)

environment = os.getenv(
    "APP_ENV",
    "development",
)
```

Secrets should normally come from Kubernetes Secrets or an integrated external secret-management system.

---

# 28. Inspect Environment Variables

```python
import os

for key, value in os.environ.items():
    print(key, value)
```

Do not do this in production logs because secrets may be exposed.

---

# 29. Safe Environment Debugging

Instead of printing everything:

```python
safe_keys = [
    "APP_ENV",
    "AWS_REGION",
    "BUILD_NUMBER",
]

for key in safe_keys:
    print(
        f"{key}={os.getenv(key)}"
    )
```

Never print:

```text
PASSWORD
TOKEN
SECRET
PRIVATE_KEY
```

---

# 30. Environment Validation

Create:

```python
import os


def require_env(name):
    value = os.getenv(name)

    if not value:
        raise RuntimeError(
            f"Required environment variable "
            f"{name} is missing"
        )

    return value
```

Use:

```python
environment = require_env("APP_ENV")
```

This reusable pattern is useful across DevOps scripts.

---

# 31. Validate Multiple Variables

```python
import os


def validate_environment(required):
    missing = [
        name
        for name in required
        if not os.getenv(name)
    ]

    if missing:
        raise RuntimeError(
            f"Missing environment variables: "
            f"{', '.join(missing)}"
        )
```

Use:

```python
validate_environment([
    "APP_ENV",
    "AWS_REGION",
])
```

---

# 32. Environment Variable Types

Environment variables are strings.

```python
port = os.getenv("PORT")
```

`port` is:

```text
"8080"
```

not:

```text
8080
```

Convert explicitly:

```python
port = int(os.getenv("PORT", "8080"))
```

---

# 33. Boolean Environment Variables

Do not use:

```python
bool(os.getenv("DEBUG"))
```

because:

```python
bool("false")
```

is:

```text
True
```

Instead:

```python
value = os.getenv(
    "DEBUG",
    "false",
).lower()

debug = value in {
    "true",
    "1",
    "yes",
    "on",
}
```

---

# 34. Environment Configuration Pattern

```text
Environment
    |
    +--> APP_ENV
    +--> AWS_REGION
    +--> LOG_LEVEL
    +--> IMAGE_TAG
    |
    v
Configuration validation
    |
    v
Application / script
```

Keep configuration parsing separate from business logic.

---

# 35. `sys.argv`

`sys.argv` contains command-line arguments.

Example:

```python
import sys

print(sys.argv)
```

Run:

```bash
python script.py hello world
```

Output resembles:

```text
[
    "script.py",
    "hello",
    "world"
]
```

---

# 36. First Command-Line Argument

```python
import sys

if len(sys.argv) < 2:
    raise SystemExit(
        "Usage: python script.py <name>"
    )

name = sys.argv[1]

print(name)
```

This works for very simple scripts.

For production CLI tools, use `argparse`, covered later in the Python Intermediate section.

---

# 37. Why `argparse` Is Better

Instead of manually parsing:

```python
sys.argv
```

use:

```python
argparse
```

for:

```text
--environment
--region
--namespace
--threshold
--dry-run
```

The dedicated file:

```text
08-Argparse.md
```

will cover this deeply.

---

# 38. `sys.exit()`

Exit a Python program:

```python
import sys

sys.exit(0)
```

Convention:

```text
0 -> success
non-zero -> failure
```

---

# 39. Exit Code 1

```python
import sys

sys.exit(1)
```

This indicates failure.

CI/CD systems commonly use exit codes to decide whether a job succeeds or fails.

---

# 40. DevOps Exit Code Flow

```text
Python script
      |
      v
operation
      |
  +---+---+
  |       |
success  failure
  |       |
  v       v
exit 0  exit 1+
  |       |
  v       v
CI pass  CI fail
```

This is one of the most important concepts in automation scripting.

---

# 41. `raise SystemExit`

Instead of:

```python
sys.exit(1)
```

you can use:

```python
raise SystemExit(1)
```

Both terminate the program with an exit status.

---

# 42. `print()` vs Exit Code

Bad automation:

```python
print("Deployment failed")
```

and then the script exits with:

```text
0
```

The CI pipeline may think the deployment succeeded.

Better:

```python
print("Deployment failed")
sys.exit(1)
```

---

# 43. Meaningful Exit Codes

You can define:

```text
0 -> success
1 -> general failure
2 -> invalid arguments
3 -> configuration failure
4 -> external dependency failure
```

The exact scheme should be documented and consistent.

Do not create arbitrary codes that nobody understands.

---

# 44. `sys.version`

```python
import sys

print(sys.version)
```

Useful for troubleshooting Python runtime differences.

---

# 45. `sys.version_info`

```python
import sys

print(sys.version_info)
```

You can check:

```python
if sys.version_info < (3, 11):
    raise RuntimeError(
        "Python 3.11+ is required"
    )
```

---

# 46. Why Python Version Matters

A script can work on:

```text
Python 3.12
```

and fail on:

```text
Python 3.9
```

because of:

```text
syntax
standard-library features
dependency compatibility
runtime behavior
```

Always define supported Python versions for production automation.

---

# 47. `sys.executable`

Find the exact Python interpreter:

```python
import sys

print(sys.executable)
```

Example:

```text
/opt/automation/.venv/bin/python
```

This is extremely useful when debugging:

```text
works locally
fails in Jenkins
```

---

# 48. CI/CD Python Troubleshooting

Run:

```python
import sys

print("Python:", sys.version)
print("Executable:", sys.executable)
```

This tells you exactly which interpreter the pipeline uses.

---

# 49. `sys.path`

Inspect module search locations:

```python
import sys

for path in sys.path:
    print(path)
```

Useful when debugging:

```text
ModuleNotFoundError
```

---

# 50. Do Not Modify `sys.path` Casually

Avoid:

```python
sys.path.append("../")
```

as a permanent solution.

It can hide:

```text
packaging problems
working-directory problems
deployment problems
```

Use a proper package/project structure.

---

# 51. `sys.platform`

```python
import sys

print(sys.platform)
```

Typical values include:

```text
linux
win32
darwin
```

Useful when automation has platform-specific behavior.

---

# 52. Platform-Specific Commands

Linux:

```text
systemctl
df
du
ip
ss
```

Windows:

```text
PowerShell
Get-Service
Get-Process
```

If your DevOps automation is Linux-specific, explicitly document that assumption.

---

# 53. `os.name`

```python
import os

print(os.name)
```

Common values:

```text
posix
nt
```

This is a broad OS family indicator.

For detailed platform information, use `platform`.

---

# 54. `platform` Module

Although this file focuses on `os` and `sys`, `platform` is useful:

```python
import platform

print(platform.system())
print(platform.release())
print(platform.machine())
```

Example:

```text
Linux
5.x
x86_64
```

---

# 55. CPU Count

```python
import os

print(os.cpu_count())
```

This returns the number of logical CPUs visible to the process/environment.

In containers, CPU quotas can mean the host CPU count is not the same as the effective CPU limit. For container-aware resource decisions, inspect the platform's configured limits as well.

---

# 56. Environment vs Host Resources

In Kubernetes:

```text
os.cpu_count()
```

may reflect CPUs visible to the runtime rather than the application's Kubernetes CPU limit.

Do not use it blindly to determine application resource limits.

---

# 57. Process ID

```python
import os

print(os.getpid())
```

Useful for:

```text
process identification
debugging
logging
signals
```

---

# 58. Parent Process ID

```python
print(os.getppid())
```

Useful when troubleshooting process hierarchies.

---

# 59. Current User ID

On Unix-like systems:

```python
import os

print(os.getuid())
```

Current group:

```python
print(os.getgid())
```

These are useful for Linux automation.

They are not available in the same form on all operating systems.

---

# 60. Effective User ID

On Unix-like systems:

```python
print(os.geteuid())
```

This can help determine the effective privilege level.

---

# 61. Check Root Privileges

On Linux:

```python
import os

if os.geteuid() == 0:
    print("Running as root")
else:
    print("Not running as root")
```

Do not automatically require root unless the operation truly needs it.

---

# 62. Principle of Least Privilege

Bad:

```text
run every automation as root
```

Better:

```text
run as non-root
      |
      +--> request only required permissions
```

For AWS:

```text
least-privilege IAM role
```

For Kubernetes:

```text
least-privilege RBAC
```

For Linux:

```text
appropriate user/group permissions
```

---

# 63. User and Group Troubleshooting

Useful Linux commands:

```bash
whoami
id
groups
```

Python can retrieve runtime identity, but Linux commands remain important for full diagnosis.

---

# 64. Process Environment

Every process has an environment.

Python accesses it through:

```python
os.environ
```

Child processes normally inherit the parent's environment unless explicitly changed.

---

# 65. Passing Environment to Subprocess

```python
import os
import subprocess

env = os.environ.copy()

env["APP_ENV"] = "staging"

subprocess.run(
    ["python", "deploy.py"],
    env=env,
    check=True,
)
```

Copying the environment preserves existing variables.

---

# 66. Avoid Replacing the Entire Environment Accidentally

Bad:

```python
subprocess.run(
    ["command"],
    env={"APP_ENV": "production"},
)
```

This may remove important inherited variables.

Prefer:

```python
env = os.environ.copy()
env["APP_ENV"] = "production"
```

then pass `env`.

---

# 67. Environment and AWS CLI

A Python wrapper may execute:

```bash
aws sts get-caller-identity
```

with:

```python
subprocess.run(
    ["aws", "sts", "get-caller-identity"],
    check=True,
)
```

The AWS CLI inherits the process environment and credential configuration.

However, if a Python SDK is sufficient, direct SDK use is often cleaner than parsing CLI output.

---

# 68. Environment and `kubectl`

Similarly:

```python
subprocess.run(
    ["kubectl", "get", "pods"],
    check=True,
)
```

may use:

```text
KUBECONFIG
```

from the environment.

For programmatic Kubernetes automation, the Kubernetes client library can provide structured API access.

---

# 69. `os.system()`

Python provides:

```python
os.system("ls -l")
```

It is simple but limited.

For serious DevOps automation, prefer:

```python
subprocess.run(...)
```

because it provides better control over:

```text
arguments
stdout
stderr
return code
timeouts
environment
```

---

# 70. Why Avoid `os.system()`

With:

```python
os.system(command)
```

you have less structured control.

With:

```python
subprocess.run(
    ["kubectl", "get", "pods"],
    capture_output=True,
    text=True,
    check=True,
)
```

you can inspect the result properly.

---

# 71. Shell Injection

Dangerous:

```python
command = f"rm -rf {user_input}"

os.system(command)
```

or:

```python
subprocess.run(
    command,
    shell=True,
)
```

with untrusted input.

This can enable command injection.

---

# 72. Safer Command Execution

Prefer:

```python
subprocess.run(
    ["rm", "-f", filename],
    check=True,
)
```

But still validate:

```text
filename
allowed directory
operation
```

For destructive operations, path validation is essential.

---

# 73. `subprocess.run()` Basic Pattern

```python
import subprocess

result = subprocess.run(
    ["uname", "-a"],
    capture_output=True,
    text=True,
    check=False,
)

print(result.returncode)
print(result.stdout)
print(result.stderr)
```

---

# 74. `check=True`

```python
subprocess.run(
    ["systemctl", "status", "nginx"],
    check=True,
)
```

If the command returns non-zero:

```text
CalledProcessError
```

is raised.

Use this when a non-zero status should be treated as an exception.

---

# 75. `check=False`

```python
result = subprocess.run(
    ["systemctl", "is-active", "nginx"],
    check=False,
)
```

Useful when a non-zero result is expected and should be evaluated as data.

Example:

```text
nginx active -> 0
nginx inactive -> non-zero
```

---

# 76. Capture Standard Output

```python
result = subprocess.run(
    ["df", "-h"],
    capture_output=True,
    text=True,
)

print(result.stdout)
```

---

# 77. Capture Standard Error

```python
print(result.stderr)
```

This is important when diagnosing failed commands.

---

# 78. Return Code

```python
if result.returncode == 0:
    print("Success")
else:
    print("Failure")
```

Never assume that no Python exception means the external command succeeded when `check=False`.

---

# 79. Command Timeout

```python
subprocess.run(
    ["curl", "https://example.com"],
    timeout=10,
    check=True,
)
```

Production automation should have appropriate timeouts.

An external system can otherwise cause a pipeline to hang indefinitely.

---

# 80. Timeout Handling

```python
import subprocess

try:
    subprocess.run(
        ["curl", "https://example.com"],
        timeout=10,
        check=True,
    )
except subprocess.TimeoutExpired:
    print("Command timed out")
```

---

# 81. Working Directory for Subprocess

```python
subprocess.run(
    ["terraform", "validate"],
    cwd="/workspace/infrastructure",
    check=True,
)
```

This is safer than changing the entire Python process's working directory.

---

# 82. Environment for Subprocess

```python
import os
import subprocess

env = os.environ.copy()
env["TF_IN_AUTOMATION"] = "true"

subprocess.run(
    ["terraform", "plan"],
    cwd="/workspace/infrastructure",
    env=env,
    check=True,
)
```

---

# 83. Text Mode

Use:

```python
text=True
```

when you want strings rather than bytes.

Without it, stdout may be bytes.

---

# 84. Standard Input

You can provide input:

```python
subprocess.run(
    ["command"],
    input="data\n",
    text=True,
    check=True,
)
```

Avoid passing secrets through command-line arguments because they may appear in process listings or CI logs.

---

# 85. Process Signals

Python can work with signals:

```python
import signal
```

Signals matter in Linux and containers.

Common signals:

```text
SIGTERM
SIGINT
SIGKILL
SIGHUP
```

---

# 86. SIGTERM vs SIGKILL

```text
SIGTERM
    |
    v
process can clean up
```

```text
SIGKILL
    |
    v
process is immediately terminated
```

Applications should generally handle graceful termination where appropriate.

---

# 87. Handling SIGTERM

```python
import signal
import sys


def handle_signal(signum, frame):
    print("Shutdown requested")
    sys.exit(0)


signal.signal(
    signal.SIGTERM,
    handle_signal,
)
```

In production, use proper logging and ensure cleanup is safe and bounded.

---

# 88. Kubernetes and SIGTERM

When a Kubernetes pod is terminated, the application typically receives:

```text
SIGTERM
```

before forceful termination after the configured grace period.

Python applications should use this time to:

```text
stop accepting new work
finish safe operations
close resources
flush logs
exit
```

---

# 89. Graceful Shutdown Flow

```text
Kubernetes
    |
    v
SIGTERM
    |
    v
Python application
    |
    +--> stop new work
    +--> finish safe work
    +--> close resources
    +--> flush logs
    |
    v
exit
```

---

# 90. `os.kill()`

Python can send signals:

```python
import os
import signal

os.kill(
    pid,
    signal.SIGTERM,
)
```

Use extreme caution when operating on production processes.

Validate the PID and intended process before sending signals.

---

# 91. Process Existence

A common Unix technique is:

```python
import os

try:
    os.kill(pid, 0)
except ProcessLookupError:
    print("Process does not exist")
except PermissionError:
    print("Process exists but permission denied")
else:
    print("Process may exist")
```

Signal `0` checks process existence/permission without actually delivering a normal signal.

There can still be race conditions after the check.

---

# 92. Process Information

For richer process inspection, a third-party library such as `psutil` is often used.

Example capabilities:

```text
CPU
memory
PID
process name
open files
network connections
```

Install:

```bash
python -m pip install psutil
```

Then:

```python
import psutil

print(psutil.cpu_percent())
print(psutil.virtual_memory())
```

Use third-party libraries when the project dependency policy allows them.

---

# 93. Linux Monitoring With Python

A DevOps script might check:

```text
CPU
memory
disk
processes
services
network
```

Architecture:

```text
Python
 |
 +--> os
 +--> pathlib
 +--> subprocess
 +--> psutil
 |
 v
health result
```

---

# 94. CPU Check

With `psutil`:

```python
import psutil

usage = psutil.cpu_percent(
    interval=1
)

print(
    f"CPU: {usage}%"
)
```

For a production threshold, define the policy explicitly.

---

# 95. Memory Check

```python
import psutil

memory = psutil.virtual_memory()

print(
    f"Memory: {memory.percent}%"
)
```

---

# 96. Disk Check

```python
import shutil

total, used, free = shutil.disk_usage("/")

percent = used / total * 100

print(
    f"Disk: {percent:.1f}%"
)
```

This uses the standard library.

---

# 97. Process Check

With `psutil`:

```python
import psutil

for process in psutil.process_iter(
    ["pid", "name"]
):
    print(
        process.info["pid"],
        process.info["name"],
    )
```

Do not assume process names are stable across distributions or versions.

---

# 98. Service Check

On Linux:

```python
import subprocess

result = subprocess.run(
    ["systemctl", "is-active", "nginx"],
    capture_output=True,
    text=True,
    check=False,
)

if result.returncode == 0:
    print("nginx is active")
else:
    print("nginx is not active")
```

---

# 99. Network Port Check

Python can use:

```python
import socket

sock = socket.socket(
    socket.AF_INET,
    socket.SOCK_STREAM,
)

sock.settimeout(3)

result = sock.connect_ex(
    ("127.0.0.1", 8080)
)

sock.close()

if result == 0:
    print("Port open")
else:
    print("Port unavailable")
```

Use `socket.create_connection()` for a concise connection test.

---

# 100. HTTP Health Check

The standard library can perform basic HTTP operations, but production projects often use a dedicated HTTP client.

Conceptually:

```text
DNS
 |
 v
TCP
 |
 v
HTTP
 |
 v
status code
 |
 v
application health
```

A successful TCP connection does not guarantee that the application is healthy.

---

# 101. DNS vs Port vs HTTP

When troubleshooting:

```text
DNS works?
   |
   v
TCP port reachable?
   |
   v
TLS works?
   |
   v
HTTP response?
   |
   v
application health?
```

Python automation should check the correct layer.

---

# 102. Environment Health Check Script

A practical DevOps script can validate:

```text
Python version
required environment variables
required commands
disk space
memory
network connectivity
service state
```

Example output:

```text
Python       PASS
Environment  PASS
Disk         PASS
Memory       PASS
Docker       PASS
Kubernetes  PASS
AWS CLI      PASS
```

---

# 103. Check Required Commands

```python
import shutil

commands = [
    "git",
    "docker",
    "kubectl",
    "terraform",
]

for command in commands:
    path = shutil.which(command)

    if path:
        print(
            f"{command}: {path}"
        )
    else:
        print(
            f"{command}: NOT FOUND"
        )
```

This is very useful for CI runner validation.

---

# 104. CI Runner Validation

Before running deployment automation:

```text
Python
 |
 +--> git
 +--> docker
 +--> kubectl
 +--> helm
 +--> terraform
 +--> aws
```

Validate that required tools exist.

Fail early rather than failing halfway through deployment.

---

# 105. Command Version Checks

```python
import subprocess

result = subprocess.run(
    ["terraform", "version"],
    capture_output=True,
    text=True,
    check=False,
)

print(result.stdout)
```

You can validate supported tool versions before running automation.

---

# 106. Tool Version Policy

Example:

```text
Terraform >= approved version
kubectl = supported version range
Python = supported version
Helm = supported version
```

Avoid silently running an incompatible tool version in production.

---

# 107. Working With `PATH`

```python
import os

print(os.getenv("PATH"))
```

To split:

```python
paths = os.getenv(
    "PATH",
    "",
).split(os.pathsep)

for path in paths:
    print(path)
```

`os.pathsep` is platform-aware.

---

# 108. `shutil.which()`

Find executable:

```python
import shutil

print(
    shutil.which("kubectl")
)
```

If unavailable:

```text
None
```

This is preferable to manually parsing `PATH`.

---

# 109. Temporary Environment for a Command

```python
import os
import subprocess

env = os.environ.copy()

env["ENVIRONMENT"] = "production"

result = subprocess.run(
    ["./deploy.sh"],
    env=env,
    capture_output=True,
    text=True,
    check=False,
)
```

This lets the child process receive controlled configuration.

---

# 110. Capture Output Without Leaking Secrets

Be careful:

```python
print(result.stdout)
```

If the command outputs credentials or tokens, this can leak them.

Use:

```text
sanitized logging
```

for sensitive commands.

---

# 111. Shell Command Logging

Useful:

```text
Running: terraform validate
```

Avoid:

```text
Running: curl -H "Authorization: Bearer SECRET..."
```

Mask sensitive values.

---

# 112. Safe Command Runner

A reusable utility:

```python
import subprocess


def run_command(
    command,
    *,
    cwd=None,
    env=None,
    timeout=60,
):
    return subprocess.run(
        command,
        cwd=cwd,
        env=env,
        capture_output=True,
        text=True,
        timeout=timeout,
        check=False,
    )
```

Then:

```python
result = run_command(
    ["terraform", "validate"],
    cwd="/workspace/infra",
)
```

---

# 113. Structured Command Result

```python
def run_command(...):
    result = subprocess.run(...)

    return {
        "command": command,
        "returncode": result.returncode,
        "stdout": result.stdout,
        "stderr": result.stderr,
    }
```

This is easier to use from higher-level automation.

---

# 114. Add Failure Handling

```python
result = run_command(
    ["terraform", "validate"]
)

if result["returncode"] != 0:
    print(
        "Terraform validation failed"
    )
    print(result["stderr"])
    raise SystemExit(1)
```

This creates a clear CI/CD failure.

---

# 115. Terraform Automation Example

```text
Python
  |
  +--> validate environment
  |
  +--> check terraform
  |
  +--> run terraform fmt
  |
  +--> run terraform validate
  |
  +--> run terraform plan
  |
  +--> write report
  |
  +--> exit code
```

This is a realistic use of `os`, `sys`, and `subprocess`.

---

# 116. Docker Automation Example

Python can validate:

```python
import shutil

if not shutil.which("docker"):
    raise RuntimeError(
        "Docker CLI not installed"
    )
```

Then run:

```python
subprocess.run(
    ["docker", "ps"],
    check=True,
)
```

For structured container management, consider the Docker SDK rather than parsing CLI output.

---

# 117. Kubernetes Automation Example

Validate:

```python
if not shutil.which("kubectl"):
    raise RuntimeError(
        "kubectl not installed"
    )
```

Run:

```python
result = subprocess.run(
    [
        "kubectl",
        "get",
        "pods",
        "-n",
        "production",
    ],
    capture_output=True,
    text=True,
    check=False,
)
```

For robust automation, a Kubernetes client can provide structured API objects.

---

# 118. EKS Automation Example

A Python script might:

```text
check AWS identity
        |
        v
check kubectl
        |
        v
check EKS context
        |
        v
get nodes
        |
        v
get unhealthy pods
        |
        v
report
```

This combines:

```text
AWS
Kubernetes
Linux
Python
```

knowledge.

---

# 119. Verify AWS Identity

Using AWS CLI:

```python
result = subprocess.run(
    [
        "aws",
        "sts",
        "get-caller-identity",
    ],
    capture_output=True,
    text=True,
    check=False,
)
```

A more robust Python application can use `boto3` and inspect the caller identity directly.

---

# 120. Verify Kubernetes Context

```python
result = subprocess.run(
    [
        "kubectl",
        "config",
        "current-context",
    ],
    capture_output=True,
    text=True,
    check=False,
)
```

Do not assume the current context is the intended production cluster.

Validate explicitly before destructive operations.

---

# 121. Production Safety — Cluster Context

A dangerous script:

```python
subprocess.run(
    [
        "kubectl",
        "delete",
        "deployment",
        "payment",
    ],
    check=True,
)
```

Better:

```text
validate expected cluster
validate namespace
validate resource
dry-run where supported
require explicit production flag
log operation
```

---

# 122. Explicit Environment Guard

Example:

```python
import os

environment = os.getenv(
    "APP_ENV"
)

if environment != "production":
    raise RuntimeError(
        "Production operation requires "
        "APP_ENV=production"
    )
```

For destructive operations, use multiple independent safety checks rather than relying on one environment variable.

---

# 123. Production Confirmation

For destructive automation:

```text
environment check
+
explicit resource
+
dry-run
+
approval/control
```

Do not make production deletion the default behavior.

---

# 124. Linux Disk Health Script

A practical script:

```python
import shutil
import sys

total, used, free = shutil.disk_usage("/")

usage = used / total * 100

print(
    f"Disk usage: {usage:.1f}%"
)

if usage >= 90:
    print("CRITICAL")
    sys.exit(2)

if usage >= 80:
    print("WARNING")
    sys.exit(1)

print("OK")
sys.exit(0)
```

This can be called from:

```text
cron
Jenkins
GitHub Actions
monitoring checks
incident scripts
```

---

# 125. Why Exit Codes Matter in Monitoring

A monitoring wrapper can interpret:

```text
0 -> healthy
1 -> warning
2 -> critical
```

This is similar to conventions used by many monitoring tools.

Document the exact contract of your script.

---

# 126. Linux Memory Health Script

Using `psutil`:

```python
import sys
import psutil

usage = psutil.virtual_memory().percent

print(
    f"Memory usage: {usage:.1f}%"
)

if usage >= 90:
    sys.exit(2)

if usage >= 80:
    sys.exit(1)

sys.exit(0)
```

Do not assume host memory percentages directly represent container memory limits.

---

# 127. Combined Server Health Script

```text
CPU
 |
 +--> threshold
 |
 v
Memory
 |
 +--> threshold
 |
 v
Disk
 |
 +--> threshold
 |
 v
Service
 |
 +--> state
 |
 v
Overall result
 |
 v
Exit code
```

This is a good real-world Python-for-DevOps project.

---

# 128. Overall Health Logic

```python
statuses = [
    "OK",
    "WARNING",
    "CRITICAL",
]

priority = {
    "OK": 0,
    "WARNING": 1,
    "CRITICAL": 2,
}

overall = max(
    statuses,
    key=lambda status: priority[status],
)
```

The principle is:

```text
overall health = worst component health
```

when that matches your monitoring policy.

---

# 129. Required Environment + System Check

A production preflight script can validate:

```text
Python
environment
disk
memory
required binaries
AWS identity
Kubernetes context
```

Example:

```text
PRE-FLIGHT CHECK
----------------
Python        PASS
Environment   PASS
Disk          PASS
Memory        PASS
AWS CLI       PASS
kubectl       PASS
Terraform     PASS
AWS Identity  PASS
K8s Context   PASS

RESULT: PASS
```

---

# 130. Preflight Script Architecture

```text
preflight.py
     |
     +--> runtime.py
     +--> environment.py
     +--> filesystem.py
     +--> commands.py
     +--> aws.py
     +--> kubernetes.py
     |
     v
structured results
     |
     v
report
     |
     v
exit code
```

This follows the modular architecture from:

```text
01-Modules-and-Packages.md
```

---

# 131. Process Management With `subprocess`

A deployment may start a process:

```python
process = subprocess.Popen(
    ["python", "worker.py"]
)
```

Then:

```python
print(process.pid)
```

For most one-shot DevOps commands, `subprocess.run()` is simpler.

Use `Popen` when you need more advanced process lifecycle or streaming behavior.

---

# 132. `Popen` and Streaming Output

```python
import subprocess

process = subprocess.Popen(
    ["docker", "build", "."],
    stdout=subprocess.PIPE,
    stderr=subprocess.STDOUT,
    text=True,
)

for line in process.stdout:
    print(line, end="")

return_code = process.wait()

print(
    f"Exit code: {return_code}"
)
```

This is useful when command output must be streamed live.

---

# 133. Why Streaming CI Output Matters

If a build runs for:

```text
10 minutes
```

and Python captures everything without output:

```text
CI log appears frozen
```

Streaming output gives operators visibility.

However, make sure sensitive values are masked.

---

# 134. Subprocess Deadlocks

Advanced issue:

```text
stdout pipe
stderr pipe
```

If one pipe fills and is not consumed correctly, a child process can block.

Prefer well-tested patterns such as:

```python
subprocess.run(
    ...,
    capture_output=True,
)
```

for simple commands, or carefully manage streams when using `Popen`.

---

# 135. Shell Pipelines

You might want:

```bash
kubectl get pods | grep payment
```

Avoid blindly building:

```python
subprocess.run(
    "kubectl get pods | grep payment",
    shell=True,
)
```

Instead, either:

```text
use structured Kubernetes APIs
```

or compose commands carefully with pipes and controlled inputs.

---

# 136. Prefer APIs Over CLI Parsing

If an SDK provides structured access:

```text
boto3 -> AWS
kubernetes client -> Kubernetes
Docker SDK -> Docker
```

prefer it over:

```text
CLI -> text parsing
```

when building robust long-lived automation.

CLI wrappers are still useful for tools such as:

```text
Terraform
Helm
Git
kubectl
```

when the CLI is the approved operational interface.

---

# 137. `os` for File Environment

Although `pathlib` is preferred for many path operations, `os` remains important for:

```text
environment
process IDs
permissions
working directory
Unix APIs
low-level OS interaction
```

Know both.

---

# 138. `sys` for Runtime Control

Think of `sys` as:

```text
Python process control
```

It provides:

```text
arguments
exit status
interpreter
module path
runtime version
platform
```

---

# 139. `os` vs `sys`

| Requirement | Module |
|---|---|
| environment variables | `os` |
| filesystem | `os` / `pathlib` |
| process ID | `os` |
| current user ID | `os` |
| working directory | `os` |
| command-line arguments | `sys` |
| exit code | `sys` |
| Python executable | `sys` |
| Python version | `sys` |
| module search path | `sys` |

---

# 140. `os` vs `subprocess`

Use:

```text
os
```

for operating-system information and direct OS interfaces.

Use:

```text
subprocess
```

when you need to execute another program.

Example:

```python
os.getenv("ENV")
```

vs:

```python
subprocess.run(
    ["terraform", "validate"]
)
```

---

# 141. `pathlib` vs `os.path`

Modern Python:

```python
Path("/opt/app") / "config.yaml"
```

Older/common style:

```python
os.path.join(
    "/opt/app",
    "config.yaml",
)
```

Both are useful to understand.

Prefer `pathlib` for new code unless an existing codebase uses another consistent style.

---

# 142. `os.system` vs `subprocess`

Prefer:

```python
subprocess.run()
```

because it gives better control over:

```text
return code
stdout
stderr
timeout
environment
working directory
arguments
```

---

# 143. Production Logging

Do not rely only on:

```python
print()
```

A production script should generally use:

```python
import logging

logging.info("Starting health check")
logging.warning("Disk usage high")
logging.error("Deployment failed")
```

Detailed logging is covered in:

```text
07-Logging.md
```

---

# 144. OS Errors

Many `os` operations can raise:

```text
OSError
PermissionError
FileNotFoundError
FileExistsError
```

Example:

```python
try:
    os.chdir("/protected")
except PermissionError:
    print("Permission denied")
```

Catch the most specific exception you can meaningfully handle.

---

# 145. `errno`

For low-level error handling:

```python
import errno
```

Some applications inspect error codes.

Most DevOps scripts should prefer Python's specific exception types unless they need detailed OS-level handling.

---

# 146. Production Error Boundary

A robust script can structure errors:

```text
main()
 |
 +--> configuration error
 +--> dependency error
 +--> external command error
 +--> validation error
 +--> unexpected error
 |
 v
logging
 |
 v
exit code
```

Do not hide unexpected exceptions.

---

# 147. Avoid Broad Exception Handling

Bad:

```python
try:
    do_everything()
except Exception:
    print("Failed")
```

This can hide useful diagnostics.

Better:

```python
try:
    run_command()
except subprocess.TimeoutExpired:
    ...
except subprocess.CalledProcessError:
    ...
```

Then log unexpected exceptions appropriately.

---

# 148. Environment Preflight Example

```python
import os
import sys


def main():
    required = [
        "APP_ENV",
        "AWS_REGION",
    ]

    missing = [
        name
        for name in required
        if not os.getenv(name)
    ]

    if missing:
        print(
            f"Missing: {missing}"
        )
        sys.exit(2)

    print("Environment OK")


if __name__ == "__main__":
    main()
```

This is a useful pattern for CI/CD.

---

# 149. DevOps CLI Preflight Example

```text
preflight.py
      |
      +--> Python version
      +--> environment
      +--> git
      +--> docker
      +--> kubectl
      +--> helm
      +--> terraform
      |
      v
PASS / FAIL
```

This can prevent confusing mid-pipeline failures.

---

# 150. Jenkins Integration

A Jenkins stage could conceptually run:

```bash
python scripts/preflight.py
```

If:

```text
exit 0
```

pipeline continues.

If:

```text
exit non-zero
```

pipeline stops.

This is the fundamental Python-to-CI/CD integration pattern.

---

# 151. GitHub Actions Integration

A workflow step can execute:

```yaml
- name: Run preflight
  run: python scripts/preflight.py
```

Python's exit status determines the step result.

---

# 152. GitLab CI Integration

A job can run:

```yaml
preflight:
  script:
    - python scripts/preflight.py
```

Again:

```text
exit 0 -> success
exit non-zero -> failure
```

---

# 153. Cron Integration

Linux cron can run:

```bash
*/10 * * * * /opt/automation/.venv/bin/python /opt/automation/scripts/health.py
```

Important:

```text
absolute paths
controlled environment
logging
permissions
timeouts
```

Cron provides a much smaller environment than an interactive shell, so do not assume your usual `PATH` or variables exist.

---

# 154. Cron Environment Problem

Interactive shell:

```text
works
```

Cron:

```text
command not found
```

Possible reason:

```text
different PATH
```

Use:

```python
shutil.which("kubectl")
```

or explicit executable paths where appropriate.

---

# 155. Systemd Integration

Python automation can run under systemd.

Important configuration concepts:

```text
User=
WorkingDirectory=
Environment=
ExecStart=
Restart=
TimeoutStartSec=
```

Python should not assume the working directory or environment is identical to a shell session.

---

# 156. Containerized Python Script

Docker:

```text
ENTRYPOINT ["python", "scripts/health.py"]
```

Environment:

```text
APP_ENV=production
```

Python reads:

```python
os.getenv("APP_ENV")
```

Exit codes determine container/process success.

---

# 157. Container Signals

When a container is stopped:

```text
SIGTERM
```

may be delivered to the process.

Ensure your Python application or worker handles graceful shutdown where required.

---

# 158. Environment Variables and Docker

Example:

```bash
docker run \
  -e APP_ENV=production \
  my-image
```

Python:

```python
import os

print(
    os.getenv("APP_ENV")
)
```

Do not put secrets directly into Dockerfiles.

---

# 159. Environment Variables and Kubernetes

Deployment:

```yaml
env:
  - name: APP_ENV
    value: production
```

Python:

```python
import os

environment = os.getenv(
    "APP_ENV"
)
```

For secrets, use the platform's secret mechanisms rather than plain manifest values committed to Git.

---

# 160. Python OS Automation Project

Build:

```text
server-health/
│
├── scripts/
│   └── health_check.py
│
├── devops_utils/
│   ├── __init__.py
│   ├── environment.py
│   ├── commands.py
│   ├── disk.py
│   └── services.py
│
├── tests/
│   └── test_health.py
│
└── requirements.txt
```

Checks:

```text
environment
disk
memory
required binaries
services
```

---

# 161. Health Script Output

```text
SERVER HEALTH
=============

Environment : PASS
Python      : PASS
Disk        : PASS
Memory      : PASS
Docker      : PASS
Kubernetes  : PASS
Terraform   : PASS
Nginx       : PASS

Overall     : PASS
Exit Code   : 0
```

---

# 162. Health Script Failure

Example:

```text
SERVER HEALTH
=============

Environment : PASS
Python      : PASS
Disk        : CRITICAL
Memory      : PASS
Docker      : PASS
Kubernetes  : PASS
Terraform   : PASS
Nginx       : PASS

Overall     : CRITICAL
Exit Code   : 2
```

CI/monitoring can consume the exit code.

---

# 163. Practical Project — Disk Monitor

Build:

```text
disk_check.py
```

Arguments:

```text
--path
--warning
--critical
```

Example:

```bash
python disk_check.py \
  --path / \
  --warning 80 \
  --critical 90
```

Expected:

```text
OK
WARNING
CRITICAL
```

with meaningful exit codes.

---

# 164. Practical Project — Service Monitor

Build:

```text
service_check.py
```

Arguments:

```text
--service nginx
```

Flow:

```text
check systemctl
     |
     v
service active?
   /      \
 yes      no
  |        |
 PASS     FAIL
```

Return:

```text
0 -> active
1 -> inactive
```

---

# 165. Practical Project — CI Runner Preflight

Check:

```text
Python
Git
Docker
Terraform
kubectl
Helm
AWS CLI
```

Use:

```python
shutil.which()
```

Then run version checks.

Fail early if a required tool is missing.

---

# 166. Practical Project — EKS Preflight

Validate:

```text
AWS identity
AWS region
kubectl available
current context
cluster connectivity
node availability
```

Do not perform any destructive action.

This is a safe first automation project for EKS.

---

# 167. Practical Project — Terraform Preflight

Validate:

```text
terraform exists
version supported
working directory exists
required files exist
AWS identity available
```

Then run:

```text
terraform fmt -check
terraform validate
```

Do not run `apply` from the preflight script.

---

# 168. Practical Project — Deployment Wrapper

Build:

```text
deploy.py
```

Flow:

```text
read environment
      |
      v
validate tools
      |
      v
validate configuration
      |
      v
run tests
      |
      v
run security checks
      |
      v
deploy
      |
      v
verify
      |
      v
exit code
```

This combines many concepts learned so far.

---

# 169. Production Safety for Deployment Wrappers

A deployment wrapper should have:

```text
explicit environment
explicit artifact/image
validation
timeouts
bounded retries
safe command execution
sanitized logs
rollback strategy
exit codes
```

Never make production deployment happen accidentally because a default environment was missing.

---

# 170. Interview — What Is the Difference Between `os` and `sys`?

Answer:

> `os` provides operating-system interfaces such as environment variables, filesystem operations, process IDs, working directories, and Unix-related functionality. `sys` provides access to the Python runtime such as command-line arguments, interpreter information, module search paths, and process exit codes.

---

# 171. Interview — How Do You Read Environment Variables?

Answer:

> I normally use `os.getenv()` for optional values and validate required variables explicitly. For secrets I avoid hard-coding credentials and use the organization's approved secret or workload-identity mechanism.

---

# 172. Interview — How Do You Pass Environment Variables to a Subprocess?

Answer:

> I copy the current environment with `os.environ.copy()`, modify only the required variables, and pass that dictionary using the `env` argument of `subprocess.run()` or `Popen()`.

---

# 173. Interview — Why Prefer `subprocess` Over `os.system()`?

Answer:

> `subprocess` gives much better control over command arguments, stdout, stderr, return codes, timeouts, environment variables, and working directory. It also makes error handling and automation behavior more explicit.

---

# 174. Interview — How Do You Make a Python Script Fail a CI Pipeline?

Answer:

> I return a non-zero exit code for failure. For example, `sys.exit(1)` or another documented non-zero status. The CI system interprets the exit status and marks the job as failed.

---

# 175. Interview — What Is `sys.executable` Useful For?

Answer:

> It tells me the exact Python interpreter running the script. I use it when troubleshooting virtual-environment or CI/CD problems where `python`, `pip`, and the runtime may point to different installations.

---

# 176. Interview — How Do You Check Whether a Command Exists?

Answer:

> I use `shutil.which()`. It searches the executable path and returns the executable location if found. This is useful for CI runner preflight checks for tools such as Terraform, Docker, kubectl, Helm, and AWS CLI.

---

# 177. Interview — How Do You Execute a Linux Command From Python?

Answer:

> I normally use `subprocess.run()` with an argument list, capture stdout/stderr when needed, set an appropriate timeout, and inspect the return code. I avoid `shell=True` with untrusted input.

---

# 178. Interview — What Is Command Injection?

Answer:

> Command injection occurs when untrusted input is incorporated into a shell command in a way that allows unintended commands to execute. I reduce this risk by avoiding shell interpretation, passing arguments as a list, validating inputs, and restricting destructive operations.

---

# 179. Interview — How Do You Handle Long-Running Commands?

Answer:

> I set a timeout for commands that have an expected completion window. For commands where I need live output, I may use `subprocess.Popen()` and stream stdout while carefully handling stderr and process termination.

---

# 180. Interview — How Do You Handle SIGTERM in Kubernetes?

Answer:

> Kubernetes normally sends SIGTERM during graceful pod termination. The application should stop accepting new work, finish safe operations, close resources, flush important logs, and exit within the termination grace period.

---

# 181. Interview — Why Should You Not Run Every Python Script as Root?

Answer:

> Running everything as root increases the blast radius of a bug or compromised dependency. I prefer least privilege: a normal Linux user where possible, minimal IAM permissions in AWS, and minimal RBAC permissions in Kubernetes.

---

# 182. Scenario — Script Works Manually but Fails in Jenkins

Investigate:

```text
Python executable
PATH
environment variables
working directory
permissions
installed dependencies
tool versions
credentials/identity
```

Commands:

```bash
which python
python --version
id
pwd
env
```

Do not print secrets from `env` in a real CI log.

---

# 183. Scenario — `kubectl` Works Manually but Python Fails

Check:

```text
PATH
KUBECONFIG
current user
working directory
Python subprocess environment
kubectl path
cluster context
```

Python:

```python
import os
import shutil

print(
    shutil.which("kubectl")
)

print(
    os.getenv("KUBECONFIG")
)
```

Avoid exposing credential material.

---

# 184. Scenario — AWS Works in Shell but Not Python

Check:

```text
AWS profile
AWS_REGION
credential source
IAM role
environment
Python interpreter
boto3 version
```

If using a Python SDK, inspect the SDK's credential chain rather than assuming the shell's configuration is identical.

---

# 185. Scenario — Terraform Command Hangs

Possible causes:

```text
provider network call
credential prompt
state lock
network issue
interactive input
```

Mitigate:

```text
timeout
non-interactive flags where appropriate
CI environment configuration
clear logging
bounded retries
```

Do not blindly kill the process without understanding whether it holds a state lock or is performing a legitimate operation.

---

# 186. Scenario — Python Script Deletes Production Resources

Immediate concern:

```text
What environment?
What credentials?
What resource?
Was input validated?
Was dry-run available?
Was approval required?
```

Production destructive automation should have strong guardrails.

---

# 187. Scenario — Process Uses High CPU

Use Linux:

```bash
top
ps -ef
```

and Python/`psutil` where appropriate.

Determine:

```text
which PID
which process
CPU usage
command line
parent process
```

Then investigate application behavior rather than immediately killing the process.

---

# 188. Scenario — Disk Usage High

Start with Linux:

```bash
df -h
df -i
du -xhd1 /
```

Then Python can automate:

```text
large-file discovery
old-file discovery
report generation
safe cleanup
```

Remember that inode exhaustion and byte capacity are different problems.

---

# 189. Scenario — Memory High

Check:

```text
free -h
top
ps
```

Then determine:

```text
application
container
node
cache
process leak
```

Python automation can report memory usage, but diagnosis requires understanding Linux memory behavior.

---

# 190. Scenario — Service Down

Flow:

```text
systemctl status
       |
       v
logs
       |
       v
configuration
       |
       v
ports
       |
       v
dependencies
```

Python can automate detection and reporting but should not automatically restart services unless the recovery policy is explicitly designed and safe.

---

# 191. Scenario — CI Job Exits 0 Despite Failure

Likely cause:

```python
print("ERROR")
```

without:

```python
sys.exit(1)
```

Fix:

```python
if failure:
    sys.exit(1)
```

This is a very common automation mistake.

---

# 192. Scenario — Python Uses Wrong Configuration

Possible cause:

```text
relative path
wrong working directory
missing environment variable
different CI workspace
```

Use:

```python
print(os.getcwd())
print(sys.executable)
```

and log resolved configuration paths safely.

---

# 193. Scenario — Production Script Is Run From Different Directory

Avoid relying on:

```python
open("config.yaml")
```

unless the working directory is explicitly controlled.

Use a defined configuration path:

```python
config_path = (
    Path(__file__).resolve().parent
    / "../config/config.yaml"
).resolve()
```

Then validate the resolved location.

---

# 194. Scenario — Script Needs Different Behavior by Environment

Use explicit configuration:

```text
dev
staging
production
```

Do not scatter environment checks throughout the code.

Prefer:

```text
configuration
   |
   v
validated settings
   |
   v
business logic
```

---

# 195. Production Best Practices

```text
1. Validate environment variables.
2. Never hard-code secrets.
3. Use least privilege.
4. Prefer subprocess over os.system.
5. Avoid shell=True with untrusted input.
6. Use command timeouts.
7. Capture and inspect return codes.
8. Use explicit working directories.
9. Validate production context.
10. Use meaningful exit codes.
11. Log safely without secrets.
12. Handle SIGTERM where required.
13. Validate external tool versions.
14. Prefer SDKs/APIs over text parsing when appropriate.
15. Keep reusable logic in modules.
16. Test failure paths.
17. Use absolute or deliberately resolved paths.
18. Make destructive actions explicit and guarded.
19. Keep CI environments reproducible.
20. Document platform assumptions.
```

---

# 196. Daily DevOps Scripts You Should Be Able to Build

By the end of this topic, you should be comfortable building:

```text
check_disk.py
check_memory.py
check_service.py
check_port.py
check_tools.py
preflight.py
aws_identity_check.py
eks_health.py
terraform_preflight.py
docker_health.py
k8s_health.py
deployment_wrapper.py
cleanup.py
backup.py
log_analyzer.py
```

The purpose is not memorizing each script.

The goal is understanding the reusable patterns behind them.

---

# 197. Reusable DevOps Pattern

```text
INPUT
  |
  v
VALIDATE
  |
  v
EXECUTE
  |
  v
CAPTURE RESULT
  |
  v
EVALUATE
  |
  +------+
  |      |
PASS    FAIL
 |       |
 v       v
REPORT  LOG ERROR
 |       |
 +---+---+
     |
     v
EXIT CODE
```

This pattern appears in almost every production automation script.

---

# 198. Final Mental Model

Remember:

```text
os
 |
 +--> environment
 +--> filesystem
 +--> process identity
 +--> working directory
 +--> OS interfaces

sys
 |
 +--> CLI arguments
 +--> exit codes
 +--> Python runtime
 +--> executable
 +--> module search path
 +--> platform
```

Together with:

```text
subprocess
 |
 v
external tools
 |
 +--> Terraform
 +--> Docker
 +--> kubectl
 +--> Helm
 +--> Git
 +--> AWS CLI
 +--> Linux commands
```

This gives Python the ability to act as an **automation layer around Linux, AWS, Kubernetes, Terraform, Docker, and CI/CD**.

---

# 199. Next File

```text
04-Regex.md
```

The next file will cover regex deeply for DevOps, including:

```text
regex fundamentals
patterns
character classes
quantifiers
groups
capture groups
named groups
search
match
findall
finditer
substitution
validation
IP addresses
URLs
ports
timestamps
log parsing
Kubernetes logs
CI/CD logs
Terraform output
AWS output
security reports
large-file processing
production regex practices
performance
common mistakes
DevOps scripts
interview questions
scenario-based troubleshooting
```
