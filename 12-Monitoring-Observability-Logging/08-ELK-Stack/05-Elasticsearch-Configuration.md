# Elasticsearch Configuration

## 1. Overview

After installing Elasticsearch, the next step is configuring it correctly for the intended environment.

Elasticsearch configuration determines:

```text
Cluster identity
Node identity
Network behavior
Cluster discovery
Storage locations
Security
TLS
Memory behavior
Shard allocation
Logging
```

The main configuration file for a package-based installation is commonly:

```text
/etc/elasticsearch/elasticsearch.yml
```

Other important configuration areas include:

```text
/etc/elasticsearch/jvm.options
/etc/elasticsearch/jvm.options.d/
```

The exact paths can vary depending on how Elasticsearch was installed.

---

# 2. Configuration Architecture

A simplified configuration model is:

```text
                    Elasticsearch
                          │
          ┌───────────────┼───────────────┐
          ↓               ↓               ↓
       Cluster          Network         Storage
       Config           Config           Config
          │               │               │
          └───────────────┼───────────────┘
                          ↓
                       Security
                          ↓
                       Runtime
```

For a production cluster:

```text
ES-01
 └── elasticsearch.yml

ES-02
 └── elasticsearch.yml

ES-03
 └── elasticsearch.yml
```

The configuration must be consistent where required while allowing node-specific settings such as `node.name`.

---

# 3. Configuration File

The primary configuration file is:

```text
/etc/elasticsearch/elasticsearch.yml
```

Typical configuration categories include:

```yaml
cluster.name:
node.name:

path.data:
path.logs:

network.host:
http.port:

discovery.seed_hosts:
cluster.initial_master_nodes:

xpack.security.enabled:
```

The exact settings required depend on whether the deployment is:

```text
Development
Staging
Production
```

---

# 4. Configuration Principles

Use these principles:

```text
1. Change only what you need.
2. Keep configuration version controlled.
3. Validate changes before production.
4. Avoid exposing Elasticsearch publicly.
5. Separate development and production settings.
6. Remove bootstrap-only settings after cluster formation.
7. Keep security enabled in production.
8. Monitor the impact of configuration changes.
```

---

# 5. Cluster Name

The cluster name identifies the Elasticsearch cluster.

Example:

```yaml
cluster.name: production-logging
```

Staging:

```yaml
cluster.name: staging-logging
```

Development:

```yaml
cluster.name: dev-logging
```

Use meaningful names.

---

# 6. Why Cluster Name Matters

Consider:

```text
Production:
production-logging

Staging:
staging-logging
```

This makes it easier to identify the environment during:

```text
Monitoring
Troubleshooting
API requests
Incident response
```

---

# 7. Node Name

Each node should have a meaningful unique name.

Example:

```yaml
node.name: es-prod-01
```

Another node:

```yaml
node.name: es-prod-02
```

Another:

```yaml
node.name: es-prod-03
```

Node names are especially useful when troubleshooting cluster problems.

---

# 8. Cluster and Node Relationship

Conceptually:

```text
cluster.name:
production-logging
        │
        ├── es-prod-01
        ├── es-prod-02
        └── es-prod-03
```

All nodes belong to the same intended cluster.

---

# 9. Data Path

Elasticsearch needs a location for persistent data.

Example:

```yaml
path.data: /var/lib/elasticsearch
```

This contains Elasticsearch data such as:

```text
Indices
Shards
Segments
Cluster data
```

For production, this path should reside on appropriate persistent storage.

---

# 10. Log Path

Elasticsearch's own logs can be stored under:

```yaml
path.logs: /var/log/elasticsearch
```

These logs help troubleshoot:

```text
Startup failures
Cluster formation
Shard allocation
Security issues
JVM problems
Indexing problems
```

---

# 11. Storage Architecture

Production:

```text
Elasticsearch
      ↓
path.data
      ↓
Persistent Storage
```

AWS EC2 example:

```text
EC2
 │
 ├── Root EBS
 │
 └── Data EBS
       ↓
/var/lib/elasticsearch
```

Do not place production Elasticsearch data on temporary or ephemeral storage.

---

# 12. Network Configuration

By default, Elasticsearch may be accessible only through local interfaces depending on the installation and configuration.

For a networked deployment, configure an appropriate address.

Example:

```yaml
network.host: 10.0.10.10
```

Use the node's private IP or an appropriate network interface.

---

# 13. Avoid Public Binding

Avoid using:

```yaml
network.host: 0.0.0.0
```

as a substitute for proper network design.

If you do bind to all interfaces, the surrounding network controls must prevent unauthorized access.

