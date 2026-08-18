# 03-File-and-Directory-Automation

## Python Automation — Files and Directories

This module focuses on **general-purpose Python file and directory automation for DevOps**.

The previous Linux section covered filesystem administration. Here the focus is on building reusable Python automation that works with:

```text
application files
configuration files
deployment artifacts
directories
backups
reports
temporary workspaces
CI/CD artifacts
logs
release packages
JSON/YAML/CSV files
```

The DevOps mindset is:

```text
discover
    ↓
validate
    ↓
process
    ↓
verify
    ↓
report
```

---

# 1. Why File and Directory Automation Matters

DevOps engineers repeatedly perform tasks such as:

```text
create application directories
copy deployment files
generate configuration
move artifacts
archive releases
clean temporary files
process logs
find files
compare directories
calculate checksums
prepare CI/CD artifacts
```

Python makes these operations:

```text
repeatable
consistent
testable
portable
automatable
```

---

# 2. Core Python Modules

Important modules:

```python
pathlib
os
shutil
glob
fnmatch
tempfile
filecmp
hashlib
tarfile
zipfile
gzip
```

Supporting modules:

```python
json
yaml
csv
logging
argparse
datetime
subprocess
```

---

# 3. Prefer `pathlib`

Modern Python code should generally prefer:

```python
from pathlib import Path
```

instead of constructing paths manually with strings.

---

# 4. Create a Path

```python
from pathlib import Path

config = Path(
    "/opt/myapp/config.yaml"
)

print(config)
```

---

# 5. Join Paths

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

# 6. Current Working Directory

```python
cwd = Path.cwd()

print(cwd)
```

---

# 7. Home Directory

```python
home = Path.home()

print(home)
```

---

# 8. Resolve a Path

```python
path = Path(
    "./config/app.yaml"
)

print(path.resolve())
```

Use resolved paths carefully when symlink behavior matters.

---

# 9. Absolute vs Relative Paths

Absolute:

```text
/opt/myapp/config.yaml
```

Relative:

```text
config/config.yaml
```

For production automation, explicitly control the working directory.

---

# 10. Check Existence

```python
if path.exists():
    print("Exists")
```

---

# 11. Check File

```python
if path.is_file():
    print("Regular file")
```

---

# 12. Check Directory

```python
if path.is_dir():
    print("Directory")
```

---

# 13. Check Symlink

```python
if path.is_symlink():
    print("Symbolic link")
```

---

# 14. Create Directory

```python
path = Path(
    "/opt/myapp/logs"
)

path.mkdir()
```

---

# 15. Create Nested Directories

```python
path.mkdir(
    parents=True,
    exist_ok=True,
)
```

This is one of the most common DevOps filesystem patterns.

---

# 16. Idempotent Directory Creation

First run:

```text
directory missing
    ↓
create
```

Second run:

```text
directory exists
    ↓
continue
```

Use:

```python
mkdir(
    parents=True,
    exist_ok=True,
)
```

---

# 17. Create File

```python
status_file = Path(
    "/opt/myapp/status"
)

status_file.touch(
    exist_ok=True
)
```

---

# 18. Write Text

```python
config.write_text(
    "APP_ENV=production\n",
    encoding="utf-8",
)
```

---

# 19. Read Text

```python
content = config.read_text(
    encoding="utf-8"
)

print(content)
```

---

# 20. Append Text

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

# 21. File Open Modes

Common modes:

```text
r   read
w   write/truncate
a   append
x   create exclusively
rb  binary read
wb  binary write
```

---

# 22. Exclusive Creation

```python
with path.open(
    "x",
    encoding="utf-8",
) as file:

    file.write(
        "created once\n"
    )
```

This fails if the file already exists.

---

# 23. Stream Large Files

Avoid:

```python
content = file.read()
```

for very large files.

Prefer:

```python
with file.open(
    encoding="utf-8",
    errors="replace",
) as handle:

    for line in handle:
        process(line)
```

---

# 24. Binary Files

Use binary mode for:

```text
images
archives
PDFs
compressed files
certificates
artifacts
```

Example:

```python
with path.open("rb") as file:
    data = file.read()
```

---

# 25. File Metadata

```python
info = path.stat()

print(info.st_size)
print(info.st_mtime)
print(info.st_mode)
```

---

# 26. File Size

```python
size = path.stat().st_size

print(size)
```

Convert to MB:

```python
size_mb = (
    size / 1024 / 1024
)
```

---

# 27. Modification Time

```python
from datetime import datetime

mtime = path.stat().st_mtime

print(
    datetime.fromtimestamp(mtime)
)
```

Use timezone-aware timestamps for production reports.

---

# 28. List Directory Contents

```python
root = Path("/opt/myapp")

for item in root.iterdir():
    print(item)
```

---

# 29. List Files Only

```python
for item in root.iterdir():
    if item.is_file():
        print(item)
```

---

# 30. List Directories Only

```python
for item in root.iterdir():
    if item.is_dir():
        print(item)
```

---

# 31. Glob

```python
for file in root.glob("*.log"):
    print(file)
```

---

# 32. Recursive Glob

```python
for file in root.rglob("*.log"):
    print(file)
```

---

# 33. Common Glob Patterns

```text
*.log
*.yaml
*.yml
*.json
*.conf
app-*.log
release-*.tar.gz
```

---

# 34. `fnmatch`

```python
from fnmatch import fnmatch

if fnmatch(
    file.name,
    "*.yaml",
):
    print(file)
```

---

# 35. Find Deployment Files

```python
for file in Path(
    "deployment"
).rglob("*.yaml"):

    print(file)
```

Useful in CI/CD validation.

---

# 36. Find Test Files

```python
for file in Path(
    "."
).rglob("test_*.py"):

    print(file)
```

---

# 37. Find Large Files

```python
threshold = (
    100 * 1024 * 1024
)

for file in Path(
    "."
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

---

# 38. Find Old Files

```python
import time

cutoff = (
    time.time()
    - 7 * 24 * 60 * 60
)

