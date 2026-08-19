# Basic Questions

> Python fundamentals for DevOps Engineer interviews, with practical examples and production-oriented follow-ups.

## Interview Answer Framework

**Definition → Small example → DevOps use → Production consideration → Follow-up/trade-off**

Do not memorize long paragraphs. Practice answering each question naturally in 30–90 seconds, then expand when the interviewer asks for implementation or production details.

## 1. How to Use This File

Answer each question aloud using: definition → example → DevOps use case → production consideration. Expect follow-ups on security, errors, scale, testing, and troubleshooting.

---

## 2. What Is Python?

Python is a high-level, general-purpose programming language with readable syntax and a large ecosystem. DevOps teams use it for AWS/Kubernetes automation, API integration, CI/CD orchestration, testing, reporting, and operational tooling.

---

## 3. Why Python for DevOps?

Python has strong AWS and Kubernetes SDKs, HTTP/API libraries, JSON/YAML support, subprocess handling, testing frameworks, and easy integration with CI/CD systems.

---

## 4. Python vs Bash

Bash is excellent for short Linux command automation. Python is preferable when automation needs complex data structures, APIs, retries, testing, structured logging, or maintainable multi-step workflows.

---

## 5. CPython

CPython is the most commonly used Python implementation. It compiles source to bytecode and executes it using the Python runtime.

---

## 6. Script vs Application

A one-off script can be simple. Recurring production automation should have configuration, logging, tests, error handling, security, dependency management, and documentation.

---

## 7. Variable

A variable is a name bound to an object. Python is dynamically typed, so names do not have fixed declared types.

---

## 8. Dynamic Typing

Types are determined at runtime. Type hints, static analysis, tests, and runtime validation improve reliability in large automation projects.

---

## 9. Strong Typing

Python is dynamically typed but strongly typed; unrelated types are not silently combined in many operations.

---

## 10. Object

Everything in Python is an object, including numbers, strings, functions, classes, and modules.

---

## 11. Identity vs Equality

`is` compares object identity; `==` compares values. Use `x is None` for the common singleton check.

---

## 12. None

`None` represents absence of a value and is different from `0`, `False`, or an empty collection.

---

## 13. Boolean and Truthiness

Python treats values such as `None`, `False`, `0`, and empty collections as false-like. Validate explicitly when those values have different meanings.

---

## 14. Integer

Python integers have arbitrary precision, limited mainly by available memory.

---

## 15. Float

Python floats use binary floating-point representation. Avoid assuming decimal values are represented exactly.

---

## 16. String

Strings are immutable Unicode sequences and are heavily used for configuration, paths, API data, logs, and identifiers.

---

## 17. Bytes

Bytes represent immutable binary data and are useful for raw files, network data, and encoded payloads.

---

## 18. List

A list is an ordered mutable collection. Use it for sequences such as resources or pipeline stages.

---

## 19. Tuple

A tuple is an ordered immutable collection and is useful for fixed records or values that should not change.

---

## 20. Set

A set stores unique hashable values and is useful for deduplication and fast membership checks.

---

## 21. Dictionary

A dictionary stores key-value mappings and is fundamental for JSON responses, configuration, and resource metadata.

---

## 22. Mutable vs Immutable

Lists, dictionaries, and sets are mutable. Strings, integers, tuples, and bytes are immutable. Understanding this prevents shared-state bugs.

---

## 23. Hashable

Hashable objects can be dictionary keys or set members. Strings and integers are common examples.

---

## 24. Indexing

Python sequences use zero-based indexes. Negative indexes access elements from the end.

---

## 25. Slicing

Slicing uses `start:stop:step`; the stop index is exclusive.

---

## 26. List vs Set

Use a list when order or duplicates matter; use a set when uniqueness and membership checks matter.

---

## 27. Dictionary Lookup

Dictionary lookup is typically average O(1), making dictionaries useful for mapping IDs or names to status information.

---

## 28. Operators

