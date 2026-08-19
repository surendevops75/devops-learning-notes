# 10-Python-Production
# 07 — Configuration Management

## 1. Introduction

Production Python automation should not hard-code environment-specific values.

Configuration management separates:

```text
application logic
        +
environment-specific configuration
        +
secrets
        +
runtime overrides
```

A production DevOps script may need configuration for:

```text
AWS region
AWS account
EKS cluster
Kubernetes namespace
repository
branch
deployment timeout
API endpoint
retry count
log level
worker count
feature flags
environment
```

The goal is:

```text
same code
+
different configuration
=
reusable automation
```

---

# 2. Configuration vs Secrets

Configuration:

```text
AWS_REGION=ap-south-1
LOG_LEVEL=INFO
TIMEOUT=30
WORKERS=10
```

Secrets:

```text
API_TOKEN
DATABASE_PASSWORD
PRIVATE_KEY
ACCESS_TOKEN
```

Secrets require stronger handling.

Never treat secrets like ordinary configuration.

---

# 3. Configuration Sources

Common sources:

```text
default values
configuration files
environment variables
CLI arguments
Kubernetes ConfigMap
Kubernetes Secret
AWS Systems Manager Parameter Store
AWS Secrets Manager
CI/CD variables
secret managers
```

Define a clear precedence model.

---

# 4. Configuration Precedence

Example:

```text
defaults
   ↓
config file
   ↓
environment variables
   ↓
CLI arguments
```

Higher-priority values override lower-priority values.

Document the precedence.

---

# 5. Why Precedence Matters

Suppose:

```text
config.yaml -> production
ENVIRONMENT -> staging
```

If precedence is unclear, the automation may deploy to the wrong environment.

Configuration behavior must be deterministic.

---

# 6. Twelve-Factor Configuration

A common principle is:

```text
configuration belongs outside application code
```

Environment-specific values should not require modifying Python source code.

This improves:

```text
portability
deployment safety
testing
maintainability
```

---

# 7. Hard-Coded Configuration

Avoid:

```python
AWS_REGION = "ap-south-1"
CLUSTER = "production-cluster"
```

inside application logic when those values vary by environment.

Prefer configuration injection.

---

# 8. Environment Variables

Example:

```python
import os

region = os.getenv(
    "AWS_REGION",
    "ap-south-1",
)
```

Environment variables are useful for:

```text
deployment environment
runtime flags
URLs
timeouts
non-secret operational settings
```

---

# 9. Required Environment Variables

For mandatory configuration:

```python
import os

cluster = os.environ["EKS_CLUSTER"]
```

If missing:

```text
KeyError
```

The application fails immediately.

Failing fast is often preferable for critical configuration.

---

# 10. Explicit Validation

A better production pattern:

```python
import os

cluster = os.getenv("EKS_CLUSTER")

if not cluster:
    raise ValueError(
        "EKS_CLUSTER is required"
    )
```

Provide a clear error message.

---

# 11. Configuration Schema

Treat configuration as structured data.

Example:

```text
environment
aws
kubernetes
deployment
logging
retry
concurrency
```

This makes configuration easier to validate and reason about.

---

# 12. Dataclass Configuration

Python dataclasses are useful:

```python
from dataclasses import dataclass

@dataclass
class Config:
    environment: str
    region: str
    namespace: str
    timeout: int
    workers: int
```

Application code can then receive:

```python
config
```

instead of reading environment variables everywhere.

---

# 13. Centralized Configuration

Prefer:

```text
main
 |
 v
load_config()
 |
 v
validate_config()
 |
 v
application
```

Avoid:

```text
function A -> reads ENV
function B -> reads ENV
function C -> reads ENV
```

Centralization improves consistency.

---

# 14. Configuration Loading Layer

Architecture:

```text
Environment/File/CLI
        |
        v
Configuration Loader
        |
        v
Validation
        |
        v
Typed Config Object
        |
        v
Application
```

This is a strong production pattern.

---

# 15. Typed Configuration

Instead of passing raw strings everywhere:

```python
workers = "10"
```

convert to:

```python
workers = 10
```

during configuration loading.

The rest of the application should use the correct type.

---

# 16. Boolean Configuration

Environment variables are strings.

```text
ENABLE_DEPLOYMENT=false
```

is the string:

```python
"false"
```

Do not use:

```python
bool(os.getenv("ENABLE_DEPLOYMENT"))
```

because:

```python
bool("false")
```

is `True`.

---

# 17. Safe Boolean Parser

Example:

```python
def parse_bool(value: str) -> bool:
    normalized = value.strip().lower()

    if normalized in {"true", "1", "yes"}:
        return True

    if normalized in {"false", "0", "no"}:
        return False

    raise ValueError(
        f"Invalid boolean: {value}"
    )
```

---

# 18. Integer Configuration

Example:

```python
workers = int(
    os.getenv("WORKERS", "10")
)
```

Validate the range:

```python
if not 1 <= workers <= 100:
    raise ValueError(
        "WORKERS must be between 1 and 100"
    )
```

---

# 19. Timeout Configuration

Example:

```python
timeout = int(
    os.getenv("TIMEOUT_SECONDS", "30")
)
```

Validate:

```text
timeout > 0
```

Avoid:

```text
timeout = 0
```

unless explicitly supported.

---

# 20. Configuration Validation

Validate:

```text
required fields
types
ranges
allowed values
relationships
environment consistency
```

before performing mutations.

---

# 21. Fail Fast

Bad:

```text
load invalid config
   |
perform AWS calls
   |
create resources
   |
discover invalid value
```

Better:

```text
load config
   |
validate
   |
only then mutate infrastructure
```

---

# 22. Environment Validation

Example:

```python
if config.environment == "production":
    require_production_guard()
```

Production operations should have stronger validation.

---

# 23. Account Validation

For AWS automation:

```text
expected account
        |
        v
current account
        |
        v
compare
```

Never assume the active AWS identity is correct.

---

# 24. Region Validation

Validate:

```text
expected region
vs
configured region
```

Example:

```text
expected = ap-south-1
actual   = us-east-1
```

Fail before performing destructive operations.

---

# 25. Cluster Validation

For EKS automation:

```text
expected cluster
vs
actual cluster
```

A strong guard prevents:

```text
staging script
    |
    v
production cluster
```

accidental execution.

---

# 26. Namespace Configuration

Example:

```text
environment = production
namespace = production
```

Validate that the namespace matches expected deployment rules.

---

# 27. Configuration Cross-Validation

Configuration fields can depend on each other.

Example:

```text
environment=production
namespace=dev
```

may be invalid.

Validation should check relationships, not only individual fields.

---

# 28. Configuration File Formats

Common formats:

```text
YAML
JSON
TOML
INI
.env
```

For modern application configuration:

```text
YAML/TOML
```

can provide readable structured configuration.

---

# 29. YAML Configuration

Example:

```yaml
environment: production

aws:
  region: ap-south-1

kubernetes:
  cluster: production-eks
  namespace: production

deployment:
  timeout: 300
  workers: 10
```

---

# 30. Safe YAML Parsing

Use:

```python
yaml.safe_load(data)
```

Avoid unsafe object deserialization.

Configuration files should not execute arbitrary Python objects.

---

# 31. JSON Configuration

Example:

```json
{
  "environment": "production",
  "aws": {
    "region": "ap-south-1"
  }
}
```

JSON is useful when interoperability with other tools is important.

---

# 32. TOML Configuration

TOML is commonly used for Python project/tool configuration.

Example:

```toml
environment = "production"
workers = 10
timeout = 300
```

It can be convenient for human-managed configuration.

---

# 33. Configuration File Location

Do not assume:

```text
current working directory
```

is always the same.

Use:

```text
explicit path
environment variable
CLI argument
application config directory
```

as appropriate.

---

# 34. Relative Path Risk

This can fail:

```python
open("config.yaml")
```

when the script is executed from another directory.

Production automation should resolve configuration paths deliberately.

---

# 35. Path Resolution

Use `pathlib`:

```python
from pathlib import Path

config_path = Path(
    "/etc/myapp/config.yaml"
)
```

or construct paths from a known application/config root.

---

# 36. Default Configuration

Defaults should be safe.

Good:

```text
workers = 5
timeout = 30
log_level = INFO
```

Risky:

```text
environment = production
auto_apply = true
```

for a generic automation tool.

---

# 37. Safe Defaults

For destructive automation:

```text
auto_apply = false
dry_run = true
```

may be safer defaults, depending on the workflow.

---

# 38. Explicit Production Mode

Avoid relying on:

```text
hostname
current directory
developer machine
```

to determine environment.

Prefer:

```text
ENVIRONMENT=production
```

plus account/cluster validation.

---

# 39. Dry-Run Configuration

Example:

```yaml
deployment:
  dry_run: true
```

The application should clearly indicate:

```text
DRY RUN
```

and avoid mutation.

---

# 40. Auto-Apply Guard

A production automation tool can require:

```text
AUTO_APPLY=true
```

plus:

```text
environment validation
account validation
cluster validation
```

before mutation.

---

# 41. Configuration Immutability

After loading:

```text
configuration
```

should ideally be treated as immutable.

Avoid random functions modifying global configuration.

This makes concurrent execution safer.

---

# 42. Global Configuration Anti-Pattern

Avoid:

```python
CONFIG = {}

def function_a():
    CONFIG["region"] = "..."

def function_b():
    CONFIG["region"] = "..."
```

Concurrent operations can produce inconsistent behavior.

---

# 43. Dependency Injection

Prefer:

```python
def deploy(config, client):
    ...
```

instead of:

```python
def deploy():
    read_environment()
    create_client()
```

Dependency injection improves:

```text
testing
reusability
clarity
concurrency
```

---

# 44. Configuration Object Injection

Example:

```python
config = load_config()

deploy(
    config=config,
    client=client,
)
```

This makes dependencies explicit.

---

# 45. CLI Arguments

Use `argparse` for simple command-line tools.

Example:

```python
import argparse

parser = argparse.ArgumentParser()

parser.add_argument(
    "--environment",
    required=True,
)

args = parser.parse_args()
```

---

# 46. CLI vs Environment Variables

CLI is useful for:

```text
one-off overrides
operator commands
interactive execution
```

Environment variables are useful for:

```text
CI/CD
containers
Kubernetes
runtime configuration
```

Use a documented precedence model.

---

# 47. CLI Override

Example:

```bash
python deploy.py \
  --environment staging
```

could override:

```text
ENVIRONMENT
```

if the application's precedence says CLI has higher priority.

---

# 48. Configuration Profiles

A tool may support:

```text
development
staging
production
```

But avoid duplicating the entire configuration for every environment.

Use:

```text
shared defaults
+
environment-specific overrides
```

---

# 49. Environment-Specific Files

Example:

```text
config/
├── base.yaml
├── dev.yaml
├── staging.yaml
└── production.yaml
```

This can work well if secrets remain outside these files.

---

# 50. Configuration Layering

Example:

```text
base.yaml
    +
production.yaml
    +
environment variables
    +
CLI
    =
final configuration
```

Make merge rules deterministic.

---

# 51. Configuration Drift

Configuration can drift when:

```text
production file
CI variable
Kubernetes ConfigMap
local script
```

contain different values.

Use a clear source of truth.

---

# 52. Git as Configuration Source

For non-secret configuration:

```text
Git
  |
  v
review
  |
  v
merge
  |
  v
deployment
```

This provides:

```text
history
review
rollback
auditability
```

---

# 53. GitOps Configuration

In GitOps:

```text
Git
 |
 v
desired configuration
 |
 v
ArgoCD
 |
 v
Kubernetes
```

Python automation should generally modify the desired state rather than bypassing it.

---

# 54. Configuration and Secrets in GitOps

Store:

```text
non-secret configuration -> Git
```

For secrets use:

```text
AWS Secrets Manager
SSM Parameter Store
Kubernetes Secret with appropriate protection
external secret mechanism
```

Do not commit plaintext production secrets.

---

# 55. Kubernetes ConfigMap

ConfigMaps store non-sensitive configuration.

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "INFO"
  REGION: "ap-south-1"
```

---

# 56. Kubernetes Secret

Secrets are intended for sensitive values.

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
stringData:
  API_TOKEN: example
```

Do not assume base64 encoding itself provides encryption.

---

# 57. Secret Encryption

Kubernetes secret protection depends on cluster configuration and storage controls.

For sensitive production environments, use:

```text
encryption at rest
RBAC
KMS integration
external secret management
```

where appropriate.

---

# 58. AWS Systems Manager Parameter Store

Useful for:

```text
configuration parameters
non-secret values
secure strings
environment-specific settings
```

Example hierarchy:

```text
/myapp/prod/aws/region
/myapp/prod/deployment/timeout
```

---

# 59. AWS Secrets Manager

Use for:

```text
database credentials
API credentials
tokens
rotatable secrets
```

It provides secret storage and lifecycle capabilities.

---

# 60. Parameter Store vs Secrets Manager

Parameter Store:

```text
configuration
parameters
SecureString
hierarchical paths
```

Secrets Manager:

```text
secrets
rotation
secret lifecycle
credential-oriented workloads
```

Choose based on requirements.

---

# 61. Secret Retrieval

A Python application can retrieve a secret at runtime using an appropriate SDK/client.

Avoid:

```text
hard-code secret
write secret to logs
put secret in exception message
store plaintext secret in source
```

---

# 62. Secret Caching

Caching a secret can reduce repeated API calls.

But consider:

```text
rotation
revocation
TTL
memory exposure
```

Use a short and intentional cache lifetime when needed.

---

# 63. Configuration vs Secret Rotation

Configuration may change:

```text
monthly
```

Secret credentials may rotate:

```text
daily
hourly
automatically
```

The application should support the expected lifecycle.

---

# 64. Runtime Configuration

Containers should receive configuration through:

```text
environment
mounted config
ConfigMap
Secret
external secret mechanism
```

rather than rebuilding the image for every environment.

---

# 65. Immutable Images

Prefer:

```text
same image
+
different runtime configuration
```

