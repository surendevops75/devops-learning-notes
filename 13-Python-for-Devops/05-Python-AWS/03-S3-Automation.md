# S3-Automation

## Python for AWS DevOps — S3 Buckets, Objects, Backups, Security, Lifecycle, Auditing & Production Automation

Amazon S3 is one of the most commonly automated AWS services in DevOps.

Typical Python/Boto3 use cases include:

```text
bucket inventory
object inventory
backup automation
artifact upload
log archival
cross-region copying
versioning checks
encryption audits
public-access audits
lifecycle audits
old-object cleanup
deployment artifact management
report generation
```

The production goal is not simply:

```python
s3.upload_file(...)
```

It is:

```text
authenticate
   ↓
validate account/region
   ↓
identify bucket
   ↓
validate policy
   ↓
perform operation
   ↓
verify
   ↓
log
   ↓
report
```

---

# 1. S3 Mental Model

```text
AWS Account
    |
    +-- Bucket
          |
          +-- Prefix
                |
                +-- Object
```

An S3 object contains:

```text
key
data
metadata
tags
version
storage class
encryption information
```

---

# 2. Boto3 S3 Client

```python
import boto3

s3 = boto3.client("s3")
```

For an explicit region:

```python
s3 = boto3.client(
    "s3",
    region_name="ap-south-1",
)
```

---

# 3. S3 Resource

```python
import boto3

s3 = boto3.resource("s3")
```

Example:

```python
bucket = s3.Bucket(
    "my-example-bucket"
)
```

For detailed API control, the S3 client is commonly preferred in production automation.

---

# 4. List Buckets

```python
response = s3.list_buckets()

for bucket in response.get(
    "Buckets",
    []
):
    print(
        bucket.get("Name")
    )
```

Unlike many regional APIs, bucket listing is account-scoped.

---

# 5. Get Bucket Location

```python
response = s3.get_bucket_location(
    Bucket=bucket_name
)

print(
    response.get("LocationConstraint")
)
```

Be aware that the API has special behavior for the `us-east-1` region.

---

# 6. Bucket Name Validation

Bucket names are globally significant.

Before creating or operating on a bucket, validate:

```text
name
account
expected environment
region
```

Never assume a bucket name belongs to the account you intended.

---

# 7. Create an S3 Bucket

For regions other than the historical default behavior:

```python
s3.create_bucket(
    Bucket=bucket_name,
    CreateBucketConfiguration={
        "LocationConstraint": "ap-south-1"
    },
)
```

Use current AWS API behavior and region requirements for production code.

---

# 8. Bucket Creation Is Not the End

After creation, configure according to policy:

```text
versioning
encryption
public access block
ownership controls
lifecycle
logging/auditing
tags
bucket policy
```

---

# 9. Bucket Versioning

Check versioning:

```python
response = s3.get_bucket_versioning(
    Bucket=bucket_name
)

print(
    response.get("Status")
)
```

Typical values include:

```text
Enabled
Suspended
```

---

# 10. Enable Versioning

```python
s3.put_bucket_versioning(
    Bucket=bucket_name,
    VersioningConfiguration={
        "Status": "Enabled"
    },
)
```

Versioning is important for:

```text
accidental deletion recovery
object overwrite protection
backup workflows
audit requirements
```

---

# 11. Versioning Caveat

Versioning does not mean:

```text
infinite free backup
```

Old versions consume storage.

Combine versioning with an appropriate lifecycle policy.

---

# 12. S3 Encryption

Check default bucket encryption:

```python
response = s3.get_bucket_encryption(
    Bucket=bucket_name
)
```

Use appropriate exception handling when encryption configuration is absent.

---

# 13. Server-Side Encryption

Common approaches include:

```text
SSE-S3
SSE-KMS
```

Your organization's security requirements determine which is appropriate.

---

# 14. SSE-KMS

SSE-KMS provides integration with AWS KMS for encryption-key management.

A KMS-encrypted upload can use:

```python
s3.upload_file(
    filename,
    bucket_name,
    object_key,
    ExtraArgs={
        "ServerSideEncryption": "aws:kms",
        "SSEKMSKeyId": kms_key_id,
    },
)
```

Use the correct KMS permissions.

---

# 15. KMS Permissions

A workload may require permissions such as:

```text
kms:Encrypt
kms:Decrypt
kms:GenerateDataKey
```

depending on the operation and key policy.

Do not grant broad KMS access unnecessarily.

---

# 16. Public Access Block

A common production baseline is to block unintended public access.

Check:

```python
response = s3.get_public_access_block(
    Bucket=bucket_name
)

print(response)
```

---

# 17. Public Access Block Configuration

Example:

```python
s3.put_public_access_block(
    Bucket=bucket_name,
    PublicAccessBlockConfiguration={
        "BlockPublicAcls": True,
        "IgnorePublicAcls": True,
        "BlockPublicPolicy": True,
        "RestrictPublicBuckets": True,
    },
)
```

Only relax these controls when a documented architecture requires public access.

---

# 18. Bucket Ownership Controls

Modern S3 architectures commonly use bucket-owner-enforced object ownership.

Check:

```python
response = s3.get_bucket_ownership_controls(
    Bucket=bucket_name
)
```

Use the current AWS S3 ownership model appropriate to your environment.

---

# 19. S3 Bucket Tags

Tags can identify:

```text
Environment
Owner
Project
CostCenter
ManagedBy
DataClassification
```

Example:

```python
s3.put_bucket_tagging(
    Bucket=bucket_name,
    Tagging={
        "TagSet": [
            {
                "Key": "Environment",
                "Value": "dev",
            },
            {
                "Key": "ManagedBy",
                "Value": "PythonAutomation",
            },
        ]
    },
)
```

---

# 20. List Objects

For small examples:

```python
response = s3.list_objects_v2(
    Bucket=bucket_name
)

for obj in response.get(
    "Contents",
    []
):
    print(
        obj.get("Key")
    )
```

This response is paginated.

---

# 21. S3 Object Pagination

Use a paginator:

```python
paginator = s3.get_paginator(
    "list_objects_v2"
)

for page in paginator.paginate(
    Bucket=bucket_name
):

    for obj in page.get(
        "Contents",
        []
    ):
        print(
            obj.get("Key")
        )
```

This is the production pattern for large buckets.

---

# 22. Prefix Filtering

```python
for page in paginator.paginate(
    Bucket=bucket_name,
    Prefix="backups/"
):

    for obj in page.get(
        "Contents",
        []
    ):
        print(
            obj["Key"]
        )
```

Server-side prefix filtering reduces unnecessary processing.

---

# 23. Delimiter

S3 does not have real folders.

It uses object keys such as:

```text
logs/2026/08/app.log
```

A delimiter can help present hierarchical-looking results:

```python
Delimiter="/"
```

---

# 24. S3 Prefixes

Common organization:

```text
application/
    releases/
    logs/
    backups/
    reports/
```

Use consistent key conventions.

---

# 25. Object Metadata

Useful fields:

```text
Key
LastModified
ETag
Size
StorageClass
```

Example:

```python
for obj in page.get(
    "Contents",
    []
):
    print(
        obj.get("Key"),
        obj.get("Size"),
        obj.get("StorageClass"),
        obj.get("LastModified"),
    )
```

---

# 26. Upload a File

```python
s3.upload_file(
    "build.zip",
    bucket_name,
    "releases/build.zip",
)
```

This is one of the most common Boto3 DevOps operations.

---

# 27. Upload With Extra Arguments

```python
s3.upload_file(
    "build.zip",
    bucket_name,
    "releases/build.zip",
    ExtraArgs={
        "ServerSideEncryption": "AES256"
    },
)
```

For KMS:

```python
ExtraArgs={
    "ServerSideEncryption": "aws:kms",
    "SSEKMSKeyId": kms_key_id,
}
```

---

# 28. Upload Text Data

```python
s3.put_object(
    Bucket=bucket_name,
    Key="reports/status.txt",
    Body="Deployment successful\n",
)
```

---

# 29. Upload JSON

```python
import json

data = {
    "status": "success",
    "environment": "dev",
}

s3.put_object(
    Bucket=bucket_name,
    Key="reports/status.json",
    Body=json.dumps(data),
    ContentType="application/json",
)
```

---

# 30. Download a File

```python
s3.download_file(
    bucket_name,
    "releases/build.zip",
    "build.zip",
)
```

---

# 31. Download Object to Memory

```python
response = s3.get_object(
    Bucket=bucket_name,
    Key="reports/status.json",
)

body = response["Body"].read()

print(body.decode("utf-8"))
```

Do not load very large objects into memory unnecessarily.

---

# 32. Stream Large Objects

For large files, prefer streaming or transfer-manager functionality rather than:

```python
body = response["Body"].read()
```

for the entire object.

---

# 33. Upload Large Files

Boto3's managed S3 transfer operations can automatically use multipart upload when appropriate.

```python
s3.upload_file(
    "large-backup.tar.gz",
    bucket_name,
    "backups/large-backup.tar.gz",
)
```

For large artifacts, configure transfer behavior according to your network and workload.

---

# 34. Multipart Upload

Conceptually:

```text
large file
   ↓
split into parts
   ↓
upload parts
   ↓
complete multipart upload
```

Benefits:

```text
parallel transfer
reliability
large-object support
```

---

# 35. Upload Progress

For CI/CD artifact uploads, progress callbacks can provide operational visibility.

This is useful for:

```text
large artifacts
backups
release packages
```

---

# 36. Copy an Object

```python
s3.copy_object(
    Bucket=destination_bucket,
    Key=destination_key,
    CopySource={
        "Bucket": source_bucket,
        "Key": source_key,
    },
)
```

---

# 37. Copy Across Buckets

```text
source bucket
     ↓
S3 copy
     ↓
destination bucket
```

Permissions must exist for both sides.

---

# 38. Copy Across Regions

A cross-region copy can be used for:

```text
backup
disaster recovery
artifact replication
migration
```

For large-scale replication, evaluate native S3 replication features instead of building everything manually.

---

# 39. Delete an Object

```python
s3.delete_object(
    Bucket=bucket_name,
    Key=object_key,
)
```

This is potentially destructive.

---

# 40. Delete Multiple Objects

```python
s3.delete_objects(
    Bucket=bucket_name,
    Delete={
        "Objects": [
            {
                "Key": "old/file1.txt"
            },
            {
                "Key": "old/file2.txt"
            },
        ]
    },
)
```

Batch deletion reduces API calls.

---

# 41. Delete Safety

Before deleting:

```text
bucket
↓
environment
↓
prefix
↓
object age
↓
versioning
↓
retention policy
↓
backup requirement
```

Never implement:

```python
delete_everything()
```

without strict policy controls.

---

# 42. S3 Versioned Delete

When versioning is enabled, deleting an object can create a delete marker rather than permanently removing every version.

Understand the versioning behavior before implementing cleanup.

---

# 43. List Object Versions

