# Security

## 1. Introduction

Production Python automation frequently has access to highly privileged systems:

```text
AWS
EKS
Kubernetes
Git
Jenkins
GitHub Actions
ArgoCD
Terraform
Databases
Container registries
```

That makes Python scripts a security boundary.

A secure DevOps automation design must protect:

```text
credentials
secrets
source code
infrastructure
Kubernetes resources
CI/CD pipelines
artifacts
logs
configuration
```

Core principle:

```text
Least privilege
+
short-lived credentials
+
secret isolation
+
input validation
+
safe subprocess execution
+
secure dependencies
+
secure logging
+
environment guards
=
production-safe automation
```

---

# 2. Security Threat Model

Before writing automation, identify:

```text
What can the script access?
What can it modify?
What secrets does it receive?
Who can trigger it?
What happens if input is malicious?
What happens if credentials are compromised?
What happens if the script targets the wrong environment?
```

For DevOps automation, assume:

```text
inputs can be malicious
dependencies can contain vulnerabilities
credentials can leak
external APIs can fail
CI variables can be misconfigured
operators can make mistakes
```

---

# 3. Security Objectives

A production automation system should provide:

```text
Confidentiality
Integrity
Availability
Authentication
Authorization
Auditability
Non-repudiation where required
```

For example:

```text
Confidentiality -> don't expose AWS credentials
Integrity       -> don't deploy unauthorized code
Availability    -> don't create uncontrolled retry storms
Authentication  -> verify identity
Authorization   -> enforce permissions
Auditability    -> record who/what/when safely
```

---

# 4. Never Hardcode Secrets

Never:

```python
AWS_ACCESS_KEY = "AKIA..."
AWS_SECRET_KEY = "secret"
DB_PASSWORD = "password123"
```

This creates risks through:

```text
Git history
code review
backups
CI artifacts
developer machines
logs
```

---

# 5. Environment Variables

Environment variables can be useful for configuration:

```python
import os

api_url = os.environ["API_URL"]
```

For secrets, environment variables are better than hardcoding but are not automatically secure.

They can be exposed through:

```text
process inspection
debug output
CI diagnostics
crash dumps
misconfigured logs
```

Use a dedicated secret manager when possible.

---

# 6. Secret Management Architecture

Preferred production pattern:

```text
Python
  |
  v
Secret Manager
  |
  +--> AWS Secrets Manager
  +--> AWS Systems Manager Parameter Store
  +--> Kubernetes Secret integration
  +--> approved external secret manager
```

The application retrieves the secret at runtime rather than storing it in source code.

---

# 7. AWS Secrets Manager

Typical architecture:

```text
Python
  |
  v
IAM role
  |
  v
Secrets Manager
  |
  v
secret value
```

Permissions should be limited to the exact secret:

```text
secretsmanager:GetSecretValue
```

Avoid:

```text
secretsmanager:*
```

unless there is a specific justified requirement.

---

# 8. Parameter Store

AWS Systems Manager Parameter Store can store configuration and, where appropriate, encrypted parameters.

Architecture:

```text
Python
  |
  v
IAM role
  |
  v
SSM Parameter Store
```

Use least-privilege access to specific parameter paths.

---

# 9. Secret Manager vs Configuration

Separate:

```text
configuration
```

from:

```text
secrets
```

Example:

```text
API_URL           -> configuration
AWS_REGION        -> configuration
LOG_LEVEL         -> configuration

DB_PASSWORD       -> secret
API_TOKEN         -> secret
PRIVATE_KEY       -> secret
```

Do not put everything into one generic configuration file.

---

# 10. AWS IAM Roles

For AWS-hosted workloads, prefer:

```text
IAM role
```

over static access keys.

Examples:

```text
EC2 instance profile
EKS Pod Identity / IRSA depending on platform design
GitHub OIDC
Jenkins AWS role assumption
```

This reduces long-lived credential exposure.

---

# 11. GitHub Actions OIDC

Preferred architecture:

```text
GitHub Actions
      |
      | OIDC token
      v
AWS IAM
      |
      v
AssumeRole
      |
      v
temporary AWS credentials
```

This is safer than storing long-lived AWS access keys in repository secrets.

---

# 12. Jenkins AWS Authentication

A Jenkins pipeline can use controlled AWS credentials or assume a role.

Preferred model:

```text
Jenkins
   |
   v
approved identity
   |
   v
assume IAM role
   |
   v
temporary permissions
```

Do not give every Jenkins job administrator-level AWS credentials.

---

# 13. EKS Pod Identity

For workloads running inside EKS, use the AWS-supported pod identity mechanism appropriate to the cluster design.

Concept:

```text
Kubernetes service account
        |
        v
AWS identity mapping
        |
        v
IAM role
        |
        v
AWS API
```

The pod receives only the permissions required by its workload.

---

# 14. Least Privilege

A Python deployment tool might need:

```text
eks:DescribeCluster
```

but not:

```text
*
```

A tool that updates one S3 bucket should not automatically have:

```text
s3:*
on all resources
```

Least privilege limits blast radius.

---

# 15. Resource-Level IAM Permissions

Prefer:

```text
Action:
  s3:GetObject

Resource:
  arn:aws:s3:::company-artifacts/prod/*
```

over broad permissions:

```text
Action:
  s3:*
Resource:
  *
```

Use resource-level controls where supported.

---

# 16. Separate Read and Write Roles

A powerful production pattern:

```text
read-only automation
        |
        v
read IAM role

deployment automation
        |
        v
write IAM role
```

Diagnostics should not require production mutation privileges.

---

# 17. Separate Environments

Avoid using one credential set for:

```text
dev
staging
production
```

Prefer:

```text
dev role
staging role
production role
```

with controlled access.

---

# 18. Environment Guard

Before any production mutation:

```python
if environment != "production":
    ...
```

More importantly, verify multiple independent identifiers.

Example:

```text
expected account
expected region
expected cluster
expected namespace
expected environment
```

---

# 19. AWS Account Guard

Before production changes:

```python
EXPECTED_ACCOUNT = "123456789012"

if actual_account != EXPECTED_ACCOUNT:
    raise RuntimeError(
        "AWS account mismatch; refusing operation"
    )
```

Never assume the configured AWS profile points to the expected account.

---

# 20. AWS Region Guard

Verify:

```text
expected region
actual region
```

before destructive operations.

A correct resource name in the wrong region can still be dangerous.

---

# 21. EKS Cluster Guard

