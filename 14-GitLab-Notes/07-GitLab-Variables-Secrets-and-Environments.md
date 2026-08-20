# GitLab Variables, Secrets and Environments

> Production-focused guide to GitLab CI/CD variables, protected variables, masked secrets, environment-scoped configuration, AWS credentials, OIDC, ECR, EKS, Terraform, ArgoCD, secret rotation, security boundaries, troubleshooting, and senior DevOps interview scenarios.

---

## 1. Why Variables Matter

CI/CD pipelines need configuration that changes by:

- project
- branch
- environment
- deployment target
- region
- account
- application
- security context

Instead of hard-coding values:

```yaml
script:
  - deploy --environment production
```

use controlled variables:

```yaml
script:
  - deploy --environment "$DEPLOY_ENV"
```

Variables separate configuration from pipeline logic.

---

## 2. What Is a GitLab CI/CD Variable?

A variable is a key/value configuration value made available to pipeline jobs.

Example:

```text
APP_ENV=staging
AWS_REGION=ap-south-1
```

A shell command can consume it:

```bash
echo "$APP_ENV"
```

Sensitive values should be protected appropriately.

---

## 3. Variable Categories

Common sources include:

```text
Instance/group variables
Project variables
Environment-scoped variables
Pipeline variables
Job variables
Predefined CI/CD variables
External secret integrations
```

Exact precedence depends on the variable source and GitLab configuration.

---

## 4. Variables in `.gitlab-ci.yml`

Example:

```yaml
variables:
  APP_ENV: "test"
  AWS_REGION: "ap-south-1"
```

This is appropriate for non-sensitive configuration.

Do not store:

```text
password
private key
AWS secret access key
production token
```

in the repository.

---

## 5. Project Variables

Project variables are useful when configuration belongs to one repository.

Example:

```text
Project
 └── Variables
      ├── APP_ENV
      ├── AWS_REGION
      └── DEPLOY_ROLE
```

Sensitive project variables should be configured using the appropriate protection/masking options.

---

## 6. Group Variables

Group variables can provide shared configuration:

```text
DevOps Group
 ├── Application A
 ├── Application B
 └── Application C
        ↓
     Shared variables
```

Useful for common non-secret configuration.

Be careful with shared secrets because one compromised project may increase the blast radius.

---

## 7. Instance-Level Variables

At larger GitLab installations, instance-level variables can provide organization-wide values.

Use these sparingly.

Global variables can affect many projects unexpectedly.

---

## 8. Environment-Scoped Variables

A variable can be associated with a specific environment.

Concept:

```text
DATABASE_URL
 ├── dev → dev database
 ├── staging → staging database
 └── production → production database
```

This is much safer than:

```text
DATABASE_URL = one value for every environment
```

---

## 9. Development vs Production Variables

Example:

```text
DEV_AWS_ROLE
STAGING_AWS_ROLE
PROD_AWS_ROLE
```

Each environment should have separate access boundaries.

Do not use one powerful production credential for all environments.

---

## 10. Variable Naming Convention

Good:

```text
AWS_REGION
AWS_ACCOUNT_ID
ECR_REPOSITORY
EKS_CLUSTER_NAME
IMAGE_DIGEST
DEPLOY_ENV
```

Avoid:

```text
thing
value
abc
prodsecret
```

Names should describe purpose.

---

## 11. Secret vs Configuration

### Configuration

Usually safe to store as plain CI configuration:

```text
AWS_REGION=ap-south-1
APP_PORT=8080
LOG_LEVEL=INFO
```

### Secret

Protect:

```text
password
token
private key
API key
credential
certificate private key
```

Treat secrets differently.

---

## 12. Masked Variables

A masked variable is designed to reduce accidental exposure in job logs.

Example:

```text
DEPLOY_TOKEN=********
```

Masking is helpful but not a complete security boundary.

---

## 13. Masking Does Not Stop Malicious Code

If a job has access to a secret, malicious code may still attempt to exfiltrate it.

Therefore:

```text
Masking
+
Protected variable
+
Restricted job
+
Trusted runner
+
Least privilege
```

is stronger than masking alone.

---

## 14. Protected Variables

Protected variables can be restricted to protected branches/tags according to GitLab configuration.

Concept:

```text
Feature branch
   ↓
No production secret

Protected main
   ↓
Production secret available
```

This is a critical security boundary.

---

## 15. Why Protect Production Secrets?

Without protection:

```text
Developer branch
   ↓
CI job
   ↓
Production AWS credentials
```

A malicious or accidental change could gain production access.

