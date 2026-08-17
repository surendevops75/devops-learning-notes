# File-System-Automation

This module focuses specifically on **Linux filesystem automation with Python**.

The goal is not just learning file APIs. A DevOps engineer should be able to automate real production tasks involving:

```text
files
directories
permissions
ownership
disk usage
log files
configuration files
temporary files
mount points
cleanup
archives
symbolic links
filesystem health
```

---

# 1. Why File-System Automation Matters

Daily Linux administration often involves repetitive filesystem tasks:

```text
check disk usage
find large files
clean old logs
create application directories
copy configuration
set permissions
change ownership
archive files
verify backups
remove temporary files
inspect filesystem health
```

Python can make these tasks:

```text
repeatable
idempotent
auditable
safe
```

---

# 2. Python Modules Used

Important modules:

```python
pathlib
os
shutil
glob
fnmatch
stat
tempfile
hashlib
tarfile
zipfile
gzip
subprocess
```

For modern Python automation, prefer:

```python
pathlib
```

for most path operations.

---

# 3. `pathlib`

```python
from pathlib import Path

path = Path("/opt/myapp")
```

`Path` provides an object-oriented interface for filesystem paths.

---

# 4. Current Working Directory

```python
from pathlib import Path

print(Path.cwd())
```

Equivalent Linux concept:

```bash
pwd
```

---

# 5. Home Directory

```python
from pathlib import Path

print(Path.home())
```

Avoid hardcoding user home paths.

---

# 6. Build Paths Safely

Avoid:

```python
path = "/opt/myapp/" + filename
```

Prefer:

```python
base = Path("/opt/myapp")
path = base / filename
```

---

# 7. Check Path Existence

```python
path = Path("/etc/hosts")

if path.exists():
    print("Exists")
```

---

# 8. Check Regular File

```python
if path.is_file():
    print("File")
```

---

# 9. Check Directory

```python
if path.is_dir():
    print("Directory")
```

---

# 10. Check Symbolic Link

```python
if path.is_symlink():
    print("Symlink")
```

---

# 11. Create Directory

```python
Path("/opt/myapp").mkdir()
```

---

# 12. Create Nested Directories

```python
Path(
    "/opt/myapp/logs"
).mkdir(
    parents=True,
    exist_ok=True,
)
```

This is a common DevOps pattern.

---

# 13. Idempotent Directory Creation

Good automation should behave like:

```text
directory does not exist
        ↓
create it

directory already exists
        ↓
continue safely
```

Use:

```python
mkdir(
    parents=True,
    exist_ok=True,
)
```

---

# 14. Create an Empty File

```python
Path(
    "/opt/myapp/status"
).touch(
    exist_ok=True
)
```

---

# 15. Write a File

```python
config = Path(
    "/opt/myapp/config.txt"
)

config.write_text(
    "APP_ENV=production\n",
    encoding="utf-8",
)
```

---

# 16. Read a File

```python
content = config.read_text(
    encoding="utf-8"
)

print(content)
```

---

# 17. Append to a File

```python
with config.open(
    "a",
    encoding="utf-8",
) as file:
    file.write(
        "PORT=8080\n"
    )
```

---

# 18. Stream Large Files

Do not load a multi-GB log into memory.

Use:

```python
with log.open(
    encoding="utf-8",
    errors="replace",
) as file:

    for line in file:
        process(line)
```

---

# 19. Read File Line by Line

```python
with open(
    "/var/log/app.log",
    encoding="utf-8",
    errors="replace",
) as file:

    for number, line in enumerate(
        file,
        start=1,
    ):
        print(number, line.rstrip())
```

---

# 20. File Metadata

```python
info = path.stat()

print(info.st_size)
print(info.st_mtime)
print(info.st_mode)
```

---

# 21. File Size

```python
size = path.stat().st_size

print(size)
```

Convert to MB:

```python
size_mb = size / (
    1024 * 1024
)
```

---

# 22. Modification Time

```python
from datetime import datetime

mtime = path.stat().st_mtime

print(
    datetime.fromtimestamp(mtime)
)
```

For production reporting, prefer timezone-aware timestamps.

---

# 23. File Permissions

```python
import stat

mode = path.stat().st_mode

print(
    oct(stat.S_IMODE(mode))
)
```

Example:

```text
0o644
```

---

# 24. Change Permissions

```python
path.chmod(0o640)
```

Use least privilege.

Never use:

```bash
chmod -R 777
```

as a generic fix.

---

# 25. Linux Permission Model

Remember:

```text
owner
group
others
```

Example:

```text
-rw-r-----
```

means:

```text
owner  = read/write
group  = read
others = none
```

---

# 26. Directory Permissions

Example:

```python
Path(
    "/opt/myapp/secrets"
).chmod(0o700)
```

---

# 27. SSH Directory Permissions

Typical secure configuration:

```python
ssh_dir.chmod(0o700)
```

---

# 28. SSH Private Key

Typical private-key permission:

```python
private_key.chmod(0o600)
```

Always follow the requirements of the consuming SSH configuration.

---

# 29. Ownership

On Linux:

```python
import os

os.chown(
    path,
    uid,
    gid,
)
```

This normally requires appropriate privileges.

---

# 30. Resolve User UID

Avoid hardcoding:

```text
1000
```

Use:

```python
import pwd

user = pwd.getpwnam(
    "appuser"
)

print(user.pw_uid)
print(user.pw_gid)
```

---

# 31. Resolve Group GID

