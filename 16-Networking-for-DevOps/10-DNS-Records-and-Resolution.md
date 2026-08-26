# 16-Networking-for-DevOps
# 10-DNS-Records-and-Resolution

## 1. Purpose

This file goes deeper into DNS records and the complete DNS resolution process.

A DevOps engineer should be able to look at a DNS problem and determine:

```text
Which record is involved?
Who is authoritative?
Which resolver answered?
Was the answer cached?
Was the name delegated correctly?
Was the response NXDOMAIN, NOERROR, SERVFAIL or REFUSED?
Did the client query the expected DNS server?
Did Kubernetes/CoreDNS or Route 53 return the answer?
```

This file covers:

- DNS resource records
- DNS resolution
- recursive and iterative lookup
- authoritative resolution
- caching
- TTL
- negative caching
- DNS delegation
- DNSSEC
- Route 53
- AWS ALB/NLB DNS
- Kubernetes/CoreDNS
- EKS DNS
- ExternalDNS
- production troubleshooting
- RoboShop examples
- interview preparation

---

## 2. DNS Resource Records

DNS data is represented through:

```text
Resource Records
```

Common records:

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
```

A record normally contains:

```text
owner/name
TTL
class
type
RDATA
```

---

## 3. Generic DNS Record Format

Conceptually:

```text
name    TTL    class    type    data
```

Example:

```text
api.example.com. 300 IN A 203.0.113.10
```

Meaning:

```text
name  = api.example.com.
TTL   = 300
class = IN
type  = A
data  = 203.0.113.10
```

---

## 4. DNS Class

The Internet primarily uses:

```text
IN
```

which means:

```text
Internet
```

Other DNS classes exist historically, but DevOps engineers normally work with `IN`.

---

## 5. A Record

An A record maps a name to IPv4.

Example:

```text
api.example.com. 300 IN A 203.0.113.10
```

Query:

```bash
dig A api.example.com
```

---

## 6. Multiple A Records

A name can have multiple A records:

```text
api.example.com → 203.0.113.10
api.example.com → 203.0.113.11
api.example.com → 203.0.113.12
```

This can support basic distribution, but it is not equivalent to a full load balancer.

---

## 7. AAAA Record

AAAA maps a name to IPv6.

Example:

```text
api.example.com. 300 IN AAAA 2001:db8::10
```

Query:

```bash
dig AAAA api.example.com
```

---

## 8. A and AAAA Together

A dual-stack hostname can contain:

```text
A
AAAA
```

Example:

```text
api.example.com
    |
    +-- A    → IPv4
    |
    +-- AAAA → IPv6
```

Clients may select between address families according to operating-system and application behavior.

---

## 9. CNAME Record

CNAME maps one DNS name to another DNS name.

Example:

```text
www.example.com
       |
       v
app.example.net
```

Query:

```bash
dig CNAME www.example.com
```

---

## 10. CNAME Is a Name Alias

A CNAME target is another DNS name, not an IP address.

Incorrect conceptual use:

```text
CNAME → 203.0.113.10
```

Correct:

```text
CNAME → app.example.net.
```

---

## 11. CNAME Resolution

Suppose:

```text
www.example.com CNAME app.example.net
```

and:

```text
app.example.net A 203.0.113.20
```

The resolver follows the CNAME to obtain the final address.

---

## 12. CNAME Chain

DNS can contain chains such as:

```text
www.example.com
      |
    CNAME
      v
app.example.net
      |
    CNAME
      v
lb.example.org
      |
      v
A/AAAA
```

Long chains increase lookup complexity and should generally be avoided when unnecessary.

---

## 13. CNAME at Zone Apex

Traditional DNS rules do not allow a normal CNAME to coexist with other records at the same owner name.

This creates a problem for:

```text
example.com
```

because the zone apex normally requires records such as:

```text
SOA
NS
```

AWS Route 53 provides alias records for supported AWS resources to solve common apex-routing requirements.

---

## 14. Route 53 Alias

An alias can point a DNS name to supported AWS resources such as:

```text
ALB
NLB
CloudFront
S3 website endpoint
```

Conceptually:

```text
example.com
    |
 Alias
    v
ALB
```

Alias is an AWS DNS feature and is not the same protocol concept as a standard CNAME.

---

## 15. MX Record

MX records identify mail exchangers.

Example:

```text
example.com. 300 IN MX 10 mail.example.com.
```

The number:

```text
10
```

is the priority/preference.

Lower values have higher preference.

---

## 16. Multiple MX Records

Example:

```text
MX 10 mail-primary.example.com.
MX 20 mail-secondary.example.com.
```

Mail systems can prefer the lower preference value and use alternatives when appropriate.

---

## 17. NS Record

NS records identify authoritative nameservers.

Example:

```text
example.com. IN NS ns1.example.net.
example.com. IN NS ns2.example.net.
```

Query:

```bash
dig NS example.com
```

---

## 18. SOA Record

SOA contains zone-level information.

Query:

```bash
dig SOA example.com
```

Typical fields:

```text
MNAME
RNAME
SERIAL
REFRESH
RETRY
EXPIRE
MINIMUM
```

The precise operational interpretation of negative caching-related fields follows DNS standards and resolver behavior.

---

## 19. SOA Serial

The serial identifies the version of zone data.

When manually managed DNS zones are updated, the serial convention is commonly incremented.

Managed DNS providers may abstract this from operators.

---

## 20. SOA Refresh

Historically used by secondary DNS servers to determine when to check the primary/authoritative source for updated zone data.

Modern managed DNS systems can implement their own synchronization mechanisms.

---

## 21. SOA Retry

The retry interval is associated with how secondary servers retry after a failed refresh attempt.

---

## 22. SOA Expire

The expire value defines how long secondary servers may continue serving zone data without successfully refreshing it from the authoritative source, subject to DNS behavior.

---

## 23. SOA Minimum

The SOA minimum field has historically had multiple meanings.

Modern DNS negative caching uses the relevant SOA information to determine how long negative responses can be cached.

Do not simply interpret this field as:

```text
default TTL for every record
```

---

## 24. TXT Record

TXT stores textual DNS data.

Common uses:

```text
domain verification
SPF
DKIM
DMARC
certificate validation
service ownership
```

Query:

```bash
dig TXT example.com
```

---

## 25. TXT and Domain Verification

A provider may request:

```text
_acme-challenge.example.com TXT <token>
```

or another verification name.

The provider checks authoritative DNS for the expected value.

---

## 26. SPF

Modern SPF publication is commonly represented through TXT records.

Example conceptually:

```text
example.com TXT "v=spf1 ..."
```

Do not create an obsolete dedicated SPF record type for modern deployments.

---

## 27. DKIM

DKIM public keys are commonly published under:

```text
selector._domainkey.example.com
```

as TXT records.

---

## 28. DMARC

DMARC policy is commonly published at:

```text
_dmarc.example.com
```

as TXT.

---

## 29. PTR Record

PTR provides reverse mapping.

Example:

```text
203.0.113.10
      |
      v
