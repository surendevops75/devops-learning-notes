# SED Editor (Stream Editor)

## Introduction

Linux provides multiple tools for processing text files.

Popular tools:

```text
cat
grep
awk
cut
sed
vim
```

Among these, `sed` is one of the most powerful tools used by DevOps Engineers for:

- File Modifications
- Configuration Updates
- Text Replacement
- Automation

---

# What is SED?

SED stands for:

```text
Stream Editor
```

It is a text-processing utility used to:

```text
Read
Insert
Update
Delete
Replace
```

text in files or streams.

---

# Why is it Called Stream Editor?

SED processes data as a stream.

```text
Input File
     │
     ▼
Process Line by Line
     │
     ▼
Output
```

Instead of opening the file interactively like Vim, SED processes text automatically.

---

# SED is Non-Interactive

Vim:

```text
Open File
Edit Manually
Save
Exit
```

SED:

```text
Single Command
Automatic Processing
```

---

# Why Use SED?

Benefits:

```text
Automation
Speed
Script Friendly
Batch Processing
Configuration Management
```

---

# Basic Syntax

```bash
sed 'operation' file-name
```

Example:

```bash
sed '1d' sample.txt
```

---

# Sample File

```text
Linux
AWS
Docker
Kubernetes
Terraform
```

---

# Insert Before First Line

Command:

```bash
sed '1i Hello SED' sample.txt
```

---

# Output

```text
Hello SED
Linux
AWS
Docker
Kubernetes
Terraform
```

---

# Understanding 1i

```text
1 → First Line

i → Insert Before
```

---

# Insert After First Line

Command:

```bash
sed '1a Hello' sample.txt
```

---

# Output

```text
Linux
Hello
AWS
Docker
Kubernetes
Terraform
```

---

# Understanding 1a

```text
1 → First Line

a → Append After
```

---

# Delete Specific Line

Delete second line.

Command:

```bash
sed '2d' sample.txt
```

---

# Output

```text
Linux
Docker
Kubernetes
Terraform
```

---

# Understanding d

```text
d → Delete
```

---

# Delete Lines Matching Pattern

Command:

```bash
sed '/Hello/d' sample.txt
```

---

# Sample File

```text
Linux
Hello
Docker
Hello
Terraform
```

---

# Output

```text
Linux
Docker
Terraform
```

---

# Pattern Matching

```text
/Hello/
```

means:

```text
Find Lines Containing Hello
```

---

# Substitute Text

One of the most commonly used SED operations.

Syntax:

```bash
sed 's/old/new/'
```

---

# Example

```bash
sed 's/root/ROOT/' file.txt
```

---

# Input

```text
root user
root group
```

---

# Output

```text
ROOT user
ROOT group
```

Only the first occurrence in each line is replaced.

---

# Global Replacement

Command:

```bash
sed 's/root/ROOT/g'
```

---

# Meaning of g

```text
g → Global
```

Replace all occurrences.

---

# Example

Input:

```text
root root root
```

---

Command:

```bash
sed 's/root/ROOT/g'
```

---

Output:

```text
ROOT ROOT ROOT
```

---

# Difference Between Normal and Global Replace

Without g:

```text
root ROOT root
```

---

With g:

```text
ROOT ROOT ROOT
```

---

# Replace Specific Line

Command:

```bash
sed '3c Hello Change' file.txt
```

---

# Meaning

```text
3 → Third Line

c → Change
```

---

# Example

Input:

```text
Linux
AWS
Docker
Terraform
```

---

Output:

```text
Linux
AWS
Hello Change
Terraform
```

---

# Replace Based on Pattern

Command:

```bash
sed '/error/c NO-ERROR' file.txt
```

---

# Example

Input:

```text
success
error found
completed
```

---

Output:

```text
success
NO-ERROR
completed
```

---

# Common SED Operations

| Operation | Meaning |
|------------|----------|
| i | Insert Before |
| a | Append After |
| d | Delete |
| s | Substitute |
| c | Change |
| g | Global Replace |

