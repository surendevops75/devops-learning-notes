# 16-Networking-for-DevOps

# 03-MAC-IP-and-Network-Interfaces.md

# MAC Addresses, IP Addresses and Network Interfaces

## Purpose

Network interfaces are the point where operating systems, containers,
virtual machines and cloud workloads connect to networks.

A DevOps engineer must understand:

``` text
NIC
 ↓
MAC
 ↓
IP
 ↓
Subnet
 ↓
Route
 ↓
Gateway
 ↓
TCP/UDP
 ↓
Application
```

This knowledge is required for troubleshooting:

-   Linux servers
-   EC2
-   Docker
-   Kubernetes
-   EKS
-   AWS ENIs
-   Pod networking
-   Service networking
-   connectivity failures
-   routing failures
-   interface failures
-   container networking

------------------------------------------------------------------------

# 1. What Is a Network Interface?

A network interface is a software/hardware interface through which a
system communicates over a network.

Examples:

``` text
eth0
ens5
enp0s3
lo
docker0
vethXXXX
```

A physical server may have physical NICs.

A cloud VM normally sees a virtual network interface.

A container normally has a virtual network interface.

------------------------------------------------------------------------

# 2. Physical vs Virtual Interfaces

## Physical

Examples:

``` text
Ethernet NIC
Wi-Fi adapter
Fiber NIC
```

## Virtual

Examples:

``` text
loopback
bridge
veth
VLAN interface
bond
cloud ENI
container interface
```

Modern DevOps environments use virtual interfaces heavily.

------------------------------------------------------------------------

# 3. Linux Network Interfaces

List interfaces:

``` bash
ip link
```

Show addresses:

``` bash
ip addr
```

Short form:

``` bash
ip -br addr
```

Example:

``` text
lo      UNKNOWN  127.0.0.1/8
eth0    UP       10.0.1.20/24
```

------------------------------------------------------------------------

# 4. Interface States

Common states:

``` text
UP
DOWN
UNKNOWN
LOWER_UP
```

Check:

``` bash
ip link show eth0
```

An interface can be administratively enabled while the underlying link
state differs.

For example:

``` text
state UP
LOWER_UP
```

generally indicates the interface is enabled and the lower link is
available.

------------------------------------------------------------------------

# 5. Loopback Interface

Linux has a loopback interface:

``` text
lo
```

Typical address:

``` text
127.0.0.1/8
```

It is used for local-host communication.

Example:

``` bash
curl http://127.0.0.1:8080
```

The traffic remains local to the host.

------------------------------------------------------------------------

# 6. MAC Address

A MAC address identifies a network interface at the link layer.

Example:

``` text
02:42:ac:11:00:02
```

View:

``` bash
ip link show
```

or:

``` bash
cat /sys/class/net/eth0/address
```

------------------------------------------------------------------------

# 7. MAC Address Format

Common notation:

``` text
00:1A:2B:3C:4D:5E
```

Six hexadecimal octets:

``` text
00
1A
2B
3C
4D
5E
```

MAC addressing is associated primarily with Layer 2.

------------------------------------------------------------------------

# 8. MAC Is Not an IP Address

MAC:

``` text
Layer 2
Local/link delivery
```

IP:

``` text
Layer 3
Logical addressing/routing
```

Example:

``` text
MAC
02:42:ac:11:00:02

IP
10.0.1.20
```

Do not treat them as interchangeable.

------------------------------------------------------------------------

# 9. IP Address on an Interface

An interface can have one or more IP addresses.

Example:

``` text
eth0
  |
  +-- 10.0.1.20/24
```

Show:

``` bash
ip addr show eth0
```

An interface can also have IPv6 addresses.

------------------------------------------------------------------------

# 10. Multiple IP Addresses

Linux supports multiple addresses on one interface.

Example:

``` text
eth0
 |
 +-- 10.0.1.20/24
 +-- 10.0.1.21/24
```

This is useful for some advanced networking architectures.

Do not assume:

``` text
one interface = one IP
```

------------------------------------------------------------------------

# 11. IPv4 Address and CIDR

Example:

``` text
10.0.1.20/24
```

Here:

``` text
IP      = 10.0.1.20
Prefix  = /24
```

The `/24` determines the network boundary.

Network:

``` text
10.0.1.0/24
```

------------------------------------------------------------------------

# 12. IPv6 on an Interface

Example:

``` text
2001:db8::20/64
```

Show:

``` bash
ip -6 addr
```

IPv6 is increasingly important in cloud and enterprise networking.

------------------------------------------------------------------------

# 13. Interface Name Conventions

Common Linux names:

``` text
eth0
ens5
enp0s3
eno1
lo
docker0
vethabc123
```

