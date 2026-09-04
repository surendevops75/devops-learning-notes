# Capstone Interview Questions

## Senior DevOps / DevSecOps Production Interview Preparation

---

# 1. Purpose of This Chapter

This chapter converts the complete production capstone into senior-level DevOps and DevSecOps interview preparation.

The goal is not to memorize definitions.

The goal is to explain:

- architecture decisions
- production implementation
- troubleshooting
- security
- scalability
- availability
- CI/CD
- GitOps
- Kubernetes
- AWS
- Terraform
- observability
- disaster recovery
- incident management
- cost optimization
- operational trade-offs

A strong senior interview answer should explain:

```text
What
Why
How
Production example
Failure scenario
Troubleshooting
Trade-off
```

---

# 2. How to Answer Senior DevOps Questions

Use this structure:

```text
1. Define the concept
2. Explain why it matters
3. Explain production implementation
4. Give an example
5. Explain failure handling
6. Mention security/reliability
7. Mention trade-offs
```

Example:

> We use Argo CD for GitOps because it continuously reconciles Kubernetes with the desired state stored in Git. CI builds and validates the artifact, updates the GitOps configuration, and Argo CD deploys the change. This provides auditability, drift detection and controlled rollback. If Argo CD is temporarily unavailable, existing workloads continue running, but new desired-state changes cannot be reconciled.

This is stronger than:

> Argo CD is a GitOps tool.

---

# 3. Project Introduction

## Question 1: Explain your production DevOps project.

### Strong Answer

> I worked on a production-style microservices platform running on AWS EKS. The infrastructure was provisioned using Terraform across a multi-AZ VPC. Container images were built and validated through CI, including tests, SonarQube, Trivy and Veracode checks, and then stored in Amazon ECR. Application deployment configuration was maintained in a GitOps repository. Argo CD continuously reconciled the desired state from Git with the EKS clusters. AWS ALB provided external HTTP/HTTPS ingress. Helm was used for application packaging. Prometheus and Grafana handled metrics and visualization, while ELK provided centralized logging and Alertmanager handled alert routing. The platform also included autoscaling, security controls, rollback, backup, disaster recovery and production troubleshooting procedures.

---

# 4. Architecture Explanation

## Question 2: Explain the complete architecture.

### Answer

```text
Developer
   |
   v
Git Application Repository
   |
   v
CI Pipeline
   |
   +--> Build
   +--> Unit Tests
   +--> SonarQube
   +--> Trivy
   +--> Veracode
   |
   v
Container Image
   |
   v
ECR
   |
   v
GitOps Repository
   |
   v
Argo CD
   |
   v
EKS
   |
   +--> ALB Ingress
   +--> Services
   +--> Pods
   |
   +--> Prometheus
   +--> Grafana
   +--> ELK
   |
   v
Alertmanager
   |
   v
On-call / Operations
```

Terraform provisions the underlying AWS infrastructure.

Helm packages the applications.

GitOps controls deployment state.

Argo CD performs reconciliation.

---

# 5. AWS Questions

## Question 3: Why did you choose AWS?

### Answer

> AWS provides mature managed services for networking, identity, container orchestration, artifact storage and load balancing. EKS reduces Kubernetes control-plane management, ECR integrates with the container platform, ALB provides managed Layer 7 ingress, and IAM provides fine-grained access control.

---

## Question 4: Explain your AWS VPC architecture.

### Answer

> I use a multi-AZ VPC with public and private subnets. Public subnets host internet-facing components such as the ALB. EKS worker nodes are placed in private subnets. Routing, NAT and security controls are designed so workloads can reach required AWS services without exposing worker nodes directly to the internet.

---

## Question 5: Why private subnets for EKS worker nodes?

### Answer

> Worker nodes should not need direct inbound internet access. Keeping them private reduces the attack surface. External traffic enters through the managed ALB and is routed to Kubernetes workloads.

---

# 6. Availability Zones

## Question 6: Why do you use multiple Availability Zones?

### Answer

> A single-AZ architecture has a large failure domain. If that AZ becomes unavailable, the application can go down. Distributing worker nodes, replicas and supporting components across multiple AZs allows the application to continue operating when one AZ fails.

---

## Question 7: What happens if one AZ fails?

### Answer

> The ALB can continue routing traffic to healthy targets. Kubernetes workloads running in other AZs continue serving requests. If capacity is configured correctly, node autoscaling can replace lost capacity. I would immediately verify node health, pod distribution, ALB target health, pending pods and application error rates.

---

# 7. IAM

## Question 8: How do you implement IAM securely?

### Answer

> I follow least privilege. Different identities have different responsibilities for Terraform, CI, Argo CD and application workloads. I avoid giving administrator privileges by default and use workload-specific IAM permissions where possible.

---

## Question 9: Why is AdministratorAccess dangerous?

### Answer

> It creates excessive blast radius. If credentials are compromised or a pipeline is misconfigured, the attacker or automation could modify or delete a large portion of the environment. Least privilege limits the impact.

---

# 8. Terraform

## Question 10: Why Terraform?

### Answer

> Terraform provides declarative infrastructure as code. It allows infrastructure changes to be reviewed, version-controlled and reproduced consistently across environments.

---

## Question 11: How do you structure Terraform?

### Answer

```text
terraform/
|
+-- modules/
|   +-- vpc/
|   +-- eks/
|   +-- iam/
|   +-- ecr/
|
+-- environments/
    +-- dev/
    +-- qa/
    +-- prod/
```

Reusable modules contain common infrastructure logic while environment configurations define environment-specific values.

---

## Question 12: Where do you store Terraform state?

### Answer

> I use a remote backend with encryption, access control and state locking. I never commit Terraform state to Git because state can contain sensitive infrastructure information.

---

## Question 13: What happens if Terraform state is lost?

### Answer

> Terraform loses its knowledge of resource-to-state mappings. Recovery depends on the backend's versioning and backup capabilities. I would recover the state first rather than blindly recreating infrastructure.

---

## Question 14: What is Terraform drift?

### Answer

> Drift occurs when infrastructure changes outside Terraform differ from the declared configuration. Terraform plan helps identify the difference. In production, manual changes should be minimized and legitimate changes should be returned to code.

---

# 9. EKS

## Question 15: Why EKS instead of self-managed Kubernetes?

### Answer

> EKS provides a managed Kubernetes control plane and strong integration with AWS networking, IAM and load balancing. This allows the team to focus more on workloads and platform operations instead of managing the Kubernetes control-plane infrastructure ourselves.

---

## Question 16: What does EKS manage and what do you manage?

### Answer

> AWS manages the EKS control plane infrastructure. We still manage workload configuration, worker capacity, Kubernetes resources, security, networking configuration, IAM integration, observability and application reliability.

---

# 10. Kubernetes

## Question 17: Explain Pod, Deployment and Service.

### Answer

A Pod is the smallest deployable Kubernetes unit.

A Deployment manages replicated Pods and rolling updates.