Before Kubernetes mutation:

```text
expected cluster = production-eks
actual cluster   = ?
```

The script should explicitly verify the target.

---

# 22. Kubernetes Namespace Guard

Before changing resources:

```text
expected namespace = production
actual namespace   = ?
```

Never rely solely on the current kubeconfig context.

---

# 23. Kubernetes Context Guard

Check:

```bash
kubectl config current-context
```

or use the Kubernetes API client to establish the intended target explicitly.

A safer automation flow is:

```text
authenticate
 ↓
identify cluster
 ↓
verify cluster
 ↓
verify namespace
 ↓
perform mutation
```

---

# 24. Production Safety Gate

A strong design:

```text
target selected
      |
      v
AWS account verified?
      |
      v
region verified?
      |
      v
EKS cluster verified?
      |
      v
namespace verified?
      |
      v
Git SHA verified?
      |
      v
approval/policy satisfied?
      |
      v
mutation allowed
```

If any safety check fails:

```text
STOP
```

---

# 25. Fail Closed

Security checks should fail closed.

Bad:

```python
if not verify_cluster():
    logger.warning("Could not verify cluster")
    continue
```

Better:

```python
if not verify_cluster():
    raise SecurityError(
        "Unable to verify target cluster"
    )
```

Unknown security state should not become permission to continue.

---

# 26. Input Validation

Never trust:

```text
CLI arguments
environment variables
API input
Git branch names
file paths
resource names
user input
CI variables
```

Validate them before use.

---

# 27. Allowlist Over Blocklist

Weak:

```python
if environment != "evil":
    continue
```

Strong:

```python
ALLOWED_ENVIRONMENTS = {
    "dev",
    "staging",
    "production",
}

if environment not in ALLOWED_ENVIRONMENTS:
    raise ValueError("Invalid environment")
```

Allow only known valid values.

---

# 28. Validate Environment

Example:

```python
import re

if not re.fullmatch(
    r"[a-z0-9-]+",
    environment,
):
    raise ValueError("Invalid environment")
```

Validation rules should match the actual allowed naming convention.

---

# 29. Validate Resource Names

Kubernetes resource names have naming constraints.

Do not blindly pass arbitrary input.

Prefer:

```text
DNS-compatible validation
length limits
allowed characters
```

and use the Kubernetes client's structured APIs where possible.

---

# 30. Path Traversal

Dangerous:

```python
path = "/tmp/" + user_input
open(path)
```

An attacker might provide:

```text
../../etc/passwd
```

Use:

```text
allowlisted filenames
safe base directories
path resolution
ownership checks
```

---

# 31. Safe Path Handling

Example:

```python
from pathlib import Path

base = Path("/tmp/automation").resolve()
candidate = (base / filename).resolve()

if base not in candidate.parents:
    raise SecurityError("Path traversal detected")
```

Still validate the filename itself when possible.

---

# 32. Temporary Files

Use secure temporary file APIs:

```python
import tempfile

with tempfile.NamedTemporaryFile() as f:
    ...
```

Avoid predictable temporary filenames such as:

```text
/tmp/deployment-secret.txt
```

---

# 33. Temporary Secret Files

If a secret must temporarily exist as a file:

```text
restrict permissions
use secure temporary storage
delete promptly
never log path/content unnecessarily
```

Prefer passing secrets through secure APIs instead of files when possible.

---

# 34. Subprocess Security

Dangerous:

```python
subprocess.run(
    f"kubectl delete pod {pod_name}",
    shell=True,
)
```

If `pod_name` is attacker-controlled, command injection may occur.

---

# 35. Safe Subprocess Arguments

Prefer:

```python
subprocess.run(
    [
        "kubectl",
        "delete",
        "pod",
        pod_name,
    ],
    check=True,
)
```

Arguments are passed separately.

---

# 36. Avoid `shell=True`

Prefer:

```python
subprocess.run(
    ["terraform", "plan"],
    check=True,
)
```

over:

```python
subprocess.run(
    "terraform plan",
    shell=True,
)
```

`shlex.quote()` can help in limited shell-invocation cases, but avoiding the shell is generally safer.

---

# 37. Command Allowlisting

If a Python tool executes commands based on user input:

```text
kubectl
helm
terraform
git
```

allowlist the executable and validate each argument.

Never allow arbitrary command strings.

---

# 38. Executable Path

For security-sensitive automation, consider invoking trusted executables by controlled paths or validating the execution environment.

Avoid accidentally executing an attacker-controlled binary earlier in `PATH`.

---

# 39. Git Security

Git automation can be dangerous.

Validate:

```text
repository
branch
remote
commit
working tree
```

before pushing production configuration.

---

# 40. Avoid Blind Force Push

Do not automatically execute:

```bash
git push --force
```

on production configuration.

A stale local branch can overwrite another engineer's changes.

---

# 41. Git Integrity

For production deployment:

```text
commit SHA
repository
branch
review/approval
```

should be known.

If the workflow expects a specific commit:

```text
verify SHA
```

before deployment.

---

# 42. GitHub Actions Security

Protect:

```text
repository secrets
environment secrets
deployment environments
workflow permissions
pull request execution
```

Use minimum required `GITHUB_TOKEN` permissions.

---

# 43. Untrusted Pull Requests

Be especially careful when workflows execute code from pull requests.

A pull request can modify:

```text
workflow YAML
Python scripts
shell scripts
Terraform
Dockerfiles
```

Do not expose production secrets or deployment credentials to untrusted code.

---

# 44. CI Secret Exposure

Avoid:

```yaml
run: echo "${{ secrets.API_TOKEN }}"
```

Even accidental debug output can expose credentials.

Use secret masking provided by the CI platform, but do not rely on masking as a substitute for safe logging.

---

# 45. Fork Security

Repository forks may be controlled by untrusted users.

Be careful with:

```text
pull_request
pull_request_target
```

and any workflow that checks out attacker-controlled code while possessing privileged credentials.

---

# 46. Jenkins Security

Restrict:

```text
job permissions
credential access
agent permissions
script execution
production deployment
```

Avoid giving every Jenkins job access to every credential.

---

# 47. Jenkins Credential Isolation

Prefer:

```text
job A -> credential A
job B -> credential B
```

instead of:

```text
all jobs -> production credentials
```

---

# 48. Kubernetes RBAC

A Python service account should receive only the permissions it requires.

Example conceptual RBAC:

```yaml
rules:
  - apiGroups: ["apps"]
    resources: ["deployments"]
    verbs: ["get", "list", "watch", "patch"]
```

