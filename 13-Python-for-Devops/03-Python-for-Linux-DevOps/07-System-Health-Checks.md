# 07-System-Health-Checks

## Python for Linux DevOps

System health checks are one of the most useful Python automation patterns for DevOps engineers.

A health-check script should answer:

```text
Is the host reachable?
Is the operating system healthy?
Is CPU under pressure?
Is memory sufficient?
Is disk capacity safe?
Are critical services running?
Are required ports listening?
Is networking working?
Is DNS working?
Is the application healthy?
Are there failed systemd units?
Are there recent critical errors?
```

The goal is not to collect every possible metric.

The goal is:

> **Detect meaningful failures early, return actionable results, and integrate those results into CI/CD, monitoring, incident response, or scheduled automation.**

---

# 1. What Is a System Health Check?

A system health check is a repeatable validation of important system conditions.

Example:

```text
Health Check
     |
     +-- Host
     +-- CPU
     +-- Memory
     +-- Disk
     +-- Processes
     +-- Services
     +-- Network
     +-- DNS
     +-- Application
```

A health check should produce a clear result:

```text
PASS
WARN
FAIL
UNKNOWN
```

---

# 2. Health Check vs Monitoring

They are related but different.

Health check:

```text
Is the system healthy right now?
```

Monitoring:

```text
How has the system behaved over time?
```

Example:

```text
Python health check
    ↓
CPU = 55%
Disk = 72%
Nginx = active
Health endpoint = 200
    ↓
PASS
```

Prometheus can then monitor the same system continuously over time.

---

# 3. Health Check vs Readiness Check

A host can be healthy but not ready.

Example:

```text
Server healthy
Application starting
Database connection not ready
```

Therefore:

```text
Liveness
=
Is it alive?

Readiness
=
Can it serve traffic?

Health
=
Is the overall system operating correctly?
```

---

# 4. Health Check Categories

A production health checker can contain:

```text
1. Host availability
2. CPU
3. Load
4. Memory
5. Swap
6. Disk
7. Inodes
8. Processes
9. Services
10. Ports
11. Network interfaces
12. Routes
13. DNS
14. HTTP
15. TLS
16. Time synchronization
17. Kernel
18. System errors
19. Application state
20. Security-related checks
```

Only include checks relevant to the system.

---

# 5. Health Check Output

A useful result:

```json
{
  "name": "disk_root",
  "status": "PASS",
  "value": 61,
  "threshold": 80,
  "message": "Root filesystem usage is healthy"
}
```

Structured results are easier to process than plain text.

---

# 6. Status Model

Recommended:

```text
PASS
WARN
FAIL
UNKNOWN
```

Example:

```text
CPU = 55%   → PASS
Disk = 84%  → WARN
Service down → FAIL
Metric unavailable → UNKNOWN
```

---

# 7. Why UNKNOWN Matters

Do not automatically treat:

```text
metric unavailable
```

as:

```text
healthy
```

For example:

```text
cannot read /proc/meminfo
```

should not become:

```text
MEMORY = PASS
```

Unknown information must remain visible.

---

# 8. Health Check Exit Codes

Typical CLI contract:

```text
0 = healthy
1 = unhealthy
2 = invalid usage/configuration
3 = unable to perform checks
```

The exact convention can be defined by the project.

The important point is that CI/CD can reliably interpret the result.

---

# 9. Python Libraries

Useful standard-library modules:

```text
os
platform
pathlib
subprocess
socket
shutil
resource
time
datetime
json
logging
argparse
concurrent.futures
```

Optional external libraries:

```text
psutil
requests
PyYAML
```

---

# 10. `psutil`

`psutil` is widely useful for system information.

Install:

```bash
pip install psutil
```

It can provide:

```text
CPU
memory
swap
disk
network
processes
users
boot time
```

---

# 11. CPU Usage

Using psutil:

```python
import psutil

usage = psutil.cpu_percent(
    interval=1
)

print(usage)
```

The interval gives psutil time to calculate usage.

---

# 12. CPU Count

```python
import psutil

print(
    psutil.cpu_count(
        logical=True
    )
)
```

Physical CPU count:

```python
psutil.cpu_count(
    logical=False
)
```

---

# 13. CPU Per Core

```python
psutil.cpu_percent(
    interval=1,
    percpu=True,
)
```

Example:

```text
core 0 = 20%
core 1 = 91%
core 2 = 18%
core 3 = 22%
```

Average CPU can hide a single saturated core.

---

# 14. CPU Load Average

On Linux:

```python
import os

print(
    os.getloadavg()
)
```

Returns:

```text
1 minute
5 minutes
15 minutes
```

Load average is not the same thing as CPU percentage.

---

# 15. CPU Percentage vs Load

CPU percentage:

```text
how much CPU is being used
```

Load average:

```text
how many tasks are runnable
or waiting for certain resources
```

Interpret load relative to:

```text
number of CPU cores
```

---

# 16. Load Example

A load of:

```text
4.0
```

means something very different on:

```text
2-core server
```

versus:

```text
16-core server
```

Always consider CPU count.

---

# 17. Load Health Check

Concept:

```python
load1 = os.getloadavg()[0]
cores = os.cpu_count() or 1

ratio = load1 / cores
```

Then define organization-specific thresholds.

---

# 18. CPU Health Threshold

Example:

```text
ratio < 1.0  → PASS
1.0–2.0      → WARN
> 2.0        → FAIL
```

These are examples, not universal production thresholds.

CPU utilization and load should be evaluated together.

---

# 19. Memory Usage

```python
import psutil

memory = psutil.virtual_memory()

print(memory.total)
print(memory.available)
print(memory.percent)
```

Important distinction:

```text
free
vs
available
```

Linux uses memory for caching, so free memory alone can be misleading.

---

# 20. Available Memory

For Linux health checks, `available` is often more useful than `free`.

Example:

```python
if memory.available < 512 * 1024**2:
    print("WARN")
```

Thresholds should be based on workload requirements.

---

# 21. Memory Pressure

High memory usage can lead to:

```text
swap
OOM kills
latency
process failures
Kubernetes memory pressure
```

Do not wait for the application to crash before detecting sustained memory pressure.

---

# 22. Swap

```python
swap = psutil.swap_memory()

print(swap.percent)
```

Swap usage is not automatically a failure.

Investigate:

```text
swap utilization
swap activity
memory pressure
application latency
```

---

# 23. Swap Health

A simple check:

```text
swap = 0%      → informational
swap > 0%      → investigate
high + active  → possible problem
```

Do not blindly alert on any swap usage.

---

# 24. Disk Usage

```python
import shutil

usage = shutil.disk_usage("/")

used_percent = (
    usage.used / usage.total
) * 100

print(used_percent)
```

---

# 25. Disk Thresholds

Example:

```text
< 80%  PASS
80–90% WARN
> 90%  FAIL
```

But critical partitions may need lower thresholds.

For example:

```text
/var
/var/log
/opt
```

can have different risk profiles.

---

# 26. Check Multiple Filesystems

```python
paths = [
    "/",
    "/var",
    "/tmp",
    "/opt",
]
```

Not every path is a separate filesystem.

Use:

```bash
df -h
```

to understand the actual mount layout.

---

# 27. Disk Inodes

A filesystem can have free disk space but no free inodes.

Linux:

```bash
df -i
```

Python can call the system or use filesystem statistics.

---

# 28. Inode Exhaustion

Symptoms:

```text
cannot create files
temporary files fail
application errors
package installation failures
```

Even when:

```text
df -h
```

shows plenty of free space.

Always consider inode usage for file-heavy systems.

---

# 29. Python Inode Check

Using `os.statvfs`:

```python
import os

stats = os.statvfs("/")

total = (
    stats.f_files
)

free = (
    stats.f_ffree
)

used = total - free

percent = (
    used / total * 100
    if total
    else 0
)

print(percent)
```

---

# 30. Disk Health Result

A good result includes:

```text
mount
total
used
available
percent
inode percent
status
```

Example:

```json
{
  "mount": "/var",
  "usage_percent": 87,
  "inode_percent": 42,
  "status": "WARN"
}
```

---

# 31. Disk Growth Detection

A single snapshot tells you:

```text
current state
```

Monitoring tells you:

```text
growth rate
```

Example:

```text
Monday = 70%
Tuesday = 75%
Wednesday = 82%
```

The growth trend may be more important than the current value.

---

# 32. Log File Growth

Common causes:

```text
application errors
debug logging
log rotation failure
disk shipper failure
```

Health checks can inspect the filesystem but should not blindly delete logs.

---

# 33. Process Health

Using psutil:

```python
import psutil

for process in psutil.process_iter(
    ["pid", "name", "status"]
):
    print(
        process.info
    )
```

This can help with diagnostics.

---

# 34. Process Existence Check

```python
def process_exists(name):
    for process in psutil.process_iter(
        ["name"]
    ):
        if process.info["name"] == name:
            return True

    return False
```

For systemd services, prefer checking systemd state.

---

# 35. Systemd Service Check

```bash
systemctl is-active nginx
```

Python:

```python
import subprocess

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

healthy = (
    result.returncode == 0
)
```

---

# 36. Service Enabled vs Active

These are different:

```text
enabled
=
starts automatically

active
=
running now
```

A service can be:

```text
enabled + inactive
```

or:

```text
disabled + active
```

depending on the setup.

---

# 37. Failed Services

Linux:

```bash
systemctl --failed
```

Python can capture this for diagnostic reporting.

A failed unit does not always mean the application is down, so interpret results in context.

---

# 38. Service Health Function

```python
def service_active(name):
    result = subprocess.run(
        [
            "systemctl",
            "is-active",
            name,
        ],
        capture_output=True,
        text=True,
        check=False,
    )

    return {
        "service": name,
        "active": (
            result.returncode == 0
        ),
        "output":
            result.stdout.strip(),
    }
```

Use an allowlist for service names if input is external.

---

# 39. Port Health

Check listening ports:

```bash
ss -lnt
```

Python socket test:

```python
import socket

sock = socket.create_connection(
    ("127.0.0.1", 8080),
    timeout=3,
)

sock.close()
```

---

# 40. Port Listening vs Application Healthy

A port being open does not guarantee:

```text
application healthy
```

The process may:

```text
accept TCP
but return errors
```

Use an application-level health endpoint when available.

---

# 41. HTTP Health Check

```python
import urllib.request

response = urllib.request.urlopen(
    "http://127.0.0.1:8080/health",
    timeout=5,
)

print(response.status)
```

Close the response properly in production code.

---

# 42. HTTP Status

Typical interpretation:

```text
2xx → success
3xx → depends on expected behavior
4xx → client/request problem
5xx → server/application problem
```

Do not assume every 3xx is unhealthy.

---

# 43. HTTP Response Time

Health checks should consider latency.

Example:

```python
import time
import urllib.request

start = time.monotonic()

with urllib.request.urlopen(
    "http://127.0.0.1:8080/health",
    timeout=5,
) as response:
    status = response.status

duration = (
    time.monotonic() - start
)

print(status, duration)
```

---

# 44. Health Endpoint Design

A useful endpoint may expose:

```text
/liveness
/readiness
/health
```

Keep the endpoint lightweight.

Avoid making health checks perform expensive database queries every second.

---

