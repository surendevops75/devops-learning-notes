# 04-Configuration-Automation

## Python Automation — Configuration Management

This module focuses on using Python to **generate, validate, transform, deploy, compare, and safely update configuration** in DevOps environments.

Configuration automation is different from simply writing a file.

A production-safe workflow is:

```text
source
  ↓
template / desired state
  ↓
render
  ↓
validate
  ↓
security checks
  ↓
backup
  ↓
atomic deployment
  ↓
reload / restart
  ↓
health check
  ↓
verification
```

---

# 1. What Is Configuration Automation?

Configuration automation means using code to manage configuration consistently across:

```text
servers
applications
containers
Kubernetes
CI/CD systems
load balancers
web servers
databases
monitoring
```

Examples:

```text
nginx.conf
systemd unit files
application.yaml
.env
JSON configuration
YAML manifests
INI files
TOML files
Helm values
Kubernetes ConfigMaps
```

---

# 2. Why Configuration Automation Matters

Manual configuration causes:

```text
configuration drift
human errors
inconsistent environments
slow deployments
difficult rollback
poor auditability
```

Automation provides:

```text
repeatability
consistency
version control
validation
auditability
faster deployment
```

---

# 3. Desired-State Thinking

Think in terms of:

```text
desired configuration
        ↓
actual configuration
        ↓
compare
        ↓
apply only required changes
```

This is the same general principle behind:

```text
Terraform
Ansible
Kubernetes
GitOps
```

---

# 4. Python's Role

Python is useful for:

```text
configuration generation
validation
transformation
schema checking
environment-specific rendering
secret integration
pre-deployment checks
configuration comparison
orchestration
reporting
```

Python should not unnecessarily replace specialized configuration-management tools.

---

# 5. Configuration Sources

Configuration can come from:

```text
Git
environment variables
CLI arguments
JSON
YAML
TOML
INI
database
secret manager
parameter store
CI/CD variables
```

A good design defines precedence explicitly.

---

# 6. Configuration Precedence

Example:

```text
defaults
   ↓
configuration file
   ↓
environment variables
   ↓
CLI arguments
```

Higher-priority values override lower-priority values.

Document this clearly.

---

# 7. Separate Configuration from Code

Avoid:

```python
DB_PASSWORD = "mypassword"
```

Prefer:

```python
DB_PASSWORD = os.environ["DB_PASSWORD"]
```

or an approved secret-management mechanism.

---

# 8. Environment Variables

```python
import os

environment = os.getenv(
    "APP_ENV",
    "development",
)

print(environment)
```

---

# 9. Required Environment Variable

```python
import os

db_host = os.environ[
    "DB_HOST"
]
```

If missing, Python raises:

```text
KeyError
```

This can be useful when a required configuration must never silently default.

---

# 10. Validate Environment Variables

```python
import os

port = os.getenv(
    "PORT"
)

if not port:
    raise ValueError(
        "PORT is required"
    )

port = int(port)
```

---

# 11. Boolean Environment Variables

Avoid:

```python
bool(
    os.getenv("DEBUG")
)
```

because:

```text
"false"
```

is truthy in Python.

Use explicit parsing.

```python
def parse_bool(value):
    return value.lower() in {
        "true",
        "1",
        "yes",
        "on",
    }
```

---

# 12. Configuration Object

```python
from dataclasses import dataclass

@dataclass
class AppConfig:
    environment: str
    port: int
    debug: bool
```

This gives configuration a clear structure.

---

# 13. Configuration Validation

Validate:

```text
required fields
types
ranges
allowed values
relationships
security requirements
```

Example:

```python
if config.port not in range(
    1,
    65536,
):
    raise ValueError(
        "Invalid port"
    )
```

---

# 14. Configuration Schema

Example desired structure:

```yaml
app:
  name: myapp
  port: 8080

database:
  host: db.internal
  port: 5432
```

Validate that required sections exist before deployment.

---

# 15. YAML Parsing

Using PyYAML:

```python
import yaml

with open(
    "config.yaml",
    encoding="utf-8",
) as file:

    config = yaml.safe_load(
        file
    )
```

Use:

```python
safe_load()
```

for untrusted YAML.

---

# 16. YAML Generation

```python
import yaml

config = {
    "app": {
        "name": "myapp",
        "port": 8080,
    }
}

with open(
    "config.yaml",
    "w",
    encoding="utf-8",
) as file:

    yaml.safe_dump(
        config,
        file,
        sort_keys=False,
    )
```

---

# 17. JSON Configuration

```python
import json

with open(
    "config.json",
    encoding="utf-8",
) as file:

    config = json.load(file)
```

---

# 18. JSON Generation

```python
with open(
    "config.json",
    "w",
    encoding="utf-8",
) as file:

    json.dump(
        config,
        file,
        indent=2,
    )
```

---

# 19. TOML

Modern Python versions can use:

```python
import tomllib

with open(
    "config.toml",
    "rb",
) as file:

    config = tomllib.load(file)
```

For writing TOML, use an appropriate TOML library.

---

# 20. INI Configuration

```python
from configparser import ConfigParser

parser = ConfigParser()

parser.read(
    "app.ini"
)

host = parser[
    "database"
]["host"]
```

---

# 21. Configuration Formats

Common choices:

```text
YAML  → structured human-readable configuration
JSON  → APIs and machine-readable data
TOML  → application configuration
INI   → simple legacy/configuration systems
.env  → environment variables
```

Choose the format based on the consuming application.

---

# 22. Configuration Templates

Templates are useful when environments differ.

Example:

```text
development
staging
production
```

The structure remains the same while values differ.

---

# 23. Avoid Hardcoding Environment Values

Bad:

```python
if environment == "production":
    host = "10.10.10.10"
```

Better:

```text
configuration data
        ↓
environment-specific values
```

Keep application logic separate.

---

# 24. Template Example

A conceptual template:

```text
APP_NAME={{ app_name }}
PORT={{ port }}
ENVIRONMENT={{ environment }}
```

A templating engine such as Jinja2 can render it.

---

# 25. Jinja2 Rendering

```python
from jinja2 import (
    Environment,
    FileSystemLoader,
)

env = Environment(
    loader=FileSystemLoader(
        "templates"
    )
)

template = env.get_template(
    "app.conf.j2"
)

output = template.render(
    app_name="myapp",
    port=8080,
    environment="production",
)

print(output)
```

---

# 26. Template Directory

Example:

```text
config/
├── templates/
│   ├── app.conf.j2
│   ├── nginx.conf.j2
│   └── systemd.service.j2
├── environments/
│   ├── dev.yaml
│   ├── staging.yaml
│   └── prod.yaml
└── schemas/
    └── app-schema.yaml
```

---

# 27. Keep Templates Generic

Prefer:

```text
template
+
environment data
```

instead of:

```text
one giant template
with dozens of environment-specific conditions
```

---

# 28. Configuration Layering

A useful model:

```text
base.yaml
   +
environment.yaml
   +
runtime overrides
   =
final configuration
```

---

# 29. Deep Merge

Simple dictionary update:

```python
base.update(
    override
)
```

does not perform a deep merge.

Nested structures require deliberate merge logic.

---

# 30. Deep Merge Example

```python
def deep_merge(
    base,
    override,
):
    result = dict(base)

    for key, value in override.items():

        if (
            key in result
            and isinstance(
                result[key],
                dict,
            )
            and isinstance(
                value,
                dict,
            )
        ):
            result[key] = deep_merge(
                result[key],
                value,
            )
        else:
            result[key] = value

    return result
```

---

# 31. Configuration Precedence Example

```text
base.yaml
    ↓
prod.yaml
    ↓
environment variables
    ↓
CLI arguments
```

Final values should be deterministic.

---

# 32. Configuration Validation Before Merge

Validate each source where practical.

Then validate the final merged configuration.

This catches:

```text
invalid base
invalid environment override
invalid final combination
```

---

# 33. Required Configuration

Example:

```python
required = [
    "host",
    "port",
    "environment",
]

for key in required:
    if key not in config:
        raise ValueError(
            f"Missing: {key}"
        )
```

---

# 34. Nested Required Fields

```python
database = config.get(
    "database"
)

if not database:
    raise ValueError(
        "database configuration missing"
    )
```

---

# 35. Allowed Values

```python
allowed = {
    "development",
    "staging",
    "production",
}

if environment not in allowed:
    raise ValueError(
        "Unsupported environment"
    )
```

---

# 36. Range Validation

```python
replicas = config[
    "replicas"
]

if not 1 <= replicas <= 100:
    raise ValueError(
        "Invalid replica count"
    )
```

---

# 37. Cross-Field Validation

Example:

```text
TLS enabled
    ↓
certificate required
```

Python:

```python
if config["tls_enabled"]:
    if not config.get(
        "certificate"
    ):
        raise ValueError(
            "Certificate required"
        )
```

---

# 38. Configuration Schema Validation

For larger systems, use schema-based validation such as:

```text
JSON Schema
Pydantic
Cerberus
custom validators
```

The goal is:

```text
structure
types
constraints
```

---

# 39. Pydantic Example

```python
from pydantic import (
    BaseModel,
)

class Database(BaseModel):
    host: str
    port: int

class AppConfig(BaseModel):
    name: str
    port: int
    database: Database
```

---

# 40. Why Schema Validation?

Without validation:

```text
bad configuration
    ↓
deployment
    ↓
application starts
    ↓
runtime failure
```

With validation:

```text
bad configuration
    ↓
pipeline fails
    ↓
deployment blocked
```

---

# 41. Fail Fast

Configuration automation should fail before deployment when:

```text
required values missing
invalid type
invalid range
invalid secret reference
invalid path
invalid environment
```

---

# 42. Configuration Security

Never expose:

```text
passwords
tokens
private keys
API keys
cloud credentials
```

in:

```text
logs
Git
CI artifacts
error messages
reports
```

---

# 43. Secret References

Prefer:

```yaml
database:
  password_secret: prod-db-password
```

rather than:

```yaml
database:
  password: "SuperSecret"
```

The exact secret manager depends on the platform.

---

# 44. AWS Secret Management

In AWS environments, secrets may be stored in:

```text
AWS Secrets Manager
SSM Parameter Store
```

Python can retrieve approved values at runtime.

Do not hardcode credentials in the script.

---

# 45. Environment Variable Injection

A common pattern:

```text
secret manager
      ↓
runtime
      ↓
environment variable
      ↓
application
```

Keep secrets out of source control.

---

# 46. Kubernetes Secrets

For Kubernetes:

```text
Secret
  ↓
Pod
  ↓
environment variable / mounted file
```

Do not commit plaintext production secret values to Git.

---

# 47. ConfigMap

Use ConfigMap for:

```text
non-sensitive configuration
```

Examples:

```text
log level
feature flags
service endpoints
application settings
```

---

# 48. Configuration and GitOps

Recommended pattern:

```text
Git
 ↓
configuration
 ↓
validation
 ↓
CI
 ↓
approved change
 ↓
ArgoCD
 ↓
Kubernetes
```

Python can be part of the validation/generation stage.

---

# 49. Configuration Drift

Drift occurs when:

```text
Git says A
actual environment says B
```

Automation can detect:

```text
expected
vs
actual
```

