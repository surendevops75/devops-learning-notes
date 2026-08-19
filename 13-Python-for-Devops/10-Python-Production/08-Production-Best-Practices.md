# 10-Python-Production
# 08 — Production Best Practices

## 1. Introduction

Production Python automation is different from a script that only works on a developer laptop.

A production-grade DevOps automation system must be:

```text
correct
safe
observable
testable
repeatable
idempotent
secure
maintainable
performant
operationally predictable
```

For DevOps automation, production quality means the script can safely interact with:

```text
AWS
EKS
Kubernetes
Terraform
Git
Jenkins
GitHub Actions
ArgoCD
Docker
Linux
APIs
databases
CI/CD systems
```

without becoming a new source of operational risk.

---

# 2. Production Engineering Mindset

The first question should not be:

```text
"How do I make the script work?"
```

It should be:

```text
"What happens when this script fails?"
```

Then ask:

```text
What if AWS is unavailable?
What if Kubernetes is slow?
What if the network fails?
What if the process is killed?
What if the same task runs twice?
What if configuration is wrong?
What if credentials expire?
What if an API returns 429?
What if a deployment partially succeeds?
```

Production design starts with failure.

---

# 3. Production Quality Model

A useful model:

```text
Correctness
   +
Reliability
   +
Security
   +
Observability
   +
Performance
   +
Maintainability
   +
Recoverability
=
Production Quality
```

Optimizing only one dimension is not enough.

---

# 4. Principle: Keep It Simple

Production automation should avoid unnecessary complexity.

Prefer:

```text
simple architecture
clear functions
explicit dependencies
small modules
```

over:

```text
clever abstractions
hidden behavior
global state
deep inheritance
unnecessary frameworks
```

Simple systems are easier to operate.

---

# 5. Principle: Explicit Over Implicit

Prefer:

```python
deploy(
    config=config,
    client=client,
)
```

over:

```python
deploy()
```

where `deploy()` silently discovers global state.

Explicit dependencies improve:

```text
testing
debugging
maintenance
review
```

---

# 6. Principle: Fail Fast

If required configuration is missing:

```text
fail immediately
```

Do not continue until:

```text
AWS call
Kubernetes mutation
Git commit
deployment
```

and fail later.

---

# 7. Principle: Fail Safely

Failure should not create additional damage.

Example:

```text
deployment failed
```

should not automatically trigger:

```text
destructive cleanup
```

unless that behavior is explicitly designed and safe.

---

# 8. Principle: Fail Closed

For security-sensitive decisions:

```text
uncertain identity
      |
      v
STOP
```

not:

```text
uncertain identity
      |
      v
use default target
```

This is critical for AWS and Kubernetes automation.

---

# 9. Principle: Least Privilege

Python automation should have only the permissions it needs.

Avoid:

```text
AdministratorAccess
```

when the automation only requires:

```text
read EC2
read EKS
update specific resources
```

---

# 10. IAM for Automation

Use:

```text
IAM role
+
least privilege
+
short-lived credentials
```

where possible.

Avoid embedding long-lived AWS access keys in Python code.

---

# 11. EKS Workload Identity

For Python running inside EKS:

```text
Python Pod
   |
   v
AWS workload identity
   |
   v
IAM Role
   |
   v
AWS API
```

This avoids static credentials.

---

# 12. CI/CD Identity

For CI/CD:

```text
Jenkins/GitHub Actions
        |
        v
OIDC / role assumption
        |
        v
temporary AWS credentials
```

Prefer temporary identity over stored long-lived keys.

---

# 13. Secret Management

Secrets should come from:

```text
AWS Secrets Manager
SSM Parameter Store
CI/CD secret store
Kubernetes Secret
external secret-management system
```

depending on the architecture.

Never hard-code production credentials.

---

# 14. Never Log Secrets

Avoid:

```python
logger.info("token=%s", token)
```

Also avoid:

```python
logger.debug("config=%s", config)
```

if `config` contains secrets.

---

# 15. Redaction

Sensitive values should appear as:

```text
API_TOKEN=****
DB_PASSWORD=****
```

not actual values.

---

# 16. Secure Exception Handling

Do not include credentials in:

```text
exception messages
tracebacks
HTTP debug output
subprocess output
```

when sensitive values could be embedded.

---

# 17. Input Validation

Validate:

```text
CLI arguments
environment variables
configuration files
API payloads
repository names
paths
URLs
resource names
environment names
```

before use.

---

# 18. Allowlist Over Blocklist

For privileged automation:

```text
allowed environments
allowed AWS accounts
allowed regions
allowed clusters
allowed repositories
```

are generally safer than trying to blacklist dangerous values.

---

# 19. Production Target Validation

Before mutation:

```text
environment
    |
    v
expected AWS account
    |
    v
expected region
    |
    v
expected EKS cluster
    |
    v
expected namespace
```

All should match.

---

# 20. Production Guard

Example concept:

```python
if config.environment == "production":
    verify_production_target()
```

Production should have stronger controls than development.

---

# 21. Dry Run

Support:

```text
--dry-run
```

where appropriate.

Dry run should show:

```text
what would change
```

without actually mutating infrastructure.

---

# 22. Dry Run Must Be Honest

A dangerous implementation is:

```text
dry-run
```

but the script still:

```text
creates resources
updates Git
changes Kubernetes
```

Clearly separate:

```text
read operations
```

from:

```text
mutation operations
```

---

# 23. Idempotency

Production automation should be safe to retry.

Prefer:

```text
ensure deployment exists
ensure replicas=3
ensure tag=v2
```

over:

```text
create deployment
increment replicas
```

when repeated execution could cause unwanted effects.

---

# 24. Desired State

A powerful model:

```text
current state
     |
     v
desired state
     |
     v
reconcile
```

This aligns naturally with:

```text
Terraform
Kubernetes
GitOps
ArgoCD
```

---

# 25. Reconciliation

Instead of:

```text
do action once
```

use:

```text
inspect
compare
change only if required
verify
```

This is more resilient to retries and partial failures.

---

# 26. Verify After Mutation

Do not assume:

```python
kubectl apply
```

means:

```text
application is healthy
```

After mutation:

```text
apply
 |
 v
observe
 |
 v
verify desired state
 |
 v
verify health
```

---

# 27. AWS Verification

After AWS mutation verify:

```text
resource exists
expected status
expected tags
expected configuration
```

---

# 28. Kubernetes Verification

After deployment verify:

```text
Deployment
ReplicaSet
Pods
Ready condition
Service
Ingress
events
```

depending on the operation.

---

# 29. Deployment Verification

A production deployment should have:

```text
rollout timeout
readiness check
failure detection
rollback strategy
```

Do not wait forever.

---

# 30. Timeouts Everywhere

External operations should have bounded time.

Examples:

```text
HTTP timeout
AWS API timeout
Kubernetes timeout
subprocess timeout
database timeout
deployment timeout
queue visibility timeout
```

---

# 31. Connect Timeout vs Read Timeout

For HTTP:

```text
connect timeout
```

controls connection establishment.

```text
read timeout
```

controls waiting for response data.

Both may need separate values.

---

# 32. Retry Only Transient Failures

Good retry candidates may include:

```text
429
temporary network failure
some 5xx responses
temporary AWS throttling
```

Do not blindly retry:

```text
invalid configuration
authentication failure
authorization failure
malformed request
```

---

# 33. Exponential Backoff

Concept:

```text
1s
2s
4s
8s
16s
```

with a maximum.

This reduces pressure during outages.

---

# 34. Jitter

Without jitter:

```text
100 workers
    |
    v
all retry at 4 seconds
```

With jitter:

```text
worker A -> 3.4s
worker B -> 4.1s
worker C -> 4.7s
```

This reduces synchronized retry bursts.

---

# 35. Retry Budget

Use:

```text
maximum attempts
maximum delay
possibly global retry budget
```

Avoid infinite retries.

---

# 36. Circuit Breaker

If a dependency repeatedly fails:

```text
CLOSED
  |
  v
OPEN
  |
  v
HALF-OPEN
```

This protects worker capacity and downstream services.

---

# 37. Backpressure

Do not allow:

```text
unlimited incoming tasks
```

to become:

```text
unlimited memory
```

Use:

```text
bounded queues
bounded concurrency
rate limits
load shedding
```

---

# 38. Concurrency Limits

Define:

```text
AWS workers
Kubernetes workers
Git workers
HTTP connections
database connections
```

based on actual dependency capacity.

---

# 39. Global vs Local Concurrency

A local semaphore:

```text
10 workers per pod
```

does not mean:

```text
10 workers globally
```

With 10 pods:

```text
10 × 10 = 100
```

Design distributed concurrency intentionally.

---

# 40. Rate Limiting

Concurrency controls:

```text
active operations
```

Rate limiting controls:

```text
operations per time
```

Production APIs may require both.

---

# 41. API Quotas

Before increasing concurrency, understand:

```text
AWS API quotas
Kubernetes API capacity
GitHub API limits
database capacity
HTTP service limits
```

---

# 42. Connection Pooling

Reuse connections where supported.

Avoid:

```text
new client for every request
```

when connection reuse is appropriate.

But respect each library's thread-safety and lifecycle guarantees.

---

# 43. Resource Cleanup

Use:

```text
context managers
finally blocks
client close methods
temporary file cleanup
subprocess cleanup
```

when resources require explicit lifecycle management.

---

# 44. Context Managers

Example:

```python
with open(path) as file:
    data = file.read()
```

The resource is closed automatically.

---

# 45. Temporary Files

Use temporary-file facilities rather than predictable filenames when possible.

Ensure:

```text
permissions
cleanup
secret handling
```

are appropriate.

---

# 46. Subprocess Safety

Prefer:

```python
subprocess.run(
    ["kubectl", "get", "pods"],
    check=True,
)
```

over:

```python
subprocess.run(
    "kubectl get pods",
    shell=True,
)
```

when shell interpretation is unnecessary.

