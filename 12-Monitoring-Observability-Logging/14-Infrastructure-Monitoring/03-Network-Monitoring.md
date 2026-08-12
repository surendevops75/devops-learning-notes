# Network Monitoring

## 1. Overview

Network monitoring is the process of continuously observing network connectivity, traffic, latency, packet loss, errors, bandwidth, connections, and network-path behavior.

For a DevOps engineer, network monitoring is critical because many production application problems are actually caused by networking issues.

A typical production request path is:

```text
Client
   ↓
DNS
   ↓
Load Balancer
   ↓
Network
   ↓
EC2 / Kubernetes
   ↓
Application
   ↓
Database / External Service
```

A failure at any layer can result in:

```text
Connection timeout
Connection refused
High latency
Packet loss
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
Application timeout
Service unreachable
```

Network monitoring should therefore cover:

```text
Connectivity
DNS
Routing
TCP
Ports
Bandwidth
Throughput
Latency
Packet Loss
Packet Errors
Packet Drops
Connections
Application Connectivity
```

---

# 2. Network Monitoring Layers

A practical production network monitoring model is:

```text
                         NETWORK MONITORING
                                │
              ┌─────────────────┼─────────────────┐
              ↓                 ↓                 ↓
        Connectivity          Traffic        Performance
              │                 │                 │
        ┌─────┼─────┐      ┌────┼────┐       ┌────┼────┐
        ↓     ↓     ↓      ↓    ↓    ↓       ↓    ↓    ↓
       DNS   TCP   Ports  Bytes Packets Bandwidth Latency Loss Errors
        │     │     │
        └─────┼─────┘
              ↓
          Application
              ↓
         Dependencies
```

The goal is to answer:

```text
Is the destination reachable?
Is DNS resolving?
Is routing correct?
Is the port reachable?
Is the service listening?
Is traffic being dropped?
Is latency increasing?
Is packet loss occurring?
Are connections increasing?
Is the application responding?
```

---

# 3. Important Network Metrics

Important production network metrics include:

```text
Bandwidth
Throughput
Latency
Packet Loss
Packet Errors
Packet Drops
TCP Connections
TCP States
DNS Resolution Time
Network Interface Utilization
Network Availability
```

These metrics should be correlated instead of investigating only one metric.

For example:

```text
Latency ↑
   +
Packet Loss ↑
   +
Network Errors ↑
```

is much more meaningful than looking only at latency.

---

# 4. Network Availability

Network availability represents whether a service or network endpoint can be reached when required.

Example:

```text
Expected Availability = 100%

09:00 → Available
09:10 → Available
09:20 → Unreachable
09:21 → Unreachable
09:22 → Available
```

The outage period should be investigated.

Availability monitoring should distinguish between:

```text
Host availability
Port availability
Service availability
Application availability
```

A server being reachable does not necessarily mean the application is healthy.

---

# 5. Connectivity Monitoring

Connectivity monitoring verifies whether one endpoint can communicate with another.

Example:

```text
Application Server
       ↓
Database Server
```

The important question is:

```text
Can the application establish a connection to the database?
```

Basic tests include:

```bash
ping <host>
nc -vz <host> <port>
curl -v <url>
```

The appropriate test depends on the protocol.

---

# 6. Ping

Use:

```bash
ping <host>
```

Example:

```bash
ping 10.0.1.10
```

Ping can help determine:

```text
Host reachability
Round-trip latency
Packet loss
```

Example:

```text
64 bytes from 10.0.1.10:
time=2.31 ms
```

However:

```text
ping success
≠
application success
```

ICMP may be disabled while TCP or HTTPS is working.

---

# 7. Ping Limitations

Consider:

```text
ping works
```

but:

```text
HTTPS fails
Database connection fails
Application port is closed
```

Therefore do not depend only on ping.

A better troubleshooting flow is:

```text
Ping
   ↓
DNS
   ↓
TCP Port
   ↓
TLS
   ↓
HTTP
   ↓
Application
```

---

# 8. TCP Port Monitoring

Use:

```bash
nc -vz <host> <port>
```

Example:

```bash
nc -vz 10.0.1.10 443
```

Possible results:

```text
Connection succeeded
Connection refused
Connection timed out
```

This is useful because it tests the actual TCP port used by the application.

---

# 9. Connection Refused

Example:

```text
Connection refused
```

This generally means the destination was reachable but the connection was actively rejected.

A common reason is:

```text
No application is listening on the requested port
```

Check:

```bash
ss -lntp
```

Then verify:

```text
Application running?
Correct port?
Service listening?
Local firewall?
```

---

# 10. Connection Timeout

Example:

```text
Connection timed out
```

Investigate:

```text
DNS
Routing
Security Group
Network ACL
Firewall
Network path
Destination availability
```

A timeout commonly indicates that traffic is being dropped or the destination is not responding.

---

# 11. Listening Ports

Check listening TCP ports:

```bash
ss -lntp
```

Example:

```text
LISTEN 0 128 0.0.0.0:80
LISTEN 0 128 0.0.0.0:443
```

This confirms that a local process is listening on the port.

It does not prove that external clients can reach it.

You must also verify:

```text
Routing
Security
Firewall
Network ACL
Load Balancer
```

---

# 12. Established Connections

Check established connections:

```bash
ss -ant state established
```

Get a connection summary:

```bash
ss -s
```

Monitor connection counts over time.

A sudden increase may indicate:

```text
Traffic spike
Connection leak
Slow downstream service
Client behavior
Application issue
```

---

# 13. TCP States

Important TCP states include:

```text
LISTEN
ESTABLISHED
TIME_WAIT
CLOSE_WAIT
SYN_SENT
SYN_RECV
FIN_WAIT
```

Understanding these states helps identify connection-related problems.

---

# 14. TIME_WAIT

`TIME_WAIT` appears after TCP connections close.

A large number can occur because of:

```text
High request rate
Short-lived connections
Frequent connection creation
```

Do not automatically treat TIME_WAIT as a problem.

Investigate:

```text
Connection rate
Ephemeral ports
Connection reuse
Keep-alive configuration
```

---

# 15. CLOSE_WAIT

`CLOSE_WAIT` occurs when the remote side closes the connection but the local application has not completely closed its socket.

Check:

```bash
ss -ant state close-wait
```

Potential flow:

```text
Remote closes connection
        ↓
Local application receives close
        ↓
Application does not close socket
        ↓
CLOSE_WAIT increases
        ↓
File descriptors increase
        ↓
Resource exhaustion
```

A continuously increasing `CLOSE_WAIT` count should be investigated.

---

# 16. SYN-SENT

`SYN-SENT` means the client has sent a TCP SYN and is waiting for a response.

A large number can indicate:

```text
Destination unavailable
Network filtering
Routing problems
Server overload
```

Investigate the network path and destination.

---

# 17. SYN-RECV

`SYN-RECV` means the server has received a SYN and responded with SYN-ACK but is waiting for the final ACK.

A large buildup can indicate:

```text
Connection flood
Network problems
Client issues
Server pressure
```

Correlate with:

```text
Traffic
CPU
Connection rate
Network errors
```

---

# 18. DNS Monitoring

DNS is a critical component of network connectivity.

Typical flow:

```text
Application
     ↓
DNS Query
     ↓
DNS Resolver
     ↓
IP Address
     ↓
TCP Connection
```

DNS failures can appear as:

```text
Application timeout
Connection failure
Service unavailable
```

Monitor:

```text
DNS availability
DNS resolution time
DNS errors
Record correctness
Resolver behavior
```

---

# 19. DNS Troubleshooting

Use:

```bash
nslookup example.com
```

or:

```bash
dig example.com
```

Check:

```text
DNS resolution
Returned IP
Response time
Record type
Resolver behavior
```

For a specific record:

```bash
dig example.com A
```

---

# 20. DNS Latency

DNS lookup time contributes to overall request latency.

Example:

```text
Application Request
       ↓
DNS Lookup
       ↓
300 ms
       ↓
TCP Connection
```

If DNS is slow, the application may appear slow even when the backend application is healthy.

Therefore monitor DNS latency separately from application latency.

---

# 21. Routing

Linux routing determines where packets are sent.

Check:

```bash
ip route
```

Example:

```text
default via 10.0.1.1 dev eth0
10.0.1.0/24 dev eth0
```

Routing problems can cause:

```text
Unreachable hosts
Connection timeouts
Incorrect traffic paths
Asymmetric connectivity
```

---

# 22. Route Troubleshooting

Check:

```bash
ip route
```

Then test:

```bash
ping <destination>
```

and:

```bash
traceroute <destination>
```

or:

```bash
tracepath <destination>
```

Verify:

```text
Source network
Destination network
Default route
Specific route
Gateway
Network interface
```

---

# 23. Traceroute

Traceroute helps identify the network path to a destination.

Conceptually:

```text
Source
  ↓
Router 1
  ↓
Router 2
  ↓
Router 3
  ↓
Destination
```

It can help identify where latency or connectivity changes occur.

---

# 24. Traceroute Limitations

Traceroute is not always definitive because:

```text
ICMP may be filtered
UDP may be filtered
Routers may not respond
Load balancing may change paths
```

