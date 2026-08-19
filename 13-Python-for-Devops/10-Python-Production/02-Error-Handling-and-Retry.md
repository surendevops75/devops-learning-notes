# Error Handling and Retry

Production DevOps automation must distinguish between transient failures, permanent failures, and unknown outcomes.

Core lifecycle:

```text
operation
  ↓
failure
  ↓
capture context
  ↓
classify
  ↓
retry / fail / recover
  ↓
verify
  ↓
cleanup
  ↓
report + exit code
```

The goal is not to retry everything. The goal is to recover automatically when safe and fail safely when recovery is unsafe.

# 1. Production Error-Handling Principles

A production script should:

- preserve the root cause
- classify failures
- separate retryable and non-retryable errors
- use bounded retries
- use exponential backoff and jitter where appropriate
- enforce timeouts and deadlines
- verify uncertain outcomes
- clean up resources
- return meaningful exit codes
- provide actionable logs
- never expose secrets

The key rule is:

```text
Retry transient failures.
Stop permanent failures.
Verify uncertain outcomes.
Never blindly repeat production mutations.
```

# 2. Exception Taxonomy

Use domain-specific exceptions instead of treating every error as generic `Exception`.

```python
class AutomationError(Exception):
    pass

class ConfigurationError(AutomationError):
    pass

class AuthenticationError(AutomationError):
    pass

class AuthorizationError(AutomationError):
    pass

class ValidationError(AutomationError):
    pass

class DependencyError(AutomationError):
    pass

class OperationTimeoutError(AutomationError):
    pass

class VerificationError(AutomationError):
    pass

class RetryableError(AutomationError):
    pass
```

This allows the workflow to make explicit decisions.

# 3. Preserve the Root Cause

Use exception chaining:

```python
try:
    call_api()
except TimeoutError as exc:
    raise DependencyError(
        "Deployment API request failed"
    ) from exc
```

The `from exc` relationship preserves the original cause and makes production troubleshooting much easier.

Avoid:

```python
except Exception:
    raise RuntimeError("failed")
```

because it discards useful classification and context.

# 4. Top-Level Error Boundary

Handle final application errors at the CLI boundary:

```python
def main():
    try:
        run()
        return 0
    except AutomationError as exc:
        logger.error("automation failed: %s", exc)
        return 1
    except Exception:
        logger.exception("unexpected automation failure")
        return 1

if __name__ == "__main__":
    raise SystemExit(main())
```

CI/CD systems depend on the exit code. A failed deployment must not return `0`.

# 5. Error Context

Every production failure should provide safe context such as:

```text
operation
environment
service/resource
AWS account or region when appropriate
EKS cluster
namespace
run_id
attempt
category
elapsed time
```

Example:

```text
operation=deployment
environment=staging
service=orders
run_id=7812
attempt=3
category=TIMEOUT
```

Never include passwords, tokens, private keys, authorization headers, or secret payloads.

# 6. Error Classification

A useful classification is:

```text
INPUT
CONFIGURATION
AUTHENTICATION
AUTHORIZATION
NETWORK
TIMEOUT
RATE_LIMIT
DEPENDENCY
CONFLICT
NOT_FOUND
VALIDATION
VERIFICATION
UNKNOWN
```

The classification should be stable enough for logs, metrics, dashboards, and CI policy.

# 7. Retryability Matrix

| Failure | Usually retry? | Reason |
|---|---:|---|
| Temporary timeout | Yes | May be transient |
| HTTP 429 | Yes | Rate limiting |
| HTTP 500/502/503/504 | Sometimes/Yes | Server-side transient failure |
| HTTP 400 | No | Invalid request |
| HTTP 401 | No, unless credential refresh is supported | Authentication |
| HTTP 403 | No | Authorization |
| Invalid configuration | No | Deterministic |
| Wrong AWS account | No | Safety violation |
| Wrong EKS cluster | No | Safety violation |
| Rollout timeout | Inspect first | Current state may already have changed |
| Conflict | Depends | Refresh/reconcile may be required |
| Unknown mutation outcome | Verify first | Retry may duplicate the action |

The exact policy depends on the API contract and operation semantics.

# 8. Retry Is Not Error Handling

