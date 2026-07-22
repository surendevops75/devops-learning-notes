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