Therefore use traceroute as one troubleshooting signal.

Do not conclude that a hop is broken simply because it does not respond.

---

# 25. HTTP Connectivity

For HTTP services:

```bash
curl -I https://example.com
```

For detailed troubleshooting:

```bash
curl -v https://example.com
```

This can show:

```text
DNS resolution
TCP connection
TLS negotiation
HTTP response
Redirects
Headers
```

---

# 26. HTTP Request Timing

A request can be broken into:

```text
DNS Lookup
      ↓
TCP Connection
      ↓
TLS Handshake
      ↓
Time To First Byte
      ↓
Response
```

This helps determine where latency is introduced.

For detailed timing:

```bash
curl -o /dev/null -s -w \
"DNS: %{time_namelookup}\nConnect: %{time_connect}\nTLS: %{time_appconnect}\nTTFB: %{time_starttransfer}\nTotal: %{time_total}\n" \
https://example.com
```

---

# 27. TLS Monitoring

For HTTPS applications, monitor:

```text
Certificate validity
TLS handshake
TLS errors
Certificate chain
Protocol compatibility
```

TLS problems can appear as:

```text
Connection failure
Handshake failure
Application unavailable
```

Test TLS:

```bash
openssl s_client -connect example.com:443
```

---

# 28. Bandwidth

Bandwidth represents the maximum amount of data that a network path or interface can transfer.

Example:

```text
Network Interface
       │
       ├── Incoming Traffic
       └── Outgoing Traffic
```

Monitor:

```text
Bytes received
Bytes transmitted
Bits per second
Peak traffic
Average traffic
```

High bandwidth utilization can result in:

```text
Higher latency
Packet drops
Reduced throughput
Application timeouts
```

---

# 29. Throughput

Throughput represents the actual amount of data successfully transferred over a period of time.

Example:

```text
Network Capacity = 1 Gbps
Current Traffic  = 850 Mbps
```

The interface is heavily utilized.

Throughput should be correlated with:

```text
Latency
Packet loss
Errors
Application performance
```

High throughput is not necessarily a problem if the workload is expected.

---

# 30. Network Latency

Latency is the time required for network communication between two endpoints.

Example:

```text
Client
  ↓
Network
  ↓
Server
```

If round-trip latency is:

```text
20 ms
```

the connection is relatively fast.

If it becomes:

```text
800 ms
```

applications may become slow or start timing out.

---

# 31. Packet Loss

Packet loss occurs when packets fail to reach their destination.

Example:

```text
Packets Sent     = 1000
Packets Received = 980

Packet Loss      = 2%
```

Packet loss can cause:

```text
Retransmissions
Higher latency
Reduced throughput
Application timeouts
Poor user experience
```

Even a small amount of packet loss can become significant for latency-sensitive applications.

---

# 32. Network Errors

Network interfaces can report:

```text
RX errors
TX errors
CRC errors
Frame errors
Carrier errors
```

Check Linux network statistics:

```bash
ip -s link
```

Example:

```text
RX:
    packets
    errors
    dropped

TX:
    packets
    errors
    dropped
```

Unexpected increases should be investigated.

---

# 33. Network Drops

Packet drops are different from transmission errors.

Check:

```bash
ip -s link
```

Look for:

```text
RX dropped
TX dropped
```

Possible causes include:

```text
Traffic bursts
Buffer exhaustion
Interface limitations
Kernel queue pressure
Network congestion
```

Always compare current values with historical behavior.

---

# 34. Linux Network Interfaces

List interfaces:

```bash
ip link
```

Show IP addresses:

```bash
ip addr
```

Show interface statistics:

```bash
ip -s link
```

Example:

```text
lo
eth0
```

Interface names can vary depending on the operating system and environment.

---

# 35. Network Interface Monitoring

A production monitoring dashboard should track:

```text
Incoming traffic
Outgoing traffic
Packets received
Packets transmitted
RX errors
TX errors
RX drops
TX drops
```

Example:

```text
eth0
│
├── RX bytes
├── TX bytes
├── RX packets
├── TX packets
├── RX errors
├── TX errors
├── RX drops
└── TX drops
```

---

# 36. AWS Network Monitoring

In AWS, network monitoring includes:

```text
VPC
│
├── Subnets
├── Route Tables
├── Security Groups
├── Network ACLs
├── Internet Gateway
├── NAT Gateway
├── Load Balancer
└── EC2
```

An application can only communicate correctly when the complete network path is correctly configured.

---

# 37. AWS Public Application Network Path

Example:

```text
Internet
    ↓
Internet Gateway
    ↓
Public Subnet
    ↓
Application Load Balancer
    ↓
Private Subnet
    ↓
EC2
    ↓
Application
```

Monitor every important boundary.

---

# 38. AWS Private Application Outbound Path

For a private EC2 instance accessing the internet:

```text
Private EC2
    ↓
Private Subnet
    ↓
Route Table
    ↓
NAT Gateway
    ↓
Internet Gateway
    ↓
Internet
```

If outbound connectivity fails, inspect each layer.

---

# 39. Security Groups

Security Groups control network traffic for AWS resources.

Example:

```text
Client
   ↓
Security Group
   ↓
EC2
```

The application may be completely healthy but unreachable because the required traffic is not allowed.

Always separate:

```text
Application failure
```

from:

```text
Network access failure
```

---

# 40. Network ACLs

Network ACLs operate at the subnet level.

Example:

```text
Internet
   ↓
Network ACL
   ↓
Subnet
   ↓
Security Group
   ↓
EC2
```

Incorrect NACL rules can cause:

```text
Connection timeout
Unexpected connection failures
Asymmetric traffic problems
```

---

# 41. Route Tables

Route tables determine where network traffic goes.

Example:

```text
Private Subnet
      ↓
0.0.0.0/0
      ↓
NAT Gateway
```

If the route is missing:

```text
Private EC2
      ↓
No valid route
      ↓
Connection timeout
```

---

# 42. NAT Gateway Monitoring

Private workloads often use NAT Gateway for outbound internet access.

Architecture:

```text
Private EC2
     ↓
Route Table
     ↓
NAT Gateway
     ↓
Internet Gateway
     ↓
Internet
```

Monitor:

```text
Bytes
Packets
Connections
Errors
Traffic patterns
```

Also monitor whether NAT traffic grows unexpectedly because of application behavior.

---

# 43. Load Balancer Network Monitoring

For ALB-based applications:

```text
Client
   ↓
ALB
   ↓
Target
   ↓
Application
```

Monitor:

```text
Request Count
Target Response Time
HTTP Errors
Target Health
Connections
```

The load balancer should be monitored together with the backend target.

---

# 44. ALB 502 Troubleshooting

A 502 can indicate a problem communicating with the backend target.

Check:

```text
ALB
 ↓
Target Health
 ↓
Target Port
 ↓
Security Group
 ↓
Application
```

Inside the EC2 instance:

```bash
ss -lntp
```

Also inspect application logs.

---

# 45. ALB 503 Troubleshooting

A 503 may occur when no healthy target is available.

Check:

```text
Target health
Target registration
Application health
Deployment status
Instance capacity
```

Do not assume every 503 is a network problem.

---

# 46. ALB 504 Troubleshooting

A 504 generally indicates that the ALB waited too long for an upstream response.

Investigate:

```text
ALB
 ↓
EC2
 ↓
Application
 ↓
Database
 ↓
External Dependency
```

Possible causes:

```text
Slow application
Slow database
External API latency
Network issue
Timeout configuration
```

---

# 47. Kubernetes Network Monitoring

In Kubernetes:

```text
Client
  ↓
ALB
  ↓
Ingress
  ↓
Service
  ↓
Pod
  ↓
Application
```

Network troubleshooting must identify which layer is failing.

Useful commands:

```bash
kubectl get svc
kubectl get endpoints
kubectl get pods -o wide
```

---

# 48. Kubernetes Service Connectivity

If a Service is unavailable:

```text
Service
   ↓
Endpoints
   ↓
Pods
```

Check:

```bash
kubectl get svc
kubectl get endpoints
```

If there are no endpoints, investigate:

```text
Service selector
Pod labels
Pod readiness
Deployment
Pod health
```

The problem may not be the network itself.

---

# 49. Network Monitoring and Containers

Containerized applications involve multiple networking layers:

```text
Container Network
      ↓
Pod Network
      ↓
Node Network
      ↓
Service
      ↓
Load Balancer
```

Therefore troubleshooting must determine the exact layer where communication breaks.

---

# 50. Network Monitoring With Prometheus

For Linux and EC2 infrastructure, Node Exporter can expose network metrics to Prometheus.

Architecture:

```text
EC2 / Linux
     ↓
Node Exporter
     ↓
Prometheus
     ↓
Grafana
```

Monitor:

```text
Network Bytes
Packets
Errors
Drops
Interface Statistics
```

For multiple instances:

```text
EC2-1 ── Node Exporter ──┐
EC2-2 ── Node Exporter ──┤
EC2-3 ── Node Exporter ──┼──→ Prometheus
EC2-4 ── Node Exporter ──┤
EC2-5 ── Node Exporter ──┘
                           ↓
                        Grafana
```

