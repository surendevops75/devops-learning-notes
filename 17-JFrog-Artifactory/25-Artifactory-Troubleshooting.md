# Artifactory-Troubleshooting

## 1. Purpose

This file is a production-oriented troubleshooting guide for JFrog
Artifactory.

The goal is to troubleshoot systematically across the complete
artifact platform rather than guessing or repeatedly restarting
services.

```text
Client
  |
  v
DNS
  |
  v
Network
  |
  v
Load Balancer / Proxy
  |
  v
TLS
  |
  v
Artifactory
  |
  +--> Authentication
  +--> Authorization
  +--> Repository
  +--> Artifact
  +--> Database
  +--> Filestore
  +--> External Dependency
  |
  v
CI/CD / Kubernetes
```

This file covers:

- troubleshooting methodology
- HTTP status codes
- DNS
- network
- TLS
- load balancer
- reverse proxy
- authentication
- authorization
- repositories
- Maven
- NPM
- PyPI
- Docker
- Helm
- Kubernetes
- EKS
- CI/CD
- Jenkins
- GitHub Actions
- GitLab
- database
- storage
- performance
- high availability
- backup
- DR
- security incidents
- logs
- metrics
- common production failures
- incident response
- root-cause analysis
- interview preparation
- production troubleshooting checklist

---

# PART I — TROUBLESHOOTING FUNDAMENTALS

## 2. Troubleshooting Principle

Do not start with:

```text
restart Artifactory
```

Start with:

```text
What changed?
What is failing?
Who is affected?
When did it start?
What error is returned?
Which layer is failing?
```

---

## 3. Production Troubleshooting Flow

Use:

```text
1. Confirm the symptom
2. Define scope
3. Check recent changes
4. Identify error code
5. Check client
6. Check DNS
7. Check network
8. Check TLS
9. Check LB/proxy
10. Check Artifactory
11. Check authentication
12. Check authorization
13. Check repository
14. Check database
15. Check storage
16. Check dependency
17. Correlate logs/metrics
18. Mitigate
19. Find root cause
20. Document
```

---

## 4. Scope

Determine whether the problem affects:

```text
one user
one application
one repository
one CI pipeline
one Kubernetes cluster
all users
all repositories
```

---

## 5. Timeline

Record:

```text
first observed
last known good
deployment
configuration change
certificate change
network change
database change
```

---

# PART II — HTTP STATUS CODES

## 6. 400 Bad Request

Possible causes:

```text
malformed request
invalid API request
client configuration
proxy transformation
```

---

## 7. 401 Unauthorized

Usually indicates authentication failure.

Check:

```text
username
password
token
token expiration
credential source
registry login
```

---

## 8. 403 Forbidden

Usually authorization.

Check:

```text
user/group
permission target
repository
path
operation
```

Important:

```text
401 = identity not accepted
403 = identity accepted but operation denied
```

---

## 9. 404 Not Found

Possible causes:

```text
wrong repository
wrong artifact
wrong version
wrong path
artifact not published
repository not accessible
```

---

## 10. 409 Conflict

Possible causes depend on operation:

```text
version conflict
existing resource
repository operation conflict
```

---

## 11. 429 Too Many Requests

Possible causes:

```text
rate limiting
upstream limitation
client burst
proxy policy
```

---

## 12. 500 Internal Server Error

Investigate:

```text
Artifactory logs
database
storage
configuration
application failure
```

---

## 13. 502 Bad Gateway

Usually points toward a proxy/LB/backend communication problem.

Check:

```text
load balancer
reverse proxy
backend node
network
TLS
```

---

## 14. 503 Service Unavailable

Possible causes:

```text
all backend nodes unhealthy
maintenance
database dependency
storage issue
overload
```

---

## 15. 504 Gateway Timeout

Check:

```text
backend latency
database
storage
network
proxy timeout
large artifact operation
```

---

# PART III — FIRST RESPONSE

## 16. Production Incident

First collect:

```text
URL
repository
operation
timestamp
client
HTTP status
request ID/correlation information if available
```

---

## 17. Avoid Random Changes

Do not simultaneously change:

```text
DNS
LB
Artifactory
database
network
```

You may destroy evidence and make the incident harder to diagnose.

---

# PART IV — DNS TROUBLESHOOTING

## 18. DNS Test

Example:

```bash
nslookup artifactory.company.com
```

or:

```bash
dig artifactory.company.com
```

---

## 19. Check Resolution

Verify:

```text
expected IP
load balancer address
TTL
DNS record
```

---

## 20. DNS Failure Symptoms

```text
cannot resolve host
intermittent access
different clients resolve different targets
```

---

## 21. DNS Checklist

```text
[ ] record exists
[ ] correct target
[ ] private/public zone correct
[ ] TTL expected
[ ] resolver reachable
[ ] DR record correct
```

---

# PART V — NETWORK TROUBLESHOOTING

## 22. Connectivity

Test:

```bash
curl -vk https://artifactory.company.com/
```

Use `-k` only as a diagnostic step when certificate validation itself
is being investigated; do not use insecure TLS verification as a
production workaround.

---

## 23. Port

Typical HTTPS:

```text
443
```

Verify firewall/security-group/network-policy rules.

---

## 24. Network Layers

Check:

```text
client
route
firewall
security group
NACL
proxy
load balancer
backend
```

---

## 25. Kubernetes Network

For EKS:

```text
Pod
 |
v
Node
 |
v
VPC
 |
v
Load Balancer
 |
v
Artifactory
```

Check each boundary.

---

# PART VI — TLS TROUBLESHOOTING

## 26. TLS Errors

Examples:

```text
certificate expired
unknown CA
hostname mismatch
incomplete chain
protocol mismatch
```

---

## 27. Inspect Certificate

Example:

```bash
openssl s_client \
  -connect artifactory.company.com:443 \
  -servername artifactory.company.com
```

---

## 28. Certificate Checklist

```text
[ ] not expired
[ ] hostname matches
[ ] chain complete
[ ] trusted CA
[ ] correct certificate deployed
[ ] clients trust CA
```

---

## 29. Docker TLS Failure

If Docker cannot login:

```text
docker login artifactory.company.com
```

Check:

```text
certificate
CA trust
DNS
network
credentials
```

---

# PART VII — LOAD BALANCER

## 30. LB Health

Check:

```text
backend targets
health status
listener
TLS
timeouts
routing
```

---

## 31. One Node Failing

Symptoms:

```text
intermittent 5xx
```

Likely:

```text
one unhealthy backend
```

Check:

```text
LB target status
Artifactory node logs
node resources
database connectivity
storage
```

---

## 32. All Nodes Failing

Investigate shared dependencies:

```text
database
storage
network
certificate
configuration
```

---

# PART VIII — REVERSE PROXY

## 33. Proxy Symptoms

```text
redirect loops
wrong URL
502
504
large upload failure
```

---

## 34. Proxy Checks

Verify:

```text
backend address
TLS
host header
forwarded headers
timeouts
body size
connection limits
```

---

# PART IX — AUTHENTICATION

## 35. 401 Troubleshooting

Check:

```text
credential
token
expiration
identity provider
clock skew
client configuration
```

---

## 36. Token

Confirm:

```text
token exists
token active
token has not expired
correct identity
correct registry endpoint
```

---

## 37. SSO

If SSO fails:

```text
identity provider
certificate
redirect URL
clock
group mapping
Artifactory configuration
```

---

# PART X — AUTHORIZATION

## 38. 403 Troubleshooting

Check:

```text
user
group
permission target
repository
path
operation
```

---

## 39. Example

CI can:

```text
READ
```

but cannot:

```text
DEPLOY
```

Likely permission configuration.

---

## 40. DELETE Denied

This may be intentional.

Do not grant DELETE simply to make the error disappear.

---

