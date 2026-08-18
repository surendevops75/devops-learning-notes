# 05-Backup-Automation

## Python Automation — Backup and Restore

This module covers **production-oriented backup automation with Python**.

A backup is not simply:

```text
copy files
```

A reliable backup system must provide:

```text
selection
    ↓
consistency
    ↓
encryption/security
    ↓
backup
    ↓
integrity verification
    ↓
retention
    ↓
monitoring
    ↓
restore testing
```

The most important DevOps principle is:

> **A backup that has never been successfully restored is only an assumption.**

---

# 1. What Is Backup Automation?

Backup automation uses code and scheduling to consistently protect:

```text
configuration
application data
database dumps
deployment artifacts
logs where required
infrastructure state
certificates where appropriate
```

Python can orchestrate:

```text
file collection
compression
checksums
encryption workflows
upload
retention
verification
restore
reporting
alerting
```

---

# 2. Why Backups Matter in DevOps

Production failures can cause:

```text
data corruption
accidental deletion
bad deployment
ransomware
hardware failure
operator error
configuration loss
region failure
application failure
```

Backups provide a recovery path.

---

# 3. Backup vs Snapshot

A backup is generally:

```text
independent copy of data
```

A snapshot is generally:

```text
point-in-time storage state
```

Snapshots are useful for fast recovery, but they should not automatically be treated as a complete backup strategy.

---

# 4. Backup vs Replication

Replication:

```text
primary
   ↓
replica
```

helps availability.

Backup:

```text
primary
   ↓
independent historical copy
```

helps recovery from corruption or accidental deletion.

Replication can copy bad changes.

---

# 5. RPO

RPO:

```text
Recovery Point Objective
```

means:

> How much data loss can the business tolerate?

Example:

```text
RPO = 1 hour
```

means the recovery process should generally lose no more than approximately one hour of data, subject to system design.

---

# 6. RTO

RTO:

```text
Recovery Time Objective
```

means:

> How long can recovery take?

Example:

```text
RTO = 30 minutes
```

---

# 7. Backup Strategy

A complete design starts with:

```text
RPO
RTO
data criticality
retention
recovery location
security requirements
restore requirements
```

---

# 8. Backup Types

Common types:

```text
full
incremental
differential
snapshot
logical database backup
physical database backup
configuration backup
```

---

# 9. Full Backup

A full backup contains the selected dataset.

Advantages:

```text
simple restore
self-contained
```

Disadvantages:

```text
larger
slower
more storage
```

---

# 10. Incremental Backup

An incremental backup stores changes since the previous backup.

Example:

```text
Sunday → full
Monday → changes
Tuesday → changes
Wednesday → changes
```

Restore may require multiple backup sets.

---

# 11. Differential Backup

A differential backup stores changes since the last full backup.

Example:

```text
Sunday → full
Monday → changes since Sunday
Tuesday → changes since Sunday
Wednesday → changes since Sunday
```

---

# 12. Backup Retention

Example policy:

```text
daily    → 7 days
weekly   → 4 weeks
monthly  → 12 months
```

Actual retention must follow business, compliance, and storage requirements.

---

# 13. 3-2-1 Backup Principle

A common strategy:

```text
3 copies
2 different storage/media types
1 offsite copy
```

Modern environments may extend this with immutable/offline copies.

---

# 14. Immutable Backups

An immutable backup cannot be modified or deleted during its protection period.

Useful against:

```text
ransomware
accidental deletion
malicious modification
```

---

# 15. Backup Security

Protect backups with:

```text
encryption
access control
least privilege
MFA
separate credentials
immutability
audit logs
```

---

# 16. Backup Data Classification

Identify:

```text
critical
important
rebuildable
temporary
```

Do not waste backup storage treating every temporary artifact as critical data.

---

# 17. What Should Be Backed Up?

Potential targets:

```text
database
application configuration
infrastructure configuration
Terraform state
Kubernetes configuration where appropriate
certificates/keys under approved policy
critical uploaded data
```

---

# 18. What Should Not Be Backed Up Blindly?

Avoid automatically backing up:

```text
cache
temporary files
container writable layers
build directories
node_modules
Git working trees
large reproducible artifacts
```

unless there is a specific recovery requirement.

---

# 19. Backup Architecture

Example:

```text
Production
    |
    v
Python Backup Job
    |
    +--> Collect
    |
    +--> Validate
    |
    +--> Compress
    |
    +--> Checksum
    |
    +--> Encrypt
    |
    v
Object Storage
    |
    +--> Retention
    |
    +--> Immutability
    |
    v
Restore Environment
```

---

# 20. Python Backup Responsibilities

Python can handle:

```text
file discovery
backup manifests
compression
checksums
metadata
retention
verification
restore orchestration
```

---

# 21. Use the Right Tool

Python should not replace specialized backup systems when they already provide:

```text
database-consistent backups
incremental backups
deduplication
encryption
snapshot management
cross-region replication
```

Python is often the orchestration layer.

---

# 22. Backup Directory

Example:

```text
/backup/
├── daily/
├── weekly/
├── monthly/
├── manifests/
├── checksums/
└── restore/
```

---

# 23. Backup Naming

Use predictable names:

```text
myapp-20260817T053000Z.tar.gz
```

Include:

```text
application
timestamp
environment
version
```

where useful.

---

# 24. UTC Timestamps

Use UTC for backup identifiers.

```python
from datetime import (
    datetime,
    timezone,
)

timestamp = datetime.now(
    timezone.utc
).strftime(
    "%Y%m%dT%H%M%SZ"
)

print(timestamp)
```

---

# 25. Backup Manifest

A manifest records:

```text
backup ID
timestamp
source
files
size
checksum
version
environment
status
```

