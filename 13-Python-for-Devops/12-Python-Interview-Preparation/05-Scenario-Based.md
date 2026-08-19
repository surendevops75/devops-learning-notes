# 12-Python-Interview-Preparation
# 05 — Scenario-Based

> Scenario-based Python interview preparation for DevOps/DevSecOps roles. These scenarios emphasize production diagnosis, AWS/EKS/Kubernetes, CI/CD, GitOps, security, observability, reliability, and safe automation.

## Scenario Answer Framework

**1. Clarify impact → 2. Verify target → 3. Collect evidence → 4. Isolate failure → 5. Safe action → 6. Verify → 7. Prevent recurrence**

### Interview rule

Do not jump directly to `restart`, `retry`, `kubectl delete`, `terraform apply`, or permission escalation. Explain *why* the action is safe and what evidence supports it.

## 1. How to Approach Scenario Questions

Use: clarify the symptom → identify scope/impact → collect evidence → isolate the failure domain → make the smallest safe change → verify recovery → document/root-cause follow-up. Avoid jumping directly to a command or restart.

---

## 2. Scenario: Python Script Suddenly Fails in Production

First check the exact error, recent code/dependency/configuration changes, runtime version, environment variables, credentials, external API status, and logs. Reproduce safely and roll back only after confirming the failure source.

---

## 3. Scenario: Script Works Locally but Fails in Jenkins

Compare Python/runtime versions, installed dependencies, working directory, environment variables, credentials, network access, filesystem permissions, and PATH. Use a clean reproducible environment rather than relying on the developer machine.

---

## 4. Scenario: Script Works in Jenkins but Fails in GitHub Actions

Compare runner OS, Python version, dependency installation, OIDC/IAM configuration, repository permissions, secrets, network restrictions, and command paths. Keep the Python CLI portable and configuration-driven.

---

## 5. Scenario: Python Cannot Import a Package

Check active interpreter, virtual environment, package installation, dependency lockfile, Python version compatibility, and CI installation step. Do not simply install the package manually on the runner.

---

## 6. Scenario: Dependency Upgrade Breaks Automation

Identify the changed transitive dependency, reproduce in a clean environment, inspect compatibility/release notes, run tests, pin/lock the known-good version, and schedule a controlled upgrade.

---

## 7. Scenario: API Returns 401

Verify token/credential source, expiration, audience, role identity, endpoint, and environment. Do not respond by broadly increasing permissions.

---

## 8. Scenario: API Returns 403

Treat it as authorization rather than a transient error. Check exact identity and required permission, then apply least-privilege policy changes through the approved process.

---

## 9. Scenario: API Returns 429

Reduce concurrency, respect Retry-After when available, apply exponential backoff with jitter, use pagination/caching, and enforce a request budget.

---

## 10. Scenario: API Returns 500/503

Classify it as potentially transient, retry with bounded backoff, capture response metadata, and stop when the overall deadline is reached.

---

## 11. Scenario: API Request Hangs

Check DNS, network path, proxy, TLS, remote service latency, and client timeout configuration. Add explicit connection/read and workflow-level deadlines.

---

## 12. Scenario: API Response Is Malformed

Do not assume the response schema. Validate status/content type/required fields, capture safe diagnostics, classify the failure as non-retryable unless evidence shows otherwise.

---

## 13. Scenario: External API Changes Its JSON Structure

Use schema validation, contract tests, versioned clients, defensive parsing, and explicit handling for missing required fields. Avoid silently treating missing data as healthy.

---

## 14. Scenario: Duplicate Webhook Arrives

Use an event ID or release ID as an idempotency key. Check whether the event was already processed before performing mutations.

---

## 15. Scenario: Webhook Signature Fails

Reject the request, log safe metadata, and investigate secret/signature configuration. Never process an unverified privileged webhook.

---

## 16. Scenario: Webhook Replay Attack

Validate timestamp/nonce/event ID and reject events outside the accepted window or already processed events.

---

## 17. Scenario: Python Uses Too Much Memory

Measure process RSS and Python allocations, inspect large API responses, queues, caches, retained references, and reports. Use tracemalloc and incremental processing instead of guessing.

---

## 18. Scenario: Python Container Gets OOMKilled

Inspect container termination reason, memory limit, workload size, object retention, response payloads, and concurrency. Fix the memory behavior before simply increasing limits.

---

## 19. Scenario: Python Script Is Very Slow

Measure stage durations, API latency, subprocess time, serialization, CPU usage, and object counts. Optimize the actual bottleneck rather than rewriting working code.

