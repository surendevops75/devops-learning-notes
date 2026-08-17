# 04-Service-Management

## Python for Linux DevOps

Service management is a core Linux DevOps responsibility.

A DevOps engineer must understand how applications become services, how services start and stop, how dependencies work, how failures are investigated, and how automation can safely interact with service managers.

This chapter focuses primarily on:

```text
systemd
systemctl
journalctl
service lifecycle
service health
service dependencies
Python automation
production troubleshooting
CI/CD integration
```

The goal is not to replace `systemctl` with Python.

The goal is to use Python where custom automation, validation, orchestration, reporting, or integration is required.

---

# 1. What Is a Linux Service?

A service is a long-running process that provides functionality to other processes or users.

Examples:

```text
nginx
sshd
docker
containerd
kubelet
postgresql
redis
rabbitmq
jenkins
```

A service usually has:

```text
process
configuration
environment
user/group
ports
logs
dependencies
startup policy
health behavior
```

---

# 2. Service Management Layers

A typical Linux service stack:

```text
Application
    ↓
Process
    ↓
systemd service unit
    ↓
systemd
    ↓
Linux kernel
```

For DevOps troubleshooting:

```text
application
   ↓
service
   ↓
process
   ↓
system resources
```

You should be able to move through these layers during an incident.

---

# 3. systemd

`systemd` is the dominant service manager on many modern Linux distributions.

It manages:

```text
services
sockets
mounts
timers
devices
targets
paths
```

For services, the most common unit type is:

```text
.service
```

---

# 4. systemctl

The primary command for interacting with systemd is:

```bash
systemctl
```

Examples:

```bash
systemctl status nginx
systemctl start nginx
systemctl stop nginx
systemctl restart nginx
systemctl reload nginx
```

Python can invoke these commands when custom orchestration is required.

---

# 5. Check Service Status

```bash
systemctl status nginx
```

Important information includes:

```text
Loaded
Active
Main PID
Tasks
Memory
CPU
Recent logs
```

Python can automate status collection.

---

# 6. Start a Service

```bash
sudo systemctl start nginx
```

Start means:

```text
request systemd to start the service now
```

It does not necessarily configure the service to start after reboot.

---

# 7. Stop a Service

```bash
sudo systemctl stop nginx
```

Stopping a production service can cause downtime.

Automation should verify:

```text
correct target
change approval
service identity
maintenance window
```

when required.

---

# 8. Restart a Service

```bash
sudo systemctl restart nginx
```

Restart generally means:

```text
stop
↓
start
```

This can interrupt active connections.

Use reload when the application supports safe configuration reloads.

---

# 9. Reload a Service

```bash
sudo systemctl reload nginx
```

Reload asks the service to reload configuration without fully stopping it.

Not every service supports reload.

---

# 10. Restart vs Reload

```text
restart
  ↓
process usually stops and starts again

reload
  ↓
process remains running
  ↓
configuration is re-read
```

For production configuration changes:

```text
validate
↓
reload
↓
health check
```

is often preferable to an unnecessary restart.

---

# 11. Enable a Service

```bash
sudo systemctl enable nginx
```

This configures startup behavior for boot.

It does not necessarily start the service immediately.

---

# 12. Enable and Start

```bash
sudo systemctl enable --now nginx
```

This combines:

```text
enable
+
start
```

Useful during server provisioning.

---

# 13. Disable a Service

```bash
sudo systemctl disable nginx
```

This changes startup behavior.

It does not necessarily stop an already running service.

---

# 14. Disable and Stop

```bash
sudo systemctl disable --now nginx
```

This both:

```text
stops current service
+
prevents normal boot activation
```

Use carefully in production.

---

# 15. Mask a Service

```bash
sudo systemctl mask nginx
```

Masking prevents the service from being started normally, including dependency-based starts.

Unmask:

```bash
sudo systemctl unmask nginx
```

Masking is stronger than disabling.

---

# 16. Service States

Common systemd states:

```text
active
inactive
failed
activating
deactivating
```

A service may also appear:

```text
active (running)
active (exited)
failed
inactive (dead)
```

Interpret the state together with logs and process information.

---

# 17. Active Does Not Always Mean Healthy

A service can show:

```text
active (running)
```

while the application is unhealthy.

For example:

```text
process alive
but
HTTP endpoint returns 500
```

Therefore service health should include:

```text
process
port
application endpoint
logs
metrics
dependencies
```

---

# 18. Failed State

Check:

```bash
systemctl status myapp
```

Then:

```bash
journalctl -u myapp
```

Look for:

```text
configuration errors
permission errors
missing files
dependency failures
port conflicts
environment problems
crashes
```

---

# 19. List Services

```bash
systemctl list-units --type=service
```

To include inactive services:

```bash
systemctl list-units \
    --type=service \
    --all
```

---

# 20. Failed Services

```bash
systemctl --failed
```

This is an excellent first command during host-level incidents.

---

# 21. Service Unit Files

A service definition commonly looks like:

```ini
[Unit]
Description=My Application
After=network-online.target

[Service]
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/.venv/bin/python /opt/myapp/app.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

---

# 22. `[Unit]`

The `[Unit]` section defines:

```text
description
dependencies
ordering
conditions
conflicts
```

Example:

```ini
[Unit]
Description=My Application
After=network-online.target
```

---

# 23. `After=`

Example:

```ini
After=network-online.target
```

This controls ordering.

It does not automatically mean:

```text
network-online.target must successfully start
```

For dependency requirements, understand the difference between ordering and requirement relationships.

---

# 24. `Requires=`

```ini
Requires=redis.service
```

This establishes a dependency relationship.

If the required unit becomes unavailable, systemd can affect the dependent unit according to the dependency semantics.

---

# 25. `Wants=`

```ini
Wants=redis.service
```

This expresses a weaker relationship than `Requires=`.

Conceptually:

```text
Requires -> stronger dependency
Wants    -> preferred dependency
```

Use dependencies carefully to avoid unnecessary coupling.

---

# 26. `Before=`

```ini
Before=myapp.service
```

This defines ordering:

```text
this service
   ↓
myapp
```

It does not by itself establish that both services must run.

---

# 27. `After=` vs `Requires=`

Important interview distinction:

```text
After=
```

means:

```text
ordering
```

while:

```text
Requires=
```

means:

```text
dependency
```

You may use both:

```ini
Requires=redis.service
After=redis.service
```

when the application truly requires Redis and must start after it.

---

# 28. `[Service]`

The `[Service]` section defines how the process runs.

Common settings:

```text
User
Group
WorkingDirectory
ExecStart
ExecStop
Environment
EnvironmentFile
Restart
RestartSec
TimeoutStartSec
TimeoutStopSec
LimitNOFILE
```

---

# 29. `User=`

```ini
User=myapp
```

This runs the service as:

```text
myapp
```

rather than root.

Least privilege is an important production practice.

---

# 30. `Group=`

```ini
Group=myapp
```

This establishes the primary group for the service process.

Permissions on:

```text
files
directories
sockets
devices
```

must match the service identity.

---

# 31. WorkingDirectory

```ini
WorkingDirectory=/opt/myapp
```

This prevents scripts from depending on an unexpected current directory.

This is especially important for Python applications using relative paths.

---

# 32. ExecStart

Example:

```ini
ExecStart=/opt/myapp/.venv/bin/python /opt/myapp/app.py
```

Prefer an explicit interpreter when using a virtual environment.

Do not assume:

```text
python
```

points to the desired version.

---

# 33. ExecStart and Shell Syntax

`ExecStart=` is not automatically interpreted like:

```bash
bash -c
```

Do not write complex shell expressions expecting normal shell expansion.

If shell behavior is genuinely required, explicitly invoke an appropriate shell and understand the security implications.

---

# 34. ExecStop

Example:

```ini
ExecStop=/opt/myapp/bin/stop.sh
```

Often systemd can terminate the main process without a custom stop command.

Use custom stop logic only when needed.

---

# 35. Environment Variables

```ini
Environment="APP_ENV=production"
Environment="PORT=8080"
```

Environment variables can also be loaded from:

```ini
EnvironmentFile=/etc/myapp/myapp.env
```

Protect files containing secrets.

---

# 36. EnvironmentFile

Example:

```ini
EnvironmentFile=/etc/myapp/myapp.env
```

The file might contain:

```text
APP_ENV=production
PORT=8080
```

Avoid putting passwords and tokens into world-readable files.

---

# 37. Service Environment Debugging

A common problem:

```text
works manually
fails under systemd
```

Check:

```text
PATH
HOME
WorkingDirectory
User
Group
Environment
EnvironmentFile
Python virtual environment
permissions
```

The systemd environment is not necessarily the same as your shell environment.

---

# 38. Restart Policy

```ini
Restart=on-failure
```

Other policies include:

```text
no
on-success
on-failure
on-abnormal
on-abort
always
```

Choose according to application behavior.

---

# 39. `Restart=always`

```ini
Restart=always
```

This can be useful for continuously running services.

But it can also hide a persistent startup failure by creating a restart loop.

Combine restart behavior with:

```text
logs
rate limiting
health checks
alerting
```

---

# 40. RestartSec

```ini
RestartSec=5
```

This waits before restarting.

Useful for avoiding an immediate tight restart loop.

---

# 41. Start Limit

systemd can limit repeated restart attempts.

Inspect:

```bash
systemctl status myapp
```

If a service repeatedly fails, you may see a message indicating it hit a start-limit condition.

The correct response is to fix the underlying failure rather than repeatedly restarting.

---

# 42. TimeoutStartSec

```ini
TimeoutStartSec=60
```

Controls how long systemd waits during startup.

If the application legitimately takes longer to start, the timeout may need adjustment.

Do not simply increase it without understanding why startup is slow.

---

# 43. TimeoutStopSec

```ini
TimeoutStopSec=30
```

Controls graceful shutdown waiting time.

A well-behaved application should handle termination signals correctly.

---

# 44. Signal Handling

systemd commonly uses:

```text
SIGTERM
```

for graceful termination.

Applications should:

```text
receive signal
stop accepting work
finish safe work
close resources
exit
```

Python applications should handle signals appropriately when graceful shutdown matters.

---

# 45. Python Signal Handling

```python
import signal
import sys


