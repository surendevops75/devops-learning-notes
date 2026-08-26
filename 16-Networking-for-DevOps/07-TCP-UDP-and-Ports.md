# 16-Networking-for-DevOps
# 07-TCP-UDP-and-Ports

## 1. Purpose

TCP, UDP and ports are core transport-layer concepts for DevOps engineers.

Production systems depend on them for:

- HTTP/HTTPS
- SSH
- databases
- Redis
- RabbitMQ
- DNS
- Kubernetes Services
- AWS load balancers
- EKS workloads
- monitoring
- logging
- service-to-service communication

When an application cannot communicate, a DevOps engineer must determine whether the problem is:

```text
DNS
 ↓
IP routing
 ↓
TCP/UDP
 ↓
Port
 ↓
Listener
 ↓
TLS
 ↓
Application
```

This file focuses on TCP, UDP, ports, sockets and production troubleshooting.

---

## 2. Transport Layer

The transport layer provides end-to-end communication between application processes.

The two most important Internet transport protocols are:

```text
TCP
UDP
```

The transport layer uses:

```text
IP address
+
port
```

to identify communication endpoints.

---

## 3. TCP

TCP means:

```text
Transmission Control Protocol
```

TCP is:

```text
connection-oriented
reliable
ordered
byte-stream based
```

It provides mechanisms for:

- acknowledgements
- retransmission
- ordering
- flow control
- congestion control
- connection establishment
- connection termination

---

## 4. UDP

UDP means:

```text
User Datagram Protocol
```

UDP is:

```text
connectionless
datagram-oriented
low-overhead
```

UDP does not provide TCP's built-in:

- reliable delivery
- ordering
- retransmission
- congestion control
- connection establishment

Applications can implement reliability above UDP when required.

---

## 5. TCP vs UDP

| Feature | TCP | UDP |
|---|---|---|
| Connection establishment | Yes | No |
| Reliability | Yes | No |
| Ordering | Yes | No |
| Retransmission | Yes | No |
| Flow control | Yes | No |
| Congestion control | Yes | No |
| Data model | Byte stream | Datagram |
| Typical overhead | Higher | Lower |
| Common examples | HTTPS, SSH, databases | DNS, DHCP, NTP, QUIC |

---

## 6. What Is a Port?

A port identifies a transport-layer endpoint on a host.

IPv4 and IPv6 addresses identify network endpoints, while the port identifies the application endpoint at the transport layer.

Example:

```text
10.20.10.50:443
```

means:

```text
IP address = 10.20.10.50
port       = 443
```

The protocol also matters:

```text
TCP 443
```

is distinct from:

```text
UDP 443
```

---

## 7. Port Range

TCP and UDP ports range from:

```text
0–65535
```

TCP and UDP have separate port spaces.

Therefore a host can use:

```text
TCP 53
UDP 53
```

simultaneously.

---

## 8. Traditional Port Categories

The traditional IANA categories are:

```text
0–1023
System / Well-Known Ports

1024–49151
User / Registered Ports

49152–65535
Dynamic / Private Ports
```

Actual ephemeral-port ranges are OS-dependent and can differ from the traditional classification.

---

## 9. Common DevOps Ports

| Port | Protocol | Common purpose |
|---:|---|---|
| 22 | TCP | SSH |
| 25 | TCP | SMTP |
| 53 | UDP/TCP | DNS |
| 67 | UDP | DHCP server |
| 68 | UDP | DHCP client |
| 80 | TCP | HTTP |
| 123 | UDP | NTP |
| 443 | TCP | HTTPS |
| 3306 | TCP | MySQL |
| 5432 | TCP | PostgreSQL |
| 5672 | TCP | RabbitMQ AMQP |
| 6379 | TCP | Redis |
| 9200 | TCP | Elasticsearch commonly |
| 5601 | TCP | Kibana commonly |
| 8080 | TCP | Common application port |
| 9090 | TCP | Prometheus commonly |

Port numbers are conventions, not guarantees. Production systems can configure different ports.

---

## 10. Client and Server Ports

Typical TCP communication:

```text
Client:
10.0.1.20:51544

Server:
10.0.2.30:443
```

The server normally listens on a known port.

The client usually selects an ephemeral source port.

---

## 11. Ephemeral Port

An ephemeral port is a temporary source port used for outbound connections.

Example:

```text
10.0.1.20:51544
        |
        v
10.0.2.30:443
```

The client does not need to use port 443 locally.

It can use:

```text
51544
```

as its source port.

---

## 12. TCP Flow Five-Tuple

A TCP flow can be identified using:

```text
source IP
source port
destination IP
destination port
protocol
```

Example:

```text
10.0.1.20
51544
10.0.2.30
443
TCP
```

This concept is important for:

- NAT
- firewalls
- load balancers
- connection tracking
- packet captures
- troubleshooting

---

## 13. Socket

A socket is an operating-system communication endpoint.

For a TCP client:

```text
source IP + source port
```

is the local endpoint.

The server endpoint is:

```text
destination IP + destination port
```

A complete TCP flow combines both endpoints and the protocol.

---

## 14. Listening Socket

A server creates a listening socket.

Example:

```bash
ss -lntp
```

Possible output:

```text
LISTEN 0 4096 0.0.0.0:8080
```

This indicates a TCP listener on port 8080.

---

## 15. `0.0.0.0` Listener

A listener such as:

```text
0.0.0.0:8080
```

generally means the process accepts IPv4 connections arriving on configured local interfaces.

This is common inside containers.

---

## 16. Loopback Listener

A listener such as:

```text
127.0.0.1:8080
```

is bound to the local IPv4 loopback interface.

Remote clients normally cannot connect to it.

This is a common production mistake in containers:

```text
application listens on 127.0.0.1
Service expects Pod network access
traffic fails
```

---

## 17. TCP Connection

TCP establishes connection state before application data is exchanged.

Typical sequence:

```text
Client                    Server

SYN -------------------->

      <---------------- SYN-ACK

ACK -------------------->
```

The next file will examine the three-way handshake in much greater protocol-level detail.

---

## 18. Why TCP Is Reliable

