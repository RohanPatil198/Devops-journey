# Linux Overview

## Objective

This section covers the fundamentals of Linux, including its architecture, kernel, distributions, shell, filesystem, users, processes, and why Linux is widely used in DevOps and cloud environments.

The goal is to build a strong Linux foundation before moving into administration, networking, automation, containers, and other DevOps technologies.

---

## 1. What is Linux?

Linux is an **open-source operating system kernel**. The kernel is the core component of an operating system that manages communication between applications and the underlying hardware.

Linux provides the foundation for many operating systems called **Linux distributions** or **Linux distros**.

Examples include:

* Ubuntu
* Debian
* Rocky Linux
* Red Hat Enterprise Linux (RHEL)
* Fedora
* Amazon Linux

### Linux vs Linux Distribution

**Linux** → Kernel

**Ubuntu / Debian / Rocky Linux** → Complete operating system distributions that include the Linux kernel along with system utilities, libraries, package managers, and other software.

---

## 2. Why Linux is Important in DevOps

Linux is widely used in DevOps and cloud environments because it is stable, flexible, scriptable, lightweight, and well suited for server environments.

Linux is commonly used for:

* Cloud servers
* Web servers
* Application servers
* Databases
* Docker containers
* Kubernetes nodes
* CI/CD runners
* Monitoring systems
* Infrastructure automation

A DevOps engineer should be comfortable managing Linux systems from the command line.

---

## 3. Linux Architecture

A simplified Linux architecture looks like this:

```text
+-----------------------------+
|        Applications         |
+-----------------------------+
|       Shell / Utilities     |
+-----------------------------+
|        Linux Kernel         |
+-----------------------------+
|          Hardware           |
+-----------------------------+
```

### Hardware

The physical or virtual resources of the system:

* CPU
* RAM
* Disk
* Network interfaces
* Other devices

### Kernel

The kernel is responsible for managing system resources and providing services to applications.

It handles areas such as:

* Process management
* Memory management
* Device management
* Filesystem management
* Networking
* Security

### Shell

The shell provides an interface through which users interact with the operating system.

Examples:

* Bash
* Zsh
* Fish

For this journey, Bash will be the primary shell.

### Applications and Utilities

These are the programs users interact with, such as:

```text
ls
cp
mv
grep
ssh
systemctl
curl
```

---

## 4. Kernel vs Shell

These two concepts should not be confused.

### Kernel

The kernel directly manages system resources and hardware.

```text
Application
     ↓
   Shell
     ↓
  Kernel
     ↓
 Hardware
```

### Shell

The shell interprets commands entered by the user and interacts with the operating system.

For example:

```bash
ls
```

The shell interprets the command and the operating system accesses the filesystem to provide the result.

---

## 5. Linux Filesystem

Linux uses a **single hierarchical filesystem** that begins at:

```text
/
```

This is called the **root directory**.

Unlike Windows, where filesystems are commonly organized around drive letters such as `C:` and `D:`, Linux uses one filesystem hierarchy.

Important directories include:

| Directory | Purpose                              |
| --------- | ------------------------------------ |
| `/`       | Root of the filesystem               |
| `/home`   | Home directories of normal users     |
| `/root`   | Home directory of the root user      |
| `/etc`    | System and application configuration |
| `/var`    | Variable data such as logs           |
| `/tmp`    | Temporary files                      |
| `/usr`    | User-space programs and libraries    |
| `/opt`    | Optional or third-party software     |
| `/dev`    | Device files                         |
| `/proc`   | Process and kernel information       |
| `/boot`   | Boot-related files                   |

Understanding the filesystem is important for Linux administration and troubleshooting.

---

## 6. Users and Root

Linux is a **multi-user operating system**.

Different users can have different:

* Permissions
* Files
* Groups
* Access levels

The `root` user is the administrative user with extensive privileges.

Normal users can perform administrative operations through `sudo` when they have the required permissions.

Example:

```bash
sudo apt update
```

The command runs with elevated privileges.

