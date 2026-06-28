# Linux Commands and SSH Basics

## SSH Key Generation

SSH keys are used for secure authentication between a client and a Linux server.

### Generate SSH Keys

```bash
ssh-keygen -f daws-86s
```

### Output

```text
daws-86s       -> Private Key
daws-86s.pub   -> Public Key
```

### Common Options

#### Generate RSA Key

```bash
ssh-keygen -t rsa
```

#### Generate RSA 4096-bit Key

```bash
ssh-keygen -t rsa -b 4096
```

#### Generate ED25519 Key

```bash
ssh-keygen -t ed25519
```

#### Specify File Name

```bash
ssh-keygen -f daws-86s
```

---

# Connect to EC2 Instance

## Syntax

```bash
ssh -i <private-key> ec2-user@IP
```

### Example

```bash
ssh -i daws-86s ec2-user@98.80.251.7
```

### Common SSH Options

#### Connect Using Key

```bash
ssh -i key.pem ec2-user@IP
```

#### Custom Port

```bash
ssh -p 2222 user@server
```

#### Debug Mode

```bash
ssh -v user@server
```

#### Very Verbose

```bash
ssh -vvv user@server
```

#### Execute Remote Command

```bash
ssh user@server "uptime"
```

---

# AWS Security Groups

Security Groups act as virtual firewalls.

## Inbound Rules

```text
SSH
Port: 22
Source: 0.0.0.0/0
```

Allow SSH access from anywhere.

## Outbound Rules

```text
All Traffic
Destination: 0.0.0.0/0
```

Allow traffic to any destination.

---

# Git Bash

Git Bash provides:

- Git Client
- SSH Client
- Mini Linux Environment on Windows

Default Location:

```text
C:\Users\siva
```

---

# Linux Paths

## Absolute Path

Starts from root directory.

```text
/c/devops/daws-86s
```

## Relative Path

```text
daws-86s
```

Must exist in current directory.

---

# Linux Command Structure

```text
<command-name> <inputs> <options>
```

Example:

```bash
cp source destination
```

Inputs and options may be mandatory or optional.

---

# uname Command

Displays system information.

## Syntax

```bash
uname [options]
```

### Common Options

```bash
uname
```

Displays OS name.

```bash
uname -a
```

Displays complete system information.

```bash
uname -r
```

Displays kernel version.

```bash
uname -m
```

Displays machine architecture.

```bash
uname -n
```

Displays hostname.

---

# pwd Command

Print Working Directory.

## Syntax

```bash
pwd
```

Example Output:

```text
/home/ec2-user
```

---

# User Prompts

## Normal User

```bash
$
```

Example:

```bash
[ec2-user@server ~]$
```

## Root User

```bash
#
```

Example:

```bash
[root@server ~]#
```

---

# Linux Directories

## Root Directory

```text
/
```

Starting point of Linux filesystem.

## Root User Home Directory

```text
/root
```

## Normal User Home Directory

```text
/home/ec2-user
```

---

# CRUD Operations

| Operation | Meaning |
|-----------|---------|
| Create | Create Data |
| Read | Read Data |
| Update | Modify Data |
| Delete | Delete Data |

---

# touch Command

Creates empty files.

## Syntax

```bash
touch file-name
```

### Examples

```bash
touch notes.txt
```

```bash
touch file1 file2 file3
```

Create multiple files.

```bash
touch existing-file
```

Update timestamp.

---

# ls Command

Lists files and directories.

## Syntax

```bash
ls [options]
```

### Common Options

```bash
ls
```

List files.

```bash
ls -l
```

Long listing format.

```bash
ls -a
```

Show hidden files.

```bash
ls -la
```

Long listing with hidden files.

```bash
ls -lh
```

Human-readable sizes.

```bash
ls -lr
```

Reverse alphabetical order.

```bash
ls -lt
```

Latest modified files first.

```bash
ls -ltr
```

Oldest modified files first.

```bash
ls -R
```

Recursive listing.

---

# cat Command

Create and display file contents.

## Create File

```bash
cat > file-name
```

Enter content:

```text
Hello World
Linux Basics
```

Save:

```text
CTRL + D
```

### Common Options

```bash
cat file.txt
```

Display file.

```bash
cat -n file.txt
```

Display line numbers.

```bash
cat -b file.txt
```

Number non-empty lines.

```bash
cat -E file.txt
```

Show end of line.

```bash
cat file1 file2 > combined.txt
```

Merge files.

---

# Redirection Operators

## Overwrite

```bash
>
```

Example:

```bash
echo "Hello" > file.txt
```

## Append

```bash
>>
```

Example:

```bash
echo "World" >> file.txt
```

Adds content without replacing existing content.

---

# mkdir Command

Create directories.

## Syntax

```bash
mkdir directory-name
```

### Common Options

```bash
mkdir scripts
```

Create directory.

```bash
mkdir dir1 dir2 dir3
```

Create multiple directories.

```bash
mkdir -p project/src/main
```

Create parent directories.

```bash
mkdir -v test
```

Verbose mode.

---

# rmdir Command

Delete empty directories.

## Syntax

```bash
rmdir directory-name
```

### Common Options

```bash
rmdir test
```

Delete empty directory.

```bash
rmdir -v test
```

Verbose mode.

---

# rm Command

