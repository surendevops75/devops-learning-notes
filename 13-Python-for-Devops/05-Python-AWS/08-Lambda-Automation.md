# Lambda-Automation

## Python for AWS DevOps — Lambda Discovery, Versions, Aliases, IAM, Environment Configuration, Layers, Concurrency, Event Sources, Deployment, Monitoring & Production Automation

AWS Lambda is a serverless compute service that runs code without requiring you to manage servers.

For DevOps engineers, Lambda automation is useful for:

```text
resource inventory
function health checks
configuration audits
version management
alias management
deployment automation
IAM review
environment-variable audits
layer inventory
concurrency checks
event-source discovery
log configuration
CloudWatch monitoring
cost/capacity analysis
scheduled operations
cross-account audits
```

The key principle is:

> **Lambda is serverless, not operations-free.**

You still need to manage:

```text
IAM
configuration
deployment
versions
dependencies
timeouts
memory
concurrency
observability
security
reliability
cost
```

---

# 1. Lambda Mental Model

Typical architecture:

```text
Event Source
     |
     v
Lambda Function
     |
     +-- Execution Role
     |
     +-- Environment Variables
     |
     +-- Layers
     |
     +-- VPC (optional)
     |
     +-- CloudWatch Logs
     |
     +-- Metrics
     |
     +-- Version
     |
     +-- Alias
     |
     +-- Concurrency
```

Common event sources:

```text
API Gateway
EventBridge
S3
SQS
SNS
DynamoDB Streams
Kinesis
ALB
```

---

# 2. Lambda Boto3 Client

```python
import boto3

lambda_client = boto3.client(
    "lambda",
    region_name="ap-south-1",
)
```

Lambda is regional.

---

# 3. Validate AWS Identity

For production automation:

```python
sts = boto3.client("sts")

identity = sts.get_caller_identity()

print(identity["Account"])
print(identity["Arn"])
```

Validate the expected account before performing mutations.

---

# 4. List Lambda Functions

```python
paginator = lambda_client.get_paginator(
    "list_functions"
)

for page in paginator.paginate():

    for function in page.get(
        "Functions",
        []
    ):
        print(
            function["FunctionName"]
        )
```

Always use pagination for complete inventory.

---

# 5. Lambda Function Metadata

Useful fields:

```text
FunctionName
FunctionArn
Runtime
Role
Handler
CodeSize
Description
Timeout
MemorySize
LastModified
Version
Environment
Layers
State
StateReason
LastUpdateStatus
Architectures
PackageType
EphemeralStorage
TracingConfig
```

---

# 6. Describe a Lambda Function

```python
response = lambda_client.get_function(
    FunctionName="orders-function"
)

configuration = response[
    "Configuration"
]

print(
    configuration["Runtime"]
)

print(
    configuration["Timeout"]
)

print(
    configuration["MemorySize"]
)
```

---

# 7. Lambda Function State

Inspect:

```python
print(
    configuration.get(
        "State"
    )
)
```

Possible states include:

```text
Pending
Active
Inactive
Failed
```

Do not automatically treat every non-Active state as an incident without considering lifecycle context.

---

# 8. Last Update Status

```python
print(
    configuration.get(
        "LastUpdateStatus"
    )
)
```

This helps identify deployment/configuration update problems.

---

# 9. Lambda Runtime Inventory

```python
runtime = configuration.get(
    "Runtime"
)

print(runtime)
```

Create reports such as:

```text
python3.x
nodejs...
java...
```

Use AWS's current supported-runtime lifecycle policy when determining whether a runtime requires remediation.

---

# 10. Runtime Compliance

A Python audit can compare:

```text
actual runtime
        vs
approved runtime list
```

Example:

```text
orders-prod
Runtime: approved
PASS

legacy-function
Runtime: policy exception
REVIEW
```

Do not hardcode "latest" as a runtime requirement.

---

# 11. Lambda Architecture

```python
print(
    configuration.get(
        "Architectures"
    )
)
```

Common architecture choices include:

```text
x86_64
arm64
```

Architecture affects:

```text
runtime compatibility
layers
native dependencies
cost
performance
```

---

# 12. Lambda Package Type

Lambda supports:

```text
Zip
Image
```

Inspect:

```python
print(
    configuration.get(
        "PackageType"
    )
)
```

Container-image functions have different deployment and dependency considerations.

---

# 13. Lambda Handler

For Zip-based functions:

```python
print(
    configuration.get(
        "Handler"
    )
)
```

Typical:

```text
app.lambda_handler
```

A handler points to:

```text
module.function
```

---

# 14. Lambda Timeout

```python
timeout = configuration.get(
    "Timeout"
)

print(timeout)
```

Timeout should reflect actual workload behavior.

Avoid simply increasing timeout to hide slow dependencies.

---

# 15. Lambda Memory

```python
memory = configuration.get(
    "MemorySize"
)

print(memory)
```

Memory allocation also influences Lambda CPU/network resources.

---

# 16. Lambda Ephemeral Storage

```python
storage = configuration.get(
    "EphemeralStorage",
    {}
)

print(
    storage.get(
        "Size"
    )
)
```

This is the temporary `/tmp` storage available to the function.

Do not confuse it with persistent storage.

---

# 17. Lambda Environment Variables

Configuration may contain:

```python
environment = configuration.get(
    "Environment",
    {}
)

variables = environment.get(
    "Variables",
    {}
)
```

Do not print values.

Safe audit:

```python
for key in variables:
    print(key)
```

---

# 18. Environment Variable Security

Never log:

```text
password
token
API key
secret
private key
database credential
```

Instead report:

```text
key exists
key missing
```

---

# 19. Environment Variable Audit

