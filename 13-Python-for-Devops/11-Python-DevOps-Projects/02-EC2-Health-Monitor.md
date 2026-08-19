# 11-Python-DevOps-Projects
# 02 — EC2 Health Monitor

> Production-oriented Python project for EC2 infrastructure health monitoring, observability, alerting, and controlled DevOps operations.

## Project Scope

```text
EC2 discovery
AWS status checks
CloudWatch metrics
scheduled events
health evaluation
alert state management
multi-region / multi-account
Prometheus + Grafana + ELK
CI/CD + EKS deployment
production troubleshooting
```

## 1. Project Overview

Build a production-oriented Python EC2 health monitor using boto3. The tool discovers instances, evaluates AWS status checks, optionally reads CloudWatch/SSM signals, classifies health, produces reports, emits metrics, and integrates with CI/CD or EKS.

---

## 2. Real-World Problem

DevOps teams need to identify failed EC2 status checks, high CPU, disk or memory pressure, scheduled maintenance, missing monitoring coverage, and monitoring-system failures without manually checking every instance.

---

## 3. Health Layers

Separate infrastructure health, guest-OS health, and application health. EC2 APIs provide instance state and AWS status checks; guest metrics require CloudWatch Agent/custom metrics/SSM; application health may require ALB target health, service checks, or application metrics.

---

## 4. Architecture

Scheduler/CI/CronJob → Python monitor → configuration and STS identity → account/region guard → EC2 discovery → status/CloudWatch/event collection → health evaluator → state/alerting → JSON/CSV/S3 + Prometheus/ELK.

---

## 5. Technology Stack

Python 3, boto3/botocore, argparse, dataclasses, logging, pytest, unittest.mock, CloudWatch, Systems Manager, Prometheus, Grafana, ELK, Docker, Jenkins, GitHub Actions, Kubernetes/EKS.

---

## 6. Project Structure

Recommended modules: cli.py, config.py, identity.py, discovery.py, status.py, metrics.py, events.py, health.py, alerts.py, reporters.py, logging_config.py, models.py, plus unit and integration tests.

---

## 7. Configuration

Keep regions, environment, resource filters, thresholds, evaluation windows, worker limits, alert policy, and output destinations outside application logic.

---

## 8. Threshold Validation

Validate that thresholds are numeric, within valid ranges, and that warning thresholds are below critical thresholds. Reject invalid configuration before making AWS calls.

---

## 9. AWS Credentials

Prefer IAM roles, EKS workload identity, GitHub Actions OIDC, or controlled Jenkins role assumption. Local development can use an AWS profile. Never hard-code long-lived access keys.

---

## 10. STS Identity

Call sts:GetCallerIdentity before monitoring and record account ID and role ARN. For sensitive environments, compare the actual account with an explicitly configured expected account and fail closed on mismatch.

---

## 11. Least Privilege

A basic monitor should normally need EC2 describe/status permissions. CloudWatch permissions are added only when metrics are required; SSM permissions should be a separate optional capability.

---

## 12. EC2 Client

Create a boto3 EC2 client from a configured session. Centralize client creation so timeouts, retry behavior, region, and test injection are consistent.

---

## 13. Pagination

Use describe_instances paginator or continuation tokens. Never assume one API response contains the entire fleet.

---

## 14. Server-Side Filtering

Filter by instance state and supported tags at the AWS API where possible. This reduces payload size, memory use, and processing time.

---

## 15. Normalized Model

Normalize each instance into a model containing account, region, instance ID, type, state, IP metadata where needed, tags, status checks, and health results.

---

## 16. Instance State

Treat pending, running, stopping, stopped, shutting-down, and terminated distinctly. Normally monitor running instances and report stopped resources separately unless policy says otherwise.

---

## 17. Tag Scope

A tag such as Monitoring=enabled can define monitoring scope. Missing monitoring tags are governance findings, not necessarily infrastructure failures.

---

## 18. AWS Status Checks

DescribeInstanceStatus exposes instance and system status checks. A system impairment generally points toward AWS infrastructure; an instance impairment indicates a problem associated with the instance.

---

## 19. Status Parsing

Normalize status values such as ok, impaired, initializing, and not-applicable. Missing status data should not silently become healthy.

---

## 20. Status Severity

A failed required status check can be CRITICAL. Initializing may be WARNING or UNKNOWN depending on age and policy. Document the exact state machine.

---

## 21. Scheduled Events