```python
import grp

group = grp.getgrnam(
    "appgroup"
)

print(group.gr_gid)
```

---

# 32. Set Application Ownership

```python
import os
import pwd
import grp

user = pwd.getpwnam(
    "appuser"
)

group = grp.getgrnam(
    "appgroup"
)

os.chown(
    path,
    user.pw_uid,
    group.gr_gid,
)
```

---

# 33. Directory Listing

```python
root = Path("/opt/myapp")

for item in root.iterdir():
    print(item)
```

Equivalent Linux idea:

```bash
ls
```

---

# 34. List Files Only

```python
for item in root.iterdir():
    if item.is_file():
        print(item)
```

---

# 35. List Directories Only

```python
for item in root.iterdir():
    if item.is_dir():
        print(item)
```

---

# 36. Recursive File Search

```python
for file in root.rglob("*.log"):
    print(file)
```

Equivalent concept:

```bash
find /opt/myapp -name "*.log"
```

---

# 37. Glob Patterns

Examples:

```text
*.log
*.yaml
*.yml
*.json
*.conf
app-*.log
```

---

# 38. Find Configuration Files

```python
for file in Path(
    "/etc/myapp"
).rglob("*.conf"):

    print(file)
```

---

# 39. Find Large Files

```python
threshold = (
    500 * 1024 * 1024
)

for file in Path(
    "/var"
).rglob("*"):

    try:
        if (
            file.is_file()
            and file.stat().st_size
            > threshold
        ):
            print(file)
    except OSError:
        continue
```

Production scanners should handle permission errors and files disappearing during traversal.

---

# 40. Find Old Files

```python
import time

cutoff = (
    time.time()
    - 7 * 24 * 60 * 60
)

for file in Path(
    "/var/log/myapp"
).rglob("*"):

    try:
        if (
            file.is_file()
            and file.stat().st_mtime
            < cutoff
        ):
            print(file)
    except OSError:
        continue
```

---

# 41. Safe Cleanup Workflow

Never start with:

```text
delete
```

Use:

```text
discover
 ↓
filter
 ↓
report
 ↓
dry-run
 ↓
validate
 ↓
delete
 ↓
verify
```

---

# 42. Delete File

```python
path.unlink()
```

---

# 43. Delete If Present

```python
path.unlink(
    missing_ok=True
)
```

This supports idempotent cleanup.

---

# 44. Delete Empty Directory

```python
path.rmdir()
```

It fails if the directory contains files.

---

# 45. Recursive Directory Delete

```python
import shutil

shutil.rmtree(path)
```

This is highly destructive.

Use strict path validation before calling it.

---

# 46. Path Traversal Risk

Dangerous input:

```text
../../etc
```

or:

```text
/opt/app/../../etc
```

Never trust a user-provided path.

---

# 47. Restrict Paths

```python
base = Path(
    "/opt/myapp"
).resolve()

candidate = (
    base / user_input
).resolve()

if (
    candidate != base
    and base not in candidate.parents
):
    raise ValueError(
        "Path escapes allowed directory"
    )
```

---

# 48. Symlink Safety

A path inside an approved directory may be a symlink to somewhere else.

For destructive operations:

```text
resolve
validate
operate
```

rather than blindly following paths.

---

# 49. Temporary Directories

```python
from tempfile import TemporaryDirectory

with TemporaryDirectory() as tmp:
    workdir = Path(tmp)

    print(workdir)
```

The temporary directory is automatically cleaned up.

---

# 50. Temporary Files

```python
import tempfile

with tempfile.NamedTemporaryFile(
    mode="w",
    encoding="utf-8",
) as file:

    file.write(
        "temporary data"
    )
```

Use `tempfile` instead of predictable temporary filenames.

---

# 51. Atomic Configuration Update

Production-safe pattern:

```text
write temporary file
        ↓
validate
        ↓
atomic replace
```

---

# 52. Atomic Replace Example

```python
import os
import tempfile

target = Path(
    "/opt/myapp/config.yaml"
)

with tempfile.NamedTemporaryFile(
    mode="w",
    dir=target.parent,
    delete=False,
    encoding="utf-8",
) as file:

    file.write(
        "version: 2\n"
    )

    temp_name = file.name

os.replace(
    temp_name,
    target,
)
```

Preserve required permissions and ownership explicitly.

---

# 53. Why Atomic Replacement?

Without it:

```text
write config
 ↓
process crashes
 ↓
partial config
```

With it:

```text
write temp
 ↓
validate
 ↓
replace
```

Readers see the old or completed file.

---

# 54. Copy Files

```python
import shutil

shutil.copy2(
    source,
    destination,
)
```

`copy2()` attempts to preserve additional metadata.

---

# 55. Copy Directory

```python
shutil.copytree(
    source,
    destination,
)
```

Modern Python can merge into an existing directory:

```python
shutil.copytree(
    source,
    destination,
    dirs_exist_ok=True,
)
```

---

# 56. Move Files

```python
shutil.move(
    source,
    destination,
)
```

---

# 57. Rename Files

```python
source.rename(
    destination
)
```

---

# 58. File Comparison

```python
import filecmp

same = filecmp.cmp(
    source,
    destination,
    shallow=False,
)

print(same)
```

---

# 59. SHA-256 Checksum

```python
import hashlib

def sha256(path):
    digest = hashlib.sha256()

    with open(
        path,
        "rb",
    ) as file:

        for chunk in iter(
            lambda: file.read(
                1024 * 1024
            ),
            b"",
        ):
            digest.update(chunk)

    return digest.hexdigest()
```