Example:

```text
Required:
DB_HOST
DB_NAME
LOG_LEVEL

Function:
orders-prod

DB_HOST: PASS
DB_NAME: PASS
LOG_LEVEL: PASS
```

The actual values remain hidden.

---

# 20. Lambda IAM Execution Role

```python
role_arn = configuration.get(
    "Role"
)

print(role_arn)
```

This is the execution role used by the function.

---

# 21. Lambda Execution Role

The execution role can provide access to:

```text
CloudWatch Logs
S3
DynamoDB
SQS
SNS
Secrets Manager
KMS
other AWS APIs
```

depending on policy.

---

# 22. Least Privilege

Avoid policies like:

```text
Action: *
Resource: *
```

when they are unnecessary.

Prefer:

```text
specific actions
specific resources
```

---

# 23. Lambda Role Audit

A security automation can flag:

```text
AdministratorAccess
wildcard actions
wildcard resources
unused permissions
unexpected trust relationships
```

Use IAM Access Analyzer and CloudTrail evidence where appropriate rather than assuming static policy inspection alone proves actual usage.

---

# 24. Lambda Versions

Lambda versions provide immutable function code/configuration snapshots.

List versions:

```python
paginator = lambda_client.get_paginator(
    "list_versions_by_function"
)

for page in paginator.paginate(
    FunctionName="orders-function"
):

    for version in page.get(
        "Versions",
        []
    ):
        print(
            version["Version"]
        )
```

---

# 25. `$LATEST`

Lambda has:

```text
$LATEST
```

which represents the unpublished mutable version.

Published versions are immutable.

---

# 26. Published Versions

Production deployments commonly use:

```text
version 17
version 18
version 19
```

rather than invoking `$LATEST` directly.

---

# 27. Lambda Aliases

Aliases provide stable names such as:

```text
dev
staging
prod
```

List aliases:

```python
response = lambda_client.list_aliases(
    FunctionName="orders-function"
)

for alias in response.get(
    "Aliases",
    []
):
    print(
        alias["Name"],
        alias["FunctionVersion"]
    )
```

---

# 28. Alias Architecture

```text
prod alias
    |
    v
version 19
```

A deployment can move:

```text
prod
 ↓
version 20
```

without changing the function name used by clients.

---

# 29. Alias Safety

Before moving a production alias:

```text
validate version
validate tests
validate permissions
validate configuration
validate health
approval
```

---

# 30. Weighted Alias Routing

Lambda aliases can support traffic shifting.

Concept:

```text
prod
 |
 +-- version 19 → 90%
 |
 +-- version 20 → 10%
```

Useful for:

```text
canary deployment
gradual rollout
```

---

# 31. Canary Deployment

Typical:

```text
Deploy version 20
       ↓
5% traffic
       ↓
observe
       ↓
25%
       ↓
50%
       ↓
100%
```

Automate only with strong health gates.

---

# 32. Provisioned Concurrency

Provisioned concurrency keeps execution environments initialized.

Useful when:

```text
cold-start latency
```

is important.

Do not enable it blindly because it can increase cost.

---

# 33. Reserved Concurrency

Reserved concurrency can:

```text
limit function concurrency
reserve capacity
protect downstream systems
```

Inspect:

```python
response = lambda_client.get_function_concurrency(
    FunctionName="orders-function"
)

print(
    response.get(
        "ReservedConcurrentExecutions"
    )
)
```

---

# 34. Concurrency Audit

Check:

```text
reserved concurrency
provisioned concurrency
function traffic
downstream capacity
```

A very high Lambda concurrency can overload:

```text
RDS
APIs
third-party systems
```

---

# 35. Lambda and RDS

Important production relationship:

```text
Lambda
   |
   +---- many concurrent executions
              |
              v
             RDS
```

Without connection management, Lambda concurrency can exhaust database connections.

---

# 36. Lambda Concurrency Guardrail

For database-connected functions:

```text
Lambda concurrency
       ↓
DB connection pool
       ↓
RDS max connections
```

Set limits based on actual capacity.

---

# 37. Lambda Event Source Mappings

For stream/queue integrations:

```python
paginator = lambda_client.get_paginator(
    "list_event_source_mappings"
)

for page in paginator.paginate():

    for mapping in page.get(
        "EventSourceMappings",
        []
    ):
        print(
            mapping.get(
                "UUID"
            ),
            mapping.get(
                "State"
            )
        )
```

---

# 38. Event Source Mapping

Common sources:

```text
SQS
Kinesis
DynamoDB Streams
```

Inspect:

```text
EventSourceArn
FunctionArn
State
BatchSize
MaximumBatchingWindowInSeconds
Enabled
```

---

# 39. SQS + Lambda

Architecture:

```text
Producer
   ↓
SQS
   ↓
Lambda
   ↓
Application
```

Lambda polls the queue through an event source mapping.

---

# 40. SQS Lambda Health

Check:

```text
mapping state
queue depth
batch size
function errors
function duration
visibility timeout
DLQ
```

---

# 41. SQS Visibility Timeout

A critical relationship:

```text
SQS visibility timeout
        >
Lambda processing time
```

should be designed intentionally.

Otherwise messages may become visible again while still processing.

---

# 42. Lambda Batch Size

Batch size affects:

```text
throughput
latency
failure behavior
downstream load
```

Do not maximize batch size without testing.

---

# 43. Partial Batch Failure

For supported event-source patterns, partial batch response can prevent successfully processed messages from being retried unnecessarily.

The implementation must match the event source and Lambda integration behavior.

---

# 44. Lambda + S3

S3 can trigger Lambda through supported event notification mechanisms.

Architecture:

```text
S3 Object Created
       ↓
Lambda
       ↓
Processing
```

Avoid creating recursive triggers such as:

```text
S3 upload
 ↓
Lambda
 ↓
same bucket upload
 ↓
Lambda
```

unless intentionally designed with a safe prefix/filter.

---

# 45. Lambda + EventBridge

Architecture:

```text
EventBridge
     ↓
Lambda
```

Useful for:

```text
scheduled jobs
AWS events
custom events
automation
```

---

# 46. Lambda + SNS

Architecture:

```text
SNS
 ↓
Lambda
```

Useful for:

```text
fan-out
notifications
event processing
```

---

# 47. Lambda + API Gateway

Typical:

```text
Client
 ↓
API Gateway
 ↓
Lambda
 ↓
service/data
```

Python can audit:

```text
function
API integration
permissions
timeouts
aliases
```

---

# 48. Lambda Invocation Permission

A Lambda function may require resource-based permissions for event sources.

Inspect:

```python
response = lambda_client.get_policy(
    FunctionName="orders-function"
)

print(
    response["Policy"]
)
```

Treat the returned policy as sensitive configuration and avoid exposing it unnecessarily.

---

# 49. Lambda Resource Policy

Audit:

```text
principal
action
source ARN
source account
condition
```

Look for overly broad invocation permissions.

---

# 50. Lambda Permission Example

Desired pattern:

```text
specific service
+
specific source
+
specific function
```

Avoid:

```text
Principal = *
```

unless explicitly required and strongly constrained.

---

# 51. Lambda Layers

List:

```python
paginator = lambda_client.get_paginator(
    "list_layers"
)

for page in paginator.paginate():

    for layer in page.get(
        "Layers",
        []
    ):
        print(
            layer["LayerName"]
        )
```

Layers can package:

```text
shared libraries
runtime dependencies
utilities
```

---

# 52. Function Layers

```python
for layer in configuration.get(
    "Layers",
    []
):
    print(
        layer.get(
            "Arn"
        )
    )
```

Audit:

```text
layer ARN
version
compatibility
ownership
```

---

# 53. Layer Version Management

Old layers can accumulate.

Inventory:

```text
layer
version
created date
compatible runtimes
compatible architectures
```

Delete only after dependency analysis.

---

# 54. Lambda Code Size

```python
print(
    configuration.get(
        "CodeSize"
    )
)
```

Large packages can affect:

```text
deployment time
cold-start behavior
storage
```

Optimize dependencies where appropriate.

---

# 55. Lambda Container Images

For image-based Lambda:

```text
ECR
 ↓
Lambda
```

Audit:

```text
image URI
image digest
ECR repository
image scanning
architecture
```

Use immutable image references/digests where the deployment process requires strong reproducibility.

---

# 56. Lambda VPC Configuration

Inspect:

```python
vpc = configuration.get(
    "VpcConfig",
    {}
)

print(
    vpc.get("VpcId")
)

print(
    vpc.get("SubnetIds")
)

print(
    vpc.get("SecurityGroupIds")
)
```

---

# 57. Lambda VPC Networking

A VPC-connected Lambda may need:

```text
private subnets
route tables
security groups
NAT Gateway
VPC endpoints
```

depending on what AWS services it needs to access.

---

# 58. Lambda VPC Common Mistake

A function inside private subnets does not automatically have Internet access.

For outbound Internet access, the architecture may require:

```text
Lambda
 ↓
private subnet
 ↓
route
 ↓
NAT Gateway
 ↓
Internet
```

or appropriate VPC endpoints for AWS services.

---

# 59. Lambda VPC Troubleshooting

If a VPC Lambda times out:

```text
subnet
 ↓
route table
 ↓
NAT/VPC endpoint
 ↓
security group
 ↓
DNS
 ↓
destination
```

Check the entire path.

---

# 60. Lambda Security Groups

Audit:

```text
outbound rules
inbound rules
source/destination
```

For Lambda, security-group inbound rules often do not need to allow unsolicited inbound traffic because invocation occurs through AWS service integrations rather than direct network connections.

---

# 61. Lambda Dead Letter Queue

For supported asynchronous invocation configurations, inspect DLQ settings.

A DLQ can capture failed invocation events.

Do not confuse:

```text
DLQ
```

with:

```text
SQS event-source failure handling
```

which can use different retry/DLQ mechanisms.

---

# 62. Lambda Async Failure Handling

Asynchronous invocation can involve:

```text
retry attempts
maximum event age
destination
DLQ
```

Review these as part of reliability configuration.

---

# 63. Lambda Destinations

Destinations can route invocation results:

```text
success
failure
```

to supported targets.

Useful for:

```text
audit
event processing
failure workflows
```

---

# 64. Lambda Timeout vs Retry

A long timeout combined with aggressive retries can increase:

```text
execution cost
queue delay
downstream pressure
```

Design timeout and retry together.

---

# 65. Lambda Monitoring

Important metrics include:

```text
Invocations
Errors
Duration
Throttles
ConcurrentExecutions
IteratorAge
DeadLetterErrors
DestinationDeliveryFailures
```

Metric availability depends on the event source and configuration.

---

# 66. Lambda Error Rate

Basic concept:

```text
error rate =
errors / invocations
```

Use a meaningful time window and traffic volume before declaring an incident.

---

# 67. Lambda Duration

Monitor:

```text
average
p50
p95
p99
maximum
```

Long-tail latency can matter more than average latency.

---

# 68. Lambda Throttles

Throttling may occur because of:

```text
reserved concurrency
account concurrency
provisioned concurrency
burst/capacity limits
```

Investigate the actual concurrency configuration.

---

# 69. Lambda Logs