With protection:

```text
Protected branch
   ↓
Trusted job
   ↓
Protected variable
```

---

## 16. Environment Protection

Variables alone are not enough.

Production should use:

```text
Protected environment
+
Protected variables
+
Protected branch
+
Trusted Runner
```

Each layer controls a different part of the deployment path.

---

## 17. Environment

A GitLab environment represents a deployment target.

Examples:

```text
development
staging
production
```

A deployment job can associate itself with an environment.

Concept:

```yaml
deploy_staging:
  environment:
    name: staging
```

---

## 18. Environment Lifecycle

Typical:

```text
Build
 ↓
Deploy Dev
 ↓
Deploy Staging
 ↓
Approval
 ↓
Deploy Production
```

Each environment can have different permissions and configuration.

---

## 19. Environment Name

Environment names should be predictable:

```text
dev
staging
production
```

For dynamic review environments:

```text
review/$CI_COMMIT_REF_SLUG
```

This can create temporary environments for feature branches.

---

## 20. Review Apps

A review environment can be created for an MR.

Concept:

```text
Merge Request
      ↓
Build
      ↓
Deploy Review Environment
      ↓
Reviewer Tests
      ↓
MR Merge
      ↓
Environment Cleanup
```

Useful for web applications.

---

## 21. Dynamic Environment

Concept:

```yaml
environment:
  name: review/$CI_COMMIT_REF_SLUG
```

This creates environment identity based on the branch/reference.

Be careful about:

- naming
- cleanup
- resource limits
- secrets
- DNS
- cloud cost

---

## 22. Environment URLs

A deployment can expose a URL for an environment.

Example concept:

```yaml
environment:
  name: staging
  url: https://staging.example.com
```

This improves deployment visibility.

---

## 23. Environment Actions

Environments can represent operations such as:

```text
start
stop
deploy
rollback
```

Dynamic environments can also be stopped when no longer needed.

---

## 24. Environment Cleanup

Review environments can create temporary resources.

Without cleanup:

```text
MR 1 → environment
MR 2 → environment
MR 3 → environment
...
```

Costs can grow rapidly.

Use controlled cleanup.

---

## 25. Environment Stop Jobs

A stop workflow can remove temporary resources.

Concept:

```text
Review Environment
      ↓
MR closed/merged
      ↓
Stop/Cleanup
      ↓
Cloud resources removed
```

Ensure cleanup jobs cannot accidentally target production.

---

## 26. Environment Scopes

Environment-scoped variables can map configuration to environments.

Example:

```text
AWS_ROLE
 ├── dev
 ├── staging
 └── production
```

This prevents one variable from being reused across unrelated environments.

---

## 27. Wildcard Environment Scope

Depending on GitLab capabilities, environment scopes can use patterns.

Concept:

```text
review/*
```

for review environments.

Use predictable naming to make scoped variables reliable.

---

## 28. Variable Precedence

Multiple variables may have the same name.

Conceptually:

```text
Global
 ↓
Group
 ↓
Project
 ↓
Environment/job/pipeline context
```

The exact GitLab precedence chain depends on the variable source and version.

For production troubleshooting, inspect the actual configured sources rather than assuming precedence.

---

## 29. Avoid Duplicate Variable Names

Bad:

```text
AWS_REGION defined in:
- YAML
- group
- project
- job
- environment
```

This makes debugging difficult.

Prefer a single clear source whenever possible.

---

## 30. Configuration Ownership

Define ownership:

```text
Platform
 → AWS region/account mapping

Application
 → application settings

Security
 → security thresholds

Deployment
 → environment identity
```

Ownership reduces accidental overrides.

---

## 31. AWS Credentials — Traditional Model

Old approach:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

stored as GitLab variables.

This can work, but long-lived credentials increase risk.

---

## 32. Why Long-Lived AWS Keys Are Risky

If exposed:

```text
Key
 ↓
Attacker
 ↓
AWS API
```

The key may remain valid until revoked.

Better:

```text
Short-lived credentials
```

where supported.

---

## 33. GitLab OIDC with AWS

Preferred architecture:

```text
GitLab Job
    ↓
OIDC Token
    ↓
AWS STS
    ↓
Assume IAM Role
    ↓
Temporary Credentials
    ↓
AWS API
```

No long-lived AWS access key needs to be stored for the job.

---

## 34. OIDC Trust Boundary

AWS trust policy should restrict who can assume the role.

Consider:

```text
GitLab instance
+
project
+
branch/tag
+
environment
```

