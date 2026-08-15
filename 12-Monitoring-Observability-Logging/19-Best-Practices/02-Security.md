# 02 - Security

> Production Observability Security — Securing Prometheus, Grafana, ELK, Kubernetes/EKS, Metrics, Logs, Alerting, Dashboards, Credentials, Network Access, RBAC, TLS, Secrets, Auditability, Data Protection, Incident Response and DevSecOps Best Practices

---

# 1. Purpose

Observability systems contain sensitive production information.

Metrics can reveal:

    Infrastructure topology
    Service names
    Internal endpoints
    Capacity
    Deployment versions
    Traffic patterns

Logs can contain:

    User information
    Request data
    Error details
    Database information
    Authentication metadata

Dashboards can expose:

    Production architecture
    Business metrics
    Operational weaknesses
    Security events

Therefore:

> Observability data must be secured like production data.

Security must cover:

    Collection
       |
       v
    Transport
       |
       v
    Storage
       |
       v
    Query
       |
       v
    Visualization
       |
       v
    Alerting
       |
       v
    Retention / Deletion

---

# 2. Security Objectives

A production observability platform should provide:

    Confidentiality
    Integrity
    Availability
    Authentication
    Authorization
    Auditability
    Data protection
    Controlled retention

The objective is not simply:

> "Put a password on Grafana."

Security must protect the complete telemetry lifecycle.

---

# 3. Observability Security Architecture

Typical architecture:

    Applications
         |
         +---- Metrics ----> Prometheus
         |
         +---- Logs -------> Log Collector
         |                         |
         |                         v
         |                    Logstash
         |                         |
         |                         v
         |                   Elasticsearch
         |
         +---- Traces ------> Trace Backend

    Prometheus ---> Grafana
    Elasticsearch -> Kibana

Security controls:

    Authentication
    Authorization
    TLS
    Network policies
    Security groups
    Secrets management
    RBAC
    Encryption
    Audit logging
    Retention policies

---

# 4. Threat Model

Potential attackers include:

    External attackers
    Compromised workloads
    Malicious insiders
    Stolen credentials
    Overprivileged users
    Compromised CI/CD accounts
    Misconfigured services

Potential attacks:

    Unauthorized dashboard access
    Log data theft
    Credential exposure
    Metric manipulation
    Alert manipulation
    Data deletion
    Elasticsearch abuse
    Prometheus endpoint exposure
    Kubernetes privilege escalation

---

# 5. Principle of Least Privilege

Every identity should receive only the permissions required.

Example:

    Grafana Viewer
        |
        v
    Read dashboards

should not automatically receive:

    Kubernetes admin
    Elasticsearch admin
    AWS administrator

Likewise:

    Log collector

should not receive unnecessary cluster-wide permissions.

---

# 6. Authentication vs Authorization

Authentication asks:

> Who are you?

Authorization asks:

> What are you allowed to do?

Example:

    User logs into Grafana
           |
           v
    Authentication
           |
           v
    User identity
           |
           v
    Authorization
           |
           v
    Allowed dashboards/data

Both controls are required.

---

# 7. Strong Authentication

For production observability:

Prefer:

    SSO
    OIDC
    SAML where applicable
    MFA
    Short-lived credentials

Avoid shared accounts such as:

    admin/admin

Avoid sharing one Grafana administrator account across the team.

---

# 8. Grafana Authentication

Production Grafana should not rely on weak local credentials alone when centralized identity is available.

Typical architecture:

    Engineer
       |
       v
    Corporate SSO
       |
       v
    Grafana
       |
       v
    Role assignment

Roles may include:

    Viewer
    Editor
    Admin

Use the minimum role required.

---

# 9. Grafana Role Separation

Example:

## Viewer

Can:

    View dashboards
    Query allowed data

Cannot:

    Modify dashboards
    Change data sources
    Manage users

## Editor

Can:

    Create/edit dashboards

Should not automatically receive:

    System administration privileges

## Admin

Reserved for:

    Platform administrators

---

# 10. Grafana Security Best Practices

Protect:

    Login
    API access
    Data sources
    Dashboards
    Alert configuration
    User management

Also:

    Enable HTTPS
    Restrict network access
    Use SSO
    Use RBAC
    Rotate credentials
    Audit administrative changes

---

# 11. Grafana API Security

Grafana APIs can provide significant administrative capabilities.

Protect API tokens.

Do not:

    Hard-code tokens in Git
    Put tokens in Dockerfiles
    Put tokens in public CI logs

Use:

    Secret management
    Environment-specific credentials
    Short-lived credentials where supported

---

# 12. Grafana Data Source Security

A data source may contain credentials.

Examples:

    Prometheus
    Elasticsearch
    Loki
    SQL databases

Do not expose credentials through:

    Dashboard variables
    URLs
    Logs
    Source code

Use secure credential storage.

---

# 13. Prometheus Security

Prometheus often contains detailed infrastructure information.

Protect access to:

    Prometheus UI
    Query API
    Configuration
    Runtime endpoints
    Target information

Do not expose Prometheus directly to the public internet without a strong security design.

---

# 14. Prometheus Is Usually Not an Internet-Facing Application

A safer architecture:

    Users
       |
       v
    Private network
       |
       v
    Prometheus

Instead of:

    Internet
       |
       v
    Prometheus

Prometheus should generally be accessible only from trusted networks or through authenticated controlled access.

---

# 15. Prometheus Query Abuse

PromQL queries can be expensive.

An unauthorized user could submit expensive queries and consume:

    CPU
    Memory
    Query resources

Control:

    Access
    Query scope
    User permissions
    Infrastructure capacity

---

# 16. Prometheus Configuration Security

Protect:

    prometheus.yml
    Alert rules
    Recording rules
    Credentials
    Service discovery configuration

Do not store passwords or tokens directly in Git repositories.

Use secure secret injection mechanisms.

---

# 17. Prometheus Service Discovery Security

Service discovery may expose:

    Pod names
    IP addresses
    Service names
    Internal endpoints

Limit access to discovery information.

In Kubernetes use appropriate:

    RBAC
    ServiceAccounts
    NetworkPolicy

---

# 18. Prometheus Kubernetes RBAC

Prometheus may need permission to discover Kubernetes resources.

Do not give:

    cluster-admin

unless absolutely required.

Use a dedicated:

    ServiceAccount
    ClusterRole
    ClusterRoleBinding

with only required permissions.

---

# 19. Prometheus ServiceAccount

