# 05-Network-Automation

## Python for Linux DevOps

Network automation is one of the most useful areas of Python for DevOps engineers.

A DevOps engineer frequently needs to automate:

- connectivity checks
- DNS validation
- port checks
- HTTP/API health checks
- IP and CIDR calculations
- route inspection
- socket communication
- network inventory
- firewall validation
- service endpoint verification
- cloud network validation
- Kubernetes connectivity troubleshooting
- deployment preflight checks

The goal is not to replace mature networking tools with Python.

The goal is to use Python when custom validation, orchestration, reporting, API integration, or repeatable operational workflows are required.

---

# 1. Linux Networking Mental Model

Think about network troubleshooting in layers:

```text
Application
    ↓
DNS
    ↓
TCP/UDP
    ↓
IP routing
    ↓
Network interface
    ↓
Firewall/security rules
    ↓
Cloud networking
    ↓
Remote endpoint
```

When an application cannot connect, move through these layers systematically.

---

# 2. Essential Linux Network Commands

A DevOps engineer should know:

```bash
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
host
curl
wget
nc
telnet
tcpdump
```

Python can automate many of the checks performed by these commands.

---

# 3. IP Addresses

IPv4 example:

```text
10.0.10.25
```

IPv6 example:

```text
2001:db8::25
```

Python's standard library provides the `ipaddress` module.

```python
import ipaddress

ip = ipaddress.ip_address(
    "10.0.10.25"
)

print(ip)
print(ip.version)
```

---

# 4. IPv4 vs IPv6

```text
IPv4
32-bit
example: 192.168.1.10

IPv6
128-bit
example: 2001:db8::10
```

Modern infrastructure may use both.

Python can validate both with `ipaddress`.

---

# 5. CIDR

CIDR represents an IP network:

```text
10.0.0.0/16
```

The `/16` indicates the network prefix length.

Common examples:

```text
10.0.0.0/8
10.0.0.0/16
10.0.10.0/24
```

---

# 6. Python CIDR Validation

```python
import ipaddress

network = ipaddress.ip_network(
    "10.0.10.0/24"
)

print(network.network_address)
print(network.broadcast_address)
print(network.num_addresses)
```

---

# 7. Check Whether an IP Belongs to a Network

```python
import ipaddress

network = ipaddress.ip_network(
    "10.0.10.0/24"
)

ip = ipaddress.ip_address(
    "10.0.10.25"
)

print(ip in network)
```

This is useful for:

- subnet validation
- allowlists
- configuration checks
- deployment preflight
- cloud network validation

---

# 8. Strict vs Non-Strict Networks

```python
ipaddress.ip_network(
    "10.0.10.25/24",
    strict=False,
)
```

With `strict=False`, Python normalizes the network address.

This is useful when input may contain a host address with a prefix.

---

# 9. Subnet Calculation

```python
network = ipaddress.ip_network(
    "10.0.0.0/24"
)

for subnet in network.subnets(
    prefixlen_diff=2
):
    print(subnet)
```

This creates smaller subnets.

---

# 10. Supernet

```python
network = ipaddress.ip_network(
    "10.0.10.0/24"
)

print(
    network.supernet(
        prefixlen_diff=2
    )
)
```

Useful when validating aggregate network ranges.

---

# 11. Address Classification

Python can determine whether an IP is:

```text
private
global
loopback
multicast
link-local
reserved
```

Example:

```python
ip = ipaddress.ip_address(
    "10.0.0.10"
)

print(ip.is_private)
print(ip.is_loopback)
print(ip.is_global)
```

---

# 12. Private Networks

Common private IPv4 ranges:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

AWS VPCs commonly use private RFC1918 address space.

---

# 13. DevOps Use Case — CIDR Validation

Before provisioning infrastructure:

```python
import ipaddress

def valid_subnet(value):
    try:
        ipaddress.ip_network(
            value,
            strict=False,
        )
        return True
    except ValueError:
        return False
```

Use this in:

- Terraform prechecks
- CI validation
- configuration validation
- network inventory

---

# 14. Avoid IP Overlap

Two networks overlap if their address ranges intersect.

Python:

```python
import ipaddress

a = ipaddress.ip_network(
    "10.0.0.0/16"
)

b = ipaddress.ip_network(
    "10.0.10.0/24"
)

print(a.overlaps(b))
```

This is valuable when designing:

- VPCs
- VPC peering
- VPN networks
- Kubernetes networks
- office networks

---

# 15. Network Planning

Example:

```text
VPC
 ├── Public subnet
 ├── Private application subnet
 └── Private database subnet
```

Python can validate that planned CIDRs:

```text
do not overlap
fit inside the VPC
match expected prefix lengths
```

---

# 16. Network Interfaces

Linux command:

```bash
ip addr
```

Python can inspect local interfaces with libraries such as `psutil` when installed.

```python
import psutil

interfaces = psutil.net_if_addrs()

for name, addresses in interfaces.items():
    print(name)

    for address in addresses:
        print(
            address.family,
            address.address
        )
```

---

# 17. Interface State

Linux:

```bash
ip link
```

Look for:

```text
UP
DOWN
```

An interface may have an IP address but still have a link/state problem.

---

# 18. Interface Inventory

A DevOps inventory may capture:

```text
interface name
MAC address
IPv4
IPv6
netmask
status
```

This can be exported as:

```json
{
  "interface": "eth0",
  "addresses": [
    "10.0.10.20"
  ]
}
```

---

# 19. MAC Address

A MAC address identifies a network interface at the data-link layer.

Example:

```text
02:42:ac:11:00:02
```

In cloud environments, MAC addresses are usually less useful for application-level identity than:

```text
instance ID
ENI
private IP
hostname
```

---

# 20. Default Route

Linux:

```bash
ip route
```

Typical:

```text
default via 10.0.0.1 dev eth0
```

The default route determines where traffic goes when no more specific route exists.

---

# 21. Python Route Inspection

Python can execute:

```python
import subprocess

result = subprocess.run(
    ["ip", "route"],
    capture_output=True,
    text=True,
    check=True,
)

print(result.stdout)
```

For deeper networking automation, specialized libraries or netlink interfaces may be preferable.

---

# 22. Routing Troubleshooting

If an application cannot reach an endpoint:

```text
destination IP
 ↓
route lookup
 ↓
gateway
 ↓
interface
 ↓
firewall
 ↓
remote network
```

Start with:

```bash
ip route get <destination>
```

---

# 23. `ip route get`

Example:

```bash
ip route get 10.20.30.40
```

This can reveal:

```text
selected route
gateway
interface
source address
```

Very useful during production incidents.

---

# 24. Python Route Preflight

A Python deployment tool can call:

```python
result = subprocess.run(
    [
        "ip",
        "route",
        "get",
        "10.20.30.40",
    ],
    capture_output=True,
    text=True,
    check=False,
)

print(result.stdout)
```

Use this when routing is part of the deployment dependency.

---

# 25. DNS

DNS converts names into addresses.

Example:

```text
api.example.com
      ↓
10.0.10.25
```

DevOps troubleshooting should distinguish:

```text
DNS failure
```

from:

```text
TCP failure
```

and:

```text
application failure
```

---

# 26. Python DNS Resolution

The standard library provides:

```python
import socket

address = socket.gethostbyname(
    "example.com"
)

print(address)
```

This performs a basic hostname resolution.

---

# 27. Get All Addresses

```python
import socket

results = socket.getaddrinfo(
    "example.com",
    443,
)

for result in results:
    print(result)
```

Useful for:

```text
IPv4
IPv6
TCP
UDP
```

endpoint discovery.

---

# 28. DNS Failure

Typical errors:

```text
socket.gaierror
Name or service not known
Temporary failure in name resolution
```

Investigate:

```bash
resolvectl status
cat /etc/resolv.conf
getent hosts example.com
dig example.com
```

---

# 29. `getent hosts`

```bash
getent hosts example.com
```

This is useful because it follows the host's configured name-service resolution path.

It can be more representative than querying a specific DNS server directly.

---

# 30. `dig`