Lambda normally writes logs to CloudWatch Logs when the execution role has appropriate permissions.

Typical log group:

```text
/aws/lambda/function-name
```

Python automation can inventory log groups and retention configuration.

---

# 70. CloudWatch Log Retention

Do not leave logs indefinitely without a retention policy.

A log audit can report:

```text
function
log group
retention
```

Then compare against policy.

---

# 71. Lambda Log Group Discovery

```python
logs = boto3.client(
    "logs",
    region_name="ap-south-1",
)

response = logs.describe_log_groups(
    logGroupNamePrefix="/aws/lambda/"
)

for group in response.get(
    "logGroups",
    []
):
    print(
        group["logGroupName"],
        group.get(
            "retentionInDays"
        )
    )
```

Use pagination for complete production inventory.

---

# 72. Lambda Log Retention Automation

A safe workflow:

```text
discover
 ↓
check policy
 ↓
report missing retention
 ↓
approval/policy automation
 ↓
apply
 ↓
verify
```

Do not automatically change all existing log groups without understanding compliance requirements.

---

# 73. Lambda Tracing

Lambda can integrate with distributed tracing services.

Inspect:

```python
print(
    configuration.get(
        "TracingConfig"
    )
)
```

Enable according to your organization's observability architecture.

Do not assume every project needs the same tracing configuration.

---

# 74. Lambda Environment + Secrets

Avoid:

```text
PASSWORD=plaintext
```

Prefer:

```text
Secrets Manager
or
SSM Parameter Store
```

with IAM-controlled access.

---

# 75. Secrets Manager Pattern

```text
Lambda
 ↓
IAM role
 ↓
Secrets Manager
 ↓
secret
```

The application retrieves the secret at runtime.

Do not print the retrieved value.

---

# 76. Parameter Store Pattern

```text
Lambda
 ↓
IAM role
 ↓
SSM Parameter Store
 ↓
parameter
```

Use encryption for sensitive values as required.

---

# 77. Lambda Deployment

Common deployment models:

```text
Zip package
Container image
```

CI/CD can build and publish the artifact, then update Lambda.

---

# 78. Zip Deployment

Typical:

```text
source
 ↓
dependencies
 ↓
zip
 ↓
S3 or direct upload
 ↓
Lambda
 ↓
publish version
 ↓
update alias
```

---

# 79. Container Deployment

Typical:

```text
source
 ↓
Docker build
 ↓
security scan
 ↓
ECR
 ↓
Lambda image update
 ↓
publish/version strategy
```

---

# 80. Lambda + Jenkins

Example:

```text
Git
 ↓
Jenkins
 ↓
test
 ↓
SonarQube
 ↓
Trivy
 ↓
build
 ↓
package/image
 ↓
Lambda
 ↓
publish version
 ↓
update alias
```

The exact security stages should match the project's DevSecOps pipeline.

---

# 81. Lambda + GitHub Actions

Typical:

```text
GitHub
 ↓
GitHub Actions
 ↓
test
 ↓
security scanning
 ↓
build
 ↓
AWS authentication
 ↓
deploy
 ↓
publish version
 ↓
update alias
```

Use short-lived AWS credentials, such as OIDC-based role assumption, instead of long-lived access keys.

---

# 82. Lambda + Terraform

Terraform can manage:

```text
function
IAM role
permissions
layers
event source
aliases
versions
log groups
```

Python should not silently change Terraform-managed configuration.

Use Python for:

```text
audit
health
verification
operational workflows
```

unless the ownership model explicitly says otherwise.

---

# 83. Lambda Deployment Verification

After deployment:

```text
function update complete
 ↓
version published
 ↓
alias points to expected version
 ↓
test invocation
 ↓
metrics healthy
 ↓
logs healthy
```

---

# 84. Lambda Test Invocation

Boto3 can invoke a function for controlled tests:

```python
response = lambda_client.invoke(
    FunctionName="orders-function",
    InvocationType="RequestResponse",
    Payload=b'{"healthcheck": true}',
)

print(
    response["StatusCode"]
)
```

Only send test payloads supported by the function contract.

---

# 85. Production Smoke Test

A safe smoke test should:

```text
use non-destructive payload
avoid real customer transactions
validate expected response
check logs
check errors
```

---

# 86. Lambda Version Cleanup

Old versions can accumulate.

A cleanup process:

```text
list versions
 ↓
exclude $LATEST
 ↓
exclude versions used by aliases
 ↓
exclude protected versions
 ↓
apply retention
 ↓
dry-run
 ↓
delete
```

Never delete versions blindly.

---

# 87. Delete Function Version

```python
lambda_client.delete_function(
    FunctionName="orders-function",
    Qualifier="17",
)
```

This is destructive.

Verify that no alias or workflow depends on the version.

---

# 88. Lambda Alias Update

Concept:

```python
lambda_client.update_alias(
    FunctionName="orders-function",
    Name="prod",
    FunctionVersion="20",
)
```

Use only after deployment validation.

---

# 89. Alias Rollback

If version 20 fails:

```text
prod
 ↓
version 20
 ↓
failure
 ↓
prod → version 19
```

Rollback is fast because versions are immutable.

---

# 90. Canary Rollback

For weighted deployments:

```text
version 19 → 95%
version 20 → 5%
```

If metrics degrade:

```text
version 20 → 0%
version 19 → 100%
```

Automated rollback should have clearly defined thresholds.

---

# 91. Lambda Provisioned Concurrency Automation

For latency-sensitive functions:

```text
publish version
 ↓
configure provisioned concurrency
 ↓
warm
 ↓
shift alias
```

Monitor cost and utilization.

---

# 92. Lambda Cost Factors

