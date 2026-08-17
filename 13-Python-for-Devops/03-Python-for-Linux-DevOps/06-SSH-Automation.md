# SSH-Automation

## Python for Linux DevOps

SSH automation is one of the most practical uses of Python in Linux and DevOps environments.

DevOps engineers use SSH automation for:

- remote command execution
- server health checks
- application deployment
- configuration validation
- log collection
- service verification
- file transfer
- inventory checks
- incident diagnostics
- controlled remediation

The important principle is:

> **Use SSH automation for repeatable operational workflows, not as a replacement for every configuration-management or infrastructure tool.**

For large-scale configuration management, tools such as Ansible are usually more appropriate. Python becomes especially useful when SSH is one part of a larger workflow involving APIs, validation, reporting, CI/CD, or custom logic.

---

# 1. What SSH Provides

SSH provides secure remote access over an encrypted connection.

Typical flow:

```text
Developer / CI Runner
        |
        | SSH
        v
Linux Server
        |
        +-- command execution
        +-- file transfer
        +-- service checks
        +-- log collection
        +-- deployment
```

The default SSH server port is:

```text
22/TCP
```

The port can be changed, but changing the port alone is not a security control.

---

# 2. SSH Authentication

Common authentication methods include:

```text
password
SSH key
certificate-based authentication
hardware-backed keys
agent-based authentication
cloud-native temporary access
```

For DevOps automation, SSH keys or short-lived/cloud-native access mechanisms are generally preferable to storing passwords.

---

# 3. SSH Key Authentication

Typical files:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
~/.ssh/known_hosts
```

The private key must remain secret.

The public key is placed on the remote account, usually in:

```text
~/.ssh/authorized_keys
```

---

# 4. Generate an Ed25519 Key

```bash
ssh-keygen -t ed25519
```

A passphrase is recommended for human-managed private keys.

For automation, use an appropriate secret-management or agent strategy rather than exposing an unencrypted private key.

---

# 5. SSH Directory Permissions

Typical permissions:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

Exact permissions can vary, but private keys must not be world-readable.

---

# 6. Authorized Keys

Example:

```text
~/.ssh/authorized_keys
```

contains public keys authorized to log in as that user.

Do not confuse:

```text
authorized_keys
```

with:

```text
known_hosts
```

They solve different problems.

---

# 7. `authorized_keys` vs `known_hosts`

```text
authorized_keys
=
who is allowed to log in to this account?

known_hosts
=
which server key do I trust for this hostname/IP?
```

This distinction is important for SSH security.

---

# 8. Basic SSH Command

```bash
ssh user@server
```

Specify a key:

```bash
ssh -i ~/.ssh/id_ed25519 user@server
```

Specify a port:

```bash
ssh -p 2222 user@server
```

---

# 9. Remote Command

```bash
ssh user@server "uptime"
```

Another example:

```bash
ssh user@server \
    "systemctl is-active nginx"
```

For repeatable automation, Python can orchestrate these commands.

---

# 10. SSH Verbose Mode

Use:

```bash
ssh -v user@server
```

More detail:

```bash
ssh -vvv user@server
```

Useful for diagnosing:

```text
DNS
TCP connection
authentication
host key
key selection
server configuration
```

---

# 11. SSH Connection Troubleshooting

Think in layers:

```text
DNS
 ↓
TCP 22
 ↓
SSH handshake
 ↓
host key verification
 ↓
authentication
 ↓
authorization
 ↓
shell/command
```

Do not jump directly to changing passwords or keys.

---

# 12. Check TCP 22

```bash
nc -vz server 22
```

If TCP cannot connect, SSH authentication has not even started.

Check:

```text
route
firewall
security group
NACL
sshd listener
network connectivity
```

---

# 13. Check SSH Listener

On the server:

```bash
ss -lntp | grep ':22'
```

or:

```bash
sshd -T
```

Depending on the distribution, the service may be managed as:

```bash
systemctl status sshd
```

or:

```bash
systemctl status ssh
```

---

# 14. SSH Server Configuration

Typical file:

```text
/etc/ssh/sshd_config
```

Important settings include:

```text
Port
PermitRootLogin
PasswordAuthentication
PubkeyAuthentication
AllowUsers
AllowGroups
MaxAuthTries
ClientAliveInterval
ClientAliveCountMax
```

Always validate configuration before restarting `sshd`.

---

# 15. Validate sshd Configuration

```bash
sshd -t
```

If the command succeeds, the configuration syntax is valid.

A safe change workflow is:

```text
edit
 ↓
sshd -t
 ↓
restart/reload
 ↓
verify existing session
 ↓
test new session
```

Never make a remote SSH change without considering how you will recover if access breaks.

---

# 16. Reload vs Restart

A reload asks the service to re-read configuration where supported.

A restart stops and starts the service.

For critical remote access infrastructure:

```text
validate
 ↓
reload when appropriate
 ↓
verify
```

is often safer than an unnecessary restart.

---

# 17. Root Login

Avoid unnecessary direct root SSH access.

Prefer:

```text
normal user
 ↓
sudo
 ↓
required privileged command
```

This improves accountability and limits direct privileged access.

---

# 18. Password Authentication

For controlled production environments, organizations often disable password-based SSH authentication and use stronger mechanisms.

Typical configuration:

```text
PasswordAuthentication no
PubkeyAuthentication yes
```

Do not disable password access before confirming another working authentication path.

---

# 19. SSH Key Rotation

Keys should be rotated according to organizational policy.

Safe workflow:

```text
create new key
 ↓
install new public key
 ↓
test
 ↓
remove old key
 ↓
verify
```

Do not remove the old key before the new access path has been verified.

---

# 20. SSH Agent

An SSH agent can hold private keys so the private key does not need to be supplied repeatedly.

Check:

```bash
ssh-add -l
```

Add:

```bash
ssh-add ~/.ssh/id_ed25519
```

Agent use should still follow organizational security controls.

---

# 21. Known Hosts

SSH records trusted server keys in:

```text
~/.ssh/known_hosts
```

This helps detect unexpected host key changes.

Do not solve every host-key warning with:

```bash
ssh-keygen -R
```

without investigating why the key changed.

---

# 22. Host Key Change

A changed host key can be legitimate because of:

```text
server rebuild
instance replacement
IP reassignment
key rotation
```

But it can also indicate:

```text
man-in-the-middle attack
```

Treat unexpected changes seriously.

---

# 23. Python SSH Libraries

Common options:

```text
Paramiko
Fabric
AsyncSSH
subprocess + OpenSSH
```

For many DevOps workflows, using the system OpenSSH client through `subprocess` is a good choice when you already depend on standard SSH configuration, agents, known_hosts, and enterprise authentication.

Paramiko is useful when you need SSH directly from Python.

---

# 24. Paramiko

Install:

```bash
pip install paramiko
```

Use a virtual environment for application/tool dependencies.

---

# 25. Basic Paramiko Connection

```python
import paramiko

client = paramiko.SSHClient()

client.load_system_host_keys()

client.connect(
    hostname="server.example.com",
    username="devops",
    key_filename="~/.ssh/id_ed25519",
    timeout=10,
)

client.close()
```

In real code, expand the key path with `pathlib` or `os.path.expanduser`.

---

# 26. Host Key Verification

Avoid:

```python
client.set_missing_host_key_policy(
    paramiko.AutoAddPolicy()
)
```

as a blanket production solution.

Prefer:

```python
client.load_system_host_keys()
```

and manage trusted host keys properly.

---

# 27. Paramiko Reject Policy

```python
client.set_missing_host_key_policy(
    paramiko.RejectPolicy()
)
```

This prevents unknown hosts from being silently trusted.

The host key must already be trusted through the appropriate known-hosts mechanism.

---

# 28. Connect with an Expanded Key Path

```python
from pathlib import Path
import paramiko

key_file = Path(
    "~/.ssh/id_ed25519"
).expanduser()

client = paramiko.SSHClient()
client.load_system_host_keys()

client.connect(
    "server.example.com",
    username="devops",
    key_filename=str(key_file),
    timeout=10,
)
```

---

# 29. Remote Command Execution

```python
stdin, stdout, stderr = (
    client.exec_command(
        "uptime"
    )
)

output = stdout.read().decode()
error = stderr.read().decode()

print(output)
print(error)
```

Always consider command exit status.

---

# 30. Get Exit Status

```python
status = (
    stdout.channel.recv_exit_status()
)

print(status)
```

Interpretation:

```text
0 = success
non-zero = failure
```

---

# 31. Robust Remote Command Function

```python
def run_command(client, command):
    stdin, stdout, stderr = (
        client.exec_command(
            command,
            timeout=30,
        )
    )

    exit_code = (
        stdout.channel
        .recv_exit_status()
    )

    return {
        "command": command,
        "exit_code": exit_code,
        "stdout":
            stdout.read().decode(),
        "stderr":
            stderr.read().decode(),
    }
```

---

# 32. Why Exit Code Matters

This is dangerous:

```python
print(stdout.read())
```

without checking status.

A command may produce output and still fail.

Always evaluate:

```text
exit code
stderr
expected output
```

when necessary.

---

# 33. Command Timeout

Network connection timeout is not the same as command execution timeout.

You may need:

```text
connect timeout
command timeout
read timeout
```

depending on the library and workflow.

A production automation script should never wait forever.

---

# 34. SSH Connection Lifecycle

Use:

```text
create client
 ↓
load trusted keys
 ↓
connect
 ↓
run commands
 ↓
transfer files if required
 ↓
collect results
 ↓
close
```

Always close connections.

---

# 35. Context Manager Pattern

A small wrapper can ensure cleanup:

```python
from contextlib import contextmanager

@contextmanager
def ssh_connection(client):
    try:
        yield client
    finally:
        client.close()
```

This is especially useful when errors occur.

---

# 36. Multiple Commands

Instead of opening a new SSH connection for every command:

```text
connect once
 ↓
command 1
command 2
command 3
 ↓
close
```

Connection reuse reduces overhead.

---

# 37. Command Sequencing

Example:

```text
check service
 ↓
collect version
 ↓
collect health
 ↓
restart if explicitly allowed
 ↓
