# Boto3 Cookbook

---

# Introduction

Boto3 is the official AWS SDK for Python. It allows developers and DevOps engineers to automate AWS services programmatically.

Common Use Cases

- Infrastructure Automation
- Cloud Resource Management
- Serverless Applications
- CI/CD Pipelines
- Scheduled Automation
- Cloud Operations

---

# Install Boto3

```bash
pip install boto3
```

Verify Installation

```bash
python -c "import boto3; print(boto3.__version__)"
```

---

# Import Boto3

```python
import boto3
```

---

# Create Default Session

```python
import boto3

session = boto3.Session()
```

---

# Create Session Using Profile

```python
import boto3

session = boto3.Session(
    profile_name="dev"
)
```

---

# Create Session for Specific Region

```python
import boto3

session = boto3.Session(
    region_name="ap-south-1"
)
```

---

# Create EC2 Client

```python
import boto3

ec2 = boto3.client("ec2")
```

---

# Create EC2 Resource

```python
import boto3

ec2 = boto3.resource("ec2")
```

---

# Difference Between Client and Resource

| Client | Resource |
|---------|----------|
| Low-level API | High-level API |
| Faster | More Pythonic |
| Complete AWS API coverage | Simplified object interface |
| Returns dictionaries | Returns resource objects |

---

# List AWS Regions

```python
import boto3

ec2 = boto3.client("ec2")

response = ec2.describe_regions()

for region in response["Regions"]:
    print(region["RegionName"])
```

---

# List Availability Zones

```python
import boto3

ec2 = boto3.client("ec2")

response = ec2.describe_availability_zones()

for az in response["AvailabilityZones"]:
    print(az["ZoneName"])
```

---

# EC2 Examples

---

# List EC2 Instances

```python
import boto3

ec2 = boto3.client("ec2")

response = ec2.describe_instances()

for reservation in response["Reservations"]:
    for instance in reservation["Instances"]:
        print(instance["InstanceId"])
```

---

# List Running Instances

```python
response = ec2.describe_instances(
    Filters=[
        {
            "Name": "instance-state-name",
            "Values": ["running"]
        }
    ]
)

for reservation in response["Reservations"]:
    for instance in reservation["Instances"]:
        print(instance["InstanceId"])
```

---

# List Stopped Instances

```python
response = ec2.describe_instances(
    Filters=[
        {
            "Name": "instance-state-name",
            "Values": ["stopped"]
        }
    ]
)
```

---

# Launch EC2 Instance

```python
response = ec2.run_instances(
    ImageId="ami-xxxxxxxx",
    InstanceType="t3.micro",
    MinCount=1,
    MaxCount=1,
    KeyName="my-key",
    SecurityGroupIds=["sg-xxxxxxxx"],
    SubnetId="subnet-xxxxxxxx"
)

print(response["Instances"][0]["InstanceId"])
```

---

# Stop EC2 Instance

```python
ec2.stop_instances(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)
```

---

# Start EC2 Instance

```python
ec2.start_instances(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)
```

---

# Reboot EC2 Instance

```python
ec2.reboot_instances(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)
```

---

# Terminate EC2 Instance

```python
ec2.terminate_instances(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)
```

---

# Describe Specific Instance

```python
response = ec2.describe_instances(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)

print(response)
```

---

# Create AMI

```python
response = ec2.create_image(
    InstanceId="i-0123456789abcdef0",
    Name="Production-AMI"
)

print(response["ImageId"])
```

---

# List AMIs

```python
response = ec2.describe_images(
    Owners=["self"]
)

for image in response["Images"]:
    print(image["ImageId"])
```

---

# Deregister AMI

```python
ec2.deregister_image(
    ImageId="ami-xxxxxxxx"
)
```

---

# Create Snapshot

```python
response = ec2.create_snapshot(
    VolumeId="vol-xxxxxxxx",
    Description="Production Backup"
)

print(response["SnapshotId"])
```

---

# List Snapshots

```python
response = ec2.describe_snapshots(
    OwnerIds=["self"]
)

for snapshot in response["Snapshots"]:
    print(snapshot["SnapshotId"])
```

---

# Delete Snapshot

```python
ec2.delete_snapshot(
    SnapshotId="snap-xxxxxxxx"
)
```

---

# List Volumes

```python
response = ec2.describe_volumes()

for volume in response["Volumes"]:
    print(volume["VolumeId"])
```

---

# Create Volume

```python
response = ec2.create_volume(
    AvailabilityZone="ap-south-1a",
    Size=20,
    VolumeType="gp3"
)

print(response["VolumeId"])
```

---

# Attach Volume

```python
ec2.attach_volume(
    VolumeId="vol-xxxxxxxx",
    InstanceId="i-xxxxxxxx",
    Device="/dev/xvdf"
)
```

---

# Detach Volume

```python
ec2.detach_volume(
    VolumeId="vol-xxxxxxxx"
)
```

---

# Delete Volume

```python
ec2.delete_volume(
    VolumeId="vol-xxxxxxxx"
)
```

---

# Wait Until Instance is Running

```python
waiter = ec2.get_waiter("instance_running")

waiter.wait(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)
```

---

# Wait Until Instance is Stopped

```python
waiter = ec2.get_waiter("instance_stopped")

waiter.wait(
    InstanceIds=[
        "i-0123456789abcdef0"
    ]
)
```

---

# Wait Until Snapshot Completes

```python
waiter = ec2.get_waiter("snapshot_completed")

waiter.wait(
    SnapshotIds=[
        "snap-xxxxxxxx"
    ]
)
```

---

# Error Handling

```python
import boto3
from botocore.exceptions import ClientError

ec2 = boto3.client("ec2")

try:
    response = ec2.describe_instances()
    print(response)

except ClientError as error:
    print(error)
```

---

# Best Practices

- Reuse Boto3 clients instead of creating them repeatedly.
- Use IAM Roles instead of embedding AWS credentials.
- Use waiters for long-running operations.
- Handle `ClientError` exceptions gracefully.
- Use pagination for large result sets.
- Prefer environment variables or named profiles for authentication.
- Avoid hardcoding resource IDs in production code.
- Log API responses for troubleshooting when appropriate.

---

# Summary

This section introduced Boto3 fundamentals, sessions, clients vs resources, EC2 instance management, AMIs, EBS volumes, snapshots, waiters, and exception handling. These examples provide the foundation for automating AWS infrastructure using Python.

---