Important factors include:

```text
invocations
duration
memory
architecture
provisioned concurrency
ephemeral storage
data transfer
event source
```

Cost optimization must consider application latency and reliability.

---

# 93. Lambda Rightsizing

If duration is high:

```text
increase memory
```

may sometimes reduce execution time enough to lower total cost.

Do not assume:

```text
less memory = less cost
```

without measuring.

---

# 94. Lambda Concurrency Cost

Provisioned concurrency can introduce additional cost.

Use it when the latency requirement justifies it.

---

# 95. Lambda Cost Audit

Report:

```text
function
memory
architecture
invocations
duration
errors
throttles
provisioned concurrency
```

Then identify candidates for optimization.

---

# 96. Lambda Tagging

Recommended tags may include:

```text
Environment
Application
Owner
CostCenter
ManagedBy
Criticality
```

Use the organization's required tagging policy.

---

# 97. Lambda Tag Audit

```python
response = lambda_client.list_tags(
    Resource=function_arn
)

tags = response.get(
    "Tags",
    {}
)

print(
    tags.get("Environment")
)
```

Flag missing required tags.

---

# 98. Lambda Inventory Report

Example:

```text
Function: orders-prod
Runtime: Python
Architecture: arm64
Memory: 1024 MB
Timeout: 30 sec
State: Active
Version: 21
Alias: prod → 21
VPC: yes
Encryption: policy-compliant
```

---

# 99. Lambda Security Audit

Check:

```text
execution role
resource policy
environment configuration
VPC
security groups
layers
runtime
package
tags
log retention
```

---

# 100. Lambda Production Health Report

Example:

```text
Lambda Health
=============

Function: orders-prod
State: ACTIVE
Last Update: SUCCESS
Runtime: APPROVED
Errors: NORMAL
Throttles: 0
Duration: NORMAL
Alias: prod → 21
```

---

# 101. Lambda Scheduled Audit

A Python automation can run:

```text
daily
 ↓
list functions
 ↓
check runtimes
 ↓
check IAM
 ↓
check public/resource permissions
 ↓
check log retention
 ↓
check aliases
 ↓
check versions
 ↓
report
```

---

# 102. Lambda Multi-Account Audit

Architecture:

```text
Central Automation
       |
       +-- Dev
       +-- Staging
       +-- Production
```

For each account:

```text
AssumeRole
 ↓
GetCallerIdentity
 ↓
Lambda inventory
 ↓
compliance
 ↓
report
```

---

# 103. Lambda Multi-Region Audit

```python
regions = [
    "ap-south-1",
    "ap-southeast-1",
]

for region in regions:

    client = boto3.client(
        "lambda",
        region_name=region,
    )

    # inventory
```

Use an approved region list.

---

# 104. Lambda Cross-Account Security

Never assume:

```text
same function name
```

means:

```text
same application
```

Always identify:

```text
account
region
ARN
environment
```

---

# 105. Lambda Error Handling

```python
from botocore.exceptions import ClientError

try:

    response = lambda_client.get_function(
        FunctionName=function_name
    )

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

# 106. Lambda Retry Configuration

AWS APIs can experience transient failures.

Use Boto3 retry configuration:

```python
from botocore.config import Config

config = Config(
    retries={
        "max_attempts": 5,
        "mode": "standard",
    }
)

lambda_client = boto3.client(
    "lambda",
    config=config,
)
```

Do not retry permission or validation errors indefinitely.

---

# 107. Lambda Pagination

Always paginate APIs such as:

```text
list_functions
list_versions_by_function
list_aliases
list_event_source_mappings
list_layers
```

A script that only reads the first page is not production-grade.

---

# 108. Lambda API Throttling

Large multi-account audits can hit API limits.

Use:

```text
pagination
bounded concurrency
retry/backoff
regional batching
```

Do not launch hundreds of API requests simultaneously without control.

---

# 109. Lambda Automation Idempotency

For deployment:

```text
check current version
 ↓
publish only when required
 ↓
update alias only when target differs
```

For compliance:

```text
detect
 ↓
report
 ↓
remediate only if required
```

---

# 110. Lambda Dry Run

For destructive or broad changes:

```bash
python lambdaops.py cleanup --dry-run
```

Output:

```text
Would delete:
orders-function version 8
payments-function version 11
```

No mutation occurs.

---

# 111. Lambda CLI Design

Example:

```bash
python lambdaops.py inventory
```

```bash
python lambdaops.py audit
```

```bash
python lambdaops.py versions
```

```bash
python lambdaops.py aliases
```

```bash
python lambdaops.py health
```

```bash
python lambdaops.py events
```

```bash
python lambdaops.py cleanup --dry-run
```

---

# 112. Lambda Project Structure

```text
lambdaops/
├── cli.py
├── aws.py
├── functions.py
├── versions.py
├── aliases.py
├── events.py
├── security.py
├── monitoring.py
├── reports.py
└── config.py
```

Separate read-only discovery from mutation logic.

---

# 113. Lambda Inventory Function

```python
def list_functions(client):

    paginator = client.get_paginator(
        "list_functions"
    )

    functions = []

    for page in paginator.paginate():

        functions.extend(
            page.get(
                "Functions",
                []
            )
        )

    return functions
```

---

# 114. Lambda Compliance Function

```python
def compliance_check(function):

    return {
        "runtime": function.get(
            "Runtime"
        ),
        "timeout": function.get(
            "Timeout"
        ),
        "memory": function.get(
            "MemorySize"
        ),
        "state": function.get(
            "State"
        ),
    }
