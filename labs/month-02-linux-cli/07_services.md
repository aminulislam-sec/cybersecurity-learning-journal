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

The terminal was using a pager/display mode, so the normal columns were not immediately obvious.

I then used:
```Bash
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

> When command output is being displayed through a pager, --no-pager can make captured terminal evidence much easier to read.


