# Linux File Permissions & Ownership — Hands-on Lab

This lab is designed to make Linux permissions easier to understand by actually creating users, groups, files, and directories and then testing access.

We will use a small shared-project scenario.

The idea is:

```text
developers
     |
     ├── devuser
     └── your normal user

shared project directory
     |
     ├── files
     └── scripts
```

We will change ownership and permissions and observe what happens.

Use the Ubuntu VM for this lab.

---

# 1. Before Starting

Check your current user:

```bash
whoami
```

Then:

```bash
id
```

Make sure you are using your normal practice account.

Do not perform this lab using important production accounts or system directories.

---

# 2. Create the Lab Group

Create a group called:

```text
developers
```

Run:

```bash
sudo groupadd developers
```

Check it:

```bash
getent group developers
```

The output will look similar to:

```text
developers:x:1002:
```

The GID may be different on your VM.

Do not worry if your number does not match the example.

---

# 3. Create the Test User

Create a user:

```bash
sudo adduser devuser
```

Set a password when prompted.

The other information is optional.

Verify:

```bash
id devuser
```

You should see a UID and GID.

Also check:

```bash
getent passwd devuser
```

---

# 4. Add the User to the Developers Group

Run:

```bash
sudo usermod -aG developers devuser
```

Verify:

```bash
id devuser
```

You should now see:

```text
developers
```

in the user's supplementary groups.

Also:

```bash
groups devuser
```

---

# 5. Create the Practice Directory

We will create the lab under `/opt`.

Run:

```bash
sudo mkdir -p /opt/devops-permissions-lab
```

The `-p` option creates the directory if the parent path is required and avoids an error when appropriate.

Check it:

```bash
ls -ld /opt/devops-permissions-lab
```

Initially, it will normally belong to:

```text
root root
```

---

# 6. Create Some Practice Files

Create three files:

```bash
sudo touch /opt/devops-permissions-lab/app.conf
sudo touch /opt/devops-permissions-lab/readme.txt
sudo touch /opt/devops-permissions-lab/deploy.sh
```

Check them:

```bash
sudo ls -l /opt/devops-permissions-lab
```

You will probably see that `root` owns them because they were created using `sudo`.

This is a good opportunity to understand why ownership matters.

---

# 7. Change the Ownership

Make `devuser` the owner and `developers` the group:

```bash
sudo chown devuser:developers /opt/devops-permissions-lab/app.conf
sudo chown devuser:developers /opt/devops-permissions-lab/readme.txt
sudo chown devuser:developers /opt/devops-permissions-lab/deploy.sh
```

Check:

```bash
ls -l /opt/devops-permissions-lab
```

You should see something similar to:

```text
-rw-r--r-- 1 devuser developers 0 ... app.conf
-rw-r--r-- 1 devuser developers 0 ... readme.txt
-rw-r--r-- 1 devuser developers 0 ... deploy.sh
```

The exact timestamps and sizes will differ.

---

# 8. Understand the Current Permissions

The files will normally start with something similar to:

```text
-rw-r--r--
```

Break it down:

```text
Owner  = rw-
Group  = r--
Others = r--
```

So:

```text
Owner  → read + write
Group  → read
Others → read
```

This means the owner can modify the file, but members of the group cannot modify it yet.

---

# 9. Check Numeric Permissions

Run:

```bash
stat -c '%a %n' /opt/devops-permissions-lab/*
```

You may see:

```text
644 /opt/devops-permissions-lab/app.conf
644 /opt/devops-permissions-lab/readme.txt
644 /opt/devops-permissions-lab/deploy.sh
```

The exact values depend on the permissions created on your system.

---

# 10. Change `app.conf` to `640`

Run:

```bash
sudo chmod 640 /opt/devops-permissions-lab/app.conf
```

Check:

```bash
ls -l /opt/devops-permissions-lab/app.conf
```

You should now have:

```text
-rw-r-----
```

Meaning:

```text
Owner  → read + write
Group  → read
Others → no access
```

This is a more restrictive configuration for a configuration file.

---

# 11. Change `deploy.sh` to `750`

Run:

```bash
sudo chmod 750 /opt/devops-permissions-lab/deploy.sh
```

Check:

```bash
ls -l /opt/devops-permissions-lab/deploy.sh
```

Expected permission pattern:

```text
-rwxr-x---
```

Meaning:

```text
Owner  → read + write + execute
Group  → read + execute
Others → no access
```

This is a useful pattern for a script that should be usable by the owner and members of a trusted group.

---

# 12. Make the Script Actually Executable

Create some content in the script.

Run:

```bash
sudo bash -c 'printf "#!/bin/bash\necho \"Deployment script executed\"\n" > /opt/devops-permissions-lab/deploy.sh'
```

Then:

```bash
sudo chmod 750 /opt/devops-permissions-lab/deploy.sh
```

Check:

```bash
cat /opt/devops-permissions-lab/deploy.sh
```

You should see:

```text
#!/bin/bash
echo "Deployment script executed"
```

---

# 13. Run the Script

Try:

```bash
/opt/devops-permissions-lab/deploy.sh
```

If your current user does not have execute permission, you may receive:

```text
Permission denied
```

This is intentional.

Check your identity:

```bash
id
```

Then check the file:

```bash
ls -l /opt/devops-permissions-lab/deploy.sh
```

Think about which permission class applies to your user.

This is the important part of the exercise.

---

# 14. Test as `devuser`

Switch to the test user:

```bash
su - devuser
```

Enter the password created earlier.

Check:

```bash
whoami
```

Then:

```bash
id
```

Make sure `developers` appears in the group list.

---

# 15. Test the Configuration File

As `devuser`, run:

```bash
cat /opt/devops-permissions-lab/app.conf
```

It should be readable because `devuser` is the owner.

Now try writing to it:

```bash
echo "database=dev" >> /opt/devops-permissions-lab/app.conf
```

This should work because the owner has write permission.

Check:

```bash
cat /opt/devops-permissions-lab/app.conf
```

You should see:

```text
database=dev
```

---

# 16. Test the Script

As `devuser`, run:

```bash
/opt/devops-permissions-lab/deploy.sh
```

You should see:

```text
Deployment script executed
```

Why?

Because `devuser` is the owner and the owner has:

```text
rwx
```

---

# 17. Test Group Access

The `developers` group has:

```text
r-x
```

on the script.

This means a member of the group should be able to read and execute it, but not modify it.

This distinction is very important.

For example, a shared deployment script might be executable by developers but only modifiable by a controlled account.

---

# 18. Test With a Second User

For a more useful demonstration, create another test user:

```bash
sudo adduser tester
```

Add the user to the developers group:

```bash
sudo usermod -aG developers tester
```

Verify:

```bash
id tester
```

---

# 19. Switch to `tester`

From the current `devuser` shell:

```bash
exit
```

Then switch:

```bash
su - tester
```

Check:

```bash
whoami
id
```

You should see:

```text
tester
```

and:

```text
developers
```

in the group list.

---

# 20. Test Reading `app.conf`

Run:

```bash
cat /opt/devops-permissions-lab/app.conf
```

The file has:

```text
640
```

which means:

```text
Owner  = rw-
Group  = r--
Others = ---
```

`tester` is not the owner, but is a member of `developers`.

Therefore the group permissions apply.

Reading should work.

---

# 21. Test Writing to `app.conf`

Now try:

```bash
echo "test=value" >> /opt/devops-permissions-lab/app.conf
```

You should receive:

```text
Permission denied
```

Why?

Because the group has only:

```text
r--
```

There is no group write permission.

This is exactly what we want to demonstrate.

---

# 22. Change Group Write Permission

Exit back to your normal account:

```bash
exit
```

Then run:

```bash
sudo chmod 660 /opt/devops-permissions-lab/app.conf
```

Now:

```text
660
```

means:

```text
Owner  = rw-
Group  = rw-
Others = ---
```

Check:

```bash
ls -l /opt/devops-permissions-lab/app.conf
```

---

# 23. Test Again as `tester`

Switch to `tester`:

```bash
su - tester
```

Then:

```bash
echo "test=value" >> /opt/devops-permissions-lab/app.conf
```

This time it should work because `tester` belongs to `developers`, and the group now has write permission.

Check:

```bash
cat /opt/devops-permissions-lab/app.conf
```

This demonstrates why groups are useful for shared resources.

---

# 24. Understand What Just Happened

The setup is now approximately:

```text
app.conf

Owner:
devuser

Group:
developers

Permissions:
660

Owner:
rw-

Group:
rw-

Others:
---
```

`devuser` can read/write.

Any member of `developers` can read/write.

Users outside the group cannot access the file through these permission bits.

This is a very common Linux server pattern.

---

# 25. Test Directory Permissions

Now check the directory itself:

```bash
ls -ld /opt/devops-permissions-lab
```

Suppose it shows:

```text
drwxr-xr-x
```

Remember that directory permissions are different.

For a directory:

```text
r = list
w = create/delete/rename entries
x = traverse
```

This is why directory permissions are often just as important as file permissions.

---

# 26. Restrict the Directory

Exit back to your normal user:

```bash
exit
```

Change the directory ownership:

```bash
sudo chown devuser:developers /opt/devops-permissions-lab
```

Now give the owner and group full access:

```bash
sudo chmod 770 /opt/devops-permissions-lab
```

Check:

```bash
ls -ld /opt/devops-permissions-lab
```

Expected:

```text
drwxrwx---
```

Now:

```text
Owner  → rwx
Group  → rwx
Others → ---
```

This means only the owner and developers group should be able to access the directory.

---

# 27. Test Directory Access

Switch to `tester`:

```bash
su - tester
```

Run:

```bash
cd /opt/devops-permissions-lab
```

Because `tester` belongs to `developers`, this should work.

Now:

```bash
touch tester-file.txt
```

It should create a file.

Check:

```bash
ls -l
```

Notice who owns the new file and which group it has.

This is an important observation.

---

# 28. Enable SGID on the Shared Directory

Exit:

```bash
exit
```

Now enable SGID:

```bash
sudo chmod g+s /opt/devops-permissions-lab
```

Check:

```bash
ls -ld /opt/devops-permissions-lab
```

You should see an `s` in the group permission position.

Something similar to:

```text
drwxrws---
```

The important part is:

```text
rws
  ^
 SGID
```

---

# 29. Test SGID

Switch to `tester`:

```bash
su - tester
```

Go into the directory:

```bash
cd /opt/devops-permissions-lab
```

Create a file:

```bash
touch shared-file.txt
```

Check:

```bash
ls -l shared-file.txt
```

The file should normally inherit the directory's group:

```text
developers
```

This is one of the practical reasons SGID directories are useful.

Multiple developers can work in a shared directory while newly created files naturally belong to the shared group.

---

# 30. Symbolic Permission Practice

Exit:

```bash
exit
```

Create another file:

```bash
sudo touch /opt/devops-permissions-lab/test-symbolic.txt
```

Check:

```bash
ls -l /opt/devops-permissions-lab/test-symbolic.txt
```

Now remove read permission from others:

```bash
sudo chmod o-r /opt/devops-permissions-lab/test-symbolic.txt
```

Check again:

```bash
ls -l /opt/devops-permissions-lab/test-symbolic.txt
```

Now add group write:

```bash
sudo chmod g+w /opt/devops-permissions-lab/test-symbolic.txt
```

Check again.

This demonstrates how symbolic mode changes individual permission bits without necessarily replacing everything else.

---

# 31. Practice `umask`

Check your current value:

```bash
umask
```

Now temporarily set:

```bash
umask 077
```

Create:

```bash
touch /tmp/umask-test.txt
mkdir /tmp/umask-test-dir
```

Check:

```bash
ls -l /tmp/umask-test.txt
ls -ld /tmp/umask-test-dir
```

You should see restrictive permissions.

This demonstrates that `umask` affects the permissions assigned when new objects are created.

---

# 32. Reset Your Shell's `umask`

For the current shell, you can restore a common value:

```bash
umask 022
```

Check:

```bash
umask
```

Important: if your normal system had a different umask before the lab, use that original value instead.

