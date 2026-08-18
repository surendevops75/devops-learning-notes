# RDS-Automation

## Python for AWS DevOps — RDS Inventory, Database Health, Snapshots, Backups, Subnet Groups, Security, Monitoring & Production Automation

Amazon RDS is a managed relational database service commonly used by DevOps teams for:

```text
PostgreSQL
MySQL
MariaDB
Oracle
SQL Server
Aurora
```

Python/Boto3 can automate:

```text
database inventory
status checks
snapshot inventory
backup audits
subnet-group audits
security-group discovery
parameter-group audits
maintenance configuration
Multi-AZ checks
storage checks
engine/version reporting
read-replica inventory
event collection
CloudWatch metric queries
operational reports
cross-account audits
```

The key rule is:

> **Database automation must prioritize safety and recoverability over speed.**

A production database is not an EC2 instance that can simply be restarted or terminated without considering application impact.

---

# 1. RDS Mental Model

Typical architecture:

```text
Application
    |
    v
ALB / EKS / EC2
    |
    v
RDS
    |
    +-- DB Subnet Group
    |
    +-- Private Subnets
    |
    +-- Security Group
    |
    +-- Parameter Group
    |
    +-- Option Group
    |
    +-- Backup
    |
    +-- Monitoring
```

---

# 2. Boto3 RDS Client

```python
import boto3

rds = boto3.client(
    "rds",
    region_name="ap-south-1",
)
```

RDS is regional, so always record the region in inventory reports.

---

# 3. Validate AWS Identity

For sensitive operations:

```python
sts = boto3.client("sts")

identity = sts.get_caller_identity()

print(identity["Account"])
print(identity["Arn"])
```

Production automation should validate the expected account before modifying a database.

---

# 4. List DB Instances

```python
paginator = rds.get_paginator(
    "describe_db_instances"
)

for page in paginator.paginate():

    for db in page.get(
        "DBInstances",
        []
    ):
        print(
            db["DBInstanceIdentifier"],
            db["DBInstanceStatus"]
        )
```

Use pagination for complete inventory.

---

# 5. RDS Instance Metadata

Useful fields include:

```text
DBInstanceIdentifier
DBInstanceClass
Engine
EngineVersion
DBInstanceStatus
AvailabilityZone
MultiAZ
StorageType
AllocatedStorage
Endpoint
Port
DBSubnetGroup
VpcSecurityGroups
BackupRetentionPeriod
PreferredBackupWindow
PreferredMaintenanceWindow
```

---

# 6. RDS Status

Common states include:

```text
available
creating
backing-up
modifying
rebooting
maintenance
failed
stopped
starting
stopping
```

Never treat every non-`available` state as an incident.

Some states are normal during maintenance or provisioning.

---

# 7. RDS Health Check

A simple operational check:

```python
def is_available(db):
    return db.get(
        "DBInstanceStatus"
    ) == "available"
```

A production health check should also inspect:

```text
Multi-AZ
backup configuration
storage
endpoint
security
monitoring
events
```

---

# 8. RDS Engine Inventory

```python
for db in databases:
    print(
        db["Engine"],
        db["EngineVersion"]
    )
```

This supports:

```text
version inventory
upgrade planning
security review
standardization
```

---

# 9. Engine Version Audit

Example:

```text
prod-db-1
PostgreSQL 16.x
PASS

legacy-db
PostgreSQL older-version
REVIEW
```

Do not hardcode a version as universally insecure. Compare against your organization's supported-version policy and AWS engine lifecycle information.

---

# 10. RDS Instance Class

```python
print(
    db["DBInstanceClass"]
)
```

Examples:

```text
db.t*
db.m*
db.r*
```

Instance family selection depends on:

```text
CPU
memory
workload
IO
network
cost
```

---

# 11. RDS Storage

```python
print(
    db.get(
        "AllocatedStorage"
    )
)
```

Track:

```text
allocated storage
storage type
max allocated storage
IOPS
storage throughput where applicable
```

---

# 12. Storage Autoscaling

RDS can support storage autoscaling depending on engine/configuration.

Audit relevant fields such as:

```text
MaxAllocatedStorage
```

A database with insufficient storage planning can eventually experience application failures.

---

# 13. RDS Storage Types

Common types include:

```text
gp2
gp3
io1
io2
```

depending on engine and AWS support.

For modern workloads, evaluate gp3 where appropriate rather than assuming gp2 is always optimal.

---

# 14. RDS IOPS

For IOPS-oriented workloads:

```text
storage type
+
allocated storage
+
IOPS
```

must be evaluated together.

Do not change storage settings without workload and performance analysis.

---

# 15. RDS Multi-AZ

Check:

```python
print(
    db.get("MultiAZ")
)
```

For production databases requiring high availability:

```text
MultiAZ = True
```

may be an expected policy.

Do not apply the same availability requirement to every development database.

---

# 16. Multi-AZ Is Not a Backup

Multi-AZ primarily addresses:

```text
high availability
failover
```

It does not replace:

```text
backup
restore
point-in-time recovery
```

---

# 17. RDS Read Replicas

Read replicas can support:

```text
read scaling
reporting
analytics
migration
disaster-recovery strategies
```

They are different from Multi-AZ standby configurations.

---

# 18. List Read Replicas

RDS instance metadata can include replica information.

Inspect fields such as:

```python
db.get(
    "ReadReplicaDBInstanceIdentifiers",
    []
)
```

---

# 19. RDS Endpoint

```python
endpoint = db.get(
    "Endpoint",
    {}
)

print(
    endpoint.get("Address")
)

print(
    endpoint.get("Port")
)
```

Never log database credentials.

---

# 20. RDS Endpoint Security

Do not make a database publicly reachable merely to solve application connectivity.

Preferred pattern:

```text
Application
    ↓
Private networking
    ↓
RDS
```

with controlled security-group rules.

---

# 21. Publicly Accessible Flag

Check:

```python
print(
    db.get(
        "PubliclyAccessible"
    )
)
```

For many internal production databases, this should be:

```text
False
```

But evaluate against the actual architecture.

---

# 22. RDS Security Groups

RDS uses VPC security groups.

Inspect:

```python
for sg in db.get(
    "VpcSecurityGroups",
    []
):
    print(
        sg.get("VpcSecurityGroupId"),
        sg.get("Status")
    )
```

---

# 23. RDS Security Group Architecture

Typical:

```text
ALB/Backend SG
       ↓
Application SG
       ↓
Database SG
       ↓
RDS
```

The database SG should allow only expected application sources.

---

# 24. Database Port

Typical examples:

```text
PostgreSQL → 5432
MySQL → 3306
MariaDB → 3306
SQL Server → 1433
Oracle → 1521
```

Always use the actual endpoint port from RDS metadata rather than assuming it.

---

# 25. Database SG Audit

A security automation can flag:

```text
database port from 0.0.0.0/0
database port from ::/0
unexpected source SG
unexpected application subnet
```

---

# 26. RDS DB Subnet Group

Inspect:

```python
subnet_group = db.get(
    "DBSubnetGroup",
    {}
)

print(
    subnet_group.get(
        "DBSubnetGroupName"
    )
)
```

---

# 27. DB Subnet Group

A DB subnet group defines the subnets where RDS can place database resources.

Production design should normally use multiple Availability Zones.

---

# 28. DB Subnet Group Inventory

```python
paginator = rds.get_paginator(
    "describe_db_subnet_groups"
)

for page in paginator.paginate():

    for group in page.get(
        "DBSubnetGroups",
        []
    ):
        print(
            group["DBSubnetGroupName"],
            group["VpcId"]
        )
```

---

# 29. DB Subnet Group AZ Coverage

Inspect:

```python
for subnet in group.get(
    "Subnets",
    []
):
    print(
        subnet.get(
            "SubnetIdentifier"
        ),
        subnet.get(
            "SubnetAvailabilityZone"
        )
    )
```

Use this to validate multi-AZ subnet coverage.

---

# 30. RDS Parameter Groups

Parameter groups control engine parameters.

List:

```python
paginator = rds.get_paginator(
    "describe_db_parameter_groups"
)

for page in paginator.paginate():

    for group in page.get(
        "DBParameterGroups",
        []
    ):
        print(
            group["DBParameterGroupName"]
        )
```

---

# 31. Parameter Group Association

Check:

```python
for group in db.get(
    "DBParameterGroups",
    []
):
    print(
        group.get(
            "DBParameterGroupName"
        ),
        group.get(
            "ParameterApplyStatus"
        )
    )
```

---

# 32. Parameter Apply Status

Possible status:

```text
in-sync
pending-reboot
applying
```

A `pending-reboot` parameter change can be operationally important.

---

# 33. RDS Parameter Audit

A production audit can verify:

```text
expected parameter group
engine family
parameter values
apply method
pending reboot
```

Do not blindly modify engine parameters.

---

# 34. RDS Option Groups

Some database engines use option groups for additional features.

Inventory:

```python
paginator = rds.get_paginator(
    "describe_option_groups"
)

for page in paginator.paginate():

    for group in page.get(
        "OptionGroupsList",
        []
    ):
        print(
            group.get(
                "OptionGroupName"
            )
        )
```

---

# 35. RDS Backup Retention

Check:

```python
print(
    db.get(
        "BackupRetentionPeriod"
    )
)
```

For production databases, retention should match:

```text
RPO
compliance
business requirements
```

---

# 36. Backup Retention

Do not assume:

```text
7 days
```

is always correct.

A critical database may require:

```text
7
14
30
35
```

or another policy-defined period.

---

# 37. Preferred Backup Window

```python
print(
    db.get(
        "PreferredBackupWindow"
    )
)
```

Review this against application traffic patterns.

---

# 38. Maintenance Window

```python
print(
    db.get(
        "PreferredMaintenanceWindow"
    )
)
```

Production maintenance should be scheduled during an approved operational window.

---

# 39. Maintenance vs Backup Window

Avoid unnecessary overlap between:

```text
backup
maintenance
peak application traffic
```

unless the architecture and AWS behavior make the overlap acceptable.

---

# 40. Automated Backup

RDS automated backups support:

```text
point-in-time recovery
```