```

---

# 115. Lambda Health Classification

Example:

```python
def classify(state, update_status):

    if state != "Active":
        return "CRITICAL"

    if update_status != "Successful":
        return "REVIEW"

    return "PASS"
```

Use richer checks in production.

---

# 116. Lambda Monitoring Through CloudWatch

Python can query:

```text
Invocations
Errors
Duration
Throttles
ConcurrentExecutions
```

Then calculate:

```text
error rate
average duration
peak concurrency
```

---

# 117. Lambda Alert Logic

Example:

```text
IF error rate > policy threshold
AND invocation count is meaningful
THEN REVIEW/ALERT
```

Avoid alerts based on a single failed invocation in a low-volume function.

---

# 118. Lambda Log Analysis

Python can retrieve CloudWatch Logs for operational analysis.

Use:

```text
filter patterns
time windows
structured logs
request IDs
```

Never expose sensitive log content in reports.

---

# 119. Lambda Structured Logging

Preferred:

```json
{
  "service": "orders",
  "request_id": "abc",
  "event": "order_processed",
  "status": "success"
}
```

Do not put secrets into structured logs.

---

# 120. Lambda + ELK

Architecture:

```text
Lambda
 ↓
CloudWatch Logs
 ↓
log pipeline
 ↓
ELK
 ↓
search/dashboard
```

Python can produce audit records alongside application logs.

---

# 121. Lambda + Prometheus/Grafana

AWS Lambda metrics can be surfaced through the organization's AWS monitoring integration.

Python can supplement:

```text
lambda_compliance_failures
lambda_runtime_findings
lambda_alias_drift
lambda_version_cleanup_candidates
```

---

# 122. Lambda + ArgoCD

Lambda is not a Kubernetes workload, but GitOps principles can still apply through infrastructure/application repositories.

Example:

```text
Git
 ↓
Terraform / deployment pipeline
 ↓
Lambda
```

ArgoCD should remain focused on Kubernetes resources unless the organization has an explicit multi-platform GitOps design.

---

# 123. Lambda + DevSecOps

A secure pipeline can include:

```text
Git
 ↓
unit tests
 ↓
SAST
 ↓
dependency/SCA scan
 ↓
IaC scan
 ↓
container scan if image-based
 ↓
build
 ↓
deploy
 ↓
smoke test
 ↓
monitor
```

Use the security tools required by your organization's pipeline.

---

# 124. Lambda Dependency Security

Python Lambda dependencies can contain vulnerabilities.

Audit:

```text
requirements
lock files
package versions
transitive dependencies
```

Use an approved SCA/dependency scanning tool.

---

# 125. Lambda Layer Security

Layers can introduce:

```text
old dependencies
vulnerable libraries
untrusted code
```

Track layer ownership and versions.

---

# 126. Lambda Container Security

For image-based Lambda:

```text
Dockerfile
 ↓
build
 ↓
Trivy/security scan
 ↓
ECR
 ↓
Lambda
```

Pin dependencies where appropriate.

---

# 127. Lambda Function URL

Some Lambda functions may expose Function URLs.

Audit whether:

```text
Function URL enabled
Auth type
CORS
```

match the intended architecture.

Do not assume every Lambda should have a public URL.

---

# 128. Lambda Public Exposure Audit

Check:

```text
Function URL
API Gateway
ALB
resource policy
authentication
authorization
```

A function may be externally reachable through multiple paths.

---

# 129. Lambda Authentication

Possible front doors include:

```text
API Gateway authorization
IAM
JWT/Cognito
application auth
Function URL auth
```

Security depends on the complete request path.

---

# 130. Lambda Concurrency + Downstream Systems

Architecture:

```text
Event spike
   ↓
Lambda concurrency
   ↓
Database/API
   ↓
capacity limit
```

A Lambda scaling event can become a downstream outage.

Use:

```text
reserved concurrency
batch controls
queue buffering
connection pooling
```

where appropriate.

---

# 131. Lambda + SQS Resilience

A resilient design:

```text
Producer
 ↓
SQS
 ↓
Lambda
 ↓
processing
 ↓
success

failure
 ↓
retry
 ↓
DLQ
```

Python can audit event-source and failure-handling configuration.

---

# 132. Lambda + EventBridge Schedule

Scheduled automation:

```text
EventBridge
 ↓
Lambda
 ↓
Python/Boto3
 ↓
AWS operation
```

Good use cases:

```text
inventory
compliance
reporting
cleanup
```

with appropriate safety controls.

---

# 133. Lambda Automation Example — EBS Audit

```text
EventBridge
 ↓
Lambda
 ↓
Boto3 EC2
 ↓
find unattached volumes
 ↓
generate report
```

Do not automatically delete resources without retention/ownership validation.

---

# 134. Lambda Automation Example — RDS Audit

```text
EventBridge
 ↓
Lambda
 ↓
Boto3 RDS
 ↓
backup/encryption/Multi-AZ audit
 ↓
report
```

---

# 135. Lambda Automation Example — EKS Health

```text
EventBridge
 ↓
Lambda
 ↓
Boto3 EKS
 ↓
cluster/nodegroup audit
```

Kubernetes API access from Lambda requires appropriate network access and Kubernetes authorization.

---

# 136. Lambda Automation Example — S3 Audit

```text
EventBridge
 ↓
Lambda
 ↓
Boto3 S3
 ↓
bucket security audit
 ↓
report
```

---

# 137. Lambda Automation Example — IAM Audit

```text
EventBridge
 ↓
Lambda
 ↓
Boto3 IAM
 ↓
policy/role audit
 ↓