Example:

```json
{
  "backup_id": "myapp-20260817T053000Z",
  "environment": "production",
  "files": 1523,
  "bytes": 104857600,
  "status": "verified"
}
```

---

# 26. Why Manifests Matter

Without a manifest:

```text
backup exists
```

With a manifest:

```text
what
when
where
how large
what version
what checksum
```

can be established.

---

# 27. Create Backup Directory

```python
from pathlib import Path

backup_dir = Path(
    "/backup/myapp"
)

backup_dir.mkdir(
    parents=True,
    exist_ok=True,
)
```

---

# 28. Copy a Configuration File

```python
import shutil

shutil.copy2(
    "/etc/myapp/config.yaml",
    "/backup/myapp/config.yaml",
)
```

For production, add timestamps/versioning rather than overwriting the only previous copy.

---

# 29. Backup a Directory

```python
import shutil

shutil.copytree(
    source,
    destination,
)
```

For large production datasets, consider purpose-built backup/storage tools instead of a naive recursive copy.

---

# 30. Archive Backup

```python
import tarfile

with tarfile.open(
    "myapp.tar.gz",
    "w:gz",
) as archive:

    archive.add(
        "/opt/myapp",
        arcname="myapp",
    )
```

---

# 31. Why Compress Backups?

Compression can reduce:

```text
storage
network transfer
backup size
```

But it costs:

```text
CPU
time
```

---

# 32. Compression Trade-Off

For already compressed files:

```text
jpg
mp4
zip
gz
```

additional compression may provide little benefit.

---

# 33. Stream Large Backups

Do not load entire datasets into memory.

Prefer:

```text
stream
compress
upload
```

where supported.

---

# 34. Backup Checksum

Use SHA-256:

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

# 35. Verify Backup Checksum

```python
actual = sha256(
    backup_file
)

if actual != expected:
    raise RuntimeError(
        "Backup checksum mismatch"
    )
```

---

# 36. Why Verify Immediately?

A backup can fail because of:

```text
incomplete write
storage error
transfer corruption
unexpected truncation
```

Verification catches some of these problems early.

---

# 37. Backup Size Verification

Check:

```text
exists
non-zero
expected approximate size
checksum
manifest
```

A zero-byte backup should normally fail verification.

---

# 38. Backup Content Verification

Do not verify only:

```text
archive exists
```

Verify:

```text
archive opens
expected files exist
critical files exist
```

---

# 39. Archive Test

```python
import tarfile

with tarfile.open(
    backup_file,
    "r:gz",
) as archive:

    names = archive.getnames()

print(
    len(names)
)
```

---

# 40. Backup Manifest Generation

```python
import json

manifest = {
    "backup_id": backup_id,
    "environment": "production",
    "checksum": checksum,
    "status": "verified",
}

Path(
    "manifest.json"
).write_text(
    json.dumps(
        manifest,
        indent=2,
    ),
    encoding="utf-8",
)
```

---

# 41. Database Backups

Do not blindly copy database data files while the database is running.

Use database-native tools such as:

```text
pg_dump
pg_basebackup
mysqldump
MySQL physical backup tools
database-native snapshot mechanisms
```

depending on the database.

---

# 42. Python Database Backup Orchestration

Python can invoke:

```python
import subprocess

subprocess.run(
    [
        "pg_dump",
        "--format=custom",
        "mydb",
    ],
    check=True,
)
```

Credentials should come from secure mechanisms.

---

# 43. Database Backup Consistency

The backup must represent a consistent database state.

A filesystem copy of active database files may not be consistent.

---

# 44. PostgreSQL Example

Logical backup:

```bash
pg_dump mydb > mydb.sql
```

Python can orchestrate the command.

For large production databases, choose the database's appropriate backup strategy.

---

# 45. MySQL Example

Logical backup:

```bash
mysqldump mydb > mydb.sql
```

Again, production backup design should account for consistency, size, locking, and recovery requirements.

---

# 46. Database Restore

A database backup is useful only if restoration works.

Test:

```text
backup
 ↓
restore into isolated environment
 ↓
run validation queries
 ↓
application test
```

---

# 47. Configuration Backup

Back up:

```text
/etc/myapp
/etc/nginx
systemd units
application configuration
```

where required.

---

# 48. Terraform State Backup

Terraform state is critical.

Use:

```text
remote backend
state versioning
locking
access control
```

and follow the backend's backup/versioning capabilities.

Do not manually manipulate state files casually.

---

# 49. S3 Backend Considerations

For Terraform state stored in S3, use:

```text
versioning
restricted IAM
encryption
audit
```

and the backend's supported locking mechanism.

---

# 50. Kubernetes Configuration Backup

For GitOps-managed environments, Git is normally the source of truth.

Back up or preserve:

```text
Git repository
Helm values
manifests
sealed/encrypted secret representations
cluster-specific critical data
```

Do not treat raw Kubernetes Secrets in a repository as safe simply because they are YAML.

---

# 51. EKS Backup Strategy

An EKS recovery plan may include:

```text
infrastructure as code
Git repository
container images
database backups
persistent data backups
configuration
secrets recovery mechanism
```

The EKS control plane itself should not be treated as the only source of recoverability.

---

# 52. Persistent Volumes

Application data stored in:

```text
PersistentVolumes
```

requires a storage-specific backup strategy.

Use:

```text
volume snapshots
backup tools
object storage
database-native backup
```

as appropriate.

---

# 53. S3 Backup

For object storage, consider:

```text
versioning
lifecycle
replication
immutability
encryption
access controls
```

---

# 54. Python and S3

Python can use an AWS SDK such as `boto3` to:

```text
upload
download
list
verify metadata
```

Credentials should use:

```text
IAM roles
workload identity
credential providers
```

rather than hardcoded keys.

---

# 55. S3 Upload Example

```python
import boto3

s3 = boto3.client(
    "s3"
)

s3.upload_file(
    "backup.tar.gz",
    "my-backup-bucket",
    "myapp/backup.tar.gz",
)
```

Production code should verify the upload and handle retries appropriately.

---

# 56. Backup Encryption

Encryption can occur:

```text
at rest
in transit
before upload
```

Prefer platform-supported encryption where appropriate.

---

# 57. Client-Side Encryption

Python can encrypt a backup before upload.

But key management becomes critical:

```text
key generation
key storage
key rotation
key recovery
access control
```

Never store the encryption key next to the backup.

---

# 58. Encryption at Rest

Object storage commonly supports:

```text
server-side encryption
```

Use approved organizational keys/policies.

---

# 59. Encryption in Transit

Use:

```text
HTTPS
TLS
secure transport
```

for backup transfer.

---

# 60. Backup Credentials

Never:

```python
ACCESS_KEY = "..."
SECRET_KEY = "..."
```

in source code.

Prefer:

```text
IAM role
instance profile
workload identity
secret manager
credential provider chain
```

---

# 61. Least Privilege

Backup identity should have only required permissions.

Example:

```text
write backup
read backup for verification
list required prefix
```

Do not automatically grant:

```text
full account admin
```

---

# 62. Backup Isolation

Backups should ideally be protected from the same failure domain as production.

Examples:

```text
different account
different region
separate storage
immutable vault
offsite object storage
```

---

# 63. Ransomware Protection

Controls can include:

```text
immutability
separate credentials
MFA
restricted delete
cross-account storage
offline copies
monitoring
```

---

# 64. Retention Automation

Retention algorithm:

```text
list backups
 ↓
classify by age
 ↓
protect required copies
 ↓
delete eligible copies
 ↓
verify
```

---

# 65. Never Delete All Old Backups Blindly

Always protect:

```text
minimum recovery points
weekly/monthly copies
legal retention copies
known-good restore point
```

according to policy.

---

# 66. Backup Retention Example

```python
from datetime import (
    datetime,
    timezone,
    timedelta,
)

cutoff = (
    datetime.now(
        timezone.utc
    )
    - timedelta(days=30)
)
```

Compare backup timestamps against the retention period.

---

# 67. Dry-Run Retention

Before deletion:

```text
would delete:
backup-001
backup-002

would retain:
backup-003
backup-004
```

Production cleanup should support dry-run.

---

# 68. Retention Safety Limits

Example:

```python
if len(candidates) > 100:
    raise RuntimeError(
        "Unexpected deletion count"
    )
```

Also consider total bytes.

---

# 69. Backup Locking

Prevent:

```text
two backup jobs
```

from writing to the same target simultaneously.

Use:

```text
scheduler concurrency control
distributed lock
job orchestration
```

as appropriate.

---

# 70. Backup Job States

Useful state model:

```text
STARTED
COLLECTING
COMPRESSING
UPLOADING
VERIFYING
COMPLETED
FAILED
```

---

# 71. Backup Logging

Log:

```text
backup ID
start time
end time
source
size
destination
checksum
status
error
```

Do not log secrets.

---

# 72. Backup Metrics

Useful metrics:

```text
backup_success_total
backup_failure_total
backup_duration_seconds
backup_bytes_total
backup_age_seconds
restore_success_total
restore_failure_total
```

---

# 73. Backup Alerting

Alert on:

```text
backup failed
backup missing
backup too old
backup size abnormal
checksum mismatch
upload failed
restore test failed
```

---

# 74. Backup Freshness

A useful check:

```text
last successful backup
        ↓
current time
        ↓
backup age
```

Example:

```text
Expected: < 1 hour
Actual: 4 hours
Status: CRITICAL
```

---

# 75. Backup SLA

Define:

```text
backup frequency
RPO
retention
verification
restore test frequency
```

---

# 76. Backup Monitoring

Example:

```text
Prometheus
   ↓
backup exporter/script
   ↓
metrics
   ↓
Grafana
   ↓
alert
```

---

# 77. Backup and ELK

Backup job logs can be sent to centralized logging:

```text
Python backup
 ↓
structured logs
 ↓
ELK
 ↓
search
 ↓
incident investigation
```

---

# 78. Backup and CI/CD

CI/CD artifacts can be backed up:

```text
release package
configuration
deployment metadata
checksums
```

But artifact retention should be separate from disaster-recovery backup requirements.

---

# 79. Backup Job Exit Codes

```text
0 → success
1 → backup failure
2 → verification failure
3 → invalid configuration
```

Define your own documented exit-code contract.

---

# 80. Verification Must Fail the Job

If:

```text
backup created
checksum mismatch
```

the job should report:

```text
FAILURE
```

not:

```text
SUCCESS
```

---

# 81. Backup Pipeline

```text
preflight
 ↓
collect
 ↓
package
 ↓
checksum
 ↓
upload
 ↓
verify remote copy
 ↓
manifest
 ↓
retention
 ↓
report
```

---

# 82. Preflight Checks

Before backup:

```text
disk space
source exists
destination reachable
credentials available
permissions
backup target available
```

---

# 83. Destination Preflight

Check:

```text
bucket exists
network works
credentials valid
encryption policy available
```

Do not test with destructive operations.

---

# 84. Backup Failure Categories

```text
source failure
permission failure
storage failure
network failure
compression failure
checksum failure
authentication failure
retention failure
restore failure
```

---

# 85. Retry Strategy

Retry transient failures:

```text
network timeout
temporary storage error
service unavailable
```

Use:

```text
bounded retries
exponential backoff
jitter
```

