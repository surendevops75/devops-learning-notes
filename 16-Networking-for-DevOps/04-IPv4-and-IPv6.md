# IPv4 and IPv6

## Purpose

IP addressing is the foundation of Layer 3 networking.

DevOps engineers working with Linux, AWS, Docker, Kubernetes and EKS
must understand how IP addresses are structured, assigned, routed and
consumed.

This file covers:

-   IPv4 structure
-   binary representation
-   network and host portions
-   subnet masks
-   CIDR
-   private/public addresses
-   special IPv4 ranges
-   loopback
-   link-local
-   APIPA
-   broadcast
-   multicast
-   IPv6 structure
-   IPv6 address types
-   global unicast
-   link-local
-   unique local addresses
-   multicast
-   IPv6 routing
-   dual stack
-   AWS IPv6
-   EKS IPv6
-   container IPv6
-   address troubleshooting
-   production design
-   interview preparation

------------------------------------------------------------------------

# 1. What Is an IP Address?

An IP address is a logical address used to identify a network endpoint
for Layer 3 communication.

Examples:

``` text
IPv4:
10.0.1.25

IPv6:
2001:db8:100::25
```

An IP address works together with:

``` text
prefix/subnet
routing table
network interface
gateway
```

An IP address by itself does not determine whether communication will
succeed.

------------------------------------------------------------------------

# 2. Why DevOps Engineers Need IP Knowledge

IP addressing appears everywhere:

``` text
AWS VPC
EC2
EKS
Pods
Services
ALB
NLB
Databases
Redis
RabbitMQ
Docker
CI runners
GitOps
Monitoring
```

Production incidents often involve:

``` text
wrong CIDR
IP exhaustion
overlapping networks
wrong route
wrong destination
unexpected source IP
IPv4/IPv6 mismatch
```

------------------------------------------------------------------------

# 3. IPv4

IPv4 uses:

``` text
32 bits
```

It is written as four decimal octets.

Example:

``` text
192.168.10.25
```

Each octet is:

``` text
0–255
```

because each octet contains 8 bits.

------------------------------------------------------------------------

# 4. IPv4 Binary Representation

Example:

``` text
192.168.1.10
```

Binary:

``` text
11000000.10101000.00000001.00001010
```

Each octet:

``` text
192 = 11000000
168 = 10101000
1   = 00000001
10  = 00001010
```

Total:

``` text
8 + 8 + 8 + 8 = 32 bits
```

------------------------------------------------------------------------

# 5. IPv4 Bit Values

Each octet has:

``` text
128 64 32 16 8 4 2 1
```

Example:

``` text
192
```

is:

``` text
128 + 64
```

therefore:

``` text
11000000
```

Understanding binary helps when calculating subnets.

------------------------------------------------------------------------

# 6. Network and Host Portions

An IPv4 address can be divided into:

``` text
Network portion
+
Host portion
```

The subnet prefix determines where the boundary is.

Example:

``` text
10.0.1.25/24
```

Conceptually:

``` text
Network = 10.0.1.0/24
Host    = 25
```

------------------------------------------------------------------------

# 7. Subnet Mask

A `/24` IPv4 prefix corresponds to:

``` text
255.255.255.0
```

Examples:

``` text
/8  → 255.0.0.0
/16 → 255.255.0.0
/24 → 255.255.255.0
```

The subnet mask determines which bits identify the network.

------------------------------------------------------------------------

# 8. CIDR

CIDR means:

``` text
Classless Inter-Domain Routing
```

Example:

``` text
10.0.0.0/16
```

The `/16` means:

``` text
16 bits = network prefix
16 bits = remaining address space
```

CIDR replaced the older class-based addressing model as the primary way
of expressing network prefixes.

------------------------------------------------------------------------

# 9. CIDR Prefix Length

IPv4 supports prefixes from:

``` text
/0
```

through:

``` text
/32
```

Examples:

``` text
/0  → entire IPv4 space
/8
/16
/24
/32 → one address
```

A larger prefix number means a smaller address range.

------------------------------------------------------------------------

# 10. Network Address

For:

``` text
192.168.1.25/24
```

network address:

``` text
192.168.1.0
```

broadcast:

``` text
192.168.1.255
```

host range:

``` text
192.168.1.1
through
192.168.1.254
```

This is the traditional IPv4 subnet model.

------------------------------------------------------------------------

# 11. Broadcast Address

IPv4 supports broadcast within applicable broadcast domains.

For:

``` text
192.168.1.0/24
```

the directed broadcast address is:

``` text
192.168.1.255
```

IPv6 does not use broadcast.

------------------------------------------------------------------------

# 12. Usable IPv4 Addresses

For a traditional IPv4 subnet:

``` text
network address
+
host addresses
+
broadcast address
```

For a `/24`:

``` text
256 total addresses
```

Traditionally:

``` text
1 network
1 broadcast
254 host addresses
```

Cloud providers can reserve additional addresses. AWS subnet capacity
therefore cannot be calculated simply by assuming all traditional host
addresses are usable.

------------------------------------------------------------------------

# 13. IPv4 `/32`

A `/32` represents exactly one IPv4 address.

Example:

``` text
10.0.1.25/32
```

Common uses include:

``` text
host routes
firewall rules
allow lists
specific endpoints
```

------------------------------------------------------------------------

# 14. IPv4 `/31`

A `/31` contains two addresses and is commonly used for point-to-point
links where both addresses can be used under RFC-defined semantics.

This is more common in routed network infrastructure than ordinary
application subnets.

------------------------------------------------------------------------

# 15. IPv4 `/30`

A `/30` contains:

``` text
4 addresses
```

Traditional subnet usage:

``` text
1 network
2 hosts
1 broadcast
```

Useful historically for point-to-point links.

------------------------------------------------------------------------

# 16. IPv4 `/29`

A `/29` contains:

``` text
8 addresses
```

Traditional host capacity:

``` text
6
```

Again, cloud providers may reserve addresses or impose their own subnet
rules.

------------------------------------------------------------------------

# 17. Private IPv4 Address Ranges

RFC 1918 private ranges:

``` text
10.0.0.0/8

172.16.0.0/12

192.168.0.0/16
```

These are commonly used inside:

``` text
enterprise networks
AWS VPCs
Docker networks
Kubernetes networks
home networks
```

------------------------------------------------------------------------

# 18. AWS Private IPv4

A common AWS VPC:

``` text
10.0.0.0/16
```

could be divided into:

``` text
10.0.1.0/24
10.0.2.0/24
10.0.11.0/24
10.0.12.0/24
```

For example:

``` text
Public subnet
10.0.1.0/24

Private application subnet
10.0.11.0/24

Database subnet
10.0.21.0/24
```

The exact design depends on architecture.

------------------------------------------------------------------------

# 19. Public IPv4

Public IPv4 addresses are globally routable addresses allocated by
appropriate Internet registries/providers.

AWS resources can use public IPv4 addresses depending on configuration.

Typical production pattern:

``` text
Internet
   |
Public ALB
   |
Private EKS
```

Avoid exposing internal workloads unnecessarily.

------------------------------------------------------------------------

# 20. Loopback

IPv4 loopback is:

``` text
127.0.0.0/8
```

Commonly:

``` text
127.0.0.1
```

It refers to the local host.

Example:

``` bash
curl http://127.0.0.1:8080
```

------------------------------------------------------------------------

# 21. Unspecified IPv4 Address

``` text
0.0.0.0
```

has special meaning depending on context.

For a server:

``` text
0.0.0.0:8080
```

often means:

``` text
listen on all IPv4 interfaces
```

In routing:

``` text
0.0.0.0/0
```

means the default IPv4 route.

These are not the same semantic use.

------------------------------------------------------------------------

# 22. Link-Local IPv4

IPv4 link-local addresses are:

``` text
169.254.0.0/16
```

They are used for local-link communication when appropriate
configuration mechanisms do not provide normal addresses.

Cloud environments also use portions of link-local space for special
services.

------------------------------------------------------------------------

# 23. APIPA

APIPA means:

``` text
Automatic Private IP Addressing
```

On many systems, an automatically assigned address in:

``` text
169.254.0.0/16
```

can indicate that normal DHCP configuration did not succeed.

In cloud environments, do not automatically interpret every
`169.254.x.x` address as APIPA; link-local addresses can serve other
specialized purposes.

------------------------------------------------------------------------

# 24. IPv4 Multicast

IPv4 multicast range:

``` text
224.0.0.0/4
```

Multicast allows one sender to reach multiple interested receivers.

Conceptually:

``` text
       Sender
          |
          v
      Multicast
       /  |  \
      v   v   v
     A    B    C
```

It differs from broadcast because receivers explicitly join multicast
groups.

------------------------------------------------------------------------

# 25. IPv4 Limited Broadcast

The limited broadcast address is:

``` text
255.255.255.255
```

It is used for local broadcast scenarios.

Routers do not normally forward it as a general routed destination.

------------------------------------------------------------------------

# 26. Special IPv4 Addresses

Important ranges:

``` text
0.0.0.0/8       special/unspecified contexts
10.0.0.0/8      private
127.0.0.0/8     loopback
169.254.0.0/16  link-local
172.16.0.0/12   private
192.168.0.0/16  private
224.0.0.0/4     multicast
240.0.0.0/4     reserved/future use
```

Do not memorize ranges without understanding their purpose.

------------------------------------------------------------------------

# 27. IPv4 Address Planning

Production address planning should consider:

``` text
current workloads
future workloads
availability zones
environments
clusters
shared services
databases
network connectivity
peering
Transit Gateway
VPN
Direct Connect
overlap avoidance
```

Poor planning creates expensive migration problems later.

------------------------------------------------------------------------

# 28. AWS VPC CIDR Planning

Example:

``` text
VPC
10.0.0.0/16
```

Possible design:

``` text
AZ-a:
  public    10.0.1.0/24
  private   10.0.11.0/20
  database  10.0.21.0/24

AZ-b:
  public    10.0.2.0/24
  private   10.0.12.0/20
  database  10.0.22.0/24

AZ-c:
  public    10.0.3.0/24
  private   10.0.13.0/20
  database  10.0.23.0/24
```

The actual sizing must be based on workload and AWS limits.

------------------------------------------------------------------------

# 29. Why CIDR Planning Matters in EKS

EKS can consume large numbers of IP addresses because of:

``` text
Nodes
Pods
Load balancers
Services
secondary interfaces
```

With VPC-native networking, Pod address capacity is particularly
important.

An undersized subnet can become a production scaling bottleneck.

------------------------------------------------------------------------

# 30. Overlapping CIDRs

Suppose:

``` text
VPC-A = 10.0.0.0/16
VPC-B = 10.0.0.0/16
```

Connecting them directly can create routing ambiguity.

This affects:

``` text
VPC peering
Transit Gateway
VPN
hybrid networking
multi-account architecture
```

Plan CIDRs globally, not one VPC at a time.

------------------------------------------------------------------------

# 31. IPv4 Subnetting

Subnetting divides a larger address block into smaller networks.

Example:

``` text
10.0.0.0/24
```

can be divided into four `/26` networks:

``` text
10.0.0.0/26
10.0.0.64/26
10.0.0.128/26
10.0.0.192/26
```

Each `/26` contains:

``` text
64 addresses
```

------------------------------------------------------------------------

# 32. Binary Subnetting

For:

``` text
10.0.0.0/26
```

the final octet has:

``` text
11000000
```

as the subnet mask.

That leaves:

``` text
6 host bits
```

Therefore:

``` text
2^6 = 64 addresses
```

Traditional host capacity:

``` text
62
```

------------------------------------------------------------------------

# 33. CIDR Capacity Formula

For IPv4:

``` text
total addresses = 2^(32-prefix)
```

Examples:

``` text
/24:
2^(32-24) = 256

/20:
2^(32-20) = 4096

/16:
2^(32-16) = 65536
```

Traditional usable host count is generally:

``` text
total - 2
```

for ordinary subnets, but cloud providers may reserve addresses.

------------------------------------------------------------------------

# 34. Prefix Comparison

Compare:

``` text
10.0.0.0/16
```