Avoid:

```yaml
verbs: ["*"]
resources: ["*"]
```

---

# 49. Namespace-Scoped RBAC

Prefer a namespace-scoped Role/RoleBinding where possible:

```text
automation service
        |
        v
Role
        |
        v
production namespace
```

instead of cluster-wide privileges.

---

# 50. ClusterRole Risk

Cluster-wide permissions can affect:

```text
all namespaces
nodes
RBAC
secrets
cluster configuration
```

Use ClusterRole only when the automation genuinely needs cluster-wide access.

---

# 51. Kubernetes Secret Permissions

A Python service that only deploys Deployments should not automatically have:

```text
secrets/get
secrets/list
```

unless required.

Secret access is highly sensitive.

---

# 52. Kubernetes API Security

Use TLS verification.

Do not disable certificate verification merely to solve a connection problem.

Avoid patterns equivalent to:

```python
verify_ssl=False
```

in production.

---

# 53. TLS

Secure communication should use:

```text
HTTPS
TLS
certificate validation
trusted CA
```

Do not replace certificate validation with:

```text
ignore certificate errors
```

---

# 54. Certificate Validation

Bad:

```python
requests.get(
    url,
    verify=False,
)
```

This creates man-in-the-middle risk.

Correct:

```python
requests.get(
    url,
    verify=True,
    timeout=10,
)
```

or configure a trusted CA bundle when required.

---

# 55. API Authentication

Use appropriate mechanisms:

```text
OAuth2
OIDC
AWS SigV4
service account tokens
short-lived tokens
mTLS where required
```

Do not invent custom authentication schemes when a supported standard exists.

---

# 56. Token Lifetime

Prefer:

```text
short-lived credentials
```

over:

```text
permanent credentials
```

Short-lived credentials reduce the impact of accidental exposure.

---

# 57. Token Storage

Do not store tokens in:

```text
source code
Docker image
Git repository
plain-text config
logs
artifacts
```

Use:

```text
secret manager
CI secret store
runtime identity
```

---

# 58. Docker Image Security

Never bake secrets into:

```dockerfile
ENV AWS_SECRET_ACCESS_KEY=...
```

or:

```dockerfile
COPY .env /app/.env
```

Secrets in an image can remain in image layers and registries.

---

# 59. Build-Time Secrets

If build tools require credentials:

```text
use secure build secret mechanisms
```

rather than:

```text
ARG TOKEN=...
RUN command --token "$TOKEN"
```

because build arguments can leak into metadata/history depending on the build setup.

---

# 60. Python Dependency Security

Python applications depend on third-party packages.

Risks include:

```text
known CVEs
malicious packages
dependency confusion
typosquatting
supply-chain compromise
```

---

# 61. Pin Dependencies

Use controlled versions:

```text
requests==<approved-version>
```

or a lock/constraint strategy appropriate to the project.

Avoid uncontrolled production dependency drift.

---

# 62. Dependency Scanning

Use security tooling in CI:

```text
pip-audit
OSV-based tooling
SCA platform
approved enterprise scanner
```

Integrate the result into the DevSecOps pipeline.

---

# 63. Dependency Update Process

Do not blindly update everything in production.

Preferred:

```text
dependency update
 ↓
security scan
 ↓
unit tests
 ↓
integration tests
 ↓
build
 ↓
staging
 ↓
production
```

---

# 64. Requirements Files

A controlled project may use:

```text
requirements.in
requirements.txt
constraints.txt
```

or a modern Python dependency manager with lock files.

The important property is:

```text
reproducible dependency resolution
```

---

# 65. Virtual Environments

Use isolated environments:

```bash
python -m venv .venv
```

Then:

```bash
source .venv/bin/activate
```

This prevents unrelated system packages from affecting the application.

---

# 66. Production Dependency Isolation

Containers provide another isolation boundary:

```text
Python application
+
pinned dependencies
+
minimal base image
```

Do not install unnecessary packages into production images.

---

# 67. Minimal Container Image

A production Python image should contain only:

```text
runtime
application
required dependencies
CA certificates
required OS libraries
```

Avoid:

```text
debug tools
compilers
unused packages
credentials
source repositories
```

unless genuinely required.

---

# 68. Non-Root Containers

Prefer running Python as a non-root user.

Example:

```dockerfile
RUN useradd --create-home appuser
USER appuser
```

Exact implementation depends on the base image.

---

# 69. Why Non-Root Matters

If the application is compromised:

```text
root container
```

can increase impact.

A non-root process reduces available privileges.

It is not a replacement for other security controls.

---

# 70. Read-Only Filesystem

Where possible:

```text
read-only root filesystem
```

reduces an attacker's ability to modify the container.

If temporary writes are required:

```text
explicit writable volume/tmpfs
```

can be provided.

---

# 71. Container Capabilities

Avoid unnecessary Linux capabilities.

Use a restricted security context where appropriate.

The goal is:

```text
minimum process privileges
```

---

# 72. Kubernetes Security Context

Example conceptual configuration:

```yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
```

Additional controls may include:

```text
readOnlyRootFilesystem
capabilities.drop
seccompProfile
```

according to the workload.

---

# 73. Kubernetes Service Account

Do not automatically mount a service account token when the workload does not need Kubernetes API access.

Where supported:

```yaml
automountServiceAccountToken: false
```

For workloads that do need API access, use a dedicated service account with least privilege.

---

# 74. Network Security

A Python automation service should communicate only with required endpoints.

Use:

```text
security groups
network policies
private endpoints
VPC controls
egress restrictions
```

where appropriate.

---

# 75. Kubernetes NetworkPolicy

Example intent:

```text
automation pod
  |
  +--> Kubernetes API
  +--> required AWS/private endpoint
  |
  X--> arbitrary internet
```

Restrict unnecessary network access.

---

# 76. SSRF

Server-Side Request Forgery is relevant when Python accepts URLs and fetches them.

Dangerous:

```python
requests.get(user_supplied_url)
```

An attacker may attempt to access:

```text
internal services
metadata endpoints
private APIs
```

---

# 77. SSRF Mitigation

Use:

```text
URL allowlist
scheme allowlist
host allowlist
DNS/IP validation
egress restrictions
redirect controls
```

Never assume a URL is safe simply because it starts with `https://`.

---

# 78. AWS Metadata Endpoint

Do not allow arbitrary user-controlled requests to access instance metadata.