Example conceptual permissions:

    get
    list
    watch

for only required resources such as:

    pods
    services
    endpoints
    nodes

Review permissions regularly.

---

# 20. Kubernetes RBAC Best Practice

Separate identities:

    Prometheus ServiceAccount
    Log Collector ServiceAccount
    ArgoCD ServiceAccount
    Application ServiceAccount

Do not reuse a highly privileged ServiceAccount across workloads.

---

# 21. Network Segmentation

Separate observability traffic where practical.

Example:

    Application Network
          |
          v
    Telemetry Collection
          |
          v
    Observability Storage
          |
          v
    Engineer Access

Do not allow every workload to directly access every observability component.

---

# 22. Kubernetes NetworkPolicy

Use NetworkPolicy to restrict communication.

Example concept:

    Application
       |
       v
    Metrics collector

Allow:

    Application -> Metrics endpoint

Do not automatically allow:

    Application -> Elasticsearch
    Application -> Grafana
    Application -> Prometheus administration endpoints

---

# 23. EKS Security Groups

For AWS networking, restrict security groups by:

    Source
    Destination
    Port
    Protocol

Example:

    Engineer network
       |
       v
    Grafana:443

Do not expose:

    Grafana:3000

to:

    0.0.0.0/0

unless there is an explicit security architecture requiring it.

---

# 24. Private Subnets

Where practical, keep internal observability systems in private network locations.

Example:

    Internet
       |
       v
    Controlled access layer
       |
       v
    Private Grafana
       |
       v
    Private Prometheus
       |
       v
    Private Elasticsearch

This reduces exposure.

---

# 25. TLS

Use encryption in transit for sensitive telemetry.

Protect connections between:

    Engineers -> Grafana
    Grafana -> Prometheus
    Grafana -> Elasticsearch
    Collectors -> Logstash
    Logstash -> Elasticsearch
    Applications -> Telemetry endpoints where applicable

---

# 26. TLS Certificate Management

Certificates should have:

    Validity monitoring
    Renewal process
    Ownership
    Expiry alerts

Certificate expiration can become a production observability outage.

---

# 27. Certificate Rotation

Use controlled rotation.

Avoid:

    Manually replacing certificates on every server.

Prefer:

    Automated certificate management

where supported by the platform.

After rotation verify:

    Connectivity
    Trust chain
    Client validation

---

# 28. Encryption at Rest

Protect stored telemetry.

Examples:

    Elasticsearch storage
    Prometheus persistent volumes
    Backup storage
    Snapshots

For AWS environments, use appropriate encryption mechanisms such as:

    KMS-managed encryption

where applicable.

---

# 29. Key Management

Encryption is only as secure as key management.

Protect:

    KMS keys
    Key policies
    Access permissions
    Rotation configuration

Do not place encryption keys directly in application repositories.

---

# 30. Secrets Management

Secrets may include:

    Grafana admin credentials
    Elasticsearch credentials
    SMTP credentials
    API tokens
    TLS private keys
    Cloud credentials

Use:

    Kubernetes Secrets with appropriate controls
    AWS Secrets Manager
    AWS Systems Manager Parameter Store
    External secret-management systems

Do not commit secrets to Git.

---

# 31. Kubernetes Secrets Are Not Automatically Safe

A Kubernetes Secret is not equivalent to:

> Perfectly secure encrypted secret storage.

Security also depends on:

    RBAC
    Encryption at rest
    API access
    Namespace isolation
    Audit logging

Protect access to Secrets.

---

# 32. Secret Exposure in Environment Variables

Environment variables can accidentally appear in:

    Process inspection
    Debug output
    Crash reports
    Logs
    Diagnostic dumps

Avoid unnecessary secret exposure.

Use appropriate secret injection mechanisms.

---

# 33. Secret Exposure in Logs

Example bad log:

    DB_PASSWORD=SuperSecret123

This is a serious security problem.

Use:

    Redaction
    Filtering
    Structured logging controls

If a credential is exposed:

    Rotate it.

Do not assume deleting the log line is sufficient.

---

# 34. Authorization Header Redaction

Never store raw:

    Authorization: Bearer <token>

in centralized logs.

Redact:

    Authorization
    Cookie
    Set-Cookie
    API-Key

where appropriate.

---

# 35. Personal Data in Logs

Potentially sensitive data:

    Email
    Phone
    Address
    Account identifier
    IP address depending on context
    Payment-related information

Determine:

    What is required for debugging?

Then collect only what is necessary.

---

# 36. Data Minimization

A useful rule:

> Do not collect sensitive data merely because the logging system can store it.

Collect:

    Minimum required data

Retain:

    Minimum required duration

Expose:

    Minimum required audience

---

# 37. Log Redaction

Redaction can remove sensitive values before storage.

Example:

    password=********
    token=********

Prefer prevention at the application/logging layer where possible.

Do not rely only on Elasticsearch queries to hide secrets after they have already been ingested.

---

# 38. Structured Logging Security

Structured logs make filtering easier.

Example:

    {
      "level": "ERROR",
      "service": "payment",
      "request_id": "abc",
      "error_code": "PAYMENT_TIMEOUT"
    }

Avoid:

    {
      "password": "..."
    }

---

# 39. Elasticsearch Security

Protect:

    Cluster
    REST API
    Nodes
    Indexes
    Snapshots
    Kibana

Never treat Elasticsearch as a public unauthenticated data store.

---

# 40. Elasticsearch Network Exposure

Bad:

    Internet
       |
       v
    Elasticsearch:9200

Better:

    Private network
       |
       v
    Controlled clients
       |
       v
    Elasticsearch

Restrict port access.

---

# 41. Elasticsearch Authentication

Require authenticated access for:

    REST API
    Kibana
    Administrative operations

Use service-specific identities where possible.

---

# 42. Elasticsearch Authorization

Different users may need access to different:

    Indexes
    Data
    Operations

Example:

    Application team
        -> application-* indexes

    Security team
        -> security-* indexes

Use least privilege.

---

# 43. Elasticsearch Index Security

Protect indexes containing:

    Authentication logs
    User data
    Security events
    Application logs

Do not give every user unrestricted access.

---

# 44. Kibana Security

Kibana provides access to Elasticsearch data.

Protect:

    Kibana login
    Saved objects
    Data views
    Discover
    Administrative functions

Use:

    SSO
    RBAC
    TLS
    Network restrictions

---

# 45. Kibana Spaces / Access Separation

