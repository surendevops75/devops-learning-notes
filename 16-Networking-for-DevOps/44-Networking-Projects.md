# Networking-Projects

## 1. Purpose

This file contains production-oriented networking projects designed for a DevOps engineer working with Linux, AWS, Kubernetes and EKS.

The projects progress from fundamentals to senior-level architecture:

```text
Linux Networking
→ AWS VPC
→ Secure VPC
→ ALB/NLB
→ EKS Networking
→ NetworkPolicy
→ Private AWS Services
→ Hybrid Networking
→ Observability
→ Security
→ Production Architecture
```

Each project should be built, tested, documented and destroyed safely when cloud resources are no longer required.

---

## 2. Project Methodology

For every project follow:

```text
Requirements
→ Architecture
→ CIDR plan
→ Infrastructure
→ Security
→ Deployment
→ Testing
→ Failure testing
→ Monitoring
→ Documentation
→ Cleanup
```

---

## 3. Project 01 — Linux Network Fundamentals Lab

### Objective

Build a local Linux networking lab.

### Topics

```text
interfaces
IP addresses
routes
ports
DNS
TCP
UDP
```

### Commands

```bash
ip addr
ip route
ss -tulpn
ping
curl
dig
nc
```

### Deliverable

Create a troubleshooting document showing:

```text
source
destination
route
port
protocol
result
```

---

## 4. Project 02 — Linux Client-Server TCP Lab

### Objective

Build a simple TCP client-server application.

### Architecture

```text
Client
  |
TCP
  |
Server
```

### Tasks

```text
start listener
connect client
inspect socket
capture traffic
stop server
```

### Skills

```text
TCP
ports
socket states
packet capture
```

---

## 5. Project 03 — DNS Troubleshooting Lab

### Objective

Practice DNS resolution from Linux.

### Tasks

```bash
dig example.com
dig A example.com
dig AAAA example.com
dig +trace example.com
```

### Deliverable

Document:

```text
resolver
authoritative server
record
TTL
answer
```

---

## 6. Project 04 — Nginx Reverse Proxy

### Objective

Deploy Nginx in front of an application.

### Architecture

```text
Client
 ↓
Nginx
 ↓
Application
```

### Tasks

```text
HTTP proxy
custom domain
access logs
error logs
health endpoint
```

---

## 7. Project 05 — Nginx Load Balancing

### Objective

Run multiple application instances behind Nginx.

### Architecture

```text
             ┌→ App-1
Client → Nginx ├→ App-2
             └→ App-3
```

### Test

Stop one backend and verify remaining backends serve traffic.

---

## 8. Project 06 — AWS VPC From Scratch

### Objective

Build a production-style VPC.

### Components

```text
VPC
2+ AZs
public subnets
private subnets
route tables
Internet Gateway
NAT Gateways
```

### Deliverable

Architecture diagram and Terraform code.

---

## 9. Project 07 — Multi-AZ VPC

### Objective

Design a highly available VPC.

### Architecture

```text
              VPC
       ┌───────────────┐
       │               │
      AZ-A            AZ-B
       │               │
 Public/Private     Public/Private
```

### Validate

Terminate or isolate one AZ path and verify remaining capacity.

---

## 10. Project 08 — Production CIDR Design

### Objective

Design CIDRs before building the environment.

### Include

```text
VPC CIDR
public subnet CIDRs
private subnet CIDRs
database subnet CIDRs
future expansion
EKS Pod IP requirements
```

### Deliverable

CIDR calculation document.

---

## 11. Project 09 — Secure AWS VPC

### Objective

Implement layered network security.

### Controls

```text
Security Groups
NACLs
private subnets
restricted routes
VPC Flow Logs
```

### Validation

Prove intended traffic works and unintended traffic fails.

---

## 12. Project 10 — Private EC2 With NAT

### Objective

Deploy EC2 into a private subnet with controlled outbound access.

### Architecture

