# Linux Filesystem — Commands Reference

This document contains important Linux filesystem commands used for system administration, DevOps, troubleshooting, storage management, and system investigation.

---

## 1. `pwd`

### Purpose

`pwd` means **Print Working Directory**.

It displays the absolute path of the directory you are currently inside.

### Syntax

```bash
pwd [OPTION]
```

### Common option

```bash
-L
```

Display the logical path, including symbolic links.

```bash
-P
```

Display the physical path and resolve symbolic links.

### Example

```bash
pwd
```

Example output:

```text
/home/rohan/devops-journey
```

### Important concept

Your shell always has a **current working directory**.

Commands such as:

```bash
ls
cat file.txt
```

may operate relative to this directory.

### DevOps use

Useful when scripts or commands depend on the current location.

---

# 2. `ls`

### Purpose

Lists files and directories.

### Syntax

```bash
ls [OPTION] [FILE/DIRECTORY]
```

### Important options

### `-l` — Long listing

```bash
ls -l
```

Shows:

* Permissions
* Number of links
* Owner
* Group
* Size
* Modification time
* Filename

Example:

```text
-rw-r--r-- 1 rohan rohan 120 Sep 3 10:30 app.conf
```

---

### `-a` — All files

```bash
ls -a
```

Shows hidden files.

Linux hidden files generally begin with:

```text
.
```

Examples:

```text
.bashrc
.profile
.git
```

---

### `-h` — Human-readable sizes

Usually combined with `-l`:

```bash
ls -lh
```

Instead of:

```text
1048576
```

you may see:

```text
1.0M
```

---

### `-i` — Inode number

```bash
ls -li
```

Displays the inode number of each file.

---

### `-d` — List directory itself

```bash
ls -ld /etc
```

Without `-d`, `ls` can list the contents of `/etc`.

With `-d`, it displays information about `/etc` itself.

---

### `-R` — Recursive listing

```bash
ls -R directory/
```

Lists the directory and its subdirectories recursively.

Use carefully on large directories.

### Useful combinations

```bash
ls -la
ls -lh
ls -li
ls -ld /var/log
```

---

# 3. `cd`

### Purpose

Changes the current working directory.

### Syntax

```bash
cd [DIRECTORY]
```

### Examples

```bash
cd /etc
```

Go to `/etc`.

```bash
cd /var/log
```

Go to the system log directory.

```bash
cd ..
```

Go to the parent directory.

```bash
cd ~
```

Go to the current user's home directory.

```bash
cd -
```

Return to the previous directory.

```bash
cd /
```

Go to the filesystem root.

### Important locations

```text
/       → filesystem root
~       → current user's home
..      → parent directory
.       → current directory
```

---

# 4. `df`

### Purpose

Reports **filesystem disk-space usage**.

This is different from `du`, which we'll see later.

### Syntax

```bash
df [OPTION] [FILE]
```

### `-h` — Human-readable

```bash
df -h
```

Displays sizes such as:

```text
1.8G
20G
500M
```

instead of raw blocks.

### `-T` — Show filesystem type

```bash
df -T
```

Shows types such as:

```text
ext4
tmpfs
```

### Combined

```bash
df -Th
```

This is one of the most useful filesystem commands.

It shows:

* Filesystem
* Type
* Total size
* Used space
* Available space
* Usage percentage
* Mount point

### Example

```text
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/sda2      ext4   50G   20G   28G  42% /
```

### DevOps use

When a server reports:

```text
No space left on device
```

`df -h` is one of the first commands you should run.

---

# 5. `du`

### Purpose

`du` means **Disk Usage**.

It estimates how much space files and directories are consuming.

### Syntax

```bash
du [OPTION] [FILE/DIRECTORY]
```

### `-h` — Human-readable

```bash
du -h
```

### `-s` — Summary

```bash
du -sh /var/log
```

Instead of showing every file, it gives a summary.

### `-a` — All files

```bash
du -ah directory/
```

Includes individual files.

### Example

```bash
du -sh /var/log
```

Possible output:

```text
450M    /var/log
```

### `df` vs `du`

This distinction is very important.

```text
df → How much space is used on the filesystem?

du → Which files/directories are consuming space?
```

Typical investigation:

```bash
df -h
du -sh /var/*
```

---

# 6. `findmnt`

### Purpose

Displays mounted filesystems.

### Syntax

```bash
findmnt [OPTION] [TARGET]
```

### Basic

```bash
findmnt
```

Shows the filesystem mount hierarchy.

### `-t` — Filter by filesystem type

