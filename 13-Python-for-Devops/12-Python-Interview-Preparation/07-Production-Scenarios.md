# Production Scenarios

> Senior-level production scenarios for Python DevOps/DevSecOps interviews. Focus: real incidents, safe automation, AWS/EKS/Kubernetes, CI/CD, GitOps, Terraform, security, observability, concurrency, recovery, and operational decision-making.

## Production Scenario Answer Framework

**Impact → Target identity → Evidence → Failure domain → Mitigation → Verification → Root cause → Prevention**

### Core principle

Do not begin with a restart, retry, permission escalation, `kubectl delete`, `terraform apply`, or rollback. First establish what is happening and whether the proposed action is safe.

## 1. Production Scenario Framework

Answer production scenarios in this order: impact and blast radius → target identity → evidence → recent changes → dependency analysis → mitigation → verification → root cause → prevention. The first goal is safe stabilization, not proving a theory.

---

## 2. Scenario: Production Python Automation Is Down

Determine whether the automation itself is unavailable or its dependencies are unavailable. Check process/container status, logs, metrics, credentials, API connectivity, recent release changes, and persisted workflow state.

---

## 3. Scenario: Automation Is Running but No Work Is Completing

Inspect queue depth, worker status, external API latency, blocked threads/tasks, retry loops, deadlocks, and timeout configuration. Compare throughput with arrival rate.

---

## 4. Scenario: Automation Is Stuck in Retry

Identify the underlying error and whether it is retryable. Enforce maximum attempts and an overall deadline; do not allow infinite retries.

---

## 5. Scenario: Production Script Crashes Every Few Minutes

Correlate crash time with inputs, memory, CPU, external calls, configuration, recent deployments, and logs. Capture traceback and process/resource metrics before changing limits.

---

## 6. Scenario: Python Process Memory Slowly Increases

Use tracemalloc/profiling to identify retained objects, caches, queues, large responses, and reference cycles. Compare memory growth with workload volume.

---

## 7. Scenario: Python Process Suddenly Uses 100% CPU

Determine whether the workload is CPU-bound, an infinite loop, excessive parsing, retry storm, or lock contention. Profile or inspect stack traces before restarting.

---

## 8. Scenario: Python Service Has High Latency

Break latency into request parsing, business logic, database/API calls, serialization, queue wait, and external dependencies. Optimize the measured bottleneck.

---

## 9. Scenario: Production API Client Has Rising Latency

Check remote service latency, connection reuse, DNS/TLS, retries, response size, concurrency, and connection pool limits.

---

## 10. Scenario: API Dependency Is Completely Down

Stop aggressive retries, activate circuit-breaking/fallback behavior if supported, preserve queued work safely, and communicate dependency degradation.

---

## 11. Scenario: API Dependency Is Flapping

Use bounded retries, cooldowns, circuit breaker behavior, and dependency health correlation rather than continuously hammering the service.

---

## 12. Scenario: AWS API Throttling Is Affecting Production

Measure request rate and concurrency, inspect pagination/filtering, apply SDK-aware retries/backoff/jitter, cache safe metadata, and introduce a request budget.

---

## 13. Scenario: AWS Account Is Correct but Region Is Wrong

Stop mutation, validate region against environment configuration, inspect clients/resources already touched, and determine whether any corrective action is needed.

---

## 14. Scenario: AWS Role Has Too Much Permission

Inventory actual API calls, create a least-privilege policy, test in a lower environment, then replace the broad role with controlled permissions.

---

## 15. Scenario: AWS Credentials Are Exposed

Revoke/rotate immediately, inspect access logs, identify blast radius, remove the secret from active systems, and move to short-lived workload identity.

---

## 16. Scenario: EC2 Health Monitor Reports Everything Healthy

Validate the monitor itself: permissions, API responses, thresholds, target discovery, stale cache, and metric ingestion. Monitoring failure must not be represented as healthy.

---

## 17. Scenario: EC2 Instance Is Running but Application Is Down

Check system/application health rather than EC2 state alone: ports, process status, logs, resource pressure, dependency connectivity, and load balancer health.

---

## 18. Scenario: EC2 Instance Is Unreachable

Check instance status checks, network path, security controls, route tables, DNS, host resources, and whether the issue affects one instance or the whole environment.

---

## 19. Scenario: S3 Backup Job Reports Success but Backup Is Missing

Verify destination bucket/account/region, object count, object metadata/checksum where applicable, encryption, permissions, and durable job evidence.

---

## 20. Scenario: S3 Backup Runs Twice

Use stable backup/run IDs and destination-state comparison so rerunning does not create incorrect duplicate artifacts or inconsistent metadata.

---

## 21. Scenario: S3 Backup Is Very Slow

Measure object count, object sizes, network throughput, API calls, concurrency, multipart transfer behavior, and destination limits. Use controlled parallelism.

---

## 22. Scenario: EKS Automation Cannot Reach Kubernetes API

Verify AWS/network path, endpoint accessibility, credentials, kubeconfig/in-cluster configuration, DNS/TLS, and Kubernetes API health.

---

## 23. Scenario: EKS Automation Has RBAC Failure

Identify exact ServiceAccount, namespace, resource, and verb. Add only required permissions and test the action before production rollout.

---

## 24. Scenario: EKS Cluster Has API Pressure

Reduce broad polling/list calls, use watches where appropriate, paginate, cache safe metadata, and bound concurrency.

---

## 25. Scenario: EKS Pod Monitor Consumes Too Much CPU

Check cluster-wide polling frequency, object parsing, repeated API calls, and unnecessary processing. Reduce scope and process incrementally.

---

## 26. Scenario: EKS Pod Monitor Consumes Too Much Memory

Avoid storing complete Pod objects and historical data. Normalize required fields and stream/aggregate results.

---

## 27. Scenario: Pod Monitor Misses Short-Lived Failures

Polling can miss transient states. Use Kubernetes watches/events where appropriate and preserve event timestamps for diagnosis.

---

## 28. Scenario: Pod Monitor Generates Too Many Alerts

Deduplicate by stable incident key, correlate dependencies, require sustained failure, and use cooldowns.

---

## 29. Scenario: Kubernetes Cluster Has Many CrashLoopBackOff Pods

Determine whether there is a common image/configuration/dependency/root-cause pattern. Avoid restarting all Pods because mass restarts can increase impact.

---

## 30. Scenario: All Pods Become Unready After Deployment

Compare deployment revision with the previous version, inspect readiness probe failures, dependencies, configuration, and application logs. Consider controlled rollback.