A Service provides stable networking to a group of Pods.

```text
Deployment
    |
    v
Pods
    |
    v
Service
```

---

## Question 18: Why use Deployments?

### Answer

> Deployments provide declarative application management, replica management and controlled rolling updates. They also maintain ReplicaSets so Kubernetes can transition between application versions.

---

## Question 19: What is a Service?

### Answer

> A Service provides a stable virtual endpoint for a changing set of Pods. Pods can be replaced while clients continue communicating through the Service.

---

# 11. Kubernetes Probes

## Question 20: Difference between readiness and liveness probes?

### Answer

> Readiness determines whether the Pod should receive traffic. Liveness determines whether Kubernetes should restart the container because it is considered unhealthy.

---

## Question 21: Why can an aggressive liveness probe be dangerous?

### Answer

> If the application is slow during startup or under temporary load, an overly aggressive liveness probe may repeatedly restart healthy processes. This can create a restart loop and make the outage worse.

---

# 12. Kubernetes Resources

## Question 22: Why define CPU and memory requests?

### Answer

> Requests influence scheduling and communicate expected resource consumption to Kubernetes. Without requests, scheduling and capacity planning become less predictable.

---

## Question 23: What happens when a container exceeds its memory limit?

### Answer

> Kubernetes may terminate the container due to memory exhaustion, commonly resulting in an OOMKilled state. I would inspect memory usage, limits, application behavior and recent traffic before simply increasing the limit.

---

# 13. HPA

## Question 24: Explain HPA.

### Answer

> Horizontal Pod Autoscaler adjusts the number of Pod replicas based on metrics such as CPU, memory or configured custom metrics.

```text
Load
 |
 v
Metric
 |
 v
HPA
 |
 v
Replica count
```

---

## Question 25: Why can HPA fail to solve an outage?

### Answer

> HPA can request more Pods, but if the cluster has no available node capacity, those Pods remain pending. Therefore application autoscaling and node capacity must be designed together.

---

# 14. ALB Ingress

## Question 26: Explain ALB Ingress.

### Answer

> AWS ALB provides Layer 7 HTTP/HTTPS load balancing. Kubernetes Ingress resources define routing rules, and the AWS load balancing integration provisions and configures the ALB.

---

## Question 27: Why ALB instead of API Gateway?

### Answer

> The primary requirement is Kubernetes application ingress. ALB provides HTTP/HTTPS routing, TLS termination and Kubernetes integration without adding an unnecessary API management layer.

---

## Question 28: How would you troubleshoot an ALB 502?

### Answer

```bash
kubectl get ingress -A
kubectl describe ingress <ingress> -n <namespace>
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>
kubectl get pods -n <namespace>
```

Then inspect:

- target health
- Service port
- targetPort
- readiness
- application listener
- security groups
- logs

---

# 15. Helm

## Question 29: Why Helm?

### Answer

> Helm packages Kubernetes resources into reusable charts and allows environment-specific values. This reduces duplication and makes application deployment more consistent.

---

## Question 30: What is the difference between Chart and values?

### Answer

The chart contains reusable templates.

Values provide environment-specific configuration.

```text
Chart
 |
 +-- templates
 |
 +-- defaults

values-prod.yaml
 |
 +-- replicas
 +-- image
 +-- resources
 +-- ingress
```

---

# 16. GitOps

## Question 31: Explain GitOps.

### Answer

> GitOps treats Git as the desired-state source for deployment configuration. CI creates a validated artifact and updates the GitOps repository. Argo CD detects the change and reconciles Kubernetes to that desired state.

---

## Question 32: Why not let Jenkins run kubectl directly?

### Answer

> Direct CI deployment creates tighter coupling between the CI system and production clusters. GitOps separates artifact creation from deployment reconciliation, provides an audit trail and allows drift detection.

---

# 17. Argo CD

## Question 33: What does Argo CD do?

### Answer

> Argo CD continuously compares Git desired state with Kubernetes live state and reconciles differences.

```text
Git
 |
 v
Desired state
 |
 v
Argo CD
 |
 v
Kubernetes
```

---

## Question 34: What is OutOfSync?

### Answer

> OutOfSync means the live Kubernetes state differs from the desired state represented by Git.

Possible causes:

- manual changes
- failed sync
- incorrect configuration
- generated field differences
- deployment failure

---

## Question 35: What would you do if Argo CD shows OutOfSync?

### Answer

> First I determine whether the difference is intentional. I compare Git with the live resource, inspect the diff, review recent commits and check Argo CD sync status. If the manual change is not intended, I restore the desired configuration through Git rather than creating another manual patch.

---

# 18. Multi-Cluster Argo CD

## Question 36: Can one Argo CD manage multiple EKS clusters?

### Answer

> Yes. A centralized Argo CD control plane can manage multiple registered Kubernetes clusters, such as development, QA, production and DR environments.

---

## Question 37: What is the risk of centralized Argo CD?

### Answer

> Argo CD becomes an important management-plane dependency. Existing workloads can continue running if Argo CD is unavailable, but new desired-state changes cannot be reconciled. Therefore production Argo CD should have HA, secured cluster credentials and a recovery plan.

---

# 19. CI/CD

## Question 38: Explain your CI pipeline.

### Answer

```text
Checkout
   |
Build
   |
Unit Test
   |
SonarQube
   |
Security Scan
   |
Docker Build
   |
Trivy
   |
Veracode
   |
Push ECR
   |
Update GitOps
```

Each stage should have a clear quality or security purpose.

---

## Question 39: How do you prevent vulnerable images from reaching production?

### Answer

> I implement security gates in CI. The pipeline scans dependencies and images and evaluates severity thresholds. Failed security gates prevent the artifact from progressing to the production deployment process.

---

# 20. Immutable Images

## Question 40: Why avoid latest?

### Answer

> `latest` does not uniquely identify a release. If the same tag points to different image contents, reproducibility becomes difficult. I prefer immutable tags based on commit SHA or image digests.

---

# 21. ECR

## Question 41: Why ECR?

### Answer

> ECR integrates naturally with AWS and EKS, provides managed private image storage and supports image scanning and lifecycle management.

---

## Question 42: What happens if ECR is temporarily unavailable?

### Answer

> Existing running Pods may continue operating because they already have their images locally. New Pods that need to pull unavailable images may fail. I would inspect image pull errors, registry health, node connectivity and IAM permissions.

---

# 22. Security Tools

## Question 43: Why SonarQube, Trivy and Veracode?

### Answer

> They provide complementary coverage. SonarQube focuses heavily on code quality and static analysis, Trivy provides container and dependency/configuration scanning, and Veracode provides additional application security analysis. The objective is layered security rather than simply increasing the number of tools.

---

# 23. Secrets

## Question 44: How do you manage production secrets?

### Answer

> Production credentials should not be committed as plaintext to Git. I prefer a managed secret solution such as AWS Secrets Manager with controlled access and secure integration into workloads. Kubernetes Secrets should also be treated as sensitive data and protected accordingly.