```text
Private EC2
 ↓
Private Route Table
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

### Test

Verify outbound HTTPS and absence of direct inbound internet access.

---

## 13. Project 11 — NAT Gateway Comparison

### Objective

Compare centralized and per-AZ NAT designs.

### Compare

```text
cost
availability
cross-AZ traffic
routing
operations
```

### Deliverable

Architecture recommendation.

---

## 14. Project 12 — VPC Gateway Endpoint for S3

### Objective

Access S3 without sending traffic through NAT.

### Tasks

```text
create endpoint
associate route tables
test S3
inspect route
```

### Compare

```text
NAT path
vs
Gateway Endpoint
```

---

## 15. Project 13 — VPC Interface Endpoints

### Objective

Create private connectivity to supported AWS APIs.

### Examples

```text
ECR
Secrets Manager
STS
CloudWatch
```

### Validate

```text
DNS
endpoint ENI
security group
application connectivity
```

---

## 16. Project 14 — VPC Flow Logs

### Objective

Implement network observability.

### Capture

```text
source
destination
port
protocol
action
```

### Deliverable

Investigate both ACCEPT and REJECT flows.

---

## 17. Project 15 — Security Group Lab

### Objective

Build least-privilege SG chains.

### Architecture

```text
ALB SG
 ↓
App SG
 ↓
DB SG
```

### Rules

```text
ALB → App
App → DB
```

No unnecessary public database access.

---

## 18. Project 16 — NACL Lab

### Objective

Understand stateless subnet filtering.

### Tasks

```text
allow inbound
allow return traffic
test ephemeral ports
create intentional failure
troubleshoot
```

---

## 19. Project 17 — ALB Web Application

### Objective

Deploy an application behind an AWS Application Load Balancer.

### Architecture

```text
Internet
 ↓
ALB
 ↓
Application
```

### Include

```text
health checks
target groups
security groups
TLS
logs
```

---

## 20. Project 18 — ALB With Private EC2 Targets

### Objective

Expose only the ALB publicly while keeping targets private.

### Architecture

```text
Internet
 ↓
Public ALB
 ↓
Private EC2
```

### Security

```text
ALB SG → App SG
```

---

## 21. Project 19 — NLB TCP Application

### Objective

Deploy a TCP workload behind NLB.

### Test

```text
TCP connectivity
health checks
source IP behavior
```

---

## 22. Project 20 — HTTPS/TLS Architecture

### Objective

Implement TLS at the load balancer.

### Architecture

```text
Client
 ↓ HTTPS
ALB
 ↓ HTTP/TLS
Application
```

### Deliverable

Explain where TLS terminates and the security implications.

---

## 23. Project 21 — End-to-End TLS

### Objective

Encrypt:

```text
client → ALB
ALB → application
application → database
```

where appropriate.

### Deliverable

Certificate flow diagram.

---

## 24. Project 22 — Route 53 DNS

### Objective

Create DNS records for an application.

### Tasks

```text
A/AAAA
Alias
health checks
routing policy
```

---

## 25. Project 23 — Route 53 Private DNS

### Objective

Create internal service discovery using private hosted zones.

### Architecture

```text
Private client
 ↓
Route 53 Private Hosted Zone
 ↓
Internal service
```

---

## 26. Project 24 — Route 53 Failover

### Objective

Build active/passive DNS failover.

### Architecture

```text
Primary
   |
Health
   |
Secondary
```

### Test

Make primary unhealthy and verify DNS behavior.

---

## 27. Project 25 — CloudFront + WAF + ALB

### Objective

Build secure internet-facing architecture.

### Architecture

```text
Internet
 ↓
CloudFront
 ↓
WAF
 ↓
ALB
 ↓
EKS/EC2
```

### Include

```text
TLS
WAF rules
rate limiting
logging
origin protection
```

---

## 28. Project 26 — WAF Rate Limiting

### Objective

Protect an API from excessive request rates.

### Tasks

```text
normal traffic
high traffic
rate-limit rule
logs
false-positive testing
```

---

## 29. Project 27 — EKS Cluster Networking

### Objective

Build EKS with VPC-native networking.

### Components

```text
VPC
private subnets
EKS
VPC CNI
nodes
Pods
```

### Deliverable

Pod-to-VPC network diagram.

---

## 30. Project 28 — EKS Private Nodes

### Objective

Run worker nodes only in private subnets.

### Validate

```text
node access
ECR pulls
AWS APIs
outbound internet
```

---

## 31. Project 29 — EKS Public vs Private API Endpoint

### Objective

Compare EKS API endpoint configurations.

### Evaluate

```text
public
private
public + private
```

### Deliverable

Security and operations comparison.

---

## 32. Project 30 — Kubernetes Service Networking

### Objective

Build:

```text
Deployment
Service
Endpoints
```

### Test

```text
ClusterIP
DNS
Pod-to-Service
```

---

## 33. Project 31 — Kubernetes DNS

### Objective

Build and troubleshoot service discovery.

### Test

```bash
nslookup service.namespace.svc.cluster.local
```

### Failure test

Break DNS access using NetworkPolicy and recover it.

---

## 34. Project 32 — Kubernetes NetworkPolicy

### Objective

Implement default-deny networking.

### Architecture

```text
Frontend
 ↓
