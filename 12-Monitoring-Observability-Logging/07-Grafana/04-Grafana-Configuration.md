# Grafana Configuration

## 1. Overview

Installing Grafana is only the first step.

A production Grafana deployment requires configuration for:

```text
Server
Authentication
Authorization
Database
Data Sources
Dashboards
Alerting
Security
Plugins
Logging
Performance
High Availability
Provisioning
```

A useful configuration model is:

```text
                    Grafana
                       │
       ┌───────────────┼────────────────┐
       ↓               ↓                ↓
    Server          Security         Database
       │               │                │
       ├── HTTP        ├── Auth         └── PostgreSQL
       ├── HTTPS       ├── RBAC
       └── Root URL    └── TLS
                       │
       ┌───────────────┼────────────────┐
       ↓               ↓                ↓
  Data Sources     Dashboards        Alerting
       │               │                │
       ↓               ↓                ↓
 Prometheus         Git/Files       Alertmanager
 Elasticsearch
 Jaeger
```

---

# 2. Grafana Configuration Methods

Grafana configuration can be managed using:

```text
1. grafana.ini
2. Environment variables
3. Helm values
4. Provisioning files
5. Kubernetes Secrets
6. ConfigMaps
```

In a production Kubernetes environment, configuration is usually managed through:

```text
Git
 ↓
Helm values
 ↓
ConfigMaps / Secrets
 ↓
Grafana
```

---

# 3. Configuration Hierarchy

A practical configuration flow is:

```text
Git Repository
      ↓
Helm Values
      ↓
Kubernetes Resources
      ↓
Grafana Configuration
```

Sensitive values should follow:

```text
Secret Manager
      ↓
Kubernetes Secret
      ↓
Grafana
```

Do not store passwords or private credentials in plain Git.

---

# 4. Grafana Configuration File

Traditional Grafana installations commonly use:

```text
/etc/grafana/grafana.ini
```

Important sections include:

```text
[server]
[database]
[security]
[users]
[auth]
[auth.generic_oauth]
[log]
[analytics]
[alerting]
[plugins]
```

The exact available settings depend on the Grafana version.

---

# 5. Server Configuration

The `[server]` section controls server-level behavior.

Common settings include:

```text
Protocol
HTTP address
HTTP port
Domain
Root URL
Serve from sub-path
```

Example:

```ini
[server]
protocol = http
http_port = 3000
domain = grafana.example.internal
root_url = https://grafana.example.internal
```

---

# 6. HTTP Port

Grafana commonly listens on:

```text
3000
```

Example:

```ini
[server]
http_port = 3000
```

Inside Kubernetes, the Service can expose another port externally:

```text
Service port 80
       ↓
Container port 3000
```

Architecture:

```text
ALB
 ↓
Service :80
 ↓
Grafana :3000
```

---

# 7. Root URL

The root URL defines how users access Grafana.

Example:

```ini
[server]
root_url = https://grafana.example.internal
```

This is important when Grafana is behind:

```text
ALB
Ingress
Reverse proxy
SSO
HTTPS
```

---

# 8. Why Root URL Is Important

An incorrect root URL can cause:

```text
Login redirect problems
OAuth callback failures
Incorrect dashboard links
Incorrect alert URLs
Redirect loops
```

Production example:

```text
User
 ↓
https://grafana.example.internal
 ↓
ALB
 ↓
Grafana
```

Grafana should know that its external URL is:

```text
https://grafana.example.internal
```

---

# 9. HTTPS Configuration

Grafana can terminate TLS itself, but in Kubernetes/AWS environments TLS is often terminated at the ALB.

Common architecture:

```text
User
 ↓ HTTPS
ALB
 ↓ HTTP/internal TLS as designed
Grafana
```

If TLS terminates at the ALB:

```ini
[server]
protocol = http
```

The exact proxy settings must ensure Grafana correctly understands the external HTTPS URL.

---

# 10. HTTPS at Grafana

Grafana can also terminate TLS directly.

Conceptually:

```ini
[server]
protocol = https
http_port = 3000
cert_file = /path/to/cert
cert_key = /path/to/key
```

