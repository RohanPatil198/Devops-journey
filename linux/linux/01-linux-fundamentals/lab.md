 # Linux Fundamentals — Hands-on Lab

## Objective

Practice basic Linux commands for system identification, filesystem navigation, file and directory creation, file manipulation, and basic file inspection.

---

## Environment

| Item             | Details                 |
| ---------------- | ----------------------- |
| Operating System | Ubuntu Server 24.04 LTS |
| Environment      | VMware Virtual Machine  |
| Shell            | Bash                    |
| Access           | Local VM terminal       |

---

# Lab 1 — Identify the System

### Step 1: Check the current user

```bash
whoami
```

**Observed Result:**

```text
<!-- Add your actual output here -->
```

### Step 2: Check hostname

```bash
hostname
```

**Observed Result:**

```text
<!-- Add your actual hostname -->
```

### Step 3: Check OS information

```bash
cat /etc/os-release
```

**Observed Result:**

```text
<!-- Add relevant output -->
```

### Step 4: Check kernel version

```bash
uname -r
```

**Observed Result:**

```text
<!-- Add your actual output -->
```

### Learning

These commands help identify the current user, system hostname, Linux distribution, and kernel version.

---

# Lab 2 — Filesystem Navigation

### Step 1: Check current location

```bash
pwd
```

### Step 2: List the current directory

```bash
ls
```

### Step 3: Display detailed information

```bash
ls -la
```

### Step 4: Navigate to `/etc`

```bash
cd /etc
pwd
```

### Step 5: Navigate to `/var/log`

```bash
cd /var/log
pwd
```

### Step 6: Return to the home directory

```bash
cd ~
pwd
```

### Learning

Practiced navigation using absolute paths and the `~` home-directory shortcut.

---

# Lab 3 — Create a Directory Structure

Create a Linux practice environment:

```bash
mkdir -p ~/linux-fundamentals/{applications,logs,scripts,config}
```

Verify:

```bash
find ~/linux-fundamentals -type d
```

Expected structure:

```text
linux-fundamentals/
├── applications/
├── config/
├── logs/
└── scripts/
```

---

# Lab 4 — Create Files

Create log files:

```bash
touch ~/linux-fundamentals/logs/access.log
touch ~/linux-fundamentals/logs/error.log
```

Create scripts:

```bash
touch ~/linux-fundamentals/scripts/backup.sh
touch ~/linux-fundamentals/scripts/health-check.sh
```

Create configuration:

```bash
touch ~/linux-fundamentals/config/app.conf
```

Verify:

```bash
find ~/linux-fundamentals -type f
```

---

# Lab 5 — Add Content to a File

Create some sample application log data:

```bash
echo "INFO: Application started" > ~/linux-fundamentals/logs/access.log
echo "ERROR: Database connection failed" > ~/linux-fundamentals/logs/error.log
```

Read the files:

```bash
cat ~/linux-fundamentals/logs/access.log
cat ~/linux-fundamentals/logs/error.log
```

### Learning

`>` redirects command output into a file.

It creates the file if it does not exist and **overwrites existing content**.

This behavior will become important when we study shell scripting and redirection.

---

# Lab 6 — Append Data

Add another log entry:

```bash
echo "INFO: User login successful" >> ~/linux-fundamentals/logs/access.log
```

View the file:

```bash
cat ~/linux-fundamentals/logs/access.log
```

### Learning

`>>` appends data to the existing file instead of overwriting it.

Difference:

```text
>   → overwrite
>>  → append
```

---

# Lab 7 — Copy a File

Copy the error log:

```bash
cp ~/linux-fundamentals/logs/error.log ~/linux-fundamentals/applications/
```

Verify:

```bash
ls -l ~/linux-fundamentals/applications/
```

### Learning

`cp` creates a separate copy while leaving the original file unchanged.

---

# Lab 8 — Rename a File

Rename the copied file:

```bash
mv ~/linux-fundamentals/applications/error.log \
~/linux-fundamentals/applications/application-error.log
```

Verify:

```bash
ls -l ~/linux-fundamentals/applications/
```

### Learning

`mv` can be used to move a file or rename it.

---

# Lab 9 — Inspect File Information

Check the file type:

```bash
file ~/linux-fundamentals/applications/application-error.log
```

Check detailed metadata:

```bash
stat ~/linux-fundamentals/applications/application-error.log
```

Observe:

* File size
* Permissions
* Owner
* Group
* Timestamps
* Inode

---

# Lab 10 — Find Files

Find all files:

```bash
find ~/linux-fundamentals -type f
```

Find all directories:

```bash
find ~/linux-fundamentals -type d
```

Find the application error log:

```bash
find ~/linux-fundamentals -name "application-error.log"
```

### Learning

`find` is useful for locating files during administration and troubleshooting.

---

# Lab 11 — Remove a File

Remove the original error log:

```bash
rm ~/linux-fundamentals/logs/error.log
```

Verify:

```bash
ls ~/linux-fundamentals/logs/
```

The copied file should still exist:

```text
applications/application-error.log
```

### Learning

`rm` removes files. It should be used carefully because deleted files are generally not moved to a recycle bin.

---

# Lab 12 — Command Documentation

Use the Linux manual:

```bash
man ls
```

Read the available options.

Exit using:

```text
q
```

Then:

```bash
ls --help
```

Compare the quick help output with the manual page.

### Learning

Linux systems provide built-in documentation, so administrators do not need to memorize every command option.

---

# Lab 13 — Final Directory Structure

Run:

```bash
find ~/linux-fundamentals
```

The final structure should look approximately like:

```text
linux-fundamentals/
├── applications/
│   └── application-error.log
├── config/
│   └── app.conf
├── logs/
│   └── access.log
└── scripts/
    ├── backup.sh
    └── health-check.sh
```

---

# Practical Questions

After completing the lab, answer these in your own words.

### 1. What is the difference between `>` and `>>`?

**Answer:**

<!-- Write your answer -->

### 2. What is the difference between `cp` and `mv`?

**Answer:**

<!-- Write your answer -->

### 3. What is the difference between an absolute and relative path?

**Answer:**

<!-- Write your answer -->

### 4. What does `~` represent?

**Answer:**

<!-- Write your answer -->

### 5. Why would you use `less` instead of `cat`?

**Answer:**

<!-- Write your answer -->

### 6. Why is `rm` potentially dangerous?

**Answer:**

<!-- Write your answer -->

### 7. Why is `find` useful for a DevOps engineer?

**Answer:**

<!-- Write your answer -->

---

# DevOps Relevance

The commands practiced in this lab are foundational to Linux server administration.

They are commonly used when:

* Inspecting servers
* Navigating application directories
* Finding configuration files
* Investigating logs
* Managing application files
* Troubleshooting services
* Writing Bash scripts
* Working with Docker containers
* Managing cloud Linux instances

---

# Completion Checklist

* [ ] Identified Linux user
* [ ] Identified hostname
* [ ] Identified OS version
* [ ] Identified kernel version
* [ ] Practiced filesystem navigation
* [ ] Created directories
* [ ] Created files
* [ ] Read file contents
* [ ] Used output redirection
* [ ] Copied files
* [ ] Moved/renamed files
* [ ] Inspected file metadata
* [ ] Found files using `find`
* [ ] Removed a file
* [ ] Used `man`
* [ ] Used `--help`

## Status

**Theory:** Completed

**Hands-on:** Completed after executing all exercises

**Environment:** Ubuntu Server 24.04 LTS on VMware

**Next Topic:** Linux Filesystem in Detail