---

## Question 45: What would you do if a password was accidentally committed?

### Answer

> I would immediately rotate or revoke the credential. Then I would remove the secret from the repository history using an approved secret-remediation process, investigate access logs, determine whether the credential was used, and implement secret scanning to prevent recurrence. Removing the text from the latest commit alone is not sufficient because the credential may remain in Git history.

---

# 24. Kubernetes Security

## Question 46: How do you secure Kubernetes?

### Answer

Use layered controls:

- RBAC
- network policies
- pod security controls
- least privilege
- image scanning
- secrets management
- admission policies where appropriate
- audit logging
- resource controls
- restricted production access

---

# 25. Prometheus

## Question 47: Why Prometheus?

### Answer

> Prometheus provides time-series metric collection and PromQL, and it integrates well with Kubernetes. It can evaluate alert rules and expose metrics to Grafana.

---

## Question 48: What is PromQL?

### Answer

> PromQL is Prometheus's query language for selecting, aggregating and calculating time-series metrics.

Example:

```promql
rate(http_requests_total[5m])
```

This calculates the per-second rate over the previous five minutes.

---

# 26. Golden Signals

## Question 49: What are the four golden signals?

### Answer

```text
Latency
Traffic
Errors
Saturation
```

These provide a user-oriented view of service health.

---

# 27. SLI/SLO/SLA

## Question 50: Explain SLI, SLO and SLA.

### Answer

SLI:

> The measured indicator.

SLO:

> The target for the indicator.

SLA:

> A formal service commitment, often with business consequences.

Example:

```text
SLI = successful request percentage
SLO = 99.9%
SLA = contractual availability commitment
```

---

# 28. Alerting

## Question 51: What makes a good production alert?

### Answer

A good alert should identify:

- what failed
- severity
- environment
- service
- owner
- impact
- runbook
- recommended response

Example:

```yaml
labels:
  severity: critical
  environment: prod
  team: checkout
  service: checkout
```

---

## Question 52: How do you prevent alert fatigue?

### Answer

> I use appropriate thresholds, `for` durations, SLO-based alerting, grouping, inhibition, deduplication, ownership and runbooks. I continuously review alert frequency and remove alerts that are not actionable.

---

## Question 53: What is alert flapping?

### Answer

> Alert flapping occurs when an alert repeatedly changes between firing and resolved states because the condition is near the threshold. I reduce it with appropriate thresholds, evaluation windows, hysteresis-like logic where appropriate, and `for` durations.

---

# 29. Alertmanager

## Question 54: What does Alertmanager do?

### Answer

> Alertmanager receives alerts and manages grouping, routing, deduplication, silences and inhibition before sending notifications to configured receivers.

---

## Question 55: What is inhibition?

### Answer

> Inhibition suppresses lower-level alerts when a higher-level alert indicates a known root cause.

For example:

```text
NodeDown
 |
 +--> suppress PodDown
 +--> suppress NodeCPU
 +--> suppress NodeMemory
```

This reduces alert storms.

---

# 30. Recording Rules

## Question 56: Why use recording rules?

### Answer

> Recording rules precompute frequently used PromQL expressions. They reduce query cost and make dashboards and alerts more efficient, especially for expensive aggregations.

---

# 31. ELK

## Question 57: Why ELK?

### Answer

> ELK provides centralized log collection, indexing and search. It allows operators to investigate application and infrastructure events across multiple Kubernetes workloads.

---

## Question 58: Prometheus vs ELK?

### Answer

```text
Prometheus -> metrics
Grafana    -> visualization
ELK        -> logs
```

They solve different observability problems.

---

# 32. Kubernetes Troubleshooting

## Question 59: A Pod is CrashLoopBackOff. What do you do?

### Answer

```bash
kubectl get pod <pod> -n <namespace>
kubectl describe pod <pod> -n <namespace>
kubectl logs <pod> -n <namespace>
kubectl logs <pod> -n <namespace> --previous
```

Check:

- application exception
- configuration
- secret
- environment variables
- probes
- permissions
- image
- resource limits
- dependency failures

Do not restart blindly.

---

# 33. Pending Pod

## Question 60: A Pod is Pending. How do you troubleshoot?

### Answer

```bash
kubectl describe pod <pod> -n <namespace>
```

Look for:

- insufficient CPU
- insufficient memory
- taints
- affinity
- node selectors
- topology constraints
- PVC problems

Then inspect:

```bash
kubectl get nodes
kubectl describe node <node>
```

---

# 34. OOMKilled

## Question 61: A Pod is OOMKilled. What do you investigate?

### Answer

I check:

```bash
kubectl describe pod <pod> -n <namespace>
```

Then compare:

- memory request
- memory limit
- actual usage
- application heap
- traffic
- historical metrics
- recent deployment changes

The correct fix might be application optimization rather than simply increasing the memory limit.

---

# 35. Node NotReady

## Question 62: A node is NotReady. What do you check?

### Answer

```bash
kubectl get nodes
kubectl describe node <node>
```

Then inspect:

- kubelet
- container runtime
- node CPU
- memory
- disk pressure
- network
- instance health
- IAM
- CNI
- recent AWS events

---

# 36. Service Not Working

## Question 63: A Service has no traffic. What do you check?

### Answer

```bash
kubectl get svc -n <namespace>
kubectl get endpoints -n <namespace>
kubectl get endpointslice -n <namespace>
kubectl get pods -n <namespace> --show-labels
```

The most common issue is a selector mismatch or Pods failing readiness.

---

# 37. Deployment Not Updating

## Question 64: Deployment is stuck during rollout.

### Answer

```bash
kubectl rollout status deployment/<name> -n <namespace>
kubectl describe deployment <name> -n <namespace>
kubectl get rs -n <namespace>
kubectl get pods -n <namespace>
```

Then inspect:

- image pull
- readiness
- resource limits
- scheduling
- application startup
- admission policy
- events

---

# 38. ALB 504

## Question 65: What does an ALB 504 make you think about?

### Answer

A 504 generally points toward timeout or backend responsiveness problems.

I investigate:

- ALB timeout
- target health
- application response time
- Service routing
- Pod readiness
- downstream dependencies
- database latency
- network connectivity

I correlate ALB metrics with Prometheus and application logs.

---

# 39. DNS Troubleshooting

## Question 66: Application hostname does not resolve.

### Answer

Check:

```bash
dig app.example.com
nslookup app.example.com
```

Then validate:

- Route 53 record
- hosted zone
- DNS delegation
- ALB hostname
- TTL
- certificate hostname
- client DNS cache

---

# 40. TLS Troubleshooting

## Question 67: HTTPS returns certificate errors.

### Answer

Check:

```bash
openssl s_client -connect app.example.com:443 -servername app.example.com
```

Validate:

- certificate chain
- hostname
- expiration
- ALB listener
- ACM certificate association
- TLS policy
- DNS target

---

# 41. Terraform Failure

## Question 68: Terraform apply failed halfway. What do you do?

