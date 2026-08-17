# Datetime-and-Time

## DevOps Focus

Time handling looks simple until it breaks production automation.

DevOps scripts constantly work with:

- deployment timestamps
- log timestamps
- AWS resource creation times
- Kubernetes events
- token expiration
- certificate expiration
- CI/CD duration
- retry delays
- timeouts
- maintenance windows
- backups
- monitoring windows
- SLA/SLO calculations
- incident timelines
- scheduled jobs
- file age
- cleanup policies

The most important production rule is:

> **Use timezone-aware UTC timestamps for machine-to-machine data whenever possible. Convert to local time only for human presentation.**

---

## 1. `datetime` Module

Python provides:

```python
from datetime import datetime
```

Current local time:

```python
now = datetime.now()

print(now)
```

Current UTC time:

```python
from datetime import datetime, timezone

now = datetime.now(timezone.utc)

print(now)
```

For production automation, prefer the timezone-aware UTC version.

---

## 2. `date`

Use `date` when you care only about the calendar date.

```python
from datetime import date

today = date.today()

print(today)
```

Example:

```text
2026-08-16
```

Useful for:

```text
reports
maintenance dates
retention dates
billing periods
```

---

## 3. `time`

`time` represents a clock time without a calendar date.

```python
from datetime import time

maintenance = time(
    hour=2,
    minute=30,
)

print(maintenance)
```

Output:

```text
02:30:00
```

Be careful when scheduling real events: a time without a date or timezone is often ambiguous.

---

## 4. `datetime`

A `datetime` combines:

```text
date + time
```

Example:

```python
from datetime import datetime

deployment = datetime(
    2026,
    8,
    16,
    10,
    30,
)

print(deployment)
```

This is a **naive datetime** because it has no timezone information.

---

## 5. Naive vs Aware Datetime

Naive:

```python
datetime.now()
```

Aware:

```python
datetime.now(timezone.utc)
```

A naive datetime does not tell you which timezone it represents.

An aware datetime has timezone information.

For distributed DevOps systems, timezone-aware values are safer.

---

## 6. Why Naive Datetimes Cause Production Bugs

Imagine:

```text
Server A -> UTC
Server B -> IST
Server C -> UTC
```

A timestamp:

```text
10:00
```

is ambiguous.

It could mean:

```text
10:00 UTC
```

or:

```text
10:00 IST
```

Comparisons, expiration checks, and incident timelines can become wrong.

---

## 7. Always Prefer UTC for Internal Data

Recommended:

```python
from datetime import datetime, timezone

timestamp = datetime.now(
    timezone.utc
)
```

Store and exchange:

```text
UTC
```

Convert to local timezone only when displaying information to humans.

---

## 8. ISO 8601

A common timestamp format is:

```text
2026-08-16T10:30:00+00:00
```

This is ISO 8601 style.

Python:

```python
timestamp = datetime.now(
    timezone.utc
)

print(timestamp.isoformat())
```

Output resembles:

```text
2026-08-16T10:30:00+00:00
```

---

## 9. UTC `Z` Notation

Many systems use:

```text
2026-08-16T10:30:00Z
```

`Z` means:

```text
UTC
```

Python's `isoformat()` commonly produces:

```text
+00:00
```

Both represent UTC.

If a system specifically requires `Z`, format it according to that API's contract.

---

## 10. Parse ISO 8601

Python can parse common ISO timestamps:

```python
from datetime import datetime

text = (
    "2026-08-16T10:30:00+00:00"
)

timestamp = datetime.fromisoformat(
    text
)

print(timestamp)
```

The resulting datetime is timezone-aware.

---

## 11. Parse `Z`

Modern Python versions support:

```python
text = (
    "2026-08-16T10:30:00Z"
)

timestamp = datetime.fromisoformat(
    text
)
```

If supporting older Python versions or inconsistent external formats, normalize the input according to the API contract.

---

## 12. Formatting With `strftime`

Example:

```python
timestamp.strftime(
    "%Y-%m-%d %H:%M:%S"
)
```

Common format codes:

```text
%Y -> year
%m -> month
%d -> day
%H -> hour
%M -> minute
%S -> second
```

Example:

```text
2026-08-16 10:30:00
```

---

## 13. `strptime`

`strptime()` parses a known string format.

```python
text = (
    "2026-08-16 10:30:00"
)

timestamp = datetime.strptime(
    text,
    "%Y-%m-%d %H:%M:%S",
)
```

Use this when the input format is known and fixed.

---

## 14. `strftime` vs `strptime`

Remember:

```text
strftime -> datetime -> string

strptime -> string -> datetime
```

Example:

```python
text = timestamp.strftime(
    "%Y-%m-%d"
)

date_value = datetime.strptime(
    text,
    "%Y-%m-%d",
)
```

---

## 15. `timedelta`

Use `timedelta` for durations.

```python
from datetime import timedelta

duration = timedelta(
    minutes=30
)

print(duration)
```

Other values:

```python
timedelta(
    seconds=10
)

timedelta(
    minutes=5
)

timedelta(
    hours=2
)

timedelta(
    days=7
)
```

---

## 16. Add Time

```python
from datetime import datetime, timedelta, timezone

now = datetime.now(
    timezone.utc
)

future = now + timedelta(
    hours=2
)

print(future)
```

Useful for:

```text
expiration
maintenance windows
retention
scheduled checks
```

---

## 17. Subtract Time

```python
past = now - timedelta(
    days=7
)
```

Useful for:

```text
logs older than 7 days
files older than 30 days
events in the previous hour
```

---

## 18. Calculate Duration

```python
start = datetime.now(
    timezone.utc
)

# operation

end = datetime.now(
    timezone.utc
)

duration = end - start

print(duration)
```

The result is a `timedelta`.

---

## 19. Deployment Duration

A deployment script can record:

```text
deployment started
deployment finished
```

Then calculate:

```python
duration = finished - started

print(
    f"Deployment duration: {duration}"
)
```

This can feed CI/CD reporting.

---

## 20. Seconds From Duration

```python
seconds = duration.total_seconds()

print(seconds)
```

Useful when:

```text
comparing against thresholds
recording metrics
calculating SLO windows
```

Example:

```python
if duration.total_seconds() > 600:
    print(
        "Deployment exceeded 10 minutes"
    )
```

---

## 21. Milliseconds

```python
milliseconds = (
    duration.total_seconds()
    * 1000
)
```

Use milliseconds when the external system expects them.

Do not confuse:

```text
seconds
milliseconds
microseconds
```

---

## 22. Unix Epoch

Unix time represents elapsed time since:

```text
1970-01-01 00:00:00 UTC
```

Example:

```python
timestamp = datetime.now(
    timezone.utc
)

epoch = timestamp.timestamp()

print(epoch)
```

Example output:

```text
1786876200.123
```

The exact value depends on the current time.

---

## 23. Epoch to Datetime

```python
from datetime import datetime, timezone

epoch = 1786876200

timestamp = datetime.fromtimestamp(
    epoch,
    tz=timezone.utc,
)

print(timestamp)
```

Always know whether an API expects:

```text
seconds
```

or:

```text
milliseconds
```

---

## 24. Epoch Milliseconds

Some APIs use:

```text
Unix milliseconds
```

Convert:

```python
milliseconds = int(
    timestamp.timestamp() * 1000
)
```

Convert back:

```python
timestamp = datetime.fromtimestamp(
    milliseconds / 1000,
    tz=timezone.utc,
)
```

---

## 25. Seconds vs Milliseconds Bug

A classic production mistake:

```text
seconds = 1786876200
```

versus:

```text
milliseconds = 1786876200000
```

Passing milliseconds where seconds are expected can produce a wildly incorrect date.

Always check the API contract.

---

## 26. Timezone

Python provides:

```python
from datetime import timezone
```

UTC:

```python
timezone.utc
```

Timezone-aware datetime:

```python
datetime.now(
    timezone.utc
)
```

For named geographic timezones, use Python's `zoneinfo`.

---

## 27. `zoneinfo`

Python provides:

```python
from zoneinfo import ZoneInfo
```