instead of:

```text
development image
staging image
production image
```

with environment-specific values baked into the image.

---

# 66. Docker ENV

Docker can provide:

```dockerfile
ENV LOG_LEVEL=INFO
```

But avoid baking secrets into image layers.

Secrets included during image build can remain recoverable from image history/layers.

---

# 67. Kubernetes Environment Injection

Example:

```yaml
env:
  - name: LOG_LEVEL
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: LOG_LEVEL
```

For secrets:

```yaml
env:
  - name: API_TOKEN
    valueFrom:
      secretKeyRef:
        name: app-secret
        key: API_TOKEN
```

---

# 68. Mounted Configuration

Configuration can also be mounted as files.

Useful for:

```text
large configuration
certificate bundles
templates
application config
```

---

# 69. Configuration Reload

Some long-running services can reload configuration without restart.

Possible approaches:

```text
SIGHUP
file watcher
dynamic configuration service
API
```

But reload behavior must be explicitly designed and tested.

---

# 70. Immutable Configuration vs Reload

Immutable configuration:

```text
simple
predictable
restart required
```

Dynamic reload:

```text
no restart
more complexity
race conditions
partial update risks
```

For many DevOps tools, immutable startup configuration is safer.

---

# 71. Configuration Validation Libraries

Options include:

```text
dataclasses
pydantic
attrs
custom validation
```

A schema library can provide:

```text
type validation
default values
constraints
clear errors
```

---

# 72. Pydantic

Example concept:

```python
from pydantic import BaseModel

class Config(BaseModel):
    environment: str
    workers: int = 10
    timeout: int = 30
```

Pydantic can validate input into structured models.

---

# 73. Configuration Schema

A schema can enforce:

```text
workers >= 1
timeout > 0
environment in allowed values
region format
required cluster
```

This moves errors toward startup instead of runtime.

---

# 74. Configuration Versioning

Configuration structure can evolve.

Example:

```yaml
version: 2
```

This can help when migrating between schema versions.

---

# 75. Backward Compatibility

If configuration changes from:

```text
timeout
```

to:

```text
deployment.timeout
```

provide a controlled migration strategy rather than silently ignoring the old setting.

---

# 76. Configuration Deprecation

When removing a field:

```text
old_config
```

should ideally produce:

```text
deprecation warning
```

before being removed.

---

# 77. Unknown Configuration Keys

Consider rejecting unexpected keys:

```text
workers
timeout
typo_timeout
```

If `typo_timeout` is silently ignored, the application may use an unsafe default.

Strict validation can prevent this class of error.

---

# 78. Configuration Type Coercion

Be careful with automatic conversions.

Example:

```text
"false"
```

should not accidentally become:

```text
True
```

as discussed earlier.

Explicit conversion is safer.

---

# 79. Configuration Error Messages

Good:

```text
Invalid DEPLOYMENT_TIMEOUT:
expected integer > 0, received "abc"
```

Bad:

```text
invalid config
```

Errors should tell operators how to fix the problem.

---

# 80. Secret-Safe Errors

Never include:

```text
API_TOKEN=...
password=...
secret=...
```

in validation errors.

Use:

```text
API_TOKEN is missing
```

not:

```text
API_TOKEN value abc123 is invalid
```

---

# 81. Configuration Logging

Log safe configuration metadata:

```text
environment=production
region=ap-south-1
workers=10
timeout=300
```

Do not log:

```text
password
token
private key
secret
```

---

# 82. Redaction

If configuration objects may contain sensitive values, implement redaction.

Example:

```text
API_TOKEN=****
DB_PASSWORD=****
```

rather than exposing actual values.

---

# 83. Debug Mode

Debug configuration can expose sensitive information.

Avoid:

```text
dump entire config object
```

in production logs.

Use explicit safe fields.

---

# 84. Environment Variable Leakage

Environment variables may be visible through:

```text
process inspection
debug output
crash reports
CI logs
```

Do not put sensitive values in environment variables unless the runtime/security model is appropriate.

---

# 85. CI/CD Variables

CI/CD systems often provide:

```text
repository variables
environment variables
secrets
```

Use secret-specific storage for credentials.

Ensure masking is enabled where supported.

---

# 86. Jenkins Configuration

Use:

```text
Jenkins Credentials
```

rather than hard-coded secrets.

Inject credentials only into the scope where needed.

---

# 87. GitHub Actions Configuration

Use:

```text
GitHub Secrets
Environment Secrets
Variables
```

and appropriate environment protection rules.

Never echo secrets into workflow logs.

---

# 88. GitLab CI/CD Configuration

Use:

```text
CI/CD Variables
masked variables
protected variables
```

according to environment/security requirements.

---

# 89. AWS IAM Role Configuration

Prefer:

```text
IAM role
```

over:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

embedded in configuration files.

For EKS workloads, use the appropriate workload identity mechanism.

---

# 90. Configuration and OIDC

For CI/CD:

```text
GitHub Actions
      |
      v
OIDC token
      |
      v
AWS IAM role
      |
      v
temporary credentials
```

This avoids long-lived static AWS credentials.

---

# 91. Configuration and EKS Pod Identity

For workloads running in EKS:

```text
Pod
 |
 v
AWS workload identity
 |
 v
IAM role
 |
 v
AWS APIs
```

This is preferable to embedding AWS credentials in configuration.

---

# 92. Configuration Hierarchy Example

```text
base defaults
      |
      v
environment file
      |
      v
ConfigMap/parameter
      |
      v
environment variable
      |
      v
CLI override
      |
      v
validated Config object
```

Secrets are resolved separately.

---

# 93. Configuration Ownership

Define who owns each setting.

Example:

```text
application team -> app timeout
platform team -> cluster endpoint
security team -> credential policy
SRE/DevOps -> deployment policy
```

Ownership prevents conflicting changes.

---

# 94. Configuration Review

Production configuration should be reviewed like code.

Check:

```text
correct environment
safe defaults
resource limits
timeouts
retry limits
security
cost impact
```

---

# 95. Configuration Change Management

A safe change process:

```text
edit
 ↓
validate
 ↓
review
 ↓
merge
 ↓
deploy
 ↓
verify
```

Avoid manual production edits without auditability when possible.

---

# 96. Configuration Rollback

Git-based configuration makes rollback easier:

```text
current version
      |
      v
bad configuration
      |
      v
revert commit
      |
      v
deploy previous version
```

---

# 97. Configuration Drift Detection

Compare:

```text
desired configuration
vs
actual runtime configuration
```

For Kubernetes/GitOps:

```text
Git desired state
vs
cluster state
```

ArgoCD can detect drift for managed resources.

---

# 98. Python and Configuration Drift

A Python automation tool should avoid silently modifying configuration outside the intended source of truth.

If GitOps is authoritative:

```text
Python -> Git
```

rather than:

```text
Python -> direct cluster mutation
```

for managed configuration.

---

# 99. Configuration in Terraform

Terraform variables can come from:

```text
.tfvars
environment variables
CLI
Terraform Cloud/CI variables
```

Python orchestration should pass values explicitly and safely.

---