```python
paginator = s3.get_paginator(
    "list_object_versions"
)

for page in paginator.paginate(
    Bucket=bucket_name
):

    for version in page.get(
        "Versions",
        []
    ):
        print(
            version.get("Key"),
            version.get("VersionId"),
            version.get("IsLatest"),
        )
```

---

# 44. Delete a Specific Version

```python
s3.delete_object(
    Bucket=bucket_name,
    Key=object_key,
    VersionId=version_id,
)
```

This is highly destructive.

Use only under a defined retention policy.

---

# 45. S3 Lifecycle

Lifecycle policies automate:

```text
transition
expiration
noncurrent-version cleanup
incomplete multipart cleanup
```

This is usually preferable to custom Python cleanup for recurring lifecycle rules.

---

# 46. Lifecycle Example

Concept:

```text
Day 0
 ↓
S3 Standard

Day 30
 ↓
lower-cost storage class

Day 90
 ↓
archive

Day 365
 ↓
expire
```

Exact policies depend on access patterns and compliance requirements.

---

# 47. Python vs S3 Lifecycle

Use Python when:

```text
custom business logic
complex reporting
approval workflow
cross-service decisions
```

Use native lifecycle when:

```text
simple age-based transitions
expiration
noncurrent-version retention
```

Prefer AWS-native lifecycle for straightforward storage lifecycle management.

---

# 48. Storage Classes

Common classes include:

```text
S3 Standard
S3 Intelligent-Tiering
S3 Standard-IA
S3 One Zone-IA
S3 Glacier Instant Retrieval
S3 Glacier Flexible Retrieval
S3 Glacier Deep Archive
```

Choose based on:

```text
access frequency
retrieval requirements
retention
cost
availability requirements
```

---

# 49. Storage Class Inventory

```python
storage_classes = {}

for page in paginator.paginate(
    Bucket=bucket_name
):
    for obj in page.get(
        "Contents",
        []
    ):
        storage_class = obj.get(
            "StorageClass",
            "STANDARD"
        )

        storage_classes[
            storage_class
        ] = storage_classes.get(
            storage_class,
            0
        ) + 1
```

---

# 50. S3 Size Report

```python
total_bytes = 0

for page in paginator.paginate(
    Bucket=bucket_name
):

    for obj in page.get(
        "Contents",
        []
    ):
        total_bytes += obj.get(
            "Size",
            0
        )

print(
    total_bytes
)
```

For large-scale analytics, consider S3 Inventory or other native reporting capabilities rather than scanning every object repeatedly.

---

# 51. Human-Readable Size

```python
def to_mb(size):
    return size / (
        1024 * 1024
    )
```

For production reports, use clear units such as:

```text
MiB
GiB
TiB
```

---

# 52. Find Large Objects

```python
threshold = 1024 * 1024 * 1024

for page in paginator.paginate(
    Bucket=bucket_name
):

    for obj in page.get(
        "Contents",
        []
    ):

        if obj.get(
            "Size",
            0
        ) > threshold:

            print(
                obj["Key"]
            )
```

---

# 53. Find Old Objects

Concept:

```python
from datetime import datetime, timezone

now = datetime.now(
    timezone.utc
)
```

Compare:

```text
now - LastModified
```

against a defined retention threshold.

---

# 54. Do Not Confuse LastModified With Business Retention

An object's age does not automatically mean:

```text
safe to delete
```

Business requirements may require retention for:

```text
audit
legal
backup
rollback
compliance
```

---

# 55. S3 Object Tags

Object tags can store metadata.

Example:

```python
s3.put_object_tagging(
    Bucket=bucket_name,
    Key=object_key,
    Tagging={
        "TagSet": [
            {
                "Key": "Environment",
                "Value": "dev",
            }
        ]
    },
)
```

---

# 56. Get Object Tags

```python
response = s3.get_object_tagging(
    Bucket=bucket_name,
    Key=object_key,
)

for tag in response.get(
    "TagSet",
    []
):
    print(
        tag["Key"],
        tag["Value"]
    )
```

---

# 57. S3 Object Metadata vs Tags

Metadata:

```text
attached to object
```

Tags:

```text
key/value classification
```

Use tags when the data needs to be queried or classified according to supported S3 workflows.

---

# 58. S3 Bucket Policy

Bucket policies control resource-based access.

Python can retrieve:

```python
response = s3.get_bucket_policy(
    Bucket=bucket_name
)
```

Handle the case where a policy does not exist.

---

# 59. Public Bucket Audit

A production audit can inspect:

```text
public access block
bucket policy
ACL configuration
ownership controls
```

Do not assume a single API tells the entire public-access story.

---

# 60. ACL Awareness

Modern S3 architectures often rely on bucket policies and bucket-owner-enforced object ownership instead of ACL-based access control.

When auditing older environments, still understand ACLs.

---

# 61. Get Bucket ACL

```python
response = s3.get_bucket_acl(
    Bucket=bucket_name
)
```

Use this as one input to a broader access audit.

---

# 62. Encryption Audit

For every bucket, check:

```text
default encryption
encryption type
KMS key where applicable
```

Generate:

```text
PASS
WARN
FAIL
```

according to policy.

---

# 63. Versioning Audit

Report:

```text
bucket
versioning status
environment
data classification
```

Example:

```text
prod-backups → Enabled → PASS
dev-temp     → Suspended → REVIEW
```

Do not impose the same policy on every bucket without considering data requirements.

---

# 64. Public Access Audit

Example output:

```text
Bucket                  Public Access
--------------------------------------
prod-artifacts          BLOCKED
prod-logs               BLOCKED
public-assets           REVIEW
```