In Kubernetes, certificates should normally be managed using Kubernetes Secrets or the platform's certificate-management solution.

---

# 11. Kubernetes Configuration

With Helm, configuration is commonly passed through:

```yaml
grafana.ini:
  server:
    domain: grafana.example.internal
    root_url: https://grafana.example.internal
```

Example:

```yaml
grafana.ini:
  server:
    protocol: http
    http_port: 3000
    domain: grafana.example.internal
    root_url: https://grafana.example.internal
```

The exact Helm values structure should be checked against the chart version being used.

---

# 12. Environment Variables

Grafana configuration can also be supplied using environment variables.

Example:

```text
GF_SERVER_ROOT_URL=https://grafana.example.internal
```

Another example:

```text
GF_SECURITY_ADMIN_USER=admin
```

However, credentials should not be exposed in plain Git configuration.

---

# 13. Configuration Precedence

When multiple configuration mechanisms are used, understand which source controls each setting.

A practical model is:

```text
Base configuration
      ↓
grafana.ini
      ↓
Environment variables
      ↓
Runtime configuration
```

Always verify the actual effective configuration for your Grafana version and deployment method.

---

# 14. Database Configuration

Grafana needs its own database.

Development:

```text
Grafana
 ↓
SQLite
```

Production HA:

```text
Grafana A ──┐
            ├── PostgreSQL
Grafana B ──┘
```

---

# 15. PostgreSQL Configuration

Example:

```ini
[database]
type = postgres
host = postgres.example.internal:5432
name = grafana
user = grafana
```

The password should come from a Secret.

Do not configure:

```ini
password = my-production-password
```

inside Git.

---

# 16. Kubernetes Secret for Database Credentials

Conceptually:

```text
AWS Secrets Manager
        ↓
External Secrets
        ↓
Kubernetes Secret
        ↓
Grafana
        ↓
PostgreSQL
```

This keeps credentials outside source control.

---

# 17. Database Connection Pool

Grafana maintains database connections.

Configuration can include:

```ini
[database]
max_open_conn = 100
max_idle_conn = 25
```

These are examples, not universal production values.

Tune them based on:

```text
Grafana replicas
Concurrent users
Database capacity
Query workload
Database limits
```

---

# 18. Database Connection Problems

If Grafana cannot connect to PostgreSQL, check:

```text
Database hostname
Port
Username
Password
Database name
Network connectivity
Security groups
Network policies
TLS requirements
Database availability
```

From Kubernetes:

```bash
kubectl get pods -n monitoring
```

Then inspect Grafana logs:

```bash
kubectl logs deployment/grafana -n monitoring
```

---

# 19. Authentication Configuration

Authentication answers:

```text
Who is the user?
```

Grafana supports multiple authentication mechanisms.

Common examples:

```text
Local authentication
LDAP
OAuth
OIDC
SAML
Enterprise SSO
```

For enterprise environments, centralized identity is preferred.

---

# 20. Local Authentication

A simple deployment may use:

```text
Username
Password
```

However, production environments should avoid shared local accounts.

Instead:

```text
Engineer
 ↓
SSO
 ↓
Grafana
```

---

# 21. Disable Anonymous Access

For production monitoring systems, anonymous access should normally be disabled unless there is a deliberate public-use case.

Example:

```ini
[auth.anonymous]
enabled = false
```

This prevents unauthenticated users from accessing dashboards.

---

# 22. Generic OAuth / OIDC

Grafana can integrate with an OAuth/OIDC identity provider.

Conceptual configuration:

```ini
[auth.generic_oauth]
enabled = true
name = SSO
client_id = <client-id>
client_secret = <secret>
scopes = openid profile email
```

The actual settings depend on the identity provider.

---

# 23. OAuth Architecture

```text
Engineer
   ↓
Grafana
   ↓
Identity Provider
   ↓
Login
   ↓
Authorization
   ↓
Grafana
```

Example:

```text
Engineer
   ↓
Company SSO
   ↓
Identity Provider
   ↓
OIDC
   ↓
Grafana
```

---

# 24. OAuth Secrets

