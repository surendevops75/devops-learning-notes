# Coding Questions

> Python coding preparation for DevOps/DevSecOps interviews. Starts with core coding patterns and progresses into AWS, Kubernetes, CI/CD, GitOps, Terraform, security, observability, concurrency, testing, and production-grade automation.

## Coding Interview Framework

**Understand → clarify constraints → implement → test → complexity → production hardening**

### DevOps coding rule

Do not stop at code that works for one input. Explain how it behaves with large datasets, API failures, retries, timeouts, credentials, concurrent execution, and production-scale workloads.

## 1. Coding Interview Strategy

For DevOps coding rounds, first clarify input/output and constraints, then write a simple correct solution, test edge cases, explain complexity, and finally discuss production hardening such as logging, validation, timeouts, retries, and security.

---

## 2. Reverse a String

Use slicing for the simple case: `s[::-1]`. Complexity is O(n) time and O(n) space for the resulting string.

---

## 3. Check Palindrome

Normalize only when the problem requires it, then compare the string with its reverse or use two pointers. Explain whether case/punctuation should be ignored.

---

## 4. Count Characters

Use `collections.Counter` for frequency counting. Complexity is O(n) time and O(k) additional space where k is the number of distinct characters.

---

## 5. First Non-Repeating Character

Count characters first, then scan the original string to find the first count of one. This preserves original ordering and is O(n).

---

## 6. Remove Duplicates from List

Use a set when order does not matter. If order must be preserved, maintain a set of seen values and append unseen items.

---

## 7. Preserve Order While Deduplicating

For hashable values, track `seen` and build a result list. Explain that set membership is average O(1).

---

## 8. Find Duplicate Values

Use a Counter and return keys with counts greater than one. This is clearer than nested loops.

---

## 9. Two Sum

Use a dictionary mapping value to index while scanning once. Average O(n) time and O(n) space.

---

## 10. Find Missing Number

For values from 0..n, use arithmetic sum or XOR. Explain integer overflow considerations for other languages; Python integers do not overflow normally.

---

## 11. Find Maximum and Minimum

Use built-ins when allowed. If implementing manually, maintain running min/max in one pass.

---

## 12. Second Largest Number

Maintain the two largest distinct values or sort if constraints are small. Clarify whether duplicates count.

---

## 13. Merge Two Sorted Lists

Use two pointers to achieve O(n+m) time instead of concatenating and sorting.

---

## 14. Intersection of Two Lists

Use sets for unique intersection. If multiplicity matters, use Counter.

---

## 15. Union of Lists

Use sets for unique union, or ordered tracking if original order is required.

---

## 16. Anagram Check

Compare character frequencies with Counter or sorted normalized strings. Counter is O(n) expected.

---

## 17. Word Frequency

Split/parse according to requirements and count with Counter. For large files, process incrementally.

---

## 18. Find Most Frequent Word

Normalize according to the problem, count with Counter, then apply a deterministic tie-break rule.

---

## 19. Flatten One-Level List

Use a comprehension such as `[x for sub in data for x in sub]` when input is known to be one level deep.

---

## 20. Flatten Nested List

Use recursion or an explicit stack. For arbitrary deeply nested structures, discuss recursion depth and iterative alternatives.

---

## 21. Chunk a List

Use slicing in a loop for simple data. Validate chunk size is positive.

---

## 22. Rotate List

Use slicing for a simple fixed rotation. Normalize rotation count using modulo list length.

---

## 23. Move Zeros to End

Use a stable two-pointer/write-index approach to preserve nonzero ordering.

---

## 24. Find Common Elements

Use sets for unique values or Counters for multiplicity. Avoid O(n²) membership checks for large lists.

---

## 25. Sort Dictionary by Value

Use `sorted(d.items(), key=lambda item: item[1])` and specify ascending/descending requirements.

---

## 26. Group Dictionary Records

Use `defaultdict(list)` to group records by namespace, environment, owner, or status.

---

## 27. Count Status Values

Use Counter for statuses such as HEALTHY/WARNING/CRITICAL. This maps directly to DevOps health reports.

---

## 28. Merge Dictionaries

Use `{**a, **b}` or `a | b` on supported Python versions, with later values overriding duplicate keys.

---

## 29. Safely Access Nested Data

Use explicit validation or helper functions rather than long chained indexing that can throw KeyError/TypeError.

---

## 30. Parse JSON String

