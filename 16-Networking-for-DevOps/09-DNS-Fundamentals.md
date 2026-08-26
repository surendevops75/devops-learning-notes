# 16-Networking-for-DevOps
# 09-DNS-Fundamentals

## 1. Purpose

DNS is one of the most important dependencies in modern DevOps environments.

Almost every production request begins with name resolution:

```text
Application
    |
    v
DNS resolution
    |
    v
IP address
    |
    v
TCP/UDP connection
    |
    v
Application protocol
```

A DNS problem can therefore appear as:

```text
application outage
API timeout
Kubernetes Service failure
EKS microservice failure
ALB access problem
database connection failure
external API failure
slow application startup
```

This file builds a production-level DNS foundation before the next file goes deeper into DNS records and resolution.

---

## 2. What Is DNS?

DNS means:

```text
Domain Name System
```

DNS translates human-readable names into information such as IP addresses.

Example:

```text
www.example.com
        |
        v
203.0.113.10
```

DNS is a distributed hierarchical naming system.

---

## 3. Why DNS Is Needed

Without DNS, users and applications would need to remember IP addresses.

Instead of:

```text
https://203.0.113.10
```

we use:

```text
https://api.example.com
```

DNS also enables infrastructure to change addresses without requiring every client to hard-code the new IP.

---

## 4. DNS Is More Than IP Resolution

DNS can provide:

```text
IPv4 addresses
IPv6 addresses
mail servers
aliases
service discovery
text metadata
certificate-validation records
load-balancing information
```

Examples include:

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
```

These records are covered in greater depth in the next file.

---

## 5. DNS Hierarchy

DNS is hierarchical:

```text
.
|
+-- com
|    |
|    +-- example
|         |
|         +-- www
|         +-- api
|
+-- org
|
+-- in
```

The hierarchy starts at the root:

```text
.
```

---

## 6. Root

The DNS root is represented by:

```text
.
```

It is the top of the DNS hierarchy.

Root servers do not normally provide the final IP address for:

```text
www.example.com
```

Instead, they help direct recursive resolvers toward the appropriate top-level-domain servers.

---

## 7. Top-Level Domain

A TLD is a domain immediately below the root.

Examples:

```text
.com
.org
.net
.in
.io
```

For:

```text
www.example.com
```

the TLD is:

```text
.com
```

---

## 8. Authoritative Domain

Under a TLD can be an authoritative domain.

Example:

```text
example.com
```

The authoritative DNS infrastructure for `example.com` contains records for names under that zone.

---

## 9. Hostname

A complete name can contain labels:

```text
www.example.com
```

Labels are separated by dots:

```text
www
example
com
```

The trailing root dot is normally omitted in everyday usage:

```text
www.example.com.
```

and:

```text
www.example.com
```

represent the same fully qualified domain name when interpreted as an FQDN.

---

## 10. FQDN

FQDN means:

```text
Fully Qualified Domain Name
```

Example:

```text
api.production.example.com.
```

The final:

```text
.
```

represents the DNS root.

---

## 11. DNS Labels

DNS names are built from labels.

Example:

```text
api.prod.example.com.
```

contains:

```text
api
prod
example
com
.
```

Each label represents one level of the hierarchy.

---

## 12. DNS Zones

A DNS zone is an administratively managed portion of the DNS namespace.

For example:

```text
example.com
```

can be a DNS zone.

A zone contains resource records and delegation information.

---

## 13. Zone vs Domain

A domain is a namespace concept.

A zone is an administrative DNS data boundary.

A domain can contain delegated subdomains that are managed through separate zones.

Example:

```text
example.com
     |
     +-- dev.example.com
```

`dev.example.com` can be delegated to different authoritative DNS servers.

---

## 14. Delegation

Delegation allows responsibility for part of a namespace to be transferred to another set of authoritative DNS servers.

Conceptually:

```text
example.com
     |
     +-- dev.example.com
              |
              +-- authoritative servers
```

This is important in enterprise DNS architectures.

---

## 15. Recursive Resolver

A recursive resolver performs DNS lookups on behalf of clients.

Typical flow:

```text
Client
  |
  v
Recursive Resolver
  |
  +--> Root
  |
  +--> TLD
  |
  +--> Authoritative DNS
```

The resolver then returns the result to the client.

---

## 16. Authoritative DNS Server

An authoritative DNS server holds the authoritative records for one or more zones.

It answers from its configured DNS data.

Example:

```text
Authoritative server
       |
       +-- api.example.com → 203.0.113.20
```

It does not need to recursively search the entire DNS hierarchy for every authoritative answer.

---

## 17. Recursive vs Authoritative

### Recursive resolver

```text
Find the answer for me.
```

### Authoritative server

```text
I am authoritative for this zone.
Here is the record.
```

This distinction is fundamental.

---

## 18. Stub Resolver

Applications usually do not implement full DNS recursion themselves.

The operating system commonly provides a resolver interface.

Conceptually:

```text
Application
    |
    v
OS resolver
    |
    v
Configured DNS server
```

The exact implementation differs across Linux distributions and resolver stacks.

---

## 19. Linux `/etc/resolv.conf`

Inspect:

```bash
cat /etc/resolv.conf
```

You may see:

```text
nameserver 10.0.0.2
search example.internal
options timeout:2 attempts:3
```

The exact contents depend on the host environment.

---

## 20. `nameserver`

A line such as:

```text
nameserver 10.0.0.2
```

specifies a DNS server to query.

A machine may have multiple configured nameservers.

---

## 21. Search Domains

Example:

```text
search prod.example.com example.com
```

If an application attempts to resolve:

```text
api
```

the resolver may try search-domain combinations according to resolver behavior.

This can be convenient but can also cause unexpected queries and delays.

---

## 22. Resolver Options

`/etc/resolv.conf` can contain options controlling resolver behavior.

Examples:

```text
timeout
attempts
ndots
```

The exact semantics depend on the resolver implementation.

---

## 23. `ndots`

A common Kubernetes setting is:

```text
options ndots:5
```

This can cause short or partially qualified names to be attempted with search domains before an absolute query.

This becomes important for Kubernetes DNS performance.

---

## 24. DNS Query

A DNS query generally asks:

```text
What resource record exists for this name and type?
```

Example:

```text
api.example.com
Type: A
```

The resolver returns the appropriate answer if one is available.

---

## 25. DNS Response

A response can contain:

```text
Answer section
Authority section
Additional section
```

It can also contain metadata such as:

```text
status
flags
TTL
```

---

## 26. DNS Query Types

Common query types:

```text
A
AAAA
CNAME
MX
NS
SOA
TXT
PTR
SRV
```

Their detailed behavior is covered in the next file.

---

## 27. A Record

An A record maps a hostname to an IPv4 address.

Example:

```text
api.example.com → 203.0.113.10
```

---

## 28. AAAA Record

An AAAA record maps a hostname to an IPv6 address.

Example:

```text
api.example.com → 2001:db8::10
```

---

## 29. CNAME

A CNAME creates an alias to another DNS name.

Example:

```text
www.example.com
      |
      v