Backend
 ↓
Database
```

### Policy

Allow only intended flows.

---

## 35. Project 33 — Namespace Isolation

### Objective

Prevent unrelated namespaces from communicating.

### Example

```text
frontend namespace
backend namespace
database namespace
```

Implement explicit policies.

---

## 36. Project 34 — Egress NetworkPolicy

### Objective

Control outbound traffic.

### Allow

```text
DNS
database
approved API
AWS service
```

Deny everything else.

---

## 37. Project 35 — Kubernetes Ingress

### Objective

Expose an application through Ingress.

### Architecture

```text
Internet
 ↓
ALB
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

---

## 38. Project 36 — Internal Kubernetes Ingress

### Objective

Create an internal load balancer for private applications.

### Validate

```text
internal DNS
internal routing
SG
NetworkPolicy
```

---

## 39. Project 37 — ALB Controller Project

### Objective

Deploy AWS Load Balancer Controller.

### Learn

```text
IAM
Pod identity
Ingress
Service
ALB
target groups
```

---

## 40. Project 38 — ALB IP vs Instance Targeting

### Objective

Compare target modes.

### Evaluate

```text
instance
IP
```

### Deliverable

Explain traffic paths and security implications.

---

## 41. Project 39 — Pod-to-Pod Networking

### Objective

Trace traffic between Pods on:

```text
same node
different node
different AZ
```

### Use

```bash
ip route
ss
tcpdump
```

---

## 42. Project 40 — Service-to-Service Networking

### Objective

Build multiple microservices.

```text
frontend
backend
orders
payments
database
```

Implement DNS and NetworkPolicy.

---

## 43. Project 41 — EKS to RDS

### Objective

Connect private EKS workloads to private RDS.

### Architecture

```text
Pod
 ↓
NetworkPolicy
 ↓
RDS SG
 ↓
RDS
```

### Test

```text
connection success
unauthorized connection failure
```

---

## 44. Project 42 — EKS to ElastiCache

### Objective

Secure application-to-cache traffic.

### Test

```text
6379 connectivity
SG
NetworkPolicy
DNS
```

---

## 45. Project 43 — EKS to S3 Through VPC Endpoint

### Objective

Remove unnecessary NAT dependency for S3.

### Compare

```text
Pod → NAT → Internet → S3
```

with:

```text
Pod → VPC Endpoint → S3
```

---

## 46. Project 44 — EKS Private AWS API Access

### Objective

Use private endpoints for AWS APIs where appropriate.

### Include

```text
ECR
Secrets Manager
STS
CloudWatch
```

---

## 47. Project 45 — EKS Egress Architecture

### Objective

Build controlled outbound access.

### Architecture

```text
Pod
 ↓
NetworkPolicy
 ↓
Proxy/Firewall/NAT
 ↓
Approved destination
```

---

## 48. Project 46 — EKS Network Debugging Lab

### Objective

Create intentional failures.

Break:

```text
DNS
SG
NACL
route
NetworkPolicy
Service
Ingress
```

Recover each one.

---

## 49. Project 47 — EKS DNS Incident Lab

### Objective

Simulate:

```text
CoreDNS failure
NetworkPolicy DNS block
upstream DNS failure
```

Document the troubleshooting path.

---

## 50. Project 48 — EKS TCP Incident Lab

### Objective

Simulate:

```text
port blocked
wrong targetPort
route missing
NACL return traffic blocked
```

---

## 51. Project 49 — HTTP Troubleshooting Lab

### Objective

Generate:

```text
404
401
403
429
500
502
503
504
```