---

# 47. Command Injection

Never concatenate untrusted values into shell commands.

Bad:

```python
command = f"kubectl get {resource}"
```

Better:

```python
subprocess.run(
    ["kubectl", "get", resource],
    check=True,
)
```

Validate the resource too.

---

# 48. Shell Environment

Child processes inherit environment variables.

Pass only required values when sensitive environments are involved.

---

# 49. File Permissions

Sensitive configuration files should not be world-readable.

Use the least permissions required by the process.

---

# 50. Atomic Writes

For important configuration:

```text
write temporary file
       |
       v
flush/close
       |
       v
atomic rename
```

This avoids partially written files being consumed.

---

# 51. Logging Levels

Typical levels:

```text
DEBUG
INFO
WARNING
ERROR
CRITICAL
```

Production default is commonly:

```text
INFO
```

with DEBUG enabled only when necessary.

---

# 52. Structured Logging

Prefer fields:

```text
timestamp
level
service
environment
run_id
task_id
resource
operation
status
duration
```

This improves ELK searches.

---

# 53. Correlation ID

Every automation run should ideally have:

```text
run_id
```

Example:

```text
run_id=deploy-20260818-001
```

---

# 54. Task ID

For concurrent workflows:

```text
run_id
task_id
```

allow operators to trace individual work.

---

# 55. Metrics

Track:

```text
tasks_total
tasks_failed_total
task_duration_seconds
retry_total
throttle_total
queue_depth
active_workers
```

---

# 56. Prometheus Metric Design

Avoid high-cardinality labels.

Bad:

```text
task_id=<unique UUID>
```

Better:

```text
operation=deploy
environment=production
status=success
```

---

# 57. Grafana

Useful dashboards:

```text
task throughput
error rate
P95/P99 duration
queue depth
worker utilization
retry rate
API throttling
```

---

# 58. ELK

Use ELK for:

```text
centralized logs
search
correlation
failure analysis
audit investigation
```

Include structured fields consistently.

---

# 59. Alerting

Alert on symptoms such as:

```text
high failure rate
queue age
API throttling
deployment failures
worker saturation
memory growth
```

Avoid alerting on every individual transient error.

---

# 60. Health Checks

A long-running Python service can expose:

```text
liveness
readiness
```

separately.

---

# 61. Liveness

Liveness answers:

```text
Is the process alive?
```

Do not make liveness depend on every external dependency.

Otherwise a temporary AWS outage could cause Kubernetes to restart every pod.

---

# 62. Readiness

Readiness answers:

```text
Can this instance safely receive work?
```

Readiness can consider important dependencies when appropriate.

---

# 63. Startup Checks

At startup validate:

```text
configuration
credentials/identity
required dependencies
```

But avoid startup checks that unnecessarily prevent recovery from temporary dependencies.

---

# 64. Graceful Shutdown

Handle:

```text
SIGTERM
SIGINT
```

Flow:

```text
signal
  |
  v
stop accepting work
  |
  v
finish safe in-flight work
  |
  v
cleanup
  |
  v
exit
```

---

# 65. Kubernetes Termination

Kubernetes generally sends:

```text
SIGTERM
```

then waits for the termination grace period.

If the process does not exit:

```text
SIGKILL
```

may follow.

---

# 66. Shutdown Deadline

Do not allow graceful shutdown to wait forever.

Use:

```text
shutdown deadline
```

and define what happens to incomplete tasks.

---

# 67. Configuration Immutability

Load configuration once per run:

```text
load
 |
 v
validate
 |
 v
snapshot
```

Then pass it explicitly.

This prevents hidden runtime changes.

---

# 68. Dependency Injection

Prefer:

```python
def deploy(config, aws_client, k8s_client):
    ...
```

over hidden globals.

Benefits:

```text
testing
clarity
mocking
reusability
```

---

# 69. Small Functions

Prefer functions with one clear responsibility:

```text
load_config()
validate_target()
create_client()
deploy()
verify()
report()
```

Avoid giant functions containing the entire deployment workflow.

---

# 70. Single Responsibility

A function should not simultaneously:

```text
read YAML
create AWS client
deploy Kubernetes
send Slack message
write audit log
```

Separate responsibilities.

---

# 71. Separation of Concerns

Architecture:

```text
CLI
 |
 v
Configuration
 |
 v
Domain Logic
 |
 +--> AWS Adapter
 +--> Kubernetes Adapter
 +--> Git Adapter
 |
 v
Verification
 |
 v
Reporting
```

---

# 72. Adapter Pattern

Hide vendor/client-specific details behind functions or classes.

Example:

```text
EKSClient
GitClient
TerraformRunner
ArgoCDClient
```

The core logic should not depend on every low-level implementation detail.

---

# 73. Avoid Over-Abstraction

Do not create:

```text
BaseUniversalCloudResourceManagerFactory
```

for a small script.

Use abstractions when they solve an actual maintenance problem.

---

# 74. Type Hints

Use type hints:

```python
def deploy(
    config: Config,
    cluster: str,
) -> DeploymentResult:
    ...
```

Benefits:

```text
readability
IDE support
static analysis
maintenance
```

---

# 75. Return Structured Results

Instead of:

```python
return True
```

consider:

```python
DeploymentResult(
    success=True,
    resource="orders",
    version="v3",
)
```

Structured results improve reporting.

---

# 76. Explicit Exit Status

CLI automation should return:

```text
0 success
non-zero failure
```

so CI/CD can reliably detect failures.

---

# 77. Configuration Errors vs Runtime Errors

Distinguish:

```text
configuration failure
```

from:

```text
AWS API failure
```

and:

```text
deployment verification failure
```

Different failures require different responses.

---

# 78. Error Taxonomy

Example:

```text
ConfigError
AuthenticationError
AuthorizationError
ValidationError
TransientDependencyError
DeploymentError
VerificationError
```

A clear taxonomy improves retry behavior.

---

# 79. Don't Catch Everything

Avoid:

```python
try:
    ...
except Exception:
    pass
```

This hides production failures.

---

# 80. Catch at the Right Layer

Low-level code should add context.

Higher-level orchestration should decide:

```text
retry?
fail?
continue?
rollback?
```

---

# 81. Preserve Root Cause

Use exception chaining:

```python
raise DeploymentError(
    "Deployment failed"
) from exc
```

This preserves the original cause.

---

# 82. Retry Context

Log:

```text
operation
attempt
max_attempts
error type
delay
```

without exposing secrets.

---

# 83. Partial Failure Reporting

For batch automation:

```text
Total: 100
Success: 94
Failed: 5
Skipped: 1
```

This is more useful than:

```text
job failed
```

---

# 84. Deterministic Behavior

Production automation should produce predictable results for the same:

```text
input
configuration
target state
```

Avoid hidden randomness except where intentionally used for jitter.

---

# 85. Time and Timezones

Use timezone-aware timestamps for operational records.

Avoid ambiguous local times in distributed systems.

Prefer UTC internally for logs/metrics where appropriate.

---

# 86. Unique Run IDs

Generate a unique identifier for each run.

Example:

```text
run_id=20260818-1118-a8f2
```

Do not use sensitive information in IDs.

---

# 87. Correlating CI/CD Runs

Include:

```text
Jenkins build number
GitHub Actions run ID
commit SHA
deployment ID
```

where appropriate.

---

# 88. Git Commit as Deployment Identity

For GitOps:

```text
commit SHA
```

is a strong deployment reference.

Log:

```text
commit
environment
config version
```

---

# 89. Reproducibility

A production run should ideally be reproducible from:

```text
code version
configuration version
dependency versions
input
target environment
```

---

# 90. Dependency Pinning

Pin important dependencies:

```text
requirements.txt
constraints.txt
poetry/uv configuration
```

according to the team's packaging strategy.

Avoid uncontrolled dependency upgrades in production.

---

# 91. Dependency Updates

Update dependencies through:

```text
PR
tests
security scanning
compatibility checks
deployment
```

not random production upgrades.

---

# 92. Vulnerability Scanning

Scan:

```text
Python dependencies
container images
IaC
source code
```

using the DevSecOps tooling appropriate to the organization.

---

# 93. SBOM

For production software supply chains, consider generating a Software Bill of Materials.

It helps identify:

```text
components
versions
vulnerabilities
```

---

# 94. Virtual Environments

Use isolated Python environments for development and build processes.

This prevents dependency contamination.

---

# 95. Reproducible Builds

Use:

```text
locked dependency versions
immutable build inputs
controlled base images
```

where appropriate.

---

# 96. Dockerfile Best Practices

For Python containers:

```text
small trusted base image
non-root user
dependency installation separated from source where useful
no secrets in image
health checks where appropriate
```

---

# 97. Non-Root Container

Prefer running as a non-root user when the application does not require root.

This reduces impact of application compromise.

---

# 98. Read-Only Root Filesystem

Where compatible:

```yaml
readOnlyRootFilesystem: true
```

This reduces writable attack surface.

---

# 99. Linux Capabilities

Drop unnecessary Linux capabilities.

Apply the principle:

```text
only required privileges
```

---

# 100. Resource Requests and Limits

Python workloads in Kubernetes should have appropriate:

```text
CPU requests
memory requests
CPU limits
memory limits
```

based on observed usage.

---

# 101. OOMKilled

If a Python process is killed due to memory pressure:

```text
inspect memory usage
heap behavior
queue size
worker count
object retention
container limits
```

Do not simply increase memory without understanding the cause.

---

# 102. Memory Leaks

Common causes:

```text
global caches
unbounded lists
unclosed resources
retained task objects
growing queues
```

Use profiling and metrics to investigate.

---

# 103. Streaming

For large data:

```python
for item in stream:
    process(item)
```

instead of:

```python
items = list(stream)
```

This reduces memory pressure.

---

# 104. Pagination

Never assume an API returns all results in one response.

Use pagination:

```text
page 1
page 2
page 3
...
```

and process incrementally.

---

# 105. Server-Side Filtering