app.example.net
```

The resolver continues resolution for the target name.

---

## 30. MX

MX records identify mail exchange servers for a domain.

Example:

```text
example.com
    |
    +-- mail.example.com
```

Mail delivery uses MX records rather than simply assuming the web server's address.

---

## 31. NS

NS records identify authoritative nameservers for a DNS zone or delegated namespace.

Example:

```text
example.com
   |
   +-- ns1.example-dns.com
   +-- ns2.example-dns.com
```

---

## 32. SOA

SOA means:

```text
Start of Authority
```

It contains zone-level administrative information.

Important fields include:

```text
primary/authoritative server
responsible-party field
serial
refresh
retry
expire
negative-cache TTL-related value
```

---

## 33. TXT

TXT records store text data.

Common uses include:

```text
SPF-related information
domain verification
DKIM
DMARC-related records
certificate validation
service configuration
```

---

## 34. PTR

PTR records support reverse DNS.

Example:

```text
203.0.113.10
      |
      v
api.example.com
```

Reverse DNS uses the:

```text
in-addr.arpa
```

namespace for IPv4.

IPv6 uses:

```text
ip6.arpa
```

---

## 35. SRV

SRV records can advertise services, including:

```text
service
protocol
priority
weight
port
target
```

They are useful for service discovery in systems that support SRV.

---

## 36. DNS Transport

DNS traditionally uses:

```text
UDP 53
```

for many standard queries.

DNS can also use:

```text
TCP 53
```

when required.

---

## 37. Why DNS Uses UDP

UDP provides low overhead and avoids requiring a connection handshake for every standard query.

A typical query is small and fits into a single datagram in common cases.

---

## 38. Why DNS Uses TCP

TCP can be required for:

```text
large responses
zone transfers
truncated UDP responses
certain protocol requirements
```

DNS implementations can retry over TCP when necessary.

---

## 39. DNS Over TLS

DNS over TLS is commonly called:

```text
DoT
```

It encrypts DNS communication over TLS.

The conventional port is:

```text
853
```

---

## 40. DNS Over HTTPS

DNS over HTTPS is commonly called:

```text
DoH
```

It carries DNS queries over HTTPS, commonly using:

```text
TCP 443
```

and potentially modern HTTP transport behavior.

---

## 41. Traditional DNS vs DoT vs DoH

```text
Traditional DNS
UDP/TCP 53

DoT
TLS 853

DoH
HTTPS 443
```

Production architecture should choose based on security, control, compatibility and operational requirements.

---

## 42. DNS Caching

DNS responses can be cached.

Caching reduces:

```text
latency
DNS traffic
recursive resolver load
authoritative server load
```

The cache lifetime is influenced by TTL.

---

## 43. TTL

TTL means:

```text
Time To Live
```

A DNS record's TTL indicates how long a resolver may cache the record before it needs to refresh it.

Example:

```text
TTL = 300
```

means approximately:

```text
5 minutes
```

for the DNS cache lifetime under normal resolver behavior.

---

## 44. TTL Is Not a Global Propagation Timer

A common misconception is:

```text
DNS TTL = exact global propagation time
```

Reality is more complex because:

```text
different resolvers
negative caching
application caches
OS caches
local DNS caches
provider behavior
```

can affect observed timing.

---

## 45. Positive Caching

If:

```text
api.example.com → 10.0.1.20
TTL = 300
```

a resolver can reuse the result for the applicable TTL period.

---

## 46. Negative Caching

Resolvers can cache certain negative responses.

Example:

```text
NXDOMAIN
```

can be cached according to the relevant DNS negative-caching rules and SOA information.

Therefore adding a record does not always immediately make it visible to every client.

---

## 47. NXDOMAIN

NXDOMAIN means the queried DNS name does not exist.

Example:

```text
does-not-exist.example.com
```

can return:

```text
NXDOMAIN
```

This differs from an existing name that has no record of the requested type.

---

## 48. NOERROR With No Answer

A name may exist while the requested record type does not.

For example:

```text
example.com
```

may exist, but an A query could return no A answer while other records exist.

This is different from NXDOMAIN.

---

## 49. DNS Cache Poisoning

DNS cache poisoning attempts to cause a resolver to cache malicious DNS information.

Defenses include:

```text
DNSSEC
secure resolver configuration
randomized query identifiers/ports
network security
trusted DNS infrastructure
```

---

## 50. DNSSEC

DNSSEC adds cryptographic validation to DNS data.

It helps provide:

```text
data authenticity
data integrity
```

It does not encrypt normal DNS queries.

---

## 51. DNSSEC Does Not Encrypt DNS

Important distinction:

```text
DNSSEC
→ validates DNS data

DoT/DoH
→ encrypts DNS transport
```

They address different security concerns.

---

## 52. DNS Architecture

A simplified architecture:

```text
Application
     |
     v
Stub Resolver
     |
     v
Recursive Resolver
     |
     v
+---------+
| Root    |
+---------+
     |
     v
+---------+
| TLD     |
+---------+
     |
     v
+---------------+
| Authoritative |
+---------------+
     |
     v
DNS answer
```

---

## 53. Recursive Resolution

Suppose the client asks for:

```text
www.example.com
```

The recursive resolver may need to discover:

```text
root
→ .com
→ example.com authoritative server
→ www.example.com
```

The resolver then returns the final result to the client.

---

## 54. Root Referral

The root does not normally provide the final A record for:

```text
www.example.com
```

It provides information that allows the resolver to continue toward:

```text
.com
```

---

## 55. TLD Referral

The TLD servers can direct the resolver toward the authoritative nameservers for:

```text
example.com
```

---

## 56. Authoritative Answer

The authoritative server for the relevant zone can return:

```text
www.example.com
A
203.0.113.10
```

The recursive resolver can cache it according to TTL.

---

## 57. Iterative vs Recursive Queries

A recursive client asks:

```text
Give me the answer.
```

An iterative interaction allows the resolver to receive referrals and continue the lookup itself.

The client generally talks to a recursive resolver rather than directly performing the full hierarchy traversal.

---

## 58. DNS Resolver Cache

A recursive resolver may cache:

```text
A
AAAA
CNAME
NS
negative answers
```

according to DNS rules and TTLs.

This is why repeated lookups can be faster.

---

## 59. Local Cache

DNS information may be cached at multiple levels:

```text
browser
application
OS
local DNS cache
recursive resolver
```

Therefore debugging DNS requires knowing which layer is returning the result.

---

## 60. Browser DNS Behavior

Modern browsers can have their own DNS behavior or caches.

Some environments also support secure DNS mechanisms.

If:

```bash
dig
```

and the browser disagree, inspect:

```text
browser cache
DoH
proxy
OS resolver
```

---

## 61. DNS Resolver Selection

A production host may use:

```text
local resolver
corporate resolver
cloud-provided resolver
Kubernetes CoreDNS
public resolver
```

The correct choice depends on architecture and security requirements.

---

## 62. AWS VPC DNS

AWS VPCs provide DNS functionality through the VPC resolver.

The commonly used VPC resolver address is based on the VPC network range and is often reachable through the VPC's resolver mechanism.

In a typical VPC:

```text
VPC CIDR
10.0.0.0/16
```

the resolver is commonly:

```text
10.0.0.2
```

but use the environment's actual resolver configuration rather than hard-coding assumptions.

---

## 63. AWS DNS Support

VPC DNS behavior depends on VPC settings such as:

```text
enableDnsSupport
enableDnsHostnames
```

Check your actual VPC configuration when troubleshooting.

---

## 64. Route 53

AWS Route 53 provides managed DNS capabilities.

Common uses include:

```text
public DNS
private DNS
health checks
routing policies
domain registration
service discovery
```

---

## 65. Public Hosted Zone

A public hosted zone contains DNS records intended to be resolvable through the public DNS hierarchy.

Example:

```text
example.com
```

can have public records for:

```text
www.example.com
api.example.com
```

---

## 66. Private Hosted Zone

A private hosted zone is associated with one or more VPCs.

Example:

```text
internal.example.com
```

can resolve only within the intended private DNS environment.

---

## 67. Public vs Private DNS

```text
Public zone
    |
