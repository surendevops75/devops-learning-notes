# TCP-Three-Way-Handshake

## 1. Purpose

The TCP three-way handshake is the foundation of TCP connection establishment.

A DevOps engineer should understand it at packet level because many production incidents appear as:

```text
connection timeout
connection refused
SYN-SENT
SYN-RECV
high latency
connection backlog pressure
SYN flood
load balancer failures
intermittent application connectivity
```

This file goes deeper than the previous TCP overview and focuses on:

- TCP connection establishment
- SYN
- SYN-ACK
- ACK
- sequence numbers
- acknowledgement numbers
- Initial Sequence Numbers
- TCP flags
- TCP states
- retransmissions
- SYN backlog
- SYN cookies
- packet captures
- Linux troubleshooting
- AWS/EKS scenarios
- production incidents
- connection termination
- RST
- interview preparation

---

## 2. TCP Connection Establishment

A TCP connection is normally established using:

```text
SYN
SYN-ACK
ACK
```

Conceptually:

```text
Client                                      Server

  |                                           |
  | ----------- SYN ------------------------> |
  |                                           |
  | <-------- SYN + ACK --------------------- |
  |                                           |
  | ----------- ACK ------------------------> |
  |                                           |
  |              ESTABLISHED                  |
```

---

## 3. Why Three Messages?

TCP needs both endpoints to establish state and synchronize sequence information.

The handshake allows:

```text
Client → Server:
"I want to establish a connection."

Server → Client:
"I received your request and I also want the connection."

Client → Server:
"I received your response."
```

After this, both sides can transition to:

```text
ESTABLISHED
```

---

## 4. TCP Is Full Duplex

A TCP connection supports independent data flow in both directions.

Conceptually:

```text
Client  ==================>  Server
        <==================
```

The handshake establishes state for both directions.

---

## 5. TCP Header

A TCP segment contains fields including:

```text
source port
destination port
sequence number
acknowledgement number
data offset
flags
window
checksum
urgent pointer
options
payload
```

The handshake mainly depends on:

```text
source port
destination port
sequence number
acknowledgement number
flags
window
options
```

---

## 6. Source Port

The client normally selects an ephemeral source port.

Example:

```text
10.0.1.10:51544
```

---

## 7. Destination Port

The destination port identifies the server-side service.

Example:

```text
10.0.2.20:443
```

The initial connection therefore looks like:

```text
10.0.1.10:51544
        ↓
10.0.2.20:443
```

---

## 8. SYN Flag

SYN means synchronization.

The client sends a SYN segment to initiate a TCP connection.

Example conceptually:

```text
Flags = SYN
```

The SYN carries an Initial Sequence Number.

---

## 9. Initial Sequence Number

The Initial Sequence Number is commonly called:

```text
ISN
```

The client chooses an ISN for its sequence number space.

Example:

```text
Client ISN = 1000
```

Real operating systems use unpredictable sequence-number generation rather than simple fixed values.

---

## 10. SYN Consumes Sequence Space

A critical TCP rule:

```text
SYN consumes one sequence number.
```

Suppose:

```text
Client sends SYN
Seq = 1000
```

The next sequence number after the SYN is conceptually:

```text
1001
```

This is why the server's acknowledgement is:

```text
ACK = 1001
```

---

## 11. Server SYN

The server also selects its own Initial Sequence Number.

Example:

```text
Server ISN = 5000
```

The server sends:

```text
SYN
ACK
Seq = 5000
Ack = 1001
```

---

## 12. Client Final ACK

The client receives:

```text
Seq = 5000
Ack = 1001
```

It responds:

```text
ACK
Seq = 1001
Ack = 5001
```

The client is now acknowledging the server's SYN.

---

## 13. Complete Example

Assume:

```text
Client ISN = 1000
Server ISN = 5000
```

Handshake:

```text
Client → Server
SYN
Seq=1000

Server → Client
SYN-ACK
Seq=5000
Ack=1001

Client → Server
ACK
Seq=1001
Ack=5001
```

Both endpoints can now enter:

```text
ESTABLISHED
```

---

## 14. Sequence Number Meaning

The sequence number identifies the position of transmitted bytes in the TCP byte stream.

It is not simply:

```text
packet number
```

It represents a position in the TCP sequence space.

---

## 15. Acknowledgement Number Meaning

The acknowledgement number generally means:

```text
the next sequence number expected
```

Example:

```text
received bytes through 1999
next expected = 2000
ACK = 2000
```

---

## 16. SYN and ACK Relationship

If:

```text
Client ISN = X
```

then server acknowledges:

```text
X + 1
```

If:

```text
Server ISN = Y
```

then client acknowledges:

```text
Y + 1
```

because each SYN consumes one sequence number.

---

## 17. TCP Flags

Important flags include:

```text
SYN
ACK
FIN
RST
PSH
URG
ECE
CWR
```

Modern TCP may also use additional ECN-related signaling.

---

## 18. SYN-ACK

The second handshake packet contains:

```text
SYN = 1
ACK = 1
```

It simultaneously:

```text
acknowledges the client's SYN
and
synchronizes the server's sequence space
```

---

## 19. Final ACK

The third packet normally contains:

```text
ACK = 1
```

It acknowledges the server's SYN.

After this, normal application data can flow.

---

## 20. TCP State: Client

Typical client-side state transition:

```text
CLOSED
  |
  | connect()
  v
SYN-SENT
  |
  | SYN-ACK received
  v
ESTABLISHED
```

---

## 21. TCP State: Server

Typical passive-open sequence:

```text
CLOSED
  |
  | bind()
  | listen()
  v
LISTEN
  |
  | SYN received
  v
SYN-RECEIVED
  |
  | final ACK
  v
ESTABLISHED
```

---

## 22. `listen()`

A server application normally performs operations conceptually equivalent to:

```text
socket()
bind()
listen()
accept()
```

