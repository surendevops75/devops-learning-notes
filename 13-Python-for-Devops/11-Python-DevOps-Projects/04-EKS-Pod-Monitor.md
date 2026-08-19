# 11-Python-DevOps-Projects
# 04 — EKS Pod Monitor

> Production-oriented Python Kubernetes monitoring project for EKS pod health, workload correlation, troubleshooting, observability, alerting, and DevOps integration.

## Project Scope

```text
Kubernetes Python Client
EKS authentication
Pod discovery
Container state analysis
CrashLoopBackOff / OOMKilled
Readiness / Liveness / Startup probes
Pending / scheduling failures
Events
Node / PVC / Service correlation
Metrics Server
Prometheus + Grafana
ELK
Jenkins / GitHub Actions / ArgoCD
EKS CronJob
Production troubleshooting
```

## 1. Project Overview

Build a production-grade Python Kubernetes pod monitor for EKS. The monitor discovers pods, evaluates lifecycle state, container readiness, restart behavior, resource pressure, scheduling failures, probes, images, and recent events, then produces actionable health results and integrates with Prometheus, Grafana, ELK, CI/CD, and incident workflows.

---

## 2. Real-World Problem

Kubernetes can report a pod as Running while the application is unhealthy, or a pod can remain Pending because of scheduling, resource, taint, PVC, or image problems. A useful monitor must distinguish infrastructure, scheduling, workload, and application-level symptoms rather than treating Running as healthy.

---

## 3. Architecture

Scheduler/CronJob → Python Kubernetes client → API authentication → namespace/pod discovery → pod/container observation → event/resource correlation → health rules → state/alerting → JSON reports → Prometheus/ELK → Grafana/incident workflow.

---

## 4. Technology Stack

Python 3, kubernetes Python client, argparse, dataclasses, logging, pytest, unittest.mock, Docker, Kubernetes/EKS, Helm, Prometheus, Grafana, ELK, Jenkins, GitHub Actions, and optionally AWS APIs for EKS context and account validation.

---

## 5. Repository Structure

Recommended modules: cli.py, config.py, kube_client.py, discovery.py, pod_status.py, containers.py, events.py, resources.py, probes.py, health.py, state.py, alerts.py, reporters.py, metrics.py, logging_config.py, models.py, plus tests/unit and tests/integration.

---

## 6. Authentication

Inside EKS, prefer a dedicated Kubernetes ServiceAccount with appropriate RBAC. For AWS identity, use EKS workload identity where AWS APIs are required. Do not embed kubeconfig files or static AWS credentials in the container.

---

## 7. In-Cluster Configuration

Use kubernetes.config.load_incluster_config() when running inside Kubernetes. The pod receives API server and ServiceAccount context through the cluster environment.

---

## 8. Local Configuration

For development outside the cluster, use load_kube_config() and an explicitly selected kubeconfig context. Never assume the developer's current context is production.

---

## 9. Context Safety

Display or validate cluster/context identity before monitoring. A production monitor should fail closed if the expected cluster or environment does not match configuration.

---

## 10. RBAC Principle

Grant only read permissions needed for monitoring: pods, pod status, events, nodes where required, and workload objects only when the monitor correlates higher-level ownership.

---

## 11. RBAC Namespaces

If monitoring is namespace-scoped, bind a Role/RoleBinding rather than a cluster-wide ClusterRole whenever possible. Cluster-wide monitoring requires explicit justification.

---

## 12. ServiceAccount

Create a dedicated ServiceAccount for the monitor. Do not reuse the default ServiceAccount of an application namespace.

---

## 13. API Client

Create a Kubernetes CoreV1Api client for pods, events, nodes, and namespaces as required. Keep client construction injectable for testing.

---

## 14. Apps API

Use AppsV1Api when correlating pods with Deployments, ReplicaSets, StatefulSets, or DaemonSets. This can explain whether a pod is part of a controlled workload.

---

## 15. Batch API

Use BatchV1Api when the monitor must understand Jobs or CronJobs. A failed Job pod should be evaluated differently from a continuously running service.

---

## 16. API Timeouts

Use bounded Kubernetes API request timeouts. A monitoring cycle must not hang forever because an API call is stalled.

---

## 17. API Retry

Retry transient API connectivity or server failures with bounded backoff. Do not retry RBAC Forbidden or malformed requests indefinitely.

---

## 18. API Rate Pressure

Large clusters can produce significant API load. Use server-side selectors, namespace scoping, pagination where supported, bounded concurrency, and sensible polling intervals.

---

## 19. Discovery Strategy

Prefer list_namespaced_pod for namespace-focused monitoring or list_pod_for_all_namespaces only when cluster-wide visibility is required.

---

## 20. Label Selectors

Use Kubernetes label selectors to limit monitoring to approved workloads, environments, or application labels. Server-side filtering reduces API payload and processing.

---

## 21. Field Selectors

Field selectors can narrow objects by fields such as status.phase or spec.nodeName where supported. Use them carefully and verify API support for the resource.

---

## 22. Pagination

Use Kubernetes API pagination where supported for large collections. Test multi-page discovery and continuation behavior.

---

## 23. Namespace Scope

Define whether the monitor covers one namespace, a configured list, or the entire cluster. Never silently expand from namespace scope to cluster scope.

---

## 24. Pod Identity

Identify pods by namespace/name and UID. The UID distinguishes recreated pods that reuse the same name.

---

## 25. Pod UID

Use pod UID in internal state keys when tracking a specific pod lifecycle. Avoid using only pod name because controllers can recreate pods with the same name.

---

## 26. Pod Phase

Pod phases include Pending, Running, Succeeded, Failed, and Unknown. Phase is useful but insufficient for application health.

---

## 27. Pending Is Not One Problem

A Pending pod may be waiting for scheduling, image pull, volume attachment, admission, or another condition. Events and container state explain the actual cause.

---

## 28. Running Is Not Healthy

A Running pod can have unready containers, failing probes, high restarts, stale processes, or application-level failures. Readiness and container states must be evaluated.

---

## 29. Succeeded Pods

Succeeded pods can be healthy for Jobs but irrelevant or unexpected for long-running Deployments. Health rules must depend on workload type.

---

## 30. Failed Pods

Failed pods require context. A failed Job pod may be an expected transient execution failure or a critical batch failure depending on job policy.

---

## 31. Unknown Phase

Unknown should normally be treated as a monitoring or cluster communication concern and should not silently become healthy.

---

## 32. Container Status

Inspect containerStatuses and initContainerStatuses. Container state contains running, waiting, and terminated information that explains many pod failures.

---

## 33. Waiting State

Waiting reasons such as ContainerCreating, ImagePullBackOff, ErrImagePull, CrashLoopBackOff, CreateContainerConfigError, and CreateContainerError should map to actionable health conditions.

---

## 34. Terminated State

Terminated state provides exit code, reason, signal, start time, and finish time. These fields are important for diagnosing crashes and completed workloads.

---

## 35. Exit Code

Non-zero exit codes often indicate application failure, but interpretation is workload-specific. Exit code 137 commonly corresponds to SIGKILL and can be associated with OOM termination, but confirm the actual reason.

---

## 36. OOMKilled

If the terminated reason is OOMKilled, inspect container memory limits, actual usage metrics, workload behavior, and recent changes. Do not simply increase memory without evidence.

---

## 37. Restart Count

Track restartCount and its change over time rather than only using an absolute value. A pod with 100 historical restarts may be stable now; a pod gaining ten restarts in five minutes is actively unhealthy.

---

## 38. Restart Rate

Calculate restart rate over an observation window. This is more useful for alerting than a fixed lifetime restart count.

---

## 39. CrashLoopBackOff

CrashLoopBackOff indicates repeated container startup failures with backoff. Inspect current and previous logs, termination state, configuration, secrets, probes, dependencies, and resource limits.

---

## 40. Previous Logs

When a container has restarted, kubectl logs --previous is often critical. A Python monitor can report that previous-container diagnostics are required, but should not assume logs are available unless explicitly collected.

---

## 41. ImagePullBackOff

Inspect image name/tag, registry access, imagePullSecrets, IAM/workload identity for ECR where applicable, network access, and node-level image runtime conditions.

---

## 42. ErrImagePull

ErrImagePull indicates the image could not be retrieved. Events often contain the exact registry or authentication error.

---

## 43. CreateContainerConfigError

This commonly points to missing ConfigMaps, Secrets, invalid environment references, or other container configuration problems. Correlate pod events and spec references.

---

## 44. CreateContainerError

Inspect events and container runtime messages. Avoid generic remediation because the exact reason can vary.

---

## 45. Init Containers

A pod may remain unready because an init container is failing or repeatedly restarting. Monitor init-container states separately from application containers.

---

## 46. Init Failure

Common causes include dependency checks, migration scripts, permissions, missing files, or unavailable services. Report the init container name and reason.