Delete files and directories.

## Syntax

```bash
rm [options] file-name
```

### Common Options

```bash
rm file.txt
```

Delete file.

```bash
rm -r directory
```

Recursive delete.

```bash
rm -f file.txt
```

Force delete.

```bash
rm -rf directory
```

Recursive force delete.

```bash
rm -i file.txt
```

Interactive delete.

⚠️ Be careful with:

```bash
rm -rf /
```

---

# cd Command

Change directory.

## Syntax

```bash
cd directory
```

### Common Examples

```bash
cd /c/devops/daws-86s
```

Change directory.

```bash
cd ..
```

Move one step back.

```bash
cd
```

Go to home directory.

```bash
cd /
```

Go to root directory.

### Case Sensitive

```bash
Cd
```

❌ Invalid

```bash
cd
```

✅ Correct

---

# cp Command

Copy files and directories.

## Syntax

```bash
cp source destination
```

### Common Options

```bash
cp file1.txt backup.txt
```

Copy file.

```bash
cp -r source destination
```

Copy directory.

```bash
cp -i file1 file2
```

Interactive mode.

```bash
cp -p file1 backup
```

Preserve permissions.

```bash
cp -v file1 backup
```

Verbose mode.

```bash
cp -a source destination
```

Archive mode.

---

# mv Command

Move or rename files.

## Syntax

```bash
mv source destination
```

### Examples

```bash
mv file1.txt file2.txt
```

Rename file.

```bash
mv file1.txt /tmp
```

Move file.

```bash
mv project /opt
```

Move directory.

```bash
mv -i file1 file2
```

Interactive mode.

---

# Current Directory Symbol

```text
.
```

Represents current directory.

Example:

```bash
cp file.txt .
```

---

# grep Command

Search text inside files.

## Syntax

```bash
grep pattern file
```

### Examples

```bash
grep Linux notes.txt
```

### Common Options

```bash
grep -n Linux notes.txt
```

Show line numbers.

```bash
grep -c Linux notes.txt
```

Count matches.

```bash
grep -i linux notes.txt
```

Ignore case.

```bash
grep -v Linux notes.txt
```

Show non-matching lines.

```bash
grep -w Linux notes.txt
```

Match whole word.

```bash
grep -r Linux .
```

Recursive search.

```bash
grep -l Linux *.txt
```

Show file names only.

---

# SSH Configuration

Location:

```text
~/.ssh/config
```

Correct:

```text
config
```

Wrong:

```text
config.txt
```

### Example

```text
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

---

# wget Command

Download files from URLs.

## Syntax

```bash
wget <url>
```

### Examples

```bash
wget https://example.com/file.txt
```

```bash
wget -O notes.txt https://example.com/file.txt
```

Save with different name.

```bash
wget -c https://example.com/file.iso
```

Resume download.

---

# curl Command

Transfer data from URLs.

## Syntax

```bash
curl <url>
```

### Examples

```bash
curl https://example.com
```

Display content.

```bash
curl -o file.txt https://example.com/file.txt
```

Save output.

```bash
curl -I https://google.com
```

Display headers only.

---

# printf Command

Formatted output.

## Syntax

```bash
printf "Hello World"
```

### Example

```bash
printf "Name: %s\n" "Surendra"
```

---

# echo Command

Display text on terminal.

## Syntax

```bash
echo "Hello World"
```

### Common Options

```bash
echo -e "Hello\nWorld"
```

Enable escape sequences.

```bash
echo -n "Hello"
```

Do not print new line.

---

# cut Command

Extract fields from text.

## Syntax

```bash
cut -d delimiter -f field-number
```

### Example

```bash
echo "devops/aws/linux" | cut -d "/" -f2
```

Output:

```text
aws
```

### Common Options

```bash
cut -d ":" -f1 /etc/passwd
```

Extract usernames.

```bash
cut -c 1-5 file.txt
```

Extract characters.

---

# awk Command

Advanced text processing utility.

## Syntax

```bash
awk -F "/" '{print $1}'
```

### Example

```bash
echo "devops/aws/linux" | awk -F "/" '{print $1}'
```

Output:

```text
devops
```

### Common Examples

```bash
awk '{print $1}' file.txt
```

Print first column.

```bash
awk '{print NF}' file.txt
```

Print number of fields.

```bash
awk 'NR==1' file.txt
```

Print first line.

```bash
awk '/ERROR/' app.log
```

Search pattern.

---

# sudo Command

Execute commands as another user.

## Become Root

```bash
sudo su
```

Remain in current directory.

## Become Root and Move to Root Home

```bash
sudo su -
```

Move to:

```text
/root
```

### Other Examples

```bash
sudo yum update
```

Run command as root.

```bash
sudo vi /etc/hosts
```

Edit protected file.

---

# Interview Questions

## Difference Between Absolute and Relative Paths

### Absolute Path

```text
/c/devops/daws-86s
```

Starts from root.

### Relative Path

```text
daws-86s
```

Starts from current directory.

---

## Difference Between sudo su and sudo su -

| Command | Result |
|----------|----------|
| sudo su | Become root and stay in current directory |
| sudo su - | Become root and move to /root |

---

## Difference Between > and >>

| Operator | Purpose |
|-----------|----------|
| > | Overwrite |
| >> | Append |