---

# 50. Configuration Comparison

For JSON/YAML, compare parsed structures instead of raw text when formatting differences should not matter.

```python
if expected == actual:
    print("No drift")
```

---

# 51. Raw Text vs Structured Comparison

Raw text detects:

```text
formatting changes
comments
ordering
whitespace
```

Structured comparison detects:

```text
semantic differences
```

Choose intentionally.

---

# 52. Configuration Checksum

For exact file identity:

```python
checksum = sha256(
    config_file
)
```

Useful for:

```text
artifact verification
deployment validation
change detection
```

---

# 53. Configuration Backup

Before replacing a critical configuration:

```text
current
   ↓
backup
   ↓
new config
   ↓
validate
   ↓
deploy
```

---

# 54. Atomic Configuration Update

```python
import os
import tempfile

target = Path(
    "/opt/myapp/config.yaml"
)

with tempfile.NamedTemporaryFile(
    mode="w",
    dir=target.parent,
    delete=False,
    encoding="utf-8",
) as file:

    file.write(
        rendered_config
    )

    temporary = file.name

os.replace(
    temporary,
    target,
)
```

---

# 55. Preserve Permissions

When replacing configuration, be careful:

```text
new file
```

may have different:

```text
owner
group
mode
```

from the original.

Set required metadata explicitly.

---

# 56. Configuration Deployment

Example:

```text
render
 ↓
validate syntax
 ↓
validate schema
 ↓
security scan
 ↓
backup
 ↓
atomic replace
 ↓
reload
 ↓
health check
```

---

# 57. Nginx Configuration

Typical workflow:

```text
generate nginx.conf
 ↓
nginx -t
 ↓
install
 ↓
nginx reload
 ↓
endpoint health check
```

Do not reload invalid configuration.

---

# 58. Systemd Configuration

Example:

```text
myapp.service
```

Workflow:

```text
generate
 ↓
validate
 ↓
install
 ↓
daemon-reload
 ↓
restart/reload
 ↓
health check
```

---

# 59. Application Configuration

Example:

```yaml
server:
  port: 8080

database:
  host: db.internal
  port: 5432

logging:
  level: INFO
```

Python can validate and render this before deployment.

---

# 60. Configuration File Ownership

Example policy:

```text
application config → app user/group
secret config      → restricted user/group
system config      → root-owned
```

Actual ownership should follow the service architecture.

---

# 61. Configuration Permissions

Examples:

```text
normal config → 0640
secret file   → 0600
directory     → 0750
```

These are examples, not universal requirements.

---

# 62. File-Based Secrets

If a secret must be written to a file:

```text
create securely
write
set restrictive permissions
avoid logging
use
cleanup
```

Use approved secret-management facilities whenever possible.

---

# 63. Secret Leakage Through Exceptions

Bad:

```python
raise Exception(
    f"Invalid password: {password}"
)
```

Never include secret values in exceptions.

---

# 64. Secret Leakage Through Debugging

Avoid:

```python
print(config)
```

when configuration contains secrets.

Instead redact:

```python
{
    "host": "db.internal",
    "password": "***REDACTED***"
}
```

---

# 65. Configuration Redaction

```python
SENSITIVE_KEYS = {
    "password",
    "token",
    "secret",
    "private_key",
}

def redact(data):
    result = {}

    for key, value in data.items():
        if key.lower() in SENSITIVE_KEYS:
            result[key] = "***REDACTED***"
        else:
            result[key] = value

    return result
```

For nested structures, recursively redact.

---

# 66. Configuration Logging

Log:

```text
configuration version
environment
validation result
change ID
checksum
deployment status
```

Do not log:

```text
secret values
```

---

# 67. Configuration Versioning

Every deployment should identify:

```text
Git commit
release version
configuration version
environment
timestamp
```

---

# 68. Configuration Change Audit

Useful record:

```json
{
  "environment": "production",
  "version": "2026.08.17",
  "commit": "abc123",
  "status": "success"
}
```

---

# 69. Configuration Promotion

```text
development
      ↓
staging
      ↓
production
```

Promote the same tested configuration/artifact where possible instead of rebuilding it differently for each environment.

---

# 70. Immutable Configuration Artifact

Example:

```text
config-2026.08.17.tar.gz
```

Checksum:

```text
SHA-256
```

This makes promotion traceable.

---

# 71. Environment Substitution

Example template:

```text
DATABASE_HOST={{ database_host }}
DATABASE_PORT={{ database_port }}
```

Values:

```text
dev
staging
production
```

Keep secret values outside templates.

---

# 72. Avoid Excessive Template Logic

Bad:

```text
if production
  if region
    if customer
      ...
```

Move business/environment data into configuration structures.

---

# 73. Configuration Repository

Example:

```text
config-repo/
├── base/
│   └── app.yaml
├── dev/
│   └── values.yaml
├── staging/
│   └── values.yaml
└── production/
    └── values.yaml
```

---

# 74. Git as Source of Truth

For GitOps:

```text
Git
 ↓
desired configuration
 ↓
controller
 ↓
environment
```

Avoid manual changes to live configuration because they create drift.

---

# 75. Python Validation in GitOps

Pipeline:

```text
pull request
 ↓
Python validation
 ↓
schema validation
 ↓
security checks
 ↓
review
 ↓
merge
 ↓
ArgoCD sync
```

---

# 76. Kubernetes Manifest Validation

Python can load YAML documents:

```python
import yaml

with open(
    "deployment.yaml",
    encoding="utf-8",
) as file:

    documents = list(
        yaml.safe_load_all(file)
    )
```

Then validate required fields.

---

# 77. Kubernetes Configuration Check

Example:

```python
for document in documents:

    if not document:
        continue

    if "apiVersion" not in document:
        raise ValueError(
            "Missing apiVersion"
        )

    if "kind" not in document:
        raise ValueError(
            "Missing kind"
        )

    if "metadata" not in document:
        raise ValueError(
            "Missing metadata"
        )
```

Use Kubernetes-native validation tools for complete schema validation.

---

# 78. Helm Values

Python can validate:

```text
values.yaml
environment values
required keys
image tags
replica ranges
resource settings
```

Do not duplicate Helm's entire rendering engine unnecessarily.

---

# 79. Image Tag Validation

Example:

```python
image = config[
    "image"
]

if image.endswith(":latest"):
    raise ValueError(
        "Mutable latest tag not allowed"
    )
```

This is a useful DevOps policy check.

---

# 80. Resource Configuration Validation

Example:

```text
requests
limits
replicas
ports
probes
```

Python can enforce organizational policies before deployment.

---

# 81. Configuration Policy

Example policies:

```text
production replicas >= 2
CPU limits required
memory limits required
image tag cannot be latest
TLS required
debug disabled
```

---

# 82. Policy Validation

```python
if (
    environment == "production"
    and replicas < 2
):
    raise ValueError(
        "Production requires HA"
    )
```

---

# 83. Configuration and Security

Validate:

```text
debug disabled
TLS enabled
secure ports
restricted permissions
approved image registry
approved domains
approved secret references
```

---

# 84. Configuration and DevSecOps

Python can be one stage:

```text
source
 ↓
Python validation
 ↓
SonarQube
 ↓
dependency scan
 ↓
Trivy
 ↓
Veracode
 ↓
approval
 ↓
deployment
```

Do not treat Python validation as a replacement for dedicated security tools.

---

# 85. CI/CD Configuration Validation

A pipeline should fail before deployment if:

```text
syntax invalid
schema invalid
security policy violated
required value missing
environment mismatch
```

---

# 86. Pre-Deployment Gate

```text
Configuration
      ↓
Syntax
      ↓
Schema
      ↓
Policy
      ↓
Security
      ↓
Approval
      ↓
Deploy
```

---

# 87. Configuration Diff

Before deployment, generate:

```text
old
vs
new
```

Example:

```text
replicas:
- 2
+ 4

image:
- myapp:1.4
+ myapp:1.5
```

Review changes before promotion.

---

# 88. Avoid Blind Replacement

Do not automatically replace:

```text
production configuration
```

without:

```text
validation
diff
approval
backup
```

when the environment requires controlled change management.

---

# 89. Configuration Rollback

Keep:

```text
current
previous
```

or versioned configurations.

Rollback:

```text
new config
   ↓
failure
   ↓
previous config
   ↓
validate
   ↓
reload
```

---

# 90. Configuration History

Example:

```text
configs/
├── 2026-08-15.yaml
├── 2026-08-16.yaml
└── 2026-08-17.yaml
```

Keep retention according to policy.

---

# 91. Configuration Drift Detection Job

Scheduled job:

```text
expected config
      ↓
actual config
      ↓
compare
      ↓
drift?
      ↓
alert
```

---

# 92. Configuration Drift Report

```text
Environment: production

Expected image:
myapp:1.5

Actual image:
myapp:1.4

Status:
DRIFT DETECTED
```

---

# 93. Don't Automatically Correct Every Drift

Automatic correction can be dangerous during incidents.

Depending on the system:

```text
detect
 ↓
alert
 ↓
investigate
 ↓
reconcile
```

may be safer.

---

# 94. Configuration Reload vs Restart

Reload:

```text
process remains running
configuration re-read
```

Restart:

```text
process stops
process starts
```

Prefer reload when the application supports it and when it is safe.

---

# 95. Service Reload Workflow

```text
update config
 ↓
validate
 ↓
reload
 ↓
health check
```

---

# 96. Service Restart Workflow

```text
update config
 ↓
validate
 ↓
restart
 ↓
wait
 ↓
health check
```

---

# 97. Health Check

After configuration deployment:

```text
process status
endpoint
dependency connectivity
logs
metrics
```

---

# 98. Configuration Deployment Failure

If validation fails:

```text
do not modify active config
```

If reload fails:

```text
restore known-good config
```

If health check fails:

```text
rollback
```

---

# 99. Transaction-Like Deployment

Think:

```text
prepare
 ↓
validate
 ↓
commit
 ↓
verify
```

If commit cannot be safely performed:

```text
abort
```

---

# 100. Configuration Automation Architecture

```text
             Git
              |
              v
      Configuration Source
              |
              v
          Python CLI
              |
       +------+------+
       |             |
       v             v
   Rendering      Validation
       |             |
       +------+------+
              |
              v
          Security
              |
              v
          Packaging
              |
              v
         CI/CD Pipeline
              |
              v
       Deployment Target
              |
              v
       Reload / Restart
              |
              v
        Health Check
              |
              v
          Verification
```

---

# 101. Recommended Project Structure

```text
config_automation/
├── cli.py
├── loader.py
├── renderer.py
├── validator.py
├── merger.py
├── secrets.py
├── deployer.py
├── rollback.py
├── diff.py
├── reporter.py
├── templates/
├── configs/
├── schemas/
└── tests/
```

---

# 102. Loader

Responsibilities:

```text
read YAML
read JSON
read TOML
read environment
```

---

# 103. Renderer

Responsibilities:

```text
load template
inject values
produce final configuration
```

---

# 104. Merger

Responsibilities:

```text
base
+
environment
+
runtime
=
final
```

---

# 105. Validator

Responsibilities:

```text
required fields
types
ranges
cross-field rules
security policies
```

---

# 106. Deployer

