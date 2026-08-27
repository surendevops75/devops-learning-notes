# ARP-and-ICMP

## 1. Purpose

ARP and ICMP are foundational networking mechanisms that DevOps engineers encounter when troubleshooting Linux, AWS, Kubernetes, and EKS environments.

This file covers:

- ARP fundamentals
- ARP request/reply
- ARP cache
- gratuitous ARP
- proxy ARP
- ARP failure troubleshooting
- IPv6 Neighbor Discovery
- ICMP fundamentals
- ICMPv4 and ICMPv6
- ping
- traceroute
- TTL
- Time Exceeded
- Destination Unreachable
- Path MTU Discovery
- ICMP security
- AWS/EKS implications
- production troubleshooting
- RoboShop scenarios
- interview preparation

---

## 2. What Is ARP?

ARP stands for:

```text
Address Resolution Protocol
```

It maps an IPv4 address to a Layer-2 MAC address on a local network.

Conceptually:

```text
IPv4:
10.0.1.20

ARP:
10.0.1.20 → aa:bb:cc:dd:ee:ff
```

---

## 3. Why ARP Is Needed

IP operates at Layer 3.

Ethernet delivery requires a destination MAC address.

Therefore, when a host knows:

```text
destination IP
```

but needs:

```text
destination MAC
```

ARP can resolve it on the local IPv4 network.

---

## 4. ARP Scope

ARP is local-link technology.

A host does not ARP across the internet for a remote server.

For a remote destination, it normally resolves the MAC address of the local next-hop gateway.

Example:

```text
Client
10.0.1.10
   |
   | ARP for gateway
   v
10.0.1.1
```

The router then forwards the IP packet onward.

---

## 5. Local Destination vs Remote Destination

Suppose:

```text
Client:
10.0.1.10/24

Destination A:
10.0.1.20

Destination B:
10.0.2.20
```

Destination A is local to:

```text
10.0.1.0/24
```

The client ARPs for:

```text
10.0.1.20
```

Destination B is remote.

The client normally ARPs for:

```text
10.0.1.1
```

the configured next-hop gateway.

---

## 6. ARP Workflow

Basic workflow:

```text
Host A
  |
  | ARP Request
  | "Who has 10.0.1.20?"
  |
  v
Local Network
  |
  | ARP Reply
  | "10.0.1.20 is at MAC-B"
  |
  v
Host A
```

Host A stores the result temporarily.

---

## 7. ARP Request

An ARP request asks:

```text
Who has IP address X?
```

The request is generally sent as a Layer-2 broadcast on the local Ethernet segment.

Typical Ethernet destination:

```text
ff:ff:ff:ff:ff:ff
```

---

## 8. ARP Reply

The host owning the requested IP responds with its MAC address.

Example:

```text
10.0.1.20 is-at 02:11:22:33:44:55
```

The sender stores this mapping in its neighbor/ARP cache.

---

## 9. ARP Cache

Hosts do not normally perform ARP for every packet.

They maintain a cache:

```text
IP → MAC
```

Linux:

```bash
ip neigh
```

Example:

```text
10.0.1.1 dev eth0 lladdr 02:aa:bb:cc:dd:ee REACHABLE
```

---

## 10. Why ARP Caching Matters

Caching reduces:

```text
broadcast traffic
latency
CPU processing
```

Without caching, every packet could require address resolution.

---

## 11. ARP Cache States

Linux neighbor entries can show states such as:

```text
REACHABLE
STALE
DELAY
PROBE
FAILED
INCOMPLETE
PERMANENT
```

Exact state transitions depend on kernel neighbor discovery behavior.

---

## 12. REACHABLE

Example:

```text
10.0.1.1 dev eth0 lladdr ... REACHABLE
```

The neighbor is considered recently reachable.

---

## 13. STALE

A stale entry is not necessarily broken.

It means the cached information has not recently been confirmed.

Traffic can trigger neighbor validation.

---

## 14. INCOMPLETE

Example:

```text
10.0.1.1 dev eth0 INCOMPLETE
```

The system is attempting to resolve the neighbor but has not received a usable response.

---

## 15. FAILED

Example:

```text
10.0.1.1 dev eth0 FAILED
```

Neighbor resolution has failed.

Potential causes:

```text
wrong IP
interface issue
VLAN issue
security filtering
network path issue
unreachable gateway
```

---

## 16. Inspect ARP/Neighbor Cache

Linux:

```bash
ip neigh
```

IPv4-focused:

```bash
ip -4 neigh
```

IPv6:

```bash
ip -6 neigh
```

---

## 17. Legacy ARP Command

Some systems provide:

```bash
arp -n
```

However, modern Linux troubleshooting should generally prefer:

```bash
ip neigh
```

---

## 18. Add a Static Neighbor Entry

Linux can configure a permanent neighbor entry:

```bash
sudo ip neigh replace 10.0.1.20 lladdr aa:bb:cc:dd:ee:ff dev eth0 nud permanent
```

Use carefully.

A wrong static MAC can break connectivity.

---

## 19. Delete a Neighbor Entry

```bash
sudo ip neigh del 10.0.1.20 dev eth0
```

Only do this when you understand the impact.

---

## 20. Flush Neighbor Entries

```bash
sudo ip neigh flush dev eth0
```

This can temporarily cause new ARP/neighbor resolution and should be used carefully on production systems.