Python provides arithmetic, comparison, logical, identity, membership, assignment, and bitwise operators.

---

## 29. Short Circuiting

`and` and `or` stop evaluating once the result is known. This can safely guard optional values.

---

## 30. if/elif/else

Conditional statements select execution paths. Production safety rules should remain explicit and readable.

---

## 31. for Loop

`for` iterates over an iterable and is commonly used for AWS resources, Kubernetes objects, files, and API results.

---

## 32. while Loop

`while` repeats while a condition remains true. Always use bounded attempts/deadlines for production retry loops.

---

## 33. break and continue

`break` exits a loop; `continue` skips to the next iteration. Use them only when they keep control flow clear.

---

## 34. range

`range()` generates bounded integer sequences without creating a full list.

---

## 35. enumerate

`enumerate()` provides index-value pairs without manually maintaining a counter.

---

## 36. zip

`zip()` combines iterables element by element. Validate lengths when positional alignment matters.

---

## 37. Function

Functions encapsulate reusable behavior. Production functions should have clear inputs, outputs, and side effects.

---

## 38. Function Arguments

Python supports positional and keyword arguments. Keyword arguments improve clarity for configuration-heavy calls.

---

## 39. Default Arguments

Defaults are evaluated when a function is defined. Never use mutable defaults such as `items=[]`; use `None` and initialize inside.

---

## 40. return

Use explicit return values for reusable logic instead of making every function print output.

---

## 41. Type Hints

Type hints document interfaces and support tools such as mypy. They complement, rather than replace, runtime validation.

---

## 42. Docstrings

Docstrings document public functions/classes, including side effects, exceptions, and important assumptions.

---

## 43. Scope

Python resolves names through local, enclosing, global, and built-in scopes.

---

## 44. LEGB

LEGB means Local, Enclosing, Global, Built-in and describes normal name lookup.

---

## 45. global

`global` modifies a module-level name from a function. Avoid global mutable state in production automation.

---

## 46. Comprehensions

List/set/dictionary comprehensions provide concise collection creation. Keep them simple enough to remain readable.

---

## 47. Generator Expression

Generator expressions create values lazily and can reduce memory use for large datasets.

---

## 48. lambda

Lambda creates a small anonymous function. Use it for simple transformations; named functions are clearer for complex logic.

---

## 49. map/filter/reduce

These functional tools transform/filter/combine values. Comprehensions or built-ins are often clearer in production code.

---

## 50. args and kwargs

`*args` collects positional arguments and `**kwargs` collects keyword arguments. Use them for flexible wrappers, not to hide a clear interface.

---

## 51. Unpacking

`*` unpacks iterables and `**` unpacks mappings. Use carefully when external data shapes are not guaranteed.

---

## 52. Module

A module is generally a Python file that can be imported.

---

## 53. Package

A package organizes related modules into a reusable project structure.

---

## 54. Standard Library

Useful modules include pathlib, os, subprocess, json, logging, argparse, datetime, collections, and concurrent.futures.

---

## 55. main Guard

`if __name__ == '__main__':` ensures CLI execution code runs only when the file is executed directly.

---

## 56. Environment Variables

Use `os.getenv()` or `os.environ` for runtime configuration. Validate required variables at startup.

---

## 57. Secrets

Never hard-code passwords, API tokens, cloud keys, private keys, or Kubernetes Secret values.

---

## 58. Pathlib

`pathlib.Path` provides readable filesystem path handling and avoids fragile manual path concatenation.

---

## 59. File Handling

Use `with open(...)` so files close reliably even if an exception occurs.

---

## 60. JSON

The `json` module handles API payloads, configuration, and release reports.

---

## 61. load vs loads

`json.load()` reads JSON from a file-like object; `json.loads()` parses a string.

---

## 62. dump vs dumps

`json.dump()` writes JSON to a file-like object; `json.dumps()` returns a JSON string.

---

## 63. YAML

