# Networking-Fundamentals.md

## Purpose

Networking is a core DevOps foundation. Production DevOps systems depend
on communication between developers, Git providers, CI runners,
registries, cloud APIs, load balancers, Kubernetes components, Pods,
databases and monitoring systems.

This file establishes the networking foundation required for AWS, VPC,
EKS, Kubernetes, Docker, ALB/NLB, DNS, TLS, CI/CD, GitOps, microservices
and production troubleshooting.

------------------------------------------------------------------------

# 1. What Is Networking?

Computer networking is the communication of data between two or more
systems using agreed protocols.

Typical DevOps communication:

``` text
Developer → Git
CI → ECR
Argo CD → Git
Argo CD → EKS API
User → ALB
ALB → Kubernetes Service
Service → Pod
Pod → Database
Pod → Redis
Pod → RabbitMQ
```

A network provides mechanisms for:

-   addressing
-   routing
-   transport
-   name resolution
-   reliability
-   security
-   encryption
-   traffic distribution

------------------------------------------------------------------------

# 2. Why Networking Matters to DevOps

A large percentage of production failures cross a networking boundary.

Examples:

``` text
DNS failure
TCP timeout
connection refused
HTTP 502
HTTP 503
HTTP 504
TLS certificate failure
ALB target unhealthy
ECR unreachable
Kubernetes API unreachable
Database connection failure
```

A strong DevOps engineer should be able to move from:

``` text
"Application is down"
```

to:

``` text
DNS
 ↓
IP
 ↓
Route
 ↓
TCP/UDP
 ↓
Firewall/security
 ↓
TLS
 ↓
HTTP
 ↓
Load balancer
 ↓
Service
 ↓
Pod
 ↓
Application
```

------------------------------------------------------------------------

# 3. Basic Networking Terminology

  Term            Meaning
  --------------- -------------------------------------
  Host            Device connected to a network
  Client          System requesting a service
  Server          System providing a service
  Packet          Network-layer data unit
  Frame           Data-link-layer data unit
  IP address      Logical network address
  MAC address     Link-layer address
  Port            Logical application endpoint
  Protocol        Communication rules
  Router          Connects/forwards between networks
  Switch          Connects systems in a local network
  Firewall        Controls traffic
  DNS             Resolves names
  Proxy           Intermediary for traffic
  Load balancer   Distributes traffic
  Gateway         Entry/exit point between networks

------------------------------------------------------------------------

# 4. Network Types

Common network types include:

-   LAN --- Local Area Network
-   WAN --- Wide Area Network
-   MAN --- Metropolitan Area Network
-   PAN --- Personal Area Network
-   data-center networks
-   cloud virtual networks

AWS example:

``` text
VPC
 |
 +-- Public Subnets
 |
 +-- Private Application Subnets
 |
 +-- Database Subnets
```

------------------------------------------------------------------------

# 5. Network Interface

A network interface connects a system to a network.

Linux commands:

``` bash
ip link
ip addr
```

Typical interfaces:

``` text
eth0
ens5
enp0s3
```

A network interface may have:

-   MAC address
-   IPv4 address
-   IPv6 address
-   subnet information
-   link state

------------------------------------------------------------------------

# 6. MAC Address

A MAC address identifies a network interface at the data-link layer.

Example:

``` text
00:1A:2B:3C:4D:5E
```

MAC addresses are primarily relevant to Layer 2 communication.

------------------------------------------------------------------------

# 7. IP Address

An IP address identifies a network endpoint at the network layer.

IPv4:

``` text
192.168.1.10
```

IPv6:

``` text
2001:db8::10
```

IPv4 uses 32 bits.

IPv6 uses 128 bits.

------------------------------------------------------------------------

# 8. IPv4

IPv4 is represented as four octets:

``` text
192.168.1.10
```

Each octet ranges from:

``` text
0–255
```

An address is interpreted together with a subnet mask/CIDR.

Example:

``` text
10.0.1.20/24
```

The `/24` determines how much of the address identifies the network.

------------------------------------------------------------------------

# 9. Private IPv4 Ranges

Private IPv4 ranges are:

``` text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

They are commonly used for internal networks and AWS VPCs.

Example:

``` text
10.0.0.0/16
```

------------------------------------------------------------------------

# 10. Public IP

A public IP can be globally routed through the Internet, subject to
routing and provider controls.

A common production pattern is:

``` text
Internet
   |
Public-facing ALB
   |
Private application workloads
```

Avoid making internal workloads directly Internet-facing unless there is
a justified design.

------------------------------------------------------------------------

# 11. Loopback

IPv4 loopback:

``` text
127.0.0.1
```

Hostname:

``` text
localhost
```

It refers to the local host.

Example:

``` bash
curl http://127.0.0.1:8080
```

Traffic does not leave the host.

------------------------------------------------------------------------

# 12. 0.0.0.0

`0.0.0.0` is commonly used by servers to mean:

``` text
listen on all local IPv4 interfaces
```

Compare:

``` text
127.0.0.1:8080
```

with:

``` text
0.0.0.0:8080
```

The first is local-only; the second can accept traffic arriving through
configured interfaces, subject to firewall and application behavior.

------------------------------------------------------------------------

# 13. Ports

An IP identifies a host endpoint while a port identifies an application
endpoint.

Example:

``` text
10.0.1.20:443
```

Common ports:

    Port Common use
  ------ -------------------------
      22 SSH
      53 DNS
      80 HTTP
     123 NTP
     443 HTTPS
    3306 MySQL
    5432 PostgreSQL
    5672 RabbitMQ
    6379 Redis
    8080 Common application port
    9090 Common Prometheus port

Ports range from:

``` text
0–65535
```

------------------------------------------------------------------------

# 14. Socket

A socket represents a communication endpoint.

A TCP connection is commonly identified by:

``` text
source IP
source port
destination IP
destination port
protocol
```

Example:

``` text
10.0.1.10:51532
        ↓
10.0.2.20:443
```

The client normally uses an ephemeral source port.

------------------------------------------------------------------------

# 15. Client-Server Model

``` text
Client
10.0.1.10
   |
   | TCP connection
   v
Server
10.0.2.20:443
```

The server listens on a known port.

The client usually connects from an ephemeral port.

------------------------------------------------------------------------

# 16. Protocol

A protocol defines communication rules.

Examples:

``` text
Ethernet
IP
TCP
UDP
ICMP
DNS
HTTP
HTTPS
SSH
TLS
```

Protocols define formats, sequencing, addressing and communication
behavior.

------------------------------------------------------------------------

# 17. TCP

TCP is connection-oriented and provides:

-   reliable delivery
-   ordered delivery
-   retransmission
-   flow control
-   congestion control
-   connection establishment

Typical connection establishment:

``` text
Client                     Server

SYN ---------------------->

    <---------------- SYN-ACK

ACK ---------------------->
```

------------------------------------------------------------------------

# 18. UDP

UDP is connectionless and has lower protocol overhead than TCP.

It does not provide TCP-style:

-   connection establishment
-   guaranteed delivery
-   ordering
-   retransmission

Common examples include DNS queries and some real-time or streaming
traffic.

------------------------------------------------------------------------

# 19. TCP vs UDP

  Feature          TCP              UDP
  ---------------- ---------------- ------------------------
  Connection       Yes              No
  Reliability      Yes              No TCP guarantee
  Ordering         Yes              No
  Retransmission   Yes              No
  Overhead         Higher           Lower
  Examples         HTTPS, SSH, DB   DNS, real-time traffic

Do not assume UDP is always faster or better; protocol choice depends on
requirements.

------------------------------------------------------------------------

# 20. TCP Three-Way Handshake

``` text
Client                     Server

SYN ---------------------->

    <---------------- SYN-ACK

ACK ---------------------->
```

The handshake establishes shared TCP connection state.

------------------------------------------------------------------------

# 21. TCP Connection Termination

Normal TCP termination uses FIN/ACK exchanges.

Simplified:

``` text
Client                     Server

FIN ---------------------->

    <---------------- ACK

    <---------------- FIN