Separate environments or teams where required.

Example:

    Production
    Staging
    Security

Users should only see the information needed for their role.

---

# 46. Elasticsearch Snapshots

Backups may contain the same sensitive information as the live cluster.

Protect:

    Snapshot repository
    IAM permissions
    Encryption
    Access
    Retention

A secure cluster with an insecure backup is still a security risk.

---

# 47. Log Retention Security

Long retention increases:

    Exposure window
    Storage cost
    Attack impact

Define retention based on:

    Operational requirements
    Compliance
    Security
    Incident investigation

Delete data when it is no longer required.

---

# 48. Immutable Security Logs

Some security/audit logs may require stronger protection.

Consider:

    Restricted write access
    Restricted deletion
    Longer retention
    Separate storage
    Integrity controls

Security logs should not be easily altered by the same identities being monitored.

---

# 49. Audit Logging

Record important administrative actions.

Examples:

    Grafana user changes
    Dashboard permission changes
    Elasticsearch administration
    Kubernetes RBAC changes
    AWS IAM changes
    Alert rule changes

Audit logs help answer:

> Who changed what and when?

---

# 50. Kubernetes Audit Logs

Kubernetes audit logging can capture API activity.

Useful for investigating:

    Who changed a deployment?
    Who deleted a resource?
    Who modified RBAC?
    Who accessed sensitive resources?

Use appropriate retention and access controls.

---

# 51. AWS Audit Sources

For AWS environments, security investigations may use services such as:

    CloudTrail
    VPC Flow Logs
    AWS Config
    Security Hub

These complement application observability.

Note:

> Operational application monitoring and AWS security auditing serve different purposes.

---

# 52. Observability vs Security Monitoring

Observability asks:

    Is the application healthy?

Security monitoring asks:

    Is someone abusing or compromising the system?

They overlap but are not identical.

Examples:

    500 errors -> observability

    Unusual IAM activity -> security monitoring

Both should be integrated operationally.

---

# 53. Protecting Alert Channels

Alerts may contain sensitive information.

Protect:

    Email
    Slack/Teams channels
    PagerDuty-style systems
    Webhooks

Avoid sending:

    Passwords
    Tokens
    Sensitive request payloads

through alert notifications.

---

# 54. Alert Injection

If alert messages include unsanitized user-controlled data, attackers may inject misleading content.

Example:

    Username
    URL
    Error message

Sanitize alert content where appropriate.

---

# 55. Webhook Security

Webhook endpoints should use:

    Authentication
    TLS
    Signature validation
    IP restrictions where appropriate
    Secret rotation

Do not expose unauthenticated administrative webhooks.

---

# 56. Notification Credentials

Protect:

    SMTP passwords
    Webhook tokens
    API keys

Use secret management.

Do not put notification credentials into:

    Git
    Public dashboards
    Pipeline logs

---

# 57. Prometheus Exporter Security

Exporters can expose operational information.

Examples:

    Node exporter
    Database exporter

Restrict access to:

    /metrics

where necessary.

Do not expose exporter endpoints publicly.

---

# 58. Blackbox Monitoring Security

Synthetic probes may access:

    Internal URLs
    Login endpoints
    Sensitive services

Secure:

    Probe credentials
    Target definitions
    Probe network access

Do not store real user credentials unnecessarily.

---

# 59. Monitoring Endpoint Authentication

An application's:

    /metrics

endpoint may expose:

    Internal metric names
    Host information
    Versions
    Business counters

Where risk requires it, protect the endpoint using network restrictions or authentication.

---

# 60. Avoid Sensitive Metric Labels

Never use labels containing:

    Password
    Token
    Email
    Full user ID
    Payment data

Metrics are not the right place for sensitive request context.

---

# 61. Metric Integrity

Attackers who can modify metrics may:

    Hide an attack
    Trigger false alerts
    Suppress detection
    Create alert storms

Restrict write access to telemetry systems.

---

# 62. Alert Rule Integrity

Unauthorized alert rule changes can disable detection.

Protect:

    Prometheus rules
    Alertmanager config
    Grafana alerts

Use:

    Version control
    Code review
    RBAC
    Deployment approvals

---

# 63. Dashboard Integrity

A malicious dashboard could:

    Hide an outage
    Show misleading metrics
    Redirect users to malicious links

Protect dashboard editing rights.

---

# 64. Monitoring Configuration as Code

Version control:

    Prometheus config
    Alert rules
    Recording rules
    Grafana dashboards
    Alertmanager config
    Logstash pipelines
    Elasticsearch policies

Use pull requests and review.

---

# 65. DevSecOps Pipeline for Observability

Example:

    Developer
       |
       v
    Git
       |
       v
    CI
       |
       +---- Lint
       +---- Security scan
       +---- Configuration validation
       +---- Secret scan
       |
       v
    Approval
       |
       v
    Deployment
       |
       v
    Production

Observability configuration should receive the same engineering discipline as application code.

---

# 66. Secret Scanning

Scan repositories for:

    Passwords
    API keys
    Tokens
    Private keys
    Cloud credentials

Useful controls include:

    Pre-commit scanning
    CI secret scanning
    Repository scanning

If a real secret is committed:

> Rotate it immediately.

Removing the file from the latest commit does not necessarily remove it from Git history.

---

# 67. Container Security

Observability containers should follow standard security practices.

Use:

    Minimal images
    Regular patching
    Non-root where supported
    Read-only filesystem where possible
    Dropped Linux capabilities
    Resource limits

---

# 68. Kubernetes Pod Security

For observability workloads, consider:

    runAsNonRoot
    readOnlyRootFilesystem
    allowPrivilegeEscalation: false
    drop capabilities
    seccomp profile

Apply only what the component supports and test thoroughly.

---

# 69. Privileged Containers

Avoid privileged containers unless explicitly required.

A compromised privileged container can significantly increase cluster risk.

Review:

    securityContext
    hostNetwork
    hostPID
    hostPath
    privileged

carefully.

---

# 70. HostPath Security

Some monitoring agents need host filesystem access.

Example:

    node exporter

If hostPath is required:

    Mount only required paths
    Use read-only mounts where possible
    Restrict the workload
    Review privileges

Host access expands the attack surface.

---

# 71. DaemonSet Security

Monitoring agents often run as:

    DaemonSets

Because they run on every node, a compromised agent can have a large blast radius.

Use:

    Minimal permissions
    Minimal image
    Network restrictions
    SecurityContext
    Regular patching

