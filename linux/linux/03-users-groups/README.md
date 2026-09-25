# Linux Users and Groups

## Overview

Linux is a **multi-user operating system**. This means multiple users can have accounts on the same Linux system, and each user can have different access to files, directories, processes, applications, and system resources.

For example, on a Linux server you might have:

```text
root
rohan
developer
deploy
jenkins
```

Each account can have its own identity, permissions, home directory, and access level.

Linux uses **users and groups** as one of the fundamental mechanisms for controlling access to system resources.

Understanding users and groups is important before learning:

- File permissions
- SSH administration
- `sudo`
- Service management
- Server security
- Application deployment
- CI/CD
- Docker
- Kubernetes
- Access control

A simple way to understand the relationship is:

```text
User
  ↓
Belongs to one or more groups
  ↓
Groups and user permissions
  ↓
Determine access to resources
```

---

# 1. What is a User?

A **user account** represents an identity on a Linux system.

When you log into Ubuntu, Linux needs to know:

- Who are you?
- What is your user ID?
- Which groups do you belong to?
- What is your home directory?
- Which shell should be used?
- What permissions do you have?

Linux stores this information as part of the user's account configuration.

For example, if your username is:

```text
rohan
```

Linux associates that username with information such as:

```text
Username: rohan
UID: 1000
Primary group: rohan
Home: /home/rohan
Shell: /bin/bash
```

The exact values depend on the system.

---

# 2. Usernames and UIDs

Linux internally identifies users using a number called a **UID**, or User ID.

For example:

```text
rohan → UID 1000
```

The username is mainly the human-readable representation.

The operating system uses the UID internally when checking ownership and permissions.

You can see your identity with:

```bash
whoami
```

To see more detailed information:

```bash
id
```

Example:

```text
uid=1000(rohan) gid=1000(rohan) groups=1000(rohan),27(sudo)
```

This tells us that the user has:

- UID `1000`
- Primary group ID `1000`
- Membership in the `sudo` group

---

# 3. Root User

Linux has a special administrative account called:

```text
root
```

The root user normally has UID:

```text
0
```

Root has extensive privileges over the system.

For example, root can normally:

- Create or delete users
- Install system packages
- Change system configuration
- Start and stop services
- Access protected files
- Change ownership
- Change permissions
- Manage disks and filesystems

You can identify the current user with:

```bash
whoami
```

If the result is:

```text
root
```

you are operating as the root account.

### Why root should be used carefully

Root can make changes that can affect the entire system.

For example:

```bash
rm -rf
```

can permanently delete files when used incorrectly.

Therefore, Linux administrators normally use a normal account and obtain elevated privileges only when required.

This is one reason `sudo` is important.

---

# 4. Normal Users

A normal user is an account intended for regular system use.

For example:

```text
rohan
developer
admin
```

Normal users generally have limited privileges.

They can normally:

- Work with their own files
- Run applications
- Create files in locations where they have permission
- Use their assigned groups
- Access permitted system resources

They normally cannot modify protected system files without elevated privileges.

For example:

```bash
cat /etc/passwd
```

may be allowed because `/etc/passwd` is generally readable.

But modifying system configuration usually requires additional privileges.

---

# 5. System Users

Linux also has accounts used by services and applications.

Examples may include users such as:

```text
www-data
sshd
nobody
systemd-network
```

The exact accounts vary between distributions and installed software.

These accounts are generally not intended for interactive human login.

They allow services to run with a specific identity instead of running everything as root.

For example, a web server may run under a dedicated service account.

This improves security because if the service is compromised, the attacker may have only the permissions of that service account rather than unrestricted root privileges.

---

# 6. Why Services Use Separate Users

Consider a web application.

A bad design would be:

```text
Web application
      ↓
    root
```

If the application is compromised, the attacker could potentially gain root-level access.

A more secure design is:

```text
Web application
      ↓
web-service user
      ↓
Only required permissions
```

This follows the principle of:

> **Least privilege**

A user or service should have only the permissions required to perform its job.

This concept becomes very important in:

- Linux security
- Docker
- Kubernetes
- CI/CD
- Cloud security
- Application security

---

# 7. `/etc/passwd`

Linux maintains local user account information in:

```text
/etc/passwd
```

You can view it with:

```bash
cat /etc/passwd
```

A typical entry looks like:

```text
rohan:x:1000:1000:Rohan Patil:/home/rohan:/bin/bash
```

The fields are separated by:

```text
:
```

The general structure is:

```text
username:password-placeholder:UID:GID:GECOS:home:shell
```

For example:

```text
rohan
x
1000
1000
Rohan Patil
/home/rohan
/bin/bash
```

### Important fields

#### Username

```text
rohan
```

