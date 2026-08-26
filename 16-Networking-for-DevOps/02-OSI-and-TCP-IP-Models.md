# 16-Networking-for-DevOps

# 02-OSI-and-TCP-IP-Models.md

# OSI and TCP/IP Models

## Purpose

The OSI and TCP/IP models provide mental frameworks for understanding
how network communication works.

A DevOps engineer does not need to memorize networking theory without
context. The important goal is to use the models to answer practical
questions:

``` text
Where did the communication fail?
Which protocol is involved?
Which device is responsible?
Which layer should I investigate?
```

These models are especially useful when troubleshooting:

-   Linux connectivity
-   AWS VPC networking
-   ALB/NLB
-   Kubernetes networking
-   EKS
-   Docker
-   DNS
-   HTTP/HTTPS
-   TLS
-   firewalls
-   NetworkPolicies
-   service-to-service communication
-   CI/CD connectivity
-   GitOps connectivity

------------------------------------------------------------------------

# 1. What Is a Network Model?

A network model divides communication into logical layers.

Instead of treating networking as one large system:

``` text
Application
Network
Hardware
???
```

we separate responsibilities.

For example:

``` text
Application
Transport
Network
Data Link
Physical
```

Each layer has a specific responsibility and interacts with adjacent
layers.

------------------------------------------------------------------------

# 2. Why Layering Exists

Networking is complex.

Without layering, every application would need to understand:

``` text
Ethernet
MAC
IP
routing
TCP
TLS
HTTP
```

Instead:

``` text
Application
    ↓
Transport
    ↓
Network
    ↓
Data Link
    ↓
Physical
```

Each layer hides implementation details from the layer above.

This creates:

-   modularity
-   interoperability
-   easier troubleshooting
-   protocol independence
-   simpler design

------------------------------------------------------------------------

# 3. OSI Model

OSI stands for:

``` text
Open Systems Interconnection
```

The OSI model contains seven layers:

``` text
7 Application
6 Presentation
5 Session
4 Transport
3 Network
2 Data Link
1 Physical
```

Mnemonic:

``` text
All
People
Seem
To
Need
Data
Processing
```

From bottom to top:

``` text
Physical
Data Link
Network
Transport
Session
Presentation
Application
```

------------------------------------------------------------------------

# 4. Seven OSI Layers

    Layer Name           Main responsibility
  ------- -------------- ----------------------------------------------
        7 Application    Network services to applications
        6 Presentation   Data representation, encryption, compression
        5 Session        Session establishment/management
        4 Transport      End-to-end transport
        3 Network        Logical addressing and routing
        2 Data Link      Frames and local delivery
        1 Physical       Signals and physical transmission

The OSI model is primarily a conceptual reference model.

Real protocols do not always map perfectly to exactly one layer.

------------------------------------------------------------------------

# 5. Layer 1 --- Physical

The Physical layer deals with transmission of raw bits/signals.

Examples:

-   cables
-   fiber
-   radio
-   electrical signals
-   optical signals
-   physical connectors
-   transmission media

Conceptually:

``` text
010101010101...
```

At this layer, the concern is how bits physically travel.

------------------------------------------------------------------------

# 6. Physical Layer DevOps Relevance

Cloud engineers rarely troubleshoot physical cables directly in AWS.

However, the concept still matters.

Physical problems can appear as:

``` text
interface down
packet loss
link instability
hardware failure
```

In cloud environments, the provider abstracts most physical
infrastructure.

You still need to understand the logical consequences.

------------------------------------------------------------------------

# 7. Layer 2 --- Data Link

The Data Link layer provides communication over a local network segment.

Important concepts:

``` text
Ethernet
MAC addresses
frames
switching
VLANs
ARP-related local delivery behavior
```

Primary addressing concept:

``` text
MAC address
```

Primary data unit:

``` text
Frame
```

------------------------------------------------------------------------

# 8. Frame

At Layer 2, data is transported in frames.

Conceptually:

``` text
+---------------------------+
| Ethernet Header           |
+---------------------------+
| Payload                   |
+---------------------------+
| Frame Check Sequence      |
+---------------------------+
```

The exact structure depends on the link technology.

------------------------------------------------------------------------

# 9. MAC Address

A MAC address is associated with a network interface at the link layer.

Example:

``` text
02:42:ac:11:00:02
```

Switches use Layer 2 information to forward frames within local
networks.

------------------------------------------------------------------------

# 10. Switch

A switch connects devices in a local network.

``` text
Host A ----+
           |
Host B ----+---- Switch
           |
Host C ----+
```

A switch learns which MAC addresses are reachable through which ports
and uses that information to forward frames.

------------------------------------------------------------------------

# 11. Layer 2 Broadcast

Some Layer 2 traffic is broadcast within a local broadcast domain.

Conceptually:

``` text
Host A
  |
  | Broadcast
  v
Switch
 / | \
B  C  D
```

Broadcast behavior is one reason network segmentation matters.

