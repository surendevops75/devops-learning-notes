# Section 10 - Kubernetes Security Troubleshooting Scenarios

---

## Scenario 101

### Interview Question

A developer receives **"Forbidden"** when trying to create a Pod. How would you troubleshoot it?

### Production-Level Answer

A "Forbidden" error usually indicates an authorization problem. Kubernetes has authenticated the user, but RBAC is preventing the requested action.

Always investigate permissions before assuming an infrastructure issue.

### Approach

Check the error message.

Verify the user's permissions.

```bash
kubectl auth can-i create pods --as=<user>
```

Review Roles.

```bash
kubectl get roles
kubectl get rolebindings
```

Review ClusterRoles.

```bash
kubectl get clusterroles
kubectl get clusterrolebindings
```

### Common Root Causes

- Missing Role
- Missing RoleBinding
- Wrong namespace
- Incorrect ClusterRole
- Least privilege restriction

### Best Practices

- Follow least privilege
- Review RBAC regularly
- Audit permission changes
- Avoid cluster-admin access

---

## Scenario 102

### Interview Question

A Pod cannot access the Kubernetes API even though it is running successfully. How would you troubleshoot it?

### Production-Level Answer

Applications communicate with the Kubernetes API using a ServiceAccount.

Verify that the Pod has the correct ServiceAccount and permissions.

### Approach

Check Pod configuration.

```bash
kubectl describe pod <pod-name>
```

Verify ServiceAccount.

```bash
kubectl get sa
```

Check permissions.

```bash
kubectl auth can-i get pods --as=system:serviceaccount:<namespace>:<serviceaccount>
```

### Common Root Causes

- Wrong ServiceAccount
- Missing RoleBinding
- Token disabled
- RBAC restriction

### Best Practices

- Use dedicated ServiceAccounts
- Grant minimum permissions
- Avoid using default ServiceAccount

---

## Scenario 103

### Interview Question

A Pod admission request is rejected before scheduling. What would you investigate?

### Production-Level Answer

Admission Controllers validate requests before Kubernetes creates resources.

Investigate admission policies before troubleshooting the workload itself.

### Approach

Describe the resource.

Review Events.

Check

- Admission Controller
- Pod Security Admission
- Validating Webhooks
- Mutating Webhooks

### Common Root Causes

- Policy violation
- Required labels missing
- Image policy rejection
- Security restrictions

### Best Practices

- Validate manifests before deployment
- Test admission policies
- Keep admission rules documented

---

## Scenario 104

### Interview Question

A Pod is rejected because it violates the Pod Security Standards. How would you troubleshoot it?

### Production-Level Answer

Pod Security Admission enforces security standards such as Restricted, Baseline, and Privileged.

Review which security requirement is being violated.

### Approach

Describe the namespace.

```bash
kubectl describe ns <namespace>
```

Review

- Security Context
- Privileged mode
- Capabilities
- HostPath
- HostNetwork

### Common Root Causes

- Privileged container
- Running as root
- HostPath volume
- Missing SecurityContext

### Best Practices

- Use Restricted policy
- Avoid privileged containers
- Configure SecurityContext
- Validate manifests during CI

---

## Scenario 105

### Interview Question

A container fails because it cannot write to the filesystem after enabling security hardening. How would you investigate?

### Production-Level Answer

Security hardening often enables read-only filesystems.

Applications must only write to approved writable locations.

### Approach

Verify

- SecurityContext
- readOnlyRootFilesystem
- Volume mounts
- Application paths

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

### Common Root Causes

- Read-only filesystem
- Wrong mount path
- Missing writable volume
- Application design issue

### Best Practices

- Design stateless applications
- Mount writable volumes
- Test hardened images

---

## Scenario 106

### Interview Question

An image is rejected during deployment because of an Image Policy. What would you investigate?

### Production-Level Answer

Image policies restrict which container images are allowed inside the cluster.

Verify image source, registry, and signatures.

### Approach

Check

- Admission Controller
- Image Policy
- Registry
- Image signature
- Allowed repositories

Review Events.

### Common Root Causes

- Untrusted registry
- Unsigned image
- Wrong repository
- Policy violation

### Best Practices

- Sign images
- Use trusted registries
- Enforce image policies
- Scan images before deployment

---

## Scenario 107

### Interview Question

A ServiceAccount token appears to have expired. How would you troubleshoot it?

### Production-Level Answer

Modern Kubernetes versions use short-lived ServiceAccount tokens.

