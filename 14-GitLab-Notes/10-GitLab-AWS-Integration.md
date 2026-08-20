# 13-GitLab — 10 GitLab AWS Integration

> Production-oriented guide to integrating GitLab CI/CD with AWS services, IAM, OIDC, STS, EC2, VPC, S3, ECR, EKS, RDS, Route 53, ALB, Terraform, CloudFormation, secrets, multi-account architecture, security, troubleshooting, and senior DevOps interview scenarios.

---

## 1. Why Integrate GitLab with AWS?

GitLab CI can automate AWS infrastructure and application delivery.

Typical architecture:

```text
Developer
   ↓
GitLab
   ↓
GitLab Runner
   ↓
AWS Identity
   ↓
AWS Services
```

Examples:

```text
ECR
EKS
S3
EC2
RDS
ALB
Route 53
Terraform
```

---

## 2. GitLab AWS Integration Models

Common authentication approaches:

```text
Static access keys
OIDC / web identity
Instance/workload identity
Assumed IAM roles
```

For GitLab CI, short-lived OIDC-based AWS credentials are preferred where supported.

---

## 3. AWS Authentication

The CI job must authenticate before calling AWS APIs.

Typical secure flow:

```text
GitLab Job
   ↓
OIDC token
   ↓
AWS STS
   ↓
IAM Role
   ↓
Temporary credentials
   ↓
AWS API
```

---

## 4. Why Avoid Static AWS Keys?

Static keys create:

```text
Long-lived credential
 ↓
Stored in CI
 ↓
Potential exposure
```

If leaked, the attacker can use them until revoked or expired according to the credential configuration.

OIDC reduces this risk.

---

## 5. AWS STS

AWS Security Token Service provides temporary credentials.

Concept:

```text
Identity
 ↓
STS
 ↓
Temporary credentials
 ↓
AWS API
```

Temporary credentials typically include:

```text
Access key ID
Secret access key
Session token
```

---

## 6. IAM Role

An IAM role defines:

```text
Who can assume it
+
What permissions the role has
```

For GitLab:

```text
GitLab OIDC
 ↓
IAM trust policy
 ↓
Role
 ↓
IAM permissions
```

---

## 7. IAM Trust Policy

Trust policy answers:

```text
Who can assume this role?
```

Permission policy answers:

```text
What can the role do?
```

Do not confuse the two.

---

## 8. IAM Permission Policy

Example concept:

```text
GitLab production role
 ↓
ECR push
 ↓
Specific repository
```

Avoid:

```text
AdministratorAccess
```

when only ECR access is required.

---

## 9. GitLab OIDC

OIDC allows GitLab to issue an identity token for a CI job.

AWS validates the token through an IAM OIDC provider.

Flow:

```text
GitLab
 ↓
OIDC token
 ↓
AWS IAM OIDC provider
 ↓
STS AssumeRoleWithWebIdentity
 ↓
Temporary AWS credentials
```

---

## 10. OIDC Trust Restrictions

Restrict the trust relationship using appropriate GitLab claims such as:

```text
issuer
audience
project
ref
environment
```

Exact claim names depend on the GitLab/AWS integration configuration.

---

## 11. Environment-Specific AWS Roles

Recommended:

```text
GitLab
 ├── Dev Role
 ├── Staging Role
 └── Production Role
```

Each role should have different permissions.

---

## 12. AWS Account Separation

A strong enterprise model:

```text
Dev AWS Account
Staging AWS Account
Production AWS Account
```

GitLab assumes:

```text
Dev Role
Stage Role
Prod Role
```

This creates a strong account-level boundary.

---

## 13. AWS Region Variable

Example:

```text
AWS_REGION=ap-south-1
```

Use a consistent variable name across jobs.

Always validate the region before destructive operations.

---

## 14. AWS Account Validation

Before production changes:

```bash
aws sts get-caller-identity
```

Verify:

```text
Account ID
Role ARN
Principal
```

This is one of the most useful CI safety checks.

---

## 15. AWS CLI Installation

Runner jobs need AWS CLI if they directly call AWS APIs.

Possible approaches:

```text
Prebuilt CI image
+
AWS CLI
```

or:

```text
Runner image containing AWS CLI
```

Prefer immutable approved CI images.

---

## 16. AWS CLI Version

Pin or centrally manage the AWS CLI version when reproducibility matters.

Avoid uncontrolled tool changes in production pipelines.

---

## 17. AWS SDK

Applications/scripts can also use AWS SDKs.

Examples:

```text
Python → boto3
Java → AWS SDK
Node.js → AWS SDK
```

Use workload identity rather than embedding credentials.

---

## 18. GitLab CI AWS Variables

Common non-secret variables:

```text
AWS_REGION
AWS_ACCOUNT_ID
ECR_REGISTRY
ECR_REPOSITORY
EKS_CLUSTER_NAME
S3_BUCKET
```

Sensitive authentication should preferably be provided through OIDC/workload identity.

---

## 19. AWS Resource Naming

Use predictable names:

```text
company-environment-service
```

Example:

```text
roboshop-prod-user
```

Naming improves:

- automation
- cost allocation
- troubleshooting
- ownership

---

## 20. AWS Tags

Automate AWS resource tagging.

Useful tags:

```text
Environment
Application
Owner
Project
ManagedBy
CostCenter
```

Example:

```text
ManagedBy=Terraform
Environment=production
```

---

## 21. AWS EC2 Integration

GitLab CI can automate EC2 operations.

Examples:

```text
AMI creation
Instance validation
ASG operations
Deployment
Health checks
```

Use IAM roles rather than static keys.

---

## 22. EC2 Deployment

A legacy deployment model:

```text
GitLab
 ↓
Build
 ↓
Artifact
 ↓
SSH/SSM
 ↓
EC2
```

For modern container platforms, prefer:

```text
GitLab
 ↓
ECR
 ↓
EKS
```

when Kubernetes is the target architecture.

---

## 23. AWS Systems Manager

SSM can avoid direct SSH access.

Concept:

```text
GitLab
 ↓
AWS IAM
 ↓
SSM
 ↓
EC2
```

This can reduce exposed SSH access.

---

## 24. SSM vs SSH

### SSH

```text
CI
 ↓
SSH port
 ↓
Server
```

### SSM

```text
CI
 ↓
AWS API
 ↓
SSM
 ↓
Server
```

SSM can simplify access control and auditing.

---

## 25. EC2 Health Automation

Example workflow:

```text
GitLab scheduled pipeline
 ↓
AWS CLI
 ↓
EC2 status checks
 ↓
Report
```

This is useful for operational automation.

---

## 26. Auto Scaling Groups

GitLab can trigger controlled ASG operations where required.

Example:

```text
Deployment
 ↓
ASG refresh
 ↓
New instances
 ↓
Health validation
```

Be careful with production capacity.

---

## 27. S3 Integration

S3 is useful for:

```text
Artifacts
Backups
Terraform state
Reports
Logs
Packages
```

Use separate buckets or prefixes based on trust boundaries.

---

## 28. Terraform Backend in S3

Your Terraform architecture can use:

```text
GitLab CI
 ↓
Terraform
 ↓
S3 backend
```

Protect:

```text
bucket
state
IAM
encryption
versioning
```

---

## 29. Terraform State Security

Terraform state may contain sensitive information.

Use:

```text
S3 encryption
+
restricted IAM
+
versioning
+
state locking where configured
```

Never publish state as a normal CI artifact.

---

## 30. S3 Bucket Policy

Restrict access to trusted identities.

Avoid public access.

Use:

```text
Block Public Access
```

for private infrastructure buckets where applicable.

---

## 31. S3 Versioning

Enable versioning for important state/data buckets.

Benefits:

```text
Accidental overwrite
 ↓
Previous object version
 ↓
Recovery
```

Versioning does not replace access control.

---

## 32. S3 Lifecycle

Use lifecycle rules for:

```text
Temporary CI artifacts
Build reports
Old backups
Cache
```

Do not apply aggressive lifecycle policies to Terraform state without understanding recovery requirements.

---

## 33. ECR Integration

Architecture:

```text
GitLab CI
 ↓
OIDC
 ↓
AWS IAM
 ↓
ECR
 ↓
Docker Image
```

Use repository-specific permissions.

---

## 34. ECR Build Flow

```text
Docker build
 ↓
Trivy
 ↓
AWS identity
 ↓
ECR login
 ↓
Docker push
 ↓
Digest
```

Capture the digest for deployment.

---

## 35. ECR Repository Policy

For cross-account workflows:

```text
Source account
 ↓
IAM
 ↓
ECR repository policy
 ↓
Destination repository
```

Review both identity and resource policies.

---

## 36. ECR Lifecycle Policy

Keep:

```text
Release images
Rollback images
Required history
```

Delete:

```text
Old CI images
Unreferenced development images
```

according to policy.

---

## 37. EKS Integration

Common production architecture:

```text
GitLab
 ↓
ECR
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
```

GitLab should generally not require cluster-admin permissions.

---

## 38. EKS Authentication

AWS CLI can retrieve EKS access configuration.

Concept:

```bash
aws eks update-kubeconfig ...
```

But direct CI-to-cluster access should be minimized in GitOps architectures.

---

## 39. EKS IAM vs Kubernetes RBAC

Two authorization layers exist:

```text
AWS identity
      ↓
EKS authentication/access
      ↓
Kubernetes RBAC
```

A valid AWS identity does not automatically mean unrestricted Kubernetes access.

---

## 40. EKS Access Entries

Modern EKS environments can use EKS access management mechanisms to map IAM identities to Kubernetes permissions.

Use least privilege.

Avoid giving every CI role cluster-admin.

---

## 41. Direct `kubectl` Deployment

Legacy/direct model:

```text
GitLab
 ↓
kubectl
 ↓
EKS
```

This can be valid in some environments, but creates a larger CI-to-cluster security boundary.

---

## 42. GitOps Deployment

Preferred for your architecture:

```text
GitLab
 ↓
Build/Scan/Push
 ↓
GitOps repository
 ↓
ArgoCD
 ↓
EKS
```

CI needs repository write capability rather than unrestricted cluster access.

---

## 43. GitLab + ArgoCD

Flow:

```text
GitLab CI
   │
   ├── Build
   ├── Test
   ├── Scan
   └── Push ECR
          │
          ▼
    GitOps Repository
          │
          ▼
        ArgoCD
          │
          ▼
         EKS
```

This separates CI from CD reconciliation.

---

## 44. EKS Cluster Validation

After deployment, validate:

```bash
kubectl get nodes
kubectl get pods -A
kubectl get deployments -A
```

Use targeted namespace checks in production.

---

## 45. EKS Pod Health

Check:

```bash
kubectl get pods -n <namespace>
```

Then:

```bash
kubectl describe pod <pod> -n <namespace>
```

Look for:

```text
ImagePullBackOff
CrashLoopBackOff
OOMKilled
Readiness failures
Scheduling problems
```

---

## 46. AWS ALB Integration

Your architecture uses ALB ingress.

Typical flow:

```text
Internet
 ↓
AWS ALB
 ↓
Kubernetes Ingress
 ↓
Service
 ↓
Pod
```

GitLab CI should build/deploy the application configuration, while AWS Load Balancer Controller manages the ALB resources.

---

## 47. ALB Infrastructure

Terraform can manage:

```text
VPC
Subnets
Security Groups
EKS
IAM
ALB-related infrastructure
```

Kubernetes manifests/Helm can define application ingress behavior.

Keep infrastructure and application responsibilities clear.

---

## 48. RDS Integration

GitLab/Terraform can provision RDS.

Typical:

```text
Terraform
 ↓
RDS
```

Application:

```text
EKS
 ↓
RDS
```

Database credentials should come from a secure secret-management system.

---

## 49. RDS Deployment Safety

Never make destructive database changes automatically without safeguards.

Use:

```text
plan
review
approval
backup
migration strategy
```

Production database changes need stronger controls than application image deployment.

---

## 50. Route 53 Integration

Terraform can manage:

```text
Hosted zones
DNS records
Aliases
```

Example architecture:

```text
Route 53
 ↓
ALB
 ↓
EKS
```

DNS changes should be reviewed before production application changes.

---

## 51. VPC Integration

Terraform can create:

```text
VPC
Subnets
Route tables
Internet Gateway
NAT Gateway
Security Groups
```

GitLab CI executes Terraform using an AWS identity with appropriate permissions.

---

## 52. VPC Security

CI should not automatically have unrestricted networking access.

Use:

```text
private subnets
security groups
NAT
VPC endpoints
```

according to architecture.

---

## 53. AWS Private Endpoints

Private workloads may access AWS services using VPC endpoints where appropriate.

Benefits:

```text
Reduced internet dependency
Controlled traffic path
Potential cost/security improvements
```

Evaluate endpoint costs and service availability.

---

## 54. Terraform + GitLab AWS Flow

```text
Developer
 ↓
GitLab MR
 ↓
Terraform fmt
 ↓
Terraform validate
 ↓
Terraform plan
 ↓
Security checks
 ↓
Review
 ↓
Protected apply
 ↓
AWS
```

This is a strong infrastructure pipeline.

---

## 55. Terraform Plan Artifact

The plan can be stored as a protected artifact for review.

Do not expose it broadly because it may contain sensitive information.

---

## 56. Terraform Apply Protection

Production:

```text
MR
 ↓
Plan
 ↓
Review
 ↓
Approval
 ↓
Protected job
 ↓
Apply
```

Do not give feature branches production apply permissions.

---

## 57. Terraform State Locking

State concurrency must be controlled.

Modern Terraform supports locking behavior through supported backends/configuration.

Use the backend's documented locking capabilities.

---

## 58. Terraform Module Architecture

Your infrastructure can use modules:

```text
VPC
Security
Bastion
ECR
ACM
ALB
EKS
RDS
S3
```

GitLab CI orchestrates the Terraform workflow.

---

## 59. AWS Credentials for Terraform

Prefer:

```text
GitLab OIDC
 ↓
AWS role
 ↓
Terraform
```

instead of:

```text
AWS access key in GitLab variable
```

---

## 60. Terraform Provider Credentials

Terraform can obtain AWS credentials through the AWS provider's supported credential chain.

Avoid hard-coding:

```hcl
access_key = "..."
secret_key = "..."
```

---

## 61. Terraform Environment Variables

Common AWS environment variables:

```text
AWS_REGION
AWS_ROLE_ARN
AWS_WEB_IDENTITY_TOKEN_FILE
```

The exact OIDC setup determines which variables are exported.

---

## 62. AWS IAM Policy for Terraform

Terraform often needs broader permissions than application deployment because it manages infrastructure.

Still:

```text
Terraform role
 ≠
Administrator by default
```

Scope permissions to the managed infrastructure where practical.

---

## 63. Terraform Destroy Protection

Never allow arbitrary production destroy from a normal branch.

Use:

```text
protected environment
manual approval
restricted role
separate workflow
```

---

## 64. AWS Secrets Manager

Use Secrets Manager for runtime secrets where appropriate.

Architecture:

```text
Application
 ↓
IAM identity
 ↓
Secrets Manager
 ↓
Secret
```

Do not copy runtime secrets into Docker images.

---

## 65. Parameter Store

AWS Systems Manager Parameter Store can store configuration and secrets depending on requirements.

Use appropriate encryption and access controls.

---

## 66. Secrets in GitLab vs AWS

Prefer GitLab variables for:

```text
CI-specific credentials
pipeline configuration
temporary deployment identity

```

Prefer AWS secret services for:

```text
runtime application secrets
database credentials
application API secrets
```

Keep ownership clear.

---

## 67. AWS KMS

KMS can support encryption of:

```text
S3
EBS
RDS
Secrets Manager
other AWS resources
```

