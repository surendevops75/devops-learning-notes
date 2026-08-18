# 02 — Docker Automation with Python

## 1. Overview

Docker is one of the most important tools in a modern DevOps workflow.

Python can automate Docker operations such as:

- Building images
- Tagging images
- Running containers
- Inspecting containers
- Reading logs
- Managing networks
- Managing volumes
- Pushing images
- Pulling images
- Cleaning unused resources
- Validating images
- Integrating Docker with CI/CD
- Updating deployment configuration
- Performing pre-deployment checks
- Automating container troubleshooting

The important production principle is:

> **Python should automate Docker consistently while preserving image security, reproducibility, resource controls, and deployment policy.**

A typical workflow is:

```text
Source Code
    |
    v
Python / CI Automation
    |
    v
Docker Build
    |
    v
Security Scan
    |
    v
Image Tag
    |
    v
Registry
    |
    v
Kubernetes / EKS
```

---

# 2. Docker Automation Architecture

A production Docker automation system can look like:

```text
                    Python Automation
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
     Docker Engine     Registry API       CI/CD
          |                |                |
          v                v                v
      Containers      ECR / Registry     Jenkins /
      Images          Metadata           GitHub Actions
          |
          v
      Kubernetes
```

Python may communicate with Docker through:

```text
Docker SDK for Python
Docker CLI
Docker Registry APIs
Cloud provider APIs
CI/CD APIs
```

---

# 3. Why Automate Docker?

Manual:

```text
docker build
docker tag
docker push
docker run
docker logs
docker inspect
```

Automation:

```text
Python
 |
 +-- validate context
 +-- build image
 +-- inspect image
 +-- scan image
 +-- tag image
 +-- push image
 +-- validate registry
 +-- update GitOps
```

Automation becomes valuable when the same Docker workflow is executed repeatedly across:

```text
Developers
CI pipelines
Multiple services
Multiple environments
Multiple repositories
```

---

# 4. Docker Concepts Required

Before automating Docker, understand:

```text
Image
Container
Registry
Repository
Tag
Digest
Dockerfile
Build context
Layer
Volume
Network
Container runtime
```

Basic relationship:

```text
Dockerfile
    |
    v
Image
    |
    v
Container
```

Registry relationship:

```text
Image
   |
   v
Registry
   |
   v
Pull
   |
   v
Container
```

---

# 5. Docker Installation

On systems using Docker Engine, install Docker according to the official package/repository instructions for the operating system.

Verify:

```bash
docker version
```

Check daemon:

```bash
docker info
```

Test:

```bash
docker run hello-world
```

Production environments should use an approved Docker/container runtime installation method rather than ad-hoc scripts.

---

# 6. Python Docker SDK

Install:

```bash
pip install docker
```

Import:

```python
import docker
```

Create a client:

```python
client = docker.from_env()
```

This typically connects using the environment's Docker configuration.

Test:

```python
print(
    client.version()
)
```

---

# 7. Docker SDK Architecture

```text
Python
  |
  v
Docker SDK
  |
  v
Docker Engine API
  |
  v
Docker daemon
  |
  +-- Images
  +-- Containers
  +-- Networks
  +-- Volumes
```

The Docker SDK is generally cleaner than parsing CLI output for structured automation.

---

# 8. Docker CLI vs Docker SDK

Docker SDK is useful for:

```text
Container lifecycle
Image management
Networks
Volumes
Events
Inspection
Structured responses
```

CLI/subprocess is useful when:

```text
A specific CLI feature is required
The organization already standardizes on CLI commands
A Docker plugin/extension is involved
```

Use structured APIs where possible.

---

# 9. Verify Docker Connectivity

```python
import docker


client = docker.from_env()

try:
    client.ping()
    print("Docker daemon reachable")

except docker.errors.DockerException as exc:
    print(
        f"Docker unavailable: {exc}"
    )
```

Production automation should fail clearly if the Docker daemon is unavailable.

---

# 10. Docker Daemon Permissions

On Linux, Docker access may require:

```text
root
```

or membership in the Docker group.

Important security consideration:

> **Access to the Docker daemon is highly privileged.**

A user who can control Docker may effectively gain host-level privileges.

Do not casually grant Docker socket access to untrusted workloads.

---

# 11. Docker Socket

Common Linux socket:

```text
/var/run/docker.sock
```

A containerized automation tool may mount:

```text
/var/run/docker.sock
```

but this has serious security implications.

Architecture:

```text
Python container
      |
      v
Docker socket
      |
      v
Host Docker daemon
```

Treat this as privileged access.

---

# 12. Build Image

Docker CLI:

```bash
docker build -t payment:1.2.0 .
```

Python:

```python
image, logs = client.images.build(
    path=".",
    tag="payment:1.2.0"
)

print(image.id)
```

Build output:

```python
for entry in logs:
    if "stream" in entry:
        print(
            entry["stream"],
            end=""
        )
```

---

# 13. Build Context

Docker build context determines which files are available to the build.

Bad context:

```bash
docker build /
```

This may expose huge amounts of data to the build process.

Better:

```bash
docker build .
```

with a strong:

```text
.dockerignore
```

---

# 14. .dockerignore

Typical entries:

```text
.git
.gitignore
.venv
__pycache__
node_modules
*.log
.env
coverage
tests
```

The exact contents depend on the application.

Benefits:

```text
Smaller context
Faster builds
Less accidental data exposure
Better caching
```

---

# 15. Build Automation Validation

Before building:

```text
Dockerfile exists
Build context exists
.dockerignore exists
Required files exist
No secret files in context
```

Example:

```python
from pathlib import Path


context = Path(".")

if not (
    context / "Dockerfile"
).exists():
    raise RuntimeError(
        "Dockerfile not found"
    )
```