A bucket intentionally serving public content may have an approved exception.

---

# 65. Bucket Inventory Project

Build:

```bash
python s3ops.py inventory
```

Collect:

```text
bucket
region
versioning
encryption
public access block
tags
object count
size
```

---

# 66. S3 Inventory Architecture

```text
CLI / Scheduler
      ↓
Boto3 Session
      ↓
STS account validation
      ↓
List buckets
      ↓
For each bucket
      ↓
Security/configuration checks
      ↓
Object metrics
      ↓
Report
```

---

# 67. Bucket Inventory Data Model

```python
{
    "bucket": "...",
    "region": "...",
    "versioning": "...",
    "encryption": "...",
    "public_access_block": True,
    "object_count": 123,
    "size_bytes": 456789
}
```

Keep report structures separate from raw AWS responses.

---

# 68. S3 Backup Automation

A simple backup workflow:

```text
source
 ↓
identify files
 ↓
upload to S3
 ↓
encryption
 ↓
versioning/lifecycle
 ↓
verify
 ↓
report
```

---

# 69. Backup Key Design

Use deterministic keys:

```text
backups/
  application/
    2026/
      08/
        17/
```

Or use:

```text
backup timestamp
application
environment
```

according to your recovery strategy.

---

# 70. Backup Naming

Example:

```text
backups/roboshop/prod/
2026-08-17T10-30-00/
database.dump
```

Avoid ambiguous filenames such as:

```text
backup-final-final2.zip
```

---

# 71. Backup Encryption

Use:

```text
SSE-S3
```

or:

```text
SSE-KMS
```

according to organizational requirements.

For sensitive backups, understand KMS key policies and recovery dependencies.

---

# 72. Backup Verification

Uploading successfully does not guarantee your backup strategy works.

Verify:

```text
object exists
size expected
encryption expected
metadata/tag expected
restore procedure
```

---

# 73. Restore Testing

A backup is useful only if it can be restored.

Production backup practice:

```text
backup
 ↓
restore test
 ↓
validate
 ↓
document
```

Do not wait for a disaster to discover that the restore process is broken.

---

# 74. S3 Artifact Repository

CI/CD can store:

```text
build artifacts
Helm packages
configuration bundles
reports
release archives
```

Example:

```text
artifacts/
  application/
    release-1.2.3/
      build.zip
```

---

# 75. CI/CD Artifact Upload

```python
s3.upload_file(
    artifact_path,
    bucket_name,
    artifact_key,
)
```

Recommended controls:

```text
encryption
restricted bucket policy
versioning where useful
retention
checksum/integrity validation
```

---

# 76. Deployment Artifact Verification

Before deployment:

```text
artifact exists
 ↓
download
 ↓
verify checksum
 ↓
deploy
```

Do not blindly deploy any object that happens to exist in the bucket.

---

# 77. S3 + Jenkins

Architecture:

```text
Jenkins
 ↓
build
 ↓
test
 ↓
package
 ↓
Boto3
 ↓
S3 artifact bucket
 ↓
deployment
```

---

# 78. S3 + GitHub Actions

```text
GitHub Actions
 ↓
OIDC
 ↓
IAM Role
 ↓
Python/Boto3
 ↓
S3
```

No long-lived AWS access key needs to be stored when OIDC is configured correctly.

---

# 79. S3 + EKS

EKS workloads may use S3 for:

```text
configuration
artifacts
reports
backups
data
```

Use an AWS-supported pod identity mechanism and least-privilege IAM permissions.

---

# 80. S3 + Lambda

A common event-driven architecture:

```text
S3 object uploaded
       ↓
S3 event
       ↓
Lambda
       ↓
Python processing
```

Use event-driven architecture instead of polling when it fits the use case.

---

# 81. S3 Event Automation

Example:

```text
upload *.json
 ↓
event
 ↓
Lambda
 ↓
validate/process
```

Do not create an infinite event loop where Lambda writes back to the same triggering prefix unintentionally.

---

# 82. S3 + EventBridge

S3 events can participate in event-driven workflows using AWS event services.

Example:

```text
S3
 ↓
EventBridge
 ↓
Lambda / Step Functions / notification
```

Use event-driven designs for workflows that should react to changes.

---

# 83. S3 + ELK

S3 can act as a storage layer for logs and archived data.

```text
application
 ↓
logging pipeline
 ↓
S3 archive
```

Python can automate:

```text
archive validation
inventory
retention reports
```

---

# 84. S3 + Prometheus/Grafana

Prometheus should not scan every S3 object directly.

Instead, expose metrics from your automation:

```text
s3_buckets_total
s3_objects_processed_total
s3_audit_failures_total
s3_backup_success_total
s3_backup_failure_total
```

Grafana visualizes the automation health.

---

# 85. S3 + Terraform

Terraform can own:

```text
bucket
bucket policy
encryption
versioning
lifecycle
public access block
```

Python/Boto3 can perform:

```text
inventory
operational reports
custom workflows
audits
```

Avoid uncontrolled drift.

---

# 86. S3 + Ansible

Ansible can orchestrate workflows while Python/Boto3 can perform custom S3 logic.

Example:

```text
Ansible
 ↓
invoke Python
 ↓
Boto3
 ↓
S3
```

Keep responsibility clear.

---

# 87. S3 + AWS CLI

Boto3 is not always required.

For simple operations:

```bash
aws s3 cp file.txt s3://bucket/
```

Python becomes useful when you need:

```text
conditional logic
complex filtering
cross-service integration
reports
testing
reusable automation
```

---

