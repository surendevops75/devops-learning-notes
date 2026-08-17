# 04-Regex

## 1. Overview

Regular expressions, commonly called **regex**, are extremely useful for DevOps automation.

DevOps engineers frequently process unstructured text such as:

- application logs
- Kubernetes logs
- CI/CD logs
- Terraform output
- Linux command output
- deployment reports
- security scan reports
- configuration files
- filenames
- version strings
- URLs
- IP addresses
- timestamps
- request IDs
- error messages

Regex provides a way to:

```text
search
match
extract
validate
replace
transform
```

text based on patterns.

A practical DevOps workflow is:

```text
Raw text
   |
   v
Regex pattern
   |
   +--> find matches
   +--> extract fields
   +--> validate input
   +--> replace content
   |
   v
Structured data
   |
   v
Automation / report / decision
```

Important principle:

> Regex is a powerful text-processing tool, but it should not be used as a replacement for a proper parser when the input is already a structured format such as JSON, YAML, XML, or a Kubernetes API response.

---

# 2. Importing `re`

Python provides regex through the standard library:

```python
import re
```

No external package is required.

---

# 3. Simple Search

```python
import re

text = "Deployment failed"

match = re.search(
    r"failed",
    text,
)

if match:
    print("Found")
```

`re.search()` looks anywhere in the string.

---

# 4. Raw Strings

Prefer:

```python
r"\d+"
```

instead of:

```python
"\\d+"
```

Raw strings make regex patterns easier to read because Python does not interpret most backslashes first.

---

# 5. Literal Text

Pattern:

```python
r"ERROR"
```

matches:

```text
ERROR
```

Example:

```python
if re.search(r"ERROR", line):
    print("Error found")
```

---

# 6. Case Sensitivity

By default:

```python
re.search(
    r"error",
    "ERROR",
)
```

does not match.

Use:

```python
re.search(
    r"error",
    "ERROR",
    re.IGNORECASE,
)
```

---

# 7. Dot `.`

The dot matches almost any single character.

```python
r"a.c"
```

can match:

```text
abc
a1c
a-c
```

It does not normally match a newline.

---

# 8. Escaping the Dot

If you need a literal period:

```python
r"\."
```

Example:

```python
re.search(
    r"\.log$",
    "application.log",
)
```

---

# 9. Character Classes

Square brackets define a character class:

```python
r"[abc]"
```

matches one character that is:

```text
a
b
c
```

---

# 10. Range

```python
r"[a-z]"
```

matches a lowercase letter.

```python
r"[A-Z]"
```

matches an uppercase letter.

```python
r"[0-9]"
```

matches a digit.

---

# 11. Multiple Ranges

```python
r"[A-Za-z0-9]"
```

matches one alphanumeric character.

Useful for:

```text
IDs
versions
names
tokens
```

but do not use broad patterns for sensitive values without considering security requirements.

---

# 12. Negated Character Class

```python
r"[^0-9]"
```

means:

```text
any character except a digit
```

The `^` has different meanings depending on its location.

At the beginning of a character class:

```text
[^...]
```

means negation.

---

# 13. `\d`

`\d` matches a digit.

```python
r"\d"
```

Example:

```text
1
5
9
```

For ASCII-only expectations, be aware that Python regex Unicode behavior can make `\d` broader than `[0-9]`.

---

# 14. `\D`

`\D` means:

```text
not a digit
```

Example:

```python
r"\D+"
```

matches one or more non-digit characters.

---

# 15. `\w`

`\w` matches word characters.

In Python's Unicode-aware default behavior, this is broader than only:

```text
A-Z
a-z
0-9
_
```

For strict ASCII identifiers, use an explicit character class when necessary.

---

# 16. `\W`

`\W` means:

```text
not a word character
```

---

# 17. `\s`

`\s` matches whitespace such as:

```text
space
tab
newline
```

Example:

```python
r"\s+"
```

matches one or more whitespace characters.

---

# 18. `\S`

`\S` means:

```text
non-whitespace
```

---

# 19. Quantifier `*`

```python
r"a*"
```

means:

```text
zero or more a characters
```

Matches:

```text
""
"a"
"aa"
"aaa"
```

---

# 20. Quantifier `+`

```python
r"a+"
```

means:

```text
one or more
```

Matches:

```text
a
aa
aaa
```

but not an empty string.

---

# 21. Quantifier `?`

```python
r"a?"
```

means:

```text
zero or one
```

Useful for optional characters.

Example:

```python
r"https?"
```

matches:

```text
http
https
```

---

# 22. Exact Repetition

```python
r"\d{4}"
```

means exactly four digits.

Matches:

```text
2026
8080
1234
```

---

# 23. Minimum Repetition

```python
r"\d{2,}"
```

means:

```text
at least two digits
```

---

# 24. Range Repetition

```python
r"\d{2,4}"
```

means:

```text
between 2 and 4 digits
```

---

# 25. Start Anchor `^`

```python
r"^ERROR"
```

means:

```text
ERROR must occur at the beginning
```

Example:

```text
ERROR database unavailable
```

matches.

---

# 26. End Anchor `$`

```python
r"ERROR$"
```

means:

```text
ERROR must occur at the end
```

Useful for file extensions:

```python
r"\.log$"
```

---

# 27. Full String Validation

For validating an entire string, prefer:

```python
re.fullmatch(
    pattern,
    value,
)
```

instead of relying only on:

```text
^
$
```

Example:

```python
if re.fullmatch(
    r"[A-Za-z0-9_-]+",
    service_name,
):
    print("Valid")
```

---

# 28. `re.match()`

`re.match()` attempts a match at the beginning of the string.

```python
match = re.match(
    r"ERROR",
    "ERROR database down",
)
```

It succeeds.

But:

```python
re.match(
    r"ERROR",
    "2026 ERROR",
)
```

does not.

---

# 29. `re.search()`

`re.search()` searches anywhere:

```python
match = re.search(
    r"ERROR",
    "2026 ERROR database down",
)
```

This succeeds.

---

# 30. `match()` vs `search()`

```text
match()
   |
   v
beginning of string

search()
   |
   v
anywhere in string
```

For log analysis, `search()` is often the more useful operation.

---

# 31. `re.fullmatch()`

`fullmatch()` requires the entire string to match.

```python
re.fullmatch(
    r"\d{4}",
    "2026",
)
```

matches.

```python
re.fullmatch(
    r"\d{4}",
    "year=2026",
)
```

does not.

---

# 32. Match Object

```python
match = re.search(
    r"ERROR",
    "ERROR database unavailable",
)

if match:
    print(match.group())
```

Output:

```text
ERROR
```

---

# 33. Match Position

```python
print(match.start())
print(match.end())
```

This gives the character positions of the match.

---

# 34. Capturing Groups

Use parentheses:

```python
r"ERROR (.*)"
```

Example:

```python
text = "ERROR database unavailable"

match = re.search(
    r"ERROR (.*)",
    text,
)

if match:
    print(match.group(1))
```

Output:

```text
database unavailable
```

---

# 35. Multiple Groups

```python
pattern = (
    r"service=(\w+) "
    r"status=(\d+)"
)

text = (
    "service=payment status=500"
)

match = re.search(
    pattern,
    text,
)

if match:
    print(match.group(1))
    print(match.group(2))
```