---

# 51. Network Dashboard

A useful production dashboard should contain:

```text
┌────────────────────────────────────────────┐
│             NETWORK OVERVIEW               │
├────────────────────────────────────────────┤
│ Network Throughput                         │
├────────────────────────────────────────────┤
│ RX Traffic                                 │
├────────────────────────────────────────────┤
│ TX Traffic                                 │
├────────────────────────────────────────────┤
│ Packet Rate                                │
├────────────────────────────────────────────┤
│ RX Errors / TX Errors                      │
├────────────────────────────────────────────┤
│ RX Drops / TX Drops                        │
├────────────────────────────────────────────┤
│ Established Connections                    │
├────────────────────────────────────────────┤
│ TIME_WAIT / CLOSE_WAIT                     │
├────────────────────────────────────────────┤
│ Network Latency                            │
├────────────────────────────────────────────┤
│ Packet Loss                                │
└────────────────────────────────────────────┘
```

---

# 52. Network Alerts

Useful alerts include:

```text
High packet loss
High network latency
High interface utilization
Network errors
Network drops
Unexpected connection growth
High TCP connection count
DNS failures
Service unreachable
```

Alerts should be based on actual production behavior rather than arbitrary thresholds.

---

# 53. High Network Latency Troubleshooting

Scenario:

```text
Application latency increased
```

Check:

```text
1. DNS latency
2. TCP connection time
3. TLS handshake
4. Network RTT
5. Application processing
6. Database latency
7. External dependencies
```

Commands:

```bash
ping <host>
curl -v <url>
traceroute <host>
ss -s
```

Then correlate the results with application metrics.

---

# 54. Packet Loss Troubleshooting

Scenario:

```text
Packet Loss = 5%
```

Troubleshooting:

```text
Source
  ↓
Interface
  ↓
Network Path
  ↓
Destination
  ↓
Application
```

Commands:

```bash
ip -s link
ping <destination>
traceroute <destination>
```

Then correlate with infrastructure metrics.

---

# 55. Connection Exhaustion

Suppose:

```text
Application requests are failing
```

Check:

```text
TCP connections
File descriptors
Ephemeral ports
Connection pools
```

Commands:

```bash
ss -s
ulimit -n
```

Possible flow:

```text
Connections ↑
      ↓
File Descriptors ↑
      ↓
Limit Reached
      ↓
New Connections Fail
```

---

# 56. Ephemeral Port Exhaustion

Clients use ephemeral ports for outbound TCP connections.

If an application continuously creates short-lived connections:

```text
Connection Creation ↑
       ↓
Ephemeral Ports Consumed
       ↓
TIME_WAIT ↑
       ↓
New Connections Fail
```

Investigate:

```text
Connection reuse
Connection pools
TIME_WAIT
Destination patterns
Application configuration
```

---

# 57. Network Monitoring During Deployment

Before deployment:

```text
Traffic        → Normal
Latency        → Normal
Errors         → Normal
Connections    → Normal
```

After deployment:

```text
Connections ↑
Latency ↑
Errors ↑
```

Investigate:

```text
New application behavior
Connection pooling
New dependencies
Configuration changes
Network policy
Service ports
```

---

# 58. Network Monitoring During Scaling

When traffic increases:

```text
Traffic ↑
   ↓
Connections ↑
   ↓
Bandwidth ↑
   ↓
Application Load ↑
```

Monitor:

```text
Network throughput
Connections
Latency
Errors
Target health
Instance capacity
```

---

# 59. Network Monitoring and Dependencies

A service can be healthy while its dependency is unavailable.

Example:

```text
Order Service
      ↓
Payment API
      X
```

The Order Service may show:

```text
CPU       → Normal
Memory    → Normal
```

while requests fail because:

```text
Payment API → Timeout
```

Logs and distributed tracing help identify the failing dependency.

---

# 60. Network Troubleshooting Framework

Use this sequence during incidents:

```text
1. Is the destination reachable?
        ↓
2. Does DNS resolve?
        ↓
3. Is there a valid route?
        ↓
4. Is the port listening?
        ↓
5. Is traffic allowed?
        ↓
6. Is TCP connection successful?
        ↓
7. Is TLS successful?
        ↓
8. Is HTTP successful?
        ↓
9. Is the application healthy?
        ↓
10. Are dependencies healthy?
```

This provides a structured troubleshooting approach instead of random command execution.

---

# 61. Example: EC2 Cannot Reach Internet

Scenario:

```text
Private EC2
      ↓
curl https://example.com
      ↓
Timeout
```

Check:

```text
Subnet
 ↓
Route Table
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Security Group
 ↓
NACL
 ↓
DNS
```

Commands:

```bash
ip route
nslookup example.com
curl -v https://example.com
```

---

# 62. Example: EC2 Cannot Reach Database

Scenario:

```text
Application
     ↓
Database
     X
```

Check:

```text
DNS
Route
Security Group
NACL
Database Port
Database Availability
Application Configuration
```

Test:

```bash
nc -vz <database-host> <port>
```

---

# 63. Example: ALB Cannot Reach EC2

Architecture:

```text
Client
  ↓
ALB
  ↓
EC2
```

Check:

```text
Target Health
EC2 Application
Listening Port
Security Group
NACL
Route
```

Inside EC2:

```bash
ss -lntp
```

---

# 64. Example: High Connection Count

Scenario:

```text
ESTABLISHED Connections ↑
```

Investigate:

```text
Traffic Increase?
Connection Leak?
Slow Backend?
Long Keep-Alive?
Client Behavior?
Database Connections?
```

Correlate with:

```text
CPU
Memory
Application Latency
Logs
Traces
```

---

# 65. Network Monitoring With ELK

Network-related failures can also be investigated through centralized logs.

Architecture:

```text
EC2 / Application
       ↓
Log Collector
       ↓
Logstash
       ↓
Elasticsearch
       ↓
Kibana
```

Search for:

```text
connection timeout
connection refused
DNS error
upstream timeout
502
503
504
connection reset
network error
```

Logs should be correlated with the exact incident time.

---

# 66. Network Monitoring and Tracing

For distributed applications:

```text
Client
  ↓
Service A
  ↓
Service B
  ↓
Database
```

Tracing can show where latency is introduced.

Example:

```text
Service A
  ├── 20 ms
  ↓
Service B
  ├── 800 ms
  ↓
Database
  └── 20 ms
```

This indicates that Service B or its communication path requires investigation.

---

# 67. Production Network Monitoring Architecture

```text
                         USERS
                           │
                           ↓
                          DNS
                           │
                           ↓
                    LOAD BALANCER
                           │
                           ↓
                    ┌──────────────┐
                    │ Application  │
                    └──────┬───────┘
                           │
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
           EC2            EKS         External API
             │             │
             │             ↓
             │          Service
             │             │
             │            Pod
             │
             └─────────────┬─────────────┘
                           ↓
                       Database
                           │
                           ↓
                    Network Metrics
                           │
                  ┌────────┴────────┐
                  ↓                 ↓
              Prometheus           ELK
                  ↓                 ↓
               Grafana            Kibana
                  │
                  ↓
              Alerts
```

---

# 68. Network Monitoring Best Practices

```text
1. Monitor bandwidth.
2. Monitor throughput.
3. Monitor latency.
4. Monitor packet loss.
5. Monitor packet errors.
6. Monitor packet drops.
7. Monitor TCP connections.
8. Monitor TCP states.
9. Monitor DNS performance.
10. Monitor listening ports.
11. Monitor routing.
12. Monitor network interfaces.
13. Monitor AWS networking components.
14. Monitor load balancers.
15. Monitor application connectivity.
16. Monitor dependency connectivity.
17. Correlate network metrics with application metrics.
18. Use centralized logging for failures.
19. Use tracing for distributed requests.
20. Create actionable alerts.
21. Monitor trends instead of only current values.
22. Test network failure scenarios.
23. Monitor critical communication paths.
24. Keep network dashboards aligned with production architecture.
25. Validate alerts periodically.
```

---

# 69. Common Network Monitoring Mistakes

### Mistake 1: Checking Only Ping

```text
ping works
```

does not prove:

```text
HTTPS works
Database works
Application works
```

### Mistake 2: Ignoring DNS

DNS failures can look like application failures.

### Mistake 3: Ignoring TCP States

Connection leaks can eventually exhaust resources.

### Mistake 4: Ignoring Packet Drops

Packet drops can result in:

```text
Retries
Latency
Throughput degradation
Timeouts
```

### Mistake 5: Checking Only EC2

The problem may exist in:

```text
ALB
Route Table
NAT Gateway
Security Group
NACL
DNS
```

### Mistake 6: Assuming Every 5xx Is an Application Bug

A 5xx response may be caused by:

```text
Load Balancer
Network
Target Health
Application
Database
External Dependency
```

### Mistake 7: Looking at Only One Metric

Network incidents should be investigated using:

```text
Metrics
+
Logs
+
Traces
```

---

# 70. Production Network Monitoring Checklist