Inspect EC2 status information for scheduled events and surface maintenance or retirement events. Severity should consider event timing and workload redundancy.

---

## 22. CPU Metrics

CPUUtilization is normally available through CloudWatch under AWS/EC2 with an InstanceId dimension. Use an evaluation window rather than reacting to one isolated sample.

---

## 23. CPU Thresholds

Example policy: 80% warning and 95% critical, evaluated across multiple periods. Thresholds are workload-specific and should be configurable.

---

## 24. CPU Context

High CPU does not automatically mean unhealthy; batch processing, deployments, load tests, and compute-heavy workloads may intentionally use high CPU.

---

## 25. Memory Metrics

Basic EC2 APIs do not directly expose Linux guest memory utilization. Use CloudWatch Agent, custom metrics, or an authorized host-monitoring mechanism.

---

## 26. Disk Metrics

Filesystem usage is also not directly exposed by basic EC2 APIs. Use CloudWatch Agent, SSM diagnostics, or another approved host monitoring mechanism.

---

## 27. Metric Freshness

Validate datapoint timestamps. Missing or stale metrics should become UNKNOWN or a monitoring failure rather than being interpreted as zero.

---

## 28. CloudWatch Client

Create a CloudWatch client with explicit botocore configuration for bounded connect/read timeouts and an appropriate SDK retry mode.

---

## 29. GetMetricData

For larger fleets, batch multiple metric queries with CloudWatch GetMetricData where practical instead of issuing a separate API call for every metric and instance.

---

## 30. API Cost

High-frequency polling across many instances increases API usage and operational cost. Choose a monitoring interval based on detection requirements.

---

## 31. Retry Policy

Retry transient throttling, timeouts, and temporary service failures with bounded exponential backoff and jitter. Do not retry permanent authorization or validation failures.

---

## 32. SDK Retries

Understand botocore's configured retry behavior before adding custom retries. Avoid accidentally multiplying attempts through nested retry layers.

---

## 33. Timeouts

Configure bounded connect and read timeouts. A monitoring run must have a predictable maximum duration.

---

## 34. Concurrency

Monitoring many regions/accounts can use a bounded ThreadPoolExecutor because AWS calls are I/O-bound. Start conservatively and increase concurrency only after measuring throttling and duration.

---

## 35. Rate Limiting

Combine bounded concurrency with retry/backoff and, at larger scale, rate limiting. More threads are not automatically better.

---

## 36. Multi-Region

Maintain an explicit region list and create region-scoped clients. Isolate regional failures so one bad region does not hide results from healthy regions.

---

## 37. Multi-Account

Use a central monitoring identity and assume a dedicated read-only role in each approved account. Verify the resulting STS identity before collection.

---

## 38. Account Isolation

Return account-level results independently. Record which accounts succeeded, failed, or were skipped and why.

---

## 39. Region Isolation

Similarly isolate region-level results. A single regional API failure should be visible without discarding other regional results.

---

## 40. Health Model

Represent HEALTHY, WARNING, CRITICAL, and UNKNOWN explicitly. Keep health severity separate from Python logging severity.

---

## 41. Condition Codes

Use stable machine-readable codes such as STATUS_CHECK_FAILED, CPU_HIGH, CPU_CRITICAL, DISK_CRITICAL, METRICS_UNAVAILABLE, and SCHEDULED_EVENT.

---

## 42. Rule Precedence

Critical conditions override warnings. If an instance has both high CPU and an impaired status check, the final state is CRITICAL while preserving both conditions.

---

## 43. Pure Evaluation

Keep health evaluation independent of AWS API calls. Feed normalized observations into pure functions/classes so the rules are easy to test.

---

## 44. Health Result

A health result should contain instance ID, severity, condition codes, observed values, account, region, and evaluation timestamp.

---

## 45. Fleet Summary

Summarize total, healthy, warning, critical, and unknown instances. Never hide critical or unknown counts behind a single health percentage.

---

## 46. Health Percentage

A healthy percentage can be useful for dashboards, but it must be accompanied by absolute unhealthy counts and monitoring coverage.

---

## 47. Missing Data

If required metrics are unavailable, classify the result as UNKNOWN or monitoring failure according to policy. Do not use metric=None as CPU=0.

---

## 48. Hysteresis

Use different enter/recovery thresholds when noisy metrics cause alert flapping. Example: enter warning at 80% and recover below 75%.

---

## 49. Consecutive Breaches

For noisy workloads, require multiple consecutive breaches before changing state. This reduces transient false positives.