You can check your normal value in a fresh terminal if necessary.

---

# 33. Practice `namei`

Now inspect the full path:

```bash
namei -l /opt/devops-permissions-lab/app.conf
```

Read the output from left to right.

You should be able to identify permissions for:

```text
/
opt
devops-permissions-lab
app.conf
```

This is useful when troubleshooting a `Permission denied` error.

---

# 34. Create a Permission Problem Intentionally

Create:

```bash
sudo touch /opt/devops-permissions-lab/private.txt
```

Set:

```bash
sudo chmod 600 /opt/devops-permissions-lab/private.txt
```

Change owner:

```bash
sudo chown devuser:developers /opt/devops-permissions-lab/private.txt
```

Now inspect:

```bash
ls -l /opt/devops-permissions-lab/private.txt
```

It should look approximately like:

```text
-rw------- ... devuser developers ... private.txt
```

The group has no permissions.

---

# 35. Test the Permission Problem

Switch to `tester`:

```bash
su - tester
```

Try:

```bash
cat /opt/devops-permissions-lab/private.txt
```

You should receive:

```text
Permission denied
```

Now investigate rather than immediately changing the permission.

Run:

```bash
whoami
```

```bash
id
```

```bash
ls -l /opt/devops-permissions-lab/private.txt
```

Ask yourself:

```text
Am I the owner?
Am I in the file's group?
What permissions does the group have?
```

The answer should explain why access is denied.

---

# 36. Fix the Problem Properly

Exit:

```bash
exit
```

If developers genuinely need to read the file, change the permission to:

```bash
sudo chmod 640 /opt/devops-permissions-lab/private.txt
```

Now:

```text
Owner  = rw-
Group  = r--
Others = ---
```

Because `tester` belongs to `developers`, the group permission now allows reading.

Test:

```bash
su - tester
```

Then:

```bash
cat /opt/devops-permissions-lab/private.txt
```

It should work.

This is a better solution than:

```bash
chmod 777 private.txt
```

because we gave only the required access.

---

# 37. Optional ACL Practice

If `setfacl` is available, check:

```bash
which setfacl
```

If it is installed, create:

```bash
sudo touch /opt/devops-permissions-lab/acl-test.txt
```

Set:

```bash
sudo chmod 600 /opt/devops-permissions-lab/acl-test.txt
```

Now give `tester` read access:

```bash
sudo setfacl -m u:tester:r /opt/devops-permissions-lab/acl-test.txt
```

Check:

```bash
getfacl /opt/devops-permissions-lab/acl-test.txt
```

You should see an additional user ACL for `tester`.

Test as `tester`:

```bash
su - tester
```

Then:

```bash
cat /opt/devops-permissions-lab/acl-test.txt
```

The user should be able to read it even though the normal `other` permission is not allowing access.

This demonstrates why ACLs are useful when the normal owner/group/others model is not enough.

---

# 38. Check the Effective Permissions

When ACLs are used, `getfacl` can show a `mask` entry.

For example:

```text
user::rw-
user:tester:r--
group::---
mask::r--
other::---
```

The ACL mask limits the effective permissions of named users/groups and the group class.

This is an advanced concept, so you do not need to memorize it yet. The important point is that ACLs add another layer that must be considered during troubleshooting.

---

# 39. Permission Troubleshooting Challenge

Now intentionally create this situation:

```text
Directory:
developers group

File:
Owner = devuser
Group = developers
Permissions = 640

User:
tester
Member of developers
```

Ask:

### Can `tester` read the file?

Yes.

### Can `tester` modify it?

No.

Why?

Because:

```text
640
```

means:

```text
Owner  = rw-
Group  = r--
Others = ---
```

The group has read permission but not write permission.

---

# 40. Another Challenge

Change the file to:

```bash
sudo chmod 600 /opt/devops-permissions-lab/app.conf
```

Now ask:

> Can `tester` read it?

No.

Why?

Because:

```text
600
```

means:

```text
Owner  = rw-
Group  = ---
Others = ---
```

Even though `tester` belongs to `developers`, the group has no permissions.

---

