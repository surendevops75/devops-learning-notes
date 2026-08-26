# IP Addressing and Subnetting

## Purpose

Subnetting is one of the most important practical networking skills for
a DevOps engineer.

Understanding subnetting is required when designing and troubleshooting:

-   AWS VPCs
-   public and private subnets
-   EKS clusters
-   Kubernetes Pod networks
-   Docker networks
-   VPNs
-   Transit Gateway
-   VPC peering
-   hybrid connectivity
-   firewall rules
-   route tables
-   IP allowlists
-   load balancers
-   databases
-   production environments

The goal is not merely to memorize CIDR tables.

The goal is to calculate and reason about:

``` text
IP
 ↓
Prefix
 ↓
Network
 ↓
Host range
 ↓
Broadcast
 ↓
Routing
 ↓
Capacity
```

------------------------------------------------------------------------

# 1. What Is Subnetting?

Subnetting divides a larger IP network into smaller logical networks.

Example:

``` text
10.0.0.0/24
```

can be divided into:

``` text
10.0.0.0/26
10.0.0.64/26
10.0.0.128/26
10.0.0.192/26
```

Subnetting allows organizations to create controlled network boundaries.

------------------------------------------------------------------------

# 2. Why Subnetting Matters in DevOps

A production environment should not normally put every workload into one
giant network.

Instead:

``` text
Internet-facing
Application
Data
Management
Security
```

can have different network segments.

This improves:

``` text
routing
security
capacity management
blast-radius control
operations
```

------------------------------------------------------------------------

# 3. CIDR Review

CIDR notation:

``` text
10.0.0.0/24
```

means:

``` text
IP network = 10.0.0.0
Prefix     = 24 bits
```

IPv4 has:

``` text
32 total bits
```

Therefore:

``` text
host bits = 32 - prefix
```

------------------------------------------------------------------------

# 4. Core Subnet Formula

For IPv4:

``` text
Total addresses = 2^(32 - prefix)
```

Examples:

``` text
/24 → 2^8  = 256
/25 → 2^7  = 128
/26 → 2^6  = 64
/27 → 2^5  = 32
/28 → 2^4  = 16
/29 → 2^3  = 8
/30 → 2^2  = 4
```

------------------------------------------------------------------------

# 5. Traditional Usable Host Formula

For ordinary IPv4 subnets:

``` text
usable hosts = total addresses - 2
```

because:

``` text
1 = network address
1 = broadcast address
```

Example:

``` text
/26
64 total
62 traditional usable
```

Cloud platforms may reserve additional addresses.

------------------------------------------------------------------------

# 6. Prefix Length Meaning

A larger prefix number means a smaller network.

Compare:

``` text
/16
/20
/24
```

Ordering by size:

``` text
/16  largest
/20
/24  smallest
```

This is one of the most common interview questions.

------------------------------------------------------------------------

# 7. Subnet Mask

Common masks:

``` text
/8   = 255.0.0.0
/16  = 255.255.0.0
/24  = 255.255.255.0
```

For non-octet boundaries:

``` text
/25 = 255.255.255.128
/26 = 255.255.255.192
/27 = 255.255.255.224
/28 = 255.255.255.240
/29 = 255.255.255.248
/30 = 255.255.255.252
```

------------------------------------------------------------------------

# 8. Binary Mask Concept

A subnet mask contains:

``` text
1s = network bits
0s = host bits
```

For `/26`:

``` text
11111111.11111111.11111111.11000000
```

Therefore:

``` text
network bits = 26
host bits    = 6
```

------------------------------------------------------------------------

# 9. `/25` Calculation

A `/25` has:

``` text
32 - 25 = 7 host bits
```

Therefore:

``` text
2^7 = 128 addresses
```

Traditional usable:

``` text
126
```

Mask:

``` text
255.255.255.128
```

------------------------------------------------------------------------

# 10. `/26` Calculation

A `/26` has:

``` text
6 host bits
```

Therefore:

``` text
2^6 = 64 addresses
```

Traditional usable:

``` text
62
```

Mask:

``` text
255.255.255.192
```

------------------------------------------------------------------------

# 11. `/27` Calculation

A `/27` has:

``` text
5 host bits
```

Therefore:

``` text
32 addresses
```

Traditional usable:

``` text
30
```

Mask:

``` text
255.255.255.224
```

------------------------------------------------------------------------

# 12. `/28` Calculation

A `/28` has:

``` text
4 host bits
```

Therefore:

``` text
16 addresses
```

Traditional usable:

``` text
14
```

Mask:

``` text
255.255.255.240
```

------------------------------------------------------------------------

# 13. `/29` Calculation

A `/29` has:

``` text
3 host bits
```

Therefore:

``` text
8 addresses
```

Traditional usable:

``` text
6
```

Mask:

``` text
255.255.255.248
```

------------------------------------------------------------------------

# 14. `/30` Calculation

A `/30` has:

``` text
2 host bits
```

Therefore:

``` text
4 addresses
```

Traditional usage:

``` text
network
2 hosts
broadcast
```

Mask:

``` text
255.255.255.252
```

------------------------------------------------------------------------

# 15. CIDR Quick Reference

  Prefix     Total IPv4 Addresses          Traditional Hosts
  -------- ---------------------- --------------------------
  /16                      65,536                     65,534
  /17                      32,768                     32,766
  /18                      16,384                     16,382
  /19                       8,192                      8,190
  /20                       4,096                      4,094
  /21                       2,048                      2,046
  /22                       1,024                      1,022
  /23                         512                        510
  /24                         256                        254
  /25                         128                        126
  /26                          64                         62
  /27                          32                         30
  /28                          16                         14
  /29                           8                          6
  /30                           4                          2
  /31                           2   point-to-point semantics
  /32                           1                one address

Cloud-specific reservation rules must be applied separately.

------------------------------------------------------------------------

# 16. Powers of Two to Memorize

Useful values:

``` text
2^1  = 2
2^2  = 4
2^3  = 8
2^4  = 16
2^5  = 32
2^6  = 64
2^7  = 128
2^8  = 256
2^9  = 512
2^10 = 1024
2^11 = 2048
2^12 = 4096
2^16 = 65536
```

These make subnet calculations fast during interviews.

------------------------------------------------------------------------

# 17. The Block Size Method

For subnetting, block size can make calculations easier.

Example:

``` text
/26
```

Mask:

``` text
255.255.255.192
```

Block size:

``` text
256 - 192 = 64
```

Therefore subnet boundaries are:

``` text
0
64
128
192
```

------------------------------------------------------------------------

# 18. `/26` Subnets in a `/24`

Given:

``` text
192.168.1.0/24
```

divide into `/26`.

Block size:

``` text
64
```

Networks:

``` text
192.168.1.0/26
192.168.1.64/26
192.168.1.128/26
192.168.1.192/26
```

There are:

``` text
4 subnets
```

------------------------------------------------------------------------

# 19. `/27` Subnets in a `/24`

Block size:

``` text
32
```

Networks:

``` text
192.168.1.0/27
192.168.1.32/27
192.168.1.64/27
192.168.1.96/27
192.168.1.128/27
192.168.1.160/27
192.168.1.192/27
192.168.1.224/27
```

Total:

``` text
8 subnets
```

------------------------------------------------------------------------

# 20. `/28` Subnets in a `/24`

Block size:

``` text
16
```

Boundaries:

``` text
0
16
32
48
64
80
96
112
128
144
160
176
192
208
224
240
```

Therefore:

``` text
16 /28 subnets
```

fit into one `/24`.

------------------------------------------------------------------------

# 21. Finding the Network Address

Given:

``` text
192.168.10.75/26
```

Mask:

``` text
255.255.255.192
```

Block size:

``` text
64
```

Boundaries:

``` text
0
64
128
192
```

75 falls between:

``` text
64–127
```

Therefore:

``` text
Network = 192.168.10.64/26
```

------------------------------------------------------------------------

