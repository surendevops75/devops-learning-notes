# Network-Debugging-Commands

## 1. Purpose

This file is a production-oriented command reference for diagnosing networking problems across Linux, AWS, Kubernetes and EKS.

The objective is not to memorize commands independently. The objective is to use commands in a structured flow:

```text
DNS
 ↓
IP
 ↓
Route
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
Load Balancer
 ↓
Kubernetes Service
 ↓
EndpointSlice
 ↓
Pod
 ↓
Application
```

---

## 2. Production Debugging Philosophy

A strong DevOps engineer does not immediately restart Pods or modify Security Groups.

First:

```text
Observe
↓
Form hypothesis
↓
Run targeted command
↓
Collect evidence
↓
Identify failing layer
↓
Make smallest safe change
↓
Verify
```

---

## 3. First Question

Always ask:

```text
What exactly is failing?
```

Examples:

```text
DNS resolution?
TCP connection?
TLS handshake?
HTTP response?
Application response?
Pod-to-Pod communication?
External-to-EKS traffic?
```

---

## 4. Basic Debugging Layers

```text
Layer 1: DNS
Layer 2: IP
Layer 3: Routing
Layer 4: TCP/UDP
Layer 5: TLS
Layer 6: HTTP
Layer 7: Application
```

---

## 5. Core Command Toolkit

The commands you should be comfortable with:

```bash
ip
ss
ping
traceroute
tracepath
dig
nslookup
host
curl
wget
nc
telnet
openssl
tcpdump
nmap
arp
ip neigh
ethtool
route
iptables
nft
conntrack
```

Not every production host will contain every command.

---

## 6. Check Interface Configuration

```bash
ip addr
```

or:

```bash
ip a
```

Use it to inspect:

```text
interfaces
IPv4 addresses
IPv6 addresses
prefixes
interface state
```

---

## 7. Show One Interface

```bash
ip addr show eth0
```

---

## 8. Show Link State

```bash
ip link
```

Look for:

```text
UP
DOWN
LOWER_UP
```

---

## 9. Bring Interface Information Into Context

```bash
ip -br addr
```

Useful compact output:

```text
interface
state
address
```

---

## 10. Check Default Route

```bash
ip route
```

Look for:

```text
default via <gateway> dev <interface>
```

---

## 11. Show Routing Table

```bash
ip route show
```

---

## 12. Show Route to Destination

One of the most useful commands:

```bash
ip route get 8.8.8.8
```

It can show which:

```text
interface
source address
gateway
```

Linux intends to use.

---

## 13. Why `ip route get` Matters

Instead of asking:

```text
"Which route should Linux use?"
```

ask Linux directly:

```bash
ip route get <destination>
```

---

## 14. Check Local Address

```bash
hostname -I
```

---

## 15. Check Hostname

```bash
hostname
```

---

## 16. Check FQDN

```bash
hostname -f
```

---

## 17. Check Hosts File

```bash
cat /etc/hosts
```

---

## 18. Check Resolver Configuration

```bash
cat /etc/resolv.conf
```

Look for:

```text
nameserver
search
options
```

---

## 19. Check DNS With `dig`

```bash
dig example.com
```

---

## 20. Short DNS Output

```bash
dig +short example.com
```

---

## 21. Query A Record

```bash
dig example.com A
```

---

## 22. Query AAAA Record

```bash
dig example.com AAAA
```

---

## 23. Query CNAME

```bash
dig example.com CNAME
```

---

## 24. Query MX

```bash
dig example.com MX
```

---

## 25. Query TXT

```bash
dig example.com TXT
```

---

## 26. Query Specific DNS Server

```bash
dig @8.8.8.8 example.com
```

---

## 27. Compare DNS Servers

```bash
dig @8.8.8.8 example.com
dig @1.1.1.1 example.com
```

Useful when investigating resolver-specific problems.

---

## 28. DNS Trace

```bash
dig +trace example.com
```

Useful for understanding the delegation chain.

---

## 29. Check DNS Authority

```bash
dig example.com NS
```

---

## 30. Reverse DNS

```bash
dig -x 10.0.1.10
```

---

## 31. `nslookup`

```bash
nslookup example.com
```

Useful when `dig` is unavailable.

---

## 32. `host`

```bash
host example.com
```

---

## 33. Compare DNS Tools

```text
dig:
detailed DNS troubleshooting

nslookup:
simple interactive/query utility

host:
compact DNS lookup
```

---

## 34. Check Resolver Reachability

Find the configured resolver:

```bash
cat /etc/resolv.conf
```

Then test the resolver using:

```bash
dig @<dns-server> example.com
```

---

## 35. DNS Failure Categories

```text
NXDOMAIN
SERVFAIL
timeout
wrong record
stale record
wrong resolver
DNSSEC issue
delegation issue
```

---

## 36. NXDOMAIN

Means the queried DNS name does not exist from the responding DNS perspective.

---

## 37. SERVFAIL

Indicates the resolver failed to successfully answer the query.

Investigate the DNS chain and authoritative configuration.

---

## 38. DNS Timeout

Possible causes:

```text
network
firewall
resolver unavailable
UDP/TCP DNS blocked
```

---

## 39. DNS Over TCP

DNS can use TCP as well as UDP.

Test TCP:

```bash
dig +tcp example.com
```

---

## 40. DNS Query Size

Large DNS responses may trigger TCP fallback depending on conditions.

---

## 41. Check Listening Ports

```bash
ss -lntup
```

Shows listening TCP/UDP sockets where permissions permit process information.

---

## 42. TCP Listening Ports

```bash
ss -lnt
```

---

## 43. UDP Listening Ports

```bash
ss -lnu
```

---

## 44. Established Connections

```bash
ss -nt
```

---

## 45. All TCP Connections

```bash
ss -ant
```

---

## 46. Process Information

```bash
ss -lntp
```

May require root privileges to show process ownership.

---

## 47. Filter Port

```bash
ss -lntp | grep ':8080'
```

---

## 48. Count Connections

```bash
ss -ant | wc -l
```

---

## 49. Connection State Summary

```bash
ss -s
```

Useful for identifying unusually high:

```text
TIME-WAIT
ESTABLISHED
SYN-RECV
```

counts.

---

## 50. TIME_WAIT

A normal TCP state after connection closure.

Large numbers may be expected for high connection churn.

Do not automatically treat TIME_WAIT as a failure.

---

## 51. SYN-RECV

Can indicate connections waiting for completion of the handshake.

A sudden unusual increase can warrant investigation.

---

## 52. ESTABLISHED

Represents active TCP connections.

---

## 53. FIN-WAIT

Represents TCP connection shutdown stages.

---

## 54. Listening Port Verification

If the application should listen on 8080:

```bash
ss -lntp | grep ':8080'
```

---

## 55. Application Binding Problem

If the process listens on:

```text
127.0.0.1:8080
```

a remote client cannot normally reach it through the host's external interface.

---

## 56. Correct Server Binding

A server intended to accept remote traffic may need to bind to:

```text
0.0.0.0:8080
```

or an appropriate specific interface/address.

---

## 57. IPv6 Binding

A service may listen on:

```text
[::]:8080
```

depending on OS/socket configuration.

---

## 58. Check Port With `nc`

```bash
nc -vz example.com 443
```

This tests basic TCP connectivity.

---

## 59. Netcat Timeout

```bash
nc -vz -w 5 example.com 443
```

---

## 60. UDP With Netcat

UDP testing requires understanding that lack of a response does not necessarily prove the port is closed.

---

## 61. Telnet

```bash
telnet example.com 80
```

Useful for basic TCP connectivity when available.

---

## 62. Telnet Limitation

Telnet is not a proper TLS/HTTP diagnostic tool.

Use `openssl` and `curl` for HTTPS.

---

## 63. Ping

```bash
ping 10.0.0.1
```

Tests ICMP reachability where ICMP is permitted.

---

## 64. Ping Limitation

A failed ping does not prove that TCP/HTTPS is unavailable.

ICMP can be intentionally blocked.

---

## 65. Ping With Count

```bash
ping -c 4 10.0.0.1
```

---

## 66. Ping IPv6

```bash
ping6 2001:db8::1
```

or:

```bash
ping -6 2001:db8::1
```

---

## 67. Packet Loss

Observe:

```text
packet loss
latency
variance
```

---

## 68. Traceroute

```bash
traceroute example.com
```

Shows a path using traceroute probes.

---

## 69. TCP Traceroute