# 88. Presigned URL

Boto3 can generate temporary URLs for object access.

```python
url = s3.generate_presigned_url(
    "get_object",
    Params={
        "Bucket": bucket_name,
        "Key": object_key,
    },
    ExpiresIn=900,
)
```

This can provide time-limited access without making the bucket public.

---

# 89. Presigned URL Security

Treat the URL as a credential during its validity period.

Do not:

```text
log it publicly
post it in tickets
embed it permanently
```

Use a suitable short expiration.

---

# 90. S3 Head Object

Check object metadata without downloading the object:

```python
response = s3.head_object(
    Bucket=bucket_name,
    Key=object_key,
)

print(
    response.get("ContentLength")
)
```

Useful for:

```text
existence checks
size validation
metadata inspection
```

---

# 91. Object Existence Check

```python
from botocore.exceptions import ClientError

try:
    s3.head_object(
        Bucket=bucket_name,
        Key=object_key,
    )

    exists = True

except ClientError:
    exists = False
```

In production, classify the error rather than treating every `ClientError` as "not found."

---

# 92. S3 Copy Verification

After a copy:

```text
destination object exists
 ↓
size matches
 ↓
metadata/encryption matches policy
```

For important workflows, validate checksums or other integrity information appropriate to the transfer.

---

# 93. Object Checksums

S3 supports checksum mechanisms for object integrity.

For critical artifact workflows, use an appropriate checksum strategy instead of relying only on object size.

---

# 94. ETag Is Not Always a Simple MD5

Do not blindly interpret:

```text
ETag == MD5
```

for every object.

Multipart uploads and encryption can make ETag semantics different.

Use explicit checksum mechanisms when integrity verification matters.

---

# 95. S3 Request Retry

Configure retries:

```python
from botocore.config import Config

config = Config(
    retries={
        "max_attempts": 5,
        "mode": "standard",
    }
)

s3 = boto3.client(
    "s3",
    config=config,
)
```

Use appropriate timeout settings for long transfers.

---

# 96. S3 Transfer Configuration

For large transfers, Boto3 transfer configuration can control:

```text
multipart threshold
chunk size
concurrency
```

Tune based on:

```text
file size
network
CPU
memory
AWS limits
```

---

# 97. Controlled Concurrency

Do not upload hundreds of files with unlimited threads.

Use bounded concurrency:

```text
worker count
 ↓
API pressure
 ↓
network pressure
```

Monitor throughput and failures.

---

# 98. S3 API Throttling

Possible causes:

```text
high request volume
massive object scan
unbounded concurrency
frequent polling
```

Solutions:

```text
pagination
batching
native S3 features
bounded concurrency
retry/backoff
```

---

# 99. Prefer S3 Inventory for Large Audits

If a bucket contains millions or billions of objects, repeatedly scanning with `list_objects_v2` may be inefficient.

For large-scale inventory, consider native S3 Inventory and analyze the generated inventory data.

---

# 100. S3 Inventory Architecture

```text
S3
 ↓
S3 Inventory
 ↓
inventory files
 ↓
Python
 ↓
analysis/report
```

This is often better than making a full object-list API scan every day.

---

# 101. S3 Batch Operations

For very large object sets, evaluate S3 Batch Operations.

Use Python to:

```text
generate/prepare job
monitor job
report results
```

rather than individually issuing millions of API calls when a native batch workflow fits.

---

# 102. S3 Lifecycle vs Python Cleanup

### Native lifecycle

Best for:

```text
age-based expiration
storage transitions
noncurrent version expiration
```

### Python

Best for:

```text
custom eligibility
business rules
approval
cross-service validation
reporting
```

---

# 103. S3 Delete Safety Policy

A production cleanup script might require:

```text
Environment
Owner
Prefix
Age
Protected
Retention
Backup status
```

before deletion.

---

# 104. Protected Objects

Object or bucket policy can include:

```text
Protected=true
```

The automation should skip protected resources.

---

# 105. Dry-Run Cleanup

```bash
python s3ops.py cleanup \
    --prefix backups/dev/ \
    --dry-run
```

Output:

```text
Would delete:
backups/dev/old-001.zip
backups/dev/old-002.zip
```

No deletion occurs.

---

# 106. S3 Backup Cleanup

A safe workflow:

```text
identify candidates
 ↓
check age
 ↓
check retention
 ↓
check protected tag/policy
 ↓
check backup requirement
 ↓
dry-run
 ↓
approval
 ↓
delete
 ↓
verify
```

---

# 107. S3 Bucket Deletion

A bucket must be empty before deletion in common S3 workflows.

A versioned bucket can be especially complex because:

```text
objects
delete markers
object versions
```

must be handled correctly.

---

# 108. Never Empty Production Buckets Casually

Avoid scripts that recursively delete a bucket without:

```text
account validation
bucket allowlist
environment validation
dry-run
approval
backup confirmation
```

---

# 109. S3 Bucket Deletion Project

If you build a cleanup tool:

```text
discover bucket
 ↓
validate allowlist
 ↓
check environment
 ↓
check protection
 ↓
list versions
 ↓
dry-run
 ↓
approval
 ↓
delete
 ↓
verify
```

---

# 110. S3 Security Audit Project

Check:

```text
public access block
bucket policy
ownership controls
encryption
versioning
logging/auditing requirements
```

Output:

```text
PASS
WARN
FAIL
```

---

# 111. S3 Compliance Reporter

Example:

```text
Bucket: prod-artifacts

Encryption: PASS
Versioning: PASS
Public Access: PASS
Ownership: PASS
Tags: PASS
Lifecycle: WARN
```

