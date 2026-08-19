# Lesson 6 — Linux Processes: ps, top, pstree, pgrep and kill

> Month 2 — Linux CLI Mastery  
> Week 6 — Processes + Services  
> Part of my 24-month journey to become a Cybersecurity Documentation & Automation Engineer.

---

## Objective

In this lesson, I learned how to:

- understand what a Linux process is
- identify running processes
- understand PID and PPID
- see which user owns a process
- inspect my own processes
- understand parent-child process relationships
- monitor processes in real time with `top`
- search for processes with `pgrep`
- inspect process information through `/proc`
- safely terminate a process with `kill`
- connect process analysis with cybersecurity

The main idea I wanted to understand was simple:

> A process is a program that is currently running.

A process has an identity, an owner, a parent, a state, and resource usage.

From a cybersecurity perspective, learning to read processes means learning to ask:

> "What is running on this machine, who started it, and should it be running?"

---

## 1. My Environment

I performed this lesson on my primary Linux learning machine.

```text
Operating System: Linux Mint 22.3 (Zena)
Kernel: 6.17.0-40-generic
Architecture: x86_64
Hostname: aminulislam
User: aminul
Home directory: /home/aminul

I verified the system with:
```Bash
whoami
hostname
uname -a
pwd
```
My actual output included:
```
aminul
aminulislam
Linux aminulislam 6.17.0-40-generic #40~24.04.1-Ubuntu SMP PREEMPT_DYNAMIC ...
/home/aminul
```
This immediately reminded me that the terminal is not just a place for entering commands.

It is also a place for asking the operating system questions.

## 2. What Is a Process?

A program is a set of instructions.

A process is that program while it is actually running.

For example:
```text
Firefox installed on disk
        ↓
Firefox launched
        ↓
Firefox becomes a running process
```
The same idea applies to:

- Bash
- Firefox
- Cinnamon
- NetworkManager
- PipeWire
- systemd
- terminal applications
- background services

A computer can have hundreds of processes running at the same time.

My own system demonstrated this clearly.

## 3. PID — Process ID

Every running process has a Process ID, or PID.

For example, my Bash shell had:

`PID = 10460`

I confirmed this with:
```Bash
echo $$
```
Output:
```
10460
```
The special Bash variable:
```Bash
$$
```
represents the PID of the current shell.

I could also see the same process using:
```Bash
ps
```
Output:
```
    PID TTY          TIME CMD
  10460 pts/0    00:00:00 bash
  10703 pts/0    00:00:00 ps
```
This was one of my first important observations.

When I ran `ps`, the command itself appeared as a process.

In other words:
```text
Bash
  └── ps
```
The command I used to look at processes became a process itself.