# 22. Finding the Broadcast Address

For:

``` text
192.168.10.75/26
```

network:

``` text
192.168.10.64
```

next network:

``` text
192.168.10.128
```

Therefore broadcast is one address before the next network:

``` text
192.168.10.127
```

------------------------------------------------------------------------

# 23. Finding Host Range

For:

``` text
192.168.10.64/26
```

network:

``` text
192.168.10.64
```

broadcast:

``` text
192.168.10.127
```

Traditional host range:

``` text
192.168.10.65
–
192.168.10.126
```

------------------------------------------------------------------------

# 24. Complete `/26` Example

Given:

``` text
192.168.10.75/26
```

Answer:

``` text
Network:
192.168.10.64

Broadcast:
192.168.10.127

Traditional host range:
192.168.10.65–126

Total:
64

Traditional usable:
62
```

------------------------------------------------------------------------

# 25. Finding a Network from `/27`

Given:

``` text
10.20.30.118/27
```

Mask:

``` text
255.255.255.224
```

Block size:

``` text
32
```

Boundaries:

``` text
0
32
64
96
128
160
192
224
```

118 belongs to:

``` text
96–127
```

Therefore:

``` text
Network = 10.20.30.96/27
```

------------------------------------------------------------------------

# 26. Broadcast for `/27`

For:

``` text
10.20.30.96/27
```

next network:

``` text
10.20.30.128
```

broadcast:

``` text
10.20.30.127
```

host range:

``` text
10.20.30.97–126
```

------------------------------------------------------------------------

# 27. Finding a Network from `/28`

Given:

``` text
10.20.30.141/28
```

block size:

``` text
16
```

boundaries:

``` text
128
144
```

141 belongs to:

``` text
128–143
```

Therefore:

``` text
Network = 10.20.30.128/28
Broadcast = 10.20.30.143
Hosts = 129–142
```

------------------------------------------------------------------------

# 28. Finding a Network from `/20`

Given:

``` text
10.10.37.20/20
```

Mask:

``` text
255.255.240.0
```

The interesting octet is the third.

Block size:

``` text
256 - 240 = 16
```

Boundaries:

``` text
0
16
32
48
64
...
```

37 belongs to:

``` text
32–47
```

Therefore:

``` text
Network = 10.10.32.0/20
Broadcast = 10.10.47.255
```

------------------------------------------------------------------------

# 29. Finding a Network from `/21`

Given:

``` text
172.16.35.20/21
```

Mask:

``` text
255.255.248.0
```

Block size:

``` text
256 - 248 = 8
```

Third-octet boundaries:

``` text
0
8
16
24
32
40
```

35 belongs to:

``` text
32–39
```

Therefore:

``` text
Network = 172.16.32.0/21
Broadcast = 172.16.39.255
```

------------------------------------------------------------------------

# 30. Finding a Network from `/18`

Given:

``` text
10.20.75.10/18
```

Mask:

``` text
255.255.192.0
```

Block size:

``` text
256 - 192 = 64
```

Third-octet boundaries:

``` text
0
64
128
192
```

75 belongs to:

``` text
64–127
```

Therefore:

``` text
Network = 10.20.64.0/18
Broadcast = 10.20.127.255
```

------------------------------------------------------------------------

# 31. Subnetting by Borrowing Bits

Suppose:

``` text
10.0.0.0/24
```

You want four equal subnets.

Four requires:

``` text
2^2 = 4
```

Borrow two host bits.

New prefix:

``` text
/24 + 2 = /26
```

Therefore:

``` text
/26
```

------------------------------------------------------------------------

# 32. Subnetting by Eight

Suppose:

``` text
192.168.1.0/24
```

You need:

``` text
8 subnets
```

Required bits:

``` text
2^3 = 8
```

New prefix:

``` text
/27
```

------------------------------------------------------------------------

# 33. Subnetting by Sixteen

Need:

``` text
16 subnets
```

Required bits:

``` text
2^4 = 16
```

From:

``` text
/24
```

new prefix:

``` text
/28
```

------------------------------------------------------------------------

# 34. Reverse Calculation

Given:

``` text
10.0.0.0/24
```

and requirement:

``` text
at least 50 hosts per subnet
```

Need:

``` text
2^h - 2 >= 50
```

Try:

``` text
h = 6
```

``` text
2^6 - 2 = 62
```

Therefore:

``` text
6 host bits
```

Prefix:

``` text
32 - 6 = /26
```

Use:

``` text
/26
```

------------------------------------------------------------------------

# 35. Host Requirement of 100

Need at least:

``` text
100 hosts
```

Try:

``` text
2^7 - 2 = 126
```

Therefore:

``` text
7 host bits
```

Prefix:

``` text
32 - 7 = /25
```

Use:

``` text
/25
```

------------------------------------------------------------------------

# 36. Host Requirement of 500

Need at least:

``` text
500 hosts
```

Try:

``` text
2^9 - 2 = 510
```

Therefore:

``` text
9 host bits
```

Prefix:

``` text
32 - 9 = /23
```

Use:

``` text
/23
```

------------------------------------------------------------------------

# 37. Host Requirement of 1000

Need:

``` text
1000 hosts
```

Try:

``` text
2^10 - 2 = 1022
```

Prefix:

``` text
32 - 10 = /22
```

Use:

``` text
/22
```

------------------------------------------------------------------------

# 38. Subnet Requirement vs Host Requirement

These are different questions.

### Question A

``` text
How many subnets?
```

You borrow bits.

### Question B

``` text
How many hosts per subnet?
```

You reserve enough host bits.

Interviewers often test whether you can distinguish the two.

------------------------------------------------------------------------

# 39. VLSM

VLSM means:

``` text
Variable Length Subnet Mask
```

It allows different subnet sizes inside the same larger address block.

Example:

``` text
Application → /22
Management  → /26
Database    → /27
Monitoring  → /28
```

This avoids wasting address space.

------------------------------------------------------------------------

# 40. Why VLSM Matters

Without VLSM, organizations may allocate equal-sized subnets even when
workloads have different requirements.

Example:

``` text
App needs 1000 IPs
Monitoring needs 20 IPs
```

Giving both `/22` wastes addresses.

VLSM lets you size appropriately.

------------------------------------------------------------------------

# 41. VLSM Design Example

Parent:

``` text
10.0.0.0/16
```

Requirements:

``` text
App:       1000
Database:  200
Monitoring: 50
Management: 20
```

Possible allocations:

``` text
App:
10.0.0.0/22

Database:
10.0.4.0/24

Monitoring:
10.0.5.0/26

Management:
10.0.5.64/27
```

Exact allocation should leave room for future growth.

------------------------------------------------------------------------

# 42. VLSM Allocation Rule

When manually allocating VLSM subnets:

``` text
1. Sort requirements largest to smallest.
2. Allocate largest first.
3. Select the smallest suitable CIDR.
4. Continue with the next available aligned block.
5. Reserve future growth.
```

This prevents fragmentation and overlap.

------------------------------------------------------------------------

# 43. Subnet Alignment

A subnet must begin on a valid boundary for its prefix.

For `/26`:

``` text
0
64
128
192
```

Invalid example:

``` text
10.0.0.20/26
```

The address `10.0.0.20` can be an IP inside a `/26`, but it is not the
network address of a `/26`.

Its network is:

``` text
10.0.0.0/26
```

------------------------------------------------------------------------

# 44. Network Address vs Host Address

This distinction is critical.

``` text
10.0.0.0/26
```

is a network.

``` text
10.0.0.20/26
```

is a host address within that network.

Do not call `10.0.0.20/26` the network address.

------------------------------------------------------------------------

# 45. Broadcast Calculation

General method:

``` text
Find network
Find next subnet
Subtract one address
```

Example:

``` text
Network:
10.0.0.64/26

Next:
10.0.0.128

Broadcast:
10.0.0.127
```

------------------------------------------------------------------------