This is dangerous:

```python
for _ in range(100):
    try:
        deploy()
        break
    except Exception:
        time.sleep(1)
```

It has:

- unlimited semantic patience
- no classification
- fixed delay
- no jitter
- no timeout budget
- no idempotency analysis
- no cancellation handling

A production retry loop must be deliberate.

# 9. Bounded Retries

Define both:

```text
maximum attempts
maximum elapsed time
```

For example:

```python
MAX_ATTEMPTS = 3
MAX_ELAPSED_SECONDS = 120
```

Attempts and retries are different:

```text
attempt 1 = initial call
attempt 2 = retry 1
attempt 3 = retry 2
```

# 10. Exponential Backoff

A common schedule is:

```text
1s
2s
4s
8s
16s
```

Formula:

```text
delay = base_delay × 2^(attempt - 1)
```

Always cap it:

```python
delay = min(delay, max_delay)
```

# 11. Jitter

Without jitter:

```text
100 workers fail
 ↓
all wait 4 seconds
 ↓
100 workers retry together
```

This can create a thundering herd.

Full jitter can be:

```python
import random

delay = min(
    max_delay,
    base_delay * (2 ** (attempt - 1))
)

sleep_time = random.uniform(0, delay)
```

Jitter spreads retry traffic across time.

# 12. Retry Budget

A robust policy includes:

```text
max attempts
max backoff
max elapsed time
```

A deadline can be created with:

```python
deadline = time.monotonic() + 120
```

Before every retry:

```python
remaining = deadline - time.monotonic()

if remaining <= 0:
    raise OperationTimeoutError(
        "Retry budget exhausted"
    )
```

Use `time.monotonic()` for elapsed-time calculations because wall-clock adjustments should not affect the deadline.

# 13. Generic Retry Pattern

Conceptual implementation:

```python
def retry(operation, max_attempts=3):
    for attempt in range(1, max_attempts + 1):
        try:
            return operation()
        except RetryableError:
            if attempt == max_attempts:
                raise

            delay = min(
                30,
                1 * (2 ** (attempt - 1))
            )
            delay = random.uniform(0, delay)
            time.sleep(delay)

    raise RuntimeError("unreachable")
```

A real production helper should additionally support:

```text
deadline
logging
metrics
cancellation
exception filtering
operation name
```

# 14. Retry Policy Object

Keep retry behavior explicit and testable:

```python
from dataclasses import dataclass

@dataclass(frozen=True)
class RetryPolicy:
    max_attempts: int = 3
    base_delay: float = 1.0
    max_delay: float = 30.0
    max_elapsed: float = 120.0
```

Different operations can use different policies.

# 15. Operation-Specific Retry Policies

Do not use one retry rule for everything.

Example:

```text
GET status                  -> 5 attempts
read-only AWS discovery     -> bounded retry
deployment mutation         -> 3 attempts + idempotency
production delete           -> usually no automatic retry
rollout polling             -> deadline-based polling
```

The correct policy depends on risk and operation semantics.

# 16. Read vs Write

Read operations are generally easier to retry:

```text
GET
LIST
DESCRIBE
```

Writes require more care:

```text
CREATE
UPDATE
DELETE
```

A timeout on a mutation does not prove that the server did nothing.

# 17. The Most Important Timeout Rule

Suppose:

```text
POST create resource
 ↓
server creates resource
 ↓
response is lost
 ↓
client times out
```

If Python blindly retries the POST, it may create a duplicate.

After a mutation timeout:

```text
do not immediately repeat
 ↓
query actual state
 ↓
determine:
  already succeeded
  still running
  failed
  unknown
```

This is one of the most important production automation patterns.

# 18. Idempotency

Idempotency means repeated execution converges to the same desired state.

Prefer:

```python
replicas = 3
```

over a relative mutation such as:

```python
replicas += 1
```

For creates, use where supported:

```text
idempotency keys
stable resource names
resource existence checks
server-side idempotency
```

Retry + idempotency is much safer than retry alone.

# 19. Desired-State Thinking

The safest automation pattern is:

```text
desired state
+
current state
=
next safe action
```

For example:

```text
DELETE + resource absent -> SUCCESS
GET + resource absent -> possibly FAILURE
```

