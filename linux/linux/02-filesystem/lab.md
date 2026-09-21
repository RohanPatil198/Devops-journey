# Linux Filesystem — Hands-on Lab

## Objective

In this lab, we will investigate the Linux filesystem on the Ubuntu Server VM.

You will practice:

* Filesystem hierarchy
* Important directories
* Disk usage
* Mounted filesystems
* Disks and partitions
* Filesystem types
* UUIDs
* Inodes
* File metadata
* Searching files
* Disk-space investigation
* Virtual filesystems

---

# Environment

**OS:** Ubuntu Server 24.04 LTS
**Virtualization:** VMware Workstation
**Shell:** Bash

Before starting:

```bash
whoami
hostname
pwd
```

Record your actual output in this document.

---

# Lab 1 — Explore the Root Filesystem

## Step 1

Run:

```bash
ls /
```

### What does `/` mean?

`/` is the root of the Linux filesystem hierarchy.

You should see directories such as:

```text
boot
dev
etc
home
proc
root
run
sys
tmp
usr
var
```

## Step 2

Get detailed information:

```bash
ls -la /
```

### Why `-la`?

```text
-l → long listing
-a → include hidden entries
```

Observe:

* Permissions
* Owner
* Group
* Size
* Directories

---

# Lab 2 — Investigate Important Locations

Run each command and observe the output.

## `/etc`

```bash
ls -la /etc | head
```

Purpose:

```text
System and application configuration.
```

## `/var`

```bash
ls -la /var
```

Purpose:

```text
Variable system/application data.
```

## `/var/log`

```bash
ls -lah /var/log
```

Purpose:

```text
System and application logs.
```

## `/home`

```bash
ls -la /home
```

Purpose:

```text
Normal user home directories.
```

## `/tmp`

```bash
ls -la /tmp
```

Purpose:

```text
Temporary files.
```

---

# Lab 3 — Check Filesystem Types and Disk Usage

Run:

```bash
df -h
```

Record:

* Total size
* Used
* Available
* Usage %
* Mount point

Then run:

```bash
df -Th
```

### Questions

1. What filesystem type is your `/` filesystem?
2. How much total space does it have?
3. How much is currently used?
4. What is the mount point?

Record your actual output below:

```text
Output:

[Paste your output here]
```

---

# Lab 4 — Investigate Your Disks

Run:

```bash
lsblk
```

Then:

```bash
lsblk -f
```

### Understand the hierarchy

Identify:

```text
Disk
↓
Partition
↓
Filesystem
↓
Mount point
```

For example:

```text
sda
├── sda1
└── sda2
```

Your VM may use a different device name, such as:

```text
sda
nvme0n1
```

Do not assume the device name.

### Record

```text
Disk:

Partitions:

Filesystem:

Mount point:
```

---

# Lab 5 — Investigate Mounts

Run:

```bash
findmnt
```

This shows the mounted filesystem hierarchy.

Now investigate the filesystem containing `/`:

```bash
findmnt -T /
```

And `/var/log`:

```bash
findmnt -T /var/log
```

### Question

Are `/` and `/var/log` on the same filesystem?

Record:

```text
Answer:
```

---

# Lab 6 — Check UUID and Filesystem Information

Run:

```bash
sudo blkid
```

You may see:

```text
/dev/sda2: UUID="..." TYPE="ext4"
```

Identify:

```text
Device:
UUID:
Filesystem type:
```

### Why is UUID important?

UUIDs allow Linux to identify filesystems reliably, especially for persistent mounting through:

```text
/etc/fstab
```

---

# Lab 7 — Investigate Inodes

Create a test directory:

```bash
mkdir -p ~/filesystem-lab/inodes
```

Create files:

```bash
touch ~/filesystem-lab/inodes/file1.txt
touch ~/filesystem-lab/inodes/file2.txt
```

Now:

```bash
ls -li ~/filesystem-lab/inodes
```

You should see inode numbers.

Example:

```text
123456 -rw-r--r-- ... file1.txt
123457 -rw-r--r-- ... file2.txt
```