within the configured retention period.

This is different from manually created snapshots.

---

# 41. Manual Snapshot

Create:

```python
response = rds.create_db_snapshot(
    DBInstanceIdentifier=db_identifier,
    DBSnapshotIdentifier=snapshot_name,
)
```

This is a mutating operation.

Use a unique and policy-compliant snapshot name.

---

# 42. Snapshot Naming

Example:

```text
prod-orders-pre-release-20260817
```

Useful metadata:

```text
environment
application
purpose
timestamp
```

---

# 43. Snapshot Inventory

```python
paginator = rds.get_paginator(
    "describe_db_snapshots"
)

for page in paginator.paginate():

    for snapshot in page.get(
        "DBSnapshots",
        []
    ):
        print(
            snapshot.get(
                "DBSnapshotIdentifier"
            ),
            snapshot.get(
                "Status"
            )
        )
```

---

# 44. Snapshot Status

Common states include:

```text
creating
available
deleting
failed
```

Do not treat snapshot creation as complete until the snapshot reaches the expected state.

---

# 45. Snapshot Verification

```text
snapshot created
      ↓
poll status
      ↓
available
      ↓
record ARN
      ↓
report success
```

Use bounded polling.

---

# 46. Waiter vs Polling

For operations where an appropriate AWS waiter exists, use it.

Otherwise:

```text
poll
sleep
timeout
```

Avoid infinite loops.

---

# 47. Snapshot Polling

Concept:

```python
import time

for _ in range(60):

    response = rds.describe_db_snapshots(
        DBSnapshotIdentifier=snapshot_name
    )

    status = response[
        "DBSnapshots"
    ][0]["Status"]

    if status == "available":
        break

    time.sleep(10)
```

Production code should include timeout handling and failure states.

---

# 48. Snapshot Delete

```python
rds.delete_db_snapshot(
    DBSnapshotIdentifier=snapshot_name
)
```

Destructive.

Never delete snapshots solely by age without checking:

```text
retention
owner
environment
restore requirements
compliance
```

---

# 49. Snapshot Cleanup

Safe workflow:

```text
discover
 ↓
filter
 ↓
protected check
 ↓
retention check
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

# 50. RDS Snapshot Cost

Snapshots consume storage.

A cost audit can identify:

```text
old snapshots
unused snapshots
duplicate snapshots
test snapshots
```

But storage billing behavior can be more nuanced than simply summing snapshot sizes, so use AWS billing/cost data for authoritative cost analysis.

---

# 51. Snapshot Copy

Snapshots can be copied for:

```text
cross-region DR
cross-account backup
migration
```

Use appropriate encryption and KMS permissions.

---

# 52. Cross-Region Snapshot Copy

Concept:

```text
Production Region
       ↓
Snapshot
       ↓
Copy
       ↓
DR Region
```

Python can orchestrate the workflow.

For recurring DR, evaluate native AWS backup/replication capabilities where appropriate.

---

# 53. Encrypted Snapshot Copy

For encrypted snapshots, KMS key permissions and destination key selection matter.

Do not assume S3-style encryption semantics.

---

# 54. RDS Encryption

Check:

```python
print(
    db.get(
        "StorageEncrypted"
    )
)
```

For production sensitive databases:

```text
StorageEncrypted = True
```

is commonly required.

---

# 55. KMS Key

If encrypted:

```python
print(
    db.get(
        "KmsKeyId"
    )
)
```

Do not expose sensitive key configuration unnecessarily in external logs.

---

# 56. RDS Encryption Audit

Report:

```text
DB
Encrypted
KMS key configured
Backup encryption
Snapshot encryption
```

Encryption compliance should match organizational policy.

---

# 57. IAM and RDS

A Python automation role may need permissions such as:

```text
rds:DescribeDBInstances
rds:DescribeDBSnapshots
rds:DescribeDBSubnetGroups
```

Snapshot creation requires additional write permissions.

Grant only the operations the automation actually performs.

---

# 58. RDS Read-Only Role

For inventory:

```text
describe only
```

Prefer a dedicated read-only automation role.

---

# 59. RDS Snapshot Role

A backup automation role may need:

```text
describe
create snapshot
copy snapshot
```

but should not automatically receive:

```text
delete database
modify database
```

unless explicitly required.

---

# 60. RDS Delete Protection

Check:

```python
print(
    db.get(
        "DeletionProtection"
    )
)
```

For important production databases, deletion protection can provide an additional safety layer.

---

# 61. Deletion Protection

Do not disable deletion protection merely to make automation easier.

A production deletion workflow should require:

```text
approval
backup
change record
dependency review
```

---

# 62. RDS Deletion Workflow

```text
identify DB
 ↓
validate account
 ↓
validate environment
 ↓
check deletion protection
 ↓
confirm owner
 ↓
verify latest backup/snapshot
 ↓
approval
 ↓
delete
 ↓
verify
```

---

# 63. RDS Stop/Start

Some RDS configurations support stopping and starting.

Use carefully because:

```text
application impact
availability impact
billing behavior
restart behavior
```

must be considered.

---

# 64. Dev Environment Scheduler

A common automation:

```text
development DB
 ↓