GitLab CI may provision KMS resources through Terraform.

---

## 68. KMS Permissions

Avoid broad:

```text
kms:*
```

permissions unless genuinely required.

Use key policies and IAM carefully.

---

## 69. AWS CloudTrail

CloudTrail provides API audit events.

For CI troubleshooting:

```text
GitLab job
 ↓
AWS API
 ↓
CloudTrail
```

This can help identify:

```text
Who called?
Which role?
Which resource?
When?
```

---

## 70. AWS Audit Correlation

Correlate:

```text
GitLab pipeline ID
+
commit SHA
+
AWS role
+
CloudTrail event
```

This creates a useful production audit trail.

---

## 71. AWS Cost Awareness

CI/CD can generate:

```text
EC2 cost
EKS cost
NAT Gateway cost
S3 storage
ECR storage
data transfer
```

Optimize:

```text
Runner capacity
image size
cache
resource lifecycle
```

---

## 72. GitLab Runner in AWS

Runner architecture:

```text
GitLab
 ↓
Runner
 ↓
AWS
```

Runner can run:

```text
EC2
EKS
```

depending on executor and design.

---

## 73. Runner on EC2

Architecture:

```text
EC2
 ↓
GitLab Runner
 ↓
Docker/Shell executor
```

Secure with:

```text
IAM role
private subnet
security groups
patching
monitoring
```

---

## 74. Runner on EKS

Architecture:

```text
EKS
 ↓
GitLab Runner
 ↓
Ephemeral job Pods
```

Benefits:

- elasticity
- isolation
- Kubernetes scheduling
- easier horizontal scaling

---

## 75. Runner AWS IAM

Runner infrastructure itself may need AWS access.

Use:

```text
Runner identity
+
job identity
```

separately where possible.

Do not give the Runner administrator privileges just because one job needs AWS.

---

## 76. Runner Job Identity

A job may need:

```text
ECR push
```

while another needs:

```text
Terraform
```

Separate roles can reduce blast radius.

---

## 77. AWS OIDC Environment Boundary

Example:

```text
feature/*
 → no prod role

main
 → staging role

release/protected
 → production role
```

Enforce this through GitLab protection and AWS trust conditions.

---

## 78. Production AWS Role

Production role should have:

```text
minimal actions
+
minimal resources
+
trusted GitLab claims
```

Avoid:

```text
Principal: *
```

or broad trust conditions.

---

## 79. AWS Multi-Account Pipeline

Example:

```text
GitLab
   │
   ├── Dev Account
   │
   ├── Stage Account
   │
   └── Production Account
```

Use separate roles and policies.

---

## 80. Cross-Account Role Assumption

Flow:

```text
GitLab OIDC
 ↓
CI role
 ↓
STS
 ↓
Target account role
 ↓
AWS resource
```

Keep trust relationships explicit.

---

## 81. AWS Organization Integration

Enterprise environments may use:

```text
AWS Organizations
Control Tower
Service Control Policies
```

SCPs can restrict what even an IAM role can do.

CI permissions must work within those organizational controls.

---

## 82. SCP Troubleshooting

If IAM appears correct but operation fails:

```text
Check SCP
 ↓
Check permission boundary
 ↓
Check session policy
 ↓
Check resource policy
```

Not every `AccessDenied` is caused by the role's identity policy.

---

## 83. IAM Permission Boundaries

A permission boundary limits maximum permissions an IAM principal can receive.

This can be useful for controlled automation roles.

---

## 84. AWS Resource Policies

Some services use resource-based policies.

Examples:

```text
S3 bucket policy
ECR repository policy
KMS key policy
Secrets Manager resource policy
```

Authorization may depend on both identity and resource policies.

---

## 85. AWS Session Tags

Where supported, session tags can improve identity tracking.

Useful for:

```text
pipeline
project
environment
deployment
```

This can improve auditability.

---

## 86. AWS CLI Safety

For destructive commands:

```text
Validate account
Validate region
Validate resource
Require approval where appropriate
```

Example:

```bash
aws sts get-caller-identity
```

before:

```bash
aws eks ...
```

or other production mutation.

---

## 87. Production Guardrails

Examples:

```text
PRODUCTION=true
```

alone is not a security control.

Use:

```text
protected environment
+
protected branch
+
AWS role trust
+
IAM least privilege
+
manual approval
```

---

## 88. GitLab Scheduled AWS Jobs

Scheduled pipelines can automate:

```text
EC2 health checks
S3 cleanup
ECR cleanup reports
AWS inventory
cost reports
certificate checks
```

Use a dedicated read-only AWS role for monitoring jobs.

---

## 89. Read-Only AWS Role

For health checks:

```text
Describe
List
Get
```

permissions may be sufficient.

Do not grant:

```text
Delete
Update
Put
```

unless required.

---

## 90. AWS Resource Inventory

A scheduled GitLab job can collect:

```text
EC2
EKS
ECR
S3
RDS
ALB
```

and produce a report artifact.

Use read-only permissions.

---

## 91. AWS Backup Automation

GitLab can orchestrate backup workflows, but backup execution should use dedicated IAM roles and verify success.

Example:

```text
Schedule
 ↓
Backup API
 ↓
Validate
 ↓
Report
```

---

## 92. S3 Backup Verification

Do not assume:

```text
backup command succeeded
```

means backup is usable.

