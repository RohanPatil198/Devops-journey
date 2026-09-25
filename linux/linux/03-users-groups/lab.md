# Linux Users & Groups — Hands-on Lab

This lab is designed for an Ubuntu Server practice VM.

The goal is not just to run commands. The idea is to create a small user and group setup, inspect how Linux stores the information, modify it, and understand what changes when a user becomes a member of a group.

Use a test user for this lab so that you do not accidentally modify an important account.

---

# 1. Lab Environment

The lab uses:

```text
OS       : Ubuntu Server
Shell    : Bash
User     : your normal Ubuntu user
Privileges: sudo
```

We will create:

```text
User  : devuser
Group : developers
```

These are only for practice.

---

# 2. Check Your Current User

Start by checking which account you are using.

```bash
whoami
```

Then:

```bash
id
```

And:

```bash
groups
```

You should see your username, UID, primary GID, and groups.

For example:

```text
uid=1000(rohan) gid=1000(rohan) groups=1000(rohan),27(sudo)
```

Do not copy this output into your documentation unless it is actually what your VM shows.

---

# 3. Check Existing Users

Use:

```bash
getent passwd
```

This can produce a lot of output because Linux has both normal users and system/service accounts.

Instead of going through the entire output, check your own account:

```bash
getent passwd $(whoami)
```

You can also check the root account:

```bash
getent passwd root
```

You should see information similar to:

```text
rohan:x:1000:1000:Rohan:/home/rohan:/bin/bash
```

Pay attention to:

```text
username
UID
GID
home directory
login shell
```

---

# 4. Check Existing Groups

Run:

```bash
getent group
```

Again, there may be many groups.

Check some specific groups:

```bash
getent group sudo
```

and:

```bash
getent group root
```

You can also check your own groups:

```bash
groups $(whoami)
```

---

# 5. Create a Practice Group

Now create the group that we will use in the lab.

```bash
sudo groupadd developers
```

Verify it:

```bash
getent group developers
```

You should get something similar to:

```text
developers:x:1002:
```

The exact GID will probably be different on your system.

That's normal.

Record the actual output in your notes.

---

# 6. Create a Practice User

For Ubuntu, use:

```bash
sudo adduser devuser
```

You will be asked for a password.

You may also be asked for:

```text
Full Name
Room Number
Work Phone
Home Phone
Other
```

These fields are optional.

You can leave them blank by pressing Enter.

At the end, Ubuntu will ask you to confirm the information.

---

# 7. Verify the New User

Check whether the user exists:

```bash
getent passwd devuser
```

Then:

```bash
id devuser
```

And:

```bash
groups devuser
```

You should see something similar to:

```text
uid=1001(devuser) gid=1001(devuser) groups=1001(devuser)
```

The exact numbers will depend on your system.

---

# 8. Check the Home Directory

Because we used `adduser`, a home directory should have been created.

Check:

```bash
ls -ld /home/devuser
```

You should see something similar to:

```text
drwx------ ... devuser devuser ... /home/devuser
```

The exact permissions may differ.

The important thing is to notice:

```text
Owner = devuser
Group = devuser
Location = /home/devuser
```

The detailed meaning of `drwx------` will be covered in the file permissions topic.

---

# 9. Inspect the User's Account Information

Run:

```bash
getent passwd devuser
```

You can also inspect the account with:

```bash
id devuser
```

Compare the information.

Think about:

- What is the UID?
- What is the primary GID?
- What is the home directory?
- What is the login shell?

Write your actual values in your notes.

Example:

```text
Username:
UID:
Primary GID:
Home:
Shell:
```

---

# 10. Add the User to the Developers Group

Now add `devuser` to the `developers` group.

Use:

```bash
sudo usermod -aG developers devuser
```

The important part is:

```text
-aG
```

`-G` specifies supplementary groups.

`-a` means append.

This prevents us from unintentionally replacing the user's existing supplementary groups.

---

# 11. Verify Group Membership

Run:

```bash
id devuser
```

You should now see something similar to:

```text
uid=1001(devuser) gid=1001(devuser) groups=1001(devuser),1002(developers)
```

Also run:

```bash
groups devuser
```

And:

```bash
getent group developers
```

The last command should show `devuser` as a member.

---

# 12. Understand the Group Change

Before adding the user to the group, the account looked roughly like:

```text
devuser
 └── primary group: devuser
```

After adding the group:

```text
devuser
 ├── primary group: devuser
 └── supplementary group: developers
```

This is important in Linux administration because shared access is often managed through groups.

For example, a company might have:

```text
developers
database
docker
monitoring
security
```

Users can be placed into the appropriate groups instead of giving every user unrestricted administrator access.

---

# 13. Test the New User

Switch to the test user:

```bash
su - devuser
```

Enter the password you created earlier.

Now check:

```bash
whoami
```

You should get:

```text
devuser
```

