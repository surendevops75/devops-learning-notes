# 04-IAM-Automation

## Python for AWS DevOps — IAM Roles, Policies, Access Audits, AssumeRole, Least Privilege & Production Automation

AWS Identity and Access Management is one of the highest-risk areas for DevOps automation.

Python/Boto3 can automate:

```text
IAM inventory
role inventory
policy inventory
access-key audits
password/access audits
role assumption
cross-account automation
policy analysis
least-privilege reviews
credential rotation workflows
unused-resource reporting
compliance checks
```

The most important rule is:

> **IAM automation must be more restrictive than the infrastructure automation it controls.**

Never solve an automation problem by blindly granting:

```text
AdministratorAccess
```

---

# 1. IAM Mental Model

Think of AWS authorization as:

```text
Principal
    ↓
Identity-based policy
    ↓
Resource-based policy
    ↓
Permission boundaries
    ↓
SCPs
    ↓
Session policies
    ↓
KMS/resource-specific authorization
    ↓
Allowed / Denied
```

The exact authorization path depends on the AWS service and resource.

---

# 2. IAM Is Global

IAM is generally an AWS account-level/global service.

Unlike EC2:

```text
EC2 → regional
IAM → global
```

Your Boto3 IAM client normally does not need a region for ordinary IAM operations.

```python
import boto3

iam = boto3.client("iam")
```

---

# 3. STS Identity Check

Before performing sensitive automation, identify the caller:

```python
import boto3

sts = boto3.client("sts")

identity = sts.get_caller_identity()

print(identity)
```

Useful fields:

```text
Account
Arn
UserId
```

---

# 4. Why Validate the Account?

Imagine a cleanup script intended for:

```text
dev account
```

but the credentials point to:

```text
production account
```

The script can successfully execute against the wrong environment.

A production-safe workflow begins with:

```text
Who am I?
Which account?
Which role?
```

---

# 5. Account Guard

```python
EXPECTED_ACCOUNT = "123456789012"

identity = sts.get_caller_identity()

if identity["Account"] != EXPECTED_ACCOUNT:
    raise RuntimeError(
        "Unexpected AWS account"
    )
```

Never hardcode real production account IDs into public repositories.

Use secure configuration.

---

# 6. IAM Users vs Roles

### IAM User

Typically represents:

```text
long-lived identity
```

### IAM Role

Typically represents:

```text
temporary permissions
```

Modern DevOps automation should prefer:

```text
roles
+
temporary credentials
```

over long-lived access keys.

---

# 7. IAM Role Mental Model

```text
Principal
   ↓
AssumeRole
   ↓
Temporary credentials
   ↓
AWS API
```

The role has:

```text
trust policy
+
permissions policies
```

---

# 8. Trust Policy vs Permissions Policy

### Trust policy

Answers:

> Who can assume this role?

### Permissions policy

Answers:

> What can the role do after it is assumed?

This distinction is extremely important in interviews and production troubleshooting.

---

# 9. Get IAM Role

```python
response = iam.get_role(
    RoleName="DevOpsAutomationRole"
)

role = response["Role"]

print(role["Arn"])
```

---

# 10. List IAM Roles

```python
paginator = iam.get_paginator(
    "list_roles"
)

for page in paginator.paginate():

    for role in page.get(
        "Roles",
        []
    ):
        print(
            role.get("RoleName")
        )
```

Use pagination for account-wide inventory.

---

# 11. Role Metadata

Useful role fields include:

```text
RoleName
Arn
CreateDate
Path
AssumeRolePolicyDocument
Description
MaxSessionDuration
RoleLastUsed
```

---

# 12. Role Last Used

```python
last_used = role.get(
    "RoleLastUsed"
)

if last_used:
    print(
        last_used.get(
            "LastUsedDate"
        )
    )
```

This can support access reviews, but do not interpret "not recently used" as automatically safe to delete.

---

# 13. List IAM Users

```python
paginator = iam.get_paginator(
    "list_users"
)

for page in paginator.paginate():

    for user in page.get(
        "Users",
        []
    ):
        print(
            user.get("UserName")
        )
```

---

# 14. IAM User Metadata

Useful fields:

```text
UserName
UserId
Arn
CreateDate
PasswordLastUsed
Path
Tags
```

---

# 15. Access Keys

List a user's access keys:

```python
response = iam.list_access_keys(
    UserName=username
)

for key in response.get(
    "AccessKeyMetadata",
    []
):
    print(
        key["AccessKeyId"],
        key["Status"],
        key["CreateDate"]
    )
```

---

# 16. Access-Key Risk

Long-lived access keys can create risk:

```text
leaked credential
repository exposure
developer machine compromise
difficult rotation
unknown ownership
```

Prefer:

```text
IAM roles
OIDC
temporary credentials
```

---

# 17. Access-Key Audit

A useful audit checks:

```text
key age
status
owner
last-used information
```

Then classify:

```text
current
old
inactive
review
```

---

# 18. Access Key Last Used

```python
response = iam.get_access_key_last_used(
    AccessKeyId=access_key_id
)

last_used = response.get(
    "AccessKeyLastUsed",
    {}
)

print(
    last_used.get("LastUsedDate")
)
```

The response can also identify the service and region associated with recent use.

---

# 19. Important Audit Limitation

"Not recently used" does not automatically mean:

