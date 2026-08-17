# File-Handling

## 1. Overview

File handling is one of the most practical parts of Python for DevOps.

DevOps engineers constantly work with:

- Linux configuration files
- application logs
- deployment manifests
- JSON files
- YAML files
- CSV reports
- environment files
- Terraform files
- Kubernetes manifests
- backup files
- temporary files
- generated reports
- CI/CD artifacts
- certificates
- inventory files

A typical automation flow is:

```text
Read file
   |
   v
Parse content
   |
   v
Validate
   |
   v
Transform
   |
   v
Write result
   |
   v
Report / Deploy
```

The important goal is not simply knowing how to open a file. The goal is being able to build **safe, reliable file-processing automation**.

---

# 2. Opening a File

Basic syntax:

```python
file = open("config.txt")
```

But this approach requires manually closing the file.

Prefer:

```python
with open("config.txt", encoding="utf-8") as file:
    content = file.read()
```

The `with` statement automatically closes the file.

---

# 3. Why `with` Matters

Bad:

```python
file = open("config.txt")

content = file.read()

file.close()
```

If an exception occurs before `close()`:

```text
file may remain open
```

Better:

```python
with open("config.txt", encoding="utf-8") as file:
    content = file.read()
```

The context manager handles cleanup.

---

# 4. File Modes

Common modes:

```text
r  -> read
w  -> write
a  -> append
x  -> create only
b  -> binary
t  -> text
+  -> read and write
```

Examples:

```python
open("file.txt", "r")
open("file.txt", "w")
open("file.txt", "a")
```

---

# 5. Read Mode

```python
with open("app.log", "r", encoding="utf-8") as file:
    content = file.read()

print(content)
```

If the file does not exist:

```text
FileNotFoundError
```

---

# 6. Write Mode

```python
with open("report.txt", "w", encoding="utf-8") as file:
    file.write("Deployment successful\n")
```

Important:

```text
w
```

truncates the existing file.

It does not append.

---

# 7. Append Mode

```python
with open("deployment.log", "a", encoding="utf-8") as file:
    file.write("Deployment completed\n")
```

Existing content remains.

New content is added at the end.

---

# 8. Create-Only Mode

```python
with open("newfile.txt", "x", encoding="utf-8") as file:
    file.write("Created")
```

If the file already exists:

```text
FileExistsError
```

This is useful when accidentally overwriting an existing file must be prevented.

---

# 9. Reading the Entire File

```python
with open("config.txt", encoding="utf-8") as file:
    content = file.read()
```

Good for:

```text
small files
configuration
small reports
```

Not ideal for huge logs.

---

# 10. Reading One Line

```python
with open("app.log", encoding="utf-8") as file:
    line = file.readline()

print(line)
```

---

# 11. Reading All Lines

```python
with open("app.log", encoding="utf-8") as file:
    lines = file.readlines()
```

This loads all lines into memory.

For large files, prefer iteration.

---

# 12. Iterating Over a File

```python
with open("app.log", encoding="utf-8") as file:
    for line in file:
        print(line)
```

This is much better for large log files because Python processes the file incrementally.

---

# 13. Strip Newline Characters

A line often contains:

```text
"ERROR database unavailable\n"
```

Use:

```python
line = line.strip()
```

Be careful: `strip()` removes whitespace from both ends, not only the newline.

---

# 14. Remove Only the Newline

If preserving other whitespace matters:

```python
line = line.rstrip("\n")
```

This is more precise.

---

# 15. Search a File

```python
with open("app.log", encoding="utf-8") as file:
    for line in file:
        if "ERROR" in line:
            print(line)
```

This is a simple but useful log-analysis pattern.

---

# 16. Count Errors

```python
error_count = 0

with open("app.log", encoding="utf-8") as file:
    for line in file:
        if "ERROR" in line:
            error_count += 1

print(f"Errors: {error_count}")
```

---

# 17. Build an Error Report

```python
errors = []

with open("app.log", encoding="utf-8") as file:
    for line in file:
        if "ERROR" in line:
            errors.append(line.strip())

print("\n".join(errors))
```

For very large logs, avoid retaining every matching line unless you actually need them.

---

# 18. Large Log File Processing

Do not do this blindly:

```python
content = file.read()
```

for multi-gigabyte logs.

Prefer:

```python
with open("application.log", encoding="utf-8") as file:
    for line in file:
        process(line)
```

Memory usage stays much lower.

---

# 19. Streaming File Processing

Conceptually:

```text
Large log
   |
   +--> line 1 --> process
   +--> line 2 --> process
   +--> line 3 --> process
   +--> ...
```

Instead of:

```text
Large log
   |
   v
load entire file
   |
   v
memory
```

---

# 20. Process Only Matching Lines

```python
def find_errors(path):
    with open(path, encoding="utf-8") as file:
        for line in file:
            if "ERROR" in line:
                yield line.rstrip("\n")
```

This uses a generator.

The caller can process matches one at a time.

---

# 21. Generators for Large Logs

```python
for error in find_errors("application.log"):
    print(error)
```

This is memory-efficient because matching lines are produced lazily.

---

# 22. File Encoding

Use explicit encoding when practical:

```python
open(
    "config.txt",
    encoding="utf-8",
)
```

This avoids depending on the system's default encoding.

---

# 23. Encoding Errors

Production environments may contain files with unexpected encodings.

Do not blindly use:

```python
errors="ignore"
```

because it can silently lose data.

Prefer to identify the encoding or fail clearly when data integrity matters.

---

# 24. Binary Files

For binary data:

```python
with open("certificate.der", "rb") as file:
    data = file.read()
```

