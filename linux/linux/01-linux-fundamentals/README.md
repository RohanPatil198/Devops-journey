 # Linux Fundamentals

## Objective

Understand how to work with the Linux command line and perform basic filesystem operations.

This section covers the fundamental commands used to navigate the Linux system, inspect files and directories, create and modify files, and understand the current working environment.

These commands form the foundation for Linux administration, scripting, troubleshooting, Docker, Kubernetes, and DevOps.

---

# 1. Command Line Interface

Linux servers are commonly managed through a **Command Line Interface (CLI)** instead of a graphical interface.

A user enters a command into a shell, the shell interprets the command, and Linux performs the requested operation.

```text
User
  ↓
Shell
  ↓
Linux Kernel
  ↓
System Resources
```

For this learning journey, the primary shell is **Bash**.

---

# 2. Basic Command Structure

A Linux command generally follows this structure:

```text
command [options] [arguments]
```

For example:

```bash
ls -l /var/log
```

Here:

* `ls` → command
* `-l` → option
* `/var/log` → argument

Not every command requires options or arguments.

Example:

```bash
pwd
```

---

# 3. pwd — Print Working Directory

`pwd` displays the directory in which you are currently working.

```bash
pwd
```

Example output:

```text
/home/rohan
```

This is useful because many Linux commands operate relative to the current directory.

---

# 4. ls — List Directory Contents

`ls` displays files and directories.

```bash
ls
```

Useful options:

```bash
ls -l
```

Displays detailed information.

```bash
ls -a
```

Displays hidden files.

```bash
ls -la
```

Displays hidden files in detailed format.

Example:

```text
drwxr-xr-x 2 rohan rohan 4096 Sep 1 Documents
-rw-r--r-- 1 rohan rohan  120 Sep 1 notes.txt
```

The detailed output contains information such as permissions, owner, group, size, modification time, and filename.

Permissions and ownership will be studied separately.

---

# 5. cd — Change Directory

`cd` is used to move between directories.

Example:

```bash
cd /etc
```

Move to your home directory:

```bash
cd ~
```

You can also simply use:

```bash
cd
```

Move to the parent directory:

```bash
cd ..
```

Move to the previous directory:

```bash
cd -
```

Understanding `cd` and Linux paths is essential because most Linux administration work happens from the terminal.

---

# 6. mkdir — Create Directories

`mkdir` creates a directory.

```bash
mkdir devops
```

Create multiple directories:

```bash
mkdir linux docker networking
```

Create nested directories:

```bash
mkdir -p devops/linux/labs
```

The `-p` option creates parent directories when they do not already exist.

---

# 7. touch — Create Files

`touch` can create an empty file.

```bash
touch notes.txt
```

Multiple files can be created:

```bash
touch file1.txt file2.txt file3.txt
```

`touch` can also update a file's timestamps if the file already exists.

---

# 8. cat — Display File Contents

`cat` displays the contents of a file.

```bash
cat notes.txt
```

It can also be used to combine files, although its most common beginner use is viewing small text files.

---

# 9. less — View Large Files

For larger files, `less` is more practical than `cat`.

```bash
less /var/log/syslog
```

You can move through the file without loading the entire content into your terminal.

Useful keys:

```text
↑ / ↓    Move
Space    Next page
b        Previous page
q        Quit
```

Log analysis is an important DevOps skill, so becoming comfortable with `less` is useful.

---

# 10. head and tail

`head` displays the beginning of a file.

```bash
head notes.txt
```

Display the first 20 lines:

```bash
head -n 20 notes.txt
```

`tail` displays the end of a file.

```bash
tail notes.txt
```

Display the last 20 lines:

```bash
tail -n 20 notes.txt
```

A particularly useful option for monitoring logs is:

```bash
tail -f application.log
```

`-f` follows the file and displays new lines as they are added.

This is frequently useful when troubleshooting applications and services.

---

# 11. cp — Copy Files

`cp` copies files or directories.

Example:

```bash
cp notes.txt backup.txt
```

This creates a copy named `backup.txt`.

To copy a directory and its contents:

```bash
cp -r source destination
```

The `-r` option means recursive.

---

# 12. mv — Move or Rename

`mv` is used to move or rename files and directories.

Rename a file:

```bash
mv old.txt new.txt
```

