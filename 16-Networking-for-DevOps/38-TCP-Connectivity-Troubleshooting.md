# TCP-Connectivity-Troubleshooting

## 1. Purpose

TCP connectivity is the layer between IP reachability and application protocols such as HTTP.

A production request commonly follows:

```text
DNS
 ↓
IP routing
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
Application
```

When TCP fails, applications can report:

```text
connection timeout
connection refused
connection reset
no route to host
broken pipe
connection closed
```

This file provides a production-oriented methodology for diagnosing TCP connectivity across Linux, AWS, Kubernetes, EKS, load balancers, Nginx, firewalls, NAT and production application environments.

---

## 2. Core TCP Troubleshooting Principle

Always separate:

```text
DNS
IP routing
TCP
TLS
HTTP
```

A successful DNS lookup does not prove TCP connectivity.

A successful TCP connection does not prove TLS works.

A successful TLS handshake does not prove HTTP works.

---

## 3. TCP Connectivity Questions

Before changing anything, establish:

```text
source IP
destination IP
destination port
protocol
timestamp
expected listener
network path
```

---

## 4. Example Request

```text
Pod
 ↓
NAT / VPC
 ↓
Internet
 ↓
api.example.com:443
```

Troubleshoot:

```text
name resolution
route
security controls
TCP handshake
TLS
HTTP
```

---

## 5. TCP Ports

Common ports:

```text
22   SSH
53   DNS/TCP
80   HTTP
443  HTTPS
5432 PostgreSQL
3306 MySQL
6379 Redis
6443 Kubernetes API
8080 common application port
9090 Prometheus
```

Port numbers alone do not prove which application is actually listening.

---

## 6. Listening Socket

Check:

```bash
ss -lntp
```

Example:

```text
LISTEN 0 4096 0.0.0.0:8080
```

---

## 7. TCP Socket States

Important states:

```text
LISTEN
SYN-SENT
SYN-RECV
ESTABLISHED
FIN-WAIT-1
FIN-WAIT-2
CLOSE-WAIT
CLOSING
LAST-ACK
TIME-WAIT
CLOSED
```

---

## 8. LISTEN

A process is waiting for incoming connections.

---

## 9. SYN-SENT

A client has sent a SYN and is waiting for a response.

If many connections remain in:

```text
SYN-SENT
```

investigate:

```text
network path
firewall
destination
packet loss
```

---

## 10. SYN-RECV

A server has received a SYN and sent a SYN-ACK but has not completed the handshake.

Large numbers can indicate:

```text
connection bursts
backlog pressure
packet loss
SYN flood
```

---

## 11. ESTABLISHED

TCP connection is established.

It does not guarantee:

```text
application health
HTTP success
business success
```

---

## 12. TIME-WAIT

A socket remains temporarily after connection closure to protect TCP from delayed packets.

A moderate amount is normal.

---

## 13. CLOSE-WAIT

The remote side closed the connection, but the local application has not fully closed its socket.

A large growing number often indicates an application resource-management problem.

---

## 14. FIN-WAIT

Represents stages of orderly connection shutdown.

Interpret states together with packet captures and application behavior.

---

## 15. Three-Way Handshake

Normal TCP establishment:

```text
Client                  Server

SYN -------------------->

       <---------------- SYN-ACK

ACK -------------------->

       ESTABLISHED
```

---

## 16. SYN

The client requests a TCP connection.

---

## 17. SYN-ACK

The server acknowledges the SYN and sends its own sequence information.

---

## 18. ACK

The client acknowledges the server's SYN.

---

## 19. Successful Handshake

After the final ACK:

```text
TCP connection established
```

The application protocol can begin.

---

## 20. TCP Handshake Troubleshooting

If you see:

```text
SYN
SYN-ACK
ACK
```

TCP establishment succeeded.

Move to:

```text
TLS
application protocol
```

---

## 21. SYN With No Response

Pattern:

```text
SYN
SYN
SYN
...
```

Possible causes:

```text
packet filtering
routing
server unavailable
security group
NACL
firewall
wrong IP
```

---

## 22. SYN Followed by RST

Pattern:

```text
SYN
RST
```

Common interpretation:

```text
destination reachable
but no listener or connection refused
```

Firewall behavior can also produce RSTs.

---

## 23. SYN-ACK Never Reaches Client

Possible:

```text
return path
NACL
firewall
security device
routing
```

---

## 24. Final ACK Never Reaches Server

Possible:

```text
return-path filtering
client-side firewall
packet loss
```

---

## 25. `nc` Connectivity Test

```bash
nc -vz example.com 443
```

This tests TCP connectivity to the destination port.

---

## 26. `telnet`

```bash
telnet example.com 443
```

Useful for basic TCP connectivity, but `nc` and purpose-built tools are usually more convenient.

---

## 27. `curl` TCP Test

```bash
curl -v telnet://example.com:443
```

Use appropriate protocol-specific tests where possible.

---

## 28. `timeout`

```bash
timeout 5 nc -vz 10.0.1.10 8080
```

Useful for bounded diagnostics.

---

## 29. `ss`

List TCP sockets:

```bash
ss -ant
```

---

## 30. Listening TCP Ports

```bash
ss -lnt
```

---

## 31. Process Information

```bash
ss -lntp
```

Requires appropriate privileges to see process information.

---

## 32. Established Connections

```bash
ss -ant state established
```

---

## 33. SYN-SENT Connections

```bash
ss -ant state syn-sent
```

---

## 34. TIME-WAIT Connections

```bash
ss -ant state time-wait
```

---

## 35. Socket Summary

```bash
ss -s
```

Useful during connection storms.

---

## 36. Destination-Specific Sockets

```bash
ss -ant dst 10.0.1.20
```

---

## 37. Source-Specific Sockets

```bash
ss -ant src 10.0.1.10
```

---

## 38. TCP Statistics

```bash
nstat -az
```

Useful for kernel-level TCP counters.

---

## 39. Network Statistics

```bash
ip -s link
```

Look for:

```text
RX errors
TX errors
drops
```

---

## 40. Interface State

```bash
ip link
```

---

## 41. Interface Addresses

```bash
ip addr
```

---

## 42. Routing Table

```bash
ip route
```

---

## 43. Route Lookup

```bash
ip route get 10.0.1.20
```

This helps determine:

```text
interface
source IP
next hop
```

---

## 44. Source Address Selection

```bash
ip route get 10.0.1.20 from 10.0.1.10
```

Useful when multiple interfaces/routes exist.

---

## 45. Default Route

```bash
ip route | grep default
```

---

## 46. Missing Route

If:

```bash
ip route get <destination>
```

fails, investigate routing before TCP.

---

## 47. `traceroute`

```bash
traceroute 10.0.1.20
```

---

## 48. TCP Traceroute

```bash
traceroute -T -p 443 example.com
```

This can be more relevant than ICMP/UDP traceroute when diagnosing a TCP service.

---

