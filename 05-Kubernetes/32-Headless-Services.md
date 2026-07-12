# Headless Services

## Introduction

Earlier we learned:

```text
Services

ClusterIP

NodePort

LoadBalancer

StatefulSets
```

---

A normal Kubernetes Service provides:

```text
Single Entry Point

Load Balancing

Service Discovery
```

---

However stateful applications have different requirements.

---

Examples

```text
MongoDB

Redis

MySQL

PostgreSQL

Cassandra
```

---

These applications need:

```text
Peer Discovery

Replication

Direct Pod Communication
```

---

For this Kubernetes provides:

```text
Headless Service
```

---

# What is a Headless Service?

A Headless Service is a Kubernetes Service without a ClusterIP.

---

Normal Service

```yaml
spec:
  type: ClusterIP
```

---

Headless Service

```yaml
spec:
  clusterIP: None
```

---

Notice

```text
No Cluster IP
```

is assigned.

---

# Why Do We Need Headless Services?

Normal services perform:

```text
Load Balancing
```

---

Example

```text
Pod-1

Pod-2

Pod-3
    ↓

ClusterIP
```

---

Request reaches:

```text
Any Pod
```

---

Application does not know:

```text
Which Pod Responded
```

---

For databases this is not enough.

---

Database clusters need:

```text
Direct Communication

Pod Identification

Replication
```

---

# What is Peer Discovery?

Peer Discovery means:

```text
Finding Other Members

Inside The Same Cluster
```

---

Stateful applications need to know:

```text
Who Is Available

Who Is Primary

Who Is Secondary

Who Is Replica
```

---

Examples

```text
MongoDB Replica Set

Redis Cluster

MySQL Cluster
```

---

This process is called:

```text
Peer Discovery
```

---

# Problem With Normal Services

Suppose

```text
mongodb-0

mongodb-1

mongodb-2
```

---

Using normal service

```text
mongodb-service
```

---

Application sees only:

```text
One ClusterIP
```

---

Result

```text
Cannot Identify
Individual Members
```

---

Replication becomes difficult.

---

# Solution: Headless Service

Headless Service returns:

```text
All Pod IP Addresses
```

instead of:

```text
Single Cluster IP
```

---

Architecture

```text
mongodb-0

mongodb-1

mongodb-2
```

---

Headless Service returns:

```text
IP-1

IP-2

IP-3
```

---

Applications can communicate directly.

---

# Headless Service Architecture

```text
StatefulSet
      │

      ├── mongodb-0

      ├── mongodb-1

      └── mongodb-2

             │

             ▼

      Headless Service

             │

             ▼

    DNS Records Created
```

---

Each Pod gets:

```text
Stable DNS Name
```

---

# DNS Records

Headless Service creates:

```text
Individual DNS Records
```

---

Example

```text
mongodb-0.mongodb

mongodb-1.mongodb

mongodb-2.mongodb
```

---

Applications can directly connect to:

```text
Specific Pod
```

---

# Normal Service vs Headless Service

## Normal Service

Returns

```text
One Cluster IP
```

---

Example

```text
10.96.10.20
```

---

Traffic is:

```text
Load Balanced
```

---

## Headless Service

Returns

```text
All Pod IPs
```

---

Example

```text
10.1.1.10

10.1.1.11

10.1.1.12
```

---

No load balancing.

---

Direct communication.

---

# Headless Service YAML

```yaml
apiVersion: v1

kind: Service

metadata:
  name: mongodb

spec:

  clusterIP: None

  selector:

    app: mongodb

  ports:

  - port: 27017
```

---

Important

```yaml
clusterIP: None
```

---

This makes the service:

```text
Headless
```

---

# Verify Headless Service

Create

```bash
kubectl apply -f headless-service.yaml
```

---

Check

```bash
kubectl get svc
```

---

Output

```text
mongodb
```

---

Describe

```bash
kubectl describe svc mongodb
```

---

Notice

```text
ClusterIP: None
```

---

# StatefulSet Integration

Headless Services are commonly used with:

```text
StatefulSets
```

---

Example

```yaml
spec:

  serviceName: mongodb
```

---

This links:

```text
StatefulSet

+

Headless Service
```

---

# MongoDB Example

Replica Set

```text
mongodb-0

mongodb-1

mongodb-2
```

---

Each member discovers:

```text
Other Members
```

through DNS.

---

Example

```text
mongodb-0.mongodb

mongodb-1.mongodb

mongodb-2.mongodb
```

---

Replication works correctly.

---

# Do Stateful Applications Need ClusterIP?