---

# 72. Container Image Security

Scan observability images for vulnerabilities.

Pipeline:

    Build
       |
       v
    Image scan
       |
       v
    Policy
       |
       v
    Registry
       |
       v
    Deployment

Use tools such as Trivy where appropriate.

---

# 73. Image Provenance

Know:

    Which image
    Which version
    Which source
    Which build
    Which dependencies

was deployed.

Use immutable image tags or digests where practical.

---

# 74. Supply Chain Security

Protect:

    Base images
    Dependencies
    Helm charts
    Kubernetes manifests
    Monitoring plugins
    Exporters

Validate artifacts before production use.

---

# 75. Helm Security

Helm values may contain:

    Credentials
    URLs
    Tokens
    Configuration

Do not commit secrets in plain text values files.

Use secure secret-management integrations.

---

# 76. Git Security

Protect repositories containing:

    Monitoring configuration
    Alert rules
    Dashboard definitions
    Log pipelines

Use:

    Branch protection
    Pull requests
    CODEOWNERS
    Secret scanning
    Signed commits where required

---

# 77. CI/CD Credentials

A pipeline deploying monitoring infrastructure should use a dedicated identity.

Avoid:

    One administrator credential for everything.

Separate:

    Build identity
    Deployment identity
    AWS identity
    Kubernetes identity

---

# 78. AWS IAM for Observability

Use separate IAM roles for:

    Prometheus integrations
    Log collectors
    Snapshot operations
    Deployment pipelines

Grant only required actions.

---

# 79. IAM Role Separation

Example:

    Log collector
       |
       +---- Required logging permissions

    Snapshot service
       |
       +---- Required snapshot permissions

    Deployment pipeline
       |
       +---- Required deployment permissions

Do not combine all into:

    AdministratorAccess

---

# 80. IRSA / Pod Identity Security

For EKS workloads needing AWS access, use workload-specific identity mechanisms.

The goal is:

    Pod
      |
      v
    Dedicated IAM role
      |
      v
    Required AWS permissions

Avoid distributing static AWS access keys inside pods.

---

# 81. Static Cloud Credentials

Bad:

    AWS_ACCESS_KEY_ID
    AWS_SECRET_ACCESS_KEY

hard-coded into:

    Deployment YAML

Better:

    EKS workload identity
    IAM role
    Short-lived credentials

---

# 82. Elasticsearch IAM / AWS Integration

Where Elasticsearch/OpenSearch is integrated with AWS identity, define:

    Who can access?
    Which indexes?
    Which operations?
    Which network paths?

Do not treat cloud-managed logging services as automatically secure simply because AWS hosts them.

---

# 83. Network Encryption

Use TLS for:

    Client -> Grafana
    Collector -> Logstash
    Logstash -> Elasticsearch
    Grafana -> Data source

Also secure:

    Internal service-to-service traffic

where the threat model requires it.

---

# 84. Mutual TLS

For highly sensitive environments, mutual TLS can provide:

    Client authentication
    Server authentication
    Encrypted transport

Use it when justified by architecture and operational capability.

Do not introduce mTLS solely because it sounds more secure; complexity must be manageable.

---

# 85. Certificate Trust

Do not disable certificate validation to solve TLS errors.

Bad emergency workaround:

    insecureSkipVerify = true

Better:

    Fix certificate
    Fix trust chain
    Fix hostname
    Fix CA configuration

---

# 86. Network Exposure Review

Regularly review:

    Public IPs
    Load balancers
    Security groups
    NetworkPolicies
    Ingress rules
    Firewall rules

Ask:

> Does this observability component actually need to be reachable from the internet?

Usually the answer is no for internal telemetry stores.

---

# 87. Port Exposure

Common internal ports may include:

    Grafana: 3000
    Prometheus: 9090
    Elasticsearch: 9200
    Kibana: 5601

These ports should not automatically be internet-accessible.

Use controlled access paths.

---

# 88. Bastion / VPN / Zero-Trust Access

For internal observability tools, access may use:

    VPN
    Zero-trust access
    Bastion
    Identity-aware proxy

The exact mechanism depends on enterprise architecture.

The important principle is:

> Authenticate and authorize before allowing access to sensitive operational data.

---

# 89. Data Residency

Telemetry may contain:

    User data
    Customer data
    IP addresses
    Application payloads

Consider:

    Region
    Data residency
    Compliance
    Cross-border transfer

before selecting storage architecture.

---

# 90. Compliance

Depending on the organization, observability data may be affected by:

    Privacy requirements
    Security standards
    Industry regulations
    Retention requirements

Do not assume logs are exempt from compliance obligations.

---

# 91. Log Deletion

Deletion policies should be:

    Documented
    Automated
    Auditable

Do not rely on engineers manually deleting old indexes.

---

# 92. Legal Hold / Incident Preservation

When security or legal investigations require evidence preservation:

    Do not delete relevant data automatically.

Coordinate retention exceptions with the appropriate security/legal process.

---

# 93. Backup Security

Backups must have:

    Encryption
    Access control
    Retention
    Integrity
    Recovery testing

A backup repository is a high-value target.

---

# 94. Restore Security

Test that restored telemetry:

    Has correct permissions
    Retains encryption
    Does not expose secrets
    Can be accessed only by authorized users

Disaster recovery must preserve security controls.

---

# 95. Monitoring Security Events

Useful security signals include:

    Failed authentication
    Privilege changes
    RBAC changes
    Unusual API activity
    Secret access
    Network anomalies
    Unexpected workload changes

These can complement security-specific platforms.

---

# 96. Anomalous Access

Example:

    Engineer normally accesses Grafana during business hours.

Suddenly:

    Login from unusual location
    Repeated failed authentication
    Administrative changes

This should be investigated according to security procedures.

---

# 97. Privilege Escalation Detection

Watch for:

    cluster-admin grants
    RoleBinding changes
    IAM policy changes
    Privileged pod creation
    HostPath access
    ServiceAccount changes

These can be high-risk events.

---

# 98. Kubernetes Audit + Observability

A useful architecture:

    Kubernetes API
       |
       v
    Audit logs
       |
       v
    Centralized logging
       |
       v
    Elasticsearch
       |
       v
    Kibana / Security analysis

This provides visibility into administrative activity.

---

# 99. Security Incident Example — Exposed Grafana

## Symptoms

Grafana was accidentally exposed publicly.