Use `json.loads()` inside controlled exception handling and validate the expected schema after parsing.

---

## 31. Read JSON File

Use `json.load()` with a context manager, then validate required fields.

---

## 32. Write JSON Report

Use `json.dump()` with explicit encoding and structured data. Avoid writing secrets into the report.

---

## 33. Parse YAML Safely

Use a safe loader such as `yaml.safe_load()` and validate the resulting structure.

---

## 34. Read a Large Log File

Iterate over the file line-by-line instead of calling `read()` on a potentially huge file.

---

## 35. Count ERROR Lines

Stream the file and increment a counter for matching lines. This uses O(1) memory apart from output state.

---

## 36. Top Error Messages

Normalize/extract error signatures while streaming and use Counter. Keep raw log content out of memory.

---

## 37. Find Recent Log Entries

If the file is large, process from the end using an appropriate reverse/stream strategy rather than loading everything.

---

## 38. Parse Key-Value Lines

Split each line carefully, validate the delimiter, strip whitespace, and handle malformed records according to policy.

---

## 39. Disk Usage Checker

Use `shutil.disk_usage()` or filesystem APIs, calculate used percentage, and compare with configurable thresholds.

---

## 40. Directory Size

Walk files and sum sizes while handling permission errors and symlinks according to policy.

---

## 41. Find Old Files

Use pathlib/os stat timestamps and an age threshold. For cleanup, add dry-run and protected-path controls.

---

## 42. Safe File Cleanup

Require explicit root path, allowlist/pattern, age threshold, dry-run, and safeguards against path traversal or deleting `/`/system paths.

---

## 43. File Copy Automation

Use shutil and preserve required metadata only when necessary. Validate source/destination and handle existing files deliberately.

---

## 44. Create Backup

Copy to a uniquely named destination and verify success. For production backups, add retention, integrity, encryption, and audit requirements.

---

## 45. Checksum a File

Read in chunks and update `hashlib.sha256()` rather than loading the entire file into memory.

---

## 46. Compare File Hashes

Compute hashes using chunked reads and compare exact digest values.

---

## 47. CSV Parsing

Use the `csv` module rather than manually splitting on commas because quoted fields can contain commas.

---

## 48. Generate CSV Report

Use `csv.DictWriter` with a fixed schema and safe output path.

---

## 49. Environment Variable Reader

Read with `os.getenv`, validate required variables, and avoid printing secret values.

---

## 50. Configuration Validator

Check required keys, allowed values, numeric ranges, URLs, paths, and environment identity before performing external operations.

---

## 51. CLI with argparse

Define subcommands/options, validate arguments, provide help, and return nonzero exit codes on operational failure.

---

## 52. CLI Exit Code

Use 0 for success and nonzero values for failure. Keep the contract stable because CI systems depend on it.

---

## 53. Subprocess Runner

Use `subprocess.run([...], check=True, capture_output=True, text=True, timeout=...)` with a safe argument list.

---

## 54. Safe Command Builder

Keep command and arguments separate. Validate user-controlled values and avoid shell=True.

---

## 55. Run Terraform from Python

Wrap Terraform commands in a controlled runner, capture exit status/output, enforce timeout, and preserve plan/apply evidence.

---

## 56. Run Helm from Python

Use argument arrays, controlled values/files, timeout, exit handling, and avoid passing secrets through command-line arguments.

---

## 57. Run Git from Python

Use subprocess with separate arguments and validate repository path, branch, remote, and expected SHA.

---

## 58. Run kubectl from Python

Prefer the Kubernetes API client when practical; if kubectl is required, invoke it safely with explicit context and timeout.

---

## 59. Retry Function

Implement a reusable retry helper with max attempts, exponential backoff, jitter, retryable exception/status classification, and a total deadline.

---

## 60. Retry with Jitter

Compute exponential delay, cap it, then add bounded randomness so multiple workers do not retry simultaneously.

---

## 61. Retry Only Transient Errors

Do not retry invalid input, authentication, authorization, or policy failures merely because a call failed.

---

## 62. Timeout Wrapper

Wrap network/subprocess operations with explicit timeout and propagate remaining workflow deadline.

---

## 63. Polling Function

Poll until a desired state or deadline is reached, increasing interval gradually and avoiding fixed tight loops.

---

## 64. Wait for Kubernetes Rollout