verify
```

Do not automatically restart services simply because a check failed.

---

# 38. Shell Commands and Safety

Avoid:

```python
client.exec_command(
    f"rm -rf {user_input}"
)
```

Untrusted input can become command injection.

Prefer:

```text
validated arguments
fixed commands
structured APIs
allowlists
```

---

# 39. Shell Injection

Dangerous pattern:

```python
command = (
    "systemctl restart "
    + service_name
)
```

If `service_name` is user-controlled, it may be interpreted as shell syntax depending on execution context.

Use strict validation or a safe command construction strategy.

---

# 40. Avoid `shell=True`

With local `subprocess`, avoid:

```python
subprocess.run(
    command,
    shell=True,
)
```

for untrusted input.

Prefer argument lists:

```python
subprocess.run(
    [
        "ssh",
        "user@server",
        "systemctl",
        "status",
        "nginx",
    ],
    check=True,
)
```

---

# 41. System OpenSSH from Python

Example:

```python
import subprocess

result = subprocess.run(
    [
        "ssh",
        "devops@server",
        "uptime",
    ],
    capture_output=True,
    text=True,
    timeout=30,
    check=False,
)

print(result.returncode)
print(result.stdout)
print(result.stderr)
```

This uses the system's OpenSSH behavior and configuration.

---

# 42. When `subprocess + ssh` Is Preferable

Use system OpenSSH when you want:

```text
~/.ssh/config
known_hosts
ssh-agent
enterprise SSH configuration
ProxyJump
existing OpenSSH behavior
```

It is often simpler than reimplementing those features in Python.

---

# 43. When Paramiko Is Preferable

Paramiko is useful when you need:

```text
Python-native SSH
programmatic SFTP
custom connection handling
integration into a Python application
```

Choose the simplest secure solution.

---

# 44. SSH Config

Typical:

```text
~/.ssh/config
```

Example:

```text
Host app-prod
    HostName 10.0.10.20
    User devops
    IdentityFile ~/.ssh/id_ed25519
```

Then:

```bash
ssh app-prod
```

This is very useful for human operations and Python scripts using OpenSSH.

---

# 45. ProxyJump

A bastion architecture may look like:

```text
Developer
   |
   v
Bastion
   |
   v
Private EC2
```

OpenSSH supports:

```bash
ssh -J bastion user@private-host
```

This avoids exposing private instances directly to the internet.

---

# 46. ProxyJump in Python

Using OpenSSH:

```python
subprocess.run(
    [
        "ssh",
        "-J",
        "devops@bastion",
        "devops@private-host",
        "hostname",
    ],
    check=True,
)
```

This is often simpler than implementing a custom bastion tunnel.

---

# 47. AWS Bastion Architecture

A traditional pattern:

```text
Internet
   |
   v
Bastion
   |
   v
Private instances
```

Modern AWS environments may instead use:

```text
SSM Session Manager
```

to avoid exposing SSH to the internet.

---

# 48. SSH vs AWS SSM

SSH provides:

```text
traditional remote shell
```

AWS Systems Manager can provide:

```text
managed session access
centralized access control
audit integration
no inbound SSH requirement
```

For AWS infrastructure, use the organization's approved access model.

---

# 49. SSH and IAM

SSH itself does not replace AWS IAM.

A secure AWS environment may use:

```text
IAM
 ↓
SSM
 ↓
instance
```

instead of:

```text
internet
 ↓
port 22
 ↓
instance
```

---

# 50. File Transfer

SSH supports secure file transfer.

Common tools:

```text
scp
sftp
rsync over SSH
Paramiko SFTP
```

---

# 51. SCP

Example:

```bash
scp app.tar.gz \
    user@server:/tmp/
```

Download:

```bash
scp user@server:/var/log/app.log .
```

---

# 52. SFTP

SFTP is an SSH-based file transfer protocol.

Python Paramiko:

```python
sftp = client.open_sftp()

sftp.put(
    "local.txt",
    "/tmp/remote.txt",
)

sftp.close()
```

---

# 53. SFTP Download

```python
sftp.get(
    "/var/log/app.log",
    "app.log",
)
```

Use explicit paths and permissions.

---

# 54. SFTP Directory Listing

```python
for item in sftp.listdir_attr(
    "/var/log"
):
    print(
        item.filename,
        item.st_size,
    )
```

This can support log collection and inventory tools.

---

# 55. SFTP File Permissions

SFTP can interact with file metadata.

Be careful when changing:

```text
ownership
permissions
modes
```

because incorrect changes can break applications or expose secrets.

---

# 56. Atomic File Deployment

Avoid writing directly to a live configuration file when possible.

Better:

```text
upload temporary file
 ↓
validate
 ↓
atomic rename
 ↓
reload service
 ↓
verify
```

This reduces the chance of leaving a partially written configuration.

---

# 57. Configuration Deployment

Example:

```text
local config
      ↓
SFTP upload
      ↓
remote /tmp/config.new
      ↓
syntax validation
      ↓
mv config.new config
      ↓
systemctl reload
      ↓
health check
```

---

# 58. Nginx Deployment Example

Workflow:

```text
upload nginx config
 ↓
nginx -t
 ↓
systemctl reload nginx
 ↓
curl health endpoint
```

Never reload before validating configuration.

---

# 59. Systemd Deployment Example

```text
upload unit file
 ↓
systemd-analyze verify
 ↓
systemctl daemon-reload
 ↓
systemctl restart app
 ↓
systemctl is-active app
```

Then perform application-level health validation.

---

# 60. Application Deployment Over SSH

A basic workflow:

```text
build artifact
 ↓
transfer artifact
 ↓
backup current version
 ↓
install new version
 ↓
restart/reload
 ↓
health check
 ↓
rollback if needed
```

Modern deployment platforms may provide safer mechanisms, but this pattern is useful for understanding deployment automation.

---

# 61. Versioned Releases

Use:

```text
/opt/myapp/releases/
    2026-08-17-1000/
    2026-08-17-1100/
```

Then:

```text
/opt/myapp/current
```

can point to the active release.

---

# 62. Symlink-Based Deployment

Concept:

```text
releases/
 ├── v1
 └── v2

current -> v2
```

Rollback:

```text
current -> v1
```

This can make rollback fast when implemented carefully.

---

# 63. Deployment Verification

After deployment:

```text
process active?
port listening?
health endpoint?
expected version?
logs clean?
dependencies reachable?
```

Do not consider:

```text
systemctl restart
```

as proof of successful deployment.

---

# 64. Remote Health Check

Example commands:

```bash
systemctl is-active myapp
ss -lntp
curl -f http://127.0.0.1:8080/health
```

A Python tool can execute and aggregate these checks.

---

# 65. Daily DevOps Script — SSH Health Check

```python
def check_server(client):
    checks = [
        "hostname",
        "uptime",
        "systemctl is-active nginx",
        "df -h /",
        "free -m",
    ]

    results = []

    for command in checks:
        results.append(
            run_command(
                client,
                command,
            )
        )

    return results
```

---

# 66. Better Health Check Design

Instead of checking only:

```text
server reachable
```

check:

```text
SSH
CPU/load
memory
disk
service
port
HTTP health
```

Only include checks that are relevant to the host role.

---

# 67. Daily DevOps Script — Disk Check

Remote command:

```bash
df -P /
```

Python can parse the result and fail if usage exceeds an agreed threshold.

Example policy:

```text
< 80% = normal
80–90% = warning
> 90% = critical
```

Thresholds should be environment-specific.

---

# 68. Daily DevOps Script — Memory Check

```bash
free -m
```

Prefer parsing structured sources where possible instead of brittle human-formatted output.

For Linux, `/proc/meminfo` can provide machine-readable information.

---

# 69. Daily DevOps Script — Process Check

```bash
pgrep -af myapp
```

Better when available:

```bash
systemctl is-active myapp
```

For systemd-managed applications, systemd state is generally a better source of truth.

---

# 70. Daily DevOps Script — Port Check

```bash
ss -lnt
```

Python can additionally perform a real connection test to the expected port.

---

# 71. Daily DevOps Script — Application Health

```bash
curl -fsS \
    http://127.0.0.1:8080/health
```

Possible Python equivalent:

```python
import urllib.request

with urllib.request.urlopen(
    "http://127.0.0.1:8080/health",
    timeout=5,
) as response:
    print(response.status)
```

---

# 72. Daily DevOps Script — Collect Logs

Example:

```bash
journalctl \
    -u myapp \
    -n 100 \
    --no-pager
```

Python can collect this output into a structured incident bundle.

Do not automatically dump massive logs into CI output.

---

# 73. Incident Bundle

A useful SSH diagnostic tool can collect:

```text
hostname
date
uptime
disk
memory
CPU/load
service state
listening ports
recent service logs
routes
DNS
application health
```

Output:

```text
incident-20260817-host01/
```

---

# 74. Daily DevOps Script — Incident Collector

Concept:

```text
SSH
 ↓
collect commands
 ↓
write local files
 ↓
create JSON summary
 ↓
compress
```

This is extremely useful during production incidents.

---

# 75. Incident Collection Safety

Avoid collecting:

```text
private keys
environment files
password files
secret manager tokens
```

unless there is an explicit secure process.

Diagnostic automation must not become a secret-exfiltration mechanism.

---

# 76. Parallel SSH

For many servers:

```text
inventory
 ↓
bounded worker pool
 ↓
SSH
 ↓
run health check
 ↓
collect result
```

Use:

```python
ThreadPoolExecutor
```

for I/O-bound workloads.

---

# 77. Parallel SSH Example

```python
from concurrent.futures import (
    ThreadPoolExecutor,
)

hosts = [
    "app01",
    "app02",
    "app03",
]

def check_host(host):
    # establish SSH
    # run checks
    # return structured result
    return host

with ThreadPoolExecutor(
    max_workers=10
) as executor:

    results = list(
        executor.map(
            check_host,
            hosts,
        )
    )
```

Bound the concurrency.

---

# 78. Why Bounded Concurrency Matters

If you have:

```text
500 servers
```

and create:

```text
500 simultaneous SSH connections
```

you may overload:

```text
CI runner
bastion
sshd
network
DNS
remote hosts
```

Use controlled concurrency.

---

# 79. SSH Connection Pooling

SSH connections are more expensive than ordinary function calls.

For repeated commands against the same server:

```text
reuse connection
```

when practical.

For independent jobs, carefully designed parallel connections may be simpler.

---

# 80. Inventory

A simple inventory might be:

```yaml
hosts:
  - name: app01
    address: 10.0.10.10
    user: devops

  - name: app02
    address: 10.0.10.11
    user: devops
```

Never store private keys or passwords in this file.

---

# 81. Environment-Specific Inventory

Example:

```text
inventory/
├── dev.yaml
├── staging.yaml
└── prod.yaml
```

Use explicit environment selection:

```bash
python health.py \
    --environment staging