Modern Linux distributions use predictable interface naming.

Do not hard-code `eth0` in scripts unless the environment guarantees it.

------------------------------------------------------------------------

# 14. Interface Configuration

Temporarily add an address:

``` bash
sudo ip addr add 10.0.1.50/24 dev eth0
```

Bring interface up:

``` bash
sudo ip link set eth0 up
```

Remove address:

``` bash
sudo ip addr del 10.0.1.50/24 dev eth0
```

These commands modify runtime state.

Persistent configuration depends on the Linux distribution and network
manager.

------------------------------------------------------------------------

# 15. NetworkManager

Many Linux systems use NetworkManager.

Check:

``` bash
nmcli device status
```

Show connections:

``` bash
nmcli connection show
```

NetworkManager configuration differs from direct `ip` commands.

Important distinction:

``` text
ip command
    → runtime kernel networking state

NetworkManager
    → persistent/network connection management
```

------------------------------------------------------------------------

# 16. Routing and Interfaces

An interface is normally associated with routes.

Example:

``` bash
ip route
```

Output:

``` text
10.0.1.0/24 dev eth0
default via 10.0.1.1 dev eth0
```

This means:

``` text
10.0.1.0/24
    → directly reachable through eth0

other destinations
    → default gateway 10.0.1.1
```

------------------------------------------------------------------------

# 17. Source IP Selection

When a host has multiple addresses, Linux must choose an appropriate
source address for outgoing traffic.

Example:

``` text
eth0
  10.0.1.20
  10.0.1.21
```

Check route selection:

``` bash
ip route get 8.8.8.8
```

The output can show the selected:

``` text
interface
source IP
gateway
```

This is useful when diagnosing unexpected source addresses.

------------------------------------------------------------------------

# 18. Gateway

A gateway is the next hop used to reach another network.

Example:

``` text
Host
10.0.1.20
   |
eth0
   |
10.0.1.1
Gateway
```

Check:

``` bash
ip route
```

------------------------------------------------------------------------

# 19. ARP

ARP maps IPv4 addresses to MAC addresses on a local network.

Conceptually:

``` text
Who has 10.0.1.1?
```

The device owning that IP responds with its MAC address.

Linux:

``` bash
ip neigh
```

------------------------------------------------------------------------

# 20. Neighbor Table

Example:

``` bash
ip neigh
```

Possible output:

``` text
10.0.1.1 dev eth0 lladdr 02:aa:bb:cc:dd:ee REACHABLE
```

This tells you:

``` text
IP
interface
MAC
state
```

------------------------------------------------------------------------

# 21. Neighbor States

Common states include:

``` text
REACHABLE
STALE
DELAY
PROBE
FAILED
INCOMPLETE
```

These states help diagnose local neighbor-resolution behavior.

------------------------------------------------------------------------

# 22. ARP Troubleshooting

If a host cannot reach a same-subnet destination, inspect:

``` bash
ip neigh
```

Then:

``` bash
ip route
```

Then:

``` bash
tcpdump -ni eth0 arp
```

Possible symptoms:

``` text
INCOMPLETE
FAILED
```

may indicate neighbor resolution problems.

------------------------------------------------------------------------

# 23. IPv6 Neighbor Discovery

IPv6 does not use ARP.

It uses Neighbor Discovery Protocol through ICMPv6.

Useful concepts include:

``` text
Neighbor Solicitation
Neighbor Advertisement
Router Solicitation
Router Advertisement
```

This is an important IPv4 vs IPv6 difference.

------------------------------------------------------------------------

# 24. Ethernet

Ethernet is a common Layer 2 technology.

It carries frames.

Conceptually:

``` text
Source MAC
Destination MAC
Payload
```

Linux interfaces commonly represent Ethernet connectivity.

------------------------------------------------------------------------

# 25. Broadcast Domain

A broadcast domain is the set of devices that receive Layer 2 broadcast
traffic.

VLANs and routing boundaries can separate broadcast domains.

Cloud networking abstracts some physical Layer 2 details, but the
concept remains useful for understanding segmentation.

------------------------------------------------------------------------

# 26. Virtual Ethernet Pair

A veth pair creates two connected virtual interfaces.

Conceptually:

``` text
veth-A <==========> veth-B
```

Traffic entering one end appears at the other.

This is heavily used in Linux containers.

------------------------------------------------------------------------

# 27. Container veth Architecture

Simplified:

``` text
Container
   |
 veth
   |
Host
   |
Bridge
   |
Host NIC
   |
Network
```

More accurately, one end of a veth pair is placed inside a network
namespace while the other remains in another namespace.

------------------------------------------------------------------------

# 28. Network Namespace

