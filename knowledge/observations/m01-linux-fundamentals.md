# M1 Observation Notes

## Expectations before investigating

1. **What is a system call?**
   I think a system call is a request made to the host system that runs the kernel.

2. **Why are the shell and the terminal emulator different things?**
   I think the difference is the kernel each one uses: the shell uses the operating system's kernel, while the terminal emulator uses the emulator's own kernel.

3. **What does `uname -a` report?**
   I think it returns details about the system. The `-a` flag suggests it returns all available details.

4. **What does `/etc/os-release` report, and why isn't it the same as `uname -a`?**
   I think the difference is scope: `/etc/os-release` shows when the OS was released, while `uname -a` reports broader details about the system itself.

5. **What lives in `/dev`?**
   I think `/dev` holds files used for the development environment.

6. **What lives in `/proc`?**
   I think `/proc` holds items related to procedures, possibly functions.

7. **What does "everything is a file" mean in practice?**
   I think it means that everything in the Linux filesystem, directories included, is treated as a file in the sense that it can be edited.

8. **How do you list running processes, and what does the kernel have to do with them?**
   I think `ls` has a flag for listing running processes, since it already lists files and directories. I think running processes are stored within the kernel.

## What I learned from research

1. **System call:** how a program asks the kernel to do something it can't do on its own, like reading a file.

2. **Shell vs terminal emulator:**
   - **Shell** (like bash): a command interpreter that parses what you type and runs programs.
   - **Terminal emulator:** the window that displays text and sends your keystrokes to the shell.
   - When you type a command, it goes from the terminal to the shell. The shell then invokes the program, and the program makes the system calls.

3. **`uname -a` vs `/etc/os-release`:**
   - **`uname -a`:** information about the running kernel (name, version,
     hostname, architecture, build info).
   - **`/etc/os-release`:** information from the distro's files (name, version, ID).
   - They differ because one describes the kernel and the other describes the distribution. If we used a virtual machine instead of WSL2, `uname -a`would change. If we also used a different distribution in that VM, `/etc/os-release` would change too.

4. **`/dev` and `/proc`:**
   - **`/dev`:** device files that represent hardware, like disks, terminals, and `/dev/null`.
   - **`/proc`:** a special directory where the kernel exposes information as files, like CPU details, memory usage, and per-process info.

5. **"Everything is a file":** many system resources (devices, processes, kernel data) can be read and written with the same file operations, even though they aren't real files on disk.

6. **Running processes:** you can list them with `ps aux`, which shows PIDs, commands, and CPU and memory use. The kernel creates, schedules, and tracks every process.

## System Identity

### Raw output: `uname -a`
```text
$ uname -a
Linux CANATOY-1063 6.18.54.1-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Tue Oct  6 19:13:07 UTC 2026 x86_64 GNU/Linux
```
**Observation:** Shows information about the kernel I'm using. Since I'm running WSL2 on Windows, the kernel version is 6.18.54.1-microsoft-standard-WSL2. The "microsoft" in the name shows this kernel is built by Microsoft, not by Ubuntu.

### Raw output: `cat /etc/os-release`
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
**Observation:** Shows information about the distribution running on the kernel. In this case, I'm using Ubuntu 26.04.1 LTS. I expected /etc/os-release to show when the OS was released, but it actually shows the distro's name, version, and ID.

### The four facts

| Fact | My value | Source | Where in the output |
| --- | --- | --- | --- |
| Kernel version | 6.18.54.1-microsoft-standard-WSL2 | `uname -a` | 3rd field |
| Kernel architecture | x86_64 | `uname -a` | field before `GNU/Linux` |
| Distro name | Ubuntu | `/etc/os-release` | `NAME=` line |
| Distro version | 26.04.1 LTS | `/etc/os-release` | `VERSION=` line |

### WSL2 vs VM
If I switched from WSL2 to a VM, the kernel version in `uname -a` would change, because a VM boots Ubuntu's own kernel instead of Microsoft's WSL2 kernel. The hostname would change too, but the architecture would likely stay x86_64 on the same CPU. If the same distro and version were installed, `/etc/os-release` would stay the same. This shows the kernel and the distro are independent facts.


## 4. Kernel, shell, terminal: who's who

```text
$ ps -p $$ -o pid,ppid,comm,args
    PID    PPID COMMAND         COMMAND
  12782   12781 bash            bash --rcfile /dev/fd/63
```
**Observation:** Shows the shell I'm using. In this case it's bash, running as process 12782 and started by process 12781.

```text
$ ls -l /proc/$$/fd
total 0
lrwx------ 1 ejandrea ejandrea 64 Oct 13 16:32 0 -> /dev/pts/3
lrwx------ 1 ejandrea ejandrea 64 Oct 13 16:32 1 -> /dev/pts/3
lrwx------ 1 ejandrea ejandrea 64 Oct 13 16:32 2 -> /dev/pts/3
lrwx------ 1 ejandrea ejandrea 64 Oct 13 16:32 255 -> /dev/pts/3
lr-x------ 1 ejandrea ejandrea 64 Oct 13 16:32 63 -> 'pipe:[94755]'
lrwx------ 1 ejandrea ejandrea 64 Oct 13 16:32 7 -> /dev/ptmx
```
**Observation:** Shows the connections my shell has open. Input (0), output (1) and errors (2) all go through /dev/pts/3, the pseudo-terminal that connects my shell to the terminal.

```text
$ type cd
cd is a shell builtin
```
**Observation:** Shows that cd is built into bash, so bash runs it itself without starting a separate program.