```bash
traceroute -T -p 443 example.com
```

Useful when ICMP/UDP probes are filtered.

---

## 70. Tracepath

```bash
tracepath example.com
```

Can help inspect path and PMTU information.

---

## 71. MTR

If installed:

```bash
mtr example.com
```

Combines path and repeated probe information.

---

## 72. MTR Report

```bash
mtr -rw example.com
```

Useful for a report-style view.

---

## 73. Traceroute Limitation

A hop that does not respond does not automatically mean traffic is broken.

Routers may rate-limit or filter traceroute probes.

---

## 74. Packet Capture

Use:

```bash
tcpdump
```

when higher-level tools do not explain the behavior.

---

## 75. Capture on Interface

```bash
sudo tcpdump -i eth0
```

---

## 76. Capture TCP Port

```bash
sudo tcpdump -i eth0 tcp port 443
```

---

## 77. Capture Host

```bash
sudo tcpdump -i eth0 host 10.0.1.10
```

---

## 78. Capture Source

```bash
sudo tcpdump -i eth0 src host 10.0.1.10
```

---

## 79. Capture Destination

```bash
sudo tcpdump -i eth0 dst host 10.0.1.20
```

---

## 80. Capture SYN Packets

```bash
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'
```

---

## 81. Capture TCP Handshake

```bash
sudo tcpdump -i eth0 \
  'tcp[tcpflags] & (tcp-syn|tcp-ack) != 0'
```

---

## 82. Write Capture to File

```bash
sudo tcpdump -i eth0 -w capture.pcap
```

---

## 83. Read Capture

```bash
tcpdump -r capture.pcap
```

---

## 84. Read With Wireshark

A `.pcap` file can be inspected in Wireshark when available.

---

## 85. Production Packet Capture Caution

Packet captures may contain:

```text
IP addresses
headers
metadata
possibly sensitive application data
```

Use authorization and appropriate handling.

---

## 86. TCP Retransmissions

A packet capture can reveal repeated TCP segments indicating retransmission.

---

## 87. TCP Reset

Look for:

```text
RST
```

which can indicate connection rejection/termination depending on context.

---

## 88. SYN Without SYN-ACK

If a client repeatedly sends:

```text
SYN
SYN
SYN
```

without receiving SYN-ACK, investigate:

```text
route
firewall
security group
NACL
server listener
```

---

## 89. SYN-ACK Without ACK

May indicate return-path or client-side problems.

Use packet capture at multiple points if possible.

---

## 90. TLS Debugging

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com
```

---

## 91. TLS Certificate

Inspect:

```text
subject
issuer
validity
SAN
chain
```

---

## 92. Certificate Dates

```bash
openssl s_client -connect example.com:443 \
  -servername example.com </dev/null 2>/dev/null \
  | openssl x509 -noout -dates
```

---

## 93. Certificate Subject

```bash
openssl s_client -connect example.com:443 \
  -servername example.com </dev/null 2>/dev/null \
  | openssl x509 -noout -subject
```

---

## 94. Certificate SAN

```bash
openssl s_client -connect example.com:443 \
  -servername example.com </dev/null 2>/dev/null \
  | openssl x509 -noout -ext subjectAltName
```

---

## 95. TLS Protocol Test

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  -tls1_2
```

---

## 96. TLS 1.3 Test

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  -tls1_3
```

---

## 97. TLS SNI Importance

Without the expected SNI hostname, a shared TLS endpoint may return a different certificate or behavior.

---

## 98. Curl DNS Test

```bash
curl -v https://example.com
```

---

## 99. Curl Headers

```bash
curl -I https://example.com
```

---

## 100. Curl Redirects

```bash
curl -IL https://example.com
```

---

## 101. Curl Timing

```bash
curl -o /dev/null -s -w \
'DNS:%{time_namelookup}\nConnect:%{time_connect}\nTLS:%{time_appconnect}\nTTFB:%{time_starttransfer}\nTotal:%{time_total}\n' \
https://example.com
```

---

## 102. Curl Verbose Mode

```bash
curl -v https://example.com
```

Useful for:

```text
DNS
TCP
TLS
HTTP
redirects
headers
```

---

## 103. Curl Insecure Option

```bash
curl -k https://example.com
```

Use only for controlled troubleshooting.

Do not use `-k` as a production security solution.

---

## 104. Curl Custom Host Header

```bash
curl -H 'Host: api.example.com' \
  https://<endpoint>/
```

Useful for host-based routing tests when appropriate.

---

## 105. Curl Resolve

One of the best host-routing tests:

```bash
curl --resolve api.example.com:443:<IP> \
  https://api.example.com/
```

It forces the connection to the specified IP while preserving hostname/SNI behavior.

---

## 106. Curl Interface

```bash
curl --interface eth0 https://example.com
```

Useful on hosts with multiple interfaces.

---

## 107. Curl Source IP

Where supported:

```bash
curl --interface 10.0.1.20 https://example.com
```

---

## 108. Curl Proxy Bypass

```bash
curl --noproxy '*' https://example.com
```

Useful when proxy environment variables interfere with testing.

---

## 109. Check Proxy Environment

```bash
env | grep -i proxy
```

---

## 110. Common Proxy Variables

```text
HTTP_PROXY
HTTPS_PROXY
ALL_PROXY
NO_PROXY
```

---

## 111. NO_PROXY in Kubernetes

Incorrect `NO_PROXY` configuration can cause internal traffic to go through an external proxy.

---

## 112. Kubernetes DNS

Check:

```bash
kubectl get pods -n kube-system
```

Look for CoreDNS Pods.

---

## 113. CoreDNS Service

```bash
kubectl get svc -n kube-system kube-dns
```

---

## 114. CoreDNS Endpoints

```bash
kubectl get endpointslice \
  -n kube-system \
  -l k8s-app=kube-dns
```

---

## 115. CoreDNS Logs

```bash
kubectl logs -n kube-system \
  -l k8s-app=kube-dns
```

Label selectors can vary by installation.

---

## 116. Kubernetes DNS Test Pod

Use a temporary diagnostic Pod with DNS tools:

```bash
kubectl run dns-test \
  --image=registry.k8s.io/e2e-test-images/dnsutils:1.3 \
  --rm -it --restart=Never -- sh
```

Verify the image/version against your cluster policy before production use.

---

## 117. DNS Inside Pod

```bash
nslookup kubernetes.default
```

---

## 118. DNS Service Name

```bash
nslookup frontend.roboshop.svc.cluster.local
```

---

## 119. Kubernetes Service Lookup

```bash
getent hosts frontend.roboshop.svc.cluster.local
```

---

## 120. Kubernetes DNS Search Domains

Inside a Pod:

```bash
cat /etc/resolv.conf
```

---

## 121. Kubernetes DNS Configuration

Typical:

```text
search <namespace>.svc.cluster.local svc.cluster.local cluster.local
nameserver <cluster-DNS-IP>
```

Exact values depend on cluster configuration.

---

## 122. Service Debugging

```bash
kubectl get svc -n roboshop
```

---

## 123. Service Details

```bash
kubectl describe svc frontend -n roboshop
```

Check:

```text
selector
port
targetPort
type
endpoints
```

---

## 124. EndpointSlice Debugging

```bash
kubectl get endpointslice \
  -n roboshop
```

---

## 125. Detailed EndpointSlice

```bash
kubectl describe endpointslice \
  -n roboshop
```

---

## 126. No Endpoints

If a Service has no endpoints:

```text
selector mismatch
Pods not ready
labels wrong
```

are common causes.

---

## 127. Verify Pod Labels

```bash
kubectl get pods \
  -n roboshop \
  --show-labels
```

---

## 128. Verify Service Selector

```bash
kubectl get svc frontend \
  -n roboshop \
  -o yaml
```

---

## 129. Compare Selector and Labels

Example:

```text
Service selector:
app=frontend

Pod:
app=frontend
```

They must match according to the Service selector.

---

## 130. Pod IPs

```bash
kubectl get pods -n roboshop -o wide
```

---

## 131. Test Pod-to-Pod

From a diagnostic Pod:

```bash
curl http://<pod-ip>:8080
```

Only use HTTP if the application actually speaks HTTP.

---

## 132. Test Service DNS

```bash
curl http://frontend.roboshop.svc.cluster.local
```

---

## 133. Test Service Port

```bash
nc -vz frontend.roboshop.svc.cluster.local 80
```

---

## 134. Test Service IP

```bash
kubectl get svc frontend -n roboshop
```

Then:

```bash
nc -vz <cluster-ip> 80
```

---

## 135. Test Target Port

Find the Pod IP and actual listening port:

```bash
kubectl get pods -n roboshop -o wide
ss -lnt
```

---

## 136. Service Port vs TargetPort

Example:

```yaml
ports:
  - port: 80
    targetPort: 8080