The account name.

#### Password field

```text
x
```

This does not mean the password itself is stored there.

On modern Linux systems, password hashes are normally stored in:

```text
/etc/shadow
```

#### UID

```text
1000
```

The user's numeric ID.

#### GID

```text
1000
```

The user's primary group ID.

#### GECOS

```text
Rohan Patil
```

Optional descriptive information.

#### Home directory

```text
/home/rohan
```

#### Login shell

```text
/bin/bash
```

The shell normally started for an interactive login.

---

# 8. `/etc/shadow`

Password-related information for local accounts is stored in:

```text
/etc/shadow
```

This file contains password hashes and password-aging information.

Because this information is sensitive, access to `/etc/shadow` is restricted.

You may need elevated privileges to inspect it:

```bash
sudo cat /etc/shadow
```

Do not modify this file manually during normal administration.

Use tools such as:

```bash
passwd
```

for password management.

---

# 9. `/etc/group`

Group information is stored in:

```text
/etc/group
```

You can inspect it using:

```bash
cat /etc/group
```

A typical entry looks like:

```text
developers:x:1001:rohan,user2
```

The general format is:

```text
groupname:password-placeholder:GID:members
```

For example:

```text
developers
x
1001
rohan,user2
```

The group allows multiple users to be managed together.

---

# 10. What is a Group?

A **group** is a collection of users.

Groups make it easier to manage access for multiple users.

For example, imagine an organization with:

```text
rohan
amit
sneha
rahul
```

Instead of assigning access individually, you could create:

```text
developers
```

and add the required users to that group.

Then a directory could be owned by:

```text
developers
```

and group permissions could determine what members can do.

Conceptually:

```text
              developers
             /    |    \
          Rohan  Amit  Sneha
             |
        Shared access
             |
        Project files
```

This becomes very useful on shared servers.

---

# 11. Primary Group

Every Linux user has a **primary group**.

For example:

```text
User:
rohan

Primary group:
rohan
```

You can check it with:

```bash
id rohan
```

Example:

```text
uid=1000(rohan) gid=1000(rohan) groups=1000(rohan),27(sudo)
```

Here:

```text
uid=1000(rohan)
```

is the user identity.

```text
gid=1000(rohan)
```

is the primary group.

The remaining groups are supplementary groups.

---

# 12. Supplementary Groups

A user can belong to additional groups.

For example:

```text
rohan
├── rohan
├── sudo
├── docker
└── developers
```

These additional memberships are called **supplementary groups**.

They allow the user to receive additional access without changing the primary group.

For example, membership in the `sudo` group may allow a user to execute commands with elevated privileges, depending on the system's sudo configuration.

---

# 13. Checking User Identity

The most useful command for understanding your current identity is:

```bash
id
```

Example:

```text
uid=1000(rohan) gid=1000(rohan) groups=1000(rohan),27(sudo)
```

You can also use:

```bash
whoami
```

which simply displays the current username.

Another useful command is:

```bash
groups
```

which displays the groups associated with the current user.

---

# 14. Creating Users

A new user can be created using:

```bash
sudo useradd username
```

However, `useradd` is a relatively low-level tool.

On Ubuntu, administrators commonly use:

```bash
sudo adduser username
```

`adduser` provides an interactive interface and can create the user's home directory and guide you through account setup.

For example:

```bash
sudo adduser devuser
```

Depending on the configuration, the command asks for:

- Password
- Full name
- Other optional information

It can create:

```text
/home/devuser
```

for the user.

---

# 15. User Home Directory

A normal user generally has a home directory.

For example:

```text
/home/devuser
```

This is where the user's personal files and user-specific configuration can be stored.

You can see the home directory from the account database:

```bash
getent passwd devuser
```

---

# 16. Deleting Users

A user can be removed with:

```bash
sudo userdel username
```

If you also want to remove the user's home directory, an option such as:

```bash
sudo userdel -r username
```

can be used.

### Important warning

The `-r` option removes the user's home directory and associated mail spool where applicable.

Therefore, don't use it until you have verified that the account's data is no longer required.

---

# 17. Modifying Users

Linux provides:

```bash
usermod
```

for modifying existing accounts.

For example, changing a user's supplementary groups may involve:

```bash
sudo usermod -aG groupname username
```

The options here are important:

```text
-a
→ Append the user to supplementary groups.

-G
→ Specify supplementary groups.
```

The `-a` option is particularly important.

Using `-G` without `-a` can replace the user's existing supplementary group memberships rather than simply adding another group.

This is a common administration mistake.

---

# 18. Creating Groups

A group can be created using:

```bash
sudo groupadd developers
```

This creates a group called:

```text
developers
```

You can verify it using:

```bash
getent group developers
```