Example:

```python
india = ZoneInfo(
    "Asia/Kolkata"
)

now = datetime.now(
    india
)

print(now)
```

This is preferable to manually adding or subtracting hours.

---

## 28. Why Manual Timezone Math Is Dangerous

Bad:

```python
local = utc + timedelta(
    hours=5,
    minutes=30,
)
```

This assumes a fixed offset.

Named timezone rules are more reliable:

```python
ZoneInfo(
    "Asia/Kolkata"
)
```

For regions with daylight-saving changes, manual offsets can produce incorrect results.

---

## 29. Convert UTC to IST

```python
utc_time = datetime.now(
    timezone.utc
)

india_time = utc_time.astimezone(
    ZoneInfo("Asia/Kolkata")
)

print(india_time)
```

Keep UTC internally and convert for display.

---

## 30. Convert Between Timezones

```python
source = ZoneInfo(
    "America/New_York"
)

target = ZoneInfo(
    "Asia/Kolkata"
)

time_value = datetime.now(
    source
)

converted = time_value.astimezone(
    target
)
```

`astimezone()` converts an aware datetime.

---

## 31. Daylight Saving Time

Some regions change clocks during the year.

For example:

```text
UTC offset can change
```

Do not encode DST rules manually.

Use:

```python
ZoneInfo("America/New_York")
```

when working with that region.

---

## 32. India Time

India uses:

```text
Asia/Kolkata
```

The standard offset is:

```text
UTC+05:30
```

India does not use the same seasonal DST changes seen in many other regions.

Still, use the named timezone rather than hard-coding `+05:30` when representing India as a geographic timezone.

---

## 33. Timezone-Aware Comparison

Good:

```python
now = datetime.now(
    timezone.utc
)

expiry = datetime.fromisoformat(
    expiry_text
)

if now >= expiry:
    print("Expired")
```

Both values must be comparable aware datetimes.

---

## 34. Naive/Aware Comparison Error

This is problematic:

```python
naive = datetime.now()

aware = datetime.now(
    timezone.utc
)

naive < aware
```

Python can raise:

```text
TypeError
```

because the two values have incompatible timezone semantics.

---

## 35. Normalize Before Comparing

If external data may use different timezones:

```python
expiry_utc = expiry.astimezone(
    timezone.utc
)

now_utc = datetime.now(
    timezone.utc
)

if now_utc >= expiry_utc:
    ...
```

Normalize machine comparisons to UTC.

---

## 36. Token Expiration

A DevOps script may need to check:

```text
AWS token
JWT
temporary credential
certificate
API credential
```

Example:

```python
if expiry <= datetime.now(
    timezone.utc
):
    raise RuntimeError(
        "Credential expired"
    )
```

---

## 37. Expiration Buffer

Do not wait until the exact expiration moment.

Use a safety buffer:

```python
refresh_at = expiry - timedelta(
    minutes=5
)

if datetime.now(
    timezone.utc
) >= refresh_at:
    refresh()
```

This helps prevent failures caused by network latency and clock differences.

---

## 38. Certificate Expiration

Conceptual workflow:

```text
certificate expiry
      |
      v
parse timestamp
      |
      v
UTC
      |
      v
expiry - now
      |
      v
days remaining
      |
      v
alert
```

Example:

```python
remaining = (
    expiry - now
).total_seconds()

if remaining < 7 * 86400:
    print(
        "Certificate expires soon"
    )
```

---

## 39. File Age

For a file:

```python
from pathlib import Path
import time

path = Path("app.log")

age_seconds = (
    time.time()
    - path.stat().st_mtime
)

print(age_seconds)
```

Useful for:

```text
stale logs
old backups
temporary files
artifact cleanup
```

---

## 40. File Modified Time

```python
mtime = path.stat().st_mtime
```

Convert to UTC:

```python
modified = datetime.fromtimestamp(
    mtime,
    tz=timezone.utc,
)
```

This gives a timezone-aware timestamp.

---

## 41. Cleanup Old Files

Concept:

```python
cutoff = datetime.now(
    timezone.utc
) - timedelta(
    days=30
)
```

For each file:

```text
modified < cutoff
```

then apply your retention policy.

Never delete production data based only on a hard-coded age without verifying the correct path and retention requirements.

---

## 42. Log Timestamp Parsing

Logs may contain:

```text
2026-08-16T10:30:00Z
```

Parse:

```python
timestamp = datetime.fromisoformat(
    text.replace("Z", "+00:00")
)
```

Then normalize:

```python
timestamp = timestamp.astimezone(
    timezone.utc
)
```

Only use string replacement when the input format is known.

---

## 43. Log Time Windows

To find events in the last hour:

```python
now = datetime.now(
    timezone.utc
)

cutoff = now - timedelta(
    hours=1
)

if event_time >= cutoff:
    process(event)
```

This is common in monitoring and incident automation.

---

## 44. Future Event Detection

```python
if event_time > now:
    print(
        "Event is in the future"
    )
```

Useful for:

```text
scheduled maintenance
certificate expiry
deployment windows
renewal dates
```

---

## 45. Incident Timeline

A production incident may contain:

```text
10:03 alert fired
10:05 engineer acknowledged
10:12 rollback started
10:18 service recovered
```

Convert all timestamps to one timezone:

```text
UTC
```

Then calculate:

```text
time to acknowledge
time to mitigate
time to recover
```

---

## 46. MTTA

Mean Time to Acknowledge:

```text
acknowledged_at - alert_at
```

Python:

```python
mtta = (
    acknowledged_at
    - alert_at
)

print(
    mtta.total_seconds()
)
```

---

## 47. MTTR

Mean Time to Recovery:

```text
recovered_at - incident_start
```

Example:

```python
mttr = (
    recovered_at
    - incident_start
)
```

Aggregate across incidents for operational reporting.

---

## 48. SLA Calculation

Suppose an SLA requires:

```text
99.9% availability
```

For a 30-day period, the allowable downtime is approximately:

```text
30 days * 24 hours * 0.1%
```

Python:

```python
period = timedelta(
    days=30
)

allowed = (
    period.total_seconds()
    * 0.001
)
```

Always use the contract's exact measurement window and exclusions.

---

## 49. SLO Window

An SLO script may calculate:

```text
window start
window end
total duration
bad duration
good duration
error budget
```

Time arithmetic should use timezone-aware UTC timestamps.

---

## 50. Error Budget

If:

```text
SLO = 99.9%
```

then:

```text
error budget = 0.1%
```

The exact budget depends on the measurement period.

Python can calculate:

```python
budget = total_seconds * (
    1 - slo
)
```

where:

```python
slo = 0.999
```

---

## 51. Deployment Window

Example:

```python
window_start = datetime(
    2026,
    8,
    16,
    22,
    0,
    tzinfo=timezone.utc,
)

window_end = window_start + timedelta(
    hours=2
)

now = datetime.now(
    timezone.utc
)

if window_start <= now <= window_end:
    print("Deployment allowed")
```

In production, obtain the approved window from the authoritative scheduling system.

---

## 52. Maintenance Window

Do not rely on local server time.

Use:

```text
authoritative timezone
UTC internally
explicit timezone conversion for display
```

This prevents different servers from interpreting the same maintenance window differently.

---

## 53. Retry Delay

Simple retry:

```python
import time

for attempt in range(3):
    try:
        operation()
        break
    except Exception:
        time.sleep(5)
```

For production, catch specific exceptions and use bounded retry policies.

---

## 54. Exponential Backoff

Concept:

```text
1s
2s
4s
8s
16s
```

Python:

```python
delay = 2 ** attempt

time.sleep(delay)
```

Add a maximum:

```python
delay = min(
    2 ** attempt,
    60,
)
```

---

## 55. Exponential Backoff With Jitter

If thousands of clients retry at exactly the same time, they can create a retry storm.

Use jitter:

```python
import random

delay = min(
    2 ** attempt,
    60,
)

delay += random.uniform(
    0,
    1,
)

time.sleep(delay)
```

Production SDKs may already implement retry policies, so do not duplicate them blindly.

---

