# 01-Linux-Automation

## Python for Linux DevOps

Python is widely used by DevOps engineers to automate Linux administration, diagnostics, deployment prechecks, maintenance, and incident response.

Typical automation includes:

- Command execution
- File and directory management
- Permissions and ownership
- Process inspection
- Service management
- CPU, memory, disk and inode checks
- Log analysis
- Network connectivity checks
- Configuration validation
- Cleanup and backup
- CI/CD helper scripts
- Production diagnostics

> **Core principle:** Use Python to automate repeatable Linux operations while keeping commands explicit, validated, observable, idempotent, and safe.

---

# 1. Linux Automation Mindset

A production automation script should not simply run a command.

A stronger flow is:

```text
Input
  |
  v
Validate
  |
  v
Preflight
  |
  v
Collect current state
  |
  v
Perform action
  |
  v
Verify result
  |
  +----> Failure -> recovery/escalation
  |
  v
Log + exit code
```

This mindset is useful for:

- Deployment scripts
- Maintenance jobs
- Health checks
- Incident diagnostics
- Service remediation
- CI/CD automation

---

# 2. Why Python for Linux Automation?

Bash is excellent for:

```text
short command sequences
shell pipelines
simple administration
```

Python becomes more useful when you need:

```text
complex logic
structured data
reusable functions
testing
JSON/YAML
API integration
concurrency
error handling
logging
AWS/Kubernetes integration
```

Do not replace Bash just because Python exists. Choose the simplest reliable tool.

---

# 3. Important Python Modules

Common modules for Linux DevOps automation:

```python
os
pathlib
subprocess
shutil
platform
socket
logging
tempfile
json
argparse
time
datetime
pwd
grp
resource
```

Useful third-party package:

```text
psutil
```

Common mapping:

```text
Filesystem      -> pathlib / shutil
Commands        -> subprocess
Processes       -> psutil
System info     -> platform / os / /proc
Networking      -> socket
Logging         -> logging
CLI             -> argparse
Structured data -> json
```

---

# 4. Basic Linux Python Script

```python
#!/usr/bin/env python3

print("Linux automation started")
```

Run:

```bash
python3 script.py
```

Or:

```bash
chmod +x script.py
./script.py
```

For a project virtual environment:

```bash
.venv/bin/python script.py
```

---

# 5. Current User

Linux:

```bash
whoami
```

Python:

```python
import getpass

print(getpass.getuser())
```

Numeric identity:

```python
import os

print("UID:", os.getuid())
print("GID:", os.getgid())
```

This is useful when behavior differs between:

```text
root
jenkins
ec2-user
ubuntu
application users
```

---

# 6. Check for Root

```python
import os

if os.geteuid() == 0:
    print("Running as root")
else:
    print("Not running as root")
```

Some operations require elevated privileges, but do not automatically run everything as root.

Use least privilege.

---

# 7. Environment Variables

```python
import os

environment = os.environ.get("ENVIRONMENT")

if not environment:
    raise RuntimeError("ENVIRONMENT is not configured")

print(environment)
```

Required variables:

```python
required = [
    "ENVIRONMENT",
    "APP_NAME",
]

for name in required:
    if not os.environ.get(name):
        raise RuntimeError(
            f"Missing environment variable: {name}"
        )
```

Never hardcode passwords, tokens, or private keys.

---

# 8. Current Working Directory

```python
from pathlib import Path

print(Path.cwd())
```

Although `os.chdir()` exists:

```python
import os

os.chdir("/tmp")
```

prefer explicit `Path` objects and absolute paths in larger automation.

---

# 9. Execute Linux Commands

Use `subprocess`.

```python
import subprocess

result = subprocess.run(
    ["hostname"],
    capture_output=True,
    text=True,
)

print(result.stdout)
print(result.returncode)
```

Important properties include:

```text
stdout
stderr
returncode
timeout
environment
input
```

---

# 10. Prefer Argument Lists

Preferred:

```python
subprocess.run(
    ["systemctl", "restart", "nginx"],
    check=True,
)
```

Avoid constructing shell strings unnecessarily:

```python
subprocess.run(
    "systemctl restart nginx",
    shell=True,
)
```

Argument lists avoid unnecessary shell parsing and reduce injection risk.

---

# 11. `check=True`

```python
subprocess.run(
    ["systemctl", "status", "nginx"],
    check=True,
)
```

If the command returns a non-zero exit code, Python raises:

```text
subprocess.CalledProcessError
```

Use this when command failure should stop the operation.

---

# 12. Handling Command Failure

```python
import subprocess

try:
    subprocess.run(
        ["systemctl", "restart", "nginx"],
        check=True,
    )
except subprocess.CalledProcessError as exc:
    print(f"Command failed: {exc}")
```

For production, use logging rather than only `print()`.

---

# 13. Capture stderr

```python
result = subprocess.run(
    ["systemctl", "status", "nginx"],
    capture_output=True,
    text=True,
)

print("STDOUT:")
print(result.stdout)

print("STDERR:")
print(result.stderr)

print("Return code:", result.returncode)
```

`stderr` often contains the most useful diagnostic information.

---

# 14. Command Timeout

Never allow external operations to hang forever.

```python
import subprocess

try:
    subprocess.run(
        ["curl", "-I", "https://example.com"],
        capture_output=True,
        text=True,
        timeout=10,
        check=True,
    )
except subprocess.TimeoutExpired:
    print("Command timed out")
```

Timeouts are especially important for:

```text
network operations
service checks
CI jobs
deployment scripts
external tools
```

---

# 15. `shell=True` Security

This is dangerous when input is untrusted:

```python
subprocess.run(
    f"rm -rf {user_input}",
    shell=True,
)
```

If input contains shell syntax, it can become command injection.

Prefer:

```python
subprocess.run(
    ["rm", "-f", validated_path],
    check=True,
)
```

For destructive filesystem work, prefer Python filesystem APIs after strong validation.

---

# 16. Check Command Availability

```python
import shutil

path = shutil.which("docker")

if path:
    print("Docker:", path)
else:
    print("Docker not installed")
```

Preflight:

```python
for command in ["curl", "systemctl"]:
    if not shutil.which(command):
        raise RuntimeError(
            f"Required command missing: {command}"
        )
```

---

# 17. Reusable Command Runner