Usually yes.

---

Production setup often contains:

```text
ClusterIP Service

+

Headless Service
```

---

ClusterIP

```text
Application Access
```

---

Headless Service

```text
Peer Discovery
```

---

# Example Architecture

```text
Application
      │

      ▼

ClusterIP Service
      │

      ▼

MongoDB StatefulSet
```

---

Peer Discovery

```text
mongodb-0.mongodb

mongodb-1.mongodb

mongodb-2.mongodb
```

through Headless Service.

---

# Production Example

MongoDB Replica Set

Requirements

```text
Replication

Member Discovery

Stable DNS Names
```

---

Solution

```text
StatefulSet

+

Headless Service
```

---

Benefits

```text
Automatic Discovery

Reliable Replication

Stable Communication
```

---

# Common Problems

## DNS Resolution Failure

Cause

```text
Headless Service Missing
```

---

Verify

```bash
kubectl get svc
```

---

Check

```text
ClusterIP: None
```

---

## Pod Cannot Discover Members

Cause

```text
Wrong Service Name
```

---

Verify

```yaml
serviceName:
```

in StatefulSet.

---

## Replication Failure

Cause

```text
DNS Records Missing
```

---

Check

```bash
nslookup mongodb-0.mongodb
```

---

# Troubleshooting

List Services

```bash
kubectl get svc
```

---

Describe Service

```bash
kubectl describe svc mongodb
```

---

Check DNS

```bash
nslookup mongodb-0.mongodb
```

---

Check StatefulSet

```bash
kubectl get sts
```

---

Verify Endpoints

```bash
kubectl get endpoints
```

---

# Common Interview Questions

## What is a Headless Service?

### Short Answer

A Headless Service is a Service created with:

```yaml
clusterIP: None
```

that provides direct access to individual Pod IPs.

### Detailed Explanation

Unlike normal services, Headless Services do not create a ClusterIP. Instead they return the IP addresses of all matching Pods.

### Production Example

MongoDB Replica Sets use Headless Services for communication between database members.

### Follow-Up Questions

- What is clusterIP: None?
- Does Headless Service perform load balancing?
- What DNS records are created?

---

## Why Do Stateful Applications Need Headless Services?

### Short Answer

To perform peer discovery and communicate directly with individual Pods.

### Detailed Explanation

Stateful applications need to know the identity of every cluster member for replication and synchronization.

### Production Example

MongoDB replicas discover each other through DNS names created by a Headless Service.

### Follow-Up Questions

- What is peer discovery?
- Why is ClusterIP not enough?
- Which applications use Headless Services?

---

## What is Peer Discovery?

### Short Answer

Peer Discovery is the process of finding other members in a distributed cluster.

### Detailed Explanation

Cluster members continuously identify and communicate with each other for replication, leader election, and synchronization.

### Production Example

MongoDB Replica Sets identify primary and secondary nodes through peer discovery.

### Follow-Up Questions

- Why do databases need peer discovery?
- Which Kubernetes resource enables it?
- Does Deployment require peer discovery?

---

## Difference Between ClusterIP and Headless Service?

### Short Answer

ClusterIP provides a single virtual IP, while Headless Service provides individual Pod IPs.

### Detailed Explanation

ClusterIP performs load balancing. Headless Service returns DNS records for each Pod and enables direct communication.

### Production Example

Frontend applications use ClusterIP, while MongoDB StatefulSets use Headless Services.

### Follow-Up Questions

- Which service performs load balancing?
- Which service supports peer discovery?
- Can both be used together?

---

## Why Are Headless Services Commonly Used With StatefulSets?

### Short Answer

Because StatefulSets require stable network identities for stateful applications.

### Detailed Explanation

Headless Services create predictable DNS names that match StatefulSet Pod identities.

### Production Example

MongoDB StatefulSet Pods are reachable as:

```text
mongodb-0.mongodb

mongodb-1.mongodb

mongodb-2.mongodb
```

### Follow-Up Questions

- What is a stable hostname?
- How does StatefulSet preserve identity?
- What happens if a Pod restarts?

---

# Key Takeaways

```text
Headless Service is created using clusterIP: None.

Headless Service does not perform load balancing.

It returns individual Pod IP addresses.

Headless Services enable peer discovery.

Peer discovery is critical for distributed databases.

StatefulSets commonly use Headless Services.

Headless Services create stable DNS records.

MongoDB, Redis, MySQL, and Cassandra commonly use Headless Services.

Production stateful applications often use both ClusterIP and Headless Services.
```