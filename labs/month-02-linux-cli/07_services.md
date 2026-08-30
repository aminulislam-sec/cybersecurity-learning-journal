# Lesson 07 — Linux Services and `systemd`

## Objective

In this lesson, I learned what Linux services are and how Linux uses **systemd** to manage them.

My main goals were to learn how to:

- identify running services
- check whether a service is active
- check whether a service is enabled at boot
- inspect service details
- find failed services
- inspect service configuration
- examine service dependencies
- connect services to their underlying processes
- inspect service logs
- create and manage a small user-level systemd service
- look at services from a cybersecurity perspective

The most important idea I wanted to understand was:

> A service is not just a name ending in `.service`. It is a managed unit that can start a program, monitor it, restart it, record its activity, and interact with other parts of the operating system.

## Background

Before this lesson, I had already learned about **processes**.

A process is a program that is currently running.

This lesson added another layer:

**Process → Service → systemd**

A process is the actual running program.

A service is a managed job that systemd knows how to start, stop, monitor, and control.

`systemd` is the service manager used by my Linux Mint system.

My system reported:
```Bash
systemd 255 (255.4-1ubuntu8.17)
```
So I was not simply learning commands in isolation. I was investigating the service-management system actually running on my computer.

## Environment

My Linux Mint system reported:
```
systemd 255 (255.4-1ubuntu8.17)
```
The system uses the unified hierarchy:
```
default-hierarchy=unified
```
The commands in this lesson were run from my normal user account:
```
aminul@aminulislam:~$
```
I used both:
```Bash
systemctl
```
for system services and:
```Bash
systemctl --user
```
for my own user-level service.

That distinction became one of the most useful lessons of the day.

## Key Concept 1 — What Is a Linux Service?

A simple way I now understand it is:

> **A process is something running. A service is something systemd manages.**

For example:
```
bluetooth.service
```
is the service.

Its actual process was:
```
/usr/libexec/bluetooth/bluetoothd
```
My system showed:
```
Main PID: 868 (bluetoothd)
```
So:
```
bluetooth.service
       ↓
systemd manages it
       ↓
bluetoothd
       ↓
PID 868
```
This connects Lesson 6 and Lesson 7.

## Key Concept 2 — systemd

I checked the version:
```Bash
systemctl --version
```
My output:
```
systemd 255 (255.4-1ubuntu8.17)
```
`systemd` is responsible for managing many services and other units on the system.

The `systemctl` command is the main tool I used to communicate with systemd.

Official systemd tooling includes commands for listing units, starting and stopping them, checking status, viewing configuration, checking dependencies, enabling/disabling units, and inspecting failed units.

## Key Concept 3 — Listing Services

I started with:
```Bash
systemctl list-units --type=service
```
The first thing that surprised me was the display.

My terminal initially showed something like:
```
UNIT
accounts-daemon.service
alsa-restore.service
apparmor.service
avahi-daemon.service
bluetooth.service
...
```
The terminal was using a pager/display mode, so the normal columns were not immediately obvious.

I then used:
```
systemctl list-units --type=service --no-pager
```
This was much easier to read.

I could now see:
```Text
UNIT                    LOAD   ACTIVE   SUB      DESCRIPTION

accounts-daemon.service loaded active   running  Accounts Service
...
bluetooth.service       loaded active   running  Bluetooth service
...
cups.service            loaded active   running  CUPS Scheduler
...
```
I also saw:
```
casper-md5check.service loaded failed failed
```
This taught me an important documentation lesson:

> When command output is being displayed through a pager, `--no-pager` can make captured terminal evidence much easier to read.

## Understanding `LOAD`, `ACTIVE`, and `SUB`

The output contained:
```
LOAD
ACTIVE
SUB
```
The system itself explained these fields:
```
LOAD   → Reflects whether the unit definition was properly loaded.
ACTIVE → The high-level unit activation state.
SUB    → The low-level unit activation state.
```
I learned not to treat these three words as the same thing.

For example:
```
loaded active running
```
means the service definition is loaded and the service is currently running.

But I also saw:
```
loaded active exited
```
For example:
```
apparmor.service
alsa-restore.service
console-setup.service
```
That was an important lesson.

`active` **does not always mean** `running`.

Some services perform a task and then exit successfully.

## Lesson 4 — A Real Failed Service

One of the most valuable discoveries in this lesson was:
```
casper-md5check.service
```
It appeared as:
```
loaded failed failed
```
I investigated it instead of ignoring it.

I ran:
```Bash
systemctl status casper-md5check.service
```
The result showed:
```
Active: failed (Result: exit-code)
```
and:
```
Main PID: 1213 (code=exited, status=1/FAILURE)
```
The log showed:
```
.fopen md5_file: No such file...
Checking integrity...
Failed to start casper-md5check.service
```
This was a very useful cybersecurity lesson.

A failed service is not automatically evidence of malware.

It is **an observation that requires investigation**.

A security-minded person should ask:

1. What service failed?
2. When did it fail?
3. Why did it fail?
4. What executable did it try to run?
5. What does the journal say?
6. Is the failure expected on this machine?

I should never jump from:

> "service failed"

to:

> "my computer has been hacked."

Evidence first. Conclusions second.

## Lesson 5 — Seeing Only Running Services

I used:
```Bash
systemctl list-units --type=service --state=running
```
My system returned services including:
```
accounts-daemon.service
avahi-daemon.service
bluetooth.service
cron.service
cups.service
dbus.service
fwupd.service
NetworkManager.service
polkit.service
rsyslog.service
systemd-journald.service
...
```
This is useful because a huge list can be overwhelming.

Instead of asking:

> "What services exist?"

I can ask:

> "What services are running right now?"

That is much more useful during an initial investigation.

## Lesson 6 — `list-units` **vs** `list-unit-files`

