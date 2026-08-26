# 16-Networking-for-DevOps
# 06-CIDR-and-Subnet-Calculations

## 1. Purpose

CIDR and subnet calculations are core networking skills for DevOps engineers.

They are used when designing:

- AWS VPCs
- EKS clusters
- private and public subnets
- route tables
- security groups
- NACLs
- Kubernetes Pod CIDRs
- Service CIDRs
- NetworkPolicies
- VPNs
- Transit Gateway networks
- VPC peering
- hybrid connectivity
- firewall rules
- IP capacity plans

A DevOps engineer should be able to calculate subnet ranges without depending entirely on a calculator.

---

## 2. What Is CIDR?

CIDR means:

**Classless Inter-Domain Routing**

CIDR represents an IP network using:

```text
network-address/prefix-length
```

Example:

```text
10.0.0.0/16
```

The `/16` means:

```text
first 16 bits = network prefix
remaining 16 bits = host portion
```

CIDR replaced the old rigid classful addressing model and allows networks to be allocated in flexible sizes.

---

## 3. Why CIDR Matters in DevOps

CIDR directly affects:

```text
routing
subnet size
IP capacity
security rules
network isolation
AWS VPC design
EKS Pod capacity
Kubernetes networking
```

Bad CIDR planning can create production problems such as:

```text
IP exhaustion
overlapping networks
impossible VPC peering
insufficient EKS Pod IPs
routing conflicts
```

---

## 4. IPv4 Address Structure

IPv4 contains:

```text
32 bits
```

Example:

```text
192.168.10.25
```

It contains four octets:

```text
192 | 168 | 10 | 25
```

Each octet contains:

```text
8 bits
```

Therefore:

```text
8 × 4 = 32 bits
```

---

## 5. Binary Representation

Every IPv4 address can be represented in binary.

Example:

```text
192 = 11000000
168 = 10101000
10  = 00001010
25  = 00011001
```

Therefore:

```text
192.168.10.25
=
11000000.10101000.00001010.00011001
```

CIDR calculations become much easier when you understand the bit boundary.

---

## 6. Prefix Length

In:

```text
192.168.10.0/24
```

the prefix length is:

```text
24
```

Therefore:

```text
Network bits = 24
Host bits = 32 - 24 = 8
```

---

## 7. Host-Bit Formula

For IPv4:

```text
host bits = 32 - prefix length
```

Example:

```text
/20

host bits:
32 - 20 = 12
```

---

## 8. Number of Addresses

The total number of IPv4 addresses in a subnet is:

```text
2^(host bits)
```

Example:

```text
/24

host bits = 8

2^8 = 256 addresses
```

---

## 9. Common CIDR Sizes

| CIDR | Total IPv4 addresses |
|---|---:|
| /8 | 16,777,216 |
| /9 | 8,388,608 |
| /10 | 4,194,304 |
| /11 | 2,097,152 |
| /12 | 1,048,576 |
| /13 | 524,288 |
| /14 | 262,144 |
| /15 | 131,072 |
| /16 | 65,536 |
| /17 | 32,768 |
| /18 | 16,384 |
| /19 | 8,192 |
| /20 | 4,096 |
| /21 | 2,048 |
| /22 | 1,024 |
| /23 | 512 |
| /24 | 256 |
| /25 | 128 |
| /26 | 64 |
| /27 | 32 |
| /28 | 16 |
| /29 | 8 |
| /30 | 4 |
| /31 | 2 |
| /32 | 1 |

---

## 10. Usable Hosts in Traditional IPv4 Subnets

For ordinary subnetting:

```text
usable hosts = total addresses - 2
```

because traditionally:

```text
network address
broadcast address
```

are not assigned to hosts.

Example:

```text
/24
256 total
254 traditional usable
```

This rule has important exceptions such as `/31`, `/32`, and cloud-provider-specific subnet reservations.

---

## 11. /32

A `/32` represents exactly one IPv4 address:

```text
10.0.0.10/32
```

Common uses:

```text
host route
firewall rule
security rule
allow-list
routing
```

---

## 12. /31

A `/31` contains:

```text
2 addresses
```

It is commonly used for point-to-point links where both addresses can be used under RFC-defined point-to-point behavior.

Do not blindly apply the traditional:

```text
2 - 2 = 0
```

host formula to `/31`.

---

## 13. Subnet Mask

CIDR corresponds to a subnet mask.

Examples:

```text
/8  = 255.0.0.0
/16 = 255.255.0.0
/24 = 255.255.255.0
```

---

## 14. Prefix-to-Mask Table

| Prefix | Mask |
|---|---|
| /8 | 255.0.0.0 |
| /16 | 255.255.0.0 |
| /17 | 255.255.128.0 |
| /18 | 255.255.192.0 |
| /19 | 255.255.224.0 |
| /20 | 255.255.240.0 |
| /21 | 255.255.248.0 |
| /22 | 255.255.252.0 |
| /23 | 255.255.254.0 |
| /24 | 255.255.255.0 |
| /25 | 255.255.255.128 |
| /26 | 255.255.255.192 |
| /27 | 255.255.255.224 |
| /28 | 255.255.255.240 |
| /29 | 255.255.255.248 |
| /30 | 255.255.255.252 |

---

## 15. The Important Bit Values

An 8-bit octet uses:

```text
128 64 32 16 8 4 2 1
```

These values are the foundation of manual subnet calculations.

---

## 16. Subnet Mask Octet Values

Common partial masks:

```text
128
192
224
240
248
252
254
255
```

Mapping:

```text
10000000 = 128
11000000 = 192
11100000 = 224
11110000 = 240
11111000 = 248
11111100 = 252
11111110 = 254
11111111 = 255
```

---

## 17. Block Size

Block size is:

```text
256 - subnet-mask-octet
```

Example:

```text
/26
mask = 255.255.255.192

block size:
256 - 192 = 64
```

Networks occur every 64 addresses:

```text
0
64
128
192
```

---

## 18. /25 Block Size

```text
mask = 255.255.255.128

block:
256 - 128 = 128
```

Networks:

```text
0
128
```

---

## 19. /27 Block Size

```text
mask = 255.255.255.224

block:
256 - 224 = 32
```

Networks:

```text
0
32
64
96
128
160
192
224
```

---

## 20. /28 Block Size

```text
mask = 255.255.255.240

block:
256 - 240 = 16
```

Networks:

```text
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

---

## 21. Finding the Network Address

Example:

```text
192.168.10.77/26
```

Mask:

```text
255.255.255.192
```

Block size:

```text
64
```

Ranges:

```text
0–63
64–127
128–191
192–255
```

77 falls inside:

```text
64–127
```

Therefore:

```text
Network = 192.168.10.64
Broadcast = 192.168.10.127
```

---

## 22. Finding First and Last Host

For:

```text
192.168.10.77/26
```

network:

```text
192.168.10.64
```

broadcast:

```text
192.168.10.127
```

traditional usable range:

```text
192.168.10.65
-
192.168.10.126
```

---

## 23. Another Example

Given:

```text
172.16.35.200/20
```

Mask:

```text
255.255.240.0
```

Block size in third octet:

```text
256 - 240 = 16
```

Third-octet ranges:

```text
0–15
16–31
32–47
48–63
...
```

35 belongs to:

```text
32–47
```

Therefore:

```text
Network = 172.16.32.0/20
Broadcast = 172.16.47.255
```

---

## 24. /21 Example

Given:

```text
10.20.37.55/21
```

Mask:

```text
255.255.248.0
```

Block:

```text
256 - 248 = 8
```

Third-octet ranges:

```text
0–7
8–15
16–23
24–31
32–39
40–47
```

37 belongs to:

```text
32–39
```

Therefore:

```text
Network = 10.20.32.0/21
Broadcast = 10.20.39.255
```

---

## 25. /22 Example

Given:

```text
10.20.14.100/22
```

Mask:

```text
255.255.252.0
```

Block:

```text
4
```

Third-octet ranges:

```text
0–3
4–7
8–11
12–15
16–19
```

14 belongs to:

```text
12–15
```

Therefore:

```text
Network = 10.20.12.0/22
Broadcast = 10.20.15.255
```

---

## 26. /23 Example

Given:

```text
10.20.14.100/23
```

Mask:

```text
255.255.254.0
```

Block:

```text
2
```

Third-octet ranges:

```text
12–13
14–15
16–17
```

14 belongs to:

```text
14–15
```

Therefore:

```text
Network = 10.20.14.0/23
Broadcast = 10.20.15.255
```

---

## 27. /25 Example

Given:

```text
192.168.1.200/25
```

Ranges:

```text
0–127
128–255
```

Therefore:

```text
Network = 192.168.1.128
Broadcast = 192.168.1.255
Hosts = 192.168.1.129–254
```

---

## 28. /30 Example

```text
192.168.1.10/30
```

Block size:

```text
4
```

Ranges:

```text
8–11
```

Therefore:

```text
Network = 192.168.1.8
Broadcast = 192.168.1.11
Traditional usable:
192.168.1.9
192.168.1.10
```

---

## 29. Subnetting a /24 into /25

Original:

```text
192.168.1.0/24
```

Split into:

```text
192.168.1.0/25
192.168.1.128/25
```

Two equal subnets are created.

Each contains:

```text
128 addresses
```

---

## 30. Subnetting /24 into /26

A `/24` has 256 addresses.

A `/26` has 64 addresses.

Therefore:

```text
256 / 64 = 4
```

Subnets:

```text
192.168.1.0/26
192.168.1.64/26
192.168.1.128/26
192.168.1.192/26
```

---

## 31. Subnetting /24 into /27

```text
256 / 32 = 8
```

Subnets:

```text
.0/27
.32/27
.64/27
.96/27
.128/27
.160/27
.192/27
.224/27
```

---

## 32. Subnet Count Formula

If you borrow:

```text
n bits
```

from the host portion:

```text
number of subnets = 2^n
```

Example:

```text
/24 → /26
```

Borrowed:

```text
26 - 24 = 2 bits
```

Subnets:

```text
2^2 = 4
```

---

## 33. Host Capacity Formula

If:

```text
h = host bits
```

then:

```text
addresses = 2^h
```

Traditional usable hosts:

```text
2^h - 2
```

Again, cloud-specific reservations and special prefixes require separate consideration.

---

## 34. VLSM

VLSM means:

**Variable Length Subnet Masking**

It allows different subnet sizes within the same larger network.

Example:

```text
10.0.0.0/16
```

could be divided into:

```text
large subnet
medium subnet
small subnet
```

instead of forcing every subnet to be the same size.

---

## 35. Why VLSM Matters

VLSM reduces IP waste.

Example:

```text
Production:
needs 500 hosts

Monitoring:
needs 50 hosts