---

# 60. Why Checksums?

Useful for:

```text
artifact verification
backup integrity
configuration comparison
download verification
deployment verification
```

---

# 61. Archive Files

Python supports:

```text
tarfile
gzip
zipfile
```

Example:

```python
import tarfile

with tarfile.open(
    "backup.tar.gz",
    "w:gz",
) as archive:

    archive.add(
        "/etc/myapp",
        arcname="myapp",
    )
```

---

# 62. Archive Security

Never blindly extract untrusted archives.

Entries may contain:

```text
../../etc/passwd
```

or malicious links.

Validate extraction paths.

---

# 63. Linux Disk Usage

Python can invoke:

```bash
df -h
```

through `subprocess`.

```python
import subprocess

result = subprocess.run(
    ["df", "-h"],
    capture_output=True,
    text=True,
    check=True,
)

print(result.stdout)
```

---

# 64. Disk Usage with Python

For a filesystem:

```python
import shutil

usage = shutil.disk_usage("/")

print(
    "Total:",
    usage.total,
)

print(
    "Used:",
    usage.used,
)

print(
    "Free:",
    usage.free,
)
```

---

# 65. Disk Usage Percentage

```python
percent = (
    usage.used
    / usage.total
) * 100

print(
    f"{percent:.2f}%"
)
```

---

# 66. Inode Usage

Disk space and inodes are different.

Check:

```bash
df -i
```

Python can invoke it:

```python
subprocess.run(
    ["df", "-i"],
    check=True,
)
```

---

# 67. Why Inodes Matter

A server can have:

```text
free disk space
```

but:

```text
0 free inodes
```

because millions of tiny files exist.

---

# 68. Common Disk Consumers

Investigate:

```text
/var/log
/var/lib/docker
/var/lib/containerd
/tmp
application directories
backup directories
core dumps
```

Do not delete files blindly.

---

# 69. Deleted-but-Open Files

A file can be deleted while a process still holds it open.

Linux diagnostic:

```bash
lsof +L1
```

Python can orchestrate this command.

---

# 70. Log Cleanup

Typical policy:

```text
compress after 1 day
retain 14 days
delete after retention
```

The exact policy belongs to the application/organization.

---

# 71. Don't Fight the Logging System

Determine whether logs are managed by:

```text
logrotate
journald
container runtime
application logging
central logging platform
```

Avoid creating competing rotation mechanisms.

---

# 72. Docker Filesystem

Common locations can include:

```text
/var/lib/docker
/var/lib/containerd
```

Do not manually delete internal container-runtime files.

Use the runtime's supported cleanup commands.

---

# 73. Kubernetes Filesystem

Container filesystems are generally ephemeral.

Persistent application data should use:

```text
PersistentVolume
PersistentVolumeClaim
object storage
database
```

as appropriate.

---

# 74. Configuration Files

Common Linux configuration locations:

```text
/etc
/etc/systemd/system
/etc/nginx
/etc/ssh
/opt/myapp/config
```

Python can automate generation and validation.

---

# 75. Systemd Unit Files

Example path:

```text
/etc/systemd/system/myapp.service
```

After modifying a unit:

```bash
systemctl daemon-reload
```

Python:

```python
subprocess.run(
    [
        "systemctl",
        "daemon-reload",
    ],
    check=True,
)
```

---

# 76. Service Configuration Workflow

```text
generate
 ↓
write temporary file
 ↓
validate
 ↓
install
 ↓
daemon-reload if required
 ↓
restart/reload
 ↓
health check
```

---

# 77. Nginx Configuration

After changing Nginx configuration:

```bash
nginx -t
```

Python:

```python
subprocess.run(
    ["nginx", "-t"],
    check=True,
)
```

Only reload after validation succeeds.

---

# 78. SSH Configuration

For:

```text
/etc/ssh/sshd_config
```

validate with the appropriate SSH daemon configuration test before reloading/restarting.

Do not make unvalidated SSH changes remotely because a bad configuration can lock administrators out.

---

# 79. Application Directory Standard

A common application structure:

```text
/opt/myapp/
├── bin/
├── config/
├── logs/
├── data/
├── releases/
└── current
```

Python can create and validate this structure.

---

# 80. Directory Bootstrap Script

```python
from pathlib import Path

base = Path("/opt/myapp")

directories = [
    base / "bin",
    base / "config",
    base / "logs",
    base / "data",
    base / "releases",
]

for directory in directories:
    directory.mkdir(
        parents=True,
        exist_ok=True,
    )
```

---

# 81. Application Permissions

Example policy:

```text
bin      root-owned/read-execute
config   app-readable
logs     app-writable
data     app-writable
secrets  restricted
```

Actual ownership depends on the service design.

---

# 82. Permission Audit

```python
import stat

mode = path.stat().st_mode

if mode & stat.S_IWOTH:
    print(
        "World writable:",
        path,
    )
```

World-writable files can be a security concern.

---

# 83. Find World-Writable Files

```python
for file in Path(
    "/opt/myapp"
).rglob("*"):

    try:
        mode = file.stat().st_mode

        if mode & stat.S_IWOTH:
            print(file)

    except OSError:
        continue
```

---

# 84. Find Setuid Files

```python
if mode & stat.S_ISUID:
    print(
        "Setuid file:",
        file,
    )
```

Scope security scans carefully.

---

# 85. File Ownership Audit

Check:

```text
UID
GID
mode
```

against the expected application security policy.

---

# 86. Parent Directory Permissions