```

---

# 82. Avoid Production Mistakes

Never assume:

```text
app01 = production
```

because of naming.

Validate environment from:

```text
inventory
cloud tags
configuration
explicit CLI option
```

---

# 83. SSH Host Verification in Inventory

An inventory should identify hosts through:

```text
DNS name
IP
known host key
cloud instance identity
```

Do not blindly trust whatever IP is supplied.

---

# 84. Bastion-Based Inventory

Example:

```yaml
bastion: bastion.internal

hosts:
  - app01.private
  - app02.private
```

The SSH layer can use:

```text
ProxyJump
```

to reach private hosts.

---

# 85. SSH Config + Inventory

A strong pattern is:

```text
inventory
 ↓
logical host names
 ↓
~/.ssh/config
 ↓
authentication + proxy
 ↓
OpenSSH
```

This keeps connection details out of application code.

---

# 86. Python `argparse` for SSH Tools

A useful CLI:

```bash
python ssh_health.py \
    --host app01 \
    --user devops \
    --timeout 10
```

Options might include:

```text
--host
--inventory
--user
--port
--timeout
--command
--environment
--dry-run
--output
```

---

# 87. Never Accept Arbitrary Commands by Default

This is dangerous:

```bash
python tool.py \
    --command "rm -rf /"
```

A production tool should expose controlled operations such as:

```text
health
restart-service
collect-logs
deploy
rollback
```

rather than arbitrary shell execution where possible.

---

# 88. Allowlisted Commands

Example:

```python
COMMANDS = {
    "uptime": "uptime",
    "disk": "df -P /",
    "memory": "free -m",
    "nginx": "systemctl is-active nginx",
}
```

Then:

```python
command = COMMANDS[action]
```

This is safer than arbitrary command input.

---

# 89. Privilege Escalation

Sometimes a command requires root privileges.

Use:

```bash
sudo systemctl restart myapp
```

Prefer a tightly scoped sudo policy.

Avoid:

```text
NOPASSWD: ALL
```

for automation accounts unless there is an exceptional, reviewed requirement.

---

# 90. Sudo Principle of Least Privilege

Better:

```text
automation user
 ↓
sudo systemctl restart myapp
```

rather than:

```text
automation user
 ↓
full root shell
```

---

# 91. Sudoers Validation

After modifying sudoers:

```bash
visudo -c
```

Never leave an invalid sudoers configuration on a production host.

---

# 92. Automation User

A dedicated account may be preferable:

```text
devops-automation
```

with:

```text
required SSH key
required sudo commands
required file permissions
```

Avoid sharing one personal key among an entire team.

---

# 93. SSH Account Lifecycle

Automation accounts should have:

```text
owner
purpose
rotation policy
access scope
expiration/review
audit trail
```

Remove unused accounts and keys.

---

# 94. SSH Agent Forwarding

Agent forwarding can be useful in some workflows but increases risk.

Understand:

```text
SSH_AUTH_SOCK
```

and the fact that a compromised remote host may potentially interact with the forwarded agent.

Do not enable agent forwarding by default.

---

# 95. `ForwardAgent`

OpenSSH config:

```text
ForwardAgent no
```

is a safer default unless forwarding is explicitly required.

---

# 96. SSH Port Forwarding

SSH supports:

```text
local forwarding
remote forwarding
dynamic forwarding
```

Example local forward:

```bash
ssh -L 15432:db.internal:5432 bastion
```

This makes a remote service reachable through the local port.

---

# 97. Port Forwarding Use Case

For troubleshooting a private database:

```text
Laptop
  |
  | localhost:15432
  v
Bastion
  |
  v
Private DB:5432
```

This can be useful for controlled administration.

---

# 98. Port Forwarding Security

Treat tunnels as privileged access.

Avoid:

```text
open-ended tunnels
public exposure
unapproved forwarding
```

Close tunnels when the task is complete.

---

# 99. SSH Tunnels in Python

If using OpenSSH, Python can orchestrate:

```text
ssh -L
```

through `subprocess`.

For programmatic tunnels, specialized SSH libraries can be used.

Always define:

```text
lifetime
destination
local bind address
authentication
```

---

# 100. File Permissions During Deployment

After transferring a file:

```bash
chmod
chown
```

may be required.

Example:

```bash
install \
    -o app \
    -g app \
    -m 0644 \
    /tmp/app.conf \
    /etc/myapp/app.conf
```

`install` can be safer and more explicit than a sequence of copy/chmod/chown operations.

---

# 101. Atomic Deployment

Use:

```text
temporary path
 ↓
validate
 ↓
atomic move
```

Example:

```bash
mv /tmp/app.conf \
   /etc/myapp/app.conf
```

within the same filesystem.

The exact atomicity and application behavior should be considered before relying on a move.

---

# 102. Remote File Hash

Before/after transfer:

```bash
sha256sum file
```

Python:

```python
import hashlib

def sha256_file(path):
    digest = hashlib.sha256()

    with open(path, "rb") as file:
        for chunk in iter(
            lambda: file.read(1024 * 1024),
            b"",
        ):
            digest.update(chunk)

    return digest.hexdigest()
```

This helps verify local artifacts.

---

# 103. Remote Hash Verification

After upload:

```bash
sha256sum /tmp/app.tar.gz
```

Compare with the expected artifact digest.

Do not rely only on filename.

---

# 104. Artifact Integrity

Deployment should ideally use:

```text
immutable artifact
 ↓
known digest
 ↓
transfer
 ↓
verify digest
 ↓
install
```

This reduces accidental deployment of the wrong file.

---

# 105. SSH + JFrog Artifactory

A pipeline may:

```text
Build artifact
 ↓
publish to Artifactory
 ↓
download approved artifact
 ↓
verify checksum
 ↓
deploy over SSH
 ↓
health check
```

This is safer than copying arbitrary local build output directly to production.

---

# 106. SSH + Jenkins

Example pipeline concept:

```text
Checkout
 ↓
Build
 ↓
Test
 ↓
SonarQube
 ↓
Trivy
 ↓
Artifact repository
 ↓
SSH deployment
 ↓
Health check
 ↓
Rollback if required
```

Use Jenkins credentials management rather than embedding keys in the Jenkinsfile.

---

# 107. SSH + GitHub Actions

A GitHub Actions workflow may use:

```text
OIDC/cloud authentication
or
secure deployment credentials
```

Store secrets in the platform's secret system.

Do not echo private keys in logs.

---

# 108. SSH + Ansible

A common architecture:

```text
Python
 ↓
custom preflight
 ↓
Ansible
 ↓
configuration
 ↓
Python
 ↓
post-deployment validation
```

This separation keeps responsibilities clear.

---

# 109. Python + Ansible Inventory

Python can consume inventory information generated by another system.

Avoid maintaining duplicate inventories if a source of truth already exists.

---

# 110. SSH + Terraform

Terraform provisions:

```text
EC2
VPC
Security Groups
IAM
ALB
```

Python can perform:

```text
post-provision checks
service validation
application health
```

Do not use SSH provisioners as the default mechanism for application deployment when a better deployment system exists.

---

# 111. Terraform Provisioners

`remote-exec` and `file` provisioners can be useful in limited situations but have tradeoffs.

Problems can include:

```text
ordering complexity
non-idempotent scripts
network dependency
reproducibility
state behavior
```

Prefer declarative Terraform resources for infrastructure and dedicated deployment/configuration tools for software.

---

# 112. SSH and Kubernetes Nodes

SSH can be used for node-level diagnostics when authorized:

```text
disk
memory
network
systemd
container runtime
kernel
```

But Kubernetes operational work should normally use:

```text
kubectl
Kubernetes API
node diagnostics
observability
```

before manually changing nodes.

---

# 113. EKS Node Troubleshooting

Useful node checks:

```bash
uptime
df -h
free -m
ss -lntp
systemctl status kubelet
systemctl status containerd
journalctl -u kubelet
```

Python can collect these remotely during incidents.

---

# 114. Kubelet Health

If a node is:

```text
NotReady
```

investigate:

```text
kubelet
container runtime
disk pressure
memory pressure
network
CNI
certificates
API server connectivity
```

SSH should be one diagnostic tool, not the only one.

---

# 115. Container Runtime Checks

Depending on the environment:

```bash
systemctl status containerd
crictl info
crictl ps
```

These can help determine whether the runtime is healthy.

---

# 116. Log Collection from Node

Example:

```bash
journalctl \
    -u kubelet \
    --since "30 min ago" \
    --no-pager
```

Python can retrieve this and save it locally.

---

# 117. Production Incident — SSH Failure

Symptoms:

```text
Cannot SSH to server
```

Troubleshoot:

```text
DNS
 ↓
TCP 22
 ↓
security group/firewall
 ↓
sshd listener
 ↓
host key
 ↓
authentication
 ↓
authorization
```

This avoids randomly changing keys or restarting services.

---

# 118. Production Incident — SSH Timeout

Check:

```bash
nc -vz host 22
```

Then:

```bash
ip route get <host-ip>
```

On cloud infrastructure inspect:

```text
security group
NACL
route table
network ACL
instance state
```

---

# 119. Production Incident — Permission Denied

Possible causes:

```text
wrong username
wrong key
public key not installed
authorized_keys permissions
home directory permissions
sshd configuration
account locked
PAM policy
```

Run:

```bash
ssh -vvv user@host
```

for client-side diagnostics.

---

# 120. Production Incident — Host Key Verification Failed

Do not immediately remove the old key.

First determine:

```text
Was the server rebuilt?
Did the IP change?
Was the key rotated?
Could this be an attack?
```

Then update trusted host information through the approved process.

---

# 121. Production Incident — `sshd` Won't Restart

First inspect:

```bash
sshd -t
```

Then:

```bash
journalctl -u sshd
```

or the distribution's SSH service name.

A configuration syntax error is a common cause.

---

# 122. Production Incident — Deployment Succeeds but App Is Down

Do not assume SSH success means application success.

Check:

```text
artifact
permissions
configuration
service
port
health endpoint
dependencies
logs
```

---

# 123. Production Incident — Deployment Partially Completed

Possible state:

```text
artifact uploaded
service stopped
new service failed
```

A good deployment tool needs:

```text
backup
health validation
rollback strategy
```

---

# 124. Rollback Strategy

A safe workflow:

```text
current v1
 ↓
deploy v2
 ↓
health fails
 ↓
restore v1
 ↓