for file in Path(
    "artifacts"
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

# 39. Copy a File

```python
import shutil

shutil.copy(
    source,
    destination,
)
```

---

# 40. Copy with Metadata

```python
shutil.copy2(
    source,
    destination,
)
```

`copy2()` attempts to preserve additional metadata.

---

# 41. Copy a Directory

```python
shutil.copytree(
    source,
    destination,
)
```

---

# 42. Copy into Existing Directory

Modern Python:

```python
shutil.copytree(
    source,
    destination,
    dirs_exist_ok=True,
)
```

Use carefully because existing files can be overwritten.

---

# 43. Move a File

```python
shutil.move(
    source,
    destination,
)
```

---

# 44. Rename a File

```python
source.rename(
    destination
)
```

---

# 45. Delete a File

```python
path.unlink()
```

---

# 46. Delete If Present

```python
path.unlink(
    missing_ok=True
)
```

Useful for idempotent cleanup.

---

# 47. Delete Empty Directory

```python
path.rmdir()
```

The directory must be empty.

---

# 48. Delete Directory Recursively

```python
shutil.rmtree(
    directory
)
```

This is destructive.

Never call it on an unvalidated user-controlled path.

---

# 49. Safe Recursive Delete

Before:

```python
shutil.rmtree(target)
```

validate:

```text
target exists
target is expected type
target is within approved root
target is not a protected directory
target matches expected pattern
```

---

# 50. Path Traversal

Dangerous input:

```text
../../etc
```

or:

```text
../../home/user
```

Never assume a user-provided relative path is safe.

---

# 51. Restrict a Path

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
        "Invalid path"
    )
```

---

# 52. Symlink Safety

A path can point outside the intended directory through a symlink.

For sensitive or destructive operations:

```text
resolve
 ↓
validate
 ↓
operate
```

---

# 53. Temporary Directory

```python
from tempfile import TemporaryDirectory

with TemporaryDirectory() as tmp:
    workdir = Path(tmp)

    print(workdir)
```

Useful for:

```text
builds
archives
test data
deployment staging
temporary extraction
```

---

# 54. Temporary File

```python
import tempfile

with tempfile.NamedTemporaryFile(
    mode="w",
    encoding="utf-8",
) as file:

    file.write(
        "temporary content"
    )
```

---

# 55. Why Temporary Files?

They allow you to:

```text
stage changes
validate content
avoid partial target writes
```

---

# 56. Atomic File Replacement

Production pattern:

```text
create temporary file
        ↓
write complete content
        ↓
validate
        ↓
os.replace()
```

---

# 57. Atomic Replacement Example

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

For production, establish required permissions and ownership explicitly.

---

# 58. Why Atomic Replacement?

Without atomic replacement:

```text
application reads file
        ↓
file is partially rewritten
        ↓
invalid configuration
```

With atomic replacement:

```text
old file
   ↓
new file prepared
   ↓
validated
   ↓
replaced
```

---

# 59. Configuration Deployment

A safe workflow:

```text
generate
 ↓
validate
 ↓
backup
 ↓
atomic replace
 ↓
reload
 ↓
health check
```

---

# 60. JSON Files

```python
import json

data = json.loads(
    path.read_text(
        encoding="utf-8"
    )
)
```

---

# 61. Write JSON

```python
path.write_text(
    json.dumps(
        data,
        indent=2,
    ) + "\n",
    encoding="utf-8",
)
```

---

# 62. Update JSON Configuration

```python
data["replicas"] = 3

path.write_text(
    json.dumps(
        data,
        indent=2,
    ) + "\n",
    encoding="utf-8",
)
```

For critical production files, use an atomic write.

---

# 63. YAML Files

```python
import yaml

data = yaml.safe_load(
    path.read_text(
        encoding="utf-8"
    )
)
```

Use:

```python
yaml.safe_load()
```

for untrusted YAML rather than unsafe object construction.

---

# 64. YAML Update

```python
data["image"] = (
    "myapp:2.0"
)

path.write_text(
    yaml.safe_dump(
        data,
        sort_keys=False,
    ),
    encoding="utf-8",
)
```

Be aware that serialization can change formatting/comments.

---

# 65. CSV Files

```python
import csv

with open(
    "servers.csv",
    newline="",
    encoding="utf-8",
) as file:

    reader = csv.DictReader(file)

    for row in reader:
        print(row)
```

---

# 66. CSV Generation

```python
import csv

rows = [
    {
        "host": "server01",
        "status": "healthy",
    },
    {
        "host": "server02",
        "status": "warning",
    },
]

with open(
    "report.csv",
    "w",
    newline="",
    encoding="utf-8",
) as file:

    writer = csv.DictWriter(
        file,
        fieldnames=[
            "host",
            "status",
        ],
    )

    writer.writeheader()
    writer.writerows(rows)
```

---

# 67. Configuration Backup

```python
from datetime import datetime, timezone
import shutil

timestamp = datetime.now(
    timezone.utc
).strftime(
    "%Y%m%dT%H%M%SZ"
)

backup = Path(
    f"/backup/config-{timestamp}.yaml"
)

shutil.copy2(
    source,
    backup,
)
```

---

# 68. File Checksums

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

# 69. Why Use SHA-256?

Useful for:

```text
artifact verification
backup validation
deployment verification
file comparison
download verification
```

---

# 70. Compare Files by Hash

```python
if sha256(source) == sha256(
    destination
):
    print("Identical content")
```

For very large files, consider whether a direct comparison or metadata-first strategy is more efficient.

---

# 71. Directory Comparison

Python provides:

```python
import filecmp

result = filecmp.dircmp(
    left,
    right,
)
```

Useful for deployment verification.

---

# 72. Compare Directory Contents

```python
print(
    result.left_only
)

print(
    result.right_only
)

print(
    result.diff_files
)
```

---

# 73. File Synchronization

For large-scale synchronization, don't automatically reinvent:

```text
rsync
```

Python can orchestrate it when appropriate.

---

# 74. Rsync from Python

```python
import subprocess

subprocess.run(
    [
        "rsync",
        "-a",
        source,
        destination,
    ],
    check=True,
)
```

Validate source/destination before executing.

---

# 75. File Compression

Python supports:

```text
gzip
tarfile
zipfile
```

---

# 76. Gzip a File

```python
import gzip
import shutil

with open(
    source,
    "rb",
) as src:

    with gzip.open(
        destination,
        "wb",
    ) as dst:

        shutil.copyfileobj(
            src,
            dst,
        )
```

---

# 77. TAR Archive

```python
import tarfile