---

# 112. Multi-Region S3 Reporting

Because S3 buckets have regional placement, build a report that includes:

```text
bucket
region
environment
encryption
versioning
```

Use the bucket location API.

---

# 113. Multi-Account S3 Reporting

Architecture:

```text
Management account
      ↓
AssumeRole
      ↓
Dev account
      ↓
Staging account
      ↓
Production account
      ↓
S3 inventory
```

Validate the account identity at every target.

---

# 114. S3 Cross-Account Access

Cross-account access can involve:

```text
IAM identity policy
bucket policy
KMS key policy
SCP
permissions boundary
```

For KMS-encrypted objects, both S3 and KMS authorization must be considered.

---

# 115. S3 Replication

For disaster recovery:

```text
source bucket
      ↓
replication
      ↓
destination region/account
```

Native S3 replication is generally preferable to custom Python loops for continuous replication.

Python can audit replication configuration and report status.

---

# 116. Replication Audit

Check:

```text
replication configured
destination
rules
status
priority/filter
```

Use native replication metrics/status where available.

---

# 117. S3 Backup vs Replication

Replication:

```text
availability/disaster recovery
```

Backup:

```text
recovery from deletion/corruption
```

Replication alone may replicate unwanted deletions or changes depending on configuration.

Design both according to recovery objectives.

---

# 118. RPO and RTO

For S3 recovery planning:

```text
RPO = how much data loss is acceptable
RTO = how quickly recovery must happen
```

Automation should support the organization's RPO/RTO requirements.

---

# 119. S3 Logging and Auditing

For sensitive buckets, integrate with AWS-native audit services and your logging platform.

Python can summarize:

```text
configuration
compliance
operation results
```

Do not build a custom access-log system when AWS-native auditing already provides the required capability.

---

# 120. S3 Automation Logging

Log:

```text
operation
bucket
prefix
object count
result
duration
error code
```

Avoid logging:

```text
secret object contents
presigned URLs
credentials
sensitive data
```

---

# 121. S3 Automation Metrics

Useful metrics:

```text
s3_automation_runs_total
s3_automation_failures_total
s3_objects_processed_total
s3_bytes_processed_total
s3_upload_failures_total
s3_cleanup_candidates_total
```

---

# 122. S3 Backup Automation Example

```python
import boto3
from pathlib import Path

s3 = boto3.client("s3")

def upload_backup(
    file_path,
    bucket,
    key,
):

    s3.upload_file(
        str(file_path),
        bucket,
        key,
        ExtraArgs={
            "ServerSideEncryption": "AES256"
        },
    )

backup = Path(
    "/backup/application.tar.gz"
)

upload_backup(
    backup,
    "example-backup-bucket",
    "backups/application.tar.gz",
)
```

Production code should add:

```text
account validation
configuration
logging
verification
retry handling
```

---

# 123. S3 Artifact Upload Function

```python
def upload_artifact(
    s3,
    file_path,
    bucket,
    key,
):

    s3.upload_file(
        file_path,
        bucket,
        key,
        ExtraArgs={
            "ServerSideEncryption": "AES256"
        },
    )
```

---

# 124. S3 Download Function

```python
def download_object(
    s3,
    bucket,
    key,
    destination,
):

    s3.download_file(
        bucket,
        key,
        destination,
    )
```

---

# 125. S3 Object Listing Function

```python
def list_objects(
    s3,
    bucket,
    prefix="",
):

    paginator = s3.get_paginator(
        "list_objects_v2"
    )

    objects = []

    for page in paginator.paginate(
        Bucket=bucket,
        Prefix=prefix,
    ):

        objects.extend(
            page.get(
                "Contents",
                []
            )
        )

    return objects
```

---

# 126. S3 Bucket Configuration Function

A reusable audit function can collect:

```text
region
versioning
encryption
public access block
ownership
tags
```

Keep each API call independently error-aware because a configuration may be absent.

---

# 127. S3 CLI Design

Example:

```bash
python s3ops.py buckets
```

```bash
python s3ops.py inventory
```

```bash
python s3ops.py audit
```

```bash
python s3ops.py upload
```

```bash
python s3ops.py backup
```

```bash
python s3ops.py cleanup --dry-run
```

---

# 128. S3 CLI Safety

Useful arguments:

```text
--profile
--region
--bucket
--prefix
--environment
--dry-run
--output
```

For destructive operations:

```text
--confirm
```

should not be the only safety mechanism; account and resource policy validation should still happen.

---

# 129. S3 Environment Configuration

Example:

```yaml
environment: dev

bucket_policy:
  required_versioning: true
  required_encryption: true
  block_public_access: true

cleanup:
  min_age_days: 90
```

Validate this configuration before execution.

---

# 130. S3 Automation Testing

Test:

```text
bucket discovery
object listing
filter logic
tag selection
dry-run
error handling
cleanup eligibility
```

Use mocks/stubs for unit tests.

---

# 131. S3 Stub Testing

Use:

```python
from botocore.stub import Stubber

stubber = Stubber(s3)
```

Then provide deterministic API responses.

This keeps unit tests independent of real buckets.

---

# 132. S3 Integration Testing

Use a dedicated test account/bucket.

Test:

```text
upload
head
download
copy
tagging
versioning
cleanup
```

Clean up test resources afterward.

---

# 133. S3 Test Bucket Naming

Use a unique test bucket naming strategy.

Concept:

```text
python-automation-test-<unique-id>
```

Avoid accidentally colliding with existing global bucket names.

---