## 49. `mtr`

```bash
mtr -rw example.com
```

Use carefully in production and avoid excessive probing.

---

## 50. TCP `mtr`

Some implementations support TCP mode:

```bash
mtr --tcp --port 443 example.com
```

Check the installed version's options.

---

## 51. Packet Capture

Basic:

```bash
tcpdump -ni any host 10.0.1.20
```

---

## 52. Capture TCP Port

```bash
tcpdump -ni any tcp port 443
```

---

## 53. Capture SYN Packets

```bash
tcpdump -ni any 'tcp[tcpflags] & tcp-syn != 0'
```

---

## 54. Capture RST Packets

```bash
tcpdump -ni any 'tcp[tcpflags] & tcp-rst != 0'
```

---

## 55. Capture FIN Packets

```bash
tcpdump -ni any 'tcp[tcpflags] & tcp-fin != 0'
```

---

## 56. Capture Source and Destination

```bash
tcpdump -ni any \
  'host 10.0.1.10 and host 10.0.1.20 and tcp port 443'
```

---

## 57. Capture to File

```bash
tcpdump -ni any -w tcp-debug.pcap tcp port 443
```

Store captures securely because traffic can contain sensitive information.

---

## 58. Read Capture

```bash
tcpdump -nn -r tcp-debug.pcap
```

---

## 59. Wireshark

Wireshark can provide detailed packet analysis.

Use it when:

```text
handshake behavior
retransmissions
windowing
resets
TLS
```

need deeper inspection.

---

## 60. SYN Retransmissions

If the client repeatedly sends:

```text
SYN
SYN
SYN
```

without response, investigate the path.

---

## 61. TCP Retransmissions

Retransmissions can indicate:

```text
packet loss
congestion
receiver behavior
network device issues
```

Do not assume every retransmission means a network outage.

---

## 62. Duplicate ACKs

Duplicate ACKs can indicate missing packets and may lead to retransmission.

---

## 63. Packet Loss

Packet loss can cause:

```text
TCP retransmissions
higher latency
lower throughput
timeouts
```

---

## 64. TCP Reset

RST indicates an aborted/reset connection.

Find:

```text
which side sent RST
```

using packet capture.

---

## 65. Connection Refused

Typical Linux error:

```text
ECONNREFUSED
```

Often means the destination actively rejected the connection, commonly because no process is listening on the target address/port.

---

## 66. Connection Timeout

Typical:

```text
ETIMEDOUT
```

Often indicates no successful response within the timeout.

Investigate:

```text
routing
firewall
packet loss
destination
```

---

## 67. No Route to Host

Typical:

```text
EHOSTUNREACH
ENETUNREACH
```

Investigate:

```text
route
gateway
interface
network policy
```

---

## 68. Broken Pipe

A process may receive:

```text
EPIPE
```

when writing to a closed connection.

Investigate why the peer closed the connection.

---

## 69. Connection Reset by Peer

Common:

```text
ECONNRESET
```

Use packet capture and logs to determine which component reset the connection.

---

## 70. TCP vs UDP

TCP provides:

```text
connection-oriented transport
reliability
ordering
retransmission
flow control
congestion control
```

UDP does not provide these in the same way.

---

## 71. TCP Port Accessibility

Testing:

```bash
nc -vz host 443
```

checks TCP connection establishment.

It does not prove:

```text
TLS
HTTP
application health
```

---

## 72. TCP and TLS

After TCP:

```text
TCP handshake
 ↓
TLS handshake
 ↓
HTTP
```

---

## 73. TLS Failure After TCP Success

If:

```bash
nc -vz host 443
```

works but:

```bash
curl https://host
```

fails,

investigate:

```text
TLS
certificate
SNI
ALPN
```

---

## 74. TCP and HTTP

If:

```text
TCP connects
TLS succeeds
HTTP returns 503
```

TCP is healthy.

Investigate the HTTP/application layer.

---

## 75. Source Port

Client connections use ephemeral source ports.

Example:

```text
10.0.1.10:43122 → 10.0.2.20:443
```

---

## 76. Ephemeral Port Exhaustion

A client can run out of available source ports.

Symptoms:

```text
new outbound connections fail
```

Investigate:

```bash
ss -s
```

and application connection behavior.

---

## 77. NAT Port Exhaustion

Many clients sharing a NAT device can consume large numbers of translated source ports.

AWS NAT Gateway environments should be monitored for connection/port pressure.

---

## 78. Connection Pooling

HTTP clients should generally reuse connections where appropriate.

Without pooling:

```text
many TCP handshakes
many TLS handshakes
high CPU
high latency
```

---

## 79. Keep-Alive

Persistent TCP connections reduce connection establishment overhead.

---

## 80. Idle Connection Timeout

A connection can be closed by:

```text
client
proxy
load balancer
server
firewall
```

after inactivity.

---

## 81. Keep-Alive Mismatch

Example:

```text
LB idle timeout = 60s
backend keepalive = 300s
```

An idle connection may be closed by the LB before the backend expects it.

Tune based on actual architecture.

---

## 82. Load Balancer TCP Flow

For an ALB:

```text
Client
 ↓ TCP
ALB
 ↓ TCP
Target
```

The ALB terminates and creates separate connections.

---

## 83. NLB TCP Flow

A Network Load Balancer can provide Layer 4 forwarding behavior.

Exact source-IP preservation and flow behavior depend on configuration and target type.

---

## 84. NLB Troubleshooting

Check:

```text
listener
target group
target health
security groups
routes
application listener
```

---

## 85. ALB Target Connection

ALB-to-target traffic uses the target group's configured port/protocol behavior.

Verify:

```text
listener
target group
health check
security group
```

---

## 86. Security Group TCP Rule

Example concept:

```text
ALB SG → target SG
TCP 8080
```

The target SG should permit the intended source according to architecture.

---

## 87. Security Group Is Stateful

AWS Security Groups are stateful.

Return traffic for an allowed connection is automatically permitted by the security group connection tracking behavior.

---

## 88. NACL Is Stateless

Network ACLs are stateless.

Return traffic must be explicitly allowed according to the rules.

---

## 89. TCP and NACLs

For a client connecting to:

```text
TCP/443
```

consider:

```text
destination port
ephemeral return port
```

when evaluating stateless filtering.

---

## 90. Kubernetes NetworkPolicy

A Pod can be blocked even if AWS security groups permit traffic.

Check:

```bash
kubectl get networkpolicy -A
```

---

## 91. NetworkPolicy Egress

If Pod cannot connect outward:

```text
egress policy
```

may block the destination.

---

## 92. NetworkPolicy Ingress

If another Pod cannot connect:

```text
ingress policy
```

may block the source.

---

## 93. Kubernetes Service Port

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

Clients connect to:

```text
Service:80
```

while traffic reaches Pods on:

```text
8080
```

---

## 94. Wrong `targetPort`

A Service may exist but point to a port where the application is not listening.

---

## 95. Pod Listener