api.example.com
```

Query:

```bash
dig -x 203.0.113.10
```

---

## 30. Reverse DNS Namespace

IPv4 reverse DNS uses:

```text
in-addr.arpa
```

For:

```text
203.0.113.10
```

the reverse lookup name is:

```text
10.113.0.203.in-addr.arpa.
```

---

## 31. IPv6 Reverse DNS

IPv6 reverse DNS uses:

```text
ip6.arpa
```

The address is represented in reversed hexadecimal nibble form.

Use:

```bash
dig -x <IPv6-address>
```

rather than constructing the reverse name manually unless needed for advanced debugging.

---

## 32. SRV Record

SRV records advertise service endpoints.

Format conceptually:

```text
_service._proto.name
priority weight port target
```

Example:

```text
_ldap._tcp.example.com
```

can identify an LDAP service.

---

## 33. SRV Priority

Lower priority values are preferred.

Example:

```text
10
20
```

means the `10` endpoint is preferred over `20` when both are usable.

---

## 34. SRV Weight

Weight can distribute traffic among endpoints with the same priority.

It is a protocol-level mechanism, not equivalent to an application load balancer.

---

## 35. CAA Record

CAA means:

```text
Certification Authority Authorization
```

It specifies which certificate authorities are authorized to issue certificates for a domain.

Example conceptually:

```text
example.com CAA 0 issue "letsencrypt.org"
```

---

## 36. CAA and TLS

CAA can reduce unauthorized certificate issuance risk.

Before issuing a certificate, a CA can check the domain's CAA policy.

---

## 37. Wildcard DNS

A wildcard record can match otherwise-unmatched names under a DNS label.

Example:

```text
*.example.com
```

can provide an answer for:

```text
foo.example.com
bar.example.com
```

when no more specific matching record exists.

---

## 38. Wildcard Limitations

Wildcard matching is not:

```text
match everything everywhere
```

DNS wildcard rules are hierarchical and depend on existing names/delegations.

Test exact names with:

```bash
dig
```

rather than assuming wildcard behavior.

---

## 39. DNS Resolution Example

Suppose:

```text
www.example.com
```

needs an IPv4 address.

The client asks a recursive resolver:

```text
A www.example.com
```

The resolver checks its cache first.

---

## 40. Cache Hit

If cached:

```text
Resolver cache
      |
      v
www.example.com A 203.0.113.10
```

the resolver returns the result without walking the DNS hierarchy again.

This is a major reason DNS is fast.

---

## 41. Cache Miss

If not cached:

```text
Client
 |
Recursive Resolver
 |
Cache miss
 |
Root
 |
TLD
 |
Authoritative
```

The resolver obtains the answer and caches it according to applicable TTL.

---

## 42. Recursive Query

Client to recursive resolver:

```text
Client:
"Resolve www.example.com for me."
```

The resolver performs the necessary work.

---

## 43. Iterative Query

A DNS server can return a referral rather than the final answer.

Conceptually:

```text
Root:
"I do not have the final answer.
Ask these .com servers."
```

Then:

```text
TLD:
"Ask these example.com authoritative servers."
```

---

## 44. Root Lookup

For:

```text
www.example.com
```

the recursive resolver starts with root-server knowledge.

It asks the root infrastructure where to find:

```text
.com
```

---

## 45. TLD Lookup

The resolver then queries a `.com` nameserver.

The TLD response directs it toward the authoritative servers for:

```text
example.com
```

---

## 46. Authoritative Lookup

The resolver asks the authoritative server:

```text
A www.example.com
```

The authoritative response contains the configured record or appropriate negative answer.

---

## 47. Recursive Resolver Returns Result

The recursive resolver returns:

```text
203.0.113.10
```

to the client.

It also caches the response according to TTL and DNS rules.

---

## 48. Complete Resolution Flow

```text
Application
    |
    v
Stub Resolver
    |
    v
Recursive Resolver
    |
    +--> Cache?
          |
       +--+--+
       |     |
      Yes    No
       |     |
       |     v
       |    Root
       |     |
       |     v
       |    TLD
       |     |
       |     v
       | Authoritative
       |     |
       +-----+
             |
             v
          Answer