---

## 31. Scenario: Deployment Has Desired Replicas but Traffic Is Zero

Check readiness, Service selectors, EndpointSlices, Ingress/controller, target health, DNS, and traffic routing.

---

## 32. Scenario: Service Endpoints Disappear

Inspect Pod readiness and labels/selectors. A Pod can be Running while not being eligible for Service endpoints.

---

## 33. Scenario: ALB Target Health Fails After Release

Correlate target health reason with readiness, Service port mapping, application listener, security controls, health-check path, and response codes.

---

## 34. Scenario: ALB Returns 504

Investigate target response latency, network path, application timeout, downstream dependency, and ALB timeout settings. Determine whether failure is localized or systemic.

---

## 35. Scenario: ALB Returns 503

Check whether healthy targets/endpoints exist, Service routing, target registration, readiness, and controller state.

---

## 36. Scenario: Ingress Works for One Host but Not Another

Compare DNS, host rules, certificates, paths, backend Services, and controller configuration.

---

## 37. Scenario: Kubernetes Node Has MemoryPressure

Identify memory-consuming Pods, eviction events, allocatable capacity, requests/limits, and recent scheduling changes. Stabilize capacity before changing workloads.

---

## 38. Scenario: Kubernetes Node Has DiskPressure

Inspect filesystem usage, container images, logs, ephemeral storage, and orphaned data. Clean only known-safe resources and address capacity policy.

---

## 39. Scenario: Python Cleanup Removes Wrong Resources

Stop further execution, determine exact deletion scope, restore where possible, rotate credentials if compromise is suspected, and add hard target/allowlist guards.

---

## 40. Scenario: Cleanup Script Has No Dry Run

Before production use, implement a discovery-only mode that produces exact proposed mutations and proves no write API is called.

---

## 41. Scenario: Cleanup Script Uses Broad Delete Permissions

Replace broad permissions with resource-specific permissions where possible and enforce selectors/allowlists in application logic.

---

## 42. Scenario: ArgoCD Application Is OutOfSync

Determine whether drift is expected. Compare Git desired state and live state; reconcile through Git when GitOps ownership applies.

---

## 43. Scenario: ArgoCD Application Is Degraded

Inspect health checks, resource status, Pod events/logs, dependencies, and recent revision changes. Do not assume ArgoCD is the root cause.

---

## 44. Scenario: ArgoCD Sync Is Stuck

Inspect operation phase, controller logs/events, resource health, hooks, admission policies, and Kubernetes API conditions.

---

## 45. Scenario: ArgoCD Sync Succeeds but Deployment Fails

ArgoCD sync success means desired manifests were applied/reconciled; verify Kubernetes rollout and application health separately.

---

## 46. Scenario: ArgoCD Detects Manual Drift

Identify the manual change and ownership. Restore through Git rather than continuing manual mutation.

---

## 47. Scenario: GitOps Repository Is Unavailable

Do not invent desired state. Existing cluster state can continue running; deployment automation should fail safely until the source of truth is available.

---

## 48. Scenario: GitOps Commit Is Wrong

Stop promotion, identify exact commit and artifact identity, correct the repository through normal review, and verify ArgoCD reconciles to the intended revision.

---

## 49. Scenario: Git Repository Has a Conflict During Promotion

Do not force-push. Fetch/rebase or regenerate the intended change safely, inspect the diff, and retry only after validating the target revision.

---

## 50. Scenario: Jenkins Pipeline Is Stuck

Identify the exact stage, executor/resource state, Python subprocess, external API, or waiting operation. Use stage timeout and preserve logs.

---

## 51. Scenario: Jenkins Agent Runs Out of Disk

Inspect workspace, Docker layers, artifacts, caches, and logs. Clean only safe paths and add retention/capacity controls.

---

## 52. Scenario: Jenkins Uses Expired Credentials

Identify credential provider and expiration, replace with supported short-lived identity, and avoid embedding credentials in job configuration.

---

## 53. Scenario: GitHub Actions Workflow Cannot Assume AWS Role

Check OIDC permissions, trust policy, repository/branch/environment conditions, role ARN, audience, and workflow identity.

---

## 54. Scenario: GitHub Actions PR Can Access Production Credentials

Treat as a security boundary failure. Remove inherited credentials, protect production environments, and restrict privileged workflows to trusted code.

---

## 55. Scenario: CI Pipeline Builds an Image but ECR Push Fails

Check registry authentication, IAM permissions, repository existence, region/account, network, and image size. Preserve build artifact evidence.

---

## 56. Scenario: ECR Image Tag Already Exists

Prefer immutable digest identity. If tags are mutable by policy, determine whether overwrite is allowed and record the resulting digest.

---

## 57. Scenario: Image Digest Does Not Match Expected

Stop deployment and trace source SHA → build → registry digest → GitOps manifest → running image. Never deploy a mismatched artifact.

---

## 58. Scenario: JFrog Artifact Is Missing

Verify repository, artifact coordinates/version, credentials, network, retention/lifecycle policy, and whether the build published successfully.

---

## 59. Scenario: SonarQube Is Unavailable During Release

Apply release policy for missing quality evidence. Do not silently treat unavailable security/quality checks as passed.

---

## 60. Scenario: Trivy Scanner Is Unavailable

Fail closed if scanning is a mandatory gate. Preserve evidence and retry in a controlled manner rather than bypassing the scan.

---

## 61. Scenario: Veracode Is Delayed

Determine whether policy permits a pending state. If not, stop promotion until required evidence exists.

---

## 62. Scenario: Security Gate Is Blocking Every Release

Check whether the gate configuration, severity threshold, scanner version, or exception policy changed recently. Do not disable security globally as the first fix.

---

## 63. Scenario: Critical Vulnerability Is Discovered After Deployment

Assess exploitability and exposure, identify affected artifacts/environments, mitigate or roll back according to risk, patch/rebuild/rescan, and record the incident.

---

## 64. Scenario: Secret Is Found in a Docker Image

Assume compromise, rotate/revoke the secret, determine image distribution, rebuild without the secret, scan image history/layers, and update secret handling.

---

## 65. Scenario: Secret Is Found in Git History

Rotate first; repository cleanup alone does not invalidate a leaked credential. Then follow approved history-removal and incident procedures.

---

## 66. Scenario: Python Package Is Compromised

Identify affected versions/artifacts, freeze affected builds, verify provenance, remove/upgrade package, rotate potentially exposed credentials, and rebuild from trusted dependencies.

---

