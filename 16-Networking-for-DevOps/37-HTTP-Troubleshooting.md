# 16-Networking-for-DevOps
# 37-HTTP-Troubleshooting

## 1. Purpose

HTTP troubleshooting is a core production DevOps skill.

A modern request can travel through:

```text
Client
  ↓
DNS
  ↓
Internet / VPC routing
  ↓
Firewall / Security Group / NACL
  ↓
CDN / WAF
  ↓
Load Balancer
  ↓
Reverse Proxy / Ingress
  ↓
Service
  ↓
Application
  ↓
Database / External Dependency
```

A failure at any layer can appear to users as:

```text
timeout
connection refused
502
503
504
400
401
403
404
409
429
500
TLS error
slow response
```

This file provides a production-oriented methodology for diagnosing HTTP and HTTPS failures across Linux, AWS, Kubernetes, EKS, Nginx, ALB/NLB, Ingress, APIs and distributed systems.

---

## 2. HTTP Troubleshooting Principle

Never start with:

```text
"the application is down"
```

Start with:

```text
Which layer failed?
```

Separate:

```text
DNS
TCP
TLS
HTTP
application
dependency
```

---

## 3. Layered Request Model

```text
DNS
 ↓
TCP connection
 ↓
TLS handshake
 ↓
HTTP request
 ↓
HTTP response
 ↓
application processing
```

Each layer should be tested independently.

---

## 4. First Production Questions

Ask:

```text
Which URL?
Which HTTP method?
Which client?
Which source network?
Which timestamp?
Which response code?
Which request ID?
Is the failure consistent?
```

---

## 5. Capture the Exact URL

Example:

```text
https://api.example.com/v1/orders
```

Do not troubleshoot only:

```text
api.example.com
```

The path can affect:

```text
routing
authentication
authorization
application logic
```

---

## 6. HTTP Methods

Common methods:

```text
GET
POST
PUT
PATCH
DELETE
HEAD
OPTIONS
```

The same URL can behave differently depending on method.

---

## 7. GET

Typically retrieves a resource.

Example:

```bash
curl -v https://api.example.com/orders
```

---

## 8. POST

Typically submits data or creates a resource.

```bash
curl -v \
  -X POST \
  -H 'Content-Type: application/json' \
  -d '{"name":"test"}' \
  https://api.example.com/orders
```

---

## 9. PUT

Often replaces a resource representation.

---

## 10. PATCH

Typically partially modifies a resource.

---

## 11. DELETE

Requests resource deletion.

---

## 12. HEAD

Similar to GET but requests response headers without the response body.

```bash
curl -I https://example.com
```

---

## 13. OPTIONS

Often used to discover supported methods and for CORS preflight.

```bash
curl -i -X OPTIONS https://api.example.com
```

---

## 14. HTTP Status Code Classes

```text
1xx informational
2xx success
3xx redirection
4xx client-side/request issue
5xx server-side/upstream issue
```

---

## 15. 200 OK

Request succeeded.

But:

```text
HTTP 200
```

does not automatically mean the business operation succeeded.

Applications can return an application-level error inside a 200 response.

---

## 16. 201 Created

Common for successful resource creation.

---

## 17. 202 Accepted

Request accepted for asynchronous processing.

---

## 18. 204 No Content

Successful request with no response body.

---

## 19. 301 Moved Permanently

Permanent redirect.

---

## 20. 302 Found

Temporary redirect.

---

## 21. 304 Not Modified

Used with cache validation.

---

## 22. 307 Temporary Redirect

Preserves the HTTP method during redirect.

---

## 23. 308 Permanent Redirect

Permanent redirect while preserving method semantics.

---

## 24. 400 Bad Request

Common causes:

```text
malformed request
invalid JSON
invalid parameters
invalid syntax
```

---

## 25. 401 Unauthorized

Usually means authentication is required or credentials are invalid.

---

## 26. 403 Forbidden

The server understood the request but refuses authorization.

Possible causes:

```text
IAM/authz
WAF
application authorization
network policy at application layer
```

---

## 27. 404 Not Found

Possible causes:

```text
wrong URL
wrong route
wrong host
Ingress rule mismatch
application route missing
resource does not exist
```

---

## 28. 405 Method Not Allowed

The endpoint exists but does not support the requested HTTP method.

---

## 29. 408 Request Timeout

The server timed out waiting for the request.

---

## 30. 409 Conflict

Often indicates a state conflict.

---

## 31. 413 Content Too Large

Request body exceeds configured limits.

Check:

```text
proxy
load balancer
Ingress
application
```

---

## 32. 414 URI Too Long

The request URI exceeds supported limits.

---

## 33. 415 Unsupported Media Type

Often caused by incorrect:

```text
Content-Type
```

---

## 34. 422 Unprocessable Content

The request syntax may be valid but semantic validation failed.

---

## 35. 429 Too Many Requests

Rate limiting or throttling.

Investigate:

```text
client rate
API gateway
WAF
application
upstream
```

---

## 36. 500 Internal Server Error

Application-side failure.

Do not stop at the load balancer.

Check:

```text
application logs
stack traces
dependencies
resource limits
```

---

## 37. 501 Not Implemented

Server does not support the requested functionality.

---

## 38. 502 Bad Gateway

A proxy/load balancer received an invalid response or could not communicate correctly with an upstream.

Common causes:

```text
backend crash
wrong backend port
connection reset
invalid upstream response
protocol mismatch
```

---

## 39. 503 Service Unavailable

Possible causes:

```text
no healthy targets
service unavailable
overload
maintenance
application unavailable
```

---

## 40. 504 Gateway Timeout

A gateway/proxy waited too long for an upstream response.

Common causes:

```text
slow application
database latency
external dependency
network path
timeout mismatch
```

---

## 41. 505 HTTP Version Not Supported

The server does not support the requested HTTP version.

---

## 42. 507 Insufficient Storage

Can appear when a server cannot store the representation.

---

## 43. 508 Loop Detected

Can indicate an application/proxy processing loop.

---

## 44. 5xx Does Not Always Mean Application Code

A 5xx can originate from:

```text
CDN
WAF
reverse proxy
Ingress
load balancer
service mesh
application
```

Always identify which component generated it.

---

## 45. `curl` Is the Primary Tool

Start with:

```bash
curl -v https://example.com
```

---

## 46. Headers Only

```bash
curl -I https://example.com
```

---

## 47. Follow Redirects

```bash
curl -L -v https://example.com
```

---

## 48. Show Response Headers

```bash
curl -i https://example.com
```

---

## 49. Silent Output

```bash
curl -s https://example.com
```

---

## 50. Fail on HTTP Errors

```bash
curl -f https://example.com
```

---

## 51. Timing With `curl`

```bash
curl -o /dev/null \
  -s \
  -w 'dns=%{time_namelookup} connect=%{time_connect} tls=%{time_appconnect} starttransfer=%{time_starttransfer} total=%{time_total}\n' \
  https://example.com
```

This separates major latency components.

---

## 52. DNS Latency

`time_namelookup` helps indicate time spent resolving the hostname.

---

## 53. TCP Connect Latency

`time_connect` helps identify TCP connection establishment time.

---