## 56. Retry Policy

A robust retry system defines:

```text
maximum attempts
initial delay
maximum delay
backoff factor
jitter
retryable errors
deadline
```

Do not retry:

```text
every exception
authentication failures
invalid configuration
permanent validation errors
```

---

## 57. Timeout vs Retry

A timeout controls:

```text
how long one operation waits
```

A retry controls:

```text
how many times an operation is attempted
```

They are different.

Example:

```text
request timeout = 10 seconds
max attempts = 3
```

Total wall-clock time can exceed 10 seconds because retries add additional time.

---

## 58. Operation Deadline

A better production model can use an overall deadline:

```text
start
 |
 +--> attempt
 |
 +--> retry
 |
 +--> retry
 |
 v
deadline reached
```

This prevents a retry loop from running indefinitely.

---

## 59. Measuring Function Duration

For elapsed runtime, prefer a monotonic clock:

```python
import time

start = time.monotonic()

operation()

elapsed = (
    time.monotonic()
    - start
)

print(elapsed)
```

Why?

Because wall-clock time can jump due to:

```text
NTP correction
manual clock changes
daylight-saving transitions
```

`monotonic()` is designed for measuring elapsed time.

---

## 60. `time.time()` vs `time.monotonic()`

Use:

```text
time.time()
```

for:

```text
Unix timestamp
wall-clock time
file timestamps
```

Use:

```text
time.monotonic()
```

for:

```text
duration
timeouts
elapsed runtime
retry deadlines
```

This distinction is extremely important in production automation.

---

## 61. Timeout Example

```python
start = time.monotonic()

while True:
    if check_ready():
        break

    if (
        time.monotonic() - start
        > 300
    ):
        raise TimeoutError(
            "Service did not become ready"
        )

    time.sleep(5)
```

This is safer than comparing wall-clock timestamps for elapsed timeout logic.

---

## 62. Kubernetes Rollout Wait

A Python script may poll:

```text
deployment status
```

until:

```text
available replicas == desired replicas
```

Use:

```python
deadline = (
    time.monotonic()
    + 300
)
```

Then stop polling when the deadline is reached.

---

## 63. Polling Interval

Do not poll every millisecond.

Use:

```python
time.sleep(5)
```

or an appropriate interval.

Polling should balance:

```text
responsiveness
API load
cost
rate limits
```

---

## 64. Polling With Deadline

```python
deadline = (
    time.monotonic()
    + 300
)

while time.monotonic() < deadline:
    status = get_status()

    if status == "READY":
        break

    time.sleep(5)
else:
    raise TimeoutError(
        "Timed out waiting for READY"
    )
```

This is a common DevOps automation pattern.

---

## 65. Clock Skew

Distributed systems may have slightly different clocks.

Example:

```text
server A -> 10:00:00
server B -> 09:59:57
```

A three-second difference can affect:

```text
token validation
TLS
distributed logs
event ordering
monitoring
```

Use synchronized system clocks and avoid assuming exact equality between timestamps from different systems.

---

## 66. NTP

Linux systems commonly synchronize time using services such as:

```text
chrony
systemd-timesyncd
```

DevOps engineers should understand that time synchronization is infrastructure.

If timestamps are inconsistent across nodes, check:

```bash
timedatectl
```

and the host's time synchronization service.

---

## 67. Timezone Configuration in Linux

Check:

```bash
timedatectl
```

Example information:

```text
Local time
Universal time
RTC time
Time zone
NTP synchronized
```

For distributed production systems, UTC is often preferred for server operations.

---

## 68. Kubernetes Time

Kubernetes components and containers commonly use UTC-oriented timestamps.

Do not assume:

```text
container timezone == developer laptop timezone
```

If application logs need local presentation, convert them at the visualization/reporting layer where possible.

---

## 69. Docker Container Time

Containers normally use the host's system clock but may have different timezone configuration.

Avoid application logic that depends on a particular local timezone.

Prefer:

```text
UTC timestamps
```

inside application and infrastructure automation.

---

## 70. AWS Timestamps

AWS APIs commonly return timezone-aware timestamps.

When comparing them:

```python
now = datetime.now(
    timezone.utc
)

if resource_time < now:
    ...
```

Normalize external values to UTC if required by the API/client representation.

---

## 71. Kubernetes Event Timestamps

Kubernetes objects can contain timestamps such as:

```text
creationTimestamp
lastTransitionTime
```

Example:

```python
created = datetime.fromisoformat(
    metadata["creationTimestamp"]
)
```

Use these for:

```text
resource age
rollout duration
incident timelines
```

---

## 72. Resource Age

```python
age = (
    datetime.now(timezone.utc)
    - created
)

print(
    age.total_seconds()
)
```

This can identify:

```text
old pods
stale resources
long-running jobs
```

---

## 73. Kubernetes Job Duration

For a completed Job:

```text
startTime
completionTime
```

Then:

```python
duration = (
    completion
    - start
)
```

Useful for detecting:

```text
slow migrations
slow backups
slow batch processing
```

---

## 74. CI/CD Pipeline Duration

Record:

```text
pipeline_start
pipeline_end
```

Calculate:

```python
duration = (
    pipeline_end
    - pipeline_start
)
```

Track over time to identify:

```text
build slowdown
test slowdown
deployment slowdown
```

---

## 75. Build Queue Time

A CI/CD pipeline has multiple time components:

```text
queue time
build time
test time
security scan time
deployment time
```

Track each separately.

Total duration alone does not tell you where the bottleneck is.

---

## 76. Retry Storm Detection

If many operations retry simultaneously:

```text
API failure
 |
 +--> client A retry
 +--> client B retry
 +--> client C retry
 ...
```

Use:

```text
exponential backoff
jitter
bounded retries
server-provided retry hints
```

when supported.

---

## 77. HTTP Retry-After

Some APIs return:

```text
Retry-After
```

The value may represent:

```text
seconds
```

or:

```text
HTTP date
```

Parse according to the HTTP specification and client behavior rather than assuming one format.

---

## 78. Cache Expiration

A cache entry can contain:

```python
expires_at = datetime.now(
    timezone.utc
) + timedelta(
    minutes=10
)
```

Check:

```python
if datetime.now(
    timezone.utc
) >= expires_at:
    refresh()
```

For high-performance in-process TTL logic, monotonic time can also be appropriate because the requirement is elapsed duration rather than wall-clock time.

---

## 79. TTL

TTL means:

```text
Time To Live
```

Examples:

```text
DNS
cache
temporary credentials
session tokens
locks
temporary files
```

The implementation should distinguish:

```text
absolute expiry timestamp
```

from:

```text
elapsed TTL
```

---

## 80. Lease Expiration

Distributed systems often use leases:

```text
lease acquired
   |
   v
expires after N seconds
```

Use a monotonic deadline for local elapsed-time checks when appropriate:

```python
deadline = (
    time.monotonic()
    + lease_seconds
)
```

---

## 81. Lock Timeout

Example:

```python
deadline = (
    time.monotonic()
    + 30
)

while not acquire_lock():
    if time.monotonic() >= deadline:
        raise TimeoutError(
            "Lock acquisition timed out"
        )

    time.sleep(1)
```

This prevents infinite waits.

---

## 82. Cron vs Python Time Logic

Cron is useful for:

```text
run every day
run every hour
scheduled jobs
```

Python is useful for:

```text
calculate expiry
calculate duration
validate windows
retry
poll
```

Use the right layer for the responsibility.

---

## 83. Scheduled Maintenance

A production maintenance script may receive:

```yaml
maintenance:
  start: "2026-08-20T22:00:00Z"
  end: "2026-08-20T23:00:00Z"
```

Python:

```python
start = datetime.fromisoformat(
    config["maintenance"]["start"]
)

end = datetime.fromisoformat(
    config["maintenance"]["end"]
)

now = datetime.now(timezone.utc)

if start <= now <= end:
    run_maintenance()
```

Validate that:

```text
start < end
```

---

## 84. Date Arithmetic

```python
from datetime import date, timedelta

today = date.today()

tomorrow = (
    today + timedelta(days=1)
)

next_week = (
    today + timedelta(days=7)
)
```

