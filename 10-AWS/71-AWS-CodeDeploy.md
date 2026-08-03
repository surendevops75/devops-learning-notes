# AWS CodeDeploy

---

# Introduction

AWS CodeDeploy is a fully managed deployment service that automates application deployments to compute services such as Amazon EC2, Amazon ECS, AWS Lambda, and on-premises servers.

Manual deployments are time-consuming, error-prone, and difficult to scale. AWS CodeDeploy automates deployments, minimizes downtime, supports rollback, and provides deployment strategies for highly available production environments.

AWS CodeDeploy integrates with

- AWS CodeCommit
- AWS CodeBuild
- AWS CodePipeline
- Amazon EC2
- Amazon ECS
- AWS Lambda
- AWS Auto Scaling
- Amazon S3
- AWS IAM
- Amazon CloudWatch

It enables reliable, repeatable, and automated software deployments.

---

# What is AWS CodeDeploy?

AWS CodeDeploy automates application deployments.

It helps organizations

- Deploy Applications
- Minimize Downtime
- Automate Rollbacks
- Standardize Releases
- Improve Deployment Reliability

Workflow

```text
Source Code

↓

Build Artifact

↓

AWS CodeDeploy

↓

Deployment Group

↓

Target Servers

↓

Application Updated
```

---

# Why AWS CodeDeploy?

Without CodeDeploy

```text
Manual Deployment

↓

Human Errors

↓

Application Downtime

↓

Failed Releases
```

Problems

- Manual Deployments
- Inconsistent Releases
- Downtime
- Difficult Rollbacks

With CodeDeploy

```text
Deployment Package

↓

CodeDeploy

↓

Automated Deployment

↓

Successful Release
```

---

# Real World Problem Statement

A company operates

- 300 EC2 Instances
- Amazon ECS Cluster
- Lambda Functions
- Multiple Production Environments

Requirements

- Automated Deployment
- Zero Downtime
- Rollback Support
- Deployment Monitoring

AWS CodeDeploy automates application deployments.

---

# Enterprise Architecture

```text
Developer

        │

CodeCommit

        │

CodeBuild

        │

CodeDeploy

        │

────────┼──────────────

│        │             │

EC2     ECS        Lambda

        │

Production
```

---

# Core Components

AWS CodeDeploy consists of

- Applications
- Deployment Groups
- Deployment Configurations
- Deployment Revisions
- AppSpec File
- Deployment Strategies
- CodeDeploy Agent

---

# Application

An application is the deployment target managed by CodeDeploy.

Examples

- Java Application
- Node.js API
- Python Service
- Lambda Function

---

# Deployment Group

A Deployment Group identifies deployment targets.

Examples

- Production EC2
- Development EC2
- ECS Cluster
- Lambda Alias

Deployment Groups use

- EC2 Tags
- Auto Scaling Groups
- ECS Services

---

# Deployment Revision

A deployment revision contains the application version.

Sources

- Amazon S3
- GitHub
- CodeCommit

Typically includes

- Application Code
- AppSpec File
- Deployment Scripts

---

# AppSpec File

The AppSpec file tells CodeDeploy how to deploy the application.

Example

```yaml
version: 0.0

os: linux

files:

- source: /

  destination: /var/www/html
```

The AppSpec file may also define lifecycle hooks.

---

# Lifecycle Hooks

Typical lifecycle events

```text
ApplicationStop

↓

BeforeInstall

↓

Install

↓

AfterInstall

↓

ApplicationStart

↓

ValidateService
```

Hooks execute custom deployment scripts.

---

# Deployment Configurations

Deployment configurations control deployment speed.

Common Configurations

- OneAtATime
- HalfAtATime
- AllAtOnce

Custom deployment configurations are also supported.

---

# Deployment Strategies

## In-Place Deployment

Application is updated on existing instances.

Workflow

```text
EC2 Instance

↓

Stop Application

↓

Deploy New Version

↓

Start Application
```

Advantages

- Lower Cost
- Simple Deployment

Disadvantages

- Temporary Downtime

---

## Blue/Green Deployment

New environment is created before switching traffic.

Workflow

```text
Blue Environment

↓

Green Environment

↓

Testing

↓

Traffic Shift

↓

Blue Removed
```

Advantages

- Zero Downtime
- Easy Rollback
- Safer Deployments

Disadvantages

- Higher Infrastructure Cost

---

# EC2 Deployment

CodeDeploy deploys applications using the CodeDeploy Agent.

Requirements

- CodeDeploy Agent Installed
- IAM Role
- EC2 Connectivity

---

# ECS Deployment

Supports Blue/Green deployments for containers.

Workflow

```text
ECS Service

↓

New Task Set

↓

Traffic Shift

↓

Old Task Set Removed
```

---

# Lambda Deployment

Supports deployment traffic shifting.

Traffic Options

- Canary
- Linear
- All-at-Once

---

# Automatic Rollback

Rollback can occur when

- Deployment Fails
- CloudWatch Alarm Triggers
- Validation Fails

Workflow

```text
Deployment Failed

↓

Rollback

↓

Previous Version Restored
```

---

# CloudWatch Integration

CloudWatch monitors deployments.

Examples

- Failed Deployment
- CPU Utilization
- Health Checks
- Application Metrics

CloudWatch alarms can trigger rollback.

---

# AWS CLI

Create Deployment

```bash
aws deploy create-deployment
```

List Applications

```bash
aws deploy list-applications
```

