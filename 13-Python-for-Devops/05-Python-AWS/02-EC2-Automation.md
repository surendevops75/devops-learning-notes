# EC2-Automation

## Python for AWS DevOps — EC2 Discovery, Operations, Tagging, Scheduling, Auditing & Production Automation

EC2 automation is one of the most common real-world uses of Python and Boto3 in DevOps.

A production EC2 automation workflow should not simply do:

```python
ec2.stop_instances(...)
```

It should understand:

```text
AWS account
↓
region
↓
IAM permissions
↓
resource discovery
↓
filters/tags
↓
current state
↓
safety validation
↓
dry-run
↓
operation
↓
wait/verify
↓
logging
↓
report/notification
```

This module focuses on **production-oriented EC2 automation**.

---

# 1. EC2 Automation Use Cases

Python/Boto3 can automate:

```text
instance inventory
instance discovery
start/stop/reboot
instance state checks
tagging
AMI creation
AMI inventory
EBS audits
public IP audits
security-group discovery
instance scheduling
cost optimization
health checks
deployment validation
cross-account reporting
compliance checks
```

---

# 2. EC2 Boto3 Client

```python
import boto3

ec2 = boto3.client(
    "ec2",
    region_name="ap-south-1",
)
```

---

# 3. Basic EC2 Discovery

```python
response = ec2.describe_instances()

for reservation in response.get(
    "Reservations",
    []
):
    for instance in reservation.get(
        "Instances",
        []
    ):
        print(
            instance.get("InstanceId")
        )
```

For production, use a paginator.

---

# 4. EC2 Paginator

```python
paginator = ec2.get_paginator(
    "describe_instances"
)

for page in paginator.paginate():

    for reservation in page.get(
        "Reservations",
        []
    ):
        for instance in reservation.get(
            "Instances",
            []
        ):
            print(
                instance.get("InstanceId")
            )
```

This avoids processing only the first page.

---

# 5. EC2 Instance Metadata

Useful fields include:

```text
InstanceId
InstanceType
State
PrivateIpAddress
PublicIpAddress
SubnetId
VpcId
SecurityGroups
IamInstanceProfile
LaunchTime
Tags
ImageId
Placement
```

---

# 6. Extract Instance Details

```python
instance_id = instance.get(
    "InstanceId"
)

instance_type = instance.get(
    "InstanceType"
)

private_ip = instance.get(
    "PrivateIpAddress"
)

public_ip = instance.get(
    "PublicIpAddress"
)

image_id = instance.get(
    "ImageId"
)
```

---

# 7. Instance State

```python
state = instance.get(
    "State",
    {}
)

state_name = state.get(
    "Name"
)

print(
    instance_id,
    state_name
)
```

Possible states include:

```text
pending
running
stopping
stopped
shutting-down
terminated
```

---

# 8. EC2 State Machine

```text
pending
   ↓
running
   ↓
stopping
   ↓
stopped
```

or:

```text
running
   ↓
shutting-down
   ↓
terminated
```

Do not assume every transition is instantaneous.

---

# 9. Filter Running Instances

```python
response = ec2.describe_instances(
    Filters=[
        {
            "Name": "instance-state-name",
            "Values": ["running"],
        }
    ]
)
```

Prefer server-side filtering.

---

# 10. Filter by Instance ID

```python
response = ec2.describe_instances(
    InstanceIds=[
        "i-xxxxxxxx"
    ]
)
```

Use actual IDs from configuration or discovery rather than hardcoding production resources.

---

# 11. Filter by VPC

```python
response = ec2.describe_instances(
    Filters=[
        {
            "Name": "vpc-id",
            "Values": [
                "vpc-xxxxxxxx"
            ],
        }
    ]
)
```

---

# 12. Filter by Subnet

```python
response = ec2.describe_instances(
    Filters=[
        {
            "Name": "subnet-id",
            "Values": [
                "subnet-xxxxxxxx"
            ],
        }
    ]
)
```

---

# 13. Filter by AMI

```python
response = ec2.describe_instances(
    Filters=[
        {
            "Name": "image-id",
            "Values": [
                "ami-xxxxxxxx"
            ],
        }
    ]
)
```

---

# 14. Filter by Instance Type

```python
response = ec2.describe_instances(
    Filters=[
        {
            "Name": "instance-type",
            "Values": [
                "t3.medium"
            ],
        }
    ]
)
```

---

# 15. Filter by Tag

AWS tag filters can be used directly.

```python
response = ec2.describe_instances(
    Filters=[
        {
            "Name": "tag:Environment",
            "Values": ["dev"],
        }
    ]
)
```

---

# 16. Multiple Tag Filters

```python
Filters=[
    {
        "Name": "tag:Environment",
        "Values": ["dev"],
    },
    {
        "Name": "tag:ManagedBy",
        "Values": [
            "PythonAutomation"
        ],
    },
]
```

This is safer than selecting all instances.

---

# 17. Tag-Based Automation

A production automation policy could be:

```text
Environment=dev
ManagedBy=PythonAutomation
Protected!=true
```

Then only matching resources are eligible.

---

# 18. Extract EC2 Tags

```python
tags = instance.get(
    "Tags",
    []
)

for tag in tags:
    print(
        tag.get("Key"),
        tag.get("Value")
    )
```

