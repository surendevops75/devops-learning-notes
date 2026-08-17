# 08-Argparse

## DevOps Focus

DevOps engineers frequently turn Python scripts into command-line tools.

Examples:

```text
python deploy.py
python cleanup.py
python health_check.py
python aws_inventory.py
python k8s_check.py
python backup.py
python rollback.py
```

A production CLI should be:

```text
predictable
self-documenting
validated
safe
scriptable
automation-friendly
```

Python's built-in `argparse` module provides the standard foundation for building these tools.

> Core principle: **A DevOps CLI should make the correct action easy and the dangerous action difficult.**

---

## 1. Import `argparse`

```python
import argparse
```

Create a parser:

```python
parser = argparse.ArgumentParser(
    description="Deployment automation tool"
)
```

Then:

```python
args = parser.parse_args()
```

---

## 2. First Simple CLI

```python
import argparse

parser = argparse.ArgumentParser(
    description="DevOps utility"
)

parser.add_argument(
    "--environment"
)

args = parser.parse_args()

print(args.environment)
```

Run:

```bash
python tool.py --environment production
```

Output:

```text
production
```

---

## 3. Why CLI Arguments Matter

Without arguments:

```python
environment = "production"
```

The script is hard-coded.

With arguments:

```bash
python deploy.py --environment production
```

the same script can run against:

```text
dev
staging
production
```

without changing source code.

---

## 4. Positional Arguments

```python
parser.add_argument(
    "environment"
)
```

Run:

```bash
python deploy.py production
```

Access:

```python
args.environment
```

Positional arguments are useful when the command has a natural required sequence.

---

## 5. Optional Arguments

```python
parser.add_argument(
    "--environment"
)
```

Run:

```bash
python deploy.py \
    --environment production
```

Optional arguments are better when commands have multiple independent settings.

---

## 6. Short Options

You can provide:

```python
parser.add_argument(
    "-e",
    "--environment",
)
```

Then either works:

```bash
python deploy.py -e production
```

or:

```bash
python deploy.py --environment production
```

Use short flags for commonly used options.

---

## 7. Help Automatically

`argparse` automatically provides:

```bash
python deploy.py --help
```

Example:

```text
usage: deploy.py [-h] --environment ENVIRONMENT

DevOps deployment utility
```

This is one major advantage of `argparse`.

---

## 8. Help Is Part of the Interface

A production CLI should explain:

```text
what the command does
required arguments
optional arguments
allowed values
examples where useful
```

A good CLI should be understandable without opening the source code.

---

## 9. Required Optional Argument

```python
parser.add_argument(
    "--environment",
    required=True,
)
```

Now:

```bash
python deploy.py
```

fails with a useful error.

Correct:

```bash
python deploy.py \
    --environment production
```

---

## 10. Defaults

```python
parser.add_argument(
    "--timeout",
    type=int,
    default=300,
)
```

If omitted:

```python
args.timeout
```

is:

```text
300
```

Defaults should be safe and documented.

---

## 11. Type Conversion

Without:

```python
type=int
```

the value is a string.

Use:

```python
parser.add_argument(
    "--timeout",
    type=int,
)
```

Then:

```python
args.timeout
```

is an integer.

---

## 12. Boolean Flag

A common flag:

```python
parser.add_argument(
    "--dry-run",
    action="store_true",
)
```

Run:

```bash
python cleanup.py --dry-run
```

Then:

```python
args.dry_run
```

is:

```text
True
```

Without the flag:

```text
False
```

---

## 13. Why `store_true` Is Useful

It is ideal for switches:

```text
--dry-run
--verbose
--force
--debug
--skip-validation
```

Example:

```bash
python deploy.py --dry-run
```

---

## 14. Dangerous `--force`

A destructive script might have:

```python
parser.add_argument(
    "--force",
    action="store_true",
)
```

But do not make `--force` silently bypass all safety controls.

For production deletion tools, combine it with:

```text
environment validation
resource validation
dry-run
confirmation policy
allowlists
```

---

## 15. `choices`

Restrict values:

```python
parser.add_argument(
    "--environment",
    choices=[
        "dev",
        "staging",
        "production",
    ],
)
```

Invalid:

```bash
python deploy.py \
    --environment test123
```

`argparse` rejects it.

---

## 16. Why `choices` Matters

It prevents invalid input before business logic starts.

Instead of:

```python
if environment not in allowed:
    ...
```

you can express the CLI contract directly.

Use `choices` when the valid set is small and stable.

---

## 17. Integer Validation

Basic:

```python
parser.add_argument(
    "--retries",
    type=int,
)
```

But:

```text
-5
```

may still be accepted.

Use custom validation for business constraints.

---

## 18. Custom Type Validation

```python
def positive_int(value):
    value = int(value)

    if value <= 0:
        raise argparse.ArgumentTypeError(
            "must be positive"
        )

    return value
```

Then:

```python
parser.add_argument(
    "--retries",
    type=positive_int,
)
```

---

## 19. Positive Timeout

```python
def positive_int(value):
    value = int(value)

    if value <= 0:
        raise argparse.ArgumentTypeError(
            "value must be greater than zero"
        )

    return value
```

Use:

```python
parser.add_argument(
    "--timeout",
    type=positive_int,
    default=300,
)
```

---

## 20. Range Validation

```python
def percentage(value):
    value = int(value)

    if not 0 <= value <= 100:
        raise argparse.ArgumentTypeError(
            "must be between 0 and 100"
        )

    return value
```

Useful for:

```text
threshold
percentage
replica percentage
rollout percentage
```

---

## 21. `nargs`

Accept multiple values:

```python
parser.add_argument(
    "--service",
    nargs="+",
)
```

Run:

```bash
python deploy.py \
    --service user product payment
```

Then:

```python
args.service
```

contains a list.

---

## 22. One or More Values

```text
nargs="+"
```

means:

```text
one or more
```

Example:

```bash
--service user payment
```

---

## 23. Zero or More Values

```text
nargs="*"
```

means:

```text
zero or more
```

Use carefully because missing values can be ambiguous.

---

## 24. Exactly N Values

```python
parser.add_argument(
    "--range",
    nargs=2,
)
```

Example:

```bash
--range 10 20
```

Useful when an option always requires a fixed number of related values.

---

## 25. `action="append"`

Useful when the same option can appear multiple times:

```python
parser.add_argument(
    "--tag",
    action="append",
)
```

Run:

```bash
python deploy.py \
    --tag team=platform \
    --tag env=prod
```

Result:

```python
[
    "team=platform",
    "env=prod",
]
```

---

## 26. `action="count"`

Useful for verbosity:

```python
parser.add_argument(
    "-v",
    "--verbose",
    action="count",
    default=0,
)
```

Then:

```bash
-v
```

means:

```text
1
```

and:

```bash
-vv
```

means:

```text
2
```

You can map verbosity to logging levels.

---

## 27. Verbose Logging

```python
if args.verbose >= 2:
    level = logging.DEBUG
elif args.verbose == 1:
    level = logging.INFO
else:
    level = logging.WARNING
```