Get Deployment

```bash
aws deploy get-deployment \
--deployment-id d-XXXXXXXX
```

---

# Terraform

```hcl
resource "aws_codedeploy_app" "app" {

  name = "my-application"

}
```

---

# CloudFormation

```yaml
Resources:

  CodeDeployApplication:

    Type: AWS::CodeDeploy::Application
```

---

# Python (Boto3)

```python
import boto3

deploy = boto3.client("codedeploy")

response = deploy.list_applications()

print(response)
```

---

# Enterprise Production Architecture

```text
          Developers

               │

        CodeCommit

               │

         CodeBuild

               │

        AWS CodeDeploy

               │

 ┌─────────────┼──────────────┐

 │             │              │

EC2          ECS         Lambda

               │

Blue/Green Deployment

               │

Production
```

---

# Best Practices

- Prefer Blue/Green deployments for production
- Validate deployments before shifting traffic
- Use CloudWatch alarms for rollback
- Automate deployments using CodePipeline
- Store artifacts in Amazon S3
- Use IAM least privilege
- Test AppSpec lifecycle hooks
- Deploy gradually using Canary or Linear strategies
- Monitor deployment metrics
- Keep deployment scripts idempotent
- Version deployment artifacts
- Automate rollback procedures

---

# Common Mistakes

- Deploying directly to production
- Skipping validation tests
- Ignoring CloudWatch alarms
- Incorrect AppSpec file
- Missing CodeDeploy Agent
- No rollback configuration
- Hardcoding deployment scripts
- Using AdministratorAccess IAM roles
- Deploying all servers simultaneously
- Not testing Blue/Green deployments

---

# Troubleshooting

## Deployment Failed

Check

- AppSpec File
- Deployment Logs
- IAM Permissions
- Deployment Scripts

---

## CodeDeploy Agent Offline

Verify

- Agent Installed
- Agent Running
- EC2 IAM Role
- Network Connectivity

---

## Rollback Failed

Check

- Previous Revision
- Deployment Configuration
- CloudWatch Alarm
- Artifact Availability

---

## Lifecycle Hook Failed

Verify

- Script Permissions
- Exit Codes
- Script Path
- Application Logs

---

## ECS Deployment Failed

Check

- Task Definition
- Load Balancer
- Health Checks
- Target Group

---

# Interview Questions

## Basic

1. What is AWS CodeDeploy?
2. Why use CodeDeploy?
3. What is a Deployment Group?
4. What is an AppSpec file?
5. What is Blue/Green deployment?
6. What is In-Place deployment?
7. What is the CodeDeploy Agent?
8. What are lifecycle hooks?
9. How does rollback work?
10. Which compute services integrate with CodeDeploy?

---

## Intermediate

11. Explain Deployment Groups.
12. Explain deployment configurations.
13. Explain AppSpec lifecycle hooks.
14. Explain Blue/Green deployments.
15. Explain ECS deployments.
16. Explain Lambda deployments.
17. Explain CloudWatch integration.
18. Explain rollback strategy.
19. Explain deployment monitoring.
20. Explain deployment automation.

---

## Advanced

21. Design enterprise deployment architecture using CodeDeploy.
22. Explain CodeDeploy vs Jenkins deployments.
23. Design zero-downtime deployment strategy.
24. Explain Canary vs Linear deployment.
25. Design secure deployment pipelines.
26. Explain Blue/Green architecture.
27. Design enterprise rollback procedures.
28. Explain operational deployment best practices.
29. Design multi-region deployment architecture.
30. Best practices for AWS CodeDeploy.

---

# Production Scenarios

### Scenario 1

Your production application runs on 500 EC2 instances.

How would Blue/Green deployment reduce deployment risk?

---

### Scenario 2

A deployment fails after updating half the production servers.

How would CodeDeploy recover automatically?

---

### Scenario 3

Your ECS application requires zero-downtime deployments.

Which deployment strategy would you choose?

---

### Scenario 4

A Lambda function must gradually shift traffic to a new version.

Which deployment options would CodeDeploy provide?

---

### Scenario 5

An application deployment repeatedly fails during the **AfterInstall** lifecycle hook.

Which logs and configuration files would you investigate?

---

### Scenario 6

An enterprise requires automated deployments, rollback, CloudWatch monitoring, and CI/CD integration.

How would CodeDeploy integrate with CodeBuild and CodePipeline to satisfy these requirements?

---

# Cheat Sheet

| Component | Purpose |
|-----------|---------|
| Application | Deployment Target |
| Deployment Group | Target Resources |
| AppSpec File | Deployment Instructions |
| Lifecycle Hooks | Deployment Scripts |
| Blue/Green | Zero-Downtime Deployment |
| In-Place | Update Existing Servers |
| CodeDeploy Agent | EC2 Deployment Agent |
| CloudWatch | Monitoring & Rollback |
| Amazon S3 | Artifact Storage |
| CodePipeline | Deployment Automation |

---

# Summary

AWS CodeDeploy is a fully managed deployment service that automates software deployments to Amazon EC2, Amazon ECS, AWS Lambda, and on-premises servers. By supporting deployment groups, AppSpec files, lifecycle hooks, Blue/Green and In-Place deployment strategies, automatic rollback, CloudWatch integration, and seamless integration with CodeCommit, CodeBuild, and CodePipeline, CodeDeploy enables organizations to deliver applications reliably with minimal downtime and improved operational efficiency.