Poll Deployment status/conditions with a deadline and inspect failure conditions instead of sleeping for a fixed duration.

---

## 65. Wait for AWS Resource

Use boto3 waiters where supported; otherwise poll with backoff, state validation, and a deadline.

---

## 66. API GET Function

Use requests.Session, timeout, status validation, safe logging, and typed/validated response parsing.

---

## 67. API POST Function

Validate payload, use secure authentication, timeout, status handling, idempotency where supported, and safe retries only when appropriate.

---

## 68. Handle HTTP Errors

Convert status codes into meaningful exceptions such as AuthenticationError, AuthorizationError, RateLimitError, or TransientAPIError.

---

## 69. Parse Retry-After

Honor a valid Retry-After value when supported, but still enforce a maximum delay and overall deadline.

---

## 70. Pagination Helper

Yield pages/results incrementally so callers do not need to load all resources into memory.

---

## 71. AWS Resource Iterator

Use boto3 paginators and yield normalized resource records one at a time.

---

## 72. Kubernetes Resource Iterator

Use appropriate list/pagination behavior and yield only normalized fields required by the caller.

---

## 73. Concurrent API Calls

Use ThreadPoolExecutor with a bounded worker count for independent I/O operations.

---

## 74. Collect Future Errors

Call `future.result()` and classify exceptions; never assume submitted work succeeded.

---

## 75. Concurrent Health Checks

Run independent checks concurrently, aggregate normalized results, and preserve target-specific errors.

---

## 76. Bounded Concurrency

Make worker count configurable and set different limits per dependency where service capacity differs.

---

## 77. Queue Worker

Use a bounded queue and a fixed worker pool to implement backpressure.

---

## 78. Producer-Consumer

Producer discovers work, bounded queue buffers it, workers process it, and the system stops safely when the queue/deadline is exceeded.

---

## 79. Thread-Safe Counter

Use locks or aggregate results per worker and combine them centrally. Avoid unnecessary shared mutable state.

---

## 80. Async HTTP Concept

Use async clients and `asyncio.gather` for many I/O tasks, but avoid blocking synchronous calls inside the event loop.

---

## 81. Async Timeout

Use asyncio timeout mechanisms around async operations and propagate cancellation safely.

---

## 82. Process Pool

Use ProcessPoolExecutor for CPU-heavy independent work when multiprocessing overhead is justified.

---

## 83. Complexity Question

State time and space complexity for your algorithm and identify the operation that dominates at production scale.

---

## 84. Optimize O(n²) Search

Replace nested membership loops with dictionaries/sets when the data semantics permit.

---

## 85. Optimize N+1 API Calls

Use bulk/list endpoints, caching, or bounded concurrency. Measure request reduction and API latency.

---

## 86. Memory Optimization

Stream large inputs, use generators, avoid deep copies, bound queues/caches, and retain only required fields.

---

## 87. Cache Function

Use caching only when inputs are stable and stale data is acceptable. Define TTL/invalidation behavior.

---

## 88. LRU Cache Example

Use functools.lru_cache for deterministic functions where bounded reuse is valuable; do not cache environment-dependent state indefinitely.

---

## 89. Dataclass Coding

Define a dataclass for structured health/release results with typed fields and safe defaults.

---

## 90. Enum Coding

Use an Enum for fixed states instead of repeated magic strings.

---

## 91. Custom Exception

Create domain exceptions such as ConfigurationError, AuthenticationError, DeploymentError, and PolicyViolation to make failure handling explicit.

---

## 92. Exception Chaining

Wrap low-level errors with domain context using `raise ... from exc` while preserving the root cause.

---

## 93. Context Manager Coding

Implement a context manager when a resource requires guaranteed cleanup or setup/teardown.

---

## 94. Decorator Coding

Create a small decorator for timing/logging and use functools.wraps. Keep side effects explicit.

---

## 95. Type Hints Coding

Add useful annotations for public interfaces and structured data. Avoid using Any everywhere.

---

## 96. Protocol Coding

Define a protocol for a client interface so a real AWS/Kubernetes client and a test fake can both satisfy it.

---

## 97. Dependency Injection Coding

Pass clients/services into functions/classes rather than creating global clients inside business logic.

---

## 98. Pure Policy Function

Create a pure function that receives normalized health data and returns a release decision. This is easy to unit test.

---

## 99. Health Status Function

Convert observed values and thresholds into HEALTHY/WARNING/CRITICAL/UNKNOWN with explicit handling for missing data.