---

## 20. Scenario: AWS Health Check Takes 30 Minutes

Look for sequential API calls and N+1 patterns. Use pagination, filtering, caching, and bounded concurrency while respecting AWS throttling.

---

## 21. Scenario: AWS API Gets Throttled

Inspect request volume and concurrency. Add SDK-aware retries, exponential backoff/jitter, pagination, caching, and a per-run request budget.

---

## 22. Scenario: AWS Resource Not Found

Verify account, region, resource ID, resource lifecycle state, and eventual consistency. Never assume the resource is absent without checking the target environment.

---

## 23. Scenario: Automation Is Modifying the Wrong AWS Account

Stop immediately. Verify STS identity, expected account/role mapping, region, environment configuration, and CI credential source. Fail closed before further mutation.

---

## 24. Scenario: AWS Credentials Expire Mid-Run

Use supported short-lived credential mechanisms and refresh behavior. Break the workflow into bounded operations so a stale credential does not leave an untracked partial mutation.

---

## 25. Scenario: EKS Automation Targets Wrong Cluster

Stop before mutation. Verify AWS account/region and Kubernetes server/cluster identity against the environment mapping. Do not rely only on kubeconfig context name.

---

## 26. Scenario: Kubernetes API Returns Forbidden

Inspect ServiceAccount/RBAC identity, namespace, resource, and verb. Grant only the missing permission if justified; do not use cluster-admin as a workaround.

---

## 27. Scenario: Kubernetes API Is Slow

Check API-server load, request count, cluster scale, watch/list behavior, pagination, and concurrency. Reduce unnecessary cluster-wide polling.

---

## 28. Scenario: Pod Is CrashLoopBackOff

Inspect previous logs, `describe` events, termination reason, exit code, OOMKilled status, probes, configuration, Secrets/ConfigMaps, and dependency connectivity before restarting.

---

## 29. Scenario: Pod Is OOMKilled

Check memory usage pattern, requests/limits, application behavior, large payloads, caches, and recent releases. Determine whether to optimize code, adjust resources, or both.

---

## 30. Scenario: Pod Stuck Pending

Inspect scheduler events, resource requests, node capacity, taints/tolerations, affinity, quotas, and PVC availability.

---

## 31. Scenario: ImagePullBackOff

Validate exact image/tag/digest, registry existence, node network access, imagePullSecrets/identity, registry permissions, and recent image changes.

---

## 32. Scenario: Deployment Rollout Hangs

Inspect Deployment conditions, ReplicaSets, Pod events, readiness probes, scheduling, image pulls, application startup, and resource constraints. Use a rollout deadline.

---

## 33. Scenario: Deployment Rolls Out but Traffic Fails

Check Pod readiness, Service selector, EndpointSlices, Service ports, Ingress, target health, DNS/TLS, and application response.

---

## 34. Scenario: Service Has No Endpoints

Compare Service selector labels with Pod labels, then check Pod readiness and EndpointSlices.

---

## 35. Scenario: ALB Targets Are Unhealthy

Correlate Ingress configuration, Service/NodePort path, target group health reason, readiness, security controls, application listener, and response code.

---

## 36. Scenario: Ingress Returns 404

Verify Ingress host/path rules, controller configuration, Service backend, application route, DNS, and whether the request reaches the expected load balancer.

---

## 37. Scenario: Ingress Returns 502/503

Check target health, Service endpoints, readiness, application port/listener, network/security controls, and controller events.

---

## 38. Scenario: Kubernetes Events Show FailedScheduling

Inspect resource requests, node allocatable capacity, taints, affinity, topology constraints, quotas, and recent cluster changes.

---

## 39. Scenario: Kubernetes Node NotReady

Inspect node conditions, kubelet/runtime health, resource pressure, networking, recent node changes, and workloads affected before taking remediation.

---

## 40. Scenario: Kubernetes Cleanup Script Deletes Too Much

Stop the job, verify scope, inspect audit logs, protect production namespaces/resources, and redesign with allowlists, dry-run, age filters, and environment identity checks.

---

## 41. Scenario: Python Cleanup Job Must Run in EKS

Use a dedicated Job/CronJob, ServiceAccount with minimal RBAC, EKS workload identity for AWS access, resource limits, deadlines, concurrency control, dry-run, and audit logging.

---

## 42. Scenario: CronJob Runs Twice

Check schedule/controller behavior and overlapping execution. Add concurrency policy and application-level idempotency/locking.

---