def shutdown(signum, frame):
    print("Shutdown requested")
    sys.exit(0)


signal.signal(
    signal.SIGTERM,
    shutdown,
)
```

Production applications should coordinate shutdown with:

```text
worker threads
child processes
network connections
queues
temporary files
```

---

# 46. Graceful Shutdown

A good service shutdown:

```text
SIGTERM
   ↓
stop accepting requests
   ↓
finish safe work
   ↓
close connections
   ↓
flush logs
   ↓
exit
```

A forced:

```text
SIGKILL
```

does not allow normal cleanup.

---

# 47. `KillMode=`

systemd controls process termination behavior with settings such as:

```ini
KillMode=control-group
```

This matters when a service creates child processes.

Understand whether systemd should terminate:

```text
main process
all processes in cgroup
```

according to application requirements.

---

# 48. Process Tracking

systemd tracks service processes using cgroups.

This is important because an application can spawn:

```text
worker
child
subprocess
```

and systemd can still manage the service as a group.

---

# 49. Cgroups

A service's cgroup can provide resource controls such as:

```text
CPU
memory
tasks/process count
I/O
```

This connects service management with Linux resource management.

---

# 50. Memory Limits

Modern systemd can configure memory controls.

For example:

```ini
MemoryMax=1G
```

If a service exceeds its configured memory limit, it may be terminated according to systemd/kernel behavior.

This should be coordinated with application sizing.

---

# 51. CPU Limits

systemd supports CPU/resource controls.

For example, appropriate CPU accounting and quota controls can constrain service resources.

Do not impose restrictive CPU limits without measuring application requirements.

---

# 52. File Descriptor Limits

A service may need many open files.

Example:

```ini
LimitNOFILE=65535
```

Useful for:

```text
web servers
proxies
databases
high-connection applications
```

But increasing limits does not fix an application leak.

---

# 53. Process Limits

A service can hit process/thread limits.

Investigate:

```text
TasksMax
ulimit
cgroup task limits
application worker configuration
```

A sudden increase in worker processes may indicate a runaway condition.

---

# 54. Service Dependencies

A microservice may depend on:

```text
network
DNS
database
cache
message broker
filesystem
certificate
secret
```

Systemd can model host-level dependencies, but application-level health still requires application checks.

---

# 55. Dependency Ordering

Example:

```text
network
  ↓
database
  ↓
application
```

Do not assume:

```text
database process started
=
database ready
```

A service may need:

```text
port available
authentication ready
schema ready
```

before the application can work.

---

# 56. Service Readiness

systemd service startup and application readiness are different concepts.

Example:

```text
systemd: active
application: listening
database: unavailable
API: unhealthy
```

A production health check should test the actual dependency chain.

---

# 57. Type=`simple`

Example:

```ini
[Service]
Type=simple
ExecStart=/opt/myapp/bin/start
```

The process started by `ExecStart` is treated as the main process.

This is common for modern foreground applications.

---

# 58. Type=`exec`

On supported systemd versions:

```ini
Type=exec
```

helps ensure execution has actually occurred before startup is considered successful.

Understand the systemd version available on your distribution.

---

# 59. Type=`forking`

Older daemon-style applications may:

```text
start
fork
parent exits
child continues
```

For those applications:

```ini
Type=forking
```

may be appropriate.

Modern applications should generally stay in the foreground when practical.

---

# 60. Type=`oneshot`

For a task that runs and exits:

```ini
[Service]
Type=oneshot
ExecStart=/opt/scripts/backup.py
```

Useful for:

```text
maintenance
migration
initialization
one-time jobs
```

Timers can schedule such services.

---

# 61. Type=`notify`

Some applications can notify systemd when they are ready.

Conceptually:

```text
process starts
   ↓
initialization
   ↓
READY notification
   ↓
systemd considers service ready
```

This is useful for applications with meaningful initialization stages.

---

# 62. Service Unit Location

Common locations include:

```text
/etc/systemd/system
/usr/lib/systemd/system
/lib/systemd/system
```

Distribution layout can vary.

Locally managed overrides commonly belong under:

```text
/etc/systemd/system
```

---

# 63. `systemctl cat`

Use:

```bash
systemctl cat myapp
```

This shows the unit file and applicable drop-ins.

Excellent for debugging configuration.

---

# 64. `systemctl show`

```bash
systemctl show myapp
```

This exposes structured properties.

You can query:

```bash
systemctl show myapp \
    -p ActiveState \
    -p SubState \
    -p MainPID
```

Python can parse this output.

---

# 65. `systemctl is-active`

```bash
systemctl is-active nginx
```

Possible result:

```text
active
```

This is convenient for automation.

---

# 66. `systemctl is-enabled`

```bash
systemctl is-enabled nginx
```

Useful when provisioning or auditing boot configuration.

---

# 67. `systemctl is-failed`

```bash
systemctl is-failed nginx
```

Can be used in automation to detect failure state.

---

# 68. `systemctl daemon-reload`

After changing a unit file:

```bash
sudo systemctl daemon-reload
```

is required for systemd to recognize the updated unit definition.

It does not restart the service.

---

# 69. Common Unit Change Workflow

```text
edit unit
   ↓
systemctl daemon-reload
   ↓
systemctl restart/reload
   ↓
systemctl status
   ↓
journalctl
   ↓
health check
```

Do not forget `daemon-reload` after changing unit definitions.

---

# 70. Drop-In Overrides

Instead of modifying vendor unit files:

```bash
systemctl edit myapp
```

creates a drop-in override.

This is preferred for local customizations because package updates can replace vendor files.

---

# 71. Drop-In Example

```ini
[Service]
Environment="APP_ENV=production"
Restart=on-failure
RestartSec=5
```

Then:

```bash
systemctl daemon-reload
systemctl restart myapp
```

---

# 72. Why Drop-Ins Are Better

Benefits:

```text
package-safe
easy to audit
small changes
clear separation
upgrade-friendly
```

Avoid editing distribution-managed unit files directly unless there is a specific reason.

---

# 73. Service Logs

Use:

```bash
journalctl -u myapp
```

For recent logs:

```bash
journalctl -u myapp -n 100
```

Follow:

```bash
journalctl -u myapp -f
```

---

# 74. Logs Since Boot

```bash
journalctl -u myapp -b
```

Useful when the problem started after reboot.

Previous boot:

```bash
journalctl -u myapp -b -1
```

---

# 75. Logs by Time

```bash
journalctl \
    -u myapp \
    --since "1 hour ago"