and:

``` text
10.0.0.0/24
```

`/16` is larger.

``` text
/16 → 65,536 IPv4 addresses
/24 → 256 IPv4 addresses
```

Larger prefix length means smaller subnet.

------------------------------------------------------------------------

# 35. Subnetting for Environments

A production organization may separate:

``` text
Dev
QA
Staging
Prod
Shared Services
Management
Security
```

Example:

``` text
Dev VPC     10.10.0.0/16
QA VPC      10.20.0.0/16
Prod VPC    10.30.0.0/16
Shared VPC  10.40.0.0/16
```

The exact scheme is organizational.

------------------------------------------------------------------------

# 36. IP Allocation Strategy

Reserve space for future growth.

Example:

``` text
10.30.0.0/16
```

could reserve:

``` text
10.30.0.0/20   platform
10.30.16.0/20  applications
10.30.32.0/20  data
10.30.48.0/20  future
```

Avoid consuming every address range immediately.

------------------------------------------------------------------------

# 37. IPv6

IPv6 uses:

``` text
128 bits
```

Example:

``` text
2001:db8:1234:5678::10
```

IPv6 was designed primarily to provide a vastly larger address space and
improve aspects of Internet addressing and routing.

------------------------------------------------------------------------

# 38. IPv6 Representation

IPv6 uses hexadecimal groups separated by colons.

Example:

``` text
2001:0db8:0000:0000:0000:0000:0000:0010
```

Can be compressed to:

``` text
2001:db8::10
```

------------------------------------------------------------------------

# 39. IPv6 Compression Rules

Leading zeros in each group can be removed.

Example:

``` text
0db8 → db8
```

A consecutive sequence of zero groups can be compressed using:

``` text
::
```

`::` can normally be used only once in an address.

------------------------------------------------------------------------

# 40. IPv6 Example

Full:

``` text
2001:0db8:0001:0000:0000:0000:0000:0025
```

Compressed:

``` text
2001:db8:1::25
```

The compression must be unambiguous.

------------------------------------------------------------------------

# 41. IPv6 Prefix

A common subnet size is:

``` text
/64
```

Example:

``` text
2001:db8:100:10::/64
```

This leaves:

``` text
64 bits
```

for interface addressing within the subnet.

------------------------------------------------------------------------

# 42. IPv6 Address Types

Important IPv6 categories:

``` text
Global Unicast
Link-Local
Unique Local
Multicast
Loopback
Unspecified
```

Unlike IPv4, IPv6 does not use broadcast.

------------------------------------------------------------------------

# 43. IPv6 Global Unicast

Global unicast addresses are intended for globally routable IPv6
communication.

Common range:

``` text
2000::/3
```

Example:

``` text
2001:db8:100::10
```

`2001:db8::/32` is reserved for documentation examples and is not a real
public allocation.

------------------------------------------------------------------------

# 44. IPv6 Link-Local

IPv6 link-local range:

``` text
fe80::/10
```

Link-local addresses are required for many IPv6 interface/network
operations.

They are only valid on the local link.

A link-local address is normally associated with an interface.

------------------------------------------------------------------------

# 45. IPv6 Unique Local Address

IPv6 Unique Local Addresses are:

``` text
fc00::/7
```

The commonly used locally assigned portion is:

``` text
fd00::/8
```

They are intended for local/private networking.

Conceptually similar in purpose to private IPv4 addressing, but not
identical in behavior or architecture.

------------------------------------------------------------------------

# 46. IPv6 Multicast

IPv6 multicast:

``` text
ff00::/8
```

IPv6 relies heavily on multicast.

Examples include Neighbor Discovery and router discovery mechanisms.

------------------------------------------------------------------------

# 47. IPv6 Loopback

IPv6 loopback:

``` text
::1
```

Equivalent conceptually to:

``` text
127.0.0.1
```

Example:

``` bash
curl http://[::1]:8080
```

------------------------------------------------------------------------

# 48. IPv6 Unspecified

IPv6 unspecified address:

``` text
::
```

It means no address is specified in contexts where that semantic is
valid.

It is not a normal destination address for general communication.

------------------------------------------------------------------------

# 49. IPv6 No Broadcast

IPv6 does not use broadcast.

Instead it uses:

``` text
multicast
```

This changes several network discovery mechanisms compared with IPv4.

------------------------------------------------------------------------

# 50. IPv6 Neighbor Discovery

IPv6 uses ICMPv6 Neighbor Discovery for functions including:

``` text
neighbor discovery
router discovery
address resolution
duplicate address detection
```

Important messages include:

``` text
Neighbor Solicitation
Neighbor Advertisement
Router Solicitation
Router Advertisement
```

------------------------------------------------------------------------

# 51. IPv6 Duplicate Address Detection

DAD helps determine whether an IPv6 address is already in use on a local
link before normal use.

This is an important part of IPv6 address configuration.

------------------------------------------------------------------------

# 52. SLAAC

SLAAC means:

``` text
Stateless Address Autoconfiguration
```

A host can construct an IPv6 address based on information advertised by
routers.

This reduces dependence on traditional DHCP-style address assignment for
many IPv6 designs.

------------------------------------------------------------------------

# 53. DHCPv6

DHCPv6 can also provide IPv6 configuration.

Organizations may use:

``` text
SLAAC
DHCPv6
or
a combination
```

depending on requirements.

Do not assume IPv6 configuration always works exactly like IPv4 DHCP.

------------------------------------------------------------------------

# 54. IPv4 vs IPv6

  Feature               IPv4         IPv6
  --------------------- ------------ --------------
  Address size          32-bit       128-bit
  Example               10.0.0.10    2001:db8::10
  Broadcast             Yes          No
  Multicast             Yes          Yes
  Loopback              127.0.0.1    ::1
  Link-local            169.254/16   fe80::/10
  Private/local         RFC1918      ULA
  Neighbor resolution   ARP          NDP/ICMPv6
  Common subnet         Variable     /64 common

------------------------------------------------------------------------

# 55. IPv4 and IPv6 Dual Stack

Dual stack means a system supports both:

``` text
IPv4
+
IPv6
```

Example:

``` text
Application
   |
   +-- IPv4 → 10.0.1.20
   |
   +-- IPv6 → 2001:db8::20
```

This can provide gradual migration.

------------------------------------------------------------------------

# 56. Why Dual Stack Can Be Complex

Supporting both protocols means troubleshooting:

``` text
IPv4 DNS
IPv6 DNS
IPv4 route
IPv6 route
IPv4 security
IPv6 security
IPv4 application behavior
IPv6 application behavior
```

An application may appear healthy over IPv4 while failing over IPv6.

------------------------------------------------------------------------

# 57. Happy Eyeballs

Modern clients can attempt IPv6 and IPv4 connectivity in ways designed
to reduce user-visible delays when one path is unavailable.

Therefore:

``` text
IPv6 exists
```

does not necessarily mean the application will always wait for IPv6
before trying IPv4.

Client behavior matters.

------------------------------------------------------------------------

# 58. DNS and IPv6

IPv4 commonly uses:

``` text
A
```

IPv6 uses:

``` text
AAAA
```

Example:

``` bash
dig A app.example.com
dig AAAA app.example.com
```

A hostname can return both.

------------------------------------------------------------------------

# 59. IPv6 Troubleshooting

Check addresses:

``` bash
ip -6 addr
```

Routes:

``` bash
ip -6 route
```

Neighbor information:

``` bash
ip -6 neigh
```

Test:

``` bash
ping6 <ipv6>
```

or:

``` bash
ping -6 <ipv6>
```

------------------------------------------------------------------------

# 60. IPv6 Curl

Use:

``` bash
curl -6 https://example.com
```

Force IPv4:

``` bash
curl -4 https://example.com
```

This is useful when diagnosing dual-stack behavior.

------------------------------------------------------------------------

# 61. IPv4 vs IPv6 Connectivity Test

Example:

``` bash
curl -4 -v https://example.com
curl -6 -v https://example.com
```

Interpretation:

``` text
IPv4 works
IPv6 fails
```

means the application may still appear healthy to IPv4-only clients.

------------------------------------------------------------------------

# 62. AWS IPv6

AWS VPCs can support IPv6 addressing.

An architecture can use:

``` text
VPC
 |
 +-- IPv4 CIDR
 |
 +-- IPv6 CIDR
```

Subnets can be configured for IPv6 addressing.

The exact routing and egress design must be planned deliberately.

------------------------------------------------------------------------

# 63. AWS IPv6 Egress

IPv6 workloads generally do not use NAT Gateway in the same way IPv4
private workloads do.

For IPv6 Internet access, an AWS architecture commonly uses an:

``` text
Internet Gateway
```

with appropriate IPv6 routes and security controls.

An egress-only Internet gateway is used for outbound-only IPv6 Internet
access from private IPv6 resources.

------------------------------------------------------------------------

# 64. Egress-Only Internet Gateway

An egress-only Internet gateway supports outbound IPv6 Internet
communication while preventing unsolicited inbound Internet-initiated
connections.

Conceptually:

``` text
Private IPv6 workload
        |
        v
Egress-only IGW
        |
        v
Internet
```

Security groups still matter.

------------------------------------------------------------------------

# 65. AWS IPv6 Address Planning

An IPv6-enabled VPC requires deliberate planning for:

``` text
VPC IPv6 CIDR
subnet IPv6 CIDRs
routing
security groups
NACLs
DNS
application compatibility
monitoring
```

Do not simply enable IPv6 without checking application and security
behavior.

------------------------------------------------------------------------

# 66. EKS IPv6

EKS supports IPv6 networking configurations.

IPv6 EKS changes important assumptions around:

``` text
Pod addresses
Service addressing
VPC networking
CNI behavior
DNS
external connectivity
```

The exact architecture depends on the EKS IPv6 mode and AWS CNI
configuration.

------------------------------------------------------------------------

# 67. EKS IPv6 Design Considerations

Before adopting IPv6:

``` text
application support
container image behavior
DNS
load balancers
security controls
NetworkPolicies
observability
external dependencies
database support
CI/CD tools
monitoring
```

Validate the complete path.

------------------------------------------------------------------------

# 68. Kubernetes Service and IPv6

Kubernetes supports IP families.

A Service can be:

``` text
IPv4
IPv6
dual-stack
```

The actual behavior depends on:

``` text
cluster configuration
CNI
Kubernetes version
service configuration
cloud integration
```

------------------------------------------------------------------------

# 69. Pod Addressing

With IPv4 Kubernetes:

``` text
Pod → IPv4 address
```

With IPv6:

``` text
Pod → IPv6 address
```

With dual-stack:

``` text
Pod may have both families
```

Do not assume every Kubernetes installation uses the same IP-family
model.

------------------------------------------------------------------------

# 70. IPv6 and NetworkPolicy

NetworkPolicy behavior must be tested with the selected CNI.

Security rules should explicitly account for:

``` text
IPv4
IPv6
```

if both are enabled.

A rule that protects IPv4 traffic does not automatically mean an IPv6
path has identical behavior.

------------------------------------------------------------------------

# 71. IPv4 NAT

IPv4 private workloads often use NAT for Internet egress.

Example:

``` text
Private subnet
      |
      v
NAT Gateway
      |
      v
Internet Gateway
      |
      v
Internet
```

IPv6 generally avoids the same address-translation requirement because
of its large address space.

------------------------------------------------------------------------

# 72. NAT Is Not Security by Itself

NAT is an address-translation mechanism.

Do not treat:

``` text
NAT
```

as equivalent to:

``` text
firewall
```

Security should be implemented through:

``` text
security groups
NACLs
NetworkPolicies
host controls
application authentication
```

as appropriate.

------------------------------------------------------------------------

# 73. Private IPv4 Does Not Mean Secure

A private address:

``` text
10.0.0.10
```

does not automatically mean:

``` text
trusted
secure
allowed
```

Security still depends on:

``` text
routing
security groups
NACLs
firewalls
NetworkPolicies
authentication
authorization
```

------------------------------------------------------------------------

# 74. IPv6 Global Address Does Not Mean Publicly Accessible

An IPv6 address can be globally routable, but access still depends on:

``` text
routing
security groups
NACLs
firewalls
application controls
```

Therefore:

``` text
global address
≠
automatically exposed service
```

------------------------------------------------------------------------

# 75. Source and Destination IP

For a connection:

``` text
Source:
10.0.1.20

Destination:
10.0.2.30
```

Routers forward traffic based primarily on destination addressing.

Security controls may inspect:

``` text
source
destination
protocol
port
```

------------------------------------------------------------------------

# 76. Source IP Preservation

Load balancers, proxies and NAT can change what the backend sees as the
source address.

Example:

``` text
Client
 |
ALB
 |
Backend
```

The backend may see an intermediary IP at the network layer while
application headers can carry original client information depending on
the architecture.

This distinction is critical for:

``` text
logging
security
rate limiting
auditing
```

------------------------------------------------------------------------

# 77. Pod IP and Source IP

In Kubernetes, source IP behavior can vary depending on:

``` text
Service type
kube-proxy behavior
CNI
load balancer configuration
externalTrafficPolicy
proxying
```

Do not assume that a Pod always sees the original client IP.

------------------------------------------------------------------------

# 78. `externalTrafficPolicy`

For Kubernetes Services of applicable types:

``` yaml
spec:
  externalTrafficPolicy: Local
```

can preserve client source IP in supported traffic paths by avoiding
certain cross-node forwarding behavior.

It can also affect:

``` text
load distribution
health checks
available endpoints
```

Use it deliberately.

------------------------------------------------------------------------

# 79. IPv4 Address Exhaustion

IPv4 space is limited.

Production symptoms:

``` text
new workload cannot receive IP
new ENI/IP allocation fails
Pods remain Pending
subnet has no free addresses
```

Monitor address utilization before capacity becomes critical.

------------------------------------------------------------------------

# 80. AWS Subnet IP Exhaustion

Check available addresses in AWS.

A subnet with insufficient free addresses can prevent:

``` text
EC2 placement
Pod IP allocation
load balancer ENI allocation
other network resource creation
```

Always include address capacity in capacity planning.

------------------------------------------------------------------------

# 81. Kubernetes IP Exhaustion

Possible causes:

``` text
small Pod CIDR
small subnet
high Pod density
ENI limits
CNI configuration
IPAM behavior
```

Symptoms:

``` text
Pod Pending
CNI errors
FailedCreatePodSandbox
```

------------------------------------------------------------------------

# 82. Overlapping Pod and VPC CIDRs

A poorly designed cluster may use address ranges that overlap with:

``` text
VPC
on-premises
VPN
other VPCs
partner networks
```

This becomes especially problematic when connecting networks.

Plan address ranges across the organization.

------------------------------------------------------------------------

# 83. VPN and CIDR Overlap

Example:

``` text
AWS VPC:
10.0.0.0/16

On-prem:
10.0.0.0/16
```

Routing cannot cleanly distinguish both destinations.

Possible solutions include:

``` text
renumbering
NAT
proxy architectures
network segmentation
```

Renumbering is often the cleanest long-term design but can be
operationally expensive.

------------------------------------------------------------------------

# 84. Transit Gateway and CIDRs

AWS Transit Gateway can connect multiple VPCs and on-premises networks.

This increases the importance of:

``` text
unique CIDRs
route tables
segmentation
propagation
centralized network design
```

A large enterprise should maintain an organization-wide IPAM strategy.

------------------------------------------------------------------------

# 85. AWS VPC IPAM

AWS provides IP Address Manager capabilities for organizing and
allocating IP address space.

The goal is to manage:

``` text
CIDR allocation
account
region
environment
utilization
```

at organizational scale.

------------------------------------------------------------------------

# 86. IPv4 and DNS Troubleshooting

If:

``` bash
dig A app.example.com
```

works but:

``` bash
dig AAAA app.example.com
```

returns nothing, the application may be IPv4-only.

If AAAA exists but:

``` bash
curl -6
```

fails, investigate:

``` text
IPv6 route
security
load balancer
application listener
upstream support
```

------------------------------------------------------------------------

# 87. IPv4 and IPv6 Application Listener

Check Linux listeners:

``` bash
ss -lntp
```

An application may listen on:

``` text
0.0.0.0:8080
```

or:

``` text
[::]:8080
```

depending on socket configuration and operating-system behavior.

Do not assume an IPv4 listener automatically means IPv6 behavior is
identical.

------------------------------------------------------------------------

# 88. IPv6 Socket Troubleshooting

Inspect:

``` bash
ss -lntp
ss -lnpt6
```

Then test:

``` bash
curl -6
```

and:

``` bash
curl -4
```

This quickly separates address-family problems.

------------------------------------------------------------------------

# 89. IPv4-Mapped IPv6 Addresses

Some systems can represent IPv4 endpoints in IPv6-related socket
contexts.

Example form:

``` text
::ffff:192.0.2.10
```

This is an IPv4-mapped IPv6 representation.

It should not be confused with a normal globally routable IPv6 address.

------------------------------------------------------------------------

# 90. IPv6 URL Syntax

Because IPv6 contains colons, URLs place literal IPv6 addresses inside
brackets.

Correct:

``` text
https://[2001:db8::10]:443
```

Not:

``` text
https://2001:db8::10:443
```

The brackets separate the address from the port.

------------------------------------------------------------------------

# 91. IPv6 Reverse DNS

IPv6 reverse DNS uses:

``` text
ip6.arpa
```

IPv4 reverse DNS uses:

``` text
in-addr.arpa
```

This is useful during DNS and infrastructure troubleshooting.

------------------------------------------------------------------------

# 92. Production IP Design Principles

Use:

``` text
non-overlapping CIDRs
predictable allocation
reserved growth capacity
environment separation
AZ-aware subnet design
documented ownership
centralized IPAM
IPv4/IPv6 strategy
```

Avoid:

``` text
random CIDRs
overlapping networks
using huge ranges without planning
exhausting subnets
mixing production and development unnecessarily
```

------------------------------------------------------------------------

# 93. Example Production AWS Address Plan