## 43. Scenario: CronJob Gets Stuck

Inspect Job/Pod status, logs, resource limits, external API waits, and deadlines. Configure active deadline/backoff appropriately.

---

## 44. Scenario: ArgoCD Shows OutOfSync

Identify the Git desired state and live-state difference, determine whether the drift is expected, and reconcile through Git rather than ad-hoc kubectl changes.

---

## 45. Scenario: ArgoCD Sync Fails

Check application operation state, revision, Kubernetes events, admission policies, image availability, RBAC, and manifest errors. Fix the desired state or policy issue.

---

## 46. Scenario: ArgoCD Says Healthy but Application Is Broken

Validate application-level smoke tests, dependency health, Service routing, and user-visible behavior. Controller health alone is not sufficient.

---

## 47. Scenario: Direct kubectl Change Conflicts with ArgoCD

Determine the intended owner, revert the ad-hoc change if appropriate, and commit the desired state to Git. Avoid creating two competing control loops.

---

## 48. Scenario: GitOps Deployment Uses Wrong Image

Verify Git commit, image tag/digest, registry artifact, ArgoCD revision, rendered manifest, and running Pod image ID. Trace the immutable artifact identity end-to-end.

---

## 49. Scenario: CI Build Succeeds but Deployment Fails

Separate CI artifact success from deployment success. Verify registry, GitOps change, ArgoCD sync, Kubernetes rollout, and smoke tests.

---

## 50. Scenario: Jenkins Job Hangs on Python Command

Check subprocess/network waits, missing timeout, deadlock, input waiting, and external API calls. Add stage-specific and overall deadlines.

---

## 51. Scenario: Python Subprocess Returns Nonzero

Capture command, exit code, safe stderr, environment context, and timeout state. Classify the failure instead of hiding it behind a generic exception.

---

## 52. Scenario: subprocess Shell Injection Risk

Replace shell string construction with an argument list, validate allowed input, and avoid shell=True for untrusted data.

---

## 53. Scenario: Terraform Command Fails

Inspect exact command/exit code, provider/configuration errors, credentials, state/lock behavior, and whether the operation partially changed infrastructure.

---

## 54. Scenario: Terraform Plan Shows Unexpected Destroy

Stop apply, verify state/variables/provider versions, identify why the resource is being replaced/destroyed, and require appropriate approval before any destructive action.

---

## 55. Scenario: Terraform Apply Times Out

Do not blindly rerun. Determine whether Terraform is still running, inspect state/remote resource status, then reconcile actual state before retrying.

---

## 56. Scenario: Terraform State Is Locked

Identify the lock owner/run, determine whether it is active, and use the supported state-lock recovery process. Never remove a valid lock casually.

---

## 57. Scenario: Terraform Drift Detected

Generate/inspect plan, classify intended versus unexpected drift, and restore desired state through the approved Terraform workflow.

---

## 58. Scenario: Git Push Fails Because Remote Changed

Fetch current remote state, inspect differences, preserve unrelated changes, and update safely. Do not force-push production branches.

---

## 59. Scenario: Python Commit Automation Creates Duplicate Commits

Make the update idempotent: inspect current content/revision first and commit only when the desired state differs.

---

## 60. Scenario: Git Working Tree Is Dirty

Stop if the tool expects a clean repository. Do not silently overwrite or reset unrelated changes.

---

## 61. Scenario: SonarQube Gate Fails

Capture project/version and quality-gate evidence, stop promotion, fix the code/policy issue or use an approved exception, and rerun the gate.

---

## 62. Scenario: Trivy Finds Critical Vulnerability

Stop promotion according to policy, identify package/image layer, update/replace the dependency or image, rebuild, rescan, and preserve evidence.

---

## 63. Scenario: Veracode Scan Fails

Treat the result as a security gate failure, inspect findings and policy, remediate or follow an approved exception process, and do not silently bypass the gate.

---

## 64. Scenario: Dependency Vulnerability Is False Positive

Document exact package/version/finding and evidence, use the organization's approved exception process if appropriate, and set expiry/review rather than permanently suppressing it.

---

## 65. Scenario: Secret Appears in Python Logs

Immediately stop further exposure, rotate/revoke the credential, assess access/use, remove or restrict logs where possible, and fix the logging path.

---

## 66. Scenario: AWS Key Is Committed to Git

Revoke/rotate it immediately, inspect usage, remove it from current source and repository history according to incident procedure, and migrate to short-lived identity.

---

## 67. Scenario: Python Logs Environment Variables