---

## 21. ARP Broadcast Domain

ARP broadcasts stay within the local Layer-2 broadcast domain.

Routers normally do not forward ordinary ARP broadcasts between IP networks.

---

## 22. ARP and VLANs

Different VLANs normally represent separate Layer-2 broadcast domains.

Therefore:

```text
VLAN 10
```

does not normally receive ARP broadcasts from:

```text
VLAN 20
```

Routing between them requires a Layer-3 device.

---

## 23. ARP and Default Gateway

For remote destinations:

```text
Destination:
8.8.8.8

Route:
0.0.0.0/0 via 10.0.1.1

ARP:
10.0.1.1 → gateway MAC
```

The packet's IP destination remains:

```text
8.8.8.8
```

while the Ethernet destination is the gateway's MAC.

---

## 24. Important ARP Mental Model

For a remote destination:

```text
IP destination:
remote server

MAC destination:
local next hop
```

This distinction is extremely important in troubleshooting.

---

## 25. ARP Packet Fields

An ARP message contains information such as:

```text
hardware type
protocol type
hardware address length
protocol address length
operation
sender MAC
sender IP
target MAC
target IP
```

---

## 26. ARP Operation Codes

Common operations:

```text
1 = request
2 = reply
```

---

## 27. Gratuitous ARP

Gratuitous ARP is an ARP message sent without a normal request for resolution.

It can be used to:

```text
announce an IP/MAC association
refresh neighbor information
detect duplicate IPs
support failover
```

---

## 28. Gratuitous ARP in High Availability

Example:

```text
Node A owns VIP
        |
        v
10.0.1.100 → MAC-A

Failover

Node B owns VIP
        |
        v
10.0.1.100 → MAC-B
```

Node B can announce the new mapping so peers update their caches.

---

## 29. VRRP and ARP

High-availability systems using a virtual IP can use ARP announcements to help peers learn the current active MAC.

Common technologies include:

```text
VRRP
Keepalived
```

---

## 30. ARP Cache Poisoning

ARP has limited authentication by default.

An attacker on the same Layer-2 network may attempt to send forged ARP responses.

This can cause:

```text
traffic interception
man-in-the-middle
traffic disruption
```

---

## 31. ARP Spoofing

Example:

```text
Victim
  |
  | believes gateway MAC = attacker
  v
Attacker
  |
Gateway
```

The attacker can potentially intercept traffic.

---

## 32. ARP Security Controls

Depending on the environment:

```text
DHCP snooping
Dynamic ARP Inspection
port security
network segmentation
switch protections
```

can reduce ARP-based attacks.

Cloud networking may provide different controls and abstractions.

---

## 33. ARP in AWS

AWS VPC networking abstracts physical Layer-2 infrastructure from customers.

You normally do not administer a traditional Ethernet switch and ARP domain directly.

However, the Linux guest still has neighbor behavior and network interfaces, while AWS implements the underlying virtual network.

---

## 34. ARP Troubleshooting in EC2

Useful:

```bash
ip addr
ip route
ip neigh
```

Then test:

```bash
ping <gateway-or-destination>
```

and:

```bash
tcpdump -ni eth0 arp
```

if packet capture is available and appropriate.

---

## 35. AWS ENI and Neighbor Behavior

Elastic Network Interfaces provide virtual network interfaces to EC2 and other AWS networking components.

Linux sees an interface and IP configuration, while AWS controls the underlying virtual network implementation.

---

## 36. Why Traditional ARP Troubleshooting Is Different in AWS

You generally should not assume:

```text
traditional physical Ethernet behavior
```

for AWS virtual networking.

Use AWS-native evidence:

```text
route tables
security groups
NACLs
VPC Flow Logs
Reachability Analyzer
```

along with host-level tools.

---

## 37. ARP and Security Groups

AWS Security Groups operate as stateful virtual firewalls.

Do not assume a normal ARP request/reply is equivalent to an application TCP flow being allowed or denied by a Security Group.

Troubleshoot the AWS network path using AWS-native tools.

---

## 38. IPv6 Does Not Use ARP

IPv6 does not use ARP.

Instead, IPv6 uses:

```text
Neighbor Discovery Protocol
```

which operates through ICMPv6.

---

## 39. Neighbor Discovery Protocol

NDP provides functions including:

```text
address resolution
router discovery
prefix discovery
neighbor reachability
duplicate address detection
```

---

## 40. NDP Message Types

Important ICMPv6 messages include:

```text
Router Solicitation
Router Advertisement
Neighbor Solicitation
Neighbor Advertisement
Redirect
```

---

## 41. Neighbor Solicitation

IPv6 Neighbor Solicitation is roughly analogous to ARP resolution.

It asks:

```text
Who owns this IPv6 address?
```

It is sent using ICMPv6.

---

## 42. Neighbor Advertisement

The destination responds with information about its link-layer address/reachability.

---

## 43. Router Solicitation

A host can send:

```text
Router Solicitation
```

to discover routers on the local IPv6 network.

---

## 44. Router Advertisement

Routers can send:

```text
Router Advertisement
```

containing information such as:

```text
prefix
default router
configuration flags
```

---

## 45. Duplicate Address Detection

IPv6 uses Neighbor Discovery to help detect whether an address is already in use.

This is called:

```text
DAD
```