## 67. Scenario: Terraform Plan Shows Unexpected Replacement

Stop apply and inspect state, provider version, lifecycle settings, immutable fields, variables, and drift before approving replacement.

---

## 68. Scenario: Terraform Apply Partially Changes Infrastructure

Determine actual resource state and state-file status. Reconcile safely before retrying; never assume the timeout means no changes happened.

---

## 69. Scenario: Terraform Lock Blocks Deployment

Identify active owner/run. If stale, use supported lock recovery with evidence and approval rather than deleting locks casually.

---

## 70. Scenario: Terraform Drift Appears in Production

Determine whether drift was intentional. Restore through Terraform/Git rather than repeated manual changes.

---

## 71. Scenario: Terraform Automation Uses Wrong Workspace/State

Stop before apply, validate backend/state identity and environment mapping, and verify whether any incorrect read/write already occurred.

---

## 72. Scenario: Production Approval Exists but Plan Changed

Invalidate approval and require fresh review because the approved infrastructure state is no longer the same.

---

## 73. Scenario: Python Release Tool Updates Git but ArgoCD Does Not Sync

Verify commit/push, repository/path, ArgoCD application source, revision, credentials, webhook/polling, and controller health.

---

## 74. Scenario: ArgoCD Syncs but New Pods Use Old Image

Check manifest image reference, imagePullPolicy, registry digest, Pod template hash, and actual container image ID.

---

## 75. Scenario: Deployment Rollout Succeeds but Smoke Test Fails

Investigate application route/dependency/configuration. If release correlation is strong and rollback is safe, use the defined rollback policy.

---

## 76. Scenario: Rollback Starts but New Pods Remain

Verify desired revision, ReplicaSets, GitOps reconciliation, Pod availability, and whether another controller/commit is immediately reconciling the forward version.

---

## 77. Scenario: Rollback Is Blocked by Database Migration

Check whether application versions are schema-compatible. Use backward-compatible migration strategies or controlled remediation rather than forcing an unsafe rollback.

---

## 78. Scenario: Canary Has Higher Error Rate

Stop traffic increase, compare with baseline, inspect logs/dependencies, and rollback or hold according to the release threshold.

---

## 79. Scenario: Canary Has Higher Latency but No Errors

Treat latency as a release signal. Investigate CPU, downstream APIs, database, network, and request distribution before promotion.

---

## 80. Scenario: Blue-Green Switch Fails

Keep the known-good environment active if possible, verify routing/control-plane state, and switch only after target health is confirmed.

---

## 81. Scenario: Rolling Deployment Reduces Capacity

Check maxUnavailable/maxSurge, readiness duration, resource requests, node capacity, and PodDisruptionBudget.

---

## 82. Scenario: Deployment Causes Resource Exhaustion

Compare new resource requests/limits with node capacity and workload behavior. Pause rollout and restore capacity or revision safely.

---

## 83. Scenario: Production Smoke Test Is Flaky

Determine whether test/environment/dependency is flaky. Add only bounded retries for genuinely transient failures and improve test determinism.

---

## 84. Scenario: Health Check Depends on the Same Broken Component

Avoid circular verification. Use independent signals where possible, such as external synthetic checks or infrastructure-level health.

---

## 85. Scenario: Monitoring Is Down During Incident

Use remaining telemetry, application logs, Kubernetes events, CI evidence, and direct controlled checks. Mark unknown signals as unknown.

---

## 86. Scenario: Prometheus Has High Cardinality

Identify dynamic labels, remove them from metrics, and move detailed identifiers to structured logs/traces if available.

---

## 87. Scenario: Grafana Shows No Data

Check Prometheus ingestion/query path, scrape targets, labels, time range, dashboard variables, and whether the underlying service is actually emitting metrics.

---

## 88. Scenario: ELK Logs Are Missing

Check application logging, shipping agent, network, indexing, retention, and query filters. Missing logs do not prove no errors occurred.

---

## 89. Scenario: Python Logs Flood ELK

Reduce repetitive INFO logs, aggregate events, adjust levels, and avoid logging large payloads/secrets.

---

## 90. Scenario: Alert Storm

Identify common dependency/root cause, deduplicate by incident key, suppress dependent symptoms, and apply sustained-failure thresholds.

---

## 91. Scenario: Automated Remediation Causes More Failures

Disable or throttle remediation, identify unsafe trigger/action, verify rollback, and require human escalation for repeated failures.

---

## 92. Scenario: Python Retry Storm Overloads Dependency

Stop retries, implement exponential backoff/jitter, request budgets, concurrency limits, and circuit breaker behavior.

---

## 93. Scenario: Queue Backlog Keeps Growing

Compare arrival and processing rates, identify bottleneck, add bounded workers/capacity where safe, reduce producer rate, and prevent unbounded memory growth.

---

## 94. Scenario: Python Worker Pool Is Starved

Inspect blocked workers, external API waits, deadlocks, queue state, and worker exceptions. Add timeouts and independent resource pools.

---

## 95. Scenario: One Dependency Blocks All Automation

Use bulkhead isolation so unrelated environments/stages continue where safe.

---

## 96. Scenario: Long-Running Job Loses Its Runner

Recover using persisted checkpoint and external system state. Never assume previous stages did not execute.

---

## 97. Scenario: Long-Running Job Loses Network

Preserve state, stop unsafe retries, verify actual external mutations, and resume only from a safe checkpoint within deadline.

---

## 98. Scenario: Duplicate Production Jobs Run

Use distributed locking and idempotency keys. Determine whether both have already mutated external systems before stopping one.

---

## 99. Scenario: Production Automation Has No Checkpointing

Add durable stage state and release identity before relying on retries for multi-step workflows.

---

## 100. Scenario: Python Automation Is Not Idempotent

Identify every side effect and convert operations to desired-state comparison, stable IDs, upserts, or reconciliation where possible.

---

## 101. Scenario: External API Has No Idempotency Support

Use state comparison, locking, transaction boundaries, unique operation IDs in your own store, or reconciliation to avoid duplicate side effects.

---

## 102. Scenario: API Is Eventually Consistent

Poll for expected state with bounded backoff and deadline. Make subsequent operations conditional on observed state.

---

## 103. Scenario: API Returns Success but State Is Not Visible

Treat response as accepted, then verify asynchronously. Do not immediately assume the final state exists everywhere.

---

## 104. Scenario: Production Job Runs for Hours

Use checkpoints, heartbeat/progress metrics, memory bounds, credential lifecycle planning, stage deadlines, graceful shutdown, and recovery strategy.