Treat as a secret exposure. Remove environment dumping, rotate exposed credentials, and add tests/scanning to prevent recurrence.

---

## 68. Scenario: Production Approval Is Missing

Fail closed. Do not deploy based on an assumption that approval exists.

---

## 69. Scenario: Approval Is for an Older Image

Invalidate the approval because the artifact identity changed. Re-run security and release validation.

---

## 70. Scenario: Release Runs Twice

Use release ID/idempotency key and environment lock, inspect actual deployed state, and avoid duplicate mutation.

---

## 71. Scenario: Two Production Deployments Start Together

Acquire a deployment lock or reject one run. Do not allow concurrent releases to mutate the same environment without a deliberate concurrency design.

---

## 72. Scenario: Deployment Partially Succeeds

Determine actual state first, compare workflow checkpoints, continue only idempotent stages, or rollback according to policy.

---

## 73. Scenario: Python Process Crashes Mid-Deployment

Use persisted state and external system state to determine which stages completed. Never assume a crash means nothing changed.

---

## 74. Scenario: Rollback Fails

Stop automatic rollback loops, preserve evidence, verify current state, escalate, and use a controlled recovery procedure.

---

## 75. Scenario: Rollback Causes Database Problem

Check migration compatibility. If schema changes are not backward compatible, application rollback alone may be unsafe.

---

## 76. Scenario: Health Check Says Healthy but Users Report Failure

Validate user-visible smoke tests, ALB/Ingress routing, dependency health, application logs, and business metrics. A narrow health check may be incomplete.

---

## 77. Scenario: Prometheus Is Down During Deployment

Do not treat missing metrics as healthy. Classify observability as degraded and apply the release policy for insufficient verification evidence.

---

## 78. Scenario: ELK Is Delayed

Use local/CI logs and other telemetry where available, note the observability delay, and avoid assuming absence of logs means absence of errors.

---

## 79. Scenario: High Metric Cardinality

Remove dynamic labels such as commit SHA, image digest, Pod UID, or arbitrary exception text from Prometheus labels and place that detail in logs.

---

## 80. Scenario: Python Health Checker Sends Too Many Requests

Measure request rate, add filtering/pagination/cache, bound concurrency, introduce request budgets, and tune polling frequency.

---

## 81. Scenario: Health Checker Produces Duplicate Alerts

Add stable incident keys and state-transition logic so repeated observations update one incident instead of creating many.

---

## 82. Scenario: Health Signal Flaps

Require sustained failure or multiple observations and use a cooldown before remediation.

---

## 83. Scenario: Dependency Failure Creates 20 Alerts

Build dependency correlation so downstream symptoms reference the upstream failed component rather than triggering independent root causes.

---

## 84. Scenario: API Is Eventually Consistent

Use state polling with exponential backoff and a deadline. Do not use arbitrary fixed sleeps as the only synchronization mechanism.

---

## 85. Scenario: Retry Makes Incident Worse

Check whether the error is actually transient, whether the operation is idempotent, and whether multiple retry layers are multiplying traffic. Add retry budgets.

---

## 86. Scenario: Retry Storm

Bound worker concurrency, add exponential backoff/jitter, honor Retry-After, and use circuit breaking when a dependency remains unavailable.

---

## 87. Scenario: One Dependency Consumes All Workers

Use bulkhead isolation or separate worker pools/queues so unrelated checks remain available.

---

## 88. Scenario: Queue Memory Keeps Growing

Apply backpressure with bounded queues, reduce producer rate, increase controlled consumer capacity, and investigate downstream latency.

---

## 89. Scenario: Python Threads Do Not Improve CPU Work

The workload is likely CPU-bound. Profile it and consider multiprocessing/native optimized libraries rather than adding more threads.

---

## 90. Scenario: Python Threads Improve API Health Checks

The workload is I/O-bound, so bounded threads can overlap network waits. Still enforce service-specific concurrency limits.

---

## 91. Scenario: Asyncio Service Becomes Unresponsive

Look for blocking synchronous operations running on the event loop. Move blocking work to an executor or use async-compatible clients.

---

## 92. Scenario: Memory Grows During Large AWS Scan

Inspect retained resource objects, report accumulation, caches, and queues. Stream/process pages incrementally and retain only normalized results.

---

## 93. Scenario: API Scan Has N+1 Calls

Use bulk/list APIs, include required fields, cache stable metadata, or apply bounded concurrency after verifying service limits.

---

## 94. Scenario: Python Script Has High CPU