with tarfile.open(
    "release.tar.gz",
    "w:gz",
) as archive:

    archive.add(
        "release",
        arcname="release",
    )
```

---

# 78. ZIP Archive

```python
import zipfile

with zipfile.ZipFile(
    "release.zip",
    "w",
) as archive:

    archive.write(
        "config.yaml"
    )
```

---

# 79. Archive Extraction Security

Do not blindly extract untrusted archives.

Potential attacks include:

```text
path traversal
symlink escape
unexpected absolute paths
```

Validate every archive member.

---

# 80. Backup Directory Structure

Example:

```text
backup/
├── daily/
├── weekly/
├── monthly/
└── restore-tests/
```

---

# 81. Artifact Directory Structure

```text
artifacts/
├── incoming/
├── validated/
├── released/
├── failed/
└── archive/
```

---

# 82. File State Machine

Example:

```text
incoming
   ↓
validated
   ↓
approved
   ↓
released
   ↓
archived
```

The filesystem structure can represent workflow state.

---

# 83. Atomic File Movement

A producer can write:

```text
artifact.tmp
```

and after completion:

```text
artifact.ready
```

using an atomic rename.

Consumers process only ready files.

---

# 84. File Queue Pattern

```text
producer
 ↓
write temp
 ↓
atomic rename
 ↓
consumer detects ready file
 ↓
process
 ↓
archive
```

This avoids consuming partially written files.

---

# 85. Completion Marker

Another pattern:

```text
large-file.data
large-file.data.done
```

The `.done` marker means the producer has completed the data file.

---

# 86. File Locking

Multiple processes may access:

```text
same configuration
same state file
same backup
same release
```

Use appropriate locking when concurrent updates are possible.

---

# 87. Local vs Distributed Lock

Local file lock:

```text
single host
```

Distributed lock:

```text
multiple hosts
```

For distributed systems consider:

```text
database
Redis
DynamoDB
coordination service
```

---

# 88. File Race Conditions

Avoid relying on:

```python
if not path.exists():
    create()
```

when exclusivity is required.

Prefer atomic/exclusive operations such as:

```python
path.open("x")
```

---

# 89. File Descriptor Management

Always use:

```python
with path.open(...) as file:
    ...
```

This ensures resources are closed.

---

# 90. Too Many Open Files

Processing thousands of files simultaneously can exhaust:

```text
file descriptors
```

Use:

```text
context managers
streaming
bounded concurrency
```

---

# 91. Large Directory Performance

For very large directories, consider:

```python
import os

with os.scandir(
    "/var/log"
) as entries:

    for entry in entries:
        print(entry.name)
```

`os.scandir()` can be efficient for metadata-heavy scans.

---

# 92. Lazy Traversal

Prefer:

```python
for path in root.rglob("*"):
    ...
```

instead of:

```python
files = list(
    root.rglob("*")
)
```

when the directory tree can be very large.

---

# 93. File Processing in Batches

For thousands of files:

```text
discover
 ↓
batch 1
 ↓
verify
 ↓
batch 2
 ↓
verify
```

This limits resource usage.

---

# 94. Parallel File Processing

Parallelism can help with:

```text
independent files
network storage
many artifacts
```

But excessive concurrency can overload:

```text
disk
NFS
network
object storage
file descriptors
```

---

# 95. File Permissions

```python
path.chmod(0o640)
```

Use the minimum permission required.

---

# 96. Ownership

```python
import os

os.chown(
    path,
    uid,
    gid,
)
```

This is generally Unix-specific and may require elevated privileges.

---

# 97. Don't Hardcode IDs

Instead of:

```python
uid = 1000
```

resolve the account:

```python
import pwd

user = pwd.getpwnam(
    "appuser"
)
```

---

# 98. File Security Audit

Automate checks for:

```text
world-writable files
unexpected ownership
private-key permissions
unexpected file types
unexpected files
```

---

# 99. World-Writable Detection

```python
import stat

mode = path.stat().st_mode

if mode & stat.S_IWOTH:
    print(
        "World writable:",
        path,
    )
```

---

# 100. Private-Key Audit

Example policy:

```text
private key
    ↓
exists
    ↓
expected owner
    ↓
restricted mode
    ↓
not world-readable
```

---

# 101. Secret Files

Examples:

```text
.env
credentials
private.pem
token files
cloud credentials
```

Protect them with:

```text
least privilege
secret management
restricted storage
```

---

# 102. Never Commit Secrets

Python automation can check staged/generated files for:

```text
private keys
tokens
password patterns
cloud credentials
```

Use dedicated secret scanners for stronger protection.

---

# 103. File Upload Automation

If handling uploaded files:

```text
validate size
validate type
sanitize name
generate storage name
store outside executable paths
```

Never trust a client-provided filename.

---

# 104. Safe File Names

Prefer:

```text
UUID
database ID
content hash
controlled filename
```

rather than using untrusted names directly.

---

# 105. File Extension Is Not Security

A file called:

```text
image.jpg
```

may contain something else.

For security-sensitive uploads, validate content using appropriate parsers/signatures.

---

# 106. Maximum File Size

Always define limits for automated inputs.

Example:

```text
config <= 5 MB
upload <= 100 MB
artifact <= 1 GB
```

Use requirements appropriate to the system.

---

# 107. Temporary Workspace

A deployment script may use:

```text
/tmp/deploy-XXXX
```

for staging.

Prefer:

```python
TemporaryDirectory()
```

over manually constructing predictable names.

---

# 108. Cleanup Temporary Workspace

Context manager:

```python
with TemporaryDirectory() as tmp:
    ...
```

automatically removes the workspace after completion.

---

# 109. Sensitive Temporary Data

For:

```text
credentials
tokens
private keys
```

use approved secure secret handling.

Do not assume deleting a file guarantees physical erasure.

---

# 110. Configuration Generation

Python can generate:

```text
.env
JSON
YAML
INI
TOML
systemd units
Nginx configuration
Kubernetes manifests
```

---

# 111. Configuration Validation

Before deployment:

```text
syntax
 ↓
schema
 ↓
required fields
 ↓
environment values
 ↓
security checks
```

---

# 112. Configuration Drift

Compare:

```text
expected
vs
actual
```

using:

```text
content comparison
checksum
structured comparison
```

---

# 113. Drift Detection

Example:

```python
expected = sha256(
    expected_file
)