Prefer:

```text
API filter
```

over:

```text
download everything
filter locally
```

This reduces:

```text
network
memory
latency
API payload
```

---

# 106. Batch Operations

If an API supports batching:

```text
100 requests
```

may become:

```text
10 batch requests
```

Use batching where semantics and limits allow.

---

# 107. Avoid N+1 API Calls

Bad:

```text
list 100 resources
+
100 individual API calls
```

Look for:

```text
bulk endpoints
batch requests
embedded fields
server-side filters
cached metadata
```

---

# 108. Caching

Cache only data where stale values are acceptable.

Define:

```text
TTL
invalidation
scope
ownership
```

---

# 109. Cache Safety

Do not cache secrets indefinitely.

Do not cache authorization decisions beyond their intended validity.

---

# 110. Performance Baselines

Track:

```text
average duration
P50
P95
P99
throughput
error rate
```

before optimizing.

---

# 111. Optimize Bottlenecks

Do not optimize based on assumptions.

Measure:

```text
CPU
I/O
network
API latency
database
serialization
```

and optimize the actual bottleneck.

---

# 112. Profiling

Useful tools:

```text
cProfile
pstats
timeit
tracemalloc
```

Use profiling to identify expensive code paths.

---

# 113. Benchmark Changes

Before:

```text
P95 = 8 sec
```

After:

```text
P95 = 3 sec
```

Record measurable improvement.

---

# 114. Avoid Premature Optimization

A complex optimization that reduces:

```text
8 sec -> 7 sec
```

but doubles maintenance cost may not be worthwhile.

---

# 115. Concurrency

Use concurrency for:

```text
I/O-heavy independent tasks
```

but keep it bounded.

---

# 116. CPU Parallelism

Use processes for appropriate CPU-heavy work.

Do not expect threads to provide linear CPU scaling in standard CPython.

---

# 117. Thread Safety

When using concurrency:

```text
avoid unsafe shared mutable state
```

Verify client/library thread-safety guarantees.

---

# 118. Queue Backpressure

Use bounded queues.

This prevents:

```text
producer faster than consumer
```

from becoming:

```text
unbounded memory growth
```

---

# 119. Graceful Cancellation

Long-running work should be cancellable.

Especially important for:

```text
Kubernetes pod termination
CI/CD cancellation
operator interruption
deployment timeout
```

---

# 120. Production Signal Handling

Handle:

```text
SIGTERM
SIGINT
```

where appropriate.

Do not perform large blocking cleanup during a tiny shutdown window.

---

# 121. Locking

Use locks only around required critical sections.

Avoid:

```text
global lock
```

around slow network calls unless required.

---

# 122. Deadlock Prevention

Use:

```text
consistent lock ordering
few locks
small critical sections
timeouts where appropriate
```

---

# 123. Distributed Locking

For shared production resources:

```text
Terraform state
deployment environment
GitOps writer
```

use appropriate distributed coordination mechanisms.

---

# 124. Don't Invent Distributed Locks

Do not build a fragile lock using:

```text
random file
local variable
sleep
```

Use a proven coordination mechanism where distributed locking is genuinely required.

---

# 125. Terraform Safety

Python orchestration should:

```text
validate workspace
validate account
validate region
respect state locking
avoid concurrent applies
capture plan/apply result
verify infrastructure
```

---

# 126. Terraform Plan

Prefer:

```text
plan
review/validate
apply
verify
```

for high-risk changes.

Automation should preserve the ability to inspect intended changes where required.

---

# 127. Terraform State

Never casually manipulate:

```text
terraform.tfstate
```

Use Terraform's supported state mechanisms.

---

# 128. Git Safety

Before modifying Git:

```text
verify repository
verify branch
verify working tree
fetch latest
detect conflicts
```

---

# 129. GitOps Safety

If Git is source of truth:

```text
Python
  |
  v
validated Git change
  |
  v
CI
  |
  v
ArgoCD
  |
  v
Kubernetes
```

Avoid competing direct mutation.

---

# 130. ArgoCD Integration

Python can:

```text
update image tag
generate values
create PR
query application status
trigger controlled sync where appropriate
```

but the architecture should preserve GitOps ownership.

---

# 131. Jenkins Integration

Python automation can be used for:

```text
pre-checks
artifact verification
deployment orchestration
post-deployment validation
reporting
```

The script should return reliable exit codes.

---

# 132. GitHub Actions Integration

Python can perform:

```text
API automation
deployment validation
AWS operations
Kubernetes checks
release automation
```

Use workflow secrets and permissions carefully.

---

# 133. DevSecOps Pipeline

A production Python automation workflow can fit into:

```text
Git
 |
 v
CI
 |
 +--> tests
 +--> lint
 +--> SAST
 +--> dependency scan
 +--> secret scan
 |
 v
build
 |
 v
artifact/image
 |
 v
deployment
 |
 v
verification
 |
 v
monitoring
```

---

# 134. Security Gates

Security failures should block deployment when policy requires.

Do not allow:

```text
critical vulnerability
      |
      v
"continue anyway"
```

without explicit approved exception handling.

---

# 135. Auditability

Production automation should record:

```text
who
what
when
where
why
result
```

Example:

```text
actor=ci
operation=deploy
environment=production
commit=abc123
result=success
```

---

# 136. Avoid Sensitive Audit Data

Audit logs should not contain:

```text
password
token
private key
secret value
```

Record references/metadata instead.

---

# 137. Change Ticket Correlation

Where organizational processes require it, include:

```text
change ID
incident ID
ticket ID
PR number
```

in the automation run metadata.

---

# 138. Operational Documentation

Every production automation should document:

```text
purpose
inputs
outputs
permissions
dependencies
configuration
failure modes
rollback
troubleshooting
owner
```

---

# 139. Runbook

A runbook should answer:

```text
What does it do?
How do I run it?
How do I verify it?
What if it fails?
How do I rollback?
Who owns it?
```

---

# 140. README

A production repository should have:

```text
README.md
```

with:

```text
architecture
setup
configuration
usage
examples
troubleshooting
security
```

---

# 141. Examples

Provide safe examples:

```bash
python deploy.py \
  --environment staging \
  --dry-run
```

Avoid examples containing real credentials.

---

# 142. Make Operational Commands Copy-Safe

Commands in documentation should avoid:

```text
production destructive actions
```

without warnings and validation.

---

# 143. Testing Pyramid

Use:

```text
        E2E
       /   \
  Integration
     /     \
    Unit Tests
```

Many fast unit tests, fewer expensive integration/E2E tests.

---

# 144. Unit Tests

Test:

```text
configuration
validation
parsers
retry classification
business logic
result handling
```

---

# 145. Integration Tests

Test:

```text
AWS clients
Kubernetes API
Git
database
HTTP
```

against controlled test environments or appropriate mocks.

---

# 146. End-to-End Tests

Validate:

```text
CI/CD
deployment
verification
rollback
```

as a complete workflow.

---

# 147. Test Failure Paths

Do not test only:

```text
success
```

Test:

```text
timeout
429
500
403
404
invalid config
network failure
partial deployment
```

---

# 148. Mocking

Mock external dependencies in unit tests.

Do not mock everything in integration tests.

The goal is to validate both:

```text
logic
+
real integration
```

at appropriate layers.

---

# 149. Test Isolation

Tests should not depend on:

```text
developer machine state
current AWS profile
current Kubernetes context
local files
```

unless explicitly designed as integration tests.

---

# 150. Test Data

Use:

```text
synthetic data
test accounts
test namespaces
temporary resources
```

instead of production data where possible.

---

# 151. Static Analysis

Use:

```text
ruff
flake8
pylint
mypy
pyright
```

according to team standards.

Static analysis catches many defects before runtime.

---

# 152. Formatting

Use an agreed formatter such as:

```text
Black
Ruff formatter
```

Consistency reduces review noise.

---

# 153. Pre-Commit Checks

Automate:

```text
formatting
lint
tests
secret scanning
```

before code reaches CI.

---

# 154. CI Quality Gate

Example:

```text
lint
 |
 v
type check
 |
 v
unit tests
 |
 v
security scan
 |
 v
build
 |
 v
integration tests
```

---

# 155. Dependency Security

Scan dependencies regularly.

Track:

```text
package
version
CVE
severity
fix version
```

---

# 156. Pinning vs Ranges

For production reproducibility, exact or controlled dependency versions are generally safer than broad ranges.

Use an intentional update process.

---

# 157. Python Version Management

Standardize the supported Python version.

Example:

```text
Python 3.x
```

Document:

```text
minimum version
tested version
production version
```

---

# 158. Runtime Compatibility

Before upgrading Python:

```text
test dependencies
test SDKs
test Kubernetes clients
test CI images
test container base image
```

---

# 159. Logging Performance

Do not build expensive debug strings when debug logging is disabled.

For high-volume systems, structured logging and appropriate levels reduce overhead.

---

# 160. Avoid Logging Every Loop Iteration

Bad:

```text
100000 API calls
=
100000 log lines
```

Prefer:

```text
progress metrics
periodic summaries
sampled debug logs
```

---

# 161. Large Payload Logging

Do not log entire:

```text
Kubernetes manifests
AWS responses
HTTP bodies
```

unless needed.

Large logs increase:

```text
cost
latency
storage
noise
```

---

# 162. Error Sampling

Repeated identical failures may need aggregation rather than millions of identical log events.

Use:

```text
counters
rate-limited logging
summaries
```

---

# 163. Observability Correlation

A production failure should be traceable from:

```text
CI/CD
  |
  v
Python run ID
  |
  v
logs
  |
  v
metrics
  |
  v
AWS/EKS state
```

---

# 164. Monitoring the Automation Itself

Do not monitor only the infrastructure the script manages.

Monitor:

```text
automation health
worker health
queue health
failure rate
execution duration
```

---

# 165. SLO for Automation

Define targets such as:

```text
99% of health checks complete within 60 seconds
95% of deployment validations complete within 2 minutes
```

