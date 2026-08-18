# ConfigMap and Secret Automation with Python

## 1. Overview

Kubernetes ConfigMaps and Secrets provide configuration and sensitive-data management for applications.

Python automation can manage them through the Kubernetes API.

Typical production flow:

```text
Git / CI/CD / Secret Manager
          |
          v
   Python Automation
          |
          v
 Kubernetes API Server
          |
     +----+----+
     |         |
     v         v
 ConfigMap   Secret
     |         |
     +----+----+
          |
          v
      Deployment
          |
          v
         Pods
```

ConfigMaps are intended for non-sensitive configuration.

Secrets are intended for sensitive values, although Kubernetes Secret objects are not automatically equivalent to a fully encrypted external secret-management system.

---

# 2. ConfigMap Fundamentals

A ConfigMap stores non-sensitive configuration.

Examples:

```text
Application environment
Feature flags
URLs
Port values
Application settings
Configuration files
```

Example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: payment-config
  namespace: production
data:
  APP_ENV: production
  LOG_LEVEL: INFO
  PAYMENT_PORT: "8080"
```

Pods can consume ConfigMaps through:

- Environment variables
- Individual environment variables
- Volumes
- Mounted configuration files

---

# 3. Secret Fundamentals

A Secret stores sensitive configuration such as:

```text
Database passwords
API tokens
Credentials
TLS material
Registry credentials
Application secrets
```

Example:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: payment-secret
  namespace: production
type: Opaque
stringData:
  DB_USERNAME: payment
  DB_PASSWORD: example-password
```

Important:

Kubernetes Secret `data` values are Base64 encoded, not inherently encrypted merely because they are Base64 encoded.

Example:

```text
password
   |
   v
Base64 encoding
   |
   v
cGFzc3dvcmQ=
```

Base64 is encoding, not encryption.

---

# 4. ConfigMap vs Secret

| Feature | ConfigMap | Secret |
|---|---|---|
| Purpose | Non-sensitive configuration | Sensitive configuration |
| Typical data | URLs, flags, ports | Passwords, tokens, keys |
| API | CoreV1Api | CoreV1Api |
| Encoding | Plain text | `data` uses Base64 representation |
| Encryption at rest | Cluster configuration dependent | Cluster configuration dependent |
| Access control | RBAC | RBAC |
| External secret integration | Possible | Common |
| Recommended for passwords | No | Yes |

Mental model:

```text
ConfigMap -> configuration
Secret    -> sensitive configuration
```

---

# 5. Why Automate ConfigMaps and Secrets?

Manual creation becomes difficult when applications have:

```text
Development
Staging
Production
```

and multiple microservices.

Python automation can standardize:

```text
Create
Update
Validate
Version
Rotate
Delete
```

It can also integrate with:

```text
Jenkins
GitHub Actions
GitLab CI/CD
AWS Secrets Manager
AWS Systems Manager Parameter Store
Vault
ArgoCD
Kubernetes
```

---

# 6. Python Kubernetes Client

Install:

```bash
python3 -m pip install kubernetes
```

Recommended:

```bash
python3 -m venv .venv
source .venv/bin/activate

pip install --upgrade pip
pip install kubernetes
```

Verify:

```bash
python3 -c "import kubernetes; print(kubernetes.__version__)"
```

---

# 7. Kubernetes Client Configuration

For local automation:

```python
from kubernetes import client, config

config.load_kube_config()

v1 = client.CoreV1Api()
```

For code running inside Kubernetes:

```python
config.load_incluster_config()

v1 = client.CoreV1Api()
```

Production in-cluster automation should normally use:

```text
ServiceAccount
+
RBAC
+
in-cluster authentication
```

---

# 8. ConfigMap Object Structure

A ConfigMap contains:

```text
metadata
data
binaryData
immutable
```

Typical Python object:

```python
config_map = client.V1ConfigMap(
    metadata=client.V1ObjectMeta(
        name="payment-config",
        namespace="production"
    ),
    data={
        "APP_ENV": "production",
        "LOG_LEVEL": "INFO",
        "PAYMENT_PORT": "8080"
    }
)
```

---

# 9. Create a ConfigMap

Complete example:

```python
from kubernetes import client, config
from kubernetes.client.rest import ApiException

config.load_kube_config()

v1 = client.CoreV1Api()

config_map = client.V1ConfigMap(
    metadata=client.V1ObjectMeta(
        name="payment-config",
        namespace="production",
        labels={
            "app": "payment"
        }
    ),
    data={
        "APP_ENV": "production",
        "LOG_LEVEL": "INFO",
        "PAYMENT_PORT": "8080"
    }
)

try:
    response = v1.create_namespaced_config_map(
        namespace="production",
        body=config_map
    )

    print(
        f"ConfigMap created: "
        f"{response.metadata.name}"
    )

except ApiException as e:
    print(f"Kubernetes API error: {e}")
```

---

# 10. Verify ConfigMap

```bash
kubectl get configmap -n production
```

Detailed:

```bash
kubectl describe configmap payment-config -n production
```

Get YAML:

```bash
kubectl get configmap payment-config \
  -n production -o yaml
```

Avoid casually printing ConfigMaps into logs if they may contain information that should not be exposed.

---

# 11. Read a ConfigMap with Python

```python
config_map = v1.read_namespaced_config_map(
    name="payment-config",
    namespace="production"
)

print(config_map.data)
```

Read a specific key:

```python
log_level = config_map.data.get("LOG_LEVEL")

print(log_level)
```

Production code should handle missing keys explicitly.

---

# 12. Create ConfigMap from a Dictionary

A useful automation pattern is to separate configuration from Kubernetes object construction.

```python
application_config = {
    "APP_ENV": "production",
    "LOG_LEVEL": "INFO",
    "PAYMENT_PORT": "8080"
}

config_map = client.V1ConfigMap(
    metadata=client.V1ObjectMeta(
        name="payment-config",
        namespace="production"
    ),
    data=application_config
)
```