```python
import subprocess


def run_command(command, timeout=30):
    try:
        result = subprocess.run(
            command,
            capture_output=True,
            text=True,
            check=True,
            timeout=timeout,
        )
        return result.stdout.strip()

    except subprocess.TimeoutExpired as exc:
        raise RuntimeError(
            f"Command timed out: {command}"
        ) from exc

    except subprocess.CalledProcessError as exc:
        raise RuntimeError(
            f"Command failed: {command}
"
            f"{exc.stderr}"
        ) from exc
```

This pattern centralizes:

```text
timeouts
error handling
output handling
```

---

# 18. Logging

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)s %(message)s",
)

logger = logging.getLogger(__name__)

logger.info("Linux automation started")
```

Production automation should produce useful operational logs.

Never log:

```text
passwords
tokens
private keys
secret environment variables
```

---

# 19. Exit Codes

Success:

```python
raise SystemExit(0)
```

Failure:

```python
raise SystemExit(1)
```

CI/CD and monitoring systems depend on exit codes.

Bad:

```text
log error
continue
exit 0
```

Better:

```text
log error
return non-zero
```

---

# 20. Main Function Pattern

```python
def main():
    validate()
    preflight()
    collect_state()
    perform_action()
    verify()
    return 0


if __name__ == "__main__":
    raise SystemExit(main())
```

This structure makes automation easier to test and maintain.

---

# 21. Exception Boundary

```python
import logging

logger = logging.getLogger(__name__)


def main():
    try:
        run_automation()
        return 0
    except Exception:
        logger.exception("Automation failed")
        return 1


if __name__ == "__main__":
    raise SystemExit(main())
```

Do not catch exceptions too early and hide useful failures.

---

# 22. Filesystem Automation with pathlib

```python
from pathlib import Path

log_dir = Path("/var/log")

print(log_dir.exists())
print(log_dir.is_dir())
```

Use `pathlib` instead of manually joining strings.

---

# 23. Create Directory

```python
from pathlib import Path

directory = Path("/tmp/devops")

directory.mkdir(
    parents=True,
    exist_ok=True,
)
```

This is naturally idempotent.

---

# 24. Create and Read Files

Write:

```python
from pathlib import Path

Path("/tmp/status.txt").write_text(
    "healthy
",
    encoding="utf-8",
)
```

Read:

```python
content = Path("/tmp/status.txt").read_text(
    encoding="utf-8"
)

print(content)
```

---

# 25. Large File Handling

Do not load a multi-GB log into memory:

```python
content = file.read_text()
```

Prefer:

```python
from pathlib import Path

with Path("/var/log/app.log").open(
    "r",
    encoding="utf-8",
    errors="replace",
) as handle:
    for line in handle:
        if "ERROR" in line:
            print(line.rstrip())
```

---

# 26. File Metadata

```python
from pathlib import Path

path = Path("/etc/hosts")
stat = path.stat()

print("Size:", stat.st_size)
print("UID:", stat.st_uid)
print("GID:", stat.st_gid)
print("Mode:", oct(stat.st_mode))
```

---

# 27. Permissions

```python
import os

os.chmod(
    "/tmp/script.sh",
    0o750,
)
```

Do not use `chmod 777` as a generic troubleshooting solution.

Understand:

```text
owner
group
mode
ACL
SELinux/AppArmor
```

before changing permissions.

---

# 28. Ownership

Unix systems provide:

```python
import os

os.chown(
    "/tmp/file.txt",
    uid,
    gid,
)
```

This usually requires appropriate privileges.

Do not hardcode arbitrary IDs when portability matters.

---

# 29. User Lookup

```python
import pwd

try:
    user = pwd.getpwnam("appuser")
    print(user.pw_uid)
    print(user.pw_gid)
    print(user.pw_dir)
except KeyError:
    print("User does not exist")
```

This is preferable to parsing `id` command output.

---

# 30. Group Lookup

```python
import grp

try:
    group = grp.getgrnam("docker")
    print(group.gr_gid)
except KeyError:
    print("Group does not exist")
```

---

# 31. Idempotent User Creation

```python
import pwd
import subprocess

username = "appuser"

try:
    pwd.getpwnam(username)
    print("User already exists")
except KeyError:
    subprocess.run(
        ["useradd", "--create-home", username],
        check=True,
    )
```

The second run should not fail merely because the desired user already exists.

---

# 32. Idempotency

Idempotent automation means:

```text
Run once  -> desired state
Run again -> same desired state
```

Examples:

```text
create directory if missing
create user if missing
write configuration only when required
start service only when necessary
```

Idempotency is essential because CI/CD and automation often retry.

---

# 33. Safe Configuration Changes

Preferred workflow:

```text
read current config
      |
      v
validate new config
      |
      v
backup current config
      |
      v
write new config
      |
      v
reload/restart
      |
      v
health check
```

Do not modify critical production configuration blindly.

---

# 34. Atomic File Replacement

```python
from pathlib import Path
import os
import tempfile

target = Path("/tmp/app.conf")

