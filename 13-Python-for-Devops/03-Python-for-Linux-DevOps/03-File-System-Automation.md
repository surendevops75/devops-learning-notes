

# 50. Permission Verification

After changing permissions, verify:

```python
import stat

mode = Path(
    "/opt/app/config.yaml"
).stat().st_mode

print(stat.filemode(mode))
```

Never assume that an operation succeeded simply because the Python call returned.

---

# 51. Ownership Changes

Linux ownership can be changed with:

```python
import os

os.chown(
    "/opt/app/data",
    uid,
    gid,
)
```

This generally requires appropriate privileges.

In production, prefer a dedicated service account and explicit ownership requirements.

---

# 52. Recursive Ownership

For an entire tree, shell tools such as:

```bash
chown -R appuser:appgroup /opt/app
```

may be appropriate.

For Python automation, avoid blindly implementing recursive ownership changes unless the target is strictly validated.

Recursive ownership changes can affect:

```text
configuration
secrets
executables
mounted files
symlinks
```

---

# 53. Symbolic Links

Create:

```python
from pathlib import Path

Path(
    "/opt/app/current"
).symlink_to(
    "/opt/app/releases/v2"
)
```

This pattern is commonly used for release switching:

```text
releases/
├── v1
├── v2
└── v3

current -> v3
```

---

# 54. Symlink Deployment Pattern

A common deployment structure:

```text
/opt/myapp/
├── releases/
│   ├── 202608170900/
│   └── 202608171000/
├── shared/
│   ├── logs/
│   └── config/
└── current -> releases/202608171000
```

The service uses:

```text
/opt/myapp/current
```

A new deployment creates a new release and updates the `current` link.

---

# 55. Atomic Symlink Switching

A safer release switch can use a temporary symlink and then replace the current link.

Conceptually:

```text
new release
    ↓
temporary symlink
    ↓
atomic replacement
    ↓
current -> new release
```

This avoids leaving a partially updated release pointer.

The exact implementation must account for existing symlinks and filesystem semantics.

---

# 56. Inspect Symlink Target

```python
from pathlib import Path

path = Path(
    "/opt/app/current"
)

print(path.is_symlink())
print(path.readlink())
```

Resolve:

```python
print(path.resolve())
```

Use `readlink()` when you need the link itself rather than its resolved target.

---

# 57. Hard Links

Linux also supports hard links:

```python
Path(
    "/tmp/link"
).hardlink_to(
    "/tmp/original"
)
```

A hard link refers to the same inode.

Unlike a symbolic link:

```text
hard link -> same inode
symlink   -> path reference
```

Hard links have filesystem and directory restrictions.

---

# 58. Inodes

A file name points to an inode.

Inspect:

```python
stat = Path(
    "/tmp/file"
).stat()

print(stat.st_ino)
```

Two hard links can have:

```text
different filenames
same inode
```

This helps explain Unix filesystem behavior.

---

# 59. File Identity

For advanced checks, compare:

```python
Path(
    "/tmp/a"
).samefile(
    "/tmp/b"
)
```

This can determine whether two paths refer to the same file.

---

# 60. File Hashing

To calculate SHA-256:

```python
import hashlib

sha256 = hashlib.sha256()

with open(
    "artifact.tar.gz",
    "rb",
) as handle:

    for chunk in iter(
        lambda: handle.read(
            1024 * 1024
        ),
        b"",
    ):
        sha256.update(chunk)

print(
    sha256.hexdigest()
)
```

Useful for:

```text
artifact verification
configuration comparison
integrity checks
deployment validation
```

---

# 61. Hashing Large Files

Never assume:

```python
hashlib.sha256(
    Path("large.iso").read_bytes()
)
```

is appropriate for very large files.

Streaming chunks:

```text
read chunk
   ↓
update hash
   ↓
read next chunk
```

keeps memory consumption predictable.

---

# 62. Checksum Verification

Suppose a release provides:

```text
app.tar.gz
app.tar.gz.sha256
```

Automation can:

```text
download artifact
      ↓
calculate SHA-256
      ↓
compare expected hash
      ↓
PASS / FAIL
```

Do not deploy an artifact whose integrity verification fails.

---

# 63. File Comparison

Python provides:

```python
import filecmp

same = filecmp.cmp(
    "old.conf",
    "new.conf",
    shallow=False,
)

print(same)
```

Useful for determining whether a configuration change actually changed content.

---

# 64. Configuration Drift

A practical workflow:

```text
expected config
      |
      v
actual config
      |
      v
compare
      |
      +--> same -> no action
      |
      +--> different -> validate + update
```

This supports idempotent automation.

---

# 65. Do Not Rewrite Unchanged Files

Why?

Unnecessary rewrites can:

```text
change timestamps
trigger watchers
restart dependent services
create noisy diffs
invalidate caches
increase I/O
```

A strong automation script changes files only when required.

---

# 66. Configuration Update Pattern

```python
expected = (
    "PORT=8080\n"
    "ENV=production\n"
)

path = Path(
    "/opt/app/app.env"
)

current = (
    path.read_text(
        encoding="utf-8"
    )
    if path.exists()
    else ""
)

if current != expected:
    path.write_text(
        expected,
        encoding="utf-8",
    )
```

This is a basic idempotent pattern.

---

# 67. Configuration Validation Before Write

Never blindly write configuration.

Use:

```text
generate
   ↓
validate
   ↓
backup
   ↓
write
   ↓
validate again if required
   ↓
reload
   ↓
health check
```

For Nginx, for example:

```bash
nginx -t
```

should be performed before reloading a modified configuration.

---

# 68. Backup Before Configuration Change

```python
import shutil
from pathlib import Path

source = Path(
    "/etc/myapp/app.conf"
)

backup = Path(
    "/etc/myapp/app.conf.bak"
)

shutil.copy2(
    source,
    backup,
)
```

For critical systems, use versioned backups rather than a single `.bak` file.

---

# 69. Versioned Backups

Example:

```text
app.conf
backups/
├── app.conf.202608170900
├── app.conf.202608171000
└── app.conf.202608171100
```

This makes rollback easier.

Use a predictable naming convention and retention policy.

---

# 70. Atomic Configuration Replacement

A common pattern is:

```text
generate temporary file
        ↓
validate temporary file
        ↓
atomic replace
```

Python:

```python
import os
import tempfile
from pathlib import Path

target = Path(
    "/opt/app/app.conf"
)

with tempfile.NamedTemporaryFile(
    mode="w",
    dir=target.parent,
    delete=False,
    encoding="utf-8",
) as handle:

    handle.write(
        "PORT=8080\n"
    )

    temporary = handle.name

os.replace(
    temporary,
    target,
)
```

Always validate the temporary configuration before replacing the production file when the application supports validation.

---

# 71. Why Atomic Replacement Matters

Without atomic replacement:

```text
writer starts
   ↓
partial file
   ↓
reader opens file
   ↓
reader sees incomplete configuration
```

With atomic replacement:

```text
old file
   +
complete new file
   ↓
atomic replacement
   ↓
readers see one complete version
```

This is particularly useful for configuration and generated files.

---

# 72. File Locking

If multiple processes can modify the same file, consider locking.

Linux example:

```python
import fcntl

with open(
    "/tmp/job.lock",
    "w",
) as lock:

    fcntl.flock(
        lock,
        fcntl.LOCK_EX,
    )

    run_job()
```

Locks should be designed according to whether the workload is:

```text
single host
multiple hosts
containerized
distributed
```

---

# 73. Temporary Files

Use `tempfile`:

```python
import tempfile

with tempfile.NamedTemporaryFile(
    mode="w",
    delete=True,
) as handle:

    handle.write(
        "temporary data\n"
    )
```

Temporary files reduce the risk of leaving manually named temporary artifacts behind.

---

# 74. Temporary Directory

```python
from tempfile import TemporaryDirectory

with TemporaryDirectory() as directory:
    print(directory)
```

The directory is automatically cleaned up when the context exits.

Excellent for:

```text
tests
builds
artifact extraction
temporary transformations
```

---

# 75. Secure Temporary Files

Avoid:

```python
open(
    "/tmp/my-secret.txt",
    "w",
)
```

when a predictable filename could be attacked.

Prefer:

```python
tempfile.NamedTemporaryFile()
```

which uses safer temporary-file creation mechanisms.

---

# 76. Temporary Files and Secrets

Even temporary files can expose sensitive information.

For secrets:

```text
restrict permissions
minimize lifetime
avoid logging content
clean up
use approved secret-management mechanisms
```

If possible, avoid writing secrets to disk at all.

---

# 77. Log File Cleanup

A basic cleanup workflow:

```text
find old logs
      ↓
check age
      ↓
check file type
      ↓
check allowed directory
      ↓
dry-run
      ↓
delete/archive
```

Never delete all files matching:

```text
*.log
```

without understanding the application and retention requirements.

---

# 78. Age-Based Cleanup

```python
from pathlib import Path
import time

root = Path(
    "/var/log/myapp"
)

cutoff = (
    time.time()
    - 7 * 24 * 60 * 60
)

for path in root.glob("*.log"):
    if not path.is_file():
        continue

    if path.stat().st_mtime < cutoff:
        print(
            "Old:",
            path,
        )
```

Start with reporting before enabling deletion.

---

# 79. Dry-Run Cleanup

```python
def remove(path, dry_run=True):
    if dry_run:
        print(
            "Would remove:",
            path,
        )
        return

    path.unlink()
```

Production cleanup tools should ideally support:

```bash
--dry-run
```

---

# 80. CLI Cleanup Tool

Using `argparse`:

```python
import argparse

parser = argparse.ArgumentParser()

parser.add_argument(
    "--dry-run",
    action="store_true",
)

args = parser.parse_args()

print(
    "Dry run:",
    args.dry_run,
)
```

This makes the tool safer and easier to operate.

---

# 81. Disk Usage