# 45. Deep Health Check

A deeper endpoint may validate:

```text
database
cache
message broker
external dependency
```

Use it carefully.

If the endpoint depends on every external system, one dependency failure can make the whole application appear unhealthy.

---

# 46. Shallow vs Deep Health

Shallow:

```text
process is alive
```

Deep:

```text
application can perform required operations
```

Use both at the correct layers.

---

# 47. DNS Health

Python:

```python
import socket

ip = socket.gethostbyname(
    "example.com"
)

print(ip)
```

This tests basic resolver functionality.

---

# 48. DNS Resolution Failure

Possible causes:

```text
resolver unavailable
DNS configuration
network issue
domain problem
search domain
split-horizon DNS
```

Do not automatically assume the application is broken.

---

# 49. DNS and `/etc/resolv.conf`

Linux commonly uses:

```text
/etc/resolv.conf
```

But modern distributions may use:

```text
systemd-resolved
NetworkManager
other resolver managers
```

Inspect the actual resolver architecture.

---

# 50. DNS Health Check

```python
def dns_check(hostname):
    try:
        ip = socket.gethostbyname(
            hostname
        )

        return {
            "status": "PASS",
            "ip": ip,
        }

    except socket.gaierror as exc:
        return {
            "status": "FAIL",
            "error": str(exc),
        }
```

---

# 51. TCP Connectivity

```python
import socket

def tcp_check(
    host,
    port,
    timeout=3,
):
    try:
        with socket.create_connection(
            (host, port),
            timeout=timeout,
        ):
            return True

    except OSError:
        return False
```

---

# 52. TCP vs ICMP

A successful ping:

```text
ICMP works
```

does not prove:

```text
TCP 443 works
```

A TCP test is more relevant for application dependencies.

---

# 53. Ping Health

Python can call:

```bash
ping
```

but ping may be blocked.

Therefore:

```text
ping failure
```

does not necessarily mean:

```text
host unavailable
```

Use the protocol relevant to the actual dependency.

---

# 54. Network Interface Health

Linux:

```bash
ip -br addr
```

or:

```bash
ip link
```

Python can inspect interfaces using psutil:

```python
psutil.net_if_addrs()
```

---

# 55. Interface State

Important states:

```text
UP
DOWN
UNKNOWN
```

A down interface may be normal if it is unused.

Only check expected interfaces.

---

# 56. Network Errors

psutil can expose network counters:

```python
stats = psutil.net_io_counters(
    pernic=True
)

print(stats)
```

Useful fields include:

```text
bytes_sent
bytes_recv
packets_sent
packets_recv
errin
errout
dropin
dropout
```

---

# 57. Network Drops

Network drops can indicate:

```text
congestion
driver problems
interface issues
virtual network problems
```

A single counter snapshot is less useful than a trend.

---

# 58. Network Error Rate

For monitoring, calculate:

```text
new errors / time interval
```

rather than only checking the absolute counter.

Counters normally increase over time.

---

# 59. Default Route

Linux:

```bash
ip route
```

A host without a required default route may lose external connectivity.

But private systems may intentionally have no default route.

Always compare against expected architecture.

---

# 60. Route Health

For a dependency:

```text
database = 10.0.20.10
```

check:

```bash
ip route get 10.0.20.10
```

This is more useful than checking only the default route.

---

# 61. TLS Health

For HTTPS dependencies, health can include:

```text
DNS
TCP 443
TLS handshake
certificate validity
HTTP status
latency
```

---

# 62. Python TLS Check

```python
import socket
import ssl

context = ssl.create_default_context()

with socket.create_connection(
    ("example.com", 443),
    timeout=5,
) as sock:

    with context.wrap_socket(
        sock,
        server_hostname="example.com",
    ) as tls:

        print(
            tls.version()
        )
```

The default SSL context performs certificate validation.

---

# 63. Do Not Disable TLS Verification

Avoid:

```python
ssl._create_unverified_context()
```

for production health checks.

A TLS check that ignores certificate validation can provide false confidence.

---

# 64. Certificate Expiry

A health system can warn before certificate expiration.

Example policy:

```text
> 30 days  PASS
15–30 days WARN
< 15 days  CRITICAL
```

The actual policy should match organizational renewal processes.

---

# 65. Time Synchronization

Time matters for:

```text
TLS
Kerberos
JWT
logs
distributed systems
certificates
```

Check:

```bash
timedatectl
```

or the organization's approved time-sync service.

---

# 66. NTP Health

A healthy server should normally have:

```text
time synchronization active
reasonable clock offset
```

Large clock drift can create difficult-to-diagnose authentication and distributed-system failures.

---

# 67. System Uptime

```python
import time

uptime = (
    time.time()
    - psutil.boot_time()
)

print(uptime)
```

Long uptime is not automatically unhealthy.

Use uptime as context, not as a failure condition.

---

# 68. Boot Time

```python
from datetime import datetime

boot = datetime.fromtimestamp(
    psutil.boot_time()
)

print(boot)
```

Useful during incident timelines.

---

# 69. Kernel Version

```python
import platform

print(
    platform.release()
)
```

A health report can include kernel version for troubleshooting.

---

# 70. OS Information

```python
print(
    platform.platform()
)
```

For Linux distribution details, inspect:

```text
/etc/os-release
```

---

# 71. Failed Boot Units

```bash
systemctl --failed
```

This is useful during startup diagnostics.

Not every failed unit is critical.

Classify failures based on host role.

---

# 72. Recent Critical Logs

Linux:

```bash
journalctl \
    -p err \
    -n 100 \
    --no-pager
```

A health checker can collect a small recent sample.

Do not treat every historical error as a current outage.

---

# 73. Kernel Errors

Useful:

```bash
dmesg --level=err,warn
```

Access may be restricted depending on system configuration.

On systemd systems, journal logs may provide better access.

---

# 74. OOM Detection

Linux logs may show:

```text
Out of memory
Killed process
oom-kill
```

A health checker can search recent logs for OOM events.

Do not use a broad grep as the only source of truth.

---

# 75. OOM Health Check

Combine:

```text
current memory
swap
recent OOM events
process restarts
application health
```

This gives a much stronger signal than memory percentage alone.

---

# 76. CPU Health Should Be Multi-Signal

Use:

```text
CPU utilization
load average
run queue
process CPU
application latency
```

High CPU may be expected during a controlled batch job.

---

# 77. Memory Health Should Be Multi-Signal

Use:

```text
available memory
swap activity
OOM events
process memory
application latency
```

Do not alert on one metric in isolation.

---

# 78. Disk Health Should Be Multi-Signal

Use:

```text
capacity
inode usage
growth rate
filesystem errors
application write failures
```

---

# 79. Service Health Should Be Multi-Signal

Use:

```text
systemd state
process
port
health endpoint
logs
application metrics
```

---

# 80. Dependency Health

An application can be healthy internally but unable to reach:

```text
database
Redis
RabbitMQ
external API
DNS
object storage
```

Check critical dependencies separately.

---

# 81. Dependency Check

Example:

```python
dependencies = [
    ("database.internal", 5432),
    ("redis.internal", 6379),
    ("rabbitmq.internal", 5672),
]
```

Use TCP checks where appropriate.

Do not perform expensive authentication operations just to test reachability unless required.

---

# 82. Database Health

A TCP connection to port 5432 proves:

```text
network path + TCP listener
```

It does not prove:

```text
database accepts queries
```

Use an appropriate lightweight database health check when required.

---

# 83. Redis Health

Similarly:

```text
TCP 6379
```

does not prove Redis is functioning correctly.

A lightweight protocol-level check may provide stronger validation.

---

# 84. RabbitMQ Health

Check:

```text
port
management/API if appropriate
node health
queue state
```

Avoid making the health check itself create operational side effects.

---

# 85. External API Health

Use:

```text
DNS
TCP
TLS
HTTP
expected status
latency
```

Do not repeatedly call expensive APIs from every server's health check.

---

# 86. Health Check Frequency

Examples:

```text
local script        → on demand
CI/CD preflight     → per deployment
scheduled check     → every few minutes
monitoring probe    → application-specific
```

Choose frequency based on failure detection requirements.

---

# 87. Health Check Timeouts

Every external check should have a timeout.

Example:

```python
timeout = 5
```

Without timeouts, one dependency can block the entire health-check program.

---

# 88. Health Check Retries

Use retries carefully.

A transient network packet loss should not immediately declare a major outage.

But excessive retries can:

```text
increase load
delay alerts
hide real failures
```

---

# 89. Retry with Backoff

Example:

```text
attempt 1 → immediate
attempt 2 → 1 sec
attempt 3 → 2 sec
attempt 4 → 4 sec
```

Add jitter for distributed systems when many clients retry simultaneously.

---

# 90. Health Check Parallelism

Independent checks can run concurrently:

```text
DNS
TCP
disk
memory
service
```

This reduces total execution time.

Use bounded concurrency.

---

# 91. ThreadPoolExecutor

```python
from concurrent.futures import (
    ThreadPoolExecutor,
)

with ThreadPoolExecutor(
    max_workers=5
) as executor:

    results = list(
        executor.map(
            run_check,
            checks,
        )
    )
```

Health checks are often I/O-bound.

---

# 92. Async Health Checks

For very large numbers of network endpoints, asynchronous libraries can be useful.

Examples:

```text
asyncio
aiohttp
AsyncSSH
```

Do not introduce async complexity for a simple host-level script without a clear need.

---

# 93. Health Check Architecture

```text
CLI
 |
 v
Configuration
 |
 v
Check Registry
 |
 +-- CPU
 +-- Memory
 +-- Disk
 +-- Service
 +-- Network
 +-- DNS
 +-- HTTP
 |
 v
Result Aggregator
 |
 +-- JSON
 +-- Console
 +-- Exit Code
 |
 v
CI / Monitoring
```

---

# 94. Check Registry

Instead of hardcoding everything in `main()`:

```python
checks = [
    check_cpu,
    check_memory,
    check_disk,
    check_services,
    check_network,
]
```

This makes the tool easier to extend.

---

# 95. Standard Check Interface

A check can return:

```python
{
    "name": "memory",
    "status": "PASS",
    "value": 62,
    "threshold": 85,
    "message": "Memory healthy",
}
```

All checks should follow a consistent schema.

---

# 96. Health Result Dataclass

```python
from dataclasses import dataclass

@dataclass
class CheckResult:
    name: str
    status: str
    message: str
    value: object = None
    threshold: object = None
```

This makes result handling cleaner.

---

# 97. Health Check Class

```python
class HealthChecker:

    def __init__(self):
        self.results = []

    def add(self, result):
        self.results.append(result)

    def healthy(self):
        return all(
            item.status != "FAIL"
            for item in self.results
        )
```

Extend the model for WARN/UNKNOWN policies.

---

# 98. Configuration-Driven Checks

Example:

```yaml
thresholds:
  cpu_warn: 80
  cpu_fail: 95
  memory_warn: 80
  memory_fail: 90
  disk_warn: 80
  disk_fail: 90

services:
  - nginx
  - myapp
```

This avoids hardcoding every environment-specific value.

---

# 99. Configuration Validation

Before running checks:

```text
threshold exists?
threshold numeric?
warn < fail?
service names valid?
required paths exist?
```

