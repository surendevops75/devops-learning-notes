# Pod Security Standards (PSS)

## Introduction

Kubernetes is extremely powerful.

A Pod can potentially:

```text
Access Host Network

Access Host Filesystem

Run As Root

Run Privileged Containers

Mount Host Directories
```

---

If not controlled properly:

```text
One Compromised Pod

↓

Entire Node Compromised

↓

Entire Cluster Compromised
```

---

This is why Kubernetes introduced:

```text
Pod Security Standards (PSS)
```

---

# Why Pod Security Standards Were Created?

Suppose a developer deploys:

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: hacker-pod

spec:

  hostNetwork: true

  containers:

  - name: nginx

    image: nginx

    securityContext:

      privileged: true

      runAsUser: 0
```

---

This Pod:

```text
Runs As Root

Uses Host Network

Runs In Privileged Mode
```

---

Very Dangerous.

---

Need Security Controls.

---

# What Are Pod Security Standards?

Pod Security Standards (PSS) are Kubernetes security guidelines that define:

```text
What Pods Are Allowed To Do
```

---

Goal

```text
Reduce Attack Surface

Protect Nodes

Protect Cluster
```

---

PSS provides:

```text
Privileged

Baseline

Restricted
```

security levels.

---

# Evolution Of Pod Security

Old Kubernetes

```text
PodSecurityPolicy (PSP)
```

---

Problems

```text
Complex

Difficult To Manage

Hard To Troubleshoot
```

---

PSP Deprecated.

---

Replaced By

```text
Pod Security Standards
```

and

```text
Pod Security Admission Controller
```

---

# Security Levels

## 1. Privileged

Least Secure.

---

Allows Almost Everything.

---

Examples

```text
Root User

Host Network

Privileged Containers

HostPath Volumes
```

---

Suitable For

```text
System Components

CNI Plugins

Monitoring Agents
```

---

Not Recommended For Applications.

---

# Example

Allowed

```yaml
securityContext:

  privileged: true
```

---

Allowed

```yaml
hostNetwork: true
```

---

# 2. Baseline

Moderate Security.

---

Blocks Dangerous Configurations.

---

Allows Most Applications.

---

Examples

```text
Blocks Privileged Containers

Blocks Some Host Access
```

---

Suitable For

```text
Development

QA

General Applications
```

---

# Baseline Prevents

```text
HostPID

HostIPC

Privileged Containers
```

---

But Still Allows

```text
Normal Applications
```

---

# 3. Restricted

Highest Security Level.

---

Recommended For

```text
Production Workloads
```

---

Requires

```text
Non Root User

Read Only Security Controls

Limited Linux Capabilities
```

---

Most Secure Option.

---

# Restricted Requirements

Pods Must:

```text
Run As Non Root

Avoid Privileged Containers

Avoid Host Network

Avoid Host PID

Avoid Host IPC
```

---

# Example

Good

```yaml
securityContext:

  runAsNonRoot: true

  allowPrivilegeEscalation: false
```

---

Bad

```yaml
runAsUser: 0
```

---

Rejected.

---

# Comparing Security Levels

## Privileged

```text
Maximum Freedom

Minimum Security
```

---

## Baseline

```text
Balanced Security
```

---

## Restricted

```text
Maximum Security

Production Recommended
```

---

# Pod Security Admission Controller

PSS Enforcement Happens Through:

```text
Pod Security Admission Controller
```

---

Flow

```text
kubectl apply

↓

API Server

↓

Pod Security Admission

↓

Allow / Reject

↓

ETCD
```

---

# Namespace Based Enforcement

Pod Security Standards are applied using:

```text
Namespace Labels
```

---

Example

Restricted Namespace

```bash
kubectl label namespace production \
pod-security.kubernetes.io/enforce=restricted
```

---

Now Every Pod In Namespace Must Follow:

```text
Restricted Policy
```

---

# Example: Restricted Namespace

Namespace

```text
production
```

---

Label

```text
pod-security.kubernetes.io/enforce=restricted
```

---

Developer Deploys

```yaml
runAsUser: 0
```

---

Result

```text
Rejected
```

---

# Example: Baseline Namespace

```bash
kubectl label namespace dev \
pod-security.kubernetes.io/enforce=baseline
```

---

More Flexible.

---

Suitable For

```text
Development Teams
```

---

# Enforcement Modes

Three Modes Exist.

---

## Enforce

Immediately Reject.

---

Example

```text
Privileged Container

↓

Rejected
```

---

# Audit

Allow Resource.

---

Generate Audit Log.

---

Example

```text
Pod Allowed

↓

Security Warning Logged
```

---

# Warn

Allow Resource.

---

Display Warning To User.

---

Example

```bash
kubectl apply
```

Output

```text
Warning:

Pod violates restricted policy
```

---

# Namespace Example

```yaml
apiVersion: v1
kind: Namespace