``` text
Organization
|
+-- DEV
|    VPC 10.10.0.0/16
|
+-- QA
|    VPC 10.20.0.0/16
|
+-- PROD
|    VPC 10.30.0.0/16
|
+-- SHARED
     VPC 10.40.0.0/16
```

Within production:

``` text
10.30.0.0/16
 |
 +-- Public-A      10.30.0.0/24
 +-- Public-B      10.30.1.0/24
 +-- Public-C      10.30.2.0/24
 |
 +-- App-A         10.30.16.0/20
 +-- App-B         10.30.32.0/20
 +-- App-C         10.30.48.0/20
 |
 +-- Data-A        10.30.64.0/24
 +-- Data-B        10.30.65.0/24
 +-- Data-C        10.30.66.0/24
```

This is an illustrative design, not a universal sizing recommendation.

------------------------------------------------------------------------

# 94. RoboShop IP Architecture

``` text
                         Internet
                             |
                          Public DNS
                             |
                             v
                            ALB
                             |
                        EKS VPC network
                             |
       +---------------------+---------------------+
       |                     |                     |
       v                     v                     v
    frontend               cart                catalog
     Pods                  Pods                  Pods
       |                     |                     |
       +---------------------+---------------------+
                             |
                       Internal services
                             |
                 +-----------+-----------+
                 |           |           |
                 v           v           v
               Redis      RabbitMQ      DB
```

IP planning must accommodate:

``` text
Nodes
Pods
Services
ALB
ENIs
databases
future scaling
```

------------------------------------------------------------------------

# 95. IPv6 Migration Strategy

A practical migration may follow:

``` text
1. Inventory IPv4 dependencies.
2. Identify IPv6 support gaps.
3. Plan IPv6 address space.
4. Enable in non-production.
5. Test DNS.
6. Test application listeners.
7. Test security controls.
8. Test observability.
9. Test CI/CD.
10. Test load balancers.
11. Validate dependencies.
12. Roll out gradually.
```

Do not treat IPv6 enablement as a one-click networking change.

------------------------------------------------------------------------

# 96. Dual-Stack Production Checklist

``` text
[ ] IPv4 addressing
[ ] IPv6 addressing
[ ] DNS A records
[ ] DNS AAAA records
[ ] IPv4 routes
[ ] IPv6 routes
[ ] Security groups
[ ] NACLs
[ ] NetworkPolicy
[ ] Load balancer support
[ ] TLS
[ ] Application listeners
[ ] Database connectivity
[ ] Monitoring
[ ] Logging
[ ] CI/CD
[ ] External dependencies
[ ] Failure testing
```

------------------------------------------------------------------------

# 97. Troubleshooting Decision Tree

``` text
Application cannot connect
          |
          v
Which address family?
     /          \
   IPv4         IPv6
    |             |
Check A        Check AAAA
    |             |
Check route    Check IPv6 route
    |             |
Check TCP      Check IPv6 TCP
    |             |
Check TLS      Check TLS
    \             /
      Application
```

------------------------------------------------------------------------

# 98. Troubleshooting: Wrong Subnet

Symptom:

``` text
host cannot reach expected network
```

Check:

``` bash
ip addr
ip route
```

Confirm:

``` text
IP
prefix
network
gateway
```

A wrong CIDR can make a destination appear local when it is actually
remote, or vice versa.

------------------------------------------------------------------------

# 99. Troubleshooting: IP Conflict

Symptoms:

``` text
intermittent connectivity
ARP changes
unexpected destination
duplicate address alerts
```

Investigate:

``` bash
ip neigh
tcpdump
arping
```

In cloud environments also inspect:

``` text
ENI/IP allocation
DHCP
cloud resource metadata
```

------------------------------------------------------------------------

# 100. Troubleshooting: IP Exhaustion

Symptoms:

``` text
Pod Pending
new instance/network interface fails
load balancer provisioning issues
CNI errors
```

Check:

``` text
subnet free IPs
VPC CIDRs
ENI limits
Pod density
CNI logs
```

------------------------------------------------------------------------

# 101. Troubleshooting: IPv6 Works Differently

If:

``` bash
curl -4 https://app.example.com
```

works but:

``` bash
curl -6 https://app.example.com
```

fails:

Investigate:

``` text
AAAA record
IPv6 route
security group
NACL
ALB/NLB support
application listener
upstream dependency
```

Do not disable IPv6 immediately without identifying the root cause.

------------------------------------------------------------------------

# 102. Troubleshooting: Pod Has IP but No Connectivity

A Pod having an IP proves only that address allocation occurred.

Still verify:

``` text
route
DNS
NetworkPolicy
security group
VPC routing
destination listener
```

Example:

``` bash
kubectl exec -it <pod> -- ip addr
kubectl exec -it <pod> -- ip route
kubectl exec -it <pod> -- cat /etc/resolv.conf
```

------------------------------------------------------------------------

# 103. Troubleshooting: Service Works by IP but Not Name

If:

``` bash
curl http://10.0.2.20:8080
```

works but:

``` bash
curl http://service.namespace.svc.cluster.local:8080
```

fails:

focus on:

``` text
DNS
CoreDNS
resolver configuration
Service name
namespace
```

This is an application-layer naming issue, not necessarily an IP routing
issue.

------------------------------------------------------------------------

# 104. Troubleshooting: DNS Returns Wrong IP

Check:

``` bash
dig app.example.com
dig +trace app.example.com
```

Verify:

``` text
authoritative DNS
record
TTL
Route 53 configuration
load balancer address
```

Also account for:

``` text
client cache
resolver cache
application cache
```

------------------------------------------------------------------------

# 105. Interview --- What Is IPv4?

> IPv4 is a 32-bit logical addressing protocol. Addresses are commonly
> written as four decimal octets, and CIDR prefixes determine the
> network boundary.

------------------------------------------------------------------------

# 106. Interview --- What Is CIDR?

> CIDR is a classless method of representing IP networks using a prefix
> length. For example, `10.0.0.0/16` means the first 16 bits identify
> the network.

------------------------------------------------------------------------

# 107. Interview --- How Many Addresses Are in /24?

``` text
2^(32-24)
= 2^8
= 256
```

