# Infrastructure Health Checker

> Production-oriented Python infrastructure health checker for AWS, EKS, Kubernetes, networking, storage, CI/CD dependencies, and observability systems.

## Project Scope

```text
AWS / EKS
Kubernetes
EC2 / EBS / RDS / ECR
ALB / Ingress / DNS
Nodes / Pods / Deployments
Services / EndpointSlices
PVC / CSI / Storage
Prometheus / Grafana / ELK
Jenkins / ArgoCD / Artifactory
Dependency correlation
Health scoring
Production troubleshooting
```

## 1. Project Overview

Build a production-grade Python infrastructure health checker for AWS, EKS, Kubernetes, Linux hosts, networking, storage, CI/CD dependencies, and application infrastructure. The checker should collect health signals, classify risk, correlate related failures, produce actionable reports, expose metrics, and integrate with existing Prometheus/Grafana/ELK, Jenkins/GitHub Actions, and alerting workflows.

---

## 2. Real-World Problem

Infrastructure failures rarely appear as one isolated signal. A node may be NotReady because of disk pressure, an EKS workload may fail because a node is full, an application may be unhealthy because a load balancer target is failing, and a CI/CD pipeline may fail because ECR or an external dependency is unavailable. The checker should correlate signals instead of producing disconnected alerts.

---

## 3. Architecture

Scheduler/CronJob → Python collector → AWS/Kubernetes/Linux/API probes → normalization → threshold evaluation → dependency correlation → health score → structured report → Prometheus metrics → ELK logs → notification/incident integration.

---

## 4. Python Role

Python orchestrates health checks, API calls, normalization, correlation, reporting, retries, configuration, and remediation recommendations. It should not replace mature monitoring systems; it complements Prometheus/Grafana with targeted infrastructure validation and cross-system diagnostics.

---

## 5. Repository Structure

Recommended modules: cli.py, config.py, models.py, checks/, aws_client.py, kube_client.py, linux_client.py, network.py, thresholds.py, correlation.py, scoring.py, remediation.py, reporting.py, metrics.py, notifications.py, logging_config.py, state.py, and tests/.

---

## 6. Check Categories

Use categories such as AWS, EKS, Kubernetes, nodes, workloads, networking, storage, security, CI/CD dependencies, application endpoints, certificates, and external APIs.

---

## 7. Health States

Use HEALTHY, WARNING, CRITICAL, UNKNOWN, and SKIPPED. UNKNOWN should mean insufficient evidence rather than silently healthy.

---

## 8. Check Result Model

Each check should return check name, category, target, timestamp, status, severity, observed value, threshold, reason, evidence, duration, remediation, and error classification.

---

## 9. Run ID

Generate a unique run ID for each execution and include it in every structured result, log, metric-safe summary, and report.

---

## 10. Environment

Every run should identify environment, region, cluster, account, and scope where available. Never rely on an operator remembering which environment was checked.

---

## 11. Cluster Identity

For EKS checks, verify the expected cluster name/ARN and AWS account before collecting or acting on production resources.

---

## 12. AWS Identity

Use IAM roles and short-lived credentials. Never embed AWS access keys in the health-check image or source code.

---

## 13. EKS Authentication

Use in-cluster Kubernetes authentication for CronJob execution and AWS SDK identity only when AWS/EKS metadata is required.

---

## 14. Local Authentication

For local diagnostics, support explicit kubeconfig contexts and AWS profiles without silently using an unintended default context.

---

## 15. Scope Validation

Validate namespaces, node selectors, AWS region, cluster, and account scope before running checks.

---

## 16. Read-Only Principle

The default health checker should be read-only. Remediation must be a separate explicitly enabled capability with stronger controls.

---

## 17. Remediation Boundary

Prefer recommendations and runbooks over automatic fixes. If remediation is added, use a strict allowlist and explicit approval.

---

## 18. Configuration

Use typed configuration for thresholds, timeouts, namespaces, regions, protected resources, check enablement, concurrency, and notification settings.

---

## 19. Configuration Validation

Reject invalid thresholds, missing cluster identity, unsupported check names, malformed endpoints, and unsafe remediation settings before execution.

---

## 20. Safe Defaults

Default to read-only, conservative timeouts, bounded concurrency, no automatic remediation, and limited scope.

---

## 21. Threshold Model

Thresholds should be configurable per environment and workload class rather than hard-coded throughout the code.

---

## 22. Threshold Example

Example CPU warning/critical thresholds might be 70%/85%, but these values are illustrative and must be tuned from baseline behavior.

---

## 23. Baseline

A health checker should distinguish abnormal behavior from normal workload variation. Baselines can be static initially and evolve toward historical or SLO-aware thresholds.

---

## 24. Static vs Dynamic Thresholds

Static thresholds are easy to reason about; dynamic thresholds can reduce false positives but require reliable historical data and careful implementation.

---

## 25. Dependency Graph

Represent dependencies such as application → Service → EndpointSlice → Pod → Node → EKS cluster and CI/CD → registry → AWS identity → network.

---

## 26. Correlation

When several checks fail, identify a likely common dependency rather than reporting every downstream symptom as an independent root cause.