Responsibilities:

```text
backup
stage
atomic replace
permissions
ownership
reload
```

---

# 107. Rollback Module

Responsibilities:

```text
identify previous version
restore
validate
reload
health check
```

---

# 108. Diff Module

Responsibilities:

```text
old configuration
new configuration
semantic diff
redaction
```

---

# 109. Reporter

Output:

```text
success
failure
changed fields
version
checksum
environment
```

Never expose secrets.

---

# 110. CLI Example

```bash
python config_manager.py validate \
    --environment production
```

---

# 111. Render Command

```bash
python config_manager.py render \
    --environment production \
    --output build/app.yaml
```

---

# 112. Diff Command

```bash
python config_manager.py diff \
    --current /etc/myapp/app.yaml \
    --new build/app.yaml
```

---

# 113. Deploy Command

```bash
python config_manager.py deploy \
    --environment production \
    --dry-run
```

Then:

```bash
python config_manager.py deploy \
    --environment production
```

---

# 114. Rollback Command

```bash
python config_manager.py rollback \
    --environment production
```

---

# 115. Example Workflow

```text
$ python config_manager.py validate --environment production

Loading configuration...
Rendering templates...
Validating schema...
Checking policies...
Checking secrets references...

Validation: PASS
```

---

# 116. Deployment Output

```text
Environment: production
Version: 2026.08.17
Validation: PASS
Backup: PASS
Deployment: PASS
Reload: PASS
Health check: PASS

Status: SUCCESS
```

---

# 117. Failure Output

```text
Environment: production
Validation: FAIL

Error:
production replicas must be >= 2

Deployment blocked.
```

---

# 118. Configuration Testing

Test:

```text
valid configuration
missing fields
wrong types
invalid ports
invalid environment
invalid secret reference
invalid TLS configuration
invalid replica count
```

---

# 119. Unit Tests

Example:

```python
def test_port_validation():
    config = {
        "port": 70000
    }

    try:
        validate(config)
    except ValueError:
        assert True
```

Prefer a test framework such as pytest for production projects.

---

# 120. Test Every Environment

At minimum:

```text
development
staging
production
```

Each should pass configuration validation.

---

# 121. Golden Configuration Tests

Store known-good expected output:

```text
tests/
├── fixtures/
│   ├── input.yaml
│   └── expected.conf
```

Render and compare.

---

# 122. Configuration Snapshot Tests

Useful when:

```text
template output
```

must remain stable.

Review snapshot changes intentionally.

---

# 123. Security Tests

Check:

```text
no secret leakage
debug disabled in production
secure permissions
approved endpoints
TLS
approved images
```

---

# 124. Secret Redaction Tests

Ensure:

```text
password
token
secret
private key
```

never appear in:

```text
logs
diff
reports
exceptions
```

---

# 125. Configuration Linting

Use specialized tools where available:

```text
yamllint
JSON schema validators
Kubernetes validators
Helm lint
Nginx config test
systemd validation
```

Python can orchestrate these checks.

---

# 126. Python Should Orchestrate, Not Reinvent

If a tool already validates:

```text
nginx configuration
Kubernetes schema
Helm chart
Terraform configuration
```

call the official validator rather than implementing a partial replacement.

---

# 127. Subprocess Validation

```python
import subprocess

result = subprocess.run(
    ["nginx", "-t"],
    capture_output=True,
    text=True,
)

if result.returncode != 0:
    raise RuntimeError(
        result.stderr
    )
```

---

# 128. Avoid Shell Injection

Avoid:

```python
subprocess.run(
    f"rm {user_input}",
    shell=True,
)
```

Prefer:

```python
subprocess.run(
    [
        "command",
        user_input,
    ],
    check=True,
)
```

Validate arguments first.

---

# 129. Configuration File Permissions

After deployment:

```python
config.chmod(
    0o640
)
```

Use the organization's actual required mode.

---

# 130. Ownership Verification

After deployment verify:

```text
owner
group
mode
```

because a configuration may exist but still be unusable by the application.

---

# 131. Configuration Directory

Example:

```text
/opt/myapp/config/
├── app.yaml
├── database.yaml
└── logging.yaml
```

Keep permissions consistent with the application security model.

---

# 132. Configuration Include Files

Some systems support:

```text
main.conf
  ↓
include.d/
  ├── app.conf
  └── security.conf
```

Python can manage these files, but should understand the application's include semantics.

---

# 133. Configuration Ordering

Some configuration systems depend on:

```text
file name ordering
```

For example:

```text
10-default.conf
20-app.conf
90-override.conf
```

Automate naming deliberately.

---

# 134. Configuration Fragment Generation

Instead of rewriting one giant file:

```text
generate fragment
 ↓
validate
 ↓
deploy
```

This can reduce deployment risk for systems supporting fragments.

---

# 135. Configuration Environment Variables

Template:

```text
DATABASE_HOST=${DATABASE_HOST}
DATABASE_PORT=${DATABASE_PORT}
```

Python can generate the final environment-specific configuration while keeping secrets outside source control.

---

# 136. Configuration Defaults

Defaults should be:

```text
safe
documented
predictable
```

Do not provide dangerous production defaults.

---

# 137. Fail-Safe Defaults

Example:

```text
debug = false
TLS = true
public access = false
```

where appropriate for the application.

---

# 138. Configuration Feature Flags

Python can validate:

```yaml
features:
  payments: true
  notifications: false
```

Ensure only approved flags are accepted.

---

# 139. Configuration Type Conversion

Environment variables are strings:

```text
"8080"
"true"
"3"
```

Convert explicitly:

```python
port = int(
    os.environ["PORT"]
)

replicas = int(
    os.environ["REPLICAS"]
)
```