Never store:

```text
client_secret
```

directly in Git.

Instead:

```text
Secret Manager
      ↓
Kubernetes Secret
      ↓
Grafana
```

---

# 25. OAuth Callback URL

The identity provider needs the correct callback URL.

For example:

```text
https://grafana.example.internal/login/generic_oauth
```

The exact callback path depends on the authentication mechanism.

A wrong root URL can cause:

```text
OAuth callback mismatch
Redirect failure
Login loop
```

---

# 26. User Auto Sign-Up

Grafana can automatically create users after successful external authentication depending on configuration.

Conceptually:

```text
First login
    ↓
SSO authentication
    ↓
Grafana user created
```

Whether this is enabled should be decided according to organizational requirements.

---

# 27. Team Mapping

Enterprise authentication can map users into teams.

Example:

```text
Identity Provider
      │
      ├── Platform Team
      │       ↓
      │    Grafana Team
      │
      └── Payments Team
              ↓
          Grafana Team
```

This simplifies access management.

---

# 28. Authorization

After authentication:

```text
User
 ↓
Identity
 ↓
Team
 ↓
Role
 ↓
Dashboard / Folder access
```

Common roles include:

```text
Viewer
Editor
Admin
```

Additional RBAC capabilities depend on the Grafana edition and version.

---

# 29. Viewer Role

A viewer typically needs:

```text
View dashboards
Explore data
View alerts
```

They should not automatically receive administrative privileges.

---

# 30. Editor Role

An editor may be allowed to:

```text
Create dashboards
Modify dashboards
Create panels
Modify visualizations
```

Grant this carefully in production.

---

# 31. Admin Role

Administrators can manage broader Grafana configuration.

Examples:

```text
Users
Teams
Data sources
Server settings
Dashboards
Plugins
Alerting
```

Admin access should be restricted.

---

# 32. Data Source Configuration

Grafana needs data sources for observability backends.

Our stack includes:

```text
Prometheus
Elasticsearch / Loki
Jaeger
```

Architecture:

```text
Grafana
 ├── Prometheus
 ├── Elasticsearch / Loki
 └── Jaeger
```

---

# 33. Prometheus Data Source

Example:

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus-operated.monitoring.svc:9090
    isDefault: true
```

The Service name must match your actual Prometheus deployment.

---

# 34. Why Use Proxy Access?

With:

```yaml
access: proxy
```

the Grafana backend communicates with Prometheus.

Architecture:

```text
Browser
  ↓
Grafana
  ↓
Prometheus
```

Instead of:

```text
Browser
  ↓
Prometheus directly
```

This can simplify network access and security.

---

# 35. Elasticsearch Data Source

Conceptually:

```yaml
apiVersion: 1

datasources:
  - name: Elasticsearch
    type: elasticsearch
    access: proxy
    url: http://elasticsearch.logging.svc:9200
```

The actual URL depends on the Elasticsearch deployment.

---

# 36. Loki Data Source

If Loki is used:

```yaml
apiVersion: 1

datasources:
  - name: Loki
    type: loki
    access: proxy
    url: http://loki.logging.svc:3100
```

Loki provides log aggregation and query capabilities.

---

# 37. Jaeger Data Source

Conceptually:

```yaml
apiVersion: 1

datasources:
  - name: Jaeger
    type: jaeger
    access: proxy
    url: http://jaeger-query.tracing.svc:16686
```

The actual Service name depends on your Jaeger installation.

---

# 38. Data Source Authentication

Some backends require:

```text
Username
Password
API token
TLS certificate
CA certificate
Client certificate
```

These credentials should be stored securely.

Example architecture:

```text
Secret Manager
 ↓
Kubernetes Secret
 ↓
Grafana
 ↓
Data Source
```

---

# 39. Data Source TLS

Production backends may require TLS.

Example concept:

```text
Grafana
   ↓ HTTPS
Prometheus / Elasticsearch / Jaeger
```

Certificate validation should be enabled where appropriate.

Avoid disabling TLS verification simply to make connectivity work.

---

# 40. Dashboard Configuration

Dashboards can be:

```text
Created manually
Provisioned from files
Managed through Git
Imported
Generated automatically
```

For production:

```text
Git
 ↓