stop after working hours
 ↓
start before working hours
```

Do not apply the same scheduler blindly to:

```text
production
DR
replicas
critical integrations
```

---

# 65. Tag-Based RDS Scheduling

Example tags:

```text
Environment=dev
AutoStop=true
Schedule=business-hours
```

Python can select only explicitly opted-in resources.

---

# 66. Safe Auto-Stop Logic

```text
Environment == dev
AND
AutoStop == true
AND
not protected
```

Then:

```text
dry-run
 ↓
stop
```

This is safer than:

```text
stop every non-production-looking database
```

---

# 67. RDS Events

RDS provides events that can help identify:

```text
maintenance
failover
backup
storage
configuration changes
```

Python can retrieve event information for operational reporting.

---

# 68. Describe Events

```python
response = rds.describe_events(
    SourceType="db-instance",
    Duration=60,
)

for event in response.get(
    "Events",
    []
):
    print(
        event.get(
            "Message"
        )
    )
```

Use an appropriate duration and source filter for the workflow.

---

# 69. RDS Event Monitoring

Architecture:

```text
RDS
 ↓
Events
 ↓
Python
 ↓
classification
 ↓
notification
```

For real-time event-driven automation, evaluate native AWS event services rather than frequent polling.

---

# 70. RDS Metrics

Operational metrics can include:

```text
CPUUtilization
DatabaseConnections
FreeStorageSpace
FreeableMemory
ReadIOPS
WriteIOPS
ReadLatency
WriteLatency
NetworkReceiveThroughput
NetworkTransmitThroughput
```

Metric availability depends on engine/configuration.

---

# 71. CloudWatch Metrics

RDS publishes many metrics to CloudWatch.

Python can retrieve metrics using:

```python
cloudwatch = boto3.client(
    "cloudwatch"
)
```

Use the CloudWatch API for historical metric analysis.

---

# 72. Metric Query Concept

```python
response = cloudwatch.get_metric_statistics(
    Namespace="AWS/RDS",
    MetricName="CPUUtilization",
    ...
)
```

Use appropriate dimensions:

```text
DBInstanceIdentifier
```

and time windows.

---

# 73. RDS CPU Report

A scheduled report can calculate:

```text
average CPU
maximum CPU
minimum CPU
```

over a defined period.

This supports:

```text
capacity review
rightsizing
incident analysis
```

---

# 74. Free Storage Alert

A production health check can evaluate:

```text
FreeStorageSpace
```

against a workload-specific threshold.

Do not use only percentage thresholds without considering database size and growth rate.

---

# 75. Database Connections

Monitor:

```text
DatabaseConnections
```

High connections may indicate:

```text
connection pool problem
application leak
traffic spike
database sizing issue
```

Do not assume the database itself is always the root cause.

---

# 76. RDS Performance Troubleshooting

When latency increases:

```text
CPU
 ↓
memory
 ↓
IOPS
 ↓
storage latency
 ↓
connections
 ↓
network
 ↓
database engine
 ↓
application
```

Python can collect infrastructure-level evidence.

---

# 77. RDS Monitoring Automation

Build:

```bash
python rdsops.py health
```

Output:

```text
DB: orders-prod
Status: available
Multi-AZ: true
Encrypted: true
Backup Retention: 14d
Free Storage: healthy
CPU: normal
Connections: normal
```

---

# 78. RDS Inventory Project

Build:

```bash
python rdsops.py inventory
```

Collect:

```text
account
region
identifier
engine
version
class
status
Multi-AZ
storage
encryption
backup retention
endpoint
subnet group
security groups
```

---

# 79. RDS Compliance Project

Check:

```text
encryption
Multi-AZ
backup retention
deletion protection
public accessibility
security groups
subnet group
engine version
tags
```

---

# 80. RDS Tag Audit

Required tags might include:

```text
Environment
Application
Owner
CostCenter
ManagedBy
DataClassification
```

Python can flag missing tags.

---

# 81. RDS Tag Selection

```python
def tag_value(resource, key):
    for tag in resource.get(
        "TagList",
        []
    ):
        if tag.get("Key") == key:
            return tag.get("Value")

    return None
```

Use the tag structure returned by the specific RDS API being queried.

---

# 82. RDS Multi-Account Audit

```text
central automation
       ↓
AssumeRole
       ↓
target account
       ↓
RDS inventory
       ↓
report
```

Validate:

```text
account ID
region
environment
```

before reporting or modifying.

---

# 83. RDS Multi-Region Audit

```python
regions = [
    "ap-south-1",
    "ap-southeast-1",
]

for region in regions:

    rds = boto3.client(
        "rds",
        region_name=region,
    )

    # inventory
```

Use approved regions rather than scanning blindly.

---

# 84. RDS + EKS

Typical architecture:

```text
EKS
 ↓
private networking
 ↓
RDS
```

Python can correlate:

```text
EKS VPC
RDS VPC
subnets
security groups
ports
```

---

# 85. RDS + ALB

ALB normally does not directly connect to a database.

Typical architecture:

```text
Client
 ↓
ALB
 ↓
Application
 ↓