---

# 140. Configuration Validation Error

Good:

```text
PORT must be an integer between 1 and 65535
```

Bad:

```text
invalid config
```

Give actionable errors without exposing secrets.

---

# 141. Configuration Dependency Validation

Example:

```text
feature enabled
 ↓
endpoint required
```

Validate dependencies before deployment.

---

# 142. Production Configuration Example

```yaml
environment: production

app:
  name: orders
  port: 8080

database:
  host: orders-db.internal
  port: 5432

logging:
  level: INFO

security:
  tls: true
```

---

# 143. Production Policy Example

```python
if (
    environment == "production"
    and config["security"]["tls"]
    is not True
):
    raise ValueError(
        "Production requires TLS"
    )
```

---

# 144. Configuration Promotion

A mature pipeline:

```text
PR
 ↓
validate
 ↓
render
 ↓
diff
 ↓
security checks
 ↓
review
 ↓
merge
 ↓
staging
 ↓
verify
 ↓
production
```

---

# 145. Configuration Approval

For critical production configuration:

```text
automated checks
+
human approval
```

may be appropriate.

---

# 146. Change Window

Some configuration changes should happen during:

```text
approved maintenance window
```

especially when restart/downtime is possible.

---

# 147. Configuration Rollback Strategy

Always know:

```text
previous version
backup location
restore command
validation command
reload command
health check
```

before deployment.

---

# 148. Rollback Example

```text
new config
 ↓
validation
 ↓
deploy
 ↓
health check FAIL
 ↓
restore previous
 ↓
validate
 ↓
reload
 ↓
health check
```

---

# 149. Configuration Deployment Safety

Never assume:

```text
file written successfully
```

means:

```text
application is healthy
```

Always perform application-level verification.

---

# 150. Real-World Project — Nginx Config Automation

Build:

```text
nginx_config_manager.py
```

Capabilities:

```text
render server block
validate nginx configuration
backup current config
deploy
reload
health check
rollback
```

---

# 151. Nginx Project Workflow

```text
template
 ↓
render
 ↓
nginx -t
 ↓
backup
 ↓
deploy
 ↓
reload
 ↓
curl health endpoint
 ↓
verify
```

---

# 152. Real-World Project — Application Config Manager

Build:

```text
app_config_manager.py
```

Capabilities:

```text
load YAML
merge environment values
validate schema
redact secrets
generate config
checksum
deploy
rollback
```

---

# 153. Real-World Project — Kubernetes Config Validator

Build:

```text
k8s_config_validator.py
```

Checks:

```text
required metadata
image tags
replicas
resources
probes
security settings
secret references
```

Use Kubernetes-native validation tools as part of the pipeline.

---

# 154. Real-World Project — Configuration Drift Detector

Build:

```text
config_drift.py
```

Usage:

```bash
python config_drift.py \
    --expected expected.yaml \
    --actual actual.yaml
```

Output:

```text
Expected: version 15
Actual:   version 14

DRIFT DETECTED
```

---

# 155. Real-World Project — Configuration Promotion Tool

Workflow:

```text
development
 ↓
validate
 ↓
package
 ↓
staging
 ↓
verify
 ↓
production
```

The tool records:

```text
version
commit
environment
checksum
status
```

---

# 156. Real-World Project — Config Backup and Rollback

Capabilities:

```text
backup
list versions
restore
verify
cleanup old versions
```

---

# 157. Real-World Project — Configuration Audit

Check:

```text
unexpected changes
permissions
ownership
secret exposure
debug settings
TLS
environment
```

---

# 158. DevSecOps Configuration Pipeline

```text
Git
 ↓
Python render
 ↓
Python schema validation
 ↓
yamllint / config lint
 ↓
SonarQube
 ↓
Trivy / security checks
 ↓
Veracode where applicable
 ↓
approval
 ↓
deployment
 ↓
health check
```

---

# 159. Configuration Automation and Terraform

Terraform should manage:

```text
infrastructure
```

Python can assist with:

```text
input generation
validation
reporting
orchestration
```

Avoid using Python to manually mutate resources Terraform believes it owns.

---

# 160. Configuration Automation and Ansible

Ansible is already strong for:

```text
file
template
copy
lineinfile
service
package
```

Python is useful for:

```text
complex preprocessing
custom validation
API integration
data transformation
orchestration
```

---

# 161. Configuration Automation and Jenkins

Jenkins can execute:

```bash
python config_manager.py validate
```

Then:

```text
validation
 ↓
security
 ↓
approval
 ↓
deployment
```

---

# 162. Configuration Automation and GitHub Actions

```yaml
- name: Validate configuration
  run: |
    python config_manager.py validate
```

---

# 163. Configuration Automation and GitLab CI

```yaml
config-validation:
  script:
    - python config_manager.py validate
```

---

# 164. Configuration Automation and ArgoCD

Recommended:

```text
Git configuration
 ↓
PR
 ↓
Python validation
 ↓
merge
 ↓
ArgoCD
 ↓
EKS
```

Python should generally operate before the GitOps controller rather than manually modifying live Kubernetes resources.

---

# 165. Configuration Automation and EKS

For EKS:

```text
configuration
 ↓
Git
 ↓
CI validation
 ↓
ArgoCD
 ↓
Kubernetes
```

Python can provide policy checks around:

```text
replicas
resources
image tags
security settings
```

---

# 166. Production Incident — Invalid Config

Symptoms:

```text
application fails startup
```

Response:

```text
check deployment version
 ↓
inspect config validation
 ↓
rollback known-good config
 ↓
reload/restart
 ↓
health check
```

---

# 167. Production Incident — Config Drift

Symptoms:

```text
Git = version 20
server = version 18
```

