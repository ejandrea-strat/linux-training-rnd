# Linux Fundamentals Write-up

## What I built

Observation notes that record which kernel and distributino my system is running. My setup is WSL2 on Windows, running Ubuntu 26.04.1 LTS on the kernel 6.18.54.1-microsoft-standard-WSL2. So far, the notes cover my expectations before investigating, what I learned from reserach, and the system identity lab (uname -a, /etc/os-release, the four identity facts, and WSL2 vs VM). Still to come: shell vs terminal, /dev and /proc, and running processes.

## Why it's built this way (key decisions)

- I researched the brief's four "Research:" questions using OpenCode as a tutor, asking it to explai and quiz me instead of writing my notes.
- I ran `uname -a` and `cat /etc/os-release` first, because kernel and distro are the first two facts to check when something behaves differently across machines.
- What surprised me: I expected /etc/os-release to show when the OS was released, but it actually shows the distro's name, version, and ID. I also didn't expect the kernel name to include "microsoft". On WSL2 the kernel is built by Microsoft, while Ubuntu only provides the distro on top of it.

## How to build it (teach it to the next trainee)

Write a guide for someone who has never looked under the hood of their own machine before: what to
check, in what order, and how to tell you're looking at real evidence rather than assuming.

## Concepts worth explaining

Pick 1–2 ideas from the brief's Scope (kernel vs. shell vs. distro, how a command travels to output, or
the filesystem as abstraction) and explain each in your own words.

## What tripped me up

- I thought the shell makes the system calls when I run a command. It actually only parses the command and launches the program. The program (like `ls`) is the one making the system calls.

## Checkpoint evidence

```text
$ uname -a
Linux CANATOY-1063 6.18.54.1-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Tue Oct  6 19:13:07 UTC 2026 x86_64 GNU/Linux
```

```text
$ cat /etc/os-release
PRETTY_NAME="Ubuntu 26.04.1 LTS"
NAME="Ubuntu"
VERSION_ID="26.04"
VERSION="26.04.1 LTS (Resolute Raccoon)"
VERSION_CODENAME=resolute
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=resolute
LOGO=ubuntu-logo
```

## What I'd do differently

If you started this module over, what would you check first instead?