Move a file:

```bash
mv notes.txt /tmp/
```

Unlike `cp`, `mv` does not leave the original in its previous location.

---

# 13. rm — Remove

`rm` removes files.

```bash
rm file.txt
```

Remove an empty directory:

```bash
rmdir directory
```

Remove a directory and its contents:

```bash
rm -r directory
```

Be careful with `rm`. Command-line deletion generally does not provide the same recovery mechanism as a graphical recycle bin.

Avoid using:

```bash
rm -rf
```

unless you fully understand the command and the path being targeted.

---

# 14. clear

`clear` clears the terminal screen.

```bash
clear
```

This does not delete files or affect the system. It only clears the visible terminal output.

---

# 15. man — Manual Pages

Linux provides built-in documentation through manual pages.

For example:

```bash
man ls
```

You can use this to understand available options and command behavior.

Other examples:

```bash
man cp
man mv
man mkdir
```

Learning to use `man` is important because Linux systems contain extensive built-in documentation.

---

# 16. --help

Many commands also provide a shorter help screen:

```bash
ls --help
```

or:

```bash
cp --help
```

`man` generally provides more detailed documentation, while `--help` is useful for quickly checking syntax and options.

---

# 17. Finding Commands

Linux provides commands to identify where programs are located.

```bash
which ls
```

Example:

```text
/usr/bin/ls
```

You can also use:

```bash
type ls
```

`type` can tell you whether something is an alias, shell builtin, function, or external command.

---

# 18. File and Directory Information

`file` determines the type of a file.

```bash
file notes.txt
```

Example:

```text
notes.txt: ASCII text
```

`stat` provides detailed metadata:

```bash
stat notes.txt
```

This can show:

* File size
* Permissions
* Owner
* Timestamps
* Inode information

---

# 19. Useful Command Combinations

Linux commands become more powerful when combined.

For example:

```bash
ls -lah /var/log
```

This gives a detailed listing including hidden files and human-readable sizes.

Another example:

```bash
find . -type f
```

This finds regular files under the current directory.

The important idea is not memorizing hundreds of commands. It is learning how to combine a small set of commands to investigate and solve problems.

---

# 20. Hands-on Practice

Create a practice environment:

```bash
mkdir -p ~/linux-fundamentals/{applications,logs,scripts,config}
```

Create files:

```bash
touch ~/linux-fundamentals/logs/access.log
touch ~/linux-fundamentals/logs/error.log
touch ~/linux-fundamentals/scripts/backup.sh
touch ~/linux-fundamentals/scripts/health-check.sh
touch ~/linux-fundamentals/config/app.conf
```

Check the structure:

```bash
find ~/linux-fundamentals
```

Copy a file:

```bash
cp ~/linux-fundamentals/logs/error.log ~/linux-fundamentals/applications/
```

Rename it:

```bash
mv ~/linux-fundamentals/applications/error.log ~/linux-fundamentals/applications/application-error.log
```

Check:

```bash
ls -l ~/linux-fundamentals/applications
```

---

# 21. DevOps Relevance

These commands may appear simple, but they are used constantly in real DevOps environments.

For example, when troubleshooting a server you may need to:

```text
Find where you are
      ↓
Inspect directories
      ↓
Find configuration files
      ↓
Read logs
      ↓
Check files
      ↓
Modify configuration
      ↓
Verify the result
```

Commands such as:

```bash
pwd
ls
cd
find
cat
less
head
tail
cp
mv
rm
```

are therefore basic building blocks for server administration and troubleshooting.

---

# 22. Key Takeaways

* The Linux CLI is a primary interface for managing Linux servers.
* `pwd` shows the current working directory.
* `ls` lists files and directories.
* `cd` changes the current directory.
* `mkdir` creates directories.
* `touch` creates empty files.
* `cat` displays file contents.
* `less` is useful for viewing large files.
* `head` and `tail` inspect parts of files.
* `cp` copies files/directories.
* `mv` moves or renames files/directories.
* `rm` removes files/directories.
* `man` provides detailed command documentation.
* Linux commands can be combined to perform administration and troubleshooting tasks.

## Status

**Theory:** Completed

**Hands-on:** In Progress

**Environment:** Ubuntu Server 24.04 LTS on VMware

**Next:** Users, Groups, sudo and File Permissions

