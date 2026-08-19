# Python API Authentication for DevOps Engineers

## 1. Overview

Authentication answers:

> **Who are you?**

Authorization answers:

> **What are you allowed to do?**

For DevOps API automation, authentication is one of the most important production concerns because Python may connect to:

```text
GitHub
Jenkins
GitLab
ArgoCD
Kubernetes
AWS
SonarQube
Prometheus
Elasticsearch
Internal APIs
```

A production automation system must understand:

```text
API keys
Basic authentication
Bearer tokens
Personal access tokens
JWT
OAuth 2.0
OIDC
Access tokens
Refresh tokens
Token expiration
AWS IAM
AWS SigV4
Kubernetes ServiceAccounts
ArgoCD authentication
Secret management
Credential rotation
Least privilege
Authentication troubleshooting
```

The key principle is:

> **Authentication should be secure, short-lived where possible, least-privileged, externally managed, and never exposed in logs or source code.**

---

# 2. Authentication vs Authorization

These are different.

### Authentication

```text
Who are you?
```

Example:

```text
Bearer token
API key
Username/password
OIDC identity
AWS IAM identity
Kubernetes ServiceAccount
```

### Authorization

```text
What can you do?
```

Example:

```text
Read repository
Trigger workflow
Deploy application
Read logs
Modify production
```

Flow:

```text
Client
  |
  | Authentication
  v
Identity
  |
  | Authorization
  v
Allowed operation
```

---

# 3. Example

Suppose Python calls:

```text
POST /deployments
```

The server may first authenticate:

```text
Token valid?
```

Then authorize:

```text
Does this identity have deployment permission?
```

Possible results:

```text
401 Unauthorized
=
Authentication problem

403 Forbidden
=
Authorization problem
```

---

# 4. Why Authentication Matters in DevOps

A deployment automation script can have access to:

```text
Production
Databases
Kubernetes
Container registries
Source repositories
CI/CD
Secrets
Monitoring
```

If its credentials are compromised, the blast radius can be significant.

Therefore:

```text
Least privilege
+
Short-lived credentials
+
Secret management
+
Rotation
+
Audit
```

are essential.

---

# 5. Authentication Methods

Common API authentication methods:

```text
API Key
Basic Authentication
Bearer Token
Personal Access Token
JWT
OAuth 2.0
OIDC
mTLS
AWS IAM / SigV4
Kubernetes ServiceAccount
```

Different APIs use different mechanisms.

Do not assume every API supports Bearer tokens.

---

# 6. API Key Authentication

An API key is usually a static credential identifying the caller.

Example:

```python
headers = {
    "X-API-Key": api_key
}

response = requests.get(
    url,
    headers=headers,
    timeout=10
)
```

Some APIs use:

```text
Authorization: Api-Key <key>
```

The exact format is provider-specific.

---

# 7. API Key Security

Treat an API key like a password.

Do not:

```python
API_KEY = "secret-value"
```

inside source code.

Do not commit:

```text
config.yaml
.env
credentials.json
```

containing real credentials.

Use:

```text
CI secret store
Secret manager
Vault
Environment injection
OIDC
```

where appropriate.

---

# 8. API Key Rotation

Static API keys should be rotatable.

Example:

```text
Old key
   |
   v
Create new key
   |
   v
Update automation
   |
   v
Validate
   |
   v
Revoke old key
```

Avoid:

```text
Delete old key
+
Create new key
+
Hope automation works
```

because this can cause unnecessary downtime.

---

# 9. Basic Authentication

HTTP Basic Authentication sends:

```text
username
password
```

encoded in an HTTP Authorization header.

It is not encryption by itself.

Therefore:

```text
Basic Auth + HTTPS
```

is required for secure transmission.

---

# 10. Requests Basic Authentication

Requests provides:

```python
from requests.auth import HTTPBasicAuth

response = requests.get(
    url,
    auth=HTTPBasicAuth(
        username,
        password
    ),
    timeout=10
)
```

Do not log the `auth` values.

---

# 11. Basic Authentication Use Cases

Common examples:

```text
Internal APIs
Jenkins
Legacy systems
HTTP services
```

But modern systems often prefer:

```text
Bearer tokens
OAuth/OIDC
mTLS
Workload identity
```

depending on requirements.

---

# 12. Bearer Authentication

A bearer token is sent as:

```http
Authorization: Bearer <token>
```

Requests:

```python
headers = {
    "Authorization":
        f"Bearer {access_token}"
}

response = requests.get(
    url,
    headers=headers,
    timeout=10
)
```

---

# 13. Why Is It Called Bearer?

Conceptually:

> Whoever possesses the bearer token can use it.

Therefore token protection is critical.

If leaked:

```text
Attacker
   |
   v
Token
   |
   v
API
```

The API may treat the attacker as the legitimate caller until the token expires or is revoked.

---

# 14. Bearer Token Best Practices

Use:

```text
HTTPS
Short expiration
Least privilege
Secure storage
Rotation
No logging
No source control
```

Where supported, prefer short-lived tokens over long-lived static credentials.

---

# 15. Personal Access Tokens

A Personal Access Token (PAT) is commonly used for developer/platform APIs.

Examples:

```text
GitHub
GitLab
Other developer platforms
```

Example:

```python
headers = {
    "Authorization":
        f"Bearer {token}"
}
```

The exact authentication format depends on the platform.

---

# 16. PAT Security

Do not use a highly privileged personal token for production automation when a dedicated machine identity is available.

Better:

```text
Human account
=
Human operations

Service identity
=
Automation
```

This improves:

```text
Audit
Rotation
Ownership
Least privilege
```

---

# 17. Service Accounts

A service account represents an application or automation workload.

Example:

```text
Jenkins deployment automation
        |
        v
Service identity
        |
        v
ArgoCD API
```

Advantages:

```text
No dependency on employee account
Controlled permissions
Independent rotation
Better auditing
```

---

# 18. Machine Identity

A machine identity should answer:

```text
Which workload is making this request?
```

Examples:

```text
Jenkins release runner
GitHub Actions workflow
Kubernetes pod
AWS EC2 workload
```

Use workload-specific credentials rather than sharing one global credential.

---

# 19. Authentication Architecture

Production:

```text
CI/CD
  |
  v
Workload Identity
  |
  v
Token / Credential
  |
  v
API
  |
  v
Authorization Policy
```

Avoid:

```text
CI/CD
  |
  v
Shared admin password
```

---

# 20. Environment Variables

A simple pattern:

```python
import os

token = os.environ["API_TOKEN"]
```

Configuration:

```text
API_URL
API_TOKEN
ENVIRONMENT
```

Advantages:

```text
No hardcoded secret
Easy CI injection
Easy environment separation
```

But environment variables are not a full secret-management solution by themselves.

---

# 21. CI/CD Secret Injection

Example conceptual flow:

```text
Jenkins Credential Store
        |
        v
Pipeline
        |
        v
Environment variable
        |
        v
Python
```

Python reads:

```python
token = os.environ["API_TOKEN"]
```

The token should not be written to files unless required and securely handled.

---

# 22. Secret Manager

A production platform may use:

```text
AWS Secrets Manager
HashiCorp Vault
Azure Key Vault
Kubernetes Secrets
CI secret stores
```

Architecture:

```text
Python
  |
  v
Secret Provider
  |
  v
Credential
  |
  v
API
```

Use the organization's approved secret-management platform.

---

# 23. Secret Rotation Architecture

Good:

```text
Secret Manager
      |
      v
Short-lived credential
      |
      v
Python
      |
      v
API
```

Rotation happens centrally.

Bad:

```text
credential.py
      |
      v
Hardcoded permanent token
```

---

# 24. OAuth 2.0

OAuth 2.0 is an authorization framework commonly used for delegated access and obtaining access tokens.

Important concepts:

```text
Client
Authorization Server
Resource Server
Access Token
Refresh Token
Scopes
```

Flow:

```text
Client
  |
  v
Authorization Server
  |
  v
Access Token
  |
  v
Resource Server/API
```

OAuth 2.0 is not itself a single authentication protocol.

For user authentication/identity, OIDC builds on OAuth 2.0.

---

# 25. OAuth 2.0 Components

### Resource Owner

The entity controlling the resource.

### Client

The application requesting access.

### Authorization Server

Issues tokens.

### Resource Server

API that accepts access tokens.

Architecture:

```text
User / Workload
      |
      v
Authorization Server
      |
      v
Access Token
      |
      v
Resource Server
```

---

# 26. Access Token

An access token authorizes API access.

Example:

```http
Authorization: Bearer eyJ...
```

It normally has:

```text
Expiration
Scopes
Audience
Issuer
```

depending on token format and provider.

---

# 27. Refresh Token

A refresh token may be used to obtain a new access token without requiring the user/workload to authenticate again.

Conceptually:

```text
Refresh Token
      |
      v
Authorization Server
      |
      v
New Access Token
```

Refresh tokens are highly sensitive and require secure storage.

---

# 28. Access Token vs Refresh Token

| Feature | Access Token | Refresh Token |
|---|---|---|
| Used for API calls | Yes | Usually no |
| Short-lived | Usually | Often longer-lived |
| Sent to resource API | Yes | Usually no |
| Used to obtain new token | No | Yes |
| Sensitivity | High | Very high |

Exact behavior depends on the identity provider.

---

# 29. OAuth Scopes

A scope limits what an access token can do.

Example:

```text
repo:read
repo:write
deploy:read
deploy:write
```

Conceptually:

```text
Token
 |
 +-- repo:read
 +-- deploy:read
```

The API checks whether the requested operation requires an allowed scope.

---

# 30. Least Privilege with Scopes

Bad:

```text
scope = admin:*
```

Better:

```text
scope = deployment:read
```

if the automation only needs read access.

For deployment:

```text
deployment:write
```

may be required.

---

# 31. OAuth Client Credentials Flow

This is particularly relevant for machine-to-machine automation.

Conceptually:

```text
Python workload
      |
      | client credentials
      v
Authorization Server
      |
      v
Access Token
      |
      v
API
```

No interactive user login is required.

Use it when the identity provider/API supports it.

---

# 32. Client Credentials Flow

Conceptual sequence:

```text
1. Python authenticates as a client
2. Authorization server validates client
3. Authorization server issues access token
4. Python sends token to API
5. Token expires
6. Python obtains another token
```

Do not hardcode client secrets in source.

---

# 33. OAuth Authorization Code Flow

Common for user-delegated access:

```text
Browser
 |
 v
Authorization Server
 |
 v
Authorization Code
 |
 v
Client
 |
 v
Access Token
```

This is less common for unattended CI workloads than client credentials or workload identity.

---

# 34. PKCE

PKCE improves authorization-code flows, especially for public clients.

Conceptually:

```text
code_verifier
      |
      v
Authorization
      |
      v
code
      |
      v
Token exchange
      |
      v
Verify code_verifier
```

For backend DevOps automation, use the flow required by the identity provider rather than adding PKCE without understanding the client type.

---

# 35. OIDC

OpenID Connect (OIDC) is an identity layer built on OAuth 2.0.

It adds identity information through an ID token.

Conceptually:

```text
OAuth 2.0
+
Identity
=
OIDC
```

Important for modern cloud CI/CD because workloads can obtain temporary cloud credentials without storing long-lived secrets.

---

# 36. OIDC in GitHub Actions

Conceptual architecture:

```text
GitHub Actions
      |
      | OIDC token
      v
Cloud Identity Provider
      |
      v
Temporary Credentials
      |
      v
AWS
```

This can eliminate long-lived AWS access keys in GitHub Actions.

---

# 37. OIDC Trust Relationship

Example:

```text
GitHub workflow
       |
       v
OIDC token
       |
       v
AWS IAM trust policy
       |
       v
Assume role
       |
       v
Temporary AWS credentials
```

The IAM trust policy should restrict:

```text
Repository
Organization
Branch
Environment
Workflow
Subject claims
```

according to the platform's capabilities and security requirements.

---

# 38. Why OIDC Is Better Than Static Keys

Static:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

may remain valid until rotated.

OIDC:

```text
Workload
 |
 v
Identity token
 |
 v
Temporary role credentials
```

Benefits:

```text
Short lifetime
No permanent cloud secret
Better auditing
Reduced credential leakage risk
```

---

# 39. AWS IAM Authentication

AWS APIs are commonly authenticated using:

```text
IAM credentials
```

and request signing.

Python should normally use:

```text
boto3
```

for AWS API operations.

Example:

```python
import boto3

ecr = boto3.client("ecr")

response = ecr.describe_repositories()
```

This is preferable to manually implementing AWS API signing.

---

# 40. AWS Credential Provider Chain

Boto3 can obtain credentials through supported credential sources such as:

```text
Environment
Shared AWS configuration
IAM role
Container credentials
Instance metadata
Web identity/OIDC
```

The exact resolution order depends on the SDK configuration/version.

The key production principle is:

> Prefer workload identity/roles and temporary credentials over hardcoded static keys.

---

# 41. EC2 IAM Role

Architecture:

```text
EC2
 |
 v
Instance Role
 |
 v
Temporary Credentials
 |
 v
AWS API
```

Python:

```python
import boto3

s3 = boto3.client("s3")

response = s3.list_buckets()
```

No AWS secret needs to be embedded in the Python code.

---

# 42. EKS Pod Identity / IRSA Concepts

For Kubernetes workloads, AWS access can be mapped to workload identity.

Conceptual:

```text
Pod
 |
 v
Kubernetes ServiceAccount
 |
 v
AWS Identity
 |
 v
Temporary Credentials
 |
 v
AWS API
```

Depending on the EKS setup, this can use:

```text
IRSA
EKS Pod Identity
```

Use the mechanism standardized by the environment.

---

# 43. Kubernetes Authentication

The Kubernetes Python client can authenticate using kubeconfig:

```python
from kubernetes import config

config.load_kube_config()
```

For in-cluster automation:

```python
config.load_incluster_config()
```

The second approach uses the pod's Kubernetes identity.

---

# 44. In-Cluster Authentication

Architecture:

```text
Python Pod
    |
    v
Kubernetes ServiceAccount
    |
    v
Token / projected identity
    |
    v
Kubernetes API
```

Then:

```python
from kubernetes import client, config

config.load_incluster_config()

api = client.CoreV1Api()

pods = api.list_namespaced_pod(
    namespace="prod"
)
```

---

# 45. Kubernetes ServiceAccount

A ServiceAccount provides workload identity inside Kubernetes.

Example:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: deployment-automation
  namespace: devops
```

Do not give it cluster-admin unless absolutely required.

---

# 46. Kubernetes RBAC

Authentication:

```text
Who is the ServiceAccount?
```

RBAC:

```text
What can it do?
```

Example:

```text
ServiceAccount
      |
      v
Role
      |
      v
RoleBinding
      |
      v
Allowed API operations
```

---

# 47. Kubernetes Least Privilege

If Python only needs to:

```text
get deployments
get pods
```

do not grant:

```text
create/delete secrets
```

or:

```text
cluster-admin
```

Use the smallest required permission set.

---

# 48. Kubernetes Role Example

Conceptual:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployment-reader
rules:
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list"]
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
```

This is an example only; permissions should match actual requirements.

---

# 49. ArgoCD Authentication

ArgoCD can expose authenticated APIs.

A common pattern is:

```text
Python
 |
 | token
 v
ArgoCD API
```

Example conceptually:

```python
headers = {
    "Authorization":
        f"Bearer {argocd_token}"
}
```

The exact authentication configuration depends on the ArgoCD deployment and identity provider.

---

# 50. ArgoCD Service Account

For automation, prefer a dedicated machine identity/service account.

Example:

```text
automation
 |
 v
ArgoCD project/RBAC
 |
 v
Allowed applications
```

Do not use an administrator token for every deployment workflow.

---

# 51. ArgoCD RBAC

A production automation identity may need:

```text
applications:get
applications:sync
applications:action
```

depending on workflow.

It should not automatically receive:

```text
full admin
```

permissions.

---

# 52. GitHub Authentication

GitHub automation may use:

```text
GitHub App
Fine-grained PAT
GitHub Actions OIDC
OAuth
```

For organization-level automation, GitHub Apps can provide a dedicated machine identity with scoped permissions.

---

# 53. GitHub App Concept

Architecture:

```text
Python
 |
 v
GitHub App Identity
 |
 v
Installation Token
 |
 v
GitHub API
```

Installation tokens are preferable to sharing a personal administrator token for many automation scenarios.

---

# 54. Fine-Grained Tokens

A fine-grained token can restrict:

```text
Repository
Permissions
Operations
```

This is better than a broad token when supported.

---

# 55. Jenkins Authentication

Jenkins may use:

```text
Username + API token
Crumb
SSO
Service account
```

depending on configuration.

For API automation:

```text
Dedicated Jenkins service account
+
API token
+
minimum permissions
```

is a common pattern.

---

# 56. Jenkins Crumb

Some Jenkins configurations use CSRF protection requiring a crumb.

Conceptually:

```text
Authenticate
 |
 v
Get crumb
 |
 v
POST protected action
```

Whether a crumb is required depends on the Jenkins security configuration.

---

# 57. Authentication Flow with Crumb

```text
Python
 |
 | authenticate
 v
Jenkins
 |
 | crumb
 v
Python
 |
 | POST
 v
Jenkins
```

Do not assume every Jenkins API call requires a crumb.

---

# 58. SonarQube Authentication

SonarQube commonly supports token-based authentication.

Conceptually:

```text
Python
 |
 v
Authorization header
 |
 v
SonarQube API
```

Use a dedicated token with only required access.

---

# 59. Prometheus Authentication

Prometheus deployments may be:

```text
Unauthenticated internally
Basic Auth
Reverse proxy protected
OIDC
mTLS
```

Do not assume Prometheus itself always handles authentication.

Often authentication is enforced by:

```text
Ingress
Reverse proxy
API gateway
```

---

# 60. Elasticsearch Authentication

Elasticsearch may use:

```text
Basic Auth
API keys
Bearer tokens
mTLS
```

depending on deployment and security configuration.

Example:

```python
from requests.auth import HTTPBasicAuth

response = requests.get(
    url,
    auth=HTTPBasicAuth(
        username,
        password
    ),
    timeout=10
)
```

Use the authentication method configured by the cluster.

---

# 61. Authentication Header Centralization

Avoid:

```python
headers = {
    "Authorization": ...
}
```

in every function.

Prefer:

```python
session.headers.update({
    "Authorization":
        f"Bearer {token}"
})
```

This reduces duplication and mistakes.

---

# 62. Token Injection

Example:

```python
class APIClient:

    def __init__(
        self,
        base_url,
        token
    ):
        self.session = requests.Session()

        self.session.headers.update({
            "Authorization":
                f"Bearer {token}"
        })
```

The token should enter the client from secure configuration.

---

# 63. Token Redaction

If logs contain:

```text
Authorization
```

redact it.

Example concept:

```text
Authorization: Bearer ****REDACTED****
```

Never print the complete token.

---

# 64. Secret Redaction in CI

CI platforms may mask known secrets.

Still, do not rely exclusively on masking.

Bad:

```bash
set -x
curl -H "Authorization: Bearer $TOKEN" ...
```

This can expose secrets in logs depending on shell/tool behavior.

---

# 65. Curl Authentication Testing

Safe:

```bash
curl \
  -H "Authorization: Bearer $TOKEN" \
  https://api.example.com/health
```

But avoid:

```bash
curl -v ...
```

when sensitive headers may be printed.

Use sanitized debugging.

---

# 66. Token Expiration

Suppose:

```text
Token expires in 15 minutes
```

and deployment takes:

```text
30 minutes
```

The token may expire during polling.

Automation must understand:

```text
Token expiration
Refresh
Re-authentication
```

where supported.

---

# 67. 401 During Long-Running Workflow

Flow:

```text
Deploy
 |
 v
Poll
 |
 v
401
```

Possible cause:

```text
Access token expired
```

Correct action:

```text
Refresh/re-authenticate
```

if the authentication mechanism supports it.

Do not endlessly retry the expired token.

---

# 68. Refresh Strategy

Conceptually:

```python
try:
    call_api()
except AuthenticationExpired:
    refresh_token()
    call_api()
```

The refresh should be:

```text
bounded
secure
observable
```

---

# 69. Token Caching

If a token is valid for several minutes, avoid obtaining a new token for every API request.

Better:

```text
Token cache
 |
 +-- valid -> reuse
 |
 +-- expired/near expiry -> refresh
```

This reduces:

```text
Authentication API calls
Latency
Rate-limit pressure
```

---

# 70. Token Expiry Buffer

Do not wait until the exact expiration timestamp.

Example:

```text
Expires in 10 seconds
```

Treat it as near expiry and refresh.

A small safety buffer reduces race conditions.

---

# 71. JWT

A JSON Web Token may contain claims such as:

```text
iss
sub
aud
exp
iat
scope
roles
```

Example conceptual payload:

```json
{
  "iss": "https://issuer.example.com",
  "sub": "deployment-bot",
  "aud": "deployment-api",
  "exp": 1780000000
}
```

Do not assume every JWT claim has the same meaning across providers.

---

# 72. JWT Structure

A JWT commonly has:

```text
header.payload.signature
```

The payload is encoded, not necessarily encrypted.

Therefore:

> **Never put secrets or sensitive data into JWT claims unless the token design explicitly requires and protects them.**

---

# 73. JWT Validation

The API should validate:

```text
Signature
Issuer
Audience
Expiration
Not-before
Algorithm
Scopes/roles
```

depending on the identity provider and token policy.

---

# 74. JWT Signature

A JWT signature provides integrity/authenticity when validated against the correct signing keys.

Do not trust a JWT merely because:

```python
token.split(".")
```

works.

Signature verification must happen using a trusted key and approved library.

---

# 75. JWT Algorithm Confusion

Do not implement JWT verification manually.

Use a mature security library and enforce allowed algorithms.

The server should not blindly trust the algorithm supplied by an untrusted token.

---

# 76. OIDC ID Token vs Access Token

Important distinction:

```text
ID Token
=
identity information for the client

Access Token
=
authorization to call resource APIs
```

Do not automatically send an ID token to an API that expects an access token.

---

# 77. Audience

An access token may be intended for:

```text
deployment-api
```

If a token is issued for:

```text
another-service
```

the API should reject it.

This is why:

```text
aud
```

matters.

---

# 78. Issuer

The API should validate the token issuer:

```text
iss
```

This prevents accepting tokens from an untrusted identity provider.

---

# 79. Clock Skew

Authentication systems depend on time.

If systems have clock differences:

```text
Token valid until 12:00
Server thinks it is 12:01
```

authentication may fail.

Production environments should maintain accurate time synchronization.

---

# 80. mTLS

Mutual TLS authenticates both sides:

```text
Client certificate
        +
Server certificate
```

Flow:

```text
Python
 |
 | Client certificate
 v
Server
 |
 | Server certificate
 v
Python
```

This is useful for highly trusted internal service-to-service communication.

---

# 81. mTLS in Requests

Conceptually:

```python
response = requests.get(
    url,
    cert=(
        "/path/client.crt",
        "/path/client.key"
    ),
    verify="/path/ca.pem",
    timeout=10
)
```

Private keys must be securely protected.

---

# 82. mTLS Architecture

```text
Python workload
 |
 +-- Client Certificate
 +-- Client Private Key
 |
 v
Internal API
 |
 +-- Server Certificate
 +-- Trusted CA
```