---

# 19. Convert Tags to Dictionary

```python
tag_map = {
    tag["Key"]: tag["Value"]
    for tag in instance.get(
        "Tags",
        []
    )
}
```

Then:

```python
environment = tag_map.get(
    "Environment"
)

owner = tag_map.get(
    "Owner"
)
```

---

# 20. Get Instance Name

```python
name = tag_map.get(
    "Name",
    "unnamed"
)
```

The `Name` value is a tag, not a dedicated EC2 property.

---

# 21. EC2 Inventory Script

```python
import boto3

ec2 = boto3.client("ec2")

paginator = ec2.get_paginator(
    "describe_instances"
)

for page in paginator.paginate():

    for reservation in page.get(
        "Reservations",
        []
    ):

        for instance in reservation.get(
            "Instances",
            []
        ):

            tags = {
                tag["Key"]: tag["Value"]
                for tag in instance.get(
                    "Tags",
                    []
                )
            }

            print(
                instance.get("InstanceId"),
                tags.get("Name"),
                tags.get("Environment"),
                instance.get("InstanceType"),
                instance.get(
                    "PrivateIpAddress"
                ),
            )
```

---

# 22. Production Inventory Fields

A useful report contains:

```text
account
region
instance_id
name
state
instance_type
ami_id
vpc_id
subnet_id
private_ip
public_ip
environment
owner
launch_time
```

---

# 23. Inventory Output

Example:

```text
Instance ID       Name       Env    State     Type
-----------------------------------------------------
i-001             web-01     dev    running   t3.medium
i-002             api-01     dev    stopped   t3.small
i-003             db-01      prod   running   m6i.large
```

---

# 24. Count Instances by State

```python
from collections import Counter

states = Counter()

for instance in instances:

    state = instance.get(
        "State",
        {}
    ).get(
        "Name",
        "unknown"
    )

    states[state] += 1

print(states)
```

---

# 25. Count by Environment

```python
from collections import Counter

environments = Counter()

for instance in instances:

    tags = {
        tag["Key"]: tag["Value"]
        for tag in instance.get(
            "Tags",
            []
        )
    }

    environments[
        tags.get(
            "Environment",
            "unknown"
        )
    ] += 1
```

---

# 26. Start an EC2 Instance

```python
response = ec2.start_instances(
    InstanceIds=[
        instance_id
    ]
)
```

The API request being accepted does not necessarily mean the instance is already running.

---

# 27. Wait for Running State

```python
ec2.get_waiter(
    "instance_running"
).wait(
    InstanceIds=[
        instance_id
    ]
)
```

---

# 28. Verify Running State

```python
response = ec2.describe_instances(
    InstanceIds=[
        instance_id
    ]
)

state = response[
    "Reservations"
][0][
    "Instances"
][0][
    "State"
]["Name"]

print(state)
```

Use defensive access in production code.

---

# 29. Stop an EC2 Instance

```python
response = ec2.stop_instances(
    InstanceIds=[
        instance_id
    ]
)
```

---

# 30. Wait for Stopped State

```python
ec2.get_waiter(
    "instance_stopped"
).wait(
    InstanceIds=[
        instance_id
    ]
)
```

---

# 31. Reboot an EC2 Instance

```python
response = ec2.reboot_instances(
    InstanceIds=[
        instance_id
    ]
)
```

A reboot is different from stopping and starting an instance.

---

# 32. Stop vs Reboot

### Stop

```text
running
 ↓
stopped
```

Useful for:

```text
cost optimization
maintenance
scheduled shutdown
```

### Reboot

```text
running
 ↓
reboot
 ↓
running
```

Useful for:

```text
OS recovery
controlled restart
```

---

# 33. Terminate an EC2 Instance

```python
response = ec2.terminate_instances(
    InstanceIds=[
        instance_id
    ]
)
```

This is destructive.

Do not place termination into an ordinary automation workflow without strong controls.

---

# 34. Termination Safety

Before termination:

```text
validate account
validate region
validate environment
validate tags
validate protection
validate ownership
dry-run
approval
execute
verify
```

---

# 35. Never Select Production Broadly

Avoid logic such as:

```python
terminate_instances(
    all_instances
)
```

Use strict eligibility criteria.

---

# 36. Protected Resources

Use a tag:

```text
Protected=true
```

Then:

```python
if tag_map.get(
    "Protected"
) == "true":
    continue
```

---

# 37. Dry-Run Stop Script

Concept:

```python
if dry_run:
    print(
        f"Would stop {instance_id}"
    )
else:
    ec2.stop_instances(
        InstanceIds=[
            instance_id
        ]
    )
```

---

# 38. Production Guard Function

```python
def validate_target(
    account,
    region,
    environment,
    expected_account,
):

    if account != expected_account:
        raise RuntimeError(
            "Unexpected account"
        )

    if environment == "production":
        raise RuntimeError(
            "Production operation blocked"
        )

    if not region:
        raise ValueError(
            "Region required"
        )
```

Use environment-specific policy rather than blindly blocking all production use cases.

---

# 39. EC2 Scheduler

One of the most useful DevOps projects:

```text
start dev instances
during working hours

stop dev instances
outside working hours
```

---

# 40. Scheduler Architecture