---

## 27. Root Cause Candidate

A node-level disk-pressure failure may explain multiple Pod failures. Correlation should mark the node condition as a probable upstream cause while retaining all evidence.

---

## 28. Blast Radius

Calculate affected namespaces, workloads, nodes, services, and environments where practical.

---

## 29. Health Score

A weighted health score can summarize state, but it must never hide individual CRITICAL checks.

---

## 30. Severity

Use severity based on impact and confidence, not merely threshold breach.

---

## 31. Confidence

A check can include confidence such as HIGH, MEDIUM, or LOW when evidence quality varies.

---

## 32. Unknown Handling

API timeout, missing permissions, or unavailable telemetry should become UNKNOWN/DEGRADED rather than HEALTHY.

---

## 33. Check Timeout

Every external call and overall check must have a bounded timeout.

---

## 34. Retry Policy

Retry transient failures with bounded exponential backoff and jitter. Do not retry authentication/authorization failures indefinitely.

---

## 35. Rate Limiting

Use bounded request rates so the checker does not become a source of API load.

---

## 36. Concurrency

Use a controlled worker pool or asyncio where appropriate. Start conservatively and tune from API behavior.

---

## 37. Thread Safety

Ensure shared clients, metrics, caches, and result collectors are safe under concurrency.

---

## 38. Circuit Breaker

If an external API repeatedly fails, temporarily stop additional calls and report the dependency as degraded.

---

## 39. Partial Results

A failure in one check must not prevent unrelated checks from running unless the dependency is a confirmed prerequisite.

---

## 40. Fail Fast Dependencies

If cluster authentication fails, skip checks that require the cluster API and clearly report why.

---

## 41. AWS STS Check

Verify AWS identity and account before performing account-sensitive checks.

---

## 42. AWS Region Check

Validate the configured region and ensure regional resources are queried from the correct region.

---

## 43. EC2 Instance Health

Check instance state, status checks, launch age where relevant, and optionally system reachability for approved instances.

---

## 44. EC2 Status Checks

Distinguish system status failures from instance status failures because remediation differs.

---

## 45. EC2 State

Stopped instances may be expected. Do not mark stopped as critical without an environment-specific policy.

---

## 46. EC2 CPU

CPU saturation should be interpreted with workload type and baseline context rather than a universal threshold.

---

## 47. EC2 Memory

AWS does not provide guest memory through basic EC2 instance state APIs. Use an approved agent/Prometheus/node exporter or application telemetry instead of assuming memory is available.

---

## 48. EC2 Disk

Disk usage requires guest-level telemetry. Do not infer filesystem utilization from EC2 instance metadata.

---

## 49. EC2 Network

Check relevant network reachability using application-specific probes rather than assuming instance state means network health.

---

## 50. EBS Volume

Check volume state, attachment, encryption, and selected performance signals where available.

---

## 51. EBS Encryption

Flag unencrypted volumes only when organizational security policy requires encryption.

---

## 52. EBS Attachment

An unexpected unattached volume may be a cleanup/cost candidate, but the health checker should report it rather than delete it.

---

## 53. RDS Health

Check DB instance status, availability, storage pressure indicators where available, and connectivity through approved application-level probes.

---

## 54. RDS Connectivity

A healthy RDS status does not prove the application can connect. Use a safe authenticated or TCP-level probe where policy permits.

---

## 55. RDS Storage

Storage pressure can cause application failures. Include it in dependency correlation.

---

## 56. S3 Health

Check expected bucket accessibility and optionally object read/write behavior using a controlled test object when required.

---

## 57. S3 Security

Do not expose bucket contents or credentials in health reports.

---

## 58. IAM Health

Health checks should validate expected role identity and selected permissions rather than attempting broad IAM enumeration.

---

## 59. Route53

Check expected DNS record resolution from the relevant network location when DNS is part of application availability.

---

## 60. NAT Gateway

Check NAT gateway state and relevant connectivity symptoms. Avoid declaring NAT healthy solely from resource state.

---

## 61. VPC

Check expected VPC/subnet/security group configuration through approved configuration checks.

---

## 62. Security Group

Configuration validation can verify required ports without opening or modifying rules.

---

## 63. ALB Health

Check load balancer state, listener configuration, target group health, and application endpoint response.

---

## 64. ALB Target Health

Unhealthy targets should be correlated with Pod readiness, node state, security groups, and application health.

---

## 65. ALB Listener

Verify expected listeners and ports exist for the application.

---

## 66. Ingress

For Kubernetes Ingress, verify resource state and controller-related conditions.

---

## 67. DNS to ALB

Verify DNS resolution and HTTP/TLS response from an appropriate network location.

---

## 68. TLS Certificate

Check certificate expiration and expected hostname coverage. Do not retrieve private keys.

---

## 69. Certificate Expiry

Use warning/critical windows appropriate to organizational rotation processes, for example 30/7 days, but tune to operational reality.

---

## 70. EKS Cluster

Check cluster status, Kubernetes version policy, endpoint reachability, and control-plane configuration required by the environment.

---

## 71. EKS Version

Flag versions approaching organizational support/EKS lifecycle limits using a maintained policy rather than hard-coded assumptions.