Production architecture should be:

```text
Private Network
      ↓
Elasticsearch
```

not:

```text
Internet
   ↓
Elasticsearch
```

---

# 14. HTTP Port

Elasticsearch's HTTP API commonly uses:

```yaml
http.port: 9200
```

Example:

```text
Kibana
   ↓
HTTPS / HTTP
   ↓
9200
   ↓
Elasticsearch
```

Port `9200` is normally used for client/API communication.

---

# 15. Transport Communication

Elasticsearch nodes communicate using the transport layer.

The commonly used transport port is:

```text
9300
```

Conceptually:

```text
ES-01
  ↕
9300
  ↕
ES-02
  ↕
9300
  ↕
ES-03
```

This traffic must be allowed between Elasticsearch nodes.

---

# 16. HTTP vs Transport

Remember:

```text
HTTP
 ↓
Client/API communication
 ↓
9200
```

and:

```text
Transport
 ↓
Node-to-node communication
 ↓
9300
```

Examples:

```text
Kibana → Elasticsearch
        ↓
       9200

ES-01 → ES-02
        ↓
       9300
```

---

# 17. Discovery

In a multi-node cluster, Elasticsearch needs to discover the nodes that can participate in cluster formation.

A common setting is:

```yaml
discovery.seed_hosts:
  - es-prod-01
  - es-prod-02
  - es-prod-03
```

The seed hosts provide initial discovery information.

---

# 18. Discovery Seed Hosts

Example:

```yaml
discovery.seed_hosts:
  - 10.0.10.11:9300
  - 10.0.20.11:9300
  - 10.0.30.11:9300
```

This tells a node where it can find other nodes during discovery.

The exact addresses should be private, reachable addresses for the Elasticsearch cluster.

---

# 19. DNS-Based Discovery

Instead of hard-coding IP addresses, DNS names can be used where appropriate.

Example:

```yaml
discovery.seed_hosts:
  - es-prod-01.internal
  - es-prod-02.internal
  - es-prod-03.internal
```

This can make infrastructure changes easier when DNS is managed properly.

---

# 20. Cluster Bootstrapping

A brand-new cluster needs to establish its initial cluster state.

The setting:

```yaml
cluster.initial_master_nodes:
```

is used for initial bootstrapping of a new cluster.

Example:

```yaml
cluster.initial_master_nodes:
  - es-prod-01
  - es-prod-02
  - es-prod-03
```

The node names must correspond to the intended initial master-eligible nodes.

---

# 21. Important: Initial Master Nodes

`cluster.initial_master_nodes` is a bootstrap setting.

It should **not** be treated as a permanent cluster discovery setting.

After the cluster has successfully formed, remove it from the configuration.

Leaving it configured can create serious cluster-formation risks after future misconfiguration or restarts.

---

# 22. Discovery vs Bootstrapping

These settings have different purposes:

```text
discovery.seed_hosts
        ↓
Find nodes

cluster.initial_master_nodes
        ↓
Bootstrap a brand-new cluster
```

Think:

```text
Discovery
"Where can I find other nodes?"

Bootstrap
"Which nodes form the initial cluster?"
```

---

# 23. Production Cluster Configuration

Example conceptual configuration:

```yaml
cluster.name: production-logging
node.name: es-prod-01

path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch

network.host: 10.0.10.11
http.port: 9200

discovery.seed_hosts:
  - 10.0.10.11:9300
  - 10.0.20.11:9300
  - 10.0.30.11:9300

cluster.initial_master_nodes:
  - es-prod-01
  - es-prod-02
  - es-prod-03
```

This is a conceptual example.

Security settings should also be configured for production.

---

# 24. After Cluster Formation

Once the cluster has formed successfully:

```text
Before:
cluster.initial_master_nodes:
  - es-prod-01
  - es-prod-02
  - es-prod-03
```

Then remove the setting.

The continuing configuration should retain discovery:

```yaml
discovery.seed_hosts:
  - es-prod-01
  - es-prod-02
  - es-prod-03
```

This distinction is important.

---

# 25. Single-Node Configuration

For development:

```yaml
cluster.name: dev-logging
node.name: es-dev-01

discovery.type: single-node
```

This is useful when you intentionally want one node.

Do not use this configuration for a production multi-node cluster.

---

# 26. Production vs Development