```text
EventBridge / Scheduler
        ↓
Python Lambda / Job
        ↓
Boto3
        ↓
EC2
```

The Python process does not need to run continuously.

---

# 41. Scheduler Selection Criteria

Eligible instance:

```text
Environment=dev
Schedule=office-hours
ManagedBy=automation
Protected!=true
```

---

# 42. Start Scheduler

At start time:

```text
discover stopped eligible instances
 ↓
validate tags
 ↓
start
 ↓
wait
 ↓
report
```

---

# 43. Stop Scheduler

At shutdown time:

```text
discover running eligible instances
 ↓
validate tags
 ↓
stop
 ↓
wait
 ↓
report
```

---

# 44. Scheduler Safety

Never assume:

```text
dev = safe to stop
```

Some development resources may support:

```text
shared testing
data processing
release validation
temporary production-like testing
```

Use explicit opt-in tags.

---

# 45. Explicit Opt-In

A safer policy:

```text
AutoSchedule=true
Environment=dev
```

Only resources with `AutoSchedule=true` are modified.

---

# 46. EC2 Tagging

Boto3 can create or update tags.

```python
ec2.create_tags(
    Resources=[
        instance_id
    ],
    Tags=[
        {
            "Key": "ManagedBy",
            "Value": "PythonAutomation",
        }
    ],
)
```

---

# 47. Add Multiple Tags

```python
ec2.create_tags(
    Resources=[
        instance_id
    ],
    Tags=[
        {
            "Key": "Environment",
            "Value": "dev",
        },
        {
            "Key": "Owner",
            "Value": "platform",
        },
    ],
)
```

---

# 48. Tagging Is Not Free Metadata

Tags affect:

```text
automation selection
cost allocation
ownership
compliance
incident response
```

Design tag standards before automating around them.

---

# 49. Tag Compliance

Required tags might be:

```text
Environment
Owner
Project
CostCenter
ManagedBy
```

Report missing tags instead of automatically inventing values.

---

# 50. EC2 Public IP Audit

Find instances with:

```text
PublicIpAddress
```

and report:

```text
instance
name
environment
public IP
security groups
```

This can identify unexpected internet exposure.

---

# 51. Public IP Audit Logic

```python
if instance.get(
    "PublicIpAddress"
):
    print(
        instance["InstanceId"],
        instance["PublicIpAddress"]
    )
```

Do not automatically remove public access without understanding the architecture.

---

# 52. Private IP Inventory

```python
private_ip = instance.get(
    "PrivateIpAddress"
)
```

Useful for:

```text
inventory
incident response
network troubleshooting
```

---

# 53. VPC/Subnet Inventory

```python
vpc_id = instance.get(
    "VpcId"
)

subnet_id = instance.get(
    "SubnetId"
)
```

Useful for understanding placement.

---

# 54. Security Group Inventory

```python
security_groups = instance.get(
    "SecurityGroups",
    []
)

for group in security_groups:
    print(
        group.get("GroupId"),
        group.get("GroupName")
    )
```

---

# 55. IAM Instance Profile

```python
profile = instance.get(
    "IamInstanceProfile"
)

if profile:
    print(
        profile.get("Arn")
    )
```

This helps identify which IAM role/profile is attached.

---

# 56. AMI Information

```python
image_id = instance.get(
    "ImageId"
)
```

Useful for:

```text
image inventory
patch compliance
AMI lifecycle
deployment tracking
```

---

# 57. Launch Time

```python
launch_time = instance.get(
    "LaunchTime"
)
```

Useful for:

```text
age analysis
cost reports
deployment tracking
```

---

# 58. Find Old Instances

Concept:

```text
launch_time < threshold
```

Then report:

```text
instance
age
owner
environment
```

Do not terminate based solely on age.

---

# 59. EC2 Cost Candidate Report

Candidates can include:

```text
stopped instances
oversized instances
idle development instances
unused public IPs
unattached EBS volumes
old snapshots
```

Use read-only analysis first.

---

# 60. Stopped Instance Report

```python
response = ec2.describe_instances(
    Filters=[
        {
            "Name": "instance-state-name",
            "Values": ["stopped"],
        }
    ]
)
```

Report:

```text
instance
name
owner
environment
launch time
```

---

# 61. Long-Stopped Instance Analysis

Store timestamps and calculate:

```text
stopped duration
```

The EC2 API response can expose state transition information useful for analysis, depending on the response fields available.

---

# 62. EBS Volumes

An EC2 instance may use EBS volumes.

List volumes:

```python
response = ec2.describe_volumes()
```

Use a paginator for large environments.

---

# 63. Unattached EBS Volumes

An unattached volume typically has:

```text
Attachments=[]
```

Concept:

```python
if not volume.get(
    "Attachments"
):
    print(
        volume["VolumeId"]
    )
```

---

# 64. Unattached Volume Report

Report:

```text
volume ID
size
type
availability zone
create time
tags
```

Start with reporting before deletion.

---

# 65. EBS Cleanup Safety

Before deleting a volume:

```text
verify no attachment
verify tags
verify age
verify owner
verify backup policy
verify environment
verify protection
```

---

# 66. EBS Encryption Audit

Check:

```python
encrypted = volume.get(
    "Encrypted"
)
```

Report volumes that do not meet your organization's encryption policy.