This makes the automation reusable.

---

# 13. Environment-Specific Configuration

Avoid duplicating Python scripts for every environment.

Instead:

```python
CONFIG = {
    "dev": {
        "APP_ENV": "dev",
        "LOG_LEVEL": "DEBUG"
    },
    "staging": {
        "APP_ENV": "staging",
        "LOG_LEVEL": "INFO"
    },
    "production": {
        "APP_ENV": "production",
        "LOG_LEVEL": "WARN"
    }
}
```

Select:

```python
environment = "production"

data = CONFIG[environment]
```

For larger systems, use external configuration files or Git-managed environment definitions.

---

# 14. ConfigMap from YAML File

Suppose:

```text
application.yaml
```

contains:

```yaml
server:
  port: 8080

logging:
  level: INFO
```

Python can load the file and store it as a ConfigMap entry.

```python
from pathlib import Path

config_content = Path(
    "application.yaml"
).read_text()

config_map = client.V1ConfigMap(
    metadata=client.V1ObjectMeta(
        name="application-config",
        namespace="production"
    ),
    data={
        "application.yaml": config_content
    }
)
```

This is useful when the application expects a configuration file.

---

# 15. ConfigMap as Environment Variables

Kubernetes YAML:

```yaml
envFrom:
  - configMapRef:
      name: payment-config
```

This exposes ConfigMap keys as environment variables.

Example:

```text
APP_ENV=production
LOG_LEVEL=INFO
PAYMENT_PORT=8080
```

---

# 16. Individual ConfigMap Key

Instead of importing all keys:

```yaml
env:
  - name: APP_ENV
    valueFrom:
      configMapKeyRef:
        name: payment-config
        key: APP_ENV
```

This gives the application only the required configuration.

---

# 17. ConfigMap as a Volume

Example:

```yaml
volumes:
  - name: application-config
    configMap:
      name: application-config

containers:
  - name: payment
    volumeMounts:
      - name: application-config
        mountPath: /etc/payment
```

The Pod receives configuration files under:

```text
/etc/payment
```

This pattern is useful when applications read configuration files rather than environment variables.

---

# 18. Create a ConfigMap from a File Using Python

```python
from pathlib import Path
from kubernetes import client

content = Path(
    "application.yaml"
).read_text()

body = client.V1ConfigMap(
    metadata=client.V1ObjectMeta(
        name="application-config",
        namespace="production"
    ),
    data={
        "application.yaml": content
    }
)
```

Then:

```python
v1.create_namespaced_config_map(
    namespace="production",
    body=body
)
```

---

# 19. Update a ConfigMap

Read:

```python
config_map = v1.read_namespaced_config_map(
    name="payment-config",
    namespace="production"
)
```

Modify:

```python
config_map.data["LOG_LEVEL"] = "WARN"
```

Replace:

```python
v1.replace_namespaced_config_map(
    name="payment-config",
    namespace="production",
    body=config_map
)
```

For small changes, a patch is often safer.

---

# 20. Patch a ConfigMap

```python
patch = {
    "data": {
        "LOG_LEVEL": "WARN"
    }
}

v1.patch_namespaced_config_map(
    name="payment-config",
    namespace="production",
    body=patch
)
```

Patch only what is required.

This reduces unintended modifications.

---

# 21. Immutable ConfigMaps

Kubernetes supports immutable ConfigMaps.

Example:

```python
config_map = client.V1ConfigMap(
    metadata=client.V1ObjectMeta(
        name="payment-config-v1",
        namespace="production"
    ),
    data={
        "APP_ENV": "production"
    },
    immutable=True
)
```

Once immutable:

```text
Data cannot be changed.
```

To change configuration:

```text
Create new ConfigMap
      |
      v
Update workload reference
      |
      v
Rollout
```

This can make configuration changes more predictable.

---

# 22. Why Immutable ConfigMaps Can Be Useful

Immutable configuration can reduce accidental changes.

Example:

```text
payment-config-v1
payment-config-v2
payment-config-v3
```

Deployment explicitly references the desired version.

Benefits:

- Predictability
- Safer rollouts
- Easier rollback
- Reduced accidental mutation

Tradeoff:

- Additional resources
- Workload reference must change

---

# 23. Secret Object Structure

Python:

```python
secret = client.V1Secret(
    metadata=client.V1ObjectMeta(
        name="payment-secret",
        namespace="production"
    ),
    type="Opaque",
    string_data={
        "DB_USERNAME": "payment",
        "DB_PASSWORD": "example-password"
    }
)
```

`stringData` allows the API server to accept string values.

The resulting Secret's `data` representation is Base64 encoded.

---

# 24. Create a Secret Using stringData

```python
from kubernetes import client, config

config.load_kube_config()

v1 = client.CoreV1Api()

secret = client.V1Secret(
    metadata=client.V1ObjectMeta(
        name="payment-secret",
        namespace="production"
    ),
    type="Opaque",
    string_data={
        "DB_USERNAME": "payment",
        "DB_PASSWORD": "example-password"
    }
)

v1.create_namespaced_secret(
    namespace="production",
    body=secret
)

print("Secret created")
```

This is generally easier than manually Base64 encoding values.

---

# 25. Secret Using data

If using the `data` field directly, values must be Base64 encoded.

```python
import base64

username = base64.b64encode(
    b"payment"
).decode()

password = base64.b64encode(
    b"example-password"
).decode()

secret = client.V1Secret(
    metadata=client.V1ObjectMeta(
        name="payment-secret",
        namespace="production"
    ),
    data={
        "DB_USERNAME": username,
        "DB_PASSWORD": password
    },
    type="Opaque"
)
```

Prefer `stringData` when constructing Secrets from plain strings.

---

# 26. Read a Secret