| Setting   | Development                          | Production                     |
| --------- | ------------------------------------ | ------------------------------ |
| Cluster   | Single node                          | Multi-node                     |
| Discovery | Single-node or appropriate discovery | Seed hosts                     |
| Bootstrap | Simple                               | Initial cluster bootstrap      |
| Storage   | Local/persistent                     | Dedicated persistent storage   |
| Security  | Recommended                          | Required for production design |
| TLS       | Recommended                          | Required                       |
| HA        | Usually no                           | Yes                            |
| Backups   | Recommended                          | Required                       |

---

# 27. Node Roles

Elasticsearch nodes can have different roles.

Depending on version and architecture, roles can include:

```text
Master-eligible
Data
Ingest
Coordinating
```

A small cluster may combine roles.

A large cluster may separate them.

---

# 28. Master-Eligible Nodes

Master-eligible nodes participate in cluster management.

Conceptually:

```text
Master-eligible nodes
        ↓
Cluster coordination
```

For a production cluster, you should have enough master-eligible nodes to maintain quorum and tolerate expected failures.

---

# 29. Dedicated Master Nodes

For larger environments:

```text
Master Nodes
 ├── master-01
 ├── master-02
 └── master-03
```

Then:

```text
Data Nodes
 ├── data-01
 ├── data-02
 └── data-03
```

This separates cluster-management workloads from data workloads.

---

# 30. Data Nodes

Data nodes handle workloads such as:

```text
Indexing
Searching
Aggregations
Shard storage
```

They usually require substantial:

```text
CPU
Memory
Disk
```

depending on the workload.

---

# 31. Ingest Nodes

Ingest nodes can process documents before indexing.

Conceptually:

```text
Logstash
    ↓
Elasticsearch
    ↓
Ingest Pipeline
    ↓
Data Node
```

Use ingest pipelines when Elasticsearch-side preprocessing is appropriate.

Do not duplicate expensive processing in both Logstash and Elasticsearch without a reason.

---

# 32. Coordinating Nodes

A coordinating node can receive client requests and distribute work.

Example:

```text
Kibana
  ↓
Coordinating Node
  ↓
┌───┼───┐
↓   ↓   ↓
D1  D2  D3
```

It then combines the responses.

---

# 33. Small Production Cluster

A smaller environment may use:

```text
ES-01
ES-02
ES-03
```

with multiple roles on each node.

This keeps the architecture simpler.

---

# 34. Large Production Cluster

A larger architecture may be:

```text
              Clients
                 │
                 ↓
         Coordinating Nodes
                 │
                 ↓
             Data Nodes
                 │
        ┌────────┼────────┐
        ↓        ↓        ↓
       D1       D2       D3

Master Nodes
    │
    ├── M1
    ├── M2
    └── M3
```

The exact design should be driven by workload.

---

# 35. Security Configuration

Production Elasticsearch should have security enabled.

Conceptually:

```yaml
xpack.security.enabled: true
```

Modern Elasticsearch installations can automatically configure security during installation depending on the installation method and environment.

---

# 36. Authentication

Authentication controls who can connect.

Architecture:

```text
User
 ↓
Kibana
 ↓
Authentication
 ↓
Elasticsearch
```

Possible enterprise mechanisms can include:

```text
Native users
SSO
OIDC
SAML
LDAP
API keys
```

The supported options depend on the Elasticsearch deployment and subscription/features.

---

# 37. Authorization

Authorization controls what authenticated users can do.

Example:

```text
Platform Engineer
   ↓
Infrastructure logs

Security Engineer
   ↓
Security logs

Application Team
   ↓
Application logs
```

Use role-based access control and least privilege.

---

# 38. HTTP TLS

Client/API communication should be encrypted.

Conceptually:

```yaml
xpack.security.http.ssl:
  enabled: true
```

Then:

```text
Kibana
   ↓
HTTPS
   ↓
Elasticsearch
```

This protects HTTP API traffic.

---

# 39. Transport TLS

Node-to-node communication should also be secured.

Conceptually:

```yaml
xpack.security.transport.ssl:
  enabled: true
```

Then:

```text
ES-01
   ↓ TLS
ES-02
   ↓ TLS
ES-03
```

Elastic's security configuration examples show HTTP TLS for client connections and transport TLS for encrypted and mutually authenticated node communication.

---

# 40. Transport TLS vs HTTP TLS

Remember:

```text
HTTP TLS
   ↓
Client ↔ Elasticsearch
```

and:

```text
Transport TLS
   ↓
Elasticsearch node ↔ Elasticsearch node
```

Both protect different communication paths.

---

# 41. Certificate Configuration

TLS requires certificates and keys.

Conceptually:

```text
Certificate Authority
        ↓
Certificates
        ↓
Elasticsearch Nodes
```

Example configuration structure:

```yaml
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
```

The exact certificate configuration depends on the deployment.

---

# 42. Verification Mode

Transport TLS can use certificate verification modes.

A production environment should use an appropriate verification mode that validates the intended identities and trust chain.

Do not weaken certificate verification simply to make cluster formation work.

---

# 43. HTTP TLS Example

Conceptually:

```yaml
xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12
```

This enables encrypted HTTP communication.

The certificate must be trusted by clients such as:

```text
Kibana
Logstash
Applications
Administrators
```

---

# 44. Transport TLS Example

Conceptually:

```yaml
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
```

This protects node-to-node communication.

---

# 45. Do Not Copy Certificates

Never copy a node's private TLS identity blindly to every Elasticsearch node.

Instead:

```text
CA
 │
 ├── Certificate for ES-01
 ├── Certificate for ES-02
 └── Certificate for ES-03
```

Each node should have an appropriate identity.

---

# 46. Security Configuration Architecture

```text
                    Elasticsearch
                          │
            ┌─────────────┴─────────────┐
            ↓                           ↓
       HTTP TLS                    Transport TLS
            ↓                           ↓
     Client Security             Node Security
            │                           │
       Authentication             Cluster Trust
            │                           │
       Authorization
```

---

# 47. API Keys

For integrations such as Logstash, API keys can provide controlled access.

Conceptually:

```text
Logstash
   ↓
API Key
   ↓
Elasticsearch
```

Give the API key only the permissions required for the intended indexes and operations.

---

# 48. Logstash Permissions

Logstash should not use the Elasticsearch superuser for normal ingestion.

Instead:

```text
Logstash
   ↓
Dedicated credential
   ↓
Minimum required privileges
```

For example:

```text
Create/write application log indexes
```

rather than:

```text
Full cluster administration
```

---

# 49. Kibana Permissions

Kibana requires appropriate privileges to interact with Elasticsearch.

Users should receive roles appropriate to their responsibilities.

Example:

```text
Read-only user
   ↓
Search logs

Developer
   ↓
Search application logs

Administrator
   ↓
Manage logging platform
```

---

# 50. Data Paths

Production storage configuration should be explicit.

Example:

```yaml
path.data:
  - /data/elasticsearch
```

Multiple data paths may be supported in some deployment scenarios, but storage architecture should be designed carefully rather than simply adding disks.

---

# 51. Log Paths

Example:

```yaml
path.logs: /var/log/elasticsearch
```

If logs are being collected centrally:

```text
Elasticsearch
     ↓
/var/log/elasticsearch
     ↓
Log Collector
     ↓
ELK
```

Be careful to avoid creating a recursive logging pipeline where Elasticsearch's own logs endlessly feed back into itself.

---

# 52. Temporary and Data Storage

Keep these concepts separate:

```text
OS
 ↓
Root filesystem

Elasticsearch
 ↓
Persistent data filesystem
```

The Elasticsearch data path is the critical stateful storage location.

---

# 53. Memory Locking

Memory swapping can severely impact Elasticsearch performance.

Some environments configure memory locking:

```yaml
bootstrap.memory_lock: true
```

If you enable it, the operating system/service manager must also allow the process to lock the required memory.

Do not enable it without configuring the underlying limits correctly.

---

# 54. Swap

Elasticsearch performs poorly under memory pressure and swapping.

Production systems should be designed to avoid Elasticsearch swapping.

Check:

```bash
free -h
```

and:

```bash
swapon --show
```

Follow the Elasticsearch version's official bootstrap and memory guidance.

---

# 55. System Limits

Review:

```text
Open files
Virtual memory
Processes
Memory locking
```

For example:

```bash
ulimit -n
```

But remember that service-level limits can differ from your interactive shell.

---

# 56. `vm.max_map_count`

Elasticsearch can require an appropriate Linux virtual memory map limit.

Check:

```bash
sysctl vm.max_map_count
```

If your version requires a higher value, configure it through the operating system's persistent sysctl configuration.

Always use the value specified by the Elasticsearch version you are deploying.

---

# 57. Destructive Operations

Elasticsearch provides a setting that can require explicit index names for destructive operations.

Conceptually:

```yaml
action.destructive_requires_name: true
```

This helps prevent accidental broad deletion.

For example, an operator should be cautious with:

```text
DELETE /*
```

Production systems should use safeguards against accidental destructive operations.

---

# 58. Index Lifecycle Configuration

Configuration may also include lifecycle-related components.

The architecture is:

```text
New Logs
   ↓
Hot
   ↓
Warm
   ↓
Cold
   ↓
Delete
```