---

# 16. Build Arguments

Docker build can receive arguments.

Python:

```python
image, logs = client.images.build(
    path=".",
    tag="payment:1.2.0",
    buildargs={
        "APP_VERSION": "1.2.0"
    }
)
```

Do not use build arguments for secrets.

Build arguments can be exposed through image/build metadata depending on how they are used.

Use proper secret mechanisms for build-time secrets.

---

# 17. Docker Build Secrets

Sensitive values should not be embedded into:

```text
Dockerfile
Image layers
Build arguments
Source code
```

Use BuildKit secret mechanisms when the build requires secrets.

Architecture:

```text
Secret Manager
      |
      v
BuildKit secret
      |
      v
Build step
      |
      v
Secret not persisted in final image
```

Use your CI/CD platform's secure secret integration.

---

# 18. Image Tags

Common:

```text
payment:1.2.0
payment:2026.08.18
payment:git-abc123
```

Avoid relying only on:

```text
latest
```

because it is mutable and weak for deployment traceability.

---

# 19. Immutable Image References

Strong deployment pattern:

```text
repository@sha256:<digest>
```

instead of:

```text
repository:latest
```

Example:

```text
123456789012.dkr.ecr.ap-south-1.amazonaws.com/payment@sha256:abc...
```

This provides stronger reproducibility.

---

# 20. Image Tagging

```python
image = client.images.get(
    "payment:1.2.0"
)

image.tag(
    "123456789012.dkr.ecr.ap-south-1.amazonaws.com/payment",
    tag="1.2.0"
)
```

Verify:

```python
print(
    image.tags
)
```

---

# 21. Multiple Tags

A release may have:

```text
payment:1.2.0
payment:git-abc123
```

Python:

```python
image.tag(
    repository,
    tag="1.2.0"
)

image.tag(
    repository,
    tag="git-abc123"
)
```

Use tags consistently according to organizational policy.

---

# 22. Image Labels

Labels can provide metadata:

```python
image, logs = client.images.build(
    path=".",
    tag="payment:1.2.0",
    labels={
        "org.opencontainers.image.version": "1.2.0",
        "org.opencontainers.image.revision": "abc123"
    }
)
```

Useful metadata:

```text
Version
Git SHA
Build system
Source repository
Build timestamp
```

Use OCI-standard labels where practical.

---

# 23. Image Inspection

```python
image = client.images.get(
    "payment:1.2.0"
)

print(
    image.attrs["Id"]
)

print(
    image.attrs["RepoTags"]
)
```

Inspect:

```text
Entrypoint
Cmd
Env
Architecture
OS
Labels
Layers
Size
```

---

# 24. Image Size

```python
size = image.attrs[
    "Size"
]

print(
    f"Image size: {size / 1024 / 1024:.2f} MB"
)
```

Large images can cause:

```text
Slow pulls
Long deployments
Higher registry/storage cost
Longer startup
```

---

# 25. Docker Image Best Practices

Use:

```text
Small base images
Multi-stage builds
.dockerignore
Layer caching
Non-root user
Pinned dependencies
Minimal packages
No secrets
Health checks where appropriate
```

---

# 26. Multi-Stage Build

Example:

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build

WORKDIR /app

COPY pom.xml .
RUN mvn dependency:go-offline

COPY src ./src

RUN mvn package -DskipTests


FROM eclipse-temurin:17-jre

WORKDIR /app

COPY --from=build \
     /app/target/app.jar \
     app.jar

USER 10001

ENTRYPOINT ["java", "-jar", "app.jar"]
```

Python can automate validation/building of this image.

---

# 27. Container Run

Python:

```python
container = client.containers.run(
    "payment:1.2.0",
    detach=True,
    name="payment-test"
)

print(
    container.id
)
```

Production automation should specify:

```text
Resource limits
Network
Environment
Volumes
Security options
Restart policy
```

---

# 28. Environment Variables

```python
container = client.containers.run(
    "payment:1.2.0",
    environment={
        "APP_ENV": "test",
        "LOG_LEVEL": "INFO"
    },
    detach=True
)
```

Do not place sensitive values directly in source code.

---

# 29. Environment Secrets

Bad:

```python
environment={
    "DB_PASSWORD": "password123"
}
```

Better:

```text
Secret Manager
     |
     v
CI/CD secret
     |
     v
Runtime injection
```

For Kubernetes workloads, prefer:

```text
Kubernetes Secrets
External Secrets
AWS Secrets Manager
```

according to the architecture.

---

# 30. Container Port Mapping

```python
container = client.containers.run(
    "payment:1.2.0",
    ports={
        "8080/tcp": 8080
    },
    detach=True
)
```

Meaning:

```text
Host 8080
    |
    v
Container 8080
```

---

# 31. Container Names

```python
container = client.containers.run(
    "payment:1.2.0",
    name="payment-test",
    detach=True
)
```

Names must be unique.

For repeated CI jobs, use unique names or remove previous test containers safely.

---

# 32. Container Lifecycle

Typical:

```text
create
  |
  v
start
  |
  v
running
  |
  +-- stop
  |
  +-- restart
  |
  v
remove
```

Python:

```python
container = client.containers.create(
    "payment:1.2.0"
)

container.start()

# tests

container.stop()
container.remove()
```

Using explicit lifecycle control is useful for test automation.

---

# 33. Automatic Cleanup

Use:

```python
try:
    container.start()

    # test logic

finally:
    container.remove(
        force=True
    )
```

But make sure logs and diagnostics are collected before removal.

---

# 34. Container Logs

```python
logs = container.logs(
    stdout=True,
    stderr=True
)