Use modern AWS metadata protections and avoid exposing metadata services to untrusted workloads.

---

# 79. Command Injection

Example dangerous input:

```text
service = "orders; rm -rf /"
```

If inserted into:

```python
os.system(...)
```

the result can be catastrophic.

Use:

```text
structured subprocess arguments
allowlists
validation
```

---

# 80. SQL Injection

If Python interacts with databases, never build SQL with string concatenation.

Bad:

```python
query = (
    "SELECT * FROM users "
    f"WHERE username = '{username}'"
)
```

Use parameterized queries:

```python
cursor.execute(
    "SELECT * FROM users WHERE username = %s",
    (username,),
)
```

The exact placeholder syntax depends on the database driver.

---

# 81. Secrets in SQL Logs

Database queries may contain sensitive values.

Avoid logging:

```text
full SQL + parameters
```

when parameters contain credentials or personal data.

---

# 82. YAML Security

Be careful with unsafe YAML deserialization.

Avoid unsafe loaders for untrusted YAML.

Prefer safe parsing:

```python
yaml.safe_load(data)
```

when using PyYAML and when the data does not intentionally require custom Python objects.

---

# 83. Pickle Security

Never unpickle untrusted data.

`pickle` can execute arbitrary code during deserialization.

Bad:

```python
pickle.loads(untrusted_data)
```

Prefer safer formats:

```text
JSON
validated YAML
approved serialization formats
```

---

# 84. JSON Security

JSON is generally safer than arbitrary object serialization, but still validate:

```text
types
required fields
maximum sizes
allowed values
nested structure
```

Do not assume valid JSON means valid application input.

---

# 85. Regular Expression Security

Poorly designed regex can cause catastrophic backtracking.

This can become a denial-of-service problem.

For untrusted input:

```text
use simple patterns
limit input size
avoid pathological nested quantifiers
```

---

# 86. Input Size Limits

Limit:

```text
request body
file size
manifest size
log payload
API response
CLI input
```

Large untrusted input can consume:

```text
CPU
memory
disk
network
```

---

# 87. YAML/JSON Manifest Validation

Before deploying Kubernetes manifests:

```text
parse
validate schema
validate allowed resources
validate namespace
validate image policy
validate security fields
```

Then:

```text
deploy
```

---

# 88. Image Security

Python automation should not blindly deploy arbitrary images.

Validate:

```text
registry
repository
tag/digest
signature policy
vulnerability policy
```

where required.

---

# 89. Prefer Image Digests

Tags can move:

```text
orders:latest
```

is mutable.

A digest identifies content:

```text
orders@sha256:<digest>
```

For controlled production deployment, immutable references are safer.

---

# 90. Container Image Scanning

Integrate:

```text
Trivy
SCA/image scanner
approved enterprise scanner
```

into CI/CD.

A production Python deployment tool can verify scan results before deployment.

---

# 91. DevSecOps Pipeline

A strong pipeline:

```text
Git
 |
 v
Build
 |
 v
Unit Tests
 |
 v
SAST
 |
 v
SCA
 |
 v
Image Build
 |
 v
Trivy/Image Scan
 |
 v
Quality/Security Gates
 |
 v
Artifact Registry
 |
 v
Deploy
 |
 v
Verify
```

The Python automation can orchestrate or validate these stages.

---

# 92. Artifact Integrity

Production automation should deploy approved artifacts.

Verify:

```text
repository
version
digest
checksum
signature where supported
```

Do not silently download and execute arbitrary artifacts.

---

# 93. Supply Chain Security

Protect the chain:

```text
source
 ↓
dependencies
 ↓
build
 ↓
artifact
 ↓
registry
 ↓
deployment
```

A secure Python script is not enough if the artifact supply chain is compromised.

---

# 94. Dependency Confusion

If an internal package is:

```text
internal-devops-tool
```

an attacker could publish a public package with the same name.

Use:

```text
trusted package indexes
private repositories
dependency pinning
package provenance
```

---

# 95. JFrog Artifactory

If Artifactory is used:

```text
Python
  |
  v
approved repository
  |
  v
Artifactory
```

Use repository controls and credentials through the approved secret mechanism.

Do not embed Artifactory credentials in `requirements.txt`.

---

# 96. Secure Package Installation

Avoid arbitrary:

```bash
pip install package-from-random-url
```

in production.

Prefer approved internal/external indexes and controlled versions.

---

# 97. Secrets in Git History

Deleting a secret from the current file does not remove it from Git history.

If a secret was committed:

```text
revoke/rotate immediately
```

then clean history according to the organization's incident process.

Do not assume history rewriting alone invalidates a credential.

---

# 98. `.gitignore`

Use `.gitignore` for local secret/config files:

```text
.env
*.pem
*.key
credentials*
```

But remember:

```text
.gitignore is prevention, not secret management.
```

A developer can still force-add a file.

---

# 99. Pre-Commit Secret Scanning

Use secret scanning tools in development/CI.

Examples:

```text
Gitleaks
GitHub secret scanning
approved enterprise scanners
```

Block credentials before they reach the repository.

---

# 100. Log Secret Scanning

CI should also scan generated artifacts/logs where appropriate.

A build may accidentally expose:

```text
environment variables
stack traces
command arguments
configuration
```

---

# 101. Artifact Security

Do not publish artifacts containing:

```text
.env
credentials
private keys
CI workspace secrets
Kubeconfigs
```

Review:

```text
build output
test reports
debug bundles
logs
```

before publishing.

---

# 102. Kubeconfig Security

A kubeconfig can contain credentials or token references.

Do not:

```text
commit kubeconfig
upload kubeconfig as artifact
log kubeconfig
```

Use short-lived or workload-specific authentication.

---

# 103. SSH Key Security

Do not store private keys in source repositories.

If automation needs SSH:

```text
secret manager
CI credential store
agent-based authentication
short-lived credentials
```

where possible.

---

# 104. File Permissions

Sensitive local files should have restrictive permissions.

For example:

```bash
chmod 600 sensitive-file
```

Directories containing private credentials may require:

```bash
chmod 700 directory
```

Exact permissions should follow the application and OS requirements.

---

# 105. Process Security

Command-line arguments can sometimes be visible through process inspection.

Avoid passing secrets like:

```bash
python deploy.py --token SECRET
```

when a safer secret injection mechanism exists.

---

# 106. Environment Variable Exposure

Environment variables can sometimes be inspected by privileged users/processes.