A more explicit logging policy is often better for production CLIs.

---

## 28. `dest`

You can control the attribute name:

```python
parser.add_argument(
    "--dry-run",
    dest="dry_run",
    action="store_true",
)
```

Then:

```python
args.dry_run
```

Hyphenated CLI options automatically map naturally to underscore attributes in common cases.

---

## 29. `metavar`

Improve help output:

```python
parser.add_argument(
    "--environment",
    metavar="ENV",
)
```

Instead of:

```text
--environment ENVIRONMENT
```

the help can show:

```text
--environment ENV
```

---

## 30. `help`

Always provide useful help text:

```python
parser.add_argument(
    "--environment",
    choices=[
        "dev",
        "staging",
        "production",
    ],
    help="Deployment target environment",
)
```

---

## 31. `description`

```python
parser = argparse.ArgumentParser(
    description=(
        "Deploy a microservice "
        "to Kubernetes"
    )
)
```

Explain the tool's primary purpose.

---

## 32. `epilog`

You can add examples:

```python
parser = argparse.ArgumentParser(
    description="Deployment utility",
    epilog=(
        "Example: "
        "python deploy.py "
        "--service payment "
        "--environment staging"
    ),
)
```

---

## 33. `formatter_class`

For better example formatting:

```python
parser = argparse.ArgumentParser(
    formatter_class=(
        argparse.RawDescriptionHelpFormatter
    ),
    description="""
Deployment utility.

Examples:
  deploy.py --service payment --environment staging
  deploy.py --service payment --dry-run
""",
)
```

---

## 34. Argument Groups

Group related options:

```python
network = parser.add_argument_group(
    "Network options"
)

network.add_argument(
    "--region"
)

network.add_argument(
    "--endpoint"
)
```

This improves help readability.

---

## 35. Authentication Options

A CLI might support:

```text
--profile
--region
```

Avoid:

```text
--password
--secret-key
```

when secure credential providers can be used.

---

## 36. AWS Profile

Example:

```python
parser.add_argument(
    "--profile",
    help="AWS CLI profile to use",
)
```

Then:

```bash
python inventory.py \
    --profile production
```

The script can pass the profile to the AWS SDK/session mechanism.

---

## 37. AWS Region

```python
parser.add_argument(
    "--region",
    default="ap-south-1",
)
```

Use:

```bash
python inventory.py \
    --region ap-south-1
```

Do not hard-code production region assumptions if the tool needs to operate across environments.

---

## 38. Kubernetes Namespace

```python
parser.add_argument(
    "-n",
    "--namespace",
    default="default",
)
```

Run:

```bash
python k8s_check.py \
    --namespace production
```

---

## 39. Kubernetes Context

```python
parser.add_argument(
    "--context",
    help="Kubernetes context",
)
```

A production tool should validate the target context before destructive actions.

---

## 40. Cluster Safety

A dangerous script should never assume:

```text
current Kubernetes context
```

is correct.

Example safety flow:

```text
CLI arguments
   |
   v
target context
   |
   v
validate cluster identity
   |
   v
validate environment
   |
   v
perform action
```

---

## 41. Environment Confirmation

For destructive production actions, consider:

```text
--environment production
```

plus:

```text
explicit confirmation
```

Do not make:

```text
production
```

the default target for destructive commands.

---

## 42. Dry Run

Every destructive CLI should strongly consider:

```bash
--dry-run
```

Example:

```python
if args.dry_run:
    logger.info(
        "DRY RUN: no changes will be made"
    )
else:
    perform_changes()
```

---

## 43. Dry Run Must Be Honest

Bad:

```text
--dry-run
```

but the script still:

```text
deletes files
changes resources
applies Terraform
```

A dry run must actually prevent mutations.

---

## 44. Confirmation

For interactive tools:

```python
answer = input(
    "Type DELETE to continue: "
)

if answer != "DELETE":
    raise SystemExit(
        "Operation cancelled"
    )
```

But CI/CD should not depend on interactive prompts.

Use explicit non-interactive flags or pipeline policies.

---

## 45. Non-Interactive Mode

A CI/CD-friendly CLI should work:

```bash
python deploy.py \
    --environment production \
    --service payment \
    --non-interactive
```

No `input()` should block the pipeline.

---

## 46. Interactive vs Automation Mode

Good CLI design supports:

```text
human usage
CI/CD usage
```

but clearly separates the behaviors.

For example:

```text
interactive confirmation
versus
explicit --yes / approved pipeline flag
```

Use secure policy around destructive automation.

---

## 47. `--yes`

You may provide:

```python
parser.add_argument(
    "--yes",
    action="store_true",
)
```

But `--yes` should not bypass:

```text
environment validation
resource allowlists
authorization
safety checks
```

It should only bypass a human confirmation step when policy allows it.

---

## 48. Required Combinations

Sometimes arguments depend on each other.

Example:

```text
--rollback
--version
```

If rollback is requested, version may be required.

Validate after parsing:

```python
if args.rollback and not args.version:
    parser.error(
        "--version is required "
        "with --rollback"
    )
```

---

## 49. Mutually Exclusive Groups

```python
group = parser.add_mutually_exclusive_group(
    required=True
)

group.add_argument(
    "--deploy"
)

group.add_argument(
    "--rollback"
)
```

The user must choose exactly one.

---

## 50. Why Mutually Exclusive Groups Matter

They prevent contradictory commands:

```bash
python tool.py \
    --deploy \
    --rollback
```

The parser rejects this before execution.

---

## 51. Required Mutually Exclusive Group

With:

```python
required=True
```

the user must provide one:

```text
--deploy
```

or:

```text
--rollback
```

---

## 52. Subcommands

DevOps tools often need multiple operations:

```bash
python infra.py plan
python infra.py apply
python infra.py destroy
```

Use:

```python
subparsers = parser.add_subparsers(
    dest="command",
    required=True,
)
```

---

## 53. Basic Subcommand

```python
plan_parser = subparsers.add_parser(
    "plan",
    help="Create infrastructure plan",
)

apply_parser = subparsers.add_parser(
    "apply",
    help="Apply infrastructure changes",
)
```

Then:

```python
if args.command == "plan":
    plan()
elif args.command == "apply":
    apply()
```

---

## 54. Why Subcommands Are Useful

Instead of:

```text
--plan
--apply
--destroy
--rollback
--status
```

you can create:

```text
plan
apply
destroy
rollback
status
```

This resembles tools such as:

```text
kubectl
git
docker
terraform
```

and scales better as the CLI grows.

---

## 55. DevOps CLI Example

Design:

```bash
python platform.py \
    deploy \
    --service payment \
    --environment staging
```

Other commands:

```bash
python platform.py status
python platform.py rollback
python platform.py validate
```

---

## 56. Subcommand Arguments

```python
deploy_parser = subparsers.add_parser(
    "deploy"
)

deploy_parser.add_argument(
    "--service",
    required=True,
)

deploy_parser.add_argument(
    "--environment",
    required=True,
)
```