```python
import shutil

total, used, free = (
    shutil.disk_usage("/")
)

print(
    "Total:",
    total,
)

print(
    "Used:",
    used,
)

print(
    "Free:",
    free,
)
```

---

# 82. Human-Readable Disk Usage

```python
def gb(value):
    return value / (
        1024 ** 3
    )

print(
    f"Used: {gb(used):.2f} GB"
)
```

For production tooling, clearly label whether units are:

```text
GB = 10^9
GiB = 1024^3
```

---

# 83. Find Largest Files

```python
from pathlib import Path

root = Path(
    "/var/log"
)

files = []

for path in root.rglob("*"):
    try:
        if path.is_file():
            files.append(
                (
                    path.stat().st_size,
                    path,
                )
            )
    except OSError:
        continue

for size, path in sorted(
    files,
    reverse=True,
)[:10]:

    print(
        size,
        path,
    )
```

For huge trees, this can be expensive.

---

# 84. Avoid Full Filesystem Scans

Do not frequently run:

```python
Path("/").rglob("*")
```

in production.

It can traverse:

```text
/proc
/sys
/dev
/run
mounted disks
network filesystems
```

Prefer targeted paths:

```text
/var/log
/opt/app
/tmp
```

and exclude special filesystems when necessary.

---

# 85. Directory Size

A recursive directory-size calculation:

```python
from pathlib import Path

def directory_size(root):
    total = 0

    for path in root.rglob("*"):
        try:
            if path.is_file():
                total += path.stat().st_size
        except OSError:
            pass

    return total
```

This is useful for targeted diagnostics but can be expensive.

---

# 86. Disk Usage vs Directory Size

Important distinction:

```text
df -h
```

measures filesystem allocation.

A Python directory walk measures visible files.

They can differ because of:

```text
deleted-open files
sparse files
mount points
reserved blocks
filesystem metadata
hard links
```

Therefore do not assume:

```text
sum of visible files = df usage
```

---

# 87. Inode Usage

A filesystem can have free disk space but no free inodes.

Linux:

```bash
df -i
```

Python can invoke:

```python
import subprocess

result = subprocess.run(
    ["df", "-i"],
    capture_output=True,
    text=True,
    check=True,
)

print(result.stdout)
```

This is particularly useful when millions of tiny files exist.

---

# 88. Small-File Explosion

Common causes:

```text
temporary files
application logs
cache files
failed jobs
mail queues
build artifacts
```

Symptoms:

```text
df -h -> enough space
df -i -> 100%
```

The remediation is to identify and safely manage the large number of files.

---

# 89. File Count

```python
from pathlib import Path

count = 0

for path in Path(
    "/var/tmp/app"
).rglob("*"):

    if path.is_file():
        count += 1

print(
    "Files:",
    count,
)
```

For very large trees, use OS-level tools or targeted analysis for better performance.

---

# 90. Log Rotation

Do not write a custom log-rotation system if the operating environment already provides:

```text
logrotate
journald
container logging
centralized logging
```

Python can integrate with those systems.

For application logs, Python's `logging.handlers` can also provide rotation when appropriate.

---

# 91. Python Rotating Logs

Example:

```python
import logging
from logging.handlers import (
    RotatingFileHandler,
)

handler = RotatingFileHandler(
    "/opt/app/app.log",
    maxBytes=50 * 1024 * 1024,
    backupCount=5,
)

logger = logging.getLogger(
    "app"
)

logger.addHandler(handler)
```

In containerized production, prefer stdout/stderr plus the platform logging pipeline when that is the established architecture.

---

# 92. File Metadata Report

```python
from pathlib import Path
import stat

path = Path(
    "/opt/app/app.conf"
)

info = path.stat()

print({
    "name": path.name,
    "size": info.st_size,
    "mode": stat.filemode(
        info.st_mode
    ),
    "uid": info.st_uid,
    "gid": info.st_gid,
    "inode": info.st_ino,
})
```

This is useful in diagnostic scripts.

---

# 93. JSON Files

```python
import json
from pathlib import Path

path = Path(
    "/opt/app/config.json"
)

data = json.loads(
    path.read_text(
        encoding="utf-8"
    )
)

print(data)
```

Writing:

```python
path.write_text(
    json.dumps(
        data,
        indent=2,
    ),
    encoding="utf-8",
)
```

---

# 94. YAML Files

Python does not include YAML in the standard library.

A common package is:

```text
PyYAML
```

Example:

```python
import yaml

with open(
    "config.yaml",
    encoding="utf-8",
) as handle:

    data = yaml.safe_load(
        handle
    )
```

Prefer:

```python
yaml.safe_load()
```

over unsafe object-deserialization APIs for untrusted YAML.

---

# 95. CSV Files

```python
import csv

with open(
    "servers.csv",
    newline="",
    encoding="utf-8",
) as handle:

    reader = csv.DictReader(
        handle
    )

    for row in reader:
        print(row)
```

Useful for:

```text
server inventories
deployment lists
migration input
reports
```

---

# 96. Configuration Validation

Example:

```python
required = {
    "environment",
    "region",
    "port",
}

missing = (
    required
    - config.keys()
)

if missing:
    raise ValueError(
        f"Missing: {missing}"
    )
```

Validate before changing system state.

---

# 97. File-Based Configuration and Secrets

Configuration may contain:

```text
database host
port
application settings
credentials
tokens
certificates
```

Separate:

```text
non-secret configuration
```

from:

```text
secret material
```

Use a secret manager where possible.

---

# 98. Certificate File Checks

A basic filesystem check:

```python
from pathlib import Path

cert = Path(
    "/etc/myapp/tls/server.crt"
)

if not cert.is_file():
    raise RuntimeError(
        "Certificate missing"
    )

if cert.stat().st_size == 0:
    raise RuntimeError(
        "Certificate is empty"
    )
```

Certificate validity itself should be checked with an appropriate TLS/certificate tool or library.

---

# 99. Private Key Permissions

Private keys should normally have restrictive permissions.

Example:

```python
Path(
    "/etc/myapp/tls/server.key"
).chmod(0o600)
```

The exact ownership and permissions depend on the service account and deployment architecture.

---

# 100. Directory Permissions for Secrets

A secret directory may use:

```python
Path(
    "/etc/myapp/secrets"
).mkdir(
    mode=0o700,
    parents=True,
    exist_ok=True,
)
```

Verify the final mode and ownership.

---

# 101. Artifact Management

DevOps scripts frequently handle:

```text
JAR
WAR
ZIP
TAR.GZ
Docker build context
configuration bundles
release packages
```

The process should include:

```text
download
verify
extract
validate
deploy
cleanup
```

---

# 102. ZIP Files

```python
import zipfile

with zipfile.ZipFile(
    "release.zip"
) as archive:

    archive.extractall(
        "/tmp/release"
    )
```

Do not blindly extract untrusted ZIP files.

Archive extraction can contain path traversal attacks.

---

# 103. Safe ZIP Extraction

Before extraction, validate each member path.

Conceptually:

```text
archive member
      ↓
resolve destination
      ↓
verify inside extraction root
      ↓
extract
```

Never assume archive contents are safe merely because the archive came from a network.

---

# 104. TAR Files

```python
import tarfile

with tarfile.open(
    "release.tar.gz",
    "r:gz",
) as archive:

    archive.extractall(
        "/tmp/release"
    )
```

Modern Python versions provide extraction filters/options that should be considered for untrusted archives.

For untrusted input, use safe extraction practices rather than blindly extracting all members.

---

# 105. Artifact Extraction Security

Potential archive attacks include:

```text
../ traversal
absolute paths
symlinks
hard links
unexpected permissions
device files
```

Production artifact extraction should:

```text
validate archive
restrict destination
run with least privilege
```

---

# 106. Artifact Integrity

Before extracting:

```text
checksum
signature
trusted source
expected artifact name
expected version
```

Verify the artifact before modifying the deployment filesystem.

---

# 107. Artifact Versioning

Use directories such as:

```text
releases/
├── 1.0.0/
├── 1.1.0/
└── 1.2.0/
```

Avoid repeatedly overwriting the same release directory.

Versioned releases make:

```text
rollback
audit
comparison
cleanup
```

easier.

---

# 108. Release Cleanup

Keep a controlled number of releases:

```text
current
previous
N older releases
```

Do not delete the active release.

A safe process:

```text
identify active release
      ↓
identify protected releases
      ↓
sort older releases
      ↓
dry-run
      ↓
delete only eligible releases
```

---

# 109. Deployment Rollback

A filesystem-based rollback can be:

```text
current -> release-3
```

change to:

```text
current -> release-2
```

Then:

```text
reload/restart
health check
```

Keep rollback artifacts until the deployment is considered stable.

---

# 110. Shared Data

Do not put mutable production data inside versioned release directories.

Better:

```text
release/
shared/
```

For example:

```text
shared/logs
shared/uploads
shared/config
```

This prevents deployments from accidentally replacing application data.

---

# 111. File Permissions During Deployment

Deployment should establish:

```text
owner
group
directory permissions
file permissions
executable permissions
```

Do not rely on whatever permissions happened to exist in the build artifact.

---

# 112. Deployment File Ownership

A typical model:

```text
root
  |
  +-- application files
  |
  +-- appuser runs service
```

Or:

```text
appuser:appgroup
```

depending on security architecture.

Use least privilege.

---

# 113. File Permissions and Git

Git tracks executable permission state for files.

A script may be:

```text
-rwxr-xr-x
```

in production but:

```text
-rw-r--r--
```

after an incorrect artifact transfer.

Validate executable permissions during deployment.

---

# 114. Executable Detection

```python
import os

path = "/opt/app/bin/start.sh"

if os.access(
    path,
    os.X_OK,
):
    print("Executable")
```

This checks accessibility for the current process identity.

---

# 115. Shebang

A Python script can start with:

```python
#!/usr/bin/env python3
```

Then:

```bash
chmod +x script.py
./script.py
```

For production service execution, an explicit interpreter path may be preferable when a controlled virtual environment is required.

