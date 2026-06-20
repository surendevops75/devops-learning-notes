# Vim Editor and Text Processing

## What is Vim?

Vim stands for **Vi Improved**.

It is one of the most powerful text editors available in Linux and Unix systems.

### Why Vim?

- Lightweight
- Fast
- Available in almost every Linux distribution
- No GUI required
- Widely used by Linux Administrators and DevOps Engineers

---

# Creating Files

## Using touch

```bash
touch file.txt
```

Creates an empty file.

---

## Using cat

### Create New File

```bash
cat > file.txt
```

Enter content:

```text
Hello World
Linux Notes
```

Press:

```text
CTRL + D
```

to save.

---

### Append Data

```bash
cat >> file.txt
```

Adds content to existing file.

---

# Opening a File in Vim

```bash
vim file.txt
```

### Behavior

#### If file exists

```text
Opens existing file
```

#### If file does not exist

```text
Creates and opens new file
```

---

# Vim Modes

Vim has three important modes.

## 1. Command Mode (Default Mode)

When Vim opens, it starts in Command Mode.

Used for:

- Navigation
- Copy
- Paste
- Delete
- Search

---

## 2. Insert Mode

Used for editing text.

### Enter Insert Mode

```text
i
```

Insert before cursor.

### Other Insert Commands

```text
I
```

Insert at beginning of line.

```text
a
```

Append after cursor.

```text
A
```

Append at end of line.

```text
o
```

Open new line below.

```text
O
```

Open new line above.

### Exit Insert Mode

Press:

```text
ESC
```

---

## 3. Command-Line Mode

Accessed using:

```text
:
```

Used for:

- Save
- Quit
- Search and Replace
- Settings

---

# Vim Save and Exit Commands

## Save and Quit

```vim
:wq
```

Write and Quit.

---

## Save Only

```vim
:w
```

Save file.

---

## Quit

```vim
:q
```

Quit editor.

Works only if no unsaved changes exist.

---

## Quit Without Saving

```vim
:q!
```

Discard changes and quit.

---

# Display Line Numbers

```vim
:set nu
```

Example:

```text
1 Hello
2 Linux
3 DevOps
```

---

# Navigation Commands

## Go to First Line

```vim
gg
```

---

## Go to Last Line

```vim
Shift + G
```

or

```vim
G
```

---

## Go to Specific Line

```vim
:25
```

Moves cursor to line 25.

---

# Search Operations

## Search from Top

```vim
:/root
```

Searches from beginning.

---

## Search from Bottom

```vim
:?root
```

Searches from bottom.

---

## Next Match

```vim
n
```

---

## Previous Match

```vim
N
```

---

## Remove Search Highlight

```vim
:noh
```

---

# Delete Operations

## Delete Current Line

```vim
dd
```

---

## Delete Multiple Lines

```vim
5dd
```

Deletes 5 lines.

---

## Delete Specific Line

```vim
:28d
```

Deletes line 28.

---

# Copy and Paste Operations

## Copy Current Line

```vim
yy
```

(Yank)

---

## Copy 5 Lines

```vim
5yy
```

---

## Paste Below

```vim
p
```

---

## Paste Above

```vim
P
```

---

# Undo and Redo

## Undo

```vim
u
```

---

## Redo

```vim
CTRL + r
```

---

# Search and Replace

## Replace First Occurrence in Specific Line

```vim
:28s/root/ROOT
```

Only first occurrence in line 28.

---

## Replace All Occurrences in Specific Line

```vim
:29s/root/ROOT/g
```

g = Global

---

## Replace All Occurrences in Entire File

```vim
:%s/root/ROOT/g
```

---

## Replace With Confirmation

```vim
:%s/root/ROOT/gc
```

Ask before each replacement.

---

# Vim Cheat Sheet

| Command | Purpose |
|----------|----------|
| i | Insert mode |
| ESC | Exit insert mode |
| :w | Save |
| :q | Quit |
| :wq | Save and quit |
| :q! | Quit without saving |
| gg | First line |
| G | Last line |
| dd | Delete line |
| yy | Copy line |
| p | Paste below |
| P | Paste above |
| u | Undo |
| CTRL+r | Redo |
| /word | Search |
| :set nu | Line numbers |