---

## 50. State Machine

Typical lifecycle: HEALTHY → WARNING → CRITICAL → RECOVERED. Support escalation from WARNING to CRITICAL and recovery from either state.

---

## 51. Alert Deduplication

Deduplicate alerts using a deterministic key such as account, region, instance, and condition. Repeated polling should not produce repeated pages.

---

## 52. Alert Cooldown

Use a cooldown window or state-based alerting. Re-alert only when severity changes or a meaningful escalation condition occurs.

---

## 53. Recovery Alerts

When a previously critical condition returns to healthy, generate a recovery notification if the incident policy requires it.

---

## 54. Alert Aggregation

For fleet incidents, aggregate multiple unhealthy instances into a summary rather than sending hundreds of identical notifications.

---

## 55. Notification Interface

Create a notifier interface so SNS, Slack, email, or an incident-management system can be implemented independently from health evaluation.

---

## 56. Alert Failure

Alert delivery failures must be logged and measured. A monitor that detects an incident but cannot notify operators has an operational failure of its own.

---

## 57. Heartbeat

Expose a last-success timestamp so operators can detect when the monitoring system itself has stopped running.

---

## 58. Prometheus Metrics

Useful metrics include monitor runs, failures, duration, API errors, throttles, unhealthy-instance counts, and last-success timestamp.

---

## 59. Metric Cardinality

Do not use every instance ID, ARN, UUID, or run ID as a Prometheus label. Keep labels bounded such as region, environment, severity, and service.

---

## 60. Grafana

A dashboard can show fleet totals, severity counts, status failures, threshold violations, monitor duration, throttling, and monitor heartbeat.

---

## 61. ELK

Send structured JSON logs to the centralized ELK stack for detailed incident investigation. Include run ID, account, region, instance, condition, and outcome.

---

## 62. Run ID

Generate one opaque UUID per monitor run and include it in logs, reports, and CI metadata. Never put credentials or secrets into the run ID.

---

## 63. JSON Report

A report can include schema version, timestamp, account, region, environment, instance records, severity, conditions, and observed metrics.

---

## 64. CSV Report

CSV is useful for operations and audit workflows. Keep columns stable and document schema changes.

---

## 65. Atomic Reports

Write reports to a temporary file and rename atomically so consumers never read a partially written report.

---

## 66. S3 Storage

Historical health reports can be stored in S3 under date/account/region prefixes. Use encryption, restricted access, lifecycle policies, and least-privilege writer permissions.

---

## 67. Historical State

Persist previous health state in S3, DynamoDB, or another durable store when the monitor needs to detect transitions across independent runs.

---

## 68. State Key

A deterministic key such as account|region|instance|condition can identify a health condition without exposing it as a high-cardinality metric label.

---

## 69. State Expiration

Remove or expire state for terminated instances so stale incidents do not remain indefinitely.

---

## 70. Monitoring vs Application Health

An EC2 instance can be healthy while nginx, a Java service, or an application is down. For application availability, correlate with ALB target health, service checks, or application metrics.

---

## 71. ALB Correlation

For web workloads, combine EC2 health with ALB target health. EC2 healthy plus ALB unhealthy can point toward application, port, listener, security-group, or health-endpoint issues.

---

## 72. ASG Correlation

If an instance belongs to an Auto Scaling Group, understand native replacement behavior before implementing custom reboot or replacement logic.

---

## 73. Immutable Infrastructure

For immutable workloads, replacing an unhealthy instance through the platform may be safer than manually repairing it.

---

## 74. Auto-Remediation

Keep remediation separate from monitoring. Start with detection and alerting; add explicit, allowlisted, idempotent remediation only after strong safety controls exist.

---

## 75. Remediation Guard

Require correct account, environment, instance eligibility, approved action, cooldown, maximum attempts, and post-action verification before automated remediation.

---

## 76. Reboot Risk

Rebooting can interrupt sessions and workloads. Never reboot simply because a status check failed without considering redundancy, scheduled events, ASG behavior, and business impact.

---

## 77. SSM Diagnostics

An optional diagnostic mode can use Systems Manager to run predefined commands such as df -h, free -m, uptime, or service checks. Do not accept arbitrary shell input.

---

## 78. SSM Security

SSM is a more privileged capability than read-only EC2 status monitoring. Keep diagnostic permissions separate and grant them only where required.

---

## 79. Command Allowlist

Map safe diagnostic names to predefined commands. Never interpolate untrusted user input into shell commands.

---