RDS
```

Do not create unnecessary ALB-to-database security rules.

---

# 86. RDS + Terraform

Terraform can manage:

```text
RDS instance
subnet group
parameter group
option group
security group
monitoring configuration
```

Python should generally focus on:

```text
inventory
health
audit
backup operations
reporting
```

Avoid conflicting ownership.

---

# 87. RDS + Jenkins

```text
Jenkins
 ↓
Python health check
 ↓
RDS inventory
 ↓
report
```

For database changes:

```text
Jenkins
 ↓
migration/test
 ↓
approval
 ↓
controlled deployment
```

Do not put arbitrary destructive RDS operations directly into a generic pipeline.

---

# 88. RDS + GitHub Actions

```text
GitHub Actions
 ↓
OIDC
 ↓
IAM role
 ↓
Python
 ↓
RDS audit/backup
```

Use a read-only role for reporting jobs.

---

# 89. RDS + Prometheus/Grafana

RDS metrics are available through AWS monitoring services.

A custom Python automation can expose higher-level metrics:

```text
rds_compliance_failures
rds_backup_check_failures
rds_instances_unavailable
rds_old_snapshot_count
```

Grafana can visualize those automation metrics.

---

# 90. RDS + ELK

Python can emit structured events:

```json
{
  "service": "rds-audit",
  "database": "orders-prod",
  "check": "encryption",
  "status": "PASS"
}
```

These can be forwarded to a centralized logging platform.

---

# 91. RDS Backup Verification Project

Workflow:

```text
identify DB
 ↓
check automated backup retention
 ↓
create manual snapshot when required
 ↓
wait for available
 ↓
record snapshot
 ↓
report
```

For critical systems, pair backup checks with periodic restore testing.

---

# 92. Restore Testing

A robust database backup process is:

```text
backup
 ↓
restore to isolated environment
 ↓
validate database
 ↓
run application checks
 ↓
document RTO/RPO
```

A backup that has never been restored is an unverified assumption.

---

# 93. RDS Snapshot Restore

Conceptually:

```python
rds.restore_db_instance_from_db_snapshot(
    DBInstanceIdentifier="restore-test",
    DBSnapshotIdentifier=snapshot_name,
)
```

This creates a database from the snapshot.

Use an isolated environment and explicit naming.

---

# 94. Restore Test Safety

Never restore test databases into production identifiers or networks accidentally.

Validate:

```text
account
region
identifier
subnet group
security group
```

before executing.

---

# 95. RDS Snapshot Copy DR

Production pattern:

```text
Primary Region
    ↓
Snapshot
    ↓
Copy to DR Region
    ↓
Validate
```

Automate status checks and reporting.

---

# 96. RDS Failure Handling

Possible errors:

```text
DBInstanceNotFound
DBSnapshotNotFound
InvalidDBInstanceState
InvalidDBSnapshotState
AccessDenied
StorageQuotaExceeded
```

Do not retry all errors indefinitely.

---

# 97. Invalid DB Instance State

Example:

```text
requested operation
        ↓
DB not in valid state
        ↓
InvalidDBInstanceState
```

The correct response may be:

```text
wait
investigate
or fail safely
```

depending on the workflow.

---

# 98. RDS Snapshot Failure

If a snapshot fails:

```text
inspect RDS status
inspect events
inspect permissions
inspect storage
retry according to policy
```

Do not blindly create hundreds of duplicate snapshots.

---

# 99. RDS API Retry

```python
from botocore.config import Config

config = Config(
    retries={
        "max_attempts": 5,
        "mode": "standard",
    }
)

rds = boto3.client(
    "rds",
    config=config,
)
```

Use bounded application-level polling separately for long-running operations.

---

# 100. RDS Logging Safety

Never log:

```text
database password
connection strings containing credentials
secret values
```

Safe operational fields include:

```text
DB identifier
region
engine
status
snapshot identifier
operation
error code
```

---

# 101. RDS Credential Management

Application credentials should generally be stored in a secrets-management system rather than:

```text
Python source
Git
Docker image
plain-text config
```

Python automation should reference secrets securely rather than print them.

---

# 102. RDS Secrets Rotation

For applications using managed secrets:

```text
secret
 ↓
rotation
 ↓
application refresh
 ↓
database authentication
```

Do not rotate database credentials without understanding application connection-pool behavior.

---

# 103. RDS Maintenance Automation

A maintenance workflow can:

```text
identify approved DBs
 ↓
check maintenance window
 ↓
check pending modifications
 ↓
notify owner
 ↓
monitor events
```

Avoid automatically forcing reboots in production.

---

# 104. Pending Reboot Audit

Check:

```python
for group in db.get(
    "DBParameterGroups",
    []
):
    if group.get(
        "ParameterApplyStatus"
    ) == "pending-reboot":

        print(
            db["DBInstanceIdentifier"]
        )
```

Then coordinate a maintenance action.

---

# 105. RDS Configuration Drift

Compare:

```text
desired configuration
        vs
actual RDS configuration
```

Potential drift:

```text
Multi-AZ
storage
backup retention
parameter group
subnet group
security group
deletion protection
tags
```

---

# 106. RDS Desired-State Report

Example:

```text
orders-prod