Verify:

```text
object exists
size
version
checksum where applicable
restore test
```

---

## 93. EBS Snapshot Automation

CI can trigger controlled snapshots.

Production safeguards:

```text
resource identification
tags
approval
retention
restore validation
```

---

## 94. RDS Backup Validation

For databases:

```text
backup created
 ↓
retention policy
 ↓
restore test
```

A backup that has never been restored is not fully validated.

---

## 95. Route 53 Change Automation

For DNS:

```text
Terraform plan
 ↓
Review
 ↓
Apply
 ↓
DNS validation
```

Avoid ad-hoc production DNS mutation from arbitrary branches.

---

## 96. ALB Validation

After infrastructure/application changes:

```text
ALB
 ↓
Target group
 ↓
Target health
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

Validate the complete request path.

---

## 97. AWS Network Troubleshooting

If CI cannot reach AWS:

```text
DNS
 ↓
Route
 ↓
Security Group
 ↓
NACL
 ↓
NAT/VPC endpoint
 ↓
AWS service
```

Check the entire path.

---

## 98. AWS Security Group

Security groups should allow only required traffic.

Do not open:

```text
0.0.0.0/0
```

for administrative ports unless explicitly justified and protected by another control.

---

## 99. NAT Gateway Dependency

Private CI runners may require NAT to reach:

```text
GitLab
Docker registries
package repositories
AWS APIs
```

NAT availability can become a CI reliability dependency.

---

## 100. VPC Endpoint Strategy

Use appropriate AWS VPC endpoints to reduce internet dependency for AWS services.

Examples may include:

```text
S3
ECR
STS
```

depending on architecture and AWS endpoint support.

---

## 101. AWS Integration Failure — `AccessDenied`

Diagnostic order:

```text
1. aws sts get-caller-identity
2. Confirm account
3. Confirm role
4. Confirm region
5. Check IAM policy
6. Check resource policy
7. Check SCP/boundary
8. Check conditions
```

This is more effective than repeatedly adding permissions.

---

## 102. AWS Integration Failure — `ResourceNotFound`

Check:

```text
Account
Region
Resource name
Environment
Terraform state
```

Many resource-not-found errors are actually wrong-region or wrong-account errors.

---

## 103. AWS Integration Failure — OIDC

Check:

```text
GitLab issuer
OIDC provider
audience
subject/claims
trust policy
role ARN
job token
```

---

## 104. AWS Integration Failure — ECR

Check:

```text
ECR registry
repository
region
identity
permissions
image tag
```

Then test authentication separately from push.

---

## 105. AWS Integration Failure — EKS

Check:

```text
AWS identity
EKS access
Kubernetes authentication
RBAC
cluster endpoint
network
namespace
```

Do not assume IAM success equals Kubernetes authorization.

---

## 106. AWS Integration Failure — S3

Check:

```text
bucket
region
IAM
bucket policy
KMS
encryption permissions
```

A KMS-protected object can fail even when S3 permissions appear correct.

---

## 107. AWS Integration Failure — KMS

Check:

```text
KMS key policy
IAM permissions
key region
encryption context
grant
```

KMS has its own authorization complexity.

---

## 108. AWS Integration Failure — Secrets Manager

Check:

```text
secret ARN
region
IAM
KMS
resource policy
```

The CI role should access only the secrets it genuinely needs.

---

## 109. AWS Integration Failure — Terraform

Check:

```text
AWS identity
backend
state lock
provider
region
permissions
resource dependency
```

Run:

```bash
terraform init
terraform validate
terraform plan
```

before apply.

---

## 110. Production AWS Change Workflow

```text
Merge Request
 ↓
Validation
 ↓
Security
 ↓
Terraform Plan
 ↓
Review
 ↓
Protected Approval
 ↓
Apply
 ↓
AWS
 ↓
Validation
```

This is the preferred pattern for infrastructure changes.

---

## 111. Production Application Workflow

```text
Code
 ↓
GitLab CI
 ↓
Test
 ↓
Docker Build
 ↓
Trivy/Security
 ↓
ECR
 ↓
Digest
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
```

This separates infrastructure provisioning from application deployment.

---

## 112. Infrastructure vs Application Separation

Infrastructure:

```text
Terraform
 ↓
AWS
```

Application:

```text
GitLab
 ↓
ECR
 ↓
GitOps
 ↓
ArgoCD
 ↓
EKS
```

This separation makes ownership and rollback clearer.

---

## 113. Terraform and ArgoCD Together

Example:

```text
Terraform
 ├── VPC
 ├── EKS
 ├── IAM
 ├── ECR
 ├── RDS
 └── ALB infrastructure

ArgoCD
 └── Kubernetes application state
```

Avoid overlapping ownership of the same resources.

---

## 114. Resource Ownership

Every resource should have one primary management system.

Example:

```text
VPC → Terraform
EKS → Terraform
ECR → Terraform
Kubernetes Deployment → ArgoCD
```

Overlapping ownership causes drift and conflicts.

---

## 115. Terraform Drift

If AWS resources are manually changed:

```text
Terraform state
 ≠
AWS
```

Run:

```bash
terraform plan
```

to identify drift.

---

## 116. GitOps Drift

If Kubernetes resources are manually changed:

```text
Git desired state
 ≠