Dashboard JSON
 ↓
ArgoCD
 ↓
Grafana
```

is preferable to unmanaged manual dashboards.

---

# 41. Dashboard Provider

A provider tells Grafana where dashboard definitions are stored.

Example:

```yaml
apiVersion: 1

providers:
  - name: Kubernetes
    orgId: 1
    folder: Kubernetes
    type: file
    disableDeletion: true
    updateIntervalSeconds: 30
    options:
      path: /var/lib/grafana/dashboards/kubernetes
```

The exact provisioning settings depend on your deployment.

---

# 42. Dashboard Folder Structure

Example:

```text
dashboards/
├── kubernetes/
│   ├── cluster.json
│   ├── nodes.json
│   └── pods.json
│
├── applications/
│   ├── payment.json
│   ├── orders.json
│   └── catalog.json
│
└── infrastructure/
    ├── rds.json
    └── alb.json
```

This provides logical organization.

---

# 43. Dashboard Ownership

Every production dashboard should ideally have an owner.

Example:

```text
Kubernetes Dashboard
Owner: Platform Team

Payment Dashboard
Owner: Payments Team

Orders Dashboard
Owner: Orders Team
```

Ownership makes maintenance and incident response easier.

---

# 44. Dashboard Variables

Use variables to avoid creating separate dashboards for every service.

Example:

```text
Environment = production
Namespace = payments
Service = payment-api
Pod = payment-api-xxxx
```

Query:

```promql
rate(
  http_requests_total{
    namespace="$namespace",
    service="$service"
  }[5m]
)
```

---

# 45. Dashboard Refresh Configuration

Dashboard refresh should match the use case.

Examples:

```text
5s
10s
30s
1m
5m
```

Avoid unnecessarily aggressive refresh intervals.

A 5-second dashboard with dozens of expensive queries can put significant load on Prometheus.

---

# 46. Time Range

Use appropriate default ranges.

For real-time troubleshooting:

```text
Last 15 minutes
Last 30 minutes
Last 1 hour
```

For historical analysis:

```text
Last 24 hours
Last 7 days
Last 30 days
```

The correct range depends on the operational use case.

---

# 47. Grafana Alerting Configuration

Grafana alerting can contain:

```text
Alert Rules
Expressions
Contact Points
Notification Policies
Mute Timings
```

Architecture:

```text
Query
 ↓
Expression
 ↓
Alert Rule
 ↓
Notification Policy
 ↓
Contact Point
 ↓
Notification
```

---

# 48. Alert Rule Example

Conceptually:

```text
Metric:
CPU utilization

Condition:
> 80%

Duration:
10 minutes

Severity:
Warning
```

The actual rule implementation depends on the chosen alerting architecture.

---

# 49. Prometheus + Alertmanager Architecture

For a Prometheus-centered monitoring platform:

```text
Prometheus
   ↓
Alert Rules
   ↓
Alertmanager
   ↓
Grouping
   ↓
Routing
   ↓
Slack / Email / Pager
```

Grafana can visualize these alerts.

---

# 50. Avoid Duplicate Alerts

Do not blindly configure:

```text
Prometheus CPU Alert
+
Grafana CPU Alert
```

for the same condition.

This can produce:

```text
Duplicate notifications
Alert fatigue
Conflicting ownership
```

Choose a clear alerting responsibility model.

---

# 51. Logging Configuration

Grafana itself should be configured with appropriate logging.

Example:

```ini
[log]
mode = console
level = info
```

Possible levels include:

```text
debug
info
warn
error
```

Use `debug` temporarily when troubleshooting.

Avoid excessive debug logging in normal production operation.

---

# 52. Grafana Logs in Kubernetes

In Kubernetes, Grafana logs are usually available through:

```bash
kubectl logs deployment/grafana -n monitoring
```

These logs can then be collected by your centralized logging system.

Architecture:

```text
Grafana
 ↓
stdout
 ↓
Log Collector
 ↓