Profile with cProfile/time-based measurements, inspect parsing/serialization/loops, and optimize the measured hotspot.

---

## 95. Scenario: Production Script Has No Logs

Add structured logging at stage boundaries, external calls, retries, decisions, and final outcome. Never solve missing observability by logging secrets.

---

## 96. Scenario: Logs Are Too Noisy

Move diagnostics to DEBUG, aggregate repetitive events, use structured fields, and keep INFO focused on workflow milestones.

---

## 97. Scenario: Log Contains Secret

Redact before emission, rotate the exposed credential, and add tests/scanners to prevent recurrence.

---

## 98. Scenario: Python Tool Returns Exit Code 0 After Failure

Fix exception handling so failures propagate to the CLI boundary and map to a nonzero exit code. Never swallow exceptions and report false success.

---

## 99. Scenario: Broad except Hides Root Cause

Replace broad handlers with specific exception classification, preserve original causes, and log safe traceback information.

---

## 100. Scenario: Production Tool Needs a New Feature

Add it behind clear interfaces/configuration, write unit/integration tests, update permissions and documentation, and deploy through the normal release process.

---

## 101. Scenario: Production Tool Needs Emergency Fix

Use the approved emergency/change process, minimize scope, preserve audit evidence, test the smallest safe change, and verify production behavior.

---

## 102. Scenario: Python Container Has Security Findings

Identify vulnerable packages/base image components, update to supported versions, rebuild and rescan, and avoid suppressing findings without evidence.

---

## 103. Scenario: Container Runs as Root

Change to non-root where possible, adjust filesystem permissions/capabilities, rebuild, test, and enforce the requirement through policy.

---

## 104. Scenario: Python Job Needs AWS Access in EKS

Use the supported EKS workload identity mechanism and a dedicated ServiceAccount/IAM role with least privilege; never mount static AWS keys.

---

## 105. Scenario: Python Job Needs Kubernetes Write Access

Separate read-only health identity from mutation identity and grant only the required resources/verbs.

---

## 106. Scenario: Python Cleanup Needs Production Access

Use explicit allowlists, dry-run, environment validation, approval, narrow RBAC/IAM, and audit logging.

---

## 107. Scenario: Python Receives Arbitrary URL

Treat it as an SSRF risk. Allowlist destinations/schemes, restrict network access, and block metadata/internal addresses.

---

## 108. Scenario: Python Receives Arbitrary File Path

Validate and normalize the path and ensure it remains under an approved root to prevent path traversal.

---

## 109. Scenario: Python Receives User-Supplied Shell Argument

Validate allowed characters/format and pass it as a separate subprocess argument; never interpolate it into a shell command.

---

## 110. Scenario: Untrusted Pickle File

Do not deserialize it. Use JSON or another safe format with schema validation.

---

## 111. Scenario: YAML Input Is Untrusted

Use a safe YAML loader and validate the resulting structure; do not use unsafe object construction.

---

## 112. Scenario: Production API Certificate Verification Fails

Do not disable TLS verification. Check CA bundle, hostname, certificate chain, proxy, and server configuration.

---

## 113. Scenario: CI Runner Has Production Credentials

Remove broad credentials from untrusted jobs, use environment protections and OIDC, and isolate privileged deployment runners.

---

## 114. Scenario: PR Code Can Trigger Production

Require protected environments/branches, trusted workflow boundaries, approvals, and restricted credentials. Untrusted PR code must not inherit production identity.

---

## 115. Scenario: Artifact Tag Was Reused

Use image digests/immutable artifacts and verify the digest during deployment.

---

## 116. Scenario: Running Pod Has Unexpected Image

Compare Git revision, ArgoCD revision, rendered manifest, image tag/digest, and actual container image ID.

---

## 117. Scenario: Build Artifact Differs Across Environments

Verify that the same immutable digest is promoted. Do not rebuild for each environment.

---

## 118. Scenario: Release Evidence Is Missing

Fail closed when required security/approval/deployment evidence cannot be verified.

---

## 119. Scenario: Audit Team Asks Who Deployed

Use correlation ID and release metadata to connect CI identity, approval, source SHA, artifact digest, GitOps commit, ArgoCD revision, and deployment result.

---

## 120. Scenario: Production Script Cannot Be Reproduced

Capture Python version, dependency lock, container/base image, configuration schema, exact source SHA, and external tool versions.

---

## 121. Scenario: Python Version Upgrade

Run unit/integration tests, validate dependency compatibility, rebuild the production image, test external SDK behavior, and roll out gradually.

---