```

Client uses:

```text
Service:80
```

Pod receives:

```text
8080
```

---

## 137. Common Service Failure

Application listens on:

```text
8080
```

but Service targets:

```text
8081
```

Result:

```text
connection failure
```

---

## 138. Kubernetes NetworkPolicy

List policies:

```bash
kubectl get networkpolicy -A
```

---

## 139. Describe NetworkPolicy

```bash
kubectl describe networkpolicy \
  <name> \
  -n <namespace>
```

---

## 140. NetworkPolicy Debugging

Confirm:

```text
source namespace
source Pod labels
destination namespace
destination Pod labels
port
protocol
policy direction
```

---

## 141. Default Deny

A policy such as:

```yaml
podSelector: {}
policyTypes:
  - Ingress
```

can deny ingress unless additional allow rules exist.

---

## 142. Test After NetworkPolicy Change

Immediately test:

```bash
nc -vz <service> <port>
```

or:

```bash
curl -v http://<service>:<port>
```

from an authorized test Pod.

---

## 143. Kubernetes Ingress

```bash
kubectl get ingress -A
```

---

## 144. Ingress Details

```bash
kubectl describe ingress \
  <name> \
  -n <namespace>
```

---

## 145. Ingress Address

```bash
kubectl get ingress \
  <name> \
  -n <namespace> \
  -o wide
```

---

## 146. Ingress Events

```bash
kubectl describe ingress \
  <name> \
  -n <namespace>
```

Look at events for controller reconciliation errors.

---

## 147. AWS Load Balancer Controller Pods

```bash
kubectl get pods \
  -n kube-system \
  -l app.kubernetes.io/name=aws-load-balancer-controller
```

---

## 148. Controller Logs

```bash
kubectl logs \
  -n kube-system \
  deployment/aws-load-balancer-controller
```

---

## 149. Controller Status

```bash
kubectl get deployment \
  -n kube-system \
  aws-load-balancer-controller
```

---

## 150. Controller Events

```bash
kubectl get events \
  -n kube-system \
  --sort-by=.lastTimestamp
```

---

## 151. AWS Load Balancer Inspection

Use AWS CLI to inspect the relevant load balancer and target groups.

Example discovery:

```bash
aws elbv2 describe-load-balancers
```

---

## 152. Target Groups

```bash
aws elbv2 describe-target-groups
```

---

## 153. Target Health

```bash
aws elbv2 describe-target-health \
  --target-group-arn <target-group-arn>
```

---

## 154. List ALB Listeners

```bash
aws elbv2 describe-listeners \
  --load-balancer-arn <load-balancer-arn>
```

---

## 155. Listener Rules

```bash
aws elbv2 describe-rules \
  --listener-arn <listener-arn>
```

---

## 156. ALB Attributes

```bash
aws elbv2 describe-load-balancer-attributes \
  --load-balancer-arn <load-balancer-arn>
```

---

## 157. Target Health Investigation

Check:

```text
target ID
state
reason
description
```

---

## 158. Security Group Inspection

```bash
aws ec2 describe-security-groups \
  --group-ids <sg-id>
```

---

## 159. Network Interfaces

```bash
aws ec2 describe-network-interfaces
```

Filter results in production scripts rather than dumping everything.

---

## 160. Route Tables

```bash
aws ec2 describe-route-tables
```

---

## 161. Subnets

```bash
aws ec2 describe-subnets
```

---

## 162. VPCs

```bash
aws ec2 describe-vpcs
```

---

## 163. Internet Gateways

```bash
aws ec2 describe-internet-gateways
```

---

## 164. NAT Gateways

```bash
aws ec2 describe-nat-gateways
```

---

## 165. VPC Flow Logs

```bash
aws ec2 describe-flow-logs
```

---

## 166. Route Verification

For a workload with a known source/destination, inspect:

```text
route table
subnet
NACL
Security Group
```

rather than assuming connectivity from subnet names.

---

## 167. AWS Reachability Analyzer

Use it when supported to analyze expected connectivity paths.

---

## 168. AWS CLI Region

If commands appear empty:

```bash
aws configure get region
```

and/or explicitly provide:

```bash
--region <region>
```

---

## 169. AWS Account

Verify you are operating against the intended account:

```bash
aws sts get-caller-identity
```

---

## 170. Production Safety

Before modifying AWS networking:

```bash
aws sts get-caller-identity
```

Confirm:

```text
account
role
region
```

---

## 171. Kubernetes Context

Before debugging:

```bash
kubectl config current-context
```

---

## 172. List Contexts

```bash
kubectl config get-contexts
```

---

## 173. Production Safety

Never assume the current `kubectl` context is production-safe.

---

## 174. Namespace

```bash
kubectl config view --minify --output 'jsonpath={..namespace}'
```

If empty, the current context may default to `default`.

---

## 175. Cluster Information

```bash
kubectl cluster-info
```

---

## 176. Nodes

```bash
kubectl get nodes -o wide
```

---

## 177. Node Conditions

```bash
kubectl describe node <node>
```

Check:

```text
Ready
NetworkUnavailable
MemoryPressure
DiskPressure
PIDPressure
```

---

## 178. Node Network Addresses

```bash
kubectl get nodes \
  -o custom-columns=NAME:.metadata.name,INTERNAL-IP:.status.addresses[?(@.type=="InternalIP")].address
```

---

## 179. Pod Network Addresses

```bash
kubectl get pods \
  -A \
  -o wide
```

---

## 180. Pod Events

```bash
kubectl describe pod <pod> -n <namespace>
```

---

## 181. Pod Logs

```bash
kubectl logs <pod> -n <namespace>
```

---

## 182. Previous Container Logs

```bash
kubectl logs <pod> \
  -n <namespace> \
  --previous
```

Useful after crashes/restarts.

---

## 183. Follow Logs

```bash
kubectl logs -f <pod> -n <namespace>
```

---

## 184. Multi-Container Pod

```bash
kubectl logs <pod> \
  -n <namespace> \
  -c <container>
```

---

## 185. Execute Shell

```bash
kubectl exec -it <pod> \
  -n <namespace> \
  -- sh
```

---

## 186. Minimal Images

Some application images contain no:

```text
bash
curl
ping
dig
nc
```

Do not modify production images just for debugging.

---

## 187. Ephemeral Debug Container

Use Kubernetes-supported ephemeral containers when appropriate:

```bash
kubectl debug -it <pod> \
  --image=nicolaka/netshoot \
  --target=<container>
```

Follow your organization's approved debug-image policy.

---

## 188. Netshoot

A diagnostic image can provide tools such as:

```text
curl
dig
tcpdump
ss
ip
mtr
```

Verify image provenance and security policy before use.

---

## 189. Debug Pod

Alternative:

```bash
kubectl run net-debug \
  --image=nicolaka/netshoot \
  --rm -it --restart=Never -- bash
```

Use an approved image in controlled environments.

---

## 190. Debug From Same Namespace

Run the diagnostic Pod in the same namespace when namespace-specific DNS/policy behavior matters.

---

## 191. Debug With Matching Labels

If NetworkPolicy depends on Pod labels, a debug Pod may need appropriate labels to reproduce the expected traffic path.

---

## 192. Debug Source Matters

Testing from:

```text
your laptop
```

does not prove:

```text
Pod → Pod
```

connectivity.

---

## 193. Debug From Correct Network Location

Always test from the same network boundary as the failing workload when possible.

---

## 194. External Test

From your workstation:

```bash
curl -v https://api.example.com
```

---

## 195. Node Test

From an authorized node:

```bash
curl -v http://<pod-ip>:8080
```

---

## 196. Pod Test

From a diagnostic Pod:

```bash
curl -v http://frontend:80
```

---

## 197. Layered Test Matrix

```text
Laptop → ALB
Pod A → Service
Pod A → Pod B
Node → Pod
Pod → External API
```

Each answers a different question.

---

## 198. HTTP Status Interpretation

```text
2xx:
success

3xx:
redirect

4xx:
client/policy/request issue

