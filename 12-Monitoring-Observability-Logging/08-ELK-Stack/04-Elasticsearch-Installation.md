# Elasticsearch Installation

## 1. Overview

This document covers installing Elasticsearch in environments that can be used for:

* Development
* Testing
* Staging
* Production

The installation approach depends on where Elasticsearch will run.

Common deployment models are:

```text
VM / EC2
   ↓
Package Installation

Kubernetes
   ↓
Operator / Helm-based Deployment

Managed Service
   ↓
Cloud-managed Elasticsearch-compatible service
```

For your real-world DevOps environment, we will understand both **Linux/EC2 installation** and **Kubernetes deployment**, with production architecture covered separately.

---

# 2. Installation Architecture

A simple standalone installation:

```text
Linux Server
     │
     └── Elasticsearch
             │
             ├── Data
             ├── Logs
             └── Configuration
```

A production cluster:

```text
             Elasticsearch Cluster
              /        |        \
             ↓         ↓         ↓
           ES-01     ES-02     ES-03
             │         │         │
            Disk      Disk      Disk
```

---

# 3. Installation Prerequisites

Before installing Elasticsearch, plan:

```text
Operating system
CPU
Memory
Disk
Network
Java/runtime requirements
Elasticsearch version
Cluster topology
Storage
Security
```

Do not start production installation before capacity planning.

---

# 4. Version Planning

Choose a specific Elasticsearch version.

Example:

```text
Elasticsearch 9.x
```

The exact version should be selected based on:

```text
Application compatibility
Kibana compatibility
Logstash compatibility
Operating system support
Official documentation
Organization standards
```

Do not mix arbitrary Elasticsearch, Kibana, and Logstash versions.

---

# 5. Version Compatibility

The core stack is:

```text
Elasticsearch
     ↑
     │
   Logstash

Elasticsearch
     ↓
   Kibana
```

Keep the stack versions compatible.

Before production deployment:

```text
Check Elasticsearch version
Check Kibana version
Check Logstash version
Check plugins
Check integrations
```

---

# 6. Infrastructure Requirements

For a Linux/EC2 installation, determine:

```text
Instance type
CPU
RAM
EBS volume
Private IP
Security groups
Operating system
```

Example development environment:

```text
1 EC2
4 vCPU
8 GB RAM
100 GB storage
```

This is only an example and should not be used as a production sizing recommendation.

---

# 7. Production Infrastructure

A production cluster could look like:

```text
                    VPC
                     │
       ┌─────────────┼─────────────┐
       ↓             ↓             ↓
     ES-01         ES-02         ES-03
       │             │             │
      AZ-A          AZ-B          AZ-C
       │             │             │
     Storage       Storage       Storage
```

Use multiple nodes and appropriate persistent storage for production.

---

# 8. Network Requirements

Elasticsearch nodes need network connectivity between themselves.

Conceptually:

```text
ES-01 ←→ ES-02
  ↑       ↑
  │       │
  └──→ ES-03
```

Clients such as Logstash and Kibana also need connectivity to Elasticsearch.

```text
Logstash ─────→ Elasticsearch
Kibana ───────→ Elasticsearch
```

---

# 9. Network Security

Do not expose Elasticsearch directly to the public internet.

Avoid:

```text
Internet
   ↓
Elasticsearch
```

Prefer:

```text
Private Network
     │
     ├── Logstash
     ├── Kibana
     └── Elasticsearch
```

If users need access, expose Kibana through a controlled endpoint instead.

---

# 10. AWS Security Group Concept

For an AWS deployment:

```text
Logstash SG
     │
     ↓
Elasticsearch SG
```

Allow only required traffic.

Conceptually:

```text
Logstash
   ↓
Elasticsearch HTTP

Elasticsearch Node
   ↔
Elasticsearch Node
```

Do not allow unrestricted access from:

```text
0.0.0.0/0
```

to Elasticsearch ports.

---

# 11. Elasticsearch Ports

Common Elasticsearch ports include:

```text
9200
9300
```

Port `9200` is commonly used for HTTP/API traffic.

Port `9300` is commonly used for internal node-to-node transport.

Example:

```text
Kibana
   ↓
9200
   ↓
Elasticsearch

ES-01
   ↔
9300
   ↔
ES-02
```

The exact network configuration depends on the deployment.

---

# 12. Operating System Preparation

Before installation:

```text
Update packages
Set hostname
Configure networking
Configure time synchronization
Configure storage
Review limits
Review kernel settings
```