```text
NETWORK
├── Availability
├── Bandwidth
├── Throughput
├── Latency
├── Packet Loss
├── Packet Errors
├── Packet Drops
├── TCP Connections
├── TCP States
├── DNS
├── Routing
└── Interfaces

AWS
├── VPC
├── Subnets
├── Route Tables
├── Security Groups
├── Network ACLs
├── NAT Gateway
├── Load Balancer
└── EC2

APPLICATION
├── HTTP Status
├── Response Time
├── Connection Pools
├── Dependencies
└── Health

OBSERVABILITY
├── Node Exporter
├── Prometheus
├── Grafana
├── ELK
├── OpenTelemetry
└── Jaeger
```

---

# 71. Commands Cheat Sheet

## Network Interfaces

```bash
ip link
ip addr
ip -s link
```

## Routing

```bash
ip route
```

## Connectivity

```bash
ping <host>
nc -vz <host> <port>
```

## DNS

```bash
nslookup <host>
dig <host>
dig <host> A
dig <host> +trace
```

## TCP

```bash
ss -lntp
ss -ant
ss -s
ss -ant state established
ss -ant state close-wait
```

## HTTP

```bash
curl -I <url>
curl -v <url>
```

## TLS

```bash
openssl s_client -connect <host>:443
```

## Network Path

```bash
traceroute <host>
tracepath <host>
```

---

# 72. Interview Question

## How do you monitor network health in production?

**Answer:**

I monitor network throughput, bandwidth utilization, latency, packet loss, packet errors, packet drops, TCP connections, DNS performance, and interface statistics.

In AWS, I also monitor the complete VPC path including:

```text
Route Tables
Security Groups
Network ACLs
NAT Gateway
Load Balancers
EC2
```

I correlate network metrics with application metrics, centralized logs, and traces to determine whether the issue is caused by networking, the application, or a downstream dependency.

---

# 73. Interview Question

## How would you troubleshoot an application that cannot connect to another server?

**Answer:**

I would troubleshoot layer by layer.

First I would verify DNS resolution, then routing, connectivity, the destination port, Security Groups/NACLs or firewall rules, and whether the destination service is listening.

I would use:

```bash
nslookup
dig
ip route
ping
nc
ss
curl -v
```

For HTTP services, I would also inspect the HTTP response and TLS negotiation.

Finally, I would check application logs and downstream dependencies.

---

# 74. Interview Question

## What is the difference between connection refused and connection timeout?

**Answer:**

A connection refused response generally means the destination was reachable but the connection was actively rejected. A common reason is that no service is listening on the requested port.

A timeout means the connection attempt did not receive a response within the expected period. Possible causes include:

```text
Routing problem
Security Group
NACL
Firewall
Network failure
Unavailable destination
```

---

# 75. Interview Question

## How would you troubleshoot a 504 from an ALB?

**Answer:**

I would first determine whether the ALB can reach the target and whether the target is healthy.

Then I would check:

```text
Target Health
Target Port
Security Groups
Application Response Time
Application Logs
Database Latency
External API Latency
```

I would also use distributed tracing to determine whether the application is waiting on a downstream dependency.

Finally, I would compare the configured timeout values with the actual response time.

---

# 76. Interview Question

## How would you troubleshoot high network latency?

**Answer:**

I would determine which part of the request is contributing to the latency.

I would investigate:

```text
DNS
TCP
TLS
Network Path
Application
Database
External Dependencies
```

I would use:

```bash
ping
traceroute
tracepath
curl -v
ss
```

and correlate the results with Prometheus/Grafana metrics, logs, and traces.

---

# 77. Interview Question

## How do you monitor network connections on Linux?

**Answer:**

I use `ss` to inspect listening and established connections and analyze TCP states such as:

```text
ESTABLISHED
TIME_WAIT
CLOSE_WAIT
SYN-SENT
SYN-RECV
```

I monitor connection counts over time and correlate unusual increases with:

```text
Application Traffic
Connection Pools
File Descriptors
Dependencies
```

---

# 78. Interview Question

## What would you check if an EC2 instance can access some services but not another?

**Answer:**

I would compare the network paths and ports.

I would verify:

```text
DNS
Routing
Destination Port
Security Groups
NACLs
Firewall
Service Listening State
VPC Connectivity
```

If the destination is an AWS service or private service, I would also verify the relevant VPC routing and endpoint configuration.

---

# 79. Interview Question

## How do network metrics help during an application incident?

**Answer:**

Network metrics help determine whether an application problem is caused by:

```text
Connectivity
Congestion
Latency
Packet Loss
Network Errors
Dependency Communication
```

For example, if application latency increases together with network latency and packet drops, networking becomes a strong candidate for investigation.