Fail early on invalid configuration.

---

# 100. Warn vs Fail

Example:

```text
disk 82%
```

could be:

```text
WARN
```

while:

```text
disk 96%
```

could be:

```text
FAIL
```

The policy should be explicit.

---

# 101. Aggregating Status

Example:

```text
Any FAIL       → overall FAIL
No FAIL + WARN → overall WARN
All PASS       → overall PASS
Only UNKNOWN   → overall UNKNOWN
```

This provides predictable behavior.

---

# 102. Critical vs Non-Critical Checks

Not every check should have equal weight.

Example:

```yaml
checks:
  ssh:
    critical: true

  disk:
    critical: true

  swap:
    critical: false

  uptime:
    critical: false
```

This prevents informational signals from unnecessarily blocking deployments.

---

# 103. Check Severity

Useful levels:

```text
INFO
WARN
CRITICAL
```

A check result can combine:

```text
status
severity
```

depending on the design.

---

# 104. Production Health Policy

Example:

```text
Critical service down → FAIL
Root disk > 90%       → FAIL
Memory > 90%          → WARN
Swap active           → INFO/WARN
Uptime 200 days       → INFO
One unused interface down → PASS
```

The correct policy depends on the host.

---

# 105. Role-Based Health Checks

Different servers require different checks.

```text
web server
 ↓
nginx
80/443
HTTP health

database
 ↓
database service
5432
storage
replication

worker
 ↓
worker service
queue connectivity
```

Do not use one identical checklist for every machine.

---

# 106. Host Role Configuration

Example:

```yaml
role: web

checks:
  services:
    - nginx

  ports:
    - 80
    - 443

  endpoints:
    - https://localhost/health
```

---

# 107. Environment-Based Thresholds

Development:

```text
disk fail = 95%
```

Production:

```text
disk fail = 90%
```

Thresholds can differ because the operational risk differs.

---

# 108. Health Checks in CI/CD

Before deployment:

```text
target reachable?
disk sufficient?
service exists?
artifact can be installed?
dependency reachable?
```

After deployment:

```text
service active?
port open?
health endpoint?
version correct?
```

---

# 109. Jenkins Health Gate

Concept:

```text
Preflight
   |
   v
PASS? ---- NO ---> STOP
 |
YES
 |
Deploy
 |
Postflight
 |
PASS? ---- NO ---> Rollback
 |
YES
 |
SUCCESS
```

---

# 110. GitHub Actions Health Gate

The Python script should return:

```text
0 → continue
non-zero → fail job
```

The workflow can then stop deployment automatically.

---

# 111. Kubernetes Health Checks

Kubernetes already supports:

```text
livenessProbe
readinessProbe
startupProbe
```

Do not duplicate these blindly with SSH-based host checks.

Python can validate:

```text
cluster/node prerequisites
deployment status
external dependencies
```

while Kubernetes handles container lifecycle health.

---

# 112. EKS Node Health

For EKS, combine:

```text
kubectl get nodes
kubectl describe node
node metrics
Prometheus
application health
```

with host-level diagnostics only when required.

---

# 113. Node Conditions

Important Kubernetes node conditions:

```text
Ready
MemoryPressure
DiskPressure
PIDPressure
NetworkUnavailable
```

A Python tool can query the Kubernetes API or invoke controlled `kubectl` commands.

---

# 114. Linux Health vs Kubernetes Health

Linux host:

```text
CPU
memory
disk
systemd
network
```

Kubernetes:

```text
node conditions
pods
deployments
services
probes
resource requests/limits
```

Application:

```text
HTTP
database
queue
business health
```

A mature health system understands all three layers.

---

# 115. Layered Health Model

```text
Layer 1: Infrastructure
        |
Layer 2: Operating System
        |
Layer 3: Runtime
        |
Layer 4: Application
        |
Layer 5: Dependencies
        |
Layer 6: User-facing path
```

A failure at a lower layer often affects higher layers.

---

# 116. User-Facing Health

The strongest test is sometimes:

```text
client
 ↓
DNS
 ↓
Load Balancer
 ↓
Ingress
 ↓
Application
 ↓
Database
```

A localhost health check does not validate the complete path.

---

# 117. Synthetic Health Checks

A synthetic check behaves like a user:

```text
resolve DNS
connect HTTPS
send request
validate response
measure latency
```

This is useful for external availability.

---

# 118. Synthetic vs Internal

Internal:

```text
localhost:8080
```

Synthetic:

```text
https://app.example.com
```

Use both for different failure modes.

---

# 119. Health Check Security

Health endpoints should not expose:

```text
passwords
tokens
internal topology
database credentials
stack traces
```

Keep output minimal.

---

# 120. Health Endpoint Authentication

Some deep health endpoints may require authentication.

Avoid placing credentials directly in scripts.

Use:

```text
secret manager
environment injection
short-lived token
```

when authentication is required.

---

# 121. Health Check and SSRF

If users can supply arbitrary URLs to a health-check tool, it can become an SSRF mechanism.

Restrict:

```text
allowed hosts
allowed schemes
allowed ports
```

when input is not fully trusted.

---

# 122. Localhost Restrictions

A public health-check API should not accept:

```text
http://127.0.0.1
http://169.254.169.254
```

or arbitrary private addresses unless explicitly intended.

This is especially important in cloud environments.

---

# 123. AWS Metadata Service

The instance metadata endpoint can contain sensitive information depending on configuration.

Health-check tools should never make arbitrary user-controlled requests to:

```text
169.254.169.254
```

---

# 124. Logging Health Results

Good:

```text
host=app01 check=disk status=WARN usage=84
```

Bad:

```text
host=app01 env=...SECRET...
```

Never serialize the entire environment into health logs.

---

# 125. Health Check Metrics

A Python checker can expose metrics such as:

```text
health_check_status
health_check_duration_seconds
health_check_failures_total
```

Prometheus can scrape these if the architecture requires it.

---

# 126. Prometheus Push vs Pull

For long-running services, Prometheus generally uses:

```text
pull/scrape
```

For short-lived jobs, Pushgateway may sometimes be appropriate, but it should not become a generic dumping ground for all application metrics.

---

# 127. Health Check JSON Endpoint

A long-running Python service could expose:

```text
GET /health
GET /ready
GET /metrics
```

But if the requirement is simply a scheduled CLI check, a web server may be unnecessary.

---

# 128. Cron Health Checks

A Linux cron job can execute:

```bash
*/5 * * * * \
/opt/health/bin/python \
/opt/health/check.py
```

Redirect output carefully.

For modern systems, systemd timers may be preferable.

---

# 129. Systemd Timer

Concept:

```text
health-check.service
health-check.timer
```

The timer can run the script periodically.

Advantages include:

```text
service isolation
logging through journal
dependency management
```

---

# 130. Exit Codes and Cron

A cron job can detect failures through:

```text
exit status
```

and integrate with an alerting mechanism.

Do not rely only on email from cron for production monitoring.

---

# 131. Health Check as a Systemd Service

A long-running checker can be managed by:

```bash
systemctl status health-agent
```

But a simple scheduled health check does not necessarily need a daemon.

---

# 132. Locking Health Checks

Avoid overlapping scheduled checks.

Example:

```text
previous run still active
        |
        v
new run starts?
        |
       NO
```

Use:

```text
flock
lockfile
systemd timer semantics
```

as appropriate.

---

# 133. Why Overlapping Checks Are Dangerous

If each check:

```text
opens 100 network connections
```

and runs every minute while taking two minutes:

```text
run 1
run 2
run 3
...
```

resource usage can grow unexpectedly.

---

# 134. Health Check Duration

Track:

```text
start
end
duration
```

A sudden increase can be an early warning.

Example:

```text
normally 2 seconds
now 25 seconds
```

Even if all checks eventually pass, latency may indicate a problem.

---

# 135. Slow Check Detection

Example policy:

```text
< 5 sec  PASS
5–10 sec WARN
> 10 sec FAIL
```

Again, thresholds depend on the check.

---

# 136. Health Check Time Budget

Set an overall budget:

```text
total health check < 30 seconds
```

Individual checks should have smaller timeouts.

This prevents a single dependency from consuming the entire execution window.

---

# 137. Timeout Budget

Example:

```text
DNS      2 sec
TCP      3 sec
HTTP     5 sec
SSH     10 sec
```

The total runtime remains bounded.

---

# 138. Health Check Dependency Ordering

Some checks depend on others.

Example:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP
```

If DNS fails, there is little value in repeatedly running the same hostname-based HTTP check.

---

# 139. Short-Circuiting

Possible strategy:

```text
DNS FAIL
 ↓
skip HTTP check
 ↓
report dependency failure
```

But do not hide the fact that HTTP was skipped.

Use:

```text
SKIPPED
```

when appropriate.

---

# 140. Health Status Model

A richer model:

```text
PASS
WARN
FAIL
UNKNOWN
SKIPPED
```

This improves reporting.

---

# 141. Dependency Graph

Example:

```text
DNS
 |
 v
TCP
 |
 v
TLS
 |
 v
HTTP
 |
 v
Application
```

The health engine can understand dependencies and avoid misleading results.

---

# 142. Health Check Message Design

Bad:

```text
FAIL
```

Better:

```text
FAIL: nginx is inactive
```

Best:

```text
FAIL: nginx is inactive; expected active; exit_code=3
```

Messages should help the engineer act.

---

# 143. Actionable Results

Example:

```text
FAIL disk_root:
usage=94%
available=4.2GB
threshold=90%
recommended_action=investigate disk growth
```

Avoid automatically deleting files unless explicitly designed and approved.

---

# 144. Health Check Remediation

A checker can optionally support:

```text
detect
 ↓
remediate
 ↓
verify
```

But remediation should be:

```text
allowlisted
idempotent
audited
safe
```

---

# 145. Detection vs Remediation

Prefer separating:

```text
health.py
```

from:

```text
remediate.py
```

This reduces accidental changes during monitoring.

---

# 146. Example Safe Remediation

Potentially:

```text
service stopped
 ↓
restart once
 ↓
verify
```

But automatic restarts should have:

```text
max attempts
cooldown
alerting
```

to avoid restart loops.

---

# 147. Restart Loop Risk

Bad automation:

```text
service down
 ↓
restart
 ↓
fails
 ↓
restart
 ↓
fails
 ↓
restart forever
```

Better:

```text
restart once
 ↓
verify
 ↓
alert
 ↓