The kernel manages TCP connection state while the application accepts established connections.

---

## 23. `connect()`

A client commonly invokes:

```text
connect()
```

The kernel creates and manages the TCP handshake.

The application normally does not manually construct SYN packets.

---

## 24. Kernel Responsibility

The Linux kernel handles:

```text
TCP state machine
sequence numbers
ACK processing
retransmission
socket queues
connection tracking
```

Applications interact through sockets.

---

## 25. SYN Transmission

When the client initiates:

```text
connect()
```

the kernel sends:

```text
SYN
```

The client enters:

```text
SYN-SENT
```

while waiting for the server response.

---

## 26. SYN-ACK Reception

If the server receives the SYN and accepts the connection attempt:

```text
SYN-ACK
```

is returned.

The client validates the response.

---

## 27. Final ACK

The client sends:

```text
ACK
```

The connection can now transition to:

```text
ESTABLISHED
```

---

## 28. What Happens if SYN-ACK Is Lost?

The client does not immediately conclude the server is dead.

TCP retransmission mechanisms cause the SYN to be retransmitted according to the operating system's TCP behavior.

Conceptually:

```text
SYN
  |
  X lost
  |
SYN retransmission
  |
  v
SYN-ACK
```

---

## 29. What Happens if SYN Is Lost?

The server never sees the original SYN.

The client retransmits.

Conceptually:

```text
Client                     Server

SYN --------X

SYN --------X

SYN -------->

             SYN-ACK
```

The exact retry timing depends on OS configuration.

---

## 30. What Happens if Final ACK Is Lost?

The server may remain in:

```text
SYN-RECEIVED
```

until it receives the final ACK or otherwise times out according to TCP behavior.

The client may already believe the connection is established.

This can create interesting packet-capture patterns during incidents.

---

## 31. Duplicate SYN

A retransmitted SYN can appear as a duplicate if the original response was lost.

TCP is designed to handle retransmissions and duplicate segments using sequence numbers and connection state.

---

## 32. SYN Retransmission

Check retransmissions with:

```bash
sudo tcpdump -ni any 'tcp[tcpflags] & tcp-syn != 0'
```

This filter identifies packets carrying SYN.

---

## 33. SYN-ACK Filter

Example:

```bash
sudo tcpdump -ni any \
'tcp[tcpflags] & (tcp-syn|tcp-ack) == (tcp-syn|tcp-ack)'
```

This helps identify SYN-ACK traffic.

---

## 34. ACK Filter

Example:

```bash
sudo tcpdump -ni any \
'tcp[tcpflags] & tcp-ack != 0'
```

This is broad because many established TCP packets also carry ACK.

Do not interpret every ACK as handshake traffic.

---

## 35. Handshake Packet Capture

Example:

```bash
sudo tcpdump -ni any \
host 10.0.2.20 and port 443
```

Expected sequence:

```text
[S]
[S.]
[.]
```

where:

```text
[S]  = SYN
[S.] = SYN + ACK
[.]  = ACK
```

---

## 36. Wireshark View

Wireshark can decode:

```text
SYN
SYN-ACK
ACK
sequence numbers
acknowledgements
TCP options
retransmissions
RTT
window size
```

Use packet captures to verify what actually happened instead of assuming the application behavior.

---

## 37. TCP Options

TCP options may be negotiated during the handshake.

Important options include:

```text
MSS
Window Scale
SACK Permitted
Timestamps
```

The exact options depend on endpoints and network conditions.

---

## 38. MSS Negotiation

The Maximum Segment Size indicates the largest TCP payload the endpoint is prepared to receive in a segment.

Example:

```text
MSS = 1460
```

is common for an IPv4 Ethernet path with:

```text
MTU = 1500
```

but actual environments can differ.

---

## 39. Window Scale

The TCP window can be scaled using the Window Scale option.

This is important for high-bandwidth/high-latency networks where the default TCP window would otherwise limit throughput.

---

## 40. SACK Permitted

SACK means:

```text
Selective Acknowledgement
```

It allows the receiver to communicate information about non-contiguous blocks of received data.

This improves recovery from packet loss.

---

## 41. TCP Timestamps

TCP timestamps can support:

```text
RTT measurement
PAWS-related protection
```

depending on implementation and configuration.

---

## 42. TCP Handshake and RTT

Handshake timing provides a basic network-latency signal.

Conceptually:

```text
SYN sent
   |
   | RTT
   v
SYN-ACK received
```

A slow SYN-ACK can indicate:

```text
network latency
server processing delay
middlebox delay
packet loss
```

It does not automatically identify the root cause.

---

## 43. TCP Connect Latency

Application metrics often report:

```text
DNS time
TCP connect time
TLS handshake time
TTFB
total request time
```

Separating these phases helps identify where latency originates.

---

## 44. `curl` Timing

Use:

```bash
curl -s -o /dev/null \
-w 'dns=%{time_namelookup}\nconnect=%{time_connect}\ntls=%{time_appconnect}\nttfb=%{time_starttransfer}\ntotal=%{time_total}\n' \
https://example.com
```

This separates important timing phases.

---

## 45. TCP Handshake vs TLS Handshake

For HTTPS:

```text
TCP:
SYN
SYN-ACK
ACK

TLS:
ClientHello
ServerHello
...
```

Therefore a TCP handshake can succeed while TLS fails.

---

## 46. TCP Handshake vs HTTP

Full flow:

```text
DNS
 ↓
TCP handshake
 ↓
TLS handshake
 ↓
HTTP request
 ↓
HTTP response
```

A DevOps engineer should troubleshoot each stage independently.

---

## 47. Connection Refused During Handshake

If the server responds with RST after SYN:

```text
SYN
<-- RST
```

the connection is actively rejected.

Possible causes:

```text
no listener
local kernel behavior
firewall
load balancer
application
```

---

## 48. Connection Timeout During Handshake