metadata:

  name: production

  labels:

    pod-security.kubernetes.io/enforce: restricted

    pod-security.kubernetes.io/audit: restricted

    pod-security.kubernetes.io/warn: restricted
```

---

# Production Example

Financial Application

Requirements

```text
No Root Containers

No Privileged Containers

No Host Access
```

---

Namespace

```text
payments-prod
```

---

Configured With

```text
restricted
```

---

Developers Cannot Deploy Unsafe Pods.

---

# Real World EKS Example

Namespace

```text
roboshop-prod
```

---

Security Requirement

```text
Only Non Root Containers
```

---

PSS Restricted Applied.

---

Developer Tries

```yaml
runAsUser: 0
```

---

Deployment Fails.

---

Security Team Happy.

---

# Why PSS Is Important?

Without PSS

```text
Any Developer

Can Deploy

Dangerous Pods
```

---

With PSS

```text
Cluster Wide Security Controls
```

---

# PSS vs OPA Gatekeeper

## PSS

Built Into Kubernetes.

---

Focused On

```text
Pod Security
```

only.

---

## OPA Gatekeeper

Policy Engine.

---

Can Enforce

```text
Any Kubernetes Policy
```

---

# PSS vs Kyverno

## PSS

Limited To

```text
Pod Security
```

---

## Kyverno

Can Validate

Can Mutate

Can Generate Resources
```

---

More Powerful.

---

# Common Production Problems

## Problem 1

Application Requires Root.

---

Restricted Mode

```text
Rejects Deployment
```

---

Need Security Review.

---

# Problem 2

Third Party Vendor Application

Uses

```text
Privileged Containers
```

---

Restricted Namespace

```text
Deployment Fails
```

---

Need Dedicated Namespace.

---

# Problem 3

Migration From PSP

Policies Behave Differently.

---

Need Testing.

---

# Troubleshooting Commands

Check Namespace Labels

```bash
kubectl get ns --show-labels
```

---

Describe Namespace

```bash
kubectl describe namespace production
```

---

Check Events

```bash
kubectl get events
```

---

Apply Resource

```bash
kubectl apply -f pod.yaml
```

---

Observe Warnings.

---

# Common Interview Questions

## What Are Pod Security Standards?

### Short Answer

Pod Security Standards are Kubernetes security guidelines that define what Pods are allowed to do.

### Detailed Explanation

PSS provides three predefined security levels—Privileged, Baseline, and Restricted—to reduce security risks and standardize workload protection across clusters.

### Production Example

```text
Production Namespace

↓

Restricted Policy

↓

Root Containers Rejected
```

### Follow-Up Questions

- Why was PSP removed?
- How is PSS enforced?
- Where are policies configured?

---

## Difference Between Privileged, Baseline And Restricted?

### Short Answer

Privileged allows almost everything, Baseline blocks dangerous settings, and Restricted enforces strong security controls.

### Detailed Explanation

These levels provide increasing levels of protection and help organizations balance security and operational requirements.

### Production Example

```text
Development

↓

Baseline
```

```text
Production

↓

Restricted
```

### Follow-Up Questions

- Which level is recommended for production?
- Which level allows privileged containers?
- Which level blocks root containers?

---

## How Is PSS Enforced?

### Short Answer

Through the Pod Security Admission Controller.

### Detailed Explanation

The API Server evaluates Pods against namespace security labels and either allows, warns, audits, or rejects requests.

### Production Example

```text
Pod Created

↓

Admission Controller

↓

Policy Check

↓

Allow / Reject
```

### Follow-Up Questions

- What happens during audit mode?
- What happens during warn mode?
- Where are labels configured?

---

## PSS vs OPA Gatekeeper vs Kyverno?

### Short Answer

PSS provides built-in Pod security controls, while OPA Gatekeeper and Kyverno provide broader policy management capabilities.

### Detailed Explanation

PSS focuses only on Pod security. OPA and Kyverno can enforce organization-wide governance, compliance, and operational policies.

### Production Example

```text
PSS

↓

Block Root Containers
```

```text
Kyverno

↓

Require Labels

Require Resource Limits

Enforce Security Rules
```

### Follow-Up Questions

- Which one is built into Kubernetes?
- Which one supports mutation?
- Which one uses Rego?
- Which one uses YAML policies?

---

# Key Takeaways

```text
Pod Security Standards replace PodSecurityPolicy.

PSS defines Privileged, Baseline, and Restricted security levels.

Restricted is recommended for production workloads.

PSS is enforced using the Pod Security Admission Controller.

Policies are applied through namespace labels.

PSS helps prevent root containers, privileged containers, and host-level access.

PSS is built into Kubernetes and provides a simple security baseline.

OPA Gatekeeper and Kyverno extend policy management beyond Pod security.
```