# 46. First and Last Host

For traditional IPv4 subnetting:

``` text
First host = network + 1
Last host  = broadcast - 1
```

Example:

``` text
Network:
10.0.0.64

Broadcast:
10.0.0.127

First:
10.0.0.65

Last:
10.0.0.126
```

Cloud-specific reservations must then be considered.

------------------------------------------------------------------------

# 47. Route Summarization

Route summarization combines multiple contiguous networks into one
larger prefix.

Example:

``` text
10.0.0.0/24
10.0.1.0/24
10.0.2.0/24
10.0.3.0/24
```

can be summarized as:

``` text
10.0.0.0/22
```

provided the addresses are aligned and the summary does not
unintentionally include unrelated networks.

------------------------------------------------------------------------

# 48. Why Route Summarization Matters

It can reduce:

``` text
routing table size
configuration complexity
route propagation
operational overhead
```

In AWS/hybrid networks, careful summarization can simplify:

``` text
Transit Gateway
VPN
BGP
on-prem routes
VPC connectivity
```

------------------------------------------------------------------------

# 49. Supernetting

Supernetting combines smaller networks into a larger block.

Example:

``` text
10.0.0.0/24
10.0.1.0/24
```

can be summarized as:

``` text
10.0.0.0/23
```

when alignment and contiguous requirements are satisfied.

------------------------------------------------------------------------

# 50. Longest Prefix Match

Routers can have multiple matching routes.

They select the most specific matching prefix.

Example:

``` text
10.0.0.0/8
10.0.10.0/24
10.0.10.128/25
```

Destination:

``` text
10.0.10.150
```

matches all three.

The most specific:

``` text
10.0.10.128/25
```

wins.

------------------------------------------------------------------------

# 51. Why Longest Prefix Match Matters in AWS

Route tables can contain:

``` text
local VPC route
Transit Gateway route
NAT route
Internet Gateway route
peering route
VPN route
```

Understanding prefix specificity helps explain unexpected routing.

------------------------------------------------------------------------

# 52. Default Route

IPv4:

``` text
0.0.0.0/0
```

IPv6:

``` text
::/0
```

These represent default routes.

They match destinations that do not have a more specific route.

------------------------------------------------------------------------

# 53. Host Route

A host route uses:

``` text
/32
```

Example:

``` text
10.0.1.50/32
```

It identifies one IPv4 address.

In routing:

``` text
10.0.1.50/32
```

is more specific than:

``` text
10.0.1.0/24
```

------------------------------------------------------------------------

# 54. Route Selection Example

Suppose:

``` text
10.0.0.0/8
10.20.0.0/16
10.20.30.0/24
10.20.30.50/32
```

Destination:

``` text
10.20.30.50
```

The selected route is:

``` text
10.20.30.50/32
```

because it is the longest prefix.

------------------------------------------------------------------------

# 55. CIDR and Security Rules

CIDRs are used in:

``` text
security groups
NACLs
firewalls
NetworkPolicies
load balancer rules
VPN rules
allowlists
```

Example:

``` text
10.20.0.0/16
```

means the rule applies to the entire CIDR, not one IP.

------------------------------------------------------------------------

# 56. `/32` Security Rule

Example:

``` text
10.20.10.15/32
```

means one specific IPv4 address.

This is useful when tightly restricting access.

But dynamic cloud workloads may make static `/32` rules operationally
difficult.

------------------------------------------------------------------------

# 57. Kubernetes NetworkPolicy CIDR

Example:

``` yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-monitoring
  namespace: payments
spec:
  podSelector:
    matchLabels:
      app: payments
  policyTypes:
    - Ingress
  ingress:
    - from:
        - ipBlock:
            cidr: 10.30.0.0/16
      ports:
        - protocol: TCP
          port: 8080
```

Actual behavior depends on CNI implementation and policy configuration.

------------------------------------------------------------------------

# 58. AWS Security Group CIDR

Example concept:

``` text
Source:
10.30.0.0/16
```

This permits traffic from addresses in that CIDR when the security group
rule and other network controls allow it.

Prefer security-group-to-security-group references where they provide a
more stable identity boundary than large CIDRs.

------------------------------------------------------------------------

# 59. Avoid `0.0.0.0/0` Unnecessarily

This means:

``` text
all IPv4 addresses
```

Example:

``` text
TCP 22 from 0.0.0.0/0
```

can expose SSH broadly.

Production principle:

``` text
minimum required source range
```

Use VPN, bastion/access tooling, identity-aware controls or other
approved mechanisms rather than globally exposing administration ports.

------------------------------------------------------------------------

# 60. CIDR in Docker

Docker commonly creates private address pools.

Example:

``` text
172.17.0.0/16
```

Inspect:

``` bash
docker network inspect bridge
```

Custom networks can use explicit subnets:

``` bash
docker network create \
  --subnet 172.30.0.0/24 \
  app-network
```

Avoid overlap with host, VPN and enterprise networks.

------------------------------------------------------------------------

# 61. Docker Network Overlap

Suppose:

``` text
Corporate VPN:
172.30.0.0/16

Docker:
172.30.0.0/16
```

Containers can experience routing ambiguity when accessing corporate
resources.

This is why enterprise Docker environments should define controlled
address pools.

------------------------------------------------------------------------

# 62. Kubernetes Pod CIDR

Some Kubernetes networking designs allocate a Pod CIDR to each node.

Conceptually:

``` text
Cluster Pod CIDR
        |
        +-- Node A → Pod CIDR
        +-- Node B → Pod CIDR
        +-- Node C → Pod CIDR
```

The exact implementation depends on the Kubernetes networking plugin.

------------------------------------------------------------------------

# 63. Service CIDR

Kubernetes Services commonly use a separate virtual IP range.

Example:

``` text
10.96.0.0/12
```

This is a common default in some Kubernetes distributions, not a
universal requirement.

Service IPs are virtual and handled by the cluster networking
implementation.

------------------------------------------------------------------------

# 64. Pod CIDR vs Service CIDR

These should not overlap.

Conceptually:

``` text
VPC CIDR
10.0.0.0/16

Pod CIDR
10.244.0.0/16

Service CIDR
10.96.0.0/12
```

The exact ranges depend on the platform.

------------------------------------------------------------------------

# 65. EKS Networking Difference

AWS VPC CNI commonly gives Pods VPC-routable addresses rather than using
a simple cluster-internal Pod CIDR model.

Therefore, do not blindly apply networking assumptions from:

``` text
kubeadm + Calico
```

to:

``` text
EKS + AWS VPC CNI
```

Always identify the actual CNI and networking mode.

------------------------------------------------------------------------

# 66. AWS Subnet Sizing

AWS subnet planning must consider provider reservations.

For an IPv4 subnet, AWS reserves five IPv4 addresses in each subnet for
networking purposes.

Therefore:

``` text
traditional usable calculation
```

is not the same as:

``` text
AWS available IPv4 address count
```

Always use AWS-specific capacity information when planning EKS.

------------------------------------------------------------------------

# 67. AWS `/24` Example

A `/24` contains:

``` text
256 total IPv4 addresses
```

AWS reserves five addresses in the subnet.

Therefore:

``` text
251 usable IPv4 addresses
```

for the subnet under AWS's documented reservation model.

This does not mean 251 Pods can necessarily run there; ENI and workload
constraints also apply.

------------------------------------------------------------------------

# 68. AWS `/20` Example

A `/20` contains:

``` text
4096 total addresses
```

AWS reserves five.

Therefore:

``` text
4091 available IPv4 addresses
```

before considering resource-specific constraints.

This illustrates why larger subnets are often useful for EKS.

------------------------------------------------------------------------

# 69. EKS IP Capacity Is More Than Subnet Size

Even if a subnet has free addresses, Pod scheduling can be constrained
by:

``` text
instance ENI limits
IPv4 addresses per ENI
CNI configuration
warm IP targets
prefix delegation
node type
```