Traditional usable host addresses:

``` text
254
```

Cloud platforms can reserve additional addresses.

------------------------------------------------------------------------

# 108. Interview --- How Many Addresses Are in /16?

``` text
2^(32-16)
= 2^16
= 65,536
```

------------------------------------------------------------------------

# 109. Interview --- Private IPv4 Ranges?

``` text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

These are defined by RFC 1918.

------------------------------------------------------------------------

# 110. Interview --- Why Is 127.0.0.1 Special?

> It is a loopback address used for communication with the local host.
> The entire `127.0.0.0/8` range is reserved for IPv4 loopback.

------------------------------------------------------------------------

# 111. Interview --- What Is 0.0.0.0?

It depends on context.

``` text
0.0.0.0/0
```

means default route.

``` text
0.0.0.0:8080
```

commonly means a server listens on all IPv4 interfaces.

------------------------------------------------------------------------

# 112. Interview --- What Is IPv6?

> IPv6 is a 128-bit Internet Protocol designed to provide a vastly
> larger address space and modernize IP networking. It uses hexadecimal
> notation and supports global unicast, link-local, multicast and
> unique-local addressing.

------------------------------------------------------------------------

# 113. Interview --- Why Does IPv6 Not Need Broadcast?

> IPv6 uses multicast and Neighbor Discovery mechanisms instead of the
> IPv4 broadcast model.

------------------------------------------------------------------------

# 114. Interview --- What Is IPv6 Link-Local?

> IPv6 link-local addresses use `fe80::/10` and are valid only on the
> local link. They are fundamental to IPv6 neighbor and router
> discovery.

------------------------------------------------------------------------

# 115. Interview --- What Is IPv6 ULA?

> Unique Local Addresses use `fc00::/7`, with `fd00::/8` commonly used
> for locally assigned addresses. They are intended for local/private
> IPv6 networking.

------------------------------------------------------------------------

# 116. Interview --- What Is Dual Stack?

> Dual stack means operating IPv4 and IPv6 simultaneously. It provides
> compatibility while organizations transition between address families.

------------------------------------------------------------------------

# 117. Interview --- A Record vs AAAA?

``` text
A     → IPv4
AAAA  → IPv6
```

Example:

``` bash
dig A app.example.com
dig AAAA app.example.com
```

------------------------------------------------------------------------

# 118. Interview --- What Happens If IPv4 and IPv6 Both Exist?

The client may select an address according to its resolver and
connection strategy. Modern clients can use mechanisms such as Happy
Eyeballs to reduce delays when one protocol path is unavailable.

------------------------------------------------------------------------

# 119. Interview --- Why Does CIDR Overlap Matter?

> Overlapping CIDRs create routing ambiguity when networks need to
> communicate. This is especially problematic with VPC peering, Transit
> Gateway, VPN and hybrid cloud connectivity.

------------------------------------------------------------------------

# 120. Interview --- Why Is EKS IP Planning Important?

> EKS workloads consume network addresses. With VPC-native networking,
> Pod IP allocation depends on VPC/subnet and CNI capacity. Poor CIDR
> planning can prevent cluster scaling even when CPU and memory are
> available.

------------------------------------------------------------------------

# 121. Interview --- How Do You Troubleshoot IPv4 Connectivity?

``` text
1. Check IP.
2. Check prefix.
3. Check route.
4. Check gateway.
5. Check DNS.
6. Check TCP/UDP.
7. Check security controls.
8. Check application.
```

Commands:

``` bash
ip addr
ip route
ip route get <destination>
ip neigh
nc -vz <host> <port>
curl
```

------------------------------------------------------------------------

# 122. Interview --- How Do You Troubleshoot IPv6 Connectivity?

``` text
ip -6 addr
ip -6 route
ip -6 neigh
ping -6
curl -6
```

Then inspect:

``` text
AAAA
IPv6 route
security
load balancer
application
```

------------------------------------------------------------------------

# 123. Production Best Practices

Use:

``` text
documented CIDR plan
non-overlapping ranges
reserved capacity
central IPAM
environment separation
AZ-aware subnetting
IPv6 strategy
monitoring
address utilization alerts
```

For EKS:

``` text
size subnets for Pod growth
understand ENI/IP limits
monitor CNI health
avoid overlapping enterprise networks
plan multi-cluster connectivity
```

------------------------------------------------------------------------

# 124. IPv4/IPv6 Security Best Practices

Never assume:

``` text
private = trusted
IPv6 = safe
NAT = firewall
IP = identity
```

Use:

``` text
security groups
NACLs
NetworkPolicies
firewalls
TLS
IAM
authentication
authorization
least privilege
```

Apply controls to both address families where both are enabled.

------------------------------------------------------------------------

# 125. Production Failure Scenario

### Incident

RoboShop frontend is reachable from IPv4 clients but unavailable to some
IPv6 clients.

Investigation:

``` text
1. dig A frontend.example.com
2. dig AAAA frontend.example.com
3. curl -4
4. curl -6
5. inspect ALB IPv6 configuration
6. inspect IPv6 routes
7. inspect security groups/NACLs
8. inspect target health
9. inspect application listener
```

Root cause should be identified at the failing layer rather than
assuming the application is broken.

------------------------------------------------------------------------

# 126. Production Failure Scenario --- EKS IP Exhaustion

### Symptom

``` text
Deployment scaled from 50 to 100 replicas.
New Pods remain Pending.
```

Check:

``` bash
kubectl describe pod <pending-pod>
kubectl get nodes -o wide
```

If CNI reports address allocation failure:

``` text
check subnet free addresses
check ENI limits
check instance type
check CNI configuration
check subnet expansion strategy
```

CPU may be available while networking capacity is exhausted.

------------------------------------------------------------------------

# 127. Production Failure Scenario --- Overlapping VPC

### Symptom

A newly connected VPC cannot reach an on-premises network.

Discovered:

``` text
AWS:
10.20.0.0/16