# 134. S3 Test Isolation

Use:

```text
test prefix
test bucket
test account
test tags
```

Do not let a unit test discover and modify unrelated production objects.

---

# 135. S3 Error Handling

```python
from botocore.exceptions import ClientError

try:
    s3.head_object(
        Bucket=bucket_name,
        Key=key,
    )

except ClientError as exc:

    code = exc.response.get(
        "Error",
        {}
    ).get("Code")

    print(
        f"S3 operation failed: {code}"
    )
    raise
```

Classify errors appropriately.

---

# 136. Common S3 Errors

Examples include:

```text
NoSuchBucket
NoSuchKey
AccessDenied
InvalidBucketName
BucketAlreadyExists
BucketAlreadyOwnedByYou
SlowDown
```

The exact error behavior depends on the API operation.

---

# 137. S3 SlowDown

`SlowDown` indicates request pressure.

Possible improvements:

```text
reduce concurrency
use backoff
avoid repeated full scans
use native inventory
batch work
```

---

# 138. AccessDenied Troubleshooting

Check:

```text
IAM policy
bucket policy
KMS key policy
SCP
permissions boundary
object ownership
requested action
resource ARN
```

---

# 139. NoSuchKey

Possible reasons:

```text
wrong key
wrong prefix
wrong bucket
wrong account
wrong region
object deleted
versioning behavior
```

Use `head_object` and verify context.

---

# 140. BucketAlreadyExists

S3 bucket names have global uniqueness requirements.

A bucket name existing does not mean:

```text
your account owns it
```

Do not attempt to operate on it until ownership is established.

---

# 141. S3 Security Checklist

```text
[ ] No hardcoded credentials
[ ] Least privilege
[ ] Encryption
[ ] Public access controls
[ ] Ownership controls
[ ] Versioning where required
[ ] Bucket policy review
[ ] KMS policy review
[ ] Sensitive data handling
[ ] No secrets in logs
```

---

# 142. S3 Reliability Checklist

```text
[ ] Pagination
[ ] Multipart transfers
[ ] Retries
[ ] Backoff
[ ] Timeouts
[ ] Transfer configuration
[ ] Verification
[ ] Checksum strategy
[ ] Partial failure handling
[ ] Restore testing
```

---

# 143. S3 Cost Checklist

```text
[ ] Storage class policy
[ ] Lifecycle
[ ] Noncurrent version retention
[ ] Incomplete multipart cleanup
[ ] Old object audit
[ ] Large object analysis
[ ] Native S3 Inventory for large-scale audits
[ ] Replication cost awareness
```

---

# 144. S3 Production Automation Checklist

```text
[ ] Account validated
[ ] Bucket validated
[ ] Region validated
[ ] Environment validated
[ ] Prefix validated
[ ] Protected resources handled
[ ] Dry-run
[ ] Approval for destructive changes
[ ] Verification
[ ] Logging
[ ] Metrics
[ ] Notification
```

---

# 145. Interview — What Is S3?

**Answer:**

> Amazon S3 is AWS's object storage service. It stores objects in buckets and is commonly used for artifacts, backups, logs, reports and application data.

---

# 146. Interview — How Do You Upload a File Using Boto3?

**Answer:**

> I can use `upload_file` on an S3 client. For production, I also consider encryption, retry behavior, transfer configuration, object key conventions and post-upload verification.

---

# 147. Interview — How Do You List All Objects in a Bucket?

**Answer:**

> I use `list_objects_v2` through a Boto3 paginator and process every page. I avoid assuming a single API response contains all objects.

---

# 148. Interview — Why Is Pagination Important in S3?

**Answer:**

> Buckets can contain very large numbers of objects. S3 returns object listings in pages, so using a paginator ensures the automation processes the complete result set.

---

# 149. Interview — How Do You Find Objects Under a Prefix?

**Answer:**

> I use the `Prefix` parameter with `list_objects_v2`, preferably through a paginator. This allows S3 to return only the relevant key namespace.

---

# 150. Interview — How Do You Download an S3 Object?

**Answer:**

> For a file I can use `download_file`. For smaller objects that need to be processed in memory, I can use `get_object`. For large objects, I avoid loading the entire object into memory.

---

# 151. Interview — How Do You Upload Large Files?

**Answer:**

> I use Boto3's managed S3 transfer functionality, which can use multipart uploads. I tune transfer configuration and concurrency according to file size and workload.

---

# 152. Interview — What Is S3 Versioning?

**Answer:**

> Versioning maintains multiple versions of an object and helps recover from accidental overwrites or deletions. It also increases storage usage, so lifecycle policies for noncurrent versions should be considered.

---

# 153. Interview — Does S3 Versioning Equal Backup?

**Answer:**

> Not necessarily. Versioning protects against some accidental changes, but a complete backup strategy also considers retention, replication, recovery testing and application-level consistency.

---

# 154. Interview — How Do You Secure an S3 Bucket?

**Answer:**

> I use least-privilege IAM and bucket policies, block unintended public access, use appropriate encryption, ownership controls and versioning where required, and continuously audit the configuration.

---

# 155. Interview — SSE-S3 vs SSE-KMS?

**Answer:**

> Both provide server-side encryption. SSE-KMS integrates with AWS KMS for additional key-management and authorization capabilities. The choice depends on security, compliance and key-management requirements.

---

# 156. Interview — How Do You Prevent Public S3 Access?

**Answer:**

> I enable appropriate S3 Block Public Access settings, use bucket-owner-enforced ownership where appropriate, review bucket policies and ACLs in legacy environments, and explicitly document approved public-use cases.