Use `date` when the time-of-day is irrelevant.

---

## 85. Month Arithmetic

A month is not always the same number of days.

Do not approximate:

```text
1 month = 30 days
```

when business rules require calendar months.

For calendar-month arithmetic, use an approved date utility such as `dateutil.relativedelta` if the project allows that dependency.

---

## 86. Business Dates

Examples:

```text
first day of month
last day of month
next business day
billing period
certificate renewal date
```

These require calendar-aware logic, not just `timedelta(days=30)`.

---

## 87. Time Parsing From Configuration

Prefer ISO 8601:

```yaml
start: "2026-08-16T22:00:00Z"
```

Avoid ambiguous strings:

```text
16/08/2026 10:00
```

because different systems may interpret them differently.

---

## 88. Timestamp Naming

Good:

```text
created_at
updated_at
expires_at
started_at
completed_at
```

Avoid vague names:

```text
time
date
timestamp
```

when multiple time meanings exist.

---

## 89. UTC Naming

For clarity, some systems use:

```text
created_at_utc
```

This can be useful when a schema has legacy naive timestamps.

But if your entire contract guarantees UTC, a consistent `created_at` convention may be cleaner.

Document the contract.

---

## 90. API Timestamp Contract

Document:

```text
format
timezone
precision
optional/required
```

Example:

```text
created_at:
ISO 8601
UTC
milliseconds optional
required
```

This avoids cross-team ambiguity.

---

## 91. Microseconds

Python can represent microseconds:

```python
timestamp.microsecond
```

Example:

```text
10:30:00.123456
```

Do not assume downstream systems preserve microsecond precision.

---

## 92. Precision Loss

A timestamp may pass through:

```text
Python
 -> JSON
 -> API
 -> database
 -> log
```

and lose precision.

If exact ordering matters, define the required precision explicitly.

---

## 93. Timestamp Ordering

When events occur very close together, timestamps alone may not establish a total ordering.

Distributed systems may require:

```text
sequence numbers
request IDs
event IDs
database ordering
logical clocks
```

Do not assume:

```text
timestamp A < timestamp B
```

always means A caused B.

---

## 94. Incident Correlation

Use:

```text
timestamp
request ID
trace/correlation ID
service
host/pod
```

to correlate events.

Time is one dimension of observability, not the entire correlation mechanism.

---

## 95. Log Window Script

```python
from datetime import datetime, timedelta, timezone

now = datetime.now(timezone.utc)
cutoff = now - timedelta(minutes=15)

for event in events:
    event_time = datetime.fromisoformat(
        event["timestamp"]
    )

    if event_time >= cutoff:
        process(event)
```

This pattern is common in monitoring automation.

---

## 96. Monitoring Silence Detection

A monitoring script may check:

```text
last successful heartbeat
```

If:

```python
now - last_seen > timedelta(
    minutes=5
):
```

then:

```text
service may be stale
```

Use a clear threshold and account for expected delays.

---

## 97. Heartbeat Monitoring

```text
service
   |
   v
heartbeat
   |
   v
timestamp
   |
   v
monitor
   |
   +--> fresh -> healthy
   |
   +--> stale -> alert
```

This is a practical DevOps use case for time arithmetic.

---

## 98. Backup Age Check

```python
age = (
    datetime.now(timezone.utc)
    - backup_time
)

if age > timedelta(hours=24):
    alert("Backup is stale")
```

Do not rely only on file modification time if the backup system provides authoritative completion timestamps.

---

## 99. Certificate Renewal Window

```python
remaining = (
    expiry
    - datetime.now(timezone.utc)
)

if remaining <= timedelta(days=30):
    print(
        "Renewal window reached"
    )
```

The threshold should match the organization's renewal process.

---

## 100. Token Refresh Window

```python
refresh_window = timedelta(
    minutes=5
)

if expiry - datetime.now(
    timezone.utc
) <= refresh_window:
    refresh_token()
```

This reduces failures caused by refreshing too late.

---

## 101. Time-Based Cleanup

Example:

```python
cutoff = datetime.now(
    timezone.utc
) - timedelta(days=7)
```

Then remove only objects that satisfy:

```text
known path
known type
older than cutoff
not excluded
```

Add a dry-run mode before enabling deletion.

---

## 102. Cleanup Dry Run

Good script behavior:

```bash
python cleanup.py --dry-run
```

Output:

```text
Would delete:
  /tmp/report-1
  /tmp/report-2
```

Only after verification:

```bash
python cleanup.py
```

This is an important production safety pattern.

---

## 103. Time-Based Alert Thresholds

Example:

```python
if age > timedelta(
    minutes=10
):
    alert()
```

Avoid scattered magic numbers.

Prefer configuration:

```yaml
stale_threshold_minutes: 10
```

Then validate the configuration.

---

## 104. Configuration-Driven Timing

```python
threshold = timedelta(
    minutes=config[
        "stale_threshold_minutes"
    ]
)
```

This makes the automation easier to operate across:

```text
dev
staging
production
```

---

## 105. Timeouts in API Calls

Use client-level timeouts:

```python
requests.get(
    url,
    timeout=10,
)
```

Do not rely on an infinite network wait.

A timeout is a production safety mechanism.

---

## 106. Connect Timeout vs Read Timeout

Many HTTP clients support separate concepts:

```text
connect timeout
read timeout
```

For example:

```python
timeout = (
    5,
    30,
)
```

where supported by the client.

The exact meaning depends on the library.

---

## 107. Timeout Budget

Suppose:

```text
overall operation = 60 seconds
```

and it calls:

```text
service A
service B
service C
```

Do not give each call:

```text
60 seconds
```

or total execution could exceed the intended deadline.

Use a remaining-time budget.

---

## 108. Remaining Deadline

Concept:

```python
deadline = (
    time.monotonic()
    + 60
)

remaining = (
    deadline
    - time.monotonic()
)
```

Pass `remaining` to downstream operations where appropriate.

---

## 109. Retry + Deadline

```python
deadline = (
    time.monotonic()
    + 60
)

for attempt in range(3):
    remaining = (
        deadline
        - time.monotonic()
    )

    if remaining <= 0:
        raise TimeoutError()

    try:
        operation(
            timeout=remaining
        )
        break
    except RetryableError:
        ...
```

This is more robust than independent fixed timeouts.

---

## 110. Timeouts in Kubernetes Polling

```python
deadline = (
    time.monotonic()
    + 300
)

while True:
    status = get_status()

    if status == "READY":
        break

    if time.monotonic() >= deadline:
        raise TimeoutError(
            "Deployment timeout"
        )

    time.sleep(5)
```

This pattern is directly useful for deployment automation.

---

## 111. Retryable vs Non-Retryable Errors

Retry:

```text
temporary network failure
HTTP 429
some HTTP 5xx
temporary unavailable dependency
```

Usually do not retry:

```text
invalid credentials
invalid configuration
HTTP 400 caused by bad input
authorization failure
schema validation failure
```

Always follow the specific API's retry guidance.

---

## 112. HTTP 429

`429 Too Many Requests` often means:

```text
rate limited
```

Use:

```text
Retry-After
backoff
jitter
```

when supported.

Do not hammer the API repeatedly.

---

## 113. Time and Rate Limits

A script that loops too quickly can cause:

```text
API throttling
CI failures
AWS throttling
Kubernetes API pressure
```

Use:

```text
bounded polling
backoff
batching
pagination
caching
```

---

## 114. Time-Based Concurrency

When multiple workers run:

```text
worker A
worker B
worker C
```

all at once, time-based retries can synchronize.

Jitter helps distribute load.

This is especially important at scale.

---

## 115. Scheduled Job Timezones

If a job must run at:

```text
09:00 Asia/Kolkata
```

do not assume:

```text
09:00 UTC
```

Use the scheduler's timezone support or convert explicitly.

Document the intended timezone.

---

## 116. Cron Timezone Pitfall

A cron entry such as:

```text
0 9 * * *
```

means different real-world times depending on the host timezone.

For fleet consistency:

```text
standardize server timezone
```

or:

```text
configure scheduler timezone explicitly
```

---

## 117. CI/CD Timezones

CI runners may run in UTC even when engineers are in another timezone.

Therefore:

```text
pipeline logs
deployment timestamps
artifact timestamps
```

should be interpreted using explicit timezone information.

---

## 118. Human Display vs Machine Storage

Recommended:

```text
Database/API/log:
UTC ISO 8601

UI/report:
user's desired timezone
```

Example:

```text
stored:
2026-08-16T10:30:00Z

displayed in India:
2026-08-16 16:00:00 IST
```

---

## 119. Convert Only at the Edge

Architecture:

```text
service
  |
  v
UTC
  |
  v
database
  |
  v
API
  |
  v
UI
  |
  v
local timezone
```

This minimizes timezone ambiguity inside the system.

---

## 120. Timezone Database

`zoneinfo` uses timezone data from the operating system or available timezone database.

Production Linux images should have appropriate timezone data when named timezone conversion is required.

---

## 121. Docker Minimal Images and Timezone Data

Very small container images may not contain full timezone database files.

If your application uses:

```python
ZoneInfo("America/New_York")
```

ensure the runtime image contains the required timezone data.

Otherwise timezone loading may fail.

---

## 122. Timezone Testing

Test:

```text
UTC
Asia/Kolkata
DST-observing zones
DST transitions
month/year boundaries
leap years
midnight boundaries
```

Time bugs often appear only at boundaries.

---

## 123. Leap Years

Python handles calendar arithmetic correctly for valid dates.

Example:

```python
from datetime import date

date(
    2028,
    2,
    29,
)
```

Do not implement leap-year logic manually unless there is a specific reason.

---

## 124. Month-End Bugs

This is risky:

```python
date(
    2026,
    2,
    31,
)
```

It is invalid.

Use calendar-aware logic for:

```text
month end
billing cycle
renewal dates
```

---

## 125. Year-End Bugs

Always test:

```text
Dec 31 -> Jan 1
```

especially for:

```text
log rotation
billing
reports
retention
certificate renewals
```

---

## 126. Date Validation

```python
from datetime import datetime

try:
    value = datetime.strptime(
        text,
        "%Y-%m-%d",
    )
except ValueError:
    raise ValueError(
        "Invalid date"
    )
```

Do not accept ambiguous date formats in infrastructure configuration.

---

## 127. Incident Duration Across Midnight

A naive script may incorrectly assume the end is on the same day.

Use full timestamps:

```text
2026-08-16T23:50:00Z
2026-08-17T00:20:00Z
```

Then:

```python
duration = end - start
```

works correctly.

---

## 128. Time Comparisons

Use:

```python
if start <= now <= end:
    ...
```

for an inclusive window.

For half-open intervals:

```text
[start, end)
```

use:

```python
if start <= now < end:
    ...
```

Choose one convention and document it.

---

## 129. Why Half-Open Intervals Matter

For adjacent windows:

```text
10:00 - 11:00
11:00 - 12:00
```

half-open intervals avoid double-counting:

```text
[10:00, 11:00)
[11:00, 12:00)
```

This is useful in metrics and reporting.

---

## 130. Monitoring Window

Example:

```python
window_start = now - timedelta(
    minutes=15
)

events_in_window = [
    event
    for event in events
    if window_start
    <= event.timestamp
    < now
]
```

This avoids including an event exactly at the upper boundary twice across adjacent windows.

---

## 131. Rolling Windows

A monitoring script may repeatedly evaluate:

```text
last 5 minutes
last 15 minutes
last 1 hour
```

Use:

```python
cutoff = now - timedelta(
    minutes=15
)
```

and a clear interval convention.

---

## 132. Time Bucketing

For reports, events may be grouped by:

```text
minute
hour
day
```

For example:

```text
10:00
10:01
10:02
```

The bucketing rule must specify timezone and boundary behavior.

---

## 133. Incident Timeline Normalization

Before analyzing logs:

```text
source A -> UTC
source B -> UTC
source C -> UTC
```

Then sort:

```python
events.sort(
    key=lambda event:
        event["timestamp"]
)
```

This creates a consistent timeline.

---

## 134. Sort Timestamps

If timestamps are all normalized ISO 8601 UTC strings with consistent formatting, lexical ordering can work.

But parsing into aware datetime objects is safer when formats may vary.

```python
events.sort(
    key=lambda e:
        datetime.fromisoformat(
            e["timestamp"]
        )
)
```

---

## 135. Monitoring Data and Clock Skew

If:

```text
event B appears before event A
```

do not immediately assume the system violated causality.

Check:

```text
clock synchronization
network delay
buffering
log ingestion delay
timestamp source
```

---

## 136. Event Time vs Ingestion Time

Observability systems may have:

```text
event_time
ingestion_time
```

They are different.

Example:

```text
application event -> 10:00
collector receives -> 10:02
```

Use the appropriate timestamp for the question being answered.

---

## 137. Log Rotation

Log rotation may depend on:

```text
time
size
both
```

Python automation should not reinvent a mature OS/container logging system unless necessary.

When implementing cleanup, understand:

```text
open file handles
rotation mechanism
retention
compression
```

---

## 138. Backup Retention

A backup policy may say:

```text
daily backups retained 30 days
```

Python can calculate the cutoff:

```python
cutoff = (
    datetime.now(timezone.utc)
    - timedelta(days=30)
)
```

But deletion should be based on the backup system's authoritative metadata.

---

## 139. Time and Cost Reports

A cost report may need:

```text
billing period
resource creation time
resource deletion time
usage duration
```

Calculate duration carefully across:

```text
timezone
month boundaries
partial hours
billing rules
```

Never assume billing duration equals simple wall-clock duration unless the billing model says so.

---

## 140. Resource Lifetime

For a cloud resource:

```python
lifetime = (
    deleted_at
    - created_at
)
```

Useful for:

```text
temporary environments
ephemeral test clusters
preview deployments
```

---

## 141. Ephemeral Environment Cleanup

Flow:

```text
environment created
       |
       v
created_at
       |
       v
TTL reached
       |
       v
verify environment
       |
       v
cleanup
```

Add safety checks:

```text
environment tag
owner
account
region
name
```

before deleting anything.

---

## 142. Time-Based Resource Safety

Never delete solely because:

```text
resource is old
```

Also check:

```text
environment
owner
protected tag
production status
active deployment
backup state
```

Time is only one input to a safe cleanup decision.

---

## 143. Production Clock Safety

Before destructive automation:

```text
verify environment
verify identity
verify resource
verify age
verify exclusions
```

Then:

```text
dry run
```

before deletion.

---

## 144. Time in Distributed Locks

A distributed lock should consider:

```text
acquisition
lease duration
renewal
expiration
clock assumptions
```

For local elapsed timers use monotonic clocks, but distributed systems should follow the lock service's authoritative semantics.

---

## 145. Time and Kubernetes Rollouts

A deployment may have:

```text
started
progressing
available
completed
failed
```

Use timestamps and status conditions together.

Do not declare success simply because a fixed amount of time has passed.

---

## 146. Fixed Sleep vs Condition Polling

Bad:

```python
time.sleep(60)

print("deployment successful")
```

Better:

```text
poll deployment condition
      |
      +--> success -> return
      |
      +--> failure -> stop
      |
      +--> timeout -> fail
```

Time should define the maximum wait, not the success condition.

---

## 147. Production Polling Pattern

```python
deadline = (
    time.monotonic()
    + 600
)

while time.monotonic() < deadline:
    status = get_rollout_status()

    if status == "SUCCESS":
        return

    if status == "FAILED":
        raise RuntimeError(
            "Rollout failed"
        )

    time.sleep(5)

raise TimeoutError(
    "Rollout timed out"
)
```

This is a strong real-world DevOps pattern.

---

## 148. Retryable Polling Errors

During polling, a temporary API failure may occur.

Use a bounded retry policy:

```text
API error
 |
 +--> retryable -> backoff
 |
 +--> permanent -> fail
```