------------------------------------------------------------------------

# 12. VLAN

A VLAN logically separates Layer 2 networks.

Conceptually:

``` text
Switch
 |
 +-- VLAN 10
 |
 +-- VLAN 20
```

Hosts in separate VLANs are logically separated at Layer 2.

Communication between them generally requires Layer 3 routing.

Cloud networking implements segmentation differently, but the conceptual
purpose---separating network domains---is still important.

------------------------------------------------------------------------

# 13. Layer 3 --- Network

Layer 3 is responsible primarily for:

-   logical addressing
-   routing
-   packet forwarding

Main protocol:

``` text
IP
```

Primary address:

``` text
IP address
```

Primary data unit:

``` text
Packet
```

------------------------------------------------------------------------

# 14. IP Packet

Conceptually:

``` text
+------------------------+
| IP Header              |
+------------------------+
| Payload                |
+------------------------+
```

Important IPv4 header information includes:

-   source IP
-   destination IP
-   TTL
-   protocol
-   fragmentation-related fields

------------------------------------------------------------------------

# 15. Router

Routers operate primarily at Layer 3.

``` text
Network A
    |
    v
 Router
    |
    v
Network B
```

A router examines destination information and consults routing
information to determine the next hop.

------------------------------------------------------------------------

# 16. Routing Table

Example:

``` text
Destination       Next Hop       Interface

10.0.1.0/24       local          eth0
10.0.2.0/24       10.0.1.1       eth0
0.0.0.0/0         10.0.1.1       eth0
```

Linux:

``` bash
ip route
```

AWS uses route tables associated with subnets.

------------------------------------------------------------------------

# 17. Longest Prefix Match

When multiple routes match a destination, routing generally prefers the
most specific matching route.

Example:

``` text
10.0.0.0/8
10.1.0.0/16
10.1.2.0/24
```

For:

``` text
10.1.2.50
```

the `/24` route is more specific.

This is an important routing concept.

------------------------------------------------------------------------

# 18. Layer 4 --- Transport

The Transport layer provides end-to-end transport between application
endpoints.

Main protocols:

``` text
TCP
UDP
```

Key concepts:

-   ports
-   connections
-   reliability
-   ordering
-   flow control
-   congestion control

------------------------------------------------------------------------

# 19. TCP at Layer 4

TCP provides:

``` text
connection establishment
reliable delivery
ordered byte stream
retransmission
flow control
congestion control
```

Example:

``` text
Client
10.0.1.10:51532
      |
      v
Server
10.0.2.20:443
```

------------------------------------------------------------------------

# 20. UDP at Layer 4

UDP provides a lightweight datagram transport.

It does not provide TCP's built-in:

``` text
connection establishment
ordered delivery
retransmission
```

Applications can implement their own reliability when required.

------------------------------------------------------------------------

# 21. Ports and Layer 4

Ports identify transport endpoints.

Examples:

``` text
TCP 22
TCP 443
TCP 5432
TCP 5672
UDP 53
```

A firewall rule such as:

``` text
allow TCP 443
```

is operating with Layer 3/Layer 4 information.

------------------------------------------------------------------------

# 22. Layer 5 --- Session

The Session layer conceptually handles:

-   establishing sessions
-   maintaining sessions
-   coordinating sessions
-   terminating sessions

In modern TCP/IP networking, session responsibilities are often
implemented by application protocols or libraries rather than a distinct
universal Layer 5 protocol.

This is why the OSI model should be treated as a conceptual model rather
than a strict implementation map.

------------------------------------------------------------------------

# 23. Layer 6 --- Presentation

The Presentation layer conceptually deals with data representation.

Responsibilities can include:

-   encoding
-   decoding
-   encryption
-   decryption
-   compression
-   serialization

Examples that may involve presentation-like responsibilities:

``` text
JSON
XML
UTF-8
TLS
compression
serialization formats
```

Again, real implementations do not always fit cleanly into one OSI
layer.

------------------------------------------------------------------------

# 24. Layer 7 --- Application

The Application layer is closest to the user/application.

Examples:

``` text
HTTP
DNS
SSH
SMTP
FTP
```

DevOps engineers work heavily at this layer.

Examples:

``` bash
curl
dig
ssh
wget
```

------------------------------------------------------------------------

# 25. OSI Layer Summary

``` text
Layer 7  Application
         HTTP, DNS, SSH

Layer 6  Presentation
         encoding, encryption, representation

Layer 5  Session
         session management

Layer 4  Transport
         TCP, UDP, ports

Layer 3  Network
         IP, routing

Layer 2  Data Link
         Ethernet, MAC, frames

Layer 1  Physical
         signals, cables, radio
```

------------------------------------------------------------------------

# 26. TCP/IP Model

The TCP/IP model is closer to real Internet protocol architecture.

A commonly used four-layer representation is:

``` text
Application
Transport
Internet
Link
```

Some references use a five-layer model by separating Physical from Link.