## 122. Scenario: Kubernetes Client Version Upgrade

Check Kubernetes server compatibility, deprecated APIs, test against representative clusters, and roll out the client change separately.

---

## 123. Scenario: boto3 Upgrade

Test credential behavior, retry configuration, service API interactions, and affected automation before promotion.

---

## 124. Scenario: Terraform Provider Upgrade

Review provider release changes, regenerate plans in a safe environment, inspect resource behavior, and require controlled approval.

---

## 125. Scenario: ArgoCD Version Upgrade

Test API compatibility, sync behavior, health checks, RBAC, and GitOps reconciliation before production rollout.

---

## 126. Scenario: Production Python Automation Is Becoming One Giant Script

Refactor by responsibility: CLI/config, clients, domain/policy, orchestration, reporting, and observability. Add tests before moving behavior.

---

## 127. Scenario: Developer Wants to Add a Direct kubectl Shortcut

Evaluate ownership and safety. If ArgoCD owns the resource, prefer updating Git or providing a controlled operational workflow rather than bypassing reconciliation.

---

## 128. Scenario: Developer Wants AdministratorAccess

Reject broad permissions. Identify exact API actions/resources and implement least privilege.

---

## 129. Scenario: Developer Wants to Disable Trivy for a Release

Do not bypass the gate informally. Use remediation or a documented, time-bound approved exception.

---

## 130. Scenario: Developer Wants to Ignore Terraform Destroy

Do not ignore it. Investigate why replacement/destruction is planned and require appropriate approval.

---

## 131. Scenario: Production Alert Is Probably a False Positive

Gather evidence, correlate dependencies, inspect recent changes, and tune the health rule only after proving the false-positive condition.

---

## 132. Scenario: Automated Remediation Keeps Restarting Pods

Implement attempt limits/cooldowns, stop repeated remediation, investigate root cause, and escalate.

---

## 133. Scenario: Deployment Health Check Times Out

Inspect rollout progress, scheduler/events, readiness, dependencies, API latency, and deadline budget. Do not automatically rollback without evidence.

---

## 134. Scenario: Smoke Test Fails Once

Check whether the failure is deterministic or transient, use a short bounded retry if the operation is safe, then fail if the policy threshold is exceeded.

---

## 135. Scenario: Smoke Test Passes but Error Rate Rises

Use observability signals and baseline comparison; smoke success is necessary but not sufficient for release health.

---

## 136. Scenario: Canary Looks Healthy but Downstream Dependency Is Degraded

Do not increase traffic blindly. Treat dependency health as part of release risk and follow the defined gate policy.

---

## 137. Scenario: Production Deployment Needs Zero Downtime

Use rolling/canary/blue-green strategy appropriate to workload, readiness probes, capacity planning, backward-compatible migrations, and health verification.

---

## 138. Scenario: Python Must Notify Slack/Email After Deployment

Treat notification as a separate non-critical dependency unless policy requires it. Deployment success should not become false solely because a notification service failed.

---

## 139. Scenario: Notification Failure Hides Deployment Result

Persist the release result before sending notification so the outcome remains queryable even if the notification channel is unavailable.

---

## 140. Scenario: CI Job Is Retried After Timeout

Verify whether the previous job actually completed or partially changed state before retrying. Use idempotency/checkpoints to prevent duplicate mutations.

---

## 141. Scenario: Kubernetes Job Is Retried After Controller Failure

Inspect whether the previous Pod performed the mutation before allowing a retry. Make the operation idempotent.

---

## 142. Scenario: Python Process Dies After API Mutation

The external mutation may have succeeded. Query actual state before retrying.

---

## 143. Scenario: External System Has No Idempotency Key

Use a stable state comparison, lock, transaction-like workflow, or reconciliation approach to avoid duplicate effects.

---

## 144. Scenario: Release Needs Human Approval

Generate a deterministic release plan/evidence package, bind approval to exact artifact/environment, then invalidate approval if inputs change.

---

## 145. Scenario: Production Rollback Is Not Safe

Stop automatic rollback and escalate with current-state evidence, especially when database/schema compatibility is uncertain.

---

## 146. Scenario: Database Migration Is Irreversible

Separate migration strategy from application deployment and use expand/contract compatibility patterns where possible.

---

## 147. Scenario: Python Automation Needs Secrets

Retrieve secrets through approved secret-management/workload identity mechanisms and keep them out of source, logs, arguments, and artifacts.

---

## 148. Scenario: Python Automation Needs Temporary Credentials