Duplicate Address Detection.

---

## 46. IPv6 Neighbor Cache

Linux:

```bash
ip -6 neigh
```

It serves a role similar to IPv4's ARP/neighbor cache.

---

## 47. ICMP

ICMP stands for:

```text
Internet Control Message Protocol
```

It is used for network control, diagnostics, and error reporting.

---

## 48. ICMP Is Not TCP

ICMP is a network-layer control protocol.

It does not provide:

```text
TCP-style reliable byte stream
```

or:

```text
TCP ports
```

---

## 49. ICMP Echo Request

`ping` commonly sends:

```text
ICMP Echo Request
```

---

## 50. ICMP Echo Reply

The destination can respond with:

```text
ICMP Echo Reply
```

This allows basic reachability testing.

---

## 51. Ping

Basic:

```bash
ping 8.8.8.8
```

IPv6:

```bash
ping6 2001:4860:4860::8888
```

Modern Linux may also support:

```bash
ping -6
```

---

## 52. What Ping Proves

A successful ping generally proves:

```text
ICMP request reached destination
ICMP response returned
```

It does not prove:

```text
TCP/443 works
HTTP works
application is healthy
TLS works
```

---

## 53. What Ping Failure Does Not Prove

Ping can fail because ICMP is:

```text
blocked
rate limited
disabled
filtered
```

while TCP/HTTPS still works.

---

## 54. Ping With Packet Size

Example:

```bash
ping -s 1400 <destination>
```

Useful for testing packet-size-related behavior.

---

## 55. Don't Overinterpret Ping

This:

```text
ping fails
```

does not automatically mean:

```text
network is down
```

Test the actual application protocol.

---

## 56. ICMP Error Messages

ICMP can report problems such as:

```text
Destination Unreachable
Time Exceeded
Parameter Problem
```

Specific types/codes differ between ICMPv4 and ICMPv6.

---

## 57. Destination Unreachable

ICMP Destination Unreachable can indicate conditions such as:

```text
network unreachable
host unreachable
protocol unreachable
port unreachable
fragmentation needed
```

The exact meaning depends on ICMP version/type/code.

---

## 58. Port Unreachable

For UDP, an ICMP Port Unreachable can indicate that the destination does not have a listener for that UDP port.

This is one reason traceroute implementations can identify the end of a path.

---

## 59. Time Exceeded

Routers decrement IPv4 TTL.

When TTL reaches zero:

```text
ICMP Time Exceeded
```

can be returned.

Traceroute uses this behavior.

---

## 60. TTL

TTL stands for:

```text
Time To Live
```

in IPv4.

It is decremented by routers.

It limits indefinite forwarding loops.

---

## 61. IPv6 Hop Limit

IPv6 uses:

```text
Hop Limit
```

instead of TTL.

The operational concept is similar.

---

## 62. Traceroute

Traceroute discovers intermediate hops by sending packets with controlled TTL/Hop Limit values.

Example:

```bash
traceroute example.com
```

---

## 63. Traceroute First Hop

With TTL:

```text
1
```

the first router decrements it to zero and may send:

```text
ICMP Time Exceeded
```

The source learns that router's address.

---

## 64. Traceroute Second Hop

Next packet uses:

```text
TTL 2
```

The first router forwards it.

The second router decrements TTL to zero and may reply.

---

## 65. Traceroute Visualization

```text
Source
  |
  | TTL=1
  v
Router 1
  |
  | TTL=2
  v
Router 2
  |
  | TTL=3
  v
Router 3
  |
Destination
```

---

## 66. Traceroute Does Not Always Show Every Hop

A hop may appear as:

```text
*
```

because:

```text
ICMP blocked
rate limited
device configured not to respond
packet lost
```

---

## 67. TCP Traceroute

Useful when ICMP/UDP is filtered:

```bash
traceroute -T -p 443 example.com
```

This sends TCP probes toward port 443.

---

## 68. UDP Traceroute

Traditional implementations may use UDP probes.

Exact behavior varies by OS/tool.

---

## 69. ICMP Traceroute

Some implementations support:

```bash
traceroute -I example.com
```

which uses ICMP Echo probes.

---

## 70. `tracepath`

Linux:

```bash
tracepath example.com
```

It can provide path information and may also expose MTU-related information.

---

## 71. `mtr`

Run:

```bash
mtr -rw example.com
```

MTR repeatedly tests the path.

Useful for:

```text
latency
loss
path stability
```

---

## 72. Packet Loss at an Intermediate Hop

Suppose:

```text
Router 3 = 20% loss
Router 4 = 0% loss
Destination = 0% loss
```

This may indicate Router 3 is rate-limiting ICMP responses rather than actually dropping transit traffic.

Do not conclude that the hop is causing application loss without end-to-end evidence.

---

## 73. Packet Loss at Destination

If:

```text
intermediate hops = 0%
destination = 20%
```

the loss is more significant and should be investigated further.

Still verify with application/TCP measurements.

---

## 74. ICMP Rate Limiting

Routers may rate-limit ICMP control responses.

Therefore:

```text
ICMP response loss
```

does not necessarily mean:

```text
packet forwarding loss
```

---

## 75. ICMP and Firewalls

Firewalls can filter ICMP.

Common mistake:

```text
block all ICMP
```

This can interfere with diagnostics and important network functions.

---