---

# In-Place Editing

By default, SED only displays output.

Original file remains unchanged.

---

# Example

```bash
sed 's/root/ROOT/g' file.txt
```

Displays modified output only.

---

# Modify Actual File

Use:

```bash
sed -i 's/root/ROOT/g' file.txt
```

---

# What Does -i Mean?

```text
In Place
```

Modify original file.

---

# Example

Before:

```text
root user
```

---

Command:

```bash
sed -i 's/root/ROOT/g' file.txt
```

---

After:

```text
ROOT user
```

---

# Real DevOps Example

Update Configuration File.

Before:

```text
DB_HOST=localhost
```

---

Command:

```bash
sed -i 's/localhost/mysql.daws86s.fun/' app.conf
```

---

After:

```text
DB_HOST=mysql.daws86s.fun
```

---

# Another Example

Update Environment:

Before:

```text
ENV=dev
```

---

Command:

```bash
sed -i 's/dev/prod/' app.conf
```

---

After:

```text
ENV=prod
```

---

# Updating Nginx Configurations

Before:

```text
listen 80;
```

---

Command:

```bash
sed -i 's/80/8080/' nginx.conf
```

---

After:

```text
listen 8080;
```

---

# SED with Variables

Example:

```bash
DB_HOST=mysql.daws86s.fun

sed -i "s/DB_HOST=.*/DB_HOST=$DB_HOST/" app.conf
```

---

# Combining with Shell Scripts

Example:

```bash
CONFIG_FILE=app.conf

sed -i 's/dev/prod/' $CONFIG_FILE
```

---

# SED vs VIM

| SED | VIM |
|--------|--------|
| Non-Interactive | Interactive |
| Script Friendly | Manual Editing |
| Automation | Human Editing |
| Fast for Bulk Changes | Better for Manual Work |

---

# Common DevOps Use Cases

## Configuration Updates

```text
Application Configs
```

---

## Environment Changes

```text
dev → qa
qa → prod
```

---

## DNS Updates

```text
localhost → domain name
```

---

## IP Address Updates

```text
Old IP → New IP
```

---

## Log Processing

```text
Replace Patterns
Remove Sensitive Data
```

---

# Common Mistakes

## Forgetting -i

Command:

```bash
sed 's/root/ROOT/g' file.txt
```

Displays output only.

File remains unchanged.

---

## Wrong Pattern

Incorrect pattern may replace unexpected text.

Always validate before using `-i`.

---

## Global Replacement Risks

```bash
sed -i 's/dev/prod/g'
```

May replace values unintentionally.

---

# Best Practices

## Test First

Before:

```bash
sed -i
```

Run:

```bash
sed
```

without `-i`.

---

## Backup Files

Example:

```bash
cp app.conf app.conf.bak
```

before modification.

---

## Use Specific Patterns

Avoid overly broad replacements.

---

# Frequently Asked Interview Questions

## What is SED?

Stream Editor used for text processing and file manipulation.

---

## Is SED Interactive?

No.

SED is a non-interactive editor.

---

## What Does s Mean?

Substitute.

---

## What Does d Mean?

Delete.

---

## What Does i Mean?

Insert before a line.

---

## What Does a Mean?

Append after a line.

---

## What Does c Mean?

Change a line.

---

## What Does g Mean?

Global replacement.

---

## What Does -i Mean?

Modify file in place.

---

## Why Is SED Important in DevOps?

Used extensively for:

- Configuration Updates
- Environment Changes
- Automation Scripts
- Infrastructure Management

---

# Key Takeaways

- SED stands for Stream Editor.
- SED is a non-interactive text processing tool.
- It can insert, append, delete, substitute, and replace text.
- `s` is used for substitution.
- `g` performs global replacement.
- `-i` modifies files directly.
- SED is heavily used in DevOps automation.
- Configuration file updates are one of the most common SED use cases.
- SED is faster and more automation-friendly than manual editing.