```

This reduces noise during incident investigation.

---

# 76. Kernel Logs

Service failures may be caused by kernel events.

Check:

```bash
journalctl -k
```

Examples:

```text
OOM
I/O errors
filesystem issues
network issues
driver failures
```

---

# 77. Service Process

Find the main PID:

```bash
systemctl show myapp \
    -p MainPID
```

Then:

```bash
ps -fp <PID>
```

This connects:

```text
systemd
→ process
```

---

# 78. Service Port

Check:

```bash
ss -lntp
```

Then confirm:

```text
service process
port
IP binding
```

A service can be:

```text
active
but listening on wrong interface
```

---

# 79. Local Health Check

Example:

```bash
curl -f http://127.0.0.1:8080/health
```

Python:

```python
import urllib.request

with urllib.request.urlopen(
    "http://127.0.0.1:8080/health",
    timeout=5,
) as response:
    print(response.status)
```

Do not confuse process health with application health.

---

# 80. Service Health Layers

A strong health check can be:

```text
systemd active
      ↓
process exists
      ↓
port listening
      ↓
HTTP endpoint works
      ↓
dependency check
      ↓
business metric healthy
```

The deeper checks should match the service.

---

# 81. Python `subprocess`

Python can run system commands:

```python
import subprocess

result = subprocess.run(
    ["systemctl", "is-active", "nginx"],
    capture_output=True,
    text=True,
)

print(result.stdout)
```

Prefer argument lists over shell strings.

---

# 82. `check=True`

```python
subprocess.run(
    ["systemctl", "restart", "nginx"],
    check=True,
)
```

If the command fails, Python raises `CalledProcessError`.

This prevents silent failures.

---

# 83. Capture Output

```python
result = subprocess.run(
    [
        "systemctl",
        "is-active",
        "nginx",
    ],
    capture_output=True,
    text=True,
)

print(
    result.stdout.strip()
)
```

Useful for automation decisions.

---

# 84. Avoid `shell=True`

Avoid:

```python
subprocess.run(
    f"systemctl restart {service}",
    shell=True,
)
```

Especially when `service` can come from user input.

Prefer:

```python
subprocess.run(
    ["systemctl", "restart", service],
    check=True,
)
```

Still validate the service name against an allowlist when appropriate.

---

# 85. Service Allowlist

For an automation tool:

```python
ALLOWED = {
    "nginx",
    "myapp",
    "worker",
}
```

Then:

```python
if service not in ALLOWED:
    raise ValueError(
        "Unsupported service"
    )
```

This prevents arbitrary service manipulation.

---

# 86. Python Service Status Helper

```python
import subprocess


def service_active(name):
    result = subprocess.run(
        [
            "systemctl",
            "is-active",
            name,
        ],
        capture_output=True,
        text=True,
    )

    return (
        result.returncode == 0
        and result.stdout.strip()
        == "active"
    )
```

---

# 87. Python Service Restart Helper

```python
import subprocess


def restart_service(name):
    subprocess.run(
        [
            "systemctl",
            "restart",
            name,
        ],
        check=True,
    )
```

Add:

```text
allowlist
logging
timeout
post-restart verification
```

for production.

---

# 88. Timeout

```python
subprocess.run(
    [
        "systemctl",
        "restart",
        "myapp",
    ],
    check=True,
    timeout=60,
)
```

A timeout prevents an automation process from waiting forever.

---

# 89. Service Verification

```python
def verify_service(name):
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

    return (
        result.stdout.strip()
        == "active"
    )
```

Restart should not be considered successful until verification passes.

---

# 90. Restart and Health Check

```python
restart_service(
    "myapp"
)

if not verify_service(
    "myapp"
):
    raise RuntimeError(
        "Service unhealthy"
    )
```

A stronger implementation should also test the application endpoint.

---

# 91. Python Service Orchestrator

Conceptually:

```text
validate
   ↓
stop
   ↓
update configuration
   ↓
validate configuration
   ↓
start
   ↓
systemd status
   ↓
port check
   ↓
HTTP health
   ↓
report
```

This is more reliable than simply running:

```text
restart
```

---

# 92. Service Configuration Backup

Before changing:

```text
/etc/myapp/app.conf
```

a Python tool can:

```text
timestamp
copy
checksum
record backup
```

Then update the configuration.

---

# 93. Configuration Validation

Example for Nginx:

```python
subprocess.run(
    ["nginx", "-t"],
    check=True,
)
```

Only after validation:

```python
subprocess.run(
    ["systemctl", "reload", "nginx"],
    check=True,
)
```

---

# 94. Generic Service Reload Pattern

```python
validate_config()

subprocess.run(
    ["systemctl", "reload", service],
    check=True,
)

if not verify_service(service):
    raise RuntimeError(
        "Reload verification failed"
    )
```

---

# 95. Service Restart Loop

Symptoms:

```text
systemctl status -> failed
journalctl -> repeated starts
Main PID changes rapidly
```

Check:

```text
configuration
dependencies
permissions
port
environment
executable
resource limits
application logs
```

Do not solve a restart loop by increasing `RestartSec` alone.

---

# 96. Service Start Failure

Typical workflow:

```bash
systemctl status myapp
journalctl -u myapp -b
systemctl cat myapp
systemctl show myapp
```

Then inspect:

```text
ExecStart
User
WorkingDirectory
Environment
permissions
dependencies
```

---

# 97. Exit Code

Systemd status can show process exit information.

Example:

```text
status=1/FAILURE
```

The numeric exit code is a clue, not the entire diagnosis.

Application logs are usually required.

---

# 98. Permission Failure

Symptoms:

```text
Permission denied
```

Investigate:

```text
User=
Group=
file ownership
directory traversal
SELinux/AppArmor
environment
```

Python may reproduce the issue under the same service identity.

---

# 99. Working Directory Failure

If:

```ini
WorkingDirectory=/opt/myapp
```

does not exist, startup can fail.

Check:

```bash
ls -ld /opt/myapp
```

This is a common cause of:

```text
works manually
fails under systemd
```

---

# 100. Python Virtual Environment Failure

Unit:

```ini
ExecStart=/opt/myapp/.venv/bin/python /opt/myapp/app.py
```

Check:

```bash
ls -l /opt/myapp/.venv/bin/python
```

and dependencies:

```bash
/opt/myapp/.venv/bin/python -m pip list
```

Do not rely on interactive shell activation.

---

# 101. Port Conflict

Service fails because another process owns the port.

Check:

```bash
ss -lntp | grep :8080
```

Then:

```text
identify PID
identify service
determine ownership
```

Do not kill another production process without understanding the impact.

---

# 102. Address Binding Failure

Application may bind:

```text
127.0.0.1:8080
```

while clients expect:

```text
0.0.0.0:8080
```

Check:

```bash
ss -lntp
```

and application configuration.

---

# 103. DNS Dependency

A service can start but fail requests because DNS is unavailable.

Investigate:

```bash
resolvectl status
getent hosts database.example
```

Then:

```text
network
DNS
route
firewall
dependency
```

---

# 104. Network Dependency

A service may require:

```text
network-online.target
```

But startup ordering alone does not guarantee external dependency readiness.

Use application-level retries and health checks where appropriate.

---

# 105. Database Dependency

Bad assumption:

```text
postgres.service active
=
database ready for application
```

Application should handle:

```text
connection retry
timeout
authentication
schema readiness
```

Systemd can provide basic host-level ordering but not replace application resilience.

---

# 106. Service Dependencies in Microservices

On a Linux host:

```text
API
 ↓
Redis
 ↓
PostgreSQL
```

But in a modern DevOps environment, services may instead be:

```text
EKS
RDS
ElastiCache
managed messaging
```

Systemd is mainly relevant to host-level components and self-managed workloads.

---

# 107. Docker Service

On a Docker host:

```bash
systemctl status docker
```

If Docker fails:

```text
containers may stop or become unavailable
```

Investigate:

```text
Docker logs
systemd journal
disk
filesystem
containerd
network
```

---

# 108. containerd Service

Kubernetes worker nodes commonly depend on:

```text
containerd
```

Check:

```bash
systemctl status containerd
```

and:

```bash
journalctl -u containerd
```

A container runtime issue can cause broad Kubernetes node problems.

---

# 109. kubelet Service

On Kubernetes nodes:

```bash
systemctl status kubelet
```

Useful logs:

```bash
journalctl -u kubelet
```

A kubelet failure can affect:

```text
Pod lifecycle
node readiness
container management
health reporting
```

---

# 110. EKS Context

In managed EKS, AWS manages the control plane, but worker nodes still have host-level components such as:

```text
kubelet
container runtime
networking components
```

The exact node architecture depends on:

```text
EC2 managed nodes
self-managed nodes
Bottlerocket
Fargate
```

Use the platform's supported troubleshooting approach.

---

# 111. Service Management and Kubernetes

When troubleshooting a Pod:

```text
Pod
 ↓