Checking only the file is not enough.

A user may be unable to read a file because a parent directory does not allow traversal.

Audit:

```text
file
parent
grandparent
...
```

when diagnosing access problems.

---

# 87. File-System Health Script

A useful DevOps script can report:

```text
filesystem
mount point
total space
used space
free space
usage percentage
inode usage
```

---

# 88. Health Threshold

Example:

```python
if percent >= 85:
    print(
        "WARNING: disk usage high"
    )

if percent >= 95:
    print(
        "CRITICAL: disk usage"
    )
```

Thresholds should be configurable.

---

# 89. Do Not Automatically Delete at 95%

A monitoring script should not assume the correct remediation.

First identify:

```text
what consumed the space
```

Then apply an approved cleanup policy.

---

# 90. File-System Health Report

Example:

```text
Filesystem Health
-----------------
Root usage: 71%
Inodes: 43%
Largest directory: /var/log
Largest file: app.log
Oldest log: 18 days
Status: WARNING
```

---

# 91. Daily DevOps Script — Large Files

```python
from pathlib import Path

root = Path("/var/log")
threshold = 500 * 1024 * 1024

for file in root.rglob("*"):
    try:
        if (
            file.is_file()
            and file.stat().st_size
            > threshold
        ):
            print(
                file,
                file.stat().st_size,
            )
    except OSError:
        continue
```

---

# 92. Daily DevOps Script — Old Files

```python
import time
from pathlib import Path

root = Path(
    "/var/log/myapp"
)

cutoff = (
    time.time()
    - 14 * 86400
)

for file in root.rglob("*.log"):
    try:
        if file.stat().st_mtime < cutoff:
            print(
                "Old:",
                file,
            )
    except OSError:
        continue
```

Start with reporting before deletion.

---

# 93. Daily DevOps Script — Directory Bootstrap

```python
from pathlib import Path

base = Path("/opt/myapp")

for name in [
    "config",
    "logs",
    "data",
    "releases",
]:
    (
        base / name
    ).mkdir(
        parents=True,
        exist_ok=True,
    )
```

---

# 94. Daily DevOps Script — Permission Audit

```python
from pathlib import Path
import stat

for path in Path(
    "/opt/myapp"
).rglob("*"):

    try:
        mode = path.stat().st_mode

        if mode & stat.S_IWOTH:
            print(
                "World writable:",
                path,
            )

    except OSError:
        continue
```

---

# 95. Daily DevOps Script — Checksum

```python
import hashlib

def checksum(path):
    digest = hashlib.sha256()

    with open(path, "rb") as file:
        for chunk in iter(
            lambda: file.read(
                1024 * 1024
            ),
            b"",
        ):
            digest.update(chunk)

    return digest.hexdigest()
```

---

# 96. Daily DevOps Script — Config Backup

```python
from pathlib import Path
import shutil

source = Path(
    "/etc/myapp/config.yaml"
)

backup = Path(
    "/backup/myapp/config.yaml"
)

backup.parent.mkdir(
    parents=True,
    exist_ok=True,
)

shutil.copy2(
    source,
    backup,
)
```

Production backup scripts should add timestamps, verification, retention, and restore testing.

---

# 97. Daily DevOps Script — Disk Check

```python
import shutil

usage = shutil.disk_usage("/")

percent = (
    usage.used
    / usage.total
) * 100

if percent >= 85:
    print(
        f"WARNING: {percent:.1f}% used"
    )
else:
    print(
        f"OK: {percent:.1f}% used"
    )
```

---

# 98. Daily DevOps Script — File Count

```python
from pathlib import Path

count = sum(
    1
    for item in Path(
        "/var/log"
    ).rglob("*")
    if item.is_file()
)

print(
    "Files:",
    count,
)
```

For huge trees, use bounded/scoped scanning.

---

# 99. Daily DevOps Script — Directory Size

```python
from pathlib import Path

total = 0

for file in Path(
    "/var/log"
).rglob("*"):

    try:
        if file.is_file():
            total += file.stat().st_size
    except OSError:
        continue

print(
    f"{total / 1024**3:.2f} GB"
)
```

---

# 100. Production Cleanup Guardrails

Before deletion:

```text
[ ] expected root
[ ] resolved path
[ ] allowed pattern
[ ] minimum age
[ ] candidate count
[ ] total size
[ ] symlink handling
[ ] dry run
[ ] backup if required
[ ] audit log
```

---

# 101. File-System Automation Architecture

```text
Python Script
     |
     +-- Path Validation
     |
     +-- File Discovery
     |
     +-- Metadata Collection
     |
     +-- Validation
     |
     +-- Action
     |
     +-- Verification
     |
     +-- Logging
     |
     +-- Metrics / Alerting
```

---

# 102. Idempotency

An operation is idempotent when running it multiple times produces the same intended state.

Example:

```python
Path(
    "/opt/myapp/logs"
).mkdir(
    parents=True,
    exist_ok=True,
)
```

---

# 103. Non-Idempotent Example

```python
with open(
    "/opt/myapp/data",
    "a",
) as file:
    file.write("entry\n")
```

Every execution changes the file.

That may be correct for logging but not for configuration management.

---

# 104. Make Configuration Automation Idempotent

Instead of:

```text
append line every run
```

prefer:

```text
read current
compare desired
update only when needed
```

---

# 105. Desired-State Model

```text
actual filesystem
       ↓
compare
       ↓
desired state
       ↓
change only if required
```

This is the same mindset used by:

```text
Ansible
Terraform
Kubernetes
GitOps
```

