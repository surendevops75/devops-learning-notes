# GitOps-Security-and-RBAC

## 1. Purpose

This file covers production security for GitOps and Argo CD in AWS/EKS environments.

The objective is to protect every stage of the deployment chain:

```text
Developer
   |
   v
Source Git
   |
   v
CI / DevSecOps
   |
   v
ECR
   |
   v
GitOps Repository
   |
   v
Argo CD
   |
   v
Kubernetes API
   |
   v
EKS Workloads
```

Security must not depend on one control.

A production GitOps platform should use defense in depth:

```text
Identity
+
Authentication
+
Authorization
+
Repository protection
+
Secrets management
+
Network security
+
Kubernetes RBAC
+
Admission policy
+
Image security
+
Audit
+
Monitoring
```

This file covers:

- GitOps security fundamentals
- Threat model
- Argo CD security architecture
- Authentication
- SSO
- OIDC
- RBAC
- AppProjects
- Repository access
- Cluster access
- Kubernetes RBAC
- ServiceAccounts
- AWS IAM
- IRSA
- EKS Pod Identity
- Multi-cluster security
- Multi-account security
- GitHub/GitLab authentication concepts
- SSH vs HTTPS
- GitHub Apps
- Webhook security
- TLS
- Secrets
- External Secrets
- AWS Secrets Manager
- Image signing
- Admission policies
- Network controls
- Argo CD hardening
- Production YAML examples
- Attack scenarios
- Troubleshooting
- Security runbooks
- Interview questions

---

# 2. Security Goals

A secure GitOps platform should provide:

```text
Confidentiality
Integrity
Availability
Authenticity
Accountability
Least privilege
Traceability
Recoverability
```

For GitOps, integrity is especially important.

If an attacker changes:

```yaml
replicas: 3
```

to:

```yaml
replicas: 300
```

the change can affect the entire cluster.

---

# 3. GitOps Security Principle

Git is the desired-state source of truth.

Therefore:

```text
Git repository security
=
deployment security
```

Protecting the Kubernetes API while leaving GitOps repositories poorly protected is incomplete security.

---

# 4. Threat Model

Potential attackers include:

```text
Compromised developer account
Compromised CI runner
Compromised CI token
Compromised GitOps bot
Compromised Argo CD account
Malicious repository contributor
Compromised container image
Stolen cluster credential
Insider threat
Supply-chain dependency attack
```

---

# 5. Main Attack Paths

### Path 1

```text
Developer
   |
   v
Git
   |
   v
CI
   |
   v
Malicious image
   |
   v
ECR
   |
   v
GitOps
   |
   v
EKS
```

### Path 2

```text
Attacker
   |
   v
GitOps repository
   |
   v
ApplicationSet
   |
   v
Argo CD
   |
   v
Multiple clusters
```

### Path 3

```text
Attacker
   |
   v
Argo CD credentials
   |
   v
Kubernetes API
```

---

# 6. Security Boundary Model

A production architecture should create multiple boundaries:

```text
+------------------+
| Source Git       |
+------------------+
        |
        v
+------------------+
| CI Identity      |
+------------------+
        |
        v
+------------------+
| ECR              |
+------------------+
        |
        v
+------------------+
| GitOps Git       |
+------------------+
        |
        v
+------------------+
| Argo CD          |
+------------------+
        |
        v
+------------------+
| Kubernetes RBAC  |
+------------------+
        |
        v
+------------------+
| Workloads        |
+------------------+
```

Each boundary should have its own authorization rules.

---

# 7. Authentication vs Authorization

This distinction is fundamental.

### Authentication

Answers:

```text
Who are you?
```

Examples:

```text
OIDC
SSO
GitHub identity
GitLab identity
AWS IAM
client certificate
```

### Authorization

Answers:

```text
What are you allowed to do?
```

Examples:

```text
RBAC
IAM policy
AppProject restrictions
Kubernetes Role
ClusterRole
```

---

# 8. Argo CD Authentication

Argo CD can support authentication through:

```text
local accounts
SSO
OIDC
```

For enterprise environments, centralized identity is generally preferred.

---

# 9. Why SSO?

Without SSO:

```text
Many local usernames/passwords
```

With SSO:

```text
Corporate Identity Provider
             |
             v
          Argo CD
```

Benefits include:

```text
central account lifecycle
MFA
group membership
offboarding
audit
password policy
```

---

# 10. OIDC

OIDC provides identity information through an identity provider.

Conceptually:

```text
User
 |
 v
Identity Provider
 |
 v
OIDC token
 |
 v
Argo CD
 |
 v
RBAC
```

The exact configuration depends on the identity provider.

---

# 11. OIDC Claims

Useful claims may include:

```text
sub
email
name
groups
```

Argo CD can map identity/group information into authorization policies.

---

# 12. Group-Based RBAC

Instead of granting every user individually:

```text
alice
bob
charlie
```

use:

```text
devops-team
platform-team
release-managers
developers
```

Then assign permissions to groups.

---

# 13. Example Enterprise Groups

```text
argocd-platform-admins
argocd-platform-readonly
argocd-dev-deployers
argocd-prod-deployers
argocd-auditors
```

Group naming should be consistent with the organization's identity standards.

---

# 14. Least Privilege

The most important RBAC rule:

```text
Give the minimum permission required.
```

Example:

A developer who only needs DEV application visibility should not receive:

```text
applications, delete
clusters, update
projects, update
```

---

# 15. Argo CD RBAC

Argo CD RBAC controls access to Argo CD resources.

Conceptually:

```text
User
 |
 v
Argo CD authentication
 |
 v
RBAC policy
 |
 v
Allowed/Denied operation
```

---

# 16. Argo CD RBAC Policy Model

Policies commonly follow the conceptual structure:

```text
p, subject, resource, action, object, effect
```

Example:

```text
p, role:dev-readonly, applications, get, roboshop-dev/*, allow
```

Group assignment can be represented conceptually as:

```text
g, developers, role:dev-readonly
```

The exact policy syntax should match the installed Argo CD version.

---

# 17. RBAC Resource Types

Important Argo CD RBAC resources include:

```text
applications
applicationsets
projects
repositories
clusters
accounts
certificates
logs
exec
```

Do not grant broad access automatically.

---

# 18. Application Permissions

Possible actions include:

```text
get
create
update
delete
sync
override
action/*
```

Grant only what is needed.

---

# 19. Read-Only Role

A read-only user may need:

```text
applications: get
projects: get
repositories: get
```

but should not have:

```text
applications: delete
applications: sync
projects: update
```

---

# 20. DEV Deployment Role

A DEV deployment role might be allowed to:

```text
get applications
sync DEV applications
```

but not:

```text
sync PROD applications
```

---

# 21. Production Deployment Role

Production deployment should be tightly controlled.

Possible model:

```text
release-manager
    |
    +--> get PROD applications
    +--> sync approved PROD applications
```

Platform administration should remain separate.

---

# 22. Platform Administrator

Platform admins may require:

```text
projects
repositories
clusters
applications
applicationsets
```

But this role should be limited to a small group.

---

# 23. Auditor Role

Auditors generally need:

```text
read-only visibility
```

and should not be allowed to:

```text
sync
delete
update
```

---

# 24. Example RBAC ConfigMap