print(
    logs.decode()
)
```

Useful for:

```text
Startup errors
Application crashes
Configuration errors
Dependency failures
```

---

# 35. Follow Container Logs

```python
for line in container.logs(
    stream=True
):
    print(
        line.decode(),
        end=""
    )
```

Be careful with indefinite streams in CI.

Use timeouts or stop conditions.

---

# 36. Container Exit Code

```python
result = container.wait()

print(
    result["StatusCode"]
)
```

Typical:

```text
0 -> success
non-zero -> failure
```

A CI test container should return a meaningful exit code.

---

# 37. Container Inspect

```python
details = container.attrs

print(
    details["State"]
)

print(
    details["Config"]
)
```

Useful fields:

```text
Status
ExitCode
OOMKilled
Error
StartedAt
FinishedAt
```

---

# 38. OOMKilled Detection

```python
state = container.attrs[
    "State"
]

if state.get("OOMKilled"):
    print(
        "Container was OOMKilled"
    )
```

This is important during Docker troubleshooting.

---

# 39. Container Health

If Dockerfile contains:

```dockerfile
HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1
```

Python can inspect:

```python
state = container.attrs[
    "State"
]

health = state.get(
    "Health"
)

print(health)
```

A healthy container should not be assumed solely because its process is running.

---

# 40. Container Resource Limits

CPU and memory limits are important.

Example:

```python
container = client.containers.run(
    "payment:1.2.0",
    mem_limit="512m",
    nano_cpus=500_000_000,
    detach=True
)
```

This roughly constrains:

```text
Memory -> 512 MB
CPU -> 0.5 CPU
```

Always size limits based on observed application behavior.

---

# 41. CPU/Memory Inspection

```python
stats = container.stats(
    stream=False
)

print(
    stats
)
```

The raw Docker stats structure requires interpretation for:

```text
CPU percentage
Memory usage
Memory limit
Network I/O
Block I/O
PIDs
```

---

# 42. Docker Stats Automation

Python can monitor:

```text
CPU
Memory
Network
Block I/O
PIDs
```

and generate:

```text
Threshold alerts
Diagnostic reports
CI test failure conditions
```

Example policy:

```text
Memory > 90%
    |
    v
Fail test
```

Do not confuse Docker-level monitoring with your production Prometheus/Grafana monitoring stack.

---

# 43. Docker Events

Docker provides an event stream.

Python:

```python
for event in client.events(
    decode=True
):
    print(event)
```

Events include:

```text
create
start
stop
die
destroy
pull
push
```

For long-running automation, implement:

```text
Filtering
Cancellation
Reconnect
Backoff
```

---

# 44. Docker Network Automation

List networks:

```python
networks = client.networks.list()

for network in networks:
    print(
        network.name
    )
```

Create:

```python
network = client.networks.create(
    "payment-network"
)
```

---

# 45. Connect Container to Network

```python
network.connect(
    container
)
```

This allows:

```text
Container A
    |
    v
Docker network
    |
    v
Container B
```

For test environments, Python can create isolated networks for integration tests.

---

# 46. Network Isolation

Example:

```text
test-network
 |
 +-- payment
 +-- mongodb
 +-- redis
```

Only containers on that network can communicate according to Docker networking rules.

This is useful for integration testing.

---

# 47. Integration Test Architecture

```text
Python Test Runner
       |
       +-- Docker network
       |
       +-- Application
       |
       +-- Database
       |
       +-- Redis
       |
       v
Integration Tests
```

After tests:

```text
Containers removed
Network removed
Volumes cleaned
```

---

# 48. Docker Volumes

List:

```python
volumes = client.volumes.list()

for volume in volumes:
    print(
        volume.name
    )
```

Create:

```python
volume = client.volumes.create(
    name="payment-data"
)
```

Mount:

```python
client.containers.run(
    "payment:1.2.0",
    volumes={
        "payment-data": {
            "bind": "/data",
            "mode": "rw"
        }
    },
    detach=True
)
```

---

# 49. Volume Safety

Never automatically delete production volumes.

Before cleanup:

```text
Is volume temporary?
Is data persistent?
Who owns it?
Is backup available?
```

Automation should clearly distinguish:

```text
temporary CI volume
```

from:

```text
production persistent data
```

---

# 50. Pull Image

```python
image = client.images.pull(
    "nginx",
    tag="latest"
)
```

For production, prefer immutable versions/digests.

Example:

```text
nginx:1.27
```

or digest:

```text
nginx@sha256:...
```

---

# 51. Registry Authentication

Docker CLI:

```bash
docker login
```

Python SDK can use:

```python
client.login(
    username=username,
    password=password,
    registry=registry
)
```

Do not hardcode credentials.

Prefer:

```text
CI/CD credential store
Cloud identity
Registry-specific authentication
```

---

# 52. Amazon ECR Integration

Your AWS/EKS environment commonly uses:

```text
ECR
```

Architecture:

```text
Python
  |
  v
AWS identity
  |
  v
ECR authentication
  |
  v
Docker
  |
  v
ECR repository
```

Use boto3 for AWS APIs and Docker SDK/CLI for Docker operations.

---

# 53. ECR Authentication

A common CLI workflow:

```bash
aws ecr get-login-password \
  --region ap-south-1 \
  | docker login \
      --username AWS \
      --password-stdin \
      <account>.dkr.ecr.ap-south-1.amazonaws.com
```

Python can automate the equivalent workflow.

Use an IAM role or approved CI credential mechanism instead of static AWS keys.

---

# 54. Boto3 + Docker SDK

Architecture:

```text
Python
 |
 +-- boto3
 |     |
 |     v
 |   AWS ECR
 |
 +-- Docker SDK
       |
       v
     Docker