---

# 86. Do Not Retry Everything

Do not endlessly retry:

```text
permission denied
invalid path
invalid credentials
corrupt source
invalid configuration
```

These require remediation.

---

# 87. Backup Partial Failure

Suppose:

```text
10 sources
8 succeed
2 fail
```

Do not automatically mark:

```text
backup complete
```

unless the backup policy explicitly defines partial success.

---

# 88. Backup Manifest Status

Example:

```json
{
  "status": "partial_failure",
  "successful_sources": 8,
  "failed_sources": 2
}
```

This makes recovery decisions explicit.

---

# 89. Backup Consistency

For a multi-file application, files may change during backup.

Possible solutions:

```text
application quiescing
snapshot
database-native backup
consistent export
filesystem snapshot
```

---

# 90. Avoid Inconsistent File Copies

Example:

```text
config.yaml
database.db
metadata.json
```

If these are changing independently, copying each at different times may create an inconsistent backup.

---

# 91. Application Quiescing

For some applications:

```text
pause writes
 ↓
backup
 ↓
resume
```

This may affect availability, so use only when appropriate.

---

# 92. Snapshot-Based Backup

```text
application/storage
      ↓
snapshot
      ↓
backup snapshot
```

Snapshots can provide a consistent point-in-time view when supported by the storage system.

---

# 93. Database-Native Backup

For databases, prefer:

```text
database-native consistency
```

rather than treating database files as ordinary files.

---

# 94. Restore Environment

A good restore process uses:

```text
isolated environment
```

such as:

```text
temporary EC2
test Kubernetes namespace
restore VM
test database
```

---

# 95. Restore Test

Example:

```text
download backup
 ↓
verify checksum
 ↓
extract
 ↓
restore database
 ↓
start application
 ↓
run smoke test
```

---

# 96. Restore Validation

Check:

```text
files exist
database opens
application starts
critical API works
data is readable
permissions correct
```

---

# 97. Restore Testing Frequency

Example:

```text
critical backups → monthly restore test
less critical    → quarterly
```

Actual frequency depends on business requirements.

---

# 98. Restore Drill

A disaster-recovery drill should answer:

```text
Can we restore?
How long does it take?
What dependencies are missing?
Are credentials available?
Is documentation correct?
```

---

# 99. Restore Documentation

Document:

```text
backup location
credentials
restore commands
dependencies
validation queries
health checks
rollback
contacts
```

---

# 100. Restore Automation

Python can automate:

```text
download
verify
extract
restore
test
report
```

---

# 101. Backup Inventory

Maintain:

```text
backup ID
timestamp
size
checksum
storage location
retention date
status
```

---

# 102. Backup Catalog

A catalog can be:

```text
JSON
CSV
database
object metadata
```

For larger systems, use a proper backup catalog rather than a single local file.

---

# 103. Backup Metadata

Example:

```json
{
  "backup_id": "prod-db-20260817",
  "type": "logical",
  "size": 4294967296,
  "checksum": "abc...",
  "storage": "s3://backup/prod/",
  "verified": true
}
```

---

# 104. Backup Naming Convention

Good:

```text
prod-orders-db-full-20260817T053000Z.dump
```

Avoid ambiguous:

```text
backup-final-new2.tar.gz
```

---

# 105. Backup Versioning

Include:

```text
application version
database schema version
configuration version
```

when relevant.

---

# 106. Backup Dependency Mapping

A backup may depend on:

```text
application version
database schema
encryption keys
secret references
network configuration
DNS
IAM
```

A recovery plan must account for dependencies.

---

# 107. Backup and Encryption Keys

If backups are encrypted:

```text
backup
+
key
=
recoverable data
```

Losing the key can make the backup useless.

---

# 108. Key Recovery

Ensure approved recovery of:

```text
encryption keys
KMS permissions
secret-manager access
```

during disaster recovery.

---

# 109. Backup Account Separation

For critical backups:

```text
production account
       ↓
backup account
```

This can reduce blast radius.

---

# 110. Cross-Region Backup

For regional disaster recovery:

```text
Region A
   ↓
backup
   ↓
Region B
```

Use service-native replication where possible.

---

# 111. Cross-Region Considerations

Check:

```text
RPO
transfer cost
latency
encryption
restore time
regional dependencies
```

---

# 112. Backup Cost Management

Storage costs can be controlled using:

```text
retention
compression
deduplication
storage tiers
lifecycle rules
```

Do not sacrifice required recovery points just to reduce cost.

---

# 113. Backup Lifecycle

Example:

```text
hot
 ↓
warm
 ↓
cold/archive
 ↓
expire
```

Object storage lifecycle policies can automate transitions.

---

# 114. Python Retention vs Storage Lifecycle

Prefer storage-native lifecycle policies for simple object expiration when available.

Use Python for:

```text
complex policy
cross-system coordination
validation
reporting
```

---

# 115. Backup Integrity

Integrity checks can include:

```text
checksum
archive test
file count
size
manifest
restore test
```

---

# 116. Checksum Is Not Restore Testing

Checksum proves:

```text
bytes match
```

It does not prove:

```text
application can restore
```

Both are useful.

---

# 117. Backup Encryption Is Not Access Control

Encryption protects data, but you still need:

```text
IAM
bucket policies
network controls
audit
MFA
```

---

# 118. Backup Storage Permissions

Follow least privilege:

```text
backup writer
backup reader
restore operator
admin
```

Separate roles where practical.

---

# 119. Backup Audit Logs

Track:

```text
backup created
backup deleted
backup restored
permission changed
retention executed
```

---

# 120. Backup Deletion Protection

For critical backups:

```text
immutability
retention lock
restricted delete
approval
```

---

# 121. Backup Cleanup