Inspect:

```bash
kubectl exec <pod> -- ss -lnt
```

if the image contains `ss`.

Otherwise use an appropriate debugging container/tool.

---

## 96. Pod IP Test

From a suitable test Pod:

```bash
nc -vz <pod-ip> 8080
```

---

## 97. Service IP Test

```bash
nc -vz <service-ip> 80
```

---

## 98. Service DNS Test

```bash
nc -vz <service-name> 80
```

DNS and TCP are being tested together, so use `dig` separately when you need to isolate DNS.

---

## 99. EndpointSlice

Check:

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=<service>
```

---

## 100. No Endpoints

If Service has no endpoints:

```text
selector
Pod labels
readiness
```

are primary suspects.

---

## 101. Endpoint Exists But Connection Fails

Check:

```text
Pod listener
targetPort
NetworkPolicy
CNI
node routing
```

---

## 102. kube-proxy

In clusters using kube-proxy, inspect its health/logs when Service routing is suspected.

---

## 103. eBPF Networking

Some Kubernetes distributions use eBPF dataplanes instead of traditional kube-proxy behavior.

Identify the actual dataplane before troubleshooting implementation-specific rules.

---

## 104. EKS VPC CNI

AWS VPC CNI assigns Pod IP addresses from VPC networking resources according to its configuration.

TCP failures can involve:

```text
Pod ENI/IP allocation
routes
security groups
subnets
```

---

## 105. Pod IP Reachability

Check:

```bash
kubectl get pod -o wide
```

Identify:

```text
Pod IP
Node
```

---

## 106. EKS Pod IP

A Pod may receive an address routable in the VPC depending on CNI mode.

Do not assume every Kubernetes network plugin has identical behavior.

---

## 107. Security Groups for Pods

If enabled, Pod traffic can be associated with specific security groups.

This can cause:

```text
Pod-to-service
Pod-to-database
Pod-to-AWS-service
```

connectivity differences.

---

## 108. EKS TCP Path

Example:

```text
Pod
 ↓
VPC route
 ↓
ENI
 ↓
Security Group
 ↓
Destination
```

---

## 109. EC2-to-Pod Troubleshooting

Test:

```bash
nc -vz <pod-ip> <port>
```

from an appropriate VPC host.

Then verify:

```text
route
SG
NACL
Pod listener
```

---

## 110. Pod-to-RDS TCP

Example:

```bash
nc -vz <rds-endpoint> 5432
```

If this fails:

```text
DNS
route
SG
NACL
RDS state
```

should be investigated.

---

## 111. Pod-to-Redis

```bash
nc -vz <redis-endpoint> 6379
```

Then test protocol separately.

---

## 112. TCP Connectivity Does Not Prove Authentication

A successful:

```bash
nc -vz db.example.com 5432
```

only proves TCP connection establishment.

Database credentials can still fail.

---

## 113. TCP to Kubernetes API

Common endpoint:

```text
6443
```

Test:

```bash
nc -vz <api-endpoint> 6443
```

Then continue with TLS/authentication.

---

## 114. EKS Private API Endpoint

For private API access, confirm:

```text
DNS
VPC route
security controls
cluster endpoint access
```

---

## 115. AWS Route Tables

Check:

```bash
aws ec2 describe-route-tables
```

Verify the subnet has the intended route.

---

## 116. AWS Security Groups

Inspect:

```bash
aws ec2 describe-security-groups \
  --group-ids <sg-id>
```

---

## 117. AWS Network Interfaces

```bash
aws ec2 describe-network-interfaces \
  --network-interface-ids <eni-id>