## 54. TLS Latency

`time_appconnect` helps identify time until TLS connection establishment.

---

## 55. Time to First Byte

`time_starttransfer` is useful for identifying server/upstream processing latency.

---

## 56. Total Time

`time_total` represents the total transfer duration.

---

## 57. Production Timing Interpretation

Example:

```text
DNS: low
connect: low
TLS: low
TTFB: high
```

Likely:

```text
application/upstream processing
```

---

## 58. Slow DNS

```text
DNS high
connect low
```

Investigate DNS rather than application performance.

---

## 59. Slow TCP

```text
DNS low
connect high
```

Investigate:

```text
routing
network
firewall
packet loss
```

---

## 60. Slow TLS

```text
connect low
TLS high
```

Investigate:

```text
TLS negotiation
certificate chain
proxy
load balancer
CPU
```

---

## 61. Slow TTFB

Likely:

```text
server processing
database
external API
queue
application contention
```

---

## 62. Slow Download

If TTFB is low but total time is high:

```text
response body
network throughput
large payload
slow client
```

may be involved.

---

## 63. Verbose `curl`

```bash
curl -v https://example.com
```

Look for:

```text
Connected
TLS handshake
request headers
response status
response headers
```

---

## 64. IPv4 Only

```bash
curl -4 -v https://example.com
```

---

## 65. IPv6 Only

```bash
curl -6 -v https://example.com
```

---

## 66. IPv4/IPv6 Problem

If:

```bash
curl -4 works
curl -6 fails
```

investigate:

```text
AAAA record
IPv6 routing
security policy
load balancer IPv6 configuration
```

---

## 67. Force a Hostname to an IP

```bash
curl --resolve api.example.com:443:203.0.113.10 \
  https://api.example.com/
```

This is extremely useful for testing a specific endpoint while preserving the hostname and TLS SNI.

---

## 68. `--connect-to`

```bash
curl --connect-to api.example.com:443:203.0.113.20:443 \
  https://api.example.com/
```

Useful for controlled endpoint testing.

---

## 69. Custom Host Header

```bash
curl -H 'Host: api.example.com' http://203.0.113.10/
```

Useful for HTTP virtual-host testing.

For HTTPS, prefer `--resolve` so TLS SNI also uses the intended hostname.

---

## 70. User-Agent Testing

```bash
curl -A 'Mozilla/5.0' https://example.com
```

Useful when behavior depends on User-Agent.

---

## 71. Headers

```bash
curl -H 'Authorization: Bearer <token>' \
  https://api.example.com
```

Never paste real production secrets into shared logs or tickets.

---

## 72. Cookies

```bash
curl -b 'session=<redacted>' https://example.com
```

---

## 73. Save Cookies

```bash
curl -c cookies.txt https://example.com
```

---

## 74. Reuse Cookies

```bash
curl -b cookies.txt https://example.com/account
```

---

## 75. POST JSON

```bash
curl -X POST \
  -H 'Content-Type: application/json' \
  -d '{"key":"value"}' \
  https://api.example.com/resource
```

---

## 76. Read JSON From File

```bash
curl -X POST \
  -H 'Content-Type: application/json' \
  --data @payload.json \
  https://api.example.com/resource
```

---

## 77. HTTP Redirect Debugging

```bash
curl -I https://example.com
curl -IL https://example.com
```

Look for:

```text
Location
status
redirect chain
```

---

## 78. Redirect Loop

Example:

```text
HTTP → HTTPS
HTTPS → HTTP
```

or:

```text
/app → /login
/login → /app
```

---

## 79. Proxy Redirect Problem

A reverse proxy may terminate TLS and forward HTTP internally.

The application must correctly understand the original scheme through trusted proxy headers.

---

## 80. `X-Forwarded-Proto`

Commonly indicates the original request scheme.

Example:

```text
X-Forwarded-Proto: https
```

---

## 81. `X-Forwarded-For`

Commonly carries client/proxy IP information.

---

## 82. `X-Forwarded-Host`

Can carry the original host.

---

## 83. Forwarded Header

Standardized:

```text
Forwarded:
```

may carry:

```text
for
proto
host
```

---

## 84. Trust Proxy Headers Carefully

Never blindly trust client-supplied forwarding headers.

Only trusted proxies should be allowed to set/overwrite them.

---

## 85. HTTP Host Header

HTTP/1.1 requests use:

```text
Host
```

for virtual hosting.

---

## 86. Wrong Host Header

A request reaching the correct IP can still hit the wrong virtual host if the Host header is incorrect.

---

## 87. SNI vs Host

For HTTPS:

```text
TLS SNI
```

selects TLS certificate/context.

Then:

```text
HTTP Host
```

can select application routing.

Both can matter.

---

## 88. TLS SNI Test

```bash
openssl s_client \
  -connect api.example.com:443 \
  -servername api.example.com
```

---

## 89. TLS Certificate Inspection

```bash
openssl s_client \
  -connect api.example.com:443 \
  -servername api.example.com \
  </dev/null
```

---

## 90. Certificate Chain

Inspect:

```text
subject
issuer
validity
SAN
chain
```

---

## 91. SAN

Modern certificate hostname validation uses Subject Alternative Names.

Check that the requested hostname is covered.

---

## 92. TLS Version Testing

```bash
curl --tlsv1.2 -v https://example.com
```

Use supported versions appropriate to your environment.

---

## 93. HTTP/1.1

```bash
curl --http1.1 -v https://example.com
```

---

## 94. HTTP/2

```bash
curl --http2 -v https://example.com
```

Support depends on the installed curl build.

---

## 95. HTTP/2 Negotiation

Check verbose TLS output for:

```text
ALPN
```

---

## 96. HTTP/3

HTTP/3 uses:

```text
QUIC
UDP
```

and is distinct from traditional TCP-based HTTP/1.1 and HTTP/2.

---

## 97. HTTP Version Isolation

If:

```text
HTTP/2 fails
HTTP/1.1 works
```

investigate:

```text
ALPN
proxy
load balancer
HTTP/2 configuration
```

---

## 98. HTTP Headers

Important headers include:

```text
Host
Authorization
Content-Type
Accept
Content-Length
Transfer-Encoding
Connection
User-Agent
Cookie
Cache-Control
Location
Set-Cookie
```

---

## 99. Content-Type

Example:

```text
Content-Type: application/json
```

---

## 100. Accept

Indicates response formats the client can accept.

---

## 101. Content-Length

Specifies message body length where applicable.

Incorrect handling can produce:

```text
truncated requests
timeouts
protocol errors
```

---

## 102. Chunked Transfer Encoding

HTTP/1.1 can use:

```text
Transfer-Encoding: chunked
```

for streaming/chunked bodies.

---

## 103. Large Request Debugging

Check limits at every layer:

```text
CloudFront
WAF
ALB
Ingress
Nginx
application
```

---

## 104. Nginx Client Body Limit

Example:

```nginx
client_max_body_size 20m;
```

If a request exceeds the configured limit, Nginx can reject it.

---

## 105. Kubernetes Ingress Body Limit

Ingress-controller-specific annotations may control body size.

Always verify the controller documentation/version before applying an annotation.

---

## 106. ALB Request Size

