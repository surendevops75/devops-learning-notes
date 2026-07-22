# Section 15 - Cluster Administration Troubleshooting Scenarios

---

## Scenario 151

### Interview Question

Several namespaces suddenly cannot create new Pods, while existing workloads continue running. How would you troubleshoot it?

### Production-Level Answer

If existing Pods are healthy but new Pods cannot be created, investigate namespace policies, quotas, and admission controls before checking the applications.

### Approach

Check Events.

```bash
kubectl get events --sort-by=.metadata.creationTimestamp
```

Describe the namespace.

```bash
kubectl describe namespace <namespace>
```

Verify

- ResourceQuota
- LimitRange
- Admission Controllers
- RBAC
- API Server health

### Common Root Causes

- ResourceQuota exceeded
- LimitRange violation
- Admission policy rejection
- API Server issue
- RBAC restriction

### Best Practices

- Monitor namespace quotas
- Configure alerts
- Review admission policies
- Audit namespace usage

---

## Scenario 152

### Interview Question

The Kubernetes API Server becomes slow and kubectl commands take several seconds to respond. How would you investigate?

### Production-Level Answer

The API Server is the control plane entry point.

High latency affects the entire cluster and should be investigated immediately.

### Approach

Verify

- API Server health
- etcd latency
- Control Plane CPU
- Memory
- API request rate

Run

```bash
kubectl cluster-info
```

Check component health.

### Common Root Causes

- High API traffic
- etcd latency
- Resource exhaustion
- Large number of watches
- Control Plane overload

### Best Practices

- Monitor API latency
- Limit excessive API polling
- Optimize controllers
- Scale the Control Plane when applicable

---

## Scenario 153

### Interview Question

Developers receive **Forbidden** errors when creating Deployments in one namespace but not another. How would you troubleshoot it?

### Production-Level Answer

Namespace-specific failures usually indicate an RBAC issue rather than a cluster-wide problem.

### Approach

Verify permissions.

```bash
kubectl auth can-i create deployments \
--namespace=<namespace> \
--as=<user>
```

Review

- Roles
- RoleBindings
- ClusterRoles
- ClusterRoleBindings

### Common Root Causes

- Missing RoleBinding
- Incorrect Role
- Namespace mismatch
- RBAC changes

### Best Practices

- Apply least privilege
- Audit RBAC regularly
- Use Groups instead of individual users

---

## Scenario 154

### Interview Question

A namespace remains in the **Terminating** state for hours. How would you troubleshoot it?

### Production-Level Answer

Namespaces usually remain in the Terminating state because Kubernetes is waiting for resources or finalizers to be removed.

### Approach

Describe the namespace.

```bash
kubectl describe namespace <namespace>
```

Check remaining resources.

```bash
kubectl api-resources --verbs=list --namespaced -o name \
| xargs -n1 kubectl get -n <namespace>
```

Verify

- Finalizers
- Custom Resources
- Operators

### Common Root Causes

- Finalizers
- Stuck CRDs
- Operator failure
- API errors

### Best Practices

- Remove unused CRDs
- Monitor namespace deletion
- Avoid orphaned resources

---

## Scenario 155

### Interview Question

After installing a Custom Resource Definition (CRD), Kubernetes rejects all custom resources. How would you troubleshoot it?

### Production-Level Answer

CRDs extend the Kubernetes API.

Failures usually occur because the CRD is missing, incompatible, or incorrectly defined.

### Approach

Check CRDs.

```bash
kubectl get crd
```

Describe the CRD.

```bash
kubectl describe crd <crd-name>
```

Verify

- API version
- Schema
- Validation
- Controller

### Common Root Causes

- Wrong API version
- CRD missing
- Validation failure
- Controller unavailable

### Best Practices

- Version CRDs carefully
- Validate schemas
- Upgrade CRDs safely

---

## Scenario 156

### Interview Question

A Kubernetes Operator stops reconciling resources after an upgrade. How would you investigate?

### Production-Level Answer

Operators continuously reconcile desired state.

If reconciliation stops, workloads may drift from their intended configuration.

### Approach

Verify

- Operator Pod
- Logs
- CRDs
- Leader election
- Events

Check Operator logs.

```bash
kubectl logs deployment/<operator-name>
```

### Common Root Causes

- Operator crash
- API incompatibility
- Leader election failure
- RBAC issue

### Best Practices

- Monitor Operator health
- Validate upgrades
- Test reconciliation regularly

---

## Scenario 157

### Interview Question

The Kubernetes Scheduler is running, but Pods remain Pending without scheduling decisions. How would you troubleshoot it?