restart/reload
 ↓
verify v1
```

Do not improvise rollback during a severe incident.

---

# 125. Blue/Green Concept

```text
Blue = current
Green = new
```

Deploy new version to Green:

```text
validate Green
 ↓
switch traffic
 ↓
monitor
```

SSH can support the server-side preparation, but load-balancer/platform tooling should normally control traffic switching.

---

# 126. Canary Concept

```text
small traffic
 ↓
new version
 ↓
observe
 ↓
increase traffic
```

This reduces blast radius.

SSH is rarely the entire canary mechanism; it may be one deployment component.

---

# 127. SSH Automation and Observability

After deployment:

```text
SSH check
 ↓
application health
 ↓
Prometheus metrics
 ↓
Grafana
 ↓
ELK logs
```

Operational validation should combine:

```text
synthetic checks
metrics
logs
application state
```

---

# 128. Remote Monitoring Agent Checks

Python can verify whether expected agents/services are running:

```text
node exporter
log shipper
application
container runtime
```

But do not assume a monitoring agent being active means the application is healthy.

---

# 129. Centralized Log Collection

SSH can collect emergency logs when centralized logging is unavailable.

Example:

```bash
journalctl -u myapp -n 500
```

However, normal production observability should use the centralized logging platform.

---

# 130. SSH as Break-Glass Access

SSH can be treated as:

```text
break-glass
```

for emergencies.

This means:

```text
strong authentication
limited users
auditing
time-bound access
least privilege
```

should be emphasized.

---

# 131. SSH Session Auditing

Organizations may use:

```text
bastion logs
SIEM
CloudTrail for related AWS actions
session recording
sudo logs
```

depending on the architecture.

Python automation should also log operational actions without secrets.

---

# 132. Time-Bound Access

For production access, prefer:

```text
temporary credentials
temporary role
temporary session
```

when supported.

Avoid permanent shared SSH credentials.

---

# 133. Shared SSH Keys

Bad practice:

```text
one private key
 ↓
entire team
```

Problems:

```text
no individual attribution
difficult rotation
large blast radius
```

Use individual or workload-specific credentials.

---

# 134. SSH Key Permissions

Private key should generally be readable only by its owner:

```bash
chmod 600 ~/.ssh/id_ed25519
```

If OpenSSH refuses a key because it is too open, fix permissions rather than disabling security checks.

---

# 135. SSH Hostname Validation

If connecting to:

```text
prod-app
```

ensure the name resolves to the intended host.

Do not silently redirect production names through untrusted DNS.

---

# 136. DNS + SSH Risk

A DNS change can redirect:

```text
ssh prod.example.com
```

to another server.

Therefore:

```text
DNS integrity
+
host key verification
```

provide important defense layers.

---

# 137. SSH Certificate Authentication

Larger organizations may use SSH certificates.

Benefits can include:

```text
centralized trust
short-lived certificates
easier rotation
less key sprawl
```

This is an advanced enterprise SSH pattern.

---

# 138. SSH Certificates vs Keys

Traditional:

```text
public key
 ↓
authorized_keys
```

Certificate-based:

```text
user key
 ↓
signed certificate
 ↓
trusted CA
 ↓
server
```

This can simplify large environments.

---

# 139. SSH Automation with Certificates

Python can often delegate authentication to system OpenSSH so enterprise certificate and agent configuration is reused.

This is another reason `subprocess + ssh` can be preferable in some environments.

---

# 140. SSH Connection Configuration

Useful OpenSSH options include:

```text
ConnectTimeout
ServerAliveInterval
ServerAliveCountMax
StrictHostKeyChecking
UserKnownHostsFile
ProxyJump
IdentityFile
```

Understand the security implications before overriding defaults.

---

# 141. Keepalive

SSH keepalive can help detect broken sessions.

Example:

```text
ServerAliveInterval 30
ServerAliveCountMax 3
```

This is not a substitute for proper network reliability.

---

# 142. Connection Timeout

Use:

```text
ConnectTimeout
```

or library-level timeouts.

A CI job should not hang indefinitely because a host is unreachable.

---

# 143. ServerAlive vs TCP Timeout

Conceptually:

```text
TCP/connect timeout
=
establishing connection

ServerAlive
=
detecting an idle/broken SSH session
```

Both may matter for long-running automation.

---

# 144. Long-Running Commands

Commands such as:

```bash
yum update
dnf update
tar
database migration
```

may take longer than ordinary health checks.

Set an appropriate timeout based on the operation.

Do not use an extremely short global timeout for every command.

---

# 145. Streaming Output

For long-running operations, streaming output may be preferable to waiting for the entire command to finish.

This allows:

```text
progress
early error detection
live logs
```

Be careful with log volume.

---

# 146. Remote Command Environment

SSH commands may not run with the same environment as an interactive shell.

Do not assume:

```text
PATH
JAVA_HOME
NVM
Python virtualenv
profile variables
```

are automatically loaded.

Use explicit paths/environment configuration.

---

# 147. Non-Interactive Shells

A command executed through SSH may not source:

```text
~/.bashrc
~/.profile
```

the same way as an interactive login shell.

This can explain:

```text
command not found
```

in automation even though the command works manually.

---

# 148. Use Absolute Paths

Prefer:

```bash
/usr/bin/systemctl
/usr/bin/python3
/usr/bin/curl
```

when path ambiguity matters.

Check with:

```bash
command -v python3
command -v curl
```

---

# 149. Environment Validation

Before deployment:

```text
whoami
hostname
pwd
env
```

Do not blindly print all environment variables because they may contain secrets.

Use targeted values.

---

# 150. Working Directory

Remote commands may start in an unexpected directory.

Use:

```bash
cd /opt/myapp && ./deploy.sh
```

or configure the working directory explicitly in the automation.

---

# 151. Remote Python Execution

You can run:

```bash
python3 /opt/tools/check.py
```

over SSH.

But for repeatable infrastructure operations, packaging the tool properly is usually better than copying ad-hoc scripts around.

---

# 152. Python Virtual Environment on Remote Host

If the remote tool has dependencies:

```bash
python3 -m venv /opt/tools/venv
```

Then:

```bash
/opt/tools/venv/bin/python \
    /opt/tools/check.py
```

Avoid depending on the system Python package set.

---

# 153. Remote Script Deployment

Workflow:

```text
Git
 ↓
CI
 ↓
test
 ↓
package
 ↓
upload
 ↓
verify checksum
 ↓
execute
 ↓
collect result
```

This makes remote automation reproducible.

---

# 154. Remote Script Versioning

Use:

```text
tool-v1.2.0.tar.gz
```

rather than:

```text
latest.tar.gz
```

for production deployment.

Immutable versions improve rollback and auditing.

---

# 155. Python Script Exit Codes

Remote script:

```python
import sys

if failed:
    sys.exit(1)

sys.exit(0)
```

The SSH automation layer should capture the exit code.

---

# 156. Structured Remote Result

Instead of only text:

```text
PASS
```

return JSON:

```json
{
  "host": "app01",
  "status": "PASS",
  "service": "myapp",
  "version": "1.8.2"
}
```

This is easier for CI/CD processing.

---

# 157. Remote JSON Parsing

```python
import json

data = json.loads(
    stdout
)

if data["status"] != "PASS":
    raise RuntimeError(
        "Remote check failed"
    )
```

Validate expected fields before using them.

---

# 158. SSH Error Classification

Common Paramiko exceptions include:

```text
AuthenticationException
BadHostKeyException
NoValidConnectionsError
SSHException
```

Use specific exception handling where possible.

---

# 159. Do Not Catch Everything

Avoid:

```python
except Exception:
    print("failed")
```

This hides useful information.

Prefer:

```python
except paramiko.AuthenticationException:
    ...
except paramiko.BadHostKeyException:
    ...
except OSError:
    ...
```

and preserve the original context.

---

# 160. Retry SSH Authentication?

Do not blindly retry authentication failures.

Repeated authentication attempts can:

```text
trigger account lockout
create security alerts
hide credential problems
```

Retry transient network failures more readily than credential failures.

---

# 161. SSH Retry Strategy

Potentially retry:

```text
temporary network timeout
connection reset
transient DNS failure
```

Usually do not retry repeatedly:

```text
authentication failure
host key mismatch
permission denied
invalid command
```

---

# 162. SSH Automation Logging

Log:

```text
host
operation
start time
duration
result
exit code
```

Do not log:

```text
private key
password
secret
authorization token
```

---

# 163. Correlation ID

For multi-host automation, assign:

```text
run_id
```

Example:

```text
deploy-20260817-1030
```

Every log entry can include the run identifier.

This makes CI/CD troubleshooting easier.

---

# 164. Output Directory

Example:

```text
artifacts/
└── deploy-20260817-1030/
    ├── summary.json
    ├── app01.json
    ├── app02.json
    └── errors.log
```

This creates a useful audit artifact.

---

# 165. SSH Automation Report

Example:

```json
{
  "run_id": "deploy-20260817-1030",
  "environment": "staging",
  "hosts": 3,
  "success": 3,
  "failed": 0
}
```

---

# 166. Daily DevOps Script — Multi-Host Health

```python
from concurrent.futures import (
    ThreadPoolExecutor,
)

def health_check(host):
    # connect
    # run safe checks
    # return structured result
    return {
        "host": host,
        "status": "PASS",
    }

hosts = [
    "app01",
    "app02",
    "app03",
]

with ThreadPoolExecutor(
    max_workers=5
) as executor:
    results = list(
        executor.map(
            health_check,
            hosts,
        )
    )

for result in results:
    print(result)
```

---

# 167. Daily DevOps Script — Service Restart

A controlled operation can be:

```python
ALLOWED_SERVICES = {
    "nginx",
    "myapp",
}

def restart_service(
    client,
    service,
):
    if service not in ALLOWED_SERVICES:
        raise ValueError(
            "Service not allowed"
        )

    return run_command(
        client,
        f"sudo systemctl restart {service}",
    )
```

In production, validate the service name carefully and use a command construction approach that prevents shell interpretation of unexpected input.

---

# 168. Better Than Arbitrary Restart

A safer CLI is:

```bash
python ops.py \
    restart \
    --service myapp
```

rather than:

```bash
python ops.py \
    --command "sudo systemctl restart myapp"
```

The tool controls what operations are possible.

---

# 169. Daily DevOps Script — Collect Node Diagnostics

```text
hostname
uptime
date
df -P
free -m
ip addr
ip route
ss -lnt
systemctl --failed
journalctl -p err -n 100
```

Collect only the information needed for the incident.

---

# 170. Production Diagnostic Bundle

Python can:

```text
SSH to node
 ↓