The same API response can have different meanings depending on the intended state.

# 20. HTTP Error Handling

With `requests`, translate low-level errors into application errors:

```python
import requests

def get_status(url):
    try:
        response = requests.get(
            url,
            timeout=10
        )
        response.raise_for_status()
        return response.json()

    except requests.Timeout as exc:
        raise OperationTimeoutError(
            "API request timed out"
        ) from exc

    except requests.HTTPError as exc:
        status = exc.response.status_code

        if status == 401:
            raise AuthenticationError(
                "API authentication failed"
            ) from exc

        if status == 403:
            raise AuthorizationError(
                "API authorization failed"
            ) from exc

        if status in {429, 500, 502, 503, 504}:
            raise RetryableError(
                f"Transient API failure: HTTP {status}"
            ) from exc

        raise DependencyError(
            f"API returned HTTP {status}"
        ) from exc
```

In real code, also validate response schemas and handle connection errors explicitly.

# 21. Retry-After

For rate limiting:

```text
HTTP 429
Retry-After: 30
```

prefer the server-provided delay when appropriate.

Validate and cap external delay values so a malformed or excessive response cannot make the job wait indefinitely.

# 22. HTTP Method Semantics

A `GET` is generally safe to retry.

A `POST` may not be.

An API may make POST effectively idempotent through:

```text
Idempotency-Key
```

Always follow the API contract instead of assuming a method is automatically safe.

# 23. AWS and Boto3

AWS SDKs already provide retry behavior for many service interactions.

Do not blindly add aggressive retries around every Boto3 call.

Use:

```text
AWS SDK transport retries
+
application-level workflow retry only where required
```

Understand the SDK retry mode and avoid multiplying retries at several layers.

# 24. Retry Amplification

Suppose:

```text
SDK retries = 3
application retries = 3
CI retries = 3
```

The number of actual requests/workflow executions can become much larger than expected.

Define ownership:

```text
SDK      -> transport-level retry
Python   -> business/workflow retry
CI       -> whole-job retry only when safe
```

Keep every layer bounded.

# 25. Kubernetes Error Handling

Typical Kubernetes conditions include:

```text
API timeout
authentication failure
RBAC denial
resource conflict
NotFound
rollout timeout
pod failure
admission rejection
```

Classify each rather than treating all `kubectl` failures as transient.

# 26. Kubernetes Rollout Timeout

If a rollout times out:

```text
deployment command timed out
 ↓
query deployment
 ↓
inspect pods
 ↓
inspect events
 ↓
inspect readiness
 ↓
inspect application logs
 ↓
determine:
  healthy
  still progressing
  failed
```

Only then decide whether to wait, retry a safe read, or recover/rollback.

# 27. Kubernetes Conflict

A resource conflict is often a concurrency issue.

Safer flow:

```text
read latest resource
 ↓
recalculate desired change
 ↓
update
```

Do not blindly replay a stale update.

Kubernetes resource versions can support optimistic concurrency.

# 28. Terraform Error Handling

Terraform is stateful infrastructure tooling. Python should not blindly retry:

```bash
terraform apply
```

If it times out:

```text
check whether Terraform is still running
check state
check lock
check cloud resources
determine actual outcome
```

Only then decide the next action.

Python should orchestrate Terraform rather than reinvent Terraform state management.

# 29. Helm Error Handling

If:

```bash
helm upgrade
```

times out:

```text
inspect release status
inspect Kubernetes resources
inspect hooks
inspect events
```

before repeating the upgrade.

The release may already have changed.

# 30. ArgoCD Error Handling

If an ArgoCD sync request times out:

```text
query application
 ↓
check sync status
 ↓
check health
 ↓
check Kubernetes rollout
```

The original sync may still be running or may have completed.

Do not automatically send another sync command without checking state.

# 31. Jenkins and GitHub Actions

If a job trigger times out:

```text
query existing runs/builds first
```

before creating another run.

Otherwise a network timeout can accidentally create duplicate CI executions.

At CI level, retry only known infrastructure failures, not deterministic:

```text
test failures
security failures
invalid configuration
quality gate failures
```

# 32. Git and GitOps

If a Git push fails because the remote changed:

```text
fetch
 ↓
read current remote state
 ↓
recalculate desired change
 ↓
commit/push according to policy
```

Do not blindly force-push production configuration.

For GitOps, the safe model is:

```text
Git desired state
 ↓
ArgoCD reconciliation
 ↓
Kubernetes actual state
```

Python should verify and orchestrate rather than bypass established GitOps controls.

# 33. Subprocess Errors

For DevOps CLIs:

```python
import subprocess

try:
    subprocess.run(
        ["kubectl", "get", "pods"],
        check=True,
        timeout=30,
        capture_output=True,
        text=True,
    )
except subprocess.TimeoutExpired as exc:
    raise OperationTimeoutError(
        "kubectl timed out"
    ) from exc
except subprocess.CalledProcessError as exc:
    raise DependencyError(
        f"kubectl failed with exit code {exc.returncode}"
    ) from exc
```

Use argument lists instead of shell strings and avoid `shell=True` unless there is a justified, controlled reason.

# 34. Polling vs Retry

They are related but different.

Retry:

```text
operation fails
 ↓
try operation again
```

Polling:

```text
operation starts
 ↓
check state
 ↓
not ready
 ↓
wait
 ↓
check again
```

For polling, use:

```text
deadline
poll interval
terminal failure detection
success condition
```

Never use an unbounded `while not ready:` loop.

# 35. Polling Example

Concept:

```python
deadline = time.monotonic() + 600

while time.monotonic() < deadline:
    state = get_rollout_state()

    if state == "READY":
        return

    if state == "FAILED":
        raise VerificationError(
            "Rollout failed"
        )

    time.sleep(5)

raise OperationTimeoutError(
    "Rollout did not become ready"
)
```

Stop immediately when a terminal failure is observed.

# 36. Eventual Consistency

Cloud APIs may not immediately show a newly created resource.

Example:

```text
create
 ↓
immediate GET
 ↓
not found
```

A short, bounded read-after-write retry may be correct.

But first rule out:

```text
wrong account
wrong region
wrong resource ID
permission problem
actual failure
```

Never label every NotFound as eventual consistency.

# 37. Unknown State

A critical state is:

```text
UNKNOWN
```

Example:

```text
mutation timed out
+
status API unavailable
```

Do not treat this as success.

For production mutations:

```text
UNKNOWN -> stop + investigate
```

unless the platform provides a safe reconciliation mechanism.

# 38. Recovery Strategies

Depending on the workflow, recovery can mean:

```text
retry
refresh state
skip an already-completed step
continue from checkpoint
rollback
compensate
cleanup
manual intervention
```

Never use:

```python
except Exception:
    continue
```

unless skipping is explicitly safe.

# 39. Partial Failure

Suppose:

```text
create infrastructure -> SUCCESS
deploy application     -> SUCCESS
update DNS             -> FAILURE
```

Restarting the entire workflow can repeat successful side effects.

Instead:

```text
inspect state
 ↓
resume from safe boundary
 ↓
retry DNS only if safe
```

This is why state-aware workflows are more reliable than command replay.

# 40. Step-Level Retry

Prefer retrying the smallest safe unit.

Bad:

```text
entire deployment workflow
```

when only the DNS call failed.

Better:

```text
infrastructure -> already complete
application     -> already complete
DNS             -> retry
```

The workflow should know what has already happened.

# 41. Fail-Fast vs Continue

Fail fast when a failure invalidates all later work:

```text
wrong AWS account
invalid configuration
security policy failure
```

Continue when resources are independent:

```text
collect diagnostics from 20 independent nodes
```

Even when continuing, the overall result must still be failed if required work failed.

# 42. Parallel Error Handling

With `concurrent.futures`, collect worker exceptions explicitly.

Example result:

```text
A -> SUCCESS
B -> FAILURE
C -> SUCCESS

successes = [A, C]
failures = [B]
overall = FAILED
```

Do not let worker exceptions disappear silently.

# 43. Cancellation

A CI runner or Kubernetes Job can receive `SIGTERM`.

Retry logic should check cancellation before sleeping or starting another attempt.

Correct lifecycle:

```text
termination requested
 ↓
stop new retries
 ↓
cleanup
 ↓
exit
```