Common binary files include:

```text
images
compressed files
archives
binary certificates
compiled artifacts
```

---

# 25. Writing Binary Data

```python
data = b"hello"

with open("data.bin", "wb") as file:
    file.write(data)
```

Use binary mode when the content is not text.

---

# 26. File Pointer

When reading:

```python
with open("file.txt", encoding="utf-8") as file:
    print(file.tell())
```

`tell()` returns the current position.

---

# 27. `seek()`

Move the file pointer:

```python
with open("file.txt", encoding="utf-8") as file:
    file.seek(0)
    content = file.read()
```

Useful when the same file object needs to be reread.

---

# 28. Reading in Chunks

For very large binary files:

```python
with open("large.bin", "rb") as file:
    while chunk := file.read(1024 * 1024):
        process(chunk)
```

This reads approximately 1 MB at a time.

---

# 29. File Existence

Using `pathlib`:

```python
from pathlib import Path

path = Path("config.yaml")

if path.exists():
    print("File exists")
```

---

# 30. Is It a File?

```python
if path.is_file():
    print("Regular file")
```

---

# 31. Is It a Directory?

```python
if path.is_dir():
    print("Directory")
```

---

# 32. File Size

```python
size = path.stat().st_size

print(size)
```

The result is in bytes.

---

# 33. File Metadata

```python
info = path.stat()

print(info.st_size)
print(info.st_mtime)
print(info.st_mode)
```

Useful metadata includes:

```text
size
modified time
permissions
ownership information
```

---

# 34. `pathlib` Paths

Prefer:

```python
from pathlib import Path

log_path = Path("/var/log/app.log")
```

over manually constructing paths:

```python
"/var/log/" + "app.log"
```

---

# 35. Joining Paths

```python
base = Path("/opt/app")

log = base / "logs" / "application.log"
```

This is cleaner and platform-aware.

---

# 36. Current Directory

```python
from pathlib import Path

current = Path.cwd()

print(current)
```

---

# 37. Home Directory

```python
home = Path.home()

print(home)
```

---

# 38. Absolute Path

```python
path = Path("config.yaml")

print(path.absolute())
```

For resolving symlinks and normalization, `resolve()` may be more appropriate.

---

# 39. Resolve a Path

```python
resolved = Path("config.yaml").resolve()
```

This can produce the absolute resolved path.

Be aware that behavior around missing paths and symlinks should be considered when using `resolve()`.

---

# 40. Create a Directory

```python
from pathlib import Path

Path("reports").mkdir()
```

If the directory already exists, an exception is raised.

---

# 41. Create Nested Directories

```python
Path("reports/2026/august").mkdir(
    parents=True,
    exist_ok=True,
)
```

This is useful for automation-generated reports.

---

# 42. Remove a File

```python
path = Path("temporary.txt")

if path.exists():
    path.unlink()
```

Use caution with automated deletion.

Always validate paths before destructive operations.

---

# 43. Remove an Empty Directory

```python
path = Path("empty-dir")

path.rmdir()
```

The directory must be empty.

---

# 44. Recursive Directory Removal

`shutil.rmtree()` can remove a directory tree.

```python
import shutil

shutil.rmtree("old-build")
```

This is highly destructive.

Never pass unvalidated user input directly into recursive deletion.

---

# 45. Copy a File

```python
import shutil

shutil.copy2(
    "config.yaml",
    "backup/config.yaml",
)
```

`copy2()` attempts to preserve metadata.

---

# 46. Move a File

```python
shutil.move(
    "old.log",
    "archive/old.log",
)
```

---

# 47. File Rename

With `pathlib`:

```python
path.rename("new-name.txt")
```

---

# 48. Safe Backup Before Modification

Before changing a production configuration:

```text
original
   |
   v
backup
   |
   v
modify
   |
   v
validate
   |
   v
activate
```

Do not modify critical files without a rollback strategy.

---

# 49. Temporary Files

Use `tempfile` instead of manually inventing names:

```python
import tempfile

with tempfile.NamedTemporaryFile(
    mode="w",
    encoding="utf-8",
) as file:
    file.write("temporary data")
```

Temporary resources should be cleaned up automatically when possible.

---

# 50. Temporary Directories

```python
import tempfile

with tempfile.TemporaryDirectory() as directory:
    print(directory)
```

Useful for:

```text
build artifacts
downloaded files
generated manifests
temporary reports
test data
```

---

# 51. File Permissions

On Linux, Python can inspect permissions:

```python
import os

mode = os.stat("script.sh").st_mode

print(oct(mode))
```

For many applications, `pathlib.Path.stat()` is a convenient starting point.

---

# 52. Change File Permissions

```python
from pathlib import Path

Path("script.sh").chmod(0o755)
```

Common:

```text
755 -> executable
644 -> regular readable file
600 -> owner-only sensitive file
```

Use the minimum permissions required.

---

# 53. Sensitive File Permissions

Examples:

```text
SSH private key -> restrictive permissions
credential file  -> restrictive permissions
application logs -> appropriate access controls
configuration    -> based on sensitivity
```

Python can modify permissions, but correct ownership and system-level controls are also important.

---

# 54. File Ownership

On Unix-like systems, ownership can be inspected with `os.stat()`.

Changing ownership generally requires OS-level privileges and may use:

```python
os.chown(...)
```

Do not run ownership-changing automation with unnecessary root privileges.

---

# 55. Atomic File Updates

For important configuration files, avoid:

```text
open original
truncate
write new content
```

If the process crashes midway, the file may become corrupted.

Prefer:

```text
write temporary file
       |
       v
validate
       |
       v
atomic replace
```