```text
safe to delete
```

Consider:

```text
break-glass usage
scheduled jobs
rare disaster recovery workflows
external integrations
documentation
owner confirmation
```

---

# 20. Deactivate an Access Key

```python
iam.update_access_key(
    UserName=username,
    AccessKeyId=access_key_id,
    Status="Inactive",
)
```

Prefer deactivation before deletion when testing whether a credential is still required.

---

# 21. Delete an Access Key

```python
iam.delete_access_key(
    UserName=username,
    AccessKeyId=access_key_id,
)
```

This is destructive.

Use a controlled process.

---

# 22. Safe Key-Rotation Workflow

```text
identify key
   ↓
confirm owner
   ↓
create replacement
   ↓
update application
   ↓
test
   ↓
deactivate old key
   ↓
observe
   ↓
delete old key
```

Do not delete first.

---

# 23. Why Rotation Matters

A production key rotation workflow should minimize:

```text
downtime
credential exposure
rollback difficulty
```

Use temporary credentials whenever possible to reduce the need for static-key rotation.

---

# 24. IAM Password Policy

Retrieve the account password policy:

```python
response = iam.get_account_password_policy()
```

Relevant settings may include:

```text
MinimumPasswordLength
RequireSymbols
RequireNumbers
RequireUppercaseCharacters
RequireLowercaseCharacters
PasswordReusePrevention
MaxPasswordAge
```

---

# 25. Password Policy Audit

A compliance script can compare:

```text
actual configuration
        vs
organizational standard
```

Then report:

```text
PASS
WARN
FAIL
```

---

# 26. IAM Groups

List groups:

```python
paginator = iam.get_paginator(
    "list_groups"
)

for page in paginator.paginate():

    for group in page.get(
        "Groups",
        []
    ):
        print(
            group.get("GroupName")
        )
```

Groups can organize permissions for IAM users.

---

# 27. Group Users

```python
response = iam.get_group(
    GroupName=group_name
)

for user in response.get(
    "Users",
    []
):
    print(
        user.get("UserName")
    )
```

---

# 28. List Attached User Policies

```python
paginator = iam.get_paginator(
    "list_attached_user_policies"
)

for page in paginator.paginate(
    UserName=username
):

    for policy in page.get(
        "AttachedPolicies",
        []
    ):
        print(
            policy["PolicyArn"]
        )
```

---

# 29. List Inline User Policies

```python
paginator = iam.get_paginator(
    "list_user_policies"
)

for page in paginator.paginate(
    UserName=username
):

    for policy_name in page.get(
        "PolicyNames",
        []
    ):
        print(
            policy_name
        )
```

---

# 30. Role Attached Policies

```python
paginator = iam.get_paginator(
    "list_attached_role_policies"
)

for page in paginator.paginate(
    RoleName=role_name
):

    for policy in page.get(
        "AttachedPolicies",
        []
    ):
        print(
            policy["PolicyArn"]
        )
```

---

# 31. Role Inline Policies

```python
paginator = iam.get_paginator(
    "list_role_policies"
)

for page in paginator.paginate(
    RoleName=role_name
):

    for policy_name in page.get(
        "PolicyNames",
        []
    ):
        print(
            policy_name
        )
```

---

# 32. Policy Types

IAM policies can be:

```text
identity-based
resource-based
permissions boundary
SCP
session policy
```

A DevOps audit should understand the complete authorization context.

---

# 33. Managed vs Inline Policies

### Managed policy

Reusable policy object:

```text
AWS managed
customer managed
```

### Inline policy

Embedded directly into:

```text
user
group
role
```

For maintainability, reusable customer-managed policies are often easier to govern.

---

# 34. Get Managed Policy

```python
response = iam.get_policy(
    PolicyArn=policy_arn
)

policy = response["Policy"]

print(
    policy["DefaultVersionId"]
)
```

---

# 35. Get Policy Version

```python
response = iam.get_policy_version(
    PolicyArn=policy_arn,
    VersionId=version_id,
)

document = response[
    "PolicyVersion"
]["Document"]
```

The returned policy document may be URL-encoded depending on the API response.

---

# 36. Decode Policy Document

```python
from urllib.parse import unquote

document = unquote(
    document
)
```

Depending on the response type, you may need to parse JSON after decoding.

---

# 37. Parse Policy JSON

```python
import json

policy = json.loads(
    document
)

print(
    policy.get("Statement")
)
```

---

# 38. Policy Structure