---

# 166. Error Budget

If automation has an SLO:

```text
allowed failures
```

can guide reliability work.

High failure rates should trigger engineering improvements.

---

# 167. Production Readiness Review

Before production:

```text
security
reliability
performance
observability
testing
rollback
documentation
ownership
```

must be reviewed.

---

# 168. Production Readiness Checklist

```text
[ ] Code reviewed
[ ] Tests passing
[ ] Security scan passing
[ ] Dependencies controlled
[ ] Configuration validated
[ ] Secrets secured
[ ] IAM reviewed
[ ] Timeouts configured
[ ] Retry policy configured
[ ] Concurrency bounded
[ ] Logging implemented
[ ] Metrics implemented
[ ] Alerts defined
[ ] Health checks configured
[ ] Graceful shutdown implemented
[ ] Rollback tested
[ ] Runbook available
[ ] Owner assigned
```

---

# 169. Ownership

Every production automation should have:

```text
technical owner
operational owner
escalation path
```

An ownerless script eventually becomes operational debt.

---

# 170. On-Call Considerations

On-call engineers should know:

```text
what alerts mean
how to verify state
how to stop automation
how to rollback
who to contact
```

---

# 171. Kill Switch

High-risk automation may need a controlled:

```text
disable/pause
```

mechanism.

Examples:

```text
feature flag
CI disable
queue pause
worker scale-down
```

---

# 172. Emergency Stop

An emergency stop should prevent new dangerous operations.

It should not blindly terminate state-mutating operations without understanding consequences.

---

# 173. Blast Radius Control

Limit:

```text
accounts
regions
clusters
namespaces
services
workers
```

per execution.

---

# 174. Progressive Delivery

For risky changes:

```text
1 service
   |
   v
small percentage
   |
   v
observe
   |
   v
expand
```

This reduces blast radius.

---

# 175. Canary Validation

Verify:

```text
error rate
latency
health
business metrics
logs
```

before expanding rollout.

---

# 176. Rollback

Rollback should be:

```text
known
tested
fast enough
observable
```

Do not discover rollback steps during an incident.

---

# 177. Rollback Is Not Always Reverse

Some operations cannot safely be reversed.

Examples:

```text
database migration
data deletion
external API side effect
```

Design forward recovery when rollback is impossible.

---

# 178. Database Migrations

Automation must distinguish:

```text
schema change
application deployment
rollback
```

Some migrations require backward-compatible design.

---

# 179. External Side Effects

If Python calls:

```text
payment API
ticket system
notification
external service
```

a timeout does not necessarily mean the operation did not happen.

Use:

```text
idempotency key
status lookup
deduplication
```

where supported.

---

# 180. Unknown Outcome

Example:

```text
POST request
   |
   v
timeout
```

The result may be:

```text
not executed
or
executed successfully
```

Do not blindly retry non-idempotent operations.

---

# 181. Idempotency Keys

For supported APIs:

```text
idempotency_key=deployment-123
```

allows retries without duplicate side effects.

---

# 182. Transaction Boundaries

Know what can be rolled back atomically.

A workflow across:

```text
AWS
Git
Kubernetes
external API
```

is not one database transaction.

Design for partial failure.

---

# 183. Saga-Like Recovery

A multi-step workflow may use:

```text
step A
step B
step C
```

If C fails:

```text
compensating action
```

may be needed for A/B where safe.

---

# 184. Avoid Over-Automating Recovery

Automatic rollback can sometimes cause more damage.

For high-risk systems:

```text
detect
alert
pause
require controlled recovery
```

may be safer.

---

# 185. Production Change Windows

Some infrastructure changes should run during controlled windows.

Python automation can enforce:

```text
allowed deployment window
```

where organizational policy requires it.

---

# 186. Environment Protection

Production may require:

```text
manual approval
protected branch
change ticket
two-person review
```

Automation should integrate with these controls rather than bypass them.

---

# 187. Separation of Duties

The person who writes code should not necessarily have unrestricted production credentials.

Use:

```text
CI identity
approval
least privilege
audit
```

to separate responsibilities.

---

# 188. Supply Chain Security

Protect:

```text
source code
dependencies
container images
CI runners
credentials
artifacts
```

from tampering.

---

# 189. Artifact Integrity

For production deployment:

```text
build artifact
  |
  v
digest/version
  |
  v
verification
  |
  v
deploy
```

Prefer immutable image digests where appropriate.

---

# 190. Image Tags vs Digests

Tag:

```text
app:latest
```

can move.

Digest:

```text
app@sha256:...
```

identifies a specific image.

Production deployments benefit from immutable references.

---

# 191. Python Automation and ECR

When deploying images:

```text
build
 |
 v
scan
 |
 v
push ECR
 |
 v
record digest
 |
 v
update deployment
 |
 v
verify
```

---

# 192. Kubernetes Deployment Verification

Check:

```text
image digest
available replicas
ready replicas
pod restarts
events
conditions
```

---

# 193. Rollout Health

A deployment can be:

```text
created successfully
```

but still:

```text
unhealthy
```

Always separate:

```text
API mutation success
```

from:

```text
application health success
```

---

# 194. Production Troubleshooting Workflow

Use:

```text
1. Identify run
2. Identify target
3. Check configuration
4. Check logs
5. Check metrics
6. Check dependency health
7. Check recent changes
8. Check resource state
9. Determine failure class
10. Recover safely
11. Verify
12. Document
```

---

# 195. First Rule During Incidents

Do not make multiple unrelated changes simultaneously.

Otherwise you lose causal information.

---

# 196. Recent Change Analysis

Ask:

```text
What changed?
When?
Which configuration?
Which code version?
Which dependency?
Which infrastructure?
```

Recent changes are often high-value investigation points.

---

# 197. Log Correlation

Search by:

```text
run_id
commit_sha
environment
service
resource
time window
```

rather than reading logs randomly.

---

# 198. Metrics Before Logs

Metrics can quickly show:

```text
global impact
error rate
latency
traffic
resource saturation
```

Then logs can explain individual failures.

---

# 199. Dependency Isolation

Determine whether the problem is:

```text
Python application
AWS
Kubernetes
network
database
Git
CI/CD
external API
```

Do not assume the Python process is the root cause.

---

# 200. Production Recovery

After fixing:

```text
verify desired state
verify application health
verify metrics
verify logs
verify no duplicate jobs
```

---

# 201. Post-Incident Review

Document:

```text
impact
timeline
root cause
contributing factors
detection
response
recovery
preventive actions
```

---

# 202. Automation Debt

Production scripts accumulate debt:

```text
old dependencies
hard-coded assumptions
unused flags
manual steps
poor tests
missing monitoring
```

Schedule maintenance.

---

# 203. Refactoring

Refactor when:

```text
failure rate rises
complexity grows
tests become difficult
operations are unclear
performance degrades
security risk increases
```

---

# 204. Backward Compatibility

When changing a production automation interface:

```text
old CLI
old config
old API
```

consider migration paths.

Do not unexpectedly break downstream CI jobs.

---

# 205. Semantic Versioning

For reusable Python packages, consider:

```text
MAJOR
MINOR
PATCH
```

and document breaking changes.

---

# 206. API Contracts

If Python exposes an API:

```text
request schema
response schema
error schema
authentication
versioning
timeouts
```

should be documented.

---

# 207. Health API

A production service can expose:

```text
/health
/ready
```

with safe information.

Do not expose secrets or unnecessary internal details.

---

# 208. API Authentication

Use appropriate:

```text
IAM
JWT
OAuth
mTLS
API keys
```

based on architecture.

Never invent custom cryptography.

---

# 209. TLS

Production HTTP clients should verify TLS certificates unless there is a deliberate, controlled exception.

Avoid:

```python
verify=False
```

as a permanent production workaround.

---

# 210. Certificate Rotation

Plan for:

```text
CA rotation
certificate expiry
private key rotation
```

without emergency manual intervention.

---

# 211. DNS Failure

Production automation should handle transient DNS failures with:

```text
timeout
retry policy
observability
```

but not infinite retries.

---

# 212. Network Failure

Network calls can fail because of:

```text
DNS
proxy
routing
security group
NACL
firewall
TLS
endpoint outage
```

Log enough context to diagnose without exposing secrets.

---

# 213. Kubernetes Network Failure

For EKS automation check:

```text
API endpoint reachability
security groups
network path
DNS
proxy
RBAC
credentials
```

---

# 214. AWS Permission Failure

For:

```text
AccessDenied
```

do not retry endlessly.

Investigate:

```text
IAM role
policy
resource policy
trust policy
condition
account
region
```

---

# 215. Kubernetes RBAC Failure

For:

```text
Forbidden
```

check:

```text
identity
Role
ClusterRole
RoleBinding
ClusterRoleBinding
namespace
verb
resource
```

---

# 216. Configuration vs Authorization

A configuration value cannot grant permissions.

If the identity lacks:

```text
update deployments
```

setting:

```text
ALLOW_DEPLOY=true
```

does not make the operation authorized.

---

# 217. Production Dependency Health

Before a large operation:

```text
AWS
Kubernetes API
Git
artifact repository
database
```

may be checked when appropriate.

Avoid expensive health checks that themselves create load.

---

# 218. Dependency Time Budget

A workflow may have:

```text
total deadline = 10 minutes
```

Do not let:

```text
AWS retry = 5 min
K8s retry = 5 min
Git retry = 5 min
```

consume 15 minutes independently.

Budget time across the workflow.

---

# 219. Deadline Propagation

Pass remaining time to downstream operations.

Concept:

```text
workflow deadline
      |
      v
remaining budget
      |
      +--> AWS timeout
      +--> K8s timeout
      +--> Git timeout
```

This avoids overrunning the overall deadline.

---

# 220. Production Concurrency Budget

Example:

```text
AWS = 10
Kubernetes = 10
Git = 3
HTTP = 20
```

These values should be based on:

```text
dependency capacity
observed latency
quota
memory
CPU
```

---

# 221. Queue Fairness

If multiple teams share automation:

```text
team A
team B
team C
```

consider:

```text
per-team quota
fair scheduling
priority
```

---

# 222. Tenant Isolation

One workload should not consume all workers.

Use:

```text
resource quotas
separate queues
separate worker pools
```

when required.

---

# 223. Load Shedding

When overloaded:

```text
reject/defer low-priority work
```

to protect critical operations.

---

# 224. Graceful Degradation

Examples:

```text
skip optional notification
reduce polling frequency
defer noncritical scans
```

while keeping:

```text
critical deployment validation
```

available.

---

# 225. Production Configuration Limits

Every dangerous configuration should have:

```text
minimum
maximum
allowed values
```

Examples:

```text
workers
retry attempts
timeout
batch size
queue size
```

---

# 226. Secure Defaults

Good:

```text
TLS verification = true
dry_run = true for dangerous local tools
workers = low safe value
retry attempts = bounded
```

Avoid:

```text
verify=False
admin credentials
unlimited retries
unbounded workers
```

---

# 227. Principle of Minimum Surprise

Operators should understand what:

```text
python deploy.py
```

will do.

Avoid hidden behavior such as:

```text
automatic production detection
implicit account switching
silent Git commits
automatic destructive cleanup
```

unless clearly documented and controlled.

---

# 228. CLI UX

Provide:

```text
--help
--version
--dry-run
--environment
--verbose
```

where appropriate.

---

# 229. Helpful `--help`

Document:

```text
required arguments
defaults
examples
dangerous operations
environment requirements
```

---

# 230. Version Information

A production tool should expose:

```bash
python deploy.py --version
```

This helps correlate behavior with code.

---

# 231. Startup Banner

A safe startup message can include:

```text
tool version
environment
target
run ID
dry-run status
```

Do not print credentials.

---

# 232. Progress Reporting

For long-running automation:

```text
Processed 40/100
Success=38
Failed=2
```

is useful.

Do not create excessive logs.

---

# 233. Machine-Readable Output

Support JSON output where CI/CD integration benefits.

Example:

```json
{
  "success": true,
  "environment": "staging",
  "resources": 10
}
```

Never include secrets.

---

# 234. Human vs Machine Output

Consider:

```text
human-readable default
JSON/structured output for automation
```

This improves CI integration.

---

# 235. Exit Code + Output

CI/CD should rely on:

```text
exit code
```

for success/failure.

Output provides diagnostic context.

---

# 236. Production Scheduling

For scheduled automation:

```text
overlap policy
timeout
retry
missed run behavior
```

must be defined.

---

# 237. Kubernetes CronJob

Consider:

```text
concurrencyPolicy
startingDeadlineSeconds
backoffLimit
activeDeadlineSeconds
```

according to the job's behavior.

---

# 238. Jenkins Scheduled Jobs

Ensure overlapping jobs do not mutate the same production environment simultaneously unless intentionally designed.

---

# 239. GitHub Actions Concurrency

Use workflow/environment concurrency controls where deployments must be serialized.

---

# 240. Monitoring Scheduled Automation

Track:

```text
last successful run
last failure
duration
missed runs
queue delay
```

---

# 241. Automation Heartbeat

Long-running workers can expose:

```text
last heartbeat
```

so operators can distinguish:

```text
healthy but idle
```

from:

```text
stuck
```

---

# 242. Stuck Task Detection

Use:

```text
task timeout
heartbeat
lease
watchdog
```

where appropriate.

---

# 243. Lease-Based Work

For distributed workers:

```text
task lease
   |
   v
worker owns task
   |
   v
heartbeat
```

If worker dies:

```text
lease expires
   |
   v
another worker can retry
```

---

# 244. Duplicate Work Tradeoff

A lease-based system may still produce:

```text
duplicate execution
```

if a worker is slow and its lease expires.

Therefore operations should remain idempotent.

---

# 245. Production Job State

Track states such as:

```text
PENDING
RUNNING
SUCCEEDED
FAILED
RETRYING
CANCELLED
EXPIRED
```

This makes workflow recovery easier.

---

# 246. State Machine

Example:

```text
PENDING
   |
   v
RUNNING
 /    \
v      v
SUCCESS FAILED
          |
          v
       RETRYING
          |
          v
       RUNNING
```

Avoid ambiguous states.

---

# 247. Persistent State

If job state must survive process restart, store it in an appropriate durable system rather than only memory.

---

# 248. Exactly-Once Execution

Do not assume distributed execution can guarantee simple exactly-once semantics.

Design around:

```text
at-least-once
+
idempotency
+
deduplication
```

when appropriate.

---

# 249. Recovery After Process Crash

After restart:

```text
discover unfinished work
   |
   v
determine state
   |
   v
resume/retry safely
```

Do not blindly repeat destructive operations.

---

# 250. Checkpointing

For large workflows:

```text
checkpoint after completed stages
```

can avoid repeating expensive work.

---

# 251. Checkpoint Safety

Checkpoints must represent:

```text
verified completion
```

not merely:

```text
API call returned
```

---

# 252. Production State Machine for Deployment

```text
VALIDATE
   |
   v
PLAN
   |
   v
APPROVE
   |
   v
APPLY
   |
   v
VERIFY
   |
   +--> SUCCESS
   |
   +--> RECOVERY
```

---

# 253. Approval Boundaries

Do not place approval after irreversible actions.

Approval should occur before the high-risk mutation.

---

# 254. Change Preview

For infrastructure:

```text
plan/diff
```

helps operators understand:

```text
what will change
```

before applying.

---

# 255. Diff-Based Automation

For configuration:

```text
desired
vs
current
```

calculate a diff.

Only apply required changes.

---

# 256. Avoid Noisy Changes

Do not rewrite configuration files if semantic content did not change.

This reduces:

```text
Git churn
unnecessary deployments
ArgoCD reconciliation
```

---

# 257. Deterministic Serialization

When generating YAML/JSON:

```text
stable ordering
stable formatting
```

reduces unnecessary diffs.

---

# 258. Git Commit Hygiene

Automated commits should include:

```text
clear message
change scope
automation identity
commit reference
```

Avoid:

```text
update
fix
change
```

without context.

---

# 259. Automated PRs

For GitOps automation, a PR can provide:

```text
review
diff
checks
approval
audit
```

before production deployment.

---

# 260. Production Branch Protection

Use:

```text
required checks
review
status gates
protected branches
```

where appropriate.

---

# 261. Release Metadata

Record:

```text
version
commit
image digest
config version
deployment time
actor
```

---

# 262. Deployment Manifest

A useful deployment record:

```text
application=orders
version=3.2.1
image_digest=sha256:...
config_version=42
environment=production
commit=abc123
```

---

# 263. Incident Correlation

When an incident occurs, these identifiers allow:

```text
deployment
configuration
logs
metrics
```

to be connected.

---

# 264. Production Documentation as Code

Keep:

```text
README
runbook
configuration schema
examples
```

version-controlled with the automation.

---

# 265. Architecture Documentation

Document:

```text
components
data flow
dependencies
permissions
failure paths
```

---

# 266. Threat Model

For privileged Python automation ask:

```text
What can an attacker control?
What credentials exist?
What targets can be reached?
What commands can execute?
What happens if config is malicious?
```

---

# 267. SSRF Protection

If the automation fetches configured URLs:

```text
validate scheme
validate host
allowlist destinations
block sensitive internal targets
```

where applicable.

---

# 268. Command Execution Boundary

Treat:

```text
kubectl
terraform
helm
aws
git
docker
```

as privileged operations.

Validate all dynamic arguments.

---

# 269. Repository Trust

Do not automatically execute:

```text
untrusted repository scripts
```

without understanding the trust boundary.

---

# 270. CI Runner Security

Python automation running in CI can often access:

```text
cloud credentials
repositories
deployment systems
```

Protect runners accordingly.

---

# 271. Ephemeral Runners

For sensitive CI workloads, ephemeral runners can reduce persistence of:

```text
credentials
workspace
artifacts
```

---

# 272. Artifact Cleanup

After CI jobs:

```text
temporary credentials
secret files
temporary configs
```

should not remain unnecessarily.

---

# 273. Workspace Isolation

Concurrent jobs should use isolated workspaces when operations modify:

```text
Git repository
Terraform files
generated manifests
```

---

# 274. Environment Isolation

Separate:

```text
dev
staging
production
```

credentials and deployment paths.

---

# 275. Production Credential Isolation

A staging job should not possess production credentials simply because the same Python code can deploy to production.

Use environment-specific permissions.

---

# 276. Approval Enforcement

The Python script should not be the only security control.

Use platform-level:

```text
IAM
branch protection
environment protection
CI approvals
network controls
```

for defense in depth.

---

# 277. Defense in Depth

Production safety should not depend on:

```text
one if statement
```

Use multiple layers:

```text
configuration
+
identity
+
policy
+
approval
+
validation
+
monitoring
```

---

# 278. Production Best Practice: No Magic Values

Avoid:

```python
if account_id == "123456789012":
```

scattered throughout code.

Centralize policy configuration.

---

# 279. Production Best Practice: No Hidden Globals

Avoid:

```python
AWS_CLIENT = boto3.client(...)
```

at import time when it creates lifecycle/test/configuration problems.

Create dependencies deliberately.

---

# 280. Import-Time Side Effects

Avoid performing:

```text
AWS API calls
Kubernetes calls
file writes
```

during module import.

Imports should generally be safe and predictable.

---

# 281. Main Entry Point

Use:

```python
def main() -> int:
    ...

if __name__ == "__main__":
    raise SystemExit(main())
```

This makes CLI behavior testable.

---

# 282. Main Workflow

A clean structure:

```text
main
 |
 +--> parse_args
 +--> load_config
 +--> validate_config
 +--> create_clients
 +--> execute
 +--> verify
 +--> report
 +--> return exit_code
```

