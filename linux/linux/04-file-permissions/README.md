# Linux File Permissions & Ownership

Linux is designed as a multi-user operating system. Multiple users, applications, services, and processes can exist on the same system at the same time. Because of this, Linux needs a way to control **who can access a file or directory and what they are allowed to do with it**.

This is where file permissions and ownership come in.

For example, imagine a Linux server where:

- a developer creates application files,
- Jenkins performs deployments,
- NGINX serves web content,
- a monitoring service reads logs,
- an administrator manages the whole system.

It would be a security problem if every user and every service could modify every file.

Linux therefore associates files and directories with an **owner**, a **group**, and a set of **permissions**. When a process attempts to access something, Linux uses this information to decide whether the requested operation should be allowed.

Understanding permissions is one of the most important Linux administration concepts because permission problems are extremely common on servers.

---

# 1. What Are Linux File Permissions?

File permissions are rules attached to files and directories that control what users are allowed to do with them.

The three basic operations are:

```text
read
write
execute
```

For a normal file, these generally mean:

### Read

The user can view the contents of the file.

For example:

```bash
cat config.txt
```

requires read access to the file.

### Write

The user can modify the contents of the file.

For example:

```bash
nano config.txt
```

requires appropriate write access if the file is going to be changed.

### Execute

The user can execute the file as a program or script, assuming the file is otherwise executable.

For example:

```bash
./backup.sh
```

requires execute permission on the script.

However, **read, write, and execute do not mean exactly the same thing for a directory**.

For directories:

- `r` allows listing the directory contents
- `w` allows creating, deleting, or renaming entries inside the directory when the required permissions are present
- `x` allows entering/traversing the directory and accessing entries within it

This difference between files and directories is very important and is a common source of confusion for beginners.

---

# 2. Why Does Linux Need Permissions?

Consider a server with this structure:

```text
/var/www/application/
├── index.html
├── config.php
└── uploads/
```

Suppose NGINX needs to read `index.html`, but application developers should be the only people allowed to modify it.

Without an access-control mechanism, any user who can access the server could potentially change the application.

Linux solves this by assigning ownership and permissions.

For example:

```text
Owner       → developer
Group       → webdevelopers
Permissions → owner can read/write
              group can read
              others can read
```

The operating system can then enforce those rules whenever a process requests access.

This is one of the basic security mechanisms behind Linux servers.

---

# 3. Ownership: Owner and Group

Every normal Linux file has an associated user owner and group owner.

For example:

```bash
ls -l application.conf
```

might show:

```text
-rw-r----- 1 rohan developers 1240 Sep 25 14:30 application.conf
```

The important ownership information here is:

```text
Owner = rohan
Group = developers
```

Linux uses these identities when deciding which permission set should apply.

There are effectively three classes of users:

```text
Owner
Group
Others
```

These three categories are used by the traditional Linux permission system.

---

# 4. Owner

The **owner** is the user account associated with the file.

For example:

```text
Owner = rohan
```

If `rohan` owns a file, Linux checks the owner permission bits when `rohan` accesses it.

The owner is normally the user who created the file, although ownership can be changed using commands such as `chown`.

Ownership is not necessarily the same thing as the person who currently manages the system.

For example, an administrator can change the owner of a file even if the administrator did not create the file.

---

# 5. Group

A group allows multiple users to share a common access policy.

For example:

```text
developers
```

could contain:

```text
rohan
amit
neha
```

If a file belongs to:

```text
Owner = rohan
Group = developers
```

then the group permissions can apply to `amit` and `neha` as members of the `developers` group.

This is much easier to manage than assigning permissions separately to every user.

This is why groups are heavily used on Linux servers.

For example:

```text
developers
database-admins
docker
www-data
jenkins
monitoring
```

can represent different access requirements.

The previous topic covered how users and groups are created and managed. Now we use those concepts to control file access.

---

# 6. Others

`Others` means users who are neither:

- the file owner, nor
- members of the file's group

For example:

```text
Owner = rohan
Group = developers
```

If `amit` is in `developers`, he is evaluated using the group permissions.

