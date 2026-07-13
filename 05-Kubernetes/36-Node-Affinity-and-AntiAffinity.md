# Node Affinity and Anti-Affinity

## Introduction

In the previous section we learned:

```text
NodeSelector
```

---

NodeSelector allows Pods to run on nodes with:

```text
Matching Labels
```

---

Example

```yaml
nodeSelector:

  project: roboshop
```

---

However NodeSelector has limitations.

---

It only supports:

```text
Exact Match
```

---

It cannot perform:

```text
OR Logic

AND Logic

Preferences

Advanced Scheduling Rules
```

---

To solve these limitations Kubernetes provides:

```text
Node Affinity
```

---

# What is Node Affinity?

Node Affinity is:

```text
Advanced Version

Of NodeSelector
```

---

It allows Pods to:

```text
Select Nodes

Using Complex Rules
```

---

Examples

```text
Run On GPU Nodes

OR

Run On High Memory Nodes
```

---

# Why Do We Need Node Affinity?

Suppose cluster contains:

```text
Node-1

project=roboshop

env=prod
```

---

```text
Node-2

project=roboshop

env=dev
```

---

```text
Node-3

project=test

env=prod
```

---

Requirement

```text
Run Pod

Only On

project=roboshop

AND

env=prod
```

---

NodeSelector becomes difficult.

---

Node Affinity solves this.

---

# Affinity Types

Kubernetes provides:

```text
Node Affinity

Pod Affinity

Pod Anti-Affinity
```

---

# Node Affinity

Node Affinity works with:

```text
Node Labels
```

---

Scheduler evaluates:

```text
Node Labels

Against Rules
```

---

# Required Affinity

Meaning

```text
Must Match
```

---

If rule is not satisfied:

```text
Pod Remains Pending
```

---

Official Name

```text
requiredDuringSchedulingIgnoredDuringExecution
```

---

Think of it as:

```text
Mandatory Rule
```

---

# Preferred Affinity

Meaning

```text
Try To Match
```

---

But not mandatory.

---

Official Name

```text
preferredDuringSchedulingIgnoredDuringExecution
```

---

Think of it as:

```text
Nice To Have
```

---

# AND Logic

Example

```text
project=roboshop

AND

env=prod
```

---

Node must satisfy:

```text
Both Conditions
```

---

Example

```text
Node-1

project=roboshop

env=prod
```

---

Eligible

---

```text
Node-2

project=roboshop

env=dev
```

---

Rejected

---

# OR Logic

Example

```text
project=roboshop

OR

project=ecommerce
```

---

Node can satisfy:

```text
Any One Condition
```

---

Eligible

```text
roboshop
```

or

```text
ecommerce
```

---

# Operators

Node Affinity supports:

```text
In

NotIn

Exists

DoesNotExist

Gt

Lt
```

---

# In Operator

Example

```yaml
operator: In
```

---

Meaning

```text
Value Must Match
```

---

Example

```text
project

In

roboshop
```

---

# NotIn Operator

Meaning

```text
Value Must Not Match
```

---

Example

```text
project

NotIn

test
```

---

# Exists Operator

Meaning

```text
Label Must Exist
```

---

Example

```text
hardware
```

exists.

---

Value doesn't matter.

---

# Node Affinity Example

```yaml
apiVersion: v1

kind: Pod

metadata:

  name: nginx

spec:

  affinity:

    nodeAffinity:

      requiredDuringSchedulingIgnoredDuringExecution:

        nodeSelectorTerms:

        - matchExpressions:

          - key: project

            operator: In

            values:

            - roboshop

  containers:

  - name: nginx

    image: nginx
```

---

Meaning

```text
Schedule Pod

Only On

project=roboshop Nodes
```

---

# AND Condition Example

```yaml
matchExpressions:

- key: project

  operator: In

  values:

  - roboshop

- key: env

  operator: In

  values:

  - prod
```

---

Meaning

```text
project=roboshop

AND

env=prod
```

---

# OR Condition Example

```yaml
nodeSelectorTerms:

- matchExpressions:

  - key: project

    operator: In

    values:

    - roboshop

- matchExpressions:

  - key: project

    operator: In

    values:

    - ecommerce
```

---

Meaning

```text
roboshop

OR

ecommerce
```

---

# Selecting Nodes Starting With r*

Trainer Requirement

```text
Select Names Starting With r*
```

---

Example Label

```text
hostname=roboshop-node-1

hostname=roboshop-node-2
```

---

Use

```yaml
operator: Exists
```

or

```yaml
operator: In
```

with matching labels.

---

Kubernetes does not directly support:

```text
Regular Expressions
```

for node names.

---

Use labels instead.

---

# Pod Affinity

Node Affinity works on:

```text
Node Labels
```

---

Pod Affinity works on:

```text
Other Pods
```

---

Meaning

```text
Run Near Another Pod
```

---

Example

```text
Frontend

Near Backend
```

---

# Pod Affinity Example

```text
Frontend Pod
```

prefers same node as:

```text
Backend Pod
```

---

Benefits

```text
Reduced Network Latency
```

---

# Pod Anti-Affinity

Opposite of Pod Affinity.

---

Meaning

```text
Do Not Run

Near Another Pod
```

---

# Why Anti-Affinity?

Suppose:

```text
Frontend Replica-1

Frontend Replica-2

Frontend Replica-3
```

all run on same node.

---

Node crashes.

---

Result

```text
Application Down
```

---

Solution

```text
Spread Replicas
```

across nodes.

---

# Pod Anti-Affinity Example

```text
frontend-1

Node-1
```

---

Scheduler avoids placing:

```text
frontend-2
```

on same node.

---

Benefits

```text
High Availability
```

---

# Affinity vs Anti-Affinity

| Feature | Affinity | Anti-Affinity |
|----------|-----------|---------------|
| Goal | Bring Together | Keep Apart |
| Use Case | Frontend + Backend | Replica Distribution |
| Performance | Better Latency | Better Availability |

---

# Production Example

Roboshop

Frontend Pods

```text
project=roboshop
```

---

Requirement

```text
Only Production Nodes
```

---

Affinity Rule

```text
project=roboshop

AND

env=prod
```

---

Result

```text
Pods Run Only

On Production Nodes
```

---

# Production Example - Anti-Affinity

Frontend

```text
3 Replicas
```

---

Anti-Affinity

```text
Do Not Place

On Same Node
```

---

Result

```text
Node-1 → frontend-1

Node-2 → frontend-2

Node-3 → frontend-3
```

---

Improves:

```text
High Availability
```

---

# Common Problems

## Pod Pending

Cause

```text
No Matching Nodes
```

---

Check

```bash
kubectl get nodes --show-labels
```

---

## Affinity Not Working

Cause

```text
Wrong Labels
```

---

Verify

```bash
kubectl describe node
```

---

## Anti-Affinity Scheduling Failure

Cause

```text
Insufficient Nodes
```

---

Example

```text
3 Replicas

1 Node
```

---

Scheduler cannot satisfy rule.

---

# Troubleshooting

View Nodes

```bash
kubectl get nodes --show-labels
```

---

Describe Pod

```bash
kubectl describe pod pod-name
```

---

Check Events

```bash
kubectl get events
```

---

Check Scheduling Errors

```text
Affinity Rules Not Satisfied
```

---

# Common Interview Questions

## What is Node Affinity?

### Short Answer

Node Affinity is an advanced scheduling feature that places Pods on nodes based on label rules.

### Detailed Explanation

Node Affinity extends NodeSelector by supporting operators, AND/OR logic, and preferred scheduling.

### Production Example

Roboshop frontend Pods run only on nodes labeled:

```text
project=roboshop

env=prod
```

### Follow-Up Questions

- How is Node Affinity different from NodeSelector?
- Which operators are supported?
- What are required and preferred affinities?

---

## Difference Between Required and Preferred Affinity?

### Short Answer

Required affinity is mandatory, preferred affinity is optional.

### Detailed Explanation

Required rules must be satisfied for scheduling. Preferred rules influence scheduling but are not mandatory.

### Production Example

Frontend Pods must run on production nodes but preferably on SSD nodes.

### Follow-Up Questions

- What happens if required rules fail?
- Can preferred rules be ignored?
- Which one is stricter?

---

## What is Pod Affinity?

### Short Answer

Pod Affinity schedules Pods close to other Pods.

### Detailed Explanation

Scheduler places Pods near matching Pods to improve communication and reduce network latency.

### Production Example

Frontend Pods run near Backend Pods.

### Follow-Up Questions

- What problem does Pod Affinity solve?
- Does it use node labels?
- How does it improve performance?

---

## What is Pod Anti-Affinity?

### Short Answer

Pod Anti-Affinity prevents Pods from being placed together.

### Detailed Explanation

Scheduler spreads replicas across nodes to improve availability and fault tolerance.

### Production Example

Frontend replicas are distributed across multiple nodes.

### Follow-Up Questions

- Why is Anti-Affinity important?
- What happens if a node fails?
- How does it improve availability?

---

## Node Selector vs Node Affinity?

### Short Answer

Node Selector provides simple matching while Node Affinity provides advanced scheduling rules.

### Detailed Explanation

Node Affinity supports operators, AND/OR logic, and preferences that Node Selector cannot provide.

### Production Example

Node Selector schedules Pods to project nodes, while Node Affinity schedules Pods to production project nodes.

### Follow-Up Questions

- Which one is more flexible?
- Which one supports OR logic?
- Which one is recommended in production?

---

# Key Takeaways

```text
Node Affinity is an advanced version of Node Selector.

Node Affinity uses node labels.

Required affinity is mandatory.

Preferred affinity is optional.

AND and OR logic are supported.

Pod Affinity places related Pods together.

Pod Anti-Affinity separates Pods.

Anti-Affinity improves availability.

Affinity improves performance.

Affinity and Anti-Affinity are important production scheduling features.
```