---

# 106. File-System Automation with Ansible

Python can generate or validate Ansible inputs, but Ansible already provides modules such as:

```text
file
copy
template
synchronize
unarchive
```

Use the specialized tool when it already solves the problem.

---

# 107. Python's Role

Python is especially useful for:

```text
complex logic
validation
orchestration
API integration
custom reporting
data transformation
```

---

# 108. File-System Automation with Terraform

Terraform should manage infrastructure state rather than arbitrary application files.

Python can:

```text
generate variables
validate files
prepare inputs
```

but avoid using Python to bypass Terraform's state model.

---

# 109. File-System Automation with CI/CD

Pipeline example:

```text
Git push
 ↓
Python validation
 ↓
filesystem/config tests
 ↓
artifact creation
 ↓
checksum
 ↓
deployment
```

---

# 110. Jenkins Example

A Jenkins stage might execute:

```bash
python scripts/validate_files.py
```

Then:

```text
validation
 ↓
build
 ↓
scan
 ↓
package
 ↓
deploy
```

---

# 111. GitHub Actions Example

```yaml
- name: Validate files
  run: |
    python scripts/validate_files.py
```

The script should return a non-zero exit code when validation fails.

---

# 112. Exit Codes

Good automation uses:

```text
0 = success
non-zero = failure
```

Example:

```python
import sys

if error:
    sys.exit(1)
```

---

# 113. Logging

Use Python's:

```python
import logging
```

rather than relying only on `print()` in production automation.

---

# 114. Safe Logging

Do not log:

```text
password
API token
private key
secret value
```

Even when troubleshooting filesystem automation.

---

# 115. JSON Reporting

Automation can produce:

```json
{
  "status": "success",
  "files_processed": 153,
  "bytes_processed": 10485760
}
```

Useful for CI/CD and monitoring integrations.

---

# 116. Error Handling

```python
try:
    path.unlink()

except FileNotFoundError:
    pass

except PermissionError:
    print(
        "Permission denied"
    )

except OSError as exc:
    print(
        f"Filesystem error: {exc}"
    )
```

---

# 117. Common Linux Filesystem Errors

```text
ENOENT  not found
EACCES  permission denied
EEXIST  already exists
ENOSPC  no space left
EROFS   read-only filesystem
ENOTDIR not a directory
```

---

# 118. Production Error Handling

Do not hide errors with:

```python
except Exception:
    pass
```

This can make production failures invisible.

---

# 119. File Race Conditions

Between:

```text
check
```

and:

```text
use
```

another process may modify the file.

This is known as a TOCTOU-style problem:

```text
Time Of Check
       ↓
Time Of Use
```

---

# 120. Prefer Atomic Operations

Where possible, use operations that combine the required check/action semantics rather than:

```text
check
then act
```

with a vulnerable gap.

---

# 121. File Locking

For cooperating processes on one Linux host, file locks can prevent concurrent changes.

Use appropriate OS locking mechanisms.

---

# 122. Distributed Locking

A local filesystem lock does not coordinate multiple hosts.

For multiple servers use an appropriate distributed mechanism such as:

```text
database
Redis
DynamoDB
coordination service
```

---

# 123. Backup Before Destructive Change

For critical configuration:

```text
backup
 ↓
change
 ↓
validate
 ↓
reload
 ↓
verify
```

---

# 124. Restore

A real backup strategy includes:

```text
backup
restore
verification
```

not just backup creation.

---

# 125. File-System Incident Example

**Problem:**

```text
server disk = 100%
```

Investigation:

```text
df -h
 ↓
df -i
 ↓
largest directories
 ↓
largest files
 ↓
deleted-open files
 ↓
logs
 ↓
container storage
 ↓
approved cleanup
```

---

# 126. File-System Incident Example — Permission Denied

Check:

```text
file owner
file mode
parent directory
ACL
SELinux
process user
mount options
```

Do not immediately change permissions.

---

# 127. File-System Incident Example — Config Corrupted

Response:

```text
stop deployment
 ↓
restore known-good
 ↓
validate
 ↓
reload
 ↓
health check
 ↓
investigate deployment process
```

---

# 128. File-System Incident Example — Cleanup Gone Wrong

Response:

```text
stop automation
 ↓
identify deleted data
 ↓
restore backup/snapshot
 ↓
preserve evidence
 ↓
find root cause
 ↓
add safety controls
 ↓
test destructive path cases
```

---

# 129. File-System Incident Example — Inodes Exhausted

Investigate:

```text
df -i
 ↓
directories with millions of files
 ↓
application temp files
 ↓
cache
 ↓
old artifacts
```

Then apply an approved retention policy.

---

# 130. File-System Incident Example — Deleted Log Still Consumes Disk

Check:

```bash
lsof +L1
```

If the application holds the deleted file open, gracefully reopen/restart logging according to the service's supported procedure.

---

# 131. File-System Incident Example — Read-Only Filesystem

Investigate:

```text
mount status
kernel logs
storage errors
filesystem health
disk state
mount options
```

Do not repeatedly retry writes without understanding why the filesystem became read-only.

---

# 132. File-System Security Checklist

```text
[ ] No path traversal
[ ] No unsafe recursive delete
[ ] Symlinks considered
[ ] Least privilege
[ ] Correct permissions
[ ] Correct ownership
[ ] Secrets protected
[ ] Temporary files secure
[ ] Archives validated
[ ] Inputs validated
[ ] Logs sanitized
```

---

# 133. Testing Filesystem Automation