Do not use:

```python
shutil.rmtree(
    backup_directory
)
```

without strict validation.

---

# 122. Safe Backup Cleanup

Workflow:

```text
discover backups
 ↓
classify
 ↓
protect required points
 ↓
dry run
 ↓
delete eligible
 ↓
verify retention
```

---

# 123. Backup Cleanup Script

```python
from pathlib import Path
import time

root = Path(
    "/backup/myapp"
)

cutoff = (
    time.time()
    - 30 * 86400
)

for backup in root.glob(
    "*.tar.gz"
):

    if backup.stat().st_mtime < cutoff:
        print(
            "Eligible:",
            backup,
        )
```

Add policy checks before deletion.

---

# 124. Backup Dry Run

Example:

```text
Retention policy: 30 days

Would delete:
prod-001.tar.gz
prod-002.tar.gz

Would retain:
prod-003.tar.gz
prod-004.tar.gz
```

---

# 125. Backup Count Guard

```python
if len(candidates) > 50:
    raise RuntimeError(
        "Unexpected backup count"
    )
```

Unexpected deletion counts should stop automation.

---

# 126. Backup Size Guard

Also calculate:

```text
total deletion bytes
```

and stop if unexpectedly large.

---

# 127. Backup Job Lock

Avoid:

```text
backup job A
backup job B
```

writing the same backup simultaneously.

---

# 128. Scheduled Backup

Cron example:

```cron
0 * * * * /usr/bin/python3 /opt/backup/backup.py
```

For modern production systems, consider:

```text
systemd timers
Jenkins
Kubernetes CronJob
cloud-native schedulers
```

---

# 129. Kubernetes CronJob Backup

Architecture:

```text
CronJob
 ↓
Python container
 ↓
backup
 ↓
object storage
 ↓
verification
 ↓
exit
```

The container should not depend on local container storage for the final backup.

---

# 130. Kubernetes CronJob Concurrency

Configure appropriate concurrency behavior so a slow backup does not unintentionally overlap with the next scheduled run.

---

# 131. Backup Job Resource Limits

For Kubernetes backup jobs, define:

```text
CPU
memory
ephemeral storage
```

because compression can consume significant resources.

---

# 132. Backup Logs

Structured output:

```json
{
  "backup_id": "prod-001",
  "status": "success",
  "duration_seconds": 124,
  "bytes": 1073741824
}
```

Useful for centralized logging.

---

# 133. Backup Monitoring with Prometheus

Expose:

```text
last_success_timestamp
last_failure_timestamp
backup_age
backup_duration
backup_size
```

---

# 134. Backup Alert Example

Conceptually:

```text
backup_age > RPO
```

should trigger an alert.

---

# 135. Backup Dashboard

Useful Grafana panels:

```text
last successful backup
backup age
backup duration
backup size
failure count
restore test status
```

---

# 136. Backup Incident

**Problem:**

```text
No successful backup for 6 hours
RPO = 1 hour
```

Response:

```text
identify failed jobs
 ↓
inspect logs
 ↓
check storage
 ↓
check credentials
 ↓
check source
 ↓
run recovery backup
 ↓
verify
 ↓
alert stakeholders
```

---

# 137. Backup Incident — Storage Full

Investigate:

```text
backup storage
retention
unexpected backup growth
duplicate backups
compression
```

Do not simply delete the oldest recovery points without understanding policy.

---

# 138. Backup Incident — Upload Failure

Check:

```text
network
credentials
bucket permissions
storage service
object size
timeouts
```

Retry only when failure is transient.

---

# 139. Backup Incident — Checksum Mismatch

Response:

```text
mark backup invalid
 ↓
do not delete known-good backup
 ↓
retry from source
 ↓
verify
 ↓
investigate corruption
```

---

# 140. Backup Incident — Restore Failure

Response:

```text
stop restore
 ↓
preserve failed evidence
 ↓
try another known-good backup
 ↓
identify dependency
 ↓
fix restore procedure
 ↓
repeat test
```

---

# 141. Backup Incident — Wrong Retention Deletion

Response:

```text
stop cleanup
 ↓
identify deleted backups
 ↓
restore if recoverable
 ↓
review policy
 ↓
add protection rules
 ↓
test cleanup
```

---

# 142. Backup Incident — Credentials Expired

Check:

```text
IAM role
secret
token
service account
permissions
```

Prefer short-lived workload credentials over static credentials.

---

# 143. Backup Incident — Encryption Key Unavailable

Recovery depends on:

```text
KMS/key availability
IAM permissions
key policy
region
secret recovery
```

A backup strategy must test this scenario.

---

# 144. Backup Incident — Backup Exists but Is Too Old

Example:

```text
RPO = 1 hour
last backup = 8 hours
```

Treat this as an incident even though a backup exists.

---

# 145. Backup Incident — Backup Job Succeeds but Data Missing

This is why:

```text
manifest
content verification
restore testing
```

matter.

---

# 146. Backup Incident — Database Restore Works but Application Fails

Investigate:

```text
schema version
application version
configuration
secrets
network
dependencies
```

A database backup alone may not represent the complete application recovery state.

---

# 147. Backup Incident — Region Failure

Recovery plan:

```text
backup in another region
 ↓
infrastructure provisioning
 ↓
restore data
 ↓
restore configuration
 ↓
restore secrets access
 ↓
deploy application
 ↓
DNS
 ↓
health checks
```

---

# 148. Backup Incident — Entire Production Account Lost

A serious DR plan should consider:

```text
separate backup account
separate credentials
IaC repository
container registry recovery
cross-account backup
DNS recovery
secret recovery
```

---

# 149. Backup Security Checklist