EKS live state
```

ArgoCD can detect and reconcile drift according to configuration.

---

## 117. AWS Tagging from Terraform

Terraform can enforce common tags:

```text
Environment
ManagedBy
Application
Owner
```

This improves operational visibility.

---

## 118. AWS Cost Allocation

Use tags and account boundaries to track:

```text
CI
EKS
ECR
RDS
NAT
S3
```

This helps identify expensive pipeline architecture.

---

## 119. GitLab AWS Integration Security Model

```text
GitLab
   │
   ▼
OIDC
   │
   ▼
AWS STS
   │
   ▼
IAM Role
   │
   ├── ECR
   ├── S3
   ├── Terraform
   └── other required services
```

Every branch/environment should receive only the permissions it requires.

---

## 120. Production AWS Role Matrix

| Job | Environment | Identity | Typical Access |
|---|---|---|---|
| Build | Any | Build role | Minimal/no AWS |
| ECR Push | Dev | Dev ECR role | Push repository |
| Terraform Plan | Dev/Stage | IaC role | Read/plan |
| Terraform Apply | Production | Protected IaC role | Approved infrastructure changes |
| GitOps Update | Production | Git identity | Git repository |
| Health Check | Any | Read-only AWS role | Describe/Get/List |

This is an example architecture, not a universal policy.

---

## 121. CI Role Separation

Avoid:

```text
One role
 ↓
All AWS permissions
```

Prefer:

```text
Build role
ECR role
Terraform role
Monitoring role
```

as appropriate.

---

## 122. AWS Policy Testing

Before production:

```text
Assume role
 ↓
Run intended command
 ↓
Confirm success
 ↓
Attempt unauthorized action
 ↓
Confirm denial
```

This validates least privilege.

---

## 123. Permission Boundary Testing

If a role requires a permission:

```text
IAM policy
+
permission boundary
+
SCP
```

all must allow the operation.

Document these layered controls.

---

## 124. AWS CLI Dry Runs

Where AWS APIs support dry-run or safe validation, use them before mutation.

Not every AWS API provides a dry-run capability.

---

## 125. Production Confirmation

For destructive operations:

```text
Target account
Target region
Target resource
Target environment
```

should be explicitly validated.

---

## 126. AWS Integration Logging

CI logs should record safe metadata:

```text
AWS account
region
role
resource identifier
pipeline ID
commit SHA
```

Do not log credentials.

---

## 127. AWS Audit Correlation

A production deployment can be traced:

```text
Git commit
 ↓
Pipeline
 ↓
Runner
 ↓
IAM role
 ↓
AWS API
 ↓
EKS/ECR/AWS resource
```

This is valuable for incident response.

---

## 128. Disaster Recovery

Consider:

```text
GitLab unavailable
AWS region failure
ECR unavailable
EKS unavailable
Runner unavailable
Terraform backend unavailable
```

CI/CD should have documented recovery procedures.

---

## 129. ECR Disaster Recovery

Consider:

```text
Cross-region replication
+
retention
+
known-good digests
```

depending on availability requirements.

---

## 130. Terraform State Disaster Recovery

Protect:

```text
S3 versioning
encryption
access
backup
locking
```

Test state recovery procedures.

---

## 131. GitLab Runner Disaster Recovery

If Runner infrastructure fails:

```text
Recreate Runner
 ↓
Restore configuration
 ↓
Authenticate
 ↓
Validate AWS identity
 ↓
Run canary job
```

Infrastructure as code makes replacement easier.

---

## 132. Production Deployment Freeze

During an incident:

```text
Stop production pipelines
 ↓
Allow only approved emergency deployment
```

Protected environments can help enforce this operational model.

---

## 133. Emergency AWS Access

Emergency access should be:

```text
audited
time-limited
approved
least privilege
```

Do not permanently increase the normal CI role because of one emergency.

---

## 134. AWS Credentials Incident

If CI credentials leak:

```text
Revoke/rotate
 ↓
Identify role
 ↓
Review CloudTrail
 ↓
Check affected resources
 ↓
Review GitLab logs
 ↓
Rebuild Runner if required
```

---

## 135. Compromised GitLab Runner

If Runner compromise is suspected:

```text
Isolate Runner
 ↓
Stop jobs
 ↓
Rotate accessible credentials
 ↓
Review logs
 ↓
Rebuild from trusted image
 ↓
Validate IAM
 ↓
Resume gradually
```

---

## 136. AWS Integration Monitoring

Monitor:

```text
OIDC failures
STS failures
AccessDenied spikes
ECR push failures
EKS deployment failures
Terraform failures
S3 failures
```

Use your existing Prometheus/Grafana/ELK observability approach where applicable.

---

## 137. AWS API Throttling

Large CI systems can generate many API requests.

Symptoms:

```text
Throttling
Slow jobs
Retries
```

Solutions:

- reduce unnecessary API calls
- use retries/backoff
- control concurrency
- cache appropriate metadata
- scale carefully

---

## 138. AWS Retry Strategy

For transient failures:

```text
Request
 ↓
Retry with backoff
 ↓
Retry
 ↓