Do not create an overly broad trust relationship.

---

## 35. OIDC Production Example

Concept:

```text
main
 ↓
GitLab pipeline
 ↓
OIDC token
 ↓
AWS production role
 ↓
ECR / approved AWS API
```

Feature branches should not automatically receive the same role.

---

## 36. Separate AWS Roles

Recommended:

```text
GitLab-Dev-Role
GitLab-Staging-Role
GitLab-Production-Role
```

Permissions should match each environment.

---

## 37. Least Privilege AWS Role

For ECR push, grant only required ECR actions.

Do not give:

```text
AdministratorAccess
```

just because the job needs:

```text
ECR push
```

---

## 38. AWS Account Separation

If multiple accounts are used:

```text
Dev Account
Staging Account
Production Account
```

map deployment identities separately.

This reduces blast radius.

---

## 39. AWS Region Variables

Example:

```text
AWS_REGION=ap-south-1
```

Verify the value before deployment.

A wrong region can result in:

```text
repository not found
cluster not found
resource not found
```

---

## 40. AWS Account Validation

Before production mutation:

```bash
aws sts get-caller-identity
```

Verify:

```text
Expected account
Expected role
```

Do not rely only on:

```text
AWS_ACCOUNT_ID
```

because a variable can be incorrect.

---

## 41. ECR Variables

Typical:

```text
AWS_REGION
ECR_REGISTRY
ECR_REPOSITORY
IMAGE_TAG
IMAGE_DIGEST
```

Example:

```text
ECR_REGISTRY = account.dkr.ecr.region.amazonaws.com
```

Avoid hard-coding environment-specific registry values throughout scripts.

---

## 42. EKS Variables

Typical:

```text
AWS_REGION
EKS_CLUSTER_NAME
K8S_NAMESPACE
DEPLOY_ENV
```

For GitOps:

```text
GITOPS_REPOSITORY
GITOPS_BRANCH
APP_PATH
```

These values should be validated before mutation.

---

## 43. Terraform Variables

Terraform CI may use:

```text
TF_VAR_region
TF_VAR_environment
TF_VAR_cluster_name
```

Sensitive Terraform variables should be handled securely.

Remember:

> A variable marked sensitive may be hidden in output, but sensitive data can still appear in state or artifacts if the design is wrong.

---

## 44. Terraform State and Secrets

Terraform state can contain sensitive values.

Therefore:

```text
Remote backend
+
Encryption
+
Restricted access
+
State locking
+
No accidental artifact upload
```

Do not publish Terraform state as a normal CI artifact.

---

## 45. GitOps Variables

A GitOps update job may use:

```text
GITOPS_TOKEN
GITOPS_REPOSITORY
GITOPS_BRANCH
IMAGE_DIGEST
```

The token should have only the required repository permissions.

---

## 46. GitLab API Token

If automation calls GitLab APIs, use the narrowest appropriate token type and scope.

Avoid a universal administrator token.

Example workflow:

```text
CI job
 ↓
Scoped identity
 ↓
GitLab API
 ↓
Required project operation
```

---

## 47. Project Access Token

A project-specific identity can reduce blast radius compared with a global administrator credential.

Use only the scopes required by the automation.

Rotate according to policy.

---

## 48. Deploy Tokens

Deploy tokens can support controlled access to repositories/registries depending on GitLab functionality.

Use a separate identity for a separate purpose.

Do not reuse deployment credentials across unrelated systems.

---

## 49. Secret Rotation

Secrets should have a rotation process:

```text
Create new secret
 ↓
Deploy/update consumers
 ↓
Verify
 ↓
Revoke old secret
 ↓
Monitor
```

Avoid simultaneous destructive rotation if consumers have not been updated.

---

## 50. Rotation Without Downtime

A safe pattern:

```text
Old credential
+
New credential
 ↓
Consumers updated
 ↓
Verify new
 ↓
Remove old
```

This is safer than:

```text
Revoke old
 ↓
Hope new works
```

---

## 51. Emergency Credential Rotation

If a secret is exposed:

```text
Identify secret
 ↓
Revoke/rotate immediately
 ↓
Identify affected systems
 ↓
Review audit logs
 ↓
Replace references
 ↓
Verify
 ↓
Root-cause analysis
```

Do not only delete the GitLab variable.

The credential may still be valid elsewhere.

---

## 52. Secret Leak in Git History

If a secret was committed:

```text
Deleting the file
```

does not necessarily remove it from Git history.

Treat the secret as compromised.

Then:

```text
Rotate credential
+
Clean history if required
+
Review exposure
```