---

## 105. Scenario: Python Process Receives SIGTERM

Stop accepting new work, checkpoint if necessary, finish safe in-flight work within the termination budget, release resources, and exit.

---

## 106. Scenario: Python Process Is Killed Without Graceful Shutdown

Design recovery around persisted state and idempotent operations because cleanup hooks may not execute.

---

## 107. Scenario: Production Config Is Missing

Fail fast and clearly. Never use broad or production-like defaults that could cause unsafe mutations.

---

## 108. Scenario: Wrong Environment Variable Selects Production

Validate configuration against an explicit environment mapping and actual cloud/Kubernetes identity before writes.

---

## 109. Scenario: Production Config Changes Without Review

Move configuration into controlled versioning/change management and validate configuration schema at startup.

---

## 110. Scenario: Python Tool Uses Hard-Coded Credentials

Remove credentials from source, rotate them, and migrate to workload identity/secret management.

---

## 111. Scenario: Python Tool Uses AdministratorAccess

Replace with least-privilege IAM/RBAC based on observed required actions.

---

## 112. Scenario: Python Tool Runs as Root

Determine whether root is actually required. Prefer non-root containers/users and minimal capabilities.

---

## 113. Scenario: Production Container Has Critical CVE

Identify whether the vulnerable component is used/exposed, update base/dependency, rebuild/rescan, and promote through the normal pipeline.

---

## 114. Scenario: Production Runner Is Compromised

Isolate runner, revoke credentials, inspect audit logs/artifacts, rebuild from trusted source, and verify pipeline integrity.

---

## 115. Scenario: Untrusted PR Executes Deployment Code

Separate untrusted build/test execution from privileged deployment workflows. Require protected environments and trusted workflow boundaries.

---

## 116. Scenario: Python Automation Accepts a URL

Treat it as potential SSRF. Validate scheme/host, block internal/metadata destinations, and restrict network egress.

---

## 117. Scenario: Python Automation Accepts a File Path

Resolve the path and enforce an approved root/allowlist to prevent traversal and destructive operations.

---

## 118. Scenario: Python Automation Parses Untrusted YAML

Use safe parsing and schema validation. Never allow arbitrary Python object construction.

---

## 119. Scenario: Python Automation Parses Pickle

Do not deserialize untrusted pickle because it can execute code. Replace it with a safe data format.

---

## 120. Scenario: Python Automation Calls Shell Commands with User Input

Use argument arrays and strict validation; avoid shell=True.

---

## 121. Scenario: Python Automation Logs Secrets

Stop further exposure, rotate leaked credentials, remove unsafe logging, and add tests/scanning.

---

## 122. Scenario: TLS Verification Fails

Do not disable certificate verification. Check CA bundle, hostname, proxy, certificate chain, and server configuration.

---

## 123. Scenario: DNS Fails in EKS

Check CoreDNS, Service/Ingress names, VPC DNS, resolver path, NetworkPolicies/security controls, and whether failure is cluster-wide.

---

## 124. Scenario: External API DNS Fails

Check resolver configuration, VPC/runner DNS, proxy, endpoint hostname, and whether other clients are affected.

---

## 125. Scenario: NetworkPolicy Blocks Python Pod

Inspect namespace/Pod labels, policy selectors, ingress/egress rules, DNS allowance, and required destinations.

---

## 126. Scenario: Python Job Cannot Reach ECR

Check network path, VPC endpoints/NAT where applicable, credentials/identity, region/account, and registry permissions.

---

## 127. Scenario: Python Job Cannot Reach S3

Check IAM, network routing/endpoints/NAT, bucket policy, region, and encryption requirements.

---

## 128. Scenario: Python Job Cannot Reach Kubernetes API

Check in-cluster DNS/service endpoint, network policy, service account token/configuration, API endpoint reachability, and RBAC.

---

## 129. Scenario: Python Job Is OOMKilled in Kubernetes

Inspect resource requests/limits and workload memory. Reduce payload retention/concurrency or increase resources based on evidence.

---

## 130. Scenario: Python Job Is CPU Throttled

Check CPU limit/request and application profile. Optimize CPU hot spots or adjust resource allocation based on workload requirements.

---

## 131. Scenario: Python Job Is Evicted

Inspect node pressure and Pod requests/priority. Improve resource sizing, scheduling, and node capacity rather than only retrying.

---

## 132. Scenario: Kubernetes Job Retries a Non-Idempotent Operation

Stop repeated execution and redesign the operation around idempotency/checkpointing before allowing retries.

---

## 133. Scenario: CronJob Overlaps

Use concurrency policy, lock, or queue semantics according to whether missed executions should be skipped or serialized.

---

## 134. Scenario: Kubernetes Job Has No Logs

Inspect Pod creation/status, container startup, image, command/args, log collection, and whether the container exits before logging.

---

## 135. Scenario: Python Job Cannot Pull Image

Check exact image/digest, registry access, credentials, ServiceAccount/workload identity, node network, and image availability.

---

## 136. Scenario: Python Job Starts with Wrong Image

Verify deployment manifest, GitOps revision, registry digest, and Pod image ID; enforce immutable image verification.

---

## 137. Scenario: Release Uses Mutable Image Tag

Prefer digest-based promotion. If tags are required, record the resolved digest and verify it before deployment.

---

## 138. Scenario: Build Recreates Artifact for Each Environment

Move to build-once/promote-many so the exact scanned artifact is deployed across environments.

---

## 139. Scenario: Release Evidence Is Incomplete

Fail closed for mandatory evidence and persist enough metadata to reconstruct source → artifact → deployment lineage.

---

## 140. Scenario: Approval Is Bound Only to Branch Name

Improve approval identity to exact commit SHA, artifact digest, environment, and relevant plan/security evidence.

---

## 141. Scenario: Audit Cannot Determine Who Changed Production

Correlate CI identity, cloud assumed role, Git commit, approval, ArgoCD revision, and Kubernetes events/deployment revision.

---

## 142. Scenario: Production Change Is Made Manually

Determine whether it was emergency/authorized, capture evidence, reconcile Git/Terraform desired state, and prevent unmanaged drift.

---

## 143. Scenario: Terraform and ArgoCD Both Manage the Same Resource

Separate ownership boundaries. One system should be authoritative for each resource property to avoid competing reconciliation.

---

## 144. Scenario: Python Tool and ArgoCD Both Mutate Kubernetes

Define a single deployment owner. Python should verify/orchestrate around ArgoCD unless the operation is explicitly outside GitOps ownership.