Do not treat every exception as a reason to continue polling indefinitely.

---

## 149. Time-Based Circuit Breaking

A service may be unavailable for:

```text
cooldown period
```

A circuit breaker can use:

```python
opened_at = time.monotonic()
```

Then:

```python
if time.monotonic() - opened_at >= cooldown:
    try_again()
```

This is elapsed-time logic, so monotonic time is appropriate.

---

## 150. Monotonic Clock and System Clock

Important distinction:

```text
datetime.now(timezone.utc)
    -> wall clock
    -> timestamps
    -> expiration dates

time.monotonic()
    -> elapsed clock
    -> timeout
    -> duration
    -> polling
```

Memorize this for DevOps interviews.

---

## 151. `sleep()` and `monotonic()`

A robust loop:

```python
deadline = (
    time.monotonic()
    + 300
)

while True:
    if check_ready():
        break

    remaining = (
        deadline
        - time.monotonic()
    )

    if remaining <= 0:
        raise TimeoutError()

    time.sleep(
        min(5, remaining)
    )
```

This avoids oversleeping beyond the deadline.

---

## 152. Timeouts Should Be Configurable

Bad:

```python
time.sleep(300)
```

Better:

```yaml
rollout_timeout_seconds: 300
poll_interval_seconds: 5
```

Then:

```python
timeout = config[
    "rollout_timeout_seconds"
]
```

Validate that:

```text
timeout > 0
poll interval > 0
poll interval < timeout
```

---

## 153. Time Configuration Validation

```python
if timeout <= 0:
    raise ValueError(
        "timeout must be positive"
    )

if poll_interval <= 0:
    raise ValueError(
        "poll interval must be positive"
    )
```

Avoid invalid production settings.

---

## 154. Alert Cooldown

Monitoring systems may suppress repeated alerts for:

```text
5 minutes
15 minutes
1 hour
```

A simple automation can track:

```python
last_alert = ...

if now - last_alert >= cooldown:
    send_alert()
```

For distributed alerting, use the monitoring system's native deduplication/cooldown features where possible.

---

## 155. Avoid Duplicate Alerts

Time can be used with an event key:

```text
service + alert type
```

Then:

```text
same alert
+
inside cooldown
=
suppress
```

This prevents alert storms.

---

## 156. Time-Based Escalation

Example:

```text
0 min -> alert engineer
10 min -> escalate
20 min -> incident manager
```

Use timestamps:

```python
elapsed = (
    now - incident_started
)
```

Then apply explicit escalation policy.

---

## 157. Incident SLA Timer

```python
elapsed = (
    datetime.now(timezone.utc)
    - incident_started
)

if elapsed >= timedelta(
    minutes=30
):
    escalate()
```

In a real system, use the incident-management platform as the authoritative source of incident state.

---

## 158. Maintenance Reminder Logic

```python
remaining = (
    maintenance_start - now
)

if remaining <= timedelta(
    hours=1
):
    notify()
```

For recurring notifications, use a scheduler rather than a continuously running Python process when possible.

---

## 159. Time in CI/CD Approvals

A deployment approval may expire:

```text
approval granted
   |
   v
expires_at
```

Check:

```python
if datetime.now(
    timezone.utc
) >= expires_at:
    reject()
```

Never assume an approval remains valid forever.

---

## 160. Artifact Expiration

Artifacts may have:

```text
created_at
expires_at
```

Before deployment:

```python
if now >= expires_at:
    raise RuntimeError(
        "Artifact expired"
    )
```

Also verify artifact identity/version/digest.

---

## 161. Docker Image Age

Image age can be derived from registry metadata.

But:

```text
old image != automatically unsafe
```

Use:

```text
vulnerability status
support policy
approval
digest
deployment history
```

alongside age.

---

## 162. Kubernetes Pod Age

Pod age can be useful for:

```text
stuck pods
unexpectedly long-running jobs
rollout verification
```

But age alone is not a health signal.

Combine it with:

```text
phase
conditions
readiness
restarts
events
```

---

## 163. Time and Health Checks

A health check should use:

```text
timeout
retry
deadline
```

not:

```text
wait forever
```

Example:

```text
HTTP check
  |
  +--> 5s timeout
  |
  +--> retry 3 times
  |
  +--> total deadline 30s
```

---

## 164. Health Check Script

```python
deadline = (
    time.monotonic()
    + 30
)

while time.monotonic() < deadline:
    try:
        check_health()
        break
    except TemporaryError:
        time.sleep(2)
else:
    raise RuntimeError(
        "Health check failed"
    )
```

---

## 165. Time and Network Reliability

Network calls can fail because of:

```text
latency
packet loss
DNS
load balancer
service overload
rate limits
```

Use:

```text
timeouts
bounded retries
backoff
jitter
```

Do not use unlimited retries.

---

## 166. Time and Database Operations

Database automation may use:

```text
query timeout
connection timeout
transaction timeout
lock timeout
```

These should be distinct where supported.

A database operation that waits indefinitely can block CI/CD or incident automation.

---

## 167. Migration Timing

A database migration script should record:

```text
migration_start
migration_end
duration
```

Then report:

```text
success/failure
duration
migration version
```

This helps identify regressions.

---

## 168. Backup Duration

Track:

```text
backup start
backup completion
backup size
```

Then:

```python
duration = completed - started
```

A growing backup duration can indicate:

```text
data growth
storage bottleneck
network bottleneck
backup configuration change
```

---

## 169. Deployment Performance

Track:

```text
build duration
test duration
security scan duration
image push duration
deployment duration
rollout duration
```

This gives a much better view of CI/CD performance than total pipeline time alone.

---

## 170. Time-Series Data

Monitoring systems store:

```text
metric value
timestamp
labels
```

A Python script may query time ranges:

```text
start
end
step
```

Use UTC and precise interval definitions.

---

## 171. Prometheus Query Window

Conceptually:

```text
now
 |
 +---- last 15 minutes ----+
```

Python can calculate:

```python
end = datetime.now(
    timezone.utc
)

start = end - timedelta(
    minutes=15
)
```

Then pass the appropriate timestamp format to the Prometheus API.

---

## 172. ELK Log Window

A logging query may filter:

```text
@timestamp >= now-15m
```

If Python constructs the query, ensure the timestamp semantics match Elasticsearch's field and timezone behavior.

---

## 173. Monitoring Clock Consistency

If:

```text
Prometheus
Grafana
Kubernetes
application
```

have inconsistent timestamps, investigate:

```text
NTP
timezone
log parsing
ingestion delay
timestamp source
```

Do not immediately blame the monitoring dashboard.

---

## 174. Timezone Conversion for Reports

```python
utc_time = datetime.now(
    timezone.utc
)

india_time = utc_time.astimezone(
    ZoneInfo("Asia/Kolkata")
)

print(
    india_time.strftime(
        "%Y-%m-%d %H:%M:%S %Z"
    )
)
```

This is appropriate at the presentation layer.

---

## 175. Report Timestamp

Good:

```text
Generated at:
2026-08-16T10:30:00Z
```

Better than:

```text
Generated at:
10:30 AM
```

because the timezone is explicit.

---

## 176. Logging Timestamp Format

A production log should ideally include:

```text
2026-08-16T10:30:00.123Z
```

or another documented UTC format.

This makes logs easier to correlate across systems.

---

## 177. Logging With Python

Use the logging module rather than `print()` for production applications.

Structured logging can include:

```text
timestamp
level
service
message
request_id
```

The logging system should control timestamp formatting consistently.

---

## 178. Time in Shell Automation

Linux commands may output:

```bash
date
date -u
```

UTC:

```bash
date -u
```

Python automation should not assume the shell's local timezone matches the intended operational timezone.

---

## 179. Linux Epoch

Shell:

```bash
date +%s
```

returns Unix seconds.

Python:

```python
import time

epoch = int(
    time.time()
)
```

Both represent wall-clock Unix time.

---

## 180. Linux File Timestamps

Useful commands:

```bash
stat file.log
```

Python:

```python
path.stat().st_mtime
```

For automation, prefer Python's filesystem APIs instead of parsing human-formatted `ls` output.

