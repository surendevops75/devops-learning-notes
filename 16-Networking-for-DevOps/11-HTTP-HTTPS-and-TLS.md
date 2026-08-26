# 16-Networking-for-DevOps
# 11-HTTP-HTTPS-and-TLS

## 1. Purpose

HTTP, HTTPS and TLS are core networking technologies for DevOps engineers.

A production request commonly follows:

```text
Client
  |
  | DNS
  v
IP address
  |
  | TCP
  v
TLS
  |
  | HTTPS
  v
Load Balancer / Reverse Proxy
  |
  v
Application
```

This file explains the complete path from HTTP fundamentals through TLS, Kubernetes, AWS ALB/NLB, Nginx, EKS, troubleshooting and production architecture.

---

## 2. What Is HTTP?

HTTP means:

```text
Hypertext Transfer Protocol
```

It is an application-layer protocol used for communication between clients and servers.

Typical examples:

```text
GET /products
POST /orders
PUT /users/123
DELETE /cart/123
```

---

## 3. HTTP Request-Response Model

Basic flow:

```text
Client
   |
   | HTTP Request
   v
Server
   |
   | HTTP Response
   v
Client
```

A request contains:

```text
method
target/path
headers
optional body
```

A response contains:

```text
status code
headers
optional body
```

---

## 4. HTTP Is an Application-Layer Protocol

HTTP operates at the application layer.

It relies on lower layers for transport:

```text
HTTP
 |
TCP / QUIC
 |
IP
 |
Ethernet / network
```

HTTP/3 uses QUIC rather than TCP.

---

## 5. HTTP and URLs

Example:

```text
https://api.example.com:443/orders?id=100
```

Components:

```text
scheme     = https
host       = api.example.com
port       = 443
path       = /orders
query      = id=100
```

---

## 6. HTTP Scheme

Common schemes:

```text
http
https
```

HTTP normally uses:

```text
TCP 80
```

HTTPS commonly uses:

```text
TCP 443
```

HTTP/3 uses QUIC, commonly over:

```text
UDP 443
```

---

## 7. HTTP Methods

Common methods:

```text
GET
POST
PUT
PATCH
DELETE
HEAD
OPTIONS
CONNECT
TRACE
```

DevOps engineers frequently troubleshoot:

```text
GET
POST
PUT
PATCH
DELETE
```

---

## 8. GET

GET retrieves a representation of a resource.

Example:

```http
GET /products
```

A successful response commonly uses:

```text
200 OK
```

GET requests should generally be safe and idempotent by HTTP semantics.

---

## 9. POST

POST submits data for processing or resource creation.

Example:

```http
POST /orders
```

Possible response:

```text
201 Created
```

POST is generally not idempotent.

---

## 10. PUT

PUT generally replaces or creates the target resource representation.

Example:

```http
PUT /users/123
```

PUT is defined as idempotent.

---

## 11. PATCH

PATCH applies partial modifications.

Example:

```http
PATCH /users/123
```

Idempotency depends on the operation and implementation.

---

## 12. DELETE

DELETE requests removal of a resource.

Example:

```http
DELETE /orders/123
```

DELETE is defined as idempotent, although repeated requests can produce different status codes.

---

## 13. HEAD

HEAD requests headers without the response body.

Useful for:

```text
checking resource metadata
testing availability
checking Content-Length
```

Example:

```bash
curl -I https://example.com
```

---

## 14. OPTIONS

OPTIONS can discover communication options for a target resource.

It is important for:

```text
CORS preflight
```

Example:

```http
OPTIONS /api/orders
```

---

## 15. CONNECT

CONNECT establishes a tunnel through a proxy, commonly used for HTTPS proxy tunneling.

Conceptually:

```text
Client
 |
CONNECT proxy
 |
TLS tunnel
 |
Target
```

---

## 16. TRACE

TRACE can reflect a request for diagnostic purposes.

It is often disabled in production because of security considerations.

---

## 17. Safe HTTP Methods

Safe methods are intended not to change server state as a result of the request semantics.

Examples:

```text
GET
HEAD
OPTIONS
```

This does not mean the server implementation can never perform side effects.

---

## 18. Idempotent Methods

Repeated execution has the same intended effect as one execution.

Common idempotent methods:

```text
GET
HEAD
PUT
DELETE
OPTIONS
TRACE
```

POST is not generally idempotent.

---

## 19. HTTP Request Structure

Example:

```http
GET /api/products HTTP/1.1
Host: api.example.com
User-Agent: curl
Accept: application/json
Authorization: Bearer <token>

```

Components:

```text
request line
headers
blank line
optional body
```

---

## 20. HTTP Response Structure

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 123
Cache-Control: no-cache

{"status":"ok"}
```

Components:

```text
status line
headers
blank line
body
```

---

## 21. HTTP Status Code Classes

Status codes have five classes:

```text
1xx informational
2xx success
3xx redirection
4xx client error
5xx server error
```

---

## 22. 1xx Responses

Examples:

```text
100 Continue
101 Switching Protocols
103 Early Hints
```

They provide intermediate information rather than the final application result.

---

## 23. 200 OK

Means the request succeeded.

Example:

```text
GET /health
→ 200 OK
```

---

## 24. 201 Created

Indicates successful resource creation.

Common with:

```text
POST
```

Example:

```text
POST /users
→ 201 Created
```

---

## 25. 202 Accepted

The request has been accepted for processing but may not have completed yet.

Useful for asynchronous workflows.

---

## 26. 204 No Content

The request succeeded but there is no response body.

Common with:

```text
DELETE
```

or successful update operations.

---

## 27. 301 Moved Permanently

Indicates permanent redirection.

Example:

```text
http://example.com
→ https://example.com
```

---

## 28. 302 Found

Indicates temporary redirection.

Clients may follow the `Location` header.

---

## 29. 303 See Other

Redirects the client to another resource, commonly using GET for the resulting retrieval.

---

## 30. 304 Not Modified

Used with conditional requests and caching.

It indicates that the cached representation can still be used.

---

## 31. 307 Temporary Redirect

Preserves the original HTTP method when redirecting.

This differs from historical 302 behavior in many client implementations.

---

## 32. 308 Permanent Redirect

Permanent redirect that preserves the request method.

---

## 33. 400 Bad Request

The server cannot process the request because the request is invalid.

Examples:

```text
malformed JSON
invalid syntax
invalid request parameters
```

---

## 34. 401 Unauthorized

Indicates that authentication is required or failed.

It does not strictly mean:

```text
user has no permission
```

That is more commonly represented by 403.

---

## 35. 403 Forbidden

The server understood the request but refuses to authorize it.

Common causes:

```text
missing permission
IAM/application authorization
WAF policy
ALB rule
Nginx access control
```

---

## 36. 404 Not Found

The requested resource was not found.

In Kubernetes environments, 404 can originate from:

```text
ALB
Ingress
Nginx
application
```

Determine which layer generated it.

---

## 37. 405 Method Not Allowed

The endpoint exists but does not allow the requested method.

Example:

```text
GET /orders
→ 200