---

# 19. Adding Users to Groups

A user can be added to a supplementary group using:

```bash
sudo usermod -aG developers rohan
```

Breakdown:

```text
usermod
→ Modify an existing user.

-a
→ Append instead of replacing existing supplementary groups.

-G developers
→ Add the user to the developers group.

rohan
→ User being modified.
```

After changing group membership, the user's current login session may not immediately reflect the change.

You can verify with:

```bash
groups rohan
```

For the current session, logging out and logging back in is often the simplest way to ensure the new group membership is applied.

---

# 20. `sudo`

`sudo` means allowing a permitted user to execute a command with elevated privileges.

Example:

```bash
sudo apt update
```

The user remains logged into their normal account, but the particular command is executed with elevated privileges according to the sudo configuration.

This is generally safer than continuously working as root.

Conceptually:

```text
Normal user
     ↓
   sudo
     ↓
Elevated command
```

---

# 21. `/etc/sudoers`

Sudo access is controlled by configuration, primarily through:

```text
/etc/sudoers
```

Additional configuration can also be stored under:

```text
/etc/sudoers.d/
```

Do not casually edit `/etc/sudoers` with a normal text editor.

Use:

```bash
sudo visudo
```

because it validates the configuration syntax before saving.

A syntax error in sudo configuration can prevent users from obtaining the intended administrative access.

---

# 22. User and Group Ownership

Linux files have an owner and group associated with them.

For example:

```bash
ls -l
```

may show:

```text
-rw-r--r-- 1 rohan developers 120 app.conf
```

Here:

```text
rohan
```

is the owner.

```text
developers
```

is the group.

These ownership values work together with permissions to determine who can access the file.

The detailed permission system will be covered in the next filesystem-permissions topic.

---

# 23. Changing Ownership

The command:

```bash
chown
```

changes the owner of a file or directory.

Example:

```bash
sudo chown devuser app.conf
```

You can change both owner and group:

```bash
sudo chown devuser:developers app.conf
```

For directories and their contents, `-R` can be used:

```bash
sudo chown -R devuser:developers /opt/myapp
```

### Important warning

Recursive ownership changes can affect many files.

Always verify the target path before using:

```bash
-R
```

---

# 24. Changing Group Ownership

The command:

```bash
chgrp
```

changes the group associated with a file.

Example:

```bash
sudo chgrp developers app.conf
```

This is useful when access needs to be controlled through group membership.

---

# 25. Account Information with `getent`

`getent` queries system databases configured through the Name Service Switch mechanism.

For example:

```bash
getent passwd
```

lists user accounts known through the configured account sources.

To find a specific user:

```bash
getent passwd rohan
```

For groups:

```bash
getent group developers
```

This can be more useful than reading `/etc/passwd` or `/etc/group` directly because Linux systems can use multiple identity sources.

For example, users may come from:

- Local files
- LDAP
- Other configured identity services

---

# 26. User and Group Management Flow

A typical administration workflow might look like:

```text
Create group
     ↓
Create user
     ↓
Add user to group
     ↓
Assign ownership
     ↓
Configure permissions
     ↓
Test access
```

For example:

```text
developers
     ↓
rohan
     ↓
/opt/project
     ↓
Owner/group permissions
     ↓
Application access
```

This pattern is common in server administration and DevOps environments.

---

# 27. User Management and Security

User management is directly connected to system security.

Good practices include:

### Use individual accounts

Avoid sharing a single account between multiple people.

Individual accounts make it easier to identify who performed an action.

### Use least privilege

Users should have only the access they need.

### Avoid unnecessary root usage

Use `sudo` when administrative privileges are required.

### Remove unused accounts

Old accounts can become unnecessary security risks.

### Control group membership

Groups such as administrative or Docker-related groups can provide significant privileges.

### Use strong authentication

For remote administration, SSH keys are commonly preferred over weak password-based authentication.

SSH will be covered later in this learning journey.

---

# 28. Users and Groups in DevOps

Users and groups appear throughout DevOps environments.

### CI/CD

Tools such as Jenkins may run under dedicated service accounts.

For example:

```text
jenkins
```

may own or access build-related files.

### Web servers

A web server may run as a restricted user such as:

```text
www-data
```

depending on the software and distribution.

### Application deployment

An application might use:

```text
deploy
```

or another dedicated account.

### Docker

Container processes can run as specific users instead of root.

Understanding UID/GID mapping becomes important when working with mounted volumes.

### Kubernetes

Containers may run using specific user IDs and security contexts.

Therefore, understanding Linux users and groups now will help later when studying containers and Kubernetes security.

---

# 29. Common Mistakes

## Mistake 1 — Working as root all the time