If `suresh` is neither `rohan` nor a member of `developers`, he falls into the `others` category.

This distinction is important because Linux does not simply ask:

> "Is this user allowed?"

It first determines which permission class applies to the user.

---

# 7. Understanding `ls -l`

The most common command for checking permissions is:

```bash
ls -l
```

Example:

```text
-rwxr-x--- 1 rohan developers 2048 Sep 25 14:30 backup.sh
```

The first part:

```text
-rwxr-x---
```

contains the file type and permission information.

It can be broken down like this:

```text
- rwx r-x ---
│ │   │   │
│ │   │   └── Others
│ │   └────── Group
│ └────────── Owner
└──────────── File type
```

So:

```text
Owner  = rwx
Group  = r-x
Others = ---
```

This means:

```text
Owner:
read + write + execute

Group:
read + execute

Others:
no permissions
```

---

# 8. The First Character: File Type

The first character is not a permission.

It identifies the type of filesystem object.

Common values include:

```text
-    regular file
d    directory
l    symbolic link
c    character device
b    block device
s    socket
p    named pipe
```

For example:

```text
-rw-r--r--
```

starts with `-`, meaning a regular file.

A directory might look like:

```text
drwxr-xr-x
```

The `d` tells us it is a directory.

This distinction matters because directory permissions behave differently from file permissions.

---

# 9. The Nine Permission Bits

After the file type, there are nine permission positions.

They are divided into three groups:

```text
rwx rwx rwx
```

The first group belongs to the owner:

```text
rwx
```

The second belongs to the group:

```text
rwx
```

The third belongs to others:

```text
rwx
```

So:

```text
-rwxr-xr--
```

means:

```text
Owner  = rwx
Group  = r-x
Others = r--
```

---

# 10. What `r`, `w`, and `x` Mean

For a regular file:

```text
r = read
w = write
x = execute
```

If a permission is not granted, Linux shows:

```text
-
```

For example:

```text
rwx
```

means all three permissions are present.

```text
r-x
```

means read and execute are allowed, but write is not.

```text
r--
```

means only read is allowed.

```text
---
```

means no permission.

---

# 11. File Permissions in More Detail

Suppose we have:

```text
-rwxr-x---
```

The owner has:

```text
rwx
```

Therefore the owner can:

- read the file
- modify the file
- execute the file

The group has:

```text
r-x
```

Therefore members of the group can:

- read the file
- execute the file
- cannot modify the file

Others have:

```text
---
```

Therefore other users have no access through the normal permission bits.

This is a common pattern for scripts and administrative files.

---

# 12. Directory Permissions Are Different

One of the most important things to understand is that `rwx` has a different practical meaning when applied to a directory.

For a directory:

### Read (`r`)

Allows the user to list the names of entries inside the directory.

For example:

```bash
ls /some/directory
```

requires directory read permission to list its contents.

### Write (`w`)

Allows creating, deleting, and renaming directory entries when combined with the required traversal permission.

For example:

```bash
touch /some/directory/file.txt
```

or:

```bash
rm /some/directory/file.txt
```

depends heavily on the permissions of the directory itself.

### Execute (`x`)

For a directory, execute means **traverse/search**.

It allows a user to enter the directory:

```bash
cd /some/directory
```

and access entries inside it when the appropriate permissions are present.

This is why directory execute permission is particularly important.

---

# 13. Why Directory `x` Permission Matters

Suppose:

```text
directory:
d--x------
```

The user may not be able to list the directory contents because there is no read permission.

But if the user knows the name of a file inside the directory and has appropriate permissions on that file, the execute permission may allow traversal to that file.

So:

```text
r = list names
x = traverse/access
```

is a useful way to remember directory permissions.

Write permission controls modification of directory entries.

---

# 14. Permission Checking Process

When a process tries to access a file, Linux does not simply look at all permissions and combine them.

It determines the identity of the process and checks the appropriate permission class.

Conceptually:

```text
Process
   |
   | UID / GIDs
   ↓
Kernel checks file metadata
   |
   ├── Is process owner?
   │
   ├── Otherwise, is process in file's group?
   │
   └── Otherwise, use "others"
   |
   ↓
Check requested permission
   |
   ↓
Allow or deny
```