## Immediate Actions

    Confirm exposure
    Restrict network access
    Review authentication
    Inspect access logs
    Rotate potentially exposed credentials
    Review administrative changes

## RCA

    Ingress/security group misconfiguration

## Prevention

    IaC policy validation
    Security review
    Public exposure detection

---

# 100. Security Incident Example — Secret in Logs

## Symptoms

Database password appears in Kibana.

## Immediate Actions

    Rotate password
    Restrict log access
    Identify exposure period
    Search for copies
    Review access logs

## Root Cause

Application logged configuration object.

## Prevention

    Secret redaction
    Logging standards
    Automated secret detection

---

# 101. Security Incident Example — Overprivileged ServiceAccount

## Symptoms

Application ServiceAccount has:

    cluster-admin

## Immediate Actions

    Assess usage
    Reduce permissions safely
    Review audit logs
    Check for suspicious activity

## Prevention

    RBAC review
    Least privilege
    Policy enforcement

---

# 102. Security Incident Example — Elasticsearch Public Exposure

## Symptoms

Port 9200 publicly reachable.

## Immediate Actions

    Restrict network access
    Validate authentication
    Review access logs
    Rotate exposed credentials
    Assess data access

## Prevention

    Private networking
    Security group restrictions
    Automated exposure scanning

---

# 103. Security Incident Example — Compromised Monitoring Agent

## Symptoms

DaemonSet behaves unexpectedly.

## Investigation

Check:

    Image
    Hash/digest
    Recent deployment
    ServiceAccount
    Host access
    Network connections
    Logs

Because a DaemonSet runs across nodes, assess blast radius carefully.

---

# 104. Security Incident Example — Alert Tampering

## Symptoms

Expected alert did not fire.

Investigation finds:

    Alert rule modified

Actions:

    Restore known-good rule
    Review audit trail
    Identify actor
    Check other monitoring changes
    Assess security impact

---

# 105. Security Incident Example — Credential Theft

If an API token used by Grafana or Logstash is exposed:

    Revoke
       |
       v
    Rotate
       |
       v
    Update consumers
       |
       v
    Validate
       |
       v
    Review access logs

Do not only replace the secret file.

---

# 106. Secure Logging Pipeline

Recommended conceptual flow:

    Application
       |
       v
    stdout/stderr
       |
       v
    Collector
       |
       +---- Redaction
       |
       +---- Validation
       |
       v
    Logstash
       |
       v
    Elasticsearch
       |
       v
    Kibana
       |
       v
    Authorized Users

Security controls should exist before sensitive data reaches long-term storage.

---

# 107. Secure Metrics Pipeline

Conceptual flow:

    Application
       |
       v
    /metrics
       |
       v
    Prometheus
       |
       v
    Grafana
       |
       v
    Authorized Engineers

Control:

    Who can scrape?
    Who can query?
    Who can modify?
    Who can administer?

---

# 108. Secure Alerting Pipeline

    Prometheus
       |
       v
    Alert Rules
       |
       v
    Alertmanager
       |
       v
    Secure Notification
       |
       v
    On-call Team

Protect:

    Rules
    Routing
    Webhooks
    Notification credentials

---

# 109. Secure Grafana Architecture

Example:

    Engineer
       |
       v
    SSO + MFA
       |
       v
    Private Access Layer
       |
       v
    Grafana
       |
       +---- Prometheus
       |
       +---- Elasticsearch

Use:

    HTTPS
    RBAC
    Network restrictions
    Secret management

---

# 110. Secure ELK Architecture

Example:

    EKS workloads
        |
        v
    Log Collector
        |
        v
    Logstash
        |
        v
    Elasticsearch cluster
        |
        v
    Kibana
        |
        v
    SSO / RBAC

Keep Elasticsearch internal.

---

# 111. Security Controls by Layer

| Layer | Security Controls |
|---|---|
| Application | Redaction, authentication, secure logging |
| Kubernetes | RBAC, NetworkPolicy, Pod Security |
| AWS | IAM, SGs, private subnets, KMS |
| Transport | TLS, certificate validation |
| Storage | Encryption, access control |
| Grafana | SSO, MFA, RBAC |
| Prometheus | Network restrictions, RBAC |
| Elasticsearch | Auth, RBAC, encryption |
| Kibana | SSO, RBAC |
| CI/CD | Secret scanning, least privilege |
| Git | Branch protection, reviews |
| Operations | Audit logs, runbooks |

---

# 112. Security Review Checklist — Prometheus

    [ ] Private network
    [ ] Authentication/access control
    [ ] RBAC
    [ ] Kubernetes permissions minimized
    [ ] Query access controlled
    [ ] TLS where required
    [ ] Secrets protected
    [ ] Storage encrypted
    [ ] Configuration version-controlled
    [ ] Monitoring endpoints protected

---

# 113. Security Review Checklist — Grafana

    [ ] HTTPS
    [ ] SSO
    [ ] MFA through identity provider
    [ ] RBAC
    [ ] Admin accounts restricted
    [ ] API tokens protected
    [ ] Data source credentials protected
    [ ] Public exposure reviewed
    [ ] Auditability enabled where applicable

---

# 114. Security Review Checklist — ELK

    [ ] Elasticsearch not publicly exposed
    [ ] Authentication enabled
    [ ] Authorization configured
    [ ] TLS
    [ ] Encryption at rest
    [ ] Index permissions
    [ ] Kibana access control
    [ ] Snapshot protection
    [ ] Retention policy
    [ ] Log redaction
    [ ] Audit logging

---

# 115. Security Review Checklist — Kubernetes

    [ ] Dedicated ServiceAccounts
    [ ] Least-privilege RBAC
    [ ] NetworkPolicies
    [ ] Pod security controls
    [ ] Secrets protected
    [ ] No unnecessary privileged containers
    [ ] Image scanning
    [ ] SecurityContext
    [ ] Audit logging
    [ ] Resource limits

---

# 116. Security Review Checklist — AWS/EKS

    [ ] IAM least privilege
    [ ] Workload identity
    [ ] Private subnets where appropriate
    [ ] Security groups restricted
    [ ] KMS encryption
    [ ] Audit logging
    [ ] VPC flow visibility
    [ ] No unnecessary public endpoints
    [ ] Backup protection
    [ ] Region/data requirements reviewed

---

# 117. Security Review Checklist — CI/CD

    [ ] Secrets not committed
    [ ] Secret scanning
    [ ] Short-lived credentials
    [ ] Least-privilege deployment role
    [ ] Branch protection
    [ ] Code review
    [ ] Image scanning
    [ ] Configuration validation
    [ ] Production approval
    [ ] Audit trail