Do not assume every workload has the same requirements.

---

# 67. EC2 AMI Creation

Boto3 can create an AMI from an instance:

```python
response = ec2.create_image(
    InstanceId=instance_id,
    Name=image_name,
    NoReboot=True,
)
```

`NoReboot=True` has consistency implications. Understand application state before choosing it.

---

# 68. AMI Creation Safety

Before creating an AMI:

```text
confirm instance
confirm application state
confirm disk consistency requirements
confirm naming
confirm tags
confirm retention policy
```

For databases, use database-aware backup mechanisms rather than assuming an EC2 image is a consistent database backup.

---

# 69. AMI Inventory

```python
response = ec2.describe_images(
    Owners=["self"]
)
```

For production, use filters and pagination where supported.

---

# 70. AMI Age Report

Useful fields:

```text
ImageId
Name
CreationDate
State
Architecture
RootDeviceType
Tags
```

---

# 71. AMI Retention

Example policy:

```text
keep latest N approved AMIs
```

But before deletion verify:

```text
launch templates
Auto Scaling groups
current deployments
disaster recovery
rollback requirements
```

---

# 72. AMI Deregistration

```python
ec2.deregister_image(
    ImageId=image_id
)
```

Deregistering an AMI does not automatically mean all associated snapshots are safely handled.

Treat snapshot cleanup as a separate controlled operation.

---

# 73. EC2 Metadata Reporting

Useful report:

```text
Account
Region
InstanceId
Name
State
Type
AMI
VPC
Subnet
Private IP
Public IP
Security Groups
IAM Profile
Launch Time
Environment
Owner
```

---

# 74. JSON Report

```python
import json

with open(
    "ec2-inventory.json",
    "w",
    encoding="utf-8",
) as file:

    json.dump(
        inventory,
        file,
        indent=2,
        default=str,
    )
```

---

# 75. CSV Report

Use Python's `csv` module or pandas when appropriate.

Example columns:

```text
instance_id
name
environment
state
instance_type
private_ip
public_ip
vpc_id
subnet_id
```

---

# 76. EC2 Health Check

A basic health report can collect:

```text
instance state
system status checks
instance status checks
```

---

# 77. Describe Instance Status

```python
response = ec2.describe_instance_status(
    IncludeAllInstances=True
)
```

Use a paginator for large fleets if supported.

---

# 78. Instance Status

Useful information includes:

```text
InstanceStatus
SystemStatus
Events
```

---

# 79. Status Check Automation

Possible workflow:

```text
discover unhealthy instances
 ↓
collect metadata
 ↓
identify owner/environment
 ↓
notify
```

Do not automatically reboot every unhealthy instance.

---

# 80. Scheduled Health Report

```text
EventBridge
 ↓
Python
 ↓
describe_instance_status
 ↓
unhealthy instances
 ↓
notification
```

---

# 81. EC2 Instance Events

AWS may report scheduled events.

Collect them for operational reporting.

Example fields may include:

```text
event type
event description
not-before
not-after
```

---

# 82. Incident Response Inventory

During an incident, Python can quickly collect:

```text
instance state
private IP
public IP
security groups
IAM profile
AMI
VPC/subnet
status checks
tags
```

This is useful for rapid triage.

---

# 83. EC2 + SSM

If Systems Manager is configured, operational automation can use SSM rather than relying on direct SSH.

Example client:

```python
ssm = boto3.client("ssm")
```

This can support:

```text
Run Command
patching workflows
inventory
operational commands
```

Use appropriate IAM permissions.

---

# 84. EC2 + SSM Architecture

```text
Python
 ↓
Boto3
 ↓
SSM
 ↓
EC2 managed instance
```

This can reduce dependence on inbound SSH access.

---

# 85. Run Command Concept

The SSM `send_command` API can execute approved commands on managed instances.

A production implementation must control:

```text
document
targets
commands
timeout
output
permissions
```

---

# 86. SSM Command Safety

Never allow arbitrary user input to become a shell command without strict validation.

Avoid:

```text
user input → shell command
```

without security controls.

---

# 87. EC2 + ECR

A deployment workflow may inspect ECR image metadata:

```text
ECR
 ↓
image
 ↓
deployment
 ↓
EC2
```

Boto3 can support image inventory and deployment checks.

---

# 88. EC2 + ALB

For applications behind an ALB:

```text
ALB
 ↓
Target Group
 ↓
EC2
```

Boto3 can inspect target health using the ELBv2 client.

---

# 89. EC2 + Terraform

Terraform can own:

```text
VPC
subnet
security group
IAM
EC2
ALB
```

Python/Boto3 can perform:

```text
inventory
health checks
audits
operational reports
```

Avoid manually changing Terraform-managed attributes unless the workflow is intentional.

---

# 90. EC2 + Jenkins

Example:

```text
Jenkins
 ↓
Python inventory
 ↓
EC2
 ↓
JSON report
 ↓
artifact
 ↓
notification
```

---

# 91. EC2 + GitHub Actions

Example:

```text
GitHub Actions
 ↓
OIDC
 ↓
AWS role
 ↓
Python/Boto3
 ↓
EC2 audit
```

---

# 92. EC2 + Prometheus/Grafana

Boto3 can collect AWS-side metadata, while Prometheus/Grafana can monitor the automation itself.

