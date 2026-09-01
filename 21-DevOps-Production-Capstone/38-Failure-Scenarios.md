# 38-Failure-Scenarios.md

# DevOps Production Capstone — Failure Scenarios

> **Scope:** Senior DevOps / DevSecOps production failure analysis for AWS, EKS, Kubernetes, Terraform, Helm, CI/CD, GitOps, Argo CD, ALB Ingress, Prometheus, Grafana, ELK, networking, security, and application operations.

> **Production model:** RoboShop-style microservices running on AWS EKS across multiple Availability Zones. Terraform manages infrastructure, CI builds/tests/scans artifacts, ECR stores images, GitOps stores desired deployment state, Argo CD reconciles clusters, ALB Ingress exposes applications, Prometheus/Grafana provide metrics/visualization, and ELK provides centralized logging.

## 1. Purpose and Production Mindset

This chapter is a production failure laboratory for the RoboShop-style AWS/EKS DevSecOps platform. The goal is not to memorize commands. The goal is to establish a repeatable incident method:

1. Confirm the symptom and business impact.
2. Establish scope: one pod, one node, one AZ, one service, one cluster, or the whole platform.
3. Preserve evidence before changing state.
4. Follow the dependency chain from client -> DNS -> ALB -> ingress -> Service -> EndpointSlice -> Pod -> application -> database/external dependency.
5. Use metrics, logs, Kubernetes events, AWS telemetry, and recent deployment history together.
6. Form a hypothesis and test it with the smallest safe command.
7. Mitigate first when customer impact is active.
8. Identify the root cause after service stability is restored.
9. Add prevention: alert, dashboard, automation, policy, test, capacity, or runbook improvement.

Production rule: never restart everything just because the system is unhealthy. Broad restarts destroy evidence and can convert a partial failure into a full outage.

## 2. Universal Incident Investigation Framework

### Phase A — Detect
Sources include Prometheus alerts, Grafana, Alertmanager, ELK, ALB metrics, AWS Health events, Argo CD, CI/CD notifications, synthetic checks, and customer reports.

### Phase B — Triage
Capture:
- UTC start time
- affected environment
- affected service
- customer impact
- error rate
- latency
- traffic level
- recent deployments
- recent infrastructure changes
- affected AZs/nodes/pods

### Phase C — Scope
Compare healthy and unhealthy instances. Determine whether the failure follows:
- a pod
- a node
- an AZ
- a deployment version
- a namespace
- a service
- a dependency
- a network path
- a region

### Phase D — Mitigate
Typical safe mitigations:
- rollback a bad application version
- scale a workload
- remove an unhealthy pod
- cordon/drain a failed node
- restore a known-good GitOps revision
- temporarily disable a non-critical feature
- fail over to a healthy dependency

### Phase E — Recover
Verify:
- readiness
- error rate
- latency
- traffic
- queue depth
- saturation
- logs
- business transaction success

### Phase F — Prevent
Create a concrete action with an owner and validation criteria.

## 4. Failure Scenario — ALB returns 502

**Symptom:** Clients receive HTTP 502 from the AWS ALB.

**Investigation**
```bash
kubectl get ingress -A
kubectl describe ingress roboshop -n roboshop
kubectl get svc -n roboshop
kubectl get endpointslice -n roboshop
kubectl get pods -n roboshop -o wide
```
Check ALB target health, listener rules, target group port, health-check path, security groups, and application readiness. Compare ALB 5xx with application 5xx.

**Likely root causes**
- Service has no ready endpoints.
- Target port does not match the application port.
- Readiness probe is failing.
- ALB health-check path returns non-2xx.
- Network policy or security group blocks traffic.
- Application accepts connections slowly or crashes.

**Fix**
Correct Service selectors/ports, probe configuration, target-group health settings, or network controls. Do not blindly increase timeouts before proving the dependency.

**Prevention**
Alert on `TargetResponseCode` and target health, expose a dedicated readiness endpoint, and test ingress health in CI/CD.

## 5. Failure Scenario — ALB returns 503

**Symptom:** ALB returns HTTP 503.

**Investigation**
```bash
kubectl get endpointslice -n roboshop
kubectl get pods -n roboshop
kubectl describe pod <pod> -n roboshop
kubectl describe svc <service> -n roboshop
```
A common cause is zero healthy targets. Check whether pods are Ready and whether the Service selector matches pod labels.

**Root cause examples**
- Deployment scaled to zero.
- All replicas are NotReady.
- Service selector mismatch.
- Pods are in CrashLoopBackOff.
- Readiness probe points to the wrong path.

**Fix**
Restore healthy replicas and correct selectors/probes. Verify ALB target health after Kubernetes reports Ready.

**Prevention**
Use minimum replicas, PodDisruptionBudget, readiness probes, deployment health gates, and an availability SLO.

## 6. Failure Scenario — Pods stuck Pending

**Symptom:** New pods remain Pending.

**Investigation**
```bash
kubectl get pods -A --field-selector=status.phase=Pending
kubectl describe pod <pod> -n <namespace>
kubectl get nodes
kubectl describe nodes
kubectl get resourcequota -A
kubectl get pvc -A
```
Read the scheduler events at the bottom of `describe pod`.

**Root causes**
- Insufficient CPU or memory.
- NodeSelector/affinity cannot be satisfied.
- Taints without matching tolerations.
- PVC cannot bind.
- Topology constraints are impossible.
- Namespace quota is exhausted.

**Fix**
Scale the node group, correct scheduling constraints, add tolerations where justified, or resolve storage/quota problems.

**Prevention**
Capacity alerts, realistic requests, Cluster Autoscaler/Karpenter strategy, and regular capacity reviews.

## 7. Failure Scenario — Pods in CrashLoopBackOff

**Symptom:** Container repeatedly starts and exits.

**Investigation**
```bash
kubectl get pod <pod> -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl logs <pod> -n <namespace>
```
Inspect exit code, termination reason, command/args, environment variables, mounted configuration, and recent image changes.

**Root causes**
- Application startup exception.
- Missing Secret/ConfigMap.
- Wrong command or argument.
- Dependency unavailable.
- Invalid configuration.
- OOM kill.
- Application exits intentionally after a failed migration.

**Fix**
Use the previous container logs to identify the first failure. Roll back a bad image/configuration when customer impact is active.

**Prevention**
Startup tests, immutable image tags/digests, configuration validation, and deployment smoke tests.

## 8. Failure Scenario — OOMKilled container

**Symptom:** Container repeatedly terminates with `OOMKilled`.

**Investigation**
```bash
kubectl describe pod <pod> -n <namespace>
kubectl top pod <pod> -n <namespace>
kubectl top nodes
kubectl get pod <pod> -n <namespace> -o jsonpath='{.status.containerStatuses[*].lastState.terminated.reason}'
```
Compare actual memory usage with container limits and node pressure.

**Root causes**
- Limit is too low.
- Application memory leak.
- Large workload spike.
- JVM heap configured too close to container limit.
- Sidecar consumes unexpected memory.

**Fix**
For immediate availability, increase replicas or an appropriate memory limit if node capacity exists. For a leak, investigate heap/profile data and release a corrected build.

**Prevention**
Memory SLOs, realistic requests/limits, JVM container-aware settings, leak detection, and alerts before hard OOM.

## 9. Failure Scenario — High CPU utilization

**Symptom:** Service latency increases while CPU saturation is high.

**Investigation**
```bash
kubectl top pods -n roboshop --sort-by=cpu
kubectl top nodes
kubectl get hpa -n roboshop
```
Compare CPU usage with requests, HPA target, request rate, and latency.

**Root causes**
- Traffic spike.
- CPU request/limit too low.
- Inefficient application code.
- HPA cannot scale because metrics are unavailable.
- Node capacity is exhausted.

**Fix**
Scale replicas, restore metrics availability, or optimize the expensive code path. Avoid simply raising CPU limits if the nodes cannot supply capacity.

**Prevention**
HPA plus node autoscaling, load testing, CPU saturation alerts, and capacity headroom.

## 10. Failure Scenario — High memory utilization

**Symptom:** Memory steadily approaches limits or node memory pressure appears.