# 100. Terraform Variable Validation

Use Terraform-side validation for infrastructure-specific constraints.

Python can validate orchestration inputs, while Terraform validates infrastructure variables.

Defense in depth is useful.

---

# 101. Helm Values

Helm commonly uses:

```text
values.yaml
values-production.yaml
--set
--set-file
```

Python automation can generate or select values, but should avoid creating uncontrolled configuration divergence.

---

# 102. Kubernetes Configuration Architecture

```text
Git
 |
 +--> ConfigMap
 |
 +--> Secret reference
 |
 v
ArgoCD
 |
 v
EKS
 |
 v
Pod
 |
 +--> environment variables
 +--> mounted files
```

---

# 103. Configuration Validation Before Deployment

Pipeline:

```text
load config
    |
    v
schema validation
    |
    v
environment/account/cluster validation
    |
    v
render manifests
    |
    v
policy/security checks
    |
    v
deploy
```

Fail before mutation whenever possible.

---

# 104. Configuration and Policy

Configuration determines:

```text
what the application does
```

Policy determines:

```text
what the application is allowed to do
```

Do not let configuration override security policy.

---

# 105. Configuration Security Boundary

Example:

```text
WORKERS=1000
```

should not be allowed to bypass a platform maximum:

```text
max_workers=50
```

The application should enforce safe upper bounds.

---

# 106. Configuration Limits

Define limits for:

```text
workers
timeouts
batch sizes
queue sizes
retry counts
payload sizes
concurrency
```

This prevents dangerous configuration mistakes.

---

# 107. Retry Configuration

Example:

```yaml
retry:
  max_attempts: 3
  base_delay: 1
  max_delay: 30
  jitter: true
```

Validate:

```text
max_attempts > 0
base_delay >= 0
max_delay >= base_delay
```

---

# 108. Concurrency Configuration

Example:

```yaml
concurrency:
  aws_workers: 10
  kubernetes_workers: 10
  git_workers: 3
```

Keep independent resource pools configurable.

---

# 109. Logging Configuration

Example:

```yaml
logging:
  level: INFO
  format: json
```

Validate allowed levels:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

---

# 110. Environment Configuration

Example:

```yaml
environment:
  name: production
  account_id: "123456789012"
  region: ap-south-1
```

Account ID should be validated against the actual AWS identity.

---

# 111. Kubernetes Configuration

Example:

```yaml
kubernetes:
  cluster: production-eks
  namespace: production
  context: production
```

Avoid trusting only the context name.

Verify actual cluster identity where possible.

---

# 112. AWS Configuration

Example:

```yaml
aws:
  region: ap-south-1
  role_arn: ...
```

Prefer runtime identity mechanisms over static credentials.

---

# 113. API Configuration

Example:

```yaml
api:
  base_url: https://api.example.internal
  connect_timeout: 5
  read_timeout: 30
```

Validate URLs and timeout ranges.

---

# 114. URL Validation

Avoid accepting arbitrary URLs for privileged automation without validation.

Consider:

```text
scheme
host
allowed domains
TLS requirement
```

This also reduces SSRF risk.

---

# 115. Configuration and SSRF

If a configuration field controls:

```text
URL to fetch
```

an attacker-controlled value may point to:

```text
localhost
metadata endpoint
internal service
```

Validate allowed destinations for privileged workloads.

---

# 116. Configuration and Command Injection

Do not directly construct:

```python
command = f"kubectl apply -f {config.path}"
```

from untrusted configuration.

Prefer:

```python
subprocess.run(
    ["kubectl", "apply", "-f", config.path],
    check=True,
)
```

and validate the path.

---

# 117. Configuration and File Paths

Configuration can contain paths:

```text
terraform_dir
manifest_dir
output_file
```

Validate:

```text
exists
expected type
allowed root
```

where security matters.

---

# 118. Configuration and Path Traversal

Avoid accepting:

```text
../../etc/passwd
```

as a trusted configuration path.

Resolve and validate paths against an allowed root.

---

# 119. Configuration and Certificates

Certificate paths should be validated.

Prefer:

```text
trusted CA bundle
TLS verification enabled
```

Do not disable TLS verification simply because configuration is inconvenient.

---

# 120. Configuration and Proxy Settings

Environment variables such as:

```text
HTTP_PROXY
HTTPS_PROXY
NO_PROXY
```

can affect network behavior.

Production automation should explicitly understand whether proxy configuration is expected.

---

# 121. Configuration and NO_PROXY

For Kubernetes/AWS workloads, incorrect `NO_PROXY` settings can cause:

```text
unexpected proxy routing
API failures
latency
security issues
```

Test network behavior in the target environment.

---

# 122. Configuration and DNS

Do not hard-code dynamic IP addresses when service discovery is available.

Prefer:

```text
DNS
Kubernetes Service
Route53
service discovery
```

where appropriate.

---

# 123. Configuration and Feature Flags

Feature flags can control:

```text
new deployment strategy
new API
experimental automation
```

Production flags need:

```text
owner
default
expiry/review date
rollback behavior
```

---

# 124. Stale Feature Flags

Flags that remain indefinitely become configuration debt.

Review and remove obsolete flags.

---

# 125. Configuration and Compatibility

When supporting multiple versions:

```text
API v1
API v2
```

configuration may select behavior.

Keep compatibility logic explicit.

---

# 126. Configuration Testing

Test:

```text
valid config
missing config
invalid type
invalid range
unknown key
conflicting values
production guard
secret missing
CLI override
environment override
```

---

# 127. Unit Testing Configuration

Example:

```python
def test_invalid_workers():
    with pytest.raises(ValueError):
        load_config({
            "workers": 0
        })
```

Configuration validation should be tested independently.

---

# 128. Integration Testing Configuration

Test against:

```text
AWS test account
test Kubernetes cluster
staging environment
mock secret manager
```

Validate that configuration reaches the actual clients correctly.

---

# 129. Configuration Contract Tests

A configuration contract can define:

```text
required fields
types
allowed values
default behavior
```

This helps prevent incompatible changes between automation components.

---

# 130. Configuration Documentation

Document:

```text
variable
description
type
default
required?
allowed values
secret?
example
```

Example:

| Variable | Type | Required | Default | Secret |
|---|---|---:|---|---|
| `ENVIRONMENT` | string | yes | — | no |
| `AWS_REGION` | string | yes | — | no |
| `WORKERS` | integer | no | 10 | no |
| `API_TOKEN` | string | yes | — | yes |

---

# 131. Configuration Naming

Use consistent names:

```text
AWS_REGION
EKS_CLUSTER
K8S_NAMESPACE
DEPLOYMENT_TIMEOUT
MAX_WORKERS
LOG_LEVEL
```

Avoid ambiguous names:

```text
TIME
NAME
VALUE
```

---

# 132. Namespacing Configuration

For large systems:

```text
AWS_REGION
K8S_NAMESPACE
DEPLOY_TIMEOUT
RETRY_MAX_ATTEMPTS
```

or structured YAML sections:

```text
aws.region
kubernetes.namespace
deployment.timeout
retry.max_attempts
```

This prevents collisions.