```bash
findmnt -t ext4
```

Shows ext4 filesystems.

### `-T` — Find filesystem containing a path

```bash
findmnt -T /var/log
```

This tells you which filesystem contains `/var/log`.

### DevOps use

Very useful when troubleshooting:

* Mounted disks
* Storage
* Containers
* Separate filesystems
* Application mount points

---

# 7. `lsblk`

### Purpose

Lists block devices such as disks and partitions.

`lsblk` means **list block devices**.

### Syntax

```bash
lsblk [OPTION]
```

### Basic

```bash
lsblk
```

Example:

```text
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda           8:0    0   50G  0 disk
├─sda1        8:1    0    1G  0 part /boot
└─sda2        8:2    0   49G  0 part /
```

### `-f` — Filesystem information

```bash
lsblk -f
```

Shows:

* Filesystem type
* UUID
* Mount point

### `-o` — Select columns

```bash
lsblk -o NAME,SIZE,FSTYPE,MOUNTPOINTS
```

This allows you to select the information you want.

### Important concept

A typical storage hierarchy can look like:

```text
Physical disk
    ↓
Partition
    ↓
Filesystem
    ↓
Mount point
```

---

# 8. `blkid`

### Purpose

Displays block-device attributes.

Most commonly used to identify:

* UUID
* Filesystem type
* Device

### Syntax

```bash
blkid [DEVICE]
```

### Example

```bash
sudo blkid
```

Example:

```text
/dev/sda2: UUID="xxxx-xxxx" TYPE="ext4"
```

### Why UUID matters

Linux can identify a filesystem using its UUID instead of relying only on `/dev/sda2`.

UUIDs are commonly used in:

```text
/etc/fstab
```

for persistent mounts.

---

# 9. `mount`

### Purpose

Attaches a filesystem to the Linux directory tree.

### Syntax

```bash
mount [OPTION] [DEVICE] [DIRECTORY]
```

Conceptually:

```bash
mount DEVICE DIRECTORY
```

Example:

```bash
sudo mount /dev/sdb1 /mnt
```

This attaches `/dev/sdb1` at:

```text
/mnt
```

### `-t` — Specify filesystem type

```bash
sudo mount -t ext4 /dev/sdb1 /mnt
```

Normally Linux can detect the filesystem automatically.

### `-o` — Mount options

```bash
mount -o OPTION
```

Mount options control how the filesystem is mounted.

Do not experiment with write-related options on important filesystems without understanding their effects.

### Important

The target directory must exist:

```bash
sudo mkdir /mnt/data
```

Then:

```bash
sudo mount /dev/sdb1 /mnt/data
```

---

# 10. `umount`

### Purpose

Unmounts a mounted filesystem.

Notice the command is:

```bash
umount
```

not:

```bash
unmount
```

### Syntax

```bash
umount [OPTION] [DEVICE|MOUNTPOINT]
```

Examples:

```bash
sudo umount /mnt/data
```

or:

```bash
sudo umount /dev/sdb1
```

### Common problem

You may receive:

```text
target is busy
```

This means something is still using the mount point.

Investigate with:

```bash
sudo lsof +D /mnt/data
```

or:

```bash
sudo fuser -vm /mnt/data
```

Don't immediately force-unmount a filesystem in production.

---

# 11. `stat`

### Purpose

Displays detailed file or filesystem metadata.

### Syntax

```bash
stat [OPTION] FILE
```

### Example

```bash
stat /etc/hostname
```

Information can include:

* File type
* Size
* Inode
* Permissions
* UID
* GID
* Access time
* Modification time
* Change time

### Important timestamps

```text
Access  → Last time file was accessed
Modify  → Last time file contents changed
Change  → Last time metadata changed
```

---

# 12. `file`

### Purpose

Determines the type of a file.

### Syntax

```bash
file [OPTION] FILE
```

Example:

```bash
file /etc/passwd
```

You can also inspect your own files:

```bash
file ~/linux-fundamentals/scripts/backup.sh
```

### Important concept

The extension is not what determines a Linux file's actual type.

For example:

```text
script.sh
```

doesn't automatically mean Linux considers it a shell script.

`file` examines the file contents/signature.

---

# 13. `ls -li` and Inodes

### Command

```bash
ls -li
```

### `-l`

Long listing.

### `-i`

Display inode number.

Example:

```text
123456 -rw-r--r-- 1 rohan rohan 50 Sep 3 test.txt
```

Here:

```text
123456
```

is the inode number.

### Why useful?

Inodes become important when investigating:

* File metadata
* Hard links
* Deleted files
* Filesystem limitations
* Disk issues

---

# 14. `find`

### Purpose

Searches for files and directories.

### Syntax

```bash
find [PATH] [EXPRESSION]
```

### Search files

```bash
find /etc -type f
```

### `-type f`

Search regular files.

### `-type d`

Search directories.

```bash
find /etc -type d
```

### `-name`

Search by name.

```bash
find /etc -name "hostname"
```

Case-sensitive.

### `-iname`

Case-insensitive name search.

```bash
find /etc -iname "HOSTNAME"
```

### Search by size

```bash
find /var/log -type f -size +100M
```

Find regular files larger than 100 MB.

### Search by modification time

```bash
find /var/log -type f -mtime -1
```

Find files modified within approximately the last 24 hours.

### DevOps use

Very useful for finding:

* Large logs
* Configuration files
* Old files
* Application artifacts
* Backup files

---

# 15. `du` + `sort`

A common disk investigation technique:

```bash
sudo du -sh /var/* 2>/dev/null | sort -h
```

### Breakdown

```text
sudo
→ Run with elevated privileges.

du
→ Calculate disk usage.

-s
→ Summary for each supplied item.

-h
→ Human-readable output.

/var/*
→ Examine items directly inside /var.

2>/dev/null
→ Redirect error messages to /dev/null.

|
→ Pipe output to another command.

sort -h
→ Sort human-readable sizes correctly.
```

This is a very useful real-world troubleshooting command.

---

# 16. `/dev/null`

`/dev/null` is a special device that discards data written to it.

Example:

```bash
echo "test" > /dev/null
```

The output is discarded.

### Common usage

```bash
command 2>/dev/null
```

This redirects standard error to `/dev/null`.

### Important

```text
/dev/null
```

does **not** mean a normal empty directory or file.

It is a special device provided by Linux.

---

# 17. Redirection

### `>`

Redirects standard output and **overwrites** the destination.

```bash
echo "hello" > test.txt
```

### `>>`

Redirects standard output and **appends**.

```bash
echo "second line" >> test.txt
```

### `2>`

Redirects standard error.

```bash
command 2> error.log
```

### `2>/dev/null`

Discard errors.

```bash
find / -name "*.log" 2>/dev/null
```

---

# 18. Pipe `|`

A pipe sends the output of one command to another command.

Example:

```bash
ls -l /etc | less
```

Another useful example:

```bash
df -h | grep "^/dev"
```

Concept:

```text
Command A
   ↓
   |
   ↓
Command B
```

---

# 19. Important filesystem locations

### `/`

Root of the filesystem.

### `/etc`

Configuration.

### `/var`

Variable data.

### `/var/log`

Logs.

### `/home`

Normal users' home directories.

### `/root`

Root user's home directory.

### `/tmp`

Temporary files.

### `/usr`

Most user-space applications, libraries and documentation.

### `/opt`

Optional/third-party software.

### `/dev`

Device files.

### `/proc`

Virtual process/kernel information.

### `/sys`

Virtual kernel/hardware information.

### `/run`

Runtime data created since boot.

### `/mnt`

Common temporary mount location.

---

# Quick Reference

| Command   | Main purpose                     |
| --------- | -------------------------------- |
| `pwd`     | Show current directory           |
| `ls`      | List files                       |
| `cd`      | Change directory                 |
| `df`      | Filesystem space                 |
| `du`      | Directory/file space usage       |
| `findmnt` | Show mounts                      |
| `lsblk`   | Show disks/partitions            |
| `blkid`   | Show UUID/filesystem information |
| `mount`   | Mount filesystem                 |
| `umount`  | Unmount filesystem               |
| `stat`    | File metadata                    |
| `file`    | Identify file type               |
| `find`    | Search files/directories         |
| `ls -li`  | Show inode                       |
| `sort -h` | Sort human-readable sizes        |
| `lsof`    | Show open files                  |
| `fuser`   | Show processes using resources   |

---

# Most Important DevOps Investigation Flow

If a server says:

```text
No space left on device
```

Start with:

```bash
df -h
```

Identify the affected filesystem.

Then:

```bash
df -Th
```

Check filesystem type.

Then investigate directories:

```bash
sudo du -sh /* 2>/dev/null | sort -h
```

If `/var` is large:

```bash
sudo du -sh /var/* 2>/dev/null | sort -h
```

Then investigate the problematic directory.

This is much closer to how Linux troubleshooting is actually performed than memorizing individual commands.

