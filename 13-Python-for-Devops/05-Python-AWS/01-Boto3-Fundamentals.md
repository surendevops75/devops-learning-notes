# Boto3-Fundamentals

## Python for AWS DevOps — Boto3, AWS APIs, Authentication, Automation & Production Practices

Boto3 is the AWS SDK for Python. For a DevOps engineer, it provides programmatic access to AWS services such as EC2, S3, IAM, VPC, RDS, EKS, Lambda, ECR, ELB, SSM, STS, Secrets Manager and many others.

The goal of this module is not merely to learn:

```python
import boto3
```

The goal is to build **secure, reusable, observable, idempotent and production-safe AWS automation**.

---

# 1. Boto3 Architecture

```text
Python
  ↓
Boto3 Session
  ↓
Credential Provider Chain
  ↓
IAM Identity
  ↓
Service Client
  ↓
AWS API
  ↓
AWS Resource
```

A DevOps automation workflow often looks like:

```text
Jenkins / GitHub Actions / Cron
            ↓
        Python script
            ↓
           Boto3
            ↓
       AWS APIs
            ↓
 EC2 / S3 / EKS / RDS / IAM
```

---

# 2. Install Boto3

```bash
pip install boto3
```

Verify:

```bash
python -c "import boto3; print(boto3.__version__)"
```

For production, pin and manage dependencies through your normal dependency-management process.

Example:

```text
boto3
botocore
```

---

# 3. Boto3 Core Concepts

The three concepts to understand first are:

```text
Session
Client
Resource
```

A fourth concept that is extremely important in production is:

```text
Credential Provider Chain
```

---

# 4. Session

A session represents configuration such as:

```text
credentials
region
profile
configuration
```

Example:

```python
import boto3

session = boto3.Session()

print(session.region_name)
```

---

# 5. Session With Region

```python
import boto3

session = boto3.Session(
    region_name="ap-south-1"
)
```

---

# 6. Session With Profile

Useful for local development:

```python
import boto3

session = boto3.Session(
    profile_name="dev",
    region_name="ap-south-1",
)
```

Do not accidentally run production operations with a development or personal profile.

---

# 7. Client

A client provides low-level access to AWS service APIs.

```python
import boto3

ec2 = boto3.client(
    "ec2",
    region_name="ap-south-1",
)
```

Other examples:

```python
s3 = boto3.client("s3")
iam = boto3.client("iam")
rds = boto3.client("rds")
eks = boto3.client("eks")
lambda_client = boto3.client("lambda")
```

---

# 8. Resource

Resources provide a higher-level object-oriented interface for services that support them.

```python
import boto3

ec2 = boto3.resource("ec2")

for instance in ec2.instances.all():
    print(instance.id)
```

Not every AWS service has a resource interface.

For production automation, understanding clients is particularly important because clients expose the underlying AWS API operations consistently.

---

# 9. Client vs Resource

### Client

```python
ec2.describe_instances()
```

Good when you need:

```text
exact API operation
API filters
paginators
waiters
complete response structure
```

### Resource

```python
ec2.instances.all()
```

Good when the higher-level interface makes simple resource operations easier.

---

# 10. Credential Provider Chain

Boto3 can obtain credentials from supported sources such as:

```text
environment variables
AWS shared credentials/configuration
AWS profiles
EC2 instance role
ECS task role
container credentials
EKS pod identity mechanisms
other supported credential providers
```

The important DevOps principle is:

> **Do not put long-lived AWS credentials inside application source code.**

---

# 11. Never Hardcode Credentials

Never do:

```python
AWS_ACCESS_KEY_ID = "..."
AWS_SECRET_ACCESS_KEY = "..."
```

Problems include:

```text
Git exposure
log exposure
container-image exposure
developer-machine exposure
credential reuse
difficult rotation
```

---

# 12. Environment Variables

For local development/testing:

```bash
export AWS_ACCESS_KEY_ID="..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_DEFAULT_REGION="ap-south-1"
```

Prefer temporary credentials whenever possible.

---

# 13. AWS CLI Profiles

Local developers can configure profiles:

```bash
aws configure --profile dev
```

Then:

```python
session = boto3.Session(
    profile_name="dev"
)
```

Use explicit profiles when working across multiple AWS accounts.

---

# 14. IAM Roles

Production workloads should generally use IAM roles.

Examples:

```text
EC2 → instance profile / IAM role
ECS → task role
EKS → AWS-supported pod identity
GitHub Actions → OIDC → IAM role
```

This avoids embedding permanent keys.

---

# 15. GitHub Actions + Boto3

A modern workflow is:

```text
GitHub Actions
      ↓
OIDC
      ↓
AWS IAM Role
      ↓
temporary credentials
      ↓
Python
      ↓
Boto3
```

This is preferable to storing long-lived AWS access keys when the environment supports OIDC federation.

---

# 16. Jenkins + Boto3

Depending on the Jenkins deployment, prefer:

```text
IAM role
short-lived credentials
approved credential integration
```

The Python process should never print credentials.

---

# 17. EC2 + Boto3

An EC2 workload can use an IAM role through its instance profile.

Then application code can simply use:

```python
import boto3

s3 = boto3.client("s3")
```

Boto3 obtains credentials through the AWS credential provider chain.

---

# 18. EKS + Boto3

A pod can use an AWS-supported pod identity mechanism.

Architecture:

```text
EKS Pod
  ↓
Pod identity
  ↓
IAM Role
  ↓
temporary credentials
  ↓
Boto3
```

Grant only the permissions required by the application.

---

# 19. STS GetCallerIdentity

Before sensitive automation, determine which identity is being used:

```python
import boto3

sts = boto3.client("sts")

identity = sts.get_caller_identity()

print(identity["Account"])
print(identity["Arn"])
```

This is extremely useful for safety checks.

---

# 20. Production Account Guard

For destructive automation:

```python
EXPECTED_ACCOUNT = "configured-safely"

sts = boto3.client("sts")

account = sts.get_caller_identity()["Account"]

if account != EXPECTED_ACCOUNT:
    raise RuntimeError(
        "Unexpected AWS account"
    )
```

Keep the expected account in environment-specific configuration rather than publishing real account identifiers in source.

---

# 21. Region Awareness

Most AWS resources are regional.

```python
ec2 = boto3.client(
    "ec2",
    region_name="ap-south-1",
)
```

Do not assume a resource exists in every region.

---

# 22. Multi-Region Automation

```python
regions = [
    "ap-south-1",
    "ap-southeast-1",
]

for region in regions:
    ec2 = boto3.client(
        "ec2",
        region_name=region,
    )

    print(
        region,
        ec2.describe_instances()
    )
```

For production, handle regional failures independently and produce a clear report.

---

# 23. IAM Least Privilege

If an inventory script only needs:

```text
Describe
List
Get
```

do not give it:

```text
AdministratorAccess
```

A production role should contain only required permissions.

---

# 24. Read-Only Automation

Examples:

```text
EC2 inventory
S3 compliance report
EKS cluster report
RDS health report
tag audit
```

should use read-only permissions whenever possible.

---

# 25. Destructive Automation

High-risk operations include:

```text
terminate EC2
delete S3 objects
delete S3 bucket
delete IAM policy
delete security group
delete VPC
delete RDS
```

These require stronger safeguards.

---

# 26. Dry Run

Provide:

```bash
python cleanup.py --dry-run
```

Output:

```text
Would terminate:
i-123
i-456
```

No modification occurs.

---

# 27. Environment Guard

A destructive script should know:

```text
account
region
environment
```

Example:

```python
if environment == "production":
    raise RuntimeError(
        "Production requires approved workflow"
    )
```

For real systems, use a controlled approval mechanism rather than relying only on a command-line flag.

---

# 28. Tags

Tags are fundamental to AWS automation.

Common tags:

```text
Name
Environment
Application
Project
Owner
CostCenter
ManagedBy
Protected
```

---

# 29. Tag-Based Selection

Example rule:

```text
Environment=dev
ManagedBy=PythonAutomation
Protected!=true
```

Only resources matching the intended policy should be modified.

---

# 30. Convert AWS Tags to a Dictionary

```python
tag_map = {
    tag["Key"]: tag["Value"]
    for tag in instance.get(
        "Tags",
        []
    )
}

environment = tag_map.get(
    "Environment"
)
```

---

# 31. Server-Side Filtering

Prefer AWS API filters where available.

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

This reduces unnecessary data transfer and processing.

---

# 32. Boto3 Response Structure

AWS responses are normally Python dictionaries and lists.

```python
response = ec2.describe_instances()

reservations = response.get(
    "Reservations",
    []
)
```

Avoid assuming optional fields always exist.

---

# 33. Safe Nested Access

```python
state = instance.get(
    "State",
    {}
)

state_name = state.get(
    "Name"
)
```

This is safer than directly indexing every nested field.

---

# 34. Describe EC2 Instances

```python
import boto3

ec2 = boto3.client("ec2")

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
            instance.get("InstanceId"),
            instance.get("InstanceType"),
            instance.get("PrivateIpAddress"),
        )
```