Check:

```bash
id
```

You should see the `developers` group.

Also check:

```bash
pwd
```

You should normally be in:

```text
/home/devuser
```

---

# 14. Understand the Login Session

Group membership is associated with a user's login session.

If you add a currently logged-in user to a new group, the existing shell may not immediately show the new group membership.

For example, if your normal account is:

```text
rohan
```

and you run:

```bash
sudo usermod -aG developers rohan
```

the current shell may still have the old group list.

A new login session normally picks up the change.

You can verify after logging in again with:

```bash
id
```

This is a common issue when administrators say:

> "I added the user to the group, but the user still cannot access it."

---

# 15. Create a Shared Practice Directory

Exit the `devuser` session first:

```bash
exit
```

You should return to your normal user.

Create a directory:

```bash
sudo mkdir /opt/devops-users-lab
```

Check it:

```bash
ls -ld /opt/devops-users-lab
```

Initially, it will normally belong to `root`.

---

# 16. Change the Group Ownership

Change the group to `developers`:

```bash
sudo chgrp developers /opt/devops-users-lab
```

Check:

```bash
ls -ld /opt/devops-users-lab
```

The group should now show:

```text
developers
```

This demonstrates an important concept:

```text
Owner → one user
Group  → one group
Others → everyone else
```

The exact permissions will be studied in the next topic.

---

# 17. Change Both Owner and Group

For practice, you can change the owner and group:

```bash
sudo chown devuser:developers /opt/devops-users-lab
```

Verify:

```bash
ls -ld /opt/devops-users-lab
```

You should now see:

```text
devuser developers
```

This is useful because many real Linux deployments require directories to belong to a particular service account and group.

---

# 18. Check the Directory as `devuser`

Switch to the test user:

```bash
su - devuser
```

Then:

```bash
cd /opt/devops-users-lab
```

Check:

```bash
pwd
```

If the directory permissions allow the user to enter it, you should be able to access it.

If you receive:

```text
Permission denied
```

don't immediately change permissions randomly.

First inspect:

```bash
ls -ld /opt/devops-users-lab
```

Then check:

```bash
id
```

This is the beginning of the troubleshooting process.

---

# 19. Create a Test File

If access is allowed:

```bash
touch test.txt
```

Check:

```bash
ls -l
```

You should see the new file.

Check the current user:

```bash
whoami
```

The file should normally be associated with the user that created it.

---

# 20. Check File Ownership

Run:

```bash
ls -l test.txt
```

You may see something similar to:

```text
-rw-r--r-- 1 devuser devuser 0 Sep 25 15:20 test.txt
```

For now, concentrate only on:

```text
Owner = devuser
Group = devuser
```

The `-rw-r--r--` part will be explained properly in the next topic.

---

# 21. Practice `chown`

Exit back to your normal account:

```bash
exit
```

Find the test file:

```bash
ls -l /opt/devops-users-lab/test.txt
```

Change its owner and group:

```bash
sudo chown devuser:developers /opt/devops-users-lab/test.txt
```

Verify:

```bash
ls -l /opt/devops-users-lab/test.txt
```

You should now see:

```text
devuser developers
```

---

# 22. Check Group Membership Again

Run:

```bash
id devuser
```

Then:

```bash
getent group developers
```

You should see the relationship:

```text
devuser → developers
```

This is the basic relationship that we will use when learning Linux permissions.

---

# 23. Investigate `/etc/passwd`

Run:

```bash
grep '^devuser:' /etc/passwd
```

You should see the account entry.

You can also use:

```bash
getent passwd devuser
```

Compare the two outputs.

The important point is that `getent` is usually preferable when you want to query the system's configured account database rather than manually parsing a file.

---

# 24. Investigate `/etc/group`

Run:

```bash
grep '^developers:' /etc/group
```

Or:

```bash
getent group developers
```

Compare the results.

You should see the group and its members.

---

# 25. Check the Password Database

You can verify that the user has an entry in `/etc/shadow`:

```bash
sudo grep '^devuser:' /etc/shadow
```

Do not copy the password hash into your GitHub repository.

The purpose of this step is only to understand where Linux stores password-related information.

---

# 26. Practice `sudo -l`

As your normal user, run:

```bash
sudo -l
```

This shows which commands your account is allowed to execute through sudo.

Do not modify sudo configuration just for this lab.

The important concept is:

```text
sudo
  ↓
checks authorization
  ↓
allows specific administrative operations
```

---

# 27. Small Troubleshooting Exercise

Suppose you add a user to a group:

```bash
sudo usermod -aG developers devuser
```

But when the user checks:

```bash
groups
```

the new group is missing.

What should you check?

First:

```bash
id devuser
```

Then:

```bash
getent group developers
```

If the group contains the user, the account configuration may already be correct.

If the user is currently logged in, start a new login session and check again.