5xx:
server/upstream issue
```

This is a starting point, not a complete diagnosis.

---

## 199. Curl HTTP Code

```bash
curl -s -o /dev/null \
  -w '%{http_code}\n' \
  https://example.com
```

---

## 200. Curl Remote IP

```bash
curl -s -o /dev/null \
  -w '%{remote_ip}\n' \
  https://example.com
```

---

## 201. Curl Effective URL

```bash
curl -s -o /dev/null \
  -w '%{url_effective}\n' \
  -L https://example.com
```

---

## 202. HTTP Method Test

```bash
curl -X GET https://example.com
```

Do not use destructive methods against production without authorization.

---

## 203. POST Test

Use a safe test endpoint:

```bash
curl -X POST \
  -H 'Content-Type: application/json' \
  -d '{"test":true}' \
  https://api.example.com/test
```

---

## 204. Header Inspection

```bash
curl -sv https://example.com \
  -o /dev/null
```

---

## 205. HTTP/2

```bash
curl -I --http2 https://example.com
```

Only use if the installed curl supports it.

---

## 206. IPv4 Only

```bash
curl -4 https://example.com
```

---

## 207. IPv6 Only

```bash
curl -6 https://example.com
```

---

## 208. IPv4 vs IPv6 Failure

If:

```bash
curl -4 ...
```

works but:

```bash
curl -6 ...
```

fails, investigate IPv6 DNS/routing/firewall/load-balancer configuration.

---

## 209. `getent`

```bash
getent hosts example.com
```

Useful because it uses the system name-service configuration rather than querying DNS in isolation.

---

## 210. NSS

Linux name resolution can involve:

```text
/etc/nsswitch.conf
```

Inspect:

```bash
cat /etc/nsswitch.conf
```

---

## 211. Hosts vs DNS

Typical `hosts:` configuration may include:

```text
files
dns
```

The order can affect resolution.

---

## 212. Resolver Cache

Some systems use local caching services such as:

```text
systemd-resolved
dnsmasq
nscd
```

Check the actual host configuration.

---

## 213. systemd-resolved

On systems using it:

```bash
resolvectl status
```

---

## 214. DNS Cache Flush

Do not blindly flush caches.

First identify which resolver/cache service is actually in use.

---

## 215. Linux Interface Statistics

```bash
ip -s link
```

Look for:

```text
RX errors
TX errors
dropped packets
```

---

## 216. NIC Statistics

```bash
ethtool -S eth0
```

Availability depends on driver/platform.

---

## 217. Interface Errors

Persistent RX/TX errors can indicate:

```text
driver
hardware
virtual interface
network path
```

investigation areas.

---

## 218. MTU

```bash
ip link show eth0
```

Look for:

```text
mtu 1500
```

or another configured value.

---

## 219. Path MTU

```bash
tracepath example.com
```

can help detect PMTU behavior.

---

## 220. MTU Failure Symptom

Small packets work:

```text
ping
```

but larger requests hang or fail.

Investigate:

```text
MTU
PMTUD
fragmentation
tunnels
```

---

## 221. Test Packet Size

```bash
ping -M do -s 1400 <destination>
```

Linux syntax/options can vary; adjust based on interface/IPv4/IPv6 context.

---

## 222. ARP Table

```bash
ip neigh
```

---

## 223. ARP State

Examples:

```text
REACHABLE
STALE
DELAY
FAILED
```

---

## 224. ARP Debugging

For local IPv4 connectivity, inspect:

```bash
ip neigh
```

---

## 225. IPv6 Neighbor Discovery

IPv6 uses Neighbor Discovery rather than ARP.

Inspect:

```bash
ip -6 neigh
```

---

## 226. Route Cache Misconception

Modern Linux does not use the old route-cache model in the same way as legacy systems.

Use:

```bash
ip route get
```

for effective route lookup.

---

## 227. Firewall Inspection: iptables

```bash
sudo iptables -L -n -v
```

---

## 228. NAT Rules

```bash
sudo iptables -t nat -L -n -v
```

---

## 229. nftables

Modern Linux systems may use:

```bash
sudo nft list ruleset
```

---

## 230. Do Not Assume iptables Is the Data Plane

On modern distributions and Kubernetes environments, iptables may coexist with or be backed by other mechanisms.

Understand the actual networking implementation.

---

## 231. Kubernetes kube-proxy

Depending on cluster configuration, kube-proxy may use:

```text
iptables
IPVS
```

or the cluster may use an alternative dataplane.

---

## 232. Check kube-proxy

```bash
kubectl get daemonset \
  -n kube-system kube-proxy
```

---

## 233. kube-proxy Logs

```bash
kubectl logs \
  -n kube-system \
  daemonset/kube-proxy
```

---

## 234. Service NAT

Kubernetes Service traffic can involve NAT/load-balancing rules depending on dataplane implementation.

---

## 235. eBPF Dataplane

Some Kubernetes networking implementations use eBPF instead of traditional kube-proxy mechanisms.

Debug using the tools appropriate to the installed CNI/dataplane.

---

## 236. CNI Identification

```bash
kubectl get pods -n kube-system
```

Look for the installed CNI components.

---

## 237. AWS VPC CNI

Typical EKS environments use AWS VPC CNI.

Inspect:

```bash
kubectl get daemonset -n kube-system aws-node
```

---

## 238. AWS VPC CNI Logs

```bash
kubectl logs \
  -n kube-system \
  daemonset/aws-node
```

---

## 239. CNI Configuration

On a node, inspect authorized CNI configuration:

```bash
ls -l /etc/cni/net.d/
```

---

## 240. CNI Binaries

```bash
ls -l /opt/cni/bin/
```

Exact files depend on the CNI.

---

## 241. Pod Sandbox Failure

If a Pod cannot get networking, inspect:

```bash
kubectl describe pod
```

and CNI logs/events.

---

## 242. Pod IP Allocation

In AWS VPC CNI environments, inspect:

```text
Pod IP assignment
ENI capacity
subnet free IPs
CNI errors
```

---

## 243. EKS IP Exhaustion

Symptoms can include Pods stuck pending or networking failures due to insufficient IP capacity.

---

## 244. Check Subnet IP Availability

Use AWS CLI to inspect subnet capacity and resource planning.

---

## 245. Node IP Limits

Instance type and CNI configuration influence available network interfaces/IPs.

---

## 246. Prefix Delegation

AWS VPC CNI can support prefix delegation in suitable configurations to improve Pod IP scalability.

Verify your deployed configuration before troubleshooting.

---

## 247. EKS Pod IP Troubleshooting

Check:

```text
kubectl get pods -o wide
aws-node logs
node type
subnet capacity
ENIs
```

---

## 248. Service Debugging Flow

```text
Service
↓
Selector
↓
EndpointSlice
↓
Pod IP
↓
Pod port
↓
Application
```

---

## 249. Ingress Debugging Flow

```text
DNS
↓
ALB
↓
Listener
↓
Rule
↓
Target Group
↓
Target health
↓
Service
↓
Pod
```

---

## 250. EKS External Debugging Flow

```text
Internet
↓
DNS
↓
WAF/CloudFront
↓
ALB/NLB
↓
AWS networking
↓
EKS
↓
Service
↓
Pod
```

---

## 251. Linux Process Port Mapping

```bash
sudo lsof -i :8080
```

---

## 252. `fuser`

```bash
sudo fuser -n tcp 8080
```

---

## 253. Process Listening Check

```bash
ps aux | grep <process>
```

Use `pgrep` for more reliable process matching.

---

## 254. `pgrep`

```bash
pgrep -af nginx
```

---

## 255. Nginx Port Check

```bash
ss -lntp | grep nginx
```

---

## 256. Nginx Configuration Test

```bash
sudo nginx -t
```

---

## 257. Nginx Active Connections

If status is configured:

```bash
curl http://127.0.0.1/nginx_status
```

---

## 258. Nginx Logs

```bash
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

---

## 259. Systemd Service Status

```bash
systemctl status nginx
```

---

## 260. Systemd Logs

```bash
journalctl -u nginx
```

---

## 261. Follow Systemd Logs

```bash
journalctl -u nginx -f
```

---

## 262. Recent Logs

```bash
journalctl -u nginx --since "10 minutes ago"
```

---

## 263. Kernel Network Logs

```bash
dmesg | grep -i -E 'network|drop|tcp|nic'
```

Use permissions appropriate to the host.

---

## 264. Kernel Socket Information

```bash
ss -s
```