run approved diagnostic commands
 ↓
capture output
 ↓
write local files
 ↓
create summary JSON
```

This can save significant time during incidents.

---

# 171. Avoid Huge Diagnostic Bundles

More data is not always better.

Collect:

```text
high-signal information
```

first.

Avoid:

```text
entire filesystem
all environment variables
all logs
all process memory
```

unless explicitly required and approved.

---

# 172. Remote Log Tail

Useful:

```bash
journalctl \
    -u myapp \
    -n 100 \
    --no-pager
```

For file logs:

```bash
tail -n 100 /var/log/myapp.log
```

Limit output to prevent overwhelming CI logs.

---

# 173. Remote Grep

Example:

```bash
grep -i "error" \
    /var/log/myapp.log \
    | tail -n 50
```

For user-controlled patterns, avoid unsafe shell construction.

---

# 174. Remote File Existence

Python SFTP:

```python
try:
    sftp.stat(
        "/etc/myapp/app.conf"
    )
    exists = True
except FileNotFoundError:
    exists = False
```

This avoids spawning a shell command for a simple file check.

---

# 175. SFTP vs Remote Shell

Use SFTP for:

```text
upload
download
stat
list
```

Use SSH commands for:

```text
service control
system queries
application commands
```

Choose the appropriate interface.

---

# 176. Deployment Preflight

Before changing a server:

```text
SSH works
disk space sufficient
artifact available
service exists
backup possible
health endpoint known
rollback artifact available
```

If critical prerequisites fail:

```text
do not deploy
```

---

# 177. Deployment Postflight

After change:

```text
service active
port listening
health endpoint
version correct
logs reasonable
dependencies reachable
```

Then report:

```text
PASS
```

or:

```text
FAIL
```

---

# 178. Deployment Gate Architecture

```text
CI/CD
  |
  v
Python SSH preflight
  |
  +-- disk
  +-- service
  +-- network
  +-- artifact
  |
  v
Deploy
  |
  v
Python SSH postflight
  |
  +-- service
  +-- port
  +-- health
  +-- version
  |
  v
Success / Rollback
```

---

# 179. SSH Automation and Rollback

A deployment tool should know:

```text
current version
new version
rollback version
```

Do not make rollback depend on remembering commands during an incident.

---

# 180. Rollback Validation

After rollback:

```text
old version running?
health endpoint?
traffic restored?
errors decreasing?
```

Then stop automated changes and investigate the root cause.

---

# 181. SSH Automation and Change Windows

Production automation should respect:

```text
maintenance window
approval
change ticket
environment
rollback plan
```

Python can enforce preconditions before performing changes.

---

# 182. Approval Gate

Concept:

```text
Python preflight
 ↓
approval
 ↓
change
 ↓
verification
```

For high-risk production operations, keep human approval outside the script where appropriate.

---

# 183. Dry Run for SSH Automation

Example:

```bash
python deploy.py \
    --host app01 \
    --dry-run
```

Output:

```text
Would upload artifact
Would update symlink
Would restart myapp
Would run health check
```

No changes occur.

---

# 184. Production Environment Guard

A tool may require:

```bash
--environment production
```

and verify:

```text
target hostname
cloud tags
inventory
```

before proceeding.

This reduces accidental cross-environment changes.

---

# 185. Blast Radius

Before a multi-host change, define:

```text
batch size
max hosts
```

Example:

```text
deploy 1 host
 ↓
health check
 ↓
deploy next 2
 ↓
health check
 ↓
continue
```

This is safer than changing every host simultaneously.

---

# 186. Rolling Deployment

Example:

```text
10 servers
 ↓
deploy 1
 ↓
validate
 ↓
deploy 2
 ↓
validate
 ↓
...
```

If a failure occurs:

```text
stop
```

and investigate.

---

# 187. Batch Size

A good batch size depends on:

```text
capacity
traffic
redundancy
application architecture
SLO
rollback time
```

There is no universal number.

---

# 188. Avoid Single-Host Assumptions

If an application has:

```text
multiple instances
```

do not assume:

```text
all instances are identical
```

Before deployment, inspect:

```text
version
configuration
health
role
```

---

# 189. Configuration Drift Detection

SSH can inspect:

```text
package versions
service state
configuration checksum
kernel
runtime version
```

Compare against expected configuration.

---

# 190. Example Drift Report

```text
app01
  nginx: expected 1.26, actual 1.26
  myapp: expected 2.4.1, actual 2.4.1
  config: MATCH

app02
  nginx: expected 1.26, actual 1.24
  myapp: expected 2.4.1, actual 2.3.9
  config: DRIFT
```

Use configuration-management tooling for remediation.

---

# 191. Package Version Checks

Depending on OS:

```bash
rpm -q nginx
```

or:

```bash
dpkg -s nginx
```

Do not assume one package manager across all Linux distributions.

---

# 192. OS Detection

```bash
cat /etc/os-release
```

Python can read this file directly.

Use OS detection to choose appropriate checks.

---

# 193. Python OS Detection

```python
from pathlib import Path

data = Path(
    "/etc/os-release"
).read_text()

print(data)
```

For production tools, parse the key-value format rather than depending on exact line order.

---

# 194. Remote Python Platform Info

```python
import platform

print(
    platform.platform()
)
print(
    platform.python_version()
)
```

This can help with diagnostics.

---

# 195. Remote Service Inventory

Use:

```bash
systemctl list-units \
    --type=service \
    --state=running
```

Avoid collecting the entire service list unless required.

---

# 196. Failed Services

High-signal check:

```bash
systemctl --failed
```

This is useful during node health diagnostics.

---

# 197. Kernel and OS Checks

Useful:

```bash
uname -r
cat /etc/os-release
```

A deployment may depend on:

```text
kernel feature
OS version
runtime
```

Validate only relevant prerequisites.

---

# 198. SSH Automation with Containers

If a container needs SSH access:

```text
container
 ↓
SSH client
 ↓
server
```

Avoid baking private keys into Docker images.

Use:

```text
secret injection
SSH agent
ephemeral credentials
```

---

# 199. SSH Private Key in CI

Never:

```dockerfile
COPY id_rsa /root/.ssh/
```

into an image.

That key becomes part of image layers and may leak.

---

# 200. CI Secret Injection

Preferred concept:

```text
CI secret store
 ↓
ephemeral job
 ↓
SSH credential
 ↓
deployment
 ↓
job cleanup
```

The exact mechanism depends on the CI platform.

---

# 201. Known Hosts in CI

Do not blindly do:

```bash
ssh-keyscan host >> known_hosts
```

without validating the expected fingerprint through a trusted source.

Host-key trust is part of the security model.

---

# 202. SSH Host Key Pinning

For tightly controlled automation:

```text
expected host key
        ↓
compare
        ↓
connect only if match
```

This can reduce trust ambiguity.

---

# 203. SSH Key Storage

Use:

```text
secret manager
CI credentials store
SSH agent
cloud-native identity
```

Avoid:

```text
Git repository
Docker image
plain YAML
`.env` committed to Git
```

---

# 204. `.gitignore`

Private local SSH material should not be committed.

Typical patterns may include:

```text
*.pem
*.key
```

but do not rely only on `.gitignore`; secret prevention should include scanning and access controls.

---

# 205. Secret Scanning

A DevSecOps pipeline should scan for accidentally committed:

```text
private keys
tokens
passwords
cloud credentials
```

Tools may include:

```text
Gitleaks
TruffleHog
```

depending on organizational standards.

---

# 206. SSH Automation and DevSecOps

A secure deployment flow:

```text
Git
 ↓
SAST
 ↓
SCA
 ↓
Container scan
 ↓
Artifact
 ↓
Approval
 ↓
SSH / deployment mechanism
 ↓
Post-deployment validation
```

Security should not be added only after deployment.

---

# 207. Network Security for SSH

Controls may include:

```text
security groups
firewalls
VPN
bastion
SSM
private subnets
MFA-backed access
short-lived credentials
```

Port 22 should not be publicly reachable unless explicitly required.

---

# 208. Security Group Example

Prefer:

```text
SSH
source = approved admin network
```

over:

```text
SSH
source = 0.0.0.0/0
```

where the architecture permits.

---

# 209. SSH on Private Instances

A common AWS pattern:

```text
Admin
 ↓
VPN / SSM / Bastion
 ↓
Private EC2
```

No direct public SSH is required.

---

# 210. SSH and Zero Trust

Modern access models often focus on:

```text
identity
device
context
least privilege
short-lived access
continuous verification
```

rather than simply trusting a network segment.

---

# 211. SSH Automation Testing

Unit test:

```text
command construction
configuration parsing
result classification
retry logic
```

Integration test:

```text
real SSH server
SFTP
host-key verification
command execution
```

Never test destructive production operations against production.

---

# 212. Testcontainers / Temporary SSH

For automated tests, use an isolated temporary environment when practical.

The goal is:

```text
repeatable
safe
disposable
```

rather than relying on permanent test servers.

---

# 213. Mock SSH Client

You can mock:

```python
client.exec_command
client.connect
```

to test business logic without a real server.

Keep connection-specific integration tests separate.

---

# 214. Test Failure Cases

Test:

```text
DNS failure
connection timeout
authentication failure
host key mismatch
command failure
permission denied
SFTP failure
remote service failure
```

A DevOps tool is only as good as its failure handling.

---

# 215. SSH Automation Performance

Performance depends on:

```text
connection setup
authentication
network latency
command runtime
file transfer
concurrency
```

For many short commands:

```text
connection reuse
```

can make a significant difference.

---

# 216. SFTP Performance

Large transfers should consider:

```text
compression
parallelism
network bandwidth
remote disk
checksums
resume capability
```

For large artifact distribution, use an artifact repository rather than turning SSH into a package distribution system.

---

# 217. Use Artifact Repositories

For production:

```text
JFrog Artifactory
S3
container registry
```

are usually better than repeatedly copying large artifacts through SSH.

SSH should perform deployment orchestration when appropriate.

---

# 218. SSH and Docker Images

For containerized applications, prefer:

```text
build image
 ↓
push ECR
 ↓
deploy to EKS
```

rather than:

```text
SSH into server
 ↓
copy application files
 ↓