---

# 118. Security Review Checklist — Logging

    [ ] Structured logs
    [ ] Secrets redacted
    [ ] Sensitive fields controlled
    [ ] Access restricted
    [ ] Retention defined
    [ ] Encryption
    [ ] Backup security
    [ ] Audit logs protected
    [ ] Log integrity considered

---

# 119. Security Review Checklist — Alerting

    [ ] Notification credentials protected
    [ ] Webhooks authenticated
    [ ] Alert content sanitized
    [ ] Sensitive payloads excluded
    [ ] Routing restricted
    [ ] Rule changes reviewed
    [ ] Audit trail available

---

# 120. Security Automation

Automate checks for:

    Public Prometheus exposure
    Public Elasticsearch exposure
    Public Kibana exposure
    Overprivileged RBAC
    Secrets in Git
    Vulnerable images
    Missing TLS
    Broad security groups
    Public S3 logging buckets
    Weak IAM policies

Automation prevents configuration drift from becoming a security incident.

---

# 121. Policy as Code

Security policies can validate:

    Kubernetes manifests
    Terraform
    Helm charts
    IAM policies
    Network policies

Examples of policy goals:

    No public Elasticsearch
    No privileged container unless approved
    No cluster-admin for application accounts
    No plaintext secrets
    No unrestricted security groups

---

# 122. Terraform Security Best Practices

For observability infrastructure:

    Review plans
    Use remote state securely
    Protect state
    Restrict IAM
    Encrypt storage
    Avoid plaintext secrets
    Use modules
    Validate security groups
    Enable logging

Terraform state itself may contain sensitive information.

Protect it accordingly.

---

# 123. Terraform State Security

Treat Terraform state as sensitive.

Protect:

    Backend access
    Encryption
    IAM
    Versioning
    Locking
    Backup

Do not expose state files publicly.

---

# 124. Kubernetes Manifest Security

Before deployment review:

    ServiceAccount
    RBAC
    SecurityContext
    NetworkPolicy
    Secrets
    Ingress
    Container image
    Host access

A monitoring agent is still a production workload and must follow security standards.

---

# 125. Helm Values Security

Avoid:

    password: "real-password"

in Git.

Instead integrate:

    Secret management

and inject values securely at deployment time.

---

# 126. GitOps Security

GitOps adds auditability but also creates a high-value repository.

Protect:

    Git repository
    Branches
    ArgoCD credentials
    Deployment keys
    Repository access

A compromised GitOps repository can become a production compromise.

---

# 127. ArgoCD Security

Use:

    SSO
    RBAC
    Project boundaries
    Repository restrictions
    Least privilege

Do not allow every user to deploy arbitrary resources into production.

---

# 128. ArgoCD Repository Credentials

Protect repository credentials.

Prefer:

    Short-lived or managed credentials where possible

Do not place private repository credentials directly into plain Git manifests.

---

# 129. Production Approval

For high-risk observability/security changes:

    Developer
       |
       v
    Pull Request
       |
       v
    Security review where required
       |
       v
    Automated validation
       |
       v
    Approval
       |
       v
    Production

---

# 130. Change Management

Security-sensitive changes include:

    RBAC
    IAM
    NetworkPolicy
    SecurityGroup
    TLS
    Secrets
    Alert rules
    Log pipelines

These should receive appropriate review and auditability.

---

# 131. Break-Glass Access

Emergency administrator access should be:

    Rare
    Controlled
    Audited
    Time-limited
    Reviewed afterward

Break-glass access should not become normal access.

---

# 132. Access Reviews

Periodically review:

    Grafana users
    Kibana users
    AWS roles
    Kubernetes RBAC
    ServiceAccounts
    API tokens
    Webhooks

Remove stale access.

---

# 133. Credential Rotation

Rotate:

    Passwords
    API tokens
    Webhook secrets
    TLS certificates
    Cloud credentials

Rotation should be tested so it does not unexpectedly break production.

---

# 134. Security Testing

Test:

    Authentication
    Authorization
    Network exposure
    TLS
    Secret handling
    RBAC
    Image vulnerabilities
    API access
    Log redaction

Security controls should be validated continuously.

---

# 135. Penetration Testing Considerations

Observability systems can be included in authorized security testing.

Potential targets:

    Grafana
    Kibana
    Prometheus
    Elasticsearch
    Alertmanager
    Exporters
    Monitoring APIs

Testing must be authorized and carefully scoped.

---

# 136. Incident Response — Security + Observability

When suspicious activity occurs:

    Detect
       |
       v
    Contain
       |
       v
    Preserve evidence
       |
       v
    Investigate
       |
       v
    Eradicate
       |
       v
    Recover
       |
       v
    Improve

Do not destroy evidence during containment.

---

# 137. Security Incident Evidence

Potential evidence:

    Application logs
    Kubernetes audit logs
    AWS audit logs
    Network logs
    Authentication logs
    Grafana access logs
    Elasticsearch audit data
    Git history
    CI/CD logs

Preserve evidence according to organizational procedures.

---

# 138. Security Incident — What Changed?

The same production debugging question applies:

> What changed?

Check:

    IAM changes
    RBAC changes
    Deployment
    Image
    ConfigMap
    Secret
    NetworkPolicy
    Security group
    Ingress
    Alert rules

---

# 139. Security Monitoring Gaps

After an incident ask:

    Could we detect it?
    Did logs contain enough evidence?
    Were audit logs enabled?
    Was access attributable to an identity?
    Could we identify the timeline?
    Could we contain quickly?

Security observability should improve after each incident.

---

# 140. Common Security Mistake — Public Grafana

Why dangerous:

    Internal architecture exposed
    Dashboards accessible
    Data source access
    Potential administrative attack surface

Better:

    Private access
    SSO
    MFA
    RBAC
    TLS

---

# 141. Common Security Mistake — Public Prometheus

Prometheus can reveal:

    Pod names
    Service names
    Internal addresses
    Versions
    Infrastructure metrics

Keep it private unless there is a deliberate secure design.

---

# 142. Common Security Mistake — Public Elasticsearch

Elasticsearch contains logs that may include:

    User data
    Credentials
    Internal architecture
    Security events

Public exposure can become catastrophic.

---

# 143. Common Security Mistake — Cluster-Admin Prometheus

Prometheus generally does not need unlimited Kubernetes permissions.