```text
$ type -a ls
ls is aliased to `ls --color=auto'
ls is /usr/bin/ls
ls is /bin/ls
```
**Observation:** Shows that ls is a separate program at /usr/bin/ls, so bash has to ask the kernel to start it. The alias just adds colors.

```text
$ ps -p 12781 -o pid,ppid,comm,args
    PID    PPID COMMAND         COMMAND
  12781   12780 Relay(12782)    /init
```

**Observation:** Shows what started my shell. It's not the terminal app but Relay(12782), a WSL helper that passes input and output between my shell and the terminal window on Windows.

**Expected vs actual:** I expected the shell and the terminal emulator to use different kernels (expectation #2), but they're actually two separate programs connected through /dev/pts/3.

## 5. The filesystem as an abstraction

```text
$ ls -l /dev | head -10
total 0
crw-r--r-- 1 root root     10, 235 Oct 13 12:36 autofs
drwxr-xr-x 2 root root        600 Oct 13 12:36 block
drwxr-xr-x 2 root root        120 Oct 13 12:36 bsg
crw------- 1 root root     10, 234 Oct 13 12:36 btrfs-control
drwxr-xr-x 2 root root       2860 Oct 13 12:36 char
crw------- 1 root root      5,   1 Oct 13 12:36 console
lrwxrwxrwx 1 root root         11 Oct 13 12:36 core -> /proc/kcore
crw------- 1 root root     10, 260 Oct 13 12:36 cpu_dma_latency
crw------- 1 root root     10, 203 Oct 13 12:36 cuse
```

**Observation:** Shows the first entries in /dev. Most start with c, meaning character devices like console, so /dev holds devices shown as files.


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

**Observation:** Shows that /proc/cpuinfo has a file size of 0. Then shows real information about my CPU even though the size was 0. The file isn't stored on disk, the kernel creates the content when I read it.

**Expected vs actual:** I expected /dev to be for the development environment and /proc to be for procedures (expectations #5 and #6), but /dev holds devices and /proc holds live kernel information, both shown as files.

## 6. What's running right now

```text
$ ps -ef | head -20
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 12:36 ?        00:00:01 /sbin/init
root           2       1  0 12:36 hvc0     00:00:01 /init
root           3       2  0 12:36 hvc0     00:00:00 init-watcher
root           7       2  0 12:36 hvc0     00:00:31 plan9 --control-socket 7 --log-level 4 --server-fd 8 --pipe-fd 10 --log-truncate
root         219       1  0 12:36 ?        00:00:00 /usr/lib/systemd/systemd-journald
systemd+     277       1  0 12:36 ?        00:00:00 /usr/lib/systemd/systemd-resolved
root         286       1  0 12:36 ?        00:00:27 /usr/lib/systemd/systemd-udevd
root         336       1  0 12:36 ?        00:00:00 /bin/sh /usr/lib/systemd/scripts/chronyd-starter.sh -n -F 1
root         341       1  0 12:36 ?        00:00:00 /usr/sbin/cron -f -P
message+     346       1  0 12:36 ?        00:00:00 @dbus-daemon --system --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only
root         358       1  0 12:36 ?        00:00:00 /usr/bin/python3 /usr/bin/networkd-dispatcher --run-startup-triggers
root         362       1  0 12:36 ?        00:00:00 /usr/lib/systemd/systemd-logind
syslog       408       1  0 12:36 ?        00:00:00 /usr/sbin/rsyslogd -n -iNONE
_chrony      463     336  0 12:36 ?        00:00:00 /usr/sbin/chronyd -n -F 1 -x
_chrony      467     463  0 12:36 ?        00:00:00 /usr/sbin/chronyd -n -F 1 -x
root         472       1  0 12:36 ?        00:00:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
root         474       1  0 12:36 tty1     00:00:00 /usr/sbin/agetty --noreset --noclear --issue-file=/etc/issue:/etc/issue.d:/run/issue.d:/usr/lib/issue.d - linux
root         525       2  0 12:36 ?        00:00:00 login -- ejandrea
ejandrea     656       1  0 12:36 ?        00:00:00 /usr/lib/systemd/systemd --user
```

**Observation:** Shows the first 20 running processes. PID 1 is /sbin/init, which started most of the services, and there are WSL-specific processes too, like WSL's own /init and plan9.

```text
$ ps -e --no-headers | wc -l
48
```

**Observation:** Shows that 48 processes are running.

```text
$ cat /proc/1/comm
systemd
```

**Observation:** Shows that process 1 is systemd, so my WSL2 runs systemd like a normal Ubuntu system.

**Expected vs actual**: I expected ls to have a flag for listing processes (expectation #8), but processes are listed with ps, which reads the process information the kernel keeps.


## From keypress to output

When I type `ls` and press Enter:
1. The terminal window on Windows sends my keystrokes through WSL's relay into `/dev/pts/3`, and bash reads them (fd 0).
2. Bash sees `ls` isn't a builtin, finds it at `/usr/bin/ls`, and asks the kernel to start it. That request is a system call.
3. `ls` asks the kernel to read the directory, using more system calls.
4. `ls` writes the result to fd 1 (`/dev/pts/3`), and the relay passes it back to the terminal window.
5. Bash shows the prompt again.

All of this runs on the kernel 6.18.54.1-microsoft-standard-WSL2 (from `uname -a`), and bash and `ls` come from Ubuntu 26.04.1 LTS (from `/etc/os-release`).