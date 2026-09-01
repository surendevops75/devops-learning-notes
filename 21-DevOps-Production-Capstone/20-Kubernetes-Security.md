# 20 --- Kubernetes Security --- Production DevOps Capstone

> Deep production-focused Kubernetes/EKS security chapter covering AWS
> IAM, EKS identity, RBAC, workload identity, secrets, NetworkPolicy,
> Pod Security Standards, container hardening, image and software
> supply-chain security, admission control, runtime security,
> observability, incident response, GitOps/Terraform/Helm security,
> multi-cluster security, production runbooks, checklists, and senior
> DevOps interview preparation.

## Chapter Objective

Security is treated as an end-to-end production architecture rather than
a collection of isolated Kubernetes commands.

## 1. Security Objectives and Threat Model

Kubernetes security in production is a layered discipline. Protect the
AWS account, VPC, EKS control plane, worker nodes, Kubernetes API,
workloads, containers, identities, secrets, network paths, admission
controls, supply chain, observability, and recovery process.

A useful threat model asks: - Who can authenticate? - What can each
identity do? - Which workloads can communicate? - Which workloads can
access AWS APIs? - Which images can run? - Which namespaces can affect
each other? - What happens if a pod is compromised? - What happens if a
node is compromised? - How quickly can credentials be revoked? - Can the
environment be rebuilt from trusted source?

Production security is not achieved by one tool. It is achieved by
reducing blast radius at every layer.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 2. Defense in Depth

``` text
AWS Organizations / IAM
        |
        v
VPC / Subnets / Security Groups / Network Controls
        |
        v
EKS Control Plane / Endpoint Controls
        |
        v
Authentication / Authorization / RBAC
        |
        v
Admission / Pod Security / Policy
        |
        v
Container Image / Supply Chain
        |
        v
Pod Security Context
        |
        v
NetworkPolicy
        |
        v
Secrets / Encryption
        |
        v
Runtime Detection / Audit Logs
        |
        v
Incident Response / Recovery
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 3. AWS Account Boundary

Keep production workloads in a dedicated AWS account or appropriately
isolated organizational structure. Separate human access from workload
access, use centralized identity where possible, enforce MFA for
privileged identities, and avoid long-lived administrator credentials.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 4. Root Account Protection

The AWS root account should not be used for routine operations. Protect
root credentials, enable strong MFA, remove unnecessary root access
keys, and establish an emergency access process. Document the
break-glass procedure and test that authorized personnel can use it when
required.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 5. IAM Least Privilege

Grant only the actions and resources required for the job. Avoid
wildcard permissions such as Action: '*' and Resource: '*' unless there
is a documented reason. Review permissions periodically and remove stale
access.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 6. IAM Roles Instead of Long-Lived Keys

Applications running in EKS should obtain AWS permissions through
workload identity rather than storing AWS access keys in Kubernetes
Secrets. Human operators should also prefer federated role assumption
over permanent access keys.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 7. EKS Identity Architecture

``` text
Human / CI / Workload
        |
        +--> Authentication
        |
        +--> Kubernetes Authorization
        |
        +--> AWS IAM Authorization
        |
        +--> Resource-level controls
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 8. EKS API Endpoint

EKS can expose a public endpoint, private endpoint, or both with
restrictions. Production environments should minimize unnecessary
exposure of the Kubernetes API. When public access is required, restrict
CIDRs and use strong authentication and authorization controls.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 9. Private EKS Endpoint

A private endpoint keeps Kubernetes API traffic inside the configured
VPC connectivity path. It can reduce exposure but requires operators and
CI systems to have network connectivity into the VPC, often through VPN,
Direct Connect, bastionless private access, or controlled runners.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 10. Control Plane Protection

Do not treat the managed EKS control plane as a replacement for workload
security. Control-plane access must be restricted, Kubernetes audit logs
should be considered for centralized monitoring, and cluster
configuration should be managed through controlled infrastructure code.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 11. Kubernetes Authentication

Authentication answers who the caller is. In EKS, identity can originate
from AWS IAM or configured Kubernetes authentication mechanisms.
Authentication alone does not grant permissions; authorization
determines what that identity can do.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 12. Kubernetes Authorization

Authorization evaluates the authenticated identity against Kubernetes
RBAC and other authorization controls. A user being able to authenticate
to the cluster must not imply cluster-admin access.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 13. RBAC Fundamentals

Kubernetes RBAC uses Role or ClusterRole objects and RoleBinding or
ClusterRoleBinding objects. Roles are namespace-scoped; ClusterRoles can
define cluster-scoped permissions or reusable permission sets.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 14. Role vs ClusterRole

Use a Role when access should remain inside one namespace. Use a
ClusterRole when cluster-scoped resources are required or when you
intentionally reuse the permission definition across namespaces. Avoid
ClusterRoleBindings for ordinary application identities.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 15. Least-Privilege RBAC

Prefer get/list/watch only where needed, and grant create/update/delete
separately when operationally necessary. Avoid verbs such as '*' and
resources such as '*' in production application roles.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 16. RBAC Example

``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-reader
  namespace: payments
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get"]
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 17. ServiceAccount

Every workload should use a deliberate ServiceAccount. Do not assume the
default ServiceAccount should be used for production applications.
Disable unnecessary token automounting where the pod does not need
Kubernetes API access.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 18. Automount Service Account Token

``` yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: payments
  namespace: payments
automountServiceAccountToken: false
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 19. Workload Identity

Use EKS workload identity mechanisms so a pod can receive only the AWS
permissions it needs. The exact mechanism depends on the EKS
architecture and organizational standards; the security principle is the
same: bind AWS permissions to a workload identity rather than embedding
static credentials.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 20. AWS Permissions Boundaries

Permissions boundaries can limit the maximum permissions that IAM roles
created by delegated teams can receive. They are useful in larger
organizations where platform teams need guardrails without centrally
managing every role.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 21. SCP Guardrails

AWS Organizations Service Control Policies can prevent entire classes of
dangerous actions across accounts. SCPs are guardrails rather than
grants: an SCP does not itself provide permission to perform an action.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 22. KMS

Use AWS KMS for encryption of sensitive data where appropriate.
EKS-related data stores, EBS volumes, databases, object storage, and
application secrets can require encryption controls. Separate key
administration from routine application permissions.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 23. Encryption at Rest

Enable encryption for persistent data and understand what key protects
each resource. Document key ownership, rotation, access policy, and
recovery behavior.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 24. Secrets Management

Do not put passwords, API tokens, private keys, or cloud credentials
directly into Git. Prefer a dedicated secrets manager integrated with
Kubernetes. Minimize the number of workloads that can read each secret.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 25. Kubernetes Secrets Are Not a Complete Secrets Strategy

Kubernetes Secrets provide an API object for sensitive configuration,
but base64 encoding is not encryption by itself. Protect etcd encryption
at rest and tightly control RBAC. For high-value secrets, integrate with
an external secrets management system.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 26. External Secrets Pattern

``` text
AWS Secrets Manager / Parameter Store
                |
                v
       External Secrets Controller
                |
                v
      Kubernetes Secret
                |
                v
             Pod
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 27. Secret Rotation

Rotation should be designed before the first production secret is
created. Determine how the application reloads a changed secret, how old
credentials are revoked, how rollout is triggered, and how failures are
detected.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 28. Secret Exposure Prevention

Never print secrets in CI logs, Kubernetes events, application logs,
shell history, or debugging output. Mask CI variables, restrict secret
access, and scan repositories and container layers for accidental
credentials.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 29. Network Security

Network security should control both north-south and east-west traffic.
Security groups and subnet architecture protect AWS networking, while
Kubernetes NetworkPolicy can restrict pod-to-pod communication.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 30. Security Groups

Use security groups to restrict traffic at AWS network interfaces. Avoid
broad inbound rules such as 0.0.0.0/0 for internal services. Review
node, load balancer, database, and endpoint security-group
relationships.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 31. NetworkPolicy

NetworkPolicy can implement default-deny and explicit allow rules
between pods and namespaces. Verify that the EKS networking
implementation supports the policy features you rely on.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 32. Default-Deny NetworkPolicy

``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
  namespace: payments
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 33. Allow-Only-Required Traffic

After default deny, explicitly allow application ingress from the
required frontend or gateway and egress to required dependencies.
Include DNS egress where needed and account for AWS service endpoints,
observability agents, and external APIs.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 34. NetworkPolicy Limitation

NetworkPolicy is not a replacement for AWS security groups, IAM, service
authentication, or encryption. A pod allowed to connect to another pod
still needs application-level authorization for sensitive operations.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 35. Namespace Isolation

Namespaces provide organizational and authorization boundaries but are
not complete security boundaries by themselves. Combine namespaces with
RBAC, NetworkPolicy, quotas, admission policy, and workload identity.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 36. Pod Security Standards

Use Kubernetes Pod Security Standards or an equivalent policy framework
to prevent dangerous pod configurations. Production namespaces should
generally enforce a restrictive baseline appropriate to the workloads
they run.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 37. Privileged Containers

Privileged containers have broad access to the host and can undermine
node isolation. Avoid privileged mode unless a platform component
genuinely requires it, and isolate such components and review their
security posture carefully.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 38. Host Network

hostNetwork gives a pod access to the node network namespace. It should
be exceptional because it reduces network isolation and can expose
host-level services.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 39. Host PID and IPC

hostPID and hostIPC reduce process and IPC isolation. They should be
prohibited for ordinary application workloads unless there is a
documented platform requirement.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 40. HostPath

hostPath can expose host filesystems to containers. Treat it as high
risk and prohibit it for ordinary applications. Platform agents using
hostPath should be reviewed and constrained.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 41. Linux Capabilities

Drop unnecessary Linux capabilities. Many applications do not need
privileged capabilities. Add back only a narrowly required capability
after validating the workload.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 42. SecurityContext