---

## 47. Readiness Probe

Readiness determines whether a pod should receive traffic. A readiness failure does not necessarily mean the container process is dead.

---

## 48. Liveness Probe

Liveness failures can cause Kubernetes to restart a container. A monitor should distinguish probe-induced restarts from application crashes where evidence permits.

---

## 49. Startup Probe

Startup probes protect slow-starting applications from premature liveness failures. A startup failure can prevent normal readiness/liveness behavior.

---

## 50. Probe Failure Diagnosis

Inspect probe type, endpoint/command, port, timeout, period, failureThreshold, recent events, and application startup time.

---

## 51. Probe Alerting

Repeated readiness failures can be WARNING or CRITICAL based on traffic impact. A single transient readiness failure should not necessarily page an operator.

---

## 52. Pod Conditions

Inspect Pod conditions such as PodScheduled, Initialized, ContainersReady, and Ready. Conditions explain lifecycle progression better than phase alone.

---

## 53. PodScheduled

If PodScheduled is False, inspect scheduler events, resource availability, taints/tolerations, node selectors, affinity, topology constraints, and PVC issues.

---

## 54. Unschedulable

Common reasons include insufficient CPU/memory, untolerated taints, affinity constraints, unavailable zones, max-pod limits, and storage topology restrictions.

---

## 55. Node Selector

A pod may remain Pending because its nodeSelector does not match any node. Report the selector and available scheduling context when practical.

---

## 56. Node Affinity

Required node affinity can make a workload unschedulable. Distinguish required constraints from preferred scheduling preferences.

---

## 57. Taints and Tolerations

Untolerated node taints can prevent scheduling. A monitor can identify the taint/toleration mismatch from pod spec and node information.

---

## 58. Topology Constraints

TopologySpreadConstraints and zone constraints can prevent scheduling or reduce placement options. Include them when diagnosing Pending pods.

---

## 59. Resource Requests

Scheduler decisions use resource requests, not current utilization. A pod can be Pending even when average cluster CPU usage appears low if allocatable capacity does not satisfy the request.

---

## 60. Resource Limits

Limits constrain container resource usage. Memory limit breaches can lead to OOMKilled; CPU limits can cause throttling without necessarily killing the container.

---

## 61. CPU Throttling

High CPU throttling can degrade application performance while the pod remains Running. Kubernetes resource metrics and cgroup/container metrics are needed for deeper diagnosis.

---

## 62. Memory Pressure

Memory pressure can cause OOMKills or node eviction. Correlate pod resource usage, limits, QoS class, and node pressure.

---

## 63. Ephemeral Storage

Pods can fail or be evicted when ephemeral-storage consumption exceeds limits or node capacity. Include ephemeral storage when production workloads depend on local files.

---

## 64. Resource Metrics

The Kubernetes API does not provide all real-time resource metrics by default. Metrics Server commonly supplies CPU/memory usage for kubectl top and autoscaling-related use cases.

---

## 65. Metrics Server

If the monitor requires current CPU/memory usage, integrate with the Metrics API where Metrics Server is installed. Treat missing metrics as UNKNOWN rather than zero.

---

## 66. Metrics API

Use the Kubernetes metrics API for resource usage where available. Keep metrics collection optional so basic pod lifecycle monitoring can still work when the metrics pipeline is unavailable.

---

## 67. Metric Freshness

Validate metric timestamps or collection freshness. Stale metrics can lead to false health conclusions.

---

## 68. Resource Thresholds

Example policy: warning at 80% memory-limit utilization and critical at 95%, but tune thresholds to workload behavior. CPU usage above 90% is not automatically unhealthy.

---

## 69. Limit Utilization

Compare usage to requests and limits deliberately. Usage percentage relative to a limit answers a different question from usage relative to a node or requested capacity.

---

## 70. QoS Classes

Kubernetes QoS classes are Guaranteed, Burstable, and BestEffort. QoS influences eviction behavior under node pressure and should be considered during diagnosis.

---

## 71. Guaranteed QoS

A Guaranteed pod has stricter resource configuration and can receive stronger protection during some resource-pressure scenarios compared with lower QoS classes.

---

## 72. Burstable QoS

Burstable workloads have requests and limits that may differ. This is common for production services but requires careful capacity and eviction analysis.

---

## 73. BestEffort QoS

BestEffort pods have no CPU/memory requests or limits and can be more vulnerable under resource pressure.

---

## 74. Node Conditions

When pods fail or are evicted, inspect node conditions such as Ready, MemoryPressure, DiskPressure, PIDPressure, and NetworkUnavailable where applicable.

---

## 75. Node NotReady

A pod may be affected by a NotReady node even if the pod itself reports Running. Correlate pod nodeName with node conditions and recent node events.

---

## 76. Node Pressure

DiskPressure or MemoryPressure can cause evictions and scheduling changes. The pod monitor should surface the node as a correlated dependency rather than blaming the pod alone.

---

## 77. Evicted Pods

Evicted pods often contain a reason/message associated with node resource pressure. Report eviction reason and node context.

---

## 78. Pod Disruption Budget

PDBs protect availability during voluntary disruptions. They are relevant when many pods are being disrupted, though a PDB does not prevent node failure.

---

## 79. Deployment Correlation

Map pods to Deployments through ownerReferences and ReplicaSets. This allows alerts to identify the owning application rather than only a pod name.

---

## 80. ReplicaSet Correlation

A Deployment-created pod is usually owned by a ReplicaSet, which is then owned by the Deployment. Walk owner references when a direct relationship is not present.

---

## 81. StatefulSet Correlation

StatefulSet pods have stable identities and storage relationships. A failing StatefulSet pod may require different recovery handling from a stateless Deployment pod.

---

## 82. DaemonSet Correlation

DaemonSet pods are expected across eligible nodes. A missing DaemonSet pod may indicate node eligibility, taint, scheduling, or controller issues.

---

## 83. Job Correlation

Job pods should be evaluated using Job completions, failures, backoffLimit, and expected execution state.

---

## 84. CronJob Correlation

CronJob-created pods are transient. Avoid treating every Succeeded pod as a failure simply because it is not Running.

---

## 85. Owner References

Use ownerReferences to correlate pod health with the workload controller. Record controller kind/name in health output.

---

## 86. Orphaned Pods

A pod without an expected controller can be a governance finding or a legitimate standalone workload. Classify it separately rather than automatically deleting it.

---

## 87. Events

Kubernetes Events often contain the most actionable explanation for Pending, image, mount, probe, and scheduling failures.

---

## 88. Event API

Use CoreV1Api event listing with appropriate namespace and field selectors. Events are ephemeral and should be collected promptly.

---

## 89. Event Ordering

Do not assume events are perfectly ordered. Sort by timestamp where possible and treat the current pod/node state as authoritative.

---

## 90. Event Correlation

Correlate events by involvedObject name, namespace, UID where available, and reason. Avoid reporting unrelated events from the same namespace.

---

## 91. Event Retention

Kubernetes event retention is limited and cluster-dependent. A monitoring system should export important events to centralized logs if historical investigation is required.

---

## 92. Common Event Reasons

Useful reasons include FailedScheduling, Failed, BackOff, Unhealthy, FailedMount, FailedAttachVolume, Pulling, Pulled, and Killing.

---

## 93. PVC Issues

A pod can remain Pending because a PersistentVolumeClaim is pending or a volume cannot attach/mount. Correlate pod volume references with PVC/PV state when storage diagnosis is required.

---

## 94. Storage Correlation

For stateful workloads, inspect PVC status, storage class, access mode, topology, attach failures, and recent events.

---

## 95. Network Correlation

A Running/Ready pod does not prove service connectivity. Application health may require Service, EndpointSlice, DNS, NetworkPolicy, and application-level checks.

---

## 96. Service Correlation

For a web service, correlate pod readiness with Service selectors and EndpointSlice membership. A ready pod that is not selected can still be unreachable through the Service.

---

## 97. EndpointSlice

EndpointSlices represent backend endpoints for Services. They can help diagnose cases where pods are Ready but traffic is not reaching expected endpoints.

---

## 98. Ingress Correlation

For ALB ingress workloads, correlate pod readiness with Service endpoints and ingress target health. An application can fail at the ingress layer even when pods look healthy.

---

## 99. EKS ALB Context

In an EKS environment using AWS Load Balancer Controller, target health can provide another signal beyond Kubernetes pod status. Keep AWS API access optional and separately permissioned.

---

## 100. Health Model

Represent HEALTHY, WARNING, CRITICAL, and UNKNOWN explicitly. Separate health severity from Python log levels.

---

## 101. Condition Codes

Use stable machine-readable codes such as POD_PENDING, CRASH_LOOP, OOM_KILLED, IMAGE_PULL_FAILURE, NOT_READY, PROBE_FAILURE, NODE_NOT_READY, EVICTED, METRICS_UNAVAILABLE, and SCHEDULING_FAILURE.