Now only the `deploy` command needs these arguments.

---

## 57. Subcommand Function

```python
def deploy_command(args):
    logger.info(
        "Deploying %s to %s",
        args.service,
        args.environment,
    )
```

Then:

```python
if args.command == "deploy":
    deploy_command(args)
```

For larger CLIs, use a function mapping or `set_defaults()`.

---

## 58. `set_defaults`

```python
deploy_parser.set_defaults(
    func=deploy_command
)
```

Then:

```python
args = parser.parse_args()
args.func(args)
```

This keeps command dispatch cleaner.

---

## 59. Production CLI Structure

Recommended:

```text
cli.py
 |
 +--> parse arguments
 |
 +--> validate target
 |
 +--> configure logging
 |
 +--> dispatch command
 |
 +--> execute business logic
 |
 +--> return exit code
```

Do not put all business logic directly into argument parsing.

---

## 60. Separate Parsing From Execution

Bad:

```python
if args.environment == "prod":
    # hundreds of lines
```

Better:

```python
def deploy(
    service,
    environment,
    version,
):
    ...
```

Then:

```python
deploy(
    args.service,
    args.environment,
    args.version,
)
```

This makes the logic testable.

---

## 61. `main()`

A common pattern:

```python
def main():
    parser = build_parser()
    args = parser.parse_args()

    return run(args)


if __name__ == "__main__":
    raise SystemExit(
        main()
    )
```

This is a strong pattern for production scripts.

---

## 62. Why Return Exit Codes?

```python
def main():
    if success:
        return 0

    return 1
```

Then:

```python
raise SystemExit(
    main()
)
```

CI/CD can interpret the process result correctly.

---

## 63. Standard Exit Code

Typically:

```text
0 -> success
non-zero -> failure
```

Do not rely on log text to communicate success/failure.

---

## 64. Parser Errors

Use:

```python
parser.error(
    "Invalid deployment configuration"
)
```

This produces a CLI-friendly error and exits with a non-zero status.

---

## 65. Argument Validation vs Business Validation

Argument validation:

```text
Is this an integer?
Is this one of the allowed choices?
Was the required argument supplied?
```

Business validation:

```text
Does the cluster exist?
Is production protected?
Does the service exist?
Is the version deployable?
```

Keep these responsibilities separate.

---

## 66. Configuration Validation

After parsing:

```python
if args.environment == "production":
    validate_production_target()
```

Do not rely solely on `argparse`.

The parser knows syntax.

The application knows operational rules.

---

## 67. Production Target Validation

Before a destructive action:

```text
CLI target
   |
   v
AWS account check
   |
   v
region check
   |
   v
cluster identity check
   |
   v
environment check
   |
   v
resource check
   |
   v
action
```

This is especially important for scripts used across multiple AWS accounts.

---

## 68. Default Values and Production Safety

Bad:

```python
default="production"
```

for a destructive command.

Better:

```text
require explicit environment
```

Example:

```python
parser.add_argument(
    "--environment",
    required=True,
)
```

This forces the operator to identify the target.

---

## 69. Default Region

A read-only inventory script may safely have:

```python
default="ap-south-1"
```

A destructive infrastructure script should consider requiring explicit target information depending on organizational policy.

---

## 70. Input Normalization

You may normalize:

```python
environment = (
    args.environment.lower()
)
```

But be careful with values where case matters.

Use `choices` where possible.

---

## 71. File Arguments

```python
parser.add_argument(
    "--config",
    type=Path,
)
```

Then:

```python
args.config
```

is a `Path` object.

Validate:

```python
if not args.config.exists():
    parser.error(
        "Config file does not exist"
    )
```

---

## 72. Directory Arguments

```python
parser.add_argument(
    "--output-dir",
    type=Path,
)
```

Validate whether the directory:

```text
exists
is writable
is the expected path
```

before writing.

---

## 73. File Safety

Never assume a path is safe because the user typed it.

For destructive tools:

```text
resolve path
validate allowed root
check file type
check environment
```

before deletion.

---

## 74. URL Arguments

```python
parser.add_argument(
    "--endpoint",
    required=True,
)
```

Validate the expected scheme:

```text
https://
```

for production APIs where appropriate.

Do not accept arbitrary endpoints if the script is intended for controlled infrastructure.

---

## 75. Regex Validation

For structured values:

```python
import re

def service_name(value):
    if not re.fullmatch(
        r"[a-z0-9-]+",
        value,
    ):
        raise argparse.ArgumentTypeError(
            "invalid service name"
        )

    return value
```

Then:

```python
parser.add_argument(
    "--service",
    type=service_name,
)
```

---

## 76. Custom `type` Functions

A custom type function should:

```text
parse
validate
return normalized value
```

Example:

```python
def positive_int(value):
    ...
    return number
```

This keeps parser logic clean.

---

## 77. Enum-Like Choices

For fixed values:

```python
choices=[
    "prometheus",
    "grafana",
    "elk",
]
```

This is simpler than custom validation.

---

## 78. Boolean Values

Avoid:

```python
type=bool
```

for command-line booleans.

Why?

```python
bool("false")
```

is:

```text
True
```

because any non-empty string is truthy.

Use:

```python
action="store_true"
```

or a proper parser for explicit true/false values.

---

## 79. Explicit Boolean Option

If a configuration truly requires:

```bash
--enabled true
```

define a custom parser:

```python
def parse_bool(value):
    value = value.lower()

    if value in {"true", "yes", "1"}:
        return True

    if value in {"false", "no", "0"}:
        return False

    raise argparse.ArgumentTypeError(
        "expected true or false"
    )
```

---

## 80. Environment Variables vs CLI

DevOps scripts may receive configuration through:

```text
CLI arguments
environment variables
configuration files
secret managers
```

Do not put secrets in CLI arguments because command lines may be visible through process inspection or CI logs.

---

## 81. Precedence

If a tool supports multiple sources, define precedence explicitly.

Example:

```text
CLI
  >
environment
  >
config file
  >
default
```

A CLI override usually has high precedence.

---

## 82. Secrets

Avoid:

```bash
python deploy.py \
    --password SuperSecret
```

Prefer:

```text
IAM role
AWS profile
secret manager
environment injected securely
```

depending on the platform.

---

## 83. CLI and AWS Credentials

A Python AWS tool should preferably use standard AWS credential resolution:

```text
IAM role
environment
AWS profile
instance/task role
```

rather than requiring:

```bash
--access-key
--secret-key
```

---

## 84. CLI and Kubernetes Credentials

Prefer:

```text
kubeconfig
service account
workload identity
```

rather than passing tokens directly as command-line arguments.

---

## 85. `--verbose` and Security

DEBUG logging can accidentally reveal:

```text
request headers
configuration
tokens
resource objects
```

Therefore:

```text
verbose != permission to log secrets
```

Sensitive fields must remain protected at every log level.

---

## 86. CLI Help and Secrets