TCP uses mechanisms such as:

```text
sequence numbers
acknowledgements
retransmissions
checksums
ordered delivery
flow control
congestion control
```

This allows applications to treat TCP as a reliable ordered byte stream.

---

## 19. TCP Is a Byte Stream

TCP does not preserve application message boundaries.

If an application writes:

```text
HELLO
WORLD
```

the receiving application might read:

```text
HELLOWORLD
```

or:

```text
HEL
LOWOR
LD
```

depending on socket buffering and read operations.

Applications need their own framing when message boundaries matter.

---

## 20. TCP Sequence Numbers

TCP assigns sequence numbers to the byte stream.

Conceptual example:

```text
Seq = 1000
Length = 500
```

The next expected sequence position is conceptually:

```text
1500
```

Actual TCP sequence-space behavior also accounts for SYN and FIN consuming sequence numbers.

---

## 21. TCP Acknowledgement

An ACK communicates the next byte the receiver expects.

Example:

```text
Sender:
Seq=1000
500 bytes

Receiver:
ACK=1500
```

Conceptually:

```text
"I have received through byte 1499 and expect 1500 next."
```

---

## 22. TCP Retransmission

If TCP determines that data was not delivered/acknowledged as expected, it can retransmit.

Possible causes:

```text
packet loss
congestion
bad network path
receiver problems
firewall behavior
```

---

## 23. TCP Ordering

Packets may arrive out of order:

```text
segment 1
segment 3
segment 2
```

TCP uses sequence numbers to reorder the byte stream before presenting it to the application.

---

## 24. TCP Flow Control

Flow control prevents a fast sender from overwhelming a slow receiver.

The receiver advertises a receive window.

Conceptually:

```text
Sender
  |
  | data
  v
Receiver buffer
```

A receiver with limited buffer space can reduce its advertised window.

---

## 25. TCP Congestion Control

Congestion control protects the network from excessive traffic.

Important concepts include:

```text
congestion window
slow start
congestion avoidance
fast retransmit
recovery mechanisms
```

Modern Linux TCP implementations use sophisticated congestion-control algorithms.

---

## 26. Flow Control vs Congestion Control

### Flow control

Protects:

```text
receiver
```

### Congestion control

Protects:

```text
network path
```

This distinction is frequently asked in interviews.

---

## 27. TCP Connection States

Important states include:

```text
LISTEN
SYN-SENT
SYN-RECEIVED
ESTABLISHED
FIN-WAIT-1
FIN-WAIT-2
CLOSE-WAIT
CLOSING
LAST-ACK
TIME-WAIT
CLOSED
```

A DevOps engineer should recognize these states during incidents.

---

## 28. LISTEN

A server waiting for incoming TCP connections is normally in:

```text
LISTEN
```

Check:

```bash
ss -lnt
```

---

## 29. SYN-SENT

After a client sends SYN and waits for the response, its state can be:

```text
SYN-SENT
```

Many persistent SYN-SENT connections can indicate:

```text
destination unreachable
packet filtering
routing problems
wrong address
server unavailable
```

---

## 30. SYN-RECEIVED

A server can enter:

```text
SYN-RECEIVED
```

after receiving SYN and sending SYN-ACK while waiting for the final ACK.

A large number can indicate:

```text
packet loss
client problems
firewall behavior
SYN flood
```

---

## 31. ESTABLISHED

An established TCP connection is represented as:

```text
ESTAB
```

in tools such as `ss`.

Example:

```bash
ss -nt state established
```

High connection counts are not automatically bad; interpret them against expected traffic.

---

## 32. FIN-WAIT-1

A host performing an active close sends FIN and can enter:

```text
FIN-WAIT-1
```

while waiting for acknowledgement and further close processing.

---

## 33. FIN-WAIT-2

After its FIN is acknowledged, the active closer can enter:

```text
FIN-WAIT-2
```

while waiting for the peer's FIN.

Large persistent counts may indicate unusual peer/application behavior.

---

## 34. CLOSE-WAIT

CLOSE-WAIT is important in production.

It means:

```text
remote peer closed its side
local TCP stack acknowledged it
local application has not closed its socket
```

A large and growing CLOSE-WAIT count commonly points toward an application-level socket/resource leak.

---

## 35. TIME-WAIT

TIME-WAIT is normal TCP behavior after an active close.

It helps:

```text
prevent delayed packets from corrupting future connections
allow old segments to expire
support reliable connection termination
```

Do not treat TIME-WAIT as automatically unhealthy.

---

## 36. TIME-WAIT vs CLOSE-WAIT

```text
CLOSE-WAIT
↓
peer closed
local application has not closed

TIME-WAIT
↓
active close completed
TCP retains state temporarily
```

This distinction is extremely useful during incidents.

---

## 37. TCP Termination

Typical graceful termination:

```text
Client                    Server

FIN -------------------->

      <---------------- ACK

      <---------------- FIN

ACK -------------------->
```

TCP is full-duplex, so connection shutdown can happen independently in each direction.

---

## 38. FIN

FIN means the sender has finished sending data in that direction.

It does not mean that all communication immediately disappears in both directions.

---

## 39. RST

RST means:

```text
Reset
```

It abruptly rejects or terminates a TCP connection.

Possible sources:

```text
server
application
firewall
proxy
load balancer
kernel
```

---

## 40. Connection Refused

Example:

```bash
nc -vz 10.0.2.20 8080
```

may return:

```text
Connection refused
```

This generally indicates that the destination was reachable enough to actively reject the connection, often because no process is listening on the requested port.

It is not proof of one specific root cause.

---

## 41. Connection Timeout

Example:

```bash
nc -vz 10.0.2.20 8080
```

may hang or time out.

Possible causes include:

```text
routing failure
security group
NACL
firewall
NetworkPolicy
destination failure
load balancer issue
return-path failure
```

---

## 42. Refused vs Timeout

Use this mental model:

```text
REFUSED
↓
someone responded with rejection

TIMEOUT
↓
no useful response arrived before timeout
```

The exact network cause must still be verified.

---

## 43. TCP Keepalive

Linux TCP keepalive settings can be inspected with:

```bash
sysctl net.ipv4.tcp_keepalive_time
sysctl net.ipv4.tcp_keepalive_intvl
sysctl net.ipv4.tcp_keepalive_probes
```

TCP keepalive can help identify stale idle peers.

It is different from application-level keepalive.

---

## 44. Application Keepalive

HTTP clients, databases and message clients can maintain persistent connections.

Example:

```text
Client
  |
  +-- persistent TCP connection
  |
  +-- persistent TCP connection
```

Benefits:

```text
lower latency
fewer handshakes
less CPU
less connection churn
```

---

## 45. Connection Pooling

Instead of creating a new connection for every request:

```text
request
 ↓
connect
 ↓
request
 ↓
close
```

an application can maintain:

```text
connection pool
 |
 +-- connection
 +-- connection
 +-- connection
 +-- connection
```

This is common for:

```text
databases
HTTP clients
Redis
RabbitMQ
microservices
```

---

## 46. Too Many Connections

Symptoms can include:

```text
CPU increase
memory increase
file descriptor exhaustion
ephemeral port exhaustion
TIME-WAIT growth
connection queue pressure
```

Investigate:

```text
connection pooling
keepalive
traffic patterns
application behavior
load balancer configuration
```

---

## 47. File Descriptors and Sockets

Linux represents sockets through file descriptors.

Check the shell limit:

```bash
ulimit -n
```

Check a process:

```bash
cat /proc/<PID>/limits
```

If an application reaches its file-descriptor limit, new connections can fail.

---

## 48. Socket Statistics

Useful command:

```bash
ss -s
```

It provides a summary of socket states.

Use it early during an incident to understand whether the host is experiencing unusual socket pressure.

---

## 49. Listening TCP Ports

```bash
ss -lnt
```

Example:

```text
LISTEN 0 4096 0.0.0.0:22
LISTEN 0 4096 0.0.0.0:8080
LISTEN 0 4096 127.0.0.1:9090
```

Interpret:

```text
22    → all IPv4 interfaces
8080  → all IPv4 interfaces
9090  → local-only IPv4
```

---

## 50. Process Owning a Port

Use:

```bash
sudo ss -lntp
```

or:

```bash
sudo lsof -i :8080
```

This can identify the process holding the socket.

---

## 51. Established Connections

```bash
ss -nt state established
```

Count approximately:

```bash
ss -nt state established | tail -n +2 | wc -l
```

For operational scripts, account for command/version differences.

---

## 52. Count TCP States

```bash
ss -tan | awk 'NR>1 {print $1}' | sort | uniq -c
```

Useful for identifying:

```text
ESTAB
TIME-WAIT
CLOSE-WAIT
SYN-SENT
SYN-RECV
```

patterns.

---

## 53. Find Connections to a Port

```bash
ss -nt '( dport = :443 )'
```

Depending on the `ss` version and filter syntax, use the appropriate expression supported by the host.

---

## 54. `netstat`

Older Linux environments may have:

```bash
netstat
```

Example:

```bash
netstat -lntp
```

Modern distributions generally prefer:

```bash
ss
```

from the `iproute2` package.

---

## 55. Netcat

`nc` is useful for TCP connectivity tests.

Example:

```bash
nc -vz app.example.com 443
```

It tests whether a TCP connection can be established.

It does not prove that:

```text
TLS
HTTP
authentication
application logic
```

works.

---

## 56. `curl`

For HTTP/HTTPS:

```bash
curl -v https://app.example.com
```

For TLS debugging:

```bash
curl -vk https://app.example.com
```

Useful information includes:

```text
DNS resolution
TCP connection
TLS negotiation
HTTP request
HTTP response
```

---

## 57. `openssl s_client`

Test TLS directly:

```bash
openssl s_client \
  -connect app.example.com:443 \
  -servername app.example.com
```

Useful for:

```text
TLS handshake
certificate chain
SNI
protocol
cipher
```

---

## 58. `tcpdump`

Capture TCP traffic:

```bash
sudo tcpdump -ni any tcp port 443
```

Specific host:

```bash
sudo tcpdump -ni any host 10.0.2.20 and port 443
```

This can show:

```text
SYN
SYN-ACK
ACK
data
FIN
RST
retransmissions
```

---

## 59. TCP Packet Flags

Common flags:

```text
SYN
ACK
FIN
RST
PSH
URG
```

For normal connection establishment:

```text
SYN
SYN-ACK
ACK
```

---

## 60. TCP Handshake in tcpdump

Conceptual output:

```text
client > server: Flags [S]
server > client: Flags [S.]
client > server: Flags [.]
```

Meaning:

```text
[S]   SYN
[S.]  SYN + ACK
[.]   ACK
```

---

## 61. Missing SYN-ACK

If packet capture shows:

```text
Client → Server: SYN
```

but no:

```text
Server → Client: SYN-ACK
```

possible causes include:

```text
server unreachable
firewall
security group
NACL
routing
wrong destination
listener failure
packet loss
```

Capture at multiple points when needed.

---

## 62. SYN-ACK Sent but ACK Missing

If the server sends SYN-ACK but never receives the final ACK, investigate:

```text
return route
client firewall
NACL
asymmetric routing
packet loss
stateful middleboxes
```

---

## 63. RST in tcpdump

A TCP reset indicates an active rejection/termination.

Investigate:

```text
listener
application
proxy
load balancer
firewall
connection state
```

Do not assume the server application is always the component sending RST.

---

## 64. TCP Retransmissions

Repeated retransmissions can indicate:

```text
packet loss
congestion
bad network path
MTU/PMTU issues
receiver problems
filtering
```

Use packet capture and application telemetry together.

---

## 65. TCP Backlog

A listening server maintains queues for incoming connection processing.

Inspect:

```bash
ss -lnt
```

Example:

```text
LISTEN 0 4096 0.0.0.0:8080
```

The exact queue semantics depend on Linux socket behavior and kernel configuration.

---

## 66. SYN Flood

A SYN flood sends large numbers of connection initiation attempts.

Potential effects:

```text
SYN queue pressure
CPU usage
connection tracking pressure
application availability issues
```

