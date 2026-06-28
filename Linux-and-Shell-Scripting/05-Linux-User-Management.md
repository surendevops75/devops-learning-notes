# Linux User Management

## Introduction

User Management is one of the most important Linux Administration tasks.

Linux is a multi-user operating system, meaning multiple users can access the same server simultaneously.

To manage users effectively, Linux provides:

- Users
- Groups
- Roles
- Permissions

---

# Basic Concepts

## User

A User represents a person or service account that can access the Linux system.

### Examples

```text
suresh
ec2-user
root
jenkins
nginx
mysql
```

---

## Group

A Group is a collection of users.

### Example

```text
DevOps Team
```

Members:

```text
suresh
ramesh
mahesh
```

All can be assigned to the same group.

---

## Role

A Role represents a responsibility or position.

### Examples

```text
DevOps Engineer
Developer
Database Administrator
System Administrator
Security Engineer
```

---

## Permissions

Permissions define what actions a user can perform.

Examples:

- Read Files
- Write Files
- Execute Programs
- Access Directories

---

# Real World Example

## Organization Structure

```text
Company
│
├── DevOps Team
│   ├── Suresh
│   ├── Ramesh
│
├── Development Team
│   ├── John
│   ├── David
│
└── QA Team
    ├── Smith
    ├── Kumar
```

---

# User Management Workflow

Typical tasks:

1. Create User
2. Create Group
3. Add User to Group
4. Remove User from Group
5. Delete User
6. Delete Group

---

# Practical Example

Goal:

```text
1. Create DevOps Group
2. Create Suresh User
3. Add Suresh to DevOps Group
4. Remove Suresh from DevOps Group
5. Delete Suresh User
```

---

# Create User

## useradd Command

Used to create users.

### Syntax

```bash
useradd <username>
```

### Example

```bash
useradd suresh
```

Creates:

```text
User: suresh
```

---

## Common useradd Options

### Create User with Home Directory

```bash
useradd -m suresh
```

---

### Specify Home Directory

```bash
useradd -d /custom/home suresh
```

---

### Specify Shell

```bash
useradd -s /bin/bash suresh
```

---

### Create User with UID

```bash
useradd -u 2001 suresh
```

---

### Create User with Primary Group

```bash
useradd -g devops suresh
```

---

### Create User with Comment

```bash
useradd -c "DevOps Engineer" suresh
```

---

# Create Group

## groupadd Command

Used to create groups.

### Syntax

```bash
groupadd <group-name>
```

### Example

```bash
groupadd devops
```

---

## Common groupadd Options

### Create Group with GID

```bash
groupadd -g 3001 devops
```

---

### Create System Group

```bash
groupadd -r devops
```

---

# Verify User Information

## id Command

Displays user information.

### Syntax

```bash
id <username>
```

### Example

```bash
id suresh
```

Output:

```text
uid=1001(suresh)
gid=1001(suresh)
groups=1001(suresh)
```

---

# Important Linux Behavior

When a user is created:

```bash
useradd suresh
```

Linux automatically creates:

```text
User  : suresh
Group : suresh
```

Default Group Name:

```text
suresh
```

---

# User Information File

## /etc/passwd

Stores user information.

### Example

```text
suresh:x:1001:1001::/home/suresh:/bin/bash
```

---

## Fields Explanation

```text
username
password_placeholder
uid
gid
comment
home_directory
login_shell
```

Example:

```text
suresh:x:1001:1001::/home/suresh:/bin/bash
```

| Field | Value |
|---------|---------|
| Username | suresh |
| UID | 1001 |
| GID | 1001 |
| Home Directory | /home/suresh |
| Shell | /bin/bash |

---

# Group Information File

## /etc/group

Stores group details.

### Example

```text
devops:x:1002:
```

---

## Fields

```text
group_name
password_placeholder
gid
members
```

---

# Password File

## /etc/shadow

Stores encrypted passwords.

### Example

```text
suresh:$6$abcxyz.....
```

### Important

Passwords are never stored in plain text.

They are stored as encrypted hashes.

---

# Primary and Secondary Groups

Every Linux user must have:

- One Primary Group
- Zero or More Secondary Groups

---

## Primary Group

Main group associated with the user.

Example:

```text
suresh
```

---

## Secondary Group

Additional groups assigned to user.

Example:

```text
devops
docker
wheel
```

---

# Add User to Primary Group

## usermod -g

### Syntax

```bash
usermod -g <group-name> <user-name>
```

### Example

```bash
usermod -g devops suresh
```

Changes primary group.

---

# Add User to Secondary Group

## usermod -aG

### Syntax

```bash
usermod -aG <group-name> <user-name>
```

### Example

```bash
usermod -aG devops suresh
```

### Important

Always use:

```bash
-aG
```

Without `-a`, existing groups may be removed.

---

# Verify Group Membership

```bash
id suresh
```

Example:

```text
uid=1001(suresh)
gid=1002(devops)
groups=1002(devops),1003(docker)
```

---

# Set User Password

## passwd Command

### Syntax

```bash
passwd <username>
```

### Example

```bash
passwd suresh
```

System asks:

```text
New Password:
Retype New Password:
```

---

## Common passwd Options

### Lock User

```bash
passwd -l suresh
```

---

### Unlock User

```bash
passwd -u suresh
```

---

### Expire Password

```bash
passwd -e suresh
```

Force password change on next login.

---

# Delete User from Group

## gpasswd -d

### Syntax

```bash
gpasswd -d <user-name> <group-name>
```

### Example

```bash
gpasswd -d suresh devops
```

Removes user from group.

---

# Delete User

## userdel Command

### Syntax

```bash
userdel <username>
```

### Example

```bash
userdel suresh
```

---

## Delete User with Home Directory

```bash
userdel -r suresh
```

Deletes:

```text
User
Home Directory
Mail Spool
```

---

# Delete Group

## groupdel Command

### Syntax

```bash
groupdel <group-name>
```

### Example

```bash
groupdel devops
```

---

# SSH Configuration

## SSH Configuration File

Location:

```text
/etc/ssh/sshd_config
```

---

# Password Authentication

Default:

```text
PasswordAuthentication no
```

Meaning:

```text
Only SSH Key Authentication Allowed
```

---

# Enable Password Authentication

Open file:

```bash
vim /etc/ssh/sshd_config
```

Modify:

```text
PasswordAuthentication yes
```

---

# Validate SSH Configuration

## sshd -t

### Syntax

```bash
sshd -t
```

Checks syntax errors.

Output:

```text
No output = Configuration is valid
```

---

# Restart SSH Service

## systemctl restart sshd

```bash
systemctl restart sshd
```

Apply new SSH configuration.

---

# Common systemctl Commands

### Start Service

```bash
systemctl start sshd
```

---

### Stop Service

```bash
systemctl stop sshd
```

---

### Restart Service

```bash
systemctl restart sshd
```

---

### Check Status

```bash
systemctl status sshd
```

---

### Enable Service

```bash
systemctl enable sshd
```

Start automatically during boot.

---

### Disable Service

```bash
systemctl disable sshd
```

---

# Frequently Asked Interview Questions

## What is UID?

UID stands for User ID.

Example:

```text
UID = 1001
```

Uniquely identifies a user.

---

## What is GID?

GID stands for Group ID.

Example:

```text
GID = 1002
```

Uniquely identifies a group.

---

## Difference Between useradd and usermod

| Command | Purpose |
|----------|----------|
| useradd | Create User |
| usermod | Modify Existing User |

---

## Difference Between Primary and Secondary Group

| Primary Group | Secondary Group |
|---------------|----------------|
| One per user | Multiple allowed |
| Default group | Additional groups |
| Mandatory | Optional |

---

## What is /etc/passwd?

Stores user account information.

---

## What is /etc/group?

Stores group information.

---

## What is /etc/shadow?

Stores encrypted user passwords.

---

## Why Use SSH Keys Instead of Passwords?

- More Secure
- Supports Automation
- Prevents Brute Force Attacks
- Industry Standard

---

# Key Takeaways

- Linux is a multi-user operating system.
- Users belong to groups.
- Every user has a UID.
- Every group has a GID.
- `/etc/passwd` stores user information.
- `/etc/group` stores group information.
- `/etc/shadow` stores encrypted passwords.
- `useradd` creates users.
- `groupadd` creates groups.
- `usermod` modifies users.
- `passwd` manages passwords.
- `gpasswd` manages group membership.
- `systemctl` manages services.
- SSH configuration is stored in `/etc/ssh/sshd_config`.