---

## 181. Avoid Parsing `ls`

Bad:

```text
ls -l
  |
  v
regex
  |
  v
file timestamp
```

Better:

```python
Path.stat()
```

This is more reliable and avoids locale/formatting issues.

---

## 182. Time and File Cleanup

Use:

```python
Path.glob()
Path.stat()
datetime
timedelta
```

to build cleanup tools.

Before deletion, verify:

```text
absolute path
allowed directory
file type
age
exclusion rules
```

---

## 183. Safe Cleanup Guard

```python
allowed_root = Path(
    "/var/tmp/myapp"
).resolve()

candidate = path.resolve()

if allowed_root not in candidate.parents:
    raise RuntimeError(
        "Refusing to delete outside allowed root"
    )
```

This protects against accidental path traversal or misconfiguration.

---

## 184. Time-Based Cache Cleanup

```text
cache directory
   |
   v
file age
   |
   +--> younger than TTL -> keep
   |
   +--> older than TTL -> candidate
   |
   v
safety checks
   |
   v
delete
```

Use dry-run mode for new cleanup automation.

---

## 185. Production Time Checklist

```text
1. Prefer UTC internally.
2. Use timezone-aware datetimes.
3. Use ZoneInfo for named timezones.
4. Avoid manual timezone offsets.
5. Use ISO 8601 for timestamps.
6. Know seconds vs milliseconds.
7. Use monotonic() for elapsed time.
8. Use datetime for wall-clock timestamps.
9. Use deadlines for polling.
10. Bound retries.
11. Add backoff and jitter.
12. Validate external timestamps.
13. Check clock synchronization.
14. Make thresholds configurable.
15. Test boundary conditions.
16. Use authoritative platform timestamps.
17. Do not infer health from age alone.
18. Use dry runs for destructive cleanup.
19. Include timezone in reports.
20. Keep time semantics explicit in APIs.
```

---

## 186. Practical Script — Current UTC Timestamp

```python
from datetime import datetime, timezone

now = datetime.now(
    timezone.utc
)

print(
    now.isoformat()
)
```

---

## 187. Practical Script — Convert UTC to India Time

```python
from datetime import datetime, timezone
from zoneinfo import ZoneInfo

utc_time = datetime.now(
    timezone.utc
)

india_time = utc_time.astimezone(
    ZoneInfo("Asia/Kolkata")
)

print(
    india_time.strftime(
        "%Y-%m-%d %H:%M:%S %Z"
    )
)
```

---

## 188. Practical Script — Expiry Check

```python
from datetime import datetime, timezone

expiry = datetime.fromisoformat(
    "2026-08-20T10:00:00+00:00"
)

now = datetime.now(
    timezone.utc
)

if now >= expiry:
    raise RuntimeError(
        "Expired"
    )

print("Still valid")
```

---

## 189. Practical Script — Expiry With Buffer

```python
from datetime import datetime, timedelta, timezone

expiry = datetime.fromisoformat(
    expiry_text
)

refresh_window = timedelta(
    minutes=5
)

now = datetime.now(
    timezone.utc
)

if expiry - now <= refresh_window:
    refresh_credential()
```

---

## 190. Practical Script — Measure Operation Duration

```python
import time

start = time.monotonic()

run_operation()

elapsed = (
    time.monotonic()
    - start
)

print(
    f"Duration: {elapsed:.2f}s"
)
```

Use `monotonic()` because this measures elapsed time rather than representing a timestamp.

---

## 191. Practical Script — Poll With Timeout

```python
import time

deadline = (
    time.monotonic()
    + 300
)

while time.monotonic() < deadline:
    status = get_status()

    if status == "READY":
        print("Ready")
        break

    if status == "FAILED":
        raise RuntimeError(
            "Operation failed"
        )

    time.sleep(5)
else:
    raise TimeoutError(
        "Operation timed out"
    )
```

---

## 192. Practical Script — Retry With Backoff

```python
import random
import time

max_attempts = 5

for attempt in range(
    max_attempts
):
    try:
        operation()
        break

    except TemporaryError:
        if attempt == max_attempts - 1:
            raise

        delay = min(
            2 ** attempt,
            60,
        )

        delay += random.uniform(
            0,
            1,
        )

        time.sleep(delay)
```

Retry only exceptions that are actually transient.

---

## 193. Practical Script — Stale Backup Check

```python
from datetime import datetime, timedelta, timezone

last_backup = datetime.fromisoformat(
    backup_timestamp
)

age = (
    datetime.now(timezone.utc)
    - last_backup
)

threshold = timedelta(
    hours=24
)

if age > threshold:
    raise RuntimeError(
        "Backup is stale"
    )
```

Prefer authoritative backup metadata over filesystem timestamps when available.

---

## 194. Practical Script — Deployment Duration

```python
from datetime import datetime, timezone

started = datetime.now(
    timezone.utc
)

deploy()

completed = datetime.now(
    timezone.utc
)

duration = (
    completed - started
)

print(
    f"Deployment took "
    f"{duration.total_seconds():.2f}s"
)
```

---

## 195. Practical Script — Incident Duration

```python
started = datetime.fromisoformat(
    incident["started_at"]
)

recovered = datetime.fromisoformat(
    incident["recovered_at"]
)

duration = recovered - started

print(
    f"Incident duration: {duration}"
)
```

Normalize timestamps to UTC before comparing them.

---

## 196. Practical Script — Maintenance Window

```python
from datetime import datetime, timezone

start = datetime.fromisoformat(
    config["start"]
)

end = datetime.fromisoformat(
    config["end"]
)

if start >= end:
    raise ValueError(
        "Invalid maintenance window"
    )

now = datetime.now(
    timezone.utc
)

allowed = (
    start <= now < end
)

if allowed:
    deploy()
else:
    print(
        "Outside maintenance window"
    )
```

---

## 197. Practical Script — Kubernetes Rollout Wait

```python
import time

timeout = 600
poll_interval = 5

deadline = (
    time.monotonic()
    + timeout
)

while time.monotonic() < deadline:
    status = get_rollout_status()

    if status == "SUCCESS":
        print(
            "Rollout successful"
        )
        break

    if status == "FAILED":
        raise RuntimeError(
            "Rollout failed"
        )

    time.sleep(poll_interval)
else:
    raise TimeoutError(
        "Rollout timed out"
    )
```

The success condition must come from actual deployment state, not merely elapsed time.

---

## 198. Practical Script — Resource Age

```python
from datetime import datetime, timezone

created = datetime.fromisoformat(
    resource["created_at"]
)

age = (
    datetime.now(timezone.utc)
    - created
)

print(
    f"Resource age: {age}"
)
```

---

## 199. Practical Script — Last 15 Minutes

```python
from datetime import datetime, timedelta, timezone

now = datetime.now(
    timezone.utc
)

cutoff = now - timedelta(
    minutes=15
)

for event in events:
    timestamp = datetime.fromisoformat(
        event["timestamp"]
    )

    if cutoff <= timestamp < now:
        process(event)
```

---

## 200. Interview — Naive vs Aware Datetime

### Question

What is the difference?

### Answer

> A naive datetime has no timezone information. An aware datetime includes timezone information and can be safely compared with other aware datetimes when their timezone semantics are correct. For distributed DevOps automation, I prefer timezone-aware UTC timestamps.

---

## 201. Interview — Why UTC?

### Answer

> Servers, containers, cloud regions, and engineers can operate in different timezones. Using UTC internally gives us a consistent reference point. I convert to local time only when presenting information to humans.

---

## 202. Interview — `datetime` vs `time.monotonic()`

### Answer

> I use timezone-aware `datetime` for wall-clock timestamps such as event time and expiration time. I use `time.monotonic()` for elapsed durations, timeouts, polling deadlines, and retry logic because wall-clock time can change due to clock synchronization or manual adjustments.

---

## 203. Interview — How Do You Handle Timeouts?

### Answer

> I define an overall deadline using a monotonic clock, poll or retry until the condition succeeds or fails, and stop when the deadline is reached. I also use bounded retries, appropriate polling intervals, and backoff rather than sleeping for a fixed time and assuming success.