Output:

```text
payment
500
```

---

# 36. Named Groups

Named groups are excellent for DevOps log parsing.

```python
pattern = (
    r"service=(?P<service>\w+) "
    r"status=(?P<status>\d+)"
)

match = re.search(
    pattern,
    text,
)

if match:
    print(
        match.group("service")
    )
    print(
        match.group("status")
    )
```

---

# 37. `groupdict()`

Named groups can become a dictionary:

```python
data = match.groupdict()

print(data)
```

Possible output:

```python
{
    "service": "payment",
    "status": "500",
}
```

This is very useful when converting unstructured logs into structured data.

---

# 38. Non-Capturing Groups

Use:

```python
(?:...)
```

when you need grouping but do not need the group's value.

Example:

```python
r"(?:http|https)://"
```

This keeps group numbering cleaner.

---

# 39. Alternation `|`

```python
r"ERROR|WARN"
```

matches either:

```text
ERROR
```

or:

```text
WARN
```

---

# 40. Grouped Alternation

```python
r"(ERROR|WARN|CRITICAL)"
```

Use:

```python
match.group(1)
```

to retrieve the matching severity.

---

# 41. Regex Flags

Common flags:

```text
re.IGNORECASE
re.MULTILINE
re.DOTALL
re.VERBOSE
```

Example:

```python
re.search(
    r"error",
    text,
    re.IGNORECASE,
)
```

---

# 42. `re.MULTILINE`

Normally:

```text
^
$
```

refer to the beginning/end of the entire string.

With:

```python
re.MULTILINE
```

they can match the beginning/end of individual lines.

Useful for multi-line configuration or log content.

---

# 43. `re.DOTALL`

Normally:

```text
.
```

does not match newline.

With:

```python
re.DOTALL
```

it can match newline characters too.

Use carefully because broad multi-line patterns can become expensive and hard to reason about.

---

# 44. `re.VERBOSE`

For complex patterns:

```python
pattern = re.compile(
    r"""
    service=
    (?P<service>\w+)
    \s+
    status=
    (?P<status>\d+)
    """,
    re.VERBOSE,
)
```

This improves readability.

---

# 45. Compiling Regex

For patterns used repeatedly:

```python
pattern = re.compile(
    r"ERROR",
)
```

Then:

```python
pattern.search(line)
```

This makes the code clearer and avoids repeatedly expressing the same pattern.

---

# 46. `findall()`

Find all matches:

```python
matches = re.findall(
    r"\d+",
    "ports 8080 9090 3000",
)

print(matches)
```

Output:

```text
['8080', '9090', '3000']
```

---

# 47. `finditer()`

For large text or when match positions/groups are needed:

```python
for match in re.finditer(
    r"\d+",
    text,
):
    print(match.group())
```

`finditer()` returns match objects lazily.

This is often preferable for large logs.

---

# 48. `findall()` vs `finditer()`

```text
findall()
   |
   v
returns collected matches

finditer()
   |
   v
returns match objects lazily
```

For large files:

```text
finditer()
```

can be a better fit.

---

# 49. Substitution

Use:

```python
re.sub(
    pattern,
    replacement,
    text,
)
```

Example:

```python
result = re.sub(
    r"ERROR",
    "FAILURE",
    text,
)
```

---

# 50. Replace Multiple Spaces

```python
cleaned = re.sub(
    r"\s+",
    " ",
    text,
)
```

This converts repeated whitespace into a single space.

Be careful not to apply this blindly to configuration formats where whitespace can be meaningful.

---

# 51. Replace Sensitive Data

A useful DevOps operation:

```python
safe = re.sub(
    r"password=\S+",
    "password=***",
    log_line,
)
```

Example:

```text
password=secret123
```

becomes:

```text
password=***
```

For robust secret redaction, account for quoting, JSON, headers, tokens, and alternate formats rather than relying on one pattern.

---

# 52. Redacting Authorization Headers

Example:

```python
pattern = re.compile(
    r"(Authorization:\s*Bearer\s+)\S+",
    re.IGNORECASE,
)

safe = pattern.sub(
    r"\1***",
    log_line,
)
```

This is useful when sanitizing HTTP logs.

---

# 53. Regex for IP Addresses

A basic IPv4 pattern:

```python
pattern = re.compile(
    r"\b(?:\d{1,3}\.){3}\d{1,3}\b"
)
```

It can find strings resembling:

```text
10.0.0.1
192.168.1.10
172.16.0.5
```

Important:

> This pattern does not guarantee each octet is between 0 and 255.

---

# 54. Validate IPv4 Correctly

For actual IP validation, prefer Python's standard library:

```python
import ipaddress

address = ipaddress.ip_address(
    "192.168.1.10"
)

print(address)
```

Use regex to locate candidates and `ipaddress` to validate them when correctness matters.

---

# 55. Extract IPv4 Candidates

```python
import re
import ipaddress

pattern = re.compile(
    r"\b(?:\d{1,3}\.){3}\d{1,3}\b"
)

for candidate in pattern.findall(text):
    try:
        address = ipaddress.ip_address(
            candidate
        )
        print(address)
    except ValueError:
        pass
```

This is a strong real-world pattern:

```text
regex -> candidate
validator -> correctness
```

---

# 56. Ports

A basic port candidate:

```python
r"\b\d{1,5}\b"
```

But this also matches:

```text
timestamps
IDs
HTTP status codes
versions
```

Better:

```text
parse context
then validate numeric range 0-65535
```

---

# 57. Port Validation

```python
port = int(value)

if not 0 <= port <= 65535:
    raise ValueError(
        "Invalid port"
    )
```

Regex should not be responsible for every semantic rule.

---

# 58. URLs

Basic URL matching:

```python
pattern = re.compile(
    r"https?://[^\s]+"
)
```

Useful for extracting URLs from logs.

For strict URL validation, use a URL parser and explicit policy rather than an enormous regex.

---

# 59. Kubernetes Pod Names

Kubernetes resource naming follows DNS-style constraints.

A simplified candidate pattern:

```python
r"[a-z0-9]([-a-z0-9]*[a-z0-9])?"
```

Do not assume a simplified regex fully captures every Kubernetes naming rule or resource-specific limit.

For actual validation, use Kubernetes APIs/schema validation where possible.

---

# 60. Kubernetes Namespace

Example:

```python
pattern = re.compile(
    r"^[a-z0-9]([-a-z0-9]*[a-z0-9])?$"
)
```

Use `re.fullmatch()`:

```python
if re.fullmatch(
    pattern,
    namespace,
):
    print("Candidate valid")
```

Then apply the appropriate length and Kubernetes-specific rules.

---

# 61. Docker Image Tags

A common extraction pattern:

```python
r"[\w.-]+:[\w.-]+"
```

may identify:

```text
payment:v42
frontend:1.2.3
```

But Docker image references have more syntax.

For robust parsing, use a dedicated parser or clearly define the subset your automation accepts.

---

# 62. Semantic Version

Basic candidate:

```python
r"\d+\.\d+\.\d+"
```

Matches:

```text
1.2.3
10.20.30
```

But full Semantic Versioning has more rules.

For strict version validation, use a proper version parser.

---

# 63. Extract Image Version From Logs