```

Boto3 handles:

```text
ECR repositories
Images
AWS metadata
IAM-integrated operations
```

Docker SDK handles:

```text
Build
Tag
Push
Pull
Container
```

---

# 55. ECR Login Through Boto3

Conceptually:

```python
import boto3
import docker


ecr = boto3.client(
    "ecr",
    region_name="ap-south-1"
)

auth = ecr.get_authorization_token()

client = docker.from_env()
```

The authorization response contains encoded registry credentials.

Decode and pass them securely to the Docker client.

Do not log the decoded credential.

---

# 56. ECR Push Architecture

```text
Source
 |
 v
Docker Build
 |
 v
Local Image
 |
 v
Security Scan
 |
 v
ECR Tag
 |
 v
Docker Push
 |
 v
ECR
```

Python can orchestrate this sequence.

---

# 57. Image Push

```python
for line in client.images.push(
    repository,
    tag="1.2.0",
    stream=True,
    decode=True
):
    print(line)
```

Monitor for errors in the streamed response.

A push operation should not be considered successful merely because the API call returned an iterable.

---

# 58. Verify Push

After push:

```text
ECR image exists
+
Expected tag exists
+
Digest captured
```

Use boto3:

```python
response = ecr.describe_images(
    repositoryName="payment",
    imageIds=[
        {
            "imageTag": "1.2.0"
        }
    ]
)
```

Capture:

```text
imageDigest
imagePushedAt
imageTags
```

---

# 59. Tag vs Digest

Suppose:

```text
payment:1.2.0
```

points to:

```text
sha256:abc...
```

The tag can potentially be moved.

The digest identifies the immutable image content.

Production deployments should prefer digest-based verification when possible.

---

# 60. Image Promotion

A safe image promotion pattern:

```text
Build
 |
 v
ECR
 |
 v
Scan
 |
 v
Record Digest
 |
 v
Promote
 |
 v
GitOps
 |
 v
EKS
```

Python can verify that the promoted tag maps to the expected digest before updating Git.

---

# 61. Docker Security Scanning

Your DevSecOps stack includes:

```text
Trivy
SonarQube
Veracode
```

For container images:

```text
Docker build
     |
     v
Trivy image scan
     |
     v
Pass / Fail
```

Example:

```bash
trivy image payment:1.2.0
```

Python can invoke Trivy through subprocess.

---

# 62. Trivy Automation

```python
import subprocess


result = subprocess.run(
    [
        "trivy",
        "image",
        "--exit-code",
        "1",
        "payment:1.2.0"
    ],
    text=True
)

if result.returncode != 0:
    raise RuntimeError(
        "Image security scan failed"
    )
```

Use the organization's actual severity and ignore-policy configuration.

---

# 63. Image Vulnerability Policy

A production policy might be:

```text
CRITICAL -> block
HIGH -> block or exception
MEDIUM -> report
LOW -> report
```

The exact policy should be organization-specific.

Python should consume the scanner's exit code/report rather than inventing security policy.

---

# 64. SBOM Automation

Modern container workflows may generate:

```text
Software Bill of Materials
```

Tools can produce:

```text
SPDX
CycloneDX
```

Architecture:

```text
Docker image
    |
    v
SBOM generator
    |
    v
SBOM
    |
    v
Artifact repository
```

Python can orchestrate generation and publication.

---

# 65. Dockerfile Validation

Python can verify:

```text
Dockerfile exists
FROM exists
USER configured
No obvious secrets
No unnecessary package installation
Expected exposed port
Expected entrypoint
```

This is policy validation, not a replacement for a full Dockerfile security scanner.

---

# 66. Non-Root Validation

Inspect image configuration:

```python
config = image.attrs[
    "Config"
]

print(
    config.get("User")
)
```

If empty, the image may run as the default container user.

For production security, prefer a dedicated non-root user where supported by the application.

---

# 67. Entrypoint Validation

```python
entrypoint = config.get(
    "Entrypoint"
)

cmd = config.get(
    "Cmd"
)

print(
    entrypoint,
    cmd
)
```

A Python validation tool can enforce expected runtime behavior.

---

# 68. Environment Validation

Inspect:

```python
environment = config.get(
    "Env",
    []
)

for item in environment:
    print(item)
```

Never blindly print image environment variables in production logs because they may contain sensitive data.

Instead, validate against forbidden keys/patterns.

---

# 69. Secret Detection in Images

A container image can accidentally contain:

```text
.env
credentials
private keys
AWS credentials
API tokens
```

Use:

```text
Trivy
Gitleaks
Dockerfile review
Filesystem scanning
```

before release.

---

# 70. Image Layer Awareness

Docker images consist of layers:

```text
Base layer
    +
Dependency layer
    +
Application layer
```

Bad Dockerfile ordering can reduce cache efficiency.

Example:

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

Any source change may invalidate dependency installation.

Better:

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
```

This improves caching.

---

# 71. Python Build Automation and Cache

Python can invoke builds with:

```python
client.images.build(
    path=".",
    tag=image_tag
)
```

Build performance depends on:

```text
Dockerfile order
Build context
.dockerignore
Cache availability
Dependency layers
Base image
```

Python does not magically make Docker builds faster; it should orchestrate a well-designed Docker build.

---

# 72. Build Failure Troubleshooting

Common causes:

```text
Invalid Dockerfile
Missing build file
Dependency failure
Network failure
Registry issue
Permission issue
Disk full
Memory pressure
Build context too large
```

Python should capture:

```text
Docker error
Build logs
Command/context
Image tag
Build duration
```

---

# 73. Docker Build Error Handling

```python
try:
    image, logs = client.images.build(
        path=".",
        tag=image_tag
    )

except docker.errors.BuildError as exc:
    for log in exc.build_log:
        print(log)

    raise
```

