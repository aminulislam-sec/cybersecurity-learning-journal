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

## 4. PPID — Parent Process ID

PID tells me:

> "Who am I?"

PPID tells me:

> "Who started me?"

I used:
```Bash
ps -u "$USER" -f
```
The output contained:
```
UID       PID    PPID   C STIME TTY   TIME CMD
aminul    1363      1   0 13:40 ?     00:00:00 /usr/lib/systemd/systemd --user
aminul    1364   1363   0 13:40 ?     00:00:00 (sd-pam)
...
aminul   10449   1363   0 16:28 ?     00:00:13 /usr/libexec/gnome-terminal...
aminul   10460  10449   0 16:28 pts/0 00:00:00 bash
```
This allowed me to see relationships such as:
```text
systemd
   │
   └── gnome-terminal
          │
          └── bash
```
My Bash process had:
```
PID  = 10460
PPID = 10449
```
So my Bash shell was started by the terminal process.

This is much more useful than simply memorizing the definition of PPID.

I actually saw the relationship.

## 5. `ps` — A Snapshot of Processes

The simplest command I used was:
```Bash
ps
```
It showed processes associated with my current terminal session.

For a much larger view, I used:
```Bash
ps aux
```
This produced a large list of running processes.

The important columns were:
```
USER
PID
%CPU
%MEM
VSZ
RSS
TTY
STAT
START
TIME
COMMAND
```
I learned to read them as questions.

### USER

Who owns the process?

For example:
```
root
aminul
avahi
systemd+
message+
```
### PID

What is the process ID?

### %CPU

How much CPU the process is currently using.

### %MEM

How much memory it is using.

### STAT

What state the process is in.

### COMMAND

What program is actually running.

## 6. Why the USER Column Matters

One of the most important security observations from `ps aux` was that processes run under different accounts.

For example, my output contained processes owned by:
```
root
aminul
avahi
message+
systemd+
polkitd
cups-br+
```
A process running as root deserves particular attention because root has extensive privileges.

That does NOT mean:

> "Every root process is malicious."

Linux itself needs many root-owned processes.

The correct security question is:

> "Is this process expected, and why is it running with this level of privilege?"

This is an important distinction.

Security analysis is not about panicking whenever I see root.

It is about understanding what I am seeing.

## 7. My `ps aux` Observation

My actual `ps aux` output showed many kernel and system processes.

For example:
```
root 1 0.0 0.1 ... /sbin/init
root 2 0.0 0.0 ... [kthreadd]
root 14 0.0 0.0 ... [ksoftirqd/0]
root 44 0.0 0.0 ... [kauditd]
```
It also showed normal desktop processes belonging to me:
```
aminul 1739 ... cinnamon --replace
aminul 2149 ... /usr/lib/firefox/firefox
aminul 10460 ... bash
```
The process list looked enormous at first.

That was actually useful.

I realized that my Linux desktop is not "one program".

It is a large collection of programs working together.

## 8. `ps -u "$USER"` — My Processes

I used:
```Bash
ps -u "$USER"
```
This displayed processes associated with my user account.

I saw processes such as:
```
systemd
pipewire
wireplumber
dbus-daemon
cinnamon
nemo
firefox-bin
gnome-terminal
bash
```
This helped me understand that logging into a graphical Linux desktop starts many background processes.

It also showed me something important:

> A process does not have to correspond to a window I can see.

Many processes work quietly in the background.

## 9. `ps -u "$USER" -f` — Add Parent Information

I then used:
```Bash
ps -u "$USER" -f
```
The `-f` option gave me more detailed information.

The most important new column for me was:
```
PPID
```
For example:
```
aminul 10460 10449 ... bash
```
This means:
```
PID  = 10460
PPID = 10449
```
So process 10449 is the parent of process 10460.

This made the parent-child concept much easier to understand.

## 10. A Safe Process for Practice

Instead of experimenting with important system processes, I created my own harmless process:
```Bash
sleep 300 &
```
The `&` tells Bash to run the command in the background.

My output was:
```
[1] 15809
```
The number:
```
15809
```
was the PID.

I later repeated the experiment and created:
```
[2] 15920
[3] 16053
[4] 16093
```
These were all instances of:
```
sleep 300
```
This was a safe way to learn process management without touching an important system process.