If:

```text
SYN
SYN
SYN
...
```

with no SYN-ACK, investigate:

```text
routing
security group
NACL
firewall
listener
destination availability
return path
packet loss
```

---

## 49. SYN-SENT Troubleshooting

Check:

```bash
ss -tan state syn-sent
```

If many connections are stuck in SYN-SENT:

```text
client is sending connection attempts
but is not receiving successful SYN-ACK responses
```

Investigate the network path and destination.

---

## 50. SYN-RECV Troubleshooting

Check:

```bash
ss -tan state syn-recv
```

A high number may indicate:

```text
many incoming connection attempts
slow application acceptance
packet loss
SYN flood
backlog pressure
```

Interpret according to workload baseline.

---

## 51. Listening Socket Backlog

A listening server needs queues for pending connection attempts.

Linux has several layers of queueing and kernel/application limits.

Useful checks include:

```bash
ss -lnt
sysctl net.core.somaxconn
```

Application-specific settings also matter.

---

## 52. `somaxconn`

Inspect:

```bash
sysctl net.core.somaxconn
```

It defines a kernel-level ceiling relevant to listen backlog behavior.

Changing it blindly is not a complete fix for connection problems.

---

## 53. Application Backlog

An application can request a listen backlog.

Conceptually:

```c
listen(fd, backlog);
```

The effective behavior depends on:

```text
application
kernel
socket implementation
```

and relevant limits.

---

## 54. SYN Backlog

The SYN backlog is associated with incomplete connection establishment.

Conceptually:

```text
SYN received
   ↓
SYN-ACK sent
   ↓
waiting for final ACK
```

This is different from established-connection queues.

---

## 55. SYN Flood

A SYN flood attempts to consume resources associated with incomplete connections.

Example:

```text
many spoofed/rapid SYNs
        |
        v
SYN-RECV pressure
        |
        v
resource exhaustion
```

---

## 56. SYN Cookies

SYN cookies allow a server to avoid storing full state for every SYN before the final ACK arrives.

The server encodes state into the SYN-ACK sequence information.

When the ACK arrives, the server can validate it and reconstruct necessary state.

---

## 57. SYN Cookies Are Not a Universal Solution

SYN cookies help against certain SYN-flood scenarios.

They do not solve:

```text
bad routing
wrong port
security groups
application crashes
DNS failure
network congestion
```

Use layered defenses.

---

## 58. Linux SYN Cookie Setting

Inspect:

```bash
sysctl net.ipv4.tcp_syncookies
```

A value of:

```text
1
```

commonly means SYN cookies are enabled when appropriate.

Exact kernel behavior should be checked for the deployed Linux version.

---

## 59. SYN Flood Protection

Production defenses can include:

```text
SYN cookies
load balancers
firewalls
rate limiting
DDoS protection
AWS Shield where applicable
network ACL strategy
capacity planning
```

---

## 60. AWS Load Balancer Handshake

For an internet-facing HTTPS ALB:

```text
Client
  |
  | TCP SYN
  v
ALB
  |
  | SYN-ACK
  v
Client
  |
  | ACK
  v
TCP established
```

TLS then occurs at the configured TLS termination point.

---

## 61. ALB TLS Termination

Typical architecture:

```text
Internet
   |
TCP 443
   |
ALB
   |
TLS termination
   |
HTTP or HTTPS
   |
Target
```

The backend protocol depends on configuration.

---

## 62. NLB TCP Flow

A TCP-oriented NLB can proxy/forward TCP traffic depending on listener and target configuration.

Conceptually:

```text
Client
 |
TCP
 |
NLB
 |
TCP
 |
Target
```

This differs from an ALB's HTTP-aware behavior.

---

## 63. EKS Ingress Handshake

For an ALB-backed Kubernetes Ingress:

```text
Client
 |
SYN
 v
ALB
 |
TCP/TLS
 v
Target
 |
Service
 |
Pod
```

There can therefore be multiple TCP connections depending on the load-balancing mode and protocol configuration.

---

## 64. Why Multiple Handshakes Matter

A client-to-ALB handshake succeeding does not automatically prove:

```text
ALB → target
```

is healthy.

A load balancer can accept a client connection while its backend target is unhealthy.

---

## 65. ALB Target Health

If clients can connect to:

```text
ALB:443
```

but requests fail, inspect:

```text
target health
health checks
security groups
Service
Pod readiness
targetPort
application listener
```

---

## 66. Security Group Handshake Failure

If the client sends:

```text
SYN
```

and the packet is blocked before reaching the listener, the client may experience a timeout.

Security groups are stateful, but the allowed source, destination, protocol and port still matter.

---

## 67. NACL Handshake Failure

NACLs are stateless.

A TCP handshake requires bidirectional traffic.

Therefore both relevant directions must be allowed.

A return-path NACL mistake can create:

```text
SYN sent
SYN-ACK blocked
timeout
```

---

## 68. Kubernetes NetworkPolicy Handshake Failure

If NetworkPolicy blocks traffic:

```text
SYN
   |
blocked
```

the client may experience a timeout depending on the networking implementation.

Check:

```bash
kubectl get networkpolicy -A
```

and inspect the applicable policies.

---

## 69. Service Selector Failure

A Service may have no backend endpoints.

Check:

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=<service>
```

If no endpoints exist:

```text
Service routing cannot reach an application Pod
```

even if the Pod itself is running.

---

## 70. targetPort Failure

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

If the application actually listens on:

```text
9090
```

backend connectivity fails.

Verify:

```bash
kubectl exec -it <pod> -- ss -lntp
```

---

## 71. Readiness Failure

A Pod can have:

```text
STATUS = Running
```

while not being ready.

Then Kubernetes may remove it from normal Service endpoints.

Check:

```bash
kubectl get pods
kubectl describe pod <pod>
```

---

## 72. TCP Probe

Example:

```yaml
readinessProbe:
  tcpSocket:
    port: 8080
  periodSeconds: 10
  timeoutSeconds: 2
  failureThreshold: 3