This was one of the most important distinctions I learned.

I ran:
```Bash
systemctl list-unit-files --type=service
```
This showed entries such as:
```
accounts-daemon.service    enabled
bluetooth.service          enabled
brltty.service             disabled
alsa-utils.service         masked
```
The difference is:
 
`list-units`
 
Shows units currently loaded into systemd's runtime view.

`list-unit-files`

Shows installed service unit files and their configured states.

This means:
```
running now
```
and:
```
enabled to start automatically
```
are **not the same question**.

## Lesson 7 — enabled Does Not Mean running

I checked Bluetooth:
```Bash
systemctl is-active bluetooth.service
```
Output:
```
active
```
Then:
```Bash
systemctl is-enabled bluetooth.service
```
Output:
```
enabled
```
So in this case:
```
active
+
enabled
```
means:

> Bluetooth is currently running and configured to be enabled.

But these are two different properties.

A service can be:
```
active but disabled
```
or:
```
inactive but enabled
```
depending on how it was started and configured.

## Lesson 8 — Inspecting a Service

I investigated:
```Bash
systemctl status bluetooth.service
```
The output gave me much more than just "running":
```
Loaded: loaded
Active: active (running)
Main PID: 868
Tasks: 1
Memory: 3.2M
CPU: 128ms
CGroup: /system.slice/bluetooth.service
```
This was almost like a small profile of the service.

Most importantly:
```
Main PID: 868
```
connected the service to the process I studied in Lesson 6.

I verified it:
```Bash
ps -p 868 -f
```
Output:
```
UID   PID   PPID   C   STIME   TTY   TIME   CMD
root  868   1      0   Aug24   ?     ...    /usr/libexec/bluetooth/bluet...
```
This gave me a very useful mental model:
```
systemd
   ↓
bluetooth.service
   ↓
Main PID 868
   ↓
bluetoothd
```

## Lesson 9 — Inspecting Service Properties

I used:
```Bash
systemctl show bluetooth.service
```
Among the properties I saw:
```
Type=dbus
Restart=on-failure
TimeoutStartUSec=1min 30s
TimeoutStopUSec=10s
MainPID=868
```
I also learned that individual properties can be queried:
```Bash
systemctl show bluetooth.service -p Id
```
Output:
```
Id=bluetooth.service
```
Then:
```Bash
systemctl show bluetooth.service -p ActiveState
```
Output:
```
ActiveState=active
```
And:
```Bash
systemctl show bluetooth.service -p SubState
```
Output:
```
SubState=running
```
Finally:
```Bash
systemctl show bluetooth.service -p MainPID
```
gave:
```
MainPID=868
```
This is extremely useful for automation because scripts can retrieve one exact property rather than parsing a large human-readable status screen.

## Mistakes I Made — Learning Moments
### Mistake 1 — Forgetting the space before -p

I accidentally typed:
```Bash
systemctl show bluetooth.service-p MainPID
```
instead of:
```Bash
systemctl show bluetooth.service -p MainPID
```
This caused the command to behave differently.

### Lesson

Command syntax matters.

> A tiny missing space can completely change how the shell interprets the command.

### Mistake 2 — Using a placeholder literally

I ran:
```Bash
systemctl show SERVICE -p MainPID
```
The output was:
```
MainPID=0
```
The problem was simple:

`SERVICE` was meant as a placeholder.

It was **not** the actual service name.

The correct command was:
```Bash
systemctl show bluetooth -p MainPID
```
which returned:
```
MainPID=868
```
### Lesson

When following technical documentation, I need to distinguish between:

`SERVICE`

meaning "replace this with the real service name"

and:
```
bluetooth.service
```
meaning an actual unit name.

### Mistake 3 — Trying to treat a service name as a command

Earlier, I made a similar conceptual mistake with services:
```
accounts-daemon.service
```
A `.service` name is not normally something I type directly into Bash as a command.

I should use:
```Bash
systemctl status accounts-daemon.service
```
or:
```Bash
systemctl start accounts-daemon.service
```
The service is managed through systemd.

## Lesson 10 — Looking at the Actual Service Definition

I used:
```Bash
systemctl cat bluetooth
```
This showed the unit definition:
```
[Unit]
Description=Bluetooth service
Documentation=man:bluetoothd(8)
ConditionPathIsDirectory=/sys/class/bluetooth

[Service]
Type=dbus
BusName=org.bluez
ExecStart=/usr/libexec/bluetooth/bluetoothd
...
Restart=on-failure
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
LimitNPROC=1
```
Then I found several security-related settings:
```
ProtectHome=true
ProtectSystem=strict
PrivateTmp=true
ProtectKernelTunables=true
ProtectControlGroups=true
```
This was one of the most important security discoveries of the lesson.

## Security Perspective — Services Are Part of the Attack Surface

A service may have:

- a network connection
- a listening port
- a privileged process
- access to files
- access to hardware
- automatic startup
- restart behavior

Therefore:

> Every unnecessary service can potentially increase the attack surface.

But that does **not** mean I should randomly disable services.

A service might be required by:

- networking
- graphics
- printing
- Bluetooth
- security controls
- system logging
- hardware management

Security is not:

> "Turn everything off."

Security is:

> **Understand what is running, why it is running, what it can access, and whether it needs to be exposed.**



My Bluetooth service provided an excellent example.

The service definition included restrictions such as:

ProtectHome=true
ProtectSystem=strict
PrivateTmp=true
ProtectKernelTunables=true
ProtectControlGroups=true

These are examples of systemd service hardening mechanisms. systemd supports settings such as ProtectHome=, ProtectSystem= and PrivateTmp= to restrict what a service can access.

That means the service is not simply:

"Run this program as root."

There can be additional restrictions around what the program is allowed to do.