---

## 53. Secret Leak in CI Logs

If a secret appears in logs:

1. Rotate it.
2. Determine who could access the logs.
3. Review artifacts/log retention.
4. Remove exposure where possible.
5. Fix the command that printed it.
6. Add preventive controls.

Never assume masking alone solves historical exposure.

---

## 54. Secret Leak in Artifact

If a secret is packaged:

```text
Artifact
 ↓
Registry/storage
 ↓
Potential exposure
```

Invalidate the credential.

Then replace the artifact.

---

## 55. Environment-Specific Secrets

Example:

```text
DEV_DB_PASSWORD
STAGING_DB_PASSWORD
PROD_DB_PASSWORD
```

Better architecture:

```text
same variable name
+
environment scope
```

Example:

```text
DB_PASSWORD
 ├── dev
 ├── staging
 └── production
```

This keeps scripts environment-neutral.

---

## 56. Environment-Scoped Secret Example

Concept:

```text
DB_PASSWORD
Environment: production
Value: ********
```

Then:

```yaml
deploy:
  environment:
    name: production
```

The production value is selected according to the configured environment scope.

---

## 57. Environment Names Must Match

If variable scope is:

```text
production
```

but job environment is:

```text
prod
```

the intended scoped variable may not be selected.

Use a consistent naming standard.

---

## 58. Review Environment Secrets

Review environments are dynamic.

Avoid giving them production credentials.

Use:

```text
review/* → test credentials
```

not:

```text
review/* → production credentials
```

---

## 59. Fork and External MR Security

Untrusted contributions should not receive sensitive protected variables.

Potential flow:

```text
External MR
 ↓
Validation
 ↓
No production credentials
```

Use isolated Runner infrastructure where appropriate.

---

## 60. Variable Exposure Through Forks

Risk:

```text
External code
 ↓
CI job
 ↓
Secret access
 ↓
Exfiltration
```

Protected variables and protected runners help, but the entire pipeline trust model must be reviewed.

---

## 61. Secret Detection

Use secret scanning in CI:

```text
Commit
 ↓
Secret detection
 ↓
Fail if policy violation
```

This complements GitLab variable controls.

---

## 62. SAST/SCA/Secret Scanning

Your DevSecOps pipeline can include:

```text
Source
 ↓
SonarQube
 ↓
Secret Detection
 ↓
Dependency/SCA checks
 ↓
Build
 ↓
Trivy
 ↓
Veracode
```

Exact tool placement depends on implementation.

---

## 63. Variable Masking and Shell Expansion

A secret can be exposed indirectly.

Examples:

```bash
echo "token=$TOKEN"
curl -v ...
set -x
env
```

Avoid commands that reveal headers, environment variables, or command arguments.

---

## 64. Secret in Error Message

A failed command may print:

```text
https://user:password@example.com
```

Avoid embedding credentials in URLs.

Use secure authentication mechanisms.

---

## 65. Secret in Terraform Plan

Terraform plans can contain sensitive configuration.

Do not publish plan files broadly.

Restrict:

```text
artifact access
retention
download permissions
```

---

## 66. Secret in Docker Layer

If a secret is passed into a Docker build incorrectly:

```text
Secret
 ↓
Docker layer
 ↓
Image history/cache
```

It may persist even if removed in a later layer.

Use secret-aware build techniques.

---

## 67. Secret in Kubernetes Manifest

Avoid:

```yaml
stringData:
  password: actual-password
```

committed to Git.

Prefer a secure secret-management workflow.

For GitOps, the repository should not become a plain-text credential store.

---

## 68. GitOps Secret Management

Possible patterns include:

```text
External secret manager
+
Kubernetes secret controller
```

or an approved encrypted-secret workflow.

The correct architecture depends on the organization's security model.

---

## 69. AWS Secrets Manager

A common AWS architecture:

```text
Application
 ↓
AWS Secrets Manager
 ↓
Secret
```

CI should not necessarily retrieve application runtime secrets.

Separate:

```text
Build credentials
Deployment credentials
Application runtime secrets
```

---

## 70. Build-Time vs Runtime Secrets

### Build-time

Needed to create the artifact.

Example:

```text
private package repository credential
```

### Runtime

Needed by the running application.

Example:

```text
database password
```

Do not bake runtime secrets into the image.

---

## 71. Deployment Credential vs Application Credential

CI may need:

```text
ECR push permission
```

The application may need:

```text
database read permission
```

These should be separate identities.

---

## 72. Environment Variable Injection