Therefore:

``` text
subnet capacity
≠
maximum Pod capacity
```

------------------------------------------------------------------------

# 70. Prefix Delegation in AWS VPC CNI

AWS VPC CNI can use prefix delegation on supported configurations.

Instead of allocating individual IP addresses in the same manner, the
CNI can work with delegated prefixes.

Benefits can include:

``` text
more efficient IP allocation
higher Pod density
reduced ENI pressure
faster address availability
```

Actual support and behavior depend on the CNI version, instance type and
cluster configuration.

------------------------------------------------------------------------

# 71. Secondary CIDR Blocks

AWS VPCs can have additional IPv4 CIDR blocks under supported
configurations.

This can help expand address capacity when the original CIDR is
insufficient.

Example:

``` text
Primary:
10.0.0.0/16

Additional:
100.64.0.0/16
```

The exact ranges and architecture must follow AWS and organizational
design requirements.

------------------------------------------------------------------------

# 72. Secondary CIDR and EKS

Additional VPC CIDRs can provide more address space for:

``` text
Pods
nodes
services
other resources
```

depending on the selected EKS networking design.

Expansion should be planned carefully because route and security
architecture may also need updates.

------------------------------------------------------------------------

# 73. RFC 6598 Shared Address Space

Range:

``` text
100.64.0.0/10
```

is shared address space defined for carrier-grade NAT and related uses.

It is not the same as RFC 1918 private address space.

It can be useful in some cloud networking designs, but should be used
only with a clear address-management plan.

------------------------------------------------------------------------

# 74. Address Allocation Hierarchy

A mature organization can plan:

``` text
Global enterprise pool
        |
        +-- AWS
        |    |
        |    +-- Account
        |         |
        |         +-- Region
        |              |
        |              +-- VPC
        |                   |
        |                   +-- Subnet
        |
        +-- On-prem
        |
        +-- Other cloud
```

This prevents accidental overlap.

------------------------------------------------------------------------

# 75. AWS IPAM Architecture

Conceptually:

``` text
AWS Organization
       |
      IPAM
       |
+------+-------+
|      |       |
Dev    QA     Prod
|      |       |
VPCs   VPCs   VPCs
```

Central IP management becomes increasingly important with:

``` text
many accounts
many regions
many VPCs
hybrid connectivity
many EKS clusters
```

------------------------------------------------------------------------

# 76. Multi-Cluster EKS Address Planning

Example:

``` text
EKS-DEV
10.10.0.0/16

EKS-QA
10.20.0.0/16

EKS-PROD-A
10.30.0.0/16

EKS-PROD-B
10.40.0.0/16
```

Avoid overlapping ranges if clusters need to communicate.

This is especially important for centralized platforms and cross-cluster
services.

------------------------------------------------------------------------

# 77. Multi-Account Address Planning

A production organization may structure:

``` text
AWS Organization
 |
 +-- Dev Account
 |     VPC 10.10.0.0/16
 |
 +-- QA Account
 |     VPC 10.20.0.0/16
 |
 +-- Prod Account
       VPC 10.30.0.0/16
```

The exact allocation should be managed centrally.

------------------------------------------------------------------------

# 78. Multi-Region Address Planning

Example:

``` text
Region A:
10.0.0.0/16

Region B:
10.1.0.0/16

Region C:
10.2.0.0/16
```

This makes global routing and troubleshooting easier.

------------------------------------------------------------------------

# 79. Environment-Based Subnetting

A production VPC might use:

``` text
Public:
10.30.0.0/20

Private application:
10.30.32.0/20

Data:
10.30.64.0/20

Platform:
10.30.96.0/20

Future:
10.30.128.0/17
```

This is illustrative.

Real sizing should be driven by:

``` text
workload
AZ count
ENI/IP requirements
load balancers
growth
```

------------------------------------------------------------------------

# 80. Availability Zone-Aware Subnetting

For three AZs:

``` text
AZ-A
AZ-B
AZ-C
```

create equivalent network tiers.

Example:

``` text
Public-A
Public-B
Public-C

Private-A
Private-B
Private-C

Data-A
Data-B
Data-C
```

This helps provide high availability.

------------------------------------------------------------------------

# 81. Why Not One Large Subnet?

A single large subnet can simplify initial deployment but can create:

``` text
blast radius
address-management problems
security complexity
AZ coupling
routing complexity
```

AWS subnets are also associated with a single Availability Zone.

Therefore production VPCs commonly use multiple subnets across AZs.

------------------------------------------------------------------------

# 82. Public vs Private Subnet Is a Routing Property

A subnet is generally considered public when its route table has a route
to an Internet Gateway.

A private subnet does not have a direct route to an Internet Gateway for
normal instance Internet access.

The label alone is not what makes it public or private.

------------------------------------------------------------------------

# 83. Public Subnet Example

``` text
10.30.0.0/24
```

Route:

``` text
10.30.0.0/16 → local
0.0.0.0/0    → Internet Gateway
```

This is a public-subnet routing pattern.

A resource still needs appropriate public addressing and security
controls to be Internet reachable.

------------------------------------------------------------------------

# 84. Private Subnet Example

``` text
10.30.10.0/24
```

Route:

``` text
10.30.0.0/16 → local
0.0.0.0/0    → NAT Gateway
```

This allows outbound Internet access through NAT for IPv4 workloads
without directly assigning public addresses.

------------------------------------------------------------------------

# 85. Isolated Subnet

An isolated subnet may have:

``` text
local VPC route
```

but no Internet egress route.

Common uses:

``` text
database
highly restricted workloads
internal data services
```

Additional routes may be added for required private services.

------------------------------------------------------------------------

# 86. Route Table and CIDR

Routing is based on destination prefixes.

Example:

``` text
10.30.0.0/16 → local
0.0.0.0/0    → NAT
```

A packet destined for:

``` text
10.30.5.20
```

matches:

``` text
10.30.0.0/16
```

rather than the default route.

------------------------------------------------------------------------

# 87. CIDR and NAT

NAT can translate addresses across network boundaries.

Example:

``` text
10.30.10.20
    |
 NAT
    |
198.51.100.20
```

NAT does not eliminate the need for correct routing.

------------------------------------------------------------------------

# 88. CIDR and VPN

A VPN commonly advertises or routes specific CIDRs.

Example:

``` text
AWS:
10.30.0.0/16

On-prem:
10.100.0.0/16
```

Traffic between these networks requires:

``` text
non-overlap
routes
VPN
security controls
```

------------------------------------------------------------------------

# 89. CIDR and VPC Peering

VPC peering generally requires non-overlapping IPv4 CIDRs.

Example:

``` text
VPC-A:
10.10.0.0/16

VPC-B:
10.20.0.0/16
```

works much better than:

``` text
VPC-A:
10.10.0.0/16

VPC-B:
10.10.0.0/16
```

------------------------------------------------------------------------

# 90. CIDR and Transit Gateway

Transit Gateway can connect many VPCs.

The larger the environment, the more important:

``` text
CIDR uniqueness
route-table segmentation
propagation
central IPAM
```

become.

------------------------------------------------------------------------

# 91. Route Summarization in Hybrid Networking

Suppose an organization has:

``` text
10.10.0.0/24
10.10.1.0/24
10.10.2.0/24
10.10.3.0/24
```

Instead of advertising four routes, it may advertise:

``` text
10.10.0.0/22
```

if appropriate.

This reduces routing complexity.

------------------------------------------------------------------------

# 92. Summary Route Risk

A summary can accidentally cover networks that should not be reachable.

Example:

``` text
10.0.0.0/8
```

is extremely broad.

If it is advertised where only:

``` text
10.20.0.0/16
```

was intended, traffic may be routed incorrectly.

Summarization must be precise.

------------------------------------------------------------------------

# 93. CIDR Overlap Detection

Given:

``` text
Network A:
10.0.0.0/16

Network B:
10.0.50.0/24
```

B is contained within A.