Do not suppress build failures.

---

# 74. Docker API Errors

Catch:

```python
docker.errors.DockerException
```

and more specific exceptions such as:

```text
BuildError
APIError
ImageNotFound
ContainerError
NotFound
```

Example:

```python
try:
    image = client.images.get(
        image_tag
    )

except docker.errors.ImageNotFound:
    print("Image not found")
```

Specific exceptions produce better troubleshooting.

---

# 75. Container Startup Troubleshooting

If container exits:

```text
docker ps -a
```

Python:

```python
container = client.containers.get(
    "payment-test"
)

print(
    container.status
)

print(
    container.logs().decode()
)
```

Then inspect:

```text
ExitCode
Error
OOMKilled
Health
Environment
Mounts
Network
```

---

# 76. Common Container Failure

### Exit code 1

Possible:

```text
Application configuration
Startup exception
Missing dependency
Invalid environment
```

Check:

```text
logs
environment
entrypoint
command
```

---

# 77. OOMKilled

If:

```python
container.attrs[
    "State"
]["OOMKilled"]
```

is true:

```text
Memory limit was exceeded.
```

Investigate:

```text
Container memory limit
Application memory usage
Heap configuration
Traffic
Memory leak
```

Do not simply increase memory without understanding the cause.

---

# 78. Restart Loop

A container can repeatedly:

```text
Start
Crash
Restart
```

Check:

```text
ExitCode
Logs
Healthcheck
RestartPolicy
Environment
Dependencies
```

Python can collect this diagnostic information automatically.

---

# 79. Port Conflict

Error may occur when:

```text
Host port already in use
```

Inspect:

```bash
docker ps
```

or:

```python
containers = client.containers.list(
    all=True
)
```

A CI test framework should avoid hard-coded host ports when parallel jobs are possible.

---

# 80. Dynamic Port Allocation

Instead of:

```python
ports={
    "8080/tcp": 8080
}
```

use:

```python
ports={
    "8080/tcp": None
}
```

when the SDK/runtime supports dynamic host-port allocation for the workflow.

Then inspect:

```python
container.attrs[
    "NetworkSettings"
]["Ports"]
```

This is useful for parallel integration tests.

---

# 81. Docker Network Troubleshooting

Check:

```python
container.attrs[
    "NetworkSettings"
]
```

Inspect:

```text
Networks
IP addresses
Aliases
Ports
```

For multi-container testing, verify:

```text
Same network
Correct service name
Correct port
Application listening address
```

---

# 82. Container Listening Address

A common issue:

```text
Application listens on 127.0.0.1
```

inside the container.

Then external container traffic may fail.

For many web applications, the application should listen on:

```text
0.0.0.0
```

inside the container.

Python can validate the container configuration but the application itself must be configured correctly.

---

# 83. Docker DNS

Containers on a user-defined Docker network can communicate using:

```text
container/service name
```

Example:

```text
payment
mongodb
redis
```

Python integration tests should use service names rather than hard-coded container IP addresses.

---

# 84. Docker Volume Troubleshooting

Check:

```python
print(
    container.attrs["Mounts"]
)
```

Investigate:

```text
Source
Destination
RW
Type
```

Common problems:

```text
Wrong path
Permission denied
Read-only mount
Missing volume
Unexpected persistent data
```

---

# 85. Permission Troubleshooting

If application cannot write:

```text
Permission denied
```

Check:

```text
Container user
File ownership
Volume ownership
Filesystem permissions
SELinux/security policy
```

Do not solve every permission problem with:

```text
chmod 777
```

Use the least privilege required.

---

# 86. Docker Cleanup Automation

List stopped containers:

```python
containers = client.containers.list(
    all=True
)

for container in containers:
    if container.status == "exited":
        print(container.name)
```

Cleanup only resources known to belong to the automation.

---

# 87. Label-Based Cleanup

Add labels:

```python
container = client.containers.run(
    image,
    labels={
        "automation": "python-tests",
        "owner": "ci"
    },
    detach=True
)
```

Then find:

```python
containers = client.containers.list(
    all=True,
    filters={
        "label": "automation=python-tests"
    }
)
```

This is much safer than deleting every stopped container.

---

# 88. Image Cleanup

Do not automatically run:

```bash
docker system prune -a
```

on production hosts.

It can delete resources required by other workloads.

Instead:

```text
Identify owned images
Check age
Check references
Apply retention policy
Delete only approved resources
```

---

# 89. CI Runner Cleanup

On ephemeral CI runners:

```text
Build
 |
 v
Test
 |
 v
Push
 |
 v
Cleanup
```

Cleanup can safely remove:

```text
Temporary containers
Temporary networks
Temporary volumes
Unneeded test images
```

On shared runners, use ownership labels and strict cleanup boundaries.

---

# 90. Docker Image Retention

A registry should have:

```text
Retention policy
Lifecycle rules
```

ECR supports lifecycle policies.

Python can inspect or manage registry metadata through boto3, but lifecycle policy should generally be centrally managed.

---

# 91. Docker + Jenkins Architecture

```text
Jenkins
   |
   v
Python script
   |
   v
Docker daemon
   |
   +-- Build
   +-- Scan
   +-- Test
   +-- Tag
   +-- Push
   |
   v
ECR
```

Jenkins should provide:

```text
Credentials
Environment
Workspace
Build metadata
```

Python should handle:

```text
Docker workflow
Validation
Error handling
```

---

# 92. Docker + GitHub Actions

```text
GitHub Actions
      |
      v
Python
      |
      v
Docker
      |
      v
Trivy
      |
      v
ECR
```

The workflow can pass:

```text
Git SHA
Version
Registry
Repository
Environment
```

to Python.

---

# 93. Image Tag from Git SHA