---

## 265. File Descriptor Pressure

A networking issue may actually be FD exhaustion.

Check:

```bash
ulimit -n
```

and process limits.

---

## 266. Process FD Count

```bash
ls /proc/<pid>/fd | wc -l
```

---

## 267. Too Many Open Files

Symptoms:

```text
accept() failures
connection failures
application errors
```

Check application and OS limits.

---

## 268. Connection Backlog

A server receiving more connections than it can accept may show backlog pressure.

Investigate:

```text
listen backlog
SYN queue
accept queue
CPU
```

---

## 269. SYN Flood vs Normal Load

A high SYN count can be caused by:

```text
legitimate burst
network loss
slow server
attack
```

Use multiple data sources before concluding.

---

## 270. Conntrack

Linux connection tracking can be inspected with:

```bash
sudo conntrack -S
```

if the tool is installed.

---

## 271. Conntrack Table

```bash
sudo conntrack -L
```

may produce very large output; filter carefully.

---

## 272. Conntrack Exhaustion

Symptoms can include:

```text
new connections fail
intermittent network failures
```

Investigate table limits and workload connection behavior.

---

## 273. TCP Retransmission Debugging

Use:

```bash
ss -ti
```

and packet captures to inspect TCP-level information.

---

## 274. TCP Statistics

```bash
nstat
```

can provide kernel network statistics where available.

---

## 275. Socket Statistics

```bash
ss -s
```

is a fast first-level connection summary.

---

## 276. Network Namespace

Linux containers use network namespaces.

Inspect process namespace where authorized:

```bash
readlink /proc/<pid>/ns/net
```

---

## 277. `nsenter`

On an authorized node, `nsenter` can inspect a container/network namespace:

```bash
sudo nsenter -t <pid> -n ip addr
```

Use carefully in production.

---

## 278. Container Namespace Debugging

This is useful when host networking looks healthy but the container network namespace is not.

---

## 279. Kubernetes Node Debug

```bash
kubectl debug node/<node> -it \
  --image=ubuntu
```

Use according to cluster security policy.

---

## 280. Host Filesystem

Node debugging can expose host filesystem/process/network information.

Treat it as privileged access.

---

## 281. Kubernetes Events

```bash
kubectl get events -A \
  --sort-by=.lastTimestamp
```

---

## 282. Event Limitations

Events are not a permanent incident history.

Export relevant evidence to approved logging systems when required.

---

## 283. Describe Everything

For a failing Pod:

```bash
kubectl describe pod <pod> -n <namespace>
```

For a Service:

```bash
kubectl describe svc <service> -n <namespace>
```

For Ingress:

```bash
kubectl describe ingress <ingress> -n <namespace>
```

---

## 284. YAML Inspection

```bash
kubectl get svc <service> \
  -n <namespace> -o yaml
```

---

## 285. JSONPath

Example:

```bash
kubectl get svc frontend \
  -n roboshop \
  -o jsonpath='{.spec.clusterIP}'
```

---

## 286. Extract Pod IP

```bash
kubectl get pod <pod> \
  -n <namespace> \
  -o jsonpath='{.status.podIP}'
```

---

## 287. Extract Service Selector

```bash
kubectl get svc frontend \
  -n roboshop \
  -o jsonpath='{.spec.selector}'
```

---

## 288. Watch Resources

```bash
kubectl get pods -n roboshop -w
```

---

## 289. Watch EndpointSlices

```bash
kubectl get endpointslice \
  -n roboshop -w
```

---

## 290. Watch Events

```bash
kubectl get events \
  -n roboshop \
  --watch
```

---

## 291. Rollout Status

```bash
kubectl rollout status \
  deployment/frontend \
  -n roboshop
```

---

## 292. Rollout History

```bash
kubectl rollout history \
  deployment/frontend \
  -n roboshop
```

---

## 293. Rollback

```bash
kubectl rollout undo \
  deployment/frontend \
  -n roboshop
```

Only when rollback is approved and understood.

---

## 294. Deployment Networking Symptoms

A deployment can appear healthy while external traffic fails due to:

```text
wrong Service selector
wrong port
readiness
Ingress rule
```

---

## 295. Service Selector Verification

```bash
kubectl get endpointslice \
  -n roboshop \
  -l kubernetes.io/service-name=frontend
```

---

## 296. EndpointSlice Conditions

Inspect whether endpoints are:

```text
ready
serving
terminating
```

depending on Kubernetes version/API details.

---

## 297. Terminating Pods

During deployment, a Pod can be terminating while traffic is draining.

---

## 298. Graceful Shutdown

Applications should handle:

```text
SIGTERM
```

and stop accepting new work appropriately.

---

## 299. Connection Draining

Load balancers can drain targets during termination according to their configuration.

---

## 300. Network Debugging During Deployment

Correlate:

```text
deployment rollout
target health
Pod readiness
connections
errors
```

---

## 301. DNS + Kubernetes Debug

Test:

```bash
dig frontend.roboshop.svc.cluster.local
```

from a suitable DNS-capable Pod.

---

## 302. Service + HTTP Debug

```bash
curl -v \
  http://frontend.roboshop.svc.cluster.local:80/
```

---

## 303. Service + TCP Debug

```bash
nc -vz \
  frontend.roboshop.svc.cluster.local \
  80
```

---

## 304. Pod + HTTP Debug

```bash
curl -v http://<pod-ip>:8080/
```

---

## 305. Pod + TCP Debug

```bash
nc -vz <pod-ip> 8080
```

---

## 306. External + HTTP Debug

```bash
curl -v https://shop.example.com
```

---

## 307. External + DNS Debug

```bash
dig +short shop.example.com
```

---

## 308. External + TLS Debug

```bash
openssl s_client \
  -connect shop.example.com:443 \
  -servername shop.example.com
```

---

## 309. External + ALB Target Debug

Use AWS CLI:

```bash
aws elbv2 describe-target-health \
  --target-group-arn <arn>
```

---

## 310. External + WAF Debug

Check WAF logs/metrics for matching blocked requests.

---

## 311. Debugging 403

Flow:

```text
curl -v
↓
HTTP status 403
↓
check WAF
↓
check ALB rules
↓
check application auth
```

---

## 312. Debugging 404

Flow:

```text
curl -v
↓
404
↓
check Host
↓
check path
↓
check Ingress
↓
check application route
```

---

## 313. Debugging 502

Flow:

```text
502
↓
target health
↓
backend port
↓
protocol
↓
TLS
↓
Pod listener
```

---

## 314. Debugging 503

Flow:

```text
503
↓
target count
↓
target health
↓
EndpointSlice
↓
Pod readiness
```

---

## 315. Debugging 504

Flow:

```text
504
↓
target response time
↓
application latency
↓
downstream latency
↓
timeouts
```

---

## 316. Debugging DNS + HTTP

If:

```bash
dig example.com
```

works but:

```bash
curl https://example.com
```

fails, move to:

```text
TCP
TLS
HTTP
```

Do not keep changing DNS configuration without evidence.

---

## 317. Debugging TCP + HTTP

If:

```bash
nc -vz example.com 443
```

works but curl fails:

```text
TCP works
TLS/HTTP remains
```

---

## 318. Debugging TLS + HTTP

If TLS handshake succeeds but HTTP fails:

```text
TLS works
HTTP/application remains
```

---

## 319. Debugging Service

If Service DNS resolves but TCP fails:

```text
DNS works
Service dataplane/application remains
```

---

## 320. Debugging Pod

If Pod IP works but Service IP fails:

```text
Pod/application works
Service dataplane/selector/routing remains
```

---

## 321. Debugging Ingress

If Service works but external URL fails:

```text
Service works
Ingress/ALB/DNS/WAF remains
```

---

## 322. Debugging ALB

If ALB is reachable but target health is unhealthy:

```text
ALB listener works
ALB-to-target path remains
```

---

## 323. Debugging Route

Use:

```bash
ip route get <destination>
```

from the actual source host/network namespace.

---

## 324. Debugging Security Group

Verify:

```text
source
destination
port
protocol
direction
```

---

## 325. Debugging NACL

Verify both:

```text
request direction
return direction
```

because NACLs are stateless.

---

## 326. Debugging NetworkPolicy

Verify:

```text
podSelector
namespaceSelector
ipBlock
ports
policyTypes
```

---

## 327. Debugging NAT

If Pods cannot reach external APIs:

```text
Pod route
→ node/VPC route
→ NAT
→ IGW
```