Response:

```text
confirm intended state
 ↓
identify manual change
 ↓
check deployment history
 ↓
reconcile
 ↓
prevent recurrence
```

---

# 168. Production Incident — Secret Exposed

Response:

```text
contain exposure
 ↓
rotate credential
 ↓
remove from artifacts/logs where possible
 ↓
investigate access
 ↓
fix secret handling
 ↓
add detection
```

---

# 169. Production Incident — Bad Nginx Config

Response:

```text
stop further deployment
 ↓
restore previous config
 ↓
nginx -t
 ↓
reload
 ↓
health check
 ↓
root-cause analysis
```

---

# 170. Production Incident — Wrong Environment Values

Example:

```text
production application
using staging database
```

Prevent with:

```text
environment validation
allowlisted endpoints
configuration policy
CI/CD checks
```

---

# 171. Production Incident — Debug Enabled in Production

Prevent with policy:

```python
if (
    environment == "production"
    and config["debug"] is True
):
    raise ValueError(
        "Debug cannot be enabled"
    )
```

---

# 172. Production Incident — TLS Disabled

Use a production policy:

```text
production
 ↓
TLS required
 ↓
certificate reference required
```

Fail deployment if violated.

---

# 173. Production Incident — Wrong Image Tag

Policy:

```text
production image
must use immutable version
```

Reject:

```text
latest
```

unless explicitly allowed by architecture/policy.

---

# 174. Production Incident — Missing Configuration

Fail before deployment:

```text
required value missing
```

Do not allow the application to discover the problem only after startup.

---

# 175. Production Incident — Configuration File Permission Changed

Verify:

```text
owner
group
mode
```

and determine why deployment changed them.

---

# 176. Production Incident — Configuration Reload Succeeds but Application Fails

A reload success only proves the configuration was accepted by the process.

Continue with:

```text
endpoint health
dependency checks
metrics
logs
```

---

# 177. Production Incident — Configuration Rollback Fails

Maintain:

```text
known-good backup
known-good checksum
tested restore procedure
```

and test rollback before an emergency requires it.

---

# 178. Configuration Automation Security Checklist

```text
[ ] No secrets in Git
[ ] No secrets in logs
[ ] No secrets in diffs
[ ] Safe YAML parser
[ ] Path validation
[ ] Command injection protection
[ ] Least privilege
[ ] Secure file permissions
[ ] Secure temporary files
[ ] Secret references
[ ] Schema validation
[ ] Production policy checks
[ ] Audit trail
```

---

# 179. Configuration Automation Reliability Checklist

```text
[ ] Idempotent
[ ] Deterministic
[ ] Validated
[ ] Atomic where required
[ ] Backed up
[ ] Versioned
[ ] Rollback available
[ ] Health checked
[ ] Retry bounded
[ ] Concurrency controlled
[ ] Observable
```

---

# 180. Configuration Automation Interview Question

## How do you safely deploy a configuration file?

**Answer:**

> I render the desired configuration, validate syntax and schema, run policy and security checks, create a backup of the active version, write the new file to a temporary location, set the required ownership and permissions, atomically replace the active file, reload the service, and perform a health check. If the health check fails, I restore the known-good version.

---

# 181. Interview Question — How Do You Manage Different Environments?

**Answer:**

> I keep a common base configuration and environment-specific values separate. The final configuration is built deterministically through an explicit precedence model. Secrets are injected through an approved secret-management mechanism rather than stored in Git.

---

# 182. Interview Question — How Do You Prevent Configuration Drift?

**Answer:**

> I use Git as the source of truth, compare expected and actual configuration when necessary, and use GitOps or configuration management to reconcile changes. I also investigate manual changes rather than simply overwriting them during an incident.

---

# 183. Interview Question — How Do You Handle Secrets?

**Answer:**

> I avoid storing plaintext secrets in source code or configuration repositories. I reference a secret-management system and inject secrets at runtime. I also ensure that logs, diffs, exceptions, and reports redact sensitive values.

---

# 184. Interview Question — Why Use Jinja2?

**Answer:**

> Jinja2 is useful when the configuration structure is mostly stable but values differ between environments. I keep environment data separate from the template and validate the rendered output before deployment.

---

# 185. Interview Question — YAML vs JSON?

**Answer:**

> YAML is generally easier for human-maintained configuration, while JSON is common for APIs and machine-oriented data. The decision should be based on the consuming system and operational requirements rather than personal preference.

---

# 186. Interview Question — How Do You Validate Configuration?

**Answer:**

> I validate syntax, schema, types, required fields, ranges, cross-field dependencies, security policies, and environment-specific rules. I fail the CI/CD pipeline before deployment if validation fails.

---

# 187. Interview Question — What Is Configuration Drift?

**Answer:**

> Configuration drift occurs when the actual deployed state differs from the intended state. For example, Git may specify one image tag while the running environment has another.

---

# 188. Interview Question — How Do You Roll Back Configuration?

**Answer:**

> I keep a known-good previous version, validate it, restore it atomically, reload the service, and perform a health check. For larger systems I prefer versioned immutable configuration artifacts.

---

# 189. Interview Question — Why Is Atomic Replacement Important?

**Answer:**

> It prevents consumers from reading a partially written configuration. The complete new file is prepared and validated before it becomes active.

---

# 190. Interview Question — What Is Configuration Precedence?

**Answer:**

> It defines which source wins when multiple sources provide the same value. A common model is defaults, configuration file, environment variables, and finally CLI arguments.

---

# 191. Interview Question — Why Not Hardcode Production Values?

**Answer:**

> Hardcoding makes configuration changes harder to audit and reuse, and it can expose sensitive information. Environment-specific values should be managed as configuration and secrets should use approved secret-management mechanisms.

