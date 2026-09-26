# Linux Process Management

Process management is one of the most important parts of Linux administration. Every time a program runs in Linux, it normally becomes a **process**. A web server, database, SSH session, shell, Jenkins agent, Docker-related process, monitoring agent, or even a simple command such as `sleep` can be represented as a process while it is running.

Understanding processes is important because a system administrator or DevOps engineer regularly needs to answer questions such as:

- Which processes are currently running?
- Which process is consuming too much CPU?
- Which process is using too much memory?
- Who owns a particular process?
- What started this process?
- What is its parent process?
- Is the process running, sleeping, stopped, or stuck?
- How can a process be stopped safely?
- Why is a process not responding?
- Which process is using a particular resource?

This topic focuses on understanding processes first and then using Linux tools to investigate and manage them.

---

## 1. What Is a Process?

A **process is a running instance of a program**.

A program is a set of instructions stored on disk. It becomes a process when the operating system loads it into memory and starts executing it.

For example:

```bash
sleep 300
```

The `sleep` command is a program. When you execute it, Linux creates a process that remains active for 300 seconds.

The important difference is:

```text
Program
  ↓
Instructions stored on disk

Process
  ↓
Running instance of that program
```

The same program can have multiple processes.

For example, if several users run the same application, Linux may have multiple processes running from the same executable.

A process requires system resources such as:

- CPU time
- Memory
- File descriptors
- Network connections
- Access permissions
- Environment variables
- Process IDs

Linux's kernel manages these resources.

---

# 2. Process ID — PID

Every running process has a unique **Process ID**, commonly called a **PID**.

For example:

```text
PID     COMMAND
1250    sshd
1832    bash
2410    python3
```

The PID allows Linux and administrators to identify a particular process.

You will frequently see PIDs when troubleshooting.

For example:

```bash
ps -ef
```

may show:

```text
root      1250     1  ... /usr/sbin/sshd
rohan     1832  1250  ... bash
rohan     2410  1832  ... python3 app.py
```

Here:

- `1250` is the PID of `sshd`
- `1832` is the PID of `bash`
- `2410` is the PID of the Python application

A PID is important because commands such as `kill`, `renice`, and `lsof` can operate on a specific process using its PID.

---

# 3. Parent Process and PPID

Processes can create other processes.

The process that creates another process is called its **parent process**.

The newly created process is called the **child process**.

Linux therefore maintains a relationship such as:

```text
Parent
  |
  ├── Child
  |
  └── Child
```

The **PPID** means **Parent Process ID**.

For example:

```text
PID    PPID    COMMAND
2000   1500    bash
2200   2000    python3
```

Here, process `2000` is the parent of process `2200`.

This relationship becomes very useful during troubleshooting because you can determine what started a process.

---

# 4. PID 1 and systemd

On a normal modern Ubuntu system, the first userspace process is usually:

```text
systemd
```

and its PID is:

```text
1
```

You can check it with:

```bash
ps -p 1 -f
```

PID 1 has an important role in the system.

It participates in:

- Starting system services
- Managing services
- Handling parts of the process lifecycle
- Adopting orphaned processes

This is also why process management and service management are closely related.

For example:

```text
Linux boot
    ↓
Kernel
    ↓
systemd (PID 1)
    ↓
Services
    ↓
Applications
```

Service management with `systemctl` will be covered more deeply in the **systemd/services** topic.

---

# 5. Process States

A process is not always actively executing on the CPU.

Linux assigns a state to a process depending on what it is currently doing.

Common process states include:

### Running — `R`

The process is currently running or ready to run.

It may be actively executing on a CPU or waiting in the scheduler's run queue.

---

### Sleeping — `S`

The process is waiting for an event.

For example, a process waiting for:

- keyboard input
- network data
- a timer
- a file operation

may be sleeping.

Sleeping does not automatically mean something is wrong.

---

### Uninterruptible Sleep — `D`

A process in `D` state is usually waiting for a kernel operation to complete, commonly involving I/O.

For example:

```text
D
```

may indicate that the process is waiting for disk or another kernel-level operation.

A process stuck in `D` state can require investigation because normal signals may not immediately terminate it.

---

### Stopped — `T`

A process can be stopped by a signal or by job-control mechanisms.

For example:

```text
Ctrl + Z
```

can stop a foreground process.

---

### Zombie — `Z`

A zombie is a process that has finished execution but whose parent has not yet collected its exit status.

A zombie is therefore not actively running.

Conceptually:

```text
Child finishes
      ↓
Child becomes zombie
      ↓
Parent collects exit status
      ↓
Zombie entry disappears
```

A small number of temporary zombies may not be a problem, but a continuously increasing number can indicate a problem with the parent application.

---

# 6. Foreground and Background Processes

When you run a command normally:

```bash
sleep 100
```

the shell waits for it to finish.