``` yaml
securityContext:
  runAsNonRoot: true
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  capabilities:
    drop:
      - ALL
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 43. Seccomp

Use an appropriate seccomp profile, commonly RuntimeDefault where
supported and compatible. Seccomp reduces the kernel system-call surface
available to a container.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 44. Read-Only Root Filesystem

A read-only root filesystem reduces persistence options for an attacker.
Applications that need writable space should use an explicitly mounted
emptyDir or another intended writable location.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 45. Non-Root User

Run application processes as a non-root UID whenever possible. Verify
the image, filesystem permissions, startup scripts, and sidecars support
non-root execution.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 46. Distroless and Minimal Images

Minimal images reduce package count and attack surface. They also reduce
the amount of software available to an attacker after compromise. Ensure
production debugging procedures do not depend on shell tools inside the
application image.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 47. Base Image Management

Use trusted, maintained base images. Pin image versions or immutable
digests where practical. Track security advisories and rebuild images
when critical vulnerabilities affect the base layer.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 48. Container Image Scanning

Scan images during CI and continuously where possible. Establish
severity thresholds, exceptions with expiration dates, and ownership for
remediation. A scanner finding is a signal to investigate, not a
substitute for risk assessment.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 49. SBOM

Generate Software Bills of Materials for production artifacts. SBOMs
improve vulnerability response by allowing teams to identify affected
packages across deployed images.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 50. Image Signing

Sign trusted container images and verify signatures at admission where
organizational maturity supports it. Signing establishes provenance and
integrity; it does not prove that the software is vulnerability-free.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 51. Supply Chain Security

Secure the path from source to running workload: source control,
dependencies, build runners, CI credentials, container build, registry,
artifact promotion, deployment manifests, and admission policy.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 52. Immutable Artifacts

Build once and promote the same immutable artifact across environments.
Avoid rebuilding separately for production because the resulting
artifact may differ from the tested version.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 53. ECR Security

Use private ECR repositories for production images, restrict repository
access, enable appropriate scanning, use lifecycle policies, and control
who can push versus pull. Production deployment roles should not
automatically have unrestricted push access.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 54. Registry Separation

Separate development and production artifact permissions. A developer
should not need permission to overwrite production artifacts. Prefer
immutable tags or digest-based deployments.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 55. Tag Immutability

Mutable tags such as latest make incident investigation and rollback
harder. Use versioned tags and preferably image digests for production
deployment references.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 56. Admission Control

Admission control provides a final security gate before Kubernetes
accepts an object. It can enforce image policies, security contexts,
resource requirements, approved registries, labels, and other
organizational controls.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 57. Validating Admission Policy

Use Kubernetes-native admission policy capabilities or an established
policy engine to reject insecure configurations before they reach
production. Start with audit mode, measure violations, then enforce
after remediation.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 58. Policy Examples

Production policy can reject privileged containers, hostPath mounts,
hostNetwork, missing resource requests, unapproved registries, mutable
production image tags, missing labels, or workloads without required
security contexts.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 59. OPA Gatekeeper

OPA Gatekeeper is a policy-as-code option for Kubernetes admission. It
can enforce organizational constraints and report violations. Operate it
as a platform component with controlled templates and testing.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 60. Kyverno

Kyverno provides Kubernetes-native policy definitions and is another
common policy-as-code choice. The important production principle is
consistent policy ownership, testing, versioning, and rollout rather
than the specific product.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 61. Policy Rollout Strategy

Begin policies in audit mode, collect violations, classify exceptions,
remediate applications, then move to enforcement. Emergency bypasses
should be time-limited and auditable.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 62. Pod Security Admission

Pod Security Admission provides built-in controls based on Pod Security
Standards. Use namespace labels deliberately and verify that platform
components with special requirements are isolated from ordinary
application namespaces.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 63. Resource Quotas as Security

ResourceQuota and LimitRange reduce the ability of one namespace or
workload to consume unlimited cluster resources. They can mitigate
accidental or malicious resource exhaustion.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 64. ResourceQuota Example

``` yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: payments
spec:
  hard:
    pods: "50"
    requests.cpu: "20"
    requests.memory: 40Gi
    limits.cpu: "40"
    limits.memory: 80Gi
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 65. LimitRange Example

``` yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: defaults
  namespace: payments
spec:
  limits:
    - type: Container
      defaultRequest:
        cpu: 100m
        memory: 128Mi
      default:
        cpu: 500m
        memory: 512Mi
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 66. API Server Authorization Review

Periodically review who has cluster-admin, namespace-admin, secret-read,
workload-write, and deployment permissions. The most dangerous
permissions are often the ability to create pods, modify workloads, read
secrets, or create RBAC bindings.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 67. Why Pod Creation Is Powerful

A principal that can create a pod may be able to mount ServiceAccount
credentials, access secrets permitted to that identity, use host-level
features if policy permits, or assume a workload identity. Pod creation
permissions should therefore be treated as powerful.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 68. Secrets RBAC

Reading Secrets is highly sensitive. Avoid granting list/watch on
Secrets unless required because list access can expose many secret
values. Prefer narrowly scoped get access where possible.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 69. RoleBinding Escalation

RBAC administration can lead to privilege escalation. Restrict who can
create or modify Roles, ClusterRoles, RoleBindings, and
ClusterRoleBindings.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 70. Impersonation

Impersonation permissions should be tightly restricted because they
allow a caller to test or exercise another identity's authorization
context.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 71. Audit Logging

Enable and centralize Kubernetes audit information appropriate to the
organization's security and retention requirements. Audit events can
help identify unauthorized changes, secret access, RBAC modifications,
and suspicious workload creation.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 72. AWS CloudTrail

Use CloudTrail for AWS API activity. Correlate CloudTrail with
Kubernetes audit logs, VPC flow information, load balancer logs, and
application telemetry during investigations.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 73. VPC Flow Logs

VPC Flow Logs can help investigate network connectivity and unexpected
traffic patterns. They are especially useful when distinguishing
Kubernetes policy problems from AWS security-group or routing problems.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 74. ALB Security

Expose only the required listeners and paths. Use HTTPS, managed
certificates, appropriate security groups, and WAF controls where the
threat model requires them. Avoid exposing administrative endpoints
publicly.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 75. TLS

Use TLS for external traffic and consider encryption between internal
services when threat requirements justify it. Certificate lifecycle
management must be automated or operationally reliable.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 76. mTLS

Mutual TLS can provide workload identity and encrypted
service-to-service communication, commonly through a service mesh or
application-level implementation. Introduce it when the security
requirements justify the operational complexity.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 77. WAF

An AWS WAF layer can block common web attack patterns, abusive requests,
and known malicious signatures before traffic reaches workloads. Tune
managed and custom rules to avoid blocking legitimate traffic.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 78. Egress Security

Unrestricted pod egress can make data exfiltration easier after
compromise. Use NetworkPolicy, VPC controls, DNS controls, proxying, or
endpoint policies according to the environment's requirements.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 79. DNS Security

Protect internal DNS and monitor unusual DNS behavior. Compromised
workloads can use DNS for command-and-control or data exfiltration, so
DNS telemetry can be valuable during incident response.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 80. AWS PrivateLink and VPC Endpoints

Use private connectivity to AWS services where appropriate. VPC
endpoints can reduce the need for internet egress for AWS API access and
can be combined with endpoint policies.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 81. Node Security

Treat worker nodes as part of the security boundary. Keep AMIs and node
components patched, minimize SSH access, use controlled SSM-style
administration where appropriate, restrict metadata access, and monitor
node-level activity.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 82. SSH Access

Avoid routine direct SSH access to production nodes. Prefer controlled
management channels with strong identity, logging, and least privilege.
If SSH is unavoidable, restrict access through network controls and
audited identity mechanisms.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 83. EC2 Instance Metadata

Protect instance metadata access and prevent application containers from
obtaining credentials that belong to the node role. Workload identity
should prevent pods from depending on broad node credentials.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 84. Node IAM Role

Keep node IAM permissions limited to what node agents actually require.
Never use the node role as a convenient source of application AWS
permissions.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 85. Node Patching

Establish a repeatable node upgrade process. Managed node groups or
automated provisioning can reduce drift, but production still needs
version compatibility testing and a controlled rollout strategy.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 86. EKS Version Lifecycle

Track supported Kubernetes and EKS versions and plan upgrades before the
environment approaches end-of-support. Test add-ons such as the VPC CNI,
CoreDNS, kube-proxy, ingress controllers, CSI drivers, policy engines,
and monitoring agents.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 87. Cluster Upgrade Security

Upgrade in stages: validate in non-production, verify add-on
compatibility, test workloads, roll through production capacity, and
monitor authentication, networking, admission, and application behavior.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 88. Container Runtime Security

Use supported container runtime components and monitor runtime events
where available. Runtime security should complement preventive controls
rather than replace them.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 89. Runtime Detection

Runtime detection can identify suspicious process execution, filesystem
access, network behavior, or privilege changes. Select a tool
appropriate to the organization's threat model and operational capacity.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 90. Falco-Style Detection

Kernel or runtime-based detection systems can flag behaviors such as
shell execution inside sensitive containers, unexpected privilege
changes, or access to credential locations. Tune rules to reduce alert
fatigue.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 91. File Integrity

Monitor sensitive paths and unexpected modifications where the
environment requires it. Immutable containers and read-only filesystems
reduce the amount of mutable state that needs protection.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 92. Process Execution Monitoring

Unexpected shells, package managers, network tools, or credential-access
commands inside production application containers can be high-value
signals. Correlate process events with deployment history to distinguish
expected operations from compromise.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 93. Security Monitoring

Build a security dashboard containing authentication failures,
privileged actions, RBAC changes, secret access, admission denials,
image vulnerabilities, suspicious runtime events, unusual network flows,
and AWS API anomalies.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 94. Security Alert Priorities

Prioritize alerts that indicate active compromise or privilege
escalation: cluster-admin changes, unexpected privileged pods, secret
reads by unusual identities, suspicious node activity, disabled security
controls, and unknown workload deployments.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 95. False Positive Management

Security controls become ineffective when every alert is treated
equally. Establish ownership, severity, suppression with justification,
expiration dates for exceptions, and continuous tuning.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 96. CI Security

Secure CI runners, credentials, dependencies, artifact storage, and
build configuration. A compromised CI identity can bypass many runtime
controls by publishing trusted-looking artifacts.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 97. CI IAM

Give CI only the AWS and repository permissions required for its job.
Separate build permissions from production deployment permissions when
possible.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 98. OIDC for CI

Use short-lived federated credentials for CI systems rather than static
AWS access keys. Restrict the trust policy by repository, branch,
environment, or workflow identity where supported.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 99. Dependency Security

Scan direct and transitive dependencies. Lock versions, review
dependency changes, and establish a process for emergency vulnerability
remediation.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 100. Secrets in Git History

Deleting a secret from the current file does not remove it from Git
history. If a secret is committed, revoke and rotate it first, then
clean history if appropriate.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 101. Pre-Commit Secret Scanning

Use developer-side and CI secret scanning to catch accidental
credentials early. Treat scanner findings as production security issues
when the credential could still be valid.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 102. Branch Protection

Protect production configuration branches with required reviews, status
checks, and controlled merge permissions. Separate emergency procedures
from routine bypasses and audit both.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 103. GitOps Security

GitOps repositories are production control planes. Protect repository
permissions, require reviews, scan manifests, sign or attest artifacts
where appropriate, and restrict who can change production application
definitions.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 104. Argo CD Security

Argo CD should use least-privilege repository credentials and
destination permissions. Separate projects by trust boundary and prevent
applications from deploying into namespaces or clusters they should not
control.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 105. Argo CD Project Isolation

Use AppProjects to constrain permitted repositories, destination
clusters/namespaces, and resource kinds. Avoid a single unrestricted
project for every team.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 106. Sync Permissions

Restrict who can create, modify, sync, delete, or override Argo CD
applications. Production synchronization should be traceable to an
approved Git change or controlled emergency procedure.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 107. GitOps Drift as a Security Signal

Unexpected live changes can indicate an operator action, compromised
credential, or unauthorized controller. Investigate significant drift
rather than automatically overwriting it without understanding the
cause.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 108. Helm Security

Treat Helm values and templates as code. Prevent insecure defaults,
untrusted image registries, privileged settings, and secret leakage.
Render manifests in CI and scan the rendered output.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 109. Terraform Security

Terraform state can contain sensitive values. Protect the remote state
backend, encrypt state at rest, restrict access, and never commit
production state files to source control.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 110. Terraform IAM

Terraform execution roles should have only the permissions required for
the infrastructure they manage. Separate plan and apply permissions when
organizational controls require it.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 111. Infrastructure Drift

Unauthorized AWS or Kubernetes changes can create security drift. Use
infrastructure scanning, Terraform plans, Kubernetes GitOps
reconciliation, CloudTrail, and periodic configuration review to detect
unexpected changes.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 112. Policy-as-Code CI Gate

Security policies should be tested before deployment. CI can reject
privileged workloads, insecure network policies, missing resource
constraints, unapproved images, or public services according to
organization standards.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 113. Image Policy Example

``` text
Allowed registry: <approved-private-registry>
Production reference: immutable digest
Required: image signature
Required: vulnerability scan
Denied: latest tag
Denied: untrusted public registry
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 114. Pod Security Policy Checklist