Defenses can include:

```text
SYN cookies
load balancers
firewalls
rate limiting
DDoS protection
```

---

## 67. UDP

UDP sends independent datagrams.

Conceptually:

```text
Client
 |
 | UDP datagram
 v
Server
```

There is no TCP-style three-way handshake.

---

## 68. UDP Characteristics

UDP provides:

```text
source port
destination port
length
checksum
datagram payload
```

It does not guarantee:

```text
delivery
ordering
duplicate prevention
retransmission
```

---

## 69. UDP Message Boundaries

Unlike TCP, UDP preserves datagram boundaries.

If an application sends:

```text
MESSAGE-A
MESSAGE-B
```

the receiver gets separate datagrams, subject to network delivery and application behavior.

---

## 70. UDP Use Cases

Common examples:

```text
DNS
DHCP
NTP
some real-time applications
some streaming systems
gaming
QUIC
```

The choice depends on application requirements.

---

## 71. DNS over UDP

Traditional DNS commonly uses:

```text
UDP 53
```

for normal queries.

DNS can also use:

```text
TCP 53
```

when required.

---

## 72. UDP Troubleshooting

A UDP port test is less definitive than TCP because there is no handshake.

For UDP:

```text
no response
```

does not necessarily mean:

```text
port closed
```

Use:

```text
application response
server logs
packet capture
```

to determine what happened.

---

## 73. UDP `nc`

Example:

```bash
nc -u -v dns.example.com 53
```

This can send UDP traffic, but a successful command invocation does not necessarily prove that the remote service received and processed the datagram.

---

## 74. TCP Port Scan vs Application Test

A TCP connection test proves:

```text
Layer 4 reachability
```

An HTTP request proves more:

```text
TCP
TLS if applicable
HTTP
application response
```

Always choose the test that answers the question you actually have.

---

## 75. Kubernetes Service Ports

Example:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cart
spec:
  selector:
    app: cart
  ports:
    - name: http
      port: 80
      targetPort: 8080
```

Traffic flow:

```text
Client
  |
  v
Service:80
  |
  v
Pod:8080
```

---

## 76. `port` vs `targetPort`

```text
port
```

is the Service-facing port.

```text
targetPort
```

is the backend destination port.

They do not have to be identical.

---

## 77. `containerPort`

Example:

```yaml
containers:
  - name: cart
    image: example/cart:1.0
    ports:
      - containerPort: 8080
```

`containerPort` is declarative metadata and does not itself cause the application to listen on that port.

The application must actually bind the port.

---

## 78. Kubernetes NodePort

A NodePort exposes a Service through a port on nodes.

Conceptually:

```text
client
 |
node-ip:nodePort
 |
Service
 |
Pod
```

For Internet-facing HTTP/HTTPS workloads on EKS, an AWS load balancer is generally preferred over directly exposing arbitrary NodePorts.

---

## 79. Kubernetes LoadBalancer

A Service can use:

```yaml
type: LoadBalancer
```

Cloud-provider integrations can provision an external load balancer.

In AWS, the resulting behavior depends on the controller and configuration.

---

## 80. ALB vs NLB

ALB is application-layer and commonly handles:

```text
HTTP
HTTPS
```

NLB is transport/network-oriented and supports:

```text
TCP
TLS
UDP
```

for supported listener configurations.

Use the appropriate AWS load balancer for the protocol and routing requirements.

---

## 81. EKS ALB Flow

For an HTTP application:

```text
Internet
   |
TCP 443
   |
AWS ALB
   |
Kubernetes target
   |
Service / Pod
   |
Application
```

With AWS Load Balancer Controller, Ingress resources can drive ALB configuration.

---

## 82. Kubernetes Service Discovery

A Service provides a stable endpoint:

```text
cart.default.svc.cluster.local
```

Applications should normally communicate through Service discovery rather than direct Pod IPs.

---

## 83. Pod-to-Service TCP Flow

Example:

```text
frontend Pod
   |
TCP 8080
   |
cart Service
   |
backend Pod
```

The exact packet path depends on:

```text
CNI
kube-proxy
eBPF implementation
Service mode
```

---

## 84. EndpointSlice

Check Service backend endpoints:

```bash
kubectl get endpointslice
```

For a specific Service:

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=cart
```

No endpoints usually means the Service has no selected usable backends.

---

## 85. Service Port Troubleshooting

Run:

```bash
kubectl get svc cart -o yaml
kubectl describe svc cart
kubectl get endpointslice \
  -l kubernetes.io/service-name=cart
```

Then inspect the Pod:

```bash
kubectl exec -it <pod> -- ss -lntp
```

Compare:

```text
Service port
targetPort
Pod listener
```

---

## 86. Readiness and TCP

A Pod can be:

```text
Running
```

but:

```text
NotReady
```

Readiness determines whether it should receive normal Service traffic.

---

## 87. TCP Readiness Probe

Example:

```yaml
readinessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
  timeoutSeconds: 2
  failureThreshold: 3
```

This verifies TCP-level availability of the port.

---

## 88. TCP vs HTTP Readiness

TCP probe:

```text
Can I connect to the port?
```

HTTP probe:

```text
Does the application return a valid health response?
```

HTTP or gRPC checks can provide deeper application health when supported.

---

## 89. Startup Probe

For slow-starting applications:

```yaml
startupProbe:
  tcpSocket:
    port: 8080
  periodSeconds: 10
  failureThreshold: 30
```

This gives the application time to start before liveness handling becomes active.

---

## 90. ALB Health Checks

ALB health checks test configured targets.

Investigate:

```text
health check port
health check path
security groups
target mode
Service
Pod readiness
application
```

A healthy TCP connection does not necessarily mean an HTTP health check succeeds.

---

## 91. AWS Security Groups and Ports

Example architecture:

```text
ALB SG:
TCP 443 from Internet

Application SG:
application port from ALB SG

Database SG:
database port from application SG
```

Prefer security-group references for AWS-to-AWS traffic where appropriate instead of broad CIDRs.

---

## 92. NACL and Ports

Network ACLs operate at subnet boundaries and are stateless.