Example:

```bash
hostnamectl set-hostname es-01
```

Verify:

```bash
hostname
```

---

# 13. Check System Resources

Before installation:

```bash
free -h
```

Check CPU:

```bash
nproc
```

Check disk:

```bash
df -h
```

Check OS:

```bash
cat /etc/os-release
```

These checks help confirm the server is suitable for the intended environment.

---

# 14. Time Synchronization

Distributed systems depend on consistent system time.

Check:

```bash
timedatectl
```

Ensure time synchronization is enabled.

For example:

```text
ES-01
  10:00:00

ES-02
  10:00:01

ES-03
  10:00:00
```

Small differences can complicate troubleshooting and operational analysis.

---

# 15. DNS / Hostname Planning

Production nodes should have predictable names.

Example:

```text
es-01.internal
es-02.internal
es-03.internal
```

This makes cluster operations easier.

---

# 16. Install Elasticsearch Repository

For Linux package installation, Elasticsearch can be installed using the official package repository.

The repository must match the version you intend to deploy.

The general process is:

```text
Add official repository
        ↓
Refresh package metadata
        ↓
Install Elasticsearch
        ↓
Configure
        ↓
Start service
```

Do not copy repository URLs from random third-party tutorials.

---

# 17. Debian / Ubuntu Installation

On Debian/Ubuntu-based systems, the general workflow is:

```bash
sudo apt-get update
```

Then configure the official Elasticsearch repository according to the Elasticsearch version you selected.

After the repository is configured:

```bash
sudo apt-get update
```

Install:

```bash
sudo apt-get install elasticsearch
```

The exact package and repository instructions should always be taken from the official documentation for the selected version.

---

# 18. RHEL / Amazon Linux Installation

For RPM-based systems, the workflow is:

```text
Add Elasticsearch RPM repository
        ↓
Refresh package metadata
        ↓
Install Elasticsearch
        ↓
Configure
        ↓
Start service
```

For example:

```bash
sudo dnf install elasticsearch
```

The exact repository configuration depends on the Elasticsearch version and operating system.

---

# 19. Verify Installation

After installation:

```bash
rpm -qa | grep elasticsearch
```

or on Debian/Ubuntu:

```bash
dpkg -l | grep elasticsearch
```

Also check:

```bash
systemctl status elasticsearch
```

---

# 20. Elasticsearch Configuration Directory

A package installation commonly places configuration under:

```text
/etc/elasticsearch/
```

Important files include:

```text
elasticsearch.yml
jvm.options
```

The exact layout can vary by installation method and version.

---

# 21. Elasticsearch Data Directory

Package installations commonly use:

```text
/var/lib/elasticsearch/
```

This directory contains Elasticsearch data.

Conceptually:

```text
/var/lib/elasticsearch/
        ↓
      Indexes
        ↓
      Shards
```

Do not store production Elasticsearch data on ephemeral storage.

---

# 22. Elasticsearch Log Directory

Package installations commonly use:

```text
/var/log/elasticsearch/
```

These are Elasticsearch's own operational logs.

Example:

```text
/var/log/elasticsearch/
        ├── server.log
        └── gc.log
```

Exact files depend on the Elasticsearch version and configuration.

---

# 23. Configuration File

The primary configuration file is:

```text
/etc/elasticsearch/elasticsearch.yml
```

This file controls settings such as:

```text
Cluster name
Node name
Network binding
Discovery
Security
Paths
```

---

# 24. Basic Configuration

A basic node configuration conceptually contains:

```yaml
cluster.name: production-logging
node.name: es-01

path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch

network.host: 0.0.0.0
```

Do not blindly use `network.host: 0.0.0.0` in production without appropriate network controls.

---

# 25. Single-Node Development

For a development environment, a single-node configuration may be appropriate.

Conceptually:

```yaml
discovery.type: single-node
```

This tells Elasticsearch that it should operate as a single-node cluster.

Use this for appropriate development/test scenarios, not as a substitute for production cluster design.

---

# 26. Production Cluster

Production should use proper cluster discovery.

Conceptually:

```text
ES-01
ES-02
ES-03
```

with discovery configuration that allows the nodes to form the intended cluster.

The exact discovery configuration depends on the Elasticsearch version and deployment model.

---

# 27. Node Name

Set a unique node name:

```yaml
node.name: es-01
```

Another node:

```yaml
node.name: es-02
```