This is a **foreground process** from the shell's perspective.

You cannot normally enter another command in that same shell until `sleep` finishes.

You can instead run it in the background:

```bash
sleep 100 &
```

The `&` tells the shell to start the command as a background job.

You immediately get the shell prompt back.

For example:

```text
sleep 100 &
[1] 2450
```

Here:

```text
[1]
```

is the shell's job number.

```text
2450
```

is the process ID.

These are different concepts.

```text
Job number → managed by the shell
PID         → managed by the operating system
```

---

# 7. Shell Jobs

Linux shells provide job-control commands such as:

```bash
jobs
bg
fg
```

For example:

```bash
sleep 300
```

Press:

```text
Ctrl + Z
```

The shell can suspend the command.

Then:

```bash
bg
```

continues it in the background.

You can bring it back to the foreground with:

```bash
fg
```

This is especially useful when working interactively in a terminal.

---

# 8. Signals

Linux processes communicate with the kernel and other processes using **signals**.

A signal is a notification sent to a process telling it that something has happened or that it should perform a particular action.

Some important signals are:

### SIGTERM — 15

Requests that a process terminate.

```bash
kill -15 PID
```

This is normally the preferred way to stop a process because the application gets an opportunity to perform cleanup.

---

### SIGKILL — 9

Immediately terminates a process.

```bash
kill -9 PID
```

The process cannot catch or handle `SIGKILL`.

Because it is forceful, it should generally be used only when normal termination does not work.

---

### SIGINT — 2

Interrupts a process.

The common example is:

```text
Ctrl + C
```

which normally sends an interrupt signal to the foreground process.

---

### SIGHUP — 1

Historically associated with a terminal hangup.

It is also commonly used by applications to request that they reload configuration.

The actual behavior depends on the application.

---

# 9. Graceful Termination vs Forceful Termination

A common mistake is immediately using:

```bash
kill -9 PID
```

A better approach is usually:

```text
SIGTERM
   ↓
Give application time to clean up
   ↓
Check whether it stopped
   ↓
SIGKILL only if necessary
```

Why does graceful termination matter?

An application may need to:

- Close files
- Finish database operations
- Close network connections
- Write pending data
- Remove temporary resources
- Save application state

Forcefully terminating it can prevent those cleanup operations.

---

# 10. Viewing Processes

Linux provides several tools for viewing processes.

The most commonly used are:

```bash
ps
top
pgrep
pstree
```

`ps` gives a snapshot of processes.

`top` provides a continuously updating view.

`pgrep` helps find processes by name or other attributes.

`pstree` shows parent-child relationships.

---

# 11. `ps`

`ps` is one of the most important process-management commands.

For example:

```bash
ps
```

shows processes associated with the current shell.

A much more useful command is:

```bash
ps aux
```

This displays processes from across the system.

Another common form is:

```bash
ps -ef
```

Both are widely used, although their output format differs.

Important information includes:

- PID
- User
- CPU usage
- Memory usage
- Parent PID
- Start information
- Command

---

# 12. `top`

`top` provides a real-time view of system processes.

Run:

```bash
top
```

It continuously updates process information.

It is useful for finding:

- High CPU processes
- High memory processes
- System load
- Number of processes
- CPU utilization
- Memory utilization

This is one of the first commands you may use when someone reports:

> "The Linux server is very slow."

You can start by checking:

```bash
top
```

and then investigate the processes consuming resources.

---

# 13. CPU and Memory Usage

Processes consume system resources.

CPU usage represents how much processor time a process is using.

Memory usage represents how much RAM a process is using.

A process consuming excessive resources can cause:

- Slow applications
- High system load
- Memory pressure
- Application failures
- Service instability

Useful commands include:

```bash
top
free -h
ps aux
```

The important point is not simply finding a process with high usage. You also need to determine **why** it is consuming resources.

For example, high CPU could be caused by:

- An expected workload
- An infinite loop
- A badly behaving application
- Heavy data processing
- An attack or unexpected activity

---

# 14. Process Priority and Nice Value

Linux uses scheduling priorities when deciding which processes receive CPU time.

The **nice value** influences process scheduling priority.

The normal nice value is commonly:

```text
0
```

The general range is:

```text
-20 to 19
```

A lower nice value generally means higher scheduling priority.

A higher nice value generally means lower scheduling priority.

For example:

```bash
nice -n 10 command
```

starts a command with a lower CPU scheduling priority than the default.

You can modify the nice value of an existing process using:

```bash
renice
```

Changing process priority requires appropriate permissions, especially when increasing priority.

---

# 15. `/proc` and Processes

Linux exposes a large amount of process information through the virtual `/proc` filesystem.

For a process with PID `2500`, you can inspect:

```text
/proc/2500/
```

Some useful files and directories include:

```text
/proc/2500/status
/proc/2500/cmdline
/proc/2500/fd/
/proc/2500/exe
```

