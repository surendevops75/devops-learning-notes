# DNS-Troubleshooting

## 1. Purpose

DNS troubleshooting is a core DevOps skill because almost every production application depends on name resolution.

A production request commonly follows:

```text
Application
   ↓
DNS resolver
   ↓
DNS hierarchy
   ↓
IP address
   ↓
TCP/UDP
   ↓
TLS
   ↓
HTTP
```

A DNS problem can therefore appear as:

```text
website unavailable
API timeout
Kubernetes Service unavailable
EKS workload cannot reach AWS service
certificate mismatch
load balancer unreachable
```

The objective of this file is to diagnose DNS problems systematically across:

```text
Linux
AWS
Route 53
Kubernetes
CoreDNS
EKS
Ingress
ALB/NLB
Private DNS
Production environments
```

---

## 2. DNS Is Not Just "Domain to IP"

DNS can provide:

```text
A
AAAA
CNAME
MX
TXT
NS
SOA
SRV
PTR
CAA
```

and other record types.

DNS can also influence:

```text
service discovery
load balancing
failover
routing
email
certificate issuance
Kubernetes service discovery
AWS private service discovery
```

---

## 3. First Troubleshooting Question

Ask:

```text
What hostname is failing?
```

Then identify:

```text
source
resolver
record type
expected answer
actual answer
```

---

## 4. Production DNS Debugging Model

Use:

```text
Client
 ↓
Local resolver configuration
 ↓
Recursive resolver
 ↓
Root
 ↓
TLD
 ↓
Authoritative DNS
 ↓
Record
```

For internal DNS:

```text
Client
 ↓
Corporate/VPC/Kubernetes resolver
 ↓
Private zone
 ↓
Internal record
```

---

## 5. DNS Failure Categories

Common failures:

```text
NXDOMAIN
SERVFAIL
REFUSED
timeout
wrong IP
stale answer
missing record
wrong zone
split-horizon mismatch
delegation failure
DNSSEC failure
resolver failure
CoreDNS failure
Route 53 configuration error
```

---

## 6. The Core DNS Commands

Know these well:

```bash
dig
nslookup
host
getent
resolvectl
```

For packet-level DNS troubleshooting:

```bash
tcpdump
```

---

## 7. Basic `dig`

```bash
dig example.com
```

Inspect:

```text
status
flags
QUESTION
ANSWER
AUTHORITY
ADDITIONAL
```

---

## 8. Short Answer

```bash
dig +short example.com
```

Useful for quickly seeing returned IPs.

---

## 9. Query A

```bash
dig example.com A
```

---

## 10. Query AAAA

```bash
dig example.com AAAA
```

---

## 11. Query CNAME

```bash
dig example.com CNAME
```

---

## 12. Query NS

```bash
dig example.com NS
```

---

## 13. Query SOA

```bash
dig example.com SOA
```

---

## 14. Query MX

```bash
dig example.com MX
```

---

## 15. Query TXT

```bash
dig example.com TXT
```

---

## 16. Query CAA

```bash
dig example.com CAA
```

---

## 17. Reverse DNS

```bash
dig -x 192.0.2.10
```

---

## 18. Query Specific Resolver

```bash
dig @8.8.8.8 example.com
```

---

## 19. Query Cloudflare Resolver

```bash
dig @1.1.1.1 example.com
```

---

## 20. Compare Resolvers

```bash
dig @8.8.8.8 example.com
dig @1.1.1.1 example.com
dig @<corporate-resolver> example.com
```

If results differ, investigate resolver behavior, caching, split DNS, or propagation.

---

## 21. DNS Trace

```bash
dig +trace example.com
```

This walks the delegation chain from the root.

---

## 22. Trace Purpose

Use `+trace` to investigate:

```text
root delegation
TLD delegation
authoritative nameservers
```

---

## 23. Authoritative Server Query

First find NS:

```bash
dig example.com NS
```

Then query one directly:

```bash
dig @ns1.example-dns.com example.com A
```

---

## 24. Recursive vs Authoritative

A recursive resolver:

```text
finds answers on behalf of clients
```

An authoritative server:

```text
hosts the zone's authoritative data
```

---

## 25. DNS Flags

Typical response flags may include:

```text
qr
aa
rd
ra
ad
```

---

## 26. `AA`

`aa` indicates:

```text
Authoritative Answer
```

when present in a DNS response.

---

## 27. `RD`

`rd` means:

```text
Recursion Desired
```

---

## 28. `RA`

`ra` indicates the server supports recursion.

---

## 29. `AD`

`ad` can indicate authenticated data when DNSSEC validation is involved.

---

## 30. DNS Status Codes

Important codes:

```text
NOERROR
NXDOMAIN
SERVFAIL
REFUSED
FORMERR
NOTIMP
```

---

## 31. NOERROR

The DNS query completed successfully.

It does not necessarily mean the requested record exists.

---

## 32. NXDOMAIN

The queried DNS name does not exist according to the responding DNS system.

---

## 33. NODATA

A name may exist while the requested record type does not.

Example:

```text
example.com exists
but AAAA record is absent
```

This is different from NXDOMAIN.

---

## 34. SERVFAIL

The resolver could not successfully complete the query.

Possible causes:

```text
DNSSEC validation
authoritative failure
delegation issue
upstream timeout
resolver problem
```

---

## 35. REFUSED

The DNS server refused the query.

Possible reasons:

```text
policy
ACL
recursion disabled
query restrictions
```

---

## 36. DNS Timeout

A timeout may indicate:

```text
network path
firewall
resolver failure
UDP filtering
TCP fallback issue
```

---

## 37. UDP vs TCP DNS

DNS commonly uses:

```text
UDP/53
```

and can use:

```text
TCP/53
```

for larger responses, zone transfers, or fallback scenarios.

---

## 38. Test DNS Over TCP

```bash
dig +tcp example.com
```

---

## 39. Compare UDP and TCP

```bash
dig example.com
dig +tcp example.com
```

If UDP fails but TCP works, investigate:

```text
UDP/53 filtering
fragmentation
MTU
middleboxes
```

---

## 40. DNS Port Test

```bash
nc -vz <dns-server> 53
```

TCP testing alone does not prove UDP DNS works.

---

## 41. UDP DNS Caveat

Do not interpret:

```bash
nc -vz <dns-server> 53
```

as a complete UDP DNS test.

Use `dig` for DNS protocol testing.

---

## 42. Check `/etc/resolv.conf`

```bash
cat /etc/resolv.conf
```

Look for:

```text
nameserver
search
options
```

---

## 43. Multiple Nameservers

Example:

```text
nameserver 10.0.0.2
nameserver 10.0.0.3
```

The resolver behavior depends on the OS resolver implementation.

---

## 44. Search Domains

Example:

```text
search example.internal
```

A short hostname may be expanded using configured search domains.

---

## 45. Kubernetes `/etc/resolv.conf`

Inside a normal Kubernetes Pod:

```bash
cat /etc/resolv.conf
```

Typical configuration points toward the cluster DNS Service.

---

## 46. `nsswitch.conf`

```bash
cat /etc/nsswitch.conf
```

Inspect:

```text
hosts:
```

because applications may use multiple name sources.

---

## 47. `getent`

```bash
getent hosts example.com
```

This is useful because it follows the system's configured name-service path.

---

## 48. `resolvectl`

On systems using systemd-resolved:

```bash
resolvectl status
```

---

## 49. Resolver Per-Link Information

```bash
resolvectl dns
```

and:

```bash
resolvectl domain
```

can show resolver configuration by interface.

---

## 50. Host vs Application DNS

A successful:

```bash
dig example.com
```

does not guarantee every application resolves the hostname the same way.

Why?

```text
application DNS library
proxy
custom resolver
/etc/hosts
NSS
DNS cache
```

may affect behavior.

---

## 51. `/etc/hosts`

Check:

```bash
cat /etc/hosts
```