---

## 100. Threshold Checker

Validate threshold ordering and units before comparing values. Avoid magic constants.

---

## 101. Release Gate Function

Accept security/artifact/deployment evidence and return a deterministic pass/fail/unknown decision.

---

## 102. Security Gate Coding

Reject a release when required scan evidence is missing or policy thresholds fail. Treat unknown security state as failure according to policy.

---

## 103. Artifact Verification Coding

Compare expected image digest/source SHA against observed deployment identity and fail on mismatch.

---

## 104. Environment Guard Coding

Map environment to expected account/cluster and validate identity before mutation.

---

## 105. Idempotency Function

Given current state and desired state, return whether a mutation is needed. Keep the decision pure.

---

## 106. State Machine Coding

Represent workflow states and allowed transitions explicitly rather than scattering string comparisons across the script.

---

## 107. Checkpoint Coding

Persist stage status and evidence so a restart can recover safely.

---

## 108. JSON Release Report

Build a structured report containing source SHA, artifact digest, scan results, approval, GitOps revision, deployment status, and timestamps.

---

## 109. Markdown Release Report

Generate a concise human-readable report from the same normalized data used for machine output.

---

## 110. Structured Log Helper

Create a logging helper that adds run ID, environment, stage, target, and duration without including secrets.

---

## 111. Secret Redaction Function

Redact configured sensitive keys and token-like values before logging/serialization. Do not depend solely on redaction; avoid collecting secrets unnecessarily.

---

## 112. Webhook Signature Verification

Calculate/verify the expected HMAC using the raw request body and secret, compare safely, and validate timestamp/replay rules.

---

## 113. Webhook Event Filter

Accept only supported event types and expected repository/environment before starting privileged workflows.

---

## 114. Path Traversal Guard

Resolve a requested path and verify it remains under an approved root.

---

## 115. URL Allowlist

Parse URLs and allow only approved schemes/hosts or configured destination patterns.

---

## 116. Safe YAML Validator

Load YAML safely, validate expected mapping/list structure, and reject unexpected object types.

---

## 117. JSON Schema Validation

Validate external JSON against a known schema before converting it into internal domain objects.

---

## 118. Log Parser Coding

Stream lines, normalize fields, handle malformed records, and return aggregate statistics plus error counts.

---

## 119. Kubernetes Pod Health Function

Given a Pod object, inspect phase, container states, readiness, restart count, and reasons and return normalized health data.

---

## 120. Deployment Health Function

Given Deployment status, calculate whether desired replicas are available and whether rollout conditions indicate failure.

---

## 121. Service Health Function

Given Service/EndpointSlice data, determine whether ready endpoints exist for expected selectors/ports.

---

## 122. Ingress Health Function

Combine Ingress/controller state, backend endpoints, target health, and HTTP smoke-test results.

---

## 123. EKS Environment Guard

Verify AWS identity and Kubernetes server identity before any write operation.

---

## 124. AWS EC2 Health Function

Normalize instance state/status checks into health results and include instance ID/region safely.

---

## 125. S3 Backup Function

Copy selected objects to a protected backup location with checks for source/destination identity, encryption, retention, and verification.

---

## 126. ECR Image Checker

Verify image exists and expected digest/tag mapping before deployment.

---

## 127. Terraform Plan Gate

Parse/consume plan results, reject unexpected destructive changes, and require approval when policy demands.

---

## 128. ArgoCD Verification Function

Query application status and verify desired revision, sync state, health, and operation result.

---

## 129. Git SHA Checker

Compare expected source SHA with local/remote state and fail when the automation is not operating on the expected revision.

---

## 130. GitOps Manifest Updater

Update only the intended image reference, verify the diff, commit only when necessary, and avoid force-push.

---

## 131. Docker Image Metadata

Read image metadata/digest from registry APIs rather than assuming tags identify immutable content.

---

## 132. CI Build Metadata

Collect build ID, source SHA, branch, actor, artifact digest, and timestamps into a release object.

---

## 133. CI Security Summary

Aggregate SonarQube, Trivy, Veracode, and SCA results into a deterministic policy decision.

---

## 134. Unit Test a Retry Helper

Mock the operation to fail transiently then succeed, assert call count/backoff behavior, and test final failure after max attempts.

---

## 135. Unit Test API 403

Mock a 403 response and assert that the client raises an authorization exception without retrying.