```

Useful when debugging:

```text
ENI
private IP
subnet
security groups
```

---

## 118. VPC Reachability Analyzer

AWS Reachability Analyzer can model network reachability between supported endpoints.

Use it when packet-level reasoning is difficult.

---

## 119. Reachability Analyzer

Useful for identifying possible blocking points involving:

```text
routes
security groups
NACLs
network interfaces
```

---

## 120. TCP Firewall

Host firewall examples:

```bash
iptables
nftables
ufw
firewalld
```

---

## 121. iptables Rules

```bash
iptables -L -n -v
```

---

## 122. nftables

```bash
nft list ruleset
```

---

## 123. firewalld

```bash
firewall-cmd --list-all
```

---

## 124. UFW

```bash
ufw status verbose
```

---

## 125. Firewall Drop

A firewall can silently drop SYN packets, resulting in connection timeouts.

---

## 126. Firewall Reject

A firewall can actively reject connections, producing an immediate failure.

---

## 127. Drop vs Reject

Conceptually:

```text
DROP → no response
REJECT → explicit refusal
```

This often affects whether the client sees timeout vs immediate error.

---

## 128. Host Firewall Debugging

Compare:

```text
client packet
server packet capture
server firewall counters
```

---

## 129. Conntrack

Linux connection tracking:

```bash
conntrack -L
```

requires appropriate privileges/tooling.

---

## 130. Conntrack Table Exhaustion

If conntrack is exhausted, new connections can fail.

Inspect relevant kernel counters and system logs.

---

## 131. Kubernetes Conntrack

High connection churn can create conntrack pressure on nodes.

NodeLocal DNSCache can also help reduce certain DNS-related conntrack pressure patterns.

---

## 132. TCP Backlog

Servers maintain queues for incoming connections.

High traffic can expose:

```text
listen backlog
SYN backlog
application accept rate
```

---

## 133. SYN Flood

Large numbers of SYNs can consume server resources.

Use:

```text
SYN cookies
firewall/WAF/LB protections
rate limiting
```

according to security architecture.

---

## 134. SYN Cookies

Linux can use SYN cookies to mitigate certain SYN-flood conditions.

Check system behavior rather than changing kernel parameters blindly.

---

## 135. Accept Queue

Even after handshake, applications need to accept connections.

A saturated accept queue can cause failures or latency.

---

## 136. Server CPU Saturation

High CPU can reduce the ability to:

```text
process packets
accept connections
serve requests
```

---

## 137. Server Memory Pressure

Memory pressure can lead to:

```text
process kills
connection failures
swapping
latency
```

---

## 138. File Descriptor Limits

TCP sockets consume file descriptors.

Check:

```bash
ulimit -n
```

and process-level limits.

---

## 139. Process Socket Limits

A process can hit its descriptor limit even when the host has available CPU/memory.

---

## 140. TCP Connection Storm

Symptoms:

```text
high SYN rate
high CPU
high connection count
many TIME_WAIT
```

Investigate:

```text
client behavior
retries
load balancer
autoscaling
```

---

## 141. Retry Storm

A failed connection can trigger retries from:

```text
client
proxy
service mesh
application
```

Multiple retry layers can multiply traffic.

---

## 142. Exponential Backoff

Prefer bounded:

```text
backoff
jitter
retry count
```

for transient failures.

---

## 143. TCP Timeout Design

Different layers can have:

```text
connect timeout
read timeout
idle timeout
request timeout
```

Keep them intentional.

---

## 144. Connect Timeout

Controls how long a client waits to establish TCP.

---

## 145. Read Timeout

Controls how long a client waits for response data.

---

## 146. Idle Timeout

Controls how long an established connection can remain idle.

---

## 147. Request Timeout

May cover the complete application request lifecycle.

---

## 148. Timeout Mismatch

Example:

```text
client connect = 5s
proxy = 30s
application = 60s
```

A client may fail before the proxy/application.

---

## 149. TCP Keepalive

TCP keepalive can detect dead idle peers depending on OS and application configuration.

---

## 150. Linux TCP Keepalive

Inspect:

```bash
sysctl net.ipv4.tcp_keepalive_time
sysctl net.ipv4.tcp_keepalive_intvl
sysctl net.ipv4.tcp_keepalive_probes
```

---

## 151. Do Not Tune Blindly

Kernel TCP settings are workload-specific.

Change them only with:

```text
baseline
reason
test
rollback
```

---

## 152. MTU

Maximum Transmission Unit affects packet size.

Common Ethernet MTU:

```text
1500
```

but cloud/VPN/container environments can differ.

---

## 153. MTU Symptoms

Possible:

```text
small packets work
large transfers fail
TLS/HTTP behaves strangely
```

---

## 154. Path MTU

The usable packet size may be lower than interface MTU because of intermediate links/tunnels.

---

## 155. PMTUD

Path MTU Discovery helps endpoints determine appropriate packet sizes.

Firewalls blocking required ICMP can break PMTUD.

---

## 156. TCP MSS

TCP Maximum Segment Size is derived from MTU and headers.

---

## 157. MSS Clamping

Some network devices clamp MSS to avoid fragmentation issues across tunnels.

Use only when required by the network design.

---

## 158. MTU Test

Linux examples may use:

```bash
ping -M do -s 1472 <destination>
```

for IPv4 path testing where supported.

Adjust size based on the expected path and do not assume 1472 universally.

---

## 159. TCP Window

TCP flow control uses receive windows.

A small effective window can limit throughput.

---

## 160. Window Scaling

Modern TCP uses window scaling to support larger receive windows.

---

## 161. Zero Window

A receiver can advertise:

```text
window = 0
```

when it cannot accept more data.

This can cause sender pauses.

---

## 162. TCP Throughput

Throughput depends on:

```text
bandwidth
RTT
window
loss
congestion
```

---

## 163. High RTT

High round-trip time can reduce performance for chatty protocols and connection setup.

---

## 164. Packet Loss + TCP

TCP retransmissions can increase latency and reduce throughput.

---

## 165. Congestion Control

Linux supports TCP congestion-control algorithms.

Inspect:

```bash
sysctl net.ipv4.tcp_congestion_control
```

---

## 166. TCP Performance

Do not tune congestion control before establishing that network performance is the actual bottleneck.

---

## 167. Jumbo Frames

Some private networks support larger MTUs.

All devices on the path must support compatible MTUs.

---

## 168. Jumbo Frame Failure

If:

```text
large packet path fails
```

but:

```text
small packet path works
```

check MTU consistency.

---

## 169. VPN TCP Issues

VPN tunnels add:

```text
encapsulation
overhead
MTU constraints
```

---

## 170. TCP Over TCP

Running a reliable TCP protocol inside another TCP tunnel can produce poor performance under loss due to interacting retransmission mechanisms.

---

## 171. TLS Over TCP

Normal HTTPS:

```text
TCP
 ↓
TLS
 ↓
HTTP
```

---

## 172. TLS Handshake Test

```bash
openssl s_client \
  -connect host:443 \
  -servername host
```

---

## 173. TCP Handshake vs TLS Handshake

TCP:

```text
SYN
SYN-ACK
ACK
```

TLS:

```text
ClientHello
ServerHello
...
```

They are separate.

---

## 174. TCP Success, TLS Failure

Check:

```text
certificate
SNI
supported protocols
cipher suites
ALPN
```

---

## 175. TLS Success, HTTP Failure

Check:

```text
Host
HTTP method
path
application
proxy
```

---

## 176. Proxy TCP Path

```text
Client
 ↓
Proxy
 ↓
Server
```

The client may establish TCP only to the proxy, not directly to the final server.

---

## 177. CONNECT Proxy

HTTPS through an HTTP proxy often uses:

```text
CONNECT host:443
```

The proxy then tunnels traffic.

---

## 178. Proxy Environment

Check:

```bash
env | grep -i proxy
```

---

## 179. NO_PROXY

Inspect:

```bash
env | grep -i no_proxy
```

Incorrect `NO_PROXY` can change TCP path significantly.

---

## 180. Kubernetes Proxy Problem

A Pod may connect through a corporate proxy unexpectedly if environment variables are injected.

---

## 181. TCP Through NAT

NAT changes address/port mappings.

Troubleshoot:

```text
source
translated source
destination
return path
```

---

## 182. Stateful NAT

NAT devices maintain connection state.

If state expires unexpectedly, an established flow may break.

---

## 183. NAT Idle Timeout

Long-lived idle connections can be terminated by NAT devices.

Design keepalives/connection reuse accordingly.

---

## 184. AWS NAT Gateway

AWS NAT Gateway provides outbound internet connectivity for private subnets.

Check:

```text
route
NAT subnet
IGW
security
NAT metrics
```

---

## 185. Private Subnet Route

Typical:

```text
0.0.0.0/0
 →
NAT Gateway
```

---

## 186. Public Subnet Route

Typical:

```text
0.0.0.0/0
 →
Internet Gateway
```

for public resources with public addressing.

---

## 187. TCP to Internet From Private Subnet

Verify:

```text
Pod/EC2
 ↓
route table
 ↓
NAT
 ↓
IGW
 ↓
Internet
```

---

## 188. NAT Failure

If many private workloads lose internet TCP connectivity simultaneously:

```text
NAT Gateway
route
subnet
```

are high-priority checks.

---

## 189. VPC Flow Logs

VPC Flow Logs can help determine whether traffic is:

```text
ACCEPT
REJECT
```

at the VPC networking visibility layer.

They do not replace packet captures.

---

## 190. Flow Log Interpretation

A rejected flow can point toward:

```text
security group
NACL
```

or other VPC networking controls depending on context.

---

## 191. Flow Logs and TCP

Use flow logs with:

```text
source IP
destination IP
source port
destination port
protocol
timestamp
```

to correlate connections.

---

## 192. Kubernetes + VPC Flow Logs

Useful for Pod IP connectivity when Pod addresses are visible in VPC networking.

---

## 193. Security Group Flow Logs

Security-group-related logging capabilities can provide additional visibility where enabled and supported.

---

## 194. Reachability Analyzer Workflow

Model:

```text
source ENI
 →
destination ENI/IP
 →