A stale hosts-file entry can override expected DNS behavior depending on NSS configuration.

---

## 52. Compare `getent` and `dig`

```bash
getent hosts example.com
dig example.com
```

If they differ, investigate:

```text
NSS
/etc/hosts
resolver configuration
IPv4/IPv6 behavior
```

---

## 53. DNS Cache

Possible cache layers:

```text
browser
application
OS
local resolver
corporate resolver
VPC resolver
CoreDNS
```

---

## 54. TTL

DNS records have a:

```text
TTL
```

which controls caching duration.

---

## 55. TTL Does Not Mean Global Propagation Timer

A DNS change is not simply:

```text
wait exactly TTL
```

because multiple caches and operational behaviors can exist.

---

## 56. Inspect TTL

```bash
dig example.com
```

Look at the answer section.

---

## 57. Cache Age

A recursive resolver may return a remaining TTL that decreases over time.

---

## 58. DNS Propagation Troubleshooting

Compare:

```bash
dig @resolver1 example.com
dig @resolver2 example.com
dig @authoritative-server example.com
```

---

## 59. Authoritative vs Cached Answer

If authoritative data is correct but recursive resolver is stale:

```text
authoritative zone
    ↓
correct

recursive resolver
    ↓
old cached answer
```

Wait for TTL/cache expiry or investigate resolver cache behavior.

---

## 60. Wrong Record

Common mistake:

```text
A record points to old ALB
```

Check:

```bash
dig +short app.example.com
```

---

## 61. Multiple A Records

Example:

```text
203.0.113.10
203.0.113.20
```

Applications may receive multiple answers.

---

## 62. DNS Load Distribution

Multiple records can distribute clients across endpoints, but behavior depends on resolver/client caching and record design.

---

## 63. CNAME Chain

Check:

```bash
dig app.example.com CNAME
```

Then follow the target.

---

## 64. CNAME Chain Debugging

```text
app.example.com
 ↓
CNAME
 ↓
target.example.net
 ↓
A
 ↓
IP
```

---

## 65. CNAME Loop

A configuration such as:

```text
a → b
b → a
```

can cause resolution failure.

---

## 66. Excessive CNAME Chains

Long chains add complexity and can increase resolution latency.

Keep DNS architecture understandable.

---

## 67. CNAME at Apex

Traditional DNS does not permit a normal CNAME at the zone apex because the apex must contain required records such as SOA and NS.

Cloud DNS providers may offer alias/flattening mechanisms.

---

## 68. AWS Route 53 Alias

Route 53 supports alias records for supported AWS resources.

Examples include:

```text
ALB
CloudFront
S3 website endpoints
```

depending on supported configurations.

---

## 69. Route 53 Hosted Zones

Two major concepts:

```text
public hosted zone
private hosted zone
```

---

## 70. Public Hosted Zone

Answers queries from the public DNS system when delegation is correctly configured.

---

## 71. Private Hosted Zone

Used for DNS names resolved inside associated VPCs through Route 53 Resolver.

---

## 72. Split-Horizon DNS

The same hostname can return different answers depending on query location.

Example:

```text
app.example.com

Internet:
public ALB

VPC:
internal ALB
```

---

## 73. Split DNS Debugging

Test from:

```text
Internet
EC2
EKS Pod
corporate network
```

Do not assume one result represents all clients.

---

## 74. Route 53 Resolver

AWS VPC DNS resolution uses Route 53 Resolver infrastructure.

The VPC DNS resolver is commonly reachable through the VPC-provided resolver address.

---

## 75. VPC DNS Attributes

Check:

```text
enableDnsSupport
enableDnsHostnames
```

when diagnosing VPC DNS behavior.

---

## 76. AWS VPC DNS CLI

```bash
aws ec2 describe-vpc-attribute \
  --vpc-id <vpc-id> \
  --attribute enableDnsSupport
```

and:

```bash
aws ec2 describe-vpc-attribute \
  --vpc-id <vpc-id> \
  --attribute enableDnsHostnames
```

---

## 77. Route 53 Hosted Zone List

```bash
aws route53 list-hosted-zones
```

---

## 78. Find Record Sets

```bash
aws route53 list-resource-record-sets \
  --hosted-zone-id <zone-id>
```

---

## 79. Query Route 53 Health Checks

```bash
aws route53 list-health-checks
```

---

## 80. Route 53 Record Debugging

Check:

```text
record name
record type
value
TTL
routing policy
health check
alias
```

---

## 81. Route 53 Routing Policies

Common policies include:

```text
simple
weighted
latency
failover
geolocation
geoproximity
multivalue
```

Use the correct policy for the architecture.

---

## 82. Weighted Routing

Weighted records distribute DNS answers according to configured weights.

---

## 83. Latency Routing

Route 53 latency-based routing chooses among configured regions based on latency measurements.

---

## 84. Failover Routing

Primary/secondary records can be used with health checks.

---

## 85. Health Check Caveat

DNS health status does not automatically mean the application is healthy in every possible user path.

Validate the actual endpoint and monitoring design.

---

## 86. Alias vs CNAME

Alias:

```text
Route 53-specific
supports certain AWS targets
```

CNAME:

```text
standard DNS record
```

---

## 87. ALB DNS Name

An ALB has an AWS-provided DNS name.

Applications commonly use Route 53 alias records to map a friendly hostname to it.

---

## 88. NLB DNS Name

NLBs also have AWS-provided DNS names.

---

## 89. DNS + ALB Debugging

```bash
dig app.example.com
dig app.example.com CNAME
```

Then compare the returned endpoint with the intended load balancer.

---

## 90. DNS + CloudFront

If using CloudFront:

```text
Route 53
 ↓
CloudFront distribution
 ↓
origin
```

Check whether the hostname points to the expected distribution.

---

## 91. DNS + WAF

WAF does not resolve DNS itself for the client.

The request reaches the AWS edge/load-balancing path after DNS resolution.

---

## 92. DNS + TLS

A DNS answer determines where the connection goes.

TLS validates the hostname through certificate/SNI behavior.

Therefore:

```text
correct DNS
+
wrong certificate
```

can still cause HTTPS failure.

---

## 93. Certificate Mismatch

Example:

```text
DNS:
api.example.com → ALB

Certificate:
www.example.com
```

The TCP path can work while TLS fails.

---

## 94. Debug Certificate After DNS

```bash
openssl s_client \
  -connect api.example.com:443 \
  -servername api.example.com
```

---

## 95. DNS and SNI

Use the actual hostname with:

```text
-servername
```

when testing shared TLS endpoints.

---

## 96. Kubernetes DNS Architecture

Typical:

```text
Pod
 ↓
CoreDNS Service
 ↓
CoreDNS
 ↓
cluster.local records
 ↓
upstream DNS for external names
```

---

## 97. CoreDNS

CoreDNS is commonly used as the Kubernetes cluster DNS server.

---

## 98. Check CoreDNS Pods

```bash
kubectl get pods -n kube-system \
  -l k8s-app=kube-dns
```

Labels may differ by distribution/version.

---

## 99. Check CoreDNS Service

```bash
kubectl get svc -n kube-system kube-dns
```

---

## 100. CoreDNS Endpoints

```bash
kubectl get endpointslice \
  -n kube-system \
  -l k8s-app=kube-dns
```

---

## 101. CoreDNS Logs

```bash
kubectl logs -n kube-system \
  -l k8s-app=kube-dns
```

---

## 102. CoreDNS Configuration

```bash
kubectl get configmap coredns \
  -n kube-system \
  -o yaml
```

---

## 103. CoreDNS Corefile

Look for plugins such as:

```text
kubernetes
forward
cache
loop
reload
loadbalance
```

depending on configuration.

---

## 104. `kubernetes` Plugin

Provides Kubernetes DNS records such as:

```text
Service
Pod
```

according to configured zones and policies.

---