container
 ↓
node
 ↓
kubelet
 ↓
container runtime
 ↓
kernel
```

If the problem is node-level, systemd becomes relevant.

---

# 112. Kubelet Troubleshooting

Typical workflow:

```bash
systemctl status kubelet
journalctl -u kubelet -n 200
```

Then inspect:

```text
node status
container runtime
disk pressure
memory pressure
network
certificates
configuration
```

---

# 113. Container Runtime Troubleshooting

For containerd:

```bash
systemctl status containerd
journalctl -u containerd
```

Then:

```bash
crictl info
crictl ps
```

where the environment supports `crictl`.

---

# 114. Service Certificates

Host services may depend on:

```text
TLS certificate
private key
CA bundle
```

A service can fail after certificate rotation because:

```text
wrong path
wrong permissions
expired certificate
invalid chain
wrong owner
```

---

# 115. Certificate Rotation Workflow

```text
new certificate
   ↓
validate certificate
   ↓
validate permissions
   ↓
backup current
   ↓
replace atomically
   ↓
test configuration
   ↓
reload service
   ↓
health check
```

---

# 116. Service Health Script

A useful Python script can report:

```text
service active
main PID
restart count
port
health endpoint
recent error logs
```

This creates a lightweight operational diagnostic.

---

# 117. Service Status JSON

Example conceptual output:

```json
{
  "service": "myapp",
  "active": true,
  "main_pid": 1820,
  "port": 8080,
  "health": "ok"
}
```

This can be consumed by:

```text
CI/CD
monitoring
automation
incident tooling
```

---

# 118. Python Service Health Reporter

```python
import subprocess


def systemd_property(
    service,
    property_name,
):
    result = subprocess.run(
        [
            "systemctl",
            "show",
            service,
            "-p",
            property_name,
        ],
        capture_output=True,
        text=True,
        check=True,
    )

    return result.stdout.strip()
```

---

# 119. Main PID

```python
pid = systemd_property(
    "myapp",
    "MainPID",
)

print(pid)
```

This connects service state to process-level troubleshooting.

---

# 120. Active State

```python
active = systemd_property(
    "myapp",
    "ActiveState",
)

print(active)
```

Possible:

```text
active
inactive
failed
activating
deactivating
```

---

# 121. Sub State

```python
sub = systemd_property(
    "myapp",
    "SubState",
)

print(sub)
```

For example:

```text
running
dead
exited
failed
```

---

# 122. Service Uptime

systemd properties can provide activation timestamps.

For operational reporting:

```text
start time
last restart
uptime
```

can be calculated from service metadata.

---

# 123. Restart Count

Depending on the service and systemd properties available, inspect:

```text
NRestarts
```

or related status information.

A high restart count is an important incident signal.

---

# 124. Service Failure Detection

Python can check:

```python
if not service_active(
    "myapp"
):
    alert()
```

But avoid implementing a complete monitoring platform if Prometheus or another monitoring system already provides service monitoring.

---

# 125. Service Monitoring Architecture

A common architecture:

```text
systemd
   ↓
service state
   ↓
node exporter / application exporter
   ↓
Prometheus
   ↓
Grafana
   ↓
Alerting
```

Python can complement this with custom checks.

---

# 126. Avoid Polling Loops

Bad:

```python
while True:
    check_service()
    time.sleep(1)
```

This can duplicate monitoring systems.

Prefer:

```text
Prometheus
systemd monitoring
health checks
event-driven automation
```

for continuous monitoring.

---

# 127. Python as an Operational Tool

Good uses:

```text
preflight checks
deployment orchestration
custom validation
multi-step remediation
report generation
inventory
configuration transformation
```

Poor use:

```text
replacing systemd
continuous polling without need
generic shell wrapper for every command
```

---

# 128. Service Remediation

A remediation script may:

```text
detect failed service
   ↓
collect diagnostics
   ↓
check known-safe condition
   ↓
restart if policy allows
   ↓
verify
   ↓
alert if still unhealthy
```

Automatic remediation should have strict boundaries.

---

# 129. Remediation Guardrails

Before automatic restart:

```text
Is restart safe?
Is deployment active?
Is maintenance running?
Is failure transient?
Has restart limit been reached?
Could restart cause data loss?
```

Automation should not turn one incident into a larger outage.

---

# 130. Restart Storm

If automation repeatedly restarts a service:

```text
failure
 ↓
restart
 ↓
failure
 ↓
restart
 ↓
resource pressure
 ↓
larger outage
```

Use:

```text
rate limits
backoff
restart counters
alerting
```

---

# 131. Backoff

A remediation tool can use:

```text
attempt 1 -> immediate
attempt 2 -> 10 sec
attempt 3 -> 30 sec
attempt 4 -> stop
```

Do not retry forever.

---

# 132. Circuit Breaker Concept

If repeated service recovery fails:

```text
stop automatic remediation
      ↓
alert human/operator
```

This prevents runaway automation.

---

# 133. Production Service Restart Policy

A mature policy may define:

```text
1 restart -> automatic
2 restarts -> automatic with delay
3 failures -> alert
persistent failure -> manual investigation
```

Actual values depend on service criticality.

---

# 134. Service Maintenance Mode

Before planned changes:

```text
disable alerts
or
silence appropriate alerts
      ↓
change
      ↓
validate
      ↓
restore alerting
```

Do not simply stop monitoring the service indefinitely.

---

# 135. Service Change Workflow

```text
ticket/change
   ↓
precheck
   ↓
backup
   ↓
change
   ↓
daemon-reload if unit changed
   ↓
config validation
   ↓
reload/restart
   ↓
health check
   ↓
monitor
   ↓
close change
```

---

# 136. Service Rollback

If a unit change breaks startup:

```text
restore previous unit/drop-in
   ↓
daemon-reload
   ↓
restart
   ↓
verify
```

Keep configuration changes version-controlled.

---

# 137. Version Control Service Units

Store custom unit files or templates in Git where organizational policy permits.

Repository:

```text
systemd/
├── myapp.service
├── myapp.env.example
└── README.md
```

Changes can then be:

```text
reviewed
tested
audited
rolled back
```

---

# 138. Configuration as Code

For repeatable hosts:

```text
Git
 ↓
Ansible/Terraform/cloud-init
 ↓
systemd unit
 ↓
service
```

Python can provide specialized validation or deployment helpers.

---

# 139. Ansible and systemd

Ansible can manage:

```yaml
- name: Ensure myapp is running
  ansible.builtin.systemd:
    name: myapp
    state: started
    enabled: true
```

Use configuration management for repeatable desired state.

Python is valuable when custom application logic is required.

---

# 140. CI/CD and systemd

A traditional VM deployment:

```text
Git push
 ↓
CI build
 ↓
test
 ↓
security scan
 ↓
artifact
 ↓
deploy to VM
 ↓
update release
 ↓
systemctl reload/restart
 ↓
health check
```

Python can implement custom deployment verification.

---

# 141. Jenkins Service Deployment

Jenkins may execute:

```bash
sudo systemctl restart myapp
```

But direct unrestricted sudo access is dangerous.

Prefer:

```text
dedicated deployment user
restricted sudoers rule
approved commands
audit logging
```

---

# 142. Restricted sudo

Conceptually:

```text
deploy-user
   ↓
sudo
   ↓