Fail after controlled limit
```

Do not retry permanent authorization errors indefinitely.

---

## 139. CI Concurrency vs AWS Limits

High pipeline concurrency can increase:

```text
ECR API calls
STS calls
EC2 API calls
Terraform API calls
```

Tune Runner concurrency with AWS service limits in mind.

---

## 140. Terraform Parallelism

Terraform can make many AWS API calls concurrently.

Use appropriate:

```bash
terraform plan
terraform apply
```

settings and avoid excessive parallelism when APIs are throttling.

---

## 141. AWS Integration Cost Optimization

Optimize:

```text
Runner size
Runner autoscaling
NAT usage
ECR storage
S3 lifecycle
EKS node capacity
Terraform execution
```

Cost should be monitored alongside reliability.

---

## 142. GitLab Scheduled Cost Report

A read-only scheduled pipeline can collect:

```text
AWS resource inventory
cost data where permitted
unused resources
ECR image counts
```

and publish a report.

Use dedicated permissions.

---

## 143. AWS Health Check Project

Example:

```text
GitLab Schedule
 ↓
Python/AWS CLI
 ↓
EC2/EKS/ECR/RDS checks
 ↓
Report
 ↓
Artifact
```

This fits the Python DevOps automation projects covered earlier.

---

## 144. AWS Resource Automation Project

Example:

```text
GitLab
 ↓
Python script
 ↓
boto3
 ↓
AWS APIs
 ↓
Resource report/action
```

Secure the script with least-privilege IAM.

---

## 145. AWS Integration with Python

For Python automation:

```python
import boto3

ec2 = boto3.client("ec2")
response = ec2.describe_instances()
```

Credentials should come from the AWS credential chain, not hard-coded values.

---

## 146. Boto3 in GitLab CI

Flow:

```text
GitLab job
 ↓
OIDC / AWS identity
 ↓
boto3
 ↓
AWS API
```

The Python script does not need embedded credentials.

---

## 147. AWS API Error Handling

Handle:

```text
AccessDenied
Throttling
ResourceNotFound
ValidationError
Timeout
```

Separate:

```text
retryable
```

from:

```text
non-retryable
```

errors.

---

## 148. Production Python + AWS

Example pattern:

```text
Python
 ↓
Validate environment
 ↓
Get AWS identity
 ↓
Perform read/action
 ↓
Validate result
 ↓
Log safe metadata
 ↓
Exit non-zero on failure
```

This makes automation reliable.

---

## 149. GitLab AWS Integration Checklist

```text
[ ] OIDC configured
[ ] AWS IAM OIDC provider configured
[ ] Trust policy restricted
[ ] Environment-specific roles
[ ] Least-privilege permissions
[ ] AWS account validation
[ ] Region validation
[ ] ECR permissions scoped
[ ] EKS access scoped
[ ] Terraform role protected
[ ] S3 backend secured
[ ] KMS access controlled
[ ] Secrets Manager protected
[ ] CloudTrail/audit available
[ ] AWS resource tagging
[ ] Runner network secured
[ ] Private connectivity evaluated
[ ] API retry strategy
[ ] Cost monitoring
[ ] Disaster recovery
```

---

## 150. Senior Interview — How Does GitLab Authenticate to AWS?

> I prefer GitLab OIDC. The CI job obtains an OIDC identity token, AWS validates it through the configured IAM OIDC provider, STS issues temporary credentials for a restricted IAM role, and the job uses those credentials for the required AWS APIs.

---

## 151. Senior Interview — Why OIDC?

> It avoids long-lived AWS access keys in CI and provides short-lived credentials. The trust policy can restrict which GitLab project, branch, tag, or environment can assume the role.

---

## 152. Senior Interview — How Do You Secure the Production AWS Role?

> I use a protected GitLab environment, protected variables where needed, trusted runners, restricted OIDC trust conditions, separate production AWS accounts or roles, and least-privilege IAM permissions.

---

## 153. Senior Interview — How Do You Deploy to EKS?

> In my GitOps architecture, GitLab builds and scans the container, pushes it to ECR, records the immutable digest, updates the GitOps repository, and ArgoCD reconciles the change into EKS. This avoids giving GitLab CI unrestricted cluster-admin access.

---

## 154. Senior Interview — How Do You Troubleshoot AWS AccessDenied?

> First I run `aws sts get-caller-identity` to confirm the actual account and role. Then I check IAM identity policy, resource policy, trust conditions, permission boundaries, SCPs, region, and resource scope. I do not blindly add AdministratorAccess.

---

## 155. Senior Interview — How Do You Secure Terraform in GitLab?

> I use a protected Terraform role, preferably through OIDC, a secure S3 backend, encryption, state locking supported by the backend, protected production apply jobs, reviewable plans, and separate roles for lower and production environments.

---

## 156. Senior Interview — How Do You Separate Infrastructure and Application Deployment?

> Terraform manages AWS infrastructure such as VPC, EKS, ECR, RDS, and supporting infrastructure. GitLab CI builds application images, ECR stores them, and ArgoCD manages Kubernetes application state through GitOps.

---

## 157. Senior Interview — Why Not Deploy Directly With `kubectl`?

> Direct CI-to-cluster deployment can work, but it gives CI credentials a larger production security boundary. With GitOps, CI updates desired state in Git and ArgoCD performs reconciliation, giving better auditability, drift management, and separation of responsibilities.

---

## 158. Senior Interview — How Do You Handle Cross-Account AWS Deployment?

> I use separate accounts and roles, establish explicit trust relationships, restrict the target role to the required resources, and validate the assumed identity before performing production changes.

---

## 159. Senior Interview — How Do You Protect Terraform State?

> I store it in a private encrypted S3 backend with restricted IAM access, versioning, appropriate locking, and recovery procedures. I never treat Terraform state as an ordinary CI artifact.

---

## 160. Senior Interview — How Do You Handle AWS API Throttling?

> I first identify which API is being throttled, reduce unnecessary concurrency, implement controlled exponential backoff for retryable operations, and tune Runner/Terraform parallelism. I don't retry authorization or validation errors indefinitely.

---

## 161. Senior Interview — How Do You Design AWS Credentials for Microservices?

> CI build credentials, deployment credentials, and application runtime credentials are separate. CI uses OIDC roles, while applications use workload identity and secret-management mechanisms appropriate to EKS.

---

## 162. Senior Interview — How Do You Validate a Production Deployment?

> I verify the Git commit, pipeline, image digest, GitOps revision, ArgoCD sync state, Kubernetes rollout, Pod readiness, service endpoints, and application health. For AWS infrastructure I also validate the intended account and region.

---

## 163. Production GitLab + AWS Architecture

```text
                           GitLab
                              │
                              ▼
                         CI Pipeline
                              │
                     ┌────────┴────────┐
                     ▼                 ▼
                  Build/Test        Terraform
                     │                 │
                     ▼                 ▼
                    ECR               AWS
                     │           ┌─────┼─────┐
                     │           ▼     ▼     ▼
                     │          VPC   EKS   RDS
                     │
                     ▼
               Image Digest
                     │
                     ▼
                GitOps Repo
                     │
                     ▼
                   ArgoCD
                     │
                     ▼
                    EKS
                     │
                     ▼
                 Kubernetes
