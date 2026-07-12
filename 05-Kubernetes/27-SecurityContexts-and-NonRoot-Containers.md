# Security Contexts and Non-Root Containers

## Introduction

Containers are isolated environments.

---

However isolation does not automatically mean:

```text
Secure
```

---

By default many container images run as:

```text
Root User
```

---

Running applications as root introduces:

```text
Security Risks

Privilege Escalation

Host Access Risks

Compliance Issues
```

---

Kubernetes provides:

```text
SecurityContext
```

to control container and pod security settings.

---

# Why Security Matters?

Suppose an attacker gains access to:

```text
Application Container
```

---

If container runs as:

```text
Root User
```

attacker may gain:

```text
Administrative Access

System Access

Sensitive Information
```

---

This violates:

```text
Principle Of Least Privilege
```

---

Best Practice

```text
Run Containers
As Non-Root Users
```

---

# Linux Users

Linux provides:

```text
Root User

Normal Users
```

---

Root User

```text
UID 0
```

---

Capabilities

```text
Full Access

Install Software

Modify Files

Change Permissions
```

---

Normal User

```text
Limited Permissions
```

---

Safer for applications.

---

# Containers and Root Users

Many container images start as:

```text
Root User
```

---

Example

```dockerfile
FROM nginx
```

---

Container may run as:

```text
UID 0
```

unless explicitly changed.

---

# What is SecurityContext?

SecurityContext is a Kubernetes configuration that defines:

```text
User Permissions

Group Permissions

Filesystem Permissions

Security Restrictions
```

for Pods and Containers.

---

Think of SecurityContext as:

```text
Security Policy
For Containers
```

---

# SecurityContext Architecture

```text
Pod
 │
 └── SecurityContext
          │
          ├── User
          ├── Group
          ├── Privileges
          └── Filesystem Access
```

---

# Pod Level SecurityContext

Example

```yaml
spec:

  securityContext:

    runAsUser: 1000
```

---

Meaning

```text
Run All Containers
As User 1000
```

---

# Container Level SecurityContext

Example

```yaml
containers:

- name: app

  image: nginx

  securityContext:

    runAsUser: 1000
```

---

Applies only to:

```text
Specific Container
```

---

# runAsUser

Defines:

```text
Linux User ID
```

for container execution.

---

Example

```yaml
securityContext:

  runAsUser: 1000
```

---

Result

```text
Container Runs
As User 1000
```

instead of root.

---

# runAsGroup

Defines:

```text
Linux Group ID
```

---

Example

```yaml
securityContext:

  runAsGroup: 3000
```

---

Result

```text
Container Files
Owned By Group 3000
```

---

# fsGroup

Defines:

```text
Filesystem Group
```

for mounted volumes.

---

Example

```yaml
securityContext:

  fsGroup: 2000
```

---

Useful when:

```text
Applications Need
Volume Access
```

---

# runAsNonRoot

Enforces:

```text
Container Must Not Run
As Root
```

---

Example

```yaml
securityContext:

  runAsNonRoot: true
```

---

Benefits

```text
Additional Protection
```

---

If image tries to run as root:

```text
Pod Fails To Start
```

---

# Privileged Ports

Linux reserves:

```text
0 - 1023
```

as:

```text
Privileged Ports
```

---

Examples

```text
22   SSH

80   HTTP

443  HTTPS
```

---

Traditionally:

```text
Root User
```

binds to these ports.

---

# Why Applications Use 8080?

Modern containerized applications often use:

```text
8080

8443
```

instead of:

```text
80

443
```

---

Reason

```text
Non Root Containers
```

can run safely.

---

Architecture

```text
Service Port 80
        ↓

Target Port 8080
        ↓

Container
```

---

Example

```yaml
ports:

- port: 80

  targetPort: 8080
```

---

Common in:

```text
EKS

AKS

GKE
```

---

# allowPrivilegeEscalation

Controls whether a process can gain:

```text
Additional Privileges
```

---

Best Practice

```yaml
securityContext:

  allowPrivilegeEscalation: false
```

---

Benefits

```text
Reduced Attack Surface
```

---

# readOnlyRootFilesystem

Makes container filesystem:

```text
Read Only
```

---

Example

```yaml
securityContext:

  readOnlyRootFilesystem: true
```

---

Benefits

```text
Prevents File Modifications

Improves Security
```

---

Useful for:

```text
Stateless Applications
```

---

# Capabilities

Linux breaks root privileges into:

```text
Capabilities
```

---

Examples

```text
NET_ADMIN

SYS_ADMIN

NET_BIND_SERVICE
```

---

Instead of full root access:

```text
Grant Specific Capability
```

---

More secure.

---

# Dropping Capabilities

Example

```yaml
securityContext:

  capabilities:

    drop:

    - ALL
```

---

Best Practice

```text
Drop Everything

Add Only What Is Required
```

---

# Complete SecurityContext Example