A Linux network namespace provides an isolated networking environment.

It can have its own:

``` text
interfaces
IP addresses
routing table
iptables/nftables state
sockets
```

Containers commonly use network namespaces.

------------------------------------------------------------------------

# 29. Container Network Namespace

Simplified:

``` text
Host namespace
      |
      | veth
      |
Container namespace
      |
      +-- eth0
      +-- route
      +-- IP
```

This is one of the foundations of container networking.

------------------------------------------------------------------------

# 30. Docker Bridge

Typical Docker networking:

``` text
Container
   |
veth
   |
docker0 bridge
   |
Host interface
   |
Network
```

Inspect:

``` bash
ip link
ip addr
docker network ls
docker network inspect bridge
```

------------------------------------------------------------------------

# 31. Linux Bridge

A Linux bridge acts like a virtual Layer 2 switch.

Example:

``` text
veth1 ---+
         |
veth2 ---+--- bridge --- host network
         |
veth3 ---+
```

Inspect bridges:

``` bash
bridge link
bridge fdb
```

or:

``` bash
ip link
```

------------------------------------------------------------------------

# 32. Bridge Forwarding Database

A bridge maintains MAC forwarding information.

Command:

``` bash
bridge fdb
```

The forwarding database helps determine where frames should be sent.

This mirrors the behavior of physical Ethernet switches conceptually.

------------------------------------------------------------------------

# 33. Network Namespace Commands

List namespaces:

``` bash
ip netns list
```

Execute command inside a namespace:

``` bash
ip netns exec <namespace> ip addr
```

For container environments, namespaces may be managed by Docker,
containerd, Kubernetes or CNI tooling rather than manually through
`ip netns`.

------------------------------------------------------------------------

# 34. Namespace Troubleshooting

When troubleshooting containers, ask:

``` text
Which namespace is the process in?
Which interface exists there?
Which IP does it have?
Which routes exist?
Which DNS resolver is configured?
```

A common mistake is checking the host network and assuming the container
has identical networking.

------------------------------------------------------------------------

# 35. Network Interface Counters

Interface statistics can expose problems.

Command:

``` bash
ip -s link
```

Look for:

``` text
RX errors
TX errors
dropped packets
overruns
collisions
```

Cloud environments may expose additional metrics through provider
tooling.

------------------------------------------------------------------------

# 36. RX vs TX

RX means:

``` text
Receive
```

TX means:

``` text
Transmit
```

Conceptually:

``` text
Host
 ↓
TX → network
network → RX
```

High errors or drops require investigation.

------------------------------------------------------------------------

# 37. Interface Drops

Packets can be dropped for many reasons:

``` text
buffer pressure
queue limits
firewall rules
network policy
congestion
invalid packets
resource exhaustion
```

Do not automatically assume interface drops mean a physical NIC failure.

------------------------------------------------------------------------

# 38. MTU

MTU is the maximum transmission unit supported for packets on an
interface/path without requiring fragmentation at that point.

Common Ethernet MTU:

``` text
1500
```

Check:

``` bash
ip link show eth0
```

------------------------------------------------------------------------

# 39. MTU in Containers

Container networking can introduce additional overhead.

Examples:

``` text
encapsulation
overlay networking
tunneling
VPN
service mesh
```

The effective path MTU may be lower than the host interface MTU.

------------------------------------------------------------------------

# 40. MTU Troubleshooting

Symptoms:

``` text
small packets work
large packets fail
TLS hangs
some applications work
others fail
```

Useful testing:

``` bash
ping -M do -s <size> <destination>
```

The exact size must account for IP/ICMP headers.

------------------------------------------------------------------------

# 41. Promiscuous Mode

Promiscuous mode allows an interface to receive frames that are not
normally addressed to its own MAC at the interface level.

Check:

``` bash
ip link show
```

Packet capture tools can interact with promiscuous mode depending on
configuration.

Use it carefully in production because packet visibility has security
implications.

------------------------------------------------------------------------

# 42. Interface Offloading

Modern NICs may offload work from the CPU.

Examples:

``` text
TSO
GSO
GRO
checksum offload
```

These can make packet captures confusing because the packet
representation observed on the host may not exactly match what
physically travels on the wire.

This matters during deep packet troubleshooting.

------------------------------------------------------------------------

# 43. TSO

TCP Segmentation Offload allows the NIC/kernel stack to handle
segmentation efficiently.

This improves performance but can affect how traffic appears in packet
captures.

------------------------------------------------------------------------

# 44. GRO

Generic Receive Offload combines packets received from the network into
larger units for more efficient processing.

Again:

``` text
capture representation
≠
exact wire representation
```

in some environments.