```python
import os

sha = os.environ[
    "GITHUB_SHA"
]

short_sha = sha[:12]

tag = (
    f"git-{short_sha}"
)
```

This gives a traceable tag.

Example:

```text
payment:git-a1b2c3d4e5f6
```

Use the CI provider's exact environment variables.

---

# 94. Image Tag from Jenkins Build

Jenkins can provide:

```text
BUILD_NUMBER
GIT_COMMIT
JOB_NAME
```

Python can combine:

```text
application version
+
Git SHA
+
build number
```

Example:

```text
payment:1.2.0-abc123
```

The exact tagging policy should be standardized across the organization.

---

# 95. Image Traceability

A production image should answer:

```text
Which source commit built this?
Which pipeline built this?
Which version is it?
Which repository produced it?
```

Use:

```text
Git SHA
OCI labels
Build metadata
SBOM
Registry digest
```

This makes production incidents easier to investigate.

---

# 96. Docker Image Promotion

Avoid rebuilding the same source for each environment:

```text
Build once
      |
      v
Scan
      |
      v
Promote same digest
      |
      +-- Dev
      +-- Staging
      +-- Production
```

This provides stronger artifact consistency.

---

# 97. Python Promotion Validation

Before promotion:

```python
expected_digest = "sha256:..."

actual = ecr.describe_images(
    repositoryName="payment",
    imageIds=[
        {
            "imageTag": "1.2.0"
        }
    ]
)

digest = actual[
    "imageDetails"
][0]["imageDigest"]

if digest != expected_digest:
    raise RuntimeError(
        "Digest mismatch"
    )
```

This prevents promoting an unexpected image.

---

# 98. Docker Image Metadata

OCI metadata can include:

```text
org.opencontainers.image.source
org.opencontainers.image.revision
org.opencontainers.image.version
org.opencontainers.image.created
```

Python can enforce that required metadata exists.

---

# 99. Image Policy Automation

Example:

```text
Image
 |
 +-- Non-root?
 +-- Required labels?
 +-- No secrets?
 +-- Vulnerability scan passed?
 +-- Expected architecture?
 +-- Expected digest?
 +-- Size within policy?
 |
 v
Release approved
```

This is a useful platform engineering capability.

---

# 100. Multi-Architecture Images

Modern environments may require:

```text
linux/amd64
linux/arm64
```

Example architecture:

```text
Source
 |
 v
Buildx
 |
 +-- amd64
 +-- arm64
 |
 v
Multi-platform manifest
 |
 v
Registry
```

Python can orchestrate the build command where Docker SDK support does not provide the required Buildx workflow cleanly.

---

# 101. Architecture Validation

Before deployment:

```python
image.attrs.get(
    "Architecture"
)
```

For multi-platform images, inspect the registry manifest/index rather than relying only on one locally loaded image.

This matters when EKS node groups use different CPU architectures.

---

# 102. Docker Buildx

For multi-platform builds:

```bash
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --push \
  -t <registry>/payment:1.2.0 .
```

Python can invoke Buildx through `subprocess` when required.

Use argument lists and validate all externally supplied values.

---

# 103. Docker Build Cache

Production CI can use:

```text
Registry cache
Local cache
BuildKit cache
```

The goal is:

```text
Fast builds
+
Reproducible output
```

Do not allow untrusted cache sources to compromise supply-chain security.

---

# 104. Dependency Caching

For Python applications:

```dockerfile
COPY requirements.txt .

RUN pip install \
    --no-cache-dir \
    -r requirements.txt

COPY . .
```

For Node:

```dockerfile
COPY package*.json .
RUN npm ci
COPY . .
```

For Java:

```dockerfile
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
```

The correct ordering improves Docker layer reuse.

---

# 105. Docker Build Reproducibility

Reproducible builds require control over:

```text
Base image
Dependencies
Build context
Build arguments
Source revision
Build tooling
```

Avoid:

```dockerfile
FROM ubuntu:latest
```

when strict reproducibility is required.

Prefer controlled versions/digests.

---

# 106. Base Image Validation

Python can inspect:

```text
Dockerfile FROM
```

and enforce approved base images.

Example policy:

```text
Approved:
company/base-java
company/base-python
```

Reject:

```text
Unknown external base image
```

unless approved.

---

# 107. Supply Chain Security

A production Docker pipeline should consider:

```text
Trusted base images
Dependency scanning
Image scanning
SBOM
Provenance
Signed artifacts
Immutable references
Least privilege
```

Python can orchestrate these controls but should not replace specialized security tools.

---

# 108. Docker Image Signing

Organizations may use:

```text
Cosign
Sigstore
```

for image signing.

Architecture:

```text
Build
 |
 v
Scan
 |
 v
Sign
 |
 v
Registry
 |
 v
Verify
 |
 v
Deploy
```

Python can invoke signing/verification tooling through controlled subprocess calls.

---

# 109. Deployment Verification

After image push:

```text
Registry image exists
        |
        v
Digest correct
        |
        v
GitOps updated
        |
        v
ArgoCD sync
        |
        v
EKS rollout
        |
        v
Pods healthy
```

Python can participate in pre-deployment and post-deployment validation.

---

# 110. Docker Automation vs Kubernetes Automation

Do not mix responsibilities.

Docker automation:

```text
Image
Container
Network
Volume
Registry
```

Kubernetes automation:

```text
Pod
Deployment
Service
Ingress
ConfigMap
Secret
EKS
```

Your previous Python Kubernetes section covered the second layer.

This section focuses on the container/image lifecycle.

---

# 111. Docker + Kubernetes Boundary

```text
Python
 |
 +-- Docker SDK
 |     |
 |     v
 |   Image
 |
 +-- Kubernetes Client
       |
       v
     Deployment
```