start process
```

The second model is more appropriate for traditional VM-based applications.

---

# 219. SSH in Kubernetes

Do not SSH into Pods as the normal application-management mechanism.

Use:

```text
kubectl exec
logs
API
observability
```

when appropriate.

SSH is primarily for node/host-level access.

---

# 220. SSH in EKS

Use node SSH only for:

```text
deep host diagnostics
approved break-glass operations
OS-level troubleshooting
```

Prefer managed access and Kubernetes-native tooling.

---

# 221. Daily DevOps Script — SSH + Service Validation

```python
def validate_service(client, service):
    result = run_command(
        client,
        f"systemctl is-active {service}",
    )

    return result["exit_code"] == 0
```

In a hardened implementation, service names should come from an allowlist.

---

# 222. Daily DevOps Script — SSH + HTTP Validation

Remote:

```bash
curl -fsS \
    http://127.0.0.1:8080/health
```

This validates the application from the server's own network namespace.

An external check should still be used when the requirement is user-facing reachability.

---

# 223. Internal vs External Health

Internal:

```text
localhost → application
```

External:

```text
client → DNS → LB → application
```

Both can succeed or fail independently.

Use the check that matches the real dependency.

---

# 224. Daily DevOps Script — Version Check

Remote:

```bash
/opt/myapp/bin/myapp --version
```

Python can compare:

```text
expected version
actual version
```

and fail the deployment if they differ.

---

# 225. Daily DevOps Script — Configuration Checksum

Remote:

```bash
sha256sum /etc/myapp/app.conf
```

Compare against the expected checksum.

This is useful for drift detection.

---

# 226. Daily DevOps Script — Service State

```python
result = run_command(
    client,
    "systemctl is-active myapp",
)

if result["exit_code"] != 0:
    raise RuntimeError(
        "Service is not active"
    )
```

---

# 227. Daily DevOps Script — Remote Disk Guard

Before deployment:

```text
required space
+
safety margin
```

For example:

```text
artifact = 2 GB
required free space = 5 GB
```

The exact threshold depends on the application.

---

# 228. Disk Guard Workflow

```text
check disk
 ↓
enough space?
 |       |
 NO     YES
 |       |
STOP   deploy
```

This prevents avoidable partial deployments.

---

# 229. Daily DevOps Script — Remote Backup

Before modifying configuration:

```text
current config
 ↓
timestamped backup
 ↓
new config
```

Example:

```text
app.conf
app.conf.bak.20260817-1030
```

Keep backup retention controlled.

---

# 230. Backup Verification

Do not assume a backup exists because a copy command succeeded.

Verify:

```text
file exists
size reasonable
checksum if appropriate
permissions
```

---

# 231. Backup Retention

Do not accumulate unlimited backups.

Use:

```text
retention policy
```

such as:

```text
last N versions
or
last N days
```

depending on requirements.

---

# 232. Remote Cleanup

Automated cleanup should be:

```text
allowlisted
age-based
path-restricted
```

Never construct unrestricted:

```text
rm -rf
```

from user-controlled input.

---

# 233. SSH Command Result Object

A useful internal structure:

```python
{
    "host": "app01",
    "command": "uptime",
    "exit_code": 0,
    "stdout": "...",
    "stderr": "",
    "duration_ms": 123
}
```

This makes reporting and testing easier.

---

# 234. Measure Remote Command Duration

```python
import time

start = time.monotonic()

result = run_command(
    client,
    "systemctl is-active myapp",
)

duration = (
    time.monotonic()
    - start
)
```

This can identify unexpectedly slow commands.

---

# 235. SSH Command Idempotency

Read-only:

```text
systemctl is-active
df
free
ss
```

are naturally safe.

Changes should be designed carefully:

```text
ensure file exists
ensure service enabled
ensure config matches
```

rather than blind repeated mutation.

---

# 236. Ensure vs Execute

Bad:

```text
restart service every run
```

Better:

```text
check state
 ↓
restart only if required
```

But even this should be based on a clearly defined operational policy.

---

# 237. Remote Configuration Management

If a task becomes:

```text
install packages
manage users
edit dozens of files
configure services
enforce permissions
```

move the responsibility to:

```text
Ansible
image building
configuration management
```

rather than expanding a custom SSH script indefinitely.

---

# 238. Python's Role

Python is strongest as:

```text
orchestrator
validator
API client
diagnostic tool
report generator
automation glue
```

SSH is one transport layer.

---

# 239. Real-World Architecture

```text
GitHub
   |
   v
Jenkins
   |
   +-- Python validation
   |
   +-- Security checks
   |
   v
Artifact Repository
   |
   v
Deployment
   |
   +-- SSH for VM application
   |
   +-- EKS/GitOps for Kubernetes
   |
   v
Post-deployment Python checks
   |
   v
Prometheus / Grafana / ELK
```

This is a realistic hybrid DevOps model.

---

# 240. Production Best Practices

```text
Use key-based or identity-based authentication
Verify host keys
Use least privilege
Use dedicated automation accounts
Use timeouts
Bound concurrency
Classify failures
Avoid arbitrary commands
Avoid shell injection
Use SFTP for file transfer
Verify artifacts
Use checksums
Use atomic configuration deployment
Validate before restart
Have rollback
Log actions
Protect secrets
Prefer SSM/private access where appropriate
Use Ansible for configuration management
Use Kubernetes-native tools for Kubernetes workloads
```

---

# 241. Common Mistakes

Avoid:

```text
AutoAddPolicy everywhere
hardcoded private keys
hardcoded passwords
shared SSH keys
public SSH access
root login
NOPASSWD ALL
arbitrary remote command execution
shell injection
infinite retries
no command timeout
unbounded parallel SSH
deploy without health check
restart without validation
no rollback
copying secrets into logs
copying private keys into Docker images
using SSH to manage Pods
```

---

# 242. SSH Security Checklist

```text
[ ] Host keys verified
[ ] Private keys protected
[ ] No hardcoded credentials
[ ] Least privilege
[ ] Dedicated automation identity
[ ] Restricted network access
[ ] Root SSH avoided
[ ] Password auth controlled
[ ] Sudo restricted
[ ] Agent forwarding controlled
[ ] Audit logging enabled
[ ] Keys rotated
[ ] Unused keys removed
[ ] Production access reviewed
```

---

# 243. Deployment Checklist

```text
[ ] Correct environment
[ ] Correct host inventory
[ ] SSH connectivity
[ ] Host identity verified
[ ] Disk space sufficient
[ ] Artifact verified
[ ] Backup available
[ ] Configuration validated
[ ] Service state checked
[ ] Change applied
[ ] Service verified
[ ] Port verified
[ ] Health endpoint verified
[ ] Version verified
[ ] Logs reviewed
[ ] Rollback available
```

---

# 244. Interview — What Is SSH?

**Answer:**

> SSH is a secure protocol for remote administration and data transfer. In DevOps, I use it for controlled host-level operations such as diagnostics, file transfer, and VM deployments. For large-scale configuration management, I prefer tools such as Ansible, and for Kubernetes workloads I prefer Kubernetes-native mechanisms.

---

# 245. Interview — Paramiko vs OpenSSH?

**Answer:**

> Paramiko provides Python-native SSH functionality, which is useful when SSH is deeply integrated into a Python application. The system OpenSSH client is often preferable when I want to reuse existing SSH configuration, known_hosts, SSH agents, ProxyJump, enterprise authentication, and standard command-line behavior.

---

# 246. Interview — How Do You Secure SSH Automation?

**Answer:**

> I verify host keys, protect private keys, use least-privilege accounts, avoid root access, restrict sudo, limit network exposure, use timeouts, validate inputs, avoid arbitrary command execution, and keep secrets out of logs and source code. In AWS, I also consider SSM or other managed access instead of exposing port 22.

---

# 247. Interview — What Happens When SSH Says Permission Denied?

**Answer:**

> I first determine whether the failure is authentication or authorization. I use verbose SSH output, verify username and key selection, inspect authorized_keys permissions, confirm sshd configuration, and check account/PAM restrictions. I don't immediately replace keys without understanding the failure.

---

# 248. Interview — How Do You Troubleshoot SSH Timeout?

**Answer:**

> I verify DNS if a hostname is used, then test TCP connectivity to the SSH port. If TCP fails, I inspect routing, security groups, NACLs, firewalls, and the sshd listener. Authentication is not relevant until the TCP connection is established.

---

# 249. Interview — How Do You Prevent SSH Host-Key Attacks?

**Answer:**

> I use known_hosts or another trusted host-key mechanism and reject unexpected keys. I don't blindly use AutoAddPolicy or disable strict host-key checking just to make automation work. A changed host key must be investigated.

---

# 250. Interview — How Would You Deploy an Application Over SSH?

**Answer:**

> I would validate the target and prerequisites, transfer a versioned artifact, verify its integrity, back up the current release if needed, install the new version atomically, validate configuration, restart or reload the service, run application health checks, and automatically stop or roll back if a critical validation fails.

---

# 251. Interview — How Do You Handle Multiple Servers?

**Answer:**

> I use an inventory and bounded concurrency. I connect to hosts in controlled batches, run read-only health checks in parallel where appropriate, collect structured results, and stop or reduce the rollout if failures exceed the defined threshold.

---

# 252. Interview — Why Not Use SSH for Everything?

**Answer:**

> SSH is a transport mechanism, not a complete configuration-management system. If I need idempotent package installation, users, files, services, and desired-state configuration across many machines, Ansible is a better fit. For cloud infrastructure, Terraform is appropriate for infrastructure state, and for Kubernetes I use Kubernetes-native or GitOps mechanisms.

---

# 253. Interview — How Do You Avoid Command Injection?

**Answer:**

> I avoid arbitrary shell commands, validate and allowlist operation names and arguments, use structured APIs where possible, and avoid shell interpolation of untrusted input. When using subprocess, I prefer argument lists over shell=True.

---

# 254. Interview — How Do You Handle SSH Retries?

**Answer:**

> I retry transient network failures with a bounded number of attempts and backoff. I don't blindly retry authentication failures or host-key mismatches because those usually indicate configuration or security problems.

---

# 255. Interview — How Do You Transfer Files Securely?

**Answer:**

> I use SFTP or SCP over verified SSH connections. For production artifacts, I prefer a trusted artifact repository such as Artifactory or S3 and verify the artifact checksum before installation. I never transfer private keys as application artifacts.

---

# 256. Interview — How Do You Validate a Deployment?

**Answer:**

> SSH success is only the first check. I validate the service state, listening port, application health endpoint, expected version, configuration, and relevant logs. For user-facing applications I also validate the external path through the load balancer or ingress.

---

# 257. Interview — What Is a Bastion Host?

**Answer:**

> A bastion is a controlled access point used to reach private infrastructure. Instead of exposing every private server to the internet, administrators connect through the bastion or another managed access system. In AWS I also consider SSM Session Manager to avoid inbound SSH exposure.

---

# 258. Interview — What Is ProxyJump?

**Answer:**

> ProxyJump allows OpenSSH to connect through an intermediate host. It is useful when the destination is in a private network and the client can only reach a bastion. It can be configured through `ssh -J` or SSH config.

---

# 259. Interview — What Is the Difference Between SCP and SFTP?

**Answer:**

> Both can transfer files over SSH. SFTP provides a richer file-operation protocol with directory listing, stat, upload, download, and related operations. For Python automation with Paramiko, SFTP is convenient for programmatic file management.

---

# 260. Interview — How Do You Safely Update a Configuration File?

**Answer:**

> I upload a temporary file, validate its syntax, verify ownership and permissions, create a backup when appropriate, atomically replace the active configuration, reload the service, and perform a health check. I avoid leaving partially written configuration in the active path.

---

# 261. Interview — How Do You Handle Rollback?

**Answer:**

> I keep a known-good version and define the rollback procedure before deployment. If post-deployment health checks fail, I stop the rollout and restore the known-good version, then verify service and application health.

---

# 262. Interview — How Would You Integrate SSH Automation into Jenkins?

**Answer:**

> Jenkins would build and test the artifact, run security checks, publish the artifact, and then invoke a controlled deployment tool. SSH credentials would come from Jenkins credentials management. The deployment script would perform preflight, deploy a versioned artifact, run post-deployment checks, and return a non-zero exit code on failure.

---

# 263. Interview — How Would You Integrate SSH Automation into GitHub Actions?

**Answer:**

> I would use GitHub's secure credential mechanisms or cloud-native OIDC where supported. The workflow would run the Python deployment tool, keep host keys trusted, avoid printing secrets, and publish structured deployment results as artifacts.

---

# 264. Interview — How Do You Troubleshoot a Node in EKS?

**Answer:**

> I first use Kubernetes status, events, metrics, and logs. If host-level access is required, I inspect disk, memory, network, kubelet, and containerd through approved node access. I avoid making arbitrary changes directly on nodes because the node may be managed and Kubernetes state should remain the source of truth.

---

# 265. Interview — How Do You Make an SSH Deployment Idempotent?

**Answer:**

> I check current state before changing it. I deploy immutable versions, compare expected and actual configuration, create the required directory only if missing, update the active release only when necessary, and verify the resulting state. Running the same deployment twice should not create an inconsistent state.

---

# 266. Interview — What Would You Log?

**Answer:**

> I log the run ID, target host, operation, start time, duration, exit status, and high-level result. I never log private keys, passwords, tokens, authorization headers, or sensitive environment variables.

---

# 267. Interview — What Would You Monitor?

For SSH automation itself:

```text
success rate
failure rate
authentication failures
connection latency
command duration
deployment duration
rollback count
```

For the target:

```text
CPU
memory
disk
service health
network
application health
```

---

# 268. Scenario — 20 Servers, One Deployment Fails

Approach:

```text
stop rollout
 ↓
