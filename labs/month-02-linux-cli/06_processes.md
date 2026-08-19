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

## 11. Finding a Process with `pgrep`

I used:
```Bash
pgrep sleep
```
Output included:
```
15809
```
Then:
```Bash
pgrep -a sleep
```
gave:
```
15809 sleep 300
```
Later I had several:

15920 sleep 300
16053 sleep 300
16093 sleep 300

This was easier to read than searching through the enormous output of `ps aux`.

## 12. `ps -p` — Inspect One PID

When I had:
```
PID = 33109
```
I used:
```Bash
ps -p 33109
```
Output:
```
    PID TTY          TIME CMD
  33109 pts/0    00:00:00 sleep
```
This was a useful workflow:
```text
Find process
     ↓
Get PID
     ↓
Inspect PID
     ↓
Decide whether action is needed
```
This is much closer to how I want to think about process analysis.

## 13. `pstree` — See the Family Tree

I used:
```Bash
pstree -p
```
The output began with:
```
systemd(1)
```
and showed many branches underneath it.

For example, I could see relationships such as:
```text
systemd(1)
 ├── NetworkManager(992)
 ├── ModemManager(1021)
 ├── lightdm(1132)
 │    └── ...
 ├── systemd(1363)
 │    └── gnome-terminal(10449)
 │         └── bash(10460)
 │              └── pstree(...)
```
This made PPID much easier to understand visually.

A process tree is basically a family tree for running programs.

## 14. A Useful Difference

I learned:
```
ps
```
is excellent for a snapshot.
```
pstree
```
is excellent for relationships.
```
top
```
is excellent for watching activity.
```
pgrep
```
is excellent for finding a process.
```
kill
```
is used to send a signal to a process (to terminate).

Together, these commands form a small process-investigation toolkit.

## 15. `top` — Watching Processes Live

I ran:
```Bash
top
```
My actual output showed:
```
top - 19:35:14 up  5:56,  1 user,  load average: 0.46, 0.66, 0.68


Tasks: 267 total,   1 running, 266 sleeping,   0 stopped,   0 zombie


%Cpu(s): 28.9 us,  5.8 sy,  0.0 ni, 63.6 id, ...


MiB Mem : 7823.7 total, 739.3 free, 4284.3 used, 3707.4 buff/cache


MiB Swap: 2048.0 total, 2047.9 free, 0.1 used.
```
This was different from `ps`.

Instead of giving me only a snapshot, `top` continuously updated the information.

## 16. What I Learned from `top`

At that moment my machine showed:
```
267 total tasks
1 running
266 sleeping
0 stopped
0 zombie
```
I also saw CPU and memory information.

Some processes were using considerably more CPU than others.

For example:
```
PID    USER     %CPU   %MEM
2149   aminul   49.0    6.7
2419   aminul   46.4    7.8
1739   aminul   21.4    2.7
```
I learned an important security lesson:

> High CPU usage is an observation, not automatically evidence of malware.

A legitimate browser, compiler, video application, or other program can use a lot of CPU.

But an unexpected process consuming enormous resources is something worth investigating.

## 17. Creating High CPU Activity Safely

I deliberately created a CPU-heavy test process:
```Bash
yes > /dev/null &
```
My shell reported:
```
[1] 18456
```
Then I ran `top`.

The process appeared as:
```
18456 aminul ... R 100.0 0.0 ...
```
The CPU usage was:
```
100.0%
```
This was an excellent demonstration.

I had just created something that looked suspicious from a resource-monitoring perspective.

But I knew exactly why it existed.

That is an important security principle:

> Context matters.

## 18. Stopping the High-CPU Process

I stopped my test process with:
```Bash
kill 18456
```
Then I checked:
```Bash
ps -p 18456
```
The shell showed that the background job had been terminated:
```
[1]+  Terminated              yes > /dev/null
```
I also checked:
```Bash
pgrep -a yes
```
and received no process entry.

The process was gone.

## 19. What `kill` Actually Means

Before this lesson, I thought:
```
kill = destroy process
```
Now I understand that `kill` actually sends a signal to a process.

For example:
```
kill 33109
```
normally sends `SIGTERM`.

`SIGTERM` asks the process to terminate.

If necessary, another signal such as:
```Bash
kill -9 PID
```
sends `SIGKILL`.

`SIGKILL` is much more forceful and does not give the process the opportunity to perform normal cleanup. The Linux `kill(1)` documentation recommends using the normal TERM signal before KILL where possible.

So my preferred mental model is:
```text
kill
  ↓
send a signal
  ↓
process receives it
  ↓
process may terminate
```

## 20. My Safe `kill` Experiment

I created:
```Bash
sleep 300 &
```
Output:
```
[1] 33109
```
I confirmed it:
```Bash
pgrep -a sleep
```
Output:
```
33109 sleep 300
```
I inspected it:
```Bash
ps -p 33109
```
Output:
```
    PID TTY          TIME CMD
  33109 pts/0    00:00:00 sleep
```
I then inspected its `/proc` information:
```Bash
cat /proc/33109/status
```
Among the information shown was:
```
Name:   sleep
State:  S (sleeping)
Pid:    33109
PPid:   10460
Uid:    1000  1000  1000  1000
Gid:    1000  1000  1000  1000
```
This connected several concepts together:
```
Process
  ↓
PID
  ↓
PPID
  ↓
Owner
  ↓
State
  ↓
/proc information
```
The Linux `/proc/<pid>/status` interface exposes human-readable process status information, including PID, PPID, state, UID, GID and other details.

Finally:
```Bash
kill 33109
```
and:
```Bash
pgrep -a sleep
```
showed that the process had terminated.