## 76. ICMP and PMTUD

Path MTU Discovery can depend on ICMP messages.

If required ICMP messages are blocked:

```text
PMTUD can fail
```

leading to:

```text
large packet failures
TLS hangs
application timeouts
```

---

## 77. IPv4 Fragmentation Needed

IPv4 ICMP can report:

```text
Fragmentation Needed
```

This is relevant to PMTUD.

---

## 78. IPv6 Packet Too Big

IPv6 uses an ICMPv6 message:

```text
Packet Too Big
```

for PMTUD.

Routers do not fragment IPv6 transit packets.

---

## 79. ICMPv6 Is More Important Than ICMPv4

IPv6 relies on ICMPv6 for important protocol functions including:

```text
Neighbor Discovery
Router Discovery
PMTUD
```

Therefore, blindly blocking ICMPv6 can break IPv6 networking.

---

## 80. ICMP Security

ICMP can be abused for:

```text
scanning
flooding
covert channels
reconnaissance
```

but completely blocking it is usually not the correct security strategy.

Use controlled filtering and rate limiting.

---

## 81. Ping Flood

An attacker can send large numbers of ICMP Echo Requests.

Potential effects:

```text
CPU utilization
bandwidth consumption
service degradation
```

Controls include:

```text
rate limiting
DDoS protection
network ACL/firewall controls
```

---

## 82. Smurf Attack

A historical ICMP attack used broadcast amplification.

Modern networks generally implement controls that reduce this risk, but the attack is important historically for understanding ICMP abuse.

---

## 83. ICMP Redirect

ICMP Redirect can inform hosts about a more appropriate next hop.

Modern network security practices often restrict unnecessary use of redirects.

---

## 84. ICMP Source Quench

ICMP Source Quench is obsolete.

Do not build modern systems around it.

---

## 85. ICMP Timestamp

ICMP timestamp messages exist but are rarely needed in modern application environments and may be restricted for security reasons.

---

## 86. ICMP and AWS Security Groups

AWS Security Groups can be configured for ICMP traffic where supported by the relevant rule model.

Example conceptual rule:

```text
Type: All ICMP - IPv4
Source: trusted CIDR
```

Do not expose unnecessary ICMP access globally.

---

## 87. ICMP and AWS NACLs

NACLs are stateless.

If you intentionally allow ICMP, ensure the corresponding return traffic behavior is compatible with the stateless rule set.

---

## 88. ICMP and VPC Flow Logs

VPC Flow Logs can help identify whether traffic flows are accepted or rejected.

Remember that flow logs are flow metadata, not a full ICMP packet capture.

---

## 89. AWS Reachability Analyzer vs Ping

Reachability Analyzer:

```text
models supported network path/configuration
```

Ping:

```text
sends an actual ICMP test
```

They answer different questions.

---

## 90. EC2 ARP/ICMP Troubleshooting

Start:

```bash
ip addr
ip route
ip neigh
```

Then:

```bash
ping <gateway>
ping <destination>
tracepath <destination>
```

Then inspect:

```text
SG
NACL
route table
VPC Flow Logs
Reachability Analyzer
```

---

## 91. EKS ARP Considerations

With EKS, the exact Pod networking behavior depends on the CNI.

For AWS VPC CNI, Pod IPs are integrated with VPC networking through ENIs/IP allocation.

Do not assume a traditional Kubernetes overlay network when diagnosing EKS.

---

## 92. EKS IPv4 Neighbor Behavior

Linux nodes and interfaces may maintain neighbor information.

Inspect:

```bash
ip neigh
```

on nodes when investigating local network issues.

---

## 93. EKS ICMP Testing

From a controlled debug Pod:

```bash
kubectl run net-debug \
  --rm -it \
  --image=nicolaka/netshoot \
  -- /bin/bash
```

Then:

```bash
ping <pod-ip>
ping <service-ip>
ping <external-ip>
```

Interpret results carefully.

---

## 94. Service IP Ping Caveat

A successful or failed ping to a Kubernetes Service IP does not always represent the same behavior as TCP application traffic.

Service dataplane behavior and ICMP handling can differ.

Test the actual application port:

```bash
curl -v http://service:port
```

---

## 95. Pod-to-Pod ICMP

If Pod A cannot ping Pod B:

Check:

```text
Pod IPs
CNI
node routes
NetworkPolicy
security controls
CNI configuration
```

But also test TCP to the actual service port.

---

## 96. Node-to-Pod ICMP

Useful for isolating:

```text
Pod network
CNI
node network
```

but not sufficient to prove application connectivity.

---

## 97. Pod-to-External ICMP

Failure can result from:

```text
external firewall
NAT path
ICMP filtering
security controls
```

Use:

```bash
curl
nc
```

for application-specific validation.

---

## 98. TCP vs ICMP Test

Bad conclusion:

```text
ping fails → application is down
```

Better:

```text
ping fails
→ test TCP/443
→ test HTTP
→ inspect routing/firewall
```

---

## 99. `nc` for TCP

Example:

```bash
nc -vz example.com 443
```

This tests TCP connectivity rather than ICMP.

---

## 100. `curl` for HTTP

```bash
curl -v https://example.com
```

This tests:

```text
DNS
TCP
TLS
HTTP
```

depending on where the failure occurs.

---

## 101. `tcpdump`

Packet capture:

```bash
sudo tcpdump -ni eth0 icmp
```

ARP:

```bash
sudo tcpdump -ni eth0 arp
```

TCP:

```bash
sudo tcpdump -ni eth0 tcp port 443
```

Use filters carefully in production.

---

## 102. ARP Capture

Example:

```bash
sudo tcpdump -ni eth0 arp
```

You may observe:

```text
who-has 10.0.1.1
tell 10.0.1.10
```

followed by a reply.

---

## 103. ICMP Capture

```bash
sudo tcpdump -ni eth0 icmp
```

For IPv6:

```bash
sudo tcpdump -ni eth0 icmp6
```

---

## 104. ARP Request Without Reply

If you observe:

```text
ARP request
ARP request
ARP request
```

but no reply:

```text
neighbor resolution is failing
```

Investigate local connectivity and whether the expected host owns the address.

---

## 105. ICMP Echo Without Reply

If you observe:

```text
Echo Request
```

but no:

```text
Echo Reply
```

possible causes include:

```text
destination down
ICMP filtering
routing issue
security control
rate limiting
```

---

## 106. TCP SYN Without SYN-ACK

If:

```text
SYN
SYN
SYN
```

but no:

```text
SYN-ACK
```

investigate:

```text
routing
firewall
security group
NACL
listener
return path
```

This is often more useful than pinging.

---

## 107. TCP RST

If the destination returns:

```text
RST
```

the host/path may be reachable, but the connection was actively rejected/reset.

Potential causes:

```text
no listener
firewall rejection
application reset
```

---

## 108. ICMP vs TCP Troubleshooting

Use protocol-specific tests:

```text
ICMP
→ basic network reachability

TCP
→ port reachability

TLS
→ encrypted session

HTTP
→ application protocol
```

---

## 109. ARP Troubleshooting Decision Tree

```text
Need to reach local IP?
       |
       v
Route says local?
       |
      Yes
       |
       v
ip neigh
       |
       +-- MAC present → send frame
       |
       +-- INCOMPLETE → ARP resolution
       |
       +-- FAILED → investigate L2/path
```

---

## 110. Remote Destination Decision Tree

```text
Destination remote?
       |
       v
Route lookup
       |
       v
Default/Specific gateway
       |
       v
Resolve gateway MAC
       |
       v
Send Ethernet frame
       |
       v
Router forwards packet
```

---

## 111. ICMP Troubleshooting Decision Tree

```text
Ping fails
   |
   +-- DNS issue? → resolve IP first
   |
   +-- Route issue? → ip route get
   |
   +-- ICMP filtered? → test TCP
   |
   +-- SG/NACL? → inspect AWS
   |
   +-- Host firewall? → inspect host
   |
   +-- Application? → curl/nc
```

---

## 112. MTU Troubleshooting With ICMP

If:

```text
ping small works
large payload fails
```

test:

```bash
tracepath <destination>
```

and controlled packet sizes.

Investigate:

```text
MTU
PMTUD
ICMP filtering
VPN/tunnel overhead
```

---

## 113. VPN and ICMP

VPNs add encapsulation overhead.

If the tunnel's effective MTU is lower than the underlying network:

```text
large packets
→ fragmentation/PMTUD issue
```

may occur.

---

## 114. Container Network Namespace Capture

For advanced troubleshooting, identify the Pod/container network namespace and inspect:

```text
interfaces
routes
neighbor entries
packet capture
```

Use Kubernetes/runtime-supported debugging methods rather than modifying production containers unnecessarily.

---

## 115. Kubernetes Ephemeral Containers

Ephemeral containers can be used for advanced troubleshooting when enabled and appropriately controlled.

Example:

```bash
kubectl debug pod/<pod-name> -it --image=nicolaka/netshoot
```

Exact behavior depends on cluster configuration and permissions.

---

## 116. Security Principle for Debugging

Do not permanently install diagnostic tools into production images merely for troubleshooting.

Prefer:

```text
ephemeral debug container
approved debug Pod
node debugging tools
centralized observability
```

---

## 117. ARP Failure Scenario

Problem:

```text
EC2 cannot reach another local address
```

Commands:

```bash
ip route get <ip>
ip neigh
```

If:

```text
FAILED
```

continue with:

```bash
tcpdump -ni eth0 arp
```

and AWS network analysis.

---

## 118. Gateway ARP Failure Scenario

If:

```text
default via 10.0.1.1
```

but:

```text
ip neigh
```

shows:

```text
10.0.1.1 FAILED
```

investigate:

```text
interface
subnet
route
host configuration
cloud networking
```

Do not immediately change routes.

---

## 119. ICMP Destination Unreachable Scenario

If an application receives:

```text
Destination Unreachable
```

inspect the ICMP type/code where possible.

It can provide clues about:

```text
network
host
protocol
port
fragmentation
```

---

## 120. ICMP Time Exceeded Scenario

Repeated:

```text
Time Exceeded
```

can indicate:

```text
routing loop
TTL too low
path discovery probe
```

Use traceroute/mtr to understand the path.

---

## 121. Routing Loop

Example:

```text
Router A → Router B
Router B → Router A
```

Packets repeatedly circulate until TTL/Hop Limit expires.

ICMP Time Exceeded can expose the problem.

---

## 122. ARP and Routing Loop

ARP itself does not solve a Layer-3 routing loop.