Identify which layer creates each response.

---

## 52. Project 50 — Production EKS Three-Tier Architecture

### Objective

Build:

```text
Internet
 ↓
WAF
 ↓
ALB
 ↓
Frontend
 ↓
Backend
 ↓
RDS
```

### Security

```text
private nodes
NetworkPolicy
SG
TLS
secrets
logging
```

---

## 53. Project 51 — Multi-AZ EKS Application

### Objective

Deploy workloads across multiple AZs.

### Validate

```text
Pod distribution
load balancing
failure handling
database connectivity
```

---

## 54. Project 52 — EKS AZ Failure Simulation

### Objective

Test workload behavior when one AZ becomes unavailable.

### Validate

```text
Pod rescheduling
ALB target health
capacity
network paths
```

---

## 55. Project 53 — EKS IP Capacity Lab

### Objective

Understand Pod IP allocation.

### Test

Scale Pods until approaching subnet/IP limits.

Monitor:

```text
available IPs
ENIs
Pod scheduling
CNI logs
```

---

## 56. Project 54 — Prefix Delegation Lab

### Objective

Explore efficient Pod IP allocation with supported VPC CNI configurations.

### Compare

```text
traditional secondary IP allocation
prefix delegation
```

---

## 57. Project 55 — EKS CNI Troubleshooting

### Objective

Create Pod IP allocation failures.

Investigate:

```text
aws-node
IAM
ENIs
subnet capacity
instance limits
```

---

## 58. Project 56 — Karpenter Networking

### Objective

Deploy nodes dynamically and validate network requirements.

### Include

```text
subnet discovery
security groups
Pod IP capacity
node bootstrap
```

---

## 59. Project 57 — Cluster Autoscaler Networking

### Objective

Understand how node scaling interacts with:

```text
subnet capacity
ENI capacity
Pod IPs
load balancing
```

---

## 60. Project 58 — NetworkPolicy Production Baseline

### Objective

Create a reusable baseline:

```text
default deny ingress
default deny egress
DNS
monitoring
application flows
```

---

## 61. Project 59 — Zero Trust EKS Networking

### Objective

Build:

```text
identity
NetworkPolicy
mTLS
TLS
least-privilege SG
workload identity
```

---

## 62. Project 60 — Service Mesh Networking

### Objective

Deploy a service mesh in a controlled lab.

### Study

```text
sidecars
mTLS
traffic policy
retries
timeouts
observability
```

---

## 63. Project 61 — mTLS Failure Lab

### Objective

Intentionally break certificate trust.

### Troubleshoot

```text
certificate
identity
trust bundle
policy
clock
```

---

## 64. Project 62 — Kubernetes Ingress Security

### Objective

Secure external traffic.

### Include

```text
TLS
WAF
rate limiting
NetworkPolicy
authentication
```

---

## 65. Project 63 — Private Application Platform

### Objective

Build an application with no public workload endpoints.

### Architecture

```text
VPN/SSM
 ↓
Internal ALB
 ↓
EKS
 ↓
Private DB
```

---

## 66. Project 64 — Bastion vs SSM

### Objective

Compare administrative access architectures.

### Evaluate

```text
bastion
SSM
private access
auditability
attack surface
```

---

## 67. Project 65 — VPC Peering

### Objective

Connect two VPCs.

### Test

```text
VPC-A → VPC-B
```

Validate routes and security.

---

## 68. Project 66 — Transit Gateway

### Objective

Build a hub-and-spoke network.

### Architecture

```text
        TGW
      /  |  \
   VPC1 VPC2 VPC3
```

---

## 69. Project 67 — TGW Route Segmentation

### Objective

Create separate TGW route tables.

### Example

```text
Production
Shared
Inspection
```

---

## 70. Project 68 — Centralized Inspection VPC

### Objective

Route traffic through a firewall/inspection layer.

### Architecture

```text
Spoke
 ↓
TGW
 ↓
Inspection
 ↓
Destination
```

---

## 71. Project 69 — Hybrid VPN Architecture

### Objective

Connect AWS to a simulated on-prem environment.

### Include

```text
VPN
routes
security
DNS
```

---

## 72. Project 70 — Hybrid DNS

### Objective

Resolve names between AWS and on-prem.