---

## 7. Processes

A **process** is a running instance of a program.

For example, when a program starts, Linux creates a process and assigns it a Process ID (PID).

Important concepts include:

* PID
* PPID
* Foreground process
* Background process
* Signals
* Process states

Processes are important in DevOps because troubleshooting often involves identifying applications that are consuming excessive CPU or memory or have stopped responding.

---

## 8. Services

A service is a program or process designed to run in the background and provide a specific function.

Examples:

* SSH server
* Web server
* Database server

On modern Ubuntu systems, `systemd` is commonly used to manage services.

Examples:

```bash
systemctl status ssh
systemctl start ssh
systemctl stop ssh
systemctl restart ssh
```

Service management and troubleshooting are important Linux administration skills.

---

## 9. Package Management

Linux distributions provide package managers for installing, updating, and removing software.

Ubuntu/Debian commonly use:

```bash
apt
```

Examples:

```bash
sudo apt update
sudo apt upgrade
sudo apt install git
```

Package management allows software to be installed and maintained in a controlled way.

---

## 10. Linux Networking

Linux provides powerful tools for network configuration and troubleshooting.

Common commands include:

```bash
ip
ss
ping
curl
dig
nslookup
traceroute
```

These tools can be used to investigate:

* IP addresses
* Network interfaces
* Open ports
* DNS resolution
* Connectivity
* Network services

Networking knowledge is especially important for cloud and DevOps troubleshooting.

---

## 11. Linux and DevOps

Linux forms the foundation for many DevOps technologies.

```text
              Linux
                |
       +--------+--------+
       |        |        |
      Git     Docker    SSH
       |        |        |
       +--------+--------+
                |
              CI/CD
                |
       +--------+--------+
       |                 |
   Terraform         Kubernetes
       |                 |
       +--------+--------+
                |
              Cloud
```

Understanding Linux makes it easier to understand what is happening underneath these technologies.

For example:

* Docker containers run on a Linux kernel.
* Kubernetes nodes commonly use Linux.
* CI/CD runners frequently run Linux.
* Terraform provisions infrastructure where Linux systems are commonly deployed.
* Cloud virtual machines are frequently Linux-based.

---

## 12. Hands-on Environment

For this learning journey, Linux practice is performed using:

```text
Host OS       : Windows
Virtualization: VMware
Guest OS      : Ubuntu Server 24.04 LTS
Shell         : Bash
```

The Linux environment will be used for practical labs, troubleshooting exercises, scripting, system administration, and DevOps tooling.

---

## 13. Learning Approach

This Linux journey focuses on practical understanding rather than memorizing commands.

For each topic, the approach is:

```text
Theory
   ↓
Understand the Concept
   ↓
Hands-on Lab
   ↓
Troubleshooting
   ↓
Document the Result
   ↓
Commit to Git
   ↓
Push to GitHub
```

The objective is to build evidence of practical knowledge through continuous hands-on work.

---

## 14. Topics Covered

The Linux learning path will cover:

* Linux fundamentals
* Filesystem
* Users and groups
* File permissions and ownership
* Process management
* Services and systemd
* Package management
* Networking
* Storage
* Logs
* Bash scripting
* SSH
* Cron and task scheduling
* Environment variables
* Linux security
* Troubleshooting
* Linux concepts used in DevOps

---

## 15. Key Takeaways

* Linux is an open-source kernel used as the foundation of many operating systems.
* Ubuntu is a Linux distribution, not the Linux kernel itself.
* The Linux filesystem starts from `/`.
* Linux supports multiple users with different permissions.
* Processes represent running programs.
* `systemd` manages many system services.
* Package managers simplify software installation and maintenance.
* Linux provides powerful command-line tools for administration and troubleshooting.
* Strong Linux knowledge is an important foundation for DevOps.

---

## Status

**Learning Status:** In Progress

**Environment:** Ubuntu Server 24.04 LTS on VMware

**Focus:** Theory + Hands-on + Troubleshooting + Documentation