POST /orders
→ 405
```

---

## 38. 408 Request Timeout

The server timed out waiting for the request.

---

## 39. 409 Conflict

The request conflicts with the current resource state.

Common examples:

```text
duplicate resource
version conflict
state conflict
```

---

## 40. 410 Gone

The resource is intentionally no longer available.

---

## 41. 413 Content Too Large

The request body exceeds an allowed limit.

This can originate from:

```text
ALB
Nginx
Ingress
application
WAF
```

---

## 42. 414 URI Too Long

The requested URI is too long for the receiving system's configured limits.

---

## 43. 415 Unsupported Media Type

The server does not support the request body format.

Example:

```text
Content-Type: application/xml
```

when only JSON is accepted.

---

## 44. 429 Too Many Requests

Indicates rate limiting.

Common sources:

```text
API gateway
WAF
ALB/application layer
Nginx
application
```

---

## 45. 500 Internal Server Error

Generic server-side application failure.

It does not identify the exact cause.

Check:

```text
application logs
stack traces
dependency failures
```

---

## 46. 501 Not Implemented

The server does not support the functionality required to fulfill the request.

---

## 47. 502 Bad Gateway

A proxy/gateway received an invalid response from an upstream server.

Common in:

```text
ALB
Nginx
reverse proxy
Ingress
```

Potential causes:

```text
backend unavailable
connection reset
invalid upstream response
protocol mismatch
```

---

## 48. 503 Service Unavailable

The server cannot currently handle the request.

Common causes:

```text
no healthy targets
application overloaded
deployment
backend unavailable
Kubernetes Service has no endpoints
```

---

## 49. 504 Gateway Timeout

A gateway/proxy did not receive a timely response from the upstream.

Common causes:

```text
slow backend
network timeout
security group
application deadlock
dependency latency
```

---

## 50. 502 vs 503 vs 504

```text
502
→ invalid/unexpected upstream response

503
→ service unavailable/no healthy capacity

504
→ upstream response timed out
```

The exact behavior depends on the component generating the response.

---

## 51. HTTP Headers

Headers carry metadata.

Examples:

```text
Host
Content-Type
Content-Length
Authorization
Cookie
Set-Cookie
Accept
User-Agent
Cache-Control
Location
Connection
X-Forwarded-For
X-Forwarded-Proto
```

---

## 52. Host Header

Example:

```http
Host: api.example.com
```

It identifies the requested host.

It is essential for:

```text
virtual hosting
ALB routing
Nginx server blocks
```

---

## 53. Content-Type

Identifies the media type of the request/response body.

Example:

```http
Content-Type: application/json
```

---

## 54. Accept

Indicates formats the client can accept.

Example:

```http
Accept: application/json
```

---

## 55. Content-Length

Specifies body length in bytes for applicable messages.

Example:

```http
Content-Length: 1024
```

---

## 56. Transfer-Encoding

HTTP/1.1 can use:

```http
Transfer-Encoding: chunked
```

for streaming a body when its total length is not known in advance.

---

## 57. Chunked Transfer Encoding

A response can be transmitted in chunks:

```text
chunk
chunk
chunk
0
```

This allows a server to send data progressively.

---

## 58. Connection Header

HTTP/1.1 normally uses persistent connections by default.

The `Connection` header can control connection-specific behavior.

HTTP/2 and HTTP/3 handle connection management differently and generally do not use HTTP/1.1 hop-by-hop connection semantics in the same way.

---

## 59. Keep-Alive

Persistent connections allow multiple requests/responses to use the same connection.

Benefits:

```text
fewer TCP handshakes
lower latency
less CPU overhead
```

---

## 60. Connection Reuse

Instead of:

```text
request
TCP connect
request
close

request
TCP connect
request
close
```

connection reuse can provide:

```text
TCP connect
request
request
request
...
```

---

## 61. HTTP/1.0

HTTP/1.0 commonly used non-persistent connections by default.

HTTP/1.1 introduced persistent connections and many improvements.

---

## 62. HTTP/1.1

HTTP/1.1 is widely deployed.

Important characteristics:

```text
persistent connections
Host header
chunked transfer
caching
range requests
```

---

## 63. HTTP/1.1 Head-of-Line Behavior

HTTP/1.1 can suffer from application-level head-of-line limitations when multiple requests share connection sequencing depending on the usage pattern.

HTTP/2 improves multiplexing.

---

## 64. HTTP/2

HTTP/2 introduces:

```text
binary framing
multiplexing
stream IDs
header compression
stream prioritization mechanisms
```

Multiple streams can share one TCP connection.

---

## 65. HTTP/2 Multiplexing

Conceptually:

```text
One TCP connection
 |
 +-- Stream 1
 +-- Stream 2
 +-- Stream 3
 +-- Stream 4
```

This reduces the need for multiple parallel TCP connections.

---

## 66. HTTP/2 Binary Framing

HTTP/2 represents communication using binary frames rather than HTTP/1.1's textual message framing.

Common frame types include:

```text
HEADERS
DATA
SETTINGS
WINDOW_UPDATE
RST_STREAM
PING
GOAWAY
```

---

## 67. HTTP/2 Header Compression

HTTP/2 uses:

```text
HPACK
```

to reduce repeated header overhead.

---

## 68. HTTP/2 over TLS

HTTP/2 can technically be used without TLS in some specifications, but browser deployments overwhelmingly use HTTPS.

In production web systems:

```text
HTTP/2 + TLS
```

is the normal model.

---

## 69. HTTP/3

HTTP/3 uses:

```text
QUIC
```

instead of TCP.

QUIC runs over:

```text
UDP
```

---

## 70. HTTP/3 Architecture

```text
HTTP/3
  |
QUIC
  |
UDP
  |
IP
```

This changes connection establishment and stream behavior.

---

## 71. QUIC

QUIC provides transport features including:

```text
reliable delivery
streams
congestion control
TLS integration
connection migration
```

over UDP.

---

## 72. HTTP/3 and TLS

TLS 1.3 is integrated into QUIC.

Therefore HTTP/3 does not use the same separate:

```text
TCP handshake
then TLS handshake
```

sequence as traditional HTTPS over TCP.

---

## 73. HTTP Version Comparison

```text
HTTP/1.1
→ TCP
→ textual message format

HTTP/2
→ TCP
→ binary multiplexed frames

HTTP/3
→ QUIC
→ UDP
→ multiplexed streams
→ TLS 1.3 integrated
```

---

## 74. HTTP Caching

HTTP caching can reduce:

```text
latency
bandwidth
backend load
```

Headers include:

```text
Cache-Control
ETag
Last-Modified
Expires
```

---

## 75. Cache-Control

Example:

```http
Cache-Control: max-age=300
```

The response can be cached according to the specified directives.

---

## 76. ETag

ETag identifies a representation version.

Example:

```http
ETag: "abc123"
```

A client can later send:

```http
If-None-Match: "abc123"
```

---

## 77. If-None-Match

If the representation has not changed, the server can return:

```text
304 Not Modified
```

instead of the full response body.

---

## 78. Last-Modified

A server can provide:

```http
Last-Modified: ...
```

A client can use:

```http
If-Modified-Since: ...
```

for conditional requests.

---

## 79. Cookies

Cookies are sent through:

```http
Cookie:
```

and servers can set them using:

```http
Set-Cookie:
```

---

## 80. Secure Cookie

Example:

```http
Set-Cookie: session=abc; Secure
```

The `Secure` attribute restricts the cookie to secure transport.

---

## 81. HttpOnly Cookie

```http
Set-Cookie: session=abc; HttpOnly
```

prevents JavaScript from reading the cookie through standard browser APIs.

This helps reduce some XSS-related session theft risks.

---

## 82. SameSite

Common values:

```text
Strict
Lax
None
```

SameSite controls cross-site cookie sending behavior.

---

## 83. Authorization Header

Bearer authentication example:

```http
Authorization: Bearer <token>
```

Never place real production secrets in logs or examples.

---

## 84. HTTP Compression

Common compression:

```text
gzip
br
```

Compression reduces payload size but consumes CPU.

Use appropriate compression policies for:

```text
JSON
HTML
CSS
JavaScript
```

Avoid compressing already-compressed media unnecessarily.

---

## 85. Content-Encoding

Example:

```http
Content-Encoding: gzip
```

indicates that the body is compressed.

---

## 86. Reverse Proxy

A reverse proxy sits in front of backend servers.

```text
Client
 |
v
Reverse Proxy
 |