------------------------------------------------------------------------

# 27. TCP/IP Four-Layer Model

``` text
4 Application
3 Transport
2 Internet
1 Link
```

Examples:

  Layer         Examples
  ------------- -----------------
  Application   HTTP, DNS, SSH
  Transport     TCP, UDP
  Internet      IP, ICMP
  Link          Ethernet, Wi-Fi

------------------------------------------------------------------------

# 28. TCP/IP vs OSI

Mapping:

``` text
OSI                         TCP/IP

Application   ┐
Presentation  ├──────────→ Application
Session       ┘

Transport    ───────────→ Transport

Network      ───────────→ Internet

Data Link    ┐
Physical     ┘──────────→ Link
```

The TCP/IP model combines several OSI responsibilities.

------------------------------------------------------------------------

# 29. Five-Layer Practical Model

Many engineers use:

``` text
Application
Transport
Network
Data Link
Physical
```

This is often convenient for troubleshooting.

It preserves the practical distinction between:

``` text
Network
Data Link
Physical
```

while avoiding the need to treat Session and Presentation as separate
universal layers.

------------------------------------------------------------------------

# 30. Why DevOps Engineers Use the Models

Suppose an application cannot connect to a database.

Instead of guessing:

``` text
restart database
restart Pod
restart node
```

ask:

``` text
Application?
Transport?
Network?
Link?
```

Then test systematically.

------------------------------------------------------------------------

# 31. Layer-Based Troubleshooting

Example:

``` text
Application
   |
   | HTTP/SQL
   v
Transport
   |
   | TCP 443/5432
   v
Network
   |
   | IP/routing
   v
Data Link
   |
   | Ethernet/VPC networking
   v
Physical
```

This creates a troubleshooting framework.

------------------------------------------------------------------------

# 32. Layer 1 Troubleshooting

Typical symptoms:

``` text
interface down
link down
physical errors
```

Linux:

``` bash
ip link
```

In cloud environments, physical infrastructure is usually
provider-managed.

------------------------------------------------------------------------

# 33. Layer 2 Troubleshooting

Look for:

``` text
MAC behavior
ARP/neighbour resolution
VLAN/segment issues
interface state
local network delivery
```

Linux:

``` bash
ip neigh
```

Example:

``` text
10.0.1.1 dev eth0 lladdr aa:bb:cc:dd:ee:ff REACHABLE
```

------------------------------------------------------------------------

# 34. Layer 3 Troubleshooting

Look for:

``` text
IP address
subnet
route
gateway
routing table
```

Commands:

``` bash
ip addr
ip route
ip route get <destination>
```

------------------------------------------------------------------------

# 35. Layer 4 Troubleshooting

Look for:

``` text
TCP/UDP
port
listener
connection state
timeouts
resets
```

Commands:

``` bash
ss -lntp
ss -nt
nc -vz <host> <port>
```

------------------------------------------------------------------------

# 36. Layer 7 Troubleshooting

Look for:

``` text
DNS
HTTP
TLS
application protocol
application response
```

Commands:

``` bash
dig <domain>
curl -v <url>
openssl s_client -connect <host>:443 -servername <host>
```

------------------------------------------------------------------------

# 37. Example: SSH Failure

User reports:

``` bash
ssh user@server
```

fails.

Layered approach:

``` text
DNS?
 ↓
IP?
 ↓
Route?
 ↓
TCP 22?
 ↓
SSH service?
 ↓
Authentication?
```

Commands:

``` bash
dig server.example.com
ip route get <server-ip>
nc -vz <server-ip> 22
ss -lntp
```

------------------------------------------------------------------------

# 38. Example: HTTPS Failure

``` text
DNS
 ↓
IP
 ↓
TCP 443
 ↓
TLS
 ↓
HTTP
 ↓
Application
```

Commands:

``` bash
dig app.example.com
nc -vz app.example.com 443
openssl s_client -connect app.example.com:443 -servername app.example.com
curl -v https://app.example.com
```

------------------------------------------------------------------------

# 39. Example: Database Failure

``` text
Application
 ↓
DNS
 ↓
IP
 ↓
Route
 ↓
TCP 5432
 ↓
PostgreSQL
 ↓
Authentication
 ↓
Query
```

A successful TCP connection does not guarantee database authentication
or query success.

------------------------------------------------------------------------

# 40. Encapsulation

Encapsulation means each layer adds its own control information as data
moves down the stack.

Application data:

``` text
HTTP data
```

Transport:

``` text
TCP
+
HTTP data
```

Network:

``` text
IP
+
TCP
+
HTTP
```

Link:

``` text
Ethernet
+
IP
+
TCP
+
HTTP
```

------------------------------------------------------------------------

# 41. Encapsulation Diagram

``` text
Application
   |
   | HTTP
   v
+------------------+
| TCP Header       |
| HTTP Data        |
+------------------+
        |
        v
+------------------+
| IP Header        |
| TCP Segment      |
+------------------+
        |
        v
+------------------+
| Ethernet Header  |
| IP Packet        |
+------------------+
```