Do not begin another production mutation after cancellation has been requested.

# 44. Timeout Hierarchy

Use intentionally nested timeouts:

```text
individual request
    <
operation
    <
workflow
    <
CI job
```

Example:

```text
HTTP request = 10s
retry budget = 60s
rollout timeout = 10m
CI job timeout = 15m
```

Leave enough outer budget for cleanup and reporting.

# 45. Cleanup After Failure

Use `finally` or context managers:

```python
resource = create_test_resource()

try:
    run_test(resource)
finally:
    cleanup(resource)
```

Cleanup itself can fail, so record it separately.

For temporary resources, use ownership metadata:

```text
ManagedBy=DevOpsAutomation
RunID=7812
Environment=Test
```

Only resources matching the intended scope should be deleted.

# 46. Cleanup Retry

Cleanup may be retried for transient API failures, but it must still respect:

```text
ownership
deadline
environment
resource identity
```

If ownership is uncertain:

```text
STOP
```

Do not turn cleanup into a broad deletion operation.

# 47. Error Logging

A retry log should include:

```text
operation
attempt
max attempts
delay
reason
run ID
```

Example:

```text
WARN retrying operation=aws_describe
attempt=2/4
delay=2.4s
reason=rate_limit
```

Final failure:

```text
ERROR operation=aws_describe
category=RATE_LIMIT
attempts=4
elapsed=31s
```

# 48. Error Metrics

Track:

```text
automation_failures_total
automation_retries_total
automation_timeouts_total
automation_verification_failures_total
cleanup_failures_total
```

Retries themselves are operational signals. A job with:

```text
99% final success
35% retry rate
```

may still have a serious dependency reliability problem.

# 49. Error Event Schema

A structured event can look like:

```json
{
  "event": "automation_failed",
  "run_id": "7812",
  "environment": "staging",
  "service": "orders",
  "operation": "deployment",
  "category": "TIMEOUT",
  "attempt": 3,
  "elapsed_seconds": 601
}
```

Put detailed diagnostics in logs/artifacts and stable categories in metrics.

# 50. Error Codes

Stable error codes help CI and operators:

```text
CFG_001
AUTH_001
AUTHZ_001
NET_001
TIMEOUT_001
VERIFY_001
```

Use machine-readable categories/codes rather than forcing downstream systems to parse human messages.

# 51. Safe Debugging

Do not dump configuration blindly:

```python
logger.debug("config=%r", config)
```

A configuration object may contain credentials.

Instead log safe fields explicitly:

```text
environment
region
cluster
namespace
service
version
```

Use `logger.exception()` for traceback details when appropriate, but review third-party exception contents for accidental secret exposure.

# 52. Error Handling at the Correct Layer

A useful layering model:

```text
API client
   ↓
domain/service layer
   ↓
workflow
   ↓
CLI/application boundary
```

The API layer should translate low-level failures.

The workflow should decide retry/recovery.

The CLI should produce the final result and exit code.

Avoid both extremes:

```text
catch everything everywhere
```

and:

```text
turn every error into RuntimeError("failed")
```

# 53. Circuit Breaker

For long-running automation services, a circuit breaker can protect an unhealthy dependency.

States:

```text
CLOSED
  |
  | repeated failures
  v
OPEN
  |
  | wait
  v
HALF-OPEN
  |
  +-- success -> CLOSED
  |
  +-- failure -> OPEN
```

For short-lived one-shot scripts, bounded retries are often simpler. Do not introduce a circuit breaker where it adds unnecessary complexity.

# 54. Retry Storm and Thundering Herd

Example:

```text
100 jobs
 ↓
dependency fails
 ↓
all sleep 5 seconds
 ↓
100 retries together
```

Mitigate with:

```text
jitter
bounded concurrency
backoff
circuit breaking for long-running services
```

This is especially important in Jenkins/GitHub Actions environments with many simultaneous jobs.

# 55. Retry Ownership

Define which layer retries which failure:

```text
HTTP client -> transport
AWS SDK     -> SDK/service transient failures
Python      -> workflow semantics
CI          -> whole-job recovery only when safe
```

Do not stack generic retries at every layer without calculating the total attempt count.