```bash
dig example.com
```

Specific record:

```bash
dig A example.com
dig AAAA example.com
dig CNAME example.com
dig MX example.com
```

---

# 31. DNS TTL

DNS records have TTL values.

A deployment that changes DNS should consider:

```text
record TTL
resolver cache
client cache
load balancer
propagation behavior
```

Do not assume every client sees a change immediately.

---

# 32. Python DNS Health Check

```python
import socket


def dns_check(host):
    try:
        addresses = socket.getaddrinfo(
            host,
            None,
        )

        return {
            "ok": True,
            "addresses": sorted(
                {
                    item[4][0]
                    for item in addresses
                }
            ),
        }

    except socket.gaierror as exc:
        return {
            "ok": False,
            "error": str(exc),
        }
```

---

# 33. TCP

TCP provides:

```text
connection-oriented transport
reliable delivery
ordering
retransmission
flow control
```

DevOps engineers frequently troubleshoot TCP connectivity.

---

# 34. TCP Connection Test

Linux:

```bash
nc -vz example.com 443
```

Python:

```python
import socket

sock = socket.create_connection(
    ("example.com", 443),
    timeout=5,
)

sock.close()
```

---

# 35. Python TCP Port Check

```python
import socket


def port_open(
    host,
    port,
    timeout=3,
):
    try:
        with socket.create_connection(
            (host, port),
            timeout=timeout,
        ):
            return True

    except OSError:
        return False
```

---

# 36. Port Open Does Not Mean Application Healthy

If:

```text
TCP connection succeeds
```

the service may still return:

```text
HTTP 500
TLS error
authentication failure
invalid response
```

Continue to the application layer.

---

# 37. UDP

UDP is:

```text
connectionless
```

and does not provide TCP-style delivery guarantees.

Common uses:

```text
DNS
metrics
streaming
some service discovery
```

A UDP connectivity test is different from TCP.

---

# 38. Socket Basics

```python
import socket

sock = socket.socket(
    socket.AF_INET,
    socket.SOCK_STREAM,
)

sock.connect(
    ("example.com", 443)
)

sock.close()
```

Use context managers for safer cleanup.

---

# 39. Socket Timeout

Never let operational network automation wait indefinitely.

```python
sock.settimeout(5)
```

or:

```python
socket.create_connection(
    host_port,
    timeout=5,
)
```

Always define reasonable timeouts.

---

# 40. Connection Timeout vs Read Timeout

Conceptually:

```text
connection timeout
=
time allowed to establish connection

read timeout
=
time allowed waiting for data
```

HTTP clients often expose these separately.

This distinction is important for diagnosing latency.

---

# 41. HTTP

HTTP is an application-layer protocol used by:

```text
APIs
websites
health endpoints
microservices
load balancers
```

Python can perform HTTP checks using:

```text
urllib
requests
httpx
```

depending on the project.

---

# 42. HTTP Health Check with urllib

```python
import urllib.request


def http_check(url):
    try:
        with urllib.request.urlopen(
            url,
            timeout=5,
        ) as response:

            return {
                "ok": (
                    200
                    <= response.status
                    < 300
                ),
                "status":
                    response.status,
            }

    except Exception as exc:
        return {
            "ok": False,
            "error": str(exc),
        }
```

---

# 43. HTTP Status Codes

Common operational meanings:

```text
200 -> success
201 -> created
204 -> success/no body
301/302 -> redirect
400 -> client request problem
401 -> authentication required
403 -> forbidden
404 -> not found
429 -> rate limited
500 -> server error
502 -> bad gateway
503 -> unavailable
504 -> gateway timeout
```

---

# 44. Health Endpoint Design

Typical:

```text
/health
```

or:

```text
/healthz
```

A simple health endpoint may indicate process readiness.

A deeper readiness endpoint may verify required dependencies.

---

# 45. Liveness vs Readiness

```text
Liveness
=
should this process be restarted?

Readiness
=
can this instance receive traffic?
```

Do not make liveness depend on every downstream system unless there is a strong reason.

---

# 46. HTTP Headers

Python:

```python
request = urllib.request.Request(
    "https://example.com/health",
    headers={
        "User-Agent":
            "devops-health-check"
    },
)
```

Headers may matter for:

```text
authentication
routing
content negotiation
tracing
application behavior
```

---

# 47. Authentication

For API automation, credentials may be:

```text
environment variables
secret manager
IAM
short-lived tokens
```

Avoid hardcoding:

```text
passwords
API keys
access tokens
```

in Python source.

---

# 48. HTTPS and TLS

TLS protects:

```text
confidentiality
integrity
server authentication
```

When troubleshooting HTTPS, separate:

```text
DNS
TCP
TLS
HTTP
application
```

---

# 49. TLS Troubleshooting

Useful command:

```bash
openssl s_client \
    -connect example.com:443 \
    -servername example.com
```

Inspect:

```text
certificate
chain
expiry
hostname
TLS negotiation
```

---

# 50. Python TLS Check

Python's `ssl` module provides TLS functionality.

```python
import socket
import ssl

context = ssl.create_default_context()

with socket.create_connection(
    ("example.com", 443),
    timeout=5,
) as sock:

    with context.wrap_socket(
        sock,
        server_hostname="example.com",
    ) as tls:

        print(
            tls.version()
        )
```

Do not disable certificate verification just to make a failing connection work.

---

# 51. Certificate Verification

Bad:

```python
ssl._create_unverified_context()
```

for production validation.

Better:

```python
ssl.create_default_context()
```

Certificate failures should be investigated rather than bypassed.

---

# 52. Certificate Expiry

Python can inspect certificate metadata using SSL libraries.

Operationally, monitor:

```text
expiry date
issuer
subject
SAN
chain
```

Certificate monitoring should happen before expiration.

---

# 53. SAN

Modern TLS hostname verification relies heavily on:

```text
Subject Alternative Name
```

If the hostname is not covered by the certificate SAN, validation can fail.

---

# 54. HTTP vs HTTPS Troubleshooting

```text
DNS resolves
 ↓
TCP 443 succeeds
 ↓
TLS succeeds
 ↓
HTTP returns 503
```

This means:

```text
network is probably working
application/backend is likely the next layer
```

---

# 55. `curl` as a Diagnostic Tool

```bash
curl -v https://example.com
```

Useful for seeing:

```text
DNS
TCP
TLS
HTTP
headers
redirects
```

Python can automate selected parts of the same workflow.

---

# 56. HTTP Redirects

Check:

```text
HTTP → HTTPS
HTTP → another host
```

A health check should know whether redirects are expected.

Do not blindly treat every 3xx response as healthy.

---

# 57. HTTP Timeout

Always define a timeout:

```python
urllib.request.urlopen(
    url,
    timeout=5,
)
```

An unavailable endpoint should not hang your deployment pipeline indefinitely.

---

# 58. Retry Strategy

Transient network failures may justify retries.

Use:

```text
attempt
 ↓
backoff
 ↓
attempt
 ↓
backoff
 ↓
final failure
```

Do not retry indefinitely.

---

# 59. Exponential Backoff

Example:

```text
1 sec
2 sec
4 sec
8 sec
```

Add jitter in distributed systems to avoid synchronized retries.

---

# 60. Retryable vs Non-Retryable Errors

Potentially retryable:

```text
connection timeout
temporary DNS failure
503
429
```

Usually not fixed by retry:

```text
401
403
invalid request
bad configuration
certificate hostname mismatch
```

Exact behavior depends on the application.

---

# 61. Network Retry Example

```python
import time
import urllib.request


def request_with_retry(
    url,
    attempts=3,
):
    delay = 1

    for attempt in range(
        attempts
    ):
        try:
            with urllib.request.urlopen(
                url,
                timeout=5,
            ) as response:

                return response.status

        except OSError:
            if (
                attempt
                == attempts - 1
            ):
                raise

            time.sleep(delay)
            delay *= 2
```

Production clients should classify exceptions and status codes carefully.

---

# 62. Avoid Retry Storms

If 500 servers all retry every second:

```text
backend failure
 ↓
500 clients retry
 ↓
more traffic
 ↓
backend overload
 ↓
worse failure
```

Use:

```text
backoff
jitter
timeouts
circuit breakers
```

---

# 63. Network Connectivity Matrix

For microservices:

```text
service A → service B
service A → database
service B → cache
service C → message broker
```

A Python script can test a defined matrix:

```text
source
destination
port
protocol
expected result
```

---

# 64. Connectivity Matrix Example

```python
checks = [
    (
        "api",
        "db.internal",
        5432,
    ),
    (
        "api",
        "redis.internal",
        6379,
    ),
    (
        "worker",
        "rabbitmq.internal",
        5672,
    ),
]
```

Use this during environment validation.

---

# 65. Parallel Connectivity Checks

For many independent checks, use:

```text
concurrent.futures
```

instead of sequential waits.

Example:

```python
from concurrent.futures import (
    ThreadPoolExecutor
)
```

Network checks are commonly I/O-bound.

---

# 66. Thread Pool Example

```python
from concurrent.futures import (
    ThreadPoolExecutor,
)
import socket


def check(item):
    host, port = item

    try:
        with socket.create_connection(
            (host, port),
            timeout=3,
        ):
            return (
                host,
                port,
                True,
            )

    except OSError:
        return (
            host,
            port,
            False,
        )


targets = [
    ("db.internal", 5432),
    ("redis.internal", 6379),
]

with ThreadPoolExecutor(
    max_workers=10
) as executor:

    for result in executor.map(
        check,
        targets,
    ):
        print(result)
```

---

# 67. Thread Count

Do not create hundreds of threads blindly.

Choose concurrency based on:

```text
number of endpoints
network latency
DNS behavior
remote rate limits
host resources
```

---

# 68. Async Networking

For very large numbers of network operations, asynchronous Python can be useful.

Common concepts:

```text
asyncio
async/await
async HTTP clients
connection pooling
```

Use async when the workload benefits from high concurrency.

---

# 69. Network Automation Libraries

Common Python libraries include:

```text
requests
httpx
paramiko
dnspython
psutil
scapy
netmiko
napalm
boto3
```

Choose libraries based on the exact problem.

Do not add dependencies unnecessarily.

---

# 70. Requests

`requests` is widely used for HTTP automation.

Example:

```python
import requests

response = requests.get(
    "https://example.com/health",
    timeout=5,
)

print(response.status_code)
```

Use a virtual environment and pin dependencies appropriately in production projects.

---

# 71. Requests Session

For repeated HTTP calls:

```python
import requests

session = requests.Session()

response = session.get(
    "https://example.com/health",
    timeout=5,
)
```

Sessions can reuse connections.

---

# 72. HTTP Connection Pooling

Repeated API calls benefit from:

```text
connection reuse
pooling
timeouts
```

This reduces connection overhead.

For production automation, use a client appropriate for the workload.

---

# 73. API Automation

DevOps often interacts with:

```text
AWS APIs
GitHub
GitLab
Jenkins
Kubernetes
monitoring systems
DNS providers
load balancers
ticketing systems
```

Network automation frequently means API automation rather than raw socket programming.

---

# 74. REST API Workflow

```text
authenticate
 ↓
GET resource
 ↓
validate state
 ↓
POST/PATCH change
 ↓
poll status
 ↓
verify
```

Python is excellent for this type of orchestration.

---

# 75. API Timeouts

Always set:

```python
timeout=10
```

or an appropriate timeout.

Never depend on an unlimited default timeout for operational automation.

---

# 76. API Error Handling

Handle:

```text
HTTP status
connection errors
timeouts
JSON parsing errors
authentication errors
rate limits
```

Example:

```python
response.raise_for_status()
```

when using `requests`.

---

# 77. JSON API

```python
response = requests.get(
    url,
    timeout=5,
)

data = response.json()
```

Validate the expected fields before using them.

---

# 78. API Authentication

Preferred approaches:

```text
IAM role
short-lived token
secret manager
environment injection
OIDC
```

Avoid:

```python
TOKEN = "secret-token"
```

in source code.

---

# 79. API Rate Limits

Many services return:

```text
429 Too Many Requests
```

A production client should:

```text
respect Retry-After
backoff
limit concurrency
cache where appropriate
```

---

# 80. DNS Automation

Common automation tasks:

```text
resolve host
validate record
compare expected IP
check CNAME
verify DNS before deployment
```

Example:

```python
import socket

expected = "10.0.10.25"

actual = socket.gethostbyname(
    "api.internal"
)

if actual != expected:
    raise RuntimeError(
        "Unexpected DNS target"
    )
```

Avoid brittle exact-IP assumptions when DNS intentionally returns multiple addresses.

---

# 81. Multiple DNS Addresses

```python
import socket

addresses = {
    item[4][0]
    for item in socket.getaddrinfo(
        "api.example.com",
        443,
    )
}

print(addresses)
```

This handles multiple A/AAAA results.

---

# 82. DNS Round Robin

A hostname can resolve to:

```text
10.0.0.10
10.0.0.11
10.0.0.12
```

Do not assume:

```text
one hostname = one IP
```

---

# 83. Network Interface Monitoring

Using `psutil`:

```python
import psutil

stats = psutil.net_if_stats()

for name, stat in stats.items():
    print(
        name,
        stat.isup,
        stat.speed,
        stat.mtu,
    )
```

This can support host preflight checks.

---

# 84. Network Counters

```python
import psutil

counters = (
    psutil.net_io_counters(
        pernic=True
    )
)

for name, value in counters.items():
    print(
        name,
        value.bytes_sent,
        value.bytes_recv,
    )
```

Useful for operational reporting.

---

# 85. Packet Errors

Network counters may include:

```text
packets sent
packets received
errors
drops
```

High errors/drops can indicate interface or network problems.

Interpret metrics alongside:

```text
cloud networking
driver
traffic load
application behavior
```

---

# 86. Linux Neighbor Table

Linux:

```bash
ip neigh
```

This shows neighbor/ARP-related information.

Useful for local network troubleshooting.

---

# 87. ARP

ARP maps IPv4 addresses to MAC addresses on local Ethernet-style networks.

In cloud networks, virtualization means you should not assume traditional physical LAN behavior.

Still, `ip neigh` can be useful for diagnosing local connectivity.

---

# 88. ICMP

`ping` uses ICMP echo requests.

Important:

```text
ping failure
≠
application unreachable
```

Firewalls commonly block ICMP while allowing TCP/HTTPS.

Never use ping as the only connectivity test.

---

# 89. Python Ping Caveat

Python's standard library does not provide a simple universal privileged ICMP ping API.

For DevOps automation, prefer:

```text
TCP connection
HTTP health
subprocess ping where appropriate
```

based on the actual dependency being tested.

---

# 90. TCP Is Usually Better for Service Checks

If the application requires:

```text
TCP 443
```

test:

```text
TCP 443
```

rather than:

```text
ICMP
```

because the service dependency is TCP.

---

# 91. Port Scanner vs Health Checker

A health checker asks:

```text
Is expected service reachable?
```

A port scanner asks:

```text
What ports are open?
```

Do not turn a production health script into an unrestricted port scanner.

---

# 92. Security Principle

Network automation should have:

```text
least privilege
defined targets
defined ports
timeouts
rate limits
logging
```

Avoid broad scanning unless explicitly authorized.

---

# 93. Firewall Concepts

Linux firewall technologies may include:

```text
nftables
iptables
firewalld
ufw
```

The exact tool depends on the distribution.

Python can inspect firewall state but should not blindly modify firewall rules.

---

# 94. Firewalld

Common commands:

```bash
firewall-cmd --state
firewall-cmd --list-all
```

Python can invoke controlled commands where appropriate.

---

# 95. nftables

Modern Linux systems increasingly use:

```text
nftables
```

Inspect:

```bash
nft list ruleset
```

Understand the active firewall stack before making changes.

---

# 96. Firewall Troubleshooting

If:

```text
service listening
```

but remote connection fails:

check:

```text
local firewall
cloud security group
network ACL
route
load balancer
remote firewall
```

---

# 97. AWS Network Context

For AWS DevOps engineers, network troubleshooting may involve:

```text
VPC
subnet
route table
security group
network ACL
internet gateway
NAT gateway
load balancer
DNS
```

Python can automate AWS API checks using `boto3`.

---

# 98. AWS SDK with Python

```python
import boto3

ec2 = boto3.client(
    "ec2"
)

response = ec2.describe_vpcs()

for vpc in response["Vpcs"]:
    print(
        vpc["VpcId"],
        vpc["CidrBlock"],
    )
```

Prefer IAM roles rather than static AWS access keys on AWS-hosted workloads.

---

# 99. AWS Security Groups

A preflight tool can inspect:

```text
security group
inbound rules
outbound rules
ports
source CIDRs
```

Example logic:

```text
Application
 ↓
requires TCP 443
 ↓
security group permits 443
 ↓
route exists
 ↓
target reachable
```

---

# 100. AWS Route Tables

Python can query route tables:

```python
ec2.describe_route_tables()
```

Use this to validate:

```text
default route
NAT route
IGW route
VPC peering route
Transit Gateway route
```

---

# 101. AWS Subnet Validation

A deployment validator can confirm:

```text
subnet belongs to expected VPC
subnet CIDR is expected
subnet is in expected AZ
route table is correct
security group is correct
```

This is a strong real-world DevOps automation use case.

---

# 102. AWS ALB Health

Python can use AWS APIs to inspect:

```text
load balancer
target groups
target health
listeners
```

A deployment pipeline can verify that new targets become healthy before continuing.

---

# 103. ALB Target Health

Conceptually:

```text
Deploy application
 ↓
register target
 ↓
ALB health check
 ↓
healthy
 ↓
receive traffic
```

Python can poll target health through AWS APIs.

---

# 104. AWS NAT Gateway

Private subnet outbound traffic may use:

```text
private subnet
 ↓
route table
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

If package installation fails on private instances, inspect the entire chain.

---

# 105. NAT Troubleshooting

Check:

```text
subnet route
NAT gateway
NAT subnet route
internet gateway
security group
network ACL
DNS
```

A Python script can validate configuration through AWS APIs.

---

# 106. DNS in AWS

Common components:

```text
Route 53
private hosted zones
public hosted zones
VPC DNS
service discovery
```

Python can use:

```python
boto3.client(
    "route53"
)
```

for controlled DNS automation.

---

# 107. Route 53 Automation

Possible tasks:

```text
validate record
create record
update record
remove record
wait for change
verify resolution
```

Always validate the intended hosted zone and record before modifying DNS.

---

# 108. DNS Change Safety

Before changing:

```text
record name
record type
TTL
value
routing policy
health check
```

Then:

```text
apply
 ↓
wait
 ↓
resolve
 ↓
verify
```

---

# 109. Kubernetes Networking

Kubernetes networking includes:

```text
Pod IP
Service IP
Node IP
Ingress/LoadBalancer
CoreDNS
CNI
NetworkPolicy
```

Python can help with custom diagnostics and API-based validation.

---

# 110. Kubernetes DNS

Inside a cluster:

```text
service.namespace.svc.cluster.local
```

DNS is usually handled by:

```text
CoreDNS
```

A Pod that cannot resolve a service may have:

```text
CoreDNS
network
CNI
DNS configuration
```

issues.

---

# 111. Kubernetes Service Connectivity

Troubleshooting:

```text
Pod
 ↓
Service
 ↓
EndpointSlice
 ↓
Pod
```

Check:

```bash
kubectl get svc
kubectl get endpointslice
kubectl get pods -o wide
```

---

# 112. Kubernetes NetworkPolicy

NetworkPolicy can restrict:

```text
ingress
egress
namespace traffic
pod traffic
```

A successful local service check does not prove another namespace can reach it.

---

# 113. Kubernetes Ingress

Typical path:

```text
Client
 ↓
DNS
 ↓
Load Balancer
 ↓
Ingress
 ↓
Service
 ↓
Pod
```

Troubleshoot from outside to inside.

---

# 114. ALB Ingress

In AWS EKS:

```text
Internet
 ↓
Route 53
 ↓
ALB
 ↓
Target
 ↓
Kubernetes workload
```

The target may be:

```text
Pod
or
Node
```

depending on the configured target mode.

---

# 115. Network Troubleshooting in EKS

A practical order:

```text
DNS
 ↓
ALB
 ↓
security groups
 ↓
target health
 ↓
Ingress
 ↓
Service
 ↓
EndpointSlice
 ↓
Pod
 ↓
container
```

Python can automate API-based checks around this path.

---

# 116. Network Preflight for Deployment

Before deployment:

```text
DNS resolves
database port reachable
cache reachable
broker reachable
required AWS resources exist
TLS valid
```

If preflight fails:

```text
stop deployment
```

rather than deploying a known-broken configuration.

---

# 117. Preflight Script Design

```text
input
 ↓
validate
 ↓
run independent checks
 ↓
collect results
 ↓
summarize
 ↓
exit 0/1
```

This works well in CI/CD.

---

# 118. CI/CD Network Preflight

Example:

```text
Jenkins/GitHub Actions
        ↓
Python preflight
        ↓
DNS
TCP
HTTPS
AWS API
        ↓
PASS / FAIL
        ↓
deployment
```

---

# 119. Exit Codes

Use:

```text
0 = success
non-zero = failure
```

Example:

```python
import sys

if not all_checks_passed:
    sys.exit(1)

sys.exit(0)
```

CI systems can then stop deployment automatically.

---

# 120. Structured Preflight Output

Example:

```json
{
  "dns": true,
  "database": true,
  "redis": true,
  "api": true,
  "tls": true
}
```

This is easier to parse than free-form shell output.

---

# 121. Network Validation with YAML

A configuration file could define:

```yaml
checks:
  - name: database
    host: db.internal
    port: 5432

  - name: redis
    host: redis.internal
    port: 6379

  - name: api
    url: https://api.internal/health
```

Python can load this and execute the checks.

---

# 122. Configuration-Driven Checks

Architecture:

```text
YAML
 ↓
Python validator
 ↓
DNS/TCP/HTTP checks
 ↓
JSON report
```

Benefits:

```text
reusable
environment-specific
easy to review
CI-friendly
```

---

# 123. Environment Profiles

Example:

```text
dev.yaml
qa.yaml
prod.yaml
```

Same Python code:

```text
different configuration
```

This avoids duplicating scripts.

---

# 124. Secrets in Network Configuration

Do not store:

```text
password
API token
private key
```

inside:

```text
Git-tracked YAML
```

Use:

```text
secret manager
environment injection
OIDC
IAM
```

---

# 125. SSH Networking

Python can automate SSH using:

```text
Paramiko
Fabric
subprocess + ssh
```

Use SSH only when required.

Prefer APIs/configuration management for scalable infrastructure.

---

# 126. Paramiko

Typical:

```python
import paramiko

client = paramiko.SSHClient()

client.set_missing_host_key_policy(
    paramiko.RejectPolicy()
)

client.connect(
    hostname,
    username=username,
    key_filename=key_file,
    timeout=10,
)
```

Avoid blindly accepting unknown host keys in production.

---

# 127. SSH Host Key Security

Bad:

```python
AutoAddPolicy()
```

without considering trust.

Better:

```text
known_hosts
trusted host keys
managed host identity
```

This protects against man-in-the-middle attacks.

---

# 128. SSH Command Execution

```python
stdin, stdout, stderr = (
    client.exec_command(
        "systemctl is-active myapp"
    )
)

print(
    stdout.read().decode()
)
```

Capture:

```text
stdout
stderr
exit status
```

---

# 129. SSH Automation at Scale

For many hosts:

```text
inventory
 ↓
parallel connections
 ↓
command
 ↓
collect results
 ↓