---

## 145. Scenario: Python Tool and Terraform Both Create Same AWS Resource

Establish ownership and state boundaries. Do not allow duplicate resource creation or unmanaged resources.

---

## 146. Scenario: Production Automation Needs Emergency Break-Glass

Use explicit authorization, narrow scope, temporary credentials/permissions, audit logging, and mandatory post-incident reconciliation.

---

## 147. Scenario: Emergency Change Was Successful

Record exact change and evidence, reconcile source of truth, remove temporary access, and create follow-up prevention work.

---

## 148. Scenario: Emergency Change Failed

Stabilize, preserve evidence, verify current state, and avoid repeated uncontrolled attempts.

---

## 149. Scenario: Database Is Unavailable During Deployment

Pause rollout, assess application behavior, verify readiness/dependency policy, and rollback only if schema compatibility makes it safe.

---

## 150. Scenario: Redis/Cache Is Unavailable

Determine whether application can degrade gracefully. Do not automatically declare deployment healthy if critical functionality is impaired.

---

## 151. Scenario: RabbitMQ Is Unavailable

Inspect queue depth, producers/consumers, connection failures, retry behavior, and message durability. Avoid creating a producer retry storm.

---

## 152. Scenario: Notification Service Is Down

Separate notification failure from deployment outcome. Persist the release result and retry notification independently if appropriate.

---

## 153. Scenario: Release Verification Service Is Down

If it is mandatory evidence, stop promotion. If policy allows alternative verification, use independent checks and record degraded verification.

---

## 154. Scenario: Health Check Returns UNKNOWN

Investigate telemetry/API/RBAC/configuration failure first. UNKNOWN should not silently become success.

---

## 155. Scenario: Health Check Returns CRITICAL for Everything

Validate checker configuration, target discovery, threshold units, stale cache, and dependency/API failures before declaring the whole environment unhealthy.

---

## 156. Scenario: Production Incident Has Many Symptoms

Build a timeline and dependency graph. Identify the smallest upstream failure explaining the largest number of symptoms.

---

## 157. Scenario: Recent Change Is the Obvious Suspect

Use evidence to establish correlation and impact. Roll back safely when justified, but continue root-cause analysis afterward.

---

## 158. Scenario: No Recent Application Change

Investigate infrastructure, configuration, certificates, credentials, dependencies, DNS, network, cloud service health, and scheduled changes.

---

## 159. Scenario: Incident Impact Is Growing

Prioritize mitigation: stop rollout, reduce traffic, disable unsafe automation, isolate affected component, or revert a known-safe change.

---

## 160. Scenario: Incident Is Stable but Root Cause Unknown

Maintain current safe state, gather evidence, avoid unnecessary changes, and document unknowns explicitly.

---

## 161. Scenario: Post-Incident Review

Document timeline, impact, detection, root cause/contributing factors, mitigation, recovery, what worked, what failed, and preventive actions with owners.

---

## 162. Scenario: Preventing Repeat Incidents

Convert manual discoveries into automated tests, policy gates, monitoring, runbooks, least-privilege controls, and safe automation.

---

## 163. Scenario: Production Script Needs Observability

Emit structured logs, bounded Prometheus metrics, run/stage correlation IDs, duration, success/failure counts, retry counts, and dependency error categories.

---

## 164. Scenario: Metric Labels Become Huge

Move high-cardinality identifiers into logs. Keep metric labels bounded and based on stable dimensions such as environment/service/status.

---

## 165. Scenario: Logs Contain Full API Payloads

Stop verbose payload logging in production. Log safe summaries, IDs, status codes, latency, and sanitized error details.

---

## 166. Scenario: Production Automation Needs Debugging

Increase logging for a controlled period or target, capture safe diagnostics, reproduce in lower environment, and avoid permanently enabling expensive DEBUG logs.

---

## 167. Scenario: Python Automation Has No Run ID

Introduce a stable run/release correlation ID and propagate it through logs, metrics, reports, GitOps, and notifications.

---

## 168. Scenario: Production Automation Has No Timeout

Add stage-specific and overall deadlines. Every network/subprocess/polling operation should have bounded waiting.

---

## 169. Scenario: Production Automation Has Multiple Retry Layers

Map all retry layers and calculate worst-case delay/request multiplication. Keep retry responsibility clear and bounded.

---

## 170. Scenario: Production Automation Uses Fixed Sleep Everywhere

Replace fixed sleeps with state polling and bounded backoff when waiting for asynchronous systems.

---

## 171. Scenario: Production Automation Uses Unbounded ThreadPool

Bound worker count and apply dependency-specific concurrency limits.

---

## 172. Scenario: Python Async Code Calls Blocking boto3 Directly

Either use synchronous code with threads for I/O workloads or move blocking SDK calls off the event loop; do not block asyncio unintentionally.

---

## 173. Scenario: Python Threaded Code Shares Mutable State

Minimize shared state, use locks where necessary, or collect per-worker results and merge centrally.

---

## 174. Scenario: Python Multiprocessing Is Slower

Measure serialization/process startup overhead and workload granularity. Multiprocessing is not automatically faster for small tasks.

---

## 175. Scenario: Production Script Has Race Condition

Identify shared state/ordering, reproduce under controlled concurrency, then add synchronization, idempotency, or state-version checks.

---

## 176. Scenario: Distributed Lock Is Stale

Verify lock owner/heartbeat and whether work is actually active. Use lease/expiry semantics and safe recovery rather than deleting locks blindly.

---

## 177. Scenario: Release State Says Deploying but Cluster Is Healthy

Treat persisted state and observed state separately. Reconcile actual ArgoCD/Kubernetes state and update the workflow checkpoint.

---

## 178. Scenario: Release State Says Failed but Deployment Succeeded

Verify actual deployment identity and health, then reconcile the workflow record rather than redeploying unnecessarily.

---

## 179. Scenario: Workflow Restarts After Checkpoint

Load persisted state, verify external state for each completed stage, and resume only where the next operation is safe.

---

## 180. Scenario: Checkpoint Is Corrupt

Fail safely, reconstruct state from external authoritative systems where possible, and avoid destructive assumptions.

---

## 181. Scenario: External State and Checkpoint Disagree

External system state should be verified directly before mutation. Mark checkpoint stale and reconcile it.

---

## 182. Scenario: Production Python Tool Has Growing Technical Debt

Prioritize safety boundaries, tests around critical mutations, modular clients/policies, dependency upgrades, observability, and removal of duplicated logic.

---