---

## 72. EKS Add-ons

Check required managed add-ons and their status/version compatibility.

---

## 73. EKS Node Groups

Check desired/current/ready capacity and health of managed node groups.

---

## 74. EKS Autoscaling

Check autoscaler/managed scaling signals where used and correlate with pending Pods.

---

## 75. EKS Capacity

Compare requested workload resources with node allocatable capacity and pending scheduling conditions.

---

## 76. EKS API

Measure API request success/latency where possible and detect repeated throttling.

---

## 77. EKS IAM

Verify workload/node identity configuration for selected critical workloads.

---

## 78. EKS OIDC

Where IAM Roles for Service Accounts or Pod Identity is used, verify expected identity configuration for critical workloads.

---

## 79. EKS Security

Check approved security configuration such as endpoint access, encryption, and logging configuration when these are part of platform policy.

---

## 80. Kubernetes API

Measure API availability, request latency, authentication, and authorization for required read operations.

---

## 81. Kubernetes Nodes

Check Ready condition, pressure conditions, allocatable resources, taints, and age where useful.

---

## 82. Node Ready

A NotReady node is a high-priority infrastructure signal. Capture condition reason and affected workloads.

---

## 83. Node Memory Pressure

MemoryPressure indicates node-level memory stress. Correlate with Pod requests/limits and eviction events.

---

## 84. Node Disk Pressure

DiskPressure can cause evictions and scheduling failures. Check filesystem/container runtime usage where telemetry is available.

---

## 85. Node PID Pressure

PIDPressure can indicate process exhaustion and may cause workload failures.

---

## 86. Node Conditions

Record all relevant conditions, not only Ready.

---

## 87. Node Taints

Unexpected taints can prevent scheduling. Compare with approved node-group configuration.

---

## 88. Node Labels

Unexpected label changes can affect affinity and workload placement.

---

## 89. Node Allocatable

Compare requested resources with allocatable capacity to detect scheduling pressure.

---

## 90. Pod Density

High pod count per node can cause networking or resource pressure. Use platform-specific limits and baseline.

---

## 91. Container Runtime

Check runtime health where node-level telemetry is available.

---

## 92. Kubelet

Kubelet failures can cause NotReady nodes and workload instability. Use node condition/events/log telemetry rather than direct invasive actions.

---

## 93. Container Restart

High restart counts indicate workload instability. Evaluate rate over time instead of total restart count alone.

---

## 94. CrashLoopBackOff

Detect CrashLoopBackOff and collect recent reason/log metadata without exposing secrets.

---

## 95. OOMKilled

Detect OOMKilled and correlate with container memory limits and node memory pressure.

---

## 96. ImagePullBackOff

Detect image pull failures and correlate with registry availability, image existence, and credentials.

---

## 97. ErrImagePull

Record the image reference and reason while avoiding credentials from error output.

---

## 98. Pending Pods

Inspect scheduler events, resource requests, taints, affinity, quotas, and PVC binding.

---

## 99. Failed Pods

Classify failures and correlate with owning workload and node.

---

## 100. Evicted Pods

Detect eviction reason such as memory/disk pressure and correlate with node conditions.

---

## 101. Terminating Pods

Long termination can indicate finalizers, volume unmount, or controller issues.

---

## 102. Readiness Probe

Readiness failures remove Pods from service endpoints. Track duration and affected replicas.

---

## 103. Liveness Probe

Liveness failures can cause restart loops. Correlate with application startup and dependency behavior.

---

## 104. Startup Probe

Startup probe failures can prevent slow-starting applications from becoming ready.

---

## 105. Deployment Health

Check desired, updated, available, and unavailable replicas plus rollout conditions.

---

## 106. ReplicaSet Health

Identify unexpected ReplicaSet states during deployments.

---

## 107. StatefulSet Health

Check ready replicas and update status while respecting stable identity semantics.

---

## 108. DaemonSet Health

Compare desired/current/ready/available node coverage.

---

## 109. Job Health

Check active, succeeded, failed, and stuck Jobs.

---

## 110. CronJob Health

Check suspended state, last schedule, active Jobs, and repeated failures.

---

## 111. Service Health

Validate Service existence, selector, port configuration, and EndpointSlice population.

---

## 112. EndpointSlice

An empty EndpointSlice can explain application unavailability. Correlate with Pod readiness and selectors.

---

## 113. Service Selector

A selector mismatch can create a Service with no endpoints even when Pods are running.

---

## 114. NetworkPolicy

Unexpected NetworkPolicy changes can block traffic. Health checking should detect configuration drift where policy snapshots are available.

---

## 115. DNS

Check CoreDNS health, service DNS resolution, and external DNS resolution where relevant.

---

## 116. CoreDNS

Check CoreDNS Pods, readiness, restarts, and error signals.

---

## 117. Ingress Controller

Check controller Pods, errors, configuration sync, and load balancer integration.

---

## 118. ALB Controller

For AWS Load Balancer Controller, correlate controller health with Ingress/TargetGroupBinding behavior.

---

## 119. Network Connectivity

