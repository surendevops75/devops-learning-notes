# Kubernetes Cleanup Automation

> Production-oriented Python Kubernetes cleanup automation for EKS with policy-driven discovery, dry-run safety, controlled deletion, auditability, observability, and production troubleshooting.

## Project Scope

```text
Kubernetes Python Client
EKS authentication
RBAC / least privilege
Jobs / Pods / ReplicaSets
Lifecycle policies
Protected resources
Dry-run + approval
UID / ResourceVersion validation
Bounded deletion
Deletion verification
Prometheus + Grafana
ELK
Jenkins / GitHub Actions
Helm + ArgoCD
Production troubleshooting
Senior interview preparation
```

## 1. Project Overview

Build a production-grade Python automation that safely identifies and cleans up stale Kubernetes resources in EKS. The project focuses on Jobs, completed Pods, failed Jobs, terminated Pods, old ReplicaSets, unused ConfigMaps/Secrets where safely provable, and other explicitly approved resources. Cleanup must be policy-driven, dry-run first, auditable, idempotent, RBAC-restricted, and protected against accidental deletion.

---

## 2. Real-World Problem

Long-running Kubernetes clusters accumulate completed Jobs, failed Pods, old ReplicaSets, stale test resources, and other objects. Uncontrolled cleanup increases API/storage overhead and operational noise, while overly aggressive deletion can destroy debugging evidence or application state. The automation therefore separates discovery, classification, approval, deletion, verification, and reporting.

---

## 3. Core Principle

Never define cleanup as 'delete old objects'. Define it as 'delete objects that satisfy an explicit, reviewed policy and are proven safe to remove'. Age is only one signal.

---

## 4. Architecture

Scheduler/CronJob → Python Kubernetes client → cluster identity validation → discovery → policy engine → candidate classification → dry-run/report → approval guard → bounded deletion → verification → audit report → Prometheus/ELK/Grafana → incident/ticket workflow.

---

## 5. Technology Stack

Python 3, kubernetes Python client, argparse, dataclasses, datetime, logging, json, csv, pytest, unittest.mock, Docker, Kubernetes/EKS, Helm, ArgoCD, Prometheus, Grafana, ELK, Jenkins, and GitHub Actions.

---

## 6. Repository Structure

Recommended modules: cli.py, config.py, kube_client.py, discovery.py, policy.py, classifiers.py, safety.py, deletion.py, verification.py, state.py, reporters.py, metrics.py, alerts.py, logging_config.py, models.py, plus unit and integration tests.

---

## 7. Cleanup Scope

Define resource types explicitly. A first production scope can include completed Jobs, failed Jobs beyond retention, terminated Pods, and old ReplicaSets that are no longer referenced by active workloads. Avoid broad cleanup of arbitrary resources.

---

## 8. Resource Allowlist

Use an allowlist of resource kinds that the automation is permitted to inspect and delete. Unknown resource kinds must never become deletable merely because they match an age condition.

---

## 9. Delete Allowlist

Use a stricter allowlist for actual deletion than for discovery. A resource can be discoverable for reporting without being eligible for automated deletion.

---

## 10. Namespace Scope

Support one namespace, a configured namespace list, or approved cluster-wide scope. Never silently escalate namespace scope to all namespaces.

---

## 11. Production Scope

Require explicit production enablement. Development/staging can use more permissive policies, while production should require stronger age, ownership, and protection checks.

---

## 12. Cluster Identity

Validate expected EKS cluster identity and AWS account before any delete-capable operation. A typo in kubeconfig must not result in deleting resources from the wrong cluster.

---

## 13. AWS Identity

When EKS metadata is needed, call STS/EKS APIs with a dedicated workload identity. Do not embed AWS access keys in the cleanup container.

---

## 14. In-Cluster Authentication

Use load_incluster_config() for EKS CronJob execution. For local development, use load_kube_config() with an explicitly selected context.

---

## 15. Context Safety

Print or log the cluster/context identity in dry-run output and validate it against configuration before enabling deletion.

---

## 16. RBAC Principle

The cleanup ServiceAccount should have read access to resources it evaluates and delete access only to explicitly approved resource types and scopes.

---

## 17. Namespace RBAC

For namespace-specific cleanup, prefer Role/RoleBinding. Cluster-wide deletion requires a ClusterRole/ClusterRoleBinding and should be exceptional.

---

## 18. ServiceAccount

Create a dedicated cleanup ServiceAccount. Never reuse the default application ServiceAccount or a broad administrative account.

---

## 19. Least Privilege