The exact lifecycle implementation depends on Elasticsearch version and deployment features.

---

# 59. Index Templates

Templates can ensure new indexes receive consistent:

```text
Mappings
Settings
Aliases
```

Conceptually:

```text
Template
   ↓
New Index
   ↓
Consistent Mapping
```

This is particularly useful for log indexes.

---

# 60. Example Log Mapping

A useful logging structure could be:

```text
@timestamp      → date
service         → keyword
environment     → keyword
level           → keyword
namespace       → keyword
pod              → keyword
status_code     → integer
response_time   → long
message         → text
trace.id        → keyword
```

Stable mappings make searching and aggregation predictable.

---

# 61. Avoid Mapping Explosion

Bad application behavior:

```json
{
  "user_field_123": "...",
  "user_field_456": "...",
  "user_field_789": "..."
}
```

If applications continuously create new field names, Elasticsearch can accumulate huge numbers of mapped fields.

Prefer:

```json
{
  "user_field": "value"
}
```

with controlled structure.

---

# 62. Configuration for Kubernetes

When Elasticsearch runs on Kubernetes, configuration may be supplied through:

```text
Helm values
Custom Resource
ConfigMap
Environment variables
Secret
Operator configuration
```

The exact mechanism depends on the deployment method.

---

# 63. GitOps Configuration

For your environment, keep configuration in Git:

```text
observability/
└── elasticsearch/
    ├── base/
    ├── dev/
    ├── staging/
    └── prod/
```

Example:

```text
prod/
├── cluster.yaml
├── storage.yaml
├── security.yaml
└── monitoring.yaml
```

ArgoCD can then reconcile the desired state into EKS.

---

# 64. Configuration Flow

Your DevSecOps flow:

```text
Developer
   ↓
Git
   ↓
Pull Request
   ↓
Review
   ↓
GitHub Actions
   ↓
Validation
   ↓
Security Scan
   ↓
Merge
   ↓
ArgoCD
   ↓
EKS
   ↓
Elasticsearch
```

This avoids manual configuration drift.

---

# 65. Configuration Validation

Before deploying a configuration change:

```text
Syntax
 ↓
Security
 ↓
Connectivity
 ↓
Cluster behavior
 ↓
Performance
```

Test in:

```text
Development
   ↓
Staging
   ↓
Production
```

---

# 66. Configuration Changes

A safe process:

```text
1. Make one logical change.
2. Commit to Git.
3. Review.
4. Validate.
5. Deploy to staging.
6. Check cluster health.
7. Check logs.
8. Monitor.
9. Promote to production.
```

Avoid manually editing production configuration without recording the change.

---

# 67. Restart vs Dynamic Configuration

Some Elasticsearch settings can be changed dynamically.

Others require a restart.

Therefore:

```text
Before changing:
Check whether the setting is dynamic.
```

Do not automatically restart the entire cluster for every configuration change.

---

# 68. Rolling Configuration Changes

For a production cluster:

```text
ES-01
 ↓
Change
 ↓
Validate

ES-02
 ↓
Change
 ↓
Validate

ES-03
 ↓
Change
 ↓
Validate
```

Maintain cluster availability according to the change procedure.

Never restart every Elasticsearch node simultaneously unless the procedure explicitly requires it.

---

# 69. Cluster Health During Changes

Before a maintenance operation:

```text
Check:
Cluster health
Shard allocation
Node availability
Disk
```

Then make the change.

After the change:

```text
Check:
Cluster health
Shard allocation
Indexing
Search
```

---

# 70. Production Configuration Example

Conceptual example:

```yaml
cluster.name: production-logging
node.name: es-prod-01

path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch

network.host: 10.0.10.11
http.port: 9200

discovery.seed_hosts:
  - es-prod-01:9300
  - es-prod-02:9300
  - es-prod-03:9300

xpack.security.enabled: true

xpack.security.http.ssl:
  enabled: true
  keystore.path: certs/http.p12

xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
```

For the initial cluster bootstrap, `cluster.initial_master_nodes` is temporarily configured and then removed after the cluster forms.

---

# 71. Node-Specific Configuration

Node 1:

```yaml
node.name: es-prod-01
network.host: 10.0.10.11
```

Node 2:

```yaml
node.name: es-prod-02
network.host: 10.0.20.11
```

Node 3:

```yaml
node.name: es-prod-03
network.host: 10.0.30.11
```

Common configuration:

```yaml
cluster.name: production-logging

discovery.seed_hosts:
  - es-prod-01:9300
  - es-prod-02:9300
  - es-prod-03:9300
```

