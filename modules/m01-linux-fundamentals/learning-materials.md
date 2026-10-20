# M1 Learning Materials — Linux Fundamentals

> **Purpose.** A self-contained study guide for every concept, command, and question named in
> [`brief.md`](./brief.md). It teaches the material; it does **not** replace the lab. Work the lab
> on your own machine and record what really happened in your own
> [`write-up-template.md`](./write-up-template.md).
>
> **Audience.** Data engineers who are new to Linux internals. No prior exposure to kernels,
> processes, or the Linux filesystem is assumed, and every command is explained before it is used.
> You need the Linux environment set up in M0 (WSL2, a VM, or Linux-in-a-container) and a terminal.
>
> **Scope note.** Every example output below is **illustrative** unless it is explicitly labeled as
> coming from the author's observation notes
> ([`knowledge/observations/m01-linux-fundamentals.md`](../../knowledge/observations/m01-linux-fundamentals.md)).
> The labeled outputs were captured on WSL2 on a specific machine; values such as hostname, PID,
> kernel build date, and CPU model will differ on yours. Never copy an example as evidence — run the
> command and paste your own output.
>
> **How to use this guide.** Each section follows **Understand → Try → Verify → Continue**:
> read the explanation, run the commands on your own system, confirm your output matches the
> *shape* of the examples, then move on. The guided path takes roughly **60–90 minutes**. M1 shares
> Week 1 of the course with M2 (5–6 estimated hours in total).

## Contents