destination port
```

and inspect the reported path/restrictions.

---

## 195. TCP to RDS

Typical checks:

```text
DNS
route
RDS SG
client SG
NACL
RDS availability
listener
```

---

## 196. RDS Connection Refused

Possible:

```text
wrong endpoint
wrong port
database not accepting connections
security configuration
```

---

## 197. RDS Timeout

More commonly investigate:

```text
route
SG
NACL
network path
```

before database authentication.

---

## 198. TCP to ElastiCache

Check:

```text
VPC
subnet
security group
port
route
cluster state
```

---

## 199. TCP to External API

Check:

```text
DNS
route
NAT
egress security
proxy
remote endpoint
```

---

## 200. TCP to Corporate API

Check:

```text
Route
VPN/DX
Transit Gateway
Resolver
firewall
corporate route
```

DNS and TCP must both be tested.

---

## 201. Hybrid TCP Path

```text
EKS
 ↓
VPC
 ↓
Transit Gateway
 ↓
VPN/DX
 ↓
Corporate firewall
 ↓
Corporate server
```

---

## 202. Asymmetric Routing

Traffic can travel:

```text
outbound path A
return path B
```

Some stateful devices may drop asymmetric flows.

---

## 203. Asymmetric Routing Symptoms

Possible:

```text
SYN leaves
SYN-ACK appears elsewhere
connection never establishes
```

Use packet captures at multiple points.

---

## 204. Route Tables and Return Path

Always check:

```text
forward route
return route
```

not only the source's route.

---

## 205. Multi-Homed Server

A server with multiple interfaces can respond through an unexpected interface.

Check:

```bash
ip route
ip rule
ip addr
```

---

## 206. Policy Routing

Linux can use:

```text
ip rule
```

for source-based or policy routing.

---

## 207. Policy Routing Debugging

```bash
ip rule
ip route show table main
ip route show table all
```

---

## 208. Network Namespace

Containers can have separate network namespaces.

A host's TCP state may not represent the container's exact state.

---

## 209. Container Network Namespace

Use:

```bash
nsenter
```

or Kubernetes debugging mechanisms where authorized.

---

## 210. Ephemeral Port Range

Inspect:

```bash
sysctl net.ipv4.ip_local_port_range
```

---

## 211. Port Exhaustion Diagnosis

If new outbound TCP connections fail:

```text
check source port availability
check connection reuse
check TIME_WAIT
check NAT
```

---

## 212. TIME_WAIT Explosion

Large TIME_WAIT counts can occur with:

```text
short-lived connections
high request rate
lack of keep-alive
```

First fix connection behavior rather than blindly reducing timers.

---

## 213. CLOSE-WAIT Explosion

Usually investigate application socket lifecycle.

---

## 214. SYN-RECV Explosion

Investigate:

```text
traffic spike
SYN flood
backlog
application accept rate
```

---

## 215. SYN-SENT Explosion

Investigate:

```text
unreachable destination
firewall drops
routing
remote service
```

---

## 216. Established Explosion

Could indicate:

```text
legitimate high traffic
connection leak
keep-alive
long-lived streams
```

Use application context.

---

## 217. TCP Retransmission Counters

Use:

```bash
nstat -az | grep -i retrans
```

Exact counter names vary by kernel/version.

---

## 218. Interface Drops

```bash
ip -s link
```

Check:

```text
RX dropped
TX dropped
errors
```

---

## 219. NIC Errors

Hardware/virtual NIC problems can manifest as:

```text
packet loss
retransmissions
```

Investigate host and cloud metrics.

---

## 220. TCP Debugging With `ss -ti`

```bash
ss -ti
```

Can expose TCP-level details such as:

```text
RTT
cwnd
retransmissions
```

availability depends on Linux version.

---

## 221. RTT

Round-trip time measures time for packets to travel between endpoints and return.

---

## 222. Congestion Window

TCP congestion window controls how much unacknowledged data can be in flight.

---

## 223. Retransmission Timeout

TCP uses retransmission mechanisms to recover lost packets.

---

## 224. TCP Slow Start

New connections initially increase sending rate cautiously.

Repeatedly creating short connections can increase overhead.

---

## 225. Connection Reuse

Persistent connections can improve:

```text
latency
CPU
TLS overhead
throughput
```

---

## 226. HTTP Client Pooling

Production clients should configure sensible:

```text
max connections
idle connections
idle timeout
connect timeout
read timeout
```

---

## 227. Database Pooling

The same principle applies to database connections.

---

## 228. Connection Pool Exhaustion

Symptoms:

```text
request latency rises
TCP connections may not increase
application waits
timeouts occur
```

---

## 229. TCP Backpressure

Slow receivers can reduce sender throughput through:

```text
receive window
congestion control
```

---

## 230. Zero Window Debugging

Packet capture can show:

```text
Window Size = 0
```

Investigate receiver application processing.

---

## 231. TCP Window Scaling

Large high-latency links often require window scaling for full throughput.

---

## 232. Bandwidth-Delay Product

The amount of data needed in flight to fully utilize a high-bandwidth path depends on:

```text
bandwidth × RTT
```

---

## 233. TCP Performance Diagnosis

Do not confuse:

```text
low throughput
```

with:

```text
connectivity failure
```

First establish whether the connection succeeds.

---

## 234. Network Troubleshooting Order

Use:

```text
1. interface
2. route
3. destination reachability
4. TCP handshake
5. TLS
6. application
```

---

## 235. TCP Decision Tree

```text
TCP connection fails
       |
       v
Does route exist?
 |             |
NO            YES
 |             |
route        Does SYN get response?
               |             |
              NO            YES
               |             |
          firewall/path    Does final ACK complete?
                            |          |
                           NO         YES
                            |          |
                       return path    TCP OK
```

---

## 236. SYN Capture Decision Tree

```text
Client capture

SYN leaves?
 |
 +-- NO → local application/stack
 |
 +-- YES
      |
      v
SYN-ACK returns?
 |
 +-- NO → path/firewall/destination
 |
 +-- YES
      |
      v
ACK leaves?
 |
 +-- NO → client stack/application
 |
 +-- YES → TCP established
```

---

## 237. Connection Refused Decision Tree

```text
SYN
 ↓
RST
 ↓
ECONNREFUSED
```

Check:

```text
listener
port
destination IP
service
firewall behavior
```

---

## 238. Timeout Decision Tree

```text
SYN
 ↓
no response
 ↓
timeout
```

Check:

```text
route
SG
NACL
firewall
destination
return path
```

---

## 239. Reset Decision Tree

```text
connection
 ↓