```

---

## 49. Delegation

Suppose:

```text
example.com
```

delegates:

```text
dev.example.com
```

to different nameservers.

The parent zone contains delegation information.

The child zone becomes authoritative for:

```text
dev.example.com
```

---

## 50. Glue Records

When delegated nameservers are inside the delegated domain, parent-side glue can be required.

Example:

```text
dev.example.com
```

delegated to:

```text
ns1.dev.example.com
```

The resolver needs an address for the nameserver itself.

Glue prevents a circular dependency.

---

## 51. DNS Referral

A referral contains information allowing the resolver to continue toward the authoritative servers.

The resolver does not necessarily receive the final A/AAAA answer at each hierarchy level.

---

## 52. Authoritative Answer vs Referral

Authoritative answer:

```text
aa flag
final record data
```

Referral:

```text
NS information
delegation path
```

Understanding this distinction is useful with:

```bash
dig +trace
```

---

## 53. `dig +trace`

Run:

```bash
dig +trace www.example.com
```

It shows the resolution path through:

```text
root
TLD
authoritative
```

This is one of the best tools for learning DNS resolution.

---

## 54. Querying a Resolver

```bash
dig @10.0.0.2 www.example.com
```

This asks the specified resolver.

Useful when debugging:

```text
corporate DNS
AWS VPC resolver
CoreDNS
public resolver
```

---

## 55. Querying an Authoritative Server

First:

```bash
dig NS example.com
```

Then:

```bash
dig @<authoritative-server> www.example.com
```

Compare with the recursive result.

---

## 56. Diagnosing Cache Differences

Suppose:

```text
Recursive resolver → old IP
Authoritative server → new IP
```

Possible explanation:

```text
recursive resolver still has cached data
```

Check:

```text
TTL
cache age
negative caching
resolver behavior
```

---

## 57. TTL Countdown

Suppose an authoritative record has:

```text
TTL = 300
```

A recursive resolver may return:

```text
TTL = 287
```

later because approximately 13 seconds have elapsed since caching.

The exact displayed TTL depends on resolver behavior.

---

## 58. DNS TTL and Application Cache

Changing DNS does not necessarily change application behavior immediately because applications may have their own:

```text
DNS caches
connection pools
HTTP connection reuse
service discovery caches
```

---

## 59. DNS Propagation Myth

DNS is not literally:

```text
change record
→ wait for Internet propagation
```

Instead:

```text
authoritative data changes
→ cached answers expire
→ resolvers refresh
→ clients receive new answers
```

Different caches can make the transition appear gradual.

---

## 60. Negative Caching Example

Suppose:

```text
api.example.com
```

does not exist.

A resolver receives:

```text
NXDOMAIN
```

It may cache the negative response.

If the record is created immediately afterward, some clients can continue receiving the negative result until the applicable negative-cache lifetime expires.

---

## 61. DNS Cache Flush

Different layers have different cache mechanisms.

Possible caches:

```text
browser
OS
systemd-resolved
nscd
CoreDNS
recursive resolver
application
```

Do not assume restarting the browser clears every DNS cache.

---

## 62. Linux DNS Cache

Check whether the host uses:

```text
systemd-resolved
```

for example:

```bash
resolvectl status
```

Availability depends on distribution and configuration.

---

## 63. `resolvectl query`

On systems using systemd-resolved:

```bash
resolvectl query example.com
```

can show resolver behavior.

---

## 64. Kubernetes DNS Cache

CoreDNS can cache responses through its `cache` plugin.

NodeLocal DNSCache can add another caching layer.

Therefore:

```text
authoritative DNS
```

can differ temporarily from:

```text
Pod resolution
```

because of intermediate caches.

---

## 65. DNS Record Selection

A resolver must determine which record set applies to the query.

For:

```text
api.example.com A
```

it looks for the matching owner name and record type.

CNAME behavior can redirect the resolution process.

---

## 66. CNAME and Other Records

At the same owner name, a normal CNAME has exclusivity requirements.

You should not configure:

```text
www.example.com CNAME app.example.net
www.example.com A 203.0.113.10
```

as ordinary DNS data simultaneously.

Use the correct DNS design or provider-specific alias mechanism.

---

## 67. Route 53 Alias vs CNAME

For an AWS ALB:

```text
app.example.com
```

can use a Route 53 alias to:

```text
ALB
```

Advantages include:

```text
zone apex support
AWS resource integration
no conventional CNAME requirement
```

Exact supported resources and behavior should be checked against current AWS documentation when designing infrastructure.

---

## 68. Route 53 Record Types

Route 53 supports common records including:

```text
A
AAAA
CAA
CNAME
MX
NS
PTR
SOA
SRV
TXT
```

and routing-policy features.

---

## 69. Simple Routing

Simple routing returns a single record set or simple set of values.

Use it for straightforward DNS mappings.

---

## 70. Weighted Routing

Weighted routing assigns weights to records.

Example:

```text
blue  = 90
green = 10
```

This can support controlled traffic distribution.

---

## 71. Weighted Routing Is Not Guaranteed Exact Percentage

DNS caching and resolver behavior mean weighted routing should not be interpreted as:

```text
exactly 90% of every request
```

It is DNS-level distribution.

---

## 72. Latency Routing

Latency-based routing can select an endpoint based on latency considerations across AWS regions.

Useful for:

```text
multi-region applications
```

---

## 73. Failover Routing

Failover routing supports:

```text
primary
secondary
```

with health-check-aware behavior where configured.

---

## 74. Geolocation Routing

Geolocation can route according to geographic location.

Example:

```text
India → ap-south
Europe → eu-west
US → us-east
```

Use fallback behavior carefully.

---

## 75. Geoproximity Routing

Geoproximity routing uses geographic proximity concepts and can support traffic shifting through bias.

Use it when the architecture requires geographic traffic management.

---

## 76. Multivalue Answer Routing

Multiple healthy values can be returned for a name.

It can provide a basic availability-oriented distribution mechanism.

It should not be treated as a replacement for an application load balancer.

---

## 77. Health Checks

Route 53 health checks can influence DNS routing policies.

Health checking can be based on supported endpoint protocols/configurations.

---

## 78. DNS Failover Caveat

DNS failover is affected by caching.

If a resolver cached the primary answer:

```text
primary healthy
```

and the primary becomes unhealthy, clients may continue using the cached result until the applicable TTL behavior permits a refresh.

---

## 79. DNS and ALB Hostname

AWS load balancers have AWS-managed DNS names.

Example conceptually:

```text
internal-my-alb-123456.ap-south-1.elb.amazonaws.com
```

Applications normally use Route 53 aliases/custom domains rather than hard-coding the load balancer's IP addresses.

---

## 80. ALB IP Addresses

Do not treat ALB IP addresses as permanent application configuration.

AWS can change underlying load-balancer infrastructure.

Use the DNS name/alias abstraction.

---

## 81. DNS and NLB

NLBs also expose DNS names.

Depending on architecture, Route 53 can map application names to NLB endpoints using supported alias behavior.

---

## 82. DNS and EKS Ingress

Typical:

```text
app.example.com
       |
       v
Route 53
       |
       v
ALB
       |
       v
Ingress
       |
       v
Service
       |
       v
Pod
```

DNS is only the first stage of the traffic path.

---

## 83. ExternalDNS Architecture

Typical:

```text
Git
 |
Argo CD
 |
Ingress
 |
ExternalDNS
 |
Route 53
```

ExternalDNS observes Kubernetes resources and manages DNS records.

---

## 84. ExternalDNS Ownership

Production ExternalDNS deployments should use ownership mechanisms to avoid collisions between multiple controllers or environments.

Use:

```text
TXT ownership
```

or supported controller configuration according to the chosen deployment.

---

## 85. ExternalDNS Domain Filtering

Limit ExternalDNS to approved domains/zones where possible.

Conceptually:

```text
--domain-filter=example.com
```

This reduces accidental changes outside the application's DNS boundary.

---

## 86. ExternalDNS IAM

Grant only required Route 53 permissions.

Avoid:

```text
AdministratorAccess
```

for a DNS controller.

Use least privilege.

---

## 87. DNS and GitOps

A production workflow can be:

```text
Developer
   |
Git
   |
CI
   |
Helm/Ingress change
   |
Argo CD
   |
EKS
   |
ExternalDNS
   |
Route 53
```

This creates an auditable path from code to DNS.

---

## 88. DNS and Terraform

Terraform can manage:

```text
Route 53 zones
records
resolver rules
resolver endpoints
VPC associations
```

A common boundary is:

```text
Terraform
→ infrastructure DNS

GitOps/ExternalDNS
→ application DNS
```

The exact ownership model should be explicit to avoid two systems managing the same record.

---

## 89. DNS Ownership Conflict

Bad design:

```text
Terraform → app.example.com
ExternalDNS → app.example.com
```

Both controllers can continuously change the same resource.

This creates configuration drift and operational confusion.

---

## 90. Production DNS Ownership

Document:

```text
Who owns the zone?
Who owns each record?
Which tool writes it?
Which repository contains desired state?
How is rollback performed?
```

---

## 91. DNSSEC Chain of Trust

DNSSEC validation uses a chain involving:

```text
root
 |
TLD
 |
zone
 |
record
```

Cryptographic signatures allow validating resolvers to verify signed DNS data.

---

## 92. DNSSEC DS Record

Delegation Signer records connect a child DNSSEC chain to its parent.

Conceptually:

```text
Parent
 |