```

---

## 164. Recommended Identity Architecture

```text
                     GitLab
                        │
                       OIDC
                        │
                        ▼
                       STS
                        │
          ┌─────────────┼─────────────┐
          ▼             ▼             ▼
       Dev Role      Stage Role     Prod Role
          │             │             │
          ▼             ▼             ▼
       Dev AWS       Stage AWS      Prod AWS
```

Each role has a separate trust and permission boundary.

---

## 165. Recommended Production Flow

```text
Developer
   ↓
Merge Request
   ↓
GitLab CI
   ├── Test
   ├── SonarQube
   ├── Trivy
   ├── Veracode
   └── Docker Build
            ↓
           ECR
            ↓
       Image Digest
            ↓
       GitOps Commit
            ↓
          ArgoCD
            ↓
           EKS
            ↓
      Application Pods
```

---

## 166. Final AWS Security Checklist

```text
[ ] No long-lived AWS keys where OIDC is practical
[ ] OIDC trust restricted
[ ] Separate environment roles
[ ] Least-privilege IAM
[ ] Account validated
[ ] Region validated
[ ] ECR scoped
[ ] EKS access scoped
[ ] Terraform apply protected
[ ] S3 backend private
[ ] KMS controlled
[ ] Secrets separated
[ ] Cross-account trust restricted
[ ] SCPs understood
[ ] Resource policies reviewed
[ ] CloudTrail/audit available
[ ] CI network path secured
[ ] API throttling handled
[ ] Cost monitored
[ ] Recovery tested
```

---

## 167. Final Mental Model

```text
                    GitLab
                       │
                       ▼
                    OIDC
                       │
                       ▼
                     STS
                       │
                 Temporary Role
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
         ECR          S3          Terraform
          │                         │
          ▼                         ▼
       Images                     AWS Infra
          │
          ▼
      GitOps Repo
          │
          ▼
        ArgoCD
          │
          ▼
         EKS
          │
          ▼
      Applications
```

> **The key production principle is to make GitLab an authenticated, least-privileged AWS client. Use OIDC and temporary credentials, separate identities by environment and responsibility, keep Terraform and application delivery boundaries clear, store images in ECR, and use GitOps/ArgoCD for Kubernetes delivery.**

---

## Section Progress

```text
13-GitLab/
├── 01-GitLab-Fundamentals.md
├── 02-GitLab-Repository-and-Git-Workflow.md
├── 03-GitLab-Branches-and-Merge-Requests.md
├── 04-GitLab-CI-CD-Fundamentals.md
├── 05-GitLab-CI-CD-Configuration.md
├── 06-GitLab-Runners.md
├── 07-GitLab-Variables-Secrets-and-Environments.md
├── 08-GitLab-Artifacts-Caching-and-Dependencies.md
├── 09-GitLab-Docker-and-Container-Registry.md
├── 10-GitLab-AWS-Integration.md ✓
├── 11-GitLab-Kubernetes-and-EKS.md
├── 12-GitLab-Terraform-and-IaC.md
├── 13-GitLab-ArgoCD-and-GitOps.md
├── 14-GitLab-DevSecOps.md
├── 15-GitLab-Security.md
├── 16-GitLab-Advanced-CI-CD.md
├── 17-GitLab-Multi-Environment-Deployments.md
├── 18-GitLab-Advanced-Pipelines.md
├── 19-GitLab-API-and-Automation.md
├── 20-GitLab-Monitoring-and-Observability.md
├── 21-GitLab-Production-Troubleshooting.md
├── 22-GitLab-Production-Architecture.md
├── 23-GitLab-DevOps-Projects.md
└── 24-GitLab-Interview-Preparation.md
```

**Next: `11-GitLab-Kubernetes-and-EKS.md`**