# PART XI — REPOSITORY TROUBLESHOOTING

## 41. Repository Not Found

Check:

```text
repository name
URL
repository exists
virtual membership
permissions
```

---

## 42. Virtual Repository

Check:

```text
virtual repository
included repositories
order
remote availability
permissions
```

---

## 43. Remote Repository

Check:

```text
upstream URL
DNS
TLS
credentials if required
upstream status
cache
```

---

# PART XII — MAVEN TROUBLESHOOTING

## 44. Maven Dependency Failure

Example:

```text
Could not resolve dependency
```

Check:

```text
settings.xml
repository URL
credentials
permission
artifact version
remote repository
```

---

## 45. Maven 401

Likely:

```text
credentials
```

---

## 46. Maven 403

Likely:

```text
permission
```

---

## 47. Maven 404

Check:

```text
groupId
artifactId
version
repository
```

---

## 48. Maven Upload Failure

Check:

```text
DEPLOY permission
repository type
version policy
storage
```

---

# PART XIII — NPM TROUBLESHOOTING

## 49. NPM Install Failure

Check:

```text
.npmrc
registry URL
token
scope
package version
remote repository
```

---

## 50. NPM Registry

Example:

```bash
npm config get registry
```

Confirm it points to the intended Artifactory endpoint.

---

## 51. NPM 403

Check:

```text
scope permission
repository permission
token
```

---

# PART XIV — PYPI TROUBLESHOOTING

## 52. pip Install Failure

Check:

```text
index URL
trusted CA
credentials
package
version
remote repository
```

---

## 53. pip Authentication

Check the configured credentials without printing secrets.

Never expose:

```text
password
token
```

in shell history or logs.

---

# PART XV — DOCKER TROUBLESHOOTING

## 54. Docker Login

Test:

```bash
docker login artifactory.company.com
```

Check:

```text
DNS
TLS
credentials
registry URL
permissions
```

---

## 55. Docker Pull Failure

Example:

```text
pull access denied
```

Check:

```text
image name
tag
digest
authentication
READ permission
repository
```

---

## 56. Docker Push Failure

Check:

```text
DEPLOY permission
repository
image name
storage
network
```

---

## 57. Image Manifest Problems

Check:

```text
architecture
manifest
tag
digest
registry compatibility
```

---

# PART XVI — HELM TROUBLESHOOTING

## 58. Helm Pull Failure

Check:

```text
repository URL
OCI support
credentials
artifact version
TLS
```

---

## 59. Helm OCI

Verify client configuration and the exact supported Artifactory/Helm
combination.

---

# PART XVII — KUBERNETES

## 60. ImagePullBackOff

Start:

```bash
kubectl describe pod <pod-name>
```

Look for:

```text
401
403
404
TLS
DNS
timeout
```

---

## 61. 401 in Kubernetes

Likely:

```text
imagePullSecret
credential
token
```

---

## 62. 403 in Kubernetes

Likely:

```text
Artifactory READ permission
repository path
identity
```

---

## 63. 404 in Kubernetes

Check:

```text
registry
repository
image
tag
digest
```

---

## 64. TLS Error in Kubernetes

Check:

```text
CA trust
node/container runtime trust
certificate chain
DNS
```

---

## 65. EKS Network Troubleshooting

Check:

```text
Pod
Node
Route
Security Group
NACL
VPC
Load Balancer
Artifactory
```

---

# PART XVIII — JENKINS

## 66. Jenkins Download Failure

Check:

```text
credentials
repository
network
TLS
permission
```

---

## 67. Jenkins Publish Failure

Check:

```text
service identity
DEPLOY permission
repository
artifact size
storage
```

---

## 68. Jenkins Credentials

Verify the credential ID, not the secret value.

Do not print:

```text
echo $TOKEN
```

---

# PART XIX — GITHUB ACTIONS

## 69. Actions Failure

Check:

```text
secret
environment protection
runner network
OIDC/token
repository
permissions
```