```text
[ ] Encryption at rest
[ ] Encryption in transit
[ ] Least privilege
[ ] Separate backup credentials
[ ] Secret management
[ ] Immutability where required
[ ] Audit logging
[ ] Restricted deletion
[ ] Cross-account protection
[ ] Cross-region copy where required
```

---

# 150. Backup Reliability Checklist

```text
[ ] RPO defined
[ ] RTO defined
[ ] Backup schedule
[ ] Retention
[ ] Integrity verification
[ ] Manifest
[ ] Restore test
[ ] Monitoring
[ ] Alerting
[ ] Recovery documentation
```

---

# 151. Backup Script Architecture

```text
backup.py
├── config
├── collector
├── packager
├── checksum
├── uploader
├── verifier
├── retention
├── reporter
└── restore
```

---

# 152. Collector

Responsibilities:

```text
identify sources
validate paths
collect metadata
```

---

# 153. Packager

Responsibilities:

```text
copy
archive
compress
```

---

# 154. Checksum Module

Responsibilities:

```text
hash
compare
verify
```

---

# 155. Uploader

Responsibilities:

```text
upload
retry
verify destination
```

---

# 156. Verifier

Responsibilities:

```text
archive readable
expected files
checksum
manifest
```

---

# 157. Retention Module

Responsibilities:

```text
list
classify
protect
delete
verify
```

---

# 158. Restore Module

Responsibilities:

```text
download
verify
extract
restore
health check
```

---

# 159. Reporter

Output:

```text
backup ID
status
duration
size
checksum
destination
restore status
```

---

# 160. Example Backup CLI

```bash
python backup.py run \
    --environment production
```

---

# 161. Dry-Run CLI

```bash
python backup.py retention \
    --days 30 \
    --dry-run
```

---

# 162. Verify CLI

```bash
python backup.py verify \
    --backup prod-20260817T053000Z
```

---

# 163. Restore CLI

```bash
python backup.py restore \
    --backup prod-20260817T053000Z \
    --target /restore/test
```

Only restore to approved targets.

---

# 164. Status CLI

```bash
python backup.py status
```

Output:

```text
Last success: 2026-08-17 05:30 UTC
Age: 42 minutes
Status: HEALTHY
```

---

# 165. Backup Configuration

Example:

```yaml
backup:
  source: /opt/myapp
  retention_days: 30
  compression: gzip
  verify: true
  destination: s3://my-backups/myapp
```

Never put secret credentials directly into this file.

---

# 166. Backup Policy Validation

```python
if retention_days < 7:
    raise ValueError(
        "Retention below policy"
    )
```

---

# 167. Backup Configuration Security

Validate:

```text
destination approved
retention approved
encryption enabled
verification enabled
```

---

# 168. Backup Preflight

```python
source.exists()
destination_available()
credentials_available()
disk_space_available()
```

All should pass before a large backup begins.

---

# 169. Backup Space Calculation

Before creating a local archive:

```text
source size
+
compression workspace
+
temporary files
```

must fit available storage.

---

# 170. Avoid Local Disk Exhaustion

For large backups:

```text
stream
direct upload
snapshot
```

may be better than:

```text
create huge local archive
then upload
```

---

# 171. Backup Through Streaming

Conceptually:

```text
source
 ↓
compress
 ↓
encrypt
 ↓
upload
```

This minimizes temporary disk usage.

---

# 172. Backup Performance

Measure:

```text
files/sec
MB/sec
compression CPU
network throughput
duration
```

---

# 173. Backup Bottlenecks

Possible bottlenecks:

```text
disk read
compression CPU
network
object storage
database dump
encryption
```

Measure before optimizing.

---

# 174. Backup Parallelism

Parallelism can help independent sources, but excessive parallelism can overload:

```text
disk
database
network
storage API
```

Use bounded concurrency.

---

# 175. Database Backup Scheduling

Avoid running heavy backups during:

```text
peak traffic
maintenance
large deployment
batch processing
```

unless architecture supports it.

---

# 176. Backup and Application Performance

A backup can affect:

```text
CPU
disk I/O
network
database load
```

Monitor production impact.

---

# 177. Backup Window

Define:

```text
start
maximum duration
expected completion
```

If a backup exceeds the window, investigate.

---

# 178. Backup SLA Violation

Example:

```text
Expected completion: 06:00
Actual: 08:30
```

Even if backup eventually succeeds, the SLA may be violated.

---

# 179. Backup Testing Pyramid

```text
checksum
   ↓
archive verification
   ↓
sample restore
   ↓
full restore drill
   ↓
disaster recovery exercise
```

Use all levels according to criticality.

---

# 180. Restore Test Automation

A Python workflow can:

```text
select latest backup
 ↓
download
 ↓
verify checksum
 ↓
restore isolated environment
 ↓
run smoke tests
 ↓
destroy test environment
 ↓
report
```

---

# 181. Restore Test Result

```json
{
  "backup": "prod-001",
  "checksum": "PASS",
  "restore": "PASS",
  "application_test": "PASS",
  "duration_seconds": 812
}
```

---

# 182. Backup Documentation

Document:

```text
what is backed up
where it is stored
how often
retention
RPO
RTO
restore procedure
ownership
contacts
```

---

# 183. Backup Runbook

Example:

```text
1. Identify latest verified backup
2. Verify checksum
3. Restore infrastructure
4. Restore configuration
5. Restore secrets access
6. Restore database
7. Deploy application
8. Validate
9. Update DNS
10. Monitor
```

---

# 184. Backup Dependency Order

A recovery may require:

```text
network
 ↓
IAM
 ↓
storage
 ↓
database
 ↓
configuration
 ↓
application
 ↓
DNS
```

Actual order depends on architecture.

---

# 185. Backup and Infrastructure as Code