```python
secret = v1.read_namespaced_secret(
    name="payment-secret",
    namespace="production"
)
```

Secret values are represented under:

```python
secret.data
```

They are Base64 encoded.

Decode only when the automation genuinely needs the plaintext value.

---

# 27. Decode Secret Data

```python
import base64

encoded_password = secret.data["DB_PASSWORD"]

password = base64.b64decode(
    encoded_password
).decode()
```

Do not print:

```python
print(password)
```

in production logs.

Avoid exposing credentials through:

```text
stdout
CI logs
debug logs
exception messages
```

---

# 28. Secret Types

Common Secret types include:

```text
Opaque
kubernetes.io/tls
kubernetes.io/dockerconfigjson
kubernetes.io/basic-auth
```

The most common application Secret is:

```text
Opaque
```

---

# 29. TLS Secret

TLS Secret structure:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-tls
  namespace: production
type: kubernetes.io/tls
data:
  tls.crt: <base64>
  tls.key: <base64>
```

Python:

```python
secret = client.V1Secret(
    metadata=client.V1ObjectMeta(
        name="app-tls",
        namespace="production"
    ),
    type="kubernetes.io/tls",
    data={
        "tls.crt": certificate_base64,
        "tls.key": private_key_base64
    }
)
```

Private keys must be handled extremely carefully.

For AWS ALB, TLS certificates are commonly managed through ACM rather than placing private key material into Kubernetes.

---

# 30. Docker Registry Secret

For private container registries, Kubernetes can use:

```text
kubernetes.io/dockerconfigjson
```

Example:

```python
secret = client.V1Secret(
    metadata=client.V1ObjectMeta(
        name="registry-credentials",
        namespace="production"
    ),
    type="kubernetes.io/dockerconfigjson",
    data={
        ".dockerconfigjson": docker_config_base64
    }
)
```

Pods can reference the Secret through:

```yaml
imagePullSecrets:
  - name: registry-credentials
```

In EKS with ECR, prefer AWS-native authentication mechanisms rather than unnecessarily maintaining static registry credentials.

---

# 31. Secret as Environment Variables

Example:

```yaml
env:
  - name: DB_USERNAME
    valueFrom:
      secretKeyRef:
        name: payment-secret
        key: DB_USERNAME

  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: payment-secret
        key: DB_PASSWORD
```

This keeps the Secret object separate from the Deployment specification.

---

# 32. Secret Using envFrom

```yaml
envFrom:
  - secretRef:
      name: payment-secret
```

This imports all keys.

Use carefully because adding a new Secret key can automatically expose it to the application environment.

Explicit `secretKeyRef` entries can provide tighter control.

---

# 33. Secret as a Volume

Example:

```yaml
volumes:
  - name: payment-secret
    secret:
      secretName: payment-secret

containers:
  - name: payment
    volumeMounts:
      - name: payment-secret
        mountPath: /etc/payment/secrets
        readOnly: true
```

The application can read:

```text
/etc/payment/secrets/
```

This can be preferable for applications that consume credentials as files.

---

# 34. ConfigMap + Secret + Deployment Architecture

Example:

```text
                    Deployment
                        |
             +----------+----------+
             |                     |
             v                     v
        ConfigMap               Secret
             |                     |
             +----------+----------+
                        |
                        v
                       Pod
                        |
             +----------+----------+
             |                     |
             v                     v
        Environment           Mounted files
```

Non-sensitive:

```text
APP_ENV
LOG_LEVEL
API_URL
```

Sensitive:

```text
DB_PASSWORD
API_TOKEN
PRIVATE_KEY
```

---

# 35. Python Automation for Deployment Configuration

A deployment automation can create:

```text
1. ConfigMap
2. Secret
3. Deployment
4. Service
5. Ingress
```

Order:

```text
ConfigMap
   |
Secret
   |
Deployment
   |
Service
   |
Ingress
```

The workload references the ConfigMap and Secret.

---

# 36. ConfigMap and Secret Dependency

Suppose a Deployment contains:

```yaml
envFrom:
  - configMapRef:
      name: payment-config
  - secretRef:
      name: payment-secret
```

The Python automation should verify those resources before considering the application configuration complete.

A good workflow:

```text
Create/Update ConfigMap
        |
        v
Create/Update Secret
        |
        v
Validate both
        |
        v
Create/Update Deployment
        |
        v
Wait for rollout
```

---

# 37. Idempotent ConfigMap Automation

Bad:

```python
v1.create_namespaced_config_map(...)
```

on every execution.

Better:

```text
Read ConfigMap
      |
      +-- 404 -> Create
      |
      +-- Exists -> Compare
                     |
                     +-- Same -> No-op
                     |
                     +-- Different -> Patch
```

Example:

```python
from kubernetes.client.rest import ApiException

def ensure_configmap(v1, namespace, body):
    name = body.metadata.name

    try:
        existing = v1.read_namespaced_config_map(
            name=name,
            namespace=namespace
        )

        print(f"ConfigMap exists: {name}")
        return existing

    except ApiException as e:
        if e.status == 404:
            return v1.create_namespaced_config_map(
                namespace=namespace,
                body=body
            )

        raise
```

---

# 38. Idempotent Secret Automation

Same principle:

```python
def ensure_secret(v1, namespace, body):
    name = body.metadata.name

    try:
        existing = v1.read_namespaced_secret(
            name=name,
            namespace=namespace
        )

        print(f"Secret exists: {name}")
        return existing

    except ApiException as e:
        if e.status == 404:
            return v1.create_namespaced_secret(
                namespace=namespace,
                body=body
            )

        raise