---

## 102. Rule Precedence

Critical conditions override warnings, but preserve every condition in the result. Example: OOMKilled plus readiness failure remains CRITICAL with both evidence items.

---

## 103. Workload-Aware Rules

Rules must consider workload type. A Succeeded Job pod is healthy when expected, while a Succeeded Deployment pod would be abnormal.

---

## 104. Restart Policy

Respect pod restartPolicy when interpreting terminated containers. Always, OnFailure, and Never have different expected behaviors.

---

## 105. Healthy Criteria

A typical long-running service pod is healthy when it is Running, required containers are Ready, there are no active critical waiting/terminated states, restart rate is normal, and required metrics/dependencies are available.

---

## 106. Unknown Criteria

Return UNKNOWN when essential monitoring data is unavailable or the monitor cannot safely determine state. Do not convert missing API/metric data into healthy.

---

## 107. State Tracking

Persist previous health state when detecting transitions. State keys should include cluster identity, namespace, pod UID, and condition code where appropriate.

---

## 108. Pod Lifecycle State

Pod names can be reused after deletion, so include UID in lifecycle state. Clean up state for deleted pods after a defined retention period.

---

## 109. Alert Deduplication

Use a deterministic key such as cluster|namespace|pod_uid|condition. This prevents repeated alerts on every polling cycle.

---

## 110. Alert Cooldown

Use state transitions or cooldown periods to prevent alert storms. Escalate when severity changes or a condition remains unhealthy beyond policy.

---

## 111. Recovery Alerts

Generate recovery notifications when a previously active critical condition returns to healthy if incident policy requires it.

---

## 112. Alert Aggregation

Aggregate failures by workload, namespace, node, or cluster during broad incidents so operators receive useful summaries instead of thousands of individual alerts.

---

## 113. Cluster Incident Detection

If many pods across namespaces fail simultaneously on one node or cluster, identify the shared dependency and raise a cluster/node-level incident rather than treating each pod independently.

---

## 114. Node Blast Radius

Count affected pods per node. A high concentration can indicate node-level failure, pressure, networking, or runtime problems.

---

## 115. Namespace Blast Radius

A namespace-wide spike can indicate quota, policy, secret/configuration, admission, or namespace-level dependency issues.

---

## 116. Deployment Blast Radius

If multiple replicas of the same Deployment become unhealthy together, prioritize deployment configuration, image, application, dependency, or rollout issues.

---

## 117. Rollout Correlation

Correlate recent Deployment/ReplicaSet changes with pod failures. A new ReplicaSet with widespread failures is a strong deployment signal.

---

## 118. Image Correlation

If many pods fail after an image change, compare image references/digests and rollout timestamps. Tags alone are less reliable than immutable digests for forensic accuracy.

---

## 119. Image Digest

Report the image digest when available. Digests identify the exact image content more reliably than mutable tags.

---

## 120. Image Security

A pod monitor should not become an image vulnerability scanner. Integrate vulnerability data from the DevSecOps pipeline or registry when a security posture view is required.

---

## 121. Secrets

Do not log Secret values. If a pod fails because a Secret is missing or inaccessible, report the Secret name only when permitted and avoid exposing content.

---

## 122. ConfigMaps

Configuration failures can cause CreateContainerConfigError or application startup failures. Report missing references without exposing sensitive configuration data.

---

## 123. Environment References

Check env and envFrom references for missing ConfigMaps or Secrets when diagnosis requires spec inspection.

---

## 124. Admission Failures

Pods can be rejected before creation by admission policies. Such failures may not appear as pod failures and should be monitored through API/server/audit signals when required.

---

## 125. Policy Correlation

Security policies, Pod Security Admission, Kyverno, Gatekeeper, or other admission systems can prevent workloads from starting. The monitor should distinguish admission failures from runtime failures.

---

## 126. Security Context

A container can fail because a security context prevents required operations. Do not recommend weakening security controls as the default remediation.

---

## 127. Privileged Containers

A monitor should report security posture findings separately from runtime health when governance requires it.

---

## 128. NetworkPolicy

NetworkPolicy can cause application connectivity failures while pod status remains healthy. Application-level synthetic checks may be needed to detect this.

---

## 129. DNS

Cluster DNS failures can affect applications without changing pod phase. Correlate application errors, CoreDNS health, and service discovery when diagnosing dependency failures.

---

## 130. ServiceAccount

A pod can be Running but fail API calls because its ServiceAccount lacks permissions. Report authorization failures only through approved application/log signals.

---

## 131. Node Runtime

Container runtime problems can produce image, startup, or termination failures. Correlate pod events with node conditions and runtime logs when available.

---

## 132. Kubelet

Kubelet health can affect pod lifecycle reporting and readiness. If multiple unrelated pods on one node fail, inspect node/kubelet signals.

---

## 133. Control Plane

API server or control-plane problems can cause monitoring data to be stale or unavailable. Distinguish cluster observability failure from application failure.

---

## 134. API Availability

Record Kubernetes API call failures separately from pod health so operators know when the monitor itself lacks reliable data.

---

## 135. Monitoring Heartbeat

Expose a last-successful-scan timestamp and scan duration. A stale heartbeat is itself an alert condition.

---

## 136. Prometheus Metrics

Useful metrics include monitor_runs_total, monitor_failures_total, scan_duration_seconds, pods_by_phase, unhealthy_pods, restart_events, api_errors_total, and last_success_timestamp.

---

## 137. Metric Labels

Use bounded labels such as cluster, environment, namespace, workload_kind, severity, and reason. Avoid pod name, pod UID, container ID, or image digest as persistent Prometheus labels at scale.

---

## 138. Metric Cardinality

High-cardinality labels can overload Prometheus. Put detailed pod identity in structured logs or event records instead.

---

## 139. Grafana Dashboard

Recommended panels: unhealthy pods by namespace, pending pods, restart spikes, OOMKills, image-pull failures, node concentration, API errors, scan duration, and monitor freshness.

---

## 140. ELK Logging

Send structured JSON logs to ELK for detailed troubleshooting. Include cluster, namespace, pod name, UID, workload, condition, reason, event summary, and run ID where appropriate.

---

## 141. Log Volume

Avoid logging every healthy pod every cycle at INFO. Use aggregate summaries and emit detailed records for unhealthy or changed states.

---

## 142. Run ID

Generate an opaque UUID per scan and include it in logs/reports. Never use sensitive identifiers or credentials as run IDs.

---

## 143. JSON Report

A report can include schema version, cluster, timestamp, scan duration, totals, health counts, pod records, conditions, workload owners, and event summaries.

---

## 144. CSV Report

CSV can provide operational exports with stable columns such as namespace, pod, UID, workload, phase, ready, restarts, severity, and primary condition.

---

## 145. Atomic Report

Write reports to temporary storage and rename atomically when local files are used so downstream consumers do not read partial output.

---

## 146. Configuration

Support CLI arguments, environment variables, and config files with a documented precedence such as CLI > environment > file > defaults.

---

## 147. Configuration Validation

Validate namespaces, thresholds, intervals, selectors, expected cluster identity, resource limits, and alert settings before querying the API.

---

## 148. Threshold Validation

Reject negative intervals, invalid percentages, empty cluster identity, and contradictory warning/critical thresholds before execution.

---

## 149. Polling Interval

Choose the polling interval based on incident detection requirements and API load. Very frequent polling can create unnecessary API pressure.

---

## 150. Observation Window

Use a time window for restart-rate and resource rules rather than reacting to one instantaneous observation.

---

## 151. Consecutive Failures

Require multiple consecutive unhealthy observations for noisy conditions such as transient readiness or CPU pressure when immediate paging is not justified.

---

## 152. Hysteresis

Use separate trigger and recovery thresholds when resource signals oscillate near a boundary. This reduces alert flapping.

---

## 153. Concurrency

Use bounded ThreadPoolExecutor only where API calls are independent and the cluster/API server can tolerate the load. Namespace or cluster isolation may be safer than unrestricted concurrency.

---

## 154. Per-Cluster Limits

Maintain separate concurrency limits per cluster to prevent one large cluster from consuming all monitoring workers.

---

## 155. Multi-Cluster

For multiple EKS clusters, store approved cluster contexts/endpoints and identities in configuration. Validate each cluster independently and continue only where partial monitoring is acceptable.

---

## 156. Multi-Account

If AWS metadata is required, use a central role that can assume approved roles into target accounts. Validate STS identity before AWS-side operations.

---

## 157. Cluster Identity

Kubernetes API endpoints alone are not always friendly identifiers. Configure an expected cluster name/environment and record AWS EKS cluster identity when available.

---

## 158. EKS API Integration

Use boto3 EKS APIs only when the monitor needs cluster metadata or AWS-specific signals. Keep Kubernetes API monitoring functional without unnecessary AWS permissions.