```python
pattern = re.compile(
    r"image=(?P<image>[\w./-]+):(?P<tag>[\w.-]+)"
)

match = pattern.search(
    "image=payment:v42"
)

if match:
    print(
        match.groupdict()
    )
```

Output:

```python
{
    "image": "payment",
    "tag": "v42",
}
```

---

# 64. Timestamps

Common timestamp:

```text
2026-08-16 14:30:45
```

Pattern:

```python
r"\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2}"
```

Regex can extract it.

For actual date/time validation and conversion, use `datetime`.

---

# 65. Timestamp Extraction + Parsing

```python
from datetime import datetime
import re

pattern = re.compile(
    r"\d{4}-\d{2}-\d{2} "
    r"\d{2}:\d{2}:\d{2}"
)

match = pattern.search(text)

if match:
    timestamp = datetime.strptime(
        match.group(),
        "%Y-%m-%d %H:%M:%S",
    )
```

This is better than making regex validate calendar correctness.

---

# 66. Log Level Extraction

Given:

```text
2026-08-16 INFO payment started
2026-08-16 ERROR database failed
```

Pattern:

```python
pattern = re.compile(
    r"\b(?P<level>DEBUG|INFO|WARN|WARNING|ERROR|CRITICAL)\b"
)
```

Then:

```python
match = pattern.search(line)

if match:
    print(
        match.group("level")
    )
```

---

# 67. HTTP Status Extraction

Example log:

```text
GET /api/orders 500
```

Pattern:

```python
r"\b(?P<status>[1-5]\d{2})\b"
```

This finds:

```text
500
```

but context is important because other three-digit numbers may exist.

---

# 68. HTTP Error Detection

```python
pattern = re.compile(
    r"\b(?P<status>[45]\d{2})\b"
)

match = pattern.search(line)

if match:
    status = int(
        match.group("status")
    )
```

Use structured log fields when available instead of regex.

---

# 69. Request ID Extraction

Example:

```text
request_id=abc-123-def
```

Pattern:

```python
r"request_id=(?P<request_id>[\w-]+)"
```

Then:

```python
match.group("request_id")
```

---

# 70. UUID Extraction

Basic UUID candidate:

```python
pattern = re.compile(
    r"\b[0-9a-fA-F]{8}-"
    r"[0-9a-fA-F]{4}-"
    r"[0-9a-fA-F]{4}-"
    r"[0-9a-fA-F]{4}-"
    r"[0-9a-fA-F]{12}\b"
)
```

For strict validation, use the `uuid` module after extraction.

---

# 71. AWS Account ID

AWS account IDs are commonly 12 digits:

```python
r"\b\d{12}\b"
```

This is a candidate pattern.

If processing sensitive logs, remember that account IDs can be security-relevant information and should be handled according to your organization's policy.

---

# 72. AWS ARN Candidate

A broad candidate:

```python
r"arn:[^\s]+"
```

Useful for extraction.

For strict ARN parsing, a dedicated parser or carefully defined components are preferable.

---

# 73. Terraform Resource References

A pattern might locate references such as:

```text
aws_instance.web
aws_vpc.main
aws_iam_role.app
```

Example:

```python
r"\baws_[a-z0-9_]+\.[a-z0-9_]+\b"
```

This is useful for quick extraction from text.

Do not use regex as a Terraform parser.

---

# 74. Terraform Error Extraction

Given:

```text
Error: creating EC2 instance
```

Pattern:

```python
r"^Error:\s*(?P<message>.+)$"
```

with:

```python
re.MULTILINE
```

can extract error lines from a multi-line output.

---

# 75. CI/CD Error Detection

Potential patterns:

```text
ERROR
Error:
FAILED
FAILURE
exit code
Job failed
```

Example:

```python
pattern = re.compile(
    r"\b(?:ERROR|FAILED|FAILURE)\b",
    re.IGNORECASE,
)
```

Avoid declaring a build failed solely because a word appears in a historical or expected message. Prefer the actual command exit status and structured CI result.

---

# 76. Docker Build Error Extraction

A script can identify candidate error lines:

```python
pattern = re.compile(
    r"^(?:ERROR|error|failed).*",
    re.MULTILINE,
)
```

But the Docker command's exit code remains the authoritative build result.

---

# 77. Kubernetes Log Filtering

```python
pattern = re.compile(
    r"\bERROR\b",
    re.IGNORECASE,
)

with open(
    "pod.log",
    encoding="utf-8",
) as file:
    for line in file:
        if pattern.search(line):
            print(line.rstrip())
```

For large logs, stream line by line.

---

# 78. Extract Service + Status + Latency

Suppose:

```text
service=payment status=500 latency=842ms
```

Pattern:

```python
pattern = re.compile(
    r"""
    service=(?P<service>\S+)
    \s+
    status=(?P<status>\d+)
    \s+
    latency=(?P<latency>\d+)ms
    """,
    re.VERBOSE,
)
```

Then:

```python
data = pattern.search(
    line
).groupdict()
```

Result:

```python
{
    "service": "payment",
    "status": "500",
    "latency": "842",
}
```

---

# 79. Convert Extracted Values

Regex returns strings.

```python
data["status"] = int(
    data["status"]
)

data["latency"] = int(
    data["latency"]
)
```

Then automation can perform numerical comparisons.

---

# 80. Regex + Structured Output

A useful transformation:

```text
raw log
   |
   v
regex extraction
   |
   v
dictionary
   |
   v
JSON
```

Example:

```python
import json

print(
    json.dumps(
        data,
        indent=2,
    )
)
```

This bridges unstructured logs and automation.

---

# 81. Parse Apache/Nginx-Like Logs

A simplified access log may contain:

```text
10.0.0.5 - - [16/Aug/2026:10:30:00 +0000] "GET /api HTTP/1.1" 200 1234
```

A regex can extract:

```text
IP
timestamp
method
path
protocol
status
bytes
```

For production, prefer the logging format's structured fields or a dedicated parser if available.

---

# 82. Named Group Access Log Pattern

Conceptually:

```python
pattern = re.compile(
    r"""
    (?P<ip>\S+)
    \s+\S+\s+\S+
    \s+\[(?P<timestamp>[^\]]+)\]
    \s+"(?P<method>\S+)
    \s+(?P<path>\S+)
    \s+(?P<protocol>[^"]+)"
    \s+(?P<status>\d{3})
    \s+(?P<bytes>\d+)
    """,
    re.VERBOSE,
)
```

Named groups make the parser easier to maintain.

---

# 83. Why Regex Can Fail on Logs

Logs may contain:

```text
quoted strings
escaped quotes
multiline messages
JSON
stack traces
variable formats
timestamps
nested fields
```

A regex written for one exact format can break when the application changes its logging format.

Prefer structured logging whenever possible.

---

# 84. Regex for JSON Is a Bad Idea

Bad:

```python
re.search(
    r'"status":\s*(\d+)',
    json_text,
)
```

This may work for a quick extraction but is fragile.

Better:

```python
import json

data = json.loads(
    json_text
)

status = data["status"]
```

Use regex to locate text boundaries only when necessary.

---

# 85. Regex for YAML Is Usually a Bad Idea

Do not parse Kubernetes YAML with:

```python
re.sub(...)
```

Instead:

```python
import yaml

data = yaml.safe_load(...)
```

Then modify:

```python
data["spec"]["replicas"] = 4
```

Regex is for text patterns, not structured document parsing.

---

# 86. Regex for Terraform Is Usually a Bad Idea

Do not build a full Terraform parser with regex.

Use:

```text
terraform commands
Terraform JSON output
HCL tooling
```

Regex can still be useful for extracting small pieces from human-readable command output.

---

# 87. Regex for Kubernetes API Data

Prefer:

```text
Kubernetes API
       |
       v
structured object
```

over:

```text
kubectl text
       |
       v
regex
```

If you need a quick shell-style script, text parsing can be acceptable, but production automation should prefer structured APIs or JSON output.

---

# 88. Kubernetes JSON Output

Instead of:

```bash
kubectl get pods
```

use:

```bash
kubectl get pods -o json
```

Then parse JSON.

This avoids fragile whitespace-based regex parsing.

---

# 89. When Regex Is Appropriate

Good uses:

```text
extract request IDs
find error lines
extract timestamps
redact secrets
validate simple identifiers
find URLs
find IP candidates
extract image tags
normalize text
```

---

# 90. When Regex Is Not Appropriate

Avoid regex as the primary parser for:

```text
JSON
YAML
XML
Terraform/HCL
Kubernetes API objects
complex programming languages
nested recursive structures
```

Use a parser.

---

# 91. Regex Performance

Simple pattern:

```python
r"ERROR"
```

is cheap.

Complex nested patterns can become expensive.

Potentially dangerous:

```python
r"(a+)+$"
```

on specially crafted input can cause excessive backtracking.

---

# 92. Catastrophic Backtracking

Some regex patterns can take extremely long on certain inputs.

This matters when processing:

```text
user input
large logs
external data
HTTP requests
CI artifacts
```

Avoid unnecessarily nested ambiguous quantifiers.

---

# 93. ReDoS

Regular Expression Denial of Service can happen when an attacker causes a vulnerable regex to consume excessive CPU.

For security-sensitive services:

```text
keep patterns simple
limit input size
avoid ambiguous nested quantifiers
test worst-case inputs
```

---

# 94. Limit Input Size

If a regex only needs to inspect:

```text
first 10 KB
```

do not feed it:

```text
2 GB
```

without reason.

Stream or limit input where possible.

---

# 95. Compile Reusable Patterns

```python
ERROR_PATTERN = re.compile(
    r"\bERROR\b"
)
```

Then:

```python
if ERROR_PATTERN.search(line):
    ...
```

This improves readability and centralizes the pattern.

---

# 96. Regex Constants

Good:

```python
LOG_LEVEL_PATTERN = re.compile(
    r"\b(DEBUG|INFO|WARN|ERROR|CRITICAL)\b"
)
```

Bad:

```python
if re.search(
    r"\b(DEBUG|INFO|WARN|ERROR|CRITICAL)\b",
    line,
):
    ...
```

repeated across many functions.

Centralize reusable patterns.

---

# 97. Regex Tests

A production regex should have tests.

Example:

```python
def test_request_id():
    text = (
        "request_id=abc-123"
    )

    match = PATTERN.search(text)

    assert match
    assert (
        match.group("request_id")
        == "abc-123"
    )
```

Test:

```text
valid input
invalid input
edge cases
unexpected formatting
empty input
large input
```

---

# 98. False Positives

Suppose you search:

```python
r"ERROR"
```

You may match:

```text
NO_ERROR
ERROR_CODE
previous ERROR
```

Use boundaries/context:

```python
r"\bERROR\b"
```

when appropriate.

---

# 99. Word Boundaries

`\b` represents a word boundary.

Example:

```python
r"\bERROR\b"
```

matches:

```text
ERROR
```

but not:

```text
ERROR_CODE
MYERROR
```

---

# 100. Word Boundary Caveat

Word boundaries depend on what Python considers a word character.

For strict protocol formats, explicit delimiters can be more reliable than assuming `\b` always matches the desired boundary.

---

# 101. Extract Key-Value Pairs

Example:

```text
env=prod region=ap-south-1 service=payment
```

Pattern:

```python
pattern = re.compile(
    r"(?P<key>\w+)=(?P<value>\S+)"
)
```

Then:

```python
for match in pattern.finditer(text):
    print(
        match.group("key"),
        match.group("value"),
    )
```

---

# 102. Build a Dictionary From Key-Value Pairs

```python
data = {
    match.group("key"):
    match.group("value")
    for match in pattern.finditer(text)
}
```

Result:

```python
{
    "env": "prod",
    "region": "ap-south-1",
    "service": "payment",
}
```

This works for simple key-value formats, not arbitrary quoted or nested syntax.

---

# 103. Extract Environment

```python
pattern = re.compile(
    r"\benv=(?P<env>[A-Za-z0-9_-]+)\b"
)

match = pattern.search(line)

if match:
    environment = match.group("env")
```

---

# 104. Extract Namespace

```python
pattern = re.compile(
    r"\bnamespace=(?P<namespace>[a-z0-9-]+)\b"
)
```

Then validate against Kubernetes naming rules if required.

---

# 105. Extract Pod Name

```python
pattern = re.compile(
    r"\bpod=(?P<pod>[a-z0-9.-]+)\b"
)
```

Again, this is an extraction pattern, not a complete Kubernetes validator.

---

# 106. Extract Container Name

```python
pattern = re.compile(
    r"\bcontainer=(?P<container>[a-z0-9-]+)\b"
)
```

Useful in log processing.

---

# 107. Extract Deployment Version

Example:

```text
deployment=payment image=payment:v42
```

Pattern:

```python
pattern = re.compile(
    r"image=(?P<image>[\w./-]+):(?P<tag>[\w.-]+)"
)
```

Use the extracted tag to build a deployment report.

---

# 108. Compare Version Tags

```python
current = "v41"
deployed = "v42"
```

Regex can extract them.

For version ordering, do not compare strings blindly:

```text
v10 < v9
```

lexicographically.

Use a version parser or explicit numeric comparison.

---

# 109. Regex + `datetime`

Regex:

```text
extract timestamp
```

Datetime:

```text
parse timestamp
```

This division of responsibility is important.

---

# 110. Regex + `ipaddress`

Regex:

```text
find candidate IP
```

`ipaddress`:

```text
validate IP
```

This is safer than trying to encode every IP rule into one enormous regex.

---

# 111. Regex + `urllib.parse`

Regex can locate:

```text
URL candidate
```

Then:

```python
from urllib.parse import urlparse

parsed = urlparse(url)
```

Use structured parsing for:

```text
scheme
hostname
port
path
query
```

---

# 112. Regex + `pathlib`

Regex can extract a filename pattern:

```python
r"backup-\d{8}\.tar\.gz"
```

Then `pathlib` can perform:

```text
exists
stat
move
delete
```

---

# 113. Regex + File Handling

```python
pattern = re.compile(
    r"\bERROR\b"
)

with open(
    "application.log",
    encoding="utf-8",
) as file:
    for line in file:
        if pattern.search(line):
            print(line.rstrip())
```

This is a foundational DevOps automation pattern.

---

# 114. Regex + CSV

Suppose a CSV field contains:

```text
service=payment status=500
```

Read CSV using `csv`, then use regex only on the field.

Do not parse the CSV itself with regex.