The server verifies the client certificate.

---

# 83. Certificate Rotation

Certificates expire.

Production systems need:

```text
Issue
Deploy
Validate
Rotate
Revoke
```

Do not wait until expiration to discover that automation depends on an old certificate.

---

# 84. Secret vs Certificate

### Secret

```text
API token
Password
Private key
```

### Certificate

```text
Public identity
```

A private key is still secret material.

Never expose private keys in logs or source control.

---

# 85. Authentication Through Reverse Proxy

Architecture:

```text
Python
 |
 v
ALB / Ingress / API Gateway
 |
 | Authentication
 v
Service
```

Authentication may occur at:

```text
Gateway
Proxy
Application
```

or multiple layers.

---

# 86. Authentication vs Network Security

A private endpoint is not automatically authenticated.

Example:

```text
Private VPC
+
No authentication
```

still means any workload with network access may potentially call it.

Use:

```text
Network controls
+
Authentication
+
Authorization
```

---

# 87. Defense in Depth

Production:

```text
Network
   +
TLS
   +
Authentication
   +
Authorization
   +
Application validation
   +
Audit
```

No single control should be treated as sufficient for sensitive operations.

---

# 88. Secret Store Access

If Python itself needs to read secrets:

```text
Python
 |
 v
Workload identity
 |
 v
Secret manager
 |
 v
Secret
 |
 v
Target API
```

This creates two authentication relationships:

```text
Python -> Secret Manager
Python -> Target API
```

Both need least privilege.

---

# 89. Bootstrap Problem

How does Python authenticate to retrieve its API secret?

Use:

```text
Workload identity
```

rather than:

```text
hardcoded master secret
```

Example:

```text
EKS Pod
 |
 v
AWS workload identity
 |
 v
Secrets Manager
 |
 v
API token
```

---

# 90. Secret Retrieval Timing

Prefer retrieving secrets:

```text
At runtime
```

rather than baking them into:

```text
Docker image
```

Example:

```text
Image
=
code only

Runtime
=
credential injection
```

---

# 91. Secret Lifetime

If possible:

```text
Short-lived credential
```

is safer than:

```text
Permanent credential
```

Examples:

```text
OIDC token
Temporary AWS credentials
Short-lived access token
Dynamic Vault secret
```

---

# 92. Credential Scope

Credential scope should match:

```text
Application
Environment
Resource
Operation
```

Example:

```text
staging deployment bot
```

should not automatically deploy to:

```text
production
```

---

# 93. Production Credential Separation

Use separate identities:

```text
dev automation
staging automation
production automation
```

Avoid:

```text
one token for all environments
```

This limits blast radius.

---

# 94. Production Approval

Authentication does not mean authorization to deploy.

Even if Python has:

```text
valid production token
```

the workflow may still require:

```text
approval
change ticket
environment protection
manual gate
```

Do not bypass those controls.

---

# 95. Authentication Audit

Track:

```text
Identity
API
Operation
Environment
Timestamp
Result
Request ID
```

This answers:

```text
Who deployed?
```

and:

```text
Which credential/identity called the API?
```

---

# 96. Authentication Failure Matrix

| Status | Meaning | Typical Action |
|---|---|---|
| 400 | Invalid request | Fix request |
| 401 | Authentication failure | Check token/credential |
| 403 | Authorization failure | Check permissions |
| 429 | Rate limit | Backoff |
| 500+ | Server issue | Retry if safe |

Do not treat 401 and 403 as the same problem.

---

# 97. 401 Troubleshooting

Check:

```text
Token exists
Token not expired
Correct token format
Correct Authorization header
Correct issuer
Correct audience
Correct credentials
Clock synchronization
```

If using OAuth:

```text
Access token vs ID token
```

is also important.

---

# 98. 403 Troubleshooting

Check:

```text
User/service identity
Role
Scope
RBAC
Repository permissions
Environment permissions
ArgoCD project permissions
AWS IAM policy
Kubernetes RBAC
```

The credential may be valid but insufficient.

---

# 99. Authentication Troubleshooting Flow

```text
Request fails
 |
 v
Status code?
 |
 +-- 401
 |    |
 |    v
 |  Authentication
 |
 +-- 403
 |    |
 |    v
 |  Authorization
 |
 +-- 429
 |    |
 |    v
 |  Rate limit
 |
 +-- 5xx
      |
      v
   Server/dependency
```

---

# 100. Debugging Without Leaking Secrets

Do not print:

```text
TOKEN
PASSWORD
PRIVATE KEY
Authorization header
Cookie
```

Instead print:

```text
Token present: yes
Token length: <redacted>
Token expires: <timestamp>
API host: api.example.com
Status: 401
Request ID: abc123
```

Even token length can be unnecessary; use only safe diagnostics.

---

# 101. Authentication Testing

Test:

```text
Valid credential
Expired credential
Missing credential
Wrong credential
Insufficient scope
Wrong audience
Wrong issuer
Revoked credential
Certificate failure
```

This is important for production automation.

---

# 102. Secret Rotation Testing

A production system should be tested for:

```text
Old secret
New secret
Transition period
Revocation
Failure recovery
```

Do not assume rotation works just because a secret manager shows a new version.

---

# 103. Credential Rotation Without Downtime

A safe pattern:

```text
Old credential valid
       |
       v
Create new credential
       |
       v
Deploy/update automation
       |
       v
Validate
       |
       v
Revoke old credential
```

This provides a controlled transition.

---

# 104. Authentication in Containers

Bad:

```dockerfile
ENV API_TOKEN=secret
```

because the secret can become part of image/config metadata depending on how it is built and deployed.

Better:

```text
Image
=
application

Runtime
=
secret injection
```

---

# 105. Authentication in Kubernetes

Possible pattern:

```text
Deployment
 |
 v
ServiceAccount
 |
 v
Workload Identity
 |
 v
Cloud/API
```

If a static secret is unavoidable:

```text
Secret
 |
 v
Pod
```

with:

```text
RBAC
Encryption at rest
Rotation
Access controls
```