Management:
needs 10 hosts
```

Using identical huge subnets wastes address space.

---

## 36. VLSM Example

Required:

```text
500 hosts
100 hosts
50 hosts
20 hosts
```

Start with largest requirement.

500 hosts require at least:

```text
512 addresses
```

Therefore:

```text
/23
```

because:

```text
2^9 = 512
32 - 9 = 23
```

---

## 37. VLSM Planning Table

| Requirement | Minimum addresses | CIDR |
|---:|---:|---|
| 500 | 512 | /23 |
| 100 | 128 | /25 |
| 50 | 64 | /26 |
| 20 | 32 | /27 |

In real cloud environments, reserve additional capacity rather than planning only for today's host count.

---

## 38. Subnetting Strategy

For VLSM:

1. Sort requirements from largest to smallest.
2. Allocate the largest subnet first.
3. Continue sequentially.
4. Ensure boundaries are aligned.
5. Leave growth space.
6. Document every allocation.

---

## 39. Supernetting

Supernetting combines contiguous networks into a larger summarized prefix.

Example:

```text
10.0.0.0/24
10.0.1.0/24
10.0.2.0/24
10.0.3.0/24
```

can be summarized as:

```text
10.0.0.0/22
```

when alignment and contiguous requirements are satisfied.

---

## 40. Route Summarization

Route summarization reduces routing-table entries.

Instead of:

```text
10.0.0.0/24
10.0.1.0/24
10.0.2.0/24
10.0.3.0/24
```

advertise:

```text
10.0.0.0/22
```

This can improve:

```text
routing-table size
route management
network scalability
```

---

## 41. Longest Prefix Match

Routers choose the most specific matching route.

Example:

```text
10.0.0.0/8
10.10.0.0/16
10.10.20.0/24
```

For destination:

```text
10.10.20.50
```

the `/24` route wins because it is the longest matching prefix.

---

## 42. CIDR Overlap

Two CIDRs overlap when they share IP addresses.

Example:

```text
10.0.0.0/16
10.0.50.0/24
```

The second is contained inside the first.

This is an overlap.

---

## 43. Why CIDR Overlap Is Dangerous

Overlapping CIDRs can break:

```text
VPC peering
Transit Gateway routing
VPN connectivity
hybrid networking
Kubernetes network design
IP allow-lists
route summarization
```

Organizations should maintain a centralized IPAM strategy.

---

## 44. AWS VPC CIDR Planning

Example:

```text
VPC:
10.0.0.0/16
```

Possible subnet design:

```text
Public AZ-A:
10.0.0.0/24

Public AZ-B:
10.0.1.0/24

Private AZ-A:
10.0.10.0/20

Private AZ-B:
10.0.26.0/20
```

The exact plan should be based on workload and future growth.

---

## 45. AWS Subnet Sizing

AWS subnet sizing must account for AWS-reserved addresses.

For an IPv4 subnet, AWS reserves five IP addresses in each subnet, including:

```text
network address
VPC router
DNS
future use
broadcast address
```

AWS does not support IPv4 broadcast traffic, but the address is still reserved.

Therefore AWS usable IPv4 capacity differs from the traditional:

```text
2^host-bits - 2
```

formula.

---

## 46. AWS Subnet Example

For:

```text
10.0.1.0/24
```

total:

```text
256
```

AWS-reserved:

```text
5
```

available IPv4 addresses:

```text
251
```

Always verify current AWS documentation for service-specific behavior and limits.

---

## 47. EKS IP Planning

EKS can consume large numbers of IP addresses.

Depending on the networking mode, IPs may be consumed by:

```text
nodes
Pods
load balancer targets
network interfaces
secondary IPs
prefix allocations
```

AWS VPC CNI configuration strongly affects Pod IP consumption.

---

## 48. Why EKS CIDR Planning Matters

If a cluster runs out of assignable VPC IPs:

```text
new Pods may remain Pending
nodes may fail to obtain addresses
scaling can fail
deployments can stall
```

Therefore subnet capacity must be planned before production growth.

---

## 49. Pod IP Planning

Suppose:

```text
100 nodes
```

and each node may support:

```text
50 Pods
```

The theoretical Pod demand could approach:

```text
5,000 Pod IPs
```

Actual capacity depends on:

```text
instance type
ENI limits
prefix delegation
CNI configuration
reserved IPs
DaemonSets
system Pods
```

Do not use only the Pod count as the sizing formula.

---

## 50. Secondary CIDR Strategy

AWS VPCs can use additional CIDR blocks where supported.

This can help expand address space.

But adding CIDRs does not automatically solve every routing or workload IP allocation issue.

Plan:

```text
routing
subnet placement
CNI behavior
security rules
future connectivity
```

together.

---

## 51. Kubernetes Service CIDR

Kubernetes commonly uses a dedicated CIDR for ClusterIP Services.

Example:

```text
172.20.0.0/16
```

Pod CIDRs and Service CIDRs must not conflict with:

```text
VPC
on-premises
VPN
Transit Gateway
peered networks
```

---

## 52. Pod CIDR vs Service CIDR

### Pod CIDR

Used for:

```text
Pod IP allocation
```

### Service CIDR

Used for:

```text
virtual ClusterIP addresses
```

These represent different address spaces and have different networking behavior.

---

## 53. EKS VPC CNI Difference

With AWS VPC CNI, Pods commonly receive VPC-routable IP addresses from ENIs or delegated prefixes rather than using a traditional overlay Pod CIDR model.

Therefore:

```text
Pod IP capacity
=
VPC subnet capacity
+
CNI/IP allocation model
```

This is an important EKS-specific distinction.

---

## 54. CIDR and Security Groups

Security group rules can use CIDRs.

Example:

```text
10.0.0.0/16
```

allows traffic from the entire CIDR when the rule's protocol/port also matches.

Prefer:

```text
security-group references
```

where appropriate instead of broad CIDRs for AWS-to-AWS application communication.

---

## 55. CIDR and NACLs

NACLs are subnet-level controls and commonly use CIDR-based rules.

Example:

```text
10.0.0.0/16
```

A NACL rule can allow or deny traffic from that network according to protocol, port and direction.

Remember:

```text
NACL = stateless
Security Group = stateful
```

---

## 56. CIDR and NetworkPolicy

Kubernetes NetworkPolicy can use:

```yaml
ipBlock:
  cidr: 10.0.0.0/16