The host resolves the next hop locally; routers make subsequent routing decisions.

---

## 123. Duplicate IP Detection

Symptoms can include:

```text
intermittent connectivity
MAC address changing
ARP cache instability
```

Investigate:

```bash
ip neigh
tcpdump -ni eth0 arp
```

and the network's IP allocation system.

---

## 124. Gratuitous ARP and Duplicate IP

Gratuitous ARP can help hosts announce address ownership, and it can expose competing ownership of the same IP.

---

## 125. ARP Cache Instability

If a single IP repeatedly maps to different MAC addresses:

```text
IP → MAC-A
IP → MAC-B
IP → MAC-A
```

investigate:

```text
duplicate IP
HA/failover
ARP spoofing
network misconfiguration
```

---

## 126. Production ARP Checklist

```text
[ ] Correct local subnet
[ ] Correct route
[ ] Correct gateway
[ ] Neighbor entry
[ ] No duplicate IP
[ ] Interface UP
[ ] Expected MAC ownership
[ ] No L2 security issue
[ ] Cloud networking validated
```

---

## 127. Production ICMP Checklist

```text
[ ] Destination IP correct
[ ] Route correct
[ ] ICMP allowed where required
[ ] SG/NACL checked
[ ] Host firewall checked
[ ] Rate limiting considered
[ ] Return path checked
[ ] TCP test performed
[ ] Application test performed
```

---

## 128. RoboShop ARP/ICMP Context

RoboShop runs on EKS.

A developer reports:

```text
frontend cannot reach cart
```

Do not immediately assume ARP.

Start with:

```text
DNS
Service
Endpoints
Pod IP
NetworkPolicy
TCP
application
```

Then inspect node/VPC networking if needed.

---

## 129. RoboShop Service Test

From a debug Pod:

```bash
curl -v http://cart:8080
```

If DNS resolves but TCP fails:

```text
Service
endpoints
network policy
CNI
routing
```

need investigation.

---

## 130. RoboShop Pod Test

Get Pod IP:

```bash
kubectl get pods -o wide
```

Then test:

```bash
curl -v http://<cart-pod-ip>:8080
```

If direct Pod traffic works but Service traffic fails:

```text
Service/endpoints/dataplane
```

becomes the primary investigation area.

---

## 131. RoboShop External Test

From an approved debug Pod:

```bash
curl -v https://www.example.com
```

If this fails:

```text
DNS
egress route
NAT/VPC endpoint
SG
NACL
```

are candidates.

---

## 132. RoboShop ALB Test

External:

```text
Client
 |
Route 53
 |
ALB
 |
Target
```

Check:

```text
ALB DNS
listener
target health
security groups
subnets/routes
Ingress
Service
Pod
```

---

## 133. Production Failure: Ping Fails but Website Works

Likely:

```text
ICMP filtered
```

Verify:

```bash
curl -vk https://application.example.com
```

If HTTP/TLS works, do not "fix" a non-problem merely to make ping succeed.

---

## 134. Production Failure: Website Times Out

Run:

```bash
curl -v https://application.example.com
```

Determine whether failure occurs during:

```text
DNS
TCP
TLS
HTTP
```

Then inspect the corresponding layer.

---

## 135. Production Failure: HTTPS Hangs Only With Large Responses

Consider:

```text
MTU
PMTUD
MSS
ICMP filtering
VPN
firewall
```

Use:

```bash
tracepath
tcpdump
```

and controlled tests.

---

## 136. Production Failure: One AZ Has Connectivity Issues

Compare:

```text
AZ-A
AZ-B
AZ-C
```

Check:

```text
subnet route table
NAT
NACL
security group
node/CNI
```

A zonal comparison is often powerful.

---

## 137. Production Failure: New Pod Has No Egress

Check:

```text
Pod IP
node
node subnet
route table
NAT/VPC endpoint
CNI
SG
NACL
DNS
```

Compare with a working Pod.

---

## 138. Production Failure: New Node Has Network Issues

Check:

```text
node subnet
route table
ENI/IP availability
security groups
CNI daemon
DNS
```

Then compare with a healthy node in another AZ.

---

## 139. Production Failure: DNS Works but Service Fails

Separate:

```text
DNS
```

from:

```text
Service routing
```

Check:

```bash
kubectl get svc
kubectl get endpointslice
```

then test the Service port.

---

## 140. Production Failure: Service Works by IP but Not by Name

Likely focus:

```text
CoreDNS
DNS policy
search domains
NetworkPolicy
```

Check:

```bash
nslookup service.namespace.svc.cluster.local
```

from the affected Pod.

---

## 141. Production Failure: Service Name Resolves but Connection Refused

This indicates DNS is working.

Focus on:

```text
Service port
targetPort
Endpoints
Pod listener
application
```

Use:

```bash
nc -vz <service> <port>
curl -v
```

---

## 142. Production Failure: Connection Reset

Investigate:

```text
listener
application
proxy
load balancer
firewall
```

Use packet capture where appropriate.

---

## 143. Production Failure: Connection Hangs

Potential causes:

```text
packet drop
routing
firewall
NACL
MTU
application stall
```

Compare:

```text
SYN
SYN-ACK
ACK
TLS
HTTP
```

with packet capture.

---

## 144. Packet Capture Golden Rule

Capture both sides where possible.