Use bounded TCP/HTTP probes to approved endpoints. Do not perform broad port scanning.

---

## 120. HTTP Probe

Check status code, response time, and optionally expected response markers.

---

## 121. TCP Probe

A TCP connection verifies reachability but not application correctness.

---

## 122. DNS Probe

Validate resolution from the same network context as the workload when possible.

---

## 123. Latency

Track response latency and compare with application-specific SLOs.

---

## 124. HTTP Error Rate

A high 5xx rate is more useful than a single failed request. Prefer Prometheus/application metrics for sustained error-rate analysis.

---

## 125. Storage PVC

Check PVC phase, binding, storage class, and selected capacity signals.

---

## 126. PVC Pending

A Pending PVC can block StatefulSets and other workloads. Correlate with StorageClass and CSI health.

---

## 127. PV Health

Check phase and claim association for selected persistent volumes.

---

## 128. CSI Driver

Check CSI controller/node components and their health.

---

## 129. StorageClass

Verify expected StorageClass exists and is not unexpectedly changed.

---

## 130. EBS CSI

For EKS using EBS CSI, check controller/node Pods and relevant workload mount failures.

---

## 131. Volume Mount Failure

Correlate Pod events, PVC/PV state, CSI health, and node conditions.

---

## 132. ResourceQuota

Quota exhaustion can cause scheduling/deployment failures. Include quota usage where relevant.

---

## 133. LimitRange

Unexpected LimitRange changes can affect default requests/limits.

---

## 134. HPA

Check HPA desired/current replicas, target metrics, conditions, and inability to scale.

---

## 135. HPA Metrics

Missing metrics can prevent expected autoscaling. Correlate Metrics Server/Prometheus Adapter health.

---

## 136. Metrics Server

Check Metrics Server availability if HPA or resource metrics depend on it.

---

## 137. VPA

If VPA is used, inspect recommendation/update behavior without assuming it is universally enabled.

---

## 138. PDB

A restrictive PodDisruptionBudget can block maintenance or rollout actions. Report it as a dependency signal.

---

## 139. Scheduler

Check scheduler health and correlate Pending Pods with scheduling events.

---

## 140. Controller Manager

Check controller-manager health through platform-supported telemetry rather than assuming all controllers are healthy.

---

## 141. Events

Collect relevant recent Kubernetes Events for failing resources, with bounded lookback and rate limits.

---

## 142. Event Deduplication

Deduplicate repeated events so reports emphasize meaningful transitions rather than thousands of identical messages.

---

## 143. Event Retention

Do not assume Kubernetes Events provide long-term history; centralize events if historical investigation is required.

---

## 144. Application Dependency

Map workloads to external dependencies such as databases, queues, APIs, and registries where configuration allows.

---

## 145. RabbitMQ

For workloads using RabbitMQ, check approved health endpoints/metrics and correlate connection failures with application errors.

---

## 146. External API

Check critical external APIs with low-rate synthetic requests and strict timeouts.

---

## 147. Dependency Timeout

A slow dependency can appear as application CPU/memory pressure. Correlation should include latency and timeout evidence.

---

## 148. Dependency DNS

If an application cannot resolve a dependency, check DNS from the relevant network context.

---

## 149. Dependency TLS

Validate certificate chain and expiry for critical external endpoints.

---

## 150. Dependency Authentication

Use dedicated low-privilege test credentials or non-sensitive health endpoints.

---

## 151. Database Health

Use database-native health metrics or controlled connectivity probes. Do not run expensive queries as routine health checks.

---

## 152. Database Connection Pool

Application connection pool exhaustion can appear as latency/errors despite a healthy database. Correlate application metrics.

---

## 153. Queue Health

Queue depth, consumer lag, and consumer health can indicate asynchronous processing problems.

---

## 154. RabbitMQ Queue Depth

Use queue metrics to distinguish broker availability from consumer processing backlog.

---

## 155. CI/CD Dependency

Health check Jenkins, GitHub Actions, GitLab where applicable, artifact repository, container registry, and ArgoCD when they are critical deployment dependencies.

---

## 156. Jenkins Health

Check Jenkins availability and selected controller/agent signals through API.

---

## 157. GitHub Health

Use official status/API signals rather than aggressive polling.

---

## 158. Artifactory Health

Check repository API availability and authentication using a safe read operation.

---

## 159. ECR Health

Verify expected repository accessibility and image lookup permissions.

---

## 160. ArgoCD Health

Check server/API availability and selected critical application status.

---

## 161. ArgoCD Application

Validate Sync/Health status and target cluster for critical applications.

---

## 162. GitOps Drift

Detect unexpected drift through ArgoCD status and surface it as a configuration signal.

---

## 163. Prometheus Health

Check Prometheus availability, query API responsiveness, and scrape health for critical targets.

---

## 164. Grafana Health

Check dashboard/API availability where Grafana is part of operational workflow.

---

## 165. ELK Health

Check Elasticsearch cluster health and ingestion path where required.

---

## 166. Elasticsearch

Check cluster status, node availability, shard health, disk pressure, and indexing errors according to architecture.

---

## 167. Logstash

Check pipeline health and persistent queue/error signals if Logstash is used.