ACK ---------------------->
```

An RST is different from a normal graceful close.

------------------------------------------------------------------------

# 22. TCP Reset

RST indicates that a TCP connection is being reset.

Possible causes include:

-   no listening application
-   application forcibly closing
-   firewall/network device behavior
-   protocol mismatch
-   backend termination

Always investigate the actual context.

------------------------------------------------------------------------

# 23. Connection Refused vs Timeout

These are different symptoms.

### Connection refused

Often indicates active rejection, commonly because no process is
listening on the destination port or a system explicitly rejected it.

Check:

``` bash
ss -lntp
```

### Timeout

No expected response arrived within the timeout.

Possible causes:

``` text
routing
firewall
security group
NACL
network path
unresponsive application
```

------------------------------------------------------------------------

# 24. ICMP and Ping

ICMP is used for network control and diagnostics.

Example:

``` bash
ping 8.8.8.8
```

Ping does NOT prove that:

-   DNS works
-   TCP works
-   port 443 is open
-   HTTPS works
-   the application works

A system may allow ICMP but block TCP 443.

------------------------------------------------------------------------

# 25. Traceroute

Traceroute helps identify network hops.

Commands:

``` bash
traceroute example.com
tracepath example.com
```

A missing response from one hop does not necessarily mean the path is
broken because intermediate devices may filter diagnostic traffic.

------------------------------------------------------------------------

# 26. DNS

DNS stands for Domain Name System.

It resolves names into network information.

Example:

``` text
app.example.com
       |
       v
IP address
```

DNS is used throughout DevOps:

``` text
Git
ECR
EKS
ALB
Databases
Microservices
Monitoring
Internal services
```

------------------------------------------------------------------------

# 27. Common DNS Records

Important records:

``` text
A
AAAA
CNAME
MX
TXT
NS
SOA
PTR
```

Examples:

``` text
A     → IPv4
AAAA  → IPv6
CNAME → another DNS name
MX    → mail exchange
TXT   → text/verification/security data
NS    → authoritative name server
PTR   → reverse lookup
```

------------------------------------------------------------------------

# 28. DNS Troubleshooting

Commands:

``` bash
dig example.com
nslookup example.com
```

Examples:

``` bash
dig A example.com
dig AAAA example.com
dig CNAME example.com
dig +trace example.com
```

Check resolver configuration:

``` bash
cat /etc/resolv.conf
```

------------------------------------------------------------------------

# 29. DNS vs Routing

DNS answers:

``` text
"What destination information does this name represent?"
```

Routing answers:

``` text
"How should packets reach that destination?"
```

Therefore:

``` text
DNS success
≠
network connectivity success
```

------------------------------------------------------------------------

# 30. HTTP

HTTP is an application-layer protocol.

Example:

``` http
GET /health HTTP/1.1
Host: app.example.com
```

A request can contain:

``` text
method
path
headers
body
```

Common methods:

``` text
GET
POST
PUT
PATCH
DELETE
HEAD
OPTIONS
```

------------------------------------------------------------------------

# 31. HTTP Status Codes

### 2xx

Success:

``` text
200 OK
201 Created
204 No Content
```

### 3xx

Redirect/cache-related:

``` text
301
302
304
```

### 4xx

Request/client-side problems:

``` text
400
401
403
404
408
429
```

### 5xx

Server/upstream problems:

``` text
500
502
503
504
```

------------------------------------------------------------------------

# 32. HTTP 502

502 generally indicates that a gateway/proxy received an invalid
response from an upstream server.

Possible causes:

``` text
backend failure
wrong port
protocol mismatch
upstream connection problem
proxy configuration
```

In Kubernetes/AWS:

``` text
Client
 ↓
ALB
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

Check each boundary.

------------------------------------------------------------------------

# 33. HTTP 503

503 indicates that the service is unavailable.

Kubernetes-related causes can include:

``` text
no ready Pods
incorrect Service selector
failed readiness probes
unhealthy load-balancer targets
application unavailable
```

Useful commands:

``` bash
kubectl get pods -n <namespace>
kubectl get svc -n <namespace>
kubectl get endpointslice -n <namespace>
```

------------------------------------------------------------------------

# 34. HTTP 504

