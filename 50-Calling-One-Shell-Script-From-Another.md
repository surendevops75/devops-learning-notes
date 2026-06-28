# Calling One Shell Script From Another

## Introduction

As shell scripts grow larger, it is common to split functionality into multiple scripts.

Example:

```text
main.sh
common.sh
mysql.sh
frontend.sh
```

Instead of writing everything in a single file, we can call one script from another.

---

# Why Call Another Script?

Benefits:

- Reusability
- Maintainability
- Modular Design
- DRY Principle

---

# DRY Principle

DRY means:

```text
Don't Repeat Yourself
```

Avoid duplicating code.

Instead of copying common functions into multiple scripts, keep them in one file and reuse them.

---

# Ways to Call Another Script

There are two common methods:

```text
1. sh <script-name>

2. source <script-name>
```

---

# Method 1: Using sh

Syntax:

```bash
sh script2.sh
```

Example:

```bash
sh common.sh
```

---

# How It Works

```text
script1
    │
    ▼
Creates New Process
    │
    ▼
Runs script2
    │
    ▼
Returns Back
```

---

# Separate Process

When using:

```bash
sh script2.sh
```

script2 runs in a completely separate process.

---

# Example

## script1.sh

```bash
#!/bin/bash

NAME="DevOps"

sh script2.sh
```

---

## script2.sh

```bash
#!/bin/bash

echo $NAME
```

Output:

```text

```

No output because:

```text
NAME belongs to script1 process.
```

script2 cannot access it.

---

# Environment Isolation

Using:

```bash
sh script2.sh
```

means:

```text
script1 variables
≠
script2 variables
```

---

# Failure Behavior

Suppose:

## script2.sh

```bash
#!/bin/bash

exit 1
```

---

## script1.sh

```bash
#!/bin/bash

sh script2.sh

echo "Script1 Continues"
```

Output:

```text
Script1 Continues
```

Because:

```text
script2 failure does not directly terminate script1.
```

They run in different processes.

---

# Method 2: Using source

Syntax:

```bash
source script2.sh
```

Alternative:

```bash
. script2.sh
```

Both are equivalent.

---

# How It Works

```text
script1
    │
    ▼
Loads script2
    │
    ▼
Runs In Same Process
```

---

# Same Process

When using:

```bash
source script2.sh
```

script2 executes inside script1's shell process.

---

# Example

## script1.sh

```bash
#!/bin/bash

NAME="DevOps"

source script2.sh
```

---

## script2.sh

```bash
#!/bin/bash

echo $NAME
```

Output:

```text
DevOps
```

Because:

```text
Both scripts share the same environment.
```

---

# Access to Variables

Using source:

```bash
source script2.sh
```

allows access to:

- Variables
- Functions
- Environment Variables

from script1.

---

# Function Reuse Example

## common.sh

```bash
#!/bin/bash

GREET() {
    echo "Hello DevOps"
}
```

---

## main.sh

```bash
#!/bin/bash

source common.sh

GREET
```

Output:

```text
Hello DevOps
```

---

# Why source is Common in DevOps

Many teams create:

```text
common.sh
```

containing:

- Functions
- Colors
- Logging Logic
- Validation Functions

and then source it in all scripts.

---

# Failure Behavior with source

Suppose:

## script2.sh

```bash
#!/bin/bash

exit 1
```

---

## script1.sh

```bash
#!/bin/bash

source script2.sh

echo "Hello"
```

Output:

```text
Script exits
```

Because:

```text
Both scripts run in same process.
```

Failure affects script1.

---

# Comparison

| Feature | sh script.sh | source script.sh |
|----------|-------------|------------------|
| Process | New Process | Same Process |
| Variable Access | No | Yes |
| Function Access | No | Yes |
| Environment Sharing | No | Yes |
| Failure Impact | Limited | Affects Parent Script |
| Common DevOps Usage | Less | More |

---

# Real DevOps Example

## common.sh

```bash
RED="\e[31m"
GREEN="\e[32m"

VALIDATE(){

    if [ $1 -eq 0 ]
    then
        echo "SUCCESS"
    else
        echo "FAILURE"
        exit 1
    fi

}
```

---

## mysql.sh

```bash
#!/bin/bash

source common.sh

dnf install mysql -y

VALIDATE $? "MySQL Installation"
```

---

## frontend.sh

```bash
#!/bin/bash

source common.sh

dnf install nginx -y

VALIDATE $? "Nginx Installation"
```

---

# Best Practices

## Use source for Shared Logic

Example:

```text
Functions
Variables
Colors
Logging
```

---

## Use sh for Independent Execution

When script2 should run separately.

---

## Avoid Duplicate Code

Store common code in:

```text
common.sh
```

and source it.

---

# Frequently Asked Interview Questions

## How Can One Shell Script Call Another?

```bash
sh script.sh
```

or

```bash
source script.sh
```

---

## Difference Between sh and source?

```text
sh      → New Process

source  → Same Process
```

---

## Can script2 Access script1 Variables Using sh?

No.

---

## Can script2 Access script1 Variables Using source?

Yes.

---

## Which Method is Common in DevOps?

```bash
source
```

because it allows sharing functions and variables.

---

# Key Takeaways

- Shell scripts can call other scripts using `sh` or `source`.
- `sh` creates a new process.
- `source` runs in the same process.
- Variables and functions are not shared with `sh`.
- Variables and functions are shared with `source`.
- `source` is commonly used for reusable DevOps utility scripts.
- Shared files such as `common.sh` improve maintainability and support the DRY principle.