Use short-lived credentials with limited scope and duration, and ensure the workflow can refresh or terminate safely.

---

## 149. Scenario: Python Automation Runs for Hours

Use checkpoints, heartbeats, deadlines, credential refresh strategy, bounded memory, progress metrics, and graceful shutdown.

---

## 150. Scenario: Long-Running Workflow Loses Network

Classify the failure, retry only safe operations, preserve checkpoint, and continue only after dependency recovery within the workflow deadline.

---

## 151. Scenario: Long-Running Workflow Loses CI Runner

Persist state externally so a replacement worker can determine actual progress.

---

## 152. Scenario: Two Workers Process Same Release

Use a distributed lock/idempotency key and verify ownership before mutation.

---

## 153. Scenario: Health Checker Sees No Pods

First verify namespace/cluster identity, permissions, selectors, and API response; empty result may be a configuration error rather than a healthy cluster.

---

## 154. Scenario: Health Checker Reports Everything Unknown

Check credentials/RBAC, API connectivity, configuration, observability dependencies, and client version before trusting the result.

---

## 155. Scenario: Monitoring System Is Down

Report UNKNOWN/degraded observability and use other evidence; do not silently convert monitoring failure into application health.

---

## 156. Scenario: Python Script Produces Different Results in Two Environments

Compare Python/runtime versions, dependencies, timezone/locale, configuration, API versions, permissions, and input data.

---

## 157. Scenario: Timezone Bug in Scheduled Automation

Use timezone-aware datetimes, define schedule timezone explicitly, and prefer UTC internally for timestamps and audit data.

---

## 158. Scenario: Daylight Saving Changes Behavior

Avoid naive local timestamps for business-critical scheduling. Store timezone-aware values and test boundary transitions.

---

## 159. Scenario: Clock Skew Affects Webhook Validation

Use tolerant timestamp windows and trusted server time where possible; reject stale/replayed events without assuming exact client clock accuracy.

---

## 160. Scenario: Large Log File Crashes Parser

Process line-by-line or stream chunks, aggregate only required information, and avoid loading the whole file into memory.

---

## 161. Scenario: Parser Gets Malformed Lines

Treat individual malformed records according to policy, preserve counts/evidence, and avoid crashing the entire batch unless data integrity requires it.

---

## 162. Scenario: Batch Has 1 Failed Item Out of 10,000

Use partial-result handling if items are independent, record failed item IDs/errors, retry only safe transient failures, and define final success criteria.

---

## 163. Scenario: Batch Has One Critical Security Failure

Stop promotion/mutation according to policy even if other checks succeed. Security-critical failures should not be hidden by aggregate success.

---

## 164. Scenario: Python API Client Has N+1 Pattern

Redesign around bulk/list APIs, caching, pagination, or bounded concurrency; measure request reduction before/after.

---

## 165. Scenario: Python Automation Needs to Scale 100x

Identify API/request/memory/CPU bottlenecks, partition work, add bounded concurrency and backpressure, reduce payloads, and enforce quotas/budgets.

---

## 166. Scenario: Automation Is Costly

Measure API calls, runtime, runner/container resources, duplicate work, and log volume; optimize repeated calls and scheduling frequency without reducing safety.

---

## 167. Scenario: Python Tool Has No Tests

Start with pure policy/normalization unit tests, then mock external APIs, add failure-path tests, and finally integration tests.

---

## 168. Scenario: Tests Pass but Production Fails

Check whether tests mocked too much. Add contract/integration tests for real API schemas, authentication, permissions, and controller behavior.

---

## 169. Scenario: Mock Hides Real AWS Error

Use realistic exception/status fixtures and periodic integration tests against isolated AWS resources.

---

## 170. Scenario: Kubernetes Mock Hides API Behavior

Add contract/integration tests against a real test cluster or representative API server version.

---

## 171. Scenario: Production Bug Has No Reproduction

Preserve correlation ID, source SHA, configuration version, API response metadata, safe logs, and external state; build a minimal reproduction from evidence.

---

## 172. Scenario: Root Cause Is Unknown

Do not invent certainty. Classify the incident as unresolved/unknown, preserve evidence, reduce impact safely, and continue investigation.

---

## 173. Scenario: Multiple Possible Root Causes

Rank hypotheses using dependency relationships, recent changes, timing, severity, and direct evidence. Test the least risky/highest-value hypothesis first.

---

## 174. Scenario: Recent Deployment Is Suspected

Compare deployment timeline with metrics/logs/events, verify whether rollback would be safe, and avoid assuming correlation equals causation.