Multi-AZ:
Desired = true
Actual  = true
PASS

Encryption:
Desired = true
Actual  = true
PASS

Backup Retention:
Desired = 14
Actual  = 7
FAIL
```

This is more useful than a generic "database healthy" message.

---

# 107. RDS Compliance JSON

```python
report = {
    "database": db_id,
    "encryption": encrypted,
    "multi_az": multi_az,
    "backup_retention": retention,
    "deletion_protection": protection,
}
```

Serialize to JSON for pipeline consumption.

---

# 108. RDS CLI Design

Example:

```bash
python rdsops.py inventory
```

```bash
python rdsops.py health
```

```bash
python rdsops.py snapshots
```

```bash
python rdsops.py audit
```

```bash
python rdsops.py backup
```

```bash
python rdsops.py events
```

---

# 109. CLI Arguments

Useful:

```text
--profile
--region
--db
--environment
--output
--format
--dry-run
```

For destructive operations:

```text
--confirm
```

can be an additional guard.

---

# 110. RDS Health Architecture

```text
STS
 ↓
Account validation
 ↓
RDS inventory
 ↓
status
 ↓
backup
 ↓
encryption
 ↓
Multi-AZ
 ↓
storage
 ↓
security
 ↓
events
 ↓
metrics
 ↓
report
```

---

# 111. RDS Audit Output

Example:

```text
RDS Audit
=========

DB: orders-prod
Status: PASS
Encryption: PASS
Multi-AZ: PASS
Backup Retention: PASS
Deletion Protection: PASS
Public Access: PASS
Security Group: PASS
Subnet Group: PASS
Parameter Group: PASS
```

---

# 112. RDS Cost Audit

Potential candidates:

```text
development DBs
oversized instances
old snapshots
unnecessary replicas
unused DBs
```

Use workload metrics and AWS billing data before making rightsizing decisions.

---

# 113. RDS Rightsizing

Do not use:

```text
CPU alone
```

Consider:

```text
CPU
memory
connections
IOPS
latency
storage
network
peak traffic
```

Then recommend changes rather than blindly resizing production.

---

# 114. Dev RDS Scheduler Project

Architecture:

```text
Event/Scheduler
      ↓
Python
      ↓
Find AutoStop=true
      ↓
Validate environment
      ↓
Stop
      ↓
Report
```

Start workflow:

```text
Scheduler
 ↓
find stopped dev DBs
 ↓
start
 ↓
wait
 ↓
report
```

---

# 115. Production Safety Guard

```python
if environment == "prod":
    raise RuntimeError(
        "Automatic stop disabled"
    )
```

A better design also uses explicit allowlists/tags and IAM restrictions.

---

# 116. RDS Snapshot Retention Script

Concept:

```text
list snapshots
 ↓
filter by tag/name
 ↓
calculate age
 ↓
exclude protected
 ↓
dry-run
 ↓
approval
 ↓
delete candidates
```

Use native backup/lifecycle capabilities where they satisfy the requirement.

---

# 117. Snapshot Protection

Use explicit metadata such as:

```text
Retention=long
Protected=true
Environment=prod
```

Your cleanup script should honor protected resources.

---

# 118. RDS Inventory Function

```python
def list_databases(rds):

    paginator = rds.get_paginator(
        "describe_db_instances"
    )

    databases = []

    for page in paginator.paginate():
        databases.extend(
            page.get(
                "DBInstances",
                []
            )
        )

    return databases
```

---

# 119. Snapshot Inventory Function

```python
def list_snapshots(rds):

    paginator = rds.get_paginator(
        "describe_db_snapshots"
    )

    snapshots = []

    for page in paginator.paginate():
        snapshots.extend(
            page.get(
                "DBSnapshots",
                []
            )
        )

    return snapshots
```

---

# 120. RDS Security Audit Function

Concept:

```python
def audit_database(db):

    return {
        "encrypted": db.get(
            "StorageEncrypted"
        ),
        "multi_az": db.get(
            "MultiAZ"
        ),
        "public": db.get(
            "PubliclyAccessible"
        ),
        "deletion_protection": db.get(
            "DeletionProtection"
        ),
    }
```

---

# 121. RDS Error Handling

```python
from botocore.exceptions import ClientError

try:
    response = rds.describe_db_instances()

except ClientError as exc:

    error = exc.response.get(
        "Error",
        {}
    )

    print(
        error.get("Code"),
        error.get("Message")
    )

    raise
```

---

# 122. RDS Testing With Stubber

```python
from botocore.stub import Stubber

stubber = Stubber(rds)

stubber.add_response(
    "describe_db_instances",
    {
        "DBInstances": [
            {
                "DBInstanceIdentifier":
                    "test-db",
                "DBInstanceStatus":
                    "available",
            }
        ]
    },
)

stubber.activate()
```

Use deterministic API responses for unit tests.

---

# 123. RDS Integration Testing

Use:

```text
dedicated AWS account
test database
test snapshot
test subnet group
test security group
```

Never perform destructive integration tests against production databases.

---

# 124. RDS Monitoring Metrics for DevOps

Useful operational indicators:

```text
CPU
connections
free storage
memory
IOPS
latency
network throughput
replica lag where applicable
```

The exact metric set should match the engine and workload.

---

# 125. RDS Alert Automation

A Python job can produce:

```text
CRITICAL
DB unavailable

