# 15-GitOps-Secrets-Management.md

## 1. Purpose

Secrets management is one of the most important parts of production GitOps.

A GitOps repository should contain the desired configuration required to deploy an application, but it should not become a plaintext password store.

For an AWS/EKS + Argo CD environment, a strong production design separates:

```text
Application configuration
        |
        v
GitOps Repository
        |
        +--------------------+
                             |
Sensitive values             |
        |                    |
        v                    |
AWS Secrets Manager          |
        |                    |
        v                    |
External Secrets Operator    |
        |                    |
        v                    |
Kubernetes Secret <----------+
        |
        v
Application Pod
```

This file covers:

- Secret fundamentals
- Why secrets are different from configuration
- Kubernetes Secrets
- Base64 versus encryption
- Encryption at rest
- AWS Secrets Manager
- AWS KMS
- External Secrets Operator
- SecretStore
- ClusterSecretStore
- ExternalSecret
- IRSA
- EKS Pod Identity
- Secret refresh
- Secret rotation
- Secret versioning
- Multi-environment secrets
- Multi-account architecture
- Multi-cluster architecture
- Helm and secrets
- Kustomize and secrets
- Sealed Secrets
- SOPS
- Git encryption patterns
- Argo CD repository credentials
- CI credentials
- Database credentials
- API tokens
- TLS certificates
- ALB/ACM considerations
- Secret injection patterns
- Runtime secret consumption
- Secret backup and disaster recovery
- Secret leakage response
- Production YAMLs
- RoboShop integration
- Troubleshooting
- Interview preparation

---

# 2. What Is a Secret?

A secret is sensitive information that should not be exposed to unauthorized users or systems.

Examples:

```text
database passwords
API keys
OAuth client secrets
AWS credentials
private keys
TLS private keys
Git credentials
registry credentials
application tokens
encryption keys
```

---

# 3. Configuration vs Secret

Configuration:

```yaml
LOG_LEVEL: INFO
ENVIRONMENT: prod
SERVICE_PORT: 8080
```

Secret:

```text
DATABASE_PASSWORD
PAYMENT_API_TOKEN
PRIVATE_KEY
```

The distinction is based on sensitivity, not file format.

A YAML file can contain either.

---

# 4. Why Secrets Are Special

Configuration can usually be reviewed openly.

Secrets require:

```text
confidentiality
access control
rotation
audit
secure storage
controlled distribution
```

---

# 5. The GitOps Secret Problem

GitOps wants:

```text
Git = desired state
```

But storing:

```yaml
password: MyProductionPassword
```

in Git creates a security problem.

Even if the file is later deleted:

```text
Git history
```

may retain the secret.

---

# 6. Secret Lifecycle

A production secret has a lifecycle:

```text
Create
  |
  v
Store
  |
  v
Authorize
  |
  v
Retrieve
  |
  v
Consume
  |
  v
Rotate
  |
  v
Revoke
  |
  v
Destroy
```

Every stage requires controls.

---

# 7. Secret Management Principles

Use:

```text
Never hard-code
Least privilege
Short-lived credentials where possible
Centralized secret storage
Encryption
Audit
Rotation
Environment isolation
No unnecessary duplication
```

---

# 8. Secret Sources

Common sources include:

```text
AWS Secrets Manager
HashiCorp Vault
Azure Key Vault
Google Secret Manager
Kubernetes Secret
Sealed Secrets
SOPS
external secret platforms
```

For the RoboShop AWS/EKS environment, AWS Secrets Manager + External Secrets Operator is a strong practical pattern.

---

# 9. Recommended RoboShop Architecture

```text
Developer
   |
   v
GitOps Repository
   |
   +--> ExternalSecret manifest
   |
   v
Argo CD
   |
   v
EKS
   |
   v
External Secrets Operator
   |
   v
AWS Secrets Manager
   |
   v
Kubernetes Secret
   |
   v
RoboShop Pod
```

The GitOps repository contains the reference to the secret, not the plaintext value.

---

# 10. Kubernetes Secret

Kubernetes has a native Secret resource.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: cart-secrets
  namespace: roboshop
type: Opaque
stringData:
  DATABASE_PASSWORD: change-me
```

This is valid Kubernetes syntax, but the plaintext value must not be committed to Git.

---

# 11. `data` vs `stringData`

`data` expects base64-encoded values.

Example:

```yaml
data:
  PASSWORD: cGFzc3dvcmQ=
```

`stringData` accepts plaintext during manifest submission:

```yaml
stringData:
  PASSWORD: password
```

Neither makes a Git repository safe.

---

# 12. Base64 Is Not Encryption

This is a critical interview concept.

```text
Base64 != encryption
```

Anyone can decode:

```text
cGFzc3dvcmQ=
```

back into:

```text
password
```

Therefore:

```yaml
data:
  PASSWORD: ...
```

does not automatically mean the secret is secure.

---

# 13. Kubernetes Secret Storage

A Kubernetes Secret is stored as Kubernetes API data.

Whether it is encrypted at rest depends on the cluster's encryption configuration.

Therefore production security requires:

```text
RBAC
+
encryption at rest
+
network security
+
secret lifecycle controls
```

---

# 14. Encryption at Rest

A production Kubernetes cluster should protect sensitive data stored by the API server.

In AWS environments, KMS-backed encryption can be part of the EKS security design.

Conceptually:

```text
Kubernetes Secret
       |
       v
API Server
       |
       v
Encryption
       |
       v
KMS / encryption provider
       |
       v
Persistent storage
```

---

# 15. AWS KMS

AWS Key Management Service can provide managed cryptographic key infrastructure.

KMS concepts include:

```text
KMS key
key policy
IAM permissions
encryption
decryption
rotation
audit
```

---

# 16. KMS Key Policy

KMS authorization is influenced by:

```text
IAM policies
KMS key policy
grants
```

Do not grant broad:

```text
kms:*
```

permissions unless genuinely required.

---

# 17. AWS Secrets Manager

AWS Secrets Manager is designed for storing sensitive values.

Example logical secret:

```text
/roboshop/prod/cart
```

It may contain:

```json
{
  "DATABASE_USERNAME": "cart",
  "DATABASE_PASSWORD": "REDACTED",
  "DATABASE_HOST": "cart-db.internal",
  "API_TOKEN": "REDACTED"
}
```

The actual secret value belongs in Secrets Manager, not Git.

---

# 18. Secrets Manager Advantages

It provides capabilities such as:

```text
encryption
IAM access control
versioning
rotation support
audit integration
regional service availability
```

---

# 19. Secret Naming Strategy

Use predictable paths:

```text
/roboshop/dev/cart
/roboshop/qa/cart
/roboshop/prod/cart
```

For larger environments:

```text
/company/roboshop/prod/cart
/company/roboshop/prod/payment
```

---

# 20. Why Paths Matter

Paths make access policies easier to reason about.

Example:

```text
cart-prod role
    |
    +--> /roboshop/prod/cart
```

instead of:

```text
cart-prod role
    |
    +--> every secret
```

---

# 21. One Secret vs Multiple Secrets

You can store:

```text
one application secret object
```

or split values:

```text
/cart/database
/cart/api
/cart/oauth
```

Choose based on:

```text
rotation lifecycle
access boundaries
ownership
application design
```

---

# 22. Secret Boundary Design

If:

```text
database password
```

and:

```text
external API token
```

have different owners and rotation schedules, separate secrets may be cleaner.

---

# 23. External Secrets Operator

External Secrets Operator, commonly abbreviated ESO, synchronizes secret values from external providers into Kubernetes Secrets.

Architecture:

```text
ExternalSecret
       |
       v
ESO Controller
       |
       v
AWS Secrets Manager
       |
       v