Applications should request fresh tokens automatically.

### Approach

Verify

- Kubernetes version
- Token projection
- ServiceAccount
- API authentication

Check Pod specification.

### Common Root Causes

- Expired token
- Legacy token usage
- Incorrect token mount
- Authentication failure

### Best Practices

- Use projected ServiceAccount tokens
- Avoid static tokens
- Rotate credentials automatically

---

## Scenario 108

### Interview Question

Sensitive information is accidentally committed inside a ConfigMap instead of a Secret. What should you do?

### Production-Level Answer

ConfigMaps are not designed for sensitive information.

Immediately migrate sensitive values into Secrets or an external Secret Manager.

### Approach

Identify

- Exposed credentials
- Applications using ConfigMap
- Git history
- CI/CD pipelines

Rotate all exposed credentials.

### Common Root Causes

- Human error
- Poor review process
- Missing security validation
- GitOps mistake

### Best Practices

- Store secrets only in Secrets
- Use AWS Secrets Manager or HashiCorp Vault
- Scan repositories for secrets
- Rotate leaked credentials immediately

---

## Scenario 109

### Interview Question

Your security team asks how to verify who deleted a production Deployment yesterday. How would you investigate?

### Production-Level Answer

Use Kubernetes Audit Logs to identify who performed the action, when it occurred, and from which client.

Audit Logs provide the source of truth for security investigations.

### Approach

Review

- Kubernetes Audit Logs
- API Server logs
- Cloud audit logs
- GitOps history

Correlate timestamps with deployment history.

### Common Root Causes

- Human error
- Unauthorized access
- Automation script
- GitOps synchronization

### Best Practices

- Enable Audit Logs
- Store logs centrally
- Protect audit records
- Review privileged access regularly

---

## Scenario 110

### Interview Question

Your Kubernetes cluster passes all health checks, but a security audit identifies excessive permissions across multiple ServiceAccounts. How would you improve the environment?

### Production-Level Answer

Healthy clusters are not necessarily secure.

Review permissions and reduce access according to the Principle of Least Privilege.

### Approach

Review

- ServiceAccounts
- Roles
- ClusterRoles
- RoleBindings
- ClusterRoleBindings

Identify unnecessary permissions.

Use

```bash
kubectl auth can-i
```

to validate required access.

### Common Root Causes

- Overuse of cluster-admin
- Shared ServiceAccounts
- Broad ClusterRoles
- Missing RBAC review

### Best Practices

- Apply least privilege
- Use dedicated ServiceAccounts
- Audit RBAC periodically
- Integrate IAM where applicable
- Monitor security continuously

---

# Section 11 - Amazon EKS Production Troubleshooting Scenarios

---

## Scenario 111

### Interview Question

One of your Amazon EKS Worker Nodes suddenly becomes **NotReady**, while the remaining nodes continue working normally. How would you troubleshoot it?

### Production-Level Answer

A single NotReady node usually indicates a node-level issue rather than a cluster-wide problem.

Investigate both the Kubernetes node status and the underlying EC2 instance.

### Approach

Check node status.

```bash
kubectl get nodes
```

Describe the node.

```bash
kubectl describe node <node-name>
```

Verify

- EC2 Instance status
- Kubelet
- Container Runtime
- Disk utilization
- Memory utilization
- Network connectivity

Check EC2 health in AWS Console.

### Common Root Causes

- EC2 instance reboot
- Kubelet stopped
- Container runtime failure
- Network issue
- Disk full
- Memory exhaustion

### Best Practices

- Enable Managed Node Groups
- Monitor node health
- Configure Auto Scaling
- Replace unhealthy nodes automatically

---

## Scenario 112

### Interview Question

New Pods remain in the Pending state because the Cluster Autoscaler is not adding Worker Nodes. How would you investigate?

### Production-Level Answer

Cluster Autoscaler should automatically provision additional Worker Nodes when scheduling fails because of insufficient resources.

If scaling does not occur, investigate the Autoscaler configuration and AWS infrastructure.

### Approach

Check pending Pods.

```bash
kubectl describe pod <pod-name>
```

Check Cluster Autoscaler.

```bash
kubectl logs -n kube-system deployment/cluster-autoscaler
```

Verify

- IAM permissions
- Auto Scaling Groups
- Node Group limits
- Autoscaler configuration

### Common Root Causes

- IAM permission denied
- Incorrect Auto Scaling Group tags
- Autoscaler crash
- Maximum node count reached
- Resource limits