### Answer

> I inspect the exact failed resource and provider error, determine whether Terraform state reflects the resources that were successfully created, and run another plan after resolving the underlying issue. I avoid manually deleting resources unless necessary because that can create additional state drift.

---

# 42. Terraform Lock

## Question 69: Terraform says the state is locked.

### Answer

> First I determine whether another legitimate operation is running. I never force-unlock blindly. If the lock is genuinely stale, I follow the team's approved recovery process and verify state consistency before continuing.

---

# 43. Helm Failure

## Question 70: Helm upgrade failed. What do you do?

### Answer

```bash
helm status <release> -n <namespace>
helm history <release> -n <namespace>
helm get values <release> -n <namespace>
helm get manifest <release> -n <namespace>
```

Then inspect Kubernetes:

```bash
kubectl get pods -n <namespace>
kubectl describe pod <pod> -n <namespace>
```

If safe, roll back to a known-good revision.

---

# 44. GitOps Drift

## Question 71: Someone manually changed a production Deployment.

### Answer

> Argo CD should detect the drift. I first determine whether the change was authorized. If not, I restore the Git-defined desired state. The important lesson is to fix the source of truth instead of maintaining hidden manual changes.

---

# 45. Production Incident

## Question 72: Production error rate suddenly increases. What do you do?

### Answer

```text
1. Confirm the alert
2. Establish scope
3. Check recent deployments
4. Check application error rate
5. Check latency
6. Check ALB health
7. Check Pods
8. Check dependencies
9. Check infrastructure
10. Mitigate impact
11. Communicate
12. Preserve evidence
13. Identify root cause
14. Recover
15. Conduct post-incident review
```

---

# 46. First Five Minutes

## Question 73: What do you do during the first five minutes of an outage?

### Answer

> My priority is impact assessment and stabilization. I confirm whether the alert is real, determine affected services and environments, check recent changes, verify user-facing symptoms and identify whether rollback or traffic mitigation is appropriate. I communicate early rather than waiting for perfect root-cause certainty.

---

# 47. Rollback

## Question 74: How do you roll back a bad Kubernetes deployment?

### Answer

For a Deployment:

```bash
kubectl rollout history deployment/<name> -n <namespace>
kubectl rollout undo deployment/<name> -n <namespace>
```

In GitOps, the preferred long-term correction is usually a Git revert or restoration of the known-good desired state so Argo CD converges to the correct version.

---

# 48. Database Rollback

## Question 75: Why is database rollback harder?

### Answer

> Application versions can often be switched quickly, but database schema changes may not be backward compatible. I prefer backward-compatible migrations so old and new application versions can coexist during deployment and rollback.

---

# 49. Disaster Recovery

## Question 76: What is RPO?

### Answer

> Recovery Point Objective defines the maximum acceptable amount of data loss measured in time.

Example:

```text
RPO = 15 minutes
```

means losing more than approximately 15 minutes of data would violate the requirement.

---

## Question 77: What is RTO?

### Answer

> Recovery Time Objective defines the maximum acceptable time to restore service.

Example:

```text
RTO = 60 minutes
```

---

# 50. HA vs DR

## Question 78: Difference between HA and DR?

### Answer

> HA keeps services available during expected component failures, such as a node or AZ failure. DR addresses recovery from larger disasters such as regional failure, major data corruption or infrastructure loss.

---

# 51. Backup

## Question 79: What makes a backup strategy production-ready?

### Answer

It must define:

- what is backed up
- frequency
- retention
- encryption
- access control
- RPO
- restore procedure
- restore testing
- ownership

A backup that has never been restored is not proven.

---

# 52. Cost Optimization

## Question 80: How do you reduce AWS costs without damaging reliability?

### Answer

> I first identify the actual cost drivers. I review node utilization, autoscaling, NAT Gateway traffic, EBS volumes, ALB usage, ECR retention, logging volume, Elasticsearch storage, Prometheus cardinality and data transfer. I optimize waste before reducing availability or redundancy.

---

# 53. EKS Cost

## Question 81: How do you optimize EKS costs?

### Answer

- right-size nodes
- use autoscaling
- right-size requests
- eliminate idle workloads
- use appropriate instance families
- evaluate workload scheduling
- control observability costs
- review data transfer
- use appropriate purchase models where justified

---

# 54. Logging Cost

## Question 82: Why can ELK become expensive?

### Answer

> High log volume combined with long retention creates significant ingestion and storage costs. Verbose application logs and excessive indexing can amplify the problem.

---

# 55. Prometheus Cardinality

## Question 83: What is metric cardinality?

### Answer

> Cardinality is the number of unique time series generated by metric labels.

Bad:

```text
user_id
request_id
session_id
```

These can create enormous numbers of unique series.

Better:

```text
service
method
status
environment
```

---

# 56. Security Incident

## Question 84: A production credential leaked. What is your response?

### Answer

```text
1. Revoke/rotate credential
2. Identify affected systems
3. Review access logs
4. Determine exposure window
5. Contain access
6. Remove secret from source control
7. Search for related credentials
8. Notify security/incident owners
9. Implement prevention
```

---

# 57. NetworkPolicy

## Question 85: Why use Kubernetes NetworkPolicy?

### Answer

> NetworkPolicy restricts pod-to-pod traffic and limits lateral movement. Instead of allowing every workload to communicate with every other workload, we define only required communication paths.

---

# 58. Security Groups vs NetworkPolicy

## Question 86: What is the difference?

### Answer

> AWS security groups control network access at AWS networking interfaces and are stateful. Kubernetes NetworkPolicy controls Pod-level network communication according to the supported networking implementation. They operate at different layers and complement each other.

---

# 59. Alert Storm

## Question 87: What happens if a node goes down?

Possible alerts:

```text
NodeDown
PodDown
DeploymentUnavailable
TargetUnhealthy
HighErrorRate
HighLatency
```

Without inhibition and grouping, this can become an alert storm.

The correct design uses root-cause-aware routing.

---

# 60. SLO Alerting

## Question 88: Why use SLO-based alerts?

### Answer

> Resource metrics are useful, but user impact is more important. SLO-based alerting connects monitoring to service reliability. A service may run at 90% CPU without user impact, while a 2% error rate may represent a serious outage.

---

# 61. Deployment Alerts

## Question 89: What deployment metrics do you monitor?

Examples:

- rollout duration
- failed deployments
- unavailable replicas
- restart rate
- image pull failures
- deployment frequency
- rollback frequency
- Argo CD sync failures
- OutOfSync state

---

# 62. DORA Metrics

## Question 90: What are DORA metrics?

Common delivery metrics include:

- deployment frequency
- lead time for changes
- change failure rate
- time to restore service

They help evaluate software delivery performance.

---

# 63. Production Capacity

## Question 91: How do you perform capacity planning?

### Answer

I analyze:

- historical utilization
- peak traffic
- growth rate
- pod resource requests
- node utilization
- autoscaling behavior
- database capacity
- ALB throughput
- observability capacity