---

## 159. AWS Load Balancer Correlation

If using ALB ingress, optionally correlate pod/service state with AWS target health through the AWS Load Balancer Controller or ELB APIs. Treat this as a separate collector.

---

## 160. Collector Isolation

Design Kubernetes, Metrics API, Events, and optional AWS collectors independently so failure in one does not corrupt all health results.

---

## 161. Collector Results

Each collector should return data plus a clear error state. This enables the health engine to classify UNKNOWN rather than assuming missing data is healthy.

---

## 162. Pure Health Engine

Keep health evaluation as deterministic functions/classes over normalized observations. Do not place API calls inside the decision rules.

---

## 163. Dependency Injection

Inject Kubernetes clients, AWS clients, clock functions, notifiers, reporters, and configuration so unit tests can replace them with fakes.

---

## 164. Dataclasses

Use dataclasses for normalized PodObservation, ContainerObservation, HealthResult, AlertState, ScanSummary, and Configuration models.

---

## 165. Schema Version

Version report schemas so dashboards, automation, and downstream consumers can evolve without breaking old reports.

---

## 166. Error Taxonomy

Classify failures into authentication, authorization, API availability, rate limiting, invalid configuration, metrics unavailable, event unavailable, and workload health.

---

## 167. Exit Codes

A CLI can use 0 for healthy, 1 for warning, 2 for critical, and 3 for monitoring/system failure. Document the contract for Jenkins/GitHub Actions.

---

## 168. Dry Run

A monitoring dry-run should perform read-only discovery and show intended classifications without sending alerts or making mutations.

---

## 169. No Mutation Principle

The core pod monitor should be read-only. Restarting pods, deleting resources, or scaling workloads belongs to a separate, explicitly controlled remediation system.

---

## 170. Auto-Remediation Boundary

If remediation is added, require allowlisted actions, namespace/workload eligibility, account/cluster validation, cooldown, maximum attempts, audit logging, and post-action verification.

---

## 171. Restart Remediation Risk

Automatically deleting or restarting a pod can worsen an outage, especially for StatefulSets or single-replica services. Diagnose first and follow workload-specific runbooks.

---

## 172. Deployment Rollback

Rollback should normally be performed through the deployment/GitOps workflow rather than arbitrary pod deletion. The monitor should surface evidence and optionally trigger an approved workflow.

---

## 173. GitOps Integration

For ArgoCD-managed applications, the monitor can correlate pod failures with Application sync status and recent Git revisions when appropriate. It should not directly modify the GitOps desired state.

---

## 174. Jenkins Integration

Jenkins can execute scheduled cluster health checks and fail a pipeline when critical conditions exist. Use scoped Kubernetes credentials or workload identity rather than broad static credentials.

---

## 175. GitHub Actions

GitHub Actions can run integration tests or controlled monitoring jobs. Prefer short-lived identity and avoid storing long-lived kubeconfig credentials in repository secrets.

---

## 176. EKS CronJob

Deploy the monitor as a CronJob with a dedicated ServiceAccount, RBAC, resource requests/limits, activeDeadlineSeconds, backoffLimit, history limits, and concurrencyPolicy Forbid.

---

## 177. CronJob Overlap

If scan duration can exceed the schedule interval, prevent overlapping jobs. Overlap causes duplicate alerts and unnecessary API pressure.

---

## 178. Container Security

Run the monitor as non-root, drop Linux capabilities, disable privilege escalation, use a read-only filesystem where compatible, and use a minimal image.

---

## 179. Image Pinning

Pin the production monitor image by immutable digest where possible. Mutable tags can make troubleshooting and rollback harder.

---

## 180. Dependency Security

Pin or constrain Python dependencies, run dependency vulnerability scanning, and review Kubernetes client upgrades for API compatibility.

---

## 181. Network Policy

If cluster policy supports it, allow the monitor only the API server and required telemetry destinations. Avoid unrestricted egress.

---

## 182. Secret Management

Do not put kubeconfig, AWS access keys, Slack tokens, or other secrets in the image or source code. Use Kubernetes Secrets or an approved external secret system only when required.

---

## 183. Pod Security

Apply a restrictive security context and Pod Security Admission-compatible configuration. Monitoring should not require privileged access.

---

## 184. Resource Limits

Set resource requests/limits based on measured pod count and scan behavior. Avoid excessive memory by streaming API results and summaries.

---

## 185. Large Cluster Memory

Do not retain every Pod object indefinitely. Normalize only required fields and process collections incrementally where possible.

---

## 186. Large Cluster Design

For very large clusters, use namespace partitioning, selectors, bounded workers, incremental state, and event-driven triggers combined with periodic reconciliation.

---

## 187. Event-Driven Extension

Kubernetes watch APIs can reduce polling latency, but watches must handle disconnects, resource versions, relists, and missed events. Keep periodic reconciliation as a safety net.

---

## 188. Watch API

A watch stream can receive Added, Modified, and Deleted events. Treat it as a change stream, not as permanent truth; reconnect and resynchronize when the stream becomes invalid.

---

## 189. Resource Version

Kubernetes watches use resourceVersion for consistency. If the server reports that the requested resource version is too old, perform a fresh list and restart the watch.

---

## 190. Watch Failure

Network interruption, API server restarts, timeouts, and expired resource versions can terminate a watch. The monitor must recover without losing the authoritative current state.

---

## 191. Event vs Watch

Kubernetes Events describe causes; object watches describe resource changes. They solve different problems and can be correlated.

---

## 192. Queue Architecture

A watch-based monitor can push pod change events into a bounded queue and let worker logic recompute affected health states.

---

## 193. Backpressure

Use bounded queues so an event burst cannot consume unlimited memory. Coalesce repeated updates for the same pod when possible.

---

## 194. Periodic Reconciliation

Even an event-driven monitor should periodically list current state to recover from missed events, restarts, or state-store inconsistencies.

---

## 195. Node Correlation

When a pod issue is detected, inspect node state only when required or when multiple affected pods suggest a shared node problem.

---

## 196. Node API Permissions

Node visibility is cluster-wide and sensitive in some environments. Do not request node permissions if namespace-only monitoring does not need them.

---

## 197. Events RBAC

Events may be namespace-scoped or cluster-wide depending on collection design. Request only the scope required by the monitor.

---

## 198. Metrics RBAC

Metrics API access requires appropriate authorization. Validate it independently from core pod access.

---

## 199. API Forbidden

A 403 Forbidden should produce a clear RBAC diagnosis. Do not repeatedly retry an authorization failure.

---

## 200. API Unauthorized

A 401 Unauthorized suggests expired/invalid credentials or authentication configuration. In-cluster ServiceAccount identity should be checked.

---

## 201. API Timeout

Timeouts can indicate API server load, network policy, DNS, or client configuration. Record duration and endpoint context without exposing sensitive endpoint details.

---

## 202. API 429

HTTP 429 or equivalent rate limiting should trigger backoff and reduced concurrency rather than aggressive retries.

---

## 203. API 5xx

Transient server errors can be retried with bounded backoff. Persistent 5xx responses should surface as monitoring failure.

---

## 204. Stale Data

Record collection timestamps and distinguish current observations from cached or previous state.

---

## 205. Partial Cluster View

If pod discovery succeeds but metrics or events fail, report partial observability explicitly. Do not label all pods healthy solely from the available subset.

---

## 206. Monitoring Confidence

Optionally include a confidence field based on which collectors succeeded. A health result derived from incomplete data should be visibly marked.

---

## 207. Health Summary

Summarize total pods, healthy, warning, critical, unknown, pending, restarts, OOMKills, image failures, and API/collector failures.

---

## 208. Workload Summary

Group health by Deployment, StatefulSet, DaemonSet, Job, CronJob, namespace, and node to identify blast radius.

---

## 209. Namespace Summary

A namespace dashboard should show unhealthy pods, pending pods, restart rate, and top failure reasons.

---

## 210. Cluster Summary

Cluster-level summary should show API availability, node readiness, unhealthy workloads, resource pressure, and monitor freshness.

---

## 211. Alert Priority

Page only conditions requiring immediate human action; route lower-severity findings to dashboards or ticketing. Avoid paging on every pod restart.

---

## 212. Alert Routing

Route by environment, namespace, workload owner, or severity. Keep routing configuration outside the health engine.

---

## 213. Notification Interface

Implement notifier interfaces for Slack, email, SNS, PagerDuty-like systems, or ticketing. Unit tests can use a fake notifier.

---

## 214. Alert Failure

If alert delivery fails, record it as a monitoring failure and expose a metric. Detection without notification is an operational gap.

---

## 215. Incident Correlation

A single node failure affecting many pods should generate a shared incident context. A deployment image failure affecting many replicas should be correlated separately.

---

## 216. Deployment Incident