---

# Head Command

Displays first lines of a file.

## Syntax

```bash
head file.txt
```

Default:

```text
First 10 lines
```

---

## Common Options

### First 5 Lines

```bash
head -n 5 file.txt
```

---

### First 20 Lines

```bash
head -n 20 file.txt
```

---

# Tail Command

Displays last lines of a file.

## Syntax

```bash
tail file.txt
```

Default:

```text
Last 10 lines
```

---

## Common Options

### Last 5 Lines

```bash
tail -n 5 file.txt
```

---

### Last 20 Lines

```bash
tail -n 20 file.txt
```

---

### Live Monitoring

```bash
tail -f application.log
```

Continuously monitors file updates.

Commonly used for:

- Application Logs
- Nginx Logs
- Apache Logs
- Kubernetes Logs

---

# Extract Specific Lines from File

## Example

Display lines 6 to 8.

```bash
head -n 8 file.txt | tail -n 3
```

### Explanation

```text
head -n 8
```

Gets first 8 lines.

```text
tail -n 3
```

Gets last 3 from those 8.

Result:

```text
Line 6
Line 7
Line 8
```

---

# Display Lines 55 to 65

```bash
head -n 65 file.txt | tail -n 11
```

Result:

```text
55
56
57
...
65
```

---

# Piping

## What is Pipe?

Pipe passes output of one command as input to another command.

Symbol:

```bash
|
```

---

## Syntax

```bash
command1 | command2
```

---

## Examples

### Example 1

```bash
ls -l | grep txt
```

Display only txt files.

---

### Example 2

```bash
cat file.txt | grep Linux
```

Search Linux in file.

---

### Example 3

```bash
ps -ef | grep nginx
```

Find nginx process.

---

### Example 4

```bash
head -n 65 file.txt | tail -n 11
```

Display lines 55 to 65.

---

# Common Text Processing Commands

## grep

Search text.

```bash
grep DevOps notes.txt
```

---

## cut

Extract fields.

```bash
echo "devops/aws/linux" | cut -d "/" -f2
```

Output:

```text
aws
```

---

## awk

Advanced text processing.

```bash
echo "devops/aws/linux" | awk -F "/" '{print $1}'
```

Output:

```text
devops
```

---

# Production Use Cases

## View Application Logs

```bash
tail -f app.log
```

---

## Search Errors

```bash
grep ERROR app.log
```

---

## Replace Configuration Values

```vim
:%s/old-ip/new-ip/g
```

---

## Open Configuration Files

```bash
vim /etc/ssh/sshd_config
```

```bash
vim /etc/hosts
```

```bash
vim /etc/nginx/nginx.conf
```

---

# Frequently Asked Interview Questions

## What is Vim?

Vim is a command-line text editor used to create and modify files in Linux systems.

---

## Difference Between Vi and Vim

| Vi | Vim |
|----|-----|
| Original Editor | Improved Version |
| Limited Features | More Features |
| Basic Editing | Advanced Editing |

---

## Difference Between Head and Tail

| Command | Purpose |
|----------|----------|
| head | Top lines |
| tail | Bottom lines |

---

## What is tail -f?

Used for real-time monitoring of log files.

Example:

```bash
tail -f /var/log/messages
```

---

## What is a Pipe?

A pipe (`|`) sends output of one command as input to another command.

Example:

```bash
ps -ef | grep nginx
```

---

# Key Takeaways

- Vim is the most widely used Linux editor.
- Vim has Command Mode, Insert Mode, and Command-Line Mode.
- `head` displays top lines.
- `tail` displays bottom lines.
- `tail -f` is used for log monitoring.
- Pipes (`|`) connect commands together.
- Search and Replace is one of Vim's most powerful features.
- Vim is heavily used in DevOps, Linux Administration, Kubernetes, and Cloud environments.