actual = sha256(
    deployed_file
)

if expected != actual:
    print(
        "Configuration drift"
    )
```

---

# 114. Checksum Limitation

A checksum tells you:

```text
content changed
```

It does not tell you:

```text
who changed it
why
```

Use:

```text
Git
audit logs
configuration management
```

for attribution.

---

# 115. Versioned Release Directories

Example:

```text
/opt/myapp/
├── releases/
│   ├── 1.0.0/
│   ├── 1.1.0/
│   └── 1.2.0/
└── current -> releases/1.2.0
```

---

# 116. Release Deployment

```text
download artifact
 ↓
verify checksum
 ↓
create release directory
 ↓
extract
 ↓
validate
 ↓
set permissions
 ↓
health check
 ↓
promote
```

---

# 117. Immutable Release

After deployment:

```text
releases/1.2.0
```

should not be modified.

For the next change create:

```text
releases/1.3.0
```

---

# 118. Atomic Release Promotion

Use a controlled `current` pointer:

```text
current -> 1.2.0
```

Then promote:

```text
current -> 1.3.0
```

only after validation.

---

# 119. Rollback

If the new release fails:

```text
current -> 1.2.0
```

Then verify application health.

---

# 120. Release Cleanup

Delete only releases that are:

```text
old
inactive
outside retention
not required for rollback
```

Never delete the active release.

---

# 121. File-System Automation in Docker

Python can prepare:

```text
Docker build context
generated files
configuration
validation reports
```

but should avoid placing secrets into images.

---

# 122. Docker Build Context

Before building:

```text
check .dockerignore
remove unnecessary artifacts
check secrets
validate required files
```

---

# 123. Common Build Context Problems

Avoid including:

```text
.env
.git
.ssh
private keys
large logs
build caches
cloud credentials
```

unless explicitly required and safe.

---

# 124. File Automation in Kubernetes

Python can help:

```text
validate manifests
generate test artifacts
prepare configuration
collect diagnostics
```

Kubernetes should remain responsible for:

```text
pods
deployments
volumes
rollouts
desired state
```

---

# 125. ConfigMap vs Secret

```text
ConfigMap → non-sensitive configuration
Secret    → sensitive values
```

Do not put passwords in ConfigMaps.

---

# 126. Kubernetes Mounted Files

Applications may receive:

```text
ConfigMap volumes
Secret volumes
PersistentVolume mounts
emptyDir
```

Python scripts should understand that container filesystems may be ephemeral.

---

# 127. Persistent Data

Do not rely on:

```text
container writable layer
```

for durable application data.

Use:

```text
PersistentVolume
S3/object storage
database
```

as appropriate.

---

# 128. File Automation in EKS

Useful DevOps automation:

```text
collect pod diagnostics
validate manifests
inspect mounted configuration
generate reports
prepare deployment artifacts
```

---

# 129. CI/CD File Automation

Example:

```text
Git push
 ↓
Python validation
 ↓
unit tests
 ↓
artifact creation
 ↓
checksum
 ↓
security scan
 ↓
publish
 ↓
deploy
```

---

# 130. Jenkins Integration

Example:

```bash
python scripts/validate_files.py
```

A failed validation should return a non-zero exit code.

---

# 131. GitHub Actions Integration

```yaml
- name: Validate deployment files
  run: |
    python scripts/validate_files.py
```

---

# 132. GitLab CI Integration

```yaml
validate:
  script:
    - python scripts/validate_files.py
```

---

# 133. Exit Codes

```text
0       success
nonzero failure
```

Example:

```python
import sys

if validation_failed:
    sys.exit(1)
```

---

# 134. File Automation Reports

Useful formats:

```text
human-readable
JSON
CSV
```

Example JSON:

```json
{
  "status": "success",
  "files_processed": 120,
  "files_failed": 0
}
```

---

# 135. Logging

Production scripts should use:

```python
import logging
```

rather than only:

```python
print()
```

---

# 136. Safe Logging

Never log:

```text
passwords
tokens
private keys
secret values
customer data
```

---

# 137. File Operation Metrics

Useful aggregate metrics:

```text
files_processed_total
files_failed_total
bytes_processed_total
backup_success_total
backup_failure_total
cleanup_success_total
```

Avoid using full filenames as metric labels because that can create high cardinality.

---

# 138. Scheduled File Automation

Common schedulers:

```text
cron
systemd timer
Jenkins
GitHub Actions
GitLab CI
Kubernetes CronJob
```

---

# 139. Kubernetes CronJob

Example architecture:

```text
CronJob
 ↓
Python container
 ↓
perform task
 ↓
exit code
 ↓
logs
```

Do not rely on the container filesystem for durable state.

---

# 140. Avoid Overlapping Scheduled Jobs

Use:

```text
lock
```

or scheduler-level concurrency controls.

A daily job should not accidentally run twice simultaneously.

---

# 141. Backup Workflow

```text
validate source
 ↓
create archive/copy
 ↓
checksum
 ↓
verify
 ↓
store
 ↓
retention
 ↓
report
```

---

# 142. Restore Workflow

```text
select backup
 ↓
verify integrity
 ↓
extract to staging
 ↓
validate
 ↓
restore
 ↓
verify application
```

---

# 143. Backup Verification

A backup is not proven useful simply because:

```text
backup.tar.gz
```

exists.

Verify:

```text
archive readable
expected files
integrity
permissions
restore capability
```

---

# 144. File Archive Integrity

After creating an archive:

```text
exists
 ↓
non-zero size
 ↓
can be opened
 ↓
expected content
 ↓
checksum if required
```

---

# 145. Artifact Verification

```text
artifact
 ↓
checksum
 ↓
trusted checksum
 ↓
match?
```

If no:

```text
STOP
```

---

# 146. File Processing Pipeline

A robust workflow:

```text
Input
 ↓
Validate
 ↓
Stage
 ↓
Process
 ↓
Verify
 ↓
Promote
 ↓
Archive
```

---

# 147. Example Artifact Pipeline

```text
incoming/
    ↓
validate
    ↓
validated/
    ↓
scan
    ↓
released/
    ↓
archive/
```

---

# 148. File Queue

Example:

```text
incoming/job.json
```

Consumer:

```text
read
 ↓
validate
 ↓