Use them carefully for secrets and prefer runtime secret-manager retrieval when practical.

---

# 107. Memory Handling

Python does not provide guaranteed secure memory wiping for ordinary strings.

Therefore:

```text
avoid unnecessary secret copies
avoid logging secrets
keep secret lifetime short
use dedicated secret mechanisms
```

Do not claim that simply doing:

```python
secret = None
```

guarantees secure memory erasure.

---

# 108. Error Handling and Security

Do not expose sensitive details in user-facing errors.

Bad:

```text
DB connection failed:
postgres://admin:password@internal-db...
```

Better:

```text
Database connection failed.
```

Detailed safe diagnostics can be logged separately.

---

# 109. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
What are you allowed to do?
```

Example:

```text
AWS identity = deployment-role
permission   = update EKS deployment
```

Both controls are required.

---

# 110. RBAC Review

Periodically review:

```text
IAM policies
Kubernetes Roles
RoleBindings
service accounts
CI credentials
repository permissions
```

Remove unused access.

---

# 111. Credential Rotation

Use a rotation process:

```text
credential created
 ↓
used
 ↓
rotated
 ↓
old credential revoked
```

Avoid credentials that never expire.

---

# 112. Secret Rotation Architecture

For application secrets:

```text
Secret Manager
       |
       v
new secret
       |
       v
application refresh/redeploy
       |
       v
old secret revoked
```

Design applications so rotation can happen without emergency code changes.

---

# 113. Production Access

Production mutation should require stronger controls than development.

Examples:

```text
environment approval
protected branch
IAM role restriction
CI deployment environment
manual approval
change record
```

The exact controls depend on organizational policy.

---

# 114. Separation of Duties

Avoid one identity controlling:

```text
source approval
build
security approval
production deployment
```

without independent controls where organizational policy requires separation.

---

# 115. Python Script as a Privileged Tool

If a Python script can:

```text
delete AWS resources
modify Kubernetes RBAC
deploy production
change Terraform
```

treat it like privileged infrastructure software.

Apply:

```text
code review
testing
dependency scanning
access controls
audit logging
release management
```

---

# 116. Secure CLI Design

A production CLI should:

```text
validate arguments
show target environment
require explicit destructive flags
verify context
avoid secrets in arguments
return correct exit codes
```

For dangerous operations, consider explicit confirmation or policy-controlled approval.

---

# 117. Destructive Operation Guard

Example:

```python
if action == "delete" and environment == "production":
    if not approved:
        raise SecurityError(
            "Production deletion is not approved"
        )
```

Better still, enforce authorization outside the Python code too.

---

# 118. Defense in Depth

Do not rely on one check.

For production deployment:

```text
Git branch protection
+
CI environment protection
+
IAM permissions
+
Python target validation
+
Kubernetes RBAC
+
admission/security policy
```

Multiple independent controls reduce risk.

---

# 119. Admission Controls

Kubernetes admission policies can enforce:

```text
allowed registries
non-root containers
required labels
resource limits
security context
image policy
```

Python automation should work with these controls rather than bypass them.

---

# 120. Do Not Bypass Security Gates

Never make automation automatically bypass:

```text
Trivy failure
SonarQube quality gate
RBAC denial
admission rejection
image policy
approval requirement
```

A security control that can be bypassed by the deployment script is not a reliable control.

---

# 121. Security Event Logging

Record security-relevant events:

```text
authentication failure
authorization failure
target mismatch
production deployment
security gate failure
credential refresh
policy rejection
```

Do not log the secret itself.

---

# 122. Security Metrics

Potential metrics:

```text
authentication_failures_total
authorization_failures_total
security_gate_failures_total
deployment_rejections_total
```

Avoid labels containing:

```text
token
username if unnecessarily high-cardinality
secret values
request IDs
```

---

# 123. Security Alerts

Alert on meaningful patterns:

```text
repeated authentication failures
unexpected production deployment
large increase in authorization failures
security gate bypass attempts
unexpected target environment
```

---

# 124. Dependency Vulnerability Response

When a vulnerability is discovered:

```text
identify affected dependency
 ↓
assess exposure
 ↓
update/patch
 ↓
run tests
 ↓
scan
 ↓
build
 ↓
deploy
 ↓
verify
```

Do not leave vulnerable dependencies indefinitely because "the script is internal."

---

# 125. Python Security Tooling

Useful categories:

```text
SAST
SCA
secret scanning
dependency auditing
container scanning
IaC scanning
linting
```

Examples that may be used depending on organizational standards:

```text
Bandit
pip-audit
Gitleaks
Trivy
SonarQube
```

---

# 126. Bandit

Bandit can identify common Python security issues.

Example:

```bash
bandit -r .
```

Typical findings may include:

```text
unsafe subprocess usage
weak cryptography
dangerous functions
```

Review findings rather than blindly suppressing them.

---

# 127. pip-audit

Example:

```bash
pip-audit
```

It checks Python dependencies for known vulnerabilities according to its configured vulnerability data sources.

Integrate it into CI.

---

# 128. Gitleaks

Use secret scanning to identify credentials accidentally committed to Git.

Example:

```bash
gitleaks detect
```

Exact CLI options depend on the installed version.

---

# 129. Trivy

Trivy can scan:

```text
container images
filesystem
IaC
configuration
```

A DevSecOps pipeline can use it as a security gate.

---

# 130. SonarQube

SonarQube can provide:

```text
code quality
security findings
bugs
code smells
coverage-related analysis
```

Python automation should fail the pipeline when the defined quality/security gate is not satisfied.

---

# 131. Secure CI Pipeline

Recommended flow:

```text
Checkout
   |
   v
Dependency validation
   |
   v
Secret scan
   |
   v
SAST
   |
   v
Unit tests
   |
   v
Dependency/SCA scan
   |
   v
Build
   |
   v
Image scan
   |
   v
Security/quality gates
   |
   v
Artifact publish
   |
   v
Deployment
   |
   v
Verification
```

---

# 132. Security Gates

Security gates should be deterministic.

Examples:

```text
critical vulnerability -> BLOCK
secret detected        -> BLOCK
quality gate failed    -> BLOCK
unsigned artifact      -> BLOCK if policy requires
wrong target           -> BLOCK
```

Do not turn security failures into retries.

---

# 133. Secure Deployment Verification

After deployment verify:

```text
expected image digest
expected namespace
expected replicas
expected readiness
expected service
expected ingress
expected health
```

This prevents silent deployment to the wrong state.

---

# 134. Supply Chain Verification

Before production:

```text
source verified
 ↓