------------------------------------------------------------------------

# 45. GSO

Generic Segmentation Offload allows the networking stack to work with
larger packets and defer segmentation.

These optimizations are performance features rather than
application-level networking protocols.

------------------------------------------------------------------------

# 46. Cloud Network Interface

Cloud providers expose virtual network interfaces to compute workloads.

AWS uses:

``` text
Elastic Network Interface
ENI
```

An ENI can be associated with:

``` text
private IPs
security groups
MAC address
subnet
```

------------------------------------------------------------------------

# 47. AWS ENI

Conceptually:

``` text
EC2/EKS Node
      |
      v
     ENI
      |
      +-- Private IP
      +-- MAC
      +-- Security Groups
      +-- Subnet
```

AWS networking relies heavily on ENIs.

------------------------------------------------------------------------

# 48. Primary and Secondary Private IPs

An ENI can have multiple private IP addresses depending on AWS resource
and configuration.

This is important for:

``` text
EC2
EKS
AWS VPC CNI
```

Do not assume one ENI always equals one IP.

------------------------------------------------------------------------

# 49. EKS and AWS VPC CNI

AWS VPC CNI integrates Kubernetes networking with AWS VPC networking.

Conceptually:

``` text
Pod
 |
VPC-native networking
 |
ENI/IP allocation
 |
AWS VPC
```

This differs from a simple Linux bridge-only container network.

------------------------------------------------------------------------

# 50. Pod IP in AWS VPC CNI

With VPC-native Pod networking, Pod addresses are allocated from
VPC/subnet-related address pools according to the CNI configuration.

This can make Pods directly routable within the VPC networking model.

The exact allocation behavior depends on EKS/CNI configuration.

------------------------------------------------------------------------

# 51. EKS Node Interfaces

A node may have:

``` text
primary ENI
secondary ENIs
multiple secondary IP addresses
```

The number of usable addresses depends on:

``` text
instance type
ENI limits
IPv4/IPv6 mode
CNI configuration
subnet capacity
```

This directly affects Pod density.

------------------------------------------------------------------------

# 52. EKS IP Exhaustion

A production EKS cluster can experience IP exhaustion.

Symptoms may include:

``` text
Pod stuck Pending
CNI allocation errors
new Pod cannot receive IP
```

Check:

``` bash
kubectl get pods -A
kubectl describe pod <pod> -n <namespace>
kubectl get nodes
```

Also inspect AWS subnet/IP capacity and CNI logs.

------------------------------------------------------------------------

# 53. EKS Interface Debugging

Useful Kubernetes checks:

``` bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl describe node <node>
```

On the node, if access is available:

``` bash
ip addr
ip route
ip link
```

Also inspect CNI components in the relevant `kube-system` resources.

------------------------------------------------------------------------

# 54. Kubernetes Pod Network Namespace

A Pod has a network namespace shared by its containers.

Important consequence:

``` text
Containers in the same Pod
share the same network namespace
```

Therefore they can communicate through:

``` text
localhost
```

Example:

``` text
Container A → localhost:8080 → Container B
```

provided Container B listens on that port.

------------------------------------------------------------------------

# 55. Pod vs Container Networking

A Pod is not simply one isolated network interface per container.

Typical model:

``` text
Pod network namespace
       |
       +-- eth0
       |
       +-- Container A
       +-- Container B
```

Containers in the Pod share:

``` text
IP
network namespace
ports
localhost
```

This is critical for Kubernetes troubleshooting.

------------------------------------------------------------------------

# 56. Sidecar Networking

Example:

``` text
Pod
 |
 +-- Application Container
 |
 +-- Sidecar
```

Both containers share the same network namespace.

They can communicate using:

``` text
localhost
```

This pattern is used by some service-mesh and proxy architectures.

------------------------------------------------------------------------

# 57. Host Network Mode

A workload can use the host's network namespace in certain
container/Kubernetes configurations.

Conceptually:

``` text
Container
    |
    v
Host network namespace
```

This changes isolation and port behavior.

It should be used deliberately because it reduces network namespace
isolation.

------------------------------------------------------------------------

# 58. Network Interface Naming in Kubernetes

Inside a Pod, the primary interface is commonly:

``` text
eth0
```

However, the host may have many interfaces:

``` text
eth0
vethXXXX
cni-related interfaces
```

Never assume the Pod's `eth0` is the host's physical interface.

------------------------------------------------------------------------

# 59. Find Pod IP

``` bash
kubectl get pods -o wide -n <namespace>
```

Example:

``` text
NAME       IP           NODE
cart       10.0.2.15    node-1
```

This is useful when troubleshooting:

``` text
Pod-to-Pod
Pod-to-Service
Pod-to-Database
```