Therefore:

``` text
overlap = yes
```

Use tools rather than guessing for large address plans.

Python's `ipaddress` module is excellent for automation.

------------------------------------------------------------------------

# 94. Python CIDR Check

Example:

``` python
from ipaddress import ip_network

a = ip_network("10.0.0.0/16")
b = ip_network("10.20.0.0/24")

print(a.overlaps(b))
```

Output:

``` text
True
```

This is useful for CI validation of infrastructure repositories.

------------------------------------------------------------------------

# 95. Terraform CIDR Functions

Terraform provides CIDR functions such as:

``` text
cidrsubnet
cidrsubnets
cidrhost
```

Example:

``` hcl
cidrsubnet("10.0.0.0/16", 4, 2)
```

These functions help generate predictable subnet allocations.

------------------------------------------------------------------------

# 96. Terraform and AWS VPC Design

Example concept:

``` hcl
variable "vpc_cidr" {
  default = "10.30.0.0/16"
}

locals {
  private_subnets = [
    "10.30.0.0/20",
    "10.30.16.0/20",
    "10.30.32.0/20"
  ]
}
```

Production Terraform should keep address planning explicit, reviewable
and version controlled.

------------------------------------------------------------------------

# 97. Infrastructure-as-Code CIDR Validation

A production pipeline can validate:

``` text
CIDR syntax
CIDR overlap
required subnet size
AZ distribution
reserved ranges
environment boundaries
```

before Terraform applies changes.

This prevents many networking mistakes before deployment.

------------------------------------------------------------------------

# 98. GitOps and Network Configuration

Network configuration can also be managed declaratively.

Example:

``` text
Git
 |
Terraform
 |
AWS VPC
```

or:

``` text
Git
 |
Kubernetes manifests
 |
NetworkPolicy
```

The same GitOps principles can therefore apply to network-related
configuration.

------------------------------------------------------------------------

# 99. CIDR and GitOps Safety

A pull request changing:

``` text
10.30.16.0/20
```

should be reviewed for:

``` text
overlap
route impact
security impact
capacity impact
downstream dependencies
```

Networking changes should not be treated as ordinary application-only
changes.

------------------------------------------------------------------------

# 100. Production Subnet Design Workflow

``` text
1. Inventory environments.
2. Inventory regions.
3. Inventory accounts.
4. Estimate workloads.
5. Reserve growth.
6. Allocate VPC CIDRs.
7. Allocate subnet CIDRs.
8. Validate non-overlap.
9. Define route tables.
10. Define security boundaries.
11. Define IP monitoring.
12. Automate validation.
```

------------------------------------------------------------------------

# 101. EKS Production Subnet Workflow

``` text
VPC
 |
 +-- AZ-A
 |    +-- Public
 |    +-- Private
 |    +-- Data
 |
 +-- AZ-B
 |    +-- Public
 |    +-- Private
 |    +-- Data
 |
 +-- AZ-C
      +-- Public
      +-- Private
      +-- Data
```

Then estimate:

``` text
nodes
Pods
load balancers
ENIs
IP consumption
future growth
```

------------------------------------------------------------------------

# 102. EKS IP Capacity Incident

### Symptom

``` text
Deployment scales from 200 to 400 Pods.
New Pods fail.
```

First:

``` bash
kubectl get pods -A -o wide
kubectl describe pod <pod> -n <namespace>
```

Look for:

``` text
FailedCreatePodSandbox
CNI errors
IP allocation errors
```

Then check AWS:

``` text
subnet free IPs
ENI capacity
instance type
CNI configuration
prefix delegation
```

------------------------------------------------------------------------

# 103. EKS Subnet Exhaustion vs CPU Exhaustion

CPU exhaustion:

``` text
Insufficient cpu
```

Network exhaustion:

``` text
CNI/IP allocation failure
```

These are different capacity dimensions.

A node can have:

``` text
40% CPU
```

and still fail to create Pods because:

``` text
no available IP
```

This is a critical production lesson.

------------------------------------------------------------------------

# 104. EKS Address Capacity Monitoring

Monitor:

``` text
subnet available IPs
Pod count
node count
ENI count
IP allocation
CNI errors
```

Set alerts before capacity reaches a critical threshold.

Do not wait until:

``` text
0 free IPs
```

------------------------------------------------------------------------

# 105. Production CIDR Documentation

Maintain a table:

  Environment   Account   Region     VPC CIDR       Purpose
  ------------- --------- ---------- -------------- -----------------
  Dev           Dev       Region-A   10.10.0.0/16   Development
  QA            QA        Region-A   10.20.0.0/16   Testing
  Prod          Prod      Region-A   10.30.0.0/16   Production
  Shared        Shared    Region-A   10.40.0.0/16   Shared services

Add:

``` text
owner
created date
purpose
growth reserve
connected networks
```

------------------------------------------------------------------------

# 106. Production CIDR Naming

Use predictable names:

``` text
prod-us-east-1-vpc
prod-us-east-1-private-a
prod-us-east-1-private-b
prod-us-east-1-public-a
```

Names should communicate:

``` text
environment
region
resource
AZ/tier
```

------------------------------------------------------------------------

# 107. CIDR Naming in Terraform

Example:

``` hcl
locals {
  vpc_cidr = "10.30.0.0/16"

  private_subnets = {
    az_a = "10.30.16.0/20"
    az_b = "10.30.32.0/20"
    az_c = "10.30.48.0/20"
  }
}
```

This is clearer than scattered magic strings.

------------------------------------------------------------------------

# 108. Production Best Practice: Reserve Growth

Bad:

``` text
Use 100% of VPC space immediately.
```

Better:

``` text
Current allocation
+
future allocation
+
new workload reserve
```

Address space is infrastructure capacity.

------------------------------------------------------------------------

# 109. Production Best Practice: Avoid Random Private Ranges

Do not create:

``` text
VPC-1 = 10.0.0.0/16
VPC-2 = 10.0.0.0/16
VPC-3 = 192.168.1.0/24
```

without an enterprise plan.

This becomes painful when networks need to connect later.

------------------------------------------------------------------------

# 110. Production Best Practice: Central IPAM

At scale, maintain:

``` text
enterprise CIDR registry
```

or use AWS IPAM plus organizational processes.

Every network should have:

``` text
owner
purpose
region
account
CIDR
connectivity
```

------------------------------------------------------------------------

# 111. Production Best Practice: Separate Data Networks

Database workloads often require stronger isolation.

Example:

``` text
Application subnet
      |
      | allowed database port
      v
Database subnet
```

Avoid broad:

``` text
0.0.0.0/0
```

access to databases.

------------------------------------------------------------------------

# 112. Production Best Practice: EKS Private Subnets

A common EKS production pattern:

``` text
Internet
   |
Public ALB
   |
Private EKS nodes
   |
Private Pods
   |
Private data services
```

The nodes and workloads do not need public IPs for ordinary inbound
application traffic.

------------------------------------------------------------------------

# 113. Production Best Practice: ALB Instead of Direct Pod Exposure

For RoboShop:

``` text
Internet
   |
ALB
   |
Ingress
   |
Service
   |
Pods
```

Do not expose individual Pod IPs directly to the Internet.

------------------------------------------------------------------------

# 114. Production Best Practice: Security Groups

Use narrow sources.

Prefer:

``` text
ALB SG
   ↓
Application SG
   ↓
Database SG
```

over:

``` text
0.0.0.0/0
```

for internal application dependencies.

------------------------------------------------------------------------

# 115. Production Best Practice: NetworkPolicy

Use Kubernetes NetworkPolicy where supported.

Example model:

``` text
frontend
   ↓
cart
   ↓
redis
```

and deny unnecessary:

``` text
frontend → database
```

This reduces lateral movement.

------------------------------------------------------------------------

# 116. Production Best Practice: Test Network Changes

Before production:

``` text
terraform plan
CIDR validation
route validation
security review
connectivity test
rollback plan
```