RST
```

Find:

```text
sender
timestamp
application state
proxy/LB
firewall
```

---

## 240. TCP Production Runbook

```text
1. Confirm source.
2. Confirm destination.
3. Confirm destination port.
4. Resolve DNS separately.
5. Check source route.
6. Check return route.
7. Test nc.
8. Capture SYN/SYN-ACK.
9. Identify firewall/security control.
10. Check listener.
11. Check application.
12. Continue to TLS/HTTP only after TCP works.
```

---

## 241. Kubernetes TCP Runbook

```text
1. Test Pod → Pod IP.
2. Test Pod → Service IP.
3. Test Pod → Service DNS.
4. Check EndpointSlice.
5. Check Service port/targetPort.
6. Check NetworkPolicy.
7. Check CNI.
8. Check node routing.
9. Check AWS SG/NACL where applicable.
10. Test external path.
```

---

## 242. EKS TCP Runbook

```text
1. Identify Pod IP and node.
2. Check route.
3. Check VPC CNI behavior.
4. Check Security Groups.
5. Check NACL.
6. Check VPC route table.
7. Check NAT/IGW/TGW/VPN/DX as applicable.
8. Check VPC Flow Logs.
9. Use Reachability Analyzer where appropriate.
10. Capture packets when needed.
```

---

## 243. Production Scenario: TCP Timeout

Symptom:

```text
Application cannot connect to RDS.
```

Evidence:

```text
DNS resolves.
SYN leaves Pod.
No SYN-ACK.
```

Next:

```text
check RDS SG
check NACL
check route
check subnet
```

Do not investigate database credentials yet.

---

## 244. Production Scenario: Connection Refused

Symptom:

```text
nc -vz host 8080
→ refused
```

Check:

```bash
ss -lntp | grep 8080
```

If nothing listens:

```text
application not listening
```

---

## 245. Production Scenario: Pod Service Failure

```text
Pod IP: works
Service IP: fails
```

Check:

```text
Service
EndpointSlice
kube-proxy/eBPF dataplane
NetworkPolicy
```

---

## 246. Production Scenario: One Node Fails

```text
Pods on node A → works
Pods on node B → fails
```

Compare:

```text
node route
CNI
iptables/eBPF
security
network interface
```

---

## 247. Production Scenario: One AZ Fails

```text
AZ-a → works
AZ-b → timeout
```

Compare:

```text
subnet route
NACL
NAT
load balancer target
AZ-specific dependency
```

---

## 248. Production Scenario: NAT Problem

Symptoms:

```text
many private workloads
external TCP failures
```

Check:

```text
private route
NAT Gateway
NAT metrics
connection volume
```

---

## 249. Production Scenario: Port Exhaustion

Symptoms:

```text
new outbound connections fail
existing connections work
```

Check:

```text
ephemeral port range
TIME_WAIT
connection pooling
NAT port pressure
```

---

## 250. Production Scenario: Firewall Drop

Capture:

```text
SYN leaves client
no response
```

Firewall logs show:

```text
DROP
```

Root cause:

```text
network security rule
```

---

## 251. Production Scenario: Firewall Reject

Capture:

```text
SYN
RST
```

Firewall logs show:

```text
REJECT
```

Client receives immediate refusal.

---

## 252. Production Scenario: MTU

Symptoms:

```text
TCP connects
small requests work
large responses hang
```

Check:

```text
MTU
MSS
PMTUD
ICMP
VPN/tunnel
```

---

## 253. Production Scenario: Keepalive

Symptoms:

```text
long-lived HTTP connection
idle period
next request fails
```

Check:

```text
LB idle timeout
NAT idle timeout
firewall state timeout
application keepalive
```

---

## 254. Production Scenario: Connection Reset

Symptoms:

```text
intermittent ECONNRESET
```

Capture packets to identify:

```text
client reset
server reset
proxy reset
```

---

## 255. Production Scenario: SYN Flood

Symptoms:

```text
huge SYN-RECV count
connection failures
```

Investigate:

```text
traffic source
LB/WAF protections
SYN cookies
backlog
```

---

## 256. Production Scenario: Application Leak

Symptoms:

```text
CLOSE-WAIT continuously increases
```

Likely:

```text
application does not close sockets correctly
```

Fix application resource handling.

---

## 257. Production Scenario: Connection Churn

Symptoms:

```text
TIME_WAIT extremely high
```

Investigate:

```text
short-lived connections
lack of keepalive
client pool configuration
```

---

## 258. Production Scenario: High Retransmissions

Symptoms:

```text
latency
throughput degradation
TCP retransmissions
```

Check:

```text
packet loss
NIC
network path
congestion
```

---

## 259. Production Scenario: Asymmetric Routing

Symptoms:

```text
SYN reaches server
SYN-ACK leaves server
client never receives it
```

Check:

```text
return route
firewall
routing tables
```

---

## 260. Production Scenario: Multi-Homed Host

A server has:

```text
eth0
eth1
```

but response leaves through the wrong interface.

Check:

```bash
ip rule
ip route
```

---

## 261. Production Scenario: Proxy

Application environment contains:

```text
HTTPS_PROXY
```

TCP traffic unexpectedly goes through corporate proxy.

Check:

```bash
env | grep -i proxy
```

---

## 262. Production Scenario: NO_PROXY

Internal service works directly but fails when proxy is used.

Check:

```text
NO_PROXY
```

and internal service domains.

---

## 263. Production Scenario: TLS Failure

```text
nc → success
curl → TLS error
```

TCP is healthy.

Move to:

```text
certificate
SNI
TLS versions
ALPN
```

---

## 264. Production Scenario: HTTP Failure

```text
nc → success
openssl → success
curl → 503
```

TCP and TLS are healthy.

Investigate:

```text
load balancer
Ingress
application
```

---

## 265. TCP Observability

Monitor:

```text
connection count
SYN rate
RST rate
retransmissions
RTT
packet loss
TIME_WAIT
CLOSE_WAIT
```

---

## 266. Linux TCP Metrics

Useful commands:

```bash
ss -s
nstat -az
ip -s link
```

---

## 267. TCP Dashboard

A production dashboard can show:

```text
active connections
new connections/sec
resets/sec
retransmissions
latency
connection errors
```

---

## 268. Load Balancer Metrics

For AWS load balancers, monitor relevant metrics for:

```text
active connections
new connections
target response time
5xx
rejected connections
```

Exact metric names depend on the AWS service.

---

## 269. Network Flow Logs

Use VPC Flow Logs to correlate:

```text
source
destination
port
protocol
accept/reject
```

---

## 270. TCP Incident Timeline

Record:

```text
first failure
peak failure
deployment
configuration change
traffic change
recovery
```

---

## 271. Correlation

Correlate:

```text
TCP failures
application logs
load balancer logs
CloudWatch metrics
VPC Flow Logs
deployment events
```

---

## 272. TCP Packet Capture Safety

Captures can contain:

```text
IP addresses
ports
payload data
```

Store and share according to security policy.

---

## 273. Minimize Packet Capture

Prefer narrow filters:

```bash
tcpdump -ni any host <ip> and tcp port <port>
```

instead of capturing all traffic.

---

## 274. Capture During Incident

Capture enough traffic to establish:

```text
SYN
SYN-ACK
ACK
RST
FIN
retransmission
```

---

## 275. Do Not Capture Indefinitely

Long captures create:

```text
large files
performance overhead
security exposure
```

Use bounded capture windows.

---

## 276. TCP Debugging Evidence

Good evidence:

```text
client sent SYN
server sent no SYN-ACK
```

Better than:

```text
"network seems broken"
```

---

## 277. Root Cause Statement

Use:

```text
The application timed out connecting to RDS because the target
security group did not allow TCP/5432 from the application's security
group.
```

rather than:

```text
RDS was down.
```

---

## 278. Change Management

Before changing:

```text
SG
NACL
route
firewall
kernel TCP parameters
```

record current state.

---

## 279. Safe TCP Changes

Prefer:

```text
small scope
controlled test
rollback
monitoring
```

---

## 280. Avoid Blind Kernel Tuning

Do not immediately change:

```text
tcp_tw_reuse
tcp_fin_timeout
somaxconn
tcp_max_syn_backlog
```

without evidence.

---

## 281. Why Blind Tuning Is Dangerous

Changing kernel networking parameters can:

```text
hide application bugs
change connection behavior
create new failure modes
```

---

## 282. Application First

If:

```text
CLOSE-WAIT
```

is growing, investigate application socket handling before changing kernel timers.

---

## 283. Network First

If:

```text
SYN
→ no response
```

investigate network path/security before application HTTP code.

---

## 284. TCP Interview Question: How Does TCP Handshake Work?

Answer:

```text
The client sends SYN, the server replies with SYN-ACK, and the client
sends ACK. After this three-way handshake the TCP connection is
established and the application protocol can begin.
```

---

## 285. Interview: Connection Refused vs Timeout?

Answer:

```text
Connection refused usually means the destination actively rejected
the connection, commonly because no listener exists. A timeout often
means the connection attempt received no successful response, so I
investigate routing, firewalls, security groups, NACLs and packet loss.
```

---

## 286. Interview: How Do You Debug TCP?

Answer:

```text
I identify source, destination and port, verify routing, test the
port with nc, and capture the handshake if needed. I determine
whether SYN, SYN-ACK and ACK are present. Only after TCP succeeds do I
move to TLS and HTTP.
```

---

## 287. Interview: How Do You Identify a Firewall Drop?

Answer:

```text
I capture traffic at the source and destination where possible and
check firewall, Security Group, NACL and VPC Flow Log evidence. A
SYN leaving without a response strongly suggests a path or filtering
issue, but I verify with logs rather than assuming.
```

---

## 288. Interview: What Does SYN-SENT Mean?

Answer:

```text
The local host has sent a SYN and is waiting for the peer's response.
A large number of SYN-SENT sockets can indicate unreachable
destinations, filtering, packet loss or a remote service problem.
```

---

## 289. Interview: What Does CLOSE-WAIT Mean?

Answer:

```text
The remote peer has closed its side, but the local application has
not closed its socket. A growing CLOSE-WAIT count often indicates an
application resource-management issue.
```

---

## 290. Interview: What Does TIME-WAIT Mean?

Answer:

```text
TIME-WAIT is part of normal TCP connection cleanup. It helps prevent
delayed packets from an old connection from interfering with a new
connection. A high count can be normal during heavy short-lived
connection workloads.
```

---

## 291. Interview: How Do You Troubleshoot TCP to RDS?

Answer:

```text
I verify DNS, then test TCP/5432 from the actual workload. If the
connection times out, I inspect route tables, security groups, NACLs
and network paths. If TCP connects, I move to database protocol and
authentication rather than continuing network troubleshooting.
```

---

## 292. Interview: TCP Connects but HTTP Fails?

Answer:

```text
TCP only proves transport connectivity. I next test TLS if HTTPS is
used, then inspect HTTP status, Host header, proxy/load-balancer
routing and application behavior.
```

---

## 293. Interview: How Do You Debug EKS Service Connectivity?

Answer:

```text
I test Pod IP, Service IP and Service DNS separately. I inspect
Service selectors, EndpointSlices, targetPort, NetworkPolicies and
the cluster networking dataplane. In EKS I also consider VPC CNI,
security groups and routes.
```

---

## 294. Interview: How Do You Debug NAT Connectivity?

Answer:

```text
I verify the private subnet route to the NAT Gateway, NAT placement,
internet path and security controls. For high-volume failures I also
inspect NAT metrics and connection/port pressure.
```

---

## 295. Interview: What Causes TCP Retransmissions?

Answer:

```text
Packet loss, congestion, receiver behavior and network problems can
cause retransmissions. I use packet captures, interface counters and
TCP statistics to determine whether retransmissions are actually
abnormal for the workload.
```

---

## 296. Interview: What Is MTU?

Answer:

```text
MTU is the maximum IP packet size that can be transmitted over an
interface without fragmentation at that layer. VPNs and tunnels can
reduce effective path MTU, causing large-packet failures if PMTUD is
not working correctly.
```

---

## 297. Interview: What Is Asymmetric Routing?

Answer:

```text
It occurs when traffic travels through different paths in each
direction. Stateful firewalls and network devices can reject the
return traffic because they do not see the expected connection state.
```

---

## 298. Interview: Why Are Connection Pools Important?

Answer:

```text
Connection pooling reduces repeated TCP and TLS handshakes and lowers
connection churn. Poor pool sizing can cause either excessive
connections or request waiting, so I monitor pool utilization,
timeouts and downstream capacity.
```

---

## 299. Interview: How Do You Troubleshoot Intermittent Connection Resets?

Answer:

```text
I correlate reset events by timestamp, source, destination, target,
AZ and Pod. Packet capture tells me which endpoint sent the RST. I
then inspect application restarts, load balancer behavior, proxy
timeouts, keepalive settings and network devices.
```

---

## 300. Interview: What Is the Difference Between TCP and HTTP?

Answer:

```text
TCP is a transport-layer protocol that provides reliable ordered byte
delivery. HTTP is an application-layer protocol carried over TCP in
HTTP/1.1 and HTTP/2 deployments, while HTTP/3 uses QUIC over UDP.
```

---

## 301. Interview: Why Can `nc` Work While `curl` Fails?

Answer:

```text
nc tests TCP connectivity only. curl also performs TLS and HTTP
operations. If nc succeeds and curl fails, I investigate TLS,
certificate/SNI, HTTP routing, authentication or application behavior.
```

---

## 302. Interview: Why Can TCP Work From Node but Not Pod?

Answer:

```text
The Pod may have a different network namespace, route, NetworkPolicy,
proxy configuration or CNI path. I test from the actual Pod and
compare the Pod networking state with the node.
```

---

## 303. Interview: What Is a SYN Flood?

Answer:

```text
A SYN flood sends large numbers of connection initiation packets,
consuming server or network resources. Mitigation can include
load-balancer protections, SYN cookies, filtering and rate limiting
according to the architecture.
```

---

## 304. Interview: Should You Reduce TIME_WAIT?

Answer:

```text
Not as a first response. TIME_WAIT is normal TCP behavior. I first
investigate connection churn and connection pooling. Kernel tuning
should be evidence-driven and tested carefully.
```

---

## 305. Interview: How Do You Troubleshoot a 3-Way Handshake?

Answer:

```text
I capture traffic and look for SYN, SYN-ACK and ACK. If SYN leaves
without SYN-ACK, I investigate destination/path filtering. If
SYN-ACK returns but ACK does not complete, I investigate the return
path or client stack. If all three exist, TCP is established and I
move upward.
```

---

## 306. Senior Scenario: Production RDS Timeout

Architecture:

```text
EKS Pod
 ↓