dependencies verified
 ↓
artifact built
 ↓
artifact scanned
 ↓
artifact approved
 ↓
artifact identity verified
 ↓
deployment
```

---

# 135. Secure Artifact Promotion

Prefer:

```text
build once
scan once
promote the same artifact
```

rather than rebuilding separately for every environment.

This preserves artifact identity.

---

# 136. Environment Promotion

Example:

```text
artifact v1
   |
   v
dev
   |
   v
staging
   |
   v
security/approval
   |
   v
production
```

The Python automation should deploy the approved artifact rather than silently rebuilding it.

---

# 137. Production Secret Flow

```text
CI identity
    |
    v
IAM/OIDC
    |
    v
Secret Manager
    |
    v
Python runtime
    |
    v
API call
    |
    v
secret leaves memory as soon as practical
```

Never:

```text
Git -> secret -> Python source
```

---

# 138. Secure Kubernetes Flow

```text
Python
  |
  v
dedicated ServiceAccount
  |
  v
RBAC
  |
  v
Kubernetes API
  |
  v
namespace-scoped resources
```

For AWS access:

```text
Pod
 |
 v
AWS pod identity
 |
 v
IAM role
 |
 v
AWS API
```

---

# 139. Production Security Architecture

```text
                         Git
                          |
                  branch protection
                          |
                          v
                   CI/CD Pipeline
                          |
          +---------------+---------------+
          |               |               |
          v               v               v
       SAST/SCA       Secret Scan      Image Scan
          |               |               |
          +---------------+---------------+
                          |
                          v
                    Security Gates
                          |
                          v
                  Python Automation
                          |
          +---------------+---------------+
          |               |               |
          v               v               v
        AWS             EKS            ArgoCD
          |               |               |
       IAM role          RBAC       GitOps policy
          |               |               |
          +---------------+---------------+
                          |
                          v
                      Production
```

---

# 140. Threat-to-Control Mapping

| Threat | Primary controls |
|---|---|
| Hardcoded secret | Secret manager, secret scanning |
| Credential theft | Short-lived credentials, IAM |
| Excessive AWS permissions | Least-privilege IAM |
| Excessive Kubernetes access | RBAC |
| Command injection | Argument lists, validation |
| SQL injection | Parameterized queries |
| SSRF | URL allowlists, egress controls |
| Path traversal | Safe path resolution |
| Dependency vulnerability | SCA/pip-audit |
| Malicious package | Trusted repositories, pinning |
| Secret in Git | Secret scanning, rotation |
| Secret in logs | Safe logging |
| Wrong production target | Account/cluster/environment guards |
| Container compromise | Non-root, restricted security context |
| Unauthorized deployment | CI/environment controls |
| Image compromise | Scanning, digest/signature policy |
| CI credential exposure | OIDC/short-lived credentials |
| Git overwrite | Branch protection, no blind force-push |

---

# 141. Security Testing

Test security controls themselves.

Examples:

```text
wrong AWS account
wrong region
wrong cluster
wrong namespace
invalid environment
malicious filename
command injection input
invalid URL
expired credential
403 response
missing secret
secret accidentally logged
unapproved image
security scan failure
```

Expected behavior should be:

```text
safe failure
no unauthorized mutation
clear audit/log event
non-zero exit
```

---

# 142. Security Unit Tests

Example:

```python
def test_wrong_environment_is_rejected():
    with pytest.raises(SecurityError):
        validate_target(
            environment="production",
            expected_environment="staging",
        )
```

Security validation should be tested like business logic.

---

# 143. Security Integration Tests

Test actual:

```text
IAM permissions
Kubernetes RBAC
secret manager access
CI identity
TLS
artifact registry
```

where the test environment permits it.

---

# 144. Negative Testing

Security testing must include negative paths.

Examples:

```text
no credentials
expired token
invalid token
insufficient IAM permission
wrong namespace
malformed manifest
untrusted image
blocked registry
```

The automation should fail safely.

---

# 145. Permission Testing

Verify that the service can:

```text
GET required resource -> SUCCESS
PATCH required resource -> SUCCESS
```

but cannot:

```text
DELETE unrelated resource -> DENIED
READ unrelated secret -> DENIED
MODIFY RBAC -> DENIED
```

This demonstrates least privilege.

---

# 146. Security Regression Testing

Whenever permissions change:

```text
run security tests
```

Whenever dependencies change:

```text
run vulnerability scan
```

Whenever deployment scope changes:

```text
run environment guard tests
```

---

# 147. Security Review Checklist

```text
[ ] No hardcoded secrets
[ ] Secret manager integration
[ ] Short-lived credentials
[ ] Least-privilege IAM
[ ] Least-privilege Kubernetes RBAC
[ ] Environment isolation
[ ] Account guard
[ ] Region guard
[ ] Cluster guard
[ ] Namespace guard
[ ] Input validation
[ ] Safe subprocess execution
[ ] No unnecessary shell=True
[ ] TLS verification
[ ] SSRF protection
[ ] Path traversal protection
[ ] SQL parameterization
[ ] Safe YAML loading
[ ] No unsafe pickle
[ ] Dependency pinning/locking
[ ] SCA
[ ] SAST
[ ] Secret scanning
[ ] Image scanning
[ ] Secure container
[ ] Non-root execution
[ ] Security context
[ ] Network restrictions
[ ] Secret-safe logging
[ ] Audit events
[ ] Security alerts
[ ] Negative tests
```

---

# 148. Production Security Checklist

Before production deployment:

```text
Identity
[ ] Correct AWS role
[ ] Correct Kubernetes service account
[ ] No static credentials where avoidable

Authorization
[ ] Least privilege
[ ] Namespace scope
[ ] Production role protected

Secrets
[ ] Runtime retrieval
[ ] No Git secrets
[ ] No image secrets
[ ] No log secrets
[ ] Rotation process

Code
[ ] Input validation
[ ] Safe subprocess
[ ] Secure deserialization
[ ] Dependency scanning

Infrastructure
[ ] Correct AWS account
[ ] Correct region
[ ] Correct EKS cluster
[ ] Correct namespace

Supply chain
[ ] Trusted source
[ ] Approved dependencies
[ ] Scanned image
[ ] Approved artifact
[ ] Immutable artifact identity

Runtime
[ ] Non-root
[ ] Restricted capabilities
[ ] Network controls
[ ] TLS verification