---

# 35. Pagination

A critical production concept:

> One AWS API call does not necessarily return every resource.

Many list/describe operations paginate.

---

# 36. Boto3 Paginator

Use a paginator when available:

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
                instance["InstanceId"]
            )
```

---

# 37. Why Pagination Matters

A script may appear correct with:

```text
10 EC2 instances
```

but fail silently in an account containing:

```text
500 EC2 instances
```

because it processed only the first response.

Always check pagination requirements.

---

# 38. Manual Pagination

Some APIs may require handling continuation tokens explicitly.

Concept:

```python
token = None

while True:

    params = {}

    if token:
        params["NextToken"] = token

    response = client.some_operation(
        **params
    )

    process(response)

    token = response.get(
        "NextToken"
    )

    if not token:
        break
```

Use the built-in paginator when one is available.

---

# 39. Waiters

Some AWS operations are asynchronous.

Example:

```text
start EC2
 ↓
initializing
 ↓
running
```

Boto3 waiters can poll for known resource states.

```python
waiter = ec2.get_waiter(
    "instance_running"
)

waiter.wait(
    InstanceIds=[instance_id]
)
```

---

# 40. Waiter Safety

Consider:

```text
timeout
retry behavior
failure states
API rate
```

Never build an unbounded polling loop.

---

# 41. Client Configuration

Botocore configuration can control retries and timeouts.

```python
from botocore.config import Config

config = Config(
    retries={
        "max_attempts": 5,
        "mode": "standard",
    },
    connect_timeout=10,
    read_timeout=30,
)

ec2 = boto3.client(
    "ec2",
    config=config,
)
```

Tune values for the actual workload.

---

# 42. AWS Exceptions

Use `ClientError` for AWS service errors.

```python
from botocore.exceptions import ClientError

try:
    response = ec2.describe_instances()

except ClientError as exc:
    error = exc.response.get(
        "Error",
        {}
    )

    print(
        error.get("Code")
    )
```

---

# 43. Do Not Swallow Exceptions

Avoid:

```python
try:
    ...
except Exception:
    pass
```

This can turn a failed production operation into a false success.

---

# 44. Classify AWS Errors

Useful categories:

```text
authentication
authorization
validation
throttling
network
service failure
resource state
business rule
```

---

# 45. AccessDenied

Typical meaning:

```text
IAM or resource authorization problem
```

Investigate:

```text
identity
IAM policy
resource policy
SCP
permissions boundary
session policy
```

---

# 46. Throttling

Possible causes:

```text
too many API calls
too much concurrency
large polling frequency
```

Solutions:

```text
pagination
backoff
bounded concurrency
batching
caching
```

---

# 47. Retryable vs Permanent Errors

Retry candidates may include:

```text
throttling
temporary service errors
some network failures
```

Do not blindly retry:

```text
AccessDenied
InvalidParameter
invalid resource
```

---

# 48. Exponential Backoff

Conceptually:

```text
attempt 1
 ↓
wait
attempt 2
 ↓
wait longer
attempt 3
 ↓
wait longer
```

Add jitter when appropriate to reduce synchronized retries.

---

# 49. Idempotency

Production automation should be safe to run repeatedly.

Bad:

```text
every run creates another resource
```

Better:

```text
check current state
 ↓
desired state?
 ↓
yes → no-op
no → change
```

---

# 50. Client Tokens

Some AWS APIs support idempotency/client tokens for create operations.

Use them where supported, especially when retries could otherwise create duplicate resources.

---

# 51. Resource State

Always distinguish:

```text
API request accepted
```

from:

```text
resource fully ready
```

Example:

```text
RunInstances
 ↓
API success
 ↓
initializing
 ↓
running
```

---

# 52. Eventual Consistency

AWS systems can have propagation delays.

A resource may be created successfully but not immediately visible through another API.

Use:

```text
waiters
bounded polling
backoff
```

where necessary.

---

# 53. Timeouts

Every network automation should consider:

```text
connect timeout
read timeout
overall operation timeout
```

Without bounds, a stuck API call can hold a worker indefinitely.

---

# 54. Client Reuse

Avoid:

```python
for resource in resources:
    client = boto3.client("ec2")
```

Prefer:

```python
ec2 = boto3.client("ec2")

for resource in resources:
    process(ec2, resource)
```

Reuse clients appropriately.

---

# 55. Controlled Concurrency

AWS API operations are I/O-bound, so concurrency can help, but excessive concurrency causes throttling.

Use bounded workers:

```python
from concurrent.futures import ThreadPoolExecutor