## 183. Scenario: Team Wants Faster Automation

Measure current bottlenecks, optimize API calls/concurrency safely, cache stable metadata, parallelize independent work, and preserve rate limits and deadlines.

---

## 184. Scenario: Team Wants More Parallelism

First identify dependency limits and shared-state conflicts. Add bounded concurrency only where independent work can safely run in parallel.

---

## 185. Scenario: Team Wants Fewer Logs for Cost

Keep security/audit/error evidence, reduce repetitive logs, aggregate high-volume events, and retain correlation identifiers.

---

## 186. Scenario: Team Wants to Skip Tests for Hotfix

Use targeted tests plus approved emergency process; do not remove all verification because the change is urgent.

---

## 187. Scenario: Team Wants to Disable Production Guard

Reject bypass without a documented need and compensating control. Production guardrails are part of the automation's safety boundary.

---

## 188. Scenario: Python Tool Is Used by Multiple Environments

Use one versioned codebase with explicit environment configuration and target validation; avoid hidden environment-specific behavior.

---

## 189. Scenario: Dev Works but Production Fails Due to Permissions

Compare actual identities/permissions and use least-privilege policy. Do not copy production-admin permissions into development as a fix.

---

## 190. Scenario: Staging Does Not Match Production

Document environmental differences, use representative integration tests, and validate critical APIs/configuration/permissions before production.

---

## 191. Scenario: Production Depends on a Manual Step

Automate it only after defining safe inputs, authorization, idempotency, verification, and rollback. Manual steps should be explicit and auditable.

---

## 192. Scenario: Production Automation Needs Human Decision

Generate deterministic evidence and recommendation, then require explicit approval for high-risk actions rather than hiding the decision inside code.

---

## 193. Scenario: Python Automation Must Choose Rollback or Forward Fix

Evaluate blast radius, rollback safety, database compatibility, artifact availability, and time-to-recovery. Choose the lowest-risk path based on evidence.

---

## 194. Scenario: Production Release Is Healthy but Slow

If user impact is acceptable, investigate performance separately rather than rolling back solely because duration increased. Compare against release SLOs.

---

## 195. Scenario: Production Release Is Fast but Unhealthy

Speed is irrelevant if health gates fail. Stabilize and rollback/mitigate according to policy.

---

## 196. Scenario: Automation Cannot Prove Success

Return UNKNOWN/FAILED according to the criticality of the evidence rather than claiming success.

---

## 197. Scenario: Automation Cannot Prove Failure

Investigate observability gaps and external state; do not trigger destructive remediation based only on missing evidence.

---

## 198. Scenario: Python Tool Is Asked to Delete All Failed Pods

Do not perform blanket deletion. Classify failures first; deleting Pods can remove valuable evidence and may increase impact.

---

## 199. Scenario: Python Tool Is Asked to Restart All Pods

Reject blanket remediation unless explicitly justified. Restart only known affected workloads with bounded attempts and post-restart verification.

---

## 200. Scenario: Production Memory Leak Is Suspected

Capture memory trend, allocation evidence, workload volume, object retention, and release correlation. Mitigate safely while engineering a permanent fix.

---

## 201. Scenario: Production CPU Spike Is Suspected to Be a Memory Problem

Use metrics and profiling to distinguish CPU, memory, I/O, GC, and external wait. Do not assume resource type from one signal.

---

## 202. Scenario: Python Garbage Collection Causes Latency

Measure allocation/GC behavior, object lifetime, workload pattern, and pause impact before changing GC settings.

---

## 203. Scenario: Large JSON API Causes Latency and Memory

Use streaming/pagination if supported, parse only needed fields, process incrementally, and avoid repeated serialization.

---

## 204. Scenario: Python Report Generation Crashes

Check result accumulation and serialization size. Stream report output or summarize incrementally.

---

## 205. Scenario: Production Automation Needs File Locking

Use locking only for local single-host coordination; for distributed workers use a durable distributed lock/lease appropriate to the environment.

---

## 206. Scenario: Local Lock File Remains After Crash

Use PID/lease/staleness validation and safe cleanup. Do not simply delete a lock without determining whether another process owns it.

---

## 207. Scenario: Batch Job Needs Exactly-Once Processing

Explain that exactly-once end-to-end behavior is difficult across independent systems. Design idempotent consumers, durable state, unique operation IDs, and reconciliation.

---

## 208. Scenario: Message Is Delivered Twice

Make processing idempotent using a message/event ID and durable processed-state record.

---

## 209. Scenario: Message Is Lost

Use durable queues/acknowledgment/retry semantics and monitor dead-letter/backlog behavior according to the messaging system.

---

## 210. Scenario: Dead-Letter Queue Grows

Classify failed messages, identify poison messages/root cause, replay safely after remediation, and prevent infinite retry loops.

---

## 211. Scenario: Production API Client Leaks Connections

Use Session/context management, response closing, bounded connection pools, and tests under concurrency.

---

## 212. Scenario: Connection Pool Is Exhausted

Inspect unclosed responses, worker count, timeout behavior, and pool size. Fix leaks before simply increasing the pool.

---

## 213. Scenario: DNS/TLS Calls Consume All Timeout Budget

Use layered timeouts and an overall deadline so one dependency cannot consume the entire workflow.

---

## 214. Scenario: Production Automation Uses UTC and Local Time Incorrectly

Use timezone-aware datetimes, explicit timezone configuration, and UTC for stored/audit timestamps.

---

## 215. Scenario: Certificate Expires During Deployment

Pause/stop release if TLS is required, renew through approved certificate automation, verify the new certificate chain/hostname, then resume.

---

## 216. Scenario: Token Expires During Long Workflow

Use short-lived identity with supported refresh/reacquisition, and ensure stage operations handle expiration safely.

---

## 217. Scenario: CI Secret Rotation Breaks Automation

Check secret provider/version/configuration, update consumers through controlled rollout, and avoid storing old credentials as fallback.

---

## 218. Scenario: Python Dependency Cannot Be Downloaded in CI

Check package repository, network/proxy, pinned version availability, lockfile, and internal artifact mirror. Avoid installing arbitrary unpinned packages.

---

## 219. Scenario: Package Repository Is Compromised

Freeze affected builds, verify artifact provenance/integrity, rotate credentials if exposed, move to trusted versions/mirror, and rebuild.

---

## 220. Scenario: Python Image Pulls Different Base Image Over Time

Pin/digest critical base images according to policy and record image provenance.

---

## 221. Scenario: Production Automation Needs Reproducibility