Internet DNS

Private zone
    |
VPC/internal DNS
```

This separation is fundamental in AWS architectures.

---

## 68. Split-Horizon DNS

Split-horizon DNS means the same or related name can resolve differently depending on where the client is.

Example:

```text
api.example.com

Internet → public ALB
VPC      → private endpoint
```

This can support hybrid architectures.

---

## 69. DNS and EKS

Kubernetes workloads depend heavily on DNS.

Examples:

```text
service.namespace.svc.cluster.local
```

and:

```text
external-api.example.com
```

EKS therefore requires reliable DNS resolution for both cluster-internal and external dependencies.

---

## 70. CoreDNS

CoreDNS is commonly used as the DNS service inside Kubernetes clusters.

Architecture:

```text
Pod
 |
DNS query
 |
CoreDNS
 |
+-- Kubernetes Service discovery
|
+-- upstream resolver
```

---

## 71. Kubernetes DNS

A Kubernetes Service can commonly be reached through:

```text
service.namespace.svc.cluster.local
```

Example:

```text
cart.default.svc.cluster.local
```

The exact DNS behavior depends on Service type and cluster configuration.

---

## 72. Short Kubernetes Names

From a Pod in the same namespace:

```text
cart
```

may resolve through Kubernetes search domains.

The resolver may construct names such as:

```text
cart.default.svc.cluster.local
```

depending on `/etc/resolv.conf` and resolver behavior.

---

## 73. Kubernetes `/etc/resolv.conf`

Inside a Pod, inspect:

```bash
kubectl exec -it <pod> -- cat /etc/resolv.conf
```

You may see:

```text
nameserver 10.100.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

Addresses vary by cluster.

---

## 74. CoreDNS Service

Inspect:

```bash
kubectl get svc -n kube-system kube-dns
```

Kubernetes commonly uses the Service named:

```text
kube-dns
```

even when CoreDNS is the actual DNS implementation.

---

## 75. CoreDNS Pods

Check:

```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

The label and deployment details can vary by Kubernetes distribution/version.

---

## 76. CoreDNS Configuration

Inspect:

```bash
kubectl -n kube-system get configmap coredns -o yaml
```

The Corefile defines plugins and upstream behavior.

---

## 77. CoreDNS Plugins

Common plugins include:

```text
kubernetes
forward
cache
errors
health
ready
loop
reload
```

The exact configuration depends on the cluster.

---

## 78. Kubernetes Plugin

The `kubernetes` CoreDNS plugin provides DNS records derived from Kubernetes resources.

It can resolve:

```text
Services
Pods
```

according to configured behavior.

---

## 79. Forward Plugin

The CoreDNS `forward` plugin sends queries outside the Kubernetes DNS domain to upstream resolvers.

Conceptually:

```text
Pod
 |
CoreDNS
 |
forward
 |
AWS/VPC resolver
 |
Internet/internal DNS
```

---

## 80. CoreDNS Cache

The `cache` plugin reduces repeated upstream lookups.

Caching improves:

```text
latency
upstream efficiency
resilience
```

but stale/old results remain possible until TTL-related cache behavior expires.

---

## 81. CoreDNS Scaling

DNS is a critical cluster dependency.

Monitor:

```text
CPU
memory
DNS query rate
latency
errors
timeouts
pod availability
```

Scale CoreDNS appropriately for cluster size and workload.

---

## 82. CoreDNS Failure

If CoreDNS is unavailable:

```text
Service discovery fails
external name resolution may fail
applications can time out
```

This can look like a broad microservice outage.

---

## 83. DNS Failure Symptoms in Kubernetes

Common symptoms:

```text
temporary failure in name resolution
SERVFAIL
NXDOMAIN
connection timeout
unknown host
application startup failure
```

Start by testing DNS directly.

---

## 84. DNS Test Pod

A temporary debugging Pod can use:

```bash
kubectl run dns-debug \
  --rm -it \
  --image=busybox:1.36 \
  --restart=Never -- sh
```

Then:

```bash
nslookup kubernetes.default
```

Image availability and tags can vary; use an approved debugging image in production.

---

## 85. `nslookup`

Basic query:

```bash
nslookup example.com
```

Kubernetes:

```bash
nslookup kubernetes.default
```

It is useful for quick checks but provides less diagnostic detail than `dig`.

---

## 86. `dig`

Basic:

```bash
dig example.com
```

IPv4:

```bash
dig A example.com
```

IPv6:

```bash
dig AAAA example.com
```

Nameserver:

```bash
dig NS example.com
```

---

## 87. `dig +short`

For concise output:

```bash
dig +short example.com
```

This is convenient in scripts and quick troubleshooting.

---

## 88. `dig +trace`

To inspect iterative resolution:

```bash
dig +trace example.com
```

This can show:

```text
root
TLD
authoritative
```

referrals.

It is extremely useful for understanding DNS hierarchy.

---

## 89. `dig @server`

Query a specific resolver:

```bash
dig @10.0.0.2 example.com
```

This helps compare:

```text
local resolver
corporate resolver
public resolver
authoritative server
```

---

## 90. Query Authoritative Server Directly

First find nameservers:

```bash
dig NS example.com
```

Then:

```bash
dig @ns1.example.com www.example.com
```

This helps determine whether the authoritative data differs from a recursive cache.

---

## 91. `dig +stats`

Example:

```bash
dig +stats example.com
```

It provides query timing and response statistics.

---

## 92. DNS Response Flags

Common flags include:

```text
qr
aa
rd
ra
ad
cd
```

Important examples:

```text
aa = authoritative answer
rd = recursion desired
ra = recursion available
ad = authenticated data
```

Interpret based on the actual query and resolver.

---

## 93. Authoritative Answer

If the response contains:

```text
aa
```

the responding server is indicating that the answer is authoritative for the relevant zone.

---

## 94. Recursion Desired

A client can request recursion using:

```text
RD
```

Recursive resolvers normally support it.

Authoritative servers may not provide recursive service.

---

## 95. SERVFAIL

SERVFAIL means the resolver could not successfully complete the query.

Possible causes:

```text
upstream failure
DNSSEC validation problem
authoritative server issue
network failure
configuration error
timeout
```

Do not confuse SERVFAIL with NXDOMAIN.

---

## 96. REFUSED

REFUSED indicates that the DNS server refused to perform the requested operation.

Possible reasons:

```text
policy
recursion disabled
access control
server configuration
```

---

## 97. DNS Timeout

A DNS timeout can be caused by:

```text
UDP 53 blocked
TCP 53 blocked
CoreDNS unavailable
resolver unavailable
network route
security group
NACL
firewall
upstream DNS failure
```

---

## 98. DNS and NetworkPolicy

A Kubernetes default-deny egress policy can accidentally block DNS.

Example problem:

```text
Pod
 |