```

This tests TCP-level port availability.

It does not prove application correctness.

---

## 73. HTTP Probe

Example:

```yaml
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  periodSeconds: 10
```

This provides a higher-level health signal.

---

## 74. Startup Probe

For applications requiring initialization:

```yaml
startupProbe:
  tcpSocket:
    port: 8080
  periodSeconds: 10
  failureThreshold: 30
```

Startup probes can prevent liveness checks from killing a process before it finishes starting.

---

## 75. TCP Handshake and Connection Pooling

Every new TCP connection requires handshake overhead.

Without pooling:

```text
request
 ↓
SYN
 ↓
SYN-ACK
 ↓
ACK
 ↓
request
```

With persistent connections:

```text
connection
   |
   +-- request
   +-- request
   +-- request
```

Pooling can significantly reduce connection overhead.

---

## 76. Connection Storm

A connection storm occurs when many clients create connections simultaneously.

Examples:

```text
application restart
Kubernetes rollout
autoscaling
database failover
cache failure
load balancer recovery
```

Potential effects:

```text
SYN pressure
CPU spikes
connection queue pressure
database connection exhaustion
ephemeral-port pressure
```

---

## 77. Kubernetes Rolling Deployment Storm

Suppose:

```text
500 Pods restart
```

and each Pod immediately opens:

```text
100 connections
```

Potentially:

```text
50,000 connection attempts
```

can occur over a short period.

Connection pooling, startup staggering and dependency capacity planning become important.

---

## 78. Database Connection Storm

A common production incident:

```text
EKS deployment
   |
many Pods restart
   |
each creates large DB pool
   |
database max connections exceeded
```

The TCP handshake can succeed while database connection admission fails.

This illustrates why:

```text
TCP connectivity != application capacity
```

---

## 79. Backoff

Clients should use controlled retry behavior.

Good retry strategies can include:

```text
exponential backoff
jitter
maximum retry count
circuit breakers
connection pooling
```

Avoid synchronized retry storms.

---

## 80. SYN Retransmission vs Application Retry

These are different layers.

TCP:

```text
retransmits packets
```

Application:

```text
retries requests
```

A single application request can therefore experience transport retransmissions plus application-level retries.

---

## 81. Failure Amplification

Example:

```text
TCP packet loss
   |
connection latency
   |
application timeout
   |
application retry
   |
more TCP connections
   |
more congestion
```

This feedback loop can amplify an outage.

---

## 82. TCP Handshake and Timeouts

Several timeout layers may exist:

```text
TCP SYN retry timeout
application connect timeout
TLS timeout
HTTP request timeout
load balancer timeout
database timeout
```

Misaligned timeout values can create confusing behavior.

---

## 83. Timeout Design

Production systems should intentionally define:

```text
DNS timeout
TCP connect timeout
TLS timeout
request timeout
idle timeout
retry timeout
```

Do not use extremely long defaults without understanding the failure impact.

---

## 84. TCP Handshake and NAT

With NAT:

```text
Client private IP:port
        |
        v
NAT translation
        |
        v
Public IP:translated-port
        |
        v
Server
```

The NAT device tracks the flow.

A failure in NAT state or capacity can prevent new connections.

---

## 85. NAT Gateway Connection Pressure

For high-scale workloads:

```text
many Pods
 ↓
many outbound connections
 ↓
NAT Gateway
 ↓
many destination flows
```

Monitor:

```text
connection count
port allocation
NAT errors
flow patterns
```

and consider VPC endpoints or architecture changes for AWS services where appropriate.

---

## 86. TCP Handshake and DNS

The handshake cannot begin until the client has a destination IP.

Typical sequence:

```text
DNS query
 ↓
IP address
 ↓