### Include

```text
Route 53 Resolver
inbound endpoint
outbound endpoint
forwarding rules
```

---

## 73. Project 71 — Direct Connect Architecture Study

### Objective

Design a Direct Connect architecture.

### Deliverable

Compare:

```text
VPN
Direct Connect
DX + VPN backup
```

---

## 74. Project 72 — Network Firewall Project

### Objective

Implement centralized network filtering.

### Include

```text
inspection
routing
state
logging
failure behavior
```

---

## 75. Project 73 — VPC Flow Log Investigation

### Objective

Use flow logs to diagnose a failed connection.

### Create

```text
allowed traffic
blocked traffic
```

Then identify the difference.

---

## 76. Project 74 — CloudTrail Network Change Investigation

### Objective

Investigate a simulated route or SG modification.

### Deliverable

```text
who
what
when
resource
before
after
```

---

## 77. Project 75 — DNS Security Monitoring

### Objective

Monitor DNS behavior.

Look for:

```text
unexpected domains
high query volume
suspicious destinations
```

---

## 78. Project 76 — Egress Monitoring

### Objective

Identify unexpected outbound traffic from EKS.

### Correlate

```text
Pod
DNS
destination
Flow Logs
application
```

---

## 79. Project 77 — Network Security Dashboard

### Objective

Create a dashboard showing:

```text
rejected flows
NAT traffic
ALB errors
DNS errors
network latency
Pod network failures
```

---

## 80. Project 78 — Prometheus Network Metrics

### Objective

Monitor network-related Kubernetes metrics.

Track:

```text
Pod restarts
network errors
DNS
resource saturation
```

---

## 81. Project 79 — Grafana Network Dashboard

### Objective

Create operational dashboards for:

```text
ALB
NAT
EKS
CoreDNS
network traffic
```

---

## 82. Project 80 — Network Incident Runbook

### Objective

Create a reusable incident runbook.

Sections:

```text
symptom
scope
commands
decision tree
mitigation
rollback
validation
postmortem
```

---

## 83. Project 81 — Broken Network GameDay

### Objective

Intentionally introduce multiple failures.

Example:

```text
NetworkPolicy
+
DNS
+
SG
```

Teams must identify the root cause.

---

## 84. Project 82 — AZ Failure GameDay

Simulate one-AZ impairment and validate:

```text
traffic distribution
Pod rescheduling
capacity
database access
```

---

## 85. Project 83 — NAT Failure GameDay

Simulate loss of a NAT path.

Validate:

```text
external API behavior
failure detection
fallback
```

---

## 86. Project 84 — DNS Failure GameDay

Simulate resolver disruption.

Validate:

```text
application behavior
caching
alerts
recovery
```

---

## 87. Project 85 — Security Group Failure GameDay

Block a required path.

Expected workflow:

```text
detect
identify
restore
document
prevent
```

---

## 88. Project 86 — NetworkPolicy Failure GameDay

Block:

```text
frontend → backend
```

and recover it without opening unnecessary access.

---

## 89. Project 87 — Load Balancer Failure GameDay

Break:

```text
health check
target port
security group
```

and diagnose the issue.

---

## 90. Project 88 — Production Network Architecture Project

### Objective

Design a complete production platform.

### Architecture

```text
Internet
 ↓
Route 53
 ↓
CloudFront
 ↓
WAF
 ↓
ALB
 ↓
EKS
 ↓
RDS
```

### Include

```text
multi-AZ
private subnets
NAT
VPC endpoints
NetworkPolicy
Flow Logs
monitoring
```

---

## 91. Project 89 — Secure EKS Reference Architecture

Build a reusable architecture containing:

```text
private nodes
restricted API
ALB
WAF
NetworkPolicy
Pod identity
RDS
VPC endpoints
logging
monitoring
```

---

## 92. Project 90 — Senior-Level Network Architecture Capstone

### Business Requirement

Build a highly available multi-AZ e-commerce platform.

### Requirements

```text
public web traffic
private workloads
private database
external APIs
AWS services
zero-trust principles
centralized logging
DR
```

### Deliverables

```text
architecture diagram
CIDR plan
Terraform
Kubernetes manifests
NetworkPolicies
security model
traffic matrix
monitoring
runbooks
failure tests
cost analysis
```