```

Do not compare or log plaintext Secret values.

---

# 39. Secret Comparison

This is an important production consideration.

For ConfigMaps, comparing:

```python
desired.data
existing.data
```

is generally straightforward.

For Secrets, do not expose values just to compare them.

Safer approaches include:

```text
External secret manager as source of truth
Version identifiers
Checksums handled carefully
Secret resource version/metadata
Controlled rotation workflow
```

If a value must be compared, keep the operation entirely in memory and never log the plaintext.

---

# 40. Secret Rotation

Secret rotation means replacing an existing credential with a new one.

Example:

```text
Old DB password
      |
      v
Generate / retrieve new password
      |
      v
Update external database credential
      |
      v
Update Kubernetes Secret
      |
      v
Restart/reload application
      |
      v
Validate application
      |
      v
Retire old credential
```

The order matters.

Do not rotate a database credential in a way that causes the application to lose access before the new credential is deployed.

---

# 41. Production Secret Rotation Strategy

A safer pattern is:

```text
1. Generate new credential
2. Add new credential to target system
3. Update Kubernetes Secret
4. Restart/reload workload
5. Validate connectivity
6. Remove old credential
```

For systems supporting dual credentials, use:

```text
old credential + new credential
```

during transition.

This reduces downtime risk.

---

# 42. AWS Secrets Manager Integration

For AWS environments, a strong production architecture is:

```text
AWS Secrets Manager
        |
        v
External Secret Controller
        |
        v
Kubernetes Secret
        |
        v
Pod
```

Python can also retrieve secrets using AWS SDKs when the architecture explicitly requires Python to perform the integration.

However, avoid turning Python scripts into unnecessary secret-storage layers.

Prefer:

```text
Secret Manager
     |
     v
Kubernetes integration
```

over:

```text
Secret Manager
     |
     v
Python logs / files
     |
     v
Kubernetes
```

---

# 43. Python + boto3 Architecture

If Python must retrieve a secret from AWS:

```python
import boto3

client = boto3.client(
    "secretsmanager",
    region_name="ap-south-1"
)

response = client.get_secret_value(
    SecretId="production/payment/database"
)

secret_string = response["SecretString"]
```

Do not print:

```python
print(secret_string)
```

Use IAM permissions with least privilege.

---

# 44. EKS Authentication for AWS APIs

For EKS automation, avoid static AWS access keys where possible.

Use:

```text
IAM role
+
EKS Pod Identity / IRSA where applicable
```

Architecture:

```text
Python Pod
    |
    v
AWS identity
    |
    v
IAM role
    |
    v
Secrets Manager
```

This eliminates long-lived credentials in the application.

---

# 45. Python + AWS Secrets Manager + Kubernetes

A possible automation flow:

```text
Python
  |
  +-- Authenticate to AWS
  |
  +-- Retrieve secret
  |
  +-- Validate secret structure
  |
  +-- Create/patch Kubernetes Secret
  |
  +-- Trigger workload rollout if required
  |
  +-- Validate application
```

Security requirements:

```text
No plaintext logs
No temporary files
Least privilege IAM
Least privilege Kubernetes RBAC
Short-lived identity
```

---

# 46. GitOps Considerations

This is especially important in your environment because ArgoCD is used for GitOps.

Do not put plaintext Secrets into Git.

Bad:

```yaml
stringData:
  DB_PASSWORD: my-production-password
```

inside a normal Git repository.

Better approaches include:

```text
External Secrets
Sealed Secrets
SOPS
Cloud secret manager integration
```

The exact solution depends on the organization's security architecture.

---

# 47. GitOps Architecture for Secrets

Recommended model:

```text
Git
 |
 +-- Deployment manifests
 +-- ConfigMap definitions
 +-- Secret references
 |
 v
ArgoCD
 |
 v
EKS
 |
 v
External Secret Controller
 |
 v
AWS Secrets Manager
```

The actual sensitive value stays outside normal Git source control.

---

# 48. ConfigMap GitOps Architecture

ConfigMaps are usually safe to manage through Git when they contain non-sensitive values.

```text
Git
 |
 v
ArgoCD
 |
 v
ConfigMap
 |
 v
Deployment
 |
 v
Pod
```

This provides:

- Version history
- Review
- Rollback
- Auditability
- Reproducibility

---

# 49. Configuration Drift

Suppose Git contains:

```text
LOG_LEVEL=INFO
```

but someone manually changes the live ConfigMap to:

```text
LOG_LEVEL=DEBUG
```

Now:

```text
Git state != Cluster state
```

ArgoCD can detect this drift if it manages the resource.

Python should not silently bypass GitOps governance.

---

# 50. ConfigMap Update and Pod Behavior

A critical Kubernetes concept:

Changing a ConfigMap does not universally cause a Deployment rollout automatically.

For environment variables:

```yaml
envFrom:
  - configMapRef:
      name: payment-config
```

existing Pods generally retain the environment they received at startup.

A new Pod must be started to receive updated environment variables.

For mounted ConfigMap volumes, Kubernetes can update the projected files asynchronously, subject to Kubernetes behavior and application reload behavior.

Therefore:

```text
ConfigMap update
      |
      +-- Environment variable -> usually requires Pod restart
      |
      +-- Volume mount -> file may update
```

Applications still need a mechanism to reload changed configuration if live updates are expected.

---

# 51. Secret Update and Pod Behavior

Similar principle applies to Secrets.

Environment variables:

```text
Secret -> env variable -> Pod startup
```

Changing the Secret does not automatically rewrite the existing environment variable inside a running process.

Mounted Secret volumes can receive updated content, but the application must know how to reload it.

For applications that require restart after secret rotation:

```text
Secret update
    |
    v
Deployment rollout
    |
    v
New Pods
```

---

# 52. Triggering a Deployment Rollout

A common approach is to update a Pod template annotation.

Example:

```python
from datetime import datetime, timezone

timestamp = datetime.now(
    timezone.utc
).isoformat()

patch = {
    "spec": {
        "template": {
            "metadata": {
                "annotations": {
                    "config-reload": timestamp
                }
            }
        }
    }
}
```

Then patch the Deployment:

```python
apps = client.AppsV1Api()