---

## 168. Kibana

Check availability and saved dashboard access where operational teams depend on it.

---

## 169. Observability Dependency

If monitoring is unavailable, distinguish application health from observability health. Do not declare an application healthy solely because metrics are missing.

---

## 170. Prometheus Query Safety

Use bounded, pre-approved PromQL queries. Never accept arbitrary user-supplied PromQL in a privileged health-check process.

---

## 171. Metrics Cardinality

Do not emit resource name/UID as Prometheus labels. Keep detailed identities in logs.

---

## 172. Health Metrics

Expose checker_runs_total, checks_total, checks_failed_total, checks_unknown_total, check_duration_seconds, and last_success_timestamp.

---

## 173. Metric Labels

Use bounded labels such as environment, category, check, outcome, and severity.

---

## 174. Dashboard

Create a dashboard showing overall status, critical checks, node health, workload health, AWS dependencies, CI/CD dependencies, and check duration.

---

## 175. ELK Logging

Send structured JSON with run ID, check name, target, status, severity, observed value, threshold, reason, and remediation.

---

## 176. No Secret Logging

Never log tokens, passwords, Kubernetes Secret data, Authorization headers, or sensitive cloud credentials.

---

## 177. Report

Generate Markdown/JSON reports containing summary, critical findings, warnings, unknown checks, evidence, likely root causes, and recommended actions.

---

## 178. Machine-Readable Report

Keep a stable JSON schema so incident/ticketing systems can consume health results.

---

## 179. Human Report

Make the human report action-oriented: what failed, impact, likely cause, evidence, and next step.

---

## 180. Incident Integration

Create or update an incident/ticket only for policy-defined conditions. Avoid opening duplicate incidents for every individual symptom.

---

## 181. Alert Deduplication

Use a stable incident key based on environment, dependency, and failure class rather than resource UID alone.

---

## 182. Recovery Detection

Emit recovery events when a previously critical dependency returns healthy.

---

## 183. Alert Hysteresis

Require sustained failure or multiple observations before alerting on noisy checks.

---

## 184. Flapping Detection

Track state changes and suppress repeated alert/recovery cycles when appropriate.

---

## 185. Maintenance Windows

Allow approved maintenance windows to suppress selected alerts while still recording health results.

---

## 186. Silence Safety

Silencing should never hide security-critical findings unless explicitly governed.

---

## 187. Health Check Scheduling

Run frequent lightweight checks and less frequent expensive checks according to operational importance.

---

## 188. CronJob Deployment

Deploy the checker as an EKS CronJob with concurrencyPolicy: Forbid, activeDeadlineSeconds, backoffLimit, history limits, resource requests/limits, and dedicated ServiceAccount.

---

## 189. CronJob Frequency

Do not run expensive cluster-wide checks every minute without evidence that the API and cluster can support the load.

---

## 190. Overlap Protection

Prevent overlapping executions using CronJob concurrencyPolicy and, for multi-worker architectures, a coordination mechanism.

---

## 191. Container Security

Run as non-root, drop capabilities, use read-only filesystem where possible, and minimize the runtime image.

---

## 192. NetworkPolicy

Permit only required API/observability/notification destinations.

---

## 193. RBAC

Use read-only Kubernetes permissions for health checks. Grant access only to resource types actually checked.

---

## 194. RBAC Namespace Scope

Prefer namespace-scoped Roles when cluster-wide node/resource checks are unnecessary. Cluster-wide read permissions should be justified.

---

## 195. AWS IAM

Grant only required Describe/Get/List permissions. Avoid AdministratorAccess.

---

## 196. Workload Identity

Use EKS Pod Identity or IRSA according to platform standards for AWS API access.

---

## 197. CI Integration

Run unit/lint/security tests in Jenkins or GitHub Actions before publishing the health-check image.

---

## 198. DevSecOps Pipeline

Checkout → lint/type-check → pytest → SonarQube → dependency scan → Trivy → build → image scan → publish → deploy to staging → integration test → GitOps production deployment.

---

## 199. ArgoCD Deployment

Manage CronJob, ServiceAccount, RBAC, ConfigMap, NetworkPolicy, and deployment values through GitOps.

---

## 200. Helm

Use Helm values for environment-specific thresholds, schedules, namespaces, endpoints, and alert settings.

---

## 201. Image

Use an approved Python base image and pin production image by digest where possible.

---

## 202. Image Scanning

Run Trivy against the final image and block unacceptable vulnerabilities according to policy.

---

## 203. SBOM

Generate an SBOM when required and associate it with the exact image digest.

---

## 204. Supply Chain

Use signed images/provenance where organizational policy supports them.

---

## 205. Testing Strategy

Keep check functions small and deterministic. Mock AWS/Kubernetes/HTTP clients for unit tests and use isolated integration environments for real API behavior.

---

## 206. Unit Tests

Test healthy, warning, critical, unknown, timeout, unauthorized, rate-limited, and malformed-response scenarios.

---

## 207. Threshold Tests

Test exact threshold boundaries and environment-specific threshold overrides.

---

## 208. Correlation Tests

Create synthetic node/POD/deployment/service failure combinations and verify root-cause ranking.