VPC
 ↓
RDS
```

Evidence:

```text
DNS → correct
SYN → leaves
SYN-ACK → absent
```

Conclusion:

```text
TCP path/filtering problem.
```

Investigate:

```text
RDS Security Group
NACL
route table
subnet
VPC path
```

---

## 307. Senior Scenario: TCP Works, API Times Out

Evidence:

```text
nc → success
TLS → success
HTTP → 504
```

Conclusion:

```text
TCP is healthy.
```

Investigate:

```text
application
database
external dependency
proxy timeout
```

---

## 308. Senior Scenario: Only New Connections Fail

Existing TCP sessions work.

Investigate:

```text
ephemeral ports
SYN backlog
conntrack
NAT port pressure
connection limits
```

---

## 309. Senior Scenario: Existing Connections Drop After Idle

Investigate:

```text
NAT idle timeout
LB idle timeout
firewall state timeout
keepalive
```

---

## 310. Senior Scenario: Large Responses Fail

Small responses work.

Investigate:

```text
MTU
MSS
PMTUD
fragmentation
firewall ICMP filtering
```

---

## 311. Senior Scenario: CLOSE-WAIT Growth

Investigate:

```text
application socket lifecycle
```

not simply:

```text
network
```

---

## 312. Senior Scenario: TIME-WAIT Growth

Investigate:

```text
connection churn
short-lived HTTP
connection pooling
client behavior
```

---

## 313. Senior Scenario: High SYN-RECV

Investigate:

```text
traffic burst
SYN flood
backlog
server capacity
```

---

## 314. Senior Scenario: High SYN-SENT

Investigate:

```text
remote reachability
firewall
route
destination availability
```

---

## 315. Senior Scenario: One Target Fails

If:

```text
ALB
 ↓