Never put secrets in:

```text
help examples
default values
argument descriptions
error messages
```

Help output is often visible in CI logs and support tickets.

---

## 87. CLI for Terraform Automation

Example design:

```bash
python infra.py \
    plan \
    --environment staging
```

Then:

```bash
python infra.py \
    apply \
    --environment staging
```

And:

```bash
python infra.py \
    destroy \
    --environment dev \
    --dry-run
```

---

## 88. CLI for Kubernetes Automation

Example:

```bash
python k8s_tool.py \
    rollout \
    --namespace production \
    --deployment payment \
    --timeout 600
```

The parser validates the input.

The execution layer handles Kubernetes logic.

---

## 89. CLI for AWS Inventory

```bash
python inventory.py \
    --service ec2 \
    --region ap-south-1 \
    --output json
```

Possible output choices:

```text
table
json
csv
```

---

## 90. Output Format Choice

```python
parser.add_argument(
    "--output",
    choices=[
        "table",
        "json",
        "csv",
    ],
    default="table",
)
```

This is useful for both humans and automation.

---

## 91. Machine-Readable Output

CI/CD may prefer:

```bash
--output json
```

while an engineer may prefer:

```bash
--output table
```

Keep logs separate from command output when possible.

---

## 92. stdout vs stderr

A useful CLI convention:

```text
stdout -> normal command output
stderr -> errors/diagnostics
```

This allows:

```bash
python tool.py --output json > result.json
```

without mixing logs into the JSON output.

---

## 93. Logging to stderr

For CLI tools, logging can be configured to stderr:

```python
handler = logging.StreamHandler(
    sys.stderr
)
```

Then structured command output can remain clean on stdout.

---

## 94. CLI + CI/CD

Example:

```bash
python deploy.py \
    --service payment \
    --environment staging \
    --version "$VERSION"
```

CI should be able to:

```text
supply arguments
capture output
interpret exit code
retry where appropriate
```

without human interaction.

---

## 95. CI Environment Variables

CI can provide:

```text
VERSION
CI_COMMIT_SHA
CI_PIPELINE_ID
```

The CLI can combine them with explicit arguments.

Example:

```bash
python deploy.py \
    --service payment \
    --version "$CI_COMMIT_SHA"
```

---

## 96. CLI Idempotency

A DevOps command should ideally be safe to run repeatedly when the operation is intended to be idempotent.

Example:

```bash
python deploy.py \
    --service payment \
    --version abc123
```

If the desired state already exists, the tool should recognize it rather than making unnecessary changes.

---

## 97. Idempotent vs Destructive

Read-only:

```text
status
inventory
validate
plan
```

usually have lower risk.

Mutating:

```text
deploy
apply
update
```

require more validation.

Destructive:

```text
destroy
delete
cleanup
```

require the strongest safeguards.

---

## 98. Command Risk Classification

A useful design:

```text
READ
  status
  inventory

CHANGE
  deploy
  apply

DESTRUCTIVE
  destroy
  delete
```

Apply progressively stronger controls.

---

## 99. Confirmation Policy

For destructive commands:

```text
dev:
  maybe confirmation

staging:
  confirmation

production:
  explicit authorization + safeguards
```

The exact policy should match organizational controls.

---

## 100. Dry Run + Plan

A powerful workflow:

```text
parse
  |
validate
  |
plan
  |
show intended changes
  |
approval
  |
apply
```

This is safer than immediately mutating infrastructure.

---

## 101. CLI Error Messages

Bad:

```text
invalid
```

Better:

```text
error: environment must be one of:
dev, staging, production
```

Best errors explain:

```text
what is wrong
what is expected
how to fix it
```

without exposing sensitive information.

---

## 102. `parser.error()`

Example:

```python
if args.timeout > 3600:
    parser.error(
        "--timeout cannot exceed 3600 seconds"
    )
```

This produces a standard CLI error.

---

## 103. Error Handling Strategy

Recommended:

```text
argparse validation
      |
      v
business validation
      |
      v
try operation
      |
      +--> expected failure -> clear error
      |
      +--> unexpected failure -> traceback/log
      |
      v
correct exit code
```

---

## 104. Logging + CLI Errors

Avoid:

```text
same error printed five times
```

For example:

```text
logger.error(...)
print(...)
parser.error(...)
raise
```

can produce duplicate messages.

Choose one clear reporting path.

---

## 105. CLI Testing

Test:

```text
required argument missing
invalid choice
invalid type
default values
dry run
production safeguards
subcommands
exit codes
help output
```

---

## 106. Test With `subprocess`

You can test the actual CLI:

```python
result = subprocess.run(
    [
        "python",
        "deploy.py",
        "--help",
    ],
    capture_output=True,
    text=True,
)
```

Then inspect:

```python
result.returncode
result.stdout
result.stderr
```

---

## 107. Unit Test Parser

Create:

```python
def build_parser():
    ...
    return parser
```

Then test:

```python
args = parser.parse_args(
    [
        "--environment",
        "staging",
    ]
)
```

This avoids launching the entire application.

---

## 108. Parser Factory

Recommended:

```python
def build_parser():
    parser = argparse.ArgumentParser(
        description="Deployment utility"
    )

    parser.add_argument(
        "--environment",
        required=True,
    )

    return parser
```

Then:

```python
def main():
    parser = build_parser()
    args = parser.parse_args()
```

This is easy to test.

---

## 109. Production CLI Project Structure

Example:

```text
deploy_tool/
├── cli.py
├── commands/
│   ├── deploy.py
│   ├── rollback.py
│   └── status.py
├── services/
│   ├── kubernetes.py
│   └── aws.py
├── logging_config.py
└── tests/
    └── test_cli.py
```

Keep CLI parsing separate from infrastructure logic.

---

## 110. Small Script Structure

For a smaller tool:

```python
import argparse
import logging
import sys


def build_parser():
    parser = argparse.ArgumentParser(
        description="Deployment utility"
    )

    parser.add_argument(
        "--environment",
        required=True,
    )

    parser.add_argument(
        "--dry-run",
        action="store_true",
    )

    return parser


def main():
    parser = build_parser()
    args = parser.parse_args()

    logging.basicConfig(
        level=logging.INFO
    )

    if args.dry_run:
        logging.info(
            "Dry run enabled"
        )

    deploy(
        args.environment
    )

    return 0


if __name__ == "__main__":
    raise SystemExit(
        main()
    )
```

---

## 111. Production Script — Deploy