Pin dependencies, container/runtime version, external tool versions, configuration schema, and source SHA.

---

## 222. Scenario: Python Tool Works on One Runner Only

Identify hidden dependencies such as installed binaries, PATH, filesystem state, credentials, network, or local caches; package them into a reproducible environment.

---

## 223. Scenario: Production Automation Uses a Shared Mutable Config File

Move configuration into version-controlled/managed sources with explicit environment selection and immutable release association.

---

## 224. Scenario: Incident Is Caused by Configuration Drift

Identify source-of-truth ownership, reconcile desired state, and add drift detection/prevention.

---

## 225. Scenario: Python Tool Detects Drift

Report exact resource/property difference and owner; do not automatically mutate unless remediation is explicitly safe and authorized.

---

## 226. Scenario: Production Automation Needs Self-Healing

Define allowed failure classes and reversible actions, then implement bounded remediation, verification, cooldown, and escalation.

---

## 227. Scenario: Self-Healing Loops Forever

Set maximum remediation attempts, cooldown, circuit breaker, and human escalation. Repeated failure means the automated action is not solving the root cause.

---

## 228. Scenario: Production Tool Cannot Access Monitoring

Use independent checks where possible and report observability degradation; never treat lack of data as healthy.

---

## 229. Scenario: Production Tool Cannot Access Notification

Persist result independently and retry notification separately. Notification failure should not erase deployment outcome.

---

## 230. Scenario: Release Completes but Audit Report Fails

Persist core release evidence before generating secondary reports. Reconstruct the report from durable metadata.

---

## 231. Scenario: Production Deployment Has No Rollback Artifact

Stop promotion until a known-good immutable artifact is identified or define a safe forward-fix strategy. Do not invent a rollback target.

---

## 232. Scenario: Production Deployment Has No Previous Version

Use available immutable release history/GitOps revisions and assess compatibility. If no safe recovery exists, prioritize mitigation and controlled forward fix.

---

## 233. Scenario: Release Artifact Was Deleted

Check registry retention/lifecycle and backups/mirrors. Restore/promote from a trusted immutable source rather than rebuilding blindly.

---

## 234. Scenario: Production Environment Is Missing Required IAM Permission

Do not grant AdministratorAccess. Identify exact API action/resource, update least-privilege role through change control, and test.

---

## 235. Scenario: Production Environment Is Missing Required Kubernetes Permission

Identify resource/verb/namespace and update RBAC narrowly. Test with the same ServiceAccount.

---

## 236. Scenario: Automation Needs Cross-Account Access

Use an explicit trust relationship and role assumption with least privilege; validate source identity and target account before mutation.

---

## 237. Scenario: Automation Needs Cross-Region Operations

Represent region explicitly in configuration, create separate clients where required, and verify target resources before each mutation.

---

## 238. Scenario: Multi-Region Health Check

Run independent regional checks with bounded concurrency and return per-region health plus aggregate status.

---

## 239. Scenario: Multi-Account Health Check

Validate each account/role independently, isolate failures, and never let one account's credentials/configuration leak into another.

---

## 240. Scenario: One Region Fails in Multi-Region Deployment

Pause promotion according to policy, determine whether traffic can fail over safely, and avoid declaring global success.

---

## 241. Scenario: Production Change Has Unknown Blast Radius

Prefer read-only discovery, staged rollout, canary, or approval before mutation. If impact cannot be bounded, do not automate destructive action.

---

## 242. Scenario: Production Automation Needs a New AWS API

Read current SDK/API documentation, implement least privilege, add mocks/integration tests, and verify throttling/error behavior.

---

## 243. Scenario: Production Automation Needs a New Kubernetes API

Check server/client compatibility and API version, test resource schema/status behavior, and update RBAC accordingly.

---

## 244. Scenario: Production Automation Needs a New CI Integration

Define authentication, webhook/API contract, failure/retry behavior, idempotency, permissions, and audit evidence before implementation.

---

## 245. Scenario: Production Automation Is Hard to Troubleshoot

Add stage boundaries, correlation ID, structured error classification, duration metrics, external-state evidence, and a runbook.

---

## 246. Scenario: Production Automation Is Hard to Test

Separate pure policy logic from external clients and orchestration. Inject clients so unit tests do not require real infrastructure.

---

## 247. Scenario: Production Automation Is Hard to Review

Keep functions small, use typed models, explicit side-effect boundaries, stable interfaces, and tests for every mutation path.

---

## 248. Scenario: Production Automation Is Hard to Roll Back

Treat the automation itself as a versioned artifact, keep previous releases available, and ensure state/checkpoints are compatible.

---

## 249. Scenario: Production Automation Is Changing Too Many Resources

Add plan/diff mode, target allowlists, concurrency limits, maximum mutation counts, and approval thresholds.

---

## 250. Scenario: Python Tool Must Stop After 10 Errors

Use an explicit error budget/circuit threshold and stop new work after the threshold while preserving failed-item evidence.

---

## 251. Scenario: Batch Has Partial Success

Return per-item results and aggregate status. Retry only safe transient failures and make final success criteria explicit.

---

## 252. Scenario: Batch Must Be Atomic

If the external system cannot provide a transaction, explain the limitation and use compensation/reconciliation rather than claiming atomicity.

---

## 253. Scenario: Compensation Action Fails

Record both original and compensation failures, stop cascading attempts, and escalate with current-state evidence.

---

## 254. Scenario: Production Automation Needs Rollforward

If rollback is unsafe, use a known-good forward fix, keep artifact identity immutable, and verify each stage.

---

## 255. Scenario: Incident Requires Disabling a Feature

Use feature flag/configuration controls if available, verify scope, record the action, and restore only after validation.

---

## 256. Scenario: Feature Flag Service Is Down

Use safe default behavior defined by the application and release policy. Do not assume the flag state if it cannot be verified.

---

## 257. Scenario: Configuration Service Is Down

Use explicitly approved cached configuration only if freshness and safety are acceptable; otherwise fail closed.

---

## 258. Scenario: Cached State Is Stale

Define TTL/invalidation and distinguish cached informational data from critical deployment state. Never use stale identity/security state for authorization.

---

## 259. Scenario: Production Automation Has Clock-Dependent Logic

Use injected clocks in tests, timezone-aware timestamps, and explicit time windows.

---

## 260. Scenario: Scheduled Automation Misses a Run

Define whether missed work should be skipped, replayed once, or caught up. Use idempotent execution before enabling catch-up.

---

## 261. Scenario: Scheduled Automation Runs During Deployment