with ThreadPoolExecutor(
    max_workers=5
) as executor:
    results = executor.map(
        process_resource,
        resources,
    )
```

Tune worker count from measured behavior and service limits.

---

# 56. AWS API Quotas

Before increasing concurrency, understand:

```text
service API limits
account limits
operation limits
burst behavior
```

A fast script that gets throttled is not a production-ready script.

---

# 57. Batch Operations

Use batch APIs where supported.

Benefits:

```text
fewer API requests
lower latency
lower throttling risk
```

Still respect API limits and request-size constraints.

---

# 58. Caching

If the same AWS metadata is requested repeatedly:

```text
query once
 ↓
cache
 ↓
reuse
```

Do not cache rapidly changing state longer than is appropriate.

---

# 59. Cross-Account Automation

Common architecture:

```text
CI/CD
  ↓
Management/Tools Account
  ↓
STS AssumeRole
  ↓
Target Account
  ↓
Boto3
```

---

# 60. AssumeRole

```python
import boto3

sts = boto3.client("sts")

response = sts.assume_role(
    RoleArn=role_arn,
    RoleSessionName="devops-automation",
)

credentials = response["Credentials"]
```

Use the returned temporary credentials to create a session for the target account.

---

# 61. Why AssumeRole?

Benefits:

```text
temporary credentials
centralized access
cross-account automation
auditability
less credential distribution
```

---

# 62. Cross-Account Safety

Before modifying resources:

```text
validate account ID
validate assumed role
validate region
validate environment
```

---

# 63. Service Clients Used in DevOps

Common Boto3 clients:

```python
boto3.client("ec2")
boto3.client("s3")
boto3.client("iam")
boto3.client("vpc")
boto3.client("rds")
boto3.client("eks")
boto3.client("lambda")
boto3.client("ecr")
boto3.client("elbv2")
boto3.client("ssm")
boto3.client("secretsmanager")
boto3.client("sts")
boto3.client("sqs")
boto3.client("sns")
```

---

# 64. ECR

```python
ecr = boto3.client("ecr")
```

Useful for:

```text
repository inventory
image metadata
image cleanup workflows
deployment reports
```

---

# 65. ALB / ELBv2

```python
elbv2 = boto3.client("elbv2")
```

Useful for:

```text
load balancer inventory
listeners
target groups
target health
```

This is especially relevant to EKS architectures using AWS Load Balancer Controller/ALB ingress.

---

# 66. SSM

```python
ssm = boto3.client("ssm")
```

Useful for:

```text
Parameter Store
Run Command
automation workflows
managed instance operations
```

---

# 67. Secrets Manager

```python
secrets = boto3.client(
    "secretsmanager"
)
```

Use it for approved secret retrieval instead of putting passwords or API keys in source.

---

# 68. SQS

```python
sqs = boto3.client("sqs")
```

Useful for:

```text
asynchronous automation
work queues
notification workers
event-driven processing
```

---

# 69. SNS

```python
sns = boto3.client("sns")
```

Useful for:

```text
AWS-native notifications
fan-out
event publishing
```

---

# 70. CloudFormation

```python
cloudformation = boto3.client(
    "cloudformation"
)
```

Useful for:

```text
stack status
resource inventory
deployment reporting
```

---

# 71. Route 53

```python
route53 = boto3.client("route53")
```

Useful for:

```text
hosted zone inventory
DNS records
operational checks
```

---

# 72. Boto3 + Terraform

Keep responsibilities clear.

Terraform:

```text
desired infrastructure state
```

Boto3:

```text
operational automation
audits
reports
custom workflows
```

Do not bypass Terraform's ownership of infrastructure without a deliberate reason.

---

# 73. Boto3 + Ansible

Ansible:

```text
configuration management
orchestration
```

Boto3:

```text
direct AWS API access
custom Python logic
```

They can work together.

---

# 74. Boto3 + Jenkins

Example:

```text
Jenkins
 ↓
Python
 ↓
Boto3
 ↓
AWS
 ↓
report
 ↓
notification
```

Use cases:

```text
deployment validation
resource inventory
cleanup
health checks
```

---

# 75. Boto3 + GitHub Actions

```text
GitHub Actions
 ↓
OIDC
 ↓
IAM Role
 ↓
Python/Boto3
 ↓
AWS
```

This is a strong modern CI/CD authentication pattern.

---

# 76. Boto3 + ArgoCD

For Kubernetes:

```text
Git
 ↓
ArgoCD
 ↓