This increases the risk of accidentally modifying or deleting important system files.

Prefer:

```bash
sudo command
```

when elevated privileges are actually required.

---

## Mistake 2 — Using `usermod -G` without `-a`

For example:

```bash
sudo usermod -G developers rohan
```

can replace existing supplementary groups.

Usually, when adding a group, use:

```bash
sudo usermod -aG developers rohan
```

---

## Mistake 3 — Forgetting to start a new session

After adding a user to a group, the existing login session may not immediately have the updated group membership.

Verify with:

```bash
groups
```

and, when necessary, log out and back in.

---

## Mistake 4 — Changing ownership recursively without checking the path

This:

```bash
sudo chown -R ...
```

can modify many files.

Always verify the target first.

---

## Mistake 5 — Editing `/etc/sudoers` incorrectly

Use:

```bash
sudo visudo
```

instead of directly editing the file with a normal editor.

---

# 30. Important Files and Locations

The following locations are particularly important for this topic:

```text
/etc/passwd
    Local user account information

/etc/shadow
    Password hashes and password-aging information

/etc/group
    Local group information

/etc/gshadow
    Group security information

/etc/sudoers
    Main sudo configuration

/etc/sudoers.d/
    Additional sudo configuration

/home/
    Normal users' home directories

/root/
    Root user's home directory
```

Do not manually modify sensitive account databases unless you specifically understand the consequences. Prefer standard administration commands.

---

# 31. Important Commands

The main commands covered in this topic are:

```text
whoami
id
groups
passwd
useradd
adduser
userdel
usermod
groupadd
groupdel
gpasswd
getent
chown
chgrp
sudo
visudo
```

The accompanying `commands.md` explains these commands in detail, including syntax, important options, arguments, examples, and practical usage.

---

# 32. Practical Lab

The hands-on lab will create a controlled environment for practicing user and group administration.

The lab will cover:

1. Identifying the current user
2. Understanding UID and GID
3. Inspecting `/etc/passwd`
4. Inspecting `/etc/group`
5. Creating a test user
6. Creating a test group
7. Adding a user to a group
8. Verifying group membership
9. Creating a shared directory
10. Assigning group ownership
11. Testing access
12. Changing user information
13. Setting and changing passwords
14. Understanding `sudo`
15. Investigating account information
16. Removing test users and groups safely

The lab will use test accounts rather than modifying important system users.

---

# 33. Example User Management Scenario

Suppose an organization has a development server.

Three developers need access to:

```text
/opt/project
```

Instead of giving permissions separately to every user, an administrator can create:

```text
developers
```

Then:

```text
Rohan ─────┐
Amit ──────┼──> developers
Sneha ─────┘
                ↓
          /opt/project
```

The directory can be assigned to the `developers` group.

The users then receive access based on group membership and filesystem permissions.

This is much easier to manage than changing access individually for every user.

---

# 34. Key Takeaways

After completing this topic, you should understand:

- Linux is a multi-user operating system.
- Users represent identities on a Linux system.
- UIDs are the numeric identities used internally for users.
- The root user normally has UID `0`.
- Normal users generally have limited privileges.
- System users are commonly used to run services with restricted privileges.
- Groups allow multiple users to be managed together.
- Every user has a primary group and can have supplementary groups.
- `/etc/passwd` contains local user account information.
- `/etc/shadow` contains password hashes and related account-aging information.
- `/etc/group` contains local group information.
- `id` displays UID, GID, and group membership.
- `sudo` allows permitted users to execute commands with elevated privileges.
- `usermod -aG` is commonly used to add a user to a supplementary group.
- `chown` changes ownership.
- `chgrp` changes group ownership.
- `getent` can query configured system identity databases.
- Least privilege is an important security principle.
- User and group management is fundamental to Linux administration, DevOps, and security.

---

# Practical Status

- [ ] Users and UIDs understood
- [ ] Root user understood
- [ ] Normal and system users understood
- [ ] Groups understood
- [ ] Primary and supplementary groups understood
- [ ] `/etc/passwd` understood
- [ ] `/etc/shadow` understood
- [ ] `/etc/group` understood
- [ ] `sudo` understood
- [ ] User-management commands practiced
- [ ] Group-management commands practiced
- [ ] Ownership concepts practiced
- [ ] Shared-access lab completed
- [ ] Troubleshooting questions completed
- [ ] Actual lab results documented
- [ ] Changes committed to Git
- [ ] Changes pushed to GitHub

---

## Environment

```text
Host OS: Windows 11
Virtualization: VMware Workstation Pro
Guest OS: Ubuntu Server 24.04 LTS
Network: NAT
Shell: Bash
Repository: devops-journey
```

This topic is part of the Linux section of the DevOps learning journey.