---

## 70. Fork Workflow

If failure occurs only on forks:

```text
secret availability
```

may be intentional.

Do not expose production registry credentials to untrusted code.

---

# PART XX — GITLAB

## 71. GitLab Runner Failure

Check:

```text
runner network
masked variable
protected variable
branch protection
Artifactory credentials
```

---

# PART XXI — DATABASE

## 72. Database Symptoms

```text
slow requests
timeouts
500
repository operations hanging
```

---

## 73. Database Checks

Monitor:

```text
CPU
memory
connections
locks
latency
storage
replication
```

---

## 74. Connection Exhaustion

Symptoms:

```text
timeouts
new operations fail
intermittent errors
```

Investigate:

```text
connection pool
DB capacity
application load
long-running operations
```

---

# PART XXII — STORAGE

## 75. Storage Full

Symptoms:

```text
upload failures
unstable operations
errors
```

Check:

```text
capacity
growth
retention
large artifacts
```

---

## 76. Storage Latency

Symptoms:

```text
slow downloads
slow uploads
high request latency
```

Check:

```text
IOPS
throughput
latency
network storage
object storage
```

---

# PART XXIII — PERFORMANCE

## 77. Slow Artifactory

Do not immediately scale.

First identify:

```text
CPU
memory
database
storage
network
request type
artifact size
client location
```

---

## 78. High CPU

Check:

```text
request load
concurrency
large operations
background tasks
JVM/application behavior
```

---

## 79. High Memory

Check:

```text
heap
traffic
cache
large requests
memory leaks
JVM configuration
```

Use supported JFrog guidance for JVM tuning.

---

## 80. High Latency

Break latency into:

```text
client
network
LB
Artifactory
database
storage
upstream
```

---

# PART XXIV — HIGH AVAILABILITY

## 81. Node Failure

Expected:

```text
LB detects failure
 ↓
traffic shifts
 ↓
remaining nodes serve
```

---

## 82. HA Failure

If all nodes fail:

```text
check shared dependencies
```

Especially:

```text
database
storage
network
DNS
TLS
```

---

## 83. Rolling Upgrade Issue

If one upgraded node fails:

```text
remove from traffic
 ↓
inspect logs
 ↓
rollback or remediate
 ↓
validate
```

Do not upgrade remaining nodes until the issue is understood.

---

# PART XXV — LOGGING

## 84. Logs to Collect

```text
Artifactory logs
request logs
access logs
LB logs
proxy logs
database logs
Kubernetes events
CI logs
```

Exact filenames vary by deployment/version.

---

## 85. Correlation

Correlate:

```text
timestamp
request
user
repository
artifact
node
CI build
```

---

# PART XXVI — METRICS

## 86. Core Metrics

Monitor:

```text
request rate
error rate
latency
CPU
memory
storage
database
network
```

---

## 87. Saturation

Watch:

```text
CPU saturation
memory pressure
disk capacity
network bandwidth
database connections
```

---

# PART XXVII — SECURITY INCIDENT

## 88. Malicious Artifact

If malicious artifact is discovered:

```text
Stop promotion
 ↓
Identify artifact
 ↓
Identify consumers
 ↓
Quarantine
 ↓
Review audit
 ↓
Rotate credentials if needed
 ↓
Remove/replace artifact
 ↓
Scan affected systems
```

---

## 89. Leaked Token

```text
Revoke
 ↓
Audit
 ↓
Identify scope
 ↓
Rotate
 ↓
Validate
```

---

## 90. Unauthorized Upload

Check:

```text
identity
token
permission
CI job
audit logs
artifact provenance
```

---

# PART XXVIII — BACKUP AND DR

## 91. Restore Problem

Check:

```text
backup integrity
database
filestore
configuration
version compatibility
credentials
```

---

## 92. DR Problem

Check:

```text
DNS
LB
database
storage
credentials
TLS
network
```

---

# PART XXIX — INCIDENT RESPONSE

