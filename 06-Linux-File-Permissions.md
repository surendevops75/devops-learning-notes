# Linux File Permissions

## Introduction

Linux is a multi-user operating system.

To protect files and directories from unauthorized access, Linux uses a permission model based on:

- User (Owner)
- Group
- Others

Every file and directory has permissions assigned to it.

---

# Why Permissions?

Permissions help:

- Protect sensitive data
- Restrict unauthorized access
- Improve security
- Control file operations

---

# Viewing Permissions

## ls -l

```bash
ls -l
```

Example:

```text
-rw-r--r-- 1 root root 100 Jun 20 10:00 notes.txt
```

---

# Permission Structure

```text
-rw-r--r--
```

Breakdown:

```text
-   rw-   r--   r--
│    │     │     │
│    │     │     └── Others
│    │     └──────── Group
│    └────────────── Owner/User
└─────────────────── File Type
```

---

# File Type

## Regular File

```text
-
```

Example:

```text
-rw-r--r--
```

---

## Directory

```text
d
```

Example:

```text
drwxr-xr-x
```

---

## Symbolic Link

```text
l
```

Example:

```text
lrwxrwxrwx
```

---

# Permission Types

## Read (r)

Value:

```text
4
```

Allows:

- View file contents
- Read data

Example:

```bash
cat file.txt
```

---

## Write (w)

Value:

```text
2
```

Allows:

- Modify file
- Edit file
- Delete file

Example:

```bash
vim file.txt
```

---

## Execute (x)

Value:

```text
1
```

Allows:

- Execute scripts
- Run programs

Example:

```bash
./script.sh
```

---

# Permission Groups

## Owner (u)

File creator or assigned owner.

```text
u
```

---

## Group (g)

Users belonging to same group.

```text
g
```

---

## Others (o)

Everyone else.

```text
o
```

---

# Permission Example

```text
-rw-r--r--
```

Meaning:

```text
Owner  -> rw-
Group  -> r--
Others -> r--
```

---

## Permission Calculation

### Owner

```text
rw-
```

```text
r = 4
w = 2
x = 0

4 + 2 = 6
```

Owner = 6

---

### Group

```text
r--
```

```text
4 + 0 + 0 = 4
```

Group = 4

---

### Others

```text
r--
```

```text
4 + 0 + 0 = 4
```

Others = 4

Result:

```text
644
```

---

# Numeric Permission Values

| Permission | Value |
|------------|--------|
| --- | 0 |
| --x | 1 |
| -w- | 2 |
| -wx | 3 |
| r-- | 4 |
| r-x | 5 |
| rw- | 6 |
| rwx | 7 |

---

# chmod Command

Used to change file permissions.

## Syntax

```bash
chmod permissions file-name
```

---

# Symbolic Mode

## Add Permission

```bash
chmod g+w file.txt
```

Meaning:

```text
g = Group
+ = Add
w = Write
```

---

## Remove Permission

```bash
chmod o-r file.txt
```

Remove read permission from others.

---

## Add Execute Permission

```bash
chmod u+x script.sh
```

Allow owner to execute script.

---

## Multiple Changes

```bash
chmod u+x,g+w,o-r file.txt
```

---

# Numeric Mode

## chmod 744

```bash
chmod 744 file.txt
```

Breakdown:

```text
Owner  = 7 = rwx
Group  = 4 = r--
Others = 4 = r--
```

Result:

```text
rwxr--r--
```

---

## chmod 755

```bash
chmod 755 script.sh
```

Result:

```text
rwxr-xr-x
```

Common for:

- Scripts
- Applications
- Directories

---

## chmod 777

```bash
chmod 777 file.txt
```

Result:

```text
rwxrwxrwx
```

⚠️ Everyone gets full access.

Avoid in production.

---

## chmod 644

```bash
chmod 644 file.txt
```

Result:

```text
rw-r--r--
```

Most common for files.

---

## chmod 600

```bash
chmod 600 private.pem
```

Result:

```text
rw-------
```

Common for:

- SSH Private Keys
- Sensitive Files

---

## chmod 400

```bash
chmod 400 private.pem
```

Result:

```text
r--------
```

Read-only access for owner.

Used for AWS PEM keys.

---

# Common Permission Values