Observability
[ ] Security events logged
[ ] Metrics available
[ ] Alerts configured
```

---

# 149. Common Security Anti-Patterns

Avoid:

```text
hardcoded AWS keys
admin IAM policy for automation
cluster-admin for every Python script
production credentials in developer laptops
secrets in Dockerfiles
secrets in Git
tokens in CLI arguments
logging Authorization headers
verify=False
shell=True with untrusted input
os.system(user_input)
pickle.loads(untrusted_data)
yaml.load with unsafe loader
unvalidated URLs
blind force-push
automatic security-gate bypass
latest image in production
unscanned dependencies
running container as root unnecessarily
```

---

# 150. Senior Interview — How Do You Secure Python DevOps Automation?

Strong answer:

> I start with least privilege and short-lived identity. For AWS I prefer IAM roles and OIDC-based temporary credentials rather than long-lived keys. For EKS I use dedicated service accounts and least-privilege RBAC. Secrets are retrieved from an approved secret manager and never committed or logged. I validate all inputs, avoid unsafe shell execution, verify TLS certificates, scan dependencies and images, and enforce environment/account/cluster guards before production mutations.

---

# 151. Senior Interview — How Do You Handle AWS Credentials?

Strong answer:

> I avoid hardcoded access keys. In AWS-hosted workloads I use IAM roles, and for CI systems I prefer OIDC or controlled role assumption with temporary credentials. Permissions are scoped to the exact APIs and resources required by the automation.

---

# 152. Senior Interview — How Do You Secure EKS Python Automation?

Strong answer:

> I use a dedicated Kubernetes service account with least-privilege RBAC, avoid cluster-admin unless absolutely necessary, verify the target cluster and namespace before mutations, use secure TLS communication, and restrict AWS permissions through the workload's AWS identity. I also avoid reading Kubernetes Secrets unless the automation genuinely needs them.

---

# 153. Senior Interview — What Is Least Privilege?

Strong answer:

> Least privilege means giving an identity only the permissions required to perform its task. For example, a deployment automation may need to read and patch Deployments in one namespace but should not automatically be able to delete resources across the cluster or modify RBAC.

---

# 154. Senior Interview — Why Is `shell=True` Dangerous?

Strong answer:

> If untrusted input is incorporated into a shell command, it can lead to command injection. I prefer `subprocess.run()` with a list of arguments and `shell=False`. I also validate or allowlist resource names and other command arguments.

---

# 155. Senior Interview — How Do You Prevent Secret Leakage?

Strong answer:

> I prevent secrets from entering source control, images and logs. I use a secret manager or CI secret store, prefer short-lived credentials, avoid passing secrets in command-line arguments, and scan repositories and artifacts for accidental exposure. If a credential is exposed, I rotate or revoke it immediately.

---

# 156. Senior Interview — How Do You Secure CI/CD?

Strong answer:

> I use protected environments and branches, least-privilege CI tokens, short-lived cloud credentials such as OIDC, security scanning, secret scanning and explicit deployment approvals where required. Untrusted pull-request code should never receive production credentials.

---

# 157. Senior Interview — How Do You Protect Against Wrong-Environment Deployment?

Strong answer:

> I do not trust only the environment variable. Before a production mutation I verify the expected AWS account, region, EKS cluster, namespace and deployment target. If any identity does not match the expected target, the automation fails closed.

---

# 158. Senior Interview — What Is Defense in Depth?

Strong answer:

> Defense in depth means using multiple independent controls. For production deployment I might have branch protection, CI environment protection, IAM restrictions, Python target validation, Kubernetes RBAC and admission policies. If one control fails, the others still reduce the blast radius.

---

# 159. Senior Interview — How Do You Secure Python Dependencies?

Strong answer:

> I use reproducible dependency versions or lock files, trusted package repositories, dependency vulnerability scanning such as pip-audit or an enterprise SCA tool, and regular controlled updates. Dependency changes go through tests and security gates before production.

---

# 160. Senior Interview — How Do You Secure Containers Running Python?

Strong answer:

> I use a minimal base image, remove unnecessary packages, avoid secrets in image layers, run as a non-root user, disable unnecessary privilege escalation and capabilities, use a read-only filesystem where practical, scan the image, and apply Kubernetes security controls.

---

# 161. Senior Interview — What If a Secret Is Accidentally Committed?

Strong answer:

> I treat the credential as compromised immediately. I revoke or rotate it first, then investigate where it may have propagated. After that I remove it from source/history according to the organization's process and add secret-scanning controls to prevent recurrence. Removing the current file alone is not enough.

---

# 162. Senior Interview — Why Are Short-Lived Credentials Better?

Strong answer:

> Their exposure window is smaller. If a temporary credential is accidentally exposed, it expires sooner than a permanent access key. Combined with least privilege and audit controls, this significantly reduces the blast radius.

---

# 163. Senior Interview — How Do You Secure a Production Delete Operation?

Strong answer:

> I require explicit target validation, least-privilege authorization and an approved workflow. I verify the AWS account, region, EKS cluster and namespace where relevant. For highly destructive actions I use additional approval or policy controls. The operation should fail closed if the target cannot be verified.

---

# 164. Senior Interview — How Do You Handle Security Gate Failures?

Strong answer:

> Security gate failures are deterministic policy failures, not transient errors. For example, if Trivy detects a blocking vulnerability or SonarQube's security gate fails, I stop the pipeline rather than retrying. The issue must be remediated or explicitly handled through the organization's approved exception process.

---

# 165. Senior Interview — How Do You Secure GitOps?

Strong answer:

> I protect the Git repository and production branches, use least-privilege credentials, avoid blind force-pushes, verify the intended commit and environment, and let ArgoCD reconcile the approved Git state. I also ensure the deployment identity cannot bypass repository or Kubernetes security controls.

---

# 166. Senior Interview — What Is SSRF?

Strong answer:

> SSRF occurs when a server-side application fetches attacker-controlled URLs and can therefore be abused to reach internal resources. In Python automation, I avoid arbitrary URL fetching, use host and scheme allowlists, validate resolved addresses where necessary, and enforce network egress controls.

---

# 167. Senior Interview — Why Is `verify=False` Dangerous?

Strong answer:

> It disables TLS certificate verification, allowing an attacker in the network path to impersonate the server. I keep certificate verification enabled and configure the correct CA bundle when an internal certificate authority is used.

---

# 168. Senior Interview — How Do You Secure Kubernetes RBAC?

Strong answer:

> I create a dedicated service account and grant only the verbs and resources required. I prefer namespace-scoped Roles over cluster-wide ClusterRoles. I also explicitly test denied operations such as reading unrelated Secrets or modifying RBAC.

---

# 169. Senior Interview — How Do You Secure Production Python Scripts?

Strong answer:

> I treat them as privileged infrastructure software. I use code review, reproducible dependencies, SAST/SCA/secret scanning, least-privilege identity, secure secret management, strict input validation, safe subprocess execution, production target guards, secure logging, auditability and negative security testing.

---

# 170. Real-World Scenario — Wrong AWS Account

```text
Python automation starts
        |
        v