---

## 136. Unit Test API 429

Mock 429 responses followed by success and assert bounded retries and correct final result.

---

## 137. Unit Test Timeout

Simulate a timeout and assert the workflow returns a retryable/transient classification or fails after the deadline.

---

## 138. Unit Test Wrong Account

Mock STS identity with an unexpected account and assert no mutation client is called.

---

## 139. Unit Test Idempotency

Run the same desired-state function twice and verify the second result indicates no mutation.

---

## 140. Unit Test Kubernetes OOM

Provide a mocked Pod status with OOMKilled and assert normalized CRITICAL result with correct evidence.

---

## 141. Unit Test Pending Pod

Mock scheduler events/resource conditions and verify the diagnostic category.

---

## 142. Unit Test ImagePullBackOff

Mock waiting reason and verify registry/image diagnostics are produced.

---

## 143. Unit Test Security Gate

Provide a critical vulnerability and assert promotion is rejected.

---

## 144. Unit Test Artifact Digest

Provide mismatched expected/observed digest and assert deployment gate fails.

---

## 145. Unit Test Terraform Destroy

Mock plan containing destructive change and verify apply is blocked.

---

## 146. Unit Test ArgoCD Drift

Provide desired/live revision mismatch and assert drift status is reported.

---

## 147. Unit Test Smoke Failure

Mock a failed smoke test and verify deployment is not marked successful.

---

## 148. Integration Test AWS

Use a dedicated test account/resources where possible and verify real API authentication, pagination, permissions, and cleanup.

---

## 149. Integration Test Kubernetes

Use an isolated test cluster/namespace to validate API behavior, RBAC, resource status, and rollout verification.

---

## 150. Integration Test CI

Run the Python CLI inside the same container/runtime used by CI to catch environment differences.

---

## 151. Contract Test API

Validate representative real API response schemas against the client's assumptions.

---

## 152. Property-Based Test

Use Hypothesis for parsers/validators where many edge cases are difficult to enumerate manually.

---

## 153. Test Large Input

Use production-like large logs/resource counts to expose memory and performance issues.

---

## 154. Test Malformed Input

Verify malformed JSON/YAML/logs/CLI input fails safely and does not trigger mutation.

---

## 155. Test Secret Safety

Assert that logs/reports do not contain known secret values and that sensitive fields are excluded.

---

## 156. Test Shell Safety

Test that user-controlled input remains an argument and cannot alter command structure.

---

## 157. Test Concurrent Runs

Start multiple simulated releases and verify locks/idempotency prevent conflicting mutations.

---

## 158. Test Graceful Shutdown

Send SIGTERM during processing and verify checkpoint/cleanup and bounded shutdown.

---

## 159. Test Partial Failure

Make one stage fail after another succeeds and verify recovery logic uses persisted state.

---

## 160. Test Retry Storm Protection

Simulate many workers receiving 429 and verify concurrency/backoff/request budget behavior.

---

## 161. Test Queue Backpressure

Produce work faster than workers consume and verify queue remains bounded.

---

## 162. Coding Question: Build a Health Checker

Expected design: configuration → identity validation → resource collection → normalization → concurrent checks → policy evaluation → structured report → metrics/logs → exit code.

---

## 163. Coding Question: Build a Deployment Verifier

Expected design: validate target → query ArgoCD/Kubernetes → wait with deadline → inspect rollout → run smoke test → produce evidence → return deterministic status.

---

## 164. Coding Question: Build a Cleanup Tool

Expected design: target validation → discovery → allowlist/age filters → dry-run → approval → idempotent deletion → verification → audit.

---

## 165. Coding Question: Build a CI Gate

Expected design: collect test/security/artifact evidence → validate source/artifact identity → evaluate policy → produce report → stable exit code.

---

## 166. Coding Question: Build an AWS Resource Inventory

Expected design: account/region validation → paginated discovery → normalized records → bounded concurrency only where needed → streaming output → request budget.

---

## 167. Coding Question: Build a Kubernetes Pod Monitor

Expected design: scoped API queries → normalized Pod states → reason classification → structured logs/metrics → deduplication → alert policy.

---

## 168. Coding Question: Build a Retry Library

Expected design: retry predicate → exponential backoff → jitter → max attempts → overall deadline → cancellation → final exception preservation.

---

## 169. Coding Question: Build a Safe Subprocess Library