## 105. `forward` Plugin

Forwards queries that CoreDNS does not answer locally to upstream resolvers.

---

## 106. `cache` Plugin

Caches DNS responses and can reduce upstream traffic.

---

## 107. CoreDNS Loop Detection

The `loop` plugin can detect forwarding loops.

A DNS forwarding loop can cause:

```text
SERVFAIL
high CPU
repeated queries
```

---

## 108. Kubernetes DNS Test

Launch an approved DNS utility image:

```bash
kubectl run dns-test \
  --image=registry.k8s.io/e2e-test-images/dnsutils:1.3 \
  --rm -it \
  --restart=Never \
  -- sh
```

---

## 109. Test Kubernetes DNS

Inside the Pod:

```bash
nslookup kubernetes.default
```

---

## 110. Test Service DNS

```bash
nslookup frontend.roboshop.svc.cluster.local
```

---

## 111. Test External DNS

```bash
nslookup example.com
```

---

## 112. Compare Internal and External

If:

```text
kubernetes.default works
example.com fails
```

focus on CoreDNS upstream forwarding.

---

## 113. If Both Fail

If both:

```text
cluster DNS
external DNS
```

fail, investigate:

```text
CoreDNS Pod
Service
EndpointSlice
NetworkPolicy
CNI
node networking
```

---

## 114. If External Works but Service Fails

Focus on:

```text
Kubernetes plugin
Service
EndpointSlice
namespace/name
```

---

## 115. Kubernetes Service FQDN

Typical format:

```text
<service>.<namespace>.svc.cluster.local
```

---

## 116. Same Namespace Short Name

A Pod in the same namespace may resolve:

```text
frontend
```

using its search domains.

---

## 117. Cross Namespace

Use:

```text
frontend.roboshop
```

or full FQDN:

```text
frontend.roboshop.svc.cluster.local
```

---

## 118. Wrong Namespace

A common application error:

```text
frontend.default
```

when the Service actually exists in:

```text
roboshop
```

---

## 119. Headless Service

A Service with:

```yaml
clusterIP: None
```

is headless.

DNS can return Pod endpoints rather than one virtual Service IP.

---

## 120. Headless Debugging

```bash
kubectl get svc <name> -o yaml
dig <service>.<namespace>.svc.cluster.local
```

---

## 121. StatefulSet DNS

StatefulSets commonly use stable DNS identities with a governing headless Service.

---

## 122. Pod DNS

Kubernetes Pod DNS behavior depends on:

```text
hostname
subdomain
DNS policy
cluster configuration
```

---

## 123. `dnsPolicy`

Inspect:

```bash
kubectl get pod <pod> \
  -n <namespace> \
  -o jsonpath='{.spec.dnsPolicy}'
```

---

## 124. `ClusterFirst`

The normal default for many Pods.

Cluster DNS is preferred for cluster-local names.

---

## 125. `ClusterFirstWithHostNet`

Relevant for Pods using host networking when cluster DNS behavior is required.

---

## 126. `Default`

Uses the node's DNS configuration.

---

## 127. `None`

Allows custom Pod DNS configuration through:

```yaml
dnsConfig:
```

---

## 128. Custom DNS Config

Inspect:

```bash
kubectl get pod <pod> -o yaml
```

Look for:

```text
dnsPolicy
dnsConfig
```

---

## 129. Pod `/etc/hosts`

Kubernetes may generate host entries.

Inspect:

```bash
cat /etc/hosts
```

---

## 130. Pod `/etc/resolv.conf`

Inspect:

```bash
cat /etc/resolv.conf
```

---

## 131. Search Domain Debugging

If:

```bash
nslookup frontend
```

works but:

```bash
curl frontend.example.com
```

fails, confirm the intended DNS suffix.

---

## 132. DNS Search Expansion

A short name can produce multiple queries.

Packet capture can reveal the exact names being queried.

---

## 133. DNS Packet Capture

```bash
tcpdump -ni any port 53
```

Use narrow interfaces/filters in production.

---

## 134. Capture DNS From Pod

If authorized:

```bash
tcpdump -ni eth0 port 53
```

inside an appropriate network namespace.

---

## 135. DNS Query/Response

Look for:

```text
query leaves client
response returns
```

---

## 136. DNS Response Missing

If query leaves but no response returns:

```text
resolver
route
firewall
packet loss
```

are candidates.

---

## 137. DNS Response Arrives but Application Fails

Then DNS may not be the problem.

Move to:

```text
route
TCP
TLS
HTTP
```

---

## 138. DNS Over UDP Fragmentation

Large DNS responses can be affected by fragmentation and network devices.

Use:

```bash
dig +tcp
```

to compare.

---

## 139. EDNS

Modern DNS uses EDNS mechanisms to support larger UDP payloads and additional features.

---

## 140. EDNS Debugging

`dig` output can expose EDNS information.

---

## 141. DNSSEC

DNSSEC adds:

```text
authentication
integrity
```

to DNS data.

---

## 142. DNSSEC Failure

A resolver validating DNSSEC may return:

```text
SERVFAIL
```

when signatures/chain validation fail.

---

## 143. DNSSEC Test

Use:

```bash
dig +dnssec example.com
```

---

## 144. AD Flag

When validation succeeds, a validating resolver may return:

```text
ad
```

in the response flags.

---

## 145. DNSSEC Troubleshooting

Check:

```text
DS
DNSKEY
RRSIG
chain of trust
expiration
```

---

## 146. Delegation

A parent zone delegates a child zone using NS records.

---

## 147. Delegation Debugging

```bash
dig example.com NS
dig +trace example.com
```

---

## 148. Broken Delegation

Possible symptoms:

```text
NXDOMAIN
SERVFAIL
inconsistent answers
```

---

## 149. Glue Records

Glue can be required when authoritative nameservers are within the delegated zone.

---

## 150. Nameserver Reachability

After discovering authoritative NS records, test their A/AAAA addresses and DNS responses.

---

## 151. Parent vs Child NS

The parent delegation and child zone's authoritative NS data should be consistent.

---

## 152. SOA Record

SOA contains important zone metadata such as:

```text
primary/master information
serial
refresh
retry
expire
negative TTL
```

---

## 153. Zone Serial

In traditional DNS zone operations, the serial helps secondary servers determine whether zone data changed.

---

## 154. DNS Propagation Investigation

Always query:

```text
authoritative
multiple recursive resolvers
```

instead of relying on one public checker.

---

## 155. Negative Caching

NXDOMAIN and negative responses can also be cached according to DNS rules/TTL.

---

## 156. Negative Cache Problem

A newly created record may still appear absent to a resolver that cached a previous negative answer.

---

## 157. DNS Record Creation

After creating:

```text
api.example.com A
```

verify authoritative data first.

---

## 158. Route 53 Record Verification

```bash
aws route53 list-resource-record-sets \
  --hosted-zone-id <zone-id>
```

---

## 159. Route 53 Resolver Rules

AWS can use Resolver rules for forwarding selected domains to specified DNS servers.

---

## 160. Resolver Rule Debugging

Inspect:

```text
domain
target IPs
associated VPCs
endpoint
```

---

## 161. Inbound Resolver Endpoint

Allows DNS queries into a VPC from connected networks where configured.

---

## 162. Outbound Resolver Endpoint

Allows forwarding selected DNS queries from a VPC to external/on-prem DNS servers.

---

## 163. Hybrid DNS

Typical architecture:

```text
On-prem
 ↓
VPN/DX
 ↓
Route 53 Resolver
 ↓
AWS VPC
```

---

## 164. Hybrid DNS Failure

Symptoms:

```text
AWS can resolve public names
AWS cannot resolve internal corporate names
```

Investigate:

```text
Resolver rules
outbound endpoint
routing
Security Groups
on-prem DNS
```

---

## 165. Reverse Hybrid DNS

Corporate clients may need to resolve AWS private names through an inbound Resolver endpoint.