Elasticsearch / Loki
```

---

# 53. Monitoring Grafana

Grafana should monitor itself.

A conceptual flow:

```text
Grafana
 ↓
/metrics
 ↓
Prometheus
 ↓
Grafana Dashboard
```

Monitor:

```text
Availability
HTTP requests
HTTP errors
Latency
Database health
Resource usage
```

---

# 54. Grafana Metrics Endpoint

Grafana can expose metrics for Prometheus scraping.

The exact endpoint and configuration depend on the Grafana version.

Conceptually:

```text
http://grafana:3000/metrics
```

Prometheus then scrapes it.

---

# 55. Self-Monitoring

This creates:

```text
Prometheus
   ↓
Grafana metrics
   ↓
Grafana dashboard
```

This is a common observability pattern.

However, the monitoring platform should have independent recovery paths so that a failure of Grafana does not eliminate all visibility into Grafana.

---

# 56. Grafana Health Dashboard

Create a dashboard containing:

```text
Grafana Availability
HTTP Request Rate
HTTP Error Rate
HTTP Latency
Database Health
CPU
Memory
Pod Restarts
```

---

# 57. Security Configuration

Production security should include:

```text
Authentication
Authorization
HTTPS
Secure cookies
Secret management
Network restrictions
Admin restrictions
Plugin control
```

---

# 58. Secure Cookies

When using HTTPS, secure cookie configuration should be enabled appropriately.

Conceptually:

```ini
[security]
cookie_secure = true
```

This helps ensure cookies are sent securely over HTTPS.

Verify the setting against your Grafana version and proxy architecture.

---

# 59. Cookie SameSite

Grafana can also use cookie SameSite controls.

The correct setting depends on:

```text
SSO
Embedding
Reverse proxy
Browser behavior
Authentication flow
```

Do not blindly copy a value without understanding the access architecture.

---

# 60. Secret Key

Grafana uses internal cryptographic configuration.

The secret key should be stable and securely managed across replicas.

For an HA deployment:

```text
Grafana A
   │
   └── same required secret configuration
   │
Grafana B
```

Do not generate inconsistent secrets across replicas when shared state requires consistency.

---

# 61. Admin Credentials

For production:

```text
Do not:
admin / admin
```

Use:

```text
SSO
Strong credentials
Secret management
Restricted admin access
```

The initial admin account should be secured or disabled according to the chosen authentication architecture.

---

# 62. Anonymous Dashboards

Anonymous access can be useful in controlled scenarios, but should not be enabled casually.

If enabled:

```text
Anyone who can reach Grafana
        ↓
May access configured dashboards
```

For internal enterprise monitoring:

```text
Authentication
+
Authorization
```

is normally safer.

---

# 63. Embedding Dashboards

If dashboards are embedded into another application, additional security considerations apply:

```text
Authentication
Cookies
CORS
Content Security Policy
SameSite
Frame restrictions
```

Do not disable security controls simply to make embedding work.

---

# 64. Plugin Configuration

Plugins should be managed intentionally.

Production principles:

```text
Only approved plugins
Pinned versions
Security review
Upgrade testing
Git-managed configuration
```

---

# 65. Plugin Installation Through Helm

Conceptually:

```yaml
plugins:
  - grafana-piechart-panel
```

The exact plugin name must be verified before deployment.

Avoid installing plugins directly into production pods without configuration management.

---

# 66. Provisioning Configuration

Grafana provisioning commonly manages:

```text
Data Sources
Dashboard Providers
Dashboards
Alerting
```

Example repository:

```text
provisioning/
├── datasources/
├── dashboards/
└── alerting/
```

---

# 67. ConfigMap vs Secret

Use ConfigMaps for:

```text
Non-sensitive configuration
Dashboard definitions
Provisioning definitions
```

Use Secrets for:

```text
Passwords
API tokens
OAuth secrets
Database credentials
Private certificates
```

---

# 68. Example ConfigMap

Conceptually:

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: grafana-datasources
  namespace: monitoring

data:
  datasources.yaml: |
    apiVersion: 1
    datasources:
      - name: Prometheus
        type: prometheus
        access: proxy
        url: http://prometheus:9090
```