## 93. Mitigation First

During a major outage:

```text
Restore service safely
```

before spending excessive time proving the root cause.

---

## 94. Evidence

Preserve:

```text
logs
metrics
audit events
configuration
timestamps
screenshots
commands
```

---

## 95. Root Cause

After mitigation:

```text
Timeline
 ↓
Trigger
 ↓
Failure mechanism
 ↓
Impact
 ↓
Why controls failed
 ↓
Corrective actions
```

---

# PART XXX — ROOT CAUSE ANALYSIS

## 96. Five Whys

Example:

```text
Why did deployment fail?
 ↓
Image pull failed.

Why?
 ↓
403 from Artifactory.

Why?
 ↓
Runtime identity lost READ permission.

Why?
 ↓
Permission target was changed.

Why?
 ↓
Unreviewed configuration change.
```

Root cause:

```text
change governance failure
```

not simply:

```text
Kubernetes failed
```

---

# PART XXXI — PRODUCTION SCENARIOS

## 97. Scenario — Everyone Gets 503

Check:

```text
DNS
LB
all Artifactory nodes
database
storage
network
```

---

## 98. Scenario — One Team Gets 403

Likely:

```text
permission
```

Check:

```text
group
repository
path
operation
```

---

## 99. Scenario — Only Docker Pulls Fail

Focus:

```text
Docker registry
TLS
image repository
credentials
manifest
storage
```

---

## 100. Scenario — Maven Works, NPM Fails

Focus on:

```text
NPM repository
.npmrc
scope
token
permission
```

Do not assume the entire Artifactory platform is broken.

---

## 101. Scenario — CI Fails After Certificate Rotation

Check:

```text
new certificate
CA chain
runner trust store
hostname
proxy
```

---

## 102. Scenario — EKS Only Fails

Focus on:

```text
VPC
security groups
DNS
node trust
imagePullSecret
network policy
```

---

# PART XXXII — TROUBLESHOOTING COMMANDS

## 103. DNS

```bash
nslookup artifactory.company.com
dig artifactory.company.com
```

---

## 104. HTTP

```bash
curl -v https://artifactory.company.com/
```

---

## 105. TLS

```bash
openssl s_client \
  -connect artifactory.company.com:443 \
  -servername artifactory.company.com
```

---

## 106. Kubernetes

```bash
kubectl get pods -A
kubectl describe pod <pod>
kubectl get events -A --sort-by=.lastTimestamp
```

---

## 107. Docker

```bash
docker login artifactory.company.com
docker pull <image>
```

Use sanitized output when sharing logs.

---

# PART XXXIII — TROUBLESHOOTING DECISION TREE

## 108. Client Cannot Connect

```text
Can DNS resolve?
 |
 +-- NO --> DNS
 |
 +-- YES
       |
       v
Can TCP/TLS connect?
       |
       +-- NO --> Network/TLS/LB
       |
       +-- YES
             |
             v
HTTP status?
```

---

## 109. 401

```text
Credentials
 ↓
Token
 ↓
Identity provider
 ↓
Client configuration
```

---

## 110. 403

```text
User
 ↓
Group
 ↓
Permission target
 ↓
Repository
 ↓
Path
 ↓
Operation
```

---

## 111. 404

```text
Repository
 ↓
Artifact
 ↓
Version
 ↓
Path
 ↓
Virtual/remote configuration
```

---

## 112. 5xx

```text
LB
 ↓
Artifactory
 ↓
Database
 ↓
Storage
 ↓
Network
```

---

# PART XXXIV — PRODUCTION CHECKLIST

## 113. Connectivity

```text
[ ] DNS
[ ] network
[ ] TLS
[ ] LB
[ ] proxy
```

---

## 114. Identity

```text
[ ] authentication
[ ] token
[ ] SSO
[ ] service identity
```

---

## 115. Authorization

```text
[ ] group
[ ] permission target
[ ] repository
[ ] path
[ ] operation
```