apps.patch_namespaced_deployment(
    name="payment",
    namespace="production",
    body=patch
)
```

This changes the Pod template and causes a new ReplicaSet rollout.

Use this intentionally, not as a default for every ConfigMap change.

---

# 53. Better Configuration Change Strategy

A stronger GitOps pattern is to use a configuration checksum.

Concept:

```text
ConfigMap content
      |
      v
Checksum
      |
      v
Pod template annotation
      |
      v
Deployment rollout
```

If the ConfigMap changes:

```text
checksum changes
      |
      v
Pod template changes
      |
      v
Kubernetes performs rollout
```

This pattern can be implemented through Helm templates or other deployment tooling.

---

# 54. Namespace Validation

Before creating resources:

```python
def namespace_exists(v1, namespace):
    try:
        v1.read_namespace(namespace)
        return True

    except ApiException as e:
        if e.status == 404:
            return False

        raise
```

Then:

```python
if not namespace_exists(v1, "production"):
    raise RuntimeError(
        "Namespace production does not exist"
    )
```

Avoid silently creating namespaces unless the automation owns namespace lifecycle.

---

# 55. ConfigMap Validation

Example:

```python
def validate_config(data):
    required = [
        "APP_ENV",
        "LOG_LEVEL",
        "PAYMENT_PORT"
    ]

    missing = [
        key for key in required
        if key not in data
    ]

    if missing:
        raise ValueError(
            f"Missing configuration: {missing}"
        )
```

This prevents incomplete configuration from reaching production.

---

# 56. Secret Validation

Do not validate Secrets by printing values.

Instead validate required keys:

```python
def validate_secret_keys(data):
    required = [
        "DB_USERNAME",
        "DB_PASSWORD"
    ]

    missing = [
        key for key in required
        if key not in data
    ]

    if missing:
        raise ValueError(
            f"Missing secret keys: {missing}"
        )
```

If values are fetched from an external system, validate structure and non-empty values without exposing them.

---

# 57. Secret Value Validation

Example:

```python
def validate_non_empty(value, name):
    if not value or not value.strip():
        raise ValueError(
            f"{name} is empty"
        )
```

Do not include the actual value in the error.

Good:

```text
DB_PASSWORD is empty
```

Bad:

```text
DB_PASSWORD=wrong-password
```

---

# 58. Complete ConfigMap Automation

```python
from kubernetes import client, config
from kubernetes.client.rest import ApiException


NAMESPACE = "production"
CONFIG_NAME = "payment-config"


def build_configmap():
    return client.V1ConfigMap(
        metadata=client.V1ObjectMeta(
            name=CONFIG_NAME,
            namespace=NAMESPACE,
            labels={
                "app": "payment"
            }
        ),
        data={
            "APP_ENV": "production",
            "LOG_LEVEL": "INFO",
            "PAYMENT_PORT": "8080"
        }
    )


def ensure_configmap(v1):
    desired = build_configmap()

    try:
        existing = v1.read_namespaced_config_map(
            name=CONFIG_NAME,
            namespace=NAMESPACE
        )

        current = existing.data or {}
        desired_data = desired.data or {}

        if current == desired_data:
            print("ConfigMap already matches desired state")
            return existing

        patch = {
            "data": desired_data
        }

        updated = v1.patch_namespaced_config_map(
            name=CONFIG_NAME,
            namespace=NAMESPACE,
            body=patch
        )

        print("ConfigMap updated")
        return updated

    except ApiException as e:
        if e.status == 404:
            created = v1.create_namespaced_config_map(
                namespace=NAMESPACE,
                body=desired
            )

            print("ConfigMap created")
            return created

        raise


def main():
    config.load_kube_config()

    v1 = client.CoreV1Api()

    ensure_configmap(v1)


if __name__ == "__main__":
    main()
```

---

# 59. Complete Secret Automation

For a simple example using values supplied securely to the process:

```python
import os

from kubernetes import client, config
from kubernetes.client.rest import ApiException


NAMESPACE = "production"
SECRET_NAME = "payment-secret"


def build_secret():
    username = os.environ["DB_USERNAME"]
    password = os.environ["DB_PASSWORD"]

    return client.V1Secret(
        metadata=client.V1ObjectMeta(
            name=SECRET_NAME,
            namespace=NAMESPACE
        ),
        type="Opaque",
        string_data={
            "DB_USERNAME": username,
            "DB_PASSWORD": password
        }
    )


def ensure_secret(v1):
    desired = build_secret()

    try:
        existing = v1.read_namespaced_secret(
            name=SECRET_NAME,
            namespace=NAMESPACE
        )

        print("Secret already exists")

        # Do not print or compare plaintext values.
        return existing

    except ApiException as e:
        if e.status == 404:
            created = v1.create_namespaced_secret(
                namespace=NAMESPACE,
                body=desired
            )

            print("Secret created")
            return created

        raise


def main():
    config.load_kube_config()

    v1 = client.CoreV1Api()

    ensure_secret(v1)


if __name__ == "__main__":
    main()
```

In production, prefer an external secret-management architecture rather than passing long-lived plaintext credentials through CI/CD environment variables when possible.

---

# 60. Complete ConfigMap + Secret + Deployment Workflow

```text
             CI/CD
               |
               v
        Python Automation
               |
       +-------+-------+
       |               |
       v               v
  ConfigMap          Secret
       |               |
       +-------+-------+
               |
               v
          Deployment
               |
               v
         Pods become Ready
               |
               v
         Service / Ingress
               |
               v
         Application
```

Validation:

```text
ConfigMap valid?
Secret keys valid?
Deployment references correct resources?
Pods Ready?
Application healthy?
```

---

# 61. Error Handling

Use Kubernetes API exceptions.

```python
try:
    v1.create_namespaced_secret(
        namespace=NAMESPACE,
        body=secret
    )