Do not put credentials in this ConfigMap.

---

# 69. Example Secret

Conceptually:

```yaml
apiVersion: v1
kind: Secret

metadata:
  name: grafana-db
  namespace: monitoring

type: Opaque

stringData:
  username: grafana
  password: <secret>
```

In production, prefer an external secret manager where appropriate.

---

# 70. Grafana Configuration Through Helm

A production values file might contain:

```yaml
replicas: 2

service:
  type: ClusterIP
  port: 80

resources:
  requests:
    cpu: 250m
    memory: 512Mi
  limits:
    cpu: 1
    memory: 1Gi

grafana.ini:
  server:
    domain: grafana.example.internal
    root_url: https://grafana.example.internal
```

Additional configuration would be added for:

```text
Authentication
Database
Data sources
Dashboards
Ingress
Security
```

---

# 71. Configuration Through GitOps

Recommended architecture:

```text
Git
│
├── values.yaml
├── dashboards/
├── provisioning/
└── secrets references
        │
        ↓
      ArgoCD
        │
        ↓
       EKS
        │
        ↓
     Grafana
```

Secrets themselves should remain in an approved secret-management system rather than being committed in plaintext.

---

# 72. Environment-Specific Configuration

Use separate configuration for:

```text
Development
Staging
Production
```

Example:

```text
grafana/
├── base/
│   ├── dashboards/
│   └── provisioning/
│
└── environments/
    ├── dev/
    ├── staging/
    └── production/
```

---

# 73. Production Values

Production may use:

```yaml
replicas: 2
```

Development:

```yaml
replicas: 1
```

Production may also have:

```text
SSO
TLS
External PostgreSQL
Internal ALB
Pod anti-affinity
PDB
Higher resources
```

---

# 74. Configuration Drift

Manual production changes create:

```text
Git:
replicas = 2

Cluster:
replicas = 3
```

This is configuration drift.

With GitOps:

```text
Git
 ↓
Desired state
 ↓
ArgoCD
 ↓
Cluster
```

ArgoCD can detect and reconcile drift depending on the configured sync behavior.

---

# 75. Dashboard Drift

Manual dashboard modification can also create drift.

Example:

```text
Git dashboard:
Payment v1

Grafana:
Payment v2
```

Dashboard-as-code reduces this problem.

---

# 76. Configuration Validation

Before production:

```bash
helm template grafana grafana/grafana \
  -n monitoring \
  -f values.yaml
```

Review:

```text
Deployment
Service
Ingress
ConfigMaps
Secrets references
Probes
Resources
```

---

# 77. Helm Diff

Where available, use a Helm diff workflow/plugin to inspect changes before applying them.

Conceptually:

```text
Current
   ↓
Diff
   ↓
Expected
```

This is useful for production reviews.

---

# 78. ArgoCD Diff

ArgoCD provides a desired-vs-live comparison:

```text
Git
 ↓
Desired state

Cluster
 ↓
Live state

        ↓

      Diff
```

This makes configuration drift easier to identify.

---

# 79. Configuration Change Process

A good production workflow:

```text
Developer
 ↓
Modify configuration
 ↓
Git commit
 ↓
Pull Request
 ↓
Review
 ↓
Testing
 ↓
Merge
 ↓
ArgoCD
 ↓
Production
```

Avoid direct production edits.

---

# 80. Grafana Configuration Testing

Test:

```text
Server starts
Authentication works
Data sources connect
Dashboards load
Alerts evaluate
Users have correct permissions
TLS works
Ingress works
Database works
```

---

# 81. Configuration Rollback

If a configuration change breaks Grafana:

```text
Bad commit
   ↓
Rollback Git commit
   ↓
ArgoCD
   ↓
Previous configuration
   ↓
Grafana
```

This is much safer than manually reconstructing the previous configuration.

---

# 82. Grafana Configuration Backup

Back up:

```text
Grafana database
Dashboards
Provisioning files
Helm values
Authentication configuration
Alert definitions
```

If dashboards and configuration are stored in Git, Git itself becomes an important recovery source.