Then I define headroom and test scaling behavior.

---

# 64. Performance Incident

## Question 92: Application latency increased but CPU is normal. What do you investigate?

### Answer

I do not assume CPU is the bottleneck.

I check:

- database latency
- downstream services
- network latency
- connection pools
- thread pools
- GC
- external APIs
- ALB metrics
- application logs
- request patterns

Normal CPU does not mean healthy application performance.

---

# 65. Memory Leak

## Question 93: How would you identify a memory leak?

### Answer

I look for:

```text
Memory usage
   |
   +--> continuously increasing
   |
   +--> GC behavior
   |
   +--> heap usage
   |
   +--> restart history
```

Then correlate with application release versions and traffic.

---

# 66. Disk Full

## Question 94: Node disk is full. What do you do?

### Answer

Check:

```bash
df -h
df -i
du -xhd1 /
```

Then inspect:

- container logs
- image layers
- unused images
- container runtime storage
- temporary files
- application-generated files

Do not delete arbitrary system files.

---

# 67. Kubernetes Events

## Question 95: Why are Kubernetes events useful?

### Answer

Events provide contextual information about scheduling, image pulls, probes, mounts, evictions and other cluster actions.

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

They are often one of the fastest ways to identify immediate Kubernetes failures.

---

# 68. Production Debugging Philosophy

## Question 96: What is your troubleshooting approach?

### Answer

> I start from the symptom and establish scope. Then I follow the dependency chain from user to DNS, ALB, ingress, Service, Pod, application and dependencies. I correlate metrics, logs and events with the incident timeline. I avoid making multiple changes simultaneously because that destroys evidence and makes root-cause analysis harder.

---

# 69. Observability Correlation

## Question 97: How do you correlate metrics and logs?

### Answer

> I first identify the affected service and time window using Prometheus and Grafana. Then I search ELK for application and infrastructure logs in the same time range. I correlate deployment timestamps, error spikes, latency changes and infrastructure events.

---

# 70. Grafana

## Question 98: What dashboards would you create?

Examples:

### Cluster

- node CPU
- node memory
- disk
- network
- Pod counts

### Application

- request rate
- error rate
- latency
- saturation

### Kubernetes

- restarts
- pending Pods
- unavailable replicas
- HPA behavior

### ALB

- request count
- 4xx
- 5xx
- target health
- latency

---

# 71. ELK Production Design

## Question 99: What should you consider when operating ELK in production?

### Answer

- node capacity
- shard strategy
- replicas
- storage
- retention
- index lifecycle
- ingestion throughput
- JVM memory
- disk watermarks
- backup
- access control

---

# 72. Elasticsearch Disk Watermark

## Question 100: What happens when Elasticsearch disk usage becomes too high?

### Answer

Elasticsearch uses disk watermarks to protect cluster health. High disk usage can cause shard allocation restrictions and eventually write problems. I would inspect disk usage, shard allocation, index growth and retention policies.

---

# 73. Multi-Environment Design

## Question 101: How do you separate dev, QA and prod?

### Answer

> I use separate configurations, namespaces or clusters according to isolation requirements. Production has stricter access, approval and security controls. Secrets are environment-specific and never copied casually between environments.

---

# 74. Production Approval

## Question 102: Should every production deployment require manual approval?

### Answer

> It depends on organizational risk and deployment maturity. High-risk environments may use approval gates. Lower-risk or highly automated organizations may use policy-driven continuous delivery. The important point is that production changes should have appropriate controls, auditability and rollback capability.

---

# 75. Zero Downtime

## Question 103: How do you achieve zero-downtime deployment?

### Answer

Use:

- multiple replicas
- readiness probes
- rolling updates
- PDBs
- sufficient capacity
- correct ALB health checks
- backward-compatible changes

---

# 76. PodDisruptionBudget

## Question 104: Why use PDB?

### Answer

> A PodDisruptionBudget limits how many replicas can be voluntarily disrupted simultaneously, helping maintain application availability during operations such as node maintenance.

---

# 77. Topology Spread

## Question 105: Why use topology spread constraints?

### Answer

> They help distribute replicas across failure domains such as nodes and AZs, reducing the risk of losing all replicas because of a localized failure.

---

# 78. Node Autoscaling

## Question 106: HPA vs node autoscaling?

### Answer

```text
HPA
 |
 +--> changes Pod replica count

Node autoscaling
 |
 +--> changes compute capacity
```

They solve different scaling layers.

---

# 79. Application Dependency

## Question 107: A downstream service is slow. How can you protect the platform?

### Answer

Use:

- timeouts
- retries with limits
- circuit-breaking patterns where appropriate
- connection pool controls
- graceful degradation
- caching where appropriate
- alerting

Retries must be carefully controlled because uncontrolled retries can amplify an outage.

---

# 80. Retry Storm

## Question 108: What is a retry storm?

### Answer

> A retry storm occurs when many clients repeatedly retry failed requests, increasing load on an already unhealthy dependency. This can turn a small failure into a cascading outage.

---

# 81. Cascading Failure

## Question 109: Explain cascading failure.

### Answer

```text
Database slows
   |
   v
Service requests wait
   |
   v
Connection pool exhausted
   |
   v
Request latency increases
   |
   v
Retries increase
   |
   v
More database load
```

The original problem becomes amplified by dependencies.

---

# 82. Incident Communication

## Question 110: What should an incident update contain?

Include:

```text
Impact
Current status
Affected services
Start time
Known facts
Mitigation
Next action
Owner
Next update time
```

Avoid speculation presented as fact.

---

# 83. Post-Incident Review

## Question 111: What belongs in a postmortem?

- incident summary
- impact
- timeline
- detection
- response
- root cause
- contributing factors
- resolution
- what went well
- what went poorly
- action items
- owners
- deadlines

The objective is learning and prevention, not blame.

---

# 84. Production Change

## Question 112: What makes a safe production change?

A safe change has:

- clear objective
- tested implementation
- defined risk
- monitoring
- rollback plan
- owner
- approval where required
- maintenance window where necessary
- post-change validation

---

# 85. Git Revert

## Question 113: Why is Git revert useful in GitOps?

### Answer

> Revert creates a new commit that restores the previous desired state. This preserves the audit trail and allows Argo CD to reconcile the cluster back to the known-good configuration.

---

# 86. Manual kubectl in Production

## Question 114: Is `kubectl` forbidden in production?

### Answer

> No. It remains an important troubleshooting and emergency tool. The concern is uncontrolled manual configuration changes. Routine desired-state changes should flow through GitOps, while emergency investigation and carefully governed remediation can use kubectl.

---

# 87. Emergency Change

## Question 115: What if GitOps is too slow during an emergency?

### Answer

> During a severe incident, an emergency manual mitigation may be justified if the organization's incident process permits it. After stabilization, the manual change must be captured in Git so the source of truth and actual state converge.

---

# 88. Security Hardening

## Question 116: How do you harden EKS?

### Answer