### Best Practices

- Monitor Autoscaler logs
- Configure scaling limits properly
- Test scaling periodically
- Alert on failed scale events

---

## Scenario 113

### Interview Question

Pods fail with **ImagePullBackOff** while pulling images from Amazon ECR. How would you troubleshoot it?

### Production-Level Answer

ImagePullBackOff from Amazon ECR usually indicates authentication, IAM, networking, or repository issues.

### Approach

Verify

- Image name
- Image tag
- ECR repository
- IAM Role
- IRSA
- Network connectivity

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

### Common Root Causes

- Image missing
- Wrong image tag
- IAM permission denied
- ECR authentication failure
- Repository deleted

### Best Practices

- Use immutable image tags
- Validate ECR permissions
- Monitor image pull failures
- Automate image validation

---

## Scenario 114

### Interview Question

The AWS Load Balancer Controller fails to create an Application Load Balancer (ALB). How would you troubleshoot it?

### Production-Level Answer

The AWS Load Balancer Controller provisions ALBs based on Kubernetes Ingress resources.

Investigate the controller before investigating the application.

### Approach

Verify

- Controller Pods
- Controller logs
- IAM Role
- IRSA
- Subnet tags
- Security Groups

Describe the Ingress.

```bash
kubectl describe ingress <ingress-name>
```

### Common Root Causes

- Controller not running
- Missing IAM permissions
- Incorrect subnet tags
- Invalid annotations
- OIDC issue

### Best Practices

- Monitor controller health
- Validate subnet tagging
- Use least privilege IAM
- Review controller logs regularly

---

## Scenario 115

### Interview Question

A Pod using IAM Roles for Service Accounts (IRSA) receives **AccessDenied** while accessing AWS services. How would you troubleshoot it?

### Production-Level Answer

IRSA allows Pods to securely access AWS services without static credentials.

AccessDenied usually indicates an IAM configuration issue.

### Approach

Verify

- ServiceAccount
- IAM Role
- Trust Policy
- OIDC Provider
- IAM Permissions

Describe the ServiceAccount.

```bash
kubectl describe sa <serviceaccount-name>
```

### Common Root Causes

- Wrong IAM Role
- Missing trust relationship
- OIDC provider missing
- Insufficient IAM permissions
- Wrong annotation

### Best Practices

- Use dedicated IAM Roles
- Follow least privilege
- Validate IRSA configuration
- Monitor IAM failures

---

## Scenario 116

### Interview Question

After upgrading an Amazon EKS cluster, several applications stop working. How would you investigate?

### Production-Level Answer

Cluster upgrades affect multiple Kubernetes components.

Determine whether the issue is caused by deprecated APIs, incompatible workloads, or infrastructure changes.

### Approach

Review

- Upgrade logs
- Kubernetes Events
- Application logs
- API compatibility
- CSI Drivers
- Ingress Controller

Check cluster version.

```bash
kubectl version
```

### Common Root Causes

- Deprecated APIs
- CSI incompatibility
- Ingress Controller incompatibility
- Application incompatibility
- Failed node upgrade

### Best Practices

- Test upgrades in staging
- Read Kubernetes release notes
- Upgrade components in sequence
- Validate workloads after upgrade

---

## Scenario 117

### Interview Question

New Worker Nodes join the Auto Scaling Group but never appear in the Kubernetes cluster. How would you troubleshoot it?

### Production-Level Answer

If EC2 instances are running but not registering with EKS, investigate the node bootstrap process.

### Approach

Verify

- EC2 instance
- Bootstrap script
- IAM Role
- Network
- kubelet
- API connectivity

Check node logs.

### Common Root Causes

- Bootstrap failure
- IAM Role missing
- Security Group issue
- API endpoint unreachable
- kubelet failure

### Best Practices

- Monitor node registration
- Validate bootstrap configuration
- Keep launch templates updated

---

## Scenario 118

### Interview Question

Applications suddenly lose connectivity because the Amazon VPC CNI plugin is failing. How would you investigate?

### Production-Level Answer

The AWS VPC CNI plugin manages Pod networking in Amazon EKS.

Its failure impacts Pod communication and IP allocation.

### Approach

Check VPC CNI Pods.

```bash
kubectl get pods -n kube-system
```

Review logs.

```bash
kubectl logs -n kube-system -l k8s-app=aws-node
```

Verify

- ENIs
- Available IPs
- Node networking