---

# 116. File Descriptor-Based Operations

For security-sensitive filesystem operations, operating directly on file descriptors can reduce path-race risks.

Advanced Linux/Python APIs may include:

```text
openat-style operations
directory file descriptors
O_NOFOLLOW
O_CREAT
O_EXCL
```

Use these only when the security requirements justify the complexity.

---

# 117. Exclusive File Creation

```python
import os

fd = os.open(
    "/tmp/job.lock",
    os.O_CREAT
    | os.O_EXCL
    | os.O_WRONLY,
    0o600,
)

os.close(fd)
```

`O_EXCL` can help implement exclusive creation.

Handle:

```text
FileExistsError
```

when another process already owns the file.

---

# 118. File Existence Race

Avoid:

```python
if not path.exists():
    path.write_text("data")
```

for concurrency-sensitive creation.

Another process can create the file between the check and write.

Prefer an atomic creation mechanism such as:

```text
O_CREAT | O_EXCL
```

when appropriate.

---

# 119. Atomic Rename

On the same filesystem:

```python
import os

os.replace(
    temporary,
    target,
)
```

provides atomic replacement semantics.

This is a powerful pattern for:

```text
configuration
generated state
metadata
symlinks
```

---

# 120. Cross-Filesystem Moves

A rename is generally atomic only within the same filesystem.

Moving between:

```text
/opt
```

and:

```text
/mnt
```

may involve copying.

Do not assume:

```python
shutil.move()
```

always gives an atomic operation.

---

# 121. Filesystem Mounts

A path can cross filesystem boundaries.

Check:

```bash
df -T /opt/app
```

or:

```python
os.statvfs(
    "/opt/app"
)
```

Understanding mounts matters for:

```text
disk usage
performance
atomic rename
permissions
container volumes
```

---

# 122. Mount Points

Python can inspect filesystem information using:

```python
import os

stats = os.statvfs(
    "/var/log"
)

print(
    "Block size:",
    stats.f_bsize,
)

print(
    "Free blocks:",
    stats.f_bavail,
)
```

This is useful for lower-level filesystem health checks.

---

# 123. Read-Only Filesystems

A write can fail even when permissions look correct if the filesystem is mounted read-only.

Symptoms:

```text
Read-only file system
```

Investigate:

```bash
mount
df -h
dmesg
journalctl -k
```

Do not repeatedly retry writes against a read-only filesystem.

---

# 124. Disk Full vs Permission Denied

These are different failures.

Disk full:

```text
No space left on device
```

Permission:

```text
Permission denied
```

Filesystem automation should catch and report them separately.

---

# 125. Exception Handling

Filesystem operations can raise:

```python
FileNotFoundError
PermissionError
FileExistsError
IsADirectoryError
NotADirectoryError
OSError
```

Example:

```python
try:
    path.unlink()
except FileNotFoundError:
    logger.info(
        "Already absent: %s",
        path,
    )
except PermissionError:
    logger.error(
        "Permission denied: %s",
        path,
    )
    raise
```

Do not catch every error and silently continue.

---

# 126. `OSError` as Base

Many filesystem exceptions inherit from:

```text
OSError
```

You can catch:

```python
except OSError as exc:
    logger.error(
        "Filesystem operation failed: %s",
        exc,
    )
```

But specific exceptions are often better when recovery behavior differs.

---

# 127. Error Reporting

Bad:

```python
except Exception:
    print("failed")
```

Better:

```python
except PermissionError as exc:
    logger.error(
        "Cannot read %s: %s",
        path,
        exc,
    )
    raise
```

Operators need enough context to diagnose the problem.

---

# 128. Logging Filesystem Changes

Useful:

```python
logger.info(
    "Removing old release: %s",
    path,
)
```

Do not log:

```text
secret contents
private keys
tokens
passwords
```

---

# 129. Auditability

For production automation, record:

```text
what changed
when
which host
which script/version
which operator/job
result
```

For sensitive systems, integrate with the organization's audit mechanism.

---

# 130. Dry-Run as a Standard Feature

For destructive scripts:

```bash
python cleanup.py --dry-run
```

should report:

```text
would remove:
  /var/tmp/app/a.log
  /var/tmp/app/b.log
```

Then:

```bash
python cleanup.py
```

performs the change.

---

# 131. Idempotency

A filesystem automation task should ideally converge:

```text
desired state
```

Example:

```python
directory.mkdir(
    parents=True,
    exist_ok=True,
)
```

For configuration:

```text
compare
   ↓
change only if different
```

For symlinks:

```text
verify target
   ↓
update only if needed
```

---

# 132. Convergence

Think of automation as:

```text
Current state
      |
      v
Desired state
      |
      v
Minimal required change
      |
      v
Desired state
```

This is the same philosophy behind tools such as:

```text
Ansible
Terraform
Kubernetes
```

---

# 133. Filesystem Automation and Ansible

Ansible provides:

```yaml
- name: Ensure application directory exists
  ansible.builtin.file:
    path: /opt/myapp
    state: directory
    owner: appuser
    group: appgroup
    mode: "0750"
```

Python is useful for:

```text
custom checks
data transformation
specialized workflows
API integrations
```

Do not recreate a full configuration-management engine unnecessarily.

---

# 134. Filesystem Automation and Terraform

Terraform can provision:

```text
infrastructure
storage
instances
permissions
```

Python can perform:

```text
post-provision checks
artifact preparation
configuration validation
```

Avoid modifying Terraform-managed infrastructure behind Terraform's back.

---

# 135. Filesystem Automation in CI/CD

A deployment pipeline may:

```text
checkout
   ↓
build
   ↓
create artifact
   ↓
checksum
   ↓
publish
   ↓
download
   ↓
extract
   ↓
validate
   ↓
deploy
   ↓
health check
```

Python can automate many of the filesystem-related stages.

---

# 136. Artifact Cleanup in CI

CI workspaces can accumulate:

```text
builds
logs
test reports
temporary files
dependency caches
```

Cleanup should use:

```text
workspace-specific root
retention policy
dry-run
age/size rules
```

Never recursively delete arbitrary filesystem paths from CI variables.

---

# 137. Jenkins Workspace Cleanup

A Python helper can identify:

```text
files older than N days
large artifacts
temporary directories
```

But Jenkins itself may provide workspace-cleanup capabilities.

Use the platform feature when it meets the requirement.

---

# 138. GitHub Actions Workspace

GitHub-hosted runners are typically ephemeral.

Do not build complicated local cleanup logic for resources that disappear with the runner.

For self-hosted runners, cleanup matters much more.

---

# 139. Self-Hosted Runner Cleanup

A Python cleanup job may monitor:

```text
workspace size
Docker images
temporary files
build artifacts
old logs
```

It should protect:

```text
runner configuration
active jobs
required caches
system files
```

---

# 140. Docker Storage

Container hosts can accumulate:

```text
images
containers
volumes
build cache
logs
```

Inspect with:

```bash
docker system df
```

Do not blindly run:

```bash
docker system prune -a
```

in production.

Understand what data is safe to remove.

---

# 141. Kubernetes Node Storage

Node filesystem pressure can come from:

```text
container images
container logs
ephemeral storage
emptyDir
dead containers
application files
```

Python can collect diagnostics, but Kubernetes should remain the primary control plane.

---

# 142. Log File vs Deleted File

If an application keeps a deleted log file open:

```text
rm file
```

does not immediately reclaim the blocks while the file descriptor remains open.

Use:

```bash
lsof +L1
```

to investigate.

The appropriate remediation may be:

```text
log rotation fix
application restart
FD close
```

depending on the service.

---

# 143. Sparse Files

A sparse file can have a large logical size but consume fewer physical blocks.

Therefore:

```text
st_size
```

and actual disk allocation can differ.

For disk investigations, distinguish:

```text
logical size
physical blocks
filesystem usage
```

---

# 144. Hard Links and Disk Usage

If two filenames point to the same inode:

```text
fileA -> inode 100
fileB -> inode 100
```

counting both file sizes can overestimate actual storage usage.

This is another reason simple recursive size scripts may not exactly match filesystem allocation.

---

# 145. File Descriptor and Disk Incident

Production example:

```text
df -h -> 99%
visible logs -> only 40 GB
```

Investigation:

```bash
lsof +L1
```

finds:

```text
application PID 4000
deleted app.log
open FD
```

The application still holds the deleted file.

This is a classic Linux troubleshooting scenario.

---

# 146. Log Rotation Automation

A correct architecture is:

```text
application
   ↓
writes logs
   ↓
rotation mechanism
   ↓
compressed/retained logs
   ↓
centralized logging
```

Avoid writing a custom rotation system when:

```text
logrotate
journald
container runtime
ELK pipeline
```

already handles the requirement.

---

# 147. File Compression

Python:

```python
import gzip

with gzip.open(
    "app.log.gz",
    "wt",
    encoding="utf-8",
) as handle:
    handle.write(
        "log content\n"
    )
```

For large files, stream line-by-line rather than loading the entire file.

---

# 148. Tar Archive

Create:

```python
import tarfile

with tarfile.open(
    "release.tar.gz",
    "w:gz",
) as archive:

    archive.add(
        "/opt/app/release",
        arcname="release",
    )
```

Be careful about including:

```text
secrets
temporary files
logs
credentials
```

---

# 149. ZIP Archive

```python
from pathlib import Path
import zipfile

root = Path(
    "/opt/app/release"
)

with zipfile.ZipFile(
    "release.zip",
    "w",
    compression=zipfile.ZIP_DEFLATED,
) as archive:

    for path in root.rglob("*"):
        if path.is_file():
            archive.write(
                path,
                path.relative_to(root),
            )
```

---

# 150. Exclude Sensitive Files

Before packaging:

```text
.env
private keys
credentials
logs
temporary files
.git
```

should be explicitly considered.

Use an allowlist of files/directories for sensitive deployment artifacts when possible.

---

# 151. Build Artifact Manifest