according to platform standards.

---

# 106. Kubernetes Secret Is Not Automatically a Secret Vault

A Kubernetes Secret is an API object.

Base64 encoding is not encryption.

Therefore:

```text
base64 != encryption
```

Use appropriate encryption-at-rest and access controls.

For highly sensitive credentials, consider an external secret-management solution.

---

# 107. ServiceAccount Token Projection

Modern Kubernetes environments can use projected, short-lived ServiceAccount tokens.

Conceptually:

```text
Pod
 |
 v
Projected token
 |
 v
API
```

This is generally preferable to unnecessarily distributing long-lived static tokens.

---

# 108. AWS Authentication in EKS

A modern architecture can be:

```text
Python Pod
 |
 v
Kubernetes identity
 |
 v
AWS workload identity
 |
 v
Temporary AWS credentials
 |
 v
ECR/S3/etc.
```

This avoids static AWS access keys in the container.

---

# 109. GitHub Actions Authentication to AWS

Preferred modern pattern:

```text
GitHub Actions
 |
 v
OIDC
 |
 v
AWS IAM Role
 |
 v
Temporary credentials
 |
 v
AWS API
```

Trust should be tightly scoped.

---

# 110. Jenkins Authentication to AWS

Depending on Jenkins deployment:

```text
Jenkins
 |
 v
Instance role
```

or:

```text
Jenkins
 |
 v
Workload identity
 |
 v
AWS role
```

or an approved credential mechanism.

Avoid storing long-lived root/admin credentials.

---

# 111. Authentication and CI/CD Permissions

A deployment pipeline may need:

```text
GitHub:
read repo

ECR:
push image

ArgoCD:
sync application

Kubernetes:
read status

Prometheus:
query metrics
```

Do not grant:

```text
AWS AdministratorAccess
Kubernetes cluster-admin
GitHub organization owner
```

just because the pipeline is important.

---

# 112. Permission Mapping

Create a matrix:

| System | Identity | Required Permission |
|---|---|---|
| GitHub | release-bot | workflow read/dispatch |
| ECR | CI role | image push/read |
| ArgoCD | deploy-bot | application read/sync |
| Kubernetes | verifier SA | deployment/pod read |
| Prometheus | monitor identity | query |
| ELK | log-reader | search/read |

This is a useful production design artifact.

---

# 113. Authentication Architecture for Your DevOps Stack

A strong architecture could look like:

```text
                    CI/CD
                      |
          +-----------+-----------+
          |                       |
          v                       v
       GitHub                   Jenkins
          |                       |
          +-----------+-----------+
                      |
                      v
               Python Automation
                      |
          +-----------+-----------+
          |                       |
          v                       v
   Workload Identity        Secret Provider
          |                       |
          +-----------+-----------+
                      |
                      v
                 API Clients
                      |
      +---------------+---------------+
      |       |       |       |       |
      v       v       v       v       v
   GitHub  ArgoCD   AWS    K8s   Observability
```

---

# 114. Recommended Authentication Strategy

For modern DevOps automation:

```text
Human:
SSO/OIDC

CI workload:
Workload identity/OIDC

AWS:
IAM role / temporary credentials

Kubernetes:
ServiceAccount + RBAC

ArgoCD:
Dedicated service identity + RBAC

GitHub:
GitHub App or scoped token

Internal APIs:
OAuth/OIDC or mTLS where appropriate

Legacy APIs:
Basic/API key only when required
```

---

# 115. Authentication Anti-Patterns

## Anti-pattern 1

Hardcoded password:

```python
password = "admin123"
```

---

## Anti-pattern 2

Shared admin token:

```text
Every pipeline
     |
     v
admin-token
```

---

## Anti-pattern 3

Long-lived AWS keys in GitHub secrets when OIDC is available.

---

## Anti-pattern 4

Cluster-admin ServiceAccount for read-only verification.

---

## Anti-pattern 5

`verify=False` to bypass TLS problems.

---

## Anti-pattern 6

Logging Authorization headers.

---

## Anti-pattern 7

Using employee PAT for production automation.

---

## Anti-pattern 8

One credential for dev/staging/prod.

---

## Anti-pattern 9

Using ID token where access token is required.

---

## Anti-pattern 10

Ignoring token expiration.

---

# 116. Authentication Interview Questions

## Q1. Authentication vs authorization?

Authentication verifies identity.

Authorization determines permitted actions.

---

## Q2. 401 vs 403?

```text
401 = authentication problem
403 = authorization problem
```

---

## Q3. What is a bearer token?

A token presented by the caller to access an API. Whoever possesses it may be able to use it, so it must be protected.

---

## Q4. Why should tokens be short-lived?

A shorter lifetime reduces the window in which a stolen token can be abused.

---

## Q5. What is OAuth 2.0?

An authorization framework used to obtain access tokens for protected resources.

---

## Q6. What is OIDC?

An identity layer built on OAuth 2.0 that provides standardized identity information.

---

## Q7. Access token vs ID token?

Access token:

```text
API authorization
```

ID token:

```text
Client identity information
```

Do not interchange them without understanding the provider's design.

---

## Q8. What is a refresh token?

A credential used by supported OAuth flows to obtain a new access token after/near access-token expiration.

---

## Q9. Why use OIDC in CI/CD?

It can provide short-lived workload credentials without storing long-lived cloud access keys.

---

## Q10. How does GitHub Actions authenticate to AWS using OIDC?

```text
GitHub workflow
 |
 v
OIDC token
 |
 v
AWS IAM trust policy
 |
 v
Assume role
 |
 v
Temporary credentials
 |
 v
AWS API
```

---

# 117. Interview Question — Kubernetes

## How does a Python application authenticate to Kubernetes from inside a pod?

Use:

```python
config.load_incluster_config()
```

The workload uses its Kubernetes ServiceAccount identity and associated permissions.

Authorization is controlled using:

```text
Role
RoleBinding
```

or:

```text
ClusterRole
ClusterRoleBinding
```

as appropriate.