---

# 83. Database Backup

If PostgreSQL is used:

```text
Grafana
 ↓
PostgreSQL
 ↓
Automated Backup
 ↓
Recovery
```

The backup strategy should define:

```text
RPO
RTO
Retention
Restore testing
```

---

# 84. Configuration During Disaster Recovery

If Grafana is destroyed:

```text
Git
 ↓
Helm
 ↓
Grafana
```

Then:

```text
PostgreSQL backup
 ↓
Database recovery
```

Then:

```text
Data sources
 ↓
Dashboards
 ↓
Authentication
```

are restored through configuration.

---

# 85. Grafana Production Configuration Checklist

```text
[ ] Server configuration
[ ] Correct root URL
[ ] HTTPS
[ ] DNS
[ ] Authentication
[ ] Authorization
[ ] Anonymous access disabled
[ ] Secure cookies
[ ] Database configured
[ ] PostgreSQL for HA where required
[ ] Database credentials secured
[ ] Prometheus data source
[ ] Logging data source
[ ] Jaeger data source
[ ] Dashboard provisioning
[ ] Dashboard variables
[ ] Alert configuration
[ ] Plugin management
[ ] Resource requests
[ ] Resource limits
[ ] Health probes
[ ] GitOps
[ ] Backup
[ ] Disaster recovery
```

---

# 86. Real-World EKS Configuration

A practical architecture:

```text
                          Users
                            │
                            ↓
                       Corporate SSO
                            │
                            ↓
                      Internal ALB
                            │
                            ↓
                       Grafana Ingress
                            │
                  ┌─────────┴─────────┐
                  ↓                   ↓
              Grafana A           Grafana B
                  │                   │
                  └─────────┬─────────┘
                            ↓
                       PostgreSQL
                            │
          ┌─────────────────┼─────────────────┐
          ↓                 ↓                 ↓
     Prometheus        Elasticsearch         Jaeger
       Metrics              Logs              Traces
```

Configuration:

```text
Git
 ↓
Helm Values
 ↓
ArgoCD
 ↓
EKS
 ↓
Grafana
```

---

# 87. Real-World Microservices Configuration

For your EKS microservices platform:

```text
Grafana
│
├── Cluster Dashboard
│
├── User Service
│
├── Catalog Service
│
├── Cart Service
│
├── Order Service
│
├── Payment Service
│
├── Inventory Service
│
└── Notification Service
```

Each service can have:

```text
Request rate
Error rate
Latency
CPU
Memory
Pod health
Restarts
Dependencies
```

---

# 88. Production Configuration Flow

The complete process:

```text
1. Create Helm values
          ↓
2. Configure Grafana server
          ↓
3. Configure authentication
          ↓
4. Configure database
          ↓
5. Configure data sources
          ↓
6. Configure dashboards
          ↓
7. Configure alerting
          ↓
8. Configure security
          ↓
9. Configure resources
          ↓
10. Configure ingress
          ↓
11. Store in Git
          ↓
12. Deploy with ArgoCD
          ↓
13. Validate
          ↓
14. Monitor
```

---

# 89. Troubleshooting Configuration: Grafana Won't Start

Start with:

```bash
kubectl get pods -n monitoring
```

Then:

```bash
kubectl describe pod <grafana-pod> -n monitoring
```

Then:

```bash
kubectl logs <grafana-pod> -n monitoring
```

Check:

```text
Invalid configuration
Database connection
Permission problems
Secret problems
Plugin problems
Port conflicts
```

---

# 90. Troubleshooting: Login Redirect Loop

Check:

```text
root_url
domain
HTTPS
ALB
Ingress
OAuth callback
X-Forwarded-Proto
Cookie configuration
```

Typical flow:

```text
Browser
 ↓
HTTPS
 ↓
ALB
 ↓
Grafana
 ↓
OIDC
 ↓
Grafana
```

Every layer must agree on the external URL.

---

# 91. Troubleshooting: Data Source Unavailable

Check:

```text
Grafana
 ↓
Kubernetes DNS
 ↓
Service
 ↓
Endpoints
 ↓
Backend
```