Therefore return traffic needs corresponding rules.

This differs from security groups, which are stateful.

---

## 93. Kubernetes NetworkPolicy and Ports

Example:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-redis
spec:
  podSelector:
    matchLabels:
      app: redis
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: cart
      ports:
        - protocol: TCP
          port: 6379
```

The actual policy also needs to account for namespace boundaries and the CNI's NetworkPolicy implementation.

---

## 94. TCP and DNS

DNS resolution can involve:

```text
UDP 53
TCP 53
```

depending on query behavior and resolver requirements.

If DNS is blocked, application failures can look like general networking failures.

---

## 95. TCP and TLS

HTTPS commonly follows:

```text
TCP handshake
      ↓
TLS handshake
      ↓
HTTP
```

Therefore:

```bash
nc -vz example.com 443
```

proves only TCP connectivity.

Use:

```bash
curl -vk https://example.com
```

for TLS and HTTP testing.

---

## 96. TLS SNI

When multiple HTTPS sites share infrastructure, SNI allows the client to indicate the hostname during TLS negotiation.

Example:

```text
example.com
api.example.com
shop.example.com
```

A TCP connection to port 443 alone does not identify which HTTPS application should handle the request.

---

## 97. HTTP/2

HTTP/2 commonly runs over:

```text
HTTP/2
 ↓
TLS
 ↓
TCP
```

It multiplexes multiple HTTP streams over fewer TCP connections.

---

## 98. HTTP/3 and QUIC

HTTP/3 uses:

```text
HTTP/3
 ↓
QUIC
 ↓
UDP
 ↓
IP
```

QUIC implements transport features above UDP.

This is why modern networking knowledge must distinguish:

```text
HTTP/1.1 / HTTP/2
```

from:

```text
HTTP/3
```

---

## 99. Database Ports

Typical examples:

```text
MongoDB → 27017
Redis → 6379
PostgreSQL → 5432
MySQL → 3306
RabbitMQ AMQP → 5672
```

These are defaults/conventions, not guarantees.

---

## 100. RoboShop Connectivity

Example:

```text
catalog
  |
  | TCP 27017
  v
MongoDB

cart
  |
  | TCP 6379
  v
Redis

order
  |
  | TCP 5672
  v
RabbitMQ
```

The actual RoboShop deployment must use the ports configured in its manifests/Helm values.

---

## 101. Database Port Test

Example:

```bash
kubectl exec -it <catalog-pod> -- \
  nc -vz mongodb 27017
```

If TCP succeeds but the application fails, investigate:

```text
credentials
TLS
database name
application configuration
database health
connection pool
```

---

## 102. Redis Port Test

```bash
kubectl exec -it <cart-pod> -- \
  nc -vz redis 6379
```

Then test Redis at the application/protocol layer.

TCP success does not prove Redis authentication or command execution.

---

## 103. RabbitMQ Port Test

```bash
kubectl exec -it <order-pod> -- \
  nc -vz rabbitmq 5672
```

Then verify:

```text
credentials
vhost
TLS if enabled
exchange/queue configuration
application logs
```

---

## 104. Prometheus Networking

Prometheus commonly scrapes HTTP metrics endpoints.

Example:

```text
Prometheus
   |
TCP 9100
   |
node_exporter
```

or:

```text
Prometheus
   |
TCP 8080
   |
application /metrics
```

The actual exporter/application port is configuration-dependent.

---

## 105. ELK Networking

Typical examples:

```text
Elasticsearch → 9200
Kibana → 5601
```

Logstash may use different input ports for different protocols.

Always inspect the actual configuration.

---

## 106. Port Matrix

Production teams should document:

| Source | Destination | Protocol | Port | Purpose |
|---|---|---|---:|---|
| Internet | ALB | TCP | 443 | HTTPS |
| ALB | Frontend | TCP | 8080 | Application |
| Frontend | Cart | TCP | 8080 | Service |
| Cart | Redis | TCP | 6379 | Cache |
| Order | RabbitMQ | TCP | 5672 | Messaging |
| Catalog | MongoDB | TCP | 27017 | Database |

This becomes extremely useful during incidents.

---

## 107. Port Mismatch Incident

Suppose:

```text
Application listens: 8080
Service targetPort: 8081
```

Traffic fails because the Service forwards to the wrong backend port.

Check:

```bash
kubectl get svc <service> -o yaml
kubectl exec -it <pod> -- ss -lntp
```

---

## 108. Port Chain

Trace the entire chain:

```text
Client
 ↓
ALB listener
 ↓
Target
 ↓
Kubernetes Service
 ↓
targetPort
 ↓
Pod
 ↓
Process
```

A failure anywhere can appear as a generic application outage.

---

## 109. Production TCP Troubleshooting Model

Ask:

```text
1. What is the source?
2. What is the destination?
3. What IP is being used?
4. What protocol?
5. What port?
6. Is DNS correct?
7. Is there a route?
8. Is the port listening?
9. Is traffic allowed?
10. Does the TCP handshake complete?
11. Does TLS succeed?
12. Does the application respond?
```

---

## 110. `ip route get`

Before testing a connection, determine the local route:

```bash
ip route get 10.0.2.20
```

This helps identify:

```text
interface
source address
next hop
route selection
```

---

## 111. `nc` Test

```bash
nc -vz 10.0.2.20 8080
```

Interpret:

```text
succeeded
```

as evidence of TCP connection establishment.

Interpret:

```text
refused
```

as active rejection.

Interpret:

```text
timed out
```

as lack of a successful response within the timeout.

---

## 112. `curl` Test

```bash
curl -v http://10.0.2.20:8080
```

This tests more than TCP.

It can expose:

```text
HTTP status
headers
redirects
application response
```

---

## 113. `tcpdump` Test

```bash
sudo tcpdump -ni any \
  host 10.0.2.20 and port 8080
```

Use this when command-level results are insufficient.

---

## 114. TCP Timeout Decision Tree

```text
SYN sent?
 |
 +-- No → application/socket problem
 |
 +-- Yes
      |
      +-- SYN-ACK received?
             |
             +-- No → routing/filtering/destination issue
             |
             +-- Yes
                    |
                    +-- ACK sent?
                           |
                           +-- No → return-path/client issue
                           |
                           +-- Yes → TCP established