---

# 118. Interview Question — AWS

## How should Python authenticate to AWS from EKS?

Prefer workload identity such as:

```text
EKS Pod Identity
```

or:

```text
IRSA
```

depending on the environment.

Then use:

```python
boto3
```

without hardcoding access keys.

---

# 119. Interview Question — ArgoCD

## How would you authenticate Python to ArgoCD?

Use a dedicated machine identity/token or the organization's configured SSO/service-account mechanism, with ArgoCD RBAC restricted to the required applications and actions.

Never use a global administrator credential if a scoped identity is sufficient.

---

# 120. Interview Question — GitHub

## How would you authenticate automation to GitHub?

Depending on requirements:

```text
GitHub App
Fine-grained PAT
GitHub Actions token
OIDC for supported cloud integrations
```

For long-running organizational automation, a dedicated GitHub App can provide better separation from human identities.

---

# 121. Scenario — Token Returns 401

### Problem

Python worked yesterday but now returns:

```text
401 Unauthorized
```

### Investigation

```text
1. Check token expiration
2. Check token rotation
3. Check secret injection
4. Check Authorization header format
5. Check issuer/audience if OAuth
6. Check API endpoint
7. Check server authentication logs
```

Do not immediately regenerate credentials without finding the cause.

---

# 122. Scenario — Token Returns 403

### Problem

Token is valid but deployment returns:

```text
403
```

### Investigation

```text
Identity
Scope
Role
RBAC
Repository permission
Environment policy
ArgoCD RBAC
AWS IAM
```

The credential may authenticate successfully but lack authorization.

---

# 123. Scenario — AWS Credentials Expire

### Problem

Long-running pipeline starts successfully but later AWS calls fail.

### Cause

Temporary credentials expired.

### Solution

Use SDK credential refresh mechanisms/workload identity appropriately rather than embedding static credentials.

---

# 124. Scenario — Kubernetes 403

### Problem

Python can connect to the cluster but cannot list pods.

### Answer

Authentication succeeded.

Authorization failed.

Check:

```bash
kubectl auth can-i \
  list pods \
  --as=system:serviceaccount:devops:deployment-automation \
  -n prod
```

Use the appropriate identity and namespace for the actual environment.

---

# 125. Scenario — GitHub Token Leak

### Problem

A token appears in CI logs.

### Immediate actions

```text
1. Revoke/rotate token
2. Determine exposure
3. Inspect audit logs
4. Replace credential
5. Search repositories/history if needed
6. Identify how logging occurred
7. Fix masking and code
```

Do not assume CI masking makes a leaked token safe.

---

# 126. Scenario — Certificate Expired

### Problem

API authentication fails because TLS certificate expired.

### Answer

TLS authentication/transport must be fixed.

Do not solve it with:

```python
verify=False
```

Instead:

```text
Renew certificate
Update trust chain
Validate hostname
Test CA
```

---

# 127. Scenario — OIDC Trust Failure

### Problem

GitHub Actions cannot assume an AWS role.

### Check

```text
OIDC provider configured
IAM trust policy
Repository
Branch/environment
Subject claim
Audience
Role ARN
Permissions
Workflow permissions
```

Also ensure the workflow requests the required OIDC token permission.

---

# 128. Scenario — Wrong Token Type

### Problem

API returns:

```text
401
```

even though the token is valid.

### Possible cause

Python sends:

```text
ID token
```

while API expects:

```text
Access token
```

Check:

```text
audience
issuer
token type
scope
```

---

# 129. Senior Interview Question

## Design authentication for a production deployment platform.

### Strong Answer

I would separate identities by workload and environment:

```text
GitHub
 |
 | GitHub App / scoped token
 v
Source operations

Jenkins
 |
 | workload/service identity
 v
CI operations

CI workload
 |
 | OIDC/workload identity
 v
AWS temporary credentials

Deployment automation
 |
 | dedicated ArgoCD identity
 v
ArgoCD RBAC

Kubernetes verifier
 |
 | ServiceAccount
 v
Kubernetes RBAC

Monitoring
 |
 | read-only identity
 v
Prometheus/ELK
```

I would also implement:

```text
Least privilege
Short-lived credentials
Secret manager
Rotation
Audit logging
TLS
No secret logging
Environment separation
Production approvals
```

---

# 130. Senior Scenario — One Token for Everything

## Question

Your team wants one token that can:

```text
GitHub
Jenkins
ArgoCD
AWS
Kubernetes
```

### Answer

I would reject this design.

A single credential creates excessive blast radius.

Instead:

```text
GitHub identity
Jenkins identity
AWS role
ArgoCD identity
Kubernetes ServiceAccount
```

Each should have only the permissions it requires.

---

# 131. Senior Scenario — Secret Manager Is Down

## Question

What should deployment automation do if it cannot retrieve credentials?

### Answer

Fail closed.

Do not:

```text
Use hardcoded fallback password
Use old unknown credential
Disable authentication
```

Instead:

```text
Secret retrieval fails
 |
 v
Clear error
 |
 v
Pipeline stops
 |
 v
Alert/operator action
```

Security controls should fail closed for sensitive operations.

---

# 132. Senior Scenario — Credential Rotation

## Question

How do you rotate production credentials without downtime?

### Answer

Use overlapping validity where supported:

```text
Old credential valid
        |
        v
Create new credential
        |
        v
Deploy/update consumers
        |
        v
Validate
        |
        v
Revoke old credential
```

Then verify all consumers use the new credential.

---

# 133. Senior Scenario — Production Token Scope

## Question

A deployment bot only needs to read ArgoCD application health and sync one application.

What permissions should it receive?

### Answer

Only the required:

```text
application read
+
specific sync permission
+
specific application/project scope
```

Not:

```text
ArgoCD admin
```

This is least privilege.

---

# 134. Authentication Decision Tree