AWS load balancer limits can differ from Nginx/application limits.

Check current AWS documentation for exact service limits.

---

## 107. Timeout Chain

Production systems often have:

```text
client timeout
CDN timeout
load balancer timeout
proxy timeout
application timeout
database timeout
```

---

## 108. Timeout Mismatch

Bad example:

```text
client: 30s
proxy: 60s
application: 120s
```

The client may give up before the application completes.

---

## 109. Reverse Timeout Design

A common principle:

```text
inner dependency timeout
<
application timeout
<
proxy/LB timeout
<
client timeout
```

Exact values depend on architecture.

---

## 110. 504 Troubleshooting

When a proxy returns 504:

```text
1. Identify proxy.
2. Check proxy timeout.
3. Check upstream connectivity.
4. Check application latency.
5. Check database/external dependencies.
6. Compare application logs.
```

---

## 111. ALB 504

An ALB 504 can occur when it cannot establish a connection to a target or the target does not respond within the relevant timeout behavior.

Check:

```text
target health
target port
security groups
application latency
```

---

## 112. ALB 502

Possible causes include:

```text
target connection reset
malformed response
protocol issue
target unavailable
```

Check ALB access logs and target/application logs.

---

## 113. ALB 503

Possible causes:

```text
no healthy targets
load balancer unavailable
```

Verify target group health.

---

## 114. Target Health

```bash
aws elbv2 describe-target-health \
  --target-group-arn <arn>
```

---

## 115. ALB Listener

```bash
aws elbv2 describe-listeners \
  --load-balancer-arn <arn>
```

---

## 116. ALB Rules

```bash
aws elbv2 describe-rules \
  --listener-arn <arn>
```

Check:

```text
host conditions
path conditions
priority
target group
redirect
fixed response
```

---

## 117. ALB Access Logs

Use access logs to investigate:

```text
client request
target response
processing time
status
target status
```

Field names and formats should be interpreted according to current AWS documentation.

---

## 118. ALB vs Application Status

A key production question:

```text
Who generated the status code?
```

A 503 may come from:

```text
ALB
Ingress
Nginx
application
```

Do not assume.

---

## 119. Nginx Reverse Proxy

Typical:

```text
Client
 ↓
Nginx
 ↓
upstream application
```

---

## 120. Nginx Configuration Test

```bash
nginx -t
```

---

## 121. Nginx Reload

```bash
nginx -s reload
```

Use the production-approved service/process management method.

---

## 122. Nginx Access Logs

Commonly:

```bash
tail -f /var/log/nginx/access.log
```

---

## 123. Nginx Error Logs

```bash
tail -f /var/log/nginx/error.log
```

---

## 124. Nginx Upstream Failure

Look for messages involving:

```text
connect() failed
upstream timed out
upstream prematurely closed connection
connection reset
```

---

## 125. Nginx 502

Common causes:

```text
wrong upstream port
application not listening
connection reset
protocol mismatch
```

---

## 126. Check Application Listener

```bash
ss -lntp
```

---

## 127. Check Local Backend

```bash
curl -v http://127.0.0.1:8080/health
```

Adjust port/path to your application.

---

## 128. Nginx to Remote Backend

```bash
curl -v http://10.0.1.20:8080/health
```

This separates Nginx configuration from backend connectivity.

---

## 129. Kubernetes Service Debugging

```bash
kubectl get svc
kubectl get endpointslice
```

---

## 130. Service Has No Endpoints

Possible:

```text
selector mismatch
Pods not Ready
wrong labels
```

---

## 131. Check Service Selector

```bash
kubectl get svc <service> -o yaml
```

---

## 132. Check Pod Labels

```bash
kubectl get pods --show-labels
```

---

## 133. EndpointSlice

```bash
kubectl get endpointslice \
  -l kubernetes.io/service-name=<service>
```

---

## 134. Test Service From a Pod

```bash
curl -v http://<service>:<port>/health
```

---

## 135. Test Pod IP

```bash
curl -v http://<pod-ip>:<port>/health
```

If Pod IP works but Service fails:

```text
Service/kube-proxy/CNI
```

becomes more likely.

---

## 136. Test Ingress

```bash
kubectl get ingress -A
kubectl describe ingress <name> -n <namespace>
```

---

## 137. Ingress Host Rule

Verify:

```text
host
path
backend service
port
```

---

## 138. Ingress 404

Possible:

```text
host mismatch
path mismatch
wrong ingress
controller rule
```

---

## 139. Ingress 502

Possible:

```text
backend service
target port
Pod listener
readiness
network connectivity
```

---

## 140. Ingress 503

Often indicates:

```text
no available backend
```

but exact behavior depends on controller.

---

## 141. Ingress Controller Logs

```bash
kubectl logs \
  -n <ingress-namespace> \
  <controller-pod>
```

---

## 142. AWS Load Balancer Controller

For AWS ALB Ingress, inspect:

```bash
kubectl get ingress -A
kubectl describe ingress <name> -n <namespace>
kubectl logs -n kube-system \
  deployment/aws-load-balancer-controller
```

Deployment namespace can vary by installation.

---

## 143. ALB Ingress Architecture

```text
Client
 ↓
DNS
 ↓
ALB
 ↓
Target Group
 ↓
Pod
```

Depending on target type, traffic may use:

```text
instance
```

or:

```text
ip
```

targets.

---

## 144. IP Target Mode

ALB can route directly to Pod IPs when configured with IP targets.

---

## 145. Instance Target Mode

Traffic can route to worker nodes and then through Kubernetes networking to Pods.

---

## 146. HTTP Health Check

A load balancer may use:

```text
HTTP health check
```

with:

```text
path
port
protocol
expected status
```

---

## 147. Health Endpoint

Prefer a lightweight endpoint such as:

```text
/health
```

or:

```text
/ready
```

according to application design.

---

## 148. Readiness vs Liveness

Readiness:

```text
Can receive traffic?
```

Liveness:

```text
Should container be restarted?
```

Do not make liveness depend on every external dependency unless that behavior is intentionally designed.

---

## 149. HTTP Health Check Failure

Possible:

```text
wrong path
wrong port
wrong Host header
auth requirement
application startup
dependency issue
```

---

## 150. Health Check Status

A backend may return:

```text
200
```

to a health check but still fail real user requests.

Health checks should represent meaningful availability without being unnecessarily expensive.

---

## 151. HTTP Keep-Alive

Persistent connections reduce repeated TCP/TLS handshakes.

---

## 152. Keep-Alive Failure

Mismatched idle timeout settings can produce:

```text
connection reset
intermittent 502
broken persistent connections
```

---

## 153. Load Balancer Idle Timeout

Check the configured idle timeout and compare it with:

```text
application
proxy
client
```

timeouts.

---

## 154. Connection Reset

A reset can originate from:

```text
application
proxy
load balancer
firewall
kernel
```

Use packet captures and logs to identify the source.

---

## 155. TCP Reset Investigation

```bash
tcpdump -ni any 'tcp[tcpflags] & tcp-rst != 0'
```

Use a narrower filter in production where possible.

---

## 156. FIN vs RST

FIN:

```text
orderly connection close
```

RST:

```text
connection reset/aborted
```

Interpret within the complete TCP exchange.

---

## 157. HTTP Retries

Retries can hide intermittent failures but can also amplify load.

Use:

```text
bounded retries
exponential backoff
jitter
```

where appropriate.

---

## 158. Retry Storm

If:

```text
application
→ proxy
→ service
```

all retry, a small failure can become a large outage.

---

## 159. Idempotency

Retrying:

```text
GET
```

is generally safer than blindly retrying:

```text
POST
```

because POST may create side effects.

Use application-level idempotency mechanisms where required.

---

## 160. 429 Troubleshooting

Check:

```text
rate limit source
client request rate
headers
API gateway
WAF
application
```

---

## 161. Retry-After

A 429 or 503 response may include:

```text
Retry-After
```

when supported by the service.

Clients should respect service-specific retry guidance.

---

## 162. Rate Limiting Location

Rate limiting can occur at:

```text
CDN
WAF
API Gateway
Ingress
Nginx
application
```

Identify the actual layer.

---

## 163. 403 Troubleshooting

Determine whether 403 is generated by:

```text
WAF
load balancer
Ingress
application
identity system
```

---

## 164. WAF 403

WAF rules can reject requests based on:

```text
IP
headers
URI
body
rate
managed rules
custom rules
```

---

## 165. Application 403

Application authorization can reject a request even when the network path is completely healthy.

---

## 166. 401 vs 403

Common distinction:

```text
401 → authentication problem
403 → authorization/refusal
```

Exact API semantics can vary.

---

## 167. Authentication Debugging

Check:

```text
Authorization header
token expiry
issuer
audience
clock skew
JWKS
```

---

## 168. JWT Debugging

A JWT commonly contains:

```text
header
payload
signature
```

Never expose real production tokens in logs.

---

## 169. Clock Skew

Authentication systems can reject tokens if system clocks are significantly wrong.

Check:

```bash
date -u
timedatectl status
```

---

## 170. CORS

CORS is enforced by browsers.

A curl request may succeed while browser JavaScript fails due to CORS policy.

---

## 171. CORS Preflight

Browsers may send:

```text
OPTIONS
```

before certain cross-origin requests.

---

## 172. CORS Debugging

Check:

```text
Access-Control-Allow-Origin
Access-Control-Allow-Methods
Access-Control-Allow-Headers
Access-Control-Allow-Credentials
```

---

## 173. CORS With Credentials

Credentialed cross-origin requests have stricter rules.

Do not use wildcard origin with credentials where browser rules prohibit it.

---

## 174. CORS Misdiagnosis

If:

```text
curl works
browser fails
```

check CORS and browser-specific behavior.

---

## 175. Browser DevTools

Use:

```text
Network tab
Console
Security
```

to inspect:

```text
request
response
redirect
CORS
TLS/browser errors
```

---

## 176. HTTP Cache

Caching can occur at:

```text
browser
CDN
reverse proxy
application
```

---

## 177. Cache-Control

Important directives include:

```text
max-age
no-cache
no-store
private
public
must-revalidate
```

---

## 178. 304 Debugging

A 304 response means the cached representation can be reused when validation conditions are satisfied.

Inspect:

```text
ETag
If-None-Match
Last-Modified
If-Modified-Since
```

---

## 179. ETag

An ETag identifies a representation version.

---

## 180. Conditional Request

Example:

```text
If-None-Match: "abc123"
```

---

## 181. Cache Bug

If users see stale content:

```text
browser cache
CDN cache
reverse proxy cache
application cache
```

may be involved.

---

## 182. CDN Debugging

Check:

```text
cache status
origin response
TTL
cache key
headers
query strings
cookies
```

---

## 183. CDN Cache Key

A cache key may include:

```text
host
path
query
headers
cookies
```

depending on configuration.

---

## 184. CDN 502/503

A CDN error may indicate:

```text
origin connectivity
origin TLS
origin DNS
origin response
```

not a CDN DNS issue.

---

## 185. CloudFront Origin Debugging

Separate:

```text
client
→ CloudFront
→ origin
```

and test the origin directly where safe and possible.

---

## 186. CloudFront Host Header

Origin configuration determines which host/header information reaches the origin.

Incorrect host routing can cause:

```text
404
403
TLS mismatch
```

---

## 187. HTTP Compression

Headers such as:

```text
Accept-Encoding
Content-Encoding
```

control compression negotiation.

---

## 188. Compression Problem

Check for:

```text
incorrect Content-Encoding
proxy corruption
application compression bug
```

---

## 189. Gzip

Common:

```text
Content-Encoding: gzip
```

---

## 190. Brotli

Common:

```text
Content-Encoding: br
```

---

## 191. Large Response Debugging

Measure:

```text
TTFB
download time
payload size
compression
network throughput
```

---

## 192. HTTP Streaming

Streaming responses can keep a connection open for a long time.

Check proxy/load-balancer idle timeouts.

---

## 193. Server-Sent Events

SSE requires long-lived HTTP connections.

Review:

```text
proxy buffering
idle timeout
connection limits
```

---

## 194. WebSocket

WebSocket begins with an HTTP upgrade and then becomes a persistent connection.

Check:

```text
Upgrade
Connection
proxy support
idle timeouts
```

---

## 195. WebSocket Upgrade

Typical headers:

```text
Connection: Upgrade
Upgrade: websocket
```

---

## 196. Nginx WebSocket

Nginx must be configured to pass the upgrade appropriately.

---

## 197. ALB WebSocket

ALB supports WebSocket connections, but timeout and target behavior must be considered.

---

## 198. HTTP Long Polling

Long polling keeps requests open until data or timeout.

Check all intermediate timeouts.

---

## 199. HTTP Request Queueing

Latency can increase before application execution because of:

```text
connection pool
thread pool
worker queue
proxy queue
load balancer
```

---

## 200. Thread Pool Exhaustion

Symptoms:

```text
high latency
timeouts
5xx
low CPU sometimes
```

Application metrics are required.

---

## 201. Connection Pool Exhaustion

Database or HTTP client connection pools can cause requests to wait.

---

## 202. Database-Induced 504

Path:

```text
Client
 ↓
ALB
 ↓
App
 ↓
DB
```

DB slowdown can surface as ALB 504.

---

## 203. External API-Induced 504

Path:

```text
Client
 ↓
App
 ↓
External API
```

The application may exceed the upstream timeout while waiting.

---

## 204. Dependency Timeout Design

Each dependency should have an explicit timeout.

Avoid infinite waits.

---

## 205. Circuit Breaker

Circuit breakers can prevent repeated calls to a failing dependency.

---

## 206. Bulkheads

Separate connection/thread pools can prevent one dependency from consuming all application resources.

---

## 207. HTTP Observability

Useful metrics:

```text
request rate
error rate
latency
status code
request size
response size
upstream latency
```

---

## 208. RED Method

For services:

```text
Rate
Errors
Duration
```

is a useful baseline.

---

## 209. Golden Signals

Common service signals:

```text
latency
traffic
errors
saturation
```

---

## 210. Access Log Fields

Useful fields:

```text
timestamp
method
host
URI
status
bytes
request time
upstream time
client IP
request ID
user agent
```