```

Then investigate TLS/application layers.

---

## 115. Connection Refused Decision Tree

```text
TCP SYN
  |
  v
RST
  |
  v
Active rejection
```

Check:

```text
listener
Service
targetPort
application
load balancer
firewall
```

---

## 116. CLOSE-WAIT Incident

### Symptom

```text
CLOSE-WAIT
```

connections continuously increase.

### Check

```bash
ss -tan state close-wait
```

Then identify:

```text
PID
remote endpoint
application
```

Likely issue:

```text
application is not closing sockets
```

---

## 117. TIME-WAIT Incident

### Symptom

Large number of:

```text
TIME-WAIT
```

### Investigate

```text
short-lived connections
active closer
keepalive
connection pool
proxy
load balancer
NAT
ephemeral ports
```

TIME-WAIT itself is not a failure.

---

## 118. Ephemeral Port Exhaustion

Symptoms:

```text
connect failures
cannot assign requested address
high connection churn
```

Check:

```bash
cat /proc/sys/net/ipv4/ip_local_port_range
ss -s
```

Then evaluate:

```text
connection pooling
keepalive
destination concentration
source address scaling
```

---

## 119. File Descriptor Exhaustion

Check:

```bash
ulimit -n
```

For a process:

```bash
cat /proc/<PID>/limits
```

If the process has many sockets:

```bash
ls -l /proc/<PID>/fd | wc -l
```

A network outage can actually be an application file-descriptor exhaustion problem.

---

## 120. TCP and MTU

TCP connectivity can succeed while large data transfers fail because of MTU/path-MTU problems.

Symptoms:

```text
small request succeeds
large response hangs
TLS begins successfully
transfer stalls
```

Investigate:

```text
MTU
MSS
PMTUD
VPN
overlay networking
firewalls
```

---

## 121. MSS

MSS means:

```text
Maximum Segment Size
```

It is the maximum TCP payload size negotiated for a connection.

Conceptually:

```text
MSS ≈ MTU - IP header - TCP header
```

For a simple IPv4 Ethernet example:

```text
MTU = 1500
IP header = 20
TCP header = 20

MSS ≈ 1460
```

TCP options can change the exact calculation.

---

## 122. EKS MTU

EKS networking can involve:

```text
VPC networking
CNI behavior
encapsulation where applicable
VPN
transit networking
```

Therefore determine the actual path and interface MTU before changing settings.

---

## 123. ALB Idle Timeout

Long-lived connections can be affected by intermediary idle timeouts.

Review:

```text
client timeout
ALB timeout
server keepalive
application heartbeat
```

This matters for:

```text
WebSockets
streaming
long polling
gRPC
long-running HTTP requests
```

---

## 124. NAT and TCP

NAT translates source information.

Example:

```text
Private:
10.0.1.20:51544

NAT:
198.51.100.20:40001

Destination:
203.0.113.50:443
```

The NAT device maintains connection state.

---

## 125. NAT Port Pressure

High outbound connection rates can create pressure on:

```text
ephemeral ports
NAT translation state
connection tracking
```

Use:

```text
connection reuse
VPC endpoints
multiple NAT paths
better source distribution
```

where architecture requires it.

---

## 126. Security Best Practice

Do not expose every listening port.

For example:

```text
Internet
 ↓
ALB:443
 ↓
Application:8080
 ↓
Database:5432
```

Only intended paths should be allowed.

---

## 127. Security Groups

Example:

```text
ALB SG:
443 from Internet

App SG:
8080 from ALB SG

DB SG:
5432 from App SG
```

This creates least-privilege network access.

---

## 128. Network Segmentation

Separate:

```text
public
application
data
management
monitoring
```

where justified by the architecture.

CIDR boundaries and security policies should reinforce the intended trust model.

---

## 129. Kubernetes NetworkPolicy

NetworkPolicy can restrict:

```text
source Pod
destination Pod
namespace
IP block
port
protocol
```

Do not rely only on VPC security groups for Kubernetes east-west segmentation.

---

## 130. DNS Egress and NetworkPolicy

With default-deny egress, allow DNS as required:

```text
Pod
 ↓
CoreDNS
 ↓
UDP/TCP 53
```

Otherwise Service discovery can fail and appear to be an application connectivity issue.

---

## 131. RoboShop External Flow

Production-style flow:

```text
User
 |
TCP 443
 |
AWS ALB
 |
HTTP/HTTPS
 |
Frontend
 |
Kubernetes Service
 |
Microservices
```

There is no API Gateway in this architecture.

---

## 132. RoboShop Internal Flow

Example:

```text
Frontend
 |
TCP 8080
 v
Cart
 |
TCP 6379
 v
Redis
```

and:

```text
Order
 |
TCP 5672
 v
RabbitMQ
```

and:

```text
Catalog
 |
TCP 27017
 v
MongoDB
```

---

## 133. RoboShop Failure Example

Symptom:

```text
Cart page returns errors
```

Check:

```bash
kubectl exec -it <frontend-pod> -- \
  nc -vz cart 8080
```

If it fails:

```text
DNS
Service
EndpointSlice
NetworkPolicy
Pod listener
```

If it succeeds:

```text
application protocol/configuration
```

becomes the next focus.

---

## 134. Production Port Matrix for RoboShop

Maintain a documented matrix containing:

```text
service
source
destination
protocol
port
health check
security group
NetworkPolicy
```

Example:

```text
frontend → cart
TCP 8080

cart → redis
TCP 6379

order → rabbitmq
TCP 5672