For critical networking, test failure scenarios as well.

------------------------------------------------------------------------

# 117. Production Best Practice: Don't Change CIDR Casually

Changing a VPC/subnet CIDR can have large consequences.

Dependencies may include:

``` text
routes
security groups
NACLs
VPN
Transit Gateway
DNS
firewalls
databases
applications
allowlists
monitoring
```

Treat CIDR changes as major infrastructure changes.

------------------------------------------------------------------------

# 118. Practical Exercise --- `/24` to `/26`

Given:

``` text
10.1.0.0/24
```

Create four equal subnets.

Answer:

``` text
10.1.0.0/26
10.1.0.64/26
10.1.0.128/26
10.1.0.192/26
```

------------------------------------------------------------------------

# 119. Practical Exercise --- `/24` to `/27`

Given:

``` text
10.1.0.0/24
```

Need:

``` text
8 subnets
```

Answer:

``` text
/27
```

Networks:

``` text
0
32
64
96
128
160
192
224
```

------------------------------------------------------------------------

# 120. Practical Exercise --- Find Network

Given:

``` text
192.168.100.201/27
```

Block:

``` text
32
```

Boundaries:

``` text
192
224
```

Answer:

``` text
Network:
192.168.100.192/27

Broadcast:
192.168.100.223

Host range:
192.168.100.193–222
```

------------------------------------------------------------------------

# 121. Practical Exercise --- Find Network

Given:

``` text
10.10.50.70/28
```

Block:

``` text
16
```

Boundaries:

``` text
64
80
```

Answer:

``` text
Network:
10.10.50.64/28

Broadcast:
10.10.50.79

Hosts:
10.10.50.65–78
```

------------------------------------------------------------------------

# 122. Practical Exercise --- Host Requirement

Requirement:

``` text
60 hosts
```

Find smallest subnet.

``` text
2^6 - 2 = 62
```

Therefore:

``` text
/26
```

------------------------------------------------------------------------

# 123. Practical Exercise --- Host Requirement

Requirement:

``` text
200 hosts
```

Try:

``` text
2^8 - 2 = 254
```

Therefore:

``` text
/24
```

------------------------------------------------------------------------

# 124. Practical Exercise --- Host Requirement

Requirement:

``` text
1000 hosts
```

``` text
2^10 - 2 = 1022
```

Therefore:

``` text
/22
```

------------------------------------------------------------------------

# 125. Practical Exercise --- Number of Subnets

Given:

``` text
10.0.0.0/16
```

Need:

``` text
64 equal subnets
```

Required bits:

``` text
2^6 = 64
```

New prefix:

``` text
/16 + 6 = /22
```

Therefore:

``` text
64 /22 subnets
```

------------------------------------------------------------------------

# 126. Practical Exercise --- Number of Subnets

Given:

``` text
192.168.0.0/24
```

Need:

``` text
16 subnets
```

Borrow:

``` text
4 bits
```

New prefix:

``` text
/28
```

------------------------------------------------------------------------

# 127. Practical Exercise --- VLSM

Parent:

``` text
10.50.0.0/24
```

Requirements:

``` text
Team A: 100 hosts
Team B: 50 hosts
Team C: 20 hosts
Team D: 10 hosts
```

Possible design:

``` text
Team A:
10.50.0.0/25

Team B:
10.50.0.128/26

Team C:
10.50.0.192/27

Team D:
10.50.0.224/28
```

Remaining:

``` text
10.50.0.240/28
```

can be reserved for growth.

------------------------------------------------------------------------

# 128. VLSM Alignment Check

For the previous design:

``` text
/25 → 0–127
/26 → 128–191
/27 → 192–223
/28 → 224–239
```

Everything is aligned.

The final:

``` text
240–255
```

remains available.

------------------------------------------------------------------------

# 129. Practical Exercise --- Route Summarization

Given:

``` text
10.10.0.0/24
10.10.1.0/24
10.10.2.0/24
10.10.3.0/24
```

Summary:

``` text
10.10.0.0/22
```

Reason:

``` text
4 × /24 = /22
```

and the block is aligned.

------------------------------------------------------------------------

# 130. Practical Exercise --- Longest Prefix Match

Routes:

``` text
10.0.0.0/8
10.20.0.0/16
10.20.30.0/24
10.20.30.128/25
```

Destination:

``` text
10.20.30.150
```

Matches all four.

Winner:

``` text
10.20.30.128/25
```

because it is most specific.

------------------------------------------------------------------------

# 131. Practical Exercise --- Detect Overlap

Network A:

``` text
10.0.0.0/16
```

Network B:

``` text
10.0.50.0/24
```

Answer:

``` text
overlap
```

B is contained inside A.

------------------------------------------------------------------------

# 132. Practical Exercise --- Detect Non-Overlap

Network A:

``` text
10.0.0.0/16
```

Network B:

``` text
10.1.0.0/16
```

Answer:

``` text
no overlap
```

They are adjacent distinct `/16` networks.

------------------------------------------------------------------------

# 133. Practical Exercise --- AWS Subnet

Given:

``` text
AWS subnet:
10.30.0.0/24
```

Total:

``` text
256
```

AWS reserved:

``` text
5
```

Available:

``` text
251
```

But application capacity must still account for:

``` text
ENI/IP usage
load balancers
nodes
Pods
```

------------------------------------------------------------------------

# 134. Practical Exercise --- EKS Capacity

Suppose:

``` text
Subnet:
10.30.0.0/20
```

Total:

``` text
4096
```

AWS-reserved:

``` text
5
```

Available:

``` text
4091
```

This does not mean:

``` text
4091 Pods
```

can run.

You must also consider:

``` text
nodes
ENIs
IPs per ENI
CNI behavior
other AWS resources
```

------------------------------------------------------------------------

# 135. Practical Exercise --- CIDR Planning

Design:

``` text
Dev
QA
Prod
Shared
```

Requirement:

``` text
each VPC needs growth
```

A simple example:

``` text
Dev:
10.10.0.0/16

QA:
10.20.0.0/16

Prod:
10.30.0.0/16

Shared:
10.40.0.0/16
```

Document and reserve additional ranges for future regions/accounts.

------------------------------------------------------------------------

# 136. Troubleshooting: Wrong CIDR

Symptom:

``` text
Connection goes to wrong route.
```

Check:

``` bash
ip route
```

and:

``` bash
ip route get <destination>
```

Then compare:

``` text
destination
prefix
selected route
interface
gateway
```

------------------------------------------------------------------------

# 137. Troubleshooting: Overlapping Networks

Symptoms:

``` text
VPN connection established
but internal destination unreachable
```

Check:

``` text
AWS VPC CIDR
on-prem CIDR
VPN routes
Transit Gateway routes
```

An overlap may prevent correct routing.

------------------------------------------------------------------------

# 138. Troubleshooting: IP Exhaustion

Linux:

``` bash
ip addr
ip route
```

Kubernetes:

``` bash
kubectl get pods -A -o wide
kubectl describe pod <pod>
```

AWS:

``` text
subnet available IPs
ENI capacity
```

Determine whether exhaustion is:

``` text
subnet-level
ENI-level
node-level
CNI-level
```

------------------------------------------------------------------------

# 139. Troubleshooting: Pod Pending

If Pod is:

``` text
Pending
```

do not assume CPU/memory.

Run:

``` bash
kubectl describe pod <pod> -n <namespace>
```

Look at:

``` text
Events
```

Possible network-related cause:

``` text
failed to assign IP
failed to create Pod sandbox
CNI failure
```

------------------------------------------------------------------------

# 140. Troubleshooting: Security Rule Uses Wrong CIDR

Example:

``` text
Allowed:
10.20.0.0/16
```

Actual client:

``` text
10.30.10.20
```

The rule does not match.

Use:

``` bash
ip route
```

and application/load-balancer logs to determine the actual source
address.

------------------------------------------------------------------------