---

# 72. Configuration Management

Do not maintain production nodes by manually editing each server.

Bad:

```text
SSH → ES-01 → edit
SSH → ES-02 → edit
SSH → ES-03 → edit
```

Better:

```text
Git
 ↓
Configuration
 ↓
Automation
 ↓
All nodes
```

Possible tools include:

```text
Ansible
Terraform
Helm
Kubernetes Operators
ArgoCD
```

depending on the deployment.

---

# 73. Configuration as Code

For your DevOps workflow:

```text
GitHub
   ↓
Elasticsearch Configuration
   ↓
Pull Request
   ↓
Review
   ↓
GitHub Actions
   ↓
ArgoCD
```

Benefits:

```text
Version control
Auditability
Repeatability
Rollback
Peer review
Reduced drift
```

---

# 74. Elasticsearch Configuration and Terraform

Terraform can provision:

```text
EC2
Security Groups
EBS
Networking
IAM
DNS
```

But Elasticsearch application configuration should generally be managed through a configuration-management or deployment mechanism appropriate to the environment.

Example:

```text
Terraform
   ↓
Infrastructure

Ansible / Helm / Operator / ArgoCD
   ↓
Elasticsearch Configuration
```

Separate infrastructure from application configuration where practical.

---

# 75. Elasticsearch Configuration and Ansible

Ansible can manage VM-based Elasticsearch configuration.

Conceptually:

```text
Ansible
   ↓
elasticsearch.yml
   ↓
ES-01
ES-02
ES-03
```

This is useful for self-managed EC2/VM deployments.

---

# 76. Elasticsearch Configuration and Kubernetes

For EKS:

```text
Git
 ↓
Helm / Operator Configuration
 ↓
ArgoCD
 ↓
EKS
 ↓
Elasticsearch
```

This is more aligned with your GitOps architecture.

---

# 77. Secrets Management

Do not put:

```text
Passwords
Private keys
API keys
TLS private keys
```

directly into Git.

Use:

```text
AWS Secrets Manager
Kubernetes Secrets
External Secrets
Elasticsearch secure keystore
```

depending on the credential and deployment.

---

# 78. Elasticsearch Secure Settings

Some settings contain sensitive values.

Elasticsearch provides a secure keystore mechanism for sensitive configuration values.

Conceptually:

```text
Secret
  ↓
Elasticsearch Keystore
  ↓
Elasticsearch
```

This prevents sensitive settings from being stored as ordinary plaintext configuration where supported.

---

# 79. Environment Separation

Do not reuse production configuration blindly in development.

Example:

```text
dev:
dev-logging

staging:
staging-logging

prod:
production-logging
```

Separate:

```text
Credentials
Certificates
Endpoints
Storage
Retention
Cluster names
Resource sizing
```

---

# 80. Development Configuration

Example:

```yaml
cluster.name: dev-logging
node.name: es-dev-01

discovery.type: single-node

network.host: 127.0.0.1
http.port: 9200
```

This keeps the environment simple.

---

# 81. Staging Configuration

Example concept:

```yaml
cluster.name: staging-logging
node.name: es-staging-01

network.host: 10.20.10.10
http.port: 9200

discovery.seed_hosts:
  - es-staging-01:9300
  - es-staging-02:9300
  - es-staging-03:9300
```

Use production-like security and storage where possible.

---

# 82. Production Configuration

Example:

```text
Cluster
 ↓
3+ appropriate nodes

Network
 ↓
Private subnets

Storage
 ↓
Persistent dedicated storage

Security
 ↓
TLS + authentication + RBAC

Discovery
 ↓
Private DNS / seed hosts

Monitoring
 ↓
Prometheus + Grafana
```

---

# 83. Network Security Architecture

```text
                     Internet
                        X
                        │
                        │
                     Elasticsearch
                        │
                  PRIVATE NETWORK
                        │
             ┌──────────┼──────────┐
             ↓          ↓          ↓
          Logstash    Kibana     Admin
```

Only explicitly required traffic should be allowed.

---

# 84. AWS Security Group Design

Conceptually:

```text
Kibana SG
   ↓ 9200
Elasticsearch SG

Logstash SG
   ↓ 9200
Elasticsearch SG

Elasticsearch SG
   ↔ 9300
Elasticsearch SG
```

Do not allow arbitrary sources.

---

# 85. Kubernetes Network Policy

On EKS, network policies can further restrict traffic.

Conceptually:

```text
Logstash
   ↓
Elasticsearch

Kibana
   ↓
Elasticsearch
```