except ApiException as e:
    if e.status == 403:
        raise RuntimeError(
            "RBAC denied Secret operation"
        )

    if e.status == 409:
        raise RuntimeError(
            "Secret already exists"
        )

    raise
```

Important statuses:

```text
403 -> RBAC permission problem
404 -> resource does not exist
409 -> resource conflict
429 -> API rate limiting
500 -> API server/internal error
```

---

# 62. Retry and Backoff

Secret/configuration automation can encounter temporary API errors.

Use bounded retries.

```python
import time


def retry(operation, attempts=5):
    delay = 2

    for attempt in range(1, attempts + 1):
        try:
            return operation()

        except Exception:
            if attempt == attempts:
                raise

            time.sleep(delay)
            delay *= 2
```

Production code should classify exceptions so permanent errors such as RBAC failures are not repeatedly retried.

---

# 63. RBAC

A Python application that manages ConfigMaps and Secrets needs permissions.

Example Role:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: config-secret-manager
  namespace: production
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs:
      - get
      - list
      - watch
      - create
      - update
      - patch

  - apiGroups: [""]
    resources: ["secrets"]
    verbs:
      - get
      - create
      - update
      - patch
```

Only grant:

```text
delete
```

if deletion is genuinely required.

---

# 64. Secret RBAC Is High Risk

A principal with:

```text
get secrets
```

can potentially retrieve secret values.

Therefore:

```text
Secret read access
```

should be tightly controlled.

Do not grant:

```text
get/list secrets
```

to every automation component.

Separate responsibilities where possible.

---

# 65. Security Logging

Good:

```python
logger.info(
    "Updating Secret resource=%s namespace=%s",
    SECRET_NAME,
    NAMESPACE
)
```

Bad:

```python
logger.info(
    "Password=%s",
    password
)
```

Also avoid exposing secrets through:

```text
Tracebacks
CI artifacts
Temporary files
Shell history
Debug output
Metrics labels
```

Never put secret values into Prometheus labels or other high-cardinality observability systems.

---

# 66. Secret Data in CI/CD

A common bad architecture:

```text
Jenkins
  |
  v
Plaintext password in shell
  |
  v
Python
  |
  v
Kubernetes
```

Better:

```text
Jenkins credential store / external secret manager
              |
              v
       Controlled access
              |
              v
       Python automation
              |
              v
        Kubernetes Secret
```

Even better when architecture allows:

```text
AWS Secrets Manager
       |
       v
External Secret Controller
       |
       v
Kubernetes Secret
```

This reduces the amount of application code handling secret material.

---

# 67. Secret Management in EKS

A production EKS architecture can use:

```text
AWS Secrets Manager
        |
        v
External Secrets Operator
        |
        v
Kubernetes Secret
        |
        v
EKS Pod
```

AWS identity:

```text
EKS workload
    |
    v
Pod Identity / IRSA
    |
    v
IAM role
    |
    v
Secrets Manager
```

The exact authentication mechanism should match the organization's EKS version and security standards.

---

# 68. ConfigMap and Secret Naming Strategy

Use predictable names.

Examples:

```text
payment-config
payment-secret
inventory-config
inventory-secret
```

For versioned immutable resources:

```text
payment-config-v1
payment-config-v2
```

Avoid ambiguous names such as:

```text
config1
secret-final
new-secret
test-secret-2
```

Production naming should communicate ownership and purpose.

---

# 69. Labels and Ownership

Add useful labels:

```python
labels = {
    "app": "payment",
    "component": "configuration",
    "environment": "production"
}
```

This makes resources easier to discover:

```bash
kubectl get configmaps \
  -l app=payment \
  -n production
```

Labels should follow your organization's Kubernetes labeling standards.

---

# 70. Resource Ownership

A production automation system should know which resources it owns.

Example:

```text
managed-by: python-automation
app: payment
environment: production
```

This helps prevent accidental modification of resources owned by:

```text
ArgoCD
Helm
another controller
another team
```

However, do not invent ownership labels that conflict with an existing GitOps or platform convention.

---

# 71. ConfigMap and Secret Lifecycle

A complete lifecycle:

```text
Create
  |
Validate
  |
Consume
  |
Rotate / Update
  |
Validate
  |
Rollback if required
  |
Delete when no longer needed
```

For Secrets, lifecycle management must include:

```text
Creation
Access control
Rotation
Revocation
Audit
Deletion
```

---

# 72. Safe Deletion

ConfigMap:

```python
v1.delete_namespaced_config_map(
    name="payment-config",
    namespace="production",
    body=client.V1DeleteOptions()
)
```

Secret:

```python
v1.delete_namespaced_secret(
    name="payment-secret",
    namespace="production",
    body=client.V1DeleteOptions()
)
```

Never delete a Secret without checking workload dependencies.

Before deletion:

```text
Which Deployments reference it?
Which Pods mount it?
Which applications depend on it?
Is a replacement already available?
```

---

# 73. Detect Secret/ConfigMap Consumers

Deployments can reference:

```text
env
envFrom
volumes
```

A troubleshooting script can inspect workloads and identify dependencies before changes.

This is useful for platform automation.

---

# 74. Production Change Workflow

For a ConfigMap change:

```text
1. Validate new configuration
2. Create/update ConfigMap
3. Trigger rollout if required
4. Wait for Pods
5. Run health checks
6. Verify application behavior
7. Report result
```

For a Secret rotation:

```text
1. Validate new credential
2. Ensure backend accepts new credential
3. Update Secret
4. Restart/reload application
5. Verify connectivity
6. Retire old credential
7. Record rotation event
```

---

# 75. Rollback Strategy

If a configuration change breaks the application:

```text
Current configuration
        |
        v
Application failure
        |
        v
Identify previous known-good version
        |
        v
Restore configuration
        |
        v
Rollout
        |
        v
Validate
```

