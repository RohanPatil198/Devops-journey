 # Linux Fundamentals — Command Reference

This document contains the Linux commands practiced during the fundamentals stage.

---

## 1. System and User Information

### `whoami`

Shows the currently logged-in user.

```bash
whoami
```

### `hostname`

Displays the system hostname.

```bash
hostname
```

### `uname`

Displays information about the Linux kernel.

```bash
uname -a
```

Display only the kernel release:

```bash
uname -r
```

### `cat /etc/os-release`

Displays Linux distribution information.

```bash
cat /etc/os-release
```

---

## 2. Navigation

### `pwd`

Displays the current working directory.

```bash
pwd
```

### `ls`

Lists files and directories.

```bash
ls
```

Useful options:

```bash
ls -l
ls -a
ls -la
ls -lh
```

* `-l` → detailed listing
* `-a` → include hidden files
* `-h` → human-readable sizes

### `cd`

Changes the current directory.

```bash
cd /etc
cd ~
cd ..
cd -
```

* `/etc` → absolute path
* `~` → user's home directory
* `..` → parent directory
* `-` → previous directory

---

## 3. Creating Files and Directories

### `mkdir`

Creates a directory.

```bash
mkdir test
```

Create multiple directories:

```bash
mkdir logs scripts config
```

Create nested directories:

```bash
mkdir -p project/app/config
```

### `touch`

Creates an empty file.

```bash
touch notes.txt
```

Create multiple files:

```bash
touch file1.txt file2.txt file3.txt
```

---

## 4. Viewing Files

### `cat`

Displays the contents of a file.

```bash
cat notes.txt
```

### `less`

Used to view large files interactively.

```bash
less /var/log/syslog
```

Press `q` to exit.

### `head`

Displays the beginning of a file.

```bash
head notes.txt
```

Display the first 20 lines:

```bash
head -n 20 notes.txt
```

### `tail`

Displays the end of a file.

```bash
tail notes.txt
```

Display the last 20 lines:

```bash
tail -n 20 notes.txt
```

Follow a log file:

```bash
tail -f application.log
```

`-f` continuously displays newly added lines.

---

## 5. Copying, Moving and Removing

### `cp`

Copies a file.

```bash
cp source.txt destination.txt
```

Copy a directory recursively:

```bash
cp -r source/ destination/
```

### `mv`

Moves or renames files/directories.

Rename:

```bash
mv old.txt new.txt
```

Move:

```bash
mv file.txt /tmp/
```

### `rm`

Removes a file.

```bash
rm file.txt
```

Remove a directory and its contents:

```bash
rm -r directory/
```

**Use `rm` carefully because command-line deletion generally does not use a recycle bin.**

---

## 6. Finding Files

### `find`

Searches for files and directories.

Find files in the current directory:

```bash
find . -type f
```

Find directories:

```bash
find . -type d
```

Find a specific filename:

```bash
find . -name "error.log"
```

---

## 7. File Information

### `file`

Identifies the type of a file.

```bash
file notes.txt
```

### `stat`

Displays detailed file metadata.

```bash
stat notes.txt
```

Information includes:

* Size
* Permissions
* Owner
* Inode
* Access time
* Modification time
* Change time

---

## 8. Command Help

### `man`

Displays the manual page for a command.

```bash
man ls
```

Examples:

```bash
man cp
man mv
man mkdir
```

Press `q` to exit.

### `--help`

Provides quick command usage information.

```bash
ls --help
```

---

## 9. Locating Commands

### `which`

Shows the path of an executable.

```bash
which ls
```

Example:

```text
/usr/bin/ls
```

### `type`

Shows how Bash interprets a command.

```bash
type ls
type cd
```

This can identify aliases, builtins, functions, and external commands.

---

## 10. Terminal

### `clear`

Clears the visible terminal screen.

```bash
clear
```

It does not delete files or change system data.

---

## Quick Reference

| Command    | Purpose                  |
| ---------- | ------------------------ |
| `whoami`   | Show current user        |
| `hostname` | Show system hostname     |
| `uname`    | Show kernel information  |
| `pwd`      | Show current directory   |
| `ls`       | List files/directories   |
| `cd`       | Change directory         |
| `mkdir`    | Create directory         |
| `touch`    | Create file              |
| `cat`      | Display file contents    |
| `less`     | View files interactively |
| `head`     | Show beginning of file   |
| `tail`     | Show end of file         |
| `cp`       | Copy files/directories   |
| `mv`       | Move/rename              |
| `rm`       | Remove files/directories |
| `find`     | Find files/directories   |
| `file`     | Identify file type       |
| `stat`     | Show file metadata       |
| `man`      | Manual/documentation     |
| `which`    | Locate executable        |
| `type`     | Identify command type    |
| `clear`    | Clear terminal           |

---

## Important Concepts

Before moving to the next topic, understand:

* Absolute vs relative paths
* Current directory `.`
* Parent directory `..`
* Home directory `~`
* Hidden files
* Files vs directories
* Command options and arguments
* Relative vs absolute navigation
* Copy vs move
* File metadata
* Using built-in Linux documentation