DNS query
 X
CoreDNS
```

The application then reports:

```text
unknown host
```

even though the destination service is healthy.

---

## 99. Allowing DNS Egress

A NetworkPolicy may need to permit DNS traffic to the cluster DNS Pods/Service.

The exact policy depends on:

```text
CNI
DNS Service IP
CoreDNS namespace
pod labels
protocol
port
```

Common DNS ports:

```text
UDP 53
TCP 53
```

---

## 100. DNS and Security Groups

If DNS queries leave the expected network path, AWS security controls must permit the required traffic.

For VPC resolver usage, verify the VPC DNS configuration and network path rather than assuming an external public resolver.

---

## 101. DNS and NACLs

NACLs are stateless.

If DNS traffic uses:

```text
UDP 53
```

return traffic must also be permitted appropriately.

For TCP fallback:

```text
TCP 53
```

must be handled as well.

---

## 102. DNS and Ephemeral Ports

A DNS client may send:

```text
source ephemeral port
→ destination 53
```

The return traffic goes back to that source port.

Stateless filtering must account for the return path.

---

## 103. DNS Latency

High DNS latency can affect:

```text
application startup
HTTP requests
microservice calls
database connections
external API calls
```

Measure DNS separately from TCP.

---

## 104. `curl` DNS Timing

Use:

```bash
curl -s -o /dev/null \
-w 'dns=%{time_namelookup}\nconnect=%{time_connect}\ntotal=%{time_total}\n' \
https://example.com
```

If:

```text
dns = 2.5s
connect = 0.01s
```

DNS is a strong latency suspect.

---

## 105. DNS Query Volume

Microservices can generate very high DNS traffic when:

```text
short-lived Pods start
connections are repeatedly created
clients resolve names on every request
TTL is low
applications do not cache efficiently
```

Monitor query volume before blindly increasing CoreDNS replicas.

---

## 106. `ndots:5` and Kubernetes

Kubernetes commonly configures:

```text
ndots:5
```

For an external name:

```text
api.example.com
```

a resolver may try search-domain variants before treating it as absolute, depending on resolver behavior.

This can create multiple DNS queries.

---

## 107. Fully Qualified Names in Kubernetes

Adding a trailing dot can make a name absolute:

```text
api.example.com.
```

This can avoid unnecessary search-domain expansion in resolver implementations that honor the FQDN directly.

Use this deliberately; application compatibility should be verified.

---

## 108. DNS Search Explosion

Suppose:

```text
search namespace.svc.cluster.local svc.cluster.local cluster.local
ndots:5
```

A partially qualified name can result in several queries.

At large scale this can create unnecessary CoreDNS load.

---

## 109. DNS Optimization

Potential strategies:

```text
appropriate FQDN usage
application DNS caching
connection pooling
reasonable TTLs
CoreDNS scaling
NodeLocal DNSCache
reducing unnecessary lookups
```

Choose based on measured behavior.

---

## 110. NodeLocal DNSCache

Kubernetes environments can use NodeLocal DNSCache to improve DNS performance and reduce pressure on CoreDNS.

Conceptually:

```text
Pod
 |
Node-local DNS cache
 |
CoreDNS
 |
Upstream DNS
```

This is particularly useful at larger cluster scale.

---

## 111. DNS Caching Architecture in EKS

A scalable design may look like:

```text
Pods
 |
NodeLocal DNSCache
 |
CoreDNS
 |
AWS VPC Resolver
 |
Route 53 / upstream DNS
```

The exact deployment depends on cluster requirements.

---

## 112. CoreDNS High Availability

Run multiple CoreDNS replicas so that a single Pod failure does not make cluster DNS unavailable.

Example:

```text
CoreDNS-1
CoreDNS-2
CoreDNS-3
```

Spread replicas across nodes where practical.

---

## 113. CoreDNS Scheduling

Production clusters should consider:

```text
Pod anti-affinity
topology spread
resource requests
resource limits
priority
disruption behavior
```

DNS is infrastructure, not an ordinary application workload.

---

## 114. CoreDNS Resource Requests

Example pattern:

```yaml
resources:
  requests:
    cpu: 100m
    memory: 70Mi
  limits:
    memory: 170Mi
```

Actual values should be based on cluster workload and measured usage rather than copied blindly.

---

## 115. CoreDNS Autoscaling

For large clusters, autoscaling can help adjust DNS capacity based on workload.

Monitor:

```text
query volume
CPU
latency
errors
```

before selecting scaling thresholds.

---

## 116. DNS Monitoring

Useful metrics include:

```text
query rate
request latency
SERVFAIL
NXDOMAIN
timeouts
CPU
memory
cache hit rate
upstream failures
```

Grafana dashboards can correlate DNS problems with application latency.

---

## 117. DNS Logging

CoreDNS can be configured to log DNS queries.

However, verbose DNS logging can generate significant volume.

Use it temporarily or selectively during troubleshooting.

---

## 118. CoreDNS Errors

Check:

```bash
kubectl logs -n kube-system \
  -l k8s-app=kube-dns
```

The label selector can vary by deployment.

Look for:

```text
SERVFAIL
timeout
plugin errors
upstream errors
loop detection
```

---

## 119. CoreDNS Deployment

Inspect:

```bash
kubectl get deployment -n kube-system coredns
```

Then:

```bash
kubectl describe deployment -n kube-system coredns
```

---

## 120. CoreDNS Service

Inspect:

```bash
kubectl get svc -n kube-system kube-dns
kubectl describe svc -n kube-system kube-dns
```

Check:

```text
ClusterIP
ports
endpoints
selector
```

---

## 121. CoreDNS Endpoint Check

```bash
kubectl get endpointslice \
  -n kube-system \
  -l kubernetes.io/service-name=kube-dns