WARNING
Free storage below policy

REVIEW
Backup retention below policy

PASS
Encryption enabled
```

Use a monitoring system for real-time alerting when possible; Python can enrich or aggregate the findings.

---

# 126. RDS Incident Runbook

When database connectivity fails:

```text
1. Check RDS status
2. Check recent RDS events
3. Check endpoint/DNS
4. Check security groups
5. Check subnet/network path
6. Check connections
7. Check CPU/memory/storage/IO
8. Check application connection pool
9. Check recent changes
```

Do not immediately restart the database.

---

# 127. RDS Incident — Storage Full

Symptoms:

```text
write failures
application errors
database instability
```

Investigate:

```text
FreeStorageSpace
growth rate
logs
temporary files
workload
storage autoscaling
```

Do not delete database data without understanding application and retention requirements.

---

# 128. RDS Incident — High Connections

Investigate:

```text
application deployment
connection pool
traffic spike
connection leaks
database limits
read/write architecture
```

Python can collect connection-related metrics but root cause may be in the application.

---

# 129. RDS Incident — High CPU

Check:

```text
query workload
connections
traffic
instance size
engine behavior
recent deployment
```

Do not immediately resize without understanding the workload.

---

# 130. RDS Incident — Replica Lag

Investigate:

```text
write workload
replica health
network
IO
long-running transactions
engine-specific behavior
```

Replica lag thresholds should be workload-specific.

---

# 131. RDS Incident — Failover

When Multi-AZ failover occurs:

```text
RDS
 ↓
new primary
 ↓
endpoint remains abstraction
```

Applications should use the RDS endpoint rather than hardcoded instance IP addresses.

---

# 132. RDS Endpoint Best Practice

Do not hardcode:

```text
private IP
```

Use the appropriate RDS endpoint.

This allows AWS-managed topology changes such as failover to be handled correctly.

---

# 133. RDS Disaster Recovery

Possible layers:

```text
automated backups
manual snapshots
snapshot copies
read replicas
Multi-AZ
cross-region DR
```

Choose according to:

```text
RPO
RTO
cost
business criticality
```

---

# 134. RDS Backup vs Multi-AZ

```text
Multi-AZ
→ availability/failover

Backup
→ recovery

Snapshot
→ point-in-time copy