Commands:

```bash
kubectl get svc -n monitoring
```

```bash
kubectl get endpoints -n monitoring
```

```bash
kubectl get pods -n monitoring
```

---

# 92. Troubleshooting: Dashboard Has No Data

Test the backend directly.

Prometheus:

```promql
up
```

Then check:

```text
Metric exists?
Target healthy?
Correct labels?
Correct namespace?
Correct time range?
Correct variable?
```

---

# 93. Troubleshooting: Dashboard Is Slow

Check:

```text
Panel count
Query complexity
PromQL
Refresh interval
Time range
Prometheus latency
Grafana CPU
Grafana memory
Database performance
```

Start with the slowest panel.

---

# 94. Troubleshooting: Grafana OOMKilled

Check:

```bash
kubectl describe pod <grafana-pod> -n monitoring
```

Look for:

```text
Reason: OOMKilled
```

Then investigate:

```text
Large dashboards
High concurrency
Many plugins
Heavy transformations
Low memory limit
```

Increase resources only after understanding the workload.

---

# 95. Troubleshooting: Configuration Drift

If:

```text
ArgoCD = OutOfSync
```

check:

```text
Git desired state
Cluster live state
Manual changes
Helm values
Generated resources
```

Correct the source of truth rather than repeatedly editing the live resource.

---

# 96. Interview Answer: How Do You Configure Grafana?

```text
"I manage Grafana configuration primarily through Helm values,
provisioning files and Kubernetes Secrets.

I configure the server URL, authentication, database, data sources,
dashboards, alerting, security and resource settings.

For production, sensitive credentials are stored in a secret
management system rather than Git.

Dashboards and data sources are provisioned as code and deployed
through GitOps with ArgoCD.

This prevents configuration drift and gives us version control,
review and rollback capability."
```

---

# 97. Interview Answer: How Do You Configure Grafana for HA?

```text
"For HA I run multiple Grafana replicas behind a load balancer.

I use a shared external database such as PostgreSQL for Grafana
state.

I distribute replicas across nodes or availability zones using
Kubernetes scheduling controls.

I also manage the configuration, dashboards and data sources
through GitOps so every replica is consistently configured."
```

---

# 98. Interview Answer: How Do You Secure Grafana?

```text
"I expose Grafana through HTTPS and preferably keep it behind an
internal load balancer when it is an internal monitoring platform.

For authentication I prefer enterprise SSO through OIDC or another
supported identity mechanism.

I use RBAC and teams to control dashboard access, disable anonymous
access unless explicitly required, and protect database and OAuth
credentials using secret management.

I also restrict administrative access and control plugins."
```

---

# 99. Interview Answer: How Do You Manage Grafana Configuration Drift?

```text
"I keep Grafana configuration, dashboards and data-source
definitions in Git.

ArgoCD continuously compares the desired state in Git with the
live Kubernetes state.

If someone manually changes production configuration, ArgoCD can
detect the difference.

I prefer correcting the Git source of truth rather than making
repeated manual changes in the cluster."
```

---

# 100. Final Configuration Mental Model

Remember:

```text
                    Grafana Configuration
                            │
       ┌────────────────────┼────────────────────┐
       ↓                    ↓                    ↓
     Server              Security             Database
       │                    │                    │
       ↓                    ↓                    ↓
    Root URL              SSO/RBAC           PostgreSQL
    HTTPS                 TLS
       │
       └────────────────────┬────────────────────┐
                            ↓                    ↓
                       Data Sources         Dashboards
                            │                    │
                ┌───────────┼──────────┐         │
                ↓           ↓          ↓         ↓
           Prometheus    Logs       Jaeger      Git
```

The production configuration principle is:

```text
Configuration
     ↓
Code
     ↓
Git
     ↓
Review
     ↓
ArgoCD
     ↓
EKS
     ↓
Grafana
```

And for secrets:

```text
Secret Manager
     ↓
Kubernetes Secret
     ↓
Grafana
```

A well-configured Grafana installation should be **secure, reproducible, version-controlled, observable, highly available where required, and recoverable**.