Use:

```text
Terraform
```

to rebuild:

```text
VPC
EKS
EC2
IAM
S3
RDS
ALB
```

Then restore data/configuration.

---

# 186. Backup and Git

Git provides:

```text
configuration history
code history
manifest history
```

It does not automatically replace backups for:

```text
database data
persistent user data
secrets
runtime state
```

---

# 187. Backup and ECR

Container images may need:

```text
retention
replication
rebuild strategy
```

For critical DR, ensure required images remain available or can be rebuilt.

---

# 188. Backup and RDS

For RDS, use AWS-supported:

```text
automated backups
snapshots
point-in-time recovery
cross-region options
```

Python can orchestrate checks and reporting rather than replacing AWS's native mechanisms.

---

# 189. Backup and S3

For S3 data:

```text
versioning
replication
lifecycle
immutability
```

may provide stronger recovery than repeatedly downloading and re-uploading objects with Python.

---

# 190. Backup and ALB

ALB itself is infrastructure.

Its configuration should generally be recoverable through:

```text
Terraform
CloudFormation
other IaC
```

rather than file backups.

---

# 191. Backup and Secrets

Do not simply archive plaintext secrets.

Prefer:

```text
secret manager
key management
recovery procedure
```

---

# 192. Backup and Monitoring

Backup monitoring should detect:

```text
missing backup
failed backup
old backup
unexpected size
restore failure
storage failure
```

---

# 193. Backup and Incident Management

A backup alert should create an actionable incident when:

```text
RPO is violated
critical backup failed
restore test failed
backup storage unavailable
```

---

# 194. Backup Interview Question

## What is RPO?

**Answer:**

> RPO is Recovery Point Objective. It defines the maximum acceptable amount of data loss measured in time. For example, a one-hour RPO means the recovery strategy should normally allow recovery to within approximately the last hour.

---

# 195. Interview Question — What Is RTO?

**Answer:**

> RTO is Recovery Time Objective. It defines the maximum acceptable time to restore the service after an incident.

---

# 196. Interview Question — Backup vs Snapshot?

**Answer:**

> A snapshot is generally a point-in-time representation of storage, while a backup is an independent recovery copy. Snapshots are useful for fast recovery but should not automatically be considered a complete disaster-recovery strategy.

---

# 197. Interview Question — Backup vs Replication?

**Answer:**

> Replication improves availability by maintaining another copy, but it can replicate corruption or accidental deletion. Backups provide historical recovery points.

---

# 198. Interview Question — How Do You Verify a Backup?

**Answer:**

> I verify that the backup exists, has expected size/content, passes checksum validation, has a valid manifest, and can be opened or restored. For critical systems I perform regular restore tests.

---

# 199. Interview Question — Why Is Restore Testing Important?

**Answer:**

> Because backup creation alone does not prove recoverability. Restore testing validates that the backup, encryption keys, credentials, dependencies, and recovery procedure actually work.

---

# 200. Interview Question — How Do You Secure Backups?

**Answer:**

> I use encryption, least-privilege access, separate credentials, restricted deletion, audit logging, immutability where required, and offsite or cross-account copies for critical systems.

---

# 201. Interview Question — How Do You Automate Retention?

**Answer:**

> I classify backups by age and policy, protect the required recovery points, support dry-run, apply deletion limits, delete only eligible backups, and verify the remaining recovery set.

---

# 202. Interview Question — How Do You Handle Backup Credentials?

**Answer:**

> I avoid static credentials in code. In AWS I prefer IAM roles or the SDK credential provider chain, with narrowly scoped permissions.

---

# 203. Interview Question — What If Backup Storage Is Full?

**Answer:**

> I inspect retention, unexpected growth, duplicate backups, and lifecycle configuration. I do not blindly delete old recovery points because retention requirements may be violated.

---

# 204. Interview Question — How Do You Backup a Database?

**Answer:**

> I use the database's native consistent backup mechanism or managed-service backup capability. Python can orchestrate the process, verify results, upload artifacts, and test restoration.

---

# 205. Interview Question — Why Not Copy Database Files Directly?

**Answer:**

> Active database files can be internally inconsistent while the database is running. Database-native backup tools understand transaction consistency and recovery requirements.

---

# 206. Interview Question — How Do You Handle Backup Corruption?

**Answer:**

> I mark the backup invalid, preserve known-good recovery points, retry if the cause is transient, verify the replacement, and investigate why corruption occurred.

---

# 207. Interview Question — How Do You Design a Backup System?

**Answer:**

> I start with RPO/RTO and data classification, then define backup frequency, retention, storage isolation, encryption, immutability, verification, restore testing, monitoring, and disaster-recovery procedures.

---

# 208. Interview Question — What Should Be Backed Up in an EKS Environment?

**Answer:**

> I would protect the infrastructure definition, GitOps configuration, container image availability, persistent application data, databases, required secrets recovery mechanisms, and other business-critical state. Kubernetes manifests alone are not enough if the application data lives elsewhere.

---

# 209. Interview Question — How Do You Backup Terraform?

**Answer:**

> I use a secure remote backend with state versioning and appropriate locking/access controls. I avoid manually copying and editing live state because Terraform state is a critical infrastructure artifact.

---

# 210. Interview Question — How Do You Handle Backup Retention?

**Answer:**

> I implement policy-based retention rather than deleting based only on age. I protect required daily, weekly, monthly, and compliance recovery points and use storage-native lifecycle policies where appropriate.

---

# 211. Interview Question — How Do You Monitor Backups?

**Answer:**

> I monitor last successful backup, backup age, duration, size, failures, checksum verification, and restore-test results. I alert when the backup age exceeds the RPO or when verification fails.

---