-   [ ] runAsNonRoot where supported
-   [ ] allowPrivilegeEscalation=false
-   [ ] capabilities dropped
-   [ ] read-only root filesystem where possible
-   [ ] no privileged mode
-   [ ] no hostNetwork
-   [ ] no hostPID
-   [ ] no hostIPC
-   [ ] no unnecessary hostPath
-   [ ] seccomp profile configured
-   [ ] resource requests/limits defined

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 115. Namespace Security Baseline

Every application namespace should have an ownership label, Pod Security
enforcement, default-deny network policy where practical, ResourceQuota,
LimitRange, controlled ServiceAccounts, and restricted RBAC.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 116. Production Namespace Example

``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: payments
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 117. Secure Deployment Example

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: payments
  namespace: payments
spec:
  replicas: 3
  selector:
    matchLabels:
      app: payments
  template:
    metadata:
      labels:
        app: payments
    spec:
      serviceAccountName: payments
      automountServiceAccountToken: false
      securityContext:
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: payments
          image: <approved-registry>/payments@sha256:<digest>
          securityContext:
            runAsNonRoot: true
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          resources:
            requests:
              cpu: 250m
              memory: 256Mi
            limits:
              cpu: "1"
              memory: 512Mi
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 118. Secure ServiceAccount with AWS Access

Where the workload needs AWS APIs, bind the workload identity to the
narrow IAM role required by that application and keep Kubernetes RBAC
permissions separate. Do not grant broad Kubernetes API access simply
because the pod needs an AWS API.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 119. RBAC Application Pattern

``` yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: payments-reader
  namespace: payments
subjects:
  - kind: ServiceAccount
    name: payments
    namespace: payments
roleRef:
  kind: Role
  name: app-reader
  apiGroup: rbac.authorization.k8s.io
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 120. NetworkPolicy Production Pattern

``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: payments-ingress
  namespace: payments
spec:
  podSelector:
    matchLabels:
      app: payments
  policyTypes:
    - Ingress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: frontend
      ports:
        - protocol: TCP
          port: 8080
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 121. Security Context Testing

Test security restrictions before enforcement. A secure context can
break applications that write to arbitrary paths, bind privileged ports,
require root, use Linux capabilities, or expect a writable filesystem.
Fix the application rather than weakening cluster security by default.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 122. Ephemeral Debugging

Production debugging should not require weakening the application's
security context. Use controlled ephemeral containers or dedicated
debugging workflows with explicit RBAC and auditability.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 123. Break-Glass Access

Maintain an emergency access process for incidents where normal GitOps
or RBAC paths are unavailable. Break-glass credentials should be
strongly protected, time-limited where possible, logged, reviewed after
use, and rotated if exposed.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 124. Incident: Compromised Pod

1.  Identify the workload and namespace. 2. Preserve relevant
    evidence. 3. Restrict network communication. 4. Revoke or rotate
    exposed credentials. 5. Scale or isolate the workload as
    appropriate. 6. Inspect image and runtime activity. 7. Determine
    lateral movement. 8. Redeploy from a trusted artifact. 9. Review
    persistence and RBAC changes. 10. Document root cause.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 125. Incident: Compromised ServiceAccount

Immediately determine which Kubernetes and AWS permissions the identity
had. Revoke or reduce permissions, rotate associated credentials if
applicable, inspect audit logs, identify all pods using the identity,
and investigate unauthorized API activity.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 126. Incident: Compromised AWS Role

Disable or restrict the affected role, invalidate or rotate relevant
credentials, inspect CloudTrail, identify resources modified by the
role, preserve evidence, and assess whether attacker persistence exists.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 127. Incident: Malicious Image

Stop promotion of the artifact, identify all environments using the
image digest, isolate affected workloads, revoke associated credentials
if the container could access them, replace with a trusted artifact, and
investigate the build and registry path.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 128. Incident: Secret Leak

Assume the secret is compromised. Revoke or rotate it, identify every
system that trusted it, inspect access logs, remove the secret from
source and artifacts where appropriate, and determine whether
unauthorized use occurred.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 129. Incident: Privilege Escalation

Review recent RBAC and admission changes, identify the actor, inspect
created workloads and ServiceAccounts, check for cluster-admin bindings,
investigate node access, and revoke unauthorized permissions.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 130. Incident: Cryptomining

Indicators can include unexplained CPU saturation, unexpected outbound
traffic, unknown pods, suspicious images, or processes. Isolate affected
workloads, inspect admission/audit/runtime data, remove unauthorized
resources, rotate credentials, and identify the initial access path.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 131. Incident: Data Exfiltration

Investigate unusual egress, DNS queries, AWS API access, object-storage
access, database reads, and service-to-service traffic. Contain the
identity and network path first, then establish what data was accessed.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 132. Security Incident Evidence

Preserve Kubernetes audit events, CloudTrail, VPC Flow Logs, application
logs, ALB logs, container/runtime events, Git history, CI logs, image
digests, and timestamps. Keep evidence retention aligned with
organizational requirements.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 133. Forensics Considerations

Do not immediately destroy every artifact before collecting sufficient
evidence when incident response procedures require preservation.
Coordinate containment and evidence collection with the security team.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 134. Recovery from Compromise

Recovery should use trusted source, trusted CI, verified artifacts,
clean infrastructure definitions, rotated credentials, and validated
configuration. Rebuilding from known-good state is often safer than
trying to clean a deeply compromised workload in place.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 135. Security Backups

Backups must also be protected. Encrypt them, restrict access, separate
backup administration from workload administration, and test restore
procedures. A backup accessible to every production workload is not a
strong recovery control.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 136. DR Security

Apply equivalent identity, network, admission, secrets, logging, and
artifact controls to the disaster-recovery environment. A weak secondary
environment can become an attacker entry point.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 137. Multi-Cluster Security

Treat each cluster as a separate trust boundary. Do not assume access to
one cluster should grant access to another. Use separate Argo CD
projects, credentials, AWS roles, network boundaries, and cluster RBAC
where appropriate.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 138. Multi-Account EKS Security

Separate production, staging, security, and shared-services accounts
when justified. Cross-account roles should be tightly scoped and
monitored through CloudTrail.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 139. Multi-Tenant Cluster

If multiple teams share a cluster, strengthen namespace isolation, RBAC,
NetworkPolicy, quotas, admission controls, workload identity, and
resource limits. For high-risk tenants, separate clusters or accounts
may provide a stronger boundary.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 140. Security and Autoscaling

Autoscaling can become a denial-of-service amplifier. Limit maxReplicas,
node provisioning capacity, AWS quotas, queue concurrency, and
downstream connections. Monitor unexpected resource growth and scaling
events.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 141. Security and Ingress

Public ingress is an attack surface. Use TLS, WAF where needed, strict
security groups, controlled routes, authentication/authorization at the
appropriate layer, and application-level validation.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 142. Security and Observability

Logs themselves can contain secrets and personal or sensitive data.
Apply access controls, retention policies, masking, encryption, and
secure transport to observability systems.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 143. Security Dashboard

``` text
Authentication failures
RBAC changes
Cluster-admin grants
Secret reads
Admission denials
Privileged pod attempts
Image vulnerabilities
Image signature failures
Unexpected workloads
Runtime detections
Network anomalies
AWS IAM changes
CloudTrail anomalies
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 144. Security KPIs