---

## 211. Correlation ID

A request ID allows tracing a request across:

```text
load balancer
proxy
application
downstream services
```

---

## 212. `X-Request-ID`

A common correlation header is:

```text
X-Request-ID
```

Implement consistently according to organizational standards.

---

## 213. Distributed Tracing

Use tracing to follow:

```text
HTTP request
→ service
→ database
→ external API
```

---

## 214. Trace vs Log

Trace:

```text
request path and timing
```

Log:

```text
detailed event context
```

Use both.

---

## 215. OpenTelemetry

OpenTelemetry can instrument HTTP clients and servers and export traces/metrics according to the deployment architecture.

---

## 216. HTTP Metrics by Status

Group:

```text
2xx
4xx
5xx
```

but also inspect individual codes such as:

```text
401
403
404
429
502
503
504
```

---

## 217. Error Rate

Example:

```text
5xx / total requests
```

Use meaningful aggregation windows.

---

## 218. Latency Percentiles

Monitor:

```text
p50
p90
p95
p99
```

rather than only averages.

---

## 219. Why Average Latency Is Dangerous

A small percentage of extremely slow requests can be hidden by an average.

---

## 220. HTTP Saturation

Check:

```text
CPU
memory
threads
connections
file descriptors
network
database pool
```

---

## 221. File Descriptor Exhaustion

```bash
ulimit -n
```

and process-level limits can reveal whether an application can accept more sockets/files.

---

## 222. Socket Counts

```bash
ss -s
```

---

## 223. Established Connections

```bash
ss -ant state established
```

---

## 224. Listening Ports

```bash
ss -lntp
```

---

## 225. TIME_WAIT

Large numbers of TIME_WAIT sockets can be normal, but excessive churn may indicate connection-management problems.

Do not tune kernel parameters blindly.

---

## 226. SYN Queue

High connection attempts and backlog pressure can contribute to connection failures.

Inspect application and kernel metrics before changing backlog settings.

---

## 227. Ephemeral Ports

Clients use ephemeral source ports for outbound connections.

Exhaustion can cause connection failures.

---

## 228. NAT Port Exhaustion

In AWS, high-volume outbound connections through a NAT Gateway can exhaust available source ports to a destination.

Symptoms may include intermittent connection failures.

---

## 229. NAT Gateway HTTP Failures

If many Pods share NAT:

```text
Pods
 ↓
NAT Gateway
 ↓
Internet
```

inspect:

```text
connection volume
NAT metrics
destination
retry behavior
```

---

## 230. HTTP and Security Groups

For ALB-to-target traffic, verify:

```text
ALB SG egress
target SG ingress
target port
```

---

## 231. HTTP and NACLs

NACLs are stateless.

Allow required request and return traffic.

---

## 232. HTTP and NetworkPolicy

Kubernetes NetworkPolicy can block:

```text
Ingress
Egress
```

even when AWS security groups permit the traffic.

---

## 233. Layered Security

A request may pass:

```text
AWS SG
```

but fail:

```text
NetworkPolicy
```

or:

```text
application authorization
```

---

## 234. HTTP Request Path Example

```text
Internet
 ↓
Route 53
 ↓
CloudFront
 ↓
WAF
 ↓
ALB
 ↓
Ingress/controller
 ↓
Service
 ↓
Pod
```

Debug each boundary.

---

## 235. Production HTTP Troubleshooting Sequence

```text
1. Reproduce.
2. Capture exact URL/method.
3. Check DNS.
4. Test TCP.
5. Test TLS.
6. Inspect HTTP response.
7. Identify response generator.
8. Check proxy/LB logs.
9. Check target health.
10. Check application logs.
11. Check dependencies.
12. Correlate timestamps/request ID.
```

---

## 236. Scenario: 502 From ALB

```text
curl
 ↓
ALB 502
```

Check:

```text
target health
target port
application listener
target logs
ALB access logs
connection resets
```

---

## 237. Scenario: 503 From ALB

Check:

```bash
aws elbv2 describe-target-health \
  --target-group-arn <arn>
```

If no healthy targets:

```text
readiness
health path
port
SG
application
```

---

## 238. Scenario: 504 From ALB

Check:

```text
target response time
application logs
database latency
external dependency
ALB idle timeout
```

---

## 239. Scenario: 404 From ALB

Check:

```text
listener rule
host
path
target
```

---

## 240. Scenario: 404 From Application

If ALB routes successfully but application returns 404:

```text
application route
method
path rewrite
version
```

---

## 241. Scenario: 403 From WAF

Inspect:

```text
WAF sampled requests/logs
rule match
client IP
URI
headers
```

---

## 242. Scenario: 403 From Application

Inspect:

```text
identity
roles
permissions
JWT
application policy
```

---

## 243. Scenario: 429

Check:

```text
rate limit key
client volume
burst
Retry-After
upstream throttling
```

---

## 244. Scenario: Browser Fails, Curl Works

Check:

```text
CORS
cookies
redirects
browser cache
proxy
TLS/browser policy
```

---

## 245. Scenario: Curl Works, Application Fails

Check:

```text
application proxy
DNS path
runtime DNS cache
certificate trust
environment variables
HTTP client timeout
```

---

## 246. Scenario: Application Works Locally, Fails Through ALB

Check:

```text
Host
X-Forwarded-Proto
health checks
target port
SG
listener
path routing
```

---

## 247. Scenario: Works Through Pod IP, Fails Through Service

Check:

```text
Service selector
EndpointSlice
port
targetPort
kube-proxy/CNI
NetworkPolicy
```

---

## 248. Scenario: Works Through Service, Fails Through Ingress

Check:

```text
Ingress host
path
controller
TLS
backend service
annotations
```

---

## 249. Scenario: Works Through ALB DNS, Fails Through Custom Domain

Check:

```text
Route 53
DNS
certificate
SNI
host routing
```

---

## 250. Scenario: HTTPS Fails but HTTP Works

Check:

```text
certificate
TLS version
SNI
listener
security group
redirect configuration
```

---

## 251. Scenario: HTTP Redirect Loop

Check:

```text
proxy TLS termination
X-Forwarded-Proto
application HTTPS redirect
ALB redirect
Ingress redirect
```

---

## 252. Scenario: Large POST Fails

Check request-size limits at:

```text
CDN
WAF
ALB
Ingress
Nginx
application
```

---

## 253. Scenario: Long Request Times Out

Measure:

```text
DNS
TCP
TLS
TTFB
total
```

Then identify the layer where time is spent.

---

## 254. Scenario: Intermittent 502

Look for:

```text
specific targets
specific AZ
connection resets
application restarts
keep-alive mismatch
```

---

## 255. Scenario: Intermittent 503

Look for:

```text
target health flapping
Pod readiness changes
deployment rollout
capacity shortage
```

---

## 256. Scenario: Intermittent 504

Look for:

```text
latency spikes
database pool
external dependency
CPU throttling
GC pauses
network loss
```

---

## 257. Scenario: Only One Pod Fails

Compare:

```text
Pod IP
Pod logs
readiness
resources
node
environment
```

---

## 258. Scenario: Only One Node Fails

Compare:

```text
node networking
CNI
iptables/eBPF
security groups
route
DNS
```

---

## 259. Scenario: Only One AZ Fails

Compare:

```text
subnet
route
NAT
load balancer targets
AZ-specific dependencies
```

---

## 260. Scenario: Deployment Causes HTTP 503

Check:

```text
readiness
rolling update
maxUnavailable
maxSurge
target registration delay
```

---

## 261. Kubernetes Rolling Deployment

If too many Pods become unavailable simultaneously, capacity can temporarily drop.

---

## 262. Readiness Gate

AWS load balancer integrations may use readiness-related mechanisms to coordinate Pod readiness with target registration.

Understand the exact controller/version behavior in your cluster.

---

## 263. HTTP Graceful Shutdown

Applications should stop accepting new requests and allow active requests to complete according to shutdown policy.

---

## 264. Termination Grace Period

Kubernetes:

```yaml
terminationGracePeriodSeconds:
```

gives the application time to shut down gracefully.

---

## 265. PreStop Hook

A `preStop` hook can help coordinate graceful shutdown, but it must be designed carefully and not treated as a substitute for correct readiness and termination behavior.

---

## 266. Connection Draining

Load balancers can drain connections during target deregistration.

Verify the actual target deregistration and timeout behavior.

---

## 267. HTTP During Deployments

Watch:

```text
5xx
latency
target health
Pod readiness
registration/deregistration
```

---

## 268. Blue/Green HTTP Deployment

Typical:

```text
DNS/LB
 ↓
Blue
```

then controlled switch to:

```text
Green
```

---

## 269. Canary HTTP Deployment

Route a controlled percentage of traffic to a new version.

Monitor:

```text
5xx
latency
business metrics
```

---

## 270. HTTP Smoke Test

After deployment:

```bash
curl -fsS https://api.example.com/health
```

Then test a representative business endpoint.

---

## 271. Health Check Is Not Enough

A successful health check does not prove:

```text
authentication
database operations
critical business path
```

works.

---

## 272. Production Smoke Tests

Include:

```text
DNS
TLS
health
authentication
representative API
```

where appropriate.

---

## 273. HTTP Security Headers

Common:

```text
Strict-Transport-Security
Content-Security-Policy
X-Content-Type-Options
Referrer-Policy
Permissions-Policy
```

Exact requirements depend on the application.

---

## 274. HSTS

```text
Strict-Transport-Security
```

instructs compliant browsers to use HTTPS for the domain according to configured policy.

---

## 275. Security Header Troubleshooting

A security header issue may appear only in browsers, not curl.

Use browser DevTools.

---

## 276. HTTP Request Smuggling

Proxy/backend parsing differences can create security risks.

Keep:

```text
proxy
load balancer
application
```

HTTP parsing behavior compatible and patched.

---

## 277. Host Header Injection

Applications should validate hostnames and trusted proxy behavior.

Do not use arbitrary Host values for security-sensitive redirects or URL generation without validation.

---

## 278. Open Redirect

Incorrect redirect handling can allow attackers to control destination URLs.

Validate redirect targets.

---

## 279. HTTP Header Injection

Sanitize user-controlled values before placing them into response headers.

---

## 280. Sensitive Headers

Protect:

```text
Authorization
Cookie
Set-Cookie
internal tracing headers
```

from accidental logging.

---

## 281. Log Redaction

Never log:

```text
password
API key
access token
session cookie
private secret
```

in plaintext.

---

## 282. HTTP Audit Logs

Record enough context for troubleshooting:

```text
timestamp
request ID
endpoint
status
latency
source
```

while respecting privacy/security requirements.

---

## 283. Production Incident Example

Symptom:

```text
Users receive 504 after deployment.
```

Method:

```text
1. Confirm timing.
2. Compare pre/post deployment latency.
3. Check ALB 504 metrics.
4. Check target response time.
5. Inspect application logs.
6. Inspect DB latency.
7. Roll back if user impact is significant.
```

---

## 284. Incident Example: 503 During Rollout

Possible cause:

```text
readiness probes pass too late
target registration delay
insufficient replicas
```

Check rollout and target health timelines.

---

## 285. Incident Example: 502 After Nginx Change

Check:

```bash
nginx -t
```

Then inspect:

```text
upstream address
upstream port
protocol
DNS
error log
```

---

## 286. Incident Example: 403 After WAF Change

Compare:

```text
rule configuration
deployment time
blocked requests
URI
headers
client source
```

---

## 287. Incident Example: 429 After Traffic Spike

Determine:

```text
normal traffic
new traffic
rate limit threshold
burst
upstream throttling
```

---

## 288. Incident Example: Only POST Fails

Check:

```text
method routing
Content-Type
body size
CSRF
authentication
WAF inspection
```

---

## 289. Incident Example: GET Works, POST 413

Likely request-size or body processing limit.

Compare limits across every proxy layer.

---

## 290. Incident Example: Browser Gets CORS Error

Check:

```text
OPTIONS
Access-Control-Allow-Origin
credentials
allowed methods
allowed headers
```

---

## 291. Incident Example: TLS Works With ALB DNS but Not Custom Host

Use:

```bash
curl --resolve custom.example.com:443:<alb-ip> \
  https://custom.example.com/
```

Then inspect certificate/SNI and host-based routing.

---

## 292. Incident Example: One ALB Target Unhealthy

Compare:

```text
target port
security group
application listener
health endpoint
node/pod
```

against healthy targets.

---

## 293. Incident Example: HTTP Works Internally but Not Externally

Compare:

```text
public DNS
internet gateway
public subnet
security group
NACL
ALB listener
WAF
```

---

## 294. Incident Example: HTTP Works From Node but Not Pod

Compare:

```text
Pod DNS
Pod route
NetworkPolicy
CNI
proxy environment
```

---

## 295. Incident Example: HTTP Works From Pod but Not Through Service

Focus on:

```text
Service
EndpointSlice
ports
selectors
kube-proxy/CNI
```

---

## 296. Incident Example: HTTP Works Through Service but Not Ingress

Focus on:

```text
Ingress
controller
host/path
TLS
backend
```

---

## 297. Incident Example: HTTP Works Through ALB but Not Route 53

Focus on:

```text
DNS record
alias
health/routing policy
```

---

## 298. Incident Example: HTTP Works From One Region Only

Check:

```text
DNS routing
CDN
regional ALB
regional targets
network path
```

---

## 299. Incident Example: Slow Only at Peak

Investigate:

```text
connection pools
CPU
GC
database
autoscaling
NAT
load balancer capacity
```

---

## 300. HTTP Troubleshooting Commands

```bash
curl -v https://example.com
curl -I https://example.com
curl -IL https://example.com
curl -4 -v https://example.com
curl -6 -v https://example.com
curl --http1.1 -v https://example.com
curl --http2 -v https://example.com
curl --resolve host:443:IP https://host/
openssl s_client -connect host:443 -servername host
ss -lntp
ss -s
tcpdump -ni any port 443
kubectl get svc
kubectl get endpointslice
kubectl get ingress -A
kubectl describe ingress <name> -n <namespace>
aws elbv2 describe-target-health --target-group-arn <arn>
```

---