---

## 166. Private Hosted Zone Association

A private hosted zone must be associated with the intended VPC(s).

---

## 167. Private Zone Debugging

Confirm:

```text
zone exists
zone is private
VPC association exists
record exists
client is in associated network
```

---

## 168. Same Name Public and Private

AWS can have public and private hosted zones with the same domain name.

The query source determines which namespace is applicable.

---

## 169. Split-Horizon AWS Example

```text
api.example.com

Public:
ALB public endpoint

Private:
internal ALB
```

---

## 170. EKS + Private DNS

EKS workloads may resolve:

```text
internal services
private APIs
RDS endpoints
AWS private endpoints
```

through VPC DNS.

---

## 171. VPC Endpoint DNS

Interface VPC endpoints can provide private DNS names when configured.

---

## 172. Interface Endpoint Debugging

Check:

```text
Private DNS enabled
endpoint exists
endpoint ENIs
Security Groups
VPC DNS support
```

---

## 173. AWS Service Private DNS

When private DNS is enabled for supported interface endpoints, applications can continue using standard AWS service hostnames while resolving to private endpoint addresses.

---

## 174. PrivateLink DNS

PrivateLink architectures often depend on DNS to steer consumers to private endpoint addresses.

---

## 175. EKS API Private Endpoint

If an EKS cluster uses private API endpoint access, clients must have network connectivity to the private endpoint.

DNS resolution alone does not prove reachability.

---

## 176. EKS DNS Debugging

```bash
dig <eks-endpoint>
curl -v https://<eks-endpoint>
```

Then separate:

```text
DNS
TCP
TLS
authentication
authorization
```

---

## 177. CoreDNS Upstream Failure

Symptoms:

```text
cluster.local works
external names fail
```

Check:

```text
CoreDNS forward configuration
upstream resolver
VPC DNS
network path
```

---

## 178. CoreDNS CrashLoop

```bash
kubectl get pods -n kube-system
kubectl describe pod <coredns-pod> -n kube-system
kubectl logs <coredns-pod> -n kube-system
```

---

## 179. CoreDNS Resource Pressure

High DNS query volume can expose:

```text
CPU pressure
memory pressure
insufficient replicas
```

Check resource requests/limits and metrics.

---

## 180. CoreDNS Scaling

Production clusters should size CoreDNS according to:

```text
Pod count
query volume
NodeLocal DNSCache
workload behavior
```

---

## 181. NodeLocal DNSCache

NodeLocal DNSCache can reduce DNS latency and conntrack pressure in Kubernetes environments.

---

## 182. NodeLocal DNSCache Debugging

Determine whether the cluster uses it before assuming Pods directly query CoreDNS Service IP.

---

## 183. DNS Cache Architecture

Possible:

```text
Pod
 ↓
NodeLocal DNSCache
 ↓
CoreDNS
 ↓
upstream
```

---

## 184. DNS Amplification

Uncontrolled high-volume DNS queries can create load.

Investigate:

```text
application retries
short TTLs
service discovery behavior
misconfigured clients
```

---

## 185. DNS Query Storm

Symptoms:

```text
CoreDNS CPU high
DNS latency high
SERVFAIL increases
```

---

## 186. Query Storm Investigation

Check:

```text
CoreDNS metrics
logs
application query patterns
cache hit ratio
```

---

## 187. Application DNS Retry Problem

An application may retry DNS queries aggressively when an upstream service fails.

This can amplify the incident.

---

## 188. DNS Metrics

Useful metrics include:

```text
request rate
latency
response codes
cache hits
cache misses
upstream failures
```

---

## 189. Prometheus + CoreDNS

CoreDNS commonly exposes metrics when configured.

Use Prometheus to monitor:

```text
DNS query rate
latency
errors
```

---

## 190. Grafana DNS Dashboard

A production dashboard can show:

```text
queries/sec
SERVFAIL
NXDOMAIN
latency
CoreDNS CPU/memory
```

---

## 191. DNS Logs

Logs can reveal:

```text
forwarding errors
timeouts
plugin errors
```

But metrics are usually better for high-volume trends.

---

## 192. DNS Monitoring

Alert on:

```text
SERVFAIL rate
CoreDNS unavailable replicas
DNS latency
upstream failure
```

---

## 193. Avoid Alerting Only on NXDOMAIN

NXDOMAIN may be normal for some applications.

Alert based on:

```text
baseline
critical names
error rate
user impact
```

---

## 194. DNS Availability SLO

A DNS SLO can measure:

```text
successful resolution
latency
critical record availability
```

---

## 195. DNS Incident: Website Down

Start:

```bash
dig +short www.example.com
```

If no answer:

```text
DNS path
```

If correct answer:

```text
move to TCP
```

---

## 196. DNS Incident: API Timeout

Test:

```bash
dig +short api.example.com
nc -vz api.example.com 443
curl -v https://api.example.com
```

---

## 197. DNS Incident: Wrong IP

Check:

```bash
dig api.example.com A
dig @authoritative-server api.example.com A
```

---

## 198. DNS Incident: Only One Office Affected

Suspect:

```text
local resolver
split DNS
network path
cached response
```

Compare with another resolver/location.

---

## 199. DNS Incident: Only EKS Affected

Suspect:

```text
CoreDNS
NodeLocal DNSCache
NetworkPolicy
CNI
VPC DNS
```

---

## 200. DNS Incident: Only AWS Private Names Affected

Suspect:

```text
Private Hosted Zone
VPC association
Resolver rules
VPC DNS attributes
```

---

## 201. DNS Incident: External Names Fail in EKS

Test:

```bash
nslookup kubernetes.default
nslookup example.com
```

If cluster names work and external names fail:

```text
CoreDNS upstream/VPC resolver path
```

---

## 202. DNS Incident: Internal Name Fails in EKS

Check:

```text
private hosted zone
Resolver rule
VPC association
```

---

## 203. DNS Incident: Certificate Error After DNS Change

Check:

```text
new DNS target
TLS certificate
SNI
ALB listener
```

---

## 204. DNS Incident: DNS Works on Laptop but Not Pod

Compare:

```bash
dig
cat /etc/resolv.conf
```

from both contexts.

Then inspect:

```text
CoreDNS
VPC DNS
NetworkPolicy
```

---

## 205. DNS Incident: DNS Works on Node but Not Pod

Suspect:

```text
Pod DNS policy
CoreDNS Service
CNI
NetworkPolicy
NodeLocal DNSCache
```

---

## 206. DNS Incident: Pod DNS Intermittent

Investigate:

```text
CoreDNS replicas
NodeLocal DNSCache
packet loss
conntrack
resource pressure
upstream resolver
```

---

## 207. DNS Incident: SERVFAIL

Run:

```bash
dig example.com
dig +trace example.com
dig @authoritative-server example.com
```

Then investigate the exact failing delegation/upstream.

---

## 208. DNS Incident: NXDOMAIN

Confirm:

```bash
dig example.com
dig @authoritative-server example.com
```

If authoritative also returns NXDOMAIN:

```text
record/zone configuration
```

If authoritative is correct:

```text
cache/delegation/resolver
```

---

## 209. DNS Incident: REFUSED

Identify the responding server:

```bash
dig example.com
```

Then investigate its policy/ACL.

---

## 210. DNS Incident: UDP Fails, TCP Works

Run:

```bash
dig example.com
dig +tcp example.com
```

Investigate:

```text
UDP fragmentation
firewall
MTU
network device
```

---

## 211. DNS Incident: TCP Fails, UDP Works

Investigate:

```text
TCP/53 filtering
resolver TCP listener
firewall
middlebox
```

---

## 212. DNS Incident: High Latency

Use:

```bash
dig example.com
```

Compare multiple resolvers and measure repeatedly.

---

## 213. Repeated DNS Testing

Example:

```bash
for i in {1..10}; do
  dig +stats +short example.com
done
```