```

If CoreDNS has no usable endpoints, cluster DNS cannot function normally.

---

## 122. Pod DNS Debugging

Inside a failing Pod:

```bash
cat /etc/resolv.conf
```

Then:

```bash
nslookup kubernetes.default
```

Then:

```bash
nslookup <service>.<namespace>
```

Then test external DNS:

```bash
nslookup example.com
```

This separates:

```text
cluster DNS
```

from:

```text
external DNS
```

problems.

---

## 123. DNS Debugging Sequence

```text
1. Inspect /etc/resolv.conf
2. Query cluster Service
3. Query CoreDNS Service
4. Query external domain
5. Query CoreDNS directly if needed
6. Check CoreDNS Pods
7. Check EndpointSlices
8. Check NetworkPolicy
9. Check node/VPC networking
10. Check upstream DNS
```

---

## 124. Test CoreDNS Directly

Find the DNS Service IP:

```bash
kubectl get svc -n kube-system kube-dns
```

Then from a debug Pod:

```bash
nslookup kubernetes.default <DNS-SERVICE-IP>
```

This can isolate resolver path issues.

---

## 125. Test External DNS Directly

```bash
nslookup example.com <DNS-SERVICE-IP>
```

If internal Kubernetes names work but external names fail:

```text
CoreDNS Kubernetes plugin may work
upstream forwarding may be failing
```

---

## 126. Test Public Resolver Carefully

For diagnosis, you can compare against a public resolver when network policy permits:

```bash
dig @1.1.1.1 example.com
```

or:

```bash
dig @8.8.8.8 example.com
```

Do not use public resolvers as a production workaround without understanding enterprise DNS, privacy, routing and security requirements.

---

## 127. Internal DNS

Enterprise environments often require names such as:

```text
db.internal.example.com
```

to resolve only through internal DNS.

Do not assume public DNS will know private records.

---

## 128. Hybrid DNS

AWS environments may integrate:

```text
VPC
Route 53
on-premises DNS
Direct Connect
VPN
Route 53 Resolver endpoints
```

This supports hybrid name resolution.

---

## 129. Route 53 Resolver

Route 53 Resolver provides DNS resolution within AWS and can support hybrid DNS architectures through resolver endpoints and forwarding rules.

---

## 130. Inbound Resolver Endpoint

An inbound resolver endpoint allows DNS queries from connected networks to be sent into AWS DNS resolution.

Example:

```text
On-prem
   |
VPN / Direct Connect
   |
Inbound Resolver Endpoint
   |
VPC DNS
```

---

## 131. Outbound Resolver Endpoint

An outbound resolver endpoint allows DNS queries from AWS networks to be forwarded to external/internal DNS servers.

Example:

```text
EKS
 |
VPC Resolver
 |
Outbound Resolver Endpoint
 |
On-prem DNS
```

---

## 132. Resolver Rules

Route 53 Resolver rules can route queries for specific domains to designated DNS servers.

Example:

```text
corp.example.com
       |
       v
on-prem DNS
```

while:

```text
amazonaws.com
       |
       v
AWS resolver
```

---

## 133. Multi-Account DNS

Large AWS organizations may have:

```text
DNS/shared services account
      |
      +-- Dev account
      +-- QA account
      +-- Prod account
```

Private hosted zones and resolver configurations can be associated across VPCs/accounts using appropriate AWS mechanisms.

---

## 134. Multi-Region DNS

A global architecture can use:

```text
Route 53
 |
 +-- us-east
 |
 +-- eu-west
 |
 +-- ap-south
```

Routing policies can direct users toward appropriate endpoints.

---

## 135. DNS Failover

DNS-based failover can return different endpoints depending on health and routing policy.

Example:

```text
Primary
   |
healthy
   |
return primary

Primary unhealthy
   |
return secondary
```

DNS failover has caching implications because clients and resolvers may retain previous answers until TTL-related behavior allows refresh.

---

## 136. DNS Weighted Routing

Weighted DNS can distribute traffic between records.

Example:

```text
90% → primary
10% → canary
```

It can support controlled traffic distribution, but DNS-level traffic splitting is not equivalent to per-request load balancing.

---

## 137. DNS Latency-Based Routing

Latency-based routing can direct clients toward an endpoint based on AWS's latency measurements.

It is useful for multi-region architectures.

---

## 138. DNS Geolocation

Geolocation routing can return different DNS responses based on client geographic location.

Use it carefully because client resolver location may not always equal the end user's exact location.

---

## 139. DNS and ALB

A common AWS architecture:

```text
User
 |
Route 53
 |
ALB
 |
EKS Ingress
 |
Service
 |
Pod
```

DNS resolves the ALB hostname or an application alias to the appropriate endpoint.

---

## 140. DNS and ALB Alias

Route 53 can use alias functionality to map a domain to supported AWS resources such as an ALB.

Conceptually:

```text
app.example.com
      |
      v
ALB DNS name
```

---

## 141. DNS and Kubernetes Ingress

An application domain:

```text
shop.example.com
```

can resolve to an ALB created from Kubernetes Ingress configuration.

The DNS record and Ingress lifecycle should be managed consistently.

---

## 142. ExternalDNS

ExternalDNS can synchronize Kubernetes resources with DNS providers.

Conceptually:

```text
Kubernetes Ingress/Service
        |
        v
ExternalDNS
        |
        v
Route 53
```

This enables declarative DNS management.

---

## 143. ExternalDNS Security

ExternalDNS must be granted least-privilege permissions.

Avoid allowing unrestricted:

```text
route53:*
```

when narrower permissions are sufficient.

Use IAM policies scoped to intended hosted zones/resources where practical.

---

## 144. GitOps and DNS

In a GitOps environment:

```text
Git
 |
DNS configuration
 |
Argo CD
 |
Kubernetes
 |
ExternalDNS
 |
Route 53
```

This can make DNS changes auditable and repeatable.

---

## 145. DNS and Terraform

Terraform can manage:

```text
Route 53 zones
records
resolver rules
resolver endpoints
VPC DNS settings
```

Use Terraform for infrastructure-level DNS and GitOps for application-level DNS when that separation fits the organization's operating model.

---

## 146. DNS Change Safety

Before changing DNS:

```text
record type
TTL
current value
target
health checks
routing policy
failover behavior
```

must be understood.

---

## 147. Low TTL Tradeoff

Lower TTL:

```text
faster DNS changes
more DNS queries
higher resolver load
```

Higher TTL:

```text
better caching
fewer queries
slower change visibility
```

Choose TTL based on operational requirements.

---

## 148. DNS Migration

During migration:

```text
old endpoint
    |
    v
new endpoint
```

common strategies include:

```text
lower TTL before planned change
validate new endpoint
change record
monitor
restore higher TTL after stabilization
```

Do not assume every client respects TTL perfectly.

---

## 149. Blue-Green DNS

Conceptually:

```text
app.example.com
      |
      +-- Blue
      |
      +-- Green
