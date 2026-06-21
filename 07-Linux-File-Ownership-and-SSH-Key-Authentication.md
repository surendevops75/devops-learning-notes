# Linux File Ownership and SSH Key Authentication

## File Ownership in Linux

Every file and directory in Linux has:

- Owner (User)
- Group
- Permissions

Example:

```bash
ls -l
```

Output:

```text
-rw-r--r-- 1 suresh devops 120 Jun 21 notes.txt
```

Breakdown:

```text
-rw-r--r--
│
├── Owner : suresh
└── Group : devops
```

---

# Ownership vs Permissions

## chmod

Used to change permissions.

```bash
chmod 755 file.txt
```

Who can execute?

- File Owner
- Root User

---

## chown

Used to change ownership.

```bash
chown <user>:<group> file-name
```

Example:

```bash
chown suresh:devops notes.txt
```

### Important

Even the owner of the file cannot change ownership.

Only:

- Root User
- Sudo User (with required privileges)

can execute `chown`.

---

# chown Command

## Syntax

```bash
chown [options] user:group file
```

---

## Change Owner

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

### What Does -R Mean?

Recursive.

Changes ownership for:

```text
project/
├── file1
├── file2
└── subfolder/
    ├── app.py
    └── test.py
```

Everything inside will be updated.

---

# Verify Ownership

```bash
ls -l
```

Example:

```text
-rw-r--r-- 1 suresh devops file.txt
```

---

# SSH Key Authentication

## Why SSH Keys?

Instead of:

```text
Username + Password
```

we use:

```text
Public Key + Private Key
```

Benefits:

- More Secure
- No Password Sharing
- Supports Automation
- Industry Standard

---

# SSH Key Pair

When we generate SSH keys:

```bash
ssh-keygen -f suresh
```

Linux creates:

```text
suresh       -> Private Key
suresh.pub   -> Public Key
```

---

# Public Key

Can be shared.

Example:

```text
suresh.pub
```

Stored on the server.

---

# Private Key

Must be kept secret.

Example:

```text
suresh
```

Used to authenticate.

Never share private keys.

---

# SSH Login Flow

```text
Client
│
├── Private Key
│
▼
Linux Server
│
├── authorized_keys
│
└── Public Key
```

If keys match:

```text
Login Successful
```

---

# How Linux Validates SSH Keys

Step 1:

User executes:

```bash
ssh -i suresh suresh@IP
```

Step 2:

Server checks:

```text
/home/suresh/.ssh/authorized_keys
```

Step 3:

Public key found?

```text
YES
```

Step 4:

Key matches?

```text
YES
```

Step 5:

```text
Access Granted
```

---

# authorized_keys File

Location:

```text
/home/<username>/.ssh/authorized_keys
```

Example:

```text
/home/suresh/.ssh/authorized_keys
```

Contains:

```text
ssh-rsa AAAAB3Nza...
```

User's public key.

---

# Employee SSH Access Workflow

Suppose a new employee joins.

Name:

```text
sivakumar
```

---

## Step 1: Employee Generates Keys

```bash
ssh-keygen -f sivakumar
```

Generated:

```text
sivakumar
sivakumar.pub
```

---

## Step 2: Employee Sends Public Key

```text
sivakumar.pub
```

Send to Linux Administrators.

Never send:

```text
sivakumar
```

(private key)

---

## Step 3: Admin Creates User

```bash
useradd sivakumar
```

---

## Step 4: Create .ssh Directory

```bash
mkdir /home/sivakumar/.ssh
```

---

## Step 5: Set Directory Permissions

```bash
chmod 700 /home/sivakumar/.ssh
```

Meaning:

```text
Owner  -> rwx
Group  -> ---
Others -> ---
```

Only owner has access.

---

## Step 6: Set Ownership

```bash
chown sivakumar:sivakumar /home/sivakumar/.ssh
```

---

## Step 7: Create authorized_keys

```bash
touch /home/sivakumar/.ssh/authorized_keys
```

---

## Step 8: Set File Permissions