depending on architecture.

---

## 328. NAT Debug Commands

Inspect:

```bash
ip route
```

and AWS resources:

```bash
aws ec2 describe-nat-gateways
```

---

## 329. Egress Test

From a diagnostic Pod:

```bash
curl -v https://example.com
```

---

## 330. Egress DNS Test

```bash
dig example.com
```

---

## 331. Egress TCP Test

```bash
nc -vz example.com 443
```

---

## 332. Egress Failure Matrix

```text
DNS fails:
resolver/network

DNS works, TCP fails:
route/firewall/NAT

TCP works, TLS fails:
certificate/TLS

TLS works, HTTP fails:
application/proxy
```

---

## 333. AWS CLI EKS

```bash
aws eks describe-cluster \
  --name <cluster>
```

Inspect relevant cluster networking configuration.

---

## 334. EKS Cluster Endpoint

The Kubernetes API endpoint may be:

```text
public
private
public + private
```

depending on configuration.

---

## 335. EKS API Connectivity

If `kubectl` fails:

```text
check current context
AWS credentials
cluster endpoint
DNS
network access
authorization
```

---

## 336. Update Kubeconfig

```bash
aws eks update-kubeconfig \
  --region <region> \
  --name <cluster>
```

---

## 337. EKS Authentication

Verify:

```bash
aws sts get-caller-identity
```

before investigating Kubernetes authorization.

---

## 338. `kubectl` Debugging

```bash
kubectl auth can-i get pods -A
```

Tests current Kubernetes authorization.

---

## 339. EKS API DNS

Resolve the cluster endpoint with:

```bash
dig <cluster-endpoint-hostname>
```

---

## 340. Kubernetes API TCP

Test the endpoint on its expected port using appropriate tools.

---

## 341. EKS API TLS

Use:

```bash
openssl s_client \
  -connect <endpoint>:443 \
  -servername <endpoint>
```

where appropriate.

---

## 342. Production EKS API Debugging

Separate:

```text
AWS identity
↓
EKS authentication
↓
Kubernetes authorization
↓
network connectivity
```

---

## 343. Common Mistake

A user may say:

```text
"Network is down"
```

when the actual issue is:

```text
IAM authorization
```

Always distinguish layers.

---

## 344. Kubernetes Authorization

```bash
kubectl auth can-i \
  get pods \
  -n roboshop
```

---

## 345. AWS Identity

```bash
aws sts get-caller-identity
```

---

## 346. Network Identity

Determine:

```text
source IP
source namespace
source Pod
source node
```

when debugging policy.

---

## 347. Source IP Debug

From a Pod:

```bash
ip addr
```

and:

```bash
ip route
```

---

## 348. Destination Debug

Determine:

```text
DNS name
resolved IP
port
protocol
```

before testing.

---

## 349. Standard Debug Template

```text
Source:
Destination:
Protocol:
Port:
Expected:
Actual:
First failing layer:
Evidence:
Change:
Verification:
```

---

## 350. Incident Example: DNS Failure

```text
Symptom:
curl: Could not resolve host

Commands:
dig api.example.com
cat /etc/resolv.conf
getent hosts api.example.com
```

Then inspect resolver/network.

---

## 351. Incident Example: TCP Failure

```text
Symptom:
Connection timed out

Commands:
ip route get <destination>
nc -vz -w 5 <destination> <port>
ss -s
tcpdump
```

Then inspect network controls.

---

## 352. Incident Example: Connection Refused

```text
nc: Connection refused
```

Usually indicates that the destination is reachable but no acceptable listener is accepting the connection, though network devices can also generate rejects.

Check:

```bash
ss -lntp
```

on the destination where authorized.

---

## 353. Incident Example: Connection Timeout

Possible:

```text
dropped packets
firewall
route
security group
NACL
listener unreachable
```

---

## 354. Incident Example: TLS Certificate Error

Commands:

```bash
openssl s_client ...
curl -v ...
```

Check:

```text
expiry
SAN
issuer
chain
SNI
```

---

## 355. Incident Example: HTTP 503

Commands:

```bash
curl -v
kubectl get endpointslice
aws elbv2 describe-target-health
```

---

## 356. Incident Example: Service Has No Endpoints

Commands:

```bash
kubectl get svc -o yaml
kubectl get pods --show-labels
kubectl get endpointslice
```

Compare selectors.

---

## 357. Incident Example: NetworkPolicy

Commands:

```bash
kubectl get networkpolicy -A
kubectl describe networkpolicy <name>
```

Then test from an actual source Pod.

---

## 358. Incident Example: EKS Pod IP Exhaustion

Commands:

```bash
kubectl get pods -A -o wide
kubectl logs -n kube-system daemonset/aws-node
aws ec2 describe-subnets
```

Check IP capacity and CNI errors.

---

## 359. Incident Example: High Latency

Measure:

```text
DNS
TCP connect
TLS
TTFB
total time
```

using curl timing.

---

## 360. Curl Timing Interpretation

```text
time_namelookup:
DNS

time_connect:
TCP connection

time_appconnect:
TLS handshake

time_starttransfer:
server first-byte latency

time_total:
complete request
```

---

## 361. Latency Breakdown

If DNS is high:

```text
resolver
```

If connect is high:

```text
network/TCP
```

If appconnect is high:

```text
TLS
```

If TTFB is high:

```text
application/upstream
```

---

## 362. Packet Loss

Check:

```bash
ping
mtr
tcpdump
```

but remember that ICMP behavior may not represent application traffic.

---

## 363. Packet Loss vs Retransmission

TCP retransmissions are often more meaningful for TCP application performance than ICMP loss alone.

---

## 364. MTU Incident

Symptoms:

```text
TCP connects
small requests work
large requests hang
```

Check:

```bash
tracepath
tcpdump
ip link
```

---

## 365. Proxy Incident

Symptoms:

```text
external curl works with --noproxy
internal request fails normally
```

Check:

```bash
env | grep -i proxy
```

and NO_PROXY.

---

## 366. IPv6 Incident

Symptoms:

```text
IPv4 works
IPv6 fails
```

Test:

```bash
curl -4
curl -6
dig A
dig AAAA
```

---

## 367. Load Balancer Host Rule Incident

Test:

```bash
curl --resolve api.example.com:443:<ALB-IP> \
  https://api.example.com/
```

This separates DNS from ALB host routing.

---

## 368. DNS vs ALB Incident

If `--resolve` works but normal DNS fails:

```text
DNS issue
```

If both resolve correctly but request fails:

```text
ALB/TLS/routing/backend
```

---

## 369. TLS vs ALB Incident

If:

```bash
openssl s_client ...
```

fails:

```text
TLS/listener/certificate
```

If TLS succeeds but curl returns 404:

```text
HTTP routing/application
```

---

## 370. Production Debugging Order

Recommended order:

```text
1. Confirm scope
2. Confirm source
3. Confirm destination
4. Resolve DNS
5. Check route
6. Test TCP
7. Test TLS
8. Test HTTP
9. Inspect load balancer
10. Inspect Kubernetes
11. Inspect application
```

---

## 371. Do Not Restart First

Restarting may temporarily hide:

```text
connection leak
resource exhaustion
race condition
bad deployment
```

and destroys useful evidence.

---

## 372. Capture Evidence

Before changing production:

```text
timestamp
commands
outputs
metrics
logs
resource versions
```

where permitted.

---

## 373. Time Correlation

Use consistent timestamps across:

```text
client
ALB
WAF
Kubernetes
application
```

---

## 374. UTC

Production systems commonly log in UTC.

Know the timezone before correlating incidents.

---

## 375. Command Safety

Prefer read-only commands first:

```text
get
describe
show
status
logs
```

---

## 376. Dangerous Commands

Be cautious with:

```text
iptables -F
nft flush ruleset
ip route flush
kubectl delete
```

Never execute destructive networking commands casually in production.

---

## 377. Firewall Flush Warning

Never blindly run:

```bash
iptables -F
```

on a remote production system.

You may lock yourself out.

---

## 378. Route Flush Warning

Never blindly flush production routes.

---

## 379. Kubernetes Delete Warning

Do not delete:

```text
Service
Ingress
NetworkPolicy
Pod
```

just to "test" unless the impact is understood and authorized.

---

## 380. Temporary Debug Changes

If a temporary rule is required:

```text
document it
time-limit it
monitor it
remove it
```

---

## 381. Production Packet Capture