---

## 116. Platform

```text
[ ] Artifactory node
[ ] database
[ ] storage
[ ] capacity
[ ] HA
```

---

## 117. Client

```text
[ ] Maven
[ ] NPM
[ ] PyPI
[ ] Docker
[ ] Helm
[ ] Kubernetes
```

---

## 118. Operations

```text
[ ] logs
[ ] metrics
[ ] audit
[ ] recent changes
[ ] backup
[ ] DR
```

---

# PART XXXV — INTERVIEW PREPARATION

## 119. How Do You Troubleshoot Artifactory 403?

Answer:

```text
I first confirm the identity and operation, then inspect group
membership, permission targets, repository scope and path permissions.
I compare the failing request with a known-good identity and review
recent permission changes before modifying access.
```

---

## 120. How Do You Troubleshoot ImagePullBackOff?

Answer:

```text
I inspect kubectl describe output and classify the error as
authentication, authorization, DNS, TLS, network, repository or
artifact-not-found. Then I test the same registry endpoint from an
appropriate environment and validate the runtime identity and image
reference.
```

---

## 121. How Do You Troubleshoot 503?

Answer:

```text
I check load-balancer target health first, then Artifactory node
health and shared dependencies such as database, storage and network.
I correlate the failure timestamp with application and infrastructure
logs before taking recovery action.
```

---

## 122. How Do You Troubleshoot Slow Artifactory?

Answer:

```text
I break the latency into client, network, load balancer, Artifactory,
database and storage layers. I review request rate, CPU, memory,
database latency, storage latency and artifact size to identify the
actual bottleneck rather than scaling blindly.
```

---

## 123. How Do You Troubleshoot CI Publish Failure?

Answer:

```text
I verify the registry endpoint, TLS, service identity, token,
repository, DEPLOY permission, artifact size and storage. I also
check whether the failure affects one pipeline or all pipelines to
determine whether it is identity-specific or platform-wide.
```

---

## 124. What Is Your Production Troubleshooting Approach?

Answer:

```text
I start with impact and scope, identify the exact error and timeline,
check recent changes, then troubleshoot layer by layer from DNS and
network through load balancing, Artifactory, authentication,
authorization, database and storage. I mitigate safely, preserve
evidence, identify root cause and document preventive actions.
```

---

# PART XXXVI — GOLDEN RULES

## 125. Rules

```text
1. Troubleshoot systematically, not randomly.

2. Start with scope and impact.

3. Record the exact error and timestamp.

4. Check recent changes early.

5. 401 generally points to authentication.

6. 403 generally points to authorization.

7. 404 generally requires repository/artifact/path investigation.

8. 5xx requires platform/dependency investigation.

9. Check DNS before assuming Artifactory is down.

10. Check TLS before changing application configuration.

11. Check the load balancer before restarting nodes.

12. Check shared dependencies when every node fails.

13. Do not grant permissions just to hide a 403.

14. Do not disable TLS verification as a permanent workaround.

15. Do not print secrets while troubleshooting.

16. Compare failing and known-good requests.

17. Separate client problems from platform problems.

18. Use logs and metrics together.

19. Preserve evidence during security incidents.

20. Mitigate production impact safely.

21. Do not make many unrelated changes simultaneously.

22. Validate every remediation.

23. Test the exact package ecosystem that is failing.

24. Kubernetes ImagePullBackOff is a symptom, not a root cause.

25. Docker failures require registry-specific investigation.

26. CI failures require identity and permission investigation.

27. Storage and database are common shared dependencies.

28. Capacity problems can appear as application failures.

29. Test backups instead of assuming they work.

30. Document root cause and preventive actions.

31. Automate recurring health checks where appropriate.

32. Maintain current runbooks.

33. Learn the platform architecture before troubleshooting production.

34. Never declare an incident resolved until representative traffic
    succeeds.

35. Validate the exact Artifactory version and deployment topology
    before applying version-specific remediation.
```

---