Example:

```text
source capture:
SYN sent

destination capture:
SYN never received
```

This strongly narrows the problem to the path between them.

---

## 145. ICMP and TCP Layered Test

Use:

```bash
ping <ip>
nc -vz <ip> <port>
curl -v http://<ip>:<port>
```

Interpret:

```text
ping = ICMP
nc = TCP
curl = TCP + application protocol
```

---

## 146. ARP and TCP Layered Test

For a local destination:

```text
ARP resolves
→ Ethernet works

TCP SYN succeeds
→ transport path works

HTTP succeeds
→ application protocol works
```

---

## 147. Don't Troubleshoot From Only One Host

For production incidents, collect evidence from:

```text
source
destination
intermediate network
AWS control plane
Kubernetes control plane
application
```

This avoids false assumptions.

---

## 148. ARP vs NDP Summary

```text
IPv4
→ ARP

IPv6
→ Neighbor Discovery / ICMPv6
```

---

## 149. ICMPv4 vs ICMPv6 Summary

ICMPv4 supports:

```text
Echo
errors
diagnostics
PMTUD-related messages
```

ICMPv6 additionally plays a major role in:

```text
Neighbor Discovery
Router Discovery
IPv6 configuration
PMTUD
```

---

## 150. Command Reference

### Linux

```bash
ip addr
ip route
ip route get <destination>
ip rule
ip neigh
ip -6 neigh
ping <destination>
tracepath <destination>
traceroute <destination>
mtr -rw <destination>
ss -ntp
tcpdump -ni eth0 arp
tcpdump -ni eth0 icmp
tcpdump -ni eth0 icmp6
```

---

## 151. AWS CLI Reference

```bash
aws ec2 describe-route-tables
aws ec2 describe-network-interfaces
aws ec2 describe-security-groups
aws ec2 describe-network-acls
aws ec2 describe-subnets
aws ec2 describe-vpcs
```

Use filters and resource IDs to narrow results.

---

## 152. Kubernetes Reference

```bash
kubectl get pods -o wide
kubectl get svc
kubectl get endpoints
kubectl get endpointslice
kubectl get networkpolicy -A
kubectl get nodes -o wide
kubectl get pods -n kube-system
kubectl logs -n kube-system <cni-pod>
```

---

## 153. Interview: What Does ARP Do?

ARP resolves an IPv4 address to a MAC address on the local link.

---

## 154. Interview: Does ARP Resolve Remote Server MAC?

No.

For a remote destination, the host normally resolves the MAC of the local next-hop gateway.

---

## 155. Interview: Why Is ARP Broadcast Local?

ARP is designed for local-link address resolution and routers do not normally forward the broadcast across routed networks.

---

## 156. Interview: What Is an ARP Cache?

A temporary mapping of IPv4 neighbor addresses to Layer-2 addresses.

---

## 157. Interview: What Is Gratuitous ARP?

An unsolicited ARP announcement used for purposes such as updating neighbor mappings, failover, or duplicate address detection.

---

## 158. Interview: What Is ARP Spoofing?

Forged ARP information that causes hosts to associate an IP address with an attacker's MAC address.

---

## 159. Interview: How Do You Check ARP on Linux?

```bash
ip neigh
```

---

## 160. Interview: How Do You Capture ARP?

```bash
tcpdump -ni eth0 arp
```

---

## 161. Interview: What Replaced ARP in IPv6?

IPv6 uses Neighbor Discovery Protocol through ICMPv6.

---

## 162. Interview: What Does ICMP Do?

ICMP provides network control, diagnostics, and error reporting.

---

## 163. Interview: What Does Ping Test?

It tests ICMP Echo Request/Reply reachability, not general application health.

---

## 164. Interview: Does Ping Prove HTTPS Works?

No.

HTTPS requires:

```text
DNS
TCP
TLS
HTTP
```

and may work even if ICMP is blocked.

---

## 165. Interview: Why Does Traceroute Work?

It manipulates TTL/Hop Limit so intermediate routers can return Time Exceeded messages.

---

## 166. Interview: What Is ICMP Time Exceeded?

An ICMP error generated when the packet's TTL/Hop Limit expires in transit.

---

## 167. Interview: What Is ICMP Destination Unreachable?

An ICMP error indicating that a destination or required delivery condition cannot be reached.

---

## 168. Interview: Why Should ICMP Not Be Blocked Everywhere?

Some ICMP messages are required for diagnostics and important network behavior such as PMTUD, and ICMPv6 is essential to IPv6 operation.

---

## 169. Interview: How Do You Troubleshoot Ping Failure?

Check:

```text
IP
route
ICMP filtering
SG
NACL
host firewall
return path
```

Then test TCP/application connectivity.

---

## 170. Interview: How Do You Troubleshoot ARP Failure?

Use:

```bash
ip neigh
tcpdump -ni eth0 arp
```

and verify:

```text
local subnet
interface
gateway
address ownership
```

---

## 171. Interview: What Is PMTUD?

Path MTU Discovery determines the largest packet size that can traverse a path without problematic fragmentation.

---

## 172. Interview: Why Can Blocking ICMP Break Applications?

PMTUD can depend on ICMP error messages. Blocking them can create black-hole behavior for packets that exceed the path MTU.

---

## 173. Interview: What Is an ICMP Black Hole?