I would then correlate the network behavior with logs and traces to identify the exact affected component.

---

# 80. Interview Question

## How would you troubleshoot an EC2 instance that cannot access the internet?

**Answer:**

I would first identify whether the EC2 instance is public or private.

For a private instance, I would verify:

```text
Private Subnet
   ↓
Route Table
   ↓
NAT Gateway
   ↓
Internet Gateway
   ↓
Internet
```

Then I would check:

```text
Security Group
NACL
Route
NAT Gateway
DNS
```

I would use:

```bash
ip route
nslookup example.com
curl -v https://example.com
```

---

# 81. Interview Question

## How would you troubleshoot a network problem after a deployment?

**Answer:**

I would compare the network behavior before and after deployment.

I would check:

```text
Latency
Connections
Packet Loss
Errors
Traffic
Ports
Service Health
```

Then I would review:

```text
Recent configuration changes
New dependencies
New ports
Security rules
Application connection pools
Load Balancer configuration
```

I would correlate the network metrics with application logs and traces to determine whether the deployment introduced the problem.

---

# 82. Production Incident Example

## Scenario

Users report:

```text
Application is very slow.
```

### Step 1 — Check Application

```text
Latency ↑
Error Rate ↑
```

### Step 2 — Check Network

```text
Network Latency ↑
Packet Loss ↑
```

### Step 3 — Check Path

```text
Client
 ↓
ALB
 ↓
EC2
 ↓
Database
```

### Step 4 — Identify Dependency

Tracing shows:

```text
Application
    ↓
Database
    ↓
High latency
```

### Step 5 — Root Cause

The application itself is healthy, but the database/network path is causing the delay.

### Lesson

```text
Application Symptoms
        ↓
Network Investigation
        ↓
Dependency Investigation
        ↓
Root Cause
```

---

# 83. Production Incident Example

## Scenario

Users receive:

```text
504 Gateway Timeout
```

Troubleshooting:

```text
Client
  ↓
ALB
  ↓
Target
  ↓
Application
  ↓
Database
```

Check:

```text
1. Target healthy?
2. Application listening?
3. Security Group correct?
4. Network path available?
5. Application response time?
6. Database latency?
7. External dependency latency?
8. Timeout configuration?
```

Possible root cause:

```text
Application
     ↓
Database query
     ↓
Slow response
     ↓
ALB timeout
     ↓
504
```

The 504 is the symptom; the database latency may be the root cause.

---

# 84. Production Incident Example

## Scenario

An application cannot connect to another EC2 instance.

Start with:

```text
DNS
 ↓
Route
 ↓
Security Group
 ↓
NACL
 ↓
TCP Port
 ↓
Application
```

Commands:

```bash
nslookup <host>
ip route
ping <host>
nc -vz <host> <port>
```

On the destination:

```bash
ss -lntp
```

Then check application logs.

This prevents random changes and provides a structured troubleshooting path.

---

# 85. Final Mental Model

```text
                           CLIENT
                              │
                              ↓
                             DNS
                              │
                              ↓
                         LOAD BALANCER
                              │
                              ↓
                         NETWORK PATH
                              │
             ┌────────────────┼────────────────┐
             ↓                ↓                ↓
          ROUTING         SECURITY         CONNECTIVITY
             │                │                │
        Route Table       SG / NACL          TCP
             │                │                │
             └────────────────┼────────────────┘
                              ↓
                             EC2
                              │
                    ┌─────────┼─────────┐
                    ↓         ↓         ↓
                  PORT      SERVICE   APPLICATION
                    │         │         │
                    └─────────┼─────────┘
                              ↓
                       DEPENDENCIES
                    ┌─────────┼─────────┐
                    ↓         ↓         ↓
                 DATABASE    CACHE    EXTERNAL API
                              │
                              ↓
                        OBSERVABILITY
                              │
              ┌───────────────┼───────────────┐
              ↓               ↓               ↓
           METRICS           LOGS           TRACES
              ↓               ↓               ↓
          Prometheus          ELK        OpenTelemetry
              ↓               ↓               ↓
           Grafana          Kibana          Jaeger
                              │
                              ↓
                         ROOT CAUSE
                              │
                              ↓
                         REMEDIATION
                              │
                              ↓
                      VALIDATE RECOVERY
```

**Key Principle:** Network monitoring is not simply checking whether a server responds to `ping`.

A production DevOps engineer should understand the complete communication path:

```text
DNS
 ↓
Routing
 ↓
Security
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
Application
 ↓
Dependencies
```

And correlate:

```text
Metrics
+
Logs
+
Traces
=
Faster Root Cause Analysis
```
````