```python
import argparse
import logging
import sys


logger = logging.getLogger(
    __name__
)


def build_parser():
    parser = argparse.ArgumentParser(
        description=(
            "Deploy a service "
            "to Kubernetes"
        )
    )

    parser.add_argument(
        "--service",
        required=True,
        help="Service name",
    )

    parser.add_argument(
        "--environment",
        required=True,
        choices=[
            "dev",
            "staging",
            "production",
        ],
        help="Target environment",
    )

    parser.add_argument(
        "--version",
        required=True,
        help="Image tag or immutable version",
    )

    parser.add_argument(
        "--dry-run",
        action="store_true",
        help="Show intended action without changing resources",
    )

    parser.add_argument(
        "--timeout",
        type=int,
        default=600,
        help="Rollout timeout in seconds",
    )

    return parser


def deploy(
    service,
    environment,
    version,
    dry_run,
    timeout,
):
    logger.info(
        "Deploying service=%s "
        "environment=%s version=%s",
        service,
        environment,
        version,
    )

    if dry_run:
        logger.info(
            "Dry run enabled; "
            "no changes will be made"
        )
        return

    # Actual Kubernetes deployment logic.


def main():
    parser = build_parser()
    args = parser.parse_args()

    if args.timeout <= 0:
        parser.error(
            "--timeout must be positive"
        )

    try:
        deploy(
            service=args.service,
            environment=args.environment,
            version=args.version,
            dry_run=args.dry_run,
            timeout=args.timeout,
        )

    except Exception:
        logger.exception(
            "Deployment failed"
        )
        return 1

    return 0


if __name__ == "__main__":
    logging.basicConfig(
        level=logging.INFO,
        format=(
            "%(asctime)s "
            "%(levelname)s "
            "%(message)s"
        ),
    )

    sys.exit(main())
```

---

## 112. Production Script — Cleanup

```python
import argparse
import logging
from pathlib import Path


logger = logging.getLogger(
    __name__
)


def build_parser():
    parser = argparse.ArgumentParser(
        description=(
            "Clean old files from "
            "an approved directory"
        )
    )

    parser.add_argument(
        "--path",
        type=Path,
        required=True,
    )

    parser.add_argument(
        "--days",
        type=int,
        required=True,
    )

    parser.add_argument(
        "--dry-run",
        action="store_true",
    )

    return parser
```

A real implementation should additionally validate the allowed root and protected paths before deletion.

---

## 113. Production Script — Kubernetes Status

```python
import argparse


def build_parser():
    parser = argparse.ArgumentParser(
        description=(
            "Check Kubernetes deployment status"
        )
    )

    parser.add_argument(
        "--namespace",
        "-n",
        required=True,
    )

    parser.add_argument(
        "--deployment",
        required=True,
    )

    parser.add_argument(
        "--context",
        help="Kubernetes context",
    )

    parser.add_argument(
        "--timeout",
        type=int,
        default=300,
    )

    return parser
```

---

## 114. Production Script — AWS Inventory

```python
import argparse


def build_parser():
    parser = argparse.ArgumentParser(
        description=(
            "Generate AWS resource inventory"
        )
    )

    parser.add_argument(
        "--region",
        required=True,
    )

    parser.add_argument(
        "--service",
        choices=[
            "ec2",
            "s3",
            "rds",
            "eks",
        ],
        required=True,
    )

    parser.add_argument(
        "--output",
        choices=[
            "table",
            "json",
            "csv",
        ],
        default="table",
    )

    parser.add_argument(
        "--profile",
    )

    return parser
```

---

## 115. Production Script — Subcommands

```python
import argparse


def build_parser():
    parser = argparse.ArgumentParser(
        description="Infrastructure CLI"
    )

    subparsers = parser.add_subparsers(
        dest="command",
        required=True,
    )

    plan = subparsers.add_parser(
        "plan",
        help="Create a plan",
    )

    plan.add_argument(
        "--environment",
        required=True,
    )

    apply = subparsers.add_parser(
        "apply",
        help="Apply changes",
    )

    apply.add_argument(
        "--environment",
        required=True,
    )

    destroy = subparsers.add_parser(
        "destroy",
        help="Destroy resources",
    )

    destroy.add_argument(
        "--environment",
        required=True,
    )

    destroy.add_argument(
        "--dry-run",
        action="store_true",
    )

    return parser
```

---

## 116. Subcommand Dispatch

```python
def main():
    parser = build_parser()
    args = parser.parse_args()

    if args.command == "plan":
        return run_plan(args)

    if args.command == "apply":
        return run_apply(args)

    if args.command == "destroy":
        return run_destroy(args)

    parser.error(
        "Unknown command"
    )
```

With `set_defaults()`, this can be made cleaner for larger tools.

---

## 117. Production CLI — Rollback

Design:

```bash
python platform.py rollback \
    --service payment \
    --environment production \
    --version previous
```

Before execution:

```text
validate service
validate environment
validate version
validate cluster
validate authorization
```

Then:

```text
rollback
wait
verify health
report result
```

---

## 118. Production CLI — Health Check

```bash
python health.py \
    --endpoint https://api.example.com/health \
    --timeout 10 \
    --retries 3
```

The script should:

```text
parse
validate
call endpoint
retry transient failures
report status
return exit code
```

---

## 119. Production CLI — Certificate Check

```bash
python cert_check.py \
    --host api.example.com \
    --port 443 \
    --warning-days 30
```

The parser handles:

```text
host
port
warning threshold
```

The execution layer handles TLS/certificate inspection.

---

## 120. Production CLI — Log Analysis

```bash
python logs.py \
    --service payment \
    --since 15m \
    --level ERROR
```

Possible parser fields:

```text
service
namespace
time window
level
output
```

For complex duration formats such as `15m`, use a custom parser rather than assuming `argparse` can interpret it automatically.

---

## 121. Parse Duration Argument

```python
def duration(value):
    units = {
        "s": 1,
        "m": 60,
        "h": 3600,
    }

    if not value:
        raise argparse.ArgumentTypeError(
            "duration cannot be empty"
        )

    suffix = value[-1]
    number = value[:-1]

    if suffix not in units:
        raise argparse.ArgumentTypeError(
            "use s, m, or h"
        )

    try:
        number = float(number)
    except ValueError:
        raise argparse.ArgumentTypeError(
            "invalid duration"
        )

    if number <= 0:
        raise argparse.ArgumentTypeError(
            "duration must be positive"
        )

    return number * units[suffix]
```

Then:

```python
parser.add_argument(
    "--since",
    type=duration,
)
```

---

## 122. CLI for Monitoring

Example:

```bash
python monitor.py \
    --service payment \
    --threshold 5 \
    --timeout 60
```

The CLI should define:

```text
what service
what threshold
how long to wait
what constitutes failure
```

---

## 123. CLI for Security Scanning

```bash
python security.py scan \
    --image payment:latest \
    --severity high \
    --fail-on-findings
```

Potential flags:

```text
--image
--severity
--output
--fail-on-findings
```

Avoid accepting registry credentials as CLI parameters.

---

## 124. CLI for Terraform

```bash
python terraform_tool.py plan \
    --environment staging

python terraform_tool.py apply \
    --environment staging \
    --approve
```

A safer design may require:

```text
plan
review
explicit approval
apply
```

instead of making apply automatic.

---

## 125. CLI for GitOps

```bash
python gitops.py sync \
    --application payment \
    --environment staging
```

Possible checks:

```text
Git revision
ArgoCD application
target cluster
desired revision
health status
```