```text
Is API cloud-native?
 |
 +-- AWS
 |    |
 |    v
 |  IAM/workload identity
 |
 +-- Kubernetes
 |    |
 |    v
 |  ServiceAccount + RBAC
 |
 +-- GitHub
 |    |
 |    v
 |  GitHub App/scoped token
 |
 +-- Internal modern API
 |    |
 |    v
 |  OAuth/OIDC
 |
 +-- Legacy API
      |
      v
   API key/basic auth
   if required
```

Use the provider's supported and secure mechanism.

---

# 135. Production Authentication Checklist

```text
Identity
[ ] Dedicated machine identity
[ ] Human and automation identities separated
[ ] Environment-specific identities

Credentials
[ ] No hardcoded secrets
[ ] Secret manager/CI store
[ ] Short-lived where possible
[ ] Rotation process
[ ] Revocation process

Authorization
[ ] Least privilege
[ ] Scopes limited
[ ] RBAC limited
[ ] Production access restricted

Transport
[ ] HTTPS
[ ] TLS verification
[ ] Trusted CA
[ ] mTLS where required

OAuth/OIDC
[ ] Correct issuer
[ ] Correct audience
[ ] Correct scopes
[ ] Expiration handled
[ ] Refresh handled where applicable

CI/CD
[ ] OIDC/workload identity where supported
[ ] No long-lived cloud keys where avoidable
[ ] Secret masking
[ ] No secret logging

Kubernetes
[ ] ServiceAccount
[ ] RBAC
[ ] No unnecessary cluster-admin
[ ] In-cluster authentication

AWS
[ ] IAM role/workload identity
[ ] Temporary credentials
[ ] Least privilege

Operations
[ ] Audit logging
[ ] Request IDs
[ ] Credential rotation tested
[ ] Failure handling
```

---

# 136. Production Authentication Architecture

```text
                           CI/CD
                             |
               +-------------+-------------+
               |                           |
               v                           v
          GitHub Actions                Jenkins
               |                           |
               | OIDC                      | Workload Identity
               v                           v
          Identity Provider          Credential Provider
               |                           |
               v                           v
          AWS IAM Role                Temporary Identity
               |                           |
               +-------------+-------------+
                             |
                             v
                      Python Automation
                             |
          +------------------+------------------+
          |                  |                  |
          v                  v                  v
      GitHub API          ArgoCD             EKS
          |                  |                  |
      GitHub App        Service Identity   ServiceAccount
          |                  |                  |
          v                  v                  v
      Scoped API          ArgoCD RBAC       K8s RBAC
                             |
                             v
                        AWS APIs
                             |
                        IAM Policy
```

---

# 137. Authentication Mental Model

Think:

```text
IDENTITY
   |
   v
CREDENTIAL
   |
   v
AUTHENTICATION
   |
   v
AUTHORIZATION
   |
   v
RESOURCE
```

For production:

```text
Identity
+
Short-lived credential
+
TLS
+
Least privilege
+
Audit
+
Rotation
```

---

# 138. Most Important Interview Points

Remember these:

```text
401 != 403

Authentication != Authorization

Bearer token != ID token

Access token != Refresh token

JWT payload is encoded, not automatically encrypted

OIDC builds identity on OAuth 2.0

Use workload identity for CI/cloud where possible

Use boto3 for AWS APIs

Use Kubernetes ServiceAccount + RBAC for in-cluster automation

Do not hardcode credentials

Do not log credentials

Do not disable TLS verification

Use dedicated service identities

Use least privilege

Rotate credentials

Prefer short-lived credentials

Reconcile after uncertain state-changing requests
```

---

# 139. Final Production Principle

A secure API automation system should look like:

```text
                    Workload
                       |
                       v
                Workload Identity
                       |
                       v
               Short-Lived Token
                       |
                       v
                    HTTPS
                       |
                       v
                    API
                       |
                       v
                Authentication
                       |
                       v
                 Authorization
                       |
                       v
                  Operation
                       |
                       v
                    Audit
```

Not:

```text
Python
 |
 v
admin-password.txt
 |
 v
Production
```

---

# 140. What You Should Be Able to Explain

Before moving to the next topic, you should be comfortable explaining:

```text
Authentication vs authorization
API keys
Basic authentication
Bearer tokens
PATs
Service accounts
Machine identities
Secret management
Credential rotation
OAuth 2.0
Client credentials
Authorization code
PKCE
OIDC
Access tokens
Refresh tokens
ID tokens
JWT
Issuer
Audience
Scopes
Token expiration
Token refresh
mTLS
TLS verification
AWS IAM
Boto3 credential chain
AWS workload identity
EKS authentication
Kubernetes ServiceAccounts
Kubernetes RBAC
ArgoCD authentication
GitHub authentication
GitHub Apps
Jenkins authentication
SonarQube authentication
Prometheus authentication
Elasticsearch authentication
401 troubleshooting
403 troubleshooting
OIDC trust troubleshooting
Secret leakage response
Credential rotation
Least privilege
Production authentication architecture
```

---

# 141. Connection to the Next Topic

Current progress:

```text
08-Python-APIs/
├── 01-HTTP-and-REST.md       ✓
├── 02-Requests-Library.md    ✓
├── 03-API-Automation.md      ✓
├── 04-Authentication.md      ✓
├── 05-API-Error-Handling.md
└── 06-DevOps-API-Projects.md
```

Next:

# `05-API-Error-Handling.md`

That file will focus deeply on:

```text
HTTP errors
Connection errors
Timeouts
Retries
Backoff
Jitter
Rate limits
Circuit breakers
Bulkheads
Fallbacks
Idempotency
Distributed failures
Partial failures
Exception design
Structured error handling
API error classification
Observability
Alerting
Production troubleshooting
Failure recovery
Rollback
Testing failure scenarios
Senior-level interview questions
Production incident scenarios
```

The progression is:

```text
HTTP/REST
      ↓
Requests
      ↓
API Automation
      ↓
Authentication
      ↓
Error Handling
      ↓
DevOps API Projects
```

> **Authentication establishes who the automation is. Authorization defines what it can do. Error handling determines how safely it behaves when the distributed system fails.**