---

## 204. Interview — Why Is `time.sleep(60)` Not Enough for Deployment Verification?

### Answer

> A fixed sleep does not tell me whether the deployment succeeded. The deployment could fail after 10 seconds or take 90 seconds. I should poll the actual Kubernetes rollout condition and use a timeout only as the maximum waiting period.

---

## 205. Interview — How Do You Handle Timezones?

### Answer

> I store and exchange timestamps in UTC, use timezone-aware datetimes, and use `zoneinfo` for named timezone conversions. I avoid manually adding fixed offsets because they do not correctly model timezone rules such as daylight saving time.

---

## 206. Interview — Seconds vs Milliseconds

### Answer

> I check the API contract before converting timestamps. Unix seconds and Unix milliseconds differ by a factor of 1,000, and passing one where the other is expected can produce completely incorrect dates.

---

## 207. Interview — How Would You Detect a Stale Backup?

### Answer

> I obtain the authoritative last-successful-backup timestamp, normalize it to UTC, calculate its age, compare it with the configured threshold, and trigger an alert if the age exceeds the threshold. I would not rely only on a local file timestamp if the backup system provides better metadata.

---

## 208. Interview — How Do You Implement Retry Backoff?

### Answer

> I define retryable errors, maximum attempts, a bounded exponential backoff, and jitter. I also use an overall deadline where appropriate. I avoid retrying permanent errors such as invalid credentials or invalid configuration.

---

## 209. Scenario — Logs From Two Servers Have Different Times

Investigate:

```text
timezone
NTP synchronization
clock offset
timestamp source
log ingestion delay
container timezone
```

Check Linux:

```bash
timedatectl
```

Then normalize logs to UTC before comparing them.

---

## 210. Scenario — Token Appears Valid but API Rejects It

Check:

```text
expiration timestamp
local system clock
NTP synchronization
timezone conversion
token issuer
audience
refresh buffer
```

Clock skew can cause apparently valid tokens to be rejected.

---

## 211. Scenario — Deployment Script Waits Forever

Likely problem:

```text
no timeout
```

Fix:

```text
monotonic deadline
poll status
handle failed condition
bounded interval
```

Never allow a production automation job to wait indefinitely.

---

## 212. Scenario — Deployment Fails After Exactly 300 Seconds

Check:

```text
configured timeout
CI job timeout
HTTP timeout
Kubernetes rollout timeout
load balancer timeout
```

A repeated exact duration strongly suggests a configured deadline.

---

## 213. Scenario — Retry Storm During AWS Throttling

Likely cause:

```text
many workers
+
same retry interval
+
no jitter
```

Fix:

```text
bounded exponential backoff
jitter
rate limiting
AWS SDK retry configuration
```

Do not simply increase the number of retries.

---

## 214. Scenario — Cleanup Deletes New Files

Check:

```text
timezone interpretation
mtime conversion
cutoff calculation
path selection
clock synchronization
```

Use UTC-aware comparisons and test the script with `--dry-run` before deletion.

---

## 215. Scenario — Incident Timeline Is Out of Order

Check:

```text
clock skew
different timezone
event timestamp vs ingestion timestamp
buffering
timestamp parsing
```

Normalize timestamps to UTC and retain correlation IDs.

---

## 216. Scenario — CI Pipeline Suddenly Takes 20 Minutes

Break total duration into:

```text
queue
checkout
build
test
security scan
image push
deployment
rollout
```

Measure each component separately with monotonic elapsed-time measurement.

---

## 217. Scenario — Certificate Alert Fires Too Late

Possible causes:

```text
wrong timezone
seconds/milliseconds confusion
no refresh buffer
incorrect expiry parsing
clock skew
```

Fix by:

```text
UTC
aware datetime
correct epoch unit
configured renewal threshold
clock synchronization
```

---

## 218. Scenario — Cron Runs at the Wrong Local Time

Check:

```bash
timedatectl
```

Then verify:

```text
host timezone
scheduler timezone
container timezone
DST behavior
```

Prefer an explicit timezone policy.

---

## 219. Scenario — API Timeout Causes Long CI Jobs

Suppose:

```text
request timeout = 30s
retries = 5
```

The total job time may become much larger than 30 seconds.

Use:

```text
overall deadline
per-request timeout
bounded retries
backoff
```

to control total execution time.

---

## 220. Scenario — Kubernetes Polling Hits API Too Frequently

Reduce:

```text
poll frequency
```

and use:

```text
watch mechanisms
event-driven mechanisms
appropriate client libraries
backoff
```

when available.

Do not continuously poll a large cluster every second unless there is a strong operational reason.

---

## 221. Production Architecture — Deployment Automation

```text
CI/CD
  |
  v
Python deployment controller
  |
  +--> start timestamp
  |
  +--> deploy
  |
  +--> poll Kubernetes
  |       |
  |       +--> success
  |       +--> failure
  |       +--> timeout
  |
  +--> end timestamp
  |
  v
duration + result
  |
  v
report / metrics
```

This combines:

```text
datetime
monotonic
timeouts
polling
Kubernetes
CI/CD
```

---

## 222. Production Architecture — Certificate Monitoring

```text
certificate source
       |
       v
expiry timestamp
       |
       v
UTC normalization
       |
       v
remaining duration
       |
       +--> > 30 days -> normal
       |
       +--> <= 30 days -> warning
       |
       +--> <= 7 days -> urgent
       |
       +--> expired -> critical
```

Thresholds should be configurable.

---

## 223. Production Architecture — Stale Backup Monitor

```text
backup platform
      |
      v
last successful backup
      |
      v
UTC datetime
      |
      v
age calculation
      |
      v
policy threshold
      |
      +--> healthy
      +--> warning
      +--> critical
```

Use the backup platform's authoritative state.

---

## 224. Production Architecture — Incident Timeline

```text
Prometheus alert
       |
       v
incident created
       |
       v
acknowledged
       |
       v
mitigation
       |
       v
recovery
       |
       v
datetime calculations
       |
       +--> MTTA
       +--> MTTR
       +--> downtime
```

Keep all event timestamps normalized and traceable.

---

## 225. Production Architecture — GitOps Deployment Window

```text
Git change
    |
    v
CI validation
    |
    v
approved window?
    |
 +--+--+
 |     |
no    yes
 |     |
stop  merge
       |
       v
ArgoCD
       |
       v
EKS
       |
       v
rollout polling
       |
       v
deadline
```

Time controls when the action may happen; actual platform state determines whether it succeeded.

---

## 226. Final DevOps Cheat Sheet

### Wall-clock timestamp

```python
datetime.now(timezone.utc)
```

### ISO 8601

```python
timestamp.isoformat()
```

### Parse ISO

```python
datetime.fromisoformat(text)
```

### Duration

```python
end - start
```

### Duration seconds

```python
duration.total_seconds()
```

### Unix seconds

```python
timestamp.timestamp()
```

### Unix milliseconds

```python
int(timestamp.timestamp() * 1000)
```

### Named timezone

```python
ZoneInfo("Asia/Kolkata")
```

### Convert timezone

```python
timestamp.astimezone(zone)
```

### Elapsed runtime

```python
time.monotonic()
```

### Sleep

```python
time.sleep(seconds)
```

### Timeout deadline

```python
deadline = time.monotonic() + timeout
```

---

## 227. Final Mental Model

```text
                         TIME
                          |
          +---------------+---------------+
          |                               |
     WALL CLOCK                      ELAPSED TIME
          |                               |
     datetime                          monotonic
          |                               |
      UTC / TZ                       timeout / retry
          |                               |
    timestamps                         polling
    expiration                         duration
    incidents                          deadlines
    reports                            backoff
          |
          v
     ISO 8601 / epoch
          |
          v
AWS / Kubernetes / CI/CD / Logs
```

Remember:

```text
UTC + timezone-aware datetime
        =
reliable distributed timestamps

monotonic()
        =
reliable elapsed-time measurement
```

That distinction is one of the most important Python concepts for a DevOps engineer.

---