Typical structure:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::example-bucket/*"
    }
  ]
}
```

---

# 39. Effect

Usually:

```text
Allow
Deny
```

Explicit denies are important because they can override otherwise-allowed actions.

---

# 40. Action

Examples:

```text
ec2:DescribeInstances
s3:GetObject
s3:PutObject
eks:DescribeCluster
iam:PassRole
```

Avoid:

```text
Action: "*"
```

unless there is a documented and tightly controlled reason.

---

# 41. Resource

Prefer specific resources:

```text
arn:aws:s3:::example-bucket/*
```

over:

```text
*
```

where the service supports resource-level permissions.

Some AWS actions require `Resource: "*"`.

---

# 42. Least Privilege

Instead of:

```json
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
```

prefer:

```json
{
  "Effect": "Allow",
  "Action": [
    "ec2:DescribeInstances",
    "ec2:DescribeInstanceStatus"
  ],
  "Resource": "*"
}
```

The exact permissions depend on the automation.

---

# 43. IAM Automation Role Example

A read-only EC2 inventory role might require:

```text
ec2:DescribeInstances
ec2:DescribeInstanceStatus
sts:GetCallerIdentity
```

It should not automatically receive:

```text
ec2:TerminateInstances
iam:CreateUser
s3:DeleteBucket
```

---

# 44. Read vs Write Permissions

Separate:

```text
read-only inventory
```

from:

```text
mutating automation
```

For example:

```text
inventory-role
```

and:

```text
scheduler-role
```

can have different permissions.

---

# 45. IAM Policy Validation

AWS provides IAM policy validation capabilities.

Python automation can use AWS APIs to validate policy syntax where supported.

Still combine automated validation with human security review for high-risk policies.

---

# 46. Access Analyzer

AWS IAM Access Analyzer can help identify:

```text
external access
unused access findings
policy validation issues
```

Boto3 can interact with the Access Analyzer service.

```python
accessanalyzer = boto3.client(
    "accessanalyzer"
)
```

---

# 47. IAM Access Analyzer

Useful for:

```text
resource access analysis
policy validation
organization-wide review
```

Use native AWS analysis capabilities rather than trying to reproduce all IAM authorization logic in custom Python.

---

# 48. Analyzer Listing

```python
paginator = accessanalyzer.get_paginator(
    "list_analyzers"
)

for page in paginator.paginate():

    for analyzer in page.get(
        "analyzers",
        []
    ):
        print(
            analyzer.get("name")
        )
```

---

# 49. Access Analyzer Findings

Findings can identify potentially unintended external access.

Your automation can:

```text
collect findings
 ↓
classify
 ↓
notify owner
 ↓
track remediation
```

Avoid automatically changing permissions based solely on a finding without validation.

---

# 50. IAM Role Creation

```python
response = iam.create_role(
    RoleName="DevOpsAutomationRole",
    AssumeRolePolicyDocument=trust_policy,
    Description="Automation role",
)
```

Role creation should be governed by:

```text
naming
trust policy
permissions
tags
path
ownership
```

---

# 51. Trust Policy Example

Conceptually:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::123456789012:role/TrustedRole"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Use the narrowest trusted principal possible.

---

# 52. Trust Policy Risk

Dangerous pattern:

```json
"Principal": "*"
```

This can create unintended access depending on additional conditions and resource policy context.

Treat broad trust relationships as high-priority security findings.

---

# 53. Trust Policy Conditions

Conditions can restrict assumptions based on context.

For example:

```text
external ID
source identity
organization
principal tags
```

Use conditions where they meaningfully reduce risk.

---

# 54. AssumeRole

STS:

```python
sts = boto3.client(
    "sts"
)
```

Assume:

```python
response = sts.assume_role(
    RoleArn=role_arn,
    RoleSessionName="DevOpsAutomation",
)
```

---

# 55. Temporary Credentials

Response contains:

```text
AccessKeyId
SecretAccessKey
SessionToken
Expiration
```

Never print these values.

---

# 56. Create a Client With Assumed Credentials

```python
credentials = response[
    "Credentials"
]

target_ec2 = boto3.client(
    "ec2",
    aws_access_key_id=credentials[
        "AccessKeyId"
    ],
    aws_secret_access_key=credentials[
        "SecretAccessKey"
    ],
    aws_session_token=credentials[
        "SessionToken"
    ],
)
```

Prefer established Boto3 credential/session patterns and avoid unnecessary credential handling in application code.

---

# 57. Better Role-Assumption Pattern

Use a Boto3 session:

```python
session = boto3.Session(
    aws_access_key_id=credentials[
        "AccessKeyId"
    ],
    aws_secret_access_key=credentials[
        "SecretAccessKey"
    ],
    aws_session_token=credentials[
        "SessionToken"
    ],
)

ec2 = session.client(
    "ec2"
)
```

---

# 58. Cross-Account Architecture

```text
Central Automation Account
          ↓
STS AssumeRole
          ↓
Dev Account
          ↓
EC2/S3/IAM
```

Repeat for:

```text
staging
production
security
shared services
```

---

# 59. Cross-Account Safety

For each account:

```text
AssumeRole
 ↓
GetCallerIdentity
 ↓
verify expected account
 ↓
perform operation
```

Never assume that successful role assumption means you reached the intended account.

---

# 60. External ID

For third-party or delegated access patterns, an external ID can help mitigate certain confused-deputy risks.

Example:

```python
response = sts.assume_role(
    RoleArn=role_arn,
    RoleSessionName="Automation",
    ExternalId=external_id,
)
```

Use only where the trust relationship is designed for it.

---

# 61. Role Session Name

Use meaningful session names:

```text
jenkins-ec2-audit
github-s3-deploy
security-compliance-scan
```

This improves auditability.

---

# 62. Session Duration

Roles have a maximum session duration.

For automation:

```text
short-lived session
```

is preferable when the job does not need a long session.

Do not request longer sessions than necessary.

---

# 63. IAM PassRole

A common DevOps permission is:

```text
iam:PassRole
```

This is high-impact.

It can allow a principal to pass a role to an AWS service that can then act with that role's permissions.

---

# 64. PassRole Least Privilege

Avoid:

```json
"Action": "iam:PassRole",
"Resource": "*"
```

when unnecessary.

Restrict the role ARN(s) that can be passed.

---

# 65. Why PassRole Matters in EKS/EC2

Examples:

```text
EC2 instance profile
ECS task role
Lambda execution role
CloudFormation execution role
```

Automation that creates these resources may require `iam:PassRole`.

---

# 66. IAM Role Tags

```python
iam.tag_role(
    RoleName=role_name,
    Tags=[
        {
            "Key": "ManagedBy",
            "Value": "PythonAutomation",
        },
    ],
)
```

Tags help identify ownership and automation.

---

# 67. IAM User Tags

```python
iam.tag_user(
    UserName=username,
    Tags=[
        {
            "Key": "Owner",
            "Value": "platform",
        }
    ],
)
```

Use tags as governance metadata, not as a substitute for authorization.

---

# 68. IAM Path

IAM resources can use paths:

```text
/service-role/
```

or:

```text
/application/
```

Paths can help organize resources and support policy/reporting conventions.

---

# 69. Role Naming Convention

Example:

```text
devops-automation-readonly
devops-automation-ec2-scheduler
devops-automation-s3-backup
```

Names should communicate:

```text
purpose
scope
environment
```

---

# 70. IAM Inventory Project

Build:

```bash
python iamops.py inventory
```

Collect:

```text
users
roles
groups
managed policies
access keys
role last-used data
```

---

# 71. IAM Inventory Architecture

```text
Python
 ↓
STS identity
 ↓
IAM APIs
 ↓
users/roles/policies
 ↓
normalize
 ↓
JSON/CSV
 ↓
security report
```

---

# 72. User Inventory

For each user collect:

```text
username
arn
created
password last used
access-key status
access-key age
tags
```

---

# 73. Role Inventory

For each role collect:

```text
role name
arn
created
last used
trust policy
attached policies
inline policies
tags
```

---

# 74. Access-Key Inventory

Report:

```text
user
access key ID
status
created
last used
age
```

Do not report secret access keys.

---

# 75. Credential Audit

Classify:

```text
ACTIVE + RECENT
ACTIVE + OLD
INACTIVE
NEVER USED
```

Then send for review.

---

# 76. IAM Key Age

```python
from datetime import datetime, timezone

now = datetime.now(
    timezone.utc
)

age_days = (
    now - create_date
).days
```

Use the organization's rotation policy rather than assuming a universal age threshold.

---

# 77. Never Delete Keys Based Only on Age

An old key can still be:

```text
business-critical
rarely used
disaster-recovery credential
external integration
```

Use owner and usage validation.

---

# 78. IAM Policy Audit Project

Workflow:

```text
list roles
 ↓
list attached policies
 ↓
retrieve policy versions
 ↓
parse JSON
 ↓
detect risky patterns
 ↓
report
```

---

# 79. Risky Policy Patterns

Potential findings:

```text
Action="*"
Resource="*"
iam:PassRole on "*"
broad trust principal
AdministratorAccess
unnecessary write permissions
```

These are findings for review, not automatic proof of a vulnerability.

---

# 80. Detect Wildcard Actions

Concept:

```python
action = statement.get(
    "Action"
)

if action == "*":
    print(
        "Wildcard action"
    )
```

Remember that `Action` may also be a list.

---

# 81. Normalize Action

```python
actions = statement.get(
    "Action",
    []
)

if isinstance(
    actions,
    str
):
    actions = [actions]
```

Then inspect each action.

---

# 82. Detect Wildcard Resources

```python
resources = statement.get(
    "Resource",
    []
)

if isinstance(
    resources,
    str
):
    resources = [resources]

if "*" in resources:
    print(
        "Wildcard resource"
    )
```

Interpret findings in service context.

---

# 83. Detect PassRole

```python
if "iam:PassRole" in actions:
    print(
        "Review PassRole"
    )
```

Also handle wildcard action patterns.

---

# 84. Detect Broad Trust

Inspect:

```text
AssumeRolePolicyDocument
```

Look for broad:

```text
Principal
```

and missing/weak conditions.

Do not build a simplistic parser that declares every `*` unsafe without considering the complete trust policy.

---

# 85. IAM Compliance Report

Example:

```text
IAM Compliance Report
=====================

Access Keys:
Old: 7
Inactive: 3
Review: 4

Roles:
Broad Trust: 2
Wildcard Permissions: 5
PassRole Review: 3

Password Policy:
PASS
```

---

# 86. IAM + Access Analyzer

A mature audit pipeline:

```text
IAM inventory
      ↓
policy analysis
      ↓
Access Analyzer findings
      ↓
risk classification
      ↓
owner notification
      ↓
remediation tracking
```

---

# 87. IAM + CloudTrail

For authorization investigations:

```text
IAM configuration
+
CloudTrail API activity
```

can help determine:

```text
who
did what
when
from where
```

Python can query appropriate AWS APIs or downstream logs, but should not replace the native audit trail.

---

# 88. IAM + Security Hub

Where enabled, security findings can be aggregated into AWS Security Hub.

Python can integrate reporting with the security workflow.

---

# 89. IAM + CI/CD

A CI/CD pipeline should receive:

```text
temporary role credentials
```

rather than:

```text
permanent access keys
```

Preferred architecture:

```text
CI system
 ↓
OIDC / workload identity
 ↓
STS
 ↓
IAM Role
 ↓
AWS
```

---

# 90. GitHub Actions OIDC

Concept:

```text
GitHub Actions
       ↓
OIDC token
       ↓
AWS STS
       ↓
AssumeRoleWithWebIdentity
       ↓
temporary credentials
```

The IAM trust policy should restrict:

```text
repository
organization
branch/environment
audience
```

as appropriate.

---

# 91. Jenkins IAM Role

Depending on Jenkins deployment:

```text
Jenkins
 ↓
instance/task/workload identity
 ↓
IAM role
 ↓
Boto3
```

or a secure external credential mechanism can be used.

Avoid storing long-lived AWS secrets in Jenkins when a role-based design is available.

---

# 92. IAM + EKS

EKS workloads should use AWS-supported workload identity mechanisms.

Concept:

```text
Pod
 ↓
service account
 ↓
AWS IAM role
 ↓
temporary credentials
 ↓
AWS API
```

This avoids distributing shared static AWS keys across pods.

---

# 93. IAM + Lambda

Lambda functions commonly use:

```text
execution role
```

The role should contain only permissions required by the function.

Python can audit Lambda execution roles.

---

# 94. IAM + EC2

EC2 applications should use:

```text
instance profile
 ↓
IAM role
 ↓
temporary credentials
```

Avoid embedding access keys in:

```text
/etc
source code
environment variables
AMI
user-data
```

unless there is an exceptional, controlled reason.

---

# 95. IAM + S3

A backup role may need:

```text
s3:GetObject
s3:PutObject
s3:ListBucket
```

but not:

```text
s3:DeleteBucket
iam:*
```

Scope permissions to the actual bucket and prefixes where supported.

---

# 96. IAM + ECR

A CI/CD role pulling/pushing ECR images may need a specific set of:

```text
ecr:GetAuthorizationToken
ecr:BatchCheckLayerAvailability
ecr:PutImage
ecr:InitiateLayerUpload
...
```

Use AWS documentation and service-specific policy guidance rather than guessing permissions.

---

# 97. IAM + EKS Deployment

A deployment role may require:

```text
EKS access
ECR access
IAM PassRole
```

depending on architecture.

Separate:

```text
build role
deployment role
cluster administration role
```

when practical.

---

# 98. IAM Permission Boundaries

A permissions boundary sets the maximum permissions an identity can receive.

Concept:

```text
Identity policy
      ∩
Permissions boundary
      =
Effective permissions
```

It does not itself grant permissions.

---

# 99. Get Role Policy Boundary

Role metadata can include:

```python
role.get(
    "PermissionsBoundary"
)
```

Use this during governance audits.

---

# 100. SCP Awareness

Service Control Policies operate at the AWS Organizations level.

A role can have:

```text
Allow
```

in its IAM policy and still be denied by an SCP.

This is a common troubleshooting scenario.

---

# 101. IAM Deny Troubleshooting

When AWS returns:

```text
AccessDenied
```

check:

```text
identity policy
resource policy
permissions boundary
SCP
session policy
KMS policy
trust relationship
```

Do not assume the attached IAM policy is the only control.

---

# 102. IAM Simulation

AWS IAM provides policy simulation capabilities for evaluating authorization scenarios.

Use simulation tools for:

```text
policy validation
authorization troubleshooting
least-privilege analysis
```

Do not treat simulation as a replacement for production testing.

---

# 103. IAM Role Deletion Safety

Before deleting a role:

```text
check last used
check attached policies
check trusted principals
check dependencies
check IaC
check Lambda/EKS/EC2 references
check owner
```

Never delete a role solely because it appears old.

---

# 104. Detach Role Policy

```python
iam.detach_role_policy(
    RoleName=role_name,
    PolicyArn=policy_arn,
)
```

This changes permissions.

Use approval and change management where required.

---

# 105. Delete Role

```python
iam.delete_role(
    RoleName=role_name
)
```

The role may need attached policies removed first.

This is destructive.

---

# 106. IAM Policy Version Management

Managed policies can have multiple versions.

Audit:

```text
default version
old versions
policy changes
```

Keep policy history understandable.

---

# 107. Delete Nondefault Policy Version

A nondefault policy version can be deleted when no longer required.

Do not automatically remove old versions without understanding change-management and rollback requirements.

---

# 108. IAM Automation Safety Model

For any permission-changing operation:

```text
discover
 ↓
identify owner
 ↓
validate account
 ↓
validate resource
 ↓
show diff
 ↓
dry-run
 ↓
approval
 ↓
change
 ↓
verify
 ↓
audit
```

---

# 109. Policy Diff

Before changing a policy:

```text
current policy
      ↓
proposed policy
      ↓
diff
      ↓
security review
```

A useful automation should show:

```text
added actions
removed actions
changed resources
changed principals
```

---

# 110. Never Log Secrets

Never log:

```text
SecretAccessKey
session token
password
private key
presigned URL
```

Safe:

```text
role ARN
account ID where appropriate
access key ID
operation
resource ARN
request ID
```

Even non-secret identifiers should be handled appropriately in external logs.

---

# 111. IAM Request IDs

AWS responses/errors can provide request metadata useful for troubleshooting.

Log:

```text
operation
error code
request ID
```

when available.

---

# 112. IAM Retry Configuration

```python
from botocore.config import Config

config = Config(
    retries={
        "max_attempts": 5,
        "mode": "standard",
    }
)

iam = boto3.client(
    "iam",
    config=config,
)
```

Avoid aggressive polling against IAM.

---

# 113. IAM Automation Error Handling

```python
from botocore.exceptions import ClientError

try:
    response = iam.get_role(
        RoleName=role_name
    )

except ClientError as exc:

    error = exc.response.get(
        "Error",
        {}
    )

    print(
        error.get("Code")
    )

    raise
```

Classify errors rather than retrying every failure.

---

# 114. Common IAM Errors

Examples:

```text
NoSuchEntity
AccessDenied
EntityAlreadyExists
DeleteConflict
LimitExceeded
MalformedPolicyDocument
InvalidInput
```

---

# 115. NoSuchEntity

Possible causes:

```text
wrong account
wrong role/user name
resource deleted
wrong path
```

Validate context before troubleshooting the resource itself.

---

# 116. EntityAlreadyExists

Possible causes:

```text
role already exists
user already exists
policy already exists
```

Make provisioning workflows idempotent:

```text
exists → inspect/reconcile
doesn't exist → create
```

---

# 117. DeleteConflict

A role/user/policy may have dependencies.

Check:

```text
attached policies
inline policies
instance profiles
service references
```

before deletion.

---

# 118. IAM Limits

AWS IAM has quotas and limits.

Automation should:

```text
reuse roles
avoid unnecessary policy creation
avoid excessive identities
monitor quota-related errors
```

Do not create a new IAM role for every execution.

---

# 119. Idempotent IAM Role Creation

Concept:

```python
try:
    iam.get_role(
        RoleName=role_name
    )

    exists = True

except iam.exceptions.NoSuchEntityException:
    exists = False
```

Then:

```text
exists → validate/reconcile
missing → create
```

---

# 120. Reconciliation

A mature IAM automation does not merely create resources.

It compares:

```text
desired state
     vs
actual state
```

Then determines:

```text
no change
update
create
remove
review
```

---

# 121. IAM Desired State

Example:

```yaml
role:
  name: devops-ec2-readonly

permissions:
  - ec2:DescribeInstances
  - ec2:DescribeInstanceStatus
```

Python can validate actual configuration against this policy.

---

# 122. IAM Policy as Code

Keep policies in version control:

```text
iam/
  roles/
    ec2-readonly.json
    s3-backup.json
```

Then:

```text
Git
 ↓
review
 ↓
test
 ↓
CI
 ↓
deploy
```

This provides auditability.

---

# 123. Policy CI Pipeline

```text
pull request
 ↓
JSON validation
 ↓
IAM policy validation
 ↓
security checks
 ↓
diff
 ↓
approval
 ↓
deployment
```

Useful checks include:

```text
wildcards
PassRole
broad principals
unexpected resources
```

---

# 124. IAM + Terraform

Terraform can manage:

```text
IAM roles
policies
policy attachments
instance profiles
OIDC providers
```

Python can perform:

```text
inventory
audit
access review
reporting
```

Avoid having two systems independently mutate the same policy.

---

# 125. IAM + Ansible

Ansible can orchestrate:

```text
policy deployment
role configuration
audit workflows
```

Python can perform custom analysis where needed.

---

# 126. IAM + Jenkins

```text
Jenkins
 ↓
assume automation role
 ↓
Python
 ↓
IAM audit
 ↓
report
```

Use the narrowest role possible.

---

# 127. IAM + GitHub Actions

```text
GitHub
 ↓
OIDC
 ↓
STS
 ↓
IAM role
 ↓
Python
 ↓
AWS
```

Trust policy should restrict the GitHub identity to the intended repository/environment.

---

# 128. IAM + ArgoCD

ArgoCD itself does not eliminate AWS IAM requirements.

For EKS workloads:

```text
ArgoCD
 ↓
Kubernetes resources
 ↓
workload identity
 ↓
IAM role
 ↓
AWS API
```

Separate deployment permissions from application runtime permissions.

---

# 129. IAM Security Automation Project

Build:

```bash
python iamops.py audit
```

Checks:

```text
access keys
password policy
roles
trust policies
wildcards
PassRole
permissions boundaries
```

---

# 130. IAM Credential Audit Project

Workflow:

```text
list users
 ↓
list access keys
 ↓
get last-used
 ↓
calculate age
 ↓
classify
 ↓
report
```

Output:

```text
User       Key Status   Age    Last Used
-----------------------------------------
builduser  Active       18d    2h
legacy     Active       410d   never
```

---

# 131. IAM Role Audit Project

Workflow:

```text
list roles
 ↓
last-used analysis
 ↓
policy inventory
 ↓
trust analysis
 ↓
PassRole analysis
 ↓
report
```

---

# 132. IAM Policy Audit Project

Workflow:

```text
list managed policies
 ↓
get default version
 ↓
decode
 ↓
parse JSON
 ↓
inspect actions/resources/principals
 ↓
risk classification
```

---

# 133. IAM Access Analyzer Project

Workflow:

```text
Access Analyzer
 ↓
findings
 ↓
Python
 ↓
classify
 ↓
owner mapping
 ↓
ticket/notification
```

---

# 134. Cross-Account IAM Audit

```text
central account
     ↓
AssumeRole
     ↓
account A
     ↓
IAM inventory

central account
     ↓
AssumeRole
     ↓
account B
     ↓
IAM inventory
```

Aggregate:

```text
account
role
user
policy
finding
```

---

# 135. IAM Compliance Dashboard

Useful metrics:

```text
users_total
roles_total
active_access_keys
old_access_keys
unused_roles
wildcard_policies
broad_trust_roles
PassRole_findings
AccessAnalyzer_findings
```

---

# 136. IAM + Prometheus/Grafana

Expose automation metrics:

```text
iam_audit_runs_total
iam_audit_failures_total
iam_users_total
iam_roles_total
iam_access_keys_total
iam_findings_total
```

Do not expose secret values as metric labels.

---

# 137. IAM + ELK

Structured logs:

```text
iam_audit_started
iam_policy_reviewed
iam_finding_detected
iam_role_assumed
iam_audit_completed
```

Use Kibana for investigation and trend analysis.

---

# 138. IAM Notifications

Example:

```text
IAM Security Audit

Old access keys: 6
Broad trust roles: 2
Wildcard policies: 4
PassRole findings: 3

Action: security review required.
```

Send only the necessary information.

---

# 139. IAM Automation Testing

Unit-test:

```text
account guard
policy parser
wildcard detection
PassRole detection
trust-policy analysis
key classification
dry-run
```

---

# 140. IAM Stub Testing

```python
from botocore.stub import Stubber

stubber = Stubber(iam)

stubber.add_response(
    "get_role",
    {
        "Role": {
            "RoleName": "test-role",
            "Arn": "arn:aws:iam::123:role/test-role",
        }
    },
)

stubber.activate()
```

Use deterministic API responses in unit tests.

---

# 141. IAM Integration Testing

Use:

```text
dedicated test account
test roles
test policies
test users only when required
```

Never run destructive IAM tests against production.

---

# 142. IAM Test Isolation

Use naming:

```text
automation-test-*
```

and tags:

```text
Environment=test
ManagedBy=AutomationTest
```

Clean up test resources after execution.

---

# 143. Production Failure — Wrong Account

Symptoms:

```text
unexpected users
unexpected roles
unexpected policies
```

Response:

```text
stop
verify caller identity
verify account
verify role
```

---

# 144. Production Failure — AssumeRole Denied

Check:

```text
source identity permissions
trust policy
target role ARN
external ID
SCP
permissions boundary
session policy
```

---

# 145. Production Failure — PassRole Denied

Check:

```text
iam:PassRole
target role ARN
service principal
source identity
SCP
permissions boundary
```

Do not simply grant `iam:PassRole` on `*`.

---

# 146. Production Failure — Policy Looks Correct but Access Is Denied

Investigate:

```text
explicit Deny
SCP
permissions boundary
resource policy
session policy
KMS key policy
wrong account
wrong role
```

IAM authorization is evaluated across multiple policy layers.

---

# 147. Production Failure — Old Access Key Still Used

Do not delete immediately.

Workflow:

```text
identify application
 ↓
contact owner
 ↓
create replacement
 ↓
update application
 ↓
test
 ↓
deactivate old key
 ↓
observe
 ↓
delete
```

---

# 148. Production Failure — Role Deleted Accidentally

Recovery depends on:

```text
role configuration
IaC
trust policy
permissions
dependent resources
```

This is why IAM should be managed as code and changes should be reviewed.

---

# 149. Production Failure — Over-Permissioned Role

Do not immediately remove permissions during a live incident.

First:

```text
identify usage
 ↓
identify application
 ↓
analyze CloudTrail/access data
 ↓
design least privilege
 ↓
test
 ↓
reduce permissions
```

---

# 150. Production Failure — Automation Creates Too Many Roles

Investigate:

```text
role naming
idempotency
job design
cleanup
reuse
```

Prefer reusable roles with narrowly defined responsibilities.

---

# 151. Production Failure — Broad Trust Detected

Investigate:

```text
who can assume role
why
from where
conditions
external identity
```

Then remediate according to the security process.

---

# 152. IAM Production Checklist

```text
[ ] STS caller validation
[ ] Expected account validation
[ ] Least privilege
[ ] Roles over static keys
[ ] OIDC where available
[ ] Narrow trust policies
[ ] PassRole restrictions
[ ] Permissions boundaries where appropriate
[ ] SCP awareness
[ ] Access Analyzer
[ ] Credential audit
[ ] Policy-as-code
[ ] Change review
[ ] Logging
[ ] Testing
```

---

# 153. Interview — User vs Role?

**Answer:**

> An IAM user is a long-lived identity, while an IAM role is designed to provide temporary credentials to a trusted principal. For DevOps automation I prefer roles and temporary credentials whenever possible.

---

# 154. Interview — Trust Policy vs Permission Policy?

**Answer:**

> A trust policy defines who can assume a role. The permissions policy defines what the role can do after it is assumed.

---

# 155. Interview — How Do You Authenticate Python to AWS?

**Answer:**

> I use the Boto3 credential provider chain and prefer IAM roles, workload identity or OIDC-based temporary credentials. I avoid hardcoded or long-lived access keys.

---

# 156. Interview — How Do You Validate the AWS Account?

**Answer:**

> I call STS `GetCallerIdentity`, compare the returned account ID against the expected account, and stop the workflow if there is a mismatch.

---

# 157. Interview — How Do You Automate Cross-Account AWS Operations?

**Answer:**

> I use STS AssumeRole into a tightly scoped target role, create a Boto3 session using the temporary credentials, validate the target account with GetCallerIdentity, then perform the required operation.

---

# 158. Interview — What Is Least Privilege?

**Answer:**

> Least privilege means granting only the actions, resources and conditions required for a workload to perform its intended function, and nothing more.

---

# 159. Interview — Why Is `iam:PassRole` Dangerous?

**Answer:**

> PassRole can allow a principal to pass a privileged IAM role to an AWS service. If it is broadly scoped, the effective privilege of the automation can become much larger than expected.

---

# 160. Interview — How Do You Audit IAM Access Keys?

**Answer:**

> I list users and their keys, collect status, creation date and last-used information, classify old or unused credentials, identify owners and recommend deactivation before deletion.

---

# 161. Interview — Would You Automatically Delete Old Access Keys?

**Answer:**

> No. Age alone is not enough. I first identify ownership and usage, create replacement credentials or migrate to roles where possible, deactivate the old key, observe, and then delete it after validation.

---

# 162. Interview — How Do You Detect Wildcard IAM Policies?

**Answer:**

> I retrieve policy versions, decode and parse the JSON, normalize Action and Resource into lists, and flag broad patterns such as `Action:*`, `Resource:*`, and overly broad PassRole permissions for review.

---

# 163. Interview — What Can Cause AccessDenied Even With an Allow Policy?

**Answer:**

> Explicit denies, SCPs, permissions boundaries, session policies, resource policies, KMS key policies, incorrect trust relationships or using the wrong account/role can all cause an authorization failure.

---

# 164. Interview — What Is a Permissions Boundary?

**Answer:**

> A permissions boundary defines the maximum permissions an IAM identity can receive. It does not grant permissions by itself; effective permissions are constrained by the boundary.

---

# 165. Interview — What Is an SCP?

**Answer:**

> A Service Control Policy is an AWS Organizations control that defines the maximum available permissions for accounts or organizational units. An IAM Allow can still be blocked by an SCP.

---

# 166. Interview — How Do You Secure GitHub Actions AWS Access?

**Answer:**

> I use GitHub OIDC with an AWS IAM role and temporary credentials. The trust policy is restricted to the intended repository and deployment context instead of storing long-lived AWS access keys.

---

# 167. Interview — How Do You Secure EKS Pod Access to AWS?

**Answer:**

> I use an AWS-supported workload identity mechanism so the Kubernetes workload receives temporary credentials associated with a narrowly scoped IAM role. I avoid shared static access keys inside pods.

---

# 168. Interview — How Would You Build an IAM Audit Tool?

**Answer:**

> I would validate the account, inventory users and roles, inspect access keys, retrieve policies, analyze trust relationships and risky permissions, integrate Access Analyzer findings, generate a report and publish metrics and notifications.

---

# 169. Interview — How Do You Test IAM Automation?

**Answer:**

> I unit-test policy parsing and decision logic with Boto3 stubs, then use an isolated AWS test account for integration testing. I never run destructive IAM tests against production.

---

# 170. Interview — How Do You Make IAM Automation Idempotent?

**Answer:**

> Before creating or modifying a resource, I check whether it exists and compare the current configuration with the desired state. Existing compliant resources become no-ops; differences are reconciled intentionally.

---

# 171. Interview — How Do You Manage IAM Policies in DevOps?

**Answer:**

> I keep policies in version control, validate them in CI, review permission changes, scan for broad permissions, deploy through controlled infrastructure-as-code workflows and use AWS-native analysis capabilities for ongoing review.

---

# 172. Interview — How Do You Rotate a Production Access Key?

**Answer:**

> I identify the consumer, create a replacement, update the application securely, test it, deactivate the old key, monitor for failures and finally delete the old key. My preferred long-term solution is to migrate the workload to temporary role credentials.

---

# 173. Interview — How Do You Investigate an IAM Incident?

**Answer:**

> I first identify the affected account and principal, inspect the authorization configuration and relevant CloudTrail activity, determine what changed or was accessed, contain the issue according to incident procedures, then remediate and document the root cause.

---

# 174. Interview — How Do You Prevent IAM Drift?

**Answer:**

> I use infrastructure as code and policy as code, CI validation, code review and scheduled audits. Python/Boto3 can detect drift and report it without becoming a second uncontrolled source of truth.

---

# 175. Final IAM Automation Mental Model

```text
Identify Caller
      ↓
Validate Account
      ↓
Use Temporary Credentials
      ↓
Least Privilege
      ↓
Narrow Trust
      ↓
Narrow Permissions
      ↓
Validate Desired State
      ↓
Dry-Run / Review
      ↓
Apply Change
      ↓
Verify
      ↓
Audit
      ↓
Monitor
```

The key DevOps principle is:

> **Identity is part of infrastructure. Treat IAM changes with the same discipline as production code.**

Next:

```text
05-VPC-Automation.md
```

will cover VPCs, subnets, route tables, internet/NAT gateways, security groups, network ACLs, ENIs, VPC endpoints, network discovery, connectivity audits and production network automation.