TCP SYN
```

Therefore:

```text
DNS failure
```

can appear to users as:

```text
application connectivity failure
```

even though TCP itself is healthy.

---

## 87. `dig` Before TCP Testing

Use:

```bash
dig app.example.com
```

Then:

```bash
nc -vz app.example.com 443
```

Then:

```bash
curl -vk https://app.example.com
```

This layered sequence is highly effective.

---

## 88. TCP Handshake and IPv6

A hostname can resolve to:

```text
A
AAAA
```

The client may attempt IPv6 and IPv4 according to operating-system and application behavior.

A broken IPv6 path can cause unexpected connection delays or failures.

Check:

```bash
dig A example.com
dig AAAA example.com
```

---

## 89. Dual-Stack Troubleshooting

Compare:

```bash
curl -4 -v https://example.com
curl -6 -v https://example.com
```

If IPv4 works and IPv6 fails:

```text
investigate IPv6 routing
security
listener
DNS
network configuration
```

---

## 90. Packet Capture Best Practice

Capture as close as possible to the failing endpoint.

Example:

```bash
sudo tcpdump -ni any \
'host 10.0.2.20 and tcp port 443'
```

Avoid enormous unrestricted captures during production incidents.

Use:

```text
host
port
interface
packet count
```

filters.

---

## 91. Capture to File

```bash
sudo tcpdump -ni any \
-w /tmp/tcp-handshake.pcap \
host 10.0.2.20 and port 443
```

Then inspect with Wireshark or `tshark` where available.

---

## 92. Capture Only SYN Packets

```bash
sudo tcpdump -ni any \
'tcp[tcpflags] & tcp-syn != 0'
```

This is useful for identifying:

```text
connection attempts
SYN retransmissions
unexpected clients
SYN floods
```

---

## 93. Capture RST Packets

```bash
sudo tcpdump -ni any \
'tcp[tcpflags] & tcp-rst != 0'
```

Investigate:

```text
who sent the RST
what connection it belonged to
what happened immediately before it
```

---

## 94. Capture FIN Packets

```bash
sudo tcpdump -ni any \
'tcp[tcpflags] & tcp-fin != 0'
```

This helps investigate graceful connection termination.

---

## 95. `ss` With Process Information

```bash
sudo ss -tanp
```

This can show:

```text
state
local address
peer address
process
```

It is often the fastest first tool for socket-state incidents.

---

## 96. `ss` Memory Information

On Linux, `ss` can expose additional socket memory information:

```bash
ss -m
```

Use this when investigating socket-buffer pressure.

---

## 97. TCP Timers

`ss` can expose timer-related information:

```bash
ss -o state established
```

This can help identify sockets with active timers.

Exact output varies by Linux version.

---

## 98. TCP Statistics

Useful:

```bash
nstat -az
```

It can expose kernel network statistics.

Examples of interesting counters can include:

```text
retransmissions
resets
listen overflows
failed connections
```

Interpret counters against workload and baseline.

---

## 99. `/proc/net/snmp`

Linux exposes protocol statistics through:

```bash
cat /proc/net/snmp
```

This can provide TCP/IP counters for deeper kernel-level investigation.

---

## 100. TCP Retransmission Monitoring

Useful signals:

```text
TCP retransmits
packet loss
RTT
connection failures
```

Prometheus node exporters and eBPF-based observability tools can expose additional network telemetry depending on deployment.

---

## 101. Production Handshake Incident: No SYN-ACK

### Symptom

```text
curl hangs
nc times out
```

### Capture

```text
SYN
SYN
SYN
```

No:

```text
SYN-ACK
```

### Investigation

```text
route
security group
NACL
NetworkPolicy
listener
load balancer
destination
return path
```

---

## 102. Production Handshake Incident: RST

### Symptom

```text
connection refused
```

Capture:

```text
SYN
RST
```

### Investigation

```text
listener
Service
targetPort
application
load balancer
firewall
```

---

## 103. Production Handshake Incident: SYN-RECV Explosion

### Symptom

```bash
ss -tan state syn-recv | wc -l
```

shows a very large count.

Investigate:

```text
traffic spike
SYN flood
backlog
CPU
packet loss
load balancer
application
```

---

## 104. Production Handshake Incident: Client SYN-SENT Explosion

### Symptom

Many:

```text
SYN-SENT
```

connections.

Likely direction:

```text
clients are attempting connections
but successful SYN-ACK responses are not arriving
```

Investigate:

```text
destination
routing
firewall
NACL
security group
network path
```

---

## 105. Production Handshake Incident: ALB Healthy, Backend Unhealthy

### Symptom

```text
TCP 443 to ALB succeeds
```

but requests return:

```text
5xx
```

Investigate:

```text
ALB target health
target group
Service
EndpointSlice
Pod readiness
targetPort
Pod listener
application
```

---

## 106. Production Handshake Incident: EKS NetworkPolicy

### Symptom

A service worked before a NetworkPolicy change.

Now:

```text
connection timeout
```

Check:

```bash
kubectl get networkpolicy -A
```

Verify that:

```text
source
destination
namespace
port
protocol
```

are allowed.

---

## 107. Production Handshake Incident: NACL

### Symptom

Connection attempts time out after subnet changes.

Check:

```text
subnet NACL
inbound rules
outbound rules
ephemeral return ports
```

Because NACLs are stateless, return traffic must be explicitly permitted.

---

## 108. Production Handshake Incident: Application Binds Localhost

### Symptom

Pod is Ready or process is running, but Service traffic fails.

Check:

```bash
kubectl exec -it <pod> -- ss -lntp
```

If application listens on:

```text
127.0.0.1:8080
```

remote Pod traffic cannot normally reach it.

Bind to the appropriate Pod interface/address, commonly:

```text
0.0.0.0:8080
```

when appropriate.

---

## 109. Production Handshake Incident: Port Mismatch

Example:

```text
Service targetPort = 8080
application listener = 8081
```

TCP handshake to the intended backend fails.

Correct the configuration or application listener.

---

## 110. Production Handshake Incident: Slow TLS After Fast TCP

Metrics:

```text
TCP connect = 5 ms
TLS = 2 seconds
```

This indicates the TCP handshake is not the primary delay.

Investigate:

```text
TLS certificate
CPU
crypto
TLS version
network middleboxes
server processing
```

---

## 111. Production Handshake Incident: Fast TCP, Slow HTTP

Metrics:

```text
connect = 5 ms
TLS = 20 ms
TTFB = 5 seconds
```

TCP is probably healthy.

Investigate:

```text
application processing
database
Redis
RabbitMQ
thread pools
CPU
locks
downstream services
```

---

## 112. Production Handshake Incident: Database Connection Storm

### Symptom

Application rollout causes:

```text
database max connections
```

### Root cause

Every new Pod creates an oversized connection pool.

### Fix

Use:

```text
bounded pool
connection reuse
startup staggering
backoff
proper pool sizing
database capacity planning
```

---

## 113. Production Handshake Incident: TIME-WAIT Growth

Large:

```text
TIME-WAIT
```

after a traffic spike.

Investigate:

```text
active closer
short-lived HTTP connections
connection reuse
proxy behavior
NAT
load balancer
```

Do not immediately disable TCP safety behavior.

---

## 114. Production Handshake Incident: CLOSE-WAIT Growth

Large:

```text
CLOSE-WAIT
```

with stable or increasing trend.

Investigate the application for sockets that are not being closed after peer termination.

---

## 115. Production Handshake Incident: Ephemeral Port Exhaustion

Symptoms:

```text
connect failures
EADDRNOTAVAIL
```

Check:

```bash
cat /proc/sys/net/ipv4/ip_local_port_range
ss -s
```

Then investigate:

```text
connection churn
pooling
destination concentration
NAT
source addresses
```

---

## 116. Production Handshake Incident: MTU

### Symptom

```text
TCP connects
small requests work
large transfers stall
```

Investigate:

```text
MTU
MSS
PMTUD
VPN
encapsulation
firewall
```

Packet capture can reveal retransmissions and missing acknowledgements.

---

## 117. TCP Termination Overview

TCP termination normally uses:

```text
FIN
ACK
FIN
ACK
```

It is separate from the three-way establishment handshake.

---

## 118. Four-Way Close

Typical:

```text
Client                    Server