---

# 56. Atomic Replace

Python provides:

```python
import os

os.replace(
    "config.tmp",
    "config.yaml",
)
```

On supported filesystems, this provides an atomic replacement operation.

---

# 57. Production Configuration Update

Safer flow:

```text
Generate config
      |
      v
Write temp file
      |
      v
Parse / validate
      |
      v
Backup current config
      |
      v
Atomic replace
      |
      v
Reload service
      |
      v
Health check
```

This is much safer than directly editing a live file.

---

# 58. Configuration File Validation

Before deployment:

```python
from pathlib import Path

config = Path("config.yaml")

if not config.is_file():
    raise FileNotFoundError(
        "Configuration file missing"
    )
```

Then parse and validate its content.

---

# 59. Environment File

A simple `.env`-style file might contain:

```text
APP_ENV=production
PORT=8080
```

Do not blindly execute the file as Python or shell code.

Treat it as data and parse it using an appropriate library or controlled parser.

---

# 60. Avoid `eval()` for Configuration

Never do:

```python
config = eval(file.read())
```

on untrusted configuration.

`eval()` can execute arbitrary Python code.

Use:

```text
JSON
YAML safe loader
TOML
validated environment variables
```

as appropriate.

---

# 61. JSON File Handling

```python
import json

with open("config.json", encoding="utf-8") as file:
    config = json.load(file)
```

Write:

```python
with open("config.json", "w", encoding="utf-8") as file:
    json.dump(
        config,
        file,
        indent=2,
    )
```

---

# 62. YAML File Handling

Using PyYAML:

```python
import yaml

with open("config.yaml", encoding="utf-8") as file:
    config = yaml.safe_load(file)
```

Use:

```python
yaml.safe_load()
```

for untrusted YAML instead of unsafe loaders.

---

# 63. CSV File Handling

Python includes:

```python
import csv
```

Read:

```python
with open(
    "inventory.csv",
    newline="",
    encoding="utf-8",
) as file:
    reader = csv.DictReader(file)

    for row in reader:
        print(row)
```

This is useful for infrastructure inventories and reports.

---

# 64. CSV Write

```python
import csv

rows = [
    {
        "name": "web-01",
        "status": "running",
    },
    {
        "name": "web-02",
        "status": "stopped",
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
        fieldnames=["name", "status"],
    )

    writer.writeheader()
    writer.writerows(rows)
```

---

# 65. File Handling With Logs

Typical automation:

```text
application.log
      |
      v
Python
      |
      +--> count ERROR
      +--> count WARN
      +--> extract request IDs
      +--> find failed services
      +--> generate report
```

This is one of the most common practical file-processing tasks.

---

# 66. Log Analyzer Script

```python
from pathlib import Path


def analyze_log(path):
    errors = 0
    warnings = 0

    with Path(path).open(
        encoding="utf-8"
    ) as file:
        for line in file:
            if "ERROR" in line:
                errors += 1
            elif "WARN" in line:
                warnings += 1

    return {
        "errors": errors,
        "warnings": warnings,
    }
```

---

# 67. Log Analyzer CLI

Later, `argparse` can make it:

```bash
python log_analyzer.py \
    --file /var/log/application.log
```

Output:

```text
Errors: 14
Warnings: 32
```

The CLI layer should remain separate from the reusable analysis function.

---

# 68. Find Failed Deployments From a Report

```python
from pathlib import Path


def find_failed_lines(path):
    with Path(path).open(
        encoding="utf-8"
    ) as file:
        for line in file:
            if "FAILED" in line:
                yield line.rstrip("\n")
```

Generators are useful when the report can be large.

---

# 69. Count Lines Without Loading the File

```python
count = 0

with open("app.log", encoding="utf-8") as file:
    for _ in file:
        count += 1

print(count)
```

For simple shell-style operations, Linux tools such as `wc -l` may be faster or more natural, but Python is useful when the count is part of a larger automation workflow.

---

# 70. Tail-Like File Reading

A simplistic approach:

```python
from collections import deque

with open("app.log", encoding="utf-8") as file:
    last_lines = deque(
        file,
        maxlen=100,
    )
```

This keeps only the last 100 lines.

For extremely large or actively written files, specialized tail logic may be needed.

---

# 71. File Locking

When multiple processes may write the same file, consider concurrency.

Possible solutions include:

```text
file locking
atomic replacement
single writer
database
queue
```

Python's standard library does not provide one universal cross-platform file-locking abstraction.

Choose a solution appropriate to the operating system and workload.

---

# 72. Race Conditions

This pattern can be unsafe:

```python
if not path.exists():
    path.write_text("data")
```

Another process can create the file between the check and write.

For exclusive creation:

```python
path.open("x", encoding="utf-8")
```

Use atomic operations when correctness depends on avoiding races.

---

# 73. Symlink Considerations

A path may point to a symbolic link.

Before destructive operations, understand whether you are operating on:

```text
the link
```

or:

```text
the target
```

Security-sensitive automation should validate paths carefully.

---

# 74. Path Traversal

Never trust user input:

```python
filename = user_input
path = Path("/opt/app") / filename
```

because input such as:

```text
../../etc/passwd
```

may escape the intended directory.

Validate and constrain paths.

---

# 75. Safe Path Validation

Conceptually:

```python
base = Path("/opt/app").resolve()
target = (base / user_input).resolve()

if base not in target.parents and target != base:
    raise ValueError("Invalid path")
```

For security-sensitive systems, test edge cases involving symlinks and filesystem boundaries carefully.

---

# 76. File Name Validation

If a script expects:

```text
backup.tar.gz
```

do not accept arbitrary input without validation.

Use explicit rules:

```text
allowed characters
allowed extension
maximum length
expected directory
```

---

# 77. Backup Script Design

A production backup script should consider:

```text
source path
destination
timestamp
permissions
disk space
retention
failure handling
verification
logging
```

Basic flow:

```text
source
  |
  v
validate
  |
  v
create destination
  |
  v
copy
  |
  v
verify
  |
  v
report
```

---

# 78. Timestamped Backup

```python
from datetime import datetime, timezone
from pathlib import Path
import shutil

timestamp = datetime.now(
    timezone.utc
).strftime("%Y%m%dT%H%M%SZ")

source = Path("config.yaml")
destination = Path(
    f"backup/config-{timestamp}.yaml"
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

Use UTC timestamps for consistent automation across environments.

---

# 79. Backup Verification

After copying:

```python
if destination.exists():
    print("Backup created")
```

For stronger verification:

```text
file exists
size matches
checksum matches
permissions are correct
```

For critical data, checksum verification can provide stronger assurance.

---

# 80. File Checksums

Using `hashlib`:

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

This processes large files incrementally.

---

# 81. Backup Checksum Verification

```python
source_hash = sha256_file(source)
backup_hash = sha256_file(destination)

if source_hash != backup_hash:
    raise RuntimeError(
        "Backup verification failed"
    )
```

This is useful when backup integrity matters.

---

# 82. Cleanup Script

A common DevOps task is deleting old files.

Example:

```python
from pathlib import Path
from datetime import datetime, timedelta, timezone

cutoff = datetime.now(
    timezone.utc
) - timedelta(days=7)

for path in Path("reports").glob("*.log"):
    modified = datetime.fromtimestamp(
        path.stat().st_mtime,
        tz=timezone.utc,
    )

    if modified < cutoff:
        path.unlink()
```

Production cleanup should include:

```text
dry-run
path validation
logging
exclusions
error handling
retention policy
```

---

# 83. Always Add Dry Run to Destructive Automation

Instead of immediately deleting:

```text
DELETE old.log
```

support:

```bash
python cleanup.py --dry-run
```

Output:

```text
WOULD DELETE: reports/old.log
WOULD DELETE: reports/old2.log
```

Then execute only after verification.

---

# 84. Cleanup Script Architecture

```text
discover files
      |
      v
filter by policy
      |
      v
dry-run?
   /       \
 yes       no
  |         |
 report    delete
            |
            v
         verify