DS
 |
Child DNSKEY
 |
RRSIG
 |
DNS data
```

---

## 93. DNSKEY

DNSKEY records publish public keys used in DNSSEC.

Resolvers use the relevant DNSSEC records to validate signatures.

---

## 94. RRSIG

RRSIG contains signatures over DNS resource-record sets.

A validating resolver can use these signatures to verify authenticity.

---

## 95. NSEC/NSEC3

DNSSEC can use denial-of-existence mechanisms such as:

```text
NSEC
NSEC3
```

to prove that a DNS name or record does not exist.

---

## 96. DNSSEC Failure

A DNSSEC validation problem can result in:

```text
SERVFAIL
```

even though the domain exists.

Compare:

```bash
dig +dnssec example.com
```

and test through another validating resolver when investigating.

---

## 97. DNS Query Packet

A DNS query contains fields such as:

```text
transaction ID
flags
question count
answer count
authority count
additional count
question section
```

The DNS header and sections are important for packet-level debugging.

---

## 98. DNS Response Packet

A response can include:

```text
header
question
answer
authority
additional
```

The authority section is particularly useful for understanding:

```text
delegation
SOA
negative answers
```

---

## 99. DNS Transaction ID

Traditional DNS uses a transaction identifier to match responses with queries.

Modern DNS security also relies on additional protections such as source-port randomization and DNSSEC where applicable.

---

## 100. DNS Flags

Important flags:

```text
QR
AA
TC
RD
RA
AD
CD
```

Examples:

```text
QR = response
AA = authoritative answer
TC = truncated
RD = recursion desired
RA = recursion available
AD = authenticated data
CD = checking disabled
```

---

## 101. Truncated DNS Response

A DNS response can indicate:

```text
TC
```

when the UDP response is truncated.

The resolver may retry using TCP where appropriate.

---

## 102. EDNS

EDNS extends DNS capabilities beyond the original DNS message limitations.

It can support:

```text
larger UDP payload sizes
DNSSEC-related signaling
additional options
```

---

## 103. EDNS and MTU

Large UDP DNS responses can interact badly with:

```text
MTU
fragmentation
firewalls
middleboxes
```

This can create intermittent DNS failures.

---

## 104. DNS Over UDP Fragmentation

A large UDP DNS response can be fragmented at the IP layer in some scenarios.

Middleboxes that drop fragments can cause:

```text
DNS timeout
SERVFAIL
```

This is one reason TCP fallback and modern DNS mechanisms matter.

---

## 105. DNS Packet Capture

Capture DNS:

```bash
sudo tcpdump -ni any port 53
```

More targeted:

```bash
sudo tcpdump -ni any host 10.0.0.2 and port 53
```

---

## 106. DNS Query Packet Analysis

Look for:

```text
source IP
destination DNS server
source port
destination port 53
query name
query type
response
latency
retransmission
```

---

## 107. DNS Query ID

In packet analysis, correlate:

```text
query
response
```

using the transaction identifier and flow information.

---

## 108. DNS TCP Capture

Capture TCP DNS:

```bash
sudo tcpdump -ni any tcp port 53
```

Useful when investigating:

```text
large responses
truncation
zone transfers
TCP fallback
```

---

## 109. DNS Resolution Timing

For a cache miss:

```text
Client → Resolver
Resolver → Root
Resolver → TLD
Resolver → Authoritative
Authoritative → Resolver
Resolver → Client
```

The client usually does not directly see every upstream query.

---

## 110. Recursive Resolver Cache as an Optimization

Without caching:

```text
every client query
→ hierarchy traversal
```

With caching:

```text
client query
→ resolver cache
→ answer
```

This dramatically reduces DNS infrastructure load.

---

## 111. Cache Invalidation

DNS does not have a universal global:

```text
flush all caches
```

Instead:

```text
TTL expires
resolver refreshes
```

Operational DNS changes must account for cached answers.

---

## 112. Low TTL Before Migration

A common migration strategy:

```text
Days before:
reduce TTL

Migration:
change record

After stabilization:
increase TTL
```

This should be planned rather than changed impulsively.

---

## 113. DNS Rollback Timing

Suppose:

```text
TTL = 3600
```

A rollback may not immediately reach every client because some resolvers still have cached answers.

For critical migrations, TTL planning is essential.

---

## 114. DNS Record Validation

Before production changes:

```bash
dig A app.example.com
dig AAAA app.example.com
dig CNAME app.example.com
dig NS example.com
dig SOA example.com
```

Use only the queries relevant to the record being changed.

---

## 115. DNS Consistency Check

Compare:

```bash
dig @resolver1 app.example.com
dig @resolver2 app.example.com
dig @authoritative app.example.com
```

If answers differ, investigate:

```text
cache
delegation
authoritative configuration
split DNS
resolver policy
```

---

## 116. DNS Delegation Troubleshooting

Use:

```bash
dig NS example.com
dig +trace example.com
```

Check:

```text
parent NS
child NS
glue
authoritative responses
```

---

## 117. Broken Delegation

Symptoms:

```text
SERVFAIL
intermittent resolution
some resolvers work
some fail
```

Possible causes:

```text
incorrect NS
missing glue
unreachable authoritative server
DNSSEC mismatch
```

---

## 118. Stale Delegation

If a domain's parent still delegates to old nameservers while the operator expects new nameservers:

```text
resolver
 |
parent
 |
old authoritative
```

The resolver will follow the delegation it receives.

Always inspect the parent delegation.

---

## 119. Authoritative Server Failure

If all authoritative servers are unreachable:

```text
recursive resolvers may fail after cached data expires
```

This is why authoritative DNS redundancy matters.

---

## 120. Multiple Authoritative Servers

Production zones normally use multiple authoritative servers.

Example:

```text
ns1
ns2
ns3
```

This improves resilience.

---

## 121. DNS Provider Failure

Managed DNS reduces operational burden, but production teams should still understand:

```text
provider dependency
registrar
delegation
authoritative service
health
```

Critical domains should have documented recovery procedures.

---

## 122. Registrar vs DNS Provider

Registrar:

```text
domain registration
```

DNS provider:

```text
DNS hosting
```

They can be the same company or different providers.

---

## 123. Nameserver Delegation at Registrar

The registrar's nameserver configuration tells the global DNS hierarchy which authoritative DNS servers are responsible for the domain.

Example:

```text
example.com
   |
   +-- ns1.provider.com
   +-- ns2.provider.com