```

But for Pod-to-Pod policy, label selectors are often more maintainable than hard-coded IP ranges.

---

## 57. CIDR and Route Tables

A route can contain:

```text
destination CIDR
target
```

Example:

```text
10.20.0.0/16 → Transit Gateway
```

Routes are evaluated using longest-prefix matching.

---

## 58. Default Route

The IPv4 default route is:

```text
0.0.0.0/0
```

It matches destinations not covered by more specific routes.

Example:

```text
10.0.0.0/16 → local
0.0.0.0/0 → NAT Gateway
```

Traffic to:

```text
10.0.5.20
```

uses the more specific `/16` route.

Internet traffic uses the default route.

---

## 59. Host Route

A `/32` route represents one IPv4 address.

Example:

```text
10.10.10.50/32
```

It is more specific than:

```text
10.10.0.0/16
```

Therefore the `/32` wins for that destination.

---

## 60. CIDR Calculation Workflow

For any address:

```text
1. Identify prefix.
2. Calculate host bits.
3. Convert prefix to mask.
4. Find interesting octet.
5. Calculate block size.
6. Locate the containing range.
7. Determine network.
8. Determine broadcast.
9. Determine host range.
10. Verify mathematically.
```

---

## 61. Interesting Octet

The interesting octet is where the subnet mask is neither:

```text
255
```

nor:

```text
0
```

Example:

```text
255.255.240.0
```

interesting octet:

```text
third octet
```

This makes manual calculations much faster.

---

## 62. Fast CIDR Method

Example:

```text
172.31.44.90/20
```

Mask:

```text
255.255.240.0
```

Block:

```text
16
```

Third octet:

```text
44
```

Nearest lower multiple of 16:

```text
32
```

Therefore:

```text
Network:
172.31.32.0/20
```

Broadcast:

```text
172.31.47.255
```

---

## 63. Verify with Binary

For:

```text
172.31.44.90/20
```

third octet:

```text
44 = 00101100
```

mask:

```text
240 = 11110000
```

AND:

```text
00101100
11110000
--------
00100000
```

```text
00100000 = 32
```

Therefore:

```text
172.31.32.0
```

---

## 64. AND Operation

Network address can be calculated using:

```text
IP address
AND
subnet mask
=
network address
```

This is the fundamental mathematical operation behind subnet identification.

---

## 65. Broadcast Calculation

Once the network is known:

```text
broadcast = last address in subnet
```

For:

```text
10.10.32.0/20
```

the range is:

```text
32–47
```

Therefore:

```text
broadcast = 10.10.47.255
```

---

## 66. Finding the Next Subnet

For:

```text
192.168.10.0/26
```

block size:

```text
64
```

next subnet:

```text
192.168.10.64/26
```

Then:

```text
128
192
```

---

## 67. Finding Previous Subnet

For:

```text
192.168.10.128/26
```

previous subnet:

```text
192.168.10.64/26
```

The block size remains:

```text
64
```

---

## 68. Determining Whether an IP Belongs to a CIDR

Given:

```text
Network:
10.0.0.0/16

IP:
10.0.25.50
```

The IP is inside the CIDR because:

```text
10.0.x.x
```

matches the first 16 bits.

---

## 69. Python CIDR Check

Python's standard library provides:

```python
import ipaddress

network = ipaddress.ip_network("10.0.0.0/16")
ip = ipaddress.ip_address("10.0.25.50")

print(ip in network)
```

Output:

```text
True
```

---

## 70. Python Subnet Information

```python
import ipaddress

network = ipaddress.ip_network("10.0.10.0/24")

print(network.network_address)
print(network.broadcast_address)
print(network.netmask)
print(network.prefixlen)
print(network.num_addresses)
```

This is useful for automation and Terraform validation pipelines.

---

## 71. Python Subnet Creation

```python
import ipaddress

network = ipaddress.ip_network("10.0.0.0/24")

for subnet in network.subnets(prefixlen_diff=2):
    print(subnet)
```

This divides the `/24` into four `/26` networks.

---

## 72. Python Supernet

```python
import ipaddress

network = ipaddress.ip_network("10.0.0.0/24")
print(network.supernet())
```

Use supernetting carefully; summarization requires correct alignment and route semantics.

---

## 73. Terraform CIDR Functions

Terraform provides useful CIDR functions such as:

```text
cidrsubnet()
cidrhost()
cidrnetmask()
cidrhost()
```

These are extremely useful for automated AWS network design.

---

## 74. Terraform `cidrsubnet`

Example:

```hcl
variable "vpc_cidr" {
  default = "10.0.0.0/16"
}