---

## 209. RBAC Tests

Verify the checker works with only intended read permissions and fails clearly when a required permission is missing.

---

## 210. API Error Tests

Mock 401, 403, 404, 409, 429, 5xx, timeout, DNS failure, and TLS failure.

---

## 211. Kubernetes Tests

Test NotReady nodes, pressure conditions, Pending Pods, CrashLoopBackOff, OOMKilled, image pull failures, readiness failures, deployment rollouts, and PVC problems.

---

## 212. AWS Tests

Mock EC2, EKS, EBS, RDS, ECR, Route53, and STS responses relevant to the configured checks.

---

## 213. Network Tests

Test DNS failure, TCP timeout, HTTP 4xx/5xx, slow response, and TLS expiry conditions.

---

## 214. Prometheus Tests

Test query timeout, empty result, invalid result shape, and expected metric values.

---

## 215. Report Tests

Verify JSON schema, Markdown formatting, stable fields, and secret redaction.

---

## 216. Metric Tests

Verify bounded metric labels and correct counters/gauges/histograms.

---

## 217. Concurrency Tests

Run concurrent checks and verify result integrity, bounded worker count, and no race on shared state.

---

## 218. Performance Tests

Measure API request count, memory, runtime, and concurrency against realistic cluster sizes.

---

## 219. Large Cluster Test

Test with thousands of Pods/nodes or synthetic data to ensure results are processed incrementally.

---

## 220. Graceful Shutdown

Handle SIGTERM and stop safely, preserving partial results and avoiding false HEALTHY status.

---

## 221. Exit Codes

Define stable exit codes for healthy, warning, critical, configuration failure, authentication failure, and infrastructure/API failure.

---

## 222. Production Troubleshooting: Checker Down

Inspect CronJob schedule, Job history, image pull, RBAC, ServiceAccount, resource limits, and container logs.

---

## 223. Production Troubleshooting: Wrong Cluster

Stop any remediation, verify EKS cluster/account/region, inspect configuration, and correct environment mapping.

---

## 224. Production Troubleshooting: False Critical

Review threshold, baseline, duration, dependency correlation, and evidence before changing the threshold.

---

## 225. Production Troubleshooting: False Healthy

Check whether missing telemetry was incorrectly treated as healthy. Unknown must not silently become healthy.

---

## 226. Production Troubleshooting: Too Many API Calls

Inspect list/watch frequency, per-resource enrichment, concurrency, duplicate checks, and polling intervals.

---

## 227. Production Troubleshooting: API 429

Reduce concurrency and request frequency, add backoff/jitter, and remove redundant API queries.

---

## 228. Production Troubleshooting: RBAC 403

Identify exact resource/verb/namespace required and update least-privilege RBAC through GitOps.

---

## 229. Production Troubleshooting: API Timeout

Check control-plane health, network policy, DNS, client timeout, and whether the checker is overloading the API.

---

## 230. Production Troubleshooting: Node NotReady

Inspect node conditions, kubelet/runtime signals, networking, disk/memory/PID pressure, and affected workloads.

---

## 231. Production Troubleshooting: DiskPressure

Check node filesystem/container runtime usage, evicted Pods, image layers, logs, and storage telemetry.

---

## 232. Production Troubleshooting: MemoryPressure

Check node memory utilization, Pod requests/limits, OOMKills, eviction events, and workload concentration.

---

## 233. Production Troubleshooting: Pending Pods

Inspect scheduler events, resource requests, taints, affinity, quotas, PVCs, and node capacity.

---

## 234. Production Troubleshooting: CrashLoop

Inspect previous container logs, exit code, OOMKilled, probes, configuration, dependency connectivity, and recent deployment.

---

## 235. Production Troubleshooting: ImagePullBackOff

Check image reference/digest, ECR/registry access, credentials, network, image existence, and node pull events.

---

## 236. Production Troubleshooting: Readiness Failure

Check probe endpoint, port, timeout, application startup, dependencies, and recent code/config changes.

---

## 237. Production Troubleshooting: OOMKilled

Compare memory usage with limits, traffic, recent release, runtime behavior, and node pressure.

---

## 238. Production Troubleshooting: PVC Pending

Check StorageClass, CSI driver, provisioner events, quotas, availability zone constraints, and requested storage.

---

## 239. Production Troubleshooting: Service No Endpoints

Compare Service selector with Pod labels and readiness; inspect EndpointSlices.

---

## 240. Production Troubleshooting: ALB Targets Unhealthy

Check target health reason, Pod readiness, NodePort/service path, security groups, ingress configuration, and application response.

---

## 241. Production Troubleshooting: DNS Failure

Check CoreDNS Pods, service endpoints, upstream resolution, NetworkPolicy, and VPC DNS configuration.

---

## 242. Production Troubleshooting: External API Failure

Verify DNS, TCP/TLS, HTTP response, authentication, provider status, and whether the failure is isolated or regional.

---

## 243. Production Troubleshooting: RDS Unreachable

Check DB status, security groups, network path, DNS, credentials/secret rotation, connection limits, and application connection pool.

---

## 244. Production Troubleshooting: RabbitMQ Backlog