Read Replica
→ read scaling / selected DR scenarios
```

They solve different problems.

---

# 135. RDS Compliance Checklist

```text
[ ] Encryption
[ ] Backup retention
[ ] Multi-AZ where required
[ ] Deletion protection
[ ] Private accessibility
[ ] Security group review
[ ] Subnet group coverage
[ ] Supported engine version
[ ] Parameter group
[ ] Tags
[ ] Monitoring
[ ] Restore testing
```

---

# 136. RDS Reliability Checklist

```text
[ ] Backup verified
[ ] Restore tested
[ ] Multi-AZ where required
[ ] Storage capacity monitored
[ ] Connection capacity monitored
[ ] Maintenance window
[ ] Failover behavior understood
[ ] Application uses endpoint
[ ] DR plan
```

---

# 137. RDS Security Checklist

```text
[ ] No public access unless approved
[ ] Database SG restricted
[ ] Encryption enabled
[ ] KMS access controlled
[ ] Credentials managed securely
[ ] IAM least privilege
[ ] No passwords in logs
[ ] Audit trail enabled
```

---

# 138. RDS Cost Checklist

```text
[ ] Instance sizing
[ ] Storage sizing
[ ] Storage autoscaling
[ ] Old snapshots
[ ] Read replicas
[ ] Dev auto-stop
[ ] Cross-region copies
[ ] Data transfer
```

---

# 139. Interview — What Is RDS?

**Answer:**

> Amazon RDS is a managed relational database service. AWS manages much of the underlying infrastructure and operational work while DevOps teams manage configuration, security, availability, backups, connectivity and application integration.

---

# 140. Interview — How Do You Automate RDS With Python?

**Answer:**

> I use Boto3 to inventory instances, check status, inspect backups and snapshots, validate encryption and Multi-AZ settings, audit subnet/security configuration, collect events and metrics, and generate operational reports.

---

# 141. Interview — Multi-AZ vs Read Replica?

**Answer:**

> Multi-AZ primarily provides high availability and failover. Read replicas primarily support read scaling and can be used in selected disaster-recovery architectures. They solve different problems.

---

# 142. Interview — Is Multi-AZ a Backup?

**Answer:**

> No. Multi-AZ improves availability and failover. Backups and snapshots provide recovery capabilities.

---

# 143. Interview — How Do You Automate RDS Backups?

**Answer:**

> I first verify automated backup retention. For a required manual backup, I create a snapshot, wait until it becomes available, verify it, tag or record it, and report the result. I also include periodic restore testing.

---

# 144. Interview — How Do You Safely Delete RDS Snapshots?

**Answer:**

> I identify candidates using explicit retention rules, exclude protected snapshots, perform a dry-run, obtain approval for production data, delete selected snapshots and verify the result.

---

# 145. Interview — How Do You Secure RDS?

**Answer:**

> I keep the database private where possible, restrict the database security group to required application sources, enable encryption, manage credentials securely, use least-privilege IAM and monitor database events and metrics.

---

# 146. Interview — How Do You Troubleshoot RDS Connectivity?

**Answer:**

> I check the database status and endpoint, then trace the network path through subnets, route tables, security groups and NACLs. I also verify DNS, port configuration and application connection behavior.

---

# 147. Interview — How Do You Check RDS Health With Boto3?

**Answer:**

> I inspect DBInstanceStatus and combine that with recent RDS events, backup configuration, storage, connections and relevant CloudWatch metrics. A status of `available` alone is not a complete health check.

---

# 148. Interview — How Do You Automate RDS Dev Start/Stop?

**Answer:**

> I use explicit tags such as `Environment=dev` and `AutoStop=true`, validate the account and environment, exclude protected resources, execute the change during an approved schedule and report failures.

---

# 149. Interview — Would You Auto-Stop Production RDS?

**Answer:**

> No. Production databases should be controlled through explicit change management. An automated scheduler should use an allowlist or explicit opt-in tags and never infer production eligibility from a resource name alone.

---

# 150. Interview — How Do You Check RDS Encryption?

**Answer:**

> I inspect the `StorageEncrypted` field and, when encrypted, review the associated KMS key configuration according to the organization's policy.

---

# 151. Interview — How Do You Audit RDS Backups?

**Answer:**

> I inspect backup retention, backup windows, snapshot inventory and backup status. For critical databases I also verify that restore testing is performed because configuration alone does not prove recoverability.

---

# 152. Interview — How Do You Handle an RDS Storage-Full Incident?

**Answer:**

> I confirm FreeStorageSpace and growth rate, inspect workload and database behavior, check whether storage autoscaling is configured, and follow the approved capacity or remediation procedure. I avoid deleting data as an emergency shortcut.

---

# 153. Interview — How Do You Handle RDS High CPU?

**Answer:**

> I correlate CPU with connections, I/O, workload changes, deployments and database behavior. I determine whether the issue is query/workload-related or capacity-related before resizing.

---

# 154. Interview — How Do You Handle RDS Failover?

**Answer:**

> I verify the RDS event and instance state, confirm the application uses the RDS endpoint rather than a hardcoded IP, and monitor reconnection and application health after failover.

---

# 155. Interview — How Do You Test RDS Backups?

**Answer:**

> I periodically restore a snapshot into an isolated environment, validate the database and application connectivity, measure recovery time and document the result against the expected RTO/RPO.

---

# 156. Interview — How Do You Prevent RDS Automation From Deleting Production?

**Answer:**

> I validate the AWS account, environment, database identifier, deletion protection, ownership and backup status. Destructive workflows use dry-run and explicit approval, and production databases are excluded by default.

---

# 157. Interview — How Do You Integrate RDS Automation With Terraform?

**Answer:**

> Terraform remains the source of truth for infrastructure configuration. Python/Boto3 performs operational checks, audits, reporting and controlled workflows. I avoid having Python silently mutate Terraform-managed configuration.

---

# 158. Interview — How Do You Automate RDS Across Accounts?

**Answer:**

> I use STS AssumeRole into a tightly scoped role in each target account, validate the returned account identity, enumerate approved regions and collect normalized RDS inventory.

---

# 159. Interview — How Do You Monitor RDS With Python?

**Answer:**

> I can query RDS configuration and CloudWatch metrics, calculate health indicators and publish structured results. For real-time alerting, I prefer the organization's monitoring platform and use Python for enrichment and automation.

---

# 160. Interview — What Would You Automate First for RDS?

**Answer:**

> I would start with read-only inventory and compliance checks, then backup verification and reporting. After that I would add explicitly scoped development automation such as scheduled start/stop.

---

# 161. Interview — How Do You Make RDS Automation Idempotent?

**Answer:**

> I check the current database state before acting, make operations conditional on the desired state and avoid repeatedly creating snapshots or applying changes when the requested state already exists.

---

# 162. Interview — What Is the Biggest RDS Automation Mistake?

**Answer:**

> Treating a database like a stateless server. Database operations have consequences for data, availability and recovery, so every automation should include state validation, backup awareness and safe rollback or recovery planning.

---

# 163. Final RDS Automation Mental Model

```text
Validate Account
       ↓
Discover Database
       ↓
Check Status
       ↓
Check Network
       ↓
Check Security
       ↓
Check Backup
       ↓
Check HA
       ↓
Check Storage
       ↓
Check Events/Metrics
       ↓
Generate Report
       ↓
Operate Only With Guardrails
       ↓
Verify
```

The key DevOps principle is:

> **For databases, automation must protect availability and recoverability before optimizing convenience or cost.**

Next:

```text
07-EKS-Automation.md
```

will cover EKS cluster discovery, node groups, Kubernetes/EKS integration, add-ons, networking, IAM, cluster health, scaling, upgrades, node troubleshooting and production EKS automation.