process
 ↓
move to processed/
```

---

# 149. Avoid Partial Queue Files

Producer:

```text
job.json.tmp
```

After completion:

```text
rename → job.json
```

Consumer only watches:

```text
*.json
```

---

# 150. File-Based Lock

A lock can prevent:

```text
two cleanup jobs
two deployment jobs
two backup jobs
```

from running together.

Use a robust locking mechanism rather than simply checking whether a `.lock` file exists.

---

# 151. Stale Lock Problem

If a process crashes:

```text
app.lock
```

may remain.

A production lock should provide reliable ownership/recovery semantics rather than deleting old lock files blindly.

---

# 152. File-System Health

Monitor:

```text
disk usage
inode usage
directory growth
artifact count
temporary file count
backup size
```

---

# 153. Disk Usage Script

```python
import shutil

usage = shutil.disk_usage("/")

percent = (
    usage.used
    / usage.total
) * 100

print(
    f"Disk: {percent:.1f}%"
)
```

---

# 154. Large Artifact Detection

```python
threshold = (
    1 * 1024 * 1024 * 1024
)

for file in Path(
    "artifacts"
).rglob("*"):

    try:
        if (
            file.is_file()
            and file.stat().st_size
            > threshold
        ):
            print(
                "Large:",
                file,
            )
    except OSError:
        continue
```

---

# 155. Cleanup Policy

Example:

```text
incoming artifacts → 2 days
failed artifacts   → 7 days
released artifacts → 30 days
```

Retention should be configurable.

---

# 156. Dry Run

A destructive script should support:

```bash
python cleanup.py \
    --days 14 \
    --dry-run
```

Output:

```text
WOULD DELETE:
artifact-001
artifact-002
artifact-003
```

---

# 157. Candidate Limits

Example:

```python
if len(candidates) > 1000:
    raise RuntimeError(
        "Unexpected candidate count"
    )
```

Also consider total size limits.

---

# 158. Blast Radius

Before destructive automation ask:

```text
How many files?
How much data?
Which directory?
Which environment?
Which users?
```

Bound the blast radius.

---

# 159. Environment Guard

For production automation:

```python
if environment != "production":
    ...
```

Do not rely on a simple environment string alone for critical safety; use explicit deployment controls.

---

# 160. Production Confirmation

For highly destructive operations, consider:

```text
dry run
approval
change ticket
maintenance window
```

according to organizational policy.

---

# 161. File Automation Testing

Test:

```text
file exists
file missing
directory exists
directory missing
permission denied
symlink
path traversal
large file
empty file
binary file
concurrent execution
```

---

# 162. Temporary Test Environment

```python
from tempfile import TemporaryDirectory

with TemporaryDirectory() as tmp:
    root = Path(tmp)

    (root / "config").mkdir()

    (root / "config/app.yaml").write_text(
        "version: 1\n",
        encoding="utf-8",
    )
```

---

# 163. Mock Destructive Operations

When unit testing:

```text
shutil.rmtree
os.remove
subprocess
os.chown
```

can be mocked where appropriate.

---

# 164. Integration Testing

Use:

```text
temporary directories
containers
test VMs
```

instead of production paths.

---

# 165. Security Test Cases

Test:

```text
../
../../
absolute path
symlink
broken symlink
empty input
special characters
spaces
unicode
huge filename
huge file
```

---

# 166. Production File Automation Checklist

```text
[ ] Explicit root
[ ] Input validation
[ ] Path traversal protection
[ ] Symlink handling
[ ] Idempotency
[ ] Dry run
[ ] Candidate limits
[ ] Size limits
[ ] Least privilege
[ ] Secure temporary files
[ ] Atomic updates
[ ] Verification
[ ] Logging
[ ] Error handling
[ ] Locking
[ ] Recovery
```

---

# 167. Daily DevOps Script — Directory Bootstrap

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

# 168. Daily DevOps Script — Find Large Files

```python
from pathlib import Path

threshold = 500 * 1024 * 1024

for file in Path(
    "/var/log"
).rglob("*"):

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

# 169. Daily DevOps Script — Old Artifact Report

```python
from pathlib import Path
import time

root = Path("artifacts")

cutoff = (
    time.time()
    - 14 * 86400
)

for file in root.rglob("*"):

    try:
        if (
            file.is_file()
            and file.stat().st_mtime
            < cutoff
        ):
            print(
                "Old artifact:",
                file,
            )
    except OSError:
        continue
```

---

# 170. Daily DevOps Script — Permission Audit

```python
from pathlib import Path
import stat

for file in Path(
    "/opt/myapp"
).rglob("*"):

    try:
        mode = file.stat().st_mode

        if mode & stat.S_IWOTH:
            print(
                "World writable:",
                file,
            )
    except OSError:
        continue
```

---

# 171. Daily DevOps Script — File Checksum

```python
import hashlib

def sha256(path):
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

# 172. Daily DevOps Script — Config Backup

```python
from pathlib import Path
from datetime import datetime, timezone
import shutil

source = Path(
    "/etc/myapp/config.yaml"
)

timestamp = datetime.now(
    timezone.utc
).strftime(
    "%Y%m%dT%H%M%SZ"
)

destination = Path(
    f"/backup/config-{timestamp}.yaml"
)

destination.parent.mkdir(
    parents=True,
    exist_ok=True,
)

shutil.copy2(
    source,
    destination,
)
```

Add checksum and verification in production.

---

# 173. Daily DevOps Script — Artifact Package

```python
import tarfile

with tarfile.open(
    "release.tar.gz",
    "w:gz",
) as archive:

    archive.add(
        "release",
        arcname="release",
    )
```

---

# 174. Daily DevOps Script — Directory Comparison

```python
import filecmp

comparison = filecmp.dircmp(
    "expected",
    "actual",
)

print(
    "Missing:",
    comparison.left_only,
)

print(
    "Extra:",
    comparison.right_only,
)

print(
    "Different:",
    comparison.diff_files,
)
```

---

# 175. Daily DevOps Script — File Inventory

```python
from pathlib import Path

for file in Path(
    "/opt/myapp"
).rglob("*"):

    try:
        if file.is_file():
            info = file.stat()

            print(
                file,
                info.st_size,
                info.st_mtime,
            )
    except OSError:
        continue