A useful release artifact can include:

```text
manifest.json
```

containing:

```json
{
  "version": "1.2.0",
  "build": "1842",
  "commit": "abc123",
  "sha256": "..."
}
```

This improves traceability.

---

# 152. Manifest Validation

Deployment can verify:

```text
expected version
commit
artifact checksum
required files
```

before activation.

This prevents deploying incomplete artifacts.

---

# 153. Required Files Check

```python
required = [
    Path("bin/start.sh"),
    Path("config/app.yaml"),
]

for path in required:
    if not path.is_file():
        raise RuntimeError(
            f"Missing artifact file: {path}"
        )
```

This should happen before deployment.

---

# 154. File Permission Validation

```python
import stat

mode = Path(
    "bin/start.sh"
).stat().st_mode

if not (
    mode & stat.S_IXUSR
):
    raise RuntimeError(
        "start.sh is not executable"
    )
```

---

# 155. Configuration File Validation

Check:

```text
exists
regular file
not empty
expected owner
expected permissions
valid syntax
required keys
```

Then deploy.

---

# 156. Prevent Empty Configuration

```python
path = Path(
    "/opt/app/config.yaml"
)

if not path.is_file():
    raise RuntimeError(
        "Config missing"
    )

if path.stat().st_size == 0:
    raise RuntimeError(
        "Config is empty"
    )
```

An empty configuration can be much worse than a missing configuration because it may appear valid to a simplistic deployment check.

---

# 157. File Age Validation

```python
import time

age = (
    time.time()
    - path.stat().st_mtime
)

if age > 3600:
    print(
        "Configuration is older than 1 hour"
    )
```

Use age checks only when the business/operational requirement makes sense.

---

# 158. Detect Unexpected Changes

A checksum can be stored:

```text
known hash
```

Then compare:

```python
current_hash = calculate_hash(
    path
)

if current_hash != expected_hash:
    raise RuntimeError(
        "Unexpected file change"
    )
```

Useful for:

```text
critical configuration
deployment artifacts
integrity checks
```

---

# 159. Filesystem Security Checklist

```text
[ ] Validate paths
[ ] Avoid arbitrary recursive deletion
[ ] Use least privilege
[ ] Protect secrets
[ ] Avoid predictable temporary files
[ ] Validate archives
[ ] Verify checksums
[ ] Avoid unsafe symlink handling
[ ] Use atomic replacement
[ ] Avoid chmod 777
[ ] Check ownership
[ ] Check permissions
[ ] Log changes
[ ] Support dry-run
```

---

# 160. Production File Change Workflow

```text
Receive change
      ↓
Validate input
      ↓
Check current state
      ↓
Generate new content
      ↓
Validate new content
      ↓
Create backup
      ↓
Atomic replacement
      ↓
Validate final state
      ↓
Reload service
      ↓
Health check
      ↓
Log result
```

This is a strong pattern for production configuration automation.

---

# 161. Production Scenario — Disk 100%

Situation:

```text
/var = 100%
```

First:

```bash
df -h /var
df -i /var
```

Then investigate:

```text
largest directories
large files
logs
container storage
deleted-open files
```

Do not immediately delete everything under `/var`.

---

# 162. Production Scenario — Inodes 100%

Situation:

```text
df -h -> 40%
df -i -> 100%
```

Likely causes:

```text
millions of small files
cache
temporary files
mail queues
application artifacts
```

Find the directory with the highest file count and apply the correct retention policy.

---

# 163. Production Scenario — Configuration Breaks Nginx

Workflow:

```text
configuration changed
      ↓
nginx fails
      ↓
nginx -t
      ↓
identify invalid config
      ↓
restore/fix
      ↓
nginx -t
      ↓
systemctl reload nginx
      ↓
health check
```

Never reload a known-invalid configuration.

---

# 164. Production Scenario — Deployment Artifact Missing File

Before activating:

```python
required_files = [
    "bin/start.sh",
    "config/app.yaml",
]
```

Validate all required paths.

If any are missing:

```text
fail deployment
```

rather than activating a partial release.

---

# 165. Production Scenario — Wrong Permissions

Application fails:

```text
Permission denied
```

Check:

```bash
namei -l /opt/app/config.yaml
```

and:

```text
owner
group
directory permissions
file permissions
SELinux
```

Do not immediately use:

```bash
chmod -R 777
```

---

# 166. Production Scenario — Symlink Points to Missing Release

Check:

```python
current = Path(
    "/opt/app/current"
)

print(
    current.is_symlink()
)

print(
    current.resolve()
)
```

If the target is missing:

```text
rollback to known-good release
```

according to the deployment policy.

---

# 167. Production Scenario — Cleanup Deletes Active Release

Prevention:

```text
read current symlink
      ↓
resolve active release
      ↓
mark protected
      ↓
exclude active release
      ↓
delete only old releases
```

Never let cleanup logic assume that the newest directory is always the active release.

---

# 168. Production Scenario — Artifact Checksum Failure

If:

```text
expected SHA-256
!=
calculated SHA-256
```

then:

```text
stop deployment
do not extract
do not activate
investigate artifact source
```

Integrity verification should happen before modifying the target system.

---

# 169. Production Scenario — Read-Only Filesystem

If Python reports:

```text
OSError: [Errno 30] Read-only file system
```

investigate:

```text
mount state
filesystem errors
kernel logs
storage health
```

Do not keep retrying the same write operation.

---

# 170. Production Scenario — Permission Denied

Check:

```text
effective user
group
file mode
parent directories
ACL
SELinux/AppArmor
mount options
```

Python:

```python
import os

print(
    os.geteuid()
)

print(
    os.getegid()
)
```

---

# 171. Production Scenario — Cron Cleanup Fails

Compare:

```text
manual shell environment
```

with:

```text
cron environment
```

Check:

```text
PATH
working directory
Python interpreter
permissions
environment variables
```

Use absolute paths.

---

# 172. Production Scenario — Systemd Script Fails

Inspect:

```bash
systemctl status myapp
journalctl -u myapp
systemctl cat myapp
```

Common causes:

```text
wrong WorkingDirectory
wrong ExecStart
wrong user
missing environment
wrong Python interpreter
missing permissions
```

---

# 173. Production Scenario — File Changes Trigger Unexpected Restart

Possible cause:

```text
file watcher
```

Applications may watch:

```text
configuration
certificates
directories
```

Avoid rewriting unchanged files.

This is another reason idempotent filesystem automation matters.

---

# 174. Production Scenario — Application Reads Partial Config

Cause:

```text
direct write to active file
```

Safer pattern:

```text
write temp
validate
atomic replace
```

Then reload the service.

---

# 175. Production Scenario — Secret File Exposed

If a secret file has overly broad permissions:

```text
identify access
      ↓
restrict permissions
      ↓
check ownership
      ↓
rotate secret if exposure occurred
      ↓
audit access
```

Do not assume changing permissions alone eliminates prior exposure.

---

# 176. Production Scenario — Temporary Files Fill Disk

Find:

```text
/tmp
/var/tmp
application-specific temp directories
```

Then determine:

```text
which process creates them
why they aren't removed
retention policy
safe cleanup
```

Fix the producer when possible instead of repeatedly cleaning symptoms.

---

# 177. Production Scenario — Large Log File

A single log reaches:

```text
80 GB
```

Do not simply:

```bash
rm app.log
```

Investigate:

```text
log rotation
application logging rate
disk usage
open file descriptor
central logging
```

If an application still has the file open, deleting it may not reclaim space immediately.

---

# 178. Production Scenario — Deleted File Still Uses Disk

Check:

```bash
lsof +L1
```

Identify:

```text
PID
file
size
FD
```

Then use the approved application/service procedure to close the descriptor.

---

# 179. Production Scenario — Too Many Small Files

Investigate:

```text
inode usage
file count
directory distribution
application behavior
cache/temporary retention
```

Use:

```bash
df -i
```

before assuming the issue is disk capacity.

---

# 180. Production Scenario — Archive Extraction Failure

Possible causes:

```text
corrupt archive
permission
disk full
inode exhaustion
path traversal protection
unexpected archive structure
```

Check:

```text
checksum
archive contents
target filesystem
permissions
available space
```

---

# 181. Production Scenario — Deployment Rollback

If release `v3` is unhealthy:

```text
current -> v3
```

rollback:

```text
current -> v2
```

Then:

```text
reload/restart
health check
metrics
logs
```

Keep evidence from the failed release for post-incident analysis.

---

# 182. Production Scenario — Cleanup Job Is Dangerous

Before production execution:

```text
unit tests
test directory
staging host
dry-run
peer review
limited permissions
monitoring
rollback/recovery plan
```

Filesystem deletion deserves the same engineering discipline as infrastructure changes.

---

# 183. Python Filesystem Utility Pattern

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

Keep utilities small and reusable.

---

# 184. Safe File Reader

```python
from pathlib import Path


def read_text(path):
    return Path(path).read_text(
        encoding="utf-8",
        errors="replace",
    )
```

For large files, use streaming instead.

---

# 185. Safe File Writer

```python
from pathlib import Path


def write_text(path, content):
    target = Path(path)

    target.parent.mkdir(
        parents=True,
        exist_ok=True,
    )

    target.write_text(
        content,
        encoding="utf-8",
    )
```

For critical configuration, upgrade this to an atomic write workflow.

---

# 186. Atomic Writer Function

```python
import os
import tempfile
from pathlib import Path


def atomic_write(path, content):
    target = Path(path)

    with tempfile.NamedTemporaryFile(
        mode="w",
        dir=target.parent,
        delete=False,
        encoding="utf-8",
    ) as handle:

        handle.write(content)
        temporary = handle.name

    os.replace(
        temporary,
        target,
    )
```

In production, also consider:

```text
permissions
ownership
fsync requirements
validation
failure cleanup
```

depending on durability requirements.

---

# 187. Safe Delete Function

```python
from pathlib import Path


def delete_file(path):
    target = Path(path)

    if not target.is_file():
        return False

    target.unlink()

    return True
```