Metrics:

```text
ec2_inventory_runs_total
ec2_inventory_failures_total
ec2_instances_processed_total
ec2_operation_duration_seconds
```

---

# 93. EC2 + ELK

Structured logs can flow:

```text
Python
 ↓
stdout/file
 ↓
Logstash/agent
 ↓
Elasticsearch
 ↓
Kibana
```

Useful for:

```text
automation audit
failure analysis
operation history
```

---

# 94. EC2 Automation Project — Inventory CLI

Build:

```text
ec2ops.py
```

Commands:

```bash
python ec2ops.py inventory
```

```bash
python ec2ops.py running
```

```bash
python ec2ops.py stopped
```

```bash
python ec2ops.py health
```

```bash
python ec2ops.py tag-audit
```

---

# 95. CLI Architecture

```text
ec2ops.py
   |
   +-- inventory
   +-- running
   +-- stopped
   +-- health
   +-- tag-audit
   +-- public-ip-audit
```

---

# 96. Inventory Function

```python
def list_instances(ec2):

    paginator = ec2.get_paginator(
        "describe_instances"
    )

    instances = []

    for page in paginator.paginate():

        for reservation in page.get(
            "Reservations",
            []
        ):

            instances.extend(
                reservation.get(
                    "Instances",
                    []
                )
            )

    return instances
```

---

# 97. Normalize EC2 Data

Instead of passing the complete AWS response everywhere:

```python
def normalize_instance(instance):

    tags = {
        tag["Key"]: tag["Value"]
        for tag in instance.get(
            "Tags",
            []
        )
    }

    return {
        "instance_id": instance.get(
            "InstanceId"
        ),
        "name": tags.get("Name"),
        "environment": tags.get(
            "Environment"
        ),
        "owner": tags.get("Owner"),
        "state": instance.get(
            "State",
            {}
        ).get("Name"),
        "type": instance.get(
            "InstanceType"
        ),
        "private_ip": instance.get(
            "PrivateIpAddress"
        ),
        "public_ip": instance.get(
            "PublicIpAddress"
        ),
        "vpc_id": instance.get(
            "VpcId"
        ),
        "subnet_id": instance.get(
            "SubnetId"
        ),
    }
```

---

# 98. Why Normalize?

It separates:

```text
AWS response structure
```

from:

```text
your application/report structure
```

This makes reporting and testing easier.

---

# 99. EC2 Start Function

```python
def start_instance(
    ec2,
    instance_id,
    dry_run=True,
):

    if dry_run:
        return {
            "action": "start",
            "instance_id": instance_id,
            "dry_run": True,
        }

    response = ec2.start_instances(
        InstanceIds=[instance_id]
    )

    return response
```

Production code should additionally validate target account, region and resource policy.

---

# 100. EC2 Stop Function

```python
def stop_instance(
    ec2,
    instance_id,
    dry_run=True,
):

    if dry_run:
        return {
            "action": "stop",
            "instance_id": instance_id,
            "dry_run": True,
        }

    return ec2.stop_instances(
        InstanceIds=[instance_id]
    )
```

---

# 101. Safe Stop Workflow

```text
discover
 ↓
filter
 ↓
validate tags
 ↓
exclude protected
 ↓
dry-run
 ↓
approve
 ↓
stop
 ↓
wait
 ↓
verify
 ↓
report
```

---

# 102. EC2 Automation Error Handling

```python
from botocore.exceptions import ClientError

try:
    ec2.stop_instances(
        InstanceIds=[instance_id]
    )

except ClientError as exc:

    error = exc.response.get(
        "Error",
        {}
    )

    logger.error(
        "EC2 stop failed: %s",
        error.get("Code"),
    )

    raise
```

---

# 103. Do Not Retry Everything

Do not retry:

```text
InvalidInstanceID
AccessDenied
InvalidParameter
protected resource
wrong environment
```

These require correction or intentional handling.

---

# 104. EC2 API Retry Configuration

```python
from botocore.config import Config

config = Config(
    retries={
        "max_attempts": 5,
        "mode": "standard",
    },
)

ec2 = boto3.client(
    "ec2",
    config=config,
)
```

---

# 105. EC2 Operation Logging

```python
logger.info(
    "ec2_stop_requested",
    extra={
        "instance_id": instance_id,
        "environment": environment,
    },
)
```

Log safe operational metadata.

---

# 106. EC2 Operation Metrics

Useful metrics:

```text
ec2_start_total
ec2_stop_total
ec2_operation_failures_total
ec2_operation_duration_seconds
```

---

# 107. EC2 Automation Notification

After a scheduled operation:

```text
Started: 12
Stopped: 18
Skipped: 4
Failed: 1
```

Send through your notification automation.

---

# 108. EC2 Cost Optimization Workflow

```text
inventory
 ↓
identify candidates
 ↓
validate tags
 ↓
estimate impact
 ↓
report
 ↓
approval
 ↓
execute
 ↓
verify
```

Do not automate deletion merely because a resource looks unused.

---

# 109. EC2 Rightsizing Candidate Report

Possible indicators:

```text
instance type
CPU metrics from monitoring system
memory metrics from agent
network activity
business owner
environment
```

Boto3 alone does not provide complete utilization data. Combine it with your monitoring/observability stack.