```

---

# 176. Production Scenario — Disk Usage Suddenly Increases

Investigate:

```text
filesystem
 ↓
directory sizes
 ↓
file sizes
 ↓
logs
 ↓
artifacts
 ↓
temporary files
 ↓
deleted-open files
```

Then apply an approved remediation.

---

# 177. Production Scenario — Configuration File Partially Written

Fix the deployment process:

```text
write temp
 ↓
validate
 ↓
atomic replace
```

Do not modify the active file directly.

---

# 178. Production Scenario — Cleanup Deletes Wrong Files

Immediate response:

```text
stop automation
 ↓
preserve evidence
 ↓
identify deleted files
 ↓
restore
 ↓
fix selection logic
 ↓
add path validation
 ↓
add dry-run
 ↓
add safety limits
 ↓
test
```

---

# 179. Production Scenario — Two Jobs Modify Same Directory

Use:

```text
locking
```

or redesign the workflow so jobs work on independent staging directories.

---

# 180. Production Scenario — Artifact Corrupted During Copy

Use:

```text
copy
 ↓
checksum
 ↓
compare
 ↓
promote only if valid
```

---

# 181. Production Scenario — Release Fails After Deployment

Use versioned releases:

```text
release-41
release-42
```

If 42 fails:

```text
current -> release-41
```

Then verify application health.

---

# 182. Production Scenario — Secrets in Build Directory

Remove them from:

```text
build context
artifact
Git repository
logs
```

If credentials were exposed, rotate them according to incident policy.

---

# 183. Production Scenario — Archive Extraction

Never blindly:

```python
archive.extractall(
    destination
)
```

for untrusted input.

Validate each archive member path before extraction.

---

# 184. Production Scenario — Millions of Files

Use:

```text
scoped traversal
lazy iteration
os.scandir
batch processing
bounded concurrency
```

If the workload remains too large, reconsider the storage design.

---

# 185. Production Scenario — File Permissions Wrong

Check:

```text
owner
group
mode
umask
ACL
filesystem
security context
deployment method
```

Do not use `chmod 777` as a workaround.

---

# 186. Production Scenario — File Exists but Application Cannot Read It

Check:

```text
process user
file mode
file owner
parent directory
ACL
SELinux/AppArmor
container security context
mount
```

---

# 187. Production Scenario — Backup Exists but Cannot Restore

Treat the backup as unreliable until proven otherwise.

Add:

```text
restore tests
archive verification
checksum
documentation
```

---

# 188. Production Scenario — Temporary Files Fill Disk

Identify:

```text
producer
directory
file count
file age
file size
```

Then implement retention and fix the producer if necessary.

---

# 189. Production Scenario — Release Directory Cleanup

Only delete:

```text
inactive
old
non-required
```

releases.

Never delete the active release.

---

# 190. Production Scenario — Need Zero-Downtime File Deployment

Use:

```text
immutable release
 ↓
validation
 ↓
health check
 ↓
atomic promotion
 ↓
verification
```

---

# 191. Production Scenario — Need to Change Thousands of Files

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
```

Stop when the failure rate exceeds the agreed threshold.

---

# 192. Production Scenario — Python Runs as Root

Minimize privileges.

If root is required:

```text
strict paths
allowlist
dry run
bounded operations
logging
locking
tests
rollback
```

---

# 193. Production Scenario — Read-Only Filesystem

Investigate:

```text
mount
kernel logs
storage
filesystem health
disk state
```

Do not simply retry writes.

---

# 194. Production Scenario — Deleted File Still Uses Space

Use:

```bash
lsof +L1
```

A process may still hold the deleted file open.

---

# 195. Production Scenario — CI/CD Artifact Cleanup

Use:

```text
artifact age
artifact state
retention policy
active release
candidate count
total size
```

Never delete artifacts solely because their names look old.

---

# 196. Production Scenario — Different Files Across Servers

Compare:

```text
checksum
size
mtime
owner
group
permissions
```

Then compare the deployment process/environment.

---

# 197. Production Scenario — Need to Generate Configuration

Use:

```text
template
 ↓
render
 ↓
validate
 ↓
write temporary
 ↓
atomic replace
 ↓
reload
 ↓
health check
```

---

# 198. Production Scenario — Nginx Configuration

Use:

```text
generate
 ↓
nginx -t
 ↓
install
 ↓
reload
 ↓
health check
```

---

# 199. Production Scenario — Systemd Service File

Use:

```text
generate
 ↓
install
 ↓
daemon-reload
 ↓
enable/start
 ↓
status
 ↓
health check
```

---

# 200. Production Scenario — File Queue

Producer:

```text
write temporary
 ↓
atomic rename
```

Consumer:

```text
detect ready file
 ↓
process
 ↓
move to processed
```

---

# 201. Production Scenario — File Processing Fails

Move the file to:

```text
failed/
```

or another controlled failure state instead of repeatedly retrying forever.

---

# 202. Retry Strategy

For transient filesystem/network operations:

```text
attempt
 ↓
wait
 ↓
retry
```

Use:

```text
bounded retries
exponential backoff
jitter
```

when appropriate.

---

# 203. Don't Retry Permanent Errors

Examples:

```text
permission denied
invalid path
invalid configuration
unsupported file type
```

Usually require correction rather than repeated retries.

---

# 204. File Automation Architecture

```text
CLI
 |
 v
Validation
 |
 v
Discovery
 |
 v
Business Logic
 |
 +--> Copy
 |
 +--> Move
 |
 +--> Backup
 |
 +--> Cleanup
 |
 +--> Verify
 |
 v
Reporting
```

---

# 205. Recommended Project Structure

```text
file_manager/
├── cli.py
├── paths.py
├── scanner.py
├── copier.py
├── mover.py
├── cleanup.py
├── backup.py
├── checksum.py
├── validator.py
├── reporter.py
└── tests/
```

---

# 206. Separate Responsibilities

Avoid one 1000-line script.

Prefer:

```text
scanner
validator
executor
reporter
```

with clear responsibilities.

---

# 207. CLI Design

Example:

```bash
python file_manager.py scan \
    --path /opt/myapp
```

```bash
python file_manager.py backup \
    --path /etc/myapp
```

```bash
python file_manager.py cleanup \
    --path artifacts \
    --days 30 \
    --dry-run
```

---

# 208. Dry-Run Design