For example:

```bash
cat /proc/2500/status
```

can provide information about:

- Process state
- PID
- Parent PID
- User/group information
- Memory information
- Threads

This is useful when normal process commands do not provide enough detail.

---

# 16. Processes and File Descriptors

A running process can have open files, sockets, pipes, and other resources represented through file descriptors.

Linux represents these under:

```text
/proc/<PID>/fd/
```

For example:

```bash
ls -l /proc/2500/fd/
```

can help identify resources currently opened by a process.

The `lsof` command is another useful tool:

```bash
lsof -p 2500
```

It lists open files associated with the process.

This can be extremely useful during troubleshooting.

---

# 17. Process Ownership

Every process runs with a user identity.

For example:

```text
root
www-data
mysql
jenkins
rohan
```

The process owner's permissions influence what the process can access.

You can see the owner using:

```bash
ps aux
```

This is important from a security perspective.

A service should generally run with only the privileges it requires.

For example, a web application does not automatically need full root privileges.

This follows the principle of **least privilege**.

---

# 18. Process vs Service

A process and a service are related, but they are not exactly the same thing.

A **process** is a running instance of a program.

A **service** is typically a long-running application or background component managed by the operating system or a service manager.

For example:

```text
nginx service
      ↓
nginx processes
```

On Ubuntu, `systemd` commonly manages services.

You will study service management separately, including:

```bash
systemctl status
systemctl start
systemctl stop
systemctl restart
```

---

# 19. Why Process Management Matters in DevOps

Process management is directly relevant to DevOps operations.

Consider a server running:

```text
Nginx
Application
Database
Monitoring agent
SSH
Docker
CI/CD agent
```

If the server becomes slow, you need to investigate the running processes.

A typical investigation could be:

```text
Check system state
      ↓
top
      ↓
Identify high CPU/memory process
      ↓
Find PID
      ↓
Inspect process
      ↓
Find parent/service
      ↓
Check logs/configuration
      ↓
Take corrective action
```

This is much better than randomly killing processes.

---

# 20. Common Troubleshooting Scenarios

### Scenario 1 — High CPU

Start with:

```bash
top
```

Identify the process consuming CPU.

Then:

```bash
ps -p PID -f
```

to inspect it.

---

### Scenario 2 — Application is not responding

Find the application:

```bash
pgrep -a application_name
```

Then inspect its process information.

If appropriate, request graceful termination:

```bash
kill PID
```

and investigate why it became unresponsive.

---

### Scenario 3 — Process refuses to stop

First try:

```bash
kill PID
```

Wait and verify.

Only if necessary:

```bash
kill -9 PID
```

---

### Scenario 4 — Many zombie processes

Check:

```bash
ps aux
```

and look for `Z` process states.

Then investigate the parent process rather than trying to repeatedly kill the zombies themselves.

---

# 21. Common Mistakes

### Using `kill -9` for everything

This bypasses graceful application cleanup.

Use normal termination first.

### Killing the wrong PID

Always verify:

```bash
ps -p PID -f
```

before terminating an important process.

### Confusing job ID with PID

For:

```text
[1] 2450
```

`1` is the shell job number.

`2450` is the PID.

### Assuming high CPU always means a problem

A process performing legitimate heavy work can intentionally consume CPU.

Always understand what the process is doing.

### Running administrative commands unnecessarily

Process management should follow least privilege. Use `sudo` only when required.

---

# 22. Important Commands

The main commands covered in this topic are:

```text
ps
top
pgrep
pkill
pstree
pidof
kill
killall
jobs
bg
fg
nice
renice
uptime
free
vmstat
lsof
watch
```

Important process information can also be found under:

```text
/proc/<PID>/
```

---

# 23. Interview Concepts to Understand

You should be able to explain:

- What is a process?
- Program vs process
- What is a PID?
- What is a PPID?
- What is PID 1?
- What is systemd?
- What are process states?
- What is a zombie process?
- What is an orphan process?
- Foreground vs background process
- What does `&` do?
- What does `Ctrl+C` do?
- What does `Ctrl+Z` do?
- What is a signal?
- SIGTERM vs SIGKILL
- What is `ps`?
- `ps aux` vs `ps -ef`
- What is `top`?
- What is `pgrep`?
- What is `pkill`?
- What is nice value?
- What is `renice`?
- What is `/proc/<PID>`?
- How do you identify a high-CPU process?
- How do you safely terminate a process?
- What is the difference between a process and a service?

---

## Key Takeaway

Process management is not simply about starting and killing processes. The important skill is being able to **observe a process, understand its state and resource usage, identify its ownership and parent, and then take the appropriate action safely**.

That approach is useful for Linux administration, DevOps, cloud operations, and production troubleshooting.

**Status:** Theory covered — practical commands and lab follow.