locals {
  private_a = cidrsubnet(var.vpc_cidr, 4, 0)
  private_b = cidrsubnet(var.vpc_cidr, 4, 1)
}
```

The exact allocation should be designed intentionally rather than relying on arbitrary numbering.

---

## 75. Terraform CIDR Planning

Production Terraform should ideally derive:

```text
VPC CIDR
subnet CIDRs
AZ allocations
environment ranges
```

from a documented IP plan.

Avoid unexplained magic CIDRs scattered across modules.

---

## 76. Production IPAM

An organization should maintain:

```text
environment
region
account
VPC
CIDR
subnet
AZ
purpose
owner
```

in a central IPAM process.

AWS VPC IPAM can help centralize allocation and monitoring.

---

## 77. Environment CIDR Strategy

Example:

```text
DEV:
10.10.0.0/16

QA:
10.20.0.0/16

PROD:
10.30.0.0/16
```

This reduces overlap risk.

---

## 78. Multi-Account CIDR Strategy

Example:

```text
Shared:
10.0.0.0/16

Dev:
10.10.0.0/16

QA:
10.20.0.0/16

Prod:
10.30.0.0/16
```

The exact hierarchy should account for:

```text
regions
future accounts
future acquisitions
on-prem networks
partner networks
```

---

## 79. Multi-Region CIDR Planning

Example:

```text
us-east-1:
10.0.0.0/16

us-west-2:
10.1.0.0/16

ap-south-1:
10.2.0.0/16
```

This creates predictable routing boundaries.

---

## 80. Avoiding Overlap

Never casually allocate:

```text
VPC-A = 10.0.0.0/16
VPC-B = 10.0.0.0/16
```

if you expect direct routing between them.

Overlapping networks make routing ambiguous.

---

## 81. VPC Peering and CIDR

VPC peering requires non-overlapping IPv4 CIDRs for normal routing between the VPCs.

Before peering:

```text
check all VPC CIDRs
check secondary CIDRs
check connected networks
```

---

## 82. Transit Gateway and CIDR Planning

Transit Gateway environments become difficult when multiple attachments use overlapping networks.

Centralized IP planning is therefore essential.

A network that works independently today can become difficult to integrate tomorrow.

---

## 83. VPN and On-Premises CIDR

Suppose:

```text
AWS:
10.0.0.0/16

On-prem:
10.0.0.0/8
```

There is overlap.

Routing can become problematic.

A production hybrid network should use non-overlapping ranges whenever possible.

---

## 84. CIDR Design for EKS

Example:

```text
VPC:
10.0.0.0/16
```

Subnets:

```text
Public:
10.0.0.0/20
10.0.16.0/20
10.0.32.0/20

Private:
10.0.64.0/18
10.0.128.0/18
```

The actual design should reserve capacity for:

```text
nodes
Pods
load balancers
future AZs
future clusters
```

---

## 85. EKS IP Exhaustion Symptoms

Common symptoms:

```text
Pods Pending
FailedCreatePodSandBox
CNI errors
IP allocation errors
node scaling issues
```

Check:

```bash
kubectl describe pod <pod>
kubectl get nodes
kubectl -n kube-system logs -l k8s-app=aws-node
```

Also inspect AWS subnet free IP capacity.

---

## 86. CIDR Planning and Autoscaling

Autoscaling increases:

```text
nodes
Pods
network interfaces
IP allocations
```

Therefore subnet sizing must account for:

```text
peak capacity
not average capacity
```

---

## 87. Capacity Planning Formula

A simplified planning approach:

```text
Required IPs =
expected node IPs
+
expected Pod IPs
+
load balancer/network interface needs
+
system overhead
+
growth buffer
```

The exact EKS VPC CNI allocation model must be included.

---

## 88. Growth Buffer

Do not allocate a subnet so tightly that:

```text
current usage = 90–95%
```

before production launch.

Leave operational headroom.

A commonly used planning principle is to reserve meaningful unused capacity for bursts and scaling.

---

## 89. Subnet Fragmentation

Many small allocations can create fragmented address space.

Example:

```text
10.0.0.0/28
10.0.0.32/28
10.0.0.64/28
...
```

Later you may have enough total IPs but not enough contiguous CIDR space for a required subnet.

Plan address blocks hierarchically.

---

## 90. Hierarchical Allocation

Example:

```text
10.0.0.0/8
|
+-- 10.10.0.0/16 DEV
+-- 10.20.0.0/16 QA
+-- 10.30.0.0/16 PROD
```

Then:

```text
PROD /16
|
+-- AZ-A /20
+-- AZ-B /20
+-- AZ-C /20
```

This makes future planning easier.

---

## 91. AZ-Aware Subnet Allocation

Production EKS should normally span multiple AZs.

Example:

```text
AZ-A:
10.30.0.0/20

AZ-B:
10.30.16.0/20

AZ-C:
10.30.32.0/20
```

Each subnet should be large enough for its expected workload.

---

## 92. Public vs Private CIDRs

Example:

```text
VPC:
10.30.0.0/16

Public:
10.30.0.0/20
10.30.16.0/20

Private:
10.30.64.0/20
10.30.80.0/20
```

The routing policy, not the CIDR alone, determines whether a subnet is public or private.

---

## 93. What Makes a Subnet Public?

A subnet is commonly called public when its route table has a route to an Internet Gateway:

```text
0.0.0.0/0 → Internet Gateway
```

and resources have suitable public addressing.

A CIDR such as:

```text
10.0.0.0/24
```

does not itself make a subnet public.

---

## 94. Private Subnet

A private subnet commonly has:

```text
0.0.0.0/0 → NAT Gateway
```

for outbound Internet access.

It does not directly route Internet traffic through an Internet Gateway for private IPv4 addresses.

---

## 95. CIDR and ALB

An ALB can have nodes/interfaces in selected subnets.

Production ALB subnet planning should consider:

```text
multiple AZs
available IP capacity
routing
security groups
future scaling
```

---

## 96. ALB IP Capacity

Load balancers can consume addresses in their subnets.

Do not size production subnets solely around current EC2 instances.

Managed services also consume network resources.

---

## 97. CIDR and NLB

NLB networking also requires subnet capacity.

High-scale environments should account for:

```text
NLB addresses
targets
nodes
future scale
```

and AWS-specific behavior.

---

## 98. CIDR and NAT Gateway

NAT Gateway requires subnet placement and public connectivity.

Typical architecture:

```text
Private subnet
   |