The same discovery logic should be used for:

```text
dry run
real execution
```

Only the final action changes.

---

# 209. Example Dry-Run

```text
Environment: staging

Would create:
  /opt/myapp/releases/42

Would copy:
  153 files

Would change:
  8 permissions

Would promote:
  release-42
```

---

# 210. Example Production Output

```text
Preflight      PASS
Validation     PASS
Copy           PASS
Checksum       PASS
Permissions    PASS
Promotion      PASS
Health Check   PASS

Status: SUCCESS
```

---

# 211. Failure Output

```text
Preflight      PASS
Validation     FAIL

Status: STOPPED

Reason:
invalid configuration schema
```

Stop instead of continuing.

---

# 212. File Automation Interview Questions

## Q1. Why use `pathlib`?

**Answer:**

> `pathlib` provides a cleaner object-oriented interface for filesystem paths and operations. It reduces manual string manipulation and makes code easier to maintain.

---

## Q2. What is the difference between `copy()` and `copy2()`?

**Answer:**

> Both copy file contents, while `copy2()` attempts to preserve additional metadata such as timestamps. I choose based on whether metadata preservation is required.

---

## Q3. How do you safely delete files?

**Answer:**

> I explicitly scope the root directory, validate the resolved paths, filter by known patterns and age, use dry-run mode, apply candidate limits, handle symlinks, and verify the final state.

---

## Q4. How do you prevent path traversal?

**Answer:**

> I resolve the candidate path and verify that it remains within an approved base directory. I also account for symlinks and reject unexpected absolute paths.

---

## Q5. Why use atomic file replacement?

**Answer:**

> It prevents readers from seeing partially written configuration or artifact files. I write and validate a temporary file first and then atomically replace the target.

---

## Q6. How do you process a 10 GB file?

**Answer:**

> I stream the file line by line or in chunks rather than loading it into memory. This keeps memory usage bounded.

---

## Q7. How do you find files recursively?

**Answer:**

```python
for file in Path(
    root
).rglob("*.log"):
    ...
```

For very large trees I consider lazy traversal and `os.scandir()`.

---

## Q8. How do you compare two files?

**Answer:**

> For small/simple comparisons I can use `filecmp`. For artifact integrity I generally use SHA-256 checksums.

---

## Q9. How do you make file automation idempotent?

**Answer:**

> I compare current state with desired state and make changes only when required. Operations such as `mkdir(..., exist_ok=True)` naturally support idempotency.

---

## Q10. How do you safely update YAML?

**Answer:**

> I parse YAML with a safe parser, validate the required schema, modify structured data, serialize it, and use an atomic replacement strategy for critical production files.

---

## Q11. How do you safely handle temporary files?

**Answer:**

> I use `tempfile`, avoid predictable filenames, keep temporary data scoped, and clean it up automatically where possible.

---

## Q12. How do you handle symlinks?

**Answer:**

> I explicitly detect and resolve symlinks when validating paths. For destructive operations I prevent links from redirecting the operation outside the intended directory.

---

## Q13. How do you verify an artifact?

**Answer:**

> I calculate a checksum and compare it with a trusted expected checksum before promoting or executing the artifact.

---

## Q14. How do you implement rollback?

**Answer:**

> I use versioned release directories and an active release pointer. A failed deployment can switch the pointer back to a known-good version.

---

## Q15. How do you prevent two deployment scripts from running together?

**Answer:**

> I use locking or CI/CD concurrency controls. For multi-host systems I use distributed coordination rather than a local file lock.

---

## Q16. How do you handle millions of files?

**Answer:**

> I avoid giant in-memory lists, use scoped/lazy traversal, process in batches, control concurrency, and reconsider the storage architecture if filesystem scanning becomes too expensive.

---

## Q17. How do you handle permission errors?

**Answer:**

> I identify whether the problem is file mode, ownership, parent directory permissions, ACL, security policy, or the process identity. I fix the actual cause rather than using broad permissions.

---

## Q18. Why should you not use `chmod 777`?

**Answer:**

> It grants unnecessary permissions and can create a security vulnerability. The correct approach is least privilege based on the application's required access.

---

## Q19. What is the difference between disk space and inode usage?

**Answer:**

> Disk space represents storage capacity while inodes represent filesystem objects. A filesystem can have free storage but no free inodes if it contains huge numbers of small files.

---

## Q20. How do you design a production cleanup script?

**Answer:**

> I define an explicit target, validate the path, filter by pattern and retention period, calculate candidate count and size, support dry-run, use locking, perform deletion only after safety checks, and verify/report the result.

---

# 213. Advanced Interview Scenario — Production Artifact Deployment

**Question:**

You receive a release archive and need to deploy it safely.

**Answer:**

```text
download
 ↓
checksum verification
 ↓
extract into temporary/versioned directory
 ↓
validate files
 ↓
validate configuration
 ↓
set ownership/permissions
 ↓
preflight
 ↓
health check
 ↓
atomic promotion
 ↓
post-deployment verification
```

If validation fails:

```text
do not promote
```

---

# 214. Advanced Interview Scenario — Log Cleanup

**Question:**

A production server is running out of space because logs are growing.

**Answer:**

> I first determine whether logrotate, journald, the application, container runtime, or centralized logging owns rotation. I identify the largest consumers and deleted-open files. Then I apply the approved retention policy, preferably compressing and deleting only eligible files. Finally I verify disk and inode recovery.

---

# 215. Advanced Interview Scenario — Configuration Corruption

**Answer:**

> I would restore the last known-good version, validate it, reload the service, and verify health. Then I would investigate whether the deployment was writing directly to the live file and change it to staged validation plus atomic replacement.

---

# 216. Advanced Interview Scenario — Cleanup Script Deletes Production Data

**Answer:**

> I would immediately stop the automation, preserve evidence, identify the deletion scope, restore from available backups/snapshots, and investigate the root cause. I would then add path restrictions, symlink protection, dry-run, candidate limits, approval, and regression tests.

---

# 217. Advanced Interview Scenario — File Permissions Differ Between Servers

**Answer:**

> I compare UID/GID, modes, umask, ACLs, filesystem type, security contexts, deployment copy method, and service users. I would make permissions explicit in the deployment process and verify them.

---

# 218. Advanced Interview Scenario — Application Cannot Read a Secret