---

## 126. CLI for Image Promotion

```bash
python promote.py \
    --image payment \
    --source staging \
    --target production \
    --digest sha256:...
```

Validate:

```text
digest exists
source environment
target environment
approval
security scan
```

---

## 127. CLI and Image Tags

Avoid relying only on:

```text
latest
```

for production deployment.

Prefer immutable:

```text
commit SHA
version
digest
```

The CLI can require an explicit immutable identifier.

---

## 128. CLI for EKS Troubleshooting

Example:

```bash
python eks_debug.py pod \
    --namespace production \
    --pod payment-abc123
```

Possible commands:

```text
pod
node
deployment
service
ingress
```

Each subcommand can gather targeted diagnostics.

---

## 129. CLI Troubleshooting Workflow

```text
command
  |
  v
validate target
  |
  v
collect diagnostics
  |
  +--> pod status
  +--> events
  +--> logs
  +--> resource usage
  |
  v
summarize
  |
  v
exit code
```

Do not make troubleshooting tools destructive by default.

---

## 130. CLI and Production Incident Response

A useful incident CLI may support:

```bash
python incident.py collect \
    --service payment \
    --namespace production \
    --window 15m
```

Collect:

```text
logs
events
deployment status
pod status
recent changes
```

and produce a report.

---

## 131. CLI Output and Automation

For humans:

```text
Deployment successful
Service: payment
Version: abc123
Duration: 84s
```

For machines:

```json
{
  "status": "success",
  "service": "payment",
  "version": "abc123",
  "duration_seconds": 84
}
```

Supporting both is valuable.

---

## 132. Output Must Not Mix With Logs

If stdout is JSON:

```bash
python tool.py --output json > result.json
```

then logs should go to stderr.

Otherwise:

```text
INFO...
{"status": "success"}
```

would not be valid JSON.

---

## 133. `--quiet`

A CLI may provide:

```python
parser.add_argument(
    "--quiet",
    action="store_true",
)
```

Then reduce nonessential output.

Do not suppress errors that automation needs to diagnose.

---

## 134. `--debug`

A CLI may provide:

```python
parser.add_argument(
    "--debug",
    action="store_true",
)
```

Then configure:

```python
logging.DEBUG
```

But still protect secrets.

---

## 135. `--version`

Many CLI tools should provide version information.

A simple implementation:

```python
parser.add_argument(
    "--version",
    action="version",
    version="deploy-tool 1.0.0",
)
```

Be careful not to confuse this with a deployment target's `--version` argument.

Use distinct names where necessary.

---

## 136. Version Flag Naming

If the tool needs:

```text
--version
```

for the application version, use:

```text
--release
```

or:

```text
--image-tag
```

for deployment target version if appropriate.

Clear naming prevents ambiguity.

---

## 137. CLI Compatibility

Once a CLI is used in CI/CD, changing:

```text
argument names
defaults
output format
exit codes
```

can break pipelines.

Treat the CLI as an API.

---

## 138. Backward Compatibility

If changing:

```text
--environment
```

to:

```text
--env
```

consider supporting both temporarily:

```python
parser.add_argument(
    "--environment",
    "--env",
)
```

Then deprecate the old form deliberately.

---

## 139. Deprecation

When removing an option:

```text
old option
   |
   v
warning
   |
   v
migration period
   |
   v
remove
```

Do not silently change behavior in production automation.

---

## 140. CLI Documentation

Document:

```text
purpose
installation
usage
commands
arguments
examples
exit codes
safety
environment requirements
```

The `--help` output is only one layer of documentation.

---

## 141. Shell Completion

Large CLIs may eventually support shell completion.

This improves:

```text
command discovery
argument discovery
allowed values
```

but is an enhancement rather than a requirement for basic Python automation.

---

## 142. Configuration Files + Argparse

A CLI may accept:

```bash
python deploy.py \
    --config deploy.yaml
```

Then:

```text
config file
     |
     v
defaults
     |
     v
CLI overrides
```

Make precedence explicit.

---

## 143. `fromfile_prefix_chars`

`argparse` can support arguments from files in certain designs.

Example:

```python
parser = argparse.ArgumentParser(
    fromfile_prefix_chars="@"
)
```

Then:

```bash
python tool.py @args.txt
```

Use carefully and understand how the file is parsed.

For modern DevOps configuration, YAML/JSON files are often clearer when the configuration is complex.

---

## 144. Argparse + Environment Variables

`argparse` does not automatically manage environment-variable precedence.

Implement explicitly:

```python
default = os.getenv(
    "DEPLOY_ENV"
)
```

Then:

```python
parser.add_argument(
    "--environment",
    default=default,
)
```

If CLI is supplied, it overrides the default.

---

## 145. CLI + Config Precedence

A robust configuration flow:

```text
built-in default
       |
       v
config file
       |
       v
environment variable
       |
       v
CLI argument
```

The highest-precedence source should be documented.

---

## 146. Avoid Hidden Defaults

Bad:

```text
environment comes from current shell
```

without telling the user.

Better:

```text
--environment is explicit
```

or show the resolved configuration before executing a high-risk operation.

---

## 147. Resolved Configuration

Before a destructive operation:

```text
Target:
  environment: production
  cluster: prod-eks
  region: ap-south-1
  service: payment
  version: abc123
```

Then:

```text
validate
execute
```

This reduces operator mistakes.

---

## 148. CLI Confirmation Summary

For production:

```text
You are about to deploy:

service: payment
environment: production
cluster: prod-eks
version: abc123

Continue? [y/N]
```

For CI:

```text
--non-interactive
```

with explicit authorization controls.

---

## 149. Do Not Use Interactive Prompts in CI

A pipeline can hang indefinitely if:

```python
input()
```

is called.

Use:

```text
non-interactive mode
explicit flags
pipeline approval
```

instead.

---

## 150. CLI Signal Handling

Long-running CLI tools may receive:

```text
SIGTERM
SIGINT
```

For example:

```text
Ctrl+C
CI job cancellation
Kubernetes termination
```

Handle cleanup appropriately without hiding the termination signal.

---

## 151. Timeout + CLI

A user may provide:

```bash
--timeout 600
```

Use it to define a monotonic deadline.

Do not implement:

```python
time.sleep(600)
```

and assume success.

---

## 152. Retry + CLI

Expose safe controls:

```bash
--retries 3
--timeout 60
```

but do not allow users to accidentally configure:

```text
millions of retries
```

Validate maximum values.

---

## 153. Maximum Retry Validation

```python
def bounded_retries(value):
    value = int(value)

    if not 0 <= value <= 10:
        raise argparse.ArgumentTypeError(
            "retries must be 0-10"
        )

    return value
```

Production tools should have sane upper bounds.

---

## 154. Maximum Timeout Validation

```python
def timeout_value(value):
    value = int(value)

    if not 1 <= value <= 3600:
        raise argparse.ArgumentTypeError(
            "timeout must be 1-3600 seconds"
        )

    return value
```

Avoid unlimited waits.