Check queue depth, consumer health, broker resources, network connectivity, and application processing rate.

---

## 245. Production Troubleshooting: ArgoCD Degraded

Check application health, sync revision, Pods, Services, Ingress, events, and recent GitOps commit.

---

## 246. Production Troubleshooting: Prometheus Down

Check Prometheus Pods, PVC, resource pressure, query/API availability, and scrape configuration.

---

## 247. Production Troubleshooting: ELK Degraded

Check Elasticsearch cluster health, disk pressure, Logstash pipelines, ingestion errors, and Kibana availability.

---

## 248. Production Troubleshooting: Certificate Expiring

Verify certificate issuer/renewal controller, DNS validation, secret reference, and renewal events. Do not manually replace certificates without following the platform process.

---

## 249. Production Troubleshooting: Monitoring Missing

Distinguish actual service failure from missing observability. Use independent probes when possible.

---

## 250. Production Troubleshooting: Flapping

Increase evaluation window/debounce, inspect intermittent dependency failures, and avoid hiding persistent critical problems.

---

## 251. Production Troubleshooting: High Runtime

Break down check durations, API latency, number of objects, external calls, and serialization/reporting time.

---

## 252. Production Troubleshooting: High Memory

Check whether the checker stores entire cluster object sets, large reports, logs, or unbounded result queues.

---

## 253. Production Troubleshooting: Duplicate Alerts

Inspect incident keys, state persistence, alert deduplication, overlapping jobs, and multiple checker instances.

---

## 254. Production Troubleshooting: Missing Recovery

Check whether state is persisted and whether the checker can distinguish a recovery from a first healthy run.

---

## 255. Production Troubleshooting: Configuration Drift

Compare deployed ConfigMap/Helm values with Git and verify ArgoCD sync state.

---

## 256. Production Troubleshooting: Permission Drift

Compare ServiceAccount/RBAC with GitOps configuration and recent changes.

---

## 257. Production Troubleshooting: Image Drift

Verify running image digest against approved GitOps/image registry metadata.

---

## 258. Production Troubleshooting: Wrong Threshold

Compare environment policy version and recent Git changes; do not tune production thresholds directly on the live container.

---

## 259. Production Troubleshooting: Notification Failure

Record health results independently and repair notification integration without suppressing critical health states.

---

## 260. Production Troubleshooting: Remediation Accident

If automatic remediation exists and causes impact, disable the remediation path, preserve audit data, assess affected resources, and revert through approved procedures.

---

## 261. Interview: Explain Project

I built a Python infrastructure health checker that collects AWS, EKS, Kubernetes, networking, storage, workload, CI/CD, and observability signals, normalizes them into structured results, correlates related failures, and produces actionable health reports. It is read-only by default, uses least-privilege access, bounded retries/concurrency, Prometheus/Grafana/ELK integration, and production-safe troubleshooting workflows.

---

## 262. Interview: Why Not Prometheus Only

Prometheus is excellent for metrics and alerting, but a Python health checker can orchestrate API-level configuration checks, cross-system dependency validation, AWS/Kubernetes correlation, and custom operational reports.

---

## 263. Interview: Why Correlation

Independent alerts can hide the common cause. For example, one NotReady node can produce Pending/Evicted/Unhealthy Pods and ALB target failures. Correlation helps operators prioritize the upstream signal.

---

## 264. Interview: Unknown State

If telemetry is unavailable, I return UNKNOWN rather than HEALTHY. Otherwise an observability outage could create a false sense of infrastructure health.

---

## 265. Interview: Read-Only

The base health checker should not mutate infrastructure. That reduces blast radius and allows it to run with read-only permissions.

---

## 266. Interview: Thresholds

Thresholds are configuration, not hard-coded business logic. I tune them using workload baselines and environment-specific requirements.

---

## 267. Interview: API Throttling

I reduce request volume with selectors/caching, bound concurrency, and use exponential backoff with jitter for 429 responses.

---

## 268. Interview: EKS Health

I check cluster/API availability, node readiness/pressure, workload states, scheduling, storage, networking, add-ons, and selected AWS dependencies.

---

## 269. Interview: Node Failure

I first identify the node condition and affected workloads, then correlate memory/disk/PID pressure, kubelet/runtime, scheduling, and infrastructure signals.

---

## 270. Interview: Pod Troubleshooting

I inspect Pod status, events, previous logs, probes, resources, image pull state, owner workload, node conditions, and service endpoints.

---

## 271. Interview: ALB Troubleshooting

I correlate ALB target health with Ingress, Service, EndpointSlices, Pod readiness, NodePort path, security groups, and application response.

---

## 272. Interview: Production Security

I use dedicated ServiceAccounts, read-only RBAC, IAM workload identity, non-root containers, restricted egress, secret redaction, and no automatic remediation by default.

---

## 273. Interview: Metrics

I expose bounded metrics such as check status, duration, and category. Detailed resource identities stay in ELK logs to avoid high cardinality.

---

## 274. Interview: ELK

Structured logs include run ID, check, target, status, severity, evidence, and remediation while excluding secrets.

---

## 275. Interview: Automation Deployment

