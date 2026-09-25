# Linux Users & Groups — Commands

This file covers the commands used to check, create, modify, and manage users and groups in Linux.

The examples are written for Ubuntu Server and are intended for a practice VM.

---

## 1. `whoami`

`whoami` shows the username of the account currently being used in the terminal.

### Syntax

```bash
whoami
```

### Example

```bash
whoami
```

Possible output:

```text
rohan
```

This is useful when working with `sudo`, switching between users, or troubleshooting permissions.

---

## 2. `id`

The `id` command gives more information about a user than `whoami`.

It normally shows:

- UID — User ID
- GID — primary Group ID
- Groups — supplementary groups

### Syntax

```bash
id [username]
```

### Example

```bash
id
```

Example output:

```text
uid=1000(rohan) gid=1000(rohan) groups=1000(rohan),27(sudo)
```

This means:

- UID `1000` belongs to user `rohan`
- primary GID is `1000`
- the user is also a member of group `sudo`

You can check another user:

```bash
id devuser
```

### Useful options

#### Show only UID

```bash
id -u devuser
```

#### Show only primary GID

```bash
id -g devuser
```

#### Show all group IDs

```bash
id -G devuser
```

#### Show group names instead of IDs

```bash
id -Gn devuser
```

`-n` asks for names instead of numbers.

---

## 3. `groups`

`groups` shows the groups that a user belongs to.

### Syntax

```bash
groups [username]
```

### Example

```bash
groups
```

or:

```bash
groups devuser
```

Example:

```text
devuser : devuser developers
```

Here `devuser` is the user's primary group and `developers` is an additional group.

---

# 4. `getent`

`getent` is useful for looking up information from Linux's configured account databases.

For local users, it can read `/etc/passwd`. It can also work with other configured identity sources.

### List users

```bash
getent passwd
```

This gives output similar to:

```text
root:x:0:0:root:/root:/bin/bash
rohan:x:1000:1000:Rohan:/home/rohan:/bin/bash
```

### Check one user

```bash
getent passwd devuser
```

This is usually better than manually searching `/etc/passwd` because it uses the system's configured account lookup mechanism.

### List groups

```bash
getent group
```

### Check one group

```bash
getent group developers
```

Example:

```text
developers:x:1002:devuser
```

---

# 5. `/etc/passwd`

`/etc/passwd` contains basic information about local user accounts.

You can view it with:

```bash
cat /etc/passwd
```

A line looks like:

```text
devuser:x:1001:1001:Development User:/home/devuser:/bin/bash
```

The fields are separated by `:`.

They mean:

```text
username : password-placeholder : UID : GID : comment : home : shell
```

For example:

```text
devuser
1001
1001
/home/devuser
/bin/bash
```

Modern Linux systems normally keep the actual password hashes in `/etc/shadow`, not `/etc/passwd`.

---

# 6. `/etc/shadow`

`/etc/shadow` stores password-related information such as password hashes and password aging information.

Because this file contains sensitive information, normal users usually cannot read it.

To inspect it as an administrator:

```bash
sudo cat /etc/shadow
```

Do not modify this file manually.

For normal account management, use commands such as:

```bash
passwd
usermod
chage
```

---

# 7. `/etc/group`

`/etc/group` contains group information.

View it with:

```bash
cat /etc/group
```

A typical entry:

```text
developers:x:1002:devuser
```

The fields are:

```text
group_name : password_placeholder : GID : members
```

You can also use:

```bash
getent group developers
```

---

# 8. `adduser`

On Ubuntu, `adduser` is a convenient interactive way to create a user.

### Syntax

```bash
sudo adduser username
```

### Example

```bash
sudo adduser devuser
```

Ubuntu will ask for:

- password
- full name
- other optional information

It normally creates the user's home directory as well.

For a beginner using Ubuntu, `adduser` is often easier to understand than the lower-level `useradd`.

---

# 9. `useradd`

`useradd` is the lower-level command for creating users.

### Basic example

```bash
sudo useradd devuser
```

Depending on the options used, this may not create the home directory automatically.

For a normal user with a home directory:

```bash
sudo useradd -m devuser
```

### Important options

#### `-m`