---

# 133. Configuration Secrets in Memory

Even if secrets are not logged, they may exist in process memory.

Reduce exposure by:

```text
retrieve only when needed
avoid unnecessary copies
limit lifetime
do not put in global objects
```

Python cannot provide perfect memory erasure guarantees, so focus on reducing exposure.

---

# 134. Configuration in Crash Dumps

Crash dumps/debugging systems may capture process memory.

Sensitive applications should consider organizational controls around:

```text
core dumps
debug dumps
heap dumps
error reporting
```

---

# 135. Configuration and Process Arguments

Do not pass secrets through CLI arguments:

```bash
python deploy.py --token SECRET
```

Process arguments can be exposed through system process inspection.

Prefer secure secret injection mechanisms.

---

# 136. Configuration and Shell History

Avoid commands like:

```bash
export API_TOKEN=secret
```

in shared terminals/history when possible.

Use secure secret stores and CI/CD secret injection.

---

# 137. Configuration and `.env`

`.env` files are convenient for local development.

Do not treat:

```text
.env
```

as a production secret-management solution.

Ensure secrets are not committed to Git.

---

# 138. `.gitignore`

A local secret file can be excluded:

```gitignore
.env
*.secret
```

But `.gitignore` does not remove a secret that was already committed.

Rotate exposed credentials.

---

# 139. Secret Scanning

Use security tools such as:

```text
Gitleaks
Trivy
SonarQube
GitHub secret scanning
```

according to the pipeline design.

---

# 140. Configuration and Supply Chain

Do not blindly load configuration from:

```text
untrusted repository
untrusted URL
untrusted package
```

Configuration can influence:

```text
commands
URLs
deployment targets
credentials
```

Treat configuration as potentially privileged input.

---

# 141. Configuration Integrity

For production configuration:

```text
Git review
branch protection
signed commits where required
deployment approval
checksum/signature where needed
```

can protect against unauthorized modification.

---

# 142. Configuration Ownership in GitOps

A strong model:

```text
Application config -> application repo
Infrastructure config -> infrastructure repo
Secrets -> secret manager
```

Avoid multiple competing sources of truth.

---

# 143. Configuration Repository Structure

Example:

```text
config/
├── base/
│   └── config.yaml
├── environments/
│   ├── dev.yaml
│   ├── staging.yaml
│   └── production.yaml
└── schemas/
    └── config.schema.json
```

---

# 144. Configuration Promotion

A safe promotion path:

```text
dev
 |
 v
staging
 |
 v
production
```

with validation at each stage.

---

# 145. Configuration Promotion and Git

Use:

```text
PR
review
CI validation
security checks
merge
deployment
```

rather than manual copy/paste.

---

# 146. Configuration Rollback Strategy

Before deployment:

```text
record current version
```

After deployment:

```text
verify
```

If failure:

```text
revert to known-good configuration
```

---

# 147. Configuration and Observability

Log:

```text
config_version
environment
application_version
```

This allows operators to correlate behavior with configuration changes.

---

# 148. Config Version

Example:

```text
config_version=2026-08-18.3
```

Do not include secrets in configuration identifiers.

---

# 149. Configuration Change Audit

Track:

```text
who changed
what changed
when changed
why changed
approval
deployment result
```

Git history and CI/CD records can provide much of this.

---

# 150. Dynamic Configuration Risks

Dynamic configuration can change behavior without redeployment.

Risks:

```text
unexpected runtime change
incompatible value
partial rollout
lack of audit
```

Use dynamic configuration only where its benefits justify the complexity.

---

# 151. Configuration Reload Race

If multiple threads read a configuration object while another thread replaces it, ensure the update is atomic from the application's perspective.

Prefer:

```text
build new immutable config
swap reference
```

over:

```text
mutate shared config field-by-field
```

---

# 152. Configuration Snapshot

At startup, create:

```text
config snapshot
```

and use it consistently during one operation.

This prevents one workflow from seeing:

```text
timeout=30
```

and later:

```text
timeout=60
```

unexpectedly.

---

# 153. Per-Run Configuration

For CI/CD:

```text
workflow run
    |
    v
load config
    |
    v
validate
    |
    v
immutable run config
```

This provides deterministic execution.

---

# 154. Configuration and Concurrency

Concurrent tasks should usually share:

```text
immutable configuration
```

rather than modifying shared configuration.

Example:

```python
def worker(config, item):
    ...
```

All workers receive the same safe snapshot.

---

# 155. Configuration and Testing

Dependency injection makes tests easier:

```python
config = Config(
    environment="test",
    region="test-region",
    namespace="test",
    timeout=5,
    workers=2,
)
```

No need to mutate global environment state for every test.

---

# 156. Configuration and Mocking

External configuration sources can be mocked:

```text
AWS Secrets Manager
SSM
environment
filesystem
```

This keeps unit tests deterministic.

---

# 157. Configuration and Local Development

Provide a safe developer configuration:

```yaml
environment: development
dry_run: true
workers: 2
log_level: DEBUG
```

Never use production credentials by default.

---

# 158. Developer Safety Guard

A local script should not silently detect:

```text
production
```

and execute destructive actions.

Require explicit intent for production.

---

# 159. Production Confirmation

For high-risk CLI tools, require an explicit confirmation mechanism such as:

```text
--confirm-production
```

combined with:

```text
account validation
cluster validation
```

Do not rely on a confirmation flag alone.

---

# 160. Environment Allowlist

Example:

```python
ALLOWED_ENVIRONMENTS = {
    "development",
    "staging",
    "production",
}
```

Reject unknown environments.

---

# 161. Environment-Specific Policies

Example:

```text
development -> auto apply allowed
staging -> auto apply allowed
production -> approval required
```

Keep these rules explicit.

---

# 162. Configuration Policy Enforcement

Use policy checks for:

```text
production concurrency
minimum timeout
maximum retries
allowed regions
allowed accounts
required security checks
```

Configuration should not bypass these controls.

---

# 163. Configuration and Resource Limits

Example:

```yaml
concurrency:
  workers: 10
  max_workers: 20
```

If the user supplies:

```text
workers=1000
```

reject it.

---

# 164. Configuration and Cost Controls

Configuration can control expensive operations:

```text
scan all repositories
scan all regions
discover all accounts
```

Set safe limits:

```text
max repositories
max regions
max concurrent calls
```

---

# 165. Configuration and AWS Multi-Account

For multi-account automation:

```text
account configuration
       |
       v
assume role
       |
       v
validate identity
       |
       v
perform operation
```

Never rely only on an account alias.

---

# 166. Multi-Region Configuration

Example:

```yaml
regions:
  - ap-south-1
  - ap-southeast-1
```

Validate allowed regions before starting.

For destructive operations, explicit allowlists are safer.

---

# 167. Configuration and EKS Multi-Cluster

Example:

```yaml
clusters:
  staging: staging-eks
  production: production-eks
```

The selected environment should resolve to the expected cluster and then be verified against the actual cluster identity.

---

# 168. Configuration and Kubernetes Context

Do not blindly run:

```bash
kubectl config current-context
```

and assume it is safe.