Do not grant the cleanup job wildcard permissions such as */* or unrestricted delete verbs. Restrict verbs, resources, API groups, namespaces, and where possible resource names/patterns through policy architecture.

---

## 20. Read-Only Discovery

The discovery phase should be executable with read-only credentials. Deletion should be a separate execution mode with stronger authorization.

---

## 21. Two-Phase Design

Phase 1 produces candidates and a signed/versioned plan. Phase 2 executes only an approved plan whose cluster, policy, and resource identities still match current state.

---

## 22. Dry Run

Dry-run is the default. It should discover candidates, explain why each candidate qualifies, show planned deletion, and perform zero mutations.

---

## 23. Explicit Delete Flag

Require an explicit command such as --execute or an equivalent guarded mode before deletion. Never infer execution from environment alone.

---

## 24. Production Confirmation

For production, require an additional confirmation token or approved execution parameter that cannot be accidentally passed by a generic scheduled job.

---

## 25. Policy as Code

Store cleanup policies in version-controlled configuration so age thresholds, labels, namespaces, protected resources, and retention rules are reviewable.

---

## 26. Configuration Validation

Validate resource kinds, namespace scope, age thresholds, protected namespaces, execution mode, cluster identity, and deletion limits before discovery or deletion.

---

## 27. Configuration Precedence

Use a documented precedence such as CLI > environment > configuration file > safe defaults. Avoid hidden configuration sources.

---

## 28. Safe Defaults

Default to dry-run, small deletion limits, explicit namespace scope, and conservative retention. Production defaults should fail closed.

---

## 29. Maximum Deletions

Set a hard maximum number of objects deletable in one run. Abort when the candidate count exceeds the configured safety threshold instead of deleting everything automatically.

---

## 30. Maximum Percentage

Optionally limit deletion to a percentage of the discovered eligible population. This protects against a policy bug that suddenly marks most resources for deletion.

---

## 31. Rate Limit

Use bounded deletion concurrency and a maximum deletion rate to avoid overwhelming the Kubernetes API server or controllers.

---

## 32. Batch Size

Delete resources in bounded batches and report progress. Do not create an unbounded deletion queue for very large clusters.

---

## 33. Deletion Timeout

Use bounded API request timeouts and an overall job deadline so cleanup cannot run indefinitely.

---

## 34. Retry Policy

Retry transient API failures and throttling with bounded exponential backoff and jitter. Do not retry Forbidden, NotFound, or invalid-resource errors forever.

---

## 35. NotFound Handling

If an object disappears between discovery and deletion, classify it as already cleaned rather than treating it as a dangerous failure.

---

## 36. Conflict Handling

If a resource changes between discovery and deletion, re-read or use resourceVersion checks where appropriate. Never blindly delete a resource after significant state change.

---

## 37. ResourceVersion

Capture resourceVersion during discovery when it helps protect against stale decisions. If the resource has materially changed, re-evaluate policy before deletion.

---

## 38. UID Protection

Use namespace/name plus UID for candidate identity. A new object can reuse a name after the old object was deleted.

---

## 39. Generation

For controller resources, generation can help determine whether configuration changed. It is not a universal deletion-safety mechanism but can provide useful context.

---

## 40. Owner References

Inspect ownerReferences before deleting objects. Ownership is a critical signal for distinguishing controller-managed resources from standalone resources.

---

## 41. Controller Owner

A resource owned by an active controller should not be deleted merely because it is old. Evaluate whether the controller is expected to recreate or retain it.

---

## 42. Orphan Detection

Orphaned resources can be cleanup candidates, but orphan status must be proven. Do not treat missing owner references as automatic permission to delete.

---

## 43. Finalizers

Resources with finalizers may require cleanup controllers or external systems. The automation should normally report them rather than forcibly removing finalizers.

---

## 44. Finalizer Safety

Never remove finalizers automatically as a generic cleanup action. A finalizer may protect external resources or data consistency.

---

## 45. DeletionTimestamp

A resource with deletionTimestamp is already terminating. Do not repeatedly issue deletion calls; monitor completion and classify it separately.

---

## 46. Graceful Deletion

Use Kubernetes deletion semantics rather than bypassing controllers. Respect propagation behavior and resource-specific cleanup expectations.

---

## 47. Propagation Policy

Understand Foreground, Background, and Orphan propagation before deleting controller resources. Default behavior should be explicit and tested.

---

## 48. Jobs

Jobs are a natural cleanup target because completed and failed Jobs can accumulate. However, retention must preserve operational evidence and compliance requirements.

---

## 49. Completed Jobs

A Job with status.succeeded > 0 can be eligible after a configured retention period, provided it is not protected by policy.

---

## 50. Failed Jobs

Failed Jobs should usually have a longer diagnostic retention than successful Jobs because they contain evidence of incidents.

---

## 51. Active Jobs

Never delete active Jobs merely because their creation timestamp is old unless an explicit timeout/abandonment policy exists and has been reviewed.

---

## 52. Job Completion Time

Prefer completion time over creation time when deciding retention for completed Jobs because completion reflects when the workload became inactive.

---

## 53. Job Start Time

Start time can help identify long-running or stuck Jobs, but it should not replace active-state checks.

---

## 54. Job Conditions

Inspect Job conditions such as Complete and Failed, plus active/succeeded/failed counts, before classification.

---

## 55. BackoffLimit

A Job that has exhausted backoffLimit may be failed, but deletion policy should preserve enough evidence for troubleshooting.

---

## 56. ActiveDeadlineSeconds

Job activeDeadlineSeconds can cause termination. Record the reason before cleanup so incident analysis remains possible.

---

## 57. TTL Controller

Kubernetes supports TTL-after-finished for Jobs. Prefer the native TTL controller when its semantics satisfy the requirement rather than duplicating deletion logic in Python.

---

## 58. Python vs TTL

Python cleanup is useful for custom policies, cross-resource correlation, reporting, approvals, and environments where native TTL configuration cannot cover the desired lifecycle.

---

## 59. Completed Pods

Completed Pods from Jobs can accumulate. They can be cleaned after their associated Job is retained long enough for troubleshooting.

---

## 60. Failed Pods

Failed Pods often contain valuable incident evidence. Use a longer retention window and protect pods associated with recent incidents or failed Jobs.

---

## 61. Succeeded Pods

Succeeded Pods can be safely cleaned after the associated workload's retention period when no policy requires them for audit or debugging.

---

## 62. Running Pods

Never delete Running application Pods as generic cleanup. Running state requires workload-specific maintenance or remediation workflows.

---

## 63. Pending Pods

Do not classify Pending Pods as cleanup candidates merely because they are old. A Pending pod can represent a real production scheduling incident.

---

## 64. Unknown Pods

Unknown status should be treated as a diagnostic condition, not a cleanup condition.

---

## 65. Terminating Pods

A long-Terminating pod can indicate finalizers, volume, node, or controller issues. Report and troubleshoot before considering any forced action.

---

## 66. Evicted Pods

Evicted Pods can be cleanup candidates after sufficient retention, but preserve enough evidence to diagnose node pressure incidents.

---

## 67. CrashLoop Pods

A CrashLooping Pod is active troubleshooting evidence. Never delete it merely because it exceeds an age threshold.

---

## 68. ImagePull Pods

ImagePullBackOff and ErrImagePull pods are troubleshooting evidence and should not be treated as stale cleanup candidates while active.

---

## 69. Old ReplicaSets

Old ReplicaSets can accumulate after Deployment rollouts. Only delete ReplicaSets that are no longer needed by the Deployment's revision history and are outside retention policy.

---

## 70. ReplicaSet Ownership

Verify the ReplicaSet is owned by a Deployment before applying Deployment-specific cleanup rules.

---

## 71. Active ReplicaSet

Never delete the active/current ReplicaSet for a Deployment.

---

## 72. Revision History

Deployment revisionHistoryLimit controls how many old ReplicaSets are retained. Prefer this native mechanism when it satisfies retention needs.

---

## 73. Python vs RevisionHistory

Python cleanup can complement native revision history when custom audit, cross-namespace policy, or historical exceptions are required.

---

## 74. StatefulSets

Do not generically delete old StatefulSet Pods or resources because they can have stable identity and persistent storage semantics.

---

## 75. DaemonSets

Do not delete DaemonSet Pods as stale resources; controllers intentionally maintain them.

---

## 76. Deployments

Do not delete Deployments merely because they are old or inactive unless the policy explicitly targets abandoned applications and includes ownership/approval safeguards.

---

## 77. Services

Unused Services are difficult to prove safely. Avoid generic automatic Service deletion unless ownership and application lifecycle are explicitly modeled.

---

## 78. ConfigMaps

Stale ConfigMaps can be dangerous to clean because applications may reference them indirectly or through historical revisions. Prefer owner labels and explicit lifecycle metadata.

---

## 79. Secrets

Never automatically delete Secrets based only on age. Secret references may be hidden in workloads, external systems, or recovery procedures.

---

## 80. PVCs

Never generically delete old PVCs. PVC deletion can destroy or detach application data and may trigger storage deletion depending on reclaim policy.

---

## 81. PVs

PersistentVolumes require explicit storage lifecycle ownership. They should not be part of generic cleanup.

---

## 82. Namespaces

Never include namespaces in generic cleanup. Namespace deletion is a high-impact operation requiring separate lifecycle management.

---

## 83. CRDs

Never delete CustomResourceDefinitions as part of routine cleanup. They define APIs and can cascade into significant resource loss.

---

## 84. Custom Resources

Custom Resources require operator-specific semantics. Discover/report them only unless a dedicated, reviewed controller integration exists.

---

## 85. Nodes

Nodes are infrastructure resources and must never be deleted by a generic Kubernetes cleanup script.

---

## 86. Events

Kubernetes Events are naturally short-lived and can usually be handled by cluster event retention mechanisms. Python should not aggressively delete them unless there is a specific documented requirement.

---

## 87. Leases

Leases can be used for leader election and heartbeat. Never treat them as generic stale objects without understanding their owner and purpose.

---

## 88. ServiceAccounts

Do not delete old ServiceAccounts automatically based on age because workload references can be indirect and security impact is high.

---

## 89. Roles and RoleBindings

Authorization resources require ownership-aware lifecycle management. Generic age-based deletion can create outages.

---

## 90. NetworkPolicies

NetworkPolicies should never be deleted as generic stale objects because their absence can change security posture immediately.

---

## 91. Ingress

Ingress resources may appear unused but can be connected to external load balancers and DNS. Keep them outside generic cleanup unless ownership is explicit.

---

## 92. StorageClasses

StorageClasses are cluster configuration and should never be handled by generic cleanup.

---

## 93. PriorityClasses

PriorityClasses can affect scheduling and should not be age-cleaned generically.

---

## 94. ResourceQuotas

ResourceQuotas are policy objects, not temporary resources. Do not delete automatically.

---

## 95. LimitRanges

LimitRanges affect default resource behavior and should not be generic cleanup targets.

---

## 96. Labels

Require explicit lifecycle labels for custom resources where possible, such as cleanup=true, retention=30d, owner=team-a, or environment=dev.

---

## 97. Lifecycle Labels

Labels make cleanup intent explicit. A resource lacking the required lifecycle marker should not be automatically deleted in production.

---

## 98. Protected Label

Support a protected=true or do-not-delete=true label as a safety override. Protection must take precedence over all cleanup rules.

---

## 99. Protected Namespace

Maintain an explicit protected namespace list including system and critical application namespaces. Protected namespaces should be denied before candidate evaluation.

---

## 100. System Namespaces

Namespaces such as kube-system contain critical components and should never be included in broad cleanup policies.

---

## 101. Critical Workloads

Maintain an allowlist of critical workloads that require stronger retention or manual approval.

---

## 102. Environment Labels

Use environment metadata to apply different policies. Production should have stricter retention and deletion limits than development.

---

## 103. Owner Labels

Require an owner/team label for cleanup candidates in production so an accountable team can be identified.

---

## 104. Application Labels

Use standardized application labels for grouping and reporting. Do not assume arbitrary labels are reliable ownership evidence.

---

## 105. Cost Center Labels

Optional cost-center labels can help report cleanup savings without becoming a deletion criterion by themselves.

---

## 106. Age Calculation

Use timezone-aware UTC timestamps and calculate age from the appropriate lifecycle timestamp rather than local system time.

---

## 107. Age Threshold

An example policy could retain successful Jobs for seven days and failed Jobs for fourteen days. These are examples only and must be replaced by approved organizational policy.

---

## 108. Creation vs Completion

For completed Jobs, completion time is usually the better retention reference. For abandoned resources, creation time may be relevant only after active-state and ownership checks.

---

## 109. Minimum Retention

Enforce a minimum retention floor so a misconfigured policy cannot delete newly created objects.

---

## 110. Maximum Age

Maximum age is not sufficient for deletion safety. Combine it with state, owner, labels, environment, dependencies, and protected-resource checks.

---

## 111. Policy Precedence

Recommended precedence: protected override → resource safety exclusion → namespace exclusion → active-state exclusion → ownership validation → explicit lifecycle policy → age → deletion limits.

---

## 112. Candidate Model

Represent each candidate with kind, API group, namespace, name, UID, creation/completion timestamp, owner, labels, reason, policy version, risk level, and deletion eligibility.

---

## 113. Risk Level

Use LOW, MEDIUM, HIGH, and BLOCKED risk classifications. High-risk resources should require manual approval or remain report-only.

---

## 114. Deletion Eligibility

Keep eligibility as an explicit boolean plus a list of blocking reasons. This makes policy behavior explainable and testable.

---

## 115. Explainability

Every deletion candidate should answer: what resource, why eligible, which policy, which threshold, what protections passed, and what would block deletion.

---

## 116. Dry-Run Report

A dry-run report should contain cluster identity, policy version, scan time, candidate counts, eligible counts, blocked counts, and per-resource reasons.

---

## 117. Audit Report

After execution, record attempted, successful, already-gone, failed, skipped, and blocked resources plus error classification and execution identity.

---

## 118. Run ID

Generate a unique run ID for every cleanup execution and include it in logs, metrics, reports, and alerts.

---

## 119. Policy Version

Include a Git commit or policy version in every run so a later incident can reconstruct which cleanup rules were active.

---

## 120. Plan Hash

Optionally hash the normalized candidate plan. Execution can verify that the approved plan has not changed unexpectedly before deletion.

---

## 121. Approval Workflow

For high-risk production cleanup, generate a plan artifact, obtain approval, then execute only that exact plan if cluster identity and resource UIDs still match.

---

## 122. Stale Plan

An approved plan should expire after a short configurable window. A stale plan must be regenerated rather than blindly executed.

---

## 123. TOCTOU Risk

Time-of-check/time-of-use is a key cleanup risk: a resource can change after discovery. Revalidate critical fields immediately before deletion.

---

## 124. Revalidation

Before deleting a high-risk candidate, re-fetch it and verify UID, owner, state, labels, policy conditions, and expected timestamps.

---

## 125. Deletion Preconditions

A safe delete precondition can require exact UID, approved namespace, approved kind, protected-label check, inactive state, age threshold, and policy-version match.

---

## 126. Resource Locking

Kubernetes does not provide a generic application-level cleanup lock. Use a ConfigMap/Lease or external coordination mechanism only when needed to prevent overlapping cleanup workers.

---

## 127. CronJob Overlap

Set concurrencyPolicy: Forbid for a scheduled cleanup job so two cleanup executions do not run concurrently.

---

## 128. Leader Election

For multiple long-running cleanup workers, use Kubernetes Lease-based leader election or an external lock to ensure only one deletion coordinator is active.

---

## 129. Idempotency

Deletion is naturally close to idempotent: deleting an already-removed object should be treated as success/already-cleaned. The surrounding workflow must also be safe to repeat.

---

## 130. Deletion Propagation

Use explicit propagation behavior for resources with dependents. Never assume default behavior is safe for every resource type.

---

## 131. Delete Collection

Avoid broad collection deletes unless the policy has been independently proven safe. Per-resource deletion provides stronger auditability and safety controls.

---

## 132. Batch Deletion

Bounded batches can improve throughput while preserving progress reporting and failure isolation.

---

## 133. Parallel Deletion

Parallel deletion can increase API pressure and controller churn. Start with sequential or low-concurrency deletion and tune from measurements.

---

## 134. API Throttling

A 429 response should reduce concurrency/back off. Do not respond to throttling by creating more workers.

---

## 135. API Errors

Classify 401/403/404/409/429/5xx separately because remediation differs.

---

## 136. Forbidden

403 indicates an RBAC/policy issue and should stop deletion rather than being retried indefinitely.

---

## 137. Unauthorized

401 indicates authentication failure. Stop the run and fix identity configuration.

---

## 138. NotFound

404 after a valid discovery means another actor may already have removed the object. Record it as already absent.

---

## 139. Conflict

409 can indicate resource state changed. Re-read and re-evaluate rather than blindly retrying deletion.

---

## 140. RateLimited

429 requires backoff, jitter, and possibly lower concurrency. Track rate-limit metrics.

---

## 141. ServerError

5xx errors can be transient. Use bounded retries and stop if the Kubernetes API remains unavailable.

---

## 142. Timeout

Timeouts should be bounded and observable. A timeout does not prove the deletion failed or succeeded; re-read the resource before retrying when safe.

---

## 143. Deletion Verification

After deletion, optionally re-read the resource. A NotFound response confirms absence; a still-present resource may be terminating or deletion may have failed.

---

## 144. Terminating Verification

A resource with deletionTimestamp may be in the process of deletion. Record this as DELETING rather than falsely reporting deleted.

---

## 145. Finalizer Verification

If deletion remains blocked by finalizers, report the finalizer and owning controller. Do not strip it automatically.

---

## 146. Post-Run Verification

Compare attempted deletions with actual state, classify failures, and calculate cleanup success rate.

---

## 147. Safety Metrics

Track candidates discovered, candidates eligible, deletions attempted, deletions succeeded, deletions blocked, deletions failed, protected objects, and policy violations.

---

## 148. Prometheus Metrics

Useful metrics include cleanup_runs_total, cleanup_failures_total, cleanup_candidates_total, cleanup_deleted_total, cleanup_blocked_total, cleanup_errors_total, cleanup_duration_seconds, and cleanup_last_success_timestamp.

---

## 149. Metric Labels

Use bounded labels such as cluster, environment, namespace class, resource kind, outcome, and policy. Avoid resource name, UID, or arbitrary label values as Prometheus labels.

---

## 150. Metric Cardinality

Detailed resource identities belong in structured logs/reports, not persistent Prometheus labels.

---

## 151. Grafana Dashboard

Show cleanup run status, candidate count, deletion count, blocked count, failure count, API throttling, runtime, last successful run, and cleanup savings indicators where measurable.

---

## 152. ELK Logging

Send structured JSON logs to ELK with run ID, cluster, namespace, resource kind/name/UID, policy version, decision, reason, and error classification. Never log Secret values.

---

## 153. Log Levels

INFO for run summaries and important lifecycle events, DEBUG for detailed candidate analysis, WARNING for blocked/unexpected conditions, and ERROR for failed operations.

---

## 154. Per-Object Logging

For large clusters, avoid verbose INFO logs for every object. Emit detailed logs for candidates, deletions, failures, and policy blocks while aggregating healthy/non-candidate resources.

---

## 155. Audit Identity

Record the Kubernetes ServiceAccount identity and AWS role identity when available so deletion actions can be traced.

---

## 156. Audit Destination

Store reports in the standard protected log/report platform. If reports are persisted to S3, use encryption and restricted access.

---

## 157. Sensitive Metadata

Resource names, labels, image references, and namespace names can expose architecture. Restrict access to cleanup reports according to operational policy.

---

## 158. No Secret Logging

Never log Secret data, token values, kubeconfig contents, environment secrets, or complete Kubernetes Secret objects.

---

## 159. Backup Before Delete

Do not assume every Kubernetes object should be backed up before deletion. For critical state, use resource-specific backup mechanisms such as Velero or application-native backups rather than a generic script.

---

## 160. Velero Boundary

For cluster disaster recovery, Velero or another purpose-built backup platform may be better than Python cleanup. Python should focus on lifecycle cleanup and custom policy.

---

## 161. GitOps Boundary

ArgoCD should manage desired state. Cleanup automation must not delete resources that Git declares as desired unless the lifecycle policy explicitly models that behavior.

---

## 162. GitOps Safety

A resource managed by ArgoCD should normally be excluded from generic cleanup unless it is a generated/temporary resource with explicit lifecycle metadata.

---

## 163. Generated Resources

Controllers may generate resources that appear unused. Owner references and controller semantics must be checked before deletion.

---

## 164. Helm Resources

Helm-managed resources can be recreated or modified during upgrades. Avoid deleting them based only on age or lack of obvious activity.

---

## 165. Annotation Detection

Where organizational tooling writes ownership or lifecycle annotations, use them as policy inputs only after validating their standard and trust model.

---

## 166. ArgoCD Metadata

ArgoCD labels/annotations can identify application ownership, but cleanup should never use them as the sole proof that deletion is safe.

---

## 167. Job History

Jobs generated by CI/CD or CronJobs may have different retention requirements. Group cleanup by owner and job policy rather than one global age threshold.

---

## 168. CronJob Ownership

A Job may be owned by a CronJob. Before deleting a Job, identify its owner and apply the owner-specific retention policy.

---

## 169. Manual Jobs

Manually created Jobs may lack a CronJob owner. Use explicit lifecycle labels or a longer retention policy for them.

---

## 170. Failed Evidence

Failed Jobs and Pods can be valuable evidence. Production policies should retain failures longer than successful ephemeral workloads.

---

## 171. Incident Protection

Resources associated with an active incident, ticket, or investigation should be protected from cleanup until the incident owner releases them.

---

## 172. Incident Label

An approved incident-protection label or annotation can prevent cleanup. Ensure the protection mechanism is governed and auditable.

---

## 173. Maintenance Protection

Resources under maintenance should be protected for the maintenance window and then re-evaluated automatically.

---

## 174. Legal/Compliance Hold

Where required, a compliance hold must override normal cleanup retention. The cleanup engine should support a protected state without exposing sensitive legal details.

---

## 175. Retention Tiers

Use daily/weekly or environment-specific retention tiers only when the organization has defined them. Avoid hard-coded universal retention values.

---

## 176. Development Cleanup

Development namespaces can use shorter retention to reduce clutter, but still protect active workloads and explicitly protected resources.

---

## 177. Staging Cleanup

Staging often needs enough historical evidence for release validation. Retain failed resources longer than successful temporary resources.

---

## 178. Production Cleanup

Production cleanup should use conservative retention, explicit lifecycle labels, strict maximum deletion limits, and preferably approval for high-risk resource types.

---

## 179. Namespace Exclusion

Maintain a protected namespace list for system, platform, security, observability, and other critical infrastructure namespaces.

---

## 180. Application Exclusion

Support protected workload names or label selectors for critical applications.

---

## 181. Resource Exclusion

Allow exact resource exclusions for exceptional cases, but keep them version-controlled and auditable.

---

## 182. Exclusion Precedence

A protected resource must remain protected even if it satisfies every other cleanup condition.

---

## 183. Policy Conflict

If multiple policies conflict, choose the safer outcome and mark the resource as BLOCKED until the conflict is resolved.

---

## 184. Unknown Metadata

Missing metadata should normally cause BLOCKED/UNKNOWN rather than automatic deletion when the metadata is required for safety.

---

## 185. Clock Skew

Use UTC and consider cluster/client clock differences when evaluating age. Avoid relying on local wall-clock assumptions.

---

## 186. Time Zones

Store and compare timestamps in UTC. Human-readable reports can display local time separately if needed.

---

## 187. Timestamp Selection

Use completion time for completed Jobs, deletion/start timestamps for lifecycle diagnostics, and creation time only when the policy explicitly defines it.

---

## 188. Resource Creation Race

A newly created resource can be discovered during a cleanup scan. The minimum retention floor prevents immediate deletion even if other fields look eligible.

---

## 189. Controller Race

A controller can recreate or update a resource while cleanup runs. Revalidation and owner checks reduce the risk of deleting an active object.

---

## 190. Rollout Race

Old ReplicaSets can become active again during rollback. Re-check Deployment revision/owner state immediately before deletion.

---

## 191. Job Race

A Job may transition from active to completed during the scan. Evaluate current status before deletion rather than relying on initial discovery state.

---

## 192. Pod Race

A Pod may transition from Failed to recreated/removed. UID-based identity and revalidation prevent deleting a replacement object with the same name.

---

## 193. Deletion Race

Another controller/operator may delete the object between validation and deletion. Treat NotFound as already handled and log the race.

---

## 194. Policy Simulation

Before changing cleanup policy, run it in report-only mode against historical or current data and compare expected candidates with owner feedback.

---

## 195. Canary Cleanup

Start with one low-risk namespace or resource type, validate deletion and monitoring, then expand scope.

---

## 196. Progressive Rollout

Expand from development → staging → limited production namespaces → broader production only after measuring false positives and safety.

---

## 197. Rollback Policy

Cleanup is destructive, so rollback means restoring deleted resources through native controllers, backups, or recreation workflows. Do not assume deleted objects can always be recovered.

---

## 198. Deletion Recovery

For Jobs/Pods controlled by higher-level resources, controllers may recreate resources, but this is not guaranteed for standalone objects. Treat deletion as irreversible unless a recovery mechanism exists.

---

## 199. Pre-Deletion Export

If audit policy requires preserving selected object manifests, export the normalized metadata before deletion. Do not automatically export Secrets in plaintext.

---

## 200. Manifest Snapshot

For safe recovery of non-secret objects, store sanitized YAML/JSON manifests before deletion when approved.

---

## 201. Sanitization

If manifests are archived, remove Secret data and other sensitive fields unless encrypted storage and approved policy explicitly allow their retention.

---

## 202. Cleanup Savings

Estimate saved object count, API/list noise, and storage/log overhead. Do not claim infrastructure cost savings unless measured.

---

## 203. API Efficiency

Cleanup can improve operational hygiene, but Kubernetes API performance depends on cluster architecture. Measure API latency and request volume before claiming performance improvement.

---

## 204. Observability Noise

Removing stale resources can reduce dashboard/log clutter, especially completed Jobs and Pods, but retention should preserve enough incident evidence.

---

## 205. Monitoring Interaction

Cleanup should coordinate with Prometheus, ELK, and alerting so deletion of historical Kubernetes objects does not accidentally remove needed incident context from centralized systems.

---

## 206. Event Interaction

Events may disappear independently of object cleanup. Centralized event collection should be used when historical evidence is required.

---

## 207. SLO Interaction

Cleanup should never compromise application availability. Availability/SLO impact takes precedence over hygiene goals.

---

## 208. Cost vs Safety

A small amount of extra storage is usually preferable to deleting production evidence or state prematurely. Retention should be policy-driven, not optimized blindly for cost.

---

## 209. Production Deployment

Deploy the cleanup worker through Helm/ArgoCD as an EKS CronJob. Start in dry-run mode and keep execution disabled until policy validation and owner review are complete.

---

## 210. CronJob Configuration

Use concurrencyPolicy: Forbid, activeDeadlineSeconds, backoffLimit, successfulJobsHistoryLimit, failedJobsHistoryLimit, resource requests/limits, and a dedicated ServiceAccount.

---

## 211. Container Security

Run non-root, drop capabilities, disable privilege escalation, use a read-only filesystem where compatible, and use a minimal Python image.

---

## 212. Image Pinning

Pin the production image by digest where possible and retain the Git commit/configuration version in the deployment metadata.

---

## 213. Network Policy

Allow Kubernetes API access and only required observability/report destinations. Avoid unrestricted egress when cluster policy supports tighter controls.

---

## 214. CI Pipeline

Checkout → lint → unit tests → SAST → dependency scan → build → Trivy scan → integration tests → package/publish → deploy to staging → dry-run validation → production approval.

---

## 215. Jenkins Integration

Jenkins can execute manual or scheduled cleanup plans, publish reports, and require approval before production execution. Use short-lived identity or workload-based authentication.

---

## 216. GitHub Actions

GitHub Actions can test the policy engine and build the image. For AWS/Kubernetes access, prefer OIDC or short-lived credentials rather than static secrets.

---

## 217. ArgoCD Integration

ArgoCD manages the CronJob, RBAC, ConfigMap, and policy configuration. Policy changes should go through Git review.

---

## 218. Helm Values

Put namespace scope, retention thresholds, protected namespaces, maximum deletions, schedules, and alert settings in environment-specific Helm values.

---

## 219. ConfigMap

Non-secret cleanup policy can be mounted through a ConfigMap. Secrets should not be placed in the ConfigMap.

---

## 220. Policy Validation Job

Optionally run a separate validation job that checks policy syntax, protected namespaces, resource allowlists, and cluster identity before enabling execution.

---

## 221. Admission Guard

A cluster admission policy can enforce that cleanup workloads have required securityContext, ServiceAccount, and approved image provenance.

---

## 222. Security Scanning

Use SonarQube/SAST, dependency scanning, Trivy image scanning, and manifest policy checks in the existing DevSecOps pipeline.

---

## 223. Supply Chain

Use an approved base image, pinned dependencies, SBOM generation where required, image signing, and admission verification where the platform supports it.

---

## 224. Testing Strategy

Keep policy evaluation pure and deterministic. Mock Kubernetes APIs for unit tests and use an isolated EKS/Kubernetes environment for integration tests.

---

## 225. Policy Unit Tests

Test protected labels, protected namespaces, active resources, age boundaries, owner checks, lifecycle labels, risk levels, maximum deletion limits, and conflicting policies.

---

## 226. Age Boundary Test

Test exactly-at-threshold, just-before-threshold, and just-after-threshold timestamps. Boundary bugs are common in retention logic.

---

## 227. Protected Resource Test

Verify protected=true and do-not-delete labels override every deletion rule.

---

## 228. Namespace Test

Verify protected namespaces are blocked even when resources satisfy all other cleanup criteria.

---

## 229. Active Job Test

Verify active Jobs are never selected by completed/failed retention rules.

---

## 230. Completed Job Test

Verify completed Jobs are selected only after the configured retention period and only when not protected.

---

## 231. Failed Job Test

Verify failed Jobs use the correct, usually longer, retention period.

---

## 232. ReplicaSet Test

Verify active ReplicaSets and ReplicaSets referenced by current Deployment revisions are blocked.

---

## 233. Pod Test

Verify Running, Pending, CrashLoop, ImagePull, Succeeded, Failed, Evicted, and Terminating Pods receive the correct classification.

---

## 234. Owner Test

Verify owner references are parsed and unexpected/missing ownership causes safe blocking where required.

---

## 235. Finalizer Test

Verify resources with finalizers are not forcibly altered and are classified for investigation.

---

## 236. UID Test

Verify a candidate with an old UID cannot cause deletion of a replacement resource with the same name.

---

## 237. Revalidation Test

Change a candidate's labels/state between discovery and execution and verify deletion is skipped.

---

## 238. Plan Hash Test

Modify the candidate plan after approval and verify execution refuses to continue when plan hashing is enabled.

---

## 239. Deletion Limit Test

Verify the run aborts before deletion when candidate count exceeds the configured maximum.

---

## 240. Percentage Limit Test

Verify excessive candidate percentages trigger a safety stop.

---

## 241. Dry Run Test

Assert that discovery occurs but no delete API call is made in dry-run mode.

---

## 242. RBAC Test

Verify insufficient permissions produce a clear error and no destructive fallback behavior.

---

## 243. NotFound Test

Mock a deletion race and verify NotFound is recorded as already absent.

---

## 244. Conflict Test

Mock a resource change and verify the automation revalidates or safely skips deletion.

---

## 245. 429 Test

Mock throttling and verify bounded retries, jitter, and eventual failure classification.

---

## 246. Timeout Test

Mock API timeout and verify the run does not hang indefinitely.

---

## 247. Integration Environment

Use a dedicated test cluster or isolated namespaces with disposable resources. Never run destructive integration tests against production.

---

## 248. Failure Injection

Create test Jobs, Pods, ReplicaSets, labels, owner references, finalizers, and protected resources to validate policy behavior.

---

## 249. Restore/Recovery Test

For resource types that can be recreated by controllers, verify expected controller behavior after cleanup. For standalone objects, validate the documented recovery process.

---

## 250. Audit Test

Verify every deletion attempt has run ID, resource identity, policy version, decision, and outcome.

---

## 251. Metrics Test

Verify metrics counts match the execution report and do not contain high-cardinality resource labels.

---

## 252. Logging Test

Verify structured logs contain required fields and no secret values.

---

## 253. Security Test

Run the container with restricted security context and verify the application works without root or unnecessary Linux capabilities.

---

## 254. Performance Test

Measure discovery and deletion performance against realistic object counts. Tune concurrency based on Kubernetes API behavior rather than arbitrary worker counts.

---

## 255. Large Cluster Test

Use large synthetic datasets or a staging cluster to test memory use, API request count, pagination, and runtime.

---

## 256. Memory Test

Verify that object processing does not retain entire cluster objects unnecessarily. Use bounded queues and incremental processing.

---

## 257. Concurrency Test

Test bounded deletion workers and verify API throttling remains within acceptable levels.

---

## 258. Graceful Shutdown Test

Send SIGTERM during discovery and deletion and verify the job stops safely without claiming successful completion.

---

## 259. Exit Code Contract

Test documented exit codes for healthy/no candidates, candidates in dry-run, deletion failures, policy validation failures, and monitoring/API failures.

---

## 260. Production Troubleshooting: Wrong Cluster

Stop execution immediately, verify kube context/cluster ARN/account, inspect job configuration, and ensure destination/deletion permissions were not used against the wrong environment.

---

## 261. Production Troubleshooting: Too Many Candidates

Do not increase deletion limits. Switch to dry-run, compare policy version, protected scope, age calculation, selectors, and candidate reasons.

---

## 262. Production Troubleshooting: Unexpected Production Deletion

Stop the automation, revoke/delete execution permissions if necessary, preserve audit logs, identify affected resources, inspect controllers/backups, and perform incident response.

---

## 263. Production Troubleshooting: RBAC Forbidden

Check ServiceAccount, Role/ClusterRole, RoleBinding/ClusterRoleBinding, namespace scope, and recent GitOps changes. Do not grant cluster-admin as a quick fix.

---

## 264. Production Troubleshooting: Stuck Deletion

Inspect deletionTimestamp, finalizers, owner controller, API events, and dependent resources. Do not remove finalizers blindly.

---

## 265. Production Troubleshooting: API Throttling

Check overlapping jobs, concurrency, cluster size, API request volume, and retry behavior. Reduce workers and schedule frequency if required.

---

## 266. Production Troubleshooting: Job Cleanup Missed

Check Job completion timestamp, policy thresholds, owner labels, protected labels, namespace scope, and whether the native TTL controller already handles it.

---

## 267. Production Troubleshooting: Failed Jobs Deleted Too Early

Compare failed-job retention policy with approved requirements and policy version. Immediately protect affected resources from further deletion and investigate recovery of available evidence.

---

## 268. Production Troubleshooting: Old ReplicaSet Deleted

Check Deployment revision history and rollout/rollback state. Restore through the Deployment/GitOps workflow rather than manually recreating an old ReplicaSet unless required.

---

## 269. Production Troubleshooting: Pod Evidence Missing

Check whether cleanup retention was too short, centralized logs/events are available, and whether incident-protection labels were applied.

---

## 270. Production Troubleshooting: Cleanup Job Overlap

Inspect CronJob concurrencyPolicy, previous Job duration, retries, and whether multiple releases created duplicate cleanup CronJobs.

---

## 271. Production Troubleshooting: Cleanup Loop

Check whether the monitor is deleting objects that are immediately recreated by a controller or another automation. Identify the owner and exclude or redesign the policy.

---

## 272. Production Troubleshooting: Recreated Resources

A controller may recreate a deleted object because it is still desired. Remove the lifecycle conflict rather than repeatedly deleting the resource.

---

## 273. Production Troubleshooting: GitOps Conflict

If ArgoCD recreates a cleaned resource, determine whether it is desired state. Cleanup should not fight GitOps; change the lifecycle design or repository state.

---

## 274. Production Troubleshooting: Secret Risk

If a cleanup report contains secret data, treat it as a security incident, restrict access, rotate exposed credentials where necessary, and fix report sanitization.

---

## 275. Production Troubleshooting: Wrong Namespace

Validate namespace configuration and RBAC bindings. Protected namespaces should block execution even if the job is misconfigured.

---

## 276. Production Troubleshooting: Stale Plan

Reject the plan, regenerate candidates, and revalidate current UIDs/state. Never extend a stale approval window silently.

---

## 277. Production Troubleshooting: Plan Hash Mismatch

Treat it as a safety failure. Determine who changed the plan/configuration, review the diff, and require fresh approval.

---

## 278. Production Troubleshooting: Delete Timeout

Re-read the resource to determine whether it is absent, terminating, or still active. Avoid blind repeated deletes.

---

## 279. Production Troubleshooting: Partial Failure

Keep successful deletion results, classify failures, and schedule a targeted retry rather than rerunning an unrestricted cleanup with a new broad candidate set.

---

## 280. Production Troubleshooting: High API Load

Inspect list frequency, selectors, per-resource enrichment, concurrency, multiple monitor instances, and event queries. Optimize request count before increasing resources.

---

## 281. Production Troubleshooting: High Memory

Check whether the script stores all cluster objects/candidates in memory, report size, concurrency queues, and unbounded logs. Process incrementally.

---

## 282. Production Troubleshooting: High Runtime

Separate discovery, policy evaluation, deletion, verification, and reporting durations. Identify whether API latency or application processing is the bottleneck.

---

## 283. Production Troubleshooting: No Candidates

Verify scope, labels, age timestamps, resource states, policy version, and whether native Kubernetes TTL/lifecycle mechanisms already removed objects.

---

## 284. Production Troubleshooting: Zero Resources

A zero-result scan can mean a clean environment, wrong namespace, wrong cluster, insufficient permissions, or selector error. Validate expected scope before reporting success.

---

## 285. Production Troubleshooting: Metrics Missing

Check Prometheus scrape configuration, ServiceMonitor if used, metrics endpoint, network policy, and whether the short-lived CronJob design is scrapeable.

---

## 286. Production Troubleshooting: ELK Missing Logs

Check stdout logging, pod termination, log collector configuration, namespace filters, and structured JSON parsing.

---

## 287. Production Troubleshooting: Alert Failure

Check notification destination, network policy, credentials/identity, rate limits, and alert delivery metrics. A cleanup run should remain auditable even when notification fails.

---

## 288. Production Troubleshooting: Policy Drift

Compare deployed policy version with Git and approved configuration. Roll back unexpected changes and review recent commits.

---

## 289. Production Troubleshooting: CronJob Not Running

Check CronJob schedule, suspended state, ServiceAccount/RBAC, image pull, Job creation, resource quotas, and cluster scheduling.

---

## 290. Production Troubleshooting: ImagePullBackOff

Check image repository/tag/digest, registry access, ECR permissions, node network, image lifecycle, and image pull events.

---

## 291. Production Troubleshooting: Container Crash

Inspect logs, Python traceback, configuration validation, Kubernetes API connectivity, RBAC, and dependency versions.

---

## 292. Production Troubleshooting: Authentication Failure

Verify in-cluster configuration, ServiceAccount token projection, API server reachability, and AWS identity if EKS metadata is used.

---

## 293. Production Troubleshooting: Kube API 403

Review exact resource/verb/namespace requested and update only the required RBAC rule through GitOps/approved change control.

---

## 294. Production Troubleshooting: Kube API 429

Reduce concurrency, use selectors, increase schedule interval if appropriate, and tune retry/backoff.

---

## 295. Production Troubleshooting: Kube API 5xx

Check cluster/control-plane health and retry only transient errors with bounded attempts.

---

## 296. Production Troubleshooting: Resource Conflict

Re-fetch the object, compare UID/labels/owner/state, and rerun policy evaluation before any delete retry.

---

## 297. Production Troubleshooting: Finalizer

Identify the finalizer's controller and intended cleanup semantics. Do not remove it as a generic workaround.

---

## 298. Production Troubleshooting: Owner Missing

Treat unexpected missing ownership as a governance or orphan condition and require explicit policy before deletion.

---

## 299. Production Troubleshooting: Active Workload

If a supposedly stale object is active, block deletion and investigate why policy classified it incorrectly.

---

## 300. Production Troubleshooting: Protected Resource

Protected resources must remain untouched. Review why the resource was included in discovery and ensure the protection rule is evaluated first.

---

## 301. Production Troubleshooting: Deletion Limit Triggered

Stop safely, publish the candidate report, review the policy, and run a narrower approved scope rather than increasing the limit blindly.

---

## 302. Production Troubleshooting: Excessive Failures

If deletion failures exceed a threshold, stop the run and investigate shared causes such as RBAC, API availability, or finalizers.

---

## 303. Production Troubleshooting: Controller Churn

Deleting many controller-owned objects can cause controllers to recreate them and create API churn. Exclude or redesign the policy.

---

## 304. Production Troubleshooting: Storage Impact

Do not delete PVC/PV resources to reduce storage usage. Use storage-specific lifecycle and backup policies.

---

## 305. Production Troubleshooting: Compliance Conflict

If cleanup conflicts with retention or legal requirements, block deletion and escalate through the approved compliance process.

---

## 306. Production Troubleshooting: Incident Conflict

If an incident is active, protect relevant resources and preserve evidence until the incident owner approves cleanup.

---

## 307. Production Troubleshooting: Recovery

For accidentally deleted resources, identify controller ownership, available manifests/backups, GitOps desired state, and centralized logs/events. Do not assume generic Kubernetes recovery is possible.

---

## 308. Interview: Explain the Project

I built a Python Kubernetes cleanup automation for EKS that discovers explicitly approved stale resources, evaluates them through policy and safety rules, defaults to dry-run, and performs bounded, auditable deletion only after validation. The design includes protected namespaces/labels, owner and UID checks, revalidation, deletion limits, RBAC least privilege, retries, metrics, ELK logging, GitOps deployment, and production troubleshooting.

---

## 309. Interview: Why Python

Python provides a mature Kubernetes client and makes policy evaluation, reporting, testing, and integration with AWS/CI/CD straightforward. It is used for custom lifecycle logic rather than replacing native Kubernetes controllers unnecessarily.

---

## 310. Interview: Why Not kubectl Shell

Shell scripts become difficult to maintain when cleanup needs structured object parsing, owner relationships, policy evaluation, retries, audit reports, testing, and multi-resource correlation. Python provides stronger abstractions and testability.

---

## 311. Interview: Why Dry Run

Cleanup is destructive. Dry-run allows operators to inspect exactly what would be deleted and why before any mutation occurs.

---

## 312. Interview: Why Allowlist

An allowlist fails closed. A newly introduced Kubernetes resource kind cannot accidentally become deletable because it happens to match an age condition.

---

## 313. Interview: Protected Labels

A protected label gives application owners a clear escape hatch. It must be evaluated before eligibility and should override deletion rules.

---

## 314. Interview: Owner References

Owner references show controller relationships. They help determine whether an object is managed and whether deletion could cause controller churn or unexpected behavior.

---

## 315. Interview: UID

Names can be reused after deletion. UID ensures the cleanup decision refers to the exact object discovered during planning.

---

## 316. Interview: ResourceVersion

ResourceVersion helps detect changes between discovery and execution. For high-risk cleanup, I re-fetch and validate the object before deletion.

---

## 317. Interview: TOCTOU

A cleanup job has a time-of-check/time-of-use window. Revalidation of UID, state, owner, labels, and age reduces the risk of deleting a resource that became active after discovery.

---

## 318. Interview: Jobs

Completed and failed Jobs are common cleanup targets. I use completion/failure state, owner, age, protected labels, and retention policy rather than creation age alone.

---

## 319. Interview: TTL Controller

If simple Job retention is enough, I prefer Kubernetes TTL-after-finished. Python is justified for custom cross-resource policy, reporting, approval, or lifecycle logic.

---

## 320. Interview: Failed Pods

Failed Pods often contain valuable troubleshooting evidence, so I use longer retention and protect incident-related resources rather than deleting them immediately.

---

## 321. Interview: ReplicaSets

I only consider old ReplicaSets after verifying Deployment ownership, inactive revision state, retention policy, and current rollout/rollback status.

---

## 322. Interview: ConfigMaps/Secrets

I do not generically delete old ConfigMaps or Secrets because references can be indirect and deletion can break workloads. They require explicit lifecycle ownership.

---

## 323. Interview: PVCs

PVCs are excluded from generic cleanup because deletion can affect application data and storage reclaim behavior.

---

## 324. Interview: RBAC

The cleanup ServiceAccount gets only read access plus delete permissions for explicitly approved resources. Production deletion permissions are separated from discovery permissions where practical.

---

## 325. Interview: Wrong Cluster Protection

I validate the expected EKS cluster/account before deletion and require explicit production execution. A kubeconfig context mistake should fail closed.

---

## 326. Interview: API Throttling

I use server-side selectors, bounded concurrency, batching, client retries, exponential backoff, jitter, and deletion rate limits. I never solve throttling by increasing concurrency.

---

## 327. Interview: Idempotency

If an object disappears before deletion, I classify it as already cleaned. Re-running the job discovers only current eligible resources and does not rely on previous deletion attempts being repeated.

---

## 328. Interview: Approval

For high-risk production cleanup, I generate a plan containing exact resource UIDs and policy version, require approval, then revalidate the plan before execution.

---

## 329. Interview: Maximum Deletions

A hard deletion cap protects against policy bugs. If candidate count exceeds the cap, the job stops and produces a report instead of automatically deleting the entire candidate set.

---

## 330. Interview: GitOps

I treat GitOps desired state as a safety boundary. Cleanup should not fight ArgoCD by deleting resources that Git declares as desired.

---

## 331. Interview: Monitoring

I expose run status, candidate count, deleted count, blocked count, failures, duration, API throttling, and last successful run. Detailed resource identities remain in logs/reports rather than high-cardinality metrics.

---

## 332. Interview: Production Incident

If cleanup deletes unexpected resources, I stop execution, preserve audit evidence, revoke or disable deletion access if necessary, determine the blast radius, inspect controller/GitOps recovery paths, and treat it as a production incident.

---

## 333. Interview: EKS CronJob

I deploy the cleanup as a restricted EKS CronJob with a dedicated ServiceAccount, non-root security context, resource limits, active deadline, backoff limit, history limits, and concurrencyPolicy Forbid.

---

## 334. Interview: Testing

I unit-test pure policy rules with mocked Kubernetes objects and use isolated integration tests for RBAC, deletion, owner relationships, finalizers, and API behavior. Destructive tests never run against production.

---

## 335. Interview: Native vs Custom

I prefer native Kubernetes lifecycle controls such as TTL-after-finished and Deployment revisionHistoryLimit when they solve the requirement. Python is reserved for custom policy and cross-resource orchestration.

---

## 336. Interview: Cleanup Safety

My main safety mechanisms are dry-run default, resource allowlists, protected namespaces/labels, owner validation, UID checks, revalidation, deletion caps, bounded concurrency, explicit production approval, and complete audit reporting.

---

## 337. 60-Second Project Answer

I developed a Python Kubernetes cleanup automation for EKS that safely identifies stale Jobs, Pods, and old controller artifacts using explicit lifecycle policies. The automation is dry-run by default and uses resource allowlists, protected namespaces and labels, owner references, UID/resource-version validation, revalidation before deletion, bounded concurrency, deletion limits, and least-privilege RBAC. It integrates with Helm/ArgoCD, Jenkins/GitHub Actions, Prometheus/Grafana, and ELK, and includes production runbooks for API failures, accidental candidates, controller conflicts, finalizers, and policy drift. I also prefer native Kubernetes TTL and revision-history controls wherever they are sufficient.

---

## 338. Final Workflow

Validate identity → validate configuration → discover approved resources → normalize metadata → classify candidates → apply protected/safety rules → generate dry-run plan → approve if required → revalidate exact resources → delete bounded batches → verify state → publish audit report → emit metrics/logs → return documented exit code.

---

## 339. Final Checklist: Scope

[ ] Resource allowlist
[ ] Delete allowlist
[ ] Namespace scope
[ ] Protected namespaces
[ ] Protected workloads
[ ] Environment policy
[ ] Owner/lifecycle requirements

---

## 340. Final Checklist: Safety

[ ] Dry-run default
[ ] Explicit execution flag
[ ] Cluster/account validation
[ ] UID checks
[ ] Revalidation
[ ] Maximum deletions
[ ] Maximum percentage
[ ] Bounded concurrency
[ ] Rate limits
[ ] Stale-plan protection

---

## 341. Final Checklist: Kubernetes

[ ] Owner references
[ ] ResourceVersion where needed
[ ] Finalizers respected
[ ] DeletionTimestamp handled
[ ] Propagation policy explicit
[ ] Active resources protected
[ ] Native TTL considered
[ ] GitOps resources protected

---

## 342. Final Checklist: Security

[ ] Dedicated ServiceAccount
[ ] Least-privilege RBAC
[ ] No cluster-admin
[ ] Non-root container
[ ] No static AWS keys
[ ] No Secret logging
[ ] Restricted network policy
[ ] Dependency scanning
[ ] Trivy image scanning
[ ] Immutable image

---

## 343. Final Checklist: Observability

[ ] Run ID
[ ] Policy version
[ ] Candidate metrics
[ ] Deletion metrics
[ ] Blocked/failed metrics
[ ] Last-success metric
[ ] Structured ELK logs
[ ] Grafana dashboard
[ ] Alerting for failed cleanup
[ ] Audit trail

---

## 344. Final Checklist: Deployment

[ ] Helm/GitOps
[ ] EKS CronJob
[ ] concurrencyPolicy Forbid
[ ] activeDeadlineSeconds
[ ] backoffLimit
[ ] Job history limits
[ ] Resource requests/limits
[ ] Production approval
[ ] Rollback/recovery plan

---

## 345. Final Checklist: Testing

[ ] Policy unit tests
[ ] Age boundary tests
[ ] Protected-resource tests
[ ] Owner/UID tests
[ ] Revalidation tests
[ ] RBAC tests
[ ] API error tests
[ ] Integration tests
[ ] Large-cluster tests
[ ] Security tests

---

## 346. Final Production Principles

1. Cleanup is a destructive operation, not a simple script.
2. Default to dry-run.
3. Use explicit resource allowlists.
4. Protect namespaces and resources before evaluating age.
5. Never delete active workloads generically.
6. Prefer native Kubernetes lifecycle controls when they solve the problem.
7. Validate owner references and exact UIDs.
8. Revalidate immediately before deletion.
9. Enforce hard deletion limits.
10. Use bounded concurrency and API backoff.
11. Never remove finalizers blindly.
12. Keep Secrets/PVCs/CRDs/nodes/namespaces outside generic cleanup.
13. Respect GitOps desired state.
14. Monitor and audit every execution.
15. Treat unexpected deletion as a production incident.

---

## Repository Progress

```text
11-Python-DevOps-Projects/
├── 01-AWS-Resource-Automation.md        ✓
├── 02-EC2-Health-Monitor.md             ✓
├── 03-S3-Backup-Automation.md           ✓
├── 04-EKS-Pod-Monitor.md                ✓
├── 05-Kubernetes-Cleanup-Automation.md  ✓
├── 06-CI-CD-Automation.md
├── 07-Infrastructure-Health-Checker.md
└── 08-End-to-End-DevOps-Automation.md
```

**Next file: `06-CI-CD-Automation.md`**