stop
```

---

# 148. Disk Remediation Risk

Never automatically run:

```bash
rm -rf /var/log/*
```

as a generic disk remediation.

Instead:

```text
identify largest files
verify retention policy
rotate/archive
remove only approved temporary data
```

---

# 149. Health Check and Log Rotation

A common production issue:

```text
disk usage increasing
```

Investigate:

```text
logrotate
journald retention
application logs
container logs
temporary files
```

The health checker should point toward investigation rather than blindly deleting data.

---

# 150. Container Disk Health

For Docker hosts:

```bash
docker system df
```

But cleanup must be deliberate.

Unused images can consume significant space.

Do not blindly run:

```bash
docker system prune -a
```

on production hosts.

---

# 151. Kubernetes Disk Health

For Kubernetes nodes, inspect:

```text
node filesystem
container runtime storage
image filesystem
ephemeral storage
pod usage
```

Kubernetes may report:

```text
DiskPressure
```

when thresholds are crossed.

---

# 152. Ephemeral Storage

Applications can consume node storage through:

```text
logs
temporary files
emptyDir
container writable layers
images
```

Health checks should consider workload-specific storage behavior.

---

# 153. PID Pressure

Linux can run out of process IDs.

Kubernetes exposes:

```text
PIDPressure
```

Check:

```bash
ps -e --no-headers | wc -l
```

and relevant process limits.

---

# 154. File Descriptor Health

Processes can run out of file descriptors.

Check:

```bash
ulimit -n
```

and:

```bash
cat /proc/sys/fs/file-nr
```

depending on the diagnostic requirement.

---

# 155. File Descriptor Exhaustion

Symptoms:

```text
Too many open files
socket creation failures
HTTP failures
database connection failures
```

This can look like a network or application problem.

---

# 156. Python File Descriptor Check

The Python process can inspect its own resource limits:

```python
import resource

soft, hard = resource.getrlimit(
    resource.RLIMIT_NOFILE
)

print(soft, hard)
```

For system-wide diagnostics, inspect the host separately.

---

# 157. Process Limits

Useful concepts:

```text
ulimit
systemd LimitNOFILE
container limits
kernel limits
```

A healthy CPU/memory/disk system can still fail because of process limits.

---

# 158. Thread Health

Excessive threads can cause:

```text
memory consumption
scheduler pressure
context switching
```

Inspect process/thread counts when diagnosing high resource usage.

---

# 159. Context Switching

High context switching can indicate:

```text
too many runnable tasks
thread explosion
high concurrency
```

Use OS metrics and workload context before declaring a failure.

---

# 160. System Load Investigation

When load is high:

```text
CPU busy?
I/O wait?
many runnable processes?
disk latency?
network filesystem?
```

High load does not automatically mean CPU saturation.

---

# 161. I/O Wait

High I/O wait can indicate:

```text
slow disk
storage saturation
network filesystem
heavy database activity
```

Check:

```text
CPU breakdown
disk latency
iostat
application behavior
```

---

# 162. Python and `iostat`

Python can call:

```bash
iostat
```

if the package is installed.

But do not assume it exists on every Linux system.

A portable health checker should handle missing tools gracefully.

---

# 163. Missing Command Handling

Bad:

```python
subprocess.run(
    ["iostat"]
)
```

without handling missing executable.

Better:

```python
try:
    result = subprocess.run(
        ["iostat"],
        ...
    )
except FileNotFoundError:
    return unknown_result(
        "iostat unavailable"
    )
```

---

# 164. Prefer Native Python APIs

For:

```text
CPU
memory
disk
network
processes
```

psutil is often cleaner than parsing command output.

Use shell commands when:

```text
Linux-specific data
systemd
journalctl
specialized tools
```

are required.

---

# 165. Avoid Brittle Parsing

Avoid relying on:

```python
output.split()[3]
```

for command output whose format can change.

Prefer:

```text
structured APIs
/proc
/sys
JSON output
machine-readable command options
```

when available.

---

# 166. Machine-Readable Commands

Many tools support:

```text
--json
-P
--no-pager
```

where appropriate.

Use machine-readable output in automation.

---

# 167. `df -P`

The POSIX format:

```bash
df -P
```

is generally easier to parse consistently than human-oriented output.

Still prefer Python filesystem APIs when they meet the requirement.

---

# 168. `systemctl show`

For structured systemd information:

```bash
systemctl show nginx
```

It provides key-value properties that are more automation-friendly than human status output.

---

# 169. `journalctl --output`

For machine processing, journalctl offers structured output modes.

Choose the appropriate output format instead of parsing decorative terminal output.

---

# 170. Health Check Unit Tests

Test:

```text
threshold logic
status aggregation
missing metrics
timeouts
invalid configuration
command failures
```

Example:

```python
def test_disk_fail():
    assert (
        classify_disk(95)
        == "FAIL"
    )
```

---

# 171. Boundary Testing

Always test thresholds:

```text
79.9
80.0
80.1

89.9
90.0
90.1
```

Boundary errors are common in monitoring scripts.

---

# 172. Mocking System Metrics

For unit tests, mock:

```python
psutil.cpu_percent
psutil.virtual_memory
shutil.disk_usage
```

so tests don't depend on the machine running them.

---

# 173. Integration Testing

Integration tests should validate:

```text
real filesystem
real network socket
real HTTP endpoint
real systemd service
```

in an isolated environment.

---

# 174. Health Check CI Pipeline

```text
Pull Request
 ↓
Lint
 ↓
Unit Tests
 ↓
Integration Tests
 ↓
Security Scan
 ↓
Package
```

The health checker itself should be tested like production software.

---

# 175. Python Linting

Common tools:

```text
ruff
black
```

depending on project standards.

A consistent codebase is easier to operate.

---

# 176. Type Checking

For larger projects:

```text
mypy
pyright
```

can help catch incorrect assumptions before deployment.

---

# 177. Dependency Pinning

Example:

```text
psutil==...
```

Use a controlled dependency strategy.

Do not automatically upgrade production automation dependencies without testing.

---

# 178. Requirements File

Example:

```text
psutil
PyYAML
```

Pin or constrain versions according to project policy.

---

# 179. Virtual Environment

Create:

```bash
python3 -m venv .venv
```

Activate:

```bash
source .venv/bin/activate
```

Install:

```bash
pip install -r requirements.txt
```

---

# 180. Health Checker CLI

Example:

```bash
python health.py
```

Options:

```text
--config
--role
--environment
--json
--verbose
--timeout
--check
--output
```

---

# 181. `argparse`

Example:

```python
import argparse

parser = argparse.ArgumentParser()

parser.add_argument(
    "--config",
    required=True,
)

parser.add_argument(
    "--json",
    action="store_true",
)

args = parser.parse_args()
```

---

# 182. Selective Checks

Useful:

```bash
python health.py \
    --check disk
```

or:

```bash
python health.py \
    --check network
```

This is useful during troubleshooting.

---

# 183. All Checks

```bash
python health.py \
    --check all
```

The default should be safe and predictable.

---

# 184. JSON Output

```bash
python health.py \
    --json
```

Useful for:

```text
CI
automation
dashboards
machine processing
```

---

# 185. Human Output

Example:

```text
SYSTEM HEALTH
--------------------------------
CPU       PASS   41%
Memory    PASS   58%
Disk      WARN   84%
Nginx     PASS   active
HTTP      PASS   200 / 0.12s
DNS       PASS
--------------------------------
OVERALL   WARN
```

Human-readable output is useful for engineers.

---

# 186. Quiet Mode

A CI tool can support:

```bash
--quiet
```

and return only the exit code.

This reduces unnecessary log noise.

---

# 187. Verbose Mode

```bash
--verbose
```

can include:

```text
command details
timings
debug information
```

Never expose secrets in verbose output.

---

# 188. Health Check Logging Levels

Use:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

Appropriately.

Do not log every normal metric at ERROR level.

---

# 189. Correlation IDs

For scheduled or CI runs:

```text
health-run-20260817-1200
```

include the ID in:

```text
logs
JSON
notifications
artifacts
```

---

# 190. Health Report

A report can contain:

```json
{
  "timestamp": "2026-08-17T12:00:00Z",
  "host": "app01",
  "overall": "WARN",
  "checks": []
}
```

Use UTC internally for cross-system correlation.

---

# 191. Time Zones

Servers may run:

```text
UTC
```

while engineers work in:

```text
IST
```

Store timestamps consistently and convert for presentation.

---

# 192. Clock Skew

If server timestamps differ significantly, incident timelines become confusing.

Time synchronization is therefore part of system health.

---

# 193. Health Check Notifications

A failed health check can trigger:

```text
Jenkins failure
alert
Slack notification
email
incident platform
```

Avoid sending notifications for every transient warning.

Use alert policies and deduplication.

---

# 194. Alert Fatigue

Bad:

```text
disk 80% → alert
disk 80% → alert
disk 80% → alert
```

Better:

```text
threshold
+
duration
+
deduplication
+
recovery notification
```

Monitoring systems are better suited for continuous alert state.

---

# 195. Python vs Prometheus Alerting

Python is useful for:

```text
preflight
on-demand diagnostics
deployment validation
scheduled checks
```

Prometheus/Alertmanager is better suited for:

```text
continuous time-series monitoring
alert rules
deduplication
silencing
routing
```

Use each where it fits.

---

# 196. Health Check and Prometheus

A Python health checker can expose:

```text
health_check_status 1
health_check_duration_seconds 0.42
```

Prometheus can then monitor those results.

---

# 197. Health Check and Grafana

Grafana can visualize:

```text
health status
duration
failure count
resource trends
```

A single health result is less valuable than its history.

---

# 198. Health Check and ELK

Send structured health events to:

```text
Logstash
 ↓
Elasticsearch
 ↓
Kibana
```

Useful fields:

```text
host
environment
check
status
value
threshold
duration
run_id
```

---

# 199. Observability Correlation

A health failure should lead to:

```text
health result
 ↓
metrics
 ↓
logs
 ↓
deployment history
```

This reduces mean time to recovery.

---

# 200. Production Health Dashboard

Useful panels:

```text
Healthy Hosts
Warning Hosts
Failed Hosts
Disk Risk
Memory Risk
Service Failures
Network Failures
Recent Deployments
Health Check Duration
```

---

# 201. Fleet Health

For 100 servers:

```text
95 PASS
4 WARN
1 FAIL
```

A fleet-level health summary is more useful than scrolling through 100 outputs.

---

# 202. Fleet Health Aggregation

```python
summary = {
    "total": len(results),
    "pass": 0,
    "warn": 0,
    "fail": 0,
    "unknown": 0,
}
```

Increment based on result status.

---

# 203. Failure Percentage

Example:

```text
failed hosts / total hosts * 100
```

For rollout decisions:

```text
0% failure → continue
> 5%        → pause
```

The threshold should be defined by the deployment policy.

---

# 204. Canary Health

Before rolling out widely:

```text
deploy canary
 ↓
health checks
 ↓
metrics
 ↓
logs
 ↓
decision
```

Python can automate the decision based on explicit criteria.

---

# 205. Health During Rolling Deployment

Check:

```text
new version healthy
old instances healthy
error rate acceptable
latency acceptable
capacity sufficient
```

A simple process-level check is not enough for a distributed application.

---

# 206. Capacity Health

A deployment can make an otherwise healthy cluster unhealthy if capacity becomes insufficient.

Check:

```text
CPU headroom
memory headroom
disk
connections
replicas
```

before a large rollout.

---

# 207. Resource Headroom

Example:

```text
current CPU = 55%
deployment expected +20%
```

Potential peak:

```text
75%
```

The deployment may be safe.

But actual behavior must be validated with workload data.

---

# 208. Connection Health

Applications may fail because of:

```text
file descriptors
database connections
TCP sockets
ephemeral ports
```

Health checks should include these when they are known bottlenecks.

---

# 209. TCP Connection States

Linux:

```bash
ss -s
```

Useful for identifying:

```text
TIME_WAIT
ESTABLISHED
SYN-SENT
CLOSE-WAIT
```

Large unexpected counts can indicate application/network issues.

---

# 210. `CLOSE_WAIT`

A large number of:

```text
CLOSE_WAIT
```

can indicate that the application is not closing sockets correctly.

This is a diagnostic signal, not automatically a root cause.

---

# 211. `TIME_WAIT`

High `TIME_WAIT` may be normal for high connection churn.

Investigate:

```text
connection rate
ephemeral port usage
application behavior
load-balancer behavior
```

before changing kernel settings.

---

# 212. Ephemeral Ports

A host making many outbound connections can exhaust ephemeral ports.

Symptoms:

```text
cannot establish new connections
intermittent network failures
```

This is an advanced health-check scenario.

---

# 213. File Descriptor and Socket Health

Combine:

```text
open files
socket counts
process limits
connection states
application connection pools
```

to diagnose resource exhaustion.

---

# 214. Process Zombie Check

Linux processes can become:

```text
Z
```

zombies.

A few may be harmless depending on the system, but persistent growth indicates a parent-process problem.

---

# 215. Zombie Count

Using psutil:

```python
zombies = 0

for process in psutil.process_iter(
    ["status"]
):
    if (
        process.info["status"]
        == psutil.STATUS_ZOMBIE
    ):
        zombies += 1

print(zombies)
```

Alert based on trend and host role.

---

# 216. Process Count

```python
len(
    list(
        psutil.process_iter()
    )
)
```

A sudden increase can indicate:

```text
fork storm
restart loop
worker explosion
```

Use historical comparison where possible.

---

# 217. Service Restart Count

Systemd can provide restart information depending on unit configuration.

A service repeatedly restarting is a stronger signal than simply:

```text
currently active
```

---

# 218. Application Version Drift

Check:

```text
expected version
actual version
```

across the fleet.

Example:

```text
app01 = 2.1.0
app02 = 2.1.0
app03 = 2.0.8
```

This can reveal incomplete deployments.

---

# 219. Configuration Drift

Compare:

```text
expected checksum
actual checksum
```

or use configuration management's desired state.

Python can report drift even if it does not remediate it.

---

# 220. Package Drift

Compare:

```text
required package
installed version
```

This is useful for validating deployment prerequisites.

---

# 221. Runtime Drift

Check:

```text
Java
Node.js
Python
Docker
containerd
nginx
```

versions where relevant.

Runtime mismatch is a common deployment problem.

---

# 222. Health Check Before Deployment

Example:

```text
OS compatible?
Runtime version?
Disk enough?
Memory enough?
Service exists?
Port free?
Network dependencies reachable?
```

This prevents predictable deployment failures.

---

# 223. Health Check After Deployment

Example:

```text
service active?
port listening?
health endpoint?
expected version?
dependencies?
logs?
```

This proves the change actually worked.

---

# 224. Preflight vs Postflight

```text
PRE
=
Can I safely deploy?

POST
=
Did deployment succeed?
```

They should not be identical.

---

# 225. Health Check as a Contract

A production service should define:

```text
what healthy means
what ready means
what critical means
what warning means
```

The Python tool should implement that contract.

---

# 226. Health Contract Example

```yaml
service: myapp

health:
  critical:
    - service_active
    - port_open
    - http_200

  warning:
    - disk_above_80
    - memory_above_80
```

---

# 227. Health Check Documentation

Document:

```text
purpose
inputs
checks
thresholds
exit codes
security
failure behavior
remediation
```

A health script without documentation becomes difficult to trust.

---

# 228. Production Runbook

For every FAIL result, provide a runbook reference:

```text
disk_full
→ investigate disk growth

service_down
→ check journalctl

dns_failure
→ inspect resolver/network

http_failure
→ inspect application logs/dependencies
```

This turns monitoring into actionable operations.

---

# 229. Health Check Ownership

Define:

```text
owner
team
severity
runbook
escalation
```

A technical alert without ownership often becomes noise.

---

# 230. Health Check Maintenance

Review periodically:

```text
thresholds
false positives
false negatives
unused checks
execution time
security
dependencies
```

Health checks must evolve with the infrastructure.

---

# 231. False Positives

Example:

```text
CPU > 80%
```

during a scheduled backup.

If this repeatedly triggers alerts, adjust:

```text
threshold
duration
schedule
host role
```

rather than disabling monitoring entirely.

---

# 232. False Negatives

Example:

```text
service active
```

but:

```text
HTTP returns 500
```

The check is too shallow.

Add an application-level signal.

---

# 233. Health Check Layering

Use:

```text
host check
+
service check
+
application check
+
dependency check
```

rather than one overly complicated endpoint.

---

# 234. Dependency Failure Propagation

If:

```text
database down
```

then:

```text
application unhealthy
```

But the root cause is:

```text
database
```

Good health reporting distinguishes:

```text
symptom
vs
dependency failure
```

---

# 235. Root Cause vs Symptom

Example:

```text
HTTP 503
```

may be caused by:

```text
database timeout
```

The health checker should report both when possible.

---

# 236. Health Check Event

Example:

```json
{
  "host": "app01",
  "check": "http",
  "status": "FAIL",
  "status_code": 503,
  "dependency": "database",
  "duration_ms": 2400
}
```

Structured data enables better correlation.

---

# 237. Incident Timeline

Health checks can provide timestamps:

```text
10:01 service PASS
10:03 latency WARN
10:04 database FAIL
10:04 HTTP FAIL
10:05 deployment started
```

This can help correlate the incident with changes.

---

# 238. Deployment Correlation

Include:

```text
deployment_id
commit_sha
version
environment
```

in health reports.

Then you can determine:

```text
did the failure start after deployment?
```

---

# 239. Git Commit Correlation

Example:

```json
{
  "version": "2.4.1",
  "commit": "abc1234",
  "health": "PASS"
}
```

This is valuable during release troubleshooting.

---

# 240. Health Check and Change Management

A failed post-deployment check can automatically:

```text
fail CI/CD
pause rollout
create incident
```

depending on organizational policy.

---

# 241. Health Check and GitOps

For Kubernetes:

```text
Git change
 ↓
Argo CD sync
 ↓
Kubernetes rollout
 ↓
readiness
 ↓
Prometheus/Grafana
```

Python can perform external validation after the GitOps deployment if needed.

---

# 242. Health Check and Argo CD

Do not use SSH to determine whether an EKS application rollout succeeded.

Use:

```text
Argo CD
Kubernetes
readiness
service
ingress
metrics
```

SSH is for node-level troubleshooting when necessary.

---

# 243. Health Check and Prometheus

Prometheus provides historical data such as:

```text
CPU
memory
disk
latency
error rate
```

Python health checks can complement it with:

```text
preflight
deployment validation
custom operational checks
```

---

# 244. Health Check and ELK

ELK can answer:

```text
what errors occurred?
when?
on which host?
after which deployment?
```

A Python checker can collect targeted logs during an incident.

---

# 245. Health Check and Alertmanager

Alertmanager handles:

```text
routing
grouping
deduplication
silencing
notifications
```

Python should not recreate all of these features unnecessarily.

---

# 246. Production Architecture

```text
                    USERS
                      |
                    ALB
                      |
                 APPLICATION
                      |
        +-------------+-------------+
        |             |             |
      Metrics        Logs        Health
        |             |             |
   Prometheus         ELK       Python
        |                           |
     Grafana                     CI/CD
```

The systems complement one another.

---

# 247. System Health Checker Project

## Project Goal

Build a reusable Python CLI that checks:

```text
CPU
memory
swap
disk
inodes
services
ports
DNS
HTTP
TLS
network
failed systemd units
recent critical logs
process limits
application version
```

and produces:

```text
console report
JSON report
exit code
```

---

# 248. Project Structure

```text
system-health-checker/
├── config/
│   ├── dev.yaml
│   ├── staging.yaml
│   └── prod.yaml
├── src/
│   ├── cli.py
│   ├── checks/
│   │   ├── cpu.py
│   │   ├── memory.py
│   │   ├── disk.py
│   │   ├── network.py
│   │   ├── services.py
│   │   └── application.py
│   ├── models.py
│   └── reporter.py
├── tests/
├── requirements.txt
└── README.md
```

---

# 249. Project Dependencies

Possible:

```text
psutil
PyYAML
```

Optional:

```text
requests
```

Keep the dependency footprint small.

---

# 250. Project Configuration

```yaml
thresholds:
  cpu_warn: 80
  cpu_fail: 95

  memory_warn: 80
  memory_fail: 90

  disk_warn: 80
  disk_fail: 90

services:
  - nginx

ports:
  - 80
  - 443

urls:
  - https://localhost/health
```

---

# 251. Project CPU Check

```python
import psutil

def check_cpu(
    warn=80,
    fail=95,
):
    value = psutil.cpu_percent(
        interval=1
    )

    if value >= fail:
        status = "FAIL"
    elif value >= warn:
        status = "WARN"
    else:
        status = "PASS"

    return {
        "name": "cpu",
        "status": status,
        "value": value,
    }
```

---

# 252. Project Memory Check

```python
import psutil

def check_memory(
    warn=80,
    fail=90,
):
    value = (
        psutil.virtual_memory()
        .percent
    )

    if value >= fail:
        status = "FAIL"
    elif value >= warn:
        status = "WARN"
    else:
        status = "PASS"

    return {
        "name": "memory",
        "status": status,
        "value": value,
    }
```

For more accurate Linux decisions, consider available memory and pressure rather than percentage alone.

---

# 253. Project Disk Check

```python
import shutil

def check_disk(
    path="/",
    warn=80,
    fail=90,
):
    usage = shutil.disk_usage(
        path
    )

    value = (
        usage.used
        / usage.total
        * 100
    )

    if value >= fail:
        status = "FAIL"
    elif value >= warn:
        status = "WARN"
    else:
        status = "PASS"

    return {
        "name": f"disk:{path}",
        "status": status,
        "value": value,
    }
```

---

# 254. Project Service Check

```python
import subprocess

def check_service(name):
    result = subprocess.run(
        [
            "systemctl",
            "is-active",
            name,
        ],
        capture_output=True,
        text=True,
        check=False,
    )

    return {
        "name": (
            f"service:{name}"
        ),
        "status": (
            "PASS"
            if result.returncode == 0
            else "FAIL"
        ),
        "message":
            result.stdout.strip(),
    }
```

Use trusted configuration for service names.

---

# 255. Project Port Check

```python
import socket

def check_port(
    host,
    port,
    timeout=3,
):
    try:
        with socket.create_connection(
            (host, port),
            timeout=timeout,
        ):
            return {
                "status": "PASS",
                "message": (
                    f"{host}:{port} reachable"
                ),
            }

    except OSError as exc:
        return {
            "status": "FAIL",
            "message": str(exc),
        }
```

---

# 256. Project DNS Check

```python
import socket

def check_dns(hostname):
    try:
        address = socket.gethostbyname(
            hostname
        )

        return {
            "status": "PASS",
            "address": address,
        }

    except socket.gaierror as exc:
        return {
            "status": "FAIL",
            "message": str(exc),
        }
```

---

# 257. Project HTTP Check

```python
import time
import urllib.request

def check_http(
    url,
    timeout=5,
):
    start = time.monotonic()

    try:
        with urllib.request.urlopen(
            url,
            timeout=timeout,
        ) as response:

            duration = (
                time.monotonic()
                - start
            )

            return {
                "status": (
                    "PASS"
                    if 200 <=
                    response.status < 400
                    else "FAIL"
                ),
                "status_code":
                    response.status,
                "duration":
                    duration,
            }

    except Exception as exc:
        return {
            "status": "FAIL",
            "message": str(exc),
        }
```

In production, catch narrower exception types and handle redirects according to policy.

---

# 258. Project Aggregator

```python
def overall_status(results):
    statuses = {
        item["status"]
        for item in results
    }

    if "FAIL" in statuses:
        return "FAIL"

    if "WARN" in statuses:
        return "WARN"

    if "UNKNOWN" in statuses:
        return "UNKNOWN"

    return "PASS"
```

Define your precedence explicitly.

---

# 259. Project JSON Reporter

```python
import json

def write_report(
    results,
    path,
):
    report = {
        "overall":
            overall_status(results),
        "checks":
            results,
    }

    with open(
        path,
        "w",
        encoding="utf-8",
    ) as file:

        json.dump(
            report,
            file,
            indent=2,
        )
```

---

# 260. Project Exit Code

```python
status = overall_status(
    results
)

if status == "PASS":
    raise SystemExit(0)

if status == "WARN":
    raise SystemExit(0)

raise SystemExit(1)
```

Whether WARN should fail CI is a policy decision.

---

# 261. Project Dry Run

Dry run is less important for read-only checks, but useful when remediation exists.

Example:

```bash
python health.py \
    --remediate \
    --dry-run
```

Output:

```text
Would restart nginx
```

---

# 262. Project Remediation

Example:

```text
service inactive
 ↓
allowed to restart?
 ↓
restart once
 ↓
verify
```

Remediation should never be silently enabled in production.

---

# 263. Project Remote Mode

The checker can support:

```bash
python health.py \
    --host app01
```

and execute checks over SSH.

This connects the current section with:

```text
06-SSH-Automation.md
```

---

# 264. Local vs Remote Architecture

```text
Local mode:
Python → local Linux

Remote mode:
Python → SSH → Linux
```

The check functions should ideally remain independent of the transport.

---

# 265. Transport Abstraction

Concept:

```python
class Executor:
    def run(self, command):
        ...
```

Implement:

```text
LocalExecutor
SSHExecutor
```

Then the same health checks can work in both modes.

---

# 266. Why Abstraction Helps

Without abstraction:

```text
check_disk()
```

contains SSH logic.

With abstraction:

```text
check_disk(executor)
```

can run against:

```text
local host
remote host
test mock
```

This improves testability.

---

# 267. Remote Executor

Concept:

```python
class SSHExecutor:

    def __init__(self, client):
        self.client = client

    def run(self, command):
        return ...
```

This reuses the SSH automation layer.

---

# 268. Local Executor

```python
class LocalExecutor:

    def run(self, command):
        return subprocess.run(
            command,
            ...
        )
```

Use structured arguments when possible.

---

# 269. Health Check Dependency Injection

Example:

```python
def check_service(
    executor,
    service,
):
    result = executor.run(
        [
            "systemctl",
            "is-active",
            service,
        ]
    )

    ...
```

This makes unit testing much easier.

---

# 270. Test with Fake Executor

```python
class FakeExecutor:

    def run(self, command):
        return {
            "exit_code": 0,
            "stdout": "active",
            "stderr": "",
        }
```

Tests no longer require a real Linux service.

---

# 271. Health Check Integration Test

Use an isolated Linux environment and validate:

```text
systemd
network
HTTP
disk
```

depending on the CI environment.

---

# 272. Health Check Packaging

A production tool can be packaged as:

```text
Python wheel
```

or installed through:

```text
internal package repository
```

rather than copying source files manually.

---

# 273. CLI Entry Point

A package can expose:

```bash
system-health
```

instead of:

```bash
python health.py
```

This improves usability.

---

# 274. Health Check Version

Expose:

```bash
system-health --version
```

This helps identify which checker version produced a report.

---

# 275. Self-Health

The health checker itself should validate:

```text
configuration loaded
required dependency available
time reasonable
output writable
```

Otherwise a broken checker may silently produce misleading results.

---

# 276. Self-Test

Example:

```bash
system-health \
    --self-test
```

Checks:

```text
config
permissions
dependencies
basic system APIs
```

---

# 277. Failure of the Checker

Important distinction:

```text
system unhealthy
```

vs:

```text
checker failed
```

Do not report:

```text
PASS
```

when the checker could not execute a critical test.

---

# 278. Unknown State

Example:

```text
HTTP check
→ timeout

status = UNKNOWN
```

or:

```text
FAIL
```

depending on whether timeout means service failure in that specific contract.

Define this explicitly.

---

# 279. Health Check Reliability

A health checker should itself be:

```text
fast
deterministic
low overhead
secure
fault tolerant
```

Do not make it so complex that the checker becomes another production dependency.

---

# 280. Avoid Heavy Health Checks

Avoid running:

```text
large database query
full filesystem scan
huge log search
deep package inventory
```

every minute.

Health checks should have a small operational footprint.

---

# 281. Health Check Cost

For 1,000 servers:

```text
1000 × 20 checks
```

can create substantial load.

Design for fleet scale.

---

# 282. Fleet Scheduling

Avoid starting every server check at exactly:

```text
00 seconds
```

This creates a thundering herd.

Use:

```text
jitter
distributed scheduling
central scheduler
```

as appropriate.

---

# 283. Jitter

If checks run every 5 minutes, add a small random offset:

```text
300 sec ± jitter
```

This spreads load.

---

# 284. Health Check Caching

For expensive checks, a short cache may be useful.

But stale health information can be dangerous.

Use caching only when the freshness requirement allows it.

---

# 285. Health Data Freshness

Every result should include:

```text
timestamp
duration
```

so consumers know how fresh it is.

---

# 286. Health Check SLA

Define:

```text
maximum execution time
maximum stale age
failure detection target
```

For example:

```text
health result must be < 2 minutes old
```

---

# 287. Health Check SLO

A monitoring system may track:

```text
health checker availability
```

because if the checker itself fails, you lose visibility.

---

# 288. Checker Availability

Monitor:

```text
successful executions
execution duration
failed executions
missing reports
```

A missing health result can itself be an alert condition.

---

# 289. Missing Data

Important distinction:

```text
FAIL
```

means:

```text
check ran and detected a problem
```

while:

```text
NO DATA
```

means:

```text
check did not run or result was unavailable
```

Alerting should distinguish them.

---

# 290. Health Check Heartbeat

A scheduled checker can emit:

```text
heartbeat
```

to indicate:

```text
the checker itself is alive
```

---

# 291. Production Health Architecture

```text
              Scheduler / CI
                    |
                    v
             Python Health CLI
                    |
       +------------+------------+
       |            |            |
     Local         SSH          API
       |            |            |
      Host        Hosts        Cloud/K8s
       |            |            |
       +------------+------------+
                    |
                Results
                    |
          +---------+---------+
          |         |         |
         JSON      Logs     Metrics
          |         |         |
         CI        ELK    Prometheus
```

---

# 292. AWS Health Checks

For EC2, combine:

```text
instance state
status checks
network
disk
memory
application
```

AWS infrastructure state alone does not prove application health.

---

# 293. EC2 Status Checks

AWS provides:

```text
system status
instance status
```

These can detect underlying infrastructure or instance-level issues.

Python can retrieve AWS information using:

```text
boto3
```

when required.

---

# 294. Python + Boto3

Example:

```python
import boto3

ec2 = boto3.client(
    "ec2"
)

response = (
    ec2.describe_instance_status(
        InstanceIds=[
            "i-xxxxxxxx"
        ],
        IncludeAllInstances=True,
    )
)
```

Use IAM permissions with least privilege.

---

# 295. AWS Health Architecture

```text
boto3
 ↓
EC2 status
 ↓
SSH/SSM
 ↓
OS health
 ↓
Application health
```

This gives multiple layers of validation.

---

# 296. SSM Health

Where available, AWS Systems Manager can provide managed execution without direct SSH.

Python can invoke approved SSM commands through:

```text
boto3
```

This may be preferable to opening port 22.

---

# 297. SSM vs SSH

Use SSM when:

```text
AWS-native access
private instances
centralized permissions
auditing
no inbound SSH
```

Use SSH when:

```text
required by application/tooling
approved operational model
non-AWS environment
traditional access path
```

---

# 298. EKS Health

For EKS, Python can use:

```text
boto3
Kubernetes API
kubectl
```

to inspect:

```text
nodes
pods
deployments
services
events
```

---

# 299. EKS Node Health Check

A useful workflow:

```text
kubectl get nodes
 ↓
find NotReady
 ↓
inspect node conditions
 ↓
collect node-level diagnostics
 ↓
correlate Prometheus/ELK
```

Only use host access when needed.

---

# 300. Production EKS Scenario

If a node is:

```text
NotReady
```

do not immediately:

```text
reboot node
```

First inspect:

```text
kubelet
containerd
disk pressure
memory pressure
network
CNI
API connectivity
```

---

# 301. Network Health in EKS

Check:

```text
node IP
pod CIDR
VPC routing
security groups
CNI
DNS
service connectivity
```

Python can automate preflight checks but should not replace Kubernetes and AWS observability.

---

# 302. Health Check and ALB

For an ALB-backed application:

```text
ALB target health
+
application health
+
node health
```

all matter.

A healthy server with an unhealthy ALB target still means user traffic may fail.

---

# 303. End-to-End Health

Strongest validation:

```text
DNS
 ↓
ALB
 ↓
Target
 ↓
Ingress
 ↓
Service
 ↓
Pod
 ↓
Application
 ↓
Database
```

Use the correct layer for each check.

---

# 304. Production Troubleshooting — CPU High

Step-by-step:

```text
1. Confirm CPU utilization
2. Check load average
3. Identify top processes
4. Check I/O wait
5. Check recent deployment
6. Check traffic
7. Check application logs
8. Compare historical metrics
```

Do not kill the top process without understanding its role.

---

# 305. Production Troubleshooting — Memory High

```text
1. Check available memory
2. Check swap
3. Check top memory processes
4. Check OOM events
5. Check recent deployment
6. Check application behavior
7. Check container limits
```

---

# 306. Production Troubleshooting — Disk High

```text
1. df -h
2. df -i
3. identify filesystem
4. find large directories
5. check logs
6. check deleted-open files
7. check container storage
8. check retention/rotation
```

---

# 307. Deleted Files Still Consume Disk

A process can keep a deleted file open.

Use:

```bash
lsof +L1
```

This explains situations where:

```text
file deleted
but disk still full
```

until the process closes the descriptor.

---

# 308. Python Health Tool and `lsof`

A diagnostic checker can optionally run:

```bash
lsof +L1
```

when disk usage is high.

This is a diagnostic extension, not a check that needs to run constantly.

---

# 309. Production Troubleshooting — Service Down

```text
systemctl status
 ↓
journalctl
 ↓
configuration
 ↓
dependencies
 ↓
permissions
 ↓
port
 ↓
resource limits
```

Then restart only when the recovery procedure calls for it.

---

# 310. Production Troubleshooting — HTTP 500

Check:

```text
application logs
dependencies
database
recent deployment
configuration
resource usage
```

A host-level health check alone cannot determine the application root cause.

---

# 311. Production Troubleshooting — HTTP Timeout

Check:

```text
DNS
network route
TCP
TLS
load balancer
application listener
application latency
database/dependency latency
```

---

# 312. Production Troubleshooting — DNS Failure

Check:

```text
resolver
nameserver
routing
security controls
DNS records
TTL
split-horizon configuration
```

Then compare from:

```text
application host
developer machine
other region
```

---

# 313. Production Troubleshooting — Network Port Closed

Check:

```text
service
listener
firewall
security group
NACL
route
container port
load balancer target
```

Do not change firewall rules before identifying the intended architecture.

---

# 314. Production Troubleshooting — Time Drift

Symptoms:

```text
TLS errors
authentication failures
log ordering problems
token validation failures
```

Check:

```text
timedatectl
chrony
NTP
```

according to the OS.

---

# 315. Production Troubleshooting — File Descriptor Exhaustion

Check:

```text
ulimit
process open files
system-wide limits
socket states
connection pool
application leaks
```

The correct fix may be application-level rather than simply raising the limit.

---

# 316. Production Troubleshooting — Zombie Growth

Check:

```text
parent process
application supervisor
child process handling
recent release
```

Do not blindly kill zombies; they must be reaped by their parent or the parent must be addressed.

---

# 317. Production Troubleshooting — High Load but Low CPU

Possible causes:

```text
I/O wait
disk latency
blocked tasks
network filesystem
storage
```

This is why load and CPU should be analyzed together.

---

# 318. Production Troubleshooting — Disk Low but Application Writes Fail

Possible causes:

```text
inode exhaustion
permissions
read-only filesystem
file descriptor limits
quota
container limits
```

---

# 319. Read-Only Filesystem

Check:

```bash
mount
findmnt
dmesg
```

A filesystem may become read-only after storage errors.

Do not remount blindly without understanding the underlying issue.

---

# 320. Filesystem Errors

Health diagnostics can look for:

```text
I/O errors
filesystem errors
read-only transitions
```

Storage-related failures should be treated carefully.

---

# 321. Health Check Security Review

Before production:

```text
Does it execute shell commands?
Can users control command arguments?
Does it connect to arbitrary URLs?
Does it access secrets?
Does it run as root?
Can it modify the system?
Can it exfiltrate data?
Does it log sensitive information?
```

---

# 322. Least Privilege

A read-only health checker should ideally run without root.

Only use elevated privileges for checks that genuinely require them.

---

# 323. Read-Only Health Check

Most checks can be:

```text
read CPU
read memory
read disk
read service state
read network
read logs
```

Avoid making health checks mutate state.

---

# 324. Health Check as Non-Root

Example:

```bash
sudo -u healthcheck \
    system-health
```

If some checks require privileged access, split them into a controlled privileged component.

---

# 325. Privileged Helper Risk

A privileged helper can become a privilege-escalation path if it accepts:

```text
arbitrary commands
arbitrary paths
arbitrary URLs
```

Keep privileged operations narrowly scoped.

---

# 326. Health Check Secrets

A health check should not need application secrets whenever possible.

Prefer:

```text
unauthenticated local health endpoint
```

or a safe read-only credential with minimum privileges.

---

# 327. Database Health Credentials

If a database query is required:

```text
read-only account
limited schema
limited query
secret manager
```

Avoid using the application's full-privilege database credential.

---

# 328. Health Check Network Access

Restrict outbound access if the checker does not need arbitrary internet connectivity.

This reduces the impact of compromise.

---

# 329. Health Check Supply Chain

Protect:

```text
Python dependencies
container image
CI workflow
configuration
credentials
```

A compromised health checker can become a powerful privileged tool.

---

# 330. Final Production Checklist

```text
[ ] Checks have explicit definitions
[ ] Thresholds are documented
[ ] PASS/WARN/FAIL/UNKNOWN are defined
[ ] Every network operation has timeout
[ ] Retries are bounded
[ ] Concurrency is bounded
[ ] Commands are safe
[ ] Inputs are validated
[ ] No secrets in logs
[ ] Least privilege
[ ] JSON output available
[ ] Exit codes documented
[ ] Health checks tested
[ ] CI/CD integration tested
[ ] Monitoring integration defined
[ ] Runbooks linked
[ ] Ownership defined
[ ] False positives reviewed
[ ] Fleet-scale impact considered
```

---

# 331. Interview — What Is a System Health Check?

**Answer:**

> A system health check is a repeatable validation of important operating conditions such as CPU, memory, disk, services, network connectivity, and application health. In DevOps I prefer structured results with PASS, WARN, FAIL, or UNKNOWN states and clear exit codes so the checks can integrate with CI/CD and monitoring.

---

# 332. Interview — How Do You Check CPU in Python?

**Answer:**

> I can use psutil for CPU utilization and Python's os.getloadavg on Linux for load average. I don't treat CPU percentage alone as the complete picture; I compare utilization, load, CPU count, I/O wait where relevant, and application behavior.

---

# 333. Interview — What Is the Difference Between CPU Usage and Load Average?

**Answer:**

> CPU usage measures how much CPU time is being consumed, while load average represents the number of tasks waiting for CPU or certain system resources. Load must be interpreted relative to CPU count and I/O behavior.

---

# 334. Interview — How Do You Check Memory?

**Answer:**

> With psutil I can inspect virtual memory, especially available memory and utilization. On Linux I also consider swap, OOM events, and memory pressure because free memory alone can be misleading due to filesystem caching.

---

# 335. Interview — How Do You Check Disk Usage?

**Answer:**

> I can use shutil.disk_usage for filesystem capacity and os.statvfs for inode information. I check both disk space and inode availability because a filesystem can have free capacity but still be unable to create files when inodes are exhausted.

---

# 336. Interview — How Do You Check Whether a Service Is Running?

**Answer:**

> For systemd-managed services I use `systemctl is-active` or structured systemd information. I don't rely only on process existence because a process can exist while the application is unhealthy.

---

# 337. Interview — Is an Open Port Enough to Prove an Application Is Healthy?

**Answer:**

> No. An open port proves that something is accepting TCP connections. The application may still return errors. I prefer an application-level health endpoint or protocol-level validation when available.

---

# 338. Interview — How Do You Check Network Connectivity?

**Answer:**

> I test the protocol relevant to the dependency. For a TCP service I use a TCP connection check rather than relying only on ping. For HTTPS I validate DNS, TCP, TLS, HTTP status, and latency.

---

# 339. Interview — How Do You Handle Timeouts?

**Answer:**

> Every network or remote operation gets an explicit timeout. I also use an overall execution budget so one unavailable dependency cannot block the complete health-check program indefinitely.

---

# 340. Interview — How Do You Handle Retries?

**Answer:**

> I use bounded retries with exponential backoff and sometimes jitter for transient failures. I don't blindly retry permanent failures such as authentication errors or invalid configuration.

---

# 341. Interview — How Do You Design Health Check Results?

**Answer:**

> I return structured results containing the check name, status, value, threshold, message, timestamp, and duration. JSON output makes the results easy for CI/CD and monitoring systems to consume.

---

# 342. Interview — What Should the Exit Code Be?

**Answer:**

> I normally use zero for successful execution and non-zero for a critical failure. The exact mapping can distinguish invalid configuration from system failure. WARN behavior should be explicitly defined depending on whether the tool is being used for monitoring or deployment gating.

---

# 343. Interview — How Do You Avoid False Positives?

**Answer:**

> I avoid relying on a single metric. I define thresholds based on workload behavior, use duration-based conditions where appropriate, distinguish warning from failure, and continuously review historical false positives.

---

# 344. Interview — How Do You Avoid False Negatives?

**Answer:**

> I validate at multiple layers. For example, checking that a systemd service is active is not enough, so I also check the listening port and application health endpoint. For user-facing applications I validate the external path as well.

---

# 345. Interview — How Would You Build a Health Checker for 1,000 Servers?

**Answer:**

> I would avoid opening 1,000 connections simultaneously. I would use inventory, bounded concurrency, batching, timeouts, retries for transient failures, structured results, and centralized reporting. I would also consider whether Prometheus or another monitoring platform is better for continuous checks.

---

# 346. Interview — Python or Prometheus for Health Checks?

**Answer:**

> Python is excellent for on-demand checks, deployment preflight, post-deployment validation, custom workflows, and diagnostics. Prometheus is better for continuous time-series monitoring and alerting. In production I often use both.

---

# 347. Interview — How Would You Integrate a Health Checker with Jenkins?

**Answer:**

> Jenkins runs the Python tool before deployment as a preflight gate and after deployment as a postflight validation. The Python process returns a defined exit code and JSON report. Jenkins stops the pipeline or triggers rollback when critical checks fail.

---

# 348. Interview — How Would You Check an EKS Node?

**Answer:**

> I first inspect Kubernetes node conditions, events, metrics, and workloads. If host-level diagnostics are required, I inspect disk, memory, kubelet, containerd, and networking through an approved access method. I don't use SSH as the primary way to manage Kubernetes workloads.

---

# 349. Interview — How Would You Detect Disk Pressure?

**Answer:**

> I check filesystem capacity, inode usage, growth, container/runtime storage, logs, and Kubernetes DiskPressure where applicable. I also investigate deleted-but-open files and log rotation when the reported disk usage doesn't match what I expect.

---

# 350. Interview — What Is the Difference Between Health and Readiness?

**Answer:**

> Health describes whether a component is operating correctly. Readiness describes whether it is currently capable of serving traffic. A service can be alive but not ready because it is still starting or cannot reach a required dependency.

---

# 351. Interview — Should a Health Check Restart a Failed Service?

**Answer:**

> Not by default. I prefer separating detection from remediation. If automatic remediation is explicitly required, it should be allowlisted, limited to a small number of attempts, audited, and followed by verification to avoid restart loops.

---

# 352. Interview — How Do You Secure a Health Checker?

**Answer:**

> I run it with least privilege, validate all inputs, avoid arbitrary shell commands and URLs, use timeouts, protect credentials, avoid secrets in logs, restrict network access, and keep dependencies controlled. If privileged checks are necessary, I isolate them into narrowly scoped operations.

---

# 353. Interview — How Do You Check TLS Health?

**Answer:**

> I validate DNS, TCP connectivity, the TLS handshake, certificate verification, and then the HTTP response. I use the default certificate-verifying SSL context rather than disabling certificate verification.

---

# 354. Interview — Why Can a Server Have Free Disk Space but Still Fail to Create Files?

**Answer:**

> It may have exhausted filesystem inodes. `df -h` measures capacity while `df -i` measures inode availability. File-heavy workloads can consume all inodes even when gigabytes of storage remain.

---

# 355. Interview — Why Can a Server Have Low Free Memory but Still Be Healthy?

**Answer:**

> Linux uses memory for filesystem caching, so free memory alone isn't a good health indicator. I look at available memory, swap activity, memory pressure, OOM events, and application behavior.

---

# 356. Interview — Why Can High Load Occur with Low CPU Usage?

**Answer:**

> Load includes tasks waiting on resources such as I/O, so a host can have relatively low CPU utilization while many processes are blocked on storage or another resource. I investigate I/O wait and disk/network behavior.

---

# 357. Interview — How Would You Build a Pre-Deployment Health Gate?

**Answer:**

> I would validate the environment, connectivity, disk and memory headroom, required services, ports, dependencies, artifact availability, and expected runtime versions. Only if critical preflight checks pass would I allow deployment.

---

# 358. Interview — How Would You Build a Post-Deployment Gate?

**Answer:**

> I would verify the service state, process or port, application health endpoint, expected version, dependency connectivity, and relevant logs. If critical validation fails, the deployment pipeline should stop and invoke the predefined rollback process.

---

# 359. Interview — How Do You Handle Health Check Failures Across a Fleet?

**Answer:**

> I aggregate results by host and severity, identify whether the failure is isolated or systemic, and correlate common dependencies. If all hosts fail simultaneously, I investigate shared infrastructure before debugging each machine independently.

---

# 360. Interview — What Is the Most Important Principle?

**Answer:**

> A health check should provide a reliable answer to a specific operational question. It should be fast, bounded, secure, actionable, and aligned with the actual architecture. A complicated health checker that produces noisy or misleading results is worse than a small set of trustworthy checks.

---

# 361. Real-World Scenario — Production Web Server

Architecture:

```text
Internet
   |
  ALB
   |
   v
EC2 Web Server
   |
   +-- nginx
   +-- application
   +-- local health endpoint
   |
   +-- database
```

Health checks:

```text
CPU
memory
disk
nginx
80/443
localhost health
database TCP
external HTTPS
```

---

# 362. Real-World Scenario — Deployment Validation

```text
Jenkins
  |
  v
Python preflight
  |
  +-- disk
  +-- memory
  +-- service
  +-- artifact
  |
  v
Deploy
  |
  v
Python postflight
  |
  +-- service
  +-- port
  +-- version
  +-- HTTP
  |
  v
Prometheus/Grafana
```

---

# 363. Real-World Scenario — EKS

```text
Git
 |
Argo CD
 |
EKS
 |
+-- nodes
+-- pods
+-- services
+-- ALB ingress
 |
+-- Prometheus
+-- Grafana
+-- ELK
```

Python can provide:

```text
preflight
external smoke tests
AWS checks
diagnostic collection
release validation
```

---

# 364. Real-World Scenario — Incident

Symptoms:

```text
Users report 503
```

Health checker:

```text
EC2 CPU        PASS
Memory         PASS
Disk           PASS
nginx          PASS
port 443       PASS
localhost HTTP FAIL
database TCP   PASS
```

This narrows the investigation toward:

```text
application
configuration
dependencies above TCP
```

rather than immediately blaming infrastructure.

---

# 365. Real-World Scenario — Disk Incident

Health:

```text
/ = 94%
inode = 51%
```

Investigation:

```text
du
 ↓
/var
 ↓
/var/log
 ↓
large application log
```

Then inspect:

```text
log rotation
application error rate
centralized logging
retention policy
```

Do not blindly delete the log.

---

# 366. Real-World Scenario — Memory Incident

Health:

```text
memory = 92%
swap = 45%
```

Then:

```text
check OOM events
check top processes
check recent deployment
check application metrics
check container limits
```

A health checker should surface the problem; root-cause analysis requires deeper investigation.

---

# 367. Real-World Scenario — Network Incident

Health:

```text
DNS PASS
TCP database FAIL
HTTP app PASS
```

This suggests:

```text
specific dependency path
```

Investigate:

```text
route
security group
NACL
database listener
network policy
```

rather than restarting the entire application.

---

# 368. Real-World Scenario — TLS Incident

Health:

```text
DNS PASS
TCP 443 PASS
TLS FAIL
```

Likely areas:

```text
certificate
hostname
trust chain
expiration
TLS configuration
```

This is much more precise than reporting:

```text
application down
```

---

# 369. Real-World Scenario — DNS Incident

Health:

```text
DNS FAIL
TCP unknown
HTTP skipped
```

The correct next step is:

```text
resolver investigation
```

not:

```text
restart application
```

---

# 370. Real-World Scenario — Application Version Drift

Fleet:

```text
app01 2.4.1
app02 2.4.1
app03 2.3.9
app04 2.4.1
```

The health checker should flag:

```text
app03 = version drift
```

This can explain inconsistent behavior across servers.

---

# 371. Real-World Scenario — Failed Deployment

```text
preflight PASS
deploy PASS
service PASS
HTTP FAIL
```

The deployment is not successful.

Next:

```text
logs
configuration
dependency
rollback
```

Do not report success just because the service process is active.

---

# 372. Real-World Scenario — Health Checker Failure

Suppose:

```text
Python dependency missing
```

The checker cannot run.

Correct result:

```text
CHECKER_ERROR
```

not:

```text
SYSTEM_PASS
```

Observability tools must distinguish system health from observer health.

---

# 373. Production Design Pattern

Use three layers:

```text
Detection
   |
   v
Diagnosis
   |
   v
Remediation
```

Example:

```text
Disk > 90%
   |
   v
Find filesystem/log growth
   |
   v
Apply approved cleanup/rotation
```

Do not combine all three blindly into one script.

---

# 374. Production Design Pattern — Read/Write Separation

Prefer:

```text
health-check
=
read-only
```

and:

```text
remediation
=
explicit change
```

This makes automation safer.

---

# 375. Production Design Pattern — Shared Library

Create reusable functions:

```text
check_cpu()
check_memory()
check_disk()
check_service()
check_port()
check_dns()
check_http()
```

Then compose different profiles:

```text
web
database
worker
kubernetes-node
```

---

# 376. Production Design Pattern — Role Profiles

Example:

```yaml
profiles:
  web:
    services:
      - nginx
    ports:
      - 80
      - 443

  worker:
    services:
      - worker
```

This avoids irrelevant checks.

---

# 377. Production Design Pattern — Environment Profiles

```yaml
environments:
  staging:
    disk_fail: 95

  production:
    disk_fail: 90
```

Configuration should be reviewed and version-controlled.

---

# 378. Production Design Pattern — Immutable Configuration

Prefer:

```text
Git
 ↓
review
 ↓
approved config
 ↓
deployment
```

rather than manually editing health-check thresholds on servers.

---

# 379. Production Design Pattern — Versioned Health Rules

Treat health rules like code.

```text
health-rules-v1
health-rules-v2
```

Changes should be:

```text
reviewed
tested
audited
```

---

# 380. Production Design Pattern — Observability First

Before writing a new health check, ask:

```text
Does Prometheus already measure this?
Does Grafana show it?
Does ELK contain the logs?
Does Kubernetes already expose the state?
Does AWS already expose the state?
```

Avoid duplicating existing functionality unnecessarily.

---

# 381. Production Design Pattern — Health Check Purpose

Every check should have a clear purpose:

```text
"Detect root filesystem exhaustion before deployment"
```

rather than:

```text
"Check disk because we can"
```

---

# 382. Production Design Pattern — Actionable Thresholds

A threshold should answer:

```text
What action occurs when this threshold is crossed?
```

If no action exists, the threshold may not be useful.

---

# 383. Production Design Pattern — Duration

For transient conditions:

```text
CPU > 90% for 10 seconds
```

may be different from:

```text
CPU > 90% for 30 minutes
```

Use duration where it improves signal quality.

---

# 384. Production Design Pattern — Hysteresis

Avoid flapping:

```text
WARN at 80%
recover at 79%
```

may cause frequent transitions.

Use different trigger/recovery thresholds when appropriate:

```text
alert > 90%
recover < 85%
```

---

# 385. Health State Machine

```text
PASS
 |
 | threshold crossed
 v
WARN
 |
 | critical condition
 v
FAIL
 |
 | recovery
 v
PASS
```

State transitions can reduce alert noise.

---

# 386. Recovery Checks

When a system recovers:

```text
health = PASS
```

send a recovery event only if the failure was previously active.

Do not continuously notify that everything is healthy.

---

# 387. Alert Deduplication

If:

```text
disk FAIL
```

for 30 minutes, avoid sending 30 identical alerts.

Alertmanager or another incident system should manage deduplication.

---

# 388. Health Check and SLO

For a service:

```text
availability
latency
error rate
```

are often more meaningful than host-level CPU.

Host health is supporting evidence, not the final user experience.

---

# 389. Golden Signals

Application monitoring commonly considers:

```text
Latency
Traffic
Errors
Saturation
```

A health checker should complement these signals rather than replace them.

---

# 390. Saturation

Examples:

```text
CPU saturation
memory pressure
disk I/O
connection pool
thread pool
queue depth
```

Saturation can reveal problems before complete failure.

---

# 391. Queue Health

For asynchronous workers:

```text
queue depth
consumer count
processing latency
failed messages
```

can be more useful than CPU.

---

# 392. Worker Health

A worker may be:

```text
process active
CPU low
memory normal
```

but not consuming messages.

Therefore check:

```text
queue connectivity
consumer status
processing activity
```

when required.

---

# 393. RabbitMQ Worker Example

Health:

```text
worker service PASS
RabbitMQ TCP PASS
queue depth WARN
processing latency FAIL
```

This gives a much better operational picture than service status alone.

---

# 394. Database-Backed Application

Health:

```text
app process PASS
HTTP PASS
database TCP PASS
database query FAIL
```

This indicates the application can reach the database but cannot perform the required operation.

---

# 395. Health Check Granularity

Do not create:

```text
one giant health check
```

Prefer:

```text
small independent checks
```

that can be combined.

This makes failures easier to understand.

---

# 396. Check Naming

Use stable names:

```text
cpu
memory
disk.root
disk.var
service.nginx
port.443
dns.database
http.health
tls.api
```

Stable names help dashboards and automation.

---

# 397. Health Result Metadata

Include:

```text
name
host
environment
role
timestamp
duration
status
value
threshold
message
```

This supports fleet-wide analysis.

---

# 398. Health Check Schema

Example:

```json
{
  "name": "disk.root",
  "host": "app01",
  "environment": "production",
  "status": "WARN",
  "value": 84.2,
  "threshold": 80,
  "duration_ms": 4,
  "message": "Root filesystem usage is above warning threshold"
}
```

---

# 399. Health Summary

Example:

```text
Host: app01
Role: web
Environment: production

PASS  CPU       42%
PASS  Memory    61%
WARN  Disk      84%
PASS  Nginx     active
PASS  Port 443
PASS  HTTPS     200
PASS  DNS

OVERALL: WARN
```

This is easy for engineers to understand.

---

# 400. Final Mental Model

```text
                    SYSTEM HEALTH
                          |
        +-----------------+-----------------+
        |                 |                 |
    Resources          Services          Network
        |                 |                 |
 CPU / Memory        systemd           DNS / TCP / TLS
 Disk / Inodes       Processes         Routes / Ports
 Swap / PIDs         Health API        Interfaces
        |                 |                 |
        +-----------------+-----------------+
                          |
                    Application
                          |
                Dependencies / Users
                          |
                    Health Result
                          |
             +------------+------------+
             |            |            |
           JSON          CI          Monitoring
```

The best Python health-check system is **small, deterministic, secure, actionable, and architecture-aware**.

It should tell a DevOps engineer not just that something is wrong, but **which layer is unhealthy, what evidence was collected, and what the next investigation step should be.**