Create the user's home directory.

```bash
sudo useradd -m devuser
```

Usually the home directory becomes:

```text
/home/devuser
```

#### `-d`

Specify a custom home directory.

```bash
sudo useradd -m -d /home/devuser devuser
```

#### `-s`

Specify the login shell.

```bash
sudo useradd -m -s /bin/bash devuser
```

#### `-c`

Add a comment, normally used for the user's full name or description.

```bash
sudo useradd -m -c "Development User" devuser
```

#### `-G`

Add supplementary groups.

```bash
sudo useradd -m -G developers devuser
```

Be careful with `-G` when modifying an existing user. For `usermod`, forgetting `-a` can replace the user's existing supplementary groups.

---

# 10. `passwd`

`passwd` is used to set or change a user's password.

For your own account:

```bash
passwd
```

As administrator, you can set another user's password:

```bash
sudo passwd devuser
```

### Lock an account

```bash
sudo passwd -l devuser
```

`-l` locks the password.

### Unlock an account

```bash
sudo passwd -u devuser
```

Avoid using password-related options without understanding their effect. Account locking is different from completely removing a user.

---

# 11. `usermod`

`usermod` modifies an existing user account.

This is one of the most important commands for Linux administration.

### Add a user to a group

```bash
sudo usermod -aG developers devuser
```

Here:

- `-G` specifies supplementary groups
- `-a` means append instead of replacing the existing supplementary groups

### Important warning

Do **not** casually use:

```bash
sudo usermod -G developers devuser
```

Without `-a`, this can replace the user's existing supplementary group memberships.

The safer common form is:

```bash
sudo usermod -aG groupname username
```

---

### Change login shell

```bash
sudo usermod -s /bin/bash devuser
```

### Change home directory

```bash
sudo usermod -d /home/newhome devuser
```

If you also want to move the existing home directory:

```bash
sudo usermod -d /home/newhome -m devuser
```

`-m` moves the existing home directory content.

### Change comment

```bash
sudo usermod -c "Development User" devuser
```

### Lock user

```bash
sudo usermod -L devuser
```

### Unlock user

```bash
sudo usermod -U devuser
```

---

# 12. `userdel`

`userdel` removes a user account.

### Remove the account

```bash
sudo userdel devuser
```

This removes the account but may leave the user's home directory.

### Remove the account and home directory

```bash
sudo userdel -r devuser
```

`-r` removes the user's home directory and mail spool where applicable.

Be careful with this command. Do not use it on important production accounts.

---

# 13. `groupadd`

Creates a new group.

### Syntax

```bash
sudo groupadd groupname
```

### Example

```bash
sudo groupadd developers
```

Check it:

```bash
getent group developers
```

---

# 14. `groupdel`

Deletes a group.

```bash
sudo groupdel developers
```

Be careful if the group is being used as the primary group of an existing user.

For your lab, only remove groups that you created specifically for testing.

---

# 15. `gpasswd`

`gpasswd` can also be used to manage group membership.

For example:

```bash
sudo gpasswd -a devuser developers
```

This adds `devuser` to the `developers` group.

Remove the user:

```bash
sudo gpasswd -d devuser developers
```

For everyday administration, `usermod -aG` is commonly seen and is worth knowing well.

---

# 16. `chown`

`chown` changes the owner of a file or directory.

### Syntax

```bash
sudo chown owner file
```

Example:

```bash
sudo chown devuser test.txt
```

Now `devuser` becomes the owner.

### Change owner and group

```bash
sudo chown devuser:developers test.txt
```

This sets:

```text
Owner = devuser
Group = developers
```

### Recursive ownership change

```bash
sudo chown -R devuser:developers /some/directory
```

`-R` means recursive, so it affects everything inside the directory.

Use recursive `chown` carefully. Running it against the wrong directory can cause serious permission problems.

---

# 17. `chgrp`

`chgrp` changes only the group ownership.

```bash
sudo chgrp developers test.txt
```

The owner remains unchanged, but the group becomes `developers`.

Recursive:

```bash
sudo chgrp -R developers /some/directory
```

---

# 18. `sudo`

`sudo` allows an authorized user to run a command with elevated privileges.