Another:

```yaml
node.name: es-03
```

Unique names simplify monitoring and troubleshooting.

---

# 28. Cluster Name

Use a meaningful cluster name:

```yaml
cluster.name: production-logging
```

For staging:

```yaml
cluster.name: staging-logging
```

This helps distinguish environments.

---

# 29. Network Binding

Elasticsearch needs to listen on an appropriate network interface.

For local testing:

```yaml
network.host: 127.0.0.1
```

For a private network:

```yaml
network.host: <private-ip>
```

Avoid exposing Elasticsearch directly to the internet.

---

# 30. HTTP Port

The HTTP API commonly listens on:

```text
9200
```

Verify after startup:

```bash
ss -lntp | grep 9200
```

---

# 31. Transport Port

Node-to-node communication commonly uses:

```text
9300
```

Verify:

```bash
ss -lntp | grep 9300
```

The port must be reachable between Elasticsearch nodes where required.

---

# 32. Start Elasticsearch

Enable the service:

```bash
sudo systemctl enable elasticsearch
```

Start it:

```bash
sudo systemctl start elasticsearch
```

Check status:

```bash
sudo systemctl status elasticsearch
```

---

# 33. Check Elasticsearch Logs

If the service fails:

```bash
sudo journalctl -u elasticsearch
```

For more detailed service logs:

```bash
sudo journalctl -u elasticsearch -n 100
```

You can also inspect the Elasticsearch log directory.

---

# 34. Verify Port

Check whether Elasticsearch is listening:

```bash
sudo ss -lntp | grep 9200
```

Expected concept:

```text
LISTEN
   ↓
9200
   ↓
Elasticsearch
```

---

# 35. Test Elasticsearch API

From the Elasticsearch host:

```bash
curl http://localhost:9200
```

A healthy response contains information about:

```text
Cluster
Node
Version
```

With security enabled, the request may require authentication or HTTPS.

---

# 36. Cluster Health API

Check cluster health:

```bash
curl http://localhost:9200/_cluster/health
```

A healthy cluster can report:

```text
green
```

Other states:

```text
yellow
red
```

---

# 37. Node Information

Use:

```bash
curl http://localhost:9200/_cat/nodes?v
```

This helps inspect:

```text
Node
Roles
CPU
Heap
Memory
```

---

# 38. Index Information

Use:

```bash
curl http://localhost:9200/_cat/indices?v
```

This helps identify:

```text
Index
Health
Documents
Storage
```

---

# 39. Shard Information

Use:

```bash
curl http://localhost:9200/_cat/shards?v
```

This helps troubleshoot:

```text
Primary shards
Replica shards
Shard state
Shard allocation
```

---

# 40. Elasticsearch Service Check

A basic validation sequence:

```bash
systemctl status elasticsearch
```

Then:

```bash
ss -lntp | grep 9200
```

Then:

```bash
curl http://localhost:9200
```

Then:

```bash
curl http://localhost:9200/_cluster/health
```

This gives a basic installation validation path.

---

# 41. Security During Installation

Modern Elasticsearch deployments include security capabilities.

Production should use:

```text
Authentication
Authorization
TLS
```

Do not build a production environment around unauthenticated Elasticsearch.

---

# 42. Elasticsearch HTTPS

A secure architecture is:

```text
Client
  ↓
HTTPS
  ↓
Elasticsearch
```

Instead of:

```text
Client
  ↓
HTTP
  ↓
Elasticsearch
```

The exact TLS setup depends on your Elasticsearch version and deployment.

---

# 43. Authentication

After security is configured, clients may need credentials or API keys.

Example concept:

```bash
curl -u elastic https://localhost:9200
```

The exact command depends on your configured authentication method.

Never place real credentials in shell history, scripts, Git repositories, or documentation.

---

# 44. API Keys

For applications and integrations, API keys can be preferable to sharing a high-privilege user credential.

Conceptually:

```text
Logstash
   ↓
API Key
   ↓
Elasticsearch
```

Use the minimum permissions required.

---

# 45. Least Privilege

Do not give Logstash administrative access if it only needs to:

```text
Create indexes
Write documents
Manage required templates
```

Similarly, Kibana users should receive only the permissions they need.

---

# 46. Elasticsearch Storage

Production storage should be planned around:

```text
Daily ingestion
Retention
Replica count
Index overhead
Growth
Failure headroom
```

Example:

```text
200 GB/day
×
30 days
=
6 TB raw
```