I deploy it as an EKS CronJob through Helm and ArgoCD with concurrencyPolicy Forbid, active deadlines, resource limits, dedicated identity, and controlled configuration.

---

## 276. Interview: Testing

I unit-test check logic and error classification with mocks, then use isolated integration tests against Kubernetes/AWS environments. Production remains read-only and protected.

---

## 277. Interview: Automatic Remediation

I prefer recommendation over automatic remediation because health detection and safe remediation are different problems. If remediation is added, it needs a separate allowlist, approval, audit, and rollback design.

---

## 278. Interview: Health Score

A health score is useful as a summary but cannot replace individual findings. A critical dependency must remain visible even if the aggregate score looks acceptable.

---

## 279. Interview: DORA/Operations

The checker is not a DORA system, but its deployment and incident data can contribute to operational metrics and help correlate infrastructure health with release events.

---

## 280. Interview: 60-Second Answer

I developed a Python infrastructure health checker for AWS/EKS that validates cluster, nodes, workloads, storage, networking, ALB, databases, queues, CI/CD dependencies, and observability components. It uses structured checks with bounded retries and timeouts, correlates upstream/downstream failures, reports HEALTHY/WARNING/CRITICAL/UNKNOWN states, exports Prometheus metrics, sends structured logs to ELK, and is deployed as a read-only EKS CronJob through Helm and ArgoCD. The main focus is actionable diagnosis rather than simply generating more alerts.

---

## 281. Final Workflow

Validate configuration and identity → run independent checks → normalize results → correlate dependencies → classify severity/confidence → calculate summary → generate JSON/Markdown report → expose metrics → send structured logs/notifications → persist state for recovery detection → exit with documented status.

---

## 282. Final Checklist: Identity

[ ] Correct AWS account
[ ] Correct region
[ ] Correct EKS cluster
[ ] Correct namespace scope
[ ] Dedicated ServiceAccount
[ ] Read-only permissions

---

## 283. Final Checklist: Kubernetes

[ ] API health
[ ] Node Ready
[ ] Pressure conditions
[ ] Pending Pods
[ ] CrashLoop/OOM
[ ] ImagePull failures
[ ] Probe failures
[ ] Deployments
[ ] StatefulSets
[ ] DaemonSets
[ ] Services/EndpointSlices
[ ] PVC/CSI

---

## 284. Final Checklist: AWS

[ ] STS identity
[ ] EKS status
[ ] Node groups
[ ] EBS
[ ] RDS
[ ] ECR
[ ] ALB
[ ] DNS
[ ] Required VPC/network signals

---

## 285. Final Checklist: Networking

[ ] DNS
[ ] TCP/HTTP probes
[ ] TLS expiry
[ ] ALB targets
[ ] Ingress
[ ] NetworkPolicy
[ ] External dependency connectivity

---

## 286. Final Checklist: Observability

[ ] Prometheus
[ ] Grafana
[ ] ELK
[ ] Structured logs
[ ] Bounded metrics
[ ] Alert deduplication
[ ] Recovery detection

---

## 287. Final Checklist: Security

[ ] Read-only RBAC
[ ] Least-privilege IAM
[ ] Workload identity
[ ] Non-root image
[ ] Restricted egress
[ ] Secret redaction
[ ] Image scanning
[ ] Dependency scanning

---

## 288. Final Checklist: Reliability

[ ] Timeouts
[ ] Retries
[ ] Backoff/jitter
[ ] Rate limits
[ ] Concurrency limits
[ ] Circuit breaker
[ ] Graceful shutdown
[ ] Partial-result handling

---

## 289. Final Checklist: Production

[ ] GitOps deployment
[ ] CronJob overlap protection
[ ] Resource limits
[ ] Health report
[ ] Run ID
[ ] Incident integration
[ ] Recovery alerts
[ ] Tested failure scenarios

---

## 290. Final Production Principles

1. Health checking must produce evidence, not just green/red output.
2. Unknown telemetry is not healthy telemetry.
3. Correlate failures to identify likely upstream causes.
4. Keep the base checker read-only.
5. Use least-privilege AWS and Kubernetes identities.
6. Bound every API call, retry, concurrency level, and execution time.
7. Avoid high-cardinality metrics.
8. Keep detailed evidence in structured logs and reports.
9. Prefer native monitoring for continuous metrics and Python for orchestration/correlation.
10. Deploy the checker itself through GitOps and secure CI/CD.
11. Test failure scenarios before trusting the checker in production.
12. Never let the health checker become another source of operational instability.

---

## Repository Progress

```text
11-Python-DevOps-Projects/
├── 01-AWS-Resource-Automation.md        ✓
├── 02-EC2-Health-Monitor.md             ✓
├── 03-S3-Backup-Automation.md           ✓
├── 04-EKS-Pod-Monitor.md                ✓
├── 05-Kubernetes-Cleanup-Automation.md  ✓
├── 06-CI-CD-Automation.md               ✓
├── 07-Infrastructure-Health-Checker.md  ✓
└── 08-End-to-End-DevOps-Automation.md
```

**Next file: `08-End-to-End-DevOps-Automation.md`**