---

## 155. CLI for Cleanup

Good:

```bash
python cleanup.py \
    --environment staging \
    --older-than 30d \
    --dry-run
```

Then:

```text
review
remove --dry-run
execute
```

Never default a destructive cleanup tool to production.

---

## 156. Parse Human-Friendly Age

A cleanup tool might support:

```text
7d
24h
30m
```

Convert to seconds or `timedelta` using a custom parser.

Validate:

```text
positive
reasonable maximum
supported units
```

---

## 157. CLI for Log Analysis

```bash
python analyze.py \
    --service payment \
    --level ERROR \
    --since 30m \
    --output json
```

This makes the tool reusable for:

```text
manual investigation
CI
incident automation
scheduled reports
```

---

## 158. CLI for Monitoring Checks

Example:

```bash
python check.py \
    --service payment \
    --threshold 5 \
    --timeout 30
```

Return:

```text
0 -> healthy
1 -> unhealthy
2 -> invalid configuration
```

Document custom exit codes if used.

---

## 159. Exit Codes for Monitoring

Nagios-style or custom automation may use:

```text
0 OK
1 WARNING
2 CRITICAL
3 UNKNOWN
```

If adopting such a convention, document it and remain consistent.

---

## 160. CLI as a Monitoring Plugin

Example:

```bash
python check_service.py \
    --endpoint https://example.com
```

Possible result:

```text
OK - endpoint healthy
```

or:

```text
CRITICAL - endpoint unavailable
```

Exit code communicates machine-readable state.

---

## 161. CLI for Certificate Monitoring

```bash
python cert_check.py \
    --host api.example.com \
    --warning 30 \
    --critical 7
```

Possible:

```text
0 -> >30 days
1 -> <=30 days
2 -> <=7 days/expired
```

This pattern is useful for monitoring integration.

---

## 162. CLI for Disk Monitoring

```bash
python disk_check.py \
    --path / \
    --warning 80 \
    --critical 90
```

Validate thresholds:

```text
0 <= warning < critical <= 100
```

---

## 163. Cross-Argument Validation

```python
if args.warning >= args.critical:
    parser.error(
        "--warning must be less than "
        "--critical"
    )
```

This is business validation that happens immediately after parsing.

---

## 164. CLI for Kubernetes Resource Checks

```bash
python check_k8s.py \
    --namespace production \
    --deployment payment \
    --warning 80 \
    --critical 90
```

The script can check:

```text
ready replicas
restart count
resource usage
conditions
```

and return an appropriate exit code.

---

## 165. CLI Safety Checklist

Before destructive operations:

```text
1. Require explicit target.
2. Validate environment.
3. Validate cluster/account.
4. Validate resource.
5. Validate requested version.
6. Support dry run.
7. Show resolved target.
8. Require approval where needed.
9. Prevent unsafe defaults.
10. Return correct exit code.
```

---

## 166. Common Argparse Mistakes

Avoid:

```text
1. using bool as a type
2. no required validation
3. dangerous production defaults
4. secrets in arguments
5. no help text
6. no choices for fixed values
7. mixing parser and business logic
8. interactive prompts in CI
9. no exit code handling
10. accepting contradictory flags
11. unlimited timeout
12. unlimited retries
13. unclear output
14. breaking CLI compatibility
15. hiding important defaults
```

---

## 167. `argparse` Cheat Sheet

Create parser:

```python
argparse.ArgumentParser()
```

Add argument:

```python
parser.add_argument()
```

Parse:

```python
parser.parse_args()
```

Required:

```python
required=True
```

Default:

```python
default=value
```

Type:

```python
type=int
```

Choices:

```python
choices=[...]
```

Boolean flag:

```python
action="store_true"
```

Multiple values:

```python
nargs="+"
```

Repeated option:

```python
action="append"
```

Mutually exclusive:

```python
add_mutually_exclusive_group()
```

Subcommands:

```python
add_subparsers()
```

Parser error:

```python
parser.error(...)
```

Version:

```python
action="version"
```

---

## 168. DevOps CLI Mental Model

```text
                 CLI
                  |
                  v
             argparse
                  |
        +---------+---------+
        |         |         |
      syntax    type      choices
        |         |         |
        +---------+---------+
                  |
                  v
          business validation
                  |
                  v
          safety validation
                  |
                  v
             execution
                  |
                  v
          logging/output
                  |
                  v
             exit code
```

---

## 169. Production CLI Architecture

```text
                 User / CI
                     |
                     v
              command arguments
                     |
                     v
                 argparse
                     |
                     v
              parsed arguments
                     |
          +----------+----------+
          |                     |
          v                     v
      validation             logging
          |
          v
      safety checks
          |
          v
      business logic
          |
    +-----+-----+-----+
    |           |     |
  AWS       Kubernetes  Terraform
    |           |     |
    +-----+-----+-----+
          |
          v
        result
          |
    +-----+-----+
    |           |
 stdout       stderr
    |           |
    +-----+-----+
          |
          v
      exit code
```

---

## 170. Interview — What Is `argparse`?

### Answer

> `argparse` is Python's standard-library module for building command-line interfaces. It handles argument parsing, help output, types, defaults, choices, validation, flags, and subcommands. I use it to turn Python automation scripts into reusable DevOps CLI tools.

---

## 171. Interview — Why Use `argparse` Instead of `sys.argv`?

### Answer

> `sys.argv` works for very simple scripts, but `argparse` provides structured parsing, automatic help, type conversion, required arguments, choices, mutually exclusive options, subcommands, and clear error handling. That makes it much better for production automation.

---

## 172. Interview — How Do You Make a Boolean CLI Flag?

### Answer

> I normally use `action="store_true"` for flags such as `--dry-run`, `--verbose`, or `--force`. I avoid using `type=bool` because strings such as `"false"` are still truthy in Python.

---

## 173. Interview — How Do You Validate CLI Values?

### Answer

> I use `choices` for fixed sets, `type` for basic conversion, custom type functions for constraints such as positive integers, and post-parsing business validation for rules involving multiple arguments or external resources.

---

## 174. Interview — How Do You Build a Production DevOps CLI?

### Answer

> I separate argument parsing, validation, business logic, logging, and exit-code handling. I provide clear help, safe defaults, dry-run support for destructive actions, explicit production targeting, non-interactive CI support, and structured output where automation needs it.

---

## 175. Interview — How Do You Handle Production Safety?

### Answer

> I avoid production as an implicit default for destructive commands. I require explicit environment and resource targeting, validate the AWS account or Kubernetes context, provide dry-run support, show the resolved target, and require an appropriate approval mechanism.

---

## 176. Interview — How Do You Integrate a Python CLI With CI/CD?

### Answer

> CI passes explicit arguments and environment variables, the script performs validation and automation, logs useful progress, and returns a correct process exit code. I avoid interactive prompts so the pipeline cannot hang waiting for input.

---

## 177. Interview — What Are Subcommands?

### Answer