------------------------------------------------------------------------

# 42. Decapsulation

At the destination, the reverse happens.

``` text
Ethernet
   ↓
IP
   ↓
TCP
   ↓
HTTP
   ↓
Application
```

Each layer processes the information relevant to it and passes the
payload upward.

------------------------------------------------------------------------

# 43. Protocol Data Units

Common terminology:

  Layer         PDU
  ------------- --------------------------------
  Application   Data
  Transport     Segment (TCP) / Datagram (UDP)
  Network       Packet
  Data Link     Frame
  Physical      Bits

Terminology can vary slightly by textbook.

------------------------------------------------------------------------

# 44. Segment vs Datagram

TCP data is commonly called a:

``` text
TCP segment
```

UDP data is commonly called a:

``` text
UDP datagram
```

Both are transported inside IP packets.

------------------------------------------------------------------------

# 45. Packet vs Frame

A packet belongs primarily to Layer 3.

A frame belongs primarily to Layer 2.

Example:

``` text
Frame
 └── Packet
      └── Segment
           └── Application data
```

This distinction is useful when reading packet captures.

------------------------------------------------------------------------

# 46. Addressing at Different Layers

Different layers use different identifiers.

``` text
Layer 2 → MAC
Layer 3 → IP
Layer 4 → Port
Layer 7 → Application names/URLs/protocol semantics
```

Example:

``` text
MAC:
02:42:ac:11:00:02

IP:
10.0.1.20

Port:
443

Application:
HTTPS
```

------------------------------------------------------------------------

# 47. Layer 2 vs Layer 3

Layer 2:

``` text
local network
MAC
frames
switching
```

Layer 3:

``` text
different networks
IP
packets
routing
```

Simple mental model:

``` text
Switch → local forwarding
Router → between-network forwarding
```

Real devices can perform functions across multiple layers, so this is a
conceptual simplification.

------------------------------------------------------------------------

# 48. Layer 3 vs Layer 4

Layer 3:

``` text
Where is the destination?
```

Layer 4:

``` text
Which transport/application endpoint?
```

Example:

``` text
10.0.1.20
    |
    +-- TCP 443
    +-- TCP 22
    +-- TCP 8080
```

One IP can host many services on different ports.

------------------------------------------------------------------------

# 49. Layer 4 vs Layer 7 Load Balancing

Layer 4:

``` text
TCP/UDP
IP
port
```

Layer 7:

``` text
HTTP
Host
Path
Headers
Cookies
```

Example:

``` text
app.example.com/cart
```

An L7 load balancer can route based on:

``` text
/cart
```

A pure L4 load balancer generally does not understand HTTP paths.

------------------------------------------------------------------------

# 50. AWS Mapping

Typical AWS concepts can be related to the layers:

``` text
VPC routing
    → Layer 3

Security Groups
    → Layer 3/4 filtering

NACLs
    → Layer 3/4 filtering

NLB
    → Layer 4-oriented

ALB
    → Layer 7-oriented

Route 53
    → DNS/Application-layer service

WAF
    → Application-layer security
```

These are conceptual mappings, not strict single-layer classifications.

------------------------------------------------------------------------

# 51. Kubernetes Mapping

Kubernetes networking also spans layers.

``` text
Pod IP
     → Layer 3

Service port
     → Layer 4

Ingress HTTP routing
     → Layer 7

NetworkPolicy
     → primarily Layer 3/4 controls
```

This helps connect networking theory to Kubernetes.

------------------------------------------------------------------------

# 52. EKS Mapping

An EKS request can involve:

``` text
Internet
 ↓
Route 53
 ↓
ALB
 ↓
Target group
 ↓
EKS networking
 ↓
Service
 ↓
Pod
```

Potential failures can exist at:

``` text
DNS
Layer 3 routing
Layer 4 connectivity
Layer 7 HTTP
Kubernetes control plane/data plane
```

------------------------------------------------------------------------

# 53. Docker Networking Mapping

Docker networking involves:

``` text
container interface
veth
bridge
IP
NAT
ports
```

Example:

``` text
Host
 |
Docker bridge
 |
Container
```

Later Docker networking notes can be analyzed using the same layer
model.

------------------------------------------------------------------------

# 54. Kubernetes Service Example

Consider:

``` yaml
apiVersion: v1
kind: Service
metadata:
  name: cart
spec:
  selector:
    app: cart
  ports:
    - port: 80
      targetPort: 8080
```

Conceptually:

``` text
Client
 ↓
Service :80
 ↓
Pod :8080
```

Layer 4 concepts are heavily involved.

------------------------------------------------------------------------

# 55. Kubernetes Ingress Example

``` text
Client
 |
 | HTTPS
 v
ALB
 |
 | HTTP
 v
Ingress
 |
 | path routing
 v
Service
 |
 v
Pod
```