Example:

```bash
sudo apt update
```

or:

```bash
sudo adduser devuser
```

Linux uses this instead of requiring administrators to work as `root` all the time.

You can check what commands your account is allowed to run:

```bash
sudo -l
```

---

# 19. `visudo`

The sudo configuration is mainly controlled through:

```text
/etc/sudoers
```

and:

```text
/etc/sudoers.d/
```

If you need to edit sudo configuration, use:

```bash
sudo visudo
```

Do not normally edit `/etc/sudoers` directly with a regular text editor.

`visudo` checks the syntax before saving, which helps prevent configuration mistakes that could break sudo access.

---

# 20. `last`

`last` displays information about previous login sessions.

```bash
last
```

It can be useful when investigating account activity on a server.

For example, you may see:

```text
rohan    pts/0    192.168.1.10    ...
```

The exact output depends on the system's login history.

---

# 21. `lastlog`

`lastlog` shows the most recent login information for users.

```bash
lastlog
```

This can be useful during account auditing.

---

# 22. Understanding UID and GID

Linux internally identifies users and groups using numbers.

For example:

```text
uid=1000(rohan)
gid=1000(rohan)
```

The name is easier for humans, while the UID/GID is what the operating system uses internally.

You can check:

```bash
id rohan
```

or:

```bash
id -u rohan
```

---

# 23. Primary vs Supplementary Groups

A user normally has one primary group and can belong to additional supplementary groups.

Example:

```text
uid=1001(devuser)
gid=1001(devuser)
groups=1001(devuser),1002(developers)
```

Here:

```text
Primary group       = devuser
Supplementary group = developers
```

This becomes important when controlling access to shared files and directories.

---

# 24. Useful Investigation Commands

When troubleshooting a user-related problem, these commands are useful:

```bash
whoami
id username
groups username
getent passwd username
getent group groupname
ls -l /path/to/file
```

For example, if a user says:

> "I am in the developers group, but I cannot access the directory."

You can start with:

```bash
id devuser
```

Then check the directory:

```bash
ls -ld /path/to/directory
```

Then investigate the permissions.

Permissions themselves are covered in the next topic.

---

# 25. Important Locations

| Location | Purpose |
|---|---|
| `/etc/passwd` | Basic user account information |
| `/etc/shadow` | Password hashes and password aging information |
| `/etc/group` | Group information |
| `/etc/gshadow` | Secure group-related information |
| `/etc/sudoers` | Main sudo configuration |
| `/etc/sudoers.d/` | Additional sudo configuration files |
| `/home/username` | Normal user's home directory |
| `/root` | Root user's home directory |

---

# 26. Common Mistakes

### Forgetting `-a` with `usermod`

Use:

```bash
sudo usermod -aG developers devuser
```

rather than casually using:

```bash
sudo usermod -G developers devuser
```

because the second form can replace existing supplementary groups.

### Expecting group changes immediately

If a user was already logged in when they were added to a group, their current session may not immediately reflect the new membership.

Check:

```bash
id
```

A new login session normally picks up the updated group membership.

### Editing `/etc/passwd` manually

Do not manually edit account databases when normal administration commands are available.

Use:

```bash
adduser
useradd
usermod
userdel
```

### Using root for everything

Running everything as root defeats the purpose of Linux's permission model.

Use `sudo` only when elevated privileges are actually required.

---

# Quick Reference

```bash
whoami
id
groups
getent passwd
getent group

sudo adduser devuser
sudo useradd -m devuser
sudo passwd devuser

sudo groupadd developers
sudo usermod -aG developers devuser

sudo usermod -s /bin/bash devuser
sudo userdel -r devuser
sudo groupdel developers

sudo chown devuser file.txt
sudo chown devuser:developers file.txt
sudo chgrp developers file.txt

sudo -l
sudo visudo

last
lastlog
```

The main idea to remember is:

```text
User → UID
Group → GID
User can belong to multiple groups
Primary group → default group
Supplementary groups → additional access
sudo → controlled administrative access
```

Permissions and ownership work together. In the next topic, we will use this user/group knowledge to understand exactly how Linux decides whether a user can read, modify, or execute a file.