Prefer:

```text
short duration
narrow filter
approved host
approved data handling
```

---

## 382. Capture Narrowly

Bad:

```bash
tcpdump -i any
```

for a long time.

Better:

```bash
tcpdump -i eth0 host <ip> and port 443
```

with a bounded duration.

---

## 383. Capture File Size

Use rotation/count options where needed:

```bash
tcpdump -C 50 -W 5 -w capture.pcap
```

Adjust according to disk capacity and policy.

---

## 384. Production Disk Risk

Packet captures can consume disk quickly.

Monitor disk usage.

---

## 385. Debugging Through Bastion

If direct access is unavailable:

```text
Laptop
 |
Bastion
 |
Private resource
```

Use approved access paths.

---

## 386. SSH Port Forwarding

For controlled troubleshooting:

```bash
ssh -L 8080:internal-service:80 user@bastion
```

Use only where authorized.

---

## 387. SSH Tunnel Caution

Do not create persistent tunnels that bypass security architecture.

---

## 388. Port Forward Kubernetes

```bash
kubectl port-forward \
  svc/frontend 8080:80 \
  -n roboshop
```

Useful for controlled local testing.

---

## 389. Port Forward Limitation

Port-forward proves the application can be reached through the Kubernetes API path; it does not prove normal Service/Ingress networking works.

---

## 390. Compare Access Paths

```text
port-forward works
Service fails
```

suggests:

```text
Service/networking path
```

rather than basic application availability.

---

## 391. Application Localhost Test

Inside Pod:

```bash
curl http://127.0.0.1:8080/health
```

---

## 392. Pod IP Test

```bash
curl http://<pod-ip>:8080/health
```

---

## 393. Service Test

```bash
curl http://frontend:80/health
```

---

## 394. Ingress Test

```bash
curl -H 'Host: shop.example.com' \
  http://<ingress-endpoint>/health
```

Use HTTPS/SNI where required.

---

## 395. Public Test

```bash
curl -v https://shop.example.com/health
```

---

## 396. Layered Success Matrix

```text
localhost: pass
Pod IP: pass
Service: pass
Ingress: fail
```

Focus on:

```text
Ingress/ALB
```

---

## 397. Another Matrix

```text
localhost: pass
Pod IP: fail
```

Focus on:

```text
application binding
container port
Pod networking
```

---

## 398. Another Matrix

```text
Pod IP: pass
Service: fail
```

Focus on:

```text
Service selector
EndpointSlice
Service dataplane
NetworkPolicy
```

---

## 399. Another Matrix

```text
Service: pass
Ingress: fail
```

Focus on:

```text
Ingress
ALB
target health
DNS
TLS
```

---

## 400. Another Matrix

```text
Ingress: pass
public URL: fail
```

Focus on:

```text
DNS
WAF
CloudFront
external routing
```

---

## 401. Production Troubleshooting Runbook

```text
Incident:
Timestamp:
Affected URL:
Affected users:
Source:
Destination:
Expected:
Actual:

DNS:
TCP:
TLS:
HTTP:

ALB:
WAF:
Ingress:
Service:
EndpointSlice:
Pod:

NetworkPolicy:
Security Group:
NACL:
Route:
NAT:

Logs:
Metrics:
Traces:

Root cause:
Fix:
Verification:
Preventive action:
```

---

## 402. Network Debugging Commands by Layer

| Layer | Commands |
|---|---|
| Interface | `ip addr`, `ip link` |
| Route | `ip route`, `ip route get` |
| Neighbor | `ip neigh` |
| DNS | `dig`, `host`, `nslookup`, `getent` |
| TCP | `ss`, `nc` |
| ICMP | `ping` |
| Path | `traceroute`, `tracepath`, `mtr` |
| TLS | `openssl s_client` |
| HTTP | `curl`, `wget` |
| Packets | `tcpdump` |
| Firewall | `iptables`, `nft` |
| Conntrack | `conntrack` |
| Kubernetes | `kubectl get`, `describe`, `logs`, `exec` |
| AWS | `aws ec2`, `aws elbv2`, `aws eks` |

---

## 403. Most Important Linux Commands

```bash
ip addr
ip route
ip route get <destination>
ip neigh
ss -lntup
ss -s
dig
getent hosts
ping
traceroute
mtr
nc
curl
openssl s_client
tcpdump
```

---

## 404. Most Important Kubernetes Commands

```bash
kubectl get pods -A -o wide
kubectl get svc -A
kubectl get endpointslice -A
kubectl get ingress -A
kubectl get networkpolicy -A
kubectl describe pod
kubectl describe svc
kubectl describe ingress
kubectl logs
kubectl exec
kubectl get events
kubectl debug
```

---

## 405. Most Important AWS Commands

```bash
aws sts get-caller-identity
aws eks describe-cluster
aws elbv2 describe-load-balancers
aws elbv2 describe-listeners
aws elbv2 describe-rules
aws elbv2 describe-target-groups
aws elbv2 describe-target-health
aws ec2 describe-security-groups
aws ec2 describe-route-tables
aws ec2 describe-subnets
aws ec2 describe-vpcs
aws ec2 describe-nat-gateways
aws ec2 describe-flow-logs
```

---

## 406. DNS Debugging Checklist

```text
[ ] hostname correct
[ ] A/AAAA record
[ ] CNAME
[ ] resolver
[ ] authoritative server
[ ] TTL
[ ] split DNS
[ ] private/public zone
[ ] DNSSEC if applicable
```

---

## 407. TCP Debugging Checklist

```text
[ ] destination IP
[ ] route
[ ] listener
[ ] port
[ ] SG
[ ] NACL
[ ] firewall
[ ] return path
[ ] retransmissions
```

---

## 408. TLS Debugging Checklist

```text
[ ] certificate
[ ] expiration
[ ] SAN
[ ] issuer
[ ] chain
[ ] SNI
[ ] protocol
[ ] cipher/policy
[ ] listener
```

---

## 409. HTTP Debugging Checklist

```text
[ ] Host
[ ] path
[ ] method
[ ] status
[ ] headers
[ ] redirect
[ ] cookies
[ ] CORS
[ ] application logs
```

---

## 410. Kubernetes Service Checklist

```text
[ ] Service exists
[ ] selector
[ ] labels
[ ] port
[ ] targetPort
[ ] EndpointSlice
[ ] Pod readiness
[ ] NetworkPolicy
```

---

## 411. Ingress Checklist

```text
[ ] ingressClass
[ ] hostname
[ ] path
[ ] backend Service
[ ] listener
[ ] certificate
[ ] target health
[ ] controller logs
```

---

## 412. EKS Checklist

```text
[ ] CNI
[ ] Pod IP
[ ] subnet capacity
[ ] route
[ ] SG
[ ] NACL
[ ] NetworkPolicy
[ ] aws-node
[ ] kube-proxy/dataplane
```

---

## 413. Production Anti-Patterns

Avoid:

```text
restarting everything first
flushing iptables
deleting resources blindly
testing from the wrong network
assuming ping proves HTTPS
assuming DNS failure from HTTP failure
using -k permanently
capturing all traffic indefinitely
changing multiple controls simultaneously
```

---

## 414. Production Debugging Best Practice

Always narrow the problem.

Bad:

```text
"Network is not working."
```

Good:

```text
"Pod A can resolve Service B but TCP/8080 times out.
The Service has healthy EndpointSlices. Next I am checking
NetworkPolicy and the source/destination path."
```

---

## 415. Production Engineer Mindset

Think in:

```text
source
destination
protocol
port
path
policy
evidence
```

---

## 416. Interview: How Do You Troubleshoot Network Issues?

Answer:

```text
I first identify the source, destination, protocol and expected
behavior. I validate DNS, then route selection, TCP connectivity, TLS
and HTTP. In Kubernetes I inspect the Service selector, EndpointSlices,
Pod readiness, NetworkPolicy and CNI. For EKS external traffic I also
check the load balancer listener, target health, Security Groups,
NACLs and routes. I use curl, dig, ss, nc, openssl and tcpdump for
targeted evidence rather than changing multiple layers at once.
```

---

## 417. Interview: What Is the Difference Between `curl` and `nc`?

Answer:

```text
nc primarily tests network socket connectivity. curl understands
application protocols such as HTTP and HTTPS and can expose DNS,
connection, TLS and HTTP behavior. I use nc to answer "can I establish
the socket?" and curl to answer "can the application protocol work?"
```

---