Validate:

```text
context
cluster endpoint
account
namespace
```

against expected values.

---

# 169. Configuration and AWS Profiles

Local development may use:

```text
AWS_PROFILE
```

CI/CD should generally use:

```text
workload identity
role assumption
OIDC
```

instead of developer credentials.

---

# 170. Configuration Source Availability

If the configuration service is unavailable:

```text
fail closed
```

for critical security/configuration values.

Do not silently fall back to unsafe production defaults.

---

# 171. Safe Fallbacks

A fallback is acceptable only when it is:

```text
known
documented
safe
non-sensitive
```

Example:

```text
log level -> INFO
```

is usually safer than:

```text
production target -> default cluster
```

---

# 172. Configuration Failure Modes

Examples:

```text
missing config
invalid config
stale config
conflicting config
unavailable secret store
wrong environment
wrong AWS account
wrong cluster
```

Each should have a deliberate failure behavior.

---

# 173. Configuration Error Handling

Do not continue after critical validation failure:

```text
config invalid
   |
   X
do not deploy
```

Return a clear error and non-zero exit code.

---

# 174. Exit Codes

For CLI automation:

```text
0 -> success
non-zero -> failure
```

Use meaningful non-zero behavior consistently so CI/CD can detect failures.

---

# 175. Configuration and CI/CD

Pipeline:

```text
checkout
  |
  v
load configuration
  |
  v
validate schema
  |
  v
validate secrets
  |
  v
validate environment
  |
  v
run tests
  |
  v
deploy
```

---

# 176. Configuration and DevSecOps

Security checks can validate:

```text
no plaintext secrets
allowed endpoints
secure TLS
approved images
approved regions
approved IAM roles
```

Configuration becomes part of the security boundary.

---

# 177. Configuration and SonarQube

Static analysis can identify:

```text
hard-coded credentials
unsafe patterns
configuration mistakes
```

Use it as one layer of defense.

---

# 178. Configuration and Trivy

Trivy can scan:

```text
container images
filesystem
configuration/IaC
```

depending on the configured scanning mode.

This can identify security misconfigurations before deployment.

---

# 179. Configuration and Veracode

Application security platforms can identify vulnerabilities in code and dependencies.

Configuration should still be validated by runtime/platform controls.

---

# 180. Configuration and Ansible

Ansible uses variables from:

```text
inventory
group vars
host vars
extra vars
defaults
```

Python automation integrating with Ansible should avoid creating conflicting configuration sources.

---

# 181. Configuration and JFrog Artifactory

Repository URLs and credentials should come from:

```text
secure configuration
secret manager
CI credentials
```

not source code.

---

# 182. Configuration and Maven

Build configuration can include:

```text
repository
profiles
timeouts
versions
```

Python orchestration should pass only the required values and avoid hard-coded credentials.

---

# 183. Configuration and Node.js

Python may orchestrate Node.js applications.

Keep:

```text
NODE_ENV
API URLs
feature flags
```

outside application source where environment-specific.

---

# 184. Configuration and Java

Java services commonly use:

```text
environment variables
application.yaml
system properties
```

Python automation should understand the target application's configuration contract before changing values.

---

# 185. Configuration and Bash

Avoid:

```bash
export SECRET=...
```

when a secure CI/CD secret mechanism exists.

Python subprocess calls should pass only required environment variables.

---

# 186. Subprocess Environment

Instead of changing global environment state:

```python
env = os.environ.copy()
env["ENVIRONMENT"] = "staging"

subprocess.run(
    command,
    env=env,
    check=True,
)
```

This keeps the override scoped to the subprocess.

---

# 187. Configuration and Child Processes

Child processes inherit environment variables unless explicitly changed.

Be careful not to expose:

```text
production credentials
```

to unrelated subprocesses.

---

# 188. Configuration and Temporary Files

Do not write secrets to:

```text
/tmp/config
```

unless the security model explicitly permits it.

If temporary files are necessary:

```text
restrict permissions
cleanup
avoid logging paths containing secrets
```

---

# 189. Configuration and File Permissions

Configuration containing sensitive data should have restrictive permissions.

Example:

```text
owner read/write
others no access
```

Exact permissions depend on the runtime model.

---

# 190. Configuration and Containers

For sensitive mounted files:

```text
read-only
least privilege
appropriate filesystem permissions
```

Use:

```yaml
readOnly: true
```

where appropriate.

---

# 191. Configuration and Read-Only Filesystems

Production containers may use:

```yaml
securityContext:
  readOnlyRootFilesystem: true
```

when compatible.

This reduces accidental modification of application files.

---

# 192. Configuration and Sidecars

If configuration requires a sidecar:

```text
config source
    |
    v
sidecar
    |
    v
shared volume
    |
    v
Python application
```

Ensure update semantics and permissions are well defined.

---

# 193. Configuration and Secret Rotation in Kubernetes

If a secret changes:

```text
secret manager
   |
   v
Kubernetes secret
   |
   v
pod
```

The application may need restart or reload depending on how the secret is consumed.

Test rotation behavior.

---

# 194. Configuration and AWS Secret Rotation

For rotating database credentials:

```text
rotate secret
      |
      v
application retrieves new credential
      |
      v
new connection
```

Connection pooling may mean existing connections continue using old credentials until recycled.

Design accordingly.

---

# 195. Configuration Consistency

A production deployment may involve:

```text
application version
configuration version
database schema version
infrastructure version
```

These versions must be compatible.

---

# 196. Configuration Compatibility Matrix

Example:

```text
App v3
Config v5
DB schema v8
```

must be an approved combination.

This matters during rolling deployments.

---

# 197. Configuration Rollout

For risky configuration:

```text
small scope
   |
   v
observe
   |
   v
expand
```

This can be done through:

```text
canary
feature flag
environment promotion
progressive rollout
```

---

# 198. Configuration Blast Radius

A configuration change affecting:

```text
all production pods
```

has high blast radius.

Reduce risk with:

```text
canary
staged rollout
validation
rollback
```

---

# 199. Configuration Canary

Example:

```text
100 pods
 |
 +--> 5 pods use new config
 |
 v
observe
 |
 v
95 pods
```

Only if the application/platform supports this safely.

---

# 200. Configuration Freeze

During incidents, configuration changes may be restricted to reduce variables.

Example:

```text
active production incident
        |
        v
configuration freeze
        |
        v
only emergency changes
```

---

# 201. Configuration During Incident Response

Capture:

```text
current config version
environment
application version
recent changes
```

before making additional modifications where possible.

---

# 202. Configuration and Observability Correlation

Example log:

```text
service=orders
version=3.4.1
config_version=2026-08-18.4
environment=production
```

This makes regressions easier to correlate.

---

# 203. Configuration Metrics

Useful metrics:

```text
config_load_failures_total
config_validation_failures_total
config_reload_total
config_version_info
```

Do not put secret values in metric labels.

---

# 204. Configuration Health Endpoint

For services, a health endpoint may expose safe metadata:

```json
{
  "status": "ok",
  "config_version": "2026-08-18.4"
}
```

Do not expose:

```text
credentials
tokens
private endpoints
sensitive configuration
```

---

# 205. Production Configuration Checklist

```text
[ ] Configuration separated from code
[ ] Precedence documented
[ ] Types validated
[ ] Ranges validated
[ ] Unknown keys handled
[ ] Safe defaults
[ ] Required fields enforced
[ ] Environment validated
[ ] AWS account validated
[ ] AWS region validated
[ ] EKS cluster validated
[ ] Namespace validated
[ ] Secrets separated
[ ] Secrets not logged
[ ] Secrets not committed
[ ] Production guards enabled
[ ] Dry-run available where appropriate
[ ] Configuration version tracked
[ ] Configuration changes reviewed
[ ] Rollback strategy defined
[ ] Drift detection considered
```

---

# 206. Configuration Security Checklist

```text
[ ] No hard-coded credentials
[ ] No secrets in Git
[ ] No secrets in CLI arguments
[ ] No secrets in logs
[ ] No secrets in Docker image layers
[ ] Least-privilege IAM
[ ] OIDC/workload identity where appropriate
[ ] Secret manager used
[ ] TLS verification enabled
[ ] URL allowlists where needed
[ ] Path validation
[ ] Command arguments separated
[ ] Sensitive files protected
[ ] Secret rotation supported
```

---

# 207. Configuration Testing Checklist

```text
[ ] Valid configuration
[ ] Missing required value
[ ] Invalid type
[ ] Invalid range
[ ] Unknown key
[ ] Invalid environment
[ ] Wrong AWS account
[ ] Wrong region
[ ] Wrong cluster
[ ] Wrong namespace
[ ] Missing secret
[ ] Conflicting values
[ ] CLI override
[ ] Environment override
[ ] Config file override
[ ] Production guard
[ ] Dry-run behavior
```

---

# 208. Common Configuration Anti-Patterns

Avoid:

```text
hard-coded production values
global mutable configuration
unclear precedence
silent fallback
defaulting to production
plaintext secrets
secrets in Git
secrets in CLI arguments
logging entire config
unvalidated environment variables
unbounded workers from config
unlimited retry configuration
configuration that bypasses security policy
multiple competing sources of truth
manual production edits without audit
blind trust in Kubernetes context
blind trust in AWS profile
```

---

# 209. Senior Interview — How Do You Manage Python Configuration?

Strong answer:

> I separate configuration from application logic and load it through a centralized configuration layer. I define a clear precedence between defaults, files, environment variables and CLI arguments, then validate the resulting configuration into a typed object. Critical values such as environment, AWS account, region and EKS cluster are validated before any mutation occurs.

---

# 210. Senior Interview — How Do You Handle Secrets?

Strong answer:

> I separate secrets from ordinary configuration and retrieve them from a managed secret store or workload identity mechanism. I avoid hard-coded credentials, CLI arguments and logs containing secrets. For AWS workloads I prefer IAM roles and appropriate workload identity over long-lived access keys.

---

# 211. Senior Interview — ConfigMap vs Secret?

Strong answer:

> ConfigMap is intended for non-sensitive Kubernetes configuration. Secret is intended for sensitive values, although the security of secrets also depends on RBAC, encryption at rest and cluster configuration. For higher-security environments I often integrate Kubernetes with an external secret-management system.

---

# 212. Senior Interview — Parameter Store vs Secrets Manager?

Strong answer:

> Parameter Store is useful for hierarchical configuration and parameters, including SecureString values. Secrets Manager is designed around secret lifecycle management and supports features such as rotation. I choose based on whether the value is configuration or a credential/secret with lifecycle requirements.

---

# 213. Senior Interview — How Do You Prevent Deployment to the Wrong AWS Account?

Strong answer:

> I do not rely only on the configured profile or account alias. Before mutation I call AWS identity information, compare the actual account ID against the expected account for the selected environment, and fail closed if they do not match.

---

# 214. Senior Interview — How Do You Prevent Deployment to the Wrong EKS Cluster?

Strong answer:

> I validate the selected environment, expected cluster name and actual Kubernetes cluster identity before performing mutations. For production I use explicit safeguards and require intentional confirmation or approval depending on the workflow.

---

# 215. Senior Interview — How Do You Design Configuration Precedence?

Strong answer:

> I define it explicitly, for example defaults, config file, environment variables and finally CLI overrides. The final configuration is resolved once, validated, and converted into a typed immutable configuration object so the rest of the application doesn't repeatedly read different sources.

---

# 216. Senior Interview — Why Validate Configuration Before API Calls?

Strong answer:

> Invalid configuration should fail before external side effects. Otherwise a missing or incorrect value can cause partial infrastructure changes before the error is detected. I load and validate configuration first, then create clients and perform mutations.

---

# 217. Senior Interview — How Do You Handle Production Configuration Safely?

Strong answer:

> I use Git-reviewed configuration where appropriate, secret managers for secrets, explicit environment/account/cluster validation, safe defaults, dry-run support and rollback. Production changes go through controlled CI/CD and audit mechanisms rather than ad-hoc local edits.

---

# 218. Senior Interview — How Do You Avoid Configuration Drift?

Strong answer:

> I define a source of truth and avoid multiple competing configuration systems. For GitOps-managed Kubernetes configuration, Git is the desired state and ArgoCD reconciles it. I also track configuration versions and use drift detection and review processes.

---

# 219. Senior Interview — How Do You Handle Dynamic Configuration?

Strong answer:

> I use dynamic configuration only where runtime updates provide meaningful value. I define update semantics, validation, rollback and concurrency behavior. For simpler DevOps automation, immutable configuration snapshots per run are often safer and easier to troubleshoot.

---

# 220. Senior Interview — How Do You Test Configuration?

Strong answer:

> I unit-test validation for missing fields, types, ranges and conflicting values. I also test environment/account/cluster guards and integration-test the resulting configuration against staging infrastructure. Production configuration changes are validated through CI before deployment.

---

# 221. Senior Interview — What Is the Most Dangerous Configuration Mistake in DevOps?

Strong answer:

> A configuration mistake that changes the deployment target, such as selecting the wrong AWS account or Kubernetes cluster. I treat target selection as a security boundary and verify actual runtime identity before allowing destructive operations.

---

# 222. Senior Interview — How Do You Handle Secrets in CI/CD?

Strong answer:

> I use the CI/CD platform's secret mechanism or an external secret manager and prefer short-lived workload identity such as OIDC where supported. I make sure secrets are masked, never printed, never committed, and are scoped to the minimum job/environment that requires them.

---

# 223. Senior Interview — How Do You Pass Configuration to a Container?

Strong answer:

> Non-secret configuration can be injected through environment variables, ConfigMaps or mounted configuration files. Secrets should use secret-specific mechanisms. I prefer immutable images and inject environment-specific configuration at runtime rather than rebuilding images for every environment.

---

# 224. Senior Interview — How Do You Handle Configuration Rollback?

Strong answer:

> I version configuration and keep a known-good version. If a change causes a problem, I revert to that version through the normal deployment mechanism and verify the runtime state. In GitOps systems, reverting the Git commit allows ArgoCD to reconcile the previous desired state.