aggregate report
```

Use connection limits and timeouts.

Do not create unlimited concurrent SSH connections.

---

# 130. SSH vs Ansible

Use Ansible when you need:

```text
inventory
idempotency
configuration management
privilege escalation
repeatable state
```

Use Python/Paramiko when you need:

```text
custom programmatic workflow
API integration
specialized logic
```

---

# 131. Network Device Automation

Python libraries such as:

```text
Netmiko
NAPALM
```

can automate network devices.

Use cases:

```text
configuration
show commands
inventory
validation
backup
```

---

# 132. Network Device Safety

Before changing network configuration:

```text
backup
validate
change
test
rollback
```

A bad routing/firewall change can disconnect the automation system itself.

---

# 133. Configuration Backup

For network devices:

```text
current config
 ↓
timestamped backup
 ↓
Git/object storage
```

Never overwrite the only known-good configuration.

---

# 134. Network Change Validation

After change:

```text
interface up
route exists
neighbor reachable
service reachable
expected traffic works
```

Then mark the change successful.

---

# 135. Transactional Thinking

A network change should be treated as:

```text
prepare
 ↓
validate
 ↓
apply
 ↓
verify
 ↓
commit
or
rollback
```

This is especially important for production routing and firewall changes.

---

# 136. Network Automation Idempotency

Good automation:

```text
run once -> desired state
run twice -> same desired state
```

Example:

```text
ensure DNS record exists
```

instead of:

```text
blindly create record every run
```

---

# 137. Idempotent DNS Example

Conceptually:

```python
current = get_record()

if current != desired:
    update_record()
```

This avoids unnecessary changes.

---

# 138. Idempotent Firewall Example

Conceptually:

```text
rule exists?
  yes -> do nothing
  no  -> create
```

Avoid appending duplicate rules every execution.

---

# 139. Idempotent Network Interface Configuration

Before changing:

```text
IP
route
MTU
```

check current state.

Then apply only required changes.

---

# 140. Network Automation Logging

Log:

```text
timestamp
operator/tool
target
action
previous state
desired state
result
error
```

Never log secrets.

---

# 141. Structured Logging

Example:

```python
import logging

logging.basicConfig(
    level=logging.INFO,
)

logging.info(
    "Checking endpoint %s:%s",
    host,
    port,
)
```

For larger systems, JSON logging can improve centralized analysis.

---

# 142. Network Automation Audit Trail

For production changes, record:

```text
who
what
when
where
why
result
```

This supports:

```text
incident analysis
compliance
change management
rollback
```

---

# 143. Dry Run

A safe automation tool should support:

```bash
python network_change.py \
    --dry-run
```

Dry run should show:

```text
what would change
```

without modifying the system.

---

# 144. Argparse Example

```python
import argparse

parser = argparse.ArgumentParser()

parser.add_argument(
    "--dry-run",
    action="store_true",
)

args = parser.parse_args()

if args.dry_run:
    print(
        "No changes will be made"
    )
```

---

# 145. Network Automation CLI

A production CLI might support:

```text
--environment
--config
--timeout
--retries
--dry-run
--verbose
--output
```

Validate every option.

---

# 146. Network Inventory

A Python inventory tool may collect:

```text
hostname
IP addresses
interfaces
routes
DNS
open expected ports
cloud metadata
service state
```

Output:

```text
JSON
CSV
YAML
```

---

# 147. Inventory Example

```json
{
  "hostname": "app-01",
  "interfaces": [
    "eth0"
  ],
  "addresses": [
    "10.0.10.25"
  ],
  "services": {
    "nginx": "active"
  }
}
```

---

# 148. Hostname

Python:

```python
import socket

hostname = socket.gethostname()

print(hostname)
```

Fully qualified naming depends on system DNS configuration.

---

# 149. Reverse DNS

```python
import socket

name = socket.gethostbyaddr(
    "10.0.10.25"
)

print(name)
```

Reverse DNS may not exist.

Do not treat missing PTR records as a connectivity failure unless the application requires them.

---

# 150. Local Hostname Verification

A preflight can verify:

```text
hostname
expected environment
expected IP range
expected service role
```

This helps prevent running production automation against the wrong host.

---

# 151. Environment Safety

For dangerous network changes:

```text
DEV
QA
STAGING
PROD
```

must be explicit.

Never infer production solely from hostname patterns.

---

# 152. Production Confirmation

A high-risk tool can require:

```bash
--environment production
--confirm
```

and verify both against known infrastructure metadata.

---

# 153. Network Automation Failure Modes

Common failures:

```text
DNS timeout
TCP timeout
TLS error
connection refused
connection reset
route missing
firewall denied
authentication failed
rate limit
remote unavailable
```

Classify the failure before deciding how to retry.

---

# 154. Connection Refused

Usually means:

```text
host reachable
TCP connection reached target
no process listening
or firewall actively rejected
```

Check:

```bash
ss -lntp
```

on the destination when possible.

---

# 155. Connection Timeout

Could indicate:

```text
routing
firewall
security group
network ACL
service unavailable
packet loss
wrong IP
```

Timeout is less specific than refusal.

---

# 156. Connection Reset

A reset can occur because:

```text
remote application closed connection
firewall/device reset
protocol mismatch
TLS issue
application crash
```

Inspect both endpoints.

---

# 157. No Route to Host

Possible causes:

```text
missing route
interface issue
network namespace
routing policy
unreachable gateway
```

Start with:

```bash
ip route
ip route get <destination>
```

---

# 158. Name Resolution Failure

If:

```bash
curl https://api.example.com
```

fails with name resolution:

Test:

```bash
getent hosts api.example.com
dig api.example.com
```

Then inspect:

```text
DNS configuration
resolver
network
```

---

# 159. TLS Handshake Failure

Separate:

```text
DNS -> okay
TCP -> okay
TLS -> fails
```

Then inspect:

```text
certificate
hostname
CA
TLS versions
SNI
clock
```

---

# 160. Clock and TLS

Incorrect system time can break TLS certificate validation.

Check:

```bash
date
timedatectl
```

A Python network health tool can report clock-related TLS errors but should not disable verification.

---

# 161. Proxy Environments

Enterprise environments may require:

```text
HTTP_PROXY
HTTPS_PROXY
NO_PROXY
```

A Python HTTP client may behave differently depending on environment configuration.

Understand proxy behavior before diagnosing connectivity.

---

# 162. NO_PROXY

Kubernetes environments often require correct:

```text
NO_PROXY
```

entries for:

```text
cluster services
Pod CIDRs
service CIDRs
localhost
metadata endpoints
```

Exact requirements depend on the environment.

---

# 163. Proxy Troubleshooting

Compare:

```text
shell
Python process
systemd service
CI runner
container
```

They may have different proxy environments.

---

# 164. Network Namespaces

Containers can have separate network namespaces.

Therefore:

```text
host can reach endpoint
```

does not always mean:

```text
container can reach endpoint
```

Test from the actual execution environment.

---

# 165. Container Network Debugging

Check:

```text
container IP
routes
DNS
network namespace
security policy
```

For Kubernetes:

```text
Pod network
CNI
NetworkPolicy
CoreDNS
```

---

# 166. Docker Network

Useful commands:

```bash
docker network ls
docker network inspect <network>
```

Python can use Docker SDK for controlled automation if required.

---

# 167. Docker SDK

Example concept:

```python
import docker

client = docker.from_env()

networks = client.networks.list()

for network in networks:
    print(network.name)
```

Use an API/SDK instead of parsing CLI output when building larger automation.

---

# 168. Kubernetes API

Python can interact with Kubernetes using:

```text
kubernetes Python client
```

Possible tasks:

```text
inspect Services
inspect EndpointSlices
check Pods
read Events
validate NetworkPolicies
```

---

# 169. Kubernetes Network Preflight

A custom tool can check:

```text
Service exists
EndpointSlices contain ready endpoints
DNS resolves
port is reachable
Ingress exists
```

This can be integrated into CI/CD.

---

# 170. Service Endpoint Verification

Example workflow:

```text
Deployment
 ↓
Pod Ready
 ↓
EndpointSlice updated
 ↓
Service routes traffic
 ↓