A condition where required ICMP feedback is blocked, causing endpoints to fail to learn that packets need smaller sizes or cannot be delivered.

---

## 174. Interview: How Do You Debug an ICMP Black Hole?

Check:

```text
MTU
PMTUD
firewall
VPN
ICMP filtering
```

Use:

```bash
tracepath
tcpdump
```

and controlled packet-size tests.

---

## 175. Interview: What Is the Difference Between ARP and DNS?

```text
ARP:
IP → MAC on local link

DNS:
name → IP/name information
```

They solve completely different problems.

---

## 176. Interview: What Is the Difference Between ARP and Routing?

```text
Routing:
selects next hop

ARP:
resolves local next-hop IP to MAC
```

---

## 177. Interview: What Is the Difference Between ICMP and TCP?

```text
ICMP:
network control/diagnostics

TCP:
reliable connection-oriented transport
```

---

## 178. Interview: How Do You Troubleshoot a Kubernetes Service?

Use:

```bash
kubectl get svc
kubectl get endpointslice
kubectl get pods -o wide
```

Then test:

```bash
curl
nc
```

from a suitable source Pod.

---

## 179. Interview: How Do You Troubleshoot EKS Pod Egress?

Check:

```text
Pod IP
CNI
node
subnet
route table
NAT/VPC endpoint
SG
NACL
DNS
```

---

## 180. Interview: How Do You Prove a Packet Is Leaving a Host?

Use:

```bash
tcpdump
```

on the relevant interface.

---

## 181. Interview: How Do You Prove a Packet Reaches a Destination?

Capture at the destination if possible.

Source-only capture proves transmission from the source, not successful delivery.

---

## 182. Interview: What Does `ip neigh` Tell You?

It shows the Linux neighbor table, including IPv4 ARP and IPv6 neighbor discovery information.

---

## 183. Interview: What Does `ip route get` Tell You?

It shows the route Linux would select for a specific destination, including relevant interface/source/next-hop information.

---

## 184. Interview: What Does MTR Provide?

Repeated path measurements showing hop information, latency, and apparent loss.

Interpret intermediate-hop loss carefully because of ICMP rate limiting.

---

## 185. Interview: What Is Gratuitous ARP Used For?

Common uses include:

```text
failover
neighbor cache updates
duplicate IP detection
```

---

## 186. Interview: What Is Proxy ARP?

A router can answer an ARP request on behalf of another host/network, making the requester believe the target is locally reachable.

Use is architecture-dependent.

---

## 187. Interview: What Is NDP?

IPv6 Neighbor Discovery is an ICMPv6-based protocol suite for neighbor resolution, router discovery, prefix discovery, and reachability functions.

---

## 188. Interview: What Is Router Advertisement?

An ICMPv6 message from a router that can provide hosts with information about available routers, prefixes, and IPv6 configuration.

---

## 189. Interview: What Is Neighbor Solicitation?

An ICMPv6 message used for neighbor resolution and related neighbor discovery operations.

---

## 190. Interview: What Is Neighbor Advertisement?

An ICMPv6 response/announcement conveying neighbor information.

---

## 191. Interview: What Is Duplicate Address Detection?

A mechanism used by IPv6 nodes to check whether an address is already in use before normal use.

---

## 192. Interview: What Is the Main Difference Between IPv4 and IPv6 Neighbor Resolution?

```text
IPv4:
ARP

IPv6:
ICMPv6 Neighbor Discovery
```

---

## 193. Production Troubleshooting Framework

Use this order:

```text
1. Identify exact source.
2. Identify exact destination.
3. Determine protocol.
4. Check DNS if names are involved.
5. Check route.
6. Check neighbor resolution where applicable.
7. Check security controls.
8. Check transport.
9. Check TLS.
10. Check application.
```

---

## 194. Golden Rule

Never conclude:

```text
"Network is broken"
```

from one failed ping.

Collect evidence at the protocol layer actually used by the application.

---

## 195. Production Network Evidence

Useful evidence includes:

```text
ip route get
ip neigh
tcpdump
curl -v
nc -vz
mtr
VPC Flow Logs
Reachability Analyzer
AWS route tables
Kubernetes Service/EndpointSlice
CNI logs
application logs
```

---

## 196. Final ARP Mental Model

```text
Need local IPv4 destination
        |
        v
ARP cache?
   |          |
  yes         no
   |          |
   v          v
Use MAC     ARP Request
                |
                v
            ARP Reply
                |
                v
             Cache
                |
                v
             Send
```

---

## 197. Final Remote Traffic Mental Model

```text
Application
    |
Destination IP
    |
Route lookup
    |
Next-hop gateway
    |
ARP/neighbor resolution
    |
Ethernet/VPC forwarding
    |
Router
    |
Next network
```

---

## 198. Final ICMP Mental Model

```text
Application/network event
        |
        v
ICMP control/error
        |
        +--> Echo
        +--> Destination Unreachable
        +--> Time Exceeded
        +--> PMTUD feedback
        +--> IPv6 Neighbor Discovery
```

---

## 199. Final DevOps Mental Model

```text
DNS
 |
IP
 |
Route
 |
Neighbor/ARP/NDP
 |
Security
 |
TCP/UDP
 |
TLS
 |
HTTP
 |
Application
```

When troubleshooting production connectivity, move layer by layer instead of guessing.

---