report
```

IAM permissions must be scoped to the audit Lambda.

---

# 138. Lambda Health Check Project

Build:

```bash
python lambdaops.py health
```

Report:

```text
Function
State
Update Status
Errors
Throttles
Duration
Concurrency
Alias
```

---

# 139. Lambda Security Project

Build:

```bash
python lambdaops.py security
```

Check:

```text
runtime
execution role
resource policy
public exposure
VPC
environment configuration
layers
tags
```

---

# 140. Lambda Version Cleanup Project

Build:

```bash
python lambdaops.py versions --dry-run
```

Process:

```text
list versions
 ↓
exclude aliases
 ↓
exclude protected
 ↓
apply retention
 ↓
show candidates
```

---

# 141. Lambda Log Retention Project

Build:

```bash
python lambdaops.py logs
```

Report:

```text
function
log group
retention
missing policy
```

---

# 142. Lambda Runtime Audit Project

Build:

```bash
python lambdaops.py runtimes
```

Report:

```text
function
runtime
architecture
policy status
```

---

# 143. Lambda Alias Drift

Desired:

```text
prod → version 20
```

Actual:

```text
prod → version 19
```

Python can report:

```text
ALIAS DRIFT
```

Do not automatically correct production drift unless ownership and remediation policy explicitly allow it.

---

# 144. Lambda Deployment Gate

```text
build
 ↓
test
 ↓
security scan
 ↓
deploy
 ↓
publish version
 ↓
smoke test
 ↓
canary
 ↓
metrics
 ↓
promote
```

---

# 145. Lambda Rollback Gate

```text
canary
 ↓
error rate ↑
 ↓
latency ↑
 ↓
rollback alias
 ↓
verify previous version
 ↓