GitOps provides a strong rollback mechanism for ConfigMaps.

For Secrets, rollback must consider whether the underlying credential is still valid.

---

# 76. Python + Helm

Python can orchestrate Helm commands or generate Helm values, but avoid unnecessarily mixing deployment ownership models.

Example:

```text
Python
   |
   v
Generate values
   |
   v
Helm
   |
   v
Kubernetes
```

If ArgoCD manages Helm:

```text
Git
 |
 v
Helm values
 |
 v
ArgoCD
 |
 v
Kubernetes
```

Prefer the second model when GitOps is the established deployment architecture.

---

# 77. Python + Jenkins

Example pipeline:

```text
Jenkins
  |
  +-- Checkout
  |
  +-- Test
  |
  +-- SonarQube
  |
  +-- Trivy
  |
  +-- Build image
  |
  +-- Push image
  |
  +-- Configuration validation
  |
  v
Deployment/Git update
```

Python can perform:

```text
Config validation
Manifest validation
Secret integration
Post-deployment checks
```

---

# 78. Python + GitHub Actions

Example:

```text
GitHub Actions
       |
       +-- Unit tests
       +-- Security scans
       +-- Build
       +-- Push image
       |
       v
Python automation
       |
       v
Kubernetes / Git
```

Use GitHub Actions secrets or an external secret manager for sensitive CI/CD values.

Do not commit them to repository files.

---

# 79. Troubleshooting ConfigMap Issues

### Problem

Application uses old configuration.

Check:

```bash
kubectl get configmap payment-config -n production -o yaml
```

Then check Pod:

```bash
kubectl exec <pod> -n production -- env
```

If configuration is supplied through environment variables, restart the workload after changing the ConfigMap.

Check Deployment:

```bash
kubectl rollout status deployment/payment -n production
```

---

# 80. Troubleshooting Secret Issues

### Problem

Application cannot authenticate to database.

Check:

```bash
kubectl describe pod <pod> -n production
```

Check that the Secret exists:

```bash
kubectl get secret payment-secret -n production
```

Check keys without printing secret values.

Verify:

```text
Secret name
Secret key
Pod reference
Application configuration
Credential validity
Database connectivity
```

Do not expose the password while troubleshooting.

---

# 81. Secret Does Not Exist

Possible causes:

```text
Wrong namespace
Typo in Secret name
ArgoCD resource not synced
External Secret controller failure
Secret deleted
RBAC prevents access
```

Commands:

```bash
kubectl get secret -n production
kubectl get externalsecret -n production
kubectl describe externalsecret <name> -n production
```

The last commands apply when an External Secrets solution is installed.

---

# 82. ConfigMap Does Not Exist

Check:

```bash
kubectl get configmap -n production
```

Then:

```bash
kubectl describe deployment payment -n production
```

Look for:

```text
configMapRef
configMapKeyRef
configMap volume
```

Verify namespace consistency.

A ConfigMap in:

```text
default
```

cannot be directly referenced by a Pod in:

```text
production
```

---

# 83. Wrong Key

ConfigMap:

```yaml
data:
  LOG_LEVEL: INFO
```

Deployment:

```yaml
key: LOGLEVEL
```

This mismatch can cause configuration failure.

Automation should validate referenced keys before deployment.

---

# 84. Secret Key Naming

Use consistent keys:

```text
DB_USERNAME
DB_PASSWORD
DB_HOST
DB_PORT
```

Avoid inconsistent naming:

```text
dbPassword
database_password
PASSWORD_DB
```

unless required by the application.

---

# 85. ConfigMap and Secret Testing

Before production:

```text
Unit tests
Integration tests
Kubernetes API tests
Deployment tests
Application startup tests
Credential connectivity tests
```

A useful test environment:

```text
kind
minikube
EKS development cluster
```

Production credentials should never be used in test clusters.

---

# 86. Example Unit Test Concept

Test the object builder separately from the Kubernetes API.

```python
def test_configmap():
    body = build_configmap()

    assert body.metadata.name == "payment-config"
    assert body.data["APP_ENV"] == "production"
```

This allows fast testing without requiring a cluster.

---

# 87. Production Automation Testing

Use multiple levels:

```text
Level 1
Python unit tests
       |
       v
Level 2
Manifest/object validation
       |
       v
Level 3
Ephemeral Kubernetes cluster
       |
       v
Level 4
Non-production EKS
       |
       v
Level 5
Production
```

Never test destructive Secret automation directly against production first.

---

# 88. Logging and Metrics

Useful automation metrics:

```text
configmap_create_total
configmap_update_total
secret_create_total
secret_rotation_total
automation_failure_total
automation_duration_seconds
```

Do not expose:

```text
secret_value
password
token
private_key
```

as metric labels or values.

---

# 89. Production Architecture with External Secrets

Recommended architecture:

```text
                 Git
                  |
                  v
               ArgoCD
                  |
                  v
             EKS Cluster
                  |
       +----------+----------+
       |                     |
       v                     v
   ConfigMap          ExternalSecret
                             |
                             v
                    External Secrets
                       Controller
                             |
                             v
                   AWS Secrets Manager
                             |
                             v
                       Kubernetes Secret
                             |
                             v
                            Pod
```

This separates:

```text
Configuration management
```

from:

```text
Secret storage
```

---

# 90. Important Security Principle

Kubernetes Secrets are not a replacement for a dedicated enterprise secret-management strategy.

A mature production design considers:

```text
Encryption at rest
RBAC
IAM
Audit logging
Rotation
Access boundaries
Secret lifecycle
External secret management
Backup protection
```

Security is not achieved simply by putting a password inside a Kubernetes Secret.

---

# 91. Interview Questions

## Q1. What is a ConfigMap?

A ConfigMap stores non-sensitive configuration data and allows Pods to consume that configuration as environment variables, individual keys, or mounted files.

---