```

---

## 124. Domain Registration vs Hosted Zone

Creating a Route 53 hosted zone does not by itself guarantee that a registered domain uses it.

The domain's delegation must point to the hosted zone's authoritative nameservers.

---

## 125. Route 53 Hosted Zone IDs

AWS APIs and Terraform commonly identify hosted zones using:

```text
hosted zone ID
```

Do not confuse:

```text
hosted zone ID
```

with:

```text
domain name
```

---

## 126. Route 53 Record Example

Terraform-style concept:

```hcl
resource "aws_route53_record" "app" {
  zone_id = aws_route53_zone.main.zone_id
  name    = "app.example.com"
  type    = "A"

  alias {
    name                   = aws_lb.app.dns_name
    zone_id                = aws_lb.app.zone_id
    evaluate_target_health = true
  }
}
```

The exact Terraform resource schema can vary by provider version; validate against the deployed provider version.

---

## 127. Route 53 Private Zone Example

Conceptually:

```hcl
resource "aws_route53_zone" "internal" {
  name = "internal.example.com"

  vpc {
    vpc_id = aws_vpc.main.id
  }
}
```

This creates private DNS scope for the associated VPC.

---

## 128. Kubernetes Service DNS

For a Service:

```text
cart
```

in namespace:

```text
roboshop
```

a common FQDN is:

```text
cart.roboshop.svc.cluster.local
```

---

## 129. Kubernetes DNS Search Domains

A Pod may have:

```text
search roboshop.svc.cluster.local svc.cluster.local cluster.local
```

so:

```text
cart
```

can resolve without typing the full FQDN.

---

## 130. Service DNS Resolution

Conceptually:

```text
cart.roboshop.svc.cluster.local
              |
              v
CoreDNS
              |
              v
Kubernetes Service
```

The Service then routes traffic to selected endpoints.

---

## 131. Headless Service DNS

For:

```yaml
clusterIP: None
```

DNS can expose endpoint addresses rather than a single virtual ClusterIP.

This is commonly used with:

```text
StatefulSet
database clusters
distributed systems
```

---

## 132. Pod DNS

Pod-specific DNS records can exist under Kubernetes DNS rules, depending on cluster configuration and hostname/subdomain settings.

Use Service DNS for normal service discovery.

---

## 133. Kubernetes ExternalName

An `ExternalName` Service can provide a DNS alias-like abstraction:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: payment
spec:
  type: ExternalName
  externalName: payment.example.com
```

This is DNS-based and does not create a normal proxying Service endpoint.

---

## 134. ExternalName Caveat

Applications expecting:

```text
ClusterIP
```

or normal endpoint behavior can behave differently with `ExternalName`.

Validate compatibility before using it in production.

---

## 135. CoreDNS Rewrite

CoreDNS can support advanced DNS rewriting through plugins/configuration.

Use carefully because rewrite rules can make resolution behavior harder to understand.

---

## 136. CoreDNS Stub Domains

CoreDNS can forward specific DNS zones to specific upstream servers.

Example concept:

```text
corp.example.com
      |
      v
corporate DNS
```

while other queries use normal upstream resolution.

---

## 137. Kubernetes and Corporate DNS

Hybrid EKS architecture:

```text
Pod
 |
CoreDNS
 |
+-- cluster.local → Kubernetes
|
+-- corp.example.com → on-prem DNS
|
+-- public names → AWS resolver
```

This is common in enterprise environments.

---

## 138. DNS Architecture for Private EKS

```text
                 On-Prem DNS
                     ^
                     |
             Resolver Endpoint
                     ^
                     |
Pod → CoreDNS → VPC Resolver
          |
          +---- cluster.local
          |
          +---- AWS/private DNS
```

---

## 139. DNS and Service Mesh

If using a service mesh:

```text
Application
 |
DNS
 |
Service
 |
Proxy
 |
Destination
```

Do not assume mesh telemetry replaces DNS troubleshooting.

---

## 140. DNS and HTTP Hostname

The DNS name can influence:

```text
HTTP Host
TLS SNI
certificate selection
ALB listener routing
```

Therefore DNS and TLS/HTTP configuration must be consistent.

---

## 141. DNS and TLS SNI

When accessing:

```text
api.example.com
```

the TLS client commonly sends SNI:

```text
api.example.com
```

The load balancer can use SNI to select the appropriate certificate/listener behavior.

DNS maps the name to the endpoint; TLS uses the same name for secure identity.

---

## 142. DNS Failure Can Look Like TLS Failure

If DNS returns the wrong endpoint:

```text
DNS
 ↓
wrong ALB
 ↓
certificate mismatch
```

The visible error may be TLS-related even though the root cause is DNS.

---

## 143. DNS Failure Can Look Like HTTP Failure

If DNS points to an old backend:

```text
DNS
 ↓
old ALB
 ↓
HTTP 404/503
```

The application may report an HTTP problem, but DNS is part of the root cause.

---

## 144. DNS and Blue-Green Deployments

Example:

```text
app.example.com
       |
       +-- blue
       +-- green
```

Use DNS routing or a load balancer to control the active target.

For fast request-level switching, load-balancer routing is generally more precise than DNS.

---

## 145. DNS and Canary Deployments

DNS weighted records can provide a coarse canary.

Example:

```text
95% → stable
5%  → canary
```

Use application/load-balancer controls when precise per-request or user-based routing is required.

---

## 146. DNS and Disaster Recovery

A DR design might use:

```text
Primary region
    |
Route 53
    |
health check
    |
Secondary region
```

The failover plan must include:

```text
DNS
application
database
secrets
network
certificates
```

---

## 147. DNS DR Testing

Do not assume failover works.

Regularly test:

```text
primary failure
DNS health check
secondary response
application readiness
rollback
```

---

## 148. DNS Monitoring With Prometheus

Possible metrics:

```text
CoreDNS request count
CoreDNS request duration
CoreDNS response codes
CoreDNS cache hits
CoreDNS forward failures
```

The exact metric names depend on the CoreDNS deployment/version.

---

## 149. DNS Alerts

Useful alert categories:

```text
CoreDNS unavailable
DNS latency high
SERVFAIL spike
NXDOMAIN anomaly
upstream timeout
CoreDNS CPU saturation
CoreDNS memory pressure
DNS query volume spike
```

---

## 150. DNS Logs and Privacy

DNS logs can contain:

```text
hostnames
internal service names
user-related destinations
```

Treat DNS logs as potentially sensitive operational data.

Apply appropriate:

```text
retention
access control
redaction
```

---

## 151. DNS Security Best Practices

```text
Use least privilege
Separate public/private zones
Use DNSSEC where appropriate
Restrict Route 53 changes
Audit DNS changes
Protect resolver endpoints
Avoid unrestricted recursion
Monitor DNS failures
Use approved DNS providers
Document ownership
```

---

## 152. Open Recursive Resolver Risk

An exposed recursive resolver can be abused for:

```text
DNS amplification
unauthorized recursion
traffic laundering
```

Do not expose unrestricted recursive DNS to the Internet.

---

## 153. DNS Amplification

Attackers can abuse small DNS queries to generate larger responses toward a victim.

Mitigations include:

```text
no open recursion
rate limiting
provider-level DDoS protection
network controls
```

---

## 154. DNS Cache Poisoning Defense

Use:

```text
trusted resolvers
source-port randomization
transaction ID randomization
DNSSEC validation
network security
```

---

## 155. DNS Tunneling

DNS can be abused to carry covert data.

Indicators may include:

```text
unusually long labels
high query volume
random-looking subdomains
high entropy
unusual TXT queries
```

Security monitoring can detect anomalous DNS behavior.

---

## 156. DNS Exfiltration

Attackers can encode data into DNS queries such as:

```text
encoded-data.attacker.example
```

Monitor:

```text
query length
frequency
entropy
rare domains
TXT usage
```

---

## 157. DNS Production Change Checklist

Before changing a record:

```text
[ ] Identify owner
[ ] Identify zone
[ ] Identify current record
[ ] Confirm record type
[ ] Check TTL
[ ] Check routing policy
[ ] Check health check
[ ] Validate target
[ ] Check dependencies
[ ] Plan rollback
[ ] Record change
[ ] Monitor after change
```

---

## 158. DNS Troubleshooting: Record Missing

Run:

```bash
dig A app.example.com
```

Then:

```bash
dig NS example.com
dig SOA example.com
```

Check:

```text
record exists?
correct zone?
correct nameserver?
delegation?
```

---

## 159. DNS Troubleshooting: Wrong IP

Compare:

```bash
dig @<resolver> app.example.com
dig @<authoritative> app.example.com
```

If authoritative is correct but resolver is old:

```text
cache/TTL
```

is a likely explanation.

---

## 160. DNS Troubleshooting: Wrong Nameserver

Run:

```bash
dig NS example.com
dig +trace app.example.com
```

Verify that the parent delegation points to the intended authoritative DNS provider.

---

## 161. DNS Troubleshooting: One Resolver Is Wrong

Run:

```bash
dig @resolver1 app.example.com
dig @resolver2 app.example.com
```

Possible causes:

```text
cache difference
split DNS
forwarding policy
resolver failure
```

---

## 162. DNS Troubleshooting: CoreDNS Wrong Answer

Check:

```bash
kubectl -n kube-system get configmap coredns -o yaml
kubectl -n kube-system logs -l k8s-app=kube-dns
```

Then query:

```bash
dig @<coredns-ip> service.namespace.svc.cluster.local
```

---

## 163. DNS Troubleshooting: CoreDNS Not Responding

Check:

```bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system kube-dns
kubectl get endpointslice -n kube-system
kubectl get events -n kube-system
```

Then inspect:

```bash
kubectl describe pod -n kube-system <coredns-pod>
```

---

## 164. DNS Troubleshooting: CoreDNS CrashLoopBackOff

Check:

```bash
kubectl logs -n kube-system <coredns-pod>
kubectl logs -n kube-system <coredns-pod> --previous
```

Investigate:

```text
Corefile syntax
upstream
resource limits
loop detection
network
configuration
```

---

## 165. DNS Troubleshooting: SERVFAIL in EKS

Check:

```text
CoreDNS logs
upstream resolver
VPC DNS
NetworkPolicy
security
route
DNSSEC
```

Use:

```bash
dig
dig +trace
```

where network access permits.

---

## 166. DNS Troubleshooting: NXDOMAIN in EKS

Check whether the expected Service exists:

```bash
kubectl get svc -A
```

Then:

```bash
nslookup <service>.<namespace>
```

If the Service does not exist, DNS is correctly reporting that the expected Kubernetes object is absent.

---

## 167. DNS Troubleshooting: External API

Example:

```text
orders.example.com
```

Test:

```bash
dig orders.example.com
```

Then:

```bash
nc -vz orders.example.com 443
```

Then:

```bash
curl -vk https://orders.example.com
```

This proves each layer sequentially.

---

## 168. DNS Troubleshooting: Private Route 53

Check:

```text
VPC association
hosted zone
record
resolver
route
```

A private hosted zone cannot normally be resolved from an unrelated network without appropriate DNS connectivity.

---

## 169. DNS Troubleshooting: Hybrid DNS

Check:

```text
Route 53 Resolver endpoint
security groups
subnet routing
resolver rules
on-prem DNS
VPN/Direct Connect
```

---

## 170. DNS Troubleshooting: Route 53 Record Changed but Not Visible

Check:

```text
authoritative response
TTL
recursive cache
negative cache
client cache
```

Do not immediately change the record repeatedly.

---

## 171. DNS Troubleshooting: CNAME Loop

Example:

```text
a.example.com CNAME b.example.com
b.example.com CNAME a.example.com
```

This creates a loop.

Resolvers will fail to complete resolution.

---

## 172. DNS Troubleshooting: Long CNAME Chain

Example:

```text
a
 ↓
b
 ↓
c
 ↓
d
 ↓
A
```

Long chains increase dependency and resolution complexity.

Prefer simpler DNS architecture where possible.

---

## 173. DNS Troubleshooting: Wrong Record Type

Example:

```text
application expects A
```

but only:

```text
AAAA
```

exists.

Test both:

```bash
dig A app.example.com
dig AAAA app.example.com
```

---

## 174. DNS Troubleshooting: IPv6 Only Failure

Test:

```bash
curl -4 -v https://app.example.com
curl -6 -v https://app.example.com
```

If only IPv6 fails:

```text
IPv6 routing
security
listener
DNS AAAA
```

should be investigated.

---

## 175. DNS Troubleshooting: DNS Works, TCP Fails

Example:

```text
dig → IP works
nc → timeout
```

DNS is probably not the current failing layer.

Move to:

```text
route
security group
NACL
NetworkPolicy
listener
```

---

## 176. DNS Troubleshooting: TCP Works, HTTP Fails

Example:

```text
nc → success
curl → HTTP 503
```

Move above transport:

```text
TLS
HTTP
ALB
application
backend
```

---

## 177. DNS Troubleshooting: TLS Certificate Mismatch

Check:

```text
DNS target
TLS SNI
certificate
ALB listener
```

Wrong DNS can send traffic to the wrong certificate endpoint.

---

## 178. DNS Troubleshooting: Application Uses Old IP

Check:

```text
application DNS cache
connection pool
OS cache
recursive resolver
TTL
```

Restarting an application can sometimes change behavior, but the real root cause should be identified rather than relying on restarts.

---

## 179. DNS Troubleshooting: DNS High Latency

Measure:

```bash
dig +stats example.com
```

Then compare:

```bash
dig @<resolver> example.com
dig @<authoritative> example.com
```

Determine whether latency occurs:

```text
client → resolver
resolver → upstream
authoritative
network
```

---

## 180. DNS Troubleshooting: Query Spike

Investigate:

```text
Pod deployment
application behavior
TTL changes
CoreDNS scaling
ndots
NodeLocal DNSCache
connection creation
```

---