Use the smallest RBAC scope required.

---

# 144. Common Security Mistake — Shared Admin Credentials

One shared account means:

    Poor attribution
    Harder auditing
    Difficult credential rotation

Use individual identities through centralized authentication.

---

# 145. Common Security Mistake — Secrets in Dashboards

Do not create dashboard variables containing:

    API keys
    Passwords
    Tokens

Dashboards may be visible to many users.

---

# 146. Common Security Mistake — Secrets in Alerts

Bad:

    Database password = ...

inside alert notification.

Alerts may reach:

    Email
    Chat
    Mobile
    Ticket systems

Use safe diagnostic information instead.

---

# 147. Common Security Mistake — Sensitive Logs in Debug Mode

Turning on DEBUG globally during an incident may expose:

    Tokens
    Headers
    Payloads
    Personal data

Use targeted and temporary debugging with controls.

---

# 148. Common Security Mistake — No Log Retention Policy

Unlimited retention increases:

    Cost
    Data exposure
    Compliance risk

Define lifecycle policies.

---

# 149. Common Security Mistake — No Audit Trail

If someone changes:

    Alert rules
    RBAC
    IAM
    Dashboards

and there is no audit trail:

    Accountability is weak.

---

# 150. Common Security Mistake — Disabling TLS to Fix Incident

Bad:

    Disable certificate verification.

Better:

    Fix certificate configuration.

Security controls should not be permanently weakened for convenience.

---

# 151. Common Security Mistake — Overly Broad Security Group

Bad:

    0.0.0.0/0 -> 9200

or:

    0.0.0.0/0 -> 9090

Use specific:

    Sources
    Ports
    Network paths

---

# 152. Common Security Mistake — Static AWS Keys in Pods

Static credentials:

    Harder to rotate
    Easy to leak
    Long-lived

Use workload identity instead.

---

# 153. Common Security Mistake — Unscanned Monitoring Images

Monitoring workloads have significant access.

A vulnerable image can become an attack path.

Scan and patch regularly.

---

# 154. Common Security Mistake — Ignoring DaemonSet Blast Radius

A compromised DaemonSet may run:

    On every node

Therefore its:

    RBAC
    Host mounts
    Privileges
    Image

must receive strong security review.

---

# 155. Common Security Mistake — Trusting Internal Networks

"Internal" does not automatically mean trusted.

Use:

    Authentication
    Authorization
    Network segmentation
    TLS where appropriate

Defense in depth matters.

---

# 156. Common Security Mistake — No Access Review

Former employees, old contractors or unused service accounts may retain access.

Perform periodic access reviews.

---

# 157. Common Security Mistake — No Recovery Testing

If encrypted backups cannot be restored because the key or permissions are unavailable:

    Recovery fails.

Test recovery.

---

# 158. Production Security Checklist

## Identity

    [ ] SSO
    [ ] MFA
    [ ] Individual accounts
    [ ] Least privilege
    [ ] Access reviews

## Network

    [ ] Private endpoints
    [ ] Security groups
    [ ] NetworkPolicies
    [ ] Restricted ports
    [ ] No unnecessary public exposure

## Encryption

    [ ] TLS
    [ ] Certificate management
    [ ] Encryption at rest
    [ ] KMS/key controls

## Secrets

    [ ] Secret manager
    [ ] No secrets in Git
    [ ] Rotation
    [ ] Redaction
    [ ] Limited access

## Kubernetes

    [ ] RBAC
    [ ] ServiceAccounts
    [ ] SecurityContext
    [ ] Pod security
    [ ] Image scanning
    [ ] Audit logging

## Observability

    [ ] Prometheus protected
    [ ] Grafana protected
    [ ] Elasticsearch protected
    [ ] Kibana protected
    [ ] Alertmanager protected

## Data

    [ ] Retention
    [ ] Backup
    [ ] Encryption
    [ ] Data minimization
    [ ] Privacy controls

## Operations

    [ ] Audit trail
    [ ] Incident response
    [ ] Runbooks
    [ ] Security testing
    [ ] Recovery testing

---

# 159. Production Security Architecture

A secure enterprise-style design:

    Engineers
        |
        v
    SSO + MFA
        |
        v
    Private Access Layer
        |
        +----------------------+
        |                      |
        v                      v
     Grafana                 Kibana
        |                      |
        v                      v
   Prometheus            Elasticsearch
        ^                      ^
        |                      |
        |                  Logstash
        |                      ^
        |                      |
    Applications -------- Collector
        |
        v
      EKS
        |
        +---- RBAC
        +---- NetworkPolicy
        +---- Workload Identity
        +---- Pod Security
        |
        v
      AWS
        |
        +---- IAM
        +---- KMS
        +---- Private Networking
        +---- Audit Logs

Security controls exist across every connection.

---

# 160. Security Depth Model

Think in layers:

    Layer 1
    Identity

       +

    Layer 2
    Network

       +

    Layer 3
    Authentication

       +

    Layer 4
    Authorization

       +

    Layer 5
    Encryption

       +

    Layer 6
    Data protection

       +

    Layer 7
    Audit

       +

    Layer 8
    Detection

       +

    Layer 9
    Response

No single control should be expected to protect the entire platform.

---

# 161. Security and Availability Tradeoff

Security controls can affect availability if poorly designed.

Examples:

    Certificate rotation failure
    RBAC misconfiguration
    NetworkPolicy mistake
    IAM permission removal
    Secret rotation failure

Therefore security changes need:

    Testing
    Rollback
    Monitoring
    Ownership

Security and reliability must be engineered together.

---

# 162. Security Deployment Validation

After security changes verify:

    Authentication works
    Authorized users can access
    Unauthorized users are blocked
    Applications still send telemetry
    Alerts still route
    Logs still flow
    Dashboards still work
    No unexpected public exposure exists

---

# 163. Security Change Rollback

Every high-risk security change should have a rollback strategy.

Examples:

    Bad NetworkPolicy
        -> Restore previous policy

    Bad RBAC
        -> Restore previous role

    Bad certificate
        -> Restore known-good certificate

    Bad IAM policy
        -> Restore previous approved policy

---

# 164. Security Baseline

Maintain a baseline for:

    Public endpoints
    Open ports
    IAM roles
    RBAC
    ServiceAccounts
    TLS certificates
    Container privileges
    NetworkPolicies

Detect deviations automatically.

---

# 165. Drift Detection