### Production-Level Answer

If the Scheduler is healthy but Pods remain Pending, investigate scheduling constraints and scheduler logs.

### Approach

Verify

- Scheduler Pod
- Scheduler logs
- Events
- Node resources
- Affinity rules
- Taints

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

### Common Root Causes

- Scheduler bug
- Resource shortage
- Affinity conflict
- Taints
- Scheduler configuration

### Best Practices

- Monitor scheduler metrics
- Avoid unnecessary scheduling constraints
- Plan cluster capacity

---

## Scenario 158

### Interview Question

A Kubernetes cluster contains thousands of completed Jobs, causing API performance issues. How would you resolve it?

### Production-Level Answer

Large numbers of completed resources increase API load and etcd storage usage.

Implement automatic cleanup.

### Approach

Check completed Jobs.

```bash
kubectl get jobs
```

Review

- TTLAfterFinished
- CronJob history limits
- Resource counts

### Common Root Causes

- No cleanup policy
- Excessive CronJobs
- Long retention
- Manual cleanup neglected

### Best Practices

- Configure TTLAfterFinished
- Limit Job history
- Monitor etcd object count

---

## Scenario 159

### Interview Question

An administrator accidentally deletes an important namespace. How would you respond?

### Production-Level Answer

Namespace deletion removes most namespaced resources.

Recovery depends on backups and GitOps practices.

### Approach

Verify

- Backup availability
- Git repository
- PersistentVolumes
- Application data
- Audit logs

Restore resources using backup or GitOps.

### Common Root Causes

- Human error
- Excessive permissions
- Missing backups
- Weak change management

### Best Practices

- Enable GitOps
- Backup cluster resources
- Restrict delete permissions
- Audit administrative actions

---

## Scenario 160

### Interview Question

A production cluster experiences multiple simultaneous failures affecting workloads across namespaces. Where do you begin your investigation?

### Production-Level Answer

Begin with the Control Plane before investigating individual namespaces or applications.

Follow a structured troubleshooting workflow to identify the first failing component.

### Approach

Verify in order

1. API Server
2. etcd
3. Scheduler
4. Controller Manager
5. Worker Nodes
6. CoreDNS
7. Network
8. Storage
9. Namespace Events
10. Applications

Correlate Events, logs, and metrics to determine the initial point of failure.

### Common Root Causes

- Control Plane degradation
- etcd latency
- Scheduler issue
- Infrastructure failure
- Cluster-wide configuration change

### Best Practices

- Maintain control plane monitoring
- Backup cluster state
- Use GitOps for configuration management
- Document incident response procedures
- Perform regular disaster recovery drills

---

# Section 16 - Disaster Recovery & ETCD Troubleshooting Scenarios

---

## Scenario 161

### Interview Question

The Kubernetes API Server suddenly becomes unavailable across the entire cluster. How would you troubleshoot it?

### Production-Level Answer

The API Server is the entry point to the Kubernetes Control Plane.

If it is unavailable, first determine whether the issue is with the API Server itself or the underlying etcd datastore.

### Approach

Verify

- API Server status
- Control Plane nodes
- etcd health
- Certificates
- Network connectivity

Check cluster information.

```bash
kubectl cluster-info
```

Review API Server logs.

### Common Root Causes

- API Server crash
- Expired certificates
- etcd unavailable
- Network failure
- Resource exhaustion

### Best Practices

- Monitor API Server health
- Enable HA Control Plane
- Rotate certificates before expiry
- Configure alerting

---

## Scenario 162

### Interview Question

The etcd database is running, but API requests are extremely slow. How would you investigate?

### Production-Level Answer

The API Server depends entirely on etcd.

Slow etcd performance directly impacts every Kubernetes operation.

### Approach

Verify

- etcd latency
- Disk IOPS
- CPU
- Memory
- Database size

Review etcd metrics and logs.

### Common Root Causes

- Slow storage
- Large database
- High API request rate
- Resource exhaustion
- Disk fragmentation

### Best Practices

- Use SSD storage
- Monitor etcd latency
- Compact the database regularly
- Defragment etcd periodically

---

## Scenario 163

### Interview Question

An etcd backup completed successfully, but restoration fails. How would you troubleshoot it?

### Production-Level Answer

A backup is useful only if it can be restored successfully.

Validate both the backup integrity and restore procedure.

### Approach

Verify

- Backup file
- etcd version
- Restore command
- Cluster configuration

Test restoration in a non-production environment.

### Common Root Causes

- Corrupted backup
- Version mismatch
- Incorrect restore path
- Configuration mismatch