Useful security measures include percentage of workloads running
non-root, percentage using approved signed images, critical
vulnerabilities beyond SLA, privileged workloads, cluster-admin
assignments, namespaces with default-deny policies, secrets without
rotation, unpatched cluster components, and unresolved high-severity
findings.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 145. Vulnerability Management

Prioritize vulnerabilities by exploitability, exposure, privileges
required, workload criticality, compensating controls, and availability
of fixes. Establish remediation SLAs and documented risk acceptance.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 146. Patch Management

Patch base images, application dependencies, node OS components, EKS
add-ons, ingress controllers, CSI drivers, policy engines, and security
agents. Maintain an upgrade calendar and emergency patch process.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 147. Security Exceptions

Exceptions should name the workload, exact control being bypassed,
business reason, owner, compensating controls, approval, and expiration
date. Permanent exceptions become undocumented security debt.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 148. Production Security Review

Review architecture before production: IAM, RBAC, endpoint exposure,
network policies, security groups, secrets, image provenance, admission
policy, node access, logging, alerting, backup security, DR, and
incident procedures.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 149. Threat Model Example

``` text
Threat: Compromised web pod
        |
        +--> Kubernetes API?
        |       -> ServiceAccount/RBAC restrict
        |
        +--> AWS APIs?
        |       -> workload IAM restrict
        |
        +--> Other pods?
        |       -> NetworkPolicy restrict
        |
        +--> Host?
        |       -> non-root / no privileged / seccomp
        |
        +--> Internet?
                -> controlled egress
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 150. Security Architecture

``` text
                    Internet
                       |
                    AWS WAF
                       |
                      ALB
                       |
              +--------+--------+
              |                 |
          Public APIs       Internal APIs
              |                 |
              +-------+---------+
                      |
                   EKS Pods
          +-----------+-----------+
          |           |           |
        RBAC      NetworkPolicy  Pod Security
          |           |           |
       Secrets     Service Auth  Runtime
          |
     AWS Workload Identity
          |
     Least-Privilege IAM
          |
       AWS Services
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 151. Terraform Security Controls

Codify security groups, IAM roles, KMS policies, EKS endpoint settings,
logging, encryption, VPC endpoints, and guardrails in Terraform. Use
review and CI policy checks to prevent insecure changes.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 152. Terraform Secret Handling

Never place plaintext production secrets into Terraform source. Be aware
that values can still appear in Terraform state if resources require
them. Protect state and prefer references to external secret systems.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 153. Helm Security Controls

Use Helm schema validation, safe defaults, required values, and policy
scanning. Render charts in CI and inspect the resulting manifests for
privileged settings, public services, mutable images, and missing
security contexts.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 154. CI Security Pipeline

``` text
Commit
  |
Secret Scan
  |
SAST / Dependency Scan
  |
IaC Scan
  |
Container Build
  |
Image Scan + SBOM
  |
Sign / Attest
  |
Policy Validation
  |
Artifact Promotion
  |
GitOps Update
  |
Admission Enforcement
```

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 155. Admission + CI Defense in Depth

CI catches problems before merge or artifact promotion. Admission
catches unsafe configurations that still reach the cluster. Both are
needed because no single pipeline stage can guarantee security.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 156. Production Security Runbook