Security drift examples:

    Public endpoint created
    Security group widened
    ClusterRole expanded
    Privileged container deployed
    Alert rule disabled

GitOps and policy-as-code can help detect and prevent drift.

---

# 166. Continuous Security Monitoring

Security is not:

    One-time audit.

It should continuously evaluate:

    Configuration
    Access
    Vulnerabilities
    Network exposure
    Identity activity
    Telemetry access

---

# 167. DevSecOps Integration

Security should be integrated into:

    Plan
      |
      v
    Build
      |
      v
    Scan
      |
      v
    Test
      |
      v
    Deploy
      |
      v
    Monitor
      |
      v
    Respond

Security does not end when deployment succeeds.

---

# 168. Observability Security and DevSecOps Tools

A practical pipeline may include:

    Git
      |
      +---- Secret scanning
      |
      +---- SonarQube
      |
      +---- Trivy
      |
      +---- Configuration validation
      |
      v
    Build
      |
      v
    Artifact
      |
      v
    Deployment
      |
      v
    Observability
      |
      v
    Security monitoring

Security controls should cover both software and operational infrastructure.

---

# 169. Interview — How Do You Secure Grafana?

Strong answer:

> I keep Grafana on a controlled network path, use SSO and MFA where available, apply RBAC, restrict administrative access, protect data-source credentials, use HTTPS, secure API tokens and review dashboard permissions. I also monitor administrative changes and avoid exposing Grafana directly to the public internet without a deliberate security architecture.

---

# 170. Interview — How Do You Secure Prometheus?

Strong answer:

> I keep Prometheus private, restrict access to its UI and API, use least-privilege Kubernetes RBAC for service discovery, protect configuration and credentials, secure the network path and monitor the Prometheus service itself. I also control query access and cardinality because resource exhaustion can become an availability problem.

---

# 171. Interview — How Do You Secure ELK?

Strong answer:

> I keep Elasticsearch on a private network, enable authentication and authorization, encrypt traffic and storage, protect Kibana with centralized identity and RBAC, restrict index access, protect snapshots and define retention policies. I also make sure sensitive information is redacted before it enters the logging pipeline.

---

# 172. Interview — How Do You Prevent Secrets From Entering Logs?

Strong answer:

> I define structured logging standards, explicitly exclude credentials and authentication headers, implement application-level redaction, validate log output during testing and use secret scanning where possible. If a real credential is exposed, I rotate it immediately rather than relying only on deleting the log.

---

# 173. Interview — How Do You Secure Kubernetes Monitoring?

Strong answer:

> I use dedicated ServiceAccounts and least-privilege RBAC, restrict network communication with NetworkPolicies, apply pod security controls, scan images, avoid unnecessary host privileges and protect secrets. Monitoring agents often have broad visibility, so I treat them as high-value workloads.

---

# 174. Interview — Why Should Prometheus Not Be Public?

Strong answer:

> Prometheus can expose internal service names, pod information, infrastructure metrics and operational architecture. Its query interface can also consume significant resources. I normally keep it on a private network and expose access only through authenticated and authorized paths.

---

# 175. Interview — How Do You Protect Observability Data?

Strong answer:

> I protect it through the full lifecycle: authenticated access, least-privilege authorization, network segmentation, TLS, encryption at rest, secret management, data minimization, retention controls, audit logging and secure backups. Logs and metrics are treated as sensitive production data rather than harmless diagnostic output.

---

# 176. Interview — What If a Secret Is Found in Kibana?

Strong answer:

> I would treat it as a credential exposure. First I would determine the credential type and rotate or revoke it, restrict access to the affected logs, determine the exposure period and investigate access. Then I would identify why the application logged the secret and add redaction and automated checks to prevent recurrence.

---

# 177. Interview — What If an Observability Component Is Compromised?

Strong answer:

> I would contain the affected workload, determine its privileges and network access, preserve evidence, assess the blast radius and inspect related audit logs. Because monitoring agents can have significant permissions, I would also review ServiceAccounts, host mounts, IAM roles and other nodes or workloads that use the same artifact.

---

# 178. Interview — How Do You Secure AWS Access From EKS?

Strong answer:

> I prefer workload identity with dedicated IAM roles instead of static AWS credentials inside pods. Each workload receives only the permissions required for its AWS resources. I also review trust policies, permissions, audit logs and credential exposure paths.

---

# 179. Interview — How Do You Secure Observability Configuration?

Strong answer:

> I manage Prometheus rules, Grafana dashboards, Alertmanager configuration and log pipeline configuration through version control where practical. Changes go through code review, validation and controlled deployment. This provides an audit trail and makes rollback possible.

---

# 180. Final Security Principles

> Treat metrics, logs and dashboards as sensitive production data.

> Authentication answers who the user is; authorization determines what they can do.

> Use least privilege everywhere.

> Keep internal observability systems private whenever practical.

> Never expose Elasticsearch or Prometheus publicly without a deliberate security design.

> Protect Grafana with strong authentication, authorization and HTTPS.

> Never put passwords, tokens or private keys into logs.

> Never put secrets into Git.

> Rotate credentials immediately after confirmed exposure.

> Use workload identity instead of static AWS credentials in EKS workloads.

> Use Kubernetes RBAC instead of cluster-admin shortcuts.

> Use NetworkPolicies and AWS security groups to restrict traffic.

> Encrypt sensitive telemetry in transit and at rest.

> Protect backups and snapshots as carefully as live data.

> Monitor and audit administrative changes.

> Secure the monitoring system itself.

> Security changes can cause outages, so test them and maintain rollback procedures.

> Observability security is a continuous process, not a one-time configuration.

---

# 181. Final Production Security Mental Model

Think:

    IDENTITY
       |
       v
    AUTHENTICATION
       |
       v
    AUTHORIZATION
       |
       v
    NETWORK
       |
       v
    TLS
       |
       v
    DATA PROTECTION
       |
       v
    SECRET MANAGEMENT
       |
       v
    AUDIT
       |
       v
    DETECTION
       |
       v
    INCIDENT RESPONSE
       |
       v
    CONTINUOUS IMPROVEMENT

The objective is:

    Only authorized users
          |
          v
    access the required telemetry
          |
          v
    through controlled networks
          |
          v
    using protected identities
          |
          v
    with encrypted transport
          |
          v
    while sensitive data is minimized
          |
          v
    and all important changes are auditable.

This completes:

    19-Best-Practices/
        02-Security.md

Next:

    03-Performance.md