**Investigation**
```bash
kubectl top pods -A --sort-by=memory
kubectl top nodes
kubectl describe node <node>
```
Look for one abnormal workload versus a platform-wide increase.

**Root causes**
- Memory leak.
- Increased cache size.
- Large request payloads.
- Missing memory limits.
- Too many workloads per node.

**Fix**
Scale out, restart only as an emergency mitigation, and investigate the application allocation pattern.

**Prevention**
Requests/limits, workload-level memory dashboards, JVM/Node/Python runtime metrics where applicable, and leak testing.

## 11. Failure Scenario — Node NotReady

**Symptom:** EKS node becomes NotReady.

**Investigation**
```bash
kubectl get nodes -o wide
kubectl describe node <node>
kubectl get pods -A -o wide | grep <node>
```
Check kubelet condition, disk pressure, memory pressure, network availability, EC2 instance health, node-group events, and recent AWS changes.

**Root causes**
- Kubelet failure.
- Instance/network problem.
- Disk pressure.
- Container runtime problem.
- Node IAM/bootstrap issue.
- AZ or underlying EC2 impairment.

**Fix**
If the node is unhealthy, cordon it and drain safely:
```bash
kubectl cordon <node>
kubectl drain <node> --ignore-daemonsets --delete-emptydir-data
```
Then replace the instance through the managed node group rather than manually repairing a fundamentally unhealthy host.

**Prevention**
Node health alerts, multiple AZs, PodDisruptionBudgets, and automated node replacement.

## 12. Failure Scenario — Disk pressure on node

**Symptom:** `DiskPressure=True`, pods are evicted, or image pulls fail.

**Investigation**
```bash
kubectl describe node <node>
kubectl get events -A --sort-by=.lastTimestamp
```
On the host, where access is permitted:
```bash
df -h
df -i
```

**Root causes**
- Container image layers consume disk.
- Container logs grow excessively.
- Temporary files fill ephemeral storage.
- Inode exhaustion.

**Fix**
Reduce log retention at node level, remove unnecessary images through supported runtime mechanisms, increase node disk size, or replace the node.

**Prevention**
Ephemeral-storage requests/limits, centralized logging, node disk alerts, and regular AMI/node-group maintenance.

## 13. Failure Scenario — Service has no endpoints

**Symptom:** Service exists but receives no traffic.

**Investigation**
```bash
kubectl get svc <service> -n <namespace> -o yaml
kubectl get endpointslice -n <namespace> -l kubernetes.io/service-name=<service>
kubectl get pods -n <namespace> --show-labels
```

**Root causes**
- Selector mismatch.
- Pods not Ready.
- Wrong namespace.
- Service port/targetPort mismatch.

**Fix**
Correct labels/selectors or readiness. Verify EndpointSlices populate before testing through the ALB.

**Prevention**
Service selector tests, deployment smoke tests, and endpoint availability alerts.

## 14. Failure Scenario — Deployment rollout stuck

**Symptom:** Argo CD or Kubernetes reports a rollout that never completes.

**Investigation**
```bash
kubectl rollout status deployment/<deployment> -n <namespace>
kubectl rollout history deployment/<deployment> -n <namespace>
kubectl describe deployment <deployment> -n <namespace>
kubectl get rs -n <namespace>
kubectl get pods -n <namespace>
```

**Root causes**
- New pods never become Ready.
- Image pull failure.
- Insufficient capacity.
- Bad configuration.
- Readiness probe failure.
- PDB or scheduling constraint prevents progress.

**Fix**
Identify the failing replica set. If the release is bad, revert the GitOps change or perform the approved rollback path.

**Prevention**
Progress deadlines, pre-production testing, automated smoke tests, and progressive deployment strategies.

## 15. Failure Scenario — ImagePullBackOff

**Symptom:** Pod cannot pull an ECR image.

**Investigation**
```bash
kubectl describe pod <pod> -n <namespace>
```
Look at events for authentication, repository, tag, DNS, or network errors.

**Root causes**
- Incorrect image URI/tag.
- ECR authorization issue.
- Node IAM role lacks ECR permissions.
- Private subnet cannot reach ECR endpoints/NAT.
- Image was deleted.
- Architecture mismatch.

**Fix**
Correct the image reference and IAM/network path. Prefer immutable image digests.

**Prevention**
CI verifies image existence, ECR lifecycle policy is reviewed, node roles follow least privilege, and deployment manifests use known image digests.

## 16. Failure Scenario — Argo CD OutOfSync

**Symptom:** Argo CD reports OutOfSync.

**Investigation**
```bash
argocd app get roboshop
argocd app diff roboshop
kubectl get application -n argocd
```
Determine whether the difference was caused by a legitimate Git change, a manual Kubernetes change, Helm rendering, or an ignored field.

**Root causes**
- Manual kubectl modification.
- Git contains a newer desired state.
- Controller/defaulting changes.
- Incorrect Helm values.
- Resource mutation by another controller.

**Fix**
For GitOps-owned resources, make the intended change in Git and reconcile. Do not normalize manual drift by editing the cluster.

**Prevention**
Clear ownership boundaries, automated sync policy where appropriate, RBAC restrictions, and drift alerts.

## 17. Failure Scenario — Argo CD application Degraded

**Symptom:** Argo CD Application is Synced but Degraded.

**Investigation**
```bash
argocd app get roboshop
kubectl get pods -n roboshop
kubectl get deploy -n roboshop
kubectl get events -n roboshop --sort-by=.lastTimestamp
```
Synced means desired manifests were applied; it does not mean the workload is healthy.

**Root causes**
- Pods failing readiness.
- Service unavailable.
- Ingress unhealthy.
- Deployment timeout.
- Stateful dependency failure.

**Fix**
Follow the resource health chain and repair the failing workload/dependency.

**Prevention**
Application health checks, rollout gates, and alerting on both Sync and Health state.

## 18. Failure Scenario — GitOps deploys wrong image

**Symptom:** Pipeline succeeds but production runs an unexpected image.

**Investigation**
Check the commit that changed the GitOps repository, rendered Helm output, Argo CD application revision, and running image:
```bash
kubectl get deployment <deployment> -n <namespace> -o jsonpath='{.spec.template.spec.containers[*].image}'
```

**Root causes**
- Tag reuse such as `latest`.
- Wrong environment values file.
- Race between image update commits.
- CI updated the wrong repository/path.
- Argo CD tracks the wrong branch.

**Fix**
Use immutable tags or digests and correct the GitOps update workflow.

**Prevention**
Environment-specific repository paths, protected branches, image promotion rules, and commit verification.

## 19. Failure Scenario — Prometheus target down

**Symptom:** Prometheus reports a target as down.

**Investigation**
```bash
kubectl get servicemonitor,podmonitor -A
kubectl get endpointslice -A
```
In Prometheus, inspect the target's last scrape error and endpoint.

**Root causes**
- Metrics endpoint unavailable.
- ServiceMonitor selector mismatch.
- NetworkPolicy blocks scraping.
- TLS/authentication mismatch.
- Pod restart or service change.

**Fix**
Restore the metrics endpoint and align ServiceMonitor labels/selectors.

**Prevention**
Monitor Prometheus itself and alert on `up == 0` for critical targets.

## 20. Failure Scenario — Prometheus disk full

**Symptom:** Prometheus cannot persist samples or becomes unstable.

**Investigation**
```bash
kubectl get pvc -n monitoring
kubectl describe pvc <pvc> -n monitoring
kubectl logs -n monitoring <prometheus-pod>
```

**Root causes**
- Retention too long.
- High cardinality.
- PVC too small.
- Excessive scrape volume.

**Fix**
Reduce unnecessary cardinality/scrape frequency, increase storage, and adjust retention deliberately. Do not delete the TSDB blindly.

**Prevention**
Capacity forecasting, WAL/storage alerts, recording rules, cardinality governance, and retention policies.

## 21. Failure Scenario — Alert storm

**Symptom:** Hundreds or thousands of alerts arrive during one incident.

**Investigation**
Group alerts by alertname, namespace, instance, cluster, and dependency. Determine whether one root failure generated many child symptoms.