| Permission | Meaning |
|------------|----------|
| 777 | Full access for everyone |
| 755 | Owner full, others read & execute |
| 744 | Owner full, others read only |
| 700 | Owner only |
| 644 | Common file permission |
| 600 | Sensitive files |
| 400 | Read-only for owner |

---

# chown Command

Change file owner.

## Syntax

```bash
chown owner file-name
```

Example:

```bash
chown suresh file.txt
```

---

## Change Owner and Group

```bash
chown suresh:devops file.txt
```

---

## Recursive Ownership Change

```bash
chown -R suresh:devops project/
```

---

# chgrp Command

Change group ownership.

## Syntax

```bash
chgrp group-name file-name
```

Example:

```bash
chgrp devops file.txt
```

---

# Verify Ownership

```bash
ls -l
```

Example:

```text
-rw-r--r-- 1 suresh devops 100 file.txt
```

Owner:

```text
suresh
```

Group:

```text
devops
```

---

# Directory Permissions

Permissions behave differently for directories.

---

## Read Permission

Allows:

```text
View directory contents
```

Example:

```bash
ls directory
```

---

## Write Permission

Allows:

```text
Create files
Delete files
Rename files
```

---

## Execute Permission

Allows:

```text
Enter directory
```

Example:

```bash
cd directory
```

Without execute permission:

```bash
cd directory
```

Fails.

---

# Directory Permission Example

```text
drwxr-xr-x
```

Equivalent:

```text
755
```

Meaning:

- Owner: Full Access
- Group: Read + Execute
- Others: Read + Execute

---

# SSH Key Permissions

AWS PEM file requires:

```bash
chmod 400 key.pem
```

Otherwise:

```text
Permissions 0644 are too open
```

SSH connection will fail.

---

# umask

Defines default permissions for newly created files and directories.

## Check Current umask

```bash
umask
```

Example:

```text
0022
```

---

## Default Permissions

### Files

Maximum:

```text
666
```

Apply umask:

```text
666 - 022 = 644
```

Result:

```text
rw-r--r--
```

---

### Directories

Maximum:

```text
777
```

Apply umask:

```text
777 - 022 = 755
```

Result:

```text
rwxr-xr-x
```

---

# Access Control Lists (ACL)

ACL provides fine-grained permissions beyond owner, group, and others.

---

## Set ACL

```bash
setfacl -m u:suresh:rwx file.txt
```

---

## View ACL

```bash
getfacl file.txt
```

---

## Remove ACL

```bash
setfacl -x u:suresh file.txt
```

---

# Production Examples

## Jenkins Access

```bash
chown -R jenkins:jenkins /var/lib/jenkins
```

---

## Script Execution

```bash
chmod 755 deploy.sh
```

---

## Secure Private Key

```bash
chmod 400 prod.pem
```

---

## Shared Team Directory

```bash
chown -R devops:devops /projects
chmod 775 /projects
```

---

# Frequently Asked Interview Questions

## What does chmod 777 mean?

Everyone gets:

```text
Read
Write
Execute
```

Avoid in production.

---

## Difference Between 755 and 777

### 755

```text
Owner  -> rwx
Group  -> r-x
Others -> r-x
```

---

### 777

```text
Owner  -> rwx
Group  -> rwx
Others -> rwx
```

---

## Why Use chmod 400 for PEM Files?

AWS SSH requires private keys to be accessible only by the owner.

Example:

```bash
chmod 400 key.pem
```

---

## Difference Between chown and chmod

| Command | Purpose |
|----------|----------|
| chmod | Change permissions |
| chown | Change ownership |

---

## Difference Between Owner, Group and Others

| Type | Meaning |
|--------|---------|
| Owner | File creator |
| Group | Team members |
| Others | Everyone else |

---

## What is umask?

umask determines default permissions for newly created files and directories.

---

# Key Takeaways

- Linux permissions are based on Owner, Group, and Others.
- Read = 4, Write = 2, Execute = 1.
- chmod changes permissions.
- chown changes ownership.
- chgrp changes group ownership.
- chmod 755 is common for scripts.
- chmod 644 is common for files.
- chmod 400 is common for PEM keys.
- chmod 777 should generally be avoided.
- umask controls default permissions.
- ACLs provide advanced permission control.