1.  Validate alert. 2. Identify affected identity/workload. 3. Contain
    access or traffic. 4. Preserve evidence. 5. Rotate/revoke
    credentials. 6. Remove unauthorized artifacts. 7. Rebuild from
    trusted source. 8. Validate controls. 9. Monitor for recurrence. 10.
    Complete post-incident review.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 157. Daily Security Checks

Review critical alerts, unexpected privileged workloads, failed
admission events, IAM/RBAC changes, new critical vulnerabilities,
unusual egress, and production deployment changes.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 158. Weekly Security Checks

Review privileged access, cluster-admin bindings, stale ServiceAccounts,
image vulnerabilities, security exceptions, NetworkPolicy coverage, and
significant CloudTrail activity.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 159. Monthly Security Checks

Review IAM access, secrets rotation, backup security, EKS/add-on
versions, node patching, policy coverage, incident response readiness,
and disaster recovery security.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 160. Quarterly Security Review

Perform a broader threat-model review, access recertification,
penetration/security testing where appropriate, disaster recovery
validation, policy cleanup, and architecture review against new threats
and platform changes.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 161. Security Checklist

-   [ ] Production AWS account isolated appropriately
-   [ ] Root account protected
-   [ ] MFA / federated identity enabled
-   [ ] Least-privilege IAM
-   [ ] EKS endpoint restricted appropriately
-   [ ] RBAC reviewed
-   [ ] Cluster-admin access minimized
-   [ ] ServiceAccounts deliberate
-   [ ] Workload AWS identity restricted
-   [ ] Secrets externalized and encrypted
-   [ ] Secret rotation tested
-   [ ] NetworkPolicy implemented
-   [ ] Security groups reviewed
-   [ ] Pod Security enforced
-   [ ] Non-root workloads
-   [ ] Privilege escalation disabled
-   [ ] Capabilities dropped
-   [ ] Seccomp enabled
-   [ ] Read-only filesystem where possible
-   [ ] Approved registries only
-   [ ] Images scanned
-   [ ] SBOM generated
-   [ ] Images signed/verified where required
-   [ ] Admission policies enforced
-   [ ] Audit logging enabled
-   [ ] CloudTrail enabled
-   [ ] Runtime detection configured
-   [ ] CI credentials short-lived
-   [ ] GitOps protected
-   [ ] Terraform state protected
-   [ ] Security exceptions expire
-   [ ] Incident runbooks tested
-   [ ] DR security validated

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 162. Senior Interview: How Do You Secure EKS?

I use defense in depth: AWS IAM and account boundaries, restricted EKS
API access, Kubernetes RBAC, workload identity, least-privilege
ServiceAccounts, NetworkPolicy, Pod Security Standards, secure container
images, admission policy, encrypted secrets, audit logging, runtime
detection, and tested incident response.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 163. Senior Interview: RBAC vs IAM

IAM controls access to AWS APIs and resources; Kubernetes RBAC controls
access to Kubernetes resources. They solve different authorization
problems. A workload may need an AWS role without needing Kubernetes API
permissions, and vice versa.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 164. Senior Interview: Why Is Cluster-Admin Dangerous?

Cluster-admin can modify workloads, RBAC, secrets, and cluster-level
resources. In practice, permissions such as pod creation, secret access,
and RBAC modification can also create escalation paths, so I review them
as high-risk permissions.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 165. Senior Interview: How Do You Secure Pods?

I run workloads as non-root, disable privilege escalation, drop
capabilities, use seccomp, prefer read-only root filesystems, prohibit
privileged and host-level access, enforce resource constraints, use
approved images, and apply admission policies.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 166. Senior Interview: How Do You Secure Secrets?

I keep secrets out of Git, use a dedicated secrets manager and
controlled Kubernetes integration, encrypt data at rest, restrict secret
RBAC, rotate credentials, prevent log exposure, and test the rotation
process.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 167. Senior Interview: How Do You Stop Lateral Movement?

I combine NetworkPolicy with namespace isolation, least-privilege
ServiceAccounts, restricted workload IAM, Pod Security, and
application-level authentication. Network controls alone are not
sufficient.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 168. Senior Interview: What If a Pod Is Compromised?

I contain the workload and its identities, preserve evidence, rotate
exposed credentials, inspect Kubernetes and AWS audit logs, investigate
lateral movement, replace the workload from a trusted immutable
artifact, and validate that the initial access path has been closed.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 169. Senior Interview: How Do You Secure CI/CD?

I use short-lived federated credentials, protected branches,
least-privilege deployment roles, secret scanning, dependency scanning,
IaC scanning, image scanning, SBOMs, artifact signing, immutable
artifacts, GitOps review controls, and Kubernetes admission enforcement.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 170. Senior Interview: How Do You Prevent Privileged Pods?

I enforce Pod Security Standards and admission policies, then
continuously monitor for violations. I use exceptions only when a
platform component genuinely requires elevated access and make those
exceptions explicit and reviewable.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 171. Senior Interview: How Do You Secure Multi-Cluster EKS?

I treat each cluster as a separate trust boundary, use separate
credentials and RBAC, isolate AWS roles and network paths, constrain
Argo CD projects, and avoid assuming that access to one cluster should
imply access to another.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 172. Senior Interview: Security Incident Example

A production pod was found making unexpected outbound connections. I
would first contain egress and preserve evidence, identify the
ServiceAccount and AWS role, inspect audit and runtime events, check the
image digest and CI history, rotate exposed credentials, redeploy from a
trusted artifact, and perform a root-cause review.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

## 173. Final Production Security Principles