Other workloads should not automatically have unrestricted access.

---

# 86. Configuration Monitoring

Configuration itself should be monitored.

Track:

```text
Cluster configuration
Node configuration
Security configuration
Index templates
Lifecycle policies
```

Git history provides an audit trail when configuration is managed as code.

---

# 87. Configuration Drift

Manual production change:

```text
Git:
network.host = A

Server:
network.host = B
```

This is configuration drift.

GitOps aims to keep:

```text
Desired State
     =
Actual State
```

---

# 88. ArgoCD and Elasticsearch

For EKS:

```text
Git
 ↓
Desired Elasticsearch configuration
 ↓
ArgoCD
 ↓
EKS
 ↓
Elasticsearch
```

If someone manually changes a Kubernetes-managed configuration:

```text
Manual change
      ↓
Drift
      ↓
ArgoCD detects difference
      ↓
Reconciliation
```

This fits your existing GitOps approach.

---

# 89. Configuration Rollback

Suppose a configuration change causes:

```text
Cluster instability
```

Git provides the previous known-good version:

```text
Current
   ↓
Bad configuration

Git
   ↓
Previous version
```

Rollback:

```text
Git revert
   ↓
CI validation
   ↓
ArgoCD
   ↓
Known-good configuration
```

---

# 90. Configuration Testing

Test:

```text
Syntax
Security
Connectivity
Cluster formation
Indexing
Search
Performance
```

Example:

```text
Configuration
 ↓
Staging
 ↓
Create test index
 ↓
Insert documents
 ↓
Search
 ↓
Check cluster
```

---

# 91. Configuration Troubleshooting

If Elasticsearch fails after a configuration change:

```text
1. Check systemd status
2. Read Elasticsearch logs
3. Identify changed setting
4. Validate YAML syntax
5. Check network
6. Check certificates
7. Check discovery
8. Check permissions
9. Revert known-bad change
10. Restart only when appropriate
```

---

# 92. Common Configuration Error: Wrong YAML

Example:

```yaml
cluster.name production
```

Incorrect.

Correct:

```yaml
cluster.name: production
```

YAML indentation and syntax matter.

---

# 93. Common Error: Wrong Node Name

If:

```yaml
cluster.initial_master_nodes:
  - es-prod-01
```

but the node is actually:

```yaml
node.name: es-prod-001
```

cluster bootstrapping can fail.

Always ensure the names match the intended initial master-eligible nodes.

---

# 94. Common Error: Wrong Seed Hosts

Example:

```yaml
discovery.seed_hosts:
  - 10.0.50.10
```

but the node cannot reach it.

Check:

```bash
nc -zv 10.0.50.10 9300
```

Also verify:

```text
Security groups
Network ACLs
Routing
DNS
Transport TLS
```

---

# 95. Common Error: TLS Mismatch

Example:

```text
ES-01:
Transport TLS enabled

ES-02:
Transport TLS disabled
```

This can prevent nodes from communicating correctly.

Ensure cluster nodes use compatible security configuration.

---

# 96. Common Error: Certificate Problem

Symptoms may include:

```text
Certificate validation failure
Unknown CA
Hostname mismatch
Expired certificate
```

Check:

```text
Certificate
Private key
CA
Truststore
Hostname
Expiration
Permissions
```

Do not disable TLS validation as a shortcut.

---

# 97. Common Error: Port Blocked

If:

```text
ES-01 → ES-02:9300
```

is blocked:

```text
Cluster formation
       ↓
May fail
```

Check:

```bash
nc -zv <es-02-private-ip> 9300
```

Then inspect network controls.

---

# 98. Common Error: Disk Full

Symptoms:

```text
Indexing failures
Shard allocation problems
Cluster health degradation
```

Check:

```bash
df -h
```

Then inspect:

```text
Index growth
Retention
Shard allocation
Disk watermarks
```

---

# 99. Common Error: Memory Pressure

Symptoms:

```text
Slow searches
GC pressure
Node instability
OOM
```

Check:

```bash
free -h
```

and Elasticsearch/JVM metrics.

Then review:

```text
Heap
Queries
Aggregations
Indexing load
Container limits
```

---

# 100. Production Configuration Checklist