1. [Kernel, shell, and distro](#1-kernel-shell-and-distro)
2. [System identity](#2-system-identity)
3. [The filesystem as an abstraction](#3-the-filesystem-as-an-abstraction)
4. [A live system](#4-a-live-system)
5. [Worked example: the M1 lab](#5-worked-example-the-m1-lab)
6. [Putting it together: from keypress to output](#6-putting-it-together-from-keypress-to-output)
7. [Completion checklist](#7-completion-checklist)
8. [Best practices and troubleshooting](#8-best-practices-and-troubleshooting)
9. [References](#9-references)

---

## 1. Kernel, shell, and distro

_Estimated: 10 minutes._

**Understand.** Three things are easy to blur together, so separate them first:

| Thing | What it is | Job in one line |
| --- | --- | --- |
| **Kernel** | The core program that owns the hardware | Decides which process runs, and performs every privileged operation on their behalf |
| **Shell** | An ordinary program (e.g. `bash`) | Reads what you type, interprets it, and starts other programs |
| **Distro** | A packaged collection of kernel + tools + shell + defaults (e.g. Ubuntu) | Decides which programs exist and how they behave by default |

A **system call** is how a program asks the kernel to do something it cannot do by itself — read a
file, write to the terminal, start another process. The kernel runs in a privileged mode; user
programs cannot touch hardware directly, so they must ask. When you run a command, the shell parses
it, and the program the shell starts makes system calls; the kernel performs them and returns the
result.

The **shell** and the **terminal emulator** are two different programs. The terminal emulator is the
window (on WSL2, a Windows application) that displays text and collects keystrokes. The shell is
what interprets those keystrokes as commands. They are connected by a **pseudo-terminal** — a device
file under `/dev/pts/` that carries input in one direction and output in the other.

**Try.** Inspect your own shell instead of taking this guide's word for it:

```bash
ps -p $$ -o pid,ppid,comm,args      # your shell process: PID, parent PID, name, full command
ps -p $$ -o ppid=                   # just the parent PID, for the next command
```

> **Use your own values.** The examples below contain PIDs from the author's machine
> (`12782`, `12781`). Substitute the PID and PPID that *your* two commands print.

Output from the author's observation notes:

```text
$ ps -p $$ -o pid,ppid,comm,args
    PID    PPID COMMAND         COMMAND
  12782   12781 bash            bash --rcfile /dev/fd/63
```

```bash
ls -l /proc/$$/fd                   # the files your shell has open
type cd                             # builtin or separate program?
type -a ls                          # every meaning of ls: alias, then real path
```

Output from the author's observation notes:

```text
$ ls -l /proc/$$/fd
total 0
lrwx------ 1 ejandrea ejandrea 64 Oct 13 16:32 0 -> /dev/pts/3
lrwx------ 1 ejandrea ejandrea 64 Oct 13 16:32 1 -> /dev/pts/3
lrwx------ 1 ejandrea ejandrea 64 Oct 13 16:32 2 -> /dev/pts/3
lrwx------ 1 ejandrea ejandrea 64 Oct 13 16:32 255 -> /dev/pts/3
lr-x------ 1 ejandrea ejandrea 64 Oct 13 16:32 63 -> 'pipe:[94755]'
lrwx------ 1 ejandrea ejandrea 64 Oct 13 16:32 7 -> /dev/ptmx

$ type cd
cd is a shell builtin

$ type -a ls
ls is aliased to `ls --color=auto'
ls is /usr/bin/ls
ls is /bin/ls
```

Two lessons: `cd` is handled by the shell itself (a *builtin* — no separate program runs), while
`ls` is a real program on disk that the shell must ask the kernel to start. The two paths
`/usr/bin/ls` and `/bin/ls` are the same program: on Ubuntu, `/bin` is a link to `/usr/bin`. And
file descriptors 0, 1, and 2 — standard input, output, and error — all point at `/dev/pts/3`, the
pseudo-terminal that links the shell to the terminal window.

Finally, ask what started your shell (`ps -p <your shell's PPID> -o pid,ppid,comm,args`). On WSL2
it is not the Windows terminal application but a relay process that passes text between Windows and
your Linux session — for the author:

```text
$ ps -p 12781 -o pid,ppid,comm,args
    PID    PPID COMMAND         COMMAND
  12781   12780 Relay(12782)    /init
```

> **Common misconception:** *the shell and the terminal emulator each run on their own kernel.*
> **Actually:** neither one has its own kernel; they are ordinary programs connected through a
> pseudo-terminal (`/dev/pts/3` here). On WSL2 the terminal runs on Windows and the shell runs on
> the Linux kernel.

**Verify.** `ps -p $$` shows a shell process; `ls -l /proc/$$/fd` shows fds 0, 1, 2 pointing at a
`/dev/pts/N` device; `type cd` says *builtin*; `type -a ls` shows a path.

**Continue** to [§2](#2-system-identity) to learn where the kernel and distro announce their identities.

---

## 2. System identity

_Estimated: 10 minutes._

**Understand.** Two commands answer "what am I running?" from two different angles:

- `uname -a` reports facts about the **running kernel**: kernel name, hostname, kernel release,
  build string, machine architecture, and operating system name.
- `cat /etc/os-release` reports facts written by the **distribution**: its name, version, ID, and
  links. It is a plain file of `KEY=value` lines, so you can read it with `cat`.

They are not the same fact because they are not describing the same layer: the kernel can be
replaced (a VM boots Ubuntu's kernel; WSL2 runs a Microsoft-built kernel) while the distribution
files stay put, and vice versa.

**Try.**

```bash
uname -a
cat /etc/os-release
```

Output from the author's observation notes (WSL2 on Windows):

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

Reading the fields:

| `uname -a` field | Meaning |
| --- | --- |
| `Linux` | Kernel name |
| `CANATOY-1063` | Hostname (on WSL2 this is the Windows computer name by default; your own will differ) |
| `6.18.54.1-microsoft-standard-WSL2` | Kernel release — the *version* of the kernel |
| `#1 SMP PREEMPT_DYNAMIC Tue Oct 6 19:13:07 UTC 2026` | Build number and build date — `man uname` calls this the "kernel version" (`uname -v`); the kernel version this lab asks for is the kernel release (`uname -r`) |
| `x86_64` | Machine architecture (CPU family the kernel was built for) |
| `GNU/Linux` | Operating system name reported by the kernel |

| `/etc/os-release` field | Meaning |
| --- | --- |
| `PRETTY_NAME` / `NAME` | Human-readable distribution name |
| `VERSION` / `VERSION_ID` | Distribution version, and the script-friendly version number |
| `VERSION_CODENAME` / `UBUNTU_CODENAME` | Release nicknames |
| `ID` / `ID_LIKE` | Machine-readable distro identifier and its family (Ubuntu inherits from Debian) |
| `HOME_URL`, `SUPPORT_URL`, `BUG_REPORT_URL`, `PRIVACY_POLICY_URL` | Where the distro points for help |
| `LOGO` | Icon name used by graphical front-ends |

> **Common misconception:** *`/etc/os-release` shows when the operating system was released.*
> **Actually:** it reports the distribution's identity — name, version, ID, and support links.
> Release *dates* are not part of it.

**Verify.** Both commands print output on your machine. In the author's WSL2 output, `uname -a` says
nothing about Ubuntu, and `/etc/os-release` says nothing about the kernel. Extracting the four headline facts and
comparing WSL2 with a VM happens in **[§5](#5-worked-example-the-m1-lab)**.

**Continue** to [§3](#3-the-filesystem-as-an-abstraction) to see what else the kernel exposes — as files.

---

## 3. The filesystem as an abstraction

_Estimated: 10 minutes._

**Understand.** In Linux, "a file" is better read as "a thing you can open, read, write, and close".
Two directories make that literal:

- **`/dev`** holds **device files**. Each one represents a piece of hardware or a kernel service —
  disks, terminals, `/dev/null`. In `ls -l` output the first character tells you the type:
  `c` = character device, read and written as a stream of bytes (e.g. terminals);
  `b` = block device, read and written in fixed-size blocks (e.g. disks);
  `d` = directory; `l` = link. In the author's listing, `core -> /proc/kcore` is a link pointing
  from `/dev` into `/proc`, so the two directories are not as separate as they first look.
- **`/proc`** is a **pseudo-filesystem**: a window into the kernel's own data structures. There are
  no bytes on disk behind it. The kernel generates file contents the moment you read them, and the
  per-process directories (`/proc/<pid>/`) are how tools like `ps` learn what is running.

"What about 'everything is a file'?" It means these non-file resources — devices, processes, kernel
tunables — are reached with the **same operations** used for ordinary files: open, read, write,
close. It does not mean everything is an editable document; many entries under `/proc` are
read-only, and writing to the wrong one changes kernel behavior.

**Try.**

```bash
ls -l /dev | head -10              # device files and their types (c, b, d, l)
ls -l /proc/cpuinfo                # a kernel-generated file: note the 0-byte size
head -5 /proc/cpuinfo              # reading it returns real CPU details
```

Output from the author's observation notes:

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

The `/proc/cpuinfo` example is the whole idea in two commands: its size is **0 bytes**, yet reading
it returns real CPU details. Nothing is stored on disk — the kernel produces the content on demand.

> **Common misconception:** *`/dev` is for the "development" environment, and `/proc` is for
> "procedures".* **Actually:** `/dev` is **devices**; `/proc` is **process and kernel information**.

> **Common misconception:** *"everything is a file" means every entry can be edited.*
> **Actually:** it means the same file operations reach devices and kernel data. Permissions still
> apply, and most of `/proc` is read-only.

**Verify.** Your `/dev` listing contains entries whose mode starts with `c` or `b`, and
`/proc/cpuinfo` reports a 0-byte size while still printing CPU lines.

**Continue** to [§4](#4-a-live-system) to use `/proc` to see what is running right now.

---

## 4. A live system

_Estimated: 10 minutes._

**Understand.** Every running program is a **process**: a program in memory with a **PID** (process
ID) and a **PPID** (the PID of the process that started it). The kernel creates processes, decides
which one runs on the CPU and when, tracks their state, and exposes them through `/proc`. `ps` is
just a reader of that information.

**Try.**

```bash
ps -ef | head -20                              # snapshot of processes, full format
ps -e --no-headers | wc -l                     # how many processes are running
cat /proc/1/comm                               # what is PID 1?
```

Output from the author's observation notes:

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

$ ps -e --no-headers | wc -l
48

$ cat /proc/1/comm
systemd
```

Read the columns: `PID` and `PPID` give the process and its parent; the PPID column shows the
parent-child tree (PID 1 starts most services). The count is machine-specific — 48 on the author's
system. PID 1 appears as `/sbin/init`, which *is* systemd — hence `cat /proc/1/comm` printing
`systemd`, the same service manager a normal Ubuntu installation boots. PID 2 (`/init`) and
`plan9` are WSL's own processes rather than distro services: `/init` starts the session, and
`plan9` lets Windows read the Linux files.

> **Common misconception:** *`ls` has a flag for listing running processes.*
> **Actually:** use `ps` (or `top`). `ps` reads the process information the kernel keeps; `ls` only
> ever lists filesystem entries.

**Verify.** `ps -ef` shows a PID/PPID tree headed by PID 1; your process count prints a number;
`cat /proc/1/comm` names your init system.

**Continue** to [§5](#5-worked-example-the-m1-lab), where you produce the lab evidence.

---

## 5. Worked example: the M1 lab

> **Purpose.** The three lab steps from [`brief.md`](./brief.md), with the exact evidence to capture.
> The outputs shown are the author's; yours will differ in hostname, kernel build, CPU, and distro
> values. Record your own.

_Estimated: 20–30 minutes._

### Step 1 — Capture the raw output

**Understand.** The lab is a measurement, not a summary. Run both commands in your own Linux
environment and capture the output exactly as it appears.

**Try.**

```bash
uname -a
cat /etc/os-release
```

Raw output from the author's observation notes (WSL2 on Windows, Ubuntu 26.04.1 LTS, kernel
6.18.54.1-microsoft-standard-WSL2):

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

**Capture.** Paste the raw output verbatim into your observation notes, command line included:

- Keep the `$ uname -a` and `$ cat /etc/os-release` prompts next to their output, so a reader can
  tell which command produced what.
- Do not paraphrase, reorder, or tidy the output — an edited transcript is not evidence.
- The full `/etc/os-release` is short; paste all of it rather than trimming to two lines.
- If you copy from a terminal, copy the output as text, not a screenshot, so it stays searchable.

### Step 2 — Identify the four facts

**Understand.** Four facts live in that output, spread across both commands.

**Try.** Fill in the table using your own captured output. The author's answers:

| Fact | Value in the author's output | Source command | Where in the output |
| --- | --- | --- | --- |
| Kernel version | `6.18.54.1-microsoft-standard-WSL2` | `uname -a` | 3rd field |
| Kernel architecture | `x86_64` | `uname -a` | field before `GNU/Linux` |
| Distribution name | `Ubuntu` | `cat /etc/os-release` | the `NAME=` line |
| Distribution version | `26.04.1 LTS` | `cat /etc/os-release` | the `VERSION=` line |

**Capture.** Record all four values *and* the position you read each one from. The "where" column is
the part that proves the fact came from the output rather than from memory.

### Step 3 — WSL2 versus Virtual Machine

**Understand.** The point of this step is that **kernel and distribution are independent facts**.
They are supplied by different layers, so changing one path need not change the other.

**Try.** Answer in one or two sentences: what would change in step 1's output if you switched
between the WSL2 path and the Virtual Machine path?

The author's answer, in substance:

> Switching from WSL2 to a VM changes the kernel version in `uname -a`, because a VM boots Ubuntu's
> own kernel instead of Microsoft's WSL2 kernel. The hostname changes too, but the architecture
> likely stays `x86_64` on the same CPU. If the same distro and version were installed,
> `/etc/os-release` would stay the same — which shows the kernel and the distro are independent
> facts.

For contrast, here is what the kernel line looks like on a VM running the same distribution
(**illustrative example — not captured output**):

```text
$ uname -a
Linux ubuntu-vm <version>-generic #<build>-Ubuntu SMP PREEMPT_DYNAMIC ... x86_64 GNU/Linux
```

Same distribution, different kernel: `uname -a` would change, `/etc/os-release` would not.

**Verify.** Your notes contain the raw output, the four facts with their locations, and a one- or
two-sentence WSL2-versus-VM explanation.

**Clean-up.** None — this lab only reads system information.

**Continue** to [§6](#6-putting-it-together-from-keypress-to-output) to tie the lab together into the course's headline question.

---

## 6. Putting it together: from keypress to output

_Estimated: 5 minutes._

**Understand.** The whole module resolves into one sequence. From the author's observation notes,
typing `ls` and pressing Enter:

1. The terminal window on Windows sends the keystrokes through WSL's relay into `/dev/pts/3`, and
   bash reads them as file descriptor 0.
2. Bash sees that `ls` is not a builtin, finds the program at `/usr/bin/ls`, and asks the kernel to
   start it. That request is a system call.
3. `ls` asks the kernel to read the directory — more system calls.
4. `ls` writes the result to file descriptor 1 (`/dev/pts/3`), and the relay passes it back to the
   terminal window.
5. Bash prints the prompt again.

Every layer from [§1](#1-kernel-shell-and-distro)–[§4](#4-a-live-system) appears here: the **terminal emulator** collects input; the **shell**
interprets it; the **kernel** starts the process and performs the directory read; the **filesystem
as an abstraction** supplies `/dev/pts/3` and `/usr/bin/ls`; and the **running processes** are the
shell and `ls` themselves. All of it runs on the kernel reported by `uname -a`
(`6.18.54.1-microsoft-standard-WSL2` in the author's notes), with the programs supplied by the
distribution reported by `/etc/os-release` (Ubuntu 26.04.1 LTS).

**Try.** Tell the same story using your own two outputs: name your kernel from `uname -a` and your
distribution from `/etc/os-release` as the two layers involved.

**Verify.** You can answer, unprompted: *"What happens between typing a command and seeing
output?"* This is the course's self-check at the M1–M2 boundary.

**Continue** to [§7](#7-completion-checklist) to confirm the deliverable is complete.

---

## 7. Completion checklist

_Estimated: 5 minutes._

Map your observation notes to the definition of done in [`brief.md`](./brief.md):

- [ ] **DoD-01 — Real output, not paraphrase.** The notes show the raw, pasted output of `uname -a`
      and `cat /etc/os-release` from your own terminal (Lab step 1), not facts copied from
      documentation or written from memory. *Verify via [§5](#5-worked-example-the-m1-lab), Step 1.*
- [ ] **DoD-02 — The four facts, located.** The notes name the kernel version, kernel architecture,
      distribution name, and distribution version, and show where in the output each one appears.
      *Verify via [§5](#5-worked-example-the-m1-lab), Step 2.*
- [ ] **DoD-03 — Kernel and distro are independent.** The notes explain in one or two sentences what
      would change in step 1's output when switching between the WSL2 and Virtual Machine paths.
      *Verify via [§5](#5-worked-example-the-m1-lab), Step 3.*
- [ ] **DoD-04 — The headline question.** Using your own output as evidence, you can answer
      unprompted: *"What happens between typing a command and seeing output?"* *Verify via [§6](#6-putting-it-together-from-keypress-to-output).*

When all four are ticked, transfer the evidence into your write-up
([`write-up-template.md`](./write-up-template.md)) and continue to M2.

---

## 8. Best practices and troubleshooting

_Estimated: 5 minutes._

### Best practices

- **Treat kernel and distro version as the first two facts to check** when something behaves
  differently across machines. Most "works on my machine" surprises start there ([§2](#2-system-identity)).
- **Do not assume distro-specific defaults transfer.** Package manager, default shell, and service
  manager differ between distros, and none of them is safe to guess ([§1](#1-kernel-shell-and-distro)).
- **Keep "the shell" and "the terminal emulator" distinct.** Confusing the window with the command
  interpreter leads to wrong conclusions about where problems live ([§1](#1-kernel-shell-and-distro)).

### Troubleshooting

| Symptom | Likely cause | Recommended fix |
| --- | --- | --- |
| `uname -a` shows `microsoft-standard-WSL2` in the kernel release | This is expected under WSL2: Microsoft builds the kernel, while the distribution files come from Ubuntu. It is not a mis-installed distro, and the two layers are independent ([§2](#2-system-identity)). | Nothing to fix. If you need Ubuntu's own kernel, use the Virtual Machine path instead — `uname -a` changes, `/etc/os-release` need not. |
| The terminal application never appears in `ps` output | The terminal itself runs on Windows. Only its WSL-side relay appears inside the Linux session (e.g. `Relay(...)` or `/init`, as in the author's notes). | Expect only the relay process. Use `ps -ef` and look for `/init` or `Relay`; check the shell's PPID with `ps -p $$ -o ppid=`. |
| `type l` reports an alias instead of a path | `l` is a shortcut alias for `ls -CF`, not a separate program. | Check the command you actually meant: `type -a ls` shows the alias and the real program path (`/usr/bin/ls`). |
| PID 1 is not `systemd` | systemd is not enabled for this WSL2 instance, so another init runs as PID 1. | Check the setting and enable it explicitly in `/etc/wsl.conf` under `[boot]` (`systemd=true`), then restart the WSL2 instance (`wsl --shutdown` from Windows) and re-run `cat /proc/1/comm`. |

---

## 9. References

Everything here is documented on your own machine; the offline manual pages are the primary
reference:

- `man uname` — every field `uname -a` prints.
- `man 5 os-release` — the `/etc/os-release` file format and all its keys.
- `man 5 proc` — the `/proc` pseudo-filesystem, including `/proc/pid/` and `/proc/cpuinfo`.
- `man ps` — listing processes and the meaning of each output column.

Online copies of the same pages:

- [`uname(1)`](https://man7.org/linux/man-pages/man1/uname.1.html)
- [`os-release(5)`](https://man7.org/linux/man-pages/man5/os-release.5.html)
- [`proc(5)`](https://man7.org/linux/man-pages/man5/proc.5.html)
- [`ps(1)`](https://man7.org/linux/man-pages/man1/ps.1.html)

Module artifacts:

- [`brief.md`](./brief.md) — objective, scope, lab, definition of done.
- [`tasks.md`](./tasks.md) — the ordered checklist for this module.
- [`write-up-template.md`](./write-up-template.md) — where the lab evidence goes.
- [`knowledge/observations/m01-linux-fundamentals.md`](../../knowledge/observations/m01-linux-fundamentals.md)
  — the author's captured output, used as the labeled examples in this guide.
- Course architecture: [`docs/arch-docs/linux-training.md`](../../docs/arch-docs/linux-training.md) —
  where M1 sits in the curriculum (Layer 1, Week 1, with M2).
- Curriculum resource for this stage: [What Is Linux? From Kernel to IoT, DevOps, & Supercomputers](https://www.youtube.com/watch?v=H8Q64gHNp6Y).
