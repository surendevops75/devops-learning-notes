# Terraform Functions

## Introduction

Terraform provides built-in functions that help manipulate:

```text
Strings

Lists

Maps

Numbers

Objects
```

Functions make Terraform code:

```text
Reusable

Dynamic

Readable

Maintainable
```

Without functions, Terraform configurations would require a lot of repetitive code and hardcoded values.

---

# What is a Function?

A function takes:

```text
Input
   ↓
Processing
   ↓
Output
```

Example:

```hcl
upper("devops")
```

Output:

```text
DEVOPS
```

---

# Why Do We Need Functions?

Functions help:

```text
Transform Data

Combine Data

Filter Data

Generate Dynamic Values
```

Examples:

```text
Convert Text to Uppercase

Merge Maps

Count List Elements

Split Strings

Join Strings
```

---

# Function Syntax

```hcl
function_name(arguments)
```

Example:

```hcl
length(var.instances)
```

---

# Commonly Used Functions

```text
length()

merge()

lookup()

join()

split()

upper()

lower()

format()

toset()
```

---

# length()

Returns the number of elements.

---

## Example with List

```hcl
length([
  "frontend",
  "catalogue",
  "user"
])
```

Output:

```text
3
```

---

## Production Example

```hcl
count = length(var.instances)
```

Terraform creates resources based on the number of list items.

---

# upper()

Converts text to uppercase.

Example:

```hcl
upper("devops")
```

Output:

```text
DEVOPS
```

---

# lower()

Converts text to lowercase.

Example:

```hcl
lower("DEVOPS")
```

Output:

```text
devops
```

---

# Production Example

Standardizing environment names:

```hcl
lower(var.environment)
```

---

# join()

Combines multiple values into a single string.

Syntax:

```hcl
join(separator, list)
```

---

Example

```hcl
join("-", ["dev", "frontend"])
```

Output:

```text
dev-frontend
```

---

# Production Example

Generating tags:

```hcl
join("-", [
  var.environment,
  var.component
])
```

Output:

```text
prod-catalogue
```

---

# split()

Splits a string into a list.

Syntax:

```hcl
split(separator, string)
```

---

Example

```hcl
split(",", "frontend,catalogue,user")
```

Output:

```hcl
[
  "frontend",
  "catalogue",
  "user"
]
```

---

# Production Example

Converting configuration values into lists.

---

# lookup()

Retrieves a value from a map.

Syntax:

```hcl
lookup(map, key, default)
```

---

Example

```hcl
lookup(
  {
    dev  = "t3.micro"
    prod = "t3.large"
  },
  "prod",
  "t3.small"
)
```

Output:

```text
t3.large
```

---

# Default Value Example

```hcl
lookup(
  var.instance_types,
  "qa",
  "t3.micro"
)
```

If:

```text
qa
```

does not exist,

Output:

```text
t3.micro
```

---

# Production Example

Selecting instance types based on environment.

---

# merge()

One of the most commonly used Terraform functions.

Combines multiple maps into one map.

---

# Why Do We Need merge()?

Suppose we have:

Common Tags

```hcl
{
  Project = "Roboshop"

  Owner = "DevOps"
}
```

Environment Tags

```hcl
{
  Environment = "dev"
}
```

Instead of duplicating tags, we merge them.

---

# Syntax

```hcl
merge(map1, map2)
```

---

# Example

```hcl
merge(

  {
    a = "b"

    c = "d"
  },

  {
    e = "f"

    c = "z"
  }

)
```

---

Output

```hcl
{
  a = "b"

  c = "z"

  e = "f"
}
```

---

# Important Rule

If duplicate keys exist:

```text
Last Map Wins
```

---

Example

Map 1:

```hcl
{
  Environment = "dev"
}
```

Map 2:

```hcl
{
  Environment = "prod"
}
```

Output:

```hcl
{
  Environment = "prod"
}
```

---

# Production Example

Common Tags:

```hcl
locals {

  common_tags = {

    Project = "Roboshop"

    Team = "DevOps"
  }

}
```

---

Environment Tags:

```hcl
{
  Environment = "prod"
}
```