catalog → mongodb
TCP 27017
```

---

## 135. Common Mistake: `containerPort`

Incorrect assumption:

```text
containerPort: 8080
```

means:

```text
application is listening on 8080
```

It does not.

Always verify:

```bash
ss -lntp
```

inside the container when appropriate.

---

## 136. Common Mistake: Port Reachability Equals Application Health

Incorrect:

```text
nc 443 succeeds
therefore application is healthy
```

Correct:

```text
TCP works
then test TLS
then test HTTP
then test application health
```

---

## 137. Common Mistake: TIME-WAIT Is a Leak

TIME-WAIT is normal TCP behavior.

Investigate connection churn, but do not treat every TIME-WAIT socket as a memory leak.

---

## 138. Common Mistake: CLOSE-WAIT Is a Network Failure

CLOSE-WAIT commonly indicates the peer has closed and the local application has not closed its socket.

Investigate application resource management.

---

## 139. Common Mistake: UDP `nc` Proves Port Is Open

UDP has no connection handshake.

A command completing successfully does not prove the remote application received or responded.

Use:

```text
packet capture
logs
protocol-aware testing
```

---

## 140. Common Mistake: Open Port Means Secure

A reachable port only means traffic can reach the endpoint.

Security also requires:

```text
authentication
authorization
encryption
least privilege
network segmentation
```

---

## 141. Production Runbook: TCP Failure

```text
1. Identify source Pod/host.
2. Identify destination Service/IP.
3. Resolve DNS.
4. Check route.
5. Check listener.
6. Test with nc.
7. Check SG/NACL/NetworkPolicy.
8. Capture traffic.
9. Check TLS.
10. Check application logs.
```

---

## 142. Production Runbook: Kubernetes Service Failure

```bash
kubectl get svc <service> -o yaml
kubectl describe svc <service>
kubectl get endpointslice \
  -l kubernetes.io/service-name=<service>
kubectl get pods --show-labels
kubectl exec -it <pod> -- ss -lntp
```

Compare:

```text
selector
endpoints
port
targetPort
listener
readiness
```

---

## 143. Production Runbook: ALB Backend Failure

Check:

```text
Ingress
ALB listener
target group
target health
security groups
Service
targetPort
Pod readiness
application
```

Then use:

```bash
kubectl describe ingress <ingress>
kubectl get svc
kubectl get pods
```

and AWS tooling appropriate to the environment.

---

## 144. Production Runbook: EKS Outbound TCP Failure

Check:

```text
Pod DNS
Pod route
VPC subnet
route table
NAT Gateway
security group
NACL
destination
```

For private workloads:

```text
Private subnet
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

where that architecture is used.

---

## 145. Production Architecture

```text
                        Internet
                           |
                        TCP 443
                           |
                           v
                    +--------------+
                    |   AWS ALB    |
                    +--------------+
                           |
                           v
                    Kubernetes Ingress
                           |
                           v
                    Kubernetes Service
                           |
                           v
                         Pod
                           |
                           v
                      Application
```

Each hop has its own networking and security controls.

---

## 146. TCP Layer Mental Model

```text
Application
    |
    v
Socket
    |
    v
TCP
    |
    v
IP
    |
    v
Network interface
    |
    v
Network path
```

When troubleshooting, determine which layer is failing.

---

## 147. UDP Layer Mental Model

```text
Application
    |
    v
UDP socket
    |
    v
UDP datagram
    |
    v
IP
    |
    v
Network
```

There is no TCP-style connection establishment.

---

## 148. TCP Command Cheat Sheet

```bash
ss -lntp
ss -ant
ss -s
ss -tan state established
ss -tan state close-wait
ss -tan state time-wait

nc -vz host port

curl -v URL
curl -vk URL

openssl s_client -connect host:443 -servername host

sudo tcpdump -ni any tcp port 443

sudo lsof -i :8080
```

---

## 149. Kubernetes Command Cheat Sheet

```bash
kubectl get svc
kubectl describe svc <service>
kubectl get endpointslice
kubectl get pods --show-labels
kubectl describe pod <pod>
kubectl exec -it <pod> -- ss -lntp
kubectl get networkpolicy
kubectl describe ingress <ingress>
kubectl get events --sort-by=.lastTimestamp
```

---

## 150. Interview: What Is TCP?

TCP is a connection-oriented, reliable, ordered byte-stream transport protocol that provides mechanisms such as acknowledgements, retransmissions, flow control and congestion control.

---

## 151. Interview: What Is UDP?

UDP is a connectionless datagram transport protocol with low overhead that does not provide TCP-style reliability and ordering.

---

## 152. Interview: What Is a Port?

A port identifies a transport-layer endpoint. TCP and UDP have separate port namespaces ranging from 0 to 65535.

---

## 153. Interview: What Is an Ephemeral Port?

An ephemeral port is a temporary source port commonly selected by the operating system for outbound connections.

---

## 154. Interview: What Is a Socket?

A socket is an operating-system communication endpoint used by applications to send and receive network data.

---

## 155. Interview: What Is the TCP Three-Way Handshake?

```text
SYN
SYN-ACK
ACK
```

It establishes TCP connection state and synchronizes initial sequence information.

---

## 156. Interview: Why Is TCP Called Reliable?

Because it provides mechanisms for:

```text
sequence tracking
acknowledgement
retransmission
ordering
flow control
```

---

## 157. Interview: Is TCP Message-Oriented?

No.

TCP is a byte stream. Application-level framing is required when messages have boundaries.

---

## 158. Interview: What Is CLOSE-WAIT?

It means the peer has closed its side, but the local application has not yet closed the corresponding socket.

---

## 159. Interview: What Is TIME-WAIT?

A TCP state maintained after active closure to help prevent delayed segments from interfering with later connections and to support correct TCP termination.

---

## 160. Interview: CLOSE-WAIT vs TIME-WAIT?

```text
CLOSE-WAIT:
application has not closed

TIME-WAIT:
TCP retains state after active close
```

---

## 161. Interview: What Is RST?

RST is a TCP reset used to abruptly reject or terminate a connection.

---

## 162. Interview: Connection Refused vs Timeout?

Refused generally means an active rejection occurred.

Timeout means no successful response was received within the timeout.

---

## 163. Interview: Flow Control vs Congestion Control?

```text
Flow control → receiver protection
Congestion control → network protection
```

---

## 164. Interview: How Do You Check Listening Ports?

```bash
ss -lntp
```

---

## 165. Interview: How Do You Test a TCP Port?

```bash
nc -vz host port
```

Then test the application protocol separately.