Use controlled test volume.

---

## 214. DNS Query Timing

`dig` output includes query time.

Compare:

```text
local resolver
public resolver
authoritative server
```

---

## 215. DNS Debugging With `+stats`

```bash
dig +stats example.com
```

Useful for response time and server information.

---

## 216. DNS Debugging With `+comments`

```bash
dig +comments example.com
```

---

## 217. Full DNS Output

```bash
dig +noall +answer example.com
```

---

## 218. Answer Only

```bash
dig +noall +answer example.com
```

---

## 219. DNS Server Selection

```bash
dig @10.0.0.2 example.com
```

This bypasses the default resolver configuration.

---

## 220. Debugging Wrong Resolver

If:

```bash
dig example.com
```

fails but:

```bash
dig @8.8.8.8 example.com
```

works, investigate the configured resolver rather than immediately changing application configuration.

---

## 221. Corporate DNS

Corporate environments may use:

```text
internal resolver
conditional forwarding
split-horizon
security filtering
```

Public DNS may not contain private records.

---

## 222. Never Replace Corporate DNS Blindly

Using public DNS can break:

```text
internal names
security policy
private domains
service discovery
```

---

## 223. AWS Resolver and Corporate DNS

Hybrid architectures often require conditional forwarding between:

```text
AWS
on-prem
```

---

## 224. Conditional Forwarding

Example:

```text
corp.example.com
→ corporate DNS

aws.example.com
→ Route 53
```

---

## 225. Conditional Forwarding Failure

Possible:

```text
wrong rule
wrong target
route unavailable
SG blocked
endpoint unavailable
```

---

## 226. DNS Security Groups

For Route 53 Resolver endpoints, verify the associated Security Groups allow expected DNS traffic.

---

## 227. DNS NACLs

If custom NACLs are involved, ensure both request and response traffic are permitted.

Remember:

```text
NACLs are stateless.
```

---

## 228. Resolver Endpoint IPs

Inspect the Resolver endpoint ENIs and IP addresses when debugging hybrid DNS.

---

## 229. DNS and VPN

A VPN may carry:

```text
DNS queries
```

but routing and firewall policy must permit them.

---

## 230. DNS and Direct Connect

Direct Connect can provide the network path for hybrid DNS but does not automatically configure DNS forwarding.

---

## 231. DNS and Transit Gateway

Transit Gateway provides connectivity between networks, but DNS forwarding/Resolver architecture still needs explicit configuration.

---

## 232. DNS and Service Discovery

AWS Cloud Map or other service discovery systems can provide internal names.

Identify the authoritative source before debugging.

---

## 233. DNS Ownership

For every critical hostname, know:

```text
owner
zone
record
resolver
application
```

---

## 234. Production DNS Inventory

Maintain:

```text
domain
record type
target
TTL
owner
environment
routing policy
health check
```

---

## 235. Environment Separation

Prefer clear naming:

```text
dev
stage
prod
```

and controlled DNS delegation/records.

---

## 236. Accidental Production DNS Change

Protect production DNS with:

```text
IAM
change review
Terraform
version control
audit logs
```

---

## 237. Terraform DNS

Infrastructure as Code helps track:

```text
record changes
routing policies
zone associations
```

---

## 238. Terraform DNS Drift

Compare:

```text
desired state
actual Route 53 state
```

when troubleshooting unexpected records.

---

## 239. Route 53 Change History

AWS APIs provide change information for Route 53 changes.

Use AWS audit mechanisms such as CloudTrail for broader change attribution.

---

## 240. CloudTrail

CloudTrail can help identify who/what changed DNS resources when supported.

---

## 241. DNS Change Correlation

Correlate:

```text
DNS change time
incident time
TTL
resolver cache
application behavior
```

---

## 242. DNS Rollback

Rollback should restore:

```text
previous record
previous routing policy
previous TTL
```

through the approved IaC/change process.

---

## 243. TTL Planning

Use shorter TTLs when rapid changes are expected, but do not choose extremely low TTLs without considering:

```text
query volume
resolver load
cost
stability
```

---

## 244. TTL During Migration

A common migration approach:

```text
lower TTL before change
change record
monitor
raise TTL after stabilization
```

Follow your organization's change-management process.

---

## 245. Blue/Green DNS

DNS can support blue/green architectures using:

```text
weighted routing
```

or other mechanisms.

---

## 246. DNS Cutover Risk

Cached answers mean not every client changes simultaneously.

Plan overlap between old and new infrastructure.

---

## 247. DNS Failover

Test both:

```text
primary healthy
primary unhealthy
```

before relying on DNS failover in production.

---

## 248. Health Check Target

Ensure Route 53 health checks test an endpoint that actually represents application availability.

---

## 249. DNS Failover Limitation

DNS failover is not instantaneous connection failover.

Existing connections continue until they naturally close or fail.

---

## 250. DNS-Based Load Balancing

DNS distributes answers; it does not replace application/load-balancer health behavior.

---

## 251. Kubernetes External DNS

Tools such as ExternalDNS can automate DNS records from Kubernetes resources.

---

## 252. ExternalDNS Debugging

Inspect:

```text
controller logs
IAM permissions
annotations
hostnames
target
ownership
```

---

## 253. ExternalDNS Logs

Depending on deployment:

```bash
kubectl logs \
  -n external-dns \
  deployment/external-dns
```

---

## 254. ExternalDNS Ownership

Ownership mechanisms help prevent multiple controllers from conflicting over DNS records.

---

## 255. ExternalDNS IAM

In AWS, ExternalDNS needs appropriate Route 53 permissions.

A missing IAM permission can look like a DNS synchronization failure.

---

## 256. ExternalDNS Annotation

Inspect resource annotations:

```bash
kubectl get ingress <name> -n <namespace> -o yaml
```

---

## 257. ExternalDNS + Ingress

Typical flow:

```text
Ingress
 ↓
ExternalDNS
 ↓
Route 53
 ↓
ALB
```

---

## 258. ExternalDNS Failure

If ALB exists but DNS record is missing:

```text
check ExternalDNS
check IAM
check annotations
check hosted zone
```

---

## 259. ExternalDNS and Private Zones

Ensure the controller is configured for the correct:

```text
zone type
VPC/private hosted zone
domain filter
```

---

## 260. DNS Record Ownership

Avoid two systems independently managing the same production record.

---

## 261. DNS Automation Safety

Use:

```text
domain filters
TXT ownership
IAM least privilege
separate environments
```

---

## 262. Kubernetes DNS and NetworkPolicy

A restrictive egress NetworkPolicy can block DNS.

Typical DNS destination:

```text
CoreDNS Service
```

and port:

```text
UDP/53
TCP/53
```

---

## 263. DNS Egress Policy

When writing egress policies, explicitly allow DNS traffic to the intended DNS endpoint.

---

## 264. NetworkPolicy DNS Example

Conceptually:

```yaml
egress:
  - to:
      - namespaceSelector:
          matchLabels:
            kubernetes.io/metadata.name: kube-system
    ports:
      - protocol: UDP
        port: 53
      - protocol: TCP
        port: 53
```

Exact selectors must match your cluster's DNS Pod labels/namespaces and network plugin behavior.

---

## 265. Why Allow TCP/53?

DNS can fall back to TCP.

Allowing only UDP may create intermittent or size-dependent failures.

---

## 266. DNS Policy Debugging

If normal queries work but large responses fail:

```text
check TCP/53
MTU
NetworkPolicy
```

---

## 267. CoreDNS Service IP

Get:

```bash
kubectl get svc kube-dns \
  -n kube-system
```

Use the actual ClusterIP in diagnostics where appropriate.

---

## 268. DNS Endpoint Check

```bash
kubectl get endpointslice \
  -n kube-system \
  -l k8s-app=kube-dns
```

---

## 269. CoreDNS Service Failure