```yaml
apiVersion: v1

kind: Pod

metadata:
  name: nginx

spec:

  securityContext:

    runAsNonRoot: true

    runAsUser: 1000

    runAsGroup: 3000

    fsGroup: 2000

  containers:

  - name: nginx

    image: nginx

    securityContext:

      allowPrivilegeEscalation: false

      readOnlyRootFilesystem: true

      capabilities:

        drop:

        - ALL
```

---

# Production Example

Frontend Application

Requirements

```text
Non Root User

Read Only Filesystem

Restricted Privileges
```

---

Configuration

```yaml
securityContext:

  runAsNonRoot: true

  runAsUser: 1000

  allowPrivilegeEscalation: false
```

---

Benefits

```text
Improved Security

Compliance

Reduced Risk
```

---

# SecurityContext vs Docker USER

Dockerfile

```dockerfile
USER 1000
```

---

Container Defaults To:

```text
User 1000
```

---

SecurityContext

```yaml
runAsUser: 1000
```

---

Kubernetes Enforces:

```text
Runtime User
```

---

Both can work together.

---

# Production Best Practices

## Use Non Root Containers

Good

```yaml
runAsNonRoot: true
```

---

Avoid

```text
Running As Root
```

---

## Disable Privilege Escalation

```yaml
allowPrivilegeEscalation: false
```

---

## Use Read Only Filesystems

```yaml
readOnlyRootFilesystem: true
```

---

## Drop Unnecessary Capabilities

```yaml
capabilities:
  drop:
  - ALL
```

---

## Use Dedicated Service Accounts

Avoid using:

```text
Default Service Account
```

for production workloads.

---

# Troubleshooting

Check Pod

```bash
kubectl describe pod pod-name
```

---

View Events

```text
Permission Errors

SecurityContext Errors
```

---

Check User

```bash
kubectl exec -it pod-name -- id
```

---

Output

```text
uid=1000
```

---

Verify Filesystem

```bash
kubectl exec -it pod-name -- mount
```

---

# Common Problems

## Container Cannot Start

Cause

```text
runAsNonRoot Enabled

Image Requires Root
```

---

Solution

```text
Build Image
Using Non Root User
```

---

## Permission Denied

Cause

```text
Volume Ownership

Missing fsGroup
```

---

Solution

```yaml
securityContext:

  fsGroup: 2000
```

---

## Application Cannot Write Files

Cause

```text
readOnlyRootFilesystem
```

enabled.

---

Solution

```text
Use Writable Volume
```

for required directories.

---

# Common Interview Questions

## What is a SecurityContext?

### Short Answer

A SecurityContext defines security settings for Pods and containers.

### Detailed Explanation

SecurityContext controls user IDs, group IDs, privileges, filesystem access, and security restrictions applied at runtime.

### Production Example

An EKS frontend application running as user 1000 with privilege escalation disabled.

### Follow-Up Questions

- Can SecurityContext be applied at Pod level?
- Can SecurityContext be applied at Container level?
- What security settings can it control?

---

## What is runAsNonRoot?

### Short Answer

runAsNonRoot ensures a container cannot run as root.

### Detailed Explanation

Kubernetes validates the container user and prevents startup if the container attempts to run as UID 0.

### Production Example

Roboshop frontend deployment configured with:

```yaml
runAsNonRoot: true
```

to meet security requirements.

### Follow-Up Questions

- What happens if the image uses root?
- How does runAsUser relate to runAsNonRoot?

---

## Why Should Containers Not Run As Root?

### Short Answer

Running as root increases security risks and potential attack impact.

### Detailed Explanation

A compromised root container can potentially access sensitive resources and perform privileged operations.

### Production Example

Production EKS workloads commonly run with user IDs such as 1000 instead of UID 0.

### Follow-Up Questions

- What is UID 0?
- What are the risks of root containers?
- How can Kubernetes prevent root execution?

---

## What is allowPrivilegeEscalation?

### Short Answer

Controls whether processes can gain additional privileges.

### Detailed Explanation

Setting it to false prevents processes from obtaining elevated permissions during execution.

### Production Example

Frontend applications running with:

```yaml
allowPrivilegeEscalation: false
```

to reduce attack surface.

### Follow-Up Questions

- Why is privilege escalation dangerous?
- Is it enabled by default?

---

## Why Do Containers Commonly Use Port 8080 Instead Of Port 80?

### Short Answer

Because non-root containers can safely use 8080 without requiring privileged access.

### Detailed Explanation

Ports below 1024 are traditionally privileged ports. Running on 8080 allows applications to run as non-root users.

### Production Example

Frontend container listens on 8080 while Kubernetes Service exposes port 80.

### Follow-Up Questions

- What are privileged ports?
- Can Services expose port 80 while containers use 8080?
- Why is this common in Kubernetes?

---

# Key Takeaways

```text
SecurityContext controls container security settings.

Containers should run as non-root users whenever possible.

runAsUser defines the container user ID.

runAsNonRoot prevents root execution.

allowPrivilegeEscalation should be disabled in production.

readOnlyRootFilesystem improves security.

Capabilities provide granular permissions instead of full root access.

Ports 0-1023 are privileged ports.

Container port 8080 with Service port 80 is a common production pattern.

SecurityContext is a critical Kubernetes security feature.
```