### Best Practices

- Test restores regularly
- Store backups securely
- Automate backup verification

---

## Scenario 164

### Interview Question

A Control Plane node fails in a Highly Available Kubernetes cluster. How would you respond?

### Production-Level Answer

HA clusters are designed to tolerate individual Control Plane failures.

Verify that the remaining Control Plane nodes continue serving requests.

### Approach

Check

- Remaining API Servers
- etcd quorum
- Load Balancer
- Controller Manager
- Scheduler

Verify cluster functionality before replacing the failed node.

### Common Root Causes

- Hardware failure
- Operating system issue
- Network outage
- Resource exhaustion

### Best Practices

- Deploy at least three Control Plane nodes
- Monitor quorum health
- Replace failed nodes quickly

---

## Scenario 165

### Interview Question

After restoring etcd, several Kubernetes resources are missing. How would you investigate?

### Production-Level Answer

Resources created after the backup was taken cannot be recovered from that backup.

Determine the backup timestamp before investigating further.

### Approach

Verify

- Backup timestamp
- Missing resources
- GitOps repository
- Audit logs

Compare restored resources with Git.

### Common Root Causes

- Outdated backup
- Human error
- Incomplete restore
- Missing Git synchronization

### Best Practices

- Schedule frequent backups
- Use GitOps
- Validate restore results

---

## Scenario 166

### Interview Question

A production cluster must be recovered after complete Control Plane failure. What would be your recovery strategy?

### Production-Level Answer

Recover the Control Plane first before restoring Worker Nodes or applications.

Follow the documented disaster recovery procedure.

### Approach

Recover

1. Control Plane
2. etcd
3. API Server
4. Scheduler
5. Controller Manager
6. Worker Nodes
7. Applications

Validate cluster health after each step.

### Common Root Causes

- Infrastructure failure
- Storage corruption
- Accidental deletion
- Cloud outage

### Best Practices

- Maintain DR runbooks
- Test recovery regularly
- Automate infrastructure provisioning

---

## Scenario 167

### Interview Question

A cluster backup succeeds every day, but no one has ever tested restoring it. What risk does this create?

### Production-Level Answer

Untested backups cannot be considered reliable.

Disaster recovery depends on successful restoration, not successful backup completion.

### Approach

Verify

- Backup integrity
- Restore process
- Recovery time
- Documentation

Schedule periodic recovery drills.

### Common Root Causes

- Untested backups
- Corrupted backup files
- Missing documentation
- Outdated procedures

### Best Practices

- Test backups quarterly
- Document restore procedures
- Measure Recovery Time Objective (RTO)

---

## Scenario 168

### Interview Question

An administrator accidentally deletes multiple Deployments from production. How would you recover?

### Production-Level Answer

Recovery depends on whether GitOps, backups, or exported manifests are available.

Avoid recreating resources manually if reliable configuration exists.

### Approach

Verify

- Git repository
- ArgoCD or Flux
- etcd backup
- Audit logs

Restore manifests from the source of truth.

### Common Root Causes

- Human error
- Excessive permissions
- Missing GitOps
- Lack of RBAC

### Best Practices

- Use Git as the source of truth
- Restrict delete permissions
- Enable audit logging

---

## Scenario 169

### Interview Question

A cloud region outage affects your Kubernetes cluster. How would you maintain business continuity?

### Production-Level Answer

Regional failures require disaster recovery planning beyond Kubernetes itself.

Recovery depends on infrastructure redundancy.

### Approach

Verify

- Backup region
- Infrastructure availability
- DNS failover
- Data replication
- Application readiness

Activate disaster recovery procedures.

### Common Root Causes

- Regional outage
- Infrastructure failure
- Storage unavailability
- Network disruption

### Best Practices

- Design Multi-Region architecture
- Replicate backups
- Test regional failover
- Automate recovery

---

## Scenario 170

### Interview Question

Management asks how confident you are that the Kubernetes disaster recovery plan will work. How would you answer?

### Production-Level Answer

Confidence should be based on verified recovery testing, not assumptions.

A disaster recovery plan is only effective if it has been successfully tested under realistic conditions.

### Approach

Review

- Backup reports
- Restore tests
- Recovery documentation
- RTO
- RPO
- Previous DR exercises

Present evidence from recent disaster recovery drills.

### Common Root Causes

- No recovery testing
- Outdated documentation
- Untested backups
- Incomplete automation

### Best Practices

- Conduct regular DR exercises
- Measure RTO and RPO
- Keep runbooks updated
- Review lessons learned after every drill

---