FIN -------------------->

      <---------------- ACK

      <---------------- FIN

ACK -------------------->
```

The four messages arise because each direction of the full-duplex byte stream closes independently.

---

## 119. FIN Sequence Number

Like SYN:

```text
FIN consumes one sequence number.
```

This is important when interpreting packet captures.

---

## 120. FIN-WAIT-1

The active closer sends FIN and enters:

```text
FIN-WAIT-1
```

It waits for acknowledgement and completion of the peer's close.

---

## 121. FIN-WAIT-2

After its FIN is acknowledged:

```text
FIN-WAIT-2
```

The endpoint waits for the peer's FIN.

---

## 122. CLOSE-WAIT

The peer's FIN has arrived.

The local kernel acknowledges it.

The application is expected to close its side.

---

## 123. LAST-ACK

After the application sends its FIN, it can enter:

```text
LAST-ACK
```

while waiting for the final ACK.

---

## 124. TIME-WAIT

The endpoint that actively closes can enter:

```text
TIME-WAIT
```

for a period determined by TCP implementation behavior.

It protects against delayed duplicate segments and helps ensure reliable termination.

---

## 125. RST During Establishment

A RST can reject a connection attempt.

Example:

```text
SYN
<-- RST
```

This is different from:

```text
SYN
<-- SYN-ACK
```

which indicates normal establishment progress.

---

## 126. RST During Established Connection

A connection can be reset after establishment because of:

```text
application close behavior
invalid state
kernel behavior
middlebox
load balancer
peer failure
```

Capture both sides where possible.

---

## 127. TCP Half-Close

TCP supports independent closing of each direction.

One side can stop sending:

```text
FIN
```

while still receiving data.

This is why TCP termination is not simply:

```text
one packet closes everything
```

---

## 128. TCP Handshake Security

The handshake itself is not encryption.

A TCP connection can be established without TLS.

For secure application communication:

```text
TCP
+
TLS
```

is commonly required.

---

## 129. TCP Does Not Authenticate Applications

TCP establishes transport connectivity.

It does not prove:

```text
user identity
application identity
authorization
```

Those belong to higher-level mechanisms such as:

```text
TLS certificates
mTLS
tokens
authentication
authorization
```

---

## 130. TCP Spoofing Considerations

IP/TCP security mechanisms and network controls must account for spoofing.

Modern operating systems use robust sequence-number generation, while network security systems provide additional filtering.

---

## 131. SYN Flood vs Application DDoS

A SYN flood targets:

```text
connection establishment
```

An application-layer attack may establish valid TCP connections and then overload:

```text
HTTP
database
API
business logic
```

Defenses must operate at multiple layers.

---

## 132. AWS DDoS Protection

AWS environments can use layered controls such as:

```text
AWS Shield
AWS WAF
CloudFront
load balancers
security groups
rate limiting
application controls
```

The correct architecture depends on the exposed workload.

---

## 133. Handshake Monitoring

Useful production metrics:

```text
TCP connection attempts
connection failures
SYN retransmissions
TCP retransmissions
RTT
active connections
SYN-RECV
SYN-SENT
TIME-WAIT
CLOSE-WAIT
```

Alert on abnormal trends, not isolated normal events.

---

## 134. Observability Architecture

Example:

```text
Application
   |
metrics/logs
   |
Prometheus / Grafana
   |
TCP metrics

Node
   |
network statistics
   |
node exporter / eBPF tooling

Packet path
   |
tcpdump / Wireshark
```

Combine metrics with packet evidence.

---

## 135. Useful Linux Commands

```bash
ss -s
ss -lntp
ss -tan
ss -tan state syn-sent
ss -tan state syn-recv
ss -tan state established
ss -tan state time-wait
ss -tan state close-wait

ip route get <destination>

nstat -az

sysctl net.core.somaxconn
sysctl net.ipv4.tcp_syncookies
sysctl net.ipv4.ip_local_port_range

sudo tcpdump -ni any host <ip> and port <port>
```

---

## 136. Kubernetes Commands

```bash
kubectl get svc
kubectl describe svc <service>
kubectl get endpointslice
kubectl get pods -o wide
kubectl describe pod <pod>
kubectl get networkpolicy -A
kubectl get events --sort-by=.lastTimestamp
kubectl exec -it <pod> -- ss -lntp
```

---

## 137. AWS/EKS Investigation Sequence

For an external TCP/HTTPS failure:

```text
Route 53
 ↓
ALB/NLB
 ↓
Security Group
 ↓
NACL
 ↓
Target health
 ↓
Ingress
 ↓
Service
 ↓
EndpointSlice
 ↓
NetworkPolicy
 ↓
Pod
 ↓