```text
Cluster
[ ] Correct cluster name
[ ] Unique node names
[ ] Correct node roles
[ ] Discovery configured
[ ] Bootstrap completed
[ ] cluster.initial_master_nodes removed

Network
[ ] Private IP/interface
[ ] HTTP port
[ ] Transport connectivity
[ ] Security groups
[ ] Network policies

Storage
[ ] Persistent data path
[ ] Correct permissions
[ ] Capacity planned
[ ] Backup configured

Security
[ ] Authentication
[ ] Authorization
[ ] HTTP TLS
[ ] Transport TLS
[ ] Certificates
[ ] Secrets protected

Runtime
[ ] Memory planned
[ ] Swap controlled
[ ] System limits
[ ] vm.max_map_count
[ ] JVM configured appropriately

Operations
[ ] Monitoring
[ ] Alerting
[ ] Logging
[ ] Retention
[ ] Snapshots
[ ] Disaster recovery
```

---

# 101. Real-World Configuration Workflow

For your DevOps environment:

```text
Infrastructure
     ↓
Terraform
     ↓
EC2 / EKS
     ↓
Elasticsearch Deployment
     ↓
Configuration
     ↓
Security
     ↓
Storage
     ↓
Monitoring
     ↓
Logstash
     ↓
Kibana
```

For EKS with GitOps:

```text
GitHub
   ↓
Configuration
   ↓
GitHub Actions
   ↓
Validation + Security
   ↓
ArgoCD
   ↓
EKS
   ↓
Elasticsearch
```

---

# 102. Configuration Example: Three-Node Cluster

Node 1:

```yaml
cluster.name: production-logging
node.name: es-prod-01

path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch

network.host: 10.0.10.11
http.port: 9200

discovery.seed_hosts:
  - es-prod-01:9300
  - es-prod-02:9300
  - es-prod-03:9300

cluster.initial_master_nodes:
  - es-prod-01
  - es-prod-02
  - es-prod-03
```

Node 2:

```yaml
cluster.name: production-logging
node.name: es-prod-02

path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch

network.host: 10.0.20.11
http.port: 9200

discovery.seed_hosts:
  - es-prod-01:9300
  - es-prod-02:9300
  - es-prod-03:9300

cluster.initial_master_nodes:
  - es-prod-01
  - es-prod-02
  - es-prod-03
```

Node 3:

```yaml
cluster.name: production-logging
node.name: es-prod-03

path.data: /data/elasticsearch
path.logs: /var/log/elasticsearch

network.host: 10.0.30.11
http.port: 9200

discovery.seed_hosts:
  - es-prod-01:9300
  - es-prod-02:9300
  - es-prod-03:9300

cluster.initial_master_nodes:
  - es-prod-01
  - es-prod-02
  - es-prod-03
```

After successful initial cluster formation, remove:

```yaml
cluster.initial_master_nodes:
  - es-prod-01
  - es-prod-02
  - es-prod-03
```

Keep the discovery configuration for normal node discovery.

---

# 103. Final Production Configuration Architecture

```text
                     Git
                      │
                      ↓
            Elasticsearch Config
                      │
                Pull Request
                      │
                      ↓
                GitHub Actions
                      │
              Validation/Security
                      │
                      ↓
                   ArgoCD
                      │
                      ↓
                     EKS
                      │
             ┌────────┼────────┐
             ↓        ↓        ↓
           ES-01    ES-02    ES-03
             │        │        │
             └────────┼────────┘
                      ↓
                  Elasticsearch
                      │
            ┌─────────┴─────────┐
            ↓                   ↓
         Logstash             Kibana
            │                   │
            ↓                   ↓
        Application          Engineers
            Logs
```

---

# 104. Final Mental Model

Remember Elasticsearch configuration in these layers:

```text
                    ELASTICSEARCH
                          │
        ┌─────────────────┼─────────────────┐
        ↓                 ↓                 ↓
      Cluster           Network           Storage
        │                 │                 │
    Name/Node          HTTP/Transport     Data/Logs
    Discovery          Ports/TLS          Persistent Disk
        │                 │                 │
        └─────────────────┼─────────────────┘
                          ↓
                       Security
                          │
              ┌───────────┴───────────┐
              ↓                       ↓
         Authentication          Authorization
              │                       │
              └───────────┬───────────┘
                          ↓
                       Runtime
                          │
              ┌───────────┼───────────┐
              ↓           ↓           ↓
            Memory       JVM        OS Limits
```

The most important production configuration concepts are:

```text
cluster.name
node.name
network.host
http.port
discovery.seed_hosts
cluster.initial_master_nodes
path.data
path.logs
xpack.security.enabled
HTTP TLS
Transport TLS
```

And the most important operational rule is:

**Treat Elasticsearch configuration as production infrastructure code. Version it, review it, validate it in staging, deploy it through controlled automation, monitor the result, and never leave one-time cluster-bootstrap settings in place after the cluster has formed.**