```

---

# 85. Disk Cleanup

A production cleanup script may target:

```text
old application logs
old build artifacts
old temporary files
unused reports
old Docker artifacts
```

Do not blindly delete:

```text
/var/log/*
```

or other system directories.

Use explicit paths and retention rules.

---

# 86. File Rotation Concept

Instead of allowing:

```text
application.log
```

to grow forever:

```text
application.log
application.log.1
application.log.2
...
```

Log rotation can be handled by:

```text
logrotate
application logging libraries
container runtime
platform logging systems
```

Python can interact with or generate logs, but it does not need to reinvent OS-level log rotation.

---

# 87. Configuration Generation

Python can generate:

```text
nginx.conf
docker-compose.yaml
kubernetes.yaml
terraform.tfvars
application.properties
```

But generation should be:

```text
template
   +
validated data
   |
   v
generated file
```

Avoid fragile string concatenation for complex structured formats.

---

# 88. Template-Based Configuration

Conceptually:

```text
template
   |
   +-- service name
   +-- image tag
   +-- replicas
   +-- environment
   |
   v
generated manifest
```

For Kubernetes/YAML, use a proper YAML library or templating system rather than manual string manipulation where appropriate.

---

# 89. Kubernetes Manifest Modification

After loading YAML:

```python
manifest["spec"]["replicas"] = 4
```

Then serialize it.

This is safer than:

```python
text.replace("replicas: 3", "replicas: 4")
```

because text replacement can modify the wrong location.

---

# 90. Terraform File Handling

Python may:

```text
read tfvars
generate tfvars
validate files
run terraform fmt
run terraform validate
parse JSON output
```

Avoid using regex as the primary parser for Terraform syntax when a structured representation is available.

---

# 91. CI/CD Artifact Handling

A pipeline can produce:

```text
test-report.json
security-report.json
deployment-report.csv
build-info.json
```

Python can:

```text
read
validate
aggregate
summarize
archive
publish
```

these files.

---

# 92. Build Metadata File

Example:

```python
build_info = {
    "commit": "abc123",
    "branch": "main",
    "image": "payment:v42",
    "environment": "production",
}
```

Write:

```python
import json

with open(
    "build-info.json",
    "w",
    encoding="utf-8",
) as file:
    json.dump(
        build_info,
        file,
        indent=2,
    )
```

---

# 93. Deployment Artifact Validation

Before deploying:

```python
required_files = [
    "deployment.yaml",
    "service.yaml",
    "config.yaml",
]

for filename in required_files:
    if not Path(filename).is_file():
        raise FileNotFoundError(filename)
```

This catches missing artifacts before deployment.

---

# 94. File-Based Lock

Sometimes a script should prevent concurrent execution.

A simple lock-file concept:

```text
script starts
    |
    v
check lock
    |
    +--> exists -> exit
    |
    +--> missing -> create lock
                       |
                       v
                    execute
                       |
                       v
                   remove lock
```

For reliable production concurrency control, use OS locking or another robust mechanism rather than relying only on a marker file.

---

# 95. Temporary Artifact Cleanup

Use:

```python
from tempfile import TemporaryDirectory

with TemporaryDirectory() as directory:
    # create temporary files
    # process them
    pass
```

Cleanup happens automatically when the context exits.

This is safer than manually tracking temporary paths.

---

# 96. File Handling Error Types

Common exceptions:

```text
FileNotFoundError
PermissionError
FileExistsError
IsADirectoryError
NotADirectoryError
UnicodeDecodeError
OSError
```

Catch specific exceptions where recovery is possible.

---

# 97. `FileNotFoundError`

```python
try:
    with open("config.yaml", encoding="utf-8") as file:
        data = file.read()
except FileNotFoundError:
    print("Configuration missing")
```

In production, decide whether missing configuration should:

```text
fail immediately
use a documented default
request operator action
```

---

# 98. `PermissionError`

```python
try:
    with open("/etc/app/config", encoding="utf-8") as file:
        content = file.read()
except PermissionError:
    print("Insufficient permissions")
```

Do not automatically run the entire script as root just to bypass permission problems.

---

# 99. Permission Troubleshooting

Linux commands:

```bash
ls -l /path/to/file
id
whoami
```

Check:

```text
owner
group
mode
directory permissions
ACLs
mount options
SELinux/AppArmor where applicable
```

Python file handling problems are often Linux permission problems rather than Python problems.

---

# 100. `PermissionError` in CI/CD

If a script works locally but fails in CI:

```text
developer user
      !=
CI runner user
```

Check:

```bash
id
ls -l
pwd
```

and verify the workspace permissions.

---

# 101. File Descriptor Concept

An opened file is associated with a file descriptor managed by the operating system.

Leaving many files open can exhaust process resources.

Using:

```python
with open(...):
```

helps ensure descriptors are released promptly.

---

# 102. Too Many Open Files

Possible error:

```text
OSError: [Errno 24] Too many open files
```

Possible causes:

```text
files not closed
too many concurrent files
descriptor leak
large workload
```

Use context managers and inspect OS limits when necessary:

```bash
ulimit -n
```

---

# 103. Process Many Files

```python
from pathlib import Path

for path in Path("logs").glob("*.log"):
    print(path)
```

For recursive search:

```python
for path in Path("logs").rglob("*.log"):
    print(path)
```

---

# 104. Filter Files by Size

```python
for path in Path("logs").glob("*.log"):
    if path.stat().st_size > 100 * 1024 * 1024:
        print(path)
```

This can identify large logs.

---

# 105. Find Recently Modified Files

```python
from datetime import datetime, timedelta, timezone

cutoff = datetime.now(
    timezone.utc
) - timedelta(hours=1)

for path in Path("logs").glob("*.log"):
    modified = datetime.fromtimestamp(
        path.stat().st_mtime,
        tz=timezone.utc,
    )

    if modified > cutoff:
        print(path)
```

Useful for incident investigations.

---

# 106. Find Old Files

```python
cutoff = datetime.now(
    timezone.utc
) - timedelta(days=30)

for path in Path("reports").glob("*"):
    if not path.is_file():
        continue

    modified = datetime.fromtimestamp(
        path.stat().st_mtime,
        tz=timezone.utc,
    )

    if modified < cutoff:
        print(path)
```

---

# 107. Generate Inventory Report

```python
inventory = []

for path in Path("artifacts").rglob("*"):
    if path.is_file():
        inventory.append({
            "name": path.name,
            "path": str(path),
            "size": path.stat().st_size,
        })
```

This data can later be written to:

```text
JSON
CSV
database
dashboard
```

---

# 108. JSON Inventory Report

```python
import json

with open(
    "inventory.json",
    "w",
    encoding="utf-8",
) as file:
    json.dump(
        inventory,
        file,
        indent=2,
    )
```

---

# 109. CSV Inventory Report

```python
import csv

with open(
    "inventory.csv",
    "w",
    newline="",
    encoding="utf-8",
) as file:
    writer = csv.DictWriter(
        file,
        fieldnames=[
            "name",
            "path",
            "size",
        ],
    )

    writer.writeheader()
    writer.writerows(inventory)
```

---

# 110. Log Parsing With Regex

File handling and regex often work together:

```python
import re

pattern = re.compile(
    r"ERROR.*request_id=(\w+)"
)

with open(
    "application.log",
    encoding="utf-8",
) as file:
    for line in file:
        match = pattern.search(line)

        if match:
            print(match.group(1))
```

Regex is covered in depth in:

```text
04-Regex.md
```

---

# 111. Extract IP Addresses From a File

Conceptually:

```python
import re

pattern = re.compile(
    r"\b(?:\d{1,3}\.){3}\d{1,3}\b"
)

with open(
    "network.log",
    encoding="utf-8",
) as file:
    for line in file:
        for ip in pattern.findall(line):
            print(ip)
```

A regex match is not automatically proof that an IP address is semantically valid. Validate ranges if correctness matters.

---

# 112. Extract Deployment Versions

Suppose:

```text
deployed image=payment:v42
```

A regex can extract:

```text
v42
```

This is useful for:

```text
deployment reports
rollback scripts
audit logs
release verification
```

---

# 113. Compare Two Configuration Files

For text:

```python
from pathlib import Path

old = Path("old.conf").read_text(
    encoding="utf-8"
)

new = Path("new.conf").read_text(
    encoding="utf-8"
)

if old == new:
    print("No changes")
```

For structured formats, parse first and compare data structures.

---

# 114. Configuration Diff

Conceptual:

```text
old config
     |
     v
parse
     |
     v
dictionary

new config
     |
     v
parse
     |
     v
dictionary

        |
        v
     compare
```

This is much more reliable than comparing formatting differences in YAML/JSON when the goal is semantic configuration comparison.

---

# 115. File-Based Deployment Validation

A useful pre-deployment flow:

```text
manifest files
      |
      v
existence check
      |
      v
syntax parsing
      |
      v
required fields
      |
      v
security checks
      |
      v
deployment
```

Python can orchestrate this workflow.

---

# 116. File-Based Rollback

A simple rollback design:

```text
current.yaml
     |
     v
backup.yaml
     |
     v
new.yaml
     |
     v
deploy
     |
     +--> success
     |
     +--> failure
              |
              v
         restore backup
```

For Kubernetes, GitOps or declarative deployment tooling may provide stronger rollback mechanisms than manually restoring files.

---

# 117. Never Blindly Modify Production Files

Bad:

```python
text = text.replace(
    "replicas: 3",
    "replicas: 5",
)

path.write_text(text)
```

Potential problems:

```text
wrong occurrence changed
formatting altered
file corrupted
no backup
no validation
```

Use structured parsing or templates.

---

# 118. Safe Configuration Workflow

```text
Read
 |
 v
Parse
 |
 v
Validate
 |
 v
Modify in memory
 |
 v
Serialize
 |
 v
Validate generated result
 |
 v
Write temporary file
 |
 v
Atomic replace
```

This pattern is useful for production configuration automation.

---

# 119. File Handling and Secrets

Never casually write:

```python
password
token
private_key
```

to:

```text
logs
temporary files
reports
CI artifacts
```

If a secret must temporarily exist in a file:

```text
restrict permissions
limit lifetime
avoid logging path/content
delete securely according to platform requirements
```

---

# 120. Avoid Secrets in Exceptions

Bad:

```python
raise RuntimeError(
    f"Failed using config: {config}"
)
```

If `config` contains credentials, the exception can leak them into CI logs.

Raise safe error messages.

---

# 121. File Handling in Jenkins

A Python deployment helper may:

```text
read build metadata
read environment configuration
generate deployment manifest
run validation
write deployment report
archive report
```

Important:

```text
workspace cleanup
permissions
credentials
artifact retention
concurrent builds
```

must also be considered.

---

# 122. File Handling in GitHub Actions

A workflow may:

```text
checkout repository
create Python environment
generate config
run tests
write report
upload artifact
```

Do not assume the runner's filesystem persists beyond the job unless the workflow explicitly stores artifacts/cache.

---

# 123. File Handling in Kubernetes

Containers often have ephemeral filesystems.

Do not assume:

```text
file written inside container
```

will survive:

```text
pod restart
rescheduling
deployment rollout
```

For persistent data use:

```text
PersistentVolume
object storage
database
external service
```

depending on the requirement.

---

# 124. Container Log Files

Avoid building a production design that depends on writing application logs only to local container files.

Typical Kubernetes pattern:

```text
application
    |
    v
stdout/stderr
    |
    v
container runtime
    |
    v
log collector
    |
    v
centralized logging
```

Python can process files for utilities and offline analysis, but application logging architecture should match the platform.

---

# 125. File Handling and EKS

A Python EKS automation script might:

```text
generate temporary manifest
       |
       v
validate YAML
       |
       v
run kubectl
       |
       v
capture result
       |
       v
write deployment report
       |
       v
cleanup temp files
```

Use a temporary directory for intermediate artifacts.

---

# 126. File Handling and ELK

A log-processing script may:

```text
read exported logs
       |
       v
filter errors
       |
       v
extract fields
       |
       v
write JSON report
       |
       v
send to API / archive
```

For production-scale centralized logs, use the logging platform rather than processing multi-gigabyte files manually on application servers.

---

# 127. File Handling and Prometheus

Prometheus commonly works through metrics endpoints rather than ordinary log files.

Python may:

```text
query Prometheus API
write result to JSON/CSV
generate report
```

File handling becomes the reporting layer rather than the metrics collection mechanism.

---

# 128. File Handling and Terraform

A Python helper can:

```text
create temporary tfvars
run terraform fmt
run terraform validate
run terraform plan
capture output
write report
cleanup
```

Important:

```text
never expose credentials
control working directory
capture exit codes
set timeout
```

---

# 129. File Handling and Security Scans

A CI script may read:

```text
trivy.json
sonarqube.json
security-report.csv
```

Then:

```text
parse
   |
   v
count critical/high
   |
   v
compare policy
   |
   v
fail or pass pipeline
```

This is a realistic DevSecOps file-processing workflow.

---

# 130. Security Scan Gate Example

Conceptual:

```python
if report["critical"] > 0:
    raise SystemExit(
        "Security gate failed"
    )
```

A real implementation should validate the report schema first.

---

# 131. File Handling and Artifactory

Python may:

```text
generate artifact metadata
calculate checksum
create report
upload through API
verify response
```

The actual artifact storage should be handled by the approved repository tooling/API.

---

# 132. File Handling and Certificates

A Python script can inspect:

```text
certificate file
expiration
subject
issuer
```

The resulting report can be:

```text
certificate-report.json
```

For production certificate management, integrate with the organization's certificate lifecycle tooling.

---

# 133. Certificate Expiry Report

Conceptually:

```python
report = {
    "certificate": "api.example.com",
    "expires": "...",
    "days_remaining": 21,
    "status": "warning",
}
```

This can be written to JSON or CSV and consumed by CI/monitoring.

---

# 134. Production Script — File Health Check

A reusable check might validate:

```text
exists
regular file
minimum size
maximum age
permissions
checksum
```

Example:

```python
from pathlib import Path


def validate_file(path, max_age=None):
    path = Path(path)

    if not path.is_file():
        raise FileNotFoundError(path)

    if path.stat().st_size == 0:
        raise ValueError(
            f"Empty file: {path}"
        )

    return True
```

Extend with age and permission checks as needed.

---

# 135. Production Script — Required Artifacts

```python
from pathlib import Path


def validate_artifacts(directory, required):
    directory = Path(directory)

    missing = [
        name
        for name in required
        if not (directory / name).is_file()
    ]

    if missing:
        raise FileNotFoundError(
            f"Missing artifacts: {missing}"
        )
```

This is useful in CI/CD.

---

# 136. Production Script — Log Error Summary

```python
from pathlib import Path


def summarize_log(path):
    summary = {
        "errors": 0,
        "warnings": 0,
    }

    with Path(path).open(
        encoding="utf-8"
    ) as file:
        for line in file:
            if "ERROR" in line:
                summary["errors"] += 1
            elif "WARN" in line:
                summary["warnings"] += 1

    return summary
```

This can be reused by a CI report generator.

---

# 137. Production Script — Cleanup With Dry Run

Conceptual interface:

```text
cleanup.py
    |
    +-- --directory
    +-- --days
    +-- --dry-run
```

Flow:

```text
discover
   |
filter
   |
dry-run?
 /      \
yes      no
 |        |
report   delete
```

This pattern should be used for destructive maintenance scripts.

---

# 138. Production Script — Backup

```text
backup.py
    |
    +-- source
    +-- destination
    +-- retention
    +-- verify
```

Flow:

```text
validate source
      |
      v
create backup
      |
      v
checksum
      |
      v
report
      |
      v
retention cleanup
```

---

# 139. File Handling Interview — `read()` vs Iteration

Answer:

> `read()` loads the entire file into memory, so it is appropriate for small files. Iterating over the file processes it line by line and is more memory-efficient for large logs. For multi-gigabyte production logs, I normally stream the file rather than loading it all into memory.

---

# 140. Interview — Why Use `with open()`?

Answer:

> It uses a context manager that automatically closes the file even if an exception occurs. This prevents file descriptor leaks and makes resource management safer.

---

# 141. Interview — `w` vs `a`

Answer:

> `w` opens a file for writing and truncates existing content. `a` opens it for appending and preserves existing content. For production configuration files I avoid accidental `w` operations unless replacing the file is intentional.

---

# 142. Interview — How Do You Process a Large Log?

Answer:

> I would stream it line by line rather than calling `read()` on the entire file. If I only need matching records, I can use a generator so that results are processed lazily.

---

# 143. Interview — How Do You Safely Modify Configuration?

Answer:

> I parse the configuration, validate it, modify the structured representation, serialize it to a temporary file, validate the generated file, and then atomically replace the original. For critical production configuration I also keep a rollback/backup strategy.

---

# 144. Interview — Why `pathlib`?

Answer:

> `pathlib` provides an object-oriented and cross-platform way to work with filesystem paths. It makes operations such as joining paths, checking files, creating directories, and reading metadata clearer than manual string concatenation.

---

# 145. Interview — How Do You Handle File Permissions?

Answer:

> First I inspect the running user, file ownership, mode bits, directory permissions, and relevant OS security controls. I don't simply run the whole automation as root. I grant the minimum permissions required.

---

# 146. Interview — How Do You Handle Temporary Files?

Answer:

> I use `tempfile.NamedTemporaryFile` or `TemporaryDirectory` rather than manually creating predictable temporary names. This reduces collisions and makes cleanup easier.

---

# 147. Interview — How Do You Prevent Path Traversal?

Answer:

> I don't trust user-supplied paths. I resolve the target against an approved base directory and verify that the resulting path remains inside that directory. I also consider symlinks and filesystem permissions for security-sensitive operations.

---

# 148. Interview — How Do You Generate Kubernetes YAML?

Answer:

> I prefer structured YAML manipulation or templates rather than fragile string replacement. I validate required fields and the generated manifest before applying it.

---

# 149. Interview — How Do You Handle Secrets in Files?

Answer:

> I avoid writing secrets to ordinary files whenever possible. I use approved secret-management systems, avoid logging secret contents, restrict permissions if temporary files are unavoidable, and ensure temporary artifacts are cleaned up.

---

# 150. Scenario — Disk Is 100% Full

Python script:

```text
find large files
   |
   v
identify old files
   |
   v
generate report
   |
   v
dry-run cleanup
   |
   v
delete approved files
```

But first verify:

```bash
df -h
df -i
du -xhd1 /
```

Python is the automation layer; Linux commands and filesystem concepts are still essential for diagnosis.

---

# 151. Scenario — Log File Is 8 GB

Do not:

```python
content = file.read()
```

Instead:

```python
for line in file:
    process(line)
```

If only the last N lines are needed, use a bounded approach such as `deque`.

If the application is in Kubernetes, also investigate centralized logging rather than relying on one huge local file.

---

# 152. Scenario — Configuration File Became Empty

Possible cause:

```python
open("config.yaml", "w")
```

was executed before validation completed.

Better:

```text
generate temporary file
validate
atomic replace
```

Also maintain backups for critical configuration.

---

# 153. Scenario — CI Cannot Read File

Check:

```bash
id
ls -l file
ls -ld directory
pwd
```

Then investigate:

```text
ownership
permissions
workspace
container user
mounted volume
SELinux/AppArmor
```

Do not assume Python is the root cause.

---

# 154. Scenario — Cleanup Script Deleted Wrong Files

Likely design flaw:

```text
unvalidated path
broad glob
no dry run
no exclusions
no retention verification
```

Improve:

```text
explicit base directory
resolved paths
allowed extensions
age threshold
dry-run
logging
approval where required
```

---

# 155. Scenario — Backup Exists but Is Corrupt

Add:

```text
copy
   |
checksum
   |
compare
   |
success/failure
```

For critical backups, also perform periodic restore testing.

A backup that has never been successfully restored should not be treated as fully reliable.

---

# 156. Scenario — File Processing Uses Too Much Memory

Look for:

```python
read()
readlines()
list(file)
```

Replace with:

```python
for line in file:
    ...
```

or:

```python
generator
```

Also avoid accumulating unnecessary results in large lists.

---

# 157. Scenario — File Is Modified While Script Reads It

Possible causes:

```text
application writing concurrently
log rotation
deployment replacing file
network filesystem behavior
```

Consider:

```text
file rotation semantics
snapshot/copy
locking where appropriate
retry
read-only processing
```

For active logs, design around the logging/rotation system rather than assuming the file is static.

---

# 158. Scenario — Script Works on Linux but Not Windows

Possible issue:

```text
hard-coded /
path separators
Linux commands
permissions
systemctl
```

Use:

```python
pathlib
```

for paths where cross-platform support matters.

If the script intentionally targets Linux servers, document the platform dependency.

---

# 159. Scenario — UnicodeDecodeError

Possible causes:

```text
wrong encoding
binary file opened as text
mixed encoding
corrupted input
```

Investigate the actual file encoding rather than simply ignoring decoding errors.

---

# 160. Scenario — File Descriptor Exhaustion

Symptoms:

```text
Too many open files
```

Check:

```text
open() calls
missing context managers
concurrency
OS limits
```

Use:

```python
with open(...):
```

and avoid keeping unnecessary files open.

---

# 161. Practical Project — DevOps File Utility

Build:

```text
devops-file-tools/
│
├── scripts/
│   ├── log_summary.py
│   ├── cleanup.py
│   ├── backup.py
│   └── artifact_check.py
│
├── devops_utils/
│   ├── __init__.py
│   ├── files.py
│   ├── logs.py
│   └── validation.py
│
├── tests/
│   ├── test_files.py
│   └── test_logs.py
│
└── requirements.txt
```

This project combines the module concepts from the previous file with file handling.

---

# 162. Project — Log Summary

Requirements:

```text
--file
--error-pattern
--warning-pattern
```

Output:

```text
File: application.log
Lines: 125000
Errors: 42
Warnings: 117
```

Use streaming rather than loading the entire file.

---

# 163. Project — Artifact Validator

Requirements:

```text
--directory
--required-file
```

Check:

```text
exists
regular file
non-empty
```

Exit non-zero if validation fails.

This can be integrated into CI/CD.

---

# 164. Project — Safe Cleanup

Requirements:

```text
--directory
--days
--dry-run
```

Output:

```text
WOULD DELETE: old-report.log
WOULD DELETE: build-2026-07.zip
```

Only delete when `--dry-run` is not specified.

---

# 165. Project — Configuration Backup

Requirements:

```text
source
backup directory
timestamp
checksum
```

Flow:

```text
validate source
   |
   v
copy
   |
   v
checksum
   |
   v
verify
   |
   v
report
```

---

# 166. Project — Kubernetes Manifest Generator

Input:

```text
service
image
replicas
port
```

Generate structured YAML.

Validate:

```text
service
deployment
image
replicas
port
```

Then produce:

```text
deployment.yaml
service.yaml
```

---

# 167. Project — CI Security Report Processor

Input:

```text
trivy.json
```

Process:

```text
critical
high
medium
low
```

Generate:

```text
security-summary.json
security-summary.csv
```

Fail the pipeline if the configured security policy is violated.

---

# 168. Production File Handling Architecture

```text
                 External Systems
                       |
          +------------+------------+
          |            |            |
        AWS       Kubernetes       CI
          |            |            |
          +------------+------------+
                       |
                       v
                 Python Script
                       |
                       v
                Parse / Validate
                       |
                       v
                  File Utility
                       |
          +------------+------------+
          |            |            |
         JSON         YAML         CSV
          |            |            |
          +------------+------------+
                       |
                       v
                  Report / Action
```

---

# 169. File Handling Best Practices

```text
1. Use with open().
2. Specify encoding for text.
3. Stream large files.
4. Use pathlib for paths.
5. Validate files before modifying them.
6. Use temporary files for atomic updates.
7. Keep backups for critical configuration.
8. Use dry-run for destructive scripts.
9. Never trust user-controlled paths.
10. Never log secrets.
11. Use structured parsing for JSON/YAML.
12. Handle specific exceptions.
13. Use checksums when integrity matters.
14. Control file permissions.
15. Clean temporary artifacts.
16. Test failure scenarios.
```

---

# 170. Final DevOps Mental Model

File handling is not just:

```python
open()
read()
write()
```

For a DevOps engineer, it is:

```text
             FILE / ARTIFACT
                    |
                    v
               DISCOVER
                    |
                    v
                VALIDATE
                    |
                    v
                  PARSE
                    |
                    v
               TRANSFORM
                    |
          +---------+---------+
          |                   |
          v                   v
       REPORT              ACTION
          |                   |
          +---------+---------+
                    |
                    v
                VERIFY
                    |
                    v
                 CLEANUP
```

The key principle:

> **Treat files as production resources. Validate before modifying, stream large data, avoid unsafe path handling, protect sensitive content, use atomic updates for critical configuration, and make destructive operations safe through dry-runs, retention policies, and verification.**

---