504 commonly means a gateway/proxy timed out waiting for an upstream
response.

Possible causes:

``` text
slow application
database latency
network path
security filtering
connection pool exhaustion
dependency failure
```

------------------------------------------------------------------------

# 35. HTTPS

HTTPS is HTTP protected by TLS.

``` text
HTTP + TLS = HTTPS
```

Default HTTPS port:

``` text
443
```

------------------------------------------------------------------------

# 36. TLS

TLS provides:

-   encryption
-   integrity
-   server authentication
-   optional client authentication

Simplified:

``` text
Client
   |
TLS handshake
   |
Certificate validation
   |
Key establishment
   |
Encrypted communication
```

------------------------------------------------------------------------

# 37. TLS Certificate

A certificate associates a domain identity with a public key.

The client validates factors including:

``` text
hostname
certificate chain
validity period
trust
```

For troubleshooting:

``` bash
curl -v https://example.com
```

and:

``` bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com
```

------------------------------------------------------------------------

# 38. Routing

Routing determines how traffic moves between networks.

Example:

``` text
Destination       Gateway
10.0.1.0/24       local
0.0.0.0/0         10.0.1.1
```

The default route is:

``` text
0.0.0.0/0
```

It is used when no more specific route matches.

Linux:

``` bash
ip route
ip route get 8.8.8.8
```

------------------------------------------------------------------------

# 39. Default Gateway

The default gateway is the next-hop destination for traffic that does
not match a more specific route.

Example:

``` text
Host
10.0.1.20
   |
   v
10.0.1.1
Gateway
   |
   v
Other networks
```

------------------------------------------------------------------------

# 40. NAT

NAT means Network Address Translation.

It translates address information as traffic crosses a boundary.

Common cloud pattern:

``` text
Private workload
      |
      v
NAT
      |
      v
Internet
```

Source NAT changes the source address.

Destination NAT changes destination information.

------------------------------------------------------------------------

# 41. Reverse Proxy

A reverse proxy receives traffic on behalf of backend services.

``` text
Client
  |
  v
Reverse Proxy
  |
  +--> App 1
  +--> App 2
```

Common functions:

-   TLS termination
-   routing
-   load balancing
-   header handling
-   connection management

Examples:

``` text
Nginx
ALB
Ingress implementations
```

------------------------------------------------------------------------

# 42. Load Balancing

A load balancer distributes traffic across backends.

``` text
              Load Balancer
              /     |      \
             v      v       v
           App-1  App-2   App-3
```

Benefits:

-   availability
-   scalability
-   traffic distribution
-   health-based routing

------------------------------------------------------------------------

# 43. Layer 4 vs Layer 7

Layer 4 primarily uses transport information:

``` text
TCP
UDP
IP
port
```

Layer 7 understands application protocols such as HTTP and can route
using:

``` text
host
path
headers
```

AWS examples:

``` text
NLB → Layer 4-oriented
ALB → Layer 7-oriented
```

------------------------------------------------------------------------

# 44. ALB Routing

Example:

``` text
app.example.com/cart
        |
        v
       ALB
        |
        +--> /cart
        |      |
        |      v
        |   cart-service
        |
        +--> /catalog
               |
               v
          catalog-service
```

This is common in microservice architectures.

------------------------------------------------------------------------

# 45. Network Security

Network security should be layered.

Example:

``` text
Internet
   |
   v
ALB
   |
Security Group
   |
   v
EKS
   |
NetworkPolicy
   |
   v
Service
   |
   v
Pod
```

Controls can include:

``` text
IAM
Security Groups
NACLs
NetworkPolicies
TLS
application authentication
authorization
```

------------------------------------------------------------------------

# 46. Security Groups vs NACL

AWS distinction:

``` text
Security Group → stateful
NACL            → stateless
```

A stateful control tracks connection state.

A stateless network ACL evaluates traffic according to configured rules
without the same connection-state behavior.

This distinction is critical during AWS troubleshooting.

------------------------------------------------------------------------

# 47. Least Network Access

Allow only required traffic.

Poor:

``` text
Internet
   |
all ports
   |
all workloads
```

Better:

``` text
Internet
   |
TCP 443
   v
ALB
   |
application port
   v
Service
```

Database example:

``` text
Application security group
          |
          | TCP 5432
          v
Database security group
```

Avoid public database exposure unless explicitly justified.

------------------------------------------------------------------------

# 48. Network Segmentation

A production AWS layout may look like:

``` text
VPC
 |
 +-- Public Subnets
 |      |
 |      +-- ALB
 |
 +-- Private App Subnets
 |      |
 |      +-- EKS
 |
 +-- Database Subnets
        |
        +-- RDS
```

Benefits:

-   reduced attack surface
-   isolation
-   controlled routing
-   clearer security boundaries
-   smaller blast radius

------------------------------------------------------------------------

# 49. Linux Networking Commands

Core commands:

``` bash
ip addr
ip link
ip route
ip neigh
ss
ping
traceroute
tracepath
dig
nslookup
curl
wget
nc
tcpdump
openssl
hostname
hostname -I
```

------------------------------------------------------------------------

# 50. `ss`

List listening TCP ports:

``` bash
ss -lnt
```

Include processes:

``` bash
ss -lntp
```

UDP:

``` bash
ss -lnup
```

Established connections:

``` bash
ss -nt
```

------------------------------------------------------------------------

# 51. `nc`

Netcat is useful for TCP connectivity testing:

``` bash
nc -vz 10.0.1.20 443
```

Database example:

``` bash
nc -vz database.internal 5432
```

This helps distinguish:

``` text
DNS problem
vs
TCP connectivity problem
```

------------------------------------------------------------------------

# 52. `curl`

HTTP troubleshooting:

``` bash
curl -v https://example.com
```

Headers:

``` bash
curl -I https://example.com
```

Specify a Host header:

``` bash
curl -H "Host: app.example.com" http://10.0.1.20
```

Timeout:

``` bash
curl --connect-timeout 5 https://example.com
```

------------------------------------------------------------------------

# 53. `tcpdump`

Packet capture:

``` bash
sudo tcpdump -i eth0 port 443
```

Host:

``` bash
sudo tcpdump -i eth0 host 10.0.1.20
```

DNS:

``` bash
sudo tcpdump -i eth0 port 53
```

Use packet capture carefully in production because traffic can contain
sensitive information and capture files can become large.

------------------------------------------------------------------------

# 54. Network Troubleshooting Method

Use a layered approach.

``` text
1. Application
2. DNS
3. IP
4. Routing
5. TCP/UDP
6. Security controls
7. TLS
8. HTTP
9. Load balancer
10. Backend
```

A practical sequence:

``` text
Name resolution
      ↓
Destination IP
      ↓
Route
      ↓
TCP/UDP connection
      ↓
TLS
      ↓
HTTP
      ↓
Application
```

------------------------------------------------------------------------

# 55. Website Down Example

User reports:

``` text
https://shop.example.com
```

Start:

``` bash
dig shop.example.com
```

Then:

``` bash
curl -v https://shop.example.com
```

Then inspect:

``` text
DNS
ALB
Ingress
Service
EndpointSlice
Pods
```

Do not immediately restart Kubernetes components.

------------------------------------------------------------------------

# 56. DNS Works but Application Fails

If:

``` bash
dig shop.example.com
```

returns an address but:

``` bash
curl -v https://shop.example.com
```

times out, investigate:

``` text
routing
security group
NACL
firewall
ALB
target health
```

DNS success does not prove application connectivity.

------------------------------------------------------------------------

# 57. TCP Works but HTTP Fails

If:

``` bash
nc -vz host 443
```

succeeds but HTTP returns:

``` text
503
```

then basic TCP connectivity exists.

Investigate:

``` text
TLS
HTTP
ALB
Ingress
Service
Pods
readiness
application
```

------------------------------------------------------------------------

# 58. HTTPS Fails but HTTP Works

Possible causes:

``` text
certificate
TLS configuration
listener configuration
hostname/SNI
certificate chain
security controls
```

Commands:

``` bash
curl -v http://example.com
curl -v https://example.com
openssl s_client -connect example.com:443 -servername example.com
```