---

# 115. Regex + JSON

Read JSON with:

```python
json.load()
```

Then regex can be applied to a specific string field if needed.

Example:

```text
JSON
 |
 v
structured object
 |
 v
specific log/message field
 |
 v
regex
```

---

# 116. Regex + YAML

Read YAML using a YAML parser.

Then regex can process:

```text
annotation
label
image tag
message
```

if a specific string field requires pattern extraction.

---

# 117. Regex + Security Reports

A security scanner may produce:

```text
CRITICAL: 3
HIGH: 14
MEDIUM: 20
```

Regex can extract candidates:

```python
pattern = re.compile(
    r"\b(?P<severity>CRITICAL|HIGH|MEDIUM|LOW):\s*(?P<count>\d+)"
)
```

For machine-readable reports, prefer JSON output and parse JSON directly.

---

# 118. Security Gate From Text

```python
critical = 0

match = re.search(
    r"CRITICAL:\s*(\d+)",
    report,
)

if match:
    critical = int(
        match.group(1)
    )

if critical > 0:
    raise SystemExit(1)
```

This is acceptable for a simple fixed-format report, but structured scanner output is more robust.

---

# 119. Regex + Jenkins Logs

A Python script may process an exported log:

```text
Started by user
Building...
Tests passed
Deployment failed
```

Regex can locate:

```text
failed
```

But Jenkins' actual build result/API is more authoritative than scraping logs.

---

# 120. Regex + GitHub Actions

A Python utility might inspect textual output:

```text
Error: Process completed with exit code 1.
```

But the workflow status and command exit code should be the primary signal.

---

# 121. Regex + GitLab CI

Similarly, regex can help extract:

```text
job names
failure messages
versions
artifact names
```

from logs, but structured CI metadata is preferred when available.

---

# 122. Production Log Parser

A practical parser:

```python
import re


LOG_PATTERN = re.compile(
    r"""
    timestamp=(?P<timestamp>\S+)
    \s+
    service=(?P<service>\S+)
    \s+
    level=(?P<level>DEBUG|INFO|WARN|ERROR|CRITICAL)
    \s+
    status=(?P<status>\d+)
    \s+
    message=(?P<message>.*)
    """,
    re.VERBOSE,
)


def parse_line(line):
    match = LOG_PATTERN.search(line)

    if not match:
        return None

    data = match.groupdict()

    data["status"] = int(
        data["status"]
    )

    return data
```

This creates structured records from a controlled log format.

---

# 123. Handle Unmatched Lines

Never assume every line matches:

```python
data = parse_line(line)

if data is None:
    # report or count malformed line
    continue
```

Track malformed records separately.

---

# 124. Malformed Log Monitoring

Useful metrics:

```text
total lines
parsed lines
unmatched lines
parse error percentage
```

If parsing suddenly drops from:

```text
99.9%
```

to:

```text
60%
```

the application may have changed its log format.

---

# 125. Regex Monitoring for Format Drift

A robust parser should report:

```text
parsed=99500
unmatched=500
```

instead of silently ignoring unmatched lines.

This helps detect upstream format changes.

---

# 126. Regex Parser Architecture

```text
log file
   |
   v
stream lines
   |
   v
regex
   |
   +--> matched -> structured record
   |
   +--> unmatched -> error metric
   |
   v
aggregate
   |
   v
JSON / CSV / report
```

---

# 127. Performance — Stream Large Files

Good:

```python
with open(
    "large.log",
    encoding="utf-8",
) as file:
    for line in file:
        match = PATTERN.search(line)
```

Avoid:

```python
text = file.read()

matches = PATTERN.findall(text)
```

when the file is huge and only line-level processing is needed.

---

# 128. Performance — Avoid Recompilation

Bad:

```python
for line in file:
    if re.search(
        r"ERROR",
        line,
    ):
        ...
```

Better:

```python
pattern = re.compile(
    r"ERROR"
)

for line in file:
    if pattern.search(line):
        ...
```

The main benefit here is code clarity and reuse; Python also internally caches some recent compiled patterns.

---

# 129. Performance — Keep Patterns Specific

Instead of:

```python
r".*ERROR.*"
```

often simply use:

```python
r"ERROR"
```

The simpler pattern communicates intent and avoids unnecessary work.

---

# 130. Greedy Quantifiers

Example:

```python
r".*"
```

is greedy.

It tries to consume as much as possible while still allowing the rest of the pattern to match.

---

# 131. Lazy Quantifiers

Use:

```python
r".*?"
```

for a lazy match.

Example:

```python
r"<.*?>"
```

can find the shortest tag-like segment.

Do not use regex as a full HTML/XML parser.

---

# 132. Greedy vs Lazy

```text
.*   -> greedy
.*?  -> lazy
```

Use the narrowest expression that matches the intended structure.

---

# 133. Avoid `.*` When Possible

Instead of:

```python
r"service=.* status=(\d+)"
```

prefer:

```python
r"service=(\S+) status=(\d+)"
```

when fields are whitespace-delimited.

This is easier to reason about.

---

# 134. Regex Debugging

When a pattern fails:

```text
1. simplify pattern
2. test literal text
3. add character classes
4. add groups
5. add anchors
6. test edge cases
```

Do not immediately create a huge regex.

---

# 135. Build Regex Incrementally

Start:

```python
r"ERROR"
```

Then:

```python
r"ERROR\s+"
```

Then:

```python
r"ERROR\s+(?P<message>.*)"
```

This makes debugging easier.

---

# 136. Test Positive and Negative Cases

For:

```python
r"\bERROR\b"
```

test:

```text
ERROR
ERROR database
database ERROR
ERROR_CODE
MYERROR
```

A regex is only useful if its boundaries are correct.

---

# 137. Common Regex Mistake — Missing Raw String

Potentially confusing:

```python
"\d+"
```

Prefer:

```python
r"\d+"
```

This avoids Python string escape ambiguity.

---

# 138. Common Mistake — Using `match()` for Log Search

Bad assumption:

```python
re.match(
    r"ERROR",
    line,
)
```

if the line begins with:

```text
2026-08-16 ERROR
```

Use:

```python
re.search(
    r"\bERROR\b",
    line,
)
```

---

# 139. Common Mistake — Forgetting `group()`

```python
match = re.search(
    r"status=(\d+)",
    line,
)

print(match)
```

prints the match object.

Use:

```python
print(
    match.group(1)
)
```

to retrieve the captured value.

---

# 140. Common Mistake — Assuming a Match Exists

Bad:

```python
match = pattern.search(line)

print(match.group(1))
```

If no match exists:

```text
AttributeError
```

Better:

```python
match = pattern.search(line)

if match:
    print(
        match.group(1)
    )
```

---

# 141. Common Mistake — Wrong Group Number

Pattern:

```python
r"(service)=(\w+) status=(\d+)"
```

Groups:

```text
1 -> service
2 -> value
3 -> status
```

Named groups reduce this kind of error.

---

# 142. Common Mistake — Overusing Capturing Groups

Instead of:

```python
r"(ERROR|WARN)\s+(.*)"
```

use non-capturing groups where the group is not needed:

```python
r"(?:ERROR|WARN)\s+(.*)"
```

---

# 143. Common Mistake — Parsing Structured Data With Regex

Bad:

```text
regex entire YAML
regex entire JSON
regex entire Terraform
```

Better:

```text
proper parser
   +
regex only for individual text fields when needed
```

---

# 144. Common Mistake — Ignoring Performance

A pattern that works on:

```text
100 lines
```

may be problematic on:

```text
10 GB
```

Always consider:

```text
input size
pattern complexity
streaming
backtracking
```

---

# 145. Common Mistake — Logging Sensitive Matches

If regex extracts:

```text
password
token
API key
authorization header
private key
```

do not print the matched value.

Redact it immediately or avoid extracting sensitive values unless necessary.

---

# 146. Redaction Function

```python
import re


SECRET_PATTERN = re.compile(
    r"(password|token|secret)"
    r"=(\S+)",
    re.IGNORECASE,
)


def redact(text):
    return SECRET_PATTERN.sub(
        r"\1=***",
        text,
    )
```

Test this carefully against your actual log formats.

---

# 147. Secret Redaction Limitations

A single pattern may miss:

```text
password="secret"
password: secret
"password":"secret"
Authorization: Bearer token
token=secret
token = secret
```

For security-sensitive logging, use a comprehensive redaction strategy and structured logging rather than assuming one regex catches everything.

---

# 148. Multi-Line Stack Traces

A log may look like:

```text
ERROR database failure
Traceback:
...
...
```

A line-by-line regex can detect the first line but not necessarily the complete exception.

Possible solutions:

```text
structured logging
stateful parser
multi-line log configuration
application logging format
```

Do not create an uncontrolled `.*` multi-line regex over huge files.

---

# 149. Regex and ELK

A practical flow:

```text
application logs
       |
       v
Logstash / collector
       |
       v
structured fields
       |
       v
Elasticsearch
       |
       v
Kibana
```

Regex can be used at ingestion for legacy unstructured logs.

If possible, emit structured JSON logs from the application to reduce regex dependence.

---

# 150. Regex and Prometheus

Prometheus metrics should normally be handled as structured metrics rather than regex parsing.

Regex may be used in:

```text
text-based metric inspection
label extraction
diagnostic scripts
```

but Prometheus APIs and metric formats should be parsed appropriately.

---

# 151. Regex and ArgoCD

Regex can help inspect:

```text
sync messages
CLI output
error text
image tags
```

But for robust automation, use ArgoCD APIs/CLI structured output where available.

---

# 152. Regex and Trivy

If Trivy outputs JSON:

```text
use json
```

If output is human-readable:

```text
regex can extract simple fields
```

But security gates should preferably use machine-readable scanner output.

---

# 153. Regex and SonarQube

For SonarQube:

```text
API / JSON
```

is preferable to scraping UI or human-readable logs.

Regex remains useful for quick diagnostic text processing.

---

# 154. Regex and Veracode

For security scan results:

```text
structured API/report
```

is preferable.

Regex is useful for:

```text
legacy text reports
error extraction
quick summaries
```

---

# 155. Regex and Git

Example:

```text
commit abc123
```

Extract candidate SHA:

```python
r"\b[0-9a-f]{7,40}\b"
```

Then validate according to the Git object format or repository context.

---

# 156. Regex and Branch Names

A simple allowed branch pattern might be:

```python
r"[A-Za-z0-9._/-]+"
```

But Git branch naming has specific rules.

For strict validation, use Git's own validation or repository tooling.

---

# 157. Regex and File Names

Find backup files:

```python
pattern = re.compile(
    r"^backup-\d{8}\.tar\.gz$"
)
```

Then:

```python
if pattern.fullmatch(
    path.name
):
    ...
```

This is a good regex use case.

---

# 158. Regex and Release Artifacts

Example:

```text
app-1.4.2-linux-amd64.tar.gz
```

A pattern can extract:

```text
application
version
OS
architecture
```

Then use a version parser for semantic comparison.

---

# 159. Production Artifact Pattern

```python
pattern = re.compile(
    r"""
    ^(?P<name>[a-z0-9-]+)
    -
    (?P<version>\d+\.\d+\.\d+)
    -
    (?P<os>linux|windows)
    -
    (?P<arch>amd64|arm64)
    \.tar\.gz$
    """,
    re.VERBOSE,
)
```

This is appropriate when your organization controls the exact artifact naming convention.

---

# 160. Regex + CI Artifact Validation

Flow:

```text
artifact filename
       |
       v
regex fullmatch
       |
       v
extract metadata
       |
       v
validate version/platform
       |
       v
accept/reject
```

This is a practical DevOps automation pattern.

---

# 161. Regex + Environment Validation

Example:

```python
if not re.fullmatch(
    r"(dev|staging|production)",
    environment,
):
    raise ValueError(
        "Invalid environment"
    )
```

For a small fixed set of values, an ordinary set is often simpler:

```python
{"dev", "staging", "production"}
```

Use regex when the value has a genuine pattern.

---

# 162. Regex + Image Tag Validation

If your team uses:

```text
v1.2.3
```

then:

```python
if not re.fullmatch(
    r"v\d+\.\d+\.\d+",
    tag,
):
    raise ValueError(
        "Invalid image tag"
    )
```

Then use a version parser if ordering/comparison is required.

---

# 163. Regex + Branch Policy

Example policy:

```text
feature/*
bugfix/*
release/*
hotfix/*
```

Pattern:

```python
r"(feature|bugfix|release|hotfix)/[A-Za-z0-9._/-]+"
```

Use `fullmatch()` for validation.

---

# 164. Regex + Deployment Policy

A deployment tool could allow only:

```text
release/v1.2.3
```

Pattern:

```python
r"release/v\d+\.\d+\.\d+"
```

Then:

```python
if not pattern.fullmatch(branch):
    raise SystemExit(1)
```

This is useful as one layer of CI/CD policy enforcement.

---

# 165. Regex + Security Policy

Suppose an artifact must not contain:

```text
debug
latest
snapshot
```

Regex can detect candidate strings.

But policy should be explicit:

```python
if tag == "latest":
    ...
```

For exact fixed values, normal Python conditions are clearer than regex.

---

# 166. Regex + Error Classification

Example:

```python
patterns = {
    "network": re.compile(
        r"timeout|connection refused",
        re.IGNORECASE,
    ),
    "permission": re.compile(
        r"permission denied|forbidden",
        re.IGNORECASE,
    ),
    "authentication": re.compile(
        r"unauthorized|invalid credentials",
        re.IGNORECASE,
    ),
}
```

Then:

```python
for category, pattern in patterns.items():
    if pattern.search(line):
        print(category)
```

This is useful for automated incident triage.

---

# 167. Error Classification Pipeline

```text
error log
   |
   v
normalize
   |
   v
patterns
   |
   +--> network
   +--> permission
   +--> authentication
   +--> dependency
   |
   v
classification
   |
   v
report / alert
```

Use multiple signals rather than a single regex when the classification is important.

---

# 168. Regex for Common Linux Errors

Patterns:

```python
patterns = [
    r"Permission denied",
    r"No such file or directory",
    r"Connection refused",
    r"Address already in use",
    r"Too many open files",
]
```

Compile:

```python
compiled = [
    re.compile(
        pattern,
        re.IGNORECASE,
    )
    for pattern in patterns
]
```

---

# 169. Regex for Kubernetes Errors