The ALB/Ingress layer understands application-level routing.

------------------------------------------------------------------------

# 56. DNS and OSI

DNS is an application-layer protocol in the OSI/TCP-IP conceptual model.

Typical flow:

``` text
Application
   |
DNS query
   |
UDP/TCP
   |
IP
   |
Link
```

Modern DNS can use other transports such as DNS over TLS or DNS over
HTTPS, which changes the transport/encapsulation details.

------------------------------------------------------------------------

# 57. HTTPS and OSI

Conceptually:

``` text
Application → HTTP
Security/representation → TLS
Transport → TCP
Network → IP
Link → Ethernet/Wi-Fi
```

In real protocol stacks, TLS does not cleanly correspond to only OSI
Layer 6; it is better understood as a protocol layer between application
protocols and transport.

------------------------------------------------------------------------

# 58. SSH and OSI

SSH is an application-layer protocol.

Typical stack:

``` text
SSH
 ↓
TCP
 ↓
IP
 ↓
Ethernet/Wi-Fi
```

SSH default port:

``` text
22
```

------------------------------------------------------------------------

# 59. What Happens During HTTPS?

Simplified:

``` text
DNS
 ↓
destination IP
 ↓
TCP connection
 ↓
TLS handshake
 ↓
HTTP request
 ↓
HTTP response
```

Layer mapping:

``` text
DNS/HTTP → Application
TCP      → Transport
IP       → Network
Ethernet → Data Link
```

------------------------------------------------------------------------

# 60. What Happens During a Kubernetes Request?

Example:

``` text
User
 ↓
DNS
 ↓
ALB
 ↓
Ingress routing
 ↓
Service
 ↓
Pod
```

At different points:

``` text
DNS      → Application
ALB      → L4/L7 depending on behavior
Service  → L4
Pod IP   → L3
Ethernet/VPC → lower layers
```

------------------------------------------------------------------------

# 61. Packet Capture and Layers

When using:

``` bash
tcpdump
```

you can inspect information from multiple layers.

Example:

``` bash
sudo tcpdump -nn -i eth0 port 443
```

You may see:

``` text
source IP
destination IP
source port
destination port
TCP flags
```

For deeper protocol inspection, tools can decode higher-layer protocols.

------------------------------------------------------------------------

# 62. TCP Flags

Common TCP flags:

``` text
SYN
ACK
FIN
RST
PSH
URG
```

Useful during troubleshooting.

Examples:

``` text
SYN
```

usually indicates connection initiation.

``` text
SYN-ACK
```

indicates the server responded to the initiation.

``` text
RST
```

indicates reset.

------------------------------------------------------------------------

# 63. SYN Flood Concept

A SYN flood abuses TCP connection establishment by sending many SYN
requests and attempting to exhaust server resources.

This demonstrates that transport-layer behavior can become a security
concern.

Mitigations can include:

-   provider protections
-   load balancers
-   SYN cookies
-   rate limiting
-   network controls

------------------------------------------------------------------------

# 64. MTU

MTU means Maximum Transmission Unit.

It defines the largest packet payload size supported by a network
interface/path without fragmentation at that layer.

Common Ethernet MTU:

``` text
1500 bytes
```

Cloud/container environments can use different effective MTUs depending
on encapsulation and networking technology.

------------------------------------------------------------------------

# 65. MTU Problems

MTU issues can cause:

``` text
intermittent connectivity
large packets failing
TLS/application hangs
fragmentation
packet loss
```

This can be particularly confusing because:

``` text
small packets work
large packets fail
```

is a valuable clue.

------------------------------------------------------------------------

# 66. Path MTU

Path MTU is the smallest MTU supported along a network path.

Example:

``` text
Host A
 ↓
Router
 ↓
Firewall
 ↓
Router
 ↓
Host B
```

The effective path MTU may be smaller than an endpoint interface's MTU.

------------------------------------------------------------------------

# 67. TTL

TTL means Time To Live in IPv4.

It limits how many Layer 3 hops a packet can traverse.

Each router generally decrements TTL.

If it reaches zero, the packet is discarded.

This mechanism helps prevent packets from circulating indefinitely.

Traceroute relies on TTL behavior.

------------------------------------------------------------------------

# 68. IPv6 and OSI

IPv6 occupies the Internet/Network layer conceptually, just like IPv4.

Important differences include:

-   128-bit addresses
-   different header design
-   Neighbor Discovery
-   no broadcast in the IPv4 sense
-   extensive use of multicast
-   simplified base header

DevOps engineers increasingly encounter IPv6 in cloud and enterprise
environments.

------------------------------------------------------------------------

# 69. ARP

ARP maps IPv4 addresses to MAC addresses on local networks.

Conceptually:

``` text
Who has 10.0.1.1?
```

The owner responds with its MAC address.

Linux neighbour table:

``` bash
ip neigh
```

IPv6 uses Neighbor Discovery rather than ARP.