# 56. Retry Testing

Tests should cover:

```text
success first attempt
success after one retry
success after multiple retries
all attempts fail
non-retryable exception
retry budget exhausted
backoff cap
jitter range
cancellation
timeout
```

Also test dangerous mutations to ensure retries cannot duplicate side effects.

# 57. Testing Unknown Mutation Outcomes

Example test:

```text
create request
 ↓
client timeout
 ↓
resource lookup
 ↓
resource exists with desired state
```

Expected:

```text
do not create again
verify
SUCCESS
```

Another:

```text
timeout
 ↓
state unavailable
```

Expected:

```text
UNKNOWN
STOP
```

These tests are more valuable than testing only happy-path retries.

# 58. Testing Environment Guards

Test:

```text
expected=staging
actual=production
```

Expected:

```text
BLOCK
```

No mutation should occur.

Similarly test:

```text
wrong AWS account
wrong region
wrong EKS cluster
wrong namespace
wrong Git branch
```

All safety-critical mismatches should fail closed.

# 59. Testing Error Classification

Test mappings such as:

```text
401 -> AuthenticationError
403 -> AuthorizationError
429 -> Retryable/RateLimit
503 -> Retryable/Dependency
400 -> permanent request failure
timeout -> retryable where safe
conflict -> refresh/reconcile
```

The classifier should be deterministic and should not rely on matching arbitrary exception strings.

# 60. CI/CD Error Policy

A CI/CD pipeline should distinguish:

```text
TEST_FAILURE
SECURITY_FAILURE
BUILD_FAILURE
DEPLOYMENT_FAILURE
VERIFICATION_FAILURE
INFRASTRUCTURE_FAILURE
```

Examples:

```text
SonarQube quality gate failure -> BLOCK
Trivy critical vulnerability  -> BLOCK
unit test failure              -> FAIL
temporary registry outage      -> possibly RETRY
deployment verification fail  -> FAIL/ROLLBACK
```

Do not retry deterministic policy failures.

# 61. Production Error Decision Tree

```text
Operation failed
      |
      v
Can it be classified?
      |
   +--+--+
   |     |
  NO    YES
   |     |
   v     v
 STOP  Retryable?
          |
       +--+--+
       |     |
      NO    YES
       |     |
       v     v
      FAIL  Safe to repeat?
              |
           +--+--+
           |     |
          NO    YES
           |     |
           v     v
      Verify state  Budget left?
                       |
                    +--+--+
                    |     |
                   NO    YES
                    |     |
                    v     v
                   FAIL Backoff
                          |
                          v
                        Retry
                          |
                          v
                        Verify
```

# 62. Real-World DevOps Example — EKS Deployment

Production flow:

```text
Jenkins/GitHub Actions
        |
        v
Python pre-flight
        |
        +--> AWS account guard
        +--> region guard
        +--> EKS cluster guard
        +--> namespace guard
        |
        v
Deploy/ArgoCD operation
        |
        v
Timeout?
        |
       YES
        |
        v
Query Kubernetes state
        |
   +----+----+
   |         |
 healthy   failed
   |         |
   v         v
 verify    diagnose
   |         |
   v         v
 SUCCESS   rollback/recovery
```

The important point is that a timeout does not automatically mean the deployment should be executed again.

# 63. Real-World DevOps Example — AWS Throttling

```text
Boto3 request
 ↓
ThrottlingException
 ↓
classify RATE_LIMIT
 ↓
check retry budget
 ↓
backoff + jitter
 ↓
retry
 ↓
success
```

If throttling persists:

```text
bounded failure
+
metric
+
diagnostic information
```

Then investigate:

```text
API volume
concurrency
pagination
caching
AWS service limits
```

# 64. Real-World DevOps Example — GitOps

```text
Python reads manifest
 ↓
desired image tag differs
 ↓
update Git
 ↓
push fails because remote changed
 ↓
fetch latest
 ↓
recalculate diff
 ↓
push according to concurrency policy
 ↓
ArgoCD sync/reconcile
 ↓
verify application health
```

Do not force-push blindly and do not retry a stale Git operation without refreshing state.

# 65. Real-World DevOps Example — Partial Failure