+-- Backend 1
+-- Backend 2
+-- Backend 3
```

Examples:

```text
Nginx
ALB
Envoy
HAProxy
```

---

## 87. Forward Proxy

A forward proxy represents clients when accessing external destinations.

```text
Client
 |
v
Forward Proxy
 |
v
Internet
```

Reverse proxy represents servers; forward proxy represents clients.

---

## 88. Nginx as Reverse Proxy

Typical flow:

```text
Internet
 |
Nginx
 |
Application
```

Nginx can provide:

```text
TLS termination
routing
load balancing
compression
caching
access control
```

---

## 89. AWS ALB as Reverse Proxy

Typical:

```text
Client
 |
HTTPS
 |
ALB
 |
HTTP/HTTPS
 |
Target
```

The ALB terminates or forwards connections depending on listener/target configuration.

---

## 90. TLS Termination

TLS termination means encrypted client traffic is decrypted at a component.

Example:

```text
Client
  |
HTTPS
  |
ALB
  |
HTTP
  |
Application
```

The application does not receive TLS directly in this architecture.

---

## 91. TLS Passthrough

In TLS passthrough:

```text
Client
 |
TLS
 |
Load Balancer
 |
TLS
 |
Application
```

The load balancer does not terminate the encrypted connection.

Support depends on the load-balancing technology and configuration.

---

## 92. TLS Re-encryption

A common secure architecture:

```text
Client
 |
HTTPS
 |
ALB
 |
HTTPS
 |
Backend
```

The ALB terminates client TLS and creates a separate TLS connection to the backend.

---

## 93. TLS

TLS means:

```text
Transport Layer Security
```

It provides:

```text
confidentiality
integrity
authentication
```

for secure communications.

---

## 94. HTTPS

HTTPS is:

```text
HTTP over TLS
```

Typical:

```text
HTTP
 |
TLS
 |
TCP
 |
IP
```

for HTTP/1.1 and HTTP/2.

---

## 95. TLS Provides Confidentiality

Encryption prevents passive observers from reading application data.

Without TLS:

```text
HTTP
→ readable traffic
```

With TLS:

```text
HTTPS
→ encrypted application data
```

---

## 96. TLS Provides Integrity

TLS detects unauthorized modification of protected traffic.

An attacker should not be able to silently change:

```text
HTTP response
```

without detection.

---

## 97. TLS Provides Authentication

Server certificates allow clients to authenticate the server identity through a trusted certificate chain.

---

## 98. TLS Certificate

A certificate binds an identity such as:

```text
api.example.com
```

to a public key.

It is signed by a certificate authority or chain of authorities.

---

## 99. Certificate Authority

A CA is a trusted entity that issues/signs certificates.

Examples include public CAs such as:

```text
Let's Encrypt
DigiCert
GlobalSign
```

Production trust depends on the client's trust store.

---

## 100. Root CA

A root CA certificate is generally trusted directly by the client operating system/browser trust store.

---

## 101. Intermediate CA

An intermediate CA is signed by a root or another trusted CA.

Typical chain:

```text
Root CA
   |
Intermediate CA
   |
Server Certificate
```

---

## 102. Certificate Chain

A server normally presents:

```text
leaf certificate
+
intermediate certificate(s)
```

The client builds a chain toward a trusted root.

---

## 103. Certificate Validation

A client checks things such as:

```text
signature chain
validity dates
hostname
key usage
extended key usage
trust anchor
revocation-related mechanisms
```

depending on the implementation and policy.

---

## 104. SAN

SAN means:

```text
Subject Alternative Name
```

Modern hostname validation relies primarily on SAN entries.

Example:

```text
DNS:api.example.com
DNS:www.example.com
```

---

## 105. Common Name

Historically the Common Name field was used for hostname identity.

Modern clients rely on SAN for hostname validation.

Do not depend on CN alone for modern certificates.

---

## 106. Wildcard Certificate

Example:

```text
*.example.com
```

can cover:

```text
api.example.com
www.example.com
```

but generally not:

```text
api.dev.example.com
```

because the wildcard covers one label at that level.

---

## 107. Certificate Expiration

Certificates have:

```text
Not Before
Not After
```

If expired:

```text
TLS validation fails
```

Automated certificate renewal is strongly preferred for production.

---

## 108. ACM

AWS Certificate Manager can provision and manage certificates for supported AWS services.

Common use:

```text
ACM certificate
 |
ALB HTTPS listener
```

---

## 109. ACM Certificate Validation

ACM commonly supports:

```text
DNS validation
```

which uses DNS records to prove domain control.

---

## 110. ACM and Route 53

A common AWS workflow:

```text
ACM
 |
DNS validation record
 |
Route 53
 |
Certificate issued
 |
ALB HTTPS listener
```

---

## 111. TLS Versions

Modern TLS versions include:

```text
TLS 1.2
TLS 1.3
```

Older versions such as:

```text
TLS 1.0
TLS 1.1
```

are obsolete and should generally be disabled.

---

## 112. TLS 1.2

TLS 1.2 remains widely supported and is common in enterprise environments.

Use strong cipher suites and disable obsolete algorithms.

---

## 113. TLS 1.3

TLS 1.3 simplifies and strengthens cryptographic negotiation.

It removes many legacy cryptographic options and generally reduces handshake round trips.

---

## 114. Cipher Suite

A cipher suite identifies cryptographic algorithms used by TLS.

TLS 1.2 examples can include:

```text
ECDHE
AES-GCM
ChaCha20-Poly1305
```

TLS 1.3 has a smaller set of supported cipher suites.

---

## 115. Forward Secrecy

Ephemeral key exchange such as:

```text
ECDHE
```

provides forward secrecy.

Compromise of a long-term private key should not allow decryption of previously captured sessions when ephemeral keys were used appropriately.

---

## 116. TLS Handshake

Simplified TLS 1.2 flow:

```text
ClientHello
     |
ServerHello
Certificate
ServerKeyExchange
...
     |
ClientKeyExchange
ChangeCipherSpec
Finished
     |
Encrypted application data
```

Exact messages depend on the negotiated parameters.

---

## 117. TLS 1.3 Handshake

Simplified:

```text
ClientHello + key share
        |
        v
ServerHello + key share
EncryptedExtensions
Certificate
CertificateVerify
Finished
        |
        v
Encrypted application data
```

The exact exchange can vary.

---

## 118. TLS 1.3 1-RTT

TLS 1.3 can establish secure application data with fewer round trips than typical TLS 1.2 handshakes.

This reduces connection latency.

---

## 119. Session Resumption

TLS supports mechanisms for resuming sessions.

Benefits:

```text
lower latency
less CPU
faster repeated connections
```

TLS 1.3 commonly uses session tickets.

---

## 120. SNI

SNI means:

```text
Server Name Indication
```

The client includes the intended hostname during TLS negotiation.

Example:

```text
api.example.com
```

This allows a load balancer or web server to select the correct certificate for multiple domains sharing an endpoint.

---

## 121. ALPN

ALPN means:

```text
Application-Layer Protocol Negotiation
```

It allows client and server to negotiate protocols such as:

```text
h2
http/1.1
```

HTTP/2 commonly uses:

```text
h2
```

---

## 122. TLS and HTTP/2

Typical browser flow:

```text
TCP
 |
TLS
 |
ALPN
 |
h2
 |
HTTP/2
```

---

## 123. TLS and HTTP/3

Typical:

```text
UDP
 |
QUIC
 |
TLS 1.3
 |
HTTP/3
```

---

## 124. mTLS

mTLS means:

```text
mutual TLS
```

Both sides authenticate using certificates.

```text
Client certificate
        ↕
Server certificate
```

---

## 125. mTLS Use Cases

Common:

```text
service-to-service security
zero-trust architectures
financial systems
internal APIs
service mesh
```

---

## 126. TLS Termination in EKS

Common architecture:

```text
Internet
 |