------------------------------------------------------------------------

# 70. Layer 2 and Layer 3 Interaction

Suppose:

``` text
Host A
10.0.1.10
```

needs to reach:

``` text
10.0.1.20
```

If they are on the same subnet, Layer 2 delivery can be used after
resolving the destination MAC.

If the destination is outside the local subnet:

``` text
Host
 ↓
Default gateway
 ↓
Router
 ↓
Destination network
```

This is a key networking mental model.

------------------------------------------------------------------------

# 71. Same-Subnet Communication

Example:

``` text
Host A: 10.0.1.10/24
Host B: 10.0.1.20/24
```

Both belong to:

``` text
10.0.1.0/24
```

The sender can treat the destination as local and use Layer 2 delivery.

------------------------------------------------------------------------

# 72. Different-Subnet Communication

Example:

``` text
Host A: 10.0.1.10/24
Host B: 10.0.2.20/24
```

Different networks:

``` text
10.0.1.0/24
10.0.2.0/24
```

Traffic normally goes through a router/default gateway.

``` text
Host A
 ↓
Gateway
 ↓
Router
 ↓
Host B
```

------------------------------------------------------------------------

# 73. DevOps Example --- EKS to RDS

``` text
Pod
 |
 | IP
 v
Node/VPC networking
 |
 | Route
 v
RDS network
 |
 | TCP 5432
 v
PostgreSQL
```

Potential failure layers:

``` text
L3 → routing/security
L4 → TCP 5432
L7 → PostgreSQL authentication/query
```

------------------------------------------------------------------------

# 74. DevOps Example --- Jenkins to ECR

``` text
Jenkins
 |
 DNS
 ↓
ECR endpoint
 |
 TCP 443
 ↓
TLS
 |
 HTTPS API
 ↓
ECR
```

Failure diagnosis:

``` text
DNS
 ↓
route
 ↓
TCP 443
 ↓
TLS
 ↓
authentication
 ↓
registry operation
```

------------------------------------------------------------------------

# 75. DevOps Example --- Argo CD to Git

``` text
Argo CD Repo Server
 |
 DNS
 ↓
Git provider
 |
 TCP 443 or SSH
 ↓
TLS/SSH
 ↓
Git protocol/application
```

Possible failures:

``` text
DNS
proxy
firewall
credentials
certificate
repository permissions
```

------------------------------------------------------------------------

# 76. DevOps Example --- Argo CD to EKS

``` text
Application Controller
 |
 DNS
 ↓
EKS API endpoint
 |
 TCP 443
 ↓
TLS
 ↓
Kubernetes API
 ↓
Authentication
 ↓
Authorization
```

This is a powerful example of why networking, TLS, authentication and
RBAC must be separated during troubleshooting.

------------------------------------------------------------------------

# 77. Security Controls by Layer

A useful mental model:

``` text
L2
MAC/VLAN/local controls

L3
IP/routing/security boundaries

L4
TCP/UDP ports/security groups/NACL behavior

L7
HTTP/WAF/application authentication
```

A control can operate across multiple layers.

------------------------------------------------------------------------

# 78. WAF and Layer 7

A Web Application Firewall understands web/application requests.

It can inspect concepts such as:

``` text
HTTP method
URL path
headers
query parameters
request body
```

Therefore WAF controls are strongly associated with Layer 7.

------------------------------------------------------------------------

# 79. NetworkPolicy

Kubernetes NetworkPolicy commonly controls traffic using:

``` text
Pod selectors
namespace selectors
IP blocks
ports
ingress
egress
```

Conceptually it sits around:

``` text
Layer 3 + Layer 4
```

depending on the CNI implementation.

------------------------------------------------------------------------

# 80. Why Layer Models Prevent Random Troubleshooting

Without a model:

``` text
Application broken
→ restart Pod
→ restart Service
→ restart node
→ change security group
→ change route
```

With a model:

``` text
Does DNS resolve?
 ↓
Does route exist?
 ↓
Does TCP connect?
 ↓
Does TLS succeed?
 ↓
Does HTTP succeed?
 ↓
Is application healthy?
```

The second method is much safer in production.

------------------------------------------------------------------------

# 81. Troubleshooting Matrix

  Symptom               Likely layer(s)
  --------------------- --------------------
  Interface down        L1/L2
  ARP/neighbour issue   L2
  Wrong route           L3
  No route to host      L3/L4/security
  Port closed/refused   L4
  TCP timeout           L3/L4/security
  DNS failure           L7
  TLS failure           TLS/application
  HTTP 404              L7
  HTTP 502              L7/upstream
  HTTP 503              L7/backend
  HTTP 504              L7/network/backend
  Large packets fail    MTU/path
  Pod cannot reach DB   L3/L4/L7

This is a guide, not an absolute classification.

------------------------------------------------------------------------

# 82. Production Troubleshooting Workflow

Use:

``` text
1. Define the symptom.
2. Identify source.
3. Identify destination.
4. Identify protocol.
5. Identify port.
6. Check DNS.
7. Check IP.
8. Check route.
9. Check security.
10. Check TCP/UDP.
11. Check TLS.
12. Check application protocol.
13. Check backend.
14. Confirm root cause.
15. Apply minimal safe change.
16. Verify.
17. Document.
```

------------------------------------------------------------------------

# 83. Practical Exercise --- Map a Request

Take:

``` text
https://shop.example.com/cart
```

Map:

``` text
DNS
 ↓
IP
 ↓
TCP 443
 ↓
TLS
 ↓
HTTP GET /cart
 ↓
ALB
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

Write down the layer involved at each stage.

------------------------------------------------------------------------

# 84. Practical Exercise --- Identify Layer

Classify:

``` text
Wrong subnet route
Closed TCP 443
Invalid TLS certificate
Wrong DNS record
Incorrect HTTP path
MAC resolution issue
```

Expected reasoning:

``` text
Wrong route          → L3
Closed TCP 443       → L4
TLS certificate      → TLS/application boundary
DNS record           → L7
HTTP path             → L7
MAC resolution       → L2
```

------------------------------------------------------------------------

# 85. Practical Exercise --- Use Commands by Layer

### L2

``` bash
ip neigh
```

### L3

``` bash
ip addr
ip route
```

### L4

``` bash
ss -lntp
nc -vz host 443
```

### L7

``` bash
dig example.com
curl -v https://example.com
```

### TLS

``` bash
openssl s_client -connect example.com:443 -servername example.com
```

------------------------------------------------------------------------

# 86. Practical Exercise --- Capture TCP Handshake

Run:

``` bash
sudo tcpdump -nn -i any host <destination> and port 443
```

Then:

``` bash
curl -v https://<destination>
```

Look for:

``` text
SYN
SYN-ACK
ACK
```

Then identify subsequent TLS/application traffic.

------------------------------------------------------------------------

# 87. Practical Exercise --- Find Listening Layer 4 Endpoints

Run:

``` bash
sudo ss -lntp
```

Pick one listening port.

Determine:

``` text
process
IP binding
port
```

Then test:

``` bash
nc -vz 127.0.0.1 <port>
```

This connects process-level behavior to Layer 4 networking.

------------------------------------------------------------------------

# 88. Production Architecture --- Single EKS Cluster

``` text
                    Internet
                       |
                       v
                    Route 53
                       |
                       v
                     ALB
                       |
                  HTTPS / HTTP
                       |
                       v
                 EKS Ingress
                       |
                       v
                Kubernetes Service
                       |
                  +----+----+
                  |         |
                  v         v
                Pod A     Pod B
                  |
                  v
              Database
```

Troubleshooting follows the layers:

``` text
DNS
→ ALB
→ TCP
→ HTTP
→ Service
→ Pod
→ dependency
```

------------------------------------------------------------------------

# 89. Production Architecture --- Multi-Cluster

``` text
                    Route 53
                       |
             +---------+---------+
             |                   |
             v                   v
          ALB-DEV             ALB-PROD
             |                   |
             v                   v
          EKS-DEV             EKS-PROD
             |                   |
           Pods                Pods
             |                   |
          Services            Services
```

Each cluster has independent networking boundaries.

Centralized GitOps does not eliminate the need to understand networking
in each cluster.

------------------------------------------------------------------------

# 90. Interview --- Explain OSI Model

Strong answer:

> The OSI model divides networking into seven conceptual layers:
> Physical, Data Link, Network, Transport, Session, Presentation and
> Application. It helps engineers isolate responsibilities and
> troubleshoot systematically. For example, IP and routing are Layer 3,
> TCP and ports are Layer 4, and HTTP/DNS are Layer 7.

------------------------------------------------------------------------

# 91. Interview --- Explain TCP/IP Model

Strong answer:

> The TCP/IP model is a practical model for Internet networking. A
> common four-layer representation is Application, Transport, Internet
> and Link. HTTP and DNS are application protocols, TCP/UDP are
> transport protocols, IP belongs to the Internet layer, and
> Ethernet/Wi-Fi belong to the Link layer.

------------------------------------------------------------------------

# 92. Interview --- OSI vs TCP/IP?

Strong answer:

``` text
OSI:
7 conceptual layers