Target A → works
Target B → timeout
```

compare:

```text
target B listener
SG
node
Pod
route
health
```

---

## 316. Senior Scenario: One AZ Fails

Compare:

```text
subnet
route
NACL
NAT
target
```

between AZs.

---

## 317. Senior Scenario: Corporate Network Failure

Path:

```text
EKS
 ↓
TGW
 ↓
VPN
 ↓
Firewall
 ↓
Corporate server
```

Use:

```text
packet capture
flow logs
route inspection
firewall logs
```

---

## 318. Senior Scenario: Wrong Source IP

Application expects:

```text
10.10.10.0/24
```

but sees:

```text
different NAT address
```

Investigate:

```text
NAT
proxy
load balancer
source preservation
```

---

## 319. Senior Scenario: Security Group Looks Correct

Even with correct SGs, TCP can fail because of:

```text
NACL
route
NetworkPolicy
host firewall
wrong target port
listener
```

Always troubleshoot the complete path.

---

## 320. Senior Scenario: NACL Looks Correct

Even with correct NACLs, investigate:

```text
route
SG
NetworkPolicy
application listener
return path
```

---

## 321. Senior Scenario: Route Looks Correct

A correct route does not prove:

```text
security
listener
application
return route
```

Continue through the layers.

---

## 322. TCP Production Checklist

```text
[ ] Source identified
[ ] Destination identified
[ ] Destination port identified
[ ] DNS checked separately
[ ] Source route checked
[ ] Return route checked
[ ] Listener verified
[ ] nc connectivity tested
[ ] SYN captured if needed
[ ] SYN-ACK verified
[ ] ACK verified
[ ] Firewall checked
[ ] Security Group checked
[ ] NACL checked
[ ] NetworkPolicy checked
[ ] CNI checked
[ ] NAT checked
[ ] Flow Logs checked
[ ] Retransmissions checked
[ ] MTU checked where relevant
[ ] Connection states checked
[ ] TLS tested after TCP
[ ] HTTP tested after TLS
```

---

## 323. Final TCP Command Cheat Sheet

```bash
# Sockets
ss -ant
ss -lnt
ss -lntp
ss -s
ss -ti
ss -ant state established
ss -ant state syn-sent
ss -ant state time-wait

# Connectivity
nc -vz <host> <port>
timeout 5 nc -vz <host> <port>

# Routing
ip addr
ip link
ip route
ip route get <destination>
ip rule

# Network statistics
ip -s link
nstat -az

# Tracing
traceroute -T -p 443 <host>
mtr -rw <host>

# Packet capture
tcpdump -ni any tcp port 443
tcpdump -ni any 'tcp[tcpflags] & tcp-syn != 0'
tcpdump -ni any 'tcp[tcpflags] & tcp-rst != 0'

# TLS after TCP
openssl s_client -connect <host>:443 -servername <host>

# Kubernetes
kubectl get svc
kubectl get endpointslice
kubectl get networkpolicy -A
kubectl get pods -o wide

# AWS
aws ec2 describe-route-tables
aws ec2 describe-security-groups --group-ids <sg>
aws ec2 describe-network-interfaces --network-interface-ids <eni>
aws elbv2 describe-target-health --target-group-arn <arn>
```

---

## 324. Final TCP Production Principles

```text
1. Identify source, destination and port.
2. Separate DNS from TCP.
3. Verify the route.
4. Verify the return route.
5. Verify the listener.
6. Test with nc.
7. Capture the handshake when necessary.
8. SYN without response means investigate the path.
9. SYN followed by RST means investigate refusal/listener/policy.
10. Complete handshake means TCP is established.
11. Move to TLS after TCP.
12. Move to HTTP after TLS.
13. Use ss for socket-state analysis.
14. Use tcpdump for packet evidence.
15. Use VPC Flow Logs for cloud-path evidence.
16. Use Reachability Analyzer where appropriate.
17. Check SG and NACL separately.
18. Check Kubernetes NetworkPolicy separately.
19. Check CNI for Pod networking.
20. Check NAT for private-subnet egress.
21. Investigate ephemeral port exhaustion.
22. Investigate conntrack under high connection churn.
23. Treat TIME_WAIT as normal unless workload evidence says otherwise.
24. Treat growing CLOSE-WAIT as an application concern.
25. Check MTU for large-packet failures.
26. Check asymmetric routing when handshake responses disappear.
27. Avoid blind kernel tuning.
28. Avoid blind retry increases.
29. Preserve packet/log evidence.
30. State root cause in terms of the exact failed layer.
```

---