When many pods of one Deployment fail after a rollout, inspect rollout revision, image digest, ConfigMaps, Secrets, probes, resources, and application logs.

---

## 217. Node Incident

When unrelated workloads fail on one node, inspect node conditions, kubelet/runtime, networking, disk, memory, and recent node events.

---

## 218. Cluster Incident

When failures span many nodes/namespaces, inspect control plane/API health, cluster networking, DNS, admission, storage, and cloud-provider dependencies.

---

## 219. Storage Incident

When many pods fail to mount volumes, inspect CSI controller/node components, storage class, cloud volume API, node attachment limits, and recent events.

---

## 220. Network Incident

When pods are Ready but applications fail connectivity, inspect Service/EndpointSlice, DNS, NetworkPolicy, ingress/load balancer, and application-level checks.

---

## 221. Image Registry Incident

If many pods suddenly show ImagePullBackOff, check registry availability, ECR permissions, image lifecycle, network connectivity, and image references.

---

## 222. Configuration Incident

If many pods show configuration errors after a deployment, inspect ConfigMap/Secret versions, references, admission policies, and rollout history.

---

## 223. Probe Incident

If many replicas fail readiness/liveness after a release, compare probe configuration and application startup behavior with the previous revision.

---

## 224. Memory Incident

For repeated OOMKilled containers, inspect usage versus limit, request, JVM/Node/Python runtime behavior, traffic, leaks, and recent configuration changes.

---

## 225. Restart Storm

For high restart rates, inspect termination reasons, logs, probes, dependency failures, and resource pressure. Avoid automatically increasing limits without evidence.

---

## 226. Pending Storm

For widespread Pending pods, inspect node capacity, resource requests, taints, affinity, quotas, topology constraints, and autoscaler state.

---

## 227. Quota Correlation

Namespace ResourceQuota or LimitRange can prevent pod creation or constrain resources. Include quota signals when diagnosing admission/scheduling failures.

---

## 228. HPA Correlation

For HPA-managed workloads, correlate replica count, desired/current metrics, and pod health. Scaling pressure can be a symptom rather than the root cause.

---

## 229. VPA Correlation

If Vertical Pod Autoscaler is used, consider its recommendations and update behavior when analyzing resource pressure, but avoid making the monitor dependent on VPA.

---

## 230. Cluster Autoscaler

A Pending pod due to insufficient capacity may be expected while Cluster Autoscaler scales nodes. Check autoscaler events/state before escalating permanently.

---

## 231. Autoscaler Delay

Distinguish a short-lived Pending period during legitimate scaling from a persistent scheduling failure.

---

## 232. PDB Correlation

During voluntary disruptions, PDB constraints can delay evictions and rollout operations. Use PDB information when diagnosing blocked maintenance or rollout behavior.

---

## 233. Rollout Availability

A Deployment can have insufficient available replicas even when some pods are healthy. Correlate desired, updated, ready, and available replica counts.

---

## 234. Deployment Health

For higher-level monitoring, calculate deployment availability rather than only pod health. One unhealthy pod may be tolerable if redundancy remains above the service SLO.

---

## 235. Replica Health

Define workload-level thresholds such as available replicas percentage or minimum healthy replicas. These are often more operationally meaningful than one-pod alerts.

---

## 236. Stateful Availability

StatefulSet health can depend on ordered startup and persistent storage. Evaluate unavailable replicas and volume state carefully.

---

## 237. DaemonSet Availability

DaemonSet health can be measured against desired/current/ready nodes, accounting for scheduling exclusions.

---

## 238. Job Health

Job health should consider completions, failed count, active pods, backoffLimit, and deadline rather than standard service readiness rules.

---

## 239. CronJob Health

CronJob monitoring should detect missed schedules, failed Jobs, and abnormal execution duration while accepting successful completed Jobs.

---

## 240. Pod Age

Pod age can help identify newly created pods during a rollout or frequent recreation. Age alone is not a health signal.

---

## 241. Stuck Termination

A pod stuck Terminating may indicate finalizers, volume detach issues, API problems, or node failure. Report duration and relevant events.

---

## 242. Finalizers

Finalizers can prevent deletion completion. Investigate the owning controller and finalizer purpose before removing anything manually.

---

## 243. Deletion Timestamp

A deletionTimestamp indicates that Kubernetes has begun termination. Do not classify such a pod as a normal healthy Running workload.

---

## 244. Graceful Shutdown

Long termination durations may indicate application shutdown behavior, finalizers, volume operations, or node conditions. Compare with terminationGracePeriodSeconds.

---

## 245. PreStop Hook

A preStop hook can extend termination and can fail. Include it in troubleshooting when pods remain in Terminating or rollout takes too long.

---

## 246. Termination Grace

Do not recommend reducing termination grace blindly; it may interrupt data flushing or connection draining.

---

## 247. Readiness During Shutdown

Applications should become unready before termination where appropriate so traffic drains safely.

---

## 248. ALB Target Health

For ALB-backed services, a pod can be Kubernetes Ready while the target remains unhealthy because of port, path, security group, application response, or controller configuration.

---

## 249. Ingress Troubleshooting

Correlate ingress/controller events, Service endpoints, target health, listener rules, and application response when users report HTTP failures despite healthy pods.

---

## 250. Service Selector

A mismatch between Service selectors and pod labels can produce zero endpoints even when pods are Ready.

---

## 251. Endpoint Health

Check EndpointSlice membership to verify that expected ready pods are represented as service backends.

---

## 252. Application Health

The monitor can provide infrastructure signals, but application-level synthetic checks should be used when the business requirement is end-to-end availability.

---

## 253. Synthetic Checks

An optional HTTP/TCP check can test a service endpoint. Keep it separate from pod monitoring and protect credentials used for authenticated endpoints.

---

## 254. Probe vs Synthetic

A readiness probe is local to Kubernetes traffic decisions; a synthetic check tests an external path through service/ingress dependencies. Both provide different signals.

---

## 255. Log Integration

Do not automatically stream every pod log through the monitor. Centralized logging such as ELK should handle log collection; the monitor should report relevant log access links or diagnostic hints.

---

## 256. ELK Correlation

Use pod UID, namespace, workload, and container name as structured log fields for incident correlation without creating Prometheus cardinality problems.

---

## 257. Prometheus Correlation

Use stable workload/namespace labels in metrics and link to detailed logs using dashboard variables or log exploration.

---

## 258. Grafana Alerting

Prometheus/Grafana can handle many alert conditions natively. Python should own custom correlation or orchestration that cannot be expressed cleanly in PromQL.

---

## 259. Native Kubernetes Alerts

Do not duplicate every Prometheus/Kubernetes alert in Python. Use Python where it adds cross-source logic, custom reporting, or operational automation.

---

## 260. When Python Is Not Best

For standard Kubernetes metrics and alerting, kube-state-metrics plus Prometheus may be more scalable and conventional. Python is valuable for custom diagnostics, reports, workflow integration, and specialized rules.

---

## 261. kube-state-metrics

kube-state-metrics exposes Kubernetes object-state metrics for Prometheus. A Python monitor can complement it rather than duplicating all of its functionality.

---

## 262. Metrics Server vs Prometheus

Metrics Server serves resource metrics for Kubernetes APIs/autoscaling use cases, while Prometheus provides broader time-series monitoring. Choose the source according to the question being answered.

---

## 263. Production Data Model

Normalize pod observations into fields such as cluster, namespace, name, UID, owner kind/name, phase, readiness, restarts, resource usage, node, conditions, and health findings.

---

## 264. Observation Timestamp

Every observation should carry a timezone-aware UTC timestamp so state transitions and reports remain consistent across clusters.

---

## 265. Primary Finding

Choose a primary finding for concise alerts while retaining a list of all conditions in the detailed result.

---

## 266. Finding Evidence

Include evidence such as phase, container reason, exit code, restart delta, node condition, event reason, or metric value. Avoid generic messages like 'pod unhealthy'.

---

## 267. Remediation Hint

A result can include a safe diagnostic hint such as 'inspect previous container logs and deployment revision' without automatically executing a destructive action.

---

## 268. Runbook Links

Map condition codes to internal runbooks. Keep runbook URLs/configuration outside source code where possible.

---

## 269. Report Retention

Store historical reports only as long as operational and compliance requirements justify. Reports can contain sensitive cluster metadata.

---

## 270. Report Security

Restrict report access because namespace names, pod names, image references, node names, and workload information can reveal internal architecture.

---

## 271. No Secret Leakage

Never include Secret data, token values, environment variable secret values, or kubeconfig contents in logs or reports.

---

## 272. Image Reference Security

Image repositories and tags can reveal internal projects. Treat them as operational metadata and expose only to authorized users.

---

## 273. Audit Logging

Record monitor execution identity, cluster, namespace scope, configuration version, run ID, outcome, and alert actions without exposing secrets.

---