## 80. SSM Polling

When waiting for an SSM command, use bounded polling and distinguish success, failure, timeout, cancellation, and unavailable-instance states.

---

## 81. Monitoring Profiles

A clean design can support basic EC2-only, standard EC2+CloudWatch, and diagnostic EC2+CloudWatch+SSM profiles. IAM remains the final authority.

---

## 82. Governance

Separate infrastructure health from governance findings such as missing Owner or Monitoring tags. An instance can be healthy but non-compliant.

---

## 83. Governance Severity

Use separate categories such as INFRASTRUCTURE, RESOURCE, GOVERNANCE, and MONITORING so operators understand why a finding exists.

---

## 84. Scheduler

The monitor can run from cron, Jenkins, GitHub Actions, or an EKS CronJob. The schedule should match detection requirements and API limits.

---

## 85. EKS CronJob

For Kubernetes execution, use a dedicated CronJob, ServiceAccount, workload identity, resource requests/limits, history limits, active deadline, backoff, and concurrency policy.

---

## 86. Overlap Prevention

If one scan can take longer than the schedule interval, prevent overlapping executions using CronJob concurrencyPolicy or another coordination mechanism.

---

## 87. Docker

Build a minimal Python image, install only required dependencies, run as non-root, avoid credentials in the image, scan dependencies and use an immutable image reference.

---

## 88. EKS Security

Use a dedicated ServiceAccount and workload identity. Apply runAsNonRoot, no privilege escalation, dropped capabilities, and read-only filesystem where compatible.

---

## 89. CI Pipeline

A production pipeline can run checkout → dependency install → lint → unit tests → security scan → build → image scan → deployment/inventory execution → report publication.

---

## 90. GitHub OIDC

Use GitHub Actions OIDC to assume a scoped AWS role instead of storing static AWS credentials in repository secrets.

---

## 91. Jenkins

Use a Jenkins-managed AWS role or controlled role assumption. Keep monitoring permissions separate from deployment permissions.

---

## 92. Testing Strategy

Unit-test discovery, pagination, parsing, health rules, threshold validation, retry classification, alert state transitions, report generation, and account guards. Use a dedicated AWS test account for integration tests.

---

## 93. Mocking boto3

Inject clients into service classes and use unittest.mock to simulate AWS responses. Unit tests should not require real AWS resources.

---

## 94. Pagination Test

Mock multiple API pages and verify every instance is returned. Also test empty pages and optional fields.

---

## 95. Retry Test

Mock a transient throttling error followed by success and verify bounded retry. Mock AccessDenied and verify that it is not retried.

---

## 96. Dry-Run Test

If remediation is added, assert that dry-run computes planned actions but makes zero mutation API calls.

---

## 97. Account Guard Test

Mock an unexpected STS account and assert that collection/remediation stops before mutation.

---

## 98. Integration Tests

Use an isolated AWS test account and explicitly validate its account ID. Create only controlled test resources and clean them up.

---

## 99. Failure Isolation

A single collector, account, or region can return a structured failure while other independent scopes continue when the SLA permits partial results.

---

## 100. Exit Codes

Define stable CLI exit codes such as 0 healthy, 1 warning, 2 critical, and 3 monitoring/system failure. Document them so Jenkins or GitHub Actions can consume them.

---

## 101. Production Runbook

For a critical alert: identify instance, check AWS status, CloudWatch metrics, scheduled events, ASG/ALB state, OS diagnostics if authorized, recent deployments, and then determine the least risky remediation.

---

## 102. Scenario: Status Impaired

Check whether the issue is system or instance status, inspect scheduled events/AWS Health, determine redundancy, and avoid immediate reboot unless the operational runbook calls for it.

---

## 103. Scenario: CPU Critical

Check duration and consecutive breaches, workload, deployment history, traffic, memory, process behavior, and load before concluding that the instance is unhealthy.

---

## 104. Scenario: Disk Critical

Identify the affected filesystem, inspect growth and logs, check deleted-but-open files and application behavior, and follow the storage runbook.

---

## 105. Scenario: Metrics Missing

Check CloudWatch Agent, metric namespace/dimensions, IAM, network access, timestamp freshness, and recent configuration changes. Missing data is not automatically healthy.

---

## 106. Scenario: Alert Storm

Check overlapping schedulers, duplicate workers, missing state persistence, cooldown, hysteresis, and aggregation before changing thresholds.

---

## 107. Scenario: API Throttling