```

DNS can switch the primary answer.

However, DNS caching means traffic does not necessarily move instantaneously.

---

## 150. DNS Canary

A canary strategy can use:

```text
small weighted DNS allocation
```

before increasing traffic.

For application-level precision, load balancer or service-mesh traffic controls may be more appropriate.

---

## 151. DNS Security Principles

Production DNS should follow:

```text
least privilege
zone separation
private/public separation
DNSSEC where appropriate
controlled change management
audit logging
monitoring
redundancy
```

---

## 152. DNS Access Control

Only authorized systems should modify DNS zones.

For AWS:

```text
IAM
```

should restrict:

```text
hosted zone
record changes
resolver configuration
```

to required scope.

---

## 153. DNS Auditing

Track:

```text
who changed records
what changed
when
old value
new value
```

In AWS, use appropriate CloudTrail and Route 53 logging/monitoring capabilities.

---

## 154. DNS Disaster Recovery

DNS itself is part of disaster recovery.

Consider:

```text
multiple authoritative servers
provider redundancy
health checks
multi-region endpoints
private DNS dependencies
resolver redundancy
backup/export of critical DNS configuration
```

---

## 155. DNS Backup

DNS configuration should be reproducible.

Possible approaches:

```text
Terraform
Git
DNS zone exports
infrastructure-as-code
```

Do not rely only on manual console configuration.

---

## 156. DNS Incident: NXDOMAIN

### Symptom

```text
application says host not found
```

Check:

```bash
dig service.example.com
```

If:

```text
NXDOMAIN
```

check:

```text
record exists
correct zone
correct nameserver
delegation
typo
negative cache
```

---

## 157. DNS Incident: SERVFAIL

Check:

```bash
dig service.example.com
dig +trace service.example.com
```

Then investigate:

```text
authoritative server
DNSSEC
upstream resolver
network
zone configuration
```

---

## 158. DNS Incident: Timeout

Check:

```text
/etc/resolv.conf
DNS server reachability
UDP 53
TCP 53
NetworkPolicy
Security Group
NACL
CoreDNS
upstream resolver
```

---

## 159. DNS Incident: Internal Name Fails

Example:

```text
db.internal.example.com
```

Check:

```text
private hosted zone
VPC association
resolver rules
on-prem forwarding
Route 53 Resolver
```

---

## 160. DNS Incident: External Name Fails in EKS

If:

```text
kubernetes.default
```

works but:

```text
example.com
```

fails:

```text
CoreDNS Kubernetes plugin likely works
upstream forwarding likely needs investigation
```

Check CoreDNS configuration and upstream resolver reachability.

---

## 161. DNS Incident: Internal Kubernetes Name Fails

If:

```text
example.com
```

works but:

```text
cart.default.svc.cluster.local
```

fails:

```text
CoreDNS Kubernetes plugin
kube-dns Service
EndpointSlices
CoreDNS Pods
API connectivity
```

become key investigation points.

---

## 162. DNS Incident: Intermittent Resolution

Possible causes:

```text
one unhealthy CoreDNS replica
one broken node
packet loss
upstream instability
cache inconsistencies
multiple resolver paths
DNSSEC validation
```

Compare repeated queries against specific resolvers.

---

## 163. DNS Incident: Only One Node Fails

Check:

```text
node DNS configuration
NodeLocal DNSCache
iptables/eBPF
network path
CNI
CoreDNS connectivity
node-local resolver
```

Compare with a healthy node.

---

## 164. DNS Incident: Only One Pod Fails

Check:

```bash
kubectl exec -it <pod> -- cat /etc/resolv.conf
kubectl exec -it <pod> -- nslookup kubernetes.default
```

Compare:

```text
namespace
dnsPolicy
dnsConfig
NetworkPolicy
node
```

---

## 165. `dnsPolicy`

Kubernetes Pods commonly use:

```yaml
dnsPolicy: ClusterFirst
```

This causes cluster DNS to be used for normal Pod DNS resolution.

Other policies include:

```text
Default
ClusterFirstWithHostNet
None
```

Use the appropriate policy for the workload.

---

## 166. `hostNetwork`

Pods using:

```yaml
hostNetwork: true
```

have different networking/DNS behavior.

`ClusterFirstWithHostNet` may be required for expected Kubernetes DNS behavior in certain configurations.

---

## 167. Pod `dnsConfig`

Kubernetes allows customization of DNS configuration.

Example:

```yaml
dnsConfig:
  options:
    - name: ndots
      value: "2"
```

Use carefully because global assumptions about DNS search behavior can affect application compatibility.

---

## 168. DNS and Init Containers

An init container may fail because an external dependency cannot resolve.

Example:

```text
init container
   |
DNS
   |
database.example.com
```

Therefore startup failures may be DNS failures rather than image or Kubernetes scheduling failures.

---

## 169. DNS and Service Discovery

Microservices commonly use:

```text
service.namespace.svc.cluster.local
```

rather than hard-coded Pod IPs.

This provides:

```text
stable naming
dynamic backend discovery
```

---

## 170. DNS and Pod IPs

Pod IPs are generally ephemeral.

Avoid configuration such as:

```text
cart → 10.244.2.17
```

Use:

```text
cart.default.svc.cluster.local
```

for Service-based discovery.

---

## 171. Headless Services

A headless Service:

```yaml
clusterIP: None
```

can provide DNS records for individual Pods/endpoints instead of a single virtual Service IP.

This is useful for systems requiring direct endpoint discovery.

---

## 172. StatefulSet DNS

Stateful workloads can use stable DNS identities.

Conceptually:

```text
db-0
db-1
db-2
```

can have stable DNS names under the governing headless Service.

This is important for clustered databases and stateful systems.

---

## 173. DNS and Service Mesh

A service mesh may add additional DNS/network behavior, but Kubernetes Service discovery remains a foundational mechanism.

Troubleshooting should identify whether failure occurs:

```text
DNS
sidecar
proxy
Service
application
```

---

## 174. DNS and Observability

Monitor DNS separately.

Useful dashboards:

```text
CoreDNS QPS
CoreDNS latency
SERVFAIL
NXDOMAIN
upstream errors
CoreDNS CPU
CoreDNS memory
DNS timeout rate
```

---

## 175. DNS SLO

For production DNS, define measurable targets such as:

```text
availability
p95/p99 latency
error rate
resolution success rate
```

Do not monitor only application-level symptoms.

---

## 176. DNS Dependency Mapping

For RoboShop, document:

```text
frontend
 |
cart
 |
redis

catalog
 |
mongodb

order
 |
rabbitmq
```

and external dependencies:

```text
Route 53
AWS APIs
ECR
monitoring endpoints
external APIs
```

Every hostname is a DNS dependency.

---

## 177. RoboShop DNS Flow

Example:

```text
frontend Pod
    |
    | DNS: cart
    v
CoreDNS
    |
    v
cart Service
```

Then:

```text
cart Pod
    |
    | DNS: redis
    v
CoreDNS
    |
    v
redis Service
```

---

## 178. RoboShop External DNS

For an external API:

```text
RoboShop Pod
    |
    v
CoreDNS
    |
    v
AWS/VPC Resolver
    |
    v
Internet DNS
    |
    v
External authoritative DNS
```

---

## 179. RoboShop DNS Failure Example

Symptom:

```text
frontend cannot reach cart
```

Test:

```bash
kubectl exec -it <frontend-pod> -- nslookup cart
```

If DNS fails:

```text
do not immediately troubleshoot TCP
```

Fix the DNS layer first.

---

## 180. RoboShop DNS Failure Example

Symptom:

```text
catalog cannot connect to MongoDB
```

Test:

```bash
kubectl exec -it <catalog-pod> -- nslookup mongodb
```

If resolution works:

```bash
kubectl exec -it <catalog-pod> -- nc -vz mongodb 27017
```

Now the investigation moves from:

```text
DNS
```

to:

```text
TCP
```

---

## 181. DNS Debugging Layer Model

```text
Name
 |