incident notification
```

Rollback thresholds should be defined before deployment.

---

# 146. Lambda Production Checklist

```text
[ ] Supported runtime
[ ] IAM least privilege
[ ] No secrets in code/logs
[ ] Environment reviewed
[ ] Timeout appropriate
[ ] Memory sized
[ ] Concurrency controlled
[ ] Event source healthy
[ ] DLQ/failure handling
[ ] Logs retained
[ ] Metrics monitored
[ ] Alias/version strategy
[ ] Deployment rollback
[ ] Tags present
```

---

# 147. Lambda Security Checklist

```text
[ ] Execution role scoped
[ ] Resource policy scoped
[ ] Function URL reviewed
[ ] VPC configuration reviewed
[ ] Security groups reviewed
[ ] Secrets externalized
[ ] Dependencies scanned
[ ] Layers reviewed
[ ] Image scanned if applicable
[ ] CloudWatch logs protected
```

---

# 148. Lambda Reliability Checklist

```text
[ ] Timeout tested
[ ] Retry behavior understood
[ ] Concurrency bounded
[ ] Downstream capacity reviewed
[ ] SQS DLQ configured where applicable
[ ] Async failure handling
[ ] Canary deployment
[ ] Rollback tested
[ ] Monitoring
[ ] Alerting
```

---

# 149. Interview — What Can You Automate With Boto3 for Lambda?

**Answer:**

> I can automate Lambda inventory, configuration audits, versions, aliases, event-source mappings, layers, tags, concurrency settings, permissions, deployment verification, CloudWatch integration and controlled operational workflows.

---

# 150. Interview — How Do You Deploy Lambda Safely?

**Answer:**

> I build and test the artifact, run security checks, deploy it, publish an immutable version, run a smoke test, then shift an alias through a controlled rollout. For critical functions I use canary or gradual traffic shifting and monitor errors and latency before full promotion.

---

# 151. Interview — Why Use Lambda Versions?

**Answer:**

> Versions provide immutable deployment points. They allow stable aliases such as `prod` to point to a specific version and make rollback much safer.

---

# 152. Interview — What Is a Lambda Alias?

**Answer:**

> An alias is a stable pointer to a published Lambda version. I can use aliases such as `dev`, `staging` and `prod` so clients do not need to know the underlying version number.

---

# 153. Interview — How Do You Roll Back Lambda?

**Answer:**

> I move the production alias back to the previously validated version. Because published versions are immutable, this provides a fast and deterministic rollback mechanism.

---

# 154. Interview — What Is Lambda Concurrency?

**Answer:**

> Concurrency represents the number of Lambda executions running at the same time. Reserved concurrency can limit a function and protect downstream systems, while provisioned concurrency keeps execution environments initialized to reduce cold-start latency.

---

# 155. Interview — How Can Lambda Overload RDS?

**Answer:**

> Lambda can scale horizontally very quickly. If every invocation opens database connections, a traffic spike can exhaust RDS connections. I control concurrency, use appropriate connection management and consider buffering through services such as SQS where appropriate.

---

# 156. Interview — How Do You Troubleshoot Lambda Timeout?

**Answer:**

> I inspect duration metrics and logs, then check downstream dependencies such as RDS, APIs, S3, DNS and networking. If the function is VPC-connected, I also inspect subnet routing, NAT/VPC endpoints and security groups. I don't simply increase the timeout without finding the bottleneck.

---

# 157. Interview — How Do You Troubleshoot Lambda Throttling?

**Answer:**

> I inspect reserved concurrency, account concurrency, provisioned concurrency and invocation spikes. Then I determine whether the limit is intentional protection or an actual capacity problem.

---

# 158. Interview — How Do You Secure Lambda Environment Variables?

**Answer:**

> I avoid storing sensitive credentials directly in plain environment configuration. I prefer Secrets Manager or Parameter Store with IAM-controlled access and ensure values are never written to logs.

---

# 159. Interview — How Do You Audit Lambda IAM?

**Answer:**

> I identify the execution role, inspect attached and inline policies, check for wildcard permissions and compare access with the function's actual requirements. I also review trust relationships and use AWS analysis tools where appropriate.

---

# 160. Interview — What Is a Lambda Resource Policy?

**Answer:**

> A resource-based policy controls who or what can invoke a Lambda function. I check principals, actions and source conditions to ensure invocation permissions are not broader than required.

---

# 161. Interview — How Do You Audit Lambda Runtimes?

**Answer:**

> I inventory all functions and compare their runtimes with the organization's approved runtime policy and AWS lifecycle information. I create remediation candidates rather than automatically changing production runtimes.

---

# 162. Interview — How Do You Troubleshoot SQS + Lambda?

**Answer:**

> I inspect the event-source mapping, queue depth, Lambda errors, duration, batch size, visibility timeout, retry behavior and DLQ. I also check whether downstream services are causing processing failures.

---

# 163. Interview — What Is the Relationship Between SQS Visibility Timeout and Lambda?

**Answer:**

> The visibility timeout must be designed around the expected processing duration and retry behavior. If it is too short, a message can become visible again while the original invocation is still processing.

---

# 164. Interview — How Do You Prevent S3-Lambda Recursion?

**Answer:**

> I use separate input/output buckets or carefully designed object prefixes and event filters. The Lambda should not write objects that trigger the same event path unintentionally.

---

# 165. Interview — How Do You Monitor Lambda?

**Answer:**

> I monitor invocations, errors, duration, throttles and concurrency, plus event-source-specific metrics such as iterator age where applicable. I correlate those with CloudWatch logs and downstream system health.

---

# 166. Interview — How Do You Optimize Lambda Cost?

**Answer:**

> I analyze invocation volume, duration, memory, architecture, provisioned concurrency and downstream behavior. I test memory changes because more memory can provide more CPU and sometimes reduce execution time enough to lower total cost.

---

# 167. Interview — Lambda Zip vs Container Image?

**Answer:**

> Zip packages are simple and work well for many functions. Container images are useful when dependencies or packaging requirements are better handled through Docker/ECR. The deployment pipeline and security scanning strategy differ between them.

---

# 168. Interview — How Do You Secure Lambda Container Images?

**Answer:**

> I use minimal base images, pin dependencies where practical, scan images for vulnerabilities, use ECR controls and deploy immutable image references according to the organization's policy.

---

# 169. Interview — What Are Lambda Layers?

**Answer:**

> Layers package shared libraries or dependencies separately from function code. They can reduce duplication but introduce another versioning and security lifecycle that must be managed.

---

# 170. Interview — How Do You Handle Lambda VPC Networking?

**Answer:**

> I place the function in appropriate subnets, configure security groups and ensure routes to required destinations. For Internet access from private subnets, I use an appropriate NAT architecture or VPC endpoints for supported AWS services.

---

# 171. Interview — How Do You Troubleshoot a VPC Lambda Timeout?

**Answer:**

> I verify subnet route tables, NAT or VPC endpoint configuration, security groups, DNS and destination reachability. I also determine whether the timeout occurs before or after the downstream connection attempt.

---

# 172. Interview — How Do You Integrate Lambda With CI/CD?

**Answer:**

> Jenkins or GitHub Actions can test and scan the code, build the Zip or container artifact, authenticate to AWS using short-lived credentials, deploy it, publish a version, run smoke tests and update an alias through a controlled promotion process.

---

# 173. Interview — How Does Python Fit Into Lambda CI/CD?

**Answer:**

> Python can provide deployment validation, configuration auditing, smoke tests, health checks and post-deployment verification. I keep the desired deployment state in the approved CI/CD or infrastructure workflow rather than allowing ad-hoc scripts to become an uncontrolled source of truth.

---

# 174. Interview — How Do You Make Lambda Automation Idempotent?

**Answer:**

> I check the current function version, alias target, configuration and resource state before making changes. If the desired state already exists, the script does nothing.

---

# 175. Interview — How Do You Safely Delete Old Lambda Versions?

**Answer:**

> I list versions, exclude `$LATEST`, exclude versions referenced by aliases, exclude protected versions, apply an explicit retention policy and use dry-run before deletion.

---

# 176. Interview — How Do You Automate Lambda Across Accounts?

**Answer:**

> I use STS AssumeRole into narrowly scoped target-account roles, validate the returned account ID, scan approved regions and aggregate normalized Lambda inventory and compliance findings.

---

# 177. Interview — How Do You Test Lambda Automation?

**Answer:**

> I unit-test parsing and classification logic, mock AWS APIs using tools such as Botocore Stubber, and use a dedicated AWS test environment for integration testing. I never run destructive integration tests against production functions.

---

# 178. Interview — What Is the Biggest Lambda Automation Mistake?

**Answer:**

> Treating Lambda as just application code and ignoring its surrounding infrastructure. IAM, concurrency, event sources, retries, networking, versions, aliases and observability are all part of production Lambda operations.

---

# 179. Final Lambda Automation Mental Model

```text
Validate Account
       ↓
Discover Functions
       ↓
Check Runtime
       ↓
Check IAM
       ↓
Check Environment
       ↓
Check VPC
       ↓
Check Event Sources
       ↓
Check Versions/Aliases
       ↓
Check Concurrency
       ↓
Check Logs/Metrics
       ↓
Check Security
       ↓
Generate Report
       ↓
Deploy With Guardrails
       ↓
Verify
       ↓
Rollback If Required
```

The key DevOps principle is:

> **Lambda removes server management, not operational responsibility.**

Next:

```text
09-AWS-Automation-Projects.md
```

will bring the AWS Python section together through complete production-style projects combining Boto3, Linux, IAM, EC2, S3, VPC, RDS, EKS, Lambda, monitoring, CI/CD, security and reporting.