Test:

```text
missing file
existing file
missing directory
existing directory
permission denied
symlink
path traversal
large file
empty file
binary file
read-only filesystem
concurrent execution
```

Use temporary test directories.

---

# 134. Temporary Test Environment

```python
from tempfile import TemporaryDirectory
from pathlib import Path

with TemporaryDirectory() as tmp:
    root = Path(tmp)

    file = root / "test.txt"

    file.write_text(
        "hello",
        encoding="utf-8",
    )
```

Never run destructive tests against:

```text
/
etc
var
production paths
```

---

# 135. Production Design Pattern

Use:

```text
discover
 ↓
validate
 ↓
stage
 ↓
verify
 ↓
promote
```

This pattern is useful for:

```text
configuration
releases
artifacts
backups
```

---

# 136. Example Release Layout

```text
/opt/myapp/
├── releases/
│   ├── 1.0.0/
│   ├── 1.1.0/
│   └── 1.2.0/
└── current -> releases/1.2.0
```

---

# 137. Release Deployment

```text
copy artifact
 ↓
extract new release
 ↓
validate files
 ↓
set ownership
 ↓
set permissions
 ↓
health/preflight
 ↓
switch current
 ↓
verify
```

---

# 138. Rollback

If:

```text
current -> 1.2.0
```

fails:

```text
current -> 1.1.0
```

Then verify the application.

---

# 139. Do Not Modify Active Release

Avoid:

```text
/opt/myapp/current/config.py
```

being modified in place during deployment.

Prefer versioned releases and atomic promotion.

---

# 140. File-System Automation and Containers

Containers should generally use:

```text
immutable image
```

instead of modifying application files after startup.

Python can prepare:

```text
build context
configuration
validation
diagnostics
```

but should not fight container immutability.

---

# 141. File-System Automation and EKS

In EKS, use Python for:

```text
manifest validation
diagnostic collection
artifact preparation
reporting
```

and let Kubernetes manage:

```text
pods
volumes
rollouts
desired state
```

---

# 142. File-System Automation and GitOps

With GitOps:

```text
Git
 ↓
desired configuration
 ↓
ArgoCD
 ↓
Kubernetes
```

Python should generally validate or generate the Git-controlled configuration rather than manually modifying live cluster files.

---

# 143. Daily DevOps Python Automation List

You should be able to write:

```text
1. disk_usage.py
2. large_files.py
3. inode_check.py
4. old_file_finder.py
5. log_cleanup.py
6. permission_audit.py
7. ownership_audit.py
8. config_backup.py
9. checksum.py
10. release_manager.py
11. filesystem_health.py
12. incident_bundle.py
```

---

# 144. Interview Question — `pathlib` vs `os.path`

**Answer:**

> `os.path` is still useful and widely used, but `pathlib` provides a cleaner object-oriented interface. For new Python automation I generally prefer `Path` because path manipulation and filesystem operations become easier to read and maintain.

---

# 145. Interview Question — How Do You Find Large Files?

**Answer:**

> I scope the filesystem first, recursively inspect file metadata, compare `st_size` against a threshold, handle permission/disappearing-file errors, and report the largest candidates. For very large filesystems I avoid building huge lists in memory.

---

# 146. Interview Question — How Do You Safely Delete Old Logs?

**Answer:**

> I define an explicit log directory and pattern, calculate an age cutoff, perform a dry run, apply candidate count and size limits, account for symlinks, and only then delete. I also verify that the logging system does not require those files.

---

# 147. Interview Question — How Do You Automate Permissions?

**Answer:**

> I define the desired owner, group, and mode as configuration, apply them explicitly, and verify them. I use least privilege and never solve permission issues with broad permissions such as 777.

---

# 148. Interview Question — How Do You Prevent Configuration Corruption?

**Answer:**

> I write to a temporary file, validate it, preserve required metadata, and atomically replace the target. I then reload the service and perform a health check.

---

# 149. Interview Question — How Do You Handle a 10 GB Log?

**Answer:**

> I stream it rather than using `read()` to load it into memory. For processing I use line-by-line or chunked reads, and for compression/hashing I use streaming operations.

---

# 150. Interview Question — What Happens if Disk Is Full?

**Answer:**

> File operations can raise `OSError` with `ENOSPC`. I would investigate filesystem space, inode usage, deleted-open files, quotas, and the largest consumers rather than simply retrying the write.

---

# 151. Interview Question — Why Use Checksums?

**Answer:**

> Checksums provide content integrity verification. I use SHA-256 for artifacts, backups, and configuration verification when integrity needs to be established.

---

# 152. Interview Question — Why Use Temporary Files?

**Answer:**

> They allow me to stage changes without modifying the active file. Combined with validation and atomic replacement, they reduce the risk of partial configuration or artifact writes.

---

# 153. Interview Question — What Is Idempotency?

**Answer:**

> An idempotent operation can be executed repeatedly while converging to the same desired state. For example, creating a directory with `exist_ok=True` does not fail when the directory already exists.

---

# 154. Interview Question — How Do You Handle Symlinks?

**Answer:**

> I identify whether paths are symlinks and resolve them when validating their real destination. For destructive operations I do not blindly follow symlinks because they can redirect an operation outside the intended directory.

---

# 155. Interview Question — How Do You Automate Backups?

**Answer:**

> I validate the source, create a timestamped archive or copy, verify its integrity, apply retention, store it in the approved destination, and periodically test restoration.

---

# 156. Interview Question — How Do You Implement Rollback?

