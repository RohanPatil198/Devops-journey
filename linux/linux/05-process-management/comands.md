# Linux Process Management — Commands

This document covers the main commands used to inspect, monitor, control, and troubleshoot Linux processes.

---

## 1. `ps`

`ps` displays information about currently running processes.

### Basic usage

```bash
ps
```

This normally shows processes associated with the current terminal/session.

### Common form

```bash
ps aux
```

Important fields include:

```text
USER
PID
%CPU
%MEM
VSZ
RSS
STAT
START
TIME
COMMAND
```

### Important options

```bash
ps a
```

Shows processes associated with terminals, including processes from other users depending on the system's `ps` implementation.

```bash
ps x
```

Includes processes without an associated terminal.

```bash
ps u
```

Displays user-oriented information.

A very common combination is:

```bash
ps aux
```

### `ps -ef`

Another extremely common command is:

```bash
ps -ef
```

It provides a full-format process listing.

Important fields include:

```text
UID
PID
PPID
C
STIME
TTY
TIME
CMD
```

The `PPID` field is particularly useful because it shows the parent process.

### Find a specific PID

```bash
ps -p 1234
```

For more detail:

```bash
ps -p 1234 -f
```

### View a process tree

```bash
ps -ef --forest
```

This makes parent-child relationships easier to understand.

### Specific user

```bash
ps -u rohan
```

Replace `rohan` with the actual username.

### Why it is useful

Use `ps` when you need a snapshot of processes and detailed information about a particular process.

---

# 2. `top`

`top` provides a continuously updating view of running processes.

```bash
top
```

It displays information about:

- CPU usage
- Memory usage
- Load
- Running processes
- Process states
- PIDs
- Users
- Process runtime

While `top` is running, common interactive keys include:

```text
q   quit
P   sort by CPU usage
M   sort by memory usage
k   send a signal to a process
r   change process priority
```

Use `q` to exit.

### Why it is useful

When someone says:

> "The server is slow."

`top` is often one of the first commands to run.

---

# 3. `htop`

If installed:

```bash
htop
```

`htop` is an interactive process viewer and is generally easier to navigate than `top`.

It may not be installed by default.

On Ubuntu it can be installed with:

```bash
sudo apt install htop
```

This is optional for the lab.

---

# 4. `pgrep`

`pgrep` searches for processes based on their names or other attributes.

For example:

```bash
pgrep bash
```

returns matching PIDs.

To display PID and command name:

```bash
pgrep -a bash
```

### Important options

```bash
pgrep -a name
```

Displays the process ID and command line.

```bash
pgrep -u username
```

Finds processes belonging to a particular user.

```bash
pgrep -f pattern
```

Searches the full command line instead of only the process name.

Example:

```bash
pgrep -af python
```

This can help locate Python processes.

---

# 5. `pkill`

`pkill` sends a signal to processes matching a name or other criteria.

For example:

```bash
pkill sleep
```

This can terminate matching `sleep` processes.

You can explicitly use SIGTERM:

```bash
pkill -TERM sleep
```

Or:

```bash
pkill -15 sleep
```

Be careful with `pkill` because it can affect multiple processes.

For production systems, verify what will match before using it.

---

# 6. `kill`

Despite its name, `kill` does not always mean "forcefully terminate."

Its actual purpose is to **send a signal to a process**.

Basic usage:

```bash
kill PID
```

By default, this normally sends `SIGTERM`.

For example:

```bash
kill 2500
```

requests process `2500` to terminate.

### Specify signal

```bash
kill -15 2500
```

SIGTERM.

```bash
kill -9 2500
```

SIGKILL.

```bash
kill -2 2500
```

SIGINT.

### List available signals

```bash
kill -l
```

### Important signals

| Signal | Number | Purpose |
|---|---:|---|
| SIGHUP | 1 | Hangup/reload behavior depending on application |
| SIGINT | 2 | Interrupt |
| SIGTERM | 15 | Graceful termination request |
| SIGKILL | 9 | Forceful termination |
| SIGSTOP | 19 on common Linux architectures | Stop process |
| SIGCONT | 18 on common Linux architectures | Continue stopped process |

Do not rely on signal numbers for portability when a signal name can be used.

For example:

```bash
kill -TERM PID
```

is clearer than:

```bash
kill -15 PID
```

---

# 7. `killall`

`killall` can send signals to processes based on their command name.

Example:

```bash
killall sleep
```

This can affect multiple matching processes.

Because of that, use it carefully.

For controlled testing, it is safer to create your own test processes and verify the process list before using name-based termination.

---

# 8. `jobs`

`jobs` displays jobs managed by the current shell.

Example:

```bash
jobs
```

Possible output:

```text
[1]+  Running    sleep 300 &
```

The `[1]` is the shell job number.

It is not the same as the PID.

---

# 9. Background Processes — `&`

You can start a command in the background:

```bash
sleep 300 &
```

The shell immediately returns to the prompt.

The shell may display something like:

```text
[1] 2500
```

Here:

```text
1     → job number
2500  → PID
```

---

# 10. `Ctrl + Z`

Pressing:

```text
Ctrl + Z
```

usually sends `SIGTSTP` to the foreground job and suspends it.

For example:

```bash
sleep 300
```

Then press:

```text
Ctrl + Z
```

The job becomes stopped.

Check:

```bash
jobs
```

---

# 11. `bg`

`bg` continues a stopped job in the background.

```bash
bg
```

For a specific job:

```bash
bg %1
```

Here `%1` refers to shell job number 1.

---

# 12. `fg`

`fg` brings a background/stopped job to the foreground.

```bash
fg
```

Or:

```bash
fg %1
```

---

# 13. `nohup`

`nohup` allows a command to continue running after the terminal session closes, depending on how the process is otherwise managed.

Example:

```bash
nohup command &
```

By default, output may be redirected to:

```text
nohup.out
```

This is useful for simple long-running commands, although production services should normally be managed using a proper service manager such as systemd.

---

# 14. `pstree`

`pstree` displays processes in a tree structure.

```bash
pstree
```

For PIDs:

```bash
pstree -p
```

The `-p` option includes process IDs.

You can also inspect a particular process:

```bash
pstree -p 1234
```

This is useful for understanding parent-child relationships.

---

# 15. `pidof`

`pidof` finds the PID of a running program.

Example:

```bash
pidof sshd
```

If the program is running, it may return one or more PIDs.

This is useful in scripts and administrative tasks.

---

# 16. `uptime`

```bash
uptime
```

shows information including:

- Current system time
- How long the system has been running
- Number of logged-in users
- Load averages

Example conceptually:

```text
load average: 0.10, 0.15, 0.12
```

These normally represent load averages over:

```text
1 minute
5 minutes
15 minutes
```

Load average is not simply CPU percentage. It represents work waiting for CPU and, on Linux, certain uninterruptible tasks such as some I/O activity.

---

# 17. `free`

`free` displays memory and swap information.

Use:

```bash
free -h
```

The `-h` option means human-readable units.

You may see:

```text
total
used
free
shared
buff/cache
available
```

The `available` value is often more useful than simply looking at `free`, because Linux intentionally uses unused RAM for caching.

---

# 18. `vmstat`

`vmstat` provides information about system performance.

Basic usage:

```bash
vmstat
```

You can request repeated measurements:

```bash
vmstat 2
```

This updates approximately every two seconds.

It can provide information about:

- Processes
- Memory
- Paging
- Block I/O
- Interrupts
- CPU

Use `Ctrl+C` to stop repeated output.

---

# 19. `nice`

`nice` starts a process with a specified nice value.

Example:

```bash
nice -n 10 sleep 300
```

The command starts with a nice value of 10.

Check it using:

```bash
ps -o pid,ni,comm -p PID
```

The `NI` column represents the nice value.

A higher nice value generally means the process is given lower scheduling priority.

---

# 20. `renice`

`renice` changes the nice value of an existing process.

Example:

```bash
renice 10 -p 2500
```

This changes the nice value of PID `2500`.

Check the result:

```bash
ps -o pid,ni,comm -p 2500
```

Changing a process to a higher priority generally requires elevated privileges.

---

# 21. `lsof`

`lsof` means **List Open Files**.

Linux represents many resources through file descriptors, so `lsof` is useful for identifying what a process has open.

For a process:

```bash
lsof -p 2500
```

For network-related information:

```bash
sudo lsof -i
```

For a particular port:

```bash
sudo lsof -i :80
```

This can help answer:

> Which process is using this port?

---

# 22. `/proc/<PID>/status`

Linux exposes process information under `/proc`.

For PID `2500`:

```bash
cat /proc/2500/status
```

Useful information includes:

```text
Name
State
Pid
PPid
Uid
Gid
Threads
VmSize
VmRSS
```

This is a useful low-level source of process information.

---

# 23. `/proc/<PID>/cmdline`

```bash
cat /proc/2500/cmdline
```

This shows the command-line arguments used to start the process.

Because arguments are internally separated by null characters, output may appear differently from normal command-line output.

A useful alternative is:

```bash
tr '\0' ' ' < /proc/2500/cmdline
```

---

# 24. `/proc/<PID>/exe`

```bash
ls -l /proc/2500/exe
```

This can show the executable associated with the process.

It is useful when you know the PID but want to understand which executable is running.

---

# 25. `/proc/<PID>/fd`

```bash
ls -l /proc/2500/fd/
```

This shows the process's open file descriptors.

You may see links referring to:

- Files
- Pipes
- Sockets
- Devices

This is useful when investigating resource usage.

---

# 26. `watch`

`watch` repeatedly executes a command.

Example:

```bash
watch ps -ef
```

It is useful for observing changes over time.

Another example:

```bash
watch free -h
```

Press:

```text
Ctrl+C
```

to stop it.

---

# 27. `time`

`time` measures how long a command takes to execute.

Example:

```bash
time sleep 2
```

The output can include:

```text
real
user
sys
```

Conceptually:

- `real` — total elapsed time
- `user` — CPU time spent executing user-space code
- `sys` — CPU time spent executing kernel/system calls on behalf of the process

---

# 28. Process State Codes

When using:

```bash
ps
```

you may see a `STAT` field.

Common states include:

```text
R  Running
S  Interruptible sleep
D  Uninterruptible sleep
T  Stopped
Z  Zombie
```

Additional characters may provide information about process properties.

Do not assume that every process in `S` state is problematic. Sleeping is normal for many applications.

---

# 29. Useful Investigation Commands

### Find a process

```bash
pgrep -a nginx
```

### Get detailed information

```bash
ps -p PID -f
```

### Find parent

```bash
ps -o pid,ppid,cmd -p PID
```

### See process tree

```bash
pstree -p
```

### Monitor resources

```bash
top
```

### Check memory

```bash
free -h
```

### Check load

```bash
uptime
```

### Inspect process information

```bash
cat /proc/PID/status
```

### Check open files

```bash
lsof -p PID
```

---

# 30. Safe Process Termination Workflow

A useful operational workflow is:

```text
Identify process
      ↓
Verify PID and command
      ↓
Check owner
      ↓
Check parent/service
      ↓
Send SIGTERM
      ↓
Verify process stopped
      ↓
Use SIGKILL only if necessary
```

For example:

```bash
ps -p 2500 -f
```

Then:

```bash
kill 2500
```

Verify:

```bash
ps -p 2500
```

If it still exists and there is a valid reason for forceful termination:

```bash
kill -9 2500
```

---

# Quick Reference

| Task | Command |
|---|---|
| List current processes | `ps` |
| List most processes | `ps aux` |
| Full process list | `ps -ef` |
| Inspect PID | `ps -p PID -f` |
| Monitor processes | `top` |
| Find process by name | `pgrep` |
| Send signal by name | `pkill` |
| Send signal to PID | `kill` |
| View process tree | `pstree -p` |
| Find PID | `pidof` |
| View shell jobs | `jobs` |
| Continue background job | `bg` |
| Bring job foreground | `fg` |
| Start in background | `command &` |
| Start with lower priority | `nice` |
| Change priority | `renice` |
| Check system load | `uptime` |
| Check memory | `free -h` |
| Check performance | `vmstat` |
| Open files | `lsof` |
| Inspect process internals | `/proc/PID/` |
| Repeat command | `watch` |

The most important commands to become comfortable with are:

```bash
ps aux
ps -ef
ps -p PID -f
top
pgrep
kill
jobs
bg
fg
pstree
nice
renice
free -h
uptime
lsof
```