with tempfile.NamedTemporaryFile(
    mode="w",
    dir=target.parent,
    delete=False,
    encoding="utf-8",
) as handle:
    handle.write("PORT=8080
")
    temporary = handle.name

os.replace(temporary, target)
```

Atomic replacement can prevent readers from seeing a partially written file.

---

# 35. Copy and Backup

```python
import shutil

shutil.copy2(
    "/etc/myapp/app.conf",
    "/tmp/app.conf.backup",
)
```

For production systems, use an approved backup/retention strategy.

---

# 36. Delete Old Files

```python
from pathlib import Path
import time

directory = Path("/tmp/app")
cutoff = time.time() - 7 * 24 * 60 * 60

for path in directory.iterdir():
    if path.is_file():
        if path.stat().st_mtime < cutoff:
            path.unlink()
```

Before destructive cleanup, validate:

```text
allowed root
file type
age
ownership
```

---

# 37. Dry Run

Destructive tools should support:

```text
--dry-run
```

Example:

```python
if dry_run:
    logger.info("Would delete %s", path)
else:
    path.unlink()
```

Use dry-run to inspect what automation would change before executing it.

---

# 38. Dangerous Path Validation

```python
from pathlib import Path

allowed_root = Path("/var/log").resolve()
target = Path(user_input).resolve()

if allowed_root != target and allowed_root not in target.parents:
    raise ValueError(
        "Target is outside allowed root"
    )
```

This is only one layer of protection. The complete security model should also consider symlinks, permissions, and race conditions.

---

# 39. Large File Discovery

```python
from pathlib import Path

root = Path("/var/log")

for path in root.rglob("*"):
    try:
        if path.is_file():
            if path.stat().st_size > 100 * 1024 * 1024:
                print(path)
    except OSError:
        continue
```

Do not blindly scan `/` in frequent automation. Avoid unnecessary traversal of:

```text
/proc
/sys
/dev
/run
```

---

# 40. Log Analysis

```python
from pathlib import Path

error_count = 0

with Path("/var/log/app.log").open(
    encoding="utf-8",
    errors="replace",
) as handle:
    for line in handle:
        if "ERROR" in line:
            error_count += 1

print("Errors:", error_count)
```

For production logging, understand whether the system uses:

```text
logrotate
journald
container logs
ELK
centralized logging
```

before manipulating local files.

---

# 41. Search Multiple Patterns

```python
patterns = [
    "ERROR",
    "CRITICAL",
    "Traceback",
]

with open(
    "/var/log/app.log",
    encoding="utf-8",
    errors="replace",
) as handle:
    for line in handle:
        if any(
            pattern in line
            for pattern in patterns
        ):
            print(line.rstrip())
```

---

# 42. System Information

```python
import platform

print(platform.system())
print(platform.release())
print(platform.machine())
print(platform.platform())
```

Useful when scripts must support multiple Linux distributions or architectures.

---

# 43. Linux `/etc/os-release`

```python
from pathlib import Path

data = {}

for line in Path(
    "/etc/os-release"
).read_text().splitlines():
    if "=" in line:
        key, value = line.split("=", 1)
        data[key] = value.strip('"')

print(data.get("ID"))
print(data.get("VERSION_ID"))
```

This is useful for distribution-specific behavior.

---

# 44. Package Manager Detection

```python
import shutil

if shutil.which("dnf"):
    package_manager = "dnf"
elif shutil.which("apt-get"):
    package_manager = "apt-get"
else:
    raise RuntimeError(
        "Unsupported package manager"
    )
```

Do not assume Ubuntu and RHEL-family systems have identical commands or paths.

---

# 45. Package Management

Example:

```python
subprocess.run(
    ["dnf", "install", "-y", "nginx"],
    check=True,
)
```

Package installation changes system state. Validate:

```text
package name
repository
version
permissions
disk space
authorization
```

Use the organization's configuration-management system when appropriate.

---

# 46. Process Management with psutil

Install:

```bash
python -m pip install psutil
```

Inspect processes:

```python
import psutil

for process in psutil.process_iter(
    ["pid", "name", "username"]
):
    print(process.info)
```

This is often more reliable than parsing `ps -ef`.

---

# 47. Find Process

```python
import psutil

target = "nginx"

for process in psutil.process_iter(
    ["pid", "name"]
):
    if process.info["name"] == target:
        print(process.info)
```

Process names can vary. For service health, systemd state is usually more authoritative.

---

# 48. Process Memory

```python
import psutil

for process in psutil.process_iter(
    ["pid", "name", "memory_percent"]
):
    print(process.info)
```

This is useful during:

```text
OOM investigations
memory leak analysis
capacity checks
incident diagnostics
```

---

# 49. CPU Usage

```python
import psutil

usage = psutil.cpu_percent(
    interval=1
)

print(f"CPU: {usage:.1f}%")
```

Do not treat one instantaneous reading as proof of a production incident.

Correlate with:

```text
load
process CPU
traffic
logs
deployment activity
```

---

# 50. Memory Usage

```python
import psutil

memory = psutil.virtual_memory()

print("Total:", memory.total)
print("Available:", memory.available)
print("Used:", memory.used)
print("Percent:", memory.percent)
```

---

# 51. Disk Usage

```python
import shutil

total, used, free = shutil.disk_usage("/")

usage = used / total * 100

print(f"Disk usage: {usage:.1f}%")
```

---

# 52. Multiple Mount Points

```python
import shutil

for mount in ["/", "/var", "/tmp"]:
    try:
        total, used, free = shutil.disk_usage(mount)
        usage = used / total * 100
        print(f"{mount}: {usage:.1f}%")
    except FileNotFoundError:
        print(f"{mount}: not present")
```

---

# 53. Inode Usage

Disk blocks can be available while inodes are exhausted.

Linux:

```bash
df -i
```

Python can invoke it:

```python
result = subprocess.run(
    ["df", "-i"],
    capture_output=True,
    text=True,
    check=True,
)

print(result.stdout)
```

This is important during production disk incidents.

---

# 54. Load Average

```python
import os

load_1, load_5, load_15 = os.getloadavg()

print(load_1, load_5, load_15)
```

Interpret load relative to:

```text
CPU count
workload type
historical baseline
```

---

# 55. `/proc` Automation

Linux exposes useful information through:

```text
/proc/cpuinfo
/proc/meminfo
/proc/loadavg
/proc/uptime
```

Example:

```python
from pathlib import Path

for line in Path(
    "/proc/meminfo"
).read_text().splitlines():
    if line.startswith("MemAvailable:"):
        print(line)
```

This is Linux-specific but valuable for low-level diagnostics.

---

# 56. Uptime

```python
from pathlib import Path

seconds = float(
    Path("/proc/uptime")
    .read_text()
    .split()[0]
)

print("Uptime seconds:", seconds)
```

This can be converted to days/hours/minutes for reports.

---

# 57. Resource Limits

```python
import resource

soft, hard = resource.getrlimit(
    resource.RLIMIT_NOFILE
)

print("Open file limit:", soft, hard)
```

Useful when diagnosing:

```text
Too many open files
connection exhaustion
high descriptor usage
```

---

# 58. Service Management

Modern Linux commonly uses systemd.

Check:

```python
import subprocess

result = subprocess.run(
    ["systemctl", "is-active", "nginx"],
    capture_output=True,
    text=True,
)

print(result.stdout.strip())
```

Possible states include:

```text
active
inactive
failed
```

---

# 59. Service Health

Restart:

```python
subprocess.run(
    ["systemctl", "restart", "nginx"],
    check=True,
)
```

Verify:

```python
result = subprocess.run(
    [
        "systemctl",
        "is-active",
        "--quiet",
        "nginx",
    ],
)

if result.returncode != 0:
    raise RuntimeError(
        "nginx is not active"
    )
```

Service state is not the same as application health.

---

# 60. Service Health Should Be Layered

A strong check can be:

```text
systemd state
      |
      v
TCP port
      |
      v
HTTP endpoint
      |
      v
expected response
```

A process being active does not prove the application is healthy.

---

# 61. systemd Reload

When a unit file changes:

```python
subprocess.run(
    ["systemctl", "daemon-reload"],
    check=True,
)
```

Then:

```python
subprocess.run(
    ["systemctl", "restart", "myapp"],
    check=True,
)
```

Use `daemon-reload` only when unit definitions changed.

---

# 62. Journal Logs

```python
result = subprocess.run(
    [
        "journalctl",
        "-u",
        "nginx",
        "-n",
        "50",
        "--no-pager",
    ],
    capture_output=True,
    text=True,
)

print(result.stdout)
```

Recent logs:

```python
[
    "journalctl",
    "-u",
    "nginx",
    "--since",
    "10 minutes ago",
    "--no-pager",
]
```

---

# 63. Failed Services

```python
result = subprocess.run(
    [
        "systemctl",
        "--failed",
        "--no-legend",
    ],
    capture_output=True,
    text=True,
)

print(result.stdout)
```

Use this for diagnostics rather than automatically restarting every failed service.

---

# 64. Network Connectivity

TCP connectivity:

```python
import socket

try:
    with socket.create_connection(
        ("example.com", 443),
        timeout=5,
    ):
        print("TCP reachable")
except OSError as exc:
    print(f"Connection failed: {exc}")
```

This tests TCP connectivity, not full application health.

---

# 65. DNS Check

```python
import socket

try:
    address = socket.gethostbyname(
        "example.com"
    )
    print(address)
except socket.gaierror as exc:
    print(f"DNS failure: {exc}")
```

---

# 66. HTTP Health Check

```python
from urllib.request import urlopen

with urlopen(
    "https://example.com/health",
    timeout=5,
) as response:
    print(response.status)
```

For complex HTTP automation, use an approved HTTP client library.

---

# 67. Network Diagnostic Sequence

When an application is unreachable:

```text
DNS
 |
 v
Route
 |
 v
TCP port
 |
 v
TLS
 |
 v
HTTP
 |
 v
Application dependency
```

Do not immediately restart the application without checking lower layers.

---

# 68. Listening Ports

Linux:

```bash
ss -lntp
```

Python:

```python
result = subprocess.run(
    ["ss", "-lntp"],
    capture_output=True,
    text=True,
    check=True,
)

print(result.stdout)
```

Use structured APIs where available rather than parsing human-readable output for complex automation.

---

# 69. Network Routes

```python
subprocess.run(
    ["ip", "route"],
    check=True,
)
```

Useful for:

```text
missing default route
wrong route
interface problems
```

---

# 70. Environment and Network Safety

Never log credentials embedded in:

```text
URLs
environment variables
command arguments
configuration
```

If a command must be logged, redact secrets.

---

# 71. Python + AWS EC2

For AWS API operations, prefer:

```python
import boto3

ec2 = boto3.client("ec2")

response = ec2.describe_instances()
```

rather than parsing:

```bash
aws ec2 describe-instances
```

when structured programmatic access is required.

Use IAM roles or approved credential mechanisms instead of hardcoded keys.

---

# 72. Python + Kubernetes

Python can invoke:

```bash
kubectl
```

through `subprocess`, but for structured Kubernetes automation, the Kubernetes Python client is usually preferable.

Examples of use cases:

```text
pod diagnostics
deployment checks
resource inspection
custom controllers
health checks
```

---

# 73. Python + Terraform

Python can orchestrate:

```python
subprocess.run(
    ["terraform", "plan"],
    check=True,
)
```

But Terraform should remain the authoritative infrastructure state-management tool.

Do not hide infrastructure changes behind opaque Python logic.

---

# 74. Python + Docker

CLI:

```python
subprocess.run(
    ["docker", "ps"],
    check=True,
)
```

For structured automation, consider an appropriate Docker SDK.

Avoid parsing CLI output when a stable API is available.

---

# 75. Command Output Parsing

Human-readable output can change between tool versions.

For complex logic:

```text
stable API
    >
JSON output
    >
CLI text parsing
```

when those interfaces are available.

---

# 76. Structured JSON Output

```python
import json

result = {
    "hostname": "server01",
    "healthy": True,
    "disk_percent": 62.4,
}

print(
    json.dumps(
        result,
        indent=2,
    )
)
```

JSON is useful for:

```text
CI/CD
monitoring
automation
incident reports
API consumers
```

---

# 77. CPU + Memory + Disk Health Check

```python
import shutil

import psutil

cpu = psutil.cpu_percent(interval=1)
memory = psutil.virtual_memory().percent

total, used, _ = shutil.disk_usage("/")
disk = used / total * 100

print(f"CPU: {cpu:.1f}%")
print(f"Memory: {memory:.1f}%")
print(f"Disk: {disk:.1f}%")
```

---

# 78. Threshold Health Check

```python
healthy = True

if cpu >= 90:
    healthy = False

if memory >= 90:
    healthy = False

if disk >= 90:
    healthy = False

raise SystemExit(
    0 if healthy else 1
)
```

Thresholds must be workload-specific.

---

# 79. Complete Health Check

```python
#!/usr/bin/env python3

import json
import shutil
import socket
import sys

import psutil


CPU_LIMIT = 90
MEMORY_LIMIT = 90
DISK_LIMIT = 90


def collect():
    total, used, _ = shutil.disk_usage("/")

    return {
        "hostname": socket.gethostname(),
        "cpu_percent": psutil.cpu_percent(
            interval=1
        ),
        "memory_percent":
            psutil.virtual_memory().percent,
        "disk_percent":
            used / total * 100,
    }


def main():
    result = collect()

    result["healthy"] = (
        result["cpu_percent"] < CPU_LIMIT
        and result["memory_percent"] < MEMORY_LIMIT
        and result["disk_percent"] < DISK_LIMIT
    )

    print(
        json.dumps(
            result,
            indent=2,
        )
    )

    return 0 if result["healthy"] else 1


if __name__ == "__main__":
    raise SystemExit(main())
```

---

# 80. Dry-Run Cleanup Example

```python
def delete_file(path, dry_run=False):
    if dry_run:
        logger.info(
            "Would delete %s",
            path,
        )
        return

    path.unlink()

    logger.info(
        "Deleted %s",
        path,
    )
```

A destructive script should default to safe behavior where practical.

---

# 81. Cron

Example:

```cron
*/5 * * * * /opt/health/.venv/bin/python /opt/health/check.py
```

Use:

```text
absolute interpreter
absolute script path
explicit environment
```

Cron has a minimal environment and does not behave like an interactive shell.

---

# 82. systemd Service

Example:

```ini
[Service]
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/.venv/bin/python /opt/myapp/app.py
Restart=on-failure
```

systemd can directly execute the virtual environment's Python interpreter. Activation is not required.

---

# 83. CI/CD

Jenkins/GitLab/GitHub Actions can run:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m pytest
```

A clean environment prevents accidental dependence on globally installed packages.

---

# 84. Linux Automation + DevSecOps

A pipeline can be:

```text
Checkout
   |
   v
Python environment
   |
   v
Install dependencies
   |
   v
SCA / Trivy
   |
   v
Unit tests
   |
   v
Linux automation tests
   |
   v
Build artifact
   |
   v
Deploy
```

Do not expose credentials in command output.

---

# 85. Retry Logic

Retry only transient operations.

```python
import time

for attempt in range(3):
    try:
        run_operation()
        break
    except RuntimeError:
        if attempt == 2:
            raise
        time.sleep(2)
```

Do not retry:

```text
invalid configuration
permission denied
invalid input
authentication failure
```

unless there is a clear transient cause.

---

# 86. Exponential Backoff

```python
import time

delay = 1

for attempt in range(5):
    try:
        run_operation()
        break
    except RuntimeError:
        if attempt == 4:
            raise

        time.sleep(delay)
        delay *= 2
```

Always bound retries.

---

# 87. Retry + Idempotency

Retries are safe only when the operation is safely repeatable.

Examples:

```text
mkdir -> generally safe
health check -> safe
configuration validation -> safe
```

Potentially unsafe without careful design:

```text
non-idempotent data modification
destructive operations
external transactions
```

---

# 88. File Locking

Two scheduled jobs can overlap.

Unix file locking:

```python
import fcntl

with open(
    "/tmp/myjob.lock",
    "w",
) as lock_file:

    fcntl.flock(
        lock_file,
        fcntl.LOCK_EX,
    )

    run_job()
```

For multi-host coordination, a local file lock is not sufficient.

---

# 89. Concurrency

For many independent network checks:

```python
from concurrent.futures import ThreadPoolExecutor

def check_host(host):
    return host, check(host)

hosts = [
    "server1",
    "server2",
    "server3",
]

with ThreadPoolExecutor(
    max_workers=5
) as executor:
    for result in executor.map(
        check_host,
        hosts,
    ):
        print(result)
```

Use bounded concurrency.

---

# 90. Avoid Excessive Concurrency

Too many workers can cause:

```text
CPU pressure
network overload
connection exhaustion
rate limits
remote service overload
```

Concurrency should be based on the workload and target system.

---

# 91. Linux Automation and Signals

Long-running scripts may need graceful shutdown:

```python
import signal

def handle_signal(signum, frame):
    print("Shutdown requested")

signal.signal(
    signal.SIGTERM,
    handle_signal,
)
```

This matters for:

```text
systemd
containers
Kubernetes
CI cancellation
```

---

# 92. Graceful Shutdown

A long-running automation process should:

```text
receive SIGTERM
      |
      v
stop accepting new work
      |
      v
finish safe cleanup
      |
      v
exit
```

Do not ignore termination signals.

---

# 93. Incident Diagnostic Script

A practical Linux diagnostic collector can gather:

```text
hostname
uptime
CPU
memory
disk
inode usage
load average
failed services
top processes
listening ports
recent service logs
```

Output:

```text
JSON
```

This is useful during production incidents.

---

# 94. Failed Service Diagnostics

```python
import subprocess

result = subprocess.run(
    [
        "systemctl",
        "--failed",
        "--no-legend",
    ],
    capture_output=True,
    text=True,
)

failed = result.stdout.strip()

if failed:
    print("Failed services:")
    print(failed)
```

Then collect logs only for services that require investigation.

---

# 95. Kernel Diagnostics

Commands such as:

```bash
dmesg
journalctl -k
```

can reveal:

```text
OOM events
I/O errors
filesystem problems
driver issues
network problems
```

Access may be restricted by system policy.

---

# 96. OOM Investigation

If an application is unexpectedly killed, inspect:

```text
journal
kernel messages
memory usage
container limits
process memory
recent deployment
```

Look for:

```text
Out of memory
OOM killer
Killed process
```

Do not immediately increase memory without identifying the cause.

---

# 97. Disk Incident Investigation

First:

```bash
df -h
df -i
```

Then investigate:

```text
large directories
large files
logs
container storage
temporary data
deleted-open files
```

Python can automate collection, but should not blindly delete data.

---

# 98. File Descriptor Incident

If logs show:

```text
Too many open files
```

inspect:

```text
process FD count
RLIMIT_NOFILE
application connection behavior
system configuration
```

Python's `resource` and `psutil` can help.

---

# 99. Service Active but Unhealthy

Check:

```text
systemctl state
port
HTTP health endpoint
application logs
database connectivity
DNS
upstream dependencies
```

This is why a good health check has multiple layers.

---

# 100. Configuration Failure

If a configuration update breaks a service:

```text
collect logs
validate config
restore known-good config
reload/restart
verify
```

Keep previous known-good artifacts when possible.

---

# 101. Linux Distribution Differences

Ubuntu/Debian commonly use:

```text
apt
systemd
```

RHEL-family systems may use:

```text
dnf
systemd
```

But service names, paths, packages, security policies, and defaults can differ.

Write automation around capabilities, not assumptions.

---

# 102. SELinux

On SELinux-enabled systems, Unix permissions alone may not explain access failures.

Check:

```text
security context
policy
file labels
audit logs
```

Do not disable SELinux as a generic troubleshooting fix.

---

# 103. Least Privilege

A Python script that reads application logs may not need root.

A script changing:

```text
system services
network configuration
protected files
```

may require specific privileges.

Grant only what is required.

---

# 104. Security Checklist

```text
[ ] Validate all external input
[ ] Avoid shell=True when unnecessary
[ ] Never build rm -rf from untrusted input
[ ] Protect secrets
[ ] Use least privilege
[ ] Add timeouts
[ ] Bound retries
[ ] Use approved dependencies
[ ] Scan dependencies
[ ] Avoid chmod 777
[ ] Log actions without secrets
[ ] Support dry-run for destructive tasks
```

---

# 105. Testing Linux Automation

Use:

```text
unit tests
mocked subprocess calls
temporary directories
test VMs
containers
staging hosts
```

Avoid testing destructive code first against:

```text
/etc
/var
production databases
production hosts
```

---

# 106. Temporary Directory Testing

```python
from tempfile import TemporaryDirectory
from pathlib import Path

with TemporaryDirectory() as directory:
    path = Path(directory) / "test.txt"
    path.write_text("hello")
    assert path.read_text() == "hello"
```

This is safer than testing against real system directories.

---

# 107. Mock subprocess

```python
from unittest.mock import patch

@patch("subprocess.run")
def test_service_check(mock_run):
    mock_run.return_value.returncode = 0

    # test your service-check function
```

Mocking allows command logic to be tested without modifying the host.

---

# 108. Production Safety

Before deploying Linux automation, verify:

```text
Input validation
Permissions
Timeouts
Error handling
Exit codes
Logging
Idempotency
Dry-run
Verification
Rollback
Testing
```

---

# 109. Python vs Bash

Use Bash for:

```text
simple pipelines
short command sequences
shell-native tasks
```

Use Python for:

```text
complex logic
structured data
reusable automation
testing
API integrations
large scripts
```

A DevOps engineer should know both.

---

# 110. Python vs Ansible

Ansible is excellent for:

```text
configuration management
multi-host orchestration
desired-state automation
```

Python is excellent for:

```text
custom logic
specialized automation
API integration
custom diagnostics
```

They complement each other.

---

# 111. Python vs Terraform

Terraform manages:

```text
infrastructure desired state
```

Python can:

```text
validate
orchestrate
integrate
generate data
perform custom checks
```

Do not replace Terraform state management with arbitrary Python scripts.

---

# 112. Python vs Prometheus

Prometheus is designed for:

```text
metrics
time series
alerting
```

Python health scripts are useful for:

```text
custom diagnostics
prechecks
specialized validation
```

Use established observability systems rather than building a monitoring platform from scratch.

---

# 113. Production Preflight

Before deployment:

```text
Python precheck
   |
   +--> Python version
   +--> required commands
   +--> disk
   +--> memory
   +--> configuration
   +--> service state
   +--> port
   +--> network
   |
   v
PASS / FAIL
```

A failed precheck should stop the deployment.

---

# 114. Post-Deployment Verification

```text
deployment
   |
   v
service state
   |
   v
port
   |
   v
health endpoint
   |
   v
logs
   |
   v
PASS / FAIL
```

Never assume:

```text
deployment command returned 0
=
application is healthy
```

---

# 115. Daily DevOps Uses

Python is commonly used for:

```text
1. Disk cleanup
2. Log analysis
3. Service checks
4. Process diagnostics
5. CPU/memory checks
6. Configuration validation
7. Backup jobs
8. Deployment prechecks
9. Package validation
10. Network checks
11. CI/CD helpers
12. Incident collection
13. File synchronization
14. Scheduled maintenance
15. EC2 automation
16. Kubernetes diagnostics
17. Terraform orchestration
18. Security checks
19. Compliance checks
20. Fleet reporting
```

---

# 116. Daily Script — Disk Check

```python
#!/usr/bin/env python3

import shutil
import sys

threshold = 90

total, used, free = shutil.disk_usage("/")

usage = used / total * 100

print(f"Disk usage: {usage:.1f}%")

if usage >= threshold:
    print("CRITICAL: disk usage is high")
    sys.exit(2)

sys.exit(0)
```

---

# 117. Daily Script — Service Check

```python
#!/usr/bin/env python3

import subprocess
import sys

service = "nginx"

result = subprocess.run(
    [
        "systemctl",
        "is-active",
        "--quiet",
        service,
    ],
)

if result.returncode == 0:
    print(f"{service}: OK")
    sys.exit(0)

print(f"{service}: FAILED")
sys.exit(1)
```

---

# 118. Daily Script — CPU Check

```python
#!/usr/bin/env python3

import psutil
import sys

threshold = 90

usage = psutil.cpu_percent(
    interval=1
)

print(f"CPU usage: {usage:.1f}%")

if usage >= threshold:
    print("WARNING: high CPU")
    sys.exit(1)

sys.exit(0)
```

---

# 119. Daily Script — Memory Check

```python
#!/usr/bin/env python3

import psutil
import sys

threshold = 90

usage = psutil.virtual_memory().percent

print(
    f"Memory usage: {usage:.1f}%"
)

if usage >= threshold:
    print("WARNING: high memory")
    sys.exit(1)

sys.exit(0)
```

---

# 120. Combined Health Script

```python
#!/usr/bin/env python3

import json
import shutil
import socket

import psutil


CPU_LIMIT = 90
MEMORY_LIMIT = 90
DISK_LIMIT = 90


def collect():
    total, used, _ = shutil.disk_usage("/")

    return {
        "hostname": socket.gethostname(),
        "cpu_percent": psutil.cpu_percent(
            interval=1
        ),
        "memory_percent":
            psutil.virtual_memory().percent,
        "disk_percent":
            used / total * 100,
    }


def main():
    result = collect()

    result["healthy"] = (
        result["cpu_percent"] < CPU_LIMIT
        and result["memory_percent"] < MEMORY_LIMIT
        and result["disk_percent"] < DISK_LIMIT
    )

    print(
        json.dumps(
            result,
            indent=2,
        )
    )

    return 0 if result["healthy"] else 1


if __name__ == "__main__":
    raise SystemExit(main())
```

---

# 121. Production Automation Project Structure

```text
linux-automation/
├── .venv/
├── .gitignore
├── requirements.txt
├── README.md
├── src/
│   ├── __init__.py
│   ├── commands.py
│   ├── filesystem.py
│   ├── health.py
│   ├── services.py
│   └── main.py
└── tests/
    ├── test_health.py
    └── test_filesystem.py
```

As scripts grow, separate responsibilities instead of maintaining one huge file.

---

# 122. Reusable Command Module

```python
import subprocess


def run_command(command, timeout=30):
    result = subprocess.run(
        command,
        capture_output=True,
        text=True,
        check=True,
        timeout=timeout,
    )

    return result.stdout.strip()
```

Centralizing command execution makes behavior consistent.

---

# 123. Reusable Service Module

```python
import subprocess


def is_service_active(service):
    result = subprocess.run(
        [
            "systemctl",
            "is-active",
            "--quiet",
            service,
        ],
        check=False,
    )

    return result.returncode == 0
```

---

# 124. Reusable Filesystem Module

```python
from pathlib import Path


def ensure_directory(path):
    directory = Path(path)

    directory.mkdir(
        parents=True,
        exist_ok=True,
    )

    return directory
```

---

# 125. Reusable Health Module

```python
import shutil

import psutil


def collect_health():
    total, used, _ = shutil.disk_usage("/")

    return {
        "cpu": psutil.cpu_percent(
            interval=1
        ),
        "memory": psutil.virtual_memory().percent,
        "disk": used / total * 100,
    }
```

---

# 126. Performance

Avoid:

```text
full filesystem scans
one subprocess per trivial operation
unbounded concurrency
loading huge logs into memory
frequent expensive health checks
```

Prefer:

```text
native Python APIs
bounded concurrency
streaming files
batch operations
appropriate intervals
```

---

# 127. Native APIs First

A useful decision order:

```text
Python standard library
        |
        v
approved Python library
        |
        v
Linux command
```

Use a Linux command when it is the authoritative or simplest interface.

Do not parse human-readable command output when a stable API exists.

---

# 128. Common Mistakes

Avoid:

```text
sudo pip install
shell=True with untrusted input
rm -rf from arbitrary input
chmod 777
no timeout
infinite retries
no exit code
no verification
root by default
fragile CLI parsing
scanning /
hardcoded secrets
assuming Ubuntu == RHEL
```

---

# 129. Troubleshooting — Module Not Found

If a Python Linux script says:

```text
ModuleNotFoundError
```

check:

```bash
which python
python -m pip --version
python -m pip show psutil
```

Also:

```bash
python -c 'import sys; print(sys.executable)'
```

A common cause is using a different Python interpreter from the one where the dependency was installed.

---

# 130. Troubleshooting — Script Works Manually but Not Cron

Check:

```text
PATH
Python interpreter
working directory
environment variables
permissions
credentials
file paths
virtual environment
```

Use:

```cron
/opt/tool/.venv/bin/python /opt/tool/check.py
```

instead of relying on:

```cron
python check.py
```

---

# 131. Troubleshooting — Service Active but App Down

Investigate:

```text
systemctl state
listening port
health endpoint
application logs
database
DNS
upstream services
```

A running process does not prove the application is healthy.

---

# 132. Troubleshooting — Disk Full

Check:

```bash
df -h
df -i
```

Then:

```text
large directories
large files
logs
container storage
temporary files
deleted-open files
```

Do not blindly delete files.

---

# 133. Troubleshooting — High CPU

Collect:

```text
load average
top processes
process CPU
traffic
recent deployment
application logs
system logs
```

A CPU threshold should trigger investigation, not necessarily an automatic restart.

---

# 134. Troubleshooting — High Memory

Check:

```text
top memory processes
OOM events
application behavior
container limits
recent deployment
swap
memory leak
```

Increasing memory without understanding the cause can hide the problem.

---

# 135. Troubleshooting — Too Many Open Files

Check:

```text
RLIMIT_NOFILE
process FD count
system limits
connection pools
file descriptor leaks
```

Python:

```python
import resource

print(
    resource.getrlimit(
        resource.RLIMIT_NOFILE
    )
)
```

---

# 136. Troubleshooting — Permission Denied

Check:

```text
user
group
mode
ACL
SELinux/AppArmor
mount options
service account
```

Do not immediately use `sudo` or `chmod 777`.

---

# 137. Troubleshooting — Command Hangs

Add:

```python
timeout=30
```

Handle:

```python
subprocess.TimeoutExpired
```

Then investigate:

```text
DNS
network
service
interactive prompt
deadlock
remote endpoint
```

---

# 138. Troubleshooting — Package Manager Failure

Check:

```text
distribution
package manager
repository
network
permissions
disk space
package availability
```

Use the OS-supported package-management mechanism.

---

# 139. Troubleshooting — Ubuntu Works, RHEL Fails

Compare:

```text
package manager
service name
filesystem paths
security policy
Python version
system utilities
configuration locations
```

Use capability detection rather than assumptions.

---

# 140. Troubleshooting — Configuration Breaks Service

Safe response:

```text
collect logs
   |
   v
validate configuration
   |
   v
restore known-good configuration
   |
   v
reload/restart
   |
   v
health check
```

Keep a rollback path.

---

# 141. Troubleshooting — Cleanup Deletes Wrong Files

Prevention:

```text
absolute allowed root
resolve paths
validate file type
validate age
dry-run
tests
logging
```

Do not accept arbitrary deletion paths.

---

# 142. Troubleshooting — Two Jobs Overlap

Use:

```text
file lock
distributed lock
scheduler concurrency control
```

For one Linux host:

```python
fcntl.flock()
```

may be appropriate.

For multiple hosts, use an appropriate distributed coordination mechanism.

---

# 143. Production Incident Scenario — Service Failure

Situation:

```text
nginx is unavailable
```

Python diagnostic sequence:

```text
systemctl is-active nginx
        |
        v
journalctl -u nginx
        |
        v
ss -lntp
        |
        v
HTTP health check
```

Only restart after determining that restart is an approved remediation.

---

# 144. Production Incident Scenario — Disk at 100%

Automation should collect:

```text
df -h
df -i
largest directories
largest files
log growth
container storage
```

Then report:

```json
{
  "filesystem": "/",
  "usage_percent": 99.2,
  "inode_issue": false
}
```

Do not automatically delete production data without a defined retention policy.

---

# 145. Production Incident Scenario — OOM

Collect:

```text
memory usage
top processes
kernel OOM logs
service logs
container limits
recent deployments
```

The goal is to identify whether the cause is:

```text
traffic
leak
configuration
limit
deployment
legitimate workload
```

---

# 146. Production Incident Scenario — Network Failure

Run:

```text
DNS
route
TCP
TLS
HTTP
application dependency
```

Example Python TCP check:

```python
socket.create_connection(
    ("example.com", 443),
    timeout=5,
)
```

Do not confuse a TCP success with application success.

---

# 147. Production Incident Scenario — Deployment Precheck

A precheck can verify:

```text
required commands
Python version
disk
memory
configuration
service account
network
target port
```

Then:

```text
PASS -> continue
FAIL -> stop
```

---

# 148. Production Incident Scenario — Post-Deployment Failure

After deployment:

```text
service active?
port listening?
health endpoint?
logs clean?
dependencies reachable?
```

If unhealthy:

```text
stop further rollout
collect diagnostics
rollback/escalate according to deployment policy
```

---

# 149. Production Incident Scenario — Python Dependency Vulnerability

Process:

```text
identify CVE
   |
   v
find affected dependency
   |
   v
find patched version
   |
   v
update dependency definition
   |
   v
test
   |
   v
SCA scan
   |
   v
build
   |
   v
deploy
```

Do not suppress the vulnerability without understanding the risk.

---

# 150. Production Incident Scenario — Wrong Interpreter

Symptom:

```text
package installed
but import fails
```

Check:

```bash
which python
python -m pip --version
python -m pip show package
python -c "import sys; print(sys.executable)"
```

Fix the environment rather than installing the package globally.

---

# 151. Production Incident Scenario — Stale CI Environment

A clean CI environment should recreate:

```text
Python version
dependencies
system packages
configuration
```

Do not rely on a developer's `.venv`.

---

# 152. Production Incident Scenario — Secret Appears in Logs

Treat it as exposed.

Actions should include:

```text
stop further exposure
rotate credential according to policy
restrict log access
remove/redact future output
investigate scope
```

Do not simply delete the local Python environment.

---

# 153. Production Incident Scenario — Automation Runs as Root

First identify why root is required.

Then reduce privileges where possible:

```text
dedicated service user
specific group
sudo rule
Linux capability
approved privileged helper
```

Avoid broad root access.

---

# 154. Production Incident Scenario — Command Injection

If untrusted input reaches:

```python
shell=True
```

or an unsafe shell string, treat it seriously.

Prevent with:

```text
argument lists
input validation
allowlists
no shell where unnecessary
least privilege
```

---

# 155. Production Incident Scenario — Automation Hangs

Typical causes:

```text
network timeout
interactive command
deadlock
remote service
DNS
unbounded retry
```

Add:

```text
timeouts
bounded retries
clear logging
```

---

# 156. Production Incident Scenario — Cleanup Job Overlaps

Use a lock:

```text
job starts
   |
   v
acquire lock
   |
   +--> already locked -> exit
   |
   v
cleanup
   |
   v
release
```

This prevents concurrent destructive work on the same host.

---

# 157. Production Incident Scenario — Wrong Linux Distribution

Do not blindly execute:

```text
apt
```

or:

```text
dnf
```

Detect supported capabilities and fail clearly if unsupported.

---

# 158. When Not to Write Python

Use established tools when they already solve the problem:

```text
Terraform
Ansible
systemd
cron
Prometheus
AWS Systems Manager
Kubernetes
Jenkins/GitLab CI
```

Python is strongest as a custom automation and integration layer.

---

# 159. Enterprise Linux Automation Architecture

```text
Git
 |
 v
CI/CD
 |
 +--> Python tests
 +--> dependency/SCA scan
 |
 v
Artifact
 |
 v
Approved repository
 |
 v
Deployment
 |
 v
Linux / EC2 / EKS
 |
 +--> logs
 +--> metrics
 +--> health checks
 |
 v
Prometheus / Grafana / ELK
```

Python should fit into this architecture rather than becoming an isolated collection of scripts.

---

# 160. End-to-End DevOps Example

A Linux deployment helper:

```text
Git commit
    |
    v
CI
    |
    v
Python preflight
    |
    +--> disk
    +--> memory
    +--> network
    +--> service
    +--> configuration
    |
    v
Deploy
    |
    v
Post-deployment Python checks
    |
    +--> service
    +--> port
    +--> HTTP
    +--> logs
    |
    v
PASS / FAIL
```

This is a realistic way Python supports DevOps without replacing the underlying platform tools.

---

# 161. Final Production Checklist

```text
[ ] Inputs validated
[ ] Environment validated
[ ] Required commands checked
[ ] Permissions understood
[ ] Least privilege used
[ ] Commands use argument lists
[ ] shell=True avoided when unnecessary
[ ] Timeouts configured
[ ] Retries bounded
[ ] Retry operations are safe
[ ] Filesystem paths validated
[ ] Destructive operations support dry-run
[ ] Idempotency considered
[ ] Configuration backed up where appropriate
[ ] Post-action verification exists
[ ] Errors logged
[ ] Secrets never logged
[ ] Correct exit codes returned
[ ] Tests exist
[ ] Dependencies scanned
[ ] CI uses a clean environment
[ ] Production rollback/recovery considered
```

---

# 162. Interview Questions

## What is `subprocess` used for?

**Answer:**

> I use `subprocess` when Python needs to interact with Linux commands or approved external tools. I normally pass arguments as a list, use `check=True` when failure should stop execution, capture output when required, and configure timeouts for operations that can hang.

## Why avoid `shell=True`?

> It introduces shell parsing and can create command-injection risk when input is not fully trusted. I prefer argument lists unless shell functionality is genuinely required.

## How do you make a Linux script idempotent?

> I check the current state before changing it and make operations safe to repeat. For example, I use `mkdir(..., exist_ok=True)`, check whether users or services already exist, and verify the desired state after the operation.

## How do you check disk usage?

> For a filesystem I can use `shutil.disk_usage()`. For Linux-specific diagnostics I may use `df -h` and `df -i`, while avoiding fragile parsing when a structured API is available.

## How do you check CPU and memory?

> I commonly use `psutil` for CPU, memory, process and network information. For lower-level Linux diagnostics I can inspect `/proc`.

## How do you manage systemd services?

> I can use `systemctl` through `subprocess`, but I verify both the service state and application health. A service being `active` does not necessarily mean the application is healthy.

## How do you troubleshoot a script that works manually but fails in cron?

> I check the Python interpreter, PATH, working directory, environment variables, permissions, credentials, and file paths. I normally use an absolute interpreter such as `/opt/tool/.venv/bin/python`.

## How do you handle destructive automation?

> I validate the target, constrain the allowed scope, provide dry-run support, log intended changes, make the operation idempotent where possible, and verify the result. For production I also use appropriate approvals and rollback mechanisms.

## How do you secure Linux automation?

> I use least privilege, validate inputs, avoid unsafe shell execution, protect secrets, use timeouts, bound retries, scan dependencies, avoid excessive permissions, and never log credentials.

## When would you choose Python over Bash?

> For short command sequences I may use Bash. When automation requires structured data, complex logic, reusable modules, testing, APIs, or robust exception handling, I prefer Python.

---

# 163. Final Mental Model

Linux automation with Python is not about replacing Linux fundamentals.

It is about combining them:

```text
Linux knowledge
      +
Python
      +
subprocess
      +
pathlib
      +
shutil
      +
psutil
      +
systemd
      +
networking
      +
logging
      +
error handling
      +
security
      +
idempotency
      +
testing
      =
Production DevOps automation
```

The most important production pattern is:

```text
VALIDATE
   ↓
PREFLIGHT
   ↓
COLLECT STATE
   ↓
PLAN
   ↓
EXECUTE SAFELY
   ↓
VERIFY
   ↓
LOG
   ↓
EXIT CORRECTLY
```

> **Python is the automation layer; Linux fundamentals remain the foundation.**