NAT Gateway
   |
Public subnet
   |
Internet Gateway
```

CIDR planning must keep these subnet allocations non-overlapping.

---

## 99. CIDR and Route 53

Route 53 Resolver and private hosted zones operate in the VPC context.

CIDR planning affects:

```text
DNS reachability
hybrid DNS
resolver endpoints
security controls
```

DNS and IP planning should therefore be designed together.

---

## 100. CIDR and Security Boundaries

CIDRs should reflect meaningful trust boundaries where practical.

Example:

```text
10.30.64.0/20
```

may represent application private subnets.

Avoid treating one huge CIDR as universally trusted if workloads have different security requirements.

---

## 101. CIDR and Zero Trust

Zero-trust design does not assume:

```text
same CIDR = trusted
```

Identity and explicit policy should determine access.

CIDRs are useful network selectors but are not identities.

---

## 102. CIDR in Firewall Rules

Example:

```text
allow TCP 443
source:
10.20.0.0/16
```

This permits all matching addresses in that range.

A narrower:

```text
10.20.10.0/24
```

reduces the source range.

---

## 103. CIDR Security Mistake

Bad:

```text
0.0.0.0/0 → TCP 22
```

This exposes SSH globally.

Better:

```text
VPN CIDR → TCP 22
```

or use a managed access mechanism such as AWS Systems Manager where appropriate.

---

## 104. CIDR Validation in CI

Infrastructure pipelines can validate:

```text
overlapping subnets
invalid CIDRs
wrong AZ allocations
insufficient capacity
environment overlap
```

before Terraform applies infrastructure.

---

## 105. CIDR Automation

Example Python validation:

```python
import ipaddress

networks = [
    ipaddress.ip_network("10.10.0.0/16"),
    ipaddress.ip_network("10.20.0.0/16"),
]

for i, a in enumerate(networks):
    for b in networks[i + 1:]:
        if a.overlaps(b):
            print("OVERLAP:", a, b)
```

---

## 106. CIDR Overlap Automation

Terraform can also model networks declaratively and fail validation when ranges violate the architecture.

This should be part of infrastructure quality checks in large environments.

---

## 107. Production IPAM Checklist

Maintain:

```text
[ ] VPC CIDR
[ ] Region
[ ] Account
[ ] Environment
[ ] AZ
[ ] Public/private
[ ] Subnet CIDR
[ ] Reserved capacity
[ ] EKS capacity
[ ] On-prem ranges
[ ] Partner ranges
[ ] TGW attachments
[ ] VPN ranges
[ ] Peering ranges
```

---

## 108. Manual Calculation Exercise 1

Calculate:

```text
192.168.20.77/27
```

Block:

```text
32
```

Ranges:

```text
64–95
```

Answer:

```text
Network:
192.168.20.64

Broadcast:
192.168.20.95

Traditional hosts:
192.168.20.65–94
```

---

## 109. Manual Calculation Exercise 2

Calculate:

```text
10.50.100.25/20
```

Block in third octet:

```text
16
```

100 belongs to:

```text
96–111
```

Answer:

```text
Network:
10.50.96.0/20

Broadcast:
10.50.111.255
```

---

## 110. Manual Calculation Exercise 3

Calculate:

```text
172.16.200.20/22
```

Block:

```text
4
```

200 belongs to:

```text
200–203
```

Answer:

```text
Network:
172.16.200.0/22

Broadcast:
172.16.203.255
```

---

## 111. Manual Calculation Exercise 4

Calculate:

```text
10.10.10.200/28
```

Block:

```text
16
```

Ranges include:

```text
192–207
```

Answer:

```text
Network:
10.10.10.192

Broadcast:
10.10.10.207
```

---

## 112. Manual Calculation Exercise 5

Calculate:

```text
172.20.77.10/18
```

Mask:

```text
255.255.192.0
```

Block:

```text
64
```

Third-octet ranges:

```text
0–63
64–127
128–191
192–255
```

Answer:

```text
Network:
172.20.64.0/18