## 418. Interview: Why Does Ping Work but Curl Fail?

Answer:

```text
Ping uses ICMP while curl normally uses TCP plus HTTP/HTTPS. ICMP can
work while TCP/443 is blocked, the application is down, TLS fails, or
the HTTP route is incorrect.
```

---

## 419. Interview: Why Does `nc` Work but Curl Fail?

Answer:

```text
TCP connectivity is working. I move to TLS and HTTP investigation:
certificate, SNI, protocol, HTTP status, Host header, redirects and
application behavior.
```

---

## 420. Interview: Why Does DNS Work but the Application Fail?

Answer:

```text
DNS only proves name resolution. I then test route, TCP, TLS and HTTP
to determine where the request stops.
```

---

## 421. Interview: What Does `ip route get` Tell You?

Answer:

```text
It asks the Linux routing stack how it would route traffic to a
specific destination, helping identify the selected interface,
gateway and source address.
```

---

## 422. Interview: What Does `ss` Show?

Answer:

```text
It shows socket information including listening ports, established
connections and connection states. It is useful for confirming whether
an application is actually listening and for identifying connection
pressure.
```

---

## 423. Interview: What Is `tcpdump` Used For?

Answer:

```text
It captures packets so I can verify whether traffic is leaving the
source, reaching the interface, completing the TCP handshake,
retransmitting or being reset. I use narrow filters and short captures
in production.
```

---

## 424. Interview: How Do You Debug TLS?

Answer:

```text
I use openssl s_client with the expected hostname/SNI, inspect the
certificate chain, SAN, validity and negotiated protocol, and compare
the result with the load balancer/application TLS configuration.
```

---

## 425. Interview: How Do You Debug Kubernetes DNS?

Answer:

```text
I test from inside a Pod, inspect /etc/resolv.conf, resolve the
Service FQDN, check the kube-dns/CoreDNS Service and EndpointSlices,
and inspect CoreDNS logs if necessary.
```

---

## 426. Interview: How Do You Debug a Service With No Traffic?

Answer:

```text
I inspect the Service selector and compare it with Pod labels, then
check EndpointSlices, Pod readiness, Service ports and targetPort.
After that I test the Service DNS/IP from a suitable source Pod.
```

---

## 427. Interview: How Do You Debug NetworkPolicy?

Answer:

```text
I identify the exact source Pod and destination Pod, inspect both
labels and namespaces, then evaluate ingress/egress policy direction,
ports and protocols. I reproduce the connection from the actual
source network context.
```

---

## 428. Interview: How Do You Debug EKS Pod IP Problems?

Answer:

```text
I inspect Pod IP allocation, aws-node logs, subnet available IPs,
instance networking capacity and CNI configuration. I distinguish
scheduler/resource problems from actual CNI/IP allocation failures.
```

---

## 429. Interview: How Do You Debug an ALB 503?

Answer:

```text
I check target group health and registration first, then EndpointSlices,
Pod readiness, Service configuration, targetPort and the controller
events/logs.
```

---

## 430. Interview: How Do You Debug a 502?

Answer:

```text
I focus on the ALB-to-target connection: protocol, port, TLS,
listener and application behavior. I use target health, curl from an
appropriate network location and application logs.
```

---

## 431. Interview: How Do You Debug High Latency?

Answer:

```text
I break total latency into DNS, TCP connect, TLS handshake, TTFB and
total request time. Then I correlate the slow segment with ALB metrics,
application metrics, downstream services and traces.
```

---

## 432. Interview: What Is the Difference Between Timeout and Refused?

Answer:

```text
A timeout means no usable response was received within the expected
period and can indicate filtering, routing or an unresponsive service.
Connection refused generally indicates that the destination actively
rejected the connection or no listener accepted it. I verify with
ss/tcpdump rather than assuming the exact cause from the message alone.
```

---

## 433. Interview: How Do You Debug a Proxy Issue?

Answer:

```text
I inspect HTTP_PROXY, HTTPS_PROXY, ALL_PROXY and NO_PROXY. I compare
the request with and without the proxy and verify whether internal
Kubernetes/AWS destinations are correctly excluded.
```

---

## 434. Interview: How Do You Debug IPv4 vs IPv6?

Answer:

```text
I compare A and AAAA records and test curl -4 and curl -6. If only one
protocol fails, I inspect its route, security controls, listener and
DNS configuration.
```

---

## 435. Interview: What Is a Good Production Network Debugging Sequence?

Answer:

```text
Source → Destination → DNS → Route → TCP → TLS → HTTP → Load Balancer
→ Kubernetes Service → EndpointSlice → Pod → Application.

I use evidence at each stage and stop at the first failed layer.
```

---

## 436. Final Command Cheat Sheet

```bash
# Identity/context
aws sts get-caller-identity
kubectl config current-context

# Linux
ip addr
ip route
ip route get <dst>
ip neigh
ss -lntup
ss -s

# DNS
dig <name>
dig +short <name>
dig @<server> <name>
dig +trace <name>
getent hosts <name>

# Connectivity
ping -c 4 <dst>
nc -vz <dst> <port>
traceroute <dst>
tracepath <dst>
mtr <dst>

# TLS
openssl s_client -connect <host>:443 -servername <host>

# HTTP
curl -v https://<host>
curl -I https://<host>
curl -IL https://<host>
curl --resolve <host>:443:<ip> https://<host>

# Packets
tcpdump -i eth0 host <ip>
tcpdump -i eth0 port 443

# Kubernetes
kubectl get pods -A -o wide
kubectl get svc -A
kubectl get endpointslice -A
kubectl get ingress -A
kubectl get networkpolicy -A
kubectl describe pod <pod> -n <ns>
kubectl logs <pod> -n <ns>

# AWS
aws elbv2 describe-load-balancers
aws elbv2 describe-target-groups
aws elbv2 describe-target-health --target-group-arn <arn>
aws ec2 describe-security-groups --group-ids <id>
aws ec2 describe-route-tables
aws ec2 describe-subnets
```

---

## 437. Final Troubleshooting Decision Tree

```text
Application unreachable
        |
        v
Does DNS resolve?
   |             |
  NO            YES
   |             |
DNS issue     Can TCP connect?
                 |       |
                NO      YES
                 |       |
          Route/SG/NACL  Does TLS work?
                          |       |
                         NO      YES
                          |       |
                       TLS issue  Does HTTP work?
                                    |       |
                                   NO      YES
                                    |       |
                           LB/Ingress/App  Healthy
```

---

## 438. Final Kubernetes Decision Tree

```text
Service request fails
        |
        v
Does Service resolve?
   |          |
  NO         YES
   |          |
CoreDNS     Are endpoints present?
             |          |
            NO         YES
             |          |
      selector/readiness  Can Pod IP connect?
                          |          |
                         NO         YES
                          |          |
                   NetworkPolicy/Pod  Service dataplane
```

---

## 439. Final EKS External Decision Tree

```text
Public URL fails
      |
      v
DNS resolves?
      |
      v
ALB reachable?
      |
      v
TLS works?
      |
      v
Listener rule matches?
      |
      v
Target healthy?
      |
      v
Service endpoints exist?
      |
      v
Pod responds?
      |
      v
Application healthy?
```

---

## 440. Final Production Principles

```text
1. Always identify source and destination.
2. Always identify protocol and port.
3. Test from the correct network location.
4. DNS is not TCP.
5. TCP is not TLS.
6. TLS is not HTTP.
7. HTTP success is not application correctness.
8. Ping is not proof of application reachability.
9. Service DNS resolving does not prove Service connectivity.
10. Pod readiness does not prove ALB target health.
11. ALB health does not prove application correctness.
12. NetworkPolicy must be evaluated from the real source.
13. Security Groups are stateful.
14. NACLs are stateless.
15. `ip route get` is valuable for route selection.
16. `ss` is valuable for socket state.
17. `tcpdump` provides packet-level evidence.
18. `curl` provides application-level evidence.
19. `openssl` provides TLS-level evidence.
20. `dig` provides DNS-level evidence.
21. Use narrow production packet captures.
22. Do not flush firewalls blindly.
23. Do not delete Kubernetes resources blindly.
24. Do not restart before collecting evidence.
25. Correlate logs, metrics and traces.
26. Document temporary changes.
27. Verify every remediation.
28. Automate recurring diagnostics.
29. Keep runbooks version-controlled.
30. Troubleshoot the first failed layer, not the loudest symptom.
```

---