**Root causes**
- Missing grouping.
- Missing inhibition.
- Every pod/node alert has identical priority.
- Thresholds are too sensitive.
- Dependency-aware alerting is absent.

**Fix**
Group related alerts, inhibit symptoms when a parent dependency is down, and prioritize actionable alerts.

**Prevention**
Alert design reviews, severity policy, cardinality control, and quarterly alert audits.

## 22. Failure Scenario — Alert flapping

**Symptom:** Alert repeatedly fires and resolves.

**Investigation**
Inspect the metric over time, scrape interval, evaluation interval, threshold, and workload behavior.

**Root causes**
- Threshold too close to normal operating level.
- Short transient spikes.
- Missing `for:` duration.
- Unstable dependency.

**Fix**
Add an appropriate pending duration, use rate/percentile windows, and alert on sustained user impact rather than a single sample.

**Prevention**
Use multi-window conditions and SLO-based alerts for important services.

## 23. Failure Scenario — Grafana unavailable

**Symptom:** Dashboards cannot be opened.

**Investigation**
```bash
kubectl get pods,svc,ingress -n monitoring
kubectl logs -n monitoring deploy/grafana
kubectl describe pod -n monitoring -l app.kubernetes.io/name=grafana
```

**Root causes**
- Pod crash.
- PVC issue.
- Database/config issue.
- Ingress/ALB issue.
- Resource exhaustion.

**Fix**
Restore the Grafana deployment and verify datasource connectivity. Monitoring UI failure should not stop Prometheus alert evaluation.

**Prevention**
Separate alerting availability from visualization availability and back up dashboard/configuration state.

## 24. Failure Scenario — ELK ingestion lag

**Symptom:** Logs arrive late or are missing.

**Investigation**
Check shipper/Logstash queues, Elasticsearch cluster health, ingestion rate, CPU/memory, disk, and rejected requests.

**Root causes**
- Elasticsearch indexing saturation.
- Logstash pipeline bottleneck.
- Network interruption.
- Disk watermark reached.
- Excessive log volume.

**Fix**
Protect the cluster from overload, scale the ingestion tier, reduce noisy logs, and resolve storage pressure.

**Prevention**
Ingestion SLOs, queue monitoring, index lifecycle policies, and disk watermark alerts.

## 25. Failure Scenario — Elasticsearch disk watermark

**Symptom:** Elasticsearch becomes read-only or allocation stops.

**Investigation**
Check cluster health, node disk usage, shard allocation, and index growth.

**Root causes**
- Log retention too long.
- Unexpected traffic/log explosion.
- Insufficient storage.
- Poor shard sizing.

**Fix**
Apply the approved retention policy, delete expired indices through lifecycle management, add capacity, and rebalance shards. Do not disable safety watermarks as a first response.

**Prevention**
ILM, disk alerts well before the watermark, shard sizing reviews, and ingestion controls.

## 26. Failure Scenario — Terraform apply failure

**Symptom:** Infrastructure deployment fails.

**Investigation**
```bash
terraform fmt -check
terraform validate
terraform plan
terraform state list
```
Read the first provider/API error rather than the final cascade.

**Root causes**
- IAM denied.
- Resource dependency issue.
- Quota reached.
- Invalid variable.
- Drift.
- State lock.
- Provider/API transient failure.

**Fix**
Correct the actual provider error. For state issues, follow the team's locking and recovery process; never delete the state file to make an apply succeed.

**Prevention**
Plan in CI, remote state with locking, policy checks, least-privilege CI roles, and separate environment states.

## 27. Failure Scenario — Terraform state lock stuck

**Symptom:** Terraform reports that state is locked.

**Investigation**
Confirm whether another pipeline or engineer is actively running Terraform. Inspect the lock metadata through the configured backend.

**Root causes**
- Interrupted CI job.
- Abandoned lock.
- Concurrent applies.

**Fix**
If and only if the lock owner is confirmed inactive, use the backend-supported force-unlock procedure with the lock ID. Never force unlock an active apply.

**Prevention**
Serialized infrastructure pipelines and clear ownership of environment applies.

## 28. Failure Scenario — AWS IAM AccessDenied

**Symptom:** AWS API operation fails with `AccessDenied`.

**Investigation**
Identify the principal, exact API action, resource, condition keys, SCPs, permission boundaries, session policies, and resource policies.

**Root causes**
- Missing IAM action.
- Explicit deny.
- SCP.
- Permission boundary.
- Wrong role.
- Cross-account trust issue.

**Fix**
Grant the minimum required permission at the correct layer or correct the role assumption/trust relationship.

**Prevention**
IAM policy testing, least privilege, role-based CI/CD, and CloudTrail auditing.

## 29. Failure Scenario — EKS API access failure

**Symptom:** `kubectl` cannot reach or authenticate to EKS.

**Investigation**
```bash
aws sts get-caller-identity
aws eks update-kubeconfig --region <region> --name <cluster>
kubectl cluster-info
kubectl auth can-i get pods -A
```
Check cluster endpoint configuration, IAM authentication, network reachability, and RBAC.

**Root causes**
- Wrong AWS profile/role.
- Private endpoint inaccessible from the client network.
- Missing EKS access entry/RBAC.
- Expired or incorrect credentials.

**Fix**
Use the correct role and network path, then verify Kubernetes authorization separately from AWS authentication.

**Prevention**
Standardized access roles, audited EKS access, private administrative connectivity, and break-glass procedures.

## 30. Failure Scenario — Security group blocks application traffic

**Symptom:** Connection times out between ALB, nodes, or external services.

**Investigation**
Map source -> destination -> port and check security-group inbound/outbound rules and route tables.

**Root causes**
- Missing inbound rule.
- Incorrect source SG.
- Wrong port.
- Egress restriction.
- Cross-AZ/network architecture mismatch.

**Fix**
Allow only the required source and destination on the required port.

**Prevention**
Document traffic flows and use SG references instead of broad CIDRs where possible.

## 31. Failure Scenario — NACL blocks traffic

**Symptom:** Intermittent or asymmetric network failure despite apparently correct security groups.

**Investigation**
Check subnet route tables and both inbound and outbound NACL rules. Remember that NACLs are stateless.

**Root causes**
- Ephemeral response ports blocked.
- One subnet has different rules.
- Deny rule has higher precedence.

**Fix**
Correct the stateless return path and validate from both sides.

**Prevention**
Keep NACLs simple, centrally managed, and tested rather than using them as the primary micro-segmentation layer.

## 32. Failure Scenario — DNS resolution failure

**Symptom:** Applications cannot resolve service or external hostnames.

**Investigation**
```bash
kubectl exec -n roboshop <pod> -- nslookup <hostname>
kubectl exec -n roboshop <pod> -- cat /etc/resolv.conf
```
Check CoreDNS pods, service endpoints, VPC DNS settings, and upstream resolver reachability.

**Root causes**
- CoreDNS unavailable.
- NetworkPolicy blocks DNS.
- VPC DNS configuration issue.
- Incorrect hostname.
- Upstream DNS failure.

**Fix**
Restore CoreDNS and DNS network access. Verify both cluster-internal and external resolution.

**Prevention**
CoreDNS HA, DNS latency/error alerts, and explicit DNS egress rules.

## 33. Failure Scenario — CoreDNS failure

**Symptom:** Many unrelated services fail simultaneously with connection errors.

