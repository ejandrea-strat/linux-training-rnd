# Linux Fundamentals Write-up

## What I built

Observation notes that record which kernel and distribution my system is running. My setup is WSL2 on Windows, running Ubuntu 26.04.1 LTS on the kernel 6.18.54.1-microsoft-standard-WSL2. The notes cover my expectations before investigating, what I learned from research, the system identity lab (uname -a, /etc/os-release, the four identity facts, and WSL2 vs VM), shell vs terminal, /dev and /proc, the running processes, and how a command goes from keypress to output.

## Why it's built this way (key decisions)

- I followed EPAV: Evaluate (OpenCode explained the brief's Research questions and quizzed me), Plan (I wrote my expectations and decided the order to check things), Apply (I ran every command myself, since the DoD needs output from my own terminal), and Validate (/validate passed all four DoD items)
- I researched the brief's four "Research:" questions using OpenCode as a tutor, asking it to explain and quiz me instead of writing my notes.
- I ran `uname -a` and `cat /etc/os-release` first, because kernel and distro are the first two facts to check when something behaves differently across machines.
- What surprised me: I expected /etc/os-release to show when the OS was released, but it actually shows the distro's name, version, and ID. I also didn't expect the kernel name to include "microsoft". On WSL2 the kernel is built by Microsoft, while Ubuntu only provides the distro on top of it.
- What surprised me about what was running: my shell's parent isn't the terminal app but a WSL `Relay` process, because the terminal window runs on Windows. WSL also runs its own processes, like `/init` and `plan9`, next to systemd.

## How to build it (teach it to the next trainee)

1. Write down what you expect for each Research question before looking anything up.
2. Run `uname -a` and `cat /etc/os-release`, paste the output exactly, and find the kernel version, architecture, distro name and distro version.
3. Check your shell with `ps -p $$ -o pid,ppid,comm,args`, then run the same command on the parent PID to see what started your shell. Then run `ls -l /proc/$$/fd`, `type cd` and `type -a ls`.
4. Look at `/dev` with `ls -l /dev | head -10`, then compare `ls -l /proc/cpuinfo` with `head -5 /proc/cpuinfo`.
5. List processes with `ps -ef | head -20` and check PID 1 with `cat /proc/1/comm`.

**To make sure you're looking at real evidence and not assuming:** only write down a fact if you can point to the line in your own output that shows it.

## Concepts worth explaining


**Kernel vs distro:** My kernel is built by Microsoft (6.18.54.1-microsoft-standard-WSL2), but my distro is Ubuntu 26.04.1 LTS. `uname -a` asks the running kernel, while `/etc/os-release` is a file from the distro, so they're separate facts. A VM would change the kernel but could keep the same Ubuntu.

**`/proc` as an abstraction:** `/proc/cpuinfo` has a size of 0, but reading it shows my real CPU. The kernel creates the content when I read it, so nothing is stored on disk. That's what "everything is a file" means in practice.

## What tripped me up

- I thought the shell makes the system calls when I run a command. It actually only parses the command and launches the program. The program (like `ls`) is the one making the system calls.
- I typed `type l` instead of `type ls` and got an alias for `ls -CF`. `type -a ls` shows the full picture.

## Checkpoint evidence

**Kernel and distro (lab):**

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

**My shell's parent is WSL's Relay, not the terminal app:**
```text
$ ps -p $$ -o pid,ppid,comm,args
    PID    PPID COMMAND         COMMAND
  12782   12781 bash            bash --rcfile /dev/fd/63

$ ps -p 12781 -o pid,ppid,comm,args
    PID    PPID COMMAND         COMMAND
  12781   12780 Relay(12782)    /init
```

**`cd` is a builtin, `ls` is a separate program (and my `type l` typo):**
```text
$ type cd
cd is a shell builtin

$ type -a ls
ls is aliased to `ls --color=auto'
ls is /usr/bin/ls
ls is /bin/ls

$ type l
l is aliased to `ls -CF'
```

**`/proc/cpuinfo` has a size of 0 but still has content:**
```text
$ ls -l /proc/cpuinfo
-r--r--r-- 1 root root 0 Oct 13 16:32 /proc/cpuinfo

$ head -5 /proc/cpuinfo
processor	: 0
vendor_id	: GenuineIntel
cpu family	: 6
model		: 186
model name	: Intel(R) Core(TM) 5 120U
```

**PID 1 is systemd:**
```text
$ cat /proc/1/comm
systemd
```

## What I'd do differently

I'd check `uname -r` and `cat /proc/1/comm` first, since those two lines tell me which kernel I'm on and whether systemd is running. I'd also double-check commands before running them, like when I typed `type l` instead of `type ls`.