A deployment may inject:

```text
APP_ENV
LOG_LEVEL
DATABASE_HOST
```

at runtime.

Sensitive values should come from secure secret mechanisms rather than the Git repository.

---

## 73. Variable Scope and Least Privilege

A secret should be:

```text
Available only
where needed
```

Example:

```text
Production DB secret
 ↓
Production deployment/application
```

not:

```text
Every pipeline job
```

---

## 74. Separate Build and Deploy Jobs

Good pattern:

```text
Build Job
 ↓
No production secret
 ↓
Artifact

Deploy Job
 ↓
Production credentials
 ↓
Deployment
```

This reduces credential exposure.

---

## 75. Security Gate Before Secrets

Where possible:

```text
Test
 ↓
Security
 ↓
Protected deployment
 ↓
Secrets become available
```

Do not expose production credentials to jobs that can fail basic validation.

---

## 76. Production Secret Availability

A production deployment job should be:

```text
protected
+
restricted
+
audited
```

The secret should not be available to:

```text
lint
unit tests
feature branch builds
```

unless explicitly required.

---

## 77. Variable Debugging Strategy

When a variable is missing:

```text
1. Is it defined?
2. At what scope?
3. Is environment scope correct?
4. Is protected status compatible?
5. Is pipeline source allowed?
6. Is variable overridden?
7. Is job environment correct?
```

Do not print the secret.

---

## 78. Variable Missing in Feature Branch

Possible reason:

```text
Variable is protected
```

and feature branch is not protected.

This is expected behavior.

Do not immediately unprotect the secret.

---

## 79. Variable Missing in Production

Check:

```text
Production environment name
 ↓
Variable environment scope
 ↓
Protected status
 ↓
Protected branch
 ↓
Job rules
 ↓
Runner
```

The variable may exist but be unavailable by design.

---

## 80. Wrong Environment Variable Selected

Example:

```text
Job:
environment: staging
```

but variable:

```text
environment scope: production
```

The production value should not be available.

This is an environment-boundary issue, not a random CI failure.

---

## 81. Variable Override Problem

Suppose:

```text
Project:
AWS_REGION=ap-south-1
```

but job defines:

```yaml
variables:
  AWS_REGION: us-east-1
```

The job may use the overridden value according to GitLab precedence.

Avoid unnecessary duplicate definitions.

---

## 82. Predefined Variable Debugging

Safe examples:

```bash
echo "$CI_COMMIT_SHORT_SHA"
echo "$CI_COMMIT_REF_NAME"
echo "$CI_PIPELINE_ID"
echo "$CI_ENVIRONMENT_NAME"
```

Never print secret values.

---

## 83. Environment Deployment History

GitLab environments can provide deployment visibility.

Useful information:

```text
Environment
Deployment
Commit
Pipeline
Timestamp
Status
```

This helps during incident response.

---

## 84. Environment and Rollback

A production environment should have a clear recovery path:

```text
Current revision
 ↓
Known-good revision
 ↓
Rollback
 ↓
Verification
```

Rollback should use an approved immutable artifact or Git revision.

---

## 85. Environment and GitOps

For your architecture:

```text
GitLab Environment
       ↓
Deployment visibility
       ↓
GitOps repository
       ↓
ArgoCD
       ↓
EKS
```

GitLab may represent the delivery environment while ArgoCD manages Kubernetes reconciliation.

---

## 86. Production Environment Protection

Recommended controls:

```text
Protected environment
+
Authorized deployers
+
Protected variables
+
Protected branch
+
Trusted Runner
+
Immutable artifact
```

---

## 87. Environment Access Review

Regularly review:

```text
Who can deploy?
Who can approve?
Who can access variables?
Which runners can execute?
Which tokens exist?
Which roles exist?
```

Remove stale access.

---

## 88. Environment Naming Standard

Recommended:

```text
dev
staging
production
```

Dynamic:

```text
review/<branch>
```

Avoid inconsistent variants:

```text
prod
production
prd
live
```

unless there is a deliberate naming model.

---

## 89. Environment Naming for Microservices

For service-specific deployments:

```text
user/dev
user/staging
user/production

cart/dev
cart/staging
cart/production
```

or use a centralized environment model.

Choose a structure that matches GitOps and operational ownership.

---

## 90. Environment Configuration Matrix

Example:

| Setting | Dev | Staging | Production |
|---|---|---|---|
| AWS Account | Dev | Staging | Prod |
| EKS Cluster | dev-eks | stage-eks | prod-eks |
| AWS Role | Dev role | Stage role | Prod role |
| ECR | Dev repo | Stage repo | Prod repo |
| Secrets | Dev | Stage | Prod |
| Approval | Low | Review | Required |