Kubernetes Secret
```

---

# 24. Why ESO?

It avoids storing plaintext secrets in Git while still allowing applications to consume standard Kubernetes Secret objects.

---

# 25. ESO Components

Important concepts include:

```text
ExternalSecret
SecretStore
ClusterSecretStore
External Secrets Operator controller
provider
target Secret
```

---

# 26. ExternalSecret

`ExternalSecret` describes:

```text
which external secret
which properties
which Kubernetes Secret
how often to refresh
```

---

# 27. SecretStore

A `SecretStore` defines access to an external secret provider within a namespace.

Example concept:

```text
roboshop namespace
       |
       v
SecretStore
       |
       v
AWS Secrets Manager
```

---

# 28. ClusterSecretStore

`ClusterSecretStore` is cluster-scoped.

It can be referenced by ExternalSecrets across namespaces, subject to provider configuration and policy.

Use it carefully because it can create a wider trust boundary.

---

# 29. SecretStore vs ClusterSecretStore

| Feature | SecretStore | ClusterSecretStore |
|---|---|---|
| Scope | Namespace | Cluster |
| Isolation | Stronger | Broader |
| Reuse | Namespace | Multiple namespaces |
| Blast radius | Smaller | Larger |
| Typical use | Team/application | Platform-managed provider |

---

# 30. Production Recommendation

Use namespace-scoped `SecretStore` when teams need strong isolation.

Use `ClusterSecretStore` when:

```text
platform team controls provider access
```

and the broader trust model is intentional.

---

# 31. AWS Provider

ESO can integrate with AWS Secrets Manager.

Conceptually:

```yaml
provider:
  aws:
    service: SecretsManager
    region: ap-south-1
```

Authentication should use workload identity rather than static AWS keys.

---

# 32. Authentication Options

For EKS, use:

```text
IRSA
```

or:

```text
EKS Pod Identity
```

depending on the organization's standard.

Do not store:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

inside ESO configuration.

---

# 33. IRSA Architecture for ESO

```text
ESO Pod
  |
  v
ESO ServiceAccount
  |
  v
AWS IAM role
  |
  v
STS
  |
  v
Secrets Manager
```

---

# 34. EKS Pod Identity Architecture for ESO

```text
ESO Pod
  |
  v
ESO ServiceAccount
  |
  v
EKS Pod Identity association
  |
  v
IAM role
  |
  v
Secrets Manager
```

---

# 35. ESO IAM Permissions

ESO should receive only the permissions needed to read approved secrets.

Conceptual:

```json
{
  "Effect": "Allow",
  "Action": [
    "secretsmanager:GetSecretValue"
  ],
  "Resource": [
    "arn:aws:secretsmanager:ap-south-1:123456789012:secret:/roboshop/prod/*"
  ]
}
```

Use exact ARNs and conditions where possible.

---

# 36. Avoid Broad SecretsManager Permissions

Avoid:

```json
"Action": "secretsmanager:*",
"Resource": "*"
```

for a production application.

The ESO role should be tightly scoped.

---

# 37. KMS Permission Consideration

If the Secrets Manager secret uses a customer-managed KMS key, the identity may need appropriate KMS permissions according to the encryption configuration.

Do not automatically grant broad KMS access.

---

# 38. Example IAM Policy Concept

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadRoboShopProductionSecrets",
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": [
        "arn:aws:secretsmanager:ap-south-1:123456789012:secret:/roboshop/prod/cart-*"
      ]
    }
  ]
}
```

The wildcard behavior of Secrets Manager ARNs should be validated for the organization's naming scheme.

---

# 39. ExternalSecret Production Example

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: cart-secrets
  namespace: roboshop
spec:
  refreshInterval: 1h

  secretStoreRef:
    name: aws-secrets
    kind: SecretStore

  target:
    name: cart-secrets
    creationPolicy: Owner

  data:
    - secretKey: DATABASE_USERNAME
      remoteRef:
        key: /roboshop/prod/cart
        property: DATABASE_USERNAME

    - secretKey: DATABASE_PASSWORD
      remoteRef:
        key: /roboshop/prod/cart
        property: DATABASE_PASSWORD

    - secretKey: API_TOKEN
      remoteRef:
        key: /roboshop/prod/cart
        property: API_TOKEN
```

Use the External Secrets Operator API version supported by the installed version.

---

# 40. What Happens Internally?

When Argo CD applies the `ExternalSecret`:

```text
1. Argo CD applies ExternalSecret.
2. Kubernetes stores the ExternalSecret.
3. ESO controller observes it.
4. ESO authenticates to AWS.
5. ESO calls Secrets Manager.
6. ESO reads requested values.
7. ESO creates/updates Kubernetes Secret.
8. Pod consumes Kubernetes Secret.
```

Argo CD is responsible for desired configuration.

ESO is responsible for external secret synchronization.

---

# 41. Important Responsibility Separation

```text
Argo CD
  |
  +--> deploy ExternalSecret definition

ESO
  |
  +--> retrieve secret value

Kubernetes
  |
  +--> store resulting Secret

Application
  |
  +--> consume Secret
```

This separation is useful for production architecture.

---

# 42. Secret Refresh

Suppose:

```text
AWS password changes
```

ESO periodically checks the external source.

Then:

```text
AWS Secrets Manager
       |
       v
ESO refresh
       |
       v
Kubernetes Secret updated
```

---

# 43. Refresh Does Not Always Restart Pods

If an application consumes a Secret as an environment variable:

```text
env:
  valueFrom:
    secretKeyRef:
```

the process usually receives the value when the container starts.

Updating the Kubernetes Secret does not automatically restart the pod.

---

# 44. Secret as Volume

If a Secret is mounted as a volume, Kubernetes can update the mounted files when the Secret changes, subject to Kubernetes behavior and application handling.

The application must actually reload the changed file.

---

# 45. Environment Variable Secret Consumption

Example:

```yaml
env:
  - name: DATABASE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: cart-secrets
        key: DATABASE_PASSWORD
```

Simple and common.

---

# 46. Volume Secret Consumption

Example:

```yaml
volumes:
  - name: app-secrets
    secret:
      secretName: cart-secrets

containers:
  - name: cart
    volumeMounts:
      - name: app-secrets
        mountPath: /var/run/secrets/roboshop
        readOnly: true
```

The application reads the secret file.

---

# 47. Which Consumption Method?

Environment variables:

```text
simple
common
```

Volume:

```text
supports file-based secrets
can support application reload patterns
```

Application design determines the correct method.

---

# 48. Secret Rotation

Rotation means replacing a secret with a new value.

Example:

```text
old password
     |
     v
new password
```

---

# 49. Why Rotate?

Reasons include:

```text
scheduled security policy
employee departure
suspected exposure
compliance
credential compromise
provider requirements
```

---

# 50. Rotation Strategy

A safe rotation process:

```text
1. Generate new credential.
2. Update external system.
3. Update Secrets Manager.
4. Synchronize Kubernetes Secret.
5. Restart/reload application if required.
6. Verify traffic.
7. Revoke old credential.
```

The exact order depends on whether the external service supports overlapping credentials.

---

# 51. Zero-Downtime Rotation

For systems that support multiple valid credentials:

```text
old + new
   |
   v
deploy consumers
   |
   v
verify
   |
   v
remove old
```

This is safer than replacing the only valid credential immediately.

---

# 52. Database Password Rotation

A database rotation can involve:

```text
DB credential
Secrets Manager
ESO
application
connection pool
```

If the application caches connections, it may need:

```text
restart
connection refresh
graceful reconnect
```

---

# 53. API Token Rotation

For API tokens:

```text
create new token
 |
 v
store new token
 |
 v
sync application
 |
 v
verify
 |
 v
revoke old token
```

---

# 54. Secret Versioning

Secrets Manager supports versions.

Conceptually:

```text
Secret
 |
 +--> version A
 +--> version B
 +--> version C