------------------------------------------------------------------------

# 59. Pod Cannot Reach Database

Architecture:

``` text
Pod
 |
 | TCP 5432
 v
Database
```

Inside the Pod:

``` bash
kubectl exec -it <pod> -- sh
```

DNS:

``` bash
getent hosts database.internal
```

TCP:

``` bash
nc -vz database.internal 5432
```

If DNS works but TCP fails, investigate:

``` text
routing
security groups
NACL
NetworkPolicy
database listener
database access rules
```

------------------------------------------------------------------------

# 60. Pod Cannot Reach Internet

Typical path:

``` text
Pod
 ↓
Node / VPC CNI
 ↓
Route Table
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

Check:

``` text
subnet route table
NAT Gateway
security group
NACL
DNS
```

The exact path depends on the EKS/VPC design.

------------------------------------------------------------------------

# 61. Networking in RoboShop

Production-style flow:

``` text
                       Internet
                           |
                           v
                       Route 53
                           |
                           v
                          ALB
                           |
                       HTTPS 443
                           |
                           v
                     Ingress Rules
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
       frontend          cart            catalog
       Service           Service          Service
          |                |                |
          v                v                v
        Pods             Pods             Pods
          |                |                |
          +----------------+----------------+
                           |
                    Internal Services
                           |
              +------------+------------+
              |            |            |
              v            v            v
            Redis       RabbitMQ       DB
```

The exact service topology can vary, but the networking principles
remain.

------------------------------------------------------------------------

# 62. Networking in CI/CD

CI/CD depends on network connectivity:

``` text
Jenkins
   |
 HTTPS
   v
Git Provider
   |
 HTTPS
   v
ECR
   |
 Kubernetes API
   v
EKS
```

A pipeline failure can therefore originate from:

``` text
DNS
proxy
routing
firewall
TLS
authentication
network policy
```

Do not assume every CI failure is a pipeline syntax error.

------------------------------------------------------------------------

# 63. Networking in GitOps

Argo CD needs connectivity to its Git repositories and target Kubernetes
APIs.

``` text
                 Git
                  |
             HTTPS / SSH
                  |
                  v
              Repo Server
                  |
                  v
          Application Controller
                  |
             HTTPS / API
                  |
                  v
                EKS
```

Failure can occur at either boundary:

``` text
Git access
or
cluster access
```

------------------------------------------------------------------------

# 64. Networking in Kubernetes

Kubernetes networking must support:

``` text
Pod → Pod
Pod → Service
Pod → DNS
Pod → External Service
User → Ingress
Ingress → Service
Service → Pod
```

Important Kubernetes networking topics that follow this file:

``` text
CNI
CoreDNS
ClusterIP
NodePort
LoadBalancer
Ingress
NetworkPolicy
kube-proxy
AWS VPC CNI
```

------------------------------------------------------------------------

# 65. Network Observability

Monitor:

``` text
latency
packet loss
connection errors
DNS failures
HTTP errors
ALB target health
network throughput
NAT Gateway behavior
load-balancer metrics
application connection pools
```

Correlate logs using:

``` text
timestamp
source
destination
port
request ID
status
latency
```

------------------------------------------------------------------------

# 66. Network Troubleshooting Decision Tree

``` text
Application unavailable
        |
        v
Does DNS resolve?
   |            |
  NO           YES
   |            |
Fix DNS      Can TCP connect?
                |       |
               NO      YES
                |       |
        Check route   TLS works?
        SG/NACL          |    |
        firewall         NO   YES
                          |     |
                     Check TLS   HTTP?
                                  |   |
                                 NO  YES
                                  |   |
                           Check app/proxy