For example:

```text
File owner = rohan
File group = developers
```

If `rohan` accesses the file, owner permissions are considered.

If another user is a member of `developers`, group permissions are considered.

If the user is neither, the `others` permissions apply.

This is the basic decision model behind traditional Unix/Linux file permissions.

---

# 15. Numeric Permissions

Linux also represents permissions using numbers.

The values are:

```text
Read    = 4
Write   = 2
Execute = 1
```

These values are added together.

For example:

```text
read + write
4 + 2 = 6
```

Therefore:

```text
rw- = 6
```

Similarly:

```text
r-x = 4 + 1 = 5
```

and:

```text
rwx = 4 + 2 + 1 = 7
```

---

# 16. Understanding `755`

A very common permission is:

```text
755
```

It represents three permission groups:

```text
7   5   5
│   │   │
│   │   └── Others
│   └────── Group
└────────── Owner
```

Calculate each number:

```text
7 = 4 + 2 + 1 = rwx
5 = 4 + 1     = r-x
5 = 4 + 1     = r-x
```

Therefore:

```text
755 = rwxr-xr-x
```

The owner has full access.

The group can read and execute.

Others can read and execute.

This is commonly used for executable files and directories, depending on the situation.

---

# 17. Understanding `644`

Another very common permission is:

```text
644
```

Calculate:

```text
6 = 4 + 2 = rw-
4 = 4     = r--
4 = 4     = r--
```

Therefore:

```text
644 = rw-r--r--
```

The owner can read and modify the file.

The group can read it.

Others can read it.

This is common for ordinary configuration, text, and web-content files when broad read access is appropriate.

The correct permission always depends on what the file contains and who needs access.

---

# 18. Understanding `700`

```text
700
```

means:

```text
7 = rwx
0 = ---
0 = ---
```

Therefore:

```text
700 = rwx------
```

Only the owner has access.

This is useful for private directories and sensitive files when only one account should access them.

---

# 19. `chmod`

`chmod` means **change mode**.

It changes the permission bits of a file or directory.

There are two common ways to use it:

```text
symbolic mode
numeric mode
```

---

# 20. Numeric `chmod`

Example:

```bash
chmod 644 file.txt
```

This changes the permissions to:

```text
rw-r--r--
```

Another example:

```bash
chmod 755 script.sh
```

This changes them to:

```text
rwxr-xr-x
```

You can verify:

```bash
ls -l script.sh
```

---

# 21. Symbolic `chmod`

Instead of numbers, you can specify the permission changes directly.

The basic classes are:

```text
u = user/owner
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

Examples:

```bash
chmod u+x script.sh
```

Adds execute permission for the owner.

```bash
chmod g+w file.txt
```

Adds write permission for the group.

```bash
chmod o-r file.txt
```

Removes read permission from others.

```bash
chmod a+x script.sh
```

Adds execute permission for everyone.

---

# 22. Why Symbolic Mode Is Useful

Suppose a file already has:

```text
rw-r-----
```

and you only want to give the owner execute permission.

You could calculate a completely new numeric mode, but you can also simply use:

```bash
chmod u+x file
```

This changes only the requested permission.

Symbolic mode is therefore useful when you want to make a specific permission change without disturbing the other permissions.

---

# 23. `chown`

`chown` means **change owner**.

It changes the user ownership of a file or directory.

Example:

```bash
sudo chown devuser file.txt
```

Now:

```text
Owner = devuser
```

You can also change both owner and group:

```bash
sudo chown devuser:developers file.txt
```

Now:

```text
Owner = devuser
Group = developers
```

This is extremely common when configuring applications and services.

For example, if an application should run as a particular service account, its files may need to be owned by that account.

---

# 24. `chgrp`

`chgrp` changes the group ownership.

Example:

```bash
sudo chgrp developers file.txt
```

The user owner remains unchanged, but the group becomes:

```text
developers
```

So:

```text
chown
```

can change the owner, while:

```text
chgrp
```

changes the group.

---

# 25. Recursive Permissions

The `-R` option means recursive.

For example:

```bash
chmod -R 755 /some/directory
```

This applies the permission change to the directory and everything underneath it.

Similarly:

```bash
chown -R devuser:developers /some/directory
```

changes ownership recursively.

Recursive operations are powerful but dangerous.

For example, accidentally running:

```bash
sudo chmod -R ...
```

against the wrong system directory can break applications or even the operating system.

Always verify the path before using `-R`.

---

# 26. Why `chmod -R 777` Is a Bad Habit

You will often see people solve permission problems by running:

```bash
chmod -R 777 directory
```

This gives read, write, and execute permissions to everyone.

Numerically:

```text
7 = rwx
7 = rwx
7 = rwx
```

So:

```text
777 = rwxrwxrwx
```

Although this can make a permission error disappear, it can also remove an important security boundary.

For example, if an application directory contains configuration files or application code, allowing every local user or compromised service to modify them can create serious security problems.

The better approach is to determine:

```text
Who needs access?
What type of access do they need?
Which user/group should own the resource?
```

Then assign the minimum permissions required.

This follows the principle of **least privilege**.

---

# 27. `umask`

`umask` controls the default permissions that are removed when new files and directories are created.

Check the current value:

```bash
umask
```

You may see:

```text
0022
```

This does not mean that every new file simply gets permission `0022`.

Instead, the system starts from default creation permissions and the umask removes certain permissions.

For regular files, the typical base is:

```text
666
```

For directories:

```text
777
```

With a common umask of:

```text
022
```

new files commonly end up around:

```text
644
```

and new directories around:

```text
755
```

The exact behavior also depends on the program creating the file because applications can request their own creation mode.

---

# 28. Why Files Usually Do Not Get Execute Permission Automatically

A common question is:

> If directories start from `777` and files from `666`, why aren't new files executable?

Because regular files normally start from:

```text
666
```

which contains:

```text
rw-rw-rw-
```

before the umask is applied.

There is no execute bit in the normal base permission for a newly created regular file.

A program or administrator can later add execute permission with:

```bash
chmod +x script.sh
```

This is an important security feature because creating a text file should not automatically make it executable.

---

# 29. Special Permissions

Linux also has special permission bits that provide additional behavior.

The three important ones are:

```text
SUID
SGID
Sticky Bit
```

These are more advanced than normal `rwx` permissions, but they are important for Linux administration and security.

---

# 30. SUID

SUID means **Set User ID**.

When SUID is set on an executable file, the process can run with the effective user identity of the file owner rather than simply the identity of the user who started it.

A classic example historically is:

```text
/usr/bin/passwd
```

Changing a password requires access to protected system files.

A normal user runs:

```bash
passwd
```

but the password-changing operation needs privileges that the normal user does not have directly.

SUID allows the program to perform its intended privileged operation under controlled conditions.

You may see an `s` in the owner execute position:

```text
-rwsr-xr-x
```

The `s` indicates SUID together with the execute bit.

SUID programs must be treated carefully because vulnerabilities in privileged SUID programs can have significant security consequences.

---

# 31. SGID

SGID means **Set Group ID**.

On an executable, SGID can cause the process to run with the effective group identity associated with the file.

On a directory, SGID has another very useful behavior.

When SGID is set on a directory, newly created files and directories inside it inherit the directory's group rather than simply using the creating user's primary group.

This is very useful for shared project directories.

For example:

```text
/opt/project
```

could belong to:

```text
developers
```

with SGID enabled.

Then developers working inside that directory can naturally create files associated with the shared group.

This is a common pattern for collaborative directories.

---

# 32. Sticky Bit

The sticky bit is commonly used on directories where many users can create files but users should generally only be able to remove their own files.

The classic example is:

```text
/tmp
```

You can inspect it with:

```bash
ls -ld /tmp
```

You may see something similar to:

```text
drwxrwxrwt
```

The final `t` represents the sticky bit.

This prevents one user from freely deleting another user's files in a shared directory, assuming the relevant ownership and privilege conditions.

---

# 33. ACLs

Traditional permissions provide:

```text
owner
group
others
```

Sometimes this is not enough.

For example, suppose:

```text
Owner = rohan
Group = developers
```

but you also want one particular user named `tester` to have read access without making `tester` a member of the developers group.

Linux supports **Access Control Lists (ACLs)** for more detailed permissions.

Common commands include:

```bash
getfacl
setfacl
```

For example:

```bash
getfacl file.txt
```

shows ACL information.

ACLs are useful when the normal owner/group/others model does not provide enough flexibility.

They are particularly useful on shared servers where different users need different access to the same resource.

---

# 34. Permissions and Security

Permissions are one layer of Linux security.

They help prevent unauthorized access between users and processes.

However, permissions should not be treated as the entire security model.

A production Linux system can also use:

- sudo policies
- ACLs
- SELinux or AppArmor
- service isolation
- namespaces
- containers
- filesystem mount options
- authentication controls
- network security
- application-level authorization

For example, changing a file to:

```text
600
```

can restrict normal Linux users from reading it, but that does not automatically protect the data from every possible privileged process.

The root account and security mechanisms such as MAC systems can change the overall access decision.

So file permissions are an important layer, not the only layer.

---

# 35. Permission Troubleshooting

Permission errors are common on Linux servers.

Suppose you get:

```text
Permission denied
```

Do not immediately run:

```bash
chmod 777
```

Instead, investigate.

First check the file:

```bash
ls -l file.txt
```

Then check your identity:

```bash
whoami
id
```

Check the directory:

```bash
ls -ld /path/to/directory
```

If the file is several directories deep, check each directory in the path.

For example:

```text
/home/rohan/project/config/app.conf
```

You may have permission on the file itself but not permission to traverse one of the parent directories.

You can investigate the path using:

```bash
namei -l /home/rohan/project/config/app.conf
```

This is a very useful troubleshooting command because it displays permissions for each component of the path.

---

# 36. A Practical Permission Investigation

Suppose:

```text
config.txt
Owner = root
Group = developers
Permissions = rw-r-----
```

A user named `rohan` tries to modify it.

First ask:

```text
Is rohan the owner?
```

If no, then:

```text
Is rohan a member of developers?
```

If yes, the group permissions apply.

The group has:

```text
r--
```

So the user can read the file but cannot write to it.

The result is:

```text
Read    → allowed
Write   → denied
Execute → denied
```

This reasoning is much more useful than memorizing permission numbers.

---

# 37. File Permissions and DevOps

Permissions appear everywhere in DevOps.

### CI/CD

A Jenkins process may need access to:

```text
workspace
build files
deployment scripts
Docker socket
configuration files
```

Giving the Jenkins account unrestricted access to the entire server would be unnecessary and risky.

Permissions and groups help limit its access.

### Web servers

A web server such as NGINX may need to read:

```text
/var/www/
```

but should not necessarily be able to modify every application file.

### Deployment

A deployment user might own:

```text
/opt/application/
```

while the application service runs under a separate service account.

Correct ownership and group permissions are necessary for this setup to work.

### Logs

Applications often write logs to:

```text
/var/log/
```

The application needs appropriate access, while ordinary users should not automatically be able to modify system logs.

### Docker

Docker environments also involve Linux users, groups, file ownership, and permissions.

For example, a file created inside a container may appear on a mounted host directory with a particular UID/GID.

This can lead to a common problem:

```text
Container creates file
        ↓