Expected design: argument arrays → validation → timeout → controlled environment → stdout/stderr limits → exit-code mapping → safe error context.

---

## 170. Coding Question: Build a REST Client

Expected design: Session → authentication → timeout → status classification → schema validation → pagination → retry policy → request budget.

---

## 171. Coding Question: Build a Release State Machine

Expected design: explicit states/transitions → persistent checkpoint → immutable release ID → idempotent stages → failure/recovery transitions → audit evidence.

---

## 172. Coding Question: Build a GitOps Promotion Tool

Expected design: validate source/artifact → update exact manifest → inspect diff → commit/push safely → verify ArgoCD revision → verify EKS rollout.

---

## 173. Coding Question: Build a Terraform Gate

Expected design: validate environment → run plan → inspect changes → block unexpected destruction → approval → apply → verify actual state.

---

## 174. Coding Question: Build a Security Gate

Expected design: collect SonarQube/Trivy/Veracode/SCA evidence → normalize findings → apply policy → handle exceptions → emit audit result.

---

## 175. Coding Question: Build a Log Analyzer

Expected design: stream files → parse safely → normalize errors → aggregate with Counter → output JSON/summary → handle malformed lines.

---

## 176. Coding Question: Build an EC2 Health Monitor

Expected design: paginated discovery → status checks → policy thresholds → structured health result → Prometheus metrics/ELK logs → exit/alert policy.

---

## 177. Coding Question: Build an EKS Pod Monitor

Expected design: read-only RBAC → scoped queries → Pod state classification → events/log evidence → bounded API usage → health report.

---

## 178. Coding Question: Build S3 Backup Automation

Expected design: validate account/bucket identity → select objects → copy with encryption → verify object metadata/checksum where appropriate → retention/audit.

---

## 179. Coding Question: Build Kubernetes Cleanup Automation

Expected design: dry-run by default → explicit namespace/resource allowlist → age/label filters → production guard → approval → deletion → verification.

---

## 180. Coding Question: Build CI/CD Automation

Expected design: trigger → validation → tests → security → artifact → infrastructure → GitOps → rollout → smoke → observability → promotion/rollback.

---

## 181. Coding Question: Optimize an N+1 Script

Measure requests first, replace per-item calls with bulk APIs or caching, use bounded concurrency only if necessary, and verify reduced API load.

---

## 182. Coding Question: Optimize Memory

Stream inputs/results, normalize only required fields, bound queues/caches, avoid deep copies, and profile with tracemalloc.

---

## 183. Coding Question: Optimize CPU

Profile with cProfile, identify the hot loop/parsing/serialization path, then improve algorithmic complexity before micro-optimizing.

---

## 184. Coding Question: Handle 100k Kubernetes Pods

Avoid one huge object graph. Paginate/process incrementally, normalize small records, use bounded concurrency, and enforce API request budgets.

---

## 185. Coding Question: Handle 1M Log Lines

Stream line-by-line, aggregate only required counters, and write results incrementally.

---

## 186. Coding Question: Handle API Rate Limit at Scale

Use a shared/bounded rate limiter, retry budgets, Retry-After, jitter, caching, and workload partitioning.

---

## 187. Coding Question: Production-Ready Script Checklist

Configuration validation, identity validation, least privilege, secure secrets, timeouts, retries, idempotency, pagination, bounded concurrency, structured logs, metrics, tests, documentation, audit, and safe exit codes.

---

## 188. Coding Interview Trade-off

A correct O(n) solution is not automatically production-ready. Discuss memory, external API limits, failure handling, input validation, concurrency, observability, and operational safety.

---

## 189. Final Coding Interview Answer

I start with a correct simple solution, state complexity, test edge cases, then harden it for DevOps: validate inputs and target identity, use safe API/subprocess handling, timeouts, bounded retries, idempotency, controlled concurrency, structured logs, metrics, tests, and clear exit codes.

---

## Section Progress

```text
12-Python-Interview-Preparation/
├── 01-Basic-Questions.md              ✓
├── 02-Intermediate-Questions.md       ✓
├── 03-Advanced-Questions.md           ✓
├── 04-Python-for-DevOps-Questions.md  ✓
├── 05-Scenario-Based.md               ✓
├── 06-Coding-Questions.md             ✓
├── 07-Production-Scenarios.md
└── 08-Mock-Interview.md
```

**Next file: `07-Production-Scenarios.md`**