v
Resolver
 |
v
IP
 |
v
Route
 |
v
TCP/UDP
 |
v
Port
 |
v
TLS
 |
v
Application
```

Never skip directly from:

```text
hostname failure
```

to:

```text
application code
```

without proving DNS.

---

## 182. DNS Troubleshooting Commands

```bash
cat /etc/resolv.conf

nslookup example.com
nslookup kubernetes.default

dig example.com
dig +short example.com
dig A example.com
dig AAAA example.com
dig NS example.com
dig SOA example.com
dig +trace example.com

getent hosts example.com
```

---

## 183. `getent hosts`

Useful because applications often use the system resolver stack.

Example:

```bash
getent hosts example.com
```

If:

```text
dig works
```

but:

```text
getent fails
```

investigate:

```text
NSS configuration
/etc/nsswitch.conf
system resolver
```

---

## 184. `/etc/nsswitch.conf`

Inspect:

```bash
cat /etc/nsswitch.conf
```

Look at:

```text
hosts:
```

The configuration determines how the system resolves hostnames across sources such as DNS and local files.

---

## 185. `/etc/hosts`

Inspect:

```bash
cat /etc/hosts
```

A static entry can override or affect resolution depending on NSS ordering.

This can cause:

```text
works on one server
fails on another
```

when `/etc/hosts` differs.

---

## 186. DNS vs `/etc/hosts`

If:

```text
/etc/hosts
```

contains:

```text
10.0.0.50 api.example.com
```

the system may resolve the name locally without querying DNS, depending on NSS configuration.

This is important when `dig` and application behavior differ.

---

## 187. DNS and Containers

Containers may receive their own:

```text
/etc/resolv.conf
/etc/hosts
```

configuration.

Always inspect from inside the actual container when debugging application DNS.

---

## 188. DNS and Sidecars

A sidecar can influence application networking but does not necessarily replace DNS.

If a sidecar is present, determine whether DNS traffic is:

```text
Pod → CoreDNS
```

or redirected through another component.

---

## 189. DNS and Proxies

An HTTP proxy may resolve the destination:

```text
client resolves host
```

or:

```text
proxy resolves host
```

depending on proxy behavior.

Therefore DNS troubleshooting must account for proxy configuration.

---

## 190. DNS and `NO_PROXY`

In Kubernetes, incorrect:

```text
HTTP_PROXY
HTTPS_PROXY
NO_PROXY
```

configuration can create confusing connectivity symptoms.

Internal names should generally bypass an external proxy where appropriate.

---

## 191. DNS and AWS PrivateLink

PrivateLink endpoints can provide private DNS names for AWS services or supported endpoint configurations.

The client may resolve the service name to private addresses within the VPC.

---

## 192. DNS and VPC Endpoints

Using VPC endpoints can reduce dependency on:

```text
NAT Gateway
Internet
```

for supported AWS services.

DNS configuration determines whether applications use the private endpoint.

---

## 193. DNS and ECR

EKS workloads pulling images from ECR depend on resolving AWS service endpoints.

In private clusters, VPC endpoints and DNS configuration are critical.

---

## 194. DNS and S3

Private AWS architectures may use S3 VPC endpoints.

Correct DNS and endpoint configuration can keep traffic within the intended AWS network path.

---

## 195. DNS and Private EKS

A private EKS architecture may involve:

```text
Private subnets
 |
CoreDNS
 |
VPC Resolver
 |
AWS service endpoints
 |
VPC endpoints
```

DNS becomes a foundational dependency for cluster operations.

---

## 196. Production DNS Architecture

A representative EKS architecture:

```text
                         Internet
                            |
                         Route 53
                            |
                           ALB
                            |
                           EKS
                            |
                  +---------+---------+
                  |                   |
               Pod A               Pod B
                  |                   |
                  +---------+---------+
                            |
                        CoreDNS
                            |
                    VPC DNS Resolver
                            |
                +-----------+-----------+
                |                       |
             Route 53              On-prem DNS
             Private               via Resolver
              Zone                  Endpoint
```

---

## 197. DNS Trust Boundaries

Typical boundaries:

```text
Internet
  |
Public DNS
  |
AWS edge
  |
VPC
  |
Private DNS
  |
EKS
  |
CoreDNS
```

Each boundary requires appropriate access and security controls.

---

## 198. DNS High Availability

Production DNS should avoid single points of failure.

For Kubernetes:

```text
multiple CoreDNS replicas
```

For authoritative DNS:

```text
multiple authoritative nameservers
```

For AWS:

```text
managed Route 53 infrastructure
```

plus appropriate regional/endpoint design.

---

## 199. DNS Capacity Planning

Plan for:

```text
queries per second
cache hit ratio
Pod count
service count
deployment frequency
autoscaling
node count
external dependency count
```

DNS load often increases with microservice count.

---

## 200. DNS Failure Domain

A single DNS failure can affect:

```text
all applications
all namespaces
all nodes
all services
```

Therefore DNS should be treated as platform infrastructure.

---

## 201. DNS Change Management

Before changing DNS:

```text
validate record
validate target
validate TTL
validate health
validate routing
validate rollback
```

For production:

```text
peer review
IaC
Git history
automated validation
monitoring
```

are preferred.

---

## 202. DNS Rollback

Keep the previous record value.

Example:

```text
Old:
api.example.com → ALB-A

New:
api.example.com → ALB-B
```

Rollback:

```text
api.example.com → ALB-A
```

Remember DNS caches may delay rollback visibility.

---

## 203. DNS Production Checklist

```text
[ ] Understand DNS hierarchy
[ ] Understand root
[ ] Understand TLD
[ ] Understand authoritative DNS
[ ] Understand recursive DNS
[ ] Understand stub resolvers
[ ] Understand FQDN
[ ] Understand zones
[ ] Understand delegation
[ ] Understand caching
[ ] Understand TTL
[ ] Understand negative caching
[ ] Understand NXDOMAIN
[ ] Understand SERVFAIL
[ ] Understand UDP 53
[ ] Understand TCP 53
[ ] Understand DoT
[ ] Understand DoH
[ ] Understand DNSSEC
[ ] Know dig
[ ] Know nslookup
[ ] Know getent
[ ] Know resolv.conf
[ ] Know nsswitch.conf
[ ] Understand CoreDNS
[ ] Understand Kubernetes DNS
[ ] Understand ndots
[ ] Understand NodeLocal DNSCache
[ ] Understand Route 53
[ ] Understand private hosted zones
[ ] Understand Resolver endpoints
[ ] Understand ExternalDNS
[ ] Understand DNS security
[ ] Understand DNS monitoring
[ ] Understand DNS disaster recovery
```

---

## 204. Interview: What Is DNS?

DNS is a distributed hierarchical naming system that maps domain names to resource records such as IP addresses and service information.

---

## 205. Interview: What Is a Recursive Resolver?

A recursive resolver obtains DNS answers on behalf of clients, following referrals and caching results.

---

## 206. Interview: What Is an Authoritative Server?

A DNS server that is authoritative for a zone and serves the zone's configured DNS records.

---

## 207. Interview: Recursive vs Authoritative?

```text
Recursive:
find the answer