**Answer:**

> For releases I use versioned immutable directories and an atomic `current` pointer. If health checks fail, I switch back to the previous known-good release and verify service health.

---

# 157. Interview Question — How Would You Design a Production Cleanup Script?

**Answer:**

> I would make the target directory explicit, validate the resolved path, filter by known patterns and age, calculate candidate count and size, support dry-run, use a lock to avoid overlapping executions, delete only approved candidates, and produce an audit report.

---

# 158. Interview Question — How Do You Handle Millions of Files?

**Answer:**

> I avoid creating a giant list, use lazy traversal or `os.scandir()`, restrict the scan scope, process in batches, and control concurrency. If the workload becomes too large for filesystem scanning, I reconsider the architecture.

---

# 159. Interview Question — What Is the Difference Between Disk Space and Inodes?

**Answer:**

> Disk space measures available storage capacity, while inodes track filesystem objects such as files and directories. A filesystem can have free GBs but still fail file creation when it has no free inodes.

---

# 160. Interview Question — Why Should You Not Delete Docker Storage Manually?

**Answer:**

> Container runtimes maintain metadata and storage relationships. Manually deleting internal files can corrupt runtime state. I use Docker/containerd's supported cleanup mechanisms instead.

---

# 161. Interview Question — How Would You Troubleshoot Permission Denied?

**Answer:**

> I check the process user, file owner/group, mode bits, parent-directory permissions, ACLs, SELinux/AppArmor, filesystem mount options, and container security context. I identify the actual access requirement before changing permissions.

---

# 162. Interview Question — How Do You Make File Automation Safe in Production?

**Answer:**

> I use allowlisted paths, input validation, dry-run support, idempotency, atomic operations, least privilege, locks, bounded destructive actions, structured logging, verification, and a recovery strategy.

---

# 163. Production Scenario — Disk 100%

**Question:**

A production EC2 server reaches 100% disk usage. What do you do?

**Answer:**

```text
df -h
 ↓
df -i
 ↓
identify affected filesystem
 ↓
find large directories
 ↓
find large files
 ↓
check deleted-open files
 ↓
check logs/container storage
 ↓
apply approved cleanup
 ↓
verify free space
```

I would not run a blind recursive delete.

---

# 164. Production Scenario — Application Cannot Read Config

**Answer:**

Check:

```text
path
owner
group
permissions
parent directory
ACL
SELinux
container user
volume mount
```

Then fix the actual cause.

---

# 165. Production Scenario — Deployment Created Partial Files

**Answer:**

> I would stop promotion, remove or isolate the incomplete release, keep the active version untouched, and change the deployment to stage files in a temporary/versioned directory before atomic promotion.

---

# 166. Production Scenario — Cleanup Removed Too Many Files

**Answer:**

> I would stop the automation, determine the exact candidate-selection bug, restore from backup/snapshot if needed, and add stronger root validation, dry-run review, maximum candidate count, maximum deletion size, and symlink/path traversal tests.

---

# 167. Production Scenario — Configuration Change Broke Service

**Answer:**

```text
rollback
 ↓
validate known-good config
 ↓
reload
 ↓
health check
 ↓
investigate
```

Then improve pre-deployment validation.

---

# 168. Production Scenario — Different Permissions Across Servers

Compare:

```text
UID/GID
mode
umask
ACL
filesystem
mount options
security context
deployment process
```

Do not assume the file content is the only difference.

---

# 169. Production Scenario — Backup Exists but Restore Fails

Treat the backup process as failed.

Investigate:

```text
archive integrity
missing files
permissions
encryption
keys
dependencies
restore procedure
```

Add regular restore tests.

---

# 170. Production Scenario — Millions of Temporary Files

Investigate:

```text
application behavior
cleanup policy
inode usage
file creation pattern
```

Then implement bounded cleanup and fix the producer if possible.

---

# 171. Production Scenario — Filesystem Becomes Read-Only

Investigate:

```text
mount status
kernel logs
storage errors
filesystem health
disk/storage state
```

Do not repeatedly retry writes.

---

# 172. Production Scenario — Deleted Log Still Uses Disk

Check:

```bash
lsof +L1
```

If a process holds the deleted file open, use the service's supported log reopen/restart procedure.

---

# 173. Production Scenario — Secret Found in Temporary Directory

Immediately:

```text
restrict access
remove exposure
determine whether it was copied elsewhere
rotate credential if exposed
review logs/artifacts
fix secret-handling workflow
```

Do not log the secret while investigating.

---

# 174. Production Scenario — Need Zero-Downtime Release

Use:

```text
versioned release
 ↓
validate
 ↓
health check
 ↓
atomic promotion
 ↓
verify
 ↓
rollback if needed
```

---

# 175. Production Scenario — Two Deployments Run Together

Use:

```text
deployment lock
```

or CI/CD concurrency control.

The system should ensure only one production promotion is active at a time.

---

# 176. Production Scenario — File Changes While Script Reads It

Determine whether the producer and consumer need coordination.

A strong pattern is:

```text
producer writes temporary file
 ↓
producer completes
 ↓
atomic rename
 ↓
consumer processes ready file
```

---

# 177. Production Scenario — Need to Modify 10,000 Files

Use:

```text
canary
 ↓
small batch
 ↓
verify
 ↓
larger batch
 ↓
monitor
 ↓
stop on abnormal failure
```

Avoid making an uncontrolled change to the entire fleet.

---

# 178. Production Scenario — Python Script Runs as Root

Minimize privileges where possible.