### Common Root Causes

- CNI crash
- ENI limit reached
- IP exhaustion
- IAM permission issue
- Network failure

### Best Practices

- Monitor VPC CNI
- Plan subnet capacity
- Enable Prefix Delegation
- Monitor ENI utilization

---

## Scenario 119

### Interview Question

An Amazon EKS application works correctly in one Availability Zone but fails in another. How would you troubleshoot it?

### Production-Level Answer

AZ-specific failures usually indicate infrastructure, networking, or storage issues rather than Kubernetes itself.

### Approach

Compare

- Worker Nodes
- Subnets
- Route Tables
- Security Groups
- EBS volumes
- Load Balancer Targets

Review Events.

### Common Root Causes

- Subnet misconfiguration
- Security Group issue
- Missing route
- EBS AZ mismatch
- Target Group unhealthy

### Best Practices

- Design for Multi-AZ
- Monitor every Availability Zone
- Test failover regularly
- Distribute workloads evenly

---

## Scenario 120

### Interview Question

A production incident affects multiple microservices running on Amazon EKS. Where do you begin your investigation?

### Production-Level Answer

Avoid investigating individual applications first.

Start from the infrastructure and progressively move toward the application layer.

Follow a structured troubleshooting methodology.

### Approach

Verify in order

1. AWS Health Dashboard
2. EKS Control Plane
3. Worker Nodes
4. VPC Networking
5. CoreDNS
6. VPC CNI
7. Ingress Controller
8. Services
9. Pods
10. Application Logs
11. Prometheus Metrics
12. Grafana Dashboards

Correlate timestamps across monitoring, logs, and Kubernetes Events before identifying the root cause.

### Common Root Causes

- AWS infrastructure issue
- Worker Node failure
- Networking issue
- DNS failure
- CNI failure
- Application deployment
- External dependency outage

### Best Practices

- Follow a top-down troubleshooting approach
- Correlate logs, metrics, and events
- Maintain production runbooks
- Perform Root Cause Analysis (RCA)
- Conduct regular disaster recovery drills

---

# Section 12 - StatefulSet Troubleshooting Scenarios

---

## Scenario 121

### Interview Question

A StatefulSet Pod is stuck in the **Pending** state, while Deployments are working normally. How would you troubleshoot it?

### Production-Level Answer

Unlike Deployments, StatefulSets require stable identities and dedicated PersistentVolumeClaims.

Investigate the storage before troubleshooting the application.

### Approach

Check StatefulSet.

```bash
kubectl get statefulset
```

Check Pods.

```bash
kubectl get pods
```

Check PVCs.

```bash
kubectl get pvc
```

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

### Common Root Causes

- PVC Pending
- StorageClass issue
- PV unavailable
- CSI Driver failure
- Node resource shortage

### Best Practices

- Monitor StatefulSet storage
- Use dynamic provisioning
- Validate PVC creation
- Alert on Pending PVCs

---

## Scenario 122

### Interview Question

A StatefulSet Pod keeps changing its identity after every restart. What would you investigate?

### Production-Level Answer

StatefulSet Pods should maintain stable identities.

Changing Pod identities usually indicates the workload is deployed incorrectly.

### Approach

Verify

- StatefulSet manifest
- Pod names
- PVC names
- Headless Service

Check

```bash
kubectl describe statefulset <statefulset-name>
```

### Common Root Causes

- Deployment used instead of StatefulSet
- StatefulSet recreated
- Incorrect manifest
- PVC deleted

### Best Practices

- Never recreate StatefulSets unnecessarily
- Preserve PVCs
- Use stable DNS names

---

## Scenario 123

### Interview Question

A StatefulSet rollout stops after updating the first Pod. How would you troubleshoot it?

### Production-Level Answer

StatefulSets update Pods sequentially.

If one Pod fails, Kubernetes stops the rollout to protect application consistency.

### Approach

Check rollout.

```bash
kubectl rollout status statefulset <statefulset-name>
```

Describe failing Pod.

```bash
kubectl describe pod <pod-name>
```

Review

- Readiness
- Storage
- Logs

### Common Root Causes

- Readiness failure
- Storage issue
- Application startup failure
- Probe failure

### Best Practices

- Validate updates in staging
- Monitor rollout progress
- Use partitioned updates if required

---

## Scenario 124

### Interview Question

A database running inside a StatefulSet cannot recover after restarting. How would you investigate?

### Production-Level Answer