TCP/IP:
4 commonly used layers
```

OSI separates:

``` text
Application
Presentation
Session
```

while TCP/IP generally combines those responsibilities into Application.

OSI separates:

``` text
Data Link
Physical
```

while the four-layer TCP/IP model combines them into Link.

------------------------------------------------------------------------

# 93. Interview --- What Is Encapsulation?

> Encapsulation is the process where each networking layer adds its own
> control information as data moves down the protocol stack. For
> example, HTTP data is carried inside TCP, TCP inside IP, and IP inside
> an Ethernet frame.

------------------------------------------------------------------------

# 94. Interview --- What Is Decapsulation?

> Decapsulation is the reverse process at the receiving system. The
> receiver processes the link-layer information, then network, transport
> and application information until the original application data is
> delivered.

------------------------------------------------------------------------

# 95. Interview --- Packet vs Frame?

> A packet is primarily a Layer 3 unit containing IP information. A
> frame is primarily a Layer 2 unit used for local link delivery. A
> frame can carry an IP packet.

------------------------------------------------------------------------

# 96. Interview --- Segment vs Packet?

> A TCP segment is a Layer 4 TCP data unit. An IP packet carries that
> segment at Layer 3.

------------------------------------------------------------------------

# 97. Interview --- Why Does an IP Need a Port?

An IP identifies a network endpoint/host, while a port identifies a
transport/application endpoint.

Example:

``` text
10.0.1.20:22
10.0.1.20:443
10.0.1.20:8080
```

The same IP can support multiple services using different ports.

------------------------------------------------------------------------

# 98. Interview --- What Layer Is DNS?

DNS is generally classified as an application-layer protocol.

However, its messages are transported using protocols such as UDP/TCP
and therefore traverse the lower layers as well.

------------------------------------------------------------------------

# 99. Interview --- What Layer Is TLS?

TLS is often described as sitting between application protocols and
transport rather than fitting cleanly into one OSI layer.

Some simplified interview material maps TLS to Presentation Layer 6, but
a more accurate production answer is:

> TLS provides a security protocol layer between application protocols
> and transport, and the OSI mapping is conceptual rather than exact.

------------------------------------------------------------------------

# 100. Interview --- What Layer Is a Load Balancer?

It depends on the load balancer.

``` text
L4 load balancer → TCP/UDP
L7 load balancer → HTTP/application semantics
```

AWS:

``` text
NLB → L4-oriented
ALB → L7-oriented
```

------------------------------------------------------------------------

# 101. Interview --- How Does This Help Kubernetes?

It lets you separate:

``` text
Pod IP / routing       → L3
Service port            → L4
Ingress path/host       → L7
```

For example:

``` text
Client
 ↓
ALB HTTPS
 ↓
Ingress routing
 ↓
Service port
 ↓
Pod IP:containerPort
```

------------------------------------------------------------------------

# 102. Interview --- How Would You Troubleshoot a 502?

Start with:

``` text
client
 ↓
DNS
 ↓
ALB
 ↓
target health
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

Then verify:

``` text
backend port
protocol
readiness
application listener
logs
```

The exact root cause depends on where the invalid upstream response
occurs.

------------------------------------------------------------------------

# 103. Interview --- How Would You Troubleshoot a TCP Timeout?

Identify:

``` text
source
destination
port
```

Then check:

``` text
DNS
IP
route
security group
NACL
NetworkPolicy
firewall
listener
return path
```

Use:

``` bash
ip route get <destination>
nc -vz <destination> <port>
ss -nt
tcpdump
```

------------------------------------------------------------------------

# 104. Production Best Practices

Remember:

``` text
Understand layers
Do not guess
Test from the actual source
Use the real destination and port
Separate DNS from connectivity
Separate TCP from application health
Use packet captures when necessary
Change one thing at a time
Prefer least-privilege network rules
Document production network paths
```

------------------------------------------------------------------------

# 105. Key Mental Model

``` text
L7
Application
HTTP / DNS / SSH
       ↓
L4
Transport
TCP / UDP / Ports
       ↓
L3
Network
IP / Routing
       ↓
L2
Data Link
MAC / Frames
       ↓
L1
Physical
Signals / Media
```

When debugging:

``` text
"What layer could produce this symptom?"
```

That question often narrows a large production problem into a small
investigation.

------------------------------------------------------------------------

# 106. Final Summary

The OSI model is:

``` text
7 Application
6 Presentation
5 Session
4 Transport
3 Network
2 Data Link
1 Physical
```

The common TCP/IP model is:

``` text
Application
Transport
Internet
Link
```

The practical DevOps mapping is:

``` text
HTTP/DNS/SSH
      ↓
TCP/UDP/Ports
      ↓
IP/Routing
      ↓
MAC/Frames
      ↓
Physical transmission
```

Most importantly:

``` text
Application failure
does not automatically mean
application code is broken.
```

The failure may exist at:

``` text
DNS
→ routing
→ security
→ TCP
→ TLS
→ HTTP
→ backend
```

Use the networking model to isolate the layer before changing production
systems.

------------------------------------------------------------------------

# 107. Next Topic

The next file is:

``` text
03-MAC-IP-and-Network-Interfaces.md
```

It will go deeper into:

``` text
MAC addresses
NICs
Ethernet
ARP
Neighbor tables
IP interfaces
Linux interface configuration
virtual interfaces
veth pairs
bridges
cloud ENIs
EKS networking interfaces
Docker interfaces
container networking
interface troubleshooting
production scenarios
interview questions
```

# End of 02-OSI-and-TCP-IP-Models.md