---

Merge:

```hcl
tags = merge(

  local.common_tags,

  {
    Environment = "prod"
  }

)
```

---

Final Tags

```hcl
{
  Project     = "Roboshop"

  Team        = "DevOps"

  Environment = "prod"
}
```

---

# toset()

Converts a list into a set.

Example:

```hcl
toset([
  "frontend",
  "catalogue",
  "user"
])
```

---

Used With

```hcl
for_each
```

---

Example

```hcl
for_each = toset(var.instances)
```

---

# format()

Formats strings dynamically.

Example:

```hcl
format(
  "%s-%s",
  "dev",
  "frontend"
)
```

Output:

```text
dev-frontend
```

---

# Chaining Functions

Functions can be combined.

Example:

```hcl
upper(
  join("-", [
    "dev",
    "frontend"
  ])
)
```

Output:

```text
DEV-FRONTEND
```

---

# Common Production Use Cases

## Tags

```hcl
merge()
```

---

## Dynamic Names

```hcl
join()

format()
```

---

## Environment Based Values

```hcl
lookup()
```

---

## Resource Count

```hcl
length()
```

---

## for_each Iteration

```hcl
toset()
```

---

# Common Mistakes

## Forgetting Duplicate Key Override

Example:

```hcl
merge(map1, map2)
```

Last value wins.

---

## Using split() Incorrectly

Wrong separator causes unexpected results.

---

## Overusing Functions

Complex nested functions reduce readability.

---

# Best Practices

## Keep Functions Readable

Avoid deeply nested expressions.

---

## Use merge() for Tags

Very common in enterprise projects.

---

## Use lookup() Instead of Large Conditions

Cleaner code.

---

## Use length() for Dynamic Counts

Avoid hardcoding numbers.

---

# Frequently Asked Interview Questions

## What Are Terraform Functions?

### Short Answer

Terraform functions are built-in utilities used to manipulate and transform data.

### Detailed Explanation

Functions allow Terraform configurations to work dynamically with strings, lists, maps, and objects. They help reduce duplication and improve code readability.

### Production Example

Using `merge()` to combine common tags and environment-specific tags.

### Follow-Up Questions

- Can functions be nested?
- What are commonly used functions?

---

## What Does merge() Do?

### Short Answer

merge() combines multiple maps into a single map.

### Detailed Explanation

When duplicate keys exist, the value from the last map overrides previous values. This behavior is heavily used for tag management.

### Production Example

Combining common tags with environment-specific tags during EC2 creation.

### Follow-Up Questions

- What happens when keys are duplicated?
- Why is merge() useful for tagging?

---

## What Does lookup() Do?

### Short Answer

lookup() retrieves a value from a map.

### Detailed Explanation

It allows Terraform to safely fetch values and optionally return a default value when the key does not exist.

### Production Example

Selecting instance types based on environment.

### Follow-Up Questions

- What happens if the key is missing?
- Why use lookup instead of conditionals?

---

## What Does length() Do?

### Short Answer

length() returns the number of elements in a list, map, or string.

### Detailed Explanation

It is commonly used with count to dynamically create resources.

### Production Example

Creating EC2 instances based on the number of application components.

### Follow-Up Questions

- Can length() work on strings?
- How is it used with count?

---

## Why Is merge() Commonly Used in Production?

### Short Answer

To combine common and environment-specific tags.

### Detailed Explanation

Enterprise environments enforce tagging standards. merge() allows centralized management of tags while still supporting environment-specific customizations.

### Production Example

Applying Project, Team, CostCenter, and Environment tags to all resources.

### Follow-Up Questions

- How do you manage tags in Terraform?
- What happens when tag keys conflict?

---

# Key Takeaways

- Functions make Terraform configurations dynamic and reusable.
- Common functions include:
  
  ```text
  length()
  merge()
  lookup()
  join()
  split()
  upper()
  lower()
  toset()
  format()
  ```

- merge() is heavily used for tagging strategies.
- lookup() simplifies environment-specific configurations.
- length() is commonly used with count.
- toset() is frequently used with for_each.
- Functions are essential for writing production-grade Terraform code.