**Investigation**
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
kubectl get svc -n kube-system kube-dns
```

**Root causes**
- CoreDNS pods unavailable.
- CPU/memory starvation.
- Bad CoreDNS configuration.
- Network path to upstream resolver broken.

**Fix**
Restore healthy replicas and resources; roll back a bad configuration if recently changed.

**Prevention**
Run multiple replicas across nodes/AZs and alert on DNS errors/latency.

## 34. Failure Scenario — NetworkPolicy blocks traffic

**Symptom:** Pod-to-pod connection fails after a security change.

**Investigation**
```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy <policy> -n <namespace>
kubectl exec -n <namespace> <pod> -- curl -v http://<service>:<port>
```
Compare the source labels, destination labels, and namespace selectors.

**Root causes**
- Default-deny policy added without required allow rule.
- Label changed.
- DNS egress not allowed.
- Monitoring scraper not allowed.

**Fix**
Add the minimum required allow policy and test from the actual source workload.

**Prevention**
Policy tests in CI and documented communication matrix.

## 35. Failure Scenario — PVC stuck Pending

**Symptom:** Stateful workload cannot start because its PVC remains Pending.

**Investigation**
```bash
kubectl get pvc,pv -A
kubectl describe pvc <pvc> -n <namespace>
kubectl get storageclass
```

**Root causes**
- Missing StorageClass.
- CSI driver problem.
- Availability-zone topology mismatch.
- Quota/capacity issue.
- Invalid access mode.

**Fix**
Resolve the storage-class/CSI/capacity problem and verify the PV binds.

**Prevention**
EBS CSI monitoring, tested StorageClasses, topology-aware scheduling, and storage capacity alerts.

## 36. Failure Scenario — ALB health check fails

**Symptom:** Targets exist but ALB marks them unhealthy.

**Investigation**
Check health-check protocol, port, path, success codes, readiness state, and security groups.

**Root causes**
- Health endpoint requires authentication.
- Wrong path.
- App listens on another port.
- ALB cannot reach target.

**Fix**
Expose a lightweight unauthenticated health endpoint suitable for load-balancer health checking and align target settings.

**Prevention**
Health-check contract tests and deployment validation.

## 37. Failure Scenario — High application latency

**Symptom:** p95/p99 latency exceeds SLO.

**Investigation**
Start with RED signals: request rate, error rate, duration. Correlate with CPU, memory, downstream calls, database latency, ALB metrics, and recent deployments.

**Root causes**
- Slow dependency.
- CPU saturation.
- Connection pool exhaustion.
- Lock/contention.
- Database query regression.
- Network retransmissions.

**Fix**
Mitigate the dominant bottleneck, such as scaling the service or rolling back a regression.

**Prevention**
Latency SLO alerts, percentile dashboards, dependency metrics, load testing, and performance budgets.

## 38. Failure Scenario — High HTTP error rate

**Symptom:** 5xx percentage exceeds the error-budget threshold.

**Investigation**
Break errors down by route, service, pod, version, status code, and dependency.

**Root causes**
- Bad release.
- Dependency outage.
- Capacity exhaustion.
- Invalid configuration.
- Database failure.

**Fix**
If correlated with a deployment, stop promotion and roll back through GitOps. If dependency-related, mitigate/fail over according to the runbook.

**Prevention**
Multi-window SLO alerts and deployment correlation.

## 39. Failure Scenario — Database connection pool exhaustion

**Symptom:** Applications report connection timeout or pool exhaustion.

**Investigation**
Check application pool metrics, database connection count, query latency, and pod replica count.

**Root causes**
- Connection leak.
- Pool size too high across many replicas.
- Slow queries hold connections.
- Database max connections reached.

**Fix**
Reduce excessive concurrency, restore healthy queries, or scale database capacity where justified.

**Prevention**
Connection-pool sizing formula, database connection alerts, query monitoring, and load tests.

## 40. Failure Scenario — CI pipeline fails during image build

**Symptom:** CI cannot build the container.

**Investigation**
Identify whether failure is source compilation, dependency resolution, Docker build, registry access, scanner, or credentials.

**Root causes**
- Broken dependency.
- Dockerfile error.
- Base image unavailable.
- Registry outage.
- Credential expiry.

**Fix**
Repair the first failing stage. Do not bypass security scans merely to force production deployment.

**Prevention**
Pinned dependencies/base images, reproducible builds, artifact caching, and pipeline observability.

## 41. Failure Scenario — Trivy finds a critical vulnerability

**Symptom:** Security gate blocks an image.

**Investigation**
Identify package, CVE, installed version, fixed version, exploitability, and whether the package is runtime-required.

**Fix**
Upgrade the base image/package and rebuild. If a temporary exception is genuinely required, document owner, expiry, rationale, and compensating control.

**Prevention**
Automated scanning, dependency updates, SBOM generation, and vulnerability SLAs.

## 42. Failure Scenario — SonarQube quality gate fails

**Symptom:** CI blocks promotion after static analysis.

**Investigation**
Review the quality gate metric: bugs, vulnerabilities, code smells, duplication, coverage, or new-code conditions.

**Fix**
Correct the code or test coverage. Do not lower the gate to hide a regression without engineering approval.

**Prevention**
Developer feedback before merge, branch protection, and measurable quality standards.

## 43. Failure Scenario — Veracode/application security gate fails

**Symptom:** Application security scan blocks release.

**Investigation**
Classify findings by severity, exploitability, source/sink, and affected build artifact.

**Fix**
Remediate the vulnerable code/dependency or follow the approved risk-acceptance process.

**Prevention**
Security scanning earlier in the SDLC and tracked remediation SLAs.

## 44. Failure Scenario — Jenkins agent unavailable

**Symptom:** Builds remain queued because an agent is unavailable.

**Investigation**
Check executor capacity, agent connectivity, labels, cloud-agent provisioning, disk, and credentials.

**Root causes**
- Capacity exhaustion.
- Agent crashed.
- Label mismatch.
- Workspace disk full.
- Network/credential issue.

**Fix**
Restore capacity or replace the failed ephemeral agent.

**Prevention**
Ephemeral agents, queue-depth alerts, workspace cleanup, and resilient controller architecture.

## 45. Failure Scenario — GitHub Actions runner failure

**Symptom:** Workflow cannot acquire or complete a runner.

**Investigation**
Check runner availability, job logs, permissions, secrets, concurrency controls, and artifact storage.

**Fix**
Restore runner capacity or correct permissions. Keep production deployment credentials short-lived where possible.

**Prevention**
Reusable workflows, least-privilege tokens, protected environments, and runner monitoring.

## 46. Failure Scenario — Helm upgrade failure

**Symptom:** Helm release fails during upgrade.

**Investigation**
```bash
helm status <release> -n <namespace>
helm history <release> -n <namespace>
helm get values <release> -n <namespace>
helm get manifest <release> -n <namespace>
```

**Root causes**
- Invalid rendered manifest.
- Immutable field change.
- Bad hook.
- Resource conflict.
- Incorrect values.

**Fix**
Render and validate before deployment. Roll back to the last known-good revision if the release is unhealthy.

**Prevention**
`helm lint`, template tests, schema validation, and deployment smoke tests.

## 47. Failure Scenario — Certificate expiry

**Symptom:** HTTPS clients receive certificate errors or ALB TLS stops working.

**Investigation**
Inspect certificate expiration, ACM association, listener configuration, DNS name, and certificate renewal events.

**Root causes**
- Certificate not renewed.
- Wrong certificate attached.
- Domain mismatch.
- DNS validation problem.

**Fix**
Restore a valid ACM certificate and validate the complete TLS path.

**Prevention**
Expiry alerts well before the deadline and automated ACM renewal monitoring.

## 48. Failure Scenario — Argo CD repository authentication failure

**Symptom:** Argo CD cannot fetch the Git repository.

**Investigation**
```bash
argocd repo list
argocd repo get <repo>
kubectl get secret -n argocd
```
Check repository URL, credential, SSH key, token expiry, and network access.

**Fix**
Rotate/update the repository credential through the approved secret mechanism.

**Prevention**
Short-lived credentials where supported, secret rotation, and alerts on repository sync failures.

## 49. Failure Scenario — Kubernetes secret missing

**Symptom:** Pods fail startup because required environment variables or mounted files are absent.

**Investigation**
```bash
kubectl get secret <secret> -n <namespace>
kubectl describe pod <pod> -n <namespace>
```
Never print secret values into incident tickets or chat.

**Root causes**
- Secret not deployed.
- Wrong namespace/name.
- External secret synchronization failure.
- GitOps ordering problem.

**Fix**
Restore the secret through the approved secret-management path and restart only the affected workload if necessary.

**Prevention**
Secret existence checks, deployment dependencies, and external-secret health alerts.

## 50. Failure Scenario — EKS node group scaling failure

**Symptom:** Cluster cannot add capacity despite Pending pods.

**Investigation**
Check node group health, ASG events, subnet capacity, EC2 quotas, instance availability, IAM, and autoscaler events.

**Root causes**
- AWS quota.
- Insufficient capacity in an AZ.
- Launch template error.
- IAM failure.
- Subnet IP exhaustion.

**Fix**
Restore a viable capacity path, possibly by using additional AZs/instance types or correcting subnet capacity.

**Prevention**
Multi-AZ node groups, quota monitoring, subnet IP planning, and capacity diversification.

## 51. Failure Scenario — Subnet IP exhaustion

**Symptom:** Pods cannot obtain IP addresses or nodes cannot scale.

**Investigation**
Review subnet free IPs, ENI/IP allocation, VPC CNI behavior, and node/pod density.

**Root causes**
- Subnets sized too small.
- Excessive pod density.
- Rapid cluster growth.

**Fix**
Expand/add subnets where architecture permits and tune pod density/instance selection.

**Prevention**
Large private subnets, secondary CIDR planning, IP utilization alerts, and capacity forecasting.

## 52. Failure Scenario — ECR unavailable from private subnet

**Symptom:** Nodes in private subnets cannot pull images.

**Investigation**
Check NAT or VPC endpoints for ECR API, ECR DKR, and S3 paths, plus DNS and security groups.

**Root causes**
- Missing endpoint.
- Broken NAT route.
- Endpoint policy too restrictive.
- DNS resolution failure.

**Fix**
Restore the private connectivity path and validate image pull.

**Prevention**
Document ECR dependency paths and monitor endpoint/NAT health.

## 53. Failure Scenario — Unexpected production configuration drift

**Symptom:** Live behavior differs from Git.

**Investigation**
Compare Argo CD diff, live manifests, Git revision, Helm values, and recent manual operations.

**Root causes**
- Direct kubectl change.
- Controller mutation.
- Emergency change not backported to Git.
- Wrong environment branch.

**Fix**
Determine intended state and reconcile Git with production deliberately.

**Prevention**
Restrict production kubectl write access, use GitOps for routine changes, and record emergency changes immediately.

## 54. Failure Scenario — Bad deployment causes customer outage

**Symptom:** Error rate spikes immediately after release.

**Investigation**
Correlate deployment timestamp with ALB/Prometheus metrics and compare old/new replica sets.

**Fix**
Stop promotion and revert the GitOps change to the last known-good commit. Verify rollout and customer SLO recovery.

**Prevention**
Automated canary/progressive rollout, smoke tests, SLO-based deployment gates, and fast rollback paths.

## 55. Failure Scenario — AZ-level workload failure

**Symptom:** A subset of pods or traffic in one Availability Zone fails.

**Investigation**
Map pods/nodes/targets by AZ:
```bash
kubectl get nodes -L topology.kubernetes.io/zone
kubectl get pods -n roboshop -o wide
```
Compare affected and healthy AZs.

**Fix**
Shift workload to healthy capacity, replace unhealthy nodes, and validate load-balancer distribution.

**Prevention**
Multi-AZ node groups, topology spread constraints, PDBs, and enough spare capacity in each AZ.

## 56. Failure Scenario — Complete application dependency outage

**Symptom:** Multiple services fail because a shared database/cache/external API is unavailable.

**Investigation**
Identify the common dependency by comparing error patterns across services.

**Fix**
Use the dependency's failover/recovery procedure. Reduce retry storms and, where supported, degrade non-critical functionality.

**Prevention**
Circuit breakers, bounded retries with jitter, dependency SLOs, bulkheads, and capacity planning.

## 57. Failure Scenario — Retry storm

**Symptom:** A dependency becomes slow and traffic to it increases rather than decreases.

**Investigation**
Compare request rate, retry rate, downstream latency, and connection counts.

**Root cause**
Unbounded or synchronized client retries amplify an existing failure.

**Fix**
Reduce retries, restore dependency capacity, and enable controlled backoff/circuit breaking.

**Prevention**
Exponential backoff, jitter, retry budgets, timeouts, and circuit breakers.

## 58. Failure Scenario — Monitoring blind spot

**Symptom:** Production fails but no alert fired.

**Investigation**
Reconstruct which signal should have detected the failure: availability, latency, error rate, saturation, logs, or synthetic checks.

**Root causes**
- Missing alert.
- Wrong threshold.
- Scrape target absent.
- Alert route broken.
- Alert was silenced incorrectly.

**Fix**
Restore monitoring and create a regression test for the alert.

**Prevention**
Alert coverage reviews mapped to SLOs and periodic failure injection/drills.

## 59. Failure Scenario — Alert notification delivery failure

**Symptom:** Prometheus/Alertmanager shows firing alerts but responders receive nothing.

**Investigation**
Check Alertmanager status, routing, receiver configuration, notification errors, credentials, and destination health.

**Fix**
Restore the notification integration and use an independent escalation path during the incident.

**Prevention**
Dead-man's-switch alerts, receiver health checks, and periodic notification tests.

## 60. Failure Scenario — Dead-man's-switch failure

**Symptom:** There is no evidence that the alerting pipeline is alive.

**Investigation**
Verify a heartbeat/always-firing test alert reaches the expected receiver.

**Fix**
Restore the monitoring pipeline and confirm notification delivery.

**Prevention**
A dedicated heartbeat alert should be treated as an operational control, not as ordinary business alert noise.

## 61. Failure Scenario — PromQL alert never fires

**Symptom:** Dashboard metric looks abnormal but the alert remains inactive.

**Investigation**
Run the exact PromQL expression in Prometheus, inspect label sets, evaluation time range, and the `for:` duration.

**Root causes**
- Label mismatch.
- Query returns empty vector.
- Wrong metric name.
- Recording rule not available.
- Threshold units misunderstood.

**Fix**
Test the query against real labels and add a rule test where possible.

**Prevention**
Version-controlled rules, promtool validation, representative test data, and alert review.

## 62. Failure Scenario — PrometheusRule rejected

**Symptom:** Kubernetes monitoring operator does not load the rule.

**Investigation**
```bash
kubectl describe prometheusrule <rule> -n monitoring
kubectl get prometheusrule -A
```
Validate YAML and PromQL.

**Fix**
Correct schema, labels, expressions, or selectors. Confirm the rule appears in Prometheus.

**Prevention**
CI validation with `promtool check rules` and Kubernetes schema validation.

## 63. Failure Scenario — Alertmanager routing sends production alerts to the wrong team

**Symptom:** Alerts reach an unrelated receiver.

**Investigation**
Trace labels through the route tree: environment, team, severity, service, alertname.

**Root causes**
- Route order wrong.
- Matcher too broad.
- Missing team label.
- Default receiver inappropriate.

**Fix**
Correct the most-specific route and test with representative labels.

**Prevention**
Routing tests, mandatory ownership labels, and a safe default receiver.

## 64. Failure Scenario — Kubernetes API server latency

**Symptom:** `kubectl` operations and controllers become slow.

**Investigation**
Check API server request rate/latency, throttling, controller behavior, admission webhooks, and cluster load.

**Root causes**
- Excessive API request volume.
- Broken controller loop.
- Admission webhook latency.
- Very large object/list operations.

**Fix**
Identify the noisy client/controller and reduce request pressure. Avoid repeatedly polling the API from ad-hoc scripts.

**Prevention**
API server monitoring, efficient controllers, and webhook timeouts/failure policies designed for availability.

## 65. Failure Scenario — Admission webhook outage

**Symptom:** New resources cannot be created or updated.

**Investigation**
Check webhook service/endpoints, certificates, timeout errors, and webhook failure policy.

**Root causes**
- Webhook pods down.
- Certificate expired.
- Service has no endpoints.
- Network path blocked.
- Webhook is too slow.

**Fix**
Restore webhook health or use the approved emergency policy if the webhook is non-critical.

**Prevention**
HA webhook deployment, certificate monitoring, conservative timeout settings, and tested failure policies.

## 66. Failure Scenario — Deployment causes insufficient node capacity

**Symptom:** New version requires more resources and old replicas cannot coexist.

**Investigation**
Compare old/new requests with node allocatable capacity and rollout strategy.

**Root cause**
RollingUpdate surge plus large resource requests exceeds cluster headroom.

**Fix**
Add capacity or adjust rollout parameters after validating application safety.

**Prevention**
Capacity headroom, resource sizing, and deployment strategy reviews.

## 67. Failure Scenario — PDB blocks node drain

**Symptom:** `kubectl drain` cannot evict a pod.

**Investigation**
```bash
kubectl get pdb -A
kubectl describe pdb <pdb> -n <namespace>
kubectl get pods -n <namespace>
```

**Root causes**
- Too few replicas.
- PDB `minAvailable` too strict.
- Existing unhealthy replicas.

**Fix**
Restore enough healthy replicas before maintenance. Do not routinely bypass PDB protection.

**Prevention**
PDBs sized with realistic replica counts and tested maintenance procedures.

## 68. Failure Scenario — Container readiness probe fails after deployment

**Symptom:** Pods are Running but never Ready.

**Investigation**
```bash
kubectl describe pod <pod> -n <namespace>
kubectl exec -n <namespace> <pod> -- curl -v http://127.0.0.1:<port>/health
```

**Root causes**
- Wrong path/port.
- Startup takes longer.
- Health endpoint depends on a non-critical downstream service.
- Probe timeout too short.

**Fix**
Correct the probe contract or use startup probes for slow initialization.

**Prevention**
Probe tests in CI and clear separation of startup, readiness, and liveness semantics.

## 69. Failure Scenario — Liveness probe causes restart loop

**Symptom:** Healthy-but-busy application repeatedly restarts.

**Root cause**
Liveness probe is checking a dependency or expensive operation rather than whether the process is fundamentally alive.

**Fix**
Make liveness lightweight and local. Use readiness for dependency availability.

**Prevention**
Probe design review and controlled failure testing.

## 70. Failure Scenario — Application log volume explodes

**Symptom:** ELK ingestion and storage grow rapidly.

**Investigation**
Identify the top logger, message pattern, namespace, pod, and deployment version.

**Root causes**
- Debug logging enabled in production.
- Error loop.
- Repeated stack traces.
- Request logging without sampling.

**Fix**
Reduce the source of repeated logging and restore appropriate log level. Protect the logging platform from overload.

**Prevention**
Structured logs, sampling, rate limits, retention policy, and log-volume alerts.

## 71. Failure Scenario — Secret leaked into logs

**Symptom:** A credential appears in application or CI logs.

**Immediate response**
Treat the secret as compromised. Revoke/rotate it, identify exposure scope, preserve evidence, and notify security according to policy.

**Investigation**
Search centralized logs and CI artifacts without copying the secret into tickets.

**Prevention**
Secret redaction, secret managers, CI masking, least privilege, and automated secret scanning.

## 72. Failure Scenario — Unexpected privileged container

**Symptom:** A workload runs with more Linux/Kubernetes privileges than intended.

**Investigation**
Inspect `securityContext`, ServiceAccount, RBAC, host namespaces, capabilities, and admission policies.

**Fix**
Remove unnecessary privileges and redeploy through GitOps.

**Prevention**
Pod Security Standards, admission controls, least-privilege RBAC, and security review.

## 73. Failure Scenario — ServiceAccount permission failure

**Symptom:** Application receives Kubernetes `Forbidden` errors.

**Investigation**
```bash
kubectl auth can-i get secrets --as=system:serviceaccount:<namespace>:<sa> -n <namespace>
kubectl get role,rolebinding,clusterrole,clusterrolebinding -A
```

**Fix**
Grant only the required resource/action at the narrowest scope.

**Prevention**
RBAC review and automated authorization tests.

## 74. Failure Scenario — Node IAM permission failure

**Symptom:** EKS node or workload-integrated AWS operation receives AccessDenied.

**Investigation**
Determine whether the action is performed by the node role, pod identity mechanism, or another assumed role.

**Fix**
Correct the identity mapping and least-privilege policy.

**Prevention**
Avoid putting broad AWS permissions on node roles; use workload-specific identity.

## 75. Failure Scenario — ALB listener rule misroutes traffic

**Symptom:** Requests reach the wrong microservice.

**Investigation**
Review host/path conditions, listener priority, ingress annotations, and generated ALB rules.

**Root causes**
- Overlapping path rules.
- Incorrect host.
- Rule priority conflict.
- Wrong backend Service.

**Fix**
Correct the Ingress specification and reconcile through GitOps.

**Prevention**
Ingress integration tests and explicit routing conventions.

## 76. Failure Scenario — DNS points to wrong ALB

**Symptom:** Users reach an old or wrong environment.

**Investigation**
Check DNS record target, TTL, hosted zone, ALB hostname, and environment naming.

**Fix**
Correct the DNS record and verify propagation and ALB certificate coverage.

**Prevention**
Infrastructure-as-code DNS, change review, and synthetic endpoint checks.

## 77. Failure Scenario — Production rollback does not restore health

**Symptom:** Application is rolled back but errors continue.

**Investigation**
Check whether the rollback changed only the image while configuration, database schema, external dependency, or infrastructure remained changed.

**Root causes**
- Backward-incompatible database migration.
- Persistent configuration change.
- Shared dependency failure.
- Infrastructure change unrelated to application image.

**Fix**
Restore the complete known-good state, not just the container image.

**Prevention**
Backward-compatible migrations, versioned configuration, and release dependency mapping.

## 78. Failure Scenario — Database migration breaks older application

**Symptom:** Rolling back the application causes schema errors.

**Root cause**
A forward-only or destructive migration was deployed before application rollback safety was established.

**Fix**
Use a compatible migration strategy. If necessary, restore database state through the approved recovery plan rather than guessing SQL in production.

**Prevention**
Expand/contract migrations, backward compatibility, tested rollback scenarios, and database backups.

## 79. Failure Scenario — Terraform drift detected

**Symptom:** Plan proposes unexpected changes.

**Investigation**
Determine whether drift came from an emergency manual change, AWS-managed behavior, or an incorrect Terraform configuration.

**Fix**
Decide explicitly whether the real-world state or Terraform configuration is authoritative, then reconcile.

**Prevention**
Restrict manual infrastructure changes and run scheduled drift detection.

## 80. Failure Scenario — AWS quota exhaustion

**Symptom:** Resource creation fails despite valid configuration.

**Investigation**
Read the exact AWS quota/API error and identify the affected resource family, region, or account.

**Fix**
Use available capacity in the short term or request quota increase through the standard process.

**Prevention**
Quota dashboards, proactive requests, and capacity planning.

## 81. Failure Scenario — EKS control-plane dependency misunderstood

**Symptom:** Team attempts to repair AWS-managed EKS control-plane components like ordinary pods.

**Root cause**
Operational ownership boundary is misunderstood.

**Fix**
Use AWS/EKS control-plane telemetry and supported configuration. Focus Kubernetes-side troubleshooting on worker/node/add-on and workload layers.

**Prevention**
Document AWS-managed versus customer-managed responsibilities.

## 82. Failure Scenario — Cluster-wide outage after security policy change

**Symptom:** Many namespaces lose connectivity after a NetworkPolicy, security-group, IAM, or admission change.

**Investigation**
Compare the exact change time with outage onset. Roll back the policy in a controlled way if it is proven causal.

**Fix**
Restore service first, then redesign the policy with staged rollout.

**Prevention**
Policy-as-code validation, canary namespaces, and change windows for high-blast-radius security controls.

## 83. Failure Scenario — Observability cardinality explosion

**Symptom:** Prometheus memory/storage increases sharply and queries slow down.

**Root causes**
- High-cardinality labels such as request IDs, user IDs, URLs, or unbounded error text.
- Excessive per-object metrics.

**Fix**
Remove unbounded labels and aggregate dimensions. Use recording rules for expensive common queries.

**Prevention**
Metric naming/label governance and cardinality review in code review.

## 84. Failure Scenario — Prometheus remote storage or scrape overload

**Symptom:** Scrapes fail or evaluation becomes delayed.

**Investigation**
Check scrape duration, sample ingestion rate, target count, rule evaluation duration, and Prometheus resource usage.

**Fix**
Reduce unnecessary scrape frequency/targets, increase resources, and optimize rules.

**Prevention**
Capacity planning based on samples/sec and rule cost.

## 85. Failure Scenario — Incident caused by clock skew

**Symptom:** TLS, authentication, tokens, logs, or monitoring timestamps behave unexpectedly.

**Investigation**
Compare node/system time with a trusted time source.

**Fix**
Restore time synchronization through the supported OS/node image configuration.

**Prevention**
Monitor time synchronization on critical hosts and systems.

## 86. Failure Scenario — Production change made outside GitOps

**Symptom:** A manual fix works temporarily but Argo CD later reverts it.

**Root cause**
The live cluster was changed without updating desired state.

**Fix**
If the change is valid, commit the equivalent desired configuration to Git. If it was emergency-only, document it and reconcile afterward.

**Prevention**
Production write controls and an emergency-change procedure.

## 87. Failure Scenario — Wrong environment deployed

**Symptom:** QA/staging artifact or configuration appears in production.

**Investigation**
Trace pipeline commit, artifact digest, GitOps commit, Helm values, Argo CD Application source, and cluster destination.

**Fix**
Stop further promotion and deploy the verified production artifact/configuration.

**Prevention**
Environment promotion controls, protected branches, explicit cluster labels, and artifact provenance.

## 88. Failure Scenario — Argo CD deploys to wrong cluster

**Symptom:** Correct manifests appear in an unintended EKS cluster.

**Investigation**
Inspect Argo CD Application destination server/name, ApplicationSet generators, project restrictions, and repository path.

**Fix**
Correct destination configuration and immediately assess unintended changes.

**Prevention**
Argo CD Projects with destination restrictions and separate environment ownership.

## 89. Failure Scenario — Multi-cluster cluster registration failure

**Symptom:** Central Argo CD cannot synchronize a managed EKS cluster.

**Investigation**
Check cluster registration, credentials, network reachability, IAM authentication, and Argo CD controller logs.

**Fix**
Restore the supported cluster credential/access path.

**Prevention**
Document cluster onboarding, credential rotation, and cluster-health alerts.

## 90. Failure Scenario — Production node drain causes outage

**Symptom:** Maintenance causes customer impact.

**Investigation**
Check replica counts, PDBs, topology spread, and whether replicas were actually distributed across nodes/AZs.

**Root causes**
- Single replica.
- PDB ineffective because only one replica exists.
- Multiple replicas on one node.
- Insufficient spare capacity.

**Fix**
Restore replicas and distribute workloads correctly.

**Prevention**
Topology spread constraints, minimum replicas, PDBs, and maintenance rehearsals.

## 91. Failure Scenario — Resource requests far above actual usage

**Symptom:** Cluster reports insufficient capacity despite low real CPU/memory utilization.

**Root cause**
Requests are oversized, causing scheduler fragmentation.

**Fix**
Measure actual usage over representative periods and right-size requests while retaining appropriate safety margins.

**Prevention**
Quarterly rightsizing and resource-efficiency dashboards.

## 92. Failure Scenario — Resource limits too low

**Symptom:** CPU throttling or OOM kills occur under normal traffic.

**Investigation**
Compare usage against limits and requests; correlate with latency and restart metrics.

**Fix**
Right-size limits and investigate application behavior.

**Prevention**
Load testing and workload resource baselines.

## 93. Failure Scenario — Application cannot reach external API

**Symptom:** External API calls time out.

**Investigation**
Check DNS, route table, NAT gateway, security groups, NetworkPolicy, proxy configuration, and external endpoint health.

**Root causes**
- No NAT route.
- Egress blocked.
- DNS failure.
- External provider outage.

**Fix**
Restore the required network path or use the provider's failover/degraded mode.

**Prevention**
External dependency SLOs and synthetic checks.

## 94. Failure Scenario — NAT gateway saturation/cost incident

**Symptom:** Private workloads experience egress latency while NAT processing/cost spikes.

**Investigation**
Inspect NAT metrics, connection counts, bytes processed, route distribution, and workload egress volume.

**Fix**
Reduce unnecessary egress, use VPC endpoints for AWS services where appropriate, and distribute NAT architecture as required.

**Prevention**
Egress monitoring and architecture reviews.

## 95. Failure Scenario — Kubernetes events disappear during incident

**Symptom:** Useful event history is unavailable when investigating after the fact.

**Root cause**
Kubernetes events are short-lived and were not centralized.

**Fix**
Use logs/metrics/AWS telemetry and reconstruct the timeline from multiple sources.

**Prevention**
Centralize relevant events and retain incident telemetry appropriately.

## 96. Failure Scenario — Incident timeline is unclear

**Symptom:** Teams disagree about when the incident started or what changed first.

**Fix**
Build a timeline from Prometheus alerts, ALB metrics, Argo CD sync history, Git commits, CI runs, AWS CloudTrail/events, and ELK logs.

**Prevention**
UTC timestamps, structured deployment annotations, and consistent incident notes.

## 97. Failure Scenario — False positive availability alert

**Symptom:** Alert fires while customers are unaffected.

**Root cause**
Alert watches an internal component rather than user-visible availability.

**Fix**
Separate symptom alerts from SLO alerts and use synthetic/business-transaction checks for critical user journeys.

**Prevention**
Alert review based on customer impact.

## 98. Failure Scenario — SLO alert fires during expected traffic surge

**Symptom:** Error/latency alert fires during planned load.

**Investigation**
Compare SLO burn rate with traffic, capacity, and planned events.

**Fix**
Do not simply disable the alert. Validate whether capacity planning was adequate.

**Prevention**
Load testing, event forecasting, and capacity headroom.

## 99. Failure Scenario — Deployment alert not correlated with release

**Symptom:** Operators cannot quickly identify that a release caused the incident.

**Fix**
Include version/image/commit labels in metrics and deployment events without creating unbounded cardinality. Correlate Argo CD revision and deployment timestamps.

**Prevention**
Release annotations and standardized deployment metadata.

## 100. Failure Scenario — Critical alert silenced accidentally

**Symptom:** Production incident occurs but no page is received.

**Investigation**
Review Alertmanager silences, creator, expiration, matcher, and incident/change context.

**Fix**
Remove an incorrect silence and restore notification. Determine why it was created.

**Prevention**
Short-lived silences, required reason/owner, and silence audits.

## 101. Failure Scenario — Prometheus unavailable but applications healthy

**Symptom:** Applications work but monitoring data stops.

**Risk**
This is an observability incident and can become a production reliability problem because operators lose detection.

**Fix**
Restore Prometheus while using ALB/application logs and AWS metrics as temporary signals.

**Prevention**
Prometheus HA, persistent storage, resource headroom, and independent monitoring of the monitoring stack.

## 102. Failure Scenario — Complete EKS cluster failure

**Symptom:** Cluster control or workloads are unavailable.

**Response**
Activate the documented DR plan. If a secondary cluster exists, use the GitOps repository and Terraform infrastructure definition to recreate or promote capacity.

**Recovery sequence**
1. Confirm AWS regional/cluster scope.
2. Protect the Git repositories and state.
3. Provision/recover infrastructure.
4. Establish EKS access.
5. Install platform components.
6. Register cluster with Argo CD.
7. Sync platform and application workloads.
8. Restore secrets/configuration.
9. Restore data dependencies.
10. Validate ALB/DNS and SLOs.

**Prevention**
Regular DR drills and measured RTO/RPO.

## 103. Failure Scenario — AWS region outage

**Symptom:** Primary region is unavailable or materially impaired.

**Response**
This is a DR event, not a normal Kubernetes restart incident.

**Investigation**
Confirm regional scope using AWS service health and independent application checks.

**Recovery**
Promote the secondary-region infrastructure/application/data according to the tested DR plan.

**Prevention**
Multi-region architecture only where business RTO/RPO justify its operational and financial complexity.

## Production Failure Triage Command Sheet

### Kubernetes
```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl get events -A --sort-by=.lastTimestamp
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
kubectl get deploy,rs,svc,ingress -n <namespace>
kubectl get endpointslice -n <namespace>
kubectl top pods -A
kubectl top nodes
```

### EKS/AWS
```bash
aws sts get-caller-identity
aws eks describe-cluster --name <cluster> --region <region>
aws eks list-nodegroups --cluster-name <cluster> --region <region>
aws cloudwatch list-metrics --namespace AWS/ApplicationELB
```

### Terraform
```bash
terraform fmt -check
terraform validate
terraform plan
terraform state list
```

### Helm
```bash
helm list -A
helm status <release> -n <namespace>
helm history <release> -n <namespace>
helm get values <release> -n <namespace>
helm get manifest <release> -n <namespace>
```

### Argo CD
```bash
argocd app list
argocd app get <app>
argocd app diff <app>
argocd app history <app>
```

### Prometheus
Use the Prometheus expression browser to test the exact alert expression. Verify:
- metric exists
- labels match
- time window is correct
- result is not empty
- recording rules are loaded
- target is being scraped

### ELK
Search by:
- timestamp
- namespace
- service
- pod
- container
- deployment version
- HTTP status
- request path
- correlation/request identifier where safe

## Senior-Level Failure Analysis Checklist

Before declaring an incident resolved, answer:

- What was the customer-visible symptom?
- Which SLI changed first?
- What dependency failed first?
- What evidence proves the root cause?
- Was there a recent deployment or infrastructure change?
- What was the blast radius?
- What mitigation restored service?
- Was the mitigation reversible?
- Did monitoring detect the incident quickly?
- Did alerts reach the correct on-call?
- Did logs and metrics provide enough evidence?
- Did the recovery preserve data integrity?
- Did the rollback include configuration/schema dependencies?
- What permanent prevention is required?
- Which runbook should be updated?
- Which alert/dashboard/test should be added?

## Failure Prevention Matrix

| Failure class | Primary detection | Immediate mitigation | Long-term prevention |
|---|---|---|---|
| Bad release | Error/latency SLO | GitOps rollback | Progressive delivery |
| Pod crash | Restart/OOM alerts | Replace/rollback | Resource and code fixes |
| Node failure | NodeReady alert | Reschedule/drain | Multi-AZ + automation |
| ALB failure | ALB 5xx/target health | Restore targets | Health-check testing |
| DNS failure | DNS synthetic checks | Restore resolver | HA CoreDNS/DNS |
| Capacity | Saturation/Pending pods | Scale out | Forecasting |
| EKS failure | Cluster telemetry | DR | Tested recovery |
| Database failure | Dependency SLO | Failover/recovery | HA + backups |
| Logging failure | Ingestion SLO | Restore pipeline | Capacity/ILM |
| Alerting failure | Dead-man switch | Alternate escalation | HA Alertmanager |
| Security event | SIEM/security alerts | Contain/rotate | Least privilege |
| Terraform drift | Scheduled plan | Reconcile | Git-only infrastructure |

## Key Production Principles

1. **Customer impact outranks component status.** A red dashboard is not automatically an outage, and a green component is not proof of user success.
2. **Use evidence before action.** Capture timestamps, logs, metrics, events, and versions.
3. **Mitigate before optimizing.** Restore service safely, then investigate deeply.
4. **Git is the source of truth for GitOps-managed state.**
5. **Never expose credentials during troubleshooting.**
6. **Do not use destructive commands as exploration commands.**
7. **Treat monitoring as production infrastructure.**
8. **Every major alert must have an owner and runbook.**
9. **Every rollback must consider configuration and database compatibility.**
10. **A repeated incident requires engineering prevention, not repeated manual recovery.**

## Interview Questions and Strong Production Answers

### Q1. A production service is returning 503. What do you check first?
I first establish whether the 503 is generated by the ALB, ingress/controller, service, or application. I check ALB target health, Service endpoints, pod readiness, recent deployments, and application logs. I avoid restarting pods until I understand whether there are healthy replicas and whether the failure is systemic.

### Q2. How do you troubleshoot CrashLoopBackOff?
I inspect `describe pod`, current logs, and especially `logs --previous`. I identify the termination reason and exit code, then check configuration, secrets, command/args, dependency availability, and resource limits. If the incident correlates with a release, I use the GitOps rollback process.

### Q3. How would you distinguish an application problem from an ALB problem?
I compare ALB metrics with application metrics. If ALB target health is bad and there are no healthy endpoints, I follow the Kubernetes path. If targets are healthy but application 5xx rises, I investigate the application and dependencies. I also test the service internally to isolate the layer.

### Q4. Why is restarting everything a bad incident response?
It destroys evidence, increases load, can restart healthy components, and may turn a localized failure into a broader outage. I prefer a targeted mitigation based on evidence.

### Q5. How do you troubleshoot an alert storm?
I group alerts by common labels and identify the root dependency. I use Alertmanager grouping and inhibition to suppress derivative symptoms, then tune thresholds and `for` durations where necessary. Critical root-cause alerts remain actionable.

### Q6. Argo CD says Synced but the application is unhealthy. Why?
Synced describes desired-state reconciliation, not application health. I inspect workload health, pods, services, ingress, events, and dependencies.

### Q7. What if rollback does not restore the application?
I verify whether the failure includes configuration, database schema, secrets, infrastructure, or external dependencies. Rolling back only the image cannot undo a destructive database migration or unrelated infrastructure change.

### Q8. How do you handle an OOMKilled pod?
I verify the termination reason, compare memory usage with requests/limits, inspect node pressure, and determine whether the issue is workload sizing or a leak. I may scale out for immediate availability, but I also fix the underlying memory behavior.

### Q9. How do you troubleshoot a Pending pod?
I read scheduler events from `kubectl describe pod`. I check CPU/memory capacity, taints/tolerations, affinity, topology constraints, quotas, and PVC binding. I do not assume that adding nodes is always the correct answer.

### Q10. What is your approach to a complete EKS outage?
I activate the tested DR procedure. I protect Git, Terraform state, artifacts, and data backups, recover infrastructure and the cluster, restore platform services, register the cluster with Argo CD, synchronize workloads, restore data, and validate end-to-end SLOs. The exact sequence is determined by measured RTO/RPO rather than improvisation.

## Production-Safe Failure Testing Patterns

Failure testing should be controlled, authorized, observable, and reversible.

### Example: simulate a deployment failure in a non-production environment
```bash
kubectl -n roboshop set image deployment/catalog   catalog=<ecr-repository>@<known-test-digest>