EKS
```

Boto3 can support AWS-side operational tasks such as:

```text
cluster metadata
ECR inspection
ALB inspection
AWS resource audits
```

Use the Kubernetes API/client for Kubernetes-native resources when appropriate.

---

# 77. Logging Boto3 Automation

Use Python logging:

```python
import logging

logger = logging.getLogger(
    __name__
)

logger.info(
    "Starting EC2 inventory"
)
```

Useful operational events:

```text
start
account
region
operation
resource count
failure
completion
duration
```

---

# 78. Never Log Credentials

Never log:

```text
access keys
secret keys
session tokens
secret values
```

Be careful with full AWS API responses because some responses can contain sensitive operational information.

---

# 79. Structured Automation Logs

A useful conceptual record:

```json
{
  "event": "ec2_inventory_complete",
  "account": "redacted",
  "region": "ap-south-1",
  "resource_count": 42
}
```

Use your organization's approved structured logging format.

---

# 80. Separate AWS Access From Business Logic

Recommended:

```text
aws/
  clients.py
  ec2.py
  s3.py

logic/
  inventory.py
  compliance.py

output/
  reports.py
  notifications.py

cli.py
```

This improves:

```text
testing
reuse
maintenance
```

---

# 81. Reusable Function

```python
def get_instances(ec2_client):

    paginator = ec2_client.get_paginator(
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

# 82. Return Data Instead of Printing Everywhere

Prefer:

```python
instances = get_instances(ec2)
```

Then:

```text
print
JSON
CSV
HTML
notification
```

can all consume the same data.

---

# 83. CLI Integration

Use `argparse`:

```python
import argparse

parser = argparse.ArgumentParser()

parser.add_argument(
    "--region",
    required=True,
)

parser.add_argument(
    "--dry-run",
    action="store_true",
)

args = parser.parse_args()
```

---

# 84. Production CLI Safety

Useful options:

```text
--profile
--region
--environment
--dry-run
--output
--confirm
```

For destructive commands, use stronger approval controls than a simple boolean when required.

---

# 85. Configuration

Keep environment-specific settings outside business logic:

```yaml
environment: dev
region: ap-south-1

required_tags:
  - Environment
  - Owner
  - Project
```

Never put secrets in ordinary configuration files.

---

# 86. Testing

Boto3 code should have:

```text
unit tests
mock/stub tests
integration tests
```

---

# 87. Botocore Stubber

`botocore.stub.Stubber` can provide deterministic AWS responses.

Concept:

```python
from botocore.stub import Stubber

stubber = Stubber(ec2)

stubber.add_response(
    "describe_instances",
    expected_response,
)

with stubber:
    response = ec2.describe_instances()
```

This allows testing without real AWS resources.

---

# 88. Integration Testing

Use a dedicated AWS test account when possible.

Workflow:

```text
create
 ↓
verify
 ↓
modify
 ↓
verify
 ↓
cleanup
```

Never make ordinary tests depend on production resources.

---

# 89. Cost-Aware Testing

AWS tests can create billable resources.

Use:

```text
small resources
short lifetimes
automatic cleanup
test tags
dedicated account
```

---

# 90. Cleanup Pattern

Where safe:

```python
try:
    create_resources()
    run_test()

finally:
    cleanup_resources()
```

Make cleanup idempotent too.

---

# 91. Test Resource Tags

Example:

```text
ManagedBy=AutomationTest
TestRun=<unique-id>
Environment=test
```

Cleanup can target only those resources.

---

# 92. Inventory Project

A useful first Boto3 project:

```text
aws-inventory.py
```

Collect:

```text
EC2
S3
RDS
EKS
Lambda
ECR
ALB
```

Output:

```text
JSON
CSV
terminal report
```

---

# 93. Inventory Architecture

```text
CLI
 ↓
Boto3 Session
 ↓
STS account check
 ↓
Region loop
 ↓
Service clients
 ↓
Paginators
 ↓
Normalization
 ↓
Report
 ↓
Notification
```

---

# 94. Inventory Output

Example:

```text
AWS Inventory
=============

EC2
Running: 24
Stopped: 8

S3
Buckets: 18

RDS
Instances: 6

EKS
Clusters: 2

Lambda
Functions: 31

ECR
Repositories: 14
```

---

# 95. Tag Compliance Project

Find resources missing:

```text
Environment
Owner
Project
ManagedBy
```

Workflow:

```text
discover
 ↓
extract tags
 ↓
validate
 ↓
report
 ↓
notify
```

---

# 96. Dev EC2 Scheduler Project

Find instances with:

```text
Environment=dev
```

and stop them outside approved hours.

Safety rules:

```text
Protected=true → never stop
Environment=production → never stop
ManagedBy mismatch → skip
wrong account → stop
```

Use an AWS-native scheduler such as EventBridge where appropriate, with Python/Boto3 executing the resource operation.

---

# 97. EBS Audit Project

Find:

```text
unattached EBS volumes
```

Report:

```text
volume ID
size
type
region
age
tags
```

Start with reporting. Automated deletion requires explicit policy and safeguards.

---

# 98. AMI Cleanup Project

Identify old AMIs using:

```text
age
tags
owner
usage
```

Before deletion, check:

```text
associated snapshots
launch templates
Auto Scaling groups
current deployments
```

---

# 99. Cross-Account Inventory Project

Architecture:

```text
Management
   ↓
AssumeRole
   ↓
Dev
   ↓
Staging
   ↓
Production
```

For every account:

```text
validate account
validate role
scan regions
collect inventory
```

Keep this project read-only initially.

---

# 100. AWS Compliance Reporter

Checks can include:

```text
required tags
public access configuration
encryption configuration
backup configuration
old resources
```

Output:

```text
PASS
WARN
FAIL
```

---

# 101. AWS Operations CLI

Example:

```bash
python awsops.py inventory
```

```bash
python awsops.py ec2-report
```

```bash
python awsops.py tag-audit
```

```bash
python awsops.py health-report
```

```bash
python awsops.py cost-candidates
```

---

# 102. Boto3 + Notification Automation

Connect this section with:

```text
04-Python-Automation/
07-Notification-Automation.md
```

Example:

```text
AWS audit
 ↓
unattached EBS found
 ↓
report
 ↓
Slack notification
```

---

# 103. Boto3 + Log Automation

Connect with:

```text
06-Log-Automation.md
```

Example:

```text
Boto3 job
 ↓
structured logs
 ↓
ELK
 ↓
Grafana/alerting context
```

---

# 104. Boto3 + Monitoring

Long-running automation can expose metrics such as:

```text
aws_api_calls_total
aws_api_failures_total
automation_runs_total
automation_duration_seconds
resources_processed_total
```

---

# 105. Production AWS Automation Architecture

```text
Jenkins / GitHub Actions / EventBridge
                    |
                    v
              Python Runner
                    |
          +---------+---------+
          |                   |
       Boto3              Logging
          |                   |
          v                   v
       AWS APIs              ELK
          |
  +-------+-------+-------+
  |       |       |       |
 EC2     S3      RDS     EKS
  |                       |
  +---------+-------------+
            |
         Reports
            |
       Notifications
```

---

# 106. AWS API Checklist

Before using an API operation, verify:

```text
operation name
required parameters
response fields
pagination
waiter
permissions
quotas
exceptions
regional scope
idempotency
```

Use current AWS documentation when implementing a production integration.

---

# 107. AWS Service Scope

Know whether the service/resource is:

```text
regional
global
account-level
resource-level
```

Do not build multi-region logic blindly.

---

# 108. Resource ARN

Many AWS resources use an ARN:

```text
arn:aws:service:region:account:resource
```

Use the exact ARN format required by the relevant service.

---

# 109. IAM Policy Layers

An authorization decision can be affected by:

```text
identity policy
resource policy
SCP
permissions boundary
session policy
explicit deny
```

An IAM `Allow` does not override an applicable explicit deny.

---

# 110. Boto3 and IAM Troubleshooting

When you get:

```text
AccessDenied
```

first identify:

```python
sts.get_caller_identity()
```

Then inspect:

```text
role
policy
resource policy
SCP
permissions boundary
```

---

# 111. Partial Failure

Suppose:

```text
100 resources
98 succeeded
2 failed
```

The automation should report:

```text
successful
failed
skipped
```

Do not report the whole operation as successful if important work failed.

---

# 112. Fail-Fast vs Continue

Use fail-fast when:

```text
operation is transactional
security condition fails
wrong account detected
critical prerequisite missing
```

Continue-and-report may be better when:

```text
independent resources are being inventoried
```

Choose based on the operation.

---

# 113. Destructive Operation Safety

A strong workflow:

```text
load config
 ↓
validate credentials
 ↓
validate account
 ↓
validate region
 ↓
validate environment
 ↓
discover resources
 ↓
validate tags
 ↓
dry-run
 ↓
approval
 ↓
execute
 ↓
verify
 ↓
report
```

---

# 114. Verify After Change

Do not assume:

```text
API returned success
```

means the final desired state is correct.

Use:

```text
waiter
status check
follow-up API call
```

where appropriate.

---

# 115. Cost Awareness

Potentially costly automation mistakes:

```text
EC2 accidentally created
RDS left running
NAT gateway created
large EBS volumes created
test resources not deleted
multi-region resources created
```

Use tags and automatic cleanup.

---

# 116. Security Checklist

```text
[ ] No hardcoded credentials
[ ] IAM least privilege
[ ] Temporary credentials
[ ] Account validation
[ ] Region validation
[ ] Secret manager
[ ] No secrets in logs
[ ] Input validation
[ ] Protected resources
[ ] Dry-run
[ ] Audit trail
```

---

# 117. Reliability Checklist

```text
[ ] Pagination
[ ] Retries
[ ] Backoff
[ ] Timeouts
[ ] Waiters
[ ] Idempotency
[ ] Eventual-consistency handling
[ ] Partial-failure handling
[ ] Controlled concurrency
[ ] API quota awareness
```

---

# 118. Maintainability Checklist

```text
[ ] Modular code
[ ] Type hints
[ ] Docstrings
[ ] Configuration separation
[ ] Unit tests
[ ] Integration tests
[ ] CLI
[ ] Structured logging
[ ] Dependency management
```

---

# 119. Interview — What Is Boto3?

**Answer:**

> Boto3 is the AWS SDK for Python. It allows Python applications and automation scripts to interact with AWS services through their APIs.

---

# 120. Interview — Client vs Resource?

**Answer:**

> A client provides low-level access to AWS service APIs and exposes API operations directly. A resource provides a higher-level object-oriented interface for services that support it. I commonly use clients for precise API automation.

---

# 121. Interview — How Does Boto3 Find Credentials?

**Answer:**

> Boto3 uses the AWS credential provider chain. Depending on the environment, credentials can come from environment variables, profiles, instance roles, container credentials, EKS identity mechanisms and other supported providers.

---

# 122. Interview — How Do You Secure Boto3 Credentials?

**Answer:**

> I avoid hardcoded keys and prefer IAM roles and temporary credentials. In CI/CD, I prefer workload identity such as OIDC when available. Secrets are stored in an approved secret-management system.

---

# 123. Interview — How Do You Authenticate Boto3 in EKS?

**Answer:**

> I use an AWS-supported EKS pod identity mechanism to associate the workload with an IAM role. Boto3 then obtains temporary credentials through its standard provider chain.

---

# 124. Interview — How Do You Authenticate Boto3 in GitHub Actions?

**Answer:**

> I use GitHub Actions OIDC federation to assume an AWS IAM role and receive temporary credentials, avoiding long-lived access keys.

---

# 125. Interview — How Do You Handle Pagination?

**Answer:**

> I check whether the operation is paginated and use a Boto3 paginator when available. This ensures the script processes the complete result set.

---

# 126. Interview — What Are Waiters?

**Answer:**

> Waiters poll an AWS resource until it reaches a known state. They are useful for asynchronous operations such as waiting for an EC2 instance to become running.

---

# 127. Interview — How Do You Handle Boto3 Errors?

**Answer:**

> I catch `ClientError`, inspect the AWS error code, classify the failure as retryable or permanent, log useful context, and fail explicitly when the operation cannot safely continue.

---

# 128. Interview — How Do You Handle Throttling?

**Answer:**

> I use appropriate SDK retry configuration, exponential backoff, jitter where useful, pagination, batching and controlled concurrency. I avoid blindly retrying permanent errors.

---

# 129. Interview — How Do You Make AWS Automation Idempotent?

**Answer:**

> I check current state before making changes, use stable identifiers and tags, use AWS-supported client tokens where available, and make repeated executions result in the same desired state instead of duplicate resources.

---

# 130. Interview — How Do You Protect Production?

**Answer:**

> I validate the AWS account, region, environment, resource tags and IAM role. Destructive commands support dry-run and approval controls, and protected resources are explicitly excluded.

---

# 131. Interview — How Do You Handle Cross-Account AWS Automation?

**Answer:**

> I use STS AssumeRole to obtain temporary credentials in each target account. I validate the target account ID before operations and use separate least-privilege roles.

---

# 132. Interview — Boto3 or Terraform?

**Answer:**

> Terraform is designed for declarative infrastructure desired-state management. Boto3 is better suited to operational automation, discovery, reporting and custom AWS workflows. I keep Terraform as the source of truth for infrastructure it manages.

---

# 133. Interview — Boto3 or Ansible?

**Answer:**

> Ansible is useful for configuration management and orchestration, while Boto3 provides direct Python access to AWS APIs. They complement each other rather than necessarily replacing each other.

---

# 134. Interview — How Do You Avoid API Throttling?

**Answer:**

> I use server-side filtering, pagination, batching, client reuse, controlled concurrency, caching where appropriate, and retry/backoff policies.

---

# 135. Interview — What Is STS AssumeRole?

**Answer:**

> AssumeRole allows an identity to obtain temporary credentials for an IAM role. It is commonly used for secure cross-account automation.

---

# 136. Interview — How Do You Know Which AWS Account a Script Is Using?

**Answer:**

> I call STS GetCallerIdentity and verify the returned account ID and principal before sensitive operations.

---

# 137. Interview — What Is the Credential Provider Chain?

**Answer:**

> It is the mechanism Boto3 uses to locate AWS credentials from supported sources such as environment variables, profiles, instance roles, container credentials and other identity providers.

---

# 138. Interview — How Do You Handle Eventual Consistency?

**Answer:**

> I recognize that an accepted AWS operation may not immediately appear through another API. I use waiters or bounded polling with backoff where appropriate and verify final state.

---

# 139. Interview — How Do You Test Boto3?

**Answer:**

> I use mocks or botocore Stubber for unit tests and a dedicated AWS test account for controlled integration tests. Production resources are not used for ordinary tests.

---

# 140. Interview — How Would You Build an AWS Inventory Tool?

**Answer:**

> I would create a read-only Boto3 session, validate account and regions, use service clients and paginators, normalize resource metadata, export JSON/CSV, and optionally send a summary notification.

---

# 141. Interview — Why Are AWS Tags Important?

**Answer:**

> Tags provide ownership and classification metadata. They allow automation to select resources safely by environment, application, owner and management policy.

---

# 142. Interview — How Would You Automate Dev EC2 Shutdown?

**Answer:**

> I would select instances using strict tags such as Environment=dev and ManagedBy=automation, exclude protected resources, validate the account and region, support dry-run, and stop only approved resources.

---

# 143. Interview — How Would You Handle an AccessDenied Error?

**Answer:**

> I would first identify the caller using STS, then inspect the IAM role and policies along with resource policies, SCPs, permissions boundaries and any applicable explicit denies.

---

# 144. Interview — What If the API Returns Only Part of the Resources?

**Answer:**

> I would check pagination. If a paginator exists, I would use it rather than assuming the first response contains the complete result.

---

# 145. Interview — What If an AWS API Call Hangs?

**Answer:**

> I would configure connect and read timeouts, inspect network connectivity and retry behavior, and check whether the code is stuck in an unbounded waiter or polling loop.

---

# 146. Interview — What If a Create Operation Is Retried?

**Answer:**

> I consider whether the operation is idempotent and use a client token where supported. I also verify resource state before retrying so a successful first attempt does not result in duplicate resources.

---

# 147. Interview — Why Validate AWS Account Before Deletion?

**Answer:**

> The same script can accidentally receive credentials for another account. Account validation prevents a development cleanup script from deleting production resources.

---

# 148. Interview — Why Use Dry Run?

**Answer:**

> Dry run provides visibility into what a script intends to change without actually changing resources. It is especially valuable for cleanup and cost-optimization automation.

---

# 149. Interview — How Do You Monitor Boto3 Automation?

**Answer:**

> I expose or record metrics such as run count, API failures, processing duration and resource counts. I use structured logs and notifications for important failures.

---

# 150. Interview — How Do You Integrate Boto3 With Your DevOps Stack?

**Answer:**

> I use Python/Boto3 for AWS operational automation, Jenkins or GitHub Actions to execute workflows, Terraform for declarative infrastructure, ArgoCD for Kubernetes GitOps, Prometheus/Grafana for metrics, and ELK for logs. Each tool has a clear responsibility.

---

# 151. Final Production Boto3 Mental Model

```text
Authentication
     ↓
IAM least privilege
     ↓
Account validation
     ↓
Region validation
     ↓
Client/session
     ↓
API filtering
     ↓
Pagination
     ↓
Waiters
     ↓
Retries/backoff
     ↓
Timeouts
     ↓
Idempotency
     ↓
Business logic
     ↓
Verification
     ↓
Logging
     ↓
Metrics
     ↓
Notification
```

The difficult part of Boto3 is not making an AWS API call.

The production challenge is making the automation:

```text
secure
idempotent
observable
retry-safe
cost-aware
environment-safe
scalable
maintainable
```

These principles will be applied to the next modules:

```text
02-EC2-Automation.md
03-S3-Automation.md
04-IAM-Automation.md
05-VPC-Automation.md
06-RDS-Automation.md
07-EKS-Automation.md
08-Lambda-Automation.md
09-AWS-Automation-Projects.md
```