------------------------------------------------------------------------

# 60. Pod IP vs Service IP

A Pod IP:

``` text
individual workload endpoint
```

A Service IP:

``` text
stable virtual service endpoint
```

Example:

``` text
Service
10.100.10.20
    |
    +--> Pod 10.0.2.15
    +--> Pod 10.0.3.22
```

The Service provides stable abstraction over changing Pod addresses.

------------------------------------------------------------------------

# 61. Why Pod IPs Change

Pods are ephemeral.

A Pod can be recreated due to:

``` text
deployment
node failure
scaling
rollout
eviction
manual deletion
```

Therefore applications should generally not hard-code Pod IPs.

Use:

``` text
Service
DNS
```

for stable discovery.

------------------------------------------------------------------------

# 62. Interface and Routing Relationship

A routing table may reference an interface:

``` text
10.0.2.0/24 dev eth0
```

This means traffic for that destination is sent through `eth0`.

Therefore when diagnosing routing, inspect both:

``` bash
ip route
ip addr
```

------------------------------------------------------------------------

# 63. Route Without Working Interface

A route can exist while the underlying interface/path is broken.

Example:

``` text
default via 10.0.1.1 dev eth0
```

but:

``` text
eth0 DOWN
```

Traffic cannot work.

Always correlate:

``` text
route
+
interface state
```

------------------------------------------------------------------------

# 64. Duplicate IP Address

Two systems using the same IP can cause serious connectivity problems.

Symptoms:

``` text
intermittent connectivity
ARP instability
traffic reaching wrong host
connection resets
```

Possible diagnostic tools include:

``` bash
ip neigh
arping
tcpdump
```

Use carefully in production.

------------------------------------------------------------------------

# 65. ARP Flux

ARP behavior can become complex on multi-interface Linux systems.

A host with multiple interfaces/IPs may respond to ARP in ways that
surprise operators if routing and ARP-related kernel settings are not
designed appropriately.

This is an advanced topic, but the key lesson is:

``` text
multiple interfaces
+
multiple addresses
=
careful routing and neighbor behavior
```

------------------------------------------------------------------------

# 66. Interface Aliases and Secondary Addresses

Modern Linux uses the same interface with multiple addresses rather than
relying on older alias syntax.

Example:

``` bash
ip addr add 10.0.1.21/24 dev eth0
```

Then:

``` bash
ip addr show eth0
```

can show multiple addresses.

------------------------------------------------------------------------

# 67. VLAN Interfaces

Linux can create VLAN interfaces.

Conceptually:

``` text
eth0
 |
 +-- eth0.10
 +-- eth0.20
```

Each VLAN can represent a logical Layer 2 segment.

This is more common in traditional enterprise environments than in
typical AWS VPC architecture, but it is important networking knowledge.

------------------------------------------------------------------------

# 68. Bonding

Linux bonding combines multiple interfaces for redundancy or
performance.

Conceptually:

``` text
eth0 ----+
         |
         +--- bond0
         |
eth1 ----+
```

Modes include different strategies for:

``` text
active/backup
load distribution
link aggregation
```

Cloud networking often provides high availability through provider
abstractions rather than manually configured bonding.

------------------------------------------------------------------------

# 69. Bridge vs Router

Bridge:

``` text
Layer 2
MAC
frames
local network
```

Router:

``` text
Layer 3
IP
packets
different networks
```

Example:

``` text
Containers
   |
Bridge
   |
Host
   |
Router
   |
External network
```

------------------------------------------------------------------------

# 70. Interface Troubleshooting Checklist

When an interface appears broken:

``` text
1. Does interface exist?
2. Is it UP?
3. Does it have an IP?
4. Is subnet correct?
5. Is route present?
6. Is gateway reachable?
7. Is DNS configured?
8. Are packets being transmitted?
9. Are packets being received?
10. Are errors/drops increasing?
```

Commands:

``` bash
ip link
ip addr
ip route
ip -s link
ip neigh
```

------------------------------------------------------------------------

# 71. Network Interface Production Incident

### Symptom

A Linux service suddenly cannot reach another internal service.

Check:

``` bash
ip link
ip addr
ip route
ip neigh
ss -nt
```

Then:

``` bash
nc -vz <destination> <port>
```

If necessary:

``` bash
tcpdump
```

Do not immediately restart the host.

------------------------------------------------------------------------

# 72. EKS Production Incident

### Symptom

New Pods cannot start.

First checks:

``` bash
kubectl get pods -A
kubectl describe pod <pod> -n <namespace>
kubectl get nodes -o wide
```

If events indicate IP allocation/CNI problems, investigate:

``` text
subnet free IPs
ENI capacity
instance type limits
AWS VPC CNI configuration
CNI logs
```

This is a common cloud-networking class of failure.

------------------------------------------------------------------------

# 73. Docker Production Incident

### Symptom

Container cannot reach another container.

Check:

``` bash
docker ps
docker network ls
docker network inspect <network>
```

Then inspect host interfaces:

``` bash
ip link
ip addr
ip route
```

Confirm:

``` text
same network
container IP
service port
routing
firewall
application listener
```

------------------------------------------------------------------------

# 74. Interface Security

Interfaces can be exposed to risks such as:

``` text
packet capture
spoofing
misconfiguration
unnecessary exposure
privilege escalation
```

Production principles:

``` text
least privilege
minimal exposed services
segmentation
host firewall
NetworkPolicy where applicable
TLS
monitoring
```

------------------------------------------------------------------------

# 75. MAC Spoofing Concept

MAC spoofing means changing the apparent source MAC address of a network
interface.

This can be used legitimately for testing or failover but can also be
abused.

Cloud platforms generally impose additional controls around virtual
networking.

DevOps engineers should understand the concept rather than attempting
unauthorized network manipulation.

------------------------------------------------------------------------

# 76. IP Spoofing Concept

IP spoofing involves creating packets with a forged source IP.

Security controls can reduce exposure to spoofed traffic.

This reinforces an important principle:

``` text
source IP alone is not identity
```

Modern cloud security should use:

``` text
IAM
authentication
authorization
security groups
network controls
TLS
```

rather than trusting an IP address as identity.

------------------------------------------------------------------------

# 77. Interface Monitoring

Monitor:

``` text
bytes received
bytes transmitted
packets received
packets transmitted
errors
drops
queue behavior
```

Linux:

``` bash
ip -s link
```

Monitoring systems can collect additional host and cloud network
metrics.

------------------------------------------------------------------------

# 78. Network Namespace Troubleshooting Workflow

For a container or Pod:

``` text
1. Enter workload namespace.
2. Check interface.
3. Check IP.
4. Check route.
5. Check DNS.
6. Test destination.
7. Check security policy.
8. Capture packets if necessary.
```

Kubernetes example:

``` bash
kubectl exec -it <pod> -- sh
```

Then:

``` bash
ip addr
ip route
cat /etc/resolv.conf
```

------------------------------------------------------------------------

# 79. `ip addr` vs `ifconfig`

Modern Linux troubleshooting should prefer:

``` bash
ip addr
ip link
ip route
```

The older:

``` bash
ifconfig
```

may still exist on some systems, but `iproute2` tools are the modern
standard.

------------------------------------------------------------------------

# 80. `route` vs `ip route`

Older:

``` bash
route -n
```

Modern:

``` bash
ip route
```

Prefer `ip route`.

------------------------------------------------------------------------

# 81. `arp` vs `ip neigh`

Older:

``` bash
arp -n
```

Modern:

``` bash
ip neigh
```

Prefer `ip neigh`.

------------------------------------------------------------------------

# 82. Production Command Sheet

### Interface

``` bash
ip -br link
ip -br addr
```

### Address

``` bash
ip addr show eth0
```

### Routes

``` bash
ip route
ip route get <destination>
```

### Neighbors

``` bash
ip neigh
```

### Statistics

``` bash
ip -s link
```

### Listening sockets

``` bash
ss -lntp
```

### Connectivity

``` bash
nc -vz <host> <port>
```

### Packet capture

``` bash
sudo tcpdump -ni any host <destination>
```

------------------------------------------------------------------------

# 83. Practical Lab --- Interface Discovery

Run:

``` bash
ip -br link
ip -br addr
```

Create a table:

``` text
Interface
State
IPv4
IPv6
```

Explain the role of every interface.

------------------------------------------------------------------------

# 84. Practical Lab --- Route and Interface

Run:

``` bash
ip route
ip route get 8.8.8.8
```

Determine:

``` text
source IP
interface
gateway
```

Then verify:

``` bash
ip addr show <interface>
```

------------------------------------------------------------------------

# 85. Practical Lab --- Neighbor Table

Run:

``` bash
ip neigh
```

Identify:

``` text
IP
MAC
interface
state
```

Then generate traffic to a local gateway and observe whether the
neighbor state changes.

------------------------------------------------------------------------

# 86. Practical Lab --- Listening Application

Start:

``` bash
python3 -m http.server 8080
```

Check:

``` bash
ss -lntp | grep 8080
```

Then:

``` bash
curl http://127.0.0.1:8080
```

Map:

``` text
process
→ socket
→ port
→ IP
→ interface
```

------------------------------------------------------------------------

# 87. Practical Lab --- Container Interface