# 212. Interview Question — What Is a 3-2-1 Strategy?

**Answer:**

> It means maintaining three copies of data, using two different storage/media types, with one copy offsite. Modern implementations may add immutable or offline protection.

---

# 213. Interview Question — What Happens if the Whole AWS Region Fails?

**Answer:**

> The recovery plan needs infrastructure reproducibility through IaC, backups in another region or account, required container images, database recovery, configuration, secrets access, and DNS/application recovery procedures.

---

# 214. Interview Question — What If the Backup Encryption Key Is Lost?

**Answer:**

> The encrypted backup may become unrecoverable. Therefore key management and disaster recovery must include key availability, permissions, rotation strategy, and recovery testing.

---

# 215. Interview Question — How Do You Prevent Ransomware from Deleting Backups?

**Answer:**

> I use isolated backup accounts, least-privilege credentials, immutability or retention lock where supported, restricted deletion, MFA, audit logging, and independent recovery credentials.

---

# 216. Interview Question — What Is the Difference Between Backup Verification and Restore Testing?

**Answer:**

> Verification checks properties such as checksum and archive integrity. Restore testing proves that the data can actually be recovered and used by the application.

---

# 217. Interview Question — How Do You Avoid Backup Impact on Production?

**Answer:**

> I schedule within an appropriate window, monitor CPU/I/O/network impact, use snapshots or native backup mechanisms when possible, control concurrency, and measure backup performance.

---

# 218. Interview Question — How Do You Handle a Partial Backup?

**Answer:**

> I explicitly report partial failure in the manifest, identify missing sources, retry where appropriate, and do not mark the backup fully successful unless the policy permits partial recovery.

---

# 219. Interview Question — How Do You Make Backup Automation Idempotent?

**Answer:**

> I generate unique backup IDs, detect existing verified backups, avoid overwriting recovery points, and make retries safe. Uploads and retention operations should be designed so repeated execution does not destroy valid recovery data.

---

# 220. Interview Question — How Do You Secure a Backup Script?

**Answer:**

> I validate paths, use least-privilege credentials, avoid shell injection, protect secrets, encrypt data, restrict destinations, use secure temporary files, log actions without sensitive values, and test failure paths.

---

# 221. Real-World Project — Python Backup Manager

Build:

```text
backup_manager/
├── cli.py
├── config.py
├── collector.py
├── packager.py
├── checksum.py
├── uploader.py
├── verifier.py
├── retention.py
├── restore.py
├── reporter.py
└── tests/
```

---

# 222. Backup Manager Commands

```bash
python backup.py run
```

```bash
python backup.py verify
```

```bash
python backup.py list
```

```bash
python backup.py retention --dry-run
```

```bash
python backup.py restore --backup BACKUP_ID
```

---

# 223. Backup Manager Workflow

```text
CLI
 ↓
Preflight
 ↓
Collect
 ↓
Package
 ↓
Checksum
 ↓
Upload
 ↓
Remote Verification
 ↓
Manifest
 ↓
Retention
 ↓
Report
```

---

# 224. Real-World Project — EKS Backup Validation

Build a tool that reports:

```text
latest database backup
latest persistent-volume backup
latest configuration commit
required image availability
secret recovery status
backup age
```

Output:

```text
EKS DR Readiness
----------------
Database: PASS
Configuration: PASS
Images: PASS
Secrets: PASS
Backup Age: PASS

Overall: READY
```

---

# 225. Real-World Project — Restore Drill

Automate:

```text
select latest verified backup
 ↓
provision isolated environment
 ↓
restore
 ↓
run tests
 ↓
measure RTO
 ↓
destroy test environment
 ↓
report
```

---

# 226. Real-World Project — Backup Monitoring

Python exporter reports:

```text
backup_age_seconds
backup_success
backup_size_bytes
backup_duration_seconds
restore_test_success
```

Prometheus collects metrics.

Grafana visualizes them.

---

# 227. Real-World Project — Backup Audit

Generate a report:

```text
Backup ID
Source
Timestamp
Size
Checksum
Storage
Retention
Verification
Restore Test
```

Export:

```text
JSON
CSV
```

---

# 228. Production Backup Workflow

```text
1. Define RPO/RTO
2. Identify critical data
3. Select backup mechanism
4. Define retention
5. Secure destination
6. Automate
7. Verify
8. Monitor
9. Test restore
10. Conduct DR drills
```

---

# 229. Final Backup Security Checklist

```text
[ ] Encryption
[ ] Least privilege
[ ] Separate credentials
[ ] Immutable copy
[ ] Offsite copy
[ ] Cross-account protection
[ ] Cross-region copy where required
[ ] Audit logging
[ ] Restricted deletion
[ ] Key recovery
```

---

# 230. Final Backup Reliability Checklist

```text
[ ] RPO defined
[ ] RTO defined
[ ] Schedule defined
[ ] Retention defined
[ ] Manifest generated
[ ] Checksum verified
[ ] Content verified
[ ] Restore tested
[ ] Monitoring configured
[ ] Alerting configured
[ ] Recovery runbook documented
```

---

# 231. Final Takeaway

A production backup system is not:

```text
copy
```

It is:

```text
protect
 ↓
verify
 ↓
retain
 ↓
monitor
 ↓
restore
 ↓
prove recovery
```

Python is valuable because it can connect these steps into an automated workflow.

The strongest DevOps engineer does not say:

> "We have backups."

They can explain:

```text
What is backed up?
Where is it stored?
How often?
What is the RPO?
What is the RTO?
How is it encrypted?
Who can delete it?
How is integrity verified?
When was the last restore test?
How long did recovery take?
What happens if the primary region fails?
```

That is the difference between **having backup files** and having a **real recovery strategy**.