This is before replication and overhead.

---

# 47. Storage Type

For AWS EC2:

```text
EC2
 ↓
EBS
 ↓
Elasticsearch
```

Use storage appropriate for the workload.

Important considerations:

```text
IOPS
Throughput
Capacity
Latency
Durability
```

---

# 48. Separate Data From Root Filesystem

For production, it is often preferable to place Elasticsearch data on dedicated persistent storage.

Conceptually:

```text
Root Disk
 ├── OS
 └── Applications

Data Disk
 └── Elasticsearch Data
```

This simplifies storage management.

---

# 49. Mount Data Storage

Example:

```text
/dev/nvme1n1
      ↓
/var/lib/elasticsearch
```

Verify:

```bash
df -h
```

Ensure the Elasticsearch data directory points to the intended storage.

---

# 50. File Ownership

The Elasticsearch service should own its data and log directories.

Check:

```bash
ls -ld /var/lib/elasticsearch
ls -ld /var/log/elasticsearch
```

Do not randomly change permissions.

Use the Elasticsearch service account and package defaults unless your environment requires otherwise.

---

# 51. Elasticsearch User

Package installations normally create an Elasticsearch service account.

Check:

```bash
id elasticsearch
```

The service should not normally run as root.

---

# 52. Running as Root

Avoid:

```text
Elasticsearch → root
```

Prefer:

```text
Elasticsearch
     ↓
elasticsearch user
```

Running stateful infrastructure as root unnecessarily increases security risk.

---

# 53. System Limits

Elasticsearch can require appropriate operating-system limits.

Important areas include:

```text
Open files
Virtual memory
Processes
Memory locking
```

Check current limits:

```bash
ulimit -a
```

Follow the official Elasticsearch requirements for your version rather than copying arbitrary tuning values.

---

# 54. Virtual Memory

Elasticsearch uses memory-mapped files.

Operating-system virtual memory settings may need to be configured appropriately.

A commonly used setting is:

```text
vm.max_map_count
```

Check:

```bash
sysctl vm.max_map_count
```

The required value should follow the Elasticsearch version's official installation guidance.

---

# 55. File Descriptor Limits

Elasticsearch may require a high number of open file descriptors.

Check:

```bash
ulimit -n
```

The service manager's limits may differ from your interactive shell.

Therefore verify the Elasticsearch service configuration rather than relying only on your shell's value.

---

# 56. Memory Planning

Elasticsearch uses JVM heap and also benefits from filesystem cache.

Conceptually:

```text
Server Memory
 ├── JVM Heap
 ├── Filesystem Cache
 └── OS / Other Processes
```

Do not allocate all server memory to Elasticsearch heap.

---

# 57. JVM Configuration

Elasticsearch JVM settings are commonly managed through:

```text
jvm.options
```

and related configuration mechanisms.

Avoid manually editing generated or package-managed files unless the official documentation instructs you to.

---

# 58. Heap Sizing

Heap sizing depends on:

```text
Node size
Workload
Elasticsearch version
Data volume
Query workload
```

Do not blindly copy a heap size from another environment.

For production:

```text
Measure
 ↓
Load test
 ↓
Monitor
 ↓
Tune
```

---

# 59. Garbage Collection

JVM garbage collection manages heap memory.

Excessive GC can indicate:

```text
Heap pressure
Large aggregations
Heavy workload
Poor sizing
```

Monitor:

```text
GC frequency
GC duration
Heap utilization
```

---

# 60. Start Elasticsearch After Configuration

After changing configuration:

```bash
sudo systemctl restart elasticsearch
```

Then immediately check:

```bash
sudo systemctl status elasticsearch
```

And:

```bash
sudo journalctl -u elasticsearch -n 100
```

---

# 61. Validate Configuration Changes

Do not make many changes at once.

A safer process:

```text
Change one setting
     ↓
Restart
     ↓
Check logs
     ↓
Check cluster health
     ↓
Continue
```

This makes troubleshooting easier.

---

# 62. Single-Node Installation Flow

For development:

```text
Install Package
      ↓
Configure
      ↓
discovery.type=single-node
      ↓
Start Service
      ↓
Check API
      ↓
Check Cluster Health
```

---

# 63. Single-Node Architecture

```text
              Elasticsearch
                    │
                 Node-01
                    │
              ┌─────┴─────┐
              ↓           ↓
           Indexes       Shards
```

This is useful for:

```text
Development
Learning
Testing
```

but it does not provide production high availability.

---

# 64. Multi-Node Installation Flow

For production:

```text
Install ES on Node 1
        ↓
Install ES on Node 2
        ↓
Install ES on Node 3
        ↓
Configure cluster
        ↓
Configure discovery
        ↓
Configure networking
        ↓
Configure security
        ↓
Start nodes
        ↓
Validate cluster
```

---

# 65. Multi-Node Architecture

```text
               Production Cluster

        ┌──────────┼──────────┐
        ↓          ↓          ↓
      ES-01      ES-02      ES-03
        │          │          │
       AZ-A       AZ-B       AZ-C
```

The nodes communicate over the private network.

---

# 66. Cluster Formation

When Elasticsearch starts, nodes use the configured discovery mechanism to find the intended cluster.

Conceptually:

```text
ES-01
 ↓
Discover
 ↓
ES-02
 ↓
ES-03
 ↓
Cluster Formation
```

Incorrect discovery configuration can prevent nodes from joining the cluster.

---

# 67. Verify Cluster Membership

After startup:

```bash
curl http://localhost:9200/_cat/nodes?v
```

Expected:

```text
es-01
es-02
es-03
```

The exact output depends on security and version.

---

# 68. Verify Cluster Health

Run:

```bash
curl http://localhost:9200/_cluster/health
```

For a properly configured cluster:

```text
number_of_nodes = expected count
status = healthy
```

A yellow state may be acceptable in some temporary scenarios, but production should be investigated until the intended health state is achieved.

---

# 69. Verify Shards

Run:

```bash
curl http://localhost:9200/_cat/shards?v
```

Check:

```text
STARTED
```

for expected shards.

Look for:

```text
UNASSIGNED
INITIALIZING
RELOCATING
```

when troubleshooting cluster state.

---

# 70. Verify Disk

Run:

```bash
df -h
```

Then inspect Elasticsearch's view:

```bash
curl http://localhost:9200/_cat/allocation?v
```

This helps identify shard and disk allocation.

---

# 71. Verify Node Resources

Check:

```bash
free -h
nproc
df -h
```

Then inspect Elasticsearch node information.

The objective is to confirm:

```text
CPU
Memory
Heap
Disk
```

are within expected ranges.

---

# 72. Create a Test Index

After installation, create a test index:

```bash
curl -X PUT "http://localhost:9200/test-logs"
```

Then verify:

```bash
curl "http://localhost:9200/_cat/indices?v"
```

---

# 73. Insert a Test Document

Example:

```bash
curl -X POST "http://localhost:9200/test-logs/_doc/1" \
  -H 'Content-Type: application/json' \
  -d '{
    "service": "payment",
    "level": "INFO",
    "message": "Elasticsearch installation test"
  }'
```

This verifies document indexing.

---

# 74. Search the Test Document

Run:

```bash
curl "http://localhost:9200/test-logs/_search?q=service:payment"
```

If the document is returned:

```text
Indexing ✓
Search ✓
```

---

# 75. Delete Test Data

After testing:

```bash
curl -X DELETE "http://localhost:9200/test-logs"
```

Verify:

```bash
curl "http://localhost:9200/_cat/indices?v"
```

Do not use delete commands against production indexes without verifying the target carefully.

---

# 76. Installation Validation Checklist

```text
[ ] Package installed
[ ] Service enabled
[ ] Service running
[ ] Port 9200 listening
[ ] Correct hostname
[ ] Correct cluster name
[ ] Correct node name
[ ] Storage mounted
[ ] Permissions correct
[ ] Network configured
[ ] Security configured
[ ] Cluster health verified
[ ] Nodes verified
[ ] Shards verified
[ ] Test indexing verified
[ ] Test search verified
```

---

# 77. Production Installation Checklist

Before production:

```text
[ ] Version selected
[ ] Capacity planned
[ ] Multiple nodes
[ ] Multiple AZs where required
[ ] Persistent storage
[ ] Security enabled
[ ] TLS configured
[ ] Authentication configured
[ ] Authorization configured
[ ] Firewall rules configured
[ ] Backup strategy
[ ] Retention strategy
[ ] Monitoring
[ ] Alerting
[ ] Disaster recovery
[ ] Upgrade strategy
```

---

# 78. Elasticsearch Installation on Kubernetes

For Kubernetes, avoid treating Elasticsearch like a stateless Deployment.

A conceptual architecture is:

```text
Kubernetes
    │
    ↓
Elasticsearch Stateful Workload
    │
 ┌──┼──┐
 ↓  ↓  ↓
ES0 ES1 ES2
│   │   │
PV  PV  PV
```

---

# 79. Kubernetes Deployment Options

Common approaches include:

```text
Elasticsearch Operator
Helm-based deployment
Manual StatefulSet
Managed Elasticsearch service
```

For production, prefer a supported operator or deployment mechanism rather than creating a large custom StatefulSet without understanding Elasticsearch's operational requirements.

---

# 80. Operator Architecture

A Kubernetes operator manages Elasticsearch resources.

Conceptually:

```text
Kubernetes API
      ↓
Elasticsearch Custom Resource
      ↓
Operator
      ↓
Elasticsearch Pods
      ↓
Persistent Volumes
```

The operator can automate parts of:

```text
Deployment
Configuration
Scaling
Certificates
Upgrades
```

The exact capabilities depend on the operator and version.

---

# 81. Kubernetes Namespace

Keep observability components separated.

Example:

```bash
kubectl create namespace logging
```

Then:

```text
logging
 ├── Elasticsearch
 ├── Logstash
 ├── Kibana
 └── Collectors
```

This makes management easier.

---

# 82. Persistent Storage

Elasticsearch requires persistent storage.

Conceptually:

```text
Elasticsearch Pod
       ↓
PersistentVolumeClaim
       ↓
StorageClass
       ↓
Persistent Storage
```

For AWS EKS, this may use an appropriate EBS-backed storage class depending on the cluster configuration.

---

# 83. Stateful Elasticsearch on EKS

Example:

```text
EKS
 │
 ├── ES-0 → PVC → EBS
 ├── ES-1 → PVC → EBS
 └── ES-2 → PVC → EBS
```

Each Elasticsearch node requires appropriate persistent storage.

---

# 84. Availability Zones in EKS

Distribute Elasticsearch nodes across AZs:

```text
AZ-A
 └── ES-0

AZ-B
 └── ES-1

AZ-C
 └── ES-2
```

Use Kubernetes scheduling rules to influence placement.

---

# 85. Pod Anti-Affinity

You generally do not want all Elasticsearch nodes on one Kubernetes worker.

Conceptually:

```text
Worker 1
 └── ES-0

Worker 2
 └── ES-1

Worker 3
 └── ES-2
```

Pod anti-affinity and topology spread constraints can help achieve this.

---

# 86. Resource Requests and Limits

Elasticsearch Pods should have carefully planned resources.

Conceptually:

```yaml
resources:
  requests:
    cpu: ...
    memory: ...
  limits:
    cpu: ...
    memory: ...
```

Do not copy arbitrary values.

Size them based on workload and test results.

---

# 87. JVM Heap in Kubernetes

Container memory and JVM heap must be planned together.

Conceptually:

```text
Pod Memory
 ├── JVM Heap
 ├── Native Memory
 ├── Filesystem Cache
 └── Other Process Memory
```

Setting the JVM heap equal to the entire container memory is unsafe.

---

# 88. Kubernetes Health Checks

Elasticsearch Pods should have appropriate health mechanisms.

Conceptually:

```text
Readiness
   ↓
Can this node receive traffic?

Liveness
   ↓
Is the process still functioning?
```

Do not use aggressive probes that cause healthy Elasticsearch nodes to restart during temporary load.

---

# 89. Elasticsearch Service

Kubernetes provides service discovery.

Conceptually:

```text
Logstash
   ↓
Elasticsearch Service
   ↓
Elasticsearch Pods
```

Example DNS concept:

```text
elasticsearch.logging.svc.cluster.local
```

The exact service name depends on your deployment.

---

# 90. Elasticsearch Installation With GitOps

For your architecture:

```text
GitHub
   ↓
Elasticsearch configuration
   ↓
Pull Request
   ↓
GitHub Actions
   ↓
Validation
   ↓
ArgoCD
   ↓
EKS
   ↓
Elasticsearch
```

This gives version-controlled infrastructure.

---

# 91. Suggested Repository Structure

```text
08-ELK-Stack/
```

for learning documentation is separate from your deployment repository.

A deployment repository could contain:

```text
observability/
│
├── elasticsearch/
│   ├── base/
│   └── overlays/
│
├── logstash/
│   ├── base/
│   └── overlays/
│
├── kibana/
│   ├── base/
│   └── overlays/
│
└── argocd/
```