---

# 283. Client Lifecycle

Create clients at an intentional scope:

```text
run
workflow
worker
```

depending on library guarantees and connection reuse needs.

---

# 284. Client Configuration

Configure:

```text
timeouts
retries
region
endpoint
connection pool
```

explicitly where the SDK supports it.

---

# 285. SDK Retry Behavior

Cloud SDKs may already implement retry behavior.

Understand:

```text
SDK retries
+
application retries
```

before adding another retry layer.

---

# 286. Double Retry Problem

If SDK does:

```text
3 retries
```

and application does:

```text
3 retries
```

a single operation may create more attempts than expected.

Define total retry behavior.

---

# 287. HTTP Retry Behavior

Do not automatically retry all methods/status codes.

Consider:

```text
idempotency
HTTP method
status code
server behavior
```

---

# 288. Kubernetes Watch

Kubernetes watches can disconnect.

Production watch code should handle:

```text
timeout
disconnect
resource version
reconnect
backoff
```

---

# 289. Polling vs Watch

Use:

```text
watch
```

where supported and appropriate.

Use polling when:

```text
simplicity
reliability
small scale
```

make it preferable.

---

# 290. Long-Running Watches

A watch should not assume:

```text
connection stays forever
```

Implement reconnect behavior.

---

# 291. API Versioning

Use stable Kubernetes API versions and validate compatibility with the cluster version.

---

# 292. Deprecation Awareness

Production automation should not rely on APIs that are already deprecated without a migration plan.

---

# 293. AWS SDK Versioning

Keep SDK versions controlled and test service API behavior after upgrades.

---

# 294. Logging PII

Do not log unnecessary:

```text
personal data
tokens
credentials
request bodies
```

Follow organizational privacy requirements.

---

# 295. Data Minimization

Collect only data required for the automation.

Less data means:

```text
less risk
less memory
less logging
less storage
```

---

# 296. Retention

Define how long automation logs/audit records are retained.

Balance:

```text
troubleshooting
audit
cost
privacy
```

---

# 297. Time Synchronization

Distributed automation relies on correct clocks for:

```text
tokens
TLS
logs
metrics
leases
deadlines
```

Use infrastructure with reliable time synchronization.

---

# 298. Time-Based Credentials

Temporary AWS credentials expire.

Long-running automation should handle expiration safely.

Prefer creating clients/credentials according to supported SDK credential refresh behavior rather than manually storing static credentials.

---

# 299. Credential Refresh

If a workflow runs for hours:

```text
credential expires
```

the application must either:

```text
refresh automatically
```

or:

```text
restart/re-authenticate safely
```

depending on the identity mechanism.

---

# 300. Production Dependency Matrix

Maintain awareness of:

```text
Python version
boto3/botocore
Kubernetes client
Terraform
Helm
kubectl
AWS CLI
Git
Docker
base image
```

Compatibility matters.

---

# 301. Environment Parity

Keep:

```text
development
staging
production
```

as similar as practical.

Differences should be intentional.

---

# 302. Production-Like Testing

Staging should exercise:

```text
same deployment mechanism
same configuration structure
same identity model where practical
same observability
```

---

# 303. Chaos Testing

For critical automation, simulate:

```text
dependency outage
pod termination
network failure
API throttling
credential expiry
```

to verify recovery.

---

# 304. Game Days

Run controlled exercises:

```text
deployment failure
AWS outage
Kubernetes API issue
bad configuration
```

and validate the runbook.

---

# 305. Recovery Time Objective

Define:

```text
How quickly must automation recover?
```

This affects:

```text
retry
timeout
checkpoint
failover
```

design.

---

# 306. Recovery Point Objective

For stateful automation:

```text
How much work can be lost?
```

This affects:

```text
checkpointing
persistent state
queue durability
```

---

# 307. Availability

For critical automation services:

```text
single process
```

may not be enough.

Use:

```text
multiple replicas
durable queue
leader election
health checks
```

where required.

---

# 308. Stateless Workers

Prefer stateless workers when possible:

```text
queue
 |
 v
worker
 |
 v
external durable state
```

This makes scaling and recovery easier.

---

# 309. Durable Queue

For critical jobs, an in-memory queue may lose work if the process dies.

Use a durable queue when job durability matters.

---

# 310. Retry Queue

Separate:

```text
normal queue
retry queue
dead-letter queue
```

where workload complexity justifies it.

---

# 311. Dead-Letter Handling

Dead-letter items should be:

```text
visible
inspectable
recoverable
alerted
```

not silently discarded.

---

# 312. Poison Job Protection

Use:

```text
max attempts
failure classification
dead-letter
manual review
```

---

# 313. Job Deduplication

Use a stable operation identity:

```text
environment + service + commit
```

where appropriate.

---

# 314. Distributed State

Store durable workflow state outside the Python process when multiple workers/instances need to coordinate.

---

# 315. Consistency Model

Know whether the external system provides:

```text
strong consistency
eventual consistency
```

This affects verification and retries.

---

# 316. AWS Eventual Consistency

Some cloud operations may not become visible everywhere immediately.

Verification should account for appropriate propagation delays rather than treating every short-lived absence as permanent failure.

---

# 317. Kubernetes API State

Kubernetes controllers reconcile asynchronously.

After creating/updating a resource:

```text
API accepted
```

does not mean:

```text
controller finished
```

Wait for the appropriate condition.

---

# 318. Controller-Aware Verification

For Deployment:

```text
observedGeneration
availableReplicas
readyReplicas
conditions
```

can be more meaningful than simply checking object existence.

---

# 319. ArgoCD Verification

For GitOps:

```text
Git desired state
 |
 v
ArgoCD
 |
 v
sync status
 |
 v
health status
```

Verify both synchronization and health.

---

# 320. Production Verification Layers

```text
API accepted
    |
    v
resource exists
    |
    v
desired state reached
    |
    v
health condition
    |
    v
business/functional check
```

The deeper the verification, the stronger the confidence.

---

# 321. Business-Level Verification

Infrastructure health does not always mean application success.

Example:

```text
Pod Ready = true
```

but:

```text
API returns 500
```

Functional checks may be required.

---

# 322. Smoke Tests

After deployment:

```text
DNS
HTTP
health endpoint
critical API
```

can be tested.

---

# 323. Canary Metrics

Use:

```text
error rate
latency
request rate
resource saturation
```

to decide whether to expand rollout.

---

# 324. Rollout Timeout

Never wait indefinitely for:

```text
deployment
pod
API
```

Define an upper bound.

---

# 325. Automatic Recovery

Use automatic recovery only when:

```text
failure is well understood
operation is safe
rollback is tested
```

---

# 326. Manual Intervention

Some failures should stop automation and require an operator.

Example:

```text
unexpected production state
```

Prefer:

```text
pause + alert
```

over:

```text
guess + mutate
```

---

# 327. Operator Controls

Provide:

```text
pause
resume
cancel
retry
rollback
status
```

where appropriate.

---

# 328. Status Command

A production CLI can support:

```bash
python deploy.py status --run-id ...
```

to inspect progress.

---

# 329. Explainability

When automation makes a decision, provide enough context:

```text
why skipped
why retried
why blocked
why deployed
```

This helps operators trust the system.

---

# 330. Audit Trail

Record:

```text
decision
reason
actor
timestamp
target
result
```

---

# 331. Production Best Practice: Safe Defaults

Examples:

```text
dry_run=true for dangerous local operations
verify_tls=true
max_retries=3
workers=5
timeout=30
```

Actual defaults should match workload and organizational standards.

---

# 332. Production Best Practice: Explicit Limits

Never allow:

```text
workers=-1
timeout=9999999
retries=1000000
batch_size=100000000
```

without validation.

---

# 333. Production Best Practice: Environment Separation

Keep:

```text
credentials
queues
configuration
permissions
```

separate across environments.

---

# 334. Production Best Practice: Immutable Releases

Prefer:

```text
versioned code
versioned image
versioned configuration
```

over mutable:

```text
latest
```

references.

---

# 335. Production Best Practice: Reproducible Deployment

Record:

```text
source commit
image digest
config version
dependency versions
target
```

---

# 336. Production Best Practice: Small Changes

Prefer:

```text
small PR
small deployment
small blast radius
```

over massive changes.

---

# 337. Production Best Practice: Automation Should Be Boring

The ideal production automation:

```text
predictable
repeatable
observable
safe
```

It should not surprise operators.

---

# 338. Production Best Practice: Design for Failure

Assume:

```text
network fails
API throttles
pod restarts
credentials expire
configuration is wrong
dependency is down
operator cancels
```

Build recovery into the design.

---

# 339. Production Best Practice: Operational Simplicity

If a feature adds:

```text
10% performance
+
200% complexity
```

question whether it belongs in production.

---

# 340. Production Best Practice: Standardize

Create reusable patterns for:

```text
config loading
logging
AWS clients
Kubernetes clients
retry
timeouts
error handling
metrics
```

Avoid copying slightly different implementations across scripts.

---

# 341. Internal Python Library

A DevOps organization may create a shared library:

```text
devops_common/
├── config.py
├── logging.py
├── aws.py
├── kubernetes.py
├── retry.py
├── validation.py
└── metrics.py
```

This reduces duplicated reliability logic.

---

# 342. Shared Library Governance

A shared library needs:

```text
versioning
tests
documentation
backward compatibility
security review
```

---

# 343. Avoid Shared Library Coupling

Do not force every script to depend on a huge internal framework.

Keep modules focused.

---

# 344. Code Review Checklist

Review:

```text
correctness
security
failure handling
retry
timeouts
concurrency
configuration
logging
tests
rollback
```

---

# 345. Security Review Checklist

Ask:

```text
Can input execute commands?
Can configuration select an arbitrary target?
Can secrets leak?
Are permissions excessive?
Can an attacker trigger expensive operations?
Can the tool access internal endpoints?
```

---

# 346. Reliability Review Checklist

Ask:

```text
What if dependency times out?
What if operation succeeds but response is lost?
What if process dies?
What if job runs twice?
What if configuration changes?
What if the API throttles?
```

---

# 347. Performance Review Checklist

Ask:

```text
Are API calls paginated?
Are calls batched?
Are connections reused?
Is concurrency bounded?
Is memory bounded?
Are large payloads streamed?
```

---

# 348. Observability Review Checklist

Ask:

```text
Can we identify a run?
Can we identify a task?
Can we measure duration?
Can we see failure rate?
Can we see queue depth?
Can we correlate logs?
Can we alert?
```

---

# 349. Deployment Review Checklist

Ask:

```text
How is target verified?
How is change previewed?
How is approval enforced?
How is health verified?
How is rollback performed?
```

---

# 350. Production Incident Checklist

```text
[ ] Identify run ID
[ ] Identify target
[ ] Check recent change
[ ] Check configuration
[ ] Check logs
[ ] Check metrics
[ ] Check dependency
[ ] Stop dangerous automation if necessary
[ ] Determine failure class
[ ] Apply smallest safe recovery
[ ] Verify
[ ] Record incident
```

---

# 351. Final Production Architecture

```text
                         Operator / CI
                              |
                              v
                     +----------------+
                     | CLI / API      |
                     +----------------+
                              |
                              v
                     +----------------+
                     | Config Loader  |
                     +----------------+
                              |
                              v
                     +----------------+
                     | Validation     |
                     +----------------+
                              |
                  +-----------+-----------+
                  |           |           |
                  v           v           v
               Identity    Policy     Safety Guard
                  |           |           |
                  +-----------+-----------+
                              |
                              v
                     +----------------+
                     | Domain Logic   |
                     +----------------+
                              |
             +----------------+----------------+
             |                |                |
             v                v                v
         AWS Adapter      K8s Adapter      Git Adapter
             |                |                |
             +----------------+----------------+
                              |
                              v
                     Retry / Timeout /
                  Concurrency / Backoff
                              |
                              v
                         Verification
                              |
                  +-----------+-----------+
                  |                       |
                  v                       v
              Metrics                  Logs
                  |                       |
                  v                       v
             Prometheus                 ELK
                  |                       |
                  +-----------+-----------+
                              |
                              v
                        Alert / Audit
```

---

# 352. Production Python Repository Structure

Recommended structure:

```text
python-devops-tool/
├── src/
│   └── devops_tool/
│       ├── __init__.py
│       ├── cli.py
│       ├── config.py
│       ├── validation.py
│       ├── logging.py
│       ├── aws.py
│       ├── kubernetes.py
│       ├── git.py
│       ├── terraform.py
│       ├── retry.py
│       ├── deployment.py
│       └── models.py
│
├── tests/
│   ├── unit/
│   └── integration/
│
├── config/
│   └── examples/
│
├── docs/
│   └── runbook.md
│
├── Dockerfile
├── pyproject.toml
├── README.md
└── .gitignore
```

---

# 353. Production Workflow

```text
Developer
   |
   v
Git PR
   |
   v
Lint + Type Check
   |
   v
Unit Tests
   |
   v
Security Scans
   |
   v
Build
   |
   v
Integration Tests
   |
   v
Approval
   |
   v
Deployment
   |
   v
Verification
   |
   v
Metrics + Logs
   |
   v
Rollback if required
```

---

# 354. Production Golden Rules

```text
1. Treat automation as production software.
2. Design for failure.
3. Validate before side effects.
4. Fail fast on invalid configuration.
5. Fail closed on target identity uncertainty.
6. Use least-privilege credentials.
7. Never hard-code secrets.
8. Never log secrets.
9. Use managed secret storage.
10. Use workload identity where possible.
11. Make operations idempotent.
12. Use desired-state reconciliation.
13. Add timeouts.
14. Retry only transient failures.
15. Use exponential backoff and jitter.
16. Bound concurrency.
17. Apply backpressure.
18. Respect API quotas.
19. Verify after mutation.
20. Separate API success from application health.
21. Use structured logging.
22. Use correlation IDs.
23. Export useful metrics.
24. Implement graceful shutdown.
25. Keep configuration immutable per run.
26. Keep functions small and explicit.
27. Use dependency injection.
28. Avoid global mutable state.
29. Test failure paths.
30. Pin/control dependencies.
31. Scan dependencies and images.
32. Use immutable release references.
33. Protect CI/CD credentials.
34. Keep production environments isolated.
35. Control blast radius.
36. Provide rollback.
37. Provide a runbook.
38. Assign ownership.
39. Audit production changes.
40. Keep the system simple.
```

---

# 355. Senior Interview — What Makes a Python Script Production-Ready?

Strong answer:

> I look beyond whether the script works. Production readiness means validated configuration, secure identity, least privilege, timeouts, bounded retries, idempotency, observability, safe concurrency, graceful shutdown, testing, dependency control, clear exit codes, verification and rollback. The script should also have an owner and a documented runbook.

---

# 356. Senior Interview — How Would You Productionize a Python AWS Automation Script?

Strong answer:

> I would externalize configuration, use IAM roles or workload identity instead of static credentials, validate the AWS account and region, configure SDK timeouts and retries, make operations idempotent, bound concurrency according to AWS quotas, add structured logs and metrics, implement dry-run where appropriate, verify resource state after mutations, and integrate the tool into CI/CD with tests and security gates.

---

# 357. Senior Interview — How Would You Productionize a Kubernetes Automation Script?

Strong answer:

> I would validate the target cluster and namespace, use Kubernetes RBAC with least privilege, configure API timeouts, handle watch/poll failures, use bounded concurrency, respect resourceVersion conflicts, make mutations idempotent, wait for actual readiness rather than only API acceptance, and expose logs and metrics for operational troubleshooting.

---

# 358. Senior Interview — How Would You Productionize Terraform Automation?

Strong answer:

> I would separate plan and apply, use remote state with locking, validate the target account and workspace, prevent concurrent applies against the same state, capture plan/apply output, return reliable exit codes, protect credentials, and verify infrastructure after apply. Production changes should also pass CI, security and approval gates.

---

# 359. Senior Interview — How Would You Integrate Python With ArgoCD?

Strong answer:

> In a GitOps architecture, I would generally use Python to generate or update the desired configuration in Git, validate it, and let ArgoCD reconcile the cluster. If Python needs to query ArgoCD, I can use its API for sync or health information. I avoid creating a competing direct Kubernetes mutation path for resources managed by ArgoCD.

---

# 360. Senior Interview — How Do You Handle a Request Timeout?

Strong answer:

> First I determine whether the operation is idempotent and whether the timeout means the outcome is unknown. For idempotent operations I can retry with bounded exponential backoff and jitter. For non-idempotent operations I first query the system for the resulting state before retrying, so I don't accidentally duplicate the side effect.

---

# 361. Senior Interview — What Happens if a Worker Dies During a Deployment?

Strong answer:

> I don't assume the deployment failed. The outcome may be unknown. I use a persistent run/task state where required, query the target system, determine the actual state, and resume or retry only if safe. Idempotency and reconciliation are key to recovery.

---

# 362. Senior Interview — How Do You Prevent Retry Storms?

Strong answer:

> I use bounded concurrency, rate limiting, exponential backoff with jitter, maximum retry attempts and sometimes circuit breakers. I also monitor throttling and downstream latency. I avoid layering application retries on top of SDK retries without understanding the total retry behavior.

---

# 363. Senior Interview — How Do You Handle Configuration Safely?

Strong answer:

> I centralize configuration loading, define precedence, validate types and ranges, separate secrets, and create an immutable configuration snapshot per run. For production I verify the actual AWS account, region and Kubernetes cluster before allowing mutation.

---

# 364. Senior Interview — How Do You Secure Python Subprocess Calls?

Strong answer:

> I prefer argument arrays with `subprocess.run()` and `shell=False`, validate dynamic arguments, avoid passing secrets in command-line arguments, set timeouts, capture output carefully, and use explicit environment variables only when required.

---

# 365. Senior Interview — How Do You Monitor Python Automation?

Strong answer:

> I use structured logs with run and task IDs, Prometheus metrics for throughput, failures, latency, queue depth and retries, and Grafana dashboards/alerts. ELK can provide centralized log search and correlation. I also monitor the automation itself rather than only the infrastructure it manages.

---

# 366. Senior Interview — How Do You Handle Partial Failure?

Strong answer:

> I define the workflow's failure semantics first. Independent tasks can report individual results, transient errors can retry within a bounded policy, permanent failures are recorded, and the final workflow status follows the defined success criteria. For multi-system workflows I design explicit recovery or compensation rather than assuming a global transaction.

---

# 367. Senior Interview — What Is the Difference Between API Success and Deployment Success?

Strong answer:

> An API returning success only means the control plane accepted the request. Kubernetes and cloud systems often reconcile asynchronously. I therefore verify the resulting resource state, controller conditions and, where necessary, application-level health before declaring the deployment successful.

---

# 368. Senior Interview — How Do You Reduce Blast Radius?

Strong answer:

> I use environment and target allowlists, least-privilege identity, production approval gates, dry-run, small changes, canary/progressive rollout, bounded concurrency and explicit account/cluster validation. The goal is to make a bad input affect the smallest possible scope.

---

# 369. Senior Interview — How Do You Handle Production Secrets?

Strong answer:

> I keep secrets outside source code and images, use managed secret stores or CI/CD secret mechanisms, prefer short-lived identity, prevent logging, restrict access, and plan for rotation. I also scan repositories and images for accidental credential exposure.

---

# 370. Senior Interview — How Do You Design for Recoverability?

Strong answer:

> I make operations idempotent, track durable state where needed, checkpoint long workflows, preserve run identifiers, implement safe retries, verify actual state after failures, and document rollback or forward-recovery procedures. Recovery should be designed before production deployment.

---

# 371. Senior Interview — What Would You Check Before Deploying a Python DevOps Tool to Production?