---

## 93. Capstone Network Architecture

```text
                    Internet
                       |
                 Route 53 / CDN
                       |
                      WAF
                       |
                      ALB
                       |
        +--------------+--------------+
        |                             |
       AZ-A                          AZ-B
        |                             |
      EKS                           EKS
        |                             |
   Frontend/Backend             Frontend/Backend
        |                             |
        +-------------+---------------+
                      |
                     RDS
                      |
                 Private Data
```

---

## 94. Capstone Security Model

```text
Internet
 ↓
WAF
 ↓
ALB SG
 ↓
App SG
 ↓
NetworkPolicy
 ↓
RDS SG
```

---

## 95. Capstone Egress

```text
EKS
 ↓
NetworkPolicy
 ↓
VPC Endpoint / NAT / Firewall
 ↓
Approved destination
```

---

## 96. Capstone Observability

Collect:

```text
ALB logs
WAF logs
VPC Flow Logs
DNS logs
CloudTrail
EKS logs
application logs
metrics
traces
```

---

## 97. Capstone Failure Tests

Test:

```text
Pod failure
node failure
AZ failure
DNS failure
NAT failure
ALB target failure
database dependency failure
NetworkPolicy error
```

---

## 98. Capstone Security Tests

Verify:

```text
public DB blocked
unauthorized Pod-to-Pod blocked
unauthorized egress blocked
management ports restricted
TLS enabled
```

---

## 99. Capstone Performance Tests

Measure:

```text
latency
throughput
connection count
DNS latency
load-balancer latency
database latency
```

---

## 100. Capstone Cost Review

Evaluate:

```text
NAT
cross-AZ
TGW
firewall
load balancers
CloudFront
logging
```

---

## 101. Project Documentation Standard

Every project should include:

```text
README.md
architecture.png
terraform/
kubernetes/
scripts/
tests/
troubleshooting.md
```

where applicable.

---

## 102. Project README Template

```markdown
# Project Name

## Objective

## Architecture

## Prerequisites

## Deployment

## Configuration

## Security

## Testing

## Failure Scenarios

## Troubleshooting

## Cleanup

## Lessons Learned
```

---

## 103. Architecture Diagram Standard

Show:

```text
users
DNS
load balancers
subnets
AZs
routes
security boundaries
workloads
databases
external services
```

---

## 104. Traffic Matrix Deliverable

Every serious project should document:

| Source | Destination | Protocol | Port | Reason | Control |
|---|---|---|---:|---|---|
| Internet | ALB | TCP | 443 | HTTPS | WAF/SG |
| ALB | App | TCP | 8080 | Application | SG |
| App | DB | TCP | 5432 | Database | SG/Policy |
| App | DNS | UDP/TCP | 53 | Resolution | Policy |
| App | AWS API | TCP | 443 | AWS service | Endpoint/NAT |

---

## 105. Security Checklist

```text
[ ] No unnecessary public endpoints
[ ] Private workloads
[ ] Least privilege SG
[ ] NetworkPolicy
[ ] TLS
[ ] Restricted management
[ ] Controlled egress
[ ] VPC Flow Logs
[ ] CloudTrail
[ ] Centralized logs
```

---

## 106. Testing Checklist

```text
[ ] Happy path
[ ] Unauthorized path
[ ] DNS failure
[ ] TCP failure
[ ] TLS failure
[ ] Pod failure
[ ] Node failure
[ ] AZ failure
[ ] Dependency failure
```

---

## 107. Production Readiness Checklist

```text
[ ] Multi-AZ
[ ] Capacity planned
[ ] CIDRs documented
[ ] Routes documented
[ ] Security rules reviewed
[ ] DNS documented
[ ] Monitoring configured
[ ] Alerts configured
[ ] Runbook written
[ ] Rollback tested
[ ] Disaster recovery considered
```

---

## 108. GitHub Portfolio Presentation

For each project publish:

```text
Problem
Architecture
Implementation
Security
Testing
Failure scenario
Results
Lessons learned
```

Never publish:

```text
credentials
private keys
secrets
sensitive IPs
production configuration
```

---

## 109. Terraform Project Structure

Recommended:

```text
terraform/
├── modules/
├── environments/
│   ├── dev/
│   └── prod/
├── networking/
├── security/
└── README.md
```

---

## 110. Kubernetes Project Structure

Recommended:

```text
kubernetes/
├── namespace/
├── deployment/
├── service/
├── ingress/
├── networkpolicy/
└── monitoring/
```

---

## 111. Validation Automation

Automate:

```text
terraform validate
terraform plan
YAML validation
policy checks
connectivity tests
```

---

## 112. Connectivity Test Script

Create tests for:

```text
DNS
TCP
HTTP
HTTPS
database
AWS endpoints
```

Return non-zero status when expected connectivity fails.

---

## 113. Network Regression Tests

Run after changes:

```text
frontend → backend
backend → DB
backend → AWS
DNS
Ingress
```

---

## 114. Negative Security Tests

Verify:

```text
frontend ✕ DB
unauthorized namespace ✕ backend
Pod ✕ arbitrary internet
public internet ✕ DB
```

---

## 115. Positive Security Tests

Verify:

```text
ALB → App
App → DB
App → DNS
App → approved AWS services
```

---

## 116. Production Scenario Documentation

For every intentional failure record:

```text
failure injected
observed symptom
evidence
root cause
mitigation
permanent fix
prevention
```

---

## 117. Project 90 Interview Preparation

Be prepared to explain:

```text
Why private subnets?
Why NAT?
Why VPC endpoints?
Why NetworkPolicy?
Why SG?
Why WAF?
Why ALB?
Why multi-AZ?
How does Pod networking work?
How do you troubleshoot DNS?
How do you troubleshoot 504?
How do you secure egress?
```

---

## 118. Senior-Level Project Expectations

A senior engineer should demonstrate:

```text
architecture
security
automation
troubleshooting
observability
failure handling
cost awareness
operability
```

not merely successful deployment.

---

## 119. Project Progression

Recommended sequence:

```text
01–10  Linux/AWS fundamentals
11–25  AWS networking/security
26–40  EKS fundamentals
41–60  EKS production networking
61–80  hybrid/security/observability
81–90  GameDays/capstone
```

---

## 120. Portfolio Priority Projects

If time is limited, prioritize:

```text
06 AWS VPC
15 SG architecture
25 CloudFront/WAF/ALB
27 EKS networking
32 NetworkPolicy
35 Ingress
41 EKS → RDS
43 EKS → S3
46 EKS troubleshooting
50 Production EKS
68 Centralized inspection
70 Hybrid DNS
79 Network dashboard
90 Production capstone
```

---

## 121. Project Evidence

For portfolio quality capture:

```text
architecture diagrams
Terraform plan
kubectl output
successful tests
failed tests
logs
dashboards
incident timeline
```

Remove sensitive information before publishing.

---

## 122. Production Architecture Review

For each project ask:

```text
What happens if one AZ fails?
What happens if NAT fails?
What happens if DNS fails?
What happens if Pods cannot get IPs?
What happens if the database is unavailable?
What happens if an SG rule is wrong?
What happens if an external API fails?
```

---

## 123. Cost Review

Ask:

```text
Can VPC endpoints reduce NAT traffic?
Is cross-AZ traffic necessary?
Can logging volume be controlled?
Is centralized inspection required for all traffic?
```

---

## 124. Security Review

Ask:

```text
What is public?
What is private?
Who can reach the database?
Who can modify DNS?
Who can modify routes?
Who can modify security groups?
Who can access the cluster?
```

---

## 125. Operations Review

Ask:

```text
How is it monitored?
How is it deployed?
How is it rolled back?
How is it tested?
How is it recovered?
Who owns it?
```

---

## 126. Final Networking Project Roadmap

```text
Foundation
   ↓
AWS VPC
   ↓
AWS Security
   ↓
Load Balancing
   ↓
DNS
   ↓
EKS Networking
   ↓
NetworkPolicy
   ↓
Private AWS Services
   ↓
Hybrid Networking
   ↓
Observability
   ↓
Security
   ↓
GameDays
   ↓
Production Capstone
```

---

## 127. Final Project Goal

The objective is not to collect 90 projects.

The objective is to become capable of taking a production requirement and designing:

```text
secure
highly available
observable
scalable
cost-aware
operable
```

network architecture.

---