---

# 110. EC2 Security Audit

Possible checks:

```text
public IP
security groups
IAM profile
subnet
VPC
tags
instance metadata configuration
```

Use the appropriate AWS APIs and security tooling for complete analysis.

---

# 111. IMDS Awareness

EC2 instances can use the Instance Metadata Service.

Security-conscious environments commonly enforce IMDSv2.

Automation should understand the instance metadata configuration when auditing EC2 security posture.

---

# 112. Instance Metadata Options

The EC2 API exposes metadata options for instances.

Audit policies such as:

```text
HttpTokens
HttpEndpoint
```

according to your organization's security standard.

---

# 113. EC2 Launch Templates

Modern environments often use:

```text
Launch Template
 ↓
Auto Scaling Group
 ↓
EC2
```

Avoid manually modifying individual instances when the desired configuration is owned by the launch template.

---

# 114. Auto Scaling Groups

The next level of EC2 automation involves:

```text
Auto Scaling
```

Use the Auto Scaling Boto3 client:

```python
autoscaling = boto3.client(
    "autoscaling"
)
```

---

# 115. ASG Automation

Useful operations include:

```text
describe groups
describe instances
desired capacity reporting
health reporting
```

Use caution with automated scaling changes.

---

# 116. EC2 Deployment Validation

After a deployment:

```text
check instance state
check target health
check application health
check deployment metadata
```

Boto3 can provide AWS infrastructure-side validation, while application checks should use appropriate application endpoints/monitoring.

---

# 117. EC2 + ALB Target Health

Use ELBv2:

```python
elbv2 = boto3.client(
    "elbv2"
)
```

Then inspect target health.

Concept:

```text
ALB
 ↓
target group
 ↓
EC2 target
 ↓
healthy/unhealthy
```

---

# 118. Production Deployment Gate

Example:

```text
deployment
 ↓
Boto3 checks EC2/ALB state
 ↓
healthy?
 ↓
yes → continue
no → stop pipeline
```

This can be integrated into Jenkins or GitHub Actions.

---

# 119. EC2 Incident Diagnostic Script

Create:

```bash
python ec2-diagnose.py \
    --instance-id i-xxxx
```

Collect:

```text
state
status checks
private IP
public IP
VPC
subnet
security groups
IAM profile
AMI
tags
launch time
```

---

# 120. Incident Output

```text
EC2 Diagnostic
==============

Instance: i-xxxx
Name: api-01
Environment: production
State: running
Type: t3.large

Private IP: 10.x.x.x
Public IP: none

VPC: vpc-xxxx
Subnet: subnet-xxxx

System Status: ok
Instance Status: ok
```

Do not expose sensitive infrastructure details in external notifications.

---

# 121. EC2 Automation Project — Tag Compliance

Required:

```text
Environment
Owner
Project
ManagedBy
```

Workflow:

```text
discover all instances
 ↓
normalize tags
 ↓
validate
 ↓
report missing tags
```

---

# 122. EC2 Automation Project — Public IP Audit

Workflow:

```text
discover instances
 ↓
find public IP
 ↓
collect security groups
 ↓
collect environment
 ↓
report
```

Use this as an audit, not automatic remediation.

---

# 123. EC2 Automation Project — Stopped Instance Audit

Workflow:

```text
find stopped
 ↓
calculate age
 ↓
collect owner/environment
 ↓
identify candidates
 ↓
notify
```

---

# 124. EC2 Automation Project — EBS Audit

Workflow:

```text
list volumes
 ↓
find unattached
 ↓
validate age/tags
 ↓
report
```

---

# 125. EC2 Automation Project — AMI Lifecycle

Workflow:

```text
list AMIs
 ↓
filter owned images
 ↓
sort by creation date
 ↓
protect approved/current
 ↓
report old images
 ↓
approval
 ↓
deregister
 ↓
handle snapshots
```

---

# 126. EC2 Automation Project — Dev Scheduler

Requirements:

```text
AutoSchedule=true
Environment=dev
Protected!=true
```

Workflow:

```text
schedule
 ↓
discover
 ↓
validate
 ↓
start/stop
 ↓
wait
 ↓
verify
 ↓
notify
```

---

# 127. EC2 Automation Project — Multi-Region Inventory

```text
regions.yaml
     ↓
Python
     ↓
Boto3
     ↓
EC2 per region
     ↓
normalize
     ↓
aggregate
     ↓
CSV/JSON
```

---

# 128. EC2 Automation Project — Multi-Account Inventory

```text
accounts.yaml
     ↓
AssumeRole
     ↓
region loop
     ↓
EC2 inventory
     ↓
aggregate
     ↓
report
```

---

# 129. Account Configuration

Concept:

```yaml
accounts:
  - name: dev
    role_arn: configured-role

  - name: staging
    role_arn: configured-role

  - name: production
    role_arn: configured-role
```

Do not commit sensitive credentials.

---

# 130. EC2 Automation With YAML

YAML can store:

```text
regions
required tags
protected environments
scheduling policy
```

Python should validate the configuration before execution.

---

# 131. Configuration Validation

Check:

```text
required keys
valid regions
valid environment names
valid boolean values
allowed actions
```

Fail before touching AWS if configuration is invalid.

---