```

------------------------------------------------------------------------

# 67. Production Incident Example

Symptom:

``` text
HTTP 504
```

Bad response:

``` text
Restart all Pods.
```

Better response:

``` text
1. Identify affected endpoint.
2. Check DNS.
3. Check ALB.
4. Check target health.
5. Check Service endpoints.
6. Check Pod readiness.
7. Check application latency.
8. Check downstream dependencies.
9. Identify bottleneck.
10. Mitigate.
11. Fix root cause.
```

------------------------------------------------------------------------

# 68. Interview --- What Happens When You Type a URL?

Strong answer:

``` text
1. Browser checks relevant caches.
2. DNS resolution obtains destination information.
3. Client determines a route.
4. TCP connection is established for TCP-based HTTPS.
5. TLS handshake establishes secure communication.
6. HTTP request is sent.
7. Load balancer/proxy routes it.
8. Backend receives it.
9. Application processes it.
10. Response returns.
11. Browser renders the result.
```

Modern browsers may use protocol variants and optimizations, but this is
the fundamental model.

------------------------------------------------------------------------

# 69. Interview --- What Happens When a Pod Calls a Database?

``` text
Pod
 ↓
DNS lookup
 ↓
Database IP
 ↓
Routing
 ↓
Security controls
 ↓
TCP 5432
 ↓
Database listener
 ↓
Authentication
 ↓
Query
```

Troubleshoot in that order.

------------------------------------------------------------------------

# 70. Interview --- Timeout vs Refused

### Refused

Usually active rejection, commonly no listener or explicit rejection.

### Timeout

Expected response did not arrive.

Potential causes:

``` text
routing
firewall
security group
NACL
network path
application delay
```

------------------------------------------------------------------------

# 71. Interview --- ALB vs NLB

### ALB

Layer 7-oriented.

Good for:

``` text
HTTP
HTTPS
host routing
path routing
application-level routing
```

### NLB

Layer 4-oriented.

Good for:

``` text
TCP
UDP
TLS
transport-level workloads
```

Choose based on application requirements.

------------------------------------------------------------------------

# 72. Interview --- Reverse Proxy

A reverse proxy receives requests for backend systems and forwards them
to appropriate upstreams.

It can provide:

``` text
routing
TLS termination
load balancing
header manipulation
connection management
```

Examples:

``` text
Nginx
ALB
Ingress implementations
```

------------------------------------------------------------------------

# 73. Interview --- Default Gateway

A default gateway is the next hop used when the routing table has no
more specific route for a destination.

------------------------------------------------------------------------

# 74. Interview --- Why Is DNS Important?

DNS provides naming and service discovery.

It decouples consumers from fixed IP addresses and is used by:

``` text
applications
databases
cloud services
load balancers
microservices
CI/CD
monitoring
```

------------------------------------------------------------------------

# 75. Interview --- How Do You Troubleshoot Networking?

Strong answer:

> I identify the source, destination, protocol and port, then
> progressively validate DNS, IP addressing, routing, TCP/UDP
> connectivity, security controls, TLS, HTTP and application health. I
> use `dig`, `ip route`, `ss`, `nc`, `curl`, `openssl` and `tcpdump`
> according to the layer being investigated.

------------------------------------------------------------------------

# 76. Interview --- EKS Connectivity Troubleshooting

Identify:

``` text
source
destination
protocol
port
```

Then inspect:

``` text
Pod IP
Node
VPC
route table
security group
NACL
NetworkPolicy
CNI
DNS
load balancer
```

Whenever possible, test from the actual workload.

------------------------------------------------------------------------

# 77. Production Networking Best Practices

Use:

``` text
private subnets for internal workloads
least-privilege security rules
controlled ingress
TLS
centralized DNS
network segmentation
monitoring
logging
documented routing
high availability
tested failure scenarios
```

Typical AWS pattern:

``` text
Public
  ↓
ALB

Private
  ↓
EKS workloads

Private/isolated
  ↓
