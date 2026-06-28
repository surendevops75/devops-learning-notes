# AWS CLI Fundamentals and Authentication

## Introduction

AWS provides multiple ways to interact with cloud services.

Common methods:

```text
1. AWS Management Console (GUI)
2. AWS CLI
3. SDKs
4. APIs
```

As DevOps Engineers, AWS CLI is one of the most important tools because it enables automation.

---

# What is AWS CLI?

AWS CLI stands for:

```text
AWS Command Line Interface
```

It is a tool that allows us to interact with AWS services using commands.

Instead of:

```text
Login to AWS Console
Navigate Multiple Screens
Click Buttons
```

we can execute commands directly.

---

# Why AWS CLI?

Benefits:

```text
Automation
Speed
Consistency
Repeatability
Scripting Integration
```

---

# AWS Console vs AWS CLI

| AWS Console | AWS CLI |
|------------|----------|
| GUI Based | Command Based |
| Manual Operations | Automated Operations |
| Suitable for Learning | Suitable for DevOps Automation |
| Slow for Repeated Tasks | Fast for Repeated Tasks |

---

# Installing AWS CLI

Verify Installation:

```bash
aws --version
```

Example Output:

```text
aws-cli/2.x.x
```

---

# AWS Authentication

Before using AWS CLI:

```text
Authentication is Required
```

AWS must know:

```text
Who You Are
What Permissions You Have
```

---

# Authentication Components

AWS CLI typically uses:

```text
Access Key ID
Secret Access Key
Region
Output Format
```

---

# Configure AWS CLI

Command:

```bash
aws configure
```

---

# Example

```text
AWS Access Key ID:
AWS Secret Access Key:
Default Region:
Default Output Format:
```

---

# Region Example

```text
us-east-1
ap-south-1
eu-west-1
```

---

# Output Formats

Supported formats:

```text
json
text
table
yaml
```

Most commonly used:

```text
json
text
```

---

# Where Credentials Are Stored?

AWS CLI stores credentials locally.

Credential File:

```text
~/.aws/credentials
```

---

# Example

```text
[default]
aws_access_key_id=XXXX
aws_secret_access_key=YYYY
```

---

# AWS Configuration File

Location:

```text
~/.aws/config
```

Example:

```text
[default]
region=us-east-1
output=json
```

---

# Verify Authentication

After configuration:

```bash
aws s3 ls
```

---

# Understanding Output

Possible Result:

```text
Bucket List
```

or

```text
No Output
```

Both may be valid.

---

# Authentication Failure

Examples:

```text
Unable to Locate Credentials
```

---

```text
InvalidClientTokenId
```

---

```text
AccessDenied
```

These indicate authentication or authorization problems.

---

# Why Validate Authentication?

Before creating resources:

```text
Verify Credentials First
```

Otherwise automation fails later.

---

# Validation Logic

```text
Run aws s3 ls

If Success
    Continue

Else
    Stop
```

---

# Example Script

```bash
aws s3 ls >/dev/null

if [ $? -ne 0 ]; then
    echo "AWS Authentication Failed"
    exit 1
fi
```

---

# AWS CLI Command Structure

General Syntax:

```bash
aws <service> <operation> <options>
```

---

# Examples

S3:

```bash
aws s3 ls
```

---

EC2:

```bash
aws ec2 describe-instances
```

---

Route53:

```bash
aws route53 list-hosted-zones
```

---

# Services and Operations

Example:

```bash
aws ec2 describe-instances
```

Breakdown:

```text
aws          → AWS CLI

ec2          → Service

describe-instances → Operation
```

---

# AWS CLI Output Formats

Default:

```json
{
  "Instances": []
}
```

---

# Why Use Queries?

AWS returns large JSON responses.

Often we need only:

```text
Instance ID
Public IP
Private IP
```

not the entire JSON.

---

# Query Option

Syntax:

```bash
--query
```

Allows extracting specific fields.

---

# Example

```bash
aws ec2 describe-instances \
--query 'Reservations[*].Instances[*].InstanceId'
```

---

# Output Text Format

Default output:

```json
[
  [
    "i-123456789"
  ]
]
```

---

Use:

```bash
--output text
```

Output:

```text
i-123456789
```

---

# Why Use Output Text?

Useful for shell scripts.

Example:

```bash
INSTANCE_ID=$(aws ec2 describe-instances \
--query 'Reservations[0].Instances[0].InstanceId' \
--output text)
```

---

# Command Substitution

Store command output:

```bash
INSTANCE_ID=$(command)
```

---

# Example

```bash
ACCOUNT_ID=$(aws sts get-caller-identity \
--query Account \
--output text)
```

---

# Useful Validation Commands

Current Identity:

```bash
aws sts get-caller-identity
```

---

List Buckets:

```bash
aws s3 ls
```

---

List Regions:

```bash
aws ec2 describe-regions
```

---

# Real DevOps Workflow

Before Automation:

```text
Install AWS CLI
        │
        ▼
Configure Credentials
        │
        ▼
Validate Authentication
        │
        ▼
Execute AWS Commands
```

---

# Common Mistakes

## Not Running aws configure

Results:

```text
Unable to locate credentials
```

---

## Wrong Region

Example:

```text
Resources Exist

But CLI Shows Nothing
```

because wrong region is configured.

---

## Invalid Access Keys

Result:

```text
Authentication Failure
```

---

## Hardcoding Credentials

Bad Practice:

```bash
AWS_ACCESS_KEY=XXXX
AWS_SECRET_KEY=YYYY
```

Avoid storing credentials inside scripts.

---

# Best Practices

## Validate Authentication First

Example:

```bash
aws s3 ls
```

---

## Use IAM Users or Roles

Avoid root credentials.

---

## Store Credentials Securely

Use:

```text
~/.aws/credentials
```

---

## Use Queries

Retrieve only required data.

---

## Use Output Text

Simplifies scripting.

---

# Frequently Asked Interview Questions

## What is AWS CLI?

A command-line tool used to interact with AWS services.

---

## Why Use AWS CLI?

To automate cloud operations.

---

## How Do You Configure AWS CLI?

```bash
aws configure
```

---

## Where Are Credentials Stored?

```text
~/.aws/credentials
```

---

## How Do You Verify Authentication?

```bash
aws s3 ls
```

or

```bash
aws sts get-caller-identity
```

---

## What Does --query Do?

Extracts specific fields from AWS CLI output.

---

## What Does --output text Do?

Converts output into plain text format.

---

## Why Use Command Substitution?

To store AWS CLI output in variables.

---

# Key Takeaways

- AWS CLI enables cloud automation.
- Authentication is required before executing AWS commands.
- `aws configure` stores credentials and configuration.
- Credentials are stored in `~/.aws/credentials`.
- `aws s3 ls` is commonly used for authentication validation.
- AWS CLI follows the format: `aws service operation`.
- `--query` extracts specific values from responses.
- `--output text` simplifies shell scripting.
- AWS CLI is a foundational tool for DevOps engineers working with AWS.