```bash
chmod 600 /home/sivakumar/.ssh/authorized_keys
```

Meaning:

```text
Owner  -> rw-
Group  -> ---
Others -> ---
```

---

## Step 9: Set Ownership

```bash
chown sivakumar:sivakumar /home/sivakumar/.ssh/authorized_keys
```

---

## Step 10: Copy Public Key

Append content from:

```text
sivakumar.pub
```

into:

```text
authorized_keys
```

Example:

```bash
cat sivakumar.pub >> /home/sivakumar/.ssh/authorized_keys
```

---

# User Login

```bash
ssh -i sivakumar sivakumar@3.87.244.153
```

Explanation:

| Component | Meaning |
|------------|----------|
| ssh | Secure Shell |
| -i | Identity File |
| sivakumar | Private Key |
| sivakumar@IP | Username and Server |

---

# SSH Directory Structure

```text
/home/sivakumar
│
└── .ssh
     │
     ├── authorized_keys
     │
     └── known_hosts
```

---

# Common SSH Commands

## Generate Keys

```bash
ssh-keygen
```

---

## Generate Named Key

```bash
ssh-keygen -f sivakumar
```

---

## Connect Using Key

```bash
ssh -i sivakumar sivakumar@IP
```

---

## Copy Public Key Automatically

```bash
ssh-copy-id user@server
```

---

## Debug SSH Connection

```bash
ssh -v user@server
```

---

## Very Verbose Debug

```bash
ssh -vvv user@server
```

---

# SSH Permission Requirements

## .ssh Directory

```bash
chmod 700 ~/.ssh
```

---

## authorized_keys

```bash
chmod 600 ~/.ssh/authorized_keys
```

---

## Private Key

```bash
chmod 400 private-key
```

or

```bash
chmod 600 private-key
```

---

# Common SSH Errors

## Permission Denied

```text
Permission denied (publickey)
```

Possible Reasons:

- Wrong private key
- Public key not in authorized_keys
- Incorrect permissions
- Wrong username

---

## Bad Permissions

```text
Permissions 0644 are too open
```

Fix:

```bash
chmod 400 private-key
```

---

## Ownership Problems

Verify:

```bash
ls -ld ~/.ssh
```

Correct:

```bash
drwx------ sivakumar sivakumar
```

---

# Production Examples

## Jenkins Server Access

```bash
ssh -i jenkins.pem ec2-user@server
```

---

## Deploy Script Access

```bash
ssh -i deploy-key ubuntu@server
```

---

## Team Member Access

```bash
useradd developer1

mkdir /home/developer1/.ssh

chmod 700 /home/developer1/.ssh

touch /home/developer1/.ssh/authorized_keys

chmod 600 /home/developer1/.ssh/authorized_keys
```

---

# Frequently Asked Interview Questions

## Difference Between chmod and chown

| Command | Purpose |
|----------|----------|
| chmod | Change Permissions |
| chown | Change Ownership |

---

## Who Can Run chown?

- Root User
- Sudo User

Normal users cannot change ownership.

---

## What is authorized_keys?

Stores public keys allowed to access a Linux server.

---

## Difference Between Public and Private Key

| Public Key | Private Key |
|------------|------------|
| Shared | Secret |
| Stored on Server | Stored on Client |
| Used for Validation | Used for Authentication |

---

## Why Use SSH Keys?

- More Secure
- Passwordless Login
- Automation Friendly
- Industry Standard

---

## Why chmod 700 for .ssh?

Only owner should access SSH configuration.

---

## Why chmod 600 for authorized_keys?

Prevent unauthorized modifications.

---

# Key Takeaways

- Ownership consists of User and Group.
- `chown` changes ownership.
- `chown -R` changes ownership recursively.
- SSH authentication uses Public and Private Keys.
- Public key is stored in `authorized_keys`.
- `.ssh` directory should have `700` permissions.
- `authorized_keys` should have `600` permissions.
- Private keys should never be shared.
- SSH key authentication is the preferred login method in production environments.