YAML is common in Kubernetes and CI/CD. Use safe loaders and validate the resulting structure.

---

## 64. Datetime

Use timezone-aware datetimes for production automation and prefer UTC internally.

---

## 65. Logging

Use Python's `logging` module instead of scattered print statements for production automation.

---

## 66. Structured Logging

Include fields such as run ID, environment, stage, resource, status, and error class so ELK can correlate events.

---

## 67. Exception

An exception represents an exceptional condition that interrupts normal control flow.

---

## 68. try/except

Catch exceptions where the program can meaningfully handle or classify the failure.

---

## 69. finally

`finally` executes cleanup regardless of success or failure.

---

## 70. raise

Use `raise` to create or propagate errors. Preserve useful context.

---

## 71. Custom Exceptions

Create domain-specific exceptions such as ConfigurationError, AuthenticationError, DeploymentError, or PolicyViolation when they improve error handling.

---

## 72. Exception Chaining

`raise NewError(...) from exc` preserves the original cause while adding domain context.

---

## 73. Bare except

Avoid bare `except:` because it catches too broadly, including interrupts.

---

## 74. Retryable Errors

Classify transient network/API errors separately from permanent validation, authentication, or authorization failures.

---

## 75. Retry Policy

Production retries need bounded attempts, total deadline, exponential backoff, and jitter.

---

## 76. Context Manager

`with` manages resource setup/cleanup and reduces resource-leak bugs.

---

## 77. Class

A class groups state and related behavior. API clients and release managers are common DevOps examples.

---

## 78. OOP

Object-oriented design can organize components such as AWSClient, KubernetesClient, ReleaseManager, and HealthChecker.

---

## 79. Constructor

`__init__` initializes an object. Avoid unexpected network calls inside constructors.

---

## 80. self

`self` refers to the current instance in an instance method.

---

## 81. Inheritance

Inheritance creates a subtype relationship. Prefer composition when components merely need reusable behavior.

---

## 82. Composition

Composition combines smaller objects and is often easier to test and change than deep inheritance.

---

## 83. Encapsulation

Encapsulation hides implementation details behind clear interfaces. Python relies mainly on conventions.

---

## 84. Polymorphism

Different implementations can provide the same interface, which is useful for interchangeable real and mocked clients.

---

## 85. Dataclass

Dataclasses reduce boilerplate for structured models such as configuration, health results, and release metadata.

---

## 86. Enum

Enums provide named constants for fixed states such as HEALTHY, WARNING, CRITICAL, and UNKNOWN.

---

## 87. Iterator

An iterator produces values one at a time using the iterator protocol.

---

## 88. Iterable

An iterable can provide an iterator and can be consumed by a `for` loop.

---

## 89. Generator

A generator function uses `yield` to produce values lazily.

---

## 90. yield

`yield` pauses a generator and returns a value while preserving execution state.

---

## 91. Generator vs List

A list stores all results in memory; a generator processes values lazily and is useful for large API/resource sets.

---

## 92. Memory Efficiency

Process paginated cloud/Kubernetes results incrementally rather than loading huge datasets into memory.

---

## 93. Garbage Collection

CPython uses reference counting plus cyclic garbage collection. Developers should still release unnecessary large objects and resources.

---

## 94. Aliasing

Two names can reference the same mutable object. Changes through one name affect the other.

---

## 95. Shallow vs Deep Copy

A shallow copy duplicates the outer object but shares nested objects; a deep copy recursively copies nested data and can be expensive.

---

## 96. Mutable Default Trap

`def f(x=[]):` reuses the same list across calls. Use `None` and initialize inside.

---

## 97. Decorator

A decorator wraps a function/class to add behavior such as logging or timing. Keep retry/security decorators explicit and bounded.

---

## 98. Regex

The `re` module handles pattern matching. Use normal string methods when they are clearer.

---

## 99. f-strings

F-strings provide readable interpolation. Never accidentally interpolate secrets into logs.

---

## 100. PEP 8