Host sees numeric UID/GID
        ↓
Host user cannot modify file
        ↓
Permission denied
```

Understanding Linux ownership makes these Docker problems much easier to troubleshoot.

---

# 38. Common Permission Mistakes

## Mistake 1: Using `777` as the solution

It may hide the immediate problem but creates unnecessarily broad access.

Instead, identify the required user/group and give only the required permissions.

---

## Mistake 2: Confusing file and directory permissions

For files:

```text
r = read content
w = modify content
x = execute
```

For directories:

```text
r = list entries
w = modify directory entries
x = traverse/access
```

These are not interchangeable concepts.

---

## Mistake 3: Forgetting parent-directory permissions

A user may have permission on:

```text
file.txt
```

but still be unable to access it because they cannot traverse one of the parent directories.

---

## Mistake 4: Using `chmod -R` without checking the path

Recursive operations affect everything below the specified path.

Always verify:

```bash
pwd
ls
```

and carefully inspect the target before using `-R`.

---

## Mistake 5: Changing ownership unnecessarily

Ownership affects how permission checks are performed.

Changing ownership just to make an error disappear can create a different security or application problem.

---

# 39. Important Commands for This Topic

The main commands to know are:

```bash
ls -l
ls -ld
chmod
chown
chgrp
umask
stat
namei
getfacl
setfacl
```

You should also be comfortable with:

```bash
whoami
id
groups
```

because permissions are based on the identity of the process accessing the resource.

---

# 40. Important Concepts to Remember

The complete picture can be understood as:

```text
User
  ↓