Ingress/ALB healthy
```

A network automation script can verify these stages.

---

# 171. Production Network Health Report

Example:

```json
{
  "dns": "PASS",
  "tcp_database": "PASS",
  "tcp_redis": "PASS",
  "https_api": "PASS",
  "tls": "PASS",
  "aws_target_health": "PASS"
}
```

This is useful in deployment gates.

---

# 172. Deployment Gate

```python
if not all_checks_pass:
    raise SystemExit(
        "Network preflight failed"
    )
```

CI receives a non-zero exit code and stops deployment.

---

# 173. Network Automation Testing

Unit test:

```text
CIDR parsing
IP validation
configuration parsing
result classification
retry logic
```

Integration test:

```text
real DNS
real endpoint
real AWS API
```

Keep production credentials out of unit tests.

---

# 174. Mocking Network Calls

Use mocks for:

```text
DNS failures
timeouts
HTTP 500
HTTP 503
connection refused
AWS API errors
```

This lets you test failure handling safely.

---

# 175. Test Retry Behavior

Verify:

```text
first failure
second failure
success
```

and:

```text
all attempts fail
```

The tool should terminate predictably.

---

# 176. Test Timeout Behavior

Ensure:

```text
unresponsive endpoint
```

does not hang indefinitely.

Use short test timeouts.

---

# 177. Test Invalid Configuration

Examples:

```text
invalid IP
invalid CIDR
invalid port
missing host
invalid URL
```

Fail early with a clear message.

---

# 178. Port Validation

Valid TCP/UDP ports:

```text
1–65535
```

Python:

```python
def valid_port(port):
    return (
        isinstance(port, int)
        and 1 <= port <= 65535
    )
```

---

# 179. Host Validation

Avoid accepting arbitrary dangerous shell strings.

Instead:

```python
host = host.strip()
```

and validate according to whether the expected value is:

```text
hostname
IP
URL
```

Do not pass untrusted input to shell commands.

---

# 180. URL Validation

A network automation tool should verify:

```text
scheme
hostname
optional port
```

For example:

```text
https://api.example.com:8443/health
```

---

# 181. SSRF Awareness

A network automation service that accepts user-provided URLs can accidentally become an SSRF tool.

Be careful with access to:

```text
localhost
127.0.0.1
private IP ranges
cloud metadata endpoints
internal services
```

Use strict destination allowlists where user input is involved.

---

# 182. Cloud Metadata

Cloud metadata endpoints can expose sensitive instance information.

Do not build generic tools that allow arbitrary users to request internal URLs.

Use IAM and provider-supported metadata access securely.

---

# 183. Network Tool Security

A production network automation tool should define:

```text
allowed destinations
allowed ports
allowed protocols
maximum concurrency
maximum request size
timeouts
authentication
audit logging
```

---

# 184. Logging Network Errors

Useful:

```python
import logging

logging.error(
    "TCP check failed host=%s port=%s error=%s",
    host,
    port,
    exc,
)
```

Avoid logging:

```text
Authorization headers
passwords
tokens
private keys
cookies
```

---

# 185. Network Automation Metrics

A custom tool can expose:

```text
check duration
success count
failure count
timeout count
retry count
```

If the tool is long-running, Prometheus metrics may be appropriate.

For short CI jobs, structured logs/artifacts may be enough.

---

# 186. Latency Measurement

```python
import time
import socket

start = time.monotonic()

with socket.create_connection(
    ("example.com", 443),
    timeout=5,
):
    pass

elapsed = (
    time.monotonic()
    - start
)

print(elapsed)
```

Use `monotonic()` for elapsed durations.

---

# 187. Latency vs Availability

A service can be:

```text
available
but slow
```

Therefore network checks should sometimes measure:

```text
success
latency
error rate
```

not just Boolean availability.

---

# 188. HTTP Latency

```python
import time
import requests

start = time.monotonic()

response = requests.get(
    url,
    timeout=5,
)

latency = (
    time.monotonic()
    - start
)

print(
    response.status_code,
    latency,
)
```

---

# 189. SLA/SLO Context

For production services:

```text
availability
latency
error rate
```

can be measured against:

```text
SLA
SLO
SLI
```

Network automation can contribute to synthetic checks.

---

# 190. Synthetic Monitoring

A synthetic check acts like a user:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
expected response
```

This can detect problems before users report them.

For continuous monitoring, use an appropriate monitoring platform rather than an unmanaged infinite Python loop.

---

# 191. DevOps Daily Script — DNS + TCP + HTTP

```python
import socket
import urllib.request


def check_dns(host):
    try:
        socket.getaddrinfo(
            host,
            None,
        )
        return True
    except OSError:
        return False


def check_tcp(host, port):
    try:
        with socket.create_connection(
            (host, port),
            timeout=3,
        ):
            return True
    except OSError:
        return False


def check_http(url):
    try:
        with urllib.request.urlopen(
            url,
            timeout=5,
        ) as response:
            return 200 <= response.status < 300
    except OSError:
        return False
```

---

# 192. DevOps Daily Script — Dependency Check

```python
checks = [
    {
        "name": "postgres",
        "host": "db.internal",
        "port": 5432,
    },
    {
        "name": "redis",
        "host": "cache.internal",
        "port": 6379,
    },
]
```

Loop through these checks and generate a deployment report.

---

# 193. DevOps Daily Script — Network Preflight

```python
def preflight(checks):
    results = []

    for check in checks:
        ok = check_tcp(
            check["host"],
            check["port"],
        )

        results.append(
            {
                "name":
                    check["name"],
                "ok": ok,
            }
        )

    return results
```

---

# 194. DevOps Daily Script — JSON Report

```python
import json

report = {
    "environment": "production",
    "checks": results,
}

print(
    json.dumps(
        report,
        indent=2,
    )
)
```

CI can save this as an artifact.

---

# 195. DevOps Daily Script — Exit on Failure

```python
import sys

failed = [
    item
    for item in results
    if not item["ok"]
]

if failed:
    print(
        "Network preflight failed"
    )
    sys.exit(1)

print(
    "Network preflight passed"
)
```

---

# 196. DevOps Daily Script — Concurrent Checks

Use:

```python
from concurrent.futures import (
    ThreadPoolExecutor
)
```

for many independent network checks.

Keep concurrency bounded.

---

# 197. DevOps Daily Script — URL Validation

A deployment validator may check:

```text
DNS
TLS
HTTP status
response latency
expected response content
```

Example expected response:

```text
"status": "healthy"
```

Validate content carefully rather than merely accepting HTTP 200.

---

# 198. Expected Response Validation

```python
data = response.json()

if data.get("status") != "healthy":
    raise RuntimeError(
        "Application unhealthy"
    )
```

This is stronger than checking only status code.

---

# 199. Network Automation Architecture

A production preflight tool:

```text
YAML config
    ↓
Python CLI
    ↓
Validation
    ↓
DNS checks
    ↓
TCP checks
    ↓
TLS checks
    ↓
HTTP checks
    ↓
AWS/Kubernetes API checks
    ↓
Structured report
    ↓
Exit code
    ↓
CI/CD gate
```

---

# 200. Network Automation with Your DevOps Stack

A practical environment may look like:

```text
GitHub
   ↓
Jenkins / GitHub Actions
   ↓
Python preflight
   ↓
Terraform infrastructure
   ↓
AWS VPC / EKS / ALB
   ↓
Kubernetes services
   ↓
Prometheus + Grafana
   ↓
ELK
```

Python acts as the validation/orchestration layer.

---

# 201. Terraform + Python

Terraform should remain responsible for:

```text
infrastructure desired state
```

Python can perform:

```text
prechecks
post-deployment validation
custom API verification
reporting
```

Do not duplicate the entire Terraform engine in Python.

---

# 202. Ansible + Python

Ansible handles:

```text
configuration
packages
services
files
users
```

Python handles:

```text
custom validation
network tests
API workflows
complex data processing
```

They complement each other.

---

# 203. Jenkins + Python

Example:

```text
Build
 ↓
Unit tests
 ↓
SonarQube
 ↓
Trivy
 ↓
Artifact
 ↓
Deploy
 ↓
Python network preflight
 ↓
Health verification
```

A failed network preflight should fail the deployment gate.

---

# 204. GitHub Actions + Python

A workflow can execute:

```yaml
- name: Network preflight
  run: |
    python scripts/network_preflight.py \
      --environment staging
```

The script returns:

```text
0 -> continue
non-zero -> stop
```

---

# 205. DevSecOps Network Checks

Security-oriented checks may validate:

```text
TLS
certificate expiry
expected ports
public exposure
security groups
network policies
```

Avoid treating security checks as a substitute for proper security tooling.

---

# 206. Public Exposure Check

A cloud validation script can identify resources that are unintentionally:

```text
public
```

Examples:

```text
public IP
0.0.0.0/0
public load balancer
public bucket endpoint
```

Always interpret findings against intended architecture.

---

# 207. Security Group Review

A dangerous rule might be:

```text
TCP 22
0.0.0.0/0
```

Whether this is acceptable depends on architecture, but production SSH access should generally be restricted or replaced with stronger access mechanisms.

---

# 208. Port Exposure Review

A Python tool can compare:

```text
expected ports
vs
configured ports
```

Example:

```text
Expected:
443

Found:
22
80
443
8080
```

Unexpected exposure should be investigated.

---

# 209. Network Configuration Drift

Desired:

```text
443 open
22 restricted
8080 private
```

Actual:

```text
443 open
22 public
8080 public
```

Python can report drift, while Terraform/Ansible can enforce desired state.

---

# 210. Drift Detection Workflow

```text
desired configuration
        ↓
actual infrastructure
        ↓
Python comparison
        ↓
drift report
        ↓
Terraform/Ansible remediation
```

---

# 211. Network Automation and GitOps

Network configuration can follow:

```text
Git
 ↓
review
 ↓
CI validation
 ↓
deployment
 ↓
post-deployment checks
 ↓
observability
```

Python can be one of the CI validation components.

---

# 212. Production Incident — DNS Failure

Scenario:

```text
Application suddenly cannot reach database
```

Check:

```bash
getent hosts db.internal
dig db.internal
```

Then compare:

```text
DNS record
resolver
VPC DNS
network path
```

Do not immediately restart the application.

---

# 213. Production Incident — Database Port Unreachable

Check:

```bash
ip route get <db-ip>
nc -vz <db-ip> 5432
```

Then inspect:

```text
security group
NACL
route
database listener
network policy
```

---

# 214. Production Incident — API Returns 503

Path:

```text
Client
 ↓
DNS
 ↓
ALB
 ↓
target
 ↓
application
```

Check:

```text
ALB target health
service health
application logs
dependency health
```

---

# 215. Production Incident — HTTPS Certificate Failure

Check:

```text
DNS
TCP 443
certificate expiry
SAN
CA chain
server configuration
system clock
```

Do not disable TLS verification as a workaround.

---

# 216. Production Incident — EKS Service Unreachable

Check:

```text
Pod ready?
Service exists?
EndpointSlice populated?
NetworkPolicy?
CoreDNS?
Ingress?
ALB target healthy?
security groups?
```

Follow the actual traffic path.

---

# 217. Production Incident — Private EC2 Cannot Install Packages

Likely chain:

```text
DNS
 ↓
route table
 ↓
NAT Gateway
 ↓
IGW
 ↓
security rules
 ↓
external repository
```

Test each layer.

---

# 218. Production Incident — One AZ Has Connectivity Issues

Compare:

```text
route tables
subnets
NACLs
security groups
NAT
load balancer targets
AZ-specific resources
```

Use automation to compare expected configuration across AZs.

---

# 219. Production Incident — Intermittent Connection Failures

Possible causes:

```text
packet loss
load balancer target
connection limits
DNS rotation
ephemeral port exhaustion
backend saturation
network device issue
```

A single successful test is insufficient.

Run controlled repeated tests and correlate with metrics.

---

# 220. Ephemeral Ports

Client connections use ephemeral source ports.

High outbound connection rates can exhaust available ports.

Symptoms may include:

```text
connect failures
timeouts
```

Investigate:

```text
connection reuse
TIME_WAIT
NAT port usage
client behavior
```

Do not simply increase limits without understanding the architecture.

---

# 221. TIME_WAIT

TCP connections can remain in:

```text
TIME_WAIT
```

after closing.

Large volumes may indicate:

```text
high connection churn
```

Connection pooling can reduce unnecessary churn.

---

# 222. Connection Pooling

For HTTP/database clients:

```text
reuse connections
```

instead of:

```text
new TCP connection
for every request
```

This improves efficiency and can reduce network pressure.

---

# 223. Database Connection Pool

Application:

```text
API workers
 ↓
connection pool
 ↓
database
```

Do not allow:

```text
worker count × pool size
```

to exceed safe database capacity.

---

# 224. Network Saturation

Symptoms:

```text
high latency
packet loss
slow downloads
timeouts
```

Check:

```text
interface metrics
cloud metrics
application throughput
network device metrics
```

---

# 225. Packet Capture

Linux:

```bash
sudo tcpdump -i eth0 port 443
```

Use packet captures carefully.

They can contain:

```text
sensitive metadata
```

and large volumes of data.

---

# 226. Python and Packet Capture

Python can integrate with tools such as:

```text
scapy
```

for specialized packet analysis.

Use only where packet-level inspection is actually needed.

---

# 227. Scapy

Scapy can construct and inspect packets.

It is powerful but should be treated as an advanced networking tool.

For normal DevOps health checks:

```text
socket
requests/httpx
subprocess
AWS/Kubernetes SDKs
```

are usually sufficient.

---

# 228. Network Automation and Security Boundaries

Never build an unrestricted automation endpoint that accepts:

```text
arbitrary command
arbitrary destination
arbitrary port
```

This can become:

```text
remote command execution
SSRF
network scanning
```

Use explicit policies.

---

# 229. Safe Network Tool Design

```text
allowlist
 ↓
validate
 ↓
timeout
 ↓
rate limit
 ↓
execute
 ↓
audit
 ↓
return result
```

---

# 230. Common Mistakes

Avoid:

```text
ping-only troubleshooting
no timeout
infinite retries
hardcoded IP assumptions
hardcoded credentials
shell=True with user input
unbounded concurrency
blind firewall changes
blind DNS changes
TLS verification disabled
unrestricted network scanning
no rollback
```

---

# 231. Production Best Practices

```text
Use timeouts
Use bounded retries
Use exponential backoff
Use jitter for distributed retries
Use connection pooling
Use allowlists
Use least privilege
Use structured logs
Use dry-run for changes
Use idempotency
Use health checks
Use external monitoring
Use version-controlled configuration
```

---

# 232. Network Automation Checklist

```text
[ ] DNS validation
[ ] IP/CIDR validation
[ ] Route validation
[ ] TCP connectivity
[ ] HTTP health
[ ] TLS validation
[ ] Timeout handling
[ ] Retry/backoff
[ ] Structured output
[ ] Exit codes
[ ] Secrets protection
[ ] Logging
[ ] Dry-run
[ ] Idempotency
[ ] Rollback
```

---

# 233. Interview — How Do You Troubleshoot Network Connectivity?

**Answer:**

> I start from the actual traffic path. I verify DNS resolution, route selection, TCP connectivity, TLS if applicable, application response, and then cloud or Kubernetes network controls. I avoid assuming that ping success means the application is reachable.

---

# 234. Interview — How Do You Check a Port with Python?

**Answer:**

```python
import socket

try:
    with socket.create_connection(
        ("host", 443),
        timeout=5,
    ):
        print("reachable")
except OSError:
    print("unreachable")
```

I always use a timeout and classify the failure appropriately.

---

# 235. Interview — How Do You Automate DNS Checks?

**Answer:**

> I can use Python's `socket` module for basic resolution or a DNS library such as `dnspython` when I need record-level control. I compare the result against expected configuration without assuming a hostname always maps to a single IP.