only allowed:
systemctl restart myapp
systemctl status myapp
```

Do not grant:

```text
sudo ALL=(ALL) NOPASSWD: ALL
```

just to make deployment easier.

---

# 143. Service Security

A secure unit can use systemd hardening features where compatible:

```text
NoNewPrivileges
PrivateTmp
ProtectSystem
ProtectHome
RestrictAddressFamilies
CapabilityBoundingSet
ReadWritePaths
```

These should be tested against application requirements.

---

# 144. `NoNewPrivileges`

```ini
NoNewPrivileges=true
```

Prevents a process and its descendants from gaining additional privileges through certain mechanisms.

Useful as part of defense in depth.

---

# 145. `PrivateTmp`

```ini
PrivateTmp=true
```

Can isolate the service's temporary directory view.

Check application compatibility before enabling.

---

# 146. `ProtectSystem`

Example:

```ini
ProtectSystem=full
```

or stronger variants depending on the desired protection.

This can make system paths read-only to the service.

---

# 147. `ProtectHome`

```ini
ProtectHome=true
```

Can restrict access to:

```text
/home
/root
/run/user
```

depending on configuration.

Useful when services do not need user home directories.

---

# 148. Read-Only Service Filesystem

A hardened service can use:

```text
read-only system paths
+
specific writable directories
```

This reduces the blast radius of compromise.

---

# 149. Writable Paths

If the application requires:

```text
/var/lib/myapp
```

you can design the service sandbox so only required paths are writable.

Principle:

```text
minimum writable surface
```

---

# 150. Capability Management

Linux capabilities can grant specific privileges without full root.

systemd can control capabilities with:

```ini
CapabilityBoundingSet=
```

Do not add capabilities unless the application requires them.

---

# 151. Network Restrictions

systemd service hardening can restrict network families.

Example concept:

```ini
RestrictAddressFamilies=AF_INET AF_INET6
```

Use only after understanding application behavior.

---

# 152. Service Hardening Workflow

```text
baseline service
   ↓
identify required access
   ↓
enable one hardening control
   ↓
test
   ↓
inspect logs
   ↓
enable next control
   ↓
document
```

Do not enable every security option blindly.

---

# 153. Service Dependencies and Security

A service may depend on:

```text
secret file
certificate
socket
database
network
```

Every dependency expands the operational and security surface.

Minimize unnecessary dependencies.

---

# 154. Socket Activation

systemd can activate services through sockets.

Concept:

```text
client
 ↓
socket
 ↓
systemd
 ↓
service starts
```

This can reduce idle resource usage for suitable services.

---

# 155. Timer Units

systemd timers can schedule jobs.

Example:

```text
backup.service
      ↑