## 274. Configuration Version

Include a Git commit/configuration version in reports so operators can identify which monitoring rules produced a finding.

---

## 275. Production Deployment

Start with a staging cluster, validate RBAC and report accuracy, then deploy to production with restricted namespace scope before expanding coverage.

---

## 276. Canary Monitoring

Monitor a small set of namespaces first, compare findings with existing Prometheus/Kubernetes alerts, then expand after false-positive tuning.

---

## 277. False Positive Tuning

Use historical incident data to tune restart thresholds, Pending duration, readiness failure duration, and resource thresholds.

---

## 278. SLO-Based Alerting

Prefer workload availability/SLO impact over raw pod count when possible. One unhealthy replica in a 10-replica service may not justify a page if availability remains above SLO.

---

## 279. Severity by Impact

Severity should consider condition, duration, number of affected replicas, workload redundancy, environment, and business criticality.

---

## 280. Production vs Nonproduction

Use different alert policies for development, staging, and production. Avoid paging production-style incidents from ephemeral development workloads.

---

## 281. Environment Labels

Use environment labels in reports, logs, and bounded metrics so dashboards and alert routing can separate production from nonproduction.

---

## 282. Owner Routing

Use approved workload owner metadata to route alerts. Do not trust arbitrary pod labels as secure authorization data.

---

## 283. Namespace Ownership

Namespace owner/team metadata can improve routing and reporting, but missing ownership should be a governance finding rather than a runtime failure.

---

## 284. Governance Findings

Separate health from governance issues such as missing labels, missing resource requests, unrestricted images, or missing ownership metadata.

---

## 285. Policy Findings

A pod may be healthy but violate organizational policy. Keep policy compliance as a separate category so operational health remains accurate.

---

## 286. Security Findings

Examples include privileged containers, hostNetwork, hostPID, hostPath usage, missing security context, or mutable image tags. These should normally be security findings, not pod failures.

---

## 287. Mutable Tags

Using latest or other mutable tags makes rollouts less deterministic. Report mutable image tags as a governance/reliability finding where policy requires immutable images.

---

## 288. ImagePullSecrets

Missing imagePullSecrets can cause image pull failures. Report the referenced secret name but never expose its contents.

---

## 289. Private ECR

For private ECR images in EKS, image pull authorization is normally handled through node/pod identity and ECR integration rather than application-level credentials. Diagnose IAM and node/network configuration before adding secrets.

---

## 290. ServiceAccount Token

Do not copy or print ServiceAccount token contents. The Kubernetes client should use in-cluster authentication securely.

---

## 291. Token Rotation

Modern Kubernetes service account tokens can be projected and rotated. Avoid designs that assume a static token lifetime.

---

## 292. API Client Reuse

Reuse client instances within a scan rather than recreating them for every pod. This reduces overhead and simplifies connection configuration.

---

## 293. Connection Pooling

The Kubernetes Python client can reuse underlying HTTP connections depending on configuration. Avoid unnecessary client construction in tight loops.

---

## 294. Batch API Calls

Where possible, collect pod data in bulk with list operations instead of making one API call per pod.

---

## 295. Avoid N+1 Calls

Do not fetch events, nodes, and owners independently for every healthy pod. Cache or batch shared data and query deeper diagnostics only for candidates.

---

## 296. Candidate-Based Diagnostics

Use a two-stage model: inexpensive discovery for all pods, then detailed event/node/metrics analysis only for unhealthy or suspicious pods.

---

## 297. Cache Node State

When many pods share a node, cache node conditions for the scan to avoid repeated API calls.

---

## 298. Cache Owner Data

If multiple pods map to the same ReplicaSet/Deployment, cache owner lookups within a scan.

---

## 299. Cache Events

Collect relevant events in a bounded way rather than querying the entire event stream repeatedly for every pod.

---

## 300. Event Filtering

Filter events by namespace and object identity when possible to reduce noise and API load.

---

## 301. Scan Phases

A clean scan can use phases: discovery → normalization → candidate selection → enrichment → evaluation → state comparison → reporting/alerting.

---

## 302. Candidate Selection

Candidates can include Pending pods, unready pods, restarted containers, recent failures, resource threshold breaches, or pods on unhealthy nodes.

---

## 303. Normal Path Efficiency

Most healthy pods should pass through a low-cost path. Expensive diagnostics should be reserved for abnormal states.

---

## 304. Performance Measurement

Measure API time, normalization time, enrichment time, evaluation time, report time, and total scan duration.

---

## 305. Performance Bottleneck

The common bottleneck is API calls, not Python rule evaluation. Optimize API request count and concurrency before micro-optimizing Python logic.

---

## 306. Memory Optimization

Normalize only required fields and process lists incrementally. Avoid retaining full Kubernetes API objects after extracting needed information.

---

## 307. Thread Safety

If using concurrent workers, ensure shared state and caches are synchronized or designed for safe access. Prefer immutable observations and isolated results.

---

## 308. Async Design

Async Kubernetes clients or async wrappers can improve I/O concurrency, but operational simplicity and SDK behavior should be considered before adopting them.

---

## 309. Concurrency Choice

For moderate monitoring workloads, bounded threads are often easier to maintain. For very large fleets, event-driven or distributed architectures may be more appropriate.

---

## 310. Backpressure

Use bounded queues and worker counts to prevent event bursts or large clusters from overwhelming the monitor.

---

## 311. Graceful Shutdown

Handle SIGTERM so an EKS CronJob or deployment can stop cleanly, finish safe operations, and avoid leaving partial state or misleading success status.

---

## 312. Active Deadline

Set Kubernetes Job activeDeadlineSeconds so a stuck monitor does not run indefinitely.

---

## 313. Backoff Limit

Set a sensible backoffLimit for failed CronJob executions. Avoid repeated retries that create overlapping pressure.

---

## 314. Job History

Use successfulJobsHistoryLimit and failedJobsHistoryLimit to retain enough evidence without accumulating unlimited Job objects.

---

## 315. CronJob Schedule

Choose a schedule that meets detection requirements. A five-minute schedule is an example, not a universal recommendation.

---

## 316. Timezone

Prefer UTC scheduling where possible to avoid daylight-saving and regional timezone ambiguity, especially for multi-cluster operations.

---

## 317. Manual Run

Provide a manual CLI mode for incident investigation, but keep write/remediation commands separate and protected.

---

## 318. Production Debug Mode

Debug logging should be explicitly enabled and should still redact secrets. Do not permanently run production monitoring at verbose levels.

---

## 319. Failure Isolation

A failure to inspect one namespace should not necessarily prevent monitoring of all other namespaces if the monitoring contract permits partial results.

---

## 320. Partial Success

Return structured scan status with successful namespaces/clusters and failed scopes. Alert on monitoring coverage gaps.

---

## 321. Coverage Metric

Track monitored pod count versus expected pod count or configured scope. Coverage gaps can be more dangerous than individual pod failures.

---

## 322. Expected Scope

Define expected namespaces/clusters and alert if they disappear from the monitor's successful scan results.

---

## 323. Cluster Registration

For multi-cluster monitoring, maintain an approved inventory of clusters and expected namespaces. Do not discover arbitrary clusters automatically from credentials.

---

## 324. Cluster Availability

If an entire cluster cannot be contacted, raise a monitoring/cluster availability incident rather than reporting zero unhealthy pods.

---

## 325. Zero Pod Trap

Zero pods returned can mean a legitimate empty namespace, a selector mismatch, an API/RBAC problem, or a cluster issue. Validate expected scope before declaring healthy.

---

## 326. RBAC Coverage

Periodically test that the monitor can read the resources required by its contract. A role change can silently reduce monitoring coverage.

---

## 327. Permission Drift

Track RBAC changes through audit/GitOps and alert when required monitoring permissions disappear.

---

## 328. Kubernetes Version

Pin/test a compatible Kubernetes Python client version and validate against the Kubernetes/EKS versions used by the organization.

---

## 329. API Deprecations

Monitor Kubernetes API deprecation notices before upgrading clusters or client libraries. Avoid depending on deprecated APIs.

---

## 330. EKS Upgrade

After an EKS upgrade, test authentication, RBAC, pod APIs, events, metrics, workload ownership, and optional AWS integrations.

---

## 331. Compatibility Test

Run integration tests against supported Kubernetes/EKS versions to catch API schema or client compatibility changes.

---

## 332. Unit Test Strategy

Unit-test normalization, phase rules, container state parsing, restart-rate calculation, event correlation, severity precedence, configuration validation, and report generation.

---

## 333. Mock API Objects

Build small representative Kubernetes API objects instead of huge fixture dumps. Test only fields the code actually consumes.

---

## 334. Pagination Test

Mock multiple pod-list pages and verify all pods are discovered exactly once.

---

## 335. Waiting State Test