Authoritative:
provide authoritative data for the zone
```

---

## 208. Interview: What Is an FQDN?

A Fully Qualified Domain Name specifies the complete DNS name within the hierarchy.

Example:

```text
api.prod.example.com.
```

---

## 209. Interview: Why Does DNS Use UDP?

UDP has low overhead and is efficient for normal DNS queries.

---

## 210. Interview: Why Can DNS Use TCP?

For larger/truncated responses, zone transfers and cases where TCP is required.

---

## 211. Interview: What Is TTL?

TTL controls how long DNS data may be cached before it should be refreshed according to DNS caching behavior.

---

## 212. Interview: What Is NXDOMAIN?

It indicates that the queried DNS name does not exist.

---

## 213. Interview: NXDOMAIN vs NOERROR/NODATA?

```text
NXDOMAIN:
name does not exist

NOERROR with no requested record:
name exists, but requested type has no answer
```

---

## 214. Interview: What Is SERVFAIL?

The resolver could not successfully complete the query.

---

## 215. Interview: What Is DNSSEC?

DNSSEC provides cryptographic validation of DNS data authenticity and integrity.

It does not encrypt DNS traffic.

---

## 216. Interview: DNSSEC vs DoH?

```text
DNSSEC:
validates DNS data

DoH:
encrypts DNS transport inside HTTPS
```

---

## 217. Interview: What Is CoreDNS?

CoreDNS is a DNS server commonly deployed in Kubernetes to provide Service discovery and forward external DNS queries.

---

## 218. Interview: How Does Kubernetes DNS Work?

Typical flow:

```text
Pod
 ↓
CoreDNS
 ↓
Kubernetes API-derived service data
```

for cluster names, and:

```text
CoreDNS
 ↓
upstream resolver
```

for external names.

---

## 219. Interview: What Is `ndots:5`?

It controls resolver behavior regarding whether a name is considered sufficiently dotted to be treated as absolute before search-domain expansion.

Kubernetes commonly uses `ndots:5`.

---

## 220. Interview: How Do You Check Pod DNS?

```bash
kubectl exec -it <pod> -- cat /etc/resolv.conf
kubectl exec -it <pod> -- nslookup kubernetes.default
```

---

## 221. Interview: How Do You Troubleshoot CoreDNS?

Check:

```bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system kube-dns
kubectl get endpointslice -n kube-system
kubectl logs -n kube-system -l k8s-app=kube-dns
kubectl get configmap -n kube-system coredns -o yaml
```

---

## 222. Interview: How Do You Test DNS Directly?

```bash
dig example.com
```

For a specific resolver:

```bash
dig @10.0.0.2 example.com
```

---

## 223. Interview: How Do You Trace DNS Delegation?

```bash
dig +trace example.com
```

---

## 224. Interview: How Do You Test Reverse DNS?

```bash
dig -x 203.0.113.10
```

---

## 225. Interview: Why Can `dig` Work While the Application Fails?

The application may use:

```text
different resolver
NSS
/etc/hosts
proxy
DoH
custom DNS behavior
```

Use:

```bash
getent hosts name
```

and inspect the application's runtime environment.

---

## 226. Interview: Why Can Internal DNS Work While External DNS Fails?

Kubernetes CoreDNS may successfully answer Kubernetes records while its upstream forwarding path is broken.

---

## 227. Interview: Why Can External DNS Work While Kubernetes DNS Fails?

The upstream resolver may work while:

```text
CoreDNS Kubernetes plugin
kube-dns Service
EndpointSlices
CoreDNS Pods
```

are unhealthy.

---

## 228. Interview: What Is a Private Hosted Zone?

A Route 53 hosted zone intended for private DNS resolution within associated VPCs and connected environments.

---

## 229. Interview: What Is Split-Horizon DNS?

The same DNS name or namespace can return different answers depending on the resolver/network context.

---

## 230. Interview: What Is ExternalDNS?

A Kubernetes controller/tool that can synchronize Kubernetes resources with DNS records in providers such as Route 53.

---

## 231. Interview: Why Is DNS Critical in EKS?

EKS workloads use DNS for:

```text
Kubernetes Services
AWS services
external APIs
databases
message brokers
```

A DNS outage can affect the entire platform.

---

## 232. Interview: How Do You Troubleshoot a DNS Timeout?

Check:

```text
resolv.conf
DNS Service
CoreDNS
UDP 53
TCP 53
NetworkPolicy
security groups
NACLs
VPC resolver
upstream DNS
```

---

## 233. Interview: How Do You Troubleshoot NXDOMAIN?

Verify:

```text
record
zone
delegation
nameserver
hostname
negative cache
```

---

## 234. Interview: How Do You Troubleshoot SERVFAIL?

Use:

```bash
dig
dig +trace
```

and investigate:

```text
authoritative DNS
DNSSEC
upstream failures
network
configuration
```

---

## 235. Interview: What Is the DNS Debugging Order?

```text
/etc/resolv.conf
 ↓
resolver reachability
 ↓
CoreDNS
 ↓
internal DNS
 ↓
external DNS
 ↓
authoritative DNS
```

---

## 236. Final DNS Mental Model

For an external application request:

```text
Application
    |
    v
OS/container resolver
    |
    v
CoreDNS / local resolver
    |
    v
Recursive resolver
    |
    v
Root
    |
    v
TLD
    |
    v
Authoritative DNS
    |
    v
IP address
    |
    v
TCP/UDP
    |
    v
TLS
    |
    v
Application
```

For Kubernetes internal service discovery:

```text
Application
    |
    v
Pod resolver
    |
    v
CoreDNS
    |
    v
Kubernetes Service data
    |
    v
Service name
    |
    v
Service IP / endpoints
```

For AWS private DNS:

```text
EKS Pod
   |
CoreDNS
   |
VPC Resolver
   |
Route 53 Private Hosted Zone
   |
Private endpoint
```

---

## 237. Final Production Rule

When a hostname fails, do not immediately test TCP.

First prove:

```text
name → IP
```

Then prove:

```text
IP → route
```

Then:

```text
IP:port → TCP
```

Then:

```text
TCP → TLS
```

Then:

```text
TLS → HTTP
```

This layered approach prevents incorrect root-cause assumptions.

---

## 238. Next File

The next planned file is:

```text
10-DNS-Records-and-Resolution.md
```

It will go deeper into:

```text
A
AAAA
CNAME
MX
NS
SOA
TXT
PTR
SRV
CAA
DNS query process
recursive resolution
iterative resolution
authoritative resolution
DNS caching
TTL
negative caching
DNSSEC
Route 53 records
alias records
weighted routing
latency routing
failover
geolocation
DNS troubleshooting
Kubernetes DNS
CoreDNS
EKS DNS
production DNS architectures
RoboShop DNS
interview questions
```

# End of 09-DNS-Fundamentals.md