If Service exists but no EndpointSlices:

```text
CoreDNS Pods
labels
readiness
```

may be wrong.

---

## 270. CoreDNS Readiness

```bash
kubectl get pods -n kube-system \
  -l k8s-app=kube-dns
```

Check:

```text
READY
STATUS
RESTARTS
```

---

## 271. CoreDNS Crash Investigation

```bash
kubectl describe pod <coredns-pod> -n kube-system
kubectl logs <coredns-pod> -n kube-system --previous
```

---

## 272. CoreDNS Configuration Rollout

After changing CoreDNS configuration, verify:

```bash
kubectl rollout status \
  deployment/coredns \
  -n kube-system
```

---

## 273. CoreDNS Replica Availability

```bash
kubectl get deployment coredns \
  -n kube-system
```

Ensure desired replicas are available.

---

## 274. DNS During Node Failure

If DNS is concentrated on failed nodes, CoreDNS scheduling/replica design matters.

---

## 275. Pod Anti-Affinity

Production DNS components should be distributed appropriately across nodes/AZs where supported.

---

## 276. DNS Capacity

Monitor:

```text
CPU
memory
QPS
latency
errors
```

---

## 277. DNS Query Amplification From Applications

Bad pattern:

```text
application repeatedly resolves same hostname
```

Better:

```text
appropriate caching
connection reuse
sensible retry behavior
```

---

## 278. DNS Retry Storm

Avoid aggressive retries such as:

```text
retry immediately forever
```

Use bounded backoff.

---

## 279. DNS Timeout Configuration

Application resolver timeouts should be appropriate for the workload.

Do not blindly increase timeouts because they can increase resource consumption.

---

## 280. Java DNS Caching

Some runtimes cache DNS results independently.

When debugging Java applications, inspect JVM DNS caching behavior.

---

## 281. Go DNS Behavior

Go applications may use different resolver paths depending on build/runtime configuration.

When behavior differs from `dig`, inspect application/runtime DNS behavior.

---

## 282. Python DNS Behavior

Python libraries can perform DNS resolution through the OS or custom libraries.

Do not assume `dig` exactly reproduces application behavior.

---

## 283. Browser DNS

Browsers can use:

```text
OS resolver
DNS-over-HTTPS
DNS cache
```

depending on configuration.

---

## 284. DNS-over-HTTPS

DoH can bypass the system resolver path from the perspective of a browser/application.

This matters when browser results differ from `dig`.

---

## 285. DNS-over-TLS

Some clients/resolvers use encrypted DNS transport.

Check architecture before assuming UDP/53 is the only DNS path.

---

## 286. Production Debugging Principle

Always determine:

```text
Which component actually performed the DNS lookup?
```

---

## 287. Proxy DNS Behavior

Depending on proxy configuration, hostname resolution may occur:

```text
client side
```

or:

```text
proxy side
```

This can change observed behavior.

---

## 288. SOCKS Proxy DNS

Some SOCKS configurations support remote DNS resolution.

Be aware when interpreting client-side DNS tests.

---

## 289. DNS and HTTP Proxy

A successful:

```bash
dig
```

does not prove the proxy can resolve the target.

---

## 290. Kubernetes Proxy Variables

Check:

```bash
env | grep -i proxy
```

inside the Pod.

---

## 291. `NO_PROXY`

Include appropriate internal domains/IPs according to architecture, for example:

```text
.cluster.local
.svc
```

where required by the application's proxy behavior.

---

## 292. Do Not Copy NO_PROXY Blindly

A giant `NO_PROXY` list can become stale and may create unexpected direct connections.

Manage it deliberately.

---

## 293. DNS and Service Mesh

Service meshes may alter traffic using sidecars or node-level dataplanes.

DNS resolution can still be performed by the application while traffic is intercepted later.

---

## 294. Service Mesh Debugging

Separate:

```text
DNS resolution
proxy interception
service discovery
TLS origination
application request
```

---

## 295. DNS and mTLS

DNS may correctly resolve while service-mesh mTLS fails.

Do not label every connection failure as DNS.

---

## 296. DNS + Istio Example

Possible path:

```text
Application
 ↓
DNS
 ↓
Envoy interception
 ↓
upstream
```

Investigate each layer independently.

---

## 297. DNS and Jaeger

Tracing can help identify whether DNS/connect/TLS/application latency is contributing, depending on instrumentation.

---

## 298. DNS and OpenTelemetry

Application telemetry can correlate DNS/connect latency with request latency when the instrumentation captures those phases.

---

## 299. DNS Observability

Use:

```text
logs
metrics
traces
packet captures
```

together.

---

## 300. DNS Alert Example

Alert when:

```text
CoreDNS availability < desired
```

or:

```text
SERVFAIL rate exceeds baseline
```

rather than on every individual failure.

---

## 301. Production DNS Dashboard

Recommended panels:

```text
DNS QPS
DNS latency
SERVFAIL
NXDOMAIN
CoreDNS CPU
CoreDNS memory
CoreDNS replicas
upstream errors
cache performance
```

---

## 302. DNS SLO Example

Define:

```text
99.9% of critical DNS queries resolve successfully
```

and:

```text
p95 DNS latency below agreed threshold
```

Exact targets depend on business requirements.

---

## 303. Critical DNS Names

Identify:

```text
public website
API
authentication
payments
databases
critical external dependencies
```

---

## 304. Synthetic DNS Monitoring

Periodically resolve critical names from:

```text
multiple regions
VPCs
clusters
```

to detect split-horizon or regional problems.

---

## 305. Synthetic HTTP Monitoring

DNS-only monitoring is insufficient.

Also test:

```text
DNS
TCP
TLS
HTTP
```

for critical endpoints.

---

## 306. DNS Change Monitoring

Track:

```text
record value
TTL
routing policy
health check
```

changes.

---

## 307. DNS Security

Protect DNS with:

```text
least-privilege IAM
MFA
IaC
change review
audit logging
private zones
```

---

## 308. Route 53 IAM

Use narrowly scoped permissions for automation.

Avoid unrestricted:

```text
route53:*
```

where possible.

---

## 309. DNSSEC for Public Domains

Consider DNSSEC where appropriate to protect DNS integrity.

---

## 310. DNS Cache Poisoning

Modern validating resolvers and DNSSEC can mitigate some DNS integrity risks.

Operational security still matters.

---

## 311. DNS Data Leakage

Private/internal names should not unintentionally be exposed through public DNS.

---

## 312. Split-Horizon Security

Ensure private records remain accessible only through intended private resolution paths.

---

## 313. Route 53 Private Zone Security

Control:

```text
VPC associations
IAM
Resolver rules
```

---

## 314. DNS Exfiltration

Unusual high-volume queries to external domains can be a security signal.

Monitor according to security policy.

---

## 315. DNS Logging

AWS Route 53 Resolver query logging can help investigate DNS activity where enabled.

---

## 316. Resolver Query Logs

Use query logging to identify:

```text
who queried
what name
when
response
```

according to the logging configuration and available fields.

---

## 317. EKS DNS Query Logging

CoreDNS logs/metrics and VPC Resolver query logs provide different perspectives.

Use both when diagnosing hybrid problems.

---

## 318. DNS Incident Evidence

Capture:

```text
query
resolver IP
timestamp
record type
status
answer
TTL
source network
```

---

## 319. DNS Runbook

```text
1. Identify hostname.
2. Identify source.
3. Identify expected record.
4. Query default resolver.
5. Query authoritative server.
6. Compare multiple resolvers.
7. Check delegation.
8. Check TTL/cache.
9. Check private/public zone.
10. Check application behavior.
```

---

## 320. Kubernetes DNS Runbook

```text
1. Check Pod /etc/resolv.conf.
2. Resolve kubernetes.default.
3. Resolve target Service.
4. Check CoreDNS Pods.
5. Check kube-dns Service.
6. Check EndpointSlices.
7. Check CoreDNS logs.
8. Check CoreDNS Corefile.
9. Check NetworkPolicy.
10. Check CNI/network path.
```