### Questions

1. What is the inode number of `file1.txt`?
2. What is the inode number of `file2.txt`?
3. Are they different?

Record your actual results.

---

# Lab 8 — Investigate File Metadata

Run:

```bash
stat ~/filesystem-lab/inodes/file1.txt
```

Identify:

```text
File type:
Size:
Inode:
Permissions:
UID:
GID:
Access time:
Modify time:
Change time:
```

### Important

Understand the difference:

```text
Access
→ File was accessed.

Modify
→ File contents changed.

Change
→ File metadata changed.
```

---

# Lab 9 — Determine File Type

Create a few test files:

```bash
touch ~/filesystem-lab/test.txt
echo "Hello Linux" > ~/filesystem-lab/test.txt
```

Now:

```bash
file ~/filesystem-lab/test.txt
```

Also test:

```bash
file /etc/passwd
file /bin/bash
```

### Question

Does `file` simply depend on the filename extension?

Explain your answer:

```text
Answer:
```

---

# Lab 10 — Search the Filesystem

Create test structure:

```bash
mkdir -p ~/filesystem-lab/search/{logs,configs,scripts}
```

Create files:

```bash
touch ~/filesystem-lab/search/logs/app.log
touch ~/filesystem-lab/search/logs/error.log
touch ~/filesystem-lab/search/configs/app.conf
touch ~/filesystem-lab/search/scripts/backup.sh
```

Find all files:

```bash
find ~/filesystem-lab/search -type f
```

Find directories:

```bash
find ~/filesystem-lab/search -type d
```

Find `.log` files:

```bash
find ~/filesystem-lab/search -type f -name "*.log"
```

Find `.conf` files:

```bash
find ~/filesystem-lab/search -type f -name "*.conf"
```

---

# Lab 11 — Understand `du`

Check your lab directory:

```bash
du -sh ~/filesystem-lab
```

### Breakdown

```text
du → disk usage
-s → summary
-h → human-readable
```

Now:

```bash
du -ah ~/filesystem-lab
```

Compare the output.

### Question

What is the difference between:

```bash
du -sh
```

and:

```bash
du -ah
```

Record your answer.

---

# Lab 12 — Investigate `/var`

This is an important real-world troubleshooting exercise.

Run:

```bash
sudo du -sh /var/* 2>/dev/null | sort -h
```

### Breakdown

```text
sudo
→ Execute with elevated privileges.

du
→ Calculate disk usage.

-s
→ Show summary.

-h
→ Human-readable.

 /var/*
→ Examine items directly inside /var.

2>/dev/null
→ Hide permission/error messages.

|
→ Send output to the next command.

sort -h
→ Sort sizes in human-readable form.
```

### Question

Which directory under `/var` consumes the most space on your VM?

Record:

```text
Largest directory:
Size:
```

---

# Lab 13 — Investigate `/proc`

Remember:

`/proc` is a virtual filesystem.

Run:

```bash
ls /proc
```

Now:

```bash
cat /proc/cpuinfo
```

Then:

```bash
cat /proc/meminfo
```

Check the kernel version:

```bash
cat /proc/version
```

### Questions

1. Is `/proc` a normal disk directory?
2. What information does `/proc/cpuinfo` provide?
3. What information does `/proc/meminfo` provide?

---

# Lab 14 — Investigate `/sys`

Run:

```bash
ls /sys
```

Then:

```bash
ls /sys/class
```

You are observing information exposed by the Linux kernel about hardware and system devices.

Do not modify files under `/sys` during this lab.

---

# Lab 15 — Investigate `/dev`

Run:

```bash
ls -la /dev | head -30
```

Look for special devices such as:

```text
/dev/null
/dev/zero
/dev/random
```

Test `/dev/null`:

```bash
echo "This will disappear" > /dev/null
```

Nothing should be displayed.

Now:

```bash
echo "Hello" > /tmp/test-output.txt
cat /tmp/test-output.txt
```

Compare this with `/dev/null`.

### Question