Typical workflow:

```text
Docker build
   |
   v
ECR
   |
   v
Kubernetes deployment
```

---

# 112. Production Troubleshooting Flow

If deployment fails after an image update:

```text
1. Verify Git commit
2. Verify image tag
3. Verify ECR digest
4. Verify ArgoCD sync
5. Check Kubernetes Deployment
6. Check Pod events
7. Check container status
8. Check container logs
9. Check image architecture
10. Check configuration/secrets
```

This avoids assuming every failure is a Docker problem.

---

# 113. ImagePullBackOff

If Kubernetes reports:

```text
ImagePullBackOff
```

check:

```text
Image repository
Image tag
Image digest
ECR permissions
Node IAM/workload configuration
Network
Registry availability
```

Python Docker automation can validate the image exists before changing GitOps configuration.

---

# 114. Wrong Image Tag

Example:

```text
GitOps:
payment:1.2.0

ECR:
payment:1.1.0
```

Deployment fails.

Prevent this with:

```text
ECR verification
+
Git update
```

workflow:

```text
Push image
 |
 v
Verify tag/digest
 |
 v
Update Git
```

---

# 115. Wrong Architecture

Suppose:

```text
EKS node -> arm64
Image -> amd64 only
```

The pod may fail to run correctly.

Python can validate:

```text
Build architecture
Registry manifest
Node architecture
```

before promotion.

---

# 116. Container Crash After Deployment

Possible:

```text
Application bug
Missing environment
Wrong port
Wrong command
Dependency unavailable
Permission issue
Architecture mismatch
Memory issue
```

Troubleshooting:

```text
kubectl describe pod
kubectl logs
kubectl logs --previous
```

Then inspect the image:

```text
ENTRYPOINT
CMD
USER
ENV
```

Python can automate collection of these diagnostics when integrated with the Kubernetes client.

---

# 117. Docker Troubleshooting Matrix

| Problem | Likely Area | First Check |
|---|---|---|
| Build fails | Dockerfile/dependency | Build logs |
| Image not found | Local/registry | Image tag |
| Push fails | Auth/network | Registry credentials |
| Container exits | Application/config | Logs |
| OOMKilled | Resources | Memory stats |
| Port conflict | Host runtime | Port mappings |
| Permission denied | User/volume | UID/GID |
| Slow build | Context/cache | `.dockerignore` |
| Slow pull | Image size/network | Image size |
| EKS ImagePullBackOff | ECR/K8s | Image + permissions |
| Wrong platform | Architecture | Manifest |
| Secret exposed | Build/source | Scan image |

---

# 118. Production Docker Automation Checklist

```text
[ ] Docker runtime available
[ ] Python environment isolated
[ ] Docker SDK version pinned/tested
[ ] Docker daemon access controlled
[ ] Docker socket treated as privileged
[ ] Dockerfile validated
[ ] .dockerignore configured
[ ] Build context minimized
[ ] No secrets in build context
[ ] Build args do not contain secrets
[ ] Build secrets handled securely
[ ] Image tags traceable
[ ] Digests captured
[ ] OCI metadata included
[ ] Non-root runtime considered
[ ] Resource limits configured
[ ] Health checks considered
[ ] Image scanned
[ ] SBOM generated where required
[ ] Base image approved
[ ] Dependencies controlled
[ ] Registry authentication secure
[ ] ECR image verified
[ ] Push failures handled
[ ] Container cleanup scoped
[ ] Volumes protected
[ ] Networks isolated
[ ] Logs captured
[ ] OOMKilled checked
[ ] Architecture verified
[ ] GitOps update validated
[ ] ArgoCD ownership respected
[ ] Production branch controls respected
[ ] Audit logs generated
```

---

# 119. Interview Questions

## Q1. How can Python automate Docker?

I can use the Docker SDK for Python to build images, inspect images, create and run containers, manage networks and volumes, collect logs, inspect container state, and push or pull images.

For Docker features not exposed conveniently through the SDK, I can use the Docker CLI through secure subprocess execution.

---

## Q2. Docker SDK vs Docker CLI?

The Docker SDK provides structured Python objects and is convenient for programmatic workflows.

The CLI is useful when I need a specific Docker command or Buildx feature.

I choose based on reliability, maintainability, and the feature required.

---

## Q3. How would you automate an image build and push?

I would:

```text
Validate Dockerfile
Check build context
Build image
Tag with immutable/traceable version
Run security scan
Push to ECR
Verify image digest
Return the digest to the pipeline
```

Then the deployment system can reference the verified artifact.

---

## Q4. Why avoid `latest`?

`latest` is mutable.

A deployment using `latest` may run different image contents at different times.

I prefer:

```text
version tag
Git SHA tag
digest
```

and ideally use immutable digest references for production deployments.

---

## Q5. How do you handle Docker build failures?

I capture Docker build logs, classify the failure, return a non-zero pipeline result, and preserve enough diagnostic information to troubleshoot.

I do not hide the exception or continue to the push stage.

---

## Q6. How do you detect an OOMKilled container?

I inspect the container's state:

```python
container.attrs["State"]["OOMKilled"]
```

If it is true, I investigate memory limits, actual memory usage, application behavior, traffic, and possible leaks.

---

## Q7. How do you safely clean Docker resources?

I never blindly run global cleanup on a shared or production host.

I label resources created by the automation and remove only resources owned by that workflow.

---

## Q8. How do you authenticate Python automation with ECR?

I use AWS identity through boto3, preferably an IAM role or CI/CD identity, obtain ECR authorization securely, authenticate Docker, push the image, and then verify the resulting ECR image digest.

I do not hardcode AWS credentials.

---