HTTPS
 |
AWS ALB
 |
HTTP
 |
Kubernetes Service
 |
Pod
```

The ALB holds the ACM certificate.

---

## 127. TLS Re-encryption in EKS

More secure backend architecture:

```text
Internet
 |
HTTPS
 |
ALB
 |
HTTPS
 |
Service
 |
Pod
```

Useful when encryption is required inside the cluster/network boundary.

---

## 128. ALB Listener

An ALB HTTPS listener commonly uses:

```text
port 443
certificate
TLS policy
routing rules
```

---

## 129. ALB Redirect HTTP to HTTPS

A common configuration:

```text
port 80
   |
redirect
   v
port 443
```

This ensures users are directed to HTTPS.

---

## 130. ALB TLS Policy

A TLS security policy controls supported protocol versions and cryptographic algorithms.

Production should use a modern AWS-managed policy appropriate to compatibility requirements.

---

## 131. Nginx TLS Termination

Example architecture:

```text
Client
 |
HTTPS
 |
Nginx
 |
HTTP
 |
Application
```

Nginx configuration commonly includes:

```text
listen 443 ssl;
ssl_certificate
ssl_certificate_key
```

---

## 132. Nginx Reverse Proxy Headers

Common headers:

```nginx
proxy_set_header Host $host;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header X-Forwarded-Proto $scheme;
```

These preserve important request context.

---

## 133. X-Forwarded-For

This header commonly carries the original client IP through proxies.

Example:

```text
X-Forwarded-For: 198.51.100.10
```

Multiple proxies can append addresses.

---

## 134. X-Forwarded-Proto

Example:

```text
X-Forwarded-Proto: https
```

This tells the backend what protocol the original client used.

It is important when TLS terminates at a proxy.

---

## 135. Proxy Trust

Applications should not blindly trust arbitrary client-supplied:

```text
X-Forwarded-For
X-Forwarded-Proto
```

Only trust these headers from known proxy/load-balancer paths.

---

## 136. HTTP Host-Based Routing

Example:

```text
api.example.com → API
shop.example.com → frontend
admin.example.com → admin
```

ALB and Nginx can use the Host header for routing.

---

## 137. Path-Based Routing

Example:

```text
/api/* → API
/static/* → frontend/static
/orders/* → order service
```

This is commonly implemented at ALB or reverse-proxy layers.

---

## 138. ALB + Kubernetes Ingress

Typical:

```text
Route 53
 |
ALB
 |
Ingress rules
 |
Kubernetes Service
 |
Pods
```

Ingress rules can define:

```text
host
path
backend
```

---

## 139. HTTP Health Checks

Load balancers commonly check an endpoint such as:

```text
/health
```

A successful application health check should indicate the correct readiness state for receiving traffic.

---

## 140. Liveness vs Readiness

Kubernetes:

```text
liveness
→ should the container be restarted?

readiness
→ should the Pod receive traffic?
```

A load balancer should generally route only to ready backends.

---

## 141. HTTP Readiness Failure

If readiness fails:

```text
Pod
 |
not ready
 |
Service endpoints
 |
traffic removed
```

This can produce:

```text
503
```

at the upstream load balancer if no healthy targets remain.

---

## 142. HTTP 503 During Deployment

A rolling deployment can produce 503 if:

```text
old Pods terminate too early
new Pods are not ready
readiness probes are incorrect
```

Production controls include:

```text
readinessProbe
RollingUpdate
maxUnavailable
maxSurge
PodDisruptionBudget
```

---

## 143. HTTP Connection Reset

A connection reset can occur when:

```text
application closes connection
proxy resets
load balancer terminates
network device rejects
Pod disappears
```

Use packet capture and logs to identify which side sent RST.

---

## 144. HTTP Timeout

Timeouts can occur at multiple layers:

```text
DNS timeout
TCP connect timeout
TLS handshake timeout
proxy timeout
application processing timeout
client timeout
```

Always identify which timeout occurred.

---

## 145. `curl` Basic Test

```bash
curl -v https://example.com
```

This shows:

```text
DNS resolution
TCP connection
TLS handshake
HTTP request
HTTP response
```

---

## 146. `curl` Headers Only

```bash
curl -I https://example.com
```

Useful for:

```text
status
headers
redirect
server
cache
```

---

## 147. `curl` Follow Redirects

```bash
curl -L -v http://example.com
```

Useful for testing:

```text
HTTP → HTTPS
```

redirects.

---

## 148. `curl` Force HTTP/1.1

```bash
curl --http1.1 -v https://example.com
```

Useful for comparing protocol behavior.

---

## 149. `curl` Force HTTP/2

```bash
curl --http2 -v https://example.com
```

Support depends on the installed curl build and TLS library.

---

## 150. `curl` Resolve Override

```bash
curl --resolve api.example.com:443:203.0.113.10 \
  https://api.example.com
```

This is extremely useful for testing a specific endpoint while preserving:

```text
Host
TLS SNI
```

for the requested hostname.

---

## 151. `curl` Insecure TLS

```bash
curl -k https://example.com
```

This disables certificate verification.

Use only for controlled troubleshooting.

Never use `-k` as a production security fix.

---

## 152. `curl` Certificate Information

```bash
curl -vI https://example.com
```

Can reveal certificate and TLS negotiation details depending on the build.

---

## 153. OpenSSL TLS Test

```bash
openssl s_client -connect example.com:443 \
  -servername example.com
```

This is a powerful TLS troubleshooting command.

---

## 154. OpenSSL Show Certificates

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  -showcerts
```

Inspect:

```text
leaf
intermediate
certificate chain
```

---

## 155. OpenSSL TLS 1.2 Test

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  -tls1_2
```

Useful for protocol compatibility testing.

---

## 156. OpenSSL TLS 1.3 Test

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  -tls1_3
```

---

## 157. Check Certificate Dates

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com </dev/null 2>/dev/null |
openssl x509 -noout -dates
```

---

## 158. Check Certificate Subject and SAN

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com </dev/null 2>/dev/null |
openssl x509 -noout -subject -issuer -ext subjectAltName
```

---

## 159. Certificate Expiration Monitoring

Production monitoring should alert before:

```text
Not After
```

is reached.

Do not wait for expiration.

---

## 160. Certificate Chain Failure

Symptoms:

```text
unable to get local issuer certificate
certificate verify failed
unknown CA
```

Common cause:

```text
missing intermediate certificate
```

The server should normally present the required chain.

---

## 161. Hostname Mismatch

Example:

```text
Requested:
api.example.com

Certificate:
admin.example.com
```

TLS verification fails.

Check SAN:

```bash
openssl s_client ...
```

---

## 162. Expired Certificate

Symptoms:

```text
certificate has expired
certificate verify failed
```

Check:

```bash
openssl x509 -noout -dates
```

---

## 163. Not Yet Valid Certificate

If system time is incorrect or certificate validity has not started:

```text
certificate is not yet valid
```

Check:

```bash
date
timedatectl
```

---

## 164. Clock Skew

TLS certificate validation depends on accurate time.

A large clock difference can cause:

```text
certificate expired
not yet valid
```

Always check NTP/time synchronization during TLS incidents.

---

## 165. SNI Failure

If a client does not send the expected SNI:

```text
wrong certificate
wrong virtual host
TLS failure
```

Test with:

```bash
openssl s_client -connect IP:443 -servername api.example.com
```

---

## 166. ALPN Failure

If HTTP/2 negotiation fails:

```text
ALPN
```

can be part of the investigation.

Compare:

```bash
curl --http1.1
curl --http2
```

---

## 167. TLS Handshake Failure

Possible causes:

```text
protocol mismatch
cipher mismatch
certificate problem
SNI
ALPN
client certificate requirement
clock
middlebox
```

Use:

```bash
openssl s_client
curl -v
tcpdump
```

---

## 168. mTLS Failure

Common causes:

```text
missing client certificate
untrusted client CA
expired client certificate
wrong certificate chain
EKU mismatch
```

---

## 169. TLS Private Key

The server private key must be protected.

Never:

```text
commit private key to Git
put it in a public image
print it in logs
```

Use:

```text
ACM
Secrets Manager
Kubernetes Secrets with encryption
external secret systems
```

as appropriate.

---

## 170. TLS Secret in Kubernetes

Typical structure:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64>
  tls.key: <base64>
```

Do not commit raw private keys to Git.

---

## 171. cert-manager

`cert-manager` can automate certificate issuance and renewal in Kubernetes.

Typical flow:

```text
Ingress
 |
cert-manager
 |
ACME/CA
 |
Kubernetes Secret
 |
Ingress Controller
```

---

## 172. cert-manager and Route 53

DNS-01 validation can use Route 53.

Conceptually:

```text
cert-manager
 |
create TXT validation record
 |
Route 53
 |
CA validates
 |
certificate issued
```

IAM permissions must be least privilege.

---

## 173. AWS Load Balancer Controller

In EKS, the AWS Load Balancer Controller can create/manage ALBs from Kubernetes Ingress resources.

Typical flow:

```text
Ingress
 |
AWS Load Balancer Controller
 |
ALB
 |
Target Group
 |
Service/Pod
```

---

## 174. ALB Target Types

AWS ALB integration commonly supports target modes such as:

```text
instance
ip
```

The selected mode changes how traffic reaches workloads.

---

## 175. IP Target Mode

With IP targets:

```text
ALB
 |
Pod IP
```

This can provide direct routing to Pods.

It is common in EKS architectures using the AWS VPC CNI.

---

## 176. Instance Target Mode

With instance targets:

```text
ALB
 |
Node
 |
NodePort
 |
Pod
```

The traffic passes through the node-level Service exposure.

---

## 177. HTTP Health Check Path

Example:

```text
/health
```

Health check configuration must match the actual application behavior.

Do not make health checks depend on unnecessary external services unless that is intentional.

---

## 178. HTTP Host Header in ALB

ALB rules can match:

```text
Host
```

Example:

```text
api.example.com
```

and forward to the API target group.

---

## 179. Path-Based ALB Routing

Example:

```text
/api/*
/cart/*
/catalog/*
```

Each can route to different target groups or Kubernetes backends depending on Ingress configuration.

---

## 180. ALB Access Logs

ALB access logs can help identify:

```text
client IP
request path
status
target status
latency
TLS details
```

Enable and analyze them according to production logging requirements.

---

## 181. ALB 4xx vs 5xx

Generally:

```text
4xx
→ client/request side

5xx
→ server/upstream side
```

But the exact source can be determined from ALB access logs and target response fields.

---

## 182. ALB 502 Troubleshooting

Check:

```text
target health
application listening port
security groups
protocol mismatch
connection reset
target response
```

Example:

```text
ALB HTTPS
→ HTTP backend
```

must match the configured target protocol.

---

## 183. ALB 503 Troubleshooting

Check:

```text
target group
healthy targets
Ingress
Service
Endpoints
Pod readiness
```

A Kubernetes Service with zero endpoints can lead to unavailable backend behavior.

---

## 184. ALB 504 Troubleshooting

Check:

```text
backend latency
network path
application processing
dependency calls
timeouts
```

Measure:

```text
ALB target response time
application latency
database latency
```

---

## 185. Nginx 502

Common causes:

```text
upstream refused connection
wrong upstream port
backend crashed
Unix socket issue
protocol mismatch
```

Inspect:

```bash
tail -f /var/log/nginx/error.log
```

---

## 186. Nginx 504

Usually indicates an upstream timeout.

Check:

```nginx
proxy_connect_timeout
proxy_read_timeout
proxy_send_timeout
```

Do not simply increase timeouts without fixing slow upstream behavior.

---

## 187. Nginx Upstream

Example:

```nginx
upstream api_backend {
    server 10.0.1.10:8080;
    server 10.0.1.11:8080;
}
```

Then:

```nginx
location / {
    proxy_pass http://api_backend;
}
```

---

## 188. Nginx TLS Configuration

Production example concept:

```nginx
server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate     /etc/nginx/tls/tls.crt;
    ssl_certificate_key /etc/nginx/tls/tls.key;

    location / {
        proxy_pass http://api_backend;
    }
}
```

Exact directives should match the deployed Nginx version and security policy.

---

## 189. Nginx HTTP to HTTPS Redirect

Example:

```nginx
server {
    listen 80;
    server_name api.example.com;

    return 301 https://$host$request_uri;
}
```

---

## 190. HTTP Security Headers

Common headers:

```text
Strict-Transport-Security
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
Permissions-Policy
```

Apply based on application requirements.

---

## 191. HSTS

HSTS means:

```text
HTTP Strict Transport Security
```

Example:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

It instructs compliant browsers to use HTTPS.

Enable carefully, especially with `includeSubDomains` and preload-related policies.

---

## 192. HSTS Risk

If HTTPS is not correctly configured for all covered subdomains, aggressive HSTS policies can make applications inaccessible.

Test before broad rollout.

---

## 193. CORS

CORS controls browser cross-origin access.

Common headers:

```text
Access-Control-Allow-Origin
Access-Control-Allow-Methods
Access-Control-Allow-Headers
Access-Control-Allow-Credentials
```

---

## 194. CORS Preflight

Browsers may send:

```http
OPTIONS
```

before certain cross-origin requests.

If the preflight fails:

```text
application request may never be sent
```

---

## 195. CORS Troubleshooting

Check:

```bash
curl -i -X OPTIONS \
  https://api.example.com/orders \
  -H 'Origin: https://shop.example.com' \
  -H 'Access-Control-Request-Method: POST'
```

Inspect:

```text
Access-Control-Allow-Origin
Allow-Methods
Allow-Headers
Allow-Credentials
```

---

## 196. HTTP Authentication

Common mechanisms:

```text
Basic
Bearer
session cookie
mTLS
OAuth 2.0
OIDC
```

Do not transmit credentials over plain HTTP.

---

## 197. Basic Authentication

Example:

```http
Authorization: Basic <encoded-credentials>
```

It must be protected by TLS.

Base64 is encoding, not encryption.

---

## 198. Bearer Tokens

Example:

```http
Authorization: Bearer <token>
```

Protect tokens against:

```text
logs
browser leakage
source control
command history
```

---

## 199. HTTP Redirect Security

Do not create redirect chains unnecessarily.

Avoid:

```text
HTTP
→ HTTP
→ HTTPS
→ HTTPS
```

Prefer a direct:

```text
HTTP
→ HTTPS
```

redirect.

---

## 200. HTTP Request Smuggling

Request smuggling can occur when front-end and back-end HTTP parsers disagree about message boundaries.

Potentially dangerous headers include conflicting:

```text
Content-Length
Transfer-Encoding
```

Keep proxy/load-balancer/server versions patched and use consistent parsing behavior.

---

## 201. HTTP Header Injection

Never directly place untrusted input into response headers without appropriate validation.

Potential risks include:

```text
response splitting
cache poisoning
security policy bypass
```

---

## 202. HTTP Host Header Attacks

Applications should validate expected hostnames where relevant.

Do not blindly trust:

```http
Host:
```

for security-sensitive URL generation.

---

## 203. HTTP Request Size Limits

Production systems should define sensible limits for:

```text
request body
headers
URI
file uploads
```

At every relevant layer:

```text
ALB
Nginx
Ingress
application
```

---

## 204. Upload Troubleshooting

If a 20 MB upload fails:

```text
ALB limit?
Nginx limit?
Ingress annotation?
application limit?
WAF limit?
```

Find the layer generating the error.

---

## 205. HTTP Streaming

Streaming can be used for:

```text
large responses
events
logs
AI responses
file downloads
```

Check proxy buffering and timeout settings.

---

## 206. WebSocket

WebSocket upgrades an HTTP connection into a persistent bidirectional channel.

Typical handshake:

```http
GET /socket HTTP/1.1
Upgrade: websocket
Connection: Upgrade
```

The proxy/load balancer must support the upgrade.

---

## 207. WebSocket Troubleshooting

Check:

```text
Upgrade headers
Connection header
ALB/Nginx support
idle timeout
backend support
TLS
```

---

## 208. ALB Idle Timeout

AWS ALB has configurable idle timeout behavior.

Long-lived connections such as:

```text
WebSocket
streaming
long polling
```

may require appropriate timeout configuration.

Use current AWS service limits/documentation when setting production values.

---

## 209. HTTP Keepalive vs ALB Idle Timeout

These are different concepts.

```text
keepalive
→ connection reuse

idle timeout
→ how long an idle connection can remain before termination
```

---

## 210. HTTP Client Timeout

Applications should configure explicit:

```text
connect timeout
read timeout
write timeout
overall request timeout
```

Never allow requests to hang indefinitely.

---

## 211. Timeout Budget

For:

```text
frontend
 → API
 → service
 → database
```

timeouts should be designed as a hierarchy.

Example:

```text
frontend timeout > API timeout > downstream timeout
```

Avoid downstream operations outliving their caller indefinitely.

---

## 212. Retry Storm

Bad retry behavior:

```text
service A
 |
100 requests
 |
service B fails
 |
A retries 3 times
 |
300 requests
```

At scale this can amplify outages.

Use:

```text
bounded retries
backoff
jitter
circuit breakers
timeouts
```

---

## 213. HTTP Retry

Safe/idempotent operations are easier to retry.

For POST:

```text
idempotency key
```

may be used when the application supports it.

---

## 214. Idempotency Key

Example:

```http
Idempotency-Key: 8f4...
```

The server can use the key to prevent duplicate processing.

This is important for payment/order APIs.

---

## 215. HTTP Rate Limiting

Rate limiting protects:

```text
application
database
downstream services
```

Common response:

```text
429
```

Include appropriate retry guidance such as:

```http
Retry-After
```

when applicable.

---

## 216. HTTP Observability

Track:

```text
request rate
error rate
latency
status codes
request size
response size
TLS errors
upstream errors
```

A common RED model is:

```text
Rate
Errors
Duration
```

---

## 217. HTTP Metrics

Useful metrics:

```text
requests/sec
2xx rate
4xx rate
5xx rate
p50
p95
p99 latency
active connections
TLS handshake failures
upstream latency
```

---

## 218. HTTP Logs

Structured logs should include:

```text
timestamp
request ID
trace ID
method
host
path
status
duration
client IP
upstream
user/service identity
```

Do not log secrets.

---

## 219. Request ID

Example:

```http
X-Request-ID: abc123
```

A request ID helps correlate:

```text
ALB
Nginx
application
database
logs
```

---

## 220. Distributed Tracing

Trace context can flow through HTTP headers.

Modern tracing commonly uses:

```text
traceparent
```

OpenTelemetry can instrument HTTP clients and servers.

---

## 221. HTTP and OpenTelemetry

A request can be traced:

```text
ALB
 |
API
 |
Cart
 |
Redis
```

Each service contributes spans.

This helps distinguish:

```text
DNS latency
network latency
application latency
dependency latency
```

---

## 222. HTTP Production Architecture

Typical AWS/EKS:

```text
Internet
   |
Route 53
   |
ALB :443
   |
TLS termination
   |
Kubernetes Ingress
   |
Service
   |
Pods
   |
Downstream Services
```

---

## 223. HTTPS Production Architecture

```text
Client
 |
DNS
 |
TCP 443
 |
TLS
 |
ALB
 |
HTTP/HTTPS
 |
EKS
```

The TLS termination point should be explicitly documented.

---

## 224. TLS Trust Boundary

Example:

```text
Internet
 |
TLS encrypted
 |
ALB
 |
Trust boundary
 |
Private EKS network
```

If backend traffic is HTTP, encryption stops at ALB.

If backend traffic is HTTPS, encryption continues to the backend.

---

## 225. TLS Re-encryption Architecture

```text
Client
 |
TLS
 |
ALB
 |
TLS
 |
Ingress/Service
 |
Pod
```

This can support stronger encryption-in-transit requirements.

---

## 226. mTLS Service-to-Service

```text
Service A
 |
client certificate
 |
TLS
 |
Service B
 |
server certificate
```

Both identities are authenticated.

---

## 227. TLS Certificate Rotation

Production rotation should be:

```text
automated
overlap-aware
monitored
rollback-capable
```

Avoid manual emergency certificate replacement whenever possible.

---

## 228. Kubernetes TLS With Ingress

Conceptually:

```yaml
spec:
  tls:
    - hosts:
        - api.example.com
      secretName: api-tls
```

The exact TLS termination behavior depends on the Ingress controller.

---

## 229. ALB Ingress TLS

AWS Load Balancer Controller can associate ACM certificates through Ingress configuration/annotations.

Conceptually:

```yaml
alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:...
```

Use the exact annotation supported by the deployed controller version.

---

## 230. Production Kubernetes Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: roboshop
  namespace: roboshop
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
    alb.ingress.kubernetes.io/ssl-redirect: '443'
spec:
  ingressClassName: alb
  rules:
    - host: shop.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 80
```

Add certificate configuration and health-check settings according to the actual AWS environment.

---

## 231. Kubernetes Service Example

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend
  namespace: roboshop
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 8080
  type: ClusterIP
```

The Service provides stable discovery for the Pods.

---

## 232. Kubernetes Deployment Example

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: roboshop
spec:
  replicas: 3
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/roboshop/frontend:1.0.0
          ports:
            - containerPort: 8080
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
```

Add production resource requests/limits, security context and rollout controls appropriate to the workload.

---

## 233. HTTPS Redirect

Desired:

```text
http://shop.example.com
          |
          v
301/redirect
          |
          v
https://shop.example.com
```

Avoid application-specific redirect loops when ALB already performs the redirect.

---

## 234. Redirect Loop

Common cause:

```text
Client → HTTPS
ALB → HTTP backend
Backend thinks request is HTTP
Backend redirects HTTPS
Client → HTTPS
...
```

Use:

```text
X-Forwarded-Proto
```

correctly.

---

## 235. Redirect Loop Troubleshooting

Check:

```bash
curl -IL https://example.com
```

Look for:

```text
Location
```

repeating between the same URLs.

Then inspect:

```text
ALB redirect rule
application proxy awareness
X-Forwarded-Proto
```

---

## 236. 502 Production Scenario

```text
Client
 |
ALB
 |
target
 X
connection reset
```

Investigate:

```text
target health
application logs
Pod restarts
security groups
target port
protocol
```

---

## 237. 503 Production Scenario

```text
ALB
 |
Target Group
 |
No healthy targets
```

Investigate:

```text
Pod readiness
Service endpoints
Ingress
target registration
health check path
port
```

---

## 238. 504 Production Scenario

```text
ALB
 |
Target
 |
slow application
```

Investigate:

```text
application latency
database
downstream service
timeouts
CPU
memory
network
```

---

## 239. TLS Production Scenario

Symptom:

```text
curl: certificate verify failed
```

Check:

```bash
openssl s_client -connect api.example.com:443 \
  -servername api.example.com
```

Then verify:

```text
certificate chain
SAN
expiration
trust
```

---

## 240. HTTP Production Scenario

Symptom:

```text
HTTP 404
```

Determine source:

```text
ALB?
Nginx?
Ingress?
application?
```

Use:

```bash
curl -v
```

and inspect logs at each layer.

---

## 241. HTTP Production Scenario

Symptom:

```text
HTTP 429
```

Check:

```text
rate limiter
WAF
ALB/app layer
application logs
Retry-After
```

Do not blindly increase capacity without identifying the rate-limiting layer.

---

## 242. HTTP Production Scenario

Symptom:

```text
HTTP 500
```

Check:

```text
application logs
trace ID
dependency failures
recent deployment
configuration
```

---

## 243. HTTP Production Scenario

Symptom:

```text
HTTP 502
```

Check:

```text
upstream availability
target port
protocol
connection resets
```

---

## 244. HTTP Production Scenario

Symptom:

```text
HTTP 504
```

Check:

```text
upstream latency
network path
timeouts
database
downstream services
```

---

## 245. HTTP Troubleshooting Layer Model

```text
DNS
 |
IP
 |
TCP
 |
TLS
 |
HTTP
 |
Proxy
 |
Load Balancer
 |
Ingress
 |
Service
 |
Pod
 |
Application
```

Always isolate the failing layer.

---

## 246. HTTP Troubleshooting Commands

```bash
curl -v https://example.com
curl -I https://example.com
curl -IL http://example.com
curl --http1.1 -v https://example.com
curl --http2 -v https://example.com
curl --resolve example.com:443:203.0.113.10 https://example.com
openssl s_client -connect example.com:443 -servername example.com
dig example.com
ss -ntp
tcpdump -ni any port 443
```

---

## 247. TCP vs TLS vs HTTP Timing

Use:

```bash
curl -s -o /dev/null \
-w 'dns=%{time_namelookup}\nconnect=%{time_connect}\nstarttransfer=%{time_starttransfer}\ntotal=%{time_total}\n' \
https://example.com
```

This helps identify where time is spent.

---

## 248. HTTP Packet Capture

Capture:

```bash
sudo tcpdump -ni any host <server-ip> and port 443
```

For HTTP:

```bash
sudo tcpdump -ni any host <server-ip> and port 80
```

HTTPS payload is encrypted, but packet timing, IPs, ports and TCP behavior remain visible.

---

## 249. TLS Packet Capture

For TLS:

```bash
sudo tcpdump -ni any port 443
```

You can observe:

```text
TCP handshake
TLS ClientHello
TLS ServerHello
packet sizes
retransmissions
connection termination
```

without decrypting application payload.

---

## 250. TLS Handshake Failure Decision Tree

```text
ClientHello sent?
 |
No → TCP/network
 |
Yes
 |
ServerHello received?
 |
No → TLS/proxy/server/network
 |
Yes
 |
Certificate received?
 |
No → server/TLS configuration
 |
Yes
 |
Certificate valid?
 |
No → chain/SAN/expiry/time
 |
Yes
 |
Handshake complete?
 |
No → protocol/cipher/SNI/ALPN/mTLS
 |
Yes
 |
HTTP response?
 |
No → application/proxy
```

---

## 251. HTTP Security Checklist

```text
[ ] HTTPS enabled
[ ] HTTP redirected appropriately
[ ] TLS 1.2/1.3 policy
[ ] Strong certificates
[ ] Automated renewal
[ ] Private keys protected
[ ] HSTS considered
[ ] Secure cookies
[ ] HttpOnly cookies
[ ] SameSite configured
[ ] CORS restricted
[ ] Request limits
[ ] Rate limiting
[ ] Security headers
[ ] Proxy headers trusted correctly
[ ] Logs do not contain secrets
```

---

## 252. TLS Security Checklist

```text
[ ] TLS 1.0 disabled
[ ] TLS 1.1 disabled
[ ] Modern TLS policy
[ ] Valid certificate
[ ] SAN correct
[ ] Full certificate chain
[ ] Certificate renewal automated
[ ] Private key protected
[ ] SNI verified
[ ] ALPN verified
[ ] mTLS where required
[ ] Clock synchronization
[ ] Monitoring
```

---

## 253. EKS HTTP/HTTPS Checklist

```text
[ ] Route 53 record
[ ] ALB exists
[ ] ACM certificate
[ ] HTTPS listener
[ ] HTTP redirect
[ ] IngressClass
[ ] Ingress rules
[ ] Service
[ ] Endpoints
[ ] Pod readiness
[ ] Target health
[ ] Security groups
[ ] NetworkPolicy
[ ] Application port
[ ] Health check path
[ ] Timeout configuration
```

---

## 254. RoboShop HTTPS Flow

```text
Developer
   |
Git
   |
CI/CD
   |
Container Image
   |
ECR
   |
GitOps
   |
Argo CD
   |
EKS
   |
AWS Load Balancer Controller
   |
ALB :443
   |
ACM Certificate
   |
RoboShop frontend
```

---

## 255. RoboShop Internal HTTP Flow

Example:

```text
frontend
 |
HTTP
 |
cart.roboshop.svc.cluster.local
 |
CoreDNS
 |
cart Service
 |
cart Pod
```

---

## 256. RoboShop External HTTPS Flow

```text
User
 |
DNS
 |
Route 53
 |
ALB
 |
TLS termination
 |
Ingress
 |
frontend Service
 |
frontend Pod
```

---

## 257. RoboShop Troubleshooting Example

Symptom:

```text
shop.example.com returns 503
```

Check:

```bash
dig shop.example.com
```

Then:

```bash
kubectl get ingress -n roboshop
kubectl get svc -n roboshop
kubectl get endpoints -n roboshop
kubectl get pods -n roboshop
```

Then inspect ALB target health.

---

## 258. RoboShop TLS Troubleshooting

```bash
openssl s_client \
  -connect shop.example.com:443 \
  -servername shop.example.com
```

Check:

```text
certificate
SAN
issuer
expiry
TLS version
ALPN
```

---

## 259. RoboShop Backend Troubleshooting

If ALB is healthy but frontend cannot call cart:

```bash
kubectl exec -it <frontend-pod> -- nslookup cart
kubectl exec -it <frontend-pod> -- curl -v http://cart
```

This isolates:

```text
DNS
HTTP
Service
Pod
```

---

## 260. Production Architecture Summary

```text
                         Internet
                            |
                         Route 53
                            |
                         HTTPS :443
                            |
                      +-----v-----+
                      |    ALB    |
                      +-----+-----+
                            |
                     TLS termination
                            |
                     Kubernetes Ingress
                            |
                      Kubernetes Service
                            |
                          Pods
                         /    \
                        /      \
                 Service DNS   External APIs
                      |             |
                   CoreDNS       VPC DNS
                                     |
                                  Route 53
```

---

## 261. Interview: What Is HTTP?

HTTP is an application-layer request-response protocol used to transfer representations between clients and servers.

---

## 262. Interview: HTTP vs HTTPS?

```text
HTTP
→ plain HTTP

HTTPS
→ HTTP protected by TLS
```

---

## 263. Interview: What Is HTTP/2?

A binary, multiplexed HTTP protocol that commonly runs over TLS/TCP and supports multiple streams on a connection.

---

## 264. Interview: What Is HTTP/3?

HTTP/3 runs over QUIC, which runs over UDP, and integrates TLS 1.3 into the QUIC transport.

---

## 265. Interview: HTTP/1.1 vs HTTP/2?

```text
HTTP/1.1
→ text-oriented message format
→ persistent connections
→ limited multiplexing model

HTTP/2
→ binary frames
→ multiplexed streams
→ HPACK
```

---

## 266. Interview: HTTP/2 vs HTTP/3?

```text
HTTP/2
→ TCP

HTTP/3
→ QUIC/UDP
```

HTTP/3 avoids TCP-level head-of-line blocking between independent QUIC streams.

---

## 267. Interview: What Is a 502?

A gateway/proxy received an invalid or unusable response from an upstream according to that component's semantics.

---

## 268. Interview: What Is a 503?

The service is currently unavailable, often because no healthy backend is available or the application cannot serve requests.

---

## 269. Interview: What Is a 504?

A gateway/proxy timed out waiting for an upstream response.

---

## 270. Interview: 401 vs 403?

```text
401
→ authentication required/failed

403
→ request understood but authorization denied
```

---

## 271. Interview: What Is TLS?

TLS provides encrypted communication, integrity protection and server authentication.

---

## 272. Interview: What Is a Certificate?

A certificate binds an identity to a public key and is signed through a certificate trust chain.

---

## 273. Interview: What Is a CA?

A Certificate Authority issues/signs certificates that clients can validate through a trusted chain.

---

## 274. Interview: What Is SNI?

SNI allows the client to indicate the intended hostname during TLS negotiation so a server can select the correct certificate/configuration.

---

## 275. Interview: What Is ALPN?

ALPN negotiates the application protocol over a TLS connection, such as:

```text
h2
http/1.1
```

---

## 276. Interview: What Is mTLS?

Mutual TLS authenticates both the server and client using certificates.

---

## 277. Interview: What Is TLS Termination?

The component at the termination point decrypts client TLS traffic.

Example:

```text
Client
 |
HTTPS
 |
ALB
 |
HTTP
 |
Backend
```

---

## 278. Interview: What Is TLS Passthrough?

The load balancer/proxy forwards the TLS connection without terminating it, allowing the backend to perform TLS termination.

---

## 279. Interview: What Is TLS Re-encryption?

TLS is terminated at one layer and a new TLS connection is established toward the backend.

---

## 280. Interview: How Do You Debug a Certificate Error?

Use:

```bash
openssl s_client -connect host:443 -servername host
```

Check:

```text
SAN
expiry
chain
issuer
trust
```

---

## 281. Interview: How Do You Check TLS Version?

```bash
openssl s_client -connect host:443 -servername host -tls1_2
openssl s_client -connect host:443 -servername host -tls1_3
```

---

## 282. Interview: What Causes TLS Handshake Failure?

Common causes:

```text
unsupported TLS version
cipher mismatch
certificate problem
SNI
ALPN
mTLS
clock skew
network middlebox
```

---

## 283. Interview: Why Is SNI Important With ALB?

Multiple HTTPS hostnames can share an ALB/listener, and SNI allows the ALB to select the appropriate certificate.

---

## 284. Interview: Why Can a Browser Work While `curl` Fails?

Possible differences:

```text
TLS versions
cipher suites
certificate trust
proxy
DNS
HTTP version
SNI
ALPN
```

Compare:

```bash
curl -v
openssl s_client
```

---

## 285. Interview: Why Does `curl -k` Work?

Because `-k` disables certificate verification.

It does not prove the certificate is valid.

Never treat it as a production fix.

---

## 286. Interview: How Do You Debug HTTP 503 in EKS?

Check:

```bash
kubectl get ingress
kubectl get svc
kubectl get endpoints
kubectl get pods
```

Then:

```text
ALB target health
readiness probe
health check path
security groups
application port
```

---

## 287. Interview: How Do You Debug HTTP 502 From ALB?

Check:

```text
target health
backend port
protocol
connection reset
application logs
Pod restarts
security groups
```

---

## 288. Interview: How Do You Debug HTTP 504?

Measure:

```text
ALB target response time
application latency
database latency
downstream calls
network
timeouts
```

---

## 289. Interview: How Do You Test an ALB Backend Directly?

Depending on target architecture, test the Service/Pod path from inside the cluster:

```bash
kubectl exec -it <debug-pod> -- curl -v http://service.namespace.svc.cluster.local:port
```

This separates:

```text
ALB
```

from:

```text
Kubernetes/application
```

---

## 290. Interview: What Is `X-Forwarded-For`?

A proxy header commonly used to preserve original client IP information through proxy chains.

---

## 291. Interview: What Is `X-Forwarded-Proto`?

It identifies the protocol used by the original client connection, such as:

```text
https
```

when TLS terminated at a proxy.

---

## 292. Interview: How Do Redirect Loops Happen Behind a Proxy?

Commonly:

```text
Client HTTPS
→ proxy terminates TLS
→ backend sees HTTP
→ backend redirects to HTTPS
→ proxy repeats
```

Correct proxy-forwarded protocol handling prevents this.

---

## 293. Interview: What Is HSTS?

A browser security mechanism that instructs clients to use HTTPS for a domain for a specified period.

---

## 294. Interview: What Is CORS?

A browser security mechanism controlling whether web pages can make cross-origin requests to another origin.

---

## 295. Interview: Why Does CORS Use OPTIONS?

Browsers can send a preflight OPTIONS request to verify that the cross-origin operation is permitted.

---

## 296. Interview: What Is HTTP Keepalive?

Connection persistence allowing multiple HTTP requests to reuse a connection instead of establishing a new TCP connection each time.

---

## 297. Interview: What Is a Reverse Proxy?

A server-side proxy that receives client requests and forwards them to backend servers.

---

## 298. Interview: Forward vs Reverse Proxy?

```text
Forward proxy
→ represents clients

Reverse proxy
→ represents servers
```

---

## 299. Interview: How Does DNS Fit Into HTTPS?

The client first resolves the hostname to an endpoint, then establishes transport and TLS, then sends HTTP over the secure connection.

---

## 300. Interview: What Is the Correct Troubleshooting Order?

Strong answer:

```text
DNS
→ TCP
→ TLS
→ HTTP
→ proxy/load balancer
→ Kubernetes
→ application
```

Do not skip layers.

---

## 301. Production Final Checklist

```text
[ ] DNS resolution verified
[ ] TCP connectivity verified
[ ] TLS certificate valid
[ ] SAN correct
[ ] Certificate chain valid
[ ] TLS policy modern
[ ] SNI working
[ ] ALPN working where required
[ ] HTTP status understood
[ ] Host routing correct
[ ] Path routing correct
[ ] Health checks correct
[ ] Readiness probes correct
[ ] ALB target healthy
[ ] Service endpoints available
[ ] Timeouts configured
[ ] Retries bounded
[ ] Rate limiting configured
[ ] Logs structured
[ ] Trace IDs available
[ ] Metrics monitored
[ ] Secrets protected
[ ] HTTP security headers reviewed
[ ] HSTS reviewed
[ ] CORS reviewed
[ ] Certificate renewal automated
[ ] Incident runbook documented
```

---

## 302. Final Mental Model

For a production HTTPS request:

```text
1. DNS
   hostname → IP

2. TCP
   client → server:443

3. TLS
   certificate
   SNI
   ALPN
   key exchange
   encrypted channel

4. HTTP
   request
   headers
   body

5. Load Balancer
   host/path routing
   health check
   target selection

6. Kubernetes
   Ingress
   Service
   Pod

7. Application
   business logic

8. Dependencies
   database
   cache
   message broker
   external API
```

If you understand this chain, most HTTP/HTTPS production incidents become a structured troubleshooting problem rather than guesswork.

---

## 303. Next File

The next planned file is:

```text
12-Routing-and-Route-Tables.md
```

It will cover:

```text
routing fundamentals
default routes
static routing
dynamic routing
route tables
longest prefix match
next hop
gateway
ARP interaction
Linux routing
ip route
ip rule
policy routing
AWS route tables
VPC routing
public/private subnet routing
NAT routing
IGW
Transit Gateway
VPC peering
EKS routing
Pod routing
VPC CNI
Network debugging
production scenarios
RoboShop
interview questions
```

# End of 11-HTTP-HTTPS-and-TLS.md