## 181. Production DNS Runbook

### Step 1

Identify:

```text
hostname
record type
source workload
destination environment
```

### Step 2

Run:

```bash
dig <hostname>
```

### Step 3

Inspect:

```bash
cat /etc/resolv.conf
```

### Step 4

Query the configured resolver directly.

### Step 5

Compare with authoritative DNS.

### Step 6

For Kubernetes:

```bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system kube-dns
```

### Step 7

Check network controls.

### Step 8

Move to TCP only after DNS is proven.

---

## 182. RoboShop DNS Record Model

A typical production environment may use:

```text
shop.example.com
api.example.com
grafana.example.com
```

public DNS can map application domains to ALBs.

Inside EKS:

```text
frontend
cart
catalog
user
payment
shipping
```

use Kubernetes Service DNS.

---

## 183. RoboShop Internal DNS

Example:

```text
cart.roboshop.svc.cluster.local
catalog.roboshop.svc.cluster.local
user.roboshop.svc.cluster.local
```

The exact namespace depends on the deployment.

---

## 184. RoboShop External Entry

```text
User
 |
Route 53
 |
ALB
 |
Ingress
 |
frontend Service
 |
frontend Pod
```

---

## 185. RoboShop Microservice Dependency

Example:

```text
cart
 |
redis.roboshop.svc.cluster.local
```

If DNS fails:

```text
cart
```

may report:

```text
connection error
```

even though Redis itself is healthy.

---

## 186. RoboShop MongoDB Dependency

Example:

```text
catalog
 |
mongodb.roboshop.svc.cluster.local
```

Troubleshoot:

```text
DNS
→ TCP 27017
→ MongoDB
```

in that order.

---

## 187. RoboShop RabbitMQ Dependency

Example:

```text
shipping/order
 |
rabbitmq.roboshop.svc.cluster.local
```

Check:

```bash
nslookup rabbitmq
nc -vz rabbitmq 5672
```

---

## 188. Production GitOps DNS Flow

```text
Developer
   |
Git
   |
CI
   |
Helm/Ingress
   |
Argo CD
   |
EKS
   |
ExternalDNS
   |
Route 53
   |
Users
```

The desired DNS state can therefore be version-controlled indirectly through Kubernetes manifests.

---

## 189. DNS Repository Ownership

Example:

```text
gitops-repo/
├── applications/
├── environments/
├── ingress/
├── external-dns/
└── platform/
```

Keep DNS ownership explicit.

---

## 190. Production DNS Security

Use:

```text
IAM least privilege
Route 53 change auditing
private/public zone separation
approved DNS controllers
ExternalDNS domain filtering
ownership records
DNSSEC where applicable
resolver endpoint controls
```

---

## 191. Production DNS Reliability

Use:

```text
multiple CoreDNS replicas
CoreDNS resource planning
NodeLocal DNSCache where needed
multiple authoritative servers
Route 53 managed DNS
health checks
monitoring
tested DR
```

---

## 192. Interview: What Is an A Record?

An A record maps a DNS name to an IPv4 address.

---

## 193. Interview: What Is an AAAA Record?

An AAAA record maps a DNS name to an IPv6 address.

---

## 194. Interview: What Is CNAME?

A CNAME aliases one DNS name to another DNS name.

---

## 195. Interview: Why Can't a Normal CNAME Usually Be Used at the Zone Apex?

Because the zone apex needs records such as SOA and NS, while a normal CNAME has exclusivity requirements. AWS Route 53 alias records solve common AWS apex-routing scenarios.

---

## 196. Interview: What Is an Alias Record?

An AWS Route 53 feature that can map DNS names to supported AWS resources without using a conventional CNAME.

---

## 197. Interview: What Is MX?

MX identifies mail exchange servers and includes preference values.

---

## 198. Interview: What Is NS?

NS identifies authoritative nameservers for a zone/delegation.

---

## 199. Interview: What Is SOA?

SOA contains administrative and zone-control information such as serial and refresh-related values.

---

## 200. Interview: What Is PTR?

PTR provides reverse DNS mapping from an IP address to a name.

---

## 201. Interview: What Is SRV?

SRV publishes service discovery information including priority, weight, port and target.

---

## 202. Interview: What Is CAA?

CAA controls which certificate authorities are authorized to issue certificates for a domain.

---

## 203. Interview: How Does DNS Resolution Work?

```text
client
→ recursive resolver
→ root
→ TLD
→ authoritative
→ resolver cache
→ client
```

if the answer is not already cached.

---

## 204. Interview: Recursive vs Iterative?

Recursive means the resolver is responsible for obtaining the final answer.

Iterative means a server can return referrals and the resolver continues the lookup.

---

## 205. Interview: What Happens During a Cache Hit?

The recursive resolver returns the cached answer without querying the upstream hierarchy again.

---

## 206. Interview: What Is DNS Delegation?

Delegation transfers authority for a child namespace to specified authoritative nameservers.

---

## 207. Interview: What Are Glue Records?

Glue provides address information for delegated nameservers when necessary to resolve a nameserver whose name is within the delegated namespace.

---

## 208. Interview: What Does `dig +trace` Do?

It shows the iterative DNS resolution path starting from root information and following referrals toward the authoritative answer.

---

## 209. Interview: How Do You Find the Authoritative Servers?

```bash
dig NS example.com
```

Then query one directly.

---

## 210. Interview: How Do You Check Authoritative Data?

```bash
dig @<authoritative-server> app.example.com
```

---

## 211. Interview: Why Can Authoritative and Recursive Answers Differ?

Because the recursive resolver may still have cached data or be subject to different DNS policies.

---

## 212. Interview: What Is Negative Caching?

Caching of negative DNS responses such as NXDOMAIN according to DNS negative-caching rules.

---

## 213. Interview: Why Does a Newly Created DNS Record Sometimes Not Resolve Immediately?

Because intermediate resolvers or clients can have cached negative or old responses.

---

## 214. Interview: What Is Route 53 Weighted Routing?

A DNS routing policy that distributes responses among records according to configured weights.

---

## 215. Interview: Is Route 53 Weighted Routing Exact 90/10 Traffic?

No. DNS caching and resolver behavior mean it is not a precise per-request traffic split.

---

## 216. Interview: What Is Route 53 Failover Routing?

A routing policy that can select primary/secondary records based on health-check configuration.

---

## 217. Interview: What Is Split-Horizon DNS?

Different DNS answers are returned for the same namespace/name depending on the network or resolver context.

---

## 218. Interview: How Does Kubernetes DNS Work?

CoreDNS answers Kubernetes Service-related names and forwards external queries to upstream DNS.

---

## 219. Interview: How Do You Check Kubernetes DNS Configuration?

```bash
kubectl exec -it <pod> -- cat /etc/resolv.conf
```

---

## 220. Interview: How Do You Test a Kubernetes Service Name?