identify failed host
 ↓
compare state
 ↓
check logs
 ↓
determine whether failure is isolated
 ↓
rollback failed host if needed
 ↓
decide whether remaining rollout is safe
```

Do not blindly continue across all servers.

---

# 269. Scenario — SSH Works Manually but Not in Jenkins

Likely differences:

```text
SSH agent
known_hosts
PATH
environment variables
user
key
ProxyJump
network location
permissions
```

Compare the Jenkins execution environment with the interactive environment.

---

# 270. Scenario — Script Works Manually but Fails Through Python

Check:

```text
working directory
PATH
shell initialization
environment
command quoting
permissions
timeout
user
```

Non-interactive SSH sessions often have a different environment.

---

# 271. Scenario — Deployment Hangs

Possible causes:

```text
SSH connection
DNS
command waiting for input
sudo password prompt
package manager lock
application migration
network call
```

Use:

```text
timeouts
non-interactive commands
sudo configuration
structured logs
```

to prevent indefinite hangs.

---

# 272. Scenario — `sudo` Hangs

A common cause is:

```text
sudo waiting for a password
```

Automation should use an approved non-interactive privilege model.

Do not simply pipe passwords into sudo.

---

# 273. Scenario — New SSH Key Does Not Work

Check:

```text
public key installed?
authorized_keys permissions?
home directory permissions?
correct username?
correct private key?
sshd configuration?
SELinux/PAM?
```

Use:

```bash
ssh -vvv
```

for detailed client diagnostics.

---

# 274. Scenario — Server Rebuilt with Same IP

SSH reports host-key mismatch.

Correct response:

```text
verify rebuild
verify new fingerprint
update trusted host key
```

Do not automatically overwrite known_hosts entries without verification.

---

# 275. Scenario — Artifact Uploaded but Service Won't Start

Check:

```text
artifact checksum
permissions
ownership
configuration
dependencies
runtime version
systemd status
journal logs
```

Then decide whether rollback is required.

---

# 276. Scenario — Service Is Active but Application Is Broken

This is why:

```text
systemctl is-active
```

is not enough.

Check:

```text
port
HTTP health
application logs
dependency connectivity
expected version
```

---

# 277. Scenario — All Servers Fail SSH

This suggests a shared dependency:

```text
bastion
DNS
security group
NACL
network route
credential
SSH CA
```

Investigate the common path before debugging each server individually.

---

# 278. Scenario — One Server Fails SSH

This is more likely host-specific:

```text
sshd
firewall
host state
key
permissions
network interface
```

Compare it against a healthy server.

---

# 279. Scenario — SSH Connection Is Slow

Measure:

```text
DNS latency
TCP connection
authentication
server load
reverse DNS/PAM behavior
network latency
```

Use verbose SSH output and server logs.

---

# 280. Scenario — SFTP Transfer Is Slow

Check:

```text
network bandwidth
CPU
disk I/O
cipher/compression
artifact size
remote filesystem
```

For very large artifacts, use an artifact repository instead of SSH file transfer.

---

# 281. Scenario — Production SSH Port Is Public

Recommended direction:

```text
remove broad exposure
 ↓
private access
 ↓
VPN / bastion / SSM
 ↓
least privilege
 ↓
audit
```

Do not simply move SSH from port 22 to another port and call the issue solved.

---

# 282. Scenario — Private EC2 Needs Emergency Access

Prefer the organization's approved mechanism:

```text
SSM Session Manager
or
VPN
or
bastion
```

If direct SSH is required, use restricted access and time-bound credentials.

---

# 283. Scenario — Automation Account Has Root

Reduce privileges:

```text
automation user
 ↓
specific sudo commands
```

Review every command that requires elevated access.

---

# 284. Scenario — SSH Key Is Exposed

Treat the key as compromised.

Typical response:

```text
revoke/remove public key
 ↓
rotate credentials
 ↓
investigate usage
 ↓
review logs
 ↓
replace deployment secret
```

Do not continue using the exposed key.

---

# 285. Scenario — Private Key Accidentally Committed

Response:

```text
remove from access
rotate immediately
scan repository/history
identify usage
replace credential
```

Deleting the file in a later commit does not make the credential safe.

---

# 286. Scenario — Host Key Suddenly Changes

Treat as a security event until explained.

Investigate:

```text
rebuild
key rotation
IP reuse
DNS change
possible MITM
```

---

# 287. Scenario — Deployment Works on One Host but Not Another

Compare:

```text
OS
package versions
runtime
configuration
environment
permissions
disk
network
service state
```

A drift report can automate much of this comparison.

---

# 288. Scenario — Application Cannot Reach Database After Deployment

From the target host:

```text
DNS database hostname
 ↓
route
 ↓
TCP database port
 ↓
TLS if applicable
 ↓
authentication
 ↓
application configuration
```

Do not immediately blame the database.

---

# 289. Scenario — Service Restarts Repeatedly

Check:

```text
systemd status
journal logs
exit code
configuration
dependencies
resource limits
```

SSH automation can collect diagnostics but should not continuously restart the service.

---

# 290. Scenario — Disk Full During Deployment

Immediate actions:

```text
stop deployment
identify filesystem
remove only safe temporary data
restore capacity
verify
```

Do not run broad deletion commands without understanding what is consuming disk.

---

# 291. Scenario — Memory Pressure During Deployment

Check:

```text
free
vmstat
top
systemd
application processes
OOM events
```

A deployment script should have resource preflight checks when deployments are resource-intensive.

---

# 292. Scenario — Network Failure During Deployment

The deployment tool should:

```text
timeout
stop
preserve known state
report
allow safe retry
```

It should not leave half-installed artifacts without a recovery strategy.

---

# 293. Production SSH Automation Design

A mature design:

```text
Input
 ↓
Validate
 ↓
Authenticate
 ↓
Verify host
 ↓
Preflight
 ↓
Change
 ↓
Postflight
 ↓
Report
 ↓
Rollback if required
```

Every stage should have clear failure behavior.

---

# 294. SSH Automation Project

## Project: Linux Fleet Health and Deployment Automation

Architecture:

```text
GitHub
   ↓
Jenkins
   ↓
Python CLI
   ↓
Inventory
   ↓
SSH / SFTP
   ↓
Linux fleet
   |
   +-- systemd
   +-- nginx
   +-- application
   +-- filesystem
   +-- network
   ↓
JSON results
   ↓
Jenkins artifacts
   ↓
Prometheus/Grafana/ELK
```

---

# 295. Project Features

Implement:

```text
health
deploy
rollback
collect-logs
check-version
check-disk
check-service
```

Use:

```text
argparse
logging
Paramiko/OpenSSH
YAML
JSON
concurrency
```

---

# 296. Project Repository

```text
ssh-automation/
├── config/
│   ├── dev.yaml
│   ├── staging.yaml
│   └── prod.yaml
├── scripts/
│   ├── health.py
│   ├── deploy.py
│   ├── rollback.py
│   └── collect_logs.py
├── src/
│   ├── ssh_client.py
│   ├── checks.py
│   └── reporting.py
├── tests/
├── requirements.txt
└── README.md
```

---

# 297. Project SSH Client Layer

Keep connection logic separate:

```text
ssh_client.py
```

Responsibilities:

```text
connect
execute
upload
download
close
```

Business logic should not be tightly coupled to Paramiko internals.

---

# 298. Project Checks Layer

```text
checks.py
```

Responsibilities:

```text
disk
memory
service
port
HTTP
version
configuration
```

Return structured results.

---

# 299. Project Reporting Layer

```text
reporting.py
```

Responsibilities:

```text
JSON
console
summary
failure count
duration
```

This separation makes testing easier.

---

# 300. Project Configuration

```yaml
environment: staging