Strong answer:

```text
Configuration
Identity
Permissions
Secrets
Dependencies
Tests
Timeouts
Retry policy
Concurrency
Logging
Metrics
Alerts
Target validation
Rollback
Runbook
Ownership
```

Then I would run a controlled staging test and failure-injection scenarios.

---

# 372. Real-World Scenario — Python Deploys to Wrong EKS Cluster

Potential causes:

```text
wrong kubeconfig
wrong context
wrong environment variable
wrong CI credential
wrong cluster mapping
```

Protection:

```text
expected cluster
+
actual cluster verification
+
AWS account validation
+
production approval
```

---

# 373. Real-World Scenario — Deployment API Succeeds but Pods Fail

Workflow:

```text
API mutation success
        |
        v
Deployment created
        |
        v
Pods scheduled
        |
        v
readiness failure
```

Python must report:

```text
deployment accepted
but application verification failed
```

Then investigate:

```text
events
logs
probes
image
configuration
resources
```

---

# 374. Real-World Scenario — API Returns 429

Do not:

```text
increase workers
```

Instead:

```text
reduce concurrency
+
rate limit
+
exponential backoff
+
jitter
+
batch/filter
```

---

# 375. Real-World Scenario — Process OOMKilled

Investigate:

```text
worker count
queue size
large response objects
caches
pagination
memory limits
retained references
```

Use:

```text
tracemalloc
profiling
metrics
```

rather than only increasing memory.

---

# 376. Real-World Scenario — Python Pod Receives SIGTERM

Expected:

```text
stop new tasks
finish safe work
cleanup
exit
```

within the Kubernetes termination grace period.

---

# 377. Real-World Scenario — Duplicate GitOps Commit

Possible cause:

```text
two workers read same state
both generate changes
both push
```

Protection:

```text
single writer
pull/rebase
conflict detection
PR workflow
idempotency
```

---

# 378. Real-World Scenario — Terraform Apply Interrupted

Do not immediately run another apply blindly.

Check:

```text
CI job state
Terraform lock
actual infrastructure state
backend state
```

Then recover using Terraform-supported procedures.

---

# 379. Real-World Scenario — AWS Credentials Expire

For long-running automation:

```text
temporary credentials
        |
        v
refresh mechanism
        |
        v
new credentials
```

Use SDK-supported credential providers rather than embedding long-lived keys.

---

# 380. Real-World Scenario — Configuration Accidentally Sets 1000 Workers

Validation:

```text
requested = 1000
maximum = 50
```

Result:

```text
reject configuration
```

Do not start the worker pool.

---

# 381. Real-World Scenario — Secret Appears in Logs

Immediate response:

```text
stop further exposure
identify affected logs
rotate credential if necessary
remove/expire access
fix redaction
add regression test
```

Treat exposed credentials as compromised according to security policy.

---

# 382. Real-World Scenario — Dependency Is Down

Avoid:

```text
infinite retries
```

Use:

```text
timeout
bounded retry
circuit breaker
queue/defer
alert
```

depending on criticality.

---

# 383. Real-World Scenario — Unknown Deployment Outcome

If:

```text
request timed out
```

do not immediately repeat a non-idempotent operation.

First:

```text
query current state
```

then determine whether another attempt is safe.

---

# 384. Real-World Scenario — Production Config Drift

If GitOps manages the resource:

```text
Git desired
vs
cluster actual
```

Use:

```text
ArgoCD drift detection
```

and restore desired state according to policy.

---

# 385. Real-World Scenario — Large Repository Automation

For thousands of repositories:

```text
pagination
+
bounded concurrency
+
GitHub API rate limiting
+
caching
+
incremental processing
```

Avoid loading every repository into memory at once.

---

# 386. Real-World Scenario — Multi-Account AWS Scan

Architecture:

```text
account list
    |
    v
bounded worker pool
    |
    +--> assume role
    +--> validate account
    +--> scan
    +--> report
```

Use per-account isolation and rate limits.

---

# 387. Real-World Scenario — Multi-Cluster EKS Health Check

Architecture:

```text
cluster inventory
       |
       v
bounded concurrency
       |
       +--> cluster A
       +--> cluster B
       +--> cluster C
       |
       v
health results
```

Each target should be validated before querying.

---

# 388. Real-World Scenario — Production Cleanup Script

Before deleting anything:

```text
environment validation
account validation
cluster validation
resource allowlist
dry-run
diff
approval
delete
verify
```

Deletion automation requires stronger safeguards than read-only automation.

---

# 389. Destructive Operation Guard

For:

```text
delete
destroy
terminate
remove
```

require stronger validation than:

```text
list
describe
get
```

---

# 390. Production Delete Confirmation

A confirmation should identify:

```text
environment
account
region
cluster
resource count
resource names
```

so the operator knows what will be affected.

---

# 391. Safety Token

For highly sensitive operations, require an explicit environment-specific confirmation token or approval mechanism rather than relying only on:

```text
yes
```

---

# 392. Production Automation Levels

A useful maturity model:

```text
Level 1 -> script works
Level 2 -> tested
Level 3 -> observable
Level 4 -> secure/reliable
Level 5 -> recoverable/scalable
```

---

# 393. Level 1 — Functional

```text
script executes
basic error handling
```

---

# 394. Level 2 — Tested

```text
unit tests
integration tests
CI
```

---

# 395. Level 3 — Observable

```text
structured logs
metrics
alerts
run IDs
```

---

# 396. Level 4 — Production Safe

```text
least privilege
timeouts
retry
idempotency
target validation
rollback
```

---

# 397. Level 5 — Production Platform

```text
distributed workers
durable queues
multi-instance recovery
SLOs
capacity management
advanced audit
```

Do not build Level 5 complexity when Level 4 solves the actual problem.

---

# 398. Production Best Practices Summary

```text
Code
  -> simple, typed, modular

Configuration
  -> externalized, validated, immutable

Security
  -> least privilege, secret manager, workload identity

Reliability
  -> timeout, retry, backoff, idempotency

Concurrency
  -> bounded, rate-limited, backpressured

Observability
  -> logs, metrics, alerts, correlation

Deployment
  -> preview, approval, apply, verify

Recovery
  -> rollback, checkpoint, reconciliation

Operations
  -> runbook, ownership, audit

Testing
  -> unit, integration, E2E, failure injection
```

---

# 399. Final Python Production Mental Model

```text
                         PRODUCTION
                              |
        +---------------------+---------------------+
        |                     |                     |
      CODE               CONFIGURATION          SECURITY
        |                     |                     |
     modular               validated            least privilege
     typed                 immutable             secrets
     tested                versioned              identity
        |                     |                     |
        +---------------------+---------------------+
                              |
                              v
                         RELIABILITY
                              |
        +---------------------+---------------------+
        |                     |                     |
     timeout               retry                idempotency
     backoff               jitter               recovery
        |                     |                     |
        +---------------------+---------------------+
                              |
                              v
                         OPERATIONS
                              |
        +---------------------+---------------------+
        |                     |                     |
   observability          deployment             rollback
     logs/metrics         verification           runbook
        |                     |                     |
        +---------------------+---------------------+
                              |
                              v
                       SAFE AUTOMATION
```

---

# 400. The DevOps Standard

A Python automation script should not be considered production-ready merely because:

```text
"It works on my machine."
```

The production standard is:

```text
It works.
+
It fails safely.
+
It can be observed.
+
It can be tested.
+
It can be repeated.
+
It can be recovered.
+
It is secure.
+
It is maintainable.
+
It is operationally understood.
```

The strongest DevOps mindset is:

> **Automate aggressively, but make every production automation safe to run, safe to retry, easy to observe, and possible to recover.**

---

# 401. Final Production Checklist

```text
Architecture
[ ] Clear responsibilities
[ ] Simple design
[ ] Explicit dependencies
[ ] No unnecessary abstraction

Code
[ ] Type hints
[ ] Small functions
[ ] No hidden global state
[ ] Structured results
[ ] Reliable exit codes

Configuration
[ ] Externalized
[ ] Validated
[ ] Safe defaults
[ ] Target verified
[ ] Immutable per run
[ ] Versioned

Security
[ ] Least privilege
[ ] No hard-coded secrets
[ ] Secret manager
[ ] Workload identity
[ ] TLS verification
[ ] Input validation
[ ] Command execution protected

Reliability
[ ] Timeouts
[ ] Retry classification
[ ] Exponential backoff
[ ] Jitter
[ ] Retry limits
[ ] Circuit breaker where needed
[ ] Idempotency
[ ] Recovery strategy

Concurrency
[ ] Bounded workers
[ ] Bounded queues
[ ] Rate limiting
[ ] Backpressure
[ ] Shared state protected
[ ] Graceful shutdown

Observability
[ ] Structured logs
[ ] Run ID
[ ] Task ID
[ ] Metrics
[ ] Dashboards
[ ] Alerts
[ ] Audit trail

Deployment
[ ] Dry-run where appropriate
[ ] Target validation
[ ] Change preview
[ ] Approval
[ ] Apply
[ ] Verify
[ ] Rollback

Testing
[ ] Unit tests
[ ] Integration tests
[ ] E2E tests
[ ] Failure-path tests
[ ] Security tests
[ ] Load tests where needed
[ ] Failure injection where justified

Operations
[ ] README
[ ] Runbook
[ ] Owner
[ ] Escalation path
[ ] Kill switch where needed
[ ] Incident procedure

Supply Chain
[ ] Dependency control
[ ] Vulnerability scanning
[ ] Image scanning
[ ] Immutable artifacts
[ ] Reproducible builds
```

---

# 402. Python Production Section Complete

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
└── 08-Production-Best-Practices.md   ✓
```

## Python Production Section — Complete

The complete section now covers:

```text
Production scripting
Error handling
Retry strategies
Logging
Observability
Security
Performance
Concurrency
Configuration management
Production best practices
```

This completes:

```text
10-Python-Production
```