This is intentionally narrow.

Do not turn every helper into a generic recursive deletion function.

---

# 188. Find Large Files Function

```python
from pathlib import Path


def find_large_files(
    root,
    minimum_bytes,
):
    root = Path(root)

    results = []

    for path in root.rglob("*"):
        try:
            if (
                path.is_file()
                and path.stat().st_size
                >= minimum_bytes
            ):
                results.append(path)
        except OSError:
            continue

    return results
```

Use targeted roots and appropriate error handling.

---

# 189. Old File Detection

```python
import time
from pathlib import Path


def is_older_than(
    path,
    seconds,
):
    age = (
        time.time()
        - Path(path).stat().st_mtime
    )

    return age > seconds
```

This is useful for:

```text
cleanup
artifact retention
stale lock detection
temporary files
```

---

# 190. Stale Lock Files

A lock file may remain after an abnormal process termination.

Never assume:

```text
lock file exists
=
job is running
```

Validate the lock mechanism and process state.

If using PID-based locking, verify:

```text
PID exists
expected command
expected user
```

---

# 191. File-Based State

A small automation task may store:

```json
{
  "last_run": "2026-08-17T10:00:00Z",
  "status": "success"
}
```

Use atomic writes so a crash does not leave corrupt state.

---

# 192. State File Corruption

If a state file is invalid:

```text
parse
  ↓
validation fails
  ↓
do not blindly overwrite
  ↓
backup/quarantine
  ↓
recover known-good state
```

The correct recovery depends on the application.

---

# 193. File Permissions in State Files

State files may contain operational information.

Set appropriate permissions:

```python
Path(
    "/var/lib/myapp/state.json"
).chmod(0o640)
```

Ownership should match the service account and operational requirements.

---

# 194. `/var/lib` vs `/tmp`

Use:

```text
/tmp
```

for temporary data.

Use an appropriate persistent application directory such as:

```text
/var/lib/myapp
```

for state that must survive reboot.

Do not store persistent application state in `/tmp`.

---

# 195. `/var/log`

Traditional Linux application logs may be under:

```text
/var/log
```

But modern systems may use:

```text
journald
container stdout/stderr
centralized logging
```

Always understand the platform's logging architecture.

---

# 196. `/etc`

Configuration is commonly stored under:

```text
/etc
```

Do not modify files there without:

```text
backup
validation
appropriate privileges
change control
```

---

# 197. `/opt`

Third-party or custom application deployments commonly use:

```text
/opt/myapp
```

A structured layout:

```text
/opt/myapp/
├── releases/
├── current
├── shared/
└── scripts/
```

can simplify release management.

---

# 198. `/usr/local`

Locally installed software may be placed under:

```text
/usr/local
```

Avoid mixing application-managed files with distribution-managed package files without a clear strategy.

---

# 199. `/run`

Runtime state is commonly stored under:

```text
/run
```

It is generally temporary and recreated during boot.

Do not use `/run` for persistent application data.

---

# 200. `/tmp` Security

`/tmp` is commonly shared.

Therefore:

```text
predictable filenames
world-readable data
temporary secrets
unsafe symlinks
```

can create security problems.

Use secure temporary-file APIs.

---

# 201. File-System Automation Performance

For performance:

```text
avoid unnecessary scans
stream large files
use pathlib efficiently
avoid repeated stat calls
batch work where possible
limit concurrency
avoid unnecessary subprocesses
```

A filesystem script can itself become a production performance problem.

---

# 202. `Path.stat()` Cost

Every `stat()` call can involve filesystem work.

If scanning millions of files:

```python
path.stat()
```

for every operation can become expensive.

Use directory iteration APIs that expose metadata where appropriate, or use OS-level tools for very large-scale inventory tasks.

---

# 203. `os.scandir()`

For high-performance directory traversal:

```python
import os

with os.scandir(
    "/var/log"
) as entries:

    for entry in entries:
        print(
            entry.name,
            entry.is_file(),
        )
```

`scandir()` can be more efficient than repeatedly constructing paths and calling `stat()`.

---

# 204. Recursive `os.walk()`

```python
import os

for root, dirs, files in os.walk(
    "/opt/app"
):
    for name in files:
        print(
            os.path.join(
                root,
                name,
            )
        )
```

`os.walk()` is useful for traditional recursive filesystem automation.

---

# 205. Prune Directories

With `os.walk()`:

```python
for root, dirs, files in os.walk(
    "/opt/app"
):
    dirs[:] = [
        d for d in dirs
        if d not in {
            ".git",
            "node_modules",
        }
    ]

    for name in files:
        print(
            os.path.join(
                root,
                name,
            )
        )
```

Pruning unnecessary directories can dramatically reduce scan time.

---

# 206. Symlink Traversal

Be careful when recursively traversing symlinks.

Following symlinks can:

```text
escape intended directory
create cycles
scan unexpected filesystems
```

Prefer not to follow symlinks unless explicitly required.

---

# 207. Filesystem Race Conditions

A path can change after inspection:

```text
check path
   ↓
another process changes path
   ↓
operation acts on different object
```

This is a TOCTOU problem:

```text
Time Of Check
      vs
Time Of Use
```

Use atomic OS operations where possible.

---

# 208. Atomicity

Important filesystem operations include:

```text
rename/replace
exclusive create
link operations
file descriptor operations
```

Understand which operations are atomic on the filesystem you are using.

---

# 209. Durability vs Atomicity

Atomicity means:

```text
operation appears all-at-once
```

Durability means:

```text
data survives failure according to storage guarantees
```

They are not the same.

For critical data, additional synchronization such as:

```text
flush
fsync
directory fsync
```

may be required depending on the durability requirement.

---

# 210. Production Data Safety

Before automating changes to:

```text
database data directories
persistent volumes
application uploads
certificates
system configuration
```

understand:

```text
backup
consistency
application locking
filesystem
recovery
```

Do not treat arbitrary filesystem copying as a valid database backup.

---

# 211. Database Files

Never copy a live database's internal data directory casually.

Use:

```text
database-native backup
snapshot
replication
approved backup tool
```

Filesystem automation can coordinate these operations but should not replace database consistency mechanisms.

---

# 212. Persistent Volumes

In Kubernetes:

```text
Pod
  ↓
PersistentVolumeClaim
  ↓
PersistentVolume
  ↓
storage backend
```

A Python script inside the Pod should not assume it owns the entire underlying filesystem.

Respect:

```text
mount path
storage class
access mode
application ownership
```

---

# 213. NFS and Network Filesystems

Filesystem operations on NFS can have different performance and locking characteristics.

Potential issues:

```text
latency
stale handles
locking
availability
network failures
```

Do not assume local-disk behavior.

---

# 214. Filesystem Monitoring

Useful indicators:

```text
disk usage
inode usage
directory growth
file count
I/O latency
open files
read-only state
mount availability
```

Python can collect custom metrics, but established exporters should be preferred when available.

---

# 215. Alerting on Disk

A good alert should consider:

```text
percentage used
absolute free space
inode usage
growth rate
filesystem type
business criticality
```

Example:

```text
95% full
```

may be acceptable for one filesystem and critical for another.

---

# 216. Growth-Rate Analysis

A filesystem at:

```text
70%
```

may be more dangerous than one at:

```text
90%
```

if the first is growing:

```text
5% per hour
```

while the second is stable.

Automated reports can include:

```text
current usage
historical usage
growth rate
estimated exhaustion time
```

---

# 217. Cleanup Policy

Never make cleanup based only on:

```text
"disk is high"
```

Define:

```text
what can be deleted
when
how much
retention
approval
exceptions
```

Examples:

```text
temporary files > 7 days
build artifacts > 14 days
old releases > 5 versions
logs according to retention policy
```

---

# 218. Cleanup Escalation

A safer policy:

```text
disk < 80%
    ↓
no cleanup

80-90%
    ↓
report

90-95%
    ↓
approved cleanup

>95%
    ↓
urgent operational response
```

Actual thresholds should be based on the environment.

---

# 219. Backup Before Cleanup

For important files:

```text
identify
   ↓
backup/archive
   ↓
verify backup
   ↓
delete
```

Do not assume that copying a file automatically means a recoverable backup exists.

---

# 220. Backup Verification

At minimum verify:

```text
backup exists
size reasonable
checksum if appropriate
archive readable
destination available
```

For critical systems, perform restore testing.

---

# 221. Production File Automation Checklist

```text
[ ] Target path explicit
[ ] Path validated
[ ] Current state inspected
[ ] File type validated
[ ] Ownership checked
[ ] Permissions checked
[ ] Existing data protected
[ ] Backup considered
[ ] Dry-run available
[ ] Idempotency considered
[ ] Atomic update used where appropriate
[ ] Errors handled
[ ] Logging enabled
[ ] Secrets protected
[ ] Result verified
[ ] Rollback possible
```

---

# 222. Daily DevOps Script — Find Large Logs

```python
#!/usr/bin/env python3

from pathlib import Path

ROOT = Path(
    "/var/log/myapp"
)

LIMIT = (
    500 * 1024 * 1024
)

for path in ROOT.rglob("*.log"):
    try:
        if (
            path.is_file()
            and path.stat().st_size
            >= LIMIT
        ):
            print(path)
    except OSError:
        continue
```

This reports rather than deletes.

---

# 223. Daily DevOps Script — Old Temporary Files

```python
#!/usr/bin/env python3

from pathlib import Path
import time

ROOT = Path(
    "/var/tmp/myapp"
)

AGE = (
    7 * 24 * 60 * 60
)

cutoff = (
    time.time() - AGE
)

for path in ROOT.iterdir():
    try:
        if (
            path.is_file()
            and path.stat().st_mtime
            < cutoff
        ):
            print(
                "Old:",
                path,
            )
    except OSError:
        continue
```

Add dry-run and explicit deletion only after testing.

---

# 224. Daily DevOps Script — Required Files