- private nodes
- least privilege IAM
- RBAC
- network policies
- image scanning
- secure secrets
- restricted API access
- audit logging
- regular patching
- Pod security controls
- controlled production access

---

# 89. Container Hardening

## Question 117: How do you harden container images?

### Answer

- minimal base image
- remove unnecessary packages
- scan dependencies
- scan image
- run as non-root
- read-only filesystem where possible
- drop unnecessary Linux capabilities
- do not embed secrets

---

# 90. CI Credential Security

## Question 118: How do you protect CI credentials?

### Answer

Use:

- secret stores
- short-lived credentials where possible
- least privilege
- masked logs
- restricted pipeline permissions
- credential rotation
- separate identities by environment

Never print credentials for debugging.

---

# 91. Supply Chain Security

## Question 119: How do you secure the software supply chain?

### Answer

I secure:

```text
Source
 |
Dependencies
 |
Build
 |
Tests
 |
Static analysis
 |
Image
 |
Registry
 |
GitOps
 |
Deployment
```

Controls include branch protection, dependency scanning, image scanning, signed or traceable artifacts where implemented, and restricted deployment permissions.

---

# 92. Branch Protection

## Question 120: Why protect production branches?

### Answer

> Branch protection prevents unauthorized direct changes and enforces review requirements. This is especially important for infrastructure and GitOps repositories because a Git change can become a production infrastructure or application change.

---

# 93. Infrastructure and Application Separation

## Question 121: Why separate Terraform and GitOps repositories?

### Answer

> They represent different lifecycle domains. Terraform manages infrastructure, while GitOps manages Kubernetes desired state. Separating them reduces coupling and clarifies ownership and permissions.

---

# 94. Helm and GitOps Separation

## Question 122: Why separate application source from deployment configuration?

### Answer

> Application source changes and deployment configuration changes have different responsibilities. The application repository produces artifacts, while the GitOps repository controls where and how those artifacts are deployed.

---

# 95. Production Architecture Trade-off

## Question 123: What is one important architecture trade-off?

### Answer

> GitOps improves auditability and deployment control but introduces another management layer. Argo CD must itself be secured, monitored and recoverable. Similarly, ELK gives powerful logging but has significant operational and storage cost.

A senior engineer should be able to explain both benefits and costs.

---

# 96. Single Point of Failure

## Question 124: How do you identify a single point of failure?

Ask:

```text
If this component disappears,
does the service continue?
```

Apply this to:

- node
- AZ
- load balancer
- database
- registry
- Git
- Argo CD
- Prometheus
- ELK
- DNS
- IAM dependency

---

# 97. Architecture Review

## Question 125: How would you review this architecture as a senior engineer?

I review:

```text
Requirements
 |
Availability
 |
Scalability
 |
Security
 |
Observability
 |
Deployment
 |
Failure handling
 |
DR
 |
Cost
 |
Operational maturity
```

Then identify:

- risks
- bottlenecks
- single points of failure
- operational gaps
- unnecessary complexity

---

# 98. What Would You Improve?

## Question 126: If you had more time, what would you improve?

### Strong Answer

> I would prioritize measurable improvements rather than simply adding tools. I would strengthen SLO-based alerting, automated restore testing, DR drills, workload identity, policy enforcement, metric cardinality controls, log lifecycle management, capacity planning and resilience testing.

---

# 99. Production Readiness

## Question 127: When do you call an application production-ready?

### Answer

Not simply when it deploys.

I require:

```text
Deployment
+
Security
+
Observability
+
HA
+
Scaling
+
Backup
+
Recovery
+
Rollback
+
Runbooks
+
Ownership
```

---

# 100. Final Senior-Level Questions

## Question 128: What is the difference between DevOps and simply knowing tools?

### Answer

> Tool knowledge tells me how to operate individual components. DevOps requires understanding how the complete delivery and runtime system behaves. A senior engineer must understand dependencies, failure modes, automation, security, reliability, cost and business impact.

---

## Question 129: What is the most important DevOps skill in production?

### Answer

> Systematic problem solving. Production incidents rarely respect tool boundaries. A failure may involve Linux, networking, Kubernetes, AWS, application code and databases simultaneously. The engineer must trace the system logically instead of assuming the problem belongs to one tool.

---

## Question 130: What is your production troubleshooting philosophy?

### Answer

> I do not start by restarting components. I first establish the symptom, scope and timeline. Then I follow the dependency chain and correlate metrics, logs and events. I make the smallest safe mitigation that reduces user impact, preserve evidence, identify root cause and implement prevention.

---

# 101. Rapid-Fire Round

## Question 131: What is ECR?

Private AWS container registry.

## Question 132: What is EKS?

Managed Kubernetes service from AWS.

## Question 133: What is Helm?

Kubernetes application packaging and templating system.

## Question 134: What is Argo CD?

GitOps continuous delivery and reconciliation controller.

## Question 135: What is Prometheus?

Time-series metrics and alert evaluation platform.

## Question 136: What is Grafana?

Metrics visualization and dashboard platform.

## Question 137: What is ELK?

Elasticsearch, Logstash and Kibana logging/search stack.

## Question 138: What is Alertmanager?

Prometheus alert routing and notification management component.

## Question 139: What is Terraform?

Infrastructure as code tool.

## Question 140: What is ALB?

AWS Application Load Balancer.

---

# 102. Command-Based Interview Round

## Question 141: How do you see all Pods?

```bash
kubectl get pods -A
```

## Question 142: How do you inspect a Pod?

```bash
kubectl describe pod <pod> -n <namespace>
```

## Question 143: How do you view logs?

```bash
kubectl logs <pod> -n <namespace>
```

## Question 144: How do you view previous container logs?

```bash
kubectl logs <pod> -n <namespace> --previous
```

## Question 145: How do you check nodes?

```bash
kubectl get nodes
```

## Question 146: How do you check cluster events?

```bash
kubectl get events -A --sort-by=.lastTimestamp
```

## Question 147: How do you check rollout?

```bash
kubectl rollout status deployment/<name> -n <namespace>
```

## Question 148: How do you roll back a Deployment?

```bash
kubectl rollout undo deployment/<name> -n <namespace>
```

## Question 149: How do you see Helm history?

```bash
helm history <release> -n <namespace>
```

## Question 150: How do you inspect Terraform changes?

```bash
terraform plan
```

---

# 103. Scenario: Production Deployment Increased Errors

## Question 151

A deployment completed successfully, but HTTP 5xx increased. What do you do?

### Answer

First:

```text
Confirm impact
```

Then:

```text
Deployment timeline
      |
      v
5xx increase
      |
      v
Application logs
      |
      v
Pod health
      |
      v
Dependencies
```

Check:

```bash
kubectl rollout status deployment/<name> -n <namespace>
kubectl get pods -n <namespace>
kubectl logs <pod> -n <namespace>
```

If the evidence clearly connects the deployment to the incident and rollback is safe, restore the previous known-good version.

Then investigate the defect after service recovery.

---