If root is unavoidable:

```text
strict path allowlist
input validation
dry run
bounded actions
logging
tests
locked execution
rollback
```

---

# 179. Production Scenario — Nginx Config Automation

Use:

```text
generate
 ↓
nginx -t
 ↓
install/replace
 ↓
reload
 ↓
health check
```

Never reload an unvalidated production configuration.

---

# 180. Production Scenario — Systemd Unit Automation

Use:

```text
write unit
 ↓
validate
 ↓
install
 ↓
daemon-reload
 ↓
enable/start
 ↓
systemctl status
 ↓
application health check
```

---

# 181. Production Scenario — Release Rollback

If:

```text
current -> release-42
```

and release 42 fails:

```text
current -> release-41
```

Then verify:

```text
service
endpoint
logs
metrics
```

---

# 182. Production Scenario — Filesystem Permission Fix

Never start with:

```bash
chmod -R 777
```

Instead identify:

```text
who needs access?
what operation?
which path?
which security boundary?
```

Then grant only the required permission.

---

# 183. Production Scenario — Artifact Verification

```text
artifact downloaded
 ↓
checksum calculated
 ↓
trusted checksum compared
 ↓
artifact accepted
 ↓
deployment
```

If the checksum fails:

```text
STOP
```

---

# 184. Real DevOps Project — Filesystem Health Checker

Build:

```text
filesystem_health.py
```

Input:

```bash
python filesystem_health.py \
    --path / \
    --warning 80 \
    --critical 90
```

Output:

```text
Filesystem: /
Usage: 76%
Inodes: 43%
Status: OK
```

---

# 185. Real DevOps Project — Log Cleanup Tool

Build:

```text
log_cleanup.py
```

Usage:

```bash
python log_cleanup.py \
    --path /var/log/myapp \
    --days 14 \
    --dry-run
```

Output:

```text
Candidates: 83
Total size: 2.8 GB

DRY RUN
No files deleted.
```

---

# 186. Real DevOps Project — Permission Auditor

Build:

```text
permission_audit.py
```

Check:

```text
world-writable files
private-key permissions
unexpected ownership
unexpected modes
```

Output:

```text
PASS: config.yaml
PASS: private.key
FAIL: temp.log is world writable
```

---

# 187. Real DevOps Project — Release Manager

Build:

```text
release_manager.py
```

Capabilities:

```text
create release
validate release
promote release
show current
rollback
cleanup old releases
```

---

# 188. Real DevOps Project — Backup Manager

Capabilities:

```text
backup
checksum
verify
list
restore
retention
```

---

# 189. Real DevOps Project — Incident Collector

Collect:

```text
disk
inode
process
service
network
recent logs
configuration metadata
deployment version
```

Exclude:

```text
passwords
tokens
private keys
customer data
```

---

# 190. Recommended Production Workflow

For almost every filesystem automation task:

```text
1. Understand the requirement
2. Identify exact filesystem scope
3. Validate inputs
4. Check permissions
5. Discover current state
6. Calculate intended changes
7. Dry run
8. Stage changes
9. Validate
10. Promote atomically
11. Verify
12. Log result
13. Provide rollback/recovery
```

---

# 191. Key Commands to Know

Linux:

```bash
pwd
ls
find
du
df
df -i
stat
chmod
chown
ln
cp
mv
rm
mkdir
touch
tar
gzip
rsync
lsof
mount
```

Python equivalents/orchestration:

```text
Path
os
shutil
stat
tempfile
hashlib
tarfile
gzip
subprocess
```

---

# 192. Important DevOps Principle

Do not use Python merely because Python can do something.

Use the right tool.

Examples:

```text
rsync → file synchronization
logrotate → system log rotation
systemd → service lifecycle
Kubernetes → container orchestration
Ansible → configuration management
Terraform → infrastructure provisioning
Python → logic/orchestration/integration
```

---

# 193. Strong Interview Answer

> **I use Python filesystem automation mainly for repeatable Linux operations such as directory provisioning, configuration deployment, log analysis, cleanup, backup validation, permission audits, release management, and incident diagnostics. In production I focus heavily on idempotency and safety. I validate paths, avoid unsafe recursive deletion, handle symlinks, use temporary files and atomic replacement for critical configurations, preserve required ownership and permissions, stream large files, verify checksums, log the operation, and maintain rollback or recovery options.**

---

# 194. Final Checklist

```text
[ ] pathlib
[ ] file I/O
[ ] directory operations
[ ] metadata
[ ] permissions
[ ] ownership
[ ] symlinks
[ ] path traversal
[ ] temporary files
[ ] atomic writes
[ ] copy/move
[ ] checksums
[ ] archives
[ ] disk usage
[ ] inode usage
[ ] cleanup
[ ] backups
[ ] configuration files
[ ] systemd
[ ] Nginx
[ ] Docker/EKS considerations
[ ] CI/CD
[ ] logging
[ ] error handling
[ ] locking
[ ] testing
[ ] production incidents
```

---

# 195. Final Takeaway

The purpose of Python filesystem automation in DevOps is not to replace basic Linux knowledge.

It is to combine Linux knowledge with automation:

```text
Linux filesystem knowledge
          +
Python
          +
safe automation
          +
verification
          +
production discipline
```

The strongest DevOps engineer can explain not only **how to create, modify, copy, or delete a file**, but also:

```text
where
why
under which user
with which permissions
with what safety controls
what happens if it fails
how to verify it
how to roll it back
```

That is the level expected from production DevOps automation.