```bash
kubectl exec -it <pod> -- nslookup <service>
```

---

## 221. Interview: How Do You Check CoreDNS?

```bash
kubectl get pods -n kube-system
kubectl get svc -n kube-system kube-dns
kubectl get endpointslice -n kube-system
kubectl logs -n kube-system -l k8s-app=kube-dns
```

---

## 222. Interview: Why Does Kubernetes Use `kube-dns` Service Name With CoreDNS?

The Service name is retained for Kubernetes DNS compatibility even though CoreDNS is the DNS implementation.

---

## 223. Interview: What Is ExternalDNS?

A controller/tool that synchronizes Kubernetes resource information with external DNS providers such as Route 53.

---

## 224. Interview: How Do You Secure ExternalDNS?

Use:

```text
least-privilege IAM
domain filtering
ownership controls
dedicated service account
auditing
```

---

## 225. Interview: What Is a DNS Ownership Conflict?

Two independent systems attempt to manage the same DNS record.

Example:

```text
Terraform
+
ExternalDNS
```

Both write:

```text
app.example.com
```

This should be avoided.

---

## 226. Interview: How Do You Troubleshoot Wrong DNS Answers?

Compare:

```bash
dig @<recursive> name
dig @<authoritative> name
dig +trace name
```

Then determine whether the difference is caused by:

```text
cache
delegation
split DNS
record configuration
```

---

## 227. Interview: How Do You Troubleshoot SERVFAIL?

Start with:

```bash
dig name
dig +trace name
```

Then investigate:

```text
authoritative servers
DNSSEC
upstream DNS
network
resolver logs
```

---

## 228. Interview: How Do You Troubleshoot NXDOMAIN?

Verify:

```text
name spelling
zone
record
delegation
authoritative response
negative cache
```

---

## 229. Interview: How Do You Troubleshoot DNS Timeout in EKS?

Check:

```text
Pod /etc/resolv.conf
CoreDNS
kube-dns Service
EndpointSlices
NetworkPolicy
node networking
VPC resolver
UDP/TCP 53
upstream DNS
```

---

## 230. Interview: DNS Works but `curl` Fails. What Next?

DNS has likely completed successfully.

Move through:

```text
route
TCP
TLS
HTTP
application
```

---

## 231. Interview: `dig` Works but Application Cannot Resolve. Why?

Possible reasons:

```text
different resolver path
NSS
/etc/hosts
application DNS cache
proxy
custom resolver
```

Use:

```bash
getent hosts name
```

and inspect application configuration.

---

## 232. Interview: Why Can a Wrong DNS Record Cause a TLS Error?

DNS may send the client to an endpoint whose certificate does not match the requested hostname.

---

## 233. Interview: What Is DNSSEC?

DNSSEC uses cryptographic signatures to validate DNS data.

---

## 234. Interview: Does DNSSEC Encrypt DNS?

No.

DNSSEC provides authenticity/integrity validation, not transport confidentiality.

---

## 235. Interview: DoH vs DoT?

```text
DoH:
DNS over HTTPS

DoT:
DNS over TLS
```

Both encrypt DNS transport.

---

## 236. Interview: Why Does DNS Use TCP?

Large/truncated responses, zone transfers and protocol requirements can require TCP.

---

## 237. Interview: What Is EDNS?

An extension mechanism that expands DNS capabilities, including larger UDP payload support and additional options.

---

## 238. Interview: What Causes DNS Fragmentation Problems?

Possible causes include:

```text
large responses
MTU mismatch
fragment filtering
middleboxes
```

---

## 239. Interview: What Is a DNS Amplification Attack?

An attacker abuses recursive DNS infrastructure to generate larger responses toward a victim.

---

## 240. Interview: What Is DNS Tunneling?

Using DNS queries/responses as a covert channel to transport data.

---

## 241. Interview: How Would You Debug a DNS Problem in Production?

Strong answer:

```text
First identify the exact hostname and query type.
Then inspect the workload's resolver configuration.
Query the configured resolver with dig.
Compare the result with authoritative DNS.
For Kubernetes, test CoreDNS and the kube-dns Service.
Check NetworkPolicy and network reachability.
Check TTL/cache behavior.
Only after proving DNS should I move to TCP, TLS and application debugging.
```

---

## 242. Production DNS Decision Tree

```text
Hostname
   |
   v
Resolver configured?
   |
   +-- No → fix resolver configuration
   |
   v
Resolver reachable?
   |
   +-- No → network/security investigation
   |
   v
DNS response?
   |
   +-- NXDOMAIN → name/zone/delegation
   |
   +-- SERVFAIL → resolver/upstream/DNSSEC
   |
   +-- Timeout → resolver/network
   |
   v
Correct IP?
   |
   +-- No → record/cache/delegation
   |
   v
TCP works?
   |
   +-- No → routing/security/listener
   |
   v
TLS works?
   |
   +-- No → certificate/SNI/TLS
   |
   v
HTTP works?
   |
   +-- No → application/load balancer
```

---

## 243. Production DNS Mental Model

Remember:

```text
DNS does not "send traffic to the application."

DNS gives the client information needed to locate the application endpoint.
```

Then:

```text
DNS
 ↓
IP
 ↓
routing
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
application
```

---

## 244. Production DNS Checklist

```text
[ ] Know A
[ ] Know AAAA
[ ] Know CNAME
[ ] Know MX
[ ] Know NS
[ ] Know SOA
[ ] Know TXT
[ ] Know PTR
[ ] Know SRV
[ ] Know CAA
[ ] Know wildcard behavior
[ ] Know recursive resolution
[ ] Know iterative resolution
[ ] Know delegation
[ ] Know glue
[ ] Know caching
[ ] Know TTL
[ ] Know negative caching
[ ] Know NXDOMAIN
[ ] Know SERVFAIL
[ ] Know REFUSED
[ ] Know DNSSEC
[ ] Know EDNS
[ ] Know dig +trace
[ ] Know Route 53 routing policies
[ ] Know Route 53 aliases
[ ] Know CoreDNS
[ ] Know Kubernetes DNS
[ ] Know ExternalDNS
[ ] Know EKS DNS
[ ] Know DNS packet capture
[ ] Know DNS production troubleshooting
```

---

## 245. Next File

The next planned file is:

```text
11-HTTP-HTTPS-and-TLS.md
```

It will cover:

```text
HTTP architecture
HTTP/1.0
HTTP/1.1
HTTP/2
HTTP/3
methods
status codes
headers
cookies
sessions
keep-alive
connection reuse
chunked transfer
compression
proxies
reverse proxies
TLS
certificates
CA
certificate chain
SNI
ALPN
mTLS
TLS handshake
cipher suites
HTTPS troubleshooting
ALB
Nginx
Kubernetes Ingress
EKS
production incidents
RoboShop
interview questions
```

# End of 10-DNS-Records-and-Resolution.md