Review worker count, polling frequency, duplicate API calls, SDK retries, and account/region concurrency. Reduce load before increasing retry attempts.

---

## 108. Scenario: Wrong Region

If zero instances are unexpectedly found, verify region and account first. Never infer that an empty result means the fleet is gone.

---

## 109. Scenario: Application Down

If EC2 is healthy but the application is unavailable, move up the monitoring stack: ALB target health, service status, application metrics, logs, and endpoint checks.

---

## 110. Polling vs Events

Polling is simple for reconciliation; EventBridge/SQS can provide lower-latency event-driven triggers. A robust design can use events for triggers and periodic polling for reconciliation.

---

## 111. Event Idempotency

Event-driven processing must tolerate duplicate events. Use event IDs or deterministic state keys and verify current resource state before side effects.

---

## 112. Event Ordering

Do not assume events arrive in perfect order. Treat the event as a trigger and current AWS state as authoritative during reconciliation.

---

## 113. Dead-Letter Queue

For an event-driven extension, failed SQS processing can be moved to a dead-letter queue for investigation and replay.

---

## 114. Capacity Planning

Estimate accounts × regions × instances and choose worker limits based on API quotas, expected duration, memory, and alert volume.

---

## 115. Large Fleet Design

At scale, use an account/region work queue, bounded workers, per-account fairness, batched CloudWatch queries, durable results, and centralized metrics/logs.

---

## 116. Data Separation

Store detailed instance health in S3 or a database, operational metrics in Prometheus, and detailed event logs in ELK. Each system serves a different purpose.

---

## 117. Security of Reports

Reports can reveal account IDs, private IPs, instance IDs, and application names. Restrict access and encrypt storage according to organizational policy.

---

## 118. UTC

Use timezone-aware UTC timestamps for reports, state comparisons, and alerts so multi-region operations remain consistent.

---

## 119. Detection Latency

Measure time from an actual health transition to monitor detection. This helps determine whether the polling interval and metric windows meet operational requirements.

---

## 120. Alert Latency

Measure time from detection to notification. Alert transport can fail independently from health evaluation and should be monitored.

---

## 121. Monitor SLO

Define an SLO for monitor execution success and freshness, such as the percentage of scheduled runs completed successfully and within the expected interval.

---

## 122. Self Monitoring

Track last successful execution, run duration, API failures, throttles, alert failures, and report failures. Monitoring infrastructure must be observable itself.

---

## 123. Dependency Injection

Pass AWS clients, clock functions, notifiers, and reporters into classes instead of creating global clients. This improves testing and configuration control.

---

## 124. Pure Rules

Health rules should accept observations and configuration and return deterministic results. Keep AWS API calls outside the decision layer.

---

## 125. Models

Use dataclasses for configuration, observations, health results, alerts, and collector results. Frozen configuration models help prevent accidental mutation.

---

## 126. Reporter Interface

Implement JSON, CSV, console, and S3 reporters behind a common interface so storage format does not affect monitoring logic.

---

## 127. Notifier Interface

Implement SNS, Slack, or incident-platform notifiers independently from health evaluation. Tests can inject a fake notifier.

---

## 128. Config Precedence

Document precedence such as CLI > environment variables > config file > defaults. Validate the final merged configuration before execution.

---

## 129. No Secrets in Config

Thresholds and regions are not secrets. Store only actual secrets in approved secret-management systems, and avoid unnecessarily placing credentials in configuration.

---

## 130. CloudWatch vs Python

Use native CloudWatch alarms when a simple threshold alarm is sufficient. Python adds value for custom correlation, cross-account reporting, governance, and complex workflows.

---

## 131. Avoid Reinventing AWS

Do not build a Python polling system when a mature AWS-native mechanism already solves the requirement more reliably. Use Python where custom logic provides clear value.

---

## 132. Observability Stack

For the user's DevOps stack, Prometheus and Grafana can cover operational metrics and dashboards, while ELK can provide structured detailed logs and incident investigation.

---

## 133. Production Anti-Patterns

Avoid hard-coded credentials, no pagination, infinite retries, unbounded concurrency, treating missing metrics as zero, alerting every poll, automatic reboot by default, high-cardinality Prometheus labels, and mixing monitoring with cleanup.

---

## 134. Interview: Python Choice

Python is well suited because boto3 provides mature AWS API integration, Python has strong testing and data-processing libraries, and the same automation can run locally, in CI/CD, or in Kubernetes.

---

## 135. Interview: boto3