---

## 321. EKS DNS Runbook

```text
1. Confirm VPC DNS support.
2. Confirm Pod DNS configuration.
3. Check CoreDNS.
4. Check NodeLocal DNSCache if used.
5. Check VPC Resolver path.
6. Check private hosted zones.
7. Check Resolver rules.
8. Check Security Groups/NACLs.
9. Check CNI.
10. Verify from the affected Pod.
```

---

## 322. Public DNS Runbook

```text
1. Query public resolver.
2. Query authoritative NS.
3. Query parent delegation.
4. Check A/AAAA/CNAME.
5. Check TTL.
6. Check DNSSEC.
7. Check Route 53 record.
8. Check ALB/CloudFront target.
9. Check TLS.
10. Verify HTTP.
```

---

## 323. DNS Decision Tree

```text
Hostname fails
    |
    v
Does default resolver answer?
  |              |
 NO             YES
  |              |
Check resolver   Is answer correct?
                  |          |
                 NO         YES
                  |          |
           authoritative     Test TCP
           + delegation        |
                               v
                              TLS
                               |
                               v
                              HTTP
```

---

## 324. Resolver Decision Tree

```text
Default resolver fails
        |
        v
dig @public-resolver
        |
        +---- works → local resolver/path
        |
        +---- fails
                 |
                 v
          dig @authoritative
                 |
          +------+------+
          |             |
        works          fails
          |             |
       delegation/     zone/
       resolver        authority
```

---

## 325. Kubernetes DNS Decision Tree

```text
Pod DNS fails
    |
    v
kubernetes.default works?
   |              |
  NO             YES
   |              |
CoreDNS path    external name works?
   |              |          |
   |             NO         YES
   |              |          |
   |          upstream      DNS OK
   |          forwarding
```

---

## 326. Production Example: Public API

Architecture:

```text
api.example.com
      ↓
Route 53
      ↓
ALB
      ↓
Ingress
      ↓
Service
      ↓
Pod
```

Troubleshoot in that order.

---

## 327. Production Example: Internal API

```text
Pod
 ↓
CoreDNS
 ↓
VPC Resolver
 ↓
Private Hosted Zone
 ↓
Internal ALB
```

Each layer can independently fail.

---

## 328. Production Example: Corporate API

```text
EKS Pod
 ↓
CoreDNS
 ↓
Route 53 Resolver rule
 ↓
Outbound Resolver Endpoint
 ↓
VPN/DX
 ↓
Corporate DNS
 ↓
Corporate API
```

---

## 329. Corporate DNS Failure

If public DNS works but corporate DNS fails:

```text
CoreDNS
→ Resolver rule
→ endpoint
→ route
→ firewall
→ corporate DNS
```

---

## 330. Private Hosted Zone Failure

If only private names fail:

```text
zone association
record
VPC DNS
Resolver
```

are primary suspects.

---

## 331. Route 53 Record Debugging Example

```bash
aws route53 list-resource-record-sets \
  --hosted-zone-id Z123456789
```

Then:

```bash
dig @<authoritative-server> api.example.com
```

---

## 332. DNS and ALB Target Health

A correct DNS answer does not prove the ALB has healthy targets.

Continue:

```text
DNS
→ ALB
→ target health
```

---

## 333. DNS and NLB

NLB DNS can resolve successfully while backend target health or listener configuration is broken.

---

## 334. DNS and CloudFront

CloudFront DNS can be correct while:

```text
origin
TLS
cache
WAF
```

causes the application request to fail.

---

## 335. DNS and Route 53 Health Checks

If a failover record is returning an unexpected target, inspect:

```text
health check
routing policy
record association
```

---

## 336. Weighted Record Debugging

Check all records in the weighted set and their weights.

---

## 337. Latency Record Debugging

Check:

```text
region
record
health
client resolver
```

---

## 338. Geolocation Debugging

The returned answer can vary based on resolver/client location.

Test from multiple locations.

---

## 339. DNS Testing From Multiple Regions

Useful tools:

```text
regional synthetic probes
EC2 test hosts
approved monitoring platforms
```

---

## 340. DNS Reproduction

Always record:

```text
where the query originated
```

because DNS answers can vary by location and network.

---

## 341. `dig +trace` Limitation

`+trace` is useful for delegation analysis, but it is not always a perfect reproduction of how a normal recursive resolver behaves.

---

## 342. DNS Caching Misconception

Do not assume:

```text
"DNS hasn't propagated"
```

without comparing authoritative and recursive answers.

---

## 343. Correct Diagnosis

Instead say:

```text
"The authoritative record is updated, but resolver X still has the previous cached answer."
```

when evidence supports it.

---

## 344. DNS Debugging With Evidence

Good:

```text
Authoritative:
new IP

Resolver A:
old IP

Resolver B:
new IP
```

This is actionable.

---

## 345. Bad Diagnosis

```text
"DNS is broken."
```

without identifying:

```text
which resolver
which name
which record
which source
```

---

## 346. Production Interview: What Is DNS?

Answer:

```text
DNS is a distributed naming system that maps domain names to records
such as IP addresses and service information. In DevOps it is also
used for service discovery, load distribution, failover and routing.
```

---

## 347. Interview: How Do You Troubleshoot DNS?

Answer:

```text
I first query the default resolver using dig, then compare the result
with authoritative and independent resolvers. I inspect status codes,
record types, TTL, delegation and private/public DNS behavior. If the
answer is correct, I move to TCP/TLS/HTTP rather than continuing to
treat the issue as DNS.
```

---

## 348. Interview: NXDOMAIN vs SERVFAIL?

Answer:

```text
NXDOMAIN means the queried DNS name does not exist according to the
responding DNS system. SERVFAIL means the resolver failed to
successfully complete the query. SERVFAIL can result from DNSSEC,
delegation, upstream or resolver problems.
```

---

## 349. Interview: Why Does `dig` Work but Application DNS Fail?

Answer:

```text
The application may use a different resolver path through NSS,
hosts-file entries, a runtime-specific resolver, a proxy or its own
DNS cache. I compare getent, /etc/resolv.conf and application
behavior rather than assuming dig represents every DNS lookup.
```

---

## 350. Interview: What Is Split-Horizon DNS?

Answer:

```text
Split-horizon DNS provides different answers for the same hostname
depending on the client network or resolver path. For example,
Internet users may resolve an application to a public ALB while VPC
clients resolve the same name to an internal ALB.
```

---

## 351. Interview: How Does Kubernetes DNS Work?

Answer:

```text
Pods normally query the cluster DNS Service. CoreDNS serves
cluster-local records and forwards external queries to configured
upstream resolvers. Service names are normally available under the
cluster.local DNS domain.
```

---

## 352. Interview: What Is a Headless Service?

Answer:

```text
A headless Service uses clusterIP: None. DNS can return individual
Pod endpoints rather than a single virtual Service IP. It is commonly
used for StatefulSet service discovery.
```

---

## 353. Interview: How Do You Debug CoreDNS?

Answer:

```text
I test DNS from an affected Pod, inspect /etc/resolv.conf, CoreDNS
Pods, the kube-dns Service, EndpointSlices, CoreDNS logs and Corefile.
I then determine whether cluster-local names or external names fail,
which narrows the problem to the Kubernetes plugin or upstream
forwarding path.
```

---

## 354. Interview: Why Allow TCP/53?

Answer:

```text
DNS primarily uses UDP but can use TCP for large responses and other
cases. A NetworkPolicy or firewall allowing only UDP can therefore
create intermittent or size-dependent DNS failures.
```

---

## 355. Interview: What Is Route 53 Private Hosted Zone?

Answer:

```text
It is an AWS-managed DNS zone intended for private name resolution
within associated VPCs. It is commonly used for internal applications,
private load balancers and hybrid DNS architectures.
```