# 41. Another Challenge

Change the file to:

```bash
sudo chmod 664 /opt/devops-permissions-lab/app.conf
```

Now:

```text
Owner  = rw-
Group  = rw-
Others = r--
```

Ask:

> Can `tester` modify it?

Yes, assuming `tester` is a member of the file's group and no ACL or other mechanism changes the effective access.

This is a good example of why group membership matters.

---

# 42. Cleanup

After completing the exercises, remove the test users.

First return to your normal account:

```bash
exit
```

Make sure:

```bash
whoami
```

shows your normal user.

Then remove the test users:

```bash
sudo userdel -r devuser
sudo userdel -r tester
```

Remove the practice group:

```bash
sudo groupdel developers
```

Remove the lab directory:

```bash
sudo rm -rf /opt/devops-permissions-lab
```

Be extremely careful with `rm -rf`.

Before running it, verify:

```bash
ls -ld /opt/devops-permissions-lab
```

and make sure you are deleting only your practice directory.

Remove the temporary umask files:

```bash
rm -f /tmp/umask-test.txt
rm -rf /tmp/umask-test-dir
```

---

# 43. Final Verification

Check that the test user is gone:

```bash
getent passwd devuser
```

```bash
getent passwd tester
```

Both should return no result.

Check the group:

```bash
getent group developers
```

It should also return no result.

Check the lab directory:

```bash
ls -ld /opt/devops-permissions-lab
```

It should no longer exist.

---

# 44. What You Should Understand After This Lab

You should now be able to look at:

```text
-rwxr-x---
```

and explain:

```text
Owner  → rwx
Group  → r-x
Others → ---
```

You should also understand that:

```text
File permissions
      +
File ownership
      +
User identity
      +
Group membership
      ↓
Linux access decision
```

You practiced:

- `chmod`
- `chown`
- `chgrp`
- numeric permissions
- symbolic permissions
- directory permissions
- group-based access
- SGID
- `umask`
- ACLs
- `namei`
- permission troubleshooting
- least-privilege thinking

---

# 45. Interview Practice

Try answering these without looking at the README.

### 1. What are Linux file permissions?

### 2. What does `rwxr-xr--` mean?

### 3. What is the difference between owner, group, and others?

### 4. What does `chmod 755 file.sh` do?

### 5. What is the difference between `chmod 755` and `chmod u+x`?

### 6. What is the difference between `chown` and `chgrp`?

### 7. What does `chmod -R` do?

### 8. Why is `chmod -R 777` considered a bad practice?

### 9. What is the difference between file `x` permission and directory `x` permission?

### 10. What is `umask`?

### 11. What is SUID?

### 12. What is SGID and why is it useful on shared directories?

### 13. What is the sticky bit?

### 14. What are ACLs?

### 15. A user gets `Permission denied`. What would you check?

### 16. Why can a user have permission on a file but still be unable to access it?

### 17. What is the purpose of `namei -l`?

### 18. What does `640` mean?

### 19. What does `600` mean?

### 20. What does `750` mean?

Try to explain the answers in your own words rather than memorizing definitions.

---

# 46. GitHub Documentation

Do not copy all the example output from this lab directly into your repository.

After performing the lab, add your actual observations.

For example:

```text
Practice user:
devuser

UID:
[actual UID from my VM]

Practice group:
developers

GID:
[actual GID from my VM]

Shared directory:
/opt/devops-permissions-lab

Permission tested:
640

Observed:
Group member could read but could not write.
```

This makes the repository evidence of actual practice instead of a collection of copied commands.

---

# 47. Git Commit

Once you have completed the lab and updated the documentation:

```bash
cd ~/devops-journey
```

Check your changes:

```bash
git status
```

Review them:

```bash
git diff
```

Add the topic:

```bash
git add linux/04-file-permissions/
```

Commit:

```bash
git commit -m "Add Linux file permissions commands and hands-on lab"
```

Push:

```bash
git push
```

After pushing, verify the files on GitHub.

The completed topic should contain:

```text
linux/
└── 04-file-permissions/
    ├── README.md
    ├── commands.md
    └── lab.md
```