backup.timer
```

This can replace cron for many Linux workloads.

---

# 156. Timer Status

```bash
systemctl list-timers
```

Check:

```text
NEXT
LAST
PASSED
UNIT
ACTIVATES
```

Useful during scheduled-job troubleshooting.

---

# 157. Python + systemd Timer

A Python script can be run by:

```ini
[Service]
Type=oneshot
ExecStart=/opt/tools/.venv/bin/python /opt/tools/backup.py
```

Timer:

```ini
[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true
```

This creates a native Linux scheduling model.

---

# 158. `Persistent=true`

For timers, persistence can allow a missed run to be triggered after the system returns, subject to timer semantics.

Useful for:

```text
maintenance
backups
reports
cleanup
```

---

# 159. Timer vs Cron

systemd timers provide:

```text
journal integration
dependency handling
service lifecycle
resource controls
status visibility
```

Cron remains useful and widespread.

Choose the scheduler that fits the environment.

---

# 160. Service and Timer Relationship

A timer should normally trigger a service:

```text
backup.timer
      ↓
backup.service
      ↓
Python script
```

This separates:

```text
when to run
```

from:

```text
what to run
```

---

# 161. Service Failure Notification

A service failure can trigger:

```text
monitoring alert
log pipeline
incident automation
```

Do not rely solely on a local Python script running on the same failed host.

External monitoring is more reliable.

---

# 162. External Health Monitoring

Better architecture:

```text
Host/systemd
   ↓
metrics/logs
   ↓
Prometheus/ELK
   ↓
alert
   ↓
incident response
```

Python remediation can be an additional layer.

---

# 163. Service Health vs Monitoring

Monitoring asks:

```text
Is service healthy?
```

Service management asks:

```text
How should service be started/stopped?
```

Incident response asks:

```text
Why is it unhealthy?
```

Keep these responsibilities conceptually separate.

---

# 164. Service Troubleshooting Framework

Use:

```text
1. Confirm symptom
2. Check systemd state
3. Check recent logs
4. Check process
5. Check resources
6. Check ports
7. Check dependencies
8. Check configuration
9. Reproduce safely
10. Remediate
11. Verify
12. Document
```

---

# 165. Production Scenario — Service Failed After Reboot

Check:

```bash
systemctl status myapp
systemctl is-enabled myapp
journalctl -u myapp -b
```

Then verify:

```text
dependencies
mounts
network
secrets
certificates
working directory
permissions
```

---

# 166. Production Scenario — Service Works Manually but Not Under systemd

Likely differences:

```text
PATH
HOME
working directory
user
group
environment
Python virtualenv
permissions
```

Compare the manual command with `ExecStart`.

---

# 167. Production Scenario — Service Starts Then Dies

Check:

```bash
systemctl status myapp
journalctl -u myapp -b
```

Then inspect:

```text
exit code
application exception
missing dependency
configuration
resource limits
```

---

# 168. Production Scenario — Service Restarts Every Few Seconds

Check:

```text
Restart=
RestartSec=
start-limit
application logs
exit code
dependency failures
```

Then fix the root cause.

---

# 169. Production Scenario — Service Is Active but API Is Down

Check:

```text
MainPID
listening ports
binding address
application health endpoint
upstream dependency
firewall
load balancer
```

`active (running)` alone is not sufficient.

---

# 170. Production Scenario — Service Reload Fails

Check:

```text
configuration syntax
permissions
certificate
environment
application reload support
journal logs
```

If reload is unsupported, use the service's documented restart mechanism.

---

# 171. Production Scenario — Port Already in Use

Workflow:

```bash
ss -lntp | grep :8080
```

Then:

```text
identify process
identify owner
identify expected service
```

Do not kill the process blindly.

---

# 172. Production Scenario — Unit File Changed but systemd Ignores It

Likely missing:

```bash
systemctl daemon-reload
```

Then:

```bash
systemctl restart myapp
```

Verify with:

```bash
systemctl cat myapp
```

---

# 173. Production Scenario — Package Upgrade Breaks Custom Unit

If the custom unit was edited under a vendor-managed directory, package upgrades may overwrite it.

Use:

```text
/etc/systemd/system
```

or drop-ins for local customizations where appropriate.

---

# 174. Production Scenario — Service Cannot Access Config

Check:

```text
User
Group
file permissions
parent directory permissions
SELinux/AppArmor
```

Use:

```bash
namei -l /path/to/config
```

---

# 175. Production Scenario — Service Cannot Access Certificate

Check:

```text
certificate path
private-key path
owner
group
permissions
expiry
chain
```

Then validate service configuration before reload.

---

# 176. Production Scenario — Kubelet Failed

Check:

```bash
systemctl status kubelet
journalctl -u kubelet -b
```

Then:

```text
container runtime
disk
memory
network
certificates
node configuration
```

Correlate with Kubernetes node events.

---

# 177. Production Scenario — containerd Failed

Check:

```bash
systemctl status containerd
journalctl -u containerd
```

Then:

```text
disk
socket
container runtime config
network
image storage
```

A runtime failure can cause multiple Pods to become unhealthy.

---

# 178. Production Scenario — Docker Service Failed

Check:

```bash
systemctl status docker
journalctl -u docker
```

Then inspect:

```text
containerd
storage
daemon.json
disk
network
```

---

# 179. Production Scenario — Systemd Reports OOM

Check:

```bash
journalctl -k
systemctl status myapp
```

Look for:

```text
Out of memory
Killed process
memory pressure
```

Then correlate:

```text
service memory
container memory
host memory
```

---

# 180. Production Scenario — Service Hits File Limit

Symptoms:

```text
too many open files
```

Check:

```bash
systemctl show myapp -p LimitNOFILE
```

and process descriptors.

Increasing the limit may help, but also investigate descriptor leaks.

---

# 181. Production Scenario — Service Hits Process Limit

Check:

```text
TasksMax
process count
thread count
worker configuration
```

Then identify whether the application is intentionally scaled or leaking processes.

---

# 182. Production Scenario — Service Has High CPU

Service management only tells you the process is running.

Continue with:

```text
PID
top
pidstat
process threads
application logs
metrics
```

Then determine whether CPU is:

```text
expected workload
runaway loop
traffic spike
retry storm
```

---

# 183. Production Scenario — Service Has High Memory

Check:

```text
systemd memory
process RSS
cgroup memory
host memory
OOM events
```

Then:

```text
application behavior
cache
heap
worker count
```

---

# 184. Production Scenario — Service Has D-State Process

A process in:

```text
D
```

is usually waiting in uninterruptible kernel I/O.

Investigate:

```text
storage
NFS
filesystem
block device
kernel logs
```

Restarting repeatedly may not fix the underlying I/O problem.

---

# 185. Production Scenario — Service Cannot Stop

Check:

```bash
systemctl status myapp
ps -ef
```

Look for:

```text
D-state
hung child
long shutdown
dependency
```

Only use forced termination when appropriate.

---

# 186. Production Scenario — SIGKILL Required

`SIGKILL`:

```text
cannot be caught
cannot be handled
does not allow graceful cleanup
```

Use it as a last resort.

After forced termination, investigate:

```text
why graceful shutdown failed
```

---

# 187. Production Scenario — Service Start Is Slow

Measure:

```bash
systemd-analyze blame
```

and service-specific startup timing.

Investigate:

```text
DNS
mounts
dependencies
database connection
application initialization
large configuration
```

Do not simply increase timeout.

---

# 188. `systemd-analyze`

Useful commands:

```bash
systemd-analyze
systemd-analyze blame
systemd-analyze critical-chain
```

These help understand boot and dependency timing.

---

# 189. Critical Chain

```bash
systemd-analyze critical-chain myapp.service
```

This helps identify dependencies contributing to startup timing.

---

# 190. Service Dependency Tree

```bash
systemctl list-dependencies myapp
```

Reverse relationships can also be useful:

```bash
systemctl list-dependencies \
    --reverse myapp
```

This helps determine impact before changing a service.

---

# 191. Production Change Impact

Before stopping a service:

```text
what depends on it?
```

Use:

```bash
systemctl list-dependencies \
    --reverse myapp
```

Then check:

```text
load balancer
clients
other services
scheduled jobs
```

---

# 192. Service Maintenance and Load Balancers

For a production API:

```text
remove instance from LB
       ↓
drain connections
       ↓
reload/restart
       ↓
health check
       ↓
rejoin LB
```

Do not restart a critical instance without considering traffic draining.

---

# 193. Blue/Green Deployment

Host-level service management can support:

```text
blue -> current
green -> new
```

Deploy green:

```text
validate
health check
```

Then switch traffic.

This reduces downtime and simplifies rollback.

---

# 194. Canary Deployment

A service can be introduced to:

```text
small percentage
```

of traffic.

Observe:

```text
errors
latency
CPU
memory
business metrics
```

Then expand or rollback.

Systemd can manage the service on a host, but traffic control belongs to the deployment/load-balancing layer.

---

# 195. Service Management in Immutable Infrastructure

In immutable infrastructure:

```text
build image
 ↓
test image
 ↓
deploy instance
```

You may minimize manual service changes after provisioning.

systemd still manages services inside the image, but the infrastructure lifecycle is controlled by the platform.

---

# 196. Golden Images

A VM image may contain:

```text
Python runtime
application
systemd unit
dependencies
hardening
monitoring agent
```

At boot:

```text
systemd
 ↓
application
```

This makes server provisioning more repeatable.

---

# 197. Cloud-Init and systemd

Cloud-init may:

```text
install packages
write configuration
create users
enable services
```

Then:

```text
systemd
```

takes over service lifecycle.

Avoid long-running application logic inside cloud-init when a proper service is more appropriate.

---

# 198. Service Management with Terraform

Terraform may provision:

```text
EC2
security groups
IAM
DNS
```

Then:

```text
cloud-init/Ansible
```

can configure:

```text
systemd
application
```

Keep infrastructure state and service configuration responsibilities clear.

---

# 199. Service Management with Ansible

Typical pattern:

```text
package
 ↓
configuration
 ↓
validate
 ↓
systemd daemon-reload
 ↓
service restart/reload
 ↓
health check
```

Ansible's systemd module is preferable to manually wrapping `systemctl` for standard configuration-management tasks.

---

# 200. Python's Role in Service Automation

Python is strongest when the workflow requires:

```text
custom logic
conditional decisions
API calls
data transformation
validation
multi-service orchestration
custom reporting
```

Use native tools for simple operations.

---

# 201. Python Service Wrapper

Example:

```python
import subprocess


def run_systemctl(
    action,
    service,
):
    allowed_actions = {
        "start",
        "stop",
        "restart",
        "reload",
        "status",
    }

    if action not in allowed_actions:
        raise ValueError(
            "Unsupported action"
        )

    return subprocess.run(
        [
            "systemctl",
            action,
            service,
        ],
        capture_output=True,
        text=True,
        check=False,
        timeout=60,
    )
```

Add service allowlisting for higher-risk tools.

---

# 202. Production-Grade Wrapper

A mature wrapper should include:

```text
input validation
allowlist
timeout
logging
structured result
exit code
retry policy
health verification
```

It should not blindly expose:

```text
arbitrary systemctl arguments
```

to users.

---

# 203. Service Automation Result

A useful result structure:

```python
{
    "service": "myapp",
    "action": "restart",
    "returncode": 0,
    "active": True,
    "stdout": "...",
    "stderr": "...",
}
```

This is easier to integrate into automation than raw shell output.

---

# 204. Service Preflight

Before restart:

```text
service exists
unit loaded
configuration valid
dependencies available
disk sufficient
certificate valid
port expected
```

Not every service requires every check.

Use checks appropriate to the failure modes.

---

# 205. Service Postflight

After restart:

```text
active
main PID exists
port listening
health endpoint
recent errors
dependency connectivity
```

Then report:

```text
success
or
failure
```

---

# 206. Post-Restart Verification Script

```python
import subprocess
import time


def wait_for_service(
    service,
    attempts=10,
):
    for _ in range(attempts):

        result = subprocess.run(
            [
                "systemctl",
                "is-active",
                service,
            ],
            capture_output=True,
            text=True,
        )

        if (
            result.stdout.strip()
            == "active"
        ):
            return True

        time.sleep(2)

    return False
```

Use bounded retries.

---

# 207. Health Endpoint Verification

```python
import urllib.request


def health_check(url):
    try:
        with urllib.request.urlopen(
            url,
            timeout=5,
        ) as response:

            return (
                200
                <= response.status
                < 300
            )

    except Exception:
        return False
```

For production code, catch expected network exceptions specifically and log failures.

---

# 208. Service Deployment Verification

```text
systemd active
      AND
port listening
      AND
health endpoint returns success
```

is much stronger than:

```text
systemd active
```

alone.

---

# 209. Health Check Retry

Applications may need time after process startup.

Use:

```text
start
 ↓
wait
 ↓
health
 ↓
retry
 ↓
health
 ↓
success/failure
```

Use a bounded timeout.

---

# 210. Avoid Fixed Long Sleeps

Bad:

```python
time.sleep(60)
```

Better:

```text
poll health
until success
or timeout
```

This can make deployments faster while remaining reliable.

---

# 211. Service Readiness Timeout

Example:

```text
maximum 120 seconds
poll every 5 seconds
```

If health does not recover:

```text
fail deployment
collect diagnostics
rollback
```

according to deployment policy.

---

# 212. Automated Rollback

A controlled workflow:

```text
new release
 ↓
activate
 ↓
health fails
 ↓
collect diagnostics
 ↓
rollback
 ↓
health check
 ↓
alert
```

Do not automatically rollback every transient failure without understanding the deployment system.

---

# 213. Rollback Guardrails

Before automatic rollback:

```text
Was old release available?
Is rollback known to be safe?
Did database schema change?
Are migrations backward compatible?
Are shared files compatible?
```

Database migrations are a major reason rollback is not always trivial.

---

# 214. Service Management and Database Migrations

A deployment may be:

```text
application v2
+
database schema v2
```

Rolling application back to v1 may fail if schema v2 is incompatible.

Use backward-compatible migration strategies.

---

# 215. Service Management and Feature Flags

Feature flags can reduce the need for immediate rollback.

Architecture:

```text
new code
 ↓
feature disabled
 ↓
deploy
 ↓
health check
 ↓
enable gradually
```

This separates deployment from feature activation.

---

# 216. Incident Data Collection

Before remediation, collect:

```text
systemctl status
journalctl
MainPID
resource metrics
ports
configuration version
deployment version
recent changes
```

This preserves evidence.

---

# 217. Avoid Evidence Destruction

A restart can:

```text
change process state
rotate logs
clear transient conditions
```

Before restarting during an incident, collect enough evidence to understand the failure when possible.

---

# 218. Service Incident Timeline

Record:

```text
10:01 alert
10:03 service failed
10:05 logs collected
10:07 configuration identified
10:10 rollback
10:12 health restored
```

This helps post-incident analysis.

---

# 219. Service Change Correlation

When a service fails, ask:

```text
What changed?
```

Look for:

```text
deployment
configuration
package update
certificate rotation
DNS
firewall
kernel
storage
dependency
```

A timeline often identifies the root cause faster than random commands.

---

# 220. Service Logs and ELK

If service logs are shipped:

```text
systemd/journal
 ↓
Logstash/Filebeat/agent
 ↓
Elasticsearch
 ↓
Kibana
```

You can correlate:

```text
service restart
application errors
deployment
host events
```

---

# 221. Service Metrics and Prometheus

Prometheus can monitor:

```text
process availability
CPU
memory
filesystem
service-specific metrics
```

Python custom exporters can expose additional metrics when necessary.

---

# 222. Avoid Custom Monitoring When Exporters Exist

If:

```text
node_exporter
```

already provides a metric, do not write a Python polling script just to duplicate it.

Use Python for genuinely custom logic.

---

# 223. Service Alert Runbook

An alert should link to a runbook containing:

```text
symptoms
commands
logs
health checks
known causes
safe remediation
rollback
escalation
```

This makes service incidents repeatable.

---

# 224. Runbook Example

```text
Alert:
myapp unavailable

1. systemctl status myapp
2. journalctl -u myapp -n 100
3. systemctl show myapp -p MainPID
4. ss -lntp
5. curl localhost:8080/health
6. check recent deployment
7. validate configuration
8. follow rollback policy
```

---

# 225. Common Service Management Mistakes

Avoid:

```text
restart without diagnosis
restart loops
sudo systemctl restart arbitrary service
editing vendor unit files
forgetting daemon-reload
assuming active = healthy
ignoring logs
ignoring dependencies
using root unnecessarily
unrestricted sudo
no health check
no rollback
```

---

# 226. Daily DevOps Script — Service Audit

```python
import subprocess

services = [
    "nginx",
    "docker",
    "myapp",
]

for service in services:
    result = subprocess.run(
        [
            "systemctl",
            "is-active",
            service,
        ],
        capture_output=True,
        text=True,
    )

    print(
        service,
        result.stdout.strip(),
    )
```

For production, use structured output and error handling.

---

# 227. Daily DevOps Script — Failed Service Report

```python
import subprocess

result = subprocess.run(
    [
        "systemctl",
        "--failed",
        "--no-legend",
        "--no-pager",
    ],
    capture_output=True,
    text=True,
    check=False,
)

if result.stdout.strip():
    print(
        "Failed services:"
    )
    print(result.stdout)
else:
    print(
        "No failed services"
    )
```

---

# 228. Daily DevOps Script — Service Status JSON

```python
import subprocess
import json


def get_status(service):
    properties = [
        "ActiveState",
        "SubState",
        "MainPID",
        "NRestarts",
    ]

    result = {}

    for prop in properties:
        output = subprocess.run(
            [
                "systemctl",
                "show",
                service,
                "-p",
                prop,
            ],
            capture_output=True,
            text=True,
            check=True,
        )

        key, _, value = (
            output.stdout.strip()
            .partition("=")
        )

        result[key] = value

    return result


print(
    json.dumps(
        get_status("myapp"),
        indent=2,
    )
)
```

---

# 229. Daily DevOps Script — Restart With Verification

```python
import subprocess
import time


def restart_and_verify(
    service,
    attempts=10,
):
    subprocess.run(
        [
            "systemctl",
            "restart",
            service,
        ],
        check=True,
        timeout=60,
    )

    for _ in range(attempts):
        result = subprocess.run(
            [
                "systemctl",
                "is-active",
                service,
            ],
            capture_output=True,
            text=True,
        )

        if (
            result.stdout.strip()
            == "active"
        ):
            return True

        time.sleep(2)

    return False
```

A production implementation should also perform an application health check.

---

# 230. Daily DevOps Script — Service Port Check

```python
import socket


def port_open(
    host,
    port,
):
    with socket.socket(
        socket.AF_INET,
        socket.SOCK_STREAM,
    ) as sock:

        sock.settimeout(3)

        return (
            sock.connect_ex(
                (host, port)
            )
            == 0
        )


print(
    port_open(
        "127.0.0.1",
        8080,
    )
)
```

This verifies TCP reachability, not application correctness.

---

# 231. Daily DevOps Script — Service Health

```python
import subprocess
import urllib.request


def systemd_active(service):
    result = subprocess.run(
        [
            "systemctl",
            "is-active",
            service,
        ],
        capture_output=True,
        text=True,
    )

    return (
        result.stdout.strip()
        == "active"
    )


def http_health(url):
    try:
        with urllib.request.urlopen(
            url,
            timeout=5,
        ) as response:
            return (
                200
                <= response.status
                < 300
            )
    except Exception:
        return False


service_ok = systemd_active(
    "myapp"
)

endpoint_ok = http_health(
    "http://127.0.0.1:8080/health"
)

print({
    "service": service_ok,
    "endpoint": endpoint_ok,
})
```

---

# 232. Daily DevOps Script — Configuration Then Reload

```python
import subprocess


def validate_and_reload():
    subprocess.run(
        ["nginx", "-t"],
        check=True,
        timeout=30,
    )

    subprocess.run(
        [
            "systemctl",
            "reload",
            "nginx",
        ],
        check=True,
        timeout=30,
    )


validate_and_reload()
```

The key principle is:

```text
validate first
reload second
```

---

# 233. Daily DevOps Script — Collect Service Diagnostics

```python
import subprocess


def run(command):
    result = subprocess.run(
        command,
        capture_output=True,
        text=True,
        check=False,
        timeout=30,
    )

    return {
        "command": command,
        "returncode":
            result.returncode,
        "stdout":
            result.stdout,
        "stderr":
            result.stderr,
    }


commands = [
    [
        "systemctl",
        "status",
        "myapp",
        "--no-pager",
    ],
    [
        "systemctl",
        "show",
        "myapp",
        "-p",
        "MainPID",
    ],
]

for command in commands:
    print(
        run(command)
    )
```

---

# 234. Daily DevOps Script — Failed Services JSON

```python
import subprocess
import json

result = subprocess.run(
    [
        "systemctl",
        "--failed",
        "--no-legend",
        "--no-pager",
    ],
    capture_output=True,
    text=True,
    check=False,
)

services = []

for line in (
    result.stdout.splitlines()
):
    fields = line.split()

    if fields:
        services.append(
            fields[0]
        )

print(
    json.dumps(
        {
            "failed_services":
                services
        },
        indent=2,
    )
)
```

---

# 235. Daily DevOps Script — Timer Audit

```python
import subprocess

result = subprocess.run(
    [
        "systemctl",
        "list-timers",
        "--all",
        "--no-pager",
    ],
    capture_output=True,
    text=True,
    check=True,
)

print(result.stdout)
```

This can be integrated into scheduled-job audits.

---

# 236. Daily DevOps Script — Kubelet Health

```python
import subprocess


def active(service):
    result = subprocess.run(
        [
            "systemctl",
            "is-active",
            service,
        ],
        capture_output=True,
        text=True,
    )

    return (
        result.stdout.strip()
        == "active"
    )


for service in [
    "kubelet",
    "containerd",
]:
    print(
        service,
        active(service),
    )
```

Use only on hosts where these services are expected.

---

# 237. Daily DevOps Script — Service Resource Summary

Combine:

```text
systemctl show
ps
```

to collect:

```text
MainPID
CPU
RSS
restart count
active state
```

This connects service management with process troubleshooting.

---

# 238. Service Dependency Health Script

A custom preflight can check:

```text
systemd service
database port
cache port
DNS
certificate
filesystem
```

Then produce:

```json
{
  "service": "myapp",
  "dependencies": {
    "database": "ok",
    "redis": "ok",
    "dns": "ok"
  }
}
```

Use this for deployment verification rather than continuous monitoring.

---

# 239. Production Service Automation Architecture

```text
Git
 ↓
CI/CD
 ↓
Artifact
 ↓
Configuration management
 ↓
systemd
 ↓
Application
 ↓
Health endpoint
 ↓
Prometheus / ELK
 ↓
Alerting
```

Python can sit between:

```text
CI/CD
configuration
deployment
validation
```

when custom logic is required.

---

# 240. Service Management Security Checklist

```text
[ ] Service runs as non-root
[ ] Least-privilege user
[ ] Restricted sudo
[ ] Unit files version-controlled
[ ] Secrets protected
[ ] Environment files restricted
[ ] Required capabilities minimized
[ ] Filesystem access restricted
[ ] Network access restricted where possible
[ ] No unrestricted shell input
[ ] No unnecessary shell=True
[ ] Logs do not expose secrets
[ ] Restart automation rate-limited
```

---

# 241. Service Management Reliability Checklist

```text
[ ] Correct startup type
[ ] Correct dependencies
[ ] Correct ordering
[ ] Correct restart policy
[ ] Graceful shutdown
[ ] Reasonable timeouts
[ ] Health checks
[ ] Bounded retries
[ ] No restart storm
[ ] Resource limits understood
[ ] Logs available
[ ] Rollback procedure documented
```

---

# 242. Deployment Checklist

```text
[ ] Artifact verified
[ ] Configuration backed up
[ ] Configuration validated
[ ] Service dependencies checked
[ ] daemon-reload if required
[ ] Reload preferred when safe
[ ] Restart only when needed
[ ] Service active verified
[ ] Port verified
[ ] Application health verified
[ ] Metrics checked
[ ] Logs checked
[ ] Traffic restored
```

---

# 243. Troubleshooting Command Cheat Sheet

```bash
systemctl status SERVICE

systemctl start SERVICE

systemctl stop SERVICE

systemctl restart SERVICE

systemctl reload SERVICE

systemctl enable SERVICE

systemctl disable SERVICE

systemctl daemon-reload

systemctl --failed

systemctl list-units --type=service --all

systemctl list-dependencies SERVICE

systemctl list-dependencies --reverse SERVICE

systemctl cat SERVICE

systemctl show SERVICE

systemctl is-active SERVICE

systemctl is-enabled SERVICE

journalctl -u SERVICE

journalctl -u SERVICE -n 100

journalctl -u SERVICE -f

journalctl -u SERVICE -b

journalctl -k

systemd-analyze blame

systemd-analyze critical-chain SERVICE
```

---

# 244. Python Service Management Cheat Sheet

```python
import subprocess


def systemctl(*args):
    return subprocess.run(
        [
            "systemctl",
            *args,
        ],
        capture_output=True,
        text=True,
        check=False,
        timeout=60,
    )


def active(service):
    result = systemctl(
        "is-active",
        service,
    )

    return (
        result.stdout.strip()
        == "active"
    )


def restart(service):
    result = systemctl(
        "restart",
        service,
    )

    if result.returncode != 0:
        raise RuntimeError(
            result.stderr
        )
```

---

# 245. Interview Question — What Is systemd?

**Answer:**

> systemd is a Linux system and service manager. It manages services, startup ordering, dependencies, processes, resource controls, timers, sockets, and other unit types. As a DevOps engineer, I commonly use systemctl for lifecycle operations and journalctl for service logs.

---

# 246. Interview Question — Restart vs Reload?

**Answer:**

> Restart normally stops and starts the service again, while reload asks the running service to re-read configuration without fully stopping. If the application supports a safe reload, I prefer it for configuration changes because it can reduce disruption.

---

# 247. Interview Question — What Is daemon-reload?

**Answer:**

> `systemctl daemon-reload` tells systemd to reload unit definitions after a unit file or drop-in has changed. It does not itself restart the service.

---

# 248. Interview Question — After vs Requires?

**Answer:**

> `After=` controls startup ordering, while `Requires=` expresses a dependency. They solve different problems and can be used together when a service both requires another unit and must start after it.

---

# 249. Interview Question — Why Does a Service Work Manually but Fail Under systemd?

**Answer:**

> The environments differ. I check the systemd user, group, PATH, working directory, environment variables, virtual environment, permissions, dependencies, and exact ExecStart command. I also inspect `journalctl -u service`.

---

# 250. Interview Question — How Do You Troubleshoot a Failed Service?

**Answer:**

```text
systemctl status
      ↓
journalctl -u
      ↓
systemctl cat
      ↓
systemctl show
      ↓
process/resources
      ↓
ports
      ↓
dependencies
      ↓
configuration
      ↓
health check
```

I fix the root cause rather than repeatedly restarting the service.

---

# 251. Interview Question — How Do You Make a Python Service Reliable?

**Answer:**

> I run it with a dedicated service account, use an explicit Python virtual environment, define a clear working directory, handle SIGTERM for graceful shutdown, configure appropriate restart behavior and timeouts, expose a health endpoint, and integrate logs and metrics with the platform.

---

# 252. Interview Question — How Do You Secure systemd Services?

**Answer:**

> I use a non-root user, least privilege, restricted file access, protected environment files, minimal capabilities, appropriate systemd sandboxing features, restricted sudo access, and version-controlled unit configuration.

---

# 253. Interview Question — How Would You Automatically Restart a Failed Service?

**Answer:**

> First I would determine whether systemd's restart policy is sufficient. If custom remediation is needed, I would collect diagnostics, check whether the failure is safe to remediate automatically, apply bounded retries with backoff, verify service and application health, and alert if recovery fails.

---

# 254. Interview Question — Why Is `active (running)` Not Enough?

**Answer:**

> Because it only tells me that systemd considers the process running. The application can still be unhealthy, unable to accept connections, returning errors, or unable to reach dependencies. I verify the port, health endpoint, logs, and relevant metrics.

---

# 255. Interview Question — How Do You Handle Service Configuration Changes?

**Answer:**

> I version the configuration, validate it before activation, create a backup when appropriate, use atomic replacement for important files, reload the service when supported, and verify application health afterward.

---

# 256. Interview Question — What Happens if a systemd Service Keeps Restarting?

**Answer:**

> I inspect the journal and exit status first. Then I check configuration, permissions, dependencies, ports, environment, resource limits, and application exceptions. I also check whether systemd has reached its restart/start-limit threshold. I do not simply keep restarting it.

---

# 257. Interview Question — How Does systemd Relate to Kubernetes?

**Answer:**

> On Kubernetes worker nodes, systemd can manage host-level components such as kubelet and containerd. If a Kubernetes incident is actually node-level, I may move from Kubernetes commands to systemctl and journalctl to investigate the node services.

---

# 258. Interview Question — How Would You Troubleshoot kubelet?

**Answer:**

```bash
systemctl status kubelet
journalctl -u kubelet -b
```

Then I correlate the logs with:

```text
kubectl get nodes
kubectl describe node
container runtime
disk pressure
memory pressure
network
certificates
```

---

# 259. Interview Question — How Would You Use Python Here?

**Answer:**

> I would use Python for custom service preflight checks, deployment orchestration, configuration validation, health verification, structured reporting, or controlled remediation. For simple service lifecycle operations, I would use systemctl directly or configuration-management tooling.

---

# 260. Final Service Management Mental Model

Think in this order:

```text
UNIT
 ↓
SYSTEMD STATE
 ↓
PROCESS
 ↓
RESOURCES
 ↓
PORT
 ↓
APPLICATION
 ↓
DEPENDENCIES
 ↓
HEALTH
 ↓
MONITORING
```

For automation:

```text
VALIDATE
 ↓
CHANGE
 ↓
VERIFY
 ↓
RECOVER
 ↓
REPORT
```

For production incidents:

```text
DETECT
 ↓
PRESERVE EVIDENCE
 ↓
DIAGNOSE
 ↓
SAFE REMEDIATION
 ↓
VERIFY
 ↓
MONITOR
 ↓
DOCUMENT
```

> **The purpose of Python service-management automation is not to replace systemd. It is to make complex operational workflows safer, repeatable, observable, and verifiable.**
