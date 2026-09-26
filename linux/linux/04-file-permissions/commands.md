# Linux File Permissions & Ownership — Commands

This file covers the commands used to inspect and manage Linux file permissions, ownership, groups, and access control.

The examples use Ubuntu Server.

Before changing permissions on important system files, always check the target carefully. For this topic, use files and directories created specifically for practice.

---

# 1. `ls -l`

The first command you should use when investigating permissions is:

```bash
ls -l
```

Example:

```text
-rw-r--r-- 1 rohan developers 1200 Sep 26 18:20 config.txt
```

The important part is:

```text
-rw-r--r--
```

Break it down:

```text
-    rw-    r--    r--
│     │      │      │
│     │      │      └── others
│     │      └───────── group
│     └──────────────── owner
└────────────────────── file type
```

The owner is:

```text
rohan
```

The group is:

```text
developers
```

---

# 2. `ls -ld`

When checking a directory, there is an important difference between:

```bash
ls -l directory
```

and:

```bash
ls -ld directory
```

`ls -l directory` normally shows the contents of the directory.

`ls -ld directory` shows the directory itself.

Example:

```bash
ls -ld /opt/devops-lab
```

This is particularly useful when troubleshooting directory permissions.

---

# 3. `ls -la`

The `-a` option shows hidden files.

```bash
ls -la
```

`-l` gives detailed information.

`-a` includes entries beginning with `.`.

For example:

```text
.bashrc
.profile
.bash_logout
```

Hidden files are not necessarily security-protected. The leading `.` mainly means that normal `ls` does not display them by default.

---

# 4. `stat`

`stat` displays detailed metadata about a file or directory.

Example:

```bash
stat file.txt
```

You may see:

```text
File: file.txt
Size: 100
Access: (0644/-rw-r--r--)
Uid: (1000/rohan)
Gid: (1002/developers)
```

This is useful when you need more information than `ls -l` provides.

You can specifically look at the numeric permissions:

```bash
stat -c '%a %n' file.txt
```

Example:

```text
644 file.txt
```

The format string:

```text
%a
```

means numeric permissions.

```text
%n
```

means filename.

---

# 5. Understanding Numeric Permissions

Linux uses three values for owner, group, and others.

```text
r = 4
w = 2
x = 1
```

Add them together.

For example:

```text
rwx = 4 + 2 + 1 = 7
rw- = 4 + 2     = 6
r-x = 4 + 1     = 5
r-- = 4         = 4
--- = 0
```

Therefore:

```text
755 = rwxr-xr-x
644 = rw-r--r--
700 = rwx------
600 = rw-------
```

This is the basis of numeric `chmod`.

---

# 6. `chmod`

`chmod` changes file or directory permissions.

Basic syntax:

```bash
chmod MODE FILE
```

Example:

```bash
chmod 644 file.txt
```

This sets:

```text
Owner  = rw-
Group  = r--
Others = r--
```

Verify:

```bash
ls -l file.txt
```

---

# 7. `chmod` with `755`

For an executable script:

```bash
chmod 755 script.sh
```

This gives:

```text
Owner  = rwx
Group  = r-x
Others = r-x
```

You can verify:

```bash
ls -l script.sh
```

---

# 8. `chmod 700`

```bash
chmod 700 private.sh
```

This means:

```text
Owner  = rwx
Group  = ---
Others = ---
```

This is useful when a script or directory should only be accessible by its owner.

---

# 9. `chmod 600`

```bash
chmod 600 secret.txt
```

This gives:

```text
Owner  = rw-
Group  = ---
Others = ---
```

This is commonly appropriate for files containing sensitive information when only the owner should access them.

The correct permission depends on the application and security requirements.

---

# 10. Symbolic `chmod`

Instead of specifying the complete numeric permission, you can make a specific change.

The classes are:

```text
u = owner
g = group
o = others
a = all
```

Permissions:

```text
r = read
w = write
x = execute
```

---

## Add execute permission for owner

```bash
chmod u+x script.sh
```

This adds execute permission to the owner without changing the other permissions.

---

## Add write permission for group

```bash
chmod g+w file.txt
```

---

## Remove write permission from others

```bash
chmod o-w file.txt
```

---

## Add execute permission for everyone

```bash
chmod a+x script.sh
```

Be careful with `a+x`. It gives execute permission to owner, group, and others.

---

# 11. `chmod` Using Multiple Changes

You can make multiple changes in one command.

Example:

```bash
chmod u+x,g-w script.sh
```

This means:

```text
u+x → add execute for owner
g-w → remove write from group
```

Another example:

```bash
chmod u=rw,g=r,o= file.txt
```

This explicitly sets:

```text
Owner  = rw-
Group  = r--
Others = ---
```

This is equivalent to:

```text
640
```

---

# 12. `chmod -R`

`-R` means recursive.

Example:

```bash
chmod -R 755 /opt/devops-lab
```

It applies the change to the directory and everything inside it.

This option must be used carefully.

For example:

```bash
sudo chmod -R 777 /
```

would be extremely dangerous.

Never use recursive commands on large system paths without understanding exactly what they will modify.

---

# 13. `chown`

`chown` changes the owner of a file or directory.

Syntax:

```bash
sudo chown USER FILE
```

Example:

```bash
sudo chown devuser file.txt
```

Check:

```bash
ls -l file.txt
```

The owner should now be:

```text
devuser
```

---

# 14. `chown` Owner and Group

You can change both at the same time:

```bash
sudo chown devuser:developers file.txt
```

The format is:

```text
chown USER:GROUP FILE
```

For example:

```text
Owner = devuser
Group = developers
```

This is very common when preparing application directories.

---

# 15. Change Only the Group with `chown`

You can also change only the group using:

```bash
sudo chown :developers file.txt
```

The empty part before `:` means that the owner should not be changed.

---

# 16. `chown -R`

Recursive ownership:

```bash
sudo chown -R devuser:developers /opt/devops-lab
```

This changes ownership of the directory and everything below it.

Again, verify the path first.

---

# 17. `chgrp`

`chgrp` changes the group ownership.

Example:

```bash
sudo chgrp developers file.txt
```

Check:

```bash
ls -l file.txt
```

The owner remains unchanged.

Only the group changes.

---

# 18. `chgrp -R`

Recursive group change:

```bash
sudo chgrp -R developers /opt/devops-lab
```

This changes the group ownership of the directory and its contents.

---

# 19. `umask`

Check the current umask:

```bash
umask
```

Example:

```text
0022
```

You can also display it symbolically:

```bash
umask -S
```

Possible output:

```text
u=rwx,g=rx,o=rx
```

The umask determines which permissions are removed from the permissions requested when new files and directories are created.

---

# 20. Temporarily Change `umask`

For a shell session, you can test a different umask:

```bash
umask 077
```

Then create a file:

```bash
touch test.txt
```

Check:

```bash
ls -l test.txt
```

You will typically see something similar to:

```text
-rw-------
```

because the umask prevents group and others from receiving the normal default permissions.

This is useful for understanding how default permissions work.

The change normally affects the current shell/session unless configured elsewhere.

---

# 21. `namei`

`namei` is extremely useful when troubleshooting permissions on a path.

Example:

```bash
namei -l /home/rohan/project/config/app.conf
```

It displays each component of the path.

For example:

```text
f: /home/rohan/project/config/app.conf
drwxr-xr-x /
drwxr-x--- home
drwx------ rohan
drwxr-x--- project
drwxr-x--- config
-rw-r----- app.conf
```

The exact output will depend on your system.

This helps identify cases where the file itself looks accessible but a parent directory prevents traversal.

---

# 22. `getfacl`

`getfacl` displays Access Control List information.

Example:

```bash
getfacl file.txt
```

You may see:

```text
user::rw-
group::r--
other::---
```

If additional ACL entries exist, they will also be shown.

ACLs are useful when normal owner/group/others permissions are not enough.

---

# 23. `setfacl`

`setfacl` modifies ACLs.

For example:

```bash
sudo setfacl -m u:tester:r file.txt
```

This gives the user `tester` read access through an ACL.

Check:

```bash
getfacl file.txt
```

You may see:

```text
user::rw-
user:tester:r--
group::r--
other::---
```

ACLs should be used intentionally. If a file has unexpected ACL entries, they can make permission troubleshooting confusing.

---

# 24. `setfacl -x`

Remove a specific ACL entry:

```bash
sudo setfacl -x u:tester file.txt
```

Then verify:

```bash
getfacl file.txt
```

---

# 25. `setfacl -m g:GROUP`

You can also give a group specific ACL permissions.

Example:

```bash
sudo setfacl -m g:developers:rwx directory
```

This gives the `developers` group `rwx` access through an ACL.

For directories, ACL behavior needs to be understood carefully because directory traversal and child objects are separate concerns.

---

# 26. SUID

You can set SUID using symbolic mode:

```bash
chmod u+s program
```

Check:

```bash
ls -l program
```

You may see:

```text
-rwsr-xr-x
```

The `s` in the owner execute position represents SUID.

Do not experiment with SUID on random system programs. For the lab, we will demonstrate the concept using a controlled file.

---

# 27. SGID

Set SGID on a directory:

```bash
chmod g+s shared-directory
```

Check:

```bash
ls -ld shared-directory
```

You may see:

```text
drwxrwsr-x
```

The `s` in the group execute position represents SGID.

On directories, SGID is particularly useful for shared project directories because newly created files can inherit the directory's group.

---

# 28. Sticky Bit

Set the sticky bit:

```bash
chmod +t shared-directory
```

Check:

```bash
ls -ld shared-directory
```

You may see:

```text
drwxrwxrwt
```

The `t` represents the sticky bit.

The standard `/tmp` directory commonly uses this behavior.

---

# 29. `find` with Permissions

`find` can locate files based on permissions.

Example:

```bash
find . -type f -perm 600
```

This searches for regular files with exactly the specified permission pattern.

You can also search for executable files:

```bash
find . -type f -perm /111
```

`/111` means that at least one execute bit is set.

For example, a file with:

```text
700
```

or:

```text
755
```

can match.

---

# 30. Finding World-Writable Files

A world-writable file allows others to write to it.

You can search for such files with:

```bash
find /path -type f -perm -002
```

The exact search should be restricted to an appropriate path.

Do not blindly scan the entire filesystem as root just for practice.

World-writable files can sometimes be legitimate, but unexpected ones deserve investigation.

---

# 31. `sudo`

Some permission and ownership operations require administrator privileges.

For example:

```bash
sudo chown devuser:developers file.txt
```

Without `sudo`, a normal user may receive:

```text
Operation not permitted
```

`sudo` does not permanently make your user root.

It executes a particular command with elevated privileges according to the system's sudo configuration.

---

# 32. `id`

Permissions are based on the identity of the process.

Therefore, when troubleshooting, check:

```bash
id
```

For another user:

```bash
id devuser
```

This shows:

```text
UID
primary GID
supplementary groups
```

Group membership is especially important when checking whether group permissions should apply.

---

# 33. `groups`

Check the groups of the current user:

```bash
groups
```

Or another user:

```bash
groups devuser
```

This is useful when you expect group-based access but receive `Permission denied`.

---

# 34. `lsattr`

Linux filesystems can have additional attributes that are separate from normal permissions.

You can inspect them with:

```bash
lsattr file.txt
```

For normal beginner permission troubleshooting, this is not usually the first command to use.

It becomes useful when a file behaves unexpectedly even though normal permissions look correct.

---

# 35. `getcap`

Linux can also use file capabilities for giving programs specific privileges without giving them the full power associated with traditional SUID.

You can inspect capabilities with:

```bash
getcap /path/to/program
```

This is an advanced security concept, but it is useful to know that Linux has mechanisms beyond normal `rwx` permissions and SUID.

---

# 36. Permission Troubleshooting Flow

When you receive:

```text
Permission denied
```

use a structured approach.

### Step 1 — Check the current user

```bash
whoami
id
```

### Step 2 — Check the target

```bash
ls -l file.txt
```

### Step 3 — Check the parent directory

```bash
ls -ld /path/to/directory
```

### Step 4 — Check the complete path

```bash
namei -l /path/to/file
```

### Step 5 — Check ACLs if necessary

```bash
getfacl file.txt
```

### Step 6 — Check ownership

Ask:

```text
Who owns it?
Which group owns it?
Is my user in that group?
```

### Step 7 — Change only what is actually required

For example:

```bash
sudo chown devuser:developers file.txt
```

or:

```bash
chmod 640 file.txt
```

Do not automatically use `777`.

---

# 37. Quick Reference

```bash
# View permissions
ls -l
ls -ld directory
stat file

# Change permissions
chmod 644 file
chmod 755 script.sh
chmod u+x script.sh
chmod g+w file
chmod o-r file
chmod -R 755 directory

# Change ownership
sudo chown user file
sudo chown user:group file
sudo chown -R user:group directory

# Change group
sudo chgrp group file
sudo chgrp -R group directory

# Default permissions
umask
umask -S

# Investigate paths
namei -l /path/to/file

# ACL
getfacl file
setfacl -m u:user:r file
setfacl -x u:user file

# Special permissions
chmod u+s program
chmod g+s directory
chmod +t directory

# Identity
whoami
id
groups
```

---

# 38. Important Permission Values

| Numeric | Symbolic | Meaning |
|---:|---|---|
| `0` | `---` | No access |
| `1` | `--x` | Execute |
| `2` | `-w-` | Write |
| `3` | `-wx` | Write + execute |
| `4` | `r--` | Read |
| `5` | `r-x` | Read + execute |
| `6` | `rw-` | Read + write |
| `7` | `rwx` | Read + write + execute |

Common combinations:

| Permission | Meaning |
|---|---|
| `600` | Owner read/write only |
| `640` | Owner read/write, group read |
| `644` | Owner read/write, everyone else read |
| `700` | Owner full access |
| `750` | Owner full, group read/execute |
| `755` | Owner full, group/others read/execute |
| `770` | Owner/group full access |
| `775` | Owner/group full, others read/execute |
| `777` | Everyone full access — use only when genuinely required |

---

# 39. Commands You Should Be Comfortable With

For Linux administration and DevOps interviews, you should be comfortable explaining:

```text
ls -l
chmod
chown
chgrp
umask
stat
namei
getfacl
setfacl
```

You should also understand:

```text
rwx
owner
group
others
UID
GID
numeric permissions
symbolic permissions
SUID
SGID
sticky bit
ACL
least privilege
```

The important part is not just remembering commands.

You should be able to look at:

```text
-rwxr-x---
```

and explain what it means, who can access the file, and what would happen if a particular user tried to read, modify, or execute it.