---

# 192. Interview Question — How Does Python Fit into GitOps?

**Answer:**

> Python can validate, transform, render, and policy-check configuration before it reaches GitOps deployment. Once approved and merged, the GitOps controller remains responsible for reconciling the desired state with the Kubernetes environment.

---

# 193. Interview Question — How Do You Validate Kubernetes YAML?

**Answer:**

> I can use Python to perform custom policy checks and structural validation, but I also use Kubernetes-native validation tools because they understand the actual Kubernetes API schema and semantics.

---

# 194. Interview Question — What If Configuration Reload Succeeds but the Application Is Broken?

**Answer:**

> A successful reload only proves the service accepted the configuration syntax. I then perform application-level health checks, inspect logs and metrics, and roll back if the service is unhealthy.

---

# 195. Interview Question — How Do You Prevent Production Debug Mode?

**Answer:**

> I enforce an explicit policy in the validation stage. If the environment is production and debug is enabled, validation fails and the deployment is blocked.

---

# 196. Interview Question — How Do You Handle Configuration Changes in CI/CD?

**Answer:**

> The pipeline validates the configuration, renders it, generates a diff, performs security/policy checks, runs tests, and requires the appropriate approval before deployment. The deployment records the configuration version and commit.

---

# 197. Interview Question — How Do You Make Configuration Automation Idempotent?

**Answer:**

> I compare the desired and actual state and only apply changes when necessary. I also use atomic replacement and deterministic rendering so repeated executions converge to the same state.

---

# 198. Interview Question — What Should a Configuration Audit Log Contain?

**Answer:**

```text
environment
version
Git commit
actor/change ID
timestamp
validation result
deployment result
checksum
rollback result
```

It should never contain secret values.

---

# 199. Interview Question — What Happens if the Config Deployment Process Crashes?

**Answer:**

> If the deployment uses temporary staging and atomic replacement, the active configuration remains unchanged. The temporary file can be cleaned up safely on the next run.

---

# 200. Interview Question — How Do You Handle Concurrent Configuration Deployments?

**Answer:**

> I use CI/CD concurrency controls or a reliable deployment lock so only one promotion is active. Each deployment is staged independently, and only the approved version is promoted.

---

# 201. Interview Question — How Do You Safely Run External Validators?

**Answer:**

> I use `subprocess.run()` with an argument list rather than shell interpolation, validate inputs, capture output, check the return code, and avoid `shell=True` unless there is a specific trusted requirement.

---

# 202. Interview Question — What Is Your Configuration Deployment Philosophy?

**Answer:**

> Configuration should be treated as a versioned, testable artifact rather than an ad-hoc file edit. I want every production change to be validated, traceable, reversible, and verified after deployment.

---

# 203. Real DevOps Project — Complete Configuration Manager

Build:

```text
config-manager/
├── cli.py
├── loader.py
├── renderer.py
├── merger.py
├── validator.py
├── policy.py
├── secrets.py
├── diff.py
├── deployer.py
├── rollback.py
├── reporter.py
├── templates/
├── environments/
├── schemas/
└── tests/
```

---

# 204. Project Workflow

```text
CLI
 ↓
Load
 ↓
Merge
 ↓
Render
 ↓
Validate
 ↓
Policy
 ↓
Security
 ↓
Diff
 ↓
Backup
 ↓
Deploy
 ↓
Reload
 ↓
Health Check
 ↓
Report
```

---

# 205. Project Features

The project should support:

```text
validate
render
diff
deploy
rollback
status
checksum
backup
```

---

# 206. Project Example

```bash
python config_manager.py validate \
    --env production
```

```bash
python config_manager.py render \
    --env production
```

```bash
python config_manager.py diff \
    --env production
```

```bash
python config_manager.py deploy \
    --env production \
    --dry-run
```

```bash
python config_manager.py rollback \
    --env production
```

---

# 207. Production Configuration Pipeline

```text
Developer
    ↓
Git Pull Request
    ↓
Python Validation
    ↓
Schema Validation
    ↓
Policy Checks
    ↓
Security Checks
    ↓
Review
    ↓
Merge
    ↓
CI/CD
    ↓
Artifact
    ↓
Staging
    ↓
Health Check
    ↓
Production
    ↓
Verification
```

---

# 208. Important DevOps Principle

Do not think:

```text
"Python can modify the config file."
```

Think:

```text
"Python can safely move the system
from its current configuration
to an explicitly validated desired state."
```

---

# 209. Final Configuration Automation Checklist

```text
[ ] Configuration source defined
[ ] Precedence defined
[ ] Secrets separated
[ ] Schema defined
[ ] Validation implemented
[ ] Policy checks implemented
[ ] Templates versioned
[ ] Environment values separated
[ ] Diff available
[ ] Backup available
[ ] Atomic update used
[ ] Ownership verified
[ ] Permissions verified
[ ] Reload/restart validated
[ ] Health check implemented
[ ] Rollback tested
[ ] Audit logging implemented
[ ] CI/CD integrated
[ ] GitOps integrated where applicable
```

---

# 210. Final Takeaway

Production configuration automation is not:

```text
read file
change value
write file
```

It is:

```text
desired state
      ↓
deterministic rendering
      ↓
validation
      ↓
security policy
      ↓
review
      ↓
backup
      ↓
atomic deployment
      ↓
reload
      ↓
health check
      ↓
verification
      ↓
rollback if required
```

The strongest DevOps engineer understands that **configuration is part of the production system**.

Treat it like code:

```text
version it
test it
review it
validate it
secure it
deploy it
monitor it
rollback it
```

That is the core of reliable Python configuration automation.