> Subcommands allow one CLI to expose related operations such as `plan`, `apply`, `destroy`, `status`, and `rollback`. I use `add_subparsers()` because it keeps each command's arguments and help separate and makes the CLI scale better.

---

## 178. Interview — How Do You Handle Secrets in CLI Tools?

### Answer

> I avoid passing secrets as command-line arguments because process arguments can be exposed. For AWS I prefer IAM roles and standard credential resolution. For other systems I use approved secret-management mechanisms or secure environment injection.

---

## 179. Scenario — Deployment Script Accidentally Targets Production

Investigate:

```text
default environment
current kube context
AWS account
region
config precedence
```

Fix:

```text
explicit target
account validation
cluster identity validation
dry run
production safeguards
```

---

## 180. Scenario — CI Pipeline Hangs

Check for:

```text
input()
interactive confirmation
password prompt
authentication prompt
infinite polling
missing timeout
```

Use:

```text
non-interactive mode
explicit credentials mechanism
bounded deadlines
```

---

## 181. Scenario — User Passes `--timeout abc`

`argparse` should reject it with:

```python
type=int
```

before the deployment begins.

This is an example of input validation happening at the CLI boundary.

---

## 182. Scenario — User Passes `--timeout -1`

Basic `type=int` accepts:

```text
-1
```

Use a custom validator:

```python
positive_int
```

or post-parse validation.

---

## 183. Scenario — User Passes `--environment production --dry-run`

This should be allowed because dry-run does not mutate resources.

The tool should still validate:

```text
production account
cluster
service
configuration
```

if those values are needed to produce a meaningful plan.

---

## 184. Scenario — User Passes Both Deploy and Rollback

Use:

```python
add_mutually_exclusive_group(
    required=True
)
```

or subcommands:

```text
deploy
rollback
```

Do not allow contradictory operations to reach business logic.

---

## 185. Scenario — Cleanup Tool Deletes Wrong Files

CLI validation alone is not enough.

Use:

```text
explicit path
allowed root
resolved path
environment validation
age threshold
dry run
protected paths
```

The execution layer must enforce safety.

---

## 186. Scenario — CI Reports Success Despite Python Error

Check:

```text
exception handling
return value
sys.exit()
shell command behavior
pipeline error handling
```

Correct:

```python
raise SystemExit(
    main()
)
```

with:

```text
0 -> success
non-zero -> failure
```

---

## 187. Scenario — CLI Output Must Be JSON

Do not mix:

```text
INFO logs
```

with:

```json
{"status": "success"}
```

on stdout.

Use:

```text
stdout -> JSON result
stderr -> logs
```

This makes shell automation reliable.

---

## 188. Scenario — Need Multiple Services

Use:

```python
nargs="+"
```

or:

```python
action="append"
```

Choose based on desired command syntax.

Example:

```bash
--service user payment cart
```

or:

```bash
--service user \
--service payment \
--service cart
```

---

## 189. Scenario — Need Plan/Apply/Destroy

Use subcommands:

```bash
tool plan
tool apply
tool destroy
```

Each command gets its own arguments and safety policy.

This is cleaner than dozens of boolean flags.

---

## 190. Scenario — Need Different Output Formats

Use:

```python
choices=[
    "table",
    "json",
    "csv",
]
```

Then keep:

```text
logs
```

separate from:

```text
command output
```

---

## 191. Scenario — Need `--since 15m`

Use a custom argument parser:

```text
15m
1h
30s
```

Convert to seconds or `timedelta`.

Do not rely on `type=int` for human-friendly duration syntax.

---

## 192. Scenario — Need a Production Approval

Separate:

```text
parse
validate
plan
approval
apply
verify
```

Do not combine all of these into one giant function.

---

## 193. Scenario — CLI Has 20 Arguments

Consider:

```text
subcommands
argument groups
configuration file
environment variables
```

Do not create an unreadable command with dozens of unrelated flags.

---

## 194. Scenario — CLI Is Used by Other Teams

Treat the CLI as an API.

Protect:

```text
argument compatibility
output format
exit codes
defaults
help text
```

Introduce breaking changes deliberately.

---

## 195. Scenario — Need a Safe Destroy Command

Design:

```bash
python infra.py destroy \
    --environment staging \
    --dry-run
```

Then:

```text
validate
show resources
review
execute
```

For production, require stronger authorization and safeguards.

---

## 196. Production Best Practices

```text
1. Use argparse for CLI parsing.
2. Provide clear --help.
3. Use explicit required targets for risky operations.
4. Use choices for fixed values.
5. Use custom types for numeric constraints.
6. Separate parser from business logic.
7. Use subcommands for large tools.
8. Support --dry-run.
9. Avoid secrets in CLI arguments.
10. Support non-interactive CI usage.
11. Return correct exit codes.
12. Keep stdout machine-readable when required.
13. Send diagnostics to stderr.
14. Validate AWS/Kubernetes targets.
15. Use safe defaults.
16. Bound retries and timeouts.
17. Keep CLI compatibility stable.
18. Test invalid arguments.
19. Log operational context.
20. Make destructive actions difficult to perform accidentally.
```

---

## 197. Final DevOps CLI Checklist

Before shipping a Python automation tool, ask:

```text
Can the user understand --help?
Can invalid input be rejected early?
Are dangerous defaults avoided?
Can CI run without prompts?
Are secrets protected?
Is dry-run available?
Are production targets explicit?
Are AWS/Kubernetes targets validated?
Are retries/timeouts bounded?
Are logs useful?
Is stdout clean for machine-readable output?
Are exit codes correct?
Can the parser be unit tested?
Can the CLI evolve without breaking pipelines?
```

If the answer is yes, the script is much closer to production quality.

---

## 198. Final Mental Model

```text
              PYTHON DEVOPS CLI

                   command
                      |
                      v
                 argparse
                      |
             +--------+--------+
             |        |        |
           type     choices   flags
             |        |        |
             +--------+--------+
                      |
                      v
               parsed arguments
                      |
                      v
             business validation
                      |
                      v
               safety checks
                      |
                      v
            AWS / Kubernetes /
          Terraform / APIs / Linux
                      |
                      v
                  result
                      |
             +--------+--------+
             |                 |
           logs              output
             |                 |
          stderr             stdout
             |                 |
             +--------+--------+
                      |
                      v
                  exit code
```

Remember:

```text
argparse
    =
CLI interface

validation
    =
input safety

business logic
    =
actual DevOps work

logging
    =
operational evidence

exit code
    =
automation result
```

A good DevOps Python script is not just code that works once.

It should be a **reusable, safe, testable, automation-friendly command-line tool**.

---

## 199. Next File

```text
09-Virtual-Environments.md
```

The next topic will cover Python environments for DevOps:

```text
venv
pip
requirements.txt
package isolation
dependency versions
pip freeze
dependency conflicts
system Python
CI/CD environments
Docker environments
AWS/Kubernetes automation dependencies
reproducible builds
security
dependency scanning
production practices
real DevOps scripts
interview questions
scenario-based questions
```