# 141. Troubleshooting: Source IP Changed

Traffic path:

``` text
Client
 ↓
NAT
 ↓
ALB
 ↓
Pod
```

The backend may not see the original network source address.

Investigate:

``` text
NAT
load balancer
proxy
externalTrafficPolicy
X-Forwarded-For
Proxy Protocol
```

Use the correct source-of-truth for each layer.

------------------------------------------------------------------------

# 142. Troubleshooting: Route Looks Correct but Traffic Fails

Correct route does not prove connectivity.

Continue:

``` text
route
 ↓
security
 ↓
network policy
 ↓
TCP listener
 ↓
application
```

Test:

``` bash
nc -vz <host> <port>
```

Then:

``` bash
tcpdump
```

if necessary.

------------------------------------------------------------------------

# 143. Troubleshooting: One Subnet Works, Another Fails

Compare:

``` text
CIDR
route table
NACL
security group
DNS
AZ
NAT
endpoint routing
```

Do not assume the application changed.

The difference may be the subnet's network path.

------------------------------------------------------------------------

# 144. Troubleshooting: Different AZ Behavior

If:

``` text
AZ-A works
AZ-B fails
```

compare:

``` text
subnet route table
NACL
available IPs
NAT architecture
load balancer registration
CNI behavior
```

AZ-specific misconfiguration is a common cloud troubleshooting pattern.

------------------------------------------------------------------------

# 145. Production Incident Runbook

### Incident

New EKS nodes cannot scale.

### Step 1

Check autoscaler/Karpenter events.

### Step 2

Check subnet capacity.

### Step 3

Check instance ENI limits.

### Step 4

Check route/security configuration.

### Step 5

Check AWS service quotas.

### Step 6

Check whether the requested AZ has capacity.

### Step 7

Determine whether the failure is:

``` text
compute
network
IP
quota
AZ capacity
```

------------------------------------------------------------------------

# 146. Production Incident --- CIDR Expansion

Before expanding address space:

``` text
1. Check current CIDRs.
2. Check route tables.
3. Check connected networks.
4. Check overlapping ranges.
5. Check security rules.
6. Check Terraform.
7. Check EKS CNI requirements.
8. Test in non-production.
9. Create rollback plan.
10. Monitor after change.
```

------------------------------------------------------------------------

# 147. Production CIDR Governance

Every new CIDR request should include:

``` text
requester
environment
account
region
purpose
required capacity
growth estimate
parent CIDR
connectivity requirements
overlap validation
approval
```

This makes network changes auditable.

------------------------------------------------------------------------

# 148. Security: CIDR Is Not Identity

A CIDR tells you:

``` text
where an address belongs
```

It does not prove:

``` text
who owns the request
```

Do not build authentication solely around:

``` text
source IP
```

Use identity-based controls where possible.

------------------------------------------------------------------------

# 149. Security: Network Segmentation

Example:

``` text
Internet
   |
 ALB
   |
Application
   |
Database
```

Rules:

``` text
Internet → ALB : 443
ALB → App      : 8080
App → DB       : 5432
```

Do not allow:

``` text
Internet → DB
```

------------------------------------------------------------------------

# 150. Security: Least-Privilege CIDRs

Prefer:

``` text
10.30.16.0/20
```

over:

``` text
10.0.0.0/8
```

when the application only needs one subnet.

Smaller source ranges reduce accidental exposure.

------------------------------------------------------------------------

# 151. Security: Dynamic Workloads

Static IP allowlists can become fragile with:

``` text
Kubernetes
autoscaling
serverless
ephemeral nodes
```

Prefer stable security identities when supported:

``` text
security groups
service identities
NetworkPolicy selectors
IAM
```

rather than constantly updating IP allowlists.

------------------------------------------------------------------------

# 152. Monitoring CIDR Capacity

Monitor:

``` text
available subnet IPs
allocated IPs
Pod count
node count
ENI count
load balancer ENIs
```

Create alerts for:

``` text
high utilization
rapid consumption
allocation failures
```

------------------------------------------------------------------------

# 153. Production Address Dashboard

A useful dashboard can show:

``` text
VPC
Subnet
CIDR
Total IPs
Available IPs
Utilization %
AZ
Environment
EKS cluster
```

This turns networking capacity into an observable resource.

------------------------------------------------------------------------

# 154. Disaster Recovery and CIDRs

DR architectures should account for:

``` text
primary VPC
DR VPC
primary region
DR region
VPN
Transit Gateway
DNS
database replication
```

Do not accidentally reuse overlapping CIDRs in a DR environment that
must later connect to primary.

------------------------------------------------------------------------

# 155. Multi-Region DR Example

``` text
Primary:
us-east-1
10.30.0.0/16

DR:
us-west-2
10.31.0.0/16
```

Distinct ranges simplify:

``` text
routing
replication
VPN
Transit Gateway
failover
```

------------------------------------------------------------------------

# 156. RoboShop Production CIDR Design

Illustrative:

``` text
PROD VPC
10.30.0.0/16
|
+-- Public-A  10.30.0.0/24
+-- Public-B  10.30.1.0/24
+-- Public-C  10.30.2.0/24
|
+-- App-A     10.30.16.0/20
+-- App-B     10.30.32.0/20
+-- App-C     10.30.48.0/20
|
+-- Data-A    10.30.64.0/24
+-- Data-B    10.30.65.0/24
+-- Data-C    10.30.66.0/24
```

The application subnets are intentionally larger because EKS workload/IP
consumption can be significant.

------------------------------------------------------------------------

# 157. RoboShop Network Flow

``` text
Developer
   |
Git
   |
CI
   |
ECR
   |
GitOps
   |
Argo CD
   |
EKS
   |
ALB
   |
Frontend
   |
Services
   |
Data
```

Each infrastructure boundary should have an intentional CIDR/routing
design.

------------------------------------------------------------------------

# 158. Terraform + EKS Example

Conceptual:

``` hcl
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"

  name = "roboshop-prod"
  cidr = "10.30.0.0/16"

  azs = [
    "us-east-1a",
    "us-east-1b",
    "us-east-1c"
  ]

  private_subnets = [
    "10.30.16.0/20",
    "10.30.32.0/20",
    "10.30.48.0/20"
  ]

  public_subnets = [
    "10.30.0.0/24",
    "10.30.1.0/24",
    "10.30.2.0/24"
  ]
}
```

Use approved module versions and validate subnet sizing before
production.

------------------------------------------------------------------------

# 159. Terraform CIDR Validation Example

Conceptual validation:

``` hcl
variable "vpc_cidr" {
  type = string

  validation {
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "vpc_cidr must be a valid CIDR."
  }
}
```

More sophisticated validation can enforce:

``` text
allowed prefix
non-overlap
environment ranges
```

------------------------------------------------------------------------

# 160. CI Network Validation

A production pipeline can run:

``` text
Terraform fmt
Terraform validate
Terraform plan
CIDR overlap test
security policy test
```

before merge.

This catches networking errors before infrastructure changes reach
production.

------------------------------------------------------------------------

# 161. Python CIDR Automation

Example:

``` python
from ipaddress import ip_network

networks = [
    ip_network("10.10.0.0/16"),
    ip_network("10.20.0.0/16"),
    ip_network("10.30.0.0/16"),
]

for i, first in enumerate(networks):
    for second in networks[i + 1:]:
        if first.overlaps(second):
            print("OVERLAP:", first, second)
```

This is useful for enterprise IP inventory validation.

------------------------------------------------------------------------

# 162. Interview --- What Is Subnetting?

> Subnetting divides a larger IP network into smaller logical networks
> by borrowing bits from the host portion and increasing the prefix
> length.

------------------------------------------------------------------------

# 163. Interview --- What Is CIDR?

> CIDR expresses an IP network using an address and prefix length, such
> as `10.0.0.0/16`. The prefix determines the network portion.

------------------------------------------------------------------------