# 132. EC2 Automation With Notifications

Connect:

```text
EC2 scheduler
 ↓
result summary
 ↓
Notification Automation
```

Example:

```text
EC2 Scheduler Result

Started: 8
Stopped: 15
Skipped: 3
Failed: 1
```

---

# 133. EC2 Automation With ELK

Logs:

```text
ec2_stop_requested
ec2_stop_success
ec2_stop_failed
ec2_scheduler_completed
```

Search and visualize in Kibana.

---

# 134. EC2 Automation With Prometheus

Expose:

```text
ec2_automation_runs_total
ec2_automation_failures_total
ec2_instances_processed_total
ec2_operation_duration_seconds
```

Grafana can show automation health.

---

# 135. EC2 Automation With Jenkins

Pipeline:

```text
checkout
 ↓
create virtualenv
 ↓
install dependencies
 ↓
run tests
 ↓
run dry-run
 ↓
approval
 ↓
run Boto3 operation
 ↓
publish report
 ↓
notify
```

---

# 136. EC2 Automation With GitHub Actions

Pipeline:

```text
checkout
 ↓
Python tests
 ↓
OIDC authentication
 ↓
Boto3
 ↓
dry-run/report
 ↓
approved operation
```

Use protected environments for sensitive workflows.

---

# 137. EC2 Automation With Terraform

Recommended responsibility:

```text
Terraform
 ↓
EC2 infrastructure lifecycle

Python/Boto3
 ↓
inventory
audit
operational actions
```

If Python changes infrastructure that Terraform owns, the next Terraform run may revert the change.

---

# 138. EC2 Automation With Ansible

Example:

```text
Boto3
 ↓
discover instances
 ↓
select targets
 ↓
Ansible
 ↓
configuration
```

This can be useful when inventory is dynamic.

---

# 139. Production Failure — Wrong Account

Detection:

```python
account = sts.get_caller_identity()[
    "Account"
]
```

If unexpected:

```text
STOP
```

Never continue with destructive operations.

---

# 140. Production Failure — AccessDenied

Investigate:

```text
IAM role
policy
resource policy
SCP
permissions boundary
```

Do not solve by immediately granting AdministratorAccess.

---

# 141. Production Failure — Throttling

Investigate:

```text
request rate
concurrency
polling
pagination
retry settings
```

Then reduce API pressure.

---

# 142. Production Failure — Instance Not Ready

Possible state:

```text
pending
```

Use:

```text
waiter
```

or bounded polling.

---

# 143. Production Failure — Instance ID Not Found

Possible reasons:

```text
wrong region
wrong account
terminated instance
invalid ID
```

First validate:

```text
account
region
ID
```

---

# 144. Production Failure — Duplicate Action

Example:

```text
stop requested twice
```

Design the script around current state:

```text
already stopped → no-op
```

---

# 145. Production Failure — Terminated Instance

If the resource is already gone:

```text
cleanup workflow
```

may treat it as an expected end state, depending on the operation.

---

# 146. Production Failure — Scheduler Stops Wrong Instance

Investigate:

```text
selection filters
tags
account
region
protected tag
configuration
script version
```

Use explicit opt-in tags to reduce risk.

---

# 147. Production Failure — Cost Spike

Investigate:

```text
new instances
instance type
region
test resources
unattended resources
```

Use tags to trace ownership.

---

# 148. Production Failure — Public Exposure

If an unexpected public IP is detected:

```text
identify instance
identify environment
identify security groups
identify owner
```

Then follow the approved security remediation process.

Do not automatically modify production networking without understanding dependencies.

---

# 149. Production Failure — Script Hangs

Check:

```text
connect timeout
read timeout
waiter
polling loop
thread pool
AWS service
network
```

Bound all external operations.

---

# 150. Production Failure — Partial Success

Report:

```text
success: 23
failed: 2
skipped: 5
```

Include failure reasons without leaking secrets.

---

# 151. EC2 Production Checklist

```text
[ ] Account validated
[ ] Region validated
[ ] IAM least privilege
[ ] Tag filters
[ ] Protected resources
[ ] Dry-run
[ ] Pagination
[ ] Retry strategy
[ ] Timeouts
[ ] Waiters
[ ] Idempotency
[ ] Logging
[ ] Metrics
[ ] Notifications
[ ] Tests
[ ] Cost controls
```

---

# 152. Interview — How Do You List All EC2 Instances?

**Answer:**

> I create an EC2 client, use the `describe_instances` paginator, iterate through reservations and instances, normalize the required fields, and return structured data rather than printing the raw AWS response everywhere.

---

# 153. Interview — Why Use a Paginator for EC2?

**Answer:**

> `describe_instances` can return paginated results. Using the paginator ensures the script processes all instances instead of only the first API response.

---

# 154. Interview — How Do You Find Only Running Instances?

**Answer:**

> I use the `instance-state-name` filter with the `running` value. Server-side filtering is preferable because AWS returns only the relevant resources.

---

# 155. Interview — How Do You Find Instances by Environment?

**Answer:**

> I use the EC2 tag filter, for example `tag:Environment=dev`, and combine it with additional safety tags such as `ManagedBy=PythonAutomation`.

---

# 156. Interview — How Do You Stop EC2 Instances Safely?

**Answer:**