Broadcast:
172.20.127.255
```

---

## 113. Manual Calculation Exercise 6

How many `/26` subnets fit into `/22`?

Difference:

```text
26 - 22 = 4
```

Therefore:

```text
2^4 = 16
```

Answer:

```text
16 /26 subnets
```

---

## 114. Manual Calculation Exercise 7

How many addresses are in:

```text
10.0.0.0/19
```

Host bits:

```text
32 - 19 = 13
```

Addresses:

```text
2^13 = 8192
```

Traditional usable:

```text
8190
```

---

## 115. Manual Calculation Exercise 8

How many addresses are in:

```text
10.0.0.0/28
```

Host bits:

```text
4
```

Addresses:

```text
16
```

Traditional usable:

```text
14
```

---

## 116. Manual Calculation Exercise 9

Which CIDR is larger?

```text
10.0.0.0/20
10.0.0.0/24
```

Answer:

```text
/20
```

A smaller prefix number means more host bits and therefore a larger address block.

---

## 117. Manual Calculation Exercise 10

Does:

```text
10.10.50.0/24
```

fit inside:

```text
10.10.0.0/16
```

Yes.

The `/16` covers:

```text
10.10.0.0
through
10.10.255.255
```

---

## 118. Manual Calculation Exercise 11

Does:

```text
10.11.50.0/24
```

fit inside:

```text
10.10.0.0/16
```

No.

The second octet differs:

```text
10.11
vs
10.10
```

---

## 119. Manual Calculation Exercise 12

Summarize:

```text
10.10.0.0/24
10.10.1.0/24
```

They can be summarized as:

```text
10.10.0.0/23
```

because they are contiguous and correctly aligned.

---

## 120. Manual Calculation Exercise 13

Can:

```text
10.10.1.0/24
10.10.2.0/24
```

be summarized as:

```text
10.10.0.0/22
```

No.

A `/22` covers:

```text
10.10.0.0–10.10.3.255
```

and would include an additional network.

Correct summarization depends on the exact set of routes and alignment.

---

## 121. Interview: What Is CIDR?

CIDR is a classless IP addressing and routing method represented by a prefix length such as `/24`. It allows flexible allocation and route aggregation.

---

## 122. Interview: How Many Addresses Does /24 Have?

```text
2^(32-24)
=
2^8
=
256
```

---

## 123. Interview: How Many Hosts Does /24 Have?

Traditional answer:

```text
254
```

But cloud platforms may reserve addresses, so AWS subnet usable capacity is different.

---

## 124. Interview: How Do You Calculate Network Address?

Use:

```text
IP AND subnet mask
```

or the block-size method.

---

## 125. Interview: What Is Block Size?

Block size determines the increments at which subnet boundaries occur.

Formula:

```text
256 - interesting-octet mask
```

---

## 126. Interview: What Is VLSM?

VLSM allows different subnet sizes within the same address space, reducing IP waste and supporting efficient allocation.

---

## 127. Interview: What Is Supernetting?

Supernetting combines multiple contiguous, correctly aligned networks into a larger summarized prefix.

---

## 128. Interview: What Is Route Summarization?

Route summarization advertises a larger aggregate prefix instead of multiple smaller prefixes, reducing routing-table complexity.

---

## 129. Interview: What Is Longest Prefix Match?

The most specific matching route wins.

Example:

```text
10.0.0.0/8
10.10.0.0/16
10.10.10.0/24
```

For `10.10.10.50`, `/24` wins.

---

## 130. Interview: Why Is CIDR Overlap Bad?

Overlapping networks make routing and connectivity between environments difficult or impossible, especially with:

```text
VPC peering
Transit Gateway
VPN
hybrid networks
```

---

## 131. Interview: How Would You Design CIDRs for Dev, QA and Prod?

Example:

```text
DEV  = 10.10.0.0/16
QA   = 10.20.0.0/16
PROD = 10.30.0.0/16
```

Then divide each environment into AZ-aware public/private subnet ranges with growth capacity.

---

## 132. Interview: Why Is EKS CIDR Planning Important?

Because AWS VPC CNI commonly assigns VPC-routable IP addresses to Pods. Insufficient subnet capacity can prevent Pod scheduling and cluster scaling.

---

## 133. Interview: Pod CIDR vs Service CIDR?

Pod CIDR is associated with Pod addressing in Kubernetes networking.

Service CIDR is used for virtual ClusterIP addresses.

In EKS with AWS VPC CNI, Pod addresses commonly come from VPC networking rather than a traditional overlay-only Pod CIDR design.

---

## 134. Interview: What Is AWS's IPv4 Subnet Reservation?

AWS reserves five IPv4 addresses in each subnet, so the usable count differs from the traditional `2^host-bits - 2` calculation.

---

## 135. Interview: What Is 0.0.0.0/0?

It is the IPv4 default route and matches all IPv4 destinations.

More-specific routes take precedence.

---

## 136. Interview: What Is a /32?

A single IPv4 address.

Common uses:

```text
host routes
allow-lists
security rules
```

---

## 137. Interview: What Is a /31?

A two-address IPv4 prefix commonly used for point-to-point links under appropriate RFC behavior.

---

## 138. Interview: How Do You Prevent CIDR Overlap in Terraform?

Use:

```text
central IP plan
variables
validation
CIDR functions
automated overlap checks
```

and make the pipeline fail before infrastructure creation.

---

## 139. Production Scenario: VPC Peering Failure

### Symptom

VPC peering exists but traffic cannot be routed correctly.

### Investigation

Check:

```text
VPC CIDRs
route tables
security groups
NACLs
```

First verify that CIDRs do not overlap.

---

## 140. Production Scenario: EKS Pods Pending

### Symptom

New Pods remain:

```text
Pending
```

### Investigation

```bash
kubectl describe pod <pod>
kubectl -n kube-system logs -l k8s-app=aws-node
```

Check:

```text
subnet free IPs
ENI/IP capacity
CNI configuration
prefix delegation
instance limits
```

---

## 141. Production Scenario: New Private Subnet Cannot Be Created

Possible cause:

```text
parent CIDR is fragmented
```

The VPC may have enough total addresses but no suitable contiguous free CIDR block.

This demonstrates why hierarchical IP planning matters.

---

## 142. Production Scenario: Hybrid VPN Routing Conflict

AWS:

```text
10.50.0.0/16
```

On-prem:

```text
10.50.0.0/16
```

Traffic between them is ambiguous.

Correct long-term solution is normally:

```text
renumber
or
introduce carefully designed NAT/translation architecture
```

rather than trying to force overlapping routes.

---

## 143. Production Scenario: Security Rule Too Broad

Rule:

```text
TCP 5432
source:
10.0.0.0/8
```

This may expose PostgreSQL to far more systems than required.

Better:

```text
application security group
```

or a narrowly scoped source CIDR.

---

## 144. Production Scenario: EKS IP Exhaustion

Symptoms:

```text
FailedCreatePodSandBox
IP allocation errors
Pods pending
```

Response:

```text
1. Check subnet available IPs.
2. Check aws-node logs.
3. Check ENI/IP limits.
4. Check prefix delegation.
5. Check node instance types.
6. Expand/add subnet capacity if architecture allows.
7. Rebalance workloads.
```

---

## 145. Production Scenario: ALB Scaling and Subnet Capacity

An ALB requires subnet capacity for its infrastructure.

If subnet IP capacity is critically low, managed load-balancer behavior and scaling can be affected.

Production subnets should retain sufficient headroom.

---

## 146. CIDR Design for RoboShop

Example:

```text
RoboShop PROD VPC
10.30.0.0/16
```

Public:

```text
10.30.0.0/20
10.30.16.0/20
10.30.32.0/20
```

Private application:

```text
10.30.64.0/20
10.30.80.0/20
10.30.96.0/20
```

Data/services:

```text
10.30.128.0/20
10.30.144.0/20
```

These are examples for planning, not universal production values.

---

## 147. RoboShop Network Capacity

Plan capacity for:

```text
EKS nodes
Pods
ALB
NAT Gateway
internal services
monitoring
logging
future scaling
```

Do not size only around the current number of EC2 nodes.

---

## 148. CIDR Documentation

Every production network should document:

```text
CIDR
purpose
environment
region
AZ
route table
owner
allocation date
reserved range
future expansion
```

This prevents accidental reuse.

---

## 149. Network Change Review

Before changing a CIDR, review:

```text
routing
security groups
NACLs
NetworkPolicies
DNS
VPN
TGW
VPC peering
EKS
load balancers
Terraform
monitoring
```

IP changes are architecture changes, not just configuration edits.

---

## 150. Production CIDR Checklist

```text
[ ] No overlaps
[ ] Growth capacity
[ ] Multi-AZ design
[ ] Public/private separation
[ ] EKS Pod capacity
[ ] Load balancer capacity
[ ] NAT architecture
[ ] Hybrid connectivity
[ ] TGW connectivity
[ ] Peering compatibility
[ ] Security boundaries
[ ] IPAM documentation
[ ] Terraform validation
[ ] Disaster recovery ranges
```

---

## 151. Quick Reference: Prefix Size

```text
/16 = 65,536 addresses
/17 = 32,768
/18 = 16,384
/19 = 8,192
/20 = 4,096
/21 = 2,048
/22 = 1,024
/23 = 512
/24 = 256
/25 = 128
/26 = 64
/27 = 32
/28 = 16
/29 = 8
/30 = 4
/31 = 2
/32 = 1
```

---

## 152. Quick Reference: Host Bits

```text
/16 → 16 host bits
/20 → 12 host bits
/21 → 11 host bits
/22 → 10 host bits
/23 → 9 host bits
/24 → 8 host bits
/25 → 7 host bits
/26 → 6 host bits
/27 → 5 host bits
/28 → 4 host bits
/29 → 3 host bits
/30 → 2 host bits
```

---

## 153. Quick Reference: Block Size

```text
/17 → 128
/18 → 64
/19 → 32
/20 → 16
/21 → 8
/22 → 4
/23 → 2
/24 → 1
/25 → 128
/26 → 64
/27 → 32
/28 → 16
/29 → 8
/30 → 4
```

The block applies to the first octet in which the subnet mask is partial.

---

## 154. CIDR Mental Shortcut

Remember:

```text
/24
↓
8 host bits
↓
256 addresses