Candidate patterns:

```text
CrashLoopBackOff
ImagePullBackOff
ErrImagePull
OOMKilled
FailedMount
FailedScheduling
Readiness probe failed
Liveness probe failed
```

Example:

```python
K8S_ERRORS = re.compile(
    r"""
    CrashLoopBackOff|
    ImagePullBackOff|
    ErrImagePull|
    OOMKilled|
    FailedMount|
    FailedScheduling|
    probe\ failed
    """,
    re.IGNORECASE | re.VERBOSE,
)
```

---

# 170. Kubernetes Incident Classification

```python
match = K8S_ERRORS.search(
    log_text
)

if match:
    print(
        match.group()
    )
```

Then map the error to a troubleshooting workflow.

Regex identifies the symptom; it does not prove the root cause.

---

# 171. Regex for Docker Errors

Candidate patterns:

```text
pull access denied
no space left on device
port is already allocated
container exited
permission denied
```

Again, use actual Docker exit codes and structured inspection to determine the root cause.

---

# 172. Regex for Terraform Errors

Candidate patterns:

```text
Error:
│ Error:
Invalid
Unauthorized
AccessDenied
timeout
```

Use regex to extract context, but the Terraform exit code and machine-readable output are more reliable for automation.

---

# 173. Regex for AWS CLI Errors

Candidate patterns:

```text
AccessDenied
Unauthorized
ExpiredToken
ResourceNotFound
Throttling
```

For robust AWS automation, prefer SDK exceptions and response metadata rather than parsing CLI text.

---

# 174. Regex for Network Troubleshooting

A diagnostic script can classify output from:

```text
ping
curl
nc
ss
dig
```

Examples:

```text
Connection refused
timed out
Name or service not known
Temporary failure in name resolution
```

But when possible, use direct socket/DNS APIs for structured checks.

---

# 175. Regex for `df` Output

You can parse:

```bash
df -h
```

with regex, but it is better to use:

```python
shutil.disk_usage()
```

for disk capacity.

This illustrates an important rule:

> Use regex when text is the input; use an API/library when structured information is already available.

---

# 176. Regex for `ps` Output

Parsing:

```bash
ps -ef
```

with regex can be fragile because command lines and spacing vary.

For process automation, use:

```text
psutil
```

where appropriate.

---

# 177. Regex for `systemctl` Output

If possible:

```bash
systemctl is-active nginx
```

provides a simple status signal.

Do not parse the full human-readable status output when a machine-friendly command exists.

---

# 178. Regex Decision Framework

Before using regex ask:

```text
Is input structured?
      |
  yes | no
      | 
      v
Use parser/API      Regex may help
```

Then ask:

```text
Do I need validation?
      |
      v
Use regex + semantic validator
```

Then:

```text
Is input huge?
      |
      v
Stream it
```

---

# 179. Production Regex Checklist

```text
1. Define the exact input format.
2. Prefer structured parsing when available.
3. Use raw strings.
4. Use named groups for extracted fields.
5. Compile reusable patterns.
6. Use fullmatch for validation.
7. Test positive and negative cases.
8. Stream large files.
9. Avoid catastrophic backtracking.
10. Limit untrusted input size.
11. Do not log sensitive matches.
12. Validate semantic values separately.
13. Monitor unmatched records.
14. Document assumptions.
15. Keep patterns readable.
```

---

# 180. Practical Project — Log Parser

Build:

```text
log_parser.py
```

Input:

```text
application.log
```

Extract:

```text
timestamp
service
level
status
latency
request_id
message
```

Output:

```text
JSON
CSV
summary
```

Requirements:

```text
stream file
handle malformed lines
count parse failures
avoid logging secrets
```

---

# 181. Practical Project — Secret Redactor

Build:

```text
redact_logs.py
```

Detect and redact:

```text
password
token
API key
Authorization header
```

Input:

```text
raw.log
```

Output:

```text
sanitized.log
```

Requirements:

```text
never print original secret
preserve non-sensitive content
test multiple formats
```

---

# 182. Practical Project — Kubernetes Error Classifier

Input:

```text
kubectl logs output
```

Classify:

```text
OOM
image pull
probe
network
permission
application
```

Output:

```text
category
matching line
pod/service
severity
```

Use regex only for candidate detection; investigate actual Kubernetes state for root cause.

---

# 183. Practical Project — CI Failure Analyzer

Input:

```text
pipeline.log
```

Detect candidate categories:

```text
dependency
test
build
security
Docker
Terraform
Kubernetes
AWS
network
```

Generate:

```text
failure-summary.json
```

This is a strong Python-for-DevOps portfolio project.

---

# 184. Practical Project — Artifact Validator

Input:

```text
payment-v1.4.2-linux-amd64.tar.gz
```

Extract:

```text
name
version
OS
architecture
```

Validate:

```text
approved naming convention
supported version format
supported platform
```

---

# 185. Practical Project — Deployment Log Analyzer

Input:

```text
deployment.log
```

Calculate:

```text
total lines
errors
warnings
HTTP 4xx
HTTP 5xx
timeouts
failed services
request IDs
```

Output:

```text
deployment-report.json
```

---

# 186. Practical Project — Security Report Gate

Input:

```text
security-report.json
```

Prefer JSON parsing.

If a legacy text report must be supported:

```text
regex extraction
      |
      v
normalized dictionary
      |
      v
policy evaluation
      |
      v
exit code
```

---

# 187. Interview — What Is Regex?

Answer:

> Regex is a pattern language used to search, match, extract, validate, and replace text. In DevOps I commonly use it for log analysis, extracting identifiers, validating controlled naming conventions, redacting secrets, and processing legacy command output.

---

# 188. Interview — `match()` vs `search()`

Answer:

> `re.match()` checks from the beginning of the string, while `re.search()` looks for a match anywhere in the string. For log lines where the target can appear anywhere, I normally use `search()`.

---

# 189. Interview — `findall()` vs `finditer()`

Answer:

> `findall()` returns collected matches, while `finditer()` produces match objects lazily. For large input or when I need match positions and named groups, I prefer `finditer()`.

---

# 190. Interview — Why Use Named Groups?

Answer:

> Named groups make extraction more readable and maintainable. Instead of remembering that group 3 contains latency, I can use `match.group("latency")`.

---

# 191. Interview — How Do You Parse Large Logs?

Answer:

> I stream the file line by line, compile reusable regex patterns, extract only the fields I need, and avoid loading the entire log into memory. For structured logs I prefer the existing structured format over regex.

---

# 192. Interview — Would You Parse JSON With Regex?

Answer:

> No, not as the primary parser. I would use `json.loads()` because JSON is structured. Regex may still be useful for processing one string field inside the parsed JSON.

---

# 193. Interview — How Would You Validate an IP?

Answer:

> I might use regex to locate candidate IPv4 strings, but I would use Python's `ipaddress` module for semantic validation. Regex alone can easily accept values such as `999.999.999.999`.

---

# 194. Interview — How Do You Prevent Regex Performance Problems?

Answer:

> I keep patterns simple, avoid ambiguous nested quantifiers, limit untrusted input size, stream large data, test worst-case inputs, and avoid patterns that can cause catastrophic backtracking.

---

# 195. Interview — How Do You Redact Secrets With Regex?