**Answer:**

> I verify the secret path, mount, process identity, owner/group, mode, parent-directory permissions, container security context, and secret-management mechanism. I do not make the secret world-readable.

---

# 219. Advanced Interview Scenario — Two Releases Deploy Simultaneously

**Answer:**

> I would introduce deployment concurrency control, stage each release independently, and allow only one promotion to change the active release pointer. Each release should be immutable and independently verifiable.

---

# 220. Advanced Interview Scenario — Backup Restore Fails

**Answer:**

> I would classify the backup as unreliable, identify whether the issue is corruption, missing files, permissions, encryption, keys, or restore dependencies, and then add automated restore testing.

---

# 221. Advanced Interview Scenario — Archive Contains Malicious Paths

**Answer:**

> I would reject unsafe archive members before extraction. Each member path must resolve inside the intended extraction directory, and symlink/hardlink behavior must also be considered.

---

# 222. Advanced Interview Scenario — Need to Modify 50,000 Files

**Answer:**

> I would not execute all changes blindly. I would validate a sample, perform a canary batch, verify results, increase the batch size gradually, monitor failures, and stop automatically when a safety threshold is exceeded.

---

# 223. Advanced Interview Scenario — Filesystem Is Read-Only

**Answer:**

> I would investigate the mount state, kernel logs, storage errors, filesystem health, and underlying disk/storage condition. I would avoid repeated write retries until the root cause is understood.

---

# 224. Advanced Interview Scenario — Disk Looks Empty but Space Is Missing

**Answer:**

> I would investigate deleted-but-open files with tools such as `lsof +L1`, filesystem reservations, quotas, containers, and mount-specific usage. Directory scans alone may not reveal deleted-open files.

---

# 225. Advanced Interview Scenario — Artifact Changes Between Build and Deploy

**Answer:**

> I would use immutable versioned artifacts and verify checksums at promotion time. The deployment system should consume the exact artifact produced by the build rather than a mutable `latest` file.

---

# 226. Advanced Interview Scenario — Zero-Downtime File Deployment

**Answer:**

> I would use immutable release directories, validate the new release, perform a health check, atomically switch the active pointer, and keep the previous release available for rollback.

---

# 227. Real Project — File Management CLI

Build:

```text
file_manager.py
```

Commands:

```bash
python file_manager.py scan \
    --path /opt/myapp
```

```bash
python file_manager.py copy \
    --source release \
    --destination staging
```

```bash
python file_manager.py checksum \
    --path release.tar.gz
```

```bash
python file_manager.py cleanup \
    --path artifacts \
    --days 30 \
    --dry-run
```

---

# 228. Project Architecture

```text
file_manager/
├── cli.py
├── scanner.py
├── copier.py
├── cleanup.py
├── backup.py
├── checksum.py
├── archive.py
├── validator.py
├── reporter.py
└── tests/
```

---

# 229. Scanner Module

Responsibilities:

```text
find files
find directories
calculate sizes
calculate age
collect metadata
```

---

# 230. Copier Module

Responsibilities:

```text
validate source
validate destination
copy
preserve metadata where required
verify
```

---

# 231. Cleanup Module

Responsibilities:

```text
discover candidates
validate paths
dry run
safety limits
delete
report
```

---

# 232. Backup Module

Responsibilities:

```text
create backup
checksum
verify
retention
restore
```

---

# 233. Checksum Module

Responsibilities:

```text
SHA-256
stream large files
compare expected value
report mismatch
```

---

# 234. Archive Module

Responsibilities:

```text
create tar
create zip
extract safely
verify archive
```

---

# 235. Validator Module

Responsibilities:

```text
path validation
file type validation
schema validation
permissions
ownership
checksums
```

---

# 236. Reporter Module

Output:

```text
human-readable
JSON
CSV
exit status
```

---

# 237. Example Workflow

```text
CLI
 ↓
Validate
 ↓
Scan
 ↓
Plan
 ↓
Dry Run
 ↓
Execute
 ↓
Verify
 ↓
Report
```

---

# 238. Production Guardrails

For destructive operations:

```text
1. Explicit target
2. Approved base path
3. Path traversal protection
4. Symlink awareness
5. Dry run
6. Candidate count limit
7. Size limit
8. Allowlisted patterns
9. Lock
10. Audit log
11. Verification
12. Recovery plan
```

---

# 239. DevOps Mindset

Do not think:

```text
"How do I delete these files?"
```

Think:

```text
"Which files are safe to delete,
why are they safe,
what happens if my selection is wrong,
and how do I recover?"
```

---

# 240. Key Takeaways

```text
pathlib
    ↓
clean path handling

shutil
    ↓
copy/move/tree operations

tempfile
    ↓
safe staging

filecmp
    ↓
comparison

hashlib
    ↓
integrity

tarfile/zipfile
    ↓
archives

validation
    ↓
safety

atomic replacement
    ↓
safe updates

verification
    ↓
confidence

logging
    ↓
auditability
```

---

# 241. Final Production Checklist

```text
[ ] Is the target path explicit?
[ ] Is user input validated?
[ ] Can path traversal occur?
[ ] Can a symlink escape the target?
[ ] Is the operation idempotent?
[ ] Is dry-run supported?
[ ] Are destructive actions bounded?
[ ] Are permissions correct?
[ ] Is ownership correct?
[ ] Are secrets protected?
[ ] Are large files streamed?
[ ] Are temporary files secure?
[ ] Is atomic replacement used where required?
[ ] Are checksums used where appropriate?
[ ] Are archives validated?
[ ] Is concurrency controlled?
[ ] Are failures handled?
[ ] Is the result verified?
[ ] Is the operation logged?
[ ] Is rollback/recovery possible?
```

---

# 242. Final Interview Answer

> **I use Python file and directory automation to handle repeatable DevOps tasks such as artifact management, configuration generation, backups, cleanup, release staging, file validation, and deployment preparation. I prefer `pathlib` for path operations and use modules such as `shutil`, `tempfile`, `hashlib`, `filecmp`, and `tarfile` based on the requirement. In production, I focus on idempotency, path validation, symlink safety, least privilege, atomic updates, checksums, dry-run support, bounded destructive operations, logging, verification, and rollback.**

The key principle is:

```text
Automate the filesystem,
but never automate blindly.
```