## Q2. What is a Kubernetes Secret?

A Secret is a Kubernetes resource intended to hold sensitive configuration such as passwords, tokens, certificates, and credentials.

---

## Q3. Is Base64 encryption?

No.

Base64 is an encoding mechanism.

It does not provide confidentiality.

---

## Q4. Which Python API manages ConfigMaps and Secrets?

Both are managed through:

```python
client.CoreV1Api()
```

---

## Q5. What is the difference between stringData and data in a Secret?

`stringData` accepts plaintext strings and the Kubernetes API converts them into the Secret's data representation.

`data` expects Base64-encoded values.

---

## Q6. Should passwords be stored in ConfigMaps?

No.

Sensitive values should be stored in Secrets or, preferably for many production architectures, an external secret manager integrated with Kubernetes.

---

## Q7. Does updating a ConfigMap automatically restart Pods?

No.

A ConfigMap update does not automatically restart Pods.

Environment variables already loaded by a running process will not automatically change.

Mounted ConfigMap files can be updated by Kubernetes, but the application must support reloading if live changes are required.

---

## Q8. How do you make ConfigMap automation idempotent?

Read the ConfigMap first.

If it does not exist, create it.

If it exists, compare the desired configuration and patch the differences.

---

## Q9. How would you rotate a production Secret safely?

I would first make the new credential valid in the backend system, then update the Kubernetes Secret, restart or reload the workload, validate application connectivity, and only then revoke the old credential.

---

## Q10. Why should Secrets not be logged?

Because logs often have broader access and longer retention than the application itself. Logging secret values can cause credential exposure and security incidents.

---

# 92. Scenario-Based Interview Questions

## Scenario 1

### Interviewer

The ConfigMap was updated, but the application still uses the old value.

### Strong Answer

First I would identify how the application consumes the ConfigMap.

If it is using environment variables, the existing process retains the old value, so I would trigger a controlled rollout.

If it is using a mounted ConfigMap file, I would check whether the projected file has updated and whether the application supports configuration reload.

---

## Scenario 2

### Interviewer

A Secret exists, but the application cannot authenticate.

### Strong Answer

I would verify:

```text
Secret name
Secret key
Namespace
Pod reference
Application environment variable
Credential validity
Backend connectivity
```

I would not print the Secret value. I would validate the configuration and connectivity without exposing credentials.

---

## Scenario 3

### Interviewer

The Python script creates a Secret successfully but fails on the second run.

### Strong Answer

The script is probably not idempotent.

The second execution likely receives:

```text
409 Conflict
```

I would implement an ensure pattern:

```text
Read
 |
 +-- 404 -> Create
 |
 +-- Exists -> Update/Patch if required
```

For Secret values, I would avoid logging or exposing plaintext during comparison.

---

## Scenario 4

### Interviewer

A developer wants to store a production database password in Git because ArgoCD needs it.

### Strong Answer

I would not store the plaintext password in Git.

I would use an external secret-management solution such as AWS Secrets Manager integrated through an External Secrets pattern, or another approved encrypted-secret mechanism.

ArgoCD should manage the desired reference/configuration while the sensitive value remains protected.

---

## Scenario 5

### Interviewer

A Secret rotation caused production downtime.

### Strong Answer

I would review the rotation sequence.

A safer process is:

```text
Create new credential
      |
Add new credential to backend
      |
Update Kubernetes Secret
      |
Restart/reload application
      |
Validate connectivity
      |
Revoke old credential
```

If the backend supports dual credentials, I would use an overlap period to avoid an authentication gap.

---

## Scenario 6

### Interviewer

Your Python automation has permission to create Deployments but receives 403 when creating Secrets.

### Strong Answer

I would check the ServiceAccount's Role or ClusterRole and RoleBinding.

A 403 indicates an authorization problem.

I would verify:

```bash
kubectl auth can-i create secrets \
  --as=system:serviceaccount:automation:service-ingress-automation \
  -n production
```

Then I would add only the minimum required Secret permissions.

---

## Scenario 7

### Interviewer

ArgoCD keeps reverting a ConfigMap changed by Python.

### Strong Answer

That indicates GitOps drift.

If ArgoCD is the source of truth, I would not continue making direct changes to the live ConfigMap.

I would update the desired configuration in Git and allow ArgoCD to reconcile it.

---

# 93. Production Checklist

```text
[ ] ConfigMap contains only non-sensitive data
[ ] Secrets are not committed as plaintext
[ ] External secret management evaluated
[ ] Kubernetes authentication configured
[ ] RBAC follows least privilege
[ ] Namespace validated
[ ] Required ConfigMap keys validated
[ ] Required Secret keys validated
[ ] No secret values logged
[ ] No credentials stored in source code
[ ] No credentials written to temporary files
[ ] Idempotent create/update logic implemented
[ ] API errors handled
[ ] Retry strategy bounded
[ ] Configuration changes tested
[ ] Secret rotation procedure tested
[ ] Rollback procedure documented
[ ] Deployment restart behavior understood
[ ] GitOps ownership understood
[ ] ArgoCD drift considered
[ ] Production validation implemented
```

---

# 94. Key Takeaways

The core model is:

```text
ConfigMap
   |
   +-- Non-sensitive configuration

Secret
   |
   +-- Sensitive configuration

Deployment
   |
   +-- Consumes both

Pod
   |
   +-- Runs application
```

Python provides:

```text
CoreV1Api
   |
   +-- ConfigMaps
   +-- Secrets
```

Production automation should provide:

```text
Validation
Idempotency
Security
RBAC
Logging
Retries
Rotation
Rollback
Health checks
GitOps compatibility
```

The most important security principle is:

> **Do not confuse Kubernetes Secret with encryption, and never treat Base64 encoding as security.**

The most important DevOps principle is:

> **Automate configuration consistently, but keep secret values in a proper secret-management system and respect GitOps ownership.**