hosts:
  - name: app01
    address: 10.0.10.20
    user: devops

checks:
  service: myapp
  port: 8080
  health_url: http://127.0.0.1:8080/health
```

Never put passwords/private keys in this file.

---

# 301. Project Health Flow

```text
load config
 ↓
validate
 ↓
connect SSH
 ↓
verify host
 ↓
collect diagnostics
 ↓
run health checks
 ↓
return JSON
```

---

# 302. Project Deployment Flow

```text
load config
 ↓
preflight
 ↓
artifact verification
 ↓
backup
 ↓
upload
 ↓
install
 ↓
validate config
 ↓
restart/reload
 ↓
health check
 ↓
success / rollback
```

---

# 303. Project Rollback Flow

```text
health check fails
 ↓
stop rollout
 ↓
restore known-good release
 ↓
restart/reload
 ↓
health check
 ↓
report
```

If rollback itself fails, escalate immediately rather than retrying destructive operations indefinitely.

---

# 304. Project CI/CD Gate

```text
Python exits 0
    ↓
Jenkins continues

Python exits non-zero
    ↓
Jenkins stops
```

This simple contract makes custom automation easy to integrate.

---

# 305. Project Security

Use:

```text
Jenkins credentials
SSH agent
known_hosts
least privilege
secret scanning
artifact checksum
audit logs
```

Do not store credentials in:

```text
Git
Docker image
Python source
YAML
console logs
```

---

# 306. Project Testing

Test:

```text
successful SSH
authentication failure
host-key mismatch
timeout
command failure
SFTP failure
health failure
rollback
multiple hosts
```

Use mock clients for unit tests and isolated SSH servers for integration tests.

---

# 307. Project Production Improvements

Add:

```text
structured JSON logs
retry policy
bounded concurrency
dry-run
approval gate
environment guard
batch deployment
metrics
notifications
```

The tool should remain small enough to understand and maintain.

---

# 308. Daily Commands to Remember

```bash
ssh user@host
ssh -i key user@host
ssh -v user@host
ssh -vvv user@host
ssh -J bastion user@host

scp file user@host:/tmp/
sftp user@host

nc -vz host 22
ss -lntp
systemctl status sshd
sshd -t
journalctl -u sshd
```

---

# 309. Python Modules to Remember

Standard library:

```text
subprocess
socket
pathlib
hashlib
json
logging
argparse
time
concurrent.futures
contextlib
```

External:

```text
paramiko
fabric
asyncssh
boto3
requests/httpx
PyYAML
```

Install only what the project actually needs.

---

# 310. Most Important DevOps Scripts

You should be comfortable building:

```text
1. SSH health checker
2. Multi-host diagnostic collector
3. Service validator
4. Remote log collector
5. Artifact deployer
6. Configuration validator
7. Version checker
8. Disk preflight checker
9. Rolling deployment tool
10. Rollback tool
```

These are much more valuable for DevOps interviews than generic Python exercises alone.

---

# 311. Practical Script — SSH Health Checker

```python
#!/usr/bin/env python3

import json
import subprocess
import sys


def run(host, command):
    result = subprocess.run(
        [
            "ssh",
            host,
            command,
        ],
        capture_output=True,
        text=True,
        timeout=30,
        check=False,
    )

    return {
        "command": command,
        "exit_code":
            result.returncode,
        "stdout":
            result.stdout.strip(),
        "stderr":
            result.stderr.strip(),
    }


def main():
    host = sys.argv[1]

    checks = [
        "hostname",
        "uptime",
        "df -P /",
        "free -m",
        "systemctl --failed",
    ]

    results = [
        run(host, command)
        for command in checks
    ]

    print(
        json.dumps(
            results,
            indent=2,
        )
    )


if __name__ == "__main__":
    main()
```

For production, add explicit argument validation, host-key policy, structured error handling, and safer command construction.

---

# 312. Practical Script — Service Validation

```python
import subprocess


def service_active(
    host,
    service,
):
    result = subprocess.run(
        [
            "ssh",
            host,
            "systemctl",
            "is-active",
            service,
        ],
        capture_output=True,
        text=True,
        timeout=20,
        check=False,
    )

    return (
        result.returncode == 0
    )
```

The service should come from a validated allowlist.

---

# 313. Practical Script — Remote Version Check

```python
def get_version(
    host,
    binary,
):
    result = subprocess.run(
        [
            "ssh",
            host,
            binary,
            "--version",
        ],
        capture_output=True,
        text=True,
        timeout=20,
        check=False,
    )

    if result.returncode != 0:
        raise RuntimeError(
            result.stderr
        )

    return result.stdout.strip()
```

Do not allow arbitrary binary paths from untrusted input.

---

# 314. Practical Script — Remote File Hash

```python
def remote_hash(
    host,
    path,
):
    result = subprocess.run(
        [
            "ssh",
            host,
            "sha256sum",
            path,
        ],
        capture_output=True,
        text=True,
        timeout=30,
        check=False,
    )

    if result.returncode != 0:
        raise RuntimeError(
            result.stderr
        )

    return (
        result.stdout
        .split()[0]
    )
```

Validate the path if it comes from external input.

---

# 315. Practical Script — Deployment Gate

```python
def deployment_gate(results):
    failures = [
        item
        for item in results
        if not item["ok"]
    ]

    if failures:
        return False

    return True
```

Then:

```python
if not deployment_gate(results):
    raise SystemExit(1)
```

---

# 316. Practical Script — Controlled Rollout

```text
for batch in batches:

    deploy(batch)

    results = health_check(batch)

    if failed(results):
        rollback(batch)
        stop_rollout()
```

This is safer than deploying to every server at once.

---

# 317. Practical Script — Dry Run

```python
def deploy(
    host,
    dry_run=False,
):
    if dry_run:
        print(
            f"Would deploy to {host}"
        )
        return

    # perform deployment
```

Dry-run mode should not accidentally perform any mutation.

---

# 318. Practical Script — JSON Result

```python
result = {
    "host": host,
    "operation": "health",
    "status": "PASS",
    "exit_code": 0,
}
```

This can be consumed by:

```text
Jenkins
GitHub Actions
ELK
custom dashboards
```

---

# 319. Practical Script — Duration

```python
import time

start = time.monotonic()

# operation

duration = (
    time.monotonic()
    - start
)
```

Store duration in seconds or milliseconds.

---

# 320. Practical Script — Logging

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

logging.info(
    "Starting health check"
)
```

Do not include secrets in log messages.

---

# 321. Practical Script — Retry Wrapper

```python
import time


def retry(
    function,
    attempts=3,
):
    delay = 1

    for attempt in range(
        attempts
    ):
        try:
            return function()

        except OSError:
            if (
                attempt
                == attempts - 1
            ):
                raise

            time.sleep(delay)
            delay *= 2
```

Production retry logic should distinguish transient and permanent errors.

---

# 322. Practical Script — Host Inventory

```python
hosts = [
    {
        "name": "app01",
        "address": "10.0.10.20",
        "user": "devops",
    },
    {
        "name": "app02",
        "address": "10.0.10.21",
        "user": "devops",
    },
]
```

Keep secrets outside inventory.

---

# 323. Practical Script — Concurrent Health

```python
from concurrent.futures import (
    ThreadPoolExecutor,
)

with ThreadPoolExecutor(
    max_workers=5
) as executor:

    results = list(
        executor.map(
            check_host,
            hosts,
        )
    )
```

Choose concurrency based on environment capacity.

---

# 324. Practical Script — Save Report

```python
from pathlib import Path
import json

Path(
    "health-report.json"
).write_text(
    json.dumps(
        results,
        indent=2,
    ),
    encoding="utf-8",
)
```

This creates a CI artifact.

---

# 325. Practical Script — Fail CI

```python
if any(
    not item["ok"]
    for item in results
):
    raise SystemExit(1)
```

This is the bridge between Python automation and CI/CD.

---

# 326. SSH Automation Interview Framework

When asked:

> "How would you automate this?"

Answer using:

```text
1. Input
2. Authentication
3. Validation
4. Execution
5. Verification
6. Error handling
7. Reporting
8. Security
9. Rollback
```

This demonstrates production thinking.

---

# 327. Example Interview Structure

Question:

> How would you deploy an application to 20 Linux servers using Python?

Answer structure:

```text
Inventory
 ↓
Validate environment
 ↓
SSH host verification
 ↓
Preflight
 ↓
Artifact checksum
 ↓
Controlled batch rollout
 ↓
Service restart/reload
 ↓
Health checks
 ↓
Stop on failure
 ↓
Rollback
 ↓
Structured report
```

---

# 328. Strong DevOps Interview Answer

> I would not simply loop over 20 hosts and execute SSH commands. I would build a controlled deployment workflow with inventory validation, secure authentication, host-key verification, preflight checks, versioned artifacts, bounded concurrency, batch rollout, post-deployment health checks, structured logging, and rollback. I would also keep infrastructure provisioning in Terraform and configuration management in Ansible rather than putting everything into a Python SSH script.

---

# 329. Key Takeaways

Remember:

```text
SSH is a transport
Python is an automation layer
Ansible is configuration management
Terraform is infrastructure as code
Kubernetes API is Kubernetes management
SSM can replace direct SSH in AWS
Artifact repositories manage artifacts
Observability verifies outcomes
```

The strongest DevOps engineer knows where each tool belongs.

---

# 330. Final Mental Model

```text
                DEVOPS AUTOMATION
                       |
        +--------------+--------------+
        |              |              |
     Terraform       Ansible        Python
        |              |              |
 Infrastructure   Configuration   Custom logic
        |              |              |
        +--------------+--------------+
                       |
                    SSH/API
                       |
                 Linux / AWS / EKS
                       |
          +------------+------------+
          |            |            |
       Services      Network      App
          |            |            |
          +------------+------------+
                       |
              Prometheus / Grafana
                       |
                       ELK
```

> **Use Python to connect the pieces, validate the real state, automate repetitive operations, and make deployment workflows safer—not to replace every specialized DevOps tool.**