> I validate account, region and environment, select instances using strict tags, exclude protected resources, support dry-run, execute the stop operation, wait for the stopped state when required, verify the result and report failures.

---

# 157. Interview — How Do You Start an Instance and Know It Is Running?

**Answer:**

> `start_instances` only initiates the operation. I use the `instance_running` waiter or bounded state polling and then verify the final state.

---

# 158. Interview — Stop vs Reboot?

**Answer:**

> Stop transitions an instance to the stopped state and can reduce compute charges. Reboot restarts the operating system while keeping the instance running afterward. The appropriate choice depends on the operational requirement.

---

# 159. Interview — How Do You Protect Production?

**Answer:**

> I validate the AWS account and region, require explicit environment and ownership tags, exclude protected resources, support dry-run, use least-privilege IAM, and place destructive operations behind approval controls.

---

# 160. Interview — How Would You Automate Dev Instance Shutdown?

**Answer:**

> I would use explicit opt-in tags such as `Environment=dev` and `AutoSchedule=true`, exclude `Protected=true`, run the workflow on a scheduler, stop eligible instances, wait for the desired state, verify and notify.

---

# 161. Interview — How Would You Find Unused EBS Volumes?

**Answer:**

> I would list volumes using a paginator, identify volumes with no attachments, collect age and tags, and generate a report. I would not automatically delete them until ownership, backup and retention policies are verified.

---

# 162. Interview — How Would You Audit Public EC2 Instances?

**Answer:**

> I would find instances with public IP addresses, collect their VPC, subnet, security groups and environment tags, and produce a security review report. Remediation would follow the approved security process.

---

# 163. Interview — How Would You Automate AMI Cleanup?

**Answer:**

> I would list owned AMIs, identify candidates using age and tags, protect current and approved images, verify launch templates and Auto Scaling dependencies, then perform controlled deregistration and snapshot cleanup according to policy.

---

# 164. Interview — How Do You Handle EC2 API Throttling?

**Answer:**

> I use pagination, server-side filtering, client reuse, bounded concurrency and Boto3 retry configuration with backoff. I also monitor API usage and avoid unnecessary polling.

---

# 165. Interview — How Do You Make EC2 Operations Idempotent?

**Answer:**

> I inspect the current state before changing it. For example, if an instance is already stopped, a stop operation becomes a no-op. For creation workflows, I use stable identifiers, tags and client tokens where supported.

---

# 166. Interview — What Happens If a Terraform-Managed EC2 Instance Is Changed With Boto3?

**Answer:**

> It depends on the attribute and Terraform configuration. If the Boto3 change creates drift in an attribute Terraform manages, a future Terraform plan may detect and revert it. I keep infrastructure ownership clear.

---

# 167. Interview — How Do You Authenticate EC2 Automation in Jenkins?

**Answer:**

> I prefer an IAM role or short-lived credential mechanism supported by the Jenkins environment. The role receives only the EC2 permissions required by the job, and credentials are never printed in logs.

---

# 168. Interview — How Do You Authenticate EC2 Automation in GitHub Actions?

**Answer:**

> I use GitHub Actions OIDC to assume an AWS IAM role and receive temporary credentials. The workflow uses Boto3 through the normal credential provider chain.

---

# 169. Interview — How Do You Monitor an EC2 Automation Job?

**Answer:**

> I record structured logs and metrics such as execution count, failures, duration and resources processed. I can send important failures to the notification stack and visualize automation health in Grafana.

---

# 170. Interview — How Would You Build an EC2 Diagnostic Tool?

**Answer:**

> I would accept an instance ID and region, validate the AWS account, then collect state, status checks, network information, security groups, IAM profile, AMI, tags and launch time. The tool would produce a structured report for incident response.

---

# 171. Interview — Why Should You Not Automatically Reboot Every Unhealthy Instance?

**Answer:**

> An unhealthy status can have different causes and the instance may host a critical workload. Automatic reboot can worsen an incident. I would first classify the failure and use a controlled remediation policy.

---

# 172. Interview — How Do You Avoid Stopping the Wrong Dev Instance?

**Answer:**

> I use explicit opt-in tags, strict environment filters, account and region validation, protected-resource checks and dry-run output before execution.

---

# 173. Interview — How Do You Handle Partial EC2 Operation Failure?

**Answer:**

> I track each instance independently and report successful, failed and skipped operations. If the operation is safe to continue, the workflow can process remaining resources; otherwise it fails fast.

---

# 174. Interview — What EC2 Automation Would You Build First?

**Answer:**

> I would start with a read-only EC2 inventory and health-reporting tool. Then I would add tag compliance, cost-candidate reporting, controlled scheduling and only afterward introduce destructive operations.

---

# 175. Final EC2 Automation Mental Model

```text
Discover
   ↓
Filter
   ↓
Identify
   ↓
Validate account/region
   ↓
Validate tags/environment
   ↓
Check current state
   ↓
Dry-run
   ↓
Approve
   ↓
Execute
   ↓
Wait
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

> **Automate EC2 operations based on explicit desired policy, not broad assumptions.**

The next module will build on this foundation with:

```text
03-S3-Automation.md
```

covering S3 buckets, objects, synchronization, lifecycle, versioning, encryption, compliance, backups, cleanup, reporting and production-safe automation.
