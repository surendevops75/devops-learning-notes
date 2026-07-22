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