On-prem:
10.20.0.0/16
```

The problem is address-space overlap.

Long-term solution:

``` text
renumber
```

or use an explicitly designed translation/proxy architecture where
appropriate.

------------------------------------------------------------------------

# 128. Production Failure Scenario --- IPv6 Security Gap

A service was secured for IPv4 but accidentally became reachable over
IPv6.

Root cause:

``` text
IPv6 path/security rules were not reviewed
```

Lesson:

``` text
IPv4 security review
≠
IPv6 security review
```

When dual stack is enabled, audit both.

------------------------------------------------------------------------

# 129. Production Architecture --- IPv4 EKS

``` text
                       Internet
                           |
                       Route 53 A
                           |
                           v
                          ALB
                           |
                         IPv4
                           |
                           v
                         EKS
                           |
              +------------+------------+
              |                         |
           Pod IPv4                  Pod IPv4
              |                         |
              +------------+------------+
                           |
                        VPC IPv4
                           |
                      RDS / Redis
```

------------------------------------------------------------------------

# 130. Production Architecture --- IPv6 EKS Concept

``` text
                       Internet
                           |
                      Route 53 AAAA
                           |
                           v
                    IPv6-capable LB
                           |
                           v
                          EKS
                           |
                    IPv6 Pod network
                           |
                    IPv6 VPC routing
                           |
                    IPv6-compatible
                     dependencies
```

Exact AWS/EKS capabilities depend on the selected configuration and
service support.

------------------------------------------------------------------------

# 131. Production Architecture --- Dual Stack

``` text
                       DNS
                    /         \
                   A           AAAA
                   |             |
                   v             v
                 IPv4          IPv6
                   \             /
                    \           /
                       ALB/LB
                          |
                         EKS
                       /     \
                    IPv4     IPv6
                    Pods      Pods
```

The exact Kubernetes and AWS implementation must be validated for the
cluster version and networking configuration.

------------------------------------------------------------------------

# 132. Capacity Planning Checklist

``` text
[ ] VPC CIDR
[ ] subnet CIDRs
[ ] AZ distribution
[ ] node count
[ ] Pod count
[ ] Pod density
[ ] ENI limits
[ ] IP-per-ENI limits
[ ] load balancer requirements
[ ] database IP requirements
[ ] future growth
[ ] multi-cluster requirements
[ ] hybrid connectivity
[ ] IPv6 requirements
```

------------------------------------------------------------------------

# 133. Practical CIDR Exercise

Given:

``` text
10.0.0.0/24
```

divide into four equal networks.

Answer:

``` text
10.0.0.0/26
10.0.0.64/26
10.0.0.128/26
10.0.0.192/26
```

Each contains:

``` text
64 addresses
```

------------------------------------------------------------------------

# 134. Practical CIDR Exercise

Given:

``` text
10.0.0.0/16
```

how many `/24` networks fit?

Difference:

``` text
24 - 16 = 8
```

Therefore:

``` text
2^8 = 256
```

So 256 `/24` blocks fit inside the `/16`.

------------------------------------------------------------------------

# 135. Practical CIDR Exercise

Given:

``` text
10.0.10.0/24
```

what is the network address of:

``` text
10.0.10.150
```

Answer:

``` text
10.0.10.0
```

Broadcast:

``` text
10.0.10.255
```

Traditional host range:

``` text
10.0.10.1–10.0.10.254
```

------------------------------------------------------------------------

# 136. Practical CIDR Exercise

Given:

``` text
10.0.10.64/26
```

range:

``` text
10.0.10.64
through
10.0.10.127
```

Traditional host range:

``` text
10.0.10.65–10.0.10.126
```

Broadcast:

``` text
10.0.10.127
```

------------------------------------------------------------------------

# 137. Practical IPv6 Exercise

Compress:

``` text
2001:0db8:0000:0000:0000:0000:0000:0010
```

Answer:

``` text
2001:db8::10
```

------------------------------------------------------------------------

# 138. Practical IPv6 Exercise

Identify:

``` text
fe80::1234
```

It is:

``` text
IPv6 link-local
```

Identify:

``` text
::1
```

It is:

``` text
IPv6 loopback
```

Identify:

``` text
ff02::1
```

It is:

``` text
IPv6 multicast
```

------------------------------------------------------------------------

# 139. Command Reference

## IPv4

``` bash
ip -4 addr
ip -4 route
ip -4 neigh
```

## IPv6

``` bash
ip -6 addr
ip -6 route
ip -6 neigh
```

## DNS

``` bash
dig A example.com
dig AAAA example.com
```

## Connectivity

``` bash
ping <ipv4>
ping -6 <ipv6>
curl -4 https://example.com
curl -6 https://example.com
```

## Sockets

``` bash
ss -lnt
ss -lnt6
```

------------------------------------------------------------------------

# 140. Final Mental Model

IPv4:

``` text
32 bits
   ↓
CIDR prefix
   ↓
network
   ↓
host
   ↓
route
   ↓
gateway
```

IPv6:

``` text
128 bits
   ↓
prefix
   ↓
subnet
   ↓
interface address
   ↓
neighbor discovery
   ↓
route
```

Cloud:

``` text
VPC CIDR
   ↓
Subnet CIDR
   ↓
ENI/IP
   ↓
Route Table
   ↓
Security
   ↓
Workload
```

Kubernetes/EKS:

``` text
VPC
 ↓
Subnet
 ↓
ENI/IP capacity
 ↓
CNI
 ↓
Pod IP
 ↓
Service
 ↓
Ingress/Load Balancer
```

------------------------------------------------------------------------

# 141. Final Production Checklist

``` text
[ ] Understand IPv4 binary
[ ] Understand CIDR
[ ] Calculate subnet capacity
[ ] Know private IPv4 ranges
[ ] Know special IPv4 ranges
[ ] Understand loopback
[ ] Understand link-local
[ ] Understand multicast
[ ] Understand IPv6 notation
[ ] Know IPv6 address types
[ ] Understand NDP
[ ] Understand SLAAC
[ ] Understand dual stack
[ ] Understand A vs AAAA
[ ] Plan non-overlapping CIDRs
[ ] Plan EKS IP capacity
[ ] Understand ENI/IP limits
[ ] Monitor subnet utilization
[ ] Plan IPv6 security
[ ] Test both address families
```

------------------------------------------------------------------------