Test ImagePullBackOff, CrashLoopBackOff, CreateContainerConfigError, and generic waiting states with expected severity and evidence.

---

## 336. OOM Test

Mock OOMKilled termination and verify CRITICAL classification when policy requires it.

---

## 337. Restart Test

Provide previous and current restart counts/timestamps and verify restart-rate calculations and hysteresis.

---

## 338. Probe Test

Mock readiness/liveness/startup condition failures and verify the monitor distinguishes them correctly.

---

## 339. Scheduling Test

Mock PodScheduled=False and FailedScheduling events for insufficient resources, taints, and affinity failures.

---

## 340. Node Test

Mock Node NotReady/MemoryPressure/DiskPressure and verify affected pod correlation.

---

## 341. Metrics Test

Mock Metrics API values, missing metrics, stale metrics, and threshold breaches.

---

## 342. Alert State Test

Test new alert, repeated observation, severity escalation, recovery, and cooldown behavior.

---

## 343. RBAC Failure Test

Mock 403 Forbidden and verify that the monitor classifies it as authorization failure without endless retry.

---

## 344. API Failure Test

Mock timeout, 429, 500, and authentication errors and verify distinct error handling.

---

## 345. Integration Test

Use a dedicated test cluster or isolated namespace to create representative Deployments, Jobs, failing containers, ConfigMaps, and resource conditions.

---

## 346. Failure Injection

Controlled tests can intentionally create image pull failures, failing probes, bad configuration references, and resource pressure. Keep them isolated from production.

---

## 347. No Production Mutation

The monitor integration tests should never delete or restart production pods. Test destructive workflows only in isolated environments.

---

## 348. Security Testing

Scan dependencies, container images, Kubernetes manifests, and source code. Verify the monitor runs with only intended permissions.

---

## 349. Container Scan

Use Trivy in the existing DevSecOps pipeline to scan the monitor image and fail according to organizational severity policy.

---

## 350. SAST

Use SonarQube or the existing SAST process to detect code-quality and security issues before deployment.

---

## 351. Dependency Pinning

Pin direct dependencies and use a controlled update process. Regularly review the Kubernetes Python client and transitive dependencies.

---

## 352. SBOM

Generate an SBOM where organizational policy requires software supply-chain visibility.

---

## 353. Image Signing

If the platform uses signed images, sign the approved monitor image and enforce admission policy where appropriate.

---

## 354. Deployment Manifest

Kubernetes deployment configuration should define ServiceAccount, RBAC, CronJob, securityContext, resources, environment/config, image, schedule, and history controls.

---

## 355. Helm Integration

A Helm chart can package the monitor's CronJob, ServiceAccount, RBAC, ConfigMap, and optional ServiceMonitor resources.

---

## 356. Values Separation

Keep cluster-specific namespaces, schedules, thresholds, selectors, and alert destinations in Helm values rather than templates.

---

## 357. GitOps

Store Helm values/manifests in Git and deploy through ArgoCD if the cluster uses GitOps. Avoid manually changing production manifests outside the approved workflow.

---

## 358. ServiceMonitor

If the monitor exposes Prometheus metrics through an HTTP endpoint, a ServiceMonitor can allow Prometheus Operator-based discovery where installed.

---

## 359. Metrics Endpoint

A long-running deployment can expose /metrics. A short CronJob can instead push summary data to an approved metrics gateway only if organizational architecture supports it; avoid unnecessary push-based designs.

---

## 360. CronJob Metrics Caveat

Short-lived CronJobs can be difficult to scrape directly because they terminate quickly. Prefer durable monitoring of the monitor, report metrics, or a supported push/collector architecture.

---

## 361. ELK Shipping

Use the cluster's standard log collector to ship monitor stdout/stderr to ELK rather than embedding an Elasticsearch client unless there is a specific requirement.

---

## 362. Stdout Logging

Containerized applications should normally log structured JSON to stdout and let the platform log collector handle transport.

---

## 363. Log Correlation Fields

Include cluster, namespace, pod, UID, workload, condition, severity, run ID, and config version as structured fields where available.

---

## 364. Alert Payload

An alert should contain concise impact, affected workload, primary condition, evidence, first-seen time, current state, and runbook reference.

---

## 365. Alert Dedup Key

A stable key should remain constant across repeated scans for the same condition but change when a new pod UID or materially different incident appears.

---

## 366. Recovery Key

Recovery must use the same condition identity as the firing alert so downstream systems can close the correct incident.

---

## 367. Incident Duration

Track first-seen and last-seen timestamps to calculate condition duration.

---

## 368. Flapping Detection

Track frequent healthy/unhealthy transitions and classify them separately. Flapping may indicate probe instability, resource pressure, dependency instability, or application startup problems.

---

## 369. Flapping Remediation

Investigate root cause rather than simply increasing alert cooldown. Excessive suppression can hide a real outage.

---

## 370. Pod Availability

For replicated workloads, compute available replicas and unavailable replicas before assigning service-level severity.

---

## 371. Minimum Healthy Replicas

A workload can remain serviceable while one pod fails. Configure minimum healthy replica thresholds based on application SLO and redundancy.

---

## 372. Single Replica Risk

A one-replica critical service deserves higher severity for a pod failure because there is no redundancy.

---

## 373. Criticality Labels

Use approved application criticality metadata to adjust alert severity, but validate that the metadata is governed and not freely spoofable.

---

## 374. Maintenance Windows

Support approved maintenance windows to suppress or downgrade expected alerts without disabling monitoring entirely.

---

## 375. Suppression Safety

Maintenance suppression should have an expiration and audit trail. Permanent blanket suppression is dangerous.

---

## 376. Deployment Windows

During controlled deployments, temporarily different thresholds may be appropriate, but normal monitoring should resume automatically.

---

## 377. Alert Suppression vs Silence

Keep suppression state separate from health state. A pod can remain unhealthy even while notifications are temporarily suppressed.

---

## 378. Production Runbook

For a critical pod alert: confirm cluster/API health → identify workload owner → inspect phase/conditions → inspect container waiting/terminated state → inspect events → check recent rollout → inspect resources/metrics → check node/storage/network dependencies → consult logs → apply the least risky approved remediation.

---

## 379. Runbook: CrashLoop

Check current and previous logs, exit code, termination reason, configuration references, secrets/configmaps, probes, dependencies, resource limits, and recent deployment changes.

---

## 380. Runbook: Pending

Check PodScheduled condition, scheduler events, resource requests, node capacity, taints/tolerations, node affinity, topology constraints, quotas, PVCs, and autoscaler state.

---

## 381. Runbook: Image Pull

Check image name/tag/digest, registry availability, ECR permissions, imagePullSecrets, node network, and recent image changes.

---

## 382. Runbook: OOMKilled

Check memory usage, limit, request, restart pattern, application heap/runtime settings, traffic, recent code/config changes, and node pressure.

---

## 383. Runbook: Probe Failure

Check probe configuration, endpoint/command, port, timeout, startup duration, application health, and whether a recent release changed behavior.

---

## 384. Runbook: Evicted

Check node memory/disk pressure, pod QoS, ephemeral storage, node events, and workload recovery behavior.

---

## 385. Runbook: Volume Mount

Check PVC/PV status, CSI events, storage class, access mode, topology, cloud volume state, and node attachment limits.

---

## 386. Runbook: Service Unreachable

Check pod readiness, Service selectors, EndpointSlices, DNS, NetworkPolicy, ingress/controller state, ALB target health, and application endpoint behavior.

---

## 387. Runbook: Node NotReady

Identify affected pods, inspect node conditions/events, kubelet/runtime, networking, storage, and cloud infrastructure. Determine whether workload rescheduling is occurring.

---

## 388. Runbook: API Failure

Check cluster API availability, authentication, RBAC, network policy, DNS, client timeouts, rate limiting, and recent cluster changes.

---

## 389. Runbook: Alert Storm

Check duplicate monitor instances, CronJob overlap, missing state persistence, restart-rate thresholds, aggregation, and shared node/cluster incidents.

---

## 390. Runbook: Monitor Blind Spot

Check expected cluster/namespace scope, API permissions, discovery counts, collector failures, metrics availability, and monitor heartbeat.

---

## 391. Runbook: False Healthy

Investigate whether Running was incorrectly treated as healthy, metrics were missing, events were unavailable, or workload-level availability was ignored.

---

## 392. Runbook: False Critical

Check transient startup conditions, deployment rollouts, maintenance windows, restart-rate window, threshold configuration, and workload redundancy.

---

## 393. Production Anti-Patterns

Avoid cluster-wide permissions by default, treating Running as healthy, infinite API retries, one API call per pod, unbounded concurrency, high-cardinality Prometheus labels, automatic pod deletion, logging secrets, mutable production image tags, and duplicate alerting already covered by Prometheus.

---

## 394. Design Principle