kubectl -n roboshop rollout status deployment/catalog --timeout=120s
kubectl -n roboshop get pods
kubectl -n roboshop get events --sort-by=.lastTimestamp
```

### Example: validate rollback
```bash
kubectl -n roboshop rollout history deployment/catalog
kubectl -n roboshop rollout undo deployment/catalog
kubectl -n roboshop rollout status deployment/catalog --timeout=120s
```

### Example: validate alert rule syntax
```bash
promtool check rules alerts/*.yaml
```

### Example: validate Kubernetes manifests
```bash
kubectl apply --dry-run=server -f manifests/
```

Do not perform destructive node termination, database deletion, credential revocation, or region-failure tests in production unless they are explicitly covered by an approved resilience-testing program.

## Final Appendix — Symptom → Investigation → Root Cause → Fix → Prevention Template

Use this structure for every real production incident.

### Symptom
State exactly what the customer/operator observed:
- HTTP status
- latency
- affected service
- start time
- environment
- scope

### Investigation
Record commands, dashboards, logs, metrics, AWS evidence, Git/Argo CD revisions, and timestamps.

### Root Cause
State the causal chain, not merely the final failed component.

Bad:
> Pod crashed.

Good:
> Release 2026.08.31 introduced an invalid database connection configuration. Pods failed startup, readiness never became healthy, the Service had no ready endpoints, ALB target health fell to zero, and customer requests returned 503.

### Fix
Document the mitigation and permanent correction separately.

### Prevention
At least one of:
- alert
- dashboard
- test
- automation
- capacity change
- security control
- runbook
- architecture change
- code fix

## Completion Standard

A senior engineer should be able to take any scenario in this chapter and answer:

1. What is the first safe command?
2. What evidence distinguishes the major hypotheses?
3. What is the blast radius?
4. What is the fastest safe mitigation?
5. What should not be done?
6. How is the root cause proven?
7. How is the system validated after recovery?
8. What alert should detect recurrence?
9. What permanent engineering change prevents recurrence?
10. How would the incident be explained to an interviewer or incident-review audience?

This is the operational standard for the remaining Production Capstone chapters.