Keep environment-specific configuration separate.

---

# 92. Development Environment

Development:

```text
Single Elasticsearch Node
Single Kibana
Minimal Logstash
Minimal Storage
```

Architecture:

```text
App
 ↓
Collector
 ↓
Logstash
 ↓
ES
 ↓
Kibana
```

This is enough to learn the complete pipeline.

---

# 93. Staging Environment

Staging can be closer to production:

```text
Multiple Elasticsearch nodes
Multiple Logstash replicas
Persistent storage
Security
TLS
Monitoring
Backup testing
```

This is where production changes should be validated.

---

# 94. Production Environment

Production:

```text
Multiple AZs
Multiple Elasticsearch nodes
Persistent storage
Replica shards
Logstash HA
Kibana HA
TLS
RBAC
Snapshots
Monitoring
Alerting
Retention
Disaster recovery
```

---

# 95. Managed Elasticsearch

Instead of operating Elasticsearch yourself, an organization may use a managed service.

Conceptually:

```text
Applications
     ↓
Logstash
     ↓
Managed Elasticsearch
     ↓
Kibana / Visualization
```

Benefits can include:

```text
Reduced infrastructure management
Automated maintenance
Managed backups
Managed scaling options
```

Trade-offs include:

```text
Cost
Feature differences
Vendor dependency
Network design
Service-specific limitations
```

---

# 96. Self-Managed vs Managed

| Area           | Self-Managed                | Managed                   |
| -------------- | --------------------------- | ------------------------- |
| Infrastructure | You manage                  | Provider manages          |
| Upgrades       | You manage                  | Provider-assisted/managed |
| Storage        | You manage                  | Provider-managed          |
| Scaling        | You manage                  | Provider options          |
| Cost           | Infrastructure + operations | Service cost              |
| Control        | High                        | Depends on service        |
| Operations     | Higher                      | Lower                     |

Choose based on organizational requirements.

---

# 97. Installation Troubleshooting: Service Won't Start

Check:

```bash
sudo systemctl status elasticsearch
```

Then:

```bash
sudo journalctl -u elasticsearch -n 100
```

Then inspect:

```text
Configuration
Permissions
Memory
Disk
Port
Java/runtime
Security configuration
```

Do not repeatedly restart without reading the error.

---

# 98. Troubleshooting: Port Not Listening

Check:

```bash
ss -lntp | grep 9200
```

If nothing is listening:

```text
Elasticsearch may not be running
```

Check:

```bash
systemctl status elasticsearch
```

Then:

```bash
journalctl -u elasticsearch
```

---

# 99. Troubleshooting: Out of Memory

Check:

```bash
free -h
```

Then inspect Elasticsearch/JVM logs.

Potential causes:

```text
Insufficient memory
Large queries
Heavy aggregations
High indexing load
Poor heap configuration
```

Do not simply increase heap without understanding total memory requirements.

---

# 100. Troubleshooting: Disk Full

Check:

```bash
df -h
```

Then:

```bash
du -sh /var/lib/elasticsearch/*
```

Identify:

```text
Large indexes
Unexpected files
Rapid growth
Retention problems
```

Follow the approved retention and recovery process.

---

# 101. Troubleshooting: Node Cannot Join Cluster

Check:

```text
Cluster name
Node configuration
Discovery settings
Network connectivity
Transport port
Security/TLS
DNS
```

Test connectivity between nodes.

Example:

```bash
nc -zv <node-ip> 9300
```

Use network testing carefully and only where appropriate.

---

# 102. Troubleshooting: Cluster Yellow

First inspect:

```bash
curl http://localhost:9200/_cluster/health
```

Then:

```bash
curl "http://localhost:9200/_cat/shards?v"
```

Look for:

```text
UNASSIGNED
```

Then investigate why the replica cannot be allocated.

---

# 103. Troubleshooting: Cluster Red

Check:

```text
Primary shard availability
Node failures
Disk
Corruption
Allocation
Recent configuration changes
```

A red cluster requires immediate investigation because one or more primary shards are unavailable.

---

# 104. Troubleshooting: Logs Not Reaching Elasticsearch

Trace:

```text
Application
   ↓
Collector
   ↓
Logstash
   ↓
Elasticsearch
```

At Elasticsearch:

```text
Check connectivity
Check authentication
Check permissions
Check cluster health
Check index
Check indexing errors
```

---