1.  Assume every identity is potentially compromised.
2.  Minimize permissions before relying on detection.
3.  Treat pod creation and secret access as powerful privileges.
4.  Keep AWS IAM and Kubernetes RBAC responsibilities separate.
5.  Reduce network reachability.
6.  Reduce container privileges.
7.  Use trusted immutable artifacts.
8.  Enforce policy before workloads become incidents.
9.  Centralize and correlate security telemetry.
10. Make credential rotation and recovery routine.
11. Protect the CI/CD and GitOps control plane.
12. Test security controls under failure, not only during audits.

### Production validation

-   Verify the control is actually enforced, not merely configured.
-   Test both the expected path and the denied path.
-   Confirm logs and alerts identify violations.
-   Document ownership, exceptions, and recovery procedures.

### Operator questions

1.  What identity is making this request?
2.  What is the minimum permission required?
3.  What happens if that identity is compromised?
4.  What network paths remain available?
5.  Can the workload access secrets or AWS APIs unnecessarily?
6.  How would we detect and contain abuse?

# Appendix A --- Production Security Validation Commands

> Commands below are examples for authorized cluster administration and
> security validation. Adapt namespaces, names, and contexts to the
> environment.

## A.1 Cluster Context

``` bash
kubectl config current-context
kubectl cluster-info
kubectl get nodes -o wide
kubectl version
```

## A.2 RBAC Inspection

``` bash
kubectl get clusterrolebindings
kubectl get rolebindings -A
kubectl auth can-i --list
kubectl auth can-i get secrets -n payments
kubectl auth can-i create pods -n payments
```

## A.3 Workload Security Review

``` bash
kubectl get pods -A -o wide
kubectl get deploy -A
kubectl get sa -A
kubectl get networkpolicy -A
kubectl get resourcequota -A
kubectl get limitrange -A
```

## A.4 Admission and Security Context Review

``` bash
kubectl get ns --show-labels
kubectl get pods -n payments -o yaml
kubectl describe pod <pod> -n payments
```

## A.5 Events

``` bash
kubectl get events -A --sort-by=.lastTimestamp
kubectl get events -n payments --sort-by=.lastTimestamp
```

## A.6 Image Verification

``` bash
kubectl get pods -n payments -o jsonpath='{range .items[*].spec.containers[*]}{.image}{"
"}{end}'
```

## A.7 NetworkPolicy Review

``` bash
kubectl get networkpolicy -A
kubectl describe networkpolicy -n payments
```

# Appendix B --- Example Security Policy Matrix

  Control                      Development   Staging         Production
  ---------------------------- ------------- --------------- -------------------
  Pod Security                 Warn          Audit/Enforce   Enforce
  Default Deny NetworkPolicy   Recommended   Required        Required
  Signed Images                Recommended   Required        Required
  Image Scan                   Required      Required        Required
  Secret Manager               Required      Required        Required
  Non-root                     Required      Required        Required
  Privileged Pods              Exception     Exception       Exception
  Cluster-admin                Restricted    Restricted      Break-glass
  Public API Endpoint          Controlled    Controlled      Minimized
  Audit Logging                Required      Required        Required
  Runtime Detection            Recommended   Required        Required
  Security Exceptions          Tracked       Tracked         Expiring approval

# Appendix C --- Security Ownership Model

  Area                Primary Owner     Review
  ------------------- ----------------- --------------------
  AWS IAM             Cloud/Platform    Security
  EKS RBAC            Platform          Security
  Pod Security        Platform          Security/App Teams
  Images              Dev Teams         Security/Platform
  CI Security         DevOps            Security
  GitOps              Platform/DevOps   Security
  NetworkPolicy       App/Platform      Security
  Secrets             App/Platform      Security
  Incident Response   Security/SRE      Engineering
  DR Security         Platform/SRE      Security

# Appendix D --- Final Production Security Gate

A production workload should not be considered ready until the team can
answer:

1.  Which AWS role can this workload assume?
2.  Which Kubernetes API permissions does its ServiceAccount have?
3.  Which namespaces can it reach?
4.  Which external destinations can it reach?
5.  Which secrets can it read?
6.  Which container image digest is deployed?
7.  How was the artifact built and promoted?
8.  Can the container run as non-root?
9.  What happens if the pod is compromised?
10. What detects suspicious behavior?
11. How are credentials rotated?
12. Who can deploy or change the workload?
13. How is unauthorized drift detected?
14. How is the workload recovered after compromise?
15. What evidence is retained for an investigation?

# Appendix E --- Final Architecture Review

The production security architecture should form a chain:

``` text
Trusted Source
      |
      v
Protected Git
      |
      v
Secure CI
  |   |   |
  |   |   +--> Secret Scan
  |   +------> Dependency / IaC Scan
  +----------> Image Scan / SBOM
      |
      v
Signed Immutable Artifact
      |
      v
Protected Registry
      |
      v
GitOps
      |
      v
Admission Policy
      |
      v
Hardened Pod
      |
      +--> Kubernetes RBAC
      |
      +--> AWS Workload Identity
      |
      +--> NetworkPolicy
      |
      +--> Pod Security
      |
      v
Protected Dependencies
      |
      v
Security Telemetry
      |
      v
Detection -> Response -> Recovery
```

# Final Takeaway

Production Kubernetes security is strongest when every layer assumes the
previous layer can fail. IAM limits cloud access. RBAC limits Kubernetes
access. NetworkPolicy limits reachability. Pod Security limits kernel
and container privileges. Image controls limit supply-chain risk.
Admission blocks unsafe configurations. Runtime detection identifies
suspicious behavior. Audit and CloudTrail provide evidence. Incident
response and rebuild procedures provide recovery.

The objective is not to make compromise mathematically impossible. The
objective is to make unauthorized access difficult, blast radius small,
detection fast, containment practical, and recovery repeatable.