UID / Groups
  ↓
Process identity
  ↓
File or directory
  ↓
Owner + Group + Permission bits
  ↓
Linux access check
  ↓
Allow / Deny
```

For a file:

```text
-rwxr-x---
```

read it as:

```text
File type
   ↓
Owner permissions
   ↓
Group permissions
   ↓
Others permissions

- | rwx | r-x | ---
```

And remember:

```text
r = 4
w = 2
x = 1
```

Therefore:

```text
7 = rwx
6 = rw-
5 = r-x
4 = r--
0 = ---
```

---

# 41. Recommended Way to Think About Permissions

Instead of memorizing:

```text
755
644
700
777
```

ask four questions:

```text
1. Who owns the resource?
2. Which group owns it?
3. Who needs access?
4. What type of access do they actually need?
```

Then choose the appropriate permissions.

For example:

```text
Application configuration
        ↓
Only application owner should modify it
        ↓
Others may only need read access
        ↓
Choose restrictive permissions
```

This approach is much more useful in real administration than memorizing permission numbers.

---

# 42. What We Will Practice

The hands-on lab for this topic will create a realistic shared directory.

We will practice:

```text
Create files
      ↓
Inspect permissions
      ↓
Change permissions
      ↓
Create groups
      ↓
Change ownership
      ↓
Test access as different users
      ↓
Use numeric permissions
      ↓
Use symbolic permissions
      ↓
Understand directory permissions
      ↓
Use SGID for a shared directory
      ↓
Investigate permission-denied problems
```

The goal is to understand **why access succeeds or fails**, not simply memorize commands.

---

# 43. Key Takeaways

Linux permissions provide a basic access-control mechanism for files and directories.

Every file has an owner and group, and the traditional permission model separates users into:

```text
owner
group
others
```

The basic permissions are:

```text
read
write
execute
```

For files, these control reading, modification, and execution.

For directories, they control listing, modification of entries, and traversal.

`chmod` changes permissions.

`chown` changes ownership.

`chgrp` changes group ownership.

`umask` influences the permissions assigned to newly created files and directories.

SUID, SGID, and the sticky bit provide additional behavior.

ACLs provide more granular access control when the traditional owner/group/others model is not sufficient.

Most importantly, when you see:

```text
Permission denied
```

do not immediately use `chmod 777`.

First understand:

```text
Who am I?
Who owns the resource?
What group owns it?
Which groups am I in?
What permissions are configured?
What permissions exist on the parent directories?
```

That way of thinking is what you will use when troubleshooting real Linux and DevOps systems.

---

# 44. Environment

This topic is being studied and practiced on:

```text
Host OS       : Windows 11
Virtualization: VMware Workstation
Guest OS      : Ubuntu Server 24.04 LTS
Shell         : Bash
Network       : NAT
```

The commands and examples are intended primarily for Ubuntu/Linux environments, although the underlying Unix permission concepts apply broadly across Linux distributions.

---

# 45. Status

```text
Topic  : Linux File Permissions & Ownership
Status : Learning + Hands-on Practice
Level  : Beginner → Intermediate
```

Next:

```text
commands.md
lab.md
```

These will take the concepts from this README and apply them directly on the Ubuntu VM.