```

This can support controlled rotation.

---

# 55. Version Stages

AWS Secrets Manager commonly uses staging labels such as:

```text
AWSCURRENT
AWSPREVIOUS
```

These can help identify current and previous versions.

---

# 56. Secret Version Risk

Do not assume old versions are automatically harmless.

If a previous credential remains valid:

```text
old credential
```

may still provide access.

Rotation should include revocation when appropriate.

---

# 57. Multi-Environment Secrets

Use separate paths:

```text
/roboshop/dev/cart
/roboshop/qa/cart
/roboshop/prod/cart
```

and separate IAM permissions.

---

# 58. Never Use PROD Secret in DEV

A DEV workload should not have IAM permission to:

```text
/roboshop/prod/*
```

unless there is a documented and justified exception.

---

# 59. Environment IAM Boundary

Example:

```text
DEV ESO Role
   |
   +--> /roboshop/dev/*

QA ESO Role
   |
   +--> /roboshop/qa/*

PROD ESO Role
   |
   +--> /roboshop/prod/*
```

---

# 60. Multi-Cluster Secrets

For multiple EKS clusters:

```text
EKS-DEV
  -> DEV secret store/path

EKS-QA
  -> QA secret store/path

EKS-PROD
  -> PROD secret store/path
```

This reduces cross-cluster secret exposure.

---

# 61. Multi-Account Secrets

A mature AWS architecture can use:

```text
AWS DEV Account
  -> DEV Secrets Manager

AWS QA Account
  -> QA Secrets Manager

AWS PROD Account
  -> PROD Secrets Manager
```

This provides strong environment isolation.

---

# 62. Central Secret Account

Some organizations centralize secrets:

```text
Security Account
       |
       +--> secret access policies
       |
       +--> application accounts
```

This can provide central governance but increases cross-account complexity.

---

# 63. Cross-Account Secret Access

Conceptually:

```text
EKS PROD
   |
   v
IAM Role
   |
   v
Cross-account permission
   |
   v
Secrets Manager in Security Account
```

Use explicit trust and resource policies.

---

# 64. Secret Resource Policies

AWS Secrets Manager supports resource-based policies in appropriate scenarios.

Use them carefully.

Avoid:

```text
Principal: "*"
```

for sensitive secrets.

---

# 65. Region Strategy

Secrets may need to exist in:

```text
ap-south-1
```

or another region.

For DR:

```text
primary region
       |
       v
replicated secret
       |
       v
DR region
```

Design replication deliberately.

---

# 66. Disaster Recovery

A DR strategy should define:

```text
where secrets live
how they are replicated
who can restore
how IAM is restored
how ESO reconnects
```

---

# 67. Secret Backup

Do not simply export plaintext secret values into ordinary backup storage.

Prefer:

```text
AWS-managed secret storage
encrypted backup
KMS
strict IAM
tested restore
```

---

# 68. Recovery Scenario

Suppose the primary EKS cluster is lost.

Recovery:

```text
Terraform
   |
   v
new EKS
   |
   v
Argo CD
   |
   v
ESO
   |
   v
Secrets Manager
   |
   v
Kubernetes Secrets
   |
   v
Applications
```

Externalized secrets simplify cluster rebuilds.

---

# 69. Secret Management with Terraform

Terraform may provision:

```text
Secrets Manager secret metadata
IAM roles
IAM policies
KMS keys
ESO infrastructure
```

Be careful if Terraform manages actual secret values.

---

# 70. Terraform State Risk

If a secret value is provided directly to Terraform, it may appear in:

```text
Terraform state
```

Even with a secure backend, this expands the secret exposure surface.

---

# 71. Better Terraform Pattern

Prefer Terraform to create:

```text
secret container
IAM policy
```

while secret values are populated through:

```text
secure operational workflow
```

or an approved secret-management integration.

---

# 72. Terraform State Security

For the RoboShop environment:

```text
S3 backend
```

should have:

```text
encryption
least-privilege IAM
versioning
restricted access
audit
```

and state locking according to the Terraform/backend version and configuration.

---

# 73. Argo CD and Repository Credentials

Argo CD itself needs credentials for private Git repositories.

These are also secrets.

Examples:

```text
SSH private key
Git token
GitHub App private key
```

---

# 74. Repository Credential Lifecycle

```text
Create
 |
 v
Store in Argo CD
 |
 v
Use
 |
 v
Rotate
 |
 v
Revoke
```

Use dedicated credentials, not personal administrator credentials.

---

# 75. Argo CD Repository Secret Example

Conceptual:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: repo-roboshop
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
type: Opaque
stringData:
  type: git
  url: https://github.com/company/roboshop-gitops.git
  username: gitops-readonly
  password: REDACTED
```

Do not commit a real credential.

---

# 76. Better Repository Credential Strategy

Use:

```text
read-only token
```

or:

```text
GitHub App
```

with only the required repository permissions.

---

# 77. CI Credentials

Jenkins/GitHub Actions may need:

```text
ECR push
GitOps repository write
security scanner credentials
```

Do not give one credential all permissions.

---

# 78. GitOps Bot

A GitOps bot may need:

```text
read source
write GitOps repository
```

It should not have:

```text
AWS AdministratorAccess
Kubernetes cluster-admin
```

unless there is a compelling architectural reason.

---

# 79. GitHub Actions OIDC

For AWS access, GitHub Actions can use OIDC federation so that a workflow obtains temporary AWS credentials.

Conceptually:

```text
GitHub Actions
      |
      v
OIDC token
      |
      v
AWS STS
      |
      v
IAM Role
      |
      v
ECR
```

This avoids storing long-lived AWS keys in GitHub.

---

# 80. Jenkins AWS Access

For Jenkins, prefer an equivalent short-lived workload identity or secure credential integration where available.

Avoid:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

hard-coded in Jenkinsfiles.

---

# 81. Helm Secrets

Do not place:

```yaml
database:
  password: SuperSecret
```

in:

```text
values.yaml
```

---

# 82. Helm External Secret Pattern

Use:

```yaml
database:
  existingSecret: cart-secrets
```

and create:

```yaml
ExternalSecret
```

separately.

---

# 83. Helm Template Example

Application chart:

```yaml
env:
  - name: DATABASE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: {{ .Values.existingSecret }}
        key: DATABASE_PASSWORD
```

Values:

```yaml
existingSecret: cart-secrets
```

No plaintext password is required.

---

# 84. Kustomize Secrets

Kustomize can generate Secrets, but plaintext input files still require careful handling.

Do not assume:

```text
secretGenerator
```

automatically makes the source secret safe for Git.

---

# 85. Sealed Secrets

Sealed Secrets provides a pattern where encrypted secret manifests can be committed to Git.

Architecture:

```text
Plain secret
    |
    v
Sealing process
    |
    v
Encrypted manifest
    |
    v
Git
    |
    v
Sealed Secrets controller
    |
    v
Kubernetes Secret
```

---

# 86. Sealed Secrets Advantage

Git contains:

```text
encrypted representation
```

rather than plaintext.

---

# 87. Sealed Secrets Risk

The controller's private key is extremely important.

If the sealing key is lost:

```text
existing encrypted secrets
```

may not be recoverable.

Therefore:

```text
backup
rotation strategy
DR
```

matter.

---

# 88. SOPS

SOPS can encrypt selected values in files.

A common architecture:

```text
Git
 |
 v
SOPS encrypted file
 |
 v
Argo CD / plugin/integration
 |
 v
decrypted manifest
 |
 v
Kubernetes
```

The exact integration depends on the Argo CD deployment and approved tooling.

---

# 89. SOPS Encryption Keys

SOPS can use key systems such as:

```text
AWS KMS
GCP KMS
Azure Key Vault
PGP
age
```

AWS KMS is particularly relevant for an AWS environment.

---

# 90. Sealed Secrets vs SOPS vs External Secrets

| Approach | Secret value in Git | External source | Main strength |
|---|---|---|---|
| Kubernetes Secret | Yes if plaintext | No | Native/simple |
| Sealed Secrets | Encrypted | No | Git-native encrypted secret |
| SOPS | Encrypted | Optional | File-level encryption |
| External Secrets | No | Yes | Central external secret management |

For AWS/EKS production, External Secrets + Secrets Manager is often a strong default when the organization already uses AWS-native secret management.

---

# 91. When Sealed Secrets Is Useful

Useful when:

```text
Git-centric secret lifecycle
```

is preferred and external secret infrastructure is unavailable or unsuitable.

---

# 92. When External Secrets Is Useful

Useful when:

```text
central secret rotation
AWS Secrets Manager
IAM
audit
```

are already part of the platform.

---

# 93. Secret Management Decision

For RoboShop:

```text
AWS Secrets Manager
        +
External Secrets Operator
        +
IRSA / EKS Pod Identity
```

is the recommended primary pattern for these notes.

---

# 94. SecretStore Production Example

Example:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets
  namespace: roboshop
spec:
  provider:
    aws:
      service: SecretsManager
      region: ap-south-1
```

Authentication configuration depends on the ESO version and selected AWS identity mechanism.

---

# 95. ExternalSecret with Creation Policy

```yaml
target:
  name: cart-secrets
  creationPolicy: Owner
```

`Owner` indicates ESO owns the generated Secret.

This can help make lifecycle behavior predictable.

---

# 96. ExternalSecret Refresh Policy

A typical:

```yaml
refreshInterval: 1h
```

means the controller periodically refreshes the secret.

The exact behavior depends on ESO version and configuration.

---

# 97. Refresh Interval Tradeoff

Short:

```text
1m
```

means faster propagation but more provider/API activity.

Long:

```text
24h
```

means lower API activity but slower rotation propagation.

Choose based on:

```text
security
rotation requirements
API limits
application behavior
```

---

# 98. Secret Refresh Is Not Deployment Sync

Important:

```text
Argo CD sync
```

and:

```text
ESO secret refresh
```

are different mechanisms.

Argo CD manages:

```text
ExternalSecret desired configuration
```

ESO manages:

```text
secret value synchronization
```

---

# 99. Secret Drift

Suppose someone manually changes the generated Kubernetes Secret.

ESO may restore it from the external source.

This is different from Argo CD drift correction.

---

# 100. Desired Secret Source

For the external secret architecture:

```text
AWS Secrets Manager
```

is the source of truth for the sensitive value.

Git is the source of truth for:

```text
which secret to consume
```

---

# 101. Two Sources of Truth

This is an important distinction:

```text
Git
  -> secret reference/configuration

Secrets Manager
  -> secret value
```

There is no contradiction because each system owns a different part of the state.

---

# 102. Secret Naming Convention

Use consistent names:

```text
/roboshop/{environment}/{service}
```

Example:

```text
/roboshop/prod/cart
/roboshop/prod/payment
/roboshop/prod/orders
```

---

# 103. Metadata Tags

AWS Secrets Manager resources can be tagged.

Useful metadata:

```text
Application=RoboShop
Environment=prod
Service=cart
Owner=platform
ManagedBy=terraform
```

Do not put secret values into tags.

---

# 104. Secret Ownership

Every production secret should have:

```text
owner
purpose
environment
rotation policy
support team
```

---

# 105. Secret Rotation Schedule

Do not blindly rotate everything every 30 days.

Consider:

```text
credential type
risk
provider capability
downtime risk
business requirement
```

Some credentials may support automatic rotation while others require coordinated application changes.

---

# 106. Database Secret Rotation Runbook

```text
1. Confirm database supports overlapping credentials.
2. Create new credential.
3. Store new credential in Secrets Manager.
4. Confirm ESO refresh.
5. Restart/reload application if needed.
6. Verify application connections.
7. Revoke old credential.
8. Monitor errors.
9. Record rotation.
```

---

# 107. Secret Rotation Failure

Possible symptoms:

```text
authentication failures
connection refused
HTTP 401/403
CrashLoopBackOff
application startup failures
```

Check:

```text
Secrets Manager
ESO status
Kubernetes Secret
pod environment/volume
application logs
```

---

# 108. ExternalSecret Status

Useful command:

```bash
kubectl get externalsecret -n roboshop
```

Then:

```bash
kubectl describe externalsecret cart-secrets -n roboshop
```

Look for:

```text
conditions
events
provider errors
refresh state
```

---

# 109. SecretStore Troubleshooting

```bash
kubectl get secretstore -n roboshop
kubectl describe secretstore aws-secrets -n roboshop
```

Check:

```text
provider configuration
authentication
events
```

---

# 110. ClusterSecretStore Troubleshooting

```bash
kubectl get clustersecretstore
kubectl describe clustersecretstore aws-secrets
```

Check:

```text
provider
conditions
authentication
```

---

# 111. ESO Controller Logs

Find the controller:

```bash
kubectl get pods -A | grep external-secrets
```

Then:

```bash
kubectl logs -n external-secrets deploy/external-secrets
```

The deployment name can differ.

---

# 112. Secret Exists but Pod Fails

Check:

```bash
kubectl get secret cart-secrets -n roboshop
kubectl describe secret cart-secrets -n roboshop
```

Then:

```bash
kubectl describe pod <pod> -n roboshop
kubectl logs <pod> -n roboshop
```

Remember that Secret metadata can be inspected without printing secret values.

---

# 113. Avoid Printing Secret Values

Do not casually execute:

```bash
kubectl get secret cart-secrets -o yaml
```

on a shared terminal and paste the output into tickets/chat.

The output may contain base64-encoded sensitive values.

---

# 114. Safer Secret Debugging

Inspect metadata:

```bash
kubectl describe secret cart-secrets -n roboshop
```

This normally shows keys and metadata without directly displaying values.

---

# 115. AWS CLI Secret Inspection

Avoid exposing secret values unnecessarily.

Use metadata operations where possible:

```bash
aws secretsmanager describe-secret \
  --secret-id /roboshop/prod/cart \
  --region ap-south-1
```

If retrieving the actual value is necessary, protect the terminal and output.

---

# 116. IAM Troubleshooting

If ESO gets:

```text
AccessDeniedException
```

check:

```text
IAM role
trust policy
identity association
secret ARN
region
KMS permissions
```

---

# 117. Region Mismatch

If the secret is in:

```text
us-east-1
```

but ESO is configured for:

```text
ap-south-1
```

the lookup may fail.

Always verify:

```text
secret region
provider region
cluster environment
```

---

# 118. Wrong Secret Path

Example:

```text
Git expects:
/roboshop/prod/cart
```

but AWS contains:

```text
/roboshop/production/cart
```

ESO will not find the expected object.

Use standardized naming.

---

# 119. Property Name Mismatch

AWS secret:

```json
{
  "DB_PASSWORD": "..."
}
```

ExternalSecret:

```yaml
property: DATABASE_PASSWORD
```

will fail because the property names differ.

---

# 120. JSON vs Plain Secret

AWS Secrets Manager can store:

```text
JSON
```

or:

```text
plain text
```

If using:

```yaml
property:
```

the secret should contain a structured value with the requested property.

---

# 121. JSON Secret Example

```json
{
  "DATABASE_USERNAME": "cart",
  "DATABASE_PASSWORD": "REDACTED"
}
```

ExternalSecret can map individual properties into Kubernetes Secret keys.

---

# 122. One Secret to Multiple Kubernetes Keys

Example:

```yaml
data:
  - secretKey: DB_USER
    remoteRef:
      key: /roboshop/prod/cart
      property: DATABASE_USERNAME

  - secretKey: DB_PASSWORD
    remoteRef:
      key: /roboshop/prod/cart
      property: DATABASE_PASSWORD
```

---

# 123. One External Secret per Application

A common pattern:

```text
cart
  -> cart-secrets

payment
  -> payment-secrets

orders
  -> orders-secrets
```

This provides clearer ownership.

---

# 124. Shared Secrets

Avoid sharing a secret broadly.

If multiple services genuinely require the same credential:

```text
document ownership
restrict IAM
consider separate credentials
```

A shared secret increases blast radius.

---

# 125. Database Credentials Per Service

For microservices:

```text
cart -> cart DB user
orders -> orders DB user
payment -> payment DB user
```

is generally safer than:

```text
all services -> one DB admin account
```

---

# 126. Database Least Privilege

A service account should receive only required DB privileges.

For example:

```text
cart user
 -> CRUD on cart database
```

not:

```text
DROP DATABASE
CREATE USER
SUPERUSER
```

unless genuinely required.

---

# 127. Secret Injection with Helm

Production chart:

```yaml
env:
  - name: DATABASE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: {{ .Values.secretName }}
        key: DATABASE_PASSWORD
```

Values:

```yaml
secretName: cart-secrets
```

---

# 128. Secret Injection with Kustomize

Base Deployment can reference:

```yaml
secretKeyRef:
  name: cart-secrets
```

The environment overlay defines the ExternalSecret.

---

# 129. Environment Overlay

Example:

```text
overlays/
├── dev/
│   ├── kustomization.yaml
│   └── external-secret.yaml
├── qa/
│   ├── kustomization.yaml
│   └── external-secret.yaml
└── prod/
    ├── kustomization.yaml
    └── external-secret.yaml
```

---

# 130. Production GitOps Structure

```text
gitops-repo/
├── applications/
├── applicationsets/
├── projects/
├── environments/
│   ├── dev/
│   │   └── secrets/
│   ├── qa/
│   │   └── secrets/
│   └── prod/
│       └── secrets/
├── helm/
│   └── roboshop/
└── platform/
    └── external-secrets/
```

The repository contains secret references, not secret values.

---

# 131. Argo CD Application Example

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: roboshop-cart-prod
  namespace: argocd
spec:
  project: roboshop-prod

  source:
    repoURL: https://github.com/company/roboshop-gitops.git
    targetRevision: main
    path: environments/prod/cart

  destination:
    name: eks-prod
    namespace: roboshop

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

This deploys the `ExternalSecret` configuration as part of the application.

---

# 132. Important Separation

The Application manifest does not contain:

```text
DATABASE_PASSWORD
```

It contains only the desired deployment path.

---

# 133. AppProject Security for Secrets

Production Project should allow:

```text
ExternalSecret
```

only if the application is supposed to manage it.

Platform teams may instead separate:

```text
platform secret infrastructure
```

from:

```text
application secret references
```

---

# 134. Resource Whitelist

If using AppProject resource restrictions, include the External Secrets API resources needed by the platform.

Example:

```yaml
namespaceResourceWhitelist:
  - group: external-secrets.io
    kind: ExternalSecret
```

The exact policy depends on whether applications or the platform owns these resources.

---

# 135. Who Owns SecretStore?

Recommended:

```text
Platform
  -> SecretStore / ClusterSecretStore

Application
  -> ExternalSecret
```

This separates infrastructure credentials from application configuration.

---

# 136. Why Not Let Every Team Create SecretStore?

Because a SecretStore defines access to external secret providers.

If teams can arbitrarily create provider credentials, they may obtain access to secrets outside their intended scope.

---

# 137. SecretStore Governance

Platform controls:

```text
AWS region
provider
identity
IAM role
allowed secret paths
```

Application teams control:

```text
ExternalSecret reference
```

within their Project/namespace.

---

# 138. Namespace-Level Isolation

Example:

```text
roboshop
  |
  +--> cart ExternalSecret
  +--> payment ExternalSecret
```

Each application should have only the secrets it requires.

---

# 139. Secret Access Through IAM

A key security boundary is:

```text
ESO identity
       |
       v
AWS IAM
       |
       v
specific secret
```

Kubernetes namespace naming alone does not protect AWS secrets.

IAM must enforce the boundary.

---

# 140. IAM Conditions

Where supported, use conditions to make policies more restrictive.

Examples may include:

```text
resource tags
principal conditions
source identity
```

Use AWS documentation and organizational standards to select safe conditions.

---

# 141. Secret Audit

Track:

```text
who changed secret metadata
who rotated it
which IAM role accessed it
when it changed
```

AWS CloudTrail is relevant for Secrets Manager/KMS audit even though it is not part of the user's application monitoring stack.

---

# 142. Important Monitoring Distinction

The application observability stack remains:

```text
Prometheus
Grafana
ELK
```

AWS audit services can complement it for security events.

---

# 143. Secret Alerts

Useful alerts include:

```text
unexpected secret access
AccessDenied spikes
secret deletion
secret rotation failure
ESO sync failure
KMS access failure
```

---

# 144. Secret Deletion Protection

AWS Secrets Manager deletion can have recovery behavior depending on configuration.

Production deletion should require controlled access.

---

# 145. Accidental Secret Deletion

If a required secret disappears:

```text
application fails
ESO fails
pods may fail
```

Recovery depends on:

```text
secret recovery window
backup/replication
versioning
DR process
```

---

# 146. Secret Store Failure

If AWS Secrets Manager becomes unavailable:

```text
existing Kubernetes Secret
```

may remain available to running workloads, depending on consumption method.

However:

```text
new pods
secret refresh
rotation
```

may fail.

This is why availability testing matters.

---

# 147. Cached Secrets

A running application may continue using the current secret after an external secret store outage.

But if the pod restarts while the secret cannot be recreated or refreshed:

```text
startup may fail
```

---

# 148. Designing for Secret Provider Outage

Consider:

```text
existing Secret persistence
refresh behavior
pod restart behavior
provider availability
multi-region strategy
```

---

# 149. Secret Provider Disaster Recovery

For critical workloads:

```text
Primary Secrets Manager
        |
        v
replication / backup
        |
        v
DR region/account
```

and IAM/ESO configuration must also be recoverable.

---

# 150. Secret Migration

When moving from:

```text
Kubernetes plaintext Secret
```

to:

```text
AWS Secrets Manager + ESO
```

use a controlled migration.

---

# 151. Migration Steps

```text
1. Inventory existing secrets.
2. Classify sensitivity.
3. Create AWS Secrets Manager entries.
4. Create IAM access.
5. Deploy ESO.
6. Deploy SecretStore.
7. Deploy ExternalSecret.
8. Validate generated Secret.
9. Update application.
10. Remove plaintext source.
11. Rotate credentials if exposure occurred.
```

---

# 152. Secret Inventory

Maintain a list:

```text
secret
owner
environment
application
source
rotation
dependency
```

Example:

| Secret | App | Environment | Source | Owner |
|---|---|---|---|---|
| cart DB | cart | prod | Secrets Manager | DB team |
| payment API | payment | prod | Secrets Manager | payment team |
| Git credential | Argo CD | prod | Argo CD secret | platform |

---

# 153. Secret Classification

Example:

```text
Critical:
  database root credential
  private signing key

High:
  application API token

Medium:
  internal integration credential
```

Classification determines controls and rotation urgency.

---

# 154. Secret Ownership

Do not allow:

```text
unknown owner
```

Every production secret should have an accountable team.

---

# 155. Secret Expiration

Not every secret supports native expiration.

Track expiration externally when necessary:

```text
credential expiry date
certificate expiry date
rotation deadline
```

---

# 156. TLS Certificates

TLS certificates have:

```text
certificate
private key
CA chain
expiration
```

Private keys are secrets.

---

# 157. AWS ACM

For AWS ALB Ingress, AWS Certificate Manager can manage certificates.

This can avoid placing a private TLS key in Kubernetes.

Architecture:

```text
Client
 |
 HTTPS
 |
 v
AWS ALB
 |
 ACM certificate
 |
 v
Kubernetes Service
```

This is often preferable for ALB TLS termination.

---

# 158. Kubernetes TLS Secret

If TLS terminates inside Kubernetes, a Secret may contain:

```yaml
type: kubernetes.io/tls
```

with:

```text
tls.crt
tls.key
```

The private key must be protected.

---

# 159. Certificate Rotation

Use automated certificate management where appropriate.

Monitor:

```text
expiration
renewal
deployment
```

---

# 160. API Keys

API keys should be:

```text
stored externally
scoped
rotated
audited
```

Do not place them in:

```text
Dockerfile
Git repository
Helm values
ConfigMap
```

---

# 161. Dockerfile Secret Risk

Never do:

```dockerfile
ENV API_TOKEN=secret
```

This can expose the secret through image metadata/history and registry access.

---

# 162. Build-Time Secrets

If a build requires credentials:

```text
use secure build secret mechanisms
```

rather than embedding credentials in layers.

---

# 163. CI Logs

Secrets can leak through:

```text
echo $PASSWORD
set -x
debug output
failed commands
```

Mask secrets and avoid printing them.

---

# 164. Jenkins Credential Binding

Use Jenkins credential mechanisms instead of plaintext variables in Jenkinsfiles.

Ensure credentials are not accidentally echoed.

---

# 165. GitHub Actions Secrets

Use:

```text
repository secrets
environment secrets
OIDC
```

according to the use case.

Protect production environments.

---

# 166. Secret Masking Is Not Enough

Masking reduces accidental log exposure.

It does not replace:

```text
least privilege
rotation
secure storage
```

---

# 167. Secret Leak Detection

Use:

```text
Gitleaks
TruffleHog
GitHub secret scanning
```

where approved.

Run scans:

```text
pre-commit
CI
repository monitoring
```

---

# 168. What If Secret Appears in Git History?

Do not only delete the file.

Treat the credential as compromised.

```text
revoke
rotate
investigate
rewrite history if appropriate
```

---

# 169. Git History Retention

Even after history rewrite:

```text
forks
clones
CI logs
artifacts
cache
```

may contain the old secret.

Therefore rotation is mandatory.

---

# 170. Secret Leak Incident Workflow

```text
Detection
   |
   v
Revoke
   |
   v
Rotate
   |
   v
Investigate
   |
   v
Contain
   |
   v
Remove exposure
   |
   v
Verify
   |
   v
Improve controls
```

---

# 171. Production Secret Security Checklist

```text
[ ] No plaintext secrets in Git
[ ] No secrets in Dockerfiles
[ ] No secrets in ConfigMaps
[ ] No secrets in logs
[ ] Secrets Manager used for sensitive values
[ ] KMS protection reviewed
[ ] ESO deployed securely
[ ] IAM least privilege
[ ] IRSA/Pod Identity
[ ] Environment isolation
[ ] SecretStore restricted
[ ] ExternalSecret ownership defined
[ ] Rotation policy
[ ] Secret versioning understood
[ ] Backup/DR plan
[ ] Audit logging
[ ] Secret scanning
[ ] Credential revocation process
[ ] Certificate renewal
```

---

# 172. RoboShop Secret Architecture

```text
                    GitHub
                       |
                       v
               GitOps Repository
                       |
             ExternalSecret YAML
                       |
                       v
                   Argo CD
                       |
                       v
                     EKS
                       |
                       v
             External Secrets Operator
                       |
              IAM / IRSA / Pod Identity
                       |
                       v
              AWS Secrets Manager
                       |
                       v
              Kubernetes Secret
                       |
                       v
                RoboShop Pod
```

---

# 173. RoboShop Secret Paths

```text
/roboshop/dev/cart
/roboshop/dev/payment
/roboshop/dev/orders

/roboshop/qa/cart
/roboshop/qa/payment
/roboshop/qa/orders

/roboshop/prod/cart
/roboshop/prod/payment
/roboshop/prod/orders
```

---

# 174. RoboShop IAM Isolation

```text
ESO-DEV
  -> /roboshop/dev/*

ESO-QA
  -> /roboshop/qa/*

ESO-PROD
  -> /roboshop/prod/*
```

If separate clusters use separate ESO identities, the IAM boundary is stronger.

---

# 175. RoboShop Application Example

Cart Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cart
  namespace: roboshop
spec:
  replicas: 3
  selector:
    matchLabels:
      app: cart
  template:
    metadata:
      labels:
        app: cart
    spec:
      serviceAccountName: cart
      containers:
        - name: cart
          image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart@sha256:REPLACE
          env:
            - name: DATABASE_USERNAME
              valueFrom:
                secretKeyRef:
                  name: cart-secrets
                  key: DATABASE_USERNAME
            - name: DATABASE_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: cart-secrets
                  key: DATABASE_PASSWORD
```

---

# 176. RoboShop ExternalSecret

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: cart-secrets
  namespace: roboshop
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets
    kind: SecretStore
  target:
    name: cart-secrets
    creationPolicy: Owner
  data:
    - secretKey: DATABASE_USERNAME
      remoteRef:
        key: /roboshop/prod/cart
        property: DATABASE_USERNAME
    - secretKey: DATABASE_PASSWORD
      remoteRef:
        key: /roboshop/prod/cart
        property: DATABASE_PASSWORD
```

---

# 177. RoboShop SecretStore

A platform-managed SecretStore can conceptually look like:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets
  namespace: roboshop
spec:
  provider:
    aws:
      service: SecretsManager
      region: ap-south-1
```

The actual provider authentication fields must match the ESO version and EKS identity model used by the platform.

---

# 178. RoboShop ServiceAccount

Traditional IRSA example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-secrets
  namespace: external-secrets
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/roboshop-eso-prod
```

This is an example only; use the official ESO deployment and AWS identity configuration for the installed version.

---

# 179. RoboShop Secret Flow

```text
PR approved
   |
   v
GitOps merge
   |
   v
Argo CD sync
   |
   v
ExternalSecret created/updated
   |
   v
ESO reconciles
   |
   v
Secrets Manager
   |
   v
Kubernetes Secret
   |
   v
Pod starts
   |
   v
Application reads secret
```

---

# 180. Secret Rotation in RoboShop

Example:

```text
Cart DB password rotation
```

Flow:

```text
DB
 |
 v
new credential
 |
 v
Secrets Manager
 |
 v
ESO
 |
 v
Kubernetes Secret
 |
 v
cart pods reload/restart
 |
 v
verify
 |
 v
old credential revoked
```

---

# 181. RoboShop Production Failure

Scenario:

```text
AWS Secrets Manager unavailable
```

Potential result:

```text
existing pods
 -> may continue using current values

new pods
 -> may fail if secret cannot be created/refreshed
```

Therefore monitor:

```text
ESO
Secrets Manager
pod restarts
application startup
```

---

# 182. Secret Troubleshooting Commands

```bash
kubectl get externalsecret -A
kubectl describe externalsecret cart-secrets -n roboshop

kubectl get secretstore -n roboshop
kubectl describe secretstore aws-secrets -n roboshop

kubectl get secret cart-secrets -n roboshop
kubectl describe secret cart-secrets -n roboshop

kubectl get pods -n external-secrets
kubectl logs -n external-secrets deploy/external-secrets
```

---

# 183. AWS Troubleshooting Commands

Metadata:

```bash
aws secretsmanager describe-secret \
  --secret-id /roboshop/prod/cart \
  --region ap-south-1
```

List:

```bash
aws secretsmanager list-secrets \
  --region ap-south-1
```

Use commands that reveal actual secret values only when necessary and in a protected environment.

---

# 184. IAM Troubleshooting

Check the ServiceAccount:

```bash
kubectl get sa external-secrets \
  -n external-secrets \
  -o yaml
```

Then verify the AWS role trust relationship and permissions.

---

# 185. Kubernetes Secret Missing

If:

```bash
kubectl get secret cart-secrets -n roboshop
```

returns NotFound:

Check:

```text
ExternalSecret status
ESO logs
SecretStore
IAM
AWS secret path
```

---

# 186. Secret Exists but Is Stale

Check:

```text
refreshInterval
ESO controller health
AWS secret version
Kubernetes Secret update timestamp
application reload behavior
```

---

# 187. Application Still Uses Old Password

Possible cause:

```text
environment variable was loaded at pod startup
```

Solution:

```text
restart pods
```

or implement application-level secret reload.

Do not restart production blindly; use controlled rollout.

---

# 188. Secret Store AccessDenied

Likely causes:

```text
wrong IAM role
wrong trust policy
wrong secret ARN
wrong account
wrong region
KMS permission issue
```

---

# 189. KMS AccessDenied

If Secrets Manager uses a customer-managed KMS key:

```text
inspect key policy
inspect IAM policy
inspect role
```

Do not grant broad KMS permissions as a quick workaround.

---

# 190. ESO Pod Has No AWS Credentials

Check:

```text
ServiceAccount
IRSA annotation or Pod Identity association
IAM trust
pod restart
EKS OIDC configuration if using IRSA
```

---

# 191. SecretStore Is Ready but ExternalSecret Fails

Check:

```text
remoteRef.key
remoteRef.property
target secret
namespace
provider region
AWS permission
```

---

# 192. ExternalSecret Repeatedly Reconciles

Potential causes:

```text
external value changes
provider failure
invalid target
refresh configuration
conflicting controller behavior
```

Inspect events and controller logs.

---

# 193. Argo CD Reports ExternalSecret OutOfSync

Check:

```bash
argocd app get roboshop-prod
argocd app diff roboshop-prod
```

Remember:

```text
Argo CD controls ExternalSecret configuration
```

not the secret value itself.

---

# 194. Argo CD Shows Secret Difference

If Argo CD manages a generated Kubernetes Secret directly while ESO also manages it, controllers may fight.

Avoid dual ownership.

Preferred:

```text
Argo CD -> ExternalSecret
ESO -> Kubernetes Secret
```

not:

```text
Argo CD -> Secret
ESO -> same Secret
```

---

# 195. Ownership Principle

One resource should have one clear controller.

```text
ExternalSecret
 -> ESO owns generated Secret

Application
 -> Argo CD owns ExternalSecret
```

This avoids reconciliation conflicts.

---

# 196. Ignore Differences

Do not blindly configure Argo CD:

```yaml
ignoreDifferences:
```

for generated Secrets.

First determine whether the resource should be directly managed by Argo CD at all.

---

# 197. Secret Drift Model

```text
Git
 |
 | defines ExternalSecret
 v
Argo CD
 |
 v
ExternalSecret
 |
 v
ESO
 |
 v
Kubernetes Secret
```

This is intentional multi-controller architecture.

---

# 198. Secret Management Anti-Patterns

Avoid:

```text
plaintext passwords in Git
AWS keys in YAML
Secrets in ConfigMaps
passwords in Dockerfiles
passwords in Helm values
shared admin credentials
one IAM role for all workloads
one secret for all services
public secret repositories
long-lived CI credentials
```

---

# 199. More Anti-Patterns

Avoid:

```text
Argo CD Git write access without need
CI cluster-admin
ESO access to all Secrets Manager secrets
all namespaces sharing ClusterSecretStore without controls
manual secret changes with no record
no rotation plan
no backup/DR plan
```

---

# 200. Secret Management Production Checklist

```text
Architecture
[ ] External secret source selected
[ ] Ownership boundaries documented
[ ] Environment separation
[ ] Multi-account design reviewed

AWS
[ ] Secrets Manager
[ ] KMS
[ ] IAM least privilege
[ ] IRSA or Pod Identity
[ ] CloudTrail/audit

ESO
[ ] Operator secured
[ ] SecretStore/ClusterSecretStore policy
[ ] refresh interval
[ ] ownership
[ ] monitoring

Kubernetes
[ ] RBAC
[ ] encryption at rest
[ ] no unnecessary API access
[ ] pod security
[ ] secret consumption reviewed

GitOps
[ ] no plaintext values
[ ] protected repo
[ ] ExternalSecret manifests reviewed
[ ] AppProject restrictions
[ ] no dual ownership

Operations
[ ] rotation
[ ] backup
[ ] DR
[ ] incident response
[ ] secret scanning
```

---

# 201. Interview Question: Why Should Secrets Not Be Stored in Git?

### Answer

> Git is designed for durable version history, while secrets require confidentiality and rotation. A plaintext secret committed to Git can remain in history, clones, forks and CI artifacts even after deletion. I keep secret values in a dedicated secret manager and store only references in Git.

---

# 202. Interview Question: Is a Kubernetes Secret Encrypted?

### Answer

> A Kubernetes Secret is a Kubernetes API resource, and its YAML representation commonly uses base64 encoding. Base64 is not encryption. Encryption at rest must be explicitly configured and access must also be protected through RBAC and other controls.

---

# 203. Interview Question: How Do You Manage Secrets in EKS?

### Answer

> For AWS/EKS I prefer AWS Secrets Manager as the external source, External Secrets Operator for synchronization, and IRSA or EKS Pod Identity for AWS authentication. Git contains ExternalSecret configuration but not plaintext secret values.

---

# 204. Interview Question: What Is External Secrets Operator?

### Answer

> ESO is a Kubernetes controller that synchronizes secrets from external providers such as AWS Secrets Manager into Kubernetes Secret resources. Argo CD can deploy the ExternalSecret definition, while ESO retrieves the actual value.

---

# 205. Interview Question: SecretStore vs ClusterSecretStore?

### Answer

> SecretStore is namespace-scoped, while ClusterSecretStore is cluster-scoped and can be referenced across namespaces. I prefer SecretStore when stronger namespace isolation is needed and ClusterSecretStore when a platform-managed shared provider is intentional.

---

# 206. Interview Question: How Does ESO Authenticate to AWS?

### Answer

> In EKS I avoid static AWS credentials. I use workload identity, typically IRSA or EKS Pod Identity, to associate the ESO ServiceAccount with a least-privilege IAM role.

---

# 207. Interview Question: Why Should ESO Not Read Every Secret?

### Answer

> ESO is a control-plane component with potentially broad access. If compromised, excessive Secrets Manager permissions create a large blast radius. I scope its IAM policy to only the required secret paths or ARNs.

---

# 208. Interview Question: What Happens When a Secret Changes?

### Answer

> ESO detects the external secret change during reconciliation and updates the Kubernetes Secret. Whether the application immediately uses the new value depends on how the application consumes the Secret. Environment variables normally require a pod restart, while mounted secret files can update and may be reloadable by the application.

---

# 209. Interview Question: Does Argo CD Rotate Secrets?

### Answer

> Not normally. Argo CD manages desired Kubernetes configuration. Secret rotation belongs to the secret-management system, such as AWS Secrets Manager and its rotation process. ESO then synchronizes the resulting value.

---

# 210. Interview Question: Why Use AWS Secrets Manager Instead of Kubernetes Secrets?

### Answer

> AWS Secrets Manager provides centralized secret lifecycle management, IAM authorization, encryption integration, versioning, rotation capabilities and AWS audit integration. Kubernetes Secrets can then be treated as runtime delivery objects rather than the primary secret store.

---

# 211. Interview Question: Why Is Secrets Manager + ESO Better for GitOps?

### Answer

> Git remains the source of truth for deployment configuration while Secrets Manager remains the source of truth for sensitive values. This keeps Git reviewable and reproducible without exposing credentials, while ESO bridges the external secret into Kubernetes.

---

# 212. Interview Question: What Is IRSA?

### Answer

> IRSA maps a Kubernetes ServiceAccount to an AWS IAM role through the EKS OIDC identity provider. Pods can obtain temporary AWS credentials through web identity instead of storing long-lived AWS keys.

---

# 213. Interview Question: What Is EKS Pod Identity?

### Answer

> EKS Pod Identity is an AWS-native mechanism for associating EKS ServiceAccounts with IAM roles. It provides workload-level AWS permissions without embedding static access keys in pods.

---

# 214. Interview Question: IRSA or Pod Identity?

### Answer

> Both provide workload identity. I choose based on the organization's EKS standard, supported features and operational requirements. The important principle is to avoid static AWS credentials and maintain least-privilege IAM.

---

# 215. Interview Question: How Do You Rotate a Database Password Without Downtime?

### Answer

> If the database supports overlapping credentials, I create the new credential, store it in Secrets Manager, allow ESO to synchronize it, roll the application safely, verify new connections, and then revoke the old credential. I avoid changing the only valid credential before consumers are ready.

---

# 216. Interview Question: What If ESO Cannot Reach AWS?

### Answer

> Existing pods may continue using the Secret they already have, depending on their consumption mechanism. However, new pods or refresh operations may fail. I check ESO health, IAM, network connectivity, Secrets Manager availability and application restart behavior.

---

# 217. Interview Question: Why Not Let Argo CD Manage the Kubernetes Secret Directly?

### Answer

> If the secret value is stored in Git, it creates exposure. If ESO also manages the same Secret, there can be competing ownership. I prefer Argo CD managing the ExternalSecret declaration and ESO managing the generated Kubernetes Secret.

---

# 218. Interview Question: Can Terraform Manage Secrets?

### Answer

> Terraform can provision secret infrastructure, but secret values supplied to Terraform can appear in Terraform state. I therefore carefully separate secret infrastructure from sensitive values and secure the Terraform backend. For runtime secret values I prefer a dedicated secret-management workflow.

---

# 219. Interview Question: What Happens If a Secret Is Leaked to Git?

### Answer

> I treat it as compromised immediately. I revoke or rotate it, investigate exposure, inspect Git and CI history, remove the active credential, update the secret store and verify access logs. Deleting the file alone is not sufficient.

---

# 220. Interview Question: How Do You Protect Production Secrets from DEV?

### Answer

> I use separate AWS accounts or secret namespaces/paths where appropriate, separate IAM roles, environment-specific ExternalSecrets and least-privilege policies. The DEV identity should not be able to read `/roboshop/prod/*`.

---

# 221. Interview Question: What Is Secret Rotation vs Secret Refresh?

### Answer

> Rotation changes the underlying credential value. Refresh is the process of detecting and synchronizing that changed value into the consumer environment. ESO can perform refresh, but the application may still need a restart or reload to consume the rotated credential.

---

# 222. Interview Question: How Do You Secure TLS Secrets?

### Answer

> If AWS ALB terminates TLS, I prefer ACM so the private key does not need to be stored in Kubernetes. If TLS terminates inside Kubernetes, I protect the `kubernetes.io/tls` Secret using RBAC, encryption at rest and secure certificate rotation.

---

# 223. Interview Question: What Is the Difference Between SOPS and External Secrets?

### Answer

> SOPS encrypts secret values for storage in Git, while External Secrets keeps values in an external secret manager and stores only references in Git. SOPS is Git-centric encrypted configuration; External Secrets is external-source secret management.

---

# 224. Interview Question: What Is the Difference Between Sealed Secrets and External Secrets?

### Answer

> Sealed Secrets stores encrypted secret manifests in Git and decrypts them in the cluster. External Secrets stores the actual values in an external secret provider and synchronizes them into Kubernetes. The latter is attractive when AWS Secrets Manager is already the enterprise secret source.

---

# 225. Interview Question: How Do You Secure ESO?

### Answer

> I use least-privilege IAM, workload identity, restricted namespaces, minimal network access, protected operator deployment, monitoring and controlled SecretStore configuration. I do not give ESO unrestricted access to every secret in the AWS account.

---

# 226. Interview Question: Why Is SecretStore Ownership Important?

### Answer

> SecretStore configuration determines how an application can reach an external secret provider. If every application team can freely configure it, they may be able to access secrets outside their intended boundary. Platform teams should normally control the provider identity and allowed access.

---

# 227. Interview Question: How Do You Handle Secret Disaster Recovery?

### Answer

> I keep the primary secret source recoverable or replicated, protect encryption keys, document IAM and ESO configuration, and test recovery. A DR rebuild should be able to recreate EKS, Argo CD and ESO and reconnect them to the correct secrets without requiring plaintext secret files.

---

# 228. Interview Question: How Do You Monitor Secret Management?

### Answer

> I monitor ESO reconciliation failures, SecretStore health, pod startup failures, AWS Secrets Manager access errors and rotation failures. Security auditing can use AWS audit logs, while Prometheus/Grafana/ELK can provide operational visibility.

---

# 229. Interview Question: Explain the Complete RoboShop Secret Flow

### Answer

> A RoboShop service such as Cart has an ExternalSecret definition in the GitOps repository. Argo CD deploys that definition to the EKS cluster. External Secrets Operator authenticates to AWS using IRSA or EKS Pod Identity and reads `/roboshop/prod/cart` from Secrets Manager. ESO creates the `cart-secrets` Kubernetes Secret. The Cart Deployment references that Secret through `secretKeyRef`. When the database credential is rotated, Secrets Manager changes, ESO refreshes the Kubernetes Secret, and the application is restarted or reloaded according to its secret-consumption design.

---

# 230. Production Secret Architecture Summary

```text
                         GitHub
                            |
                            v
                    GitOps Repository
                            |
                    ExternalSecret YAML
                            |
                            v
                         Argo CD
                            |
                            v
                           EKS
                            |
                            v
                External Secrets Operator
                            |
                 IRSA / EKS Pod Identity
                            |
                            v
                  AWS Secrets Manager
                            |
                          KMS
                            |
                            v
                    Kubernetes Secret
                            |
                            v
                       RoboShop Pod
                            |
                    +-------+-------+
                    |               |
                    v               v
                 Database          API
```

---

# 231. Final Rules

```text
1. Never commit plaintext secrets to Git.
2. Base64 is not encryption.
3. Use a dedicated secret manager for production secret values.
4. For AWS/EKS, AWS Secrets Manager is a strong external source.
5. Use External Secrets Operator to synchronize values.
6. Use IRSA or EKS Pod Identity instead of static AWS credentials.
7. Scope IAM permissions to exact secret paths.
8. Separate DEV, QA and PROD secret access.
9. Protect SecretStore configuration.
10. Prefer platform ownership of provider credentials.
11. Let Argo CD manage ExternalSecret definitions.
12. Let ESO manage generated Kubernetes Secrets.
13. Avoid dual ownership of generated Secrets.
14. Understand environment-variable refresh behavior.
15. Design rotation before production.
16. Prefer overlapping credentials for zero-downtime rotation.
17. Protect KMS keys and policies.
18. Audit secret access.
19. Scan repositories for accidental secrets.
20. Treat leaked secrets as compromised immediately.
21. Rotate rather than merely delete leaked values.
22. Protect CI credentials.
23. Prefer OIDC for AWS CI access.
24. Do not put credentials in Dockerfiles.
25. Do not expose secrets in CI logs.
26. Use ACM for ALB TLS where appropriate.
27. Encrypt and protect backups.
28. Test secret recovery.
29. Minimize shared secrets.
30. Give every secret an owner.
31. Monitor ESO failures.
32. Monitor rotation failures.
33. Protect multi-account and multi-cluster boundaries.
34. Treat secret-management infrastructure as production control-plane infrastructure.
35. Keep Git as the source of truth for secret configuration and the secret manager as the source of truth for secret values.
```

---

# 232. Next File

```text
16-ArgoCD-Advanced-Features.md
```

The next file will cover advanced Argo CD behavior in depth, including:

- Sync options
- Prune behavior
- Self-heal
- Replace
- Force
- Server-Side Apply
- ApplyOutOfSyncOnly
- CreateNamespace
- PruneLast
- PrunePropagationPolicy
- RespectIgnoreDifferences
- Retry policies
- Sync waves
- Hooks
- PreSync
- Sync
- PostSync
- SyncFail
- Skip
- Hook deletion policies
- Resource health
- Custom health checks
- Resource tracking
- Ignore differences
- Revision history
- Rollback
- Refresh
- Hard refresh
- Webhooks
- Notifications
- Argo CD CLI advanced operations
- Application actions
- Resource overrides
- Custom plugins
- CMP concepts
- Repo-server security
- ApplicationSet advanced patterns
- Progressive synchronization
- Selective sync
- Production examples
- Advanced troubleshooting
- RoboShop advanced deployment patterns
- Advanced interview scenarios