Stateful applications depend on persistent storage.

Verify whether the application is accessing the correct PersistentVolume.

### Approach

Check

- PVC
- PV
- Mount path
- Database logs
- Volume permissions

Describe PVC.

```bash
kubectl describe pvc <pvc-name>
```

### Common Root Causes

- Wrong mount path
- Missing PVC
- Storage corruption
- Permission issue
- Database recovery failure

### Best Practices

- Backup databases regularly
- Test recovery procedures
- Monitor storage latency

---

## Scenario 125

### Interview Question

Your StatefulSet Pods cannot communicate using stable DNS names. How would you troubleshoot it?

### Production-Level Answer

StatefulSets rely on Headless Services for stable DNS.

Verify the networking configuration before investigating the application.

### Approach

Check Headless Service.

```bash
kubectl get svc
```

Verify

- clusterIP: None
- Service selector
- Pod DNS

Test DNS.

```bash
nslookup <pod-name>.<service-name>
```

### Common Root Causes

- Headless Service missing
- Wrong selector
- DNS failure
- CoreDNS issue

### Best Practices

- Always use Headless Services
- Validate StatefulSet DNS
- Monitor CoreDNS

---

## Scenario 126

### Interview Question

A StatefulSet Pod remains in the **Terminating** state because the volume cannot be detached. How would you troubleshoot it?

### Production-Level Answer

Stateful workloads cannot safely terminate until storage operations complete.

Investigate storage before forcing deletion.

### Approach

Check

- PVC
- PV
- CSI Driver
- Node status
- Volume attachment

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

### Common Root Causes

- CSI Driver failure
- Volume attachment issue
- Node unavailable
- Storage backend problem

### Best Practices

- Monitor CSI Drivers
- Avoid force deletion
- Verify storage health

---

## Scenario 127

### Interview Question

Scaling a StatefulSet from 3 replicas to 5 replicas fails. How would you troubleshoot it?

### Production-Level Answer

Each new replica requires its own PersistentVolumeClaim.

Verify storage availability before investigating Kubernetes scheduling.

### Approach

Check

```bash
kubectl get pvc
```

Review

- StorageClass
- Available storage
- Node capacity
- Scheduler Events

### Common Root Causes

- Storage unavailable
- PVC Pending
- CSI Driver issue
- Cluster capacity exhausted

### Best Practices

- Monitor storage growth
- Plan capacity
- Use dynamic provisioning

---

## Scenario 128

### Interview Question

A StatefulSet cannot scale down because one Pod refuses to terminate. How would you investigate?

### Production-Level Answer

StatefulSets terminate Pods in reverse order.

If the last Pod cannot terminate, scaling stops.

### Approach

Check

- Finalizers
- Volume attachments
- Application shutdown
- PreStop hook

Describe the Pod.

```bash
kubectl describe pod <pod-name>
```

### Common Root Causes

- Finalizer
- Storage issue
- Long-running shutdown
- Application lock

### Best Practices

- Implement graceful shutdown
- Monitor termination time
- Validate PreStop hooks

---

## Scenario 129

### Interview Question

A StatefulSet is accidentally deleted. What happens to the data?

### Production-Level Answer

Deleting a StatefulSet does not automatically delete PersistentVolumes if the Reclaim Policy is Retain.

Always verify storage before attempting recovery.

### Approach

Check

```bash
kubectl get pvc
kubectl get pv
```

Verify

- Reclaim Policy
- Storage backend
- Remaining volumes

### Common Root Causes

- Manual deletion
- Incorrect cleanup
- Reclaim Policy Delete
- Human error

### Best Practices

- Use Retain for production databases
- Backup data regularly
- Protect StatefulSets using RBAC

---

## Scenario 130

### Interview Question

A production database running as a StatefulSet becomes unavailable after a node failure. How would you troubleshoot it?

### Production-Level Answer

Stateful workloads recover differently from stateless applications.

Focus on node recovery, storage attachment, and application recovery in sequence.

### Approach

Verify

- Worker Node
- PVC
- PV
- CSI Driver
- Database logs
- Pod Events

Ensure the volume is attached to the new node before expecting the database to start.

### Common Root Causes

- Node failure
- Volume attachment delay
- Storage corruption
- Database recovery process
- CSI Driver issue

### Best Practices

- Deploy across multiple Availability Zones
- Monitor storage attachment
- Test failover regularly
- Automate backups
- Validate disaster recovery procedures

---