---

## 175. Scenario: No Recent Deployment but Failure Started

Investigate dependency/configuration/infrastructure/external changes, not only application code.

---

## 176. Scenario: Python Health Check Is a Single Point of Failure

Keep monitoring/verification independent where possible, use redundant execution, bounded failure handling, and avoid making one automation process the only source of truth.

---

## 177. Scenario: Release Controller Itself Fails

Externalize workflow state and ensure deployment ownership remains with GitOps/infrastructure systems so controller failure does not erase actual desired/observed state.

---

## 178. Scenario: Python Script Is Used by Multiple Teams

Define a stable CLI/API contract, version releases, document configuration, maintain compatibility, and provide deprecation paths.

---

## 179. Scenario: Breaking CLI Change

Version the interface or provide backward-compatible arguments, update consumers, test both paths, and communicate deprecation before removal.

---

## 180. Scenario: Python Tool Needs Configuration for Dev/Staging/Prod

Use one codebase with environment-specific configuration and explicit target validation rather than separate scripts with duplicated logic.

---

## 181. Scenario: Production Config Is Missing

Fail fast at startup with a clear safe error. Never substitute an unsafe default such as production credentials or broad permissions.

---

## 182. Scenario: User Gives Invalid Environment

Reject it against an allowlist. Never construct account/cluster/resource names directly from arbitrary input without validation.

---

## 183. Scenario: User Gives Production as a Flag

Require stronger confirmation/approval and still validate the actual target identity.

---

## 184. Scenario: Dry Run Is Inaccurate

Ensure dry-run uses the same discovery/policy logic as execution but prevents mutations. Test that no write API is called.

---

## 185. Scenario: Dry Run Accidentally Changes State

Treat it as a severe bug: stop use, identify mutation path, add tests that assert zero writes, and fix before re-enabling.

---

## 186. Scenario: Python Tool Needs a Rollback Command

Rollback must target a known-good immutable revision and verify compatibility/health; it should not mean simply 'run the previous script'.

---

## 187. Scenario: Release Metadata Is Lost

Use durable storage/artifacts for release evidence and include enough identifiers to reconstruct source/artifact/deployment lineage.

---

## 188. Scenario: Audit Requires Exact Deployment Identity

Record source SHA, image digest, GitOps commit, ArgoCD revision, Kubernetes deployment revision, identity, approval, and timestamps.

---

## 189. Scenario: Production Incident During Deployment

Stabilize first, determine blast radius, pause further promotion, collect evidence, decide rollback/mitigation, verify recovery, and only then resume normal delivery.

---

## 190. Scenario: Incident Requires Emergency Change

Use a narrow, reversible, authorized change; preserve before/after evidence and follow with root-cause/remediation work.

---

## 191. Scenario: Python Tool Is the Root Cause

Disable or isolate the automation if needed, stop repeated mutations, recover the environment safely, preserve evidence, and add guardrails/tests before reactivation.

---

## 192. Scenario: Automation Deletes Production Data

Stop further execution, assess scope, preserve logs/audit evidence, restore from approved recovery mechanisms, rotate compromised credentials if applicable, and conduct a post-incident review.

---

## 193. Scenario: Python Tool Has Excessive Permissions

Inventory actual API usage, create least-privilege IAM/RBAC policy, test in lower environment, then replace broad permissions.

---

## 194. Scenario: Python Tool Is Compromised

Revoke credentials, isolate runner/container, inspect source/artifact integrity, review audit logs, rotate affected secrets, and redeploy from a trusted source.

---

## 195. Scenario: Supply Chain Dependency Is Compromised

Freeze affected builds, identify versions/artifacts, rotate potentially exposed credentials, verify package integrity, update to trusted versions, and review build provenance.

---

## 196. Scenario: Final Interview Scenario

When given any production Python scenario, avoid saying 'I would restart it' as the first response. Start with scope and evidence, verify identity/target, classify the failure, choose the smallest safe action, verify recovery, and explain preventive controls.

---

## Section Progress

```text
12-Python-Interview-Preparation/
├── 01-Basic-Questions.md              ✓
├── 02-Intermediate-Questions.md       ✓
├── 03-Advanced-Questions.md           ✓
├── 04-Python-for-DevOps-Questions.md  ✓
├── 05-Scenario-Based.md               ✓
├── 06-Coding-Questions.md
├── 07-Production-Scenarios.md
└── 08-Mock-Interview.md
```

**Next file: `06-Coding-Questions.md`**