```python
#!/usr/bin/env python3

from pathlib import Path

required = [
    Path("/opt/myapp/bin/start.sh"),
    Path("/opt/myapp/config/app.yaml"),
]

missing = [
    str(path)
    for path in required
    if not path.is_file()
]

if missing:
    print(
        "Missing files:"
    )

    for path in missing:
        print(path)

    raise SystemExit(1)

print("Required files present")
```

Useful as a deployment precheck.

---

# 225. Daily DevOps Script — Configuration Check

```python
#!/usr/bin/env python3

from pathlib import Path

config = Path(
    "/opt/myapp/config/app.yaml"
)

if not config.is_file():
    print(
        "ERROR: configuration missing"
    )
    raise SystemExit(1)

if config.stat().st_size == 0:
    print(
        "ERROR: configuration empty"
    )
    raise SystemExit(1)

print(
    "Configuration file exists"
)
```

---

# 226. Daily DevOps Script — Artifact Checksum

```python
#!/usr/bin/env python3

import hashlib
from pathlib import Path

artifact = Path(
    "release.tar.gz"
)

sha256 = hashlib.sha256()

with artifact.open("rb") as handle:
    for chunk in iter(
        lambda: handle.read(
            1024 * 1024
        ),
        b"",
    ):
        sha256.update(chunk)

print(
    sha256.hexdigest()
)
```

---

# 227. Daily DevOps Script — Release Directory

```python
from pathlib import Path

release = Path(
    "/opt/myapp/releases/1.2.0"
)

release.mkdir(
    parents=True,
    exist_ok=False,
)

print(
    "Created:",
    release,
)
```

Using `exist_ok=False` intentionally fails if the release already exists, which can prevent accidental overwriting.

---

# 228. Daily DevOps Script — Current Release

```python
from pathlib import Path

current = Path(
    "/opt/myapp/current"
)

if not current.is_symlink():
    raise RuntimeError(
        "current is not a symlink"
    )

target = current.resolve()

print(
    "Current release:",
    target,
)
```

---

# 229. Daily DevOps Script — File Integrity

```python
import hashlib
from pathlib import Path


def sha256_file(path):
    digest = hashlib.sha256()

    with Path(path).open(
        "rb"
    ) as handle:

        for chunk in iter(
            lambda: handle.read(
                1024 * 1024
            ),
            b"",
        ):
            digest.update(chunk)

    return digest.hexdigest()
```

This helper can be reused in artifact pipelines.

---

# 230. Daily DevOps Script — Directory Inventory

```python
from pathlib import Path

root = Path(
    "/opt/myapp"
)

for path in root.iterdir():
    print({
        "path": str(path),
        "file": path.is_file(),
        "directory": path.is_dir(),
        "symlink": path.is_symlink(),
    })
```

Useful for diagnostics.

---

# 231. Daily DevOps Script — Permission Audit

```python
from pathlib import Path
import stat

root = Path(
    "/opt/myapp/config"
)

for path in root.rglob("*"):
    try:
        if path.is_file():
            mode = stat.filemode(
                path.stat().st_mode
            )

            print(
                mode,
                path,
            )
    except OSError:
        continue
```

Extend this with approved permission policies.

---

# 232. Daily DevOps Script — Secret Permission Audit

```python
from pathlib import Path

root = Path(
    "/etc/myapp/secrets"
)

for path in root.rglob("*"):
    if not path.is_file():
        continue

    mode = (
        path.stat().st_mode
        & 0o777
    )

    if mode & 0o077:
        print(
            "Potentially broad permissions:",
            path,
            oct(mode),
        )
```

This is a starting point, not a complete secrets-security audit.

---

# 233. Daily DevOps Script — Find Empty Files

```python
from pathlib import Path

root = Path(
    "/opt/myapp"
)

for path in root.rglob("*"):
    try:
        if (
            path.is_file()
            and path.stat().st_size == 0
        ):
            print(path)
    except OSError:
        continue
```

Useful for finding:

```text
empty configuration
failed artifacts
unexpected files
```

---

# 234. Daily DevOps Script — File Count by Extension

```python
from collections import Counter
from pathlib import Path

root = Path(
    "/opt/myapp"
)

counts = Counter()

for path in root.rglob("*"):
    if path.is_file():
        counts[
            path.suffix or "<none>"
        ] += 1

for extension, count in (
    counts.most_common()
):
    print(
        extension,
        count,
    )
```

Useful for identifying unexpected file growth.

---

# 235. Daily DevOps Script — Directory Size Report

```python
from pathlib import Path

def directory_size(root):
    total = 0

    for path in Path(root).rglob("*"):
        try:
            if path.is_file():
                total += path.stat().st_size
        except OSError:
            continue

    return total


size = directory_size(
    "/opt/myapp"
)

print(
    f"{size / 1024**3:.2f} GiB"
)
```

For huge trees, use optimized tools or filesystem metrics.

---

# 236. Daily DevOps Script — Stale Files

```python
from pathlib import Path
import time

root = Path(
    "/opt/myapp/tmp"
)

threshold = (
    time.time()
    - 24 * 60 * 60
)

for path in root.iterdir():
    try:
        if (
            path.is_file()
            and path.stat().st_mtime
            < threshold
        ):
            print(
                "Stale:",
                path,
            )
    except OSError:
        continue
```

---

# 237. Daily DevOps Script — Deployment Preflight

```python
from pathlib import Path
import shutil

required_commands = [
    "systemctl",
    "curl",
]

for command in required_commands:
    if not shutil.which(command):
        raise RuntimeError(
            f"Missing command: {command}"
        )

required_paths = [
    Path("/opt/myapp"),
    Path("/opt/myapp/config"),
]

for path in required_paths:
    if not path.is_dir():
        raise RuntimeError(
            f"Missing directory: {path}"
        )

print(
    "Filesystem preflight passed"
)
```

---

# 238. Daily DevOps Script — Backup Configuration

```python
from datetime import datetime, timezone
from pathlib import Path
import shutil

source = Path(
    "/opt/myapp/config/app.yaml"
)

timestamp = datetime.now(
    timezone.utc
).strftime(
    "%Y%m%d%H%M%S"
)

backup = Path(
    f"/opt/myapp/backups/"
    f"app.yaml.{timestamp}"
)

backup.parent.mkdir(
    parents=True,
    exist_ok=True,
)

shutil.copy2(
    source,
    backup,
)

print(
    "Backup:",
    backup,
)
```

Use a retention policy to prevent backup directories from growing indefinitely.

---

# 239. Daily DevOps Script — Cleanup Old Backups

```python
from pathlib import Path
import time

root = Path(
    "/opt/myapp/backups"
)

age = (
    30 * 24 * 60 * 60
)

cutoff = (
    time.time() - age
)

for path in root.glob(
    "app.yaml.*"
):
    try:
        if (
            path.is_file()
            and path.stat().st_mtime
            < cutoff
        ):
            print(
                "Would remove:",
                path,
            )
    except OSError:
        continue
```

Use dry-run first and keep a minimum number of backups where required.

---

# 240. Production File Automation Architecture

A mature filesystem automation service may look like:

```text
Git
 |
 v
CI/CD
 |
 +--> Unit tests
 +--> Security scan
 +--> Dependency scan
 |
 v
Versioned artifact
 |
 v
Artifact repository
 |
 v
Deployment
 |
 +--> preflight
 +--> backup
 +--> extract
 +--> validate
 +--> atomic switch
 +--> service reload
 +--> health check
 |
 v
Metrics / Logs
 |
 +--> Prometheus
 +--> Grafana
 +--> ELK
```

Python can act as the custom automation layer within this architecture.

---

# 241. Testing Filesystem Automation

Test against:

```text
temporary directories
mock filesystems
test VMs
containers
staging environments
```

Avoid testing destructive logic directly against:

```text
/etc
/var
production volumes
```

---

# 242. Temporary Directory Tests

```python
from pathlib import Path
from tempfile import TemporaryDirectory

with TemporaryDirectory() as directory:
    root = Path(directory)

    app = root / "app"

    app.mkdir()

    file = app / "config.txt"

    file.write_text(
        "test\n"
    )

    assert file.is_file()
```

This gives each test an isolated filesystem area.

---

# 243. Test Idempotency

Example:

```python
ensure_directory(
    root / "app"
)

ensure_directory(
    root / "app"
)
```

The second execution should not fail.

Test:

```text
first run
second run
third run
```

and verify the final state remains correct.

---

# 244. Test Destructive Operations

Test cases should include:

```text
valid target
missing target
wrong file type
outside allowed root
permission denied
already deleted
symlink
empty directory
non-empty directory
```

Security-sensitive deletion deserves negative tests.

---

# 245. Mock Filesystem Errors

Test:

```text
PermissionError
FileNotFoundError
OSError
disk-full behavior
invalid configuration
```

The automation should fail clearly rather than silently corrupting state.

---

# 246. Unit vs Integration Tests

Unit tests:

```text
path validation
hash calculation
age calculation
configuration parsing
```

Integration tests:

```text
real filesystem
real permissions
service reload
artifact extraction
systemd interaction
```

Use both where appropriate.

---

# 247. CI Quality Gates

A Python filesystem automation repository can use:

```text
ruff
pytest
mypy
bandit
dependency scanner
```

depending on the team's standards.

A typical pipeline:

```text
lint
 ↓
unit tests
 ↓
security scan
 ↓
build
 ↓
integration tests
```

---

# 248. Dependency Management

Prefer:

```text
requirements.txt
pyproject.toml
lock/constraint strategy
```

and a virtual environment.

Do not install arbitrary packages globally on production hosts.

---

# 249. Python Version Pinning

Production automation should define a supported Python version.

Example:

```text
Python 3.12
```

Then validate:

```python
import sys

if sys.version_info < (
    3,
    11,
):
    raise RuntimeError(
        "Unsupported Python version"
    )
```

Use the organization's actual supported version.

---

# 250. Virtual Environment