Process listener
```

---

## 138. Packet-Level Investigation Sequence

```text
1. Capture source.
2. Capture destination if possible.
3. Find SYN.
4. Check SYN-ACK.
5. Check final ACK.
6. Check retransmissions.
7. Check RST.
8. Check TCP options.
9. Check application payload.
10. Compare timestamps.
```

---

## 139. First Question During a Handshake Failure

Ask:

```text
Did the SYN leave the client?
```

If not:

```text
application/socket/kernel issue
```

If yes:

```text
inspect the path
```

---

## 140. Second Question

Ask:

```text
Did the SYN reach the server?
```

If unknown:

```text
capture closer to server
```

---

## 141. Third Question

Ask:

```text
Did the server send SYN-ACK?
```

If no:

```text
listener
firewall
kernel
backlog
destination
```

If yes:

```text
inspect return path
```

---

## 142. Fourth Question

Ask:

```text
Did the client send final ACK?
```

If no:

```text
client-side path
state
packet loss
```

If yes:

```text
TCP establishment completed
```

Move upward to:

```text
TLS
HTTP
application
```

---

## 143. Production Debugging Rule

Do not say:

```text
"Network is down."
```

Instead state:

```text
"SYN leaves the client, but no SYN-ACK returns."
```

This is actionable.

---

## 144. Production Debugging Rule

Do not say:

```text
"Port 443 is broken."
```

Instead state:

```text
"TCP handshake succeeds, but TLS negotiation fails after ClientHello."
```

This identifies the failing layer.

---

## 145. Production Debugging Rule

Do not say:

```text
"Kubernetes Service is broken."
```

Instead determine:

```text
Service has no EndpointSlices
```

or:

```text
Endpoint exists but targetPort does not match listener
```

or:

```text
NetworkPolicy denies ingress
```

---

## 146. TCP Handshake Decision Tree

```text
Application request
        |
        v
DNS resolves?
   |           |
  No          Yes
   |           |
DNS issue      v
           SYN sent?
             |
         +---+---+
         |       |
        No      Yes
         |       |
      socket     v
       issue   SYN-ACK?
                 |
             +---+---+
             |       |
            No      Yes
             |       |
        path/filter   v
                  final ACK?
                     |
                 +---+---+
                 |       |
                No      Yes
                 |       |
             return     TCP
               path     established
                         |
                         v
                        TLS
                         |
                         v
                        HTTP
```

---

## 147. Interview: Explain the Three-Way Handshake

A strong answer:

```text
The client sends SYN with its Initial Sequence Number.
The server responds with SYN-ACK, acknowledging the client's SYN
and advertising its own Initial Sequence Number.
The client sends the final ACK acknowledging the server's SYN.
Both sides then enter ESTABLISHED.
```

---

## 148. Interview: Why Does SYN Consume a Sequence Number?

SYN occupies one position in the TCP sequence space so that the connection can reliably synchronize sequence numbering.

---

## 149. Interview: Why Does SYN-ACK Acknowledge ISN+1?

Because the SYN consumes one sequence number.

---

## 150. Interview: What Is ISN?

Initial Sequence Number. It is the starting sequence value selected for a TCP endpoint's sequence space.

---

## 151. Interview: What Happens If SYN-ACK Is Lost?

The client retransmits the SYN according to TCP retransmission behavior.

The server may retransmit its SYN-ACK after receiving the duplicate SYN.

---

## 152. Interview: What Happens If the Final ACK Is Lost?

The server can remain in SYN-RECEIVED and retransmit SYN-ACK according to its TCP behavior while waiting for the handshake to complete.

---

## 153. Interview: SYN-SENT Means What?

The local endpoint has initiated the connection and is waiting for the server's response.

---

## 154. Interview: SYN-RECV Means What?

The server has received the client's SYN, sent a SYN-ACK, and is waiting for the handshake to complete.

---

## 155. Interview: What Is a SYN Flood?

An attack or traffic pattern involving excessive SYN attempts that can consume resources associated with incomplete TCP connections.

---

## 156. Interview: What Are SYN Cookies?

A defensive mechanism that allows a server to avoid storing full connection state for every SYN before receiving the final ACK.

---

## 157. Interview: Are SYN Cookies a Complete DDoS Solution?

No. They address a class of SYN-flood resource pressure but do not replace layered DDoS, firewall, load-balancing and application protections.

---

## 158. Interview: What Is a TCP Backlog?

It represents queueing capacity associated with incoming connection handling. Linux has distinct mechanisms for incomplete and completed connection queues.

---

## 159. Interview: How Do You Detect SYN Flood Symptoms?

Look at:

```bash
ss -tan state syn-recv
```

and packet captures, system counters, load-balancer telemetry and traffic patterns.

---

## 160. Interview: How Do You Capture a TCP Handshake?

```bash
sudo tcpdump -ni any host <ip> and port <port>
```

Look for:

```text
SYN
SYN-ACK
ACK
```

---

## 161. Interview: What Does `[S.]` Mean in tcpdump?

It commonly represents:

```text
SYN + ACK
```

---

## 162. Interview: What Does `[R.]` Mean?

It commonly indicates:

```text
RST + ACK
```

depending on the packet.

---

## 163. Interview: How Do You Debug SYN-SENT?

Check:

```text
destination IP
route
security controls
firewall
NACL
listener
packet capture
return path
```

---

## 164. Interview: How Do You Debug SYN-RECV?

Check:

```text
incoming traffic volume
SYN flood
backlog
server CPU
packet loss
client final ACK
```

---

## 165. Interview: TCP Handshake vs TLS Handshake?

TCP establishes the transport connection.

TLS negotiates encryption and authentication above TCP.

---

## 166. Interview: Can TCP Work While HTTPS Fails?

Yes.

TCP can succeed while:

```text
TLS
certificate validation
SNI
HTTP
application
```

fails.

---

## 167. Interview: Can a Load Balancer Accept TCP but Backend Be Down?

Yes.

The frontend connection and backend connection/health are separate concerns.

---

## 168. Interview: Why Can ALB 443 Work but Application Fail?

Because:

```text
client → ALB
```

can succeed while:

```text
ALB → target
```

fails.

---

## 169. Interview: How Does `targetPort` Affect Handshake?

The Service forwards traffic toward the configured backend port. If no process listens there, backend TCP establishment can fail.

---

## 170. Interview: What Is the Difference Between Connection Refused and Timeout?

```text
Refused:
active rejection