boto3 provides structured AWS API access without spawning CLI subprocesses. Service clients map closely to AWS APIs and support paginators and botocore configuration.

---

## 136. Interview: Memory

Basic EC2 APIs do not provide Linux guest memory utilization. Use CloudWatch Agent/custom metrics or an approved host monitoring mechanism.

---

## 137. Interview: Throttling

Use SDK retries appropriately, classify throttling as transient, apply bounded exponential backoff with jitter, control concurrency, and monitor throttle rates.

---

## 138. Interview: Wrong Account

Call STS GetCallerIdentity, compare actual and expected account, and fail closed before performing sensitive operations.

---

## 139. Interview: Large Fleet

Use server-side filters, pagination, batched CloudWatch queries, bounded concurrency, per-account fairness, and durable storage. Keep resource IDs out of Prometheus labels.

---

## 140. Interview: Auto Remediation

Separate detection from remediation. Require eligibility, account/environment validation, allowlisted action, cooldown, maximum attempts, and post-action verification.

---

## 141. Interview: EC2 vs Application

EC2 health checks do not prove application health. Correlate infrastructure status with ALB target health, service checks, application metrics, and logs.

---

## 142. Interview: Testing

Keep health evaluation pure and mock boto3 clients. Test success, pagination, missing metrics, throttling, access denial, account mismatch, alert transitions, and remediation safety.

---

## 143. Interview: EKS Deployment

Package the monitor as a non-root image, run it as a CronJob, use EKS workload identity, configure resource limits and concurrency controls, and export logs/metrics.

---

## 144. Interview: Monitoring the Monitor

Expose a last-success timestamp and monitor run/error/duration metrics. A stale heartbeat is itself an operational alert.

---

## 145. 60-Second Project Answer

I built a Python/boto3 EC2 health monitor that discovers instances across configured AWS accounts and regions, checks EC2 status and CloudWatch metrics, evaluates configurable health rules, and produces reports and alerts. I designed it with least-privilege IAM, STS account validation, pagination, bounded concurrency, retries, metric freshness checks, alert deduplication, structured logging, Prometheus/ELK observability, and EKS/CI deployment options. Detection is separated from remediation so production changes remain controlled and auditable.

---

## 146. Final Workflow

Parse CLI → load and validate config → resolve identity → verify account/region → discover instances → collect status/metrics/events → evaluate health → compare state → alert on transitions → write report → emit metrics/logs → return documented exit code.

---

## 147. Final Checklist: Identity

Verify IAM role, least privilege, STS account, cross-account trust, temporary credentials, and absence of static production keys.

---

## 148. Final Checklist: Reliability

Verify pagination, filters, timeouts, retry classification, exponential backoff, jitter, bounded concurrency, API quota awareness, and partial-failure handling.

---

## 149. Final Checklist: Monitoring

Verify status checks, CloudWatch metrics where required, metric freshness, configurable thresholds, consecutive-breach logic, scheduled events, and monitoring heartbeat.

---

## 150. Final Checklist: Alerting

Verify deduplication, cooldown, state transitions, recovery alerts, aggregation, notification failure metrics, and runbook mapping.

---

## 151. Final Checklist: Deployment

Verify container scanning, non-root execution, immutable image, EKS workload identity, resource limits, CronJob overlap prevention, CI OIDC, and rollback.

---

## 152. Final Checklist: Security

Verify no secrets in source/logs/images, TLS verification enabled, restricted report access, encrypted storage, and separate optional SSM permissions.

---

## 153. Final Project Summary

The project demonstrates Python AWS automation, EC2/CloudWatch/SSM integration, IAM/STS, multi-account and multi-region design, monitoring, alerting, Prometheus/Grafana/ELK observability, CI/CD, Docker/EKS deployment, testing, security, troubleshooting, and production engineering.

---

## Recommended Production CLI

```bash
python -m ec2_monitor.cli health \
  --environment production \
  --region ap-south-1
```

## Recommended Repository Progress

```text
11-Python-DevOps-Projects/
├── 01-AWS-Resource-Automation.md        ✓
├── 02-EC2-Health-Monitor.md             ✓
├── 03-S3-Backup-Automation.md
├── 04-EKS-Pod-Monitor.md
├── 05-Kubernetes-Cleanup-Automation.md
├── 06-CI-CD-Automation.md
├── 07-Infrastructure-Health-Checker.md
└── 08-End-to-End-DevOps-Automation.md
```

**Next file: `03-S3-Backup-Automation.md`**