The exact design depends on account topology.

---

## 91. Environment Promotion

Preferred:

```text
Build once
 ↓
Dev
 ↓
Staging
 ↓
Production
```

The artifact should remain immutable.

Environment configuration changes separately.

---

## 92. Environment Drift

If production configuration is manually changed:

```text
Git
 ≠
Live
```

For GitOps:

```text
Git desired state
 ↓
ArgoCD
 ↓
Live state
```

Use Git as the source of truth for managed configuration.

---

## 93. Environment Variables and GitOps

Do not use CI variables as a hidden substitute for GitOps configuration.

Good:

```text
GitOps
 → declarative desired state

CI variables
 → pipeline/runtime control values
```

Keep responsibilities clear.

---

## 94. Production Role ARN

A deployment job may use:

```text
PRODUCTION_ROLE_ARN
```

But verify the actual assumed identity:

```bash
aws sts get-caller-identity
```

Variables are configuration, not proof of identity.

---

## 95. OIDC Claim Restriction

For AWS OIDC, the trust policy should restrict appropriate claims.

Concept:

```text
GitLab issuer
+
project
+
ref/environment
+
audience
```

This prevents unrelated repositories from assuming the same production role.

---

## 96. OIDC Token Lifetime

OIDC provides short-lived identity.

Benefits:

```text
No permanent access key
 ↓
Short credential lifetime
 ↓
Reduced exposure window
```

Still protect the job because it can use the temporary credentials while they are valid.

---

## 97. OIDC Failure Troubleshooting

Check:

```text
GitLab OIDC configuration
 ↓
AWS IAM OIDC provider
 ↓
Trust policy
 ↓
Audience
 ↓
Subject/claims
 ↓
Role ARN
 ↓
STS call
```

Do not fix trust failures by making the role publicly assumable.

---

## 98. Environment-Scoped OIDC

A strong model:

```text
Dev pipeline
 ↓
Dev role

Staging pipeline
 ↓
Staging role

Production pipeline
 ↓
Production role
```

The trust policy should enforce the intended boundary.

---

## 99. Secret Rotation Automation

A controlled process can be:

```text
Scheduler
 ↓
Generate/obtain new credential
 ↓
Update secret store
 ↓
Deploy consumers
 ↓
Verify
 ↓
Revoke old
```

Automate where reliability and security benefit.

---

## 100. Secret Expiration Monitoring

Monitor:

```text
Token expiry
Certificate expiry
Credential age
Secret rotation date
```

Alert before expiration.

Do not wait for production failures.

---

## 101. Certificate Variables

TLS private keys/certificates may be stored as protected sensitive CI variables when required.

Better architecture may use:

```text
Certificate manager
+
Secret manager
```

rather than distributing private keys through CI.

---

## 102. SSH Keys in CI

Avoid embedding long-lived private SSH keys.

If SSH is required:

- use short-lived/scoped credentials
- restrict destination
- protect key
- disable unnecessary agent forwarding
- remove temporary files

---

## 103. GitLab Deploy Key vs CI Token

Use the credential appropriate to the operation.

For repository read/write automation, choose the narrowest supported identity.

Do not use a personal administrator credential for CI.

---

## 104. Personal Access Tokens in CI

Avoid personal tokens where possible.

Problems:

```text
Employee leaves
 ↓
Token ownership problem
```

Prefer:

```text
project/service identity
```

for automation.

---

## 105. Secret Ownership

Every production secret should have:

```text
Owner
Purpose
Scope
Rotation process
Expiration
Consumers
Emergency contact/process
```

This turns secrets into managed infrastructure.

---

## 106. Secret Inventory

Maintain an inventory of:

```text
Secret
 ↓
System
 ↓
Environment
 ↓
Owner
 ↓
Rotation
```

Unknown secrets become long-term security risks.

---

## 107. Variable Change Audit

Sensitive variable changes should be auditable.

Track:

```text
Who changed?
What changed?
When?
Why?
```

Use GitLab audit capabilities where available and appropriate.

---

## 108. Secret Access Monitoring

Monitor access to critical secret systems:

```text
AWS Secrets Manager
GitLab protected variables
Cloud IAM
Kubernetes secret systems
```

Unexpected access should trigger investigation.

---

## 109. Production Incident — Secret Leaked

Response:

```text
1. Identify credential.
2. Revoke/rotate.
3. Identify exposure window.
4. Review GitLab logs/audit.
5. Review AWS/Kubernetes logs.
6. Identify affected resources.
7. Replace credential references.
8. Fix pipeline.
9. Document incident.
```

---

## 110. Production Incident — Wrong AWS Role

Symptoms:

```text
AccessDenied
Resource not found
Wrong account
```

Check:

```bash
aws sts get-caller-identity
```

Then:

```text
Account
Region
Role
Trust
Permission
```

---

## 111. Production Incident — Production Secret Available to MR

This is a security incident.

Immediate actions:

```text
Stop exposure
 ↓
Revoke/rotate affected credentials
 ↓
Determine whether job accessed them
 ↓
Inspect logs
 ↓
Fix protected variable/environment rules
 ↓
Review runner trust
```

Do not merely remove the variable from the job after exposure.

---

## 112. Production Incident — Review Environment Uses Prod Secret

Immediately:

```text
Remove production access
 ↓
Rotate affected secret if exposed
 ↓
Replace with review-scoped credential
 ↓
Verify review pipeline
```

Review environments should never need production credentials for ordinary testing.

---

## 113. Production Incident — Secret Printed in Logs

Actions:

```text
Rotate secret
 ↓
Restrict logs
 ↓
Investigate access
 ↓
Remove logging command
 ↓
Review historical artifacts/logs
```

Masking does not eliminate historical exposure.

---

## 114. Production Incident — Variable Not Available

Check:

```text
Variable exists?
 ↓
Correct scope?
 ↓
Environment match?
 ↓
Protected?
 ↓
Branch protected?
 ↓
Pipeline source?
 ↓
Job environment?
```

This diagnostic sequence avoids random configuration changes.

---

## 115. Production Incident — Variable Has Wrong Value

Check:

```text
Duplicate variable definitions
 ↓
Precedence
 ↓
Job variables
 ↓
Pipeline variables
 ↓
Environment scope
 ↓
Included templates
```

Then consolidate configuration.

---

## 116. Production Incident — OIDC Role Cannot Be Assumed

Check:

```text
OIDC provider
 ↓
Audience
 ↓
Subject
 ↓
Trust policy
 ↓
Role ARN
 ↓
GitLab project/ref/environment
```

Do not widen trust conditions unnecessarily.

---

## 117. Senior Interview — Masked vs Protected Variables

> Masking reduces the chance that a variable value appears in logs. Protection controls where the variable is available, such as protected branches/tags according to configuration. Masking is not an access-control mechanism by itself.

---

## 118. Senior Interview — Why Environment-Scoped Variables?

> They allow the same logical variable name to resolve to environment-specific configuration while keeping application scripts consistent. For example, `DB_PASSWORD` can have different values for dev, staging, and production.

---

## 119. Senior Interview — How Do You Protect Production Secrets?

> I use protected variables/environment scope, protected branches and environments, trusted runners, least-privilege identities, short-lived credentials such as OIDC where supported, and separate build and deployment jobs so ordinary CI does not receive production credentials.

---

## 120. Senior Interview — Why OIDC Instead of AWS Keys?

> OIDC allows GitLab to exchange a short-lived identity token for temporary AWS credentials. This removes the need for long-lived access keys in CI and reduces the credential exposure window.

---

## 121. Senior Interview — How Do You Secure OIDC?

> I configure the AWS trust policy to accept only the intended GitLab issuer and appropriate project/ref/environment claims, restrict the IAM role permissions, and use separate roles for different environments.

---

## 122. Senior Interview — What If a Secret Is Committed to Git?

> I treat it as compromised immediately. I rotate or revoke the credential, investigate exposure, clean the repository history if required, replace references, and add secret-detection controls to prevent recurrence.

---

## 123. Senior Interview — Should CI Access Runtime Secrets?

> Only when necessary. I prefer separating build credentials, deployment credentials, and application runtime secrets. Runtime secrets should generally be retrieved by the application or deployment platform from an appropriate secret-management system.

---

## 124. Senior Interview — Why Separate Build and Deploy Jobs?

> It reduces the blast radius of production credentials. Build and test jobs can operate without production access, while only the protected deployment job receives the required production identity.

---

## 125. Senior Interview — How Do You Prevent Review Apps From Accessing Production?

> I use environment-scoped credentials, protected variables, restricted Runner access, separate AWS roles/accounts where appropriate, and review-specific configuration. Review jobs should never inherit production deployment credentials.

---

## 126. Senior Interview — How Do You Debug a Missing Variable?