Timeout:
no successful response before timeout
```

---

## 171. Interview: Why Is a Packet Capture Better Than `curl` Alone?

`curl` tells you the application-level result.

A packet capture can show:

```text
SYN
SYN-ACK
ACK
retransmission
RST
timing
```

which identifies where the transport exchange fails.

---

## 172. Interview: Why Can a TCP Handshake Be Slow?

Potential causes:

```text
latency
packet loss
SYN retransmission
server load
middlebox processing
routing
congestion
```

---

## 173. Interview: What Is MSS?

The maximum TCP payload size advertised for a connection.

It is related to the path MTU and helps avoid oversized TCP segments.

---

## 174. Interview: What Is Window Scaling?

A TCP option that allows larger effective receive windows, important for high-bandwidth/high-latency paths.

---

## 175. Interview: What Is SACK?

Selective Acknowledgement allows a TCP receiver to describe received non-contiguous data blocks, improving loss recovery.

---

## 176. Interview: Why Is TCP a Byte Stream?

TCP presents an ordered stream of bytes and does not preserve application write boundaries.

---

## 177. Interview: What Is Half-Close?

One direction of a TCP connection can be closed with FIN while the other direction remains open.

---

## 178. Interview: Why Does TCP Have Four-Way Close?

Because TCP is full duplex and each direction can be closed independently.

---

## 179. Interview: What Is TIME-WAIT Used For?

It helps protect against delayed duplicate segments and ensures correct TCP connection termination.

---

## 180. Interview: What Is CLOSE-WAIT?

The remote peer has closed its side, but the local application has not closed its socket.

---

## 181. Interview: What Is RST?

A TCP reset abruptly rejects or terminates a connection.

---

## 182. Interview: What Happens Before TCP Handshake if Using a Hostname?

Normally:

```text
DNS resolution
```

occurs first to obtain the destination address.

---

## 183. Interview: Why Can IPv6 Cause Connection Problems?

A hostname may resolve to AAAA and A records. If IPv6 connectivity is broken, clients may experience failures or delays depending on connection selection behavior.

---

## 184. Interview: How Do You Compare IPv4 and IPv6?

```bash
curl -4 -v https://example.com
curl -6 -v https://example.com
```

---

## 185. Interview: How Do You Find Listening Ports in Linux?

```bash
ss -lntp
```

---

## 186. Interview: How Do You Find TCP State Counts?

```bash
ss -tan | awk 'NR>1 {print $1}' | sort | uniq -c
```

---

## 187. Interview: How Do You Check TCP Kernel Statistics?

```bash
nstat -az
```

---

## 188. Interview: How Do You Check SYN Cookies?

```bash
sysctl net.ipv4.tcp_syncookies
```

---

## 189. Interview: How Do You Check the Listen Backlog Limit?

```bash
sysctl net.core.somaxconn
```

Also inspect application-specific socket configuration.

---

## 190. Interview: What Is the Correct Troubleshooting Order?

A strong production answer:

```text
DNS
→ route
→ TCP handshake
→ TLS
→ HTTP
→ application
→ dependencies
```

For Kubernetes additionally inspect:

```text
Service
→ EndpointSlice
→ readiness
→ NetworkPolicy
→ Pod listener
```

---

## 191. Interview: What Would You Say in a Production Incident?

Avoid:

```text
"The network is broken."
```

Say:

```text
"The client sends SYN packets, but no SYN-ACK returns.
I am checking the route, security controls, destination listener
and return path."
```

This demonstrates layer-by-layer debugging.

---

## 192. Production Checklist

```text
[ ] Understand SYN
[ ] Understand SYN-ACK
[ ] Understand final ACK
[ ] Understand ISN
[ ] Understand sequence numbers
[ ] Understand acknowledgement numbers
[ ] Understand TCP flags
[ ] Understand TCP state transitions
[ ] Understand SYN-SENT
[ ] Understand SYN-RECV
[ ] Understand ESTABLISHED
[ ] Understand SYN retransmission
[ ] Understand TCP options
[ ] Understand MSS
[ ] Understand window scaling
[ ] Understand SACK
[ ] Understand SYN backlog
[ ] Understand SYN cookies
[ ] Understand SYN floods
[ ] Know tcpdump
[ ] Know Wireshark
[ ] Know ss
[ ] Know nstat
[ ] Understand ALB handshake path
[ ] Understand EKS Service path
[ ] Understand NetworkPolicy
[ ] Understand NACL behavior
[ ] Understand security groups
[ ] Understand connection storms
[ ] Understand TCP timeouts
[ ] Understand MTU/MSS problems
[ ] Understand TCP termination
[ ] Understand RST
[ ] Understand TIME-WAIT
[ ] Understand CLOSE-WAIT
```

---

## 193. Final Mental Model

When debugging TCP, visualize:

```text
                    TCP CONNECTION

Client                                      Server
  |                                           |
  |  SYN, Seq=X                              |
  |------------------------------------------>|
  |                                           |
  |  SYN-ACK, Seq=Y, Ack=X+1                 |
  |<------------------------------------------|
  |                                           |
  |  ACK, Seq=X+1, Ack=Y+1                   |
  |------------------------------------------>|
  |                                           |
  |              ESTABLISHED                 |
  |                                           |
  |=========== Application Data =============|
  |                                           |
```

Remember:

```text
SYN consumes one sequence number.
FIN consumes one sequence number.
ACK means "next sequence number expected."
```

---

## 194. Final Production Debugging Model

When an application cannot connect:

```text
1. Resolve the destination.
2. Identify source and destination IPs.
3. Identify source and destination ports.
4. Check route.
5. Check whether SYN leaves.
6. Check whether SYN-ACK returns.
7. Check final ACK.
8. Check retransmissions/RST.
9. Check TLS.
10. Check HTTP/application.
11. Check Kubernetes Service and EndpointSlice.
12. Check AWS load balancer and security controls.
13. Check downstream dependencies.
```

This is the transport-layer mindset expected from a production DevOps engineer.

---