---

## 356. Interview: What Is Route 53 Resolver?

Answer:

```text
Route 53 Resolver provides DNS resolution within AWS VPCs and supports
hybrid forwarding through inbound and outbound Resolver endpoints and
rules.
```

---

## 357. Interview: DNS Works but HTTPS Fails. Why?

Answer:

```text
DNS only establishes the destination address. HTTPS still requires
TCP connectivity, TLS negotiation, certificate validation, SNI and
HTTP/application routing. I test those layers separately.
```

---

## 358. Interview: DNS Works From Laptop but Not EKS?

Answer:

```text
I compare resolver paths. From EKS I inspect Pod resolv.conf, CoreDNS,
NetworkPolicy, VPC Resolver, private hosted zones and CNI/network
connectivity. The laptop's DNS result does not prove the Pod has the
same DNS path.
```

---

## 359. Interview: How Do You Troubleshoot a DNS Change Not Taking Effect?

Answer:

```text
I query the authoritative server first. If the authoritative record
is correct, I compare recursive resolvers and TTLs to determine
whether cached data remains. I also check whether split-horizon DNS
means different clients are querying different zones.
```

---

## 360. Interview: What Is DNS TTL?

Answer:

```text
TTL tells caching resolvers how long a DNS response may be cached
before it should be refreshed. It influences how quickly changes are
observed but is not a guarantee that every client updates at exactly
the same time.
```

---

## 361. Interview: How Do You Debug a CNAME?

Answer:

```text
I query the CNAME directly, follow the target record, and verify the
final A/AAAA records. I also check for loops, excessive chains and
whether the target is the intended load balancer or service endpoint.
```

---

## 362. Interview: How Do You Debug Route 53 Failover?

Answer:

```text
I inspect the failover records, health checks, routing policy and
current authoritative answers. I then test both healthy and unhealthy
states in a controlled environment because DNS failover is not
instantaneous connection failover.
```

---

## 363. Interview: How Do You Debug DNS in EKS?

Answer:

```text
I start from the affected Pod and test both cluster-local and
external names. Then I inspect CoreDNS, kube-dns Service,
EndpointSlices, Corefile, NetworkPolicy, NodeLocal DNSCache if used,
and the VPC Resolver path.
```

---

## 364. Interview: What If CoreDNS Is Healthy but External DNS Fails?

Answer:

```text
I check the CoreDNS forward configuration and the upstream resolver
path. Then I validate VPC DNS support, routing, Security Groups,
NACLs and any NetworkPolicy affecting DNS traffic.
```

---

## 365. Interview: What If Kubernetes Service DNS Fails but External DNS Works?

Answer:

```text
I focus on the Kubernetes DNS plugin and Service discovery. I verify
the Service exists, its namespace/name is correct, and its endpoints
are present. Then I inspect CoreDNS logs and configuration.
```

---

## 366. Interview: What If DNS Returns Correct IP but Curl Times Out?

Answer:

```text
DNS is working. I move to route and TCP connectivity, then inspect
Security Groups, NACLs, firewalls, listener availability and return
path.
```

---

## 367. Interview: What If DNS Returns Multiple IPs?

Answer:

```text
That can be intentional. I inspect the record set and routing policy,
then test each endpoint and determine whether the client/resolver is
caching or rotating answers as expected.
```

---

## 368. Interview: How Do You Protect Production DNS?

Answer:

```text
I use least-privilege IAM, IaC, change review, audit logging,
environment separation and controlled automation. For public domains
I consider DNSSEC where appropriate, and for private DNS I carefully
control VPC associations and Resolver rules.
```

---

## 369. Interview: How Do You Monitor DNS?

Answer:

```text
I monitor DNS availability, latency, SERVFAIL rate, critical record
resolution, CoreDNS health and upstream failures. For critical
applications I also use synthetic checks that validate DNS plus TCP,
TLS and HTTP rather than DNS alone.
```

---

## 370. Final DNS Command Cheat Sheet

```bash
# Basic
dig example.com
dig +short example.com
dig example.com A
dig example.com AAAA
dig example.com CNAME
dig example.com NS
dig example.com SOA
dig example.com MX
dig example.com TXT
dig -x <ip>

# Resolver
dig @<resolver> example.com
dig +trace example.com
dig +tcp example.com
dig +dnssec example.com
dig +stats example.com

# System
cat /etc/resolv.conf
cat /etc/hosts
cat /etc/nsswitch.conf
getent hosts example.com
resolvectl status

# Kubernetes
kubectl get pods -n kube-system
kubectl get svc -n kube-system kube-dns
kubectl get endpointslice -n kube-system
kubectl get configmap coredns -n kube-system -o yaml
kubectl logs -n kube-system -l k8s-app=kube-dns

# AWS
aws sts get-caller-identity
aws route53 list-hosted-zones
aws route53 list-resource-record-sets --hosted-zone-id <id>
aws route53 list-health-checks
aws ec2 describe-vpc-attribute --vpc-id <id> --attribute enableDnsSupport
aws ec2 describe-vpc-attribute --vpc-id <id> --attribute enableDnsHostnames

# Packet
tcpdump -ni any port 53
```

---

## 371. Final DNS Troubleshooting Matrix

| Symptom | First Checks |
|---|---|
| NXDOMAIN | Record, zone, delegation |
| SERVFAIL | Resolver, DNSSEC, delegation |
| Timeout | Network, firewall, resolver |
| Wrong IP | Authoritative vs recursive |
| Only one network fails | Split DNS, local resolver |
| EKS external DNS fails | CoreDNS upstream/VPC Resolver |
| EKS internal DNS fails | Service/CoreDNS/private zone |
| DNS works, TCP fails | Route/SG/NACL/listener |
| TCP works, TLS fails | Certificate/SNI/TLS |
| TLS works, HTTP fails | Host/path/LB/application |

---

## 372. Final DNS Production Checklist

```text
[ ] Critical domains documented
[ ] Zone ownership documented
[ ] Public/private zones identified
[ ] Split-horizon behavior documented
[ ] Route 53 records managed through approved process
[ ] TTLs reviewed
[ ] Health checks reviewed
[ ] DNSSEC considered where appropriate
[ ] Resolver architecture documented
[ ] Hybrid forwarding documented
[ ] CoreDNS monitored
[ ] NodeLocal DNSCache documented if used
[ ] NetworkPolicy allows required DNS
[ ] TCP/53 considered
[ ] Critical DNS synthetic checks enabled
[ ] DNS changes audited
[ ] Rollback procedure documented
```

---

## 373. Final Production Principles

```text
1. Always identify the exact hostname.
2. Identify the DNS resolver being used.
3. Identify the expected record type.
4. Query the default resolver first.
5. Compare with an authoritative server.
6. Compare multiple recursive resolvers when useful.
7. Understand NXDOMAIN vs NODATA vs SERVFAIL.
8. Check TTL and negative caching.
9. Check public vs private DNS.
10. Check split-horizon behavior.
11. Follow CNAME chains.
12. Check delegation with +trace.
13. Check DNSSEC when SERVFAIL is unexplained.
14. Remember DNS can use UDP and TCP.
15. Test DNS from the actual workload.
16. Check /etc/resolv.conf in containers.
17. Check CoreDNS for Kubernetes DNS.
18. Check NetworkPolicy for DNS traffic.
19. Check VPC Resolver for AWS private DNS.
20. Check Resolver rules for hybrid DNS.
21. DNS success does not prove TCP.
22. TCP success does not prove TLS.
23. TLS success does not prove HTTP.
24. Do not call every application outage a DNS outage.
25. Use evidence before changing records.
26. Protect production DNS with least privilege.
27. Manage DNS through IaC where appropriate.
28. Monitor DNS continuously for critical systems.
29. Test failover before relying on it.
30. Document the complete DNS path.
```

---