## 301. HTTP Production Decision Tree

```text
Request fails
    |
    v
DNS resolves?
 |        |
NO       YES
 |        |
DNS      TCP connects?
         |        |
        NO       YES
         |        |
      network    TLS works?
                 |       |
                NO      YES
                 |       |
               TLS      HTTP status
                         |
             +-----------+-----------+
             |           |           |
            2xx         4xx         5xx
             |           |           |
          success    request/auth   proxy/app
```

---

## 302. 5xx Decision Tree

```text
5xx
 |
 +-- 502 → upstream connection/response
 |
 +-- 503 → availability/healthy target
 |
 +-- 504 → upstream timeout
 |
 +-- 500 → application/server failure
```

Always verify the component generating the status.

---

## 303. 4xx Decision Tree

```text
4xx
 |
 +-- 400 → malformed/invalid request
 +-- 401 → authentication
 +-- 403 → authorization/WAF
 +-- 404 → routing/resource
 +-- 405 → method
 +-- 409 → state conflict
 +-- 413 → body size
 +-- 415 → content type
 +-- 422 → semantic validation
 +-- 429 → rate limit
```

---

## 304. HTTP Production Checklist

```text
[ ] Exact URL captured
[ ] HTTP method captured
[ ] Source identified
[ ] DNS verified
[ ] TCP verified
[ ] TLS verified
[ ] HTTP status captured
[ ] Response generator identified
[ ] Request ID captured
[ ] LB/proxy logs checked
[ ] Target health checked
[ ] Application logs checked
[ ] Dependency latency checked
[ ] Timeout chain checked
[ ] Retry behavior checked
[ ] Security controls checked
[ ] Recent deployment checked
[ ] Recent DNS/LB/config change checked
```

---

## 305. Interview Preparation: HTTP Troubleshooting

### Question 1: How do you troubleshoot a 502?

Answer:

```text
First I identify which component generated the 502. If it is an ALB
or reverse proxy, I check target health, target port, connectivity,
connection resets and upstream response behavior. I then correlate
load balancer access logs with application logs. I also test the
backend directly to separate proxy problems from application
problems.
```

### Question 2: How do you troubleshoot a 504?

Answer:

```text
I measure DNS, TCP, TLS and time-to-first-byte separately. Then I
check the proxy or load balancer timeout and compare it with
application, database and external dependency latency. A 504 usually
means the upstream did not respond within the relevant timeout, so I
trace the request to the slow dependency rather than simply
increasing the timeout.
```

### Question 3: 502 vs 503 vs 504?

Answer:

```text
502 generally indicates an upstream communication or invalid
response problem, 503 generally indicates unavailable service or
healthy-target/capacity problems, and 504 generally indicates an
upstream timeout. Exact behavior depends on the component generating
the response, so I verify the source.
```

### Question 4: DNS works but curl times out. What next?

Answer:

```text
DNS is not the immediate problem. I test TCP connectivity to port 443,
then inspect routes, security groups, NACLs, firewalls and target
listeners. If TCP succeeds, I continue with TLS and HTTP.
```

### Question 5: Curl works but browser fails?

Answer:

```text
I check CORS, cookies, redirects, browser cache, proxy behavior and
browser TLS/security policies. A browser can enforce policies that
curl does not.
```

### Question 6: How do you troubleshoot an EKS Ingress 502?

Answer:

```text
I verify the Ingress host/path and backend Service, then inspect
EndpointSlices and Pod readiness. I test the Service and Pod directly,
inspect the AWS Load Balancer Controller and ALB target health, and
compare ALB access logs with application logs.
```

### Question 7: How do you troubleshoot an ALB 503?

Answer:

```text
I check target-group health first. If there are no healthy targets,
I inspect health-check path, port, readiness, security groups and
application listener behavior. I also verify whether a deployment
temporarily removed too much capacity.
```

### Question 8: How do you troubleshoot HTTP latency?

Answer:

```text
I measure DNS, connect, TLS, TTFB and total time with curl. If TTFB
is high, I investigate the application and dependencies. If connect
time is high, I investigate networking. If DNS time is high, I
investigate the resolver path.
```

### Question 9: How do you troubleshoot a redirect loop?

Answer:

```text
I inspect every Location response and determine where HTTPS is
terminated. I then verify X-Forwarded-Proto and trusted proxy
configuration. A common problem is a load balancer terminating TLS
while the backend believes the original request was HTTP and keeps
redirecting.
```

### Question 10: How do you troubleshoot 404 through Ingress?

Answer:

```text
I verify the Host header, path, path type, Ingress rule, backend
Service and controller configuration. I test the Service directly.
This tells me whether the 404 is generated by the Ingress controller
or by the application.
```

### Question 11: How do you troubleshoot intermittent 502?

Answer:

```text
I correlate failures by target, timestamp, AZ and Pod. I look for
connection resets, application restarts, readiness changes, target
health flapping and keep-alive/timeout mismatches. Comparing healthy
and unhealthy targets usually narrows the problem quickly.
```

### Question 12: Why can a 200 still indicate failure?

Answer:

```text
HTTP status only describes the HTTP-level response. An application
can return HTTP 200 while putting an error state in the response
body. For production monitoring I combine HTTP metrics with business
success metrics.
```

### Question 13: Why does an HTTPS request need both SNI and Host?

Answer:

```text
SNI is part of TLS negotiation and allows a server or load balancer to
select the appropriate certificate/context. The HTTP Host header then
participates in HTTP virtual-host or application routing. Both can
affect the final request path.
```

### Question 14: How do you test a specific ALB IP while preserving hostname?

Answer:

```bash
curl --resolve api.example.com:443:203.0.113.10 \
  https://api.example.com/
```

This allows controlled testing of a particular endpoint while using
the intended hostname for TLS and HTTP routing.

### Question 15: How do you troubleshoot a large POST failure?

Answer:

```text
I compare request-size limits at every layer: CDN/WAF, load balancer,
Ingress, Nginx and application. I identify which component generated
the error, then adjust the appropriate layer rather than increasing
limits everywhere.
```

### Question 16: What is the difference between readiness and liveness?

Answer:

```text
Readiness determines whether a Pod should receive traffic. Liveness
determines whether the container should be restarted. Readiness is
therefore directly related to load-balancer availability during
deployments.
```

### Question 17: How do you troubleshoot HTTP failure after deployment?

Answer:

```text
I correlate the first failure with deployment time, compare old and
new target behavior, check readiness and target registration, inspect
application logs and metrics, and verify configuration changes. If
user impact is significant and rollback is safe, I follow the
approved rollback procedure while continuing root-cause analysis.
```

### Question 18: What causes connection reset?

Answer:

```text
A reset can be generated by an application, proxy, load balancer,
firewall or kernel. I use TCP captures and logs to identify which
endpoint sent the reset instead of assuming the application caused it.
```

### Question 19: How do retries make incidents worse?

Answer:

```text
Retries can multiply traffic toward an already failing dependency.
Without bounded retries, exponential backoff and jitter, a small
dependency failure can become a retry storm and exhaust application,
connection-pool or downstream capacity.
```

### Question 20: How do you troubleshoot HTTP from a Kubernetes Pod?

Answer:

```text
I test the exact URL from the Pod, inspect DNS, proxy variables and
network connectivity, then test the Service and Pod IP separately.
For external endpoints I also verify VPC routing, NAT and security
controls. I correlate the request with application and load-balancer
logs.
```

---

## 306. Senior Production Scenario

### Problem

```text
Production API starts returning intermittent 504s after traffic
increases.
```

### Investigation

```text
1. DNS is stable.
2. TCP connect latency is normal.
3. TLS latency is normal.
4. TTFB increases sharply.
5. ALB target response time increases.
6. Application CPU is moderate.
7. Database connection pool is saturated.
8. Database query latency increases.
```

### Root Cause

```text
Database connection pool saturation caused application requests to
wait, causing the ALB timeout to be exceeded.
```

### Correct Fix

Do not simply increase ALB timeout.

Investigate:

```text
database capacity
connection pool sizing
slow queries
query indexes
application concurrency
caching
```

---

## 307. Senior Production Scenario: Intermittent 502

```text
ALB
 ↓
10 Pods
```

Nine Pods are healthy.

One Pod produces most 502 responses.

Investigation:

```text
target health flaps
Pod restarts
application listener disappears during restart
```

Root cause:

```text
graceful shutdown/readiness handling was incorrect.
```

Fix:

```text
proper readiness
graceful shutdown
termination handling
deployment strategy
```

---

## 308. Senior Production Scenario: 404 Only Through Domain

Direct ALB DNS works:

```text
https://internal-alb.example
```

Custom domain fails:

```text
https://api.example.com
```

Investigation:

```text
DNS correct
TLS correct
Host header reaches ALB
ALB host rule points to wrong backend
```

Root cause:

```text
host-based routing configuration.
```

---

## 309. Senior Production Scenario: Browser Failure

```text
curl → 200
browser → CORS error
```

Root cause:

```text
API response did not include the required CORS headers.
```

Fix:

```text
correct server-side CORS policy
```

not a DNS or TCP change.

---

## 310. Senior Production Scenario: EKS 503

```text
ALB → 503
```

Investigation:

```text
Ingress exists
Service exists
Pods exist
```

But:

```text
EndpointSlice → empty
```

Root cause:

```text
Service selector did not match Pod labels.
```

---

## 311. Senior Production Scenario: EKS 502

```text
ALB target healthy
ALB → 502
```

Investigation:

```text
Pod listener expects HTTPS
target group sends HTTP
```

Root cause:

```text
backend protocol mismatch.
```

---

## 312. Senior Production Scenario: Large Upload

```text
small uploads → 200
large uploads → 413
```

Investigation:

```text
Nginx client_max_body_size = 10m
application supports 50m
```

Root cause:

```text
proxy request-size limit.
```

---

## 313. Senior Production Scenario: Redirect Loop

```text
Client → HTTPS
ALB terminates TLS
ALB → HTTP
Application sees HTTP
Application redirects to HTTPS
```

Root cause:

```text
application does not correctly trust/interpret the original HTTPS
scheme.
```

---

## 314. Senior Production Scenario: Peak Traffic

```text
latency increases
5xx increases
```

Metrics show:

```text
HTTP connection pool exhausted
```

Root cause:

```text
downstream dependency/connection-pool bottleneck.
```

Fix:

```text
capacity
pool sizing
timeouts
circuit breaker
dependency optimization
```

---

## 315. Senior Production Scenario: HTTP Works on Node

```text
Node → Service works
Pod → Service fails
```

Investigate:

```text
NetworkPolicy
Pod routing
CNI
DNS
proxy environment
```

Do not change AWS security groups before verifying Kubernetes networking.

---

## 316. Senior Production Scenario: HTTP Works Through Service

```text
Pod → Service → 200
Ingress → 404
```

Focus on:

```text
Ingress host
path
path rewrite
controller
```

---

## 317. Senior Production Scenario: HTTP Works Through ALB DNS

```text
ALB DNS → 200
custom domain → 503
```

Check:

```text
Route 53
Host header
ALB listener rule
target group
```

---

## 318. HTTP Troubleshooting Golden Rules

```text
1. Reproduce before changing.
2. Preserve evidence.
3. Identify the response generator.
4. Separate DNS/TCP/TLS/HTTP.
5. Use curl timing.
6. Test from the affected network.
7. Compare direct and proxied paths.
8. Check target health.
9. Correlate logs by request ID/time.
10. Check recent changes.
11. Avoid blind timeout increases.
12. Avoid blind retries.
13. Avoid changing multiple layers simultaneously.
14. Roll back safely when required.
15. Document root cause and preventive action.
```

---

## 319. Final HTTP Cheat Sheet

```bash
# Basic
curl -v https://example.com
curl -I https://example.com
curl -IL https://example.com

# Timing
curl -o /dev/null -s \
  -w 'dns=%{time_namelookup} connect=%{time_connect} tls=%{time_appconnect} ttfb=%{time_starttransfer} total=%{time_total}\n' \
  https://example.com

# Force address
curl --resolve api.example.com:443:203.0.113.10 \
  https://api.example.com/

# HTTP versions
curl --http1.1 -v https://example.com
curl --http2 -v https://example.com

# IPv4/IPv6
curl -4 -v https://example.com
curl -6 -v https://example.com

# JSON
curl -X POST \
  -H 'Content-Type: application/json' \
  -d '{"key":"value"}' \
  https://api.example.com

# TLS
openssl s_client \
  -connect api.example.com:443 \
  -servername api.example.com

# Linux sockets
ss -lntp
ss -s

# Packet capture
tcpdump -ni any port 443

# Kubernetes
kubectl get svc
kubectl get endpointslice
kubectl get ingress -A
kubectl describe ingress <name> -n <namespace>

# AWS
aws elbv2 describe-target-health \
  --target-group-arn <arn>
aws elbv2 describe-listeners \
  --load-balancer-arn <arn>
```

---

## 320. Final Production HTTP Checklist

```text
[ ] DNS resolves correctly
[ ] Correct IP/endpoint verified
[ ] TCP connection verified
[ ] TLS certificate verified
[ ] SNI verified
[ ] HTTP method verified
[ ] Host header verified
[ ] Path verified
[ ] Response status verified
[ ] Response generator identified
[ ] Redirects checked
[ ] Authentication checked
[ ] Authorization checked
[ ] CORS checked
[ ] Request-size limits checked
[ ] Rate limits checked
[ ] Proxy/LB logs checked
[ ] Target health checked
[ ] Application logs checked
[ ] Dependency latency checked
[ ] Timeouts checked
[ ] Retries checked
[ ] Recent deployment checked
[ ] Recent infrastructure/config changes checked
```

---

## 321. End State

After completing this file, the DevOps engineer should be able to troubleshoot an HTTP request from:

```text
DNS
 ↓
TCP
 ↓
TLS
 ↓
HTTP
 ↓
CDN/WAF
 ↓
ALB/NLB
 ↓
Nginx/Ingress
 ↓
Kubernetes Service
 ↓
Pod
 ↓
Application
 ↓
Database/External Dependency
```

without treating every failure as an "application issue."

# End of 37-HTTP-Troubleshooting.md