> I check whether it exists, its scope, environment scope, protected status, branch, pipeline source, job environment, and possible overrides. I never print the secret itself during debugging.

---

## 127. Senior Interview — How Do You Handle Secret Rotation?

> I introduce the new credential, update consumers, verify successful use, then revoke the old credential. For zero-downtime rotation I temporarily support both credentials where the platform allows it.

---

## 128. Senior Interview — How Do You Secure Terraform Secrets?

> I avoid committing secrets, protect remote state, restrict backend access, use sensitive variables appropriately, prevent state/plan artifacts from being broadly exposed, and prefer cloud secret/identity mechanisms where possible.

---

## 129. Senior Interview — How Do You Secure GitOps Secrets?

> I do not store plaintext runtime secrets in the GitOps repository. I use an approved encrypted-secret or external-secret architecture, with access controlled by the workload identity and environment.

---

## 130. Senior Interview — What Is the Biggest Variable Security Mistake?

> Treating a secret as secure merely because it is masked. If untrusted code can access the variable, masking does not prevent exfiltration. The real control is limiting who and which jobs can receive the credential.

---

## 131. Production Variable Architecture

Recommended:

```text
                 GitLab
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
     Non-sensitive        Sensitive
     configuration        credentials
          │                   │
          ▼                   ▼
       CI YAML          Protected Variables
                              │
                     Environment Scope
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
             Dev           Staging        Production
              │               │               │
              ▼               ▼               ▼
          Dev Role        Stage Role       Prod Role
```

---

## 132. Recommended AWS Credential Architecture

```text
                 GitLab CI
                     │
                     ▼
                  OIDC
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
       Dev Role   Stage Role   Prod Role
          │          │          │
          ▼          ▼          ▼
       Dev AWS    Stage AWS    Prod AWS
```

Each role should have only the permissions required by its jobs.

---

## 133. Recommended GitOps Secret Architecture

```text
Application
     │
     ▼
Secret Management
     │
     ├── AWS Secrets Manager
     └── Approved external/encrypted secret mechanism
     │
     ▼
Kubernetes Secret
     │
     ▼
Application Pod
```

The GitOps repository stores desired configuration, not plaintext credentials.

---

## 134. Recommended CI/CD Environment Architecture

```text
             GitLab
                │
        ┌───────┼────────┐
        ▼       ▼        ▼
       Dev    Staging  Production
        │       │        │
        ▼       ▼        ▼
     AWS Role AWS Role AWS Role
        │       │        │
        ▼       ▼        ▼
      EKS      EKS      EKS
```

Use separate identities and appropriate protections.

---

## 135. Final Security Checklist

```text
[ ] No secrets in Git
[ ] No secrets in Docker images
[ ] No secrets in artifacts
[ ] No secrets in logs
[ ] Mask sensitive variables
[ ] Protect production variables
[ ] Scope variables to environments
[ ] Separate build/deploy credentials
[ ] Prefer OIDC for AWS
[ ] Least-privilege IAM
[ ] Separate environment roles
[ ] Protected environments
[ ] Protected branches
[ ] Trusted runners
[ ] Review/fork isolation
[ ] Secret scanning
[ ] Rotation process
[ ] Expiration monitoring
[ ] Audit changes
[ ] Monitor secret access
[ ] Tested emergency rotation
```

---

## 136. Final Mental Model

```text
                 Source Code
                     │
                     ▼
                  GitLab CI
                     │
          ┌──────────┴──────────┐
          ▼                     ▼
     Configuration            Secrets
          │                     │
          ▼                     ▼
       Variables        Protected / Scoped
                              │
                    ┌─────────┼─────────┐
                    ▼         ▼         ▼
                   Dev     Staging   Production
                    │         │         │
                    ▼         ▼         ▼
                 AWS Role  AWS Role  AWS Role
                    │         │         │
                    └─────────┼─────────┘
                              ▼
                          Deployment
                              │
                              ▼
                           ArgoCD
                              │
                              ▼
                             EKS
```

> **Variables control configuration; protected and environment-scoped variables control access boundaries; secrets require least privilege, short lifetimes, careful rotation, and strict separation between build, deployment, and runtime identities.**

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
├── 07-GitLab-Variables-Secrets-and-Environments.md ✓
├── 08-GitLab-Artifacts-Caching-and-Dependencies.md
├── 09-GitLab-Docker-and-Container-Registry.md
├── 10-GitLab-AWS-Integration.md
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

**Next: `08-GitLab-Artifacts-Caching-and-Dependencies.md`**
