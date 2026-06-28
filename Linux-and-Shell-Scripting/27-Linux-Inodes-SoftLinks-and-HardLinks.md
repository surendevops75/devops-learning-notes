# Linux Inodes, Soft Links and Hard Links

## Introduction

Linux stores files differently from Windows.

When a file is created:

```text
File Name
    │
    ▼
Inode
    │
    ▼
Actual Data Blocks
```

Understanding:

- Inodes
- Soft Links (Symbolic Links)
- Hard Links

is important for Linux Administration, DevOps, and System Engineering.

---

# What is an Inode?

An Inode is a data structure used by Linux filesystems to store information about a file.

An inode contains:

- File Permissions
- Owner Information
- Group Information
- File Size
- Creation Time
- Modification Time
- Data Block Locations

An inode does NOT contain:

```text
File Name
```

---

# File Structure in Linux

Example:

```text
notes.txt
```

Internally:

```text
notes.txt
     │
     ▼
 Inode Number
     │
     ▼
 Actual Data
```

---

# View Inode Number

Create a file:

```bash
touch notes.txt
```

Check inode:

```bash
ls -i notes.txt
```

Example Output:

```text
123456 notes.txt
```

Here:

```text
123456
```

is the inode number.

---

# View Detailed Information

```bash
ls -li
```

Example:

```text
123456 -rw-r--r-- 1 ec2-user ec2-user 0 Jun 23 notes.txt
```

---

# Why Inodes are Important

Linux accesses files through inode numbers.

The file name is only a reference.

```text
File Name
     │
     ▼
 Inode
     │
     ▼
 Data
```

---

# What is a Link?

A link is another way to access a file.

Linux supports:

1. Soft Link (Symbolic Link)
2. Hard Link

---

# Soft Link (Symbolic Link)

A Soft Link is a shortcut pointing to another file.

Windows Equivalent:

```text
Shortcut
```

Example:

```text
chrome.exe
```

Desktop shortcut points to:

```text
Actual Chrome Installation
```

---

# Create Soft Link

Syntax:

```bash
ln -s <source-file> <softlink-name>
```

Example:

```bash
touch original.txt

ln -s original.txt shortcut.txt
```

---

# Verify Soft Link

```bash
ls -l
```

Output:

```text
shortcut.txt -> original.txt
```

---

# Check Inode Numbers

```bash
ls -li
```

Example:

```text
123456 original.txt

654321 shortcut.txt
```

Notice:

```text
Different Inodes
```

---

# Soft Link Architecture

```text
shortcut.txt
      │
      ▼
original.txt
      │
      ▼
Actual Data
```

---

# What Happens if Original File is Deleted?

Example:

```bash
rm original.txt
```

Soft link still exists:

```bash
ls -l
```

Output:

```text
shortcut.txt -> original.txt
```

But:

```text
Link is Broken
```

because original file no longer exists.

---

# Soft Link Characteristics

- Different Inode
- Points to File Name
- Similar to Windows Shortcut
- Breaks if Original File is Deleted
- Can Link Across Filesystems

---

# Hard Link

A Hard Link is another file name pointing to the same inode.

Example:

```text
original.txt
hardlink.txt
```

Both point to:

```text
Same Inode
```

---

# Create Hard Link

Syntax:

```bash
ln <source-file> <hardlink-name>
```

Example:

```bash
touch original.txt

ln original.txt hardlink.txt
```

---

# Verify Hard Link

```bash
ls -li
```

Example:

```text
123456 original.txt

123456 hardlink.txt
```

Notice:

```text
Same Inode
```

---

# Hard Link Architecture

```text
original.txt
        │
        ▼
      Inode
        ▲
        │
hardlink.txt
        │
        ▼
    Actual Data
```

Both names point to the same file data.

---

# What Happens if Original File is Deleted?

Example:

```bash
rm original.txt
```

Hard link still works.

```bash
cat hardlink.txt
```

Output:

```text
File Contents Available
```

Reason:

```text
Data still referenced by inode
```

---

# What Happens if Hard Link is Deleted?

Example:

```bash
rm hardlink.txt
```

Original file still exists.

Data is removed only when:

```text
All Hard Links are Deleted
```

---

# Hard Link Characteristics

- Same Inode
- Same Data
- Does Not Break if Original Name is Deleted
- More Efficient
- Cannot Cross Filesystems
- Cannot Link Directories (Normally)

---

# Soft Link vs Hard Link

| Soft Link | Hard Link |
|------------|------------|
| Different Inode | Same Inode |
| Shortcut | Same File |
| Breaks if Original Deleted | Continues Working |
| Can Cross Filesystems | Cannot Cross Filesystems |
| Uses `ln -s` | Uses `ln` |

---

# Commands Summary

## View Inode

```bash
ls -i
```

---

## Detailed Inode Information

```bash
ls -li
```

---

## Create Soft Link

```bash
ln -s source target
```

Example:

```bash
ln -s original.txt shortcut.txt
```

---

## Create Hard Link

```bash
ln source target
```

Example:

```bash
ln original.txt hardlink.txt
```

---

## Remove Link

```bash
rm link-name
```

---

# Real World Examples

## Soft Links

Commonly used for:

```text
Configuration Files
Application Shortcuts
Version Management
```

Example:

```text
java -> java-21
```

---

## Hard Links

Used internally by:

```text
Backup Systems
Filesystem Operations
Storage Optimization
```

---

# Troubleshooting

## Check if File is a Soft Link

```bash
ls -l
```

Output:

```text
shortcut -> original
```

---

## Check Inode Numbers

```bash
ls -li
```

---

## Find All Hard Links

```bash
find . -samefile original.txt
```

---

# Frequently Asked Interview Questions

## What is an Inode?

A data structure containing metadata about a file.

---

## Does an Inode Store File Name?

No.

File names are stored separately.

---

## What is a Soft Link?

A shortcut pointing to another file.

---

## What is a Hard Link?

Another file name pointing to the same inode.

---

## Which Link Uses the Same Inode?

```text
Hard Link
```

---

## Which Link Breaks When Original File is Deleted?

```text
Soft Link
```

---

## Can Soft Links Cross Filesystems?

```text
Yes
```

---

## Can Hard Links Cross Filesystems?

```text
No
```

---

## Which Command Creates a Soft Link?

```bash
ln -s source target
```

---

## Which Command Creates a Hard Link?

```bash
ln source target
```

---

# Key Takeaways

- Linux stores file metadata in inodes.
- File names point to inode numbers.
- Soft links are similar to Windows shortcuts.
- Soft links have different inode numbers from the original file.
- Hard links share the same inode as the original file.
- Deleting the original file breaks a soft link.
- Deleting the original file does not affect a hard link.
- `ln -s` creates soft links.
- `ln` creates hard links.
- Understanding inodes and links is important for Linux administration and troubleshooting.