Suppose:

```text
Terraform infrastructure -> SUCCESS
Helm deployment           -> SUCCESS
smoke test                 -> FAILURE
```

Correct behavior:

```text
do not rerun Terraform blindly
inspect application
collect Kubernetes/ELK/Prometheus evidence
determine whether rollback is required
```

The automation should preserve the successful state and recover from the failing stage.

# 66. Real-World DevOps Example — Cleanup

```text
create temporary namespace
 ↓
run integration tests
 ↓
test fails
 ↓
finally cleanup
 ↓
Kubernetes API returns timeout
 ↓
bounded cleanup retry
 ↓
still fails
 ↓
record orphaned resource
 ↓
reconciliation/alert
```

The original test failure must remain visible, and cleanup failure must not be hidden.

# 67. Anti-Patterns

Avoid:

```text
except Exception: pass
retry forever
retry every exception
no timeout
fixed retry delay
no jitter for distributed retries
blindly retrying CREATE
blindly retrying production DELETE
treating timeout as proof of failure
restarting an entire workflow after one step fails
retrying security policy failures
returning exit code 0 after failure
logging secrets
catching and replacing every exception with a generic message
```

Each pattern either hides failure, increases blast radius, or wastes resources.

# 68. Production Error-Handling Checklist

```text
[ ] Exception hierarchy
[ ] Root cause preservation
[ ] Stable error categories
[ ] Retryable classification
[ ] Non-retryable classification
[ ] Maximum attempts
[ ] Maximum elapsed time
[ ] Exponential backoff
[ ] Jitter
[ ] Operation timeout
[ ] Cancellation
[ ] Idempotency analysis
[ ] Post-timeout verification
[ ] Structured logging
[ ] Error metrics
[ ] Correct exit codes
[ ] Cleanup
[ ] Recovery/rollback policy
[ ] Secret-safe logging
[ ] Unit tests
[ ] Integration tests
[ ] Failure-path tests
```

# 69. Retry Checklist

Before retry:

```text
[ ] Is the error transient?
[ ] Is the operation safe to repeat?
[ ] Is current state known?
[ ] Is idempotency available?
[ ] Are attempts remaining?
[ ] Is time remaining?
[ ] Has cancellation been requested?
```

During retry:

```text
[ ] log attempt
[ ] backoff
[ ] jitter
[ ] update metrics
```

After retry:

```text
[ ] verify state
[ ] record final result
[ ] return non-zero on final failure
```

# 70. Senior Interview — What Errors Should Be Retried?

Strong answer:

> I retry only known transient failures and only when repeating the operation is safe. Typical examples are temporary network failures, timeouts, rate limiting, and selected 5xx responses. I normally fail immediately for invalid configuration, authorization failures, safety-guard failures, and deterministic application errors. For mutations, I first consider idempotency and whether the outcome is already known.

# 71. Senior Interview — Why Exponential Backoff and Jitter?

Strong answer:

> Exponential backoff reduces pressure on a dependency that is already failing. Jitter prevents many workers from waking up and retrying at exactly the same time, which avoids thundering-herd behavior. I also cap the delay and enforce a total retry deadline.

# 72. Senior Interview — What If a Deployment Times Out?

Strong answer:

> I do not immediately repeat the deployment. First I query the actual state because the server may have completed the operation even though the client timed out. In EKS I inspect the deployment, pods, events, readiness and application health. Based on the state I either verify success, continue waiting, or execute the defined recovery/rollback path.

# 73. Senior Interview — How Do You Prevent Duplicate Resources?

Strong answer:

> I design mutations to be idempotent where possible. I use stable identifiers, desired-state operations, resource existence checks, and idempotency keys when supported. For an uncertain create request, I query the resource before issuing another create request.

# 74. Senior Interview — How Do You Handle 401 vs 403?

Strong answer:

> A 401 is an authentication problem. If the platform supports credential refresh, I may refresh and retry in a controlled way. A 403 is normally an authorization problem, so repeating the same request does not help. I fail and surface the missing permission or identity issue.

# 75. Senior Interview — Retry vs Polling

Strong answer:

> Retry repeats an operation that failed. Polling checks the state of an operation that may already be running. For example, retrying an API request is different from polling an EKS rollout until the deployment reaches the desired state. Both need deadlines, but their semantics are different.

# 76. Senior Interview — What Is a Retry Budget?

Strong answer:

> A retry budget limits both attempts and elapsed time. For example, I might allow four attempts with a two-minute maximum elapsed time. This prevents exponential backoff or repeated retries from consuming the entire CI job.

# 77. Senior Interview — What Is Retry Amplification?

Strong answer:

> Retry amplification happens when several layers retry independently. For example, the SDK, application, and CI system can each retry the same failure. I define retry ownership at each layer and keep all limits bounded so the total request volume remains predictable.

# 78. Senior Interview — How Do You Handle Terraform Timeout?

Strong answer:

> I inspect whether Terraform is still running, check state and locking, and inspect cloud resources before doing anything else. I do not blindly rerun `terraform apply`, because the original operation may have partially or completely changed infrastructure.

# 79. Senior Interview — How Do You Handle ArgoCD Timeout?

Strong answer:

> I query the ArgoCD application after the timeout. The sync could still be running or could already have completed. I inspect sync status, application health and Kubernetes rollout before deciding whether another action is safe.

# 80. Senior Interview — How Do You Handle Partial Failure?

Strong answer:

> I model the workflow as explicit steps and maintain state/checkpoints. If an early step succeeded and a later step failed, I inspect actual state rather than restarting everything. I continue from a safe checkpoint, retry the failed operation if it is safe, or execute a tested recovery/rollback path.

# 81. Senior Interview — How Do You Handle Cleanup Failure?

Strong answer:

> I keep the original failure visible and record cleanup failure separately. Temporary resources are tagged with ownership and run IDs. Cleanup can use bounded retries for transient errors, and persistent orphaned resources are handled through reconciliation or alerting.

# 82. Senior Interview — Why Is `except Exception: pass` Dangerous?

Strong answer:

> It hides failures and can cause the automation to continue with an invalid or partially changed state. In production, I catch only errors I can handle, preserve the root cause, and let unexpected errors reach the application boundary where they become a clear non-zero result.

# 83. Senior Interview — Why Is Timeout Not Proof of Failure?

Strong answer:

> A timeout means the client stopped waiting. A server may have completed the mutation before the response was lost. Therefore, after a mutation timeout I query actual state before deciding whether to retry.

# 84. Senior Interview — How Do You Test Retry Logic?

Strong answer:

> I test first-attempt success, success after retry, retry exhaustion, non-retryable failures, backoff limits, deadline exhaustion, cancellation, and uncertain mutation outcomes. I also test that retrying cannot create duplicate resources.

# 85. Final Production Model

Remember this model:

```text
                 FAILURE
                    |
                    v
                 CLASSIFY
                    |
        +-----------+-----------+
        |           |           |
     PERMANENT   TRANSIENT    UNKNOWN
        |           |           |
        v           v           v
       FAIL     SAFE TO RETRY? VERIFY
                    |
                 +--+--+
                 |     |
                NO    YES
                 |     |
                 v     v
                FAIL  BUDGET?
                         |
                      +--+--+
                      |     |
                     NO    YES
                      |     |
                      v     v
                     FAIL BACKOFF
                           |
                           v
                         RETRY
                           |
                           v
                         VERIFY
                           |
                           v
                        CLEANUP
                           |
                           v
                     FINAL RESULT
```

Production Python error handling is about making failure behavior deterministic, observable, and safe.

The DevOps mindset is:

```text
Do not ask:
"Can I retry this?"

Ask:
"Why did it fail?"
"Is the failure transient?"
"Is the operation safe to repeat?"
"Do I know the current state?"
"How much retry budget remains?"
"What is the safest final outcome?"
```

# 86. Section Progress

```text
10-Python-Production/
│
├── 01-Production-Scripting.md        ✓
├── 02-Error-Handling-and-Retry.md    ✓
├── 03-Logging-and-Observability.md
├── 04-Security.md
├── 05-Performance.md
├── 06-Concurrency.md
├── 07-Configuration-Management.md
└── 08-Production-Best-Practices.md
```

Next file:

```text
03-Logging-and-Observability.md
```