read current AWS identity
        |
        v
account != expected production account
        |
        v
SECURITY ERROR
        |
        v
STOP
```

Correct behavior:

```text
no Terraform
no EKS mutation
no S3 deletion
no deployment
```

---

# 171. Real-World Scenario — Wrong EKS Cluster

```text
expected = prod-eks
actual   = dev-eks
```

Correct:

```text
reject
log safe security event
exit non-zero
```

Never:

```text
continue with warning
```

---

# 172. Real-World Scenario — Expired AWS Credential

```text
AWS request
   |
   v
ExpiredToken
   |
   v
credential refresh/role refresh
   |
   v
retry safe operation
```

If refresh is unavailable:

```text
fail clearly
```

Do not print the token or credential contents.

---

# 173. Real-World Scenario — Kubernetes RBAC Denial

```text
Python
  |
  v
Kubernetes API
  |
  v
403 Forbidden
  |
  v
AuthorizationError
  |
  v
FAIL
```

Do not retry repeatedly because the same RBAC policy will continue to deny the request.

---

# 174. Real-World Scenario — Secret Manager Failure

```text
Python
  |
  v
Secrets Manager
  |
  v
timeout
  |
  v
bounded retry
  |
  v
success
```

If retries are exhausted:

```text
FAIL
```

Never fall back to:

```text
hardcoded password
```

---

# 175. Real-World Scenario — Security Scanner Failure

```text
Trivy
  |
  v
critical vulnerability
  |
  v
security gate
  |
  v
BLOCK
```

Correct:

```text
no deployment
```

Incorrect:

```text
retry Trivy until it passes
```

---

# 176. Real-World Scenario — Malicious Input

Input:

```text
orders; rm -rf /
```

If used with:

```python
shell=True
```

it may become a command injection.

Safe architecture:

```text
validate
  |
  v
allowlist
  |
  v
structured subprocess arguments
```

---

# 177. Real-World Scenario — Secret in Git

```text
AWS key committed
      |
      v
secret scanner
      |
      v
BLOCK CI
      |
      v
rotate credential
      |
      v
remove secret
      |
      v
investigate exposure
```

The scanner is a prevention control, not a substitute for credential rotation.

---

# 178. Real-World Scenario — Untrusted PR

```text
Pull Request
    |
    v
workflow executes changed Python code
    |
    +--> production credentials?
    |
   YES
    |
    v
SECURITY RISK
```

Correct design:

```text
untrusted PR
  |
  v
limited permissions
  |
  v
tests/scans only
```

Production credentials should be available only to trusted deployment workflows.

---

# 179. Real-World Scenario — Container Compromise

```text
Python container compromised
        |
        v
non-root user
        |
        v
limited capabilities
        |
        v
restricted service account
        |
        v
restricted IAM role
        |
        v
network controls
```

Defense in depth reduces the blast radius.

---

# 180. Security Architecture for the User's DevOps Stack

```text
                         GitHub
                           |
                  branch/environment policy
                           |
                           v
                 Jenkins / GitHub Actions
                           |
        +------------------+------------------+
        |                  |                  |
        v                  v                  v
     SonarQube          Trivy            Secret Scan
        |                  |                  |
        +------------------+------------------+
                           |
                           v
                    Python Automation
                           |
              +------------+------------+
              |                         |
              v                         v
           AWS IAM                  Kubernetes RBAC
              |                         |
              v                         v
       AWS / EKS APIs              EKS Resources
              |
              v
       Secrets Manager
              |
              v
        Runtime Secrets
```

---

# 181. Security Golden Rules

```text
1. Never hardcode credentials.
2. Prefer short-lived identities.
3. Use least-privilege IAM.
4. Use least-privilege Kubernetes RBAC.
5. Separate environments.
6. Verify AWS account and region.
7. Verify EKS cluster and namespace.
8. Fail closed on security uncertainty.
9. Validate all external input.
10. Avoid shell=True for untrusted input.
11. Never disable TLS verification to solve a shortcut.
12. Never log secrets.
13. Do not put secrets in Docker images.
14. Scan dependencies.
15. Scan container images.
16. Scan repositories for secrets.
17. Protect CI credentials.
18. Protect production Git branches.
19. Use immutable/verified artifacts where possible.
20. Treat privileged Python automation as production infrastructure software.
```

---

# 182. Final Security Model

```text
                 UNTRUSTED INPUT
                       |
                       v
                 VALIDATION
                       |
                       v
                  AUTHENTICATE
                       |
                       v
                  AUTHORIZE
                       |
                       v
               TARGET VERIFICATION
                       |
             +---------+---------+
             |                   |
          SAFE                 UNSAFE
             |                   |
             v                   v
          EXECUTE               STOP
             |
             v
          VERIFY
             |
             v
          AUDIT
             |
             v
        OBSERVABILITY
```

Production security is not one feature.

It is a chain:

```text
Identity
   +
Least privilege
   +
Secrets management
   +
Input validation
   +
Secure execution
   +
Supply-chain security
   +
Environment guards
   +
Runtime controls
   +
Auditability
```

The DevOps mindset is:

```text
Do not ask:
"Does the script work?"

Ask:
"Who can run it?"
"What can it access?"
"What can it modify?"
"What happens if credentials leak?"
"What happens if input is malicious?"
"What happens if it targets production incorrectly?"
"Can the blast radius be contained?"
"Can we prove what happened?"
```

---

# 183. Section Progress

```text
10-Python-Production/
│
├── 01-Production-Scripting.md        ✓
├── 02-Error-Handling-and-Retry.md    ✓
├── 03-Logging-and-Observability.md   ✓
├── 04-Security.md                    ✓
├── 05-Performance.md
├── 06-Concurrency.md
├── 07-Configuration-Management.md
└── 08-Production-Best-Practices.md
```

Next:

```text
05-Performance.md
```