Run:

``` bash
docker run -d --name net-test nginx
```

Inspect:

``` bash
docker inspect net-test
docker network inspect bridge
```

Then compare with:

``` bash
ip link
ip addr
```

Understand:

``` text
container namespace
veth
bridge
host interface
```

------------------------------------------------------------------------

# 88. Practical Lab --- Kubernetes Pod Interface

Run:

``` bash
kubectl get pod -o wide -n <namespace>
```

Then:

``` bash
kubectl exec -it <pod> -n <namespace> -- ip addr
kubectl exec -it <pod> -n <namespace> -- ip route
```

Compare:

``` text
Pod networking
vs
Node networking
```

------------------------------------------------------------------------

# 89. Practical Lab --- MTU

Check:

``` bash
ip link show
```

Find MTU.

Then carefully test packet sizes with:

``` bash
ping -M do -s <size> <destination>
```

Record:

``` text
successful size
failed size
path
```

------------------------------------------------------------------------

# 90. Practical Lab --- Interface Failure Simulation

In an isolated lab environment only:

``` bash
sudo ip link set <interface> down
```

Observe:

``` text
ping
curl
route
application
```

Then restore:

``` bash
sudo ip link set <interface> up
```

Never perform destructive interface experiments on production systems
without an approved procedure.

------------------------------------------------------------------------

# 91. Practical Lab --- EKS Network Investigation

For a test EKS environment:

``` bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
kubectl describe node <node>
```

Investigate:

``` text
node IP
Pod IP
subnet
availability zone
```

Then correlate with AWS networking.

------------------------------------------------------------------------

# 92. Interview --- What Is a NIC?

> A Network Interface Card is the interface through which a system
> connects to a network. In cloud environments it is often virtualized;
> in AWS, an Elastic Network Interface provides virtual network
> connectivity with properties such as private IP addresses, MAC
> address, subnet association and security groups.

------------------------------------------------------------------------

# 93. Interview --- MAC vs IP?

> MAC is a Layer 2 identifier used for local link delivery, while IP is
> a Layer 3 logical address used for routing between networks.

------------------------------------------------------------------------

# 94. Interview --- Can One Interface Have Multiple IPs?

Yes.

Linux supports multiple addresses on one interface.

Example:

``` text
eth0
 ├── 10.0.1.20/24
 └── 10.0.1.21/24
```

Cloud ENIs can also have multiple IP addresses according to resource and
configuration limits.

------------------------------------------------------------------------

# 95. Interview --- What Is ARP?

> ARP resolves an IPv4 address to a MAC address on a local network.
> Linux exposes neighbor information using `ip neigh`.

------------------------------------------------------------------------

# 96. Interview --- Does IPv6 Use ARP?

No.

IPv6 uses Neighbor Discovery through ICMPv6.

------------------------------------------------------------------------

# 97. Interview --- What Is a veth Pair?

> A veth pair is a pair of virtual Ethernet interfaces connected to each
> other. It is commonly used to connect a container or network namespace
> to a host-side bridge or networking system.

------------------------------------------------------------------------

# 98. Interview --- What Is a Network Namespace?

> A Linux network namespace provides an isolated networking environment
> with its own interfaces, addresses, routes and sockets. Containers
> commonly use network namespaces.

------------------------------------------------------------------------

# 99. Interview --- How Does a Pod Network Differ From a Container?

> Kubernetes Pods share a network namespace among their containers.
> Therefore containers within the same Pod normally share one Pod IP and
> can communicate through localhost.

------------------------------------------------------------------------

# 100. Interview --- What Is an AWS ENI?

> An Elastic Network Interface is a virtual network interface in AWS. It
> can have private IP addresses, a MAC address, subnet association and
> security groups. EKS networking uses ENIs extensively, particularly
> with the AWS VPC CNI.

------------------------------------------------------------------------

# 101. Interview --- Why Can EKS Pods Fail Because of IP Exhaustion?

Because VPC-native Pod networking consumes available IP capacity from
the configured network address pools. If subnet IPs or ENI/IP allocation
capacity becomes exhausted, new Pods may not receive addresses.

Investigate:

``` text
subnet capacity
ENI limits
instance type
CNI configuration
CNI logs
```

------------------------------------------------------------------------

# 102. Interview --- What Happens If an Interface Is Down?

Traffic using that interface cannot normally be transmitted through it.

Check:

``` bash
ip link
ip addr
ip route
```

Then determine whether another route/interface can provide connectivity.

------------------------------------------------------------------------

# 103. Interview --- What Is MTU?

> MTU is the maximum transmission unit supported for packets on an
> interface/path without fragmentation at that point. MTU mismatches can
> cause intermittent or size-dependent connectivity failures.