PEP 8 provides Python style guidance. Follow the repository's automated formatting/linting policy.

---

## 101. Linting

Ruff/Flake8 can catch style and some correctness issues before runtime.

---

## 102. Type Checking

Mypy or similar tools catch interface/type mistakes before runtime.

---

## 103. Virtual Environment

A virtual environment isolates dependencies from system Python and other projects.

---

## 104. venv

Python's built-in `venv` module creates isolated environments.

---

## 105. pip

pip installs Python packages from configured package indexes.

---

## 106. requirements.txt

A requirements file records dependencies. Pin/constrain versions appropriately for reproducible builds.

---

## 107. Lockfile

A lockfile records resolved dependency versions and improves reproducibility.

---

## 108. Private Package Index

Artifactory or another private index can host dependencies. Authenticate securely and never put credentials in source.

---

## 109. Dependency Security

Scan dependencies for known vulnerabilities and update through controlled testing.

---

## 110. ImportError

ImportError indicates an import problem or missing imported symbol.

---

## 111. ModuleNotFoundError

A specific ImportError indicating the requested module could not be found.

---

## 112. SyntaxError

SyntaxError indicates invalid Python syntax.

---

## 113. NameError

NameError occurs when a referenced name is not defined.

---

## 114. TypeError

TypeError occurs when an operation receives an inappropriate type.

---

## 115. ValueError

ValueError occurs when a type is valid but the value is inappropriate.

---

## 116. KeyError

KeyError occurs when a dictionary key is absent. Use `.get()` when absence is expected, but validate required fields.

---

## 117. IndexError

IndexError occurs when a sequence index is outside its valid range.

---

## 118. FileNotFoundError

Occurs when a requested filesystem path does not exist.

---

## 119. PermissionError

Indicates insufficient operating-system/filesystem permissions.

---

## 120. JSONDecodeError

Indicates invalid JSON input. External responses should be validated.

---

## 121. CLI

Use argparse or an approved CLI framework for command-line automation.

---

## 122. Exit Codes

Return stable exit codes so Jenkins/GitHub Actions can distinguish success from failure.

---

## 123. stdout/stderr

Use stdout for normal output and stderr for errors/diagnostics when appropriate. Keep machine-readable output separate from human logs.

---

## 124. subprocess

Use subprocess when an external tool such as Git, Helm, Terraform, or kubectl must be invoked and no suitable Python API exists.

---

## 125. subprocess Security

Prefer argument arrays and `check=True`; avoid `shell=True` with untrusted input.

---

## 126. Command Timeout

Set subprocess timeouts so a stuck command cannot hang a CI pipeline indefinitely.

---

## 127. Shell Injection

Never concatenate untrusted branch names, tags, paths, or user input into shell commands.

---

## 128. Collections

The collections module provides useful structures such as Counter, defaultdict, and deque.

---

## 129. Counter

Counter is useful for summarizing log levels, resource statuses, or failure categories.

---

## 130. defaultdict

defaultdict simplifies grouping/counting when a default value is appropriate.

---

## 131. secrets

Use the secrets module for security-sensitive random values; do not use random for credentials/tokens.

---

## 132. hashlib

hashlib provides cryptographic hashes such as SHA-256 for integrity and release metadata.

---

## 133. Hash vs Encryption

Hashing provides one-way integrity representation; encryption provides reversible confidentiality with a key.

---

## 134. Base64

Base64 is encoding, not encryption. It must not be used to protect secrets.

---

## 135. pickle

Never load untrusted pickle data because deserialization can execute arbitrary code. Prefer JSON/safe formats across trust boundaries.

---

## 136. Testing

Unit tests validate small pieces of logic; integration tests validate system interactions; end-to-end tests validate the full workflow.

---

## 137. pytest

pytest is a common Python testing framework and is especially useful for DevOps automation.

---

## 138. Mocking

Mock AWS/Kubernetes/CI APIs in unit tests so tests are deterministic and do not require live infrastructure.