## Q9. How do you prevent secrets from entering a Docker image?

I use:

```text
.dockerignore
BuildKit secrets
Secret scanning
Trivy
Secure runtime injection
```

I avoid putting secrets in Dockerfiles, build arguments, source code, or image environment variables.

---

## Q10. How would you integrate Docker automation with ArgoCD?

Python would build and scan the image, push it to ECR, verify the digest, and update the GitOps repository with the approved image reference.

ArgoCD would then detect the Git change and deploy it to EKS.

---

# 120. Scenario-Based Interview Questions

## Scenario 1 — Build Works but Push Fails

### Interviewer

Your Python script builds the Docker image successfully, but ECR push fails.

### Strong Answer

I would check:

```text
AWS identity
ECR permissions
Repository existence
Registry URL
ECR authentication
Network connectivity
Docker daemon
```

I would not rebuild the image unnecessarily if the local image is already valid.

After fixing authentication, I would push the existing image and verify the digest.

---

## Scenario 2 — Image Exists but Kubernetes Cannot Pull It

### Strong Answer

I would verify:

```text
Repository
Tag/digest
ECR region
ECR permissions
Node IAM configuration
Network connectivity
Image architecture
```

Then I would check:

```bash
kubectl describe pod
```

for the exact image-pull event.

---

## Scenario 3 — Image Is 1.5 GB

### Strong Answer

I would investigate the Dockerfile and apply:

```text
Smaller base image
Multi-stage build
.dockerignore
Layer optimization
Dependency cleanup
Package cleanup
```

I would not simply accept the large image because it increases pull time and deployment overhead.

---

## Scenario 4 — Container Immediately Exits

### Strong Answer

I would inspect:

```text
Container logs
Exit code
ENTRYPOINT
CMD
Environment
Mounted files
Dependencies
```

I would determine whether the failure is an application error, configuration problem, or container runtime issue.

---

## Scenario 5 — Container Is OOMKilled

### Strong Answer

I would verify the configured memory limit and inspect actual memory usage.

Then I would investigate:

```text
Application heap
Memory leak
Traffic pattern
Dependency behavior
```

I would avoid blindly increasing memory because that may hide the underlying issue.

---

## Scenario 6 — Two CI Jobs Use Docker on the Same Host

### Strong Answer

I would isolate resources using:

```text
Unique container names
Unique networks
Labels
Temporary volumes
Separate workspaces
Controlled cleanup
```

For larger workloads, I would prefer ephemeral CI runners to reduce cross-job contamination.

---

## Scenario 7 — Docker Socket Mounted into a Python Container

### Interviewer

Is this safe?

### Strong Answer

Mounting `/var/run/docker.sock` gives the container access to the host Docker daemon and can effectively provide highly privileged host access.

I would treat it as a security boundary and prefer alternatives such as:

```text
Ephemeral builders
Rootless/buildless approaches where appropriate
Dedicated build infrastructure
BuildKit/build services
```

depending on the platform architecture.

---

## Scenario 8 — Image Tag Was Reused

### Interviewer

A deployment uses `payment:1.2.0`, but the tag was overwritten with a different image.

### Strong Answer

This is a reproducibility problem.

I would capture and deploy the image digest rather than relying solely on the mutable tag.

I would also enforce an image-tag immutability policy in the registry where appropriate.

---

## Scenario 9 — Python Updates GitOps Before ECR Push Completes

### Strong Answer

That ordering is incorrect.

The safe sequence is:

```text
Build
 |
 v
Scan
 |
 v
Push
 |
 v
Verify ECR digest
 |
 v
Update GitOps
```

Git should reference an artifact that is already available and verified.

---

## Scenario 10 — Production Deployment Uses a Different Image Than Expected

### Strong Answer

I would trace:

```text
Git commit
 -> GitOps revision
 -> ArgoCD application revision
 -> Kubernetes Deployment image
 -> ECR tag/digest
```

If tags are mutable, I would compare the actual image digest and move toward immutable digest-based deployment.

---

# 121. Senior-Level Docker Automation Thinking

A junior question is:

```text
How do I run docker build from Python?
```

A senior-level question is:

```text
How do I build, secure, verify,
promote, deploy, observe,
and troubleshoot the exact same artifact?
```

The production mental model is:

```text
Source
 |
 v
Build
 |
 v
Scan
 |
 v
SBOM
 |
 v
Sign
 |
 v
Registry
 |
 v
Digest verification
 |
 v
GitOps
 |
 v
ArgoCD
 |
 v
EKS
 |
 v
Runtime verification
```

Python can orchestrate the workflow, while specialized tools remain responsible for:

```text
Docker -> container lifecycle
Trivy -> vulnerability scanning
ECR -> artifact registry
ArgoCD -> GitOps reconciliation
EKS -> Kubernetes runtime
Prometheus/Grafana -> metrics
ELK -> logs
```

---

# 122. Final Mental Model

Remember Docker automation as five layers:

```text
Layer 1 — Build
    |
    +-- Dockerfile
    +-- Build context
    +-- Cache
    +-- Multi-stage
    |
    v
Layer 2 — Secure
    |
    +-- Trivy
    +-- SBOM
    +-- Secrets
    +-- Base image
    |
    v
Layer 3 — Publish
    |
    +-- Tag
    +-- Digest
    +-- ECR
    |
    v
Layer 4 — Deploy
    |
    +-- GitOps
    +-- ArgoCD
    +-- EKS
    |
    v
Layer 5 — Operate
    |
    +-- Logs
    +-- Metrics
    +-- Health
    +-- Troubleshooting
```

The key production principle is:

> **Build once, verify the artifact, promote the same artifact, deploy through the controlled GitOps path, and keep enough metadata to trace every production container back to its source commit.**