---

# 236. Interview — How Do You Handle Network Retries?

**Answer:**

> I classify retryable failures, use bounded retries, exponential backoff, and jitter where appropriate. I avoid retrying authentication or configuration errors and avoid infinite retries because they can amplify an outage.

---

# 237. Interview — How Would You Validate an API Before Deployment?

**Answer:**

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP status
 ↓
response body
 ↓
latency
```

I would make the script return a non-zero exit code if the required checks fail so CI/CD can stop the deployment.

---

# 238. Interview — How Do You Troubleshoot an AWS Private Subnet?

**Answer:**

> I verify the subnet's route table, NAT or other intended egress path, security groups, NACLs, DNS resolution, and destination reachability. I also confirm the destination is actually reachable from that subnet and that the application is using the expected address.

---

# 239. Interview — How Do You Troubleshoot EKS Networking?

**Answer:**

> I follow the request path from DNS to the load balancer or ingress, then target health, Kubernetes Service, EndpointSlice, Pod, and finally the container. I also check CoreDNS, CNI behavior, NetworkPolicies, security groups, and node-level networking when appropriate.

---

# 240. Interview — How Would You Build a Network Preflight Tool?

**Answer:**

> I would make it configuration-driven. The YAML would define expected DNS names, TCP endpoints, HTTP health URLs, and cloud resources. Python would validate inputs, execute bounded concurrent checks, collect structured results, write JSON output, and return a non-zero exit code on failure. I would integrate it as a deployment gate.

---

# 241. Interview — What Is the Difference Between Connection Refused and Timeout?

**Answer:**

> Connection refused usually means the TCP attempt reached the destination but there was no listener or an active rejection. A timeout is less specific and can indicate routing, firewall filtering, packet loss, an unavailable destination, or other network-path problems.

---

# 242. Interview — Why Should You Not Disable TLS Verification?

**Answer:**

> TLS verification protects against certificate and hostname attacks. Disabling verification hides configuration problems and can create a security vulnerability. I investigate the certificate, hostname, CA chain, and system time instead.

---

# 243. Interview — When Would You Use Python Instead of Ansible?

**Answer:**

> I prefer Ansible for standard configuration-management tasks because it provides inventory, idempotency, and desired-state modules. I use Python when I need custom validation, complex API orchestration, specialized network checks, or data processing that does not fit naturally into a configuration-management task.

---

# 244. Interview — How Does Python Fit Into AWS DevOps?

**Answer:**

> Python with boto3 can validate and orchestrate AWS resources such as VPCs, subnets, route tables, security groups, ALBs, and target groups. I would use it for preflight and post-deployment verification while keeping Terraform responsible for infrastructure desired state.

---

# 245. Interview — How Do You Prevent a Network Automation Script From Becoming Dangerous?

**Answer:**

> I use strict input validation, destination and action allowlists, timeouts, bounded concurrency, least-privilege credentials, dry-run support, audit logging, and explicit environment selection. I also avoid shell injection and unrestricted user-controlled network destinations.

---

# 246. Interview — How Do You Make Network Automation Idempotent?

**Answer:**

> I first read the current state, compare it with the desired state, and make a change only when necessary. For example, before updating a DNS record I verify the current record and change it only if it differs from the desired configuration.

---

# 247. Interview — What Network Checks Belong in CI/CD?

Good candidates include:

```text
DNS
required TCP ports
TLS certificate
health endpoint
AWS target health
Kubernetes Service/Endpoint state
required dependencies
```

Avoid checks that require unsafe production mutations.

---

# 248. Interview — How Do You Handle Secrets?

**Answer:**

> I avoid hardcoding credentials. In AWS I prefer IAM roles and short-lived credentials. In CI/CD I use the platform's secret management or OIDC mechanisms. Secrets should not appear in source code, logs, command-line output, or generated reports.

---

# 249. Interview — How Do You Troubleshoot an ALB 503?

**Answer:**

> I verify DNS first, then ALB listener configuration, target group health, security groups, routing, and the backend application. In EKS I continue through ingress configuration, Service, EndpointSlice, Pod readiness, and application health. A 503 generally means I need to inspect the load-balancer/backend layer rather than simply restart Pods.

---

# 250. Interview — What Is a Good Network Health Check?

**Answer:**

> A good check tests the actual dependency. For an HTTPS API, I would validate DNS, establish TCP, validate TLS, send an HTTP request, verify the expected status/body, and measure latency. The check should have a timeout and bounded retries.

---

# 251. Real-World Project — Deployment Network Preflight

Architecture:

```text
GitHub
   ↓
Jenkins
   ↓
Build + Security Scan
   ↓
Artifact
   ↓
Deployment
   ↓
Python Network Preflight
   ├── DNS
   ├── Database TCP
   ├── Redis TCP
   ├── API HTTPS
   └── AWS Target Health
   ↓
Health Verification
   ↓
Prometheus / Grafana
```

---

# 252. Project Implementation

Example repository:

```text
network-preflight/
├── config/
│   ├── dev.yaml
│   ├── staging.yaml
│   └── prod.yaml
├── scripts/
│   ├── dns_check.py
│   ├── tcp_check.py
│   ├── http_check.py
│   └── preflight.py
├── tests/
└── README.md
```

---

# 253. Project Workflow

```text
1. Load environment config
2. Validate configuration
3. Resolve DNS
4. Check required ports
5. Check HTTPS endpoints
6. Validate TLS
7. Query AWS/Kubernetes APIs
8. Produce JSON report
9. Return CI exit code
```

---

# 254. Project Failure Handling

If:

```text
database unreachable
```

then:

```text
preflight = FAIL
deployment = STOP
```

This is better than:

```text
deploy first
discover failure later
```

---

# 255. Project Success Criteria

A successful deployment gate should prove:

```text
network dependencies reachable
TLS valid
application endpoint healthy
load-balancer target healthy
required cloud resources present
```

It should not attempt to prove every possible production condition.

---

# 256. Network Troubleshooting Decision Tree

```text
Cannot reach service
        |
        v
DNS resolves?
   |          |
  NO         YES
   |          |
DNS fix     route?
              |
              v
          TCP works?
          |        |
         NO       YES
          |        |
     route/firewall TLS?
                    |
                    v
                HTTP works?
                 |       |
                NO      YES
                 |       |
          app/backend   healthy
```

---

# 257. Layered Troubleshooting

Use this mental model:

```text
Layer 7 -> HTTP/API
Layer 6 -> TLS/serialization
Layer 4 -> TCP/UDP
Layer 3 -> IP/routing
Layer 2 -> interface/neighbor
Layer 1 -> physical/cloud underlying infrastructure
```

In cloud environments, the lower layers are abstracted, but the troubleshooting logic remains useful.

---

# 258. Network Troubleshooting Principle

Do not ask:

```text
"Why is the application down?"
```

only.

Ask:

```text
"At which layer does the request fail?"
```

Then test that layer directly.

---

# 259. Network Automation Principle

A good DevOps network script should be:

```text
small
focused
safe
observable
testable
repeatable
idempotent
```

Avoid building one enormous script that changes:

```text
DNS
firewall
routes
AWS
Kubernetes
application
```

without clear boundaries.

---

# 260. Final Network Automation Mental Model

```text
IP
 ↓
CIDR
 ↓
Interface
 ↓
Route
 ↓
DNS
 ↓
TCP/UDP
 ↓
TLS
 ↓
HTTP
 ↓
Cloud networking
 ↓
Kubernetes networking
 ↓
Application
```

For automation:

```text
VALIDATE
 ↓
CONNECT
 ↓
VERIFY
 ↓
REPORT
```

For changes:

```text
READ CURRENT STATE
 ↓
COMPARE DESIRED STATE
 ↓
DRY RUN
 ↓
APPLY
 ↓
VERIFY
 ↓
ROLLBACK IF SAFE
```

> **Python is most valuable in DevOps networking when it turns repeated connectivity validation, infrastructure API checks, deployment gates, and complex troubleshooting workflows into safe, repeatable automation.**