This is a common Linux administration issue.

---

# 28. Another Troubleshooting Exercise

Suppose:

```bash
id devuser
```

shows:

```text
groups=1001(devuser),1002(developers)
```

but the user cannot access a directory.

Do not assume the group configuration is wrong.

Check the directory:

```bash
ls -ld /path/to/directory
```

Then check the user's identity:

```bash
id devuser
```

The next question is whether the directory's permissions actually allow the group to access it.

That leads directly into the next topic:

**Linux File Permissions and Ownership.**

---

# 29. Cleanup

After completing the lab, remove the test directory:

```bash
sudo rm -rf /opt/devops-users-lab
```

Be extremely careful with `rm -rf`.

Always verify the path before pressing Enter.

Then remove the test user:

```bash
sudo userdel -r devuser
```

Remove the test group:

```bash
sudo groupdel developers
```

Verify:

```bash
getent passwd devuser
```

If the user was removed, this should return no result.

Check the group:

```bash
getent group developers
```

It should also return no result.

---

# 30. Final Verification

Run:

```bash
getent passwd devuser
```

```bash
getent group developers
```

Both should normally return nothing after cleanup.

Check your own account:

```bash
whoami
```

```bash
id
```

Make sure you are back on your normal user account.

---

# 31. What You Practiced

By completing this lab, you should now understand:

- how Linux identifies users with UIDs
- how Linux identifies groups with GIDs
- primary groups
- supplementary groups
- user home directories
- `/etc/passwd`
- `/etc/shadow`
- `/etc/group`
- creating users
- creating groups
- adding users to groups
- modifying users
- checking account information
- changing ownership
- changing group ownership
- using `sudo`
- why group changes may require a new login session
- basic troubleshooting of user/group access

---

# 32. Practical Questions

After completing the lab, try answering these without looking at the commands file.

### Question 1

What is the difference between UID and GID?

### Question 2

What is the difference between a primary group and a supplementary group?

### Question 3

Why do we normally use:

```bash
usermod -aG developers devuser
```

instead of:

```bash
usermod -G developers devuser
```

### Question 4

Where is basic user account information stored?

### Question 5

Where are password hashes stored?

### Question 6

What is the purpose of `/etc/group`?

### Question 7

What does this command do?

```bash
chown devuser:developers file.txt
```

### Question 8

Why should you use `sudo` instead of working as root all the time?

### Question 9

A user was added to a group but their current terminal does not show the group. What could be the reason?

### Question 10

What is the difference between:

```bash
chown
```

and:

```bash
chgrp
```

Try answering these from understanding rather than memorizing definitions.

---

# 33. DevOps Connection

Users and groups are not just Linux administration concepts. They appear frequently in DevOps environments.

For example, a CI/CD server such as Jenkins may run using a dedicated service account rather than root.

A web server may run under a restricted account such as `www-data`.

Deployment directories may be owned by a deployment user and a specific group.

Docker and Kubernetes also use user/group identities when controlling access to files and processes.

A typical server may therefore contain several different accounts:

```text
administrator
developer
jenkins
www-data
deploy
monitoring
```

Giving every service root access would create unnecessary security risk.

The better approach is to give each account only the access it actually needs.

This is the basic idea of **least privilege**.

---

# 34. Evidence for GitHub

Do not fill this lab with made-up output.

After running the commands, add small notes showing what actually happened on your VM.

For example:

```text
User created:
devuser

UID:
1001

Primary GID:
1001

Supplementary group:
developers

Home directory:
/home/devuser
```

The numbers may be different on your system. Use your actual values.

This makes the repository genuine proof that you performed the lab.

---

# 35. Completion Checklist

Before marking the topic complete, confirm that you can:

- [ ] Explain UID and GID
- [ ] Explain primary and supplementary groups
- [ ] Check your current user with `whoami`
- [ ] Inspect users with `id` and `getent`
- [ ] Create a user
- [ ] Create a group
- [ ] Add a user to a group
- [ ] Verify group membership
- [ ] Explain why `-aG` is important
- [ ] Change ownership with `chown`
- [ ] Change group ownership with `chgrp`
- [ ] Understand `/etc/passwd`
- [ ] Understand `/etc/shadow`
- [ ] Understand `/etc/group`
- [ ] Explain the purpose of `sudo`
- [ ] Troubleshoot a basic group-membership issue
- [ ] Clean up the test account and group

---

# 36. Commit the Work

After completing the lab and updating the documentation with your actual observations:

```bash
cd ~/devops-journey
```

Check your changes:

```bash
git status
```

Review the files:

```bash
git diff
```

Then:

```bash
git add linux/03-users-groups/
```

Commit:

```bash
git commit -m "Add Linux users and groups commands and hands-on lab"
```

Push:

```bash
git push
```

Your GitHub history will then show that you completed the Users & Groups topic as part of the Linux learning journey.