Why is `/dev/null` useful in shell scripting and troubleshooting?

---

# Lab 16 — Practical Disk Investigation Challenge

### Scenario

Imagine your Ubuntu server reports:

```text
No space left on device
```

You need to investigate.

### Step 1

```bash
df -h
```

Identify the filesystem with the highest usage.

### Step 2

```bash
df -Th
```

Identify its filesystem type.

### Step 3

If `/` is the affected filesystem:

```bash
sudo du -sh /* 2>/dev/null | sort -h
```

### Step 4

If `/var` appears large:

```bash
sudo du -sh /var/* 2>/dev/null | sort -h
```

### Step 5

If `/var/log` appears large:

```bash
sudo du -sh /var/log/*
```

### Step 6

Find large log files:

```bash
sudo find /var/log -type f -size +50M -ls
```

### Your investigation

Do **not** delete anything.

Document:

```text
Filesystem investigated:

Usage:

Largest directory:

Largest file found:

Possible reason:

Recommended next investigation:
```

This is the kind of troubleshooting evidence you should eventually be comfortable discussing in an interview.

---

# Lab 17 — Cleanup

Remove only the directories created by this lab:

```bash
rm -rf ~/filesystem-lab
```

Before executing `rm -rf`, verify the path carefully:

```bash
ls -la ~/filesystem-lab
```

Then remove it.

### Important warning

Never casually run:

```bash
rm -rf /
```

or use `rm -rf` on an unknown path.

Always verify the target before deletion.

---

# Challenges

Try these without looking at the command reference.

### Challenge 1

Find the filesystem containing `/var/log`.

### Challenge 2

Display filesystem type and disk usage in human-readable format.

### Challenge 3

Find all `.log` files under `/var/log`.

### Challenge 4

Display the inode number of `/etc/hostname`.

### Challenge 5

Find directories under `/etc`.

### Challenge 6

Find files larger than 10 MB under `/var`.

### Challenge 7

Determine which directory directly under `/var` consumes the most space.

### Challenge 8

Identify the filesystem and mount point of your VM's root filesystem.

---

# Troubleshooting Questions

Answer these in your own words:

### Q1. What is the difference between `/` and `/root`?

### Q2. What is the difference between `/etc` and `/var`?

### Q3. What is the difference between `df` and `du`?

### Q4. What is a mount point?

### Q5. What is an inode?

### Q6. Why is `/proc` called a virtual filesystem?

### Q7. Why does Linux have `/dev`?

### Q8. What is the difference between a disk, partition, filesystem, and mount point?

### Q9. Why are UUIDs useful?

### Q10. How would you investigate a "No space left on device" problem?

---

# GitHub Documentation

Do not simply paste example outputs from this document.

Replace the sections requiring output with the **actual output from your Ubuntu VM**.

Your final directory should be:

```text
linux/
└── 02-filesystem/
    ├── README.md
    ├── commands.md
    └── lab.md
```

Your `lab.md` should contain evidence of your actual work.

---

# Completion Checklist

* [ ] Explored `/`
* [ ] Investigated `/etc`
* [ ] Investigated `/var`
* [ ] Investigated `/var/log`
* [ ] Used `df`
* [ ] Used `du`
* [ ] Used `lsblk`
* [ ] Used `blkid`
* [ ] Used `findmnt`
* [ ] Investigated inodes
* [ ] Used `stat`
* [ ] Used `file`
* [ ] Used `find`
* [ ] Investigated `/proc`
* [ ] Investigated `/sys`
* [ ] Investigated `/dev`
* [ ] Completed disk investigation challenge
* [ ] Completed troubleshooting questions
* [ ] Documented actual outputs
* [ ] Cleaned up lab files

---

# Git Commit

After completing the lab and replacing the placeholders with your actual results:

```bash
cd ~/devops-journey

git add linux/02-filesystem/

git commit -m "Add Linux filesystem commands and hands-on lab"

git push
```

The commit history now provides evidence that you didn't just study filesystem theory—you practiced it on your Ubuntu VM.