Use Kubernetes-native mechanisms for Kubernetes state whenever possible and use Python for custom correlation, workflow, reporting, and automation that provides measurable operational value.

---

## 395. Interview: Why Python

Python provides a mature Kubernetes client, strong testing ecosystem, clear data modeling, and easy integration with AWS, CI/CD, observability, and operational tooling.

---

## 396. Interview: Kubernetes Client

The kubernetes Python client maps to Kubernetes APIs and supports in-cluster authentication, kubeconfig-based development, resource operations, and typed API methods.

---

## 397. Interview: Running vs Healthy

Running only means the pod has reached the Running phase. I also inspect container readiness, waiting/terminated reasons, restart behavior, probes, events, resource signals, and workload availability.

---

## 398. Interview: CrashLoop

I inspect previous logs, termination reason and exit code, events, configuration/secret references, probes, resources, dependencies, and recent deployment changes.

---

## 399. Interview: Pending

I check PodScheduled and FailedScheduling events, requests versus allocatable capacity, taints/tolerations, affinity, topology constraints, PVCs, quotas, and autoscaler behavior.

---

## 400. Interview: OOMKilled

I confirm the termination reason, compare memory usage with limit/request, inspect restart frequency and application behavior, and check whether node memory pressure is also present.

---

## 401. Interview: ImagePullBackOff

I check the exact image reference, registry/ECR access, imagePullSecrets, node network, IAM, and events for the precise pull failure.

---

## 402. Interview: Readiness vs Liveness

Readiness controls traffic eligibility; liveness can trigger container restarts. A readiness failure does not necessarily mean the process is dead.

---

## 403. Interview: Metrics Server

Metrics Server provides Kubernetes resource metrics for CPU and memory use cases. It is different from Prometheus, which provides broader time-series monitoring.

---

## 404. Interview: Prometheus

For standard Kubernetes metrics, kube-state-metrics and Prometheus may be preferable. Python should add custom correlation rather than duplicate every metric-based alert.

---

## 405. Interview: API Throttling

Use selectors, batching, bounded concurrency, SDK/client retries, exponential backoff, jitter, and reasonable polling intervals. Do not increase workers blindly.

---

## 406. Interview: RBAC

Use a dedicated ServiceAccount with the smallest Role/ClusterRole needed. Namespace-scoped monitoring should use namespace-scoped permissions when possible.

---

## 407. Interview: EKS Authentication

Inside EKS, use in-cluster Kubernetes authentication and workload identity for AWS APIs when needed. Avoid static kubeconfig or AWS keys in the container.

---

## 408. Interview: Multi-Cluster

Validate an approved cluster inventory, authenticate independently, isolate failures per cluster, and track coverage so an unreachable cluster is not reported as healthy.

---

## 409. Interview: Node Correlation

If multiple unrelated pods on one node fail, inspect node conditions and events because the node may be the shared root cause.

---

## 410. Interview: Application Health

Kubernetes pod health does not prove end-to-end availability. I correlate readiness with Service/EndpointSlice, ingress/ALB target health, and application-level checks when required.

---

## 411. Interview: Auto Remediation

I keep monitoring read-only and separate remediation. Any automated action requires allowlists, cluster/environment validation, cooldown, audit logging, and post-action verification.

---

## 412. Interview: Large Cluster

Use server-side selectors, incremental processing, bounded concurrency, candidate-based enrichment, caching shared node/owner data, and periodic reconciliation. Avoid N+1 API calls.

---

## 413. Interview: Event Watch

Watch APIs reduce polling latency but require reconnect and resourceVersion handling. I use periodic reconciliation as the source-of-truth safety net.

---

## 414. Interview: Alert Deduplication

Use stable keys based on cluster, namespace, pod UID, and condition. Track state transitions so repeated scans do not create duplicate incidents.

---

## 415. Interview: Workload Severity

Severity should consider workload type, replica redundancy, duration, environment, criticality, and actual service impact rather than simply counting unhealthy pods.

---

## 416. Interview: Testing

I unit-test pure health rules and mocked Kubernetes responses, then use isolated integration tests for real API behavior, RBAC, events, metrics, and representative failure scenarios.

---

## 417. Interview: EKS CronJob

I package the monitor as a non-root container, deploy it as a CronJob with workload identity and restricted RBAC, prevent overlap, set resource limits and deadlines, and ship structured logs to ELK.

---

## 418. Interview: GitOps

For ArgoCD-managed EKS environments, the monitor deployment is managed through Git. The monitor reports health and can correlate sync state but should not bypass GitOps by mutating desired state directly.

---

## 419. Interview: Monitoring the Monitor

I expose last-success timestamp, scan duration, API failures, collector failures, and coverage metrics so a stale or blind monitor is itself detected.

---

## 420. 60-Second Project Answer

I built a Python Kubernetes pod monitor for EKS that discovers workloads, evaluates pod and container states, correlates events, resources, probes, nodes, and workload controllers, and produces actionable health results. I designed it with least-privilege RBAC, in-cluster authentication, bounded API concurrency, candidate-based enrichment, restart and resource thresholds, alert deduplication, structured ELK logs, Prometheus/Grafana metrics, and EKS CronJob deployment. The monitor remains read-only, while remediation is handled separately through controlled DevOps workflows.

---

## 421. Final Workflow

Authenticate → validate cluster/scope → discover pods → normalize state → identify candidates → enrich with events/resources/nodes/workload owners → evaluate health → correlate shared incidents → compare previous state → alert on transitions → publish report/metrics/logs → return documented exit code.

---

## 422. Final Checklist: Authentication

[ ] Dedicated ServiceAccount
[ ] Least-privilege RBAC
[ ] In-cluster auth tested
[ ] Expected cluster validated
[ ] No static credentials
[ ] AWS workload identity only when required

---

## 423. Final Checklist: Detection

[ ] Pod phase
[ ] Container states
[ ] Restart rate
[ ] OOMKilled
[ ] Image pull failures
[ ] Probes
[ ] Scheduling conditions
[ ] Events
[ ] Node conditions
[ ] Resource metrics where required

---

## 424. Final Checklist: Correlation

[ ] Deployment/ReplicaSet
[ ] StatefulSet
[ ] DaemonSet
[ ] Job/CronJob
[ ] Node
[ ] PVC/storage
[ ] Service/EndpointSlice
[ ] ALB/Ingress when required

---

## 425. Final Checklist: Reliability

[ ] API timeouts
[ ] Retry classification
[ ] Backoff/jitter
[ ] Bounded concurrency
[ ] Pagination
[ ] Candidate-based enrichment
[ ] Failure isolation
[ ] Partial-result reporting
[ ] Monitor heartbeat

---

## 426. Final Checklist: Alerting

[ ] Stable alert keys
[ ] Deduplication
[ ] Cooldown/hysteresis
[ ] Recovery alerts
[ ] Aggregation
[ ] Workload/SLO-aware severity
[ ] Maintenance suppression with expiry
[ ] Notification failure monitoring

---

## 427. Final Checklist: Security

[ ] Non-root container
[ ] No privilege escalation
[ ] Restricted RBAC
[ ] No secret leakage
[ ] Network policy where appropriate
[ ] Dependency scanning
[ ] Trivy image scanning
[ ] SAST
[ ] Immutable image

---

## 428. Final Checklist: Deployment

[ ] Helm/GitOps
[ ] EKS CronJob
[ ] Workload identity
[ ] Resource limits
[ ] activeDeadlineSeconds
[ ] backoffLimit
[ ] concurrencyPolicy Forbid
[ ] Job history limits
[ ] Rollback tested

---

## 429. Final Production Principles

1. Running does not mean healthy.
2. Diagnose container state and events, not just pod phase.
3. Separate infrastructure, workload, governance, and monitoring failures.
4. Use least-privilege RBAC.
5. Avoid N+1 Kubernetes API calls.
6. Use bounded concurrency and retries.
7. Treat missing metrics as UNKNOWN.
8. Correlate pods with workloads and shared dependencies.
9. Prefer native Prometheus/Kubernetes monitoring where it already solves the problem.
10. Keep the core monitor read-only.
11. Prevent duplicate alerts and CronJob overlap.
12. Monitor the monitor itself.
13. Use GitOps for deployment changes.
14. Tune severity around actual workload availability and SLO impact.
15. Test real failure scenarios in an isolated cluster.

---

## Repository Progress

```text
11-Python-DevOps-Projects/
├── 01-AWS-Resource-Automation.md        ✓
├── 02-EC2-Health-Monitor.md             ✓
├── 03-S3-Backup-Automation.md           ✓
├── 04-EKS-Pod-Monitor.md                ✓
├── 05-Kubernetes-Cleanup-Automation.md
├── 06-CI-CD-Automation.md
├── 07-Infrastructure-Health-Checker.md
└── 08-End-to-End-DevOps-Automation.md
```

**Next file: `05-Kubernetes-Cleanup-Automation.md`**