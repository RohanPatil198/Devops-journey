# Linux Process Management — Hands-on Lab

This lab focuses on actually working with Linux processes rather than only reading about them.

The exercises use controlled processes such as `sleep`, so they are safe to practice on a Linux VM.

---

# 1. Check the Current System

Start by checking your current user and system uptime:

```bash
whoami
```

```bash
uptime
```

Then check the current processes:

```bash
ps
```

You will normally see processes belonging to your current shell.

Now run:

```bash
ps aux
```

Spend some time looking at the output.

Identify:

- Your username
- PID
- CPU usage
- Memory usage
- Process state
- Command

---

# 2. Understand `ps -ef`

Run:

```bash
ps -ef
```

Look specifically at:

```text
UID
PID
PPID
TTY
CMD
```

Pick one process and identify:

```text
PID:
PPID:
User:
Command:
```

Now inspect that process:

```bash
ps -p PID -f
```

Replace `PID` with an actual PID.

The purpose of this exercise is to understand that a PID identifies a specific running process.

---

# 3. Understand Parent and Child Processes

Run:

```bash
pstree -p
```

Look at the process hierarchy.

You should see relationships similar to:

```text
systemd
  └─ ...
      └─ bash
```

Your exact tree will depend on the Ubuntu VM.

Now check PID 1:

```bash
ps -p 1 -f
```

Identify which process is running as PID 1.

On a standard Ubuntu Server installation, this is normally `systemd`.

---

# 4. Create a Controlled Process

Start:

```bash
sleep 300
```

The terminal will appear to wait.

The `sleep` process is running in the foreground.

Open another terminal session to the VM.

Find the process:

```bash
pgrep -a sleep
```

You should see the `sleep` process and its PID.

Now inspect it:

```bash
ps -p PID -f
```

Replace `PID` with the actual PID.

---

# 5. Stop the Foreground Process

Return to the terminal where:

```bash
sleep 300
```

is running.

Press:

```text
Ctrl+C
```

The command should terminate before the 300 seconds have completed.

This demonstrates the relationship between:

```text
Ctrl+C
   ↓
SIGINT
   ↓
Foreground process
```

---

# 6. Start a Background Process

Run:

```bash
sleep 300 &
```

The shell should return immediately.

You may see:

```text
[1] 1234
```

The exact numbers will be different.

Remember:

```text
[1]  → shell job number
1234 → PID
```

Check:

```bash
jobs
```

Then:

```bash
pgrep -a sleep
```

You should be able to identify the same process.

---

# 7. Understand Job Control

Start another process:

```bash
sleep 300
```

Press:

```text
Ctrl+Z
```

The process should become stopped.

Check:

```bash
jobs
```

You should see the job marked as stopped.

Now continue it in the background:

```bash
bg
```

Check:

```bash
jobs
```

It should now be running in the background.

Bring it back to the foreground:

```bash
fg
```

Then terminate it using:

```text
Ctrl+C
```

This exercise demonstrates:

```text
Foreground
    ↓
Ctrl+Z
    ↓
Stopped
    ↓
bg
    ↓
Background
    ↓
fg
    ↓
Foreground
```

---

# 8. Inspect Process Information Through `/proc`

Start a background process:

```bash
sleep 300 &
```

Find its PID:

```bash
pgrep -a sleep
```

Assume the PID is:

```text
2500
```

Do not use `2500` literally unless that is actually your PID.

Check:

```bash
cat /proc/2500/status
```

Look for:

```text
Name
State
Pid
PPid
Uid
Gid
Threads
```

Now check:

```bash
ls -l /proc/2500/exe
```

And:

```bash
ls -l /proc/2500/fd/
```

These exercises show that Linux exposes detailed process information through `/proc`.

---

# 9. Compare `ps` With `/proc`

For the same process, run:

```bash
ps -p PID -f
```

Then:

```bash
cat /proc/PID/status
```

Compare the information.

You should recognize information such as:

```text
PID
PPID
User/UID
State
```

The purpose is to understand that commands such as `ps` are presenting information maintained by the operating system, with `/proc` exposing much of the underlying process information directly.

---

# 10. Monitor Processes With `top`

Run:

```bash
top
```

Spend some time observing:

- Load average
- CPU usage
- Memory usage
- Number of processes
- Individual process CPU usage
- Individual process memory usage

Press:

```text
P
```

to sort by CPU usage.

Press:

```text
M
```

to sort by memory usage.

Exit:

```text
q
```

Do not terminate processes from `top` during this exercise unless you intentionally created the test process.

---

# 11. Check Memory and Load

Run:

```bash
free -h
```

Observe:

```text
total
used
free
available
```

Then:

```bash
uptime
```

Observe the three load averages.

Finally:

```bash
vmstat 2
```

Allow it to run for a few seconds.

Stop it with:

```text
Ctrl+C
```

---

# 12. Practice Process Termination

Create a test process:

```bash
sleep 300 &
```

Find it:

```bash
pgrep -a sleep
```

Inspect it:

```bash
ps -p PID -f
```

Send SIGTERM:

```bash
kill PID
```

Verify:

```bash
ps -p PID
```

The process should no longer appear.

This demonstrates graceful termination.

---

# 13. Practice SIGKILL Carefully

Create another test process:

```bash
sleep 300 &
```

Find its PID:

```bash
pgrep -a sleep
```

For this controlled test only, send:

```bash
kill -9 PID
```

Then verify:

```bash
ps -p PID
```

The purpose is to understand the difference between:

```text
SIGTERM
```

and:

```text
SIGKILL
```

Do not make `kill -9` your normal process-management method.

---

# 14. Process Priority

Start:

```bash
nice -n 10 sleep 300 &
```

Find the PID:

```bash
pgrep -a sleep
```

Check its nice value:

```bash
ps -o pid,ni,comm -p PID
```

You should see the nice value associated with the process.

Now create another test process:

```bash
sleep 300 &
```

Check its nice value:

```bash
ps -o pid,ni,comm -p PID
```

Compare the two.

The first process was started with a higher nice value and therefore a lower scheduling priority.

---

# 15. Change Priority With `renice`

Use the PID of a controlled `sleep` process.

Check its current value:

```bash
ps -o pid,ni,comm -p PID
```

Change it:

```bash
renice 15 -p PID
```

Verify:

```bash
ps -o pid,ni,comm -p PID
```

You should see the new nice value.

Terminate the test process when finished:

```bash
kill PID
```

---

# 16. Find Processes by Name

Create a few test processes:

```bash
sleep 300 &
sleep 300 &
```

Now:

```bash
pgrep -a sleep
```

You should see multiple PIDs.

This demonstrates why name-based commands such as `pkill` must be used carefully.

Terminate only your test processes individually:

```bash
kill PID1
kill PID2
```

---

# 17. Investigate Open Files

Start a controlled process:

```bash
sleep 300 &
```

Find its PID:

```bash
pgrep -a sleep
```

Then:

```bash
lsof -p PID
```

You may see files and descriptors associated with the process.

Now inspect:

```bash
ls -l /proc/PID/fd/
```

The two views provide different ways to understand resources associated with a process.

---

# 18. Check Process State

Start:

```bash
sleep 300 &
```

Find the PID:

```bash
pgrep -a sleep
```

Then:

```bash
ps -o pid,ppid,stat,cmd -p PID
```

Look at the `STAT` column.

For a normal `sleep` process, it will commonly be in a sleeping state.

The exact state can vary depending on timing.

---

# 19. Observe a Process Tree

Start a background process:

```bash
sleep 300 &
```

Find its PID:

```bash
pgrep -a sleep
```

Then inspect its parent:

```bash
ps -o pid,ppid,cmd -p PID
```

Now use:

```bash
pstree -p
```

Try to locate the shell and the `sleep` process.

This helps connect:

```text
Shell
  ↓
Child process
```

with the PID/PPID information shown by `ps`.

---

# 20. Optional CPU Investigation

If you want to practice identifying a high-CPU process, use a controlled test.

Start:

```bash
yes > /dev/null &
```

Immediately find it:

```bash
pgrep -a yes
```

Then:

```bash
top
```

You should see the `yes` process consuming significant CPU.

This is intentional.

**Do not leave it running.**

Find its PID and terminate it:

```bash
kill PID
```

Verify:

```bash
pgrep -a yes
```

If necessary, terminate the controlled test process with:

```bash
kill -9 PID
```

The important lesson is the troubleshooting workflow:

```text
Observe high CPU
      ↓
Identify PID
      ↓
Identify command
      ↓
Determine whether usage is expected
      ↓
Take appropriate action
```

---

# 21. Troubleshooting Exercise

Imagine someone reports:

> "My Linux VM is slow."

Start with:

```bash
uptime
```

Then:

```bash
free -h
```

Then:

```bash
top
```

Identify whether the problem appears related to:

- CPU
- Memory
- High load
- A particular process

Then investigate a suspicious PID with:

```bash
ps -p PID -f
```

and:

```bash
cat /proc/PID/status
```

The important part is not simply finding the largest number. Try to understand **what the process is and why it is consuming resources**.

---

# 22. Cleanup

Check for the test processes you created:

```bash
pgrep -a sleep
```

If any test processes are still running, terminate them individually:

```bash
kill PID
```

Also check:

```bash
pgrep -a yes
```

Make sure the intentional CPU test is no longer running.

Finally:

```bash
jobs
```

Check that you do not have unwanted background jobs remaining in your current shell.

---

# 23. Practical Questions

After completing the lab, you should be able to answer these without looking at the commands:

### Question 1
What is the difference between a program and a process?

### Question 2
What is a PID?

### Question 3
What is a PPID?

### Question 4
What is PID 1 used for?

### Question 5
What is the difference between `ps` and `top`?

### Question 6
What is the difference between `ps aux` and `ps -ef`?

### Question 7
What happens when you run:

```bash
command &
```

### Question 8
What does `Ctrl+Z` do?

### Question 9
What is the difference between `bg` and `fg`?

### Question 10
What is a signal?

### Question 11
Why should SIGTERM generally be preferred before SIGKILL?

### Question 12
What is a zombie process?

### Question 13
How would you identify a process consuming high CPU?

### Question 14
How would you find the parent of a process?

### Question 15
What information can be found under `/proc/PID/`?

### Question 16
What does the nice value control?

### Question 17
What is the difference between a process and a service?

---

# 24. Useful Troubleshooting Flow

When dealing with an unknown process problem, think in this order:

```text
Is the system slow?
       ↓
Check uptime / load
       ↓
Check CPU and memory
       ↓
Identify suspicious process
       ↓
Find PID
       ↓
Check user and command
       ↓
Check PPID
       ↓
Inspect /proc/PID
       ↓
Check application/service logs
       ↓
Take the appropriate action
```

The main goal of process management is not simply to know commands such as `kill` or `ps`.

The real skill is being able to **identify what is running, understand why it is running, determine whether its behavior is expected, and safely manage it**.