Answer:

> I identify the specific formats used by the application, replace the sensitive value with a fixed mask, and ensure the original value is never logged. I test multiple formats because one regex rarely catches every possible secret representation.

---

# 196. Scenario — Log Parser Suddenly Stops Extracting Fields

Investigate:

```text
application log format changed
timestamp changed
field order changed
new quoting
new multiline behavior
new service version
```

Check:

```text
matched count
unmatched count
sample unmatched lines
deployment history
```

Do not silently discard unmatched records.

---

# 197. Scenario — Regex Uses 100% CPU

Investigate:

```text
pattern complexity
input size
nested quantifiers
backtracking
unexpected multiline input
```

Actions:

```text
stop unsafe processing
limit input
simplify pattern
test worst-case input
replace with parser if appropriate
```

---

# 198. Scenario — Security Scanner Regex Gives Wrong Result

Possible cause:

```text
human-readable format changed
regex matched unrelated text
missing severity boundary
duplicate results
```

Better:

```text
use scanner JSON/API
validate schema
apply explicit policy
```

Do not rely on a fragile text scrape for a critical security gate.

---

# 199. Scenario — Kubernetes Parser Breaks After Upgrade

Likely issue:

```text
kubectl human-readable output changed
```

Better:

```bash
kubectl get ... -o json
```

and parse JSON.

Or use the Kubernetes API/client.

This is a classic example of why structured interfaces are better than regex against CLI formatting.

---

# 200. Scenario — Log Contains Password

Immediate action:

```text
do not print match
redact
rotate exposed credential if necessary
identify why secret was logged
```

Regex can help with automated redaction, but the permanent fix is to stop the application or pipeline from logging the secret.

---

# 201. Scenario — Regex Validates Deployment Name

Use:

```python
if not re.fullmatch(
    r"[a-z0-9-]+",
    name,
):
    raise ValueError(
        "Invalid deployment name"
    )
```

Then apply:

```text
maximum length
platform-specific rules
reserved names
```

if required.

---

# 202. Scenario — Extract Error From Terraform

Use regex to locate:

```text
Error:
```

and capture nearby context.

But also capture:

```text
terraform return code
```

and prefer:

```text
Terraform JSON output
```

when machine-readable details are available.

---

# 203. Scenario — Extract Image Tag

Input:

```text
image=registry.example.com/payment:v1.4.2
```

Use named groups:

```python
pattern = re.compile(
    r"image=(?P<image>[^:\s]+)"
    r":(?P<tag>\S+)"
)
```

Be aware that registry references can contain ports, so a production parser should account for:

```text
registry.example.com:5000/payment:v1.4.2
```

rather than assuming the first colon separates image and tag.

---

# 204. Advanced Image Reference Parsing

Because container image references can contain:

```text
registry
registry port
repository
namespace
tag
digest
```

a simplistic regex can easily be wrong.

For production-grade image handling, use a dedicated parser or define the accepted reference grammar precisely.

---

# 205. Regex and Digest References

Example:

```text
payment@sha256:abcdef...
```

A candidate pattern:

```python
r"@sha256:[0-9a-fA-F]{64}"
```

can locate a SHA-256 digest.

The digest should be treated as an immutable content identifier, unlike a mutable tag such as:

```text
latest
```

---

# 206. Regex and Kubernetes Image Validation

A CI policy may reject:

```text
:latest
```

using:

```python
if re.search(
    r":latest$",
    image,
):
    raise SystemExit(
        "latest tag is not allowed"
    )
```

But image parsing should account for registry ports and digests.

---

# 207. Regex and Production Deployment Policies

Regex can enforce simple policies such as:

```text
release branch naming
artifact naming
environment naming
version format
resource naming candidates
```

Combine regex with:

```text
semantic validation
platform APIs
policy engines
```

for production controls.

---

# 208. Regex Utility Module

A reusable module might contain:

```python
import re


ERROR_PATTERN = re.compile(
    r"\bERROR\b",
    re.IGNORECASE,
)

REQUEST_ID_PATTERN = re.compile(
    r"request_id=(?P<request_id>[\w-]+)"
)

STATUS_PATTERN = re.compile(
    r"\b(?P<status>[1-5]\d{2})\b"
)
```

Then other scripts can import them.

This connects regex with:

```text
01-Modules-and-Packages.md
```

---

# 209. Recommended Regex Utility Structure

```text
devops_utils/
├── __init__.py
├── regex_patterns.py
├── log_parser.py
├── redaction.py
└── validators.py
```

Keep:

```text
patterns
parsing
validation
redaction
```

separate where the project becomes large.

---

# 210. Regex Test Suite

```text
tests/
├── test_patterns.py
├── test_log_parser.py
├── test_redaction.py
└── test_validators.py
```

Include:

```text
happy paths
edge cases
invalid input
security cases
large input
format changes
```

---

# 211. DevOps Regex Cheat Sheet

```text
.        any character
\d       digit
\D       non-digit
\w       word character
\W       non-word character
\s       whitespace
\S       non-whitespace
*        zero or more
+        one or more
?        zero or one
{n}      exactly n
{n,}     at least n
{n,m}    n through m
^        beginning
$        end
[]       character class
[^]      negated character class
()       capture group
(?:)     non-capturing group
(?P<>)   named group
|        OR
\b       word boundary
```

---

# 212. Python Regex API Cheat Sheet

```python
re.search()
re.match()
re.fullmatch()
re.findall()
re.finditer()
re.sub()
re.split()
re.compile()
```

Useful match methods:

```python
match.group()
match.group(1)
match.group("name")
match.groups()
match.groupdict()
match.start()
match.end()
match.span()
```

---

# 213. Final DevOps Mental Model

Do not think:

```text
Regex = complicated symbols
```

Think:

```text
raw text
   |
   v
identify structure
   |
   v
write smallest useful pattern
   |
   v
extract candidate
   |
   v
semantic validation
   |
   v
structured data
   |
   v
automation decision
```

The strongest DevOps engineers know when to use regex and, equally importantly, **when not to use regex**.

---

# 214. Final Rules to Remember

```text
1. Use raw strings for regex patterns.
2. Prefer search() for text scanning.
3. Prefer fullmatch() for validation.
4. Use named groups for extraction.
5. Use finditer() for large input.
6. Compile reusable patterns.
7. Keep patterns simple.
8. Avoid catastrophic backtracking.
9. Stream large log files.
10. Do not parse JSON/YAML with regex.
11. Use semantic validators after extraction.
12. Never print secrets matched by regex.
13. Test positive and negative cases.
14. Track unmatched log records.
15. Prefer structured APIs over CLI text scraping.
16. Use exit codes for automation decisions.
17. Document the exact input format.
18. Treat regex as one component of a larger parser.
```

---

# 215. Next File

```text
05-JSON-YAML-CSV.md
```

The next file will cover:

```text
JSON
JSON parsing
JSON generation
nested JSON
JSON validation
JSON APIs
YAML
safe YAML parsing
Kubernetes manifests
Helm values
Terraform-related data
CSV
CSV reports
DictReader
DictWriter
CI/CD reports
security scan reports
AWS/Kubernetes automation
configuration transformation
JSON/YAML/CSV conversion
production automation
error handling
security
real DevOps scripts
interview questions
scenario-based questions
```