# 164. Interview --- How Many IPs in `/24`?

``` text
2^(32-24)
= 256
```

Traditional usable:

``` text
254
```

AWS subnet availability differs because AWS reserves five addresses.

------------------------------------------------------------------------

# 165. Interview --- How Many IPs in `/20`?

``` text
2^(32-20)
= 4096
```

Traditional usable:

``` text
4094
```

AWS subnet available capacity is:

``` text
4091
```

before resource-specific considerations.

------------------------------------------------------------------------

# 166. Interview --- How Do You Find Network Address?

> Convert or inspect the subnet mask, determine the subnet boundary, and
> find the block containing the given host address. The block's first
> address is the network address.

Example:

``` text
192.168.1.70/26
```

belongs to:

``` text
192.168.1.64/26
```

------------------------------------------------------------------------

# 167. Interview --- How Do You Find Broadcast?

> Find the next subnet boundary and subtract one address.

For:

``` text
192.168.1.64/26
```

next:

``` text
192.168.1.128
```

broadcast:

``` text
192.168.1.127
```

------------------------------------------------------------------------

# 168. Interview --- What Is VLSM?

> Variable Length Subnet Masking allows different subnet sizes within a
> larger address block, allowing more efficient use of IP space.

------------------------------------------------------------------------

# 169. Interview --- What Is Supernetting?

> Supernetting combines multiple contiguous networks into a larger
> summarized prefix, reducing route count when the networks are aligned
> and logically appropriate.

------------------------------------------------------------------------

# 170. Interview --- What Is Longest Prefix Match?

> When multiple routes match a destination, the most specific route,
> meaning the longest matching prefix, is selected.

------------------------------------------------------------------------

# 171. Interview --- Why Does CIDR Overlap Matter?

> Overlapping address spaces make routing ambiguous when networks need
> to communicate. It is a major design problem for VPC peering, VPN,
> Transit Gateway and hybrid cloud connectivity.

------------------------------------------------------------------------

# 172. Interview --- Why Is EKS IP Planning Important?

> EKS networking consumes IP addresses for nodes, Pods, ENIs and other
> resources. In VPC-native networking, insufficient subnet or ENI/IP
> capacity can prevent Pods from being created even when CPU and memory
> are available.

------------------------------------------------------------------------

# 173. Interview --- Is `/24` Always 254 Usable IPs?

No.

Traditional IPv4 subnetting:

``` text
254
```

AWS:

``` text
251 available
```

because AWS reserves five IPv4 addresses in each subnet.

Other platforms may have different rules.

------------------------------------------------------------------------

# 174. Interview --- What Is the Difference Between Public and Private Subnet?

In AWS, the key distinction is routing.

A public subnet has a route to an Internet Gateway.

A private subnet does not have a direct route to an Internet Gateway for
normal outbound instance traffic and may use NAT or other private
connectivity.

------------------------------------------------------------------------

# 175. Interview --- Why Should VPCs Not Overlap?

> Overlapping VPC CIDRs create routing ambiguity and complicate
> connectivity through peering, Transit Gateway, VPN and hybrid
> networks.

------------------------------------------------------------------------

# 176. Interview --- What Is `0.0.0.0/0`?

> It is the IPv4 default route and represents all IPv4 destinations not
> matched by a more specific route.

------------------------------------------------------------------------

# 177. Interview --- What Is `/32`?

> A `/32` represents one IPv4 address. It is commonly used for host
> routes and highly specific rules.

------------------------------------------------------------------------

# 178. Interview --- Why Is `0.0.0.0/0` Dangerous in Security Rules?

> It represents every IPv4 source. Using it on sensitive ports can
> expose services to the Internet. Production rules should use the
> narrowest practical source range or identity-based control.

------------------------------------------------------------------------

# 179. Interview --- Why Can EKS Run Out of IPs Before CPU?

> Network capacity is an independent resource. A node or subnet can have
> sufficient CPU and memory while the CNI has no available IP addresses
> or ENI capacity for additional Pods.

------------------------------------------------------------------------

# 180. Interview --- How Do You Troubleshoot EKS IP Exhaustion?

``` text
1. Describe the failed Pod.
2. Inspect events.
3. Check subnet available IPs.
4. Check node instance type.
5. Check ENI/IP limits.
6. Check AWS VPC CNI.
7. Check prefix delegation if enabled.
8. Check recent scaling.
9. Check load balancer/IP consumption.
10. Plan capacity expansion.
```

------------------------------------------------------------------------

# 181. Interview --- How Do You Validate CIDR Overlap?

Options:

``` text
Python ipaddress
Terraform validation
AWS IPAM
IPAM tooling
network design databases
```

Example Python:

``` python
ip_network("10.0.0.0/16").overlaps(
    ip_network("10.0.20.0/24")
)
```

------------------------------------------------------------------------

# 182. Interview --- What Is Route Summarization?

> Route summarization combines multiple contiguous networks into a
> broader prefix to reduce routing-table complexity.

------------------------------------------------------------------------

# 183. Interview --- Why Use Different Subnets Across AZs?

> AWS subnets are tied to Availability Zones. Separate subnets across
> multiple AZs support high availability and allow workloads to
> distribute across failure domains.

------------------------------------------------------------------------

# 184. Interview --- Why Reserve Address Space?

> Address space is a capacity resource. Reserving future ranges avoids
> difficult renumbering when workloads, clusters, accounts or regions
> grow.

------------------------------------------------------------------------

# 185. Interview --- What Is the Difference Between Pod CIDR and Service CIDR?

> Pod CIDR represents the address space used for Pod networking in
> networking models that allocate Pod CIDRs, while Service CIDR is used
> for Kubernetes Service virtual IPs. In EKS with AWS VPC CNI, Pod
> addressing follows AWS VPC networking behavior, so the exact model
> differs from other CNIs.

------------------------------------------------------------------------

# 186. Interview --- Does a Pod IP Need to Be Stable?

Usually no.

Pods are ephemeral.

Use:

``` text
Service
DNS
```

for stable application discovery.

------------------------------------------------------------------------

# 187. Interview --- Why Should Docker Networks Avoid Corporate CIDRs?

> Overlapping container and corporate networks can create routing
> ambiguity when containers need to reach VPN or on-premises resources.

------------------------------------------------------------------------

# 188. Production Subnetting Checklist

``` text
[ ] VPC CIDR planned
[ ] CIDR does not overlap
[ ] Environment ranges separated
[ ] Region ranges planned
[ ] AZ ranges planned
[ ] Public subnets sized
[ ] Private subnets sized
[ ] Data subnets sized
[ ] EKS IP capacity estimated
[ ] ENI limits considered
[ ] Growth reserved
[ ] Route tables documented
[ ] Security rules reviewed
[ ] IPAM maintained
[ ] Terraform validation enabled
[ ] Monitoring enabled
```

------------------------------------------------------------------------

# 189. Final Mental Model

Subnetting is not just:

``` text
calculate /24
```

Production subnetting is:

``` text
Business requirements
        ↓
IP capacity
        ↓
CIDR allocation
        ↓
Subnet design
        ↓
AZ distribution
        ↓
Routing
        ↓
Security
        ↓
EKS/CNI capacity
        ↓
Growth
        ↓
Monitoring
```

------------------------------------------------------------------------

# 190. Final Summary

The most important formulas are:

``` text
IPv4 total addresses:
2^(32-prefix)

Host bits:
32-prefix

Traditional hosts:
2^host_bits - 2

Number of equal subnets:
2^(borrowed bits)
```

For practical troubleshooting:

``` text
ip addr
ip route
ip route get <destination>
ip neigh
```

For cloud:

``` text
VPC CIDR
Subnet CIDR
Route Table
ENI
IP capacity
Security
```

For EKS:

``` text
Subnet
 ↓
CNI
 ↓
ENI/IP allocation
 ↓
Pod IP
 ↓
Service
 ↓
ALB
```

------------------------------------------------------------------------