Database
```

------------------------------------------------------------------------

# 78. Common Networking Mistakes

Avoid:

``` text
ping works → application must work
DNS works → TCP must work
TCP works → HTTP must work
open all ports to 0.0.0.0/0
public databases without justification
ignoring DNS
changing many controls simultaneously
troubleshooting without identifying source/destination
```

------------------------------------------------------------------------

# 79. Practical Lab --- Inspect Interfaces

Run:

``` bash
ip addr
ip link
```

Identify:

``` text
interface
IPv4
IPv6
subnet
link state
```

------------------------------------------------------------------------

# 80. Practical Lab --- Inspect Routes

Run:

``` bash
ip route
ip route get 8.8.8.8
```

Identify:

``` text
interface
source address
gateway
```

------------------------------------------------------------------------

# 81. Practical Lab --- Inspect Listening Ports

Run:

``` bash
sudo ss -lntp
```

Identify:

``` text
process
address
port
protocol
```

------------------------------------------------------------------------

# 82. Practical Lab --- DNS

Run:

``` bash
dig example.com
dig +trace example.com
```

Identify:

``` text
resolver
answer
TTL
authority
```

------------------------------------------------------------------------

# 83. Practical Lab --- HTTP

Run:

``` bash
curl -v https://example.com
```

Identify:

``` text
DNS
TCP
TLS
HTTP
status
headers
```

------------------------------------------------------------------------

# 84. Practical Lab --- TCP Connectivity

Run:

``` bash
nc -vz example.com 443
```

Compare with another port and understand that reachability is
protocol/port specific.

------------------------------------------------------------------------

# 85. Practical Lab --- TLS

Run:

``` bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com
```

Inspect:

``` text
certificate
issuer
validity
TLS version
cipher
```

------------------------------------------------------------------------

# 86. Practical Lab --- Simple HTTP Server

Run:

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

This demonstrates the relationship between:

``` text
process
socket
port
IP
HTTP
```

------------------------------------------------------------------------

# 87. Practical Lab --- 127.0.0.1 vs 0.0.0.0

Bind a server to:

``` text
127.0.0.1
```

and observe that it is local-only.

Then bind to:

``` text
0.0.0.0
```

and understand that the process can accept traffic arriving through
local interfaces, subject to network controls.

------------------------------------------------------------------------

# 88. Practical Lab --- Network Failure

Build a test application and deliberately create:

``` text
wrong port
wrong DNS name
blocked port
wrong route
invalid TLS
```

For each failure, record:

``` text
symptom
command
observed layer
root cause
fix
```

This is much more valuable than memorizing commands.

------------------------------------------------------------------------

# 89. Production Architecture

``` text
                         INTERNET
                            |
                            v
                       Route 53
                            |
                            v
                     AWS ALB :443
                            |
                            v
                    EKS Ingress Layer
                            |
                            v
                     Kubernetes Service
                            |
                     +------+------+
                     |             |
                     v             v
                   Pod-1         Pod-2
                     |             |
                     +------+------+
                            |
                            v
                    Internal Services
                            |
             +--------------+--------------+
             |                             |
             v                             v
           Redis                       Database
```

Security boundaries:

``` text
Internet
 ↓
ALB
 ↓
Security Group
 ↓
EKS
 ↓
NetworkPolicy
 ↓
Application
 ↓
Database boundary
```

------------------------------------------------------------------------

# 90. Final Networking Mental Models

## Model 1 --- Name to Application

``` text
DNS
 ↓
IP
 ↓
Route
 ↓
TCP/UDP
 ↓
TLS
 ↓
HTTP
 ↓
Load Balancer
 ↓
Service
 ↓
Pod
 ↓
Application
```

## Model 2 --- Security

``` text
Identity
 ↓
Network boundary
 ↓
Firewall
 ↓
TLS
 ↓
Authentication
 ↓
Authorization
```

## Model 3 --- Troubleshooting

``` text
Symptom
 ↓
Scope
 ↓
Source
 ↓
Destination
 ↓
DNS
 ↓
Route
 ↓
Port
 ↓
Security
 ↓
Protocol
 ↓
Application
 ↓
Root cause
```

------------------------------------------------------------------------

# 91. Production Checklist

Before considering a production network ready:

``` text
[ ] IP address plan
[ ] CIDR plan
[ ] subnet design
[ ] route tables
[ ] Internet/NAT requirements
[ ] security groups
[ ] NACLs
[ ] DNS
[ ] TLS
[ ] ingress
[ ] load balancing
[ ] network segmentation
[ ] Kubernetes NetworkPolicies
[ ] monitoring
[ ] logging
[ ] network visibility
[ ] high availability
[ ] failure scenarios
[ ] disaster recovery
[ ] documentation
```

------------------------------------------------------------------------