Coordinate through maintenance windows, locks, or safe concurrency design to prevent conflicting mutations.

---

## 262. Scenario: Production Python Job Is Repeatedly Evicted

Inspect requests/limits, node pressure, priority, workload size, and cluster capacity. Correct scheduling/resource design.

---

## 263. Scenario: Python Job Is Throttled by CPU

Measure CPU usage/throttling and workload requirements; optimize or adjust resources rather than blindly increasing limits.

---

## 264. Scenario: Python Job Uses Too Many Network Connections

Bound concurrency and connection pools, reuse Sessions, and close responses correctly.

---

## 265. Scenario: Production Automation Creates Too Many Kubernetes Jobs

Add deduplication, run IDs, cleanup policy, and concurrency controls.

---

## 266. Scenario: Old Jobs/Pods Accumulate

Configure retention/TTL where supported and safely clean historical resources without deleting active evidence.

---

## 267. Scenario: Production Python Job Has No Resource Requests

Set requests/limits based on measured workload so scheduling and capacity planning are predictable.

---

## 268. Scenario: Production Python Service Has Bad Liveness Probe

Make liveness process-focused and cheap; dependency health should generally affect readiness or separate health signals rather than causing restart loops.

---

## 269. Scenario: Readiness Probe Causes Deployment Failure

Check probe path/port/timing, startup duration, dependency requirements, and application health endpoint semantics.

---

## 270. Scenario: Startup Takes Too Long

Use startupProbe where appropriate, configure realistic deadlines, and distinguish initialization from steady-state health.

---

## 271. Scenario: Python Service Gets Restarted Repeatedly

Inspect termination reason, exit code, liveness events, OOM, configuration, and startup timing. Do not increase probe thresholds without evidence.

---

## 272. Scenario: Production Service Has No Graceful Shutdown

Implement SIGTERM handling, stop accepting work, finish/checkpoint safe operations, and respect termination grace period.

---

## 273. Scenario: Production Deployment Is Successful but Metrics Reset

Understand Pod/process restart behavior and use Prometheus counters appropriately; do not interpret a process restart as business metric loss.

---

## 274. Scenario: Counter Metrics Reset After Restart

Use counters as monotonic within a process and rely on Prometheus rate/increase semantics across restarts.

---

## 275. Scenario: Python Metrics Have Unbounded Labels

Remove dynamic label values and use logs for detailed identifiers.

---

## 276. Scenario: Production Logging Is Missing Correlation

Introduce run/release/incident ID at the workflow boundary and propagate it consistently.

---

## 277. Scenario: Production Error Has No Context

Wrap it with operation/target/stage context while preserving the original exception cause.

---

## 278. Scenario: Broad Exception Handler Hides Failure

Replace with specific exception classes and ensure the CLI returns nonzero on operational failure.

---

## 279. Scenario: Error Handler Itself Fails

Keep error reporting defensive: sanitize/serialize safely and provide a minimal fallback error path.

---

## 280. Scenario: Production Automation Returns Success Too Early

Define success as verified desired state, not merely request acceptance. Add post-operation verification.

---

## 281. Scenario: API Says Deployment Accepted but Pods Are Old

Follow the deployment chain and verify Git revision, ArgoCD sync, Deployment template, Pod image ID, and rollout.

---

## 282. Scenario: Health Check Uses Stale Cache

Identify cache TTL and invalidate after mutations where required; use direct authoritative state for critical verification.

---

## 283. Scenario: Production Tool Has Conflicting Sources of Truth

Define ownership: Git for GitOps desired state, Terraform state/config for infrastructure, Kubernetes for observed runtime state, and Python for orchestration/policy.

---

## 284. Scenario: Python Tool Changes Terraform State Directly

Avoid direct state manipulation unless using supported Terraform mechanisms. Treat state as Terraform-owned.

---

## 285. Scenario: Python Tool Changes Kubernetes Resource Outside GitOps

If ArgoCD owns it, reconcile through Git unless it is an explicitly documented emergency operation.

---

## 286. Scenario: Production Tool Needs to Override Policy

Use an explicit approved exception path with audit evidence and expiry rather than hidden code flags.

---

## 287. Scenario: Emergency Override Flag Is Left Enabled

Make overrides time-bound, environment-scoped, auditable, and automatically rejected after expiry.

---

## 288. Scenario: Production Automation Is Successful but Unsafe

Success means the operation completed, not that the design is safe. Review blast radius, permissions, retries, observability, and recovery.

---

## 289. Scenario: Production Automation Is Fast but Fragile

Prioritize deterministic behavior, failure handling, idempotency, and observability over raw speed.

---

## 290. Scenario: Production Automation Is Slow but Safe

Measure bottlenecks and optimize without removing safety gates or evidence requirements.

---

## 291. Scenario: Need to Choose Threads vs Async

For I/O-bound work, both can help; choose based on existing libraries and architecture. Bound concurrency either way. For CPU-bound work, consider processes/native acceleration.

---

## 292. Scenario: Need to Choose Threads vs Processes

Use threads for I/O waiting with thread-safe clients; use processes for CPU-heavy independent work when serialization/startup overhead is acceptable.

---

## 293. Scenario: Need to Choose Async vs Sync

Choose async when high-volume concurrent I/O and async-compatible libraries justify the complexity; otherwise a synchronous design with bounded threads may be simpler.

---

## 294. Scenario: Production Automation Needs Parallel AWS Calls

Use bounded worker pools, service-specific rate limits, retries, and request budgets.

---

## 295. Scenario: Production Automation Needs Parallel Kubernetes Calls

Scope requests and use bounded concurrency while protecting API-server capacity.

---

## 296. Scenario: Production Automation Needs Parallel Git Operations

Avoid concurrent writes to the same repository/branch. Parallelize independent read operations only.

---

## 297. Scenario: Production Automation Needs Parallel Terraform

Do not run conflicting applies against the same state. Use locking and environment separation.

---

## 298. Scenario: Production Automation Needs Parallel ArgoCD Operations

Respect application/environment ownership and avoid simultaneous conflicting syncs.

---

## 299. Scenario: Final Production Scenario Answer

A strong answer consistently shows: I verify impact and target first, collect evidence, classify transient versus permanent failure, use the smallest safe mitigation, verify actual external state, preserve audit evidence, and then add prevention through tests, policy, observability, and safer automation.

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
├── 07-Production-Scenarios.md         ✓
└── 08-Mock-Interview.md
```

**Next file: `08-Mock-Interview.md`**