------------------------------------------------------------------------

# 104. Interview --- Why Do Containers Need Virtual Interfaces?

> Containers commonly use isolated network namespaces. A virtual
> Ethernet pair connects the container namespace to host/container
> networking, allowing traffic to move between the container and broader
> network.

------------------------------------------------------------------------

# 105. Interview --- How Do You Troubleshoot a Pod Network Issue?

I would check:

``` text
Pod IP
Pod network namespace
interface
route
DNS
Service
NetworkPolicy
CNI
node networking
VPC routing
security groups
NACL
```

Commands:

``` bash
kubectl get pod -o wide
kubectl describe pod
kubectl exec
ip addr
ip route
ss
nc
dig
tcpdump
```

------------------------------------------------------------------------

# 106. Production Architecture --- Linux Host

``` text
Application
     |
   Socket
     |
   TCP/UDP
     |
     IP
     |
   eth0
     |
  Gateway
     |
   Network
```

------------------------------------------------------------------------

# 107. Production Architecture --- Docker

``` text
Application
     |
Container namespace
     |
   eth0
     |
   veth pair
     |
docker bridge
     |
   host NIC
     |
   network
```

------------------------------------------------------------------------

# 108. Production Architecture --- EKS

``` text
Pod
 |
Pod network namespace
 |
eth0
 |
AWS VPC CNI
 |
ENI / VPC IP connectivity
 |
VPC route/security
 |
AWS network
```

Exact implementation depends on EKS networking mode and CNI
configuration.

------------------------------------------------------------------------

# 109. Production Architecture --- RoboShop

``` text
                       Internet
                           |
                        Route 53
                           |
                           v
                          ALB
                           |
                           v
                      EKS Ingress
                           |
              +------------+------------+
              |            |            |
              v            v            v
          frontend       cart        catalog
           Service      Service       Service
              |            |            |
              v            v            v
            Pods         Pods         Pods
              |
              +--------------------------+
                                         |
                              Internal Services
                                         |
                         +---------------+---------------+
                         |               |               |
                         v               v               v
                       Redis          RabbitMQ          DB
```

Every arrow represents a network boundary that can fail.

------------------------------------------------------------------------

# 110. Production Network Interface Checklist

``` text
[ ] Interface exists
[ ] Interface is UP
[ ] Link is available
[ ] MAC is correct
[ ] IP is correct
[ ] CIDR is correct
[ ] Route exists
[ ] Gateway is correct
[ ] Neighbor resolution works
[ ] MTU is appropriate
[ ] RX/TX errors are acceptable
[ ] RX/TX drops are understood
[ ] Security controls allow required traffic
[ ] DNS works
[ ] Application listens on expected address/port
```

------------------------------------------------------------------------

# 111. Key Mental Model

Always connect:

``` text
Interface
   ↓
MAC
   ↓
IP
   ↓
Subnet
   ↓
Route
   ↓
Gateway
   ↓
Port
   ↓
Protocol
   ↓
Application
```

For Kubernetes:

``` text
Pod
 ↓
Network Namespace
 ↓
eth0
 ↓
CNI
 ↓
Node/VPC
 ↓
Service/Ingress
 ↓
ALB
 ↓
Internet
```

For AWS:

``` text
ENI
 ↓
Private IP
 ↓
Subnet
 ↓
Route Table
 ↓
Security Group/NACL
 ↓
VPC
```

------------------------------------------------------------------------

# 112. Final Summary

A network interface is the operating system's connection point to a
network.

A MAC address identifies the link-layer interface.

An IP address provides logical network addressing.

A subnet defines an address range and routing boundary.

A route determines where packets go.

A gateway provides the next hop to other networks.

A port identifies a transport endpoint.

A protocol defines communication behavior.

In production:

``` text
NIC / ENI
 ↓
MAC
 ↓
IP
 ↓
CIDR
 ↓
Route
 ↓
Gateway
 ↓
TCP/UDP
 ↓
Application
```

Understanding this chain is essential for Linux, AWS, Docker, Kubernetes
and EKS troubleshooting.

------------------------------------------------------------------------

# 113. Next Topic

Next:

``` text
04-IPv4-and-IPv6.md
```

It will go deeply into:

``` text
IPv4 structure
binary representation
network/host portions
subnet masks
CIDR
private/public addressing
loopback
link-local
APIPA
IPv6 structure
IPv6 address types
global unicast
link-local
multicast
unique local addresses
IPv6 routing
dual stack
AWS IPv6
EKS IPv6
Docker IPv6
address troubleshooting
production design
interview preparation
```

# End of 03-MAC-IP-and-Network-Interfaces.md