# 104. Scenario: All Pods Pending

## Question 152

### Answer

Check:

```bash
kubectl get pods -A
kubectl describe pod <pod> -n <namespace>
kubectl get nodes
```

Likely causes:

- node capacity
- taints
- affinity
- resource requests
- autoscaler failure

Then check cloud-side capacity and node autoscaling.

---

# 105. Scenario: ALB Healthy but Application Fails

## Question 153

### Answer

The ALB may only verify a shallow health endpoint.

I check:

- application internal dependencies
- business endpoint health
- Service routing
- Pod readiness
- database
- application logs

A basic `/health` response does not prove every dependency is healthy.

---

# 106. Scenario: Prometheus Has No Data

## Question 154

Check:

```text
Prometheus targets
ServiceMonitor
PodMonitor
RBAC
networking
metrics endpoint
scrape configuration
```

Then inspect Prometheus target health.

---

# 107. Scenario: Grafana Dashboard Is Empty

## Question 155

Check:

1. Grafana data source
2. Prometheus availability
3. query
4. time range
5. labels
6. metric names
7. dashboard variables

Do not assume Prometheus is broken.

---

# 108. Scenario: ELK Logs Missing

## Question 156

Trace:

```text
Pod
 |
container logs
 |
collector
 |
Logstash/ingestion
 |
Elasticsearch
 |
Kibana
```

Find the first broken link.

---

# 109. Scenario: Argo CD Sync Failed

## Question 157

Check:

```text
Application status
sync status
health status
repository access
manifest rendering
cluster permissions
Kubernetes events
```

Then identify whether the failure is Git, rendering, authorization or Kubernetes runtime related.

---

# 110. Scenario: ImagePullBackOff

## Question 158

Check:

```bash
kubectl describe pod <pod> -n <namespace>
```

Investigate:

- image name
- tag/digest
- ECR repository
- node IAM
- network access
- registry availability
- image architecture

---

# 111. Scenario: Service Works Internally but Not Externally

## Question 159

Test:

```text
Pod
 |
Service
 |
Ingress
 |
ALB
 |
DNS
 |
Internet
```

Validate each layer independently.

---

# 112. Scenario: CPU Is High

## Question 160

Do not immediately scale.

Check:

- traffic increase
- application behavior
- CPU throttling
- inefficient code
- recent deployment
- request/limit configuration
- HPA behavior

Then determine whether scaling or application remediation is appropriate.

---

# 113. Scenario: Memory Is High

## Question 161

Check:

- memory trend
- Pod limit
- JVM heap if Java
- garbage collection
- application release
- traffic
- memory leak indicators

---

# 114. Scenario: Disk Is Filling

## Question 162

Check:

```bash
df -h
df -i
```

Then identify the largest consumers and determine whether the cause is:

- logs
- container images
- application data
- temporary files
- filesystem issue

---

# 115. Scenario: Network Timeout

## Question 163

Investigate:

```text
DNS
 |
routing
 |
security group
 |
NACL
 |
NetworkPolicy
 |
Service
 |
Pod
 |
application
```

Use appropriate tools such as:

```bash
dig
curl
nc
ss
ip route
```

from an authorized diagnostic environment.

---

# 116. Scenario: IAM AccessDenied

## Question 164

Check:

- caller identity
- IAM policy
- resource policy
- trust policy
- SCP if applicable
- permission boundary
- role assumption
- region/resource ARN

Never assume an AccessDenied error is caused only by the attached identity policy.

---

# 117. Scenario: Terraform Wants to Recreate Production

## Question 165

Stop before applying.

Review:

```bash
terraform plan
```

Determine:

- why the diff exists
- state drift
- provider changes
- variable changes
- resource replacement requirement

Never approve a destructive plan without understanding it.

---

# 118. Scenario: Production Certificate Expires

## Question 166

Monitor certificate expiry proactively.

For an AWS-managed certificate, verify:

- certificate status
- DNS validation
- ALB listener association
- renewal status

An expired certificate becomes a customer-facing incident.

---

# 119. Scenario: Alertmanager Is Sending Too Many Alerts

## Question 167

Review:

- grouping
- group interval
- repeat interval
- inhibition
- severity
- thresholds
- `for` duration
- duplicate rules

Then identify the root cause rather than simply muting everything.

---

# 120. Scenario: One Service Causes Cascading Failure

## Question 168

Use:

- timeouts
- controlled retries
- circuit-breaking patterns
- connection limits
- graceful degradation
- dependency monitoring

The objective is to contain the blast radius.

---

# 121. Senior Design Question

## Question 169: Design a production microservices platform from scratch.

### Answer Framework

Start with requirements:

```text
Traffic
Availability
RPO
RTO
Security
Compliance
Regions
Budget
Team size
```

Then design:

```text
AWS
 |
VPC
 |
EKS
 |
ALB
 |
Microservices
 |
Data
```

Then:

```text
Terraform
CI
ECR
GitOps
Argo CD
```

Then:

```text
Prometheus
Grafana
ELK
Alertmanager
```

Finally:

```text
Security
HA
DR
Backup
Cost
Runbooks
```

---

# 122. Senior Design Question: How Would You Make It Highly Available?

Answer:

```text
Multi-AZ VPC
+
Multi-AZ nodes
+
Multiple Pod replicas
+
Topology spreading
+
PDB
+
ALB
+
Autoscaling
+
HA dependencies
```

Then validate using failure testing.

---

# 123. Senior Design Question: How Would You Make It Secure?

Answer:

```text
Least privilege IAM
+
RBAC
+
Private nodes
+
NetworkPolicy
+
Security groups
+
TLS
+
Secrets management
+
Image scanning
+
SAST
+
Audit logging
+
Restricted access
```

---

# 124. Senior Design Question: How Would You Make It Recoverable?

Answer:

```text
Backups
+
Restore testing
+
GitOps
+
Terraform
+
ECR strategy
+
Database recovery
+
DR region where required
+
Runbooks
+
DR drills
```

---

# 125. Senior Design Question: How Would You Reduce Deployment Risk?

Answer:

```text
Automated tests
+
Security gates
+
Immutable image
+
GitOps
+
Readiness probes
+
Rolling update
+
Observability
+
SLO alerts
+
Rollback
```

---

# 126. Senior Design Question: How Would You Reduce MTTR?

Answer:

> I would improve detection, diagnostics and recovery. That means actionable alerts, good dashboards, centralized logs, runbooks, clear ownership, automated rollback where safe, well-tested recovery procedures and strong incident communication.

---

# 127. Senior Design Question: How Would You Reduce MTTR Without Adding Tools?

### Answer

Improve:

- alert quality
- dashboards
- runbooks
- ownership
- standard commands
- incident procedures
- automation
- documentation

Operational maturity often produces more benefit than another monitoring product.

---

# 128. Behavioral Question

## Question 170: Tell me about a difficult production incident.

### Answer Structure

Use:

```text
Situation
Task
Action
Result
Learning
```

Example:

> We experienced a production latency increase after a deployment. I first established that the increase started immediately after the release and affected one service. Metrics showed latency rising without a major CPU increase. I correlated application logs and identified increased downstream database latency. We rolled back the application change to stabilize traffic, then investigated the query behavior and added a performance regression test. The main lesson was to correlate application and dependency metrics rather than assuming infrastructure CPU was the bottleneck.

---

# 129. Behavioral Question

## Question 171: Tell me about a time you disagreed with a deployment decision.

### Strong Answer

> I focused the discussion on measurable risk rather than personal preference. I explained the failure mode, proposed a safer alternative, and suggested a validation or rollback mechanism. If the final decision remained different, I ensured the risk and mitigation were documented.

---

# 130. Behavioral Question

## Question 172: What do you do when you make a production mistake?

### Answer

> I communicate immediately, contain the impact, restore service, preserve evidence, understand the cause and document the prevention. I do not hide the mistake because delayed disclosure increases impact.

---

# 131. Behavioral Question

## Question 173: How do you handle pressure during an outage?

### Answer

> I separate urgency from randomness. I focus on impact, evidence and the next safest action. I communicate clearly, avoid multiple simultaneous changes and keep a timeline. This helps the team remain systematic under pressure.

---

# 132. Leadership Question

## Question 174: How do you mentor junior DevOps engineers?

### Answer

> I teach the reasoning process rather than only commands. For example, instead of saying which kubectl command to run, I explain how to trace a request from DNS to ALB, Service, Pod and dependency. This creates engineers who can troubleshoot unfamiliar incidents independently.

---

# 133. Architecture Challenge

## Question 175: What would you remove from this architecture if cost became a major constraint?

### Answer

> I would not remove critical HA or security controls blindly. I would first identify waste in node sizing, observability retention, log volume, NAT traffic, idle environments, storage and CI capacity. Then I would consider whether some non-production components can use lower-cost configurations.

---

# 134. Architecture Challenge

## Question 176: What would you never compromise on?

### Answer

> I would not intentionally compromise credential security, production access controls, backup integrity, basic availability requirements or the ability to recover from a failed deployment.

---

# 135. Final Senior Interview Framework

When answering any production question, think:

```text
Requirement
   |
Architecture
   |
Implementation
   |
Security
   |
Observability
   |
Failure
   |
Recovery
   |
Trade-off
```

This framework works for:

- AWS
- Kubernetes
- Terraform
- CI/CD
- GitOps
- monitoring
- security
- DR
- cost
- troubleshooting

---

# 136. Final 30 Questions to Practice Without Notes

1. Explain your complete architecture.
2. Why EKS?
3. Why ALB?
4. Why private worker nodes?
5. Explain multi-AZ.
6. Explain Terraform state.
7. Explain Terraform drift.
8. Explain GitOps.
9. Why Argo CD?
10. CI vs CD?
11. Why ECR?
12. Why immutable images?
13. Explain Helm.
14. Explain HPA.
15. Explain PDB.
16. Explain readiness vs liveness.
17. Troubleshoot CrashLoopBackOff.
18. Troubleshoot Pending Pod.
19. Troubleshoot ImagePullBackOff.
20. Troubleshoot ALB 502.
21. Explain Prometheus.
22. Explain PromQL.
23. Explain SLI/SLO/SLA.
24. Explain Alertmanager.
25. Explain ELK.
26. Explain production incident response.
27. Explain rollback.
28. Explain RPO/RTO.
29. Explain disaster recovery.
30. Explain the biggest architecture trade-off.

---

# 137. Final Interview Answer

If an interviewer asks:

## "Why should we hire you for a senior DevOps role?"

A strong production-oriented answer is:

> I focus on operating systems and platforms reliably rather than only implementing individual tools. I understand the complete path from source code through CI, security validation, container registry and GitOps deployment into Kubernetes and AWS. I can work across Terraform, EKS, Kubernetes, Helm, Argo CD, ALB, Prometheus, Grafana and ELK, but more importantly I understand how these components interact during production failures. I approach incidents systematically, use metrics and logs to establish evidence, prioritize user impact, recover safely and then work on prevention. I also consider security, scalability, disaster recovery and cost when designing or operating a platform.

---

# 138. Final Capstone Interview Checklist

Before a senior interview, be able to explain without notes:

## AWS

- [ ] VPC
- [ ] subnets
- [ ] routing
- [ ] NAT
- [ ] security groups
- [ ] IAM
- [ ] EKS
- [ ] ECR
- [ ] ALB

## Kubernetes

- [ ] Pods
- [ ] Deployments
- [ ] Services
- [ ] Ingress
- [ ] ConfigMaps
- [ ] Secrets
- [ ] probes
- [ ] requests/limits
- [ ] HPA
- [ ] PDB
- [ ] scheduling
- [ ] RBAC
- [ ] NetworkPolicy

## Terraform

- [ ] modules
- [ ] state
- [ ] backend
- [ ] locking
- [ ] plan
- [ ] apply
- [ ] drift
- [ ] recovery

## CI/CD

- [ ] build
- [ ] test
- [ ] SonarQube
- [ ] Trivy
- [ ] Veracode
- [ ] image
- [ ] ECR
- [ ] GitOps update

## GitOps

- [ ] Git desired state
- [ ] Argo CD
- [ ] sync
- [ ] drift
- [ ] rollback
- [ ] multi-cluster

## Observability

- [ ] Prometheus
- [ ] PromQL
- [ ] Grafana
- [ ] ELK
- [ ] Alertmanager
- [ ] golden signals
- [ ] SLOs
- [ ] alert routing

## Production

- [ ] troubleshooting
- [ ] incidents
- [ ] rollback
- [ ] backup
- [ ] restore
- [ ] DR
- [ ] RPO
- [ ] RTO
- [ ] security
- [ ] cost

---

# 139. Final Takeaway

Senior DevOps interviews are rarely about memorizing:

```text
"kubectl get pods"
```

They are about explaining:

```text
Why the system was designed this way
How it behaves
How it fails
How you detect failure
How you troubleshoot it
How you recover
How you prevent recurrence
```

A senior engineer should be able to move between abstraction levels:

```text
Business requirement
       |
Architecture
       |
AWS
       |
Kubernetes
       |
Container
       |
Application
       |
Linux
       |
Network
       |
Database
```

The strongest answers connect these layers.

---

# 140. End of Capstone Interview Questions

The next chapter is:

```text
41-Final-DevOps-Mock-Interview.md
```

It will simulate a complete senior DevOps production interview across the entire capstone, including:

- interviewer questions
- strong candidate answers
- follow-up questions
- troubleshooting rounds
- architecture design
- AWS
- Terraform
- EKS
- Kubernetes
- Helm
- CI/CD
- DevSecOps
- GitOps
- Argo CD
- ALB
- Prometheus
- Grafana
- ELK
- incident response
- DR
- rollback
- security
- cost optimization
- scenario-based questions
- senior-level evaluation criteria