---

## 166. Interview: How Do You Troubleshoot a TCP Timeout?

Use:

```text
DNS
routing
security groups
NACLs
firewalls
NetworkPolicy
listener
load balancer
tcpdump
application logs
```

in a layered sequence.

---

## 167. Interview: How Do You Troubleshoot CLOSE-WAIT?

Identify the owning process and investigate its connection lifecycle, connection pool and socket cleanup behavior.

---

## 168. Interview: How Do You Troubleshoot TIME-WAIT?

Investigate:

```text
connection churn
keepalive
pooling
active closer
NAT
load balancer
ephemeral port capacity
```

before changing kernel parameters.

---

## 169. Interview: Why Can TCP 443 Work While HTTPS Fails?

Because TCP is Layer 4 while HTTPS requires TLS and HTTP above it.

TCP success does not prove TLS or application success.

---

## 170. Interview: What Is Kubernetes `targetPort`?

It is the backend port on selected Pods to which Service traffic is directed.

---

## 171. Interview: Does `containerPort` Make an Application Listen?

No.

The application process must bind and listen itself.

---

## 172. Interview: ALB vs NLB?

ALB is application-layer and commonly used for HTTP/HTTPS.

NLB is transport-oriented and supports TCP and other supported transport protocols.

---

## 173. Interview: Why Use Connection Pooling?

To reuse connections, reduce handshake overhead and connection churn, and improve latency/resource efficiency.

---

## 174. Interview: What Is Ephemeral Port Exhaustion?

It occurs when a client/NAT path cannot allocate sufficient source ports for new outbound connections.

---

## 175. Interview: Why Can a Running Pod Be Unreachable?

Because:

```text
Running ≠ Ready
```

The Service may have no endpoints, the process may not be listening, or network policy/security controls may block traffic.

---

## 176. Interview: How Do You Debug a Kubernetes Service?

```text
Service selector
↓
EndpointSlice
↓
Pod readiness
↓
targetPort
↓
process listener
↓
NetworkPolicy
↓
CNI/network
```

---

## 177. Interview: How Do You Prove Where a TCP Connection Fails?

Use packet capture.

For example:

```bash
sudo tcpdump -ni any host <ip> and port <port>
```

Look for:

```text
SYN
SYN-ACK
ACK
RST
retransmissions
```

---

## 178. Interview: What Is a Five-Tuple?

```text
source IP
source port
destination IP
destination port
protocol
```

It identifies a network flow.

---

## 179. Interview: Why Is UDP Harder to Test?

Because UDP has no connection handshake.

No response does not necessarily mean the destination port is closed.

---

## 180. Interview: What Is an ALB Health Check?

It is a load-balancer health mechanism used to determine whether configured targets should receive traffic. It is distinct from Kubernetes readiness and application-level health.

---

## 181. Interview: Why Can an ALB Be Healthy but the Application Fail?

Because:

```text
health check path may be shallow
```

or the real request can differ in:

```text
host
path
authentication
dependencies
```

Health checks should represent meaningful service availability without creating excessive dependency coupling.

---

## 182. Interview: Why Can a Service Have Running Pods but No Traffic?

Possible reasons:

```text
selector mismatch
no EndpointSlices
Pods NotReady
targetPort mismatch
NetworkPolicy
application not listening
```

---

## 183. Interview: What Is MSS?

MSS is the maximum TCP payload size negotiated for a connection.

It is related to MTU and can help avoid fragmentation at the transport layer.

---

## 184. Interview: What Is TCP Keepalive?

TCP keepalive is a mechanism for probing idle connections to determine whether a peer remains reachable.

It is not the same as HTTP keepalive.

---

## 185. Interview: What Is the Difference Between TCP Keepalive and HTTP Keepalive?

```text
TCP keepalive:
transport-level peer probing

HTTP keepalive:
application/protocol-level connection reuse
```

They solve different problems.

---

## 186. Interview: Why Does EKS IP Planning Matter to TCP?

Because Pods and load-balancing infrastructure need usable network addresses. Insufficient IP capacity can prevent new workloads and network endpoints from being created.

---

## 187. Production Checklist

```text
[ ] Understand TCP
[ ] Understand UDP
[ ] Understand ports
[ ] Understand sockets
[ ] Understand five-tuples
[ ] Understand ephemeral ports
[ ] Know TCP states
[ ] Know CLOSE-WAIT
[ ] Know TIME-WAIT
[ ] Know RST
[ ] Know connection refused vs timeout
[ ] Know ss
[ ] Know nc
[ ] Know curl
[ ] Know tcpdump
[ ] Know TLS testing
[ ] Understand Kubernetes Service ports
[ ] Understand targetPort
[ ] Understand EndpointSlices
[ ] Understand ALB/NLB
[ ] Understand EKS networking
[ ] Understand connection pooling
[ ] Understand file descriptor limits
[ ] Understand NAT port pressure
[ ] Understand MTU/MSS
[ ] Understand NetworkPolicy
[ ] Maintain port matrices
```

---

## 188. Final Mental Model

For a TCP application request:

```text
Application
   |
Socket
   |
Ephemeral source port
   |
TCP
   |
Destination IP
   |
Destination port
   |
Network path
   |
Server listener
   |
TLS
   |
Application protocol
```

For Kubernetes:

```text
Client
   |
Service
   |
EndpointSlice
   |
Pod IP
   |
targetPort
   |
Application listener
```

For EKS:

```text
Internet
   |
ALB
   |
EKS
   |
Service
   |
Pod
   |
Application
```

When something breaks, find the first layer where reality differs from the expected flow.

---

## 189. Next File

The next planned file is:

```text
08-TCP-Three-Way-Handshake.md
```

That file will go much deeper into:

```text
SYN
sequence numbers
ISN
SYN-ACK
ACK
TCP state transitions
SYN backlog
SYN cookies
connection establishment
connection termination
RST
retransmissions
packet captures
Wireshark
tcpdump
network latency
handshake failures
SYN floods
production incidents
Linux troubleshooting
AWS/EKS scenarios
interview questions
```

# End of 07-TCP-UDP-and-Ports.md