Create:

```bash
python3 -m venv .venv
```

Use:

```bash
.venv/bin/python
```

and:

```bash
.venv/bin/pip
```

This avoids dependency conflicts with system Python.

---

# 251. Never Replace System Python Carelessly

On many Linux distributions, system components depend on the distribution-managed Python environment.

Do not:

```text
replace /usr/bin/python
remove system Python
globally upgrade packages
```

without understanding the OS package model.

---

# 252. Python Automation Installation

A production application can use:

```text
/opt/myapp/
├── .venv/
├── src/
├── config/
└── scripts/
```

Run:

```bash
/opt/myapp/.venv/bin/python \
    /opt/myapp/src/main.py
```

This makes the runtime explicit.

---

# 253. Environment Files

If using:

```text
.env
```

ensure it is not accidentally packaged or committed when it contains secrets.

Typical controls:

```text
.gitignore
permissions
secret manager
deployment-time injection
```

---

# 254. `.gitignore`

Typical Python repository:

```gitignore
.venv/
__pycache__/
*.pyc
.env
.pytest_cache/
```

Add project-specific build artifacts and secrets according to policy.

---

# 255. File Automation Repository

Example:

```text
filesystem-automation/
├── pyproject.toml
├── README.md
├── src/
│   └── filesystem_automation/
│       ├── paths.py
│       ├── cleanup.py
│       ├── artifacts.py
│       ├── config.py
│       └── health.py
├── scripts/
│   ├── disk_report.py
│   └── cleanup.py
└── tests/
    ├── test_paths.py
    ├── test_cleanup.py
    └── test_artifacts.py
```

Separate reusable logic from command-line entry points.

---

# 256. Logging Strategy

Use:

```python
import logging

logger = logging.getLogger(
    __name__
)
```

Log:

```text
operation
target
result
duration
error
```

Avoid logging:

```text
file contents
private keys
passwords
tokens
secret environment variables
```

---

# 257. Structured Logging

For automation platforms, structured output can be useful:

```json
{
  "operation": "cleanup",
  "target": "/var/tmp/myapp",
  "files_removed": 12,
  "dry_run": true,
  "status": "success"
}
```

This can be consumed by:

```text
CI/CD
ELK
monitoring
incident tooling
```

---

# 258. Exit Codes

For CLI automation:

```text
0 -> success
1 -> general failure
2 -> usage/validation failure
```

Exact codes should be documented.

For monitoring scripts, a non-zero exit code should clearly indicate failure.

---

# 259. File-System Health Report

A useful report can include:

```json
{
  "filesystem": "/",
  "disk_percent": 82.1,
  "free_bytes": 19384756224,
  "inode_percent": 41.3,
  "largest_files": [
    "/var/log/app.log"
  ],
  "status": "healthy"
}
```

This can feed dashboards or incident automation.

---

# 260. Production Automation Safety Model

Before executing a filesystem operation:

```text
Is the path correct?
Is the operation necessary?
Is the target expected?
Can it be retried?
Is it idempotent?
Can it be rolled back?
Could it expose secrets?
Could it affect active services?
Could it cross a filesystem boundary?
Could it follow a symlink?
Could it delete more than intended?
```

These questions prevent many production incidents.

---

# 261. Common Mistakes

Avoid:

```text
rm -rf equivalent without validation
chmod 777
hardcoded /tmp filenames
blind archive extraction
blind recursive chmod/chown
rewriting unchanged configuration
loading huge files into memory
scanning /
ignoring inode usage
ignoring deleted-open files
writing secrets to logs
running as root unnecessarily
overwriting production files directly
no backup
no verification
no dry-run
```

---

# 262. Interview Question — Why Use pathlib?

**Answer:**

> I prefer `pathlib` because it provides a clear object-oriented interface for filesystem paths and avoids a lot of manual string manipulation. It also integrates naturally with file existence checks, directory traversal, reading, writing, and path composition.

---

# 263. Interview Question — How Do You Delete Files Safely?

**Answer:**

> I first validate the target path and ensure it is within an approved directory. I check the file type and current state, support dry-run where appropriate, log the intended change, and only then delete it. For recursive deletion I use especially strict validation and testing.

---

# 264. Interview Question — How Do You Handle Large Log Files?

**Answer:**

> I avoid loading the entire file into memory. I stream it line-by-line or in chunks, identify the relevant patterns, and rely on logrotate or the centralized logging platform for retention rather than building uncontrolled custom deletion logic.

---

# 265. Interview Question — What Is Idempotent Filesystem Automation?

**Answer:**

> It means repeated execution converges to the same desired state. For example, creating a directory with `parents=True` and `exist_ok=True`, or updating a configuration only when its content differs from the desired configuration.

---

# 266. Interview Question — Why Use Atomic File Replacement?

**Answer:**

> It prevents consumers from observing a partially written configuration or generated file. I write and validate a complete temporary file and then replace the target atomically when the filesystem semantics allow it.

---

# 267. Interview Question — How Do You Find Large Files?

**Answer:**

> For targeted Python automation I can use `pathlib` and `stat().st_size`, while for very large filesystem scans I may use optimized OS-level tools. I avoid scanning the entire root filesystem unnecessarily and exclude special or mounted filesystems where appropriate.

---

# 268. Interview Question — Disk Is Full but Visible Files Do Not Explain It

**Answer:**

> I compare `df -h` and `df -i`, then check deleted-open files with `lsof +L1`. I also consider mount points, filesystem metadata, sparse files, and hard links. A deleted file can continue consuming space while a process still holds its file descriptor.

---

# 269. Interview Question — Why Is chmod 777 Bad?

**Answer:**

> It grants broad permissions to everyone and can create security vulnerabilities. I determine the required user and group and grant only the minimum permissions required by the application.

---

# 270. Interview Question — How Do You Safely Deploy Configuration?

**Answer:**

> I generate the desired configuration, validate its syntax, create a backup if appropriate, write it to a temporary file, atomically replace the active configuration, reload the service, and perform a health check. If validation fails, I don't activate the new configuration.

---

# 271. Interview Question — How Do You Handle Symlinks in Automation?

**Answer:**

> I explicitly distinguish symlinks from regular files, inspect their targets, and avoid accidentally following links outside the intended directory. For release deployments, I can use a versioned release directory and an atomic `current` symlink switch.

---

# 272. Interview Question — How Do You Prevent Path Traversal?

**Answer:**

> I validate input against an allowed root, resolve paths, reject targets outside that root, avoid unsafe recursive operations, and use OS-level safe primitives when the threat model requires stronger protection.

---

# 273. Interview Question — How Do You Handle Archive Extraction?

**Answer:**

> I verify the artifact first, validate archive members against the extraction directory, reject unsafe paths or links, run with least privilege, and extract only trusted content. I never blindly extract untrusted archives.

---

# 274. Interview Question — How Do You Manage Release Rollbacks?

**Answer:**

> I keep immutable versioned releases and use a stable pointer such as a `current` symlink. A rollback switches the pointer back to the last known-good release, then reloads the service and verifies health.

---

# 275. Interview Question — Why Not Modify Production Files Directly?

**Answer:**

> Direct modification can expose applications to partial writes and makes rollback harder. For important configuration I prefer a validated temporary file followed by atomic replacement and service health verification.

---

# 276. Interview Scenario — `/var/log` Is 98% Full

**Answer:**

```text
df -h /var/log
       ↓
df -i /var/log
       ↓
largest directories/files
       ↓
log rotation status
       ↓
deleted-open files
       ↓
container/application logs
       ↓
approved retention cleanup
       ↓
verify free space
```

I would not simply delete the largest log file.

---

# 277. Interview Scenario — Nginx Config Was Changed by Python

**Answer:**

I would verify:

```bash
nginx -t
```

before reload.

Then:

```bash
systemctl reload nginx
```

and verify:

```text
service state
HTTP health
logs
metrics
```

If validation fails, I restore the known-good configuration rather than activating the bad version.

---

# 278. Interview Scenario — Deployment Has Wrong File Ownership

Check:

```bash
namei -l /opt/myapp/current/config/app.yaml
```

Then inspect:

```text
owner
group
directory permissions
file permissions
service user
```

Correct only the required ownership instead of recursively changing the entire filesystem.

---

# 279. Interview Scenario — Cleanup Script Deleted the Wrong File

Immediate priorities:

```text
stop further execution
preserve logs
identify scope
check backups
determine affected services/data
restore if possible
```

Then:

```text
root cause analysis
path validation
dry-run
permissions
tests
approval
```

Destructive automation should have safeguards before production execution.

---

# 280. Interview Scenario — Artifact Extracts Successfully but Application Fails

Possible causes:

```text
missing file
wrong permissions
wrong owner
incorrect symlink
configuration
dependency
wrong executable bit
```

Run a deployment validation before activating the release.

---

# 281. Interview Scenario — File Exists but Application Cannot Read It

Check:

```text
file permissions
parent directory permissions
service user
group
ACL
SELinux/AppArmor
mount
```

Use:

```bash
namei -l /path/to/file
```

to inspect permissions along the entire path.

---

# 282. Interview Scenario — Disk Usage Does Not Match `du`

Possible explanations:

```text
deleted-open files
mount points
hard links
sparse files
filesystem metadata
reserved blocks
```

Compare:

```bash
df -h
du -xsh /var/*
lsof +L1
```

where appropriate.

---

# 283. Interview Scenario — Cleanup Job Works Manually but Not in Jenkins

Check:

```text
workspace
user
PATH
Python interpreter
permissions
environment variables
working directory
container image
```

Make the job explicit:

```bash
/opt/tool/.venv/bin/python \
    /opt/tool/scripts/cleanup.py \
    --dry-run
```

---

# 284. Interview Scenario — Python Script Is Slow

Investigate:

```text
number of files
directory depth
stat calls
network filesystem
repeated scans
subprocess count
logging volume
```

Optimize:

```text
scandir
targeted paths
streaming
pruning
batch operations
```