# 105. Installation Best Practices

```text
1. Pin the Elasticsearch version.
2. Use official packages or supported deployment methods.
3. Separate environments.
4. Use persistent storage.
5. Plan capacity before production.
6. Keep Elasticsearch private.
7. Enable security.
8. Use TLS.
9. Use least privilege.
10. Monitor JVM and disk.
11. Test backups.
12. Test recovery.
13. Use GitOps for Kubernetes configuration.
14. Validate changes in staging.
15. Keep versions compatible across ELK.
```

---

# 106. Real-World Installation Flow

For a Linux/EC2 deployment:

```text
EC2 Provisioning
       ↓
OS Preparation
       ↓
Storage Configuration
       ↓
Repository Configuration
       ↓
Elasticsearch Installation
       ↓
elasticsearch.yml
       ↓
Security Configuration
       ↓
System Limits
       ↓
Service Start
       ↓
API Validation
       ↓
Cluster Validation
       ↓
Monitoring
```

---

# 107. Real-World EKS Installation Flow

For Kubernetes:

```text
EKS
 ↓
Namespace
 ↓
Storage Class
 ↓
Elasticsearch Operator / Supported Deployment
 ↓
Elasticsearch Cluster
 ↓
Persistent Volumes
 ↓
Security
 ↓
Network Policies
 ↓
Monitoring
 ↓
Snapshots
 ↓
Validation
```

---

# 108. Production Architecture After Installation

The final production architecture becomes:

```text
                         USERS
                           │
                           ↓
                        Route 53
                           │
                           ↓
                          ALB
                           │
                           ↓
                        Kibana
                           │
                           ↓
                Elasticsearch Cluster
              ┌────────────┼────────────┐
              ↓            ↓            ↓
             ES1          ES2          ES3
              │            │            │
             PV           PV           PV
              ↑            ↑            ↑
              └────────────┼────────────┘
                           ↑
                     Logstash Cluster
                    ┌──────┴──────┐
                    ↓             ↓
                  LS-A          LS-B
                    ↑             ↑
                    └──────┬──────┘
                           ↑
                    Log Collectors
                           ↑
                         EKS
                           ↑
                    Application Pods
```

---

# 109. Installation Verification

After installation, verify the complete chain:

```text
Elasticsearch
      ↓
Cluster Health
      ↓
Node Health
      ↓
Index Creation
      ↓
Document Insertion
      ↓
Document Search
      ↓
Logstash Integration
      ↓
Kibana Integration
```

Do not consider installation complete simply because the systemd service is running.

---

# 110. Final Elasticsearch Installation Checklist

```text
Infrastructure
[ ] CPU planned
[ ] Memory planned
[ ] Storage planned
[ ] Network planned

Installation
[ ] Correct version
[ ] Official repository/package
[ ] Service installed
[ ] Service enabled

Configuration
[ ] Cluster name
[ ] Node name
[ ] Network binding
[ ] Discovery
[ ] Data path
[ ] Log path

Security
[ ] Authentication
[ ] Authorization
[ ] TLS
[ ] Network restrictions
[ ] Secrets protected

Storage
[ ] Persistent disk
[ ] Correct ownership
[ ] Sufficient capacity
[ ] Backup strategy

Cluster
[ ] Nodes joined
[ ] Cluster health verified
[ ] Shards verified
[ ] Replicas verified

Validation
[ ] API works
[ ] Test index created
[ ] Test document inserted
[ ] Test search works
[ ] Test data removed

Operations
[ ] Monitoring
[ ] Alerting
[ ] Retention
[ ] Snapshots
[ ] Disaster recovery
```

---

# 111. Final Mental Model

The installation process should be remembered as:

```text
                 ELASTICSEARCH
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
    Infrastructure            Configuration
          │                       │
       CPU/RAM                 Cluster
       Storage                 Network
       Network                 Security
          │                       │
          └───────────┬───────────┘
                      ↓
                 Elasticsearch
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
        Node 1      Node 2      Node 3
          │           │           │
         Disk        Disk        Disk
          │           │           │
          └───────────┼───────────┘
                      ↓
                  Cluster
                      │
              ┌───────┴───────┐
              ↓               ↓
           Logstash         Kibana
```

The key principle is:

**Installing Elasticsearch is not just installing a package. In a real production environment, installation includes infrastructure preparation, persistent storage, cluster formation, networking, security, capacity planning, validation, monitoring, backup, and recovery planning.**