---

# 225. Senior Interview — How Do You Handle Configuration During a Production Incident?

Strong answer:

> I first identify the application, configuration and infrastructure versions involved. I avoid making multiple uncontrolled changes. I capture the current state, apply the smallest safe change through the approved process, monitor the result, and keep rollback available.

---

# 226. Real-World Scenario — Wrong AWS Region

Configuration:

```text
AWS_REGION=us-east-1
```

Expected:

```text
ap-south-1
```

Safe flow:

```text
load config
   |
   v
expected region
   |
   v
validate AWS identity/region
   |
   X
fail before mutation
```

---

# 227. Real-World Scenario — Wrong EKS Cluster

Input:

```text
environment=production
cluster=staging-eks
```

Do not execute.

Validate:

```text
environment
cluster mapping
actual cluster identity
```

Then fail clearly.

---

# 228. Real-World Scenario — Missing Secret

Bad:

```text
secret missing
   |
   v
use empty string
   |
   v
deployment
```

Better:

```text
secret missing
   |
   X
fail validation
   |
   v
do not deploy
```

---

# 229. Real-World Scenario — Invalid Workers

Configuration:

```text
WORKERS=10000
```

Policy:

```text
maximum = 50
```

Fail:

```text
WORKERS exceeds configured safety limit
```

Do not silently clamp unless that behavior is explicitly designed and documented.

---

# 230. Real-World Scenario — Conflicting Configuration

Inputs:

```text
config.yaml -> staging
ENVIRONMENT -> production
```

If precedence says environment wins:

```text
final = production
```

But still validate:

```text
AWS account
cluster
namespace
```

before mutation.

---

# 231. Real-World Scenario — Configuration Drift

Git says:

```text
replicas=5
```

Cluster has:

```text
replicas=3
```

If ArgoCD manages the resource:

```text
ArgoCD detects drift
      |
      v
reconciliation
```

Python should not create a competing direct mutation path.

---

# 232. Real-World Scenario — Secret Rotation

Application uses:

```text
database password v1
```

Secret manager rotates:

```text
password v2
```

Application design must determine:

```text
reload automatically?
restart?
new connections use v2?
existing connections?
```

Test this before relying on automatic rotation.

---

# 233. Real-World Scenario — Production Config Change

Safe flow:

```text
change config
   |
   v
schema validation
   |
   v
security checks
   |
   v
PR review
   |
   v
CI
   |
   v
staging
   |
   v
production
   |
   v
verify
```

---

# 234. Real-World Scenario — Local Script

Developer runs:

```bash
python deploy.py
```

The script detects production credentials.

Safe design:

```text
production detected
       |
       v
require explicit production mode
       |
       v
verify account
       |
       v
verify cluster
       |
       v
confirm/approve
       |
       v
execute
```

---

# 235. Real-World Scenario — Configuration Service Down

If required configuration cannot be retrieved:

```text
configuration unavailable
```

Do not:

```text
default to production target
```

Instead:

```text
fail closed
report dependency failure
```

---

# 236. Real-World Scenario — Secret Accidentally Logged

Example:

```text
logger.info("config=%s", config)
```

If config contains a token, it may be exposed.

Fix:

```text
redacted representation
```

and test log output for secret leakage.

---

# 237. Real-World Scenario — Environment Variable Boolean

Input:

```text
AUTO_APPLY=false
```

Bad:

```python
bool(os.getenv("AUTO_APPLY"))
```

Result:

```text
True
```

Correct:

```text
explicit boolean parser
```

---

# 238. Real-World Scenario — Configuration in Docker

Bad:

```dockerfile
ENV API_TOKEN=production-secret
```

This can expose the credential through image metadata/layers.

Better:

```text
runtime secret injection
```

---

# 239. Real-World Scenario — Configuration in GitOps

Bad:

```text
Python -> kubectl patch
```

while:

```text
Git -> desired state
```

Better:

```text
Python
  |
  v
Git change
  |
  v
PR/validation
  |
  v
ArgoCD
  |
  v
EKS
```

---

# 240. Production Configuration Architecture

```text
                  Configuration Sources
                         |
        +----------------+----------------+
        |                |                |
      Git              Env             CLI
        |                |                |
        +----------------+----------------+
                         |
                         v
                 Configuration Loader
                         |
                         v
                 Schema Validation
                         |
                         v
              Environment/Target Guards
                         |
                         v
                Typed Config Snapshot
                         |
             +-----------+-----------+
             |                       |
             v                       v
      Application Logic       Secret Manager
                                     |
                                     v
                              Runtime Secret
```

---

# 241. DevOps Configuration Architecture

```text
Git
 |
 +--> Terraform
 |
 +--> Helm
 |
 +--> Kubernetes manifests
 |
 +--> Python configuration
 |
 v
CI/CD
 |
 +--> validation
 +--> SonarQube
 +--> Trivy
 +--> tests
 |
 v
ArgoCD / Deployment
 |
 v
EKS
```

---

# 242. Configuration Golden Rules

```text
1. Keep configuration outside application logic.
2. Define precedence explicitly.
3. Validate before side effects.
4. Fail fast on invalid critical configuration.
5. Fail closed for target identity failures.
6. Separate secrets from ordinary configuration.
7. Never log secrets.
8. Never hard-code credentials.
9. Prefer workload identity for AWS.
10. Use immutable per-run configuration.
11. Validate AWS account and region.
12. Validate EKS cluster and namespace.
13. Use safe defaults.
14. Put upper bounds on dangerous values.
15. Track configuration versions.
16. Review production configuration changes.
17. Use one source of truth.
18. Avoid configuration drift.
19. Test configuration independently.
20. Make rollback possible.
```

---

# 243. Final Mental Model

Think of configuration management as:

```text
                 SOURCE
                   |
        +----------+----------+
        |          |          |
       Git        ENV        CLI
        |          |          |
        +----------+----------+
                   |
                   v
                MERGE
                   |
                   v
               VALIDATE
                   |
       +-----------+-----------+
       |           |           |
     TYPES       TARGET      SECURITY
       |           |           |
       +-----------+-----------+
                   |
                   v
             CONFIG SNAPSHOT
                   |
                   v
              APPLICATION
                   |
                   v
              SAFE ACTION
```

The DevOps mindset is:

> **Configuration is part of the production control plane. Treat it with the same discipline as code, infrastructure, security, and deployment pipelines.**

For Python DevOps automation:

```text
externalized configuration
+
strict validation
+
target verification
+
secret separation
+
immutable config snapshot
+
versioning
+
auditability
+
rollback
=
production-safe configuration management
```

---

# 244. Section Progress

```text
10-Python-Production/
│
├── 01-Production-Scripting.md        ✓
├── 02-Error-Handling-and-Retry.md    ✓
├── 03-Logging-and-Observability.md   ✓
├── 04-Security.md                    ✓
├── 05-Performance.md                 ✓
├── 06-Concurrency.md                 ✓
├── 07-Configuration-Management.md    ✓
└── 08-Production-Best-Practices.md
```

Next:

```text
08-Production-Best-Practices.md
```