---

## 139. AWS DevOps Example

With boto3, query EC2/EKS/S3 resources, filter results, and report status. Production code must consider pagination, permissions, retries, throttling, and safe credentials.

---

## 140. Kubernetes DevOps Example

With the Kubernetes Python client, list Pods, inspect container states, and report unhealthy workloads. Consider RBAC, API load, timeouts, and cluster identity.

---

## 141. API DevOps Example

Call an HTTP API with a timeout, validate status, parse JSON safely, classify retryable errors, and return structured results.

---

## 142. Log Parsing Example

Read a log and count ERROR/WARN/INFO using Counter. For large logs, process line by line rather than loading the entire file.

---

## 143. Disk Check Example

Read filesystem statistics, calculate usage percentage, and compare against configurable warning/critical thresholds.

---

## 144. Retry Coding Example

Implement bounded retries with exponential backoff and jitter; retry only transient failures and preserve the final cause.

---

## 145. Cleanup Coding Example

Find old files using pathlib, but production cleanup requires dry-run, allowlists, age thresholds, protected paths, and audit logging.

---

## 146. Interview Answer Pattern

For a basic question: define it, give a small example, explain where you used it, then mention one production consideration.

---

## 147. Common Interview Mistake

Giving only textbook definitions. DevOps interviewers usually want practical application, failure handling, security, and troubleshooting.

---

## 148. Production Readiness

A production Python tool should have configuration validation, dependency control, secure credentials, structured logging, timeouts, retries, tests, documentation, and controlled deployment.

---

## 149. When to Use Python

Use Python when automation needs structured data, APIs, complex logic, reusable components, testing, or maintainability beyond a short shell command.

---

## 150. When Not to Use Python

Do not add Python just to wrap one simple Linux command if a small, readable shell command is sufficient.

---

## 151. Boto3

boto3 is the AWS SDK for Python. Production use requires least-privilege IAM, pagination, throttling awareness, and reliable error handling.

---

## 152. Kubernetes Python Client

It provides access to Kubernetes APIs from Python. Production use requires correct RBAC, cluster identity, timeouts, retries, and controlled API volume.

---

## 153. Pagination

Many cloud APIs paginate results. Use SDK paginators or continue through pages rather than assuming one response contains everything.

---

## 154. API Rate Limits

Avoid aggressive loops. Use pagination, caching, bounded concurrency, and backoff.

---

## 155. Idempotency

An automation is idempotent when repeating the same desired operation does not create unintended extra changes. This is critical for CI/CD and infrastructure tooling.

---

## 156. Dry Run

Dry-run shows intended changes without mutation. It is particularly important for cleanup, infrastructure, and deployment automation.

---

## 157. Security

Never hard-code secrets, avoid shell injection, use least privilege, validate inputs, use TLS, redact logs, and scan dependencies/images.

---

## 158. External API Testing

Mock external clients for unit tests and use isolated environments for integration tests.

---

## 159. Production Troubleshooting

Start with the failed operation, inspect structured logs and the exception cause, verify configuration and identity, check dependencies, reproduce safely, and apply the smallest controlled fix.

---

## 160. Final Basic Answer

Python is a high-level, dynamically typed language with readable syntax and a strong ecosystem. In DevOps I use it for AWS/Kubernetes APIs, CI/CD orchestration, automation, testing, and reporting. For production I focus on validation, security, idempotency, logging, retries, timeouts, testing, and safe deployment.

---

## Section Progress

```text
12-Python-Interview-Preparation/
├── 01-Basic-Questions.md              ✓
├── 02-Intermediate-Questions.md
├── 03-Advanced-Questions.md
├── 04-Python-for-DevOps-Questions.md
├── 05-Scenario-Based.md
├── 06-Coding-Questions.md
├── 07-Production-Scenarios.md
└── 08-Mock-Interview.md
```

**Next file: `02-Intermediate-Questions.md`**