Every +1 prefix bit
↓
half the address space
```

Therefore:

```text
/24 = 256
/25 = 128
/26 = 64
/27 = 32
/28 = 16
```

---

## 155. Production Mental Model

Think of IP planning as:

```text
Organization
    |
    +-- Accounts
    |
    +-- Regions
    |
    +-- VPCs
    |
    +-- AZs
    |
    +-- Subnets
    |
    +-- Nodes
    |
    +-- Pods
    |
    +-- Services
```

Every layer needs a deliberate address strategy.

---

## 156. Final Summary

CIDR is not just an interview topic.

It determines how production infrastructure communicates.

You should be able to:

```text
calculate network
calculate broadcast
calculate host range
calculate subnet count
calculate address capacity
design VLSM
summarize routes
detect overlaps
size AWS subnets
plan EKS IP capacity
design multi-account CIDRs
design multi-region CIDRs
automate validation
troubleshoot IP exhaustion
```

The most important formulas are:

```text
host bits = 32 - prefix

addresses = 2^host_bits

traditional usable IPv4 hosts = addresses - 2

subnet count = 2^borrowed_bits

block size = 256 - interesting-octet mask
```

And the most important production principle is:

```text
PLAN IP SPACE BEFORE DEPLOYING THE INFRASTRUCTURE.
```

---

## 157. Next File

The next file in the planned Networking sequence is:

```text
07-TCP-UDP-and-Ports.md
```

It will cover TCP/UDP and ports as a dedicated topic, while the following file will go deeply into:

```text
08-TCP-Three-Way-Handshake.md
```

This keeps the planned structure intact.

# End of 06-CIDR-and-Subnet-Calculations.md