before adding concurrency.

---

# 285. Interview Scenario — Millions of Files in One Directory

First determine:

```text
why
which application
retention
inode pressure
filesystem performance
```

Then design:

```text
partitioning
sharding
cleanup
archive
application fix
```

Do not delete files blindly.

---

# 286. Interview Scenario — Filesystem Became Read-Only

Answer:

> I would stop repeatedly writing to the filesystem and investigate the mount state, kernel logs, filesystem errors, storage health, and recent infrastructure events. I would follow the platform's recovery procedure rather than forcing writes.

---

# 287. Interview Scenario — Secret Found in Artifact

Actions:

```text
stop distribution
identify exposure
rotate secret
remove from future builds
fix artifact process
audit access
```

Do not assume deleting the artifact from one location removes all copies.

---

# 288. Interview Scenario — Release Symlink Is Broken

Check:

```python
current.is_symlink()
current.readlink()
current.resolve()
```

Then identify the last known-good release and follow the rollback procedure.

---

# 289. Interview Scenario — Temporary Directory Is Full

Check:

```text
disk
inodes
largest files
file count
process ownership
cleanup history
```

Then fix:

```text
retention
application leak
job cleanup
filesystem capacity
```

---

# 290. Interview Scenario — Configuration Is Modified Repeatedly

Possible cause:

```text
automation not idempotent
```

Compare:

```text
desired content
actual content
```

Only write when necessary.

Also check:

```text
file watchers
deployment agents
configuration management
multiple automation jobs
```

---

# 291. Interview Scenario — Two Deployments Run at the Same Time

Possible risks:

```text
same release directory
configuration race
current symlink race
cleanup race
service restart race
```

Use:

```text
CI/CD concurrency control
distributed lock where required
unique release IDs
atomic activation
```

---

# 292. Interview Scenario — Python Deletes Symlink Target

The script intended to remove:

```text
current
```

but followed the symlink and deleted:

```text
release
```

Lesson:

```text
distinguish link from target
```

Use:

```python
path.is_symlink()
```

and carefully choose operations that act on the link itself.

---

# 293. Interview Scenario — File Permission Changes Do Not Fix Access

Investigate:

```text
SELinux
ACL
mount options
parent directory
service user
container user
```

Unix mode bits are only one part of access control.

---

# 294. Interview Scenario — Application Cannot Write to Mounted Volume

Check:

```text
mount path
UID/GID
filesystem permissions
read-only mount
security context
storage backend
```

Do not simply run the application as root.

---

# 295. Interview Scenario — Build Artifact Contains `.env`

Actions:

```text
stop publication
remove secret from artifact process
rotate exposed credentials
check Git history if committed
audit distribution
add CI secret scanning
```

The correct remediation includes the secret itself, not only the file.

---

# 296. Interview Scenario — Backup Is Successful but Restore Fails

Lesson:

```text
backup success != recoverability
```

Test:

```text
archive integrity
checksum
restore
permissions
ownership
application compatibility
```

Perform restore drills for critical systems.

---

# 297. Interview Scenario — Python Automation Runs as Root

Ask:

```text
Why?
```

Then reduce privileges:

```text
dedicated user
group
specific sudo rule
capability
approved privileged helper
```

Root should be the exception, not the default.

---

# 298. Interview Scenario — Cleanup Script Has `shell=True`

Review whether shell execution is actually needed.

Prefer:

```python
subprocess.run(
    ["find", ...],
    check=True,
)
```

or better, use Python filesystem APIs when they provide the required behavior.

Validate all external input.

---

# 299. Interview Scenario — File Is Deleted but Space Is Not Reclaimed

Answer:

> I would check for deleted-open files using `lsof +L1`. If a process still has the file descriptor open, the filesystem blocks remain allocated until that descriptor is closed. I would then follow the service-specific procedure to close the descriptor safely.

---

# 300. Interview Scenario — `df` and Python Directory Scan Disagree

Answer:

> I would check for deleted-open files, mount points, hard links, sparse files, filesystem metadata, and reserved blocks. `df` measures filesystem allocation while a directory scan measures visible files, so they are not guaranteed to match.

---

# 301. Interview Scenario — Why Use Atomic Writes in CI/CD?

Answer:

> Atomic writes reduce the chance that a deployment or configuration consumer sees a partially written file. I generate and validate the complete file first, then replace the target atomically when the filesystem supports it.

---

# 302. Interview Scenario — Why Does a File Change Trigger a Service Restart?

Answer:

Possible causes include:

```text
configuration watcher
systemd path unit
application watcher
deployment controller
operator
```

I identify which component watches the file before changing automation behavior.

---

# 303. Interview Scenario — How Would You Automate Cleanup Safely?

Answer:

> I would define an approved root and retention policy, identify candidates by file type and age, produce a dry-run report, test the deletion logic, protect active files, execute with least privilege, and verify disk/inode recovery afterward.

---

# 304. Interview Scenario — How Would You Build a Production File Audit Tool?

Collect:

```text
path
type
size
owner
group
permissions
mtime
inode
hash when appropriate
```

Output:

```text
JSON/CSV
```

Then integrate with:

```text
CI/CD
security scanning
ELK
compliance reporting
```

---

# 305. Interview Scenario — How Would You Automate Release Management?

Architecture:

```text
build
  ↓
versioned artifact
  ↓
checksum
  ↓
release directory
  ↓
extract
  ↓
validate
  ↓
activate symlink
  ↓
restart/reload
  ↓
health check
  ↓
retain previous release
```

This creates a predictable rollback path.

---

# 306. Interview Scenario — How Would You Prevent Disk Exhaustion?

Monitor:

```text
disk %
inode %
growth rate
largest directories
log growth
temporary files
container storage
```

Then establish:

```text
retention
rotation
cleanup
capacity alerts
runbooks
```

Do not rely on emergency deletion.

---

# 307. Interview Scenario — How Would You Automate Log Cleanup?

Answer:

> First I determine whether logrotate, journald, container logging, or centralized logging already handles retention. If custom cleanup is required, I define an approved directory, retention policy, dry-run mode, file-type checks, age checks, and verification.

---

# 308. Interview Scenario — How Would You Validate a Deployment Artifact?

I would check:

```text
checksum/signature
version
manifest
required files
file types
permissions
configuration syntax
executable bits
```

Only after validation would I activate it.

---

# 309. Interview Scenario — How Would You Make Filesystem Automation Production Safe?

Answer:

```text
explicit paths
input validation
least privilege
idempotency
atomic writes
backups
dry-run
timeouts
logging
error handling
testing
rollback
post-action verification
```

---

# 310. Daily DevOps Filesystem Workflow

A typical day may involve:

```text
check disk
   ↓
inspect logs
   ↓
validate configuration
   ↓
prepare artifact
   ↓
deploy release
   ↓
switch symlink
   ↓
restart/reload
   ↓
health check
   ↓
cleanup old releases
```

Python can automate the custom logic while Linux and platform tools remain the foundation.

---

# 311. Production Filesystem Mental Model

Think in layers:

```text
Path
 ↓
File type
 ↓
Permissions
 ↓
Ownership
 ↓
Filesystem
 ↓
Mount
 ↓
Process
 ↓
Application
 ↓
Service
 ↓
Deployment
```

A filesystem problem is often connected to a higher or lower layer.

---

# 312. Final Best Practices

```text
1. Prefer pathlib for normal path manipulation.
2. Stream large files.
3. Avoid unnecessary filesystem scans.
4. Validate paths before destructive operations.
5. Prefer atomic replacement for critical files.
6. Do not rewrite unchanged configuration.
7. Use secure temporary-file APIs.
8. Verify ownership and permissions.
9. Treat symlinks carefully.
10. Validate archives before extraction.
11. Verify artifact integrity.
12. Keep versioned releases.
13. Protect active releases.
14. Use approved retention policies.
15. Do not replace database-native backups.
16. Respect container and volume boundaries.
17. Handle filesystem exceptions explicitly.
18. Log changes without secrets.
19. Support dry-run.
20. Test destructive code in isolated environments.
```

---

# 313. Final Production Checklist

```text
[ ] Path is explicit
[ ] Path is validated
[ ] Allowed root is defined
[ ] Symlink behavior is understood
[ ] File type is checked
[ ] Ownership is correct
[ ] Permissions are correct
[ ] Secrets are protected
[ ] Existing data is protected
[ ] Backup exists when required
[ ] Artifact checksum verified
[ ] Configuration validated
[ ] Atomic replacement used where appropriate
[ ] Dry-run available
[ ] Idempotency tested
[ ] Errors handled
[ ] Logs generated
[ ] Exit codes correct
[ ] Disk checked
[ ] Inodes checked
[ ] Rollback possible
[ ] Post-change health check completed
```

---

# 314. Final Mental Model

Filesystem automation with Python should follow:

```text
INPUT
  ↓
VALIDATE PATH
  ↓
CHECK CURRENT STATE
  ↓
CALCULATE DESIRED STATE
  ↓
PLAN
  ↓
BACKUP IF REQUIRED
  ↓
APPLY SAFELY
  ↓
ATOMICALLY REPLACE WHEN APPROPRIATE
  ↓
VERIFY
  ↓
LOG
  ↓
RETURN CORRECT EXIT CODE
```

For destructive operations:

```text
DRY-RUN
   ↓
REVIEW
   ↓
EXECUTE
   ↓
VERIFY
```

For production deployments:

```text
ARTIFACT
   ↓
INTEGRITY CHECK
   ↓
EXTRACT SAFELY
   ↓
VALIDATE
   ↓
VERSIONED RELEASE
   ↓
ATOMIC ACTIVATION
   ↓
SERVICE RELOAD
   ↓
HEALTH CHECK
   ↓
ROLLBACK IF REQUIRED
```

> **The goal of filesystem automation is not simply to create, copy, move, or delete files. The goal is to make Linux filesystem changes predictable, repeatable, secure, observable, and recoverable.**