---

# 157. Interview — How Do You Automate S3 Cleanup?

**Answer:**

> First I identify candidates using explicit prefixes, tags, age and retention policy. I run a dry-run report, exclude protected data, obtain approval when required, then delete and verify. For simple age-based lifecycle, I prefer native S3 Lifecycle rules.

---

# 158. Interview — Python Cleanup or S3 Lifecycle?

**Answer:**

> I use S3 Lifecycle for straightforward age-based transitions and expiration. I use Python when custom business logic, reporting, cross-service validation or approval is required.

---

# 159. Interview — How Do You Handle Millions of Objects?

**Answer:**

> I avoid repeatedly scanning the entire bucket with list APIs if a native capability is more appropriate. I consider S3 Inventory, S3 Batch Operations and lifecycle policies depending on the use case.

---

# 160. Interview — What Is S3 Inventory?

**Answer:**

> S3 Inventory provides scheduled object inventory reports. It is useful for large-scale auditing and analytics without repeatedly listing every object through API calls.

---

# 161. Interview — How Do You Handle S3 Throttling?

**Answer:**

> I use controlled concurrency, pagination, batching, retries with backoff and native S3 capabilities where possible. I avoid unnecessarily scanning huge buckets repeatedly.

---

# 162. Interview — How Do You Verify an S3 Upload?

**Answer:**

> I can use `head_object` to verify existence and metadata, compare expected size, and use an appropriate checksum strategy for important artifacts.

---

# 163. Interview — Is S3 ETag Always an MD5?

**Answer:**

> No. ETag semantics can differ for multipart uploads and other scenarios. For important integrity checks I prefer explicit S3 checksum mechanisms rather than assuming the ETag is an MD5 hash.

---

# 164. Interview — How Do You Generate a Temporary S3 Download URL?

**Answer:**

> I can use `generate_presigned_url` to create a time-limited URL for a specific object. I treat the URL as sensitive during its validity period and use an appropriate short expiration.

---

# 165. Interview — How Do You Back Up Files to S3?

**Answer:**

> I upload files using deterministic object keys, enable appropriate encryption, use versioning/lifecycle where required, verify the backup and regularly test restoration.

---

# 166. Interview — How Do You Automate Cross-Region S3 Backup?

**Answer:**

> For continuous replication I prefer native S3 replication capabilities. Python/Boto3 can audit or orchestrate custom backup workflows when additional business logic is required.

---

# 167. Interview — S3 Replication vs Backup?

**Answer:**

> Replication improves availability and disaster recovery, while backup focuses on recoverability from deletion, corruption or other data-loss scenarios. Replication alone may not satisfy all backup requirements.

---

# 168. Interview — How Do You Integrate S3 With CI/CD?

**Answer:**

> The pipeline builds and tests an artifact, authenticates to AWS using a secure mechanism such as OIDC or an IAM role, uploads the artifact to a controlled S3 bucket, verifies it, and passes the artifact reference to the deployment stage.

---

# 169. Interview — How Do You Integrate S3 With EKS?

**Answer:**

> An EKS workload can access S3 through an AWS-supported pod identity mechanism and a least-privilege IAM role. I avoid static AWS keys inside Kubernetes secrets when workload identity is available.

---

# 170. Interview — How Do You Handle S3 AccessDenied?

**Answer:**

> I identify the caller, verify the requested bucket/key and action, then inspect IAM policies, bucket policies, KMS permissions, SCPs, permissions boundaries and ownership controls.

---

# 171. Interview — How Do You Avoid Deleting the Wrong S3 Data?

**Answer:**

> I validate account, bucket, environment, prefix and retention policy, exclude protected resources, use dry-run, and require approval for high-risk operations. I never rely on a broad bucket-wide delete command.

---

# 172. Interview — What Would You Automate First With S3?

**Answer:**

> I would begin with read-only bucket inventory and security/compliance reporting. Then I would add artifact and backup workflows, followed by controlled cleanup where the retention policy is well defined.

---

# 173. Interview — How Do You Test S3 Automation?

**Answer:**

> I use mocked or stubbed Boto3 responses for unit tests and an isolated test bucket/account for integration tests. Test data is tagged and cleanup is automated.

---

# 174. Interview — How Do You Handle Terraform-Managed S3 Buckets?

**Answer:**

> Terraform should remain the source of truth for infrastructure configuration such as bucket policies, encryption, versioning and lifecycle. Boto3 can handle operational reporting and workflows while avoiding unintended infrastructure drift.

---

# 175. Interview — What Is Your Production S3 Automation Pattern?

**Answer:**

> I authenticate using temporary credentials, validate account and bucket, apply least-privilege permissions, use paginators and controlled concurrency, configure retries and timeouts, protect destructive operations with dry-run and approval, verify the final state, and publish logs, metrics and notifications.

---

# 176. Final S3 Automation Mental Model

```text
Authenticate
     ↓
Validate Account
     ↓
Validate Bucket
     ↓
Validate Region
     ↓
Validate Environment
     ↓
Apply Least Privilege
     ↓
Discover
     ↓
Filter
     ↓
Process
     ↓
Retry/Backoff
     ↓
Verify
     ↓
Log
     ↓
Measure
     ↓
Notify
```

The key DevOps principle is:

> **Use native S3 capabilities for standard storage behavior and Python/Boto3 when custom operational logic is actually required.**

Next:

```text
04-IAM-Automation.md
```

will cover IAM users, roles, policies, policy audits, access-key auditing, least privilege, AssumeRole, cross-account automation and production-safe IAM operations.