A conceptual production-style configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.default: role:readonly

  policy.csv: |
    # Platform administrators
    g, argocd-platform-admins, role:platform-admin

    # DEV deployers
    g, argocd-dev-deployers, role:dev-deployer

    # Production release managers
    g, argocd-prod-deployers, role:prod-deployer

    # Auditors
    g, argocd-auditors, role:auditor

    # DEV read-only access
    p, role:dev-deployer, applications, get, roboshop-dev/*, allow
    p, role:dev-deployer, applications, sync, roboshop-dev/*, allow
    p, role:dev-deployer, projects, get, *, allow

    # PROD read/sync access
    p, role:prod-deployer, applications, get, roboshop-prod/*, allow
    p, role:prod-deployer, applications, sync, roboshop-prod/*, allow
    p, role:prod-deployer, projects, get, *, allow

    # Auditor
    p, role:auditor, applications, get, *, allow
    p, role:auditor, projects, get, *, allow

    # Platform administrator
    p, role:platform-admin, *, *, *, allow
```

This is an example policy model. Production policies should be narrowed further and validated against the organization's Argo CD version and object naming conventions.

---

# 25. Why Default Role Matters

If:

```text
policy.default
```

is too permissive, every authenticated user can receive excessive access.

A common approach is:

```text
role:readonly
```

or another intentionally limited default.

---

# 26. Deny Policies

A deny rule can be used when a broad allow would otherwise grant access.

Example concept:

```text
deny production destructive actions
```

However, avoid overly complex policy sets.

A simpler least-privilege model is easier to audit.

---

# 27. AppProject Security

Argo CD Projects are a major security boundary.

An AppProject can restrict:

```text
source repositories
destination clusters
destination namespaces
resource types
```

---

# 28. Why Projects Matter

Without strong Project boundaries, an Application may potentially be configured to target:

```text
unexpected cluster
unexpected namespace
unexpected repository
```

Projects constrain the blast radius.

---

# 29. Production AppProject

Example:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: roboshop-prod
  namespace: argocd
spec:
  description: RoboShop production applications

  sourceRepos:
    - https://github.com/company/roboshop-gitops.git

  destinations:
    - name: eks-prod
      namespace: roboshop
    - name: eks-prod
      namespace: roboshop-platform

  clusterResourceWhitelist:
    - group: ""
      kind: Namespace

  namespaceResourceWhitelist:
    - group: ""
      kind: ConfigMap
    - group: ""
      kind: Secret
    - group: ""
      kind: Service
    - group: apps
      kind: Deployment
    - group: autoscaling
      kind: HorizontalPodAutoscaler
    - group: networking.k8s.io
      kind: Ingress
```

The exact resource allowlist should match what the platform actually needs.

---

# 30. Project Source Restrictions

A production Project should not normally allow:

```text
*
```

for repositories.

Prefer:

```yaml
sourceRepos:
  - https://github.com/company/roboshop-gitops.git
```

This prevents arbitrary repositories from becoming deployment sources.

---

# 31. Project Destination Restrictions

Avoid:

```yaml
destinations:
  - namespace: "*"
    server: "*"
```

for production workloads.

Prefer explicit:

```text
cluster
namespace
```

boundaries.

---

# 32. Namespace Isolation

Example:

```text
roboshop-dev
roboshop-qa
roboshop-prod
```

Applications should not automatically have access across these namespaces.

---

# 33. Project-Based Environment Isolation

```text
roboshop-dev Project
       |
       v
DEV cluster/namespaces

roboshop-prod Project
       |
       v
PROD cluster/namespaces
```

This reduces accidental cross-environment deployment.

---

# 34. Cluster Resource Restrictions

Cluster-wide resources are high risk:

```text
ClusterRole
ClusterRoleBinding
CustomResourceDefinition
Namespace
```

Do not allow every application team to manage them.

---

# 35. Namespace Resource Restrictions

Application teams commonly need:

```text
Deployment
Service
ConfigMap
Secret
Ingress
HPA
PDB
```

They usually should not manage:

```text
ClusterRole
ClusterRoleBinding
CRD
Node
```

unless specifically required.

---

# 36. AppProject Roles

Projects can also define roles for project-scoped access.

This can support:

```text
application-team
release-team
read-only
```

within a specific Project.

---

# 37. Project Role Example

Conceptually:

```yaml
roles:
  - name: developer
    description: DEV application access
    policies:
      - p, proj:roboshop-dev:developer, applications, get, roboshop-dev/*, allow
      - p, proj:roboshop-dev:developer, applications, sync, roboshop-dev/*, allow
    groups:
      - argocd-dev-developers
```

Use project-level roles when they provide cleaner isolation than global roles.

---

# 38. Repository Credentials

Argo CD must authenticate to private Git repositories.

Common mechanisms include:

```text
HTTPS credentials
SSH keys
GitHub App
```

Choose based on enterprise policy.

---

# 39. HTTPS Authentication

Conceptually:

```text
Argo CD
   |
   | HTTPS + token
   v
Git provider
```

Use a narrowly scoped token.

Do not use a personal administrator token.

---

# 40. SSH Authentication

Conceptually:

```text
Argo CD
   |
   | SSH private key
   v
Git server
```

The private key must be protected.

---

# 41. SSH Host Verification

Argo CD should verify the Git server's host identity.

Do not disable host verification just to solve:

```text
known_hosts
```

errors.

That creates a man-in-the-middle risk.

---

# 42. GitHub App

For GitHub environments, a GitHub App can provide:

```text
installation-scoped access
fine-grained permissions
central administration
credential rotation
```

This can be preferable to broad personal access tokens.

---

# 43. Argo CD Repository Secret

A repository credential is stored as a Kubernetes Secret managed by Argo CD.

Do not manually copy plaintext private keys into GitOps repositories.

---

# 44. Example Repository Secret Concept

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: repo-roboshop-gitops
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
type: Opaque
stringData:
  type: git
  url: https://github.com/company/roboshop-gitops.git
  username: gitops-bot
  password: ${TOKEN}
```

Do not commit `${TOKEN}` or a real token to Git.

Prefer Argo CD CLI, secure secret injection, or an approved secret-management workflow to create this resource.

---

# 45. Why Repository Credentials Need Protection

If an attacker obtains an Argo CD Git credential, they may:

```text
read private repositories
```

depending on scope.

If the credential has write access, impact is much greater.

Argo CD should normally have read-only Git access.

---

# 46. Repository Read-Only Principle

Ideal:

```text
Argo CD
  |
  +--> read GitOps repo
```

CI:

```text
CI
  |
  +--> write GitOps repo through controlled PR process
```

Separate identities.

---

# 47. Cluster Credentials

Argo CD requires credentials to target clusters.

For centralized multi-cluster architecture:

```text
Argo CD management cluster
       |
       +--> EKS DEV credentials
       +--> EKS QA credentials
       +--> EKS PROD credentials
```

These credentials are highly sensitive.

---

# 48. Cluster Registration

Conceptually:

```bash
argocd cluster add <context>
```

This can configure the target cluster for Argo CD management.

Do not execute blindly in production.

Review the generated access and service account permissions.

---

# 49. Cluster Access Security

A target cluster credential should grant only what Argo CD actually needs.

Avoid blindly assigning:

```text
cluster-admin
```

without understanding the consequences.

---

# 50. Argo CD Cluster Secret

Registered cluster credentials are stored in the Argo CD control plane as Secrets.

Protect:

```text
argocd namespace
```

especially:

```text
cluster secrets
repo credentials
TLS credentials
```

---

# 51. Kubernetes RBAC

Kubernetes RBAC controls API access:

```text
Subject
 |
 v
Role / ClusterRole
 |
 v
RoleBinding / ClusterRoleBinding
 |
 v
Kubernetes API
```

---

# 52. Role vs ClusterRole

### Role

Namespace-scoped.

```text
namespace: roboshop
```

### ClusterRole

Cluster-scoped permissions.

Can be used for:

```text
cluster resources
```

or bound into namespaces.

---

# 53. RoleBinding

A RoleBinding grants a Role to a subject in a namespace.

Example:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cart-read
  namespace: roboshop
subjects:
  - kind: ServiceAccount
    name: cart
    namespace: roboshop
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cart-read
```

---

# 54. Avoid ClusterRoleBinding

Do not use:

```yaml
kind: ClusterRoleBinding
```

for application pods unless cluster-wide permissions are genuinely required.

Namespace-scoped access is safer.

---

# 55. ServiceAccounts

Each workload should use an intentional ServiceAccount.

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cart
  namespace: roboshop
```

---

# 56. ServiceAccount Token Security

Modern Kubernetes environments should avoid assuming every workload needs broad long-lived credentials.

Use:

```text
projected tokens
AWS workload identity
least privilege
```

as appropriate.

---

# 57. AWS IAM for Service Accounts

EKS workloads may need AWS API access.

A secure design maps:

```text
Kubernetes ServiceAccount
        |
        v
AWS IAM role
        |
        v
AWS service
```

rather than storing:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

inside a Kubernetes Secret.

---

# 58. IRSA

IAM Roles for Service Accounts, commonly called IRSA, allows a Kubernetes ServiceAccount to assume an AWS IAM role using web identity.

Conceptually:

```text
Pod
 |
 v
ServiceAccount
 |
 v
OIDC identity
 |
 v
AWS STS
 |
 v
IAM Role
 |
 v
AWS API
```

---

# 59. IRSA Annotation

A traditional IRSA ServiceAccount commonly uses:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cart
  namespace: roboshop
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/roboshop-cart
```

The IAM trust policy must restrict the intended EKS OIDC provider and ServiceAccount identity.

---

# 60. EKS Pod Identity

Amazon EKS also provides EKS Pod Identity as another workload identity mechanism.

Conceptually:

```text
Pod
 |
 v
ServiceAccount
 |
 v
EKS Pod Identity association
 |
 v
IAM Role
 |
 v
AWS API
```

Organizations should choose one approved approach and standardize it.

---

# 61. IRSA vs EKS Pod Identity

Both solve:

```text
workload -> AWS IAM
```

but use different mechanisms.

The important production principle is:

```text
No static AWS credentials in application pods.
```

---

# 62. IAM Policy Example

A cart service that needs a specific AWS resource should receive only the required actions.

Avoid:

```json
{
  "Effect": "Allow",
  "Action": "*",
  "Resource": "*"
}
```

Prefer a narrowly scoped policy.

---

# 63. Example Least-Privilege IAM

Conceptual example:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::company-cart-config/*"
    }
  ]
}
```

Use the minimum actions and resources required.

---

# 64. EKS Multi-Account Security

Production environments often use:

```text
DEV account
QA account
PROD account
```

Argo CD may centrally manage clusters in each account.

The trust model should be explicit:

```text
Argo CD
 |
 +--> role/credential -> DEV
 +--> role/credential -> QA
 +--> role/credential -> PROD
```

---

# 65. Account Isolation

A compromise of DEV should not automatically provide:

```text
PROD IAM credentials
```

Use separate:

```text
AWS accounts
IAM roles
network boundaries
cluster credentials
```

---

# 66. Central Argo CD Risk

A centralized Argo CD instance is powerful.

If compromised, it may affect:

```text
many clusters
```

Therefore the Argo CD management cluster requires stronger security than an ordinary application cluster.

---

# 67. Blast Radius

Centralized model:

```text
One Argo CD
   |
   +--> 10 clusters
```

Advantage:

```text
central management
```

Risk:

```text
central control-plane blast radius
```

Mitigations:

```text
HA
RBAC
Projects
network restrictions
cluster permissions
backup
monitoring
```

---

# 68. Separate Argo CD Instances

Large organizations may use:

```text
Argo CD Platform A
   |
   +--> business unit clusters

Argo CD Platform B
   |
   +--> regulated clusters
```

This reduces blast radius at the cost of operational complexity.

---

# 69. Management Cluster Security

The Argo CD management cluster should have:

```text
restricted API access
private networking where appropriate
strong IAM
MFA for human access
encrypted secrets
monitoring
backup
patching
HA
```

---

# 70. Argo CD API Server Exposure

Avoid exposing Argo CD unnecessarily to the public Internet.

Prefer:

```text
private endpoint
VPN
corporate network
identity-aware access
```

depending on requirements.

---

# 71. TLS

Use TLS for:

```text
user -> Argo CD
Argo CD -> Git
Argo CD -> Kubernetes API
webhooks
```

Do not disable TLS verification merely to bypass certificate errors.

---

# 72. Certificate Management

Production certificates should have:

```text
renewal process
monitoring
rotation
trusted CA
```

Expired certificates can stop deployments.

---

# 73. Network Security

Argo CD components need only the network access they require.

Potential controls:

```text
Security Groups
NetworkPolicies
private subnets
firewalls
proxy
VPC endpoints
```

---

# 74. Kubernetes NetworkPolicy

NetworkPolicy can restrict pod-to-pod traffic.

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: cart
  namespace: roboshop
spec:
  podSelector:
    matchLabels:
      app: cart
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: roboshop
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              name: roboshop
```

This is only a starting example; production policies should explicitly model required dependencies and DNS.

---

# 75. Default-Deny NetworkPolicy

A stronger model is:

```text
deny by default
```

then allow:

```text
DNS
required application dependencies
required ingress
required egress
```

Test carefully because overly restrictive policies can break applications.

---

# 76. GitOps and NetworkPolicy

Network policies can themselves be managed through GitOps.

This provides:

```text
review
versioning
audit
rollback
```

But a bad NetworkPolicy can block the GitOps control plane or application traffic.

---

# 77. Control Plane Safety

Be careful when GitOps manages:

```text
NetworkPolicy
Ingress
CNI configuration
Argo CD itself
```

A bad change can affect the deployment mechanism.

---

# 78. Argo CD Self-Management

Argo CD can manage its own configuration through GitOps.

Benefits:

```text
versioned configuration
repeatable recovery
```

Risks:

```text
bad RBAC change
bad repo credential
bad Project
```

can impact Argo CD itself.

Use a secure bootstrap/recovery path.

---

# 79. Break-Glass Access

Production platforms should have emergency access.

Example:

```text
break-glass identity
```

with:

```text
strong authentication
limited users
auditing
alerting
```

Do not use it for normal deployments.

---

# 80. Break-Glass Procedure

```text
1. Incident declared
2. Emergency identity approved
3. Access granted
4. Change performed
5. Evidence recorded
6. Git desired state updated
7. Temporary access removed
8. Incident reviewed
```

---

# 81. Secrets in Git

Never commit:

```text
AWS secret keys
database passwords
API tokens
private keys
TLS private keys
Git tokens
```

in plaintext.

---

# 82. Why Base64 Is Not Encryption

Kubernetes Secret data is commonly represented as base64.

Example:

```yaml
data:
  password: cGFzc3dvcmQ=
```

This is encoding, not encryption.

Base64 does not make a secret safe to commit into a public repository.

---

# 83. Sealed Secrets

One approach is:

```text
encrypted secret manifest
```

stored in Git.

The cluster controller decrypts it.

This can be useful but introduces:

```text
controller key management
backup requirements
rotation
```

---

# 84. External Secrets Operator

Another common approach:

```text
Git
 |
 v
ExternalSecret
 |
 v
External Secrets Operator
 |
 v
AWS Secrets Manager
 |
 v
Kubernetes Secret
```

Only a reference is stored in Git.

---

# 85. Example ExternalSecret

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: cart-secrets
  namespace: roboshop
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secretsmanager
    kind: ClusterSecretStore

  target:
    name: cart-secrets
    creationPolicy: Owner

  data:
    - secretKey: DATABASE_PASSWORD
      remoteRef:
        key: /roboshop/prod/cart
        property: DATABASE_PASSWORD
```

The API version should match the installed External Secrets Operator version.

---

# 86. SecretStore Security

The operator itself needs AWS access.

Use:

```text
IRSA
```

or:

```text
EKS Pod Identity
```

rather than static AWS keys.

---

# 87. AWS Secrets Manager

A production secret may be stored as:

```text
/roboshop/prod/cart
```

with fields:

```text
DATABASE_PASSWORD
API_TOKEN
```

Control access through IAM.

---

# 88. Secret Rotation

A good system supports:

```text
rotation
 |
 v
AWS Secrets Manager
 |
 v
External Secrets
 |
 v
Kubernetes Secret
 |
 v
application restart/reload
```

The application's ability to reload secrets determines the final behavior.

---

# 89. Secret Rotation Risk

If an application reads a secret only at startup:

```text
Secret changes
```

but:

```text
existing pod keeps old value
```

until restarted.

Plan rotation accordingly.

---

# 90. Secret Access Principle

Argo CD should not need to read secret values merely to deploy:

```text
ExternalSecret reference
```

The workload identity should obtain the secret through the approved mechanism.

---

# 91. Kubernetes Secret Encryption

EKS/Kubernetes supports encryption-at-rest mechanisms.

Verify the cluster's configuration for:

```text
secret encryption
KMS integration
```

where required by security policy.

---

# 92. AWS KMS

KMS can protect encryption keys used by AWS services and cluster secret encryption configurations.

Key management includes:

```text
rotation
IAM
key policies
audit
```

---

# 93. Secret Access Logging

Monitor:

```text
AWS Secrets Manager access
KMS operations
Kubernetes Secret access where observable
```

Unexpected secret access should trigger investigation.

---

# 94. Repository Secret Scanning

CI should scan source and GitOps repositories for accidental secrets.

Tools may include:

```text
Gitleaks
TruffleHog
Git provider secret scanning
```

Use tools approved by the organization.

---

# 95. Secret Leak Response

If a secret is committed:

```text
1. Revoke/rotate immediately.
2. Remove active credential.
3. Determine exposure.
4. Investigate access logs.
5. Remove from repository history if required.
6. Update secret store.
7. Review how it happened.
```

Removing a Git file alone does not make an exposed secret safe.

---

# 96. Webhook Security

Argo CD webhooks can receive repository events.

Protect with:

```text
HTTPS
signature validation
secret
IP/network restrictions where appropriate
```

---

# 97. GitHub Webhook Concept

```text
GitHub
 |
 | signed event
 v
Argo CD
 |
 v
refresh
```

Argo CD should verify the event according to supported configuration.

---

# 98. Webhook Replay

A secure webhook system should consider:

```text
replay
duplicate events
spoofed events
```

Do not treat any unauthenticated HTTP request as trusted deployment input.

---

# 99. Repository Webhook Scope

Only configured repositories should be able to trigger relevant refreshes.

Avoid overly broad webhook configurations.

---

# 100. CI Webhook Security

Jenkins webhooks should also be protected.

Prefer:

```text
Git provider signature
trusted integration
authentication
```

rather than an anonymous endpoint that starts arbitrary builds.

---

# 101. Runner Security

CI runners can be highly privileged.

Secure them with:

```text
ephemeral runners
patching
minimal software
network controls
no persistent secrets
isolated jobs
```

---

# 102. Docker Socket Risk

A CI runner exposing:

```text
/var/run/docker.sock
```

can effectively provide host-level privileges to a container.

Use safer build architectures where practical:

```text
BuildKit
rootless builders
Kaniko
isolated runners
```

depending on organizational requirements.

---

# 103. CI Supply Chain

Protect:

```text
CI actions/plugins
base images
dependencies
build tools
runner images
```

Pin trusted versions where practical.

---

# 104. Jenkins Plugin Security

Keep:

```text
Jenkins
plugins
agents
```

updated.

Remove unused plugins.

Limit administrative access.

---

# 105. GitHub Actions Security

Use:

```yaml
permissions:
  contents: read
```

by default where possible.

Grant only required permissions:

```yaml
id-token: write
pull-requests: write
contents: write
```

when a specific job needs them.

---

# 106. Pull Request Security

Never allow untrusted PR code to automatically access production secrets.

Especially consider:

```text
forked pull requests
```

because the code being executed may be controlled by an external contributor.

---

# 107. Environment Secrets

Production secrets should be available only to workflows that are:

```text
trusted
approved
protected
```

Do not expose them to arbitrary branch builds.

---

# 108. GitOps Branch Protection

Protect:

```text
main
```

with:

```text
required PR
required checks
required reviewers
no force push
```

---

# 109. CODEOWNERS

Example:

```text
/projects/          @platform-team
/applicationsets/   @platform-team
/environments/prod/ @platform-team @release-team
```

This creates ownership boundaries.

---

# 110. GitOps Review Rules

Production changes should be reviewed for:

```text
image
resources
replicas
Ingress
network
RBAC
secrets references
securityContext
```

---

# 111. Protect ApplicationSets

ApplicationSets can affect many applications.

Therefore:

```text
applicationsets/
```

should have stronger ownership than ordinary application values.

---

# 112. Protect AppProjects

AppProjects can alter:

```text
source repositories
destinations
resource permissions
```

They should be platform-controlled.

---

# 113. Protect Cluster Registration

Cluster credentials should not be modified by ordinary application teams.

Cluster registration is a platform administration function.

---

# 114. Argo CD Accounts

Local Argo CD accounts should be minimized.

If SSO is available:

```text
prefer SSO
```

and disable unused local accounts.

---

# 115. Initial Admin Account

The initial Argo CD admin account is powerful.

After bootstrap:

```text
secure it
rotate credentials
limit usage
```

If SSO is configured, organizations often minimize reliance on the built-in admin identity.

---

# 116. Password Security

Never put:

```text
admin password
```

in:

```text
GitOps YAML
Jenkinsfile
README
shell history
```

Use the platform's secure credential mechanism.

---

# 117. Argo CD CLI Authentication

A user may authenticate using:

```bash
argocd login <server>
```

The authentication method depends on server configuration.

Avoid sharing CLI tokens.

---

# 118. Argo CD API Tokens

Automation may use tokens.

Treat them as credentials:

```text
scope
expiration/rotation
storage
audit
```

must be controlled.

---

# 119. Automation Account Design

Example:

```text
release-bot
```

should have:

```text
sync production applications
```

only if that is its explicit responsibility.

It should not automatically have:

```text
platform-admin
```

permissions.

---

# 120. Repository Credential Rotation

When rotating Git credentials:

```text
create new credential
validate access
switch Argo CD
remove old credential
verify Applications
```

Avoid deleting the old credential first without a recovery path.

---

# 121. Cluster Credential Rotation

Similarly:

```text
create new access path
validate
switch
remove old
```

Monitor Applications after rotation.

---

# 122. AWS Role Rotation

IAM role credentials obtained through:

```text
OIDC
IRSA
Pod Identity
```

are preferable to long-lived static keys because credentials can be temporary.

---

# 123. EKS Cluster Endpoint

For sensitive production clusters, consider private API endpoint architecture where appropriate.

This reduces direct Internet exposure of the Kubernetes API.

---

# 124. Argo CD to Private EKS

If Argo CD is outside the target cluster/network:

```text
Argo CD
 |
 v
private connectivity
 |
 v
EKS API
```

Possible network architecture:

```text
VPC peering
Transit Gateway
VPN
Direct Connect
```

depending on the organization.

---

# 125. Central Argo CD Network Architecture

```text
                 Corporate Network
                        |
                        v
                 Argo CD Management
                        |
              +---------+---------+
              |         |         |
              v         v         v
           VPC-DEV   VPC-QA    VPC-PROD
              |         |         |
             EKS       EKS       EKS
```

Use controlled routing and firewall/security controls.

---

# 126. NetworkPolicy Does Not Replace AWS Network Security

Kubernetes NetworkPolicy operates at the pod networking layer.

AWS controls may protect:

```text
VPC
subnets
load balancers
security groups
API endpoints
```

Use both where appropriate.

---

# 127. AWS Security Groups

Control traffic between:

```text
Argo CD
EKS API
ALB
nodes
databases
```

according to architecture.

Avoid:

```text
0.0.0.0/0
```

when a narrower source is possible.

---

# 128. EKS Access Management

Modern EKS environments can use AWS-native access mechanisms for human/admin cluster access.

Keep:

```text
human access
workload access
Argo CD access
```

separate.

---

# 129. Human vs Machine Identity

Human:

```text
SSO + MFA
```

Machine:

```text
IAM role
ServiceAccount
OIDC
Pod Identity
```

Do not share human credentials with automation.

---

# 130. Kubernetes ServiceAccount for Argo CD

Argo CD components run using ServiceAccounts.

Review their RBAC permissions.

Do not assume:

```text
system:serviceaccount:argocd:...
```

is automatically safe.

---

# 131. Argo CD Component Permissions

Important components include:

```text
API Server
Application Controller
Repo Server
ApplicationSet Controller
Redis
```

Review their access and security posture.

---

# 132. Repo Server Security

Repo Server processes repository content and renders manifests.

Because it handles potentially untrusted configuration:

```text
keep updated
restrict network access
limit privileges
```

---

# 133. Application Controller Security

The Application Controller interacts with Kubernetes APIs.

Compromise can have substantial impact.

Use:

```text
least privilege
network controls
RBAC
monitoring
```

---

# 134. API Server Security

The API Server handles:

```text
user/API requests
authentication
authorization
```

Protect it through:

```text
TLS
SSO
RBAC
network access
```

---

# 135. Redis Security

Redis is used by Argo CD internally.

Do not expose it unnecessarily.

Use:

```text
internal networking
TLS/authentication where supported/configured
network restrictions
```

and keep the deployment patched.

---

# 136. ApplicationSet Controller Security

ApplicationSet can generate Applications dynamically.

It should be treated as a high-trust platform component.

Protect:

```text
applicationsets/
```

with strict repository permissions.

---

# 137. Argo CD Namespace Security

The `argocd` namespace contains highly sensitive components and credentials.

Use:

```text
NetworkPolicy
Pod Security controls
RBAC
resource limits
monitoring
```

as appropriate.

---

# 138. Pod Security

Argo CD and application workloads should avoid unnecessary:

```yaml
privileged: true
```

and:

```yaml
allowPrivilegeEscalation: true
```

where not required.

---

# 139. SecurityContext

Example:

```yaml
securityContext:
  runAsNonRoot: true
  seccompProfile:
    type: RuntimeDefault
```

Container-level:

```yaml
securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
```

Validate compatibility with the application.

---

# 140. Read-Only Root Filesystem

Where supported:

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

This reduces the ability of compromised processes to persist changes.

Applications that require writable directories should use appropriate ephemeral volumes.

---

# 141. Resource Limits

Security also includes resource protection.

Set:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 128Mi
  limits:
    cpu: 500m
    memory: 512Mi
```

Values must be sized for the real workload.

---

# 142. Denial-of-Service Protection

Use:

```text
ResourceQuota
LimitRange
HPA
PDB
requests/limits
network policies
```

to reduce accidental resource exhaustion.

---

# 143. ResourceQuota Example

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: roboshop-prod
  namespace: roboshop
spec:
  requests.cpu: "10"
  requests.memory: 20Gi
  limits.cpu: "20"
  limits.memory: 40Gi
  pods: "50"
```

Tune according to workload requirements.

---

# 144. LimitRange Example

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: defaults
  namespace: roboshop
spec:
  limits:
    - type: Container
      default:
        cpu: 500m
        memory: 512Mi
      defaultRequest:
        cpu: 100m
        memory: 128Mi
```

This helps prevent workloads from being created without resource settings.

---

# 145. SecurityContext Example

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
      securityContext:
        runAsNonRoot: true
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: cart
          image: 123456789012.dkr.ecr.ap-south-1.amazonaws.com/roboshop/cart@sha256:REPLACE
          securityContext:
            allowPrivilegeEscalation: false
            capabilities:
              drop:
                - ALL
            readOnlyRootFilesystem: true
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
```

The image digest is intentionally a placeholder.

---

# 146. Pod Security Standards

Kubernetes supports Pod Security Standards concepts such as:

```text
Privileged
Baseline
Restricted
```

Production namespaces should use the strongest profile compatible with the workloads.

---

# 147. Namespace Labels for Pod Security

Conceptual example:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

Test before enforcing broadly.

---

# 148. Admission Controllers

Admission controls can enforce security before resources are persisted.

Examples:

```text
Pod Security Admission
Kyverno
OPA Gatekeeper
```

---

# 149. Admission Policy Example

A policy might require:

```text
container images only from approved ECR registries
```

or:

```text
runAsNonRoot=true
```

---

# 150. Why Admission Policy Matters

Even if a malicious GitOps PR passes CI:

```text
Kubernetes admission
```

can prevent unsafe resources.

This creates another security boundary.

---

# 151. Image Registry Allowlist

Production policy can restrict images to:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/*
```

instead of allowing arbitrary:

```text
docker.io/*
```

where organizational policy requires private registries.

---

# 152. Image Signing

A stronger supply-chain model:

```text
CI
 |
 v
Build
 |
 v
Scan
 |
 v
Sign
 |
 v
ECR
 |
 v
Admission verification
 |
 v
EKS
```

Tools such as Cosign can be used for signing where approved.

---

# 153. Image Digest

Even with signing, deploy by immutable digest when possible:

```yaml
image:
  repository: ...
  digest: sha256:...
```

This makes the artifact identity explicit.

---

# 154. Signed GitOps Changes

Organizations may also require:

```text
signed commits
signed tags
protected branches
```

This helps verify the source of changes.

---

# 155. GitOps Supply Chain

Security should cover:

```text
Source
 |
 v
Dependencies
 |
 v
Build
 |
 v
Image
 |
 v
Registry
 |
 v
GitOps
 |
 v
Cluster
```

Every stage is part of the software supply chain.

---

# 156. Dependency Pinning

Application dependencies should be pinned where practical.

Avoid uncontrolled:

```text
latest
```

dependencies.

---

# 157. Helm Dependency Security

If a chart uses external dependencies:

```yaml
dependencies:
  - name: example
    version: 1.2.3
    repository: https://charts.example.com
```

validate:

```text
source
version
integrity
ownership
```

---

# 158. Helm Repository Security

Argo CD should use trusted chart repositories.

Prefer:

```text
HTTPS
authenticated private registry
OCI registry
```

where appropriate.

---

# 159. OCI Helm Charts

OCI registries can store Helm charts.

Conceptually:

```text
CI
 |
 v
Helm package
 |
 v
OCI registry
 |
 v
Argo CD
 |
 v
EKS
```

Use the organization's approved registry and authentication method.

---

# 160. Kustomize Security

Kustomize overlays can alter:

```text
image
resources
namespace
patches
```

Review overlays carefully because patches can change security-sensitive fields.

---

# 161. Environment Isolation

Security boundaries should include:

```text
DEV
QA
PROD
```

Production should not inherit unrestricted configuration from DEV.

---

# 162. Secrets Across Environments

Use separate secret paths:

```text
/roboshop/dev/cart
/roboshop/qa/cart
/roboshop/prod/cart
```

This prevents accidental cross-environment secret access.

---

# 163. IAM Role Per Workload

Prefer:

```text
cart -> cart IAM role
payment -> payment IAM role
```

rather than:

```text
all pods -> one giant IAM role
```

---

# 164. IAM Role Per Environment

Depending on architecture:

```text
cart-dev-role
cart-qa-role
cart-prod-role
```

can provide stronger isolation.

---

# 165. Production IAM Example

Conceptually:

```text
cart-prod
  |
  +--> s3:GetObject
  +--> specific bucket/path
```

No access to:

```text
payment secrets
other accounts
unrelated S3 buckets
```

---

# 166. Cross-Account IAM

If a workload needs another AWS account:

```text
Pod identity
 |
 v
Role A
 |
 v
STS AssumeRole
 |
 v
Role B
 |
 v
AWS resource
```

Trust relationships should be narrowly scoped.

---

# 167. Cross-Account Risk

Do not create:

```text
Principal: *
```

or unrestricted trust relationships.

Restrict:

```text
source account
role
external conditions
```

as appropriate.

---

# 168. ECR Pull Security

EKS workloads must be able to pull images.

Verify:

```text
node role
pod identity architecture
ECR repository policy
network connectivity
```

depending on the EKS setup.

---

# 169. Argo CD Image Pull vs Pod Image Pull

Important:

```text
Argo CD
```

does not normally pull application images.

The Kubernetes runtime does.

Argo CD applies:

```text
image reference
```

to Kubernetes.

---

# 170. Repository Credential vs ECR Credential

Do not confuse:

```text
Git credentials
```

with:

```text
ECR pull credentials
```

Git is used by Argo CD to obtain desired configuration.

ECR is used by Kubernetes workloads to obtain container images.

---

# 171. Production Identity Map

```text
Developer
   -> SSO

CI
   -> OIDC/IAM role

Argo CD
   -> Git read credential
   -> cluster credential

Pod
   -> ServiceAccount
   -> IAM role

Auditor
   -> SSO
   -> read-only Argo CD role
```

---

# 172. Security Matrix

| Identity | Git | ECR | Argo CD | EKS |
|---|---|---|---|---|
| Developer | source read/write | usually no | limited | limited |
| CI | GitOps PR/write | push | no direct need | no direct need |
| Argo CD | read | usually no | controller | deployment permissions |
| Application Pod | no | pull path | no | own namespace/runtime |
| Auditor | read | no | read-only | read-only if required |

The exact implementation varies.

---

# 173. Human Production Access

Avoid giving developers:

```text
cluster-admin
```

just to troubleshoot applications.

Provide:

```text
read-only
logs
events
namespace-scoped access
```

where possible.

---

# 174. Argo CD Exec Permission

Argo CD can expose pod exec functionality if enabled/configured.

Treat:

```text
exec
```

as a privileged capability.

A shell inside a production pod can be equivalent to significant application access.

Do not grant it broadly.

---

# 175. Log Access

Read-only logs are safer than:

```text
exec
```

for many troubleshooting scenarios.

Use:

```bash
kubectl logs
```

or controlled Argo CD log access.

---

# 176. Delete Permission

Deleting an Argo CD Application can delete managed resources depending on configuration and finalizers.

Therefore:

```text
applications, delete
```

should be tightly restricted.

---

# 177. Prune Security

Automatic pruning can remove resources that disappear from Git.

This is powerful.

Production teams should understand:

```text
prune
```

before enabling it broadly.

---

# 178. Self-Heal Security

Self-heal means Argo CD can correct manual drift.

This improves desired-state integrity.

But if Git contains a bad state:

```text
selfHeal
```

will continuously restore the bad state.

Therefore:

```text
Git security
```

is essential.

---

# 179. Ignore Differences Risk

Overusing:

```yaml
ignoreDifferences
```

can hide important drift.

Only ignore fields that are intentionally mutated by another controller.

---

# 180. Resource Tracking

Argo CD tracks managed resources.

Security concern:

```text
incorrect ownership/tracking
```

can cause unexpected resources to be managed or removed.

Use consistent labels/annotations and avoid ambiguous ownership.

---

# 181. App-of-Apps Security

The root Application can generate child Applications.

Therefore:

```text
root Application
```

is highly privileged.

Protect:

```text
applications/
```

and the root configuration.

---

# 182. ApplicationSet Security

ApplicationSet can dynamically create Applications targeting multiple clusters.

Protect:

```text
applicationsets/
```

because a malicious change could expand deployment scope.

---

# 183. Cluster Generator Security

If using cluster labels:

```text
environment=prod
```

a malicious cluster metadata change could potentially affect ApplicationSet targeting.

Treat cluster registration and labels as trusted control-plane configuration.

---

# 184. Multi-Cluster Security Model

```text
                Argo CD
                   |
        +----------+----------+
        |          |          |
        v          v          v
     DEV EKS    QA EKS     PROD EKS
        |          |          |
      Role       Role       Role
        |          |          |
     Limited    Limited    Restricted
```

Avoid one universal cluster-admin identity if architecture permits narrower permissions.

---

# 185. Cluster Credential Segmentation

Use separate credentials for:

```text
DEV
QA
PROD
```

This prevents one credential compromise from automatically granting every environment.

---

# 186. Multi-Account Security Architecture

```text
                 Central Argo CD
                       |
          +------------+------------+
          |            |            |
          v            v            v
     AWS DEV       AWS QA       AWS PROD
       acct          acct          acct
          |            |            |
         EKS          EKS          EKS
```

Use explicit IAM/network trust relationships.

---

# 187. Production Argo CD Project Model

```text
Project: roboshop-dev
  repos: roboshop-gitops
  cluster: DEV
  namespaces: roboshop-dev

Project: roboshop-qa
  repos: roboshop-gitops
  cluster: QA
  namespaces: roboshop-qa

Project: roboshop-prod
  repos: roboshop-gitops
  cluster: PROD
  namespaces: roboshop
```

---

# 188. Separation of Duties

A mature model:

```text
Developer
   |
   +--> code review

Security
   |
   +--> security policy

Release Manager
   |
   +--> production approval

Platform
   |
   +--> Argo CD / EKS

Application Team
   |
   +--> runtime ownership
```

---

# 189. Four-Eyes Principle

For high-risk production changes:

```text
author != approver
```

This prevents one person from creating and approving a sensitive deployment alone.

---

# 190. Audit Trail

A complete audit trail should connect:

```text
developer
source commit
CI build
security result
image digest
GitOps PR
approver
Argo CD revision
Kubernetes resources
runtime telemetry
```

---

# 191. Security Monitoring

Alert on:

```text
new Argo CD admin
RBAC changes
Project changes
cluster registration changes
repository credential changes
unexpected production sync
ApplicationSet changes
break-glass use
failed authentication spikes
```

---

# 192. Monitoring Argo CD Security

Use the existing:

```text
Prometheus
Grafana
ELK
```

stack where appropriate.

Do not rely only on application metrics.

---

# 193. Argo CD Logs

Inspect:

```bash
kubectl logs -n argocd deploy/argocd-server
kubectl logs -n argocd deploy/argocd-repo-server
kubectl logs -n argocd deploy/argocd-application-controller
```

Actual deployment names can differ by Argo CD version and installation method.

---

# 194. RBAC Troubleshooting

If a user receives:

```text
permission denied
```

check:

```text
identity
group claim
RBAC policy
project role
resource
action
object
```

---

# 195. Check SSO Group Mapping

If a user should have:

```text
argocd-prod-deployers
```

but cannot sync:

```text
verify OIDC group claim
verify group-to-role mapping
verify Argo CD RBAC policy
```

---

# 196. AppProject Permission Failure

If Application creation fails:

```text
check sourceRepos
check destinations
check namespace
check resource whitelist
```

The Project may intentionally deny the operation.

---

# 197. Cluster Permission Failure

If sync fails with:

```text
forbidden
```

check Kubernetes RBAC for the Argo CD identity.

---

# 198. Repository Permission Failure

If Argo CD cannot access Git:

```bash
argocd repo list
```

Then inspect:

```text
credential
URL
TLS
SSH host key
repository permissions
```

---

# 199. Secret Permission Failure

If External Secrets cannot retrieve a secret:

Check:

```text
ServiceAccount
IAM role
trust policy
IAM policy
secret ARN/path
region
network
```

---

# 200. IRSA Troubleshooting

Check:

```bash
kubectl get sa cart -n roboshop -o yaml
```

Verify:

```text
IAM role annotation
```

Then inspect:

```text
IAM trust policy
OIDC provider
ServiceAccount namespace/name
```

---

# 201. EKS Pod Identity Troubleshooting

Verify:

```text
Pod Identity association
ServiceAccount
IAM role
trust configuration
pod restart
```

Use AWS and Kubernetes inspection commands appropriate to the deployed EKS configuration.

---

# 202. Kubernetes RBAC Troubleshooting

Use:

```bash
kubectl auth can-i get deployments \
  --as=system:serviceaccount:roboshop:cart \
  -n roboshop
```

This tests whether the identity has the requested permission.

---

# 203. Argo CD Cluster Permission Troubleshooting

Inspect the cluster registration:

```bash
argocd cluster list
```

Then inspect the target cluster and Kubernetes RBAC.

---

# 204. Unauthorized Argo CD Sync

If:

```bash
argocd app sync roboshop-prod
```

fails due to authorization:

Check:

```text
Argo CD user role
AppProject role
application object
action permission
```

---

# 205. Application Deletion Security

Before deleting:

```bash
argocd app delete roboshop-prod
```

understand:

```text
cascade behavior
finalizers
managed resources
```

A production deletion can remove substantial infrastructure.

---

# 206. Finalizers

Argo CD Applications can use finalizers to control cleanup behavior.

A common cascading finalizer can cause managed resources to be deleted when the Application is deleted.

Treat Application deletion as a privileged operation.

---

# 207. Backup Security

Backups can contain:

```text
cluster credentials
repo credentials
Argo CD configuration
Kubernetes Secrets
```

Encrypt backups.

Restrict backup access.

Test restore.

---

# 208. Disaster Recovery Security

A DR process must restore:

```text
Argo CD configuration
Projects
Applications
ApplicationSets
repository credentials
cluster registrations
secret integrations
```

without exposing sensitive credentials.

---

# 209. Secret Backup

Do not casually export all Kubernetes Secrets into an unprotected backup.

Use:

```text
encrypted backup
KMS
restricted access
retention policy
```

---

# 210. Argo CD High Availability

For production, consider HA for critical Argo CD components.

Security and availability are related:

```text
single component failure
```

should not unnecessarily stop deployment control.

---

# 211. Upgrade Security

Keep Argo CD updated for:

```text
security fixes
bug fixes
Kubernetes compatibility
```

Test upgrades before production.

---

# 212. Upgrade Strategy

Recommended:

```text
read release notes
 |
 v
test DEV
 |
 v
test QA
 |
 v
backup
 |
 v
upgrade PROD
 |
 v
verify applications
```

---

# 213. Version Pinning

Pin:

```text
Argo CD version
Helm chart
controller images
```

rather than silently consuming arbitrary latest versions.

---

# 214. Container Image Security

Use:

```text
trusted registry
immutable references
vulnerability scanning
signing
admission policy
```

where appropriate.

---

# 215. Base Image Vulnerability

A successful deployment can still be insecure if the image contains:

```text
critical CVE
```

Trivy and other scanners should be part of CI.

---

# 216. False Positives

Security scanners can report findings that need validation.

Do not blindly:

```text
ignore all CVEs
```

Use:

```text
risk assessment
exception process
expiration
ownership
```

---

# 217. Security Exceptions

An exception should contain:

```text
finding
reason
risk
owner
approval
expiration
mitigation
```

Avoid permanent undocumented exceptions.

---

# 218. GitOps Security Policy

A production GitOps policy should define:

```text
who can modify GitOps
who can approve PROD
who can sync PROD
who can modify Projects
who can register clusters
who can manage secrets
who can use break-glass
```

---

# 219. Security Ownership Matrix

| Area | Owner |
|---|---|
| Source code | Application team |
| CI | DevOps/platform |
| Security scanning | Security + DevOps |
| ECR | Platform |
| GitOps repo | Platform/GitOps |
| Argo CD | Platform |
| EKS | Cloud/platform |
| IAM | Cloud/security |
| Secrets | Security/platform |
| Production approval | Release/business owner |

Adapt this to the organization.

---

# 220. Production Security YAML Set

A realistic GitOps security repository might include:

```text
security/
├── projects/
│   ├── roboshop-dev.yaml
│   └── roboshop-prod.yaml
├── rbac/
│   └── argocd-rbac.yaml
├── namespaces/
│   ├── dev.yaml
│   └── prod.yaml
├── network-policies/
│   └── roboshop.yaml
├── quotas/
│   └── prod.yaml
└── policies/
    └── admission/
```

---

# 221. Example Production Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
  labels:
    environment: prod
    team: roboshop
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

---

# 222. Example ServiceAccount

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: cart
  namespace: roboshop
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/roboshop-cart-prod
```

If using EKS Pod Identity instead, use the organization's configured association mechanism rather than adding an IRSA annotation unnecessarily.

---

# 223. Example Role

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: cart-config-reader
  namespace: roboshop
rules:
  - apiGroups:
      - ""
    resources:
      - configmaps
    resourceNames:
      - cart-config
    verbs:
      - get
```

---

# 224. Example RoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: cart-config-reader
  namespace: roboshop
subjects:
  - kind: ServiceAccount
    name: cart
    namespace: roboshop
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: cart-config-reader
```

---

# 225. Important RBAC Detail

Use:

```yaml
resourceNames:
```

when possible to limit access to a specific resource.

Compare:

```text
get all ConfigMaps
```

with:

```text
get only cart-config
```

The second is more restrictive.

---

# 226. Kubernetes Secret Access

Avoid giving a workload:

```text
get secrets
```

for the entire namespace.

Prefer an external secret integration or tightly scoped access when application design requires Kubernetes API access.

---

# 227. Application Pods Usually Do Not Need Kubernetes API Access

A normal application:

```text
cart
```

does not need:

```text
get pods
get secrets
list deployments
```

Remove unnecessary permissions.

---

# 228. Disable Automatic ServiceAccount Token Mounting

If a pod does not need the Kubernetes API:

```yaml
automountServiceAccountToken: false
```

Example:

```yaml
spec:
  automountServiceAccountToken: false
```

This reduces unnecessary credential exposure.

---

# 229. Workload Identity Exception

If the workload needs AWS identity:

```text
ServiceAccount
```

may still be used for workload identity depending on the EKS mechanism.

Do not disable or modify identity-related configuration without understanding the selected AWS integration.

---

# 230. Application SecurityContext

A production workload can use:

```yaml
securityContext:
  runAsNonRoot: true
  seccompProfile:
    type: RuntimeDefault
```

and:

```yaml
securityContext:
  allowPrivilegeEscalation: false
  capabilities:
    drop:
      - ALL
```

---

# 231. Linux Capabilities

Dropping:

```text
ALL
```

reduces kernel-level privileges.

Only add a capability when the application genuinely requires it.

---

# 232. Privileged Containers

Avoid:

```yaml
privileged: true
```

for application workloads.

Privileged containers can access powerful host resources.

---

# 233. Host Namespace Access

Avoid unnecessary:

```yaml
hostNetwork: true
hostPID: true
hostIPC: true
```

These can increase host-level exposure.

---

# 234. HostPath

Avoid:

```yaml
hostPath
```

unless absolutely necessary.

It can expose host filesystem resources to containers.

---

# 235. Security Review of Helm Charts

Before allowing a chart into production, inspect:

```text
ServiceAccount
RBAC
securityContext
hostPath
privileged
hostNetwork
hostPID
CRDs
ClusterRole
ClusterRoleBinding
webhooks
```

---

# 236. Third-Party Helm Charts

Do not assume a popular chart is automatically safe.

Review:

```text
source
maintainer
version
permissions
images
RBAC
network access
```

---

# 237. Argo CD and Helm Secrets

Do not put secret values into:

```yaml
values.yaml
```

in plaintext.

Use:

```text
External Secrets
secret references
approved secret plugin/integration
```

---

# 238. Sensitive Helm Values

Avoid:

```yaml
databasePassword: SuperSecret123
```

Instead:

```yaml
existingSecret: cart-secrets
```

and let the secret integration populate it.

---

# 239. Secret References

Example:

```yaml
env:
  - name: DATABASE_PASSWORD
    valueFrom:
      secretKeyRef:
        name: cart-secrets
        key: DATABASE_PASSWORD
```

The value itself is not stored in the deployment manifest.

---

# 240. Secret Name Security

Even secret names can reveal information.

Use consistent but non-sensitive naming.

Avoid embedding:

```text
password value
API key
```

in resource names.

---

# 241. ConfigMap Security

ConfigMaps are not secret stores.

Safe examples:

```text
LOG_LEVEL
ENVIRONMENT
FEATURE_FLAG
SERVICE_URL
```

Unsafe:

```text
password
private key
API token
```

---

# 242. Ingress Security

For AWS ALB:

```text
TLS
HTTPS
security groups
WAF where required
authentication where appropriate
```

should be considered.

---

# 243. ALB TLS

Use ACM-managed certificates where appropriate.

Do not store private TLS keys directly in GitOps repositories when AWS-managed TLS can be used.

---

# 244. ALB Security Group

Restrict:

```text
80/443
```

according to the expected client network.

Prefer HTTPS for production application traffic.

---

# 245. AWS WAF

For Internet-facing production applications, AWS WAF can provide:

```text
managed rules
rate limiting
IP controls
common attack protection
```

It is complementary to Kubernetes security.

---

# 246. GitOps Security Does Not Equal Runtime Security

GitOps protects:

```text
deployment process
desired state
```

but application security still requires:

```text
secure code
secure dependencies
network controls
IAM
runtime hardening
monitoring
```

---

# 247. Security Layers

```text
Code
 |
 v
CI Security
 |
 v
Image Security
 |
 v
GitOps Security
 |
 v
Argo CD RBAC
 |
 v
Kubernetes Admission
 |
 v
Runtime IAM
 |
 v
Network Security
 |
 v
Monitoring
```

---

# 248. Production Security Checklist

```text
[ ] SSO enabled
[ ] MFA enforced by IdP
[ ] Local admin minimized
[ ] RBAC least privilege
[ ] Projects restricted
[ ] Repositories allowlisted
[ ] Destinations restricted
[ ] Cluster credentials protected
[ ] GitOps branch protected
[ ] CODEOWNERS configured
[ ] Production approval enabled
[ ] CI has no cluster-admin
[ ] Argo CD Git access read-only
[ ] Secrets externalized
[ ] IAM workload identity enabled
[ ] Network controls enabled
[ ] TLS enabled
[ ] Webhooks authenticated
[ ] Images scanned
[ ] Images immutable
[ ] Image signing considered
[ ] Admission policies considered
[ ] Audit enabled
[ ] Alerts configured
[ ] Break-glass process documented
[ ] Backups encrypted
[ ] DR tested
```

---

# 249. Security Troubleshooting Runbook

## User cannot log in

Check:

```text
SSO
OIDC
IdP
group claim
certificate
Argo CD server
```

---

# 250. User Can Log In but Cannot Sync

Check:

```text
RBAC
Project role
Application object
resource/action/object
```

---

# 251. Application Cannot Be Created

Check:

```text
AppProject sourceRepos
destinations
namespace
resource restrictions
```

---

# 252. Argo CD Cannot Read Git

Check:

```bash
argocd repo list
```

Then:

```text
credential
repository URL
SSH host key
TLS
network
Git permissions
```

---

# 253. Argo CD Cannot Deploy to EKS

Check:

```text
cluster registration
Kubernetes API connectivity
cluster credentials
Kubernetes RBAC
Project destination
namespace permissions
```

---

# 254. ExternalSecret Fails

Check:

```text
ExternalSecret status
SecretStore
ServiceAccount
IAM role
AWS policy
secret path
AWS region
network
```

---

# 255. Pod Cannot Access AWS

Check:

```text
ServiceAccount
IAM role
OIDC/Pod Identity
trust policy
IAM policy
pod restart
```

---

# 256. Admission Policy Blocks Deployment

Check:

```text
policy violation
securityContext
image registry
image signature
required labels
resources
```

Do not disable the policy as the first response.

---

# 257. GitOps Security Incident Response

If unauthorized production GitOps change is detected:

```text
1. Freeze promotion.
2. Identify commit/PR.
3. Identify actor.
4. Revert malicious desired state.
5. Verify Argo CD.
6. Verify cluster resources.
7. Rotate compromised credentials.
8. Inspect CI logs.
9. Inspect Git audit logs.
10. Inspect AWS/Kubernetes audit logs.
11. Determine blast radius.
12. Document incident.
```

---

# 258. Compromised CI Token

If a GitOps write token is compromised:

```text
revoke immediately
```

Then:

```text
review Git history
review PRs
review automation logs
rotate credentials
verify production
```

---

# 259. Compromised Argo CD Credential

If an Argo CD account is compromised:

```text
disable account/token
rotate credentials
review sync history
review Application changes
review Projects
review clusters
review Kubernetes audit logs
```

---

# 260. Compromised AWS Role

If an IAM role is compromised:

```text
disable/restrict trust
review CloudTrail
revoke sessions where applicable
rotate dependent credentials
inspect accessed resources
```

---

# 261. Malicious GitOps Commit

Suppose:

```yaml
clusterRole:
  rules:
    - apiGroups: ["*"]
      resources: ["*"]
      verbs: ["*"]
```

appears unexpectedly.

Treat it as a security incident.

Do not simply merge/reconcile it because:

```text
Git is source of truth
```

The source itself may be compromised.

---

# 262. Malicious ApplicationSet

An attacker could change:

```text
destination cluster
```

from:

```text
DEV
```

to:

```text
PROD
```

Protect ApplicationSet files with:

```text
CODEOWNERS
branch protection
required review
```

---

# 263. Malicious AppProject

An attacker could change:

```yaml
destinations:
  - server: "*"
    namespace: "*"
```

This can dramatically increase deployment scope.

AppProjects should be platform-owned.

---

# 264. Malicious Repository Source

An attacker could change:

```yaml
sourceRepos:
  - https://malicious.example.com/repo.git
```

if Project permissions are too broad.

Use explicit repository allowlists.

---

# 265. Malicious Image Tag

An attacker could change:

```yaml
tag: latest
```

or point to an untrusted registry.

Controls:

```text
Git review
image allowlist
immutable digests
admission policy
```

---

# 266. Supply-Chain Attack

A dependency could be compromised before CI builds.

Defenses:

```text
dependency scanning
SBOM
trusted registries
signed artifacts
dependency pinning
provenance
```

---

# 267. Container Escape Risk

If an attacker gains code execution inside a privileged container, host compromise may become possible.

Reduce risk through:

```text
non-root
drop capabilities
seccomp
read-only filesystem
no privileged
no host namespaces
Pod Security
```

---

# 268. Kubernetes API Attack

If a pod receives unnecessary Kubernetes API credentials:

```text
compromised application
       |
       v
Kubernetes API
```

could escalate.

Use:

```text
automountServiceAccountToken: false
```

for workloads that do not need Kubernetes API access.

---

# 269. AWS Metadata Attack

Modern EKS workload identity mechanisms reduce reliance on direct instance metadata access.

Use:

```text
IRSA
```

or:

```text
EKS Pod Identity
```

for AWS permissions.

---

# 270. Node Role Permissions

Node IAM roles should not contain broad application permissions.

Application AWS permissions should generally belong to:

```text
workload identity
```

rather than:

```text
node role
```

---

# 271. Security Review Before Production

Review:

```text
Git
CI
ECR
Argo CD
EKS
IAM
Secrets
Network
Admission
Monitoring
DR
```

No single layer is sufficient.

---

# 272. Production RoboShop Security Architecture

```text
Developer
   |
   v
GitHub + SSO/MFA
   |
   v
Jenkins / GitHub Actions
   |
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
ECR
   |
   v
Protected GitOps Repo
   |
   v
PR + CODEOWNERS
   |
   v
Central Argo CD
   |
   +--> Project: DEV
   +--> Project: QA
   +--> Project: PROD
   |
   v
EKS
   |
   +--> Kubernetes RBAC
   +--> Pod Security
   +--> Admission
   +--> NetworkPolicy
   |
   v
RoboShop
   |
   +--> ALB
   +--> Services
   +--> AWS workload identity
   +--> External Secrets
   |
   v
Prometheus / Grafana / ELK
```

---

# 273. RoboShop IAM Example

```text
cart
 |
 v
cart ServiceAccount
 |
 v
cart-prod IAM role
 |
 +--> required AWS permissions only
```

Payment should have its own identity:

```text
payment
 |
 v
payment-prod IAM role
```

---

# 274. RoboShop Secret Example

```text
GitOps
 |
 v
ExternalSecret
 |
 v
AWS Secrets Manager
 |
 v
Kubernetes Secret
 |
 v
payment Pod
```

Git contains no plaintext payment credentials.

---

# 275. RoboShop Production Project

```text
roboshop-prod
```

should allow:

```text
source:
  roboshop-gitops

destination:
  PROD EKS

namespace:
  roboshop
```

and only required resource types.

---

# 276. RoboShop Production RBAC

```text
Developers
  -> read PROD
  -> no sync

Release Managers
  -> read PROD
  -> sync approved applications

Platform Admins
  -> manage Projects/Clusters/Applications

Auditors
  -> read-only
```

---

# 277. RoboShop Security Incident Example

Scenario:

```text
Payment pod is compromised.
```

Attacker attempts:

```text
AWS API
Kubernetes API
other namespaces
```

Controls should limit:

```text
IAM role
ServiceAccount
RBAC
NetworkPolicy
Pod Security
```

---

# 278. Security Interview: Why Is GitOps a Security Improvement?

### Answer

> GitOps can reduce the need for CI systems and developers to hold direct production Kubernetes credentials. CI publishes immutable artifacts and updates Git, while Argo CD inside the deployment environment reconciles the approved desired state. Git protection, RBAC and admission controls then provide multiple security boundaries.

---

# 279. Security Interview: What Is the Biggest Risk of GitOps?

### Answer

> Git becomes a high-value deployment control plane. If an attacker can modify production GitOps configuration, Argo CD may faithfully deploy the malicious state. Therefore repository protection, least privilege, CODEOWNERS, protected branches, approvals and monitoring are critical.

---

# 280. Security Interview: Why Use AppProjects?

### Answer

> AppProjects provide boundaries around source repositories, target clusters, namespaces and resource types. They prevent an application team from arbitrarily deploying to another environment or managing cluster-wide resources.

---

# 281. Security Interview: How Do You Secure Argo CD?

### Answer

> I use SSO/OIDC, least-privilege RBAC, restricted AppProjects, private or controlled API access, TLS, protected repository credentials, restricted cluster credentials, network controls, monitoring, encrypted backups and regular upgrades. I also protect ApplicationSets and Projects because they have broad control-plane impact.

---

# 282. Security Interview: How Do You Manage Secrets?

### Answer

> I avoid plaintext secrets in Git. In AWS/EKS I prefer AWS Secrets Manager combined with External Secrets Operator or another approved secret-management solution. The operator uses workload identity such as IRSA or EKS Pod Identity to access AWS without static credentials.

---

# 283. Security Interview: IRSA vs Static AWS Keys

### Answer

> Static keys are long-lived credentials that are difficult to manage safely. IRSA uses Kubernetes ServiceAccount identity and AWS STS to obtain temporary credentials for an IAM role. This provides better isolation and reduces credential leakage risk.

---

# 284. Security Interview: What Is EKS Pod Identity?

### Answer

> EKS Pod Identity is an AWS-native mechanism for associating Kubernetes ServiceAccounts with IAM roles so pods can obtain AWS permissions without storing static access keys. It provides another workload identity option alongside IRSA.

---

# 285. Security Interview: Should Argo CD Have Git Write Access?

### Answer

> Normally no. Argo CD should read the desired state from Git. CI or a release automation identity may update GitOps through controlled PRs. Separating Git read access from Git write access reduces the blast radius.

---

# 286. Security Interview: Should Jenkins Have Cluster-Admin?

### Answer

> No. In a GitOps architecture Jenkins should normally publish the image and update the GitOps repository. Argo CD should handle cluster deployment. This removes broad Kubernetes credentials from CI.

---

# 287. Security Interview: How Do You Protect Production?

### Answer

> I use protected Git branches, CODEOWNERS, required checks, production approval, restricted Argo CD Projects, least-privilege RBAC, immutable image references, secret management, admission policies and monitoring. I also maintain rollback and break-glass procedures.

---

# 288. Security Interview: Why Is `latest` Unsafe?

### Answer

> It is mutable. The same Git configuration can result in different images at different times. Immutable tags or, preferably, image digests provide deterministic deployment and better auditability.

---

# 289. Security Interview: What Does Kubernetes RBAC Protect?

### Answer

> Kubernetes RBAC controls which identities can perform which API actions on which resources. It can be namespace-scoped through Roles and RoleBindings or cluster-wide through ClusterRoles and ClusterRoleBindings.

---

# 290. Security Interview: Role vs ClusterRole?

### Answer

> A Role is namespace-scoped. A ClusterRole can define permissions for cluster-scoped resources and can also be bound into namespaces. I prefer namespace-scoped Roles whenever possible.

---

# 291. Security Interview: Why Avoid Cluster-Admin?

### Answer

> Cluster-admin can effectively control the entire Kubernetes cluster. If a credential is compromised, the blast radius is enormous. Least-privilege roles reduce the impact of credential compromise.

---

# 292. Security Interview: How Do You Secure Application Pods?

### Answer

> I use non-root execution, dropped Linux capabilities, disabled privilege escalation, seccomp, read-only filesystems where possible, Pod Security standards, resource limits, NetworkPolicies and workload-specific IAM.

---

# 293. Security Interview: What If GitOps Is Compromised?

### Answer

> I revoke the compromised credential, identify unauthorized commits, revert the desired state, inspect Argo CD and Kubernetes audit history, rotate affected credentials, verify cluster resources, and investigate the CI and Git provider logs. Then I strengthen the repository controls that allowed the change.

---

# 294. Security Interview: What If Argo CD Is Compromised?

### Answer

> I treat it as a control-plane incident. I isolate or disable the compromised identity, rotate credentials, inspect Applications, ApplicationSets, Projects and cluster registrations, review Kubernetes audit logs, verify production resources, and restore Argo CD from a trusted configuration if necessary.

---

# 295. Security Interview: Why Protect ApplicationSets?

### Answer

> ApplicationSets can generate many Applications dynamically and can target multiple clusters. A malicious ApplicationSet change can therefore have a much larger blast radius than changing one application.

---

# 296. Security Interview: Why Protect AppProjects?

### Answer

> AppProjects control where applications can deploy and what resources they can manage. Broadening a Project can effectively expand an application's authority across clusters or namespaces.

---

# 297. Security Interview: What Is Defense in Depth?

### Answer

> It means relying on multiple independent controls. For GitOps I would use Git branch protection, CI security scanning, ECR controls, Argo CD RBAC, AppProjects, Kubernetes RBAC, admission policies, IAM workload identity, NetworkPolicies and monitoring. If one layer fails, another can still reduce the impact.

---

# 298. Security Interview: How Do You Secure Multi-Cluster Argo CD?

### Answer

> I isolate target cluster credentials, use environment-specific Projects, restrict destinations, separate AWS accounts where appropriate, use least-privilege Kubernetes access, protect the Argo CD management cluster, monitor changes and maintain encrypted backups. I also consider separate Argo CD instances for highly sensitive boundaries.

---

# 299. Security Interview: How Do You Handle Break-Glass Access?

### Answer

> Break-glass access is reserved for emergencies, strongly authenticated, tightly restricted and audited. After the incident, the temporary access is removed and any manual changes are reconciled back into Git.

---

# 300. Security Interview: What Is the Difference Between Argo CD RBAC and Kubernetes RBAC?

### Answer

> Argo CD RBAC controls what users and groups can do through the Argo CD control plane, such as viewing or syncing Applications. Kubernetes RBAC controls what identities can do against the Kubernetes API. Both layers are needed.

---

# 301. Security Interview: Why Do You Need Both AppProject and Kubernetes RBAC?

### Answer

> They protect different boundaries. AppProject controls Argo CD deployment scope, while Kubernetes RBAC controls API authorization in the cluster. A strong system uses both so an application cannot escape its GitOps-defined scope or Kubernetes permissions.

---

# 302. Security Interview: Why Does GitOps Need Security Monitoring?

### Answer

> GitOps controls production state, so unexpected changes to Projects, ApplicationSets, cluster registrations or production Applications can indicate a high-impact compromise. Monitoring those control-plane changes is therefore important.

---

# 303. Security Interview: How Do You Protect CI Secrets?

### Answer

> I avoid hard-coded credentials, use Jenkins credentials or GitHub Actions secrets/environments, prefer AWS OIDC federation, restrict workflow permissions and prevent untrusted PR code from accessing production secrets.

---

# 304. Security Interview: How Do You Secure GitHub Actions Forks?

### Answer

> I do not expose production credentials to arbitrary forked pull requests. Fork workflows should receive only the minimum permissions required, and privileged deployment jobs should require trusted branches or protected environments.

---

# 305. Security Interview: How Do You Protect GitOps PRs?

### Answer

> I use protected branches, required checks, CODEOWNERS, required reviewers, limited bot permissions, automated manifest validation and production-specific approval rules.

---

# 306. Security Interview: How Do You Protect Argo CD API?

### Answer

> I use TLS, SSO/OIDC, least-privilege RBAC and controlled network access. I avoid unnecessary public exposure and monitor authentication and administrative activity.

---

# 307. Security Interview: How Do You Protect Repository Credentials?

### Answer

> I use dedicated read-only credentials for Argo CD, prefer short-lived or narrowly scoped credentials where supported, store them in secure platform-managed Secrets, rotate them regularly and never commit them to Git.

---

# 308. Security Interview: How Do You Protect ECR?

### Answer

> I use IAM least privilege, repository policies where needed, immutable image promotion, vulnerability scanning, lifecycle controls and potentially image signing/admission verification. CI receives push permissions only for the repositories it owns.

---

# 309. Security Interview: How Do You Prevent Unauthorized Images?

### Answer

> I restrict allowed registries, use immutable image references, scan images in CI, optionally sign them and enforce admission policies that reject images from unapproved registries or without required signatures.

---

# 310. Security Interview: What If CI Says Security Passed but Kubernetes Rejects It?

### Answer

> That can be expected. CI provides pre-deployment validation while Kubernetes admission provides an enforcement boundary at deployment time. I inspect the admission policy violation rather than disabling the control.

---

# 311. Security Interview: What If a Secret Is Committed?

### Answer

> I immediately rotate or revoke the credential. Then I investigate exposure, remove the secret from active history where required, update the secret-management system, inspect access logs and fix the pipeline or developer workflow that allowed the leak.

---

# 312. Security Interview: Why Is Base64 Not Secret Encryption?

### Answer

> Base64 is an encoding mechanism. Anyone who has the encoded value can decode it. Kubernetes Secret security requires proper encryption at rest, access control and secure secret management.

---

# 313. Security Interview: What Is the Most Important GitOps Security Principle?

### Answer

> Protect the desired-state source of truth. Because Argo CD continuously reconciles Git, an unauthorized GitOps change can become a production change. Therefore Git repository security, Argo CD RBAC and Kubernetes authorization must all be treated as production security controls.

---

# 314. Security Interview: Explain Your RoboShop Security Architecture

### Answer

> RoboShop application code is stored in Git and built through Jenkins or GitHub Actions. CI performs tests and DevSecOps checks using SonarQube, Trivy and Veracode, then publishes immutable images to ECR. CI does not have production cluster-admin access. It updates the protected GitOps repository. Argo CD reads that repository and deploys to EKS through controlled Projects and cluster permissions. Secrets are externalized through AWS Secrets Manager and External Secrets, with workload identity using IRSA or EKS Pod Identity. Application pods run with least-privilege ServiceAccounts, security contexts and NetworkPolicies. Prometheus, Grafana and ELK provide operational visibility.

---

# 315. Security Design Review Checklist

Before approving the architecture:

```text
Identity
[ ] SSO
[ ] MFA
[ ] group mapping
[ ] local account minimization

Argo CD
[ ] RBAC
[ ] Projects
[ ] API exposure
[ ] TLS
[ ] component security
[ ] cluster credentials

Git
[ ] branch protection
[ ] CODEOWNERS
[ ] PR approval
[ ] bot permissions
[ ] secret scanning

CI
[ ] OIDC
[ ] ephemeral runners
[ ] no cluster-admin
[ ] protected environments
[ ] least privilege

Images
[ ] immutable
[ ] scanned
[ ] approved registry
[ ] signing where required

Secrets
[ ] no plaintext Git secrets
[ ] Secrets Manager
[ ] External Secrets
[ ] workload identity
[ ] rotation

Kubernetes
[ ] RBAC
[ ] Pod Security
[ ] NetworkPolicy
[ ] admission
[ ] securityContext
[ ] resource limits

AWS
[ ] IAM least privilege
[ ] account separation
[ ] private connectivity where needed
[ ] CloudTrail

Operations
[ ] monitoring
[ ] alerting
[ ] audit
[ ] backup
[ ] DR
[ ] break-glass
```

---

# 316. Final Security Architecture

```text
                         SSO / MFA
                            |
                            v
                       Developers
                            |
                            v
                   Protected GitHub
                            |
                     +------+------+
                     |             |
                     v             v
                    CI        GitOps Repo
                     |             |
              +------+             |
              |                    |
              v                    v
         SonarQube/Trivy       CODEOWNERS
         /Veracode             Branch Protection
              |                    |
              v                    |
             ECR                   |
              |                    |
              +--------->----------+
                            |
                            v
                       Central Argo CD
                            |
                 +----------+----------+
                 |          |          |
                 v          v          v
              Project    Project    Project
                DEV        QA        PROD
                 |          |          |
                 v          v          v
               EKS        EKS        EKS
                 |          |          |
            Kubernetes RBAC / Admission
                 |          |          |
          +------+----------+----------+------+
          |                 |                 |
          v                 v                 v
       Workload          Secrets           Network
       Identity          Manager           Policies
          |                 |                 |
          +-----------------+-----------------+
                            |
                            v
                 Prometheus / Grafana / ELK
```

---

# 317. Final Rules to Remember

```text
1. GitOps repository is a production security boundary.
2. CI should not need cluster-admin.
3. Argo CD should normally have read-only Git access.
4. Protect ApplicationSets.
5. Protect AppProjects.
6. Protect cluster registrations.
7. Use least-privilege RBAC.
8. Separate DEV, QA and PROD permissions.
9. Never store plaintext secrets in Git.
10. Use workload identity for AWS access.
11. Prefer immutable images.
12. Consider image signing.
13. Use admission policy as a final enforcement layer.
14. Protect Argo CD itself as a control plane.
15. Use SSO/MFA for humans.
16. Separate human and machine identities.
17. Monitor security-sensitive GitOps changes.
18. Encrypt backups.
19. Test credential rotation.
20. Test disaster recovery.
21. Maintain break-glass access.
22. Reconcile emergency manual changes back into Git.
23. Treat centralized Argo CD as a high-value control plane.
24. Minimize blast radius across clusters and accounts.
25. Security must cover source -> CI -> registry -> GitOps -> Argo CD -> Kubernetes -> runtime.
```

---